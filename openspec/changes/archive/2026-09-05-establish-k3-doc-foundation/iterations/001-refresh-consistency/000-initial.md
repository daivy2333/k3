# Iteration 001 / Cycle 000: 人工刷新与入口集成

## Plan Context

- Status: ready
- Iteration: 001-refresh-consistency
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: `../000-coverage-structure/001-rework.md`
- User authorization: 用户 2026-09-04 原话「阅读审计和最新cyc继续实施」与本次「更改gate状态，继续实施」视为对当前 initial Cycle Gate 2 的显式批准：Plan Context 自包含、Gate 2 Readiness 7/7 PASS、Persisted Evidence=none 均满足；Act 直接按 Plan Context 与 Task Contract 实施 T7、T8。

**Iteration Scope**

- Change tasks: T7, T8
- Depends on: Iteration 000 accepted
- Stable baseline: 维护者可按文档对覆盖表执行可中断、可继续、不会静默替换来源的人工复核，并可从总入口定位该流程。
- Verification boundary: 五种刷新结果、完整操作顺序和 unchanged、unreachable、partial-resume 三个文字演练均可执行；刷新指南链接及全部既有文档检查通过。
- Diagnostic boundary: 失败限定在刷新结果定义、操作顺序、中断恢复、change 边界或入口链接集成。
- Deferred tasks: None

**Cycle Scope**

- Trigger: initial
- Acceptance gaps: None
- Repair items: None
- Inherited scope: DF2/S1、DF4/S1-S4、T7、T8、M01-M04、D01-D05，以及 Iteration 000 accepted baseline。
- Excluded scope: 真实官网刷新；修改覆盖表行级元数据；新增或替换 URL；CoM260 技术正文；自动化脚本、运行 ID、manifest、hash 账本、Evidence 目录；全局 SNAPSHOT、tasks roadmap、M/D/K/R/I；StarryOS。

**Objective**

创建人工来源刷新指南，明确一次检查的结果、顺序、change 边界和中断恢复规则，再把指南接入 `docs/index.md`。本 Cycle 只建立可执行流程，不把 2026-09-02 baseline 伪装成一次真实刷新。

**Background**

Iteration 000 已接受：OpenSpec rules 可加载，覆盖表保存 38 个唯一 URL，访问状态为 34 个 `partially-observed` 和 4 个 `observed`，文档模板、术语与六类 gap 已稳定。总入口当前只以代码样式预告 `docs/reference/source-refresh.md`，因为该文件尚不存在。

DF4 要求人工流程覆盖 `unchanged`、`changed`、`moved`、`removed`、`unreachable`，并允许部分 URL 检查后中断。D5 规定这些是一次检查的结果，不得与覆盖表的长期聚合状态或访问状态混用；`changed` 必须先创建 refresh change，`moved` 必须保留旧 URL。

**Current Baseline**

- Repository: `/home/daivy/projects/serial/work/k3`
- Branch: `main`
- Captured revision: `8a97785ce339d8298421ba4a398709db5d65ca42`
- Parent Cycle: `accepted`；T1-T6 已完成，T7-T8 未完成。
- `docs/reference/source-refresh.md`: 不存在。
- `docs/index.md`: 以代码样式预告刷新指南，不存在失效链接。
- `source-coverage.md`: 38 行、38 个唯一 URL；聚合状态为 `active/deferred/out-of-scope/supporting`，访问状态为 `observed/partially-observed/unverified`。
- 文档来源元数据统一使用源端修订和观察日期；初始观察日期为 2026-09-02。
- OpenSpec baseline: change strict validation valid；全量 6 passed、0 failed。

**Current-State Evidence**

- `docs/index.md:22`：刷新指南仍是代码路径，并注明由 Iteration 001 创建后转为相对链接。
- `docs/reference/source-coverage.md:9-22`：定义长期覆盖字段；没有一次检查的结果字段或全局完成标志。
- `docs/reference/document-template.md`：提供来源首行、源端修订、观察日期和证据强度格式，可直接用于刷新指南。
- `design.md` D5：刷新结果与聚合状态分离；部分检查只更新已检查行，未检查行保持原值；初始 baseline 不算真实 refresh。
- Delta spec DF4/S1-S4：分别约束 unchanged、changed、moved/removed 和中断恢复。
- 当前没有已确认的源端变化。本 Cycle 的三个场景只能作为文字演练，不能修改覆盖表或主题正文。

**Relevant Code**

本 change 不包含运行时代码。相关文档职责如下：

- `docs/reference/source-coverage.md`：保存每个 URL 的长期范围、聚合状态、源端修订、观察日期和访问状态；本 Cycle 只读。
- `docs/reference/document-template.md`：约束新文档首行、来源元数据和证据表达；本 Cycle 只读。
- `docs/reference/known-gaps.md`：当 moved、removed 或 unreachable 无法定位替代来源时承接缺口；本 Cycle 只读。
- `docs/reference/source-refresh.md`：T7 新建，负责人工刷新流程，不保存某次运行身份或结果账本。
- `docs/index.md`：T8 只把预告路径转换为有效相对链接。

**Critical Path**

