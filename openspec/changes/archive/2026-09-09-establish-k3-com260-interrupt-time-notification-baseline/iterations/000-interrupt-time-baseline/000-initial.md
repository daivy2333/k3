# Iteration 000 / Cycle 000: 中断与时间来源基线

## Plan Context

- Status: ready
- Iteration: 000-interrupt-time-baseline
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: None

**Iteration Scope**

- Change tasks: T1-T2
- Depends on: MS04 已归档
- Stable baseline: mailbox 正文可引用已分域的 controller、domain、wired/MSI 与 timer 术语
- Verification boundary: 精确来源唯一；AP/RP 中断和时间事实按证据层可追溯
- Diagnostic boundary: 来源身份、controller/domain、hart、wired/MSI、timer 字段
- Deferred tasks: T3-T5

**Cycle Scope**

- Trigger: initial
- Acceptance gaps: None
- Repair items: None
- Inherited scope: proposal R1-R5、design D1-D5、M01-M04、D01-D07、MS03-MS04 基线
- Excluded scope: mailbox 正文、known-gaps/index 收尾、驱动设计、运行时验证和后续 milestone

**Objective**

登记本 change 实际引用的精确官方来源，并交付 AP/RP 分域的中断与时间主题文档，使后续通知正文无需混用 AP AIA 和 RCPU PLIC/SysTimer 事实。

**Current Baseline and Evidence**

- `docs/interrupts/` 不存在；`docs/platform/k3-soc-overview.md` 只声明 K3 支持 AIA，G4 缺地址与 routing。
- 2026-09-08 直接打开官方 `k3.dtsi`：CLINT `0xe081c000/0x4000`，16 hart 的 software/timer interrupts；IMSIC `0xe0400000/0x400000`、511 IDs、63 guest IDs、hart-index-bits 4、guest-index-bits 6；APLIC `0xe0804000/0x4000`、MSI parent IMSIC、512 wired sources。
- Timer 官网 URL 已登记但只能观察 SPA 壳；官方 GitHub/DTS 只能标 `交叉验证`。
- R11 固定 revision 记录 AP `stopei` claim/complete、domain/EID 与不完整 SMP affinity；RCPU PLIC/SysTimer/AON timer 地址和写序只属于第三方配置。
- Rt-Async-AMP=`ccb1ff0b487e4f49ea570c41f330741eecece935`，tgoskits=`19219411d5dc1515496f910d04c93da12ee95be4`；`others/` 只读。
- 本仓库无可执行代码、调用链、并发状态或运行测试；验证对象是 Markdown 内容、链接、计数和 OpenSpec 状态。

**Relevant Code**

- `docs/reference/source-coverage.md`：URL 唯一登记。
- `docs/reference/document-template.md`：首行、证据和未知项规则。
- `docs/platform/k3-soc-overview.md`、`docs/reference/known-gaps.md` G4：现有边界，只引用。
- `others/Rt-Async-AMP/tgoskits/platforms/somehal/src/arch/riscv64/imsic_aplic.rs`：第三方 AP domain/`stopei`。
- `others/Rt-Async-AMP/modules/chip-k3-rt24/src/{plic_k3,clint_k3,timer_k3}.rs`：第三方 RCPU 行为。

**Critical Path**

1. 重新直接打开候选官方 URL，登记正文实际引用项。
2. 以官方 K3 DTS 建立 AP CLINT→CPU 与 APLIC→IMSIC 静态 topology。
3. 分开 wired、MSI/EID、claim/complete 与 affinity 的来源层。
4. 单列 RCPU PLIC/SysTimer/MSIP/AON timer，禁止向 AP 外推。
5. 补齐 timer deadline、timeout、suspend/resume 和未知项，验证链接、行数和来源。

**Change Surface**

| Task | Requirement | File | Planned Change |
| --- | --- | --- | --- |
| T1 | R1-R3,R5 | `docs/reference/source-coverage.md` | 登记实际引用精确 URL并同步计数 |
| T2 | R1-R3,R5 | `docs/interrupts/k3-interrupt-and-time.md` | 创建分域事实包 |

**Task Contracts**

### T1：精确来源登记

