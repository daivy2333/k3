> 来源: https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/image.md（源端修订: unknown；观察日期: 2026-09-07）；https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/boot.md（源端修订: unknown；观察日期: 2026-09-07）；https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/boot.md（源端修订: unknown；观察日期: 2026-09-07）；https://github.com/spacemit-com/linux-6.18/tree/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-07）；https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260.dts（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-07）；https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260.dtsi（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-07）；https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260_kit_v02.dts（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-07）；https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-07）；https://github.com/spacemit-com/docs-product/blob/main/zh/k3_com260/com260_user_guide.md（源端修订: V2.0/2026-03-19；观察日期: 2026-09-07）；https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/eco/k3_com260/com260_user_guide.md（源端修订: unknown；观察日期: 2026-09-07；partially-observed: SPA 壳正文未直接取得）

# K3 / CoM260 镜像与目标 DTS

> 文档定位: 整理 K3 CoM260 Kit 的可证镜像类型、写入方式以及目标 DTS 候选集合; 不选定未被直接证据唯一映射的 DTS, 不推定地址、GMAC ring 布局或 PHY 寄存器。
> 证据等级: 仅使用 `官方事实` / `交叉验证` / `推论` / `未知项` 四级枚举; 仅当 `k3_com260.dts`、`k3_com260_kit_v02.dts`、`k3_com260.dtsi` 三个 CoM260 命名文件直接读到时, 才记录其 `model` / `compatible` / `include` / `chosen` / `&resmem` 字段; `k3.dtsi`(K3 SoC 顶层 DTSI)同样直接打开, 但仅用于根 `#address-cells` / `#size-cells` / `memory@` / `reserved-memory` 等 SoC 级别事实, 不写 CoM260 板级字段; 其他 CoM260 命名候选仅以候选身份登记, 不写具体字段。
> 边界: 本文档只覆盖镜像与 DTS 候选; 启动阶段与介质见 [`com260-boot-chain.md`](com260-boot-chain.md); 板级连接器与 UART 物理参数见 [`../platform/com260-board-resources.md`](../platform/com260-board-resources.md); SoC 能力概述见 [`../platform/k3-soc-overview.md`](../platform/k3-soc-overview.md)。

## 目录

1. 镜像、DTS 与证据边界
2. 镜像类型与写入方式
3. CoM260 DTS 候选集合
4. 已观察字段(model / compatible / include / chosen / &resmem)
5. 产品版本与 DTS 命名映射
6. 未知项闭包
7. 修订快照
8. 边界声明

## 1. 镜像、DTS 与证据边界

- 本文档只整理可由直接来源(官网文档正文、官方 GitHub 仓库 DTS 文件、用户指南目录)证明的镜像类型、写入方式与 DTS 候选; 不展开镜像解包/校验/签名验证, 不展开 buildroot 构建流程, 不展开镜像制品分发的版本号管理。
- DTS 字段按 `NO CHANGE WITHOUT TEST WITNESS` 原则采集: 只有直接打开 `k3_com260.dts`、`k3_com260_kit_v02.dts`、`k3_com260.dtsi` 三个 CoM260 命名文件后才记录其字段; `k3.dtsi` 同样直接打开, 仅用于 SoC 级别 `memory@` / `reserved-memory` / `cpus` / `#address-cells` / `#size-cells` 等根节点事实; 其余 4 个 CoM260 命名候选(`k3_com260_ifx.dts`、`k3_com260_ifx2.dts`、`k3_com260_ifx_tq.dts`、`k3_com260_tq.dts`)只列文件名, 不写字段。
- 唯一映射要求文件名(实际 DTS 路径)、产品版本(`K3-CoM260_P1_LP5315B_32X2_v03_20260312`, 来自 com260_user_guide.md V2.0 资料下载部分)与文档修订三组字段全部对齐; 任一字段不对齐即记 `未唯一映射`, 详见 §5。
- 本文档不下载、解析、登记镜像制品, 不登记 buildroot 配置文件(`defconfig`、`kconfig.fragment`、`board/...`); 也不登记 SpacemiT K3 SDK / SDK2-1.0 等发行版的具体路径, 这些在 MS04-MS07 范围。

## 2. 镜像类型与写入方式

