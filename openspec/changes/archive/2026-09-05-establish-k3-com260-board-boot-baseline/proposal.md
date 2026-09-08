## Why

MS01-MS02 已建立来源覆盖、文档模板和刷新机制，但 `docs/` 仍没有 K3 技术主题正文。MS03 需要先把 K3 SoC、CoM260 模组、目标 Kit 和启动链分层整理，避免后续 platform、UART、IRQ、DMA 与 GMAC change 各自重复解释目标板资源，或把芯片能力误写成板上已连接设备。

## Source Baseline and Change Type

- 权威入口: https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs
- 已登记硬件来源: R04 的 K3 datasheet/root overview、CoM260 root overview/hardware resources
- 已登记启动来源: R05 的 `boot.md`、`image.md`、`device_management.md`
- 交叉验证来源: R08 的 `docs-chip`、`docs-product`、`docs-buildroot` 与 `linux-6.18`
- 当前观察基线: R04/R05 为 2026-09-02；R08 为 2026-09-05
- 已观察源端修订: K3 datasheet V1.6（2026-07-15）、CoM260 datasheet V1.2（2026-07-10）；其余页面未确认时保持 `unknown`
- Change 类型: aggregation；整理现有来源，不预设官网发生 refresh 变化
- 对应 milestone: MS03，一个 milestone 对应本 change

## What Changes

- 创建 `docs/platform/k3-soc-overview.md`，只整理 K3 SoC 级 CPU/hart、内存控制器、启动能力和与后续平台工作直接相关的资源能力。
- 创建 `docs/platform/com260-board-resources.md`，把 K3 SoC、CoM260 模组和目标 Kit/载板三层分开，整理 DRAM、UFS、SPI Flash、TF Card、debug UART、GMAC/PHY、连接器及供电归属。
- 创建 `docs/boot/com260-boot-chain.md`，整理启动模式、启动介质优先级、Boot ROM、OpenSBI、U-Boot 与后续 payload 的已证事实和未知边界。
- 创建 `docs/boot/com260-image-and-dts.md`，整理镜像类型、写入方式、DTS/compatible 的使用入口，以及装载地址和内存布局的证据状态。
- 更新 `docs/index.md`，把 platform 与 boot 职责链接到实际文档。
- 在引用 CoM260 datasheet、user guide 或具体 DTS 前，先把尚未登记的直接 URL 增加到 `source-coverage.md`；只在发现现有 G1-G6 未覆盖的新缺口时更新 `known-gaps.md`。
- 为后续 MS04-MS07 输出可引用的板级事实、证据等级和未知项，不修改 StarryOS。

## Capabilities

### New Capabilities

- `k3-com260-board-boot-baseline`: 定义 K3 SoC、CoM260 模组/Kit、启动链、镜像与 DTS 的分层聚合和验收行为。

### Modified Capabilities

- None.

## Scope Decisions

- 默认交付四篇主题文档；若任一文件接近 450 行，按 `document-template.md` 在同一主题目录继续拆分，不删减事实类别。
- “K3 CoM260”默认指模组；只有官方产品资料或目标 DTS 明确证明载板/Kit 组成时，才写 Kit 级事实。证据不足时保留未知项，不把 Pico-ITX 资源移植到 CoM260。
- SoC 的“具备某控制器”不等于 CoM260 板上已连接或可用。资源矩阵必须分别记录 SoC 能力、模组引出、载板连接和当前证据等级。
- 启动文档只记录来源明确给出的介质、阶段和镜像关系。装载地址、DRAM 保留区、固件交接寄存器或 DTS 选择无法由官网、对应官方仓库或目标 DTS 证明时，写为未知项。
- `docs-product`、`docs-chip`、`docs-buildroot` 和 `linux-6.18` 只作 `交叉验证`；官网正文仍是唯一 `官方事实` 来源。
- 新发现 URL 必须先进入覆盖表才能出现在主题文档首行或正文引用中；新增行不构成来源 refresh，也不改变未检查行的观察日期。
- 本 change 不把 G3-G6 等既有缺口伪装成已解决；只按获得的证据更新其状态或增加不重复的新缺口。

## Scenario Gaps and Defaults

