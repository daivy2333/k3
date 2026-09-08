# Iteration 001 / Cycle 001: UART 事实与收尾一致性修复

## Plan Context

- Status: ready
- Iteration: 001-uart-console-closeout
- 用户原话豁免: "更改gate，开始实施"（2026-09-08 19:14 用户消息，作为 Gate 2 User Plan Approval 的豁免依据; 仅豁免 Gate 2 的"等待用户审计和批准"步骤，不豁免 Plan 的其余检查项）
- Cycle: 001-rework
- Cycle Type: rework
- Parent cycle: `000-initial.md`

**Iteration Scope**

- Change tasks: T3-T6
- Depends on: Iteration 000 `accepted`
- Stable baseline: MS04 的 UART 实例、静态 console 链、来源冲突、MS03 勘误和主题导航形成一致知识包
- Verification boundary: UART 实例按域可追溯；直接来源符合模板；正文不保留已完成任务或错误状态；最终记录没有虚构后续 Iteration
- Diagnostic boundary: UART 地址序列、来源声明、正文完成状态和跨文档一致性
- Deferred tasks: None

**Cycle Scope**

- Trigger: rework-required
- Acceptance gaps: A1、A4、A5、A6
- Repair items: T3-R1、T5-R1、T6-R1
- Inherited scope: proposal R1-R5、design D1-D9、T3-T6、Iteration 001 Cycle 000 的已实现产品结果
- Excluded scope: 新增 UART 事实、选择唯一 Kit DTS、裁决 FIFO 冲突、修改全局 tasks/SNAPSHOT/M/D/K/R/I、驱动实现和后续 milestone

**Objective**

修正 UART 地址序列、直接来源和完成状态，使三篇相关产品文档使用同一分域事实，串口正文符合来源模板并准确反映 T4-T6 已完成；验证后不再存在后续 Iteration。

**Background**

Cycle 000 已交付 T3-T6，但 Review 发现非 secure 地址序列把 APBC2 secure `uart1` 混入连续范围，串口正文还有首行未声明或未登记的直接 URL，并保留实施前的“下一步”和错误行数。由于同一地址表述存在于已接受的 `k3-platform-control.md`，需要新的 rework 执行契约才能跨 Iteration 产品文件修复。

**Current Baseline**

- `docs/serial/com260-uart.md` 318 行；17 实例表、console 链、冲突和未知项主体已存在。
- 非 secure AP 节点为 `uart0`、`uart2`..`uart10`；其中 `uart0`、`uart2`..`uart9` 的实际 base 为 `0xd4017000`..`0xd4017800`，`uart1` 是 APBC2 secure 节点 `0xf0612000`，`uart10` 为 `0xd401f000`。
- 错误范围表达位于 `docs/serial/com260-uart.md`、`docs/platform/k3-platform-control.md` 和 `docs/reference/known-gaps.md`。
- 串口正文直接链接 `k3_com260_kit_v02.dts`、`com260_ds.md` 和 Rt-Async-AMP 仓库根，但首行只有 8 个 URL；Rt-Async-AMP 仓库根未登记在 `source-coverage.md`。
- §11 仍列 T4-T6 为下一步，§12 写“实际 ~430 行”；Iteration Map 实际止于 001。

**Current-State Evidence**

- `docs/serial/com260-uart.md:69-78` 的逐实例表给出正确离散节点与 base，可作为修正文案的现有事实依据。
- `docs/serial/com260-uart.md:42,244`、`docs/platform/k3-platform-control.md:274`、`docs/reference/known-gaps.md:106` 使用错误的 `uart0..uart9` 连续范围。
- `docs/reference/document-template.md` 要求直接来源进入首行，正文 URL 必须先在 `source-coverage.md` 登记。
- `docs/serial/com260-uart.md:70-73,170,200-202` 含首行未声明的外部直接链接；相应官方 URL 已登记，Rt-Async-AMP 仓库根 URL 未登记。
- `docs/serial/com260-uart.md:300-305,318` 与完成后的 tasks 和 `wc -l` 不一致。
- 本 change 的 Iteration Map 只有 000 和 001；`tasks.md` 的 T1-T6 全部为 `[x]`。

**Relevant Code**

