## 1. Iteration 000 — AP/RP 生命周期与共享窗口基线

- [x] 1.1 [T1] 在 `docs/reference/source-coverage.md` 核对生命周期正文实际引用的官方 URL；验证既有 URL 只扩充 `amp` 职责、新 URL 仅在直接打开并引用时登记，且 URL 总数等于唯一数。
- [x] 1.2 [T2] 创建 `docs/amp/k3-amp-shared-memory-lifecycle.md`，整理镜像/握手、地址 alias、窗口布局、PMA/PBMT、初始化所有权和 reset/re-init；验证阶段状态、四级证据、未知项、相对链接及 ≤450 行约束。

## 2. Iteration 001 — Ring、RPC、通知与主题收尾

- [x] 2.1 [T3] 在 `docs/reference/source-coverage.md` 核对消息路径正文实际引用的官方 URL；验证没有把 R09–R12 第三方文件重复登记为官方来源，且 URL 总数等于唯一数。
- [x] 2.2 [T4] 创建 `docs/amp/k3-rpc-ring-notification.md`，整理请求/响应/urgent、BUSY、ring、doorbell、等待、RPC 完成与失败恢复；验证正常、错误、超时、取消、并发和 reset 边界均有证据或未知项且文档 ≤450 行。
- [x] 2.3 [T5] 更新 `docs/reference/known-gaps.md` 的 G4/G5/G7 关联并新增不重复的 G11；验证每项保留当前证据、禁止推断、解除条件和影响主题，状态汇总与正文一致。
- [x] 2.4 [T6] 更新 `docs/reference/terminology.md` 的 AMP、RPC、ring、doorbell 和共享窗口术语；验证主写法、英文原词、别名和使用边界不与 mailbox hardware channel 或 DMA ring 混用。
- [x] 2.5 [T7] 更新 `docs/index.md` 的 AMP 正文入口、主题职责、来源/缺口/术语计数与状态；验证链接可解析、计数来自权威文档实际值且其他主题状态不变。

## Task Contracts

### T1：生命周期来源覆盖

- Requirement/Scenario: R1/S1-S2；R2/S3-S4；R6/S11-S12。
- Depends on: None。
- Targets: `docs/reference/source-coverage.md`。
- Current behavior: 覆盖表有 70 行且 70 个唯一 URL；已登记 K3 boot、Linux K3/CoM260 DTS、`k3-rdomain.dtsi` 和 PMA/PBMT 标准来源，但没有 `amp` 主题职责。
- Required behavior: 生命周期正文的每个直接官方 URL 在覆盖表恰有一行；既有行扩充主题职责，新行仅用于已打开且正文实际引用的官方来源。
- Required changes: 核对 R01、boot/image、K3/CoM260 DTS 与正文首行候选；记录观察对象、日期和证据边界，不预设 URL 总数增加。
- Preserve: 既有 70 行身份、状态和观察日期；M01、D04-D07；官网 SPA 壳的 `partially-observed` 状态。
- Forbidden: 不批量刷新来源；不把 R09–R12 或第三方源码路径登记为官方来源；不修改 `others/`。
- Test witness: 变更前 URL 行数/唯一数为 `70/70`，现有行没有 `amp` 主题职责。
- GREEN condition: 生命周期正文首行中的官方 URL 均唯一登记；覆盖表总数等于唯一数；新增或修改行元数据完整。
- Verification: URL 总数/唯一数、正文首行反向核对、`git diff --check`、scoped diff、strict OpenSpec validate。
- Stop when: 需要引入 M01/R08 之外的新权威来源，或来源变化改变 D1–D8、目标板范围或 Acceptance。

### T2：AP/RP 生命周期与共享窗口事实包

