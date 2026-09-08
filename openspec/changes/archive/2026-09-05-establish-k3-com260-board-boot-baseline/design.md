## Context

MS01-MS02 已建立 38 行来源覆盖表、主题文档模板和一次真实 refresh 基线，但 `docs/` 仍只有总入口与 reference 文档。MS03 是第一批技术正文，需要同时协调 K3 SoC、CoM260 模组、CoM260 Kit/载板、K3 Buildroot SDK 和 Linux DTS 五类证据。如果不先固定层级，后续会把 SoC 控制器能力误写成板上连接，或把某个 CoM260 变体 DTS 当成当前目标 Kit。

Plan 调查确认：K3 datasheet 可提供 CPU、内存控制器和启动介质等 SoC 能力；CoM260 datasheet V1.2 与 user guide V2.0 可分别提供模组和开发套件事实；官方 `linux-6.18` 的 K3 DTS 目录同时包含 `k3_com260.dts`、`k3_com260_ifx*.dts`、`k3_com260_kit_v02.dts`、`k3_com260_tq.dts` 等多个变体。user guide 所示产品版本含 `v03`，当前证据不能把它与文件名含 `v02` 的 DTS 唯一对应。

官网正文仍是唯一 `官方事实` 来源。SpacemiT 官方 GitHub 文档和 DTS 只提供 `交叉验证`；官网为 SPA 壳时，不能把仓库正文提升为官网事实。当前覆盖表尚未登记 CoM260 datasheet、user guide、所用 GitHub 文档页和 DTS 目录的精确 URL，必须先补行再引用。

本仓库是纯 Markdown，没有运行时入口、并发状态或代码测试。实现关键路径是：登记精确来源 → 建立 SoC/模组/Kit 分层 → 形成资源矩阵 → 建立启动阶段与镜像边界 → 记录 DTS 非唯一性和未知项 → 接入总入口。

## Goals / Non-Goals

**Goals:**

- 形成四篇可独立引用的 K3/CoM260 platform 与 boot 主题文档。
- 让每项板级结论同时表达对象层级、来源、证据等级和未知边界。
- 为 CPU/hart、DRAM、UFS、SPI Flash、TF Card、debug UART、GMAC/PHY 和连接器提供统一资源矩阵。
- 固定 Boot ROM、OpenSBI、U-Boot、payload/OS、镜像和 DTS 的已证关系，不补造地址或 handoff 状态。
- 在目标 DTS 不唯一时保存候选、影响和解除条件，供 MS04-MS07 继续调查。
- 以两个可独立验证的逻辑 Iteration 完成 MS03。

**Non-Goals:**

- 不聚合 Pico、RV2768、Shelf 或其他载板正文。
- 不修改 StarryOS、bootloader、Linux、DTS 或任何可执行代码。
- 不展开 UART、IRQ、DMA/IOMMU、GMAC/PHY 的寄存器级实现。
- 不从 K1、通用 RISC-V、DesignWare IP、Linux 默认值或相邻板型推定 K3 常量。
- 不执行来源 refresh，不改写未选择来源的观察日期。

## Decisions

### D1：使用四篇主题文档而非单篇事实包

`platform/k3-soc-overview.md` 只承担 SoC 能力，`platform/com260-board-resources.md` 承担模组与 Kit 资源，`boot/com260-boot-chain.md` 承担启动阶段，`boot/com260-image-and-dts.md` 承担制品与设备树。这样每篇都能保持一个清晰责任边界，也避免接近 500 行时再从混合文档中拆事实。

替代方案是按官网页面逐页镜像。该方式会复制来源目录结构，并使同一资源散落在 datasheet、user guide 和 SDK 页面中，不符合 D02 的主题驱动结构，因此不采用。

### D2：所有资源使用四层事实模型

板级资源矩阵固定使用 `SoC 能力`、`CoM260 模组`、`Kit/载板`、`未知边界` 四层。来源只证明某层时，其他层不得自动继承。CPU/hart 也保留该模型：SoC 核心数量不自动证明软件可用 hart 集合。

事实强度继续使用 `官方事实`、`交叉验证`、`推论`、`未知项`。本 change 允许必要的推论，但必须列出组成事实且不能产生地址、容量、连接或版本的新常量。

替代方案是按资源只保留一个“最终值”。该方式无法表达芯片能力与板上连接的差异，也无法容纳版本冲突，因此不采用。

### D3：精确 URL 先登记，来源身份不升级

主题文档实际引用的精确 URL 必须先写入 `source-coverage.md`。Iteration 000 登记 K3 datasheet 的 GitHub 对应页、CoM260 datasheet/user guide 的官网与 GitHub 对应页，以及 hardware-resources 的 GitHub 对应页；Iteration 001 登记 boot、image 的 GitHub 对应页和 K3 DTS 目录。