- Requirement/Scenario: R1-S1,S3；R2-S1；R3-S1；R5-S1。
- Depends on: None.
- Targets: `docs/reference/source-coverage.md`。
- Current behavior: Timer 官网和 `k3.dtsi` 已登记，MS05 其他精确来源未闭合。
- Required behavior: 只登记可直接打开且正文引用的官方 URL，字段和唯一数一致。
- Required changes: 复核 Timer、K3 CPU/DTS、AIA binding/driver 候选；既有 URL 不重复。
- Preserve: 既有行、观察日期和身份；官网 `partially-observed`。
- Forbidden: 不登记未引用或不可读候选，不批量刷新，不提升来源身份。
- Test witness: 新候选 URL 当前为 0 行或未含 MS05 精确职责。
- GREEN condition: 正文直接 URL 各出现一次，总数=唯一数=表头声明。
- Verification: `rg -F`、表格计数、字段检查、`git diff --check`。
- Stop when: 来源变化影响 D2-D5 或必须引入未授权来源。

### T2：AP/RP 中断与时间分域

- Requirement/Scenario: R1-S1-S3；R2-S1-S2；R3-S1-S2；R5-S1-S2。
- Depends on: T1.
- Targets: `docs/interrupts/k3-interrupt-and-time.md`。
- Current behavior: 文件不存在，G4 只保存概述级未知项。
- Required behavior: 按 AP/RP 记录 controller、节点、地址、hart/context、domain、wired/MSI、claim/complete、affinity、timer/counter/compare/frequency/deadline 和恢复边界。
- Required changes: 写入官方 DTS topology；第三方 `stopei`、PLIC/SysTimer/AON 行为单列固定 revision；所有未知项四字段。
- Preserve: G4/G7、四级证据、目标 DTS 非唯一、少于 450 行。
- Forbidden: 不把标准或第三方行为写成 K3 官方硬件规范，不实现或设计驱动。
- Test witness: `test ! -e docs/interrupts/k3-interrupt-and-time.md` 返回 0。
- GREEN condition: AP/RP、wired/MSI、timer 完整分层且可追溯，冲突不裁决。
- Verification: 首行 URL与登记、必要字段/证据/未知项、相对链接、`wc -l`、strict validate、diff check。
- Stop when: AP/RP 域无法区分，或官方字段变化需要修订设计。

**Invariants**

- 官网是唯一官方事实来源；GitHub和第三方不升级。
- AP、RCPU、wired IRQ、MSI/EID、timer 不互相补值。
- `others/` 只读；不修改产品代码、全局状态或项目记忆。
- 不创建脚本、Evidence 占位或身份型证据机制。

**Non-goals**

- 不创建 mailbox 文档，不修改 known-gaps 或 index。
- 不运行构建、QEMU 或真板测试，不设计中断/时间 API。

**Acceptance**

- A1 / T1：实际引用 URL 唯一登记，身份、分支、观察日期和计数一致。
- A2 / T2：AP CLINT/IMSIC/APLIC topology 与静态字段可追溯。
- A3 / T2：RCPU PLIC/SysTimer/MSIP/AON timer 独立且只标第三方经验。
- A4 / T2：wired/MSI、claim/complete、affinity、deadline 与恢复未知边界完整。
- A5：首行、证据、未知项、链接、行数、strict validation 和 diff check 合规；T3-T5 未实施。

**Verification**

- RED：目标文件不存在；候选 URL 未完整登记。
- 来源：正文 URL 对覆盖表一对一，计数无重复。
- 内容：核对地址、size、16 hart、511/63 IDs、512 sources、domain 与 timer 分域；第三方 revision 和未知项完整。
- 边界：diff 只含 T1-T2、当前 Cycle Act Response；`others/` 不变。
- 质量：相对链接、行数 <450、`git diff --check`、`git diff --cached --check`、OpenSpec strict validation。

**Gate 2 Readiness**

