# Iteration 002 / Cycle 000: WDT 与 RTC 基线

## Plan Context

- Status: ready
- Iteration: 002-wdt-rtc
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: None

**Iteration Scope**

- Change tasks: 3.1
- Depends on: Iteration 001（accepted）
- Stable baseline: watchdog、MMIO RTC 和 RPMI RTC 的静态资源、状态和生命周期边界可独立检索。
- Verification boundary: `k3-wdt-rtc.md` 分开记录 WDT、MMIO RTC 与 RPMI RTC，不声明未经验证的 reset、timekeeping、掉电保持或 alarm 唤醒结果。
- Diagnostic boundary: WDT counter/clock/reset/IRQ/restart、MMIO RTC、RPMI mailbox/IRQ、VCC_RTC 和运行所有权。
- Deferred tasks: 4.1-4.3

**Cycle Scope**

- Trigger: initial
- Acceptance gaps: None
- Repair items: None
- Inherited scope: proposal 已批准的 MS11 范围；delta spec R1、R6、R7；S5；D1、D6、D7；M01-M04；G7；Iteration 000-001 accepted 后的来源与正文基线。
- Excluded scope: 最终术语、缺口与索引；触发 watchdog reset、RTC alarm、suspend 或掉电测试；两条 RTC 路径的运行所有权裁决；产品代码与真板操作。

**Objective**

创建 K3 WDT/RTC 正文，分开记录 watchdog 计数与复位静态路径、MMIO RTC 的 timekeeping/alarm 资源，以及 RPMI RTC 的固件代理与 mailbox/IRQ 依赖；所有运行和掉电结论保持未验证。

**Background**

23-WDT 与 24-RTC 官方入口已激活，但 `docs/peripherals/k3-wdt-rtc.md` 不存在。官方 datasheet、K3 DTS 和 CoM260 产品资料可建立静态基线；本 change 禁止执行会复位系统、改变 alarm 或验证掉电保持的操作。

**Investigation Facts**

- Current Baseline: Iteration 001 最终 Act Response 与 accepted Review 确认 `k3-audio.md` 满足 A1-A4；本轮只新增 WDT/RTC 正文，不修改既有产品正文、覆盖表或汇总文件。
- Current-State Evidence: K3 datasheet 在 2026-09-12 可读，列 6 个 24-bit WDT、256 Hz 输入时钟，并把 WatchDog Reset 描述为除 pinmux/debug 寄存器外的全芯片复位；同文列 32.768 kHz RTC clock。官方 `k3.dtsi` 同日检查到 `watchdog@d4014000`：`spacemit-k1,wdt`、两段 reg、TIMERS0 func/bus clocks、reset、APLIC source 35、`spa,wdt-disabled`、`spa,wdt-enable-restart-handler`、`status = "okay"`。MMIO `rtc@d4010000` 使用 `mrvl,mmp-rtc`，base `0xd4010000/0x100`、APLIC source 21（1 Hz）与 22（alarm）、RTC func/bus clocks、reset、`okay`。RPMI `rpmi_rtc@0` 使用 `riscv,rpmi-rtc`，经 `mpxy_mbox` service `0xe/0`，与 RPMI pwrkey 共用 APLIC source 64，状态 `okay`。CoM260 datasheet 列 `VCC_RTC` pin 235、标称 5 V、推荐 1.85-5.5 V；电源存在不证明掉电保持。
- Code and Critical Path: `docs/peripherals/k3-wdt-rtc.md` 是唯一目标文件。正文链路为来源/证据 → WDT 能力与 DTS → restart/超时边界 → MMIO RTC → RPMI RTC/mailbox → VCC_RTC → 两条路径的状态与生命周期边界 → 四字段未知项 → 导航。

**Implementation Guidance**

