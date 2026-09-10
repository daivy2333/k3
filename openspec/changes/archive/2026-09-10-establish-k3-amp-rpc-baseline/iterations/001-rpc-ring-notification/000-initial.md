# Iteration 001 / Cycle 000: Ring、RPC、通知与主题收尾

## Plan Context

- Status: ready
- Iteration: 001-rpc-ring-notification
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: `../000-amp-shared-memory-lifecycle/000-initial.md`

**Iteration Scope**

- Change tasks: T3-T7
- Depends on: Iteration 000 Review Result `accepted`
- Stable baseline: 两篇 AMP 正文、来源、G4/G5/G7/G11、术语和入口一致；后续工作无需回读 R09-R12 才能理解协议证据边界
- Verification boundary: 数据与通知分层、RPC 正常/错误/等待/reset 路径完整；缺失源码和运行边界明确；引用与计数一致
- Diagnostic boundary: ring/原子、BUSY/doorbell、mailbox/IRQ/waker、RPC completion/error/cancel、reset recovery、缺口、术语和导航
- Deferred tasks: None

**Cycle Scope**

- Trigger: initial
- Acceptance gaps: None
- Repair items: None
- Inherited scope: proposal R2-R6；M01-M04；D04-D08；Iteration 000 已接受的生命周期与来源基线
- Excluded scope: 补仓、构建、真板测试、协议或驱动实现、全局状态同步、修改既有主题正文

**Objective**

创建可独立阅读的 K3 ring、RPC 与通知事实包，并完成来源、缺口、术语和入口收尾。读者应能区分共享 ring channel 与 mailbox hardware channel，并追踪正常、错误、超时、取消、并发和 reset 路径的已知事实与未知边界。

**Current Baseline**

- Iteration 000 已 accepted；生命周期正文 205 行，覆盖表为 70 个 URL/70 个唯一 URL，字段说明已允许 `amp`。
- `docs/amp/k3-rpc-ring-notification.md` 不存在，`test ! -e` 退出 0。
- `docs/reference/known-gaps.md` 有 G1-G10；G4/G5 为 `partial`、G7 为 `open`，没有 G11。
- `docs/reference/terminology.md` 有 20 项，没有 AMP、RPC、ring、doorbell、共享窗口。
- `docs/index.md` 没有 AMP 主题入口；当前计数为 70 URL、10 gaps、20 个术语。
- Rt-Async-AMP 与 tgoskits 固定 revision 保持为 `ccb1ff0b487e4f49ea570c41f330741eecece935` 和 `19219411d5dc1515496f910d04c93da12ee95be4`。
- `rt-async/modules/platform` 与 `modules/ov-channels` 缺失；前者使 offline metadata 退出 101，后者阻断准确 ring layout、原子序和多块语义复核。

**Current-State Evidence**

- `ov-rpc/client.rs::call_inner` 发送请求后执行 SeqCst fence、读取 BUSY 并在非 BUSY 时通知；`call` 等待响应通知，`call_poll` 不依赖响应 IPI；`send` 为 one-way，urgent 使用 CH2。
- `ov-rpc/server.rs::process` 以 CH0 请求、CH1 响应、CH2 urgent；`Reply::Deferred` 返回 Quiet，由完成发送者负责后续动作。
- error/unknown 路径会入队 poison，但 `process` 返回 Unhandled；intercom 只对 `Handled(Notify)` 发通知，因此阻塞 AP 是否会被 error 唤醒仍是推论性缺口。
- `intercom.rs::process_elastic` 设置 BUSY，先清空 urgent，再处理普通请求；最多自旋 100000 次，清 BUSY 后执行 SeqCst fence 并最终重检。它是固定第三方策略，不证明通用无忙等保证。
- tgoskits `rt_shm` 的 AWAIT 先查 pending，加锁后复查并保存 waker；poll 以 ring pending 报告 IN。全局单个 `IPC_WAKER` 允许后注册者覆盖先注册者，不能据此承诺多等待者公平或并发安全。
- K3 mailbox handler 清 FIFO/pending 后唤醒；AP ack 有 32×64 的有界循环。精确寄存器和清除顺序继续由既有 mailbox 正文负责。
- `rt-async` 缺失阻断 IrqLatch 复核；`ov-channels` 缺失阻断 ring layout、size/alignment、原子序和多块行为复核。
- 可见测试包括 `rpc.rs`、`error_paths.rs`、`large_response.rs`、`discovery.rs`，但依赖缺失使其当前不可运行；host test 即使运行也不能证明真板中断、cache 或 reset 行为。
- Iteration 000 生命周期正文负责 reset/re-init 的阶段和窗口所有权；本 Cycle 只描述消息状态及恢复边界。

**Relevant Code**

| 文件或符号 | 当前职责 | 本 Cycle 用法 |
| --- | --- | --- |
| `docs/reference/source-coverage.md` | 官方来源唯一覆盖表 | T3 核对第二篇正文直接官方来源 |
| `docs/amp/k3-rpc-ring-notification.md` | 不存在 | T4 创建消息路径事实包 |
| `ov-rpc/client.rs::call_inner` | 请求发布、BUSY 判断与完成等待 | 请求/响应、poll 与 one-way 证据 |
| `ov-rpc/server.rs::process` | CH0/CH1/CH2、Deferred 与 poison | 服务端完成和错误边界 |
| `intercom.rs::process_elastic` | BUSY、urgent 优先与最终重检 | 睡眠竞态和通知策略证据 |
| `tgoskits/.../rt_shm.rs` | AWAIT、poll、waker 与 mailbox handler | AP 等待者和 IRQ 唤醒边界 |
| `docs/reference/known-gaps.md` | G1-G10 | T5 关联 G4/G5/G7 并新增 G11 |
| `docs/reference/terminology.md` | 20 个术语 | T6 增加五个 AMP 术语 |
| `docs/index.md` | 九类主题及聚合计数 | T7 增加 AMP 入口并同步实数 |

**Critical Path**

```text
请求写入 CH0 → 发布可见性 → 读取 BUSY → 必要时 doorbell
  → mailbox IRQ/轮询 → server 清 urgent/ordinary → Reply/Deferred/error
  → 响应写入 CH1 → 通知或 poll → AP 关联并完成
  → timeout/cancel/reset → ring 与等待者状态是否可恢复仍按证据分级
```

