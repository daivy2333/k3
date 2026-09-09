## 1. Iteration 000 — 中断与时间来源基线

- [x] 1.1 [T1] 在 `docs/reference/source-coverage.md` 登记正文实际引用且已直接打开的 Timer、K3 CPU/DTS、AIA binding/driver 精确 URL，并同步唯一 URL 数。
- [x] 1.2 [T2] 创建 `docs/interrupts/k3-interrupt-and-time.md`，分域整理 AP CLINT/AIA/APLIC/IMSIC 与 RCPU PLIC/SysTimer/MSIP/AON timer。

## 2. Iteration 001 — mailbox 通知与收尾

- [x] 2.1 [T3] 创建 `docs/interrupts/com260-mailbox-notification.md`，整理 AP/RP mailbox、FIFO/pending、source/EID、自测和通知/数据边界。
- [x] 2.2 [T4] 复核 `docs/reference/known-gaps.md` G4；按实际官方证据更新为 `partial` 或保持 `open`，仅在不重复时新增缺口。
- [x] 2.3 [T5] 更新 `docs/index.md` 的 interrupts 入口和状态，并按权威文件同步实际来源与缺口计数。

## Task Contracts

### T1：精确来源登记

- Requirement/Scenario: R1-S1,S3；R2-S1；R3-S1；R5-S1。
- Depends on: None.
- Targets: `docs/reference/source-coverage.md`。
- Current behavior: Timer 官网 URL 已登记；`k3.dtsi` 已登记但备注未覆盖 AIA；CPU DTSI、AIA binding/driver 等精确 URL尚未按 MS05 登记。
- Required behavior: 只登记可打开且正文实际引用的 URL；同步总数、唯一数、身份、分支和观察日期。
- Preserve: 既有行及身份；官网 `partially-observed`；M01、D04-D06。
- Forbidden: 不登记未引用候选，不刷新无关行，不把 GitHub 标为官方事实。
- Test witness: 候选精确 URL 当前未登记，目标正文不存在。
- GREEN condition: 正文全部直接 URL 各登记一次，计数一致，无重复。
- Verification: URL 精确计数、表格计数、字段检查、diff check。
- Stop when: 来源变化影响域模型或需非授权来源。

### T2：AP/RP 中断与时间分域

- Requirement/Scenario: R1-S1-S3；R2-S1-S2；R3-S1-S2；R5-S1-S2。
- Depends on: T1.
- Targets: `docs/interrupts/k3-interrupt-and-time.md`。
- Current behavior: 文件和目录不存在；事实散落在 SoC 概述、DTS 与第三方分析。
- Required behavior: 记录 AP CLINT/IMSIC/APLIC 静态拓扑、wired/MSI 字段和 RCPU PLIC/SysTimer/MSIP/AON timer；分开官方 DTS 与第三方行为。
- Required changes: 包含地址、reg size、hart、IDs/sources、parent/domain、claim/complete、affinity、counter/compare/frequency/deadline 与 suspend/resume 未知边界。
- Preserve: G4、G7；AP/RP 分域；四级证据；少于 450 行。
- Forbidden: 不把 `stopei`、RP 写序或第三方 DTS 数值提升为硬件规范，不设计驱动。
- Test witness: interrupts 正文不存在，G4 缺静态 topology。
- GREEN condition: AP/RP、wired/MSI、timer 均可追溯，冲突和未知项未裁决。
- Verification: 首行来源、必要字段/域/证据、未知项、链接、行数、strict validate。
- Stop when: 官方 DTS 与调查基线实质变化，或 AP/RP 无法区分。

### T3：mailbox 通知链

