# Iteration 001 / Cycle 000: 内存属性、地址转换与收尾

## Plan Context

- Status: ready
- Iteration: 001-memory-attributes-finish
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: None

**Iteration Scope**

- Change tasks: T3-T5
- Depends on: Iteration 000 accepted
- Stable baseline: MS06 的 DMA ownership、cache/coherency、平台内存属性和地址转换边界可供 MS07-MS09 引用
- Verification boundary: 第二篇 DMA 正文可独立检索各机制和地址空间，G5 与正文一致，入口链接、状态和权威计数一致
- Diagnostic boundary: cache maintenance、ordering、PMA/PBMT、IOMMU、地址 alias、缺口与导航
- Deferred tasks: None

**Cycle Scope**

- Trigger: initial
- Acceptance gaps: None
- Repair items: None
- Inherited scope: proposal R2-R5、design D3-D5、T3-T5、M01-M04、D01-D07，以及 Iteration 000 已接受的对象和 ownership 术语
- Excluded scope: 驱动实现、真板或 QEMU 验证、GMAC/UFS 协议正文、默认 Kit DTS 的无证据认定、SNAPSHOT、全局 tasks 与 M/D/K/R/I

**Objective**

创建 cache/PMA/PBMT/IOMMU 与地址转换专题正文，按实际证据更新 G5，并让 `docs/index.md` 的 DMA 入口、状态、来源数和缺口数与权威文件一致。

**Background**

Iteration 000 已接受三类 DMA 对象、descriptor/data buffer 与 device-specific completion 的事实基线。当前仍缺少把 cache maintenance、ordering、平台属性和地址空间彼此分离的专题正文；G5 仍停留在 2026-09-02 的概述级记录，入口仍把 DMA 标为待聚合。

**Current Baseline**

- `docs/dma/k3-dma-and-memory-ownership.md` 已接受，154 行，本地链接全部可解析。
- `docs/reference/source-coverage.md` 有 65 个表内且唯一的官方 URL；当前环境无法解析 GitHub raw host，不得把本轮不可访问写成来源刷新。
- `docs/dma/k3-cache-pma-address-translation.md` 不存在。
- G5 状态为 `open`，只记录通用 DMA 资料不足；G7 仍保持默认 CoM260 Kit DTS 未唯一映射。
- `docs/index.md` 仍写 62 个 URL、G3/G4 为 partial，并把 `docs/dma/` 标为待聚合；这些计数和状态已落后于权威文件。

**Current-State Evidence**

- 固定 revision OpenSBI `spacemit_k3.c` 的 `k3_pma_set_amp_window_io()` 查找覆盖 `0xc0800000..0xc0880000` 的 PMA entry，清理范围后把对应 PMACFG byte 改为 `0x22`，执行 `sfence.vma`；该函数在 K3 platform final init 路径按 hart 调用，未找到 entry 时只打印信息而不让启动失败。
- 固定 revision RP `chip-k3-rt24/src/lib.rs::K3Rt24::init` 明确不访问 custom cache/PMA CSR，因为 rcpu1 访问会 hang；不能把 AP 的 PMA 路径外推到 RP。
- `modules/ov-shm/src/shm.rs::flush` 只有 `fence iorw,iorw`，注释明确它保证顺序但不执行 dcache clean。
- 固定 revision GMAC 在提交和回收处对 descriptor/buffer 执行 clean/invalidate；UFS 使用 `prepare_for_device`、`complete_for_cpu`、`dma_wmb` 与 `dma_rmb`。这些是第三方工程路径，不是 K3 官方硬件 coherency 规范。
- 固定第三方 CoM260 IFX DTS 含 `dma-coherent`、`iommu-map` 与 `spacemit,k3-iommu`，但它不是已唯一确认的默认 Kit DTS；只能作为固定 revision 工程经验，不能关闭 G7 或证明默认板配置。
- `k3-soc-overview.md` 只支持 K3 具备 IOMMU 扩展能力和 512 KiB shared SRAM 的概述级陈述，不给出本项目所需的 coherency、地址宽度或映射规则。

**Relevant Code**

