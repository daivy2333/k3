> 来源: https://github.com/spacemit-com/docs-chip/blob/main/zh/key_stone/k3/k3_docs/k3_ds.md（源端修订: 2026-08-25；观察日期: 2026-09-07）；https://github.com/spacemit-com/docs-product/blob/main/zh/k3_com260/com260_ds.md（源端修订: 2026-08-25；观察日期: 2026-09-07）；https://github.com/spacemit-com/docs-product/blob/main/zh/k3_com260/com260_user_guide.md（源端修订: 2026-03-19；观察日期: 2026-09-07）；https://github.com/spacemit-com/docs-product/blob/main/zh/k3_com260/com260_hw_resources.md（源端修订: unknown；观察日期: 2026-09-07）

# CoM260 板级资源矩阵

> 范围: CoM260 模组本身 + K3 CoM260 开发套件 (Kit) 实际引出/启用的资源。不覆盖: K3 SoC 控制器层全集（见 [k3-soc-overview.md](k3-soc-overview.md)）、寄存器位、IRQ/DMA/GMAC 驱动路径、PMIC 内部时序。
> 边界: SoC 能力 ≠ 模组引出 ≠ Kit 启用。来源只证明某一层时，其他层不得自动继承。

> 来源身份: 三个直接来源均为 SpacemiT 官方 GitHub 对应页，官网正文仍为 SPA 壳未直接取得。本文档正文事实由 GitHub 页面直接证实，按四级证据等级标 `交叉验证`；官网行不参与正文事实；同一参数在不同来源出现冲突时按 §10 显式并列。

## 目录

