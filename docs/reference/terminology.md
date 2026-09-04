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

## 标题层使用示例

- 正确: `## CoM260 Kit 板级资源`
- 错误: `## CoM260 Kit / 套件板级资源`（混排并列）
- 正确: `## GMAC 控制器与 PHY`
- 错误: `## GMAC 控制器 / 千兆 MAC`（混排并列）

## 与覆盖表、缺口文档的关系

- 主题文档的 `主题位置` 字段使用本表「主题」集合的标准词；自定义主题词需先在本表登记。
- 当术语存在多种合法主写法时，由本表唯一指定；其余写法仅作「别名」使用。
- 跨模块冲突或新增术语建议先登记到 `improvements/spec.md`，由批准后的 change 升格。
