> 来源: https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/23-WDT.md, https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/24-RTC.md（源端修订: unknown；观察日期: 2026-09-02）；supporting: https://github.com/spacemit-com/docs-chip/blob/main/zh/key_stone/k3/k3_docs/k3_ds.md（源端修订: 2026-08-25 V1.8；观察日期: 2026-09-12）, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-rdomain.dtsi, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260.dtsi, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260.dts, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260_kit_v02.dts, https://github.com/spacemit-com/docs-product/blob/main/zh/k3_com260/com260_ds.md（源端修订: branch k3-br-v1.0.y / unknown；观察日期: 2026-09-07）

# K3 WDT、MMIO RTC 与 RPMI RTC 控制器及生命周期边界

本文按 SoC WDT / RTC 能力、WDT DTS 资源、MMIO RTC 静态资源、RPMI RTC 固件代理与 mailbox 依赖、VCC_RTC 板级电源，以及三条路径的运行边界和生命周期约束分层整理 K3 Watchdog 与 RTC。平台 provider 与 clock/reset 见[平台控制资源](../platform/k3-platform-control.md)，中断拓扑与 APLIC source 见[中断与时间](../interrupts/k3-interrupt-and-time.md)，AP/RP 生命周期与 mailbox 通知见[AMP 共享内存生命周期](../amp/k3-amp-shared-memory-lifecycle.md)与[RPC ring 通知](../amp/k3-rpc-ring-notification.md)，CoM260 板级资源见[CoM260 板级资源](../platform/com260-board-resources.md)，DTS 候选范围见[镜像与 DTS](../boot/com260-image-and-dts.md)，术语见[术语表](../reference/terminology.md)，缺口见[已知缺口](../reference/known-gaps.md)。

## 1. 范围与证据

- **官方事实**：K3 datasheet 描述 K3 SoC WDT 与 RTC 子系统能力。Buildroot 23-WDT.md 与 24-RTC.md 是当前 MS11 主题的权威入口；正文在当前环境未能直接取得（`raw.githubusercontent.com` DNS 解析失败），不依据标题补写驱动步骤、用户接口或真板验证。
- **交叉验证**：SpacemiT Linux 6.18 `k3-br-v1.0.y` 的 `k3.dtsi`（AP 域）、`k3-rdomain.dtsi`（RCPU 域）、`k3_com260.dtsi` / `k3_com260.dts` / `k3_com260_kit_v02.dts`（CoM260 模组与 Kit 变体）提供 watchdog、MMIO RTC、RPMI RTC 节点、clock/reset、APLIC IRQ、restart 属性和 status 字段。
- **推论**：只在多个已列事实共同支持时使用，不替代缺失的 programmer reference、reset 范围、alarm 行为、掉电保持或 RPMI mailbox 服务实现。
- **未知项**：本项目没有执行 watchdog 喂狗、超时复位、RTC alarm 唤醒、掉电保持或 RPMI RTC 时间同步，也没有读取 RPMI service 0xe 的实现源码。

SoC 能力、DTS 节点、模组电源、固件代理与运行结果属于不同层。任一层存在都不自动证明下一层成立。`status = "okay"`、`spa,wdt-enable-restart-handler` 属性、`mpxy_mbox` 引用与 VCC_RTC 引脚存在不证明 watchdog 计时、复位、alarm、timekeeping、掉电保持或 RPMI 服务已经运行。

## 2. SoC WDT 与 RTC 能力

K3 datasheet 把 WDT 与 RTC 描述为独立子系统：

| 子系统 | datasheet 描述 | 关键参数 | 等级 |
|---|---|---|---|
| WatchDog | 6 个 24-bit WDT，输入时钟 256 Hz | WatchDog Reset 描述为除 pinmux / debug 寄存器外的全芯片复位；当前 `k3.dtsi` 仅观察到 1 个 `watchdog` 节点 | 官方事实（datasheet §WDT / §Reset） |
| RTC | 32.768 kHz RTC clock | datasheet 的当前核对范围不支持补写 RTC 寄存器布局、timekeeping、alarm 精度或掉电保持范围 | 官方事实（datasheet Clock & Reset） |

datasheet 描述与官方 DTS 静态资源对应：`k3.dtsi` AP 域含 1 个 `watchdog` 节点（§3），1 个 MMIO `rtc` 节点（§4），1 个 `rpmi_rtc` 节点（§5）。`spacemit-k1,wdt` 字符串只说明软件绑定，不证明 K3 WDT 寄存器全集与 K1 相同。`mrvl,mmp-rtc` 字符串只说明软件绑定，不证明 K3 MMIO RTC 寄存器全集与 MMP 相同。`riscv,rpmi-rtc` 字符串只说明 RPMI 协议族绑定，不证明 RPMI service 0xe 在 K3 上由哪一固件、哪一时钟源驱动。

