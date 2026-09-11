# Iteration 001 / Cycle 000: UFS 协议、数据路径与恢复基线

## Plan Context

- Status: ready
- Iteration: 001-ufs-protocol-data-recovery
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: None

**Iteration Scope**

- Change tasks: T3
- Depends on: Iteration 000（`accepted`）
- Stable baseline: UFS 从板级资源、MPHY/UniPro 到 UTP/SCSI、DMA/cache、完成和 recovery 的路径可独立检索，并保留同步轮询和第三方证据边界。
- Verification boundary: `docs/storage/k3-ufs.md` 覆盖 R1-R6 的 UFS 场景，明确 IRQ 只解析未注册、UIC 和 transfer 均轮询、timeout/OCS/fatal 后至多一次 controller recovery retry、recovery 失败后 fatal latch；首行来源、相对链接和未知项有效。
- Diagnostic boundary: UFS platform resource、link initialization、descriptor/doorbell、LUN、DMA/cache、timeout/fatal/recovery。
- Deferred tasks: T4-T6

**Cycle Scope**

- Trigger: initial
- Acceptance gaps: None
- Repair items: None
- Inherited scope: 已批准 proposal 的 MS09 范围、R1-R7、M01-M04、D1-D6、Iteration 000 的来源责任与证据等级。
- Excluded scope: QSPI/SPI/SDHCI 正文修改、known-gaps/terminology/index 收敛、驱动实现、真板 UFS I/O、性能或破坏性介质操作、无关 OpenSpec spec 修复。

**Objective**

创建一篇独立 UFS 正文，使读者能按证据等级追踪 K3/CoM260 静态资源、MPHY/UniPro/link、UTP/SCSI、descriptor 与 DMA/cache、同步完成和错误恢复，同时识别固定第三方实现不能升级为官方硬件保证的边界。

**Background**

Iteration 000 已接受：四个存储官网入口具有当前职责，非 UFS 存储正文已建立。UFS 官方 SPA 入口仍为 `partially-observed`，但覆盖表已有 2026-09-09 直接观察的 docs-buildroot raw UFS supporting row；固定 tgoskits revision `19219411d5dc1515496f910d04c93da12ee95be4` 提供可审计的 K3 UFS 同步 block driver。本 Cycle 只把这些材料整理成 UFS 主题，不修改其实现。

**Current Baseline**

- `docs/storage/k3-ufs.md` 不存在；`test ! -e docs/storage/k3-ufs.md` 退出 0。
- `docs/reference/source-coverage.md` 的 UFS 官网入口为 `current / active / partially-observed`；raw docs-buildroot UFS 行为 `supporting / observed`，唯一 URL 总数为 70。
- [Iteration 000](../000-storage-sources-and-sdhci/000-initial.md) 最终 Act Response 已通过来源唯一性、链接、格式和 change 严格校验，Plan Review 接受 A1-A5。
- 工作区已有用户和前序 milestone 的未提交内容；T3 只新增目标正文并更新当前 change 反馈，不改写这些内容。

**Current-State Evidence**

