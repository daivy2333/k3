## 1. Iteration 000 — 平台控制与来源基线

- [x] 1.1 [T1] 在 `docs/reference/source-coverage.md` 登记本 change 实际引用的 PINCTRL/UART/Clock/Reset、RCPU/pinctrl DTSI、8250 binding/driver 精确 URL，并同步唯一 URL 数。
- [x] 1.2 [T2] 创建 `docs/platform/k3-platform-control.md`，形成 AP、APBC2 secure 与 RCPU 分域的 pinctrl、clock、reset、APBC/CCU 和 UART 初始化依赖基线。

## 2. Iteration 001 — UART、console 与跨文档收尾

- [x] 2.1 [T3] 创建 `docs/serial/com260-uart.md`，聚合 17 路 UART 实例、DTS 属性、CoM260 UART0/console 路径、来源冲突和固定 revision 第三方经验。
- [x] 2.2 [T4] 修正 `docs/boot/com260-image-and-dts.md` 中 `serial0` 静态映射的旧未知项，并把运行时 console/cmdline 的后续归属调整到 MS04。
- [x] 2.3 [T5] 复核 `docs/reference/known-gaps.md` 的 G1-G7；只有出现语义不重复且影响 MS04 的新缺口时才新增，否则记录无需修改的验证结论。
- [x] 2.4 [T6] 更新 `docs/index.md`，链接平台控制与串口正文，并把 serial 状态改为已聚合。

## Task Contracts

### T1：精确来源可追溯且身份不升级

- Requirement/Scenario: R1/S1-S4，R2/S1-S5，R3/S1-S4，R5/S1-S2、S6。
- Depends on: None.
- Targets: `docs/reference/source-coverage.md`。
- Current behavior: PINCTRL/UART/Clock/Reset 只有官网行和 GitHub 仓库根行；`k3.dtsi` 与 `k3_com260.dtsi` raw URL 已登记，但 RCPU/pinctrl DTSI、8250 binding/driver 和四个 docs-buildroot 对应页未精确登记。
- Required behavior: 重新直接打开正文实际需要的候选 URL；只登记已观察并将被引用的唯一 URL，字段包含来源类型、范围、主题、优先级、聚合状态、修订/分支、观察日期、访问状态和限制；同步表头实际唯一 URL 数。
- Required changes: 候选包括 docs-buildroot `01-PINCTRL.md`、`05-UART.md`、`16-Clock.md`、`Reset.md`，linux-6.18 `k3-rdomain.dtsi`、`k3-pinctrl.dtsi`、`8250_of.c`、`8250.yaml`；已存在的 `k3.dtsi` 与 `k3_com260.dtsi` 不重复添加。
- Preserve: 既有 51 行和各自观察日期；官网行的 `official-doc/partially-observed` 身份；M01/D04/D05。
- Forbidden: 不登记无法打开或正文不引用的候选；不把 GitHub 页面/源码标为 `official-doc`；不使用 GitHub revision 代填官网；不批量刷新既有行。
- Test witness: 修改前八个候选精确 URL 均未登记，表头唯一 URL 数为 51。
- GREEN condition: 每个正文引用 URL 在表中恰好一次；表格数据行数、唯一数和表头声明一致；新增行身份与直接观察相符。
- Verification: 用 `rg -F` 逐个检查实际引用 URL；提取表格首列比较总数与唯一数；检查新增行字段；`git diff --check`。
- Stop when: 外部页面身份、分支或内容相对 2026-09-08 调查发生变化并影响 D3-D6，或正文必须依赖未获授权的非官方来源。

### T2：平台控制资源图按域分离

