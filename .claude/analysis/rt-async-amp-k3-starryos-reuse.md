# Rt-Async-AMP 面向 StarryOS 的 K3 复用清单

> Snapshot: [SNAPSHOT](../docs/SNAPSHOT.md)
> Captured revision: k3 `573162934e6ebdb6fe5d254c09cd29a922231e15`；Rt-Async-AMP `ccb1ff0b487e4f49ea570c41f330741eecece935`；tgoskits `19219411d5dc1515496f910d04c93da12ee95be4`
> Observed branch: k3 `main`；Rt-Async-AMP `master`；tgoskits `feat/rt-async-amp`；对照 StarryOS `linshi` @ `6fcc602de48217a125d40f1f34635138464a2b9e`
> Captured at: 2026-09-08
> See also: [启动与板级](rt-async-amp-k3-boot-platform.md)、[共享内存与通知](rt-async-amp-k3-shared-memory.md)、[驱动与异步边界](rt-async-amp-k3-drivers.md)、[原有官方资料分析](k3-official-docs-for-starryos-async-drivers.md)

## 结论

本次识别出十组有用材料。优先级最高的是 AP APLIC/IMSIC、中断驱动的 PXA UART、GMAC DMA/IRQ 队列、PMA+SRAM 生命周期以及 AP/RP 镜像链；UFS 的初始化和恢复也有价值，但其运行时仍是同步轮询。这些都是调查输入，不是已经在我们 StarryOS 上验证的移植补丁。

`tgoskits` 已在主仓 gitlink 要求的提交和 `feat/rt-async-amp` 分支，不再是阻塞项。仍缺 `rt-async` 与 `ov-channels`；OpenSBI 所需提交已在本地，但目录多嵌套一层。由于本次没有做目标构建和真板测试，仍不宜用代码行数表达“可直接复用百分比”。

用户已批准四主题文档探索，并要求记录需补充的仓库。本次仅创建分析和 R 索引；保留原仓库、现有产品文档与活跃 change 的状态。

## 十组复用材料

“可读”表示相关实现或测试文件存在，不表示构建、真板或移植验收通过。编号只是本分析内分类，不是全局 task。

| 组 | 资料 | 现有程度 | 给 StarryOS 的价值 | 对应分析 |
| --- | --- | --- | --- | --- |
| 1 | 镜像、RP handshake、内嵌 DTS 与打包 | 可读 | 分清 AP/RP 镜像、上传与装载地址，核对完整 PT_LOAD | [启动](rt-async-amp-k3-boot-platform.md) |
| 2 | X100 OpenSBI PMA 补丁 | 可读 | 按地址找 entry、初始化前清 cache、逐 hart 设置；需验证板上实效 | [共享内存](rt-async-amp-k3-shared-memory.md) |
| 3 | mailbox MMIO、ISR 和门铃 | 可读 | 收发分通道、先排 FIFO 再清 pending、IRQ 与队列分离 | [共享内存](rt-async-amp-k3-shared-memory.md) |
| 4 | PXA UART、pinctrl、CCU/APBC | 可读 | 硬件访问、初始化顺序、引脚所有权、THRE/TEMT；需适配 async 层 | [驱动](rt-async-amp-k3-drivers.md) |
| 5 | RT24 PLIC、SysTimer、AON | 可读 | 避免 AP/RP 混用；保存特殊 hart stride 和 reset 差异 | [驱动](rt-async-amp-k3-drivers.md) |
| 6 | IPC/PBMT/延迟测试入口 | 可读 | 行为测试拆分、错误/大响应边界、分开 IRQ 和 spin 性能 | 本文及[共享内存](rt-async-amp-k3-shared-memory.md) |
| 7 | SRAM 所有权、RPC、BUSY 和恢复 | 部分 | AP init/mmap/poll 已闭合，ring 原子与 RP IrqLatch 仍缺 | [共享内存](rt-async-amp-k3-shared-memory.md) |
| 8 | AP APLIC/IMSIC 和 rt_shm | 可读 | FDT IRQ domain、EID/MSI 路由、stopei claim/complete、mailbox 自测 | [驱动](rt-async-amp-k3-drivers.md) |
| 9 | AP GMAC/PHY/DMA | 可读 | DWMAC5、RGMII phase、MDIO、非一致 DMA ring、IRQ 唤醒 | [驱动](rt-async-amp-k3-drivers.md) |
| 10 | AP UFS | 可读 | MPHY/UniPro、UTP/SCSI、DMA 维护、fatal/timeout recovery；需异步化 | [驱动](rt-async-amp-k3-drivers.md) |

