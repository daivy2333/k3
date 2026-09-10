# Iteration 000 / Cycle 000: GMAC、MDIO、PHY 与 RGMII 板级基线

## Plan Context

- Status: ready
- Iteration: 000-gmac-phy-baseline
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: None

**Iteration Scope**

- Change tasks: T1、T2
- Depends on: MS04、MS05、MS06 已完成
- Stable baseline: K3/CoM260 GMAC 实例、平台资源和 PHY/RGMII 静态配置链可独立引用，后续数据面文档无需重新选择目标 DTS 或证据等级
- Verification boundary: 正文直接来源唯一登记；SoC/模组/Kit/DTS 变体分层；`eth1` 到 PHY 的静态链、冲突和未知项完整
- Diagnostic boundary: 来源身份、DTS 变体、GMAC 实例、platform resources、MDIO/PHY/RGMII、reset/delay-line
- Deferred tasks: T3-T5

**Cycle Scope**

- Trigger: initial
- Acceptance gaps: None
- Repair items: None
- Inherited scope: proposal R1-R3、R5 中属于板级链路的 S1-S9、S13-S15；design D1-D4、D7；M01-M04、D02、D04-D07
- Excluded scope: T3-T5、MAC/MTL/DMA 数据面、IRQ handler、缺口状态收尾、index 更新、驱动实现、真板、EtherCAT/TSN/网络栈

**Objective**

完成精确来源登记和 `docs/network/com260-gmac-phy.md`，使 K3 SoC、CoM260 模组、CoM260 DTS 变体、`eth1` 平台资源、MDIO、PHY 与 RGMII 的静态关系按证据等级可独立检索，同时保留默认 Kit DTS、PHY 实物型号和运行时 link 的未知边界。

**Background**

MS04 已整理 pinctrl/clock/reset/APMU provider，MS05 已整理 APLIC→IMSIC 静态拓扑，MS06 已整理设备内建 DMA 与 ownership。现有 GMAC 事实散落于 platform、boot、DMA、interrupts 和 reference 文档；`docs/network/` 尚不存在。Gate 1 已由用户以“批准”批准 R1-R5、S1-S15、默认假设和 Non-goals。

**Current Baseline**

- 主仓 revision: `b48482707990d10ca1ab7dc1fd36035e90f89e84`，branch `main`。
- 工作区带有 MS06 收尾的既有未提交修改；本 Cycle 不覆盖这些修改。
- `docs/index.md` 与 `source-coverage.md` 当前为 69 个 URL、69 个唯一值；SNAPSHOT 仍描述 68 个，本 Cycle 不刷新 SNAPSHOT。
- `docs/network/com260-gmac-phy.md` 与 `docs/network/k3-gmac-dma-irq.md` 均不存在。
- `openspec list` 只有当前 change 活跃；本 change 在 Cycle 创建前通过非 strict validate。
- 固定 revision 第三方输入：Rt-Async-AMP `ccb1ff0` / `master`，tgoskits 与其 StarryOS `19219411d` / `feat/rt-async-amp`。

**Current-State Evidence**

