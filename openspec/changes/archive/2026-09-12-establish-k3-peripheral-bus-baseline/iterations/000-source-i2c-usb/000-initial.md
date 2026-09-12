# Iteration 000 / Cycle 000: 来源与 I2C/USB 基线

## Plan Context

- Status: ready
- Iteration: 000-source-i2c-usb
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: None

**Iteration Scope**

- Change tasks: 1.1, 1.2
- Depends on: None
- Stable baseline: 五个 MS10 来源具有当前职责，I2C 与 USB 的控制器、设备、PHY、角色和板级关系有独立正文。
- Verification boundary: 覆盖表五行状态和 URL 唯一性正确；`k3-i2c-and-usb.md` 覆盖资源分层、数量冲突、共享 PHY、变体及未知项；相对链接有效。
- Diagnostic boundary: 来源状态、I2C controller/client、USB controller/PHY/role/Hub、I²C 数量冲突和 USB/PCIe PHY 互斥。
- Deferred tasks: 2.1, 3.1, 4.1-4.3

**Cycle Scope**

- Trigger: initial
- Acceptance gaps: None
- Repair items: None
- Inherited scope: proposal 已批准的全部 requirement、场景、假设和 Non-goals；本轮只执行 tasks 1.1、1.2。
- Excluded scope: PCIe/CAN/EtherCAT 正文、最终术语/缺口/索引、真板验证、source refresh、既有主题改写。

**Objective**

建立 MS10 来源责任和 I2C/USB 正文，使读者能区分控制器、client、PHY、Host/DRD/role/Hub、板级变体与未验证行为。

**Background**

`docs/buses/` 尚不存在，五个来源仍为 future/deferred。K3 datasheet 与官方 DTS 已提供可审计静态事实，但 Buildroot 五个操作入口及 USB 子树当前不可直接读取。Gate 1 已由用户以 `批准` 通过。

**Current Baseline**

- Revision: `c2d4387d6463825e7d586a0d2fa96040e32dfc54`；branch: `main`。
- `docs/index.md:68` 把 `docs/buses/` 标为待聚合。
- `docs/reference/source-coverage.md:48,51-53,55` 的五行均为 `future / deferred / unknown / 2026-09-02 / partially-observed`。
- `docs/platform/k3-soc-overview.md:99,103` 记录 USB 能力与 9 路 I²C；当前官方英文 K3 datasheet 记录 USB Port B/C/D SuperSpeed PHY 与 PCIe 互斥，并写“最多 10 路 I²C”，形成待保留冲突。
- `docs/boot/com260-image-and-dts.md:77-78,91-95,116` 已记录 CoM260 USB、I2C client、PCIe 和 EtherCAT 静态摘要。
- 基线命令：`test ! -e docs/buses/k3-i2c-and-usb.md` 退出 0；`openspec validate establish-k3-peripheral-bus-baseline --strict` 退出 0。

**Current-State Evidence**

- I2C 控制器链：官方 `k3.dtsi` 的 aliases 可见 `i2c0..i2c6` 与 `i2c8`；节点使用 `spacemit,k1-i2c`，含 MMIO、APLIC IRQ、func/bus clocks、reset、400 kHz 默认频率和 disabled 初态。`k3-rdomain.dtsi` 另有 `r_i2c0/1`，同为 disabled。
- I2C client：`k3_com260.dtsi` 启用 `i2c1/2`，`i2c2` 挂只读 `atmel,24c02@50`；既有 DTS 摘要还记录 `i2c0` 的 Type-C 控制器、`i2c3` 的显示/触控及基础款 `i2c5` 摄像头。
- USB 链：K3 datasheet 区分 USB 2.0 Host、Port A USB 3.0 DRD 和 Port B/C/D USB 3.0 Host；后者 SuperSpeed PHY 与 PCIe 共享且只能择一。USB controller 具有集成 DMA、中断和 suspend 能力，但当前没有 K3 运行路径证据。
- CoM260：共享 DTS 启用 Port A U2/U3 PHY、DRD/OTG role switch 与 Port B U2/U3 PHY；基础款通过 FUSB301 建立 Type-C orientation/role endpoint，Port B 挂 VL817 USB2/USB3 Hub；Kit V02 删除 `monitor-vbus` 且 Type-C 节点结构不同。
- 板级接口：既有板级文档记录四个 USB Type-A 和一个仅下载、不供电的 Type-C；40 Pin、显示、摄像头和 EEPROM 使用不同 I2C 链路。
- 来源错误路径：本地 `curl` 访问 raw GitHub 因 DNS 失败（exit 6），网页缓存也不能打开五个 Buildroot 页面。Act 不得把不可访问解释为页面不存在。
- 状态变化：本轮只把覆盖表聚合职责从 future/deferred 调整为当前 active，并创建正文；访问状态、源端修订和未展开信息按实际证据保留。
- 测试入口：仓库无测试框架；使用 `rg`/`awk`/shell 检查内容与链接，使用 `git diff --check` 和 OpenSpec strict validation。

