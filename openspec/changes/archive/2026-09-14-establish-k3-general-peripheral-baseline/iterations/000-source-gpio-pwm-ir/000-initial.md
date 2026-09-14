# Iteration 000 / Cycle 000: 来源与 GPIO/PWM/IR-RX 基线

## Plan Context

- Status: ready
- Iteration: 000-source-gpio-pwm-ir
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: None

**Iteration Scope**

- Change tasks: 1.1, 1.2
- Depends on: None
- Stable baseline: 六个 MS11 来源具有当前职责，GPIO、PWM 与 IR-RX 的控制器、复用、IRQ、consumer 和板级边界有独立正文。
- Verification boundary: 覆盖表六行状态和 URL 唯一性正确；`k3-gpio-pwm-ir.md` 覆盖对象分层、PWM 冲突、CoM260 复用候选和四字段未知项；相对链接有效。
- Diagnostic boundary: 来源状态、GPIO controller/pinctrl/IRQ、PWM channel/pinmux/consumer、IR-RX controller/input 和板级映射。
- Deferred tasks: 2.1, 3.1, 4.1-4.3

**Cycle Scope**

- Trigger: initial
- Acceptance gaps: None
- Repair items: None
- Inherited scope: proposal 中已批准的 MS11 范围；delta spec R1-R4、R7；M01-M04；G7；R04–R06、R08。
- Excluded scope: Audio 和 WDT/RTC 正文；最终术语、缺口、索引与 SNAPSHOT/tasks；产品代码或真板操作；PWM 数量裁决。

**Objective**

把六个 MS11 官方入口从未来职责调整为当前通用外设职责，并创建一篇可追溯的 GPIO/PWM/IR-RX 正文；正文必须区分控制器、复用、IRQ、consumer、板级候选和运行证据，保留 PWM 数量冲突及所有未确认边界。

**Background**

`docs/index.md` 当前把 `docs/peripherals/` 标为待聚合，六个目标 URL 在 `source-coverage.md` 中均为 `future / deferred / unknown / partially-observed`，目标正文不存在。用户已批准三篇正文拆分和不执行真板验证的范围。

**Investigation Facts**

- Current Baseline: `main` at `d8b71f1`；工作区既有未跟踪 `others/`，本 change 不读取其内容作为官方 K3 产品事实，也不得修改。`docs/peripherals/` 三个目标文件均不存在；覆盖表六行仍为 deferred；索引只有待聚合目录行。
- Current-State Evidence:
  - K3 datasheet `docs-chip/en/key_stone/k3/k3_docs/k3_ds.md` 在 2026-09-12 直接观察：概览列 30 路 PWM；详细 PWM 章节列 20 个独立通道 PWM0–PWM19、195.3 Hz–12.8 MHz、6-bit divider、10-bit period counter、15-bit pulse counter；GPIO 章节列输入中断、复位默认输入、独立置位/清除/读取和双边沿能力。
  - 官方 `linux-6.18` `k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi` 在 2026-09-12 直接观察：`gpio@d4019000` 使用 `spacemit,k3-gpio`，base `0xd4019000/0x100`，core/bus clocks，APLIC source 58，兼作 GPIO 与 interrupt controller，`gpio-ranges` 覆盖 4×32；pinctrl 是独立节点，APLIC source 60。
  - 同一 DTS 提供 PWM0–PWM19；每个节点使用 `spacemit,k1-pwm` 与 `marvell,pxa910-pwm`，包含独立 base、func/bus clocks、reset、3-cell specifier，默认 `disabled`；未观察到 `pwm20`。
  - 同一 DTS 提供 `ircrx0@d4017e00`（APLIC source 69）与 `ircrx1@d4017f00`（source 20），均使用 `spacemit-k1,irc`、102.4 MHz clock、reset，默认 `disabled`。
  - 官方 CoM260 datasheet `docs-product/zh/k3_com260/com260_ds.md` 在 2026-09-12 直接观察：金手指表包含多组 PWM/IR/GPIO 复用候选与电压域，并单列 FAN_PWM；这些只证明 pinmux/引出候选，不证明目标 DTS 已选择或通道运行。
  - 六个 `docs-buildroot` 对应页及 raw URL 在 2026-09-12 均未取得正文：Web 返回 cache miss，shell 网络返回 `Could not resolve host: raw.githubusercontent.com`。该失败只确定当前访问边界，不刷新既有 2026-09-02 观察日期。