datasheet 6 个 WDT 与 DTS 单个 `watchdog` 节点属于不同证据层：datasheet 描述 SoC 集成视角，DTS 描述当前 Linux 设备树视角。两者均不证明 6 个 WDT 都已经具备 Linux 节点或其中 1 个是默认启用实例。

## 3. WDT DTS 资源与属性

`k3.dtsi` AP 域包含 1 个 watchdog 节点，标签 `watchdog`（`watchdog@d4014000`），`status = "okay"`。

| 字段 | 值 | 等级 |
|---|---|---|
| base | `0xd4014000`，两段 reg | 交叉验证 |
| compatible | `spacemit-k1,wdt` | 交叉验证 |
| clocks | `&syscon_apbc CLK_APBC_TIMERS0`（func）+ `&syscon_apbc CLK_APBC_TIMERS0_BUS`（bus） | 交叉验证 |
| resets | `&syscon_apbc RESET_APBC_TIMERS0` | 交叉验证 |
| APLIC IRQ | source 35 | 交叉验证 |
| 属性 | `spa,wdt-disabled`、`spa,wdt-enable-restart-handler` | 交叉验证 |
| 状态 | `okay` | 交叉验证 |

`spa,wdt-disabled` 与 `spa,wdt-enable-restart-handler` 是软件配置字段，名称提示默认行为，但当前证据不包含其语义细节、是否由 driver 解析、是否与 sysfs / device tree attribute 接口暴露。`spacemit-k1,wdt` 字符串只说明软件绑定，不证明 K3 WDT 寄存器全集与 K1 相同；当前 K3 datasheet 不展开 watchdog 寄存器布局、errata 与 bridge 行为。

datasheet §Reset 把 WatchDog Reset 描述为除 pinmux / debug 寄存器外的全芯片复位；该描述不证明：

- Linux 当前已启用 watchdog 计时；
- 喂狗路径、timeout 配置、超时复位范围和复位向量已经实测；
- restart handler 由 Linux 注册并由 BootROM / U-Boot 处理。

`k3_com260.dtsi` / `k3_com260.dts` / `k3_com260_kit_v02.dts` 当前未观察到 `watchdog` 节点字段改写或新增 watchdog 节点；节点继承自 `k3.dtsi`。`k3-rdomain.dtsi` 当前未观察到 RCPU 域 watchdog 节点。

## 4. MMIO RTC 静态资源

`k3.dtsi` AP 域包含 1 个 MMIO RTC 节点，标签 `rtc`（`rtc@d4010000`），`status = "okay"`。

| 字段 | 值 | 等级 |
|---|---|---|
| base | `0xd4010000`，长度 `0x100` | 交叉验证 |
| compatible | `mrvl,mmp-rtc` | 交叉验证 |
| clocks | `&syscon_apbc CLK_APBC_RTC`（func）+ `&syscon_apbc CLK_APBC_RTC_BUS`（bus） | 交叉验证 |
| resets | `&syscon_apbc RESET_APBC_RTC` | 交叉验证 |
| APLIC IRQ | source 21（1 Hz tick）；source 22（alarm） | 交叉验证 |
| 状态 | `okay` | 交叉验证 |

`mrvl,mmp-rtc` 字符串只说明软件绑定，不证明 K3 MMIO RTC 寄存器全集与 MMP 相同。APLIC source 21 / 22 分别对应 1 Hz tick 与 alarm；这不证明：

- 1 Hz tick 已注册到 Linux clockevents 或 clocksource；
- alarm 路径已经配置并能唤醒系统；
- timekeeping 精度、漂移与掉电保持范围已经实测；
- Linux 当前选择 MMIO RTC 而非 RPMI RTC 作为系统 RTC，或反之。

`k3_com260.dtsi` / `k3_com260.dts` / `k3_com260_kit_v02.dts` 当前未观察到 `rtc` 节点字段改写或新增 RTC 节点；节点继承自 `k3.dtsi`。`k3-rdomain.dtsi` 当前未观察到 RCPU 域 MMIO RTC 节点。

## 5. RPMI RTC 固件代理与 mailbox 依赖

`k3.dtsi` AP 域包含 1 个 RPMI RTC 节点，标签 `rpmi_rtc`（`rpmi_rtc@0`），`status = "okay"`。

