# Rt-Async-AMP 的 K3 共享内存与通知链

> Snapshot: [SNAPSHOT](../docs/SNAPSHOT.md)
> Captured revision: Rt-Async-AMP `ccb1ff0b487e4f49ea570c41f330741eecece935`；tgoskits `19219411d5dc1515496f910d04c93da12ee95be4`；OpenSBI `7a2df083ed06373c506e2e6f4e09bbd168202f2d`
> Observed branch: Rt-Async-AMP `master`；tgoskits `feat/rt-async-amp`；OpenSBI `feat/pma-audio-io`
> Captured at: 2026-09-08
> See also: [启动](rt-async-amp-k3-boot-platform.md)、[驱动](rt-async-amp-k3-drivers.md)、[复用与缺失仓库](rt-async-amp-k3-starryos-reuse.md)

## 结论与范围

最有价值的内容是共享内存生命周期、PMA 修改的位置、APLIC/IMSIC 门铃路由和数据/通知分离机制。补齐的 AP 侧代码证明 `reserved-memory → ioremap → SharedMemory::at → ioctl/mmap/poll` 的闭环已实现，也暴露单等待者和 mailbox 工作预算不足的边界。这里研究的是 AP↔RT24 SRAM 通道，不能据此认定所有 DDR DMA 或 IOMMU 一致性已解决。

本文回答窗口如何定位、初始化与恢复由谁负责、PMA 如何设置、通知和 RPC 如何传递，以及哪些错误路径阻止直接复用。所有硬件效果只按仓库记录表述；本次没有真板运行证据。

## 地址、布局与初始化状态

[RP DTS:182 起](../../others/Rt-Async-AMP/its/rt-async-k3.dts) 声明 `reg=<0 0 0 0x19000>`，注释把它解释为主域 `0xc0800000` 的 RCPU 本地 SRAM 别名。[AP DTS](../../others/Rt-Async-AMP/tgoskits/os/StarryOS/configs/board/spacemit-k3-com260-ifx.dts) 把 `0xc0800000..0xc0819000` 声明为 `no-map` reserved-memory，并在 `ov,rt-async-amp` 节点重复同一 `reg`。两边配置是自洽的，但“0 和 `0xc0800000` 确实映射同一 SRAM”仍需真板双向读写证明。

| 对象 | 当前可读实现 / 声明 | 核对边界 |
| --- | --- | --- |
| 窗口大小 | RP/AP DTS 与 `rtshm-abi::K3_SHM_SIZE` 都为 `0x19000` | AP 会以整窗 ioremap，用户 mmap 也暴露整窗；需另做访问权限限制 |
| 三通道 | CH0 请求、CH1 响应、CH2 urgent；`RpcClient` 与 `RpcServer` 同值 | Ring 实现在 ov-channels 子模块中，尚缺 |
| 布局估算 | DTS 注释：`0x100 + 3 × 0x8200 = 0x18700`，尾部留 `0x800` scratch，正好到 `0x18f00` | 不能替代缺失类型的 size/alignment 检查 |
| 基址 0 | `ShmDriver` 用 `usize::MAX` 作为未 probe 哨兵，允许地址 0 | 必须补查 `SharedMemory::at(0)` 的实现；若构造普通 Rust 零地址引用，会有语言层有效性问题 |
| probe 失败 | 缺 `reg` 或 size 时 panic；`base()` probe 前 panic | `size()` 实际返回初值 0，没有像注释所称那样主动 panic |

关键接口：[shm.rs:36/60/81](../../others/Rt-Async-AMP/modules/ov-shm/src/shm.rs) 的 `ShmDriver::probe`、`base`、`flush`；[client.rs:41/79](../../others/Rt-Async-AMP/modules/ov-rpc/src/client.rs) 的 channel 常量和 `RpcClient`；[server.rs:288](../../others/Rt-Async-AMP/modules/ov-rpc/src/server.rs) 的 `RpcServer`。

源码中的启动状态关系：

```text
RP Board probe 只登记窗口
  ├─ wait_ready：等待 3s → 轮询 is_valid → 发布 SHM_BASE
  │                                  └─ 超过 fallback_ms → 本地 init
  └─ magic_watchdog：等待 3s → 每 1s 检查 → invalid 时 init，最多 8 次
AP 路径：SPL/U-Boot 完成 → rt_shm probe
  → ioremap 窗口 → valid 则保留，invalid 则 init
  → 复位 mailbox 残留 FIFO/pending → 使能 IRQ → 自测
```

