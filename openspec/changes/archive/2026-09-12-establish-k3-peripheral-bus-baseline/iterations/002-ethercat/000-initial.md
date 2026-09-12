# Iteration 002 / Cycle 000: EtherCAT 基线

## Plan Context

- Status: ready
- Iteration: 002-ethercat
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: None

**Iteration Scope**

- Change tasks: 3.1
- Depends on: Iteration 001
- Stable baseline: EtherCAT master 的静态依赖、软件边界和未知运行条件可独立检索。
- Verification boundary: `k3-ethercat.md` 引用 MS07 且不重复 GMAC/DMA；静态绑定不被写成协议或实时性能结论。
- Diagnostic boundary: `ec_master`、`eth1` 依赖、软件组成、周期/同步/恢复证据缺口。
- Deferred tasks: 4.1-4.3

**Cycle Scope**

- Trigger: initial
- Acceptance gaps: None
- Repair items: None
- Inherited scope: proposal 已批准的 R1、R5、R6 及 D1、D2、D5、D6；Iterations 000-001 已接受的来源与总线分层基线。
- Excluded scope: 修改 GMAC/PHY/DMA 正文、实现或配置 EtherCAT 栈、真板验证、最终术语/缺口/索引和 source refresh。

**Objective**

建立 EtherCAT 独立正文，记录 `ec_master → eth1` 静态依赖，并将 Ethernet 数据面、EtherCAT 软件/协议与未经验证的实时运行条件分开。

**Background**

MS07 已记录 CoM260 `eth1` 的 GMAC、PHY、RGMII、DMA ring 和 IRQ 边界。CoM260 共享 DTS 还包含 `ec_master` 到 `eth1` 的静态 phandle，但当前没有 EtherCAT 独立正文，官方 Buildroot 页面也未直接取得。

**Current Baseline**

- `docs/buses/k3-ethercat.md` 不存在。
- `docs/boot/com260-image-and-dts.md` 记录 `&ec_master { master0 { main-device = <&eth1>; }; }`。
- `docs/network/com260-gmac-phy.md` 记录 `eth1 → RGMII → MDIO/PHY` 静态链，同时明确不覆盖 EtherCAT/TSN 协议。
- `docs/network/k3-gmac-dma-irq.md` 记录 K3 glue、DWMAC MAC/MTL/DMA、descriptor ring 与 IRQ 边界，同时明确不覆盖 EtherCAT 协议。
- `22-EtherCAT.md` 官方入口为 `current / active / partially-observed / unknown`；正文在 2026-09-11 未直接取得。

**Current-State Evidence**

- 静态依赖：CoM260 共享 DTS 的 `ec_master/master0/main-device` 指向 `eth1`；该 phandle 只表达静态关联，不证明 master driver probe、网口独占或帧交换。
- 物理与数据面：MS07 已闭合 `eth1` 的 GMAC/PHY/RGMII 静态层，以及 DWMAC descriptor/DMA/IRQ 的证据边界；本轮应链接这些文档，不复制字段、状态机或第三方实现细节。
- 软件边界：当前直接材料没有确认 EtherCAT master 实现、用户态工具、配置来源、slave 发现、Distributed Clocks 或普通网络栈共存策略。
- 运行边界：没有周期、抖动、同步精度、帧丢失、link-down、slave disappearance、超时、取消、恢复或真板日志。
- 板型边界：`ec_master` 位于共享 DTS 不等于任一顶层 CoM260 候选是默认 Kit DTB，也不证明目标板网络口已按 EtherCAT 用途布线或配置。
- 测试入口：以目标文件缺失作为 RED；使用 `rg`、相对链接检查、重复内容扫描、行数、`git diff --check` 和 OpenSpec strict validation 验证。

**Relevant Code**

