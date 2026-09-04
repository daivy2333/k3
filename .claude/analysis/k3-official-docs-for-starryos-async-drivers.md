# K3 官方资料面向 StarryOS 异步驱动的聚合分析

> Snapshot: [SNAPSHOT](../docs/SNAPSHOT.md)
> Captured revision: 8a97785ce339d8298421ba4a398709db5d65ca42
> Observed branch: main
> Captured at: 2026-09-02
> Related repository: StarryOS `net-k3` at `b83e800aa937568eff3a11c32e840b0b8730eade`

## 结论

K3 官网不是一个适合一次性照搬的单册手册，而是芯片资料、生态板卡资料和 Buildroot 外设指南组成的持续更新文档树。截至 2026-09-02，本次从官网页面及其官方文档仓库所能核对到的主路径中，已有 **至少 52 篇非索引主题页**；这个数字尚未计入 USB 子树、K3 RV2768、K3 Shelf，以及 `dpdk`、`esos`、`kernel_debug`、`media` 等未逐页清点的目录，因此只是下界，不是官网总数。

为帮助 StarryOS 开发 K3 异步设备驱动，建议不要按官网目录全量搬运。第一轮聚合应限定为 **15 个主题**，优先解除 StarryOS MS09–MS12 的板级事实、启动、中断、GMAC、DMA/cache 和异步完成通知依赖；第二轮再按实际驱动方向补充 13 类外设。显示、媒体、AI 和发行版应用层资料暂不进入驱动基础集。

当前最重要的阻塞不是异步抽象，而是 **目标 K3 板卡尚未在 StarryOS 中选定，且仓库还没有 K3 platform descriptor、AIA 中断控制器、GMAC 或 DMA/cache 实现**。K3 product brief 和 Linux 使用指南能证明能力与资源组成，但不能单独充当寄存器级编程手册。任何 MMIO、IRQ、时钟、复位、pinctrl、PHY 和 cache coherency 常量都必须继续由目标板 DTS、官方内核驱动和实机观测交叉确认。

## 调研问题与范围

本分析回答五个问题：

1. K3 官网当前大约有多少内容，哪些目录与设备驱动直接相关？
2. StarryOS 已经具备哪些可复用的异步驱动契约，K3 端还缺什么？
3. 第一轮应聚合哪些官方页面，什么顺序最能降低返工？
4. 哪些页面只需登记链接，等对应驱动进入里程碑后再展开？
5. 哪些事实不能从现有页面推出，必须留作 DTS、源码或实机验证项？

范围仅覆盖 K3 芯片及官方 K3 生态板卡，目标仓库为只读调查。本文不选择目标板、不修改 StarryOS 产品代码，也不将 Linux 驱动实现直接翻译成 Rust 驱动。

## 来源与计数方法

K3 文档站使用动态页面，单个入口不能稳定返回完整目录树。因此本次采用两层核对：

- 以 [K3 官方文档入口](https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs) 及其具体文档页作为内容权威源；
- 以 SpacemiT 官方组织下的 `docs-chip`、`docs-product`、`docs-buildroot` 仓库观察目录和文件名，用于清点与发现，不以仓库目录替代官网正文。

“主题页”排除 `index.md` 和静态资源目录。同一 Markdown 文件只计一次。无法完整展开的目录不估算，因此总数采用下界。

| 区域 | 已观察主题页 | 说明 |
| --- | ---: | --- |
| 芯片资料 `k3_docs`、`k3_hw`、`k3_sw` | ≥ 4 | Product Brief、datasheet、SDK guide、hardware FAQ；用户手册子树未完整展开 |
| 板卡资料 K3 CoM260、K3 Pico | 7 | CoM260 4 篇、Pico 3 篇；RV2768 与 Shelf 未计入 |
| Buildroot `device/peripheral_driver` | 29 | 不含索引；USB 子目录未展开 |
| Buildroot `device` 上层主题 | 7 | boot、device management、HMP、PLT、solution management、standby、TLV |
| Buildroot 根级主题与 release note | 5 | FAQ、image、intro、source、当前 release note |
| **已观察下界** | **≥ 52** | 未计 USB 子树及其他 SDK 目录 |