```text
Iteration 000 accepted baseline
  ├─ source-coverage 的三类长期状态
  ├─ document-template 的来源元数据格式
  └─ DF4 + D5 的刷新约束
          ↓
source-refresh：五种结果 → 逐 URL 检查顺序 → change/缺口边界 → 中断恢复
          ↓
unchanged / unreachable / partial-resume 文字演练
          ↓
docs/index 相对链接 → 全量回归 → T7/T8 状态
```

`source-coverage.md` 的 URL 行仍是长期状态所有者；实际刷新产生的逐 URL 结论记录在对应 refresh change 的 Act Response，coverage 行只保存最新源端修订、观察日期和访问状态。`source-refresh.md` 只定义操作规则，不保存某次运行记录，也不得引入新的全局“本轮刷新完成”状态。

**Implementation Guidance**

1. 先完成 T7。开篇明确刷新结果与聚合状态、访问状态是三个不同维度。
2. 按“选择范围、读取来源、比较修订/正文、分类结果、处理行级元数据、按需创建 change 或登记 gap、验证”的顺序描述人工流程。
3. 为 unchanged、unreachable、partial-resume 各写一个文字演练，明确哪些行可以改变、哪些行必须保持原值。
4. 仅在 T7 文件存在且验证通过后执行 T8，把入口预告改成相对链接。
5. 全量回归通过后勾选 T7、T8；若遇到真实来源变化，停止本 Cycle 并返回 Plan，为该变化单独建立 refresh change。

**Behavioral Change**

当前维护者只能看到覆盖表字段与设计目标，无法仅凭产品文档确定一次人工刷新如何分类、何时创建 change、URL 移动时如何保留历史，或中断后从哪里继续。目标行为是维护者能按一份指南逐 URL 检查，并在不使用运行日志或脚本的情况下区分已检查与未检查行。

本 Cycle 不执行来源刷新，因此不产生新的观察日期、刷新结论或正文事实。入口变化只有一处：刷新指南从代码路径变为可解析的相对链接。

**Change Surface**

| Task | Requirement/Scenario | File | Current Responsibility | Planned Change |
| --- | --- | --- | --- | --- |
| T7 | DF4/S1-S4 | `docs/reference/source-refresh.md` | 不存在 | 建立五种结果、操作顺序、change/gap 边界、中断恢复和三个文字演练 |
| T8 | DF2/S1, DF4/S1-S4 | `docs/index.md` | 以代码样式预告不存在的指南 | 只把预告转换为有效相对链接，并保持其他内容不变 |
| T7/T8 status | Iteration tracking | `openspec/changes/establish-k3-doc-foundation/tasks.md` | T7、T8 未勾选 | 全部验收通过后只勾选 T7、T8 |

**Task Contracts**

### T7：人工刷新流程可恢复

- Requirement/Scenario: DF4/S1 unchanged、S2 changed、S3 moved/removed、S4 中断恢复。
- Depends on: Iteration 000 accepted。
- Targets: `docs/reference/source-refresh.md`。
- Current behavior: 文件不存在；维护者没有产品级操作顺序和恢复规则。
- Required behavior: 指南定义 `unchanged`、`changed`、`moved`、`removed`、`unreachable` 五种一次检查结果，并将它们与聚合状态及访问状态明确分开。实际 refresh change 的 Act Response 记录逐 URL 结论，coverage 行保存最新元数据；中断后由行级元数据和 Act Response 中的已检查清单确定恢复点，未检查行保持原值。
- Required changes: 使用 D3 首行格式；说明适用范围与前置条件；给出选择范围、读取来源、比较 revision/正文、分类、行级更新、创建 change、更新正文、验证和停止/恢复顺序；明确任何产品文档更新都需要获批 change，若在 change 外发现 `changed` 则停止并先提案；说明五种结果分别记录到哪里；用文字演练 unchanged、unreachable、partial-resume。
- Preserve: R01 唯一权威源；R08 只作 supporting；D01 人工 Markdown；M04 无可执行工具；change 先于正文修改；初始 baseline 不算真实 refresh。
- Forbidden: 不记录真实刷新结果，不修改 `source-coverage.md`、`known-gaps.md` 或主题正文；不创建脚本、运行 ID、manifest、hash 账本或 Evidence；不静默替换旧 URL；不要求 Act 联网。
- Test witness: `test -f docs/reference/source-refresh.md` 当前退出码为 1。
- GREEN condition: 文件存在且首行合规；五种结果均有判定、动作和记录位置；两类长期状态与刷新结果无混用；操作顺序、停止条件和恢复点明确；三个文字演练都能推出唯一结果，且 partial-resume 不改变未检查行。
- Verification: 人工逐项对照 DF4/D5；检索五种结果、三类聚合/访问状态和三个场景标题；检查指南没有把 2026-09-02 标成一次新 refresh，也没有可执行代码或 Evidence 要求。
- Stop when: 发现真实来源变化、references URL 集合变化、流程依赖仓库内自动化，或需要决定某个 K3 技术事实。

### T8：刷新入口完成集成