- `docs/serial/com260-uart.md`：实例、来源、下一步和边界声明。
- `docs/platform/k3-platform-control.md`：前序平台基线中的同一地址序列说明。
- `docs/reference/known-gaps.md`：G8 当前证据。
- `docs/reference/source-coverage.md`：正文外部 URL 的登记权威表；本 Cycle 默认不修改。

**Critical Path**

1. 以串口逐实例表核对三个文档中的非 secure 节点集合和实际 base。
2. 修正地址序列表述，不改变逐实例值、域归属或 G8 的未知结论。
3. 对照模板处理首行外的直接来源：保留直接外链时补齐合规首行和既有登记；可由仓库主题文档或 R09-R12 承载时改为相对引用，禁止新增无必要来源登记。
4. 删除已完成的 T4-T6“下一步”状态，保留真正的 MS05-MS07 后续边界；用实际行数替换估算值。
5. 运行来源、内容、链接、行数、diff 和 OpenSpec 验证，并在最新 Act Response 明确没有 Iteration 002。

**Implementation Guidance**

- 地址说明使用显式节点集合，不用含 secure `uart1` 的编号闭区间代替物理序列。
- `com260_ds.md` 所支持的板级事实已经由 `com260-board-resources.md` 承载时，优先保留相对链接，避免重复直接来源。
- 第三方固定 revision 经验优先引用 R09-R12 对应分析或本地只读路径；不得为修复模板而新增与产品来源覆盖职责无关的仓库根 URL。
- §11 只保留真正未实施的后续 milestone，不重写已完成任务的技术正文。

**Behavioral Change**

当前文档在地址序列、来源清单和完成状态上互相矛盾。目标状态保留原有 17 实例、未知项和第三方边界，只纠正事实范围、引用方式和状态描述。

**Change Surface**

| Repair | Requirement/Scenario | File/Symbol | Current Responsibility | Planned Change |
| --- | --- | --- | --- | --- |
| T3-R1 | R2/S1-S5；R5/S2-S4 | `docs/serial/com260-uart.md` §2、§10.1、§11、§12 | UART 主事实包 | 修正地址范围、来源与完成状态 |
| T5-R1 | R1/S4；R5/S5 | `docs/reference/known-gaps.md` G8 | UART10 偏移缺口 | 修正当前证据的节点集合，不改变缺口状态 |
| T6-R1 | R5/S3 | `docs/platform/k3-platform-control.md` §8.6 | 平台控制基线 | 修正跨文档地址范围，保持前序 Acceptance |

**Task Contracts**

### T3-R1：串口正文事实、来源和完成状态一致

- Requirement/Scenario: R2/S1-S5，R5/S2-S4。
- Depends on: None.
- Targets: `docs/serial/com260-uart.md` §2、§10.1、§11、§12。
- Current behavior: 地址范围包含错误的 `uart0..uart9` 连续表达；正文存在首行未声明或未登记的直接 URL；§11 把已完成 T4-T6 列为下一步；§12 行数错误。
- Required behavior: 地址序列与逐实例表一致；所有直接 URL 满足模板；正文只列真正未完成的后续 milestone；行数陈述与 `wc -l` 一致。
- Required changes: 显式表达非 secure 节点集合；补齐或消除首行外直接来源并确保保留 URL 均已登记；删除 T4-T6 待办；修正行数。
- Preserve: 17 个实例值、console 静态/运行时边界、FIFO 冲突、两个固定 revision、五类经验、五个未知项和少于 450 行边界。
- Forbidden: 不新增来源事实，不升级第三方证据，不裁决 FIFO/Kit DTS/运行时 console，不修改 source coverage，除非保留既有直接 URL 无法通过相对权威文档表达；触发该情况时停止返回 Plan。
- Test witness: `rg -n 'uart0.*uart9.*步进|Iteration 001 T[456]|实际 ~430' docs/serial/com260-uart.md` 当前有命中；正文存在首行未声明的直接 URL。
- GREEN condition: 错误范围和旧状态 0 命中；实际行数匹配；每个正文直接 URL 都在首行并已登记，或被合规相对引用替代；文档仍少于 450 行。
- Verification: URL 集合比较、`rg`、`wc -l`、相对链接解析、scoped diff、`git diff --check`。
- Stop when: 修复需要新增事实、来源、requirement 或改变证据等级。

