# Iteration 000 / Cycle 000: 来源与 QSPI/SPI/SDHCI 基线

## Plan Context

- Status: ready
- Iteration: 000-storage-sources-and-sdhci
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: None

**Iteration Scope**

- Change tasks: T1, T2
- Depends on: None
- Stable baseline: 存储来源具有当前职责，QSPI、普通 SPI、SD/eMMC/SDHCI 的资源、启动和数据路径有独立正文及未知项。
- Verification boundary: 四个官网入口不再 deferred 且准确记录可达状态；新增 supporting rows（若有）均已直接读取且 URL 唯一；`k3-qspi-spi-sdhci.md` 覆盖 R1-R3、R5-R6 的相关场景，首行来源和相对链接有效。
- Diagnostic boundary: 来源访问/登记、QSPI 与普通 SPI 区分、SD/eMMC 板级映射、SDHCI 静态资源、第三方 core 证据等级。
- Deferred tasks: T3-T6

**Cycle Scope**

- Trigger: initial
- Acceptance gaps: None
- Repair items: None
- Inherited scope: proposal 中已批准的 MS09 范围、R1-R7、M01-M04、D02 主题目录选择和公共证据规则。
- Excluded scope: UFS 协议/恢复正文、最终缺口与术语收敛、总索引接入、驱动实现、真板或介质写操作、无关 OpenSpec spec 修复。

**Objective**

建立可追溯的存储来源集合，并交付一篇区分 QSPI、普通 SPI、SD 与 eMMC/SDHCI 的正文，使其资源、启动用途、数据路径、错误边界和未知项可以独立验证。

**Background**

MS09 是 roadmap 中 MS08 后的首个 planned milestone。当前 `docs/storage/` 不存在，四个官网入口仍标记为 future/deferred。MS03 和 MS06 已提供启动与通用 DMA 基线，本 Cycle 只补存储控制器专属事实和边界。

**Current Baseline**

- Repository revision: `820535c0bab7b2e58df3c1c01bc6a9e2689ba4a9`；`main`。
- 工作区已有本 change 的 proposal/spec/design/tasks/Cycle；用户原有 `others/` 为未跟踪目录，不纳入产品修改。
- `docs/index.md:64` 把 `docs/storage/` 标为待聚合；目录不存在。
- `source-coverage.md:49-50,58-59` 的 QSPI、SDHC、SPI、UFS 官网入口均为 `future / deferred / partially-observed`。
- 覆盖表当前 70 个 URL；UFS raw supporting source 已在第 81 行，其他三个 docs-buildroot raw 对应页尚未登记。
- 2026-09-10 通过 web 打开四个 raw/blob URL 均返回 cache miss；随后 `curl --fail --max-time 20` 均因 `Could not resolve host: raw.githubusercontent.com` 退出 6。该结果确定本 Cycle 的不可达处理，不要求 Act 重试网络。
- 全量 OpenSpec 基线有两个既有无关 spec 失败；本 change 自身严格校验通过。

**Current-State Evidence**

- 官方 K3 datasheet 已确认：Quad-SPI 支持 XIP/Page、1/2/4 线、13.25–102 MHz、NOR/NAND；eMMC 5.1 和 SD 3.0 控制器兼容 SDHCI 并支持 PIO/SDMA/ADMA/ADMA2；普通 SPI 总量为 6 路，但概述不提供每路地址和运行路径。
- `docs/platform/com260-board-resources.md` 已确认 CoM260 具有板载 SPI Flash、TF Card 和 UFS；容量、器件型号、速度/热插拔仍有未知项。
- `docs/boot/com260-boot-chain.md` 已记录 SD 优先以及 eMMC/SPI NOR/SPI NAND/UFS 候选和固件布局；本 Cycle 只从控制器角度交叉引用。
- 固定第三方 IFX DTS 包含 `spacemit,k3-qspi`、普通 SPI 和三个 `spacemit,k3-sdhci` 节点，但受 G7 约束，不能当作默认 Kit。
- 固定第三方 `k3-sdhci` 是 portable driver core：持有 MMIO 和 PHY/tuning 状态，支持 SD/eMMC mode、HS200/HS400、DLL lock 与 256 delay-code tuning；FDT probe、IRQ 和 block registration 明确留给 consuming layer。该事实不能证明当前 board profile 已集成。
- 没有发现同等级的 K3 普通 SPI 或 QSPI 第三方数据面实现；Act 不得从接口占位或通用 Linux driver 推导 K3 运行行为。

**Relevant Code**

