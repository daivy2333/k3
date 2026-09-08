## Context

MS03 已建立 K3 SoC、CoM260 模组/Kit、启动链和 DTS 的分层基线。当前产品文档只确认 CoM260 Kit 的 UART0 物理接口、115200-8-N-1，以及共享 DTSI 中的 `stdout-path = "serial0:115200"`；`com260-image-and-dts.md` 仍把 `serial0` 映射写成未知项，并把运行时 console 误交给 MS07。

2026-09-08 的 Phase 2 复核使用主仓库 revision `bcdb076eb8fa6c13010e05438c947a560b9b43ea`。官方 `docs-buildroot` UART 页面直接说明：K3 当前有 11 路 AP UART 与 6 路 RCPU UART，使用 `8250_of.c` 和 `8250.yaml`；两域节点均使用 `spacemit,k1-uart`, `intel,xscale-uart`，RCPU 节点另有 `spacemit,rcpu-uart`。官方 `k3.dtsi` 当前把 `serial0..10` 映射到 `uart0..10`，把 `serial11..16` 映射到 `r_uart0..5`；`uart0` 位于 `0xd4017000`、IRQ 42、`reg-shift=2`、`reg-io-width=4`、`fifo-size=256`、`tx-threshold=32`，状态为 `okay`。共享 `k3_com260.dtsi` 使能 `uart0` 与 `uart0_0_cfg`，同时设置 `stdout-path` 和 `console=ttyS0,115200`。

平台控制事实分为三类：官网 PINCTRL/Clock/Reset 页面仍为 `partially-observed`；官方 GitHub 文档、DTS、binding 和 driver 可提供交叉验证或官方源码行为；Rt-Async-AMP/tgoskits 只提供固定 commit 下的第三方经验。Rt-Async-AMP revision 为 `ccb1ff0b487e4f49ea570c41f330741eecece935`，内嵌 tgoskits revision 为 `19219411d5dc1515496f910d04c93da12ee95be4`，均与 Explorer 捕获一致。

第三方实现揭示了必须保留来源边界的冲突。Rt-Async-AMP 的 RCPU PXA UART 使用 32-bit/stride-4 MMIO、固定 divisor 8、UUE/OUT2 和轮询 THRE；AP UART5 实验实现还会读取或改写 APBC/MPMU，并在发送前重写 pad83。tgoskits 的 PXA UART 使用 64-byte FIFO、PXA divisor errata 序列、32-pass/256-sample IRQ budget、THRE mask/rearm、TEMT idle 和短写。它们不能覆盖官方 K3 DTS 的 FIFO 256 或证明 CoM260 Kit 的硬件规范。

本仓库没有运行时状态、调用链或代码测试。实施链路是：登记精确来源 → 建立平台控制资源图 → 建立 UART/console 事实包 → 修正 MS03 已解除的静态未知项 → 更新导航与必要缺口 → 运行内容、链接、行数、OpenSpec 和 diff 验证。

## Goals / Non-Goals

**Goals:**

- 形成 `k3-platform-control.md` 和 `com260-uart.md` 两篇可独立引用的主题文档。
- 分开表达 AP、Secure APBC2 与 RCPU 的 pinctrl、clock、reset、MMIO 和 IRQ 资源。
- 建立 UART 实例、DTS 属性、console 静态路径、物理接口和运行时未知项之间的可追溯关系。
- 将 UUE/OUT2、THRE/TEMT、IRQ budget、divisor errata 和初始化顺序保存为固定 revision 的第三方经验。
- 修正 MS03 中已经被直接 DTS 证据解除的 `serial0` 未知项和错误 milestone 归属。

**Non-Goals:**

- 不设计或实现 UART 驱动、异步数据面、copier、ring、waker、flush/tcdrain 或 TTY/VFS 接口。
- 不修改 Linux、DTS、bootloader、Rt-Async-AMP、tgoskits 或 StarryOS 产品代码。
- 不将 K1、Pico、其他 CoM260 变体或第三方实测提升为当前 Kit 硬件事实。
- 不展开中断控制器、DMA/IOMMU、GMAC/PHY 或存储数据面。
- 不通过构建第三方工程或上板测试为文档事实制造新的运行时证据。

## Decisions