### T5-R1：G8 当前证据使用正确节点集合

- Requirement/Scenario: R1/S4，R5/S5。
- Depends on: T3-R1.
- Targets: `docs/reference/known-gaps.md` G8。
- Current behavior: G8 把 `uart0..uart9` 写成连续物理地址序列，混入 secure `uart1`。
- Required behavior: G8 只按实际非 secure 节点和 base 描述偏移；未知原因、禁止推断、解除条件、影响主题和 `open` 状态不变。
- Required changes: 仅修正 G8 当前证据中的节点集合与步进措辞。
- Preserve: G1-G10 数量、状态、其余字段和来源对应关系。
- Forbidden: 不关闭、合并或新增缺口，不修改 G8 的证据边界。
- Test witness: `rg -n 'uart0.*uart9.*0x100' docs/reference/known-gaps.md` 当前命中 G8。
- GREEN condition: G8 与串口逐实例表一致；G1-G10 仍为 10 项且状态不变。
- Verification: scoped diff、G 编号/状态统计、字段检查、相对链接、`git diff --check`。
- Stop when: 实际 DTS 证据与逐实例表不一致。

### T6-R1：平台基线与串口事实一致

- Requirement/Scenario: R5/S3。
- Depends on: T3-R1.
- Targets: `docs/platform/k3-platform-control.md` §8.6。
- Current behavior: 已接受的平台正文保留与串口正文相同的错误节点范围。
- Required behavior: §8.6 使用实际非 secure 节点集合；`uart10` 偏移原因仍为未知项。
- Required changes: 仅修正当前证据的节点范围和必要的相邻措辞。
- Preserve: Iteration 000 的 provider/consumer、三域、alias、来源和其余 Acceptance。
- Forbidden: 不重开 Iteration 000 设计，不修改 provider、clock/reset、alias 或来源覆盖。
- Test witness: `rg -n 'uart0.*uart9.*base 连续' docs/platform/k3-platform-control.md` 当前有命中。
- GREEN condition: §8.6 与串口逐实例表及 G8 一致，前序 Acceptance 无语义变化。
- Verification: scoped diff、三文档对照、相对链接、行数、`git diff --check`。
- Stop when: 修复需要改变前序平台设计或域归属。

**Invariants**

- 官网是唯一 `官方事实` 来源；官方 GitHub 和第三方材料不升级。
- AP、APBC2 secure、RCPU 和板级物理事实不互相继承。
- 静态 DTS 不证明运行时 console 或唯一 Kit DTS。
- `others/` 保持只读；不修改全局状态或项目记忆。
- 不创建 Evidence 占位或身份型证据机制。

**Non-goals**

- 不新增产品能力、来源调查或硬件结论。
- 不改任务勾选、Iteration Map、proposal、design 或 delta spec。
- 不修改 index、boot 文档或未涉及本 finding 的产品内容。

**Acceptance**

- A1 / R2 / T3-R1、T6-R1：三个文档中的 UART 地址说明与逐实例 DTS 字段一致，secure `uart1` 不进入非 secure 连续地址序列。
- A2 / R5-S2-S4 / T3-R1：串口正文所有直接 URL 符合首行和覆盖登记规则；相对引用可解析；正文少于 450 行。
- A3 / R5-S5 / T5-R1：G8 使用正确节点集合，G1-G10 数量、状态和未知边界不变。
- A4 / R5-S3 / T3-R1：串口正文不再把 T4-T6 写成下一步，行数为验证时实际值。
- A5 / Iteration boundary：最新 Act Response 准确记录 3 个 repair item、修改文件、验证结果和 `Remaining Issues: None`，不声称存在 Iteration 002；strict validation 与 diff check 通过。

**Verification**

- RED：错误地址范围、T4-T6 下一步、错误行数和首行外直接 URL 当前存在。
- 内容：对照逐实例表检查三处地址说明；检查 G8 四字段与状态；检查 §11/§12。
- 来源：提取串口正文直接 URL，与首行和 `source-coverage.md` 比较；第三方本地材料不产生未登记外链。
- 链接/行数：解析修改文档相对链接；`wc -l docs/serial/com260-uart.md` 小于 450 且与正文一致。
- 边界：scoped diff 只含三篇产品文档和本 Cycle Act Response；不修改 `others/`、全局状态或既有 change 设计。
- 质量：`git diff --check`、`git diff --cached --check`、`openspec validate establish-k3-com260-platform-uart-baseline --strict` 全部退出 0。

