# Iteration 001 / Cycle 001: 内存属性事实与边界返工

## Plan Context

- Status: ready
- Iteration: 001-memory-attributes-finish
- Cycle: 001-rework
- Cycle Type: rework
- Parent cycle: `000-initial.md`

**Iteration Scope**

- Change tasks: T3-T5
- Depends on: Iteration 000 accepted
- Stable baseline: MS06 的 DMA ownership、cache/coherency、平台内存属性和地址转换边界可供 MS07-MS09 引用
- Verification boundary: 第二篇 DMA 正文准确区分机制、地址空间与证据等级；G5、入口、来源和全部本地链接一致
- Diagnostic boundary: IOMMU、PMA/PBMT、AP/RP hart 作用域、设备 cache/地址路径、未知项和导航
- Deferred tasks: None

**Cycle Scope**

- Trigger: rework-required
- Acceptance gaps: A1-A6；正文存在实质事实冲突、缺少契约要求的对象路径和地址层，两个本地链接失效，G5/index/task 状态建立在未满足的 T3 上
- Repair items: T3-R1、T3-R2、T3-R3、T4-R1、T5-R1
- Inherited scope: proposal R2-R5、design D3-D5、T3-T5、M01-M04、D01-D07，以及 Iteration 000 已接受的对象与 ownership 术语
- Excluded scope: 驱动实现、真板/QEMU 验证、默认 Kit DTS 认定、SNAPSHOT、全局 tasks、M/D/K/R/I 和 change 归档

**Objective**

修正 cache/PMA/PBMT/IOMMU 正文的事实和证据边界，补齐 GMAC/UFS/共享内存的 cache 与地址路径，消除失效链接并恢复未知项四字段；随后同步 G5、index 和 T3-T5 状态。

**Background**

父 Cycle 的 Act Response 为 `reported`，但 Act 同时改写了不可变 Plan Context，并越权把 Plan Review 写成 `accepted`。独立审计复现了多个阻断 Acceptance 的问题。父 Cycle 已出现终态文字，按冻结规则保留为历史现场；本 Cycle 取代其未经独立 Review 的接受结论。

**Current Baseline**

- `docs/dma/k3-cache-pma-address-translation.md` 已创建，79 行。
- coverage 表有 68 行且 URL 唯一；G5 和 index 已按 `partial`、68 URL、DMA 已聚合同步；T3-T5 已勾选。
- 父 Act 把两个不存在的目录链接列为 MISS，却判定链接检查 PASS。
- Persisted Evidence 为 `none`；当前问题均可从仓库文档、固定 revision 源码和直接验证重跑。

**Current-State Evidence**

- proposal 的 IOMMU 场景明确规定：找不到目标设备的节点、驱动、domain 或 map/unmap 时，不得推定 IOMMU 不存在、bypass 或 identity mapping。
- `k3-soc-overview.md` 记录官方资料在概述层声明 K3 支持 IOMMU 扩展；固定第三方 CoM260 IFX DTS 又含 `iommu-map` 与 `spacemit,k3-iommu`。正文 §6 却写“当前证据不含 RISC-V IOMMU”“无独立 IOMMU translation”“当前默认设计无 IOMMU”，属于证据不足时推定不存在。
- OpenSBI `spacemit_k3.c` 注释把 AMP-window PMA 修改限定为 X100 harts，`spacemit_k3_final_init()` 在这些 hart 上调用。RP `K3Rt24::init` 只证明 rcpu1 不能访问 custom cache/PMA CSR。正文 §3 声称 rcpu1 的所有 cache/PMA 设置必须经该 OpenSBI final-init，错误连接了两个执行域。
- 同一 OpenSBI 第三方注释称该 silicon 忽略 Svpbmt；正文一处据此称 PMA 是 K3 唯一机制，另一处又称 AP 端由 MMU/Svpbmt 决定，内部冲突，并把第三方作者记录提升为已证硬件事实。
- T3 契约要求分别记录 GMAC/UFS 与共享内存的 cache/地址路径，并区分 CPU VA、CPU PA、device address/IOVA。正文没有相应矩阵或设备 cache API 路径，反而用“无 IOMMU translation 介入”替代未知边界。
- 正文链接 `../network/` 与 `../storage/`，目标目录不存在；独立扫描输出两项 `MISS`。A5 要求全部本地链接可解析，未来目录不在豁免范围。
- 三处“未知项”没有分别具备当前证据、禁止推断、解除条件和影响范围。父 Act 只统计“当前证据/解除条件”，没有验证四字段。
- `docs/index.md` 的第二篇 DMA 摘要复制了多项技术结论，包括尚未接受或表述过强的 Svpbmt/IOMMU 结论，违反入口只承担导航、状态和计数的不变量。
- 父 Act 修改 Plan Context 的 Status 和 User Plan Approval，并自行填写 Plan Review 终态，违反 Plan/Act 写入边界；该偏差不由产品文档修复，但必须保留在审计链中。

