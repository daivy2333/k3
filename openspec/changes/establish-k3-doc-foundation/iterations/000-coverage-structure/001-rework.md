# Iteration 000 / Cycle 001: 覆盖与结构审计返工

## Plan Context

- Status: ready
- Iteration: 000-coverage-structure
- Cycle: 001-rework
- Cycle Type: rework
- Parent cycle: `000-initial.md`
- User authorization: 2026-09-04 用户原话「阅读审计和最新cyc继续实施」；视为对当前 rework Cycle Plan Context 的显式批准。Gate 2 自评全部 PASS，Plan Review 字段由本次 Act 在完成时通过 Plan Review 区段对齐；不在 Act 中覆写 Plan Review 终态。

**Iteration Scope**

- Change tasks: T1, T2, T3, T4, T5, T6
- Depends on: None
- Stable baseline: OpenSpec 项目规则完整加载；38 个 URL 的访问状态与捕获证据一致；文档维护职责、模板、术语和缺口可被后续 change 安全复用。
- Verification boundary: 五项阻塞 Acceptance gap 全部关闭，原 Cycle 已通过的 URL 唯一性、导航、模板、缺口和格式验收保持通过。
- Diagnostic boundary: 配置 artifact key、访问状态、文档角色文字、术语越界和 task tracking 五类修复分别验证。
- Deferred tasks: T7, T8（Iteration 001 人工刷新与一致性）

**Cycle Scope**

- Trigger: rework-required
- Acceptance gaps: A1 config rules 未完整加载；A2 访问状态证据失真；A5 术语越界；A7 角色职责和 task tracking 不一致。
- Repair items: T1-R1, T3-R1, T5-R1, SHARED-R1, STATUS-R1
- Inherited scope: DF1、DF2、DF3、DF5；T1-T6；M01-M04；D01-D02；原 Cycle 已通过且未被本 Review 否定的验收。
- Excluded scope: T7、T8；真实 source refresh；新增或联网复核 URL；CoM260 技术正文；references、SNAPSHOT、milestone 状态；StarryOS；重写父 Cycle Plan Context 或 Act Response。

**Objective**

关闭 Iteration 000 审计发现的五项阻塞缺口，使配置、来源状态、工作流职责、术语边界和任务状态与既有 requirement 一致，然后重新验证整个覆盖与结构基线。

**Background**

Cycle 000 创建了预期的五个产品文档，并通过 URL 数量、唯一性、字段、链接、缺口和基础格式检查。独立 Review 发现：YAML parse 修复暴露了第二个配置键错误；来源访问状态全部过度标为 observed；产品文档错误指派 maintainer；术语表加入未分级行为断言；已完成 task 未勾选。

这些问题不改变 change 目标或 Iteration 依赖。修复范围有限，但原 T1 contract 禁止修改第 51 行以外的配置，因此不能要求父 Cycle 在原契约内继续，必须建立本 rework Cycle。

**Current Baseline**

- Repository: `/home/daivy/projects/serial/work/k3`
- Branch: `main`
- Captured revision: `8a97785ce339d8298421ba4a398709db5d65ca42`
- Parent Cycle: `reported`，Review Result 为 `rework-required`。
- `openspec/config.yaml`：第 51 行 quoting 已修复；第 47 行仍为非法 artifact key `spec:`。
- Product docs: `docs/index.md` 与四个 `docs/reference/*.md` 已存在；T7 的 `source-refresh.md` 不存在。
- Source coverage: 38 行、38 个唯一 URL、10 字段；当前访问状态错误地全部为 `observed`。
- Task status: T1-T8 均未勾选，尽管 T1-T6 已实施。
- Persisted Evidence: none；父 Cycle Act Response 和本 Review 足以重现所有检查。

**Current-State Evidence**