**Relevant Code**

- `docs/reference/source-coverage.md`：URL 唯一来源责任表。
- `docs/reference/document-template.md`：首行来源和证据等级格式。
- `docs/platform/k3-soc-overview.md`：SoC 数量与能力基线。
- `docs/platform/com260-board-resources.md`：Kit 接口和物理连接。
- `docs/boot/com260-image-and-dts.md`：CoM260 DTS 变体摘要。
- `docs/platform/k3-platform-control.md`、`docs/interrupts/k3-interrupt-and-time.md`、`docs/dma/k3-dma-and-memory-ownership.md`：资源、IRQ、DMA 交叉引用目标。

**Critical Path**

官方入口与 supporting source → 覆盖表职责/可达状态 → I2C/USB 控制器和设备链 → 证据等级与未知项 → 后续 PCIe/CAN Iteration 使用共享 PHY 基线。本轮不改变外部系统或运行状态。

**Implementation Guidance**

先完成 task 1.1，再以覆盖表 URL 生成正文首行。正文按“范围与证据 → I2C 实例/client/冲突 → USB controller/PHY/role/Hub → 板级变体 → 错误与未知项 → 导航”组织。若能直接读取 Buildroot 页面，只补与当前契约一致且有来源的事实；出现会改变 requirement、拆分或验收的内容时停止返回 Plan。

**Behavioral Change**

当前读者只能从索引看到待聚合入口和散落事实。完成后五个来源属于当前 MS10 聚合职责，I2C/USB 有唯一正文入口；不可达、数量冲突、共享 PHY 和未验证运行状态均显式可见。既有文档、运行接口和外部状态不变。

**Change Surface**

| Task | Requirement/Scenario | File/Symbol | Current Responsibility | Planned Change |
|---|---|---|---|---|
| 1.1 | R1/R6；来源可读/不可达 | `docs/reference/source-coverage.md` 五个 URL 行 | future/deferred 入口 | 调整为当前 buses/active 职责并保留实际可达边界 |
| 1.2 | R1/R2；资源分层/复合依赖/子树不可见 | `docs/buses/k3-i2c-and-usb.md` | 文件不存在 | 创建 I2C/USB 分层正文 |

**Task Contracts**

### 1.1：五个来源具有当前职责

- Requirement/Scenario: R1 资源与板级分层；R6 来源可追溯；官方页面可读/不可访问。
- Depends on: None
- Targets: `docs/reference/source-coverage.md` 中 06-I2C、10-USB、11-PCIe、15-CAN、22-EtherCAT 五行。
- Current behavior: 五行均为 `future / deferred`，备注为“非当前目标”。
- Required behavior: 五行指向当前 MS10/buses 职责并标为 active；源端修订、观察日期、访问状态只按实际观察更新；USB 目录未展开必须保留。
- Required changes: 删除“非当前目标”语义；没有直接读取的新 URL 不新增 supporting row；每个 URL 仍恰好一行。
- Preserve: D04 URL 唯一键、M01 权威源、其它 65 行内容和既有 staged/untracked 文件。
- Forbidden: 不把不可达写成 removed；不伪造修订日期；不修改主题正文或全局计数。
- Test witness: `rg -n '06-I2C|10-USB|11-PCIe|15-CAN|22-EtherCAT' docs/reference/source-coverage.md` 当前显示五行 future/deferred。
- GREEN condition: 五行均为 buses/current/active 语义，状态字段与实际观察一致，URL 唯一。
- Verification: 对五个 URL 分别 `rg -F -c` 得 1；内容扫描无 `future | deferred | 非当前目标`；`git diff --check` 退出 0。
- Stop when: 页面内容表明 URL 已移动/删除，或需要改变来源 schema、M01/D04/D07。

### 1.2：I2C/USB 对象链和边界可检索

