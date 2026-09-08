> 来源: https://github.com/spacemit-com/docs-chip/blob/main/zh/key_stone/k3/k3_docs/k3_ds.md（源端修订: 2026-08-25；观察日期: 2026-09-07）；https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs/k3_ds.md（源端修订: unknown；观察日期: 2026-09-02）

# K3 SoC 概述

> 范围: SpacemiT Key Stone K3 SoC 自身能力。不覆盖 CoM260 模组引脚、Kit 板级信号、寄存器位、IRQ/DMA/GMAC 驱动路径。
> 边界: SoC 能力 ≠ CoM260 引出/启用。引出与启用清单见 [com260-board-resources.md](com260-board-resources.md)。

> 来源身份: 官网正文仍为 SPA 壳，本文档正文中由 GitHub 对应页直接证实的事实标 `交叉验证`；官网正文未直接取得的事实保留为 `未知项` 或在 GitHub 页面无依据时标 `未知项`。本文档不把 GitHub 仓库内容升级为官网事实。
> 修订快照: 本文档基于 GitHub 页 V1.8（2026-08-25）公开声明。V1.8 较 V1.6/V1.7 的差异在第 7 节注明。

## 目录

- [1. CPU 子系统与 hart](#1-cpu-子系统与-hart)
- [2. 内存控制器](#2-内存控制器)
- [3. 启动能力](#3-启动能力)
- [4. 存储接口 (SoC 控制器能力)](#4-存储接口-soc-控制器能力)
- [5. 多媒体与显示 (SoC 控制器能力)](#5-多媒体与显示-soc-控制器能力)
- [6. IO 扩展接口 (SoC 控制器能力)](#6-io-扩展接口-soc-控制器能力)
- [7. 物理与环境](#7-物理与环境)
- [8. 未知项与边界](#8-未知项与边界)
- [9. 修订快照](#9-修订快照)
- [10. 边界声明](#10-边界声明)

## 1. CPU 子系统与 hart

> 本节事实均经 GitHub 对应页 V1.8 直接证实；官网正文仍为 SPA 壳，未直接取得。

| 维度 | 事实 | 证据等级 | 来源 |
|---|---|---|---|
| 通用核 | 8 × SpacemiT X100 64-bit RISC-V 核，四发射乱序执行；完全符合 RVA23 规范；最大主频 2.4 GHz；总 CPU 性能 130 KDMIPS；SpecINT2006 > 9.0/GHz；每 8 核共享 8 MB L2 缓存 | 交叉验证 | k3_ds.md §1.2 + §2.1.1 |
| 通用核 L1 | 每核 64 KB L1 I-Cache + 64 KB L1 D-Cache | 交叉验证 | k3_ds.md §2.1.1 |
| AI 核 | 8 × SpacemiT A100 RISC-V AI 核；60 TOPS 通用 AI 算力；完整支持 RVA23* 规范（不含 Hypervisor 扩展） | 交叉验证 | k3_ds.md §1.2 + §2.1.2 |
| AI 核 L1 | 每核 32 KB L1 I-Cache + 32 KB L1 D-Cache | 交叉验证 | k3_ds.md §2.1.2 |
| 实时核 | 2 × RT24 64-bit RISC-V 核（OpenHW Group CVA6 派生，RV64GC，六级顺序单发射）；专责系统管理、电源管理与低功耗任务调度 | 交叉验证 | k3_ds.md §1.2 + §2.1.3 |
| 虚拟化 | 支持 RVH 1.0、RISC-V AIA 和 IOMMU 扩展；提供 CPU、内存、中断、IO 的完整硬件虚拟化能力 | 交叉验证 | k3_ds.md §1.2 |
| 中断 | RISC-V AIA；具体 ACLINT/APLIC/IMSIC 地址与 hart delivery 边界见 [未知项 §8](#8-未知项与边界) | 交叉验证（架构）/ 未知项（具体） | k3_ds.md §1.2 |
| 安全 | RISC-V PMP；安全启动/安全存储/签名验证；AES/SHA/RSA/SM2/SM3/SM4 | 交叉验证 | k3_ds.md §1.2 |
| 调试 | JTAG (DTM)；调试代理 (OpenOCD)；progbuf/sysbus 两种 JTAG 内存访问模式 | 交叉验证 | k3_ds.md §2.1.4 |
| Trace | 符合 RISC-V N-Trace 协议；X100/A100 各自独立的追踪编码器 + ATB 桥 | 交叉验证 | k3_ds.md §2.1.5 |

## 2. 内存控制器

> 本节事实均经 GitHub 对应页 V1.8 §1.2 + §2.2.2 直接证实。

| 维度 | 事实 | 证据等级 | 来源 |
|---|---|---|---|
| DRAM 类型 | LPDDR5（最高 6400 MT/s）/ LPDDR4x（最高 4266 MT/s） | 交叉验证 | k3_ds.md §1.2 + §2.2.2 |
| 最大寻址空间 | 32 GB | 交叉验证 | k3_ds.md §1.2 + §2.2.2 |
| 通道拓扑 | 双通道 LPDDR4x/LPDDR5 控制器；每通道 32 位数据宽度；每通道支持两个 Rank | 交叉验证 | k3_ds.md §2.2.2 |
| 带宽 | 51 GB/s | 交叉验证 | k3_ds.md §1.2 |
| 频率调节 | 支持动态频率调节（DFS），可根据带宽需求实时调整频率以优化功耗 | 交叉验证 | k3_ds.md §2.2.2 |
| 时序 | 时序参数遵循 JEDEC LPDDR5/LPDDR4x 标准（datasheet 不单独声明时序参数） | 交叉验证 | k3_ds.md §2.2.2 |

## 3. 启动能力

> 本节事实均经 GitHub 对应页 V1.8 §1.2 + §2.2.1 直接证实；介质选择优先级、下载模式握手协议与 eFuse 字段布局保留为 `未知项`，见 §8.3。

| 维度 | 事实 | 证据等级 | 来源 |
|---|---|---|---|
| Boot ROM | 128 KB Boot ROM；用于存放一级引导代码，支持从多种外部介质启动，并支持通过 USB 与 UART 下载程序 | 交叉验证 | k3_ds.md §2.2.1 |
| SRAM | 512 KB SRAM，由主 CPU 与 RCPU（RT24）共享 | 交叉验证 | k3_ds.md §2.2.1 |
| 启动模式 | datasheet 公开声明支持 download 与 local boot 两种模式 | 交叉验证 | k3_ds.md §1.2（与 §2.2.1 一致） |
| 安全启动 | 基于 eFuse 的信任根与签名验证链 | 交叉验证（能力） | k3_ds.md §1.2 |

## 4. 存储接口 (SoC 控制器能力)

> 本节事实均经 GitHub 对应页 V1.8 直接证实；每条事实仅证明 SoC 控制器层能力，不证明 CoM260 引出或 Kit 启用。章节定位：Quad-SPI = §2.2.3，eMMC = §2.2.4，SD/MMC = §2.2.5，UFS = §2.2.6。

| 维度 | 事实 | 证据等级 | 来源 |
|---|---|---|---|
| Quad-SPI | XIP 与 Page 模式；数据线 1/2/4 线独立配置；时钟 13.25–102 MHz；支持 SPI NOR / SPI NAND | 交叉验证 | k3_ds.md §2.2.3 |
| eMMC | 8 位 eMMC 5.1；HS400；详见 datasheet eMMC 章节 | 交叉验证 | k3_ds.md §2.2.4 |
| SD/MMC | SD 3.0/SDIO 3.0；UHS-I（SDR12/25/50/104 速率）；支持读等待控制与挂起/恢复 | 交叉验证 | k3_ds.md §2.2.5 |
| UFS | UFS 2.2 主机（符合 JEDEC UFS 2.2、MIPI UniPro v1.6、MIPI M-PHY v3.0 规范；HS-GEAR3 + PWM-GEAR1；支持从 UFS 直接启动） | 交叉验证 | k3_ds.md §2.2.6 |
| PCIe | 8 lanes PCIe Gen3（8 Gbps/通道），支持 RC/EP 模式及热插拔 | 交叉验证 | k3_ds.md §1.2 |

## 5. 多媒体与显示 (SoC 控制器能力)

> 本节事实均经 GitHub 对应页 V1.8 直接证实；V1.8（2026-08-25）补充了 DPU0/DPU1 接口支持说明。

| 维度 | 事实 | 证据等级 | 来源 |
|---|---|---|---|
| 视频编解码 | 多格式编解码能力（具体格式与分辨率见 datasheet 多媒体节） | 交叉验证 | k3_ds.md §1.2 |
| 3D GPU | 集成 3D 图形引擎，支持主流图形 API（具体 API 列表见 datasheet 多媒体节） | 交叉验证 | k3_ds.md §1.2 |
| 显示 | 2 路 DPU 输出，可实现双路 3840×2160@60fps | 交叉验证 | k3_ds.md §1.2 |
| DPU0 接口 | 支持 MIPI-DSI（8 通道，4.5 Gbps/通道）或 DP/eDP | 交叉验证 | k3_ds.md §1.2（V1.8 补充） |
| DPU1 接口 | 仅支持 DP/eDP（V1.8 明确 DPU1 不支持 MIPI-DSI） | 交叉验证 | k3_ds.md §1.2（V1.8 补充） |
| 摄像头 | 4 × MIPI-CSI，共 12 条通道，最多支持 12 路摄像头输入 | 交叉验证 | k3_ds.md §1.2 |

> V1.8 修订要点: 第 1.2 节明确 DPU0 支持 MIPI-DSI 或 DP/eDP，DPU1 仅支持 DP/eDP。V1.6/V1.7 未做此明确区分。本文档正文以 V1.8 为准。

## 6. IO 扩展接口 (SoC 控制器能力)

> 本节事实均经 GitHub 对应页 V1.8 §1.2 直接证实；具体基地址、IRQ 号、复位时序与多路控制器中的实例清单保留为 `未知项`，见 §8.5。

| 维度 | 事实 | 证据等级 | 来源 |
|---|---|---|---|
| 以太网 | 4 × GMAC（支持 RGMII/RMII/MII），集成 TSN 协议 | 交叉验证 | k3_ds.md §1.2 |
| USB | 3 × USB 3.0 Host；1 × USB 3.0 DRD（Type-C）；1 × USB 2.0 Host | 交叉验证 | k3_ds.md §1.2 |
| SPI / eSPI | 6 × SPI；2 × eSPI | 交叉验证 | k3_ds.md §1.2 |
| UART | 17 × UART | 交叉验证 | k3_ds.md §1.2 |
| CAN | 10 × CAN | 交叉验证 | k3_ds.md §1.2 |
| I²C | 9 × I²C | 交叉验证 | k3_ds.md §1.2 |
| PWM | 30 × PWM | 交叉验证 | k3_ds.md §1.2 |

## 7. 物理与环境

> 本节事实均经 GitHub 对应页 V1.8 §1.2 + §5.2.2 + §5.3 直接证实；工艺节点与封装类型（FCBGA 命名/球距）保留为 `未知项`，见 §8.6。

| 维度 | 事实 | 证据等级 | 来源 |
|---|---|---|---|
| TDP | 15–25 W | 交叉验证 | k3_ds.md §1.2 |
| 工作温度 | –40 °C 至 +85 °C（工业级） | 交叉验证 | k3_ds.md §1.2 + §5.2.2 |
| 结温 | 上限 125 °C | 交叉验证 | k3_ds.md §5.2.2 |
| 存储温度 | –40 °C 至 125 °C | 交叉验证 | k3_ds.md §5.2.2 |
| 热阻 | 0.23 °C/W（带散热盖） | 交叉验证 | k3_ds.md §5.3 |
| 工艺节点 | datasheet 公开层面未给出具体工艺节点 | 未知项 | 见 §8.6 |
| 封装类型 | datasheet §4.1 引脚分配使用 AA~AT、1~20 编号方案，但未给出 FCBGA 命名或球距 | 交叉验证（章节存在）/ 未知项（命名/球距） | k3_ds.md §4.1 |

## 8. 未知项与边界

> 每个未知项按模板给出 `当前证据 / 禁止推断 / 解除条件 / 影响主题` 四个字段。本节汇总所有需要后续 bring-up 决策但 datasheet V1.8 公开内容不能唯一给出的字段。

### 8.1 官网正文与 GitHub 对应页的身份边界

- 当前证据: SpacemiT 官方 K3 文档入口 `https://www.spacemit.com/.../k3_ds.md` 实际渲染为 SPA 壳，官网正文未直接取得；本仓库唯一可直接读取的 K3 datasheet 正文为 GitHub `docs-chip/.../k3_ds.md` V1.8。
- 禁止推断: 不得把 GitHub V1.8 内容升级为官网事实；不得在 `K3 datasheet` 标题下写"官网声明"等修辞；不得在官网行使用 GitHub 行的 `V1.8` 自动代填 `源端修订`。
- 解除条件: 取得 SpacemiT 官网 K3 datasheet 的非 SPA 渲染正文（如 curl 直接拿到 Markdown），或得到 SpacemiT 官方对 GitHub 页与官网正文同源性的明确说明。
- 影响主题: 全部 K3/CoM260 主题；当前所有 `交叉验证` 字段在身份边界解除前无法升级为官网事实。

### 8.2 DRAM 时序 / 工作频率 / ECC / 温度监控

- 当前证据: V1.8 §2.2.2 已给出通道拓扑（双通道、32-bit、2 Rank）、速率（LPDDR5 6400 MT/s / LPDDR4x 4266 MT/s）和 DFS 能力。V1.8 §1.2 已给出最大寻址容量 32 GB 和带宽 51 GB/s。本节未直接给出实际工作频率、时序参数、ECC 支持与温度监控范围。
- 禁止推断: 不得用 K1、其它 RISC-V SoC、Pico 板或 Linux 默认配置反推 K3 实际时序；不得沿用任何旧版本 datasheet 的字段值。
- 解除条件: 取得 V1.8 后续章节中关于实际工作频率、时序参数、ECC 与温度监控的具体数字；或从 SpacemiT 官方寄存器手册与目标 DTS `memory`/`reg` 节点交叉验证。
- 影响主题: 后续 MS06 (DMA/coherency) 与 MS03 Iteration 001 中装载地址、reserved-memory 边界；MS07 (GMAC) 的 descriptor ring 内存布局。

### 8.3 Boot ROM 介质优先级、下载模式握手与 eFuse 字段

- 当前证据: V1.8 §2.2.1 已给出 128 KB Boot ROM 与 512 KB SRAM。本节未给出 Boot ROM 介质选择优先级、USB/UART 下载模式的具体速率/协议、eFuse 字段布局。
- 禁止推断: 不得用通用 RISC-V SiFive 等价物或 Linux U-Boot 默认 bootcmd 推导 K3 介质选择顺序；不得写固定 load address 或 eFuse 字段值。
- 解除条件: 取得 SpacemiT 公开的 programmer reference 或 Boot ROM 章节；或从 `linux-6.18` K3 DTS `chosen` 节点与 OpenSBI/U-Boot 默认配置交叉验证。
- 影响主题: MS03 Iteration 001 的 `com260-boot-chain.md` 与 `com260-image-and-dts.md`；MS05 (IRQ) 的早期 console 与 timer bring-up 顺序。

### 8.4 中断控制器 (AIA/APLIC/IMSIC) 地址与 hart delivery

- 当前证据: V1.8 §1.2 仅声明"支持 RISC-V AIA 和 IOMMU 扩展"。APLIC/IMSIC 的 MMIO 地址、IRQ domain、hart routing、mask/ack/complete 寄存器布局 datasheet 概述层未给出。
- 禁止推断: 不得用 RISC-V AIA 标准或 SiFive APLIC/IMSIC 寄存器布局等同 K3 实现；不得在 DTS 或驱动中使用任何具体偏移。
- 解除条件: 取得 SpacemiT 公开 programmer reference 中 AIA 章节；或在 R08 `linux-6.18` 仓库定位 K3 APLIC/IMSIC 设备树与驱动初始化源码。
- 影响主题: G4（`docs/reference/known-gaps.md::G4`）；MS05 中断与时间主题。

### 8.5 SoC 控制器基地址 / IRQ 号 / 复位时序 / 多路实例清单

- 当前证据: V1.8 §1.2 概述层已给出 GMAC/USB/UART/SPI/eSPI/CAN/I²C/PWM 等控制器的路数；本节未给出基地址、IRQ 号、复位顺序、时钟源以及多路控制器中各路实例的具体编号。
- 禁止推断: 不得用 Linux 驱动默认偏移或 DWMAC 通用寄存器反推 K3 控制器基地址；不得为未给出的实例写固定 IRQ 编号。
- 解除条件: 取得 SpacemiT 公开寄存器手册中 SoC 控制器章节；或从 `linux-6.18` K3 DTS `reg`/`interrupts`/`clocks`/`resets` 属性交叉验证。
- 影响主题: MS04 (pinctrl/clock/reset/UART)、MS05 (IRQ)、MS06 (DMA)、MS07 (GMAC) 全部后续主题。

### 8.6 工艺节点与封装类型 (FCBGA 命名 / 球距)

- 当前证据: V1.8 §1.2 给出 TDP 15–25 W；§5.2.2 给出工作温度 -40°C~85°C、结温上限 125°C、存储温度 -40°C~125°C；§5.3 给出热阻 0.23 °C/W（带散热盖）；§4.1 引脚分配使用 AA~AT、1~20 编号方案。datasheet 公开层面未给出工艺节点、FCBGA 命名或球距等具体封装参数。
- 禁止推断: 不得沿用未经直接观察的旧值；不得用相邻 K1/Pico/RV2768 的工艺或封装等同 K3。
- 解除条件: 取得 V1.8 物理与电气章节中关于工艺节点、FCBGA 命名、球距的具体字段；或从 datasheet 完整 PDF 取得对应章节。
- 影响主题: 整机功耗与散热设计；与底板 FAN、连接器、电源规格的耦合判断；硬件 layout。

## 9. 修订快照

- 本文档基于 GitHub `docs-chip` 对应页 V1.8（2026-08-25）。
- V1.8 较 V1.7 的差异: 第 1.2 节补充 DPU0/DPU1 显示接口支持说明（DPU0 支持 MIPI-DSI 或 DP/eDP；DPU1 仅支持 DP/eDP）。
- V1.7 较 V1.6 的差异: 新增第 4.4 节引脚分配表交叉引用。
- 任何字段必须以 V1.8 GitHub 页当前内容为准；旧版字段保留在 `修订快照` 段以备 refresh change 对比。
- 后续 refresh change 须把新观察日期、源端修订与本节内容同步到覆盖表与本文档。

## 10. 边界声明

1. **SoC 能力 ≠ 板级引出/启用**: 本文档列出的 GMAC/USB/UART/PCIe/UFS/SD/eMMC/SPI/QSPI 等条目均为 SoC 控制器层能力，CoM260 模组实际引出与 Kit 底板实际连接见 [com260-board-resources.md](com260-board-resources.md)。
2. **不展开 MS04-MS07**: 本文档不包含 pinctrl/clock/reset/UART 寄存器、IRQ delivery、DMA/IOMMU coherency、GMAC PHY/MDIO 寄存器级实现；这些内容由对应 milestone 主题文档负责。
3. **不混入非目标板**: 本文档不引用 K3 Pico-ITX、K3 Pico 模组、Pico 板型或 K1/RV2768/Shelf 资料作为 K3 SoC 事实。
4. **官网/GitHub 身份不混**: 全部事实标 `交叉验证`（来源为 GitHub 对应页 V1.8）；官网正文未直接取得的事实按 §8 保留为 `未知项`，不通过 GitHub 自动代填。