- Requirement/Scenario: R1/S1-S4，R2/S1、S5，R5/S2-S4。
- Depends on: T1.
- Targets: `docs/platform/k3-platform-control.md`。
- Current behavior: 文件不存在；平台入口只有 SoC/板级资源文档，pinctrl、clock、reset、APBC/CCU 和 UART consumer 关系散落在来源中。
- Required behavior: 首行只引用 T1 已登记来源；按 AP、APBC2 secure、RCPU 列出 provider、地址/节点、consumer 属性和证据等级，形成 `pinctrl → clock/gate → reset → UART → IRQ` 的可证依赖图；第三方固定写序列单独标记。
- Required changes: 包含 AP pinctrl `0xd401e000` 的当前 DTS 属性、AP UART 的 APBC/APBC2/MPMU 关系、RCPU UART control/slow-clock 关系、provider/consumer 初始化责任、clock/reset/pin 电气未知项及解除条件。
- Preserve: 简体中文、术语主写法、首行模板、四级证据、单文件 450 行预警；SoC/模组/Kit 分层；目标 Kit DTS 非唯一边界。
- Forbidden: 不复制 17 路完整实例表；不把第三方 APBC/MPMU bit、固定 divisor、pad83 自愈或 probe 顺序写成硬件规范；不展开异步 UART、IRQ controller 或 DMA。
- Test witness: 文件不存在；platform 入口没有 AP/APBC2/RCPU provider-consumer 基线。
- GREEN condition: 三个资源域均有来源、provider、consumer 和未知边界；官方事实/官方源码/第三方经验没有互相提升；文件少于 450 行，或按 D1 合理拆分。
- Verification: 检查首行 URL 已登记；检查 AP、APBC2、RCPU、pinctrl、clock、reset、APBC/CCU、初始化依赖和四级证据；检查未知项四字段；解析相对链接；`wc -l`、`git diff --check`、OpenSpec strict validation。
- Stop when: 来源无法区分 AP 与 RCPU provider，或新证据改变资源域、provider/consumer 模型或 Gate 1 范围。

### T3：UART/console 与第三方经验形成分层事实包

- Requirement/Scenario: R2/S1-S5，R3/S1-S4，R4/S1-S4，R5/S2-S4。
- Depends on: Iteration 000 accepted.
- Targets: `docs/serial/com260-uart.md`。
- Current behavior: `docs/serial/` 和目标文件不存在；当前仅在 MS03 文档中记录 Kit UART0 物理接口和未完全解析的 console 线索。
- Required behavior: 首行只引用已登记的精确来源；建立 11 路 AP 与 6 路 RCPU 实例表、属性和 compatible/driver 路径，连接 CoM260 UART0 物理接口与 console，区分静态/运行时边界、来源冲突和固定 revision 的第三方经验。
- Required changes: 记录 address/IRQ/status/provider、32-bit stride-4、DTS FIFO 256/threshold 32、`serial0=&uart0`、`uart0_0_cfg`、`stdout-path`、`ttyS0`；并列 tgoskits FIFO 64、UUE/OUT2、Errata #20/#75、THRE/TEMT、32-pass/256-sample budget、短写，以及 Rt-Async-AMP clock/pad 风险。
- Preserve: 平台 provider 细节链接 T2 文档而不重复；保留 MS03 的 CoM260 物理接口事实与来源版本、运行时 console 未知项、目标 DTS 非唯一边界，以及第三方 revision `ccb1ff0...` 和 `1921941...`。
- Forbidden: 不声称运行时 console 已确认；不选择 FIFO 64 或 256 作为通用硬件真值；不把 `spacemit,k1-uart` 当作 K1 身份；不从第三方注释制定异步驱动设计或接线方案。
- Test witness: 文件和 serial 目录不存在；现有文档没有完整的 17 实例清单、静态 console 链、冲突表和固定 revision 经验表。
- GREEN condition: 17 路实例按域分开；DTS 字段和 console 阶段均可追溯；冲突与未知项未被擅自裁决；第三方行为边界明确；文件少于 450 行或按 D1 拆分。
- Verification: 检查首行来源及覆盖登记、AP 11/RCPU 6 数量、必要字段/地址/IRQ、console 链、256/64 冲突值、两个 commit、经验分类、未知项四字段和相对链接；运行 `wc -l` 与 `git diff --check`。
- Stop when: 官方来源改变实例或资源事实，或新运行时证据将改变静态/运行时契约而需要重新规划。

### T4：MS03 的 serial0 勘误保持有限范围

- Requirement/Scenario: R3/S1、S3-S4，R5/S3。
- Depends on: T3.
- Targets: `docs/boot/com260-image-and-dts.md`。
- Current behavior: §6.2 称 `serial0` alias 尚未单独提取并禁止解析；§6.5 与 §8 第 5 项把实际 console/cmdline 后续工作交给 MS05/MS07，但 UART/console 属于 MS04。
- Required behavior: 记录当前 `k3.dtsi` 将 `serial0` 静态映射到 `uart0`；保留 bootloader 环境和运行时 cmdline/console 未知边界；把相关影响和后续归属改为 MS04，并链接新串口文档。
- Required changes: 只修改 serial0/console 相关陈述及直接受影响的导航或修订快照，明确静态来源分支和观察日期。
- Preserve: 全部镜像、DTS 候选、CMA、GMAC、存储和非 console 未知项；目标 Kit DTS 继续保持非唯一。
- Forbidden: 不广泛重写 MS03；没有日志/镜像时不闭合运行时 console；不改变无关的 MS05-MS07 归属。
- Test witness: 现有行包含“aliases ... 尚未单独提取”、“不由 stdout-path 反推 serial0”和“Kit 实际 console ... 由 MS07 进一步展开”。
- GREEN condition: 过时的静态未知表述已移除；静态 alias 解析和剩余运行时未知项同时存在；相关影响归属为 MS04；无关章节没有语义 diff。
- Verification: `rg` 确认旧表述消失且 `serial0 = &uart0`、MS04、运行时未知项存在；审查本文件 scoped diff 和相对链接；运行 `git diff --check`。
- Stop when: 勘误需要选择唯一 Kit 顶层 DTS，或会改变其他 milestone 已批准的需求。

