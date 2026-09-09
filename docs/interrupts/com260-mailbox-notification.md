> 来源: https://github.com/spacemit-com/linux-6.18（源端修订: unknown；观察日期: 2026-09-05）; https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-07）; https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-rdomain.dtsi（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-08）

# CoM260 mailbox 通知机制

> 范围: 记录固定 revision 第三方工程中 AP 与 RP 通过 K3 mailbox 互相通知的静态配置、处理顺序和失败边界。mailbox 只传递门铃；业务状态与数据保存在共享内存，由接收方在收到通知后重新读取。
>
> 来源边界: 官方 linux-6.18 仓库只作为 K3 DTS 与中断域的交叉验证来源。mailbox4 的实例选择、USER/channel、IRQ、处理顺序和自测来自 R10–R12 指向的第三方工程：Rt-Async-AMP `ccb1ff0b487e4f49ea570c41f330741eecece935`、tgoskits `19219411d5dc1515496f910d04c93da12ee95be4`。本文同时记录来源层与四级证据：直接读取第三方配置或代码所得事实标为“`第三方经验 / 未知项`”，表示其 K3 硬件通用性或本项目适用性尚未获官方或运行证据确认；由多处固定 revision 行为组合出的链路、边界或风险标为“`第三方经验 / 推论`”并列出依据。第三方材料不构成 K3 硬件规范或本项目真板验证结果。

## 目录