- `docs/platform/k3-soc-overview.md` 记录 K3 SoC 有 4 路 GMAC、支持 RGMII/RMII/MII 和 TSN；该层不证明 CoM260 实际启用实例。
- `docs/platform/com260-board-resources.md` §2.7 记录 CoM260 模组引出 1 路 GPHY/GMAC 通道 PHY1 及 GMAC1/MDC/MDIO/INT_N 信号；PHY 型号、板上连接器和完整电气实现仍未知。
- `docs/boot/com260-image-and-dts.md` §4.1/4.2/4.3、§6.3 记录 `k3_com260.dts`、共享 `k3_com260.dtsi` 与 `k3_com260_kit_v02.dts` 的 `&eth1`、`phy-handle`、MDIO/PHY、RGMII、reset 和 phase 字段，并明确 G7 下默认 Kit DTS未唯一映射。
- `docs/reference/source-coverage.md` 已登记 09-GMAC raw、`k3.dtsi`、`k3_com260.dtsi`、`k3_com260.dts` 和 `k3_com260_kit_v02.dts`；新增官方 binding/glue driver 候选尚未登记。
- `docs/reference/known-gaps.md`：G3=`partial`，保存 CoM260 GMAC/PHY 详情；G4=`open`，保存最终 IRQ delivery；G5=`partial`，保存 coherency/IOMMU；G6=`open`，保存 GMAC programmer reference；G7=`open`，保存默认 DTS 映射。
- `docs/reference/terminology.md` 已定义 GMAC、PHY、MDIO、RGMII、DMA 和 IRQ；本 Cycle 不需新增主写法。
- 固定 revision 第三方 DTS `others/Rt-Async-AMP/tgoskits/os/StarryOS/configs/board/spacemit-k3-com260-ifx.dts:6274` 提供 `eth1@0xcac82000`、DWMAC 5.10a、RGMII、FIFO 8192、speed 1000、phase 47/53、reset GPIO/delay、MDIO 和 PHY address 1。它不是默认 Kit DTS 或官方硬件规范。
- 固定 revision tgoskits `k3_gmac::probe` 从 FDT 解析 MMIO、MAC、FIFO 和 speed，在 `GlueConfig::apply` 后初始化 core；这只用于确认字段之间的第三方消费关系，不用于关闭 G3/G6。

**Relevant Code**

- `docs/reference/source-coverage.md`: URL 唯一登记与来源元数据。
- `docs/network/com260-gmac-phy.md`: 本 Cycle 新建的板级网络主题文档。
- `docs/platform/k3-soc-overview.md`: SoC 能力上游。
- `docs/platform/com260-board-resources.md`: 模组/Kit 板级资源上游。
- `docs/boot/com260-image-and-dts.md`: CoM260 DTS 候选与 `eth1` 字段上游。
- `docs/platform/k3-platform-control.md`: AP pinctrl/clock/reset/APMU provider 边界。
- `docs/interrupts/k3-interrupt-and-time.md`: AP APLIC→IMSIC 静态路径边界。
- `docs/reference/known-gaps.md`: G3-G7 权威未知项；本 Cycle只引用，不修改。
- `others/Rt-Async-AMP/tgoskits/drivers/ax-driver/src/net/k3_gmac/mod.rs::probe`: 固定 revision 第三方 FDT 消费入口。
- `others/Rt-Async-AMP/tgoskits/drivers/ax-driver/src/net/k3_gmac/syscon.rs::GlueConfig`: 固定 revision 第三方 RGMII/APMU/reset 字段消费关系。
- `others/Rt-Async-AMP/tgoskits/drivers/ax-driver/src/net/k3_gmac/mdio.rs::Mdio`: 固定 revision Clause 22 扫描和自协商行为；运行时结果不属于本 Cycle 验证。

**Critical Path**

```text
R01 / 09-GMAC / official GitHub sources
  → source-coverage 唯一登记
  → K3 SoC 4×GMAC 能力
  → CoM260 模组 PHY1/GMAC1 引出
  → CoM260 DTS 变体中的 eth1
  → reg/IRQ/clock/reset/pinctrl/APMU
  → phy-mode=rgmii + delay-line
  → MDIO Clause 22 → PHY address 1
  → 静态配置与运行时未知边界
```

来源不可访问时保留访问状态，不生成事实。变体字段只属于对应 DTS；没有唯一映射时停止在 G7 边界。PHY ID 不进入商品型号、扩展寄存器或 internal-delay 推断。

**Implementation Guidance**

先直接打开并核对官方来源，再精准更新 coverage。随后按“能力层 → 板级层 → DTS 变体 → platform resource → MDIO/PHY/RGMII → 未知项”的顺序写正文。复用相邻文档事实时使用相对链接，不复制其完整表格。第三方 FDT 和驱动只放在单独的交叉验证/边界段。

**Behavioral Change**

当前读者需要跨 platform、boot 和 reference 文档拼接 GMAC/PHY 链，且 `docs/network/` 无正文。完成后，`com260-gmac-phy.md` 提供单一网络主题入口；它不会选择默认 DTS、改变缺口状态或声明运行时硬件已验证。

