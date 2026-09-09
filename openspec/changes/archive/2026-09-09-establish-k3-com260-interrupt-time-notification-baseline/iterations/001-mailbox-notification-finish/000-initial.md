# Iteration 001 / Cycle 000: mailbox 通知与收尾

## Plan Context

- Status: ready
- Iteration: 001-mailbox-notification-finish
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: None

**Iteration Scope**

- Change tasks: T3-T5
- Depends on: Iteration 000 accepted
- Stable baseline: MS05 的中断、时间、mailbox 通知、G4 与总入口形成一致的可引用文档包
- Verification boundary: 双向通知链、残留/重复处理、自测边界、G4 状态、入口和权威计数一致
- Diagnostic boundary: mailbox 实例与命名空间、AP/RP handler、APLIC/PLIC delivery、gap、导航
- Deferred tasks: None

**Cycle Scope**

- Trigger: Iteration 000 accepted
- Acceptance gaps: T3-T5 尚未实施
- Repair items: None
- Inherited scope: proposal R2/R4/R5、design D2-D5、Iteration 000 已接受的 AP/RP 中断与时间基线
- Excluded scope: 修改 `others/`、驱动实现、RPC/共享内存数据面设计、真板测试声明、来源 refresh、全局状态维护和归档

**Objective**

建立 AP↔RP mailbox 通知事实包，并使 G4 与 `docs/index.md` 准确反映 MS05 已解除和仍未知的范围。

**Current Baseline and Evidence**

- `docs/interrupts/com260-mailbox-notification.md` 不存在；`docs/index.md` 的 interrupts 仍为“待聚合”，只列 R05 Timer。
- `docs/reference/known-gaps.md` 有 G1-G10 共 10 项；G4 仍为 `open`，只写“未公开 APLIC/IMSIC 地址与 topology”，尚未纳入 Iteration 000 已确认的 CLINT `0xe081c000`、IMSIC `0xe0400000`、APLIC `0xe0804000`、512 sources、511 IDs 与 `msi-parent`。
- `docs/reference/source-coverage.md` 有 62 个唯一 URL；`docs/interrupts/k3-interrupt-and-time.md` 已接受，首行 6 个来源元数据与覆盖表一致。
- 固定 revision：Rt-Async-AMP `ccb1ff0b487e4f49ea570c41f330741eecece935`，tgoskits `19219411d5dc1515496f910d04c93da12ee95be4`。两者只提供第三方实现经验，不升级为 K3 官方事实。
- RP 侧 `its/rt-async-k3.dts` 只描述 rcpu1 视角的物理 mailbox4：`0xcac91000/0x400`、IRQ 69；物理 mailbox3 被注释为 rcpu0/esos 使用。`mailbox.rs` 的历史变量 `MBX3` 实际绑定物理 mailbox4，必须分开“变量/池 slot”与“物理实例”命名空间。
- AP 侧 `spacemit-k3-com260-ifx.dts` notifier 使用 mailbox4 `0xcac91000/0x400`、tx-channel 0、rx-channel 1，并把 AP 接收线描述为 APLIC source 217；RP 侧接收 IRQ 为 PLIC source 69。二者不是同一 IRQ 编号空间。
- RP `mailbox.rs` 与 AP `rt_shm.rs` 都按“排空 FIFO → 清 pending → 重读 pending”处理 NEW_MSG；AP 实现有 32 轮外层上限与每轮 64 次 drain 上限。此行为只按固定 revision 记录，不声明为硬件规范。
- AP `rt_shm.rs` 含 boot 自测和 ioctl 自测入口：本地写 FIFO 后观察 handler 计数，最多 3 次、每次 100000 次 spin；代码存在不等于本项目已运行或真板通过。
- notification 只促使接收方重新检查共享状态；mailbox channel、ov-channels channel、APLIC source、PLIC source、IMSIC EID 必须分列，不建立数值等同关系。

**Relevant Code and Documents**

- `docs/interrupts/k3-interrupt-and-time.md`：已接受的 AP/RP domain、wired/MSI 和未知项术语。
- `others/Rt-Async-AMP/its/rt-async-k3.dts`：RP mailbox4 节点、物理地址、IRQ 和 user/channel 注释。
- `others/Rt-Async-AMP/modules/chip-k3-rt24/src/mailbox.rs`：RP probe、signal、FIFO drain、pending clear、PLIC setup 与 async latch。
- `others/Rt-Async-AMP/tgoskits/os/StarryOS/configs/board/spacemit-k3-com260-ifx.dts`：AP notifier 的 mailbox base、tx/rx channel 和 AP interrupt source。
- `others/Rt-Async-AMP/tgoskits/os/StarryOS/kernel/src/pseudofs/dev/rt_shm.rs`：AP notify、handler、drain/clear/re-read、自测、单等待者和工作上限。
- `others/Rt-Async-AMP/user-apps/user-test-mbox/src/main.rs`：用户态自测入口与结果解释。
- `docs/reference/known-gaps.md` G4、`docs/index.md`：T4/T5 目标。

