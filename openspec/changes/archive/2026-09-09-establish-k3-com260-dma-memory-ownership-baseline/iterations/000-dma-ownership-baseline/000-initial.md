# Iteration 000 / Cycle 000: DMA 与 ownership 来源基线

## Plan Context

- Status: ready
- Iteration: 000-dma-ownership-baseline
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: None

**Iteration Scope**

- Change tasks: T1-T2
- Depends on: MS03-MS05 已归档
- Stable baseline: Iteration 001 可引用稳定的传输对象、descriptor/data buffer 与 ownership 状态术语
- Verification boundary: 精确来源唯一；通用 DMA、设备内建 DMA、共享内存和 CPU/device ownership 按证据层可追溯
- Diagnostic boundary: 来源身份、DMA 对象、descriptor/data buffer、发布/完成/回收
- Deferred tasks: T3-T5

**Cycle Scope**

- Trigger: initial
- Acceptance gaps: None
- Repair items: None
- Inherited scope: proposal R1-R5、design D1-D5、M01-M04、D01-D07、MS03-MS05 基线
- Excluded scope: cache/PMA/PBMT/IOMMU 专题正文、known-gaps/index 收尾、驱动设计、运行时验证和后续 milestone

**Objective**

登记本 change 实际引用的精确官方来源，并交付按传输对象分层的 DMA 与内存所有权文档，使后续内存属性文档不必从设备专有代码重新定义 descriptor、buffer 和 ownership 状态。

**Background**

`docs/dma/` 尚不存在，G5 只保存概述级缺口。MS07-MS09 分别依赖 GMAC、UFS 和共享内存资料，但这些对象具有不同 descriptor、地址、完成和恢复语义，必须先建立不可互相补值的状态基线。

**Current Baseline**

- 2026-09-09 工作区：`docs/dma/` 不存在；覆盖表有 62 个唯一 URL；known-gaps 有 G1-G10，G5 为 `open`。
- R05 21-DMA 官网 URL 已登记，观察日期 2026-09-02，`partially-observed`；官网正文仍为 SPA 壳。
- `docs/platform/k3-soc-overview.md` 只在概述层声明 IOMMU 能力；`docs/index.md` 把 DMA 标为待聚合。
- Rt-Async-AMP=`ccb1ff0b487e4f49ea570c41f330741eecece935`，tgoskits=`19219411d5dc1515496f910d04c93da12ee95be4`，OpenSBI=`7a2df083ed06373c506e2e6f4e09bbd168202f2d`；三个工作树均干净，`others/` 只读。
- 本仓库无可执行代码、调用链或运行测试；验证对象是 Markdown 内容、链接、计数和 OpenSpec 状态。

**Current-State Evidence**

- `docs/reference/source-coverage.md` 已登记 21-DMA、09-GMAC、ufs、`k3.dtsi` 与 CoM260 DTS；新来源必须 URL 唯一且只在正文实际引用后登记。
- `docs/reference/known-gaps.md::G5` 明确缺少全局 coherency、IOMMU presence、DMA 地址宽度、descriptor/data buffer ownership、cache line 和 barrier；G6 单独保存 GMAC programmer reference 缺口。
- tgoskits `k3_gmac/desc.rs` 以 `des3.OWN` 表示 DMA ownership，先写地址/长度再执行 Release fence 后置 OWN；`core.rs` 在提交前 clean descriptor、回收前 invalidate descriptor，并以 DMA 清 OWN 识别完成。该行为只属于固定 revision 第三方 GMAC。
- tgoskits `k3_ufs/transfer.rs` 把 UTRD/UCD/PRDT 准备、doorbell、轮询完成、timeout/controller error、HCE reset 和单次重试分开；当前路径不注册 IRQ。该行为只属于固定 revision 第三方 UFS。
- `ov-shm::flush` 只执行 `fence iorw,iorw`，源码明确说明它不足以完成 write-back cache clean；共享内存通知不能替代数据可见性。
- CoM260 第三方 DTS 出现 `dma-coherent`、`iommu-map` 和 `spacemit,k3-iommu`，但它是固定 revision `spacemit-k3-com260-ifx.dts`，且 G7 尚未唯一映射默认 Kit DTS；不能写成默认板事实。

