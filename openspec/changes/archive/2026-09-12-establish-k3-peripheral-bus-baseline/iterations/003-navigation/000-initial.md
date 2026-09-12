# Iteration 003 / Cycle 000: 导航、术语与缺口收敛

## Plan Context

- Status: ready
- Iteration: 003-navigation
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: None

**Iteration Scope**

- Change tasks: 4.1, 4.2, 4.3
- Depends on: Iterations 000, 001, 002
- Stable baseline: 三篇总线正文能从总索引进入，来源、术语、缺口和计数一致，MS10 可进入最终 Review。
- Verification boundary: 链接有效、术语无重复、缺口正文与汇总一致、索引计数准确、change 严格校验通过。
- Diagnostic boundary: `known-gaps.md`、`terminology.md`、`index.md` 三个汇总表面。
- Deferred tasks: None

**Cycle Scope**

- Trigger: initial
- Acceptance gaps: None
- Repair items: None
- Inherited scope: proposal 已批准的 R1-R7、D1-D6、tasks 4.1-4.3，以及 Iterations 000-002 已接受的三篇 buses 正文与来源职责。
- Excluded scope: 新增总线技术事实、改写三篇 buses 正文、source refresh、SNAPSHOT/全局 tasks/M/R/I、真板验证和 change 收尾。

**Objective**

把三篇 buses 正文接入缺口、术语和总索引，使 70 个来源、53 个术语、13 个缺口与三篇正文形成一致导航。

**Background**

Iterations 000-002 已分别接受 I2C/USB、PCIe/CAN 和 EtherCAT 正文。`docs/index.md` 仍把 `docs/buses/` 标为待聚合，术语表没有 buses 专属主写法，known-gaps 只在 G3/G7 等既有条目零散提到部分总线边界。

**Investigation Facts**

- Current Baseline: `source-coverage.md` 有 70 个唯一 URL；`terminology.md` 有 40 个基础术语；`known-gaps.md` 有 G1-G12 共 12 项，G3/G4/G5 为 `partial`；`index.md` 同步显示 70/40/12，但 `docs/buses/` 状态仍为“待聚合；R06 状态 deferred”。
- Current-State Evidence: `k3-i2c-and-usb.md`、`k3-pcie-and-can.md`、`k3-ethercat.md` 已存在并分别承载控制器/设备/PHY、静态拓扑/物理层、master/协议运行边界；三篇正文各自保留四字段未知项。coverage 的五个 MS10 官网入口已为 `buses / current / active`，URL 总数无需变化。
- Code and Critical Path: 三篇正文未知项 → `known-gaps.md` G7 与新 G13 聚合边界 → `terminology.md` 唯一主写法 → `index.md` 三入口、buses 状态和 70/53/13 计数。三项汇总必须按此顺序执行，避免提前写入错误计数。

**Implementation Guidance**

先更新 known-gaps，再登记术语，最后按实际结果修改 index。G13 聚合总线控制器到板级映射、软件运行路径和恢复证据，不复制三篇正文的全部未知项；G7 只补 buses 二级指针并保持 `open`。术语定义以正文已使用的对象边界为准，不新增技术结论。

**Behavioral Change**

当前读者只能直接打开三篇正文，无法从总索引进入，也没有 buses 术语和统一缺口入口。完成后 index 提供三篇入口，术语表固定 13 个 buses 主写法，G13 汇总独立运行闭包；来源总数保持 70，缺口总数变为 13。

**Task Contracts**

### 4.1：总线未知项映射到 G7 与 G13

- Requirement/Scenario: R1、R6；资源映射不唯一、来源不可访问、静态关系缺少运行闭包。
- Depends on: Iteration 002 accepted。
- Targets: `docs/reference/known-gaps.md` 的 G7、G13 和缺口状态汇总。
- Current behavior: G7 未列 buses 三篇正文；G1-G12 中没有统一承载 I2C/USB/PCIe/CAN/EtherCAT 板级映射、软件路径和运行恢复边界的独立缺口。
- Required behavior: G7 增加三篇 buses 正文中依赖默认目标 DTS 的二级指针并保持 `open`；新增 G13，按四字段汇总三篇正文的独立总线缺口，状态 `open`；状态表与说明同步为 13 项。
- Required changes: G13 只引用正文并概括 I2C/USB/PCIe/CAN/EtherCAT 的板级映射、controller/PHY/收发器、软件组成、周期/同步和错误恢复边界；不得复制每个 U 项全文。
- Preserve: G1-G12 的编号、状态和既有事实；G3/G4/G5 保持 `partial`；G7 不因新增引用而解除；source-coverage 不修改。
- Forbidden: 不新增 G14；不把访问失败写成页面不存在；不裁决默认 DTS、I²C 9/10 冲突、USB/PCIe PHY mux、CAN C2 或 EtherCAT 运行状态。
- Test witness: 修改前 `rg -n '^## G13\.' docs/reference/known-gaps.md` 无匹配，状态表只有 G1-G12。
- GREEN condition: G13 存在且具 `当前证据/禁止推断/解除条件/影响主题` 四字段，状态表含 G13 open；G7 引用三篇 buses 正文；G1-G12 状态不变。
- Verification: `rg` 检查 G7/G13、四字段和三篇链接；统计 `^## G[0-9]+\.` 与状态表均为 13；相对链接逐一存在；diff 审查确认既有状态未漂移。
- Stop when: 未知项需要解除既有 gap、改变状态 schema，或发现不属于 G7/G13 的第二类独立总线缺口。