| 维度 | 事实 | 证据等级 | 来源 |
| --- | --- | --- | --- |
| 镜像格式 | K3 Buildroot 默认提供 `zip` 格式镜像 | 交叉验证 | docs-buildroot image.md(直接打开 2026-09-07) |
| 镜像用途 | zip 镜像适用于 Titan Flasher, 也可解压后用 fastboot 刷机 | 交叉验证 | docs-buildroot image.md(直接打开 2026-09-07) |
| 镜像下载入口 | docs-buildroot image.md 给出一个镜像下载页入口; 该入口 URL 在本文档不重复登记, 视为 image.md 的链接目标(详见 docs/reference/source-coverage.md 对应行备注) | 交叉验证 | docs-buildroot image.md(直接打开 2026-09-07) |
| 刷机工具入口 | docs-buildroot image.md 给出一个 Titan Flasher 使用手册入口; 该入口 URL 在本文档不重复登记, 视为 image.md 的链接目标(详见 docs/reference/source-coverage.md 对应行备注) | 交叉验证 | docs-buildroot image.md(直接打开 2026-09-07) |
| 镜像适用对象 | 文档写作背景为 K3 SDK; image.md 未点名 CoM260 Kit 专用镜像 | 交叉验证 | docs-buildroot image.md(直接打开 2026-09-07) |
| 适用 SoC | 文档写作背景为 K3 系列 SOC | 交叉验证 | docs-buildroot boot.md §适用范围(直接打开 2026-09-07) |
| 镜像内部组成 | 文档未直接展开 zip 内含 `bootfs.img` / `rootfs.ext4` / FSBL / ESOS / OpenSBI / U-Boot 的具体大小或校验码; 仅 `bootfs.img` / `rootfs.ext4` 制品名在 docs-buildroot boot.md §eMMC 刷机中作为刷机命令字段出现 | 交叉验证 | docs-buildroot image.md / boot.md §eMMC 刷机(直接打开 2026-09-07) |
| CoM260 Kit 专属镜像 | docs-buildroot image.md / boot.md 未说明是否存在 com260_xxx_*.img, 也未给出命名规则 | 未知项 | docs-buildroot image.md / boot.md(直接打开 2026-09-07) |
| 镜像文件命名映射 | com260_user_guide.md V2.0 资料下载部分列出 `K3-CoM260_P1_LP5315B_32X2_v03_20260312.pdf`; docs-buildroot 未给出该产品版本对应的镜像包名 | 未知项 | com260_user_guide.md V2.0 资料下载(GitHub 对应页, 2026-09-07 直接打开); docs-buildroot image.md(直接打开 2026-09-07) |

注: 本节 §2 严格按 image.md / boot.md / 用户指南的可见身份组织, 不展开 buildroot 配置文件与镜像目录结构; docs-buildroot image.md 给出的两个外部入口(镜像下载页 / Titan Flasher 使用手册)在 coverage 中由 image.md 对应行备注描述, 本文不重复登记它们的裸 URL(避免与"正文引用 URL 先登记"的规则冲突)。

## 3. CoM260 DTS 候选集合

数据来源: linux-6.18 仓库分支 `k3-br-v1.0.y` 的 `arch/riscv/boot/dts/spacemit/` 目录 API 列出 40 条目(39 文件 + 1 子目录 `lcd/`), 详见 `https://github.com/spacemit-com/linux-6.18/tree/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit` 与 source-coverage.md 中对应行。

CoM260 命名集合共 7 个候选, 全部位于 `arch/riscv/boot/dts/spacemit/`, 按目录观察列出; 候选集合 = 6 个顶层 `.dts` + 1 个共享 base `.dtsi`, 其中 `k3_com260.dtsi` 同时是 6 个 `.dts` 共享 base, 仅作 base 身份计入 7 候选, 不重复计入 6 个 `.dts` 候选。本 Cycle 已直接打开 4 个文件 raw URL 记录字段(3 个 CoM260 命名文件 `k3_com260.dts` / `k3_com260_kit_v02.dts` / `k3_com260.dtsi`, 1 个 K3 SoC 顶层 DTSI `k3.dtsi`); 其余 4 个 ifx / tq 命名候选(`k3_com260_ifx.dts`、`k3_com260_ifx2.dts`、`k3_com260_ifx_tq.dts`、`k3_com260_tq.dts`)仅列文件名, 不写字段。

