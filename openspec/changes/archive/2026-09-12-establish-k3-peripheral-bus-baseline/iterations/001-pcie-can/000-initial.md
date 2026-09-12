# Iteration 001 / Cycle 000: PCIe 与 CAN 基线

## Plan Context

- Status: ready
- Iteration: 001-pcie-can
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: None

**Iteration Scope**

- Change tasks: 2.1
- Depends on: Iteration 000
- Stable baseline: PCIe 静态拓扑和 CAN 控制器、协议与物理层具备独立正文，并复用 USB/PCIe 共享 PHY 边界。
- Verification boundary: 正文覆盖 PCIe lane/PHY/RC/EP/插槽、AP/RP FlexCAN、CAN-FD 与板级冲突；静态能力和运行结果分离。
- Diagnostic boundary: PCIe 资源与枚举边界、USB/PCIe PHY 互斥、FlexCAN 实例和 CAN 收发器可用性。
- Deferred tasks: 3.1, 4.1-4.3

**Cycle Scope**

- Trigger: initial
- Acceptance gaps: None
- Repair items: None
- Inherited scope: proposal 已批准的 R1、R3、R4、R6 及 D1、D2、D4、D5；Iteration 000 已接受的来源和共享 PHY 基线。
- Excluded scope: EtherCAT 正文、最终术语/缺口/索引、默认 DTS 选择、真板 PCIe/CAN 验证和 source refresh。

**Objective**

建立 PCIe 与 CAN 正文，使读者能区分 SoC 能力、DTS 静态资源、板级接口及未经验证的枚举、通信和恢复行为。

**Background**

Iteration 000 已把 PCIe/CAN 官方入口调整为当前职责，并在 I2C/USB 正文记录 USB Port B/C/D 与 PCIe 的 SuperSpeed PHY 互斥。PCIe 和 CAN 的事实仍分散在 SoC、CoM260 板级资源及 DTS 摘要中。

**Current Baseline**

- `docs/buses/k3-pcie-and-can.md` 不存在。
- `docs/platform/k3-soc-overview.md` 记录 8 lanes PCIe Gen3、RC/EP、热插拔能力和 10 路 CAN，均为 SoC 能力而非板级运行结果。
- `docs/boot/com260-image-and-dts.md` 记录 CoM260 共享 DTS 的 `pcie0_rc` x4/PHY0+PHY1、`pcie3_rc`/PHY4、`pcie4_rc`/PHY5，以及基础款 `flexcan2` 和 Kit V02 的 `flexcan0..4`、`r_flexcan2` 静态状态。
- `docs/platform/com260-board-resources.md` 记录两个 M-Key NVMe 插槽、一个 E-Key Wi-Fi/BT 插槽，以及 CAN 连接器存在但功能接口表为“否”的 C2 冲突。
- PCIe/CAN Buildroot 正文仍未直接取得；源端修订保持 `unknown`。

**Current-State Evidence**

- PCIe 能力层：datasheet 只证明 Gen3、8 lanes、RC/EP 和热插拔能力，不提供 CoM260 端口模式或运行结果。
- PCIe DTS 层：已观察 CoM260 的 RC 节点与 PHY 组合；当前资料不足以唯一映射三个 controller 到三个 M.2 插槽，也没有 PERST#、CLKREQ#、link training、枚举或 MSI/MSI-X 结果。
- 共享资源层：Iteration 000 正文已记录 USB Port B/C/D SuperSpeed PHY 与 PCIe 互斥；本轮必须交叉引用，不能同时宣称共享 PHY 两侧可用。
- CAN 控制器层：基础款仅观察 `flexcan2` 启用、80 MHz 与 `can2_1_cfg`；Kit V02 观察 AP `flexcan0..4` 和 RP `r_flexcan2` 启用，不能互相代表。
- CAN 域边界：`k3-rdomain.dtsi` 中 RP FlexCAN 使用 SAPLIC；AP/RP 实例、clock/reset、pinctrl 和 IRQ 应分栏表达，不合并为单一实例数。
- CAN 物理层：Kit 资料同时写 4 Pin CAN FD 连接器/板载收发器存在和功能接口 `CAN=否`；必须保留冲突，不能解释为确定的软件禁用或硬件缺失。
- 测试入口：仓库无测试框架；使用目标文件缺失作为 RED，以 `rg`、shell 相对链接检查、行数、`git diff --check` 和 OpenSpec strict validation 验证。

