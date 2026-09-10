# Iteration 001 / Cycle 002: BUSY 提示语义返工

## Plan Context

- Status: ready
- Iteration: 001-rpc-ring-notification
- Cycle: 002-rework
- Cycle Type: rework
- Parent cycle: `001-rework.md`

**Iteration Scope**

- Change tasks: T4
- Depends on: Cycle 001 Review Result `rework-required`
- Stable baseline: BUSY 在全文只表示服务端弹性轮询状态和 doorbell 判据，不被描述为锁或并发门禁
- Verification boundary: 第 84、89、164 行与固定 revision 控制流一致，全文无相反表述
- Diagnostic boundary: `RpcClient::call_inner`、`process_elastic`、消息路径正文 BUSY 段
- Deferred tasks: None

**Cycle Scope**

- Trigger: rework
- Acceptance gaps: T4 / 父 Cycle Acceptance 2 的 BUSY 互斥语义与条件通知错误
- Repair items: T4-R3
- Inherited scope: T4、Acceptance 2；Cycle 001 已通过的第 85 行与 Act Response 单一性
- Excluded scope: 其他技术结论、父 Cycle 文件、G4 Minor、全局状态和运行验证

**Objective**

删除 BUSY 的锁/并发门禁表述，并把写端通知路径明确为 BUSY=0 时才发送 NOTIFY。

**Current Baseline**

- 第 85 行已准确描述写请求 → fence → 读 BUSY；BUSY=0 通知，BUSY=1 跳过门铃并由弹性轮询发现请求。
- 第 84 行仍称 BUSY 是“自旋锁标志”，只解决同侧重叠进入。
- 第 89 行写为“查 BUSY → NOTIFY”，遗漏 BUSY=0 条件。
- 第 164 行称 BUSY 组合可避免并发进入。

**Current-State Evidence**

- `call_inner` 写请求并 fence 后只执行 `if !is_busy() { notify(); }`，随后返回 rid。
- `process_elastic` 入口无条件 `set_busy()`，完成弹性轮询后无条件 `clear_busy()`；没有 CAS、try-lock、失败分支或重入门禁。
- BUSY=1 告知写端服务端正处于可发现请求的弹性窗口，因此可跳过 doorbell；它不证明互斥。
- 重复通知是否导致并发进入还依赖任务调度和调用约束，当前可读 BUSY 操作本身不提供该保证。

**Relevant Code**

| 文件或符号 | 当前职责 | 本 Cycle 用法 |
| --- | --- | --- |
| `docs/amp/k3-rpc-ring-notification.md:84` | BUSY 角色 | 删除锁与重入保证 |
| `docs/amp/k3-rpc-ring-notification.md:89` | 写端顺序摘要 | 补 BUSY=0 条件 |
| `docs/amp/k3-rpc-ring-notification.md:164` | 重复通知边界 | 删除 BUSY 避免并发进入的保证 |
| `client.rs::call_inner` | 条件通知 | 固定 revision 对照 |
| `intercom.rs::process_elastic` | 弹性轮询窗口 | 固定 revision 对照 |

**Behavioral Change**

正文不再把 BUSY 当作自旋锁或重入防护；它只描述服务端是否处于弹性轮询窗口以及客户端是否需要 doorbell。

**Change Surface**

| Repair | Source Task/Acceptance | Target | Required Change |
| --- | --- | --- | --- |
| T4-R3 | T4 / Acceptance 2 | 消息路径正文第 84、89、164 行 | 修正 BUSY 角色、条件通知与重复通知边界 |

**Repair Item Contract**

### T4-R3：BUSY 提示而非锁

