# Iteration 000 / Cycle 001：来源边界与资源证据等级返工

## Plan Context

- Status: ready
- Iteration: 000-gmac-phy-baseline
- Cycle: 001-replan
- Cycle Type: replan
- Parent cycle: `000-initial.md`
- Gate 2: 用户于 2026-09-09 在根会话中给出原话 `批准，开始实施`，显式授权本 replan 进入 Act。

**Iteration Scope**

- Change tasks: T1、T2
- Depends on: Cycle 000 Plan Review `replan-required`
- Stable baseline: 第一篇正文的全部直接官方来源在 coverage 唯一登记；资源矩阵不把固定 revision 第三方值提升为官方 GitHub 交叉验证。
- Verification boundary: 第一篇来源反向核对、MMIO/IRQ 证据等级、文内一致性和原 T2 全部 Acceptance 可直接审计。
- Diagnostic boundary: T1 隔离第一篇来源覆盖；T2 隔离资源事实及证据等级。
- Deferred tasks: T3-T6

**Cycle Scope**

- Trigger: replan-required
- Acceptance gaps: 原 T1 不可在第一篇正文 Cycle 内证明两篇来源完整；T2 的 MMIO/IRQ 精确值发生证据等级提升。
- Repair items: 按修订后的 T1 重验第一篇来源；修正 T2 的 MMIO/IRQ 证据等级并重跑全部验收。
- Inherited scope: 已批准的 R1-R3、R5 板级场景，D1-D7，69/69 URL 基线，原 T2 文档边界，无 Persisted Evidence。
- Excluded scope: T3-T6、第二篇正文、缺口/index 收尾、驱动实现、真板、EtherCAT/TSN/网络栈。

**Objective**

让 Iteration 000 的来源任务只验收已存在的第一篇正文，并使 `eth1` 资源矩阵对精确 MMIO/IRQ 的标签与当前可追溯证据一致。

**Current Baseline**

- `source-coverage.md` 有 69 个 URL 且 69 个唯一值；第一篇正文的直接官方 URL 已能逐项精确匹配。
- `com260-gmac-phy.md` 已创建且 238 行，原 Cycle 的链接、字段、边界和 strict validate 均通过。
- §4.1 将 `0xcac82000/0x2000` 和 AP APLIC source 133 标为“交叉验证”；§4.2 对同一值标为“固定 revision 第三方”。
- 当前环境无法直接重开候选官方 GitHub 原始文件；不得以访问失败补写新的官方支撑。
- 全局任务已把 T1 收敛到第一篇正文，并新增 T3 负责第二篇正文来源；原数据面与收尾任务顺延为 T4-T6。

**Relevant Files**

- `docs/reference/source-coverage.md`：T1 只读或精确增量目标。
- `docs/network/com260-gmac-phy.md`：T2 证据标签修正目标。
- `tasks.md`：已修订的全局任务契约；Act 只按其执行，不再改计划。
- `000-initial.md`：父 Cycle 的 Act Response 与 Review 证据。

**Critical Path**

按修订后的 T1 反向核对第一篇来源 → 修正 MMIO/IRQ 标签 → 检查 §4.1/§4.2 一致性 → 重跑 T2 全部 Acceptance → full diff review。

**Change Surface**

| Task | Target | Planned change |
| --- | --- | --- |
| T1 | `docs/reference/source-coverage.md` | 仅核对第一篇正文的直接官方 URL；无实际新增来源则保持文件不变 |
| T2 | `docs/network/com260-gmac-phy.md` | 将 MMIO 与 IRQ 的证据等级改为固定 revision 第三方，并保持官方支撑边界明确 |

**Task Contracts**

### T1：第一篇正文来源重验

- Requirement/Scenario: R1/S1-S3；R2/S4-S6；R5/S13-S14。
- Depends on: None。
- Targets: `docs/reference/source-coverage.md`。
- Current behavior: 覆盖表为 69/69，父 Cycle 已证明第一篇正文直接 URL 可精确匹配，但旧 GREEN 错误要求尚不存在的第二篇正文。
- Required behavior: 只对 `com260-gmac-phy.md` 的全部直接官方 URL 反向核对；新 URL 仍须可直接打开且被正文实际引用才登记。
- Required changes: 无新增实际来源时不改 coverage；在 Act Response 记录第一篇 URL 清单匹配结果和 69/69 唯一性。
- Preserve: 既有 URL 身份、等级、日期和状态；第二篇来源责任留给 T3。
- Forbidden: 不预登记第二篇候选，不因 DNS 失败登记 URL，不修改 `others/`。
- Test witness: 父 Cycle 只证明第一篇来源，正好对应修订后的 GREEN。
- GREEN condition: 第一篇全部直接官方 URL 各在覆盖表出现一次，实际总数等于唯一数；无未引用新增行。
- Verification: 首行 URL 反向核对、URL 总数/唯一数、scoped diff、`git diff --check` 和 strict validate。
- Stop when: 第一篇出现未登记且无法直接打开的必要官方来源，或来源变化改变 D1-D7。

