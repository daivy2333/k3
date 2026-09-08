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
  仍未知: PHY 实物型号(RTL8211F 还是 RTL8211FD 还是其他)、PHY 在 Kit 板上的连接器位置、MDIO/MMIO 寄存器、CoM260 Kit 是否为单 PHY 单网口、reset 与 ref clock 在 Kit 原理图中的具体实现。
- 禁止推断: 不得基于 Linux 驱动行为或 DWMAC 默认值反推 CoM260 实际硬件；不得假定 GMAC 实例与 Pico 板相同；不得由 PHY 字符串 `ethernet-phy-id001c.c916` 推定 PHY 寄存器布局或 EEPROM 加载流程；不得由 RGMII delayline 值 47/53 推定 Kit 实际值。
- 解除条件:
  - 取得 CoM260 模组 hardware resources 中关于 GMAC/PHY 的具体字段（实例号、MMIO、PHY 型号、MDIO/RGMII 参数）；
  - 或取得 CoM260 DTS（device tree）源文件并对照 compatible、reg、phy-handle、phy-mode；
  - 形成 R04 子条目的修订记录或新增 R 登记。
  - 仍需补: 取得 CoM260 Kit 原理图中 PHY 实物型号、PHY 板上位置、MDIO/MMIO 寄存器手册、CoM260 实物网口数量与连接器布局。
- 影响主题: `docs/network/`（GMAC、PHY、MDIO、RGMII）；间接影响 `docs/platform/`（clock、reset、pinctrl）。
- 状态变更记录: 2026-09-07 由 `open` 调整为 `partial`, 由 Iteration 001 / T11 直接打开 linux-6.18 `k3-br-v1.0.y` 分支 `k3_com260.dts` / `k3_com260_kit_v02.dts` / `k3_com260.dtsi` 提供 PHY 标识与 GMAC 模式; 仍存 PHY 实例、MMIO、寄存器、Kit 实物连接器未解。

## G4. AIA / APLIC / IMSIC 地址、IRQ domain 与 hart delivery

- 分类: 硬件事实
- 当前证据: R05 的 16-Clock.md、Reset.md、Timer.md 涉及 K3 clock/reset/timer 但未公开 APLIC 与 IMSIC 的 MMIO 地址、IRQ domain 拓扑、hart routing 规则、mask/ack/complete 协议细节。
- 禁止推断: 不得用 RISC-V AIA 标准行为直接等同 K3 实现；不得假定 K3 APLIC/IMSIC 寄存器布局与 SiFive 或其他厂商一致。
- 解除条件:
  - 取得 K3 SoC 公开寄存器手册（programmer reference 或 datasheet 寄存器章节）；
  - 或在 R08 的 linux-6.18 仓库定位 K3 APLIC/IMSIC 设备树与驱动初始化源码；
  - 形成 address、IRQ domain、hart/context、delivery 等事实条目。
- 影响主题: `docs/interrupts/`（AIA、APLIC、IMSIC、timer）；间接影响 `docs/serial/` 与 `docs/network/`（驱动 IRQ 接入）。

## G5. DMA coherency、IOMMU、cache line 与 barrier 规则

- 分类: 硬件事实
- 当前证据: R05 的 21-DMA.md 仅说明 K3 SDK 中 DMA 驱动的通用配置；未公开 K3 SoC 的 cache 一致性模型、IOMMU 是否存在、DMA 地址宽度、descriptor 与数据缓冲区的 ownership 转换规则、cache line 大小、barrier 要求。
- 禁止推断: 不得用通用 RISC-V 行为或 Linux DMA API 行为等同 K3 实际；不得假定 K3 默认全 coherent 或全 non-coherent。
- 解除条件:
  - 取得 K3 SoC 公开寄存器手册中 cache、MMU、IOMMU 章节；
  - 或在 R08 的 linux-6.18 仓库定位 K3 IOMMU 与 DMA 一致性相关 patch；
  - 形成 coherency model、IOMMU presence、barrier list 与 ownership 转换条目。
- 影响主题: `docs/dma/`；间接影响 `docs/network/`（GMAC descriptor、buffer）、`docs/storage/`（UFS/SDHC）。

## G6. K3 GMAC 寄存器、descriptor 与 interrupt ack 的 programmer reference

- 分类: 硬件事实
- 当前证据: R04 的 k3_ds.md、root_overview.md 是产品层与 overview 层资料；R05 的 09-GMAC.md 是驱动使用说明；均不替代 GMAC 寄存器、descriptor ring、interrupt cause/mask/ack 寄存器级描述。
- 禁止推断: 不得以 Linux DWMAC 行为或 Synopsys DWMAC 通用寄存器定义反推 K3 GMAC；不得假定 ack/cause 寄存器布局等同于上游驱动。
- 解除条件:
  - 取得 SpacemiT 官方 GMAC 寄存器手册（programmer reference）；
  - 或在 R08 的 linux-6.18 仓库定位 K3 GMAC 设备树与驱动的寄存器偏移表；
  - 形成寄存器偏移、复位值、interrupt cause/mask/ack 与 descriptor 字段定义条目。