- Required behavior: BUSY=1 表示服务端处于弹性轮询窗口；客户端据此跳过 doorbell。BUSY=0 时客户端发送 NOTIFY。正文不赋予 BUSY 互斥或重入保证。
- Preserve: 第 62、85 行正确控制流；其他章节、来源、链接、未知项和计数。
- Forbidden: 不称 BUSY 为 lock/spinlock；不声明它单独避免并发/重叠进入；不把 NOTIFY 写成无条件动作；不设计新同步机制。
- Test witness: 第 84、164 行包含锁/并发保证，第 89 行遗漏 BUSY=0 条件。
- GREEN: 三处表述与源码一致，全文无锁/互斥保证或无条件 NOTIFY 的相反表述。
- Verification: 搜索目标短语；人读全文 BUSY 命中；对照源码；scoped diff；diff check；strict validate。
- Stop when: 需要证明调度层并发约束或改变协议设计。

**Invariants**

- 只修改目标正文三处和本 Cycle Act Response。
- 父 Cycle、`others/`、reference/index、全局状态不变。
- 不把源码注释或 host 行为升级为真板保证。

**Non-goals**

- 证明 `process_elastic` 的调用串行化或增加锁。
- 修改实现代码、测试第三方工程或处理其他 Minor。
- Evidence、Runbook、Incident、Maintainer 或归档。

**Acceptance**

1. 第 84 行将 BUSY 描述为弹性轮询/通知提示，不是锁或 DMA ownership。
2. 第 89 行明确仅 BUSY=0 才 NOTIFY。
3. 第 164 行不声称 BUSY 防止并发进入，并保留重复通知可能造成空转的边界。
4. 全文 BUSY 表述与 `call_inner`/`process_elastic` 一致，无反向结论。
5. scoped diff 无计划外修改，diff check 和 strict validate 通过。

**Verification**

- RED：`rg -n '自旋锁标志|只解决.*重叠进入|BUSY.*避免并发进入|查 BUSY → NOTIFY' docs/amp/k3-rpc-ring-notification.md` 应命中第 84、89、164 行。
- GREEN：同一搜索应无命中。
- 人读所有 BUSY 命中并对照 `call_inner` 与 `process_elastic`。
- `git diff --check -- docs/amp/k3-rpc-ring-notification.md .../002-rework.md`。
- `openspec validate establish-k3-amp-rpc-baseline --strict`。

**Gate 2 Readiness**

| Dimension | Status | Evidence |
| --- | --- | --- |
| Investigation | PASS | 已定位三处错误与两个固定 revision 控制流 |
| Design | PASS | 只删除无源码依据的互斥语义并补条件通知 |
| Iteration Plan | PASS | 仍属于 T4 / Iteration 001 |
| Cycle Scope | PASS | 单一 repair，三处同类表述 |
| Task Contracts | PASS | 目标、保持、禁止、见证、GREEN、验证和停止条件完整 |
| Traceability | PASS | T4-R3 → T4 / Acceptance 2 |
| Verification | PASS | 搜索、源码对照、diff 与 OpenSpec Gate |
| User Plan Approval | PASS | 用户明确回复“批准实施” |

**Persisted Evidence**

- Mode: none

**Risks and Notes**

- 这是同一 Iteration 的第二个 rework Cycle；若同一 BUSY 语义 gap 再次不收敛，必须执行三次失败规则，不创建第四次同类 Cycle。
- 用户已批准本 Cycle；Gate 2 全部 PASS，可交给 Act。

## Act Response

- Status: reported

**Implemented**

- T4-R3：修正消息路径正文三处 BUSY 表述。第 84 行将其定义为弹性轮询状态提示而非锁；第 89 行补充仅 BUSY=0 才 NOTIFY；第 164 行删除并发保护保证，保留重复通知可能导致额外唤醒和空转的边界。

**Changed Files and Symbols**

- `docs/amp/k3-rpc-ring-notification.md:84`：删除自旋锁、互斥和重入保护结论。
- `docs/amp/k3-rpc-ring-notification.md:89`：将 NOTIFY 明确为 BUSY=0 条件动作。
- `docs/amp/k3-rpc-ring-notification.md:164`：删除 BUSY 避免并发进入的结论，说明串行性依赖调用和调度约束。
- 本文件 `Act Response`：记录实施、自检与验证。