这个清点只能描述 2026-09-02 的站点结构。后续聚合时，每篇 `docs/` 文档仍须按 M02 在首行记录实际来源 URL 和观察日期。

## StarryOS 当前落点

StarryOS 现有网络异步链路主要围绕 QEMU VirtIO-MMIO 建立，MS07 已完成，MS08 及目标板阶段仍在规划中。与 K3 相关的直接证据只有工作分支名 `net-k3`；这不能替代目标板选择或硬件声明。

代码层的关键边界如下：

- 根 [Cargo.toml](../../../StarryOS/Cargo.toml) 只有 QEMU、Lichee D1 和 VisionFive 2 等平台 feature，没有 K3 feature。
- [platform/mod.rs](../../../StarryOS/kernel/src/platform/mod.rs) 只导出 D1、QEMU 和 VisionFive 2，默认 descriptor 仍落到 QEMU。
- [platform/descriptor.rs](../../../StarryOS/kernel/src/platform/descriptor.rs) 已集中描述内存、console、interrupt、timer、boot 和可选 VirtIO NIC，但 interrupt 目前只有 PLIC base，无法表示 K3 AIA/APLIC/IMSIC 拓扑、clock/reset、pinctrl、DMA 或原生 MAC。
- [uart_init.rs](../../../StarryOS/kernel/src/drivers/uart_init.rs) 已有 16550 与 D1 DW APB 两条异步 UART 路径。K3 官方 UART 页给出的 `reg-shift = 2`、`reg-io-width = 4`、256-byte FIFO 和阈值配置，不能无条件套用现有 QEMU byte-MMIO 路径。
- [axdriver_net/src/lib.rs](../../../StarryOS/crates/axdriver_net/src/lib.rs) 已有 `NetQueueControl`、`NetDriverOps`、TX ledger 和 recovery contract，可作为 K3 NIC 的上层契约；K3 仍需先完成原生 MAC 的 polling 数据面、DMA ownership 和中断完成语义。

依赖关系是：

```text
目标板与 SoC 官方事实
  ├─ boot / memory / console
  ├─ pinctrl / clock / reset
  ├─ AIA/APLIC/IMSIC routing
  └─ GMAC / PHY / DMA / cache coherency
               ↓
      K3 platform descriptor 与 IRQ dispatch
               ↓
       polling RX/TX + bounded reclaim
               ↓
  NetQueueControl / wakeup / arm-and-recheck
               ↓
       StarryOS async network stack runner
```

因此，“先读 GMAC 一页就写异步 NIC”会跳过平台、DMA 与中断三个前置层，难以判断丢中断、cache stale、PHY 不起链路或队列所有权错误分别发生在哪里。

## 第一轮：15 个必须聚合的主题

第一轮不是 15 个固定文件名，而是 15 个内容职责；当一个职责需要 datasheet、板卡页和 Buildroot 页共同佐证时，应在同一主题文档中保留多来源链路。