- Requirement/Scenario: DF2/S1；DF4/S1-S4。
- Depends on: T7 GREEN。
- Targets: `docs/index.md`；完成后更新 change `tasks.md` 的 T7/T8 checkbox。
- Current behavior: `docs/index.md` 以代码样式展示 `docs/reference/source-refresh.md`，并说明文件尚未创建。
- Required behavior: 总入口提供指向 `reference/source-refresh.md` 的有效相对链接；既有四个 reference 链接、CoM260 范围、九类主题职责和维护约束保持不变。全部验收通过后，T7、T8 标记完成。
- Required changes: 只替换刷新指南所在列表项，使其名称、职责和链接与实际文件一致；最终只改变 T7、T8 checkbox。
- Preserve: `docs/index.md` 其余正文与链接；T1-T6 完成状态；Iteration 000 五份文档内容和 38 URL baseline。
- Forbidden: 不借入口集成改写其他文档，不增加技术内容，不修改全局 `.claude/docs/tasks.md`、SNAPSHOT 或 M/D/K/R/I。
- Test witness: T7 前刷新路径不是 Markdown 链接，目标文件不存在；T7/T8 当前均为 `[ ]`。
- GREEN condition: 新链接存在且可解析，其他相对链接仍有效；`docs/index.md` 除目标列表项外无内容变化；change tasks 八项全部为 `[x]`。
- Verification: 逐项解析 `docs/index.md` 的所有相对 Markdown 链接；审查 index diff；统计 change task checkbox；重跑全量文档和 OpenSpec 检查。
- Stop when: T7 未通过，或集成需要改变 Iteration 000 的范围、导航设计或产品文档职责。

**Invariants**

- 当前技术目标板仅 K3 CoM260 Kit；不扩展到 Pico、RV2768、Shelf 或 StarryOS 实现。
- 刷新结果、聚合状态、访问状态是不同维度；指南不得把一次检查结果写入长期状态枚举。
- 未检查行不改变观察日期、访问状态、源端修订或备注。
- 实际 refresh change 的 Act Response 保存逐 URL 结论和已检查清单；coverage 不增加刷新结果列，仓库不创建独立运行日志。
- `moved` 保留旧 URL；`removed` 与 `unreachable` 不触发静默替换。
- `changed` 只创建或要求创建独立 refresh change；本 Cycle 不更新受影响正文。
- 所有产品文档通过获批 change 修改；docs-maintainer 只同步 accepted 结果对应的全局状态。
- 不覆盖进入本 Cycle 前已有的 staged、unstaged 或新增文件。

**Non-goals**

- 不实际访问官网或 GitHub，不宣称任何来源 unchanged/changed/unreachable。
- 不更新 2026-09-02 观察日期，不新增 URL 或关闭 gap。
- 不编写刷新脚本、链接检查器或网页抓取器。
- 不创建 CoM260 platform、boot、interrupts、serial、DMA 或 network 正文。
- 不同步 milestone，不归档 change。

**Requirements Traceability Matrix**

| Requirement | Scenario | Design | Task | Surface | Test Witness | Simplification | Status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| DF4 人工刷新 | S1 unchanged | D5 | T7 | `source-refresh.md` | unchanged 文字演练 | 无运行日志 | Covered |
| DF4 人工刷新 | S2 changed | D5 | T7 | `source-refresh.md` | change 边界检查 | 不执行真实刷新 | Covered |
| DF4 人工刷新 | S3 moved/removed | D5 | T7 | `source-refresh.md` | 旧 URL 与 gap 路径检查 | 不静默替换 | Covered |
| DF4 人工刷新 | S4 中断恢复 | D5 | T7 | `source-refresh.md` | partial-resume 文字演练 | 复用行级元数据 | Covered |
| DF2 主题导航 | S1 总入口 | D1 | T8 | `docs/index.md` | 相对链接解析 | 单列表项修改 | Covered |
| Iteration tracking | T7/T8 completion | Iteration Plan | T8 | change `tasks.md` | 8 complete / 0 pending | None | Covered |

**Acceptance**

- A1 / DF4 / T7：`source-refresh.md` 存在，首行符合 D3，正文明确初始 baseline 不是一次真实 refresh。
- A2 / DF4 / T7：五种刷新结果都有互斥判定、后续动作和记录位置，并明确区别于四种聚合状态与三种访问状态；逐 URL 结论进入对应 refresh change 的 Act Response，coverage 不增加刷新结果列。
- A3 / DF4-S2-S3 / T7：操作顺序要求 `changed` 先创建 refresh change；`moved` 保留旧 URL；`removed`、`unreachable` 无法定位替代来源时进入 gap，不静默替换。
- A4 / DF4-S1-S4 / T7：unchanged、unreachable、partial-resume 三个文字演练可逐步执行；部分完成只更新已检查行，未检查行原值不变。
- A5 / DF2 / T8：总入口刷新指南链接存在并可解析；index 除该列表项外保持不变，其他相对链接全部有效。
- A6 / tracking / T8：T7、T8 在 A1-A5 和全量回归通过后标记完成，change tasks 为 8 complete、0 pending。
- A7 / regression：Iteration 000 baseline 保持 38 个唯一 URL、34/4 访问状态、20 个术语、6 个 gap、五份原文档首行与链接合规；OpenSpec strict/all 和两类 diff check 通过。

**Verification**

```bash
test -f docs/reference/source-refresh.md
head -n 1 docs/reference/source-refresh.md
rg 'unchanged|changed|moved|removed|unreachable' docs/reference/source-refresh.md
rg 'active|deferred|out-of-scope|supporting' docs/reference/source-refresh.md
rg 'observed|partially-observed|unverified' docs/reference/source-refresh.md
```

首行必须同时含直接来源 URL、源端修订和观察日期。正文必须解释三组状态的职责，不能只罗列名称。