**Deviations from Plan**

None。仅修改 T4-R3 指定的三处表述。

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

Spec review：三处修改分别满足 Acceptance 1-3；全文 BUSY 命中与 `call_inner`、`process_elastic` 一致，没有把 BUSY 写成锁、互斥门禁或无条件通知。

Code quality review：产品 diff 仅涉及第 84、89、164 行；来源、链接、未知项、计数和父 Cycle 均未修改。措辞区分了可读源码事实与尚未证明的调度串行性。

**Verification Evidence**

| 验证项 | 命令或操作 | 决定性输出 | 覆盖范围 | 结论 |
| --- | --- | --- | --- | --- |
| Gate 3 RED | 搜索 `自旋锁标志\|只解决.*重叠进入\|BUSY.*避免并发进入\|查 BUSY → NOTIFY` | 修复前命中第 84、89、164 行，退出 0 | T4-R3 当前错误 | PASS |
| GREEN 搜索 | 同一搜索 | `0 matches` | Acceptance 1-3 | PASS |
| 全文 BUSY 审查 | `rg -n -i 'BUSY' docs/amp/k3-rpc-ring-notification.md` 并对照固定 revision 源码 | 第 84 行明确非锁；第 89 行仅 BUSY=0 通知；第 164 行不提供并发保证 | Acceptance 4 | PASS |
| Scoped review | 人读产品三处修改和本 Cycle diff | 无计划外产品修改，父 Cycle 未改写 | Acceptance 5 | PASS |
| Whitespace | `git diff --check` | 无输出，退出 0 | 工作区修改 | PASS |
| OpenSpec | `openspec validate establish-k3-amp-rpc-baseline --strict` | `Change 'establish-k3-amp-rpc-baseline' is valid`，退出 0 | change 一致性 | PASS |

**Persisted Evidence**

None required。验证可低成本重跑，Plan Context 模式为 `none`。

**Experience Candidates**

None

**Remaining Issues**

None

**Commit or Diff Reference**

Uncommitted working tree; no commit created.

## Plan Review

- Review Result: accepted

**Findings**

None。第 84、89、164 行均已按固定 revision 控制流修正，没有未解决的 Critical、Important 或 Minor finding。

**Deviation Classification**

None。Act 严格执行 T4-R3，未修改父 Cycle 或计划外产品表面。

**Acceptance Gaps**

None。当前 Cycle Acceptance 1-5 全部满足；继承的 T4 / Iteration 001 Acceptance 2 也已闭合。

**Convergence**

reduced：上一 Cycle 新发现的 3 处 BUSY/NOTIFY 语义 gap 降至 0；同类问题全文扫描无命中。

**Evidence**

- 独立对照 `client.rs::call_inner`：写请求 → SeqCst fence → 读取 BUSY，仅 BUSY=0 时通知，随后返回 rid。
- 独立对照 `process_elastic`：入口设置 BUSY、弹性轮询、清 BUSY、fence 与最终重检；没有 CAS、锁获取或重入门禁。
- 第 84 行明确 BUSY 不是锁，第 89 行明确仅 BUSY=0 时 NOTIFY，第 164 行不再声明并发保护。
- 禁止语义搜索为 0 命中；本 Cycle Act Response 的 11 个标准字段各出现一次。
- 聚合基线保持 URL `70/70`、正文 246 行、未知项 5、术语 25、缺口 11、T1-T7 全部完成。
- `git diff --check` 退出 0；`openspec validate establish-k3-amp-rpc-baseline --strict` 退出 0。

**Follow-up Decision**

接受 Cycle 002 和 Iteration 001。当前 change 没有剩余 Iteration，可交由 `openspec-docs-maintainer` 同步并收尾。

**Iteration Plan Update**

None

**Next Cycle**

None

**Next Iteration**

None
