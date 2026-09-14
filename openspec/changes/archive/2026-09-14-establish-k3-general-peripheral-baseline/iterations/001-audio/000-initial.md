# Iteration 001 / Cycle 000: Audio 基线

## Plan Context

- Status: ready
- Iteration: 001-audio
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: None

**Iteration Scope**

- Change tasks: 2.1
- Depends on: Iteration 000（accepted）
- Stable baseline: Audio controller、DAI、sound card、endpoint、DMA、power domain 和 CoM260 引脚关系具备独立正文。
- Verification boundary: `k3-audio.md` 覆盖静态端点链、clock/reset、DMA channel、显示音频和板级 I2S；静态资源与 stream 运行结果分离。
- Diagnostic boundary: I2S/SSPA、CPU DAI、codec/display endpoint、sound card、DMA/cache 和板级 route。
- Deferred tasks: 3.1, 4.1-4.3

**Cycle Scope**

- Trigger: initial
- Acceptance gaps: None
- Repair items: None
- Inherited scope: proposal 已批准的 MS11 范围；delta spec R1、R5、R7；S4；D1、D5、D7；M01-M04；G7；Iteration 000 接受后的 70 URL current/active 来源基线。
- Excluded scope: WDT/RTC 正文；最终术语、缺口和索引；音频播放或录音；codec 型号、默认 CoM260 DTS 或 stream 参数推定；产品代码与真板操作。

**Objective**

创建一篇可追溯的 K3 Audio 正文，按 SoC 能力、I2S/SSPA controller、CPU DAI、sound card、codec/display endpoint、clock/reset、DMA、power domain 和 CoM260 引脚分层，并把静态 DTS 链与未验证的音频流明确分开。

**Background**

17-Audio 官方入口已在 Iteration 000 激活，但正文未直接取得，`docs/peripherals/k3-audio.md` 仍不存在。现有官方 datasheet、Linux DTS 和 CoM260 产品资料足以建立静态资源及板级候选基线；codec、route、buffer 生命周期、IRQ/xrun 和 stream 日志仍缺失。

**Investigation Facts**

- Current Baseline: Iteration 000 的最终 Act Response 和 accepted Review 已确认六个 MS11 URL 为 current/active、URL 总数 70 且无重复；本轮只新增 Audio 正文，不再修改覆盖表。`docs/peripherals/k3-audio.md` 不存在，`docs/index.md` 仍留待 Iteration 003 收敛。
- Current-State Evidence: K3 datasheet 在 2026-09-12 可读，Audio Subsystem 章节列出 6 路全双工 I2S、4 路半双工 I2S（其中两路连接 DP/eDP）和 2 路 DP/eDP Audio，并分别给出 I2S 格式、声道、位深和采样率能力。官方 `k3.dtsi` 同日检查到 AP 域 `i2s0`-`i2s5`：`spacemit,k1-i2s`、`#sound-dai-cells = <0>`、独立 MMIO/clock/reset、PDMA `rx/tx` channel，默认 `disabled`；另有两个默认 disabled 的 `simple-audio-card` 节点和 audio power domain。CoM260 DTS 变体中观察到 DP1 路径启用 `adma3`、`ri2s3`、`sound_card_dp1`，其 codec endpoint 指向 `dp1`；这只证明该变体的静态显示音频链。CoM260 产品资料列 I2S/SSPA 复用引脚，只证明板级候选。
- Code and Critical Path: `docs/peripherals/k3-audio.md` 是唯一目标文件。正文链路为来源与证据等级 → SoC Audio 能力 → AP I2S/SSPA controller 与 CPU DAI → sound card/codec/display endpoint → clock/reset/DMA/power domain → CoM260 引脚候选 → stream、错误和四字段未知项 → 相关主题导航。

**Implementation Guidance**

以资源表表达 AP I2S 实例及共同字段，用端点方向图或短表表达 CPU DAI 到 codec/display endpoint 的引用关系。把 datasheet 能力、SoC DTS 默认状态和特定 CoM260 DTS 覆盖分栏，不将 `okay`、sound card 或 endpoint 存在写成播放成功。DMA/cache 只链接既有 MS06 文档。

**Behavioral Change**

当前 Audio 只有 active 来源入口和散落静态事实。完成后读者可从独立正文查询 K3 Audio 控制器、DAI/端点、资源依赖与板级候选，同时明确哪些 codec、route、格式和运行行为尚未验证。