**Relevant Code**

- `docs/dma/k3-cache-pma-address-translation.md`：T3-R1 至 T3-R3 的主要修复目标。
- `docs/dma/k3-dma-and-memory-ownership.md`：已接受的对象、completion 和证据层基线。
- `docs/reference/source-coverage.md`：3 个新增标准来源的精确 URL、职责和访问状态。
- `docs/reference/known-gaps.md`：G5 当前证据、禁止推断、解除条件、状态与汇总。
- `docs/index.md`：DMA 导航、状态和权威计数。
- `openspec/changes/establish-k3-com260-dma-memory-ownership-baseline/tasks.md`：T3-T5 状态。
- `others/Rt-Async-AMP/opensbi-k3/.../spacemit_k3.c`、`modules/chip-k3-rt24/src/lib.rs`、`modules/ov-shm/src/shm.rs`：X100 PMA、RP CSR 和 fence 边界。
- `others/Rt-Async-AMP/tgoskits/drivers/ax-driver/src/{net/k3_gmac,block/k3_ufs}` 与 CoM260 IFX DTS：设备 cache API、地址写入和 IOMMU/DTS 固定 revision 经验。

**Critical Path**

1. 先把 IOMMU、PMA/PBMT 与 AP/RP 执行域恢复为可证事实、第三方作者记录和未知边界。
2. 用机制矩阵分开 cache maintenance、ordering、PMA、PBMT、页表属性和 IOMMU；禁止互相替代。
3. 用对象/地址矩阵分别记录 GMAC、UFS、共享内存的 CPU VA/PA、device address/IOVA、cache API 和不可证字段。
4. 把每个未知项恢复为四字段，删除不存在目录的链接。
5. 核对新增标准 URL 的当前规范职责和访问状态；不能直接观察的 URL 不得标 `observed`，标准来源不得承担 K3-specific 事实。
6. 按修订正文重新决定 G5；压缩 index 为导航摘要；全部 GREEN 后保留或恢复 T3-T5 checkbox。

**Implementation Guidance**

标准只解释术语与通用机制，K3-specific 结论必须来自 K3 官方资料或标明为固定 revision 第三方经验。没有目标设备的节点、domain 或 map/unmap 路径时写“未知”，不能写“不存在”或“默认无”。OpenSBI 路径写 X100/AP 侧作用域；RP 侧只写 custom CSR 不可用与现有 fence 行为。

**Behavioral Change**

修复后，正文不再宣称 K3 默认无 IOMMU，不再让 AP OpenSBI 代替 RP 配置 PMA，也不再把第三方注释提升为已验证 silicon 行为；三类对象的 cache 与地址边界可检索，全部本地链接有效，G5/index/task 状态与正文一致。

**Change Surface**