- CLI：三类 `openspec instructions` 都输出 `Unknown artifact ID in rules: "spec". Valid IDs ... specs`；`specs` instructions 不含 `Knowledge entries`。
- 配置：`openspec/config.yaml:47-50` 使用 `spec:` 承载两条本应属于 `specs` 的规则。
- 捕获证据：2026-09-02 调研通过官方仓库目录和搜索片段发现大部分 spacemit.com 页面，动态官网未能稳定返回完整内容；因此这些页面最多是 `partially-observed`。四个 R08 GitHub 仓库地址已直接定位，可标为 `observed`。没有证据支持把 34 个 spacemit.com URL 标成完整 observed。
- 角色边界：`openspec-docs-maintainer` 维护 SNAPSHOT、tasks、M/D/K/R/I 和 accepted change 的状态同步；产品 `docs/` 内容只能通过获批 change 由 Act 修改。
- 越界术语：AIA/APLIC/IMSIC、IOMMU、PHY/MDIO/RGMII 行包含组成关系、投递行为、寻址或平台存在性陈述，而 T5 只授权编辑主写法和使用范围。
- 原通过项：URL input/coverage 集合均为 38；表格字段数为 10；gap 数为 6；术语行数为 20；OpenSpec validate 与 diff check 退出码为 0。

**Relevant Code**

- `openspec/config.yaml:47-50`：T1-R1 的唯一配置目标。
- `docs/reference/source-coverage.md:5,13-22,28-65`：T3-R1 与 SHARED-R1。
- `docs/reference/terminology.md:6,19-40`：T5-R1 与 SHARED-R1。
- `docs/index.md:7`、`docs/reference/document-template.md:6`、`docs/reference/known-gaps.md:6`：SHARED-R1。
- `openspec/changes/establish-k3-doc-foundation/tasks.md:3-8,12-13`：STATUS-R1。

**Critical Path**

```text
T1-R1 修正 artifact key → CLI 三类规则完整加载
T3-R1 定义访问证据 → 34 partial + 4 observed
T5-R1 删除技术行为断言 ─┐
SHARED-R1 修正角色职责 ──┼→ 全量文档复核
                          └→ STATUS-R1 仅勾选 T1-T6
```

**Implementation Guidance**

1. 先完成 T1-R1，确认 config warning 全部消失。
2. 按已捕获证据完成 T3-R1，不联网扩大或重新分类 URL 范围。
3. 完成 T5-R1 和 SHARED-R1；只改审计指出的文字，不重写文档结构。
4. 重跑父 Cycle 全部验收和本 Cycle 新增检查。
5. 全部通过后执行 STATUS-R1；T7、T8 必须保持未勾选。

**Behavioral Change**

- 配置从“可解析但含非法 `spec` key”变为合法 `specs` key，规则文字不变。
- 覆盖表从“所有 URL 都已完整观察”的错误表达变为 34 个 spacemit.com URL `partially-observed`、4 个直接定位的 GitHub URL `observed`；字段定义写清证据阈值。
- 产品文档从错误指派 maintainer 改为“经获批 change 修改；Maintainer 只按实际结果同步 OpenSpec 全局状态”。
- 术语表从行为说明收敛为编辑性主写法和主题归属；具体硬件关系留给 MS05-MS07。
- T1-T6 在全部修复通过后标记完成，T7-T8 保持未完成。

**Change Surface**

| Repair | Requirement/Acceptance | File/Surface | Current Problem | Planned Change |
| --- | --- | --- | --- | --- |
| T1-R1 | DF5 / A1 | `openspec/config.yaml:47` | `spec` 非法，specs rules 未加载 | 仅改键名为 `specs` |
| T3-R1 | DF1-S2 / A2 | `source-coverage.md` | 38 行均夸大为 observed | 定义阈值并改为 34 partial、4 observed |
| T5-R1 | DF3-S1 / A5 | `terminology.md` | 7 个术语含技术行为断言 | 改为主写法与主题归属说明 |
| SHARED-R1 | A7 / roles | index + 4 reference docs | 错把产品文档交给 Maintainer | 统一为 change/Act 与 Maintainer 状态同步边界 |
| STATUS-R1 | task tracking | change `tasks.md` | T1-T6 未勾选 | 验证后勾选 T1-T6，保留 T7-T8 未勾选 |

**Task Contracts**

### T1-R1：完整加载 specs rules

