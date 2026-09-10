> 来源: https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs（源端修订: unknown；观察日期: 2026-09-02）

# K3 文档总入口

> 仓库范围: 仅 K3（SpacemiT Key Stone K3）；当前技术目标板固定为 K3 CoM260 Kit。
> 范围约束: M01（单一权威源）、D02（主题驱动目录）；不镜像官网 URL 树。
> 产品文档通过获批 OpenSpec change 修改；change accepted 后，docs-maintainer 仅按实际结果同步 SNAPSHOT、tasks、M/D/K/R/I 中适用的状态。

## 当前范围

- 唯一权威源: <https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs>
- 唯一技术目标板: K3 CoM260 Kit
- 其他 K3 板卡: Pico-ITX / Pico 模组等登记在覆盖表，状态 `out-of-scope`；不聚合正文。
- StarryOS 后续使用 K3 CoM260；本仓库不修改 StarryOS。

## 参考文档

- [来源覆盖表](reference/source-coverage.md): 当前登记 70 个唯一 URL；包含来源职责、目标范围、主题位置、优先级、聚合状态、源端修订、观察日期、访问状态与备注。
- [主题文档模板](reference/document-template.md): 主题文档的写作约束、可复用骨架与反例；包含单来源/多来源首行、四级证据强度、500 行拆分规则。
- [术语表](reference/terminology.md): 25 个基础术语的主写法、英文原词、别名与使用说明；标题层只使用主写法。
- [已知缺口](reference/known-gaps.md): 当前登记 G1-G11 共 11 类缺口；包含当前证据、禁止推断、解除条件与影响主题，其中 G3、G4、G5 为 `partial`。
- [来源刷新指南](reference/source-refresh.md): 人工刷新五种结果（unchanged / changed / moved / removed / unreachable）、操作顺序、change 与缺口边界、中断恢复与三段文字演练；与覆盖表的长期聚合状态、访问状态严格分离。

## 已聚合主题正文

> 入口页只列主题正文相对链接，不在入口页贴具体技术内容；技术内容写到主题文档。
> 主题正文与 `主题职责` 表的 `状态` 列保持一致；空目录与占位 overview 不建立。

- `docs/platform/`
  - [k3-soc-overview.md](platform/k3-soc-overview.md): K3 SoC 概述(pinctrl/clock/reset/显示/USB/PCIe 等控制器能力)、启动能力、BROM 引导与已知 SDK baseline 入口。已聚合(Iteration 000)。
  - [k3-platform-control.md](platform/k3-platform-control.md): K3 SoC 平台控制资源按 AP / APBC2 secure / RCPU 三域分离的 pinctrl / clock / reset / APBC / CCU provider 依赖与 UART consumer 字段映射。已聚合(Iteration 000)。
  - [com260-board-resources.md](platform/com260-board-resources.md): K3 CoM260 模组与 Kit 板级资源(电源、UART、USB、PCIe、以太网、显示、摄像头、GPIO、SPI、SD、CAN、UFS、eMMC、EtherCAT、电源域)。已聚合(Iteration 000)。
- `docs/boot/`
  - [com260-boot-chain.md](boot/com260-boot-chain.md): K3 SoC 启动能力与 K3 CoM260 Kit 已观察到的启动链路(local boot 路径: Boot ROM → FSBL/SPL → ESOS → OpenSBI → U-Boot → payload/OS; download boot 路径: Boot ROM → U-Boot Fastboot)、介质(SoC 支持 vs Kit 可用)、SDK 版本边界与未知项闭包。已聚合(Iteration 001)。
  - [com260-image-and-dts.md](boot/com260-image-and-dts.md): K3 CoM260 Kit 的可证镜像类型、写入方式、CoM260 DTS 候选集合(7 个候选 + 3 个已直接打开)、产品版本与 DTS 命名映射、未知项闭包。已聚合(Iteration 001)。
- `docs/serial/`
  - [com260-uart.md](serial/com260-uart.md): K3 SoC 17 个 UART 物理实例(AP 域 10 + APBC2 secure 1 + RCPU 域 6)的 DTS 字段、CoM260 UART0 物理接口到 `uart0` 节点与静态 console 链路、来源冲突与固定 revision 第三方经验。已聚合(Iteration 001)。
- `docs/dma/`
  - [k3-dma-and-memory-ownership.md](dma/k3-dma-and-memory-ownership.md): K3 DMA 控制器(三类传输对象: 通用 DMA controller / GMAC 内建 DMA / UFS 内建 DMA)的描述、AP↔RP 共享内存 + mailbox 通知 + descriptor / data buffer 状态机、错误超时取消与恢复、未知项 U1-U4。已聚合(Iteration 000)。
  - [k3-cache-pma-address-translation.md](dma/k3-cache-pma-address-translation.md): cache、barrier/fence、PMA/PBMT、IOMMU 与 CPU/device 地址空间的机制边界、对象路径和未知项。已聚合(Iteration 001)。
- `docs/interrupts/`
  - [k3-interrupt-and-time.md](interrupts/k3-interrupt-and-time.md): K3 AP CLINT/APLIC/IMSIC 与 RP PLIC/SysTimer/MSIP 的分域静态事实、证据边界和未知项。已聚合(Iteration 000)。
  - [com260-mailbox-notification.md](interrupts/com260-mailbox-notification.md): 固定 revision 第三方工程中的 AP↔RP mailbox 双向通知、FIFO/pending 处理、自测能力和可靠性边界。已聚合(Iteration 001)。
