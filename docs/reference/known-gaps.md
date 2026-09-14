> 来源: https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs（源端修订: unknown；观察日期: 2026-09-02）

# 已知缺口

> 适用范围: 登记 `docs/` 当前无法支撑的官方资料或硬件事实，每项含解除条件。
> 产品文档通过获批 OpenSpec change 修改；change accepted 后，docs-maintainer 仅按实际结果同步 SNAPSHOT、tasks、M/D/K/R/I 中适用的状态。
> 引用来源: 当前六类缺口与 `source-coverage.md` 的 `unknown` / `unverified` / `deferred` 状态保持一致。

## 缺口登记规则

- 每类缺口必须包含：当前证据、禁止推断、解除条件、影响主题。
- 缺口解除后必须保留历史记录，记录解除日期与依据来源；不静默删除。
- 新增缺口需在主题文档写作前先登记到本表，不得在主题文档中临时声明资料不足。
- 缺口状态：`open`（待解除）/ `partial`（部分解除，仍有未解事实）/ `closed`（已解除）。

---

## G1. 官网主题页总数未知

- 分类: 范围盘点
- 当前证据: R03 分析（2026-09-02 捕获）记录 K3 主路径至少 52 篇非索引主题页；R04-R08 覆盖其中 37 个唯一 URL。
- 禁止推断: 不得用 R04-R08 的 37 个 URL 直接声称「K3 共有 37 篇或 52 篇」；不得假定未观察子树已完整。
- 解除条件:
  - 完整遍历 R01 入口的全部子树并记录页面标题与 URL；
  - 与 R08 中 `docs-chip` / `docs-product` / `docs-buildroot` 仓库目录交叉对比；
  - 形成「总页数 = N」的事实描述，并写入 `source-coverage.md` 的备注或独立 `references/spec.md` 条目。
- 影响主题: 全部 `docs/` 主题文档；任何「全 K3 资料盘点」类主题均依赖本缺口解除。

## G2. 官网未完整展开目录

- 分类: 范围盘点
- 当前证据: R03 记录 USB、RV2768、Shelf、dpdk、esos、kernel_debug 和 media 等目录未完整展开；其它可能存在的子树同样未知。
- 禁止推断: 不得以当前覆盖表为完整集合推论 K3 不存在其它资料；不得在主题文档中暗示「其余内容无需关心」。
- 解除条件:
  - 对 R01 入口逐级展开并记录每个目录子树；
  - 与 R08 的目录结构对照，确认未覆盖子树；
  - 对暂不聚合内容（如 RV2768、Shelf）保持 `out-of-scope`，对未观察内容继续登记为缺口。
- 影响主题: 全部 `docs/` 主题文档；尤其是 platform 与 buses 主题，需确认是否存在未观察的 PCIE 子页面。

## G3. K3 CoM260 实际 GMAC 实例与 PHY 详情

- 分类: 硬件事实
- 当前证据: R05 的 09-GMAC.md 仅说明 K3 SDK 中 GMAC 驱动的通用用法；R04 的 com260_hw_resources.md 提供 CoM260 模组硬件资源列表，但未直接确认 GMAC 实例编号、MMIO 基地址、PHY 型号、MDIO 地址、PHY 地址、RGMII delay、reset 与 ref clock。Iteration 001 / T11 通过直接打开 `k3_com260.dts` 与 `k3_com260_kit_v02.dts`(linux-6.18 仓库 `k3-br-v1.0.y` 分支, 2026-09-07 观察)补充出:
  - GMAC 引用 `&eth1`, `max-speed = <1000>`, `phy-mode = "rgmii"`, `snps,reset-gpios = <&gpio 1 5 GPIO_ACTIVE_LOW>`, `snps,reset-delays-us = <0 20000 100000>`, `spacemit,clk-tuning-enable`, `spacemit,clk-tuning-by-delayline`, `spacemit,tx-phase = <47>`, `spacemit,rx-phase = <53>`;
  - PHY 标识 `ethernet-phy-id001c.c916` + `ethernet-phy-ieee802.3-c22`, reg = 0x1(`k3_com260.dts` 标记为 1, `k3_com260_kit_v02.dts` 标记为 0x1), 启用 `realtek,aldps-enable` / `realtek,clkout-disable` / `realtek,link-poll`(`k3_com260_kit_v02.dts` 改用 `&rgmii1`, 关闭 realtek 属性, 增加 `tx-fifo-depth` / `rx-fifo-depth` / `snps,tso` / `snps,force_sf_dma_mode`);
  - `&ec_master { master0 { main-device = <&eth1>; } }`, EtherCAT master 绑定 eth1。
  - [`com260-gmac-phy.md`](../network/com260-gmac-phy.md) 已把 SoC、模组、DTS 变体、`eth1`、MDIO、PHY 与 RGMII 分层；2026-09-09 直接打开的 K3 Linux GMAC glue driver 补充 `spacemit,k3-gmac`、APMU interface/delay-line 和 stmmac platform handoff，但不提供 PHY 实物型号或 Kit 连接器事实。
  仍未知: PHY 实物型号(RTL8211F 还是 RTL8211FD 还是其他)、PHY 在 Kit 板上的连接器位置、MDIO/MMIO 寄存器、CoM260 Kit 是否为单 PHY 单网口、reset 与 ref clock 在 Kit 原理图中的具体实现。
- 禁止推断: 不得基于 Linux 驱动行为或 DWMAC 默认值反推 CoM260 实际硬件；不得假定 GMAC 实例与 Pico 板相同；不得由 PHY 字符串 `ethernet-phy-id001c.c916` 推定 PHY 寄存器布局或 EEPROM 加载流程；不得由 RGMII delayline 值 47/53 推定 Kit 实际值。
- 解除条件:
  - 取得 CoM260 模组 hardware resources 中关于 GMAC/PHY 的具体字段（实例号、MMIO、PHY 型号、MDIO/RGMII 参数）；
  - 或取得 CoM260 DTS（device tree）源文件并对照 compatible、reg、phy-handle、phy-mode；
  - 形成 R04 子条目的修订记录或新增 R 登记。
  - 仍需补: 取得 CoM260 Kit 原理图中 PHY 实物型号、PHY 板上位置、MDIO/MMIO 寄存器手册、CoM260 实物网口数量与连接器布局。
