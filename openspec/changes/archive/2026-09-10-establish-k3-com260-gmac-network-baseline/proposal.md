## Why

MS04–MS06 已分别建立平台控制、中断和 DMA/内存所有权基线，但 CoM260 的 GMAC、MDIO、PHY、RGMII、descriptor ring 与设备中断仍散落在板级文档、官方 DTS 和固定 revision 第三方材料中。MS07 需要把这些事实按来源等级和设备边界聚合，避免后续把 SoC 能力、Linux 配置、第三方驱动行为或历史真板结果混为同一层证据。

本 change 是新主题聚合，不是 source refresh 或已有正文 restructure。权威入口为 R01；最近观察到的源端修订日期为 2026-09-08。官方 GitHub 文档、DTS、binding 和驱动只作交叉验证。

## What Changes

- 新建 `docs/network/com260-gmac-phy.md`，整理 K3 GMAC 实例、CoM260 模组与 Kit 的网络连接、MMIO、clock/reset、pinctrl、APMU glue、MDIO、PHY、RGMII、reset 和 delay-line 事实。
- 新建 `docs/network/k3-gmac-dma-irq.md`，整理 DWMAC5 的 MAC/MTL/DMA 分层、descriptor ring、buffer ownership、DMA 地址宽度、cache maintenance、doorbell、IRQ cause、mask/ack、reclaim 与恢复边界。
- 更新 `docs/index.md`、`docs/reference/source-coverage.md`、`docs/reference/known-gaps.md` 和必要的术语入口，使两篇网络文档可检索，来源覆盖和未知项保持唯一权威位置。
- 严格区分官网事实、官方 GitHub 交叉验证、固定 revision 第三方行为、推论和未知项；不把第三方 DHCP/ping 历史结果写成本项目验证结果。
- 保留 CoM260 Kit 目标 DTS 尚未唯一映射的边界，不用相邻板型、通用 DWMAC 行为或未核对的 PHY 型号填补缺口。

## Requirements and Scenario Sketch

### R1 — 网络主题来源可追溯

- **S1 Happy Path**：给定 R01、官方 GMAC 页面及已登记的官方 GitHub来源，执行聚合后，两篇主题文档首行列出全部直接来源、源端修订或 `unknown`、观察日期，且每个 URL 在覆盖表中恰有一行。
- **S2 Sad Path**：给定官网页面仍只暴露 SPA 壳或某个 raw URL 不可访问，执行聚合时不得把不可访问内容写成已观察事实；文档记录访问边界，并以已登记的官方 GitHub 材料作交叉验证。
- **S3 Edge Case**：同一事实在官网、DTS、Linux/U-Boot 或第三方材料之间不一致时，文档分别记录来源、适用对象和冲突，不静默选值。

### R2 — GMAC 与板级链路分层

- **S4 Happy Path**：给定 K3 SoC、CoM260 模组、Kit 和官方 DTS 材料，读者可从 `com260-gmac-phy.md` 定位 SoC GMAC 能力、实际引出通道、启用实例、MMIO、IRQ、clock/reset、pinctrl 和 APMU 依赖。
- **S5 Compatibility Boundary**：给定多个 CoM260 DTS 候选，文档分别记录基础款、Kit V02 和其他候选的已观察字段；没有唯一映射证据时不指定默认 Kit DTS。
- **S6 Error Boundary**：缺少寄存器手册、原理图或完整 binding 时，相应数值保持未知，并给出当前证据、禁止推断、解除条件和影响主题。

### R3 — MDIO、PHY 与 RGMII 关系明确

- **S7 Happy Path**：给定 `eth1`、MDIO 子节点和 PHY 节点，文档说明 PHY handle、Clause 22 地址、compatible、interface mode、reset GPIO/delay 与 RGMII phase 的静态关系。
- **S8 Edge Case**：PHY ID 字符串可定位厂商或候选器件，但缺少 datasheet 或板级证据时，不声明完整型号、扩展寄存器、内部 delay 或 EEPROM/strap 行为。
- **S9 Runtime Boundary**：静态 DTS 能证明配置输入，但不能证明 link、自协商、运行时重协商或 cable unplug/replug 已在本项目验证。

