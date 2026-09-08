# Rt-Async-AMP 的 K3 驱动与 StarryOS 异步边界

> Snapshot: [SNAPSHOT](../docs/SNAPSHOT.md)
> Captured revision: Rt-Async-AMP `ccb1ff0b487e4f49ea570c41f330741eecece935`；tgoskits `19219411d5dc1515496f910d04c93da12ee95be4`；StarryOS `6fcc602de48217a125d40f1f34635138464a2b9e`
> Observed branch: Rt-Async-AMP `master`；tgoskits `feat/rt-async-amp`；StarryOS `linshi`
> Captured at: 2026-09-08
> See also: [启动](rt-async-amp-k3-boot-platform.md)、[共享内存](rt-async-amp-k3-shared-memory.md)、[复用清单](rt-async-amp-k3-starryos-reuse.md)

## 结论与范围

可读板级代码提供两套范围：RP 侧 PXA UART、clock/reset、PLIC/SysTimer 和 mailbox，以及 AP 侧 APLIC/IMSIC、PXA UART、pinctrl、GMAC 和 UFS。AP GMAC 已有 DMA ring、cache 维护、IRQ 唤醒和队列接口，是对 StarryOS 价值最高的现成驱动参考；UFS 虽能建立同步 block device，仍以轮询完成，需要异步化后才适合我们的长期驱动约束。

本文回答哪些驱动实际接入、寄存器与初始化顺序如何表达、哪些 async 语义缺失，以及哪些内容与 MS04–MS07 有关。只把实际代码行为称为源码确认；注释引用的手册、原理图和上板测量均保留为待独立核对的来源线索。

## 实际接入与域边界

从 [K3_DRIVERS，lib.rs:55](../../others/Rt-Async-AMP/modules/chip-k3-rt24/src/lib.rs) 和 [rt-async-k3.dts](../../others/Rt-Async-AMP/its/rt-async-k3.dts) 可核对：pinctrl、CCU、mailbox、SHM、PXA UART、SysTimer/MSIP、PLIC 和 AP UART5 被注册。`soft_uart` 模块仍在源码树，但不在驱动注册表，DTS 也没有其设备节点。

tgoskits 的 AP board profile 显式开启 K3 PXA UART、UFS、GMAC、pinctrl 和 Zicbom。AP DTS 启用 UART0 `0xd4017000` / IRQ42、GMAC1 `0xcac82000` / APLIC source133、UFS `0xc0e00000` / source135；中断根为 IMSIC `0xe0400000` 和 APLIC `0xe0804000`。这些数值只是该 DTS 的配置证据。

| 资源 | 当前源码配置 | 可复用条件 |
| --- | --- | --- |
| R_UART0 | `0xc0881000`；u32 MMIO，stride=4；GPIO122/123 mode4 | 只代表 RCPU console 实例，不能直接用作 AP early console |
| AP UART5 | `0xd4017400`；TX-only polling；GPIO83 mode4 | RP 借用 AP 外设的方案，必须协调 AP 的 pinctrl/clock 使用者 |
| pinctrl | `0xd401e000`，写入 base+offset | 解析的是 `pinctrl-single,pins` 最小子集 |
| RCPU CCU | 上游 gate `0xc088003c`；UARTCTRL `0xc0881f00` | 固定频源消费者；不能替代 AP 全局 clock provider |
| RT24 PLIC | `0xe0000000 + (hart << 27)` | 非标准 per-hart 布局，不能替代 AP APLIC/IMSIC |
| RT24 SysTimer | global mtime `0xe400bff8`；hart1 mtimecmp `0xec004000`、MSIP `0xec000000`；24MHz | RP M 态实现，不证明 AP S 态可直接访问 |
| AON_TIMER1 | `0xc0889000`；代码频率 2MHz | 探针计时器，未替代生产 SysTimer |
| AP AIA | IMSIC `0xe0400000`；APLIC `0xe0804000`；512 wired sources、511 EID | 完整的 FDT probe/domain/MSI 路由参考；SMP affinity 尚不完整 |
| AP GMAC1 | DWMAC 5.10a `0xcac82000`；IRQ133；RGMII | 已有单队列 DMA/IRQ 实现，优先抽取验证 |
| AP UFS | `0xc0e00000`；IRQ135；2 lanes | 完整 bring-up/SCSI 参考，当前数据面同步轮询 |

DTS 注释称 R_UART0 IRQ17，但 UART 节点没有 `interrupts` 属性，PXA driver 也没有注册 RX/TX ISR。这是硬件资料线索与实际运行模式的区别。

## UART：能提取什么，缺少什么