- 影响主题: `docs/network/`（GMAC、PHY、MDIO、RGMII）；间接影响 `docs/platform/`（clock、reset、pinctrl）。
- 状态变更记录: 2026-09-07 由 `open` 调整为 `partial`, 由 Iteration 001 / T11 直接打开 linux-6.18 `k3-br-v1.0.y` 分支 `k3_com260.dts` / `k3_com260_kit_v02.dts` / `k3_com260.dtsi` 提供 PHY 标识与 GMAC 模式；2026-09-09 经两篇 network 正文和 K3 glue driver复核后保持 `partial`，PHY 实物型号、Kit 连接器、电气参数和完整寄存器仍未解。

## G4. AIA / APLIC / IMSIC 地址、IRQ domain 与 hart delivery

- 分类: 硬件事实
- 当前证据:
  - [`k3-interrupt-and-time.md`](../interrupts/k3-interrupt-and-time.md) 已从 linux-6.18 `k3-br-v1.0.y` 的 `k3.dtsi` 与 binding 交叉验证 AP CLINT `0xe081c000/0x4000`、IMSIC `0xe0400000/0x400000`、APLIC `0xe0804000/0x4000`、`riscv,num-ids = <511>`、`riscv,num-sources = <512>`、16 hart interrupt file 与 `msi-parent = <&simsic>` 静态关系。
  - [`com260-mailbox-notification.md`](../interrupts/com260-mailbox-notification.md) 按固定 revision 第三方经验记录 mailbox4 的 AP source 217、RP source 69 与双向 handler 链；source→EID、target hart、affinity 和真板投递未由官方来源或本项目运行证据确认。
  - [`k3-gmac-dma-irq.md`](../network/k3-gmac-dma-irq.md) 记录固定 revision GMAC handler 的 status/W1C/reclaim 和 `try_lock` 失败路径；这些行为不证明 K3 source→EID、target hart、affinity 或真板 delivery。
  - [`k3-rpc-ring-notification.md`](../amp/k3-rpc-ring-notification.md) 在 mailbox 通道的 AP source 217、RP source 69 之外独立重申"通知与 ring 状态分离、IMSIC EID 由 IRQ handler 路径消费"；其 U3（mailbox FIFO 满、IRQ 屏蔽与截断）作为 G4 的二级证据指针，但仍未解除 G4 本身。
- 禁止推断: 不得用 RISC-V AIA 标准行为直接等同 K3 实现；不得假定 K3 APLIC/IMSIC 寄存器布局与 SiFive 或其他厂商一致；不得把 AP source 217、RP source 69、mailbox channel 或 IMSIC EID 视为同一编号空间；不得凭第三方 mailbox 代码关闭本缺口。
- 解除条件:
  - 取得 K3 SoC 公开寄存器手册（programmer reference 或 datasheet 寄存器章节）；
  - 或在 R08 的 linux-6.18 仓库定位 K3 APLIC/IMSIC 驱动初始化、source→EID 与 affinity 路径；
  - 取得目标 CoM260 配置的运行时 IRQ domain、target hart、mask/ack/complete 和双向 mailbox delivery 证据。
- 影响主题: `docs/interrupts/`（AIA、APLIC、IMSIC、timer）；间接影响 `docs/serial/` 与 `docs/network/`（驱动 IRQ 接入）；`docs/amp/`（[`k3-rpc-ring-notification.md`](../amp/k3-rpc-ring-notification.md) U3 沿用 G4 证据指针）。
- 状态变更记录: 2026-09-09 由 `open` 调整为 `partial`；MS05 Iteration 000 已补齐 AP 静态地址与 APLIC→IMSIC 拓扑，Iteration 001 补充固定 revision mailbox source/handler 经验；2026-09-10 由 [`k3-rpc-ring-notification.md`](../amp/k3-rpc-ring-notification.md) Iteration 001 增加 AP source 217 / RP source 69 与 IRQ handler 路径的二次引用与 U3 证据指针；寄存器布局、source→EID、target hart、affinity 与真板 delivery 仍未解除。

## G5. DMA coherency、IOMMU、cache line 与 barrier 规则

- 分类: 硬件事实
- 当前证据:
  - R05 的 21-DMA.md 仅说明 K3 SDK 中 DMA 驱动的通用配置；未公开 K3 SoC 的 cache 一致性模型、IOMMU 是否存在、DMA 地址宽度、descriptor 与数据缓冲区的 ownership 转换规则、cache line 大小、barrier 要求。
  - Iteration 000 的 [`k3-dma-and-memory-ownership.md`](../dma/k3-dma-and-memory-ownership.md) 已闭合三类 DMA 对象、descriptor/data buffer ownership 和 device-specific completion。
  - Iteration 001 的 [`k3-cache-pma-address-translation.md`](../dma/k3-cache-pma-address-translation.md) 已分开 cache maintenance、FENCE/barrier、PMA/PBMT、CPU page table、IOMMU 和 CPU/device 地址层，并记录固定 revision GMAC/UFS/shared-SRAM 软件路径。
  - [`k3-gmac-dma-irq.md`](../network/k3-gmac-dma-irq.md) 已把 GMAC descriptor 与 data buffer、Release fence、descriptor clean/invalidate、doorbell 和回收分开；该固定 revision 路径不证明默认 Kit 的 cache line、coherency、IOMMU 或 DMA aperture。
  - [`k3-rpc-ring-notification.md`](../amp/k3-rpc-ring-notification.md) 在共享消息路径上沿用 [`k3-cache-pma-address-translation.md`](../dma/k3-cache-pma-address-translation.md) 的机制边界；该路径的 `SeqCst` fence / `fence iorw, iorw` 与 mailbox doorbell 的可见性边界仍依赖 G5 的解除。
  - K3 官方概述只声明支持 IOMMU 扩展；固定第三方 IFX DTS 含 `iommu-map`、`spacemit,k3-iommu` 和 `dma-coherent`，但 G7 尚未把该变体唯一映射为默认 Kit。