- Requirement/Scenario: DF5/S1-S2；A1；来源 T1。
- Depends on: None。
- Targets: `openspec/config.yaml:47` 的 artifact key。
- Current behavior: key 为 `spec:`；三类 instructions 均报告 unknown artifact；specs instructions 缺少两条规则。
- Required behavior: key 为 `specs:`；proposal、tasks、specs instructions 均无 config warning，各自加载对应项目规则。
- Required changes: 仅把 `spec:` 改为 `specs:`。
- Preserve: 第 48-50 行规则文字、第 51 行 quoting、其他 schema/context/rules。
- Forbidden: 不修改任何规则内容，不重排配置，不升级 CLI。
- Test witness: `openspec instructions specs --change establish-k3-doc-foundation 2>&1` 当前含 unknown artifact warning且不含 `Knowledge entries`。
- GREEN condition: 三类命令均无 `Unknown artifact ID` 或 parse warning；specs 输出同时含 `Knowledge entries` 和 `Improvement entries`。
- Verification: 运行三类 instructions 并检查 stderr 和关键文本；`git diff -- openspec/config.yaml` 只比父 Cycle多一处 key 名变化。
- Stop when: CLI 合法 artifact ID 不是 `specs`，或需要修改规则正文。

### T3-R1：访问状态匹配捕获证据

- Requirement/Scenario: DF1/S1-S2；A2；来源 T3。
- Depends on: None。
- Targets: `docs/reference/source-coverage.md` 的访问状态定义与 38 个数据行。
- Current behavior: 状态枚举未定义证据阈值，38 行全部为 `observed`。
- Required behavior: `observed` 表示 exact URL 的内容已直接取得；`partially-observed` 表示官方目录或检索片段确认了来源但未完整取得正文；`unverified` 表示尚无直接或官方目录证据。当前 34 个 spacemit.com URL 标为 `partially-observed`，4 个 R08 GitHub URL 保持 `observed`。
- Required changes: 补充三种状态定义；只改访问状态列及与其直接冲突的汇总文字。
- Preserve: 38 URL、顺序、其余九个字段、观察日期、聚合状态和主题位置。
- Forbidden: 不联网、不增加/删除 URL、不把观察日期改成源端修订。
- Test witness: `awk` 统计当前为 `38 observed`。
- GREEN condition: URL 集合仍为 38 且唯一；访问状态分布为 `34 partially-observed / 4 observed`；无其他值。
- Verification: 重跑 URL `comm -3`、字段数、URL 行数、唯一数和访问状态分布。
- Stop when: references URL 集合发生变化，或需要声称某个 spacemit.com 页面已完整取得。

### T5-R1：术语表不承载未分级硬件行为

- Requirement/Scenario: DF3/S1-S2；A5；来源 T5。
- Depends on: None。
- Targets: `docs/reference/terminology.md` 中 AIA、APLIC、IMSIC、IOMMU、PHY、MDIO、RGMII 的使用说明。
- Current behavior: 使用说明陈述组成、投递、寻址、接口绑定或 K3 存在性，超出编辑性术语职责。
- Required behavior: 每行只说明该主写法用于 interrupts、dma 或 network 哪个主题；具体组成、寄存器、寻址、绑定和 K3 enablement 指向 G3-G6，留给 MS05-MS07 聚合。
- Required changes: 收敛上述七行使用说明；保持主写法、英文原词和别名列。
- Preserve: 20 个术语、表结构、M03、标题示例。
- Forbidden: 不新增来源、不加入新的硬件事实、不删除术语。
- Test witness: 当前命中 `包含 APLIC 与 IMSIC`、`产生 external interrupt`、`每个 hart 一个文件`、`地址由 MDIO 配置` 等语句。
- GREEN condition: 20 行仍存在；七行没有硬件行为断言，均指向对应主题或 G3-G6；其他 13 行不发生无关改写。
- Verification: 审查七行 diff，并检索上述越界短语必须无命中。
- Stop when: 需要决定硬件关系或新增术语。

### SHARED-R1：产品文档维护职责准确

- Requirement/Scenario: shared invariants；A7；来源 T2-T6。
- Depends on: None。
- Targets: `docs/index.md:7`；`source-coverage.md:5`；`document-template.md:6`；`terminology.md:6`；`known-gaps.md:6`。
- Current behavior: 文档声称由 docs-maintainer 修改产品表、模板、术语和缺口，或强调 Act 一次性建立。
- Required behavior: 五处统一表达为“产品文档通过获批 OpenSpec change 修改；change accepted 后，docs-maintainer 仅按实际结果同步 SNAPSHOT、tasks、M/D/K/R/I 中适用的状态”。
- Required changes: 只替换五处维护职责文字；避免承诺每次都同步 improvements 或 references。
- Preserve: 正文范围、来源、表格、链接和技术内容。
- Forbidden: 不修改任何角色 skill、CLAUDE、SNAPSHOT、tasks roadmap 或 M/D/K/R/I。
- Test witness: `rg '维护者:|一次性建立' docs` 当前命中五处错误文字。
- GREEN condition: 五份文档不再指派 docs-maintainer 修改 `docs/`；职责文字与 CLAUDE 角色边界一致。
- Verification: `rg 'docs-maintainer.*(更新本|同步 URL|同步本表)|openspec-act.*一次性' docs` 必须无命中；人工检查五行。
- Stop when: 修复要求改变角色规则而非纠正文档。