### 4.2：登记 13 个 buses 主写法

- Requirement/Scenario: R7；正文已使用但术语表缺少统一主写法。
- Depends on: 4.1。
- Targets: `docs/reference/terminology.md` 基础术语表。
- Current behavior: 术语表有 40 项；缺少 I2C、USB、DRD、role switch、Hub、PCIe、RC、EP、FlexCAN、CAN-FD、EtherCAT、master、slave。
- Required behavior: 增加上述 13 项，分别限定总线、USB 角色、PCIe 端点、CAN 控制器/协议和 EtherCAT 对象范围，并链接对应正文位置。
- Required changes: 主写法逐项唯一；英文原词/缩写和别名不与既有行冲突；定义只复述正文已接受的职责边界。
- Preserve: 既有 40 项内容和顺序；`PHY`、`DMA`、`GMAC` 等跨主题术语继续由原条目承担。
- Forbidden: 不新增仅为凑数的术语；不把 controller 静态存在定义成功能可用；不改写 storage/network/amp 术语。
- Test witness: 修改前 13 个主写法在术语表首列均不存在，基础术语共 40 项。
- GREEN condition: 首列恰好新增 13 个唯一主写法，总数 53；每项包含对象范围和正文相对链接，全部链接有效。
- Verification: awk 提取首列检查 53 项且无重复；逐项精确匹配 1 次；提取新增链接并逐一 `test -e`；diff 审查既有 40 行未变化。
- Stop when: 正文对任一主写法存在冲突定义，或需要修改既有跨主题术语才能保持一致。

### 4.3：接入 buses 导航并同步计数

- Requirement/Scenario: R7；聚合完成、入口与基线计数冲突。
- Depends on: 4.1, 4.2。
- Targets: `docs/index.md` 的参考文档计数、已聚合主题正文、主题职责 buses 行和维护规则计数。
- Current behavior: index 显示 70 URL、40 术语、12 gaps；没有三篇 buses 正文入口；buses 状态仍为待聚合/deferred。
- Required behavior: 新增 I2C/USB、PCIe/CAN、EtherCAT 三篇入口；把 buses 状态改为三个已聚合 Iteration，并标注 G7/G13 open；同步所有计数为 70 URL、53 术语、13 gaps。
- Required changes: 描述只承担导航，不复制技术正文；全部相对链接有效；来源数量保持 70。
- Preserve: 其它主题入口、职责和状态；peripherals 继续 deferred；不修改 coverage、三篇正文或全局文档。
- Forbidden: 不宣称 MS10/change 已收尾；不把静态节点提升为运行可用；不执行 docs-maintainer 职责。
- Test witness: 修改前三篇 buses 链接均不在 index，buses 行含“待聚合”，计数为 70/40/12。
- GREEN condition: 三篇入口各出现 1 次，buses 行与正文完成状态一致，index 全部计数为 70/53/13，所有相对链接有效。
- Verification: 精确检查三篇链接和 buses 状态；独立统计 coverage URL、术语首列、gap 标题/状态表并与 index 比较；检查全部 index 相对链接；运行 `git diff --check` 与 strict validation。
- Stop when: 实际统计不等于 70/53/13，或需要修改三篇正文、coverage schema、SNAPSHOT/tasks 才能自洽。

**Invariants**

- M01-M04、D02、D04-D07 保持有效；一项信息只有一个权威位置。
- 三篇 buses 正文和五个 coverage 来源行不修改。
- G7 与 G13 均保持 `open`，G3/G4/G5 保持 `partial`。
- 不修改 `others/`、SNAPSHOT、全局 tasks、M/R/I 或既有主题正文。

**Non-goals**

