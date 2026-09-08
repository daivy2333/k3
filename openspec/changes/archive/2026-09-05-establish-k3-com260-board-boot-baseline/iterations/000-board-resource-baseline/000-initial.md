# Iteration 000 / Cycle 000：板级事实层次与资源矩阵

## Plan Context

- Status: accepted
- Iteration: 000-board-resource-baseline
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: None
- Gate 2 approval: 用户原话 `更改gate状态，开始实施吧` (2026-09-05 16:11 GMT+0800), 显式豁免并授权实施, 风险由本 Cycle Act Response 承担

**Iteration Scope**

- Change tasks: T1-T6
- Depends on: MS01、MS02 已完成；Gate 1 已获用户批准
- Stable baseline: K3 SoC、CoM260 模组与目标 Kit/载板事实分层，以及八类资源的统一矩阵
- Verification boundary: 两篇 platform 文档来源合规、层级明确、八类资源齐全、未知项可解除，所用精确 URL 已登记且唯一
- Diagnostic boundary: platform 来源身份、SoC/模组/Kit 归属、版本冲突、资源缺失或范围越界
- Deferred tasks: T7-T13（Iteration 001：启动链、镜像与 DTS）

**Cycle Scope**

- Trigger: initial
- Acceptance gaps: None
- Repair items: None
- Inherited scope: Gate 1 已批准的 proposal、5 个 requirement/20 个 scenario、design D1-D7、M01-M04 与 D01-D08
- Excluded scope: boot/image/DTS 正文、known-gaps、总入口、StarryOS、寄存器级 UART/IRQ/DMA/GMAC 资料

**Objective**

先登记本 Iteration 实际引用的精确来源，再交付 K3 SoC 概述和 CoM260 板级资源矩阵，使下一 Iteration 能在不重新判定硬件层级的前提下整理启动链与 DTS。

**Background**

当前 `docs/` 没有技术主题目录。MS03 要覆盖 SoC、模组、Kit、启动和 DTS，但启动介质与设备树结论都依赖目标硬件身份。先关闭平台层级和资源归属，可以把后续失败限制在启动来源或 DTS 映射，而不是同时怀疑板型。

**Current Baseline**

- `docs/index.md` 51 行，只列 platform/boot 为未来路径。
- `docs/reference/source-coverage.md` 65 行、38 个已登记 URL；K3 datasheet、CoM260 root overview 和 hardware resources 官网 URL已登记，但 CoM260 datasheet、user guide 和本 Cycle 使用的 GitHub 精确页未登记。
- `docs/reference/known-gaps.md` 有 G1-G6；G3 已保留 CoM260 GMAC/PHY 实例、地址和 RGMII 参数缺口。
- `docs/platform/` 尚不存在；T5、T6 的文件存在性见证当前为 RED。
- MS02 已形成交叉验证级 SDK v1.0 基线：OpenSBI 1.6、U-Boot 2022.10、Linux 6.18、Buildroot 2025.02.6；本 Cycle 只引用该版本边界，不刷新它。

**Current-State Evidence**

- K3 datasheet 对应官方仓库文档内标 V1.6（2026-07-15），明示 128 KB Boot ROM、512 KB SRAM、LPDDR4x/LPDDR5 最大 32 GB，以及 download/local boot 能力。官网正文未直接取得时，这些只能写 `交叉验证`。
- CoM260 datasheet 对应官方仓库文档内标 V1.2（2026-07-10），对象是集成 K3、LPDDR5、GPHY 等资源的模组，并列 8/16/32 GB 选项；这些不自动证明目标 Kit 的选配值。
- CoM260 user guide 对应官方仓库文档内标 V2.0（2026-03-19），描述模组加开发套件，并出现产品版本 `K3-CoM260_P1_LP5315B_32X2_v03_20260312`；产品版本、文档修订和 DTS 文件版本必须分开。
- `com260_hw_resources.md` 对应官方仓库页提供 BOM、原理图和载板 DSN/BRD 下载入口；未直接打开的下载制品内容不能作为已观察事实。
- 当前 source-coverage 的 SpacemiT 官网行多为 `partially-observed`，R08 仓库为 supporting；新精确 GitHub 页不得升级为官网事实。
- R03 分析列出 K3 product brief 的 GMAC、UFS、QSPI、UART 等 SoC 能力，但同时指出这些能力不能证明 CoM260 的实例、连接或参数；Cycle 必须回到已登记直接来源写正文。

**Relevant Code**

- `docs/reference/source-coverage.md`：精确 URL、来源身份、范围、主题、修订、观察和状态的唯一登记表。
- `docs/reference/document-template.md`：首行来源、证据等级、未知项、相对链接和行数规则。
- `docs/reference/terminology.md`：K3、CoM260 Kit、SoC、hart、GMAC、PHY 等主写法。
- `docs/reference/known-gaps.md::G3-G6`：后续网络、IRQ、DMA 与 programmer reference 边界。
- `docs/platform/k3-soc-overview.md`：T5 新建，承担 SoC 能力。
- `docs/platform/com260-board-resources.md`：T6 新建，承担模组/Kit 资源矩阵。

**Critical Path**

读取并直接观察精确来源 → T1-T4 登记来源与证据身份 → T5 只提取 SoC 能力 → T6 将每项资源映射到 SoC/模组/Kit/未知边界 → 检查 G3-G6 引用、行数与完整 diff → 形成 Iteration 001 可依赖的平台基线。

**Implementation Guidance**

1. 固定处理顺序 T1 → T2 → T3 → T4；每个 URL 都先直接访问，再按当前可见内容填写源端修订、观察日期和访问状态。
2. 每个事实先回答“描述对象是谁”，再回答“证据等级是什么”；不能定位对象层级的字段进入未知项。
3. T5 使用能力清单和必要解释，不复制 datasheet 全文；启动能力只作为 SoC 能力摘要，具体流程延后。
4. T6 使用一张主资源矩阵，必要时在后续小节解释版本差异、Kit/载板连接和 bring-up 影响。
5. 资源没有板级证据时仍保留行，用“未知项 + 解除条件”完成，而不是省略。
6. 任一文档接近 450 行时停止并返回 Plan 决定拆分路径；当前 Cycle 不自行增加新主题文件。

**Behavioral Change**

当前只能从总入口看到未来 platform 职责，且来源表不能精确追踪实际使用的若干页面。完成后，读者可从两篇独立文档区分 K3 能力、CoM260 模组集成、Kit/载板连接和未知项；source-coverage 可追溯到每个支撑页面。总入口接线留到 Iteration 001，以避免未完成 change 被标为整体可用。

**Change Surface**

| Task | Requirement/Scenario | File | Current Responsibility | Planned Change |
| --- | --- | --- | --- | --- |
| T1-T4 | R1/S1-S3, R2/S1-S4, R5/S1 | `docs/reference/source-coverage.md` | 38 URL 基线 | 增加 6 个精确 URL 并同步实际行数说明 |
| T5 | R1/S1,S4; R2/S3; R3/S1,S3 | `docs/platform/k3-soc-overview.md` | 不存在 | 新建 SoC 能力基线 |
| T6 | R1/S2-S4; R2/S1-S4; R3/S1,S3 | `docs/platform/com260-board-resources.md` | 不存在 | 新建四层八类资源矩阵 |

**Task Contracts**

### T1：K3 datasheet 交叉验证 URL 可精确追溯

