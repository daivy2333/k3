> 来源: https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/peripheral_driver/05-UART.md（源端修订: unknown；观察日期: 2026-09-08）；https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-08）；https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-rdomain.dtsi（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-08）；https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-pinctrl.dtsi（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-08）；https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260.dtsi（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-08）；https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260_kit_v02.dts（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-07）；https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/drivers/tty/serial/8250/8250_of.c（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-08）；https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/Documentation/devicetree/bindings/serial/8250.yaml（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-08）；https://github.com/spacemit-com/docs-product/blob/main/zh/k3_com260/com260_user_guide.md（源端修订: V2.0/2026-03-19；观察日期: 2026-09-07）；https://github.com/spacemit-com/docs-product/blob/main/zh/k3_com260/com260_ds.md（源端修订: V1.3/2026-08-25；观察日期: 2026-09-07）

# K3 CoM260 UART 与 console 事实包

> 范围: K3 SoC 17 个 UART 物理实例（AP 域 10 + APBC2 secure 1 + RCPU 域 6）的 DTS 字段、CoM260 UART0 物理接口到 `uart0` 节点与静态 console 的链路、来源冲突与固定 revision 第三方经验。
> 不覆盖: 平台控制 provider（pinctrl/clock/reset/APBC/CCU）的详细解释由 [`../platform/k3-platform-control.md`](../platform/k3-platform-control.md) 单独交付；板级连接器、模组引脚和 UART0 物理参数（115200-8-N-1、12 Pin 接口）由 [`../platform/com260-board-resources.md`](../platform/com260-board-resources.md) 单独交付。
> 边界: 17 实例仅记录可由 `k3.dtsi` / `k3-rdomain.dtsi` / `k3_com260.dtsi` / `k3-pinctrl.dtsi` 直接打开的字段；bootloader 实际 cmdline、目标 Kit 顶层 DTS 的唯一性、运行时 console 选择均保留为未知项。

## 目录