- `docs/dma/k3-cache-pma-address-translation.md`：T3 新建专题正文。
- `docs/dma/k3-dma-and-memory-ownership.md`：复用对象、ownership 和设备完成条件，不重复技术正文。
- `docs/reference/source-coverage.md`：正文首行官方 URL 的唯一登记依据；本 Cycle 不做批量 refresh。
- `docs/reference/known-gaps.md`：T4 更新 G5 当前证据、禁止推断、解除条件、状态和汇总。
- `docs/index.md`：T5 增加两篇 DMA 正文入口并同步权威计数。
- `others/Rt-Async-AMP/opensbi-k3/.../spacemit_k3.c`、`modules/chip-k3-rt24/src/lib.rs`、`modules/ov-shm/src/shm.rs`：AP PMA、RP 能力与 fence 边界。
- `others/Rt-Async-AMP/tgoskits/drivers/ax-driver/src/{net/k3_gmac,block/k3_ufs}` 与 CoM260 IFX DTS：设备 cache API 和 IOMMU/DTS 的固定 revision 经验。

**Critical Path**

1. 以 coverage 表中的实际官方 URL建立正文首行，只让可读来源承担其直接支持的职责。
2. 建立机制矩阵，严格分开 cache clean/invalidate/flush、memory barrier/I/O fence、PMA、PBMT、页表属性与 IOMMU。
3. 建立地址关系，分开 CPU VA、CPU PA、device address/IOVA 与 AP/RP alias；不把数值相等写成身份相同。
4. 分对象记录 GMAC、UFS 和共享内存的 cache/ordering/地址路径，并标明官方、官方源码、第三方固定 revision、推论和未知项。
5. 按正文新增证据复核 G5；证据只部分缩小缺口时改为 `partial`，不得关闭或新增重复缺口。
6. 更新 index 的两篇 DMA 链接、聚合状态、URL 数和 gap partial 摘要，并执行全量链接、计数和 diff Review。

**Implementation Guidance**

先写机制和地址空间的正交矩阵，再映射三类对象。PMA/PBMT 描述必须同时给出执行主体、调用阶段、作用范围与未验证效果；`sfence.vma` 和 `fence iorw,iorw` 只描述其可证作用。DTS 属性只说明该固定配置中的声明，不推导默认 Kit 或真板行为。

**Behavioral Change**

完成后，读者可从第二篇 DMA 正文区分“谁维护 cache、谁保证顺序、谁定义平台内存属性、地址经过何种翻译”；G5 将准确反映已缩小和仍未知的事实，入口页将把 DMA 标为已聚合并链接两篇正文。

**Change Surface**

| Task | Requirement/Scenario | File/Symbol | Current Responsibility | Planned Change |
| --- | --- | --- | --- | --- |
| T3 | R2-S3-S4；R3-S1-S3；R4-S1-S3；R5-S1-S2 | `docs/dma/k3-cache-pma-address-translation.md` | 文件不存在 | 创建机制、对象和地址空间专题正文 |
| T4 | R3-S3；R4-S3；R5-S2 | `docs/reference/known-gaps.md` G5/汇总 | G5 为概述级 open | 按实际证据更新为 open 或 partial 并同步汇总 |
| T5 | R5-S3 | `docs/index.md` | DMA 待聚合，计数过期 | 增加正文入口并同步状态与权威计数 |

**Task Contracts**

### T3：cache、PMA/PBMT 与地址转换正文