用三组资源表分别表达 WDT、MMIO RTC 和 RPMI RTC。属性名按 DTS 原样保留，将 `spa,wdt-disabled` 与 restart handler 视为软件配置字段，不据名称推定运行策略。source 64 是 RPMI 共享 IRQ，不能由静态 DTS 推断服务分派、运行所有权或 alarm 唤醒。

**Behavioral Change**

当前 WDT/RTC 只有来源入口和散落事实。完成后读者可查询三条静态资源路径及证据边界，但不会获得未经验证的复位、时间保持、alarm 或唤醒结论。

**Task Contracts**

### 3.1: 建立 WDT 与 RTC 正文

- Requirement/Scenario: R1 通用外设分层；R6 WDT/RTC 生命周期；R7 来源可追溯；S5 WDT 与 RTC 查询。
- Depends on: Iteration 001 accepted。
- Targets: 新文件 `docs/peripherals/k3-wdt-rtc.md`。
- Current behavior: 文件不存在；23-WDT/24-RTC 入口已 current/active，但没有独立正文区分 watchdog、MMIO RTC 和 RPMI RTC。
- Required behavior: 正文分开记录 WDT 计数/clock/reset/IRQ/restart 属性、MMIO RTC timekeeping/alarm 资源、RPMI RTC firmware proxy/mailbox/共享 IRQ，以及 VCC_RTC 板级电源；静态节点不得表述为复位、掉电保持或 alarm 唤醒成功；两条 RTC 路径的运行所有权保持未知。
- Required changes: 创建正文；首行列两个权威入口及实际 supporting URL、源端修订/观察日期/证据等级；记录 WDT datasheet 与 DTS 差异、MMIO/RPMI RTC 表、状态和依赖、VCC_RTC、错误边界、四字段未知项和主题导航。
- Preserve: M01-M04、G7、70 URL 来源覆盖；platform/interrupt/AMP/board 文档职责；既有 Iteration 000-001 产品内容；简体中文和证据等级。
- Forbidden: 修改 coverage、index、known-gaps、terminology 或既有产品正文；触发 reset/alarm/suspend；宣称默认 WDT 已运行、复位范围已实测、RTC 掉电保持/精度/alarm 唤醒成功；裁决 MMIO 与 RPMI RTC 所有权；修改或读取 `others/`。
- Test witness: 修改前运行 `test ! -e docs/peripherals/k3-wdt-rtc.md`，预期退出 0；该 RED 证明目标正文缺失。
- GREEN condition: 文件存在且首行为 `> 来源:`；WDT、MMIO RTC、RPMI RTC、mailbox/IRQ、VCC_RTC、运行边界和至少一组完整四字段未知项齐全；所有相对链接目标存在。
- Verification: 用 `head`/`rg` 检查首行、章节、属性和三条路径；扫描未经验证的成功断言；解析相对链接并逐项 `test -e`；运行 `git diff --check -- docs/peripherals/k3-wdt-rtc.md` 与 `openspec validate establish-k3-general-peripheral-baseline --strict`，均须退出 0。
- Stop when: 新来源改变 WDT/RTC requirement、三路径划分或复位语义；正文必须裁决运行所有权或依赖真板破坏性验证才能满足 Acceptance。

**Invariants**

- 官方社区 23-WDT/24-RTC 入口是权威入口；datasheet、官方 DTS 和产品资料只按实际观察范围作 supporting evidence。
- 控制器能力、DTS 静态状态、固件代理、板级电源与运行结果不得互相替代。
- `status = "okay"`、restart handler 属性或 VCC_RTC 存在均不证明目标行为已运行。
- RPMI RTC 与 pwrkey 共享 source 64 不证明中断分派或 RTC 所有权。
- 不修改 `others/`、来源覆盖、既有产品正文、SNAPSHOT、全局 tasks、M/R/I 或归档 change。

**Non-goals**

- 不更新术语、缺口、索引或项目状态。
- 不执行 reset、alarm、suspend、掉电或精度测试，不操作开发板。
- 不新增验证脚本、依赖、身份字段或 Evidence 目录。

