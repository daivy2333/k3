# Iteration 001 / Cycle 000：DWMAC5 数据面、IRQ 与主题收尾

## Plan Context

- Status: ready
- Iteration: 001-dma-irq-finish
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: None

**Iteration Scope**

- Change tasks: T3–T6
- Depends on: Iteration 000 / Cycle 001 Review Result `accepted`
- Stable baseline: MS07 两篇主题正文、来源覆盖、G3–G7 和总索引相互一致，可供后续驱动与 AMP/EtherCAT 规划引用
- Verification boundary: 第二篇正文的直接来源唯一登记；MAC/MTL/DMA、TX/RX ownership、IRQ/reclaim 和错误边界可独立追踪；缺口与导航按实际结果收尾
- Diagnostic boundary: 来源身份、descriptor/data buffer、cache/doorbell、IRQ/reclaim、error/reset、G3–G7 状态、导航与计数
- Deferred tasks: None

**Cycle Scope**

- Trigger: initial
- Acceptance gaps: None
- Repair items: None
- Inherited scope: proposal R1、R4、R5 的 S1–S3、S10–S15；design D1、D2、D5–D7；M01–M04、D02、D04–D07
- Excluded scope: 驱动实现或评审、真板/QEMU、异步 NIC、EtherCAT/TSN/网络栈正文、SNAPSHOT、全局 tasks、M/D/K/R/I 和 `others/` 修改

**Objective**

创建 `docs/network/k3-gmac-dma-irq.md`，用固定 revision 第三方代码说明 DWMAC5 数据面和 IRQ 控制流，同时保持官方证据与第三方行为分层；随后让来源覆盖、G3–G7 和总索引与两篇网络正文一致。

**Background**

Gate 1 已批准 R1–R5、S1–S15、默认假设和 Non-goals。Iteration 000 已建立 `com260-gmac-phy.md`，其最终 Review 为 `accepted`。本 Iteration 完成剩余 T3–T6，不改变既有 Iteration Map。

场景缺口已在 proposal 中覆盖：Happy Path 为 TX/RX ownership 和 IRQ/reclaim 闭环；Sad/Error Path 包含来源不可访问、descriptor error、buffer unavailable、fatal bus error、reset 与 link-down；Edge/Concurrency 包含多来源冲突和 IRQ `try_lock` 失败；兼容性边界保留默认 Kit DTS、G4/G5/G6 与真板未验证状态；取消与墙钟 deadline 不属于当前文档聚合目标，不从第三方轮询常量推导。

**Current Baseline**

- 主仓 revision 为 `b48482707990d10ca1ab7dc1fd36035e90f89e84`，branch `main`；工作区含 MS06 收尾、Iteration 000 和用户既有未提交内容。
- `source-coverage.md` 当前为 69 个 URL、69 个唯一值；09-GMAC、`k3.dtsi`、`k3_com260.dtsi`、`k3_com260.dts` 和 `k3_com260_kit_v02.dts` 已登记。
- `docs/network/com260-gmac-phy.md` 已存在，238 行；`docs/network/k3-gmac-dma-irq.md` 不存在。
- `known-gaps.md` 当前 G3/G4/G5=`partial`，G6/G7=`open`；G1–G2、G8–G10 不属于本轮修改范围。
- `docs/index.md` 当前记录 69 个唯一 URL、10 个缺口、20 个基础术语；network 仍未列入已聚合正文并标为待聚合。
- `terminology.md` 已定义 GMAC、PHY、MDIO、RGMII、DMA、IRQ，本轮不需要新主写法。
- 当前环境此前无法解析 `raw.githubusercontent.com`；不可访问候选不得登记或写成已观察事实。已登记来源的既有观察结果仍可复用，但新增事实必须来自当前可审计的本地材料或可直接打开的新来源。

**Current-State Evidence**