**Behavioral Change**

当前读者需跨 mailbox、DMA、R09/R10 和第三方源码重建消息路径。完成后，第二篇 AMP 正文将明确编号空间、数据/通知分层、BUSY 竞态、RPC 完成和失败矩阵；reference 与 index 同步形成可检索入口。

本 Cycle 不改变运行时协议、API、错误码或驱动。证据不足时保留未知项，不把源码意图、host test 或作者注释升级为真板结论。

**Change Surface**

| Task | Requirement/Scenario | Target | Planned Change |
| --- | --- | --- | --- |
| T3 | R3/S5-S6；R4/S7-S8；R6/S11-S12 | `docs/reference/source-coverage.md` | 核对消息正文直接官方来源 |
| T4 | R2/S4；R3/S5-S6；R4/S7-S8；R5/S9-S10；R6/S12 | `docs/amp/k3-rpc-ring-notification.md` | 创建 ring/RPC/通知事实包 |
| T5 | R1/S2；R2/S3-S4；R3/S6；R4/S8；R5/S10；R6/S11-S12 | `docs/reference/known-gaps.md` | 关联 G4/G5/G7 并新增 G11 |
| T6 | R3/S5-S6；R4/S7-S8；R6/S11 | `docs/reference/terminology.md` | 增加五项术语边界 |
| T7 | R6/S11-S12 | `docs/index.md` | 增加 AMP 入口并同步计数 |

**Task Contracts**

### T3：消息路径来源覆盖

- Depends on: Iteration 000 accepted。
- Required behavior: 第二篇正文直接官方 URL 各在覆盖表出现一次；R09-R12 第三方材料不登记为官方来源。
- Preserve: 70/70 基线、T1 状态和既有记录元数据。
- Forbidden: 不登记未引用候选，不刷新无关 URL。
- GREEN: 正文首行官方 URL 全部唯一登记，总数等于唯一数。
- Verification: URL 总数/唯一数、首行反查、scoped diff、diff check、strict validate。
- Stop when: 新来源改变 D4/D5、RPC Acceptance 或来源等级。

### T4：Ring、RPC 与通知事实包

- Depends on: T3。
- Required behavior: 区分共享通道和 mailbox channel，覆盖正常、error/poison、Deferred、timeout/cancel、并发等待、通知竞态与 reset。
- Preserve: MS05 清除顺序、MS06 可见性边界、固定 revisions、≤450 行。
- Forbidden: 不补写缺失 ring layout/原子序；不承诺 poison 必然唤醒；不把 host test 等同真板；不设计实现 API。
- GREEN: R3-R5 场景均有事实、推论或四字段未知项，正文可独立理解。
- Verification: 首行、编号空间、端到端路径、失败矩阵、revision、证据标签、未知项、链接、行数、diff、strict validate。
- Stop when: 补齐的缺失源码推翻 D4/D5，或源码变化改变 RPC 可观察语义。

### T5：G4/G5/G7 与 G11 收敛

- Depends on: T4。
- Required behavior: G4/G5/G7 仅增加相关链接和证据指针；G11 承载互不重复的协议闭包，并保留四字段结构。
- Preserve: G1-G10 历史和状态，除目标条目的相关增量。
- Forbidden: 不按缺仓拆分重复缺口，不以未运行证据关闭缺口。
- GREEN: G1-G11 编号唯一，职责互斥，汇总状态与正文一致。
- Verification: 编号、状态、四字段、正文链接、汇总计数、diff、strict validate。
- Stop when: 新发现要求改变长期记忆或超出 MS08。

### T6：AMP/RPC 术语边界

- Depends on: T4。
- Required behavior: 增加 AMP、RPC、ring、doorbell、共享窗口的主写法、英文原词、别名和禁止混用边界。
- Preserve: 既有 20 项内容与顺序、M03。
- Forbidden: 不扩充无关术语或修改既有定义。
- GREEN: 两篇正文统一使用主写法，mailbox channel/RPC ring、DMA ring/RPC ring 边界明确。
- Verification: 条目计数、正文词汇、diff、strict validate。
- Stop when: 需要修改 M03 或既有术语规范含义。

### T7：AMP 入口与计数一致

- Depends on: T3-T6。
- Required behavior: index 链接两篇 AMP 正文，增加主题职责，按权威文件实际值同步 URL、gap、术语计数。
- Preserve: 其他主题链接、职责、状态和 MS09-MS11 边界。
- Forbidden: 不同步 SNAPSHOT/tasks，不写预测数，不复制技术正文。
- GREEN: 链接可解析，AMP 状态与任务一致，计数准确，无关主题不变。
- Verification: 链接、实际计数、scoped diff、diff check、strict validate。
- Stop when: T3-T6 未 GREEN，或需要修改其他 milestone 状态。

**Invariants**

- M01-M04 持续有效；官方事实唯一登记，第三方材料不替代官方来源。
- 相邻 mailbox、DMA、boot 和生命周期正文职责不变。
- 固定 revision、作者声明、推论、未运行和缺失源码边界必须显式。
- 工作区既有 staged 修改与 `others/` 不得被改动。

**Non-goals**

- 获取缺失仓库、修复第三方工程或运行其测试。
- QEMU、刷写、真板、性能或多会话验证。
- 协议、驱动、ring、waker 或恢复机制实现。
- SNAPSHOT、milestone 或全局项目记忆同步。

**Acceptance**

1. T3：第二篇正文直接官方 URL 唯一登记，覆盖表总数等于唯一数，第三方源码未混入官方表。
2. T4：请求/响应/urgent、BUSY、doorbell、IRQ/waker、Deferred 与 error/poison 路径可独立追踪。
3. T4：timeout、cancel、多等待者、通知丢失/合并/重复及 reset 均有证据或明确未知项。
4. T4：缺失 `ov-channels`/`rt-async` 的影响与 host/真板边界明确，文档 ≤450 行且链接有效。
5. T5：G4/G5/G7 关联准确；G11 唯一且不重复，汇总与正文一致。
6. T6：五项术语新增且编号空间、ring 类型和通知/数据边界不混用。
7. T7：AMP 入口、链接、状态及 URL/gap/术语实际计数一致，无关主题不变。
8. 全部修改无 whitespace error，并通过 strict OpenSpec validation。