已发现 K3 GMAC/PHY 和 DMA descriptor 数据面，但未发现 IOMMU driver；GMAC 是“高可复用性参考”，不是“可不加验证地整体复制”。机器人 Python/ESP32/舵机业务不在当前 StarryOS 板级目标内，只检查其调用 UART/RPC 的接口，未展开业务算法。

若以“能不能显著缩短 StarryOS 实现与验证”而非代码行数计算：十组中有 5 组是高价值实现参考（PMA、mailbox、AP PXA/pinctrl、AIA、GMAC），4 组是带条件复用（镜像链、测试入口、RPC/共享内存恢复、UFS），1 组主要用于防止误用（RT24 PLIC/timer 与 AP 域的分界）。这是“资料组覆盖”，不是“可直接复制 50% 代码”。

## 与当前仓库和 StarryOS 的关系

本项目仍以官方 K3 资料为硬件事实权威。用户此次明确指定第三方仓库，故成果作为实现研究保存在 `.claude/analysis/`，不自动写入 `docs/`、不改变 M01/D06 的来源等级。

全局 tasks 尚未反映活跃 MS03 change 的进度。本次 `openspec list` 及其任务文件显示 `establish-k3-com260-board-boot-baseline` 为 6/13：Iteration 000 accepted；Iteration 001 / `000-initial` 为 ready/reported/pending，仍有五类 Review 阻塞。本批资料可支持 boot、memory、console 的后续调查，但不替代其当前修复契约，也不证明目标 Kit 唯一对应 IFX DTS。

