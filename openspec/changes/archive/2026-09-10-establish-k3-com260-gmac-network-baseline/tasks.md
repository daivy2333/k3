## 1. Iteration 000 — GMAC、MDIO、PHY 与 RGMII 板级基线

- [x] 1.1 [T1] 在 `docs/reference/source-coverage.md` 核对并登记 `docs/network/com260-gmac-phy.md` 实际引用且已直接打开的精确官方 URL，保持既有 69 行及其状态并按实际增量同步唯一 URL 数。
- [x] 1.2 [T2] 创建 `docs/network/com260-gmac-phy.md`，分层整理 K3 GMAC 能力、CoM260 模组/Kit/DTS 变体、平台资源、MDIO、PHY、RGMII、reset 与 delay-line。

## 2. Iteration 001 — DWMAC5 数据面、IRQ 与主题收尾

- [x] 2.1 [T3] 在 `docs/reference/source-coverage.md` 核对并登记 `docs/network/k3-gmac-dma-irq.md` 实际引用且已直接打开的精确官方 URL，只按实际增量同步唯一 URL 数。
- [x] 2.2 [T4] 创建 `docs/network/k3-gmac-dma-irq.md`，整理 MAC/MTL/DMA、descriptor/data buffer ownership、cache、doorbell、IRQ、回收和错误恢复边界。
- [x] 2.3 [T5] 更新 `docs/reference/known-gaps.md` 的 G3–G7 相关证据、状态汇总和对应关系；只按实际解除字段调整状态，不创建重复缺口。
- [x] 2.4 [T6] 更新 `docs/index.md` 的网络正文入口、主题职责、来源/缺口计数和状态，并验证既有术语无需修改或仅报告不一致后返回 Plan。

## Task Contracts

### T1：精确来源登记

- Requirement/Scenario: R1/S1-S3；R2/S4-S6；R5/S13-S14。
- Depends on: None。
- Targets: `docs/reference/source-coverage.md`。
- Current behavior: 覆盖表有 69 个 URL 且 69 个唯一值；已登记 09-GMAC raw、`k3.dtsi`、`k3_com260.dtsi`、`k3_com260.dts` 和 `k3_com260_kit_v02.dts`，尚未按 MS07 正文实际引用核对官方 GMAC binding/glue driver 的精确 URL。
- Required behavior: 直接打开正文所需来源；既有 URL 只扩充 network 职责，不重复建行；新 URL 仅在可打开并被正文实际引用时登记；同步表内实际总数和唯一数。
- Required changes: 核对 R01/09-GMAC、既有 CoM260/K3 DTS；检查 linux-6.18 `k3-br-v1.0.y` 的 `Documentation/devicetree/bindings/net/snps,dwmac.yaml`、相关 stmmac platform binding 和 `drivers/net/ethernet/stmicro/stmmac/dwmac-spacemit-ethqos.c` 候选。候选不存在或不可访问时不登记，正文和 G6 保留缺失边界。
- Preserve: 既有 69 行的 URL 身份、来源等级、观察日期和状态；M01、D04-D07；官网 SPA 壳保持 `partially-observed`。
- Forbidden: 不登记未引用候选；不批量刷新既有来源；不把 tgoskits 或真板报告登记为官方来源；不修改 `others/`。
- Test witness: 变更前运行 URL 行数/唯一数检查，预期 `69/69`；检查候选 URL 尚未登记。
- GREEN condition: `com260-gmac-phy.md` 的全部直接官方 URL 在覆盖表各出现一次；新增行均有类型、范围、主题、优先级、聚合状态、修订、观察日期、访问状态和备注；实际总数等于唯一数。
- Verification: 用 `awk` 提取 URL 后比较总数与 `sort -u` 计数；以正文首行 URL 反向核对覆盖表；`git diff --check`；`openspec validate establish-k3-com260-gmac-network-baseline --strict`。
- Stop when: 官方来源变化改变 D1-D7、目标 DTS 范围或文档 Acceptance；或需要引入 M01/R08 之外的新权威来源。

### T2：GMAC、MDIO、PHY 与 RGMII 事实包