- Code and Critical Path:
  - `docs/reference/source-coverage.md` 六个既有 URL 行是来源状态唯一写入点；只改 `Scope/Topic`、`Horizon`、`Aggregation` 和 Notes 中的 MS11 职责，不改变 URL、源端修订或既有观察日期。
  - `docs/peripherals/k3-gpio-pwm-ir.md` 是本 Iteration 唯一新正文；首行列六个官方入口以及实际使用的 supporting URL，并标明源端修订/观察日期/证据等级。
  - 正文路径依次为：范围和证据等级 → GPIO controller/pinctrl/IRQ → PWM channel/pinmux/consumer 与冲突 → IR-RX controller/input → CoM260 候选映射 → 错误边界和四字段未知项 → 相关主题导航。

**Implementation Guidance**

先精准更新覆盖表六行，使其指向 `peripherals / current / active`，Notes 明确正文未直接取得和当前 MS11 职责；保持 URL 总数不变。再创建正文，以表格集中表达静态资源，以分段说明责任方向和禁止推论。冲突使用“来源—观察—不能推出—解除条件”结构，不把 30 或 20 写成统一事实。

**Behavioral Change**

当前六个 URL 只有未来入口，读者无法从正文查询 GPIO/PWM/IR-RX。完成后六个入口属于当前 MS11，正文提供静态资源与板级边界；不会新增运行能力、改变既有硬件事实或裁决来源冲突。

**Task Contracts**

### 1.1: 激活六个 MS11 来源职责

- Requirement/Scenario: R1 资源与板级分层；R7 来源可追溯；S6 来源与导航收敛。
- Depends on: None
- Targets: `docs/reference/source-coverage.md` 中 02-GPIO、03-PWM、04-IR-RX、17-Audio、23-WDT、24-RTC 六行。
- Current behavior: 六行均为 `peripherals | future | deferred | unknown | 2026-09-02 | partially-observed`，Notes 写“非当前目标”。
- Required behavior: 六行改为 MS11 当前职责和 active 聚合状态；保留唯一 URL、`unknown`、2026-09-02 和 `partially-observed`；Notes 说明正文未直接取得及 supporting evidence 边界。覆盖表 URL 总数保持 70。
- Required changes: 只精准修改六行的职责、horizon、aggregation 和 Notes；Audio/WDT/RTC 虽在后续 Iteration 成文，也在当前 change 获批后同时成为 current scope。
- Preserve: 表头、字段语义、其余 64 行、既有观察日期和 URL 唯一键；MS10 及更早主题状态。
- Forbidden: 新增重复 URL；把访问失败写成观察成功；刷新观察日期；修改 R06、SNAPSHOT、全局 tasks 或产品正文。
- Test witness: 修改前运行 `rg -n '\| .*peripherals \| future \| deferred' docs/reference/source-coverage.md`，预期命中六行；运行 URL 唯一检查，预期当前 70 行无重复。
- GREEN condition: 六个目标 URL 各出现一次且均为 `peripherals | current | active`；`unknown | 2026-09-02 | partially-observed` 保持；其他 URL 行无变化。
- Verification: `rg -n '02-GPIO|03-PWM|04-IR-RX|17-Audio|23-WDT|24-RTC' docs/reference/source-coverage.md` 检查六行字段；用 `awk -F'\\|'` 提取 HTTP URL 后 `sort | uniq -d`，预期无输出；URL 总数仍为 70。
- Stop when: 实际页面可读内容要求改写 requirement、来源分类或观察日期，或覆盖表已有并行改动触及同六行。

### 1.2: 建立 GPIO/PWM/IR-RX 正文