**Change Surface**

| Task | Requirement/Scenario | File/Symbol | Current Responsibility | Planned Change |
| --- | --- | --- | --- | --- |
| T1 | R1/S1-S3；R2/S4-S6；R5/S13-S14 | `docs/reference/source-coverage.md` | 69 个唯一 URL 与来源职责 | 核对既有 GMAC/DTS URL并按正文实际引用增量登记精确官方 binding/glue URL |
| T2 | R1/S1-S3；R2/S4-S6；R3/S7-S9；R5/S13-S15 | `docs/network/com260-gmac-phy.md` | 不存在 | 创建 GMAC/MDIO/PHY/RGMII 板级事实包 |

**Task Contracts**

### T1：精确来源登记

- Requirement/Scenario: R1/S1-S3；R2/S4-S6；R5/S13-S14。
- Depends on: None。
- Targets: `docs/reference/source-coverage.md`。
- Current behavior: 69 个 URL 与 69 个唯一值；已登记 09-GMAC raw 和四个 K3/CoM260 DTS来源，未按 MS07 核对官方 GMAC binding/glue driver 精确 URL。
- Required behavior: 直接打开正文所需来源；既有 URL 只扩充职责；新 URL 仅在可打开且被正文实际引用时登记；表内总数与唯一数保持一致。
- Required changes: 核对 R01/09-GMAC、`k3.dtsi`、`k3_com260.dtsi`、`k3_com260.dts`、`k3_com260_kit_v02.dts`；检查 linux-6.18 `k3-br-v1.0.y` 的 `Documentation/devicetree/bindings/net/snps,dwmac.yaml`、相关 stmmac platform binding 与 `drivers/net/ethernet/stmicro/stmmac/dwmac-spacemit-ethqos.c`。候选不存在或不可访问时不登记，并保持 G6 缺失边界。
- Preserve: 既有 69 行的 URL 身份、来源等级、观察日期和状态；M01、D04-D07；官网 `partially-observed`。
- Forbidden: 不登记未引用候选，不批量刷新，不把第三方材料登记为官方来源，不修改 `others/`。
- Test witness: 修改前运行 URL 行数/唯一数检查，预期 `69/69`；确认候选精确 URL 尚未登记。
- GREEN condition: 正文全部直接官方 URL 各登记一次；每行字段完整；实际总数等于唯一数。
- Verification: `awk -F'|' '/^\| https?:\/\// {gsub(/^ +| +$/, "", $2); print $2}' docs/reference/source-coverage.md` 的总数与 `sort -u` 计数一致；正文首行反向核对；`git diff --check`；strict validate。
- Stop when: 来源变化改变 design、目标 DTS 或 Acceptance；或需要 M01/R08 外的新权威来源。

### T2：GMAC、MDIO、PHY 与 RGMII 事实包

- Requirement/Scenario: R1/S1-S3；R2/S4-S6；R3/S7-S9；R5/S13-S15。
- Depends on: T1。
- Targets: `docs/network/com260-gmac-phy.md`。
- Current behavior: 文件及目录不存在；事实散落在 platform、boot 和 gaps。
- Required behavior: 提供 K3 SoC → CoM260 模组 → DTS 变体 → `eth1` → platform resources → MDIO → PHY → RGMII 的静态链；每项带适用对象和证据等级。
- Required changes: 建立能力/实例矩阵、DTS 变体表、资源表、MDIO/PHY/RGMII 链、冲突/运行时边界和四字段未知项。PHY ID、地址 1、reset GPIO/delay、phase 47/53 只按直接来源范围记录。
- Preserve: G3-G7 权威职责；目标 DTS 未唯一映射；相邻文档事实；简体中文与现有术语；≤450 行。
- Forbidden: 不指定默认 DTS，不推定 PHY 完整型号/寄存器，不把第三方运行结果写成本项目验证，不展开数据面/IRQ/EtherCAT/TSN/驱动设计。
- Test witness: `test ! -e docs/network/com260-gmac-phy.md` 返回 0；index 当前 network 为待聚合。
- GREEN condition: S1-S9、S13-S15 的板级部分均有事实、冲突或未知原因；来源已登记；链接有效；无证据等级提升。
- Verification: 检查首行格式、章节/矩阵、证据标签、G3/G7 链接、禁止断言、相对链接、`wc -l` ≤450、scoped diff、`git diff --check` 和 strict validate。
- Stop when: 实际目标板超出 CoM260 范围，或两篇职责边界/Acceptance 需要变化。