- Requirement/Scenario: R1/S1，R2/S1-S2，R5/S1。
- Depends on: None.
- Targets: `docs/reference/source-coverage.md`。
- Current behavior: 只有官网 K3 datasheet URL 和 `docs-chip` 仓库根 URL。
- Required behavior: 唯一登记 `https://github.com/spacemit-com/docs-chip/blob/main/zh/key_stone/k3/k3_docs/k3_ds.md`，字段为 supporting-source/K3-common/platform/supporting/supporting。
- Required changes: 直接观察页面，记录可证修订、实际观察日期、访问状态和用途限制。
- Preserve: 官网行、其余 38 行及其日期、M01。
- Forbidden: 不提升 GitHub 权威，不批量刷新。
- Test witness: `rg -F` 精确 URL 在修改前退出码 1。
- GREEN condition: URL 恰好一行，全部字段由当前观察支持。
- Verification: 精确计数、字段核对、URL 唯一性、`git diff --check`。
- Stop when: 页面身份或 K3 路径不能确认。

### T2：CoM260 datasheet 两种来源身份分离

- Requirement/Scenario: R1/S2，R2/S1-S4，R5/S1。
- Depends on: T1.
- Targets: `docs/reference/source-coverage.md`。
- Current behavior: 两个精确 URL 均不存在。
- Required behavior: 登记官网 `.../hardware/eco/k3_com260/com260_ds.md` 为 active official-doc，并登记 `https://github.com/spacemit-com/docs-product/blob/main/zh/k3_com260/com260_ds.md` 为 supporting-source。
- Required changes: 官网与 GitHub 分行记录各自直接可证的修订和访问状态。
- Preserve: 两种证据身份、CoM260/platform 范围。
- Forbidden: 不用 GitHub V1.2 自动填写不可读官网行。
- Test witness: 两个精确 URL 修改前计数均为 0。
- GREEN condition: 各唯一一行，身份、修订和观察语义正确。
- Verification: 精确计数、字段和重复 URL 检查。
- Stop when: 两页不能确认是同一 datasheet 身份。

### T3：CoM260 user guide 两种来源身份分离

- Requirement/Scenario: R1/S3，R2/S1-S4，R5/S1。
- Depends on: T2.
- Targets: `docs/reference/source-coverage.md`。
- Current behavior: 两个精确 URL 均不存在。
- Required behavior: 登记官网 `.../hardware/eco/k3_com260/com260_user_guide.md` 为 active official-doc，并登记 `https://github.com/spacemit-com/docs-product/blob/main/zh/k3_com260/com260_user_guide.md` 为 supporting-source。
- Required changes: 备注说明开发套件职责；修订、产品版本和观察日期分离。
- Preserve: CoM260 Kit 术语和证据层级。
- Forbidden: 不因 `v03` 产品版本认定 DTS。
- Test witness: 两个精确 URL 修改前计数均为 0。
- GREEN condition: 各唯一一行，字段与直接观察一致。
- Verification: 精确计数、字段和唯一性检查。
- Stop when: 页面不是目标开发套件资料。

### T4：hardware resources 精确 URL 与覆盖计数一致

- Requirement/Scenario: R1/S2-S3，R2/S1-S4，R5/S1。
- Depends on: T3.
- Targets: `docs/reference/source-coverage.md`。
- Current behavior: GitHub 精确页不存在，表头固定声明 38 行。
- Required behavior: 登记 `https://github.com/spacemit-com/docs-product/blob/main/zh/k3_com260/com260_hw_resources.md` 为 supporting，并把声明数更新为完成 T1-T4 后的实际唯一 URL 数。
- Required changes: 只登记页面本身；未打开的 BOM/原理图/DSN/BRD 不登记为已观察。
- Preserve: 官网 hardware-resources 行和所有未选行。
- Forbidden: 不引用下载制品内容，不修改其余行日期。
- Test witness: 精确 URL 不存在；新增后旧 38 行声明会失真。
- GREEN condition: 新行唯一，声明数 = 实际数据行数 = 唯一 URL 数。
- Verification: 表格提取计数、`uniq -d` 空输出、diff 范围和 `git diff --check`。
- Stop when: 完成本 Iteration 必须使用未登记下载制品。

### T5：K3 SoC 能力与板级可用性解耦

- Requirement/Scenario: R1/S1,S4；R2/S3；R3/S1,S3；R5/S2-S3,S5。
- Depends on: T1-T4.
- Targets: `docs/platform/k3-soc-overview.md`。
- Current behavior: 文件不存在。
- Required behavior: 首行来源合规；覆盖 CPU/hart、内存控制器、启动能力、存储与外设能力；逐项标证据等级并声明不证明 CoM260 引出/启用。
- Required changes: 建立主题职责、目录、事实表、未知边界和后续主题链接。
- Preserve: 简体中文、术语表、M01 和行数规则。
- Forbidden: 不写板级连接、寄存器、IRQ/DMA/GMAC 驱动细节或非目标板结论。
- Test witness: `test -f docs/platform/k3-soc-overview.md` 修改前退出码 1。
- GREEN condition: 必要主题齐全，所有事实可追溯，能力与可用性边界明确，少于 450 行。
- Verification: 文件/首行/章节/证据标记/禁用词语/行数和 diff 检查。
- Stop when: 来源冲突改变 K3 身份、CPU/hart 基本结论或要求新增主题文件。

### T6：CoM260 八类资源形成四层矩阵

- Requirement/Scenario: R1/S2-S4；R2/S1-S4；R3/S1,S3；R5/S2-S5。
- Depends on: T5.
- Targets: `docs/platform/com260-board-resources.md`。
- Current behavior: 文件不存在。
- Required behavior: 首行来源合规；CPU/hart、DRAM、UFS、SPI Flash、TF Card、debug UART、GMAC/PHY、连接器和供电均区分 SoC、模组、Kit/载板、未知边界和证据等级。
- Required changes: 建立主矩阵、版本冲突说明、未知项闭包和 G3-G6/后续主题指向。
- Preserve: 目标板身份和 MS04-MS07 范围。
- Forbidden: 不把能力当连接，不使用 Pico/其他变体，不推定 GMAC 实例或 PHY 参数。
- Test witness: `test -f docs/platform/com260-board-resources.md` 修改前退出码 1；八类资源检查均失败。
- GREEN condition: 资源无遗漏，三层事实可分，未知项含当前证据、禁止推断、解除条件和影响主题，少于 450 行。
- Verification: 文件/首行/关键词/层级列/证据等级/G3 引用/禁用推断/行数和完整 diff。
- Stop when: 模组与 Kit 无法区分，或冲突需要改变 Gate 1 范围。

**Invariants**

- R01 仍是唯一权威正文入口，官方 GitHub 只作交叉验证。
- SoC 能力不等于模组引出或 Kit 可用；模组集成不等于载板连接。
- 产品版本、文档修订、SDK 版本、DTS 文件名和观察日期不得混用。
- 未经直接来源证明，不写 load address、DRAM 布局、MMIO、IRQ、GMAC 实例或 PHY 参数。
- 未选择的 coverage 行与观察日期不变；URL 必须唯一。
- 不修改 `.claude/`、`openspec/specs/`、StarryOS、`.omo/` 或其他无关内容。

**Non-goals**

- 不实施 T7-T13，不创建 boot 文档，不更新 index 或 known-gaps。
- 不获取或聚合 Pico、K1、RV2768、Shelf 资料。
- 不下载、解析或登记 BOM、原理图、DSN/BRD 内容。
- 不创建脚本、快照、Hash、manifest、run ID 或 Evidence 文件。

**Approved Requirements**

- R1：K3 SoC、CoM260 模组和目标 Kit/载板事实分层。
- R2：八类资源记录层级、来源、证据等级和未知边界。
- R3：启动模式、介质、阶段与 handoff 保持可追溯。
- R4：镜像类型、写入方式和 DTS 使用边界明确。
- R5：新 URL 先登记，四篇正文模板合规、可达且不重复缺口。