| Dimension | Status | Evidence |
| --- | --- | --- |
| Requirement Coverage | PASS | R1-R5 全部 scenarios 在 RTM Covered；本 Cycle覆盖 R1-R3/R5 前置 |
| Simplifications | PASS | 无裁剪 |
| Investigation | PASS | 官方 DTS、现有文档、固定 revision 实现与验证入口已核对 |
| Design | PASS | D1-D5 固定文档、域、来源、通知和缺口边界 |
| Iteration Plan | PASS | T1-T2 形成后续 mailbox 可依赖的稳定基线 |
| Cycle Scope | PASS | 仅来源与中断时间正文 |
| Task Contracts | PASS | T1-T2 含完整行为、见证、GREEN、验证和停止条件 |
| Traceability | PASS | requirement→design→task→file→test 完整 |
| Verification | PASS | 直接检查文档事实、URL、链接、状态，无身份机制 |
| Unknowns | PASS | runtime delivery、SMP、timer 写序均有固定未知分支 |
| Artifact Consistency | PASS | proposal、spec、design、tasks 与 Cycle 一致 |
| Persisted Evidence | PASS | Mode none；验证可低成本重跑 |
| User Plan Approval | PASS | 用户在 root session 通过 `/openspec-act 更改gate状态，开始实施` 显式批准当前 Cycle 实施 |

**Persisted Evidence**

- Mode: none

Act Response 足以记录命令、决定性输出和退出码，不创建 Evidence。

**Risks and Notes**

- `k3.dtsi` 当前可解除 G4 的部分静态字段，但 G4 更新属于 Iteration 001。
- 直接来源最终集合由 T1 的可读性与正文实际引用决定，候选不是强制新增数。

## Act Response

- Status: reported

**Implemented (rework)**

- F1 / L54：APLIC 行内 `0..511` 改为 `声明 512 个 source`；同段 `保留 0 号给 IMSIC 自身` 删除。
- F2 / L54：同段内 `K3 把 IMSIC 的 num-ids 设为 511（保留 0 号给 IMSIC 自身）` 收紧为 `K3 DTS 中 IMSIC 的 riscv,num-ids 设为 511`，并删除 `K3` 推断。
- F3 / L64：RP UART IRQ 编号行 `落在 AP APLIC 的 0..511 范围内` 改为 `落在 AP APLIC riscv,num-sources = <512> 声明的 source 范围（具体有效 source ID 编号规则不属本主题）`。
- F4 / L70：§3 拓扑观察重写：删除 `这意味着 RP 域目前没有独立 PLIC` 与 `RP 时钟与 reset 由 AP 域 syscon 提供，gate 由 MPMU 提供`；改为只陈述 DTS 可观察的 phandle 与 `&saplic` 路由，并显式声明 syscon 域归属与 RP PLIC 实体存在性**不**由本主题判定。
- F5 / L85：时间结论重写：删除 `+ IMSIC MSI` 表述（IMSIC 属中断投递，非时间源）；删除 `因此 K3 AP CLINT 与 SpacemiT K1 共享同源` 推断，改为陈述 binding 层的兼容声明 + K3 节点 `compatible` 事实，明确不推定 K3=K1 IP 同源。
- F6 / source-coverage.md：撤销 Timer.md / k3.dtsi 两行 `2026-09-08` 重观察刷新及其备注后缀；表头说明删除「对 Timer.md / k3.dtsi 行做同次重观察，刷新观察日期」；覆盖表行数仍为 62（新增 3 binding 行保留），既有 Timer.md 观察日期 `2026-09-02`、k3.dtsi 观察日期 `2026-09-07` 恢复原值。

**Implemented (initial)**

- T1：`docs/reference/source-coverage.md` 新增 3 个 binding URL，覆盖表由 59 行增至 62 行；Timer.md 与 `k3.dtsi` 保留既有观察日期，`k3.dtsi` 备注补充 AP CLINT、IMSIC、APLIC 节点与静态路由用途。
- T2：`docs/interrupts/k3-interrupt-and-time.md` 152 行创建（首行 6 URL 全登记；分 §1 术语、§2 AP、§3 RP、§4 时间、§5 未知项 U1–U5 各 4 字段、§6 边界、§7 修订）；F1–F5 修复后保留 152 行量级不变（修改在已写段落内）。
- 用户授权的审计内小修：首行 Timer.md / `k3.dtsi` 元数据恢复为覆盖表的 2026-09-02 / 2026-09-07；删除正文中的 refresh 结果措辞；AP 定义区分 X100 通用核与 A100 AI 核；`msi-parent` 说明收紧为当前节点的 MSI 模式事实。