- 官方 09-GMAC 对应页和 K3/CoM260 DTS 已为 GMAC 内建 DMA、`eth1`、FIFO/TSO/store-and-forward 配置提供交叉验证入口；这些材料不构成 K3 programmer reference。
- `k3-dma-and-memory-ownership.md` 已定义 descriptor 与 data buffer 分离、CPU/device ownership、通知不替代可见性，以及 device-specific recovery 边界。第二篇正文应引用该模型，不复制通用章节。
- 固定 revision tgoskits `19219411d5dc1515496f910d04c93da12ee95be4` 的 `k3_gmac::probe` 依次处理 glue、MMIO/DMA mask、MAC/FIFO 配置和 core 初始化；该顺序只说明第三方实现。
- `core.rs::K3GmacCore` 持有 TX/RX ring、生产/回收索引、in-flight buffer 和 done queue。TX/RX 提交均在 descriptor 置 OWN 后 clean cache，再写 tail pointer 并重写 ST/SR；回收前 invalidate descriptor，观察 DMA 清 OWN 后归还 buffer。
- `desc.rs::DmaDesc` 使用四个 32-bit DWMAC4/5 字段并填充到 64 bytes；Release fence 只约束 CPU 写序，不执行 cache clean。64-byte cache line 是固定 revision 第三方实现假设，不能关闭 G5。
- `core.rs::handle_irq` 读取 DMA channel status；NIS/AIS 均未置位时返回空事件，否则按读取值 W1C、记录 FBE、回收 TX/RX，再按 TI/TBU、RI/RBU 或非空 done queue 生成事件。寄存器位布局来自第三方 `regs.rs`，不能关闭 G6。
- `queue.rs::K3GmacIrqHandler` 与数据面共享 `SpinNoIrq<K3GmacCore>`；硬中断 `try_lock` 失败即返回 `Event::none()`。源码注释假设等待下一中断，但没有 worker 重查或最后一次事件后的确定性推进证据。
- `reset_dma` 有固定迭代上限；失败时初始化返回错误。相关注释另记录历史上 U-Boot 残留状态、TBU/ST 重写和 MTL FTQ 残留风险。这些只能作为固定 revision 行为或历史故障线索，不是本项目真板结论。

**Relevant Code**

- `docs/reference/source-coverage.md`：直接官方 URL 与来源元数据的唯一登记处。
- `docs/network/k3-gmac-dma-irq.md`：本 Cycle 新建的数据面与 IRQ 主题正文。
- `docs/network/com260-gmac-phy.md`：第一篇网络正文与 platform/PHY 输入边界。
- `docs/dma/k3-dma-and-memory-ownership.md`、`docs/dma/k3-cache-pma-address-translation.md`：ownership、cache、PMA/IOMMU 上游边界。
- `docs/interrupts/k3-interrupt-and-time.md`：APLIC→IMSIC 静态拓扑边界。
- `docs/reference/known-gaps.md`：G3–G7 权威状态与解除条件。
- `docs/index.md`：网络入口、来源/缺口计数和主题状态。
- `others/Rt-Async-AMP/tgoskits/drivers/ax-driver/src/net/k3_gmac/{mod,core,desc,queue,regs}.rs`：固定 revision 第三方 probe、ring、ownership、IRQ 和错误路径；只读。

**Critical Path**

```text
已登记官方 GMAC/DTS 来源
  → 第二篇首行直接来源与 coverage 反向核对
  → MAC / MTL / DMA 分层
  → descriptor 与 data buffer 分开
  → CPU prepare → OWN → cache clean → tail/ST/SR → DMA in-flight
  → IRQ 或主动 reclaim → W1C / invalidate / completion check → CPU 回收
  → descriptor error / TBU / RBU / FBE / reset / link-down 边界
  → G3–G7 按直接官方证据判断状态
  → index 链接、计数和 network 状态同步
```

任何新来源若改变 ownership、IRQ ack、错误恢复或 G3–G7 解除条件，停止并返回 Plan。固定 revision 第三方行为不得提升为官方 K3 规范。

**Implementation Guidance**

先完成 T3 的来源核对，再按 MAC/MTL/DMA 分层和 TX/RX 状态机写 T4。正文应把 descriptor、data buffer、cache 操作和通知分别表达，并把 `try_lock` 风险写为未证的推进缺口。T5 只更新 G3–G7 的新增证据、状态汇总和映射；T6 最后依据实际文件与计数更新 index。术语不够时停止，不自行修改术语表。

**Behavioral Change**

当前仓库只有通用 DMA ownership 摘要和板级 GMAC/PHY 文档，读者仍需进入第三方源码拼接 DWMAC5 ring 与 IRQ 路径。完成后，第二篇正文提供可检索的数据面状态链；index 可达两篇网络正文，G3–G7 只按实际证据变化。不会新增运行行为、驱动接口或硬件验证结论。

**Change Surface**

| Task | Requirement/Scenario | File/Symbol | Current Responsibility | Planned Change |
| --- | --- | --- | --- | --- |
| T3 | R1/S1–S3；R4/S10–S12；R5/S13–S14 | `docs/reference/source-coverage.md` | 69 个唯一 URL，已有 GMAC/DTS 来源 | 核对第二篇直接官方来源；只登记可打开且实际引用的新 URL |
| T4 | R1/S1–S3；R4/S10–S12；R5/S13–S15 | `docs/network/k3-gmac-dma-irq.md` | 不存在 | 创建 MAC/MTL/DMA、TX/RX ownership、IRQ/reclaim 与错误边界正文 |
| T5 | R2/S6；R3/S8–S9；R4/S11–S12；R5/S13–S14 | `docs/reference/known-gaps.md::G3-G7` | G3–G5 partial，G6–G7 open | 按两篇正文的实际直接证据更新内容、汇总与对应关系 |
| T6 | R5/S13–S15 | `docs/index.md` | network 待聚合，无两篇正文入口 | 增加链接并同步实际 URL、缺口、术语计数和主题状态 |