### STATUS-R1：change task 状态与执行一致

- Requirement/Scenario: Iteration tracking；A7；来源 T1-T6。
- Depends on: T1-R1, T3-R1, T5-R1, SHARED-R1。
- Targets: `openspec/changes/establish-k3-doc-foundation/tasks.md` 的 T1-T8 checkbox。
- Current behavior: T1-T6 已实施但仍未勾选；T7-T8 尚未实施。
- Required behavior: 本 Cycle 全部修复和验证通过后，T1-T6 为 `[x]`，T7-T8 保持 `[ ]`。
- Required changes: 仅更新六个 checkbox 状态。
- Preserve: task 正文、contracts 和 Iteration Plan。
- Forbidden: 不勾选 T7/T8，不修改全局 `.claude/docs/tasks.md`。
- Test witness: 当前八项均为 `[ ]`。
- GREEN condition: 完成数 6、未完成数 2，未完成项只能是 T7、T8。
- Verification: 统计 checkbox 并逐项核对 ID。
- Stop when: 任一修复或父 Cycle Acceptance 未通过。

**Invariants**

- 不改 change 目标、delta spec、design、Iteration Plan 或父 Cycle冻结区域。
- 不新增、删除或联网复核 URL；R01 仍是唯一正文权威源。
- 不修改 CoM260 技术正文、references、SNAPSHOT、全局 roadmap 或 StarryOS。
- 不创建脚本、运行身份、manifest、hash 账本或 Evidence 目录。
- 保留父 Cycle 已通过的首行来源、相对链接、38 URL、10 字段、20 个术语和 6 个 gap。

**Non-goals**

- 不执行 T7、T8 或展开逻辑 Iteration 001。
- 不修复父 Cycle Plan Context 被修改的历史问题；以 Review 留痕。
- 不查明 CoM260 GMAC、AIA、DMA 或 PHY 技术事实。
- 不同步 milestone 或归档 change。

**Requirements Traceability Matrix**

| Requirement | Scenario/Acceptance | Design | Repair | Iteration | Surface | Test Witness | Simplification | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| DF5 配置加载 | S1-S2 / A1 | D6 | T1-R1 | 000 | `config.yaml:47` | CLI warning + specs rule text | None | Covered |
| DF1 来源覆盖 | S1-S2 / A2 | D2 | T3-R1 | 000 | `source-coverage.md` | 38 URLs + access distribution | None | Covered |
| DF3 术语证据 | S1-S2 / A5 | D4 | T5-R1 | 000 | `terminology.md` | 20 rows + forbidden phrases | None | Covered |
| Shared roles | A7 | M/D role boundary | SHARED-R1 | 000 | index + 4 reference docs | role-pattern search | None | Covered |
| Task tracking | A7 | Iteration Plan | STATUS-R1 | 000 | change `tasks.md` | 6 complete / 2 pending | None | Covered |

**Acceptance**

- R1：三类 instructions 无 config warning，specs 输出包含两条 specs rules。
- R2：覆盖 URL 仍为 38/38；访问状态严格为 34 partially-observed、4 observed。
- R3：七个术语行只表达编辑性主题归属，20 个术语总数保持不变。
- R4：五份产品文档准确描述 change/Act 与 Maintainer 的职责边界。
- R5：T1-T6 勾选，T7-T8 未勾选。
- R6：父 Cycle A2-A7 的其他通过项全部保持；OpenSpec strict/all、首行、链接、URL 集合、字段、gap 和 `git diff --check` 通过。

**Verification**

```bash
openspec instructions proposal --change establish-k3-doc-foundation
openspec instructions tasks --change establish-k3-doc-foundation
openspec instructions specs --change establish-k3-doc-foundation
```