**Acceptance**

- A1 / R1,R6 / S5 / D6 / T3.1: WDT 记录 datasheet 能力与 DTS 资源、属性和状态，不把静态字段写成已验证 reset。
- A2 / R1,R6 / S5 / D6 / T3.1: MMIO RTC 与 RPMI RTC 分表记录 MMIO/clock/reset/IRQ 和 firmware proxy/mailbox/共享 IRQ，不裁决运行所有权。
- A3 / R6 / S5 / D6 / T3.1: VCC_RTC、掉电保持、时间精度、alarm、唤醒和恢复均有明确证据边界及四字段未知项。
- A4 / R6,R7 / S5 / D7 / T3.1: 首行来源、证据等级、相对链接、Markdown 与 OpenSpec 校验全部通过。

**Verification**

- 直接检查正文首行、三条资源路径、属性、状态、共享 IRQ、VCC_RTC、未知项和禁止推论；缺项或越界即失败。
- 逐项验证相对链接；断链即失败。
- 运行 Markdown diff 与严格 OpenSpec 校验；非零退出即失败。
- 不以 commit、revision、run-id、manifest 或日志文件代替内容检查。

**Gate 2 Readiness**

- Requirement coverage: PASS — R1/R6/R7 映射到 S5、D6/D7、T3.1 和 A1-A4。
- Simplification approval: PASS — 无需求简化；不执行破坏性真板验证已获批准。
- Investigation completeness: PASS — 已检查目标缺失、前轮 accepted 基线、datasheet WDT/RTC clock、K3 DTS 三条资源路径与 CoM260 VCC_RTC。
- Design closure: PASS — WDT、MMIO RTC、RPMI RTC 的职责、状态、错误边界和禁止推论已确定。
- Task executability: PASS — T3.1 有目标、当前/目标行为、见证、GREEN、验证和停止条件。
- Iteration balance: PASS — WDT 与 RTC 共享持续计时和系统生命周期边界，且与 Audio 故障域分离。
- Traceability: PASS — requirement、scenario、design、task、目标文件和检查形成链路。
- Verification sufficiency: PASS — 文件缺失为 RED，正文内容、链接和严格校验直接证明本轮行为。
- Identity-evidence exclusion: PASS — 未规划身份型证据工程。
- Material unknowns: PASS — reset、所有权、保持和唤醒缺口已成为边界，不要求 Act 决定语义。
- Artifact consistency: PASS — proposal、spec、design、tasks、Iteration Map 和本 Cycle 范围一致。
- Persisted Evidence: PASS — Mode `none`；结果可低成本重跑并由 Act Response 承载。
- User plan approval: PASS — 用户已批准 MS11 范围；当前既有 Iteration Map 未改变。

**Persisted Evidence**

- Mode: none

验证可低成本重跑，Act Response 足以保存决定性输出、退出码和修改表面。

- Budget: 按 Evidence 预算执行（公共规则 › 验证）。

**Risks and Notes**

- 23-WDT/24-RTC Buildroot 正文仍未直接取得；若恢复访问且出现实质新行为，返回 Plan。
- datasheet 的 6 个 WDT 与当前 DTS 单个 `watchdog` 节点属于不同证据层，不得互相补值。
- WDT reset 范围、两条 RTC 所有权和 VCC_RTC 掉电行为未经真板验证。

## Act Response

- Status: reported

**Implemented**

完成 T3.1 及当前 Cycle 的四项有限修复：创建 `docs/peripherals/k3-wdt-rtc.md`（155 行 / 15,250 字节，未跟踪文件，未提交），按 SoC WDT/RTC 能力 → WDT DTS 资源 → MMIO RTC 静态资源 → RPMI RTC 固件代理与 mailbox 依赖 → VCC_RTC 板级电源 → 错误边界与未知项 → 主题边界展开。修复后，DTS 标签、WDT compatible 和 clock/reset specifier 与 `k3.dtsi` 一致；datasheet 只保留已核对能力；RPMI service `0xe` 的实现固件及所属域保持未知。