**Critical Path**

1. 以固定 revision 建立 AP→RP 与 RP→AP 两条通知链，分别标注物理 mailbox、user、hardware channel、wired source、controller 和 handler。
2. 分开 mailbox channel、共享内存 channel、APLIC/PLIC source 与 EID，未观察到的 EID 保留未知。
3. 记录初始化清残留、ISR 排空/清除/重读、工作上限、单等待者和重复通知边界；不把第三方实现提升为硬件规范。
4. 记录自测“能测什么、不能测什么”，明确未在本项目运行。
5. 用 Iteration 000 的静态事实把 G4 更新为 `partial`，保留寄存器、EID、affinity、真板 delivery 等缺口。
6. 更新 interrupts 入口、已聚合状态、62 个来源与实际 G 项数，并验证所有相对链接。

**Change Surface**

| Task | Requirement | File | Planned Change |
| --- | --- | --- | --- |
| T3 | R2,R4,R5 | `docs/interrupts/com260-mailbox-notification.md` | 创建双向通知与故障边界文档 |
| T4 | R5 | `docs/reference/known-gaps.md` | G4 `open → partial`，不新增同义缺口 |
| T5 | R5 | `docs/index.md` | 增加两篇 interrupts 入口并同步状态与计数 |

**Task Contracts**

### T3：mailbox 双向通知链

- Requirement/Scenario: R4-S1-S4；R2-S2；R5-S1-S2。
- Depends on: Iteration 000 accepted。
- Targets: `docs/interrupts/com260-mailbox-notification.md`。
- Test witness: 文件不存在；`docs/index.md` 无目标链接；现有中断文档明确把 mailbox 正文留给本 Iteration。
- Required behavior: 首行来源采用覆盖表已登记 URL 的精确元数据；正文用 R10-R12 与固定 revision 路径标注第三方经验，分别画出 AP→RP、RP→AP 的 `共享状态发布 → mailbox FIFO 门铃 → wired controller → handler → 重查共享状态` 链。
- Required fields: 物理 mailbox4 与历史 `MBX3` 变量名区别；base/size；USER0/USER1；tx=0/rx=1；AP source 217、RP source 69；FIFO 4 channel × depth 8；NEW_MSG mask；初始化 drain；ISR drain/clear/re-read；工作上限；waker/单等待者；self-test 入口、尝试上限与未运行状态。
- Preserve: notification≠数据真值；AP/RP、wired/MSI、channel/source/EID 命名空间分离；固定 revision；四级证据；未知项四字段；少于 450 行。
- Forbidden: 不把第三方 DTS、注释、实测描述提升为官方事实；不声明 self-test 通过、无丢失、无重复或 SMP affinity 完整；不复制 RPC/共享内存协议正文；不修改 `others/`。
- GREEN condition: 双向链、残留与 burst 分支、自测能力边界、错误/超时/工作预算和未知项均可追溯；与中断时间文档无冲突。
- Verification: 首行来源精确匹配、固定 revision/path、字段与双向链关键词、证据等级、未知项、链接、标题、行数和 diff check。
- Stop when: 物理 mailbox/user/channel 或 AP/RP source 证据不能按来源层解释，或必须新增来源 URL 才能满足首行规则。

### T4：G4 partial 收敛

- Requirement/Scenario: R5-S2。
- Depends on: T3。
- Targets: `docs/reference/known-gaps.md` G4。
- Test witness: G4 当前未记录 Iteration 000 的静态 topology，且无 `partial` 状态记录。
- Required behavior: 把 G4 当前证据拆成已解除与未解除部分，状态改为 `partial`；引用两篇 interrupts 文档；保留 programmer reference、寄存器布局、EID/affinity、运行态 delivery 等未知边界。
- Preserve: G1-G3、G5-G10 原文和顺序；总数仍为 10，除非 T3 发现语义独立且不可并入 G4 的新缺口。
- Forbidden: 不凭第三方 mailbox 代码关闭 G4；不新增与 mailbox/中断文档未知项同义的 G 项；不修改其他 G 状态。
- GREEN condition: G4 与两篇正文一致，状态为 `partial`，四字段和状态变更记录完整，总数可解释。
- Verification: scoped diff、G 编号/数量、G4 字段/状态/链接、相邻 G 段不变、diff check。
- Stop when: T3 出现无法并入 G4 的新硬件事实缺口，或需改变既有缺口分类。

