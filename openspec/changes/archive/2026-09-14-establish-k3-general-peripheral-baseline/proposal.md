## Why

MS03、MS04、MS05 和 MS06 已记录 K3/CoM260 的板级资源、平台控制、中断与 DMA 边界，但 GPIO、PWM、IR-RX、Audio、WDT 和 RTC 仍只有来源入口或散落事实。MS11 需要把六类通用外设的 SoC 能力、DTS 静态资源、CoM260 可达性和运行证据边界整理为可检索知识，避免把控制器存在、复用引脚或启用节点误写为真板功能可用。

本 change 由 SpacemiT K3 官方社区文档的 GPIO、PWM、IR-RX、Audio、WDT 和 RTC 章节驱动；覆盖表最近观察日期为 2026-09-02，源端修订均为 `unknown`。本 change 是既有资料的聚合与重组，不是已确认源端变化后的 refresh。

## What Changes

- 建立 GPIO、PWM 与 IR-RX 主题知识，区分 GPIO controller/pinctrl/IRQ、PWM channel/pinmux/consumer 和红外接收控制器/输入事件边界。
- 建立 Audio 主题知识，整理 I2S/SSPA、sound card、codec/显示音频、clock/reset、DMA、power domain 和 CoM260 引脚关系，并引用 MS06 的 DMA/cache 所有权边界。
- 建立 WDT 与 RTC 主题知识，区分 watchdog 计数与复位策略、RTC 计时与 alarm，以及 RPMI RTC 和 MMIO RTC 两条静态路径。
- 更新来源覆盖、总索引、术语和已知缺口，使六个官方入口、三篇正文及未确认事实具有唯一落点。
- 来源不可访问、PWM 数量冲突、CoM260 DTS 变体不唯一或运行结果未知时，保留冲突、未知项和解除条件，不补写未经证实的值。

### Approved Planning Assumptions

- 正文按用户批准的默认范围拆为 `k3-gpio-pwm-ir.md`、`k3-audio.md` 和 `k3-wdt-rtc.md`；调查若证明任一文件超过 M02 的 500 行建议上限，可在不改变需求范围的前提下继续拆分。
- 只聚合官方社区入口、官方 GitHub 文档、K3 datasheet、Linux DTS 与既有主题中的可审计事实和必要推论；不执行真板 I/O、音频流、红外输入、watchdog reset 或 RTC alarm 测试。
- 六个 Buildroot 对应页当前不可直接取得正文时，保留权威 URL 和 `partially-observed` 边界，以已直接核对的官方 datasheet、DTS 和产品资料作 supporting evidence，不用通用 Linux 或第三方实现补写 K3 专属行为。
- K3 datasheet 概览的 30 路 PWM 与详细章节的 20 路 PWM0–PWM19 并列记录；本 change 不以其中一个值静默覆盖另一个。
- Persisted Evidence 默认 `none`；可重跑的 Markdown、链接、计数和 OpenSpec 校验结果写入 Act Response。

### Gate 1 Approval

- Status: PASS
- User instruction: `批准`
- Approved scope: MS11 的 GPIO、PWM、IR-RX、Audio、WDT 和 RTC 知识基线，以及三篇正文拆分、未知项处理和 Non-goals。
- Approved at: 2026-09-12

### Non-goals

- 不实现或修改 GPIO、PWM、IR-RX、Audio、WDT、RTC、DMA、IRQ、clock、reset 或 pinctrl 驱动。
- 不设计 Rust 外设 API，不修改 StarryOS、Linux、Buildroot 或第三方仓库。
- 不执行会改变引脚、电平、音频设备、系统复位、时钟或 alarm 状态的真板操作。
- 不选择 G7 尚未唯一映射的 CoM260 顶层 DTS 作为默认目标。
- 不把 SoC 控制器数量、DTS 节点、复用引脚或电源输入存在解释为当前产品版本已经可用。
- 不裁决 PWM 数量冲突，也不扩展到 K3 之外的芯片、板卡或 SpacemiT 文档章节。

