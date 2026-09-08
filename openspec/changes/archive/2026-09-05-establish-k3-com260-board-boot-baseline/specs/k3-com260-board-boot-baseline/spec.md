## ADDED Requirements

### Requirement: K3 SoC 与 CoM260 板级事实分层

聚合文档 SHALL 分别记录 K3 SoC、CoM260 模组和目标 Kit/载板事实，并 SHALL 不把某一层的能力或连接关系提升为另一层的事实。

#### Scenario: SoC 能力有直接来源

- **WHEN** K3 datasheet 或 root overview 明确陈述 CPU/hart、内存控制器、存储或外设能力
- **THEN** `k3-soc-overview.md` 按来源记录为 SoC 级事实，不声称 CoM260 已引出、连接或启用该资源

#### Scenario: CoM260 模组资源有直接来源

- **WHEN** CoM260 overview、datasheet 或 hardware resources 明确陈述模组集成器件、内存、存储、接口或引脚
- **THEN** `com260-board-resources.md` 将其归到模组层，并与 SoC 能力和载板连接分栏表达

#### Scenario: Kit 或载板证据不足

- **WHEN** 当前来源只证明 CoM260 核心板，不能证明目标 Kit 的载板组成、连接器或板载设备
- **THEN** 文档把 Kit/载板字段标为 `未知项`，列出当前证据、禁止推断、解除条件和影响主题

#### Scenario: 非目标板资料可见

- **WHEN** Pico-ITX、K3 Pico 或其他 K3 板卡提供相似资源
- **THEN** 不把其事实写入 CoM260 结论，只可在必要时标为非目标板对照且不得出现在直接来源首行

### Requirement: CoM260 资源矩阵可供后续规划引用

板级文档 SHALL 覆盖 MS03 指定的 CPU/hart、DRAM、UFS、SPI Flash、TF Card、debug UART、GMAC/PHY 和连接器，并 SHALL 为每项记录层级、来源、证据等级和未知边界。

#### Scenario: 资源归属明确

- **WHEN** 官方资料明确给出资源属于 SoC、模组或载板，以及其容量、数量、接口或连接关系
- **THEN** 资源矩阵记录该层级和可证字段，并以 `官方事实` 标注

#### Scenario: 只有官方仓库或 DTS 佐证

- **WHEN** 资源字段只在 SpacemiT 官方 GitHub 文档、目标 DTS 或官方内核中可见
- **THEN** 该字段标为 `交叉验证`，不得覆盖官网冲突或提升为官网事实

#### Scenario: 能力不能证明可用性

- **WHEN** SoC 支持某控制器，但没有 CoM260 引出、供电、pinctrl、clock/reset 或载板连接证据
- **THEN** 文档只记录 SoC 能力，并把 CoM260 可用性保留为未知项

#### Scenario: 来源之间存在冲突

- **WHEN** datasheet、user guide、DTS 或 SDK 文档对同一资源给出不同值
- **THEN** 文档并列记录来源版本和适用层级；影响目标板结论时不自行裁决，并给出解除条件

### Requirement: 启动链与启动介质保持可追溯

启动文档 SHALL 记录 K3/CoM260 的启动模式、启动介质、阶段责任和 handoff 边界，并 SHALL 不推定来源没有给出的地址或固件状态。

#### Scenario: 启动模式和介质有直接来源

- **WHEN** K3 datasheet、CoM260 文档或 SDK boot 文档明确列出 download/local boot、SD、eMMC、SPI NOR/NAND、UFS、USB Fastboot 或 UART Xmodem
- **THEN** `com260-boot-chain.md` 记录支持层级、选择或优先级及对应来源，不把 SoC 支持自动写成板级可用

#### Scenario: 固件阶段有证据

- **WHEN** SDK 文档、镜像说明或对应官方仓库明确描述 Boot ROM、OpenSBI、U-Boot 和 payload/OS 的产物或顺序
- **THEN** 文档按证据记录阶段输入、输出与已知 handoff，不补写未陈述的寄存器状态

#### Scenario: 装载地址或 DRAM 布局缺失

- **WHEN** 来源不能直接证明 load address、reserved-memory、DRAM 可用区或固件占用
- **THEN** 这些字段写为 `未知项`，不得从通用 RISC-V、K1、Pico 或工具默认值推断

#### Scenario: 启动来源版本不一致

- **WHEN** boot、image、release notes、DTS 或 bootloader 来源属于不同版本
- **THEN** 文档标出各自版本或观察日期，只在兼容关系有证据时合并为同一启动基线

### Requirement: 镜像与 DTS 使用边界明确

镜像与 DTS 文档 SHALL 说明可证的镜像类型、写入方式、目标 compatible 和设备树作用，并 SHALL 在目标 DTS 不唯一时保留候选与缺口。

#### Scenario: 镜像流程有直接来源

- **WHEN** SDK image 或 boot 文档明确描述 SD image、fastboot 包或其它制品及写入方式
- **THEN** `com260-image-and-dts.md` 记录制品、介质和操作边界，不把示例文件名或构建时间当作稳定身份

#### Scenario: 唯一 CoM260 DTS 可确认

- **WHEN** SpacemiT 官方仓库中存在与目标 CoM260/Kit 明确匹配的 DTS、compatible 和 include 链
- **THEN** 文档记录该路径及可观察的 memory、chosen、aliases 和设备启用关系，证据等级为 `交叉验证`

#### Scenario: 目标 DTS 不能唯一确认

- **WHEN** 只找到 K3 Pico、通用 K3、多个 CoM260 变体或命名不足以确认目标 Kit 的 DTS
- **THEN** 文档不选择替代 DTS，列出候选、差异、缺失影响和唯一化所需证据

### Requirement: 来源覆盖和文档导航与新增正文一致

本 change SHALL 在引用新来源前登记 URL，并 SHALL 使新增主题文档符合模板、从总入口可达且不重复既有缺口。

#### Scenario: 主题需要未登记的官方 URL

- **WHEN** CoM260 datasheet、user guide 或目标 DTS 将作为直接来源或交叉验证来源
- **THEN** `source-coverage.md` 先增加唯一完整行，再由主题文档引用该 URL；既有未检查行不变

#### Scenario: 新主题文档交付

- **WHEN** 四篇主题文档完成
- **THEN** 每篇首行来源合规、事实逐项标注证据等级、未知项具备解除条件，并从 `docs/index.md` 使用相对链接可达

#### Scenario: 文档接近行数上限

- **WHEN** 任一主题文档接近 450 行或将超过 500 行建议上限
- **THEN** 按同一主题子域拆分并保留导航、独立来源首行和完整事实类别，不以删减需求规避拆分

#### Scenario: 缺口已由 G1-G6 覆盖

- **WHEN** MS03 发现的未知项与现有范围、GMAC/PHY、AIA、DMA/IOMMU 或 programmer reference 缺口语义相同
- **THEN** 主题文档引用现有缺口；只有新证据改变状态时才更新该条目，不创建同义重复缺口

#### Scenario: 发现超出 MS03 的寄存器级资料

- **WHEN** 来源包含 clock/reset/pinctrl/UART、IRQ、DMA 或 GMAC 的寄存器和驱动细节
- **THEN** 本 change 只保存与板级归属或启动依赖直接相关的指向，不展开 MS04-MS07 正文，也不修改 StarryOS