**Task Contracts**

### 2.1: 建立 Audio 正文

- Requirement/Scenario: R1 通用外设分层；R5 Audio 数据链；R7 来源可追溯；S4 Audio 数据链和板级接口。
- Depends on: Iteration 000 accepted。
- Targets: 新文件 `docs/peripherals/k3-audio.md`。
- Current behavior: 文件不存在；17-Audio 入口已 current/active，但 Audio 事实散落于 datasheet、DTS 和板级资料，没有独立正文。
- Required behavior: 正文按 controller、CPU DAI、sound card、codec/display endpoint、clock/reset、DMA、power domain 和 CoM260 pin 候选分层；标明节点状态和证据等级；静态链不得表述为 stream 成功；codec、route、采样配置、buffer/IRQ/xrun 和真板结果以四字段未知项记录；DMA/cache 通过 MS06 正文引用。
- Required changes: 创建主题正文；首行列 17-Audio 权威入口及实际使用的官方 supporting URL、源端修订/观察日期/证据等级；记录 datasheet Audio 能力、AP `i2s0`-`i2s5` 共同资源及实例差异、sound card/endpoint、显示音频、audio power domain、CoM260 引脚候选和主题导航。
- Preserve: M01-M04、G7、70 URL 来源覆盖；既有 platform/interrupt/DMA/board 文档的权威职责；简体中文和既有证据等级；Iteration 000 产品内容。
- Forbidden: 修改 coverage、index、known-gaps、terminology 或既有产品正文；复制 MS06 的 DMA/cache 所有权模型；推定 codec 型号、默认顶层 DTS、板级 route、音频播放/录音成功、实时性能或 xrun 恢复；写真板操作步骤；读取或修改 `others/`。
- Test witness: 修改前运行 `test ! -e docs/peripherals/k3-audio.md`，预期退出 0；该 RED 证明 Audio 正文入口缺失。
- GREEN condition: 文件存在且首行为 `> 来源:`；包含 SoC Audio 能力、AP I2S/SSPA、CPU DAI、sound card、codec/display endpoint、clock/reset、DMA、power domain、CoM260 引脚候选、证据边界及至少一组完整四字段未知项；所有相对链接目标存在。
- Verification: 用 `head`/`rg` 检查首行、章节和必需术语；检查 `i2s0`-`i2s5`、`rx/tx` DMA、默认状态及 DP1 静态链；扫描禁止越界断言；解析相对 Markdown 链接并逐项 `test -e`；运行 `git diff --check -- docs/peripherals/k3-audio.md` 和 `openspec validate establish-k3-general-peripheral-baseline --strict`，均须退出 0。
- Stop when: 新来源改变三篇拆分、Audio requirement 或端点责任；无法区分 SoC 静态资源与目标板 route；正文需要选择默认 CoM260 DTS、补写未证 stream 状态机或修改 MS06 所有权模型。

**Invariants**

- 官方社区 17-Audio 入口仍是权威入口；官方 GitHub datasheet、DTS 和产品资料仅按实际观察范围提供 supporting evidence。
- SoC 能力、DTS 静态节点、CoM260 引出、sound card/endpoint 与音频流结果不得互相替代。
- `status = "okay"` 只证明所读 DTS 变体的静态启用状态，不证明 codec、route 或 stream 可用。
- DMA/cache 所有权由 MS06 正文承担，本轮只记录 Audio consumer 到 DMA provider 的引用方向。
- 不修改 `others/`、来源覆盖、既有产品正文、SNAPSHOT、全局 tasks、M/R/I 或归档 change。

**Non-goals**

- 不建立 WDT/RTC、术语、缺口或索引内容。
- 不下载、构建或运行 Linux/Buildroot，不播放或录制音频，不操作开发板。
- 不新增验证脚本、依赖、身份字段或 Evidence 目录。

**Acceptance**