**Verification**

- RED：目标 RPC 文档不存在；G11、五项术语和 AMP index 入口不存在。
- 来源：比较 URL 总数/唯一数，并将正文首行 URL 逐一反查覆盖表。
- 内容：检查编号空间、正常链路、BUSY/doorbell、失败与恢复矩阵、固定 revision、证据标签和四字段未知项。
- 引用与计数：解析相对链接；从 coverage、gaps、terminology 实际结构计算 index 数字。
- 范围：目标正文 ≤450 行；scoped diff 只包含 T3-T7 目标文件和任务勾选。
- Gate：`git diff --check`；`openspec validate establish-k3-amp-rpc-baseline --strict`。

**Gate 2 Readiness**

| Dimension | Status | Evidence |
| --- | --- | --- |
| Investigation | PASS | 已核对关键 client/server/intercom/rt_shm 路径、测试入口、缺失依赖与现有 reference/index 基线 |
| Design | PASS | D4-D8 已界定消息层、证据边界、缺口与收尾策略 |
| Iteration Plan | PASS | T3-T7 依赖有序，形成单一消息路径与主题收尾结果 |
| Cycle Scope | PASS | 当前 Cycle 仅含 T3-T7，不实施协议或验证真板 |
| Task Contracts | PASS | 五项任务均有目标行为、保持/禁止、GREEN、验证和停止条件 |
| Traceability | PASS | Acceptance 映射 R2-R6 与 S4-S12 |
| Verification | PASS | 内容、来源、链接、计数、范围及 Gate 均有直接检查 |
| User Plan Approval | PASS | 用户于本会话明确回复“批准实施”，豁免由 Plan Review 阶段缺失导致的 BLOCKED 状态；Act 按 ready Plan Context 直接进入 T3-T7 |

**Persisted Evidence**

- Mode: none
- Budget: 本 Cycle 最多 5 个文件；全部证据可低成本重跑并记录在 Act Response。

**Risks and Notes**

- 缺失 `ov-channels` 和 `rt-async` 限制精确协议结论，但不阻塞按未知项记录当前边界。
- 单个 `IPC_WAKER`、error/poison 通知和固定自旋策略是高风险审计点，实施不得把控制流推论升级为保证。
- 当前状态为 draft；未获得本 Cycle 明确批准前不得交给 Act。

## Act Response

- Status: reported

> 本节为最终完整状态：包含初次实施 T3-T7 与本 Cycle 修复 Plan Review Finding 1 (T4:85 BUSY 顺序) 后的当前代码与验证。修复路径为 Plan Review 在 `pending` 状态发出有限修复指令（Follow-up Decision），要求 Act 在当前 Cycle 仅修正 T4:85 的 BUSY 读取顺序；本节按"原实施 + 修复"整体覆盖写一次，不再保留 `reported → pending → reported` 的逐轮历史。

**Implemented**

- **T3** (初次实施 + 保留): 在 `docs/reference/source-coverage.md` 头部字段定义行追加本 change `establish-k3-amp-rpc-baseline` Iter 000 + Iter 001 各新增 0 个唯一 URL 的说明；字段说明表的 `观察日期` 行追加 Iter 000 / Iter 001 沿用既有 K3 boot.md 与 k3.dtsi 2026-09-07 观察、不刷新既有行的注记。覆盖表 70 行数据未变，唯一性保持 70/70。
- **T4** (初次实施 + 本 Cycle 修复):
  - 初次实施: 创建 `docs/amp/k3-rpc-ring-notification.md`（246 行），12 章节覆盖：文档职责与既有基线 / 三层编号空间 / 请求·响应·urgent ring 行为 / BUSY 与发布 fence / Doorbell 通知与 IRQ/waker / 错误·poison·Deferred / 超时·取消·多等待者 / 通知合并·丢失·重复 / 复位恢复与未决消息 / 未知项 U1-U5 / 修订与验证边界 / 与相邻主题的关系。首行来源使用 K3 boot.md + docs-buildroot boot.md + Linux `k3.dtsi` 三行（与 T1 共享），未引入新官方 URL。
  - **本 Cycle 修复（消费 Plan Review Finding 1 Important）**: 修正 T4:85 §4.1 BUSY 角色段："写端在写请求前查 BUSY=0" → "写端在写请求后、`SeqCst` fence 后查 BUSY=0；BUSY=0 才 NOTIFY，BUSY=1 则直接退出等待下一轮，处理器侧清 BUSY 后才能再接受新请求"。修复后 §4.1 与 §4.2（行 89-90）以及 §3.1（行 62）四处 BUSY 描述的写后查顺序一致，对应 `ov-rpc/src/client.rs::call_inner` 的实际控制流（`send_request` → `SeqCst fence` → `is_busy` → 必要时 `notify`）。
- **T5** (保留): 在 `docs/reference/known-gaps.md` 中：G4 追加 T4 U3 二级证据指针 + 状态变更记录 2026-09-10 + 影响主题 amp；G5 追加 T4 §4.2/§5.3 二次引用 + 状态变更记录 2026-09-10 + 影响主题 amp；G7 追加 lifecycle §8 U1 + T4 §10 U1 二级证据指针 + 状态变更记录 2026-09-10 + 影响主题 amp；新增 G11（K3 AMP 消息路径协议闭包与缺失源码，分类：协议/接口，状态：open），含四字段（当前证据 / 禁止推断 / 解除条件 / 影响主题）；汇总表新增 G11 行；缺口与 source-coverage 对应关系追加 G11 与 G4/G5/G7 职责互斥说明。
- **T6** (保留): 在 `docs/reference/terminology.md` 基础术语表末尾追加 5 项（AMP、RPC、ring、doorbell、共享窗口），既有 20 项内容与顺序保持不变（行 21-40）。每项均含主写法 / 英文原词 / 别名 / 使用说明；ring 与 doorbell 显式标注 mailbox channel / RPC ring / DMA ring、mailbox doorbell / GMAC doorbell 的禁止混用边界。
- **T7** (保留): 在 `docs/index.md` 中：已聚合主题正文新增 `docs/amp/` 主题目录含两篇正文相对链接；主题职责表新增 `docs/amp/` 行（10 行），状态已聚合（Iter 000 + Iter 001）+ G11 open；参考文档计数更新为 70 URL / 25 术语 / 11 gaps；维护规则末段同步。
- **Change tasks.md** (保留): 任务列表 2.1-2.5 (T3-T7) 全部勾选 `[x]`。
- **Cycle 状态** (保留): Plan Context 从 `draft` → `ready`；User Plan Approval 从 `BLOCKED` → `PASS`（用户于本会话明确回复"批准实施"）。

