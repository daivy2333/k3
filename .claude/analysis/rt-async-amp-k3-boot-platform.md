# Rt-Async-AMP 的 K3 启动与板级适配

> Snapshot: [SNAPSHOT](../docs/SNAPSHOT.md)
> Captured revision: k3 `573162934e6ebdb6fe5d254c09cd29a922231e15`；Rt-Async-AMP `ccb1ff0b487e4f49ea570c41f330741eecece935`；tgoskits `19219411d5dc1515496f910d04c93da12ee95be4`
> Observed branch: k3 `main`；Rt-Async-AMP `master`；tgoskits `feat/rt-async-amp`
> Captured at: 2026-09-08
> See also: [共享内存与通知](rt-async-amp-k3-shared-memory.md)、[驱动边界](rt-async-amp-k3-drivers.md)、[复用清单与缺失仓库](rt-async-amp-k3-starryos-reuse.md)

## 结论与范围

这里能提取两条独立镜像链的配置、RT24 握手实现、AP FIT 装载地址和 AP 侧 DTS 框架。补齐后的 `tgoskits` 证明 StarryOS 内核装载到 `0x140000000`、DTB 装载到 `0x138000000`，但这是该第三方分支的实现值，不是本项目目标板的官方事实。

本文回答：镜像如何构建和交接，哪些地址属于 RP，哪些资料能帮助我们建立 AP 启动基线，以及缺失源码阻断了什么。范围是用户指定的第三方实现研究。下文“源码确认”指当前文件行为；“仓库声称”指注释或 README，均不升级为本项目 D06 的“官方事实”。

## 镜像和调用链

```text
envs/k3-com260.toml
  └─ xtask::build::build_env
       ├─ opensbi_k3 → make → opensbi.itb       AP M 态固件
       ├─ rt-async-k3 bins → ELF → pack_itb → esos.itb
       │                                      rcpu0 占位 + rcpu1 RTOS
       └─ starryos → tgoskits/tg-xtask → starryos.uimg
                         AP kernel @ 0x140000000 + DTB @ 0x138000000
```

源码入口：[build.rs:291/338/412/453](../../others/Rt-Async-AMP/xtask/src/build.rs)，函数 `opensbi_k3`、`starryos`、环境聚合及 `pack_itb`；环境配置见 [k3-com260.toml:1](../../others/Rt-Async-AMP/envs/k3-com260.toml)。实际聚合执行三条链，README 局部“两个产物”的表述漏了 OpenSBI。

| 对象 | 已确认的配置或行为 | 对我们工作的边界 |
| --- | --- | --- |
| AP 构建入口 | `tgoskits/os/StarryOS/configs/board/spacemitk3-com260kit.toml`，调用 `cargo run --release -p tg-xtask -- starry build --config …` | 配置显式开启 K3 UART、UFS、GMAC、pinctrl 和 Zicbom；仍需在我们的构建系统中重新映射 |
| AP 产物 | ITS 把 kernel 放到 `0x140000000`，DTB 放到 `0x138000000`，并生成 `starryos.uimg` | 这与 README 用 `0x180000000` 上传整个 FIT 不冲突；上传缓冲地址不等于 FIT 内部 load address |
| AP SMP | K3 profile 没有 `smp=1`；构建函数只在 profile 指定时传 `--smp` | 构建函数旁“hart1 归 RP，AP 必须单核”是 QEMU 场景注释，不能覆盖 K3 profile |
| OpenSBI | `PLATFORM=generic`、`PLATFORM_DEFCONFIG=k3_defconfig`、`FW_PIC=y`、`CROSS_COMPILE=riscv64-linux-gnu-` | 必须核对工具链、固件和板上现状；不能拿 QEMU 固件替代 |
| RP | 自定义 `riscv64imac-k3-none-elf.json`，`-Zbuild-std=core`；链接依赖 rt-async 平台的 `link.x` | `link.x` 所在子模块缺失；自定义 target 是 RP 用途 |

AP 构建会去掉继承的 `RUSTUP_TOOLCHAIN`，让子仓按自身配置选择工具链；这是可复用的跨仓构建注意事项，不要求我们引入同一构建系统。

