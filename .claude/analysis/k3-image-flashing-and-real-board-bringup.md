# K3 CoM260 镜像、烧录与真板启动路径

> Snapshot: [SNAPSHOT](../docs/SNAPSHOT.md)
> Captured revision: k3 `820535c0bab7b2e58df3c1c01bc6a9e2689ba4a9`；Rt-Async-AMP `ccb1ff0b487e4f49ea570c41f330741eecece935`；tgoskits `19219411d5dc1515496f910d04c93da12ee95be4`
> Observed branch: k3 `main`；Rt-Async-AMP `master`；tgoskits `feat/rt-async-amp`
> Captured at: 2026-09-11
> See also: [链接缺口与存储来源评估](k3-link-gap-and-storage-source-assessment.md)、[Rt-Async-AMP 启动与板级适配](rt-async-amp-k3-boot-platform.md)、[StarryOS 复用清单](rt-async-amp-k3-starryos-reuse.md)

## 结论

当前证据支持三条不同成熟度的 K3 CoM260 部署路径：

1. **U-Boot RAM 临时引导**：把 `starryos.uimg` 上传到 `0x180000000`，退出 Fastboot 后执行 `bootm 0x180000000`。该路径不写持久介质，是首轮 StarryOS bring-up 的优先路径。
2. **U-Boot 分区更新**：把 `esos.itb` 或 `opensbi.itb` stage 到 RAM，再执行 `mtd erase` / `mtd write` 更新同名分区。该路径会改变持久固件，只能在分区、备份和恢复入口均已确认后执行。
3. **Titan ZIP 或 SD 卡整盘镜像**：官方文档确认 K3 Buildroot ZIP 可交给 Titan，也可解压后用 Fastboot；但当前仓库没有已核对的 Titan 配置、K3-Ubuntu-Images 分区文件或可直接生成 SD 卡镜像的脚本，因此尚不能形成安全的逐命令操作契约。

首轮实践应保留出厂 FSBL、ESOS、OpenSBI 和 U-Boot，只替换 RAM 中的 AP payload。看到 StarryOS 首字节后，再验证 DTB、内存、串口和异常入口。只有 AMP 共享窗确实需要定制 PMA 行为时，才进入 OpenSBI 和 ESOS 持久更新。

## 目标与范围

本分析覆盖五个主题：

- 官方 SDK 产物及 K3 启动链；
- StarryOS FIT 格式、加载地址、入口和 DTB 交接；
- RAM 引导、持久分区、Titan ZIP 和 SD 卡四类部署方式；
- FORCE_RECOVERY/FEL、串口、失败恢复和防误刷边界；
- 从镜像加载到完整 workload 的真板分层验证。

本文是后续 Plan 和 Runbook 的调查输入，不授权执行烧录。所有板上命令均未在本次会话执行；持久写命令只记录已存在的第三方流程，不能直接复制到目标板执行。

## 1. 官方启动链与产物责任

现有官方资料基线给出的 local boot 链为：

```text
Boot ROM
  → bootinfo 定位 FSBL/SPL
  → FSBL 初始化 DDR
  → 加载 ESOS、OpenSBI、U-Boot
  → OpenSBI 启动 U-Boot
  → U-Boot 加载 payload/OS
```

| 产物 | 已确认责任 | 当前边界 |
|---|---|---|
| `bootinfo_*.bin` | 写入介质，供 Boot ROM 定位 FSBL | `bootinfo_*.json` 是构建配置，不能代替 `.bin` 烧录 |
| `FSBL.bin` | DDR 初始化并加载后续固件 | 当前没有面向本板的重建、备份和恢复验证 |
| `esos.itb` | K3 特有 ESOS/RCPU 容器 | Rt-Async-AMP 用自制握手占位固件和 RT24 payload 替换部分内容，不能视为官方 ESOS |
| `fw_dynamic.itb` / `opensbi.itb` | M-mode 固件和 SBI 服务 | 文件名来自两条不同构建链；落盘前必须核对 FIT 内容和目标分区 |
| `u-boot.itb` | U-Boot Shell、Fastboot 和 OS 加载 | 首轮 bring-up 应保留板上已工作的版本 |
| `bootfs` / `rootfs` | Linux payload 和用户空间 | StarryOS FIT 本身不等同于完整 rootfs 镜像 |