- 禁止推断: 不得用通用 RISC-V 行为或 Linux DMA API 行为等同 K3 实际；不得假定 K3 默认全 coherent 或全 non-coherent；不得由第三方 OpenSBI 修改推定 PMA/PBMT 真板效果；不得由目录中未找到节点推定 IOMMU 不存在，也不得由第三方 DTS 字段推定默认启用、bypass、identity mapping 或 device domain。
- 解除条件:
  - 取得 K3 SoC 公开寄存器手册中 cache、MMU、IOMMU 章节（含 PMA mode 编码全集、cache line 大小、IOMMU 节点细节）；
  - 或在 R08 的 linux-6.18 仓库定位 K3 IOMMU 与 DMA 一致性相关 patch；
  - 或 CoM260 真板 cache coherency / IOMMU 行为实测；
  - 形成 coherency model、IOMMU presence、barrier list 与 ownership 转换条目。
- 影响主题: `docs/dma/`（[`../dma/k3-dma-and-memory-ownership.md`](../dma/k3-dma-and-memory-ownership.md) U1-U4 + [`../dma/k3-cache-pma-address-translation.md`](../dma/k3-cache-pma-address-translation.md) §1-§5）；间接影响 `docs/network/`（GMAC descriptor、buffer）、`docs/storage/`（UFS/SDHC）、`docs/amp/`（[`k3-rpc-ring-notification.md`](../amp/k3-rpc-ring-notification.md) §4.2 与 §5.3 沿用 G5 的 fence / cache / PMA 边界）。
- 状态变更记录: 2026-09-09 由 `open` 调整为 `partial`；MS06 已闭合对象 ownership、通用机制边界和固定 revision 软件路径，本 change 增加 GMAC 专题交叉引用后状态保持 `partial`；2026-09-10 由 [`k3-rpc-ring-notification.md`](../amp/k3-rpc-ring-notification.md) Iteration 001 增加 §4.2 / §5.3 对 G5 边界的二次引用；K3 cache line/coherency、PMA/PBMT 真板效果、默认 Kit IOMMU 与 device address 映射仍未解除。

## G6. K3 GMAC 寄存器、descriptor 与 interrupt ack 的 programmer reference

- 分类: 硬件事实
- 当前证据: R04 的 k3_ds.md、root_overview.md 是产品层与 overview 层资料；R05 的 09-GMAC.md 是驱动使用说明。2026-09-09 直接打开的 K3 Linux GMAC glue driver 确认 `spacemit,k3-gmac`、APMU interface/delay-line 与 stmmac platform handoff，但没有提供 DWMAC descriptor、DMA channel status/mask/W1C、复位值或错误恢复定义。[`k3-gmac-dma-irq.md`](../network/k3-gmac-dma-irq.md) 的这些细节来自固定 revision 第三方实现，不能替代 programmer reference。
- 禁止推断: 不得以 Linux DWMAC 行为或 Synopsys DWMAC 通用寄存器定义反推 K3 GMAC；不得假定 ack/cause 寄存器布局等同于上游驱动。
- 解除条件:
  - 取得 SpacemiT 官方 GMAC 寄存器手册（programmer reference）；
  - 或在 R08 的 linux-6.18 仓库定位 K3 GMAC 设备树与驱动的寄存器偏移表；
  - 形成寄存器偏移、复位值、interrupt cause/mask/ack 与 descriptor 字段定义条目。
- 影响主题: `docs/network/`（GMAC 寄存器与 descriptor、interrupt ack）；间接影响 `docs/dma/`（descriptor buffer ownership）。
- 状态变更记录: 2026-09-09 增加 K3 Linux glue driver 与 network 数据面正文后保持 `open`；glue driver 未满足 descriptor、interrupt cause/mask/ack 和复位值解除条件。

## G7. CoM260 Kit 默认目标 DTS 未唯一映射

- 分类: 硬件事实
- 当前证据: Iteration 001 / T11 直接打开 linux-6.18 仓库 `k3-br-v1.0.y` 分支 `arch/riscv/boot/dts/spacemit/` 目录, 列出 7 个 CoM260 命名候选(6 .dts + 1 .dtsi 共享 base); `k3_com260.dts`(`model = "SpacemiT K3 Com260"`)、`k3_com260_kit_v02.dts`(`model = "SpacemiT K3 Com260 Kit V02"`)、`k3_com260.dtsi`、`k3.dtsi` 四个文件已被直接打开, 其中 `k3_com260_kit_v02.dts` 名称与 Kit 最接近, 但其名称中的 `v02` 与 com260_user_guide.md V2.0 资料下载部分列出的 CoM260 产品版本 `K3-CoM260_P1_LP5315B_32X2_v03_20260312` 的 `v03` 不一致; 其余 4 个候选(`k3_com260_ifx.dts`、`k3_com260_ifx2.dts`、`k3_com260_ifx_tq.dts`、`k3_com260_tq.dts`)未直接打开, 字段全部留 `未知项`。[`com260-gmac-phy.md`](../network/com260-gmac-phy.md) 与 [`k3-gmac-dma-irq.md`](../network/k3-gmac-dma-irq.md) 均按变体和证据等级保留该边界。[`k3-amp-shared-memory-lifecycle.md`](../amp/k3-amp-shared-memory-lifecycle.md) §8 U1 与 [`k3-rpc-ring-notification.md`](../amp/k3-rpc-ring-notification.md) §10 U1 均把"目标 CoM260 实际 AMP 镜像与共享 ring 节点"留作 G7 的二级证据指针；[`k3-i2c-and-usb.md`](../buses/k3-i2c-and-usb.md)、[`k3-pcie-and-can.md`](../buses/k3-pcie-and-can.md) 与 [`k3-ethercat.md`](../buses/k3-ethercat.md) 也分别保留 controller/client/PHY、PCIe/CAN 连接和 `ec_master → eth1` 对目标 DTB 的依赖。DTS 唯一映射未解除前，这些 shared/top-level DTS 候选不能代表默认 Kit 配置。
- 禁止推断: 不得由"名称最接近"推定 `k3_com260_kit_v02.dts` 即为 CoM260 Kit 默认目标 DTS; 不得由 `compatible` 字符串推定硬件 layout 差异; 不得由 `model` 字符串推定；不得由 [`k3-amp-shared-memory-lifecycle.md`](../amp/k3-amp-shared-memory-lifecycle.md) 的第三方 DTS 注释推定目标 Kit 同样使用该 `0xc0800000/0x19000` 节点。
- 解除条件:
  - 取得 CoM260 Kit 原理图(com260_hw_resources.md 下载制品)中 DTS 路径或 board 标识;
  - 或 com260_user_guide.md 后续修订明示对应表;
  - 或 buildroot defconfig 包含 `BR2_TARGET_KERNEL_DTB` 明确指向某一 DTS;
  - 解除时需同时更新 `com260-image-and-dts.md` §3 / §4 / §5, 并登记 R04 或 R08 子条目修订。
