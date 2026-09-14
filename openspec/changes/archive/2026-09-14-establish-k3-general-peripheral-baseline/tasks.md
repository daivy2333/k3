## 1. 来源与 GPIO/PWM/IR-RX

- [x] 1.1 更新 `docs/reference/source-coverage.md` 中六个 MS11 官方入口的当前职责和可达边界，保持 URL 唯一；验证六行不再是 `future / deferred`，源端修订 `unknown` 与无法直接取得正文的状态仍准确记录。
- [x] 1.2 创建 `docs/peripherals/k3-gpio-pwm-ir.md`，记录 GPIO controller/pinctrl/IRQ、PWM channel/pinmux/consumer 与 IR-RX controller/input 边界；验证首行来源、对象分层、PWM 30/20/DTS 冲突、CoM260 复用候选、四字段未知项及相对链接有效。

## 2. Audio

- [x] 2.1 创建 `docs/peripherals/k3-audio.md`，记录 I2S/SSPA controller、CPU DAI、sound card、codec/display endpoint、clock/reset、DMA、power domain 和 CoM260 引脚关系；验证静态链不被表述为 stream 成功，DMA/cache 通过 MS06 引用且四字段未知项完整。

## 3. WDT 与 RTC

- [x] 3.1 创建 `docs/peripherals/k3-wdt-rtc.md`，记录 WDT 计数/IRQ/restart、MMIO RTC 与 RPMI RTC 的资源和生命周期边界；验证静态节点不被表述为复位、掉电保持或 alarm 唤醒成功，两条 RTC 路径及四字段未知项分开记录。

## 4. 导航与汇总

- [x] 4.1 更新 `docs/reference/known-gaps.md`，把三篇正文未知项映射到 G7 和既有相关缺口，并仅在存在独立通用外设缺口时新增递增 G 条目；验证四字段、来源对应、状态汇总和影响主题一致。
- [x] 4.2 更新 `docs/reference/terminology.md`，增加正文实际使用且现有表缺失的 GPIO/PWM/IR/Audio/WDT/RTC 术语；验证主写法无重复，定义标明对象范围和正文位置。
- [x] 4.3 更新 `docs/index.md`，加入三篇通用外设正文入口并同步 `docs/peripherals/` 状态、来源数量和缺口计数；验证全部相对链接有效，计数与覆盖表及 known-gaps 一致，并运行 `git diff --check` 与 `openspec validate establish-k3-general-peripheral-baseline --strict`。

## Iteration Plan

### Iteration 000: 来源与 GPIO/PWM/IR-RX 基线

- Tasks: 1.1, 1.2
- Depends on: None
- Stable baseline: 六个 MS11 来源具有当前职责，GPIO、PWM 与 IR-RX 的控制器、复用、IRQ、consumer 和板级边界有独立正文。
- Verification boundary: 覆盖表六行状态和 URL 唯一性正确；`k3-gpio-pwm-ir.md` 覆盖对象分层、PWM 冲突、CoM260 复用候选和四字段未知项；相对链接有效。
- Diagnostic boundary: 来源状态、GPIO controller/pinctrl/IRQ、PWM channel/pinmux/consumer、IR-RX controller/input 和板级映射。
- Non-goals: 不写 Audio 或 WDT/RTC 正文，不更新最终术语、缺口和索引。
- Balance audit: 来源责任是首篇正文的前置条件，二者合并形成可独立复用的低速 I/O 基线；单独执行来源会留下无正文状态，纳入 Audio 会跨越流式 DMA 故障域。

### Iteration 001: Audio 基线

- Tasks: 2.1
- Depends on: Iteration 000
- Stable baseline: Audio controller、DAI、sound card、endpoint、DMA、power domain 和 CoM260 引脚关系具备独立正文。
- Verification boundary: `k3-audio.md` 覆盖静态端点链、clock/reset、DMA channel、显示音频和板级 I2S；静态资源与 stream 运行结果分离。
- Diagnostic boundary: I2S/SSPA、CPU DAI、codec/display endpoint、sound card、DMA/cache 和板级 route。
- Non-goals: 不执行音频播放/录音，不推定 codec 型号、采样参数或实时性能，不更新最终导航。
- Balance audit: Audio 虽只有一个任务，但有独立的流式 DMA、端点和 power domain 故障域；合入低速 I/O 或系统计时会混合验证边界。