**Task Contracts**

### T3：第二篇正文来源唯一登记

- Requirement/Scenario: R1/S1–S3；R4/S10–S12；R5/S13–S14。
- Depends on: Iteration 000 Review Result `accepted`。
- Targets: `docs/reference/source-coverage.md`。
- Current behavior: 覆盖表为 69/69；第二篇不存在，因而尚未完成其首行来源反向核对。
- Required behavior: 第二篇实际使用的每个直接官方 URL 在覆盖表恰有一行；新 URL 只有在本轮可直接打开且将被正文引用时才登记。
- Required changes: 核对 09-GMAC 和已有 K3/CoM260 DTS；按需检查官方 binding、glue driver或其他精确来源。候选不可访问或未引用时保持 coverage 不变，并在正文保留 G6 边界。
- Preserve: 既有 69 行的 URL、等级、日期和访问状态；Iteration 000 已确认的来源职责；官网 SPA 壳的 `partially-observed` 状态。
- Forbidden: 不预登记候选，不批量刷新，不把 tgoskits/分析/真板报告登记为官方来源，不修改 `others/`。
- Test witness: `test ! -e docs/network/k3-gmac-dma-irq.md` 当前退出码 0；覆盖表当前总数/唯一数为 69/69。
- GREEN condition: 第二篇首行全部直接官方 URL 各匹配 coverage 一行；新增元数据完整；总数等于唯一数。
- Verification: URL 总数/唯一数；首行 URL 反查；新增 URL 可访问且在正文使用；scoped diff；`git diff --check`；strict validate。
- Stop when: 新来源改变 D1–D7、状态机、目标范围或 Acceptance，或要求 M01/R08 外的新权威来源。

### T4：DWMAC5 数据面与 IRQ 事实包

- Requirement/Scenario: R1/S1–S3；R4/S10–S12；R5/S13–S15。
- Depends on: T3。
- Targets: `docs/network/k3-gmac-dma-irq.md`。
- Current behavior: 专题不存在；通用 ownership 文档未展开 DWMAC5 的 MAC/MTL/DMA、ring、IRQ 和错误控制流。
- Required behavior: 读者可分别追踪 TX/RX descriptor 与 data buffer 从 CPU 准备到 CPU 回收的状态；IRQ/主动 reclaim、正常/异常状态和 `try_lock` 风险按证据等级可诊断。
- Required changes: 建立 MAC/MTL/DMA 分层表；TX/RX 状态机；descriptor/data buffer 对照；地址宽度、对齐、cache 与 fence 边界；tail/ST/SR doorbell；status/mask/W1C/reclaim；descriptor error、TBU/RBU、FBE、soft reset、link-down 和历史 MTL/残留状态边界；至少一个包含当前证据、禁止推断、解除条件和影响主题的未知项集合。
- Preserve: MS06 ownership/cache/PMA/IOMMU 边界；第一篇的 PHY/platform 职责；G4/G5/G6；固定 revision `19219411d` 与 `ccb1ff0b`；正文 ≤450 行。
- Forbidden: 不把第三方 descriptor/寄存器位、64-byte cache line 或 DMA 地址宽度提升为 K3 官方规范；不声明真板 IRQ delivery、coherency、IOMMU、reset recovery 或 link 恢复已验证；不设计异步 NIC，不展开 EtherCAT/TSN/网络栈。
- Test witness: `test ! -e docs/network/k3-gmac-dma-irq.md` 退出码 0；现有 network 主题没有数据面正文。
- GREEN condition: S10–S12 的正常、错误和并发路径都能独立追踪；官方/第三方/推论/未知项分层；G4/G5/G6 未被无依据关闭；相对链接有效。
- Verification: 首行来源与 coverage；分层表、TX/RX 状态机、descriptor/buffer、cache/doorbell、IRQ/W1C/reclaim、error/reset、revision、四级证据和四字段未知项；`wc -l` ≤450；链接解析；scoped diff；`git diff --check`；strict validate。
- Stop when: 新官方材料与第三方状态机发生实质冲突，或需要改变 MS06 ownership 模型、术语主写法或 design/Acceptance。

### T5：G3–G7 收敛