[intercom.rs:1631–1682](../../others/Rt-Async-AMP/apps/rt-async-k3/src/intercom.rs) 的 `init` 在 `shm.init()` 后发布 `SHM_BASE`；`wait_ready(10, 10_000)` 见 [shm_ping.rs:55](../../others/Rt-Async-AMP/apps/rt-async-k3/src/bin/shm_ping.rs)。[watchdog.rs:31](../../others/Rt-Async-AMP/apps/rt-async-k3/src/watchdog.rs) 在自身启动约 4s 后首次检查 invalid 并可初始化。因此，当前组合不是“纯粹等 AP 初始化，十秒后才允许 RP 写”；看门狗可以更早写入。重初始化清除未读消息也是源码注释明确承认的代价。移植时需要单一初始化所有者和恢复期间的通信停顿/错误契约，不能把 magic 自愈等同于无损恢复。

AP [`RtShmDevice::new`](../../others/Rt-Async-AMP/tgoskits/os/StarryOS/kernel/src/pseudofs/dev/rt_shm.rs) 把 init 职责放在 bootloader 之后：窗口已有有效 magic 就不清环，否则初始化。SPL 使用 SRAM、U-Boot `k3_clear_sram()` 全清、晚期脏行写回的时间点仍只来自仓库注释；U-Boot 源码和原始故障日志缺失。可复用的是“在最后一个会破坏窗口的阶段之后初始化”这个原则，不是固定 3s 或某个特定 init 所有者。

AP 驱动还实现了 `/dev/rt_shm` ioctl：`NOTIFY` 发门铃，`AWAIT` 在锁内二次检查 ring 后注册 waker，`TEST_MBOX` 在 boot 时注入本地 new_msg 验证 mailbox→APLIC→IMSIC→handler，`RD_KTS` 读时戳。`poll()` 以 CH1 ring 状态作为就绪真值，门铃只用来唤醒；这一点值得直接保留。

SPL 使用 SRAM、U-Boot `k3_clear_sram()` 全清、晚期脏行回写的时间点来自仓库注释。U-Boot 源码和原始故障日志缺失；固定 3s 不是我们板上可直接采用的安全证明。

## PMA 补丁：能够直接核对的实现

OpenSBI 实际位于 `opensbi-k3/opensbi-spacemit/`，HEAD 与主仓 gitlink 要求相同，工作树干净。路径错位影响构建，不影响本次直接阅读。

调用链为 [sbi_init.c:348/449](../../others/Rt-Async-AMP/opensbi-k3/opensbi-spacemit/lib/sbi/sbi_init.c) 的 cold/warm 初始化 → `sbi_platform_final_init` → [platform.c:272](../../others/Rt-Async-AMP/opensbi-k3/opensbi-spacemit/platform/generic/platform.c) 的 `generic_final_init` → [spacemit_k3.c:365](../../others/Rt-Async-AMP/opensbi-k3/opensbi-spacemit/platform/generic/spacemit/spacemit_k3.c) 的 `spacemit_k3_final_init`。K3 匹配表在同文件第 111 行。

`k3_pma_set_amp_window_io` 的可读行为：

1. 读取 16 个 PMAADDR，左移 2，将相邻值解释为上下界；寻找覆盖 `[0xc0800000, 0xc0880000)` 的 entry。
2. 找不到则打印信息（仅 announce 时）并返回，继续启动，不强制失败。
3. 先 `csi_dcache_clean_range` 清理该窗口，再修改 PMACFG0 或 PMACFG2 中相应 byte 为 `0x22`，最后 `sfence.vma`。
4. cold 和 warm 路径都调用该函数，只有 cold boot 输出诊断。

这比固定写某个 byte 更有参考价值。但它依据源码对 PMA 表的解释，修改的是整个覆盖 entry，可能比应用的 `0x19000` 窗口更大；找不到 entry 仍继续启动。我们的复用验证需要实际读回 entry 边界、权限/属性，以及双向读写结果，不能只看“打印了 banner”。

“X100 忽略 Svpbmt”“本板 entry4 才覆盖 audio SRAM”是作者记录的板上结论，本次仅确认补丁确实为此实现。AP 内核实现也已撤除 CBO，对内核 ioremap 和用户 mmap 都依赖 OpenSBI PMA 把物理窗口设成 IO。但 `flags()` 的注释仍称“四个 CBO 同步点保证一致性”，与模块顶部“已全部撤除”冲突，应按实际调用路径和真板读写判定，不按注释选边。

`shm::flush()` 实际只发 `fence iorw,iorw`。它不是 cache clean API，不能在未来 DDR cacheable DMA 路径中当作清缓存操作。

## 门铃与数据流

[mailbox.rs](../../others/Rt-Async-AMP/modules/chip-k3-rt24/src/mailbox.rs) 的 `MboxK3` 包含 base、irq、local/remote user 和 `IrqLatch`；DTS 指定物理 mailbox4 `0xcac91000`、RP IRQ69。变量名 `MBX3` 是遗留命名，实际不代表物理 mailbox3。

```text
AP RpcClient 写 CH0 → SeqCst fence → 若 BUSY=0，则 NOTIFY
  → AP 内核 mailbox4 ch0 写 FIFO → RP IRQ69
  → mbox_isr 排 FIFO / 清 pending / 重读 → latch.notify
  → MBX3.recv().await 返回 → process_elastic → RpcServer
  → 写 CH1 → PeerNotifier::notify → Release fence → mailbox ch1
  → AP mailbox4 ch1 / APLIC source217 / IMSIC EID
  → handler 排 FIFO 并唤醒单槽 waker → AWAIT/poll 重查 CH1
```