### T5：入口与权威计数

- Requirement/Scenario: R5-S3。
- Depends on: T3-T4。
- Targets: `docs/index.md`。
- Test witness: interrupts 行仍为“待聚合”；正文列表没有两篇目标文档；来源计数仍需对照 62，缺口计数需对照 G1-G10。
- Required behavior: 在文档入口加入 `k3-interrupt-and-time.md` 与 `com260-mailbox-notification.md` 相对链接；interrupts 标为已聚合并保留 G4 partial 边界；仅按权威文件同步来源 62、缺口 10。
- Preserve: 其他主题入口、状态和正文；不复制 mailbox 技术事实。
- Forbidden: 不修改后续 milestone 状态；不把 G4 标为 closed；不以 staged/untracked 状态替代链接验证。
- GREEN condition: 两个入口可解析；interrupts 状态、来源数、缺口数分别与目标正文、source-coverage、known-gaps 一致。
- Verification: 相对链接解析、状态文本、62/10 计数对照、scoped diff、全量 Markdown 链接检查和 diff check。
- Stop when: T3/T4 未 GREEN，或权威计数在实施时变化。

**BDD and Failure Coverage**

| Scenario | Preconditions | Action | Expected Result | Failure Boundary |
| --- | --- | --- | --- | --- |
| AP→RP | AP 发布共享状态 | AP USER0 写 mailbox4 ch0 | RP PLIC 69 handler drain/clear 后重查共享状态 | 不把门铃值当业务数据 |
| RP→AP | RP 发布共享状态 | RP USER1 写 mailbox4 ch1 | AP source 217 经 APLIC/IMSIC 到 handler 后重查共享状态 | EID 未证实则保留未知 |
| 启用前残留 | FIFO/pending 已置位 | 初始化先 drain/clear 再 enable | 首次等待不消费幽灵通知 | 只按固定 revision 描述 |
| ISR 期间 burst | drain/clear 时新消息到达 | 外层重读 RAW & EN | 有界继续处理或留下明确风险 | 不声明绝不丢通知 |
| 自测存在但未运行 | 源码含 boot/ioctl 测试 | 文档记录入口和覆盖链 | 标为能力，不标为通过证据 | 不替代双端或真板测试 |
| 多等待者 | AP waker 为单槽 | 两个等待者并发注册 | 记录后注册覆盖风险 | 不推导多消费者安全 |
| 计数基线变化 | source/G 数发生变化 | T5 对照权威文件 | 停止并返回 Plan | 不写死过期计数 |

**Verification Plan**

1. RED：目标 mailbox 文档和 index 链接不存在；G4 不含 `partial` 状态记录。
2. T3 GREEN：检查首行元数据、双向链、命名空间、fixed revision、残留/burst/self-test/单等待者/工作上限、5 类禁止声明和行数。
3. T4 GREEN：只改 G4；G1-G10 数量 10；G4 `partial`、四字段和两篇链接存在。
4. T5 GREEN：两个链接可解析；interrupts 已聚合；来源 62、缺口 10 与权威文件一致。
5. 全量：Markdown 相对链接解析、`git diff --check`、`git diff --cached --check`、`openspec validate establish-k3-com260-interrupt-time-notification-baseline --strict`。

**Stop Conditions**

- mailbox 物理实例、user/channel 或 AP/RP source 的固定 revision 证据发生实质冲突。
- 必须新增未登记 URL、刷新既有来源或修改 `others/` 才能完成正文。
- G4 之外出现必须新增的独立硬件缺口，或 source/gap 权威计数不再是 62/10。
- 需要声称本项目真板运行、自测通过、无丢通知或完整 affinity 才能满足验收。

**Gate 2 Readiness**