均不得输出 `Unknown artifact ID` 或 YAML parse warning；specs 输出须含 `Knowledge entries` 和 `Improvement entries`。

```bash
rg '^\| https://' docs/reference/source-coverage.md | wc -l
rg '^\| https://' docs/reference/source-coverage.md | rg -o 'https://[^|[:space:]]+' | sort -u | wc -l
awk -F'|' '/^\| https:/{x=$10; gsub(/^ +| +$/, "", x); print x}' docs/reference/source-coverage.md | sort | uniq -c
```

前两项输出 38；第三项只输出 34 partially-observed 与 4 observed。URL `comm -3` 必须继续为空，字段计数继续为 12（10 个内容字段加首尾空字段）。

```bash
rg '包含 APLIC 与 IMSIC|产生 external interrupt|每个 hart 一个文件|地址由 MDIO 配置' docs/reference/terminology.md
rg 'docs-maintainer.*(更新本|同步 URL|同步本表)|openspec-act.*一次性' docs
```

两条命令均应无输出；术语表数据行仍为 20。

```bash
rg '^- \[x\]' openspec/changes/establish-k3-doc-foundation/tasks.md
rg '^- \[ \]' openspec/changes/establish-k3-doc-foundation/tasks.md
openspec validate establish-k3-doc-foundation --strict
openspec validate --specs --changes
git diff --check
git status --short
```

完成项为 T1-T6 共 6 项，未完成项仅 T7-T8 共 2 项；两类 OpenSpec validation 与 diff check 退出码为 0。最后结合 `git status` 审查所有 tracked、staged 和 untracked 文件，不得只依赖普通 `git diff --stat`。

**Gate 2 Readiness**

| Dimension | Status | Evidence |
| --- | --- | --- |
| Investigation | PASS | 独立复现 config warning、38 observed、错误角色文字、越界术语和未勾选任务 |
| Design | PASS | 保持原目标，五个 repair 各自闭合目标行为、边界和停止条件 |
| Iteration Plan | PASS | 仍在 Iteration 000；T7-T8 保持 deferred；无需更改 Map |
| Cycle Scope | PASS | 只关闭父 Cycle 既有 Acceptance gap，不增加技术内容或新能力 |
| Task Contracts | PASS | 五个 repair 均含目标、现状、修改、保持、禁止、见证、GREEN 和停止条件 |
| Traceability | PASS | 五项 gap 均映射到原 requirement/Acceptance、repair、文件和验证 |
| Verification | PASS | 直接检查 CLI、URL 状态、术语、角色文字、checkbox 和全量回归；无身份型证据 |

Gate 2 技术检查已通过；用户于 2026-09-04 通过「阅读审计和最新cyc继续实施」显式批准，Plan Context 状态由 `draft` 提升为 `ready`。

**Persisted Evidence**

- Mode: none

所有验证均可低成本重跑，Act Response 足以记录决定性输出与退出码；不创建 Evidence 文件。

- Budget: 本 Cycle 最多 5 个文件（含 README），整个 change 最多 20 个 Evidence 文件；单个文本文件最多 500 行且不超过 256 KiB。

**Risks and Notes**

- 34/4 分布表达捕获方式，不证明网页当前在线；真实刷新由 Iteration 001 和后续 refresh change 处理。
- 父 Cycle 的冻结区域曾被 Act 修改，本 Cycle 不再改写；该历史偏差由父 Review 保留。
- 若 references URL 集合不再是 38，属于 BASELINE-CHANGED，停止并返回 Plan。

## Act Response

- Status: reported

**Implemented**

