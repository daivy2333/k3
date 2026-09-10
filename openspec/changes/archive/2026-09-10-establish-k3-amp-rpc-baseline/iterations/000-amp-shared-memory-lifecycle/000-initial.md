# Iteration 000 / Cycle 000: AP/RP 生命周期与共享窗口基线

## Plan Context

- Status: ready
- Iteration: 000-amp-shared-memory-lifecycle
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: None

**Iteration Scope**

- Change tasks: T1-T2
- Depends on: MS03、MS05、MS06 已完成
- Stable baseline: AP/RP 镜像/握手、共享窗口地址、初始化所有权和 reset/re-init 状态可独立引用；后续消息路径无需重新判断启动阶段或地址边界
- Verification boundary: 生命周期正文的官方来源唯一登记；阶段、所有者、地址、内存属性边界和数据损失未知项完整
- Diagnostic boundary: 镜像/握手、启动链破坏者、地址 alias、PMA/PBMT、初始化竞争、magic、reset/re-init
- Deferred tasks: T3-T7

**Cycle Scope**

- Trigger: initial
- Acceptance gaps: None
- Repair items: None
- Inherited scope: proposal R1、R2、R5 及 R6 的来源边界；M01-M04；D04-D07；MS03/MS05/MS06 已建立的事实职责
- Excluded scope: ring/RPC 消息语义、G11/术语/index 收尾、第三方补仓或构建、真板验证、产品实现、全局状态同步

**Objective**

创建可独立阅读的 K3 AP/RP 生命周期与共享窗口文档，并使其直接官方来源在覆盖表中唯一登记。读者应能判断各启动和恢复阶段的状态所有者、地址空间、允许操作、证据等级及未读数据风险。

**Background**

用户已于本会话以“同意”批准 Gate 1 的需求、BDD 和范围。现有启动、mailbox、DMA/PMA 正文分别保存局部事实，R09/R10 保存固定 revision 第三方端到端调查，但没有产品文档串联启动链破坏者、初始化竞争和 reset/re-init。

**Current Baseline**

- 项目 revision: `b484827`，分支 `main`。
- 工作区已有 MS06/MS07 staged 修改和未跟踪 `others/`；它们不是本 change 的修改目标。
- `docs/reference/source-coverage.md`: 70 个 URL 行、70 个唯一 URL；已有 boot、K3/CoM260 DTS、`k3-rdomain.dtsi`、RISC-V PMA/PBMT/FENCE 来源，尚无 `amp` 主题职责。
- `docs/reference/known-gaps.md`: G1-G10；G4/G5 为 `partial`，G7 为 `open`。
- `docs/index.md`: 九类主题，70 URL、10 gaps、20 个术语；没有 `docs/amp/`。
- `docs/amp/k3-amp-shared-memory-lifecycle.md`: 不存在，`test ! -e` 退出 0。
- Rt-Async-AMP、tgoskits、OpenSBI HEAD 分别为 `ccb1ff0b487e4f49ea570c41f330741eecece935`、`19219411d5dc1515496f910d04c93da12ee95be4`、`7a2df083ed06373c506e2e6f4e09bbd168202f2d`，对应工作树无修改。
- `rt-async/modules/platform/Cargo.toml` 和 `modules/ov-channels/Cargo.toml` 缺失；根 workspace offline metadata 因前者不存在退出 101。该失败证明材料缺失，不是产品测试失败。

**Current-State Evidence**