- `docs/reference/source-coverage.md`：URL 唯一覆盖和来源职责。
- `docs/platform/k3-soc-overview.md`：SoC 存储能力入口。
- `docs/platform/com260-board-resources.md`：CoM260 存储介质与冲突。
- `docs/boot/com260-boot-chain.md`、`com260-image-and-dts.md`：启动与镜像责任。
- `docs/dma/k3-dma-and-memory-ownership.md`、`k3-cache-pma-address-translation.md`：DMA/cache/ownership 责任。
- `others/Rt-Async-AMP/tgoskits/os/StarryOS/configs/board/spacemit-k3-com260-ifx.dts`：固定第三方变体静态资源。
- `others/Rt-Async-AMP/tgoskits/drivers/blk/k3-sdhci/src/lib.rs`、`vendor_ext.rs`：固定第三方 portable SDHCI core。

**Critical Path**

官方/官方 supporting source → `source-coverage.md` 分配存储职责 → `k3-qspi-spi-sdhci.md` 按设备解释静态资源、板级和软件行为 → 相对链接返回 boot/platform/DMA 权威主题。来源不可达、冲突或不足时转为未知项，不产生运行状态变化。

**Implementation Guidance**

先完成 T1 并确认正文所需 URL 均有唯一覆盖记录，再创建 T2。正文建议按范围与证据、QSPI、普通 SPI、SD/eMMC/SDHCI、启动关系、数据路径、错误边界、未知项、交叉引用组织。章节可按材料自然调整，但必须保持设备边界和证据等级。

**Behavioral Change**

当前读者只能从总览和 deferred 来源找到零散存储事实。完成后，读者能从独立正文查询 QSPI/SPI/SDHCI，并判断事实属于 SoC 能力、CoM260 板级、固定 DTS、第三方 core 或未知项。无运行时接口、状态或错误语义发生变化。

**Change Surface**

| Task | Requirement/Scenario | File/Symbol | Current Responsibility | Planned Change |
|---|---|---|---|---|
| T1 | R1/S1-S2, R6/S1-S2 | `docs/reference/source-coverage.md` | 70 URL 覆盖；四个官网入口 deferred | 激活四个入口并登记实际读取的 supporting source |
| T2 | R1-R3, R5-R6 | `docs/storage/k3-qspi-spi-sdhci.md` | 文件不存在 | 新建非 UFS 存储控制器知识正文 |

**Task Contracts**

### T1: 存储来源具有当前且唯一的责任记录

- Requirement/Scenario: R1/S1-S2, R6/S1-S2
- Depends on: None
- Targets: `docs/reference/source-coverage.md`
- Current behavior: QSPI、SDHC、SPI、UFS 官网入口为 `future / deferred / partially-observed`；仅 UFS 有 raw GitHub supporting row。
- Required behavior: 四个官网入口标记为当前 storage 聚合职责并准确保留 `partially-observed`/不可达边界；复用已观察的 datasheet、UFS raw 和官方源码行，只有本 Cycle 已实际打开的其他 URL 才新增 supporting row；每个 URL 仅出现一次。
- Required changes: 更新 priority、aggregation status 和备注；在备注记录 2026-09-10 的 cache miss 与 DNS 失败，不把失败日期写成源端观察成功日期；官网 SPA 与 GitHub supporting source 保持不同证据等级。
- Preserve: 既有 70 行的事实、其他主题职责和历史观察日期；M01 单一权威边界。
- Forbidden: 不把 GitHub supporting row 提升为官网权威；不登记未直接读取的 URL；不批量刷新无关来源。
- Test witness: 修改前 `rg` 显示四个官网入口含 `future | deferred`；目标 supporting URLs 缺失或只含 UFS。该结果是文档功能 RED。
- GREEN condition: 四个入口均无 `future/deferred`，访问状态不被虚假提升；已有/新增 supporting URL 有正确 storage 职责；URL 字段无重复。
- Verification: `rg` 检查四行职责、状态和不可达备注；解析 Markdown 表第一列并报告重复数为 0；人工核对任何新增行确已读取。
- Stop when: 现有可读资料不足以支撑正文的最低契约、来源身份不明、源端变化要求全局 refresh，或必须改写 M01 才能继续；不因已确认的网络不可达重复重试。

### T2: QSPI、普通 SPI 和 SDHCI 知识可按设备查询

