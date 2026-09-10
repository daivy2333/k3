## Context

MS04–MS06 已提供平台控制、APLIC/IMSIC、DMA ownership 和 cache/PMA/IOMMU 边界。网络主题仍缺少把这些前置基线连接到 CoM260 `eth1`、MDIO/PHY/RGMII 和 DWMAC5 MAC/MTL/DMA 路径的长期正文。

当前工作区在 revision `b484827` 上带有 MS06 收尾的既有未提交修改。`docs/index.md` 与 `source-coverage.md` 当前记录 69 个唯一 URL，SNAPSHOT 仍记录 68 个；本 change 不修正 SNAPSHOT，也不假定最终来源总数。`docs/network/` 和两篇目标文档尚不存在。

可复用证据分为四层：R01 官网入口与其 GMAC 页面；SpacemiT 官方 GitHub 的 `docs-buildroot`、Linux DTS/binding/driver 和 U-Boot 实现；固定 revision tgoskits K3 GMAC 驱动；固定 revision 第三方真板调试报告。后两层只说明对应代码或历史环境，不证明本项目真板行为。

## Goals / Non-Goals

**Goals**

- 建立 K3 SoC、CoM260 模组、Kit 和 DTS 变体之间的 GMAC/PHY 资源关系。
- 建立 GMAC → MDIO → PHY → RGMII 的静态配置链和运行时未知边界。
- 建立 MAC/MTL/DMA、descriptor/data buffer ownership、doorbell、IRQ 和回收状态链。
- 让来源覆盖、G3/G4/G5/G6/G7、术语与总入口保持一致且不重复承载事实。

**Non-Goals**

- 不实现或修改驱动、网络栈、异步接口、DMA、IRQ 或设备树。
- 不运行真板或以 QEMU 结果证明 CoM260 硬件。
- 不展开 EtherCAT、TSN 协议、TCP/IP、DHCP 或 socket 行为。
- 不把通用 DWMAC 或第三方实现提升为 K3 官方寄存器规范。
- 不同步 SNAPSHOT、全局 tasks 或 M/D/K/R/I。

## Decisions

### D1：按板级链路与数据面状态拆为两篇文档

`docs/network/com260-gmac-phy.md` 承载实例、平台资源、MDIO、PHY、RGMII、reset 和 delay-line；`docs/network/k3-gmac-dma-irq.md` 承载 MAC/MTL/DMA、descriptor ring、cache/ownership、IRQ 和恢复。

两篇文档分别依赖平台/板级来源和 DMA/IRQ/第三方实现来源，失败时的诊断范围也不同。替代方案是合并为单篇；这会让目标 DTS 与 PHY 证据缺口阻塞独立可交付的数据面状态基线，因此不采用。

### D2：来源按权威性和观察对象分层

官网 R01 及 GMAC 页面维持权威入口；SpacemiT 官方 GitHub 内容标为交叉验证；tgoskits 代码和真板报告标为固定 revision 第三方；跨来源归纳标为推论。每个结论同时标出适用对象，例如 K3 SoC、CoM260 shared base、Kit V02 或第三方 IFX DTS。

替代方案是以“官方仓库”统一覆盖官网与 GitHub。该做法会破坏 M01 和现有四级证据规则，因此不采用。

### D3：不在本 change 选择默认 CoM260 Kit DTS

基础款、Kit V02 和第三方 IFX DTS 分栏记录。共享字段可以归纳为 CoM260 已观察共性；变体字段必须携带变体名称。G7 未解除前，不把名称最接近或第三方正在使用的 DTS 写成默认 Kit 配置。

替代方案是选择 `k3_com260_kit_v02.dts` 或第三方 IFX DTS 作为工作基线。用户指南产品版本与 DTS 命名尚不一致，选择会引入未经证实的板级结论，因此不采用。

### D4：PHY 标识与 PHY 行为分开

`ethernet-phy-id001c.c916`、Clause 22 地址 1、`phy-handle`、reset GPIO/delay 和 RGMII phase 作为静态 DTS 输入记录。具体商品型号、扩展寄存器、strap/EEPROM、internal delay 和运行时 link 状态只在有对应 datasheet、板级材料或运行证据时记录。