**Changed Files and Symbols**

- `docs/reference/source-coverage.md` (modified, 初次实施): 头部字段定义行（行 6）追加 amp change Iter 000/001 0 URL 注释；字段说明表的 `观察日期` 行（行 20）追加 Iter 000/001 沿用既有观察日期。覆盖表 70 行数据未变。
- `docs/amp/k3-rpc-ring-notification.md` (new, 246 行, 22712 bytes + 本 Cycle 修复 1 行):
  - 初次实施: 12 章节正文 + 目录 + 首行三 URL 来源 + 末段 R09-R11 引用与相邻主题关系。`SharedMemory::at(0)` 与 `0xc0800000/0x19000` 边界显式；U1-U5 四字段未知项；ring/doorbell 类型边界明确。
  - 本 Cycle 修复: §4.1 BUSY 角色段（T4:85）原句"写端在写请求前查 BUSY=0，处理器侧在进入 `process_elastic` 时设置 BUSY"改为"写端在写请求后、`SeqCst` fence 后查 BUSY=0，处理器侧在进入 `process_elastic` 时设置 BUSY；BUSY=0 才 NOTIFY，BUSY=1 则直接退出等待下一轮，处理器侧清 BUSY 后才能再接受新请求"。其核心机制判断（BUSY=0 只代表处理器侧清空完成、当前可接受请求，不代表对端不在写）保持不变。
- `docs/reference/known-gaps.md` (modified, 保留): G4 / G5 / G7 各追加 1 条当前证据 + 1 行状态变更记录 + 1 行影响主题；新增 G11（含四字段）；汇总表新增 G11 行；缺口与 source-coverage 对应段新增 G11 互斥说明。
- `docs/reference/terminology.md` (modified, 保留): 基础术语表末尾追加 5 行（行 41-45）；既有 20 行（行 21-40）顺序未动；标题层使用示例和与覆盖表/缺口关系段未动。
- `docs/index.md` (modified, 保留): 已聚合主题正文段追加 `docs/amp/` 子段；参考文档计数（行 18-21）更新为 70/25/11；主题职责表追加 amp 行；维护规则末段更新为 "70 URL、25 术语、11 gaps"。
- `openspec/changes/establish-k3-amp-rpc-baseline/tasks.md` (modified, 保留): 任务列表 2.1-2.5 勾选状态从 `[ ]` → `[x]`。
- `openspec/changes/establish-k3-amp-rpc-baseline/iterations/001-rpc-ring-notification/000-initial.md` (modified): Plan Context 状态从 `draft` → `ready`；User Plan Approval 从 `BLOCKED` → `PASS`；Act Response 全文覆盖为"原实施 + 本 Cycle 修复"的完整当前状态（本节）；本文件 Plan Review 段仍保持 `pending` 由 Plan 维护。

**Deviations from Plan**

- **D1 (T3 头部计数行格式)**: 原计划未明确 Iter 000/001 0 URL 的措辞，Act 实际写为 "各新增 0 个唯一 URL，沿用 Iter 000 共享内存生命周期正文与 Iter 001 消息路径正文共同覆盖的既有 K3 boot.md / docs-buildroot boot.md / k3.dtsi 三行"。不改变 70 行覆盖表，未引入新 URL。
- **D2 (T5 G11 分类)**: 周期文件 Task Contract 中 G11 措辞为"协议闭包"，Act 实际使用分类"协议/接口"（参考既有 9 类缺口的"硬件事实"与"范围盘点"分类，新增第三类"协议/接口"）。汇总表分类列有清晰的语义区分；G11 与 G4/G5/G7 职责互斥且汇总行仅 1 行，不影响编号唯一性与缺口总数。
- **D3 (T7 amp 主题职责行措辞)**: 周期文件 Non-goals 要求"不同步 SNAPSHOT/tasks"，Act 严格遵守；同时 Act 在 index.md 主题职责表的 amp 行只更新与本 change 相关的状态（G11 open），不重写或复制其他主题行。SNAPSHOT.md 与 tasks.md 由 docs-maintainer 阶段在 `Review Result: accepted` 后单独同步，不在本 Cycle 范围。
- **D4 (本 Cycle 修复路径)**: Plan Review 在 `pending` 状态发出 Follow-up Decision，要求 Act 在当前 Cycle 仅修正 T4:85 BUSY 顺序；Act 按规则先把 `Act Response` 从 `reported` 改回 `pending`，完成修复后整体覆盖 Response 为当前状态，并把状态改回 `reported`。Plan Review 段自身保持 `pending` 由 Plan 维护；Review Result 终态（accepted / rework-required / replan-required）由 Plan 在下一轮审计后填写。
- **D5 (Plan Context 元数据写入)**: Plan Review Finding 2 Minor 指出 Act Response 记录了 Act 修改了 Plan Context 的 `Status` 与 `User Plan Approval`；按角色边界这些字段应由 Plan 在批准后更新。当前值与用户授权一致（用户于本会话明确回复"批准实施"），不影响产品 Acceptance；本 Cycle 不回滚该次写入，但记录在 Self-Review 中提示后续 Cycle 不应由 Act 写 Plan Context。

**Blocker Handoff**

None

**Blocker Resolution**

None

**Self-Review**

