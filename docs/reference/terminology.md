> 来源: https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs（源端修订: unknown；观察日期: 2026-09-02）

# 术语表

> 适用范围: `docs/` 下所有主题文档、覆盖表与 known-gaps 的术语统一。
> 产品文档通过获批 OpenSpec change 修改；change accepted 后，docs-maintainer 仅按实际结果同步 SNAPSHOT、tasks、M/D/K/R/I 中适用的状态。
> 写作约束来源: M03（语言与术语稳定）。

## 使用规则

- 标题层只使用主写法；同一文档中不并列中英同义词。
- 英文专有技术名词保留英文原拼写；不作中文意译。
- 当官网或驱动使用别名时，在「别名」字段记录，但正文仍使用主写法。
- 后续术语增删需创建 OpenSpec change，不得在主题文档中临时新增主写法。
- 当官方来源给出互斥命名时，本表保持现状并将冲突记录到 `known-gaps.md`。

## 基础术语

| 主写法 | 英文原词 / 缩写 | 别名 | 使用说明 |
| --- | --- | --- | --- |
| K3 | K3 (SpacemiT Key Stone K3) | SpacemiT K3、key_stone/k3 | 仓库范围（M01）唯一允许的 SoC 平台名。 |
| CoM260 Kit | K3 CoM260 Kit | CoM260、k3_com260 | 当前唯一技术目标板；指含 CoM260 模组的开发套件整体。 |
| SoC | System on Chip | 系统级芯片 | 指 K3 芯片本体；用于区分模组、底板、套件。 |
| AP | Application Processor | 应用处理器 | K3 主处理器域；与 RCPU 域相对。 |
| RCPU | Real-time Control Processing Unit | 实时控制处理器 | K3 实时控制域；与 AP 域相对。 |
| AIA | Advanced Interrupt Architecture | RISC-V AIA | interrupts 主题术语；硬件组成、寄存器与 K3 enablement 见 G4。 |
| APLIC | Advanced Platform-Level Interrupt Controller | 高级平台级中断控制器 | interrupts 主题术语；具体行为、地址与投递见 G4。 |
| IMSIC | Incoming MSI Controller | MSI 控制器 | interrupts 主题术语；具体行为、地址与投递见 G4。 |
| MMIO | Memory-Mapped I/O | 内存映射 I/O | 设备寄存器访问方式；本仓库专指 MMIO 寄存器读写。 |
| IRQ | Interrupt Request | 中断请求 | 通用中断信号；含 wired IRQ 与 MSI/MSI-X。 |
| DMA | Direct Memory Access | 直接内存访问 | 通用 DMA 控制器或设备内建 DMA；按上下文区分。 |
| IOMMU | I/O Memory Management Unit | IO 内存管理单元 | dma 主题术语；K3 上是否存在、地址转换与隔离语义见 G5。 |
| GMAC | Gigabit Media Access Controller | 千兆以太网 MAC | K3 上的以太网 MAC 控制器；用于 network 主题。 |
| PHY | Physical Layer Transceiver | 物理层收发器 | network 主题术语；型号、地址、reset、ref clock 等见 G3。 |
| MDIO | Management Data Input/Output | 管理数据接口 | network 主题术语；总线绑定与寄存器见 G3。 |
| RGMII | Reduced Gigabit Media Independent Interface | 精简千兆 MII | network 主题术语；delay/clock/reset/phy-mode 等见 G3。 |
| polling | polling | 轮询 | 驱动主动读取状态寄存器；与 async 相对。 |
| async | asynchronous | 异步 | 事件驱动 + waker 通知的执行模型；与 polling 相对。 |
| waker | waker | 唤醒器 | async 模型中用于通知任务可继续执行的句柄。 |
| coherency | cache coherency | 缓存一致性 | CPU cache 与设备 DMA 之间的可见性关系；K3 上需逐案确认。 |
| AMP | Asymmetric Multi-Processing | 非对称多处理 | amp 主题术语；K3 上 AP 与 RCPU 域的固定边界协同模式；生命周期与消息路径分别见 [`k3-amp-shared-memory-lifecycle.md`](../amp/k3-amp-shared-memory-lifecycle.md) 与 [`k3-rpc-ring-notification.md`](../amp/k3-rpc-ring-notification.md)。 |
| RPC | Remote Procedure Call | 远程过程调用 | amp 主题术语；AP↔RP 通过共享 ring 调用的请求/响应协议；与 mailbox 通知分离；通道、错误、Deferred、reset 边界见 [`k3-rpc-ring-notification.md`](../amp/k3-rpc-ring-notification.md) §3-§9。 |
| ring | ring | 环形缓冲区 | amp / network / dma 主题共用术语；按上下文区分 mailbox channel（硬件通道）、RPC ring（共享内存请求 / 响应 / urgent 通道）、DMA ring（设备内置 DMA 描述符环）；RPC ring 与 DMA ring 在硬件层、所有权模型与 reset 语义均不同，禁止混用；具体见 [`k3-rpc-ring-notification.md`](../amp/k3-rpc-ring-notification.md) §2.1 与 [`k3-gmac-dma-irq.md`](../network/k3-gmac-dma-irq.md) §3。 |
| doorbell | doorbell | 门铃 | 通知机制术语；按上下文区分 mailbox doorbell（AP↔RP 通知，写 mailbox 硬件 channel 触发对端 IRQ）与 GMAC doorbell（设备内置 DMA 写描述符所有权寄存器）；两层门铃在硬件层、寄存器偏移和 reset 语义均不同，禁止混用；具体见 [`k3-rpc-ring-notification.md`](../amp/k3-rpc-ring-notification.md) §5 与 [`k3-gmac-dma-irq.md`](../network/k3-gmac-dma-irq.md) §5。 |
| 共享窗口 | shared memory window | 共享内存窗口 | amp 主题术语；AP↔RP 通信使用的物理内存区间；地址、布局、初始化所有权、re-init 与 reset 边界见 [`k3-amp-shared-memory-lifecycle.md`](../amp/k3-amp-shared-memory-lifecycle.md) §3 与 §7；不得在两侧将 0 与 `0xc0800000` 视作可互换指针。 |
| SPI | Serial Peripheral Interface | 串行外设接口 | storage 主题术语；K3 上指通用 SPI 控制器（message-oriented），与 QSPI 行为模型不同；实例、pinmux、CoM260 板级连接、driver 形态见 [`k3-qspi-spi-sdhci.md`](../storage/k3-qspi-spi-sdhci.md) §2 / §3。 |
| QSPI | Quad SPI | 四线 SPI | storage 主题术语；面向 SPI NOR/NAND memory 与 XIP 的 quad/1/2/4 线控制器；与通用 SPI 不可互换；SoC 能力、CoM260 板载器件候选、运行路径与未知项见 [`k3-qspi-spi-sdhci.md`](../storage/k3-qspi-spi-sdhci.md) §2 / §5 / U1。 |
| SDHCI | SD Host Controller Interface | SD 主机控制器接口 | storage 主题术语；K3 上 SDHC 控制器（兼容 SD/eMMC）的 K3 vendor core；PHY/HS200/HS400/DLL/software RX tuning 见 [`k3-qspi-spi-sdhci.md`](../storage/k3-qspi-spi-sdhci.md) §4 / §6；不与 OS glue、FDT probe、IRQ 注册混用。 |
| eMMC | embedded MultiMediaCard | 内嵌多媒体卡 | storage 主题术语；与 SD 共用 SDHCI 家族但板级介质、bus width、removable 属性不同；CoM260 板级 mapping 与可证字段见 [`k3-qspi-spi-sdhci.md`](../storage/k3-qspi-spi-sdhci.md) §4 / U3。 |
| UFS | Universal Flash Storage | 通用闪存存储 | storage 主题术语；K3 上 UFS 2.2 / UniPro 1.6 / M-PHY 3.0 控制器；静态资源、MPHY/UniPro/link、UTP/SCSI、DMA/cache、轮询完成、recovery 与未知项见 [`k3-ufs.md`](../storage/k3-ufs.md)；descriptor/UTRD/UTMRD/UCD/PRDT 不外推到 QSPI/SPI/SDHCI。 |
| M-PHY | M-PHY | MPHY | storage 主题术语；UFS 的物理层（MIPI M-PHY 3.0）；主写法为 `M-PHY`，`MPHY` 仅作别名；PWM 启动 / HS-G3 升级 / lane 选择 / link startup 边界见 [`k3-ufs.md`](../storage/k3-ufs.md) §5。 |
| UniPro | Unified Protocol | UniPro 1.6 | storage 主题术语；UFS 的协议层（MIPI UniPro 1.6）；属性读写、PA/DB 与 link startup 边界见 [`k3-ufs.md`](../storage/k3-ufs.md) §5。 |
| UTP | UFS Transport Protocol | UFS 传输协议 | storage 主题术语；UFS 应用层之上的传输协议（UPIU 容器，承载 COMMAND / RESPONSE / QUERY / NOP OUT 等报文）；与底层 M-PHY/UniPro 分层，不外推到 QSPI/SPI/SDHCI；具体见 [`k3-ufs.md`](../storage/k3-ufs.md) §6。 |
| UPIU | UFS Protocol Information Unit | UFS 协议信息单元 | storage 主题术语；UTP 层承载的命令/响应/数据单元；UCD 的 command UPIU 与 response UPIU 各预留 512-byte 对齐区域；OCS 字段位于 UTRD，不在 UPIU response 内；具体见 [`k3-ufs.md`](../storage/k3-ufs.md) §6。 |
| UTRD | UTP Transfer Request Descriptor | UTP 传输请求描述符 | storage 主题术语；UFSHCI transfer list 中的传输请求描述符，包含 command type / data direction / interrupt bit、OCS、UCD base、response UPIU offset+length、PRDT offset+length；command UPIU 与 PRDT 由 UTRD 指向的 UCD 承载，task tag 在 command UPIU byte 3，不在 UTRD 字段内；具体见 [`k3-ufs.md`](../storage/k3-ufs.md) §6。 |
| UTMRD | UTP Task Management Request Descriptor | UTP 任务管理描述符 | storage 主题术语；UFSHCI transfer list 中的任务管理描述符槽位；本仓库固定 revision 第三方实现只分配并编程 UTMRD，未观察到 task-management 提交；QUERY / NOP OUT / COMMAND / RESPONSE 等报文经保留 transfer slot 的 UTRD/UCD 提交；具体见 [`k3-ufs.md`](../storage/k3-ufs.md) §6。 |
| UCD | UTP Command Descriptor | UTP 命令描述符 | storage 主题术语；UFSHCI 的命令描述符，由 UTRD 引用；承载 command UPIU 与 response UPIU 各 512-byte 对齐区域，并引用 PRDT 描述 data buffer 物理地址与长度；具体见 [`k3-ufs.md`](../storage/k3-ufs.md) §6。 |
| PRDT | Physical Region Descriptor Table | 物理区描述符表 | storage 主题术语；由 UCD 引用的 data buffer 物理区表，描述物理地址与长度；`prepare_slot` 把 PRDT 链接到 data buffer 并执行 `prepare_for_device`（cache clean + DMA fence）；具体见 [`k3-ufs.md`](../storage/k3-ufs.md) §6。 |
| SCSI | Small Computer System Interface | 小型计算机系统接口 | storage 主题术语；UFS 应用层使用的命令集（REPORT LUNS / TEST UNIT READY / INQUIRY / READ CAPACITY / READ / WRITE）；UTP 之上的语义，不外推到 QSPI/SPI/SDHCI；NOP OUT 不属 SCSI、QUERY 报文走 UTP/UPIU 容器；具体见 [`k3-ufs.md`](../storage/k3-ufs.md) §8。 |
| LUN | Logical Unit Number | 逻辑单元号 | storage 主题术语；UFS 设备逻辑单元；probe 阶段 NOP OUT / LUN 容量 / UNIT READY / flag 探测与 `register_sync_block` 注册同步块设备路径见 [`k3-ufs.md`](../storage/k3-ufs.md) §8。 |

## 标题层使用示例

- 正确: `## CoM260 Kit 板级资源`
- 错误: `## CoM260 Kit / 套件板级资源`（混排并列）
- 正确: `## GMAC 控制器与 PHY`
- 错误: `## GMAC 控制器 / 千兆 MAC`（混排并列）

## 与覆盖表、缺口文档的关系

- 主题文档的 `主题位置` 字段使用本表「主题」集合的标准词；自定义主题词需先在本表登记。
- 当术语存在多种合法主写法时，由本表唯一指定；其余写法仅作「别名」使用。
- 跨模块冲突或新增术语建议先登记到 `improvements/spec.md`，由批准后的 change 升格。