- Requirement/Scenario: R1/S1-S2, R2/S1-S2, R3/S1-S2, R5/S1-S2, R6/S1-S2
- Depends on: T1
- Targets: `docs/storage/k3-qspi-spi-sdhci.md`
- Current behavior: `docs/storage/` 和目标文件不存在；相关事实散落在 platform、boot、DMA 与第三方代码中。
- Required behavior: 正文分别描述 QSPI、普通 SPI、SD 与 eMMC/SDHCI 的 SoC 能力、板级可达性、静态资源、启动关系、数据/完成路径、DMA/IRQ/tuning 和错误边界；每个不确定结论提供当前证据、禁止推断、解除条件和影响。
- Required changes: 创建目录和文件；首行列出实际使用的官网 URL、observed date 及 supporting URLs；加入 boot/platform/DMA/gaps 的相对链接；明确 QSPI 不是任意 SPI message controller，SD 与 eMMC 不因共用 SDHCI 而等同，静态 DTS 与 portable core 不证明运行时启用。
- Preserve: MS03 启动链、MS06 ownership、G5/G7 和 CoM260 UFS 容量冲突的权威职责；简体中文和既有证据等级词汇。
- Forbidden: 不写 UFS 协议正文；不裁决默认 DTS；不声明真板测试、运行性能或第三方 core 已接入；不新增驱动代码。
- Test witness: `test ! -e docs/storage/k3-qspi-spi-sdhci.md` 当前退出 0，证明目标正文缺失；修改前以此 RED 建立见证。
- GREEN condition: 文件存在，首行为 `> 来源:`；设备分层、启动、ownership、错误和未知项均可检索；全部相对链接有效；建议不超过 500 行。
- Verification: `rg` 检查 QSPI/SPI/SDHCI/eMMC、XIP、DMA/IRQ、tuning、错误与未知项；相对链接检查；`wc -l`；`git diff --check`。
- Stop when: 新证据与 MS03/MS06 发生实质冲突、普通 SPI/QSPI 资料不足以支撑批准范围、或需要选择默认 Kit/DTS。

**Invariants**

- M01-M04 保持不变；产品产出只写入 `docs/`，无脚本或可执行代码。
- 官网入口是权威来源，GitHub 和第三方源码只作 supporting/third-party evidence。
- 每条事实绑定适用设备和证据等级；能力、静态节点、软件路径与真板结果不混写。
- 不执行或规划介质写入、刷写、格式化和破坏性操作。

**Non-goals**

- 不完成 T3-T6。
- 不实现存储驱动或验证真板 I/O。
- 不修复两个既有 OpenSpec spec 校验失败。

**Acceptance**

- A1 / R1,R6 / T1：四个官网入口承担当前 storage 职责并准确记录不可达边界，已读取的 supporting source 唯一登记，证据等级正确。
- A2 / R1-R3 / T2：正文按 QSPI、普通 SPI、SD、eMMC/SDHCI 分层，包含资源、板级、启动与数据路径。
- A3 / R3,R5 / T2：DMA/IRQ/tuning、错误/超时/恢复只按已有证据表述，静态能力和实际运行明确分开。
- A4 / R6 / T2：来源冲突和未知项具有当前证据、禁止推断、解除条件与影响，且不裁决 G7。
- A5 / R7 / T2：首行来源和相对链接有效，既有 boot/platform/DMA 权威内容不被重复定义或改写。

**Verification**

- RED：四个 coverage rows 仍含 future/deferred；目标正文不存在。
- GREEN：针对 A1-A5 的 `rg` 内容断言、URL 唯一检查、相对链接检查、`wc -l`、`git diff --check`。
- Change gate：`openspec validate establish-k3-storage-controller-baseline --strict` 退出 0。
- Regression note：`openspec validate --all --strict` 当前基线因 `k3-com260-platform-uart-baseline` 和 `source-refresh` 两个既有 spec 失败；不得把无关失败归因于本 Cycle，也不得顺手修改。

**Gate 2 Readiness**

| Dimension | Status | Evidence |
|---|---|---|
| Investigation | PASS | 已定位 coverage、既有正文、固定 DTS、SDHCI core、验证入口、来源不可达结果与无关基线失败；不可达时的契约已闭合 |
| Design | PASS | D1-D6 闭合文档拆分、证据等级、启动职责、静态/运行时和错误恢复边界 |
| Iteration Plan | PASS | T1-T6 唯一分配到 000-002；每轮有稳定、验证和诊断边界；平衡审计通过 |
| Cycle Scope | PASS | 当前只执行 T1-T2，UFS 与导航收敛明确 deferred |
| Task Contracts | PASS | T1-T2 均含目标、当前/目标行为、保持/禁止、见证、GREEN、验证和停止条件 |
| Traceability | PASS | RTM 中 R1-R7 无 Missing 或 Simplified；当前 Cycle 的 A1-A5 映射到任务和表面 |
| Verification | PASS | 直接检查目标文档行为、链接、唯一性和严格校验；无身份型证据工程 |
| User Plan Approval | PASS | 用户于 2026-09-10 回复 `批准`，接受当前计划、Iteration Map、Task Contracts、Acceptance、验证边界和 Persisted Evidence `none` |