### T2：资源证据等级修正与全量重验

- Requirement/Scenario: R1/S1-S3；R2/S4-S6；R3/S7-S9；R5/S13-S15。
- Depends on: T1。
- Targets: `docs/network/com260-gmac-phy.md`。
- Current behavior: §4.1 的 MMIO/IRQ 行标为“交叉验证”，§4.2 对相同精确值标为“固定 revision 第三方”。
- Required behavior: 在没有可直接核对的官方 SoC DTS 精确字段时，MMIO `0xcac82000/0x2000` 与 IRQ 133 均标为“固定 revision 第三方”；文字不得暗示这些精确值已由官方 DTS 独立确认。
- Required changes: 精准修改 §4.1 两行及必要的引导句；保持 §4.2 的适用对象和未知边界。
- Preserve: 其余已通过的 SoC/模组/变体、MDIO/PHY/RGMII、reset/delay-line、未知项、链接和 ≤450 行边界。
- Forbidden: 不删除精确值，不伪造官方来源，不扩展到 DMA/IRQ handler，不修改 G3-G7 或 index。
- Test witness: 同文 §4.1 与 §4.2 当前对同一值的标签不一致。
- GREEN condition: MMIO/IRQ 精确值全篇均为固定 revision 第三方，官方节点关系与第三方值清晰分开；原 T2 其余 Acceptance 继续通过。
- Verification: 定向检索精确值及相邻证据标签、章节/矩阵、未知项、相对链接、行数、scoped diff、`git diff --check` 和 strict validate。
- Stop when: 可直接打开的新官方来源证明值或拓扑不同，或修正需要改变 design/Acceptance。

**Invariants**

- 官网仍是唯一权威入口；官方 GitHub 只能交叉验证；固定 revision 第三方材料不提升等级。
- 第一篇与第二篇来源登记由 T1、T3 分别承担；每项只在对应正文可确定时验收。
- G3-G7、目标 DTS 未映射和真板未验证边界不变。
- 不修改 SNAPSHOT、全局 tasks、M/D/K/R/I 或 `others/`。

**Acceptance**

1. 第一篇正文全部直接官方 URL 在 coverage 各出现一次，URL 总数等于唯一数。
2. `0xcac82000/0x2000` 与 IRQ 133 在正文中只按固定 revision 第三方证据使用，§4.1 与 §4.2 无等级矛盾。
3. 官方 `eth1` 节点关系与第三方精确值分开表述，不扩大 G4/G6/G7 的已知范围。
4. 原 T2 的分层、静态链、未知项、链接和行数验收继续通过。
5. scoped diff 只含目标产品文件；`git diff --check` 与 strict validate 通过。

**Verification**

- URL 行总数与唯一数比较，并把第一篇首行直接 URL 逐项反查 coverage。
- `rg -n '0xcac82000|0x2000|source 133|固定 revision 第三方|交叉验证' docs/network/com260-gmac-phy.md` 后人工核对适用对象。
- 重跑父 Cycle 的 T2 字段、未知项、相对链接和行数检查。
- `git diff --check`。
- `openspec validate establish-k3-com260-gmac-network-baseline --strict`。

**Gate 2 Approval**

已获用户明确批准；本 Cycle 可进入 Act。

## Act Response

- Status: reported

**Implementation Summary**

- T1：按修订契约反向核对 `com260-gmac-phy.md` 首行的 9 个直接 URL；全部在 `source-coverage.md` 精确出现，覆盖表保持 69 行/69 唯一值，因此没有产品文件修改。
- T2：修改 `com260-gmac-phy.md` §4.1，明确官方 DTS 只支撑节点与 provider 关系；`0xcac82000/0x2000` 和 AP APLIC source 133 只由固定 revision 第三方 IFX DTS 支撑。
- T2：将 MMIO、IRQ 两行的证据等级从“交叉验证”改为“固定 revision 第三方”，与 §4.2 及未知项 U4/U5 保持一致。

**Files Changed**

- `docs/network/com260-gmac-phy.md`：一处边界说明和两处证据标签。
- `docs/reference/source-coverage.md`：SKIPPED: 9 个直接 URL 已全部唯一登记，无实际增量。
- 本 Cycle 文件：记录 Gate 2 授权和本 Act Response。

**Deviations from Plan**