- A1 / R1,R5 / S4 / D5 / T2.1: 正文按 controller、CPU DAI、sound card、codec/display endpoint 分层，区分 datasheet 能力、SoC DTS 与 CoM260 DTS 变体。
- A2 / R1,R5 / S4 / D5 / T2.1: 正文记录 clock/reset、`rx/tx` DMA channel、audio power domain 和 CoM260 I2S/SSPA 引脚候选，并引用 MS06 DMA/cache 基线而不复制其模型。
- A3 / R5 / S4 / D5 / T2.1: 节点启用、sound card 和 endpoint 不被表述为 stream 成功；codec、route、采样配置、buffer 生命周期、IRQ/xrun、实时性能和真板结果保持未知。
- A4 / R5,R7 / S4 / D7 / T2.1: 首行来源、证据等级、四字段未知项、相对链接和 Markdown/OpenSpec 校验全部通过。

**Verification**

- 直接检查新正文的首行、章节、实例表、端点链、资源依赖、状态、未知项和禁止推论；缺项或越界陈述即失败。
- 逐项验证相对链接目标存在；断链即失败。
- 运行 `git diff --check -- docs/peripherals/k3-audio.md` 与严格 OpenSpec 校验；非零退出即失败。
- 不使用 commit、revision、run-id、manifest 或日志文件代替内容检查。

**Gate 2 Readiness**

- Requirement coverage: PASS — R1/R5/R7 映射到 S4、D5/D7、T2.1 和 A1-A4。
- Simplification approval: PASS — 没有需求简化；三篇拆分和不执行真板验证已在 Gate 1 获批。
- Investigation completeness: PASS — 已检查目标文件不存在、Iteration 000 接受后的来源基线、K3 datasheet Audio 章节、K3/CoM260 DTS 静态 Audio 节点和既有 DMA/platform/board 责任边界。
- Design closure: PASS — controller/DAI/endpoint、资源依赖、证据等级、未知项和禁止推论已确定。
- Task executability: PASS — T2.1 具备目标、当前/目标行为、测试见证、GREEN、验证和停止条件。
- Iteration balance: PASS — Audio 的流式 DMA、端点和 power domain 形成独立故障域；不与低速 I/O 或系统计时合并。
- Traceability: PASS — R1/R5/R7、S4、D5/D7、T2.1、目标文件和检查形成闭合链路。
- Verification sufficiency: PASS — 文件缺失为 RED，正文内容、链接和严格校验直接证明本轮文档行为。
- Identity-evidence exclusion: PASS — 未规划 Hash、revision pin、run-id、manifest 或专用验证工具。
- Material unknowns: PASS — codec、route 和 stream 缺口已成为明确边界，不要求 Act 决定契约语义。
- Artifact consistency: PASS — proposal、spec、design、tasks、Iteration Map 和本 Cycle 使用相同范围。
- Persisted Evidence: PASS — Mode `none`；检查可低成本重跑并由 Act Response 承载。
- User plan approval: PASS — 用户在 Gate 1 批准 MS11 全部范围，并于 2026-09-12 指示“开始审计”；既有 Iteration Map 未改变，本轮可交给 Act。

**Persisted Evidence**

- Mode: none

Act Response 足以记录命令、决定性输出、退出码和修改表面；检查可低成本重跑，不满足持久 Evidence 必要性门槛。

- Budget: 按 Evidence 预算执行（公共规则 › 验证）。

**Risks and Notes**

- 17-Audio Buildroot 正文仍未直接取得；若实施时恢复且出现实质新行为，返回 Plan。
- datasheet 能力数量、AP I2S 节点和 CoM260 DP1 变体属于不同层，不应合并为单一“可用接口数”。
- CoM260 DTS 变体不唯一，G7 继续约束默认板型和 route 推定。

## Act Response

- Status: reported

**Implemented**

- 创建 `docs/peripherals/k3-audio.md`（196 行 / 20,821 字节）。
- 正文链路：首行 17-Audio 权威入口与 10 个官方 supporting URL（k3_ds.md V1.8、8 个 k3-br-v1.0.y DTS / DTSI、com260_ds.md）→ §1 范围与证据（四级证据强度）→ §2 SoC Audio 能力（datasheet §2.6 全双工 6 / 半双工 4 / DP-eDP 2）→ §3 AP 域 `i2s0`–`i2s5`（6 节点表 + 共同字段 + PDMA 通道 21-63）→ §4 RCPU 域 `ri2s0`–`ri2s3` + `adma0`–`adma3` + `sound_card_dp0/1` + `dp0/dp1` 端点 → §5 clock/reset/DMA/power domain 与 CoM260 引脚候选 → §6 板级映射与运行边界（含 CoM260 DTS 变体 DP1 静态链观察与 10 项不能推出的结论）→ §7 4 个四字段未知项 U1-U4 → §8 主题边界。
- 按 Plan Review 修复三项 Important finding：明确 sound card 同时存在 CPU endpoint 与指向 DP/eDP 的 codec endpoint、外接模拟 codec 未观察；把 AP `i2s0`–`i2s5` 的完成路径限定为其实际引用的 `&pdma` / APLIC source 72，并单列未被引用的 `pdma1` source 150；在首行补入实际使用的 `k3-dp0.dtsi` supporting URL。