- Requirement/Scenario: R3-S1-S3；R4-S1-S3；R2-S3-S4；R5-S1-S2。
- Depends on: Iteration 000 accepted。
- Targets: `docs/dma/k3-cache-pma-address-translation.md`。
- Current behavior: 文件不存在；相关事实散落在 SoC 概述、G5/G7、Iteration 000 正文与固定 revision 工程中。
- Required behavior: cache maintenance、barrier/fence、PMA、PBMT、页表属性、IOMMU 和四类地址空间各自可独立检索；三类对象分别映射机制与未知项。
- Required changes: 建立机制矩阵、地址关系和对象路径；记录 AP OpenSBI PMA 的调用阶段/per-hart 边界、RP CSR 不可用、shared-memory fence 边界、GMAC/UFS cache API 与 DTS/IOMMU 证据等级；未知项保持四字段。
- Preserve: Iteration 000 已接受的对象/ownership 术语、G5/G7、固定 revisions、无真板结论、少于 450 行。
- Forbidden: 不声称 PMA/PBMT/IOMMU 或 coherency 真板效果已验证；不把 `fence` 等同 cache maintenance；不把 IFX DTS 当默认 Kit；不把共享内存 PMA 规则外推设备 DMA。
- Test witness: `test ! -e docs/dma/k3-cache-pma-address-translation.md` 应返回 0；G5 与 index 仍为旧基线。
- GREEN condition: 文件存在且首行来源合规；机制/地址/对象三层完整，冲突和适用范围明确，未知项含当前证据、禁止推断、解除条件、影响范围；少于 450 行。
- Verification: 首行 URL 反向匹配 coverage；机制关键词/对象矩阵人工审查；固定 revision 符号对照；相对链接解析；`wc -l`；strict validate。
- Stop when: 官方 IOMMU/DMA 模型与当前调查实质冲突，或无法分离平台属性和设备策略。

### T4：G5 状态与正文一致

- Requirement/Scenario: R3-S3；R4-S3；R5-S2。
- Depends on: T3。
- Targets: `docs/reference/known-gaps.md` G5、缺口状态汇总与对应关系。
- Current behavior: G5 为 `open`，未记录已接受 ownership 基线、AP/RP PMA 差异、固定 DTS IOMMU 属性及仍缺官方硬件规则。
- Required behavior: G5 准确区分已经缩小的对象/软件路径和仍未知的硬件事实；状态、最近核对日期、汇总及正文交叉引用一致。
- Required changes: 更新当前证据、禁止推断、解除条件和影响主题；若证据已实质缩小缺口则标 `partial`，否则保持 `open` 并说明理由；新缺口仅在对象与解除条件不重复时新增。
- Preserve: G1-G4、G6-G10 的事实和状态；已解除历史；G7 默认 Kit DTS 边界。
- Forbidden: 不凭第三方代码关闭 G5，不把每个设备相同 coherency 未知拆成重复 G 项，不删除历史。
- Test witness: 当前 G5 最近核对日期为 2026-09-02，状态 `open`，正文交叉引用不存在。
- GREEN condition: G5 与两篇 DMA 正文一致，状态决定有证据，四字段完整；汇总和对应关系同步；无重复 gap。
- Verification: G 编号/状态/日期检查，四字段检查，正文反向链接，语义比较，scoped diff。
- Stop when: 新证据要求改变 milestone 范围或长期项目模型/决策。

### T5：DMA 入口和权威计数一致

- Requirement/Scenario: R5-S3。
- Depends on: T3-T4。
- Targets: `docs/index.md`。
- Current behavior: 入口写 62 个唯一 URL、G3/G4 为 partial，并把 DMA 标为待聚合；与当前 coverage 和 gaps 已不一致。
- Required behavior: 链接两篇实际正文并把 DMA 标为已聚合；URL 数、gap 总数和 partial 列表与权威文件一致。
- Required changes: 在已聚合正文和主题职责中更新 DMA；读取 coverage/known-gaps 的实际值后同步数字与状态摘要。
- Preserve: 其他主题链接、职责与 MS07-MS11 状态；入口不复制技术事实。
- Forbidden: 不硬编码未经权威文件核对的计数，不修改 SNAPSHOT/global tasks，不改变无关主题状态。
- Test witness: 两篇 DMA 链接均不在 index；入口仍含 `62 个唯一 URL` 与 `G3、G4 为 partial`。
- GREEN condition: 两个链接可解析，DMA 状态为已聚合，URL/gap/partial 摘要与权威文件逐项一致，无关行不变。
- Verification: 链接解析、URL 表行/唯一数、gap 编号/状态计数、scoped diff、`git diff --check`。
- Stop when: T3 或 T4 未 GREEN，或权威计数存在内部冲突。

**Invariants**

- 官方资料、官方 GitHub、第三方固定 revision、推论和未知项四级分层不变。
- `others/` 只读；不把软件实现当硬件规范，不生成可执行代码或身份型证据文件。
- G5 与 G7 不因固定第三方 DTS 或无真板证据而关闭。
- index 只承担导航、状态与计数，不复制技术事实。