**Relevant Code**

- `docs/buses/k3-i2c-and-usb.md`：共享 PHY 和静态/运行边界的已接受基线。
- `docs/platform/k3-soc-overview.md`：PCIe/CAN SoC 能力。
- `docs/platform/com260-board-resources.md`：M.2、CAN 连接器和 C2 冲突。
- `docs/boot/com260-image-and-dts.md`：CoM260 PCIe/FlexCAN DTS 摘要。
- `docs/interrupts/k3-interrupt-and-time.md`：AP/RP IRQ provider 边界。
- `docs/reference/source-coverage.md`：官方入口和 supporting source 唯一记录。

**Critical Path**

SoC 能力与官方 DTS → controller/lane/PHY 或 FlexCAN 资源 → CoM260 变体与连接器 → 静态/运行边界 → 后续导航与缺口收敛。

**Implementation Guidance**

先建立来源和证据分级，再分别整理 PCIe 与 CAN。PCIe 以 controller、lane/PHY、RC/EP、插槽和运行状态分层；CAN 以 AP/RP controller、CAN/CAN-FD 协议、pinctrl/clock/IRQ、收发器和连接器分层。无法唯一映射的字段写四字段未知项。

**Behavioral Change**

当前读者需要跨三篇文档拼接 PCIe/CAN 静态事实。完成后可从单一正文查询资源、变体、冲突和未验证行为；既有文档、外部接口和运行状态不变。

**Change Surface**

| Task | Requirement/Scenario | File/Symbol | Current Responsibility | Planned Change |
|---|---|---|---|---|
| 2.1 | R1/R3/R4/R6；静态映射、能力声明、控制器/收发器、板级冲突 | `docs/buses/k3-pcie-and-can.md` | 文件不存在 | 创建 PCIe/CAN 分层正文 |

**Task Contracts**

### 2.1：PCIe 与 CAN 静态链和运行边界可检索

- Requirement/Scenario: R1、R3、R4、R6；PCIe 静态端口映射/能力声明，CAN 控制器与收发器/板级资料冲突。
- Depends on: Iteration 000 accepted。
- Targets: 新建 `docs/buses/k3-pcie-and-can.md`。
- Current behavior: 目标文件不存在；能力、DTS 和板级事实分散，USB/PCIe 互斥只在 I2C/USB 正文中成文。
- Required behavior: 首行列全部直接来源；PCIe 覆盖 controller、lane/PHY、RC/EP、PERST#、CLKREQ#、M.2 插槽和预期用途；CAN 覆盖 AP/RP FlexCAN、CAN/CAN-FD、pinctrl/clock/IRQ、收发器和连接器；静态事实、冲突和运行未知项分开。
- Required changes: 交叉引用已接受的共享 PHY 边界；列出基础款与 Kit V02 的 PCIe/CAN 差异；每项技术结论使用规定证据等级；每个未知项包含当前证据、禁止推断、解除条件和影响主题。
- Preserve: G7 默认 DTS 未定边界、C2 原始冲突、官方事实与交叉验证分级、既有产品文档和 Iteration 000 正文。
- Forbidden: 不选择默认 DTS；不把 controller/PHY/插槽静态关系写成 link、枚举、NVMe、MSI/MSI-X、热插拔或 CAN 通信成功；不补写未观察的 Buildroot 命令；不更新术语、缺口或索引。
- Test witness: `test ! -e docs/buses/k3-pcie-and-can.md` 预期退出 0。
- GREEN condition: 文件存在且完整覆盖两类对象、共享 PHY、DTS 变体、C2 冲突和运行边界；相对链接有效，正文不超过 500 行。
- Verification: `rg` 检查首行、PCIe/CAN 必需术语、基础款/Kit V02 和四字段未知项；shell 提取相对链接并逐一 `test -e`；`git diff --check` 与 `openspec validate establish-k3-peripheral-bus-baseline --strict` 退出 0。
- Stop when: 新来源要求改变 requirement、PCIe/CAN 合篇边界、默认 DTS、共享 PHY结论或 C2 冲突解释，或正文合理压缩后仍超过 500 行。