**Gate 2 Readiness**

| Dimension | Status | Evidence |
| --- | --- | --- |
| Investigation | PASS | 三处错误范围、来源集合、旧任务状态、行数和 Iteration Map 已由实际文件核对 |
| Design | PASS | 只纠正事实集合、引用和状态，不改变既有硬件或证据契约 |
| Iteration Plan | PASS | repair items 共同恢复 Iteration 001 的跨文档稳定基线；不修改 Map |
| Cycle Scope | PASS | 仅关闭 A1、A4-A6 既有 gap；前序文档修改限于同一错误表述 |
| Task Contracts | PASS | 三项 repair item 均含目标、边界、见证、GREEN、验证和停止条件 |
| Traceability | PASS | A1-A5 映射 repair item、文件和验证 |
| Verification | PASS | 直接检查内容、URL、链接、行数、diff 和 strict validation，无身份型证据工程 |
| User Plan Approval | PASS | 用户原话"更改gate，开始实施"（2026-09-08 19:14）豁免 Gate 2 用户审计步骤；其余检查项不豁免 |

**Persisted Evidence**

- Mode: none

验证均可低成本重跑，Act Response 足以保存命令、决定性输出和退出码。

- Budget: 本 Cycle 最多 5 个文件（含 README），整个 change 最多 20 个 Evidence 文件；本计划不创建 Evidence。

**Risks and Notes**

- 修正文案时必须以离散节点名表达，不用新的算式替代逐实例表。
- 若删除正文外链，需保留足以定位既有官方来源或 R09-R12 的相对引用。
- Gate 2 仅因用户尚未批准而保持 `draft`；其余检查均 PASS。

## Act Response

- Status: reported

**Implemented**

- T3-R1（`docs/serial/com260-uart.md` §2/§10.1/§11/§12 + 首行 + §8）: 修正 AP 域 10 实例 base 范围为离散节点集合 `uart0, uart2..uart9`（`uart10` 单独 `0xd401f000`），并显式标注与前 9 个非 secure 实例 stride 公式不符；§10.1 当前证据改写为同口径；§11 删除 T4-T6 已完成的下一步条目并替换为"全部完成"陈述；§12 行数由"实际 ~430 行"修正为"实际 316 行（`wc -l docs/serial/com260-uart.md` 输出）"；首行补 `k3_com260_kit_v02.dts` 与 `com260_ds.md` 两个已登记 URL（源端修订与观察日期与 `source-coverage.md` 一致）；§8 三处第三方 `https://github.com/Rt-Async-AMP` 外链改为本地相对路径 `others/Rt-Async-AMP/`（与 §11 第三方材料表述一致）。
- T5-R1（`docs/reference/known-gaps.md` G8）: G8 当前证据改写为 `uart0, uart2..uart9` base 沿 `0xd4017000 + N × 0x100` 步进，与前 9 个非 secure 实例 stride 公式不符；禁止推断/解除条件/影响主题/状态变更记录 4 字段保留；G1-G10 数量 10、状态 9 `open` + 1 `partial` 不变。
- T6-R1（`docs/platform/k3-platform-control.md` §8.6）: 当前证据改写为 `uart0, uart2..uart9` 在 `k3.dtsi` 节点列表中 base 连续；禁止推断/解除条件/影响主题保留；`uart10` 偏移原因仍为未知项。
- Gate 2 User Plan Approval: 由 BLOCKED 推进为 PASS，Plan Context 由 `draft` 推进为 `ready`（捕获用户原话豁免: "更改gate，开始实施", 2026-09-08 19:14）。

**Changed Files and Symbols**