替代方案是根据 PHY ID 或第三方报告直接写 RTL8211F。现有证据不足以闭合器件变体和板级装配，因此不采用。

### D5：以 ownership 状态机组织 GMAC 数据面

TX/RX 都使用“CPU 准备 → cache clean/publish → OWN 交给 DMA → tail doorbell → in-flight → IRQ/轮询观察 → invalidate/完成检查 → CPU 回收”。descriptor 与 data buffer 分开记录；MAC、MTL 和 DMA 的配置及错误也分层记录。

替代方案是按寄存器偏移罗列。G6 尚未提供完整 programmer reference，且寄存器表无法表达 cache 可见性、所有权和回收边界，因此不采用。

### D6：IRQ 通知不替代确定性推进

官方 DTS 只用于记录静态 wired source 与 APLIC/IMSIC 路径。固定 revision tgoskits 的 W1C status、TX/RX reclaim、事件返回和 `try_lock` 行为作为第三方控制流记录；锁竞争失败后的后续推进保持未知，解除条件是完整 worker recheck 契约或真板压力证据。

替代方案是把“等待下一 IRQ”写成安全保证。当前材料没有证明最后一次事件后必有新中断，因此不采用。

### D7：缺口只更新既有权威条目

G3 承载 CoM260 GMAC/PHY 板级详情，G4 承载 AP interrupt delivery，G5 承载 coherency/IOMMU，G6 承载 GMAC programmer reference，G7 承载默认 DTS 映射。只有发现不属于这些对象且有独立解除条件的问题时才新增 G 条目。

替代方案是为 MDIO、RGMII、descriptor 和 IRQ 各建缺口。它会复制 G3/G4/G5/G6 的解除条件，因此不采用。

## Data and Dependency Flow

```text
K3 SoC capability
  → CoM260 module signal/PHY1 exposure
  → CoM260 DTS variant
  → eth1 platform resources
  → APMU clock/reset + pinctrl
  → RGMII + MDIO + PHY reset
  → PHY negotiation/link input
  → MAC + MTL + DMA channel
  → TX/RX descriptor and buffer ownership
  → APLIC source → IMSIC → worker/reclaim
```

第一篇文档闭合 `SoC → PHY/link input`；第二篇闭合 `MAC/MTL/DMA → IRQ/reclaim`。第二篇可以引用第一篇已确认的输入，不复制 PHY 和板级正文。

## Risks / Trade-offs

- [官网正文仍是 SPA 壳] → 保持 `partially-observed`，只对实际打开的官方 GitHub 内容使用交叉验证等级。
- [官方 Linux/U-Boot 精确文件路径或字段变化] → Act 在引用前直接打开；若变化改变 D1–D7 或 Acceptance，停止并返回 Plan。
- [当前 69 URL 与 SNAPSHOT 68 URL 不一致] → 以执行时覆盖表实际唯一计数为准，只做增量更新；不在本 change 刷新 SNAPSHOT。
- [第三方驱动含未经独立核对的寄存器常量] → 只写代码行为和历史故障，不用它关闭 G6。
- [两篇文档发生重复] → 第一篇负责配置输入，第二篇负责运行时数据路径；相邻事实使用相对链接。
- [文档超过 M02 建议上限] → 每篇目标 ≤450 行；若预计超过 500 行且需新增文档边界，返回 Plan，不由 Act 自行扩展。

## Migration Plan

Iteration 000 核对第一篇正文的来源覆盖并建立 `com260-gmac-phy.md`，形成可独立引用的板级链路。该 Iteration Review 为 `accepted` 后，Iteration 001 先核对第二篇正文的实际来源，再建立 `k3-gmac-dma-irq.md` 并同步缺口、入口和必要术语。回滚时删除本 change 新建的网络文档并还原本 change 对 reference/index 的精准修改；不得覆盖 MS06 或用户其他改动。

## Open Questions

没有阻塞 Gate 2 的实质问题。PHY 实物型号、默认 Kit DTS、完整 GMAC programmer reference、cache coherency/IOMMU 和 IRQ 最终投递均作为受 G3–G7 约束的未知项，不交给 Act 决定契约语义。