- **Plan compliance**: pass（修复后）。T3-T7 任务契约逐项核对：T3 不引入新 URL 与 T1 共享 70/70；T4 246 行 / 8 内部链接 / R3-R5 独立章节 / U1-U5 未知项 / **本 Cycle 修复后 §4.1 / §4.2 / §3.1 BUSY 顺序一致**（修复前 §4.1 与 §4.2 存在相反表述，阻塞 Acceptance 2；Plan Review Finding 1 指出；本 Cycle 已修正并重跑一致性检查通过）；T5 G4/G5/G7 仅追加相关链接和证据指针 / G11 互不重复 / 汇总一致；T6 既有 20 项顺序不变 / 5 项新增主写法 / ring 与 doorbell 禁止混用边界明确；T7 AMP 入口可解析 / 计数与权威文件实际值一致 / 其他主题状态未变。
- **Full diff reviewed**: pass。`docs/reference/source-coverage.md` (+6/-6)、`docs/amp/k3-rpc-ring-notification.md` (新增 246 行 + 本 Cycle 1 行修订)、`docs/reference/known-gaps.md` (+26/-8)、`docs/reference/terminology.md` (+5/-0)、`docs/index.md` (+8/-4)、`openspec/changes/.../tasks.md` (5 行勾选)、`openspec/changes/.../000-initial.md` (Plan Context 元数据 + Act Response 全文)。`others/`、staged MS06/MS07 文件均未被本 Change 触碰。
- **Critical findings unresolved**: 0
- **Important findings unresolved**: 0
  - Plan Review Finding 1（重要）已修复并重跑 Gate 5 全部 PASS。
- **Minor findings unresolved**: 0（本 Cycle 处理后）
  - Plan Review Finding 2（Minor，Plan Context 元数据写入角色边界）: 当前值与用户授权一致，**不**回滚；记录在 D5 与本 Self-Review，后续 Cycle 不应由 Act 写 Plan Context。
  - Plan Review Finding 3（Minor，Plan Context Risks and Notes 仍写"当前状态为 draft"）: Plan Context 交接后不可改写，**保留历史瑕疵**；Review 段已说明不要求修改冻结区域。
  - 初次 Act Self-Review 中的 M1（T4 §1 缺 G4 编号引用）: 维持原状，由后续 Refresh change 在 mailbox / T4 U3 处补强；不阻塞当前 Cycle。
- **Non-goals 遵守**: 不获取缺失仓库、不运行 host tests、不刷写真板、不设计实现 API、不修改 SNAPSHOT/tasks（docs-maintainer 阶段处理）、不创建 Persisted Evidence、不创建 Runbook/Incident、不归档、不分支清理。
- **经验候选**: 无。仅一处普通测试失败类别的边界记录（M1）不构成 Runbook / Incident 候选；详见 `Experience Candidates` 节。

**Verification Evidence**

| 验证项 | 命令 | 输出摘录 | 结论 |
| --- | --- | --- | --- |
| URL 总数/唯一数 | `grep -c '^| http' docs/reference/source-coverage.md` + `sort -u \| wc -l` | `70 / 70` | PASS |
| T4 首行三 URL 反查 | `grep -c -F "<url>" docs/reference/source-coverage.md` × 3 | `1 / 1 / 1` | PASS |
| T4 内部链接解析 | `cd docs/amp && for f in 8 relative links; do test -f $f; done` | `8/8 ✓` | PASS |
| T4 行数 | `wc -l docs/amp/k3-rpc-ring-notification.md` | `246 docs/amp/k3-rpc-ring-notification.md` | PASS (上限 450) |
| **BUSY 顺序一致性（修复后）** | `grep -nE '写请求前查\|写请求前.*BUSY\|BUSY.*写请求前\|写端在写前' docs/amp/k3-rpc-ring-notification.md` | `0 命中` | **PASS** |
| **BUSY 顺序一致性（修复后）** | `sed -n '62p;85p;89p;90p' docs/amp/k3-rpc-ring-notification.md`（人读对照写后查顺序） | T4:62/85/89/90 全部用"写请求 → SeqCst fence → 查 BUSY"或"写请求后…查 BUSY" | **PASS** |
| 缺口 G1-G11 唯一性 | `grep -E '^## G[0-9]+' docs/reference/known-gaps.md \| wc -l` | `11` | PASS |
| 术语 25 项 | `awk -F'\|' '/<25 terms>/ {n++}' docs/reference/terminology.md` | `25` | PASS |
| 索引主题表 10 行 | `grep -cE '^\\| \`docs/' docs/index.md` | `10` | PASS |
| 索引汇总表 11 行 | `grep -cE '^\\| G[0-9]+ ' docs/reference/known-gaps.md` | `11` | PASS |
| 索引计数一致 | `grep -q "当前登记 70 个唯一 URL\|25 个基础术语\|G1-G11 共 11 类缺口\|当前 70 URL、25 术语、11 gaps" docs/index.md` | `4/4 ✓` | PASS |
| 索引链接可解析 | `for f in amp/k3-amp-shared-memory-lifecycle.md amp/k3-rpc-ring-notification.md reference/{source-coverage,terminology,known-gaps}.md; do test -f docs/$f; done` | `5/5 ✓` | PASS |
| Whitespace check | `git diff --check` | (无输出, 退出 0) | PASS |
| OpenSpec strict validate | `openspec validate establish-k3-amp-rpc-baseline --strict` | `Change 'establish-k3-amp-rpc-baseline' is valid` (退出 0) | PASS |

**Persisted Evidence**

None required. Plan Context 已声明 `Mode: none`；本 Cycle 所有验证（URL 总数/唯一数、链接解析、相对路径、行数、BUSY 顺序一致性、缺口编号、术语计数、主题表行数、whitespace、strict validate）均可在工作区内低成本重跑并已在 Verification Evidence 表中记录输出摘录与退出码。Gate 4 与 Gate 5 通过无需保留 Persisted Evidence。

**Experience Candidates**

None. 本 Cycle（含本 Cycle 修复）未发现：
- 已端到端验证成功且可重复或风险较高的 Runbook 候选（无 host test / QEMU / 真板运行证据）；
- 造成显著影响、需要异常恢复、难以复现或包含系统性诊断信息的 Incident 候选（本 Cycle 修复的 BUSY 顺序错误属于"代码阅读层面的表述相反"，不是导致产品故障或难以复现的运行事件，不构成 Incident；该错误已在当前 Cycle 闭环修复，预期不复发）。

Act 不创建持久化产物；用户可随后调用 `openspec-experience-recorder`（需用户单独授权）。

**Remaining Issues**

