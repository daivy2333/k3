> 来源: https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/boot.md（源端修订: unknown；观察日期: 2026-09-02）；https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/boot.md（源端修订: unknown；观察日期: 2026-09-07）；https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-07）

# K3 AMP 消息路径：Ring、Doorbell、RPC、等待与错误恢复

> 范围: AP↔RP 的消息路径。覆盖三层编号（共享内存 ring / mailbox 硬件 channel / processor）、BUSY 与发布 fence、doorbell 与 IRQ/waker、Deferred/error/poison 路径、超时与取消、多等待者与通知合并/丢失、reset 恢复与未决消息。
>
> 来源边界: SpacemiT 官网和官方 GitHub 只支持 K3 启动阶段与 reserved-memory 的一般边界。AMP 数值、状态转换与具体 ring/RPC 行为来自 R10/R11 固定 revision 第三方调查：Rt-Async-AMP `ccb1ff0b487e4f49ea570c41f330741eecece935`、tgoskits `19219411d5dc1515496f910d04c93da12ee95be4`、OpenSBI `7a2df083ed06373c506e2e6f4e09bbd168202f2d`。第三方代码、注释和历史测量不构成 K3 官方规范或本项目真板结果。
>
> 证据等级: 使用 `官方事实`、`交叉验证`、`推论`、`未知项`。直接读取固定 revision 第三方代码所得行为写为"第三方实现 / 未知项"；跨文件组合的路径写为"第三方实现 / 推论"。
>
> 与本文相邻的镜像装载、共享窗口地址、PMA 与生命周期状态见 [`k3-amp-shared-memory-lifecycle.md`](k3-amp-shared-memory-lifecycle.md)；mailbox 寄存器和静态通知配置见 [`../interrupts/com260-mailbox-notification.md`](../interrupts/com260-mailbox-notification.md)；cache、fence、PMA/PBMT 与 IOMMU 边界见 [`../dma/k3-cache-pma-address-translation.md`](../dma/k3-cache-pma-address-translation.md)；GMAC descriptor/IRQ 完整数据面见 [`../network/k3-gmac-dma-irq.md`](../network/k3-gmac-dma-irq.md)。

## 目录