- 官方 K3 datasheet 记录 UFS 2.2、UniPro 1.6、M-PHY 3.0、HS-GEAR3/PWM-GEAR1 和直接启动能力；CoM260 板级文档确认板载 UFS，但 128 GB / 256 GB 容量冲突仍未裁决。
- 固定 IFX DTS 的 `spacemit,k3-ufshcd` 节点提供候选 MMIO、IRQ、clock/reset、clock frequency 与 lane 属性；受 G7 约束，只能作为第三方变体静态证据。
- `k3_ufs::probe` 读取 FDT `reg`、IRQ 和 lane fallback，映射 MMIO，依次执行 host → MPHY → UniPro → link pre/start/post → transfer-list setup → NOP → fDeviceInit → quirks/HS upgrade → LUN scan → `register_sync_block`。
- `init.rs` 负责 MPHY power-up/PLL lock、UniPro 属性、link startup 与 HS negotiation；无法读到 HS 能力或 HS 切换失败时保留或恢复 PWM，init/link 步骤失败向 probe 返回错误。
- `desc.rs` 定义 UTRD、UCD、PRDT 和 UPIU。单 PRDT entry 的 DBC 为 20 bit、最大 1 MiB；该数值是固定第三方实现采用的 UFSHCI layout，不应写成 K3 datasheet 保证。
- `setup_transfer_lists` 分配 UTRD/UTMRD/UCD DMA buffers，设置 list base 与 run-stop。普通 SCSI I/O 轮换使用 `0..nutrs-2`，device-management 使用保留槽 `nutrs-1`；驱动同步提交，不存在多个 outstanding 请求的并发 slot ownership。
- `prepare_slot` 填 UTRD/UCD/PRDT，对 data、UCD、UTRD 调用 `prepare_for_device`；`ring_doorbell` 在 `dma_wmb` 后写 doorbell；`poll_completion` 轮询 doorbell bit 与 fatal interrupt status，完成后对 UTRD/UCD/data 调用 `complete_for_cpu` 并检查 OCS/response。
- 驱动只解析并记录 FDT IRQ，不注册 handler，也不设置 interrupt-enable bits；UIC command 与 transfer completion 都轮询 interrupt status/doorbell。静态 IRQ 与 UTRD interrupt bit不能表述为中断推进路径。
- SCSI 层执行 NOP/QUERY、REPORT LUNS、TEST UNIT READY、INQUIRY、READ CAPACITY(10/16)、READ/WRITE(10)，选择第一个可用且优先含 MBR/GPT signature 的 regular LUN，随后以同步 block API 暴露。
- `submit_upiu` 只对 Timeout、OcsError、ControllerFatal 进入 recovery：stop lists → HCE disable → 清 doorbell → host/UniPro/link 重建 → list 重编程 → NOP 验证 → 原命令重试一次。recovery 失败设置 `fatal = true`，后续请求快速返回 ControllerFatal；第二次提交失败直接返回，不再重复 recovery。
- 当前实现不能证明未完成写入的介质状态、recovery 后数据完整性、真板性能、IRQ 模式可用性或所有 CoM260 版本的 lane/capacity 配置。

**Relevant Code**

- `docs/storage/k3-ufs.md`：本 Cycle 唯一产品目标。
- `docs/reference/source-coverage.md`：已建立的 UFS authority/supporting 来源责任，只读。
- `docs/platform/k3-soc-overview.md`、`com260-board-resources.md`：SoC 与板级 UFS 事实，只读并相对链接。
- `docs/boot/com260-boot-chain.md`：UFS 启动事实，只读并相对链接。
- `docs/dma/k3-dma-and-memory-ownership.md`、`k3-cache-pma-address-translation.md`：UFS DMA/cache/ownership 边界，只读并相对链接。
- `others/Rt-Async-AMP/tgoskits/drivers/ax-driver/src/block/k3_ufs/{mod,init,uic,desc,transfer,scsi,error}.rs`：固定第三方实现证据。
- `others/Rt-Async-AMP/tgoskits/os/StarryOS/configs/board/spacemit-k3-com260-ifx.dts`：固定第三方静态资源。

**Critical Path**

FDT probe → MMIO/lanes → host/MPHY/UniPro/link → UTRD/UTMRD/UCD lists → NOP/fDeviceInit → HS negotiation或PWM保留 → REPORT LUNS/SCSI probe → sync block registration。请求路径为 CPU 准备 UPIU/UTRD/UCD/PRDT与data → DMA publish → doorbell → 同步轮询 → DMA reclaim → OCS/response/SCSI检查；特定 controller failure 才进入一次 recovery 和一次重试。

**Implementation Guidance**