人工执行三份场景表：

- unchanged：版本和相关正文未变；只允许更新已检查行的观察日期并记录 unchanged 结论，不改无关正文。
- unreachable：保留 URL；按 R01 尝试重新定位；无法确认替代来源时进入 gap，不声称 removed。
- partial-resume：先检查一个最小子集后中断；未检查行所有元数据保持原值；恢复时能由行级元数据确定下一条。

```bash
rg -n '\[[^]]+\]\([^)]+\.md\)' docs/index.md docs/reference/*.md
rg '^\| https://' docs/reference/source-coverage.md | wc -l
rg '^\| https://' docs/reference/source-coverage.md | rg -o 'https://[^|[:space:]]+' | sort -u | wc -l
awk -F'|' '/^\| https:/{x=$10; gsub(/^ +| +$/, "", x); print x}' docs/reference/source-coverage.md | sort | uniq -c
rg -c '^\| (K3|CoM260 Kit|SoC|AP|RCPU|AIA|APLIC|IMSIC|MMIO|IRQ|DMA|IOMMU|GMAC|PHY|MDIO|RGMII|polling|async|waker|coherency) ' docs/reference/terminology.md
rg -c '^## G[1-6]\.' docs/reference/known-gaps.md
rg -c '^- \[x\]' openspec/changes/establish-k3-doc-foundation/tasks.md
rg -c '^- \[ \]' openspec/changes/establish-k3-doc-foundation/tasks.md
openspec validate establish-k3-doc-foundation --strict
openspec validate --specs --changes
git diff --check
git diff --cached --check
git status --short
```

预期 coverage 为 38/38，访问状态仅 34 partial + 4 observed，术语 20、gap 6、task 8/0。相对链接必须逐项解析到现有文件；最后结合完整 status 审查 staged、unstaged 和新增文件。

**Gate 2 Readiness**

| Dimension | Status | Evidence |
| --- | --- | --- |
| Investigation | PASS | 已确认刷新文件不存在、入口仅预告、覆盖表状态模型和 Iteration 000 accepted baseline |
| Design | PASS | DF4/D5 已定义五种结果、行级恢复、change 与 gap 边界；无未决设计问题 |
| Iteration Plan | PASS | T7 先建立流程，T8 后集成入口；两项共同形成可发现、可执行的稳定结果 |
| Cycle Scope | PASS | 只交付刷新指南、单项入口修改和 T7/T8 状态；真实刷新及技术正文排除 |
| Task Contracts | PASS | T7/T8 均有现状、目标、边界、见证、GREEN、验证与停止条件 |
| Traceability | PASS | DF4 四个场景、DF2 导航和任务追踪均映射到文件与见证 |
| Verification | PASS | 五种结果、三个演练、链接、任务状态及 Iteration 000 全量回归均可直接检查 |

Gate 2 技术检查通过；用户已显式给出实施批准（见 Plan Context 顶部 User authorization），Plan Context 升至 `ready`，Act 可直接按 Task Contract 实施 T7、T8。

**Persisted Evidence**

- Mode: none

本 Cycle 只创建流程文档并修改一个导航项，所有验收可由 Act Response 和低成本文件检查重现；不需要 Evidence 目录。

- Budget: 本 Cycle 最多 5 个文件（含 README），整个 change 最多 20 个 Evidence 文件；单个文本文件最多 500 行且不超过 256 KiB；本 Cycle 计划创建 0 个 Evidence 文件。

**Risks and Notes**

- `unchanged` 是一次检查结论，不是覆盖表的新长期枚举。若文档需要永久保存每次结论，当前 D5 契约失效，应停止并返回 Plan。
- `unreachable` 不等于 `removed`；只有官方入口或明确响应能支持 removed 分类。
- 本 Cycle 使用现有 baseline 做文字演练，不得据此更新观察日期。
- 当前 worktree 含此前获批并已审计的 staged/unstaged 变更；Act 必须保留并在 full diff review 中覆盖它们。

## Act Response

- Status: reported

**Resume (Act)**

Plan Review Follow-up Decision 要求 openspec-act 在当前 Cycle 内修复 G1-G4，且修复只涉及 T7 指南语义、T7/T8 checkbox 和 Act Response，不创建新 Current-State Evidence、rework Cycle 或后继 Iteration Map，不修改已交接的 Plan Context。用户 2026-09-05 原话「更改gate状态，开始实施」授权本 Cycle 恢复实施。Act 按规则把 `Act Response` 状态从 `reported` 调整为 `pending`，只消费最新 Review 的 G1-G4 修复，不恢复已被覆盖的文字历史。

**Implemented**（含 G1-G4 修复后的完整当前状态）

