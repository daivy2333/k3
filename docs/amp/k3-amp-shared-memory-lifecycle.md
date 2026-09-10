> 来源: https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/boot.md（源端修订: unknown；观察日期: 2026-09-02）；https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/boot.md（源端修订: unknown；观察日期: 2026-09-07）；https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-07）

# K3 AMP 与共享内存生命周期

> 范围: 串联 K3 AP/RP 镜像与握手、共享窗口地址、初始化所有权、内存属性前置、失效和重新初始化。启动介质、mailbox 寄存器、PMA/PBMT 机制和 ring/RPC 消息语义分别由既有主题或后续文档承担。
>
> 来源边界: SpacemiT 官网和官方 GitHub 只支持 K3 启动阶段、地址空间与 reserved-memory 的一般边界。AMP 数值和状态转换来自 R09/R10 固定 revision 第三方调查：Rt-Async-AMP `ccb1ff0b487e4f49ea570c41f330741eecece935`、tgoskits `19219411d5dc1515496f910d04c93da12ee95be4`、OpenSBI `7a2df083ed06373c506e2e6f4e09bbd168202f2d`。第三方代码、注释和历史测量不构成 K3 官方规范或本项目真板结果。
>
> 证据等级: 使用 `官方事实`、`交叉验证`、`推论`、`未知项`。直接读取固定 revision 第三方代码所得行为写为“第三方实现 / 未知项”；跨文件组合的路径写为“第三方实现 / 推论”。

## 目录

