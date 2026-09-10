## Why

MS03–MS05 已建立 CoM260 的板级、平台、中断和通知基线，但 DMA controller、设备内建 DMA、cache maintenance、PMA/PBMT、IOMMU 与 CPU/device ownership 仍散落在官方资料和第三方工程中。MS06 需要先形成分层、可追溯的知识基线，避免后续 GMAC、UFS、共享内存和驱动工作把某个设备的处理方式外推为 K3 全局规则。

## Source Baseline and Change Type

- 权威入口：<https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs>
- 现有官网主题：21-DMA 页面已登记，观察日期 2026-09-02，状态 `partially-observed`；官网正文仍为 SPA 壳。
- 官方交叉验证：SpacemiT `docs-chip`、`docs-buildroot`、`linux-6.18` 和可定位的 OpenSBI K3 来源。
- 第三方经验：R09–R12 所登记的 Rt-Async-AMP/tgoskits 固定 revision，只记录源码行为、风险和待验证假设。
- Change 类型：aggregation；建立 MS06 知识基线，不实现 DMA、cache、IOMMU 或设备驱动。

## What Changes

- 创建 `docs/dma/` 下的 K3 CoM260 DMA 与内存所有权主题文档，区分通用 DMA controller、设备内建 DMA、descriptor 与 data buffer。
- 整理 CPU/device ownership、完成可见性、cache line、clean/invalidate、barrier、map/unmap、物理地址与设备地址的来源支持和未知边界。
- 分层记录 PMA、PBMT、IOMMU 与地址转换；固定 revision 第三方实现只作为经验，不提升为 K3 官方硬件规范。
- 在引用前登记本 change 实际直接打开的 DMA、DTS、binding、driver、OpenSBI 或官方源码精确 URL。
- 复核 G5 与本 change 发现的独立未知项；更新 `docs/index.md` 的 DMA 入口和实际来源、缺口计数。

## Capabilities

### New Capabilities

- `k3-com260-dma-memory-ownership-baseline`：定义 K3 CoM260 DMA、cache、PMA/PBMT、IOMMU、地址转换和 CPU/device ownership 的来源分层、设备边界与未知项闭包。

### Modified Capabilities

- None.

## Scope Decisions

- 默认按“DMA 与 ownership”“cache、PMA/PBMT 与地址转换”拆成两篇正文；调查可调整文件边界，但不得删除任何验收主题。
- 通用 DMA controller、GMAC/UFS 等设备内建 DMA、AP/RP 共享内存分别记录，不把一类对象的 descriptor、寻址或一致性规则补到另一类。
- descriptor 与 data buffer、CPU ownership 与 device ownership、提交与完成可见性分别记录；只有直接来源支持时才给出转换顺序。
- Linux DMA API、DTS 属性、驱动行为、OpenSBI 设置和第三方经验按证据层分开，不将软件约定写成硬件规范。
- `fence`、cache clean/invalidate、PMA、PBMT 与 IOMMU 不互相替代；地址别名相同也不自动证明同一物理存储或一致属性。

## Scenario Gaps and Defaults

- 官网正文仍不可读时，保持官网行 `partially-observed`，只用官方 GitHub/DTS/driver 作 `交叉验证`，不代填官网修订。
- CoM260 Kit 默认顶层 DTS 未唯一映射时，只使用共享 K3/CoM260 字段；变体专有 DMA、IOMMU 或 reserved-memory 配置不写成默认板值。
- 缺少 K3 programmer reference 时，保留 G5；通用 RISC-V、Linux DMA API 或 Synopsys IP 行为不能补写 K3 cache line、coherency、地址宽度或 barrier 规则。
- Rt-Async-AMP 的 SRAM alias、OpenSBI PMA 修改、`fence iorw,iorw`、GMAC/UFS cache maintenance 只属于固定 revision 行为；没有真板结果时不声明硬件效果成立。
- IOMMU 节点、驱动或启用证据不足时记录为未知，不由 DMA 可工作反推 identity mapping、bypass 或不存在 IOMMU。
- 来源冲突、对象边界无法确定或关键字段缺失时，保留并登记未知项，不设计 Rust DMA API、驱动修复或验证工具。
- 本仓库没有运行时并发、取消或超时执行；这些只作为 ownership 竞争、完成等待和设备 reset/recovery 的资料边界记录。