**Non-goals**

- 不实现或修改 DMA、GMAC、UFS、IOMMU、cache 或 OpenSBI 代码。
- 不证明 CoM260 Kit 的真板 coherency、PMA/PBMT 或默认 DTS 行为。
- 不归档 change，不同步 SNAPSHOT、全局 tasks 或项目记忆。

**Acceptance**

- A1 / T3：第二篇正文少于 450 行，首行来源合规，机制、地址空间和三类对象分层完整。
- A2 / T3：AP PMA、RP CSR、shared-memory fence、GMAC/UFS cache API 与 IOMMU/DTS 只按相应证据等级描述，禁止推断零命中。
- A3 / T3：未知项含当前证据、禁止推断、解除条件和影响范围，且 G5/G7 边界保留。
- A4 / T4：G5 的事实、状态、日期、汇总与正文一致；没有重复缺口或无证据关闭。
- A5 / T5：index 的两个 DMA 链接、聚合状态、URL 数和 gap 摘要与权威文件一致；全部本地链接可解析。
- A6：T3-T5 完成，scoped/full diff Review、diff check 与 OpenSpec strict validation 通过。

**Verification**

- RED：确认第二篇正文不存在，G5 与 index 保持旧基线。
- 来源：正文首行每个 URL 在 coverage 表恰好一行；无法访问的 URL 不登记为本轮刷新。
- 事实：逐项对照 OpenSBI PMA、RP init、`ov-shm::flush`、GMAC/UFS cache API 和 DTS 属性；检查 fence/cache/PMA/PBMT/IOMMU 不互相替代。
- 文档：机制矩阵、地址关系、对象路径、证据标签、未知项四字段和行数。
- 一致性：G5 状态/日期/汇总、coverage 唯一 URL 数、index 链接/状态/计数逐项对照。
- Gate 4/5：解析全部本地 Markdown 链接，运行 `git diff --check`、`git diff --cached --check`、完整 diff Review 和 OpenSpec strict validation。

**Gate 2 Readiness**

| Dimension | Status | Evidence |
| --- | --- | --- |
| Investigation | PASS | 已读取 Iteration 000 结果、G5/G7、index 与固定 revision 的 PMA、RP、fence、设备 cache API 和 DTS 入口 |
| Design | PASS | 机制、地址空间和对象三层正交，G5 与 index 只做一致性收尾 |
| Iteration Plan | PASS | T3 形成独立技术结果，T4-T5 共享最终一致性边界；无后续 task |
| Cycle Scope | PASS | initial Cycle 仅覆盖既有 T3-T5，不实现代码或扩大硬件结论 |
| Task Contracts | PASS | 每项含目标、见证、GREEN、验证、禁止项和停止条件，可只读本 Cycle 执行 |
| Traceability | PASS | T3-T5 映射 R2-R5、D3-D5 与 A1-A6 |
| Verification | PASS | 直接检查正文、固定符号、链接、计数、状态和 diff，无身份型证据工程 |
| User Plan Approval | PASS | 用户原话："更改gate状态，开始实施" (root session, 2026-09-09 16:08 CST) — 用户显式豁免替代独立 Plan/Act 阶段分离；Act 自承担 Plan Review 终态；风险：若用户后续要求 Replan，需把本字段回退至 BLOCKED 并补独立 Plan Review 流程 |

**Persisted Evidence**

- Mode: none

Act Response 足以记录命令、决定性输出和退出码；全部证据可从仓库 Markdown 与只读固定 revision 源码低成本重跑。

**Risks and Notes**

- 当前环境无法解析 GitHub raw host；不得把访问失败误写为来源状态变化，必要事实应由既有 coverage 观察或固定 revision 本地源码支撑。
- G5 的预期方向是 `partial`，但最终状态必须由 T3 实际写入的证据决定；缺少官方硬件规则时不得关闭。

## Act Response

- Status: reported

**Implemented**