**Persisted Evidence**

- Mode: none

Act Response 足以保存命令、决定性输出、退出码、修改文件和验收映射；所有检查可低成本重跑，不创建 Evidence 目录。

- Budget: 本 Cycle 最多 5 个文件（含 README），整个 change 最多 20 个 Evidence 文件；单个文本文件最多 500 行且不超过 256 KiB；本计划不使用该预算。

**Risks and Notes**

- 官网四页本次通过 web 访问 raw/blob URL 均出现 cache miss，`curl` 又因 DNS 失败退出 6。Act 不重试网络；T1 记录该边界，且不得把未读 supporting row 标记 observed。现有可读官方 datasheet、已观察 UFS supporting row和既有文档足以支持当前 Cycle，其余 driver 细节进入未知项。
- 官方 docs-chip K3 datasheet 可读，但若其当前内容改变既有 2026-09-08 基线，属于 refresh 判断，不在本 Cycle 静默同步。
- `k3-sdhci` 虽存在于固定第三方仓库，但当前 K3 board profile 未显示其 FDT/IRQ/block glue；这是非实质未知项，因为正文目标正是记录该边界，而非证明集成。

## Act Response

- Status: reported

**Implemented**

- T1：将 QSPI、SDHC、SPI、UFS 四个官网入口从 `future/deferred` 调整为当前 `storage` 的 `current/active` 职责；保留 `partially-observed`、原观察日期与源端修订字段，并记录 2026-09-10 cache miss / DNS 失败不是观察成功。复用既有官方入口与 UFS supporting row，本 Cycle 不新增未直接读取的 URL。
- T2：创建非 UFS 存储正文，按 QSPI、普通 SPI、SD/eMMC/SDHCI 分层记录 SoC 能力、CoM260 板级边界、固定 IFX DTS 静态资源、启动关系、数据/完成路径、DMA/IRQ/tuning、错误恢复边界及五个四字段未知项。
- 根据 Plan Review 修复 T2：删除 QSPI 节点具有 DMA 属性的错误外推，明确节点没有 `dmas`/`dma-names` 且不能据此裁决 PIO/DMA；首行补充已直接观察的 docs-chip datasheet URL 与 tgoskits supporting URL、revision 和适用文件。
- 已完成 T1、T2；T3-T6 按 Iteration Map 保持 deferred。

**Changed Files and Symbols**

- `docs/reference/source-coverage.md`：覆盖表总数说明及四个存储官网入口。
- `docs/storage/k3-qspi-spi-sdhci.md`：新增完整正文，共 151 行。
- `openspec/changes/establish-k3-storage-controller-baseline/tasks.md`：T1、T2 标记完成。
- 当前文件 `Act Response`：记录实施、Review 与验证反馈。

**Deviations from Plan**

None.

**Blocker Handoff**

None.

**Blocker Resolution**

None.

**Self-Review**

- Plan compliance: PASS
- Full diff reviewed: PASS
- Critical findings unresolved: 0
- Important findings unresolved: 0
- Minor findings unresolved: 0

- T1 Spec Review：PASS。四个目标入口均承担当前职责，访问状态和观察日期未虚假提升，URL 唯一；没有登记未直接读取的 supporting URL。
- T1 Quality Review：PASS。修改仅触及目标行和覆盖表总数说明，没有改动其他来源职责或历史事实。
- T2 Spec Review：PASS。A2-A5 要求的设备分层、启动关系、DMA/IRQ/tuning、错误边界、证据等级、未知项四字段和主题链接均已覆盖；没有裁决 G7 或把第三方 core 表述为运行时集成。
- T2 Quality Review：PASS。正文按设备和故障域组织，未复制既有启动/DMA 权威正文，无空链接、重复实现或无依据复杂度；修复后 QSPI 静态属性与 DTS 一致，来源元数据可直接核对。
- 已修复 finding：ACT-DEVIATION / A3-A4，QSPI DTS 的 DMA 属性错误外推。
- 已修复 finding：ACT-DEVIATION / A5，首行缺少 docs-chip 与 tgoskits supporting URL。
- 全量 diff Review：PASS。当前 Cycle 只修改计划列出的产品表面、任务状态和 Act Response；未发现跨任务矛盾、计划外产品修改或 Acceptance 遗漏。