- `docs/boot/com260-boot-chain.md` 已负责 Boot ROM → FSBL/SPL → ESOS → OpenSBI → U-Boot → payload/OS；本 Cycle 只引用阶段，不复制启动介质和刷写正文。
- `docs/interrupts/com260-mailbox-notification.md` 已负责 mailbox4 USER/channel、FIFO/pending/IRQ 和等待边界；本 Cycle 只引用 doorbell 在生命周期中的上线与清理位置。
- `docs/dma/k3-dma-and-memory-ownership.md` 已区分 AP↔RP 共享内存和设备 DMA，明确 mailbox 不替代数据可见性。
- `docs/dma/k3-cache-pma-address-translation.md` 已负责 PMA/PBMT/FENCE/cache/IOMMU 机制；本 Cycle 只描述共享窗口在生命周期中的属性依赖。
- `others/Rt-Async-AMP/apps/rt-async-k3/src/intercom.rs::wait_ready` 在 3 秒读门控后轮询 magic；有效时以 Release 发布 `SHM_BASE`，超过 `fallback_ms` 后调用本地 `init`。
- `intercom.rs::init` 先调用 `SharedMemory::<3>::at(base).init()`，再发布 `SHM_BASE`，避免 ISR 在窗口初始化前读取 ring。
- `others/Rt-Async-AMP/apps/rt-async-k3/src/watchdog.rs::magic_watchdog` 在门控后每秒检查 magic，最多触发 8 次 re-init；源码说明 re-init 会清除未读消息。时间和次数属于第三方策略。
- `others/Rt-Async-AMP/modules/ov-shm/src/shm.rs::ShmDriver` 从 RP DTS 登记 base/size；R09/R10 已确认 RP 配置地址 0、AP reserved-memory 配置 `0xc0800000..0xc0819000`、窗口大小 `0x19000`。两者为同一 SRAM 的说法尚无本项目真板证据。
- tgoskits `rt_shm.rs::RtShmDevice::new` 从 reserved-memory ioremap 共享窗，valid 时保留、invalid 时初始化；其注释关于 U-Boot 清窗和迟到 cache 写回属于第三方作者声明。
- OpenSBI `spacemit_k3.c::k3_pma_set_amp_window_io` 查找覆盖 `[0xc0800000, 0xc0880000)` 的 PMA entry，clean 后写属性并 `sfence.vma`；找不到 entry 时继续启动。它修改的 entry 可能大于应用窗口。
- `rt-async` 缺失阻断 RP 启动汇编、linker、driver boot 和 IrqLatch 复核；`ov-channels` 缺失阻断准确 ring layout、size/alignment、原子序和零地址 API 复核。

**Relevant Code**

| 文件或符号 | 当前职责 | 本 Cycle 用法 |
| --- | --- | --- |
| `docs/reference/source-coverage.md` | 官方来源唯一覆盖表 | T1 精确扩充 `amp` 职责或登记实际新增 URL |
| `docs/amp/k3-amp-shared-memory-lifecycle.md` | 不存在 | T2 创建生命周期事实包 |
| `intercom.rs::init/wait_ready` | RP 初始化、等待与基址发布 | 固定 revision 第三方状态转换证据 |
| `watchdog.rs::magic_watchdog` | magic 周期检查与 re-init | 数据损失和恢复上限证据 |
| `ov-shm/src/shm.rs::ShmDriver` | RP 共享窗 base/size 与 fence | 地址、probe 和可见性边界 |
| `tgoskits/.../rt_shm.rs::RtShmDevice::new` | AP reserved-memory 映射和初始化 | AP 初始化者与 valid 保留行为 |
| `spacemit_k3.c::k3_pma_set_amp_window_io` | OpenSBI PMA entry 查找与属性修改 | AP 内存属性前置和失败边界 |

**Critical Path**

```text
AP/RP 镜像配置与握手
  → SPL/U-Boot/bootm 可能写共享 SRAM
  → AP reserved-memory/ioremap 与 OpenSBI PMA 前置
  → AP probe 保留 valid 或初始化 invalid 窗口
  → RP wait_ready 观察 magic 并发布 SHM_BASE
  → 双端在线使用
  → magic 无效或对端 reset
  → watchdog/fallback re-init
  → 未读 ring 状态可能清除，重新建立服务条件仍待定义
```

数据从 AP 地址 `0xc0800000` 与 RP 本地地址 0 进入同一“候选共享窗口”模型；alias 关系、每 hart PMA 实效和真板双向可见性保持未知。状态所有权以“谁最后可能破坏、谁初始化、谁发布可用”为主线，不按固定延迟推断安全阶段。

**Implementation Guidance**

1. T1 先从计划正文的首行来源候选反向核对覆盖表；复用既有行并只扩充主题职责。候选未被正文引用时不登记。
2. T2 使用文档模板创建 `docs/amp/` 和正文；首段先固定范围、来源层和与 MS03/MS05/MS06 的责任边界。
3. 用地址/镜像表和生命周期状态表承载主结论，再分别描述初始化竞争、内存属性前置、reset/re-init 和未知项。
4. 第三方源码行为携带固定 revision；注释、历史板上数据和综合链路分别标为第三方声明或推论。
5. 每个未知项写当前证据、禁止推断、解除条件和影响主题。不得提前新增 G11；G11 属 Iteration 001 T5。