- Requirement/Scenario: R1/S1-S3；R2/S4-S6；R3/S7-S9；R5/S13-S15。
- Depends on: T1。
- Targets: `docs/network/com260-gmac-phy.md`。
- Current behavior: 文件和 `docs/network/` 不存在；SoC/模组能力、`eth1`/PHY 字段和目标 DTS 边界散落于 `k3-soc-overview.md`、`com260-board-resources.md`、`com260-image-and-dts.md` 与 G3/G7。
- Required behavior: 读者可追踪 K3 SoC → CoM260 模组 → Kit/DTS 变体 → `eth1` → platform resources → MDIO → PHY → RGMII；每项结论带适用对象和四级证据。
- Required changes: 建立能力/实例矩阵、基础款与 Kit V02 变体表、MMIO/IRQ/clock/reset/pinctrl/APMU 字段表、MDIO/PHY/RGMII 静态链、运行时边界和四字段未知项。PHY ID、地址 1、reset GPIO/delay、phase 47/53 只按直接来源适用范围记录。
- Preserve: G3/G4/G5/G6/G7 的权威职责；目标 DTS 未唯一映射；相邻文档事实；简体中文和现有术语；文档 ≤450 行。
- Forbidden: 不指定默认 Kit DTS；不由 PHY ID 声明完整型号/寄存器；不把第三方 link/DHCP/ping 结果写成本项目验证；不展开 DMA ring、IRQ handler、EtherCAT、TSN 或驱动设计。
- Test witness: `test ! -e docs/network/com260-gmac-phy.md` 退出码 0；`docs/index.md` 当前把 network 标为待聚合。
- GREEN condition: S1-S9 和 S13-S15 的板级部分均有事实、冲突或明确未知原因；全部直接来源已登记；相对链接有效；无跨证据等级提升。
- Verification: 首行来源格式、章节/矩阵、证据标签、G3/G7 交叉引用、目标 DTS 禁止断言、相对链接、行数、scoped diff、`git diff --check` 和 strict OpenSpec validate。
- Stop when: 资料证明当前技术目标板不是本 change 覆盖的 CoM260 范围，或两篇边界无法维持且需修改 design/Acceptance。

### T3：第二篇正文的精确来源登记

- Requirement/Scenario: R1/S1-S3；R4/S10-S12；R5/S13-S14。
- Depends on: Iteration 000 Review Result `accepted`。
- Targets: `docs/reference/source-coverage.md`。
- Current behavior: 覆盖表已经核对第一篇网络正文的来源；第二篇正文尚不存在，其实际直接来源和可能新增的官方 binding/glue URL 尚不能确定。
- Required behavior: 直接打开第二篇正文所需来源；既有 URL 只扩充 network 职责，不重复建行；新 URL 仅在可打开并将被第二篇正文实际引用时登记；同步实际总数和唯一数。
- Required changes: 核对既有 GMAC/DTS 来源，并检查第二篇所需的官方 binding、glue driver 或其他精确来源；候选不存在或不可访问时不登记，正文和 G6 保留缺失边界。
- Preserve: T1 已核对的来源身份、来源等级、观察日期和状态；M01、D04-D07；官网 SPA 壳保持 `partially-observed`。
- Forbidden: 不登记未引用候选；不批量刷新既有来源；不把固定 revision 第三方材料登记为官方来源；不修改 `others/`。
- Test witness: 第二篇正文不存在，无法在 Iteration 000 证明其全部直接 URL 已登记。
- GREEN condition: 第二篇正文计划使用的全部直接官方 URL 在覆盖表各出现一次；新增行元数据完整；实际总数等于唯一数。
- Verification: URL 总数与唯一数、第二篇首行反向核对、`git diff --check` 和 strict validate。
- Stop when: 新来源改变 D1-D7、目标范围或 Acceptance，或需要引入 M01/R08 之外的新权威来源。

### T4：DWMAC5 DMA ring 与 IRQ 事实包

