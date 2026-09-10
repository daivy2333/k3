## ADDED Requirements

### Requirement: 网络主题来源可追溯

系统 SHALL 为 CoM260 GMAC 网络主题的每个直接来源记录 URL、源端修订或 `unknown`、观察日期、适用对象和证据等级，并保持主题文档来源与覆盖表一一对应。

#### Scenario: 已访问来源进入主题文档

- **WHEN** 官网页面或已登记的官方 GitHub 对应材料可直接观察
- **THEN** 两篇网络主题文档的首行 SHALL 列出直接来源及其源端修订或 `unknown` 和观察日期，且每个 URL 在 `source-coverage.md` 中恰有一行

#### Scenario: 来源不可访问

- **WHEN** 官网页面只暴露 SPA 壳或某个 raw URL 无法访问
- **THEN** 系统 SHALL 记录访问边界，不得把未观察内容写成事实，并仅把已登记的官方 GitHub 材料标为交叉验证

#### Scenario: 多来源结论冲突

- **WHEN** 官网、DTS、Linux/U-Boot 或第三方材料对同一字段给出不同结论
- **THEN** 系统 SHALL 分别记录来源、适用对象和冲突，不得静默选择其中一个值

### Requirement: GMAC 能力与板级链路分层

系统 SHALL 分开描述 K3 SoC GMAC 能力、CoM260 模组引出、Kit 板级连接和 DTS 启用实例，并记录每层可证的 MMIO、IRQ、clock/reset、pinctrl 与 APMU 依赖。

#### Scenario: 形成实例与资源矩阵

- **WHEN** K3 SoC、CoM260 模组、Kit 和官方 DTS 材料完成核对
- **THEN** `com260-gmac-phy.md` SHALL 提供按层划分的 GMAC 实例、启用状态和平台资源矩阵

#### Scenario: 多个 CoM260 DTS 候选并存

- **WHEN** 基础款、Kit V02 或其他 CoM260 DTS 候选均存在但没有唯一目标映射
- **THEN** 系统 SHALL 分别记录已观察字段并保留目标 DTS 未确认状态，不得指定默认 Kit DTS

#### Scenario: 板级资料不完整

- **WHEN** 寄存器手册、原理图、binding 或驱动材料不足以确认某个字段
- **THEN** 系统 SHALL 把该字段写为未知项，并记录当前证据、禁止推断、解除条件和影响主题

### Requirement: MDIO PHY 与 RGMII 关系明确

系统 SHALL 描述 GMAC、MDIO bus、PHY handle、Clause 22 地址、PHY compatible、interface mode、reset 和 RGMII 调相字段之间的静态关系，同时区分配置输入与运行时行为。

#### Scenario: 静态 PHY 链可追踪

- **WHEN** `eth1`、MDIO 子节点和 PHY 节点均可直接观察
- **THEN** `com260-gmac-phy.md` SHALL 关联 PHY handle、MDIO 地址、compatible、RGMII mode、reset GPIO/delay 和 phase 字段

#### Scenario: PHY ID 不能证明完整器件行为

- **WHEN** 当前证据只有 PHY ID compatible 而没有 datasheet 或板级器件证据
- **THEN** 系统 MUST NOT 声明完整型号、扩展寄存器、内部 delay、strap 或 EEPROM 行为

#### Scenario: 静态 DTS 不证明运行时链路

- **WHEN** 文档记录 link、自协商、重协商或 cable unplug/replug 行为
- **THEN** 系统 SHALL 将静态 DTS、固定 revision 第三方运行结果和本项目未验证边界分开表达

### Requirement: DMA ring IRQ 与 ownership 可诊断

系统 SHALL 为 DWMAC5 MAC、MTL 和 DMA 层建立 descriptor/data buffer ownership、doorbell、完成观察、IRQ 和回收状态链，并区分正常路径与设备专有错误恢复。

#### Scenario: TX RX 所有权完成闭环

- **WHEN** CPU 准备并提交 TX 或 RX descriptor 和 buffer
- **THEN** `k3-gmac-dma-irq.md` SHALL 描述 cache clean、OWN 交接、tail doorbell、DMA in-flight、IRQ或轮询、invalidate、完成检查和 CPU 回收顺序

#### Scenario: DMA 或链路错误

- **WHEN** 出现 descriptor error、buffer unavailable、fatal bus error、reset 或 link-down
- **THEN** 系统 SHALL 分别记录已证行为和未知恢复语义，不得用统一 OWN 操作替代设备专有恢复条件

#### Scenario: IRQ 锁竞争

- **WHEN** 固定 revision 第三方 IRQ handler 因 `try_lock` 失败而返回空事件
- **THEN** 系统 SHALL 记录潜在推进风险和解除条件，不得把它写成已验证丢中断，也不得宣称等待下一 IRQ 必然安全

### Requirement: 网络主题导航与既有基线一致

系统 SHALL 把两篇网络主题文档接入总索引，并保持术语、来源覆盖、缺口和相邻主题职责的唯一权威位置。

#### Scenario: 新主题可从总索引访问

- **WHEN** 两篇网络主题文档完成
- **THEN** `docs/index.md` SHALL 提供有效相对链接，GMAC、PHY、MDIO、RGMII、DMA 和 IRQ 术语 SHALL 与既有文档一致，相关 G3、G5、G7 或新缺口不得重复登记

#### Scenario: 相邻主题保持原职责

- **WHEN** 网络主题需要引用 boot、platform、interrupts、DMA 或 reference 文档
- **THEN** 系统 SHALL 只修改建立网络入口所必需的交叉引用、计数和状态，不得改写相邻主题的既有事实或证据等级

#### Scenario: 邻接主题不扩展

- **WHEN** 来源材料涉及 EtherCAT、TSN、网络栈或异步 NIC
- **THEN** 系统 SHALL 只记录与 GMAC 物理通路或后续依赖有关的边界，不得展开这些主题的正文或实现
