> 来源: https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs/k3_ds.md, https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/06-I2C.md, https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/10-USB（源端修订: unknown；观察日期: 2026-09-02）；supporting: https://github.com/spacemit-com/docs-chip/blob/main/zh/key_stone/k3/k3_docs/k3_ds.md（源端修订: 2026-08-25 V1.8；观察日期: 2026-09-07），https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-rdomain.dtsi, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260.dtsi, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260.dts, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260_kit_v02.dts（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-11）

# K3 I2C 与 USB 控制器和板级链路

本文按控制器、协议对象、DTS 变体和板级连接整理 K3 I2C 与 USB。平台 provider 见[平台控制资源](../platform/k3-platform-control.md)，中断拓扑见[中断与时间](../interrupts/k3-interrupt-and-time.md)，DMA 所有权边界见[DMA 与内存所有权](../dma/k3-dma-and-memory-ownership.md)，CoM260 DTS 候选范围见[镜像与 DTS](../boot/com260-image-and-dts.md)。

## 1. 范围与证据

- **官方事实**：K3 datasheet 描述控制器能力。社区 I2C 页面和 USB 目录正文尚未直接取得，不能由标题补写驱动操作。
- **交叉验证**：SpacemiT Linux 6.18 `k3-br-v1.0.y` 的 `k3.dtsi`、`k3-rdomain.dtsi` 和 CoM260 DTS 提供静态节点、资源与变体。
- **推论**：只在多个已列事实共同支持时使用，不替代缺失的板级或运行证据。
- **未知项**：本项目没有执行 I2C 扫描、USB 枚举、角色切换、DMA、IRQ、suspend 或错误注入。

SoC 能力、DTS 节点、模组引出、Kit 连接器与运行结果属于不同层。任一层存在都不自动证明下一层成立。

## 2. I2C 能力与数量冲突

| 证据 | 数量或能力 | 等级 | 边界 |
|---|---|---|---|
| 仓库既有中文 K3 datasheet 聚合 | 9 × I²C | 交叉验证 | 见 [SoC 概述](../platform/k3-soc-overview.md) §6；本 change 不改写该基线 |
| 当前官方英文 K3 datasheet | 最多 10 个独立 I²C 接口 | 交叉验证 | 英文页与既有中文聚合存在数量差异，不能静默选值 |
| 官方 AP DTS aliases | `i2c0`…`i2c6`、`i2c8`，共 8 个 alias | 交叉验证 | alias 集合不等于完整硬件数量；编号中缺 `i2c7` |
| 官方 RCPU DTS | `r_i2c0`、`r_i2c1` | 交叉验证 | 属于 RCPU 域，不与 AP alias 自动合并为产品数量 |

已观察的 AP 与 RCPU 节点都使用 `spacemit,k1-i2c` compatible。AP `i2c0/1/2` 的静态字段示例为 0x38 大小 MMIO、APLIC level-high IRQ、`func`/`bus` clocks、reset、400 kHz `clock-frequency` 和 `disabled` 初态；RCPU `r_i2c0/1` 使用各自 RCPU clock/reset provider，同样默认为 disabled。该复用字符串只说明软件绑定，不能证明 K1 与 K3 的硬件边界完全相同。

## 3. I2C controller 与 client

I2C controller 提供总线时序和传输；client 是挂在该总线地址空间中的设备。CoM260 已观察关系如下。

| Controller | Client 或用途 | DTS 范围 | 等级 | 不能推出 |
|---|---|---|---|---|
| `i2c0` | Type-C port controller `tcpc@25`；基础款含 FUSB301 endpoint | `k3_com260.dtsi` + 顶层变体 | 交叉验证 | 不能证明角色切换或方向检测已在真板成功 |
| `i2c1` | 未在共享 base 中列出 client | `k3_com260.dtsi` 启用 | 交叉验证 | 不能据 enabled 状态推定外接器件 |
| `i2c2` | 只读 `atmel,24c02` EEPROM，地址 `0x50`，ONIE TLV `product-name` | `k3_com260.dtsi` | 交叉验证 | 本项目未读取 EEPROM 实际内容 |
| `i2c3` | Raspberry Pi 7 英寸 panel `0x45`、FT5426 touch `0x38` | `k3_com260.dtsi` | 交叉验证 | 不能证明显示或触控运行成功 |
| `i2c5` | 基础款摄像头节点；Kit V02 将该 controller disabled | `k3_com260.dts` / `k3_com260_kit_v02.dts` | 交叉验证 | 两个顶层 DTS 不能互相代表 |