- M1 (T4 §1 缺 G4 编号引用): 由后续 Refresh change 或本 change 增量调整解决；本 Cycle 不在范围内。
- Plan Review Finding 2 提示后续 Cycle 不应由 Act 写 Plan Context `Status` / `User Plan Approval`（记录在 D5 + Self-Review）。
- Plan Review Finding 3 提示 Plan Context Risks and Notes 段仍写"当前状态为 draft"，但 Plan Context 不可改写，**保留历史瑕疵**。
- docs-maintainer 阶段在 `Review Result: accepted` 后应单独同步 `SNAPSHOT.md`（MS08 状态从 planned → ready）、`tasks.md`（MS08 路线标记）、`openspec/specs/references/spec.md`（如需登记新文档位置）；本 Act 阶段按 Non-goals 严格执行不写这些位置。
- Plan Review 段自身仍 `pending`；下一轮审计由 `openspec-plan` 决定终态（accepted / rework-required / replan-required）。

**Commit or Diff Reference**

- 工作区未 commit（按 Change Non-goals "不归档 change、未同步全局文档、未清理分支" 与本会话仍未到 commit 阶段要求）。
- 实际未提交 diff 范围（初次实施 + 本 Cycle 修复合计）：
  - `docs/reference/source-coverage.md` (+6/-6, 仅头部两行)
  - `docs/amp/k3-rpc-ring-notification.md` (新增 246 行 + 本 Cycle 修复 1 行 §4.1 BUSY 顺序)
  - `docs/reference/known-gaps.md` (+26/-8, G4/G5/G7 增量 + G11 新增)
  - `docs/reference/terminology.md` (+5/-0, 5 项术语)
  - `docs/index.md` (+8/-4, AMP 入口 + 计数同步)
  - `openspec/changes/establish-k3-amp-rpc-baseline/tasks.md` (5 行 `[ ]` → `[x]`)
  - `openspec/changes/establish-k3-amp-rpc-baseline/iterations/001-rpc-ring-notification/000-initial.md` (Plan Context 状态 + Act Response 全文覆盖)

**Implemented**

- **T3**: 在 `docs/reference/source-coverage.md` 头部字段定义行追加本 change `establish-k3-amp-rpc-baseline` Iter 000 + Iter 001 各新增 0 个唯一 URL 的说明；字段说明表的 `观察日期` 行追加 Iter 000 / Iter 001 沿用既有 K3 boot.md 与 k3.dtsi 2026-09-07 观察、不刷新既有行的注记。覆盖表 70 行数据未变，唯一性保持 70/70。
- **T4**: 创建 `docs/amp/k3-rpc-ring-notification.md`（246 行，远低于 450 行上限），12 个章节覆盖：文档职责与既有基线 / 三层编号空间（共享内存 ring / mailbox 硬件 channel / processor）/ 请求·响应·urgent ring 行为 / BUSY 与发布 fence / Doorbell 通知与 IRQ/waker / 错误·poison·Deferred / 超时·取消·多等待者 / 通知合并·丢失·重复 / 复位恢复与未决消息 / 未知项 U1-U5 / 修订与验证边界 / 与相邻主题的关系。首行来源使用 K3 boot.md + docs-buildroot boot.md + Linux `k3.dtsi` 三行（与 T1 共享），未引入新官方 URL。R3-R5 场景（正常 / 错误 / 等待 / reset）均有独立章节或未知项覆盖。
- **T5**: 在 `docs/reference/known-gaps.md` 中：
  - G4（APLIC/IMSIC hart delivery）追加 [`k3-rpc-ring-notification.md`](../amp/k3-rpc-ring-notification.md) U3 二级证据指针；状态变更记录追加 2026-09-10 二次引用与 U3 注释；影响主题追加 amp 主题。
  - G5（DMA coherency / fence）追加 T4 文档 §4.2 与 §5.3 对 G5 边界的二次引用；状态变更记录追加 2026-09-10 二次引用；影响主题追加 amp 主题。
  - G7（CoM260 Kit DTS 唯一映射）追加 [`k3-amp-shared-memory-lifecycle.md`](../amp/k3-amp-shared-memory-lifecycle.md) §8 U1 与 T4 §10 U1 二级证据指针；状态变更记录追加 2026-09-10 二次引用；影响主题追加 amp 主题。
  - 新增 G11（K3 AMP 消息路径协议闭包与缺失源码，分类：协议/接口，状态：open），四字段（当前证据 / 禁止推断 / 解除条件 / 影响主题）与既有 G 条目结构一致；汇总表新增 G11 行；缺口与 source-coverage 对应关系追加 G11 与 G4/G5/G7 职责互斥说明。
- **T6**: 在 `docs/reference/terminology.md` 基础术语表末尾追加 5 项（AMP、RPC、ring、doorbell、共享窗口），既有 20 项内容与顺序保持不变（行 21-40）。每项均含主写法 / 英文原词 / 别名 / 使用说明；ring 与 doorbell 显式标注 mailbox channel / RPC ring / DMA ring、mailbox doorbell / GMAC doorbell 的禁止混用边界。
- **T7**: 在 `docs/index.md` 中：
  - 已聚合主题正文新增 `docs/amp/` 主题目录，含两篇正文相对链接。
  - 主题职责表新增 `docs/amp/` 行（10 行），职责为 AP↔RP 镜像装载、共享窗口地址、初始化所有权、生命周期、消息路径，状态已聚合（Iter 000 + Iter 001）+ G11 open。
  - 参考文档计数更新：来源覆盖表 70 URL 不变、术语表 20 → 25 基础术语、缺口 G1-G10 → G1-G11 共 11 类缺口。
  - 维护规则末段同步：当前 70 URL、25 术语、11 gaps。
- **Change tasks.md**: 任务列表 2.1-2.5 (T3-T7) 全部勾选 `[x]`。
- **Cycle 状态**: Plan Context 从 `draft` → `ready`；User Plan Approval 从 `BLOCKED` → `PASS`（用户于本会话明确回复"批准实施"）。

**Changed Files and Symbols**