接收端 local user=1，远端 AP user=0；RP 接收硬件 channel0、发送 channel1，`rcpu-communicate` 属性存在时会翻转 user 映射。三个共享内存通道和 mailbox 硬件通道是两层编号，urgent CH2 也需要通知策略，不能简单逐号对应。

`setup_interrupts` 等 PLIC probe 完成后注册 ISR，设 priority=1、enable IRQ、开本地 channel0；`mbox_isr` 读取 raw & enabled，先读 FIFO，再检查剩余数，再清 pending，直到没有 pending，最后唤醒 latch。外层 claim/complete 和 latch 防竞态由缺失 rt-async 提供，当前只确认调用点。

这里值得复用“通知只提示重新检查队列”和“使能中断前先排空残留 FIFO/降低电平”的原则。AP `ack_and_clear` 已有 32 轮×64 次上限，不再是真正无界 ISR，但最坏仍可做 2048 次 MMIO 排空并且不报告截断；RP ISR 仍需补工作预算。两侧 `signal` 都没有 FIFO 满处理和返回错误。mailbox3 被 ESOS 占用的叙述也需按当前 rcpu0 占位固件重新核对。

## 忙等、错误与完成

[intercom::process_elastic:1754](../../others/Rt-Async-AMP/apps/rt-async-k3/src/intercom.rs) 设置 BUSY，处理 urgent 再处理普通请求，无消息后自旋最多 100000 次；新流量可让这一调用继续运行。退出前 clear BUSY → SeqCst fence → 再查队列。客户端 [call_inner:136](../../others/Rt-Async-AMP/modules/ov-rpc/src/client.rs) 是写请求 → SeqCst fence → 查 BUSY。可以提取睡眠竞态的两端关系；fence 是否完整配合，需要补查 ov-channels 内部原子和 AP 通知路径。

它不是无忙等的通用 async 驱动。源码注释保存约 2s 的历史窗口估值；别名和原子优化之后的当前时长未在本次测量。urgent 也只是每轮先排空，不会在每个普通请求之间重新检查，不能据此承诺急停延迟上限。

RPC 提供 `Reply::Deferred`、响应发送失败计数、多块大响应与反序列化错误测试，值得作为 API 设计参考。另有两处需实测：

- [server.rs:321](../../others/Rt-Async-AMP/modules/ov-rpc/src/server.rs) 的错误分支写 poison 后返回 `Unhandled`；[intercom::step:1865](../../others/Rt-Async-AMP/apps/rt-async-k3/src/intercom.rs) 仅对 `Handled(Notify)` 发门铃。推断：AP 若已阻塞，错误响应可能已入环却不及时唤醒。[error_paths.rs:85](../../others/Rt-Async-AMP/modules/ov-rpc/tests/error_paths.rs) 直接 `poll_responses`，没有覆盖这个通知缺口。
- [rtsh::wait_raw:307](../../others/Rt-Async-AMP/user-apps/rtsh/src/lib.rs) 每轮先 ppoll 再查客户端缓冲。需补测多个在途响应已被一次 poll 收入本地缓冲时，后续 rid 等待是否发生无谓阻塞。AP `poll` 已确认以 CH1 `has_pending()` 判定 `IN`，但不会知道用户态已缓存的 rid。
- AP 只有一个全局 `IPC_WAKER`。代码明示假设“每进程一个 IPC 等待线程”；多线程 AWAIT，或 epoll 与 AWAIT 并存时，后注册者会覆盖前者。这不能直接复用成通用多客户端 ABI。

## 验证入口与未确认项

可读测试入口包括 `user-test-ipc`（通知回显与 ADD）、`user-test-pbmt` + `pbmt_probe`（双向写读）、`user-test-bench`（D1 IRQ / D2 spin / D4 竞争分组），以及 ov-rpc 的 host tests。均未实际运行：Rt-Async-AMP workspace 仍缺 rt-async/ov-channels，且无真板；tgoskits workspace 只完成了 offline metadata 解析。

PBMT 探针有跨核写回、活性和读计数前进等有价值的行为检查；其 PID nonce、固定 CPU2 和历史时间证明机制不作为本项目复用要求。未来验证直接证明目标读写、错误和唤醒行为即可。

未确认项：PMA 在我们固件中的每 hart 实效、用户 mmap 权限与实际非缓存属性、对端 reset 时恢复、ring 多块原子发布与容量、0 地址在 Rust API 中的有效性、IrqLatch 注册后重检、AP AIA 在 SMP 下的 affinity/路由。所需子仓见[补充清单](rt-async-amp-k3-starryos-reuse.md#需要补充的仓库和材料)。