官方 Buildroot 文档已经证明 ZIP/Titan/Fastboot 关系和固件分区模型，但没有证明某个通用分区 JSON 就是当前 CoM260 Kit 的唯一正确布局。`partition_universal.json`、`partition_4M.json` 和 `bootinfo` 示例在未与当前板型、介质和容量核对前不能使用。

## 2. 当前 StarryOS / Rt-Async-AMP 产物链

本地实现的聚合入口为：

```text
cargo xtask build k3-com260
  ├─ opensbi_k3() → opensbi.itb
  ├─ build_rt_async() + k3-pack-itb.sh → esos.itb
  └─ tg-xtask + spacemitk3-com260kit.its → starryos.uimg
```

关键入口：

- [`envs/k3-com260.toml`](../../others/Rt-Async-AMP/envs/k3-com260.toml) 选择 K3 环境、StarryOS board config 和默认 RT24 payload。
- [`xtask/src/build.rs`](../../others/Rt-Async-AMP/xtask/src/build.rs) 的 `opensbi_k3`、`starryos`、`build_env`、`pack_itb` 组织三条构建链。
- [`k3-pack-itb.sh`](../../others/Rt-Async-AMP/scripts/flash/k3-pack-itb.sh) 压缩固定 payload 与新 RT24 ELF，再生成 `esos.itb`。
- [`spacemitk3-com260kit.its`](../../others/Rt-Async-AMP/tgoskits/os/StarryOS/configs/board/spacemitk3-com260kit.its) 定义 AP kernel 和 DTB 的 FIT 布局。

### AP FIT

`starryos.uimg` 内部配置为：

| 对象 | FIT load/entry | 说明 |
|---|---:|---|
| StarryOS kernel | `0x140000000` / `0x140000000` | AP kernel 的最终装载和入口地址 |
| DTB | `0x138000000` / 无 entry | 使用固定 `spacemit-k3-com260-ifx.dtb` |
| 上传缓冲区 | `0x180000000` | README 的 Fastboot stage / `bootm` 地址，不是 kernel 最终入口 |
| 最大 stage 大小 | `0x04000000` | 第三方 README 示例值；执行前需检查实际 FIT 大小和内存占用 |

FIT 使用 `type = "kernel"`、`arch = "riscv"`、`os = "linux"` 和 SHA-256 hash。`os = "linux"` 是 U-Boot FIT 元数据，不证明 payload 是 Linux。

固定 IFX DTB 与当前 Kit 的唯一映射尚未解决。RAM 引导前至少需要核对 DTB model/compatible、UART、内存、reserved-memory、PLIC/APLIC/IMSIC 以及 chosen 节点；不能只因 `bootm` 能解析 FIT 就视为板级匹配。

### RP/ESOS FIT

`esos_k3_com260_ifx.its` 包含 rcpu0/rcpu1 固件、两个 DTB 和交互 blob。当前 rcpu0 是写握手寄存器后 WFI 的占位 ELF，rcpu1 是构建得到的 rt-async ELF。固定 rcpu0 ELF 的实际入口为 `0x100200000`，LOAD 段从 `0x1001ff000` 开始；冲突检查必须覆盖完整 LOAD 段，不能只比较 entry。

这条链服务于 AMP/RP，不是启动 AP StarryOS 首字节的必要条件。首轮 AP bring-up 不应为了“产物齐全”先刷 `esos`。

## 3. 实际动手路径

### 3.1 基线保护

任何镜像操作前先完成以下记录：

- 精确板型、模组容量、当前启动介质和供电方式；
- 串口 `115200 8N1` 的完整出厂启动日志；
- OpenSBI、U-Boot、DTS model、内存和存储识别输出；
- U-Boot 中实际存在的 `mtd list`、分区名、环境变量和 Fastboot 命令；
- FORCE_RECOVERY/FEL + RESET 是否能重新进入下载模式；
- 官方恢复镜像、Titan 版本及其适用板型。

R18 已证明目标板能从 BootROM 启动到 Bianbu 4.0.1 root shell，串口物理链可用。它没有验证 Fastboot、Titan、分区备份或自编译镜像。

### 3.2 构建前检查

本次环境探测结果：