**Invariants**

- M01-M04、D02、D04-D07 保持有效。
- SoC 能力、controller 节点、模组引出、Kit 连接和运行结果不互相替代。
- USB/PCIe 共享 PHY 不能被表述为两侧可同时工作。
- 不修改 `others/`、既有产品正文或全局 tasks/SNAPSHOT。

**Non-goals**

不执行 NVMe、PCIe link、CAN 收发、性能、热插拔或故障注入；不写 EtherCAT；不更新 terminology、known-gaps、index；不创建 Evidence。

**Acceptance**

- A1 / R1、R3：PCIe controller、lane/PHY、RC/EP、PERST#、CLKREQ#、插槽和预期用途分层，静态事实不升级为运行结论。
- A2 / R1、R4：AP/RP FlexCAN、CAN/CAN-FD、资源、收发器和连接器分层，基础款与 Kit V02 差异明确。
- A3 / R3、R4：USB/PCIe PHY 互斥和 CAN C2 冲突完整保留，未知项具有四个字段。
- A4 / R6：首行直接来源均可回到 coverage 唯一行，相对链接与行数检查通过，既有正文未修改。

**Verification**

- RED/GREEN：目标文件由不存在变为存在，必需对象和边界匹配。
- 来源：首行每个 URL 与 coverage URL 列精确匹配一行。
- 链接与结构：相对链接逐一存在；正文不超过 500 行；四字段未知项计数一致。
- 范围：产品 diff 仅新增目标正文；不包含 `others/` 或既有产品文档。
- 质量：`git diff --check` 和 OpenSpec strict validation 均退出 0。

**Gate 2 Readiness**

| Dimension | Status | Evidence |
|---|---|---|
| Investigation | PASS | Iteration 000 已接受来源和共享 PHY 基线；现有 SoC、板级及 DTS 摘要定位 PCIe/CAN 能力、变体、冲突和验证入口 |
| Design | PASS | D1、D2、D4、D5 已闭合合篇、不可达、共享 PHY 和静态/运行边界 |
| Iteration Plan | PASS | task 2.1 独占 Iteration 001，依赖 000，技术正文与后续 EtherCAT/汇总分离 |
| Cycle Scope | PASS | 本轮只创建 PCIe/CAN 正文；tasks 3.1、4.1-4.3 deferred |
| Task Contracts | PASS | 目标、当前/目标行为、保持/禁止、见证、GREEN、验证和停止条件完整 |
| Traceability | PASS | task 2.1 映射 R1/R3/R4/R6、对应场景、D1/D2/D4/D5、目标文件和直接检查 |
| Verification | PASS | 目标文件、来源、术语、冲突、链接、范围和 strict validation 可直接证明 Acceptance，无身份型证据工程 |

- Gate 2 plan approval: PASS — 用户于 2026-09-11 回复 `批准`，批准本 Plan/Cycle。

**Persisted Evidence**

- Mode: none
- Act Response 足以保存可重跑命令、决定性输出、退出码和结论。
- Budget: 本 Cycle 最多 5 个文件（含 README），整个 change 最多 20 个 Evidence 文件；本计划不创建 Evidence。

**Risks and Notes**