### Iteration 002: WDT 与 RTC 基线

- Tasks: 3.1
- Depends on: Iteration 001
- Stable baseline: watchdog、MMIO RTC 和 RPMI RTC 的静态资源、状态和生命周期边界可独立检索。
- Verification boundary: `k3-wdt-rtc.md` 分开记录 WDT、MMIO RTC 与 RPMI RTC；不声明未经验证的 reset、timekeeping、掉电保持或 alarm 唤醒结果。
- Diagnostic boundary: WDT counter/clock/reset/IRQ/restart、MMIO RTC、RPMI mailbox/IRQ、VCC_RTC 和运行所有权。
- Non-goals: 不触发系统复位、alarm 或 suspend，不决定两条 RTC 路径的运行所有权，不更新最终导航。
- Balance audit: WDT 与 RTC 共享持续计时和系统生命周期边界，共同形成一个可验证成果；拆开会形成两个短小 Iteration，合入 Audio 则混合破坏性验证风险。

### Iteration 003: 导航、术语与缺口收敛

- Tasks: 4.1, 4.2, 4.3
- Depends on: Iteration 000, Iteration 001, Iteration 002
- Stable baseline: 三篇通用外设正文能从总索引进入，来源、术语、缺口和计数一致，MS11 可进入最终 Review。
- Verification boundary: 链接有效、术语无重复、缺口正文与汇总一致、索引计数准确、change 严格校验通过。
- Diagnostic boundary: `known-gaps.md`、`terminology.md`、`index.md` 三个汇总表面。
- Non-goals: 不新增通用外设技术事实，不刷新 SNAPSHOT/tasks，不修复 change 外问题。
- Balance audit: 三项共享全部正文完成后的收敛输入；提前执行会迫使未完成主题决定术语和计数，分别拆分又不能形成稳定交付。

## Requirements Traceability Matrix

| Requirement | Scenario | Design | Task | Iteration | Code Surface | Test Witness | Simplification | Status |
|---|---|---|---|---|---|---|---|---|
| R1 资源与板级分层 | 资源可证；映射不唯一 | D1, D2 | 1.1-3.1 | 000-002 | coverage；三篇 peripherals 正文 | deferred rows；目标文件缺失；资源/证据断言 | None | Covered |
| R2 GPIO 责任分离 | 静态资源；电气未知 | D4 | 1.2 | 000 | `k3-gpio-pwm-ir.md` | controller/pinctrl/IRQ 与非安全可用断言 | None | Covered |
| R3 PWM 冲突与 consumer | 数量冲突；板级候选 | D3, D4 | 1.2 | 000 | `k3-gpio-pwm-ir.md` | 30/20/PWM0–19、FAN/pinmux 与非运行断言 | None | Covered |
| R4 IR-RX 输入边界 | disabled；协议/映射缺失 | D4 | 1.2 | 000 | `k3-gpio-pwm-ir.md` | 两节点资源、disabled、unknown 断言 | None | Covered |
| R5 Audio 端点与所有权 | 静态链；codec/stream 未知 | D5 | 2.1 | 001 | `k3-audio.md`；DMA links | DAI/endpoint/DMA/power 与非 stream 断言 | None | Covered |
| R6 WDT/RTC 生命周期 | WDT 节点；双 RTC；掉电未知 | D6 | 3.1 | 002 | `k3-wdt-rtc.md` | 三路径分层、非 reset/alarm/保持断言 | None | Covered |
| R7 来源可追溯 | 可读；不可访问 | D2, D3 | 1.1, 4.1 | 000, 003 | coverage；known-gaps | URL 唯一、状态、冲突与四字段检查 | None | Covered |
| R8 导航一致 | 聚合完成；基线冲突 | D7 | 4.1-4.3 | 003 | gaps、terminology、index | 链接、计数、术语、严格校验 | None | Covered |

## Task Status

- Completed Iterations: 000-source-gpio-pwm-ir, 001-audio, 002-wdt-rtc, 003-navigation
- Current Iteration: None
- Current Cycle: None
- Deferred Iterations: None
- Persisted Evidence: none