- [1. 名称和编号空间](#1-名称和编号空间)
- [2. 固定 revision 静态配置](#2-固定-revision-静态配置)
- [3. AP 到 RP 通知链](#3-ap-到-rp-通知链)
- [4. RP 到 AP 通知链](#4-rp-到-ap-通知链)
- [5. FIFO、pending 与清除顺序](#5-fifopending-与清除顺序)
- [6. 等待、工作上限与并发边界](#6-等待工作上限与并发边界)
- [7. 自测能力与证据边界](#7-自测能力与证据边界)
- [8. 未知项](#8-未知项)
- [9. 与中断和数据面的边界](#9-与中断和数据面的边界)

## 1. 名称和编号空间

下列编号来自不同命名空间，数值相同也不表示同一资源。

| 名称 | 本文含义 | 当前可证值 | 来源层 / 证据等级 |
| --- | --- | --- | --- |
| 物理 mailbox 实例 | SoC mailbox 控制器实例 | 第三方配置选择 mailbox4，base `0xcac91000`，size `0x400` | 第三方经验 / 未知项（依据：两侧 DTS；K3 通用性待证） |
| `MBX3` | RP 驱动实例池第一个历史变量名 | 实际绑定物理 mailbox4；不是物理 mailbox3 | 第三方经验 / 未知项（依据：RP `mailbox.rs`；本项目适用性待证） |
| hardware channel | mailbox FIFO 与 NEW_MSG bit 的通道 | AP→RP 为 0；RP→AP 为 1 | 第三方经验 / 未知项（依据：两侧 notifier 配置；硬件定义待证） |
| mailbox user | 每个 user 的 IRQ 寄存器组 | AP/StarryOS 为 USER0；RP/rcpu1 为 USER1 | 第三方经验 / 未知项（依据：两侧驱动；硬件定义待证） |
| AP wired source | mailbox4 到 AP APLIC 的线中断号 | source 217 | 第三方经验 / 未知项（依据：AP DTS；官方映射待证） |
| RP wired source | mailbox4 到 RP PLIC 的线中断号 | source 69 | 第三方经验 / 未知项（依据：RP DTS；官方映射待证） |
| IMSIC EID | APLIC MSI 模式投递到 hart interrupt file 的 identity | 未知 | 第三方经验 / 未知项（第三方材料也未给出映射） |
| ov-channels channel | 共享内存数据队列编号 | 不在本文展开 | 第三方经验 / 推论（依据：数据面与 mailbox 配置使用独立字段） |

> 命名约束（第三方经验 / 推论）: `MBX3` 只能写成“RP 驱动历史变量/slot”；物理实例始终写 mailbox4。source 217、source 69、hardware channel 0/1、ov-channels channel 和 IMSIC EID 不得互相换算。依据是两侧 DTS、驱动变量绑定及各字段所属控制器不同；不存在数值映射证据。

## 2. 固定 revision 静态配置

| 侧 | 配置入口 | mailbox | user | 发送/接收 channel | 接收中断 | 来源层 / 证据等级 |
| --- | --- | --- | --- | --- | --- | --- |
| AP | `tgoskits/os/StarryOS/configs/board/spacemit-k3-com260-ifx.dts` 的 `ov,k3-mailbox-notifier` | `0xcac91000/0x400` | local USER0，remote USER1 | tx 0 / rx 1 | APLIC source 217 | 第三方经验 / 未知项（K3 硬件适用性待证） |
| RP | `its/rt-async-k3.dts` 的 `mailbox@cac91000` | `0xcac91000/0x400` | local USER1，remote USER0 | tx 1 / rx 0 | PLIC source 69 | 第三方经验 / 未知项（K3 硬件适用性待证） |

来源层 / 证据等级：`第三方经验 / 未知项`。RP DTS 注释称物理 mailbox3 已由 rcpu0/esos 使用，mailbox4 在该 DTB 中空闲，因此第三方工程选择 mailbox4。仓库没有本项目读取 rcpu0 官方 DTB 或板上枚举的独立证据；实例占用关系不得升级为官方事实。

来源层 / 证据等级：`第三方经验 / 未知项`。第三方 RP 驱动把 mailbox 描述为 4 个 hardware channel、每个 FIFO 深度 8。NEW_MSG 使用每个 channel 两位状态中的偶数位，即 `1 << (channel * 2)`。寄存器布局与该工程实现绑定；缺少公开 K3 programmer reference 时，不得将偏移和位定义写成通用硬件契约。

## 3. AP 到 RP 通知链

来源层 / 证据等级：`第三方经验 / 推论`。下列端到端链由 AP notifier、RP DTS 与 RP handler 三处固定 revision 行为组合，不表示本项目已经运行该链。

```text
AP 发布共享状态
  → Release fence
  → USER1 的 channel 0 NEW_MSG enable
  → 写 mailbox4 channel 0 FIFO
  → RP PLIC source 69
  → RP mailbox ISR 排空 FIFO、清 pending、重读状态
  → IrqLatch 唤醒等待任务
  → RP 重新读取共享状态
```

| 步骤 | 固定 revision 行为 | 边界 | 来源层 / 证据等级 |
| --- | --- | --- | --- |
| 发布 | AP `K3MailboxIpiSender::notify_peer` 在门铃前执行 Release fence | fence 只描述第三方实现，不证明所有 AP producer 都正确发布 | 第三方经验 / 未知项（本项目执行待证） |
| 门铃 | AP 对 remote USER1 enable channel 0，再向 channel 0 FIFO 写入值 `1` | FIFO 值是门铃，不是业务 payload | 第三方经验 / 推论（依据：写值固定而业务状态另存共享内存） |
| 投递 | RP DTS 把 mailbox4 配为 IRQ 69，RP 驱动经 PLIC 注册 handler | PLIC 地址、优先级和真板 delivery 未验证 | 第三方经验 / 未知项 |
| 处理 | RP ISR 排空有 pending 的 FIFO，RMW 清 pending，并循环重读 `RAW & EN` | 不据此声明任意 burst 下无丢失 | 第三方经验 / 未知项（真板行为待证） |
| 唤醒 | ISR 调用 `IrqLatch::notify`，等待任务恢复后检查共享状态 | 一次通知不等于一条业务消息 | 第三方经验 / 推论（依据：latch 与共享状态读取分离） |

## 4. RP 到 AP 通知链

来源层 / 证据等级：`第三方经验 / 推论`。下列端到端链由 RP notifier、AP DTS、已确认的 APLIC `msi-parent` 与 AP handler 组合；具体 EID、hart 和运行结果仍未知。

```text
RP 发布共享状态
  → Release fence
  → USER0 的 channel 1 NEW_MSG enable
  → 写 mailbox4 channel 1 FIFO
  → AP APLIC source 217
  → APLIC 以 MSI 模式投递到 IMSIC（EID 未知）
  → AP handler 排空 FIFO、清 pending、重读状态
  → 唤醒 AWAIT/epoll 等待者
  → AP 重新读取共享状态
```

| 步骤 | 固定 revision 行为 | 边界 | 来源层 / 证据等级 |
| --- | --- | --- | --- |
| 发布 | RP `PeerNotifier::notify` 在 `signal(1)` 前执行 Release fence | 不替代共享内存 ownership 与 cache 规则 | 第三方经验 / 未知项（本项目执行待证） |
| 门铃 | RP 对 remote USER0 enable channel 1，再写 channel 1 FIFO | channel 1 不是 ov-channels 的 channel 1 | 第三方经验 / 推论（依据：两类 channel 来自独立配置字段） |
| 投递 | AP DTS 把 mailbox4 接收线写为 APLIC source 217；Iteration 000 已确认 APLIC 的 `msi-parent = <&simsic>` | source 到 EID、目标 hart 与 affinity 未观察到 | 第三方经验 / 未知项 |
| 处理 | AP endpoint 调用 `ack_and_clear(rx_channel)`，随后返回 `PeerNotify` | handler 触发只表示门铃到达 | 第三方经验 / 推论（依据：handler 返回通知对象，不读取业务 payload） |
| 唤醒 | AP 从单槽 `IPC_WAKER` 取出 waker 并唤醒 | 多等待者会发生后注册覆盖风险 | 第三方经验 / 推论（依据：全局状态仅保存一个 `Waker`） |

## 5. FIFO、pending 与清除顺序

来源层 / 证据等级：`第三方经验 / 未知项`（K3 硬件通用性与本项目运行结果待证）。第三方两侧实现都采用以下顺序：

1. 初始化或 enable 前读取 FIFO，直到 empty。
2. 清除本地 user 的 NEW_MSG pending。
3. enable 本地接收 channel。
4. ISR 中至少读取一次 `mbox_msg`，再按 `msg_count` 排空 FIFO。
5. 对 clear 寄存器执行保留其他位的 RMW。
6. 重读 `IRQSTATUS_RAW & IRQENABLE_SET`；仍有 pending 时继续处理。
7. pending 归零后才通知等待者并返回。

证据等级：`推论`。依据是上述 drain、clear、重读控制流及 pending/FIFO 状态分离；该顺序针对两类风险：

- 启用前残留：旧 FIFO/pending 会让第一次等待误消费“幽灵通知”。
- drain/clear 竞态：ISR 处理期间到达的新门铃可能使 pending 再次置位；只清一次就返回可能让后续通知不可见。

来源层 / 证据等级：上限数值为`第三方经验 / 未知项`（运行表现待证）；风险结论为`第三方经验 / 推论`，依据是循环边界。AP `ack_and_clear` 的固定 revision 实现最多执行 32 轮外层 pending 检查，每轮最多读取 FIFO 64 次。达到上限后的 pending 状态、重试策略和错误传播没有独立验证，不能声明为“全部排空”。RP ISR 外层循环没有相同的显式轮数上限；这不证明其不会在持续输入下占满 IRQ 上下文。

## 6. 等待、工作上限与并发边界

| 边界 | 当前行为 | 风险 | 来源层 / 证据等级 |
| --- | --- | --- | --- |
| RP async wait | `IrqLatch` 使用关中断、注册 waker、重检、开中断的等待模式 | 只证明固定 revision 的注册竞态处理 | 第三方经验 / 未知项（运行结果待证） |
| AP wait | `IPC_WAKER` 是单槽 `Option<Waker>` | 两个等待者并发时后注册者可覆盖先注册者 | 第三方经验 / 推论（依据：只有一个存储槽） |
| ISR 工作量 | drain 与 pending 重读在 IRQ 上下文执行 | burst 可增加 IRQ 延迟；无本项目测量 | 第三方经验 / 推论（依据：有界/循环 drain 位于 handler） |
| 通知合并 | FIFO/pending 表示门铃状态 | 多次状态更新可由一次唤醒合并，接收方必须重查共享真值 | 第三方经验 / 推论（依据：通知值与业务状态分离） |
| 重复通知 | 清除期间的新门铃可再次置位 | handler 和上层逻辑必须允许重复唤醒 | 第三方经验 / 推论（依据：clear 后重读 pending） |
| 错误传播 | AP 自测超时返回 I/O 错误；正常通知路径主要通过状态重查恢复 | 不证明双端故障能被同步报告 | 第三方经验 / 未知项（双端传播待证） |

证据等级：`推论`；依据是 notifier/handler 只操作门铃寄存器，而业务状态由独立共享内存路径读取。本文不规定 RPC 请求数、共享内存 ring 容量、消息格式或 cache ownership；这些属于数据面。通知层只保证“尝试唤醒接收方”，不承诺通知次数与业务状态变化一一对应。

## 7. 自测能力与证据边界

来源层 / 证据等级：`第三方经验 / 未知项`（本项目是否启用或执行待证）。AP `rt_shm.rs` 包含两类自测入口：启动时自动调用的本地 mailbox IRQ 测试，以及用户态 `user-test-mbox` 可触发的 ioctl 测试。测试对本地 USER0 的 rx channel 写 FIFO，比较 `K3_MBOX_IRQ_COUNT` 是否增加。

来源层 / 证据等级：`第三方经验 / 未知项`（运行效果待证）。固定 revision 的尝试预算为 3 次；每次最多执行 100000 次 spin。失败时打印 `IRQSTATUS_RAW` 与 `IRQENABLE`，用于区分 mailbox 未置位和已置位但未投递两类线索。

证据等级：`推论`；依据是自测写 FIFO、IRQ 计数比较和 handler 控制流。若按源码运行，该自测能覆盖第三方 AP 配置中的：

```text
mailbox4 FIFO → USER0 NEW_MSG → APLIC → IMSIC/CPU → AP handler
```

证据等级：`推论`；依据是测试只在 AP 本地写入并观察单一计数。它不能覆盖：

- AP→RP 的 PLIC 69 投递与 RP handler；
- RP→AP 的真实对端写入、共享状态发布和读取；
- source 217 到具体 IMSIC EID/hart 的稳定映射；
- burst、长时间运行、SMP affinity、重复或丢失通知；
- 本项目 CoM260 真板是否实际执行并通过测试。

证据等级：`未知项`。本仓库当前只有源码入口，没有本项目运行日志，因此状态是“自测能力存在、未运行”，不得写成 PASS；解除条件是保存本项目可审计的运行输出。

## 8. 未知项

### U1. mailbox4 官方实例、user 与 channel 定义

- 当前证据: 固定 revision 第三方 DTS 与两侧驱动一致使用 `0xcac91000/0x400`、USER0/USER1、channel 0/1；官方主题来源未提供 mailbox programmer reference。
- 禁止推断: 不得把第三方地址、FIFO 深度、寄存器偏移或 user 映射写成 K3 全版本硬件规范。
- 解除条件: 取得 SpacemiT K3 programmer reference 的 mailbox 章节，或官方 DTS/binding 对应节点与字段。
- 影响主题: 本文静态配置、初始化与 ISR 顺序；G4。

### U2. AP source 217 到 IMSIC EID 与目标 hart

- 当前证据: 第三方 AP DTS 给出 APLIC source 217；官方 `k3.dtsi` 只证明 APLIC→IMSIC 的 `msi-parent`，没有本 mailbox 的 EID 与 affinity 映射。
- 禁止推断: 不得令 EID=217，不得把 source 217 与 RP source 69 或 channel 号等同，不得声明固定目标 hart。
- 解除条件: 取得 K3 mailbox 官方 DTS、运行时 IRQ domain 映射或可复现的 handler/affinity 输出。
- 影响主题: RP→AP 通知链、自测覆盖、SMP 路由；G4。

### U3. 真板双向通知可靠性

- 当前证据: 固定 revision 源码包含 residual drain、pending 重读和本地 AP 自测入口，但本项目没有运行日志。
- 禁止推断: 不得声明自测通过、无丢通知、无重复通知、burst 安全或无限期稳定。
- 解除条件: 在目标 CoM260 配置上运行双向、残留、burst、超时与恢复测试，并保存可审计结果。
- 影响主题: mailbox 可用性、后续驱动验证和运行手册。

### U4. 多等待者与工作预算

- 当前证据: AP 使用单槽 waker；AP drain 有 32×64 上限；RP ISR 未设置等价的显式轮数上限。
- 禁止推断: 不得声明多消费者安全、IRQ 延迟有界或错误总能传播到所有等待者。
- 解除条件: 明确单等待者协议，或实现并验证多等待者队列；对持续 burst 测量 ISR 工作量与超限行为。
- 影响主题: AWAIT/epoll 使用约束、调度延迟、错误恢复。

## 9. 与中断和数据面的边界

证据等级：`推论`；依据是本文固定 revision notifier/handler 控制流、Iteration 000 中断域事实以及 U2/U3 的证据缺口。

- AP APLIC/IMSIC、RP PLIC/SysTimer 与 timer 的来源边界见 [`k3-interrupt-and-time.md`](k3-interrupt-and-time.md)。
- mailbox notification 只促使接收方重查共享状态；它不携带业务数据，也不证明 cache、barrier 或 ownership 协议正确。
- 第三方工程中的共享内存 channel、RPC mode、ring 大小和消息格式不在本文展开。
- CoM260 是否实际启用该 notifier、source 217/69 是否在目标镜像中注册，以及 self-test 是否通过，均需运行证据。