| 候选 DTS 路径 | 已直接打开 | 类型 | 评估 | 证据等级 | 来源 |
| --- | --- | --- | --- | --- | --- |
| `arch/riscv/boot/dts/spacemit/k3_com260.dts` | 是(2026-09-07) | .dts(顶层) | `model = "SpacemiT K3 Com260"`, `compatible = "spacemit,k3-com260"`; include k3_com260.dtsi + k3-camera.dtsi; 3 摄像头 + 1 flexcan2 + 1 GMAC; 是命名集合中的基础款 | 交叉验证 | linux-6.18 `k3-br-v1.0.y` 分支 raw URL(直接打开 2026-09-07) |
| `arch/riscv/boot/dts/spacemit/k3_com260.dtsi` | 是(2026-09-07) | .dtsi(共享 base, 计入 7 候选) | 共享 base; include k3.dtsi / k3-rdomain.dtsi / k3-pinctrl.dtsi / k3_opp_table.dtsi / lcd/lcd_tc358762xbg_dpi_800x480.dtsi / lcd_dsi_panel.dtsi / k3-dp1.dtsi; 含 chosen(`earlycon=sbi console=ttyS0,115200 loglevel=8 random.trust_bootloader=1 unaligned_scalar_speed=fast unaligned_vector_speed=fast`, `stdout-path = "serial0:115200"`)、&resmem(`cmamem: linux,cma { compatible = "shared-dma-pool"; reusable; size = <0 0x20000000>; alloc-ranges = <1 0x40000000 0 0x20000000>; linux,cma-default; }`, 两 cell 编码地址/大小, 详见 §6.6) | 交叉验证 | linux-6.18 `k3-br-v1.0.y` 分支 raw URL(直接打开 2026-09-07) |
| `arch/riscv/boot/dts/spacemit/k3_com260_kit_v02.dts` | 是(2026-09-07) | .dts(顶层) | `model = "SpacemiT K3 Com260 Kit V02"`, `compatible = "spacemit,k3-com260-kit-v02"`; include k3_com260.dtsi(无 camera); 5 flexcan(0-4) + r_flexcan2 + 2 UART(uart4/uart5) + QSPI DMA + GMAC; 与 CoM260 Kit 名称最接近 | 交叉验证 | linux-6.18 `k3-br-v1.0.y` 分支 raw URL(直接打开 2026-09-07) |
| `arch/riscv/boot/dts/spacemit/k3_com260_ifx.dts` | 否 | .dts | 仅目录列名; 与 "Infineon" 命名假设相关但本 Cycle 未直接打开, 字段留 `未知项` | 未知项 | linux-6.18 `k3-br-v1.0.y` 分支目录(直接观察 2026-09-07) |
| `arch/riscv/boot/dts/spacemit/k3_com260_ifx2.dts` | 否 | .dts | 同上 | 未知项 | linux-6.18 `k3-br-v1.0.y` 分支目录(直接观察 2026-09-07) |
| `arch/riscv/boot/dts/spacemit/k3_com260_ifx_tq.dts` | 否 | .dts | 同上 | 未知项 | linux-6.18 `k3-br-v1.0.y` 分支目录(直接观察 2026-09-07) |
| `arch/riscv/boot/dts/spacemit/k3_com260_tq.dts` | 否 | .dts | 仅目录列名; "TQ" 命名假设与用户指南中"天嵌"硬件合作相关, 但本 Cycle 未直接打开, 字段留 `未知项` | 未知项 | linux-6.18 `k3-br-v1.0.y` 分支目录(直接观察 2026-09-07) |

不选原则(本仓库内):

- 不选 `k3-pico.dtsi` / `k3-pico-itx.dts` / `k1-*.dts` / `k3-firefly-aibox-k3.dts` / `k3-firefly-common.dtsi` / `k3_BS01DCMA.dts` / `k3_deb1.dts` / `k3_evb*.dts` / `k3_gemini_*.dts` / `k3_fpga_1x1.dts`, 这些文件命名不属于 K3 CoM260, 沿用 M01 / D01 范围规则。
- 不选 `lcd/lcd_tc358762xbg_dpi_800x480.dtsi` 作为顶层 DTS, 它是 `k3_com260.dtsi` 的 include 片段, 不是目标 Kit 的候选。
- 不在 `unknown` 字段上用相邻板型(K1、Pico、DesignWare)或 buildroot 通用示例值填充。

## 4. 已观察字段(model / compatible / include / chosen / &resmem)

> 字段采集来源直接来自 raw 文本阅读, 文件标识见 §3 表; 任何未直接观察到的字段一律不在本节出现, 转入 §6 未知项。

### 4.1 `k3_com260.dts`(基础款)

| 字段 | 值 | 等级 | 说明 |
| --- | --- | --- | --- |
| model | `SpacemiT K3 Com260` | 交叉验证 | 顶层 `/ { model = "..."; }` 直接读出 |
| compatible | `spacemit,k3-com260` | 交叉验证 | 顶层 `/ { compatible = "..."; }` 直接读出 |
| include | `k3_com260.dtsi` + `k3-camera.dtsi` | 交叉验证 | 顶部 `#include` 直接读出 |
| 摄像头 | imx219(CSI0)、ov5647(CSI2)、ov5640_max96724_max9295(CSI0 gmsl)、imx415(CSI2 disabled) | 交叉验证 | `&i2c5` 节点直接读出 |
| USB3_porta | 启用 + dr_mode=otg + monitor-vbus + usb-role-switch, 含 fusb301 type-c 控制器 | 交叉验证 | `&usb3_porta` 与 `&i2c0 { tcpc@25 { fusb301 } }` 节点直接读出 |
| USB3_portb | 启用 + VL817 4-port USB2.0 hub(0x1) + VL817 4-port USB3.0 hub(0x2) | 交叉验证 | `&usb3_portb { hub@1, hub@2 }` 节点直接读出 |
| GMAC | `&eth1 { phy-handle = <&gmac1_phy>; mdio { gmac1_phy: ethernet-phy@1 { compatible = "ethernet-phy-id001c.c916" + "ethernet-phy-ieee802.3-c22"; realtek,aldps-enable; realtek,clkout-disable; realtek,link-poll; wakeup-source; } } }` | 交叉验证 | `&eth1` + `mdio` 节点直接读出 |
| flexcan2 | 启用 + 80 MHz + pinctrl = can2_1_cfg | 交叉验证 | `&flexcan2` 节点直接读出 |
| CCIC | ccic0/2 启用; ccic1/3 禁用 | 交叉验证 | `&ccic0/1/2/3 { status }` 节点直接读出 |