- Requirement/Scenario: R1/R2；资源和连接、映射不唯一、复合设备依赖、USB 子树不可观察。
- Depends on: 1.1
- Targets: 新建 `docs/buses/k3-i2c-and-usb.md`。
- Current behavior: 文件不存在；事实分散在 platform/boot 文档。
- Required behavior: 首行列全部直接来源；正文覆盖 I2C AP/RP 实例与 client、9/10 数量冲突、USB Host/DRD/controller/PHY/role/Hub、USB/PCIe PHY 互斥、CoM260 变体、运行边界及四字段未知项。
- Required changes: 每项技术结论标注证据等级；引用既有平台/启动/中断/DMA 文档，不复制其完整职责；至少列出基础款与 Kit V02 差异。
- Preserve: 既有文档内容、G7 默认 DTS 边界、官方事实与交叉验证分级。
- Forbidden: 不裁决 I²C 数量；不声称设备已枚举、DMA/IRQ 已运行；不写未观察的 Buildroot 命令；不创建占位文档或验证脚本。
- Test witness: `test ! -e docs/buses/k3-i2c-and-usb.md` 当前退出 0。
- GREEN condition: 文件存在且包含规定对象、冲突、共享 PHY、变体、运行边界；每个未知项含 `当前证据/禁止推断/解除条件/影响主题`；相对链接均可达。
- Verification: `rg` 检查首行和必需术语；shell 提取 Markdown 相对链接并逐一 `test -e`；`wc -l` 不超过 500；`git diff --check` 退出 0。
- Stop when: 新来源要求改变三篇拆分、裁决全局 source refresh、选择默认 DTS，或正文在合理压缩后仍超过 500 行。

**Invariants**

- M01-M04、D02、D04-D07 保持有效。
- 技术结论只使用官方事实、交叉验证、推论、未知项四级。
- SoC 能力、控制器节点、模组引出、Kit 连接和运行结果不互相替代。
- 不修改 `others/`、既有 MS03-MS09 产品文档或全局 tasks/SNAPSHOT。

**Non-goals**

不写 PCIe/CAN/EtherCAT 正文；不更新 terminology、known-gaps、index；不执行真板、网络、I/O、性能或故障注入；不创建 Evidence。

**Acceptance**

- A1 / R6 / task 1.1：五个 URL 各一行并承担当前 MS10 职责；不可达和 USB 未展开状态真实保留。
- A2 / R1-R2 / task 1.2：I2C/USB 正文具备完整对象链、板级变体、证据分级和未知项。
- A3 / R2 / task 1.2：9/10 I²C 冲突与 USB/PCIe PHY 互斥均明确，不发生静默裁决。
- A4 / R1-R2 / tasks 1.1-1.2：所有直接来源、相对链接和行数约束通过验证；既有文档未被修改。

**Verification**

- 目标 RED/GREEN：目标文件从不存在变为存在且必需内容匹配。
- URL 唯一性：五个完整 URL 的出现次数各为 1。
- 链接：提取本轮新增正文的相对 Markdown 链接，逐一确认文件存在。
- 边界：`git diff --name-only` 仅包含 coverage、新正文和当前 change 内 Act Response；产品 diff 不包含既有主题或 `others/`。
- 质量：`git diff --check` 和 `openspec validate establish-k3-peripheral-bus-baseline --strict` 均退出 0。

**Gate 2 Readiness**

| Dimension | Status | Evidence |
|---|---|---|
| Investigation | PASS | Explorer 结论经 revision 检查仍适用；Plan 补查官方 datasheet、`k3.dtsi`、`k3-rdomain.dtsi` 和 CoM260 DTS；入口、状态、错误边界和验证入口已定位 |
| Design | PASS | design D1-D6 闭合文档拆分、不可达、数量冲突、共享 PHY、静态/运行边界和验证策略 |
| Iteration Plan | PASS | tasks 1.1-4.3 唯一分配到 000-003，依赖、稳定基线、验证/诊断边界和平衡审计完整 |
| Cycle Scope | PASS | 本轮仅 tasks 1.1、1.2；后续任务明确 deferred |
| Task Contracts | PASS | 两个任务均含目标、当前/目标行为、保持/禁止、见证、GREEN、验证和停止条件 |
| Traceability | PASS | tasks RTM 无 Missing 或 Simplified；本轮 Acceptance 映射 R1/R2/R6 |
| Verification | PASS | 直接检查正文、URL、链接、diff 和 strict validation；无身份型证据工程 |