[pxa_uart.rs:84/129](../../others/Rt-Async-AMP/modules/chip-k3-rt24/src/pxa_uart.rs) 的 `PxaUartRegs` 和 `init_port` 明确使用 u32 寄存器、四字节步长：

```text
clock/pinctrl 前置（框架实现缺失）
  → DLAB=1 → DLL/DLH=8/0 → 8N1 → 开 FIFO、清 RX/TX
  → IER.UUE(bit6)=1 → MCR.OUT2(bit3)=1
  → THRE 轮询写 / DR 轮询读
```

UUE/OUT2、DLL 与时钟关系、寄存器宽度适合成为 K3 PXA 硬件适配调查项；不要因“类似 16550”就直接套 QEMU byte-MMIO 配置。固定除数8的合理性仍取决于当前时钟实值。

`write_raw` 逐字节等 THRE，`read_raw` 无数据立即返回 None；`Serial::write` 对 LF 插入 CR。无 TX/RX async ring、waker、FIFO 批处理、IER 所有权锁或 TEMT drain。发送等待没有超时。源码顶部旧表还列 R_UART3/R_UART1 为机器人口，但当前 DTS 只声明 R_UART0 和 AP UART5，不应把三个 slot 池容量解读为三个可用端口。

tgoskits 另有真正接入 AP serial 框架的 [PXA UART](../../others/Rt-Async-AMP/tgoskits/drivers/serial/some-serial/src/pxa_uart/mod.rs)：32-bit MMIO/stride4、64-byte FIFO、PXA divisor errata 顺序、UUE/OUT2、RX timeout 和独立 IRQ endpoint 都已实现。IRQ handler 有 32-pass / 256-sample 预算，处理 sticky LSR 错误、Errata #20 残留 RX 和 THRE 电平触发的 mask/rearm；`UartPort::write_tx` 也保留短写语义。这些比 RP polling 实现更适合复用，但 copier task、多等待者 waker 和 drain deadline 仍应保留我们现有 StarryOS 抽象。

[ap_uart.rs:169/187/245/315](../../others/Rt-Async-AMP/modules/chip-k3-rt24/src/ap_uart.rs) 的 `ddn_rate`、`choose_clock`、`pad83_heal`、`send` 提供另一组可读机制：读取 UART0 parent mux 和 SUCCR/SUCCR_1 分频，计算 115200 除数及误差，再配置 UART5；发送前核对 pad83，发送后等 TEMT，每段轮询最多 200000 次。

这里可复用“读取启动固件已配好的时钟”“区分 THRE 接收能力与 TEMT 物理完成”“配置后实际发波形”的思路，但有三个限制：

- `pad83_heal` 强制把被 AP 改成 I2C 的 pad 改回 UART，只是原系统的修补策略。我们应先建立引脚所有者，不能让两个内核反复抢写。
- `choose_clock` 最后会写全局 SUCCR；代码注释“死源无消费者”不足以证明未来 AP 没有其他使用者。
- `send` 超时一律返回0，即使前面已写入部分数据；这个接口不能直接满足 StarryOS 的短写/已接收字节数契约。轮询次数也不是稳定的墙钟 deadline。

40pin 的 GPIO122/123、pad83、SEC UART1、电平转换和 GPIO88 关机风险来自 DTS/AP UART 注释中的 kit_v02 原理图解读。原理图未随本次材料提供，这些只能作布线核对清单，不能成为实际接线指令。

## 时钟与 pinctrl：最小实现的局限

[clock.rs:144/162/186](../../others/Rt-Async-AMP/modules/chip-k3-rt24/src/clock.rs) 的 `CLK_TABLE`、`enable_periph` 和 `Ccu::probe` 将 RUART/RI2C/RSSP ID 映射到末端寄存器。probe 保留上游 gate 的其他位；末端配置则整寄存器写 APBCLK/FNCLK/RST/mux，未保留所有原值。未知 clock id 仅警告后继续。表中出现 RI2C/SSP 不代表已有 I2C/SPI 数据面驱动。

[pinctrl_k3.rs:65](../../others/Rt-Async-AMP/modules/chip-k3-rt24/src/pinctrl_k3.rs) 的 `apply` 只解析首个 `pinctrl-0` phandle，按 offset/value 两两写入；悬空引用或缺 pins 时警告后返回，奇数尾 cell 不报错，也不检查 offset 对 reg.size 的越界。作为可信静态 DTS 的原型可参考，作为通用驱动需补输入和失败传播契约。