### 4.2 `k3_com260.dtsi`(共享 base)

| 字段 | 值 | 等级 | 说明 |
| --- | --- | --- | --- |
| include | `k3.dtsi` + `k3-rdomain.dtsi` + `k3-pinctrl.dtsi` + `k3_opp_table.dtsi` + `lcd/lcd_tc358762xbg_dpi_800x480.dtsi` + `lcd_dsi_panel.dtsi` + `k3-dp1.dtsi` | 交叉验证 | 顶部 `#include` 直接读出 |
| chosen | `bootargs = "earlycon=sbi console=ttyS0,115200 loglevel=8 random.trust_bootloader=1 unaligned_scalar_speed=fast unaligned_vector_speed=fast"`, `stdout-path = "serial0:115200"`, `rng-seed = <0x25d69b2 0xff555073 0xd23238ea 0x57aa5455 0x792478ed 0xa744f28e 0x6ba4fc54 0xa2bf20fc>` | 交叉验证 | `chosen` 节点直接读出 |
| cpus | `cpus { #address-cells = <1>; #size-cells = <0>; timebase-frequency = <24000000>; }` | 交叉验证 | `cpus` 节点直接读出; CPU 个数与 hart 拓扑由 `k3.dtsi` 提供, 本文件不含 |
| &resmem | `cmamem: linux,cma { compatible = "shared-dma-pool"; reusable; size = <0 0x20000000>; alloc-ranges = <1 0x40000000 0 0x20000000>; linux,cma-default; }` | 交叉验证 | `&resmem` 节点直接读出; 父节点 `reserved-memory` 在 `k3.dtsi` 中定义 `#address-cells = <2>; #size-cells = <2>`, 两 cell 编码后 CMA 允许区间起点为 `0x140000000`, 大小 `0x20000000`; 详见 §6.6 |
| &ec_master | `&ec_master { master0 { main-device = <&eth1>; }; }` | 交叉验证 | EtherCAT master 绑定 eth1 |
| &eth1 | `max-speed = <1000>; phy-mode = "rgmii"; snps,reset-gpios = <&gpio 1 5 GPIO_ACTIVE_LOW>; snps,reset-delays-us = <0 20000 100000>; spacemit,clk-tuning-enable; spacemit,clk-tuning-by-delayline; spacemit,tx-phase = <47>; spacemit,rx-phase = <53>;` | 交叉验证 | `&eth1` 节点直接读出 |
| &pcie0_rc | num-lanes=4, phys = phy0 + phy1 | 交叉验证 | PCIe0 x4 直接读出 |
| &pcie3_rc | phys = phy4 | 交叉验证 | PCIe3 single 直接读出 |
| &pcie4_rc | phys = phy5 | 交叉验证 | PCIe4 single 直接读出 |
| 摄像头(i2c5) | imx219(CSI0, csi-id=0)、ov5647(CSI2, csi-id=2)、ov5640-max96724-max9295(CSI0 gmsl, 4 lanes)、imx415(CSI2, disabled) | 交叉验证 | `&i2c5` 节点直接读出 |
| eeprom | `&i2c2 { eeprom@50 { compatible = "atmel,24c02"; read-only; nvmem-layout { product_name: product-name {}; } } }` | 交叉验证 | ONIE tlv layout, 持 product-name nvmem 节点, 实际值未读 |
| display | `&dsi0 { panel0@0 { compatible = "raspberrypi,dpi-panel"; force-attached = "lcd_tc358762xbg_dpi_800x480"; } }` + `&i2c3 { raspits-panel@45 { compatible = "raspberrypi,7inch-touchscreen-panel"; reg = <0x45>; }; raspits-touch-ft5426@38 { compatible = "raspits_ft5426"; reg = <0x38>; }; }` | 交叉验证 | 直接读出 |
| thermal | `thermal_top`、`thermal_vpu`、`thermal_gpu`、`thermal_cluster0/1/2/3`; 集群冷却设备与 trip 温度直接读出 | 交叉验证 | `&thermal_zones` 节点直接读出 |
| aldo/dcdc/dldo/edcdc/pvin/pwr_* | 27 个 regulator, 电压与 boot-on 直接读出 | 交叉验证 | `&rpmi_regulator` 节点直接读出 |
| 内存 / aliases | 本文件不含 `memory@` 或 `aliases` 节点, 由 k3.dtsi 提供; `memory@` 已在 §6.6 由本 Cycle 直接打开 `k3.dtsi` 后解出, 本节不重复引用其值; `aliases` 节点内容仍属 §6.2 未知项 | 内存/已知(见 §6.6), aliases/未知(见 §6.2) | 本文件未直接给出, 依赖 `k3.dtsi` |