按来源与适用范围、板级/静态资源、MPHY/UniPro/link、UTP descriptor、SCSI/LUN、DMA/cache/doorbell、同步完成、错误恢复、未知项和交叉引用组织。每个数值和行为注明官方、official supporting、固定第三方或未知；不要把源码注释中的 Linux 对照升级为本项目验证结论。

**Behavioral Change**

当前 UFS 事实分散在平台、启动、DMA 文档和第三方源码。完成后，读者可从单篇正文追踪 UFS 端到端路径及失败边界；仓库运行时行为、API 和介质状态不变化。

**Change Surface**

| Task | Requirement/Scenario | File/Symbol | Current Responsibility | Planned Change |
| --- | --- | --- | --- | --- |
| T3 | R1-R6 的 UFS 场景 | `docs/storage/k3-ufs.md` | 文件不存在 | 新建 UFS 协议、数据路径与恢复正文 |

**Task Contracts**

### T3: UFS 端到端知识可按层和错误边界查询

- Requirement/Scenario: R1/S1-S2, R2/S1-S2, R3/S1-S2, R4/S1-S2, R5/S1-S2, R6/S1-S2
- Depends on: Iteration 000 accepted
- Targets: `docs/storage/k3-ufs.md`
- Current behavior: 只有分散事实和已登记来源，目标正文不存在。
- Required behavior: 正文区分官方 SoC/板级事实、official supporting 文档、固定第三方 DTS/driver 和未知项；覆盖资源、启动、MPHY/UniPro/link、UTRD/UTMRD/UCD/PRDT/UPIU、SCSI/LUN、DMA/cache、doorbell、轮询完成、同步 block 边界、timeout/OCS/fatal/recovery。
- Required changes: 创建目标文件；首行列出 UFS 官网入口、已观察 raw UFS URL、docs-chip datasheet URL 和 tgoskits URL及各自观察/revision；加入 platform/boot/DMA/gaps 相对链接；明确 IRQ 解析但未注册、UIC/transfer 轮询、单次 recovery retry、recovery failure fatal latch；至少建立覆盖容量/默认 DTS、IRQ/完成模型、cache/coherency、写入恢复完整性的四字段未知项。
- Preserve: UFS 容量冲突与 G7 不裁决；MS03 启动、MS06 DMA/cache ownership 和来源覆盖表的权威职责不复制或改写；Iteration 000 正文不修改。
- Forbidden: 不把第三方策略提升为官方 K3 保证；不声明真板、性能、数据完整性或安全重试；不把 UFS 行为外推到 QSPI/SPI/SDHCI；不修改驱动或收尾文件。
- Test witness: 修改前运行 `test ! -e docs/storage/k3-ufs.md`，预期退出 0，作为正文缺失的 RED。
- GREEN condition: 文件存在且首行为 `> 来源:`；上述层次、完成模型、恢复分支和未知项可检索；相对链接有效；建议不超过 500 行。
- Verification: `rg` 检查 UFS 2.2、M-PHY/MPHY、UniPro、UTP/UPIU、UTRD/UTMRD/UCD/PRDT、SCSI/LUN、DMA/cache/doorbell、IRQ/轮询、timeout/OCS/fatal/recovery/retry；检查四字段未知项、来源 URL、相对链接、`wc -l`、`git diff --check` 和 change 严格校验。
- Stop when: 新证据与既有 UFS/启动/DMA 基线实质冲突，需要裁决默认 DTS/容量，无法区分官方与第三方语义，或正文必须修改 T4-T6 表面才能成立。

**Invariants**

- M01-M04 保持不变；产品产出仅新增 Markdown，不修改可执行代码。
- 官网入口是 authority，已观察 GitHub 页面是 supporting，tgoskits 是固定第三方；三者不可互相替代。
- 静态 IRQ/节点、descriptor interrupt bit 和驱动解析 IRQ 都不等于 handler 已注册或运行路径采用 IRQ。
- DMA descriptor 与 data buffer 分别执行可见性动作；doorbell 清零不单独证明 data buffer 已对 CPU 可见。
- 错误恢复不能证明未完成写入无损，也不能形成安全重试保证。