**Behavioral Change**

当前读者必须跨 boot、interrupts、dma 和 analysis 重建 AP/RP 生命周期，且无法从 `docs/` 获得单一状态模型。完成后，生命周期正文直接呈现阶段、地址、所有者、内存属性依赖、失效和重建边界；现有相邻文档继续保有各自技术细节。

本 Cycle 不改变运行时接口、协议或错误码。错误语义只作为文档行为：证据不足时输出明确未知项，不静默选择初始化者、默认 DTS、alias 或无损恢复结论。

**Change Surface**

| Task | Requirement/Scenario | File/Symbol | Current Responsibility | Planned Change |
| --- | --- | --- | --- | --- |
| T1 | R1/S1-S2；R2/S3-S4；R6/S11-S12 | `docs/reference/source-coverage.md` | 70 个官方 URL 的唯一覆盖 | 为生命周期正文核对并扩充 `amp` 主题来源 |
| T2 | R1/S1-S2；R2/S3-S4；R5/S9-S10；R6/S12 | `docs/amp/k3-amp-shared-memory-lifecycle.md` | 不存在 | 创建 AP/RP 生命周期与共享窗口事实包 |

**Task Contracts**

### T1：生命周期来源覆盖

- Requirement/Scenario: R1/S1-S2；R2/S3-S4；R6/S11-S12。
- Depends on: None。
- Targets: `docs/reference/source-coverage.md`。
- Current behavior: 70 行/70 唯一 URL；相关 boot、DTS 和标准来源已登记，但没有 `amp` 主题职责。
- Required behavior: 生命周期正文的每个直接官方 URL 在覆盖表恰有一行；既有行扩充主题职责，新行仅用于已打开且正文实际引用的官方来源。
- Required changes: 核对 R01、boot/image、K3/CoM260 DTS 与正文首行候选；记录观察对象、日期和证据边界，不预设总数增加。
- Preserve: 既有 70 行身份、状态和观察日期；M01、D04-D07；官网 SPA 壳的 `partially-observed`。
- Forbidden: 不批量刷新；不把 R09–R12 或第三方源码登记为官方来源；不修改 `others/`。
- Test witness: 变更前 URL 行数/唯一数 `70/70`，现有主题字段没有 `amp`。
- GREEN condition: 正文首行官方 URL 均唯一登记，覆盖表总数等于唯一数，新增或修改行元数据完整。
- Verification: URL 总数/唯一数、正文首行反向核对、scoped diff、`git diff --check`、strict OpenSpec validate。
- Stop when: 需要 M01/R08 之外的新权威来源，或来源变化改变 D1–D8、目标板范围或 Acceptance。

### T2：AP/RP 生命周期与共享窗口事实包

- Requirement/Scenario: R1/S1-S2；R2/S3-S4；R5/S9-S10；R6/S12。
- Depends on: T1。
- Targets: `docs/amp/k3-amp-shared-memory-lifecycle.md`。
- Current behavior: `docs/amp/` 和目标文件不存在；事实散落于 MS03/MS05/MS06 与 R09/R10。
- Required behavior: 可沿镜像/握手 → 启动链破坏者 → AP/RP 地址 → 初始化/发布 → 在线 → magic 失效/reset → re-init 查询所有者、状态、操作和数据损失。
- Required changes: 建立 AP/RP 地址和镜像表、窗口布局边界、生命周期状态表、初始化竞争、PMA/PBMT/cache 边界、reset/re-init 转换和四字段未知项；相邻主题用链接引用。
- Preserve: M01-M04；MS03/MS05/MS06 职责；G4/G5/G7；固定 revisions；四级证据；≤450 行。
- Forbidden: 不声明地址 alias 已获真板证明；不选择默认 Kit DTS；不把 3 秒/8 次或历史日志写成安全保证；不展开 ring/RPC 消息语义。
- Test witness: `test ! -e docs/amp/k3-amp-shared-memory-lifecycle.md` 退出 0；index 无 AMP 入口。
- GREEN condition: R1、R2、R5 生命周期场景均有事实、推论或明确未知原因；直接来源已登记；相对链接有效；证据未升级。
- Verification: 首行来源、目录、状态表、地址/所有权、证据标签、四字段未知项、相对链接、行数、scoped diff、`git diff --check`、strict validate。
- Stop when: 源码 revision/控制流变化使 D3/D5 失效，或需要第三篇文档/修改既有主题责任。

