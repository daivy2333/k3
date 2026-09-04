# Iteration 001 / Cycle 000: 人工刷新与入口集成

## Plan Context

- Status: draft
- Iteration: 001-refresh-consistency
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: `../000-coverage-structure/001-rework.md`
- User authorization: Pending Gate 2 approval. 用户于 2026-09-04 要求在审计通过后给出下一轮 Iteration；该指令授权创建本 Plan Context，不等同于授权 openspec-act 实施。

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

Gate 2 技术检查通过；用户实施批准尚未给出，因此 Plan Context 保持 `draft`。

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

- Status: pending

**Implemented**

Pending execution.

**Changed Files and Symbols**

Pending execution.

**Deviations from Plan**

None.

**Blocker Handoff**

None.

**Blocker Resolution**

None.

**Self-Review**

- Plan compliance: pending
- Full diff reviewed: pending
- Critical findings unresolved: pending
- Important findings unresolved: pending
- Minor findings unresolved: pending

**Verification Evidence**

Pending execution.

**Persisted Evidence**

None required.

**Experience Candidates**

None pending execution.

**Remaining Issues**

Pending execution.

**Commit or Diff Reference**

None.

## Plan Review

- Review Result: pending

**Findings**

Pending Act Response.

**Deviation Classification**

None.

**Acceptance Gaps**

Pending execution.

**Convergence**

N/A.

**Evidence**

Pending.

**Follow-up Decision**

Pending Plan Review.

**Iteration Plan Update**

None.

**Next Cycle**

None.

**Next Iteration**

None.