- Requirement/Scenario: R2/S6；R3/S8–S9；R4/S11–S12；R5/S13–S14。
- Depends on: T2、T4。
- Targets: `docs/reference/known-gaps.md::G3-G7`、状态汇总、coverage 对应关系。
- Current behavior: G3/G4/G5 为 `partial`，G6/G7 为 `open`；尚未引用本 change 的两篇网络正文。
- Required behavior: G3–G7 与正文的直接官方证据、第三方边界和解除条件一致；只有解除条件部分满足才改 `partial`，全部满足才改 `closed`。
- Required changes: 增加两篇 network 文档路径和实际新增来源；逐项判断 PHY/MDIO、IRQ delivery、coherency/IOMMU、programmer reference、默认 DTS；同步状态汇总和对应关系。
- Preserve: G1–G2、G8–G10 的正文和状态；历史状态记录；每项四字段结构。
- Forbidden: 不新建重复缺口，不用第三方代码、DHCP/ping、通用 DWMAC 或文档整理本身关闭官方事实缺口，不删除历史。
- Test witness: 变更前 G3/G4/G5=`partial`、G6/G7=`open`，且未引用两篇 network 正文。
- GREEN condition: 正文与 G3–G7 无事实或状态矛盾；每次状态变化都有直接官方证据和日期；汇总、正文链接和 coverage 映射一致。
- Verification: `rg` 检查 G3–G7 状态、四字段、network 链接和汇总；逐项比对解除条件；scoped diff；`git diff --check`；strict validate。
- Stop when: 发现需要独立新缺口、长期模型/决策变化或 milestone 扩张。

### T6：网络入口和计数收尾

- Requirement/Scenario: R5/S13–S15。
- Depends on: T3–T5。
- Targets: `docs/index.md`。
- Current behavior: index 显示 69 URL、10 缺口、20 术语；没有 network 正文列表，主题表仍标待聚合。
- Required behavior: 两篇 network 链接有效；network 状态与本 Iteration 完成范围一致；URL、缺口和术语数字来自权威文件实际值。
- Required changes: 增加两篇正文链接；更新 network 主题职责/状态；同步实际 URL 数和缺口状态摘要。术语表足够时保持 20，不修改 `terminology.md`。
- Preserve: 其他主题链接、职责、状态与 MS08–MS11；入口页不复制技术事实。
- Forbidden: 不修改 SNAPSHOT、全局 tasks 或 `terminology.md`；不硬编码计划值；不展开 EtherCAT/TSN/网络栈。
- Test witness: 两个 network 链接当前缺失，network 状态为待聚合。
- GREEN condition: 两个链接可解析；network 状态与 T3–T5 结果一致；URL、G 状态汇总和术语计数与权威文件相等；无关主题未改。
- Verification: 相对链接；URL 总数/唯一数；G 状态计数；术语表条目计数；scoped diff；`git diff --check`；strict validate。
- Stop when: T3–T5 未 GREEN，或正文需要术语表尚未定义的新主写法。

**Invariants**

- 官网是唯一权威入口；官方 GitHub只作交叉验证；固定 revision 第三方材料只说明对应代码或历史环境。
- descriptor 与 data buffer、CPU/device ownership、数据可见性和通知分别表达。
- IRQ 是推进信号，不证明数据可见性；`try_lock` 失败后的后续推进保持未知，直到有 worker recheck 或真板压力证据。
- G3–G7 是相关未知项的唯一权威位置；index 只提供入口和计数。
- 不覆盖工作区中 MS06、Iteration 000、`others/` 或用户其他既有修改。

**Non-goals**

- 不实现、修改或评审 GMAC/PHY/DMA/IRQ/网络栈代码。
- 不运行或规划真板、QEMU 网络和第三方仓库测试。
- 不设计异步 NIC、poll/select、waker、超时或取消 API。
- 不同步 SNAPSHOT、全局 tasks、M/D/K/R/I，不归档 change。
- 不创建 Evidence、抓取脚本、审计器或运行身份机制。

**Requirements Traceability Matrix**

| Requirement | Scenario | Design | Task | Iteration | Code Surface | Test Witness | Simplification | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| R1 | S1–S3 | D2 | T3,T4 | 001 | coverage；DMA/IRQ 正文 | 69/69 URL；正文不存在 | None | Covered |
| R2 | S6 | D1,D7 | T5 | 001 | G3–G7 | G3 partial、G7 open | None | Covered |
| R3 | S8–S9 | D4,D7 | T5 | 001 | G3、G7；两篇正文 | PHY/runtime 边界未解除 | None | Covered |
| R4 | S10–S12 | D5,D6 | T3–T5 | 001 | DMA/IRQ 正文；G4–G6 | 正文不存在；G4/G5 partial、G6 open | None | Covered |
| R5 | S13–S15 | D1,D7 | T3–T6 | 001 | 两篇正文；coverage；gaps；index | network 入口缺失、状态待聚合 | None | Covered |

**Acceptance**