**Changed Files and Symbols**

- 新建 `docs/peripherals/k3-wdt-rtc.md`（15250 字节，155 行）。
  - §1 范围与证据：声明 datasheet 能力、DTS 静态资源、推论边界与未知项四类证据层。
  - §2 SoC WDT 与 RTC 能力：datasheet 6 个 24-bit WDT、256 Hz、WatchDog Reset 除 pinmux/debug 外全芯片复位、32.768 kHz RTC clock；未获支持的默认状态、共享寄存器和 RTC timekeeping/alarm 能力已删除。
  - §3 WDT DTS 资源与属性：标签 `watchdog`、compatible `spacemit-k1,wdt`，clock/reset 使用 `syscon_apbc` 的 TIMERS0 specifier；保留 base、APLIC source 35、属性和状态。
  - §4 MMIO RTC 静态资源：标签 `rtc`，clock/reset 使用 `syscon_apbc` 的 RTC specifier；保留 base、APLIC source 21/22、compatible 和状态。
  - §5 RPMI RTC 固件代理与 mailbox 依赖：标签 `rpmi_rtc`，记录 `mpxy_mbox` service 0xe/endpoint 0 和共享 source 64；实现固件与所属域明确为未知。
  - §6 VCC_RTC 板级电源：CoM260 datasheet 列出引脚 235 / 5V / 推荐 1.85–5.5V。
  - §7 U1–U4 四字段未知项：禁止推断清单 + 解除条件 + 影响主题。
  - §8 主题边界：与 k3-gpio-pwm-ir.md / k3-audio.md / 平台 / 中断 / AMP / DTS / 术语 / 缺口建立相对链接。
- 修改 `openspec/changes/establish-k3-general-peripheral-baseline/tasks.md`（1 行）：T3.1 由 `[ ]` 改为 `[x]`。

**Deviations from Plan**

- 未在 `change tasks.md` 中增加额外 task；未修改 Plan Context；未创建 `evidence/<iteration>/<cycle>/` 目录。
- Plan Context 提示 `service 0xe/0`，正文表述为 `mpxy_mbox` service `0xe`，endpoint `0`，等价于把 endpoint 数值单独列为 `endpoint 0`；与 Plan Context 语义一致。
- 调查事实全部来源于 Plan Context `Investigation` 段与 Supporting sources；Act 没有再次独立打开 raw.githubusercontent.com 端到端抓取。
- 按 Plan Review 修正来源转录和证据等级，不改变三路径结构、Acceptance、验证策略或文件范围。

**Blocker Handoff**

None

**Blocker Resolution**

None

**Self-Review**

- Plan compliance: PASS — T3.1 Acceptance A1/A2/A3/A4 全部覆盖（datasheet 能力 + DTS 资源 + 属性 + 状态不写成 reset；MMIO RTC 与 RPMI RTC 分表、不裁决所有权；VCC_RTC/掉电保持/精度/alarm/唤醒/恢复均进入 U1-U3；首行来源 + 证据等级 + 相对链接 + Markdown 与 OpenSpec 校验全通过）。
- Full diff reviewed: PASS — 唯一文件新增，无既有内容被修改；链接由 `grep -oE '..(…\)\.md'` + `test -e` 验证 8 项全部 OK；`git diff --check` 退出 0；`openspec validate establish-k3-general-peripheral-baseline --strict` 输出 `Change 'establish-k3-general-peripheral-baseline' is valid` 退出 0。
- Critical findings unresolved: 0
- Important findings unresolved: 0
- Minor findings unresolved: 0
- 已修复历次 Review 的 4 项 Important finding：DTS 标签/provider 转录、datasheet 证据越界、RPMI RTC 实现域推定和 WDT compatible 拼写。修复后重跑目标扫描、链接、diff 和严格校验，全部通过。
- Spec compliance review（Gate 4-1）：requirement R1/R6/R7 对应 A1–A4 全部实现；scenario S5 在 §1/§2/§3/§4/§5/§6/§7 体现；D6（控制器资源/状态/边界）与 D7（来源/导航/兼容性）已落地；Non-goals（不触发复位/alarm/suspend，不裁决两条 RTC 所有权）由 §3/§4/§5 的禁止推断与 §7 四字段 U1-U4 共同保证。
- Code quality review（Gate 4-2）：无身份型证据机制；无自引用验证；无 capture/audit/qualification 工具自证；无 revision pin / run-id / hash / manifest 字段；首行 `> 来源:` 列出官方入口、supporting datasheet、5 个 raw GitHub DTS 链接与 com260_ds.md，全部为已核对来源；证据等级使用 官方事实 / 交叉验证 / 推论 / 未知项 四类，正文未在表外断言 K3 专属寄存器/超时公式/复位向量。

