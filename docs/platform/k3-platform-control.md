> 来源: https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/peripheral_driver/01-PINCTRL.md（源端修订: unknown；观察日期: 2026-09-08）；https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/peripheral_driver/05-UART.md（源端修订: unknown；观察日期: 2026-09-08）；https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/peripheral_driver/16-Clock.md（源端修订: unknown；观察日期: 2026-09-08）；https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/peripheral_driver/Reset.md（源端修订: unknown；观察日期: 2026-09-08）
>
> 交叉验证: https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-08）；https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-rdomain.dtsi（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-08）；https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-pinctrl.dtsi（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-08）；https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/drivers/tty/serial/8250/8250_of.c（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-08）

# K3 平台控制资源

> 范围: K3 SoC 平台控制资源（pinctrl / clock / reset / APBC / CCU）按 **AP、APBC2 secure、RCPU** 三个物理域分别记录 provider 节点、关键 capability 和 UART consumer 依赖。`pinctrl-0` / `clocks` / `resets` / `reg-shift` / `interrupt-parent` 字段映射在此文档范围内。
> 不覆盖: 17 个 UART 物理实例的完整矩阵、CoM260 Kit 物理接口到节点的 console 链路、FIFO 256 vs 64 冲突、固定 revision 第三方经验（均属 Iteration 001 T3，由 `docs/serial/com260-uart.md` 单独交付，本 Cycle 不创建）。
> 边界: 物理层不互相继承。AP、APBC2 secure 与 RCPU 分别使用各自的 core/bus clock 和 reset provider；三域 UART 的 `gate` clock 均显式引用 `&syscon_mpmu CLK_MPMU_SLOW_UART`。来源只证明某域时，其他域不得自动继承。

## 目录

