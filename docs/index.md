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

- [来源覆盖表](reference/source-coverage.md): 唯一覆盖 R01、R04-R08 的 38 个 URL；包含来源职责、目标范围、主题位置、优先级、聚合状态、源端修订、观察日期、访问状态与备注。
- [主题文档模板](reference/document-template.md): 主题文档的写作约束、可复用骨架与反例；包含单来源/多来源首行、四级证据强度、500 行拆分规则。
- [术语表](reference/terminology.md): 20 个基础术语的主写法、英文原词、别名与使用说明；标题层只使用主写法。
- [已知缺口](reference/known-gaps.md): 6 类缺口的当前证据、禁止推断、解除条件与影响主题；每类对应具体的 source-coverage 状态。
- `docs/reference/source-refresh.md`: 人工刷新状态机与操作顺序；由 Iteration 001（T7）创建，本入口在指南就绪后转为相对链接。

## 主题职责（未来路径）

> 当前 `docs/` 下尚未建立任何主题目录；以下九类职责在覆盖表的 `主题位置` 字段已分配，但目录与正文由对应聚合 change 实际产生。空目录与占位 overview 不建立。

| 主题路径 | 职责 | 当前主要来源（R05/R06 编号） | 状态 |
| --- | --- | --- | --- |
| `docs/platform/` | SoC 概述、pinctrl、clock、reset、设备管理；CoM260 板级资源归属 | R04 k3_ds / root_overview / com260_hw_resources；R05 device_management / 01-PINCTRL / 16-Clock / Reset | 待聚合 |
| `docs/boot/` | 启动流程、镜像构建；OpenSBI / U-Boot handoff | R05 boot / image | 待聚合 |
| `docs/interrupts/` | AIA / APLIC / IMSIC、timer、hart routing | R05 Timer | 待聚合；G4 阻塞寄存器级描述 |
| `docs/serial/` | UART 控制器、pinmux、early console | R05 05-UART | 待聚合 |
| `docs/dma/` | DMA 控制器、descriptor、地址宽度、ownership 转换 | R05 21-DMA | 待聚合；G5 阻塞 coherency/IOMMU |
| `docs/network/` | GMAC、PHY、MDIO、RGMII、descriptor ring、interrupt cause/ack | R05 09-GMAC | 待聚合；G3、G6 阻塞实例与寄存器级描述 |
| `docs/storage/` | SDHC、UFS、QSPI、SPI 控制器 | R06 08-SDHC / ufs / 07-QSPI / SPI | 待聚合；R06 状态 deferred |
| `docs/buses/` | I2C、PCIe、CAN、USB、EtherCAT | R06 06-I2C / 11-PCIe / 15-CAN / 10-USB / 22-EtherCAT | 待聚合；R06 状态 deferred |
| `docs/peripherals/` | GPIO、PWM、IR-RX、Audio、WDT、RTC | R06 02-GPIO / 03-PWM / 04-IR-RX / 17-Audio / 23-WDT / 24-RTC | 待聚合；R06 状态 deferred |

## 维护规则

- 任何主题目录的创建必须由对应聚合 change 产生，且首篇正文必须符合 `document-template.md`。
- 主题目录不创建空 overview 或占位文件；目录的存在与覆盖表的 `主题位置` 一致。
- 主题文档变更前必须建立测试见证（文件存在、首行合规、相对链接解析、四级证据标注、术语一致）。
- 源端变更必须创建 refresh change；`source-coverage.md` 之外不静默修改 `> 来源:` 行。

## 与全局约束的关系

- 本仓库不存放可执行代码（M04）；任何辅助工具（抓取、链接检查、渲染）必须在独立仓库并通过 `references/spec.md` 登记。
- 文档语言为简体中文（zh-CN），专有技术名词保留英文原拼写（M03）。
- 任何 K3 资料变更先在 R01 入口确认（M01）；源页面 URL 改变时创建 change 并刷新所有相关文档的 `> 来源:` 行。