| 能力 | 结果 |
|---|---|
| Cargo / Rust | `cargo 1.90.0`、`rustc 1.90.0` |
| Linux RISC-V toolchain | `riscv64-linux-gnu-gcc 11.4.0` |
| Bare-metal toolchain | `riscv64-unknown-elf-gcc 15.1.0` |
| `dtc` | 缺失 |
| `mkimage` | 缺失 |
| `lzop` | 缺失 |
| `fastboot` | 缺失 |
| USB 枚举 | `lsusb` 无法初始化 libusb，退出 99 |
| `build/k3-com260/` 产物 | 不存在 |

因此当前主机不能完成 K3 聚合构建或 Fastboot 上传。补齐工具后仍需按仓库锁定的 Rust/toolchain 要求重新验证，不能由系统版本存在推定构建可用。

安全的构建验证顺序是：

1. 运行只读依赖和 submodule 检查。
2. 构建 `starryos.uimg`，暂不构建或刷写 OpenSBI/ESOS。
3. 用 `mkimage -l` 检查 FIT 的 kernel、FDT、hash、load 和 entry。
4. 用 `stat` 检查 FIT 大小小于经板上确认的 stage 缓冲区。
5. 保存构建命令、板级配置名和决定性输出；不以文件存在替代 FIT 检查。

### 3.3 首选：RAM 临时引导

该路径的目标是证明 U-Boot 能加载 FIT，并获得 StarryOS 第一字节，不修改 UFS、eMMC、SPI-NOR 或 MTD 分区。

概念流程：

```text
已工作的 local boot
  → 串口长按 s 进入 U-Boot Shell
  → U-Boot 执行 fastboot stage 接收 FIT
  → 主机上传 starryos.uimg
  → 串口 Ctrl+C 返回 U-Boot
  → 检查上传地址/大小与 FIT
  → bootm 0x180000000
  → 观察首字节和异常
  → 复位回到出厂启动链
```

第三方 README 给出的命令角色如下；它们尚未在本板复核：

```text
U-Boot: fastboot -l 0x180000000 -s 0x04000000 usb 0
Host:   fastboot stage build/k3-com260/starryos.uimg
U-Boot: Ctrl+C
U-Boot: bootm 0x180000000
```

执行前的停止条件：

- U-Boot 不识别相同的 `fastboot -l/-s ... usb 0` 语法；
- `fastboot devices` 未唯一识别目标设备；
- FIT 大小、上传地址或最终 load 范围与 DRAM/reserved-memory 冲突；
- DTB 不是已确认的目标板变体；
- 串口基线无法恢复。

RAM 引导成功只说明 U-Boot 已解析并跳转。真正的 Boot image gate 至少还要记录 FIT 大小、上传地址、kernel/DTB load、entry、boot 命令和 U-Boot 输出。

### 3.4 持久更新 ESOS/OpenSBI

本地 README 存在以下流程：Fastboot stage 到 `$loadaddr`，然后执行 `mtd erase esos|opensbi` 和 `mtd write ...`。这些命令具有破坏性，本分析不把它们提升为可执行步骤。

进入持久更新前必须具备：

- 板上 `mtd list` 证明分区名、介质、大小和边界；
- 旧分区的可恢复备份及离板校验；
- FORCE_RECOVERY/FEL 路径已实测，主机能识别设备；
- 官方恢复包和适配的 Titan/CLI 工具可用；
- 新 FIT 经 `mkimage -l` 检查且大小不超过目标分区；
- 已明确一次只改一个组件，并定义复位后的串口判据。

建议顺序是 AP RAM boot → 必要时只更新 OpenSBI → 验证官方 local boot 和 StarryOS RAM boot均可恢复 → 最后才考虑 ESOS/RP。FSBL、bootinfo、GPT 和 U-Boot 不属于首轮修改对象。

### 3.5 Titan ZIP

官方交叉验证资料确认 K3 Buildroot ZIP 可交给 Titan Flasher，也可解压后使用 Fastboot。R21 登记了 Titan 手册与 K3-Ubuntu-Images，但本次无法访问网页正文，当前仓库也没有 Titan 项目文件。

在核对下列材料之前，不能编写“选择 ZIP 后一键烧录”的 Runbook：

