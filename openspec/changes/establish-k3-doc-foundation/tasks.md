## 1. Iteration 000 — 覆盖与结构

- [x] 1.1 [T1] 最小修复 `openspec/config.yaml` 第 51 行的 YAML quoting，并证明 CLI 加载既有 artifact rules。
- [x] 1.2 [T2] 创建 `docs/index.md`，提供当前范围、参考文档导航和九类未来主题职责。
- [x] 1.3 [T3] 创建 `docs/reference/source-coverage.md`，唯一覆盖 R01、R04-R08 当前登记的 38 个 URL。
- [x] 1.4 [T4] 创建 `docs/reference/document-template.md`，固定首行来源、多来源、证据强度和拆分格式。
- [x] 1.5 [T5] 创建 `docs/reference/terminology.md`，固定基础术语、英文原词与别名。
- [x] 1.6 [T6] 创建 `docs/reference/known-gaps.md`，登记当前可验证的来源与硬件资料缺口。

## 2. Iteration 001 — 刷新与一致性

- [ ] 2.1 [T7] 创建 `docs/reference/source-refresh.md`，定义人工刷新状态、操作顺序、中断恢复和 change 边界。
- [ ] 2.2 [T8] 更新 `docs/index.md`，在刷新指南存在后加入有效导航并完成全量一致性检查。

## Task Contracts

### T1：OpenSpec 项目规则可加载

- Requirement/Scenario: OpenSpec 项目规则可被 CLI 加载 / 请求 change artifact instructions、修复配置语法。
- Depends on: None.
- Targets: `openspec/config.yaml` 第 51 行。
- Current behavior: `openspec instructions proposal --change establish-k3-doc-foundation` 退出码为 0，但报告 `Implicit map keys need to be followed by map values at line 51` 并忽略项目配置。
- Required behavior: CLI 不再报告 config parse warning，且 instructions 输出包含现有 proposal、tasks 和 spec project rules。
- Required changes: 仅为第 51 行整个规则标量增加必要 YAML quoting。
- Preserve: 配置中的 schema、context 和所有规则文字语义。
- Forbidden: 不重排配置、不改写规则、不升级 schema 或 CLI。
- Test witness: 变更前运行上述 instructions 命令可观察 parse warning；退出码 0 不能单独判定通过。
- GREEN condition: stderr 无 parse warning，输出含 `State the source page or section`、`Each task targets a single topic-level document` 和 `Knowledge entries` 三条既有规则。
- Verification: 运行 `openspec instructions proposal --change establish-k3-doc-foundation`、`openspec instructions tasks --change establish-k3-doc-foundation`、`openspec instructions specs --change establish-k3-doc-foundation`；三者均加载项目规则且不出现 warning。
- Stop when: 修复需要改动第 51 行之外的配置语义，或 CLI 仍因另一处 YAML 问题忽略配置。

### T2：文档总入口可导航

- Requirement/Scenario: 聚合内容使用主题驱动导航 / 读者从总入口定位资料。
- Depends on: T1.
- Targets: `docs/index.md`。
- Current behavior: `docs/` 不存在，读者没有入口或主题职责说明。
- Required behavior: 总入口链接本 Iteration 已存在的 source coverage、document template、terminology 和 known gaps，并说明 platform、boot、interrupts、serial、dma、network、storage、buses 和 peripherals 的职责、优先级与当前是否已有正文。
- Required changes: 创建唯一入口；明确当前目标为 K3 CoM260 Kit，其他 K3 板卡 deferred；仅对实际存在的文件使用相对链接，未来 `source-refresh.md` 和主题路径以代码样式展示。
- Preserve: D02 主题驱动结构和 M01 的 K3 来源边界。
- Forbidden: 不创建空主题目录或占位 overview；不加入 CoM260 技术结论。
- Test witness: `test -f docs/index.md` 当前退出码为 1。
- GREEN condition: 文件存在、首行符合 D3 格式，四个当前 reference 相对链接可解析，九类主题职责均可从入口定位。
- Verification: 检查首行、相对链接目标和主题职责列表；任何链接目标不存在即失败。
- Stop when: 入口需要引用本 change 范围外的技术正文才能成立。

### T3：来源覆盖完整且唯一