**Non-goals**

- 不执行 T4-T6，不修改 index、terminology 或 known-gaps。
- 不修改 `k3-qspi-spi-sdhci.md`、来源覆盖表或第三方源码。
- 不运行真板 UFS I/O、刷写、故障注入或性能测试。

**Acceptance**

- A1 / R1-R2 / T3：正文区分 SoC、CoM260、固定 DTS 和运行时状态，说明 UFS 启动关系且不裁决容量或默认 DTS。
- A2 / R4 / T3：MPHY、UniPro、link startup、UTP/SCSI 和 LUN 层次完整，官方与第三方证据边界明确。
- A3 / R3-R4 / T3：UTRD/UTMRD/UCD/PRDT/UPIU、DMA/cache、doorbell、slot、提交、完成和回收路径可追踪。
- A4 / R3,R5 / T3：正文明确 IRQ 未注册、UIC/transfer 轮询、同步 block 访问以及 timeout/OCS/fatal 的一次 recovery retry 和 fatal latch。
- A5 / R5-R6 / T3：至少四个未知项含当前证据、禁止推断、解除条件和影响，且不声称写入无损或安全重试。
- A6 / R6 / T3：首行来源完整、相对链接有效、篇幅不超过 500 行建议上限，既有主题职责未被改写。

**Verification**

- RED：`test ! -e docs/storage/k3-ufs.md` 退出 0。
- GREEN：首行与来源 URL、层次关键词、IRQ/轮询/recovery 语义、四字段未知项、相对链接和篇幅检查全部退出 0。
- 边界：`git diff --check` 退出 0；`openspec validate establish-k3-storage-controller-baseline --strict` 退出 0。
- 全量 `openspec validate --all --strict` 的两个既有无关 spec 失败不属于本 Cycle，不运行或修复。

**Gate 2 Readiness**

| Dimension | Status | Evidence |
| --- | --- | --- |
| Investigation | PASS | 已定位 UFS authority/supporting 来源、固定 DTS、probe/init/uic/desc/transfer/scsi/error 路径、DMA/同步和错误恢复边界；目标文件缺失见证为 0 |
| Design | PASS | 既有 D1-D6 已闭合独立正文、证据等级、静态/运行时、启动、同步完成和 recovery 边界 |
| Iteration Plan | PASS | T3 唯一归属 Iteration 001，依赖已接受的 000；单篇正文形成端到端稳定基线，合并收尾任务会提前决定缺口和计数 |
| Cycle Scope | PASS | 当前只执行 T3；T4-T6 和非 UFS 表面明确排除 |
| Task Contracts | PASS | T3 含目标、当前/目标行为、保持/禁止、RED、GREEN、验证和停止条件；Act 无需回读前序 Cycle |
| Traceability | PASS | R1-R6 的 UFS 场景映射到 T3、A1-A6、目标正文与直接检查；无 Missing 或 Simplified |
| Verification | PASS | 检查目标文档行为、来源、链接、边界语义和严格校验；无身份型证据工程 |
| User Plan Approval | PASS | 用户于 2026-09-10 已批准包含 Iteration 001/T3 的完整计划、Acceptance、验证边界和 Persisted Evidence `none` |

**Persisted Evidence**

- Mode: none

Act Response 足以记录可低成本重跑的 Markdown 内容、链接、格式与 OpenSpec 校验；不创建 Evidence 目录。

- Budget: 本 Cycle 最多 5 个文件（含 README），整个 change 最多 20 个 Evidence 文件；单个文本文件最多 500 行且不超过 256 KiB；本计划不使用该预算。

**Risks and Notes**