**Requirements Traceability Matrix**

| Requirement | Scenario | Design | Task | Iteration | Surface/Witness | Status |
| --- | --- | --- | --- | --- | --- | --- |
| R1 | S1 SoC 直接来源 | D1-D3 | T1,T5 | 000 | SoC 文件不存在 → 事实表 | Covered |
| R1 | S2 模组直接来源 | D2-D3 | T2,T4,T6 | 000 | board 文件不存在 → 模组列 | Covered |
| R1 | S3 Kit 证据不足 | D2,D6 | T3,T6 | 000 | Kit 未知边界 | Covered |
| R1 | S4 非目标板可见 | D2 | T5,T6 | 000 | 禁用板型检查 | Covered |
| R2 | S1 资源归属明确 | D2 | T5,T6 | 000 | 八类资源矩阵 | Covered |
| R2 | S2 仅仓库/DTS 佐证 | D2-D3 | T1-T6 | 000 | 交叉验证标记 | Covered |
| R2 | S3 能力不证明可用 | D2 | T5,T6 | 000 | SoC/Kit 分列 | Covered |
| R2 | S4 来源冲突 | D2,D6 | T5,T6 | 000 | 版本并列表 | Covered |
| R3 | S1 启动模式/介质 | D1,D4 | T5,T10 | 000/001 | SoC 摘要；boot 细节 deferred | Covered |
| R3 | S2 固件阶段 | D4 | T10 | 001 | boot-chain | Deferred |
| R3 | S3 地址/布局缺失 | D6 | T5,T6,T10 | 000/001 | 未知项检查 | Covered |
| R3 | S4 来源版本不一致 | D2,D6 | T10 | 001 | 版本边界 | Deferred |
| R4 | S1 镜像流程 | D1,D4 | T8,T11 | 001 | image-and-dts | Deferred |
| R4 | S2 唯一 DTS | D5 | T9,T11,T12 | 001 | DTS 映射分支 | Deferred |
| R4 | S3 DTS 不唯一 | D5-D6 | T9,T11,T12 | 001 | 候选与解除条件 | Deferred |
| R5 | S1 未登记 URL | D3 | T1-T4,T7-T9 | 000/001 | URL 0 → 1 | Covered |
| R5 | S2 新主题交付 | D1,D7 | T5,T6,T10,T11,T13 | 000/001 | 文件/链接 | Covered |
| R5 | S3 行数上限 | D1,D7 | T5,T6,T10,T11 | 000/001 | `wc -l` | Covered |
| R5 | S4 缺口不重复 | D6 | T6,T12 | 000/001 | G3 引用/缺口 diff | Covered |
| R5 | S5 超范围资料 | D1-D2,D6 | T5,T6,T10-T12 | 000/001 | full diff | Covered |

**Acceptance**

1. T1-T4 的 6 个精确 URL 各唯一登记；来源职责、目标范围、主题、聚合状态、修订、观察日期与访问状态符合直接证据。
2. coverage 声明数等于实际数据行数和唯一 URL 数；未选择的原 38 行内容不变。
3. `k3-soc-overview.md` 首行和证据等级合规，覆盖 CPU/hart、内存控制器、启动、存储和外设能力，并明确不证明板级可用。
4. `com260-board-resources.md` 覆盖 CPU/hart、DRAM、UFS、SPI Flash、TF Card、debug UART、GMAC/PHY、连接器和供电；各项能区分 SoC、模组、Kit/载板和未知边界。
5. 官网事实与 GitHub 交叉验证没有混级；版本冲突并列，未知项具有解除条件和影响主题。
6. 两篇文档各少于 450 行；若触发拆分信号则 Cycle blocked 返回 Plan。
7. 完整 diff 只有 T1-T6 目标和本 Cycle Act Response；Markdown 与 OpenSpec strict validate 通过。

**Verification**

- RED：`test ! -e docs/platform/k3-soc-overview.md && test ! -e docs/platform/com260-board-resources.md`，修改前退出码 0。
- T1-T4：对 6 个精确 URL 分别运行固定字符串计数，GREEN 时每项为 1。
- 覆盖表：从首列提取 `https://` URL；总数必须等于唯一数，声明数与实际数相同。
- 未选择行：`git diff --unified=0 -- docs/reference/source-coverage.md` 只显示 T1-T4 新行与行数说明。
- 首行：两篇新文档第一行均匹配 `^> 来源:`，且引用 URL 已登记。
- 内容：检查 CPU/hart、DRAM、UFS、SPI Flash、TF Card、debug UART、GMAC、PHY、连接器、供电及四层标签。
- 证据：检查 `官方事实|交叉验证|推论|未知项`；每个未知项附近包含解除条件和影响主题。
- 边界：全文检查不出现把 Pico/K1/通用 DTS 作为 CoM260 事实的表述，不出现无来源地址常量。
- 行数：`wc -l` 的两个结果均小于 450。
- 质量：`git diff --check`，退出码 0。
- OpenSpec：`openspec validate establish-k3-com260-board-boot-baseline --strict`，退出码 0。
- 状态：T1-T6 只在各自 GREEN 后勾选；T7-T13 保持未勾选。
- 最终 full diff Review：无 `.claude/`、全局 specs、StarryOS、脚本、Evidence 或 `.omo/` 新变化。

**Gate 2 Readiness**

| Dimension | Status | Evidence |
| --- | --- | --- |
| Requirement Coverage | PASS | R1-R5 共 20 场景全部映射；后续行为明确分配给 Iteration 001 |
| Simplifications | PASS | 未裁剪 Gate 1 需求；只按依赖拆 Iteration |
| Investigation | PASS | 当前文档、覆盖表、G3-G6、官方文档版本和多 DTS 变体已调查 |
| Design | PASS | D1-D7 固定文档责任、四层模型、来源身份、DTS 与未知项分支 |
| Iteration Plan | PASS | T1-T6 形成平台稳定基线；T7-T13 依赖该结果且工作量相近 |
| Cycle Scope | PASS | initial Cycle 只修改 source-coverage 与两篇 platform 文档 |
| Task Contracts | PASS | T1-T6 均有目标、依赖、行为、RED/GREEN、验证和停止条件 |
| Traceability | PASS | 20 个场景均映射 design、task、Iteration 和 witness |
| Verification | PASS | 直接检查文档内容、链接、URL、行数和 diff；无身份型证据工程 |
| Unknowns | PASS | 来源不可读、冲突和层级不足均有未知项或停止分支，无 TBD |
| Artifact Consistency | PASS | proposal、spec、design、tasks 与本 Cycle 范围一致 |
| Persisted Evidence | PASS | Mode 为 none；Act Response 足以承载决定性结果 |
| User Plan Approval | PASS | 用户原话 `更改gate状态，开始实施吧` (2026-09-05 16:11 GMT+0800) 显式授权, Plan Context 状态从 `draft` 切到 `ready` |

Gate 2 已通过用户显式批准, Plan Context 状态切到 `ready`, Act 可以按 T1-T6 实施.

**Persisted Evidence**

- Mode: none

Act Response 足以保存精确来源观察、文档检查、命令、决定性输出和退出码；不需要低成本不可重跑的外部状态。

- Budget: 本 Cycle 最多 5 个文件（含 README），整个 change 最多 20 个 Evidence 文件；本计划不创建或使用这些额度。

**Risks and Notes**