**Changed Files and Symbols**

- 新增并修复 `docs/peripherals/k3-audio.md`（无其他产品文件修改；预存的 `docs/reference/source-coverage.md` 与 `others/Rt-Async-AMP/` 来自 Iteration 000 收尾状态，按 Plan Context Invariants 不触碰）。

**Deviations from Plan**

当前实现与原计划的唯一过程差异是 Plan Review 后的有限修复，未改变任务范围或契约。Required behavior 全部实现：① 8 段分层；② 静态链不表述为 stream 成功；③ codec、route、采样配置、buffer 生命周期、IRQ/xrun、实时性能和真板结果保持未知；④ DMA/cache 通过 MS06 正文引用；⑤ 首行 `> 来源:` 含权威入口及全部实际使用的 supporting URL；⑥ U1-U4 四字段完整。

**Blocker Handoff**

None

**Blocker Resolution**

None

**Self-Review**

- Plan compliance: PASS — 重新对照 Task Contract、A1-A4 与最新 Plan Review；三项 Acceptance gap 均已关闭，R1/R5/R7、D1/D5/D7 和 S4 边界保持。
- Full diff reviewed: PASS — 完整复查 196 行正文和工作区状态；本 Cycle 唯一产品文件仍为 `docs/peripherals/k3-audio.md`，未触碰预存 `source-coverage.md` 或 `others/`。
- Critical findings unresolved: 0
- Important findings unresolved: 0
- Minor findings unresolved: 0

**Verification Evidence**

| 验证项 | 命令 | 输出摘录 | 覆盖范围 | 结论 |
|---|---|---|---|---|
| Gate 3 RED 翻转 | `test -e docs/peripherals/k3-audio.md` | 实施前 `exit 0`（文件不存在）；实施后 `GREEN: file exists` | 目标文件存在性 | PASS |
| 首行 | `head -1 docs/peripherals/k3-audio.md` | `> 来源: https://www.spacemit.com/.../17-Audio.md（...）` | 首行模板合规 | PASS |
| 章节结构 | `grep -n "^## " docs/peripherals/k3-audio.md` | 8 个 `##` 一级章节（范围与证据 / SoC Audio 能力 / AP 域 I²S / RCPU 域 I²S / clock-reset-DMA-power / 板级映射 / 错误边界 / 主题边界） | Plan Context A1 章节覆盖 | PASS |
| 必需术语 | `grep -c` 23 个关键术语 | `i2s0` 19 / `i2s1` 11 / `i2s2` 12 / `i2s3` 16 / `i2s4` 2 / `i2s5` 11 / `ri2s0` 10 / `ri2s1` 5 / `ri2s2` 9 / `ri2s3` 14 / `adma3` 9 / `sound_card_dp1` 9 / `dp1` 15 / `audio@2` 2 / `spacemit,k1-i2s` 3 / `spacemit,k3-ri2s` 3 / `U1` 5 / `U2` 1 / `U3` 1 / `U4` 1 / `spacemit,fixed-sample-rate` 3 / `playback-only` 4 / `mclk-fs` 5 / `left_j` 7 | A2/A3 资源与端点 | PASS |
| 4 字段未知项 | `sed -n '/^### U[0-9]/,/^### U[0-9]\\|^## /p'` | U1/U2/U3/U4 每项含 当前证据 / 禁止推断 / 解除条件 / 影响主题 四字段 | A4 完整性 | PASS |
| 相对链接解析 | `test -e` 11 个相对路径 | 全部 OK（`../amp/...` / `../boot/...` / `../dma/...` / `../interrupts/...` / `../platform/...` / `../reference/...` 各路径均存在） | A4 链接有效 | PASS |
| Markdown 语法 | `git diff --check -- docs/peripherals/k3-audio.md` | `Exit: 0` | A4 静态校验 | PASS |
| OpenSpec 严格校验 | `openspec validate establish-k3-general-peripheral-baseline --strict` | `Change 'establish-k3-general-peripheral-baseline' is valid` / `Exit: 0` | A4 change 合规 | PASS |
| 行数约束 | `wc -l docs/peripherals/k3-audio.md` | 196 行 | M02 500 行上限 | PASS |
| Review 修复 | `rg` 检查旧 codec 矛盾与 `source 72/150` 表述；首行检查 `k3-dp0.dtsi` | 旧矛盾与错误 IRQ 汇总零命中；`k3-dp0.dtsi` 命中 | A1-A4 三项 gap | PASS |
| 修复后路径归属 | `rg 'CPU endpoint.*codec endpoint|source 72|pdma1.*source 150.*未' docs/peripherals/k3-audio.md` | codec endpoint 指向 `dp0/dp1`；当前 provider 为 source 72；`pdma1` 150 未被引用 | A1-A3 | PASS |