**Invariants**

- M01：K3 硬件事实可追溯至唯一官方入口；第三方材料不替代它。
- M02：聚合产出只在 `docs/`；目标文档 kebab-case、首行来源、≤450 行目标。
- M03：简体中文为主，技术词保留英文主写法。
- M04：不新增脚本、构建工具或可执行代码。
- 相邻主题职责不变；本 Cycle 只串联生命周期，不复制 boot/mailbox/PMA 细节。
- 地址、时序、次数、revision 和运行结论按实际证据等级表述。
- 工作区既有 staged 修改与 `others/` 保持不变。

**Non-goals**

- Ring layout、原子序、BUSY、RPC、doorbell/IRQ 细节和等待者模型。
- G11、术语、总入口与全局状态收尾。
- 克隆缺失仓库、修正第三方路径、构建或运行第三方工程。
- QEMU、刷写、真板或性能验证。
- 运行身份、完整性或多会话认证机制。

**Acceptance**

1. T1 / R6：生命周期正文使用的直接官方 URL 在覆盖表各出现一次；总数等于唯一数，未登记第三方源码为官方来源。
2. T2 / R1-S1：正文按阶段给出 AP/RP 镜像/握手、窗口破坏者、初始化、发布和在线状态及所有者。
3. T2 / R1-S2、R2-S3/S4：AP/RP 地址、窗口大小、PMA/PBMT/cache 依赖和缺失源码边界均有证据等级；冲突不被静默消解。
4. T2 / R5-S9/S10：valid 保留、fallback/watchdog、对端 reset、通信暂停和未读消息可能丢失均有状态转换及未知项。
5. T2 / R6-S12：第三方行为携带固定 revision，作者声明、推论、未知项和本项目未运行边界明确。
6. 文档满足首行、相对链接、证据标签、四字段未知项、≤450 行、无 whitespace error 和 strict OpenSpec validation。

**Verification**

- RED witness：`test ! -e docs/amp/k3-amp-shared-memory-lifecycle.md` 预期退出 0；`rg -c 'docs/amp|k3-amp' docs/index.md` 无匹配。
- 来源：提取 `source-coverage.md` URL 列，比较总行数与 `sort -u` 行数；从正文首行提取 URL 并逐一确认覆盖表恰有一行。
- 内容：`rg` 检查生命周期阶段、地址、所有者、PMA/PBMT、reset/re-init、固定 revision、四级证据和未知项四字段。
- 链接：解析目标文档中的相对 Markdown 链接并确认仓库内目标存在；外部 URL 只核对精确登记，不做批量 refresh。
- 范围：`wc -l` 确认目标文档 ≤450；`git diff --name-only` 和目标路径 diff 确认只改 T1/T2；检查 `others/` 状态未变。
- Gate：`git diff --check` 退出 0；`openspec validate establish-k3-amp-rpc-baseline --strict` 退出 0。
- 任一内容检查失败表示对应 Acceptance 未满足；不得仅以文件存在或 OpenSpec schema 通过声明 GREEN。

**Gate 2 Readiness**

| Dimension | Status | Evidence |
| --- | --- | --- |
| Investigation | PASS | 已核对目标 docs、70/70 URL、10 gaps、无 AMP 入口、固定 revision HEAD/状态及关键生命周期符号 |
| Design | PASS | D1-D8 闭合文档边界、证据层级、生命周期模型、恢复语义、缺口和验证策略 |
| Iteration Plan | PASS | T1-T7 全部分配至两个依赖有序、可独立验证和诊断的 Iteration；平衡审计通过 |
| Cycle Scope | PASS | 当前 Cycle 只含 T1-T2，T3-T7 明确 deferred，不包含实现或全局收尾 |
| Task Contracts | PASS | T1/T2 含目标、当前/目标行为、保持/禁止、测试见证、GREEN、验证和停止条件 |
| Traceability | PASS | RTM 的 R1-R6/S1-S12 均为 Covered；本 Cycle Acceptance 映射 R1/R2/R5/R6 |
| Verification | PASS | 验证直接检查文档内容、来源、链接、计数、范围和退出码，无身份型证据工程 |
| User Plan Approval | PASS | 用户于本会话明确回复“批准，同意开始实施” |