- Requirement/Scenario: R1-R4；S1-S3。
- Depends on: 1.1
- Targets: 新文件 `docs/peripherals/k3-gpio-pwm-ir.md`。
- Current behavior: 文件和 `docs/peripherals/` 目录不存在；GPIO/PWM/IR-RX 只有来源表、SoC/CoM260 散落事实。
- Required behavior: 正文按控制器、pinctrl、IRQ、channel、consumer、input 和板级候选分层；完整保留 PWM 30/20/PWM0–19 冲突；明确 IR-RX 两节点 disabled；每项未知内容使用“当前证据、未知内容、禁止推论、解除条件”四字段。
- Required changes: 创建主题文档；首行包含官方入口、实际 supporting sources、源端修订和观察日期；加入目录、资源表、证据等级、CoM260 候选和相关主题相对链接。
- Preserve: M01-M04、G7、既有 pinctrl/interrupt/platform 文档的权威职责；简体中文和既有四级证据表达；不复制整份引脚表。
- Forbidden: 修改既有产品正文；声称具体 GPIO 安全可用、PWM 波形已验证或 IR 遥控可用；裁决 PWM 数量；给出会改变真板状态的操作步骤；引用 K3 外板卡事实。
- Test witness: 修改前 `test ! -e docs/peripherals/k3-gpio-pwm-ir.md` 返回 0；修改前索引无正文入口。该 RED 证明目标知识入口缺失。
- GREEN condition: 文件存在且首行为 `> 来源:`；包含 GPIO controller/pinctrl/IRQ、PWM 三种数量观察、IR-RX 两节点、CoM260 候选、证据边界和至少一组完整四字段未知项；所有相对链接指向存在文件。
- Verification: `rg` 检查必需术语和禁止越界表述；`awk`/`sed` 检查首行；解析正文相对 Markdown 链接并逐项 `test -e`；`git diff --check -- docs/reference/source-coverage.md docs/peripherals/k3-gpio-pwm-ir.md` 退出 0。
- Stop when: 新来源证明三篇拆分不再适用、要求改写既有行为规格，或无法在不选择默认 CoM260 DTS 的情况下表达板级事实。

**Invariants**

- 官方社区入口仍是权威入口；GitHub 官方仓库只作 supporting evidence。
- URL 是覆盖表唯一键；本 Iteration 不改变 70 个唯一 URL 总数。
- SoC 能力、DTS 静态节点、CoM260 引出和真板运行四层不得互相替代。
- `status = "disabled"` 不等于硬件不存在，也不等于目标板可以直接启用。
- PWM 30/20/PWM0–19 冲突在本 change 内不得被静默裁决。
- 不修改 `others/`、既有产品正文、SNAPSHOT、全局 tasks、M/R/I 或归档 change。

**Non-goals**

- 不创建 Audio、WDT/RTC、术语、缺口或索引最终内容。
- 不下载、构建或运行 Linux/Buildroot，不操作开发板。
- 不新增验证脚本、依赖、身份字段或 Evidence 目录。

**Acceptance**

- A1 / R1,R7 / S6 / D2 / T1.1: 六个目标 URL 唯一且为 current/active，访问和修订边界保持准确，URL 总数仍为 70。
- A2 / R1,R2 / S1 / D4 / T1.2: 正文区分 GPIO controller、pinctrl、IRQ、consumer 与 CoM260 候选，不声明具体引脚安全可用。
- A3 / R1,R3 / S2 / D3,D4 / T1.2: 正文并列 30、20 和 DTS PWM0–19 三种观察，记录 channel/clock/reset/pinmux/consumer，不裁决冲突或声明波形成功。
- A4 / R1,R4 / S3 / D4 / T1.2: 正文记录两个 IR-RX 静态节点、IRQ/clock/reset 与 disabled 状态，并把协议、keymap、输入事件和板级映射列为未知。
- A5 / R1-R4,R7 / S1-S3,S6 / D7 / T1.1,T1.2: 首行来源、证据等级、四字段未知项、相对链接和 Markdown diff 检查全部通过。

**Verification**

- 直接检查六个来源行的字段、唯一性和总数；重复 URL、错误状态或日期变化即失败。
- 直接检查正文首行、章节、静态资源、冲突、未知项和禁止推论；缺项或越界陈述即失败。
- 逐项验证相对链接目标存在；任何断链即失败。
- 运行 `git diff --check` 和 `openspec validate establish-k3-general-peripheral-baseline --strict`；非零退出即失败。
- 不用 commit、revision、run-id 或日志文件代替上述内容检查。