| 优先级 | 主题 | 解除的 StarryOS 依赖 | 主要官方入口 |
| --- | --- | --- | --- |
| P0 | 1. K3 架构与 datasheet | CPU/hart、内存、外设能力边界 | [K3 datasheet](https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs/k3_ds.md) |
| P0 | 2. 目标板选择对照 | 明确 Pico、CoM260 或其他板，停止传播“net-k3 即目标板”的假设 | [CoM260](https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/eco/k3_com260/root_overview.md)、[Pico](https://www.spacemit.com/community/development-kit/k3-pico-itx) |
| P0 | 3. 目标板硬件资源与引脚 | 板载 MAC/PHY、存储、debug UART、供电与连接器 | [CoM260 hardware resources](https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/eco/k3_com260/com260_hw_resources.md)、[Pico hardware resources](https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/eco/k3_pico/pico_hw_resources.md) |
| P0 | 4. 启动链与镜像 | OpenSBI/U-Boot handoff、装载地址、boot media | [Boot](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/boot.md)、[Image](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/image.md) |
| P0 | 5. DTS 与设备管理方法 | 建立寄存器、IRQ、clock/reset 和 compatible 的事实入口 | [Device management](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/device_management.md) |
| P0 | 6. PINCTRL | UART/GMAC/MDIO/存储引脚复用和驱动强度 | [PINCTRL](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/01-PINCTRL.md) |
| P0 | 7. Clock | 控制器输入时钟、分频、门控和性能基线 | [Clock](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/16-Clock.md) |
| P0 | 8. Reset | probe/recovery/suspend 中的复位顺序 | [Reset](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/Reset.md) |
| P0 | 9. 中断控制器 | 外部 IRQ claim/dispatch/complete、per-hart delivery、MSI | datasheet、DTS 与官方内核源码共同确认 |
| P0 | 10. Timer | deadline、超时、退避和无忙等验证 | [Timer](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/Timer.md) |
| P0 | 11. UART | 首个可观测异步设备与早期故障输出 | [UART](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/05-UART.md) |
| P0 | 12. GMAC 与 PHY | MAC/MDIO、descriptor ring、IRQ cause、link bring-up | [GMAC](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/09-GMAC.md) |
| P0 | 13. DMA | buffer ownership、descriptor、burst、地址宽度、完成中断 | [DMA](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/21-DMA.md) |
| P0 | 14. Cache/IOMMU/内存一致性 | non-coherent DMA、barrier、map/unmap、IOMMU 可见性 | datasheet、DTS、官方内核 DMA API 使用共同确认 |
| P1 | 15. SDK source 与 release notes | 锁定 Linux/U-Boot/OpenSBI 基线，跟踪 IRQ、PCIe、GMAC 与 suspend 修复 | [Source](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/source.md)、[Release notes](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/release_notes/bl-v1.0.y.md) |

建议落地为 `docs/platform/`、`docs/interrupts/`、`docs/serial/`、`docs/network/`、`docs/dma/` 和 `docs/boot/` 等主题目录，而不是复制官网的产品树。每篇文档应把“官方明确写出的事实”“由多个来源推导的结论”“仍待实机确认的参数”分开。

## 第二轮：未来有用的链接

以下资料应先登记链接，在对应 StarryOS milestone 获批后再聚合正文：

| 驱动方向 | 官方页面 | 进入聚合的触发条件 |
| --- | --- | --- |
| GPIO / edge IRQ | [GPIO](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/02-GPIO.md) | GPIO subsystem 或外设 IRQ bring-up |
| PWM / IR | [PWM](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/03-PWM.md)、[IR-RX](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/04-IR-RX.md) | timer-backed output 或 input 事件驱动 |
| I2C | [I2C](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/06-I2C.md) | PMIC、sensor 或板载控制器接入 |
| QSPI / SPI | [QSPI](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/07-QSPI.md)、[SPI](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/SPI.md) | NOR/NAND 或通用 SPI 队列驱动 |
| SDHC / UFS | [SDHC](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/08-SDHC.md)、[UFS](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/ufs.md) | 异步 block/storage milestone |
| USB | [USB 目录](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/10-USB) | host/device controller 选型完成 |
| PCIe / NVMe | [PCIe](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/11-PCIe.md) | PCIe host、MSI/MSI-X 或 NVMe 驱动进入计划 |
| CAN | [CAN](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/15-CAN.md) | CAN-FD 或实时扩展板进入计划 |
| Audio | [Audio](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/17-Audio.md) | cyclic DMA 音频流进入计划 |
| EtherCAT | [EtherCAT](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/22-EtherCAT.md) | 实时网络或 RT24 扩展进入计划 |
| WDT / RTC | [WDT](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/23-WDT.md)、[RTC](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/24-RTC.md) | watchdog、wall clock 或 suspend/resume 进入计划 |

BT、Wi-Fi、display、V2D、media、graphics、thermal、cpufreq 和 DDR 页面暂列外围资料。它们并非永远无用，但目前不会解除 StarryOS 异步 UART/NIC/block 的直接阻塞。

## 页面能够证明什么，不能证明什么

### 已确认事实

- K3 product brief 展示 AIA、DMA、Clock & Reset、3 个 GMAC、PCIe、USB、UFS、SDIO、QSPI、11 个 AP UART 等 SoC 能力。
- 官方 UART 页描述 Linux `8250_of` 路径，示例节点含 base、IRQ、clock、reset、register stride/width、FIFO 与 threshold。
- 官方 Buildroot 发布资料表明当前 K3 SDK 基于 Linux 6.18，并持续更新 IMSIC Multi MSI、SPI、PCIe、NVMe、SD/SDIO、外部中断与 GMAC pinctrl 等内容。
- StarryOS 的上层异步 NIC contract 已存在，但没有 K3 platform、原生 GMAC、AIA 或 DMA/cache 实现。

### 合理推论

- StarryOS 的 `PlatformDescriptor` 需要扩展，而不是把 K3 常量散落进 UART 或 NIC driver。
- K3 NIC 应先以 bounded polling 验证 descriptor ownership 与 cache visibility，再接入 IRQ 和 `NetQueueControl` 的 arm-and-recheck 语义。
- UART 是验证 K3 MMIO width/stride、interrupt routing 和 wakeup 的较小切口，但它不能替代 GMAC DMA 验证。

### 仍未知，不能提前固化

- StarryOS 的最终 K3 板卡、boot media、DRAM 布局和镜像格式。
- 目标 hart 的 APLIC/IMSIC 地址、IRQ domain、claim/EOI 或 MSI delivery 细节。
- 目标板启用哪个 GMAC、PHY 型号与地址、RGMII delay、reset GPIO 和 ref clock。
- DMA 是否硬件 coherent、cache line 约束、IOMMU 是否启用，以及 descriptor/data buffer 的 barrier 规则。
- GMAC 精确 IP 版本、寄存器布局、descriptor 格式、interrupt mask/ack 顺序和 errata。

## 聚合时的验收准则

每个第一轮主题至少应留下：

1. 官网来源 URL、观察日期、文档版本或页面最近更新时间；
2. 与选定板卡有关的 DTS node、compatible、MMIO range、IRQ、clock、reset、pinctrl、DMA/IOMMU 和 PHY 字段；
3. Linux 驱动入口及其只用于交叉验证的 commit/branch；
4. 可直接用于 StarryOS 的事实、需要改造的接口、禁止推断的空白；
5. polling、IRQ、async 三层验证入口，以及至少一个 timeout/error/recovery 路径；
6. QEMU 能验证的 contract 与只能由 K3 实机证明的硬件结论分界。

完成第一轮聚合后，才具备为 StarryOS MS09 提交目标板事实 change 的材料。进入 MS10–MS12 前还需要实机 boot log、DTS、寄存器读写、IRQ counter、PHY link、DMA ring 和 cache/barrier 的运行证据。

## 风险与维护策略

- 官网页面持续更新，release note 已在数月内从 K3 Buildroot v1.0 演进到 v1.0.5。聚合文档不能只写“最新版”，必须写观察日期或明确版本。
- 官方 GitHub 仓库便于查找历史和源码，但 M01 指定官网为唯一权威内容源。GitHub 和 Linux driver 应标注为目录发现或交叉验证来源。
- 页面标题、文件名和中英文 nodepath 可能不一致。链接失效时先从 R01 根入口重新定位，不应凭旧路径复制未知内容。
- Product brief 是能力清单，不是完整 programmer reference。找不到寄存器定义时应明确记为缺口，不能从 K1、DW GMAC 或通用 8250 文档推定 K3 实现。

## 本次验证与后续入口

本次只进行了文档树清点、官网检索和 StarryOS 静态阅读，没有构建、运行测试或修改 StarryOS。后续最小行动顺序为：

1. 由用户或目标板证据明确 K3 Pico、CoM260 或其他板卡；
2. 聚合第一轮 P0 主题，并记录目标板 DTS 与官方内核基线；
3. 在 StarryOS 通过 OpenSpec change 扩展 platform descriptor 和启动/中断事实；
4. 先建立 UART/GMAC polling 最小闭环，再引入 IRQ；
5. 最后复用现有 `NetQueueControl`、waker、bounded budget 和 recovery contract 完成异步化。

相关链接已登记到 [references/spec.md](../../openspec/specs/references/spec.md)，便于后续按职责检索，而不依赖本分析正文的位置。