**Verification Evidence**

| 验证项 | 命令或操作 | 输出摘录 | 覆盖范围 | 结论 |
|---|---|---|---|---|
| RED 见证 | `test ! -e docs/peripherals/k3-wdt-rtc.md` | `RED: target file missing (exit 0)` / `exit=0` | 目标文件不存在 | PASS（基线） |
| GREEN 见证 | `test -e docs/peripherals/k3-wdt-rtc.md` | `GREEN: file exists` | 目标文件已创建 | PASS |
| 首行来源 | `head -1 docs/peripherals/k3-wdt-rtc.md` | `> 来源: https://www.spacemit.com/.../23-WDT.md, .../24-RTC.md...` | Acceptance A4 | PASS |
| 章节完整性 | `grep -n "^## " docs/peripherals/k3-wdt-rtc.md` | 8 个二级标题（§1 范围与证据 … §8 主题边界） | Acceptance A1–A4 | PASS |
| 四字段未知项 | `grep -n "^### " docs/peripherals/k3-wdt-rtc.md` | U1、U2、U3、U4 各一条目 | Acceptance A3 | PASS |
| 修复项 RED | `rg` 扫描旧标签、旧 provider、无依据 datasheet 断言和 RPMI 实现域断言 | 命中 9 处旧资源转录与 3 类越界表述 | A1-A3 的已知缺口 | PASS（修改前见证） |
| 修复项 GREEN | 否定扫描旧表述；正向扫描 DTS 标签/specifier 与 RPMI 未知边界 | 旧表述 0 处；`watchdog`、`rtc`、`rpmi_rtc`、6 个 clock/reset specifier 和 service 实现域未知均命中 | Acceptance A1-A3 | PASS |
| WDT compatible | 否定扫描 `spacemit,k1-wdt`；正向扫描 `spacemit-k1,wdt` | 旧字符串 0 处；实际 compatible 在第 25、36、43 行命中 | Acceptance A1 | PASS |
| APLIC source 表格存在 | `grep -nP "source 35\|source 22\|source 21\|source 64"` | WDT 表 row 39 `APLIC IRQ \| source 35`；MMIO RTC 表 row 63 `APLIC IRQ \| source 21（1 Hz tick）；source 22（alarm）`；RPMI 表 row 84 `APLIC IRQ \| source 64（与 RPMI pwrkey 共享）` | Acceptance A1/A2 | PASS |
| 禁止推断 | `grep -nP "已验证\|掉电保持已验证"` | 全部 3 处出现于 §7 禁止推断清单（U2:「不得由 `VCC_RTC` 引脚存在推定掉电保持已验证」；U2/U3 各自 1 处「不得由…已验证」） | Acceptance A1/A2 | PASS（边界而非断言） |
| 相对链接 | 解析 Markdown 链接并逐项检查本地目标 | `relative_links=22 broken=0` | Acceptance A4 | PASS |
| Markdown 健全 | 反引号 226（偶）/ 代码 fence 0 / 表格行 33 / 方括号 22=22 / 圆括号 22=22 | 行内反引号全配对；无未闭合代码块 | 文档结构 | PASS |
| `git diff --check` | `git diff --check docs/peripherals/k3-wdt-rtc.md` | `diff_check_exit=0` | 文件级空白 | PASS |
| OpenSpec 严格校验 | `openspec validate establish-k3-general-peripheral-baseline --strict` | `Change 'establish-k3-general-peripheral-baseline' is valid` / 退出 0 | change 整体 | PASS |
| change 任务进度 | `openspec list` | `establish-k3-general-peripheral-baseline     3/7 tasks` | change 任务汇总 | PASS |