- Requirement/Scenario: R1/S1-S2；R2/S3-S4；R5/S9-S10；R6/S12。
- Depends on: T1。
- Targets: `docs/amp/k3-amp-shared-memory-lifecycle.md`。
- Current behavior: `docs/amp/` 和目标文件不存在；启动、mailbox、ownership、PMA/PBMT 分散在 MS03/MS05/MS06 正文，R09/R10 保存第三方端到端调查。
- Required behavior: 读者可沿镜像/握手 → 启动链破坏者 → AP/RP 地址 → 初始化/发布 → 在线 → magic 失效/reset → re-init 路径查询所有者、状态、允许操作和数据损失边界。
- Required changes: 建立 AP/RP 地址和镜像表、共享窗口布局边界、生命周期状态表、初始化竞争、PMA/PBMT/cache 边界、reset/re-init 状态转换及四字段未知项；引用相邻正文而不复制其细节。
- Preserve: M01-M04；MS03/MS05/MS06 的事实职责；G4/G5/G7；固定 revisions；简体中文与四级证据；目标文档 ≤450 行。
- Forbidden: 不声明 `0` 与 `0xc0800000` 已经真板证明为同一 SRAM；不选择默认 Kit DTS；不把 3 秒/8 次或作者日志写成 K3 安全保证；不展开 ring/RPC 消息语义。
- Test witness: `test ! -e docs/amp/k3-amp-shared-memory-lifecycle.md` 退出 0；index 没有 AMP 入口。
- GREEN condition: R1、R2、R5 的生命周期场景均有事实、推论或明确未知原因；直接来源已登记；相对链接有效；证据未升级。
- Verification: 首行来源、目录、状态表、地址/所有权、证据标签、四字段未知项、相对链接、行数、scoped diff、`git diff --check`、strict validate。
- Stop when: 实际源码 revision 或生命周期控制流变化使 D3/D5 失效，或需要新增第三篇文档/修改既有主题责任。

### T3：消息路径来源覆盖

- Requirement/Scenario: R3/S5-S6；R4/S7-S8；R6/S11-S12。
- Depends on: Iteration 000 Review Result `accepted`。
- Targets: `docs/reference/source-coverage.md`。
- Current behavior: 生命周期来源已经核对；消息/RPC 正文尚不存在，R09–R12 已在 references 而非官方来源覆盖表登记。
- Required behavior: 第二篇正文的全部直接官方 URL 唯一登记；第三方行为继续由 R09–R12 提供检索入口。
- Required changes: 核对 mailbox、中断域和共享内存边界所需的既有官方 URL；只在可打开并实际引用时新增 URL。
- Preserve: T1 的来源状态和全部既有记录。
- Forbidden: 不登记未引用候选；不把第三方仓库、测试文件或分析路径写成官方来源；不刷新无关 URL。
- Test witness: 第二篇正文不存在，当前无法证明其首行来源与覆盖表一致。
- GREEN condition: 第二篇正文官方 URL 各出现一次，总数等于唯一数，第三方 revision 只在正文证据边界和 R09–R12 中出现。
- Verification: URL 总数/唯一数、正文首行反向核对、scoped diff、`git diff --check`、strict validate。
- Stop when: 新来源改变 D4/D5、RPC Acceptance 或来源等级。

### T4：Ring、RPC 与通知事实包

- Requirement/Scenario: R2/S4；R3/S5-S6；R4/S7-S8；R5/S9-S10；R6/S12。
- Depends on: T3。
- Targets: `docs/amp/k3-rpc-ring-notification.md`。
- Current behavior: 文件不存在；mailbox 正文只承载通知层，DMA 正文只承载 ownership 抽象；R10 保存共享 ring/RPC 调用点，但 `ov-channels` 和 `rt-async` 缺失。
- Required behavior: 读者可区分三条共享通道与 mailbox channel，并追踪发布、BUSY、doorbell、ISR、重检、响应关联、Deferred/error、timeout/cancel、并发等待和 reset 后状态。
- Required changes: 建立编号空间表、正常请求/响应路径、urgent/one-way/Deferred 行为、BUSY 睡眠竞态、通知丢失/合并/重复、error/poison、多块与容量边界、等待者模型、timeout/cancel/reset 状态表和四字段未知项。
- Preserve: MS05 mailbox 清除顺序和 MS06 memory visibility 边界；G4/G5/G7；固定 revisions；文档 ≤450 行。
- Forbidden: 不补写 ring layout/原子序；不承诺 poison 一定唤醒阻塞端；不把 host test 等同真板；不把 service discovery 或 request ID 变成身份验证；不设计实现 API。
- Test witness: `test ! -e docs/amp/k3-rpc-ring-notification.md` 退出 0；offline Cargo metadata 因缺 `rt-async/modules/platform/Cargo.toml` 退出 101。
- GREEN condition: R3-R5 的正常、错误、超时、取消、并发和 reset 场景均可独立追踪；缺失源码影响明确；正文不要求读者回读分析才能理解边界。
- Verification: 首行来源、编号空间、端到端路径、失败矩阵、revision、证据标签、未知项、相对链接、行数、scoped diff、`git diff --check`、strict validate。
- Stop when: 缺失子仓被补齐且其行为推翻 D4/D5，或现有源码变化改变 RPC 可观察语义。