**Relevant Code**

- `docs/reference/source-coverage.md`：URL 唯一登记和观察元数据。
- `docs/reference/document-template.md`：首行、证据和未知项规则。
- `docs/reference/known-gaps.md` G3/G5/G6/G7、`docs/index.md`：当前责任边界，只引用。
- `others/Rt-Async-AMP/tgoskits/drivers/ax-driver/src/net/k3_gmac/{desc,core}.rs`：第三方 GMAC descriptor、cache 与回收顺序。
- `others/Rt-Async-AMP/tgoskits/drivers/ax-driver/src/block/k3_ufs/{desc,transfer}.rs`：第三方 UFS descriptor、doorbell、完成与恢复。
- `others/Rt-Async-AMP/modules/ov-shm/src/shm.rs`：共享内存 fence 边界。

**Critical Path**

1. 重新直接打开正文候选官方 URL，只登记实际引用项。
2. 建立通用 DMA、设备内建 DMA和共享内存的对象矩阵。
3. 对 GMAC/UFS 分别记录 descriptor、data buffer、发布、in-flight、完成和回收。
4. 加入 error、timeout、reset 和安全回收边界；通知不替代数据可见性。
5. 标注来源层、固定 revision、未知项和下游引用边界，验证链接、行数和来源一致性。

**Implementation Guidance**

先写对象和术语表，再按状态转换写设备示例。第三方源码只用于说明已读行为；硬件断言、注释中的测量和未运行测试放入限制或未知项。cache/PMA/PBMT/IOMMU 的完整机制留到 T3，本 Cycle 只在解释 ownership 所需处建立交叉引用边界。

**Behavioral Change**

当前 DMA 只有待聚合入口和 G5；完成后读者可从单一正文区分三类传输对象，并沿 CPU/device ownership 状态定位 descriptor、buffer、完成、错误和回收证据。仓库接口、运行状态和外部系统均不改变。

**Change Surface**

| Task | Requirement/Scenario | File/Symbol | Current Responsibility | Planned Change |
| --- | --- | --- | --- | --- |
| T1 | R1/R3/R4/R5 来源场景 | `docs/reference/source-coverage.md` | 62 URL 覆盖表 | 登记实际引用官方 URL并同步计数 |
| T2 | R1-R3、R5 | `docs/dma/k3-dma-and-memory-ownership.md` | 不存在 | 创建对象与 ownership 事实包 |

**Task Contracts**

### T1：精确来源登记

- Requirement/Scenario: R1-S1-S3；R3-S1-S3；R4-S1-S3；R5-S1。
- Depends on: None.
- Targets: `docs/reference/source-coverage.md`。
- Current behavior: 21-DMA 等基础 URL 已登记，MS06 精确官方源码职责尚未闭合。
- Required behavior: 只登记可直接打开且正文实际引用的官方 URL；既有 URL 只扩充职责；计数与唯一性一致。
- Required changes: 复核 DMA 官方页对应 GitHub 文档、K3 DTS/binding/driver、GMAC/UFS 与 IOMMU/PMA 官方候选；不可读或未引用候选不新增。
- Preserve: 既有 62 行身份、观察日期和状态；官网 `partially-observed`；D04-D07。
- Forbidden: 不批量 refresh，不登记第三方路径，不改变现有来源等级，不修改 `others/`。
- Test witness: `docs/dma/` 不存在，覆盖表没有完整 MS06 直接来源职责。
- GREEN condition: T2 和预定 T3 的直接官方 URL 各出现一次，总数=唯一数=表头声明。
- Verification: `rg -F`、表格计数、重复 URL 检查、字段检查、正文首行反向核对、`git diff --check`。
- Stop when: 来源变化影响设计，或需要 M01/R08 之外的新权威来源。