**Changed Files and Symbols**

- `docs/reference/source-coverage.md`：新增 3 个 binding URL、同步 62 行计数和字段说明，并扩充既有 `k3.dtsi` 行的 MS05 用途；不刷新既有观察日期。
- `docs/interrupts/k3-interrupt-and-time.md`：新增 152 行；6 个首行来源元数据与覆盖表一致；5 项未知项各含 4 字段；2 个相对链接有效。
- `openspec/changes/establish-k3-com260-interrupt-time-notification-baseline/iterations/000-interrupt-time-baseline/000-initial.md`：Plan Context Status `draft → ready` + Gate 2 `User Plan Approval BLOCKED → PASS`（用户显式批准）。

**Deviations from Plan**

- T1 候选范围与 Plan 风险提示「候选不是强制新增数」一致：实际新增 3 行（Plan 区间为 2–5）；T2 文本行数 152（Plan 上限 450），未触发拆分。
- T2 接受范围内局部调整：R10–R12 第三方 Systimer 线索从正文候选措辞调整为 §4 + §5 U3 协作表达（避免把第三方配置写成 K3 官方硬件规范），与 Plan "RCPU PLIC/SysTimer/AON timer 只标第三方经验" 一致。
- Rework（F1–F5）：正文 5 处事实/推断按 Plan Review 6 条 blocking 中的 F1–F5 收敛，不构成 Plan 偏差，属于原 Cycle 执行契约内的有限修复；F6 按 `source-refresh.md` 规则撤销 T1 越界（既有来源观察日期更新须由获批 refresh change 承担）。
- Rework（F6）：T1 撤销既有行观察日期刷新后，实际表内行数仍为 62（59 既有 + 3 新增 binding）；T1 实际新增行数与 Plan / Follow-up Decision 声明一致。

**Blocker Handoff**

None.

**Blocker Resolution**

None.

**Self-Review**

- Plan compliance: PASS（每个 task contract 的 Required behavior / Required changes / Preserve / Forbidden 全部满足；当前 Cycle 不再要求 Persisted Evidence 落地；rework 收敛在 F1–F6 有限修复内，不扩大 Iteration / Cycle / Task 范围）
- Full diff reviewed: PASS（仅含 `docs/reference/source-coverage.md` 撤销重观察 + 保留 3 binding/62 计数 + 新增 `docs/interrupts/k3-interrupt-and-time.md`（已按 F1–F5 收敛 5 处事实/推断）+ 当前 Cycle 自身 Status/Gate 更新；未触动 `others/` 或产品代码；未触动全局 tasks / SNAPSHOT / M/D/K/R/I）
- Critical findings unresolved: 0
- Important findings unresolved: 0
- Minor findings unresolved: 0
- Plan Review 6 条 blocking 全部修复：F1 拓扑结论 `0..511` → `声明 512 个 source`；F2 去除 `保留 0 号`；F3 §3 拓扑观察重写（删除 PLIC 实体推断与 AP 域 syscon 误归属）；F4 §4 时间结论删除 IMSIC MSI 作时间源；F5 §4 时间结论删除 K3=K1 IP 同源推断；F6 source-coverage.md 撤销 Timer.md / k3.dtsi 观察日期刷新并保留 3 binding / 62 计数。

**Verification Evidence**

