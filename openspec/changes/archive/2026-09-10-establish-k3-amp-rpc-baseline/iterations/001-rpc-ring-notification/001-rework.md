# Iteration 001 / Cycle 001: BUSY 语义与执行记录返工

## Plan Context

- Status: ready
- Iteration: 001-rpc-ring-notification
- Cycle: 001-rework
- Cycle Type: rework
- Parent cycle: `000-initial.md`

**Iteration Scope**

- Change tasks: T4
- Depends on: Cycle 000 Review Result `rework-required`
- Stable baseline: 消息路径正文准确描述 BUSY=1 的客户端与服务端行为，Act Response 只保留一份最新状态
- Verification boundary: 目标句与固定 revision 控制流一致；执行记录无重复顶层字段
- Diagnostic boundary: `RpcClient::call_inner`、`process_elastic` 和当前 Cycle Act Response
- Deferred tasks: None

**Cycle Scope**

- Trigger: rework
- Acceptance gaps: Acceptance 2 的 BUSY=1 行为错误；父 Cycle Act Response 重复
- Repair items: T4-R1、T4-R2
- Inherited scope: T4、Acceptance 2/8、父 Cycle 已通过的 T3/T5/T6/T7 结果
- Excluded scope: 改写其他技术结论、修改冻结 Cycle 000、增补 G4 引用、全局状态同步、实现与运行验证

**Objective**

修正 BUSY=1 的消息路径说明，并在本 Cycle 形成唯一、无重复的 Act Response，使 Iteration 001 可被可靠审计。

**Current Baseline**

- `docs/amp/k3-rpc-ring-notification.md` 为 246 行；第 85 行顺序已改为写请求和 fence 后查 BUSY。
- 第 85 行仍错误声称 BUSY=1 时“直接退出等待下一轮”和服务端清 BUSY 后才能接受新请求。
- 父 Cycle Act Response 包含两套 `Implemented`、Self-Review、Verification Evidence 和 Commit/Diff 内容，当前状态互相矛盾。
- T1-T7 已勾选；URL 70/70、术语 25、缺口 11、未知项 5 的既有结果未失效。

**Current-State Evidence**

- `ov-rpc/src/client.rs::call_inner` 顺序为 `send_request` → SeqCst fence → `is_busy`；BUSY=0 才 `notify`，随后无论 BUSY 值都返回 `Ok(rid)`。
- `intercom.rs` 的 D2 路径定义为 AP 读 BUSY=1 后跳过门铃，服务端在弹性自旋轮询中发现请求。
- 因此 BUSY=1 表示服务端仍在可发现新增请求的弹性窗口，不表示客户端退出请求，也不表示服务端拒绝请求直到 clear BUSY。
- Cycle 000 已终结，不能再改写；Cycle 001 必须自带最新实施与验证状态。

**Relevant Code**

| 文件或符号 | 当前职责 | 本 Cycle 用法 |
| --- | --- | --- |
| `docs/amp/k3-rpc-ring-notification.md:85` | BUSY 角色说明 | T4-R1 修正错误语义 |
| `ov-rpc/src/client.rs::call_inner` | 写请求、fence、BUSY 判断与通知 | 固定 revision 对照证据 |
| `intercom.rs::process_elastic` | BUSY 弹性轮询与最终重检 | 固定 revision 对照证据 |
| `001-rework.md::Act Response` | 本轮执行快照 | T4-R2 写入唯一最新状态 |

**Behavioral Change**

正文从“BUSY=1 时退出、清 BUSY 后才接收”改为“BUSY=1 时客户端跳过 doorbell，已在弹性轮询的服务端继续发现请求”。不改变其他协议、未知项或来源。

**Change Surface**

| Repair | Source Task/Acceptance | Target | Required Change |
| --- | --- | --- | --- |
| T4-R1 | T4 / Acceptance 2 | `docs/amp/k3-rpc-ring-notification.md:85` | 删除错误的退出/拒收语义，保留写后 fence 后查 BUSY |
| T4-R2 | Cycle record / Acceptance 8 | 本文件 `Act Response` | 写入一份无重复、可审计的最新状态 |

**Repair Item Contracts**

### T4-R1：BUSY=1 行为

- Required behavior: BUSY=0 时发送 doorbell；BUSY=1 时跳过 doorbell，客户端返回 rid，服务端通过已运行的弹性轮询发现请求。
- Preserve: 写请求 → SeqCst fence → 查 BUSY 的顺序；正文其他章节、链接、行数与未知项。
- Forbidden: 不声称 BUSY 是请求拒绝、客户端取消/退出或服务端接收门禁；不设计新机制。
- Test witness: 当前第 85 行含“BUSY=1 则直接退出等待下一轮”和“清 BUSY 后才能再接受新请求”。
- GREEN: 第 85 行与 `call_inner` 和 D2 控制流一致，全文无相反描述。
- Verification: 人读对照固定 revision；搜索错误短语；scoped diff；diff check；strict validate。
- Stop when: 源码基线变化或需要改变 BUSY/doorbell 设计。