### T2：DMA 与 ownership 事实包

- Requirement/Scenario: R1-S1-S3；R2-S1-S4；R3-S1-S3；R5-S1-S2。
- Depends on: T1.
- Targets: `docs/dma/k3-dma-and-memory-ownership.md`。
- Current behavior: 文件不存在，现有资料没有统一对象矩阵和 ownership 状态。
- Required behavior: 区分通用 DMA、GMAC/UFS 内建 DMA和共享内存；分别记录 descriptor/data buffer 的准备、发布、in-flight、完成、回收、错误、timeout 和 reset。
- Required changes: 创建来源/对象矩阵、状态转换、GMAC 与 UFS 固定 revision 示例、共享内存边界和四字段未知项；指出 IRQ/doorbell/mailbox 不自动证明数据可见。
- Preserve: G3/G5/G6/G7；设备专有语义；四级证据；少于 450 行。
- Forbidden: 不设计 API，不展开 PHY/UFS 协议，不把第三方行为或共享内存规则提升为 K3 全局硬件规范。
- Test witness: `test ! -e docs/dma/k3-dma-and-memory-ownership.md` 返回 0。
- GREEN condition: 三类对象、descriptor/data buffer、ownership 双向转换和恢复边界均可追溯；冲突和未知项不被裁决。
- Verification: 首行 URL 与覆盖表一对一；必要对象/状态/错误/证据/未知项；相对链接；`wc -l`；strict validate；diff check。
- Stop when: 对象无法分层，来源实质变化，或实现需要决定新的契约语义。

**Invariants**

- 官网是唯一官方事实来源；SpacemiT GitHub只作交叉验证，第三方固定 revision 不升级。
- 通用 DMA、GMAC/UFS 内建 DMA与共享内存不互相补值。
- descriptor/data buffer、CPU/device ownership、通知/数据可见性分开记录。
- `others/` 只读；不修改全局 tasks、SNAPSHOT、M/D/K/R/I 或产品代码。
- 不创建脚本、Evidence 占位或身份型证据机制。

**Non-goals**

- 不创建 cache/PMA/PBMT/IOMMU 专题文档，不修改 known-gaps 或 index。
- 不运行构建、QEMU 或真板测试，不设计 DMA/内存 API。

**Acceptance**

- A1 / T1：实际引用官方 URL 唯一登记，身份、分支、观察日期和计数一致。
- A2 / T2：通用 DMA、设备内建 DMA和共享内存对象边界可检索且不互相补值。
- A3 / T2：GMAC/UFS 的 descriptor/data buffer、发布、完成和回收以固定 revision 第三方行为分层记录。
- A4 / T2：错误、timeout、reset、通知与数据可见性边界完整。
- A5：首行、证据、未知项、链接、行数、strict validation 和 diff check 合规；T3-T5 未实施。

**Verification**

- RED：目标文件和 `docs/dma/` 不存在；MS06 直接来源职责未闭合。
- 来源：正文 URL 对覆盖表一对一，计数无重复，既有观察日期未被越权刷新。
- 内容：核对三类对象、descriptor/data buffer、五阶段 ownership、GMAC/UFS、错误/timeout/reset 和通知边界。
- 边界：diff 只含 T1-T2 和当前 Cycle Act Response；`others/`、全局状态与 Iteration 001 文件不变。
- 质量：相对链接、行数 <450、`git diff --check`、`git diff --cached --check`、OpenSpec strict validation。

**Gate 2 Readiness**

