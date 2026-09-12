> 来源: https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs/k3_ds.md, https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/11-PCIe.md, https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/15-CAN.md, https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/eco/k3_com260/com260_user_guide.md（源端修订: unknown；观察日期: 2026-09-02/2026-09-07）；supporting: https://github.com/spacemit-com/docs-chip/blob/main/zh/key_stone/k3/k3_docs/k3_ds.md, https://github.com/spacemit-com/docs-product/blob/main/zh/k3_com260/com260_user_guide.md, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-rdomain.dtsi, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260.dtsi, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260.dts, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260_kit_v02.dts（源端修订: datasheet 2026-08-25 V1.8；user guide 2026-03-19 V2.0；DTS branch k3-br-v1.0.y；观察日期: 2026-09-07/2026-09-08）

# K3 PCIe 与 CAN 静态拓扑和运行边界

本文区分 SoC 能力、controller 资源、DTS 状态、模组/Kit 连接和运行结果。SoC 能力见[平台概述](../platform/k3-soc-overview.md)，CoM260 连接器见[板级资源](../platform/com260-board-resources.md)，DTS 候选范围见[镜像与 DTS](../boot/com260-image-and-dts.md)，中断 provider 见[中断与时间](../interrupts/k3-interrupt-and-time.md)。

## 1. 范围与证据

- **官方事实**：K3 datasheet 和 CoM260 官网入口承担能力与产品范围；PCIe/CAN Buildroot 正文尚未直接取得。
- **交叉验证**：SpacemiT GitHub datasheet、user guide 和 Linux `k3-br-v1.0.y` DTS 提供静态能力、节点与变体。
- **推论**：只说明多个已观察事实之间的候选关系，不代替原理图、目标 DTB 或运行证据。
- **未知项**：本项目没有执行 PCIe link training、枚举、NVMe、MSI/MSI-X、热插拔或 CAN 收发与错误注入。

静态 `status = "okay"` 只表示该 DTS 请求启用节点。它不证明时钟、复位、PHY、外设、电气链路、驱动 probe 或数据传输已经成功。

## 2. PCIe SoC 能力与对象边界

| 对象 | 已观察事实 | 等级 | 不能推出 |
|---|---|---|---|
| PCIe controller | K3 datasheet 声明 PCIe Gen3，合计 8 lanes | 交叉验证 | 不能推出 CoM260 的 lane 分配或所有 controller 均启用 |
| RC/EP | SoC 声明支持 Root Complex 与 Endpoint 模式 | 交叉验证 | 不能推出任一具体端口的当前模式 |
| 热插拔 | SoC 能力表声明支持 | 交叉验证 | 不能推出插槽供电、presence detect、事件 IRQ 或 OS recovery 已配置 |
| PHY/lane | CoM260 共享 DTS 为三个 RC 节点选择 PHY | 交叉验证 | PHY phandle 不等于链路完成训练 |
| 插槽 | Kit 资料列两个 M-Key NVMe 与一个 E-Key Wi-Fi/BT 插槽 | 交叉验证 | 用途标签不证明设备已安装、枚举或绑定驱动 |

controller 决定 RC/EP 事务，lane/PHY 承担电气链路，PERST# 复位端点，CLKREQ# 参与参考时钟请求，M.2 插槽提供板级连接。任何单个对象存在都不能代替完整链路。

## 3. CoM260 PCIe 静态链

| DTS 对象 | 静态配置 | 等级 | 运行边界 |
|---|---|---|---|
| `pcie0_rc` | `num-lanes = <4>`；`phys` 引用 PHY0 与 PHY1 | 交叉验证 | 只确认 x4 组合，不确认协商宽度、速率或端点 |
| `pcie3_rc` | `phys` 引用 PHY4 | 交叉验证 | 只确认单 PHY 引用，不确认对应哪个 M.2 插槽 |
| `pcie4_rc` | `phys` 引用 PHY5 | 交叉验证 | 只确认单 PHY 引用，不确认对应哪个 M.2 插槽 |