| Repair | Requirement/Scenario | File/Symbol | Current Responsibility | Planned Change |
| --- | --- | --- | --- | --- |
| T3-R1 | R3-S1-S3；R4-S1-S3 | cache/PMA/PBMT/IOMMU 正文 §2-§6 | 机制和作用域 | 修正 IOMMU、X100/RP、PMA/PBMT 与证据等级 |
| T3-R2 | R2-S3-S4；R4-S2 | 同文件对象与地址段 | 地址空间和对象路径 | 补齐 GMAC/UFS/共享内存 cache/地址矩阵 |
| T3-R3 | R5-S1-S2 | 同文件首行、未知项和链接；coverage 新增 3 行 | 来源、未知项、导航 | 核对标准职责、补四字段、清除失效链接 |
| T4-R1 | R3-S3；R4-S3；R5-S2 | `known-gaps.md` G5/汇总 | partial gap | 按修订证据重写并重新判定状态 |
| T5-R1 | R5-S3 | `index.md` DMA 摘要；`tasks.md` T3-T5 | 导航、计数和状态 | 删除技术事实复制，全部 GREEN 后同步状态 |

**Task Contracts**

### T3-R1：机制事实与执行域修正

- Requirement/Scenario: R3-S1-S3；R4-S1-S3。
- Depends on: None。
- Targets: `docs/dma/k3-cache-pma-address-translation.md` §2-§6。
- Current behavior: 推定默认无 IOMMU；把 X100 OpenSBI PMA 路径写成 rcpu1 配置通道；PMA/PBMT 表述互相冲突并提升第三方注释证据等级。
- Required behavior: IOMMU presence/enablement/mapping 保持未知；X100/AP 与 RP 能力分域；PMA/PBMT、fence/cache 互不替代，第三方作者的 silicon 结论保持第三方证据等级。
- Required changes: 删除“不含/无/默认无 IOMMU”断言；删除 rcpu1 必须经 OpenSBI final-init 和 AP 由 Svpbmt 决定的冲突句；逐项给出主体、阶段、范围和验证边界。
- Preserve: 标准术语、固定 revisions、G5/G7、无真板结论。
- Forbidden: 不由目录搜索证明硬件不存在，不把 IFX DTS 当默认 Kit，不把第三方注释标成官方事实或本项目验证结果。
- Test witness: `rg -n '当前证据不含 RISC-V IOMMU|无独立 IOMMU translation|当前默认设计无 IOMMU|所有 cache / PMA 设置必须经 OpenSBI|AP 端 MMU/Svpbmt 决定'` 当前命中正文。
- GREEN condition: 禁止模式零命中；IOMMU、PMA/PBMT 和 X100/RP 作用域与 proposal、概述和固定源码一致。
- Verification: requirement/spec 对照、固定源码符号对照、冲突模式扫描、证据等级人工审查。
- Stop when: 新官方证据改变 IOMMU presence 或 PMA/PBMT 硬件模型。

### T3-R2：设备 cache 与地址空间闭合

- Requirement/Scenario: R2-S3-S4；R4-S2。
- Depends on: T3-R1。
- Targets: 同正文的机制矩阵、地址矩阵和对象路径。
- Current behavior: 未独立区分 CPU VA/PA、device address/IOVA，也未记录 GMAC/UFS cache API 路径；现有笼统句以“无 IOMMU”替代地址边界。
- Required behavior: 三类对象分别记录 CPU 侧地址、写入设备/共享对象的地址、cache prepare/complete 或 fence 路径，以及 IOVA/映射未知项。
- Required changes: 增加紧凑矩阵；GMAC/UFS 只写固定源码可见的 clean/invalidate、prepare/complete 和 descriptor 地址行为；共享内存区分 AP 主域地址与 RP local alias。
- Preserve: ownership 正文的 completion token 和对象边界，不复制其完整状态机。
- Forbidden: 不推定 identity mapping、bypass、地址数值相等或全局 coherency；不把共享内存规则外推设备 DMA。
- Test witness: 当前正文没有 GMAC/UFS 的 `prepare_for_device`、`complete_for_cpu`、descriptor clean/invalidate 路径，也没有四类地址关系矩阵。
- GREEN condition: 三类对象均覆盖 cache 与地址边界；CPU VA/PA、device address/IOVA、AP/RP alias 独立可检索，未知映射不被结论替代。
- Verification: 对象/地址矩阵人工审查，GMAC/UFS/ov-shm 固定源码对照，禁止推断扫描。
- Stop when: 无法从固定源码区分 device-visible 地址与 CPU 地址，或官方模型与现有设计冲突。