| 字段 | 值 | 等级 |
|---|---|---|
| 节点名 | `rpmi_rtc@0` | 交叉验证 |
| compatible | `riscv,rpmi-rtc` | 交叉验证 |
| mbox | `&mpxy_mbox` service `0xe`，endpoint `0` | 交叉验证 |
| APLIC IRQ | source 64（与 RPMI pwrkey 共享） | 交叉验证 |
| 状态 | `okay` | 交叉验证 |

RPMI RTC 不直接是 MMIO 设备：静态 DTS 显示它通过 `mpxy_mbox` 向 AP 暴露 RPMI RTC client；实现 service `0xe` 的固件及所属域仍未知。DTS 节点存在不证明：

- RCPU firmware 已提供 service `0xe` 的 RTC 实现；
- service `0xe` 实际使用的时钟源、alarm 路径和掉电保持与 MMIO RTC 相同或不同；
- Linux timekeeping、alarm 与 `/dev/rtc` 接口当前使用 RPMI RTC 而非 MMIO RTC，或反之；
- mailbox 投递、消息分派、超时与错误恢复契约。

APLIC source 64 是 RPMI 共享 IRQ：当前证据显示 RPMI RTC 与 RPMI pwrkey 共用此 source。这不证明：

- RPMI 中断分派由 service ID 区分；
- RTC 与 pwrkey 的中断分派顺序、mask/ack 路径与 throttle 策略；
- 哪一 service 拥有 RTC timekeeping 所有权。

`mpxy_mbox` 与 `riscv,rpmi-rtc` 字符串只说明软件绑定，不证明 RPMI 协议在 K3 上的实现与上游 RISC-V RPMI specification 完全一致。

## 6. VCC_RTC 板级电源

CoM260 datasheet 在模组金手指表中列出 `VCC_RTC`：

| 字段 | 值 | 等级 |
|---|---|---|
| 引脚 | 235 | 交叉验证（com260_ds.md） |
| 标称电压 | 5 V | 交叉验证（com260_ds.md） |
| 推荐范围 | 1.85 V – 5.5 V | 交叉验证（com260_ds.md） |

`VCC_RTC` 引脚存在只说明模组向 Kit 底板暴露 RTC 电源通路。该引脚存在不证明：

- Kit 底板为该引脚提供独立电池或 supercap 保持；
- 实际保持时间、掉电保持范围与掉电后恢复时间已经实测；
- MMIO RTC 与 RPMI RTC 中哪一路径使用该电源或两者共用；
- 当 `VCC_RTC` 断电时，alarm 唤醒与 timekeeping 是否仍然有效。

当前 `k3_com260.dts` / `k3_com260_kit_v02.dts` 未观察到 RTC 电源域相关绑定（如 `power-domains`、`vcc-rtc-supply` 字段）；该保持电源在 Linux 设备树视角的归属未直接观察到。

## 7. 错误边界与未知项

### U1：WDT reset 范围、timeout 与 restart handler 行为

- 当前证据：`watchdog@d4014000` 节点 `status = "okay"`，含 `spa,wdt-disabled` 与 `spa,wdt-enable-restart-handler` 属性；datasheet §Reset 把 WatchDog Reset 描述为除 pinmux / debug 寄存器外的全芯片复位；datasheet WDT 章节描述 6 个 24-bit WDT 与 256 Hz 输入时钟。
- 禁止推断：不得由 `spa,wdt-enable-restart-handler` 名称推定 Linux 已经注册 restart handler；不得由 datasheet 全芯片复位描述推定复位范围与 pinmux / debug 保留范围已经实测；不得由 256 Hz 输入时钟推定 1 秒 timeout 配置或实际 timeout 周期；不得由 `status = "okay"` 推定 watchdog 当前已启动计时、喂狗或能触发复位。
- 解除条件：取得 K3 WDT 公开 programmer reference（寄存器布局、timeout 公式、restart 行为）；或 buildroot 23-WDT.md 正文给出 WDT 节点、timeout、driver 路径与 restart 流程；或真板 watchdog reset 实测（包含复位前后寄存器 dump、BootROM / U-Boot 接管行为）。
- 影响主题：本文件 WDT 主题；[CoM260 板级资源](../platform/com260-board-resources.md) reset 路径；[平台控制资源](../platform/k3-platform-control.md) APBC/CCU provider 与 WDT clock / reset 关系。

### U2：MMIO RTC 寄存器、timekeeping、alarm 与掉电保持

