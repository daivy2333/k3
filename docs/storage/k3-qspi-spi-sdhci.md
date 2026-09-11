> 来源: https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs/k3_ds.md, https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/07-QSPI.md, https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/SPI.md, https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/08-SDHC.md（源端修订: unknown；观察日期: 2026-09-02）；supporting: https://github.com/spacemit-com/docs-chip/blob/main/zh/key_stone/k3/k3_docs/k3_ds.md（源端修订: 2026-08-25 V1.8；观察日期: 2026-09-07），https://github.com/Oveln/tgoskits（revision: `19219411d5dc1515496f910d04c93da12ee95be4`；本地适用文件: `os/StarryOS/configs/board/spacemit-k3-com260-ifx.dts`, `drivers/blk/k3-sdhci/src/lib.rs`, `vendor_ext.rs`）

# K3 QSPI、SPI 与 SD/eMMC 控制器

本文从存储控制器视角整理 K3 的 QSPI、普通 SPI 与 SD/eMMC/SDHCI。启动阶段和镜像布局以[启动链](../boot/com260-boot-chain.md)和[镜像与 DTS](../boot/com260-image-and-dts.md)为准；通用 DMA、cache 和所有权规则以[DMA 与内存所有权](../dma/k3-dma-and-memory-ownership.md)和[cache/PMA/地址转换](../dma/k3-cache-pma-address-translation.md)为准。本文只补充各控制器特有的静态资源、路径和边界。

## 1. 范围与证据等级

- **官方事实**：K3 datasheet 与 SpacemiT 社区入口描述的 SoC 能力。社区驱动页正文尚未直接取得，不能据页面标题补写驱动行为。
- **板级事实**：[CoM260 板级资源](../platform/com260-board-resources.md)确认板载 SPI Flash、TF 卡，并记录介质规格缺口。
- **固定第三方静态证据**：tgoskits 的 `spacemit-k3-com260-ifx.dts` 只描述 IFX 变体，受 G7 默认 DTS 未唯一映射约束。
- **固定第三方软件行为**：tgoskits `k3-sdhci` crate 是 portable core；它能解释 PHY 与 tuning 状态机，但不能证明 CoM260 当前 board profile 已接入。
- **未验证**：本项目没有执行真板 I/O、刷写、性能、热插拔或错误注入测试。

QSPI 是面向 SPI memory 的控制器，普通 SPI 是 message/transfer 控制器；二者不能仅因都使用串行时钟和数据线而互换。SD 卡与 eMMC 可共用 SDHCI 控制器家族，但介质连接、removable 属性、总线宽度、电压和 timing 模式不同，也不能视为同一设备。

## 2. QSPI

### 2.1 能力与板级可达性

K3 datasheet 把 Quad-SPI 描述为支持 XIP 与 Page 模式、1/2/4 线传输、13.25–102 MHz，并支持 NOR 与 NAND。这里的数值是 SoC 能力，不证明板载器件支持全部模式或最高频率。

CoM260 板级资料确认存在 SPI Flash，但当前证据未唯一确认具体器件、容量、总线宽度、最高安全频率或它采用 NOR/NAND 哪一种组织。因此“有板载 SPI Flash”不能推出“可使用所有 QSPI 能力”。

### 2.2 静态资源

固定第三方 IFX DTS 含一个 `compatible = "spacemit,k3-qspi"` 的节点，并为候选板配置提供 MMIO 与映射窗口、IRQ、clock/reset 和 pinctrl。该节点没有 `dmas` 或 `dma-names`；这既不能证明 QSPI 运行时使用 DMA，也不能证明它只使用 PIO。已有属性只描述该 DTS 变体，不能作为默认 CoM260 Kit 的唯一地址或运行时配置。

### 2.3 启动与数据路径

既有启动基线把 SPI NOR、SPI NAND 列为 K3 启动候选介质。QSPI 控制器视角可分为两类：

1. XIP 让处理器从映射窗口取指或读取，但当前材料没有闭合 CoM260 的窗口地址、cache 属性和实际启用条件。
2. Page 模式通过命令、地址和数据阶段访问 flash。当前可读材料不足以确认 K3 驱动使用 PIO 还是 DMA、完成由轮询还是 IRQ 推进、以及 descriptor/buffer 的具体所有权转换。

因此 DTS 中出现 DMA/IRQ 只表示静态能力，不能写成实际数据路径。提交 buffer 前后的 cache 与所有权动作必须由具体驱动和平台一致性模型证明，本文不从通用 DMA 规则反推实现。

### 2.4 错误边界

当前证据未闭合 command timeout、FIFO 错误、DMA 错误、写/擦除失败、控制器 reset 或介质 busy 的返回与恢复语义。尤其不能由“存在 reset 属性”推出错误后会自动复位，也不能把未完成写入声明为可安全重试。