### T3-R3：来源、未知项与链接修正

- Requirement/Scenario: R5-S1-S2；A1、A3、A5。
- Depends on: T3-R1、T3-R2。
- Targets: 正文首行、所有未知项和相对链接；`docs/reference/source-coverage.md` 的 3 个新增标准来源。
- Current behavior: 两个不存在目录的链接被错误判 PASS；未知项不具备各自四字段；标准 URL/章节职责和 `observed` 状态未形成可复现的直接观察证据。
- Required behavior: 全部本地链接目标存在；每个未知项各含当前证据、禁止推断、解除条件和影响范围；标准 URL 只承担实际可确认的通用职责。
- Required changes: 将未来主题改为无链接文本；补齐或合并未知项；核对 RISC-V IOMMU 当前官方入口及 ISA manual 的 FENCE/Svpbmt 所属卷，必要时精准更正 URL、职责、计数和首行。
- Preserve: URL 唯一键和非本 Cycle 来源行；不做批量 refresh。
- Forbidden: 不允许 MISS；不可直接取得的精确页面不标本轮 `observed`；不让标准来源支撑 K3-specific 结论。
- Test witness: 独立链接扫描输出 `MISS ../network/` 和 `MISS ../storage/`；当前网络检索显示 RISC-V IOMMU 规范入口位于 `riscv-non-isa/riscv-iommu`，现有精确入口需复核。
- GREEN condition: 本地链接零 MISS；未知项四字段完整；首行 URL 与 coverage 一对一且职责准确，总行数等于唯一 URL 数。
- Verification: 链接目标扫描、未知项逐项审查、URL 表计数/唯一性/首行反向匹配、可访问或不可访问结果记录。
- Stop when: 来源修订改变 proposal/design 或要求新增权威来源体系。

### T4-R1：G5 与修订正文一致

- Requirement/Scenario: R3-S3；R4-S3；R5-S2。
- Depends on: T3-R1 至 T3-R3。
- Targets: `docs/reference/known-gaps.md` G5、汇总和对应关系。
- Current behavior: G5 把“无 RISC-V IOMMU”和 Svpbmt silicon 结论写作已部分解除证据，继承正文的过度推断。
- Required behavior: G5 只登记实际缩小的所有权/软件路径和仍未知硬件事实；状态由修订后的证据决定。
- Required changes: 删除不存在 IOMMU 的结论；把第三方 PMA/PBMT、fence 和 DTS 记录降到相应证据层；重新判断 `open` 或 `partial` 并同步日期、汇总和对应关系。
- Preserve: G1-G4、G6-G10，尤其 G7 默认 Kit DTS 边界。
- Forbidden: 不凭第三方代码关闭 G5，不重复建 gap，不删除历史。
- Test witness: 当前 G5 明列“无 RISC-V IOMMU”并以第三方源码作为 `partial` 的主要解除依据。
- GREEN condition: G5 与两篇 DMA 正文一致，四字段和状态决定可追溯，无重复或无证据关闭。
- Verification: G5/汇总/对应关系对照、正文反向链接、语义审查、scoped diff。
- Stop when: 修订证据不足以判断状态或需要改变 milestone 范围。

### T5-R1：入口与任务状态闭合