1. R1/S1–S3/T3：第二篇直接官方 URL 与 coverage 一一对应；不可访问和冲突不写成事实。
2. R4/S10/T4：MAC/MTL/DMA、TX/RX descriptor 与 buffer 的 ownership、cache、doorbell、完成观察和回收形成可追踪闭环。
3. R4/S11–S12/T4：descriptor error、TBU/RBU、FBE、reset/link-down 与 `try_lock` 失败分别记录已证行为和未知恢复边界。
4. R2/R3/R4/R5/T5：G3–G7 与两篇正文一致，不重复缺口，不以第三方证据关闭官方缺口。
5. R5/S13–S15/T6：index 可达两篇正文，network 状态和实际 URL/缺口/术语计数一致，相邻主题未扩张。
6. T3–T6：正文 ≤450 行、链接有效、URL 唯一、diff 无空白错误、strict validate 通过。

**Verification**

- RED/基线：确认第二篇不存在；coverage 为 69/69；index 无 network 链接且主题状态为待聚合；G3–G5 partial、G6–G7 open。
- T3 GREEN：第二篇首行 URL 逐项在 coverage 唯一匹配；新增 URL 可打开且实际引用；总数=唯一数。
- T4 GREEN：结构化检查分层、TX/RX 状态、descriptor/buffer、cache/doorbell、IRQ/reclaim、错误/并发路径、revision、四级证据、四字段未知项、链接和行数。
- T5 GREEN：逐项检查 G3–G7 的状态、解除证据、network 链接、汇总和 coverage 对应关系。
- T6 GREEN：解析两篇链接；从权威文件重算 URL、缺口状态和术语数；确认无关主题行未改。
- 回归：审查本 Cycle 全量 diff；`git diff --check`；`openspec validate establish-k3-com260-gmac-network-baseline --strict`。
- 所有命令在 Act Response 中记录决定性输出和退出码；验证直接观察文档内容、链接、计数和校验结果，不引入身份型证据工程。

**Gate 2 Readiness**

| Dimension | Status | Evidence |
| --- | --- | --- |
| Investigation | PASS | 目标文档、上游职责、第三方 probe/ring/IRQ 控制流、G3–G7、index 与验证入口已定位 |
| Design | PASS | D1、D2、D5–D7 已闭合来源等级、ownership、IRQ 推进和缺口责任边界 |
| Iteration Plan | PASS | T3→T4→T5→T6 依赖有序，共同形成 MS07 可引用基线；拆出任一收尾任务都不能独立形成稳定技术结果 |
| Cycle Scope | PASS | initial Cycle 仅覆盖既有 Iteration 001 的 T3–T6，无新增目标或 repair item |
| Task Contracts | PASS | 四个任务均包含目标、当前/目标行为、测试见证、GREEN、验证和停止条件 |
| Traceability | PASS | R1–R5 的本 Iteration 场景均映射到 design、task、文件和测试见证，无 Missing/Simplified |
| Verification | PASS | URL、正文结构、链接、G 状态、计数、diff 和 strict validate 直接证明 Acceptance，无身份型证据工程 |
| User Plan Approval | PASS | 用户原话：“批准，可以开始实施了”（2026-09-09） |

Gate 2 全部通过；用户已批准当前计划并授权进入 Act。

**Persisted Evidence**

- Mode: none

Markdown 内容、URL、链接、状态和计数可低成本重跑；Act Response 足以保存命令、每项不超过 20 行的决定性输出、退出码与结论。

- Budget: 本 Cycle 最多 5 个文件（含 README），整个 change 最多 20 个 Evidence 文件；单个文本文件最多 500 行且不超过 256 KiB。本 Cycle 不创建 Evidence。

**Risks and Notes**

- 新官方 binding/glue URL 若仍不可访问，T3 可保持 coverage 不变；T4 只能使用既有官方观察和固定 revision 第三方行为，G6 保持 `open`。
- 第三方代码注释中的 64-byte cache line、40-bit hardware feature、U-Boot 对照和真板历史不能自动提升为已独立核对事实。
- 若 T4 需要新主写法或单篇预计超过 500 行，停止返回 Plan；不得由 Act 修改术语表或新增第三篇正文。
- 平衡审计：T3 是 T4 的来源前置；T4 形成第二个技术结果；T5/T6 依赖两篇正文并完成唯一权威状态与入口收尾。四项合并后仍共享同一网络主题验收与明确诊断边界，无需再拆 Iteration。

## Act Response

- Status: reported

**Implemented**