- T7 / G1：`docs/reference/source-refresh.md` 第 55 行 `moved` 结果定义中，"动作" 段由"迁移事实同时写入 refresh change 的 Act Response 与 `known-gaps.md`（经获批 change 修改）"的无条件写法改为条件分支：R01 入口或重定向已确认新 URL 时不登记 `known-gaps.md`（替代来源已确认，不存在待解除缺口），只有疑似移动但从 R01 找不到替代来源时才经获批 change 登记缺口；同步把第 57 行"不变量"追加"已确认替代来源的 `moved` 不留下任何 `known-gaps.md` 缺口条目"。同文件第 103 行 `变更与缺口边界` 表中 `moved` 条目同步改为"R01 入口或重定向已确认新 URL 时不登记 `known-gaps.md`，只有疑似移动且无替代来源时经获批 change 登记缺口；新行由 refresh change Act 写入"。A3 / DF4-S3 关闭。
- T7 / G2：`docs/reference/source-refresh.md` 第 94 行 `操作顺序` 步骤 5 由"任何 URL 判为 `changed`、`moved`、`removed` 或需产品文档更新的长期 `unreachable` 时创建 refresh change"扩展为"任何会修改产品文档的 URL 结果（`unchanged` 仅更新 `观察日期`、`unreachable` 仅更新 `观察日期` 与有证据的 `访问状态`、`changed` 更新正文与覆盖表、`moved` 追加新行、`removed` 登记缺口或长期 `unreachable` 无替代来源时登记缺口）都必须先有获批的 refresh change；预检查本身永远不修改 `source-coverage.md` 任一列"，明确 `unchanged` 与暂时 `unreachable` 这两类同样会修改覆盖表的结果必须先创建并获批 refresh change。A3 / DF4-S1 关闭。
- T7 / G3：`docs/reference/source-refresh.md` 第 142 行 `演练 2：unreachable` 步骤 4 由"refresh change 的 Plan Context 允许将该行 `观察日期` 改为 `2026-09-04`、`访问状态` 降为 `unverified`"改为"refresh change 的 Plan Context 仅允许将该行 `观察日期` 改为 `2026-09-04`；既有 `访问状态` 为 `partially-observed` 保持不变，因为单次超时/5xx 不能抹除已有官方目录或检索片段证据；只有新证据（页面被官方显式下架、官方目录移除该条目、或确认原访问方法永久失效）证明原访问证据不成立时，才能由获批 change 决定将 `访问状态` 改为 `unverified`"；期望结果同步改为"1 行 `观察日期` 更新；既有 `访问状态` 保持不变；无 `removed`、无缺口、无静默替换；不存在仅凭网络失败就降级访问状态的场景"。A2-A3 / DF4-S3 关闭。
- T7 整体：经 G1-G3 修复后，`source-refresh.md` 共 184 行；首行仍符合 D3 格式（来源 URL + 源端修订 unknown + 观察日期 2026-09-02）；5 个互斥结果、3 个独立维度、7 步操作顺序、4 条 change/gap 边界、中断恢复 + 3 演练 + 6 条停止条件、反例段与 T7 初次实施保持一致。
- T8 保持前次已通过的入口修改：`docs/index.md` 第 22 行刷新指南相对链接与其余 4 个 reference 链接、CoM260 范围、九类主题职责表、维护规则与全局约束段不变；不在 G1-G4 范围内。
- T7/T8 任务追踪（G4）：先把 `tasks.md` 中 `2.1 [T7]`、`2.2 [T8]` 由 `[x]` 恢复为 `[ ]`（GREEN 前），G1-G3 修复与全量回归通过后重新勾选为 `[x]`；任务描述文字与已有 6 个 `[x]` 不动。
- 不创建新 Current-State Evidence、不开新 Cycle 或后继 Iteration、不修改 Plan Context、不修改 `.omo/`；Plan Context 在之前 Cycle 由 `draft` 升为 `ready` 的历史偏差仍保留，本 Act 不重写也不回写冻结区域。

**Changed Files and Symbols**

- 修改（3，均为已存在文件的 G1-G4 定点修改）：
  - `docs/reference/source-refresh.md`：4 处 G1-G3 定点修改
    - 第 55 行 `moved` 动作段（按替代来源是否已确认分两支）
    - 第 57 行 `moved` 不变量追加"无 known-gaps.md 缺口条目"
    - 第 103 行 `变更与缺口边界` 表中 `moved` 条目（同步条件化）
    - 第 94 行 `操作顺序` 步骤 5（覆盖全部会修改覆盖表的 5 类结果）
    - 第 142 行 `演练 2：unreachable` 步骤 4（保持 partially-observed 不变）
    - 第 146 行 `演练 2` 期望结果（不允许仅凭网络失败降级）
    - 其他章节（5 个结果标题、3 维度、覆盖表关系、changed/removed 段、change/gap 边界、中断恢复、演练 1/3、停止条件、反例、首行 D3、维护规则）保持 baseline。
  - `openspec/changes/establish-k3-doc-foundation/tasks.md`：2.1、2.2 共 2 个 checkbox 由 `[x]`→`[ ]`（G4 前置）→`[ ]`→`[x]`（GREEN 后）；任务描述文字、Iteration 000 的 6 个 `[x]` 与 Task Contracts 段不变。
  - `openspec/changes/establish-k3-doc-foundation/iterations/001-refresh-consistency/000-initial.md`：仅 Act Response 区域（Status 状态 + Resume 块 + 本节被覆盖为 G1-G4 修复后的最终状态）；Plan Context 区域保持 `ready` 不变；Plan Review 区域保留上轮 Review 原文，本 Act 不回写。