- Requirement/Scenario: R1/S1-S3；R4/S10-S12；R5/S13-S15。
- Depends on: T3。
- Targets: `docs/network/k3-gmac-dma-irq.md`。
- Current behavior: 文件不存在；`k3-dma-and-memory-ownership.md` 只提供跨设备 ownership 抽象，R11/tgoskits 与第三方报告保存固定 revision GMAC 实现和历史结果，G6 保留 programmer reference 缺口。
- Required behavior: 分开 MAC、MTL、DMA channel、TX/RX descriptor 和 data buffer；完整描述准备、发布、OWN、tail、in-flight、完成观察、invalidate、回收、IRQ 与错误/恢复边界。
- Required changes: 建立 MAC/MTL/DMA 分层表、TX/RX 状态机、descriptor/data buffer 对照、DMA 地址/对齐/cache 边界、IRQ cause/mask/W1C/reclaim 路径、`try_lock` 推进风险、MTL/soft-reset 历史故障经验和未验证项。
- Preserve: MS06 ownership/cache 边界；G4/G5/G6；固定 revision `tgoskits 19219411d` 与 Rt-Async-AMP `ccb1ff0`；文档 ≤450 行。
- Forbidden: 不把通用 DWMAC 常量或第三方寄存器表写成 K3 官方规范；不声明 cache line、IOMMU、IRQ delivery 或 reset recovery 已由本项目真板验证；不设计异步 NIC。
- Test witness: `test ! -e docs/network/k3-gmac-dma-irq.md` 退出码 0；现有 network 主题没有专门数据面正文。
- GREEN condition: S10-S12 的正常、错误和并发路径均可从文档独立追踪；固定 revision 与证据边界明确；G4/G5/G6 未被无依据关闭。
- Verification: 首行来源、状态机、分层表、revision、四级证据、四字段未知项、相对链接、行数、scoped diff、`git diff --check` 和 strict OpenSpec validate。
- Stop when: 官方材料与固定 revision 第三方路径发生影响状态机或错误语义的实质冲突，或需要修改 MS06 的既有 ownership 模型。

### T5：G3–G7 收敛且不重复

- Requirement/Scenario: R2/S6；R3/S8-S9；R4/S11-S12；R5/S13-S14。
- Depends on: T2、T4。
- Targets: `docs/reference/known-gaps.md`。
- Current behavior: G3/G5 为 `partial`，G4/G6/G7 为 `open`；G3 已记录 `eth1`、PHY ID、地址和 RGMII 字段，G6 仍缺 programmer reference，G7 仍缺默认 DTS 映射。
- Required behavior: 按两篇正文实际新增的官方证据更新 G3–G7 当前证据与未解字段；只有解除条件部分满足时才改为 `partial`，全部满足才 `closed`；汇总和对应关系同步。
- Required changes: 记录 network 文档路径与新来源；对 PHY/MDIO、coherency、IRQ delivery、programmer reference 和 DTS 映射逐项判断，不以第三方材料解除官方缺口。
- Preserve: G1-G2/G8-G10 内容和状态；历史状态变更；每项四字段结构。
- Forbidden: 不为已有对象新建重复 G；不凭源码存在、第三方 DHCP/ping 或通用 DWMAC 行为关闭缺口；不删除历史条目。
- Test witness: 变更前 G3/G5=`partial`、G4/G6/G7=`open`，且尚未引用本 change 的两篇正文。
- GREEN condition: G3–G7 与正文完全一致，状态变化都有直接官方证据和日期；汇总及 source-coverage 对应关系无矛盾。
- Verification: `rg` 检查编号/状态/四字段/正文链接；人工逐项比对解除条件；`git diff --check` 和 strict validate。
- Stop when: 新发现需要新增独立缺口、改变长期模型/决策或扩大 milestone 范围。

### T6：网络入口、计数与术语一致

- Requirement/Scenario: R5/S13-S15。
- Depends on: T1-T5。
- Targets: `docs/index.md`。
- Current behavior: `docs/index.md` 把 network 标为“待聚合”；当前显示 69 个唯一 URL、10 个缺口和 20 个基础术语；术语表已经定义 GMAC、PHY、MDIO、RGMII、DMA、IRQ。
- Required behavior: 索引链接两篇实际存在的网络正文并标明对应 Iteration；来源和缺口计数从权威文件实际值同步；network 主题职责与交付状态一致。
- Required changes: 增加 `docs/network/` 正文链接，更新 network 状态、实际 URL 计数和缺口状态摘要；验证既有术语足够，发现需要新主写法时停止返回 Plan 而不直接修改 `terminology.md`。
- Preserve: 其他主题链接、职责和状态；入口页不复制技术事实；MS08-MS11 状态；既有术语正文。
- Forbidden: 不同步 SNAPSHOT/tasks；不修改 `terminology.md`；不硬编码计划中的来源总数替代实际计数；不展开 EtherCAT/TSN/网络栈。
- Test witness: 两个 network 链接不存在，主题表仍标待聚合。
- GREEN condition: 两个链接可解析，network 状态与任务完成一致，URL/缺口/术语数字与权威文档一致，无关主题行未改。
- Verification: 相对链接解析、URL 唯一计数、G 汇总计数、术语行计数、scoped diff、`git diff --check` 和 strict validate。
- Stop when: T1-T5 未 GREEN，或正文需要术语表尚未定义的新主写法。