- 官方 SPA 正文仍不可完整读取；正文必须用已观察 raw UFS 页作 supporting，不能把它提升为 authority。
- 固定第三方 driver 的 Linux 对照注释只说明实现来源，不证明官方 Linux 或 K3 硬件采用相同语义。
- `fatal` latch、单次 retry 和 LUN 选择是固定第三方策略；未知项必须保留其他实现和真板差异。
- 文档可描述读写路径，但不得给出介质写操作步骤或声称写入恢复安全。

## Act Response

- Status: reported

**Implemented**

- 按最新 Plan Review 把 change `tasks.md` 的 T3 从未完成同步为完成，使任务状态与已通过 Gate 4/Gate 5 的产品结果及 Act Response 一致。
- 按 T3 契约创建 `docs/storage/k3-ufs.md`（153 行，26 469 字节），覆盖 K3 UFS 端到端事实包：SoC 能力与板级可达性、SoC DTS 字段、UFS 启动关系、MPHY/UniPro/Link startup、UTP/SCSI descriptor（UTRD/UTMRD/UCD/PRDT/UPIU）、DMA/cache/doorbell、SCSI/LUN scan 与 sync block 边界、轮询完成模型、timeout/OCS/fatal 与单次重试、5 个四字段未知项、主题边界与导航。
- 按 Review 修正五项 Acceptance gap：初始 PWM link 与后置 HS upgrade 的阶段、lane fallback 条件、UTRD/UCD/UPIU/task tag 布局、容量冲突的未知边界、fatal latch 后恢复边界，以及 R06 carrier 与 UFS 来源覆盖行的独立状态。
- 按 D1–D6 与 A1–A6 区分证据等级、阶段、静态/运行时、启动上下文、同步完成与恢复边界；明确 IRQ 仅解析不注册、UIC/transfer 轮询、单次 recovery retry、recovery 失败 fatal latch，以及 UCD 的 512-byte command/response 区域不能推导线上的 UPIU 长度固定为 512 字节。
- 首行包含 UFS 官方入口（SPA + GitHub raw）、docs-chip k3_ds V1.8 入口、boot/image 启动上下文、tgoskits `k3_ufs/mod.rs` 固定 revision `19219411` 入口与 `spacemit-k3-com260-ifx.dts` 固定 revision `19219411` 入口；每条 URL 标注源端修订或观察日期。

**Changed Files and Symbols**

- 新建并按 Review 修正: `docs/storage/k3-ufs.md`（26469 字节，153 行；12 个二级章节、5 个四字段未知项 U1–U5、相对主题链接和首行来源保持有效）。
- 状态同步: `openspec/changes/establish-k3-storage-controller-baseline/tasks.md`（T3 `unchecked → checked`；T4-T6 保持未完成）。
- 未修改: `docs/storage/k3-qspi-spi-sdhci.md`（Iter 000 文档，git diff 为空）、`docs/boot/`、`docs/dma/`、`docs/platform/`、`docs/network/`、`docs/amp/`、`docs/interrupts/`、`docs/serial/`、`docs/reference/source-coverage.md`、`docs/reference/known-gaps.md`、`others/Rt-Async-AMP/`、`openspec/specs/`、`openspec/changes/archive/`、`openspec/changes/establish-k3-storage-controller-baseline/iterations/000-storage-sources-and-sdhci/`。
- 验证命令期间未触发任何写入；除 `?? docs/storage/k3-ufs.md` 外，本 Cycle 范围内无任何新增/修改。

**Deviations from Plan**

- 初次报告存在 Plan Review 分类为 `ACT-DEVIATION` 的五项事实偏差；本次在当前 Cycle 内按 Review 有限修复，没有改变 Task Contract、Change Surface、Acceptance、Evidence 模式或 Iteration Plan。

**Blocker Handoff**

None.

**Blocker Resolution**

None.

**Self-Review**