- `docs/network/`
  - [com260-gmac-phy.md](network/com260-gmac-phy.md): K3 SoC、CoM260 模组与 DTS 变体中的 GMAC 实例、平台资源、MDIO、PHY、RGMII、reset 和 delay-line 静态链。已聚合(Iteration 000)。
  - [k3-gmac-dma-irq.md](network/k3-gmac-dma-irq.md): DWMAC5 MAC/MTL/DMA、TX/RX descriptor 与 data buffer ownership、cache/doorbell、IRQ/reclaim、错误恢复和并发推进边界。已聚合(Iteration 001)。
- `docs/amp/`
  - [k3-amp-shared-memory-lifecycle.md](amp/k3-amp-shared-memory-lifecycle.md): K3 AP/RP 镜像装载与握手、共享窗口地址、初始化所有权、内存属性前置（PMA / fence）、失效复位与重新初始化、状态机边界。已聚合(Iteration 000)。
  - [k3-rpc-ring-notification.md](amp/k3-rpc-ring-notification.md): AP↔RP 消息路径：三层编号（共享内存 ring / mailbox 通道 / processor）、BUSY 与发布 fence、doorbell 与 IRQ/waker、Deferred/error/poison、超时与取消、reset 恢复与未决消息；缺失 `ov-channels`/`rt-async` 边界明确。已聚合(Iteration 001)。

## 主题职责

> 以下十类职责在覆盖表的 `主题位置` 字段已分配；`platform`、`boot`、`serial`、`interrupts`、`network`、`amp` 已聚合正文（见上节），其余 4 类在各自聚合 change 之前不创建空目录或占位正文。

| 主题路径 | 职责 | 当前主要来源（R05/R06 编号） | 状态 |
| --- | --- | --- | --- |
| `docs/platform/` | SoC 概述、pinctrl、clock、reset、设备管理；CoM260 板级资源归属 | R04 k3_ds / root_overview / com260_hw_resources；R05 device_management / 01-PINCTRL / 16-Clock / Reset | 已聚合(Iteration 000)：k3-soc-overview / k3-platform-control / com260-board-resources |
| `docs/boot/` | 启动流程、镜像构建；OpenSBI / U-Boot handoff | R05 boot / image；linux-6.18 仓库 k3-br-v1.0.y 分支 | 已聚合(Iteration 001)：com260-boot-chain / com260-image-and-dts |
| `docs/interrupts/` | AIA / APLIC / IMSIC、timer、hart routing、mailbox notification | R05 Timer；linux-6.18 K3 DTS/binding；R10-R12 固定 revision 第三方分析 | 已聚合(Iteration 001)：k3-interrupt-and-time / com260-mailbox-notification；G4 partial |
| `docs/serial/` | UART 控制器、pinmux、early console | R05 05-UART | 已聚合(Iteration 001)：com260-uart |
| `docs/dma/` | DMA 控制器、descriptor、地址宽度、ownership 转换 | R05 21-DMA | 已聚合(Iteration 000)：k3-dma-and-memory-ownership；已聚合(Iteration 001)：k3-cache-pma-address-translation；G5 partial |
| `docs/network/` | GMAC、PHY、MDIO、RGMII、descriptor ring、interrupt cause/ack | R05 09-GMAC；linux-6.18 K3 DTS/glue driver；R10-R12 固定 revision 第三方分析 | 已聚合(Iteration 000)：com260-gmac-phy；已聚合(Iteration 001)：k3-gmac-dma-irq；G3/G4/G5 partial，G6/G7 open |
| `docs/amp/` | AP↔RP 镜像装载、共享窗口地址、初始化所有权、生命周期、消息路径（ring/RPC/doorbell/等待/错误恢复） | R05 boot；linux-6.18 K3 DTS；R09-R11 固定 revision 第三方分析 | 已聚合(Iteration 000)：k3-amp-shared-memory-lifecycle；已聚合(Iteration 001)：k3-rpc-ring-notification；G11 open |
| `docs/storage/` | SDHC、UFS、QSPI、SPI 控制器 | R06 08-SDHC / ufs / 07-QSPI / SPI | 待聚合；R06 状态 deferred |
| `docs/buses/` | I2C、PCIe、CAN、USB、EtherCAT | R06 06-I2C / 11-PCIe / 15-CAN / 10-USB / 22-EtherCAT | 待聚合；R06 状态 deferred |
| `docs/peripherals/` | GPIO、PWM、IR-RX、Audio、WDT、RTC | R06 02-GPIO / 03-PWM / 04-IR-RX / 17-Audio / 23-WDT / 24-RTC | 待聚合；R06 状态 deferred |

## 维护规则

- 任何主题目录的创建必须由对应聚合 change 产生，且首篇正文必须符合 `document-template.md`。
- 主题目录不创建空 overview 或占位文件；目录的存在与覆盖表的 `主题位置` 一致。
- 主题文档变更前必须建立测试见证（文件存在、首行合规、相对链接解析、四级证据标注、术语一致）。
- 源端变更必须创建 refresh change；`source-coverage.md` 之外不静默修改 `> 来源:` 行。
- 入口页不贴具体技术内容；技术内容只写到主题文档；状态/计数（当前 70 URL、25 术语、11 gaps）随主题文档增量同步，禁止脱离主题文档独立修改。

## 与全局约束的关系

- 本仓库不存放可执行代码（M04）；任何辅助工具（抓取、链接检查、渲染）必须在独立仓库并通过 `references/spec.md` 登记。
- 文档语言为简体中文（zh-CN），专有技术名词保留英文原拼写（M03）。
- 任何 K3 资料变更先在 R01 入口确认（M01）；源页面 URL 改变时创建 change 并刷新所有相关文档的 `> 来源:` 行。