### T5：G4/G5/G7 与 G11 收敛

- Requirement/Scenario: R1/S2；R2/S3-S4；R3/S6；R4/S8；R5/S10；R6/S11-S12。
- Depends on: T2、T4。
- Targets: `docs/reference/known-gaps.md`。
- Current behavior: 有 G1–G10；G4/G5 为 `partial`、G7 为 `open`，尚无独立条目承载共享窗口/ring/RPC/recovery 缺口。
- Required behavior: G4/G5/G7 只增加相关正文和新证据指针；新增 G11 承载不与三者重复的协议闭包；汇总和 source-coverage 对应关系一致。
- Required changes: 为 G11 写当前证据、禁止推断、解除条件、影响主题和状态；按正文核对 G4/G5/G7，不因第三方源码存在而关闭。
- Preserve: G1-G10 的历史和既有状态，除 G4/G5/G7 的相关增量；每项四字段结构。
- Forbidden: 不按缺失仓库拆多个重复 G；不以作者声明、host tests 或未运行入口改变状态；不删除历史。
- Test witness: 变更前有 10 个 G heading，G11 不存在。
- GREEN condition: G1-G11 编号唯一；G11 与 G4/G5/G7 职责互斥；汇总计数、状态和正文一致。
- Verification: `rg` 检查编号/状态/四字段/正文链接，人工比对解除条件，scoped diff、`git diff --check`、strict validate。
- Stop when: 新发现需要改变长期 M/D/K/R/I 或超出 MS08 的独立缺口。

### T6：AMP/RPC 术语边界

- Requirement/Scenario: R3/S5-S6；R4/S7-S8；R6/S11。
- Depends on: T2、T4。
- Targets: `docs/reference/terminology.md`。
- Current behavior: 当前术语表有 20 项，包含 AP、RCPU、mailbox 相关基础词，但没有 AMP、RPC、ring、doorbell 和共享窗口的主写法。
- Required behavior: 新主题中的关键词有唯一主写法、英文原词、允许别名和禁止混用边界。
- Required changes: 增加五项术语，明确 mailbox hardware channel 与共享 ring channel、DMA ring 与 RPC ring、通知与数据的关系。
- Preserve: 既有 20 项内容与顺序；M03；标题层不使用同义并列。
- Forbidden: 不扩充无关网络、存储或实现 API 术语；不改变既有术语定义。
- Test witness: 变更前缺少上述五项主写法。
- GREEN condition: 两篇正文统一使用新增主写法，别名可检索且边界无冲突。
- Verification: 术语条目计数、正文词汇扫描、scoped diff、`git diff --check`、strict validate。
- Stop when: 新术语需要修改 M03 或既有术语的规范含义。

### T7：AMP 入口与计数一致

- Requirement/Scenario: R6/S11-S12。
- Depends on: T1-T6。
- Targets: `docs/index.md`。
- Current behavior: index 列出九类主题和 70 URL、10 gaps、20 个术语，没有 `docs/amp/` 或两篇目标正文入口。
- Required behavior: index 链接两篇实际存在的 AMP 正文，主题职责增加 AMP 行；URL、缺口和术语计数取权威文件实际值。
- Required changes: 增加正文入口和主题职责；同步实际计数与状态，不复制技术正文。
- Preserve: 其他主题链接、职责、状态和 MS09-MS11 边界。
- Forbidden: 不同步 SNAPSHOT/tasks；不硬编码计划预测数；不把第三方事实写进入口页。
- Test witness: 两个 AMP 链接和 `docs/amp` 主题行不存在。
- GREEN condition: 两个链接可解析，AMP 状态与任务一致，计数与 reference 文档一致，无关主题未改。
- Verification: 相对链接、URL/缺口/术语实际计数、scoped diff、`git diff --check`、strict validate。
- Stop when: T1-T6 未 GREEN，或需要修改其他 milestone 状态。

