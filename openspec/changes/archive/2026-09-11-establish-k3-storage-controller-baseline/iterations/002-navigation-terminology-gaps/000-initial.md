# Iteration 002 / Cycle 000: 导航、术语与缺口收敛

## Plan Context

- Status: ready
- Iteration: 002-navigation-terminology-gaps
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: None

**Iteration Scope**

- Change tasks: T4, T5, T6
- Depends on: Iteration 000、Iteration 001（均 `accepted`）
- Stable baseline: 两篇存储正文能从总索引进入；存储未知项映射到 G5、G7 和独立存储缺口；术语、URL 数、缺口数与主题状态一致。
- Verification boundary: `known-gaps.md` 的正文、汇总和来源对应一致；术语无重复且计数准确；index 的两篇入口、storage 职责、70 URL、术语数和缺口数与权威文件一致。
- Diagnostic boundary: `known-gaps.md`、`terminology.md`、`index.md` 三个收尾表面。
- Deferred tasks: None

**Cycle Scope**

- Trigger: initial
- Acceptance gaps: None
- Repair items: None
- Inherited scope: R6/S1-S2、R7/S1-S2、D3、D6、T4-T6、M01-M04，以及已接受的两篇 storage 正文和 70 URL 来源基线。
- Excluded scope: 修改两篇 storage 正文或 source-coverage、增加来源 URL、裁决默认 DTS/UFS 容量、修改驱动、同步 SNAPSHOT/tasks、维护 M/D/K/R/I 或归档 change。

**Objective**

把已接受的存储知识正文接入缺口、术语和总索引，使读者能从一个入口定位主题、证据边界与未决问题，并使三个汇总数字可由权威文件直接复核。

**Background**

Iteration 000 已建立 `k3-qspi-spi-sdhci.md` 和当前存储来源责任；Iteration 001 已建立 `k3-ufs.md`。两篇正文各有五项四字段未知项，但 `known-gaps.md` 仍止于 G11，术语表仍有 25 项，index 仍把 storage 标为待聚合。来源覆盖表已有 70 个唯一 URL且无重复，本 Cycle 不新增来源。

**Current Baseline**

- `docs/reference/known-gaps.md` 有 G1-G11；G5 为 `partial`，G7 为 `open`，尚无存储专属缺口。
- `docs/reference/terminology.md` 有 25 个术语；SPI、QSPI、SDHCI、eMMC、UFS、M-PHY、UniPro、UTP、UPIU、UTRD、UTMRD、UCD、PRDT、SCSI、LUN 均无独立行。
- `docs/index.md` 报告 70 URL、25 术语、G1-G11；没有 storage 正文入口，主题职责仍为“待聚合；R06 状态 deferred”。
- `docs/reference/source-coverage.md` 有 70 个唯一 URL、0 重复；T1 已把四个存储官网入口设为当前职责，不需要本轮改写。
- change tasks 中 T1-T3 已完成，T4-T6 未完成。

**Current-State Evidence**

- `k3-qspi-spi-sdhci.md` U1-U3 主要受 G7 的目标 DTS/板级映射限制，U1/U4/U5 的 DMA/cache 和完成回收部分与 G5 相交；其 QSPI/SPI/SDHCI 专属 programmer/runtime/recovery 缺口不由 G5 或 G7完整承载。
- `k3-ufs.md` U1/U2/U4/U5 分别涉及专属 binding、IRQ/轮询完成、同步 block 扩展性和 fatal 重建；U3 是板级容量冲突。这些内容需要一个存储专属 G 条目，并明确 G5/G7 的交叉责任，避免复制两篇正文。
- 下一可用编号为 G12。单一 G12 可索引两篇正文的十项未知项；分别创建多个 G 会复制同一 controller/runtime/recovery 解除条件并使汇总过细。
- 术语表当前 25 行。新增 15 个正文核心术语后应为 40 行：SPI、QSPI、SDHCI、eMMC、UFS、M-PHY、UniPro、UTP、UPIU、UTRD、UTMRD、UCD、PRDT、SCSI、LUN。
- index 的来源数保持 70；新增 G12 后缺口数为 12，术语新增 15 行后为 40。