- Requirement/Scenario: R4-S1-S4；R2-S2；R5-S1-S2。
- Depends on: Iteration 000 accepted.
- Targets: `docs/interrupts/com260-mailbox-notification.md`。
- Current behavior: 文件不存在；R10 只作为分析入口。
- Required behavior: 记录物理 mailbox、AP/RP user/channel、FIFO/pending、IRQ source/EID、排空/清除/重读、自测和预算；通知只促使重查数据。
- Preserve: 固定 revisions；channel 命名空间分离；无真板结论。
- Forbidden: 不声称 self-test 已通过、无丢中断或无重复通知，不展开 RPC/共享内存数据面。
- Test witness: 目标文件不存在，无产品级通知关系图。
- GREEN condition: 双向链、残留/重复分支、自测能力边界和错误缺口完整。
- Verification: 来源、链路、字段、revision、四字段未知项、链接、行数。
- Stop when: mailbox 物理实例或 user/channel 证据冲突且无法分层。

### T4：G4 与新缺口不重复

- Requirement/Scenario: R5-S2。
- Depends on: T2-T3.
- Targets: `docs/reference/known-gaps.md`。
- Current behavior: G4 open，尚未纳入官方 `k3.dtsi` 的静态地址/topology。
- Required behavior: 按新官方证据更新 G4 已解除部分；仍缺 delivery/claim/affinity 时不得 closed。新缺口须四字段独立。
- Preserve: G1-G10 其他状态和措辞，无证据不改。
- Forbidden: 不凭第三方代码关闭 G4，不新增同义 mailbox/timer 缺口。
- Test witness: G4 为 open 且当前证据未列新 DTS 字段。
- GREEN condition: G4 状态与正文一致；新增项不重复；计数同步。
- Verification: G 编号、状态、四字段、语义比较、diff check。
- Stop when: 新证据改变 milestone 范围或下游契约。

### T5：入口与计数一致

- Requirement/Scenario: R5-S3。
- Depends on: T2-T4.
- Targets: `docs/index.md`。
- Current behavior: interrupts 待聚合且无链接；入口中的来源/缺口计数落后于权威文件。
- Required behavior: 链接两篇实际正文，标为已聚合；来源和缺口计数与覆盖表/known-gaps 一致。
- Preserve: 其他主题职责和状态。
- Forbidden: 不复制技术事实，不改变后续 milestone 状态。
- Test witness: 两个链接不存在，interrupts 为待聚合。
- GREEN condition: 链接可解析，状态和计数一致，无关行不变。
- Verification: 链接解析、计数对照、scoped diff、diff check。
- Stop when: T2-T4 未 GREEN。

## Iteration Map

| Iteration | Tasks | Outcome | Stable baseline | Verification boundary | Diagnostic boundary | Non-goals | Balance |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 000 — 中断与时间来源基线 | T1-T2 | 精确来源和 AP/RP 中断时间事实包 | mailbox 文档可引用稳定 domain/术语 | URL、域、topology、wired/MSI、timer、证据 | 来源、controller/domain、timer | mailbox、gaps、index | 来源与核心正文强依赖，2 文件 |
| 001 — mailbox 通知与收尾 | T3-T5 | 通知正文、缺口和入口一致 | MS05 可供后续 milestone 引用 | 双向通知、清除、自测、G4、链接、计数 | mailbox、gap、导航 | 数据面、驱动实现 | 3 文件共享最终一致性边界 |

## Requirements Traceability Matrix

| Requirement | Scenario | Design | Task | Iteration | Code Surface | Test Witness | Simplification | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| R1 | AP/RP domain 与 claim | D2,D3 | T1,T2 | 000 | coverage; interrupt doc | 文档不存在 | None | Covered |
| R2 | wired/MSI 与 affinity | D3 | T2 | 000 | interrupt doc | domain 表不存在 | None | Covered |
| R3 | timer/deadline | D2,D3 | T2 | 000 | interrupt doc | timer 分域不存在 | None | Covered |
| R4 | 双向 mailbox/残留/self-test | D4 | T3 | 001 | mailbox doc | 通知关系图不存在 | None | Covered |
| R5 | 来源、G4、入口 | D2,D5 | T1,T4,T5 | 000,001 | reference; index | URL/状态/链接检查 | None | Covered |

## Plan Completeness Review

- TBD/TODO: None.
- Requirement simplification: None.
- 五项 requirement 的全部 scenarios 均映射到 design、task、文件和验证。
- 两个 Iteration 各形成独立稳定基线，依赖有序；Persisted Evidence 为 `none`。