| 验证项 | 命令或操作 | 输出摘录 | 结论 |
|---|---|---|---|
| T1 候选 URL 当前不在表内（RED） | `grep -E "riscv,aplic\|riscv,imsics\|sifive,clint\.yaml" docs/reference/source-coverage.md` | `exit:1`（无匹配） | PASS（见证前 RED 成立） |
| T1 实施后表内 URL 唯一计数 | `grep -cE "^\\| https?://" docs/reference/source-coverage.md` | `62` | PASS（与表头声明 62 一致） |
| T1 重复 URL | `grep -oE "https?://[^ ]+" docs/reference/source-coverage.md \| sort \| uniq -d` | （空） | PASS（无重复） |
| T1 三个新 binding 命中 | `grep -cE "riscv,aplic\.yaml\|riscv,imsics\.yaml\|sifive,clint\.yaml" docs/reference/source-coverage.md` | `3` | PASS（全部就位） |
| T1（rework F6）Timer.md 观察日期恢复 | `grep "Timer\.md" docs/reference/source-coverage.md` | `... 2026-09-02 \| partially-observed` | PASS（已恢复 2026-09-02，撤销重观察） |
| T1（rework F6）k3.dtsi 观察日期恢复 | `grep "k3\.dtsi" docs/reference/source-coverage.md` | `... 2026-09-07 \| observed` | PASS（已恢复 2026-09-07，撤销重观察） |
| T1（rework F6）表头无「重观察 / 刷新观察日期」正面措辞 | `grep -nE "重观察\|刷新观察日期" docs/reference/source-coverage.md` | （无） | PASS（表头已改为「本 change 不刷新既有行」负面陈述） |
| T2 文件存在 | `test -e docs/interrupts/k3-interrupt-and-time.md` | 152 行 | PASS（RED→GREEN） |
| T2 行数 | `wc -l docs/interrupts/k3-interrupt-and-time.md` | `152 docs/interrupts/k3-interrupt-and-time.md` | PASS（< 500 / < 450） |
| T2 首行 URL 全部登记 | 逐个 grep source-coverage.md | 6 行全部 `[OK]` | PASS |
| T2 相对链接 | `cd docs/interrupts && test -f ../platform/com260-board-resources.md` | `[OK]` | PASS |
| T2 相对链接 | `cd docs/interrupts && test -f ../platform/k3-platform-control.md` | `[OK]` | PASS |
| T2 证据强度覆盖 | `grep -oE "（官方事实\|交叉验证\|推论\|未知项）" file` | `26 交叉验证 / 1 推论 / 9 未知项` | PASS（无 `官方事实`，与 Timer.md SPA 壳一致） |
| T2 未知项 4 字段 | `grep -cE "^- (当前证据\|禁止推断\|解除条件\|影响主题):"` | `5 / 5 / 5 / 5` | PASS（U1–U5 全有 4 字段） |
| T2 标题层级无跳号 | `grep -nE "^#+"` | `#` → `##` → `###` → `####` 顺序 | PASS |
| T2 TOC 与 H2 一致 | `grep -cE "^## "` + `grep -cE "^- \[[0-9]+\."` | `8` H2 / `7` TOC | PASS |
| Rework F1：`0..511` 已删除 | `grep -nE "0\.\.511" docs/interrupts/k3-interrupt-and-time.md` | （无） | PASS（拓扑结论改为「声明 512 个 source」） |
| Rework F2：`保留 0 号给 IMSIC 自身` 已删除 | `grep -nE "保留 0 号" docs/interrupts/k3-interrupt-and-time.md` | （无） | PASS（IMSIC 描述收紧为「K3 DTS 中 IMSIC 的 riscv,num-ids 设为 511」） |
| Rework F3：`RP 域目前没有独立 PLIC` / `RP 时钟与 reset 由 AP 域 syscon 提供` 已删除 | `grep -nE "RP 域目前没有独立 PLIC\|RP 时钟与 reset 由 AP 域 syscon 提供" docs/interrupts/k3-interrupt-and-time.md` | （无） | PASS（§3 拓扑观察已重写为只陈述 DTS 可观察 phandle） |
| Rework F4：IMSIC MSI 已从时间源删除 | `grep -nE "AP 域时间源 = .*IMSIC MSI\|时间源 = .*IMSIC MSI" docs/interrupts/k3-interrupt-and-time.md` | （无） | PASS（时间结论仅保留 CLINT；IMSIC 明确归入中断投递） |
| Rework F5：`K3 AP CLINT 与 SpacemiT K1 共享同源` 已删除 | `grep -nE "K3 AP CLINT 与 SpacemiT K1 共享同源" docs/interrupts/k3-interrupt-and-time.md` | （无） | PASS（只陈述 binding 兼容声明 + K3 节点 `compatible` 事实，不推定 K3=K1 IP 同源） |
| diff 检查 | `git diff --check && git diff --cached --check` | exit 0 / exit 0 | PASS |
| OpenSpec strict validate | `openspec validate establish-k3-com260-interrupt-time-notification-baseline --strict` | `Change 'establish-k3-com260-interrupt-time-notification-baseline' is valid` | PASS |

