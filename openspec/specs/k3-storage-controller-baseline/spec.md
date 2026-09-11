# k3-storage-controller-baseline Specification

## Purpose
建立 K3 QSPI、SPI、SDHC 和 UFS 的可追溯知识契约，使读者能够按设备查询资源、启动关系、数据路径和恢复边界，并区分官方事实、软件行为、第三方经验与未知项。

## Requirements

### Requirement: 存储控制器资源和板级可达性可追溯

知识基线 SHALL 分别描述 QSPI、SPI、SDHC 和 UFS 的控制器实例、DTS、地址、clock/reset、pinctrl、IRQ 与 DMA 资源，并区分 SoC 集成能力、CoM260 板级连接和运行时启用状态。

#### Scenario: 查询控制器静态资源

- **WHEN** 读者查询任一存储控制器的实例或平台资源
- **THEN** 文档给出已确认字段、来源、证据等级和适用设备，不把其他控制器的参数外推到当前设备

#### Scenario: 板级连接或启用状态未知

- **WHEN** DTS 节点、控制器能力和 CoM260 板级资料不能共同证明设备已连接或启用
- **THEN** 文档分别记录 SoC 能力、板级证据、运行时未知项和解除条件，不声明该路径可用

### Requirement: 启动介质关系按阶段说明

知识基线 SHALL 说明 QSPI、SPI、SDHC 和 UFS 在已证启动链中的候选或实际作用，并区分 Boot ROM、SPL/U-Boot、固件与 OS 阶段。

#### Scenario: 查询已确认的启动路径

- **WHEN** 官方启动资料或可审计软件配置把某存储介质关联到一个启动阶段
- **THEN** 文档说明参与者、介质作用、阶段交接和来源，并交叉引用既有启动基线

#### Scenario: 默认介质或启动目标不唯一

- **WHEN** 板型、DTS、镜像或启动模式不足以确定唯一介质和目标
- **THEN** 文档保留候选集合及选择条件，不指定未经证实的默认路径

### Requirement: 数据路径和资源所有权按设备分层

知识基线 SHALL 按控制器描述命令、descriptor 或 buffer、DMA、cache maintenance、提交、完成与回收路径，并引用而不重复定义 MS06 的通用内存所有权边界。

#### Scenario: 查询正常传输路径

- **WHEN** 资料足以确认某控制器从 CPU 准备到设备完成的数据路径
- **THEN** 文档给出状态变化、资源所有者、可见性动作、完成条件和证据适用范围

#### Scenario: 只有能力或静态资源描述

- **WHEN** 资料只显示 DMA、IRQ 或 descriptor 能力，不能证明运行时采用方式
- **THEN** 文档把能力与已观察行为分开，不把静态存在表述为实际路径

### Requirement: UFS 协议栈和设备边界独立

知识基线 SHALL 单独描述 UFS 的 MPHY、UniPro、UTP/SCSI、descriptor、DMA 与 cache 层次，不把 UFS 专有行为外推到 QSPI、SPI 或 SDHC。

#### Scenario: 查询 UFS 分层路径

- **WHEN** 读者查询 UFS 命令从软件提交到设备完成的路径
- **THEN** 文档按已确认材料描述协议层、数据结构、状态变化和完成通知，并标明官方与第三方证据边界

#### Scenario: 缺少协议或硬件细节

- **WHEN** 官方资料不能确认 MPHY、UniPro、UTP/SCSI 或 descriptor 的某项 K3 语义
- **THEN** 文档保留未知项和解除条件，不用通用 UFS 规范或第三方实现替代 K3 硬件事实

### Requirement: 错误、超时与恢复语义可查询

知识基线 SHALL 记录已证的命令错误、轮询、超时、fatal error、链路失败、取消、复位和重新初始化路径，包括返回结果与未完成资源的状态。

#### Scenario: 错误路径已有可审计实现

- **WHEN** 官方软件或固定 revision 第三方实现处理错误、超时或复位
- **THEN** 文档说明触发条件、状态变化、恢复动作、返回语义和适用范围

#### Scenario: 恢复完整性无法确认

- **WHEN** 资料不能证明恢复是否保留未完成请求、buffer、descriptor 或链路状态
- **THEN** 文档明确潜在数据或状态损失及解除条件，不声称无损恢复或安全重试

### Requirement: 来源冲突和不可访问状态不被隐藏

知识基线 SHALL 区分官方硬件事实、官方软件行为、固定 revision 第三方经验、推论与未知项，并按既有来源规则处理 SPA 不可访问和资料冲突。

#### Scenario: 来源可交叉验证

- **WHEN** 官方页面、DTS、驱动或其他已登记材料支持同一结论
- **THEN** 文档记录每个来源的责任和证据等级，不用来源数量替代适用性判断

#### Scenario: 来源不可访问或发生冲突

- **WHEN** 官方页面不可读取，或不同材料给出不一致的实例、参数、路径或恢复行为
- **THEN** 文档记录冲突、当前可确认范围和解除条件，不选择未经证实的结论

### Requirement: 存储主题导航和既有基线保持一致

存储主题 SHALL 接入总索引、来源覆盖、术语与已知缺口，并与启动、平台、DMA 和中断主题使用相对链接保持单一职责。

#### Scenario: 存储主题完成聚合

- **WHEN** QSPI、SPI、SDHC 和 UFS 主题文档加入仓库
- **THEN** 总索引可进入每个主题，相关来源有唯一覆盖记录，缺口包含当前证据和解除条件，术语与交叉引用一致

#### Scenario: 新证据与既有基线冲突

- **WHEN** 存储调查发现与 MS03、MS04、MS05 或 MS06 已记录事实冲突的证据
- **THEN** 计划停止静默覆盖，记录影响并决定在当前 change 修正受影响文档或另建来源 refresh change