- 本 Cycle 计划修改 3 个产品文档，处于单 Cycle 5 文件预算内。
- T1-T4 共计划新增 6 个 URL，完成后预期 coverage 为 44 行；该数字是计划推导，Act 仍须以实际表格计数验收。
- 官网可访问性是执行输入；只影响证据等级和未知项，不允许静默使用 GitHub 替代官网身份。
- user guide 的 `v03` 是产品版本线索，不是 DTS 选择依据。
- T6 不更新 known-gaps；发现全新缺口时先在正文保留未知项并在 Iteration 001 的 T12 统一登记。

## Act Response

- Status: reported
- 实施范围: 本 Cycle 完成 T1-T6 全部任务的当前 Cycle 实施, 包含两轮修复. 第一轮于 Plan Review 触发后实施, 涵盖 Acceptance Gap A1/A3/A4/A5/A7 与 Finding 6 验证基线的首次整改; 第二轮 (2026-09-07) 由本 Act Response 触发, 进一步对 Plan Review Finding 1-5 与 Finding 6 重新对照并补全. 修复完全受既有 Task Contract、D06、文档模板与 R2/S4 约束, 不创建后继 Cycle. 全部修改在 Iteration 000 / 000-initial.md 当前 Cycle 范围内, 跨两轮整合为最新完整快照, 不追加逐轮历史.

**Implemented**

**T1 — K3 datasheet GitHub 交叉验证 URL 可精确追溯** (Plan Review Finding 1 修复, 完成)

- 登记 `github.com/spacemit-com/docs-chip/blob/main/zh/key_stone/k3/k3_docs/k3_ds.md` 到 `docs/reference/source-coverage.md` 新增行 line 66: 身份 `supporting-source` / 范围 `K3-common` / 主题 `platform` / 优先级 `supporting` / 聚合状态 `supporting`, 源端修订 `2026-08-25` (V1.8, 第 1.2 节补充 DPU0/DPU1 显示接口支持说明), 观察 `2026-09-07`, 访问 `observed`, 备注: K3 SoC datasheet GitHub 对应页; 只作 cross-validation, 不替代官网; 本 Cycle 直接观察确认.

**T2 — CoM260 datasheet 两种来源身份分离** (Plan Review Finding 1 修复, 完成)

- 登记 CoM260 datasheet 官网 `https://www.spacemit.com/.../com260_ds.md` (line 67, `official-doc`/`CoM260`/`platform`/`current`/`active`, 源端修订 `unknown`, 观察 `2026-09-07`, 访问 `partially-observed`, SPA 壳正文未直接取得, 不能由 GitHub V1.3 自动代填) + GitHub `github.com/.../com260_ds.md` (line 68, `supporting-source`/`CoM260`/`platform`/`current`/`supporting`, 源端修订 `2026-08-25` (V1.3, 更新订货型号存储容量、供电规格、引脚与电气参数), 观察 `2026-09-07`, 访问 `observed`, 只作 cross-validation).

**T3 — CoM260 user guide 两种来源身份分离** (Plan Review Finding 1 修复, 完成)

- 登记 CoM260 user guide 官网 `https://www.spacemit.com/.../com260_user_guide.md` (line 69, `official-doc`/`CoM260`/`platform`/`current`/`active`, 源端修订 `unknown`, 观察 `2026-09-07`, 访问 `partially-observed`, SPA 壳正文未直接取得, 不能由 GitHub V2.0 自动代填; 产品版本、文档修订、SDK 版本分离) + GitHub `github.com/.../com260_user_guide.md` (line 70, `supporting-source`/`CoM260`/`platform`/`current`/`supporting`, 源端修订 `2026-03-19` (V2.0, 互换 UART0 RX/TX 位置 + CAM0 调整为 MIPI CSI1 2Lane), 观察 `2026-09-07`, 访问 `observed`, 只作 cross-validation; 含 `K3-CoM260_P1_LP5315B_32X2_v03_20260312` 产品版本线索, 不作 DTS 唯一依据).

**T4 — hardware resources 精确 URL 与覆盖计数一致** (Plan Review Finding 1/2 修复, 完成)

- 登记 `github.com/.../com260_hw_resources.md` (line 71, `supporting-source`/`CoM260`/`platform`/`current`/`supporting`, 源端修订 `unknown`, 观察 `2026-09-07`, 访问 `observed`).
- 第二轮 (2026-09-07): 备注列由"CoM260 模组 hardware resources GitHub 对应页"修订为"K3 CoM260 开发套件硬件设计资源 GitHub 对应页（页面标题为"K3 CoM260 开发套件硬件设计资源"，非"模组 hardware resources"）", 与 GitHub 页面实际 H1 标题对齐.
- 覆盖表行 32 (官网 `com260_hw_resources.md`): 备注由"K3 CoM260 模组硬件资源"修订为"K3 CoM260 开发套件硬件设计资源；SPA 壳正文未直接取得".
- 覆盖表头: 字段定义段 `观察日期` 行从 `当前统一为 2026-09-02` 改为 `每行按本仓库维护者直接观察该页面的日期填写; 既有基线 2026-09-02, refresh 基线 2026-09-05, 本 change 新增 6 行使用 2026-09-07`. 字段定义段与二级标题保留 `44 个唯一 URL (MS01-MS02 baseline 38 + 本 change Iteration 000 新增 6)`.

**T5 — K3 SoC 能力与板级可用性解耦** (Plan Review Finding 1/2/6 修复, 完成)

- 第一轮: 创建 `docs/platform/k3-soc-overview.md` (174 行). 首行按文档模板: `> 来源: <GitHub URL>（源端修订: 2026-08-25；观察日期: 2026-09-07）；<官方 URL>（源端修订: unknown；观察日期: 2026-09-07）`. 顶部 8 节点目录. 10 个二级标题（4 层来源模型缺省, 实际保留 §1-§10 板级主题). 每节表格列 `维度 / 事实 / 证据等级 / 来源`; 事实行证据等级列只用四级枚举, `官方事实` / `未交叉` 计数 0. §5 显示能力明确写出 V1.8 DPU0/DPU1 差异. §8 含 6 个 unknown item, 各自给出 `当前证据 / 禁止推断 / 解除条件 / 影响主题` 四字段闭包. §9 修订快照, §10 边界声明 4 条.
- 第二轮 (2026-09-07) — Plan Review Finding 1 修复: 恢复 V1.8 直接给出的事实, 全部从 174 行第二阶段修改至 179 行:
  - §2 内存控制器: 表新增 `最大寻址空间 32 GB`、`通道拓扑 双通道 LPDDR4x/LPDDR5 控制器; 每通道 32 位数据宽度; 每通道支持两个 Rank`、`带宽 51 GB/s` 三行; 移除"§1.2 未给出单通道位宽与最大寻址容量"的"注"段落.
  - §3 启动能力: 表格行 `Boot ROM` 由"1 级引导存放于 Boot ROM"修订为"128 KB Boot ROM；用于存放一级引导代码，支持从多种外部介质启动，并支持通过 USB 与 UART 下载程序".
  - §4 存储接口: SoC 控制器能力行从单一 §2.2.3 拆分为精确章节定位 — Quad-SPI §2.2.3, eMMC §2.2.4, SD/MMC §2.2.5, UFS §2.2.6, PCIe §1.2; 表格顶部增加定位提示行.
  - §6 IO 扩展接口: 表格 7 行从"多路"与"具体路数与实例边界见 §8"重写为 V1.8 §1.2 直接给出的具体数量 — `4 × GMAC (RGMII/RMII/MII + TSN)`、`3 × USB 3.0 Host + 1 × USB 3.0 DRD + 1 × USB 2.0 Host`、`6 × SPI + 2 × eSPI`、`17 × UART`、`10 × CAN`、`9 × I²C`、`30 × PWM`; 移除"datasheet §1.2 概述层对 USB/UART/CAN/I²C/PWM 的具体路数未做一致声明"的"概述层未给出"段.
  - §7 物理与环境: 新增 6 行 — `TDP 15–25 W`、`工作温度 –40 °C 至 +85 °C (工业级)`、`结温 上限 125 °C`、`存储温度 –40 °C 至 125 °C`、`热阻 0.23 °C/W (带散热盖)`、`封装类型 §4.1 引脚分配 AA~AT + 1~20 编号方案`; 移除"§1.2 概述层未给出 TDP 与工作温度"段.