## Iteration Plan

### Iteration 000：GMAC、MDIO、PHY 与 RGMII 板级基线

- Tasks: T1-T2。
- Depends on: MS04、MS05、MS06 已完成。
- Stable baseline: K3/CoM260 GMAC 实例、平台资源和 PHY/RGMII 静态配置链可独立引用，后续数据面文档无需重新选择目标 DTS 或证据等级。
- Verification boundary: 正文直接来源唯一登记；SoC/模组/Kit/DTS 变体分层；`eth1` 到 PHY 的静态链、冲突和未知项完整。
- Diagnostic boundary: 来源身份、DTS 变体、GMAC实例、platform resources、MDIO/PHY/RGMII、reset/delay-line。
- Non-goals: MAC/MTL/DMA 状态机、IRQ handler、G3–G7 状态收尾和总入口更新。
- Balance audit: T1 为 T2 的来源前置，二者共同形成可独立验收的板级知识包；数据面和 IRQ 属不同来源与故障域，留到下一 Iteration。

### Iteration 001：DWMAC5 数据面、IRQ 与主题收尾

- Tasks: T3-T6。
- Depends on: Iteration 000 Review Result `accepted`。
- Stable baseline: MS07 两篇主题文档、来源覆盖、缺口和导航一致，可供后续 K3 驱动或 AMP/EtherCAT 规划引用。
- Verification boundary: MAC/MTL/DMA 与 TX/RX ownership/IRQ 路径完整；G3–G7 和 index 与实际证据、来源及交付状态一致。
- Diagnostic boundary: descriptor/data buffer、cache/doorbell、IRQ/reclaim、error/reset、缺口状态、导航与计数。
- Non-goals: 驱动实现、真板验证、异步 NIC、EtherCAT/TSN/网络栈正文、全局状态同步。
- Balance audit: T3 是 T4 的来源前置；T4 形成第二个技术结果；T5-T6 依赖全部正文且只负责主题收尾，单独拆轮不能形成技术基线，合并后仍有清晰验证与诊断边界。

## Requirements Traceability Matrix

| Requirement | Scenario | Design | Task | Iteration | Code Surface | Test Witness | Simplification | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| R1 | S1-S3 | D2 | T1-T4 | 000,001 | coverage；两篇 network 文档 | 69/69 URL；目标文档不存在 | None | Covered |
| R2 | S4-S6 | D1,D3 | T1,T2,T5 | 000,001 | coverage；GMAC/PHY 文档；gaps | network 文档不存在；G3/G7 未闭合 | None | Covered |
| R3 | S7-S9 | D3,D4 | T2,T5 | 000,001 | GMAC/PHY 文档；gaps | PHY 事实散落，运行边界无专题 | None | Covered |
| R4 | S10-S12 | D5,D6 | T3-T5 | 001 | coverage；DMA/IRQ 文档；gaps | 数据面专题不存在；G4-G6 未闭合 | None | Covered |
| R5 | S13-S15 | D1,D7 | T2-T6 | 000,001 | 两篇 network 文档；gaps；index | network 待聚合；链接不存在 | None | Covered |

## Plan Completeness Review

- TBD/TODO: None。
- Requirement simplification: None。
- R1-R5、S1-S15 均映射到 design、task、Iteration、目标文件和测试见证。
- 两个 Iteration 各形成稳定基线，依赖有序；只允许展开 Iteration 000。
- 当前工作区的 69 URL 基线与 SNAPSHOT 的 68 URL 差异已记录，不纳入本 change 的 SNAPSHOT 修复。
- Persisted Evidence: `none`；Markdown、URL、链接、计数和 OpenSpec 验证均可低成本重跑并写入 Act Response。