- `docs/boot/com260-image-and-dts.md`：`ec_master → eth1` 静态绑定与 DTS 候选边界。
- `docs/network/com260-gmac-phy.md`：GMAC1、PHY、MDIO、RGMII 静态责任。
- `docs/network/k3-gmac-dma-irq.md`：MAC/MTL/DMA、descriptor、IRQ 和错误边界。
- `docs/reference/source-coverage.md`：EtherCAT 官方入口和 supporting DTS 唯一记录。

**Critical Path**

CoM260 shared DTS `ec_master/master0` → `main-device = <&eth1>` → MS07 GMAC/PHY/DMA 基线 → EtherCAT master 软件与协议未知项 → 后续导航和缺口收敛。

**Implementation Guidance**

正文按范围、静态绑定、对象职责、软件/协议边界、运行与错误边界、未知项和导航组织。对 GMAC/PHY/DMA 只保留依赖方向与链接；软件组成未获直接证据时列为未知，不用通用 EtherCAT 栈补写 K3 行为。

**Behavioral Change**

当前 EtherCAT 只有散落的 DTS 绑定。完成后读者可从独立正文找到静态依赖、MS07 入口及实时运行证据缺口；产品代码、网络配置和运行状态不变。

**Change Surface**

| Task | Requirement/Scenario | File/Symbol | Current Responsibility | Planned Change |
|---|---|---|---|---|
| 3.1 | R1/R5/R6；静态 master 绑定、实时证据缺失、来源不可访问 | `docs/buses/k3-ethercat.md` | 文件不存在 | 创建 EtherCAT 静态依赖与协议边界正文 |

**Task Contracts**

### 3.1：EtherCAT 静态绑定和协议边界可检索

- Requirement/Scenario: R1、R5、R6；静态 master 绑定、实时运行证据缺失、官方页面不可访问。
- Depends on: Iteration 001 accepted。
- Targets: 新建 `docs/buses/k3-ethercat.md`。
- Current behavior: 目标文件不存在；`ec_master → eth1` 只在 DTS 摘要中出现，MS07 明确不承担 EtherCAT 协议。
- Required behavior: 首行列全部直接来源；正文记录 `ec_master/master0/main-device → eth1` 静态依赖，区分 GMAC/PHY/DMA、EtherCAT master、协议配置和应用/从站，并列出软件组成、周期、同步、错误恢复和真板状态的证据边界。
- Required changes: 链接两篇 MS07 正文和 DTS 摘要；为未知项填写当前证据、禁止推断、解除条件、影响主题；明确 shared DTS 与默认目标 DTB 分离。
- Preserve: MS07 的 GMAC/PHY/DMA 权威职责、G7 默认 DTS 边界、官方事实与交叉验证分级、既有产品正文。
- Forbidden: 不复制 descriptor ring、IRQ handler、PHY reset/delay 等 MS07 数据路径；不声明 master 已 probe、slave 已发现、周期或同步精度达标、错误恢复成功或真板已运行；不补写未观察的 Buildroot 命令。
- Test witness: `test ! -e docs/buses/k3-ethercat.md` 预期退出 0。
- GREEN condition: 文件存在，静态绑定、对象/软件/协议边界、MS07 链接、运行限制和四字段未知项齐全；无重复 GMAC/DMA 数据路径；正文不超过 500 行。
- Verification: `rg` 检查首行、`ec_master`、`master0`、`main-device`、`eth1`、master/slave、周期、同步和恢复边界；检查禁止复制的 descriptor/doorbell/ring 细节未出现；提取相对链接逐一 `test -e`；`git diff --check` 与 strict validation 退出 0。
- Stop when: 新来源要求改变 EtherCAT/GMAC 职责、确认具体 master 软件契约、选择默认 DTS 或改变既有 R5 Acceptance。

**Invariants**

- M01-M04、D02、D04-D07 保持有效。
- 静态 phandle、GMAC link 和 EtherCAT 协议运行不互相替代。
- 不修改 `others/`、MS07 正文或全局 tasks/SNAPSHOT。

**Non-goals**