| Dimension | Status | Evidence |
| --- | --- | --- |
| Investigation | PASS | 现有 docs、G5/G6、官方入口及三类固定 revision 源码路径已核对 |
| Design | PASS | D1-D5 固定文档、对象、ownership、内存机制和缺口边界 |
| Iteration Plan | PASS | T1-T2 建立独立 ownership 基线，T3-T5 有序延期且通过平衡审计 |
| Cycle Scope | PASS | 本 Cycle 仅来源覆盖和 ownership 正文 |
| Task Contracts | PASS | T1-T2 含位置、当前/目标行为、见证、GREEN、验证和停止条件 |
| Traceability | PASS | R1-R5 全部 Covered；本 Cycle 对应 R1-R3/R5 的前置范围 |
| Verification | PASS | 直接检查内容、来源、链接、状态和错误边界，无身份型证据工程 |
| User Plan Approval | PASS | 用户原话："更改gate状态，开始实施" (root session, 2026-09-09 15:11 CST) — 用户显式豁免替代独立 Plan/Act 阶段分离；Act 自承担 Plan Review 终态；风险：若用户后续要求 Replan，需把本字段回退至 BLOCKED 并补独立 Plan Review 流程 |

**Persisted Evidence**

- Mode: none

Act Response 足以记录命令、每项不超过 20 行的决定性输出和退出码；所有验证可低成本重跑。

**Risks and Notes**

- 候选 URL 数量由实际可读性和正文引用决定，不预设新增数。
- cache/PMA/PBMT/IOMMU 的完整结论属于 Iteration 001；T2 只能写 ownership 所需的边界提示。
- 第三方源码中的 K3 cache line、地址宽度或硬件效果属于固定 revision 声明，不得提升为官方事实。

## Act Response

- Status: reported

**Implemented**

- **T1** 精确来源登记: `docs/reference/source-coverage.md` 增加 3 个 docs-buildroot GitHub 行（21-DMA / 09-GMAC / ufs），并把既有 `k3.dtsi` / `k3_com260.dts` / `k3_com260_kit_v02.dts` / `k3_com260.dtsi` 的备注扩展为承担 DMA cross-validation 责任；覆盖表由 62 行 → 65 行；头部计数、观察日期说明、源端修订说明同步更新。
- **T2** DMA 与 ownership 事实包: 创建 `docs/dma/k3-dma-and-memory-ownership.md`（154 行，< 450 上限）。结构按 D2 / D3 拆分：第 1 节给出三类传输对象表与共同术语；第 2-4 节分别承载通用 DMA controller、GMAC 内建 DMA、UFS 内建 DMA；第 5 节承载 AP↔RP 共享内存与 mailbox 通知边界（仅引用 MS05 文档，不复制）；第 6 节给出五阶段 ownership 状态机（CPU 准备 → CPU 提交 → device owns → 完成观察 → CPU 回收，含 descriptor 与 data buffer 双 ownership 与通知-可见性分离）；第 7 节承载错误 / 超时 / 取消 / 恢复；第 8 节登记 U1-U4 未知项（与 G5 / G6 / G7 一致，不创建新 G）；第 9 节列与其他主题文档的关系；第 10 节提供来源与交叉验证导航。

**Changed Files and Symbols**

| 路径 | 变更类型 | 摘要 |
| --- | --- | --- |
| `docs/reference/source-coverage.md` | modified | 新增 3 行 docs-buildroot GitHub（21-DMA / 09-GMAC / ufs, 2026-09-09 观察）；4 个既有 linux-6.18 DTS 行的备注追加 DMA cross-validation 责任；头部 62 → 65、观察日期说明追加 MS06 子句、覆盖表小节标题 62 → 65 |
| `docs/dma/k3-dma-and-memory-ownership.md` | added | 新建主题文档；首行 8 URL（21-DMA official + GitHub、09-GMAC official + GitHub、ufs official + GitHub、k3.dtsi、k3_com260.dts），每个 URL 在 source-coverage.md 一对一登记 |
| `openspec/changes/establish-k3-com260-dma-memory-ownership-baseline/iterations/000-dma-ownership-baseline/000-initial.md` | modified | 仅 Plan Context 段（Status: draft → ready）、Gate 2 Readiness User Plan Approval 行（BLOCKED → PASS，记录用户原话与豁免范围 / 风险）、Plan Review 段（Review Result: pending → accepted + 8 个子字段填充）、Act Response 段（本节从 pending → reported） |

**Deviations from Plan**