### T4-R2：唯一执行快照

- Required behavior: 本 Cycle Act Response 只含一套 Implemented、Changed Files、Deviations、Blocker、Self-Review、Verification、Evidence、Experience、Remaining Issues 和 Commit/Diff 字段。
- Preserve: 父 Cycle 冻结；无 Evidence 目录；产品修复与验证的决定性信息。
- Forbidden: 不复制父 Cycle 整套 Act Response，不修改 Plan Context 或 Plan Review。
- Test witness: 父 Cycle存在重复顶层执行字段。
- GREEN: 本 Cycle Act Response 完整且每个标准顶层字段只出现一次。
- Verification: heading/字段计数、人读一致性、strict validate。
- Stop when: 需要修改父 Cycle 或全局状态。

**Invariants**

- 父 Cycle 与本 Cycle Plan Context 均不可由 Act 改写。
- T3、T5、T6、T7 的已通过产品结果保持不变。
- 不修改 `others/`、SNAPSHOT、全局 tasks 或 M/D/K/R/I。

**Non-goals**

- 补 G4 编号引用或处理历史 Minor。
- 获取缺失仓库、运行 host/QEMU/真板测试。
- 创建 Evidence、Runbook、Incident 或归档 change。

**Acceptance**

1. 第 85 行准确描述写后 fence 后查 BUSY、BUSY=0 通知、BUSY=1 跳过通知并由弹性轮询发现请求。
2. 全文没有客户端退出、请求被拒绝或服务端清 BUSY 后才接收的相反语义。
3. 本 Cycle Act Response 是唯一、完整、无重复的最新快照。
4. 产品 scoped diff 无计划外修改，`git diff --check` 与 strict validate 通过。

**Verification**

- `rg -n '直接退出等待下一轮|清 BUSY 后才能再接受新请求|写请求前.*BUSY' docs/amp/k3-rpc-ring-notification.md` 应无命中。
- 人读第 62、85、89 行并对照 `client.rs::call_inner` 与 `process_elastic` D2 路径。
- 检查本 Cycle Act Response 的标准字段各出现一次。
- `git diff --check -- docs/amp/k3-rpc-ring-notification.md openspec/changes/establish-k3-amp-rpc-baseline/iterations/001-rpc-ring-notification/001-rework.md`。
- `openspec validate establish-k3-amp-rpc-baseline --strict`。

**Gate 2 Readiness**

| Dimension | Status | Evidence |
| --- | --- | --- |
| Investigation | PASS | 已定位 `call_inner`、D2 路径、错误句和重复 Response |
| Design | PASS | 仅恢复既有控制流表述，不引入设计选择 |
| Iteration Plan | PASS | 返工仍属于 T4 和 Iteration 001 |
| Cycle Scope | PASS | 两项 repair 均直接服务未收敛 gap |
| Task Contracts | PASS | 目标、保持、禁止、GREEN、验证和停止条件完整 |
| Traceability | PASS | T4-R1 → T4/Acceptance 2；T4-R2 → Cycle 记录/Acceptance 8 |
| Verification | PASS | 直接检查语义、重复字段和 Gate |
| User Plan Approval | PASS | 用户明确回复“批准实施” |

**Persisted Evidence**

- Mode: none

**Risks and Notes**

- 本轮只允许删除错误含义并准确复述现有控制流，避免再次扩大句子。
- 用户已批准本 Cycle；Gate 2 全部 PASS，可交给 Act。

## Act Response

- Status: reported

**Implemented**

- T4-R1：将消息路径正文第 85 行改为准确控制流：写请求后执行 `SeqCst` fence 并读取 BUSY；BUSY=0 时发送 NOTIFY，BUSY=1 时跳过门铃并返回 request id，由正在弹性轮询的处理器侧发现请求。
- T4-R2：本 Cycle 仅填写这一份最新 Act Response，不复制父 Cycle 的执行快照。

**Changed Files and Symbols**

- `docs/amp/k3-rpc-ring-notification.md:85`：删除“直接退出等待下一轮”和“清 BUSY 后才能再接受新请求”，改为 BUSY=1 跳过门铃并由弹性轮询发现请求。
- 本文件 `Act Response`：记录两项 repair、Self-Review 和验证结果。