[CoM260 板级资源](../platform/com260-board-resources.md)还记录 40 Pin 接口、MIPI DSI/CTP 与摄像头连接中的 I2C 信号。连接器声明不提供唯一 controller、引脚复用或已启用设备；这些字段必须由目标 DTS 或原理图解除。

## 4. USB 控制器对象

| 对象 | K3 能力或职责 | 等级 | 边界 |
|---|---|---|---|
| USB 2.0 Host | Host-only，支持 High/Full/Low Speed；使用 UTMI+ PHY | 交叉验证 | datasheet 的 controller 描述不证明 CoM260 对应端口已枚举设备 |
| USB3 Port A | USB 3.0 + USB 2.0 DRD，可作 Host 或 Device | 交叉验证 | DRD 能力不等于当前 role；role 由 DTS、Type-C 状态和软件共同决定 |
| USB3 Port B/C/D | 三个 USB 3.0 Host | 交叉验证 | SuperSpeed PHY 与 PCIe 共享，不能同时使用两种功能 |
| USB controller DMA | controller 内建 DMA | 交叉验证 | 本项目没有 descriptor、cache、完成 IRQ 或回收路径的 K3 运行证据 |
| USB 中断与 suspend | datasheet 声明具备中断和 suspend 能力 | 交叉验证 | 未观察 Linux handler、wakeup 或恢复结果 |

USB Host/DRD 是 controller 角色，USB2/USB3 PHY 承担物理层，Type-C controller 提供连接方向和角色信号，role switch 连接软件角色，Hub 扩展下游端口。它们不是同一个对象，不能用一个节点的 `status = "okay"` 代替整条链路的运行结论。

## 5. CoM260 USB 静态链

### 5.1 共享 base

`k3_com260.dtsi` 启用 Port A 的 USB2/USB3 PHY，并把 USB3 PHY orientation endpoint 连到 FUSB301 switch endpoint；`usb3_porta` 配置 `dr_mode = "otg"`、`usb-role-switch`、默认 peripheral role 和 `monitor-vbus`。Port B 的 USB2/USB3 PHY 与 controller 也标为 enabled。

### 5.2 基础款与 Kit V02

| 维度 | `k3_com260.dts` 基础款 | `k3_com260_kit_v02.dts` | 等级 |
|---|---|---|---|
| Port A VBUS | 继承 `monitor-vbus` | 删除 `monitor-vbus` | 交叉验证 |
| Type-C client | `i2c0/tcpc@25` 含 FUSB301 子节点和 endpoint | `tcpc@25` 保留 wakeup-source，但节点结构不同 | 交叉验证 |
| Port B Hub | VL817 USB2 Hub `@1` + USB3 Hub `@2` | 同样列出两个 VL817 Hub | 交叉验证 |
| 摄像头 I2C | `i2c5` 含多个 sensor | `i2c5` disabled | 交叉验证 |

这些差异说明顶层变体会改变 USB/I2C 链路，不能在 G7 解除前选择一个 DTS 作为所有 CoM260 Kit 的默认配置。

## 6. Kit 物理接口与启动用途

[CoM260 板级资源](../platform/com260-board-resources.md)记录四个 USB 3.0/2.0 Type-A 端口和一个 USB 3.0 Type-C OTG 端口；Type-C 标注“仅下载，不供电”。[启动链](../boot/com260-boot-chain.md)另行负责 Boot ROM 经 USB 下载到 U-Boot/Fastboot 的阶段关系。

Boot ROM 下载、Linux device/gadget、Linux Host 和 DRD role switch 是不同软件阶段。Type-C 可用于下载不能证明启动后的 gadget 配置，也不能证明该口可向板卡供电或向外设供电。

## 7. 数据、IRQ、错误与生命周期边界