- Titan Flasher 的当前版本、主机系统和驱动要求；
- CoM260 Kit 进入下载模式的准确按键时序和 USB 身份；
- ZIP manifest、分区 JSON、镜像命名与校验要求；
- 全量刷写会覆盖的介质和分区；
- 中断烧录后的恢复路径。

Titan 适合作为恢复或量产路径，不应成为第一次验证自编译 kernel 的入口。

### 3.6 SD 卡镜像

官方事实证明 K3 支持 SD 启动介质，并存在与 eMMC/UFS 同类的 GPT 固件布局；这不等于当前已有可写入 SD 卡的 StarryOS 整盘镜像。

生成 SD 卡方案至少需要：

- 从 K3-Ubuntu-Images、Buildroot output 或 buildroot-ext 确认实际分区描述和生成器；
- 确认 CoM260 Kit 的 boot pin/按键能选择 SD，并实测启动优先级；
- 为 FSBL、ESOS、OpenSBI、U-Boot、bootfs/rootfs 或 StarryOS payload 分配明确落点；
- 核对 DTB、U-Boot env 和启动脚本；
- 在写卡前让操作者人工确认块设备路径、容量和可丢弃性。

当前没有证据支持给出 `dd`、分区或格式化命令。错误选择主机块设备会造成不可恢复的数据丢失，因此应由后续 Plan/Runbook 在实际镜像格式确定后单独设计。

## 4. 真板分层验证

| Gate | 最小目标 | 成功证据 | 失败时回退 |
|---|---|---|---|
| 0 出厂基线 | Bianbu local boot 可恢复 | R18 的 BootROM→root shell 日志 | 修复供电、串口或出厂介质，不测试新镜像 |
| 1 Boot image | U-Boot 接收并跳转 FIT | FIT 元数据、stage 地址/大小、`bootm` 输出 | 检查格式、load/entry、DTB 和命令 |
| 2 First byte | StarryOS polling UART 输出 | board、boot hart、DTB 来源、内存和 UART 范围 | 检查 UART base/width/stride、页表和早期栈 |
| 3 Platform facts | 读取并报告 hart、内存、IRQ 控制器、UART | 与目标 DTB 和既有 K3 文档逐项一致 | 停止加载驱动，修正 DTB/平台配置 |
| 4 Register access | UART/目标设备寄存器不是全 0/全 1 | 原值、写回读值和异常状态 | 检查 MMIO、clock、reset、权限和访问宽度 |
| 5 Interrupt | 重复进入、清设备状态、EOI/complete | 至少两次可解释中断 | 分离 claim、handler、clear、EOI |
| 6 Minimal function | 内核最小操作 | UART RX/TX/drain 或块设备队列移动 | 回退寄存器、IRQ、DMA/ownership |
| 7 Workload | shell/rootfs/用户程序 | 模式、payload、日志和退出结果 | 不把 rootfs 失败归因于底层驱动 |

首字节必须使用 polling early console，不依赖任务、IRQ、rootfs、堆或异步驱动。建议第一屏只打印板名、boot hart、DTB model/compatible、内存范围、UART MMIO 和当前 Gate。

## 5. 失败与恢复矩阵

| 症状 | 优先检查 | 禁止动作 |
|---|---|---|
| Fastboot 主机看不到设备 | USB 线/端口、下载模式、主机权限、libusb、唯一设备身份 | 不开始 MTD/GPT 写入 |
| stage 失败 | 工具版本、U-Boot 语法、缓冲区地址/大小、FIT 大小 | 不改用更大的任意地址盲试 |
| `bootm` 拒绝 FIT | `mkimage -l`、arch/os/type/hash、配置节点 | 不改写持久分区绕过错误 |
| 跳转后无输出 | entry、DTB、polling UART、MMIO、栈、页表 | 不先启用 IRQ/SMP/rootfs |
| 首字节后 fault | 重定位、页表、内存属性、reserved-memory | 不归因于串口驱动成功 |
| 官方系统也不再启动 | 最近一次持久修改、分区、环境和恢复入口 | 停止继续刷写其他组件 |
| 寄存器全 0/全 1 | 映射、clock/reset、宽度、权限、总线错误 | 不运行完整 workload |