## AP 侧板级配置

[K3 board profile](../../others/Rt-Async-AMP/tgoskits/os/StarryOS/configs/board/spacemitk3-com260kit.toml) 使用 `riscv64gc-unknown-none-elf`，开启 `k3_com260kit`、K3 PXA UART、UFS、GMAC、pinctrl 以及平台/驱动两层 Zicbom。[ITS](../../others/Rt-Async-AMP/tgoskits/os/StarryOS/configs/board/spacemitk3-com260kit.its) 嵌入 `spacemit-k3-com260-ifx.dtb`，未嵌入 rootfs；DTS 的 `chosen.bootargs` 要求 `root=PARTLABEL=rootfs rw`。因此该镜像链默认依赖板上另有 `rootfs` 分区，不是自包含 initramfs 的证据。

AP DTS 实际启用 serial0 `0xd4017000`、UFS `0xc0e00000`、GMAC1 `0xcac82000`、IMSIC `0xe0400000` 和 APLIC `0xe0804000`，并标记 `/soc` 为 `dma-noncoherent`。它还保留 rcpu0/rcpu1 内存，为 `0xc0800000..0xc0819000` 建立 `no-map` reserved-memory，再用 `ov,rt-async-amp` 节点把同一区域交给 `/dev/rt_shm`。这种“先从通用内存分配器排除，再由专用驱动映射”的结构可复用；节点 phandle、完整大 DTS 和板上 serial-number 不应原样拷贝。

## RP 地址与 DTB

[app build.rs:9–30](../../others/Rt-Async-AMP/apps/rt-async-k3/build.rs) 生成 `memory.x`：RAM 起点 `0x100804000`、长度 `0x300000`、`ENTRY(__start)`、`_max_hart_id=0`、hart stack 8192 字节。DTS 却声明硬件 CPU `reg=<1>`，驱动取 FDT `boot_cpuid_phys` 算寄存器窗口。逻辑调度核编号与硬件 hart 编号如何衔接，需要补读 rt-async 启动汇编，不能凭 `_max_hart_id=0` 推断跑在 hart0。

[ITS 模板](../../others/Rt-Async-AMP/scripts/flash/esos_k3_com260_ifx.its) 的地址均为两个 cell，以下数值是源码求值结果：

| 节点 | load / entry | 内容 |
| --- | --- | --- |
| `rcpu0-fw` | `0x100200000` | rcpu0 ELF，LZO 压缩 |
| `rcpu1-fw` | `0x100804000` | rcpu1 ELF，LZO 压缩 |
| `rcpu0-dtb-com260_ifx` | `0x100f18000` | 固定 DTB |
| `rcpu1-dtb-com260_ifx` | `0x100f24800` | 固定 DTB |
| `rcpu-data-null` | `0x100e04000` | 固定 blob，LZO 压缩 |

模板顶部笼统称这些是“RT24 SRAM”地址，与 app build.rs 称 rcpu1 地址为 DDR 不一致。这里只保存镜像配置值，介质归属需要地址手册和 loader 源码确认。

源码中同时存在两种 DTB：ITS 随包携带的固定 DTB，以及 [chip build.rs:5](../../others/Rt-Async-AMP/modules/chip-k3-rt24/build.rs) 将 `its/rt-async-k3.dts` 经过 `cc -E → dtc` 编译后内嵌进 ELF 的 DTB。`K3Rt24::init` 明确使用后者。仓库声称 U-Boot `k3-rproc.c` 只搬运 ELF `PT_LOAD`、没有 DTB handoff；该 U-Boot 源码当前不在本地，所以加载者行为尚未独立核对。

固定 rcpu0 ELF 的实际 `readelf` 结果值得保留：入口为 `0x100200000`，但 LOAD 段起于 `0x1001ff000`，段长 `0x101c`。因此核对装载冲突时要检查整个段，不能只比较 entry 或 ITS 的 load 字段。实际执行地址仍需 loader 和板上证据。

## 握手、初始化和资源所有权

RP 入口宏和启动汇编缺失，能直接追到的链从 [K3Rt24::init，lib.rs:75](../../others/Rt-Async-AMP/modules/chip-k3-rt24/src/lib.rs) 开始：

