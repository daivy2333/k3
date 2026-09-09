> 来源: https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/Timer.md（源端修订: unknown；观察日期: 2026-09-02）; https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-07）; https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-rdomain.dtsi（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-08）; https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/Documentation/devicetree/bindings/interrupt-controller/riscv,aplic.yaml（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-08）; https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/Documentation/devicetree/bindings/interrupt-controller/riscv,imsics.yaml（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-08）; https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/Documentation/devicetree/bindings/timer/sifive,clint.yaml（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-08）

# K3 SoC 中断、时间和通知机制

> 范围: SpacemiT Key Stone K3 SoC 的中断投递（interrupt delivery）、时间源（time source）和核间通知（notification）三类机制的事实记录。不覆盖 CoM260 模组引脚、Kit 板级信号、寄存器位细节、驱动端 IRQ handler 路径。
> 边界: SoC 中断/时间/通知事实 ≠ CoM260 引出或启用。引出与启用清单见 [`com260-board-resources.md`](../platform/com260-board-resources.md)。本主题不展开 mailbox 协议细节与寄存器偏移，mailbox 文档由 Iter 001 / T3 在 `com260-mailbox-notification.md` 单独建立。
>
> 来源身份: Timer.md 官方页仍为 Vue SPA 壳，本节 Timer 内容以 `未知项` 处理；K3 中断/时间事实以官方 linux-6.18 仓库 `k3-br-v1.0.y` 分支的 `k3.dtsi` + `k3-rdomain.dtsi` + 三个 binding 文件为正文来源，每条事实按 D06 标 `交叉验证`；不把 GitHub 仓库或第三方分析升级为 `官方事实`。RP 域 PLIC / SysTimer / MSIP / AON timer 节点在 `k3-rdomain.dtsi` 当前未直接暴露，仅在 R10–R12 等第三方分析中以 `spacemit,k3-systimer` / `spacemit,k3-systimer-msip` 兼容串出现，列入 §5 未知项。
> 修订快照: 本文档基于覆盖表登记的 linux-6.18 `k3-br-v1.0.y` 分支 `k3.dtsi`、`k3-rdomain.dtsi` 与三个 binding 文件，不对官网正文做猜测。

## 目录