每次只改变一个层级。一次 RAM boot 失败不应触发 ESOS/OpenSBI/U-Boot/GPT 的组合更新。

## 网站来源的当前状态

R21 的网站按用途分为 Titan、K3-Ubuntu-Images、U-Boot/OpenSBI、StarryOS/ArceOS、SDK 组成和工具链。它们能补齐未来操作契约，但本次网页工具没有返回正文，命令行访问 GitHub raw URL又因 DNS 解析失败退出 6。

因此本分析只使用已在项目中直接观察过的官方 boot/image 文档，以及本地固定 revision 源码。K3-Ubuntu-Images 的 `image_flash.py`、`fastboot.yaml`、`partition_universal.json`、`gadget.yaml` 等文件名仍是待核对线索，不作为已确认接口。

## 验证记录与影响面

本次执行的非破坏性检查：

- 完整读取现有 K3 boot/image 文档、R18/R19、R20 和 Rt-Async-AMP 启动分析；
- 追踪 `envs/k3-com260.toml` → `xtask::build_env` → OpenSBI/ESOS/StarryOS 三条构建链；
- 读取 AP/RP ITS、打包脚本和 README 的 K3 手动引导说明；
- `bash -n scripts/flash/k3-pack-itb.sh`：退出 0，仅证明 shell 语法可解析；
- `file`：固定 rcpu0 为 RISC-V ELF64，两个固定 payload 为 DTB v17；
- `readelf -h -l rt24_os0_rcpu.elf`：退出 0，entry `0x100200000`，LOAD `0x1001ff000..0x10020101c`；
- 工具探测确认 `dtc`、`mkimage`、`lzop`、`fastboot` 缺失；
- `lsusb` 因 libusb 初始化失败退出 99；没有证明开发板已连接；
- 未发现 `build/k3-com260/` 产物；
- 未执行构建、Fastboot、Titan、SD 写入、MTD 写入、复位或真板启动。

本分析影响未来“StarryOS K3 镜像、部署与真板 bring-up”milestone 的范围和 Gate。它不改变当前 MS09 存储知识 change，也不证明任何新镜像可启动。

## 未确认项和下一决策点

进入 Plan 前仍需解决：

1. 首轮目标是仅启动 AP StarryOS，还是同时要求 AP/RP AMP；后者会引入 OpenSBI/ESOS 持久更新。
2. 当前 CoM260 Kit 的准确 DTS 变体、DRAM/reserved-memory 和启动介质。
3. U-Boot 当前版本是否支持 README 中的 Fastboot stage 语法，`0x180000000/0x04000000` 是否安全。
4. StarryOS 最小 payload 是否需要现有 rootfs，还是先使用无 rootfs 的早期 console 镜像。
5. Titan/K3-Ubuntu-Images 的当前 manifest、分区和恢复流程。
6. SD 启动的 boot pin、介质优先级和可重复恢复方式。

推荐的首个 change 只做到“构建并检查 AP FIT → RAM 临时引导 → polling UART 首字节 → 复位恢复出厂系统”。Titan、SD 整盘镜像、OpenSBI/ESOS 持久更新应在该 Gate 通过后分开规划。

## 关键文件

- [`docs/boot/com260-boot-chain.md`](../../docs/boot/com260-boot-chain.md)：官方启动模式、固件阶段和 Fastboot 入口。
- [`docs/boot/com260-image-and-dts.md`](../../docs/boot/com260-image-and-dts.md)：镜像格式、写入方式和 DTS 候选。
- [`k3-com260-uart-boot.md`](../runbooks/k3-com260-uart-boot.md)：已实跑的出厂启动与串口基线。
- [`k3-com260-network-dev-and-file-transfer.md`](../runbooks/k3-com260-network-dev-and-file-transfer.md)：尚未真板验证的网络传输候选流程。
- [`openspec/specs/references/spec.md`](../../openspec/specs/references/spec.md)：R21 网站候选集。
- [`others/Rt-Async-AMP/README.md`](../../others/Rt-Async-AMP/README.md)：第三方 K3 构建、RAM 引导和持久写示例。
- [`others/Rt-Async-AMP/xtask/src/build.rs`](../../others/Rt-Async-AMP/xtask/src/build.rs)：三类 K3 产物的构建编排。