## 3. 普通 SPI

### 3.1 能力与静态资源

K3 datasheet 概述给出 6 路普通 SPI，但没有在当前可读材料中逐一闭合实例地址、板级引出与运行状态。固定第三方 IFX DTS 出现多个普通 `spi@...` 节点；部分节点含 MMIO、IRQ、clock/reset、pinctrl、`dmas` 与 `dma-names = "tx", "rx"`。这些属性只适用于该变体，不能证明六路全部连接到 CoM260 Kit，也不能证明运行时实际选择 DMA。

### 3.2 数据和完成路径

普通 SPI 的基本传输对象是由消费层组织的 message/transfer，可能包含不同片选、字长、频率和全双工/半双工阶段。当前没有同等级 K3 普通 SPI 数据面实现可审计，因此只能确认控制器能力与候选静态资源，不能确认：

- FIFO 深度、阈值和装填顺序；
- PIO 与 DMA 的选择条件；
- IRQ cause、mask、ack 与完成回调顺序；
- timeout、取消、片选释放和 controller reset 的语义。

普通 SPI 节点的 DMA 属性不适用于 QSPI XIP/Page，也不能外推到 SDHCI。

### 3.3 启动边界

启动基线中的 SPI NOR/SPI NAND 指向 SPI memory 启动路径，不足以证明普通 SPI message controller 承担 Boot ROM 取镜像。若后续材料显示特定控制器复用或 handoff，需由对应板型、节点和软件调用链共同确认。

## 4. SD、eMMC 与 SDHCI

### 4.1 SoC 与板级能力

K3 datasheet 记录 eMMC 5.1 与 SD 3.0 控制器兼容 SDHCI，并支持 PIO、SDMA、ADMA 和 ADMA2。它证明控制器族能力，不证明某个实例在特定板型上启用全部传输模式。

CoM260 板级资料确认 TF 卡接口；启动基线记录 SD 优先以及 eMMC 候选。当前资料没有唯一闭合 TF 卡的最高速度、热插拔检测行为、eMMC 是否装配及其容量。SD 是 removable 卡路径，eMMC 是焊接介质路径；即使共享 host controller 逻辑，也必须分别核对 pinctrl、bus width、电压与 timing。

### 4.2 固定 DTS 中的静态资源

固定第三方 IFX DTS 含三个 `compatible = "spacemit,k3-sdhci"` 的 `mmc@...` 节点，并描述 MMIO、IRQ、core/io clocks 与 reset。aliases 将其标为 `mmc0`、`mmc1`、`mmc2`。这些编号和资源只证明 IFX 变体，不能裁决 G7，也不能把任一实例直接指定为默认 Kit 的 TF 卡或 eMMC。

静态节点存在不等于 status 启用、介质已连接、OS 已 probe 成功或传输由 IRQ 推进。缺少消费层 glue 时，portable core 本身也不构成运行路径。

### 4.3 第三方 portable core 的职责

固定 revision 的 `k3-sdhci` core 持有 SDHCI MMIO 与 K3 vendor PHY/tuning 状态，区分 SD 与 eMMC mode，包含 HS200/HS400、DLL lock 和 software RX tuning（256 个 delay code 扫描）的处理。它把 FDT probe、IRQ、block registration 与系统集成交给消费层。

因此这些源码可以支持“存在可复用的 K3 PHY/tuning 算法”，不能支持“当前 CoM260 profile 已注册块设备”“当前路径采用 IRQ”或“真板已通过 HS400”。

### 4.4 数据、DMA、IRQ 与 tuning

对一个已集成的 SDHCI 路径，CPU/驱动通常先准备命令、descriptor 或 data buffer，再把所有权交给 host，等待完成后回收并处理 cache 可见性。K3 datasheet 只列出 PIO/SDMA/ADMA/ADMA2 能力；当前证据没有闭合具体 board profile 的选择条件和 cache maintenance。

IRQ 节点只证明中断资源存在。只有消费层注册 handler、启用 cause/mask 并以完成状态驱动请求回收后，才能声明 IRQ 推进。类似地，portable core 的 tuning 算法只有在消费层调用并取得有效采样窗口后才形成运行行为。

### 4.5 错误和恢复边界

可审计的 portable core 覆盖 DLL lock 与 tuning 失败等局部错误，但没有在当前材料中闭合整个块请求的 timeout、取消、controller reset、介质移除和重试策略。失败后的 descriptor、buffer 与介质写状态必须由消费层和硬件状态共同判断；不得声称复位后请求无损或写入可安全重放。

## 5. 启动关系总览