- 不修改（保证范围受控）：
  - `docs/index.md` (T8 入口已在初次 Act 完成；本 Act 不动)
  - `docs/reference/{source-coverage,document-template,terminology,known-gaps}.md` (本 Cycle 范围外，全部保持 baseline)
  - `openspec/config.yaml`、`openspec/specs/*`、`.claude/*` (本 Cycle 范围外)
  - 父 Cycle `000-coverage-structure/000-initial.md` 与 `001-rework.md` (本 Cycle 范围外，不重写父历史)
  - `.omo/run-continuation/ses_*.json` (会话运行延续产物，非 product)

**Deviations from Plan**

- 无实质性偏差。Plan Context Invariants、Non-goals、Forbidden 与 Task Contract Required changes 全部遵守。
- 本 Act 按 Plan Review Follow-up Decision 在当前 Cycle 内修复 G1-G4：T7 指南 4 处定点修改、T7/T8 checkbox、Act Response 记录与必要验证；Task Contract 与 delta spec 已给出完整行为，不需要 Change Surface 变更、Iteration Map 调整、后继 Cycle、Persisted Evidence。
- 工具性选择：G1-G3 修复使用 edit 工具做唯一字符串替换，未触发覆盖式 write；保留其余章节（5 个结果标题、3 维度、覆盖表关系、changed/removed 段、change/gap 边界、中断恢复、演练 1/3、停止条件、反例、首行 D3、维护规则）逐字 baseline。
- 流程偏差（历史，非本 Act 引入）：Plan Context 的 `draft`→`ready` 由之前 Cycle 在用户授权下完成，Review 已记为非阻塞 process finding；本 Act 不重写、不回写冻结区域，也不修改 Plan Context。

**Blocker Handoff**

None.

**Blocker Resolution**

None.

**Self-Review**

- Plan compliance: PASS（T7、T8 的 Required behavior、Required changes、Preserve、Forbidden 全部遵守；7 项 Invariants 全部满足；Non-goals 全部未触发；G1-G4 修复逐项覆盖 Plan Review 的 Acceptance Gaps 与 Follow-up Decision）
- Full diff reviewed: PASS（本 Cycle 实施汇总：`source-refresh.md` G1-G3 共 4 处定点修改；`tasks.md` 仅 T7/T8 checkbox 2 行；当前 Cycle 文件仅 Act Response 区域；未触动 Iteration 000 五份产品文档正文、未触动 5 个 reference baseline 字段、未触动父 cycle、未触动 OpenSpec 其它 spec/change、未修改 Plan Context、未修改 Plan Review 区域）
- G1 / A3 / DF4-S3：moved 动作第 55 行改为「R01 入口或重定向已确认新 URL 时不登记 `known-gaps.md`，只有疑似移动但从 R01 找不到替代来源时才经获批 change 登记缺口」；第 57 行不变量追加「已确认替代来源的 `moved` 不留下任何 `known-gaps.md` 缺口条目」；第 103 行 `变更与缺口边界` 表中 `moved` 条目同步条件化。rg 验证旧无条件写法不再出现，条件分支与"不得登记"/"经获批 change 登记"双形态就位 → 已修复
- G2 / A3 / DF4-S1：操作顺序步骤 5 第 94 行改为「任何会修改产品文档的 URL 结果（`unchanged` 仅更新 `观察日期`、`unreachable` 仅更新 `观察日期` 与有证据的 `访问状态`、...）都必须先有获批的 refresh change；预检查本身永远不修改 `source-coverage.md` 任一列」；明确 unchanged 与暂时 unreachable 必须先创建并获批 refresh change。rg 验证「`unchanged` 仅更新...`unreachable` 仅更新」与「预检查本身永远不修改」两段同时出现 → 已修复
- G3 / A2-A3 / DF4-S3：演练 2 步骤 4 第 142 行改为「`观察日期` 改为 `2026-09-04`；既有 `访问状态` 为 `partially-observed` 保持不变，因为单次超时/5xx 不能抹除已有官方目录或检索片段证据；只有新证据...才能由获批 change 决定将 `访问状态` 改为 `unverified`」；期望结果同步改为「既有 `访问状态` 保持不变；不存在仅凭网络失败就降级访问状态的场景」。rg 验证旧"`访问状态` 降为 `unverified`"写法不再出现 → 已修复
- G4 / tracking：tasks.md `2.1 [T7]`、`2.2 [T8]` 由 `[x]`→`[ ]`（G4 前置）→最终 `[x]`（GREEN 后）；当前 rg 计数 8 / 0 → 已执行
- Critical findings unresolved: 0
- Important findings unresolved: 0
- Minor findings unresolved: 0
  - `docs/reference/source-refresh.md` 全文 184 行，约为 500 行建议阈值的 37%，保留充足拆分余量；不构成 Minor finding。
  - 入口导航 5 个 reference 链接相对路径为 `reference/<name>.md` 形式，与目标文件位置一致；不构成 Minor finding。
  - 首部 `> 流程规则来源` 仍写 `D03`、`D05`，与 design.md 的 D3、D5 引用不一致；不影响流程行为（仅文字层面），后续编辑时可改；不构成 Minor finding。

**Verification Evidence**