- Plan compliance: PASS
- Full diff reviewed: PASS（仅 `docs/storage/k3-ufs.md` 新建；本 Cycle 范围外 diff 与 MS08/R20/R21 既有变更不属于本 Cycle 责任）
- Critical findings unresolved: 0
- Important findings unresolved: 0
- Minor findings unresolved: 0
- Gate 4 阶段一（Spec compliance）: A1 的容量未知边界、A2 的 PWM/HS 阶段、A3 的 descriptor 布局、A5 的 fatal 恢复边界和 A6 的来源状态均已按 Review 修复；A1–A6 逐项复核通过。
- Gate 4 阶段二（Code quality）: 修复只触及 Review 指定段落；章节、链接和五项四字段未知项保持有效；无计划外产品改动、身份型证据工程或重复结论。
- Task status review: PASS（T1-T3 已完成，T4-T6 未完成；Current Iteration 001 与 Act Response 一致）。

**Verification Evidence**

| 验证项 | 命令或操作 | 输出摘录 | 结论 |
| --- | --- | --- | --- |
| RED 见证 | `test ! -e docs/storage/k3-ufs.md`（实施前） | `RED: target file missing`（`echo "RED: target file missing"` 触发） | PASS |
| Review 修复见证 | `rg` 检查七类旧断言（修复前） | 行 24、63、70、72、114、129、143 均命中 | PASS（重现 A1/A2/A3/A5/A6 gap） |
| Review GREEN | 精确 `rg` 排除旧断言并检索修正语义 | `STALE_ASSERTIONS_ABSENT`；PWM/HS、UCD/UPIU、task tag、来源状态、容量和 fatal 边界均命中 | PASS |
| Task 状态见证 | 修改前后检查 `tasks.md` 的 T3 checkbox | RED 命中 `- [ ] T3`；GREEN 命中 `- [x] T3`，T4-T6 仍为未完成 | PASS |
| 首行与来源 | `head -1 docs/storage/k3-ufs.md` | 以 `> 来源: https://www.spacemit.com/.../ufs.md` 开头，含 SPA UFS、GitHub UFS、k3_ds V1.8、boot、image、tgoskits mod.rs、IFX DTS 共 10 条 URL，全部带源端修订或观察日期 | PASS |
| 篇幅 | `wc -l -c docs/storage/k3-ufs.md` | `153 26469 docs/storage/k3-ufs.md`（≤ 500 行） | PASS |
| 章节完整 | `rg -n "^## " docs/storage/k3-ufs.md` | 12 个二级章节（§1 范围与证据等级 / §2 SoC 能力 / §3 静态资源 / §4 启动 / §5 MPHY/UniPro/Link / §6 UTP/SCSI descriptor / §7 DMA/cache/doorbell / §8 SCSI/LUN + sync block / §9 完成模型 / §10 错误恢复 / §11 未知项 / §12 主题边界） | PASS |
| 四字段未知项 | `rg -c "^### U[0-9]+" docs/storage/k3-ufs.md` | `5`（U1–U5，每项含「当前证据 / 禁止推断 / 解除条件 / 影响」四字段） | PASS |
| 关键词覆盖 | `rg "IRQ\|轮询\|fatal\|重试\|恢复\|不注册\|当前证据\|禁止推断\|解除条件\|影响" docs/storage/k3-ufs.md` | 在 §1 范围/§3 IRQ 字段/§6 UTP/§7 DMA/§9 完成模型/§10 错误恢复/§11 五项未知项等多处出现 | PASS |
| 相对链接 | `rg -o "\]\(([^)]+)\)"` 解析 + `test -e` 校验 | `../amp/`、`../boot/com260-boot-chain.md`、`../boot/com260-image-and-dts.md`、`../dma/k3-dma-and-memory-ownership.md`、`../interrupts/`、`../platform/com260-board-resources.md`、`../platform/k3-soc-overview.md`、`../reference/known-gaps.md`、`../reference/source-coverage.md`、`k3-qspi-spi-sdhci.md` 全部 OK | PASS |
| Git 范围 | `git diff --stat others/ docs/storage/k3-qspi-spi-sdhci.md docs/reference/ docs/boot/ docs/dma/ docs/platform/ docs/network/ docs/amp/ docs/interrupts/ docs/serial/ openspec/specs/ openspec/changes/archive/` | 仅 `docs/reference/source-coverage.md` 与 `openspec/specs/references/spec.md` 来自 MS08 既有变更（与本 Cycle 无关），其它全部空 diff | PASS |
| OpenSpec 严格校验 | `openspec validate establish-k3-storage-controller-baseline --strict` | `Change 'establish-k3-storage-controller-baseline' is valid` | PASS |