### D1：平台控制和 UART/console 分成两篇正文

`docs/platform/k3-platform-control.md` 负责 provider、资源域和初始化依赖；`docs/serial/com260-uart.md` 负责 UART 实例、寄存器访问属性、板级接口、console 路径和第三方经验。平台文档不复制 17 路完整实例表，串口文档不展开所有 clock/reset provider 的通用实现。

替代方案是把全部内容写进一篇 UART 文档。该方案会混合平台 provider 与串口 consumer，并增加超过 450 行后难以按责任拆分的风险，因此不采用。

### D2：使用“来源层 × 资源域”二维模型

每项事实同时标明来源层和资源域。来源层固定为 `官方事实`、`交叉验证`、`官方源码行为`、`第三方经验`、`推论` 或 `未知项`；其中产品模板要求的四级证据仍以 `官方事实/交叉验证/推论/未知项` 为主标签，`官方源码行为` 和 `第三方经验`作为交叉验证的子类写入来源说明。资源域至少区分 AP、APBC2 secure、RCPU 和板级物理连接。

替代方案是按 UART 编号合并事实。AP `uart0` 与 RCPU `r_uart0` 的编号相似但地址、clock/reset 和 IRQ 不同，合并会产生错误继承，因此不采用。

### D3：精确来源先登记，官网身份不升级

Iteration 000 在引用前登记四个 docs-buildroot 对应页以及平台/UART 直接使用的 raw DTS、binding 和 driver URL。已登记的 `k3.dtsi`、`k3_com260.dtsi` 不重复新增；新增候选至少包括：

- `docs-buildroot/.../01-PINCTRL.md`
- `docs-buildroot/.../05-UART.md`
- `docs-buildroot/.../16-Clock.md`
- `docs-buildroot/.../Reset.md`
- `linux-6.18/.../k3-rdomain.dtsi`
- `linux-6.18/.../k3-pinctrl.dtsi`
- `linux-6.18/.../drivers/tty/serial/8250/8250_of.c`
- `linux-6.18/.../Documentation/devicetree/bindings/serial/8250.yaml`

Act 只登记实际直接打开并用于正文的 URL，以实际观察日期和可见内容填写字段。官网对应行保持 `official-doc/partially-observed`；GitHub 页面和源码为 `supporting-source`，不得代填官网修订。

### D4：平台控制文档使用 provider-consumer 资源图

平台正文先列 AP/APBC2/RCPU provider，再按 UART consumer 表达 `pinctrl → clock/gate → reset → controller → IRQ` 的可证依赖。AP 的 `syscon_apbc`、`syscon_apbc2`、`syscon_mpmu` 与 RCPU 的 UART control provider 分开；寄存器地址和 bit 语义只有官方来源直接支持时才进入交叉验证事实。

Rt-Async-AMP 中“provider 在 consumer probe 前完成”“整寄存器写末端 clock”“pad83 自愈”等只进入第三方经验和风险，不能成为 K3 通用初始化要求。

### D5：UART 表以 DTS 字段为源码基线，以 datasheet 数量为能力基线

串口正文分别给出 11 路 AP、6 路 RCPU 的实例清单，并记录地址、IRQ、状态和适用 provider。`reg-shift=2` 与 `reg-io-width=4` 解释为当前 DTS/driver 的 32-bit、stride-4 访问契约；`fifo-size=256` 与 `tx-threshold=32` 记录为当前 K3 DTS/8250 配置行为，不推广成不可变 TRM 常量。

`spacemit,k1-uart` 是 K3 当前 binding/driver 匹配字符串，不作为 K1 硬件身份。tgoskits 的 FIFO 64 单独进入冲突表，解除条件为当前 K3 programmer manual、官方 errata 或能解释适用 IP revision 的官方材料。

### D6：console 使用四段路径并保留运行时边界

console 路径固定拆成：Kit UART0 RX/TX 与 115200-8-N-1 → `uart0` 节点与 `uart0_0_cfg` → `serial0 = &uart0` 与 `stdout-path` → `earlycon=sbi` / `console=ttyS0,115200`。前三段是当前板级资料和静态 DTS 的交叉验证；bootloader 最终环境、实际 cmdline 和启动日志仍为未知项。