- [1. 文档职责与既有基线](#1-文档职责与既有基线)
- [2. AP/RP 镜像、装载与握手](#2-aprp-镜像装载与握手)
- [3. 共享窗口地址与布局边界](#3-共享窗口地址与布局边界)
- [4. 内存属性与可见性前置](#4-内存属性与可见性前置)
- [5. 生命周期状态](#5-生命周期状态)
- [6. 初始化所有权与竞争](#6-初始化所有权与竞争)
- [7. 失效、复位与重新初始化](#7-失效复位与重新初始化)
- [8. 未知项](#8-未知项)
- [9. 修订与验证边界](#9-修订与验证边界)
- [10. 与相邻主题的关系](#10-与相邻主题的关系)

## 1. 文档职责与既有基线

- K3 官方 SDK 的 local boot 阶段为 Boot ROM → FSBL/SPL → ESOS → OpenSBI → U-Boot → payload/OS；完整介质、版本和刷写边界见 [`com260-boot-chain.md`](../boot/com260-boot-chain.md)。（`交叉验证`）
- K3 官方 `k3.dtsi` 定义两 cell 地址空间和 `reserved-memory` 容器，但当前官方来源未直接给出本第三方 AMP 窗口、双端 alias 或初始化协议。（`交叉验证` + `未知项`）
- mailbox 在 AMP 路径中只提供通知；FIFO、pending、USER/channel 与中断域见 [`com260-mailbox-notification.md`](../interrupts/com260-mailbox-notification.md)。（`推论`，依据：通知处理与共享状态访问分离）
- AP↔RP 共享内存不是设备 DMA。ownership、通知与数据可见性的共同边界见 [`k3-dma-and-memory-ownership.md`](../dma/k3-dma-and-memory-ownership.md)。（`推论`）
- 本文中的“AP 初始化”“RP fallback”“watchdog 自愈”只描述指定第三方 revision 的控制流，不规定新实现必须采用相同所有者、延时或重试次数。（`未知项`）

## 2. AP/RP 镜像、装载与握手

### 2.1 第三方镜像集合

| 对象 | 固定 revision 配置 | 生命周期含义 | 来源层 / 证据等级 |
| --- | --- | --- | --- |
| AP OpenSBI | `opensbi.itb`；generic platform + `k3_defconfig` | 在 AP payload 之前运行，并尝试配置 AMP 窗口 PMA | 第三方实现 / 未知项 |
| AP kernel | `starryos.uimg` 内 kernel load `0x140000000`、DTB load `0x138000000` | 该第三方 FIT 的内部装载地址；不是 CoM260 官方默认值 | 第三方实现 / 未知项 |
| RP rcpu0 | ITS load/entry `0x100200000`；当前 payload 为握手后永久 WFI 的占位 ELF | `esos` 容器名不表示 rcpu0 当前运行完整 ESOS | 第三方实现 / 未知项 |
| RP rcpu1 | ITS load/entry `0x100804000`；app linker RAM 起点相同 | 第三方 RTOS/应用入口；启动汇编和 linker 依赖缺失 `rt-async` | 第三方实现 / 未知项 |
| RP DTB | ITS 固定 DTB 与 rcpu1 ELF 内嵌 DTB 同时存在 | 当前 `K3Rt24::init` 使用内嵌 DTB；loader 是否另传固定 DTB 未核对 | 第三方实现 / 未知项 |

表中地址来自 [R09 启动与板级分析](../../.claude/analysis/rt-async-amp-k3-boot-platform.md) 对固定 revision 文件的读取。AP 示例把整个 FIT 上传到 `0x180000000`；上传缓冲地址不能替代 FIT 内 kernel/DTB 的 load address。（`第三方实现 / 推论`）

### 2.2 握手边界

- rcpu1 的 `spl_handshake()` 写 `0xc088007c = 1`，随后继续板级初始化。（`第三方实现 / 未知项`）
- 当前 rcpu0 占位 payload 写 `0xc088008c = 1`、执行 fence 后永久 WFI。（`第三方实现 / 未知项`）
- “U-Boot 最多轮询六秒”“只装载 ELF `PT_LOAD`”“启动前清 SRAM”等描述尚缺实际 U-Boot K3 分支核对；这些是作者声明，不是本文确认的 loader 契约。（`未知项`）
- rcpu0 固定 ELF 的 entry 是 `0x100200000`，但 LOAD 段从 `0x1001ff000` 开始。检查内存冲突时必须查看完整段范围，不能只比较 entry 或 ITS load。（`第三方实现 / 推论`）

握手寄存器写入只能说明对应 payload 的可读行为。它不能证明 AP/RP 镜像已在目标 CoM260 上按此顺序运行，也不能证明写入后共享窗口已经初始化。（`未知项`）

## 3. 共享窗口地址与布局边界

| 观察侧 | 地址与大小 | 当前解释 | 尚不能确认的内容 |
| --- | --- | --- | --- |
| AP 第三方 DTS | `0xc0800000..0xc0819000`，size `0x19000` | `no-map` reserved-memory，`ov,rt-async-amp` 节点再次引用同一区间 | 目标 Kit 是否使用该 DTS；物理窗口属性与权限 |
| AP `rt_shm` | 对上述物理区间执行 ioremap，并允许用户 mmap 整窗 | kernel 和用户态共同访问 SharedMemory | 多进程隔离、访问权限和实际 non-cacheable 属性 |
| RP 第三方 DTS | base 0、size `0x19000` | 注释称这是同一 SRAM 的 RCPU 本地 alias | 0 与 `0xc0800000` 是否在真板双向指向同一存储 |
| K3 SoC 既有正文 | SoC 有 512 KB 共享 SRAM | 只提供容量级背景 | 不提供本第三方 `0x19000` 子窗口或 alias 映射 |

前三行是固定 revision 第三方配置的自洽关系，证据等级为 `第三方实现 / 未知项`。官方 `k3.dtsi` 的两 cell 地址和 reserved-memory 结构只作 `交叉验证`，不能补证第三方窗口。

第三方 DTS 注释估算窗口为头部 `0x100`、三个 `0x8200` 通道和尾部 `0x800` scratch。由于 `ov-channels` 源码缺失，本文不确认对象 size、alignment、ring 容量或 `SharedMemory::at(0)` 的 Rust 有效性。（`未知项`）

地址 0 还是 AP 物理地址取决于访问侧。文档和后续实现不得把这两个数值当作可互换的普通指针，也不得用数值相等或共享结构类型替代 alias 的真板验证。（`推论`）

## 4. 内存属性与可见性前置

### 4.1 AP OpenSBI PMA 路径

固定 revision OpenSBI 的 `k3_pma_set_amp_window_io` 执行以下步骤：（`第三方实现 / 未知项`）

1. 读取 16 个 PMAADDR，把相邻值解释为范围。
2. 查找覆盖 `[0xc0800000, 0xc0880000)` 的 entry。
3. 找到后先 clean 该范围，再把对应 PMACFG byte 写为 `0x22`，最后执行 `sfence.vma`。
4. cold 和 warm 路径都调用；找不到 entry 时记录信息并继续启动。

PMA 搜索范围是 `0x80000`，大于应用窗口 `0x19000`。因此需要核对实际 entry 上下界、属性编码和同 entry 中的其他用途；不能把“找到并写入”直接表述为最小范围隔离。（`推论`）

### 4.2 RP 与共享访问

- RP 代码不访问 rcpu1 上可能挂死的 custom cache/PMA CSR；这只说明固定 revision 的规避策略。（`第三方实现 / 未知项`）
- `ov-shm::flush()` 只执行 `fence iorw,iorw`。fence 提供排序，不是 cache clean，也不证明另一侧立即可见。（`第三方实现 / 推论`）
- AP 代码已撤除原 user CBO 路径，并依赖 OpenSBI 把窗口设为 IO；注释仍有旧 CBO 描述，结论应以实际调用路径为准。（`第三方实现 / 未知项`）
- Svpbmt、PMA、cache 和 IOMMU 的机制边界见 [`k3-cache-pma-address-translation.md`](../dma/k3-cache-pma-address-translation.md)。本文不把共享 SRAM 的处理外推到 DDR 或设备 DMA。（`推论`）

双端上线至少依赖：地址映射指向同一物理存储、属性允许需要的读写、发布与读取具备匹配的排序、启动阶段不再破坏窗口。当前材料不能同时证明这四项在本项目真板成立。（`未知项`）

## 5. 生命周期状态

| 状态 | 当前所有者或参与者 | 允许观察/动作 | 转移条件 | 失败边界 |
| --- | --- | --- | --- | --- |
| 镜像准备 | 构建/loader 配置 | 检查 FIT、ELF `PT_LOAD`、DTB 和 reserved-memory 是否重叠 | loader 接受镜像 | U-Boot loader 源码缺失，实际装载未知 |
| 启动链可能破坏窗口 | SPL、U-Boot、bootm/cache 路径 | RP 只等待，不发布共享服务 | 最后一个窗口写入者退出 | “最后写入”只来自第三方注释，时点未独立验证 |
| AP 属性准备 | OpenSBI | 尝试把覆盖范围 PMA 设为 IO | 找到或未找到 entry 后继续启动 | 未找到仍继续，属性失败不阻止后续 probe |
| AP probe | AP `rt_shm` | ioremap；valid 则保留，invalid 则 init；清残留 mailbox 后 enable | 窗口有效且通知端就绪 | AP 与 RP 可同时拥有 init 路径 |
| RP 等待 | `wait_ready` | 门控后只读 magic | valid → 发布 `SHM_BASE`；timeout → 本地 init | 固定门控和 timeout 不是硬件保证 |
| 在线 | AP/RP 通信任务 | 读写共享状态，doorbell 促使对端重检 | magic 失效、对端 reset 或访问错误 | 数据/通知/RPC 语义由后续文档承担 |
| 失效 | 首个发现 invalid 的参与者 | 停止把旧 ring 当作可信状态 | watchdog 或启动路径选择 re-init | 在途请求、Deferred 和未读消息状态未知 |
| 重建 | RP watchdog 或其他被选定初始化者 | `SharedMemory::init` 重写头部并重新发布 | 双端重新确认可用 | re-init 会清未读消息，不是无损恢复 |

“启动链可能破坏窗口”到“AP probe”的顺序来自第三方作者对 SPL/U-Boot/cache 写回的说明，证据等级为 `未知项`。`init → SHM_BASE Release publish`、valid 保留和 watchdog re-init 是代码可读行为，证据等级为 `第三方实现 / 未知项`。整张表组合出的端到端顺序是 `推论`，本项目未执行。

## 6. 初始化所有权与竞争

第三方组合同时存在三条初始化路径：

```text
AP rt_shm probe: invalid → init
RP wait_ready: fallback timeout → init
RP magic_watchdog: invalid 且 heals < 8 → init
```

- AP probe 把初始化安排在 AP kernel 启动后；窗口已 valid 时不清 ring。（`第三方实现 / 未知项`）
- RP `wait_ready` 先等待 3 秒，再按 `poll_ms` 检查；超过 `fallback_ms` 后为兼容旧 AP kernel 而本地初始化。（`第三方实现 / 未知项`）
- RP watchdog 等待 3 秒后每秒检查，首次检查约在本任务启动后 4 秒；最多 re-init 8 次。（`第三方实现 / 未知项`）
- `intercom::init` 先初始化窗口，再以 Release 存储 `SHM_BASE`；这是防止 ISR/任务看到“地址已发布但 ring 未初始化”的本地顺序。（`第三方实现 / 推论`）

这些路径没有形成“任何时刻只有一个初始化者”的可证协议。AP probe 与 RP fallback/watchdog 的先后依赖启动时序和 magic 观察；两侧同时初始化、初始化期间收到通知、旧端仍持有 request ID 时的结果均未知。（`未知项`）

可复用的设计原则只有：最后一个可能破坏窗口的阶段结束后再初始化；发布可用状态必须晚于结构初始化；恢复期间不得把旧消息视为有效。固定 3 秒、10 秒或 8 次不是 K3 通用参数。（`推论`）

## 7. 失效、复位与重新初始化

### 7.1 窗口仍有效

AP probe 遇到 valid magic 时保留现有 ring；RP `wait_ready` 遇到 valid 时只发布本地基址。该路径有意避免重复清空，但尚未证明 magic 足以判断布局版本、两侧协议兼容或所有在途状态仍有效。（`第三方实现 / 未知项`）

### 7.2 magic 无效

RP watchdog 的策略是重新调用 `intercom::init`。源码明确承认这会清空各通道未读消息，并把持续外部破坏限制为最多 8 次自愈。日志中的历史失效时间和“只发生一次”属于作者环境记录；本项目没有原始日志或复现实验。（`第三方实现 / 未知项`）

### 7.3 对端复位

对端 reset 可能经过 U-Boot 再次清 SRAM，也可能保留物理窗口但丢失其本地 request、waker 或服务状态。当前协议没有可读证据定义以下结果：（`未知项`）

- 哪一侧先暂停新的发送；
- 在途请求和 Deferred 响应返回何种错误；
- 旧 request ID 是否允许跨 epoch 继续匹配；
- 已发布但未通知或已通知但未读取的消息如何处理；
- 双端以什么条件确认重新上线。

因此 re-init 只能写成“重新建立结构头部并可能恢复后续通信”，不能写成“恢复全部业务状态”。调用方的 timeout 或取消也不能单独证明共享 slot 已被对端释放。（`推论`）

## 8. 未知项

### U1. 目标 CoM260 的 AMP 镜像与 loader 交接

- 当前证据: 固定 revision ITS/ELF/内嵌 DTB 和握手写入可读；官方 boot 文档只提供 K3 通用阶段。
- 禁止推断: 不把第三方 IFX DTS、load address、六秒轮询或 rcpu payload 写成目标 Kit 默认配置。
- 解除条件: 取得目标板实际 U-Boot K3 分支、FIT 配置、完整 `PT_LOAD` 检查及可审计启动输出。
- 影响主题: 镜像准备、握手、最后窗口破坏者和共享服务上线。

### U2. AP/RP 地址 alias 与窗口权限

- 当前证据: AP DTS 使用 `0xc0800000/0x19000`，RP DTS 使用 `0/0x19000`；配置在第三方工程内自洽。
- 禁止推断: 不由相同 size、结构 magic 或 DTS 注释认定两侧地址必然指向同一物理 SRAM；不认定用户 mmap 整窗符合目标权限模型。
- 解除条件: 取得官方地址映射说明，并在目标 CoM260 上完成双向读写、边界和权限验证。
- 影响主题: 共享窗口定位、用户映射、数据可见性和隔离。

### U3. PMA 范围、编码与每 hart 实效

- 当前证据: 固定 revision OpenSBI 查找覆盖 `0xc0800000..0xc0880000` 的 entry 并写 `0x22`；G5 已记录 PMA/PBMT 部分证据。
- 禁止推断: 不把 `0x22`、entry 数或整个覆盖范围视为所有 K3 固件的固定配置；不由启动继续认定属性设置成功。
- 解除条件: 取得 K3 PMA programmer 资料，在所有相关 hart 读回 entry 上下界/属性，并验证共享窗读写与相邻区域不受误设影响。
- 影响主题: AP 属性准备、共享窗口可见性和 G5。

### U4. 唯一初始化者与恢复协议

- 当前证据: AP probe、RP fallback 和 RP watchdog 均可调用 init；watchdog re-init 会清未读消息。
- 禁止推断: 不把 magic 当作完整 epoch/version 协议，不把 re-init 表述为无损或并发安全。
- 解除条件: 明确每个启动/reset 场景的唯一初始化所有者、通信暂停与恢复条件，并验证并发 init、在途请求和重复 reset。
- 影响主题: 生命周期状态、错误传播、后续 ring/RPC 文档和 G11（由 Iteration 001 登记）。

### U5. 缺失子仓承载的启动和布局语义

- 当前证据: `rt-async/modules/platform` 与 `modules/ov-channels` 在固定 revision checkout 中缺失；现有调用点可读。
- 禁止推断: 不从调用者补写启动汇编、IrqLatch、ring size/alignment、原子序或零地址引用有效性。
- 解除条件: 按主仓 gitlink 补齐对应提交，完整读取实现并重新核对本文受影响的地址、发布和恢复结论。
- 影响主题: RP 启动、共享布局、上线发布和后续消息路径。

## 9. 修订与验证边界

| 材料 | 捕获状态 | 本文可用结论 | 不支持的结论 |
| --- | --- | --- | --- |
| SpacemiT 官网 boot 页面 | 2026-09-02，SPA 壳 | 官方入口和 K3 范围 | AMP 地址、loader 或初始化协议 |
| docs-buildroot boot.md | 2026-09-07，revision unknown | K3 通用启动阶段 | 第三方 AMP 时序和窗口 |
| Linux `k3.dtsi` | `k3-br-v1.0.y`，2026-09-07 | AP 地址 cells、reserved-memory 一般结构 | `0xc0800000/0x19000` 为官方 AMP 节点 |
| Rt-Async-AMP | `ccb1ff0b`，本 Cycle 核对 HEAD 匹配 | 调用点、ITS/DTS、状态控制流 | 本项目构建或真板通过 |
| tgoskits | `19219411d`，本 Cycle 核对 HEAD 匹配 | AP `rt_shm` 固定 revision 行为 | 通用 ABI、安全权限或目标板默认配置 |
| OpenSBI | `7a2df083`，本 Cycle 核对 HEAD 匹配 | PMA 查找与写入控制流 | PMA 在本项目各 hart 实际生效 |

本 Cycle 没有运行第三方 build、host tests、QEMU、刷写或真板实验。Rt-Async-AMP 根 workspace 的 offline Cargo metadata 因缺少 `rt-async/modules/platform/Cargo.toml` 退出 101；这证明依赖材料不完整，不是产品构建失败。（`未知项`）

## 10. 与相邻主题的关系

- [`com260-boot-chain.md`](../boot/com260-boot-chain.md)：K3 官方启动阶段、介质和版本边界。
- [`com260-mailbox-notification.md`](../interrupts/com260-mailbox-notification.md)：mailbox4 静态配置、FIFO/pending、IRQ 和等待风险。
- [`k3-dma-and-memory-ownership.md`](../dma/k3-dma-and-memory-ownership.md)：共享内存与设备 DMA 的对象边界。
- [`k3-cache-pma-address-translation.md`](../dma/k3-cache-pma-address-translation.md)：cache、fence、PMA/PBMT 和 IOMMU 机制。
- [R09 启动分析](../../.claude/analysis/rt-async-amp-k3-boot-platform.md) 与 [R10 共享内存分析](../../.claude/analysis/rt-async-amp-k3-shared-memory.md)：固定 revision 调查入口；正文已经写入执行所需边界，不要求读者回读分析才能理解生命周期。
- `k3-rpc-ring-notification.md`：计划在下一 Iteration 承载 ring、BUSY、doorbell、RPC、等待和错误恢复，不在本文预写。
