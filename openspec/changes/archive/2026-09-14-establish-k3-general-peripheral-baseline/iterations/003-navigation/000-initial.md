# Iteration 003 / Cycle 000: 导航、术语与缺口收敛

## Plan Context

- Status: ready
- Iteration: 003-navigation
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: None

**Iteration Scope**

- Change tasks: 4.1-4.3
- Depends on: Iteration 000、001、002（accepted）
- Stable baseline: 三篇通用外设正文可从总索引进入，来源、术语、缺口与汇总计数一致，MS11 可进入最终 Review。
- Verification boundary: 缺口字段与正文对应，术语无重复，索引链接和计数准确，Markdown 与严格 OpenSpec 校验通过。
- Diagnostic boundary: `known-gaps.md`、`terminology.md`、`index.md` 三个汇总表面。
- Deferred tasks: None

**Cycle Scope**

- Trigger: initial
- Acceptance gaps: None
- Repair items: None
- Inherited scope: R1、R7、R8；S6；D2、D7；T4.1-T4.3；M01-M04；G7；Iteration 000-002 accepted 的三篇 peripherals 正文与 70 URL 来源基线。
- Excluded scope: 修改三篇正文或 `source-coverage.md`；新增技术事实；解除未知项；刷新 SNAPSHOT、全局 tasks、M/R/I；修改或读取 `others/`。

**Objective**

把三篇 peripherals 正文的未知项、实际术语和入口同步到现有汇总文档，使读者能从 `docs/index.md` 定位正文，并从术语表和 known-gaps 追踪其边界。

**Background**

Iteration 000-002 已建立 GPIO/PWM/IR-RX、Audio、WDT/RTC 三篇正文。当前 `known-gaps.md` 只到 G13，`terminology.md` 有 53 个术语但缺少本主题的 10 个主写法，`index.md` 仍把 `docs/peripherals/` 标为待聚合。

**Investigation Facts**

- Current Baseline: 三篇正文分别包含 U1-U5、U1-U4、U1-U4，共 13 组四字段未知项；Iteration 002 最终 Act Response 已验证 WDT/RTC 三路径、22 个相对链接、Markdown diff 和严格 OpenSpec 校验。
- Current-State Evidence: `known-gaps.md` 当前登记 G1-G13，其中 G7 承担默认目标 DTS 唯一映射；尚无条目统一承担通用外设的 controller/pinmux/codec/固件代理、运行路径和恢复闭包。`terminology.md` 当前 53 项，GPIO、PWM、IR-RX、I²S、SSPA、DAI、WDT、RTC、RPMI、VCC_RTC 均无主写法行。`index.md` 记录 70 个唯一 URL、53 个术语、13 类缺口，并在主题职责表把 peripherals 标为待聚合。
- Code and Critical Path: 先在 `known-gaps.md` 建立 G14 并同步状态表和来源对应，再向 `terminology.md` 增加 10 个不重复术语，最后在 `index.md` 增加三篇入口并把计数同步为 63 个术语、14 类缺口；来源数量保持 70。

**Implementation Guidance**

G14 只聚合三篇正文已经记录的通用外设专属未知项，与 G4 的 IRQ delivery、G5 的 DMA/cache、G7 的默认 DTB 映射保持职责互斥。术语定义只给对象范围和正文位置，不复制资源表。索引只提供入口、状态和计数，不承载技术正文。

**Behavioral Change**

当前三篇正文无法从总入口进入，术语与未知项也未进入汇总。完成后总入口、术语表和缺口表与已接受正文一致，但不改变任何硬件或运行结论。

**Task Contracts**

### 4.1: 汇总 peripherals 未知项