| Dimension | Status | Evidence |
| --- | --- | --- |
| Requirement Coverage | PASS | T3-T5 覆盖 R2/R4/R5 的本 Iteration scenarios |
| Simplifications | PASS | 无裁剪 |
| Investigation | PASS | AP/RP DTS、mailbox 驱动、AP rt_shm、自测、G4 和 index 已核对 |
| Design | PASS | D2-D5 固定来源层、通知≠数据、缺口和入口边界 |
| Iteration Plan | PASS | T3→T4→T5 依赖闭合，共享最终一致性边界 |
| Cycle Scope | PASS | 仅三篇产品文档，不修改驱动或全局状态 |
| Task Contracts | PASS | 每项含见证、行为、不变量、禁止项、GREEN、验证和停止条件 |
| Traceability | PASS | requirement→task→file→test 完整 |
| Verification | PASS | 文档字段、链接、计数、diff 与 strict validate 可重跑 |
| Unknowns | PASS | EID、affinity、真板结果和硬件规范均有禁止推断分支 |
| Artifact Consistency | PASS | 与 proposal、design、tasks 和 Iteration 000 accepted 基线一致 |
| Persisted Evidence | PASS | Mode none；全部验证可低成本重跑 |
| User Plan Approval | PASS | 用户通过 `$openspec-act 更改gate状态，开始实施` 明确批准 Iteration 001 实施 |

**Persisted Evidence**

- Mode: none

Act Response 足以记录命令、决定性输出和退出码，不创建 Evidence。

**Risks and Notes**

- T3 的 mailbox 细节来自固定 revision 第三方工程；正文必须逐项标 `第三方经验` 或 `未知项`，不能因仓库内注释含“实测”而升级证据。
- `MBX3` 是历史变量名而非物理 mailbox3；这是最容易造成双向链错误的命名陷阱。
- AP source 217、RP source 69 与 IMSIC EID 属不同编号空间；当前没有 EID 映射证据。

## Act Response

- Status: reported

**Implemented**

- T3: 创建并返修 `docs/interrupts/com260-mailbox-notification.md`（187 行）。文档区分物理 mailbox4 与历史变量 `MBX3`，记录 AP→RP channel 0 / PLIC source 69、RP→AP channel 1 / APLIC source 217 两条固定 revision 第三方通知链，并把 IMSIC EID 保留为未知项。
- T3 Review 返修: 保留 `第三方经验` 作为来源层，为直接观察补充 `未知项`，为跨文件链路和风险结论补充带依据的 `推论`；没有把第三方材料升级为 `官方事实` 或 `交叉验证`。
- T3: 记录初始化 drain/clear/enable、ISR 排空 FIFO→清 pending→重读 `RAW & EN`、AP 32×64 工作上限、单槽 waker、多等待者覆盖风险，以及 boot/ioctl 自测 3×100000 spin 的能力与未运行边界。
- T4: 仅改写 `docs/reference/known-gaps.md` G4；用 Iteration 000 的静态地址与 topology 将状态从 `open` 调整为 `partial`，保留寄存器、source→EID、target hart、affinity 和真板 delivery 缺口。G1-G10 总数仍为 10，未新增同义缺口。
- T5: 更新 `docs/index.md`，加入两篇 interrupts 正文入口，将主题标为已聚合并保留 G4 partial；来源与缺口计数同步为 62 URL / 10 gaps。
- `tasks.md`: T3-T5 在 Gate 5 通过后标记完成；T1-T2 状态不在本 Cycle 修改。

**Changed Files and Sections**

- `docs/interrupts/com260-mailbox-notification.md`: 新增全文。
- `docs/reference/known-gaps.md`: G4。
- `docs/index.md`: 参考文档计数、interrupts 入口、主题职责与维护规则计数。
- `openspec/changes/establish-k3-com260-interrupt-time-notification-baseline/tasks.md`: T3-T5 checkbox。
- 本 Cycle：记录用户 Gate 2 批准并填写 Act Response。

**Deviations from Plan**

- None。没有新增来源 URL、refresh、Evidence、产品代码或 `others/` 修改。

**Gate 3 Test Witness**

| Task | RED witness | Result |
| --- | --- | --- |
| T3 | `test ! -e docs/interrupts/com260-mailbox-notification.md` | PASS：目标文件不存在 |
| T4 | G4 段不含 `partial` | PASS：状态尚未收敛 |
| T5 | `docs/index.md` 不含 `com260-mailbox-notification.md` | PASS：目标入口不存在 |

**Gate 4 Self-Review**

- Spec compliance: PASS。T3-T5 的 required behavior、required fields、Preserve、Forbidden 和 GREEN condition 均满足；首次 Review 指出的“来源层代替四级证据”问题已修复。
- Document quality: PASS。正文区分来源层、编号空间和未知项；索引不复制技术细节；G4 只记录已解除与仍未知部分。
- Full diff: PASS。目标产品文件与 change 内任务/Cycle 之外没有新增修改；`others/` 保持只读。
- Critical findings unresolved: 0。
- Important findings unresolved: 0。
- Minor findings unresolved: 0。