- Requirement/Scenario: R5-S3；A5-A6。
- Depends on: T3-R1 至 T4-R1。
- Targets: `docs/index.md` DMA 段和主题职责；change `tasks.md` T3-T5。
- Current behavior: index 复制尚未接受且部分错误的 PMA/PBMT/IOMMU 技术结论；T3-T5 在 Acceptance 未满足时已勾选。
- Required behavior: index 只保留两篇文档的主题级导航、聚合状态和权威计数；T3-T5 只在全部返工 GREEN 后为完成。
- Required changes: 压缩第二篇摘要，删除机制数值和“无 IOMMU”等事实复制；同步实际 URL/gap 计数；若返工未完成则恢复 T3-T5 未完成状态，全部通过后保持 `[x]`。
- Preserve: 其他主题职责、链接和状态，不同步 SNAPSHOT/global tasks。
- Forbidden: 不把 index 当技术事实载体，不以 checkbox 替代验证。
- Test witness: 当前 index 的第二篇摘要列出 L1/L2、PMA entries、Svpbmt no-op、IOMMU 不含和替代机制等细节。
- GREEN condition: index 只承担导航/状态/计数；全部链接和计数一致；T3-T5 状态与最终验证一致。
- Verification: scoped diff、链接解析、coverage/gap 计数、checkbox 精确计数、`openspec list`。
- Stop when: 任一上游 repair item 未 GREEN。

**Invariants**

- 父 Cycle 的 Plan Context、Act Response 和越权终态不覆写；本 Cycle 形成后继独立审计链。
- 官方资料、官方 GitHub、第三方固定 revision、推论和未知项分层不变。
- `others/` 只读；不实现代码，不生成 Evidence 占位或身份型验证机制。
- IOMMU 节点/驱动/domain/map 路径不足时保持未知；G7 不关闭。

**Non-goals**

- 不修改驱动、OpenSBI、DTS 或第三方源码。
- 不证明真板 coherency、PMA/PBMT 或 IOMMU 行为。
- 不同步项目状态或归档 change。

**Acceptance**

- A1 / T3-R1：IOMMU、PMA/PBMT、fence/cache 和 X100/RP 作用域与 requirement、官方概述及固定源码一致，无互相替代或证据升级。
- A2 / T3-R2：GMAC、UFS、共享内存的 cache 与地址路径完整；CPU VA/PA、device address/IOVA、AP/RP alias 分层。
- A3 / T3-R3：每个未知项具备四字段；全部本地链接可解析；标准来源 URL、章节职责、访问状态和计数准确。
- A4 / T4-R1：G5 与正文一致，状态有证据且 G7 边界保留。
- A5 / T5-R1：index 只承担导航、状态和计数；T3-T5 checkbox 与实际结果一致。
- A6：正文少于 450 行，full/scoped diff Review、diff check 和 OpenSpec strict validation 通过。

**Verification**

- 事实：对照 proposal/spec、SoC 概述、OpenSBI X100 PMA、RP init、ov-shm fence、GMAC/UFS cache API 和固定 DTS。
- 禁止模式：IOMMU 不存在/default no-IOMMU、rcpu1 经 AP OpenSBI 配置、fence 等同 cache clean、Svpbmt 与 PMA 冲突措辞零命中。
- 结构：机制、对象和地址矩阵完整；未知项逐项四字段。
- 来源与链接：首行 URL 在 coverage 恰好一行，表行数等于唯一数；全部本地 Markdown 目标存在。
- 一致性：G5 状态/日期/汇总、index URL/gap 摘要、T3-T5 checkbox 逐项对照。
- Gate 4/5：`wc -l`、`git diff --check`、`git diff --cached --check`、完整 diff Review和 OpenSpec strict validation。

**Gate 2 Readiness**

| Dimension | Status | Evidence |
| --- | --- | --- |
| Investigation | PASS | 已独立读取正文、父 Act、proposal/spec、SoC 概述、G5/index 和固定 revision 源码并复现两项 MISS |
| Design | PASS | 只恢复既有 R2-R5/D3-D5，不改变目标、范围或验收边界 |
| Iteration Plan | PASS | 仍为 Iteration 001 的 T3-T5；返工不修改 Map |
| Cycle Scope | PASS | 五个 repair item 只关闭既有 A1-A6 gap |
| Task Contracts | PASS | 每项含目标、来源、见证、GREEN、验证和停止条件 |
| Traceability | PASS | repair item 映射原 task、requirement 和 Acceptance |
| Verification | PASS | 直接检查事实、结构、链接、状态和计数，无身份型证据工程 |
| User Plan Approval | PASS | 用户原话：`允许你开始实施并审计接受`；明确批准本 rework，并授权实施后串联独立 Plan Review |