上述节点位于 CoM260 共享 base，因此基础款 `k3_com260.dts` 与 Kit V02 `k3_com260_kit_v02.dts` 都会继承这些静态声明。继承关系不构成默认 DTB 选择，也不证明两种板型具有相同布线、PERST#、CLKREQ# 或插槽供电。

Kit 资料列出一个 M.2 2280 M-Key、一个 M.2 2230 M-Key，预期用于 NVMe SSD；另有一个 M.2 2230 E-Key，预期用于 Wi-Fi 6/BT 5.0。当前证据没有 controller/PHY 到三个插槽的唯一映射，不能按表格顺序配对。

## 4. USB/PCIe 共享 PHY

[I2C 与 USB](k3-i2c-and-usb.md)记录 K3 USB Port B/C/D 的 SuperSpeed PHY 与 PCIe 共享，且同一 PHY 同一时刻只能选择一种功能。CoM260 DTS 中存在 PCIe PHY 引用，同时也存在 USB Port B 静态链；这两类节点不能分别视为可同时工作的证明。

解除具体互斥需要目标 DTB 的完整 PHY/mux 配置、板级原理图，以及启动日志中的 PCIe link 和 USB port 枚举结果。本文不根据 SoC 总 lane 数补齐未观察的 mux 分配。

## 5. CAN controller、协议与物理层

| 层 | 已观察事实 | 等级 | 边界 |
|---|---|---|---|
| SoC 能力 | K3 datasheet 汇总为 10 路 CAN | 交叉验证 | 不等于 CoM260 全部引出或启用 |
| AP controller | K3 AP DTS 提供 `flexcan` controller；顶层 DTS 按板型选择实例 | 交叉验证 | controller 节点不能证明 CAN/CAN-FD 帧已收发 |
| RP controller | `k3-rdomain.dtsi` 提供 `r_flexcan0..4`，中断父为 SAPLIC | 交叉验证 | RP 节点不与 AP 实例自动合并计数 |
| CAN/CAN-FD 协议 | Kit 接口称为 CAN FD；DTS 节点名包含 FlexCAN | 交叉验证 | 当前材料不足以确认每个 controller 的 CAN-FD 位时序、payload 或驱动模式 |
| 物理层 | Kit 资料声明 4 Pin 连接器和板载 CAN 收发器 | 交叉验证 | 收发器存在不证明 controller、pinctrl 或终端电阻配置正确 |

## 6. AP/RP 实例与 CoM260 变体

| 变体或域 | 静态实例 | pinctrl/clock/IRQ | 等级 |
|---|---|---|---|
| CoM260 基础款 | `flexcan2` enabled | 80 MHz；`can2_1_cfg`；AP IRQ 具体值不在既有摘要中 | 交叉验证 / 未知项（IRQ） |
| CoM260 Kit V02 | AP `flexcan0..4` enabled | 各节点记录 80 MHz；具体 pinctrl/IRQ 需逐实例核对 | 交叉验证 / 未知项（完整资源） |
| CoM260 Kit V02 RP | `r_flexcan2` enabled | RP FlexCAN 由 RCPU sysctrl/MPMU 提供 clock/reset，IRQ 接 SAPLIC | 交叉验证 |
| RP 域基线 | `r_flexcan0..4` 静态节点 | SAPLIC source 241、243、245、247、249；节点默认状态不能由 Kit V02 之外的顶层选择替代 | 交叉验证 |

基础款启用一个 AP 实例，Kit V02 启用五个 AP 实例和一个 RP 实例。两者是不同顶层 DTS 的静态选择，不能把 Kit V02 的实例集合外推到基础款，也不能据此选择目标 Kit 的默认 DTB。

## 7. CAN 板级冲突 C2

CoM260 user guide 的三个位置必须并列保留：§3.3.2 列出一个集成 CAN 收发器的 4 Pin 排针；§5.14 描述板载 CAN 收发器可连接 CAN 设备；§3.3.3 的功能接口表却写 `CAN=否`。前两项证明物理描述存在，后一项否定当前功能接口可用性；它们不能互相覆盖。

该冲突不能被解释成“硬件不存在”或“只是 OS 未启用”。解除条件是取得适用同一产品修订的原理图、BOM 和说明，确认目标 DTB 与 pinctrl/clock/driver 状态，并观察接口电气和 CAN 收发结果。