- T3：直接打开 SpacemiT Linux 6.18 `k3-br-v1.0.y` 的 K3 GMAC glue driver，确认 `spacemit,k3-gmac`、APMU interface/delay-line 和 stmmac platform handoff；将该实际引用来源登记到 coverage，URL 总数由 69 增至 70，唯一数保持一致。
- T4：创建 `docs/network/k3-gmac-dma-irq.md`，按官方交叉验证与固定 revision 第三方证据分层，记录 MAC/MTL/DMA、TX/RX descriptor 与 data buffer ownership、cache/doorbell、IRQ/W1C/reclaim、错误恢复和 `try_lock` 推进风险。
- T5：更新 G3–G7 的 network 正文与新来源指针；G3/G5 保持 `partial`，G6/G7 保持 `open`。G4 正文已记录 2026-09-09 `partial`，但汇总表仍为旧值 `open`，本轮按既有官方 `k3.dtsi` 静态拓扑修正汇总为 `partial`。
- T6：把两篇 network 正文接入 index，更新 network 主题状态和来源数；缺口保持 10 项，其中 G3/G4/G5 为 `partial`；术语表保持 20 项且无需修改。
- 限定修复 / T4 (Acceptance 3 / R4 S11–S12)：根据 `others/Rt-Async-AMP/tgoskits/drivers/ax-driver/src/net/k3_gmac/core.rs:256-279` 的真实控制流，把 §8 第 152 行 DMA soft reset timeout 的"固定 revision 第三方行为"列从"core 初始化返回错误"改为"记录 warning 后继续 stop DMA / 重配 MAC/MTL/ring / 启动 DMA，最终返回 `Ok(())`"，并在"保留边界"列加入"继续执行不证明 reset、残留状态或最终数据面可用性恢复"；同步 U4 当前证据与禁止推断：当前证据补入 `reset_dma()` 自清超时返回 `Err`、`init_hardware()` 捕获后继续完整初始化并返回 `Ok(())` 的事实链；禁止推断补入"初始化 `Ok(())`"与"记录 warning 后继续初始化不证明 reset、残留状态或最终数据面可用性恢复"。
- 限定修复 / T6 (Acceptance 5 / R5 S13–S15)：把 `docs/index.md` 维护规则中"状态/计数（当前 69 URL、10 gaps）"改为"（当前 70 URL、10 gaps）"，与摘要行和覆盖表实际值一致；全文不再出现冲突计数。

**Changed Files and Symbols**

- T3–T6 阶段：
  - `docs/reference/source-coverage.md`：新增 K3 GMAC glue driver，69/69 → 70/70。
  - `docs/network/k3-gmac-dma-irq.md`：新建 DWMAC5 数据面与 IRQ 事实包，206 行。
  - `docs/reference/known-gaps.md`：更新 G3–G7 证据、日期、状态汇总和 coverage 对应关系。
  - `docs/index.md`：增加两篇 network 正文入口并同步主题状态、URL 和缺口摘要。
  - `openspec/changes/establish-k3-com260-gmac-network-baseline/tasks.md`：T3–T6 标记完成；T1–T6 现为 6/6。
- 限定修复阶段：
  - `docs/network/k3-gmac-dma-irq.md` §8 第 152 行：表格 2/3 列改正。
  - `docs/network/k3-gmac-dma-irq.md` U4 (L189–L194)：当前证据与禁止推断两条改正。
  - `docs/index.md` 维护规则行 (L70)：URL 计数 69 → 70。
  - 本 Cycle `Act Response`：覆盖为包含 T3–T6 与限定修复的最新完整快照。
- 未触动：`others/`、MS06 收尾产物、SNAPSHOT、全局 tasks、M/D/K/R/I、术语表、coverage 行数、Gate 1 / Plan Context / Plan Review。

**Deviations from Plan**

- T5 见证发现 `known-gaps.md` 的 G4 正文已记录 2026-09-09 `open → partial`，但汇总表仍写 `open`，与 Plan Current Baseline 的 `partial` 不一致。该差异属于 T5 明确要求的状态汇总收敛；按正文已有的官方 `k3.dtsi` 静态拓扑修正为 `partial`，没有改变 requirement、design、Acceptance 或其他 G 条目。
- T3 原计划允许 coverage 无增量；实际直接打开并引用了官方 K3 GMAC glue driver，因此按契约新增 1 个 supporting-source。该来源只闭合 K3 glue/stmmac 责任边界，不关闭 G6。
- 限定修复不属于偏差：本轮按 Plan Review Follow-up Decision ("把 `Act Response` 从 `reported` 改为 `pending`，只修改 `k3-gmac-dma-irq.md` 的 reset timeout/U4 表述和 `index.md` 的旧 URL 计数；随后重跑 T4/T6 定向检查、URL/链接/计数检查、`git diff --check` 与 strict validate，覆盖 `Act Response` 为最新完整快照并恢复为 `reported`") 在当前 Cycle 限定修复，文件范围、产品文件范围、Task Contract 与 Acceptance 均未改变。