- Requirement/Scenario: R7 来源可追溯；R8 导航一致；S6 汇总查询。
- Depends on: Iteration 000-002 accepted。
- Targets: `docs/reference/known-gaps.md` 的 G14、缺口状态汇总、缺口与 source-coverage 对应。
- Current behavior: G1-G13 不包含三篇 peripherals 正文的 13 组未知项。
- Required behavior: 新增 G14，按四字段汇总 GPIO/PWM/IR-RX、Audio、WDT/RTC 的专属资料、板级映射、运行路径和恢复缺口；明确与 G4/G5/G7 的互斥边界，状态为 `open`，日期为 2026-09-12。
- Required changes: 新增 G14 正文；在状态表增加一行；在来源对应段说明六个官网入口已 current/active、URL 总数仍为 70，且 G14 不新增来源行。
- Preserve: G1-G13 内容和状态；G7 默认 DTS 职责；三篇正文的四字段语义。
- Forbidden: 解除任何缺口；把 13 个 U 条目逐项复制成长篇正文；修改来源覆盖表或新增 URL。
- Test witness: 修改前 `rg '^## G14\.' docs/reference/known-gaps.md` 预期无输出并退出 1。
- GREEN condition: G14、状态汇总和来源对应三处存在，四字段齐全，正文链接有效，G1-G13 未改变。
- Verification: 检查 G14 标题、四字段、状态行、三篇链接、G4/G5/G7 边界和 70 URL 声明；断链或字段缺失即失败。
- Stop when: 汇总必须改变既有缺口职责、状态或新增来源才能成立。

### 4.2: 补齐 peripherals 术语

- Requirement/Scenario: R8 导航一致；S6 汇总查询。
- Depends on: 4.1。
- Targets: `docs/reference/terminology.md` 的基础术语表。
- Current behavior: 53 个术语中缺少 GPIO、PWM、IR-RX、I²S、SSPA、DAI、WDT、RTC、RPMI、VCC_RTC。
- Required behavior: 增加上述 10 个主写法，每项含英文原词/缩写、别名和限定到对应正文的使用说明；主写法不得重复。
- Required changes: 在基础术语表按主题相邻加入 10 行；区分 I²S controller、SSPA CPU DAI、DAI endpoint、MMIO RTC 与 RPMI RTC，不把静态对象写成运行成功。
- Preserve: 现有 53 行及主写法；M03；正文现用拼写。
- Forbidden: 为未出现在正文的词扩表；改写既有术语；复制资源参数或裁决运行所有权。
- Test witness: 修改前逐项检查 10 个主写法，预期均缺失。
- GREEN condition: 10 个主写法各出现一次，术语总数为 63，定义链接到三篇正文且无断链。
- Verification: 解析术语表首列检查重复和计数；检查 10 个词及链接；重复、缺项或计数非 63 即失败。
- Stop when: 正文存在互斥主写法，无法在不改变正文的情况下统一。

### 4.3: 同步总入口和计数

- Requirement/Scenario: R8 导航一致；S6 汇总查询。
- Depends on: 4.1、4.2。
- Targets: `docs/index.md` 的参考文档计数、已聚合主题正文、主题职责表和维护规则计数。
- Current behavior: 索引缺少三篇 peripherals 入口，把目录标为待聚合，并显示 53 个术语、13 类缺口。
- Required behavior: 增加三篇正文入口；把 peripherals 状态改为 Iteration 000-002 已聚合并指向 G7/G14；同步为 70 URL、63 个术语、14 类缺口，partial 汇总仍为 G3-G5。
- Required changes: 在已聚合主题正文中新增 peripherals 分组；更新参考文档与维护规则中的计数；修改主题职责说明和状态。
- Preserve: 其他主题入口、状态和 70 URL；入口页不复制技术正文。
- Forbidden: 修改正文、覆盖表、SNAPSHOT 或全局任务；把 G14 写成已解除。
- Test witness: 修改前检查 `docs/index.md`，预期三篇文件名均缺失且 peripherals 状态仍为待聚合。
- GREEN condition: 三篇入口各一次，peripherals 状态与 Iteration 一致，70/63/14 三项计数与权威文件一致，所有相对链接有效。
- Verification: 统计覆盖表唯一 URL、术语表数据行和 known-gaps 标题/状态行并与索引比较；解析全部相对链接；运行 `git diff --check` 和严格 OpenSpec 校验。
- Stop when: 实际来源、术语或缺口计数与调查基线矛盾且原因无法局部定位。

**Invariants**

- 三篇正文和 `source-coverage.md` 不修改，来源总数保持 70。
- 汇总文档只索引既有事实和未知项，不新增硬件结论。
- G4/G5/G7 与新 G14 职责互斥；默认目标 DTS 仍未知。
- 不修改 SNAPSHOT、全局 tasks、M/R/I、`others/` 或归档状态。

**Non-goals**

- 不取得新来源，不解除 G1-G14。
- 不修改产品技术正文，不执行真板验证。
- 不创建脚本、依赖、身份字段或 Evidence 目录。

**Acceptance**