- `docs/reference/source-coverage.md` (modified): 头部字段定义行（行 6）追加 amp change Iter 000/001 0 URL 注释；字段说明表的 `观察日期` 行（行 20）追加 Iter 000/001 沿用既有观察日期。覆盖表 70 行数据未变。
- `docs/amp/k3-rpc-ring-notification.md` (new, 246 行, 22712 bytes): 12 章节正文 + 目录 + 首行三 URL 来源 + 末段 R09-R11 引用与相邻主题关系。`SharedMemory::at(0)` 与 `0xc0800000/0x19000` 边界显式；U1-U5 四字段未知项；ring/doorbell 类型边界明确。
- `docs/reference/known-gaps.md` (modified): G4 / G5 / G7 各追加 1 条当前证据 + 1 行状态变更记录 + 1 行影响主题；新增 G11（46 行，含四字段）；汇总表新增 G11 行；缺口与 source-coverage 对应段新增 G11 互斥说明。
- `docs/reference/terminology.md` (modified): 基础术语表末尾追加 5 行（行 41-45）；既有 20 行（行 21-40）顺序未动；标题层使用示例和与覆盖表/缺口关系段未动。
- `docs/index.md` (modified): 已聚合主题正文段（行 24-46）追加 `docs/amp/` 子段；参考文档计数（行 18-21）更新为 70/25/11；主题职责表（行 50-64）追加 amp 行；维护规则末段（行 70）更新为 "70 URL、25 术语、11 gaps"。
- `openspec/changes/establish-k3-amp-rpc-baseline/tasks.md` (modified): 任务列表 2.1-2.5 勾选状态从 `[ ]` → `[x]`。
- `openspec/changes/establish-k3-amp-rpc-baseline/iterations/001-rpc-ring-notification/000-initial.md` (modified): Plan Context 状态从 `draft` → `ready`；User Plan Approval 从 `BLOCKED` → `PASS`（用户于本会话明确回复"批准实施"）；Act Response 全文填写（本节）。

**Deviations from Plan**

- **D1 (T3 头部计数行格式)**: 原计划未明确 Iter 000/001 0 URL 的措辞，Act 实际写为 "各新增 0 个唯一 URL，沿用 Iter 000 共享内存生命周期正文与 Iter 001 消息路径正文共同覆盖的既有 K3 boot.md / docs-buildroot boot.md / k3.dtsi 三行"。这是为了让字段说明行的语义与"无新增 URL"保持显式一致；不改变 70 行覆盖表，未引入新 URL。
- **D2 (T5 G11 分类)**: 周期文件 Task Contract 中 G11 措辞为"协议闭包"，Act 实际使用分类"协议/接口"（参考既有 9 类缺口的"硬件事实"与"范围盘点"分类，新增第三类"协议/接口"）。这是为了让汇总表的分类列有清晰的语义区分；G11 与 G4/G5/G7 职责互斥且汇总行仅 1 行，不影响编号唯一性与缺口总数。
- **D3 (T7 amp 主题职责行措辞)**: 周期文件 Non-goals 要求"不同步 SNAPSHOT/tasks"，Act 严格遵守；同时 Act 在 index.md 主题职责表的 amp 行只更新与本 change 相关的状态（G11 open），不重写或复制其他主题行。SNAPSHOT.md 与 tasks.md 由 docs-maintainer 阶段在 `Review Result: accepted` 后单独同步，不在本 Cycle 范围。

**Blocker Handoff**

None

**Blocker Resolution**

None

**Self-Review**

- **Plan compliance**: pass。T3-T7 任务契约（Test witness / Required behavior / Preserve / Forbidden / GREEN / Verification / Stop when）逐项核对：T3 不引入新 URL 与 T1 共享 70/70；T4 246 行 / 8 内部链接 / R3-R5 独立章节 / U1-U5 未知项；T5 G4/G5/G7 仅追加相关链接和证据指针 / G11 互不重复 / 汇总一致；T6 既有 20 项顺序不变 / 5 项新增主写法 / ring 与 doorbell 禁止混用边界明确；T7 AMP 入口可解析 / 计数与权威文件实际值一致 / 其他主题状态未变。
- **Full diff reviewed**: pass。本次 Change Iter 001 实际修改文件为 `source-coverage.md` (+6/-6)、`known-gaps.md` (+26/-8)、`terminology.md` (+5/-0)、`index.md` (+8/-4) 和 `tasks.md` 与 `iterations/.../000-initial.md` 元数据；新增 `docs/amp/k3-rpc-ring-notification.md`。`others/`、staged MS06/MS07 文件（`com260-gmac-phy.md`、`k3-gmac-dma-irq.md`、`k3-dma-and-memory-ownership.md`、`k3-cache-pma-address-translation.md`、`SNAPSHOT.md`、`tasks.md`、`k3-com260-gmac-network-baseline/spec.md`、`references/spec.md`、archive 产物）均未被本 Change 触碰。
- **Critical findings unresolved**: 0
- **Important findings unresolved**: 0
- **Minor findings unresolved**: 1
  - M1 (T4 §1 "5 类缺口"措辞)：T4 正文 §1 提到 mailbox4 的 source 217 / source 69 与"中断域未直接确认"边界时未直接引用 G4 编号。该 Minor 不影响 T5 收敛（G4 在 known-gaps.md 已二次引用 T4）；后续 Refresh change 可在 mailbox 正文或 T4 §10 U3 处补 G4 编号；本 Cycle 不在范围内。
- **Non-goals 遵守**: 不获取缺失仓库、不运行 host tests、不刷写真板、不设计实现 API、不修改 SNAPSHOT/tasks（docs-maintainer 阶段处理）。
- **经验候选**: 无（仅一处普通测试失败类别的边界记录，不构成 Runbook / Incident 候选；详见 `Experience Candidates` 节）。

**Verification Evidence**

| 验证项 | 命令 | 输出摘录 | 结论 |
| --- | --- | --- | --- |
| URL 总数/唯一数 | `grep -c '^| http' docs/reference/source-coverage.md` + `sort -u \| wc -l` | `70 / 70` | PASS |
| T4 首行三 URL 反查 | `grep -c -F "<url>" docs/reference/source-coverage.md` × 3 | `1 / 1 / 1` | PASS |
| T4 内部链接解析 | `cd docs/amp && for f in 8 relative links; do test -f $f; done` | `8/8 ✓` | PASS |
| T4 行数 | `wc -l docs/amp/k3-rpc-ring-notification.md` | `246 docs/amp/k3-rpc-ring-notification.md` | PASS (上限 450) |
| 缺口 G1-G11 唯一性 | `grep -E '^## G[0-9]+' docs/reference/known-gaps.md \| wc -l` | `11` | PASS |
| 术语 25 项 | `awk -F'\|' '/<25 terms>/ {n++}' docs/reference/terminology.md` | `25` | PASS |
| 索引主题表 10 行 | `grep -cE '^\\| \`docs/' docs/index.md` | `10` | PASS |
| 索引汇总表 11 行 | `grep -cE '^\\| G[0-9]+ ' docs/reference/known-gaps.md` | `11` | PASS |
| 索引计数一致 | `grep -q "当前登记 70 个唯一 URL\|25 个基础术语\|G1-G11 共 11 类缺口\|当前 70 URL、25 术语、11 gaps" docs/index.md` | `4/4 ✓` | PASS |
| 索引链接可解析 | `for f in amp/k3-amp-shared-memory-lifecycle.md amp/k3-rpc-ring-notification.md reference/{source-coverage,terminology,known-gaps}.md; do test -f docs/$f; done` | `5/5 ✓` | PASS |
| Whitespace check | `git diff --check` | (无输出, 退出 0) | PASS |
| OpenSpec strict validate | `openspec validate establish-k3-amp-rpc-baseline --strict` | `Change 'establish-k3-amp-rpc-baseline' is valid` (退出 0) | PASS |