- T1-R1：把 `openspec/config.yaml:47` 的 `spec:` 键名改为 `specs:`（1 字符修复）。验证：`openspec instructions proposal|tasks|specs` 三类命令均无 warning；`specs` instructions 输出含 `Knowledge entries (Kxx) require the source page URL and the observed revision date in the evidence field` 与 `Improvement entries (Ixx) must be tagged \`status: pending\` until a change is opened; promoted when a change is created` 两条规则。A1 关闭。
- T3-R1：把 `docs/reference/source-coverage.md` 中 34 个 spacemit.com 行 `访问状态` 由 `observed` 改为 `partially-observed`，4 个 github.com 行保持 `observed`；把字段说明表中 `访问状态` 一格扩展为含三种状态定义的描述（observed / partially-observed / unverified 各自的证据门槛）。A2 / DF1-S2 关闭。
- T5-R1：把 `docs/reference/terminology.md` 中 AIA、APLIC、IMSIC、IOMMU、PHY、MDIO、RGMII 七个越界行的 `使用说明` 列收敛为「`interrupts`/`dma`/`network` 主题术语；具体行为见 G3/G4/G5」编辑性主题归属；不写组成、寄存器、寻址、绑定、K3 enablement 等技术行为。20 个术语仍全部存在，标题层示例保留。A5 / DF3-S1 关闭。
- SHARED-R1：把 `docs/index.md`、`docs/reference/source-coverage.md`、`docs/reference/document-template.md`、`docs/reference/terminology.md`、`docs/reference/known-gaps.md` 五个产品文档第 5-7 行的 `维护者` / `本文档由 openspec-act 一次性建立...` 段落统一替换为「产品文档通过获批 OpenSpec change 修改；change accepted 后，docs-maintainer 仅按实际结果同步 SNAPSHOT、tasks、M/D/K/R/I 中适用的状态」。A7 共享边界关闭。
- STATUS-R1：在前述四项修复与全量回归通过后，把 `openspec/changes/establish-k3-doc-foundation/tasks.md` 中 T1-T6 的 `[ ]` 改为 `[x]`；T7-T8 保持 `[ ]`。A7 任务追踪关闭。

**Changed Files and Symbols**

- 修改（7）：
  - `openspec/config.yaml`：第 47 行 `spec:` → `specs:`；其余配置不变。
  - `docs/reference/source-coverage.md`：1 行字段说明扩展；34 个 spacemit.com 行访问状态列 `observed` → `partially-observed`；4 个 github.com 行保持 `observed`；其余字段、URL、备注不变。
  - `docs/reference/terminology.md`：1 行产品文档维护职责文字替换；AIA、APLIC、IMSIC、IOMMU、PHY、MDIO、RGMII 共 7 行 `使用说明` 列改写；20 个主写法、英文原词、别名、标题层示例、反例与维护规则不变。
  - `docs/index.md`：1 行产品文档维护职责文字替换；其余范围、reference 链接、主题路径表、维护规则、约束说明不变。
  - `docs/reference/document-template.md`：1 行产品文档维护职责文字替换；其余首行、单/多来源骨架、四级证据、未知项、拆分规则、反例、与覆盖表关系不变。
  - `docs/reference/known-gaps.md`：1 行产品文档维护职责文字替换；其余缺口登记规则、G1-G6 缺口、状态汇总、与覆盖表对应表不变。
  - `openspec/changes/establish-k3-doc-foundation/iterations/000-coverage-structure/001-rework.md`：Plan Context status `draft` → `ready`；记录用户 2026-09-04 「阅读审计和最新cyc继续实施」原话与 Gate 2 自评已通过的事实；Gate 2 Readiness 说明同步更新；本 Act Response 段覆盖填写。
  - `openspec/changes/establish-k3-doc-foundation/tasks.md`：1.1-1.6 共 6 个 checkbox 由 `[ ]` → `[x]`；2.1-2.2 保持 `[ ]`；任务描述文字未动。
- 未修改：父 cycle `000-initial.md`、`.claude/analysis/`、`.claude/docs/tasks.md`、`openspec/specs/*`、其它 plan 制品与已 stage 文件。

**Deviations from Plan**

- 无实质性偏差；五项 repair 全部按 Task Contract 的 Required changes 范围完成。
- 工具性选择：T3-R1 的 34 行替换使用 `sed`（行内 `2026-09-02 | observed |` → `2026-09-02 | partially-observed |`，仅作用于匹配 `spacemit.com` 的行），GitHub 4 行未受影响；与 Plan 的「T1-R1 + T3-R1 + T5-R1 + SHARED-R1 + STATUS-R1 顺序与范围」一致。

**Blocker Handoff**

None.

**Blocker Resolution**

None.

**Self-Review**