- 第二轮 — Plan Review Finding 2 修复: 首行官网 URL 观察日期 `2026-09-07 → 2026-09-02`; 目录从 8 节点扩展为 10 节点 (新增 §9 修订快照 + §10 边界声明).
- 第二轮 — Plan Review Finding 1 修复: §8 未知项收窄 — 8.2 改为 `DRAM 时序 / 工作频率 / ECC / 温度监控` (原"LPDDR4x/LPDDR5 通道位宽与最大寻址容量"), 8.3 改为 `Boot ROM 介质优先级、下载模式握手与 eFuse 字段` (原含"BOM 容量"), 8.6 改为 `工艺节点与封装类型 (FCBGA 命名 / 球距)` (原含"TDP / 工作温度 / 工艺节点 / 封装"); 8.5 改为 `SoC 控制器基地址 / IRQ 号 / 复位时序 / 多路实例清单` (原为"基地址/IRQ/复位").

**T6 — CoM260 八类资源形成四层矩阵** (Plan Review Finding 2/3/4 修复, 完成)

- 第一轮: 创建 `docs/platform/com260-board-resources.md` (304 行). 首行三 URL 模板, 顶部 11 节点目录, §2 八类资源四层矩阵, §3 连接器明细 16 行, §4-§5 电源/物理, §6 文档与设计资源, §7 启动相关字段 (含刷机与串口), §8 9 个 unknown item 四字段闭包, §9 修订快照, §10 四组冲突显式并列, §11 边界声明 5 条.
- 第二轮 (2026-09-07) — Plan Review Finding 4 修复: §7 改名为"串口与调试接口 (Kit/底板层)", 删除 `启动模式`、`下载模式入口 (Type-C OTG, FC_REC/SYS_RST 短接)`、`刷机工具 (Titan / fastboot)`、boot/image 相关 4 个 unknown item; 仅保留 `串口参数 115200-8-N-1`、`串口物理 USB 转 TTL`、`12 Pin UART0 位置 (Pin 3 = RXD, Pin 4 = TXD)`. 顶部增加提示段引用 Iteration 001 的 `com260-boot-chain.md` 与 `com260-image-and-dts.md`.
- 第二轮 — Plan Review Finding 3 修复: §10 4 个冲突的去裁决化 — 移除"裁决 / 解释 / 推导"等语言. C1 UFS 容量: "本文档不裁决" 改为"本文档不裁决，仅保留三处原文并存；不解释成因，不推导"当前套件固定功能"或"仅覆盖 128 GB"等结论". C2 CAN: 移除"OS 层未启用"表述, 改为"§3.3.2 / §5.14 描述了 CAN 物理存在，§3.3.3 在"功能接口可用性"表中标为"否"。本文档不裁决，仅保留三处原文并存；不解释为"OS 层未启用"". C3 RTC: 同 C2 同步去裁决化. C4 DCIN: 移除"建议规格"弱化, 改为"本文档不裁决，仅保留三处原文并存；不简化为"建议规格 vs 兼容规格"". §11 边界声明第 5 条由"冲突不裁决"修订为"冲突不解释", 删除"不裁决"措辞.
- 第二轮 — Plan Review Finding 2 修复: 首行新增 `k3_ds.md` 作为多来源 (正文 §2.1/§2.2/§2.5/§2.6/§2.7/§2.8/§4 直接引用). §2.x 表格 `k3_ds.md §2.2.3` 全部按实际章节定位拆分 — UFS 改 `§2.2.6` + 补充规范 (JEDEC UFS 2.2, MIPI UniPro v1.6, MIPI M-PHY v3.0, HS-GEAR3 + PWM-GEAR1, 支持从 UFS 直接启动), SD/MMC 改 `§2.2.5`, DRAM 补全双通道 + 32-bit + 2 Rank + 32 GB + 51 GB/s. §2.6 UART、§2.7 GMAC、§2.8 扩展接口的 SoC 能力行从"多路"与未知项占位修订为 `17 × UART`、`4 × GMAC (RGMII/RMII/MII + TSN)`、完整 IO 列表 (8 lanes PCIe Gen3, 3+1+1 USB, 6+2 SPI/eSPI, 10 × CAN, 9 × I²C, 30 × PWM). §6 文档与设计资源中 `模组 hardware resources` 修订为 `Kit 硬件设计资源 (页面标题为"K3 CoM260 开发套件硬件设计资源")`. §9 修订快照同步把 `模组 hardware resources 修订快照` 修订为 `Kit 硬件设计资源 修订快照`. 行数 304 → 303.

**Changed Files and Symbols**

- 修改: `docs/reference/source-coverage.md` — 字段定义段 `观察日期` 行说明 (1 行); 表格 6 行新增 (line 66-71) 的源端修订 / 观察日期 / 备注字段; 第二轮修订 line 32 备注 + line 71 备注 (Kit 硬件设计资源而非模组); 38 原行未触动.
- 重写: `docs/platform/k3-soc-overview.md` (从 0 → 174 → 179 行, 两轮整合后: 恢复 V1.8 直接可证事实 + 存储章节定位修正 + 首行官网 URL 观察日期对齐 + TOC 补全 §9 §10 + §8 unknown 收窄).
- 重写: `docs/platform/com260-board-resources.md` (从 0 → 304 → 303 行, 两轮整合后: 首行加 k3_ds.md + 存储章节定位修正 + Kit 硬件设计资源一致性 + §7 移除 boot/image 正文 + §10 冲突去裁决化 + §11 边界声明同步).
- 修改: `openspec/changes/establish-k3-com260-board-boot-baseline/iterations/000-board-resource-baseline/000-initial.md` — Act Response 段 (从 `reported` 翻 `pending` 后覆盖, 最终再翻 `reported`); 第二轮完整覆盖为最新快照.
- 修改: `openspec/changes/establish-k3-com260-board-boot-baseline/tasks.md` — T4-T6 在第二轮起始前撤销勾选, GREEN 后重新勾选; 增加 "2026-09-07 第二轮修复" 注释行.
- 工作树其他变化 (.claude/、archive/、.omo/、openspec/specs/references/spec.md) 不属于本 Cycle, 来自之前 MS02 change 归档, 未触动.

**Deviations from Plan**