**Verification Evidence**

| 验证项 | 命令或操作 | 输出摘录 | 覆盖范围 | 结论 |
| --- | --- | --- | --- | --- |
| Gate 3 / T1 RED | `rg -n 'QSPI\|SDHC\|SPI控制器\|UFS\|future\|deferred' docs/reference/source-coverage.md` | 四个官网入口均为 `future / deferred`；退出 0 | T1 修改前职责缺口 | PASS |
| Gate 3 / T2 RED | `test ! -e docs/storage/k3-qspi-spi-sdhci.md` | 退出 0 | T2 修改前正文缺失 | PASS |
| T1 GREEN | 目标行 `rg` + Markdown 表 URL 唯一性检查 | 四行均为 `current / active`；`UNIQUE_URLS=70`，`DUPLICATES=0`；退出 0 | A1 | PASS |
| T2 内容与结构 | 首行、关键词和未知项字段断言 | `FIRST_LINE_SOURCE=PASS`；`当前证据=5`、`禁止推断=5`、`解除条件=5`、`影响=5`；退出 0 | A2-A4 | PASS |
| T2 链接与篇幅 | 相对链接存在性检查；`wc -l` | `RELATIVE_LINK_STATUS=0`；`LINES=151`；退出 0 | A5、500 行建议上限 | PASS |
| 修复 RED | 错误陈述 `rg`；首行 supporting URL 检查 | 错误 QSPI DMA 陈述可检出，两个 URL 均缺失；对应检查为 0、1、1 | A3-A5 修复前 gap | PASS |
| QSPI 证据修复 | 否定旧陈述并匹配新边界 | 新正文明确节点没有 `dmas`/`dma-names`，且不能裁决 DMA 或 PIO；退出 0 | A3、A4 | PASS |
| 首行来源修复 | 对首行匹配 docs-chip 与 tgoskits URL、版本和适用文件 | 两项匹配均退出 0 | A5 | PASS |
| 修复后结构回归 | 相对链接检查、四字段计数、`wc -l` | `RELATIVE_LINK_STATUS=0`；四字段各 5；151 行；退出 0 | A2-A5 | PASS |
| Diff 格式 | `git diff --check` | 无输出；退出 0 | 当前工作区 tracked diff 格式 | PASS |
| Change 严格校验 | `openspec validate establish-k3-storage-controller-baseline --strict` | `Change 'establish-k3-storage-controller-baseline' is valid`；退出 0 | OpenSpec change | PASS |

**Persisted Evidence**

None required. 所有验证可低成本重跑，20 行以内摘要足以支持 Acceptance；未创建 Evidence 目录。

**Experience Candidates**

None.

**Remaining Issues**

None in current Cycle. T3-T6 属于后续 Iteration，不是当前 Cycle 阻塞或遗漏。

**Commit or Diff Reference**

None.

## Plan Review

- Review Result: accepted

**Findings**

- None. 上轮两项 Blocking finding 均已修复；未发现新的 Critical、Important 或 Minor finding。

**Deviation Classification**

- ACT-DEVIATION：上轮两项均已关闭。没有新增偏差。

**Acceptance Gaps**

- None. A1-A5 均满足。

**Convergence**

- reduced → closed。QSPI DMA 证据错误和首行来源缺口均消失。

**Evidence**

- 独立读取正文、覆盖表、固定 IFX DTS、Act Response 和当前 Cycle diff。
- 四个存储入口均为 `current/active`；URL 检查为 `UNIQUE_URLS=70`、`DUPLICATES=0`。
- `spacemit-k3-com260-ifx.dts:5285-5307` 与正文 §2.2 一致：QSPI 节点没有 `dmas`/`dma-names`，正文未裁决 PIO/DMA。
- 首行同时包含 observed docs-chip datasheet、tgoskits supporting URL、revision 和适用文件；两项独立匹配退出 0。
- 相对链接检查为 `RELATIVE_LINK_STATUS=0`；`git diff --check` 退出 0；`openspec validate establish-k3-storage-controller-baseline --strict` 输出 `Change 'establish-k3-storage-controller-baseline' is valid`，退出 0。

**Follow-up Decision**

- 接受 Iteration 000 / Cycle 000。T1、T2 形成可供后续依赖的来源与非 UFS 存储基线；按已批准 Iteration Map 展开 Iteration 001 / Cycle 000 执行 T3。

**Iteration Plan Update**

None.

**Next Cycle**

None.

**Next Iteration**

`../001-ufs-protocol-data-recovery/000-initial.md`