**Persisted Evidence**

- Mode: none

Act Response 足以记录决定性输出和退出码；全部证据可从仓库内容低成本重跑。

**Risks and Notes**

- GitHub raw host 在本地环境仍可能不可达；访问失败只形成边界，不自动改变长期来源状态。
- 当前官方 RISC-V IOMMU 仓库检索结果指向 `riscv-non-isa/riscv-iommu`；Act 必须核对现有 URL 是否重定向及实际可读职责，不能只按仓库名称推断。

## Act Response

- Status: reported

**Implemented**

- T3-R1：删除 K3 默认无 IOMMU、rcpu1 经 AP OpenSBI 配置 PMA、AP 由 Svpbmt 决定等断言；机制矩阵分开 cache maintenance、FENCE/barrier、PMA、PBMT、CPU page table 和 IOMMU，固定 OpenSBI 结论限定为 X100 hart 的第三方修改。
- T3-R2：增加 GMAC、UFS、AP↔RP shared SRAM 的对象/地址矩阵，分别记录 device-visible 地址、cache/ordering 路径和 IOVA/domain 未知边界；增加 CPU VA/PA、device address/IOVA 与 AP/RP alias 关系图。
- T3-R3：将未知项整理为 U1-U3，每项含当前证据、禁止推断、解除条件和影响范围；删除 network/storage 未来目录链接；把标准来源更正为 RISC-V 官方 ratified 文档页与 `riscv-non-isa/riscv-iommu`。
- T4-R1：重写 G5 的部分解除依据，删除“无 RISC-V IOMMU”和 Svpbmt 已证无效结论；保留 `partial`，因为对象/软件路径和机制边界已闭合，而硬件 coherency、PMA/PBMT 效果及默认 Kit IOMMU 仍未知。
- T5-R1：将 index 的第二篇 DMA 摘要压缩为主题级导航；来源计数同步为 69；T3-T5 在全部 GREEN 后保持完成。

**Changed Files and Symbols**

- `docs/dma/k3-cache-pma-address-translation.md`：重写机制、对象/地址、PMA 作用域、IOMMU 边界、未知项和导航。
- `docs/reference/source-coverage.md`：以 4 个可直接核对的标准入口替换 3 个错误或职责不准的 GitHub 路径，计数 68 → 69。
- `docs/reference/known-gaps.md`：修正 G5 当前证据、禁止推断、状态说明、影响范围和 coverage 对应。
- `docs/index.md`：同步 69 URL，压缩 DMA 技术摘要。
- 当前 Cycle：记录 Gate 2 授权和本 Act Response；未填写 Plan Review。

**Deviations from Plan**

- 原计划预期复核 3 个标准 URL；实际发现需要用 4 个职责更精确的官方入口分别承载 PMA、Svpbmt、FENCE 与 IOMMU，因此 coverage 唯一数由 68 调整为 69。该变化不改变 requirement 或 Acceptance。

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

Spec review 先确认 A1-A6 与五个 repair item 均满足；code-quality review 随后确认 index 不再复制技术细节、无失效链接、无计划外代码或身份型证据机制。Critical/Important finding 均已修复。

**Verification Evidence**