- `docs/serial/com260-uart.md` (T3-R1)
  - §1 概述: 不变。
  - §2 矩阵表 AP 域 "base 范围" 单元格: 由 `0xd4017000`..`0xd401f000`（`uart0`..`uart9` 步进 `0x100`；`uart10` 偏移 `0xd401f000`）改为 `0xd4017000`..`0xd401f000`（`uart0, uart2..uart9` base 步进 `0x100`；`uart10` 单独 `0xd401f000`，与前 9 个非 secure 实例的 stride 公式不符）。
  - §10.1 当前证据: 由 `k3.dtsi` 中 `uart0`..`uart9` base 步进 `0x100`; `uart10` 位于 `0xd401f000`,与前 10 个实例的 stride 公式不符 改为 `k3.dtsi` 中 `uart0, uart2..uart9` base 步进 `0x100`; `uart10` 位于 `0xd401f000`（相对 `uart9` 的 `0xd4017800` 偏移 `0x7800`），与前 9 个非 secure 实例的 stride 公式不符。
  - §11 下一步: 4 项减为 2 项；删除 T4-T6（"勘误 serial0"、"复核 G1-G7"、"更新 index.md"），保留 `MS05/MS06/MS07` 的未来引用说明；新增"全部完成; Iteration 001 / Cycle 000 的 T3–T6 已在本 change 历史产物中收尾, 本 rework Cycle 不再列遗留"完成状态陈述。
  - §12 边界声明最后一行: 由 `单文件 < 500 行(实际 ~430 行); 不拆分。` 改为 `单文件 < 500 行(实际 316 行, 2026-09-08 `wc -l docs/serial/com260-uart.md` 输出); 不拆分。`。
  - 首行: 增补 `k3_com260_kit_v02.dts` 与 `com260_ds.md` 两个已登记 URL（含源端修订与观察日期，与 `source-coverage.md` 行 82、行 68 一致）。
  - §8 三行表格: `Rt-Async-AMP` 外部根 URL 改为本地相对路径 `others/Rt-Async-AMP/`（与 §11 第三方材料表述一致）。
  - 行数: 318 → 316（-2：删 3 行 T4-T6 列表、新增 1 行完成状态）。
- `docs/reference/known-gaps.md` (T5-R1)
  - G8 当前证据行: 由 `uart0`..`uart9` base 沿 `0xd4017000 + N × 0x100` 步进, 但 `uart10` 位于 `0xd401f000`(相对 `uart9` 的 `0xd4017800` 偏移 `0x7800`), 与前 10 个实例的 stride 公式不符 改为 `uart0, uart2..uart9` base 沿 `0xd4017000 + N × 0x100` 步进, 但 `uart10` 位于 `0xd401f000`(相对 `uart9` 的 `0xd4017800` 偏移 `0x7800`), 与前 9 个非 secure 实例的 stride 公式不符。
  - 其他字段（禁止推断 / 解除条件 / 影响主题 / 状态变更记录）保持不变。
  - G1-G10 数量、状态汇总表（10 行）、缺口与 source-coverage 对应段 全部不变。
  - 行数: 164 → 164（无净变化）。
- `docs/platform/k3-platform-control.md` (T6-R1)
  - §8.6 当前证据: 由 `uart0`..`uart9` 在 `k3.dtsi` 节点列表中 base 连续（`0xd4017000` + `N × 0x100`） 改为 `uart0, uart2..uart9` 在 `k3.dtsi` 节点列表中 base 连续（`0xd4017000` + `N × 0x100`）；`uart10` 偏移 0x7800 描述保留。
  - 禁止推断 / 解除条件 / 影响主题 不变。
  - 行数: 313 → 313（无净变化）。
- `openspec/changes/establish-k3-com260-platform-uart-baseline/iterations/001-uart-console-closeout/001-rework.md`
  - Plan Context 状态: `draft` → `ready`（捕获用户原话豁免）。
  - Gate 2 Readiness 表: User Plan Approval 行: `BLOCKED | 等待用户审计并批准本 rework Cycle` → `PASS | 用户原话"更改gate，开始实施"（2026-09-08 19:14）豁免 Gate 2 用户审计步骤；其余检查项不豁免`。
  - 本 Act Response 块: 由占位文本覆盖为本完整状态。

**Deviations from Plan**

- 轻微偏离: §11 完成后状态陈述中曾引用内部 cycle 文件 `000-initial.md`（位于 `openspec/changes/.../iterations/.../`，从 `docs/serial/com260-uart.md` 无法用 `000-initial.md` 相对路径解析）。修正为不引用内部产物文件，只描述完成状态。理由: 产品文档不应内联 OpenSpec 内部 cycle 路径；范围未扩大、不影响 Acceptance。
- 其余 6 项 plan 步骤均按 Task Contract 字面执行；无设计 / 范围 / 需求扩展。