不新增总线事实，不刷新来源，不运行硬件或软件栈，不修改 change proposal/design/spec，不归档或收尾 change，不创建 Evidence。

**Acceptance**

- A1 / R6 / task 4.1：三篇正文未知项映射到 G7/G13，G13 四字段和状态表完整，既有 gap 状态不变。
- A2 / R7 / task 4.2：13 个 buses 主写法唯一登记，术语总数为 53，定义与正文对象边界一致。
- A3 / R7 / task 4.3：index 提供三篇入口，buses 状态准确，70/53/13 计数与权威文档一致。
- A4 / R1/R6/R7 / tasks 4.1-4.3：相对链接、格式和 OpenSpec strict validation 通过，三篇正文、coverage 和范围外文件无变化。

**Verification**

- Task RED/GREEN：分别观察 G13 不存在、13 个术语缺失、index 无三入口且为 70/40/12；修改后检查目标状态。
- 计数：coverage URL=70、术语首列=53、gap 标题=13、gap 状态表=13，index 数字与之相等。
- 链接：检查三个修改文件内新增相对 Markdown 链接及 index 全部相对链接。
- 范围：产品 diff 仅包含 `known-gaps.md`、`terminology.md`、`index.md`；不含三篇 buses 正文、coverage、`others/` 或全局文档。
- 质量：BetterMd 禁止句式扫描、`git diff --check` 和 `openspec validate establish-k3-peripheral-bus-baseline --strict` 均退出 0。

**Gate 2 Readiness**

| Dimension | Status | Evidence |
| --- | --- | --- |
| Investigation | PASS | 已读取三个汇总目标和三篇 accepted 正文；当前计数、缺口映射、缺失术语、导航入口和验证表面已定位 |
| Design | PASS | 既有 D3/D6 与 G7/G13 分工闭合；URL 保持 70，术语和 gap 增量有唯一来源 |
| Iteration Plan | PASS | tasks 4.1-4.3 独占最终 Iteration，按缺口→术语→index 依赖排序，形成单一收敛结果 |
| Cycle Scope | PASS | 仅三个汇总文档和本 Cycle 反馈；无后续 tasks |
| Task Contracts | PASS | 三项均包含目标、当前/目标行为、保持/禁止、见证、GREEN、验证与停止条件 |
| Traceability | PASS | tasks 4.1-4.3 映射 R1/R6/R7、三目标文件和直接检查，无 Missing 或 Simplified |
| Verification | PASS | 直接检查计数、链接、状态、范围和 strict validation，无身份型证据工程 |
| User Plan Approval | PASS | 用户于 2026-09-12 回复“批准实施”，批准本 Iteration Plan/Cycle 并明确授权继续实施 |

**Persisted Evidence**

- Mode: none
- Act Response 足以保存可重跑命令、决定性输出、退出码和结论。
- Budget: 本 Cycle 最多 5 个文件（含 README），整个 change 最多 20 个 Evidence 文件；本计划不创建 Evidence。

**Risks and Notes**

- G13 是总线专属的聚合闭包；G7 继续只承担默认目标 DTS 未唯一映射，二者不能互相替代。
- `master`/`slave` 在本表中限定为 EtherCAT 对象，避免外推到通用主从命名。
- 若 Act 统计发现正文已使用且必须登记的额外主写法，属于术语集合变化，停止返回 Plan，不自行改变 53 项目标。

## Act Response

- Status: reported

**Implemented**

- Task 4.1: 在 G7 增加三篇 buses 正文的 DTS 依赖指针；新增 G13 汇总板级映射、软件运行路径和恢复闭包，状态 `open`，状态表同步为 13 项。
- Task 4.2: 增加 I2C、USB、DRD、role switch、Hub、PCIe、RC、EP、FlexCAN、CAN-FD、EtherCAT、master、slave 13 个主写法，术语总数由 40 增至 53。
- Task 4.3: 在 index 接入三篇 buses 正文，更新主题职责状态，并把计数同步为 70 个来源、53 个术语、13 个缺口。

**Changed Files and Symbols**

- `docs/reference/known-gaps.md`: G7、G13、状态汇总和 coverage 对应说明。
- `docs/reference/terminology.md`: 基础术语表新增 13 行。
- `docs/index.md`: 参考计数、buses 正文入口、主题职责和维护规则计数。
- `openspec/changes/establish-k3-peripheral-bus-baseline/tasks.md`: 勾选 tasks 4.1-4.3。
- 本 Cycle 文件：仅回填 Act Response；Plan Context 和 Plan Review 未修改。

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

