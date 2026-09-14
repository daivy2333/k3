> 来源: https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/02-GPIO.md, https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/03-PWM.md, https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/04-IR-RX.md（源端修订: unknown；观察日期: 2026-09-02）；supporting: https://github.com/spacemit-com/docs-chip/blob/main/zh/key_stone/k3/k3_docs/k3_ds.md（源端修订: 2026-08-25 V1.8；观察日期: 2026-09-07）, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-pinctrl.dtsi, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260.dtsi, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260.dts, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260_kit_v02.dts, https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/eco/k3_com260/com260_ds.md（源端修订: branch k3-br-v1.0.y / unknown；观察日期: 2026-09-07）

# K3 GPIO、PWM 与 IR-RX 控制器及板级边界

本文按控制器、pinctrl、IRQ、复用、consumer 和板级候选分层整理 K3 GPIO、PWM 与 IR-RX。平台 provider 与 pinctrl 责任见[平台控制资源](../platform/k3-platform-control.md)，中断拓扑见[中断与时间](../interrupts/k3-interrupt-and-time.md)，CoM260 板级资源见[CoM260 板级资源](../platform/com260-board-resources.md)，DTS 候选范围见[镜像与 DTS](../boot/com260-image-and-dts.md)，术语见[术语表](../reference/terminology.md)，缺口见[已知缺口](../reference/known-gaps.md)。

## 1. 范围与证据

- **官方事实**：K3 datasheet 描述 GPIO、PWM 与 IR-RX 等 SoC 能力。三个 Buildroot 对应页（02-GPIO、03-PWM、04-IR-RX）正文未直接取得，不能由标题补写驱动操作。
- **交叉验证**：SpacemiT Linux 6.18 `k3-br-v1.0.y` 的 `k3.dtsi`、`k3-pinctrl.dtsi` 与 CoM260 DTS 提供静态节点、bank、clock/reset、IRQ、复用与 pinctrl 字段。
- **推论**：只在多个已列事实共同支持时使用，不替代缺失的板级或运行证据。
- **未知项**：本项目没有执行 GPIO 边沿输入、PWM 波形采集、IR 接收解码或板级 probe。

SoC 能力、DTS 节点、模组引脚、Kit 连接器与运行结果属于不同层。任一层存在都不自动证明下一层成立。

## 2. GPIO 控制器、pinctrl 和 IRQ

### 2.1 控制器与节点字段

| 字段 | 值 | 等级 |
|---|---|---|
| 节点 | `gpio@d4019000` | 交叉验证 |
| base | `0xd4019000/0x100` | 交叉验证 |
| compatible | `spacemit,k3-gpio` | 交叉验证 |
| clocks | `core` + `bus` | 交叉验证 |
| APLIC IRQ | source 58（兼 interrupt-controller） | 交叉验证 |
| `gpio-ranges` | 覆盖 4 × 32 | 交叉验证 |
| 角色 | 兼作 GPIO controller 与 interrupt controller | 交叉验证 |

`spacemit,k3-gpio` 字符串只说明软件绑定，不证明 K3 GPIO 硬件全集与 K1 相同。

### 2.2 pinctrl 复用与电气

[平台控制资源](../platform/k3-platform-control.md)已记录 AP/APBC2/APMU 域 pad/function/driver 责任与 K3_PADCONF 模式；pinctrl 是独立节点，APLIC source 60，不与 GPIO controller 合并。`k3-pinctrl.dtsi` 内 `&pinctrl` 节点提供多组复用配置（K3_PADCONF macro），覆盖 UART、GPIO、PWM、IR-RX 等候选。

复用声明不证明目标 DTS 已选择该功能，也不证明该 pin 在 Kit 上已引出或电气匹配。CoM260 datasheet 金手指表含 GPIO 复用候选与电压域；只证明 pinmux/引出候选。

### 2.3 中断与消费

GPIO controller 自身通过 APLIC source 58 注册为 interrupt-controller；GPIO line 边沿事件由此 APLIC 源进入 AP 域。Bank 的 line-to-IRQ 映射、debounce、wakeup 与 suspend 行为未由当前证据展开。

GPIO consumer（按键、LED、PHY reset、Type-C、Hub 等）通过 `&gpio` 引用并指定 bank/line 字段；consumer 存在不证明目标 Kit 已配置为输入或输出。

K3 datasheet GPIO 章节记：复位默认输入、独立置位/清除/读取、双边沿中断能力。Bank 数量、line range、电气参数（驱动强度、上下拉、电压域、debounce 时间）按 K3 PADCONF macro 与 bank 编号展开；本仓库未直接展开 datasheet 完整引脚表。

### 2.4 不能推出的结论