**Relevant Code**

- `docs/reference/known-gaps.md`：T4 的缺口正文、状态汇总和 source-coverage 对应关系。
- `docs/reference/terminology.md`：T5 的存储术语表。
- `docs/index.md`：T6 的主题入口、主题职责和三个汇总数字。
- `docs/storage/k3-qspi-spi-sdhci.md`、`docs/storage/k3-ufs.md`：已接受输入，只读。
- `docs/reference/source-coverage.md`：70 URL 权威计数，只读。

**Critical Path**

两篇正文未知项 → G5/G7 交叉映射 + G12 存储专属闭包 → 15 个术语登记 → index 接入两篇正文并同步 70/40/12 计数。T6 依赖 T4/T5 的最终计数，必须最后执行。

**Implementation Guidance**

先写 G12，再同步缺口汇总和来源对应；随后按现有术语表格式加入 15 行；最后更新 index。G12 只保存当前证据、禁止推断、解除条件、影响主题和交叉责任，不复制十项 U 的正文。

**Behavioral Change**

当前 storage 正文存在但无法从总索引进入，术语和全局缺口也未承载其入口。完成后，导航和汇总与两篇已接受正文一致；技术事实、来源集合和运行行为不变化。

**Change Surface**

| Task | Requirement/Scenario | File/Symbol | Current Responsibility | Planned Change |
| --- | --- | --- | --- | --- |
| T4 | R6/S1-S2, R7/S1 | `docs/reference/known-gaps.md` | G1-G11 及汇总 | 新增 G12，映射 storage U 项与 G5/G7，更新汇总和来源对应 |
| T5 | R7/S1 | `docs/reference/terminology.md` | 25 个基础术语 | 增加 15 个存储术语并保持主写法唯一 |
| T6 | R7/S1-S2 | `docs/index.md` | 总入口和主题职责 | 接入两篇正文，更新 storage 状态及 70/40/12 汇总 |

**Task Contracts**

### T4: 存储未知项具有单一全局入口

- Requirement/Scenario: R6/S1-S2, R7/S1
- Depends on: T1-T3
- Targets: `docs/reference/known-gaps.md`
- Current behavior: 文档止于 G11，两篇 storage 正文的十项 U 未映射到全局缺口。
- Required behavior: 新增 G12“存储控制器板级映射、运行路径与恢复闭包”，状态 `open`、最近核对日期 `2026-09-11`；用指针覆盖两篇正文 U1-U5，明确 G5 负责 DMA/cache/coherency、G7 负责默认目标 DTS，G12 负责设备专属 binding/programmer reference、板级介质差异、消费层、完成模型、性能与错误恢复闭包。
- Required changes: G12 包含当前证据、禁止推断、解除条件、影响主题和状态变更记录；在状态汇总增加 G12；在 source-coverage 对应区增加 G12 与两篇正文的对应说明。G5/G7 状态不变，不裁决 UFS 容量或默认 DTS。
- Preserve: G1-G11 正文、编号、状态和日期；两篇 storage 正文；source-coverage。
- Forbidden: 不复制十项 U 全文；不把第三方静态或运行行为升级为官方保证；不新增 G13；不修改既有缺口状态。
- Test witness: 修改前 `rg '^## G12' docs/reference/known-gaps.md` 无输出且退出 1；汇总无 G12。
- GREEN condition: G12 五类字段完整；两篇正文 U1-U5、G5、G7 的职责可检索；汇总恰有 G1-G12，G12 为 `open / 2026-09-11`。
- Verification: 检查 G12 标题和字段、两篇相对链接、U1-U5/G5/G7 责任、汇总编号唯一与计数 12。
- Stop when: 新缺口不能由一个内聚 G12 承载、必须改变 G5/G7 定义或状态，或需要裁决容量/默认 DTS。