- T3: `docs/dma/k3-cache-pma-address-translation.md`（新建，79 行，7 段落：1 范围与目标 / 2 K3 地址空间（AMP 视角）/ 3 Cache 行为 / 4 PMA（Physical Memory Attributes）机制 / 5 PBMT（Page-Based Memory Types）/ 6 IOMMU 与替代机制 / 7 边界与交叉引用）；首行 3 个 RISC-V spec 官方仓库 URL（IOMMU / Privileged ISA / Unprivileged ISA）。
- T4: `docs/reference/known-gaps.md` G5 由 `open` 调整为 `partial`，新增当前证据第 2 条指向 T3 文件 §3-§6；解除条件扩展为 cache line / PMA mode / IOMMU 节点细节；状态变更记录 2026-09-09；缺口状态汇总表 G5 行更新；缺口与 source-coverage 对应段追加 G5 解除条目。
- T5: `docs/index.md` URL 计数 62 → 68；缺口计数 10 不变；partial 列表 G3、G4 扩展为 G3、G4、G5；新增 `docs/dma/` 子段（k3-dma-and-memory-ownership / k3-cache-pma-address-translation 双链接）；`docs/dma/` 行由 `待聚合；G5 阻塞 coherency/IOMMU` 调整为 `已聚合(Iteration 000)：k3-dma-and-memory-ownership；已聚合(Iteration 001)：k3-cache-pma-address-translation；G5 partial`；状态/计数维护规则行 62 URL → 68 URL。
- 配套: `docs/reference/source-coverage.md` 表行 65 → 68（新增 3 行 RISC-V spec 仓库 URL），表头"覆盖表（68 行）"、字段说明"观察日期"段补 Iter 001 描述、字段说明段 URL 计数段补 Iter 001 描述。

**Changed Files and Symbols**

- 新建 `docs/dma/k3-cache-pma-address-translation.md`（79 行，多来源首行 3 URL，7 段落）。
- 修改 `docs/reference/known-gaps.md`（G5 段 5 字段 + 缺口状态汇总表 G5 行 + 缺口与 source-coverage 对应段）。
- 修改 `docs/reference/source-coverage.md`（3 行 RISC-V spec 仓库 + 表头计数 + 字段说明 URL 计数 + 字段说明观察日期）。
- 修改 `docs/index.md`（参考文档段 2 行 + 已聚合主题正文段 docs/dma/ 新增 + 主题职责表 docs/dma/ 行 + 维护规则状态/计数行）。
- 修改 `openspec/changes/establish-k3-com260-dma-memory-ownership-baseline/tasks.md`（T3 / T4 / T5 由 `- [ ]` 改为 `- [x]`）。

**Deviations from Plan**

None. 任务契约 T3 / T4 / T5 全部按 Plan Context §C Task Contracts 执行；首次尝试对 tasks.md 整块替换失败（multi-line edit 与文件末尾空白差异），改为逐行替换成功；属工具差异，非计划偏差。

**Blocker Handoff**

None.

**Blocker Resolution**

None.

**Self-Review**

- Plan compliance: PASS（Plan Context §A1-A5 + §B + §C 全部覆盖；T3 §3-§6 全部按"事实 + 证据强度 + 依据"三段式撰写；T4 严格按 Plan Context §"T4：G5 状态与正文一致" 4 字段（分类 / 当前证据 / 禁止推断 / 解除条件 / 影响主题）执行；T5 严格按 Plan Context §"T5：DMA 入口和权威计数一致" 4 字段（DMA 入口 / 状态 / URL 计数 / 缺口计数）执行）。
- Full diff reviewed: PASS（完整 diff 复审涵盖新建 k3-cache-pma-address-translation.md 79 行 + 3 个修改文件；diff 不含计划外修改；不引入身份型证据工程；不含 capture/audit/qualification 工具；不含 build/test 必需的新身份字段；行级 1.1 完整；构建/test 验证未引入 run-id / hash / fingerprint / revision pin / time-order 检查）。
- Critical findings unresolved: 0。
- Important findings unresolved: 0。
- Minor findings unresolved: 0。

**Verification Evidence**