- 影响主题: `docs/platform/`(平台资源映射)、`docs/network/`(GMAC/PHY 介质归属)、`docs/buses/`（I2C/USB/PCIe/CAN/EtherCAT 板级映射）、`docs/amp/`（[`k3-amp-shared-memory-lifecycle.md`](../amp/k3-amp-shared-memory-lifecycle.md) §8 U1 + [`k3-rpc-ring-notification.md`](../amp/k3-rpc-ring-notification.md) §10 U1 沿用 G7 的 DTS 唯一映射边界）。
- 状态变更记录: 2026-09-07 由 Iteration 001 / T11 新增, 状态 `open`；2026-09-09 两篇 network 正文完成后状态保持 `open`；2026-09-10 由 [`k3-amp-shared-memory-lifecycle.md`](../amp/k3-amp-shared-memory-lifecycle.md) 与 [`k3-rpc-ring-notification.md`](../amp/k3-rpc-ring-notification.md) Iteration 001 增加共享 ring 节点 / `0xc0800000/0x19000` 节点与 G7 的二级证据指针；2026-09-12 三篇 buses 正文增加 I2C/USB/PCIe/CAN/EtherCAT 二级指针。状态仍为 `open`，未取得产品版本到顶层 DTS 的新映射证据。

## G8. K3 SoC `uart10` base 偏移

- 分类: 硬件事实
- 当前证据: Iteration 001 / T3 由 [`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) 直接打开(2026-09-08), `uart0, uart2..uart9` base 沿 `0xd4017000 + N × 0x100` 步进, 但 `uart10` 位于 `0xd401f000`(相对 `uart9` 的 `0xd4017800` 偏移 `0x7800`), 与前 9 个非 secure 实例的 stride 公式不符; [`docs/serial/com260-uart.md`](../serial/com260-uart.md) §3.2 已闭合 base 与 IRQ 字段但保留偏移原因为未知项。
- 禁止推断: 不得用 `0x100` stride 公式推导 `uart10` base; 不得假设 `uart10` 不可用或被保留; 不得在 K3 公开资料中宣称"UART10 不可访问"; 不得由 `uart10` 偏移反推其他 AP UART 实例的步进公式。
- 解除条件:
  - 取得 `k3.dtsi` 注释或 SpacemiT 公开 programmer manual 中关于 `uart10` 偏移的说明;
  - 或 `k3_com260.dts(i)` 板级文件引用 `uart10` 时显式注释;
  - 或 docs-buildroot 05-UART.md 公开 K3 UART 总线布局。
- 影响主题: [`docs/serial/`](../serial/com260-uart.md)(完整 17 实例矩阵与 aliases `serial10 = &uart10` 语义); [`docs/platform/`](../platform/com260-board-resources.md)(板级启用 `uart10` 时的地址与 IRQ 解析)。
- 状态变更记录: 2026-09-08 由 Iteration 001 / T3 新增, 状态 `open`。

## G9. `spacemit,k1-uart` compatible 字符串的 K3 硬件边界

- 分类: 硬件事实
- 当前证据: [`8250.yaml`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/Documentation/devicetree/bindings/serial/8250.yaml) 中 `spacemit,k1-uart` 与 `intel,xscale-uart` 共同作为 compatible; [`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) 与 [`k3-rdomain.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-rdomain.dtsi) 中 17 个 UART 节点(AP 11 + RCPU 6)全部使用此组合, RCPU 节点额外携带 `spacemit,rcpu-uart` 标志。
- 禁止推断: 不得将 `spacemit,k1-uart` 字符串推定为 K1 SoC 硬件身份; 不得由命名推定 K3 UART 寄存器全集等同于 K1 PXA UART; 不得由 `intel,xscale-uart` 推定 K3 沿用 PXA IP 全部行为; 不得由 `spacemit,rcpu-uart` 标志反推 RCPU 域完整硬件能力。
- 解除条件:
  - docs-buildroot 05-UART.md 公开 K3 专属 compatible 与 K1 差异;
  - 或 K3 programmer manual 给出 K3 UART IP 修订与 PXA 关系;
  - 或 SpacemiT 公开 `8250_of.c` 中 K3 专属匹配逻辑的注释说明。
- 影响主题: [`docs/serial/`](../serial/com260-uart.md)(后续 driver 选型; future Iteration 中 K3 专属 compatible 是否独立); [`docs/platform/`](../platform/k3-platform-control.md)(命名风格与 K1 体系沿用边界)。
- 状态变更记录: 2026-09-08 由 Iteration 001 / T3 新增, 状态 `open`。

## G10. K3 UART 完整寄存器语义与电气映射

- 分类: 硬件事实
- 当前证据: [`8250.yaml`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/Documentation/devicetree/bindings/serial/8250.yaml) 给出 binding schema, [`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) 给出 base + IRQ + `reg-shift` + `fifo-size`; [`8250_of.c`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/drivers/tty/serial/8250/8250_of.c) 在 `CONFIG_SOC_SPACEMIT` 条件下包含 `spacemit_8250_set_termios` 与 `spacemit_acpu_match_clk_rate`; 但完整 UART 寄存器语义、电气特性、programmer manual 缺失。
- 禁止推断: 不得用 Linux 默认值或 PXA 通用知识补齐 K3 UART 寄存器; 不得用 8250 通用 FIFO 公式反推 K3 实际 IP 行为; 不得由 PXA errata 推定 K3 适用 errata; 不得由 `spacemit_8250_set_termios` 行为反推 K3 完整寄存器集; 不得由 [`docs/serial/com260-uart.md`](../serial/com260-uart.md) §3-§5 实例字段推定寄存器偏移全集。
- 解除条件:
  - 取得 SpacemiT 公开 programmer manual 中 K3 UART IP 章节;
  - 或 K3 SoC datasheet 寄存器章节公开;
  - 或 docs-buildroot 05-UART.md 增补 K3 专属寄存器偏移表与 errata 列表。