- 影响主题: `docs/network/`（GMAC 寄存器与 descriptor、interrupt ack）；间接影响 `docs/dma/`（descriptor buffer ownership）。

## G7. CoM260 Kit 默认目标 DTS 未唯一映射

- 分类: 硬件事实
- 当前证据: Iteration 001 / T11 直接打开 linux-6.18 仓库 `k3-br-v1.0.y` 分支 `arch/riscv/boot/dts/spacemit/` 目录, 列出 7 个 CoM260 命名候选(6 .dts + 1 .dtsi 共享 base); `k3_com260.dts`(`model = "SpacemiT K3 Com260"`)、`k3_com260_kit_v02.dts`(`model = "SpacemiT K3 Com260 Kit V02"`)、`k3_com260.dtsi`、`k3.dtsi` 四个文件已被直接打开, 其中 `k3_com260_kit_v02.dts` 名称与 Kit 最接近, 但其名称中的 `v02` 与 com260_user_guide.md V2.0 资料下载部分列出的 CoM260 产品版本 `K3-CoM260_P1_LP5315B_32X2_v03_20260312` 的 `v03` 不一致; 其余 4 个候选(`k3_com260_ifx.dts`、`k3_com260_ifx2.dts`、`k3_com260_ifx_tq.dts`、`k3_com260_tq.dts`)未直接打开, 字段全部留 `未知项`。
- 禁止推断: 不得由"名称最接近"推定 `k3_com260_kit_v02.dts` 即为 CoM260 Kit 默认目标 DTS; 不得由 `compatible` 字符串推定硬件 layout 差异; 不得由 `model` 字符串推定。
- 解除条件:
  - 取得 CoM260 Kit 原理图(com260_hw_resources.md 下载制品)中 DTS 路径或 board 标识;
  - 或 com260_user_guide.md 后续修订明示对应表;
  - 或 buildroot defconfig 包含 `BR2_TARGET_KERNEL_DTB` 明确指向某一 DTS;
  - 解除时需同时更新 `com260-image-and-dts.md` §3 / §4 / §5, 并登记 R04 或 R08 子条目修订。
- 影响主题: `docs/platform/`(平台资源映射)、`docs/network/`(GMAC/PHY 介质归属)、MS07(EtherCAT 物理通路)。
- 状态变更记录: 2026-09-07 由 Iteration 001 / T11 新增, 状态 `open`。

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

---

## 缺口状态汇总

| 编号 | 分类 | 状态 | 最近核对日期 |
| --- | --- | --- | --- |
| G1 | 范围盘点 | open | 2026-09-02 |
| G2 | 范围盘点 | open | 2026-09-02 |
| G3 | 硬件事实 | partial | 2026-09-07 |
| G4 | 硬件事实 | open | 2026-09-02 |
| G5 | 硬件事实 | open | 2026-09-02 |
| G6 | 硬件事实 | open | 2026-09-02 |
| G7 | 硬件事实 | open | 2026-09-07 |
| G8 | 硬件事实 | open | 2026-09-08 |
| G9 | 硬件事实 | open | 2026-09-08 |
| G10 | 硬件事实 | open | 2026-09-08 |

## 缺口与 source-coverage 的对应

- G1、G2 对应 `source-coverage.md` 中观察日期 2026-09-02 的 38 行；未观察子树以「未登记」处理。
- G3、G4、G5、G6 对应 `source-coverage.md` 中标记为 `active` 但未提供寄存器级描述的 R04/R05 资料行。
- G3 在 2026-09-07 由 Iteration 001 / T11 部分解除, 仍需 PHY 实例、MMIO、寄存器、Kit 实物连接器进一步证据。
- G7 由 Iteration 001 / T11 新增, 对应 `source-coverage.md` 中 T7 / T8 / T9 三行(linux-6.18 仓库 `k3-br-v1.0.y` 分支目录、docs-buildroot boot.md、image.md)。
- G8、G9、G10 由 Iteration 001 / T3 新增, 对应 `source-coverage.md` 中 T11(05-UART.md)、T12(`k3.dtsi` raw)、T13(`k3-rdomain.dtsi` raw)、T14(8250.yaml)、T15(8250_of.c) 五行; 与 [`docs/serial/com260-uart.md`](../serial/com260-uart.md) §3 / §5 / §10.1 / §10.4 / §10.5 交叉引用。
- R06 的 15 个 `deferred` URL 暂不展开到本表；如后续进入聚合 change，再决定是否新增对应 G 条目。
- 镜像内部组成(`bootfs.img` / `rootfs.ext4` 容量与 partition 字段)的局部未知项见 `com260-image-and-dts.md` §6.4, 不在本 G 表登记(超出 T12 范围, 见 §8 Non-goals)。