## Scenario Sketch

### S1：查询 GPIO 资源和中断边界

- 前置状态：K3 datasheet 或 DTS 提供 GPIO controller、bank、pinctrl、clock 或 IRQ 信息。
- 动作：读者查询 GPIO 输入、输出、边沿中断和 CoM260 复用引脚关系。
- 可观察结果：正文分开控制器资源、pinctrl 复用、GPIO consumer 和板级引出，并标明证据等级。
- 失败边界：缺少目标 DTS、引脚电气状态或真板输入输出证据时，不声明特定 GPIO 可安全使用。

### S2：查询 PWM 通道和板级用途

- 前置状态：datasheet、DTS 或 CoM260 产品资料描述 PWM channel、复用引脚或 FAN PWM。
- 动作：读者查询通道数量、寄存器资源、clock/reset、pinmux、频率/占空比能力和板级用途。
- 可观察结果：正文列出已证资源，并显式保留 30 路与 20 路 PWM0–PWM19 的来源冲突。
- 失败边界：复用引脚或 FAN_PWM 存在不能证明 Linux 节点启用、通道映射或波形已经验证。

### S3：查询 IR-RX 接收链

- 前置状态：DTS 提供 IR receiver 控制器，产品资料提供 IR_RX 复用候选。
- 动作：读者查询 MMIO、IRQ、clock/reset、pinmux 和输入事件边界。
- 可观察结果：正文区分控制器静态资源、板级候选引脚和未取得的协议/运行行为。
- 失败边界：节点 disabled、目标板映射不唯一或缺少按键事件日志时，不声明红外遥控可用。

### S4：查询 Audio 数据链和板级接口

- 前置状态：DTS 提供 I2S、sound card、DMA 或 display audio 节点，CoM260 资料列出 I2S 引脚。
- 动作：读者查询 CPU DAI、codec/显示端点、clock/reset、DMA channel、power domain 和板级连接。
- 可观察结果：正文按控制器、DAI/link、DMA、codec/endpoint 和连接器分层，并引用既有 DMA/cache 基线。
- 失败边界：静态 sound card 或 I2S pin 存在不能证明 codec 型号、采样格式、音频流或实时性能。

### S5：查询 WDT 与 RTC 生命周期边界

- 前置状态：datasheet 或 DTS 描述 watchdog、MMIO RTC 或 RPMI RTC。
- 动作：读者查询计数源、clock/reset、IRQ、restart、timekeeping、alarm 和固件代理关系。
- 可观察结果：正文分开 WDT 监控/复位、MMIO RTC 和 RPMI RTC，并记录启用状态与可观察字段。
- 失败边界：静态节点或能力声明不能证明超时复位范围、alarm 唤醒、掉电保持或两条 RTC 路径的运行所有权。

### S6：来源、导航和兼容性收敛

- 前置状态：三篇正文完成，或官方页面不可访问、资料相互冲突。
- 动作：聚合者更新来源覆盖、术语、缺口和总索引。
- 可观察结果：URL 唯一、正文入口有效、术语和缺口计数一致，官方事实、supporting evidence、推论和未知项分级明确。
- 失败边界：新材料与既有基线冲突时停止静默覆盖，返回 Plan 判断是否属于本 change 或独立 refresh。

## Capabilities

### New Capabilities

- `k3-general-peripheral-baseline`: 规定 K3 GPIO、PWM、IR-RX、Audio、WDT 和 RTC 的控制器资源、板级连接、数据或状态边界、证据等级及导航要求。

### Modified Capabilities

- 无。

## Impact

- 预计新增 `docs/peripherals/` 下三篇主题文档，并修改 `docs/index.md`、`docs/reference/source-coverage.md`、`docs/reference/known-gaps.md` 与 `docs/reference/terminology.md`。
- 新增 change delta spec；不修改可执行代码、API、构建系统或运行时依赖。
- 复用 R04–R06、R08、MS03–MS06 和 G7；M01 指向的官方来源仍是硬件事实权威。