| 验证项 | 命令或操作 | 输出摘录 | 结论 |
| --- | --- | --- | --- |
| RED / 事实边界 | 禁止模式 `rg` | 原正文命中 rcpu1/OpenSBI、AP/Svpbmt、无/default IOMMU 5 处 | PASS：错误基线已见证 |
| RED / 链接 | 解析正文相对链接 | `MISS ../network/`、`MISS ../storage/` | PASS：失效链接已见证 |
| 事实 GREEN | 对照 proposal/spec 与固定 OpenSBI/RP/GMAC/UFS/ov-shm 源码 | 禁止模式 0；对象矩阵含 GMAC/UFS/shared SRAM | PASS |
| 地址分层 | 检查正文矩阵与关系图 | CPU VA/PA、device address/IOVA、AP/RP alias 均独立出现 | PASS |
| 未知项 | 扫描 U1-U3 四字段 | 3 项各有当前证据、禁止推断、解除条件、影响范围 | PASS |
| 本地链接 | 解析正文全部非 HTTP Markdown target | `broken_links=0` | PASS |
| 来源映射 | 首行 URL 反向匹配 coverage | 4/4 URL 各命中 1 行 | PASS |
| coverage | 表行与唯一 URL 计数 | `table_rows=69`；`unique_urls=69` | PASS |
| G5/index | 状态、摘要和计数对照 | G5 `partial`；index `69 URL`、`10 gaps` | PASS |
| task 状态 | `openspec list` | 5/5 tasks，Complete | PASS |
| 行数 | `wc -l docs/dma/k3-cache-pma-address-translation.md` | `103` | PASS：小于 450 |
| diff | `git diff HEAD --check`; `git diff --cached --check` | 无输出；exit 0 | PASS |
| OpenSpec | `openspec validate establish-k3-com260-dma-memory-ownership-baseline --strict` | `Change ... is valid` | PASS |

**Persisted Evidence**

None required.

**Experience Candidates**

None.

**Remaining Issues**

None within this Cycle。K3 硬件 coherency、PMA/PBMT 真板效果和默认 Kit IOMMU 按 U1-U3/G5/G7 保持未知。

**Commit or Diff Reference**

工作区包含本 change 的完整未提交 diff；未创建 commit。

## Plan Review

- Review Result: accepted

**Findings**

None blocking。独立复审确认 T3-R1、T3-R2、T3-R3、T4-R1 和 T5-R1 均满足契约。标准入口由 3 个不准确路径调整为 4 个职责明确的官方入口，使 coverage 从 68 增至 69；这是结果计数变化，不改变 requirement 或 Acceptance。

**Deviation Classification**

ACT-DEVIATION resolved.

**Acceptance Gaps**

None.

**Convergence**

reduced：父 Cycle 审计发现的 IOMMU 推定、AP/RP PMA 作用域、对象/地址路径、未知项、链接和入口摘要缺口全部关闭。

**Evidence**

- 独立禁止模式扫描：默认/不存在 IOMMU、rcpu1 经 AP OpenSBI 设置 PMA、AP 由 Svpbmt 决定和 K3 Svpbmt no-op 等过强措辞均为 0。
- 独立源码对照：X100 PMA 修改、RP custom CSR 禁止、GMAC descriptor clean/invalidate、UFS prepare/complete + DMA barrier 与正文一致。
- 三份受影响文档的本地 Markdown target 全部存在；第二篇 DMA 正文 `broken_links=0`。
- U1-U3 各含当前证据、禁止推断、解除条件和影响范围。
- 正文首行 4 个 URL 在 coverage 各命中一次；`table_rows=69`，`unique_urls=69`。
- G5 为 `partial` 且保留 hardware unknown；index 为 69 URL、10 gaps，DMA 摘要只承担主题导航。
- `openspec list` 为 5/5 tasks；正文 103 行。
- `git diff HEAD --check`、`git diff --cached --check` 和 OpenSpec strict validation 均通过。

**Follow-up Decision**

接受 Iteration 001 和整个 change 的实现结果。A1-A6 已满足，无须继续创建 Cycle 或 Iteration；按阶段边界等待用户调用 `openspec-docs-maintainer` 收尾。

**Iteration Plan Update**

None.

**Next Cycle**

None.

**Next Iteration**

None.