官网 URL 使用 `official-doc`；官方 GitHub 页面和 DTS 目录使用 `supporting-source`。官网正文不可读时，官网行保持 `partially-observed` 或本次可证状态，GitHub 内容只标 `交叉验证`。新增行以 Act 的实际观察日期为准，不批量刷新既有行。

替代方案是只引用已登记的仓库根 URL。该方式不能把事实追溯到具体页面或 DTS 集合，不满足文档模板，因此不采用。

### D4：分两个 Iteration 交付稳定基线

Iteration 000“板级事实层次与资源矩阵”完成来源登记、SoC 概述和 CoM260 资源矩阵。其稳定结果是启动文档可直接引用的目标硬件层次、资源归属和 Kit 身份边界。

Iteration 001“启动链、镜像与 DTS”在前一基线上完成启动链、镜像/DTS 文档、必要的 G3 更新或新缺口登记，以及 `docs/index.md` 导航。启动链依赖目标板层次，DTS 候选又影响板级连接结论，因此不并行实施。

替代方案是一个 Iteration 一次交付四篇正文。该方案把平台层级错误和启动/DTS 错误混在同一诊断边界中，工作量也偏大，因此不采用。

### D5：DTS 以“候选集合”表达，除非出现唯一映射证据

当前 DTS 目录证明存在多个 CoM260 变体，但文件名不证明某个 DTS 就对应 user guide 中的目标 Kit。Iteration 001 必须记录候选路径、命名差异、可证 include/compatible（只有直接打开文件后才可写）以及唯一化所需证据。

如果 Act 取得官方产品资料与 DTS 的明确映射，才可记录唯一目标 DTS；否则增加一个不与 G1-G6 重复的目标 Kit/DTS 映射缺口。不得使用 Pico 或通用 K3 DTS 代替。

### D6：未知项使用统一闭包

每个会影响 bring-up 的未知项至少写明：当前证据、禁止推断、解除条件、影响主题。装载地址、DRAM 可用区、reserved-memory、handoff 寄存器、目标 compatible、GMAC 实例和 PHY 参数均适用。与 G3-G6 相同的未知项引用或补充原条目，不创建同义项。

### D7：验证直接检查文档行为

RED 见证使用文件不存在、缺少必要章节/资源行或总入口无链接；GREEN 检查首行来源、证据等级、资源类别、相对链接、URL 唯一性、行数和全量 diff。验证结果写入 Cycle 的 Act Response，Persisted Evidence 为 `none`。

不创建快照、Hash 清单、manifest、run ID 或辅助脚本来替代正文检查。

## Risks / Trade-offs

- [官网 SPA 正文不可读] → 保留官网身份和访问边界，只使用已登记的官方 GitHub 页面做 `交叉验证`，无法证明的字段写未知项。
- [datasheet、user guide 与 DTS 版本不一致] → 并列版本、层级和适用范围；影响目标板结论时不裁决。
- [资源矩阵过宽而侵入后续 milestone] → MS03 只写资源存在性、归属和启动相关字段；寄存器、IRQ、DMA 与驱动时序指向 MS04-MS07。
- [DTS 候选较多] → 只列 CoM260 命名候选与可证差异，不复制整棵 DTS，也不选择替代板型。
- [文档接近 450 行] → 在同一主题目录按职责拆分并同步导航；不得删减事实类别。
- [新增来源导致覆盖表计数变化] → 同步表头中的实际行数与唯一性说明，不把原 38 行当作完成后的固定值。

## Migration Plan

1. Iteration 000 先登记 platform 所需精确来源，再创建 SoC 概述和 CoM260 板级资源文档。
2. 独立验证事实层次、八类资源、来源等级、未知项与行数，接受后形成平台稳定基线。
3. Iteration 001 登记 boot/image/DTS 精确来源，创建启动链和镜像/DTS 文档。
4. 按实际证据更新 G3 或新增目标 Kit/DTS 映射缺口，再更新 `docs/index.md`。
5. 验证四篇正文、覆盖表、缺口表和总入口的一致性，并进行完整 diff Review。

回退时只撤销本 change 新增的精确来源行、四篇主题文档、索引链接和实际缺口变化；不回退 MS01-MS02 的基线。全局 milestone、SNAPSHOT 和 M/D/K/R/I 由 accepted 后的 docs-maintainer 收尾。

## Open Questions

None. 来源在 Act 时能否读取及 DTS 是否可唯一映射属于已定义结果分支，不留给 Act 临时决定契约。