- 影响主题: [`docs/serial/`](../serial/com260-uart.md)(后续 driver 实现; 不在本 change 范围); MS05(IRQ budget; UART 中断 cause/ack); MS06(DMA 触发条件与 cache maintenance); MS07(GMAC 寄存器已知; UART 可类比)。
- 状态变更记录: 2026-09-08 由 Iteration 001 / T3 新增, 状态 `open`。

## G11. K3 AMP 消息路径协议闭包与缺失源码

- 分类: 协议/接口
- 当前证据: Iteration 001 由 [`k3-rpc-ring-notification.md`](../amp/k3-rpc-ring-notification.md) 汇总 R10/R11 固定 revision 调查；该正文 U1-U5 登记 ring 精确布局、AP 唯一 `IPC_WAKER`、mailbox FIFO 满与 IRQ 屏蔽、reset 后 ID 跨 epoch 状态、`SharedMemory::at(0)` Rust 有效性五项互不重复的协议级未知项。`ov-channels` 与 `rt-async` 子仓在固定 revision checkout 中缺失；`ov-channels` 的 ring size/alignment/原子序与多块发布契约、`rt-async` 的 `IrqLatch` 注册/重检/IPC_WAKER 多等待者契约均不可读。
- 禁止推断: 不由 AP 唯一 `IPC_WAKER` 推定多线程 AWAIT / epoll + AWAIT 并存安全；不假定 ring 精确布局与"0x100 + 3×0x8200 + 0x800 scratch"为通用 K3 协议；不假定 mailbox FIFO 满不会发生或 IRQ 屏蔽时间受固定预算约束；不假定 reset 后旧 request ID 仍可跨 epoch 继续匹配；不假定 `SharedMemory::at(0)` 在普通 Rust 引用下自动有效。
- 解除条件:
  - 取得 `ov-channels` 子仓并完整读取其 `Ring` 类型 size、head/tail 序、Release/Acquire 屏障、并发安全证明与多块 API；
  - 取得 `rt-async` 子仓并补查 `IrqLatch` 注册/重检契约、AP `IPC_WAKER` 的多等待者替换或扩展实现；
  - 取得 mailbox 寄存器手册并实测 FIFO 满处理、IRQ 屏蔽/恢复、截断报告契约；
  - 取得 reset 后 ID 分配、ring 头部版本、等待者取消的契约并实测覆盖；
  - 解决 [`k3-amp-shared-memory-lifecycle.md`](../amp/k3-amp-shared-memory-lifecycle.md) §8 U4 唯一初始化者与恢复协议缺口。
- 影响主题: [`docs/amp/`](../amp/)（[`k3-rpc-ring-notification.md`](../amp/k3-rpc-ring-notification.md) §3-§9 与 U1-U5；间接影响 [`k3-amp-shared-memory-lifecycle.md`](../amp/k3-amp-shared-memory-lifecycle.md) §6-§9）。与 G4（IRQ 链路）、G5（fence/cache/PMA）、G7（目标 Kit DTS 唯一映射）有交叉但职责互斥：G4 关注 APLIC/IMSIC 寄存器与 source→EID，G5 关注 cache/PMA 真板效果与 IOMMU 启用，G7 关注顶层 DTS 唯一映射，G11 关注消息路径 ring/RPC 协议闭包本身。
- 状态变更记录: 2026-09-10 由 Iteration 001 新增, 状态 `open`；本 change 仅汇总与登记五项 U 字段，不解除任何子项。

## G12. 存储控制器板级映射、运行路径与恢复闭包