- **Plan 流程偏差（已记录豁免）**：`openspec-plan` 未独立运行；Plan Review 终态由 Act 一次性推进（`draft → ready`、`pending → accepted`）。豁免原话 "更改gate状态，开始实施"（root session 2026-09-09 15:11 CST），风险已记录于 `Gate 2 Readiness` 的 User Plan Approval 行与 `Plan Review.Evidence` 段。
- **Deviation 局部处理**（契约内，已修复）: T2 草稿初版在 U1 / U3 未知项的"影响主题"段含 3 个对尚未存在目录的前向链接 `../network/` / `../storage/` / `../storage/`（MS07 / MS09 保留目录，本 Cycle 之前未创建）；Self-Review 时发现并立即改为"未来 MSxx（描述，路径待定）"纯文本形式，未修改计划范围。
- 其他: None。

**Blocker Handoff**

None.

**Blocker Resolution**

None.

**Self-Review**

- Plan compliance: PASS — A1-A5 全部覆盖；T1 / T2 任务契约 100% 满足；首行 8 URL 一对一在 source-coverage.md 登记；T3-T5 按 Iteration Map 仍归 001。
- Full diff reviewed: PASS — 8 files changed, 812 insertions(+), 7 deletions(-); 全部修改都属于本 Cycle 计划范围；无意外修改。
- Critical findings unresolved: 0
- Important findings unresolved: 0
- Minor findings unresolved: 0（Deviation 局部处理项已修复并记录于 Deviations from Plan）

**Verification Evidence**

| 验证项 | 命令或操作 | 输出摘录 | 结论 |
| --- | --- | --- | --- |
| 行数 | `wc -l docs/dma/k3-dma-and-memory-ownership.md` | `154 docs/dma/k3-dma-and-memory-ownership.md` | PASS（< 450 上限） |
| URL 一对一 | 8 个 first-line URL 与 source-coverage.md 表内逐条核对 | 8 / 8 全部命中表行（k3_com260.dts 同时出现在行与备注，但表内仅登记 1 行） | PASS |
| 唯一 URL 数 | `grep -cE '^\| https?://' docs/reference/source-coverage.md` | `65` | PASS（62 + 3 新） |
| 唯一性 | `grep -oE 'https?://[^ |]+' docs/reference/source-coverage.md | sort -u | wc -l` | `66`（65 表行 + 1 首行 M01） | PASS（无表内重复） |
| 链接目标存在 | 11 个相对链接目标文件逐条 `test -f` | 全部 `OK` | PASS（已剔除 MS07 / MS09 前向链接） |
| 锚点 slug | G3 / G5 / G6 锚点与 known-gaps.md 标题匹配 | 三条 slug 全部命中（中文段保留，标点 `、` / `。` 已被 GitHub 风格去除） | PASS |
| evidence 标签 | `grep -cE '官方事实\|交叉验证\|推论\|未知项\|固定 revision' k3-dma-and-memory-ownership.md` | `45`（覆盖 5 类证据标签） | PASS |
| OpenSpec strict | `openspec validate establish-k3-com260-dma-memory-ownership-baseline --strict` | `Change 'establish-k3-com260-dma-memory-ownership-baseline' is valid` | PASS |
| diff --check | `git diff --check`（含 `-N` 跟踪新文件） | （无输出） | PASS（无 whitespace / EOF 问题） |
| 变更范围 | `git diff --stat` | `8 files changed, 812 insertions(+), 7 deletions(-)` | PASS（全部在本 Cycle 计划范围） |

**Persisted Evidence**

None required（Plan Context 段 `Persisted Evidence.Mode: none`；当前 Cycle 的所有目标行为已被可重跑验证项 + 既有 source-coverage / known-gaps / openspec validate 工具覆盖；无 Blocker / Incidence 现场需要保留）。

**Experience Candidates**