### R4 — DMA ring、IRQ 与 ownership 可诊断

- **S10 Happy Path**：读者可从 `k3-gmac-dma-irq.md` 追踪 CPU 准备 descriptor/buffer、cache clean、OWN 交接、tail doorbell、DMA in-flight、IRQ/轮询完成、invalidate 与回收的状态变化。
- **S11 Error Path**：TX/RX descriptor error、buffer unavailable、fatal bus error、reset 或 link-down 发生时，文档分别说明已证行为和未知恢复语义，不用统一 OWN 操作替代设备专有恢复。
- **S12 Concurrency Edge**：固定 revision 第三方 IRQ handler 在 `try_lock` 失败时返回空事件；文档记录其潜在推进风险和解除条件，不把它写成已验证丢中断，也不宣称等待下一 IRQ 必然安全。

### R5 — 导航、术语和缺口保持一致

- **S13 Happy Path**：两篇新文档从 `docs/index.md` 可达，GMAC/PHY/MDIO/RGMII/DMA/IRQ 术语与既有文档一致，相关 G3/G5/G7 及新发现缺口不重复登记。
- **S14 Regression Boundary**：更新网络主题时，boot、platform、interrupts、DMA 和 reference 文档中的既有事实、来源等级和职责边界保持不变；只修改建立网络入口所必需的交叉引用、计数和状态。
- **S15 Scope Boundary**：材料涉及 EtherCAT、网络栈或异步 NIC 时，只记录与 GMAC 物理通路或后续依赖有关的边界，不展开这些主题的正文或实现。

## Default Assumptions Requiring Gate 1 Approval

- 产出拆为 `com260-gmac-phy.md` 与 `k3-gmac-dma-irq.md` 两篇；若调查证明内容能在不损失诊断边界的情况下保持单篇 ≤500 行，Plan 可在 Gate 2 前提出合并，但不能由 Act 自行决定。
- 官方 GitHub `docs-buildroot`、Linux和 U-Boot 材料为交叉验证；tgoskits 与其真板调试报告为固定 revision 第三方证据。
- 当前 change 不要求取得 CoM260 原理图、PHY datasheet 或真板；缺失材料转为带解除条件的未知项。
- Persisted Evidence 默认 `none`；Markdown 内容、链接、结构和 OpenSpec 验证可在 Act Response 中低成本重跑并报告。

## Non-goals

- 不实现、修改或评审 K3 GMAC、MDIO、PHY、DMA、IRQ、网络栈或异步 NIC 代码。
- 不运行 CoM260 真板、QEMU 网络或第三方仓库测试，不以模拟结果证明硬件行为。
- 不聚合 K3 之外的板卡，不用 K1、Pico 或通用 DesignWare 默认值补齐 CoM260 字段。
- 不展开 EtherCAT、TSN 协议、TCP/IP、DHCP、socket 或用户态网络行为。
- 不把 Linux DMA API、DWMAC 通用手册或第三方源码行为提升为 K3 官方规范。
- 不调整 MS07–MS11 roadmap，不同步 SNAPSHOT、全局 tasks 或 M/D/K/R/I。

## Capabilities

### New Capabilities

- `k3-com260-gmac-network-baseline`：定义 CoM260 GMAC/MDIO/PHY/RGMII 与 DWMAC5 DMA ring/IRQ 知识包的来源、分层、未知项和一致性要求。

### Modified Capabilities

- 无。

## Impact

- 新增：`docs/network/com260-gmac-phy.md`、`docs/network/k3-gmac-dma-irq.md`。
- 按需更新：`docs/index.md`、`docs/reference/source-coverage.md`、`docs/reference/known-gaps.md`、`docs/reference/terminology.md`。
- 计划和执行产物位于 `openspec/changes/establish-k3-com260-gmac-network-baseline/`。
- 不新增代码、脚本、构建依赖或外部运行时。

## Gate 1

- Status: approved
- User approval: “批准”（2026-09-09）
- Approved scope: R1–R5、S1–S15、默认假设和 Non-goals。