**Persisted Evidence**

- None required.
- 依据: Plan Context 明确 Persisted Evidence Mode = `none`；本 Cycle 输出可由 `wc -l` / `head -1` / `rg` / `test -e` / `openspec validate` 低成本重跑；不存在"结果无法低成本复现 / 一次性环境即将消失 / Incident 或实质 Blocker / 摘要丢失结构化信息"等白名单触发条件；`docs/storage/k3-ufs.md` 自身就是被验证目标的产物，不属于 Evidence 范畴。

**Experience Candidates**

- None。
- 实施过程未产生可端到端重复且显著风险高的操作路径（写一篇 Markdown 文档属于标准 TDD 路径）；未触发显著故障或异常恢复；现有 R18/R19 主题已经覆盖已知 Runbook 边界；MS08 已建立的「首行四类来源 / 章节命名 / 四字段未知项」模板对 UFS 同样适用，没有产生新的可重复 Runbook 候选。

**Remaining Issues**

- Iter 001 / T3 实施、内容修复和任务状态同步完成，等待 Plan 重新审计；MS09 change 的 T4-T6 归属 Iteration 002，当前 Cycle 不处理。

**Commit or Diff Reference**

- 范围: 仅 `docs/storage/k3-ufs.md`（untracked，未提交）
- 字节: 26 469 字节 / 153 行
- Git 状态: `?? docs/storage/`（与 MS08/R20/R21 既有变更同处 untracked/staged 区）
- 建议 commit message: `docs(MS09-T3): add k3-ufs.md (Iteration 001, UFS protocol/descriptor data recovery)`

## Plan Review

- Review Result: accepted

**Findings**

- None. T3 已在 change tasks 标记完成，任务状态与产品结果、Gate 4/Gate 5 和 Act Response 一致；上一版 Review 的内容与状态 gap 均已关闭。

**Deviation Classification**

None.

**Acceptance Gaps**

None. A1-A6 全部满足。

**Convergence**

reduced（T3 task 状态已同步，剩余 gap 从 1 降为 0）。

**Evidence**

- 独立内容检查：七类旧错误断言均不存在；修正后的 PWM/HS、UCD/UPIU、task tag、来源状态、容量与 fatal 边界均可检索。`wc -l -c` 输出 `153 26469`，五项未知项计数为 `5`，全部相对链接存在。
- 独立状态检查：`tasks.md` 为 T1-T3 checked、T4-T6 unchecked；T3 与 `Act Response: reported` 一致。
- 独立边界检查：来源覆盖表仍有 70 个唯一 URL、0 重复；术语表保持 25 行，known-gaps 和 index 未被 Iteration 001 修改，符合排除范围。
- 独立验证：`openspec validate establish-k3-storage-controller-baseline --strict` 输出 `Change 'establish-k3-storage-controller-baseline' is valid`；`git diff --check` 退出 0。
- 采信 Act Response：初始 RED、首行 URL 观察元数据和修复前见证无法从当前状态重建，当前内容与报告无矛盾，继续采信。

**Follow-up Decision**

接受当前 Cycle 和 Iteration 001。A1-A6、任务状态、范围和验证均无剩余阻塞；按既有 Iteration Map 展开 Iteration 002，不要求当前 Cycle 继续修复。

**Iteration Plan Update**

None.

**Next Cycle**

None.

**Next Iteration**

`../002-navigation-terminology-gaps/000-initial.md`