**Verification Evidence**

| 验证项 | 命令或操作 | 决定性输出 | 结论 |
| --- | --- | --- | --- |
| T3 来源与内容 | Python 检查首行、字段、revision、双向链、未知项和行数 | `sources=3 exact; unknowns=4x4; required_fields=PASS; lines=187` | PASS |
| T3 证据双标记返修 | Python 扫描表格与正文，检查来源层、四级证据及推论依据 | `labels=dual; inference=20; unknown=27; bare_third_party_table_cells=0` | PASS |
| T4 G4 | Python 检查 G 编号、G4 状态/字段/链接 | `gaps=10; G4=partial; fields_and_links=PASS` | PASS |
| T5 入口与计数 | Python 检查链接、62/10 计数和全量 docs 相对链接 | `links=2; source_count=62; gap_count=10; all_docs_relative_links=PASS` | PASS |
| 全量文档 Gate | 综合 Python 检查 | `sources=62; gaps=10; mailbox_lines=187; source_metadata=3/3; unknowns=4x4; relative_links=PASS` | PASS |
| OpenSpec | `openspec validate establish-k3-com260-interrupt-time-notification-baseline --strict` | `Change 'establish-k3-com260-interrupt-time-notification-baseline' is valid` | PASS |
| diff | `git diff --check` 与 `git diff --cached --check` | exit 0 / exit 0 | PASS |

**Persisted Evidence**

None required。验证可低成本重跑，未创建 Evidence 目录。

**Experience Candidates**

None。当前工作是文档聚合和静态验证，没有执行可复用的真板操作，也没有发生 Incident。

**Remaining Issues**

- G4 保持 `partial`：仍缺公开寄存器布局、source→EID、target hart、affinity 与真板双向 delivery 证据。
- 本 Cycle 不声明第三方 boot/ioctl self-test 已运行或通过，不声明 burst 下无丢失/无重复。

**Blocker Handoff**

None。

## Plan Review

- Review Result: accepted

**Findings**

- 无阻塞发现。首次 Review 的证据语义问题已关闭：`第三方经验` 仅作为来源层，直接观察使用 `未知项`，跨文件链路与风险结论使用带依据的 `推论`。
- [minor] `tasks.md` 中已经由 Iteration 000 Review 接受的 T1-T2 仍未勾选。它不阻塞当前 T3-T5 的产品验收，但 change 收尾前需要由相应状态维护流程与已接受结果对齐；本 Cycle 不回写前序 Iteration 的任务状态。

**Deviation Classification**

- RESOLVED ACT-DEVIATION: T3 已补齐来源层与四级证据双标记，未改变事实内容、来源边界或任务范围。

**Acceptance Gaps**

- None。T3-T5、R2/R4/R5 与 D06 的当前 Iteration 验收条件均满足。

**Convergence**

- 首次 Review 的单一 blocker 已在同一 Cycle 有限返修并关闭；复核未发现新的 blocker、范围漂移或返工发散。

**Evidence**

- 独立来源检查：首行 3 个 URL 均已登记；源端修订与观察日期保持精确元数据，覆盖表权威表段为 62 行。
- 独立内容检查：mailbox 文档 187 行，双向链、source 217/69、USER/channel、32×64 drain、3×100000 自测预算、单等待者边界与 4×4 未知项均存在。
- 证据标签扫描：`labels=dual; inference=20; unknown=27; bare_third_party_table_cells=0`；第三方材料未被标为 `官方事实` 或 `交叉验证`。
- 独立一致性检查：G1-G10 共 10 项，G4 为 `partial`；index 的两个 interrupts 链接、62 URL / 10 gaps 和全量 docs 相对链接均通过。
- `openspec validate establish-k3-com260-interrupt-time-notification-baseline --strict`、`git diff --check`、`git diff --cached --check` 均 PASS；Persisted Evidence 为 `none`，无 Evidence 目录不构成问题。

**Follow-up Decision**

- 无后续返修。当前 Cycle 接受；本 change 的所有 Iteration 均已 accepted，可交由 `openspec-docs-maintainer` 执行状态同步与正常收尾。

**Iteration Plan Update**

- None；T3-T5 与两轮 Iteration Map 保持不变。

**Next Cycle**

- None。

**Next Iteration**

- None；本 change 没有后续 Iteration。