### T5: 存储正文核心术语具有唯一主写法

- Requirement/Scenario: R7/S1
- Depends on: T2, T3
- Targets: `docs/reference/terminology.md`
- Current behavior: 术语表有 25 行，缺少正文使用的存储协议和 descriptor 主写法。
- Required behavior: 新增且仅新增 15 行：SPI、QSPI、SDHCI、eMMC、UFS、M-PHY、UniPro、UTP、UPIU、UTRD、UTMRD、UCD、PRDT、SCSI、LUN；每行给出英文原词/缩写、别名和设备适用范围，M-PHY 主写法吸收 `MPHY` 别名。
- Required changes: 按协议层次相邻排列；区分 SPI/QSPI、SDHCI 与介质、UTP/UPIU/descriptor；最终术语行数为 40且主写法无重复。
- Preserve: 既有 25 行和标题使用规则。
- Forbidden: 不新增正文未使用的泛化术语；不把 UFS descriptor 外推到 QSPI/SPI/SDHCI；不修改 storage 正文以迎合术语表。
- Test witness: 修改前逐项检索 15 个主写法均无独立术语行，术语行计数为 25。
- GREEN condition: 15 行各出现一次，主写法唯一，术语行计数为 40，说明能区分控制器、介质、协议层和 descriptor。
- Verification: 用表格行检索、主写法重复检查和总行计数直接验证。
- Stop when: 现有主写法与新增项发生实质冲突，或必须修改主题正文才能统一含义。

### T6: 总索引与权威计数一致

- Requirement/Scenario: R7/S1-S2
- Depends on: T4, T5
- Targets: `docs/index.md`
- Current behavior: index 无 storage 正文入口，storage 状态为待聚合，汇总为 70 URL、25 术语、11 gaps。
- Required behavior: 在“已聚合主题正文”增加 `docs/storage/` 及两篇正文入口；把主题职责状态更新为两篇已聚合并标注 G5 partial、G7/G12 open；参考文档汇总更新为 70 URL、40 术语、G1-G12 共 12 类且 G3/G4/G5 partial。
- Required changes: 保持来源数 70；链接目标存在；storage 描述只作导航，不复制技术正文。
- Preserve: 其他主题入口、职责和计数；source-coverage 的 70 URL 权威值。
- Forbidden: 不修改其它主题状态；不把 R06 carrier 整体改为 active；不声明 MS09 或 change 已收尾。
- Test witness: 修改前 index 不含两篇 storage 链接，含 `25 个基础术语`、`G1-G11` 和 `待聚合；R06 状态 deferred`。
- GREEN condition: 两篇链接存在且可解析；70/40/12 与权威文件一致；storage 状态准确且未宣称 change 收尾。
- Verification: 检索入口、职责和计数，解析相对链接，并与 source-coverage/terminology/known-gaps 计算值比较。
- Stop when: T4/T5 最终计数不是 12/40，或 index 更新需要改写其他主题状态。

**Invariants**

- 70 个唯一 URL保持不变；本 Cycle 不写 source-coverage。
- G5/G7 的正文和状态不变；G12 不复制两篇正文的十项 U。
- 术语表只定义主写法和适用范围，不承载技术结论。
- index 只提供入口、职责和计数，不复制主题事实。
- M01-M04、D1-D6 和两篇已接受 storage 正文保持不变。

**Non-goals**

- 不增加来源、刷新外部页面或重新调查驱动。
- 不裁决 CoM260 默认 DTS、UFS 128/256 GB、IRQ 可用性、DMA coherency、性能或安全恢复。
- 不更新 SNAPSHOT、全局 milestone tasks、M/D/K/R/I 或行为规格，不归档 change。

**Acceptance**