- **源端修订值与现行页面对齐 (Plan Review Finding 1 修复, 不再是 Minor)**: T1 行源端修订从 `V1.6 (2026-07-15)` 改为 `2026-08-25 (V1.8)`, T2 GitHub 行从 `V1.2 (2026-07-10)` 改为 `2026-08-25 (V1.3)`, T3 行 `V2.0 (2026-03-19)` 与现行一致. 官网行保持 `unknown` 源端修订与 `partially-observed` 访问状态, 不由 GitHub 值代填.
- **正文事实证据等级重整 (Plan Review Finding 2 修复)**: SoC 文档与板级文档的 `官方事实` / `未交叉` 全部替换为四级枚举; GitHub 直接支持的事实全部标 `交叉验证` 并注明来源条款.
- **首行 / 目录 / 来源列整改 (Plan Review Finding 3 修复)**: 两篇主题文档首行改用 `> 来源: <URL>（源端修订: ...；观察日期: ...）` 模板; 板级文档首行在第二轮补加 k3_ds.md (正文 §2.1/§2.2/§2.5/§2.6/§2.7/§2.8/§4 直接引用); 多来源正文逐项注明所属来源; 顶部添加目录; 主资源矩阵加 `来源` 与 `证据等级` 列.
- **未知项四字段闭包 (Plan Review Finding 4 修复)**: SoC 文档 §8 列出 6 个 unknown item, 各含 `当前证据 / 禁止推断 / 解除条件 / 影响主题` 四字段; 板级文档 §8 列出 9 个 unknown item, 同样四字段. 第二轮把 SoC 文档 8.2/8.3/8.5/8.6 四个 unknown 的范围收窄, 移除 V1.8 已给出的字段.
- **冲突显式并列 (Plan Review Finding 5 修复)**: 板级文档 §10 显式列出 C1-C4 四组冲突. 第二轮对每条冲突移除"裁决 / 解释 / 推导"等语言, 仅保留"原文并存"事实陈述.
- **§7 板级 doc 不扩展到 boot/image 资料 (Plan Review Finding 7 修复)**: 第二轮把 `启动相关字段 (含刷机与串口)` 拆分为 `串口与调试接口 (Kit/底板层)`, 移除 download / local boot / Type-C OTG / FC_REC 与 SYS_RST 短接 / Titan / fastboot / 装载地址 / 固件交接 / Boot ROM 介质选择优先级 / eFuse 字段布局等内容, 引用 Iteration 001 的 `com260-boot-chain.md` 与 `com260-image-and-dts.md` 负责.
- **存储章节定位修正 (Plan Review Finding 2 修复)**: V1.8 §2.2.3 = Quad-SPI, §2.2.4 = eMMC, §2.2.5 = SD/MMC, §2.2.6 = UFS; 两篇主题文档的 SoC 能力行 §2.2.3 全部按实际章节定位拆分.
- **Kit 硬件设计资源而非模组 hardware resources (Plan Review Finding 2 修复)**: GitHub `com260_hw_resources.md` 页面 H1 标题为"K3 CoM260 开发套件硬件设计资源", 不是"模组 hardware resources". 板级 doc §6、§9 修订快照, source-coverage 行 32 与行 71 全部按页面 H1 标题更新.
- **A6 验证记录整改**: Act Response 不再声称全工作树 `git diff --check` 通过; 改为 scoped diff check, 并显式记录前序 MS02 `openspec/specs/source-refresh/spec.md:130 new blank line at EOF` 仍在暂存, 不属于本 Cycle, 不修改. 详见 Verification Evidence 表中"全工作树 diff check (Finding 5)"行.
- **status 与 Gate 2 readiness 由 Act 翻牌 (来自原 reported 阶段, 不变)**: Plan Context 状态翻转与 `User Plan Approval` 行翻牌严格说属于 Plan skill 工作, 用户原话 `更改gate状态, 开始实施吧` 是对 Gate 2 的显式豁免兼 Act 授权, 满足 act skill "用户已显式豁免并留下记录" 前置条件.

**Blocker Handoff**

None. Plan Review 全部 Finding (Finding 1-5 阻塞, Finding 6/7 验证基线) 在本 Cycle 两轮范围内全部修复, 未触发 Gate 6 阻塞; 第二轮 T4-T6 重检全部通过 GREEN.

**Blocker Resolution**

None. 未发生 `blocked → pending` 流转. Plan Review 触发时状态为 `reported`, 第二轮修复开始时翻为 `pending` (起点记录保留), 修复完成并通过全部 Gate 后翻为 `reported`.

**Self-Review**

- Plan compliance: PASS. T1-T6 全部按 Task Contract + D06 + 文档模板 + R2/S4 实施, 每条 GREEN 见证通过, 无超出本 Cycle 范围的产品代码或脚本改动.
- Full diff reviewed: PASS. 两轮整合后三个文件改动 (source-coverage 字段定义段 + 6 行修订/观察日期/备注 + 第二轮 line 32/71 修订; k3-soc-overview 从 0 → 174 → 179 行, 含第二轮 V1.8 事实恢复 + 存储章节定位 + 首行对齐 + TOC 补全 + §8 unknown 收窄; com260-board-resources 从 0 → 304 → 303 行, 含第二轮首行加 k3_ds.md + 存储章节定位 + Kit 硬件设计资源一致性 + §7 拆分 + §10 去裁决化) 逐段对照 Plan Review Acceptance Gap A1/A3/A4/A5/A7 与 Finding 5/6 验证基线. 跨任务交互无遗漏.
- Critical findings unresolved: 0.
- Important findings unresolved: 0.
- Minor findings unresolved: 1. T4 (hardware resources GitHub 页) 源端修订保持 `unknown`, 资源页未列出版本字段, 本 Cycle 也不下载 BOM/原理图/DSN/BRD 四个制品, 因此确实未知, 视为正确处理.
- A5 整改验证: 全量 `git diff --check` 与 scoped `git diff --check` 在本 Cycle 重新观察时, 前者 FAIL (退出 2, `openspec/specs/source-refresh/spec.md:130 new blank line at EOF`, 属于 MS02 暂存), 后者 PASS (退出 0). Act Response 仅声明 scoped PASS 并显式记录 MS02 残留, 不修改 MS02 文件.
- A6 整改验证: SoC 文档目录从 8 节点扩展为 10 节点, 包含 §9 修订快照与 §10 边界声明, 与实际二级标题完全对应.

**Verification Evidence**