## 8. 运行、错误与生命周期边界

当前没有观察下列路径：

- PCIe PERST# 时序、CLKREQ#、reference clock、LTSSM、协商速率/宽度、配置空间、BAR、MSI/MSI-X、IOMMU、NVMe、热插拔和错误恢复；
- CAN nominal/data bitrate、CAN-FD 模式、收发器 standby、终端电阻、IRQ 收发、bus-off、error-passive、重启、suspend/resume 和 AP/RP 所有权切换。

因此本文不提供未观察的 Buildroot 操作命令，也不把通用 DesignWare PCIe、FlexCAN 或 SocketCAN 行为写成 K3 专属实现。

## 9. 未知项

### U1：PCIe controller、PHY 与 M.2 插槽映射

- 当前证据：共享 DTS 给出三个 RC 节点及 PHY0/1/4/5 引用，Kit 资料给出三个 M.2 插槽，但没有两者的唯一映射。
- 禁止推断：不得按节点编号、PHY 编号或插槽列出顺序配对。
- 解除条件：取得适用目标板修订的原理图和完整目标 DTB，核对 lane mux、插槽信号与供电。
- 影响主题：PCIe 拓扑、NVMe/Wi-Fi 用途、USB/PCIe PHY 分配和 bring-up。

### U2：PCIe 侧带信号与运行状态

- 当前证据：SoC 声明 RC/EP 和热插拔能力；当前摘要没有 PERST#、CLKREQ#、presence、link、MSI/MSI-X 或热插拔日志。
- 禁止推断：不得由能力表或 RC 节点声明端点已复位、时钟已稳定、链路已枚举或热插拔可用。
- 解除条件：核对原理图和目标 DTS 侧带 GPIO/clock 配置，并采集 link、枚举、中断及插拔恢复日志。
- 影响主题：RC/EP 模式、M.2 设备、NVMe、错误恢复和电源管理。

### U3：CAN 实例资源和 CAN-FD 能力

- 当前证据：基础款与 Kit V02 给出部分实例、80 MHz、pinctrl 或 SAPLIC IRQ；Kit 接口名称为 CAN FD。
- 禁止推断：不得补齐未观察的 AP IRQ、pinctrl、clock/reset，也不得把接口名称外推为所有 controller 的 CAN-FD 运行能力。
- 解除条件：直接核对 K3 AP/RP FlexCAN 全部节点、binding 和驱动，并在目标 DTB 上确认 nominal/data bitrate 与 probe 状态。
- 影响主题：AP/RP 实例矩阵、IRQ、clock/reset、pinctrl 和 CAN-FD 配置。

### U4：CAN 连接器可用性冲突

- 当前证据：同一 user guide 同时记录 CAN 连接器/收发器存在和功能接口 `CAN=否`。
- 禁止推断：不得选择任一表述覆盖另一表述，也不得把冲突单独归因为软件或硬件。
- 解除条件：取得同一产品修订的原理图、BOM、勘误或新版 user guide，并在明确 DTB 上验证引脚和收发。
- 影响主题：Kit 接口、收发器、终端电阻、目标 DTS 和真板测试。

### U5：PCIe/CAN Buildroot 页面内容

- 当前证据：两个官网入口已登记，但正文在 2026-09-11 未直接取得；访问失败不表示页面不存在。
- 禁止推断：不得依据标题或通用 Linux 文档补写 K3 命令、driver path、测试步骤和恢复语义。
- 解除条件：直接打开官网正文或 SpacemiT 官方 GitHub 等价页，核对源端修订、SDK 和适用板型。
- 影响主题：PCIe/CAN 软件配置、验证入口、错误处理和来源等级。

## 10. 主题边界

- USB controller、DRD、role switch、Hub 和共享 PHY 的 USB 侧定义见[I2C 与 USB](k3-i2c-and-usb.md)。
- 中断控制器和 AP/RP delivery 由[中断与时间](../interrupts/k3-interrupt-and-time.md)承担；本文只记录 controller 的 IRQ 依赖。
- 最终术语、已知缺口和总索引在后续 Iteration 统一收敛，本轮不提前修改。