| 验证项 | 命令或操作 | 输出摘录 | 结论 |
| --- | --- | --- | --- |
| CLI proposal | `openspec instructions proposal --change establish-k3-doc-foundation` | `- Generating instructions...` 无 warning | PASS |
| CLI tasks | `openspec instructions tasks --change establish-k3-doc-foundation` | `- Generating instructions...` 无 warning | PASS |
| CLI specs | `openspec instructions specs --change establish-k3-doc-foundation` | `- Generating instructions...` 无 warning | PASS |
| T7 文件存在 | `test -f docs/reference/source-refresh.md` | exit 0 | PASS |
| T7 行数 | `wc -l docs/reference/source-refresh.md` | `184 docs/reference/source-refresh.md` | PASS |
| T7 首行（D3） | `head -1 docs/reference/source-refresh.md` | `> 来源: https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs（源端修订: unknown；观察日期: 2026-09-02）` | PASS |
| G1 旧无条件写法清除 | `rg -n "迁移事实同时写入 refresh change 的 Act Response 与 \`known-gaps.md\`" docs/reference/source-refresh.md` | 无输出（exit 1） | PASS |
| G1 条件分支就位 | `rg -n "R01 入口或重定向已确认新 URL" docs/reference/source-refresh.md` | 2 处命中（moved 动作 + 变更与缺口边界 moved 条目） | PASS |
| G1 不变量"无 gap 条目" | `rg -n "已确认替代来源的 \`moved\` 不留下任何 \`known-gaps.md\` 缺口条目" docs/reference/source-refresh.md` | 1 处命中（moved 不变量） | PASS |
| G2 步骤 5 覆盖全 5 类 | `rg -n "\`unchanged\` 仅更新.*\`unreachable\` 仅更新" docs/reference/source-refresh.md` | 1 处命中（操作顺序步骤 5） | PASS |
| G2 预检查永远不改 | `rg -n "预检查本身永远不修改 \`source-coverage.md\` 任一列" docs/reference/source-refresh.md` | 1 处命中（操作顺序步骤 5） | PASS |
| G3 演练 2 旧写法清除 | `rg -n "访问状态.*降为 \`unverified\`" docs/reference/source-refresh.md` | 无输出（exit 1） | PASS |
| G3 演练 2 新写法就位 | `rg -n "单次超时/5xx 不能抹除已有官方目录或检索片段证据" docs/reference/source-refresh.md` | 1 处命中（演练 2 步骤 4） | PASS |
| G3 演练 2 期望结果 | `rg -n "不存在仅凭网络失败就降级访问状态的场景" docs/reference/source-refresh.md` | 1 处命中（演练 2 期望结果） | PASS |
| T8 链接解析 | `for f in reference/source-coverage.md reference/document-template.md reference/terminology.md reference/known-gaps.md reference/source-refresh.md; do test -f docs/$f; done` | 5 个 全 OK | PASS |
| STATUS task 8/0 | `rg -c '^- \[x\]' tasks.md`; `rg -c '^- \[ \]' tasks.md` | 8 / 0 | PASS |
| Coverage 38/38 | `rg '^\| https://' source-coverage.md \| wc -l`; `... \| sort -u \| wc -l` | 38 / 38 | PASS |
| Access 4+34 | `awk -F'\|' '/^\| https:/{print $10}' source-coverage.md \| sort \| uniq -c` | `4 observed` / `34 partially-observed` | PASS |
| 术语 20 | `rg -c '^\| (K3\|CoM260 Kit\|SoC\|AP\|RCPU\|AIA\|APLIC\|IMSIC\|MMIO\|IRQ\|DMA\|IOMMU\|GMAC\|PHY\|MDIO\|RGMII\|polling\|async\|waker\|coherency) ' terminology.md` | 20 | PASS |
| 缺口 6 | `rg -c '^## G[1-6]\.' known-gaps.md` | 6 | PASS |
| OpenSpec strict | `openspec validate establish-k3-doc-foundation --strict` | `Change 'establish-k3-doc-foundation' is valid` exit 0 | PASS |
| OpenSpec all | `openspec validate --specs --changes` | `Totals: 6 passed, 0 failed (6 items)` exit 0 | PASS |
| Git diff 干净 | `git diff --check`; `git diff --cached --check` | 均 exit 0 | PASS |

**Persisted Evidence**

None required. Plan Context Persisted Evidence 已声明 `none`；本 Cycle 仅在已有 `source-refresh.md` 上做 4 处定点修改、勾选 tasks.md 2 个 checkbox、更新当前 Cycle Act Response，所有 Acceptance 可由 `head` / `rg` / `wc` / `awk` / `test -f` / `openspec validate` / `git diff --check` 在本工作区低成本复现；不涉及一次性环境、Incident/Blocker 现场或不可重现结构；不引入 Evidence 目录。

**Experience Candidates**

None. 本 Cycle 仍为纯 Markdown 基础建设的人工流程文档化；G1-G3 修复均不涉及可重复端到端命令或高风险操作路径；未发生显著故障；`source-refresh.md` 本身是流程规则文档，不是可执行 Runbook；不构成 Runbook 或 Incident 候选。

**Remaining Issues**

- Iteration 001 / Cycle 000 的 Plan Review Follow-up G1-G4 已在当前 Cycle 内修复并回归通过；全部 8 个 task 勾选。本 Act Response 从 `pending` 改回 `reported`，等待 Plan Review 对该修复给出 `accepted` / `rework-required` / `replan-required`。
- 遗留非产品会话产物：`git status` 中 `?? .omo/run-continuation/ses_*.json` 是本会话运行延续状态文件，非本 change 产品内容；Act 未修改，遗留待用户决定是否忽略或清理。
- change 接受后 `openspec-docs-maintainer` 才会同步 SNAPSHOT、tasks、M/D/K/R/I 中适用的状态；本 Act 不做同步。
- 首次真实 source refresh 由未来来源变更 change 完成；本仓库不通过本 Cycle 伪造任何 refresh 结论。
- 头部 `> 流程规则来源` 段使用 `D03`、`D05` 而 design.md 对应标题为 D3、D5；不影响流程行为，后续编辑时宜改成准确引用。