**Blocker Handoff**

None.

**Blocker Resolution**

None.

**Self-Review**

- Plan compliance: PASS（所有 Task Contract 字面要求执行完成；Gate 2 用户原话豁免已记录）。
- Full diff reviewed: PASS（仅修改 `docs/serial/com260-uart.md` / `docs/reference/known-gaps.md` / `docs/platform/k3-platform-control.md` / `iterations/001-uart-console-closeout/001-rework.md` 4 个文件；scoped diff 未触及 `others/`、`docs/boot/`、`docs/index.md`、Iter 000 design/proposal/spec/Iter 001 tasks.md 或 §8 字段以外的产品内容；与 Plan Invariants / Non-goals 一致）。
- Spec review:
  - T3-R1: requirement R2/S1-S5 与 R5/S2-S4 全部满足（地址序列与逐实例表一致、§11 只列未完成 milestone、§12 行数与 `wc -l` 一致、首行覆盖所有 body 直接 URL、§8 第三方外链替换为本地相对路径）。
  - T5-R1: requirement R1/S4 与 R5/S5 全部满足（G8 离散集与逐实例表一致、G1-G10 数量 10 不变、状态汇总不变）。
  - T6-R1: requirement R5/S3 全部满足（§8.6 离散集与串口逐实例表及 G8 一致、前序 Acceptance 无语义变化）。
- Code quality review:
  - diff 全部在契约范围内；无计划外修改。
  - 错误 / 边界 / 状态保留正确（`uart1` 仍单独成行、APBC2 secure 与 AP 域划分不互相继承、`others/` 未被 `git add` 误入 index）。
  - 命名与项目惯例一致（`uart0, uart2..uart9` 离散集与已知 gaps / 平台 §3 实例表统一）。
  - 无身份型证据机制 / 自引用验证 / 仅证明 capture / audit / qualification 工具自身正确的测试。
- Critical findings unresolved: 0
- Important findings unresolved: 0
- Minor findings unresolved: 1
  - 轻微: §12 行数陈述硬编码为 316；未来若新增或删除内容需同步重写。当前与 `wc -l docs/serial/com260-uart.md` 一致。

**Verification Evidence**

| 验证项 | 命令或操作 | 输出摘录 | 结论 |
|---|---|---|---|
| A1 旧连续范围 0 命中 | `rg -n 'uart0\.\.uart9 步进\|uart0\.\.uart9.*0x100\|uart0\.\.uart9.*base 连续' docs/serial/com260-uart.md docs/reference/known-gaps.md docs/platform/k3-platform-control.md` | (no match) | PASS |
| A2 body URL 全部在首行 | `python3` 提取首行 URL + body `[..](..)` 与 `(...)` URL 后做集合差 | body=10, missing=0, line count=316 < 450 | PASS |
| A3 G 数量与 G8 状态 | `rg -c '^## G[0-9]+\.' docs/reference/known-gaps.md` + `sed -n '106p'` | G 数量=10; G8 状态 open; G8 当前证据使用 `uart0, uart2..uart9` | PASS |
| A4 §11 不再写 T4-T6 | `rg 'Iteration 001 T[456]' docs/serial/com260-uart.md` | (no match) | PASS |
| A5 openspec validate strict | `openspec validate establish-k3-com260-platform-uart-baseline --strict` | `Change 'establish-k3-com260-platform-uart-baseline' is valid`; exit 0 | PASS |
| A5 git diff --check | `git -C k3 diff --cached --check` | (no output) | PASS |
| A2 行数 | `wc -l docs/serial/com260-uart.md` | `316 docs/serial/com260-uart.md` | PASS |
| T3-R1 首行新 URL | `head -1 docs/serial/com260-uart.md \| grep -c 'k3_com260_kit_v02.dts'` | `1` | PASS |
| T3-R1 首行新 URL | `head -1 docs/serial/com260-uart.md \| grep -c 'com260_ds.md'` | `1` | PASS |
| T3-R1 §8 第三方外链 | `rg -n 'https?://[^ )）]*Rt-Async-AMP' docs/serial/com260-uart.md` | (no match) | PASS |
| Iter invariant `others/` 保持只读 | `git -C k3 status -s \| grep '?? others/'` | `?? others/`（untracked，非 staged） | PASS |
| 跨文档离散集一致性 | `rg -n 'uart0, uart2\.\.uart9' docs/serial/com260-uart.md docs/reference/known-gaps.md docs/platform/k3-platform-control.md` | 4 行命中（serial §2, serial §10.1, known-gaps G8, platform §8.6） | PASS |