**Gate 2 Readiness**

- Requirement coverage: PASS — delta spec R1-R4、R7 映射到 S1-S3/S6、D2-D4/D7、T1.1-T1.2 和 A1-A5。
- Simplification approval: PASS — 无需求简化；三篇拆分和未知项边界已由用户批准。
- Investigation completeness: PASS — 已直接检查覆盖表、索引、目标文件存在性、K3 datasheet、K3 DTS 和 CoM260 产品资料；调用链在纯文档仓库中对应“来源 → 覆盖行 → 主题正文 → 后续索引”。
- Design closure: PASS — 来源责任、对象边界、冲突处理、错误语义和禁止推论已确定。
- Task executability: PASS — 两个 task 均有目标位置、当前/目标行为、测试见证、GREEN、验证和停止条件。
- Iteration balance: PASS — 来源与首篇正文共同形成低速 I/O 稳定基线；Audio、WDT/RTC 和汇总各自保留独立故障域。
- Traceability: PASS — tasks.md RTM 与本 Cycle A1-A5 形成 requirement 到验证链。
- Verification sufficiency: PASS — 文件缺失/deferred 为 RED，内容、链接、URL 和严格校验直接证明目标行为。
- Identity-evidence exclusion: PASS — 未规划 Hash、revision pin、run-id、manifest 或专用验证工具。
- Material unknowns: PASS — PWM 冲突和板级/运行缺口已转为契约化边界，不要求 Act 决定语义。
- Artifact consistency: PASS — proposal、spec、design、tasks 和当前 Cycle 使用同一范围、文件拆分和任务映射。
- Persisted Evidence: PASS — Mode `none`，结果可低成本重跑并由 Act Response 承载。
- User plan approval: PASS — 用户指令 `批准`（2026-09-12）；当前计划获准交给 `openspec-act`。

**Persisted Evidence**

- Mode: none

Act Response 足以记录命令、决定性输出、退出码和修改表面；所有检查可低成本重跑，不满足创建持久 Evidence 的必要性门槛。

- Budget: 按 Evidence 预算执行（公共规则 › 验证）。

**Risks and Notes**

- Buildroot 页面当前不可达是内容边界，不阻塞静态基线；若实施时页面恢复且产生实质新行为，返回 Plan。
- `k3.dtsi` 的 PWM0–PWM19 只能说明该分支静态节点集合，不证明 30 路概览错误。
- CoM260 引脚表存在大量复用功能；正文应选取能说明边界的代表性事实，避免复制整表和暗示默认 mux。

## Act Response

- Status: reported

**Implemented**

- T1.1：六个 MS11 官方入口的覆盖表行已从 `peripherals | future | deferred` 调整为 `peripherals | current | active`，同时保留 `unknown | 2026-09-02 | partially-observed`。Notes 改为「当前 MS11 聚合的权威入口；正文未直接取得，仅以已核对的官方 datasheet、DTS 和产品资料作 cross-validation」；17-Audio/23-WDT/24-RTC 在 Notes 追加「正文由后续 Iteration 承载」。覆盖表 URL 总数仍为 70、零重复。
- T1.2：创建 `docs/peripherals/k3-gpio-pwm-ir.md`（159 行）。首行 `> 来源:` 列出三个 MS11 入口 + 八个 supporting URL（K3 datasheet、`k3.dtsi`、`k3-pinctrl.dtsi`、三个 CoM260 DTS、CoM260 datasheet），源端修订与观察日期分两组标注。文档按 `范围与证据 → GPIO 控制器/复用/IRQ → PWM 通道/冲突/consumer → IR-RX 控制器/输入 → 错误边界与四字段未知项 → 主题边界` 六段组织；包含 GPIO controller 与 pinctrl 静态资源表、PWM 三种数量观察对照表、IR-RX 两节点表、CoM260 复用候选约束、U1–U5 五组四字段未知项（每组均含「当前证据/禁止推断/解除条件/影响主题」），15 个相对链接目标全部存在。

**Changed Files and Symbols**

