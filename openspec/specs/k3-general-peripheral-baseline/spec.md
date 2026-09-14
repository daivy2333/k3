# k3-general-peripheral-baseline Specification

## Purpose

为 K3 GPIO、PWM、IR-RX、Audio、WDT 和 RTC 建立可追溯的通用外设知识契约，使 SoC 能力、DTS 静态资源、CoM260 板级连接、软件路径与未验证运行状态能够被明确区分。

## Requirements

### Requirement: 通用外设资源与板级可达性分层

知识库 SHALL 对 GPIO、PWM、IR-RX、Audio、WDT 和 RTC 分别记录 SoC 能力、控制器实例、MMIO、clock/reset、pinctrl、IRQ、DMA、power domain 或端点依赖，并区分 SoC 集成、模组引出、Kit 连接、DTS 启用状态和运行证据。

#### Scenario: 资源和连接均有证据

- **WHEN** 官方资料或官方 DTS 提供控制器资源和 CoM260 连接信息
- **THEN** 主题文档按层列出字段、来源和证据等级，不把任一层替代另一层

#### Scenario: 板级映射不能唯一确定

- **WHEN** 只有 SoC 能力、复用候选或多个 CoM260 DTS 变体，不能唯一确定目标板连接
- **THEN** 主题文档 SHALL 保留候选和未知项，不指定默认 DTS 或推定板级可用性

### Requirement: GPIO 控制、复用与中断责任分离

知识库 SHALL 区分 GPIO controller、pinctrl mux、电气配置、GPIO consumer 和 interrupt controller 责任，并记录输入、输出、置位、清除、读取和边沿中断能力的证据边界。

#### Scenario: GPIO 静态资源可确认

- **WHEN** K3 datasheet 或 DTS 提供 GPIO controller、bank/range、clock、IRQ 和 pinctrl 关系
- **THEN** 主题文档 SHALL 记录控制器资源及责任方向，不把 pinmux 候选解释为 GPIO 已被安全配置

#### Scenario: 真板电气状态未知

- **WHEN** 缺少目标 DTS、外部上拉下拉、电压域、占用者或真板输入输出证据
- **THEN** 主题文档 SHALL 把具体引脚可用性标为未知，不给出会改变板级状态的操作结论

### Requirement: PWM 通道、波形能力与 consumer 分离

知识库 SHALL 记录 PWM controller/channel、clock/reset、pinmux、频率与占空比能力和板级 consumer，并对相互冲突的通道数量保留来源、观察范围和解除条件。

#### Scenario: PWM 数量来源冲突

- **WHEN** K3 datasheet 概览声明 30 路 PWM，而详细章节和 DTS 只描述 PWM0–PWM19
- **THEN** 主题文档 SHALL 并列记录 30 与 20 的来源和适用范围，不静默选择一个值作为统一事实

#### Scenario: 板级 PWM 候选存在

- **WHEN** CoM260 资料提供 PWM 复用引脚或 FAN_PWM 信号
- **THEN** 主题文档 SHALL 区分 pin/consumer 事实、DTS 启用状态和波形运行结果，不因信号存在声明通道已可用

### Requirement: IR-RX 控制器与输入事件边界可追溯

知识库 SHALL 记录 IR-RX 控制器的 MMIO、IRQ、clock/reset、DTS 状态和候选 pinmux，并把硬件接收资源与协议解码、input event 和遥控器兼容性分开。

#### Scenario: 控制器存在但默认禁用

- **WHEN** K3 DTS 提供 IR-RX 节点且状态为 `disabled`
- **THEN** 主题文档 SHALL 记录静态资源，但不声明目标板已启用红外接收

#### Scenario: 输入协议或板级映射缺失

- **WHEN** 官方页面不可读，或缺少目标引脚、协议、keymap 和运行日志
- **THEN** 主题文档 SHALL 记录未知项和解除条件，不用通用 Linux 输入行为补写 K3 专属事实

### Requirement: Audio 数据链按端点和所有权分层

知识库 SHALL 区分 I2S/SSPA controller、CPU DAI、sound card、codec 或显示端点、clock/reset、DMA、power domain 和 CoM260 引脚，并引用既有 DMA/cache 所有权基线而不重复定义。

#### Scenario: DTS 提供音频链静态资源

- **WHEN** K3 或 CoM260 DTS 提供 I2S、DMA、sound card、display audio 或 power domain 节点
- **THEN** 主题文档 SHALL 描述端点与依赖方向、启用状态和证据等级，不把静态链解释为音频流已运行

#### Scenario: codec 或运行参数未知

- **WHEN** 缺少 codec 型号、板级 route、采样格式、stream 日志、buffer 生命周期或 xruns 证据
- **THEN** 主题文档 SHALL 保留未知项，并通过 MS06 引用说明 DMA/cache 边界

### Requirement: WDT 与 RTC 的状态和生命周期责任分离

知识库 SHALL 分开记录 watchdog 计数、超时和复位路径，MMIO RTC 的 timekeeping/alarm 路径，以及 RPMI RTC 的固件代理和 mailbox/IRQ 依赖。

#### Scenario: WDT 静态节点可确认

- **WHEN** datasheet 或 DTS 提供 WDT 位宽、计数时钟、clock/reset、IRQ 或 restart 属性
- **THEN** 主题文档 SHALL 记录静态能力和配置字段，不据此声明超时复位范围、默认启用状态或恢复结果

#### Scenario: 两条 RTC 路径同时存在

- **WHEN** DTS 同时提供 MMIO RTC 和 RPMI RTC 节点
- **THEN** 主题文档 SHALL 分别记录资源、IRQ、mailbox 和启用状态，不推定二者的运行所有权、时间同步或 alarm 唤醒关系

#### Scenario: 掉电保持和唤醒证据缺失

- **WHEN** 只有 VCC_RTC、RTC clock 或静态 alarm IRQ 信息
- **THEN** 主题文档 SHALL 不声明掉电计时保持、唤醒能力或真板时间精度已经验证

### Requirement: 来源冲突和不可访问状态可追溯

知识库 SHALL 为六类通用外设保留官方入口，以 URL 为覆盖记录唯一键，并对不可访问、资料冲突、DTS 变体或 supporting source 的适用范围使用统一证据等级和未知项字段。

#### Scenario: 官方页面可读取

- **WHEN** 官方页面可直接读取
- **THEN** 主题文档和来源覆盖 SHALL 记录实际来源、源端修订或 `unknown`、观察日期和聚合职责

#### Scenario: 官方页面不可访问

- **WHEN** 官方 SPA 或 GitHub 对应页无法直接读取
- **THEN** 聚合结果 SHALL 保留权威 URL 和不可达状态，只使用已实际核对的 supporting evidence，并记录未解除的事实缺口

### Requirement: 通用外设导航和术语保持一致

知识库 SHALL 从总索引提供所有实际创建的通用外设主题入口，并使主题职责、来源覆盖、术语、已知缺口和汇总计数一致。

#### Scenario: 通用外设主题完成聚合

- **WHEN** GPIO、PWM、IR-RX、Audio、WDT 和 RTC 正文完成
- **THEN** 总索引 SHALL 提供有效相对链接，来源覆盖和缺口 SHALL 有唯一正文落点，新增术语 SHALL 无重复主写法

#### Scenario: 新证据与既有基线冲突

- **WHEN** 新材料与 MS03–MS06 或现有缺口记录冲突
- **THEN** 本 change SHALL 停止静默覆盖并保留冲突，直到 Plan 判断其属于本 change 修正或独立 refresh