**Blocker Handoff**

None。

**Blocker Resolution**

None。

**Self-Review**

- Plan compliance: PASS
- Full diff reviewed: PASS
- Critical findings unresolved: 0
- Important findings unresolved: 0
- Minor findings unresolved: 0

- Spec compliance review: PASS。
  - T3–T6 阶段：R1、R4、R5 的 S1–S3、S10–S15 均由来源反查、TX/RX 状态链、错误/并发边界、G3–G7 和 index 覆盖；未把第三方 descriptor/IRQ 位定义、cache line、DMA 地址宽度或历史运行结果提升为 K3 官方规范。
  - 限定修复 / T4：S11–S12 的 reset / error 路径现在可按 `core.rs:256-279` 的实际控制流（`reset_dma()` → `Err` → `init_hardware()` 捕获并 `log::warn!` → 继续 stop DMA / 重配 / 启动 DMA → `Ok(())`）正确追踪；U4 把"继续初始化不证明 reset、残留状态或最终数据面可用性恢复"明确写为禁止推断。
  - 限定修复 / T6：S13–S15 要求 index 计数与权威文件实际值一致；本轮把维护规则行的 69 改为 70，入口页不再存在两个当前 URL 计数。
- Code quality review: PASS。
  - 两篇 network 文档职责分离；新正文 206 行，未复制 MS06 通用 ownership 正文；相对链接、计数和术语一致。
  - 限定修复仅触动 2 个产品文件的 3 行（`k3-gmac-dma-irq.md` 表行 1 行 + U4 段 2 行；`index.md` 维护规则 1 行），无计划外修改。
  - 未修改 `others/`、terminology、SNAPSHOT、全局 tasks、M/D/K/R/I、coverage 行数或 change 内 tasks.md。
- 已修复 finding: 1 Important (T3–T6 阶段)——G4 正文与汇总状态矛盾；汇总已按正文的官方静态拓扑依据修正为 `partial`。
- 已修复 finding: 2 Important (限定修复阶段)——(a) `k3-gmac-dma-irq.md` §8 DMA soft reset timeout 行与 `core.rs:256-279` 实际控制流相反；U4 同步补齐 warning-后-继续语义。(b) `index.md` 维护规则中"当前 69 URL"与覆盖表实际 70 URL 冲突。两者均按 Plan Review Follow-up Decision 在当前 Cycle 内修复。
- 已修复 finding: 5 Minor (T3–T6 阶段)——中英文术语黏连，不影响事实或证据等级；限定修复未引入新 Minor。

**Verification Evidence**

| 验证项 | 命令或操作 | 决定性输出 | 退出码 / 结论 |
| --- | --- | --- | --- |
| Gate 3 / T3–T4 (RED) | 目标文件不存在、coverage 总数/唯一数 | `T4_RED_absent=0`；`69` / `69` | 0 / RED 基线符合计划 |
| T3 GREEN | coverage URL 计数与第二篇首行 4 个 URL 逐项反查 | `70` / `70`；4 项均 `source_match=1` | 0 / PASS |
| T4 GREEN | 行数、分层/状态机/错误/并发关键字、四字段未知项与链接 | `206` 行；U1–U4 的四字段均 4/4；`relative_links=PASS` | 0 / PASS |
| T5 GREEN | G3–G7 汇总、四字段与正文链接检查 | `G3/G4/G5=partial`；`G6/G7=open`；各 G 保留四字段 | 0 / PASS |
| T6 GREEN (首次) | index 链接及 URL/G/术语重算 | 两篇链接存在；`70 URL`；`10 gaps`；`20 terms`；维护规则行写 69 | 0 / PASS（计数行被 Plan Review 标记为后续修复项） |
| Gate 3 / 限定修复 A (RED) | `rg -n 'core 初始化返回错误' docs/network/k3-gmac-dma-irq.md` | L152 命中 1 处 | 0 / RED 符合 Plan Review Evidence |
| Gate 3 / 限定修复 B (RED) | `grep -nE '69 URL|70 URL' docs/index.md` | L18=70；L70=69 | 0 / RED 符合 Plan Review Evidence |
| 限定修复 A GREEN | §8 L152 与 U4 当前证据/禁止推断；`rg` 复查 | 错误断言 0 命中；`记录 warning 后继续 stop DMA / 重配 MAC/MTL/ring / 启动 DMA` 在 L152；`记录 warning 后继续初始化` 在 L192；行数 206 不变 | 0 / PASS |
| 限定修复 B GREEN | `grep -nE '69 URL|70 URL' docs/index.md` + 相对链接解析 | 仅 L18/L70 各 1 行 `70 URL`；相对链接全部解析 | 0 / PASS |
| Coverage 不变性 | `awk` 提取 URL 后总数与 `sort -u` 比较 | `url_total=70 url_unique=70` | 0 / PASS（限定修复不触动 coverage） |
| Diff gate | `git diff --check` | 无输出 | 0 / PASS |
| OpenSpec gate | `openspec validate establish-k3-com260-gmac-network-baseline --strict` | `Change 'establish-k3-com260-gmac-network-baseline' is valid` | 0 / PASS |
| Task gate | `openspec list` | `establish-k3-com260-gmac-network-baseline  ✓ Complete` | 0 / PASS |