- [1. 四层来源模型](#1-四层来源模型)
- [2. 资源矩阵 (CPU/hart / DRAM / UFS / SPI Flash / TF Card / Debug UART / GMAC/PHY / 连接器扩展)](#2-资源矩阵-cpuhart--dram--ufs--spi-flash--tf-card--debug-uart--gmacphy--连接器扩展)
- [3. 连接器/扩展明细 (Kit/底板)](#3-连接器扩展明细-kit底板)
- [4. 电源](#4-电源)
- [5. 物理与形态](#5-物理与形态)
- [6. 文档与设计资源](#6-文档与设计资源)
- [7. 启动相关字段 (含刷机与串口)](#7-启动相关字段-含刷机与串口)
- [8. 未知项与边界](#8-未知项与边界)
- [9. 修订快照](#9-修订快照)
- [10. 已发现的来源冲突](#10-已发现的来源冲突)
- [11. 边界声明](#11-边界声明)

## 1. 四层来源模型

> 每条资源按 `SoC 能力 / 模组引出 / Kit 底板 / 未知边界` 四层来源记录。来源只证明某层时，其他层不得自动继承。CPU/hart 也保留该模型：SoC 核心数量不自动证明软件可用 hart 集合。

证据等级说明:

- `交叉验证`: 由官方 GitHub 对应页直接证实（来源: GitHub `docs-product` 仓库）
- `推论`: 由多个已证事实推出但源页面未直接陈述（必须列出所依据的多项事实）
- `未知项`: 当前证据不足（必须给出 `当前证据 / 禁止推断 / 解除条件 / 影响主题` 四字段闭包）

## 2. 资源矩阵 (CPU/hart / DRAM / UFS / SPI Flash / TF Card / Debug UART / GMAC/PHY / 连接器扩展)

> 矩阵的"层"列固定使用 `SoC 能力 / 模组引出 / Kit 底板 / 未知边界`；"证据等级"列只使用四级枚举；"来源"列精确指向本仓库已登记的 GitHub 精确页。`来源: k3_ds.md` 字段意味着此事实的 SoC 能力部分由 K3 datasheet 共同支撑，但本页正文事实仍以 GitHub 三个直接来源为准。

### 2.1 CPU 与 hart

| 层 | 事实 | 证据等级 | 来源 |
|---|---|---|---|
| SoC 能力 | 8 × X100 + 8 × A100 + 2 × RT24；详见 [k3-soc-overview.md §1](k3-soc-overview.md#1-cpu-子系统与-hart) | 交叉验证 | k3_ds.md §1.2 + §2.1 |
| 模组引出 | 1 × K3 主控（含 18 个核） | 交叉验证 | com260_ds.md §1.1 |
| Kit/底板 | 复用模组主控；RT24 系统管理核的可见性保留为 `未知项` | 交叉验证（复用）/ 未知项（可见性） | com260_user_guide.md §1.1 |
| 未知边界 | Boot 后 hart 启动顺序、SMP bring-up 顺序、hart 在不同 OS 镜像中的可见集合 | 未知项 | （GitHub 页面未直接证实） |

### 2.2 DRAM

| 层 | 事实 | 证据等级 | 来源 |
|---|---|---|---|
| SoC 能力 | LPDDR5（最高 6400 MT/s）/ LPDDR4x（最高 4266 MT/s）；双通道，每通道 32 位数据宽度，每通道两个 Rank；最大寻址空间 32 GB；带宽 51 GB/s | 交叉验证 | k3_ds.md §1.2 + §2.2.2 |
| 模组引出 | 2 × LPDDR5；单颗可选 4 GB / 8 GB / 16 GB | 交叉验证 | com260_user_guide.md §2 + §5.2 |
| 模组订货 | 订货型号存储容量: COM3K308128 (8GB DDR / 128GB UFS), COM3K316128 (16GB DDR / 128GB UFS), COM3K332128 (32GB DDR / 128GB UFS) | 交叉验证 | com260_ds.md §1.4 |
| Kit/底板 | 通过金手指使用模组 DRAM | 交叉验证 | com260_user_guide.md §3.3.1 |
| 未知边界 | 实际工作频率、时序参数、DDR 温度监控范围 | 未知项 | （GitHub 页面未直接证实） |

### 2.3 UFS

| 层 | 事实 | 证据等级 | 来源 |
|---|---|---|---|
| SoC 能力 | UFS 2.2 主机（符合 JEDEC UFS 2.2、MIPI UniPro v1.6、MIPI M-PHY v3.0 规范；HS-GEAR3 + PWM-GEAR1；支持从 UFS 直接启动） | 交叉验证 | k3_ds.md §2.2.6 |
| 模组引出 | UFS 2.2 板载；默认容量 128 GB | 交叉验证 | com260_user_guide.md §5.2 |
| 模组订货 | 三个公开订货型号 DDR 容量分别为 8/16/32 GB，UFS 容量均为 128 GB | 交叉验证 | com260_ds.md §1.4 |
| Kit/底板 | 通过金手指使用模组 UFS | 交叉验证 | com260_user_guide.md §3.3.1 |
| 未知边界 | 256 GB UFS 是否真实出货（user guide §2 "可选 128 GB / 256 GB"与 §3.3.3 功能接口表 "UFS (128GB)"不一致，且 com260_ds.md §1.4 仅列 128 GB）| 未知项 | 见 §10 冲突 C1 |

### 2.4 SPI Flash

| 层 | 事实 | 证据等级 | 来源 |
|---|---|---|---|
| SoC 能力 | Quad-SPI XIP/NOR/NAND，13.25–102 MHz | 交叉验证 | k3_ds.md §2.2.3 |
| 模组引出 | 板载 SPI Flash（容量与芯片型号未声明）| 交叉验证（存在）/ 未知项（容量/型号） | com260_user_guide.md §5.2 |
| Kit/底板 | 通过金手指使用模组 SPI Flash | 交叉验证 | com260_user_guide.md §3.3.1 |
| 未知边界 | 容量、芯片型号、1.8/3.3 V 电平选择 | 未知项 | （GitHub 页面未直接证实） |

### 2.5 TF Card

| 层 | 事实 | 证据等级 | 来源 |
|---|---|---|---|
| SoC 能力 | SD 3.0/SDIO 3.0；UHS-I（SDR12/25/50/104） | 交叉验证 | k3_ds.md §2.2.5 |
| 模组引出 | SDIO 3.0 引脚到金手指 | 交叉验证 | com260_ds.md §1.1 |
| Kit/底板 | 1 × TF-Card 接口；支持高速 TF 卡 | 交叉验证 | com260_user_guide.md §2 + §3.3.2 + §3.3.3 |
| 未知边界 | 实际支持 SD / SDHC / SDXC 等级、最高速度、热插拔行为 | 未知项 | （GitHub 页面未直接证实） |

### 2.6 调试串口 (UART0)

| 层 | 事实 | 证据等级 | 来源 |
|---|---|---|---|
| SoC 能力 | 17 × UART（具体实例清单保留为 `未知项`） | 交叉验证 | k3_ds.md §1.2 |
| 模组引出 | UART0 控制器引脚到金手指 | 交叉验证 | com260_ds.md §1.1 |
| Kit/底板 | 12 Pin 按钮排针含 UART0_TXD / UART0_RXD（V2.0 文档修订说明 1: 互换 RX/TX 位置；Pin 3 = UART0_RXD, Pin 4 = UART0_TXD）| 交叉验证 | com260_user_guide.md 版本表 + §5.3 |
| 串口参数 | 115200-8-N-1（按 user guide §8.2 MobaXterm 设置） | 交叉验证 | com260_user_guide.md §8.2 |
| 未知边界 | flow control 支持与否、V2.0 修订前版本的丝印或 Pin 定义 | 未知项 | （GitHub 页面未直接证实） |

### 2.7 GMAC/PHY

| 层 | 事实 | 证据等级 | 来源 |
|---|---|---|---|
| SoC 能力 | 4 × GMAC（支持 RGMII/RMII/MII），集成 TSN 协议 | 交叉验证 | k3_ds.md §1.2 |
| 模组引出 | 1 × GPHY 芯片 + 1 × GMAC 通道（PHY1）到金手指 | 交叉验证 | com260_ds.md §1.1 + §GMAC 信号表 |
| 模组引出 | 模组金手指上 GMAC1_RXD/RX_CLK/MDC/MDIO/INT_N 等 PHY1 互联信号已布局 | 交叉验证 | com260_ds.md §GMAC 信号表 |
| Kit/底板 | 1 × RJ45（自适应 10/100/1000M） | 交叉验证 | com260_user_guide.md §2 + §3.3.2 + §5.9 |
| 未知边界 | PHY 型号、MDIO 地址、TSN 是否启用、剩余 GMAC 实例是否可在复用管脚上启用、PHY1 LED/CFG_LDO0 实际配置 | 未知项 | 见 G3 (`docs/reference/known-gaps.md::G3`) |

### 2.8 连接器/扩展

| 层 | 事实 | 证据等级 | 来源 |
|---|---|---|---|
| SoC 能力 | 8 lanes PCIe Gen3（支持 RC/EP 模式及热插拔）；3 × USB 3.0 Host + 1 × USB 3.0 DRD + 1 × USB 2.0 Host；6 × SPI + 2 × eSPI；10 × CAN；9 × I²C；30 × PWM | 交叉验证 | k3_ds.md §1.2 |
| 模组引出 | 控制器引脚到金手指（多路） | 交叉验证 | com260_ds.md §1.1 |
| Kit/底板 | 详见 [§3 连接器/扩展明细](#3-连接器扩展明细-kit底板) | 交叉验证 | com260_user_guide.md §2 + §3.3.2 + 各 §5.x |

## 3. 连接器/扩展明细 (Kit/底板)

> 本节按 Kit/底板层逐条列连接器；每行均标注精确来源条款与证据等级。CAN/RTC 的连接器与"功能接口可用性"差异见 §10 冲突 C2/C3。

| 连接器 | 数量/接口 | 来源条款 | 证据等级 |
|---|---|---|---|
| M.2 2280 M-Key（NVMe SSD） | 1 | com260_user_guide.md §2 + §3.3.1 + §5.13 | 交叉验证 |
| M.2 2230 M-Key（NVMe SSD） | 1 | com260_user_guide.md §2 + §3.3.1 + §5.13 | 交叉验证 |
| M.2 2230 E-Key（Wi-Fi 6 / BT 5.0，默认 RTL8852BE） | 1 | com260_user_guide.md §2 + §3.3.1 + §5.10 | 交叉验证 |
| USB 3.0/2.0 Type-A（2 × Dual 连接器） | 4 | com260_user_guide.md §2 + §3.3.2 + §5.8 | 交叉验证 |
| USB 3.0 Type-C OTG（仅下载，不供电） | 1 | com260_user_guide.md §2 + §3.3.2 + §5.6 | 交叉验证 |
| DP 1.2 Type-A（3840×2160@60fps） | 1 | com260_user_guide.md §2 + §3.3.2 + §5.7 | 交叉验证 |
| MIPI DSI 30 Pin FPC（1 lane 2，配套 I2C + CTP）| 1 | com260_user_guide.md §3.3.2 + §5.5 | 交叉验证 |
| MIPI CSI 22 Pin FPC（CAM0 / CAM1，4+4 或 4+2+2）| 2 | com260_user_guide.md §3.3.2 + §5.4 | 交叉验证 |
| 40 Pin 双排插针（RPi 兼容：GPIO/UART/SPI/I2S/I2C） | 1 | com260_user_guide.md §2 + §3.3.2 + §5.11 | 交叉验证 |
| 12 Pin 按钮排针（LED + DEBUG UART + ACOK + Reset + Download + Power） | 1 | com260_user_guide.md §2 + §3.3.2 + §5.3 | 交叉验证 |
| 4 Pin FAN（4 线可调速，PWM + TACH） | 1 | com260_user_guide.md §2 + §3.3.2 + §5.3 | 交叉验证 |
| 4 Pin CAN FD（板载收发器）| 1 | com260_user_guide.md §2 + §3.3.2 + §5.14 | 交叉验证（连接器存在）/ 未知项（功能接口可用性，见 §10 冲突 C2）|
| 2 Pin RTC（1.85 V ~ 5 V 输入）| 1 | com260_user_guide.md §2 + §3.3.2 + §3.3.3 | 交叉验证（连接器存在）/ 未知项（功能接口可用性，见 §10 冲突 C3）|
| DC Jack（DCIN） | 1 | com260_user_guide.md §3.3.2 + §5.1 + §6.1.1 | 交叉验证（存在）/ 未知项（电源规格见 §10 冲突 C4）|
| 散热模组（支持安装散热器） | — | com260_user_guide.md §2 + §6.2 | 交叉验证 |
| EEPROM（板卡信息存储）| 1 | com260_user_guide.md §5.2 | 交叉验证 |

## 4. 电源

| 维度 | 事实 | 证据等级 | 来源 |
|---|---|---|---|
| SoC TDP | 见 [k3-soc-overview.md §7](k3-soc-overview.md#7-物理与环境)；V1.8 §1.2 未声明具体数值 | 未知项（具体值） | k3_ds.md §1.2 |
| 模组供电 | P1 PMIC + 外挂 DC-DC（LPDDR5 + 无源器件同模组）| 交叉验证 | com260_user_guide.md §3.3.1 + com260_ds.md §1.1 |
| Kit DCIN（连接器存在） | 1 × DC Jack | 交叉验证 | com260_user_guide.md §3.3.2 + §5.1 |
| Kit DCIN（规格）| 详见 §10 冲突 C4 | 未知项 | （三处来源对电压与电流要求不一致）|
| Kit 开机逻辑 | 首次通电自动开机；软件关机后短按电源键开机；红色电源 LED；散热风扇启动 | 交叉验证 | com260_user_guide.md §6.2 |
| ESD/防护 | 外设接口支持 ESD 防护接触（user guide §2 占位字段未填具体等级）| 未知项 | com260_user_guide.md §2 |

## 5. 物理与形态

| 维度 | 事实 | 证据等级 | 来源 |
|---|---|---|---|
| 模组封装 | 金手指 260 Pin（SODIMM）；69.6 mm × 45 mm | 交叉验证 | com260_user_guide.md §2 + §3.3.1 + com260_ds.md §1.1 |
| 模组厚度 | 1.2 mm（按 com260_ds 旧版快照）| 交叉验证（旧版快照）| com260_ds.md（旧 V1.2 节中保留字段）|
| 底板尺寸 | 100 mm × 70 mm | 交叉验证 | com260_user_guide.md §2 |
| 散热 | 模组与底板组合使用，支持安装散热器 | 交叉验证 | com260_user_guide.md §2 + §6.2 |
| 工作温度 | –40 °C 至 +85 °C（工业级）| 交叉验证（旧版快照）| com260_ds.md 旧 V1.2 节中保留字段 |

> V1.3（2026-08-25）较 V1.2 更新了订货型号存储容量、供电规格、引脚定义与电气参数；模组封装尺寸与工作温度字段在 V1.3 GitHub 页中亦保留。后续 refresh change 须把新观察日期同步到覆盖表与本文档。

## 6. 文档与设计资源

| 维度 | 事实 | 证据等级 | 来源 |
|---|---|---|---|
| 模组 datasheet | GitHub `docs-product/.../com260_ds.md` V1.3（2026-08-25） | 交叉验证 | 已登记 GitHub 精确页 |
| 模组 user guide | GitHub `docs-product/.../com260_user_guide.md` V2.0（2026-03-19） | 交叉验证 | 已登记 GitHub 精确页 |
| Kit 硬件设计资源 | GitHub `docs-product/.../com260_hw_resources.md`（具体修订未声明；页面标题为"K3 CoM260 开发套件硬件设计资源"）| 交叉验证（页面存在）| 已登记 GitHub 精确页 |
| BOM 文件 | `K3-COM260_KIT_V02_20260318_BOM.xls`（CDN 下载入口可见，但本仓库未实际打开）| 交叉验证（文件存在）/ 未知项（内容）| com260_hw_resources.md（GitHub 精确页）|
| 原理图 | `k3-com260_kit_v02_20260306.pdf`（CDN 下载入口可见，但本仓库未实际打开）| 交叉验证（文件存在）/ 未知项（内容）| com260_hw_resources.md（GitHub 精确页）|
| DSN（底板工程）| `K3-COM260_KIT_V02_20260306.DSN`（CDN 下载入口可见，但本仓库未实际打开）| 交叉验证（文件存在）/ 未知项（内容）| com260_hw_resources.md（GitHub 精确页）|
| BRD（底板 PCB）| `K3_COM260_KIT_V02-20260310_1330_pcb.brd`（CDN 下载入口可见，但本仓库未实际打开）| 交叉验证（文件存在）/ 未知项（内容）| com260_hw_resources.md（GitHub 精确页）|

> BOM/原理图/DSN/BRD 四个下载制品的 CDN 入口在 GitHub `com260_hw_resources.md` 可见，但本仓库未实际下载和解析。后续 refresh change 须先下载并解析后再补充器件位号、信号拓扑、阻抗控制等具体内容。

## 7. 串口与调试接口 (Kit/底板层)

> 本节只列板级串口与调试资源；启动模式、下载模式入口、刷机工具、装载地址、固件交接、Boot ROM 介质选择优先级与 eFuse 字段布局等 boot/image 资料由 Iteration 001 的 `com260-boot-chain.md` 与 `com260-image-and-dts.md` 负责，本节不提前聚合。

| 维度 | 事实 | 证据等级 | 来源 |
|---|---|---|---|
| 串口参数 | 115200-8-N-1（按 MobaXterm 设置）| 交叉验证 | com260_user_guide.md §8.2 |
| 串口物理 | USB 转 TTL 接 12 Pin 接口的 TX、RX、GND | 交叉验证 | com260_user_guide.md §8.1 |
| 12 Pin UART0 位置 | Pin 3 = UART0_RXD, Pin 4 = UART0_TXD（V2.0 修订说明 1: 互换 RX/TX 位置）| 交叉验证 | com260_user_guide.md 版本表 + §5.3 |
| 未知边界 | flow control 支持与否、V2.0 修订前版本的丝印或 Pin 定义 | 未知项 | （GitHub 页面未直接证实）|

## 8. 未知项与边界

> 每个未知项按模板给出 `当前证据 / 禁止推断 / 解除条件 / 影响主题` 四个字段。本节列出本文档范围内与 CoM260 板级决策直接相关、但 GitHub 三个直接来源不能唯一给出的字段。

### 8.1 官网/GitHub 身份边界

- 当前证据: SpacemiT 官方 CoM260 文档入口（datasheet、user guide、hardware resources）在 GitHub `docs-product` 仓库可读；官网三个对应 URL 实际渲染为 SPA 壳，官网正文未直接取得。本文档全部事实以 GitHub 三个直接来源为准。
- 禁止推断: 不得把 GitHub V1.3 / V2.0 内容升级为官网事实；不得在 `CoM260 datasheet` 标题下写"官网声明"等修辞；不得在官网行使用 GitHub 行的 `V1.3` / `V2.0` 自动代填 `源端修订`。
- 解除条件: 取得 SpacemiT 官网 K3 CoM260 datasheet / user guide / hardware resources 的非 SPA 渲染正文；或得到 SpacemiT 官方对 GitHub 页与官网正文同源性的明确说明。
- 影响主题: 全部 CoM260 主题；当前所有 `交叉验证` 字段在身份边界解除前无法升级为官网事实。

### 8.2 256 GB UFS 是否真实出货

- 当前证据: com260_user_guide.md §2 "本地存储: UFS 2.2, 可选 128 GB / 256 GB 容量"；§3.3.3 功能接口表只列 `UFS (128GB)`；com260_ds.md §1.4 订货型号表只列 8/16/32 GB DDR × 128 GB UFS 三个组合。
- 禁止推断: 不得根据 user guide §2 的"可选 256 GB"主张 CoM260 模组存在 256 GB 变体；不得根据 §3.3.3 否认 256 GB 存在。
- 解除条件: 取得 com260_ds.md 新版本或未来订货型号表中的 256 GB 型号；或 com260_hw_resources.md 列出 256 GB 下载制品。
- 影响主题: 存储与镜像主题；MS07 GMAC 不直接依赖，但 UFS 是 Kit 本地存储，影响后续 bring-up 镜像规划。

### 8.3 CAN 连接器可用性

- 当前证据: com260_user_guide.md §3.3.2 列 "1 个 4 Pin 排针, 集成 CAN 收发器"；§3.3.3 功能接口表 `CAN=否`；§5.14 描述"板载 CAN 收发器, 可直接连接 CAN 设备"。
- 禁止推断: 不得根据 §3.3.2 或 §5.14 主张 CAN 在当前产品版本中可用；不得根据 §3.3.3 否认 CAN 收发器在板上的物理存在。
- 解除条件: 取得 com260_ds.md 或 user guide 新版本中关于 CAN 软件使能（pinctrl、clock、driver binding）的明确说明；或目标 DTS 中存在 `&can0 { status = "okay"; };` 节点。
- 影响主题: MS04 与 buses 主题；CAN 收发器本身是 Kit 物理层存在，OS 层是否启用待 DTS/SDK 验证。

### 8.4 RTC 连接器与 RTC 功能

- 当前证据: com260_user_guide.md §3.3.2 列 "1 个 2 Pin 线对板连接器, 支持 1.85 V ~ 5 V 电压输入"；§3.3.3 功能接口表 `RTC=否`。
- 禁止推断: 不得根据 §3.3.2 主张 Kit 板载纽扣电池或板内 RTC 时钟；不得根据 §3.3.3 否认 2 Pin 连接器存在。
- 解除条件: 取得 com260_ds.md / user guide 新版本或 BOM/原理图中关于 RTC 时钟源（外部电池、PCF8563 之类）的明确信息；或目标 DTS 存在 `&rtc { status = "okay"; };` 节点。
- 影响主题: peripherals 主题；MS03 不展开，但 Kit 启动时间戳与日志功能可能受影响。

### 8.5 DCIN 电源规格

- 当前证据: com260_user_guide.md §5.1 "请使用支持 12 V-6 A 的电源适配器"；§6.1.1 "规格为 19V/2.37A 或 12V/5A"；§2 "电源输入: 支持 19V/2.37A 和 12V/5A DCIN 供电"。
- 禁止推断: 不得根据 §5.1 主张 12V/6A 是唯一支持规格；不得根据 §2/§6.1.1 主张 19V/2.37A 是默认规格；不得在没有实测的情况下选择任一规格。
- 解除条件: 取得 com260_ds.md V1.3（已更新供电规格）或 com260_hw_resources.md 原理图中关于 DCIN 输入范围、PMIC 兼容电压、推荐电源适配器型号的明确信息。
- 影响主题: 电源与整机功耗；MS04 platform 主题的 clock/pinctrl/reset 时序与 DCIN 上电时序耦合。

### 8.6 模组金手指完整引脚与电气参数

- 当前证据: com260_ds.md V1.3 已更新引脚定义与电气参数；本仓库仅观察到概要层（如 §1.1 接口清单、§GMAC 信号摘要、§1.4 订货型号），未直接看到完整引脚表。
- 禁止推断: 不得用 com260_ds.md V1.2 或更早版本的引脚表等同 V1.3；不得用 K1/Pico/RV2768 模组的引脚表等同 CoM260。
- 解除条件: 直接打开 com260_ds.md V1.3 完整引脚表或 com260_hw_resources.md 中下载的原理图。
- 影响主题: 模组与底板信号互联；MS04 (pinctrl/clock/reset/UART)、MS05 (IRQ)、MS07 (GMAC) 全部后续主题。

### 8.7 下载制品 (BOM/原理图/DSN/BRD) 内容

- 当前证据: com260_hw_resources.md 列出四个下载制品文件名（见 §6）；本仓库未实际下载和解析。
- 禁止推断: 不得仅凭 CDN 文件名主张器件位号、电气参数、信号拓扑、阻抗控制；不得在板级事实表中引用未解析内容。
- 解除条件: 实际下载并解析四个文件后，登记内容摘要与对应 R 登记。
- 影响主题: 全部板级主题；尤其 MS04/MS07 的实现级引用。

### 8.8 装载地址 / DRAM 保留区 / 固件交接

- 当前证据: K3 datasheet V1.8 与 CoM260 datasheet V1.3 / user guide V2.0 公开层面均未给出 load address、DRAM 保留区、Boot ROM → OpenSBI → U-Boot → payload/OS 的 handoff 寄存器值。
- 禁止推断: 不得用通用 RISC-V 约定或 SiFive 等价物反推 K3 地址布局；不得用 Linux U-Boot default bootcmd 等同 K3 默认值。
- 解除条件: 取得 SpacemiT 公开 Boot ROM 章节或 programmer reference；或从 `linux-6.18` K3 DTS `chosen` 节点与 OpenSBI/U-Boot 默认配置交叉验证。
- 影响主题: Iteration 001 的 `com260-boot-chain.md` 与 `com260-image-and-dts.md`；MS05 timer bring-up 顺序。

### 8.9 SoC 控制器在 CoM260 上的实例与基地址

- 当前证据: K3 datasheet V1.8 §1.2 概述层仅给出 GMAC/USB/UART/SPI/eSPI/CAN/I²C/PWM 等控制器的能力，未给出实例清单与基地址；com260_ds.md V1.3 给出 PHY1 互联、UART0 引脚复用等局部信号。
- 禁止推断: 不得用 Linux 驱动默认偏移或 DWMAC 通用寄存器反推 K3 控制器基地址；不得为未给出的实例写固定 IRQ 编号。
- 解除条件: 取得 SpacemiT 公开 programmer reference 中 SoC 控制器章节；或从 `linux-6.18` K3 DTS `reg`/`interrupts`/`clocks`/`resets` 属性交叉验证。
- 影响主题: MS04 (pinctrl/clock/reset/UART)、MS05 (IRQ)、MS06 (DMA)、MS07 (GMAC) 全部后续主题。

## 9. 修订快照

- 模组 datasheet 修订快照: 本文档以 V1.3（2026-08-25）公开声明为准。V1.3 较 V1.2 更新订货型号存储容量、供电规格、引脚定义与电气参数。
- Kit user guide 修订快照: 本文档以 V2.0（2026-03-19）公开声明为准。V2.0 较 V1.0 互换 UART0 RX/TX 位置、CAM0 调整为 MIPI CSI1 2Lane。
- Kit 硬件设计资源 修订快照: GitHub `com260_hw_resources.md` 页面未声明具体修订版本；BOM/原理图/DSN/BRD 四个下载制品的文件名包含 `v02` 与 `20260306/0310/0318` 时间戳，但本仓库未实际打开。
- 后续 refresh change 须把新观察日期、源端修订与本节内容同步到覆盖表与本文档。

## 10. 已发现的来源冲突

> 本节明确记录同参数在不同来源（datasheet V1.3 / user guide V2.0 / hardware resources GitHub 页）之间的不一致，并按 R2/S4 的"并列记录来源位置、适用层级和影响"原则保留全部版本。

### 冲突 C1 — UFS 容量（128 GB vs 256 GB）

- 位置 1: com260_user_guide.md §2 "本地存储: UFS 2.2, 可选 128 GB / 256 GB 容量"
- 位置 2: com260_user_guide.md §3.3.3 功能接口表 "UFS (128GB) = 是"
- 位置 3: com260_ds.md §1.4 订货型号表 三个组合均为 8/16/32 GB DDR × 128 GB UFS
- 适用层级: user guide §2 是产品规格描述；§3.3.3 是当前套件功能接口描述；datasheet §1.4 是公开订货型号清单
- 影响: 三处描述对 UFS 容量的描述不一致。本文档不裁决，仅保留三处原文并存；不解释成因，不推导"当前套件固定功能"或"仅覆盖 128 GB"等结论。
- 解除条件: 见 §8.2。

### 冲突 C2 — CAN（连接器存在 vs 功能接口 = 否）

- 位置 1: com260_user_guide.md §3.3.2 "1 个 4 Pin 排针, 集成 CAN 收发器"（连接器）
- 位置 2: com260_user_guide.md §3.3.3 功能接口表 "CAN = 否"（功能接口）
- 位置 3: com260_user_guide.md §5.14 "板载 CAN 收发器, 可直接连接 CAN 设备"（硬件描述）
- 适用层级: §3.3.2 是物理连接器；§3.3.3 是当前套件功能接口可用性；§5.14 是硬件模块描述
- 影响: §3.3.2 / §5.14 描述了 CAN 物理存在，§3.3.3 在"功能接口可用性"表中标为"否"。本文档不裁决，仅保留三处原文并存；不解释为"OS 层未启用"。
- 解除条件: 见 §8.3。

### 冲突 C3 — RTC（连接器 vs 功能接口）

- 位置 1: com260_user_guide.md §3.3.2 "1 个 2 Pin 线对板连接器, 支持 1.85 V ~ 5 V 电压输入"（连接器）
- 位置 2: com260_user_guide.md §3.3.3 功能接口表 "RTC = 否"（功能接口）
- 适用层级: §3.3.2 是物理连接器；§3.3.3 是当前套件功能接口可用性
- 影响: 与 C2 同构。本文档不裁决，仅保留两处原文并存；不解释为"OS 层未启用"。
- 解除条件: 见 §8.4。

### 冲突 C4 — DCIN 电源规格

- 位置 1: com260_user_guide.md §5.1 "请使用支持 12 V-6 A 的电源适配器"（使用前要求）
- 位置 2: com260_user_guide.md §6.1.1 "规格为 19V/2.37A 或 12V/5A"（规格清单）
- 位置 3: com260_user_guide.md §2 "电源输入: 支持 19V/2.37A 和 12V/5A DCIN 供电"（产品规格表）
- 适用层级: §5.1 是使用前要求；§2 是产品规格；§6.1.1 是规格清单
- 影响: §5.1（12V/6A）与 §2 / §6.1.1（19V/2.37A 或 12V/5A）描述了不同的电源规格。本文档不裁决，仅保留三处原文并存；不简化为"建议规格 vs 兼容规格"。
- 解除条件: 见 §8.5。

## 11. 边界声明

1. **SoC 能力 ≠ 模组引出 ≠ Kit 启用**: 矩阵中"模组引出"列是 K3-CoM260 模组金手指/板上集成的资源；"Kit/底板"列是开发套件底板引出到连接器的资源；两者不可互相替代。
2. **官网/GitHub 身份不混**: 全部正文事实由已登记的三个 GitHub 精确页直接证实，标 `交叉验证`；官网三个对应 URL 仍为 SPA 壳未直接取得，不参与正文事实。
3. **不展开 MS04-MS07**: 本文档不包含 pinctrl/clock/reset/UART 寄存器、IRQ delivery、DMA/IOMMU coherency、GMAC PHY/MDIO 寄存器级实现；这些内容由对应 milestone 主题文档负责。
4. **不混入非目标板**: 本文档不引用 K3 Pico-ITX、K3 Pico 模组、Pico 板型或 K1/RV2768/Shelf 资料作为 CoM260 事实。
5. **冲突不解释**: §10 列出的 C1-C4 冲突本文档只并列原文，不解释成因、不裁决、不推导"建议/兼容"或"OS 层未启用"等结论；后续 refresh change 须取得 com260_ds.md 新版本或 com260_hw_resources.md 解析结果后才能消除。