- A1 / R7,R8 / S6 / D7 / T4.1: G14 以四字段汇总三篇正文未知项，并与 G4/G5/G7 分责；状态表和来源对应一致。
- A2 / R8 / S6 / D7 / T4.2: 10 个 peripherals 主写法唯一、定义受正文范围约束，术语总数为 63。
- A3 / R8 / S6 / D7 / T4.3: 总索引包含三篇正文，peripherals 状态正确，70 URL、63 术语、14 gaps 与权威文件一致。
- A4 / R7,R8 / S6 / D7 / T4.1-T4.3: 所有相对链接、Markdown diff 和严格 OpenSpec 校验通过。

**Verification**

- 逐项检查 G14 四字段、状态表、来源对应、10 个术语和三篇索引入口。
- 从 `source-coverage.md`、`terminology.md`、`known-gaps.md` 独立统计 70/63/14，并与 `index.md` 比较。
- 解析三个修改文件的相对链接并逐项检查目标存在。
- 运行 `git diff --check` 与 `openspec validate establish-k3-general-peripheral-baseline --strict`；任一非零即失败。

**Gate 2 Readiness**

- Requirement coverage: PASS — R7/R8、S6、D7 映射到 T4.1-T4.3 和 A1-A4。
- Simplification approval: PASS — 无需求简化。
- Investigation completeness: PASS — 已读取三个汇总目标和三篇 accepted 正文的 U 标题，确认现有 G/术语/索引状态与计数。
- Design closure: PASS — G14 职责、10 个术语、索引入口和目标计数均已确定。
- Task executability: PASS — 三个任务均有目标、当前/目标行为、见证、GREEN、验证和停止条件。
- Iteration balance: PASS — 三项共享正文完成后的收敛输入，共同形成可验证的最终导航基线。
- Traceability: PASS — requirement、scenario、design、task、文件和验证形成链路。
- Verification sufficiency: PASS — 缺失见证、独立计数、链接和严格校验直接证明汇总一致性。
- Identity-evidence exclusion: PASS — 未规划身份型证据工程。
- Material unknowns: PASS — 所有硬件未知项只被索引，不要求 Act 解除或裁决。
- Artifact consistency: PASS — 既有 Iteration Map、tasks 与本 Cycle 范围一致。
- Persisted Evidence: PASS — Mode `none`；结果可低成本重跑并由 Act Response 承载。
- User plan approval: PASS — 用户已批准 change 与既有 Iteration Map，本轮未调整范围。

**Persisted Evidence**

- Mode: none

验证可低成本重跑，Act Response 足以记录命令、决定性输出与退出码。

- Budget: 按 Evidence 预算执行（公共规则 › 验证）。

**Risks and Notes**

- G14 聚合 13 组正文未知项，但不逐项复制；三篇正文仍是细节权威位置。
- 术语计数按基础术语表数据行统计；表头和分隔行不计入。
- 如果实施时发现现有 53 项有重复主写法，停止并返回 Plan，不通过修改旧术语吸收差异。

## Act Response

- Status: reported

**Implemented**

完成 T4.1-T4.3：新增 G14 汇总三篇 peripherals 正文的 13 组未知项；向术语表增加 10 个正文实际使用的主写法；在总索引增加三篇正文入口，并把汇总计数同步为 70 URL、63 术语、14 gaps。

**Changed Files and Symbols**

- `docs/reference/known-gaps.md`（243 行 / 36,536 字节）：新增 G14 四字段、状态变更记录、状态汇总行和 source-coverage 对应；明确 G4/G5/G7/G14 分责。
- `docs/reference/terminology.md`（96 行 / 15,607 字节）：新增 GPIO、PWM、IR-RX、I²S、SSPA、DAI、WDT、RTC、RPMI、VCC_RTC 十行，定义均链接到对应正文。
- `docs/index.md`（91 行 / 11,754 字节）：增加 peripherals 三篇正文入口，更新主题职责状态，并同步 63 个术语、G1-G14 共 14 类缺口和维护规则计数。
- `tasks.md`：T4.1-T4.3 由 `[ ]` 改为 `[x]`。

**Deviations from Plan**