**Persisted Evidence**

None required — Plan Context 显式 `Mode: none`；所有 Gate 5 验证可低成本重跑并由本 Response 记录。

**Experience Candidates**

None — 本 Cycle 为纯文档静态聚合，未涉及可重复或高风险的操作路径；不构成 Runbook 候选；无显著影响或异常恢复的故障。

**Remaining Issues**

- 17-Audio Buildroot 正文仍 `partially-observed`（U4 保留）。
- PWM 30/20 数量冲突为 Iteration 000 已立条目（U2），不属本 Cycle 范围。
- RCPU 域 `ri2s0/1/2` 与 AP 域 `i2s0`–`i2s5` 的真板启用状态、codec 探测、IRQ/xrun、buffer 生命周期均保留为 U1-U4 未知项。
- CoM260 DTS 变体不唯一（G7），本轮仅记录观察到的 `&adma3` / `&ri2s3` / `&sound_card_dp1` / `&dp1` 启用事实，不选择默认板型。

**Commit or Diff Reference**

- 工作区变更：`docs/peripherals/k3-audio.md`（新增，20,821 字节 / 196 行）。
- 不提交本 Cycle；commit 留给 MS11 最终收尾 maintainer 同步。
- `git status --short` 本 Cycle 新增：`?? docs/peripherals/`（唯一未跟踪新增）；其他 `M` 与 `??` 来自 Iteration 000 预存状态。

## Plan Review

- Review Result: accepted

**Findings**

None。上一版三项 Important finding 均已修复，未发现新增阻塞项或 Minor finding。

**Deviation Classification**

None

**Acceptance Gaps**

None。A1-A4 均满足。

**Convergence**

reduced — 三项 Acceptance gap 全部关闭。

**Evidence**

- 独立复查正文：sound card 的 CPU endpoint 指向 `ri2sN`，codec endpoint 指向 `dp0/dp1`，外接模拟 codec 保持未观察边界；旧矛盾表述零命中。
- AP `i2s0`-`i2s5` 的完成路径现限定为 `&pdma` / APLIC source 72；`pdma1` source 150 明确未被本组节点引用；旧 72/150 汇总零命中。
- 首行已加入实际使用的 `k3-dp0.dtsi` supporting URL。
- 首行、8 个章节、AP/RCPU 实例、clock/reset、DMA、power domain、CoM260 候选、U1-U4 四字段和相对链接检查均通过；链接逐项 `test -e` 无缺失。
- 采信 Act Response 的 196 行结论。Plan 新鲜运行 `git diff --check`，退出 0；运行 `openspec validate establish-k3-general-peripheral-baseline --strict`，输出 `Change 'establish-k3-general-peripheral-baseline' is valid`，退出 0。
- Persisted Evidence 为 `none`；Blocker Handoff 为 None。

**Follow-up Decision**

接受当前 Cycle。上一版三项 gap 均在原 Task Contract 内收敛，A1-A4 已满足，无需当前 Cycle 修复或后继 Cycle；按既有 Iteration Map 展开 Iteration 002。

**Iteration Plan Update**

None

**Next Cycle**

None

**Next Iteration**

`../002-wdt-rtc/000-initial.md`