**Persisted Evidence**

- Mode: none

Act Response 足以保存命令、决定性输出、退出码、改动文件和符号。全部检查可低成本重跑，没有一次性环境、Incident/Blocker 现场或会在摘要中丢失的决定性结构。

- Budget: 本 Cycle 最多 5 个文件（含 README），整个 change 最多 20 个 Evidence 文件；单个文本文件最多 500 行且不超过 256 KiB；超限需要用户明确批准。

**Risks and Notes**

- `others/` 在主仓中未跟踪，但其中三个固定 revision checkout 当前可读且干净；Act 只读核对，不纳入 diff。
- `rt-async`/`ov-channels` 缺失是已知材料边界，不阻塞生命周期文档；若实施前补齐且推翻地址/初始化模型，返回 Plan。
- 固定等待、watchdog 次数和注释中的板上时间仅作第三方策略/声明，不形成 Acceptance。
- Gate 2 全部维度 PASS；当前 Cycle 已可交给 Act。

## Act Response

- Status: reported

**Implemented**

- T1：复用 3 条既有官方 URL，把 boot 页面、docs-buildroot boot 页面和 Linux `k3.dtsi` 的主题职责扩充为 `amp`；没有新增 URL，也没有改变访问状态或观察日期。
- T2：创建 `docs/amp/k3-amp-shared-memory-lifecycle.md`，按镜像/握手、地址与布局、内存属性、生命周期、初始化竞争、reset/re-init 和未知项组织 AP/RP 共享窗口基线。
- Plan Review 修复（当前 Cycle 有限返修）：把 `amp` 加入 `docs/reference/source-coverage.md:16` 主题位置字段允许值说明，与第 35 / 72 / 84 行的 `boot+amp` / `platform+amp` 实际取值一致；URL 数量、唯一数、状态、观察日期、字段顺序和其余 9 个取值均不变。
- Change checklist：Iteration 000 的 T1、T2 已标记完成；T3-T7 保持未开始。

**Changed Files and Symbols**

- `docs/reference/source-coverage.md`：3 条既有来源记录增加 `amp` 职责与使用边界；line 16 字段说明在 `peripherals` 与 `总入口` 之间追加 `amp`。
- `docs/amp/k3-amp-shared-memory-lifecycle.md`：新增 205 行生命周期主题文档，包含 5 个四字段未知项。
- `openspec/changes/establish-k3-amp-rpc-baseline/tasks.md`：T1、T2 标记完成。
- 当前 Cycle 本区域：记录实施、修复、验证与 Self-Review。

**Deviations from Plan**

None。首次 source-coverage 补丁因磁盘原文品牌拼写与补丁上下文不同而未应用任何修改；随后按实际行精准应用，任务范围和行为未改变。Plan Review 的字段说明修复在 T1 当前 Cycle 契约内完成，不新增 Cycle、不改变 Acceptance、不修改 R1–R6 与 S1–S12。

**Blocker Handoff**

None

**Blocker Resolution**

None

**Self-Review**

- Plan compliance: PASS
- Full diff reviewed: PASS
- Critical findings unresolved: 0
- Important findings unresolved: 0
- Minor findings unresolved: 0

Spec compliance review 先检查 R1/R2/R5/R6、T1/T2、Acceptance 1-6、Invariants 和 Non-goals：原始 5 项 Acceptance 仍 PASS；Acceptance 1 的字段一致性现经 line 16 修复后满足。Code quality review 随后检查来源行精准性、字段说明插入位置、职责重复、证据升级、术语、链接、未知项结构和计划外修改：`amp` 插在 `peripherals` 与 `总入口` 之间延续既有顺序风格，不重排既有 9 个取值；`+amp` 复合值不要求 `amp` 单独出现；diff 局限于 line 16 + line 35 + line 72 + line 84 字段值；MS06/MS07 staged 与 `others/` 未触动；未引入身份型证据机制或新测试。完整 diff 复核确认本 Cycle 产品改动仅为 T1/T2 与 line 16 字段说明；既有 staged MS06/MS07 内容和 `others/` 未被修改。