None — 按 4.1 → 4.2 → 4.3 的依赖顺序实施，未修改三篇正文、`source-coverage.md`、SNAPSHOT、全局 tasks、M/R/I 或 `others/`。

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
- Spec compliance review（Gate 4-1）：A1-A4 均有直接文件和验证映射；G14 只索引正文既有未知项，不解除缺口；术语只定义对象范围；索引不复制技术正文。
- Code quality review（Gate 4-2）：G1-G13 和既有 53 个术语未改写；新增主写法唯一；70/63/14 计数可由权威文件独立计算；没有身份型证据机制或计划外文件。

**Verification Evidence**

| 验证项 | 命令或操作 | 输出摘录 | 覆盖范围 | 结论 |
|---|---|---|---|---|
| T4.1 RED | `rg '^## G14\.' docs/reference/known-gaps.md` | 无输出，退出 1 | G14 修改前缺失 | PASS（基线） |
| T4.2 RED | 逐项检查 10 个术语主写法 | 10 项均缺失 | 术语修改前状态 | PASS（基线） |
| T4.3 RED | 检查三篇文件名与 peripherals 状态 | 三篇入口缺失；第 73 行仍为 `待聚合` | 索引修改前状态 | PASS（基线） |
| 术语计数与唯一性 | 解析基础术语表首列 | `terms=63 unique=63` | A2 | PASS |
| 缺口编号与状态 | 解析 G 标题和状态表 | `gaps=14 gap_unique=14 status_rows=14` | A1/A3 | PASS |
| 来源 URL | 解析覆盖表 URL 列 | `source_urls=70 unique=70` | A3 | PASS |
| 相对链接 | 解析三个修改文件并检查目标 | known-gaps `70/0`；terminology `46/0`；index `27/0`（链接数/断链数） | A4 | PASS |
| G14 内容 | 检查标题、状态、来源对应、三篇链接和 G4/G5/G7 | G14 正文第 198 行、状态第 226 行、来源对应第 241 行 | A1 | PASS |
| 索引状态与计数 | 检查三篇入口、peripherals 状态和 70/63/14 | 三篇入口第 58-60 行；状态第 77 行；计数第 20-21、85 行 | A3 | PASS |
| Markdown diff | `git diff --check` | 无输出，退出 0 | 三个汇总文件及 change | PASS |
| OpenSpec | `openspec validate establish-k3-general-peripheral-baseline --strict` | `Change 'establish-k3-general-peripheral-baseline' is valid`，退出 0 | change 整体 | PASS |

**Persisted Evidence**

None required — 模式为 `none`；所有结果可低成本重跑，Act Response 足以承载决定性输出。

**Experience Candidates**

None — 本轮是三个 Markdown 汇总表面的常规同步，没有可复用高风险操作或重要故障。

**Remaining Issues**

None within Iteration 003 — G1-G14 的开放项按设计保留，不属于本轮解除范围。

**Commit or Diff Reference**

未提交；实际产品修改为 `docs/reference/known-gaps.md`、`docs/reference/terminology.md`、`docs/index.md`，另更新本 change 的 tasks 和当前 Cycle Act Response。

## Plan Review

- Review Result: accepted

**Findings**

None — 独立审计未发现阻塞 Acceptance 的 Critical、Important 或 Minor finding。G14、10 个术语、三篇索引入口和三个汇总计数均符合 T4.1-T4.3。

**Deviation Classification**

None

**Acceptance Gaps**

None — A1-A4 全部满足。

**Convergence**

N/A

**Evidence**

- 独立读取当前 Cycle、Act Response、三个修改文件的完整 diff，并对照三篇 peripherals 正文的 13 个 U 标题。
- 独立解析权威文件得到 `terms=63 unique=63`、`gaps=14 status_rows=14 sequence=True`、`source_urls=70 unique=70`。
- 独立解析相对链接：`known-gaps.md` 70 个、`terminology.md` 46 个、`index.md` 27 个，断链均为 0。
- 独立重跑 `git diff --check`，无输出且退出 0；`openspec validate establish-k3-general-peripheral-baseline --strict` 输出 `Change 'establish-k3-general-peripheral-baseline' is valid`，退出 0；`openspec list` 显示 change 为 `Complete`。

**Follow-up Decision**

接受 Iteration 003：T4.1-T4.3 和 A1-A4 已完成，无需当前 Cycle 修复、后继 Cycle 或下一 Iteration。change 已无剩余实施任务，可交由 `openspec-docs-maintainer` 收尾。

**Iteration Plan Update**

None

**Next Cycle**

None

**Next Iteration**

None