- [1. 概述与证据模型](#1-概述与证据模型)
- [2. 三域物理边界](#2-三域物理边界)
- [3. AP 域 provider 资源](#3-ap-域-provider-资源)
- [4. APBC2 secure 域 provider 资源](#4-apbc2-secure-域-provider-资源)
- [5. RCPU 域 provider 资源](#5-rcpu-域-provider-资源)
- [6. UART consumer 依赖（按域）](#6-uart-consumer-依赖按域)
- [7. 资源关系图](#7-资源关系图)
- [8. 未知项与边界](#8-未知项与边界)
- [9. 引用与下一步](#9-引用与下一步)
- [10. 边界声明](#10-边界声明)

## 1. 概述与证据模型

本文档按 `官方事实 / 交叉验证 / 推论 / 未知项` 四级证据等级标记，遵循 [`document-template.md`](../reference/document-template.md) 的统一规则：

- `官方事实`: 官网正文直接陈述（本文档范围内不直接命中——SpacemiT 官方社区页正文仍为 SPA 壳，来源身份详见 [source-coverage.md](../reference/source-coverage.md) 行 28-67）。
- `交叉验证`: 由官方 GitHub、DTS 或驱动行为佐证。子类在来源说明中标注：
  - `官方文档`: docs-buildroot 仓库内驱动说明页。
  - `官方源码行为`: `k3.dtsi` / `k3-rdomain.dtsi` / `k3-pinctrl.dtsi` raw 文件，以及 `8250_of.c` 中 `CONFIG_SOC_SPACEMIT` 条件下的 SpacemiT 适配。
  - `第三方经验`: `others/Rt-Async-AMP/` 用户提供的 K1 域 APBC2 描述与 8250 probe / 自愈分析。
- `推论`: 由多个事实推出但源页面未直接陈述（必须列出所依据的多项事实）。
- `未知项`: 当前证据不足；按 `当前证据 / 禁止推断 / 解除条件 / 影响主题` 四字段闭包。

本文档 90% 以上事实为 `交叉验证`（子类: `官方源码行为` / `官方文档`）；本文不出现 `官方事实` 标签。

## 2. 三域物理边界

| 域 | 地址段（来自 k3.dtsi syscon 节点） | 总线 | provider 节点名 | 引用 |
|---|---|---|---|---|
| AP | `syscon_apmu` @ `0xd4282800`；`syscon_apbc` @ `0xd4015000`；`syscon_mpmu` @ `0xd4050000` | AXI / APB / APBC | `pinctrl@d401e000`、`syscon_apbc`、`syscon_apmu`、`syscon_mpmu`、AP 域 reset 子树 | §3 |
| APBC2 secure | `syscon_apbc2` @ `0xf0610000` | APBC2 (低速 APB，独立地址域) | `syscon_apbc2`、secure 域 reset provider | §4 |
| RCPU | （独立地址空间） | RCPU 内部总线 | `&syscon_rcpu_uartctrl`（core/bus clock 与 reset）；共享 `&syscon_mpmu`（gate clock） | §5 |

> 物理归属证据等级: `交叉验证（子类: 官方源码行为）`（[k3.dtsi](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) 中 `syscon_mpmu`/`syscon_apmu`/`syscon_apbc`/`syscon_apbc2` 节点定义）+ `交叉验证（子类: 官方文档）`（[16-Clock.md](https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/peripheral_driver/16-Clock.md) 解释 `syscon_apbc`/`syscon_apmu` 域选择）。

## 3. AP 域 provider 资源

### 3.1 pinctrl provider

- 节点: `pinctrl@d401e000`（定义于 `k3.dtsi`，由 `k3-pinctrl.dtsi` 覆盖追加 `&pinctrl { ... }`）。`交叉验证（子类: 官方源码行为）`。
- 关键 capability: `K3_PADCONF(pin, func)` 宏复用 pad 编号与 func 编号。`交叉验证（子类: 官方文档）`（[01-PINCTRL.md](https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/peripheral_driver/01-PINCTRL.md)）。
- 覆盖示例: `k3-pinctrl.dtsi` 提供 `uart0_0_cfg`（pinmux `K3_PADCONF(149, 2)` tx / `K3_PADCONF(150, 2)` rx，bias-pull-up，drive-strength `25`/DS8）以及 `uart0_1_cfg`..`uart0_4_cfg`、`uart1_0_cfg` 等多组复用配置。`交叉验证（子类: 官方源码行为）`。
- 复用条目: `uart0_0_cfg` / `uart1_0_cfg` 是 CoM260 板级默认 pinctrl 候选；`uart0_2_cfg` / `uart0_3_cfg` / `uart0_4_cfg` 与 `uart1_1_cfg` 等用于其他引脚位置，板级 DTS 选定其中一组。

### 3.2 clock provider

- AP 域三个独立 syscon 节点（`交叉验证（子类: 官方源码行为）`，来自 [k3.dtsi](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi)）:
  - `syscon_apbc` @ `0xd4015000`: 提供低速 APB 外设时钟（含 UART0/UART2..UART10 的 `core`/`bus` 两项 clock）。
  - `syscon_apmu` @ `0xd4282800`: 提供 AP 主体外设时钟（不直接用于 UART consumer；用作 DMA/GMAC 等的时钟源）。
  - `syscon_mpmu` @ `0xd4050000`: 提供 AP、APBC2 secure 与 RCPU 三域 UART 的共享 `gate` clock（来自 `CLK_MPMU_SLOW_UART`）；不直接挂在各域的 core/bus clock provider 上。
- 域选择原则: `交叉验证（子类: 官方文档）`（[16-Clock.md](https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/peripheral_driver/16-Clock.md) 明示"低速 APB 外设更多挂在 `syscon_apbc`；AP 主体外设更多挂 `syscon_apmu`"）。`syscon_mpmu` 的 UART 角色不在该页直接列出。
- 边界: AP 域 UART consumer 的 `clocks` 字段三项 `core` / `bus` / `gate` 全部在此域内（见 §6.1）。

### 3.3 reset provider

- 节点: AP 域 reset 由 `k3.dtsi` 内独立 provider 提供（`syscon_apbc` 的 `RESET_APBC_UARTx` 描述符）。`交叉验证（子类: 官方源码行为）`。
- 关键 capability: docs-buildroot 明示"板级 DTS 一般不需要配置 Provider"（[Reset.md](https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/peripheral_driver/Reset.md)）。`交叉验证（子类: 官方文档）`。
- 边界: 同一 provider 可被多个 consumer 引用；板级 DTS 不可新增 provider 节点。

### 3.4 APBC / CCU 边界

- APBC: AP 域低速 APB 总线，挂在 `syscon_apbc` 域下；UART0/UART2..UART10 的 bus clock 位于此。`交叉验证（子类: 官方文档）`。
- CCU: AP 域顶层时钟单元（`syscon_apmu` 与 AP 时钟子树之间的中间层），不直接出现在 consumer 字段；本文档不展开 CCU 内部寄存器。`推论`（基于 `syscon_apmu` 节点存在与 docs-buildroot 16-Clock.md 域选择的共同推断）。
- MPMU: AP 域独立 syscon 节点（`0xd4050000`），提供 UART `gate` clock 与 `osc_32k` / `vctcxo_*` 等基础时钟。`交叉验证（子类: 官方源码行为）`。

## 4. APBC2 secure 域 provider 资源

### 4.1 域独立性

- 地址域: `syscon_apbc2` 位于 `0xf0610000`，与 AP 域 `syscon_apbc`（`0xd4015000`）处于不同地址段；不互相寻址。`交叉验证（子类: 官方源码行为）`。
- secure UART1 节点: `uart1: serial@f0612000`（`k3.dtsi` 中定义于 `syscon_apbc2` 地址段内）。`交叉验证（子类: 官方源码行为）`。
- 关键 capability: secure 域 APBC2 时钟 / reset 完全独立于 AP 域；secure UART1 的 `clocks` 字段直接引用 `syscon_apbc2` 的 `CLK_APBC2_SEC_UART1` / `CLK_APBC2_SEC_UART1_BUS`，以及 `syscon_mpmu` 的 `CLK_MPMU_SLOW_UART`。`交叉验证（子类: 官方源码行为）`。

### 4.2 pinctrl / clock / reset provider

- pinctrl: secure 域的复用需求由 AP 域 `pinctrl@d401e000` 覆盖（同一物理 pad 上对 secure UART1 的复用配置在 `k3-pinctrl.dtsi` 中提供；secure 域不另设独立 pinctrl 节点）。`交叉验证（子类: 官方源码行为）`。
- clock: `syscon_apbc2`（secure 域专用 syscon，独立于 `syscon_apbc` 与 `syscon_apmu`）。`交叉验证（子类: 官方源码行为）`。
- reset: secure 域 reset 由 `syscon_apbc2` 的 `RESET_APBC2_SEC_UART1` 描述符提供。`交叉验证（子类: 官方源码行为）`。
- 边界: 不允许跨域引用；secure 域与 AP 域 reset_id 编号独立。

### 4.3 APBC2 边界与 K1 起源

- APBC2: secure 域低速 APB 总线，承载 secure world 外设；与 APBC（AP 域低速 APB）不互相继承。`交叉验证（子类: 官方源码行为）`。
- K1 起源: secure 域字段命名沿用 K1 SoC 体系（K3 兼容 K1）；本仓库不登记 K1 域 `others/Rt-Async-AMP/` 第三方材料为 K3 官方事实。`交叉验证（子类: 第三方经验）` 仅作命名体系交叉参考。

## 5. RCPU 域 provider 资源

### 5.1 pinctrl / clock / reset provider

- 节点: `k3.dtsi` 定义 `&syscon_rcpu_uartctrl` syscon；`k3-rdomain.dtsi` 定义 RCPU UART consumer 并引用该 provider。它承担 RCPU UART 的 core/bus clock 与 reset provider 角色。`交叉验证（子类: 官方源码行为）`。
- 关键 capability: RCPU 的 core/bus clock 与 reset 独立于 AP 域的 `syscon_apbc` / `syscon_apmu`；UART `gate` clock 则与 AP、APBC2 secure 域一样引用共享的 `&syscon_mpmu`。`交叉验证（子类: 官方源码行为）`。
- 边界: RCPU 域不存在独立 pinctrl provider 节点；RCPU UART 的引脚复用由 AP 域 `pinctrl@d401e000` 覆盖（K3 SoC 内部路由，非 RCPU 自有 pad）。`交叉验证（子类: 官方源码行为）`。

### 5.2 RCPU 域 UART consumer 物理属性

| 属性 | 值 | 证据等级 |
|---|---|---|
| 实例数 | 6（`r_uart0` .. `r_uart5`） | `交叉验证（子类: 官方源码行为）`（[k3-rdomain.dtsi](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-rdomain.dtsi)） |
| base 范围 | `0xc0881000` .. `0xc0881500`（实例间距 `0x100` = 256 字节；总跨度 `0x500`，覆盖 `0x600` 字节地址窗口） | `交叉验证（子类: 官方源码行为）` |
| `reg-shift` | `<2>`（寄存器步进 4 字节） | `交叉验证（子类: 官方源码行为）` |
| compatible | `"spacemit,k1-uart"`，`"intel,xscale-uart"`（双字符串） | `交叉验证（子类: 官方源码行为）` |
| 标志 | `spacemit,rcpu-uart`（RCPU 域标识，驱动据此选择 RCPU 时钟策略） | `交叉验证（子类: 官方源码行为）` |
| `clock-names` | `"core"`，`"bus"`，`"gate"` | `交叉验证（子类: 官方源码行为）` |
| `clocks` (`core`) | `<&syscon_rcpu_uartctrl CLK_RCPU_UARTCTRL_RUART0..5>` | `交叉验证（子类: 官方源码行为）` |
| `clocks` (`bus`) | `<&syscon_rcpu_uartctrl CLK_RCPU_UARTCTRL_RUART0_BUS..5_BUS>` | `交叉验证（子类: 官方源码行为）` |
| `clocks` (`gate`) | `<&syscon_mpmu CLK_MPMU_SLOW_UART>` | `交叉验证（子类: 官方源码行为）` |
| `resets` provider | `<&syscon_rcpu_uartctrl RESET_RCPU_UARTCTRL_RUART0..5>` | `交叉验证（子类: 官方源码行为）` |
| `interrupt-parent` | `&saplic` | `交叉验证（子类: 官方源码行为）` |
| `interrupts` | 251..256 | `交叉验证（子类: 官方源码行为）` |
| `fifo-size` | 256（DTS 独立配置；不与 `tx-threshold` 互相决定） | `交叉验证（子类: 官方文档）` |
| `tx-threshold` | 32（DTS 独立配置；用于计算 `tx_loadsz = fifosize - tx_threshold`，不决定 `fifo-size`） | `交叉验证（子类: 官方文档）` |

> RCPU UART 由 K3 SoC 内部实时核固件使用；通用 8250 OF 探测路径（[8250_of.c](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/drivers/tty/serial/8250/8250_of.c)）在 `CONFIG_SOC_SPACEMIT` 条件下通过 `spacemit,rcpu-uart` 属性为 RCPU 选择 `baud*16` 动态时钟策略，与 ACPU 走 `spacemit_acpu_match_clk_rate` 固定查找表不同。`交叉验证（子类: 官方源码行为）` + `交叉验证（子类: 官方文档）`。

## 6. UART consumer 依赖（按域）

### 6.1 AP 域 UART consumer

- 实例数: 11（`uart0`，`uart2`..`uart10`，定义于 `k3.dtsi`）。`交叉验证（子类: 官方源码行为）`。
- secure 例外: `uart1: serial@f0612000` 在 `k3.dtsi` 节点列表中，但位于 APBC2 地址域（见 §4），按域划分归入 APBC2 secure 域。本节"AP 域 11 实例"含 `uart0` + `uart2`..`uart10` 共 10 个 AP 域实例，加上 `uart1` 共 11 个 K3 顶层 UART 节点；与 aliases `serial0..10` 一致。`交叉验证（子类: 官方源码行为）`。
- 关键 capability（以 `uart0: serial@d4017000` 为代表）:
  - `reg`: 来自 `k3.dtsi`（AP UART0 基准 `0xd4017000`，reg-shift `<2>`）。
  - `interrupts`: `<42 IRQ_TYPE_LEVEL_HIGH>`（UART0 单独值；其他 AP 域实例 IRQ 详见 Iteration 001 T3，由 `docs/serial/com260-uart.md` 单独交付）。
  - `interrupt-parent`: `<&saplic>`（与 secure UART1 一致；并非 `&plic`）。`交叉验证（子类: 官方源码行为）`。
  - `pinctrl-0` / `pinctrl-names`: 引用 `k3-pinctrl.dtsi` 中 `uart0_0_cfg` 等；`&pinctrl` 由 `pinctrl@d401e000` 提供。
  - `clocks`: **三项** `<&syscon_apbc CLK_APBC_UART0>`，`<&syscon_apbc CLK_APBC_UART0_BUS>`，`<&syscon_mpmu CLK_MPMU_SLOW_UART>`。
  - `clock-names`: `"core"`，`"bus"`，`"gate"`。
  - `resets`: `<&syscon_apbc RESET_APBC_UART0>`。
  - `reg-shift`: `<2>`。
- AP 域 IRQ 序列: 不能用单一公式描述；`uart0`=42，`uart1`（secure，APBC2）=43，`uart2`=44，… 每节点独立字段（详见 [k3.dtsi](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi)）。`交叉验证（子类: 官方源码行为）`。
- 绑定/driver 参考: 8250 UART 设备树绑定支持 `spacemit,k1-uart` + `intel,xscale-uart`（`intel,xscale-uart` 触发 `PORT_XSCALE` 路径）；驱动为通用 8250 OF 探测（[8250_of.c](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/drivers/tty/serial/8250/8250_of.c)）。本文不展开 API/reg 字段（属于 Iteration 001 T3）。

### 6.2 RCPU 域 UART consumer

- 实例数: 6（`r_uart0` .. `r_uart5`，由 `k3-rdomain.dtsi` 定义）。`交叉验证（子类: 官方源码行为）`。
- 关键 capability:
  - `reg`: 6 个 RCPU UART 连续位于 `0xc0881000` .. `0xc0881500`（实例间距 `0x100` 字节）。
  - `reg-shift`: `<2>`。
  - `interrupts`: 251 .. 256。
  - `interrupt-parent`: `<&saplic>`。
  - compatible: `"spacemit,k1-uart"`，`"intel,xscale-uart"`（双字符串）。
  - `spacemit,rcpu-uart` 标志存在（驱动据此选择 RCPU 时钟策略）。
  - `clocks` (`core`): `<&syscon_rcpu_uartctrl CLK_RCPU_UARTCTRL_RUART0..5>`。
  - `clocks` (`bus`): `<&syscon_rcpu_uartctrl CLK_RCPU_UARTCTRL_RUART0_BUS..5_BUS>`。
  - `clocks` (`gate`): `<&syscon_mpmu CLK_MPMU_SLOW_UART>`。
  - `clock-names`: `"core"`，`"bus"`，`"gate"`。
  - `resets`: `<&syscon_rcpu_uartctrl RESET_RCPU_UARTCTRL_RUART0..5>`。
  - `pinctrl`: 不直接引用（路由由 AP 域 `pinctrl@d401e000` 负责；见 §5.1）。

### 6.3 aliases 静态映射（来自 `k3.dtsi` aliases 节点）

`k3.dtsi` aliases 节点直接列出 17 项映射，证据已闭合（`交叉验证（子类: 官方源码行为）`）：

| alias | 节点 | 物理域 | base |
|---|---|---|---|
| `serial0` | `&uart0` | AP | `0xd4017000` |
| `serial1` | `&uart1` | APBC2 secure | `0xf0612000` |
| `serial2` | `&uart2` | AP | `0xd4017100` |
| `serial3` | `&uart3` | AP | `0xd4017200` |
| `serial4` | `&uart4` | AP | `0xd4017300` |
| `serial5` | `&uart5` | AP | `0xd4017400` |
| `serial6` | `&uart6` | AP | `0xd4017500` |
| `serial7` | `&uart7` | AP | `0xd4017600` |
| `serial8` | `&uart8` | AP | `0xd4017700` |
| `serial9` | `&uart9` | AP | `0xd4017800` |
| `serial10` | `&uart10` | AP | `0xd401f000` |
| `serial11` | `&r_uart0` | RCPU | `0xc0881000` |
| `serial12` | `&r_uart1` | RCPU | `0xc0881100` |
| `serial13` | `&r_uart2` | RCPU | `0xc0881200` |
| `serial14` | `&r_uart3` | RCPU | `0xc0881300` |
| `serial15` | `&r_uart4` | RCPU | `0xc0881400` |
| `serial16` | `&r_uart5` | RCPU | `0xc0881500` |

> alias `serial1` 指向 secure 域 `uart1`；`uart10` 物理基地址与前序 AP UART 间距不同（`0xd4017800` → `0xd401f000`，间距 `0x7800`），与 stride 模式不一致；其余 AP UART 间距 `0x100`。Iteration 001 T4 负责修正 MS03 `com260-image-and-dts.md` 的 `serial0` 静态映射旧表述。

### 6.4 第三方 8250 probe / 自愈扩展

- 第三方 `others/Rt-Async-AMP/` 含 8250 probe 顺序、pad 自愈、固定 divisor 序列的描述。`交叉验证（子类: 第三方经验）`，revision 与适用域见各材料 README。
- 禁止: 不把第三方 bit/固定 divisor/pad 自愈/probe 顺序提升为 K3 硬件规范。`边界声明`。

## 7. 资源关系图

```
                ┌──────────────────────────────────────────┐
                │            K3 SoC 平台控制               │
                └──────────────────────────────────────────┘
                                  │
        ┌─────────────────────────┼─────────────────────────┐
        │                         │                         │
   AP 域 (§3)              APBC2 secure 域 (§4)       RCPU 域 (§5)
   pinctrl@d401e000        syscon_apbc2 @0xf0610000   &syscon_rcpu_uartctrl
   syscon_apbc @d4015000   (secure-only)              (core/bus clocks + resets)
   syscon_apmu @d4282800   K1 起源命名
   AP 域 reset subtree
        │                         │                         │
        ▼                         ▼                         ▼
   10 × AP UART              1 × secure UART1          6 × RCPU UART
   (k3.dtsi uart0,           (k3.dtsi uart1)            (k3-rdomain.dtsi)
    uart2..uart10)
        │                                                   │
        │                  路由/复用                         │
        └─────────► pinctrl@d401e000 (k3-pinctrl.dtsi) ◄────┘

   共享 gate clock provider: syscon_mpmu @d4050000
   └──────────────► AP UART / secure UART1 / RCPU UART
```

UART consumer 字段映射（简化）：

| consumer 字段 | AP 域 (uart0, uart2..uart10) | APBC2 secure (uart1) | RCPU 域 (r_uart0..r_uart5) |
|---|---|---|---|
| `pinctrl-0` | `&pinctrl uart0_0_cfg` 等 | `&pinctrl uart1_0_cfg` 等 | 不直接引用 |
| `clocks` (`core`) | `&syscon_apbc CLK_APBC_UARTx` | `&syscon_apbc2 CLK_APBC2_SEC_UART1` | `&syscon_rcpu_uartctrl CLK_RCPU_UARTCTRL_RUARTx` |
| `clocks` (`bus`) | `&syscon_apbc CLK_APBC_UARTx_BUS` | `&syscon_apbc2 CLK_APBC2_SEC_UART1_BUS` | `&syscon_rcpu_uartctrl CLK_RCPU_UARTCTRL_RUARTx_BUS` |
| `clocks` (`gate`) | `&syscon_mpmu CLK_MPMU_SLOW_UART` | `&syscon_mpmu CLK_MPMU_SLOW_UART` | `&syscon_mpmu CLK_MPMU_SLOW_UART` |
| `clock-names` | `"core", "bus", "gate"` | `"core", "bus", "gate"` | `"core", "bus", "gate"` |
| `resets` | `&syscon_apbc RESET_APBC_UARTx` | `&syscon_apbc2 RESET_APBC2_SEC_UART1` | `&syscon_rcpu_uartctrl RESET_RCPU_UARTCTRL_RUARTx` |
| `reg-shift` | `<2>` | `<2>` | `<2>` |
| `interrupts` | 42, 44..（UART0=42, UART2=44, …，每节点独立） | 43 | 251..256 |
| `interrupt-parent` | `&saplic` | `&saplic` | `&saplic` |
| `compatible` | `spacemit,k1-uart` + `intel,xscale-uart` | `spacemit,k1-uart` + `intel,xscale-uart` | `spacemit,k1-uart` + `intel,xscale-uart`，带 `spacemit,rcpu-uart` 标志 |

## 8. 未知项与边界

### 8.1 三域 provider 身份的官方依据

- 当前证据: docs-buildroot 16-Clock.md 明示 `syscon_apbc` / `syscon_apmu` 域选择；docs-buildroot 01-PINCTRL.md / Reset.md / 05-UART.md 各自覆盖一部分域。`k3.dtsi` 与 `k3-rdomain.dtsi` 在 raw 文件中体现 AP、APBC2 secure 与 RCPU 域的 provider 节点。`syscon_mpmu` 作为 UART `gate` clock provider 的角色未在 docs-buildroot 16-Clock.md 直接列出。
- 禁止推断: 不得根据 16-Clock.md 的 `syscon_apbc` / `syscon_apmu` 表述推论 APBC2 secure 域也使用同一 syscon 节点；不得把 `syscon_mpmu` 的存在视为"可作通用 `gate` clock 模板"。
- 解除条件: docs-buildroot 增补 `syscon_mpmu` 与 `syscon_apbc2` 域驱动说明；或 SpacemiT 官方公开 secure 域完整字段。
- 影响主题: APBC2 secure 域相关功能（trustzone、secure world 资源）；不在本 Cycle 范围内，但影响未来 Iteration 边界。

### 8.2 APBC2 secure 域 K1 起源 vs K3 复用

- 当前证据: `k3.dtsi` 中 secure 域字段命名沿用 K1 SoC 体系（`CLK_APBC2_SEC_UART1` 命名风格）；第三方 `others/Rt-Async-AMP/` 含 K1 域 APBC2 描述。
- 禁止推断: 不得把 `others/Rt-Async-AMP/` 中 K1 域字段细节作为 K3 SoC 公开事实；不得在 K3 文档中使用 K1 SoC 文档版本号。
- 解除条件: SpacemiT 官方发布 K3 SoC APBC2 secure 域驱动说明；或 K3 datasheet 公开 secure 域寄存器布局。
- 影响主题: secure world 资源规划、trustzone 集成；MS05 之后可能展开。

### 8.3 RCPU syscon 时序与初始化顺序

- 当前证据: `k3.dtsi` 定义 `&syscon_rcpu_uartctrl` syscon 节点；`k3-rdomain.dtsi` 定义 RCPU UART consumer 并引用该 provider。docs-buildroot 16-Clock.md 仅说明 AP/APBC2/APMU 域选择，未直接覆盖 RCPU 域时钟。
- 禁止推断: 不得把 AP 域 clock provider 初始化时序直接用于 RCPU 域；不得假设 RCPU UART 在 AP Linux 启动前已可用。
- 解除条件: docs-buildroot 增补 RCPU 域 clock 说明；或 `k3.dtsi` 注释中提供 RCPU 域初始化顺序。
- 影响主题: 实时核固件加载顺序、AMP 跨核通信（MS09）。

### 8.4 第三方 8250 probe / 自愈与官方 8250_of.c 差异

- 当前证据: 第三方 `others/Rt-Async-AMP/` 含 8250 probe 顺序、pad 自愈、固定 divisor 序列的描述；Linux 6.18 `k3-br-v1.0.y` 分支的 [8250_of.c](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/drivers/tty/serial/8250/8250_of.c) 在 `CONFIG_SOC_SPACEMIT` 条件下包含 `spacemit_8250_set_termios` 与 `spacemit_acpu_match_clk_rate` 两个 K3/SpacemiT 专属函数（驱动据此分别为 ACPU 走固定查找表、为 RCPU 走 `baud*16` 动态时钟）。
- 禁止推断: 不得把第三方 probe 顺序与 pad 自愈逻辑视为 K3 SoC 硬件规范；不得在 T2 范围内把 `spacemit_8250_set_termios` 展开为正文事实（属于 Iteration 001 T3）。
- 解除条件: 取得 `k3.dtsi` 或 `k3-rdomain.dtsi` 注释中关于 K3 专属 probe 顺序的说明；或 docs-buildroot 05-UART.md 增补 K3 专属章节。
- 影响主题: 异步 UART 驱动设计（属于未来 Iteration，不在本 change 范围）。

### 8.5 Pad 自愈 / 复用恢复

- 当前证据: 第三方 `others/Rt-Async-AMP/` 含 pad 自愈与复用恢复流程。
- 禁止推断: 不得把 pad 自愈视为 `k3-pinctrl.dtsi` 提供的标准能力；不得在 K3 平台控制文档中将自愈流程列为通用 provider 责任。
- 解除条件: SpacemiT 官方在 docs-buildroot 01-PINCTRL.md 中说明 pad 自愈与复用恢复的正式流程。
- 影响主题: 复用错误恢复、运行时 pinctrl 切换；超出本 change 范围。

### 8.6 AP 域 `uart10` base 偏移

- 当前证据: `uart0, uart2..uart9` 在 `k3.dtsi` 节点列表中 base 连续（`0xd4017000` + `N × 0x100`），但 `uart10` 位于 `0xd401f000`（间距 `0x7800`）。`k3.dtsi` 与 docs-buildroot 现有页均未直接说明此偏移原因。
- 禁止推断: 不得用 `0x100` stride 公式推导 `uart10` base；不得假设 `uart10` 不可用。
- 解除条件: 取得 `k3.dtsi` 注释说明 `uart10` 偏移；或 `k3_com260.dts(i)` 板级文件引用 `uart10` 时显式注释。
- 影响主题: Iteration 001 T3 完整 17 实例表与 T4 aliases 修正。

## 9. 引用与下一步

本文档直接引用的来源（已登记于 [source-coverage.md](../reference/source-coverage.md)）：

- 交叉验证（子类: 官方文档，docs-buildroot 仓库）:
  - [01-PINCTRL.md](https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/peripheral_driver/01-PINCTRL.md)
  - [05-UART.md](https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/peripheral_driver/05-UART.md)
  - [16-Clock.md](https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/peripheral_driver/16-Clock.md)
  - [Reset.md](https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/peripheral_driver/Reset.md)
- 交叉验证（子类: 官方源码行为，linux-6.18 仓库）:
  - [k3.dtsi](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi)
  - [k3-rdomain.dtsi](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-rdomain.dtsi)
  - [k3-pinctrl.dtsi](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-pinctrl.dtsi)
  - [8250_of.c](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/drivers/tty/serial/8250/8250_of.c)

第三方材料（仅作交叉参考，不作事实依据）:

- `others/Rt-Async-AMP/` 用户提供的 K1 域 APBC2 描述与 8250 probe / 自愈分析；本仓库只读，不纳入主仓库 diff。

下一步（不在本 Cycle 范围）:

- Iteration 001: 由 T3 创建串口主题文档（目标 `docs/serial/com260-uart.md`，本 Cycle 不创建），聚合 17 路 UART 实例完整矩阵、DTS 属性、CoM260 UART0/console 路径、来源冲突与固定 revision 第三方经验。
- Iteration 001 T4: 勘误 [com260-image-and-dts.md](../boot/com260-image-and-dts.md) `serial0` 静态映射的旧未知项。
- Iteration 001 T5: 按 [known-gaps.md](../reference/known-gaps.md) G1-G7 复核新增未知项。
- Iteration 001 T6: 更新 [index.md](../index.md) 导航。

## 10. 边界声明

- AP、APBC2 secure、RCPU 三个域的物理层事实不互相继承；来源只证明某域时，其他域不得自动获得。
- 17 个 UART 物理实例 = AP 域 10（`k3.dtsi` `uart0` + `uart2`..`uart10`）+ APBC2 secure 1（`k3.dtsi` `uart1`）+ RCPU 域 6（`k3-rdomain.dtsi` `r_uart0`..`r_uart5`）；不在本文档内展开完整实例表。
- aliases `serial0`..`serial16` 的 17 项节点映射直接来自 `k3.dtsi` aliases 节点（见 §6.3），本表闭合；`uart10` 的 base 偏移原因作为未知项保留。
- 第三方材料（`others/Rt-Async-AMP/`）的 bit/divisor/pad 自愈/probe 顺序描述不提升为 K3 SoC 硬件规范。
- 官网正文（SpacemiT 官方社区）仍为唯一 `官方事实` 来源；docs-buildroot GitHub 页与 raw 文件按 `交叉验证` 标签分别标 `官方文档` / `官方源码行为` 子类。本文档不出现 `官方事实` 标签。
- 目标 Kit DTS（`k3_com260.dts` / `k3_com260_kit_v02.dts` / 其他候选）非唯一，本文档不裁决。
- 不展开驱动设计、IRQ controller、DMA、GMAC、CCU 内部寄存器、运行时 console 选择、FIFO 256 vs 64 冲突、固定 revision 第三方经验。