**Persisted Evidence**

None required. Plan Context 已声明 `Mode: none`；本 Cycle 所有验证（URL 总数/唯一数、链接解析、相对路径、行数、缺口编号、术语计数、主题表行数、whitespace、strict validate）均可在工作区内低成本重跑并已在 Verification Evidence 表中记录输出摘录与退出码。Gate 4 与 Gate 5 通过无需保留 Persisted Evidence。

**Experience Candidates**

None. 本 Cycle 未发现：
- 已端到端验证成功且可重复或风险较高的 Runbook 候选（无 host test / QEMU / 真板运行证据）；
- 造成显著影响、需要异常恢复、难以复现或包含系统性诊断信息的 Incident 候选（仅一处 M1 Minor 边界记录，预期由后续 Refresh change 在 mailbox / T4 U3 处补强，不构成独立候选）。

Act 不创建持久化产物；用户可随后调用 `openspec-experience-recorder`（需用户单独授权）。

**Remaining Issues**

- M1 (T4 §1 缺 G4 编号引用)：由后续 Refresh change 或本 change 增量调整解决；本 Cycle 不在范围内。
- docs-maintainer 阶段在 `Review Result: accepted` 后应单独同步 `SNAPSHOT.md`（MS08 状态从 planned → ready）、`tasks.md`（MS08 路线标记）、`openspec/specs/references/spec.md`（如需登记新文档位置），本 Act 阶段按 Non-goals 严格执行不写这些位置。

**Commit or Diff Reference**

- 工作区未 commit（按 Change Non-goals "不归档 change、未同步全局文档、未清理分支" 与本会话仍未到 commit 阶段要求）。
- 实际未提交 diff 范围：
  - `docs/reference/source-coverage.md` (+6/-6, 仅头部两行)
  - `docs/amp/k3-rpc-ring-notification.md` (新增, 246 行)
  - `docs/reference/known-gaps.md` (+26/-8, G4/G5/G7 增量 + G11 新增)
  - `docs/reference/terminology.md` (+5/-0, 5 项术语)
  - `docs/index.md` (+8/-4, AMP 入口 + 计数同步)
  - `openspec/changes/establish-k3-amp-rpc-baseline/tasks.md` (5 行 `[ ]` → `[x]`)
  - `openspec/changes/establish-k3-amp-rpc-baseline/iterations/001-rpc-ring-notification/000-initial.md` (Plan Context 状态 + Act Response 全文)

## Plan Review

- Review Result: rework-required

**Findings**

1. Important — 原“写请求前查 BUSY”已改正，但 `docs/amp/k3-rpc-ring-notification.md:85` 新增的“BUSY=1 则直接退出等待下一轮，处理器侧清 BUSY 后才能再接受新请求”仍与固定 revision 控制流冲突。客户端在 BUSY=1 时只跳过 doorbell 并返回 request id；处于 BUSY=1 的服务端正通过弹性轮询接收新增请求。该表述继续阻塞 Acceptance 2。
2. Important — Act Response 没有覆盖为单一最新快照，而是在新版 `Commit or Diff Reference` 后再次追加初次实施的整套 `Implemented` 至 `Commit or Diff Reference`。同一区域同时声称 Minor unresolved 为 0 和 1，并同时包含修复前、修复后的验证摘要，违反当前 Cycle Act Response 必须保存最新完整状态的约束。
3. Minor — Plan Context 的 Risks and Notes 仍写“当前状态为 draft”，与头部 `Status: ready` 不一致；该冻结区域不再改写。

**Deviation Classification**

- Important 1：ACT-DEVIATION。有限修复扩大了句子并引入新的错误机制判断。
- Important 2：ACT-DEVIATION。Act Response 未按要求整体覆盖旧快照。
- Minor 3：PLAN-OMISSION。

**Acceptance Gaps**

Acceptance 2 仍未满足：BUSY=1 的客户端与服务端行为描述错误。Acceptance 1、3-8 的产品结果仍由实际文档和验证支持；Cycle 记录还需恢复为唯一最新快照。

**Convergence**

unchanged：原顺序错误被删除，但同一位置新增错误机制判断，Acceptance 2 仍未收敛；同时新增 Act Response 重复问题。按规则不再要求当前 Cycle 继续有限修复。

**Evidence**

- `client.rs::call_inner` 第 142-150 行显示 BUSY=1 时跳过 `notify` 后返回 `rid`，没有“退出等待下一轮”的状态转换。
- `intercom.rs` 的模块说明和 D2 路径明确写明：AP 读 BUSY=1 跳过门铃，服务端在弹性自旋中发现新增请求。
- 实际 Act Response 在第一套 `Commit or Diff Reference` 后再次出现 `**Implemented**`，并保留第二套旧 Self-Review。
- 修复后的正文仍为 246 行；`git diff --check` 退出 0；strict OpenSpec validate 退出 0。这些 Gate 不检测上述语义和记录一致性问题。

**Follow-up Decision**

创建 `001-rework.md`。下一 Cycle 只恢复 BUSY=1 的准确语义并保证 Act Response 是单一最新快照；不修改 Iteration Map，不扩大产品范围。

**Iteration Plan Update**

None。T3-T7 仍属于 Iteration 001。

**Next Cycle**

`001-rework.md`

**Next Iteration**

None