- 当前证据：`rtc@d4010000` 节点 `status = "okay"`，含 base、1 Hz / alarm APLIC IRQ、func/bus clocks 与 reset；`mrvl,mmp-rtc` 字符串只说明软件绑定；CoM260 datasheet 列出 `VCC_RTC` 引脚 235 / 5 V / 1.85–5.5 V。
- 禁止推断：不得由 `mrvl,mmp-rtc` 推定 K3 MMIO RTC 寄存器全集与 MMP 相同；不得由 source 21（1 Hz）推定当前已经注册到 Linux clockevents 或 clocksource；不得由 source 22（alarm）推定 alarm 路径已经配置；不得由 `VCC_RTC` 引脚存在推定掉电保持已验证；不得由 `status = "okay"` 推定 Linux 选择 MMIO RTC 而非 RPMI RTC。
- 解除条件：取得 K3 MMIO RTC 公开 programmer reference（寄存器布局、timekeeping 公式、alarm 行为、掉电域）；或 buildroot 24-RTC.md 正文给出 MMIO RTC 节点、driver 路径与 alarm 流程；或真板 `hwclock` / `/dev/rtc` / 掉电保持实测。
- 影响主题：本文件 MMIO RTC 主题；[CoM260 板级资源](../platform/com260-board-resources.md) VCC_RTC 板级连接；[平台控制资源](../platform/k3-platform-control.md) APBC provider 与 RTC clock / reset 关系。

### U3：RPMI RTC 固件实现、service 0xe 行为与运行所有权

- 当前证据：`rpmi_rtc@0` 节点 `status = "okay"`，经 `mpxy_mbox` service `0xe` / endpoint `0` 暴露；APLIC source 64 与 RPMI pwrkey 共享；`riscv,rpmi-rtc` 字符串只说明软件绑定。
- 禁止推断：不得由 `mpxy_mbox` 引用推定 RCPU firmware 已经提供 service `0xe` 的 RTC 实现；不得由 source 64 共享推定 RPMI 中断分派由 service ID 区分；不得由节点 `okay` 推定 Linux timekeeping、alarm 与 `/dev/rtc` 当前使用 RPMI RTC；不得由 `mpxy_mbox` 引用推定 mailbox 投递、消息分派、超时与错误恢复契约已验证。
- 解除条件：取得 RCPU firmware 中 service `0xe` 的 RTC 实现源码或日志；或 buildroot 24-RTC.md 正文给出 RPMI RTC 节点、driver 路径与 RPMI 协议；或真板 RPMI RTC `hwclock` / `/dev/rtc` 实测；或 RISC-V RPMI specification 公开 K3 适用版本。
- 影响主题：本文件 RPMI RTC 主题；[AMP 共享内存生命周期](../amp/k3-amp-shared-memory-lifecycle.md) RCPU 域初始化；[RPC ring 通知](../amp/k3-rpc-ring-notification.md) RPMI mailbox 投递；[中断与时间](../interrupts/k3-interrupt-and-time.md) APLIC source 64 共享分派。

### U4：Buildroot 23-WDT / 24-RTC 正文内容与 K3 WDT/RTC 完整手册

- 当前证据：两个 Buildroot 官网入口已登记；正文在 2026-09-12 未能直接取得（`raw.githubusercontent.com` DNS 解析失败），访问失败不是页面不存在。
- 禁止推断：不得依据标题或通用 Linux / Buildroot 文档补写 K3 WDT / RTC 专属 driver path、sysfs / ioctl 接口、`hwclock` 调用、watchdog daemon 配置或 alarm 唤醒脚本。
- 解除条件：直接打开对应官网正文或 SpacemiT 官方 GitHub 等价页，并核对源端修订和适用板型；或取得 K3 公开 WDT / RTC programmer manual。
- 影响主题：本文件全部 WDT / RTC 子主题的官方事实等级、软件行为和验证入口。

## 8. 主题边界

- GPIO、PWM、IR-RX 由 [k3-gpio-pwm-ir.md](k3-gpio-pwm-ir.md) 承担；Audio 由 [k3-audio.md](k3-audio.md) 承担，本文件不重复定义。
- pinctrl、clock、reset、APBC/CCU provider 见[平台控制资源](../platform/k3-platform-control.md)；AP/RP 中断与 timer 见[中断与时间](../interrupts/k3-interrupt-and-time.md)；AP/RP 生命周期与 mailbox 通知见[AMP 共享内存生命周期](../amp/k3-amp-shared-memory-lifecycle.md)与[RPC ring 通知](../amp/k3-rpc-ring-notification.md)；DTS 候选与映射见[镜像与 DTS](../boot/com260-image-and-dts.md)。
- 术语、缺口与总索引在 MS11 最终 Iteration 统一收敛，本轮不提前修改。