对我们相邻 StarryOS 做了有限源码对照：平台选择尚无 K3，`InterruptConfig` 只承载 PLIC；UART 已有 async `UartPort` 集成，网络已有 `NetQueueControl`、TX 和 recovery 接口。因此应该复用 tgoskits 的 K3 硬件层、AIA 域模型、GMAC ring/cache 操作和 PXA IRQ errata 处理，同时保留我们已有的 copier/waker/drain/recovery 契约。详细源码位置见[驱动分析](rt-async-amp-k3-drivers.md#与我们当前-starryos-的接口对应)。

| 本项目 milestone | 可提供的输入 | 仍然不能解决 |
| --- | --- | --- |
| MS03 板级与启动 | 双镜像、RP 入口/握手、AP FIT load/entry、IFX DTS | 唯一目标 Kit 映射、官方 DRAM/保留区确认 |
| MS04 平台与串口 | AP PXA IRQ endpoint、u32 MMIO、UUE/OUT2、pinctrl provider | 我们 AP console 实例的官方确认和真板 async RX/TX |
| MS05 中断与时间 | AP APLIC/IMSIC domain、EID/MSI 路由和 stopei 语义 | SMP affinity/balancing，timer 仍需单独闭环 |
| MS06 DMA/一致性 | PMA/SRAM alias、GMAC/UFS descriptor 与 Zicbom 维护 | IOMMU、官方 coherency 范围、真板 ownership 证据 |
| MS07 GMAC/异步网络 | DWMAC5、RGMII phase、MDIO/PHY、DMA ring、IRQ/Event 队列 | 在我们框架中的防丢唤醒、性能与 recovery 验证 |

## 需要补充的仓库和材料

地址与提交直接来自 [.gitmodules](../../others/Rt-Async-AMP/.gitmodules) 和 `git submodule status`。下面的提交用于恢复原项目依赖现场，不作为硬件验收、握手或运行身份机制。本次没有自动 clone、移动目录或改 gitlink。

| 优先级 / 路径 | 仓库地址 | 主仓要求的提交 | 本次状态与补齐后检查内容 |
| --- | --- | --- | --- |
| 已补齐 `tgoskits/` | `https://github.com/Oveln/tgoskits.git` | `19219411d5dc1515496f910d04c93da12ee95be4` | `feat/rt-async-amp`，HEAD 与 gitlink 一致，工作树干净。已补读 AP TOML/ITS/DTS、AIA、rt_shm、PXA UART、pinctrl、GMAC 和 UFS；无需切换分支 |
| P1 `rt-async/` | `git@github.com:Oveln/rt-async.git` | `1ead7f09b88808ce79cab135044d8a0427261f6f` | 空目录。需要 `modules/platform`、riscv64-rt 启动/链接、driver boot、IrqLatch、executor/futures。解除 hart 编号、trap complete、注册后重检和 cs-atomics 审查缺口 |
| P1 `modules/ov-channels/` | `git@github.com:Oveln/ov-channels.git` | `50c3119073ee7e0f98f89037d54d891f97a3a84c` | 空目录。需要 SharedMemory、RingBuffer、load/store ordering、多块发布、busy、reset；核对 `0x18700` footprint 及零地址处理。主仓 path patch 要求本地源码，crates.io 同版本不保证相同 |
| 路径修正 `opensbi-k3/` | `git@github.com:Oveln/opensbi-spacemit.git` | `7a2df083ed06373c506e2e6f4e09bbd168202f2d` | 同提交已在 `opensbi-k3/opensbi-spacemit/`，分支 `feat/pma-audio-io`、干净工作树；不缺补丁内容。xtask 检查的是 `opensbi-k3/.git` 并在该目录 make，目前找不到，后续需要整理目录或按主仓子模块方式恢复 |

额外需要的材料没有主仓 gitlink 可用，不猜仓库地址：

- 原项目实际使用的 U-Boot K3 分支：`drivers/remoteproc/k3-rproc.c`、`k3_clear_sram()`、board late init 与 bootm 交接；核对六秒握手、SRAM 清理与 DTB handoff。
- 注释引用的 K3 programmer 手册及 CoM260 Kit v02 原理图：补齐文件名、版本和相关页，核对引脚、电平、可达域和时钟定义。
- 原始 K3 运行记录：启动串口、PMA entry 读回、双向共享内存读写、重复 IRQ/清除和当前固件性能；仓库只提供部分 README 表格、源码注释和测试入口，没有找到 `.log`/`.csv`/`.pdf` 文件。

补齐 `rt-async` 和 `ov-channels` 后可继续接通 RP 启动/原子/ring/唤醒调用链，无需重做已保存的主仓与 tgoskits 调查。若提供不同提交，应明确差异，重新判断对应结论的新鲜度。

## 已实际运行的检查

| 命令或操作 | 决定性输出 / 结果 | 结论 |
| --- | --- | --- |
| `git -C others/Rt-Async-AMP submodule status` | 退出0；四项仍都显示 `-` | 主仓的 submodule metadata 未初始化；`tgoskits` 和 OpenSBI 是各自独立克隆到预期路径/嵌套路径，需分别核对 |
| tgoskits `rev-parse HEAD`、`branch --show-current`、`status --short` | `19219411…` / `feat/rt-async-amp` / 无输出 | 分支和提交正确，工作树干净 |
| `cargo +stable metadata --offline --no-deps --format-version 1` （tgoskits） | 退出0 | workspace/path dependency 可解析，不代表 K3 target build 通过 |
| 嵌套 OpenSBI `rev-parse HEAD`、`branch --show-current`、`status --short` | 退出0；`7a2df083…` / `feat/pma-audio-io` / 无输出 | 可以研究对应补丁；目录与构建入口不符 |
| `cargo +stable metadata --offline --no-deps --format-version 1`（Rt-Async-AMP 根） | 退出101；`failed to read …/rt-async/modules/platform/Cargo.toml`，`No such file or directory` | workspace 解析被缺失源码阻断；这不是产品编译失败，也未替换项目工具链 |
| K3 DTS 的 `cc -E` 预处理 | 退出0，GPIO122/123/83 宏展开 | 只证明预处理；`dtc` 不在 PATH，后半段未跑 |
| `bash -n …/k3-pack-itb.sh` | 退出0 | 只证明 shell 语法 |
| `readelf -h -l …/rt24_os0_rcpu.elf` | 退出0；entry `0x100200000`，LOAD `0x1001ff000` | 应按 PT_LOAD 范围检查装载冲突 |
| `rustup toolchain list` | 未列 `nightly-2026-04-25` | 指定 RP 工具链尚缺，本次未安装 |

构建、host tests、QEMU、刷写、真板 IRQ/数据面/负载均未执行。仓库包含测试不等于测试通过，文件自称“已实测”不等于本次已验证。

## 可复用的验证入口

| 入口 | 已阅读的行为 | 必要前置与限制 |
| --- | --- | --- |
| [ov-rpc/tests/rpc.rs](../../others/Rt-Async-AMP/modules/ov-rpc/tests/rpc.rs) | call/poll/send/urgent、BUSY 跳门铃、顺序和响应关联 | 缺 ov-channels；host 内存测试不证明 SRAM/MMIO |
| [error_paths.rs](../../others/Rt-Async-AMP/modules/ov-rpc/tests/error_paths.rs) | 反序列化失败、未知方法、超长响应、Deferred、FIFO 后续消息 | 直接 poll，没覆盖错误响应唤醒已睡眠 AP |
| [large_response.rs](../../others/Rt-Async-AMP/modules/ov-rpc/tests/large_response.rs) | 255/256 边界、多块、超长与字节校验 | 缺 ring 多块实现；需双方版本一致 |
| [discovery.rs](../../others/Rt-Async-AMP/modules/ov-rpc/tests/discovery.rs) | 描述符和服务表、单块/多块发现、与普通调用交织 | 服务发现是 RPC 功能，不应扩展成运行身份验证框架 |
| [user-test-ipc](../../others/Rt-Async-AMP/user-apps/user-test-ipc/src/main.rs) + [shm_ping](../../others/Rt-Async-AMP/apps/rt-async-k3/src/bin/shm_ping.rs) | 通知回显、ADD 结果、NOTIFY/AWAIT | 需要 AP rt_shm、可用窗口和 mailbox；等待路径仍需独立 deadline 检查 |
| [user-test-pbmt](../../others/Rt-Async-AMP/user-apps/user-test-pbmt/src/main.rs) + [pbmt_probe](../../others/Rt-Async-AMP/apps/rt-async-k3/src/bin/pbmt_probe.rs) | 写可见性、RP 活性、读计数前进、时延对照 | 需要专用配对固件；不能与占用同 scratch 的业务同时跑 |
| [user-test-bench](../../others/Rt-Async-AMP/user-apps/user-test-bench/src/main.rs) / [rtbench](../../others/Rt-Async-AMP/apps/rt-async-k3/src/bin/rtbench.rs) | 文件与命令入口已定位；PING 分段在 intercom 中核对 | 未逐项审计全部微基准；只提取 IRQ/spin 分桶方法，不采用历史延迟承诺 |

建议先证明硬件访问与双向数据，再证明连续通知与睡眠边界，最后测延迟。README:442 的 user-cbo 历史对比表已由作者标明不再对应当前 PMA 固件，不能用于预估本次复用收益。

## 待核实的风险与信息冲突

详细代码位置在前三篇，这里只列会影响选择的事项：

1. 当前 rcpu0 打包的是握手占位，部分 DTS/mailbox 注释仍按完整 ESOS 占用资源解释。
2. SRAM AP 初始化策略与 RP 看门狗早期 re-init 并存；magic 修复会丢未读消息。
3. RPC poison 已入环时不一定发门铃；host 解码测试未证明阻塞端及时失败。
4. RP PXA UART 是轮询，RP 借用 AP UART5 时超时可能丢失已发送长度语义；tgoskits AP PXA 已有 IRQ endpoint，但仍需接入我们的 copier/drain 契约。
5. `atomic-cas=false`/本地关中断不能移植为 AP SMP 同步策略；零地址 alias 还需审查 Rust 引用有效性。
6. PMA 找不到覆盖 entry 时继续启动，且修改整个 entry；必须核对实际范围和读写行为。
7. 引脚“自愈”会重写跨核共享 pinmux；应先明确资源所有权。
8. GMAC IRQ handler 抢锁失败时只等下一个中断；UFS 虽解析 IRQ 却全程轮询。两者都需按 StarryOS 的异步推进契约重做压力验证。

## 交付与后续边界

四个批准主题均已回答或给出缺失原因；本批文档的 R 登记由 Explorer 限定调用 Maintainer 完成。前两轮技能不隐含修复第三方代码或当前活跃 change 的授权。

未自动登记的 I 候选：共享窗初始化所有权、RPC 错误响应通知、跨核 pinmux 所有权；证据在共享内存和驱动分析中。它们是第三方实现复用前的检查项，尚未承诺为本项目实施任务。没有把作者注释的板上结果登记为本项目 K。