**Persisted Evidence**

None required。Plan Context 与本 Act Response 的 Verification Evidence 表已记录全部 RED/GREEN 命令、决定性输出、退出码和结论；URL、Markdown、链接、计数和 OpenSpec 校验均可低成本重跑，不满足 `required` Evidence 的必要性条件。

**Experience Candidates**

None。T3–T6 阶段是可低成本重跑的文档聚合与一致性修正；限定修复阶段是按 Plan Review 文字反馈对产品文件做两处语义修正。两者均未形成 Runbook 或 Incident 候选。

**Remaining Issues**

- 本 Cycle 没有未解决的 Acceptance gap。
- G3/G4/G5 仍为 `partial`，G6/G7 仍为 `open`；PHY 实物、运行时 IRQ delivery、coherency/IOMMU、programmer reference 和默认 DTS 映射按计划保持未知。
- 工作区中 MS06 收尾产物、MS07 全部产物（两篇 network 正文、coverage 70/70、`known-gaps.md` G3–G7 修正、`index.md` 接入）以及 `others/` 仍未提交；本 Cycle 未触动 `others/`、SNAPSHOT、全局 tasks 或 M/D/K/R/I。
- change 仍待 `openspec-plan` 对本轮限定修复做新一轮独立 Review；本次未执行全局状态同步、归档、commit 或分支清理。

**Commit or Diff Reference**

Uncommitted worktree；限定修复阶段未触动 `others/` 或既有 staged 内容。

## Plan Review

- Review Result: accepted

**Findings**

- Blocking findings: None。
- Minor：Act Response 的历史记录有两处计数/结论精度问题：首次 T6 验证行在已注明维护规则仍写 69 时仍标为 `PASS`；Code quality review 把表格 1 行、U4 2 行和 index 1 行合计写成“3 行”。两处均不改变当前产品文档、验证输出或 Acceptance 结论，不要求继续返工。

**Deviation Classification**

- 上轮 `PLAN-INVALID` 已解决：产品正文现按固定 revision 控制流记录 `reset_dma()` 返回 `Err` 后由 `init_hardware()` 捕获、warning 后继续初始化并返回 `Ok(())`；Plan Context 的历史错误保持冻结，由本 Review 的证据链取代其结论。
- 上轮 `ACT-DEVIATION` 已解决：index 摘要和维护规则均为 70 URL。
- 本轮 Minor 属 `ACT-DEVIATION`，仅影响 Act Response 的历史表述精度，不阻塞 Acceptance。

**Acceptance Gaps**

None。Acceptance 1–6 均满足。

**Convergence**

`reduced`：上轮两个 Important gap 均已修复，剩余阻塞 gap 从 2 降为 0；未出现新 Acceptance gap。

**Evidence**

- `core.rs:256-279,353-368` 与 `docs/network/k3-gmac-dma-irq.md:152,189-194` 一致：reset timeout 的局部 `Err` 被捕获，初始化继续并返回 `Ok(())`；正文明确保留 reset、残留状态和数据面可用性未证边界。产品文档中“core 初始化返回错误”命中数为 0。
- `docs/reference/source-coverage.md` 重算为 70/70；第二篇正文首行 4 个直接官方 URL 在 coverage 中各匹配 1 行；`docs/index.md:18,70` 均写 70，当前 69 URL 的旧计数命中数为 0。
- 第二篇正文 206 行；检查 `docs/network/k3-gmac-dma-irq.md` 与 `docs/index.md` 共 29 个相对链接，缺失数为 0。
- known gaps 共 10 项，术语表共 20 个数据项；G3/G4/G5=`partial`、G6/G7=`open`，G3–G7 各保留四字段。
- `git diff --check` 与 `git diff --cached --check` 均退出 0；strict validate 退出 0并报告 change valid；`openspec list` 显示 change Complete。
- Persisted Evidence 为 `none`，Evidence 目录不存在符合计划，不构成缺口。

**Follow-up Decision**

无需当前 Cycle 修复。当前 Iteration 和 change 已达到既有 Acceptance；没有剩余 Iteration，可交由 `openspec-docs-maintainer` 收尾。

**Iteration Plan Update**

None。

**Next Cycle**

None。

**Next Iteration**

None。