**Invariants**

- M01 官网仍是唯一权威入口；官方 GitHub只作交叉验证。
- SoC 能力、模组引出、Kit/DTS 配置和运行时行为不互相替代。
- G3-G7 是相关未知项的唯一权威位置；本 Cycle 不修改其状态。
- 不修改产品代码、`others/`、SNAPSHOT、全局 tasks 或 M/D/K/R/I。
- 不覆盖工作区中 MS06 或用户的既有修改。

**Non-goals**

- 不建立 DWMAC5 descriptor/IRQ 专题；该工作属于 T3。
- 不更新 G3-G7 或 index；该工作属于 T4-T5。
- 不获取或要求真板、原理图、PHY datasheet。
- 不创建脚本、自动抓取器、Evidence 文件或运行身份机制。

**Acceptance**

- A1 / R1/S1-S3 / D2 / T1：正文直接官方来源各在覆盖表唯一登记；访问失败和冲突不伪装为事实。
- A2 / R2/S4-S6 / D1,D3 / T2：SoC、模组、Kit/DTS 变体与 platform resources 分层完整；默认 Kit DTS保持未知。
- A3 / R3/S7-S9 / D4 / T2：MDIO/PHY/RGMII 静态链完整；PHY 型号和运行时 link 边界明确。
- A4 / R5/S13-S15 / D1,D7 / T2：正文使用既有术语、相邻职责和 G3-G7 引用，不扩展 EtherCAT/TSN/网络栈。
- A5 / T1-T2：文档 ≤450 行、相对链接有效、无重复 URL、diff 无空白错误、change strict validate 通过。

**Verification**

- RED/基线：两个 `test ! -e` 命令确认目标文档不存在；覆盖表 URL 总数/唯一数为 69/69。
- T1 GREEN：URL 总数=唯一数；正文直接 URL 全部可在覆盖表精确匹配；新增来源可直接打开且被正文引用。
- T2 GREEN：首行来源格式、四级证据、变体矩阵、静态链、未知项和相对链接检查通过，行数 ≤450。
- 回归：仅审查本 Cycle diff；`git diff --check`；`openspec validate establish-k3-com260-gmac-network-baseline --strict`。
- 任何命令输出与退出码写入 Act Response，每项保留不超过 20 行的决定性片段。

**Gate 2 Readiness**

| Dimension | Status | Evidence |
| --- | --- | --- |
| Investigation | PASS | 当前入口、上游文档、第三方消费路径、缺口和基线命令已定位 |
| Design | PASS | D1-D7 闭合文档边界、来源等级、DTS/PHY/IRQ 未知语义 |
| Iteration Plan | PASS | T1-T2 形成板级基线；T3-T5 延后且依赖 accepted Review |
| Cycle Scope | PASS | initial Cycle 只覆盖 T1-T2，无 repair item |
| Task Contracts | PASS | 两个任务均含目标、当前/目标行为、测试见证、GREEN、验证和停止条件 |
| Traceability | PASS | A1-A5 与 R/S/D/T/文件/验证一一映射 |
| Verification | PASS | 直接检查 URL、Markdown、链接、边界和 strict validate，无身份型证据工程 |
| User Plan Approval | PASS | 用户指令：“批准进行并开始实施”（2026-09-09） |

**Persisted Evidence**

- Mode: none

Act Response 足以保存命令、决定性输出、退出码和结论；URL、Markdown、链接、计数和 OpenSpec 校验均可低成本重跑，不满足 `required` Evidence 的必要性条件。