当前证据允许确认 controller 具备 MMIO、clock/reset、IRQ 或内建 DMA 能力，但没有直接观察下列运行路径：

- I2C START/address/data/STOP、arbitration lost、NACK、timeout、bus recovery 和取消后的 controller 状态；
- USB transfer ring、endpoint/slot 状态、DMA/cache 所有权、IRQ completion、disconnect、role change、suspend/resume 和 reset recovery；
- FUSB301、VL817、EEPROM、panel、touch 或 camera 在目标 Kit 上的实际 probe、地址和运行日志。

因此正文不提供未观察的操作命令，也不把通用 Linux I2C、xHCI 或 gadget 行为写成 K3 专属实现。

## 8. 未知项

### U1：I2C 控制器总数与编号

- 当前证据：既有中文聚合为 9 路；当前英文 datasheet 为最多 10 路；AP aliases 可见 8 路，RCPU DTS 另有 2 路。
- 禁止推断：不得直接相加得出产品总数，也不得以较新英文页面覆盖中文基线。
- 解除条件：取得同一修订、同一语言范围下的 K3 programmer reference 和完整 controller instance 表，或由独立 refresh change 裁决来源变化。
- 影响主题：I2C 实例矩阵、pinctrl/clock/reset、IRQ 与后续驱动规划。

### U2：I2C client 的板级唯一映射

- 当前证据：CoM260 共享 base 和多个顶层 DTS 给出部分 client，但 G7 尚未唯一映射默认 Kit DTS，原理图未直接解析。
- 禁止推断：不得把基础款摄像头、Kit V02 disabled 状态或 40 Pin 信号外推到所有 CoM260 产品。
- 解除条件：确认目标板型号与顶层 DTB，或取得并解析对应原理图及运行时 device tree。
- 影响主题：Type-C、EEPROM、显示触控、摄像头和扩展接口。

### U3：USB Port B/C/D 与 PCIe PHY 分配

- 当前证据：datasheet 声明三路 Host 的 SuperSpeed PHY 与 PCIe 共享；CoM260 DTS 已列 Port A/B 和 PCIe PHY 配置，但没有目标产品的完整 mux 决策表。
- 禁止推断：不得同时宣称共享 PHY 上的 USB 与 PCIe 功能可用，也不得根据 SoC 路数推定 Kit 全部引出。
- 解除条件：取得目标 DTS 的 PHY/mux 完整配置、板级原理图及启动日志中的 link/port 枚举结果。
- 影响主题：本文件 USB 路径和后续 PCIe 正文。

### U4：USB 运行与恢复路径

- 当前证据：官方能力包含 DMA、中断和 suspend；CoM260 DTS 提供静态 role/PHY/Hub 节点；本项目没有 USB 子树正文或真板日志。
- 禁止推断：不得声明 xHCI、gadget、role switch、Hub reset、热插拔、suspend/resume 或 disconnect recovery 已验证。
- 解除条件：直接读取 K3 USB 子树及对应驱动，并在明确板型上采集枚举、角色切换、IRQ 和恢复证据。
- 影响主题：USB Host/device 使用、BootROM 下载后的 OS 路径、DMA/cache 和电源管理。

### U5：Buildroot I2C/USB 页面内容

- 当前证据：两个官网入口已登记；I2C 正文和 USB 子树在 2026-09-11 未能直接取得，访问失败不是页面不存在。
- 禁止推断：不得依据标题或通用 Linux 文档补写 K3 命令、driver path、测试步骤和错误语义。
- 解除条件：直接打开对应官网正文或 SpacemiT 官方 GitHub 等价页，并核对源端修订和适用板型。
- 影响主题：本文件的软件行为、验证入口和来源等级。

## 9. 主题边界

- PCIe controller、lane/PHY、RC/EP 和插槽映射由后续 `k3-pcie-and-can.md` 承担；本文只记录 USB/PCIe PHY 互斥。
- EtherCAT 与 GMAC1 的关系由后续 `k3-ethercat.md` 承担。
- 已知缺口、术语和总索引在 MS10 最终 Iteration 统一收敛，本轮不提前修改。