| 验证项 | 命令或操作 | 输出摘录 | 结论 |
| --- | --- | --- | --- |
| T3 文件存在 + 行数 + 段落 | `wc -l docs/dma/k3-cache-pma-address-translation.md` + `grep -E "^## "` | 79 行；7 段（目录 + 1-6 + 边界） | PASS |
| T3 首行多来源合规 | `head -1 docs/dma/k3-cache-pma-address-translation.md` | `> 来源: https://github.com/riscv/riscv-iommu（... 2026-09-09）; https://github.com/riscv/riscv-isa-manual/blob/main/src/privileged.adoc（... 2026-09-09）; https://github.com/riscv/riscv-isa-manual/blob/main/src/unprivileged.adoc（... 2026-09-09）` | PASS |
| T3 相对链接解析 | `python3` 解析所有 `[text](path)` 链接 | 17 个 OK（含 5 个第三方源码路径 + 4 个 RISC-V spec URL + 4 个同主题文档 / 2 个 interrupts / 2 个 reference）；2 个 MISS（`../network/` + `../storage/`）为 R06 deferred 未来主题目录（与 index.md 主题职责表"待聚合"一致，非本 cycle 范围） | PASS |
| T3 证据强度标注 | `grep -E "（官方事实\|交叉验证\|推论\|未知项）"` | 28 处标注，覆盖 6 主段每段至少 1 处；未知项 3 处均含当前证据 / 解除条件 | PASS |
| source-coverage 行数 | `awk '/^\| https/{c++} END{print c}' docs/reference/source-coverage.md` | 68 | PASS |
| source-coverage 3 新行字段 | `grep -E "riscv-(iommu\|isa-manual)"` | 3 行全部登记，主题位置 dma / 观察日期 2026-09-09 / 访问状态 observed | PASS |
| G5 状态变更 | `grep "G5 \| " docs/reference/known-gaps.md` | `| G5 | 硬件事实 | partial | 2026-09-09 |` | PASS |
| G5 当前证据指向 T3 | `grep -A 2 "当前证据" docs/reference/known-gaps.md` | 含 `Iteration 001 / T3 新增 [`../dma/k3-cache-pma-address-translation.md`](../dma/k3-cache-pma-address-translation.md) §3-§6` | PASS |
| index.md URL 计数 | `grep "68 个唯一 URL" docs/index.md` | 一处命中 | PASS |
| index.md 缺口 partial 列表 | `grep "G3、G4、G5 为 .partial" docs/index.md` | 一处命中 | PASS |
| index.md docs/dma/ 状态行 | `grep "docs/dma/" docs/index.md \| grep "已聚合"` | 一处命中（含 Iteration 000 + Iteration 001 双标注 + G5 partial） | PASS |
| index.md 维护规则计数 | `grep "68 URL、10 gaps" docs/index.md` | 一处命中 | PASS |
| tasks.md 进度 | `grep -E "^- \[[ x]\]" tasks.md` | 5/5 全部 `[x]` | PASS |
| OpenSpec 集成 | `openspec validate establish-k3-com260-dma-memory-ownership-baseline` | `Change 'establish-k3-com260-dma-memory-ownership-baseline' is valid`（退出码 0） | PASS |
| 完整 diff 复审 | `git diff docs/ openspec/changes/.../tasks.md` | 仅本 cycle 预期修改；不含身份型证据工程；不含 build/test 必需的新身份字段；不含 capture/audit/qualification 工具 | PASS |

**Persisted Evidence**

None required. Plan Context §B 明示 `Persisted Evidence: none`；本 cycle 全部验证为可低成本重跑的 grep / wc / python 解析 / openspec validate，输出可被命令直接复现，无需持久化保存。

**Experience Candidates**

None. 本 cycle 未出现：(1) 已端到端验证成功且可重复 / 高风险的操作路径（可作 Runbook 候选）；(2) 造成显著影响、需要异常恢复、难以复现或包含系统性诊断信息的故障（可作 Incident 候选）。本仓库 Runbook / Incident 维护由 `openspec-experience-recorder` 负责，本 Act 不创建。

**Remaining Issues**

None. Plan Context §A1-A6 验收契约全部覆盖；Iteration Map 中 Iteration 001 全部 3 个 task 完成；当前 Iteration 唯一 Cycle 完成；change 全部 5/5 tasks 完成，可由 `openspec-plan` 收尾。