1. `handshake::spl_handshake()` 写 `0xc088007c = 1`，然后开启 AON 探针计时器。
2. 注入内嵌 DTB，设置 `K3_DRIVERS`，调用缺失平台层的 `boot()`。
3. 注册 shutdown stub；`late_init()` 再调用 mailbox 中断配置。

[handshake.rs:30](../../others/Rt-Async-AMP/modules/chip-k3-rt24/src/handshake.rs) 的写入是 rcpu1 → CORE0；[rcpu0 占位源码:20](../../others/Rt-Async-AMP/scripts/flash/payloads/rt24_os0_rcpu.S) 则写 `0xc088008c = 1`、执行 fence 后永久 WFI。解锁 AP 最多六秒轮询属于仓库对 U-Boot 的说明，写寄存器动作可以直接确认。

当前打包脚本明确使用 rcpu0 占位 ELF，不能沿旧 DTS 注释继续声称当前 rcpu0 运行完整 ESOS。`esos` 是这里复用的分区与容器名称，不等于运行内容未替换。

## 刷写说明的适用边界

[README:152–196](../../others/Rt-Async-AMP/README.md) 给出的操作分为：RP 写 `esos` 分区、OpenSBI 写 `opensbi` 分区、AP FIT 上传 RAM 后 `bootm`。AP 示例上传地址为 `0x180000000`、缓冲大小 `0x04000000`，stage 后需要 Ctrl+C 返回 U-Boot。FIT 内的 kernel/DTB 最终装载地址已由 tgoskits ITS 补齐。

这些是原仓库的操作说明，本次未执行。`0x180000000` 是上传 FIT 的示例地址，不是已确认的 AP kernel load address。需要补齐 AP ITS、当前板上 DRAM/保留区、MTD 分区和固件版本后才能生成面向我们板子的操作契约。打包脚本会复制 ELF 并在源树 `payloads/` 下生成压缩文件，本次只做语法检查。

## 验证与缺失材料

| 检查 | 本次结果 | 能支持的结论 |
| --- | --- | --- |
| `bash -n others/Rt-Async-AMP/scripts/flash/k3-pack-itb.sh` | 退出 0 | shell 语法可解析，不代表打包成功 |
| `cc -E -P -nostdinc -undef -x assembler-with-cpp -I others/Rt-Async-AMP/its/ others/Rt-Async-AMP/its/rt-async-k3.dts` | 退出 0；展开 GPIO122、123、83 配置表达式 | include 和宏预处理可用，未完成 dtc 阶段 |
| `readelf -h -l others/Rt-Async-AMP/scripts/flash/payloads/rt24_os0_rcpu.elf` | 退出 0；ELF64 / RISC-V；entry `0x100200000`；LOAD `0x1001ff000` | 固定 ELF 布局可读取 |
| tgoskits workspace metadata | `cargo +stable metadata --offline --no-deps` 退出 0 | 当前分支的 workspace 和 path dependency 可解析，不代表 K3 target 可编译 |
| 构建 / DTB 编译 / 真板启动 | SKIPPED：`rt-async`、`ov-channels` 仍缺失，OpenSBI 路径错位，`dtc` 与指定 nightly 未安装；未连接真板 | 不声明 boot、SMP 或驱动通过 |

补充优先级：先 `rt-async` 的启动汇编、链接脚本和注册框架，以及 `ov-channels` 的 ring 布局/内存序；再核对原工程使用的 U-Boot `k3-rproc.c` 与 SRAM 清理路径。精确仓库地址和 gitlink 提交见[补充清单](rt-async-amp-k3-starryos-reuse.md#需要补充的仓库和材料)。

## 与当前 change 的关系

活跃 `establish-k3-com260-board-boot-baseline` 的 Iteration 001 仍在当前 Cycle 修复阶段。本分析为其 boot/镜像/DTS 调查提供第三方线索，不关闭 G7、不选择唯一 Kit DTS、不代替原 Cycle 的来源等级和 fresh evidence。尤其不能把 `com260_ifx` 模板直接解释成我们目标 Kit 的唯一配置。