- 不得宣称某 GPIO line 可安全驱动外部器件。
- 不得由 `spacemit,k3-gpio` 反推 K1 边界相同。
- 不得由 datasheet GPIO 章节推定完整 bank 编号、line range 或电气参数。
- 不得由 pinctrl 复用候选推定该 mux 已被目标 DTS 选中或 pin 在 Kit 上可访问。

## 3. PWM 通道、复用与数量冲突

### 3.1 SoC 能力与三种观察

| 观察来源 | 数量或描述 | 等级 | 边界 |
|---|---|---|---|
| K3 datasheet 概览章节 | 30 路 PWM | 官方事实 | 概览数字未在详细章节复述，不能用以覆盖章节定义 |
| K3 datasheet PWM 详细章节 | 20 个独立通道 `PWM0`–`PWM19`，195.3 Hz–12.8 MHz，6-bit divider，10-bit period counter，15-bit pulse counter | 官方事实 | 详细章节给出频率范围与 counter 位宽；不证明 PWM0–PWM19 全部引出 |
| 官方 `k3.dtsi` | `pwm0`–`pwm19` 共 20 个节点，使用 `spacemit,k1-pwm` + `marvell,pxa910-pwm` 双兼容，3-cell specifier，独立 base、func/bus clocks、reset，默认 `disabled`；未观察到 `pwm20` 及以上 | 交叉验证 | DTS 节点集合等于 SoC 静态定义，不等于板级启用 |

三种观察并列保留：本 change 不以 30 或 20 静默覆盖对方。DTS 的 20 路与 datasheet 详细章节一致，但与概览的 30 路存在数量冲突；解除需独立 refresh change。

### 3.2 节点字段示例（PWM0–PWM19 通用）

| 字段 | 值 | 等级 |
|---|---|---|
| compatible | `spacemit,k1-pwm`, `marvell,pxa910-pwm` | 交叉验证 |
| clocks | `func` + `bus` | 交叉验证 |
| resets | per-channel reset line | 交叉验证 |
| `#pwm-cells` | `3` | 交叉验证 |
| 默认状态 | `disabled` | 交叉验证 |

PWM IP 形态与寄存器全集未由当前证据展开；K3 datasheet PWM 章节给出频率范围与 counter 位宽，但完整寄存器布局、errata 与 bridge 行为需要 programmer manual。

### 3.3 复用与 consumer

复用候选：CoM260 datasheet 金手指表含多组 PWM/IR/GPIO 复用候选与电压域，并单列 `FAN_PWM`。这些只证明 pinmux/引出候选，不证明目标 DTS 已选择 PWM 模式，也不证明通道已被 Linux 节点启用或波形已被验证。

Consumer：FAN/背光/蜂鸣器等典型 PWM consumer 通过引用 `&pwmX` 节点使用通道。`status = "okay"` 与外接电路共同决定波形能否输出；本项目不直接观察外接电路或采集波形。

`spacemit,k1-pwm` + `marvell,pxa910-pwm` 双兼容只说明软件绑定，不证明 K3 PWM 寄存器全集与 PXA 相同。

### 3.4 不能推出的结论

- 不得宣称 30 路全部可用或 20 路全部引出。
- 不得由 `spacemit,k1-pwm` + `marvell,pxa910-pwm` 推定 K3 PWM 等同 PXA 全部行为。
- 不得由 `FAN_PWM` 引出推定 fan 调速已运行。
- 不得由 `status = "okay"` 推定具体占空比或频率。
- 不得裁决 30/20 数量冲突。

## 4. IR-RX 控制器与输入边界

### 4.1 节点与字段

| 节点 | base | compatible | APLIC source | clock | reset | 状态 |
|---|---|---|---|---|---|---|
| `ircrx0@d4017e00` | `0xd4017e00` | `spacemit-k1,irc` | 69 | 102.4 MHz | 独立 reset | `disabled` |
| `ircrx1@d4017f00` | `0xd4017f00` | `spacemit-k1,irc` | 20 | 102.4 MHz | 独立 reset | `disabled` |

两个节点并列提供，未观察到 `ircrx2` 及以上。`spacemit-k1,irc` 字符串只说明软件绑定，不证明 K3 IR-RX 硬件全集与 K1 相同。

### 4.2 输入事件与协议

IR-RX 接收端把红外信号解码为输入事件。当前证据未包含协议（NEC / RC5 / SIRC / 其他）、keymap、采样窗口或输入事件类型（`KEY_*` / `SW_*` / `MSC_*`）的描述，也不包含目标 Linux 内核 IR 子系统（`rc-core` / `lirc`）的注册路径。

`status = "disabled"` 表示 DTS 默认不启用该节点；本项目不直接观察 `status` 变化或运行时输入事件。

### 4.3 板级候选与运行边界