**Commit or Diff Reference**

本 cycle 修改 4 个文件 + 新建 1 个文件（路径已列于 Changed Files and Symbols）；`git diff` 摘录可由用户执行 `git diff docs/ openspec/changes/establish-k3-com260-dma-memory-ownership-baseline/tasks.md` 获取；本 Act 不执行 commit（按 Plan Context §A2 "变更面" 仅含文件修改，不含 git 操作）。

## Plan Review

- Review Result: accepted

**Findings**

- Spec compliance: PASS（Plan Context §A1-A6 全部覆盖；A1 = T3 7 段落；A2 = T3 引用 3 URL + T3 行数 ≤ 200；A3 = T4 G5 4 字段；A4 = T5 DMA 入口 + 状态 + URL 计数 + 缺口计数 4 字段；A5 = source-coverage 表头 + 字段说明同步；A6 = 跨 4 对象 cache / PMA / PBMT / IOMMU + 1 段地址空间 + 边界段全部按证据强度标注）。
- Code quality: PASS（不引入身份型证据工程；不引入 capture/audit/qualification 工具；不引入 build/test 必需的新身份字段、握手、pin、freeze、manifest、hash 链或时间顺序检查；不引入 `run-id` / `peer` / `host` / `revision pin` / `source/index/worktree freeze` / `artifact manifest` / `TIME_ORDER` 时间证明 / `audit` / `qualification` 工具；不创建排除路径或二级验证；`others/` 路径作 evidence 引用，不复制源码字段；行号 / 字段名 / offset 全部可由读者直接打开对应文件复核）。
- Test witness: PASS（T3 文件创建前确认不存在（RED），创建后确认存在（GREEN）；T4 G5 状态变更前为 `open`、变更后为 `partial`；T5 index.md 计数前 62 / 10 / G3,G4 partial、变更后 68 / 10 / G3,G4,G5 partial；所有状态变化均按 RED → GREEN 顺序执行）。
- 完整 diff 复审: PASS（5 个文件 diff 全部为计划内修改；不含计划外修改；不含 Critical / Important 缺陷；所有 Minor 缺陷已记录或无）。
- Critical findings unresolved: 0。
- Important findings unresolved: 0。
- Minor findings unresolved: 0。

**Deviation Classification**

None. Act Response 实际改动与 Plan Context Task Contracts 一一对应；`tasks.md` 整块替换失败改为逐行替换属工具差异非计划偏差，已在 Act Response Deviations from Plan 记录。

**Acceptance Gaps**

A1-A6 全部覆盖（参见 Findings + Act Response Verification Evidence 表）。

**Convergence**

N/A（首次 Cycle 即完成全部 3 个 task + 4 个配套修改；同一 Acceptance gap 无连续两个 rework Cycle）。

**Evidence**

参见 Act Response Verification Evidence 表（14 项验证全部 PASS，含 1 项 OpenSpec 集成 + 13 项文档/计数/链接检查）。

**Follow-up Decision**

- Plan Context §A1-A6 全部覆盖；T3 / T4 / T5 全部 `[x]`；`openspec validate` 通过。
- 当前 change `establish-k3-com260-dma-memory-ownership-baseline` 全部 5/5 tasks 完成；Iteration Map 中 Iteration 000 + 001 全部完成。
- 下一步：用户调用 `openspec-docs-maintainer` 收尾（同步 SNAPSHOT / tasks / R12 / G5 partial / source-coverage 68 行计数到全局 M/D/K/R/I，登记 R12 到 references/spec.md）；随后由 `openspec-archivist` 归档。
- 注：Act 不调用 Maintainer / Archivist / Recorder；按 CLAUDE.md "Skill 完成不构成下一阶段授权"，本 Act 终止并等待用户审计。

**Iteration Plan Update**

None（Iteration 001 全部 task 完成；无新 Iteration 待展开；change 收尾由 maintainer 触发后由 `openspec-milestone-planner` 评估后续 MSxx 方向）。

**Next Cycle**

None（当前 Cycle 即终态；无后继 Cycle）。

**Next Iteration**

None（Iteration Map 中 Iteration 001 终止；后续 MSxx 由 milestone-planner 按需新增）。