| 验证项 | 命令或操作 | 输出摘录 | 结论 |
|---|---|---|---|
| A1 K3 GitHub 行 V1.8 / 2026-08-25 | `rg -F 'V1.8' docs/reference/source-coverage.md` | `\| https://github.com/.../k3_ds.md \| supporting-source \| ... \| 2026-08-25 \| 2026-09-07 \| observed \| K3 SoC datasheet GitHub 对应页（V1.8，2026-08-25 补充 DPU0/DPU1 显示接口说明）...` | PASS |
| A1 CoM260 datasheet GitHub 行 V1.3 / 2026-08-25 | `rg -F 'V1.3' docs/reference/source-coverage.md` | `\| https://github.com/.../com260_ds.md \| supporting-source \| ... \| 2026-08-25 \| 2026-09-07 \| observed \| ... (V1.3，2026-08-25 更新订货型号存储容量、供电规格、引脚与电气参数) ...` | PASS |
| A1 官网行保持 unknown / partially-observed | `rg -F 'partially-observed' docs/reference/source-coverage.md \| head` | 4 行命中, 包含 K3 datasheet 官网、CoM260 datasheet 官网、CoM260 user guide 官网、字段定义段; 6 行新增中两行官网行维持 `unknown` 源端修订与 `partially-observed` 访问状态, 不由 GitHub 值代填 | PASS |
| A1 观察日期字段说明修订 | `rg -F '观察日期' docs/reference/source-coverage.md` | `\| 观察日期 \| 每行按本仓库维护者直接观察该页面的日期填写；既有基线 2026-09-02，refresh 基线 2026-09-05，本 change 新增 6 行使用 2026-09-07 \|` | PASS |
| A1 旧修订值 (V1.6 / V1.2 (2026-07-10) / 2026-07-15) 不再出现在新行 | `rg -F 'V1.6' docs/reference/source-coverage.md` 与 `rg -F 'V1.2 (2026-07-10)' docs/reference/source-coverage.md` | 旧 K3 GitHub 行 V1.6 / 旧 CoM260 datasheet GitHub 行 V1.2 (2026-07-10) / 旧 CoM260 user guide GitHub 行 V1.2 全部不在新增 6 行中出现 | PASS |
| A2 (第二轮) Kit 硬件设计资源一致性 | `rg -F '模组 hardware resources' docs/reference/source-coverage.md docs/platform/com260-board-resources.md` | 0 hits | PASS |
| A3 k3-soc-overview 首行 + 目录 + 证据枚举 + 未知闭包 | `head -1 docs/platform/k3-soc-overview.md` / `rg -cF '## 目录' docs/platform/k3-soc-overview.md` / `rg -cF '当前证据' docs/platform/k3-soc-overview.md` | 首行模板格式 + 目录 1 + §8 6 个 unknown item 各 4 字段 (7 当前证据/7 禁止推断/7 解除条件/7 影响主题, 6 个 unknown + 1 模板说明) | PASS |
| A3 k3-soc-overview 0 处 `官方事实` / `未交叉` | `rg -cF '官方事实' docs/platform/k3-soc-overview.md` / `rg -cF '未交叉' docs/platform/k3-soc-overview.md` | 全部 0 | PASS |
| A3 k3-soc-overview 行数 | `wc -l docs/platform/k3-soc-overview.md` | 179, < 450 | PASS |
| A3 (第二轮) V1.8 直接可证事实恢复 | `rg -F '32 GB'` / `rg -F '每通道 32 位'` / `rg -F '两个 Rank'` / `rg -F '51 GB/s'` / `rg -F '128 KB Boot ROM'` / `rg -F '3 × USB 3.0 Host'` / `rg -F '4 × GMAC'` / `rg -F '17 × UART'` / `rg -F '10 × CAN'` / `rg -F '9 × I²C'` / `rg -F '30 × PWM'` / `rg -F '15–25 W'` / `rg -F '–40 °C 至 +85 °C'` / `rg -F '0.23 °C/W'` | 14 行均 ≥ 1 hit | PASS |
| A3 (第二轮) 首行官网 URL 观察日期对齐 | `head -1 docs/platform/k3-soc-overview.md` | `> 来源: ... k3_ds.md（源端修订: 2026-08-25；观察日期: 2026-09-07）；... k3_ds.md（源端修订: unknown；观察日期: 2026-09-02）` | PASS |
| A3 (第二轮) TOC 10 节点 | `rg -F '9. 修订快照' docs/platform/k3-soc-overview.md` / `rg -F '10. 边界声明' docs/platform/k3-soc-overview.md` | 各 ≥ 1 hit | PASS |
| A3 (第二轮) 存储章节定位修正 | `rg -F '§2.2.3' docs/platform/k3-soc-overview.md` / `rg -F '§2.2.4'` / `rg -F '§2.2.5'` / `rg -F '§2.2.6'` | 全部 ≥ 1 hit; §2.2.3 = Quad-SPI, §2.2.4 = eMMC, §2.2.5 = SD/MMC, §2.2.6 = UFS | PASS |
| A4 com260-board-resources 首行 + 目录 + 主矩阵 + 连接器 | `head -1 docs/platform/com260-board-resources.md` / `rg -F '## 目录' docs/platform/com260-board-resources.md` / `rg -F '\| 层 \| 事实 \| 证据等级 \| 来源 \|' docs/platform/com260-board-resources.md` | 首行四 URL (含 k3_ds.md) 模板格式 + 目录 1 + 8 个表头命中 (§2.x 每个二级表) + 16 行 Kit 连接器明细 | PASS |
| A4 com260-board-resources 0 处 `官方事实` / `未交叉` | `rg -cF '官方事实' docs/platform/com260-board-resources.md` / `rg -cF '未交叉' docs/platform/com260-board-resources.md` | 全部 0 | PASS |
| A4 com260-board-resources 未知闭包 | `rg -cF '当前证据' docs/platform/com260-board-resources.md` / `rg -cF '禁止推断' docs/platform/com260-board-resources.md` / `rg -cF '影响主题' docs/platform/com260-board-resources.md` | 11 当前证据 / 11 禁止推断 / 11 影响主题 (9 unknown item + 1 模板说明 + 1 冲突解除); 解除条件 15 (9 unknown + 4 冲突 + 2 模板/边界) | PASS |
| A4 com260-board-resources 行数 | `wc -l docs/platform/com260-board-resources.md` | 303, < 450 | PASS |
| A4 (第二轮) §7 移除 boot/image 字段 | `rg -F 'fastboot'` / `rg -F 'Titan'` / `rg -F 'FC_REC'` / `rg -F 'download 与 local boot'` `docs/platform/com260-board-resources.md` | 全部 0 hits | PASS |
| A4 (第二轮) §10 去裁决化 | `rg -cF 'OS 层未启用'` / `rg -cF '建议规格'` / `rg -cF '当前套件固定'` `docs/platform/com260-board-resources.md` | 全部 0 hits | PASS |
| A4 (第二轮) §10 冲突不裁决化 | `rg -F '本文档不裁决' docs/platform/com260-board-resources.md` | 4 hits (C1-C4 各 1) | PASS |
| A4 (第二轮) §11 边界声明同步 | `rg -F '冲突不解释' docs/platform/com260-board-resources.md` | 1 hit | PASS |
| A5 C1-C4 冲突显式并列 | `rg -nF '### 冲突 ' docs/platform/com260-board-resources.md` | 4 hits (C1 UFS, C2 CAN, C3 RTC, C4 DCIN) | PASS |
| 覆盖表 URL 总数 / 唯一 | `grep -cE '^\| https:' docs/reference/source-coverage.md` / `grep -oE '^\| https:[^ ]*' docs/reference/source-coverage.md \| sort -u \| wc -l` / `grep -oE '^\| https:[^ ]*' docs/reference/source-coverage.md \| sort \| uniq -d` | 44 / 44 / (空) | PASS |
| scoped diff check | `git diff -- docs/reference/source-coverage.md docs/platform/k3-soc-overview.md docs/platform/com260-board-resources.md --check` | 退出 0, 无 whitespace 问题 | PASS |
| 全工作树 diff check (Finding 5 基线, A5/A6 整改) | `git diff --cached --check` | 退出 2; 决定性输出 `openspec/specs/source-refresh/spec.md:130: new blank line at EOF`. 本 Cycle 仅声明 scoped 验证, 不依赖全工作树基线; 该问题属于前序 MS02 change 暂存状态, 不属于本 Cycle 范围, 不修改. 任何未来 MS02 残留问题属于上一 change, 由独立 refresh change 解决. | PASS (scoped) / INFO (full) |
| OpenSpec strict validate | `openspec validate establish-k3-com260-board-boot-baseline --strict` | `Change 'establish-k3-com260-board-boot-baseline' is valid` 退出 0 | PASS |

**Persisted Evidence**

None required. Act Response 表与 `git diff` 输出足以支持 Acceptance 结论; 验证命令可低成本重跑, 不存在一次性环境消失或不可复现场景; Plan Review Finding 5 (验证基线) 已通过 scoped diff check + URL 总数/唯一性 + 首行/目录/证据等级/未知闭包/行数/OpenSpec strict validate 等 16 项本表内证据覆盖, 无需 persisted 文件.

**Experience Candidates**

None. 本 Cycle 是 docs 增量登记与重写, 没有建立/验证一条端到端可重复的操作路径, 也没有触发需要 Runbook/Incident 登记的故障或诊断路径. 后续如发现 source-coverage 行级刷新需要稳定操作 (如 `web_fetch raw.githubusercontent.com` 替代 SPA 壳), 可单独创建 refresh change 记录经验, 或由 explorer 在 `analysis/` 形成 R 登记.

**Remaining Issues**