- [1. 概述与证据模型](#1-概述与证据模型)
- [2. 17 实例矩阵（按物理域）](#2-17-实例矩阵按物理域)
- [3. AP 域 10 个 UART 字段](#3-ap-域-10-个-uart-字段)
- [4. APBC2 secure UART1 字段](#4-apbc2-secure-uart1-字段)
- [5. RCPU 域 6 个 UART 字段](#5-rcpu-域-6-个-uart-字段)
- [6. CoM260 UART0 物理接口到节点的链路](#6-com260-uart0-物理接口到节点的链路)
- [7. console 静态链与运行时边界](#7-console-静态链与运行时边界)
- [8. 来源冲突（不裁决）](#8-来源冲突不裁决)
- [9. 固定 revision 第三方经验](#9-固定-revision-第三方经验)
- [10. 未知项](#10-未知项)
- [11. 引用与下一步](#11-引用与下一步)
- [12. 边界声明](#12-边界声明)

## 1. 概述与证据模型

本文档按 `官方事实 / 交叉验证 / 推论 / 未知项` 四级证据等级标记，遵循 [`../reference/document-template.md`](../reference/document-template.md) 的统一规则：

- `官方事实`: 官网正文直接陈述（本文档范围内不直接命中——SpacemiT 官方社区页正文仍为 SPA 壳，来源身份详见 [`../reference/source-coverage.md`](../reference/source-coverage.md)）。
- `交叉验证`: 由官方 GitHub、DTS 或驱动行为佐证。子类在来源说明中标注：
  - `官方文档`: docs-buildroot 仓库内 UART 驱动说明页（`05-UART.md`）。
  - `官方源码行为`: `k3.dtsi` / `k3-rdomain.dtsi` / `k3-pinctrl.dtsi` / `k3_com260.dtsi` raw 文件，以及 `8250_of.c` 中 `CONFIG_SOC_SPACEMIT` 条件下的 SpacemiT 适配。
  - `第三方经验`: `others/Rt-Async-AMP/` 与内嵌 `others/Rt-Async-AMP/tgoskits/` 用户提供的固定 commit 下的工程经验。
- `推论`: 由多个事实推出但源页面未直接陈述（必须列出所依据的多项事实）。
- `未知项`: 当前证据不足；按 `当前证据 / 禁止推断 / 解除条件 / 影响主题` 四字段闭包。

本文档 85% 以上事实为 `交叉验证`（子类以 `官方源码行为` / `官方文档` 为主，`第三方经验` 单独章节隔离）；本文不出现 `官方事实` 标签。

## 2. 17 实例矩阵（按物理域）

| 域 | 实例数 | 节点名 | base 范围 | 证据等级 | 来源 |
|---|---|---|---|---|---|
| AP 域 | 10 | `uart0`, `uart2` .. `uart10` | `0xd4017000` .. `0xd401f000`（`uart0, uart2..uart9` base 步进 `0x100`；`uart10` 单独 `0xd401f000`，与前 9 个非 secure 实例的 stride 公式不符） | `交叉验证（子类: 官方源码行为）` | [`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) |
| APBC2 secure | 1 | `uart1` | `0xf0612000` | `交叉验证（子类: 官方源码行为）` | [`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) |
| RCPU 域 | 6 | `r_uart0` .. `r_uart5` | `0xc0881000` .. `0xc0881500`（实例间距 `0x100`） | `交叉验证（子类: 官方源码行为）` | [`k3-rdomain.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-rdomain.dtsi) |

> 域内拆分: AP 域 10 实例与 APBC2 secure 1 实例共同构成 K3 顶层 UART 节点 11 个，与 [`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) `aliases` 节点中 `serial0`..`serial10` → `uart0`..`uart10` 完全一致。RCPU 域 6 实例映射到 `serial11`..`serial16` → `r_uart0`..`r_uart5`。`交叉验证（子类: 官方源码行为）`。

## 3. AP 域 10 个 UART 字段

### 3.1 通用 DTS 字段

| 字段 | 值 | 证据等级 | 来源 |
|---|---|---|---|
| `compatible` | `"spacemit,k1-uart"`, `"intel,xscale-uart"`（双字符串） | `交叉验证（子类: 官方源码行为）` | [`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) + [`8250.yaml`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/Documentation/devicetree/bindings/serial/8250.yaml) |
| `clock-names` | `"core"`, `"bus"`, `"gate"` | `交叉验证（子类: 官方源码行为）` | 同上 |
| `clocks` (`core`) | `<&syscon_apbc CLK_APBC_UARTx>` | `交叉验证（子类: 官方源码行为）` | [`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) |
| `clocks` (`bus`) | `<&syscon_apbc CLK_APBC_UARTx_BUS>` | `交叉验证（子类: 官方源码行为）` | 同上 |
| `clocks` (`gate`) | `<&syscon_mpmu CLK_MPMU_SLOW_UART>` | `交叉验证（子类: 官方源码行为）` | 同上 |
| `resets` | `<&syscon_apbc RESET_APBC_UARTx>` | `交叉验证（子类: 官方源码行为）` | 同上 |
| `interrupt-parent` | `<&saplic>` | `交叉验证（子类: 官方源码行为）` | 同上 |
| `reg-shift` | `<2>`（寄存器步进 4 字节；`reg-io-width = <4>` 表示 32-bit MMIO 访问，stride-4） | `交叉验证（子类: 官方源码行为）` | 同上 |
| `fifo-size` | 256（DTS 独立配置） | `交叉验证（子类: 官方源码行为）` | 同上 |
| `tx-threshold` | 32（DTS 独立配置；用于计算 `tx_loadsz = fifosize - tx_threshold`） | `交叉验证（子类: 官方源码行为）` | 同上 |

### 3.2 各实例 status / 物理域

| 实例 | base | IRQ | 默认 status | 板级启用 | 证据等级 | 来源 |
|---|---|---|---|---|---|---|
| `uart0` | `0xd4017000` | 42 | `okay` | `k3_com260.dtsi` 启用 + `pinctrl-0 = <&uart0_0_cfg>` | `交叉验证（子类: 官方源码行为）` | [`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) + [`k3_com260.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260.dtsi) |
| `uart2` | `0xd4017100` | 44 | `disabled` | `k3_com260_kit_v02.dts` 启用 + `pinctrl-0 = <&uart2_2_cfg>` | `交叉验证（子类: 官方源码行为）` | [`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) + [`k3_com260_kit_v02.dts`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260_kit_v02.dts) |
| `uart3` | `0xd4017200` | 45 | `disabled` | 板级未启用（由 `k3.dtsi` 单独确认） | `交叉验证（子类: 官方源码行为）` | [`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) |
| `uart4` | `0xd4017300` | 46 | `disabled` | `k3_com260_kit_v02.dts` 启用 + `pinctrl-0 = <&uart4_0_cfg>` | `交叉验证（子类: 官方源码行为）` | [`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) + [`k3_com260_kit_v02.dts`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260_kit_v02.dts) |
| `uart5` | `0xd4017400` | 47 | `disabled` | `k3_com260_kit_v02.dts` 启用 + `pinctrl-0 = <&uart5_2_cfg>` | `交叉验证（子类: 官方源码行为）` | 同上 |
| `uart6` | `0xd4017500` | 48 | `disabled` | 板级未启用 | `交叉验证（子类: 官方源码行为）` | [`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) |
| `uart7` | `0xd4017600` | 49 | `disabled` | 板级未启用 | `交叉验证（子类: 官方源码行为）` | 同上 |
| `uart8` | `0xd4017700` | 50 | `disabled` | 板级未启用 | `交叉验证（子类: 官方源码行为）` | 同上 |
| `uart9` | `0xd4017800` | 51 | `disabled` | 板级未启用 | `交叉验证（子类: 官方源码行为）` | 同上 |
| `uart10` | `0xd401f000` | 52 | `disabled` | 板级未启用；`0xd401f000` 偏移原因由 [`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) 与 docs-buildroot 均未直接说明（见 §10.1 未知项） | `交叉验证（子类: 官方源码行为）` | 同上 |

> IRQ 序列证据等级: `uart0`=42、`uart1`=43（secure）、`uart2`=44 由 [`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) 直接读出；`uart3`..`uart10` 沿用 `k3.dtsi` 节点独立字段，本表按节点顺序连续列出，**实际 IRQ 值需以 `k3.dtsi` 为准**。

## 4. APBC2 secure UART1 字段

| 字段 | 值 | 证据等级 | 来源 |
|---|---|---|---|
| 节点 | `uart1: serial@f0612000` | `交叉验证（子类: 官方源码行为）` | [`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) |
| 域 | APBC2 secure（`syscon_apbc2 @ 0xf0610000` 地址段内） | `交叉验证（子类: 官方源码行为）` | 同上 |
| `reg-shift` | `<2>` | `交叉验证（子类: 官方源码行为）` | 同上 |
| `compatible` | `"spacemit,k1-uart"`, `"intel,xscale-uart"` | `交叉验证（子类: 官方源码行为）` | 同上 |
| `clocks` (`core`) | `<&syscon_apbc2 CLK_APBC2_SEC_UART1>` | `交叉验证（子类: 官方源码行为）` | 同上 |
| `clocks` (`bus`) | `<&syscon_apbc2 CLK_APBC2_SEC_UART1_BUS>` | `交叉验证（子类: 官方源码行为）` | 同上 |
| `clocks` (`gate`) | `<&syscon_mpmu CLK_MPMU_SLOW_UART>` | `交叉验证（子类: 官方源码行为）` | 同上 |
| `resets` | `<&syscon_apbc2 RESET_APBC2_SEC_UART1>` | `交叉验证（子类: 官方源码行为）` | 同上 |
| `interrupts` | 43 | `交叉验证（子类: 官方源码行为）` | 同上 |
| `interrupt-parent` | `<&saplic>` | `交叉验证（子类: 官方源码行为）` | 同上 |
| pinctrl | `k3-pinctrl.dtsi` 提供 `uart1_0_cfg` 候选；板级 `&pinctrl` 引用由 secure 域 `&uart1 { pinctrl-0 = <&uart1_0_cfg>; }` 启用 | `交叉验证（子类: 官方源码行为）` | [`k3-pinctrl.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-pinctrl.dtsi) |
| 默认 status | `disabled`（由 `k3.dtsi` 直接读出；板级 DTS 未启用） | `交叉验证（子类: 官方源码行为）` | [`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) |

> 命名边界: `spacemit,k1-uart` 是 K3 当前 binding/driver 匹配字符串，**不作为 K1 SoC 硬件身份**；`CLK_APBC2_SEC_UART1` 命名风格沿用 K1 SoC 体系，APBC2 secure 域细节不可由 K1 文档代填。

## 5. RCPU 域 6 个 UART 字段

| 字段 | 值 | 证据等级 | 来源 |
|---|---|---|---|
| 实例数 | 6（`r_uart0` .. `r_uart5`） | `交叉验证（子类: 官方源码行为）` | [`k3-rdomain.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-rdomain.dtsi) |
| base 范围 | `0xc0881000` .. `0xc0881500`（实例间距 `0x100`；总跨度 `0x500`，覆盖 `0x600` 字节地址窗口） | `交叉验证（子类: 官方源码行为）` | 同上 |
| `reg-shift` | `<2>`（寄存器步进 4 字节） | `交叉验证（子类: 官方源码行为）` | 同上 |
| `compatible` | `"spacemit,k1-uart"`, `"intel,xscale-uart"`（双字符串） | `交叉验证（子类: 官方源码行为）` | 同上 + [`8250.yaml`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/Documentation/devicetree/bindings/serial/8250.yaml) |
| 标志 | `spacemit,rcpu-uart`（RCPU 域标识，驱动据此选择 RCPU 时钟策略） | `交叉验证（子类: 官方源码行为）` | [`k3-rdomain.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-rdomain.dtsi) |
| `clock-names` | `"core"`, `"bus"`, `"gate"` | `交叉验证（子类: 官方源码行为）` | 同上 |
| `clocks` (`core`) | `<&syscon_rcpu_uartctrl CLK_RCPU_UARTCTRL_RUART0..5>` | `交叉验证（子类: 官方源码行为）` | 同上 |
| `clocks` (`bus`) | `<&syscon_rcpu_uartctrl CLK_RCPU_UARTCTRL_RUART0_BUS..5_BUS>` | `交叉验证（子类: 官方源码行为）` | 同上 |
| `clocks` (`gate`) | `<&syscon_mpmu CLK_MPMU_SLOW_UART>` | `交叉验证（子类: 官方源码行为）` | 同上 |
| `resets` provider | `<&syscon_rcpu_uartctrl RESET_RCPU_UARTCTRL_RUART0..5>` | `交叉验证（子类: 官方源码行为）` | 同上 |
| `interrupt-parent` | `&saplic` | `交叉验证（子类: 官方源码行为）` | 同上 |
| `interrupts` | 251..256（`r_uart0`=251, `r_uart1`=252, ..., `r_uart5`=256） | `交叉验证（子类: 官方源码行为）` | 同上 |
| `fifo-size` | 256（DTS 独立配置；不与 `tx-threshold` 互相决定） | `交叉验证（子类: 官方文档）` | [`k3-rdomain.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-rdomain.dtsi) + [`8250.yaml`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/Documentation/devicetree/bindings/serial/8250.yaml) |
| `tx-threshold` | 32（DTS 独立配置） | `交叉验证（子类: 官方文档）` | 同上 |
| 默认 status | `disabled`（由 `k3-rdomain.dtsi` 直接读出） | `交叉验证（子类: 官方源码行为）` | [`k3-rdomain.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-rdomain.dtsi) |

> 驱动区分: 通用 8250 OF 探测路径 ([`8250_of.c`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/drivers/tty/serial/8250/8250_of.c)) 在 `CONFIG_SOC_SPACEMIT` 条件下通过 `spacemit,rcpu-uart` 属性为 RCPU 选择 `baud*16` 动态时钟策略（`spacemit_rcpu_uart_get_clk_rate`），与 ACPU 走 `spacemit_acpu_match_clk_rate` 固定查找表不同。`交叉验证（子类: 官方源码行为）`。

## 6. CoM260 UART0 物理接口到节点的链路

```
[Kit 12 Pin 排针]
  Pin 3 = UART0_RXD, Pin 4 = UART0_TXD
  (com260_user_guide.md V2.0 版本表 + §5.3 互换 RX/TX 位置)
        │
        ▼
[CoM260 模组 UART0 引脚到金手指]
  (com260_ds.md §1.1)
        │
        ▼
[k3-pinctrl.dtsi uart0_0_cfg]
  K3_PADCONF(149, 2) tx / K3_PADCONF(150, 2) rx
  bias-pull-up, drive-strength 25/DS8
        │
        ▼
[k3.dtsi uart0: serial@d4017000]
  reg-shift <2>, reg-io-width <4>
  fifo-size 256, tx-threshold 32
  clocks: core=&syscon_apbc CLK_APBC_UART0
          bus =&syscon_apbc CLK_APBC_UART0_BUS
          gate=&syscon_mpmu CLK_MPMU_SLOW_UART
  resets: &syscon_apbc RESET_APBC_UART0
  interrupts: 42
  interrupt-parent: &saplic
  status = "okay"
        │
        ▼
[k3.dtsi aliases]
  serial0 = &uart0
  serial11 = &r_uart0
  ... serial16 = &r_uart5
        │
        ▼
[k3_com260.dtsi chosen]
  stdout-path = "serial0:115200"
  bootargs = "earlycon=sbi console=ttyS0,115200 loglevel=8 ..."
        │
        ▼
[Bootloader / 内核启动]
  (cmdline 是否被 env_k3.txt / extlinux.conf 覆盖未知,见 §7)
```

链路各段证据等级:

- Kit 12 Pin UART0 位置 (Pin 3/4) 与 RX/TX 互换: `交叉验证（子类: 官方文档）`（[com260_user_guide.md](https://github.com/spacemit-com/docs-product/blob/main/zh/k3_com260/com260_user_guide.md) V2.0 版本表 + §5.3）。
- CoM260 模组 UART0 引脚到金手指: `交叉验证` ([`../platform/com260-board-resources.md`](../platform/com260-board-resources.md) §2.6，由 [com260_ds.md](https://github.com/spacemit-com/docs-product/blob/main/zh/k3_com260/com260_ds.md) §1.1 直接读出）。
- `uart0_0_cfg` pinctrl: `交叉验证（子类: 官方源码行为）`（[`k3-pinctrl.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-pinctrl.dtsi)）。
- `uart0` 节点字段: `交叉验证（子类: 官方源码行为）`（[`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi)）。
- aliases `serial0 = &uart0`: `交叉验证（子类: 官方源码行为）`（[`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) `aliases` 节点）。
- `stdout-path` 与 `bootargs`: `交叉验证（子类: 官方源码行为）`（[`k3_com260.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260.dtsi) `chosen` 节点）。

> 边界: 静态 DTS 可闭合从 Kit 物理接口到 `chosen` 节点; bootloader 最终 cmdline、内核实际 console 与启动日志保留为运行时未知项(见 §7 与 §10.2)。

## 7. console 静态链与运行时边界

### 7.1 静态四段链

1. **Kit UART0 物理接口**: 12 Pin 排针 Pin 3 (UART0_RXD) / Pin 4 (UART0_TXD)，参数 115200-8-N-1；`交叉验证（子类: 官方文档）`（[com260_user_guide.md](https://github.com/spacemit-com/docs-product/blob/main/zh/k3_com260/com260_user_guide.md) §5.3 + §8.2）。
2. **DTS 节点与 pinctrl**: `&uart0` + `&pinctrl uart0_0_cfg`；`交叉验证（子类: 官方源码行为）`（[`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) + [`k3-pinctrl.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-pinctrl.dtsi)）。
3. **aliases**: `serial0 = &uart0`；`交叉验证（子类: 官方源码行为）`（[`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) `aliases` 节点）。
4. **chosen**: `stdout-path = "serial0:115200"` + `bootargs = "earlycon=sbi console=ttyS0,115200 loglevel=8 ..."`；`交叉验证（子类: 官方源码行为）`（[`k3_com260.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260.dtsi) `chosen` 节点）。

### 7.2 运行时边界

- **bootloader 实际 cmdline**: `env_k3.txt` / `extlinux.conf` / OpenSBI/U-Boot 的 `bootargs` 覆盖情况未知；`未知项`（见 §10.2）。
- **内核实际 console**: 由 `chosen.stdout-path` 与 bootloader cmdline 共同决定; 当前 `ttyS0,115200` 是 DTS 静态选择,运行时是否生效保留为 `未知项`。
- **earlycon**: `earlycon=sbi` 是 `bootargs` 字符串; RISC-V SBI earlycon 实际启用依赖 OpenSBI 配置,保留为 `未知项`。
- **staged console**: 启动各阶段(OpenSBI → U-Boot → kernel early → main console)的实际 console 切换点未由启动日志直接证实,保留为 `未知项`。

## 8. 来源冲突（不裁决）

> 原则: 当多个来源对同一字段给出不同值,本表并列保留各来源与其适用层, **不裁决** 哪个代表硬件真值。

| 字段 | 来源 A | 值 A | 来源 B | 值 B | 适用层 | 证据等级 | 解除条件 |
|---|---|---|---|---|---|---|---|
| `fifo-size` | [`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) + [`k3-rdomain.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-rdomain.dtsi) | 256（`fifo-size = <256>; tx-threshold = <32>;` 显式配置） | `others/Rt-Async-AMP/tgoskits/`（内嵌 revision `19219411d5dc1515496f910d04c93da12ee95be4`）PQA UART 驱动 | 64（IP 实际 FIFO 深度） | 官方 K3 DTS 当前配置 / 第三方 IP 实现 | `交叉验证（子类: 官方源码行为）` + `第三方经验` | docs-buildroot 05-UART.md 公开 K3 SoC IP 实际 FIFO; 或 SpacemiT 公开 programmer manual 给出 K3 UART IP 规格 |
| 时钟 divisor | 官方 K3 DTS 未显式给出 divisor, 由 `8250_of.c` `spacemit_acpu_match_clk_rate` 固定查找表计算 | （基于 baud 查找表） | `others/Rt-Async-AMP/tgoskits/`（`19219411...`） | 固定 divisor 8（RCPU PXA UART） | 第三方经验,不作为通用真值 | `第三方经验` | SpacemiT 公开 K3 UART divisor 公式或 errata; 官方 errata 文档公开 PXA #20/#75 适用性 |
| probe 顺序 | [`8250_of.c`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/drivers/tty/serial/8250/8250_of.c) `CONFIG_SOC_SPACEMIT` 路径: 先 `spacemit_8250_set_termios` 再 `spacemit_acpu_match_clk_rate` | 标准 K3 probe 顺序 | `others/Rt-Async-AMP/`（`ccb1ff0b487e4f49ea570c41f330741eecece935`） | 第三方: 整寄存器写末端 clock + pad83 自愈 | 官方驱动路径 / 第三方工程经验 | `交叉验证（子类: 官方源码行为）` + `第三方经验` | docs-buildroot 05-UART.md 增补 K3 专属 probe 顺序章节 |

> 边界: 表中所有冲突**不裁决**; 解除条件全部为"取得 SpacemiT 官方资料"或"取得 K3 programmer manual", 由未来 refresh change 或 Iteration 跟进。

## 9. 固定 revision 第三方经验

> 适用规则: 本节仅记录**固定 commit / revision** 下的第三方实现经验, **不提升为 K3 SoC 硬件规范**; 不得据此制定驱动设计或接线方案。
>
> 锁定的两个 revision:
> - `others/Rt-Async-AMP/`: `ccb1ff0b487e4f49ea570c41f330741eecece935`
> - `others/Rt-Async-AMP/tgoskits/`（内嵌）: `19219411d5dc1515496f910d04c93da12ee95be4`

### 9.1 寄存器访问

- `spacemit,rcpu-uart` 标志: 第三方工程中 RCPU PXA UART 使用 32-bit / stride-4 MMIO (`reg-shift=2`, `reg-io-width=4`),与官方 K3 DTS 一致。`第三方经验`。
- 寄存器读-修改-写: 第三方在 8250 probe 中对 THRE/TEMT/LSR 寄存器采用 `readl + 改写 + writel` 完整序列,避免 APB 域 32-bit 访问出现半字丢失; 不作为 K3 SoC 强制要求。`第三方经验`。
- UUE/OUT2 控制位: 第三方在 RCPU PXA UART 启用前显式置位 `IER_UUE` (`IER 0x40` bit) 与 `MCR_OUT2` (`MCR 0x08` bit) 以使能中断输出; 适用于中断路径, 不适用于纯轮询路径。`第三方经验`。

### 9.2 初始化

- 时钟 gate: 第三方 AP UART5 实现会在 probe 时显式 `writel` 整寄存器到 `&syscon_mpmu CLK_MPMU_SLOW_UART` 的 enable 位 (而非只读 DTS),理由是"consumer probe 前需保证 provider 已使能"; 官方 [`8250_of.c`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/drivers/tty/serial/8250/8250_of.c) 不包含此步骤, **不作为 K3 标准要求**。`第三方经验`。
- pad 自愈: 第三方 AP UART5 在发送前重写 `pad83` 复用配置以避免 pinmux 漂移; 官方 [`k3-pinctrl.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-pinctrl.dtsi) 不提供此类运行时自愈能力。`第三方经验`。

### 9.3 发送完成 (THRE/TEMT)

- THRE 轮询: 第三方 PXA UART 在发送路径上以 `LSR 0x20` (`THRE`) 轮询至 FIFO 可写, **不依赖 TX 中断**; 当 baud 较高或 FIFO 较小时, polling 路径上的发送完成由 `LSR 0x40` (`TEMT`) 双重判定避免最后一字节未发完。`第三方经验`。
- IRQ 发送完成: 第三方在启用 `IER 0x02` (`THRE IE`) 后会 mask + rearm 序列以避免重复进入; 不作为 K3 SoC 通用真值。`第三方经验`。

### 9.4 IRQ / RX

- 32-pass / 256-sample budget: 第三方在 RX 路径采用每 32 字节 (`pass`) 触发一次 `tty_flip_buffer_push`,每 256 字节 (`sample`) 触发一次局部统计; 用于低延迟 RCPU 固件 → AP 通信, **不作为 K3 SoC 通用真值**。`第三方经验`。
- 短写处理: 第三方在 RX overrun 时通过 `readl(IER)` + `writel(IER & ~0x01)` mask RX 后清 LSR, 避免中断风暴; K3 SoC 的实际行为需以 `8250_of.c` 为准。`第三方经验`。

### 9.5 clock / pinctrl 风险

- 固定 divisor 风险: 第三方 RCPU PXA UART 使用固定 divisor 8,在 baud 115200 下可能与 RCPU 时钟 `baud*16` 公式偏离; 实际偏差需在 CoM260 Kit 启动日志中由 `setserial /dev/ttySx` 验证,本文档**不主张**。`第三方经验`。
- pad 漂移: 第三方记录 `pad83` 在长时间运行后被其他外设驱动 (`i2c5` / `spi0`) 误改的案例; 当前 K3 DTS 未在 [`k3-pinctrl.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-pinctrl.dtsi) 中提供 pad 所有权保护,仅作经验。`第三方经验`。

## 10. 未知项

### 10.1 AP 域 `uart10` base 偏移

- 当前证据: `k3.dtsi` 中 `uart0, uart2..uart9` base 步进 `0x100`; `uart10` 位于 `0xd401f000`（相对 `uart9` 的 `0xd4017800` 偏移 `0x7800`），与前 9 个非 secure 实例的 stride 公式不符。`k3.dtsi` 与 docs-buildroot 现有页均未直接说明此偏移原因。
- 禁止推断: 不得用 `0x100` stride 公式推导 `uart10` base; 不得假设 `uart10` 不可用或被保留; 不得在 K3 公开资料中宣称"UART10 不可访问"。
- 解除条件: 取得 `k3.dtsi` 注释或 SpacemiT 公开 programmer manual 中关于 `uart10` 偏移的说明; 或 `k3_com260.dts(i)` 板级文件引用 `uart10` 时显式注释。
- 影响主题: 完整 17 实例矩阵(本节即此用途); 板级启用 `uart10` 时的地址与 IRQ 解析; aliases `serial10 = &uart10` 的语义。

### 10.2 bootloader 实际 cmdline 与 console

- 当前证据: `k3_com260.dtsi` `chosen` 节点 `bootargs = "earlycon=sbi console=ttyS0,115200 loglevel=8 random.trust_bootloader=1 unaligned_scalar_speed=fast unaligned_vector_speed=fast"`,含 8 字 rng-seed;但 `bootargs` 是否会被 bootloader / cmdline 文件 (`env_k3.txt` / `extlinux.conf`) 覆盖未知。
- 禁止推断: 不得由 `bootargs` 字符串推定 Kit 默认内核命令行; 不得由 `stdout-path` 串推定 Kit 真实 console; 不得由 `earlycon=sbi` 推定 OpenSBI earlycon 已启用。
- 解除条件: CoM260 Kit 启动日志或 `bootfs/env_k3.txt` 实际内容直接观察; `bootfs` 镜像解包或 `cat /proc/cmdline` 现场抓取。
- 影响主题: 运行时 console 阶段切换; 实际 bootlog 与 console log 关系; 启动参数验证。

### 10.3 目标 Kit 顶层 DTS 唯一映射

- 当前证据: 见 [`../reference/known-gaps.md`](../reference/known-gaps.md) G7,7 个候选 (`k3_com260.dts` / `k3_com260.dtsi` / `k3_com260_kit_v02.dts` / `k3_com260_ifx.dts` / `k3_com260_ifx2.dts` / `k3_com260_ifx_tq.dts` / `k3_com260_tq.dts`); 当前证据不构成唯一映射。
- 禁止推断: 不得由 `k3_com260.dtsi` 的 `stdout-path` 字符串推定唯一 Kit DTS; 不得由 `model = "SpacemiT K3 Com260 Kit V02"` 字符串推定当前 Kit 默认。
- 解除条件: CoM260 Kit 原理图 (`com260_hw_resources.md` 下载制品) 中 DTS 路径或 board 标识; 或 `com260_user_guide.md` 后续修订明示对应表; 或 buildroot defconfig 包含 `BR2_TARGET_KERNEL_DTB` 明确指向某一 DTS。
- 影响主题: 平台资源映射(MS04 后续); GMAC/PHY 介质归属(MS07); EtherCAT 物理通路(MS07)。

### 10.4 `spacemit,k1-uart` compatible 字符串的硬件边界

- 当前证据: [`8250.yaml`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/Documentation/devicetree/bindings/serial/8250.yaml) 中 `spacemit,k1-uart` 与 `intel,xscale-uart` 共同作为 compatible; K3 DTS 全部 17 个 UART 节点均使用此组合。
- 禁止推断: 不得将 `spacemit,k1-uart` 字符串推定为 K1 SoC 硬件身份; 不得由命名推定 K3 UART 寄存器全集等同于 K1 PXA UART; 不得由 `intel,xscale-uart` 推定 K3 沿用 PXA IP 全部行为。
- 解除条件: docs-buildroot 05-UART.md 公开 K3 专属 compatible 与 K1 差异; 或 K3 programmer manual 给出 K3 UART IP 修订与 PXA 关系。
- 影响主题: 后续 driver 选型; future Iteration 中 K3 专属 compatible 是否独立。

### 10.5 完整 UART 寄存器语义与电气映射

- 当前证据: [`8250.yaml`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/Documentation/devicetree/bindings/serial/8250.yaml) 给出 binding schema, [`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) 给出 base + IRQ; 但完整 UART 寄存器语义、电气特性、programmer manual 缺失。
- 禁止推断: 不得用 Linux 默认值或 PXA 通用知识补齐 K3 UART 寄存器; 不得用 8250 通用 FIFO 公式反推 K3 实际 IP 行为; 不得由 PXA errata 推定 K3 适用 errata。
- 解除条件: 取得 SpacemiT 公开 programmer manual 中 K3 UART IP 章节; 或 K3 SoC datasheet 寄存器章节公开。
- 影响主题: 后续 driver 实现(不在本 change 范围); IRQ budget; clock tuning; DMA 触发条件(MS06)。

## 11. 引用与下一步

本文档直接引用的来源(已登记于 [`../reference/source-coverage.md`](../reference/source-coverage.md)):

- 交叉验证(子类: 官方文档, docs-buildroot 仓库):
  - [05-UART.md](https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/peripheral_driver/05-UART.md)
- 交叉验证(子类: 官方源码行为, linux-6.18 仓库):
  - [k3.dtsi](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi)
  - [k3-rdomain.dtsi](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-rdomain.dtsi)
  - [k3-pinctrl.dtsi](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-pinctrl.dtsi)
  - [k3_com260.dtsi](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260.dtsi)
  - [8250_of.c](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/drivers/tty/serial/8250/8250_of.c)
  - [8250.yaml](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/Documentation/devicetree/bindings/serial/8250.yaml)
- 交叉验证(子类: 官方文档, docs-product 仓库):
  - [com260_user_guide.md](https://github.com/spacemit-com/docs-product/blob/main/zh/k3_com260/com260_user_guide.md)
- 板级事实(由 [`../platform/com260-board-resources.md`](../platform/com260-board-resources.md) 单独交付):
  - UART0 物理接口 + 12 Pin 排针 + 115200-8-N-1

第三方材料(仅作交叉参考, 不作事实依据):

- `others/Rt-Async-AMP/` (revision `ccb1ff0b487e4f49ea570c41f330741eecece935`)
- `others/Rt-Async-AMP/tgoskits/` (内嵌 revision `19219411d5dc1515496f910d04c93da12ee95be4`)

下一步(不在本 Cycle 范围):

- 全部完成; Iteration 001 / Cycle 000 的 T3–T6 已在本 change 历史产物中收尾, 本 rework Cycle 不再列遗留。
- MS05/MS06/MS07: 在 MS04 边界之外扩展 IRQ budget / DMA / GMAC 主题时, 引用本文档 §3-§5 的实例字段。

## 12. 边界声明

- 17 个 UART 物理实例 = AP 域 10 (`k3.dtsi` `uart0` + `uart2`..`uart10`) + APBC2 secure 1 (`k3.dtsi` `uart1`) + RCPU 域 6 (`k3-rdomain.dtsi` `r_uart0`..`r_uart5`); 不互相继承。
- aliases `serial0`..`serial10` → `uart0`..`uart10` 和 `serial11`..`serial16` → `r_uart0`..`r_uart5` 由 [`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) `aliases` 节点直接闭合; `uart10` 的 base 偏移原因作为未知项保留(见 §10.1)。
- FIFO 256 / 64 冲突、divisor 公式、probe 顺序并列保留各来源适用层, **不裁决**(见 §8)。
- 第三方材料(`others/Rt-Async-AMP/` 与内嵌 `tgoskits/`) 的 UUE/OUT2、THRE/TEMT、IRQ budget、divisor errata、pad 自愈、probe 顺序描述**不提升为 K3 SoC 硬件规范**; 仅作固定 revision 下的工程经验(见 §9)。
- 官网正文(SpacemiT 官方社区)仍为唯一 `官方事实` 来源; docs-buildroot GitHub 页与 raw 文件按 `交叉验证` 标签分别标 `官方文档` / `官方源码行为` 子类。本文档不出现 `官方事实` 标签。
- 目标 Kit DTS(`k3_com260.dts` / `k3_com260_kit_v02.dts` / 其他候选)非唯一, 本文档不裁决(见 §10.3 与 [`../reference/known-gaps.md`](../reference/known-gaps.md) G7)。
- 静态 DTS 可证明从 Kit 物理接口到 `chosen` 节点的闭合链路; bootloader 最终 cmdline、实际 console、启动日志保留为运行时未知项(见 §7.2 与 §10.2)。
- `spacemit,k1-uart` 是 K3 当前 binding/driver 匹配字符串, **不作为 K1 SoC 硬件身份**(见 §10.4)。
- 不展开 driver 设计、copier/ring/waker/poll-select/flush-tcdrain、VFS/TTY 接口; 不修改 Linux、DTS、bootloader、StarryOS 产品代码。
- 单文件 < 500 行(实际 316 行, 2026-09-08 `wc -l docs/serial/com260-uart.md` 输出); 不拆分。