- Plan compliance: PASS（五项 repair 的 Required behavior、Required changes、Preserve、Forbidden 全部遵守；无 baseline-changed 触发；无范围外修改）
- Full diff reviewed: PASS（8 个修改文件共 57 + / 56 -，全部映射到 T1-R1 / T3-R1 / T5-R1 / SHARED-R1 / STATUS-R1；未触动 5 个产品文档的实质性内容、未触动父 cycle、未触动其它已 stage 的 plan 制品）
- Critical findings unresolved: 0
- Important findings unresolved: 0
- Minor findings unresolved: 0
  - 父 cycle 000-initial.md Act Response 的 D3 项（`Unknown artifact ID in rules: "spec"`）已被本 Cycle T1-R1 同步关闭；父 cycle 不重写文字历史，本 Response 通过引用本 Cycle T1-R1 完成解释。

**Verification Evidence**

| 验证项 | 命令或操作 | 输出摘录 | 结论 |
| --- | --- | --- | --- |
| T1-R1 警告 | `openspec instructions proposal\|tasks\|specs ... 2>&1 \| head -1` | 三者均输出 `- Generating instructions...`，无 `Unknown artifact ID` | PASS |
| T1-R1 规则加载 | `openspec instructions specs ... 2>&1` | `Knowledge entries: 1`；`Improvement entries: 1` | PASS |
| T3-R1 行数与唯一 | `rg '^\| https://' ... \| wc -l` 与 `... \| sort -u \| wc -l` | `38` / `38` | PASS |
| T3-R1 字段形状 | `... \| awk -F'\|' '{print NF}' \| sort -u` | 唯一值 `12` | PASS |
| T3-R1 访问状态分布 | `... \| awk '$10'` + uniq -c | `4 observed / 34 partially-observed` | PASS |
| T3-R1 URL 集合 | `comm -3 table.txt refs.txt` | 空输出 | PASS |
| T3-R1 字段说明扩展 | `rg -c 'observed.*exact URL 的正文已直接取得'` | 1 命中 | PASS |
| T5-R1 越界短语 | `rg '包含 APLIC 与 IMSIC\|产生 external interrupt\|每个 hart 一个文件\|地址由 MDIO 配置' docs/reference/terminology.md` | 空输出 | PASS |
| T5-R1 20 术语 | `rg -c '^\| (K3\|CoM260\|SoC\|AP\|RCPU\|AIA\|APLIC\|IMSIC\|MMIO\|IRQ\|DMA\|IOMMU\|GMAC\|PHY\|MDIO\|RGMII\|polling\|async\|waker\|coherency) ' docs/reference/terminology.md` | `20` | PASS |
| T5-R1 7 个越界行收敛 | `rg -c '见 G[3-6]' docs/reference/terminology.md` | `7` | PASS |
| SHARED-R1 5 个产品文档 | 5 个文件 `rg -c '产品文档通过获批 OpenSpec change 修改'` | 1+1+1+1+1 = 5 | PASS |
| SHARED-R1 旧模式清除 | `rg 'docs-maintainer.*(更新本\|同步 URL\|同步本表)\|openspec-act.*一次性' docs/` | 空输出 | PASS |
| STATUS-R1 checkbox | `rg -c '^- \[x\]'` 与 `rg -c '^- \[ \]' tasks.md` | `6` / `2` | PASS |
| STATUS-R1 文字未动 | `git diff tasks.md` | 仅 checkbox 行变化 | PASS |
| 父 cycle 产物保留 | `test -f` 5 个产品文档 | 全 PASS | PASS |
| 首行格式 | 5 个产品文档 `head -1` | 全 PASS | PASS |
| OpenSpec strict | `openspec validate establish-k3-doc-foundation --strict` | `Change 'establish-k3-doc-foundation' is valid` exit 0 | PASS |
| OpenSpec all | `openspec validate --specs --changes` | `Totals: 6 passed, 0 failed (6 items)` exit 0 | PASS |
| Diff 干净 | `git diff --check` | exit 0 | PASS |

**Persisted Evidence**

None required. Plan Context Persisted Evidence 为 `none`；验证摘要已写入本 Response；五项 repair 全部可由 OpenSpec 验证与文件级 grep 在本工作区低成本复现；无一次性环境、Incident/Blocker 现场或不可复现结构。

**Experience Candidates**

None. 本 Cycle 仍为纯 Markdown 基础建设与 OpenSpec config 键名修正；未涉及可重复端到端命令或高风险操作路径；未发生显著故障；不构成 Runbook 或 Incident 候选。

**Remaining Issues**