- Budget: 本 Cycle 最多 5 个文件（含 README），整个 change 最多 20 个 Evidence 文件；单个文本文件最多 500 行且不超过 256 KiB；本 Cycle 不创建 Evidence。

**Risks and Notes**

- SNAPSHOT 68 与当前覆盖表 69 的差异是既有状态；T1 只按执行时实际表值增量更新。
- 官方候选 binding/glue URL 若不可访问，不阻塞基于已登记来源交付板级正文，但不得引用其内容或关闭 G6。
- 固定 revision IFX DTS 和 tgoskits 只用于字段关系交叉验证，不替代官方 CoM260 DTS。
- Gate 2 全部通过；用户已批准当前计划并授权开始实施。

## Act Response

- Status: reported

**Implemented**

- T1：核对 `source-coverage.md` 的 69 个 URL 与唯一性；确认 09-GMAC 和四个 K3/CoM260 DTS 来源已经唯一登记。候选官方 binding/glue driver 因当前环境 DNS 不可用而未登记，G6 保持未解。
- T2：创建 `docs/network/com260-gmac-phy.md`，完成 SoC/模组/Kit/DTS 分层、`eth1` 平台资源、MDIO/PHY/RGMII 静态链、来源差异、运行时边界和五个四字段未知项。
- Gate 4 修复：首轮正文未直接列出已确认的 `eth1` MMIO 和 wired source；补入 `0xcac82000/0x2000` 与 AP APLIC source 133，并注明 clock/reset 数字 ID 未独立解码。

**Changed Files and Symbols**

- `docs/network/com260-gmac-phy.md`：新建板级网络主题正文，238 行。
- `openspec/changes/establish-k3-com260-gmac-network-baseline/tasks.md`：T1、T2 标记完成；T3-T5 保持待办。
- 本 Cycle 的 `Act Response`：记录实现、Review 与验证结果。
- `docs/reference/source-coverage.md`：本 Cycle 只读核对，无新增修改；文件在执行前已有用户/MS06 未提交修改。

**Deviations from Plan**

- T1 计划检查的官方 GitHub binding/glue driver 无法通过当前环境 DNS 访问。按 Task Contract 的既定分支，未登记不可访问候选，复用已在此前 change 直接观察并登记的 09-GMAC 与 K3/CoM260 DTS 来源；不影响 T2 板级基线，G6 保持 `open`。
- T1 没有产生产品文件 diff；任务结果是核对既有 69/69 URL 并确认无需登记未引用来源。

**Blocker Handoff**

None

**Blocker Resolution**

None

**Self-Review**

- Plan compliance: PASS
- Full diff reviewed: PASS
- Critical findings unresolved: 0
- Important findings unresolved: 0
- Minor findings unresolved: 0

- Spec compliance review: PASS。R1-R3 与 R5 的本 Iteration 场景均由来源、分层矩阵、静态链和未知边界覆盖；T3-T5 未提前实施。
- Code quality review: PASS。正文使用既有术语和相对链接，未复制相邻主题的完整事实包，未修改 `others/`、SNAPSHOT、全局 tasks 或 M/D/K/R/I。
- 已修复 finding: 1 Important——补全 `eth1` MMIO 窗口与 AP APLIC source，使资源矩阵可直接执行检索。

**Verification Evidence**

| 验证项 | 命令或操作 | 决定性输出 | 退出码 / 结论 |
| --- | --- | --- | --- |
| Gate 3 / T1 | `awk` 提取覆盖表 URL并比较总数与唯一数 | `url_total=69`；`url_unique=69` | 0 / RED 基线符合计划 |
| Gate 3 / T2 | `test ! -e docs/network/com260-gmac-phy.md` | `phy_doc_absent=0` | 0 / RED：目标文件原本不存在 |
| 来源访问 | `curl -L --fail --max-time 20 .../09-GMAC.md` | `curl: (6) Could not resolve host: raw.githubusercontent.com` | 6 / 候选不登记，按契约保留 G6 |
| T1 GREEN | URL 唯一计数、既有来源精确匹配、`git diff --check`、strict validate | `url_total=69 url_unique=69`；`Change ... is valid` | 0 / PASS；coverage 无需修改 |
| T2 GREEN | 行数、未知项/字段、链接、来源、MMIO/IRQ 断言 | `lines=238 unknowns=5/5 unknown_fields=20/20 urls=69/69 links=PASS resource_values=PASS` | 0 / PASS |
| 全量 Gate | tasks 状态、目标文件边界、`git diff --check`、strict validate | T1-T2=2 complete；T3-T5=3 pending；`Change ... is valid` | 0 / PASS |

