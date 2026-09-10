> 来源: https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs/k3_ds.md（源端修订: unknown；观察日期: 2026-09-07）；https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/eco/k3_com260/com260_ds.md（源端修订: unknown；观察日期: 2026-09-07）；https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/eco/k3_com260/com260_hw_resources.md（源端修订: unknown；观察日期: 2026-09-07）；https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/09-GMAC.md（源端修订: unknown；观察日期: 2026-09-02）；https://raw.githubusercontent.com/spacemit-com/docs-buildroot/main/zh/k3_buildroot/device/peripheral_driver/09-GMAC.md（源端修订: unknown；观察日期: 2026-09-09）；https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-07）；https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260.dtsi（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-07）；https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260.dts（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-07）；https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260_kit_v02.dts（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-07）

# CoM260 GMAC、MDIO、PHY 与 RGMII

> 范围: K3 SoC、CoM260 模组与 CoM260 DTS 候选中的 GMAC/PHY 静态链路。
> 不覆盖: MAC/MTL/DMA descriptor、IRQ handler、异步网络接口、EtherCAT/TSN 协议和网络栈。
> 证据边界: 官网是唯一权威入口；SpacemiT 官方 GitHub 内容只作交叉验证；固定 revision 第三方材料单独标注。

## 目录

- [1. 结论边界](#1-结论边界)
- [2. SoC、模组与 Kit 分层](#2-soc模组与-kit-分层)
- [3. CoM260 DTS 候选](#3-com260-dts-候选)
- [4. `eth1` 平台资源](#4-eth1-平台资源)
- [5. MDIO、PHY 与 RGMII 静态链](#5-mdio-phy-与-rgmii-静态链)
- [6. 初始化依赖与运行时边界](#6-初始化依赖与运行时边界)
- [7. 来源差异](#7-来源差异)
- [8. 未知项](#8-未知项)
- [9. 相邻主题与来源导航](#9-相邻主题与来源导航)

## 1. 结论边界

当前材料能建立以下静态关系：

```text
K3 SoC GMAC capability
  → CoM260 模组引出的 GMAC1 / PHY1 信号
  → CoM260 DTS 候选中的 &eth1
  → clock / reset / pinctrl / APMU
  → phy-mode = "rgmii"
  → MDIO Clause 22 bus
  → PHY address 1
```

这条关系不等于默认 Kit 配置已经确定。`k3_com260.dts`、`k3_com260_kit_v02.dts` 与其他 CoM260 候选并存；G7 尚未把用户指南中的产品版本唯一映射到其中一个顶层 DTS。

PHY compatible `ethernet-phy-id001c.c916` 能标识已观察 DTS 中的 PHY ID。它不能单独证明商品型号、封装、strap、EEPROM、扩展寄存器或内部 delay 配置。运行时 link、自协商和 cable unplug/replug 也不能由静态 DTS 得出。

## 2. SoC、模组与 Kit 分层

| 层级 | 已观察内容 | 证据等级 | 边界 |
| --- | --- | --- | --- |
| K3 SoC | 4 路 GMAC；支持 RGMII、RMII、MII；能力概述包含 TSN | 交叉验证 | SoC 路数不等于 CoM260 全部引出或启用 |
| CoM260 模组 | 引出 1 路 GPHY 芯片与 1 路 GMAC 通道（PHY1）；金手指包含 GMAC1 RX/TX、RX_CLK、MDC、MDIO、INT_N 等信号 | 交叉验证 | 模组资料未闭合 PHY 实物型号、MDIO 寄存器和 Kit 连接器布局 |
| CoM260 shared DTS | `k3_com260.dtsi` 配置 `&eth1` 的 1000 Mbps、RGMII、PHY reset 与 delay-line | 交叉验证 | shared DTS 被多个顶层 DTS 继承，不代表某个顶层候选已被选中 |
| CoM260 基础款 DTS | `k3_com260.dts` 为 `&eth1` 增加 `gmac1_phy` MDIO 子节点 | 交叉验证 | 基础款名称不等于 CoM260 Kit 默认目标 |
| CoM260 Kit V02 DTS | `k3_com260_kit_v02.dts` 为 `&eth1` 增加 FIFO、TSO、store-and-forward DMA 字段，并使用 `rgmii1` PHY label | 交叉验证 | `v02` 与用户指南材料中的产品版本 `v03` 未唯一对应 |

SoC 和模组能力的详细来源分别见 [`k3-soc-overview.md`](../platform/k3-soc-overview.md) 与 [`com260-board-resources.md`](../platform/com260-board-resources.md)。本文件只保留建立 GMAC/PHY 静态链所需的分层结果。

## 3. CoM260 DTS 候选

### 3.1 共享配置

`k3_com260.dtsi` 中已观察的 `&eth1` 字段如下：

| 字段 | 值 | 作用范围 | 证据等级 |
| --- | --- | --- | --- |
| `max-speed` | `1000` | 该 shared DTS 的 GMAC1 最大速率输入 | 交叉验证 |
| `phy-mode` | `rgmii` | MAC 与 PHY 的接口模式 | 交叉验证 |
| `snps,reset-gpios` | GPIO bank 1、pin 5、低有效 | PHY reset 输入 | 交叉验证 |
| `snps,reset-delays-us` | `<0 20000 100000>` | reset 前、assert、deassert 三段延时输入 | 交叉验证 |
| `spacemit,clk-tuning-enable` | present | 启用 K3 GMAC 时钟调相路径 | 交叉验证 |
| `spacemit,clk-tuning-by-delayline` | present | 选择 delay-line 调相 | 交叉验证 |
| `spacemit,tx-phase` | `47` | 该 DTS 的 TX phase 输入 | 交叉验证 |
| `spacemit,rx-phase` | `53` | 该 DTS 的 RX phase 输入 | 交叉验证 |

这些值只能用于描述该 shared DTS。G7 未解除前，不把 phase 47/53 或 reset GPIO 写成所有 CoM260 Kit 硬件修订的固定参数。

### 3.2 顶层变体

| 字段 | `k3_com260.dts` | `k3_com260_kit_v02.dts` | 证据等级 |
| --- | --- | --- | --- |
| model | `SpacemiT K3 Com260` | `SpacemiT K3 Com260 Kit V02` | 交叉验证 |
| GMAC node | `&eth1` | `&eth1` | 交叉验证 |
| PHY label | `gmac1_phy` | `rgmii1` | 交叉验证 |
| PHY compatible | `ethernet-phy-id001c.c916` + `ethernet-phy-ieee802.3-c22` | `ethernet-phy-id001c.c916` | 交叉验证 |
| PHY address | `1` | `1` | 交叉验证 |
| Realtek properties | `aldps-enable`、`clkout-disable`、`link-poll`、`wakeup-source` | 未在已观察节点列出 | 交叉验证 |
| FIFO | 沿用上游节点或 shared 配置；本层未新增 | TX/RX 均为 8192 bytes | 交叉验证 |
| data-plane hints | 本层未新增 | `snps,tso`、`snps,force_sf_dma_mode` | 交叉验证 |

label 不改变 PHY 地址本身，但表明两个顶层 DTS 的引用结构不同。未在某个变体中列出的属性只表示该层没有观察到相同覆写，不能直接推断运行时禁用。

### 3.3 未直接展开的候选

官方 Linux DTS 目录还列出 `k3_com260_ifx.dts`、`k3_com260_ifx2.dts`、`k3_com260_ifx_tq.dts` 和 `k3_com260_tq.dts`。这些文件未在既有官方来源观察记录中展开字段，因此本文件不使用其值补齐基础款或 Kit V02。

固定 revision 第三方 IFX DTS 可用于理解驱动消费字段，但不能解除上述官方候选的未观察状态。

## 4. `eth1` 平台资源

### 4.1 K3 SoC 节点与 CoM260 覆写

既有官方 DTS 观察结果把 CoM260 网络口关联到 `&eth1`。`k3.dtsi` 提供 SoC 级节点及 platform provider 引用，CoM260 shared/top-level DTS 再提供板级 PHY、RGMII 和 FIFO 等字段。下表中的节点与 provider 关系来自这些官方 DTS；精确 MMIO 窗口和 wired source 数值目前只由固定 revision 第三方 IFX DTS 支撑。

| 资源维度 | 已观察关系 | 证据等级 | 未确认边界 |
| --- | --- | --- | --- |
| compatible | `spacemit,k3-gmac`、`snps,dwmac-5.10a` | 交叉验证 | 精确 glue 行为仍需官方 driver/binding 核对；G6 保留完整 programmer reference 缺口 |
| MMIO | `eth1` base `0xcac82000`，窗口 `0x2000` | 固定 revision 第三方 | 不从通用 DWMAC 默认地址推断其他实例 |
| IRQ | AP APLIC wired source 133 | 固定 revision 第三方 | source→EID、target hart 与真板 delivery 由 G4 保留 |
| clock/reset | consumer 引用 APMU clock/reset provider；第三方反编译 DTS 使用的 phandle/数字 ID 未独立还原为官方符号名 | 交叉验证 + 未知项 | provider 内部寄存器、符号 ID 与实际时序不由本文件补值 |
| pinctrl | `eth1` 使用 GMAC/RGMII 相关 pin group | 交叉验证 | 不由第三方 IFX pin group 推定所有官方顶层变体 |
| APMU glue | K3 专属字段承载 interface mode、control offset 和 delay-line offset | 固定 revision 第三方 + 官方 DTS 字段线索 | 官方 glue driver 本轮网络不可访问，不能关闭 G6 |

平台 provider 的 AP/APBC/APMU 分域见 [`k3-platform-control.md`](../platform/k3-platform-control.md)。GMAC wired IRQ 的上游拓扑见 [`k3-interrupt-and-time.md`](../interrupts/k3-interrupt-and-time.md)。

### 4.2 固定 revision 第三方配置

tgoskits `19219411d` 使用的 IFX DTS 把一个启用的 GMAC 节点描述为：

- `ethernet@cac82000`，窗口大小 `0x2000`；
- compatible 为 `spacemit,k3-gmac` 与 `snps,dwmac-5.10a`；
- RGMII、1000 Mbps、TX/RX FIFO 8192 bytes；
- phase 47/53、PHY reset GPIO 与三段 reset delay；
- MDIO Clause 22 PHY address 1。

证据等级为“固定 revision 第三方”。这些字段说明该驱动与该 DTS 如何配合，不证明官方默认 Kit DTS、实物器件或所有硬件修订使用同一值。

## 5. MDIO、PHY 与 RGMII 静态链

### 5.1 绑定关系

```text
&eth1
  ├─ phy-mode = "rgmii"
  ├─ phy-handle → gmac1_phy 或 rgmii1
  └─ mdio
       └─ ethernet-phy@1 / phy@1
            ├─ reg = 1
            └─ compatible = ethernet-phy-id001c.c916
```

- `phy-handle` 把 MAC consumer 与 PHY node 连接起来。（交叉验证）
- `reg = 1` 表示已观察 DTS 中的 Clause 22 PHY address 1。（交叉验证）
- `phy-mode = "rgmii"` 表示 MAC/PHY 使用 RGMII 接口。（交叉验证）
- reset GPIO 和 reset delays 是 PHY 初始化输入；它们不证明当前引脚在运行时已经正确翻转。（交叉验证）
- phase 47/53 是 K3 delay-line 配置输入；它们不证明 PHY internal delay 是否另行启用。（交叉验证 + 未知项）

### 5.2 PHY 标识边界

PHY ID `001c.c916` 的 OUI/型号解释需要 PHY datasheet 或等效厂商资料。本仓库当前只把它记录为 DTS compatible 字符串，不将第三方报告中的 `RTL8211F` 名称提升为官方板级事实。

基础款 DTS 出现 `realtek,aldps-enable`、`realtek,clkout-disable`、`realtek,link-poll` 和 `wakeup-source`。这些属性说明该 DTS 请求的驱动行为；没有 binding、PHY datasheet 或运行证据时，不扩写其寄存器序列或电气效果。

### 5.3 RGMII delay 边界

当前 shared DTS 同时使用 `phy-mode = "rgmii"` 和 K3 `spacemit,*phase`/delay-line 字段。静态证据表明 SoC glue 承担一组调相输入，但没有证明 PHY 端 internal delay 的启用状态。因而本文件不把 RGMII 时钟偏移归因于 MAC 或 PHY 的单一一侧，也不计算 phase 数值对应的时间。

## 6. 初始化依赖与运行时边界

固定 revision 第三方驱动的字段消费关系提供以下顺序线索：

```text
pinctrl
  → APMU bus clock / reset
  → interface mode / delay-line
  → PHY GPIO reset
  → GMAC MMIO
  → MDIO scan / PHY negotiation
  → MAC speed and duplex input
```

证据等级为“固定 revision 第三方”。该顺序不替代官方初始化规范，不能关闭 G6。

以下运行时行为未由本项目验证：

- PHY ID 是否能在每次启动稳定读取；
- 自协商是否完成及最终 speed/duplex；
- cable unplug/replug 后是否重新协商；
- reset delay 是否满足所有 Kit 修订；
- RGMII phase 是否在温压变化和不同线长下成立；
- wired IRQ 是否投递到预期 hart；
- DHCP、ping、吞吐、丢包和恢复结果。

第三方报告中的 link、DHCP 与 ping 只属于其历史环境。后续驱动规划可引用该报告作为测试线索，不能把结果改写成本项目验证。

## 7. 来源差异

| 差异 | 已观察情况 | 处理 |
| --- | --- | --- |
| 官网与官方 GitHub | 官网页面仍可能只暴露 SPA 壳；GitHub 对应页可读 | 官网保持权威入口，GitHub 标为交叉验证 |
| 基础款与 Kit V02 | PHY label、Realtek 属性、FIFO/TSO/DMA 字段不同 | 分变体记录，不静默合并 |
| Kit V02 与产品版本 | DTS 名称含 `v02`，用户指南材料出现产品版本 `v03` | G7 保持 open，不选默认 DTS |
| PHY ID 与商品型号 | DTS 提供 `001c.c916`，第三方报告称 RTL8211F | 只保留 compatible；商品型号保持未知 |
| 静态 DTS 与运行结果 | DTS 提供配置输入，第三方报告提供历史 link/DHCP/ping | 分开证据等级，不相互替代 |
| 通用 DWMAC 与 K3 glue | DWMAC compatible 提供 IP 家族线索，K3 另有 APMU/delay-line 字段 | 不用通用默认值补 K3 专属寄存器 |

## 8. 未知项

### U1. 默认 CoM260 Kit DTS

- 当前证据: 官方目录有 7 个 CoM260 命名候选；基础款、shared base、Kit V02 和 K3 SoC DTS 已观察。Kit V02 名称与用户指南材料中的产品版本未唯一对应。
- 禁止推断: 不由文件名、model、compatible 或第三方正在使用的 IFX DTS 选择默认目标。
- 解除条件: 官方 user guide/defconfig/board 标识明确映射，或原理图下载制品给出可核对的板级身份。
- 影响主题: 本文件的变体归属、后续 GMAC DMA/IRQ 文档和 G7。

### U2. PHY 实物型号与板级连接

- 当前证据: DTS compatible 为 `ethernet-phy-id001c.c916`，地址为 1；模组资料说明 PHY1/GMAC1 信号引出。
- 禁止推断: 不由 PHY ID 或第三方报告确定商品型号、封装、strap、EEPROM、LED、电压或 Kit 连接器位置。
- 解除条件: 取得 CoM260 原理图/BOM、PHY datasheet 或官方 hardware resources 中的明确器件字段。
- 影响主题: MDIO/PHY 初始化、RGMII delay、reset/ref clock 和 G3。

### U3. RGMII delay 的 MAC/PHY 分工

- 当前证据: shared DTS 使用普通 `rgmii` mode，并提供 K3 delay-line enable 与 phase 47/53。
- 禁止推断: 不把 phase 码换算成时间，不假定 PHY internal delay 已开启或关闭，不把该值推广到全部板型。
- 解除条件: 官方 K3 GMAC glue/binding、PHY binding/datasheet与对应板级时序共同说明两侧 delay 配置。
- 影响主题: link 稳定性、speed 切换和 G3/G6。

### U4. K3 GMAC 专属寄存器与初始化时序

- 当前证据: 09-GMAC、DTS 与固定 revision 第三方代码能说明字段和软件路径；本轮无法通过网络直接打开候选官方 glue driver/binding。
- 禁止推断: 不用通用 DWMAC 寄存器、tgoskits 常量或历史调试结果替代 K3 programmer reference。
- 解除条件: 取得 SpacemiT GMAC programmer reference，或直接核对官方 Linux/U-Boot K3 glue、binding 和寄存器定义。
- 影响主题: 本文件 platform resource 边界、后续 `k3-gmac-dma-irq.md` 和 G6。

### U5. 运行时 PHY 与 IRQ 行为

- 当前证据: DTS 提供静态 PHY/IRQ 输入；固定 revision 第三方材料记录特定环境的 link 和网络结果。
- 禁止推断: 不由静态节点声明自协商、重协商、IRQ target hart、丢包或恢复已通过。
- 解除条件: 在唯一映射的 CoM260 Kit 上取得 PHY/link、APLIC/IMSIC delivery 与网络数据面的可重复运行结果。
- 影响主题: G4、后续 GMAC DMA/IRQ 文档和驱动实施计划。

## 9. 相邻主题与来源导航

- SoC 能力: [`k3-soc-overview.md`](../platform/k3-soc-overview.md)。
- CoM260 模组与 Kit 资源: [`com260-board-resources.md`](../platform/com260-board-resources.md)。
- DTS 候选与版本映射: [`com260-image-and-dts.md`](../boot/com260-image-and-dts.md)。
- AP platform provider: [`k3-platform-control.md`](../platform/k3-platform-control.md)。
- APLIC/IMSIC 静态拓扑: [`k3-interrupt-and-time.md`](../interrupts/k3-interrupt-and-time.md)。
- DMA ownership 与 cache/IOMMU 边界: [`k3-dma-and-memory-ownership.md`](../dma/k3-dma-and-memory-ownership.md)、[`k3-cache-pma-address-translation.md`](../dma/k3-cache-pma-address-translation.md)。
- G3–G7: [`known-gaps.md`](../reference/known-gaps.md)。
- 来源元数据: [`source-coverage.md`](../reference/source-coverage.md)。
- DWMAC5 数据面与 IRQ: 由本 change 的后续 Iteration 创建 `k3-gmac-dma-irq.md`；当前不建立占位文件。

本文件没有展开 EtherCAT、TSN 协议、网络栈或异步 NIC。`k3_com260.dtsi` 中 `ec_master` 对 `eth1` 的引用只作为后续物理通路边界，不构成这些主题的正文或运行结论。