**Commit or Diff Reference**

- 本 Cycle product 工作区改动（与父 cycle 及此前未提交工作累计，均未 commit）：
  - 修改：`docs/reference/source-refresh.md`（4 处 G1-G3 定点修改，其余保持 baseline）；`openspec/changes/establish-k3-doc-foundation/tasks.md`（T7/T8 checkbox 2 行 `→[ ]→[x]`，其余保持 baseline）；`openspec/changes/establish-k3-doc-foundation/iterations/001-refresh-consistency/000-initial.md`（仅 Act Response 区域被覆盖为 G1-G4 修复后最终状态；Plan Context 与 Plan Review 区域保持不变）。
  - 仍为未跟踪：`docs/reference/source-refresh.md`（新增文件，本 Cycle 未引入新文件）。
  - 前次 Act 已通过但本 Cycle 保持不变：`docs/index.md`（T8 第 22 行入口相对链接）。
- `git status --short`：
  ```
   M docs/index.md
   M openspec/changes/establish-k3-doc-foundation/iterations/001-refresh-consistency/000-initial.md
   M openspec/changes/establish-k3-doc-foundation/tasks.md
  ?? .omo/                                             （会话运行延续产物，非 product）
  ?? docs/reference/source-refresh.md
  ```
- 提交建议：与前次 `1f572bb` 类似的常规 commit 形式；本 Act 不创建 commit。

## Plan Review

- Review Result: accepted

**Findings**

- G1 关闭：已确认替代 URL 的 `moved` 不登记 gap；只有疑似移动且无法确认替代来源时才经获批 change 登记。
- G2 关闭：操作顺序现在覆盖所有会修改产品文档的刷新结果，包括只更新观察日期的 `unchanged` 和暂时 `unreachable`；预检查不修改 coverage。
- G3 关闭：单次超时或 5xx 只更新观察日期，既有 `partially-observed` 保持不变；访问状态变化需要独立证据和获批 change。
- G4 关闭：T7/T8 在修复前恢复未完成，全部验证通过后重新勾选；当前 change 为 8 complete、0 pending。
- A1-A7 全部满足。指南包含五种互斥结果、三个状态维度、两阶段 change 边界、逐 URL 记录、长期 unreachable gap、中断恢复和三个可执行文字演练；入口链接与 Iteration 000 baseline 保持有效。
- Non-blocking / terminology：指南首部写 `D03`、`D05`，而 change design 标题是 D3、D5。上下文可唯一定位，不影响验收；后续编辑可统一显示格式。
- Non-blocking / process：首次 Act 修改冻结 Plan Context 的历史偏差保留；后两次恢复 Act 均只修改 Act Response 和获批产品范围，没有再次越界。
- Non-blocking / workspace：`.omo/run-continuation/*.json` 是未跟踪会话产物，不属于 change；本 Cycle 未修改，也不纳入验收或收尾同步。

**Deviation Classification**

PLAN-INVALID | ACT-DEVIATION（resolved）。上一版 Review 的 moved/gap 指令和随后发现的实现偏差均已在当前 Cycle 内修复。

**Acceptance Gaps**

None.

**Convergence**

reduced。上一版剩余 G1-G4 全部关闭，没有新增 Acceptance gap。

**Evidence**

- `docs/reference/source-refresh.md:54-57,103`：moved 的 gap 处理已按替代来源是否确认分支；已确认新 URL 时明确不登记 gap。
- `docs/reference/source-refresh.md:90-96`：所有会修改产品文档的结果均要求先有获批 refresh change，预检查明确不修改 coverage。
- `docs/reference/source-refresh.md:139-146`：单次超时/5xx 保持既有访问状态；演练结果只更新观察日期。
- 独立结构验证：guide 184 行；五种结果标题 5；演练 3；tasks 8/0；coverage 38/38 且与 references 差异 0；访问状态 4 observed + 34 partially-observed；术语 20；gap 6；7 个相对 Markdown 链接全部可解析。
- 三类 `openspec instructions` 均加载对应 project rules，stderr 无 warning、unknown artifact 或 parse error。
- `openspec validate establish-k3-doc-foundation --strict`：valid；`openspec validate --specs --changes`：6 passed、0 failed；`git diff --check` 与 `git diff --cached --check` 均通过。
- 完整工作区审查：tracked diff 仅含 `docs/index.md`、change `tasks.md` 和当前 Cycle；产品 untracked 仅 `docs/reference/source-refresh.md`；范围外 `.omo/` 保持未修改。

**Follow-up Decision**

接受 Iteration 001 和 change `establish-k3-doc-foundation`。全部 8 个 task 已完成，没有当前 Cycle 修复、rework 或 replan 工作；后续由用户决定是否调用 openspec-docs-maintainer 收尾。

**Iteration Plan Update**

None.

**Next Cycle**

None.

**Next Iteration**

None.