不实现、配置或验证 EtherCAT 栈；不测周期、同步、吞吐或恢复；不更新 terminology、known-gaps、index；不创建 Evidence。

**Acceptance**

- A1 / R1、R5：`ec_master → master0 → main-device = eth1` 静态依赖和 shared DTS 范围明确。
- A2 / R5：GMAC/PHY/DMA、EtherCAT master、协议配置、应用和 slave 对象分层，并链接 MS07 而不复制数据路径。
- A3 / R5：周期、同步精度、错误/超时/恢复和真板结果均保持未知，禁止静态绑定升级为运行结论。
- A4 / R6：首行直接来源均可回到 coverage 唯一行；相对链接、未知项结构、行数和范围检查通过。

**Verification**

- RED/GREEN：目标文件由不存在变为存在，必需对象和边界匹配。
- 来源：首行 URL 与 coverage URL 列精确匹配一行。
- 非重复：正文不展开 `descriptor ring`、doorbell、OWN、DMA channel status 或 PHY reset/delay 值。
- 链接与结构：相对链接逐一存在；正文不超过 500 行；四字段未知项计数一致。
- 范围：产品 diff 仅新增目标正文，不包含 MS07、汇总文档或 `others/`。
- 质量：`git diff --check` 和 OpenSpec strict validation 均退出 0。

**Gate 2 Readiness**

| Dimension | Status | Evidence |
|---|---|---|
| Investigation | PASS | DTS 摘要定位 `ec_master → eth1`；两篇 MS07 正文闭合依赖边界；来源状态和验证入口已定位 |
| Design | PASS | D1、D2、D5、D6 已闭合独立成文、不可达、静态/运行分离和验证策略 |
| Iteration Plan | PASS | task 3.1 独占 Iteration 002，依赖 001，协议正文与最终汇总分离 |
| Cycle Scope | PASS | 本轮只创建 EtherCAT 正文；tasks 4.1-4.3 deferred |
| Task Contracts | PASS | 目标、行为、保持/禁止、见证、GREEN、验证和停止条件完整 |
| Traceability | PASS | task 3.1 映射 R1/R5/R6、对应场景、D1/D2/D5/D6、目标文件和直接检查 |
| Verification | PASS | 静态绑定、对象边界、非重复、未知项、链接、范围和 strict validation 直接证明 Acceptance |

- Gate 2 plan approval: PASS — 用户于 2026-09-11 回复 `批准`，批准本 Plan/Cycle。

**Persisted Evidence**

- Mode: none
- Act Response 足以保存可重跑命令、决定性输出、退出码和结论。
- Budget: 本 Cycle 最多 5 个文件（含 README），整个 change 最多 20 个 Evidence 文件；本计划不创建 Evidence。

**Risks and Notes**

- EtherCAT 官方正文不可达不阻塞静态基线，但具体 master 软件、工具和配置保持未知。
- `main-device = <&eth1>` 不自动证明网口独占、普通 Ethernet 共存或 master 已启动。
- 本 Iteration 内容较少但故障域独立；与最终计数和导航合并会混合技术正文与汇总职责。

## Act Response

- Status: reported

**Implemented**

- Task 3.1: 新建 EtherCAT 正文，记录 `ec_master → master0 → main-device = <&eth1>` 的共享 DTS 静态依赖。
- 分离 GMAC/PHY/DMA、EtherCAT master、协议配置、应用与 slave 职责；链接 MS07 正文，不复制其数据路径。
- 保留 master 软件、目标 DTB、网口所有权、slave 拓扑、周期/同步、错误恢复和真板状态的未知边界。

**Changed Files and Symbols**

- `docs/buses/k3-ethercat.md`: 新增静态绑定、职责分层、运行边界和五项四字段未知项。
- `openspec/changes/establish-k3-peripheral-bus-baseline/tasks.md`: 勾选 task 3.1。
- 本 Cycle 文件：仅回填 Act Response；Plan Context 和 Plan Review 未修改。

**Deviations from Plan**