| 介质/控制器 | 已证关系 | 不能推出 |
| --- | --- | --- |
| QSPI + SPI NOR/NAND | K3 支持 QSPI，启动基线列出 SPI NOR/NAND 候选 | 默认启动介质、具体 flash、XIP 窗口或实际 DMA/IRQ 路径 |
| 普通 SPI | K3 有 6 路控制器能力，候选 DTS 有多个实例 | Boot ROM 使用普通 SPI controller、全部板级引出或运行时启用 |
| SD/SDHCI | CoM260 有 TF 卡接口，启动基线记录 SD 优先 | 任意 DTS alias 就是默认 TF 卡实例、最高 timing 或热插拔已验证 |
| eMMC/SDHCI | K3 支持 eMMC 5.1，启动基线列为候选 | CoM260 必然装配、容量、默认实例或 HS400 已验证 |

完整 Boot ROM、FSBL/SPL、OpenSBI、U-Boot 与 OS 交接仍由启动主题维护；本文不复制其顺序。

## 6. 未知项

### U1：QSPI 板载器件与运行路径

- 当前证据：官方 SoC 能力和 CoM260 板载 SPI Flash 可确认；社区驱动正文、器件型号和真板状态未取得。
- 禁止推断：不得指定 NOR/NAND、容量、频率、XIP 窗口、PIO/DMA 或 IRQ/轮询路径。
- 解除条件：取得适用 CoM260 版本的原理图/BOM、可读官方 QSPI 驱动页、目标 DTS 与驱动调用链；运行行为另需真板证据。
- 影响：启动介质选择、镜像布局、cache 属性、数据路径和错误恢复仍不能闭合。

### U2：普通 SPI 实例与板级连接

- 当前证据：官方概述给出 6 路；固定 IFX DTS 展示候选实例和资源。
- 禁止推断：不得把 IFX 地址、IRQ、DMA 或 pinctrl 当作默认 Kit，也不得声明六路均可达。
- 解除条件：取得与目标 Kit 版本唯一匹配的顶层 DTS、原理图和 pinmux 表，并确认实际驱动 probe。
- 影响：实例清单、片选连接、PIO/DMA 选择与完成路径保持未知。

### U3：SD 与 eMMC 的实例映射

- 当前证据：CoM260 有 TF 卡，K3 支持 SD/eMMC；固定 IFX DTS 有三个 SDHCI 节点。
- 禁止推断：不得用 alias 顺序裁决 TF 卡/eMMC，不得把控制器能力写成介质已装配或已启用。
- 解除条件：解决 G7，取得目标板原理图、适用 DTS include 链和运行时 probe/枚举结果。
- 影响：启动目标、bus width、电压、removable 属性和实例资源不能唯一绑定。

### U4：SDHCI 消费层与完成模型

- 当前证据：portable core 提供 PHY/tuning；FDT probe、IRQ 和 block registration 不在该 core 内。
- 禁止推断：不得声明当前 StarryOS/CoM260 profile 已集成、采用 IRQ、选择 ADMA2 或完成过 HS200/HS400。
- 解除条件：取得固定 board profile 的消费层 glue、配置和运行日志，并核对请求提交、IRQ/轮询、cache 与回收顺序。
- 影响：运行时吞吐路径、取消、timeout 和错误恢复无法形成实现保证。

### U5：错误后的资源和介质状态

- 当前证据：静态 reset/IRQ/DMA 属性与 portable core 的局部 PHY/tuning 错误可见；端到端请求恢复未闭合。
- 禁止推断：不得把 reset 属性当作自动恢复，不得声称未完成写入可安全重试。
- 解除条件：取得适用官方驱动的错误路径和真板错误注入结果，覆盖 command/data timeout、DMA error、介质 busy/移除及复位后状态。
- 影响：调用方必须把失败视为请求和介质状态可能不确定，不能据本文制定无损重试策略。

## 7. 主题边界与导航

- SoC 控制器总览：[K3 SoC 概述](../platform/k3-soc-overview.md)
- CoM260 介质和连接：[CoM260 板级资源](../platform/com260-board-resources.md)
- 启动阶段：[CoM260 启动链](../boot/com260-boot-chain.md)
- 镜像与 DTS：[CoM260 镜像与 DTS](../boot/com260-image-and-dts.md)
- DMA 与所有权：[K3 DMA 与内存所有权](../dma/k3-dma-and-memory-ownership.md)
- cache/PMA：[K3 cache、PMA 与地址转换](../dma/k3-cache-pma-address-translation.md)
- 跨主题缺口：[已知缺口](../reference/known-gaps.md)

UFS 的 MPHY、UniPro、UTP/SCSI、descriptor、完成与恢复属于后续独立正文，不在本文定义。