## Scenario Sketch

- **通用 DMA 传输**：给定官方文档、DTS、binding 或 driver 可定位 controller、channel、descriptor 和能力字段，按来源记录 burst、宽度、scatter-gather/cyclic 与完成路径；缺字段时保持未知。
- **设备内建 DMA**：给定 GMAC、UFS 或其他设备自带 DMA，单独记录其 ring/UTP、ownership 和 completion，不把设备专有格式写成通用 DMA controller 规则。
- **CPU 交给设备**：来源明确 buffer/descriptor 准备、cache maintenance 和 barrier 顺序时，记录 ownership 转换与可观察完成条件；顺序不完整时禁止补写。
- **设备归还 CPU**：来源明确 IRQ、poll、状态位或 descriptor 回收时，记录完成可见性、invalidate/read barrier 与错误边界；不得由通知到达推定数据已可见。
- **PMA/PBMT 与地址别名**：给定 OpenSBI、DTS 或驱动可定位窗口与属性，记录设置范围、执行阶段和适用 hart；缺少读回或真板双向验证时保留硬件效果未知。
- **IOMMU 与设备地址**：给定节点、驱动或映射路径时，区分 CPU 物理地址、设备地址和 IOVA；证据不足时不推定 identity mapping、bypass 或 coherency。
- **错误、超时与恢复**：来源包含 DMA error、timeout、reset 或设备重启时，记录 buffer/descriptor ownership 的恢复条件；缺少安全回收依据时标为未知。
- **来源变化或中止**：官方分支、节点或字段变化影响 ownership/coherency 模型时停止使用旧调查结论，返回 Plan；普通不可访问按既有 fallback 保存未知项。

## Non-goals

- 不设计或实现 Rust DMA API、DMA allocator、IOMMU、cache maintenance、GMAC、UFS 或其他设备驱动。
- 不修改 Linux、DTS、OpenSBI、bootloader、StarryOS、Rt-Async-AMP 或 tgoskits 产品代码。
- 不用 QEMU、第三方板卡、通用 RISC-V 或 Linux API 行为证明 CoM260 真板一致性。
- 不把共享内存一致性直接等同于 GMAC、UFS 或其他设备 DMA 一致性。
- 不展开 GMAC PHY/MDIO、UFS 协议或 AMP RPC 数据面；只记录与 DMA ownership、寻址和完成可见性直接相关的边界。
- 不建立抓取器、专用验证器、运行身份、manifest、Hash 或 Evidence 占位。

## Impact

- 预计新增两篇 `docs/dma/` 主题文档，并修改来源覆盖、known-gaps（条件性）和总入口。
- OpenSpec change 保存需求、调查、设计、任务、Cycle 与验证结果；仓库继续保持纯 Markdown。
- MS07–MS09 可引用本 change 的 ownership、coherency 和地址转换边界，不必从设备驱动行为重新推导全局规则。

## Gate 1

- Status: approved
- User approval: `批准`

| Check | Status | Evidence |
| --- | --- | --- |
| BDD gap scan | PASS | 覆盖通用与设备内建 DMA、ownership 双向转换、PMA/PBMT、IOMMU、错误恢复、来源变化和不可访问 |
| Gap decisions | PASS | 用户接受 Scenario Gaps and Defaults |
| Scenario sketch | PASS | 本 proposal 给出正常、冲突、失败和停止路径 |
| OpenSpec change | PASS | `establish-k3-com260-dma-memory-ownership-baseline` 已创建 |
| Requirements and scope approval | PASS | 用户原话：`批准` |
