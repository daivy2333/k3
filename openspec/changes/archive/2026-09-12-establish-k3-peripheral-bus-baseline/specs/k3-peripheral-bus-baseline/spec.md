## Purpose

为 K3 I2C、USB、PCIe、CAN 和 EtherCAT 建立可追溯的总线知识契约，使控制器能力、DTS 静态资源、CoM260 板级连接、软件行为和未验证运行状态能够被明确区分。

## ADDED Requirements

### Requirement: 总线资源与板级可达性分层

知识库 SHALL 对 I2C、USB、PCIe、CAN 和 EtherCAT 分别记录 SoC 能力、控制器实例、MMIO、clock/reset、pinctrl、IRQ、DMA、PHY 或协议依赖，并区分 SoC 集成、模组引出、Kit 连接和 DTS 启用状态。

#### Scenario: 资源和连接均有证据

- **WHEN** 官方资料或官方 DTS 提供控制器资源和 CoM260 连接信息
- **THEN** 主题文档按层列出字段、来源和证据等级，不把任一层替代另一层

#### Scenario: 板级映射不能唯一确定

- **WHEN** 只有 SoC 能力、相邻板型或多个 CoM260 DTS 变体，不能唯一确定目标板连接
- **THEN** 主题文档 SHALL 保留候选和未知项，不指定默认 DTS 或推定板级可用性

### Requirement: I2C 与 USB 设备链保持对象边界

知识库 SHALL 区分 I2C controller 与 client，并区分 USB controller、USB2/USB3 PHY、Host、DRD、Type-C 控制器、role switch、Hub 和 BootROM 下载用途。

#### Scenario: DTS 提供复合设备依赖

- **WHEN** CoM260 DTS 同时出现 I2C 从属设备、USB PHY、role switch 或 Hub
- **THEN** 主题文档 SHALL 描述各对象的依赖方向、板级接口和变体差异

#### Scenario: USB 子树或运行行为不可观察

- **WHEN** 官方 USB 子页未展开、不可访问或没有 K3 运行证据
- **THEN** 主题文档 SHALL 记录不可达范围和解除条件，不用通用 USB 行为补写 K3 专属事实

### Requirement: PCIe 静态拓扑与运行能力分离

知识库 SHALL 记录 PCIe controller、RC/EP 能力、lane/PHY、PERST#、CLKREQ#、板级插槽和预期设备用途，并把这些静态事实与 link training、枚举、MSI/MSI-X、NVMe 和热插拔运行结果分开。

#### Scenario: 静态端口映射可确认

- **WHEN** DTS 和板级资料足以确认 PCIe controller 到 lane、PHY 和插槽的映射
- **THEN** 主题文档 SHALL 给出映射及来源，但不据此声明设备已经枚举

#### Scenario: 只有 SoC 能力声明

- **WHEN** 资料只声明 K3 支持 RC/EP、热插拔或一定数量的 lanes
- **THEN** 主题文档 SHALL 把具体端口模式和运行结果标为未知

### Requirement: CAN 控制器、协议和物理层分离

知识库 SHALL 区分 AP/RP FlexCAN 控制器实例、CAN/CAN-FD 协议能力、pinctrl/clock/IRQ 配置、板载收发器和 Kit 连接器可用性。

#### Scenario: 控制器和收发器均存在

- **WHEN** DTS 提供 FlexCAN 节点且板级资料声明 CAN FD 收发器或连接器
- **THEN** 主题文档 SHALL 分别记录控制器与物理层，不据此声明 OS 已启用或真板通信成功

#### Scenario: 板级资料相互冲突

- **WHEN** Kit 文档同时描述 CAN 连接器存在和功能接口不可用
- **THEN** 主题文档 SHALL 保留冲突、禁止推断和解除条件，不选择其中一个表述覆盖另一个

### Requirement: EtherCAT 复用 GMAC 基线并保留协议边界

知识库 SHALL 描述 EtherCAT master 与 GMAC1 的静态依赖，引用既有 GMAC/PHY/DMA 基线，并单独记录 EtherCAT 软件组成、周期、同步、错误恢复和运行证据的已知或未知状态。

#### Scenario: 静态 master 绑定可确认

- **WHEN** DTS 中 `ec_master` 通过 `main-device` 绑定 `eth1`
- **THEN** 主题文档 SHALL 记录该静态依赖并引用既有 GMAC 文档，不重复定义 GMAC 数据路径

#### Scenario: 实时运行证据缺失

- **WHEN** 没有 EtherCAT 周期、同步精度、错误恢复或真板日志
- **THEN** 主题文档 SHALL 不声明协议已运行或满足实时性能

### Requirement: 来源冲突和不可访问状态可追溯

知识库 SHALL 为五类总线保留官方入口，以 URL 为覆盖记录唯一键，并对不可访问、目录未展开、资料冲突或 supporting source 的适用范围使用统一证据等级和未知项字段。

#### Scenario: 官方页面可读取

- **WHEN** 官方页面或其子页可直接读取
- **THEN** 主题文档和来源覆盖 SHALL 记录实际来源、源端修订或 `unknown`、观察日期和聚合职责

#### Scenario: 官方页面不可访问

- **WHEN** 官方 SPA、GitHub 页面或 USB 子页无法直接读取
- **THEN** 聚合结果 SHALL 保留权威 URL 和不可达状态，只使用已实际核对的 supporting evidence，并记录未解除的事实缺口

### Requirement: 总线导航和术语保持一致

知识库 SHALL 从总索引提供所有实际创建的总线主题入口，并使主题职责、来源覆盖、术语、已知缺口和汇总计数一致。

#### Scenario: 总线主题完成聚合

- **WHEN** I2C、USB、PCIe、CAN 和 EtherCAT 正文完成
- **THEN** 总索引 SHALL 提供有效相对链接，来源覆盖和缺口 SHALL 有唯一正文落点，新增术语 SHALL 无重复主写法

#### Scenario: 新证据与既有基线冲突

- **WHEN** 新材料与 MS03、MS04、MS05、MS07 或现有缺口记录冲突
- **THEN** 本 change SHALL 停止静默覆盖并保留冲突，直到 Plan 判断其属于本 change 修正或独立 refresh