- A1 / R6-R7 / T4：G12 以五类字段索引两篇正文 U1-U5，并与 G5/G7 分工；G1-G11 不变，汇总为 12。
- A2 / R7 / T5：15 个指定术语各有唯一主写法和适用范围，既有 25 行不变，总数为 40。
- A3 / R7 / T6：index 可进入两篇 storage 正文，storage 职责状态与 G5/G7/G12 一致。
- A4 / R6-R7 / T4-T6：source-coverage 保持 70 个唯一 URL、0 重复；index 的 70/40/12 与三个权威文件一致。
- A5 / R6-R7 / T4-T6：不裁决默认 DTS、容量、IRQ、性能或恢复完整性，不修改两篇正文、source-coverage 或其他主题状态。

**Verification**

- T4 RED/GREEN：G12 缺失 → G12 五类字段、链接、G5/G7 分工和 12 行汇总存在。
- T5 RED/GREEN：15 个术语行缺失且总数 25 → 各一行且总数 40、无重复。
- T6 RED/GREEN：storage 入口缺失且状态/计数陈旧 → 两篇链接有效，70/40/12 一致。
- 边界：`git diff --check`；`openspec validate establish-k3-storage-controller-baseline --strict`。
- 不运行全量 `openspec validate --all --strict`；两个既有无关 spec 失败不属于本 Cycle。

**Gate 2 Readiness**

| Dimension | Status | Evidence |
| --- | --- | --- |
| Investigation | PASS | 已读取两篇 storage U1-U5、known-gaps G1-G11/汇总、25 行术语、index storage 状态和 70 URL 唯一性 |
| Design | PASS | G5/G7/G12 责任、15 个术语和 70/40/12 目标值已确定，无契约语义留给 Act |
| Iteration Plan | PASS | T4-T6 共同形成导航收敛；T6 依赖 T4/T5，拆开会留下暂时错误计数，合并前序正文会混合故障域 |
| Cycle Scope | PASS | 只改 gaps、terminology、index；正文、coverage 和全局状态明确排除 |
| Task Contracts | PASS | 三个任务均含目标、当前/目标行为、保持/禁止、见证、GREEN、验证和停止条件 |
| Traceability | PASS | R6/R7 → D3/D6 → T4-T6 → 三个文件 → A1-A5，无 Missing 或 Simplified |
| Verification | PASS | 直接检查条目、链接、计数、重复和严格校验；无身份型证据工程 |
| User Plan Approval | PASS | 用户已批准包含 Iteration 002/T4-T6 的完整计划，并于 2026-09-11 授权在本轮审计无问题时接受和推进 |

**Persisted Evidence**

- Mode: none

三份 Markdown、链接和计数均可低成本重跑；Act Response 足以承载命令、决定性输出和退出码，不创建 Evidence 目录。

- Budget: 本 Cycle 最多 5 个文件（含 README），整个 change 最多 20 个 Evidence 文件；单个文本文件最多 500 行且不超过 256 KiB；本计划不使用该预算。

**Risks and Notes**

- G12 是索引层缺口，不应复制十项 U；若内容开始重复，应保留链接和责任摘要。
- R06 carrier 仍可保持 `deferred`，同时四个存储 URL 行已承担 current/active 责任；index 必须避免把两者写成同一状态。
- 术语 `M-PHY` 为主写法，`MPHY` 仅作别名；descriptor 缩写只适用于 UFS。

## Act Response

- Status: reported

**Implemented**

本 Cycle 包含两轮：初次实施覆盖 T4-T6，Plan Review 后在当前 Cycle 内对 Plan Review 列出的 4 个 Blocking 项做有限修复。Plan Invariant 与 Non-goals 全部满足；G12 责任、15 个主写法、40 数据行、12 G 条目、70 URL 五项基线不变。