## Iteration Plan

### Iteration 000：AP/RP 生命周期与共享窗口基线

- Tasks: T1-T2。
- Depends on: MS03、MS05、MS06 已完成。
- Stable baseline: AP/RP 镜像/握手、共享窗口地址、初始化所有权和 reset/re-init 状态可独立引用；消息路径不需重新判断启动阶段或地址边界。
- Verification boundary: 生命周期正文的官方来源唯一登记；阶段、所有者、地址、内存属性边界和数据损失未知项完整。
- Diagnostic boundary: 镜像/握手、启动链破坏者、地址 alias、PMA/PBMT、初始化竞争、magic、reset/re-init。
- Non-goals: ring/RPC 消息语义、G11/术语/index 收尾、真板验证和实现。
- Balance audit: T1 为 T2 提供来源前置，二者共同形成独立生命周期结果；ring/RPC 依赖缺失源码且故障域不同，留到下一 Iteration。

### Iteration 001：Ring、RPC、通知与主题收尾

- Tasks: T3-T7。
- Depends on: Iteration 000 Review Result `accepted`。
- Stable baseline: MS08 两篇正文、来源、G4/G5/G7/G11、术语和入口一致，可供后续实现调查使用而不需回读 R09–R12 才能理解契约边界。
- Verification boundary: 数据与通知分层、RPC 正常/错误/等待/reset 路径完整；缺失源码和运行边界明确；引用和计数一致。
- Diagnostic boundary: ring/原子、BUSY/doorbell、mailbox/IRQ/waker、RPC completion/error/cancel、reset recovery、缺口、术语和导航。
- Non-goals: 补仓、构建、真板测试、协议或驱动实现、全局状态同步。
- Balance audit: T3 是 T4 的来源前置；T4 形成消息路径结果；T5-T7 依赖两篇正文并负责单一主题收尾，拆成独立 Iteration 不会形成新的技术基线，合并后诊断边界仍明确。

## Requirements Traceability Matrix

| Requirement | Scenario | Design | Task | Iteration | Code Surface | Test Witness | Simplification | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| R1 | S1-S2 | D1-D3 | T1,T2,T5 | 000,001 | coverage；生命周期正文；gaps | 目标文件不存在；固定 revision 源码可读 | None | Covered |
| R2 | S3-S4 | D2,D3,D5 | T1,T2,T4,T5 | 000,001 | 生命周期/RPC 正文；gaps | `ov-channels` 缺失；metadata 退出 101 | None | Covered |
| R3 | S5-S6 | D4 | T3-T6 | 001 | coverage；RPC 正文；gaps；terminology | RPC 正文不存在；mailbox 正文仅覆盖通知 | None | Covered |
| R4 | S7-S8 | D4,D5 | T3-T6 | 001 | RPC 正文；gaps；terminology | client/server 可读；error wake/cancel 未闭合 | None | Covered |
| R5 | S9-S10 | D3,D5 | T2,T4,T5 | 000,001 | 两篇正文；gaps | watchdog/re-init 可读；真板恢复未验证 | None | Covered |
| R6 | S11-S12 | D2,D6-D8 | T1-T7 | 000,001 | 两篇正文；coverage/gaps/terminology/index | 70/70 URL；10 gaps；无 AMP index | None | Covered |

## Plan Completeness Review

- TBD/TODO: None。
- Requirement simplification: None。
- R1-R6、S1-S12 均映射到 design、task、Iteration、目标文件和测试见证。
- 两个 Iteration 分别形成生命周期和消息路径稳定基线；依赖有序，只展开 Iteration 000。
- 影响面限定为两篇新正文和四个 reference/index 文件；不修改 `others/`、全局状态或既有主题正文。
- Persisted Evidence: `none`；文档结构、URL、链接、计数、diff 和 OpenSpec 验证均可低成本重跑并记录在 Act Response。