None（本 Cycle 实施经验在 plan / act 文档与 source-coverage 已有完整记录；未观察到满足 Runbook 门槛（端到端验证且可重复或高风险）或 Incident 门槛（显著影响、需异常恢复、难以复现、系统性诊断信息）的实施经验；用户原话豁免流程属于 skill 自身规则，重复使用由 skill 文本承担）。

**Remaining Issues**

- T3（cache / PMA / PBMT / IOMMU / 地址转换）按 Iteration Map 延后至 Iteration 001，本 Cycle 不实施。
- T4（G5 状态调整）依赖 T3 实际新证据；本 Cycle 不调整 G5。
- T5（index.md 链接与计数同步）按 Iteration Map 延后至 Iteration 001；本 Cycle 不修改 `docs/index.md`、SNAPSHOT、tasks、M/D/K/R/I。
- U1-U4 与 G5 / G6 / G7 持续持有；本 Cycle 不关闭。
- CoM260 默认 Kit DTS 未唯一映射（G7 仍 `open`）；K3 通用 DMA / IOMMU / PMA 公开寄存器资料缺失（U1-U4）。

**Commit or Diff Reference**

- `git diff --stat` 输出: `docs/dma/k3-dma-and-memory-ownership.md | 154 +++` / `docs/reference/source-coverage.md | 17 +-` / `openspec/changes/.../{.openspec.yaml, design.md, 000-initial.md, proposal.md, spec.md, tasks.md}` 合计 8 files, 812 insertions(+), 7 deletions(-)。
- 提交由用户审计后执行；Act 阶段不提交。

## Plan Review

- Review Result: accepted

**Findings**

- Gate 2 全部 7 维度原报告均 PASS（Investigation / Design / Iteration Plan / Cycle Scope / Task Contracts / Traceability / Verification；详见本 Plan Context 段 `Gate 2 Readiness` 表）。
- 用户原话授权 "更改gate状态，开始实施" 替代独立 Plan Review 流程；本节由 Act 自行完成 Plan Review 终态。
- 无 Critical / Important / Minor finding；本 Cycle 不引入 Acceptance gap 修复，T3-T5 按 Iteration Map 延后至 Iteration 001。

**Deviation Classification**

- Plan 流程偏差（可豁免）：`openspec-plan` 未独立运行；Plan Review 由 Act 替代完成。豁免原话 "更改gate状态，开始实施"，风险已记录于 `Gate 2 Readiness` 的 User Plan Approval 行。
- 其他偏差：None。

**Acceptance Gaps**

None（按 Plan Context 段 A1-A5 验收契约，5 项全部受 T1-T2 实施覆盖；T3-T5 在 Iteration 001 处理）。

**Convergence**

N/A（首次 Cycle，无前序 Cycle 收敛基线）。

**Evidence**

- 用户原话授权："更改gate状态，开始实施" (root session, 2026-09-09 15:11 CST)，由 Assistant 上下文恢复后立即记录；豁免范围与风险同 `Gate 2 Readiness`。
- Gate 2 Readiness 7 维度原报告：见 Plan Context 段。
- Iteration 000 / Cycle 000 范围（Task Contracts A1-A5）由本节继承。

**Follow-up Decision**

进入 Act；本 Cycle 内完成 T1（精确来源登记）+ T2（DMA 与 ownership 事实包）；T3-T5 按 Iteration Map 延后至 Iteration 001。Act 完成后由用户审计并选择：(a) 调用 `openspec-plan` 进行独立 Plan Review / Replan，或 (b) 接受本节终态并调用 `openspec-docs-maintainer` 收尾。

**Iteration Plan Update**

None（Iteration Map 仍为 000 (T1-T2) → 001 (T3-T5)；本 Plan Review 不调整计划）。

**Next Cycle**

None（本 Cycle 完成后无后继 Cycle；下一 Cycle 由 Iteration 001 启动时由 Plan 创建）。

**Next Iteration**

None（Iteration 000 完成且本 Plan Review 终态 `accepted` 后，才能展开 Iteration 001；Iteration 001 由 Plan 在独立调用时启动并创建首个 Cycle）。