- 两个验证正则作等价调整：URL 计数由正则匹配改为去空格后的字段精确比较；肯定式结论扫描改为人工上下文审查配合负向边界定位。原因是原正则分别误解释 URL 元字符、误命中“不证明/不得声明”语句；产品内容和测试策略未改变。

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

Spec compliance review 先于文档质量审查完成。A1-A4 均覆盖；shared DTS、默认 DTB、静态/运行分层和 MS07 职责保持不变。全文未展开 descriptor、doorbell、ring、IRQ handler 或 PHY reset/delay 数据路径，也未新增无依据命令、性能或真板结论。

**Verification Evidence**

| 验证项 | 命令或操作 | 输出摘录 | 覆盖范围 | 结论 |
| --- | --- | --- | --- | --- |
| Gate 3 RED | `test ! -e docs/buses/k3-ethercat.md` | exit 0 | 新正文修改前状态 | PASS |
| Task 3.1 GREEN | 内容、来源、链接、行数与四字段 shell 检查 | `GREEN file=present lines=118 urls=2 links=4 unknowns=5`；exit 0 | A1-A4、目标正文 | PASS |
| 边界与写作审查 | 禁止句式扫描、负向运行边界定位及完整正文审查 | 8 处“不证明/不得声明/缺少证据”等决定性边界；exit 0 | 非运行结论、BetterMd 质量 | PASS |
| 差异质量 | `git diff --check` | 无输出；exit 0 | 当前工作区 diff 格式 | PASS |
| OpenSpec | `openspec validate establish-k3-peripheral-bus-baseline --strict` | `Change 'establish-k3-peripheral-bus-baseline' is valid`；exit 0 | change 结构与规范 | PASS |

**Persisted Evidence**

None required.

**Experience Candidates**

None.

**Remaining Issues**

- 正文明确保留五类未知项；它们不阻塞本 Cycle 的静态知识基线。
- tasks 4.1-4.3 属于 deferred Iteration 003，本 Cycle 未执行。

**Commit or Diff Reference**

None.

## Plan Review

- Review Result: accepted

**Findings**

- 阻塞 findings：None。
- 非阻塞 Minor findings：None。
- 独立正文审查确认静态绑定、对象分层、运行边界和五项未知项符合 A1-A4；MS07 数据路径只以链接和“不复制”边界出现。

**Deviation Classification**

None. Act 记录的两个验证正则调整属于等价检查修正，不改变产品内容、Acceptance 或测试策略。

**Acceptance Gaps**

None.

**Convergence**

N/A.

**Evidence**

- 采信 Act Response 中未失效的 Gate 3 RED 与 GREEN：目标正文由不存在变为 118 行文件，2 个首行 URL、4 个相对链接和 5 组四字段未知项均通过。
- 独立来源审计：首行官网 EtherCAT URL 与 supporting `k3_com260.dtsi` URL 在 coverage URL 列各精确匹配 1 行。
- 独立结构审计：正文 118 行；`当前证据/禁止推断/解除条件/影响主题` 各 5 项；4 个相对链接全部存在。
- 独立边界审计：`descriptor/doorbell/ring/IRQ handler/PHY reset/delay` 只出现在链接职责或明确“不复制”语句中；所有 probe、slave、周期、同步、恢复和真板表述均保持否定或未知边界。
- 新鲜 Gate：`git diff --check`、`git diff --cached --check` 和 `openspec validate establish-k3-peripheral-bus-baseline --strict` 均退出 0；OpenSpec 输出 `Change 'establish-k3-peripheral-bus-baseline' is valid`。

**Follow-up Decision**

接受本 Cycle。A1-A4 均满足，无需当前 Cycle 修复或后继 Cycle；按既有 Iteration Map 展开 Iteration 003 draft，等待 Gate 2 用户批准。

**Iteration Plan Update**

None.

**Next Cycle**

None.

**Next Iteration**

`../003-navigation/000-initial.md`