MS03 的 `com260-image-and-dts.md` 只做有限勘误：将 aliases 未解析项改为已由当前 `k3.dtsi` 闭合，并把实际 console/cmdline 的影响主题归到 MS04。不得顺手改写其他 DTS、CMA、GMAC 或镜像未知项。

### D7：第三方经验使用固定分类，不产生驱动设计需求

第三方内容按 `寄存器访问`、`初始化`、`发送完成`、`IRQ/RX`、`clock/pinctrl 风险` 五类整理，并在每类标出仓库 revision、实际文件和适用域。UUE/OUT2、PXA errata #20/#75、THRE/TEMT、32-pass/256-sample budget、短写和 pad ownership 只能作为后续驱动调查输入。

替代方案是把第三方序列融合成一套“推荐初始化”。来源之间的 clock、FIFO、IRQ 可达性和板级走线不同，融合会制造未经证明的硬件契约，因此不采用。

### D8：分两个 Iteration 形成独立稳定基线

Iteration 000“平台控制与来源基线”完成精确来源登记和平台控制文档。其稳定结果是 UART 文档可以引用已经分域的 pinctrl/clock/reset provider，而无需重复解释平台控制。

Iteration 001“UART、console 与跨文档收尾”完成 UART 文档、MS03 勘误、条件性缺口更新和总入口导航。两轮的诊断边界分别是 provider/来源错误与 UART/console/冲突错误，工作量可独立验证。

### D9：验证直接检查文档行为

测试见证使用文件或章节不存在、`serial0` 旧表述仍存在、导航缺少链接等变更前可观察状态。GREEN 直接检查首行、来源登记、AP/RCPU 字段、console 链、冲突表、未知项、相对链接和行数，并运行 `openspec validate --strict` 与 `git diff --check`。

Persisted Evidence 为 `none`。验证输出可低成本重跑，Act Response 足以保存命令、退出码和每项不超过 20 行的决定性输出。

## Risks / Trade-offs

- [官网 PINCTRL/Clock/Reset/UART 正文仍不可读] → 保持官网 `partially-observed`，只使用直接打开的官方 GitHub 页面或源码做交叉验证；无法打开的候选 URL 不登记、不引用。
- [官方分支在 Act 前变化] → Act 在写正文前重开直接来源；字段变化时停止相关任务并交回 Plan，不沿用本设计中的具体数值。
- [FIFO 256 与 64 冲突] → 分别记录官方 K3 DTS/8250 配置和 tgoskits 实现，不得裁决硬件深度。
- [目标 Kit DTS 仍不唯一] → 共享 `k3_com260.dtsi` 只证明共同配置；变体专有 UART 不写成 Kit 默认值。
- [第三方注释混有原理图和上板实测] → 只有实际代码行为可作为第三方经验；注释中的接线、clock 实值和 pad 争用保持待核对线索。
- [两篇正文接近 450 行] → 按 AP/RCPU 或 provider 子域拆分并保留入口，不删减范围边界来伪造精简。
- [来源覆盖行数变化] → 以 Act 实际新增 URL 数重新计算总数和唯一数，不把设计中的候选数量写成完成事实。

## Migration Plan

1. Iteration 000 重新打开直接来源，登记实际使用的精确 URL，再创建平台控制文档。
2. 验证 provider/consumer、AP/APBC2/RCPU 分域、来源层级和未知项；Review accepted 后形成平台稳定基线。
3. Iteration 001 创建 UART/console 文档，保留 FIFO 等冲突和第三方适用边界。
4. 有限修正 `com260-image-and-dts.md` 的 `serial0` 与 console milestone 归属；只在出现现有 G1-G7 未覆盖的新缺口时修改 `known-gaps.md`。
5. 更新 `docs/index.md`，执行内容、链接、行数、OpenSpec、diff 和全量变更审查。

回退时只撤销本 change 新增的来源行、两篇主题文档、MS03 有限勘误、索引链接和本 change 实际新增的缺口；不回退 MS01-MS03 基线。change accepted 后再由 docs-maintainer 同步全局状态。

## Open Questions

None。来源可访问性、分支变化、目标 DTS 非唯一和 FIFO 冲突都有既定结果分支，不要求 Act 决定需求或证据语义。