- Requirement/Scenario: 已发现来源具有完整覆盖记录 / 正常登记、不可访问或版本未知、非目标板资料。
- Depends on: T1.
- Targets: `docs/reference/source-coverage.md`。
- Current behavior: R01、R04-R08 共登记 38 个唯一 URL，但没有逐 URL 的范围、主题、优先级、观察和访问状态。
- Required behavior: 每个当前 URL 恰好一行；字段符合 D2；CoM260/K3-common、其他板卡和 supporting source 明确区分。
- Required changes: 从 references 转录 URL 和既有元数据；使用 2026-09-02 作为已有观察日期；没有源端修订时写 `unknown`；不可由当前证据确认访问状态时写 `unverified`。
- Preserve: R01 仍是正文权威入口；R08 只作 supporting source；原 URL 原样保留。
- Forbidden: 不补写未登记 URL，不猜测 revision 或访问结果，不修改 references。
- Test witness: `test -f docs/reference/source-coverage.md` 当前退出码为 1；references 当前唯一 URL 数为 38。
- GREEN condition: references 输入集合与覆盖表 URL 集合完全相同，且覆盖表内无重复；每行必填字段没有空值。
- Verification: 分别提取 R01、R04-R08 和覆盖表中的 URL，排序去重后使用 `comm -3` 比较，输出必须为空；覆盖表 URL 总数与唯一数均为 38。
- Stop when: R01、R04-R08 在执行前改变，或发现同一 URL 必须表达两个互斥目标职责。

### T4：后续文档格式无歧义

- Requirement/Scenario: 聚合内容使用主题驱动导航 / 新建聚合文档、一个主题依赖多个来源；术语和证据强度可区分 / 资料不足。
- Depends on: T1.
- Targets: `docs/reference/document-template.md`。
- Current behavior: M02 规定首行来源、kebab-case 和长度边界，但没有可直接复用的单来源、多来源、事实强度及未知项示例。
- Required behavior: 模板明确 D3 首行格式，并展示单来源、多来源、事实、交叉验证、推论、未知项、目录和超过 500 行时的拆分方式。
- Required changes: 提供简短可复制的 Markdown 骨架和字段说明；源端 revision 与本地 observation 分列表达。
- Preserve: M01-M04、D01-D02；中文为主且保留原始技术术语。
- Forbidden: 不把示例值写成真实 K3 硬件事实；不要求生成脚本或 Evidence 文件。
- Test witness: `test -f docs/reference/document-template.md` 当前退出码为 1。
- GREEN condition: 模板覆盖全部必需格式，示例中 unknown 不会被观察日期替代，所有示例均符合首行来源规则。
- Verification: 人工对照 M01-M04、D01-D02 和 delta spec；检查首行及代码块示例。
- Stop when: 模板需要改变已接受的 M/D 约束。

### T5：基础术语保持一致

- Requirement/Scenario: 术语和证据强度可区分 / 同一术语存在多种写法。
- Depends on: T1, T4.
- Targets: `docs/reference/terminology.md`。
- Current behavior: 相关术语散落在 analysis、references 和 roadmap 中，没有主写法或别名入口。
- Required behavior: 至少覆盖 K3、CoM260 Kit、SoC、AP、RCPU、AIA、APLIC、IMSIC、MMIO、IRQ、DMA、IOMMU、GMAC、PHY、MDIO、RGMII、polling、async、waker 和 coherency；每项记录主写法、英文原词或缩写、允许别名和使用说明。
- Required changes: 只固定表达，不新增硬件结论；指出标题层应使用的主写法。
- Preserve: M03 的简体中文与英文专有名词策略。
- Forbidden: 不扩写成硬件原理教程，不为未确认术语编造中文译名。
- Test witness: `test -f docs/reference/terminology.md` 当前退出码为 1。
- GREEN condition: 所列 20 个术语均存在且没有互相冲突的主写法；本 change 新增文档按该表使用术语。
- Verification: 对术语清单逐项检索，并全文检查标题层是否出现未登记的同义并列。
- Stop when: 官方来源给出互斥命名且无法在“不改变事实”的前提下选择主写法。

### T6：未知项具有解除条件

- Requirement/Scenario: 术语和证据强度可区分 / 资料不足以支持硬件结论；已发现来源具有完整覆盖记录 / 页面不可访问或版本未知。
- Depends on: T3, T4.
- Targets: `docs/reference/known-gaps.md`。
- Current behavior: analysis 已指出多项资料缺口，但 `docs/` 没有供后续 change 复用的缺口入口。
- Required behavior: 至少记录官网总页数、未展开目录、CoM260 GMAC/PHY、AIA routing、DMA coherency/IOMMU 和 programmer reference 六类缺口；每项含当前证据、禁止推断内容、解除条件和影响主题。
- Required changes: 将 analysis 中仍有效的未知项转为简洁的产品文档记录，不复制实现建议。
- Preserve: 未知仍是未知；GitHub 或 Linux driver 只作交叉验证。
- Forbidden: 不登记没有证据的问题，不把推论提升为官方事实，不修改 M/D/K/I。
- Test witness: `test -f docs/reference/known-gaps.md` 当前退出码为 1。
- GREEN condition: 六类缺口全部具备解除条件和影响范围；与 source coverage 的 unknown/unverified 状态一致。
- Verification: 对照 R03 analysis 和 source coverage 逐项检查；发现无解除条件的缺口即失败。
- Stop when: 调查缺口要求进入 CoM260 技术正文或新增需求。