### 4.3 `k3_com260_kit_v02.dts`(Kit V02)

| 字段 | 值 | 等级 | 说明 |
| --- | --- | --- | --- |
| model | `SpacemiT K3 Com260 Kit V02` | 交叉验证 | 顶层 `/ { model = "..."; }` 直接读出 |
| compatible | `spacemit,k3-com260-kit-v02` | 交叉验证 | 顶层 `/ { compatible = "..."; }` 直接读出 |
| include | `k3_com260.dtsi`(无 `k3-camera.dtsi`) | 交叉验证 | 顶部 `#include` 直接读出 |
| 摄像头 | `&i2c5 { status = "disabled"; }`, 无 imx219/ov5647/ov5640/imx415 节点 | 交叉验证 | `&i2c5` 节点直接读出 |
| flexcan | 0/1/2/3/4 共 5 个 + r_flexcan2, 全部 80 MHz 启用 | 交叉验证 | `&flexcan0/1/2/3/4` 与 `&r_flexcan2` 节点直接读出 |
| uart4/uart5 | 启用 + pinctrl = uart4_0_cfg/uart5_2_cfg | 交叉验证 | `&uart4` / `&uart5` 节点直接读出 |
| &qspi | spacemit,qspi-tx-dma=0, rx-dma=0 | 交叉验证 | `&qspi` 节点直接读出 |
| &usb3_porta | `/delete-property/ monitor-vbus` | 交叉验证 | `&usb3_porta` 节点直接读出 |
| &eth1 | tx-fifo-depth=8192, rx-fifo-depth=8192, snps,tso, snps,force_sf_dma_mode; phy-handle = <&rgmii1>; mdio { rgmii1: phy@1 { compatible = "ethernet-phy-id001c.c916"; reg = <0x1>; device_type = "ethernet-phy"; } }` | 交叉验证 | `&eth1` + `mdio` 节点直接读出 |
| &usb3_portb | VL817 4-port USB2.0 hub(0x1) + VL817 4-port USB3.0 hub(0x2), 注释保留但 reset 由 HW 负责 | 交叉验证 | `&usb3_portb` 节点直接读出 |
| &i2c0 | tcpc@25 { wakeup-source; }(不带 fusb301 子节点) | 交叉验证 | `&i2c0` 节点直接读出 |
| thermal_gpu cooling | map1/2 改为 `cooling-device = <&imggpu 1 2>` / `&imggpu 2 3`(与基础款 0..1 / 2..3 / 4..5 不同) | 交叉验证 | `&thermal_zones { thermal_gpu { cooling-maps } }` 节点直接读出 |

## 5. 产品版本与 DTS 命名映射

| 来源字段 | 字段值 | 等级 | 来源 |
| --- | --- | --- | --- |
| 用户指南产品版本 | `K3-CoM260_P1_LP5315B_32X2_v03_20260312` | 交叉验证 | com260_user_guide.md V2.0 资料下载部分(由 GitHub 对应页直接打开, 2026-09-07); 官网 URL 正文为 SPA 壳, 不直接给出产品版本字符串 |
| 用户指南文档版本 | V2.0, 2026-03-19 | 交叉验证 | com260_user_guide.md GitHub 对应页 V2.0 修订字段直接读出(2026-09-07); 官网 URL 正文为 SPA 壳, 仅 GitHub 对应页可证 V2.0/2026-03-19 |
| DTS 命名集合 | `k3_com260.dts`、`k3_com260_kit_v02.dts`、`k3_com260_ifx.dts`、`k3_com260_ifx2.dts`、`k3_com260_ifx_tq.dts`、`k3_com260_tq.dts`、`k3_com260.dtsi` | 交叉验证 | linux-6.18 `k3-br-v1.0.y` 分支目录(直接观察 2026-09-07) |
| 文件名与产品版本唯一映射 | 不存在: `k3_com260_kit_v02.dts` 名称含 `v02`, 但用户指南产品版本是 `v03`; 其余 5 个 DTS 名称不含版本号或含第三方前缀(ifx / tq) | 未知项 | 综合对比, 2026-09-07 |
| 解除唯一映射的条件 | CoM260 Kit 原理图(下载制品)或 com260_hw_resources.md 中命名的目标 DTS; 或 com260_user_guide.md 后续修订明确给出 Kit 与 DTS 的对应表 | 解除条件 | 综合推断, 2026-09-07 |

- 命名映射规则: 本仓库内只有同时满足 (a) 直接打开 DTS 文件, (b) 直接读到 product-name / chosen / &eth1 字段或工程目录中 partition 引用, (c) 用户指南产品版本与 DTS 命名严格对齐 三个条件时, 才允许把该 DTS 记为 CoM260 Kit 的目标 DTS; 当前 Cycle 同时满足 (a) 与 (b) 的候选只有 `k3_com260.dts` / `k3_com260.dtsi` / `k3_com260_kit_v02.dts` 三个, 三者均未满足 (c), 故全部按"未唯一映射"处理。
- 候选集合中的 `k3_com260_kit_v02.dts` 名称与 Kit 名称最接近, 但版本号不匹配(v02 vs v03), 不视为已证明映射; 仅作"未唯一映射"候选, 写入 §4.3 的字段而不作为 Kit 的事实代表。

## 6. 未知项闭包

每个未知项含: `当前证据` / `禁止推断` / `解除条件` / `影响主题` 四字段。

### 6.1 CoM260 Kit 默认目标 DTS

- 当前证据: §3 中 7 个候选均无唯一映射证据; 命名集合中 `k3_com260_kit_v02.dts` 是最接近候选, 但版本号不匹配。
- 禁止推断: 不由"名称最接近"推定该 DTS 即为 Kit 默认; 不由 `compatible` 字符串推定硬件 layout 差异; 不由 `model` 字符串推定。
- 解除条件: CoM260 Kit 原理图(com260_hw_resources.md 下载制品)中 DTS 路径或 board 标识; 或 com260_user_guide.md 后续修订明示对应表; 或 buildroot defconfig 包含 `BR2_TARGET_KERNEL_DTB` 明确指向某一 DTS。
- 影响主题: MS04(平台资源)、MS05(中断与时间)、MS07(GMAC/PHY 介质归属)、MS07(EtherCAT 物理通路)。

### 6.2 K3 SoC 顶层 `aliases` 节点

- 当前证据: `k3.dtsi` 在 2026-09-07 直接打开, 根 `#address-cells` / `#size-cells` / `memory@102000000` / `reserved-memory` 已解(详见 §6.6); Iteration 001 / T3 在 2026-09-08 由 [`docs/serial/com260-uart.md`](../serial/com260-uart.md) §3 / §5 / §7 重新观察 `k3.dtsi` 的 `aliases` 节点, 确认 `serial0`..`serial10` → `uart0`..`uart10`、`serial11`..`serial16` → `r_uart0`..`r_uart5`, 其中 `serial0` 静态指向 `&uart0`(base `0xd4017000`)。非串口别名(例如 `ethernet0` 等) 在本文件中仍按未知项保留。
- 禁止推断: 不由静态 `serial0 = &uart0` 推定 bootloader 最终 cmdline、目标 Kit 顶层 DTS 或 Kit 实际 console; 不由 `cpus { timebase-frequency = 24000000 }` 推定 timer 频率; 不由 bootargs 字符串推定 memory 段; 不由 `serial0` 的静态映射反推 RCPU 串口别名集合的运行行为。
- 解除条件: 非串口别名(`ethernet0` 等)需直接打开 `k3.dtsi` 的完整 `aliases` 节点, 并补充到 [`docs/serial/com260-uart.md`](../serial/com260-uart.md) §5 / §7 静态链; 或 com260_hw_resources.md 下载制品中 SoC TRM 给出 aliases 完整映射。
- 影响主题: MS04(平台资源; 串口别名静态映射已闭合, 后续扩展到非串口别名)、MS05(中断与时间)、MS06(DMA/IOMMU)。