### T5：MS04 缺口不与 G1-G7 重复

- Requirement/Scenario: R1/S4，R2/S4-S5，R3/S3-S4，R4/S4，R5/S5。
- Depends on: T3-T4.
- Targets: `docs/reference/known-gaps.md`。
- Current behavior: G1-G7 已覆盖范围、官网事实、programmer reference、目标 Kit DTS 和下游平台影响；目前没有证据证明必须新增 MS04 缺口。
- Required behavior: 将每个 platform/UART 未知项与 G1-G7 做语义比较。已有条目覆盖时不修改文件，并在 Act Response 记录无需修改；只有对象、当前证据、禁止推断、解除条件和影响均不被 G1-G7 覆盖时才新增。
- Required changes: 条件性修改；新增行或章节必须沿用现有缺口结构并同步实际数量，否则该文件不产生产品 diff。
- Preserve: 没有新直接证据改变对应缺口时，保留 G1-G7 的状态和措辞；G7 目标 DTS 继续开放。
- Forbidden: 现有缺口已覆盖时，不新增“缺 UART TRM”“目标 DTS 未知”“运行时 console 未知”或“来源不可用”的同义项；不凭第三方代码关闭缺口。
- Test witness: 实施前共有七个缺口，且没有已批准的第八个缺口；proposal 中的已知未知项预计均可由现有条目覆盖。
- GREEN condition: 文件不变且 Act Response 保存语义比较结果，或者每个新增/更新缺口都有新直接证据且数量、链接一致。
- Verification: 比较标题/表格并统计 G 编号；审查 scoped diff；检查新增缺口的证据和解除字段；运行 `git diff --check`。
- Stop when: 新缺口会改变已批准范围、稳定基线或下游 milestone 契约，而不只是记录未知项。

### T6：总入口反映平台与串口交付

- Requirement/Scenario: R5/S2-S3。
- Depends on: T2-T5.
- Targets: `docs/index.md`。
- Current behavior: platform 行只链接 MS03 文档；serial 行为“待聚合”且没有主题链接。
- Required behavior: platform 行链接 `k3-platform-control.md`；serial 行链接 `com260-uart.md` 并改为“已聚合”；入口页继续只承担索引职责，不复制技术细节。
- Required changes: 只增加反映实际交付所需的相对链接和简短状态/职责文字。
- Preserve: 其他主题行、来源数量、职责和 MS03 链接。
- Forbidden: 不在 index 写 UART 地址或寄存器事实；不改变 interrupts/DMA/network/storage/buses/peripherals 的状态。
- Test witness: 两个新相对链接不存在，serial 仍标为“待聚合”。
- GREEN condition: 两个链接均可解析；platform/serial 状态与实际文件一致；无关行语义不变。
- Verification: 提取 Markdown 相对链接并检查目标存在；`rg` 检查两个文件名和 serial 状态；审查 scoped diff；运行 `git diff --check`。
- Stop when: 任一目标文档未达到 GREEN，或拆分后尚无稳定索引目标。

## Iteration Map

| Iteration | Tasks | Outcome | Stable baseline | Verification boundary | Diagnostic boundary | Non-goals | Balance |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 000 — 平台控制与来源基线 | T1-T2 | 精确来源和 AP/APBC2/RCPU provider-consumer 平台控制文档 | UART/console 文档可引用已分域的 pinctrl/clock/reset 基线 | URL 唯一性、来源身份、三域控制链、未知项、链接、行数、strict validate | 来源身份、provider、资源域、clock/reset/pinctrl 关系 | UART 完整实例表、console、MS03 勘误、导航 | 2 个文件；来源登记与正文强依赖，范围单一 |
| 001 — UART、console 与跨文档收尾 | T3-T6 | UART/console 知识包和仓库导航一致 | MS04 全部需求可由正文、勘误、缺口判断和入口定位 | 17 实例、console 链、冲突/经验边界、旧表述移除、链接、全量 diff | UART 字段、静态/运行时边界、第三方分类、跨文档一致性 | 驱动实现、运行时验证、后续 milestone | 最多 4 个产品文件；共享 UART 事实和最终一致性检查，规模适中 |