- [1. 来源层分域与术语](#1-来源层分域与术语)
- [2. AP 侧中断拓扑（CLINT + IMSIC + APLIC）](#2-ap-侧中断拓扑clint--imsic--aplic)
- [3. RP 侧中断拓扑（k3-rdomain.dtsi 当前仅暴露设备节点）](#3-rp-侧中断拓扑k3-rdomaindtsi-当前仅暴露设备节点)
- [4. 时间与定时器（Timer.md 仍为 SPA 壳）](#4-时间与定时器timermd-仍为-spa-壳)
- [5. 未知项与边界](#5-未知项与边界)
- [6. 边界声明](#6-边界声明)
- [7. 修订快照](#7-修订快照)

## 1. 来源层分域与术语

> 本节术语约束来自 M03；不并列中英同义写法，不混用「AP / Application Processor」与「main CPU」以外的别名。

| 术语 | 主写法 | 边界 | 证据等级 | 来源 |
|---|---|---|---|---|
| AP | AP | 8 × X100 通用核与 8 × A100 AI 核所在域；本主题只描述其 AIA 子系统与 timer 拓扑 | 交叉验证 | `k3_ds.md` §1.2 + `k3.dtsi` 第 270 行起的 `&soc` 子树 |
| RP | RP | 2 × RT24 实时核域；本主题只描述其设备节点当前的 `interrupt-parent` 路由 | 交叉验证 | `k3-rdomain.dtsi` 第 6 行起的 `&soc { ... }` 子树 |
| AIA | AIA | RISC-V Advanced Interrupt Architecture（APLIC + IMSIC）；K3 SoC 声明支持 | 交叉验证（架构）/ 未知项（K3 实现细节） | [`k3_ds.md` §1.2](https://github.com/spacemit-com/docs-chip/blob/main/zh/key_stone/k3/k3_docs/k3_ds.md) + `k3.dtsi` 第 299–345 行 |
| CLINT | CLINT | Core Local Interruptor；M-mode timer + M-mode 软件中断；AP 侧 `clint0@e081c000` | 交叉验证 | `k3.dtsi` 第 276–297 行 + `sifive,clint.yaml` |
| IMSIC | IMSIC | Incoming MSI Controller；AIA 的 MSI 投递端；AP 侧 `simsic@e0400000` | 交叉验证 | `k3.dtsi` 第 299–333 行 + `riscv,imsics.yaml` |
| APLIC | APLIC | Advanced Platform-Level Interrupt Controller；AIA 的线中断聚合端；AP 侧 `saplic@e0804000` | 交叉验证 | `k3.dtsi` 第 335–345 行 + `riscv,aplic.yaml` |
| MSIP | MSIP | Machine Software Interrupt；CLINT 的核间软件中断通道（IPI） | 交叉验证 | `sifive,clint.yaml` `interrupts-extended` 字段 + `k3.dtsi` 第 278–295 行 `&cpuN_intc 3` |
| notification | notification | 跨核轻量信号；本主题中仅声明 mailbox 是其 K3 实际承载方式，详细协议与寄存器保留到 Iter 001 / `com260-mailbox-notification.md` | 推论 | D2 通知≠数据真值设计决策 + R10–R12 第三方分析 |

## 2. AP 侧中断拓扑（CLINT + IMSIC + APLIC）

> 本节事实均经 [`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) 直接证实（2026-09-08 观察），并由 [`riscv,aplic.yaml`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/Documentation/devicetree/bindings/interrupt-controller/riscv,aplic.yaml) / [`riscv,imsics.yaml`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/Documentation/devicetree/bindings/interrupt-controller/riscv,imsics.yaml) / [`sifive,clint.yaml`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/Documentation/devicetree/bindings/timer/sifive,clint.yaml) 三个 binding 字段佐证。本节不涉及 CoM260 板级使能。

| 节点 | 关键字段 | 事实 | 证据等级 | 来源 |
|---|---|---|---|---|
| `clint0@e081c000` | `compatible = "riscv,clint0"` | AP 侧 CLINT 在 `k3.dtsi` 第 276 行定义；reg 0xe081c000，大小 0x4000 | 交叉验证 | `k3.dtsi` 第 276–297 行 |
| `clint0@e081c000` | `interrupts-extended` 长度 | `interrupts-extended` 含 32 项（`&cpu0_intc 3` / `&cpu0_intc 7` … `&cpu15_intc 7`），覆盖 16 个 hart 的 M-mode software（`&cpuN_intc 3`）与 M-mode timer（`&cpuN_intc 7`）两类本地中断 | 交叉验证 | `k3.dtsi` 第 278–295 行 + `sifive,clint.yaml` 字段定义 |
| `simsic@e0400000` | `compatible = "riscv,imsics"` | AP 侧 IMSIC 在 `k3.dtsi` 第 299 行定义；reg 0xe0400000，大小 0x400000 | 交叉验证 | `k3.dtsi` 第 299–333 行 + `riscv,imsics.yaml` |
| `simsic@e0400000` | `riscv,num-ids = <511>` | IMSIC 暴露 511 个中断 identity（写为 511 而非 512） | 交叉验证 | `k3.dtsi` 第 316–317 行 + `riscv,imsics.yaml` |
| `simsic@e0400000` | `riscv,num-guest-ids = <63>` | guest file 数量为 63；K3 注释显式说明 X100 仅 7 个 vs-level guest file 有效、A100 全部无效 | 交叉验证 | `k3.dtsi` 第 318–330 行 |
| `simsic@e0400000` | `riscv,hart-index-bits = <4>` / `riscv,guest-index-bits = <6>` | 4 位 hart index = 16 hart；6 位 guest index = 64 中断文件 = 1 s-level + 63 vs-level | 交叉验证 | `k3.dtsi` 第 320–331 行 |
| `saplic@e0804000` | `compatible = "riscv,aplic"` | AP 侧 APLIC 在 `k3.dtsi` 第 335 行定义；reg 0xe0804000，大小 0x4000 | 交叉验证 | `k3.dtsi` 第 335–345 行 + `riscv,aplic.yaml` |
| `saplic@e0804000` | `riscv,num-sources = <512>` | APLIC 聚合 512 个线中断源 | 交叉验证 | `k3.dtsi` 第 342–343 行 + `riscv,aplic.yaml` |
| `saplic@e0804000` | `msi-parent = <&simsic>` | 该节点通过 IMSIC 以 MSI 模式投递 APLIC 汇聚的线中断 | 交叉验证 | `k3.dtsi` 第 339 行 + `riscv,aplic.yaml` |
| AP 设备消费者 | `interrupt-parent = <&saplic>` | `k3.dtsi` 中已直接打开的 AP 设备消费者（含 UART、I2C、GMAC、PINCTRL、QSPI 等）至少 50 处 `interrupt-parent` 全部指向 `&saplic`，未出现 `&cpuN_intc` 直连 | 交叉验证 | `k3.dtsi` 第 476、497、506、519…2059 等行；范围见 `interrupt-parent = <&saplic>` 命中行 |
| 16 hart 拓扑 | `&cpu0_intc` … `&cpu15_intc` | AP CLINT / IMSIC 节点 `interrupts-extended` 覆盖 `cpu0_intc` … `cpu15_intc`，共 16 个本地中断控制器入口 | 交叉验证 | `k3.dtsi` 第 278–309 行 |

> 拓扑结论: AP 域中断投递 = `外设 → APLIC（声明 512 个 source）→ IMSIC(MSI) → 各 hart INTC`；IPI（核间软件中断）= `写 CLINT MSIP → 目标 hart INTC`。CLINT 同时承担 M-mode timer（`mtimecmp`）与 M-mode software（`MSIP`）两个角色；APLIC/IMSIC 是 AIA 标准组合，`k3.dtsi` 中 APLIC `riscv,num-sources = <512>`、IMSIC `riscv,num-ids = <511>`、guest file 数量为 63 + 1 s-level = 64。

## 3. RP 侧中断拓扑（k3-rdomain.dtsi 当前仅暴露设备节点）

> 本节事实均经 [`k3-rdomain.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-rdomain.dtsi) 直接证实（2026-09-08 观察）。本节只描述 `k3-rdomain.dtsi` 当前实际暴露的字段；不存在的节点列入 §5 未知项。

| 维度 | 事实 | 证据等级 | 来源 |
|---|---|---|---|
| RP 域设备节点 | `k3-rdomain.dtsi` 全部节点为 `&soc { ... }` 的子节点，类型限定为设备消费者（serial / fdcan / pwm / ri2c / adma / ri2s / r-irc-rx / rspi），不包含任何 interrupt-controller / timer / clint 节点 | 交叉验证 | `k3-rdomain.dtsi` 全文（仅设备节点） |
| RP 设备 `interrupt-parent` | RP 域 6 个 RCPU UART（`r_uart0`…`r_uart5`）的 `interrupt-parent = <&saplic>`；5 个 RCPU flexcan（`r_flexcan0`…`r_flexcan4`）的 `interrupt-parent = <&saplic>`；RP 域 I2C、ADMA、SPI、IR-RX 等的 `interrupt-parent = <&saplic>` | 交叉验证 | `k3-rdomain.dtsi` 各节点 `interrupt-parent` 行 |
| RP UART IRQ 编号 | `r_uart0`..`r_uart5` 的 `interrupts = <251..256 IRQ_TYPE_LEVEL_HIGH>`，落在 AP APLIC `riscv,num-sources = <512>` 声明的 source 范围（具体有效 source ID 编号规则不属本主题） | 交叉验证 | `k3-rdomain.dtsi` r_uart0..r_uart5 节点 `interrupts` 行 |
| RP UART base 区间 | RP 域设备 `reg` 落在 `0xc0710000`..`0xc08de000` 区间（不与 AP 主空间 0xd4xxxxxx 重叠） | 交叉验证 | `k3-rdomain.dtsi` 各节点 `reg` 行 |
| RP 时钟 / reset 来源 | RP 设备 `clocks` 引用 `&syscon_rcpu_uartctrl` / `&syscon_rcpu_sysctrl` / `&syscon_rcpu_pwmctrl` / `&syscon_rcpu_i2cctrl` / `&syscon_rcpu_i2sctrl` / `&syscon_rcpu_spictrl`（在 `k3.dtsi` 中定义）；`resets` 引用同名节点；`clocks` 中 `gate` 一律来自 `&syscon_mpmu` | 交叉验证 | `k3-rdomain.dtsi` 各节点 `clocks` / `resets` 行 |
| RP 域 PLIC / SysTimer / MSIP / AON timer 节点 | 在 `k3-rdomain.dtsi` 2026-09-08 观察范围内**未出现** `riscv,plic` / `riscv,clint0` / `spacemit,k3-systimer` / `riscv,clint0-msip` 节点 | 未知项 | `k3-rdomain.dtsi` 全文 |
| RP 域 PLIC / SysTimer 第三方线索 | R10–R12 第三方分析中以 `systimer@e4000000`（`spacemit,k3-systimer` / `riscv,clint0`）+ `msip@e4000000`（`spacemit,k3-systimer-msip` / `riscv,clint0-msip`）等独立节点描述 RCPU 的 timer + IPI；与官方 `k3.dtsi` / `k3-rdomain.dtsi` 当前未直接观察的拓扑不一致 | 未知项 | R10–R12（Rt-Async-AMP at `ccb1ff0b`）+ tgoskits |

> 拓扑观察: `k3-rdomain.dtsi` 当前**只暴露 RP 域设备消费者**（serial / fdcan / pwm / ri2c / adma / ri2s / r-irc-rx / rspi），所有 RP 设备中断在 DTS 中 `interrupt-parent` 指向 `&saplic`，IRQ 251–256 + 241/243/245/247/249（flexcan）+ 257/258/260/261/263/264/266/267（adma）+ 269/270/271（rspi）+ 272/273（ri2c）+ 274/275（r-irc-rx）；RP 设备 `clocks` / `resets` 引用 `&syscon_rcpu_*` 与 `&syscon_mpmu` phandle（在 `k3.dtsi` 中定义）。RP 域独立的 PLIC / SysTimer / MSIP 节点在官方 DTS 当前不可见，列入 §5 未知项；syscon phandle 节点的域归属与 RP 是否存在独立 PLIC 实体，**不**由本主题判定。

## 4. 时间与定时器（Timer.md 仍为 SPA 壳）

> Timer.md 在覆盖表中的观察结果为 Vue SPA 壳，未取得正文。本节只描述从 `k3.dtsi` / `sifive,clint.yaml` / R10–R12 第三方分析观察到的事实；Timer.md 正文细节保留为 `未知项`。

| 维度 | 事实 | 证据等级 | 来源 |
|---|---|---|---|
| AP M-mode timer 提供者 | AP 侧 M-mode timer 由 `clint0@e081c000`（`riscv,clint0`）提供；`interrupts-extended` 中 16 个 `&cpuN_intc 7` 即 M-mode timer 入口 | 交叉验证 | `k3.dtsi` 第 278–295 行 + `sifive,clint.yaml` |
| AP M-mode timer 频率字段 | `k3.dtsi` 在 `&cpus` 节点未直接展开 `timebase-frequency` 字段；M-mode timer 频率继承自 `k3.dtsi` 上游配置，本节不猜测具体值 | 未知项 | `k3.dtsi` 第 270 行起的 `&soc`，需补查 `&cpus` |
| RP M-mode timer / SysTimer 节点 | RP 域独立 SysTimer / MSIP 节点在 `k3.dtsi` 与 `k3-rdomain.dtsi` 当前 2026-09-08 观察范围内**未直接暴露**；第三方线索 `systimer@e4000000`（`spacemit,k3-systimer`）未由官方 DTS 证实 | 未知项 | R10–R12 第三方分析，列为线索不作正文 |
| K3 官方 timer 驱动说明 | Timer.md 已登记为 `partially-observed`；当前仅取得 SPA 壳，未取得正文 | 未知项 | Timer.md 直连 |
| CLINT 兼容列表 | `sifive,clint.yaml` 列出 `spacemit,k1-clint`（位于 `sifive,clint0` 回落链）与 `canan,k210-clint` / `eswin,eic7700-clint` / `sifive,fu540-c000-clint` / `starfive,jh*` / `thead,c900-clint` 等同类回落；`sifive,clint0` 单独使用时还承担 `riscv,clint0` 兜底（`deprecated`，仅 QEMU virt） | 交叉验证 | `sifive,clint.yaml` compatible 段 |
| CLINT 必填字段 | `sifive,clint.yaml` `required: [compatible, reg, interrupts-extended]`；K3 的 `clint0@e081c000` 满足三项 | 交叉验证 | `sifive,clint.yaml` required 段 + `k3.dtsi` 第 276–297 行 |

> 时间结论: AP 域时间源 = `CLINT（riscv,clint0）`（`k3.dtsi` 中 `clint0@e081c000` 的 `interrupts-extended` 覆盖 16 个 `&cpuN_intc 7` 入口）；IMSIC 属中断投递（MSI 路径），**不**承担时间源角色。`sifive,clint.yaml` 把 `spacemit,k1-clint` 收录进 `sifive,clint0` 回落链是 binding 层的兼容声明；K3 节点 `clint0@e081c000` 仅声明 `compatible = "riscv,clint0"`，未声明 `spacemit,k1-clint`，不据此推定 K3 AP CLINT 与 SpacemiT K1 IP 同源。RP 域独立 timer / MSIP 节点在官方 DTS 不可见，K3 真实拓扑与第三方 R10–R12 描述之间存在未解的来源层缺口。

## 5. 未知项与边界

### 5.1 未知项汇总（每项含 4 字段）

#### U1. Timer.md 官方正文

- 当前证据: Timer.md（`/software/SDK/buildroot/k3_buildroot/device/peripheral_driver/Timer.md`）在覆盖表中的观察日期为 2026-09-02，访问状态为 `partially-observed`；当前只有 Vue SPA 壳证据，没有正文。
- 禁止推断: 不得用 `k3.dtsi` 中 `riscv,clint0` 节点或 `sifive,clint.yaml` 字段替代 Timer.md 正文细节；不得假定 Timer.md 包含 K3 专属 timer IP 描述；不得由 `sifive,clint.yaml` 的回落链反推 K3 硬件实现等同于 K1。
- 解除条件: 取得 Timer.md 官方正文（含 K3 M-mode timer 频率、`mtime` 寄存器布局、HPB / APB 映射、driver 加载路径）；或 docs-buildroot 对应 GitHub 页面提供等效正文（已确认无 GitHub 等价页）。
- 影响主题: 本主题（K3 timer 主题全部正文依赖）；间接影响 Iter 001 / `com260-mailbox-notification.md`（mailbox 时序锚点）；MS08 / 未来 K3 驱动开发。

#### U2. RP 域 PLIC 节点

- 当前证据: `k3-rdomain.dtsi` 2026-09-08 观察范围内仅含设备消费者节点；不存在 `riscv,plic` / `spacemit,k3-plic` / 类似 `interrupt-controller@...` 节点；R10–R12 第三方分析描述 RP 域存在独立 PLIC 节点，但描述与官方 DTS 现状不一致。
- 禁止推断: 不得用 R10–R12 第三方分析直接补出 RP PLIC 节点；不得假定 RP 设备中断经独立 PLIC 聚合（当前 DTS 显示全部走 `&saplic`）；不得由「RP 设备 `interrupt-parent = <&saplic>`」反推 RP 域无任何 PLIC 实体（K3 可能存在 PLIC 但 `k3-rdomain.dtsi` 未实例化）。
- 解除条件: 取得含 RP PLIC 节点的 `k3-rdomain.dtsi`（或同级）更新版本；或 K3 programmer manual 公开 RP 域中断控制器拓扑；或 K3 板级 DTS 实例化 RP PLIC 节点。
- 影响主题: 本主题 §3（RP 拓扑结论）；后续 RP 域驱动 / 固件（mailbox、RCPU 中断处理）。

#### U3. RP 域 SysTimer / MSIP 节点

- 当前证据: `k3.dtsi` 与 `k3-rdomain.dtsi` 2026-09-08 观察范围内均无 `spacemit,k3-systimer` / `spacemit,k3-systimer-msip` / `riscv,clint0-msip` 节点；R10–R12 第三方分析中以 `systimer@e4000000` + `msip@e4000000` 双节点描述 RCPU timer + IPI。
- 禁止推断: 不得用 R10–R12 描述补出 RP SysTimer 节点；不得由「AP CLINT base 0xe081c000」反推 RP CLINT base；不得由 R10–R12 中 `0xe4000000` 推定该地址是 K3 官方公开拓扑。
- 解除条件: 取得含 RP SysTimer / MSIP 节点的 `k3.dtsi` / `k3-rdomain.dtsi` 更新版本；或 K3 programmer manual 公开 RP CLINT 基地址与拓扑。
- 影响主题: 本主题 §4（RP timer）；Iter 001 / `com260-mailbox-notification.md`（mailbox 时序与 IPI）；MS08 / 未来 K3 驱动开发。

#### U4. APLIC / IMSIC 寄存器布局与 hart delivery 协议

- 当前证据: 三个 binding 字段已确认（`riscv,num-ids = <511>` / `riscv,num-sources = <512>` / `riscv,hart-index-bits = <4>` / `riscv,guest-index-bits = <6>` / `msi-parent = <&simsic>`）；具体寄存器偏移、mask/ack/complete 协议、hart file 选择规则未公开。
- 禁止推断: 不得用 RISC-V AIA 标准行为直接等同 K3 APLIC/IMSIC 寄存器实现；不得假定 K3 APLIC 寄存器布局与 SiFive 一致；不得由 `riscv,num-sources = <512>` 推断 K3 一定使用全部 512 source。
- 解除条件: 取得 K3 SoC 公开寄存器手册（programmer reference 或 datasheet 寄存器章节）；或在 linux-6.18 仓库定位 K3 APLIC/IMSIC 驱动初始化源码与寄存器偏移表。
- 影响主题: 本主题 §2（AP 拓扑完整化）；Iter 001 / `com260-mailbox-notification.md`（mailbox 通知与 MSI 投递）；G4（与现有 known-gaps.md G4 直接重合，本主题完成后 G4 状态将由 `open` 调整为 `partial`，调整在 Iter 001 / T4 执行）。

#### U5. 跨核通知承载方式与协议

- 当前证据: D2 决策确认 K3 跨核通知由 mailbox 承载，通知≠数据真值；具体 mailbox 节点、IPI 路由、寄存器偏移未在本主题展开（属于 Iter 001 / `com260-mailbox-notification.md` 范围）；R10–R12 / tgoskits 第三方分析给出 ov-shm / 平台 mailbox 抽象。
- 禁止推断: 不得用第三方 mailbox 抽象替代 K3 官方实现；不得假定 K3 mailbox 节点基地址等同于 R10–R12 假设值；不得把 mailbox 通知与 MSI 中断混为同一回事。
- 解除条件: Iter 001 / `com260-mailbox-notification.md` 完成（T3 + T4 + T5）；或 docs-buildroot 文档公开 K3 mailbox 节点；或 K3 programmer manual 公开 mailbox 寄存器。
- 影响主题: Iter 001 / `com260-mailbox-notification.md`（本主题不展开）；MS08 / 未来 K3 驱动开发。

### 5.2 与现有 known-gaps.md 的对应

- G4「AIA / APLIC / IMSIC 地址、IRQ domain 与 hart delivery」与本主题 U1–U4 直接相关。本主题完成后 G4 状态由 `open` 调整为 `partial`，由 Iter 001 / T4 在 `docs/reference/known-gaps.md` 执行（不在本 Cycle 范围）。
- G1 / G2 / G3 / G5 / G6 / G7 / G8 / G9 / G10 与本主题不直接相关，不在本 Cycle 处理。
- Timer.md 仍 `partially-observed`，源端修订保持 `unknown`，不提升身份；G4 的部分解除只到 `partial`，U1 仍待解除。

## 6. 边界声明

- 本主题**不**展开: 寄存器偏移、mask/ack/complete 协议位、IPI 完整软件路径、mailbox 协议细节、AP/RP 时钟树与 reset 链（属 [`k3-platform-control.md`](../platform/k3-platform-control.md) 与 Iter 001 mailbox 主题）。
- 本主题**不**证明: CoM260 Kit 是否实际启用 §2 / §3 节点、`k3_com260*.dts` 板级覆盖对 RP 域是否引入额外节点、Timer.md 正文细节、RP PLIC 实体存在与否。
- 本主题**遵守**:
  - M01（单一权威源 `Timer.md`，其余为 cross-validation）；
  - M02（首行 / ≤500 行 / 相对链接）；
  - M03（zh-CN 与英文术语主写法，不混用同义并列）；
  - M04（无可执行代码 / 无构建命令）；
  - D01（纯 Markdown 聚合）；
  - D02（主题驱动目录，不镜像官网 URL 树）；
  - D05（首行同时表达源端修订与观察日期，不合并）；
  - D06（四级证据，每条事实逐条标级，不整段统一）；
  - D07（刷新状态与聚合状态分离，Timer.md 仍 `partially-observed`）；
  - D08（配置修复只增必要引用，不批量刷新）。

## 7. 修订快照

| 日期 | 变更 | 来源 |
|---|---|---|
| 2026-09-08 | 初版：建立 K3 SoC 中断/时间/通知主题；分 AP / RP 两域；登记 1 个官方 + 5 个 cross-validation 源；建立 5 项未知项（U1–U5） | MS05 / Iter 000 / T2 |