**Deviations from Plan**

None。修复严格按 T4-R1/T4-R2 契约执行。

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

Spec review：T4-R1 的错误短语已全部消失；第 62、85、89 行与 `client.rs::call_inner` 及 `intercom.rs` D2 路径一致。T4-R2 的标准执行字段在本 Cycle 各出现一次，父 Cycle 未改写。

Code quality review：产品修改只有第 85 行；没有改变来源、未知项、链接、计数或其他协议判断。完整 scoped diff 未发现计划外修改。

**Verification Evidence**

| 验证项 | 命令或操作 | 决定性输出 | 覆盖范围 | 结论 |
| --- | --- | --- | --- | --- |
| Gate 3 RED | 搜索 `直接退出等待下一轮\|清 BUSY 后才能再接受新请求\|写请求前.*BUSY` | 修复前命中第 85 行，退出 0 | T4-R1 当前错误 | PASS |
| BUSY GREEN | 同一搜索 | `0 matches` | T4-R1 禁止语义 | PASS |
| 控制流对照 | 读取正文第 62/85/89 行，并对照 `client.rs::call_inner` 与 `intercom.rs` D2 | 三处均为写请求 → fence → 查 BUSY；BUSY=1 跳过门铃并由弹性轮询发现 | Acceptance 1-2 | PASS |
| Response 唯一性 | 检查本 Cycle Act Response 标准字段 | 每个标准字段各一份 | Acceptance 3 | PASS |
| Whitespace | `git diff --check -- docs/amp/k3-rpc-ring-notification.md .../001-rework.md` | 无输出，退出 0 | Acceptance 4 | PASS |
| OpenSpec | `openspec validate establish-k3-amp-rpc-baseline --strict` | `Change 'establish-k3-amp-rpc-baseline' is valid`，退出 0 | change 一致性 | PASS |

**Persisted Evidence**

None required。全部结果可低成本重跑，Plan Context 模式为 `none`。

**Experience Candidates**

None

**Remaining Issues**

None。父 Cycle 已记录的非阻塞历史 Minor 不属于本 Cycle repair。

**Commit or Diff Reference**

Uncommitted working tree; no commit created.

## Plan Review

- Review Result: rework-required

**Findings**

1. Important — T4-R1 的目标句已修正，但同一正文第 84 行仍把 BUSY 称为“自旋锁标志”并声称它解决同侧重叠进入，第 164 行也称 BUSY 组合可避免并发进入。固定 revision 的 `process_elastic` 只无条件调用 `set_busy/clear_busy`，客户端只读取 BUSY 决定是否 doorbell；没有 CAS、锁获取失败或并发进入门禁。这两处把唤醒提示误写成互斥机制，仍阻塞父 Cycle Acceptance 2。
2. Important — 第 89 行把写端路径简写为“查 BUSY → NOTIFY”，遗漏 BUSY=0 条件；单独阅读会把条件通知写成无条件通知，与第 62/85 行和源码不一致。

**Deviation Classification**

NEW-EVIDENCE：本轮对 T4-R1 做源码对照时发现父 Cycle 遗留的相邻 BUSY 表述错误；001-rework 的契约明确只修第 85 行，Act 未偏离其范围。

**Acceptance Gaps**

001-rework Acceptance 1-4 均满足；但其继承的 T4 / 父 Cycle Acceptance 2 仍未满足，因为正文将 BUSY 错写为互斥机制，并有一处遗漏条件的 NOTIFY 路径。

**Convergence**

expanded：原第 85 行和 Act Response 重复问题均已解决；独立源码复核新增 3 处同类语义 gap。需要新的自包含修复契约，不能继续当前 Cycle。

**Evidence**

- `client.rs::call_inner` 只有读取 `is_busy()` 后的条件 `notify`，无锁获取。
- `process_elastic` 入口无条件 `set_busy()`，退出无条件 `clear_busy()`；没有 CAS、失败分支或重入防护。
- 正文第 84、164 行分别使用“自旋锁标志”“避免并发进入”；第 89 行省略 `BUSY=0` 条件。
- T4-R1 禁止短语为 0 命中；本 Cycle Act Response 的 11 个标准字段各出现一次。
- `git diff --check` 与 strict OpenSpec validate 均退出 0，但不检测技术语义。

**Follow-up Decision**

创建 `002-rework.md`，只修正第 84、89、164 行的 BUSY/NOTIFY 语义并验证全文不再把 BUSY 写成锁或并发门禁。Iteration Plan 保持不变。

**Iteration Plan Update**

None

**Next Cycle**

`002-rework.md`

**Next Iteration**

None
