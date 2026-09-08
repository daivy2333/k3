# k3-com260-platform-uart-baseline Specification

## Purpose
TBD - created by archiving change establish-k3-com260-platform-uart-baseline. Update Purpose after archive.
## Requirements
### Requirement: 平台控制事实按资源域和证据层分离

平台文档 SHALL 分别记录 K3 AP 与 RCPU 域中的 pinctrl、clock、reset、APBC/CCU 和 UART 初始化依赖，并 MUST 为每项事实标明来源、版本或分支、适用板型、资源域和证据等级。

#### Scenario: AP 域控制链有直接证据

- **WHEN** 官网资料、官方 DTS、binding 或 driver 可共同定位 AP UART 的 pinctrl、clock、reset 和控制器节点
- **THEN** `k3-platform-control.md` 按信号或节点关系记录控制链，并区分 `官方事实`、`交叉验证` 和 `官方源码行为`

#### Scenario: RCPU 域使用独立资源

- **WHEN** RCPU UART 使用与 AP UART 不同的 MMIO、clock/reset、pinctrl、compatible 或驱动策略
- **THEN** 文档将其归入 RCPU 域，不借用同编号 AP UART 的地址、时钟、复位或初始化结论

#### Scenario: 控制链只有第三方实现可见

- **WHEN** 固定 clock/divisor、CCU 写序列或 pinmux 自修复只在 Rt-Async-AMP/tgoskits 中出现
- **THEN** 文档把它标为 `第三方经验` 或待验证假设，不写成 K3 硬件约束

#### Scenario: 资源依赖缺少直接来源

- **WHEN** 某 UART 实例的 clock parent、reset side effect、pin 电气属性或初始化先后无法由当前来源证明
- **THEN** 文档记录未知项、禁止推断、解除条件和受影响实例，不用相邻 UART 或 K1 默认值补齐

### Requirement: UART 实例和编程属性可追溯

串口文档 SHALL 聚合 K3 的 AP/RCPU UART 实例及其可证 MMIO、IRQ、MMIO width/stride、FIFO、threshold、clock/reset、pinctrl、compatible 和状态字段，并 SHALL 不把控制器能力等同于 CoM260 Kit 已连接或启用。

#### Scenario: UART 实例字段可由官方 DTS 定位

- **WHEN** `k3.dtsi`、RCPU DTSI 或目标板 include 链给出 UART 节点的地址、IRQ、clock/reset、stride、width、FIFO 或 threshold
- **THEN** `com260-uart.md` 按实例和资源域记录原始字段、解析结果、来源分支及板级启用状态

#### Scenario: SoC 数量与实例集合分层

- **WHEN** datasheet 证明 UART 总数或 AP/RCPU 数量，而 DTS 提供具体节点集合
- **THEN** 文档分别记录硬件能力和官方源码中的实例清单，不由任一层单独推定 CoM260 Kit 可用端口数

#### Scenario: compatible 名称包含其他代际

- **WHEN** K3 UART 节点使用 `spacemit,k1-uart`、`intel,xscale-uart` 或其他历史兼容字符串
- **THEN** 文档记录字符串、匹配顺序和实际驱动路径，不按名称把该节点归类为 K1 硬件或推导完整寄存器语义

#### Scenario: FIFO 深度来源冲突

- **WHEN** 官方 K3 DTS/SDK 与 tgoskits 等来源分别给出 256 和 64 等不同 FIFO 深度
- **THEN** 文档并列记录数值、来源、版本、资源域和适用实现；在缺少解除证据时不选择统一硬件值

#### Scenario: 目标 Kit DTS 未唯一映射

- **WHEN** CoM260 Kit 用户指南版本与现有 DTS 变体名称仍不能唯一对应
- **THEN** 共享 DTSI 字段只记为共同交叉验证，变体专有 UART/pinctrl 状态不得写成目标 Kit 默认配置

### Requirement: CoM260 UART0 与 console 路径区分静态和运行时结论

串口文档 SHALL 连接 CoM260 Kit 的 UART0 引脚和串口参数、K3 aliases、`chosen.stdout-path`、bootargs、early console 与 Linux tty 路径，并 MUST 分开记录静态 DTS 解析和运行时实际配置。

#### Scenario: serial0 静态映射可闭合

- **WHEN** 直接读取的 `k3.dtsi` aliases 明确将 `serial0` 指向 `uart0`，共享 CoM260 DTSI 又使用 `serial0:115200`
- **THEN** 文档记录 `serial0` 到具体 UART 节点的静态解析链，并修正 MS03 中相应的未解析表述

#### Scenario: 板级 UART0 物理接口有证据