- 分类: 硬件事实
- 当前证据:
  - [`docs/storage/k3-qspi-spi-sdhci.md`](../storage/k3-qspi-spi-sdhci.md) U1（QSPI 板载器件与运行路径）、U2（普通 SPI 实例与板级连接）、U3（SD 与 eMMC 的实例映射）、U4（SDHCI 消费层与完成模型）、U5（错误后的资源和介质状态）登记了非 UFS 存储的板级映射、消费层与恢复缺口；其中 U1/U4/U5 与 [`docs/dma/k3-dma-and-memory-ownership.md`](../dma/k3-dma-and-memory-ownership.md) U1–U4 在 DMA/cache 边界相交，但 QSPI/SPI/SDHCI 专属的 programmer reference、板级介质差异、removable 属性和 SDHCI tuning/HS200/HS400 并不由 G5 承载。
  - [`docs/storage/k3-ufs.md`](../storage/k3-ufs.md) U1（K3 UFS 专属 binding 缺失）、U2（IRQ 路径未启用）、U3（UFS 容量冲突 C1，与 G7 共享 DTS/容量未知项边界）、U4（同步块 I/O 在 RTOS / SMP 场景下的可扩展性）、U5（controller fatal 重建路径）登记了 UFS 专属 binding、IRQ/完成模型、同步 block 扩展性与 fatal 重建边界；其 U3 与 G7 互斥（G7 关注顶层 DTS 唯一映射，G12 关注 UFS 容量冲突在板级介质与运行路径层面的解读），U2 与 G4 互斥（G4 关注 APLIC/IMSIC 寄存器与 source→EID，G12 关注 UFS FDT IRQ 解析但未注册 handler 的运行层差异）。
  - 固定 revision 第三方 tgoskits revision `19219411d5dc1515496f910d04c93da12ee95be4` 在 [`k3_ufs/transfer.rs`](https://github.com/PlaticaIt/StarryOS/blob/19219411d5dc1515496f910d04c93da12ee95be4/drivers/ax-driver/src/block/k3_ufs/transfer.rs) 中实现 UTRD/UCD/PRDT、DMA、doorbell、轮询完成与一次 controller recovery retry + fatal latch；固定 tgoskits `k3-sdhci` core（`drivers/blk/k3-sdhci/src/lib.rs`）负责 K3 PHY、SD/eMMC mode、HS200/HS400、DLL 与 software RX tuning，但不负责 OS glue、FDT probe、IRQ 与 block registration。这两类固定第三方实现只解释一种候选变体，不能代表 K3 硬件保证。
- 禁止推断:
  - 不得由 tgoskits IFX DTS `spacemit,k3-ufshcd` 节点或 `k3-sdhci` portable core 推定默认 Kit 已启用对应路径或当前运行时使用中断推进；G7 尚未唯一映射顶层 DTS 之前，第三方变体与目标 Kit 不能等同。
  - 不得把 UFS 的 MPHY/UniPro/UTP/SCSI、DMA/cache、轮询完成、单次 recovery retry 或 fatal latch 行为外推到 QSPI、SPI 或 SDHCI。
  - 不得由 `k3_ufs` 的 `submit_upiu` 策略（Timeout/OcsError/ControllerFatal → HCE reset + 重建 link/list + NOP + 单次重试 + 失败 fatal latch）推定 K3 官方硬件保证未完成写入可安全重试或硬件复位后数据一致。
  - 不得由 SDHCI `k3-sdhci` core 存在推定 CoM260 当前 board profile 已启用 HS200/HS400、DLL 或 software RX tuning，也不得由该 core 推定 removable/bus-width 板级配置。
  - 不得在 U3 容量冲突未解除时裁决 128 GB / 256 GB 容量默认值。
- 解除条件:
  - 取得 K3 SoC 公开的 QSPI、SPI、SDHCI、UFS programmer reference 或 datasheet 寄存器章节；
  - 或在 R08 的 linux-6.18 仓库定位 K3 专属 QSPI/SPI/SDHCI/UFS 设备树与驱动的 binding/寄存器偏移表；
  - 或在 CoM260 真板取得 QSPI/SPI 板载器件型号、SD/eMMC 板级 mapping、SDHCI tuning/HS mode 配置、UFS 实际容量与 lane 配置、UFS IRQ 注册与完成路径的运行证据；
  - 形成设备专属 binding/programmer reference、板级介质差异、消费层 glue、完成模型、性能与错误恢复闭包条目。
- 影响主题: [`docs/storage/`](../storage/)（[`k3-qspi-spi-sdhci.md`](../storage/k3-qspi-spi-sdhci.md) U1–U5 + [`k3-ufs.md`](../storage/k3-ufs.md) U1–U5）；间接影响 [`docs/platform/`](../platform/com260-board-resources.md)（板级介质与控制器归属）、[`docs/boot/`](../boot/com260-boot-chain.md)（启动介质关系）、[`docs/dma/`](../dma/k3-dma-and-memory-ownership.md)（DMA/cache 边界）、[`docs/interrupts/`](../interrupts/k3-interrupt-and-time.md)（UFS IRQ 路径与 G4 互斥）；与 G5（DMA/cache/coherency）、G7（默认目标 DTS）职责互斥：G5 关注通用 DMA 与 cache 一致性，G7 关注顶层 DTS 唯一映射，G12 关注存储设备专属的板级映射、运行路径与恢复闭包。
- 状态变更记录: 2026-09-11 由 Iteration 002 新增, 状态 `open`；本 change 仅汇总两篇 storage 正文的十项 U 字段与 G5/G7 的互斥职责，不解除任何子项。

## G13. 外设总线板级映射、运行路径与恢复闭包

- 分类: 硬件事实与协议/接口
- 当前证据: [`k3-i2c-and-usb.md`](../buses/k3-i2c-and-usb.md) 记录 I2C controller/client、I²C 9/10 数量冲突、USB controller/PHY/Host/DRD/role switch/Hub 和 USB/PCIe 共享 PHY；[`k3-pcie-and-can.md`](../buses/k3-pcie-and-can.md) 记录 PCIe controller/lane/PHY/RC/EP、插槽、AP/RP FlexCAN、CAN-FD、收发器和 C2 板级冲突；[`k3-ethercat.md`](../buses/k3-ethercat.md) 记录 `ec_master → master0 → main-device = <&eth1>` 静态绑定。三篇正文都区分静态能力、DTS 候选、板级连接和运行结果，但官方 Buildroot 正文未直接取得，controller 到接口的唯一映射、软件组成、周期/同步及错误恢复仍缺证据。
- 禁止推断: 不得裁决 I²C 9/10 数量差异、USB/PCIe PHY mux、PCIe controller 到 M.2 插槽映射、CAN C2 冲突或默认目标 DTS；不得由节点、phandle、PHY、收发器或连接器存在声明设备枚举、CAN 通信、EtherCAT slave 发现、周期/同步达标或恢复成功。
- 解除条件: 取得适用同一 CoM260 产品修订的原理图、BOM、完整目标 DTB，以及 K3 I2C/USB/PCIe/CAN/EtherCAT 官方正文、binding 或驱动；在明确板型和软件版本上观察 controller/PHY/接口映射、probe/枚举/通信、错误结果与恢复路径；EtherCAT 还需记录 master/slave 配置和周期/同步测量。
- 影响主题: `docs/buses/` 三篇正文及 `docs/platform/` 板级资源；与 G3/G4/G5/G7 职责互斥：G3 负责 GMAC/PHY 详情，G4 负责 IRQ delivery，G5 负责 DMA/cache/IOMMU，G7 负责默认目标 DTS，G13 负责总线设备专属的板级映射、软件运行路径和恢复闭包。
- 状态变更记录: 2026-09-12 由 MS10 Iteration 003 新增，状态 `open`；本 change 只汇总三篇 buses 正文的未知项，不解除任何子项。

## G14. 通用外设板级映射、运行路径与恢复闭包

- 分类: 硬件事实与协议/接口
- 当前证据: [`k3-gpio-pwm-ir.md`](../peripherals/k3-gpio-pwm-ir.md) 记录 GPIO controller/pinctrl/IRQ、PWM channel/pinmux/consumer、IR-RX controller/input 及 U1-U5；[`k3-audio.md`](../peripherals/k3-audio.md) 记录 I²S/SSPA controller、DAI、sound card、codec/display endpoint、DMA/power domain 及 U1-U4；[`k3-wdt-rtc.md`](../peripherals/k3-wdt-rtc.md) 记录 WDT、MMIO RTC、RPMI RTC、mailbox/IRQ、VCC_RTC 及 U1-U4。三篇正文均区分 SoC 能力、DTS 静态资源、板级连接、固件代理与运行结果，但官方 Buildroot 正文未直接取得，板级唯一映射、运行所有权、协议参数、真板行为和恢复路径仍缺证据。
- 禁止推断: 不得裁决 PWM 30/20 数量冲突、GPIO bank 完整范围、IR-RX 协议/keymap、codec/route、AP/RCPU Audio 所有权、WDT timeout/restart、MMIO/RPMI RTC 所有权或 VCC_RTC 掉电保持；不得由节点、pinmux、IRQ、DMA、mailbox、sound card 或电源引脚存在声明 GPIO/PWM/IR、Audio、WDT reset、RTC timekeeping/alarm/唤醒已经运行。
- 解除条件: 取得适用同一 CoM260 产品修订的原理图、BOM、完整目标 DTB，以及六个 K3 Buildroot 官方正文、binding、driver 或 programmer reference；在明确板型和软件版本上观察 GPIO/PWM/IR 输入输出、Audio stream/xrun、watchdog timeout/reset、两条 RTC 路径和掉电保持的结果与恢复行为。
- 影响主题: `docs/peripherals/` 三篇正文及 `docs/platform/` 板级资源；与 G4/G5/G7 职责互斥：G4 负责 IRQ delivery，G5 负责 DMA/cache/IOMMU，G7 负责默认目标 DTS，G14 负责通用外设专属的板级映射、软件/固件运行路径、协议参数和恢复闭包。
- 状态变更记录: 2026-09-12 由 MS11 Iteration 003 新增，状态 `open`；本 change 只汇总三篇 peripherals 正文的 13 组未知项，不解除任何子项。

## G15. CoM260 镜像部署、刷写与恢复操作闭包

- 分类: 镜像部署与恢复
- 当前证据: [`k3-image-build-and-artifacts.md`](../boot/k3-image-build-and-artifacts.md) 已分离构建输入、产物、容器与消费方，[`k3-ram-boot-and-fastboot.md`](../boot/k3-ram-boot-and-fastboot.md) 已分离 BootROM、U-Boot Fastboot 与 RAM 临时启动路径，[`k3-flashing-and-recovery.md`](../boot/k3-flashing-and-recovery.md) 已分离 GPT、MTD、SD 与 Titan 四类持久化路径及恢复门槛；但当前来源仍不能闭合适用于同一 CoM260 产品修订的官方镜像包、当前启动与下载设备身份、目标介质与分区映射、Titan/工具兼容性、SD 启动选择、逐组件备份恢复和写后验证证据。
- 禁止推断: 不得把 Pico-ITX、第三方实现或 K3 通用示例中的包名、设备身份、分区偏移、介质容量、命令、工具版本或恢复步骤外推到 CoM260；不得把 RAM 启动成功视为持久化写入安全，也不得仅由文件名、分区名或工具名称裁决实际 payload、目标位置或兼容性。
- 解除条件: 取得适用于同一 CoM260 产品修订的官方镜像包与 manifest、只读板型/启动设备/目标介质/分区信息、精确工具版本与下载入口、逐组件备份和可逆恢复步骤，并在相同软硬件版本上形成身份确认、写入结果、重启后状态与失败恢复的分层证据。
- 影响主题: `docs/boot/` 的三篇 MS12 正文；与 G7、G12 职责互斥：G7 负责默认目标 DTS 的唯一映射，G12 负责存储控制器专属的板级映射、运行路径与恢复闭包，G15 负责跨启动路径的镜像包、工具、写入操作与恢复契约，不替代 G7 或 G12。
- 状态变更记录: 2026-09-14 由 MS12 Iteration 003 新增，状态 `open`；本 change 只汇总三篇 boot 正文的部署、刷写与恢复未知项，不解除任何子项。

---

## 缺口状态汇总

| 编号 | 分类 | 状态 | 最近核对日期 |
| --- | --- | --- | --- |
| G1 | 范围盘点 | open | 2026-09-02 |
| G2 | 范围盘点 | open | 2026-09-02 |
| G3 | 硬件事实 | partial | 2026-09-09 |
| G4 | 硬件事实 | partial | 2026-09-09 |
| G5 | 硬件事实 | partial | 2026-09-09 |
| G6 | 硬件事实 | open | 2026-09-09 |
| G7 | 硬件事实 | open | 2026-09-09 |
| G8 | 硬件事实 | open | 2026-09-08 |
| G9 | 硬件事实 | open | 2026-09-08 |
| G10 | 硬件事实 | open | 2026-09-08 |
| G11 | 协议/接口 | open | 2026-09-10 |
| G12 | 硬件事实 | open | 2026-09-11 |
| G13 | 硬件事实与协议/接口 | open | 2026-09-12 |
| G14 | 硬件事实与协议/接口 | open | 2026-09-12 |
| G15 | 镜像部署与恢复 | open | 2026-09-14 |

## 缺口与 source-coverage 的对应

- G1、G2 对应 `source-coverage.md` 中观察日期 2026-09-02 的 38 行；未观察子树以「未登记」处理。
- G3、G4、G5、G6 对应 `source-coverage.md` 中标记为 `active` 但未提供寄存器级描述的 R04/R05 资料行。
- G3 在 2026-09-07 由 Iteration 001 / T11 部分解除, 仍需 PHY 实例、MMIO、寄存器、Kit 实物连接器进一步证据。
- G4 在 2026-09-09 按 MS05 已观察的官方 `k3.dtsi` 静态拓扑修正汇总为 `partial`；[`k3-gmac-dma-irq.md`](../network/k3-gmac-dma-irq.md) 只补充固定 revision GMAC handler 边界，source→EID、target hart、affinity 和真板 delivery 仍未解除。
- G5 在 2026-09-09 由 Iteration 001 / T3 部分解除，由 `open` 调整为 `partial`；[`k3-cache-pma-address-translation.md`](../dma/k3-cache-pma-address-translation.md) 承载通用机制、三类对象的软件路径和未知边界；K3 cache/coherency、PMA/PBMT 真板效果、默认 Kit IOMMU 与 device address 映射仍未解除；对应 `source-coverage.md` 中 4 行 RISC-V PMA、Svpbmt、FENCE 与 IOMMU 规范入口。
- G6 在 2026-09-09 增加 K3 Linux GMAC glue driver 来源和 [`k3-gmac-dma-irq.md`](../network/k3-gmac-dma-irq.md)；glue 层不含 DWMAC descriptor/IRQ programmer reference，状态保持 `open`。
- G7 由 Iteration 001 / T11 新增, 对应 `source-coverage.md` 中 T7 / T8 / T9 三行(linux-6.18 仓库 `k3-br-v1.0.y` 分支目录、docs-buildroot boot.md、image.md)。
- G8、G9、G10 由 Iteration 001 / T3 新增, 对应 `source-coverage.md` 中 T11(05-UART.md)、T12(`k3.dtsi` raw)、T13(`k3-rdomain.dtsi` raw)、T14(8250.yaml)、T15(8250_of.c) 五行; 与 [`docs/serial/com260-uart.md`](../serial/com260-uart.md) §3 / §5 / §10.1 / §10.4 / §10.5 交叉引用。
- G11 由 Iteration 001 新增, 对应 [`k3-rpc-ring-notification.md`](../amp/k3-rpc-ring-notification.md) U1-U5 五项子字段; `source-coverage.md` 不新增 URL 行 (G11 登记的是协议闭包, 不在官方来源覆盖表内); 与 G4 / G5 / G7 互不重复, 详见 G11 影响主题字段的职责互斥说明。
- G12 由 Iteration 002 新增, 对应 [`k3-qspi-spi-sdhci.md`](../storage/k3-qspi-spi-sdhci.md) U1–U5 + [`k3-ufs.md`](../storage/k3-ufs.md) U1–U5 共十项子字段; `source-coverage.md` 不新增 URL 行 (Iteration 000 / 001 已把四个存储官网入口 + UFS docs-buildroot supporting row 设为当前职责, URL 总数仍为 70); 与 G5 / G7 互不重复, G5 负责 DMA/cache/coherency、G7 负责默认目标 DTS, G12 负责存储设备专属 binding/programmer reference、板级介质差异、消费层、完成模型、性能与错误恢复闭包, 详见 G12 影响主题字段的职责互斥说明。
- G13 由 MS10 Iteration 003 新增，对应 [`k3-i2c-and-usb.md`](../buses/k3-i2c-and-usb.md)、[`k3-pcie-and-can.md`](../buses/k3-pcie-and-can.md) 与 [`k3-ethercat.md`](../buses/k3-ethercat.md) 的未知项；五个总线官网入口已在 Iteration 000 转为当前职责，`source-coverage.md` 不新增 URL 行，总数仍为 70。G7 负责默认目标 DTS，G13 负责总线设备专属的板级映射、软件运行路径和恢复闭包。
- G14 由 MS11 Iteration 003 新增，对应 [`k3-gpio-pwm-ir.md`](../peripherals/k3-gpio-pwm-ir.md)、[`k3-audio.md`](../peripherals/k3-audio.md) 与 [`k3-wdt-rtc.md`](../peripherals/k3-wdt-rtc.md) 的 13 组未知项；六个通用外设官网入口已在 Iteration 000 转为 current/active，`source-coverage.md` 不新增 URL 行，总数仍为 70。G4/G5/G7 分别负责 IRQ delivery、DMA/cache/IOMMU 和默认目标 DTS，G14 负责通用外设专属的板级映射、运行路径和恢复闭包。
- G15 由 MS12 Iteration 003 新增，对应 [`k3-image-build-and-artifacts.md`](../boot/k3-image-build-and-artifacts.md)、[`k3-ram-boot-and-fastboot.md`](../boot/k3-ram-boot-and-fastboot.md) 与 [`k3-flashing-and-recovery.md`](../boot/k3-flashing-and-recovery.md) 的镜像包、工具、写入与恢复未知项；Iteration 000 已新增 6 个 K3-Ubuntu-Images 精确文件 URL，`source-coverage.md` 当前总数为 76。G7 负责默认目标 DTS，G12 负责存储控制器专属闭包，G15 负责跨路径的部署、刷写与恢复操作闭包。
- R06 的 15 个 `deferred` URL 暂不展开到本表；如后续进入聚合 change，再决定是否新增对应 G 条目。
- 镜像内部组成(`bootfs.img` / `rootfs.ext4` 容量与 partition 字段)的局部未知项见 `com260-image-and-dts.md` §6.4, 不在本 G 表登记(超出 T12 范围, 见 §8 Non-goals)。