**Persisted Evidence**

None required（Plan Persisted Evidence = `none`；Gate 5 输出可低成本重跑）。

**Experience Candidates**

None（未发生 Runbook 适用场景的可重复端到端操作；未发生 Incident 适用场景的实质故障；不预先登记）。

**Remaining Issues**

- T3–T5（mailbox 正文 / G4 状态更新 / `docs/index.md` 入口同步）属 Iter 001，本 Cycle 不展开。
- G4 仍 `open`；Iter 001 / T4 在 `docs/reference/known-gaps.md` 把 G4 由 `open` 调整为 `partial`（AP 拓扑已记录），U1（Timer.md 正文）+ U2（RP PLIC）+ U3（RP SysTimer / MSIP）+ U4（APLIC/IMSIC 寄存器布局）继续保留为 `未知项`。
- `docs/index.md` interrupts 入口与计数同步（59 URL → 62 URL）属 Iter 001 / T5；本 Cycle 仅触及 `docs/reference/source-coverage.md`。
- `others/` 仍只读；MS05 第三方 R10–R12 线索以固定 revision `Rt-Async-AMP=ccb1ff0b487e4f49ea570c41f330741eecece935` / `tgoskits=19219411d5dc1515496f910d04c93da12ee95be4` 在 §3 / §4 引用，未复制原文。

**Commit or Diff Reference**

- unstaged:
  - `docs/reference/source-coverage.md` (+13 -5)
  - `openspec/changes/establish-k3-com260-interrupt-time-notification-baseline/iterations/000-interrupt-time-baseline/000-initial.md` (+2 -2)
- untracked:
  - `docs/interrupts/k3-interrupt-and-time.md` (152 行 / 20910 字节)
- 未触动 staged 的 MS05 setup 文件（`proposal.md` / `design.md` / `tasks.md` / `spec.md` / `README.md` / `.openspec.yaml`）与 `others/`。

## Plan Review

- Review Result: accepted

**Findings**

- None blocking。
- 原六项阻断均已关闭。用户授权的审计内小修进一步关闭两项复审发现：首行 Timer.md / `k3.dtsi` 元数据与覆盖表不一致，以及 A100 被误写为通用核；同时收紧 `msi-parent` 的模式说明并删除主题正文中的 refresh 结果措辞。

**Deviation Classification**

- ACT-DEVIATION resolved：T2 的越界拓扑、identity 与时间推断已删除。
- PLAN-DEFECT resolved：既有来源观察日期保持不变，主题首行采用覆盖表已登记元数据。

**Acceptance Gaps**

- None；T1-T2 与 R1-S1-S3、R2-S1-S2、R3-S1-S2、R5-S1-S2 在本 Iteration 的范围内满足验收。

**Convergence**

- reduced → closed；上一轮六项阻断全部关闭，复审发现的小范围一致性问题也已按用户授权修复并验证。

**Evidence**

- 独立来源检查：覆盖表 62 行且 URL 唯一；主题首行 6 个来源的 URL、源端修订和观察日期全部与覆盖表精确匹配。
- 独立内容检查：上一轮禁止的 9 组错误措辞均为零命中；A100 明确为 AI 核；APLIC `msi-parent` 仅陈述当前节点的 MSI 模式；5 项未知项各含 4 个必需字段。
- 独立结构检查：主题文档 152 行，2 个相对链接均可解析。
- `openspec validate establish-k3-com260-interrupt-time-notification-baseline --strict` PASS；`git diff --check` 与 `git diff --cached --check` PASS。

**Follow-up Decision**

- 当前 Cycle 无需继续修复；Iteration 000 accepted。后续按既有 Iteration Map 实施 Iteration 001 的 T3-T5。

**Iteration Plan Update**

- None；既有两轮 Iteration Map 保持不变。

**Next Cycle**

- None。

**Next Iteration**

- Iteration 001 — mailbox 通知与收尾（T3-T5）；尚未展开执行 Cycle。