- **WHEN** CoM260 user guide 直接给出 UART0 RX/TX 针脚修订和 115200-8-N-1 设置
- **THEN** 文档将物理接口与控制器节点关联为带版本的 `交叉验证`，同时保留流控和旧板修订的未知边界

#### Scenario: 运行时 console 可能被覆盖

- **WHEN** 只有 DTS `chosen` 和 bootargs，缺少 bootloader 环境、镜像内容、启动日志或 `/proc/cmdline`
- **THEN** 文档不得声称 Kit 最终 console 已确认，并列出 bootloader 覆盖的解除条件

#### Scenario: earlycon 与 tty 名称来自不同阶段

- **WHEN** 固件输出、SBI early console、Linux earlycon 和 `ttyS0` 线索同时存在
- **THEN** 文档按启动阶段分别记录来源和可证关系，不把相同波特率或字符输出视为同一驱动路径的证明

### Requirement: PXA UART 工程经验保留适用边界

串口文档 SHALL 提取 Rt-Async-AMP/tgoskits 中与 K3 UART bring-up 和后续驱动规划有关的 UUE/OUT2、THRE/TEMT、FIFO、IRQ budget、divisor errata 和初始化顺序，并 MUST 保留仓库、commit、资源域和实现语境。

#### Scenario: 寄存器位用于初始化经验

- **WHEN** 第三方代码通过 UUE、OUT2、IER、FCR 或 LCR 组合完成初始化或中断启用
- **THEN** 文档记录实际写序列和实现目的，标为 `第三方经验`，不宣称该顺序对所有 K3 UART 实例都必要

#### Scenario: THRE 与 TEMT 用于不同完成条件

- **WHEN** 第三方实现分别使用 THRE 判断可继续写入、使用 TEMT 或短写处理 drain/完成条件
- **THEN** 文档区分寄存器观察和软件语义，并把 flush/tcdrain API 设计留在本 change 范围之外

#### Scenario: IRQ handler 具有处理预算

- **WHEN** 第三方实现限制单次 IRQ 的循环次数或采样量，并对 THRE 执行 mask/rearm
- **THEN** 文档记录预算、触发条件和防止活锁的意图，不把该数值写成硬件 FIFO 或 IRQ 规范

#### Scenario: divisor errata 依赖特定实现

- **WHEN** 第三方代码对 divisor、DLAB 或相邻寄存器访问采用 workaround
- **THEN** 文档记录触发上下文和证据缺口；未由 K3 官方资料确认前不得推广到所有实例

### Requirement: 来源覆盖、未知项和导航与主题正文一致

本 change SHALL 在主题正文引用新 URL 前登记唯一覆盖行，SHALL 使交付文档符合仓库模板并从总入口可达，且 SHALL 不创建与现有 G1-G7 语义重复的缺口。

#### Scenario: 需要精确官方仓库 URL

- **WHEN** 主题正文将引用 docs-buildroot UART/PINCTRL/Clock/Reset 页面、raw DTS/DTSI、binding 或 driver 文件
- **THEN** `source-coverage.md` 先登记完整 URL、类型、范围、状态、观察日期和限制，再由正文引用

#### Scenario: 官网仍为 SPA 壳

- **WHEN** 权威官网页面只能读取 SPA 外壳而官方 GitHub 对应内容可读
- **THEN** 官网覆盖行保持 `partially-observed` 和未知修订，GitHub 行独立标为 supporting，不用 GitHub 日期代填官网日期

#### Scenario: 两篇主题文档交付

- **WHEN** `k3-platform-control.md` 与 `com260-uart.md` 完成
- **THEN** 每篇首行来源合规、事实具有证据等级和适用域、未知项具有解除条件，并从 `docs/index.md` 使用相对链接可达

#### Scenario: 文档接近建议行数上限

- **WHEN** 任一主题文档接近 450 行或预计超过 500 行建议上限
- **THEN** 按 AP/RCPU 或平台控制子主题拆分，保留导航、独立来源首行和全部验收字段，不以删除事实类别规避拆分

#### Scenario: 未知项已由现有缺口覆盖

- **WHEN** 新发现的 programmer reference、目标 DTS、console 运行时配置或寄存器语义缺口与 G1-G7 相同
- **THEN** 正文引用或更新既有条目；只有语义独立且影响明确时才新增缺口

#### Scenario: 来源在调查期间变化

- **WHEN** 已读取页面、分支或第三方 commit 在 Gate 2 调查期间发生变化
- **THEN** 停止沿用受影响结论，记录新身份并重新执行相应来源核对；不得覆盖用户已有工作区修改