**Verification Evidence**

| 验证项 | 命令或操作 | 决定性输出 | 覆盖范围 | 结论 |
| --- | --- | --- | --- | --- |
| Gate 3 RED | `test ! -e docs/amp/k3-amp-shared-memory-lifecycle.md`; `rg -n 'docs/amp\|k3-amp' docs/index.md` | 目标不存在退出 0；index 查询退出 1；URL `70/70`；amp 行 `0` | T1/T2 修改前状态 | PASS |
| 来源唯一性 | 提取 coverage URL 并比较总数/唯一数；反查正文首行 URL | `url_rows=70`；`url_unique=70`；3 个首行 URL 的 `coverage_count=1` | T1、Acceptance 1 | PASS |
| 字段一致性（Plan Review 修复） | `sed -n '16p' docs/reference/source-coverage.md`；`awk` 提取主题位置取值集合 | line 16 包含 `amp`；主题取值集合 `boot / boot+amp / buses / dma / interrupts / network / peripherals / platform / platform+amp / platform+boot / platform+serial / serial / storage / workflow-support / 总入口` | Acceptance 1、Plan Review Important 修复 | PASS |
| 正文结构 | `wc -l`、heading/关键词/四字段扫描 | `doc_lines=205`；`unknown_items=5`；生命周期、地址、PMA/PBMT、reset/re-init 与四字段均命中 | T2、Acceptance 2-5 | PASS |
| 相对链接 | 提取本地 Markdown link 并逐项 `test -e` | 11 个引用全部 `link_ok` | T2、Acceptance 6 | PASS |
| 范围与格式 | `git diff --check -- docs/reference/source-coverage.md docs/amp/k3-amp-shared-memory-lifecycle.md` | 退出 0；source-coverage 仅 4 行精准修改（line 16 + line 35 + line 72 + line 84）；目标文档 205 行 | T1/T2、Invariants、Plan Review 修复 | PASS |
| OpenSpec | `openspec validate establish-k3-amp-rpc-baseline --strict` | `Change 'establish-k3-amp-rpc-baseline' is valid`，退出 0 | change 一致性 | PASS |

**Persisted Evidence**

None required

**Experience Candidates**

None

**Remaining Issues**

None for Iteration 000。T3-T7 属于已规划但未授权展开的 Iteration 001。

**Commit or Diff Reference**

Uncommitted working tree; no commit created.

## Plan Review

- Review Result: accepted

**Findings**

None。上一轮审计发现的 Important 项已修复：`docs/reference/source-coverage.md` 的主题位置允许值现已包含 `amp`，与 3 条 `boot+amp` / `platform+amp` 记录一致。

**Deviation Classification**

PLAN-OMISSION（已解决）。初始 Cycle 未明确要求同步覆盖表字段说明；返修保持在 T1、Acceptance 1 和当前 Cycle 文件范围内，没有改变需求、设计或 Iteration 计划。

**Acceptance Gaps**

None。Acceptance 1-6 全部满足。

**Convergence**

reduced：未解决发现由 1 个 Important 降至 0，且没有新增发现。

**Evidence**

- 覆盖表 URL 为 `70/70`，生命周期正文首行 3 个官方 URL 在覆盖表中各出现一次。
- 字段说明包含 `amp`；3 条目标记录的主题位置与说明一致。
- 生命周期正文 205 行、5 个四字段未知项、11 个相对链接全部有效。
- scoped `git diff --check` 退出 0。
- `openspec validate establish-k3-amp-rpc-baseline --strict` 退出 0，输出 `Change 'establish-k3-amp-rpc-baseline' is valid`。

**Follow-up Decision**

接受 Iteration 000；按既定 Iteration Plan 展开 Iteration 001 的首个 draft Cycle，不授权实施。

**Iteration Plan Update**

None。

**Next Cycle**

None。

**Next Iteration**

`../001-rpc-ring-notification/000-initial.md`