- `docs/reference/source-coverage.md`（modified）：02-GPIO、03-PWM、04-IR-RX、17-Audio、23-WDT、24-RTC 共 6 行的 Horizon/Aggregation/Notes 字段；URL 本身、源端修订字段、观察日期与其他 64 行均未改动；diff stat `1 file changed, 6 insertions(+), 6 deletions(-)`。
- `docs/peripherals/k3-gpio-pwm-ir.md`（created，159 行）：新增唯一正文；未引用任何代码符号。
- 未创建 `evidence/` 目录（Persisted Evidence = `none`）。

**Deviations from Plan**

- T1.1 的覆盖表行在 Act 调用前已由前置步骤改为目标状态。本 Act 周期对六个目标行做了合同验证（`rg` 字段、URL 唯一性、URL 总数、Notes 文案），未再做二次修改即与 T1.1 合同匹配。Diff 仍由当前 Act 周期持有，不补做同一行重复 edit；如需可观察的「由 Act 实际写入」可由后续 rebase/整理时一次性展示。
- T1.2 正文首行额外引用 `k3-pinctrl.dtsi` 作为 supporting（覆盖表 line 88 已登记），未引入覆盖表以外的新 URL，未改变 URL 总数 70。

**Blocker Handoff**

None

**Blocker Resolution**

None

**Self-Review**

- Plan compliance: PASS — T1.1 六行与 T1.2 正文均满足合同「Required behavior」「Required changes」「Preserve」「Forbidden」全部条款；未触动 `others/`、既有产品正文、SNAPSHOT、全局 tasks、M/R/I、归档 change 或他人未跟踪目录。
- Full diff reviewed: PASS — `git status` 显示仅 `docs/reference/source-coverage.md` 修改与 `docs/peripherals/` 新增；`git diff --stat` 与 `git diff` 头 60 行核对过；新增 159 行内未出现 plan 范围外字符。
- Critical findings unresolved: 0
- Important findings unresolved: 0
- Minor findings unresolved: 0
- 备注：04-IR-RX 的 Notes 仅写「DTS 和产品资料」，未像 02-GPIO/03-PWM 那样写「datasheet、DTS 和产品资料」；这是因为 K3 datasheet 中 IR-RX 内容嵌入 input/IR 子节，未形成独立章节，因此 cross-validation 描述保留一致性而非机械对齐。属于合同内可调叙述，不构成 finding。

**Verification Evidence**

| 验证项 | 命令或操作 | 输出摘录 | 覆盖范围 | 结论 |
|---|---|---|---|---|
| T1.1 RED→GREEN | `rg -n '\| .*peripherals \| future \| deferred' docs/reference/source-coverage.md` | 0 行匹配 | coverage.md 六目标行 | PASS |
| T1.1 字段全检 | `rg -n '02-GPIO\.md\|03-PWM\.md\|04-IR-RX\.md\|17-Audio\.md\|23-WDT\.md\|24-RTC\.md' docs/reference/source-coverage.md` | 6 行均 `peripherals \| current \| active` | coverage.md 六目标行 | PASS |
| T1.1 URL 唯一 | `awk … sort \| uniq -d` | 空输出 | coverage.md 70 行 | PASS |
| T1.1 URL 总数 | `awk … wc -l` | 70 | coverage.md | PASS |
| T1.2 RED→GREEN | `test ! -e docs/peripherals/k3-gpio-pwm-ir.md` | 失败 → 文件已创建 | 新文件存在性 | PASS |
| T1.2 首行 | `head -1 docs/peripherals/k3-gpio-pwm-ir.md` | `> 来源: https://www.spacemit.com/...` | 新文件首行 | PASS |
| T1.2 章节覆盖 | `rg 'GPIO controller\|pinctrl\|IRQ\|PWM0..PWM19\|ircrx0\|ircrx1' docs/peripherals/k3-gpio-pwm-ir.md` | 多命中：§2 控制器、§3 三观察、§4 两节点 | 新文件章节 | PASS |
| T1.2 4-字段 | `rg '当前证据\|禁止推断\|解除条件\|影响主题' docs/peripherals/k3-gpio-pwm-ir.md` | U1–U5 每组 4 项均出现 | 新文件 §5 | PASS |
| T1.2 链接有效 | 解析 15 处 `](../…)` 相对链接并 `test -e` | 全部 OK（k3-platform-control, com260-board-resources, k3-interrupt-and-time, com260-image-and-dts, terminology, known-gaps） | 新文件相对链接 | PASS |
| 文件大小 | `wc -l docs/peripherals/k3-gpio-pwm-ir.md` | 159 | 新文件 | PASS（远低于 500 行预算） |
| Markdown 格式 | `git diff --check -- docs/reference/source-coverage.md docs/peripherals/k3-gpio-pwm-ir.md` | exit 0 | 两文件 diff | PASS |
| OpenSpec 严格校验 | `openspec validate establish-k3-general-peripheral-baseline --strict` | `Change 'establish-k3-general-peripheral-baseline' is valid` exit 0 | change 整体 | PASS |