### T7：人工刷新流程可恢复

- Requirement/Scenario: 来源变化可以人工刷新 / unchanged、changed、moved/removed、人工刷新被中断。
- Depends on: T2, T3, T4, T5, T6.
- Targets: `docs/reference/source-refresh.md`。
- Current behavior: 只有 roadmap 对 refresh 的目标描述，没有可执行状态定义、顺序或部分完成恢复规则。
- Required behavior: 定义 unchanged、changed、moved、removed、unreachable；给出选择范围、读取来源、比较 revision/正文、更新行级状态、创建 change、更新正文与验证的顺序；部分检查不得改变未检查行。
- Required changes: 使用 Iteration 000 的覆盖字段和模板；明确初始 baseline 不算真实 refresh；以文字示例演练 unchanged、unreachable 和 partial-resume。
- Preserve: D01 手工整理、M04 无可执行工具、change 先于正文修改、R01 权威边界。
- Forbidden: 不创建运行 ID、manifest、hash 账本、脚本或 Evidence 目录；不更新 R01、tasks 或 SNAPSHOT。
- Test witness: `test -f docs/reference/source-refresh.md` 当前退出码为 1。
- GREEN condition: 五种状态、完整操作顺序、三种演练和停止条件均存在；中断后可凭行级元数据继续，不会把未检查来源标为 unchanged。
- Verification: 按指南对三个文字场景逐步检查结果；运行全量 Markdown 来源首行、相对链接和一致性检查。
- Stop when: 流程必须依赖仓库内自动化工具，或真实来源变化要求扩大为内容 refresh。

### T8：刷新入口完成集成

- Requirement/Scenario: 聚合内容使用主题驱动导航 / 读者从总入口定位资料；来源变化可以人工刷新 / 全部场景。
- Depends on: T7.
- Targets: `docs/index.md`。
- Current behavior: Iteration 000 只以代码样式预告 `docs/reference/source-refresh.md`，避免链接到不存在的文件。
- Required behavior: 刷新指南创建后，总入口提供有效相对链接并保持其他导航、范围和主题职责不变。
- Required changes: 只把 deferred 的刷新指南路径转换为相对链接，并执行全量文档一致性检查。
- Preserve: T2 已建立的 CoM260 范围、四个 reference 链接和九类主题职责。
- Forbidden: 不借集成导航改写其他文档或加入技术正文。
- Test witness: Iteration 000 基线中刷新路径不是链接；在 T7 前链接目标不存在。
- GREEN condition: `source-refresh.md` 链接存在并可解析，所有其他相对链接仍有效。
- Verification: 对 `docs/index.md` 中全部相对链接逐项执行 `test -f`，并重跑全量首行、OpenSpec 与 diff 检查。
- Stop when: T7 未完成，或更新需要改变 T2 的范围与导航设计。

## Iteration Plan

### Iteration 000: 覆盖与结构

- Tasks: T1, T2, T3, T4, T5, T6
- Depends on: None
- Stable baseline: OpenSpec 项目规则可加载；38 个 URL 完整归类；文档入口、模板、术语和缺口可供后续聚合直接复用。
- Verification boundary: config 无解析告警；URL 集合比较为空；五个 Iteration 000 文档存在且首行、相对链接、字段和术语检查通过。
- Diagnostic boundary: 配置解析、URL 覆盖、导航、格式、术语和缺口分别由单一 task/file 隔离。
- Non-goals: 不建立刷新操作指南；不聚合 CoM260 技术正文；不更新全局状态。
- Balance audit: 六个任务共同形成唯一“可开始聚合”的结构基线；单独交付模板或术语不能形成稳定结果，因此不继续拆 Iteration。配置修复虽小，但它是验证项目规则的必要前置，不单独形成 Iteration。

### Iteration 001: 刷新与一致性

- Tasks: T7, T8
- Depends on: Iteration 000
- Stable baseline: 维护者可以对覆盖表执行可中断、可继续、不会静默替换来源的人工复核。
- Verification boundary: 五种刷新状态、操作顺序和 unchanged、unreachable、partial-resume 演练通过；全量文档一致性检查通过。
- Diagnostic boundary: 失败只涉及刷新状态、操作顺序、部分完成恢复或与 Iteration 000 元数据不一致。
- Non-goals: 不等待或伪造真实官网版本变化；不执行 CoM260 技术内容刷新。
- Balance audit: T7 建立维护状态机，T8 只在该稳定结果存在后完成导航集成；二者共同形成可发现、可执行的刷新基线。并入 Iteration 000 会把首次内容结构问题与后续维护流程问题混为同一诊断域。