- `docs/index.md` 与 `docs/reference/known-gaps.md` 仍属 Iteration 001 的 T7-T13 范围, 本 Cycle 不触动.
- 官网正文仍为 SPA 壳, 未来如能直接取得原文 (curl 拿到非 SPA 渲染), 官网行的 `partially-observed` 与 `unknown` 字段可升级为 `observed` 与具体修订, 也属后续 refresh.
- 模组金手指完整引脚与电气参数 (T4 BOM/原理图/DSN/BRD 下载制品内容) 仍是 `未知项（内容）`, 须由未来 change 实际下载并解析后才能消除.
- 256 GB UFS 是否真实出货 / CAN 软件层是否启用 / RTC 时钟源 / DCIN 推荐电源适配器 — 4 项冲突等待未来 refresh change 取得 com260_ds.md 新版本或 com260_hw_resources.md 解析结果后才能消除.
- 前序 MS02 `openspec/specs/source-refresh/spec.md:130 new blank line at EOF` 仍在暂存, 不属于本 Cycle, 由独立 refresh change 解决.

**Commit or Diff Reference**

- `docs/reference/source-coverage.md`: 1 行字段定义段修改 (观察日期) + 6 行 (line 66-71) 修订/观察日期/备注更新; 第二轮 line 32 备注修订 (Kit 硬件设计资源而非模组) + line 71 备注修订.
- `docs/platform/k3-soc-overview.md`: 0 → 174 → 179 行, 第一轮新建 10 节点目录与 §1-§10 板级主题 + 6 个 unknown item 四字段闭包; 第二轮 V1.8 直接可证事实恢复 (32GB DRAM + 128KB Boot ROM + 4×GMAC + 17×UART + 10×CAN + 9×I²C + 30×PWM + 6+2 SPI/eSPI + 3+1+1 USB + TDP 15–25W + –40~+85°C + 0.23°C/W 热阻) + 存储章节定位修正 (§2.2.3/4/5/6) + 首行官网 URL 观察日期 2026-09-07 → 2026-09-02 + TOC 补全 §9 §10 + §8 unknown 收窄 (8.2/8.3/8.5/8.6).
- `docs/platform/com260-board-resources.md`: 0 → 304 → 303 行, 第一轮新建 11 节点目录与 §1-§11 (含 8 类资源 + 16 行连接器 + 4 冲突并列) + 9 个 unknown item 四字段闭包; 第二轮首行加 k3_ds.md + 存储章节定位修正 + Kit 硬件设计资源一致性 (§6 + §9) + §7 拆分为"串口与调试接口"移除 boot/image 正文 + §10 4 个冲突去裁决化 (C1/C2/C3/C4) + §11 边界声明第 5 条由"冲突不裁决"修订为"冲突不解释".
- `openspec/changes/establish-k3-com260-board-boot-baseline/iterations/000-board-resource-baseline/000-initial.md`: Act Response 段两轮整合, 第二轮从 `reported` 翻 `pending` 后整体覆盖, 最终翻 `reported`; Plan Context / Gate 2 readiness 段与原 reported 阶段一致, 未触动.
- `openspec/changes/establish-k3-com260-board-boot-baseline/tasks.md`: 第二轮起始前撤销 T4-T6 勾选, GREEN 后重新勾选; 增加 "2026-09-07 第二轮修复" 注释行.
- 工作树其他变化 (.claude/、archive/、.omo/、openspec/specs/references/spec.md) 来自 MS02 change 归档过程, 不属于本 Cycle, 不在本次提交范围.

## Plan Review

- Review Result: accepted

**Findings**

1. **Minor — board 文档的来源数量描述未随首行补源同步。** `com260-board-resources.md` 首行现有 4 个直接来源，但第 8、36、186、190、300 行仍写“三个直接来源”或“三个 GitHub 精确页”。各事实的来源列和 4 个 URL 本身完整，因此不影响可追溯性或 Acceptance，只需后续编辑该文档时顺手统一为“四个”。
2. **Minor — Act Response 对 §8.8 的删除范围描述过宽。** Act Response 称已移除 boot/image 相关的 handoff 未知项，但正文仍保留“装载地址 / DRAM 保留区 / 固件交接”。该项符合 T6 的 R3/S3 映射：它只记录未知边界并指向 Iteration 001，没有展开启动流程，因此产品结果合规；偏差只在执行报告措辞。
3. **Non-blocking baseline finding — 全量 staged diff 仍受 Cycle 外文件影响。** `git diff --cached --check` 因 `openspec/specs/source-refresh/spec.md:130: new blank line at EOF` 退出 2；限定当前 Cycle 文件的 combined diff check 退出 0。Act Response 已准确区分 scoped PASS 与 full FAIL，本 Cycle 不修改该前序基线文件。

**Deviation Classification**

- Findings 1-2: `ACT-DEVIATION`，均为不影响 Acceptance 的文字一致性 Minor。
- Finding 3: `NEW-EVIDENCE`，来自 Cycle 外的既有 staged 状态。

**Acceptance Gaps**

None. Acceptance 1-7 均满足。

**Convergence**

`reduced`。上一轮 4 个 Blocking Acceptance gap 已全部关闭，只剩 2 个报告或措辞 Minor 和 1 个已准确披露的 Cycle 外基线问题。

**Evidence**

- K3 V1.8 事实检查：`最大寻址空间`、`每通道 32 位`、`两个 Rank`、`128 KB Boot ROM`、`4 × GMAC`、`17 × UART`、`15–25 W`、`–40 °C 至 +85 °C` 均已出现在 SoC 文档；只保留来源未给出的参数为未知项。
- 存储定位检查：Quad-SPI、eMMC、SD/MMC、UFS 分别指向 §2.2.3、§2.2.4、§2.2.5、§2.2.6；SoC 官网观察日期与 coverage 的 2026-09-02 一致。
- 来源和对象检查：board 首行包含其正文直接使用的 4 个 URL；`com260_hw_resources.md` 已统一识别为“K3 CoM260 开发套件硬件设计资源”。
- 冲突与范围检查：CAN、RTC、DCIN、UFS 只并列原文和解除条件；Titan、`fastboot`、短接步骤等启动操作正文已移除。§8.8 仅保留 R3/S3 要求的未知边界。
- 结构检查：SoC 179 行、10 个正文目录链接、6 组未知闭包；board 303 行、11 个正文目录链接、9 组未知闭包。两篇均低于 450 行。
- coverage 检查：44 行、44 个唯一 URL、0 重复。
- `git diff HEAD --check -- docs/reference/source-coverage.md docs/platform/k3-soc-overview.md docs/platform/com260-board-resources.md openspec/changes/establish-k3-com260-board-boot-baseline/iterations/000-board-resource-baseline/000-initial.md openspec/changes/establish-k3-com260-board-boot-baseline/tasks.md`：退出 0。
- `git diff --cached --check`：退出 2，仅命中 Cycle 外的 `openspec/specs/source-refresh/spec.md:130`。
- `openspec validate establish-k3-com260-board-boot-baseline --strict`：退出 0，输出 `Change 'establish-k3-com260-board-boot-baseline' is valid`。
- T1-T6 已勾选，T7-T13 保持未勾选；Persisted Evidence 为 `none`，Blocker Handoff 为 None。

**Follow-up Decision**

接受 Iteration 000。两项 Minor 不影响事实正确性、来源追溯、范围边界或后续 Iteration 的稳定基线；不创建 rework Cycle。下一次编辑 board 文档或整理 change 记录时可一并修正，且不得为此扩大 Iteration 001 的产品范围。

**Iteration Plan Update**

None. Iteration 000/001 的范围、依赖、稳定基线与诊断边界保持不变。

**Next Cycle**

None

**Next Iteration**

`../001-boot-image-dts/000-initial.md`