AP [K3 pinctrl](../../others/Rt-Async-AMP/tgoskits/drivers/ax-driver/src/soc/k3/pinctrl/mod.rs) 更完整：解析 `(pin_id<<16)|mux`，支持 bias、strong pull、Schmitt、drive-strength 和 1.8V/3.3V power-source，并在每次 IO 电源域写入前执行 APBC 一次性解锁。它通过 pinctrl provider 让 consumer probe 前自动应用 `pinctrl-0`，这种 provider/consumer 分层比 RP 的直接写表更值得复用。限制是只支持 pin 0..144，drive-strength 缺 power-source 时会警告并默认 3.3V，这个降级策略应在我们的产品级设计中重新决定。

初始化顺序的实际可见部分是 `K3_DRIVERS` 把 controller 放在消费者前面。逐节点先 pinctrl、再 clock、最后 probe 是板级文件描述的平台约定；因为 rt-async 缺失，本次未独立读到 `platform::driver::boot` 的实现。

## 中断、计时和原子操作

[plic_k3.rs:29/110/149](../../others/Rt-Async-AMP/modules/chip-k3-rt24/src/plic_k3.rs) 用 FDT boot hart 算 `WIN`，priority/enable/threshold/claim 都加该偏移。`enable_irq`/`disable_irq` 是未加锁的寄存器读改写；当前单 hart 框架的串行假设不能直接扩展到 AP SMP。相同 `riscv,plic0` compatible 也不意味着 SiFive 寄存器布局兼容。

[clint_k3.rs:80](../../others/Rt-Async-AMP/modules/chip-k3-rt24/src/clint_k3.rs) 的 Timer 直接读固定 global mtime，`set_deadline` 分两次写高、低32位，`MsIP::send/clear` 在窗口未初始化时静默返回。mtimecmp 更新的瞬态语义、hart 编号和设备树 reg 范围必须按实际硬件复核；不从该实现推出所有 RV64 timer 都适用的写序。

[timer_k3.rs:70](../../others/Rt-Async-AMP/modules/chip-k3-rt24/src/timer_k3.rs) 把 clock/reset 低三位置1，最多8次写回读确认 CER。此处 reset bit0 是低有效，而 CCU UART/APBC 的 reset bit2 是高有效，不能统一为“写0总是释放”。原作者记录 AON 微基准很快，但生产替换没有收益并回滚；当前生产 deadline 仍走 SysTimer。性能数字保留为历史线索，不作为本板结论。

[自定义 target](../../others/Rt-Async-AMP/targets/riscv64imac-k3-none-elf.json) 设置 `atomic-cas=false`；[app Cargo.toml:31](../../others/Rt-Async-AMP/apps/rt-async-k3/Cargo.toml) 通过 `cs-atomics` 启用 portable-atomic 的 critical-section 回退。它依赖单 hart M 态、本地共享状态的前提。跨 AP/RP 原子必须另行审查；不能把关本核中断当成跨核互斥，也不能按性能注释批量将 Acquire 改 Relaxed。

`K3Rt24::init` 注释记录 rcpu1 访问 custom CSR `0x7ca/0xbc0` 会挂死，而 rcpu0 行为不同。当前代码没有执行这些访问；它们应作为“别照抄其他核 CSR 配置”的待核实警示，而不是在新板上盲试的测试清单。

### AP APLIC/IMSIC

[imsic_aplic.rs](../../others/Rt-Async-AMP/tgoskits/platforms/somehal/src/arch/riscv64/imsic_aplic.rs) 已实现 FDT probe、IMSIC 根域、APLIC 子域、MSI-X 子域、统一 EID bitmap 以及 `wired source → EID → leaf IRQ` 路由。APLIC 在 MSI mode 下根据 FDT trigger 配置 source，然后将目标设为当前 hart。外部中断 claim/complete 用单条 `csrrw stopei`，避免 handler 期间新到的同 EID MSI 被错清；这是值得保留的关键语义。

仍有 SMP 边界：APLIC source 首次 enable 时绑定当前 hart，disable 不释放 EID，wired-source 没有 affinity 迁移，MSI-X `Fixed` affinity 明确返回 unsupported。`secondary_init_intc` 只初始化每 hart IMSIC file 和 `sie`。所以这套实现可作为我们 AP AIA 第一版的结构参考，不能直接宣称 SMP IRQ balancing 已完成。

## GMAC 和 UFS：复用价值不同

[K3 GMAC](../../others/Rt-Async-AMP/tgoskits/drivers/ax-driver/src/net/k3_gmac/mod.rs) 是 Synopsys DWMAC 5.10a 单队列驱动。它包含 APMU/syscon 的 RGMII 模式与 delay-line phase、MDIO/PHY 自协商、MAC/MTL/DMA 初始化、描述符 ring、TX/RX buffer 所有权和 IRQ handler。DTS 会提供 `tx-phase=0x2f`、`rx-phase=0x35`、`max-speed=1000` 和 PHY/reset GPIO。它对 `/soc dma-noncoherent` 有明确处理：64-byte 对齐描述符，CPU 交权前 clean，读 DMA 回写前 invalidate，同时使用 DMA fence 保证 doorbell 顺序。