[CoM260 板级资源](../platform/com260-board-resources.md)记录 IR_RX 复用候选与连接器。复用声明不证明：

- 目标 DTS 已将 IR-RX 节点设为 `okay`；
- 遥控器型号、协议、按键映射已知；
- 真板已采集到输入事件或解码成功。

## 5. 错误边界与未知项

### U1：GPIO bank 与 line 范围的 K3 完整索引

- 当前证据：`gpio@d4019000` 提供 `gpio-ranges` 覆盖 4 × 32；K3 datasheet GPIO 章节列复位默认输入、双边沿与独立 set/clr/read 能力；本项目未直接展开 datasheet 完整 bank 编号、line range 与电气参数。
- 禁止推断：不得由 `gpio-ranges` 反推所有 bank 编号；不得由其他 SpacemiT 平台反推 K3 完整索引；不得推定 `4 × 32` 即是 K3 最终 bank 边界。
- 解除条件：取得 K3 datasheet 完整 GPIO 章节（含 bank 编号、line range、电气参数、reset 行为）；或取得 K3 公开 programmer manual GPIO 章节。
- 影响主题：本文件 GPIO 主题；[平台控制资源](../platform/k3-platform-control.md) `k3-pinctrl.dtsi` 复用候选完整性。

### U2：PWM 30/20 数量冲突的来源裁决

- 当前证据：K3 datasheet 概览 30 路；详细章节与 `k3.dtsi` 均为 20 路 `PWM0`–`PWM19`。
- 禁止推断：不得用 30 静默覆盖 20，也不得用 20 静默覆盖 30；不得因 `k3.dtsi` 仅含 20 路就推定概览数字错误。
- 解除条件：取得 K3 datasheet 同版本统一答复；或 buildroot 03-PWM.md 正文直接确认通道数与通道映射；或独立 refresh change 取得官网修订。
- 影响主题：本文件 PWM 主题；[CoM260 板级资源](../platform/com260-board-resources.md) PWM/FAN 板级可达性。

### U3：IR-RX 协议、keymap 与真板解码证据

- 当前证据：`ircrx0/1` 节点默认 `disabled`；CoM260 datasheet 金手指表列出 IR_RX 复用候选。
- 禁止推断：不得由节点 `disabled` 反推硬件不存在；不得由节点存在反推协议或 keymap 已知；不得在真板解码未观察时宣称红外遥控可用。
- 解除条件：取得 buildroot 04-IR-RX.md 正文或 K3 公开 IR 寄存器手册；直接打开目标 Linux `rc-core` / `lirc` 绑定；真板采集到输入事件并能识别协议。
- 影响主题：本文件 IR-RX 主题；[CoM260 板级资源](../platform/com260-board-resources.md) IR 板级可达性。

### U4：Buildroot 02-GPIO / 03-PWM / 04-IR-RX 正文内容

- 当前证据：三个 Buildroot 官网入口已登记；正文在 2026-09-12 未能直接取得（`raw.githubusercontent.com` DNS 解析失败），访问失败不是页面不存在。
- 禁止推断：不得依据标题或通用 Linux/Buildroot 文档补写 K3 专属命令、driver path、测试步骤、波形采集或解码流程。
- 解除条件：直接打开对应官网正文或 SpacemiT 官方 GitHub 等价页，并核对源端修订和适用板型。
- 影响主题：本文件全部三个主题的官方事实等级、软件行为和验证入口。

### U5：GPIO/PWM/IR-RX 的真板运行证据

- 当前证据：K3 datasheet 与 `k3.dtsi` / `k3-pinctrl.dtsi` / CoM260 DTS 提供静态资源；本项目未执行 GPIO 边沿输入、PWM 波形采集或 IR 接收解码。
- 禁止推断：不得依据 pinctrl 复用候选或 DTS 节点 `okay` 状态宣称具体功能在真板已工作；不得依据 CoM260 datasheet 金手指表引脚推定板级电平匹配。
- 解除条件：取得目标 Kit 的真板 probe、波形、边沿、输入事件或解码日志；或 buildroot 对应章节提供运行案例。
- 影响主题：本文件全部三个主题的运行边界与 [CoM260 板级资源](../platform/com260-board-resources.md) 板级可达性。

## 6. 主题边界

- Audio、WDT、RTC 由后续 `k3-audio.md`、`k3-wdt-rtc.md` 承担，本文件不重复定义。
- pinctrl、clock、reset、APBC/CCU provider 见 [平台控制资源](../platform/k3-platform-control.md)；AP/RP 中断与 timer 见 [中断与时间](../interrupts/k3-interrupt-and-time.md)；DTS 候选与映射见 [镜像与 DTS](../boot/com260-image-and-dts.md)。
- 术语、缺口与总索引在 MS11 最终 Iteration 统一收敛，本轮不提前修改。