- PCIe/CAN Buildroot 页面不可达不阻塞静态基线，但禁止补写其命令和恢复语义。
- controller 到 M.2 插槽、PERST# 和 CLKREQ# 若无法由现有官方材料唯一确认，必须进入未知项。
- CoM260 基础款与 Kit V02 不能互相代表。

## Act Response

- Status: reported

**Implemented**

- Task 2.1: 新建 PCIe/CAN 正文，分层记录 PCIe controller、lane/PHY、RC/EP、侧带信号、M.2 插槽，以及 AP/RP FlexCAN、CAN/CAN-FD、资源、收发器和连接器。
- 保留 USB/PCIe PHY 互斥、基础款与 Kit V02 差异、CAN C2 冲突和未经验证的枚举、通信及恢复边界。

**Changed Files and Symbols**

- `docs/buses/k3-pcie-and-can.md`: 新增 PCIe/CAN 静态拓扑与运行边界正文。
- `openspec/changes/establish-k3-peripheral-bus-baseline/tasks.md`: 勾选 task 2.1。
- 本 Cycle 文件：仅回填 Act Response。

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

首轮 GREEN 定位到契约术语顺序不一致，将表头由 `clock/pinctrl/IRQ` 改为 `pinctrl/clock/IRQ` 后通过。负向声明扫描曾匹配“不能证明已收发”等否定句，修正检查表达式后确认无肯定式运行成功声明；产品内容无需因此调整。

**Verification Evidence**

- Gate 3 RED: `test ! -e docs/buses/k3-pcie-and-can.md` 退出 0。
- Task 2.1 GREEN: 首行 11 个直接 URL 均与 coverage URL 列精确匹配 1 行；正文 123 行；18 个必需术语、4 个相对链接和五项四字段未知项检查通过，退出 0。
- Gate 4: 规格审查与文档质量审查通过；共享 PHY、C2、默认 DTS 和静态/运行边界均保留；肯定式未验证成功声明为 0。
- Gate 5: `git diff --check` 退出 0；`openspec validate establish-k3-peripheral-bus-baseline --strict` 退出 0，输出 `Change 'establish-k3-peripheral-bus-baseline' is valid`；来源、行数、未知项、链接、任务状态和 Evidence 模式检查均通过。

**Persisted Evidence**

None required.

**Experience Candidates**

None.

**Remaining Issues**

- PCIe controller/PHY 到 M.2 插槽、PERST#/CLKREQ#、完整 AP CAN 资源、CAN C2 冲突及 Buildroot 页面内容仍为正文明确记录的未知项。

**Commit or Diff Reference**

None.

## Plan Review

- Review Result: accepted

**Findings**

- 阻塞 findings：None。
- 非阻塞 Minor findings：None。
- 独立正文与差异审查确认 PCIe/CAN 对象分层完整；共享 PHY、默认 DTS、CAN C2 和静态/运行边界均未被裁决或提升。

**Deviation Classification**

None.

**Acceptance Gaps**

None.

**Convergence**

N/A.

**Evidence**

- 采信 Act Response 中未失效的 Gate 3 RED、术语修正和 Gate 4 结论。
- 独立来源审计：正文首行 11 个 URL 与 coverage URL 列精确比较，全部各匹配 1 行。
- 独立内容审计：正文 123 行、4 个相对链接有效、五项未知项的四字段计数一致；未发现肯定式的枚举、训练、通信、NVMe、MSI 或热插拔成功声明。
- 独立范围审计：`known-gaps.md`、`terminology.md`、`index.md` 无 diff；`others/` 未被本 Cycle 修改。
- 新鲜 Gate：`git diff --check` 和 `openspec validate establish-k3-peripheral-bus-baseline --strict` 均退出 0。

**Follow-up Decision**

接受本 Cycle。A1-A4 均满足，无需当前 Cycle 修复或后继 Cycle；按既有 Iteration Map 展开 Iteration 002 计划草稿，等待 Gate 2 批准。

**Iteration Plan Update**

None.

**Next Cycle**

None.

**Next Iteration**

`../002-ethercat/000-initial.md`