- Gate 2 plan approval: PASS — 用户于 2026-09-11 回复 `批准`，批准本 Plan/Cycle。

**Persisted Evidence**

- Mode: none
- Act Response 足以保存可重跑命令、决定性输出、退出码和结论。
- Budget: 本 Cycle 最多 5 个文件（含 README），整个 change 最多 20 个 Evidence 文件；本计划不创建 Evidence。

**Risks and Notes**

- Buildroot 页面不可达是已设计的内容边界，不阻塞 tasks 1.1/1.2；页面若恢复且出现契约冲突则停止返回 Plan。
- I²C 9/10 数量差异是显式冲突，不由 Act 裁决。
- 非实质措辞、表格布局和章节标题可由 Act 在契约内决定。

## Act Response

- Status: reported

**Implemented**

- Task 1.1: 将 06-I2C、10-USB、11-PCIe、15-CAN、22-EtherCAT 五个官方入口调整为 `buses / current / active`，保留源端修订 `unknown`、USB 子树未展开和访问失败边界。
- Task 1.2: 新建 I2C/USB 正文，覆盖 AP/RP I2C 实例与 client、9/10 数量冲突、USB controller/PHY/Host/DRD/role switch/Hub、USB/PCIe 共享 PHY、CoM260 变体、运行边界及五项四字段未知项。

**Changed Files and Symbols**

- `docs/reference/source-coverage.md`: 五个 MS10 来源行。
- `docs/buses/k3-i2c-and-usb.md`: 新增完整主题正文。
- `openspec/changes/establish-k3-peripheral-bus-baseline/tasks.md`: 勾选 tasks 1.1、1.2。
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

**Verification Evidence**

- Gate 3 RED: 五个覆盖行原为 `future / deferred`；`test ! -e docs/buses/k3-i2c-and-usb.md` 退出 0。
- Task 1.1 GREEN: 五个完整 URL 的 `rg -F -c` 均为 1；对应行均匹配 `buses / current / active`，且不含 `future`、`deferred` 或“非当前目标”。
- Task 1.2 GREEN: 正文 127 行；首行来源、必需对象、9/10 冲突、共享 PHY、变体和运行边界断言通过；五个未知项的四个字段计数均为 5；相对链接检查通过。
- Gate 4: 规格符合性与文档质量审查均通过；未发现 Critical、Important 或 Minor 问题。
- Gate 5: `git diff --check` 退出 0；`openspec validate establish-k3-peripheral-bus-baseline --strict` 退出 0，输出 `Change 'establish-k3-peripheral-bus-baseline' is valid`。

**Persisted Evidence**

None required.

**Experience Candidates**

None.

**Remaining Issues**

- Iterations 001-003 仍按计划 deferred；Buildroot I2C 正文与 USB 子树仍不可直接观察。

**Commit or Diff Reference**

None.

## Plan Review

- Review Result: accepted

**Findings**

- 阻塞 findings：None。
- 非阻塞 Minor findings：None。
- 独立 diff 审查确认产品范围仅包含五个 coverage 行和新增 I2C/USB 正文；`known-gaps.md`、`terminology.md`、`index.md` 与 `others/` 未被本 Cycle 修改。

**Deviation Classification**

None.

**Acceptance Gaps**

None.

**Convergence**

N/A.

**Evidence**

- 采信 Act Response 中未失效的 Gate 3 RED 与 Gate 4 逐任务审查结论。
- 独立来源审计：从正文首行提取 9 个 URL，与 `source-coverage.md` URL 列精确比较，全部各匹配 1 行，退出 0。
- 独立结构审计：五类未知项的 `当前证据/禁止推断/解除条件/影响主题` 均计数 5；相对链接逐一存在，退出 0。
- 独立范围审计：既有汇总文档无 diff；正文保留 I²C 9/10 冲突、USB/PCIe PHY 互斥及基础款/Kit V02 边界。
- 新鲜 Gate：`git diff --check` 退出 0；`openspec validate establish-k3-peripheral-bus-baseline --strict` 退出 0，change valid。

**Follow-up Decision**

接受本 Cycle。A1-A4 均满足，无需当前 Cycle 修复或后继 Cycle；按既有 Iteration Map 展开 Iteration 001 计划草稿，等待 Gate 2 批准。

**Iteration Plan Update**

None.

**Next Cycle**

None.

**Next Iteration**

`../001-pcie-can/000-initial.md`