## Requirements Traceability Matrix

| Requirement | Scenario | Design | Task | Iteration | Code Surface | Test Witness | Simplification | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| R1 | S1 AP 控制链 | D2,D4 | T1,T2 | 000 | `source-coverage.md`; `k3-platform-control.md` | AP provider/consumer 章节不存在 | None | Covered |
| R1 | S2 RCPU 独立资源 | D2,D4 | T1,T2 | 000 | 同上 | RCPU 分域章节不存在 | None | Covered |
| R1 | S3 第三方控制链 | D2,D4,D7 | T2,T3 | 000,001 | 两篇正文 | 第三方经验分类不存在 | None | Covered |
| R1 | S4 控制链未知 | D2,D4 | T2,T5 | 000,001 | platform; known-gaps | 未知项四字段检查 | None | Covered |
| R2 | S1 DTS 实例字段 | D5 | T1,T3 | 001 | source coverage; serial doc | 17 实例表不存在 | None | Covered |
| R2 | S2 数量与集合分层 | D2,D5 | T3 | 001 | serial doc | AP11/RCPU6 分层不存在 | None | Covered |
| R2 | S3 compatible 边界 | D5 | T3 | 001 | serial doc | k1 名称边界不存在 | None | Covered |
| R2 | S4 FIFO 冲突 | D5,D7 | T3,T5 | 001 | serial; known-gaps | 256/64 冲突表不存在 | None | Covered |
| R2 | S5 Kit DTS 非唯一 | D2,D5 | T2,T3,T5 | 000,001 | platform; serial; gaps | G7 与共享字段边界检查 | None | Covered |
| R3 | S1 serial0 静态映射 | D6 | T3,T4 | 001 | serial; boot/image | MS03 仍写未解析 | None | Covered |
| R3 | S2 UART0 物理接口 | D6 | T3 | 001 | serial doc | 物理接口到节点的链路不存在 | None | Covered |
| R3 | S3 runtime override | D6 | T3,T4,T5 | 001 | serial; boot/image; gaps | 运行时证据未知分支 | None | Covered |
| R3 | S4 staged console | D6 | T3,T4 | 001 | serial; boot/image | earlycon/tty 阶段区分不存在 | None | Covered |
| R4 | S1 initialization bits | D7 | T3 | 001 | serial doc | UUE/OUT2 固定 revision 章节不存在 | None | Covered |
| R4 | S2 THRE/TEMT | D7 | T3 | 001 | serial doc | 完成语义表不存在 | None | Covered |
| R4 | S3 IRQ budget | D7 | T3 | 001 | serial doc | 32/256 budget 语境不存在 | None | Covered |
| R4 | S4 divisor errata | D7 | T3,T5 | 001 | serial; gaps | Errata #20/#75 边界不存在 | None | Covered |
| R5 | S1 exact URLs | D3 | T1 | 000 | source coverage | 八个候选 URL 未登记 | None | Covered |
| R5 | S2 SPA fallback | D2,D3 | T1,T2,T3 | 000,001 | coverage; docs | 来源身份字段检查 | None | Covered |
| R5 | S3 document delivery | D1,D9 | T2,T3,T4,T6 | 000,001 | all deliverables | 交付文件与链接不存在 | None | Covered |
| R5 | S4 文档行数边界 | D1,D9 | T2,T3 | 000,001 | 两篇主题正文 | 450 行预警和拆分检查 | None | Covered |
| R5 | S5 不重复缺口 | D6,D7 | T5 | 001 | known-gaps | 语义比较和 G 编号计数 | None | Covered |
| R5 | S6 来源变化 | D3,D9 | T1-T6 | 000,001 | 全部变更面 | revision 复核与停止条件 | None | Covered |

## Plan Completeness Review

- TBD/TODO: None.
- Requirement simplification: None。
- 23 个场景全部为 `Covered`，没有 `Missing` 或 `Simplified` 行。
- T1-T6 各自只针对一个产品 Markdown 文件，并包含依赖、当前/目标行为、保留/禁止边界、见证、GREEN、验证和停止条件。
- Gate 2 批准前只展开 Iteration 000 的 Cycle；Iteration 001 保留在本 Map 中。
- Persisted Evidence: `none`；命令和决定性输出可由 Act Response 承载并可重跑。