- [1. 文档职责与既有基线](#1-文档职责与既有基线)
- [2. 三层编号空间](#2-三层编号空间)
- [3. 请求 / 响应 / urgent ring 行为](#3-请求--响应--urgent-ring-行为)
- [4. BUSY 与发布 fence](#4-busy-与发布-fence)
- [5. Doorbell 通知与 IRQ/waker](#5-doorbell-通知与-irqwaker)
- [6. 错误、poison 与 Deferred](#6-错误poison-与-deferred)
- [7. 超时、取消与多等待者](#7-超时取消与多等待者)
- [8. 通知合并、丢失与重复](#8-通知合并丢失与重复)
- [9. 复位恢复与未决消息](#9-复位恢复与未决消息)
- [10. 未知项](#10-未知项)
- [11. 修订与验证边界](#11-修订与验证边界)
- [12. 与相邻主题的关系](#12-与相邻主题的关系)

## 1. 文档职责与既有基线

- K3 官方 SDK 的 local boot 阶段为 Boot ROM → FSBL/SPL → ESOS → OpenSBI → U-Boot → payload/OS；介质、版本和刷写边界见 [`com260-boot-chain.md`](../boot/com260-boot-chain.md)。`交叉验证`
- K3 官方 `k3.dtsi` 定义两 cell 地址空间和 `reserved-memory` 容器；AP↔RP 共享窗口地址、装载地址和初始化路径见 [`k3-amp-shared-memory-lifecycle.md`](k3-amp-shared-memory-lifecycle.md)。`交叉验证` + `未知项`
- mailbox 寄存器和静态通知配置（mailbox4 base、AP source 217、RP source 69、APLIC/IMSIC 静态路由）见 [`../interrupts/com260-mailbox-notification.md`](../interrupts/com260-mailbox-notification.md)；source→EID、target hart、affinity 与真板 delivery 仍未解除。`未知项`
- cache、fence、PMA/PBMT 与 IOMMU 机制边界见 [`../dma/k3-cache-pma-address-translation.md`](../dma/k3-cache-pma-address-translation.md)。本文不把共享内存的 cache 与 barrier 假设外推到 DDR 或设备 DMA。`推论`
- 共享内存不是设备 DMA。对象 ownership、通知与数据可见性的共同边界见 [`../dma/k3-dma-and-memory-ownership.md`](../dma/k3-dma-and-memory-ownership.md)。`推论`
- 本文只描述指定第三方 revision 的控制流与可读 API 行为，不规定新实现必须采用相同所有者、超时、自旋次数或错误恢复策略。`未知项`

## 2. 三层编号空间

AP↔RP 消息路径由三层互不相同的编号组成。混淆这三层是常见错误来源，本文必须先把它们分开：

| 层 | 编号对象 | 第三方实现当前值 | 边界 |
| --- | --- | --- | --- |
| 共享内存 ring 通道 | CH0 请求 / CH1 响应 / CH2 urgent | 三个 ring 在 `0x100` 头 + `3 × 0x8200` + `0x800` 尾部 scratch 之内 | ring size/alignment/原子序由 `ov-channels` 子模块决定；该子仓在固定 revision checkout 中缺失，本文不能确认其精确结构（`未知项`） |
| mailbox 硬件 channel | AP mailbox4（base `0xcac91000`）；RP `MboxK3` 物理 mailbox4 + `rcpu-communicate` 属性 | 变量名 `MBX3` 是遗留命名，不代表物理 mailbox3 | mailbox4 的 ch0/ch1 是 AP↔RP 通知通道；channel 编号与 ring 编号无任何直接对应（`第三方实现 / 未知项`） |
| processor | AP（`0xc0800000/0x19000` 视角） / RP（base 0 + `0x19000` 视角） / 当前 hart | rcpu1 是该 AP 对端；rcpu0 占位 WFI；AP 视角与 RP 视角的物理 SRAM 是否双向一致仍待真板验证 | processor 身份与 ring 角色绑定（写端 / 读端 / 等待者），但与 mailbox 物理 channel 解耦（`第三方实现 / 推论`） |

### 2.1 共享 ring 与 mailbox 通道的关系

- ring 通道承载"数据 / 状态 / 请求"；mailbox 通道只承载"通知"。mailbox 通知既不能等同于 ring 数据，也不能替代 ring 状态轮询。（`第三方实现 / 未知项`）
- 三个共享 ring 与 mailbox 硬件 ch0/ch1 是两层编号，urgent CH2 也需要独立通知策略；不能简单按 0/1/2 ↔ 0/1/2 把两层做数字对应。（`推论`，依据：URGENT 描述与 mailbox 双通道的实现不对称）
- 当前 `RpcClient` 与 `RpcServer` 用同值枚举区分角色，并不按"哪一侧只写 / 哪一侧只读"做硬绑定；这一点决定本节"写端 / 读端"语义仅指控制流方向，不指物理所有权。（`第三方实现 / 未知项`）

### 2.2 ring 布局的估算边界

- 第三方 DTS 注释估算 `0x100 + 3 × 0x8200 = 0x18700`，尾部留 `0x800` scratch，正好到 `0x18f00`；这与 `rtshm-abi::K3_SHM_SIZE = 0x19000` 总体自洽，但注释是布局意图的说明，不是 `ov-channels` 类型 size/alignment 的证明。（`未知项`）
- `SharedMemory::at(0)` 在 `RP` 视角使用 `0` 作为基址，配合 `ShmDriver` 把 `usize::MAX` 作为未 probe 哨兵；这与 AP 视角的 `0xc0800000` 并不天然相同。地址 0 还是 AP 物理地址取决于访问侧，不得在两侧互换。（`推论`）

## 3. 请求 / 响应 / urgent ring 行为

### 3.1 写端（请求）

- `RpcClient::call_inner` 写请求 → `SeqCst` fence → 查 BUSY；BUSY=0 时执行 NOTIFY 发门铃。完整通知链：AP mailbox4 ch0 写 FIFO → RP IRQ69 → `mbox_isr` 排 FIFO / 清 pending / 重读 → `IrqLatch::notify` → `MBX3.recv().await` 返回 → `process_elastic` → `RpcServer`。（`第三方实现 / 未知项`）
- 写端写 ring 后必须发门铃以让对端从 poll/sleep 状态被唤醒；不发门铃或门铃被对端屏蔽时，对端只能依赖 poll 周期或下次中断。（`推论`）

### 3.2 读端（处理）

- `intercom::process_elastic` 在 ring 入口设置 BUSY，先处理 urgent 再处理普通请求；无消息后自旋最多 100000 次，新流量可让这一调用继续运行；退出前 clear BUSY → `SeqCst` fence → 再查队列。（`第三方实现 / 未知项`）
- `process_elastic` 不是无忙等的通用 async 驱动；其自旋次数与历史 2s 窗口估值为作者记录，本项目未独立测量。urgent 也不在每个普通请求之间重新检查，急停延迟上限不能由当前实现承诺。（`推论`）

### 3.3 响应（CH1）

- `RpcServer` 处理结束后写 CH1 → `PeerNotifier::notify` → Release fence → mailbox ch1 → AP mailbox4 ch1 / APLIC source217 / IMSIC EID → AP handler 排 FIFO 并唤醒单槽 waker → AWAIT / poll 重查 CH1。（`第三方实现 / 未知项`）
- AP `poll()` 以 CH1 `has_pending()` 作为 `IN` 真值；门铃只用于唤醒，不替代 ring 状态判断。这一点对避免"门铃到了但 ring 空"造成的虚假就绪至关重要，必须直接保留。（`推论`）

### 3.4 urgent（CH2）

- urgent 与普通请求共享 BUSY，但优先处理；urgent 没有独立 BUSY 或独立 mailbox 通道。（`第三方实现 / 未知项`）
- 写端需要 urgent 通知时同样只能走门铃；处理器侧在每轮先排空 urgent 再处理普通请求，但当前实现没有"每个普通请求之间重检 urgent"的语义。（`推论`）

## 4. BUSY 与发布 fence

### 4.1 BUSY 角色

- BUSY 是处理器侧弹性轮询状态的提示标志，不是锁或 DMA ownership 标志；写端读取它只为判断是否需要 doorbell，当前可读实现不以它提供互斥或重入保护。（`第三方实现 / 未知项`）
- 写端在写请求后、`SeqCst` fence 后读取 BUSY，处理器侧在进入 `process_elastic` 时设置 BUSY；BUSY=0 时写端发送 NOTIFY，BUSY=1 时跳过门铃并返回 request id，由正在弹性轮询的处理器侧发现新增请求。BUSY 只参与是否发送门铃的判断，不代表对端不在写。（`第三方实现 / 未知项`）

### 4.2 序与 fence

- 写端：写请求 → `SeqCst` fence → 查 BUSY → 仅 BUSY=0 时 NOTIFY。（`第三方实现 / 未知项`）
- 处理器侧：设置 BUSY → 处理 → clear BUSY → `SeqCst` fence → 再查队列。（`第三方实现 / 未知项`）
- `ov-shm::flush()` 实际只发 `fence iorw, iorw`。它不是 cache clean，不能在 DDR cacheable DMA 路径中当作清缓存操作；fence 提供排序，不证明对端立即可见。（`推论`）
- fence 与 cache 的精确关系、AP/RP 内存模型和 Svpbmt 取舍见 [`../dma/k3-cache-pma-address-translation.md`](../dma/k3-cache-pma-address-translation.md)。`推论`

### 4.3 跨处理器顺序保证

- 跨处理器顺序需要：(a) 地址映射指向同一物理存储；(b) 属性允许需要的读写；(c) 发布与读取具备匹配的排序；(d) 启动阶段不再破坏窗口。当前材料不能同时证明这四项在本项目真板成立。（`未知项`）
- BUSY 与 fence 的组合不能补偿 cache miss、跨核 TLB 失效或 fence 指令被优化器重新排序的假设；这些都是顺序保证的"组件"，不是"完整证明"。（`推论`）

## 5. Doorbell 通知与 IRQ/waker

### 5.1 门铃数据流

- AP 写 CH0 后通过 mailbox4 ch0 FIFO 触发 RP IRQ69；RP `mbox_isr` 排 FIFO → 清 pending → 重读 → `latch.notify` → `MBX3.recv().await` 返回。（`第三方实现 / 未知项`）
- RP 写 CH1 后通过 mailbox4 ch1 触发 AP APLIC source217 → IMSIC EID；AP handler 排 FIFO → 唤醒单槽 waker → AWAIT/poll 重查 CH1。（`第三方实现 / 未知项`）

### 5.2 接收端 local user

- 接收端 local user=1，远端 AP user=0；RP 接收硬件 channel0、发送 channel1；当 `rcpu-communicate` 属性存在时 user 映射翻转。（`第三方实现 / 未知项`）
- user 翻转的副作用是把"哪一侧接收"和"哪一侧发送"的实现细节分到 DTS 而不是代码；当前第三方 DTS 中不存在 `rcpu-communicate`，因此默认 user 关系按代码注释理解。（`推论`）

### 5.3 ISR 与 waker 边界

- RP `mbox_isr` 在使能前先排空残留 FIFO/降低电平；这是应当保留的初始化策略。（`推论`）
- AP `ack_and_clear` 已有 32 轮 × 64 次上限，最坏仍可做 2048 次 MMIO 排空且不报告截断；RP ISR 仍需补工作预算。两侧 `signal` 都没有 FIFO 满处理和返回错误。（`第三方实现 / 未知项`）
- 固定 revision 的 `IrqLatch` 行为依赖缺失 `rt-async` 实现；本文只确认调用点，不确认其注册、重检或防丢语义。（`未知项`）

## 6. 错误、poison 与 Deferred

### 6.1 错误响应路径

- `server.rs:321` 的错误分支写 poison 后返回 `Unhandled`；`intercom::step:1865` 仅对 `Handled(Notify)` 发门铃。AP 若已阻塞，错误响应可能已入环却不及时唤醒。（`第三方实现 / 未知项`）
- `error_paths.rs:85` 直接 `poll_responses`，没有覆盖这个通知缺口；因此"错误路径有通知"是行为描述，不是"通知必到"的保证。（`推论`）

### 6.2 Deferred 语义

- `Reply::Deferred` 允许服务器延后发送响应；当前实现保留发送失败计数，但不提供对端的"取消 Deferred"信号。（`第三方实现 / 未知项`）
- Deferred 不构成超时保证；调用方仍需独立 timeout 才能避免无限等待。Deferred 与 reset 的相互作用见 §9。（`推论`）

### 6.3 多块大响应

- 多块大响应的反序列化错误测试存在，但 `error_paths.rs` 没有覆盖"中途失败后释放资源"路径；不能据此声明多块协议对调用者资源安全。（`第三方实现 / 未知项`）

## 7. 超时、取消与多等待者

### 7.1 超时边界

- `rtsh::wait_raw:307` 每轮先 ppoll 再查客户端缓冲；多在途响应已被一次 poll 收入本地缓冲时，后续 rid 等待是否发生无谓阻塞需要补测。AP `poll` 已确认以 CH1 `has_pending()` 判定 `IN`，但不会知道用户态已缓存的 rid。（`第三方实现 / 未知项`）
- timeout 由调用方实现；不同调用方的 timeout 不影响共享 slot 已被对端释放的判断。调用方 timeout 不单独证明共享 slot 已被对端释放。（`推论`）

### 7.2 取消语义

- 当前实现没有提供"取消已发出请求"的语义；客户端只能等待响应、超时后丢弃，或在 reset 后整体回收。（`第三方实现 / 未知项`）
- 取消与 `Deferred` 的相互作用：取消后 Deferred 仍可能被发送并入环；调用方应能在收到后识别为"已取消"。（`推论`）

### 7.3 多等待者与全局 IPC_WAKER

- AP 只有一个全局 `IPC_WAKER`。代码明示假设"每进程一个 IPC 等待线程"；多线程 AWAIT 或 epoll 与 AWAIT 并存时，后注册者会覆盖前者。这不能直接复用为通用多客户端 ABI。（`第三方实现 / 未知项`）
- 复用或重新设计时必须明确：每个进程只能有一个 IPC 等待者，或者重写为多槽 waker / 多 slot / per-rid waker。当前实现的限制需要在文档中显式记录。（`推论`）

## 8. 通知合并、丢失与重复

### 8.1 合并

- mailbox 通知天然合并：FIFO 一次取出后，后续通知若未触发新的 ISR 行为（即无新请求或响应），相当于"合并为 1 次"。（`推论`，依据：FIFO 排空语义）
- ring 状态不由 mailbox 通知维护；处理器侧被唤醒后必须重查 ring 状态而不是依赖通知计数。（`推论`）

### 8.2 丢失

- 使能中断前若 FIFO 残留有数据，可能使能后立即触发一次"延迟的"通知；当前 RP `mbox_isr` 已知会先排空残留。（`第三方实现 / 推论`）
- 通知丢失的可观察后果是"对端在等但本端没醒"；只能通过 ring 状态轮询 + 等待者超时共同防御，不能依赖通知必到。（`推论`）

### 8.3 重复

- 重复通知可能让处理器侧发生多次唤醒或进入无消息的 `process_elastic`，从而浪费一次自旋预算；当前可读 BUSY 操作本身不提供并发进入保护，是否串行还取决于调用和调度约束。（`第三方实现 / 未知项`）
- 同一请求可能被 ring 状态机视为"两次新消息"，但 ring 自身的 head/tail 不会因通知重复而错位；这是 ring 与通知分离带来的固有属性。（`推论`）

## 9. 复位恢复与未决消息

### 9.1 re-init 不会保留未读消息

- 第三方 `intercom::init` 在共享窗口 magic 失效时由 watchdog 调用；其源码注释明确承认"会清空各通道未读消息"。（`第三方实现 / 未知项`）
- "持续外部破坏限制为最多 8 次自愈"是代码注释；该数值不是 K3 硬件保证，也不约束本项目实现必须采用相同上限。（`推论`）

### 9.2 复位后状态缺失

- 对端 reset 可能经过 U-Boot 再次清 SRAM，也可能保留物理窗口但丢失其本地 request、waker 或服务状态。当前协议没有可读证据定义以下结果：
  - 哪一侧先暂停新的发送；
  - 在途请求和 Deferred 响应返回何种错误；
  - 旧 request ID 是否允许跨 epoch 继续匹配；
  - 已发布但未通知或已通知但未读取的消息如何处理；
  - 双端以什么条件确认重新上线。（`未知项`）
- 生命周期上下文（re-init、magic、watchdog、reset）见 [`k3-amp-shared-memory-lifecycle.md`](k3-amp-shared-memory-lifecycle.md) §5–§7。`推论`

### 9.3 re-init 与 Ring 状态

- re-init 写头部并重新发布；原 ring 内容的清空是其副作用，不是协议的"已通知对方丢弃"语义。（`第三方实现 / 未知项`）
- 调用方 timeout 或取消不能单独证明共享 slot 已被对端释放；timeout 之后必须由 re-init 路径或显式 `cancel` 语义把旧请求排除，否则存在"对端仍在持有旧 slot 但本地认为已超时"的可能。（`推论`）

## 10. 未知项

### U1. ring 精确布局、原子序与多块发布

- 当前证据: 第三方 DTS 注释估算 `0x100 + 3 × 0x8200 + 0x800` 与 `K3_SHM_SIZE = 0x19000` 自洽；`ov-channels` 子仓在固定 revision checkout 中缺失。
- 禁止推断: 不由估算布局认定 ring size / alignment / 原子序；不推定多块响应的发布/取消语义。
- 解除条件: 取得 `ov-channels` 实现并完整读取其 `Ring` 类型 size、head/tail 序、Release/Acquire 屏障、并发安全证明与多块 API。
- 影响主题: 消息路径数据面、`SharedMemory::at(0)` 的 Rust 有效性、跨处理器顺序。

### U2. AP 唯一 IPC_WAKER 在多线程下的边界

- 当前证据: 固定 revision AP 只有一个全局 `IPC_WAKER`，代码假设"每进程一个 IPC 等待线程"；`error_paths.rs` 不覆盖多等待者。
- 禁止推断: 不把"单 IPC_WAKER"当成可复用的多客户端 ABI；不假定 epoll 与 AWAIT 并存安全。
- 解除条件: 取得 IPC_WAKER 替换或扩展实现，并在多线程 / epoll + AWAIT 组合下实测覆盖。
- 影响主题: 多客户端 API、epoll 集成、Server 并发模型。

### U3. mailbox FIFO 满、IRQ 屏蔽与截断

- 当前证据: AP `ack_and_clear` 有 32 轮 × 64 次上限但不报告截断；RP `signal` 无 FIFO 满处理；两侧 IRQ 屏蔽策略未在源码中显式说明。
- 禁止推断: 不由"ISR 已清"推定 FIFO 必能容纳新通知；不假定 IRQ 屏蔽时间与吞吐能力。
- 解除条件: 取得 mailbox 寄存器手册和固定 revision 的 IRQ 屏蔽/恢复契约；实测截断行为。
- 影响主题: 通知可靠性、错误传播、tgo.skits 第三方工程内部行为。

### U4. reset 后未决请求的 ID 与状态

- 当前证据: 第三方 `intercom::init` 会清空未读消息；旧 request ID 是否允许跨 epoch 继续匹配没有源码证明。
- 禁止推断: 不把 re-init 写成"恢复全部业务状态"；不假定旧 rid 在新 epoch 仍有效。
- 解除条件: 明确 reset 后 ID 分配、ring 头部版本、等待者取消的契约并实测。
- 影响主题: 复位恢复、协议版本管理、错误传播、生命周期 U4（由 `k3-amp-shared-memory-lifecycle.md` 登记）。

### U5. `SharedMemory::at(0)` 的 Rust 有效性

- 当前证据: RP 视角 base 0；`ShmDriver` 把 `usize::MAX` 作为未 probe 哨兵；当前没有 `ov-channels` 实现可读。
- 禁止推断: 不由"使用 base 0"推定可安全构造普通 Rust 零地址引用；不假定按 `SharedMemory::at(0)` 一定能得到合法 `&SharedMemory`。
- 解除条件: 取得 `SharedMemory::at` 实现并补查其裸指针、MaybeUninit 或 volatile 读写策略。
- 影响主题: 用户态 mmap、用户 mmap 整窗的访问权限、AP/RP alias 真板验证。

## 11. 修订与验证边界

| 材料 | 捕获状态 | 本文可用结论 | 不支持的结论 |
| --- | --- | --- | --- |
| SpacemiT 官网 boot 页面 | 2026-09-02，SPA 壳 | 官方入口和 K3 范围 | AMP 消息路径数值、ring/RPC 行为 |
| docs-buildroot boot.md | 2026-09-07，revision unknown | K3 通用启动阶段 | 第三方 AMP ring/RPC/通知时序 |
| Linux `k3.dtsi` | `k3-br-v1.0.y`，2026-09-07 | AP 地址 cells、reserved-memory 一般结构 | 共享 ring 节点、mailbox 4 节点细节 |
| Rt-Async-AMP | `ccb1ff0b`，本 Cycle 核对 HEAD 匹配 | ring 角色、BUSY、doorbell、Deferred、reset 调用点 | 本项目构建或真板通过 |
| tgoskits | `19219411d`，本 Cycle 核对 HEAD 匹配 | AP 路径 + APLIC/IMSIC 调用 + IPC_WAKER 行为 | 通用 ABI、安全权限或目标板默认配置 |
| OpenSBI | `7a2df083`，本 Cycle 核对 HEAD 匹配 | PMA 查找与写入控制流 | 消息路径 mailbox 寄存器布局 |

本 Cycle 没有运行第三方 build、host tests、QEMU、刷写或真板实验。Rt-Async-AMP 根 workspace 的 offline Cargo metadata 因缺少 `rt-async/modules/platform/Cargo.toml` 退出 101；这证明依赖材料不完整，不是产品构建失败。（`未知项`）

## 12. 与相邻主题的关系

- [`k3-amp-shared-memory-lifecycle.md`](k3-amp-shared-memory-lifecycle.md)：AP/RP 镜像装载、共享窗口地址、初始化所有权、生命周期状态、失效复位与重新初始化。消息路径不重新描述这些状态。
- [`../interrupts/com260-mailbox-notification.md`](../interrupts/com260-mailbox-notification.md)：mailbox4 静态配置、FIFO/pending、IRQ 与等待风险、RP handler 经验。本文只描述消息路径如何与 mailbox 通道交互，不重复寄存器说明。
- [`../dma/k3-dma-and-memory-ownership.md`](../dma/k3-dma-and-memory-ownership.md)：共享内存不是设备 DMA；对象 ownership、通知与数据可见性的共同边界。
- [`../dma/k3-cache-pma-address-translation.md`](../dma/k3-cache-pma-address-translation.md)：cache、fence、PMA/PBMT 与 IOMMU 机制；本文不外推到 DDR 或设备 DMA。
- [`../network/k3-gmac-dma-irq.md`](../network/k3-gmac-dma-irq.md)：GMAC descriptor 与 data buffer、Release fence、descriptor clean/invalidate、doorbell 和回收。mailbox doorbell 与 GMAC doorbell 在硬件层是不同机制，不要混用术语。
- [R10 共享内存分析](../../.claude/analysis/rt-async-amp-k3-shared-memory.md) 与 [R11 驱动分析](../../.claude/analysis/rt-async-amp-k3-drivers.md)：固定 revision 调查入口；正文已经写入执行所需边界，不要求读者回读分析才能理解消息路径。