### 6.3 CoM260 Kit 上的 PHY 型号与 GMAC 绑定

- 当前证据: `k3_com260.dts` / `k3_com260_kit_v02.dts` 均把 PHY 标识为 `ethernet-phy-id001c.c916`(Realtek) + 0x1 reg; 但 PHY 实例/线序/参考电压在两个 DTS 间存在差异(`phy-handle` 不同、`gmac1-6-pins` 仅在 dtsi 出现)。
- 禁止推断: 不由 PHY 字符串推定 PHY 寄存器布局或 EEPROM 加载流程; 不由 `&eth1 { max-speed=1000; phy-mode="rgmii" }` 推定 PHY 子节点寄存器; 不由 RGMII delayline 值(tx-phase=47, rx-phase=53)推定 CoM260 Kit 实际值。
- 解除条件: CoM260 Kit 原理图 PHY 型号字段; `&gmac1_phy`/`&rgmii1` 子节点直接观察; PHY datasheet 或 RTL 寄存器手册。
- 影响主题: MS07(网络, G3 缺口)。

### 6.4 镜像内 `bootfs.img` / `rootfs.ext4` 内部组成

- 当前证据: docs-buildroot image.md 未展开 zip 内部组成, 也未直接给出 bootfs/rootfs 容量与 partition 布局; docs-buildroot boot.md 提到镜像 `version: 1.0` 是 partition JSON 自身 schema, 不是镜像制品。
- 禁止推断: 不由 SDK 通用示例推定 bootfs 256M / rootfs 剩余; 不由 U-Boot 2022.10 / Linux 6.18 通用 size 推定; 不由任意镜像的 release 信息(size / 校验 / commit / 制品号)推回本镜像制品; 本节仅关注 zip 内部组成与 partition 字段, 不记录 release management 相关断言。
- 解除条件: docs-buildroot image.md 或镜像下载页直接展开 zip 内 `bootfs.img` / `rootfs.ext4` 容量与 partition JSON 字段(本 cycle 仅 T12 范围内改善, 不含 release management / size 校验 / commit 锁定, 见 §8 Non-goals)。
- 影响主题: MS04(平台资源)、MS07(网络/存储介质落点)。