GMAC 的异步边界也比较清晰：IRQ 上半部只生成 `Event`供 `net_poll_worker` 唤醒，数据面通过 `ITxQueue`/`IRxQueue` submit/reclaim；IRQ 上下文抢不到共享 `SpinNoIrq` 就返回 empty event。这个 try-lock 策略避免硬中断自死锁，但“等下一个 IRQ 再试”依赖后续中断一定会到；我们复用时应保留屏蔽→轮询→重装→重查契约，不把 empty event 当作充分防丢唤醒证明。初始化里约 100ms 和 PHY 协商里的时延是 CPU-frequency-dependent `spin_loop`，也应换成真正 deadline。

[K3 UFS](../../others/Rt-Async-AMP/tgoskits/drivers/ax-driver/src/block/k3_ufs/mod.rs) 覆盖 MPHY/UniPro/link startup、HS 升级、UTRD/UTMRD/UCD/PRDT、SCSI LUN 扫描、读写与超时恢复，对 DMA buffer 也有 flush/invalidate 和 `fence ow,ow` / `fence ir,ir`。它虽解析 IRQ135，但明确不注册 handler，所有 UIC 和 transfer completion 都轮询，并通过同步 block adapter 串行化访问。因此初始序列、描述符和恢复路径有高参考价值，运行时完成模型则不应直接搬进 StarryOS。

## 与我们当前 StarryOS 的接口对应

本次补读的相邻 `../StarryOS` 是 `linshi` 分支，区别于第三方 tgoskits 的 StarryOS。其 [platform/mod.rs:18/32](../../../StarryOS/kernel/src/platform/mod.rs) 导出 QEMU、D1、VisionFive2，descriptor 实际选择 D1 或 QEMU，没有 K3 分支。[descriptor.rs:25/69](../../../StarryOS/kernel/src/platform/descriptor.rs) 的 `InterruptConfig` 只有 PLIC base；需要 AP AIA 来源，RT24 PLIC 不能填补它。

| 现有接口 | 本次资料可提供的输入 | 保留的 StarryOS 能力 |
| --- | --- | --- |
| `UartPort` | PXA u32 寄存器、UUE/OUT2、clock/pinctrl 前置、TEMT | RX/TX copier、IER 同步、waker、短写与 drain |
| `PlatformDescriptor` | AP FIT/DTS、console、reserved-memory 与 AIA 结构 | 平台常量集中管理；由官方资料确认数值 |
| `NetQueueControl::arm_notify_and_check` | GMAC IRQ/Event 与 submit/reclaim 数据面 | 原有通知屏蔽/重装/重查契约，补上 try-lock 失败后的确定性推进 |
| `NetTxQueue` / recovery | DWMAC ring ownership、Zicbom 维护、DMA reset | 已有 TX 完成和恢复语义；不回退成纯轮询 |
| block async layer | UFS init、descriptor、SCSI 和 controller recovery | 用 IRQ/completion task 替代 transfer 忙等，保留 deadline/cancel 语义 |

接口实现在 [uart_init.rs:108](../../../StarryOS/kernel/src/drivers/uart_init.rs) 和 [axdriver_net/lib.rs:178/228/267](../../../StarryOS/crates/axdriver_net/src/lib.rs)。这是本次源码读取，不依赖旧分析中的 StarryOS revision。

## 验证入口与覆盖缺口

本次对 RP DTS 只完成 C 预处理，对 tgoskits 只完成 offline Cargo metadata，未跑 DTB 编译、K3 target build、IRQ、UART loopback、GMAC/UFS 数据面或真板负载。`sched_demo` 包含 timer/优先级演示及 mailbox 接收任务，但 rt-async 执行器、trap 分发和 IrqLatch 实现仍缺失，不能声明调度性能或防丢唤醒通过。

未来借鉴验证时按 polling 首字节 → 寄存器配置读回 → 重复 IRQ claim/handler/clear/complete → RX/TX ring 与唤醒 → TEMT drain → SMP 并发推进。GMAC 建议再加 PHY link up/down、ring wrap、小包风暴、cache-line ownership、IRQ 丢唤醒和 reset 后恢复；UFS 则要分开 init polling 与运行时 IRQ completion，测 timeout/fatal recovery 和并发 I/O。该仓库已显著降低 MS06/MS07 的实现调查成本，但未提供 IOMMU driver，也不替代我们板上的验证证据。

`soft_uart` 的复杂定拍/中断屏蔽方案已经退出当前板级调用链，优先保存其“跨域访问成本可能超过 bit 时间”的失败经验，不把遗留实现列入首批移植。