**Persisted Evidence**

None required

**Experience Candidates**

None

**Remaining Issues**

- 当前 Iteration 没有未解决的 Critical、Important 或 Minor finding。
- G3-G7 中的 PHY 型号、默认 DTS、programmer reference、coherency/IOMMU 和最终 IRQ delivery 按计划保持未知；其状态更新属于后续 T4。
- T3-T5 仍待 Iteration 000 的独立 Plan Review 接受后执行。

**Commit or Diff Reference**

Uncommitted worktree diff；未创建 commit。

## Plan Review

- Review Result: replan-required

**Findings**

- Important / PLAN-INVALID：T1 的全局 GREEN 要求“两篇计划正文”的全部直接官方 URL 已登记，但本 Cycle 只创建第一篇，第二篇直到下一 Iteration 才产生；原 T3 又不以 `source-coverage.md` 为目标。因此本 Cycle 不可能证明 T1 的完整 Acceptance，下一 Iteration 也缺少合法的单文件任务来补齐第二篇来源。
- Important / ACT-DEVIATION：`com260-gmac-phy.md` §4.1 把 `0xcac82000/0x2000` 与 AP APLIC source 133 标为“交叉验证”，但当前可审计材料只明确把这些精确值归于固定 revision 第三方 IFX DTS；§4.2 也将同一组值正确标为“固定 revision 第三方”。在无法直接重开官方 `k3.dtsi` 的本轮证据条件下，§4.1 构成证据等级提升和文内矛盾。

**Deviation Classification**

- PLAN-INVALID：来源登记任务跨越尚未创建的第二篇正文，Cycle/Iteration 分配与 GREEN 契约不可同时满足。
- ACT-DEVIATION：资源矩阵对精确 MMIO/IRQ 的证据等级高于当前可追溯输入。

**Acceptance Gaps**

- T1 缺少可在当前 Cycle 完成的单篇正文来源边界，原验证证据只能证明第一篇正文所用 URL 已覆盖。
- T2 的“无跨证据等级提升”未满足：MMIO 与 IRQ 两项需要降为“固定 revision 第三方”，或由可直接打开的官方 SoC DTS 精确字段重新支撑。

**Convergence**

- 首次独立 Review；两项 Important finding 均有有限修复面。若后继 Cycle 按修订契约拆分来源任务并修正两项证据标签，预计可收敛。

**Evidence**

- `tasks.md` 原 T1 与 GREEN 同时引用“两篇正文”，而 Iteration 000 只包含 T1-T2；原 T3 仅创建第二篇正文且不允许修改 coverage。
- `000-initial.md` Act Response 只证明 `com260-gmac-phy.md` 的来源反向核对和 69/69 唯一性，没有也不可能提供尚不存在的第二篇首行来源。
- `com260-gmac-phy.md` §4.1 与 §4.2 对同一 MMIO/IRQ 值给出不同证据等级；Plan Context 的 Current-State Evidence 明确把这些值归于固定 revision 第三方 DTS。
- 审计重跑 `git diff --check` 与 `openspec validate establish-k3-com260-gmac-network-baseline --strict` 均通过；结构验证不能消除上述语义缺口。

**Follow-up Decision**

- 已修订全局任务：T1 只验收第一篇正文来源；新增 T3 负责第二篇正文来源，原 T3-T5 顺延为 T4-T6。
- 后继 Cycle：`001-replan.md`。需用户重新批准其 Gate 2 后，Act 才可修正产品文档并重跑 T1-T2 验收。

**Iteration Plan Update**

None

**Next Cycle**

None

**Next Iteration**

None