### 6.5 启动参数与 chosen 字段在 Kit 上的覆盖

- 当前证据: `k3_com260.dtsi` 中 `chosen { bootargs = "earlycon=sbi console=ttyS0,115200 loglevel=8 random.trust_bootloader=1 unaligned_scalar_speed=fast unaligned_vector_speed=fast" }`, 含 8 字 rng-seed; 但 bootargs 是否会被 bootloader / cmdline 文件(env_k3.txt / extlinux.conf)覆盖未知。
- 禁止推断: 不由 bootargs 字符串推定 Kit 默认内核命令行; 不由 stdout-path 串推定 Kit 真实 console。
- 解除条件: CoM260 Kit 启动日志或 `bootfs/env_k3.txt` 实际内容直接观察; bootfs 镜像解包或 cat /proc/cmdline 现场抓取。
- 影响主题: MS05(中断与时间)、MS07(网络与存储的运行时身份)。

### 6.6 CMA 0x140000000 与 Kit DRAM 实布局的关系

- 当前证据: `k3_com260.dtsi` 的 `&resmem { cmamem: size = <0 0x20000000>; alloc-ranges = <1 0x40000000 0 0x20000000>; }` 是 raw 文本直接读出; `k3.dtsi` 定义根 `#address-cells = <2>; #size-cells = <2>`, 且 `reserved-memory { #address-cells = <2>; #size-cells = <2>; ranges; }`(本 Cycle 由 docs-boot-image-dts 在 2026-09-07 直接打开 `k3.dtsi`)。按父节点 `reserved-memory` 的两 cell 解码: 地址 cell `<1 0x40000000>` → `0x140000000`, 大小 cell `<0 0x20000000>` → `0x20000000`。即 CMA 允许区间起点物理地址 `0x140000000`, 大小 `0x20000000`(`512 MiB`)。DRAM 起点来自 `k3.dtsi` 的 `memory@102000000 { reg = <0x1 0x02000000 0x1 0xfe000000>; }`, 即 DRAM 起点 `0x102000000`, 大小 `0x1fe000000`(约 `8 GiB`, 但仍按 cell 编码读取)。
- 禁止推断: 不由 `0x140000000` 推定 DRAM 总大小或 kernel 加载地址; 不由 alloc-ranges 字段推定 Kit 实际预留区; 不由 `0x20000000` 推定 Kit 实际 CMA 大小; 不由 `k3.dtsi` 的 memory 起点推定 SoC 在所有变体上的 DRAM 起点。
- 解除条件: CoM260 Kit 原理图中 DRAM 配置字段; buildroot defconfig 中 `BR2_LINUX_KERNEL_INTREE_DTS_NAME` 实际取值; CoM260 Kit 启动日志中实际 CMA 预留大小。
- 影响主题: MS06(DMA/IOMMU)、MS07(GMAC ring 缓冲区布局)。

### 6.7 4 个未直接打开 CoM260 命名候选的字段

- 当前证据: `k3_com260_ifx.dts`、`k3_com260_ifx2.dts`、`k3_com260_ifx_tq.dts`、`k3_com260_tq.dts` 仅以目录列名出现, 字段未直接打开。
- 禁止推断: 不由 `k3_com260.dts` 字段推定 ifx/tq 变体字段; 不由 `_ifx` 后缀推定 Infineon 硬件, 不由 `_tq` 后缀推定 TQ(天嵌)硬件。
- 解除条件: 直接打开 4 个文件 raw URL; 或 com260_user_guide.md 后续修订中给出 CoM260 Kit 与这些 DTS 的对应说明。
- 影响主题: MS04(平台资源)、MS07(具体 GMAC/PHY 介质)。