- T4（初次）：在 `docs/reference/known-gaps.md` 新增 `## G12. 存储控制器板级映射、运行路径与恢复闭包` 六字段区块、汇总表追加 `| G12 | 硬件事实 | open | 2026-09-11 |`、source-coverage 对应表追加 G12 与 G5/G7 互斥职责说明；G1-G11 编号、状态、日期、字段正文均未触动。
- T5（初次）：在 `docs/reference/terminology.md` `## 基础术语` 表中按协议层次相邻顺序新增 15 行；既有 25 行未改；主写法无重复；数据行 25 → 40。
- T6（初次）：在 `docs/index.md` `## 已聚合主题正文` 新增 `docs/storage/` 分组及两条相对链接；`## 主题职责` 表 storage 行状态改为 `已聚合(Iteration 000)：k3-qspi-spi-sdhci；已聚合(Iteration 001)：k3-ufs；G5 partial，G7/G12 open`；主题集合说明由 6 类扩展为 7 类、`其余 4 类` 改为 `其余 3 类`；`## 参考文档` 术语表 25 → 40、已知缺口 G1-G11 → G1-G12；`## 维护规则` 状态/计数 `70 URL、25 术语、11 gaps` → `70 URL、40 术语、12 gaps`。
- 修复 R1（Plan Review Blocking / A2 — UFS 术语章节定位与语义）：`docs/reference/terminology.md` 10 个 UFS 术语行按 `k3-ufs.md` 实际章节校正。M-PHY / UniPro → §5；UTP / UPIU / UTRD / UTMRD / UCD / PRDT → §6；SCSI / LUN → §8。UTRD 描述由"command UPIU + PRDT + response UPIU"改为"command type / data direction / interrupt bit、OCS、UCD base、response UPIU offset+length、PRDT offset+length；task tag 在 command UPIU byte 3"；UTMRD 描述由"QUERY / ABORT TASK 描述符"改为"只分配并编程；QUERY / NOP OUT / COMMAND / RESPONSE 经保留 transfer slot 的 UTRD/UCD 提交"；UCD 描述由"PRDT 列表 + UPIU 头"改为"command UPIU 与 response UPIU 各 512-byte 对齐区域并引用 PRDT"；PRDT 描述移除"单 entry DBC 20 bit / 最大 1 MiB"第三方数值并改为"由 UCD 引用的 data buffer 物理区表 + `prepare_slot` + `prepare_for_device`"；UPIU 描述移除"OCS/response 检查"并明确"OCS 在 UTRD、不在 UPIU response"；SCSI 列表移除"NOP/QUERY"并明确"NOP OUT 不属 SCSI、QUERY 走 UTP/UPIU 容器"；LUN 行章节由 §6 / U3 改为 §8 并补充 `register_sync_block` 路径。SPI / QSPI / SDHCI / eMMC / UFS 五行非 UFS 主题术语按 Plan Review 结论未改。
- 修复 R2（Plan Review Blocking / A1,A5 — G12 UFS permalink）：`docs/reference/known-gaps.md:174` 中 [`k3_ufs/transfer.rs`](https://github.com/rt-async-amp/tgoskits) 替换为与 `k3-ufs.md:68,100` 一致的 [`k3_ufs/transfer.rs`](https://github.com/PlaticaIt/StarryOS/blob/19219411d5dc1515496f910d04c93da12ee95be4/drivers/ax-driver/src/block/k3_ufs/transfer.rs) 固定 revision permalink。G12 责任与五字段正文不变，固定 revision `19219411d5dc1515496f910d04c93da12ee95be4` 声明保留。
- 修复 R3（Plan Review Blocking / Workflow completion — tasks 状态）：`openspec/changes/establish-k3-storage-controller-baseline/tasks.md` 第 6-8 行 T4 / T5 / T6 由 `[ ]` 改为 `[x]`。T1-T3 之前已勾选保持。
- 修复 R4（Plan Review Blocking / Cycle structure — Plan Review 区域）：本轮覆盖 Act Response 时未修改 `## Plan Review` 区域（第 256-305 行）；Plan Review 中 Findings、Deviation Classification、Acceptance Gaps、Convergence、Evidence、Follow-up Decision、Iteration Plan Update、Next Cycle、Next Iteration 全部保留原文。

**Changed Files and Symbols**

- `docs/reference/known-gaps.md`（G12 区块 +23 行；R2 修复替换 G12 区块内单行 UFS permalink；汇总表 G12 行第 206 行；source-coverage 对应 G12 行第 219 行；G1-G11 未触动）
- `docs/reference/terminology.md`（初次 +15 行；R1 修复重写第 51-60 行 10 个 UFS 术语行；SPI/QSPI/SDHCI/eMMC/UFS 第 46-50 行未改；表头/分隔/标题层示例/既有 25 行未改；主写法无重复；数据行 40）
- `docs/index.md`（初次 13 行增量：storage 入口 + 主题职责行 + 主题集合说明 + 维护规则计数 + 术语表计数 + 已知缺口计数；R1/R2 修复未触动本文件；最终 80 → 83 行）
- `openspec/changes/establish-k3-storage-controller-baseline/tasks.md`（R3 修复：第 6-8 行 `[ ]` → `[x]`；T1-T3 已勾选保持）
- `openspec/changes/establish-k3-storage-controller-baseline/iterations/002-navigation-terminology-gaps/000-initial.md`（Act Response 区域状态 `reported → pending`，本轮覆盖包含原实施和 R1-R4 修复；Plan Review 区域保持不动）

**Deviations from Plan**

None. Plan Invariant "70 个唯一 URL 保持不变；本 Cycle 不写 source-coverage" 满足：`docs/reference/source-coverage.md` 未被本 Cycle 任何一轮触动；`rg '^\|.*https?://' docs/reference/source-coverage.md | wc -l` = 70（表行 70），与表中声明"本表当前含 70 个唯一 URL"一致。R1 修复仅重写术语描述与章节指针，不引入新事实、不裁决默认 DTS / 容量 / IRQ / 性能 / 恢复完整性，不改两篇 storage 正文与 source-coverage；R2 修复仅替换已接受 UFS 正文同款 permalink，不改变 G12 责任。

**Blocker Handoff**

None.

**Blocker Resolution**

None.

**Self-Review**

- Spec compliance: A1（G12 五类字段 + G5/G7 互斥 + 汇总 G12 + 修正后 UFS permalink 与 `k3-ufs.md:68,100` 一致）/ A2（15 术语唯一主写法、数据行 40；R1 修复后 10 个 UFS 术语章节定位与已接受 UFS 正文 §5-§8 一致；UTRD 描述与 `k3-ufs.md:72` 实际字段一致；UTMRD 描述与 `k3-ufs.md:70`、UCD 描述与 `k3-ufs.md:70-71`、PRDT 描述与 `k3-ufs.md:71`、SCSI 描述与 `k3-ufs.md:84-90` 实际 SCSI 命令集一致；UPIU 描述不再混入 OCS，OCS 位置与 `k3-ufs.md:72` 一致；LUN 章节与 `k3-ufs.md:84` 一致）/ A3（storage 双入口、状态 `G5 partial, G7/G12 open`）/ A4（70/40/12 与 source-coverage 70 row、terminology 40 数据行、known-gaps 12 G 条目一致）/ A5（不裁决默认 DTS、容量、IRQ、性能、恢复完整性；两篇 storage 正文和 source-coverage 未修改）全部通过。
- Code quality: `git diff --check` 退出 0；`openspec validate establish-k3-storage-controller-baseline --strict` 退出 0（输出 `Change 'establish-k3-storage-controller-baseline' is valid`）；相对链接 `storage/k3-qspi-spi-sdhci.md`、`storage/k3-ufs.md` 目标存在；R1 修复后 10 个 UFS 术语章节指针全部对齐 `k3-ufs.md` §5 / §6 / §8；R2 修复后 G12 区块 UFS permalink 域名与 `k3-ufs.md:68,100` 相同；R3 修复后 change `tasks.md` 第 6-8 行 `[x]` 与 T4/T5/T6 实施一致；R4 修复后 Plan Review 区域未被本轮覆盖触动（行号 256-305、Review Result 仍为 `pending`）。
- Critical: 0。Important: 0。Minor: 0。

**Verification Evidence**

初次实施 16 项验证 + 修复 R1-R4 验证 12 项 + Gate 5 全量校验 2 项 = 30 项全部 PASS（节选关键项）：

| 验证项 | 命令或操作 | 输出摘录 | 结论 |
|---|---|---|---|
| R1 修复: 10 个 UFS 术语章节定位 | `rg -n "^\| <term> \|" docs/reference/terminology.md` × 10 | M-PHY: §5, UniPro: §5, UTP: §6, UPIU: §6, UTRD: §6, UTMRD: §6, UCD: §6, PRDT: §6, SCSI: §8, LUN: §8 | PASS |
| R1 修复: UTRD 错误描述消失 | `rg -c 'command UPIU \+ PRDT \+ response UPIU' docs/reference/terminology.md` | 0 | PASS |
| R1 修复: UTMRD 错误描述消失 | `rg -c 'QUERY / ABORT TASK' docs/reference/terminology.md` | 0 | PASS |
| R1 修复: SCSI NOP/QUERY 错误消失 | `rg -c 'NOP/QUERY\|NOP / QUERY' docs/reference/terminology.md` | 0 | PASS |
| R1 修复: UPIU OCS/response 错误消失 | `rg -c 'OCS/response 检查' docs/reference/terminology.md` | 0 | PASS |
| R1 修复: PRDT DBC 20 bit 错误消失 | `rg -c '单 entry DBC 20 bit' docs/reference/terminology.md` | 0 | PASS |
| R2 修复: G12 UFS permalink | `rg -n 'PlaticaIt/StarryOS/blob/19219411.*k3_ufs/transfer\.rs' docs/reference/known-gaps.md` | `174:  - 固定 revision 第三方 tgoskits revision \`19219411…\` 在 [\`k3_ufs/transfer.rs\`](https://github.com/PlaticaIt/StarryOS/blob/19219411…/drivers/ax-driver/src/block/k3_ufs/transfer.rs) 中…` | PASS |
| R2 修复: rt-async-amp/tgoskits 残留 | `rg -c 'github\.com/rt-async-amp/tgoskits' docs/reference/known-gaps.md` | 0 | PASS |
| R3 修复: T4-T6 已勾选 | `rg -n '^- \[[ x]\] T[0-9] ' openspec/changes/establish-k3-storage-controller-baseline/tasks.md` | T1/T2/T3/T4/T5/T6 全部 `[x]` | PASS |
| R4 修复: Plan Review 区域保留 | `rg -n '^## Plan Review|^- Review Result: pending' openspec/changes/establish-k3-storage-controller-baseline/iterations/002-navigation-terminology-gaps/000-initial.md` | `256:## Plan Review` / `258:- Review Result: pending` | PASS |
| 跨轮基线: 术语数据行 40 | `rg '^\| [^|]+ \|' … \| sed … \| grep -v '^主写法$\|^---$' \| wc -l` | `40` | PASS |
| 跨轮基线: 缺口 G 条目 12 | `rg -c '^## G[0-9]+\.' docs/reference/known-gaps.md` | `12` | PASS |
| 跨轮基线: source-coverage 70 URL 行 | `rg '^\|.*https?://' docs/reference/source-coverage.md \| wc -l` | `70` | PASS |
| Gate 4 Code quality: diff 无越界 | `git diff --stat -- docs/index.md docs/reference/known-gaps.md docs/reference/terminology.md openspec/changes/establish-k3-storage-controller-baseline/tasks.md` | `4 files changed, 50 insertions(+), 9 deletions(-)` | PASS |
| Gate 4 Code quality: diff lint | `git diff --check` | （无输出）退出 0 | PASS |
| Gate 4 / 5: change 严格校验 | `openspec validate establish-k3-storage-controller-baseline --strict` | `Change 'establish-k3-storage-controller-baseline' is valid` 退出 0 | PASS |

**Persisted Evidence**

None required. Plan 已声明 `none`；本轮覆盖的 Act Response 完整记录初次实施与 R1-R4 修复的命令、决定性输出和退出码；三份 Markdown 目标行数 25 → 40 术语、12 G 条目、storage 双入口、UFS permalink、tasks 勾选、Plan Review 区域均可低成本重跑。不创建 `evidence/002-navigation-terminology-gaps/000-initial/` 目录。

**Experience Candidates**

None. 初次实施与本轮修复均属已受 Plan 约束的导航/术语/缺口收尾操作；命令、校验、相对链接、计数、章节指针、permalink、tasks 勾选均为可重跑静态操作；不满足 Runbook（端到端验证成功且可重复或高风险）或 Incident（已发生故障）白名单。

**Remaining Issues**

None. 初次实施与 R1-R4 修复全部完成；Plan Invariant、Non-goals、5 项基线（70 URL / 40 术语 / 12 gaps / 15 主写法 / G12 责任）均保持；本 Cycle 阻塞项为零。

**Commit or Diff Reference**

- 工作区未 commit（与既有 MS06/MS07/MS08 staged 改动一并保留）。
- 跨轮累计 `git diff --stat -- docs/index.md docs/reference/known-gaps.md docs/reference/terminology.md openspec/changes/establish-k3-storage-controller-baseline/tasks.md` = `4 files changed, 50 insertions(+), 9 deletions(-)`。
- `docs/index.md` 仅在初次实施时触动（13 行增量）；修复轮未触动。
- 修复轮 diff 集中在 `docs/reference/terminology.md`（10 个 UFS 术语行重写）、`docs/reference/known-gaps.md`（第 174 行 UFS permalink 替换）、`openspec/changes/establish-k3-storage-controller-baseline/tasks.md`（第 6-8 行 `[x]`）。

## Plan Review

- Review Result: accepted

**Findings**

- Accepted: 当前 Cycle 的 `Act Response` 已正式上报为 `reported`，实现内容、任务状态与报告一致。
- Resolved: 上轮 A2 术语语义与章节定位问题已修正；10 个 UFS 术语现与 `k3-ufs.md` §5、§6、§8 对齐，UTRD、UTMRD、UCD、PRDT、UPIU、SCSI 与 LUN 的边界描述无遗留错误。
- Resolved: 上轮 A1/A5 的 G12 UFS 链接已替换为 PlaticaIt/StarryOS 的固定 revision permalink，未改变 G12 责任或引入新事实。
- Resolved: T4-T6 已勾选，Plan Review 区域也在 Act 覆盖后保持完整。
- Accepted: G12、storage 双入口、70/40/12 计数、相对链接和 OpenSpec 严格校验均通过；Evidence 模式为 `none`，不要求持久化 Evidence 目录。

**Deviation Classification**

NONE

**Acceptance Gaps**

None.

**Convergence**

上轮 4 个 Blocking 项及其后的 1 个工作流状态问题均已收敛；无新增缺陷。

**Evidence**

- 独立状态：T1-T6 全部 `[x]`；当前 Cycle `Act Response: reported`。
- 独立内容检查：术语数据行 40、主写法无重复；G 条目 12；source-coverage URL 行 70、无重复；三份收尾文档的相对链接均存在。
- 独立语义检查：M-PHY/UniPro 指向 §5，UTP/UPIU/UTRD/UTMRD/UCD/PRDT 指向 §6，SCSI/LUN 指向 §8；修正后的描述与已接受 UFS 正文和固定源码边界一致。
- 独立链接检查：`known-gaps.md` 不再含 `github.com/rt-async-amp/tgoskits`，G12 的 `k3_ufs/transfer.rs` 使用 revision `19219411d5dc1515496f910d04c93da12ee95be4` permalink。
- 独立 Gate：`openspec validate establish-k3-storage-controller-baseline --strict` 通过；`git diff --check` 退出 0。

**Follow-up Decision**

当前 Cycle 已接受。无需返工或创建后继 Cycle；本 Iteration 的计划目标已经完成。

**Iteration Plan Update**

None.

**Next Cycle**

None.

**Next Iteration**

None.