- CoM260 datasheet 与 user guide 尚未登记在 38 行覆盖表中，默认在本 change 内新增为 `official-doc | CoM260 | platform/boot | current | active`；最终访问状态、修订和备注由 Phase 2 调查确认。
- 若官网页面只能返回 SPA 壳，默认保持该 URL 的官网事实边界，并使用对应 SpacemiT 官方 GitHub 文档或目标 DTS 作 `交叉验证`；不得把交叉验证改写为官网事实。
- 若同一参数在 datasheet、user guide、DTS 或 SDK 文档间冲突，默认并列记录来源、适用层级和版本，不自行选择一个“正确值”。冲突会影响目标板结论时，保留未知项并列出解除条件。
- 若找不到能唯一对应 CoM260 目标 Kit 的 DTS，默认不使用 K3 Pico、其他载板或通用 K3 DTS 替代；只记录候选路径和缺失影响。
- 本仓库没有运行时并发。取消、网络超时和来源不可访问按 MS02 的人工 refresh/停止规则处理；已经取得的来源事实可以保留，未取得部分不得推断。
- G1-G6 已覆盖的范围盘点、GMAC/PHY、AIA、DMA/IOMMU 和 programmer reference 缺口只引用或按证据更新，不创建同义重复条目。

## Scenario Sketch

- **正常聚合**：来源可读且层级明确时，事实进入对应主题文档，标注来源与证据等级，并从 `docs/index.md` 可达。
- **芯片能力与板级连接分离**：datasheet 只证明控制器能力时，SoC 文档记录能力；板级文档不得声称 CoM260 已连接该设备。
- **模组与 Kit 分离**：CoM260 资料只描述核心板时，模组事实正常记录；载板接口保持未知，直到产品资料或目标 DTS 明确。
- **SPA fallback**：官网正文不可读但官方 GitHub 对应文档可读时，事实标为 `交叉验证`，官网 URL 不获得虚假的正文观察状态。
- **来源冲突**：多个来源值不一致时，记录版本、层级和冲突；影响后续 bring-up 的值保持未知。
- **DTS 不匹配**：没有唯一目标 DTS 时，不借用 Pico 或通用 DTS；镜像与 DTS 文档给出候选及解除条件。
- **启动参数缺失**：装载地址、DRAM 布局或 handoff 细节无直接证据时，不推导常量；登记未知项和影响主题。
- **范围边界**：发现 UART、IRQ、DMA 或 GMAC 寄存器级细节时，只记录指向和缺口，不展开 MS04-MS07 正文。
- **中断或取消**：已完成文档保留可验证结果，未完成任务保持未勾选并记录恢复点。

## Non-goals

- 不聚合 Pico-ITX、K3 Pico 模组、RV2768、Shelf 或其他非 CoM260 板卡正文。
- 不实现或修改 StarryOS platform、bootloader、驱动或 DTS。
- 不提前完成 MS04 的 clock/reset/pinctrl/UART、MS05 的中断与时间、MS06 的 DMA/IOMMU 或 MS07 的 GMAC/PHY 数据面。
- 不从 K1、通用 DesignWare IP、Linux 框架默认值或相邻板型推定 K3 常量。
- 不创建 scraper、渲染器、链接检查脚本、页面快照、manifest、run ID、Hash 账本或 Evidence 占位目录。
- 不刷新未在本 change 明确选择的来源观察日期，不直接维护 SNAPSHOT、全局 tasks 或 M/D/K/R/I。

## Impact

- 计划中的产品修改为四篇新主题文档、`docs/index.md`、`docs/reference/source-coverage.md`，以及条件性的 `docs/reference/known-gaps.md`。
- OpenSpec change 保存需求、设计、任务、Cycle 和验证结果；仓库仍为纯 Markdown，无新增运行时依赖。
- MS04-MS07 可以引用本 change 的 SoC/模组/Kit 分层、启动阶段、资源矩阵与未知项，不必重新遍历全部产品资料。

## Gate 1

- Status: approved
- User approval: `同意`
- Proposed scope: 四篇主题文档、必要的覆盖行和索引更新，以及条件性的缺口更新。

| Check | Status | Evidence |
| --- | --- | --- |
| BDD gap scan | PASS | 覆盖正常、层级混淆、SPA fallback、来源冲突、DTS 不匹配、参数缺失、取消/超时和范围兼容性 |
| Gap decisions | PASS | 用户接受 Scenario Gaps and Defaults |
| Scenario sketch | PASS | 本 proposal 与 delta spec 覆盖正常、失败和边界路径 |
| OpenSpec change | PASS | `establish-k3-com260-board-boot-baseline` 已创建 |
| Requirements and scope approval | PASS | 用户原话：`同意` |