- None。首次 T1 见证命令把 URL 后的中文全角括号说明一并捕获，产生 9 个假缺失；将提取边界收紧到 `（` 前后，等价验证得到 9/9 匹配。该调整没有改变 Task Contract 或测试策略。

**Self-Review**

- Plan compliance: PASS。只执行 T1 来源重验和 T2 证据等级修正；T3-T6、G3-G7、index、`others/` 和全局状态均未实施。
- Spec compliance review: PASS。第一篇全部直接 URL 已覆盖；精确 MMIO/IRQ 仅以固定 revision 第三方证据使用；官方节点/provider 关系与第三方数值已分开。
- Code quality review: PASS。文档仍为 238 行，相对链接有效，术语和四级证据边界未扩张；无计划外产品修改。
- Full diff review: PASS。当前工作区存在进入本 Cycle 前的 MS06 和其他未提交修改；本 Cycle 的产品改动仅涉及 `docs/network/com260-gmac-phy.md` 上述三处，未覆盖既有改动。
- Critical findings unresolved: 0。
- Important findings unresolved: 0。
- Minor findings unresolved: 0。

**Verification Evidence**

| 验证项 | 命令或操作 | 决定性输出 | 退出码 / 结论 |
| --- | --- | --- | --- |
| Gate 3 / T1 | coverage 总数/唯一数并反查第一篇首行 URL | `url_total=69 url_unique=69 missing=none`；`T1_DIRECT_URLS=9` | 0 / GREEN baseline |
| Gate 3 / T2 | 检查 MMIO、IRQ 行的原证据等级 | `MMIO => 交叉验证`；`IRQ => 交叉验证` | 1 / 预期 RED |
| T1 GREEN | 收紧 URL 提取边界后重跑反查 | `T1_GREEN total=69 unique=69 direct=9 missing=none` | 0 / PASS |
| T2 GREEN | 检查两行第三列证据等级 | `MMIO=固定 revision 第三方`；`IRQ=固定 revision 第三方` | 0 / PASS |
| 文档边界 | 行数、定向字段和相对链接检查 | `doc_lines=238`；`relative_links=PASS` | 0 / PASS |
| Diff gate | `git diff --check` | 无输出 | 0 / PASS |
| OpenSpec gate | `openspec validate establish-k3-com260-gmac-network-baseline --strict` | `Change 'establish-k3-com260-gmac-network-baseline' is valid` | 0 / PASS |

**Persisted Evidence**

None required。全部验证可低成本重跑，决定性输出已写入本 Response。

**Experience Candidates**

None。此次为局部文档证据标签返工，没有形成需记录为 Runbook 或 Incident 的操作经验。

**Remaining Issues**

- 本 Cycle 没有未解决的 Acceptance gap。
- T3-T6 仍属于后续 Iteration；当前未创建第二篇正文，也未同步 G3-G7 或 index。
- G4/G6/G7 及真板行为边界保持未解，没有因本次标签修正而关闭。

**Commit or Diff Reference**

Uncommitted worktree diff；未创建 commit。

## Plan Review

- Review Result: accepted

**Findings**

None。独立 Review 未发现 Critical、Important 或 Minor finding。

**Deviation Classification**

None。Act 对 URL 提取边界的局部调整没有改变执行契约、验证策略或 Acceptance。

**Acceptance Gaps**

None。T1、T2 及本 replan 的五项 Acceptance 均满足。

**Convergence**

父 Cycle 的两个 Important finding 已收敛：来源职责按正文拆分，MMIO/IRQ 证据等级与可追溯输入一致。本 Iteration 无需第三个 Cycle。

**Evidence**

- 独立逐 URL 反查：第一篇正文 9 个直接 URL 在 `source-coverage.md` 中各出现一次；coverage 总数 69、唯一数 69。
- 独立资源检查：MMIO 行和 IRQ 行均标为“固定 revision 第三方”；`0xcac82000` 出现 1 次，`0x2000` 出现 2 次，source 133 出现 1 次，所在段落均明确第三方适用范围。
- 文档边界：238 行；相对链接全部可解析。
- `git diff --check`：退出码 0，无输出。
- `openspec validate establish-k3-com260-gmac-network-baseline --strict`：退出码 0，`Change 'establish-k3-com260-gmac-network-baseline' is valid`。
- 完整相关 diff 与工作区状态已审查；MS06 和其他既有修改未被本 Cycle 扩大或覆盖。

**Follow-up Decision**

- Iteration 000 Review Result 为 `accepted`，T1-T2 可作为后续数据面文档的稳定输入。
- Change 尚有 Iteration 001 的 T3-T6；本次审计不展开该 Iteration。后续调用 `openspec-plan` 为其建立 initial Cycle。