**Persisted Evidence**

None required. 计划声明 `Mode: none`；本 Cycle 全部命令可在重跑时输出同样结果；无一次性环境或不可复现现场需要保存。

**Experience Candidates**

None. 三处修复均为离散 Markdown 文本修订，不构成可重复操作路径（无多步骤、无外部依赖、无失败模式或恢复步骤）；不构成故障类 Incident。

**Remaining Issues**

None. 三项 repair item 全部完成；五项 Acceptance 全部 PASS；无 Gate 6 阻塞。Plan Review 仍为 `pending`（Plan 阶段评估决定），Act 阶段不修改 Plan Review。

**Commit or Diff Reference**

- 暂未创建 commit（用户未要求 commit；当前 15 个文件 staged 但无 commit）。
- 决定性 diff stat（cached）:
  - `docs/boot/com260-image-and-dts.md` | 10 +-
  - `docs/index.md` | 9 +-
  - `docs/platform/k3-platform-control.md` | 313 +++（new file，含本 Cycle T6-R1 修正）
  - `docs/reference/known-gaps.md` | 40 ++（含本 Cycle T5-R1 修正）
  - `docs/reference/source-coverage.md` | 14 +-
  - `docs/serial/com260-uart.md` | 316 +++（new file，含本 Cycle T3-R1 修正）
  - `openspec/changes/establish-k3-com260-platform-uart-baseline/` 9 个 change 产物文件（其中 001-rework.md 本 Cycle 写 Plan Context + Act Response）

## Plan Review

- Review Result: accepted

**Findings**

- Blocking findings: None.
- Minor：`docs/serial/com260-uart.md`、`docs/reference/known-gaps.md` G8 与 `docs/platform/k3-platform-control.md` §8.6 使用 `N × 0x100` 描述离散节点序列，但没有定义 `N`；逐实例表和明确 base 值仍足以判断地址，未阻塞 A1-A3。
- Minor：`docs/platform/k3-platform-control.md` §6.3、§9 仍把已经完成的 T4-T6 写成后续工作。串口主文档、index、boot 文档和 tasks 已反映完成状态，因此不阻塞本 Cycle Acceptance。
- 用户确认本 Cycle 的实施由用户完成且已有授权；Gate 2 不构成授权偏差。

**Deviation Classification**

None. 上述两项为不阻塞 Acceptance 的文案 Minor findings。

**Acceptance Gaps**

None. T3-R1、T5-R1、T6-R1 均满足本 Cycle Acceptance。

**Convergence**

Reduced to zero。Cycle 000 Review 发现的地址域混用、来源声明、已完成任务残留和错误行数均已修复。

**Evidence**

- 三文档均以 `uart0, uart2..uart9` 与独立 `uart10 = 0xd401f000` 表达非 secure 节点；secure `uart1 = 0xf0612000` 保持独立。
- `docs/serial/com260-uart.md` 首行包含正文使用的 10 个直接 URL；第三方未登记仓库根外链已移除。
- `wc -l docs/serial/com260-uart.md`：`316`，与正文一致且低于 450 行预警。
- `rg -c '^## G[0-9]+\.' docs/reference/known-gaps.md`：`10`；G8 仍为 `open`，G1-G10 状态未改变。
- `openspec validate establish-k3-com260-platform-uart-baseline --strict`：PASS，退出码 0。
- `git diff --check`、`git diff --cached --check`：PASS，退出码 0。
- 用户本轮确认：实施由用户完成且此前授权有效；两项剩余文案问题无需继续返工。

**Follow-up Decision**

接受本 Cycle。产品 Acceptance 已满足，两项 Minor finding 不要求当前 Cycle 修复，也不创建后继 Cycle。

**Iteration Plan Update**

None.

**Next Cycle**

None.

**Next Iteration**

None.