**Persisted Evidence**

None required — `Mode: none`；上述验证为可低成本重跑的直接命令，结果已由摘录与退出码承载；不满足公共规则白名单的必要性、可采集性或预算门槛。

**Experience Candidates**

None

**Remaining Issues**

- 17-Audio / 23-WDT / 24-RTC 正文由 Iteration 001 / 002 承载；当前 Iteration 000 仅激活其覆盖表职责，不提前写正文。
- PWM 30/20/PWM0–19 数量冲突保持并列；解除需独立 refresh change 取得官方同版本答复。
- 02-GPIO / 03-PWM / 04-IR-RX 三个 Buildroot 页面在 2026-09-12 仍不可直接取得（raw.githubusercontent.com DNS 解析失败），由 U4 跟踪；不刷新 2026-09-02 观察日期。
- `docs/index.md` 仍把 `docs/peripherals/` 标为「待聚合」；由 Iteration 003 收敛。

**Commit or Diff Reference**

- 未在本 Act 周期内做 git commit；变更已就位等待用户审计。
- diff 摘要：`docs/reference/source-coverage.md`（6 行 +/-，+6/-6）；`docs/peripherals/k3-gpio-pwm-ir.md`（new，+159）。

## Plan Review

- Review Result: accepted

**Findings**

None。独立检查未发现阻塞 Acceptance 或非阻塞 Minor finding。

**Deviation Classification**

None

**Acceptance Gaps**

None。A1-A5 均满足。

**Convergence**

N/A

**Evidence**

- 独立检查 `docs/reference/source-coverage.md` 的实际 diff：只修改六个目标行；每行均为 `peripherals | current | active`，并保留 `unknown | 2026-09-02 | partially-observed`。
- 独立统计覆盖表得到 70 个 URL，`sort | uniq -d` 无输出；A1 通过。
- 独立阅读全文及新增文件 diff：GPIO controller/pinctrl/IRQ、PWM 30/20/PWM0-PWM19 三种观察、两个 disabled IR-RX 节点、CoM260 候选边界和 U1-U5 四字段均存在；未发现真板可用性或 PWM 数量裁决等越界断言；A2-A4 通过。
- 解析正文相对 Markdown 链接并逐项执行 `test -e`，无缺失目标；首行为 `> 来源:`；A5 通过。
- 采信 Act Response 中未失效的 159 行文件大小结论。Plan 新鲜运行 `git diff --check -- docs/reference/source-coverage.md docs/peripherals/k3-gpio-pwm-ir.md`，退出 0；运行 `openspec validate establish-k3-general-peripheral-baseline --strict`，输出 `Change 'establish-k3-general-peripheral-baseline' is valid`，退出 0。
- 工作区范围检查仍只见 `docs/reference/source-coverage.md`、`docs/peripherals/`、当前 change 与既有 `others/`；Act 未触动 `others/`。

**Follow-up Decision**

接受当前 Cycle。T1.1/T1.2 的行为、边界和验证满足 Iteration 000 的既有 Acceptance，无需当前 Cycle 修复或后继 Cycle；按既有 Iteration Map 展开 Iteration 001。

**Iteration Plan Update**

None

**Next Cycle**

None

**Next Iteration**

`../001-audio/000-initial.md`