## 7. 修订快照

| 文档来源 | 已观察修订 | 观察日期 | 状态 |
| --- | --- | --- | --- |
| docs-buildroot image.md | 页面未提供版本字段; 已读出 6 行正文 | 2026-09-07 | observed |
| docs-buildroot boot.md | 页面未提供版本字段; 已读出 1740 行正文 | 2026-09-07 | observed |
| linux-6.18 `k3-br-v1.0.y` 分支 | branch HEAD(无 V 字段), 目录 40 条目(39 文件 + 1 子目录) | 2026-09-07 | observed |
| k3_com260.dts raw | 已读出 7 个 &node override 块 + model + compatible | 2026-09-07 | observed |
| k3_com260.dtsi raw | 已读出 chosen + &resmem + 27 个 rpmi_regulator + thermal zones | 2026-09-07 | observed |
| k3_com260_kit_v02.dts raw | 已读出 model + compatible + 5 flexcan + 2 uart + &qspi + &eth1 | 2026-09-07 | observed |
| k3.dtsi raw | 已读出根 #address-cells=2/#size-cells=2 + memory@102000000 + reserved-memory 子节点 | 2026-09-07 | observed |
| com260_user_guide.md(官网) | SPA 壳, 正文未直接取得 | 2026-09-07 | partially-observed |
| com260_user_guide.md(GitHub 对应页) | V2.0, 2026-03-19, 资料下载部分列出 `K3-CoM260_P1_LP5315B_32X2_v03_20260312.pdf` | 2026-09-07 | observed |

## 8. 边界声明

1. 本文档不下载、解析、登记镜像制品, 不登记 buildroot 配置文件, 不登记 SDK 发行版的版本号管理。
2. 本文档只在已直接打开的 4 个 DTS / DTSI 文件中记录 model / compatible / include / chosen / &resmem 字段(`k3_com260.dts` / `k3_com260_kit_v02.dts` / `k3_com260.dtsi` / `k3.dtsi`); 其他 4 个 CoM260 命名候选只列文件名, 不写字段。
3. CoM260 Kit 默认目标 DTS 未被唯一映射, `k3_com260_kit_v02.dts` 仅作"未唯一映射"候选; 名称与产品版本不对齐的事实按本 change design D2(四层事实模型) / D3(精确 URL 登记)标记。
4. CMA 字段 `alloc-ranges = <1 0x40000000 0 0x20000000>` 来自 `k3_com260.dtsi`; 父节点 `reserved-memory` 在 `k3.dtsi` 中定义 `#address-cells = <2>; #size-cells = <2>`, 两 cell 解码后 CMA 允许区间起点为 `0x140000000`, 大小 `0x20000000`(`512 MiB`)。本文不由此推定 DRAM 总大小或 kernel 加载地址; 不由 `0x140000000` / `0x20000000` 推定 Kit 实际 CMA 预留。
5. `chosen { bootargs }` 来自 `k3_com260.dtsi`, 不推定 Kit 实际 cmdline; Kit 实际 console 与 cmdline 的运行时边界由 MS04 进一步展开(参见 [`../serial/com260-uart.md`](../serial/com260-uart.md) §7.2 / §10.2)。
6. 文档中所有"型号/型号值"区分四组: (a) K3 SoC 型号(派生自 k3_ds.md), (b) K3 CoM260 Kit 产品版本(来自 com260_user_guide.md V2.0 `K3-CoM260_P1_LP5315B_32X2_v03_20260312`), (c) DTS 命名(`k3_com260*.dts`/`.dtsi`), (d) Linux 分支与 SDK baseline(`k3-br-v1.0.y` / OpenSBI 1.6 / U-Boot 2022.10 / Linux 6.18 / Buildroot 2025.02.6); 任一项不得互推。
7. docs-buildroot image.md / boot.md 仅作 `交叉验证` 来源; 官网 image.md / boot.md 仍是唯一 `官方事实` 来源, 但正文为 SPA 壳, 实际可见身份为 Vue SPA title="SpacemiT", `partially-observed` 状态保持。com260_user_guide.md 官网 URL 同样为 SPA 壳, 仅 GitHub 对应页可证 V2.0/2026-03-19 与产品版本字段。
8. docs-buildroot image.md 给出的镜像下载页与 Titan Flasher 使用手册两个外部入口, 不在本文档正文重复登记裸 URL, 视为已登记 image.md 对应行备注的链接目标。
