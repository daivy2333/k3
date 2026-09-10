## Context

本 change 是纯 Markdown 聚合。`docs/dma/` 尚不存在，`docs/index.md` 只把 DMA 标为“待聚合；G5 阻塞 coherency/IOMMU”。现有产品文档已确认 K3 概述层声明 IOMMU 能力，并分别保存 CoM260 DRAM、GMAC、UFS、共享内存和中断边界，但没有统一的 DMA ownership 与内存属性说明。

2026-09-09 的只读调查确认三类可用证据。R05 的 21-DMA 官网 URL 已登记但正文仍为 SPA 壳；SpacemiT 官方仓库中的 DTS、binding 和 driver 可作为交叉验证；R09-R12 及固定 revision `others/Rt-Async-AMP` 提供 GMAC/UFS descriptor、cache maintenance、OpenSBI PMA 与 AP/RP SRAM alias 的第三方行为。它们能形成对象和顺序清单，不能证明 K3 全局 coherency 或真板效果。

## Goals / Non-goals

**Goals**

- 建立通用 DMA、GMAC/UFS 内建 DMA和共享内存的对象边界。
- 建立 descriptor/data buffer、CPU/device ownership、提交/完成/回收的统一术语和问题表。
- 分开记录 cache maintenance、barrier/fence、PMA/PBMT、IOMMU 与地址转换。
- 更新精确来源、G5/新增缺口判断和 DMA 导航。

**Non-goals**

- 不设计或实现 DMA API、allocator、IOMMU、cache、GMAC、UFS 或共享内存驱动。
- 不把第三方代码注释、历史报告或源码存在写成本项目真板结论。
- 不展开 PHY/MDIO、UFS 协议、AMP RPC、性能调优或设备数据面。
- 不创建脚本、构建、仿真、上板测试或 Evidence 身份机制。

## Decisions

### D1：两篇主题文档

`docs/dma/k3-dma-and-memory-ownership.md` 承载 DMA 类型、能力、descriptor/data buffer 和 ownership 生命周期；`docs/dma/k3-cache-pma-address-translation.md` 承载 cache、barrier、PMA/PBMT、IOMMU、地址空间和 alias。两个主题的证据对象与故障域不同，拆分后各自预计低于 450 行。

替代方案是合并成单篇文档。该方案会把设备协议与平台内存属性混在同一条叙述链中，后续 MS07-MS09 难以只引用需要的边界，因此不采用。

### D2：以传输对象划分 DMA 行为

通用 DMA controller、GMAC ring、UFS UTRD/UCD/PRDT 和 AP/RP 共享 SRAM 分开记录。共同术语只用于比较，不用于互补地址宽度、descriptor 格式、cache 操作或完成语义。

替代方案是按 API 名称聚合。Linux 或第三方 API 会隐藏设备差异，也会违反“不把软件约定提升为硬件规范”的要求，因此不采用。

### D3：ownership 以可观察状态转换表达

每条设备路径使用“CPU 准备 → 发布/提交 → device owns/in flight → completion observed → CPU 回收”的状态关系，并分别记录 descriptor 与 data buffer。IRQ、doorbell 或 mailbox 只属于触发或通知，不自动证明数据可见。

替代方案是只列寄存器、函数或 cache API。该方式无法回答错误、timeout、reset 后谁可安全复用内存，因此不采用。

### D4：内存属性机制不互相替代

cache clean/invalidate、Release/Acquire、I/O fence、PMA、PBMT、页表属性和 IOMMU 各自记录对象、方向、设置层和证据。共享 SRAM 的 PMA 路径只属于对应窗口与固定 revision；GMAC/UFS 必须保留自己的 cache 和地址转换边界。

替代方案是把“non-coherent”归结为统一 flush 规则。现有源码已经显示 fence 不执行 cache clean，且不同设备的缓冲区处理不同，因此不采用。

### D5：G5 只按实际解除字段更新

官方或官方源码能补齐的 controller、DTS、IOMMU 节点和软件行为可使 G5 局部收敛；cache line、全局 coherency、硬件 barrier、设备映射或真板效果仍缺证据时不得关闭。新缺口只有在对象与解除条件不和 G5/G6 重复时才创建。

替代方案是为每个设备建立独立 coherency 缺口。这样会重复 G5 并失去全局解除条件，因此不采用。

## Risks / Trade-offs

- [官方页面仍是 SPA 壳] → 保持 `partially-observed`，只把官方 GitHub 内容标为 `交叉验证`。
- [CoM260 顶层 DTS 未唯一映射] → 使用共享 SoC/CoM260 字段，变体专有字段带变体名称，不写成默认 Kit 配置。
- [第三方源码注释包含硬件断言和历史实测] → 正文优先记录可读控制流；测量只作为作者记录，并列出本项目未运行边界。
- [GMAC/UFS 内容可能侵入 MS07/MS09] → 只保留 ownership、cache、地址和 completion；PHY、协议与完整寄存器语义留给对应 milestone。
- [候选 URL 数量在 Act 时变化] → 只登记可直接打开且正文实际引用的精确 URL；来源变化影响设计时停止返回 Plan。

## Migration Plan

无需数据迁移。Iteration 000 先建立来源与 ownership 文档；Iteration 001 再建立内存属性文档并同步缺口和入口。任一 Iteration 未被 Review 接受时，不展开下一 Iteration。

## Open Questions

没有阻塞 Gate 2 的实质问题。IOMMU 实际绑定、cache line、PMA/PBMT 真板效果和各设备 coherency 属正文必须保留的未知项，不交给 Act 决定契约语义。