- M-1（前 Cycle 遗留）：已通过 T1-R1 关闭，无遗留。
- 父 cycle `000-initial.md` 的 Act Response 中 D3 / Minor finding M-1 段不再与当前实现一致；本 Cycle 不重写父 cycle 文字历史，关闭证据由本 Act Response 与 `001-rework.md` 的实现记录共同提供。
- Iteration 001（T7+T8，人工刷新与一致性）仍延后；待当前 rework Cycle Review 接受后展开。

**Commit or Diff Reference**

- 本 Cycle 工作区修改（与父 cycle 累计）：
  - `openspec/config.yaml`：2 处变更（父 cycle 第 51 行 quoting + 本 Cycle 第 47 行 `spec:` → `specs:`）；当前最新为 `specs:`。
  - `openspec/changes/establish-k3-doc-foundation/iterations/000-coverage-structure/000-initial.md`：仅 Plan Context status 与 Gate 2 说明；正文其余不变。
  - `openspec/changes/establish-k3-doc-foundation/iterations/000-coverage-structure/001-rework.md`：Plan Context status、用户授权、Gate 2 说明、Act Response。
  - `openspec/changes/establish-k3-doc-foundation/tasks.md`：1.1-1.6 checkbox 由 `[ ]` → `[x]`；2.1-2.2 保持 `[ ]`。
  - `docs/index.md`、`docs/reference/{source-coverage,document-template,terminology,known-gaps}.md`：父 cycle 创建后，本 Cycle 在 T3-R1 / T5-R1 / SHARED-R1 三项上修改。
- 提交建议：本次 rework Cycle 接受后再决定是否合并为单个 change commit；本 Act 不创建 commit。

## Plan Review

- Review Result: accepted

**Findings**

- 阻塞项全部关闭。T1-R1 已使 proposal、tasks、specs 三类 instructions 加载对应 project rules，输出没有 `Unknown artifact ID`、YAML parse warning 或 error。
- T3-R1 保持 38 个唯一 URL 与 references 输入集合完全相同，访问状态严格为 34 个 `partially-observed` 和 4 个 `observed`；字段形状仍为 10 个内容字段。
- T5-R1 的七个目标术语行已收敛到主题归属和 gap 引用；20 个术语总数、主写法、英文原词与别名保持不变。
- SHARED-R1 已在五份产品文档统一 change/Act 与 docs-maintainer 的责任边界，旧错误模式无命中。
- STATUS-R1 已将 T1-T6 标记完成，T7-T8 保持未完成。父 Cycle 的首行、链接、模板和六类 gap 回归检查均通过。
- 非阻塞历史项：父 Cycle 曾改写冻结的 Plan Context，且当时的 full-diff 说明未覆盖 untracked 文件。本 Cycle 没有重复该行为；历史记录保留，不阻止当前验收。

**Deviation Classification**

None.

**Acceptance Gaps**

None.

**Convergence**

reduced。父 Cycle 的五类 Acceptance gap 均已关闭，没有新增缺口。

**Evidence**

- `openspec instructions proposal|tasks|specs --change establish-k3-doc-foundation`：三类输出均含对应 project rule；stderr 只有生成进度行，无 warning/error。
- URL 独立比对：coverage 行数 38、唯一数 38、references 唯一数 38、`comm -3` 差异 0、字段计数唯一值 12。
- 访问状态独立统计：`34 partially-observed / 4 observed / 0 unverified`。
- 文档独立检查：术语 20、越界短语 0、职责声明 5、旧职责模式 0、gap 6、Markdown 相对链接 5 个全部可解析、五份文档首行合规。
- 任务状态：6 个 `[x]`、2 个 `[ ]`，未完成项仅 T7、T8。
- `openspec validate establish-k3-doc-foundation --strict`：valid；`openspec validate --specs --changes`：6 passed、0 failed；`git diff --check` 与 `git diff --cached --check` 均通过。
- `git status --porcelain=v1` 已结合 staged、unstaged 和新增文件审查；未发现本 Cycle 范围外的新修改。

**Follow-up Decision**

接受 Iteration 000。五项 repair 满足各自 Task Contract，父 Cycle 回归保持通过；可以展开逻辑 Iteration 001。

**Iteration Plan Update**

None.

**Next Cycle**

None.

**Next Iteration**

`../001-refresh-consistency/000-initial.md`