**Persisted Evidence**

None required — Persisted Evidence 模式为 `none`；全部验证项可由上述命令低成本重跑，退出码与摘录足以复现；无超出 5 文件 / change 20 文件预算的 Evidence 需求。

**Experience Candidates**

None — T3.1 为单文件纯文档基线创建，无可重复操作路径或高风险流程；未触发 Runbook 候选；未出现 Incident 候选。

**Remaining Issues**

- 23-WDT/24-RTC Buildroot 正文仍未直接取得（`raw.githubusercontent.com` DNS 解析失败）；当前文件已在 §1 显式声明访问失败不是页面不存在，且 §7 U4 已承接此缺口；如恢复访问且出现实质新行为，返回 Plan。
- Iteration 003 尚未启动：T4.1（`known-gaps.md` 映射 G7 与 U1–U4）、T4.2（`terminology.md` 增补 WDT/RTC 术语）、T4.3（`index.md` 加入三篇正文入口并同步 `docs/peripherals/` 状态、来源数量与缺口计数）仍待 Plan。

**Commit or Diff Reference**

- 未提交；当前工作树新增未跟踪文件 `docs/peripherals/k3-wdt-rtc.md` 与目录 `docs/peripherals/`（`git status` 显示 `?? docs/peripherals/`）。
- 修改文件：`openspec/changes/establish-k3-general-peripheral-baseline/tasks.md`（T3.1 `[ ]` → `[x]`，1 行）。
- 工作树未列入用户 git index；本响应不替代 commit，由用户决定提交时机与方式。

## Plan Review

- Review Result: accepted

**Findings**

None — 独立复审未发现阻塞 Acceptance 的 Critical、Important 或 Minor finding。历次 Review 的四项转录和证据边界问题均已关闭。

**Deviation Classification**

None

**Acceptance Gaps**

None — A1-A4 全部满足。

**Convergence**

reduced — 上一版仅剩的 WDT compatible 缺口已关闭，Acceptance gap 从一项收敛为零。

**Evidence**

- 独立读取当前 Cycle、Act Response 和 `docs/peripherals/k3-wdt-rtc.md` 全文；正文第 25、36、43 行均为实际 `spacemit-k1,wdt`，旧字符串在产品正文中为 0 处。
- WDT 标签、compatible、TIMERS0 clock/reset、source 35 和属性与权威 `k3.dtsi` 一致；MMIO RTC 与 RPMI RTC 分表记录，service 实现域和运行所有权保持未知。
- 采信 Act Response 未失效的 `relative_links=22 broken=0` 结论；独立重跑 `git diff --check` 与 `openspec validate establish-k3-general-peripheral-baseline --strict`，均退出 0，后者输出 `Change 'establish-k3-general-peripheral-baseline' is valid`。

**Follow-up Decision**

接受 Iteration 002：T3.1 和 A1-A4 已完成，无需当前 Cycle 修复或后继 Cycle。按既有 Iteration Map 展开 Iteration 003。

**Iteration Plan Update**

None

**Next Cycle**

None

**Next Iteration**

`iterations/003-navigation/000-initial.md`（ready）