逐任务 Spec review 均先于质量 review 完成。全量 diff 对照 A1-A4 复核：G1-G12 既有状态未漂移，G3/G4/G5 保持 `partial`，G7/G13 为 `open`；新增术语不覆盖既有跨主题职责；index 不声明 change 已收尾。三篇 buses 正文、coverage、`others/`、SNAPSHOT、全局 tasks 和 M/R/I 均未修改。

**Verification Evidence**

| 验证项 | 命令或操作 | 输出摘录 | 覆盖范围 | 结论 |
| --- | --- | --- | --- | --- |
| Task 4.1 RED | 检查 G13 不存在及 gap 标题数 | `T4.1 RED: G13 absent`，12 项；exit 0 | known-gaps 修改前状态 | PASS |
| Task 4.1 GREEN | 检查 gap 标题/状态、G7/G13、链接和既有状态 | `T4.1 GREEN gaps=13 status_rows=13 G3/G4/G5=partial G7/G13=open links=PASS`；exit 0 | A1、known-gaps | PASS |
| Task 4.2 RED | 统计术语并检查 13 个主写法缺失 | `T4.2 RED terms=40 missing=13`；exit 0 | terminology 修改前状态 | PASS |
| Task 4.2 GREEN | 统计首列、重复项和新增链接 | `T4.2 GREEN terms=53 new=13 duplicates=0 links=PASS`；exit 0 | A2、terminology | PASS |
| Task 4.3 RED | 检查三入口、buses 状态及计数 | `T4.3 RED links=absent buses=deferred counts=70/40/12`；exit 0 | index 修改前状态 | PASS |
| Task 4.3 GREEN | 独立统计来源/术语/gap，检查入口和全部 index 链接 | `T4.3 GREEN urls=70 terms=53 gaps=13 buses_links=3 all_index_links=PASS`；exit 0 | A3、index | PASS |
| 文档质量 | BetterMd 禁止句式扫描与全量 diff Review | 无匹配；exit 0 | 三个汇总文档 | PASS |
| 差异质量 | `git diff --check` | 无输出；exit 0 | 当前工作区 diff 格式 | PASS |
| OpenSpec | `openspec validate establish-k3-peripheral-bus-baseline --strict` | `Change 'establish-k3-peripheral-bus-baseline' is valid`；exit 0 | change 结构与规范 | PASS |

**Persisted Evidence**

None required.

**Experience Candidates**

None.

**Remaining Issues**

- G7 与 G13 保持 `open`；正文中的未确认板级映射、软件路径和恢复行为不阻塞本 Cycle 的导航收敛。
- 本 change 的 7/7 tasks 已勾选，但仍需独立 Plan Review；本 Act 不收尾或归档 change。

**Commit or Diff Reference**

None.

## Plan Review

- Review Result: accepted

**Findings**

- 阻塞 findings：None。
- 非阻塞 Minor findings：None。
- 独立审查确认 G7/G13 分工、13 个新增术语和 index 三入口均符合 A1-A4；没有改写三篇 buses 正文、coverage 或范围外产品文档。

**Deviation Classification**

None.

**Acceptance Gaps**

None.

**Convergence**

N/A.

**Evidence**

- 采信 Act Response 中未失效的 task 4.1-4.3 RED/GREEN：gap 12→13、术语 40→53、index 70/40/12→70/53/13，全部命令退出 0。
- 独立计数审计：coverage URL 70、术语首列 53、gap 标题 13、gap 状态表 13；13 个新增术语各精确出现 1 次且首列无重复。
- 独立状态审计：G3/G4/G5 保持 `partial`，G7/G13 为 `open`；G13 含四字段和三篇 buses 正文链接。
- 独立导航审计：index 三篇 buses 链接各出现 1 次且全部存在，index 所有相对 Markdown 链接有效；buses 状态与 Iterations 000-002 一致。
- 独立范围审计：本 Iteration 产品 diff 仅为 `known-gaps.md`、`terminology.md`、`index.md`；三篇 buses 正文、coverage、`others/`、SNAPSHOT、全局 tasks 和 M/R/I 未由本 Iteration 修改。
- 新鲜 Gate：`git diff --check`、`git diff --cached --check` 和 `openspec validate establish-k3-peripheral-bus-baseline --strict` 均退出 0；OpenSpec 输出 `Change 'establish-k3-peripheral-bus-baseline' is valid`。

**Follow-up Decision**

接受本 Cycle。A1-A4 均满足，无需当前 Cycle 修复、rework 或 replan；change 的 7/7 tasks 和全部 Iteration 已完成，可由 `openspec-docs-maintainer` 执行正常收尾。

**Iteration Plan Update**

None.

**Next Cycle**

None.

**Next Iteration**

None.
