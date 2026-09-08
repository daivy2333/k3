> 来源: https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/boot.md（源端修订: unknown；观察日期: 2026-09-02）；https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/boot.md（源端修订: unknown；观察日期: 2026-09-07）；https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/image.md（源端修订: unknown；观察日期: 2026-09-02）；https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/image.md（源端修订: unknown；观察日期: 2026-09-07）；https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs/k3_ds.md（源端修订: unknown；观察日期: 2026-09-02）；https://github.com/spacemit-com/docs-chip/blob/main/zh/key_stone/k3/k3_docs/k3_ds.md（源端修订: 2026-08-25 V1.8；观察日期: 2026-09-07）

# K3 / CoM260 启动链

> 文档定位: 整理 K3 SoC 启动能力与 K3 CoM260 Kit 已观察到的启动链路, 不推定未由直接来源给出的装载地址、DRAM 保留区或固件交接寄存器。
> 证据等级: 仅使用 `官方事实` / `交叉验证` / `推论` / `未知项` 四级枚举; `官方事实` 仅适用于官网正文或 SDK 文档正文已直接读出的字段; 来自 SpacemiT 官方 GitHub 仓库的字段一律标 `交叉验证`; 没有直接来源的字段标 `未知项` 并给出四字段闭包。
> 边界: 本文档只覆盖启动模式、介质、阶段、版本与未知边界; 镜像类型、写入方式与目标 DTS 映射见 [`com260-image-and-dts.md`](com260-image-and-dts.md); 板级连接器与 UART 物理参数见 [`../platform/com260-board-resources.md`](../platform/com260-board-resources.md) §7; SoC 能力概述见 [`../platform/k3-soc-overview.md`](../platform/k3-soc-overview.md) §3。

## 目录

1. 启动链路与证据边界
2. SoC 启动能力(K3 Boot ROM 与下载/本地启动)
3. 启动模式与进入 U-Boot Fastboot 的三种操作路径
4. 启动介质(SoC 支持 vs Kit 可用)
5. 启动阶段与产物(Boot ROM → FSBL/SPL → ESOS → OpenSBI → U-Boot → payload/OS)
6. 启动阶段与 SDK 版本边界
7. 串口与早期 console(Kit 物理层)
8. 未知项闭包
9. 修订快照
10. 边界声明

## 1. 启动链路与证据边界

- 本文档范围仅限 K3 SoC 与 K3 CoM260 Kit 的启动阶段划分、介质与版本边界, 不展开 UART/IRQ/DMA/GMAC 寄存器实现; 寄存器级资料属于 MS04-MS07。
- 唯一权威入口仍是 M01 指向的 SpacemiT 社区文档总入口; 本文档正文以官网 boot.md、image.md、k3_ds.md 与对应官方 GitHub 仓库页为直接来源, 来源身份按 D04(URL 是覆盖记录的唯一键)、D05(主题文档首行同时表达修订与观察时间)、D06(证据强度采用四级标记)共同区分。
- 阶段或介质的可证性以"已直接打开并观察到"为准; 仅出现在示例命令或推断中的字段标 `推论` 或 `未知项`。
- "Kit 可用"特指目标 K3 CoM260 Kit(底板 + 模组); "SoC 支持"特指 K3 SoC 自身能力, 二者不自动等同, 沿用本 change design D2 的四层事实模型(SoC / CoM260 Module / Kit / Unknown)。
- 本仓库不存放 bootloader、U-Boot、OpenSBI、DTS 或镜像制品; 启动链仅作文字与流程整理。

## 2. SoC 启动能力(K3 Boot ROM 与下载/本地启动)

| 维度 | 事实 | 证据等级 | 来源 |
| --- | --- | --- | --- |
| 1 级引导存放 | K3 SoC 集成 128 KB Boot ROM, 用于存放一级引导代码 | 交叉验证 | k3_ds.md V1.8 §1.2(直接打开 2026-09-07) |
| 启动介质 | Boot ROM 支持从多种外部介质启动 | 交叉验证 | k3_ds.md V1.8 §1.2(直接打开 2026-09-07) |
| 程序下载 | Boot ROM 支持通过 USB 与 UART 下载程序 | 交叉验证 | k3_ds.md V1.8 §1.2(直接打开 2026-09-07) |
| 启动模式 | K3 SoC 具备 download boot 与 local boot 两种能力 | 交叉验证 | k3_ds.md V1.8 §1.2(直接打开 2026-09-07) |
| 启动优先级 | 第二级启动介质可能存在多个, 启动优先级可配置 | 交叉验证 | docs-buildroot boot.md §固件布局(直接打开 2026-09-07) |
| 启动介质切换 | BROM 可以根据 boot pin 切换不同的启动介质(NOR/NAND/eMMC/UFS), 具体取决于硬件设计 | 交叉验证 | docs-buildroot boot.md §刷机流程(直接打开 2026-09-07) |

注: k3_ds.md 在 V1.8 §1.2 概述层未给出 boot pin 引脚、优先级配置寄存器或介质选择时序; 这些字段在本 Cycle 仍标 `未知项`, 详见 §8。

## 3. 启动模式与进入 U-Boot Fastboot 的三种操作路径

K3 SoC 的启动模式只有两种: **download boot** 与 **local boot**(k3_ds.md V1.8 §1.2)。但进入刷机交互界面(U-Boot Fastboot)有三种不同的操作路径, 来源 docs-buildroot boot.md §U-Boot Fastboot 模式 三种方法如下:

| 路径 | 描述(原文) | 启动阶段身份 | 证据等级 | 来源 |
| --- | --- | --- | --- | --- |
| 1. **按键组合进入**(FEL + RESET) | "按下板子上的 FEL 键和 RESET 键进入 BROM-Fastboot 模式, 然后在上位机(PC)执行以下命令, 使板子进入 U-Boot 刷机交互界面。**BROM-Fastboot 仅用于加载启动 U-Boot Fastboot。**" | 经由 SoC **download boot**(BROM-Fastboot)路径, 是 SoC 启动模式层面的入口 | 交叉验证 | docs-buildroot boot.md §U-Boot Fastboot 模式(直接打开 2026-09-07) |
| 2. **ADB 命令进入** | "对于**已经启动到操作系统的设备**, 可以通过上位机(PC)运行以下命令 `adb reboot bootloader` 使板子进入 U-Boot 刷机交互界面。(某些固件可能移除了 ADB 功能, 因此该方法不通用。)" | 不是 SoC 启动模式, 是**已在本地 local boot 启动 OS 后**切换到 U-Boot 刷机的运行时入口 | 交叉验证 | docs-buildroot boot.md §U-Boot Fastboot 模式(直接打开 2026-09-07) |
| 3. **串口进入**(长按 `s` 键) | "**在板子启动时**, 通过串口长按 `s` 键进入 U-Boot Shell, 然后在串口执行以下命令进入 U-Boot 刷机交互界面 `fastboot 0`" | 触发时机在板子启动阶段(但经由 U-Boot Shell, 不是 Boot ROM 启动模式); 路径不需要已有 OS | 交叉验证 | docs-buildroot boot.md §U-Boot Fastboot 模式(直接打开 2026-09-07) |

注意区分: **SoC 启动模式**(download boot / local boot)是 K3 SoC 能力层面的两种启动路径; **进入 U-Boot Fastboot 的三种操作路径**是进入刷机交互界面的三种方法, 三者都是为了让板子进入 U-Boot 刷机界面, 但触发阶段不同:

- 路径 1 在 SoC download boot 阶段, BROM-Fastboot 仅用于加载 U-Boot Fastboot。
- 路径 2 在已运行 OS(从 local boot 启动)后, 由 adb 命令切换。
- 路径 3 在板子启动过程中(具体阶段未在 boot.md 给出, 可能是 U-Boot 阶段), 通过 UART 输入触发 U-Boot Shell。

SoC 启动模式与本地启动(local boot)的对照:

| SoC 启动模式 | 描述 | 已证进入方式 | 证据等级 | 来源 |
| --- | --- | --- | --- | --- |
| download boot (BROM-Fastboot) | Boot ROM 经 BROM-Fastboot 路径**仅加载 U-Boot Fastboot**(不加载 FSBL), 直接进入 U-Boot Fastboot 模式; FSBL 仅在 local boot 路径按 bootinfo 加载 | 按下 FEL 键 + RESET 键(路径 1) | 交叉验证 | docs-buildroot boot.md §U-Boot Fastboot 模式 + k3_ds.md V1.8 §1.2(均直接打开 2026-09-07) |
| local boot (NOR/NAND/eMMC/UFS) | 从本地介质启动, 经 Boot ROM → FSBL → ESOS → OpenSBI → U-Boot → payload/OS, 见 §5 启动阶段 | 由 boot pin 切换的启动介质 | 交叉验证 | docs-buildroot boot.md §固件布局 + k3_ds.md V1.8 §1.2(直接打开 2026-09-07) |

- 进入路径 2 / 路径 3 的前提不要求 SoC download boot 启动模式, 不应被称作 "download boot (ADB)" / "download boot (U-Boot 串口)"; 本文档第一轮 §3 表曾使用 "download boot (ADB)" / "download boot (U-Boot 串口)" 三种平级分类, 现按 docs-buildroot boot.md 三种方法的描述重写, 纠正 SoC 启动模式与进入刷机交互界面的语义混淆。
- CoM260 Kit 物理按键(FEL/RESET)位置、ADB 固件是否启用(boot.md §U-Boot Fastboot 模式 原文: "某些固件可能移除了 ADB 功能, 因此该方法不通用")由 Kit 自身资料决定, 不由 SoC 能力直接证明, 见 §8 未知项。
- local boot 介质的具体使用由 boot pin 硬件设计决定; 本文档不推定 CoM260 Kit 默认从哪一种介质 local boot。

## 4. 启动介质(SoC 支持 vs Kit 可用)

| 介质 | K3 SoC 控制器能力 | 启动介质身份 | 证据等级 | 来源 |
| --- | --- | --- | --- | --- |
| eMMC | 4.2/4.3/4.4/4.5/4.51/5.0/5.1 | eMMC/SD 卡/UFS 固件布局的 GPT 索引分区表; bootinfo 偏移固定 0x100000 或 0x110000 byte | 交叉验证 | docs-buildroot boot.md §固件布局(直接打开 2026-09-07) + k3_ds.md V1.8 §2.2.4(直接打开 2026-09-07) |
| SD 卡 | SD 3.0/SD 3.01/SD 4.0 | 与 eMMC 共用同一固件布局(eMMC/SD 卡/UFS) | 交叉验证 | docs-buildroot boot.md §固件布局(直接打开 2026-09-07) + k3_ds.md V1.8 §2.2.5(直接打开 2026-09-07) |
| UFS | UFS 2.2, MIPI UniPro v1.6, MIPI M-PHY v3.0, HS-GEAR3 + PWM-GEAR1, 支持从 UFS 直接启动 | eMMC/SD 卡/UFS 固件布局的 GPT 索引分区表 | 交叉验证 | docs-buildroot boot.md §固件布局(直接打开 2026-09-07) + k3_ds.md V1.8 §2.2.6(直接打开 2026-09-07) |
| SPI-NOR + BLK(NOR + eMMC/UFS/SSD) | Quad-SPI 1/2/4 线模式 | 固件部分在 SPI-NOR 上, 部分在第二级启动介质(SSD/eMMC/UFS)上; 第二级介质通过 GPT 索引分区表; bootinfo 偏移固定 0 或 0x10000 byte | 交叉验证 | docs-buildroot boot.md §固件布局 + §NOR+BLK 设备刷机(直接打开 2026-09-07) + k3_ds.md V1.8 §2.2.3(直接打开 2026-09-07) |
| SPI-NAND | 仅在启用 NAND 启动时; 与 NOR 共用同一 SPI 接口, 两者只能选其一, 系统默认使用 NOR 启动 | 全部数据可存放于 spinand, 也可增加第二级启动介质支持; bootinfo 偏移固定 0 或 0x10000 byte | 交叉验证 | docs-buildroot boot.md §固件布局 + §NAND 刷机启动(直接打开 2026-09-07) |

注: 本表只列出 K3 SoC 控制器能力与 docs-buildroot boot.md 已证实的固件布局; CoM260 Kit 实际板载介质实例、是否预留 SPI-NAND 焊盘或 eMMC/UFS 容量, 见 [`../platform/com260-board-resources.md`](../platform/com260-board-resources.md) §2 与 §3, 不在本表范围。

## 5. 启动阶段与产物(Boot ROM → FSBL/SPL → ESOS → OpenSBI → U-Boot → payload/OS)

> docs-buildroot boot.md §fsbl 启动后 原文: "fsbl 启动后, 会先**初始化 DDR**, 然后从分区表**加载 esos, opensbi 和 uboot 到内存的指定位置**, 再**运行 esos, opensbi**, 接着**opensbi 会启动 uboot**。(fsbl 如何初始化 DDR 的内容不在本文档的描述范围内)"
>
> docs-buildroot boot.md §esos, opensbi 和 uboot 加载启动 原文(节选):
> - "**esos + uboot 加载**: 在跳转 opensbi 之前, 由 `board_load_extra_fits()`(`board/spacemit/k3/spl_extra_fit.c`)额外加载, esos 先于 uboot 加载。"
> - "**esos**(Embedded SoC OS) 是 **K3 特有的 SoC 固件分区**。"
> - "**esos / uboot 加载策略**(按优先级): 1. 分区名加载(默认): 通过 env 变量 `extra_esos_partition=esos`、`extra_uboot_partition=uboot` 指定分区名, SPL 扫描分区表找到对应分区后加载 FIT 镜像。2. 绝对偏移加载: 通过 env 变量 `esos_offset`、`uboot_offset` 指定字节偏移。3. 文件系统加载(MMC/UFS, 需开启 `CONFIG_SYS_BOOTLOADER_FS_PARTITION_NAME`): 从指定文件系统分区读取 `esos.itb` 和 `u-boot.itb` 文件, 路径由 `esos_itb_path`、`uboot_itb_path` 控制, 可通过 env `bootloader_from_fs=0` 关闭。"

| 阶段 | 介质/载体 | 阶段产物 | 阶段入口 | 阶段输出 | 证据等级 | 来源 |
| --- | --- | --- | --- | --- | --- | --- |
| 1. Boot ROM | SoC 片内 128 KB ROM | 1 级引导代码(只读) | SoC 上电/复位 | 按 bootinfo 加载 FSBL, 或进入 BROM-Fastboot | 交叉验证 | k3_ds.md V1.8 §1.2(直接打开 2026-09-07) + docs-buildroot boot.md §固件布局(直接打开 2026-09-07) |
| 2. FSBL / SPL(由 bootinfo 定位) | eMMC / SPI-NOR / SPI-NAND | `factory/FSBL.bin` | BROM 通过 bootinfo 二进制分区(`bootinfo_block.bin` / `bootinfo_spinor.bin` / `bootinfo_spinand.bin`, 由 `board/spacemit/k3/configs/bootinfo_*.json` 构建配置生成)读取 FSBL 位置与长度 | 初始化 DDR, 加载 ESOS / OpenSBI / U-Boot | 交叉验证 | docs-buildroot boot.md §fsbl 启动后 + §eMMC 刷机 + §NOR+BLK 设备刷机 + §NAND 刷机启动(直接打开 2026-09-07) |
| 3. ESOS(K3 特有) | eMMC / SPI-NOR / SPI-NAND | `esos.itb`(默认按分区名 `esos` 加载) | 由 `board_load_extra_fits()`(`board/spacemit/k3/spl_extra_fit.c`)加载, esos 先于 uboot 加载 | 运行 esos | 交叉验证 | docs-buildroot boot.md §esos, opensbi 和 uboot 加载启动(直接打开 2026-09-07) |
| 4. OpenSBI | eMMC / SPI-NOR / SPI-NAND | `fw_dynamic.itb` | FSBL / ESOS handoff | SBI 运行时服务 | 交叉验证 | docs-buildroot boot.md §fsbl 启动后 + §eMMC 刷机 + §NOR+BLK 设备刷机(直接打开 2026-09-07) |
| 5. U-Boot | eMMC / SPI-NOR / SPI-NAND | `u-boot.itb`(分区名 `uboot`) | OpenSBI 启动 U-Boot | U-Boot Shell / 加载 bootfs | 交叉验证 | docs-buildroot boot.md §fsbl 启动后 + §eMMC 刷机 + §NOR+BLK 设备刷机(直接打开 2026-09-07) |
| 6. payload / OS | eMMC / UFS / SSD | `bootfs` / `rootfs` | U-Boot handoff | Linux payload 启动 | 交叉验证 | docs-buildroot boot.md §刷机流程(直接打开 2026-09-07) + docs-buildroot image.md(直接打开 2026-09-07) |

bootinfo 与 bootinfo_*.json / bootinfo_*.bin 的关系:

| 概念 | 描述 | 证据等级 | 来源 |
| --- | --- | --- | --- |
| `bootinfo_*.json` | 构建配置: 路径 `board/spacemit/k3/configs/bootinfo_*.json`, 用于保存启动介质与 FSBL 地址、长度等相关信息; 含 `spl_size_limit, 0x74000, 4` 字段(SDK 通用示例, 不是 SoC 规格) | 交叉验证 | docs-buildroot boot.md §更新 bootinfo(直接打开 2026-09-07) |
| `factory/bootinfo_block.bin` | 烧入 eMMC 的二进制: BROM 通过此分区在 0x100000 或 0x110000 byte 偏移定位 | 交叉验证 | docs-buildroot boot.md §固件布局 + §eMMC 刷机(直接打开 2026-09-07) |
| `factory/bootinfo_spinor.bin` | 烧入 SPI-NOR 的二进制: BROM 通过此分区在 0 或 0x10000 byte 偏移定位 | 交叉验证 | docs-buildroot boot.md §固件布局 + §NOR+BLK 设备刷机(直接打开 2026-09-07) |
| `factory/bootinfo_spinand.bin` | 烧入 SPI-NAND 的二进制: BROM 通过此分区在 0 或 0x10000 byte 偏移定位 | 交叉验证 | docs-buildroot boot.md §固件布局 + §NAND 刷机启动(直接打开 2026-09-07) |

注: 第一轮 §5 表把 FSBL 入口写成 "bootinfo 索引(`factory/bootinfo_*.json`)" 是错的; 实际刷入介质的是由 JSON 配置**生成**的 `bootinfo_*.bin` 二进制, JSON 仅是构建配置。docs-buildroot boot.md §eMMC 刷机 / §NOR+BLK 设备刷机 / §NAND 刷机启动 原文刷机命令是 `fastboot flash bootinfo factory/bootinfo_block.bin` 等, `bootinfo_*.json` 的描述见 §更新 bootinfo 步骤。

注: 上述阶段的 handoff 寄存器(例如 `a0`/`a1`/`a2` 在 FSBL → OpenSBI 的具体约定)不由 docs-buildroot boot.md 给出; ESOS 的运行结果、ESOS → OpenSBI 的具体交接、ESOS 在 SDK 之外是否可选均由 docs-buildroot boot.md 未覆盖, 见 §8 未知项; 装载地址、DRAM 保留区、FSBL 实际大小限制(`spl_size_limit, 0x74000, 4` 是 SDK 通用示例, 不是 SoC 规格)均见 §8 未知项。

## 6. 启动阶段与 SDK 版本边界

| 组件 | 文档中已观察的版本 | SDK baseline(MS02 refresh) | 证据等级 | 来源 |
| --- | --- | --- | --- | --- |
| U-Boot | `uboot-2022.10`(出现在 boot.md 注释中) | U-Boot 2022.10 | 交叉验证 | docs-buildroot boot.md §不支持烧写 gzip 格式镜像文件(直接打开 2026-09-07); MS02 R08 K3 路径/分支 baseline |
| OpenSBI | `fw_dynamic.itb` 制品名(无具体版本字段) | OpenSBI 1.6 | 交叉验证 | docs-buildroot boot.md §eMMC 刷机(直接打开 2026-09-07); MS02 R08 SDK baseline |
| Linux | Linux 6.18 DTS 目录(`k3-br-v1.0.y` 分支) | Linux 6.18 | 交叉验证 | linux-6.18 仓库 `k3-br-v1.0.y` 分支(直接打开 2026-09-07); MS02 R08 SDK baseline |
| Buildroot | 镜像格式 zip(适用于 Titan Flasher) | Buildroot 2025.02.6 | 交叉验证 | docs-buildroot image.md(直接打开 2026-09-07); MS02 R08 SDK baseline |

- SDK baseline 由 MS02 change `establish-k3-source-tracking-baseline` 通过 R08 supporting fallback 取得, 等级为 `交叉验证`; 本 Cycle 不刷新它。
- 上表的"文档中已观察的版本"以直接读到的版本字段为准; 没有读到具体版本的字段(OpenSBI、Buildroot)只标 SDK baseline, 不写入具体版本号。
- `partition_universal.json` 与 `partition_4M.json` 的 `version` 字段值为 `"1.0"`, 这是分区表文件自身 schema 版本, 不代表固件或 SoC 版本。

## 7. 串口与早期 console(Kit 物理层)

| 维度 | 事实 | 证据等级 | 来源 |
| --- | --- | --- | --- |
| 串口参数 | 115200-8-N-1 | 交叉验证 | com260-board-resources.md §7(Iteration 000 已交付) |
| 串口物理 | USB 转 TTL | 交叉验证 | com260-board-resources.md §7(Iteration 000 已交付) |
| 12 Pin UART0 引脚 | Pin 3 = RXD, Pin 4 = TXD | 交叉验证 | com260-board-resources.md §7(Iteration 000 已交付) |
| 早期 console 入口 | 通过串口长按 `s` 键进入 U-Boot Shell(适用 K3 系列 SoC 启动阶段) | 交叉验证 | docs-buildroot boot.md §U-Boot Fastboot 模式(直接打开 2026-09-07) |
| 串口命令 | U-Boot Shell 内 `fastboot 0` 进入 U-Boot Fastboot 模式 | 交叉验证 | docs-buildroot boot.md §U-Boot Fastboot 模式(直接打开 2026-09-07) |

注: CoM260 Kit 实际 12 Pin UART0 物理位置、引脚顺序与 USB 转 TTL 型号的板级细节由 com260-board-resources.md §7 与 §3 承担, 本文档不重复。

## 8. 未知项闭包

每个未知项含: `当前证据` / `禁止推断` / `解除条件` / `影响主题` 四字段。

### 8.1 CoM260 Kit 默认启动介质

- 当前证据: docs-buildroot boot.md 只描述 K3 SoC 三种固件布局, 不指明 CoM260 Kit 默认从哪一种 local boot; k3_ds.md V1.8 §1.2 也未给出 CoM260 介质选择。
- 禁止推断: 不由 SoC 能力推定 eMMC/UFS/SPI-NOR 任一为 Kit 默认; 不由 com260_user_guide.md V2.0 产品版本 `v03` 推定。
- 解除条件: CoM260 Kit 原理图、`com260_hw_resources.md` 下载制品、目标 DTS 中 `chosen { boot-media }` 或 `boot-pins` 字段直接观察。
- 影响主题: Iteration 001 / T11(DTS 映射)、MS04(平台资源)、MS07(GMAC/PHY 介质归属)。

### 8.2 装载地址、DRAM 保留区与 FSBL 大小限制

- 当前证据: docs-buildroot boot.md 给出 SDK 通用示例 `spl_size_limit, 0x74000, 4`, 这是 bootinfo_*.json 中的示例字段, 不是 SoC 规格; docs-buildroot boot.md 未给出 FSBL → OpenSBI 的 handoff 寄存器。
- 禁止推断: 不由示例字段值推定 SoC 物理限制; 不由 OpenSBI 1.6 / U-Boot 2022.10 通用约定推定 K3 实际值; 不从 K1、Pico、DesignWare 默认值推定。
- 解除条件: 目标 DTS 中 `memory@`、`reserved-memory`、OpenSBI/U-Boot 链接脚本或 bootinfo_*.bin 的实际字段直接观察。
- 影响主题: MS05(中断与时间)、MS06(DMA/IOMMU 与内存一致性)、MS07(GMAC ring 缓冲区布局)。

### 8.3 启动优先级配置寄存器与 boot pin 引脚

- 当前证据: docs-buildroot boot.md 描述第二级启动介质优先级可配置, BROM 根据 boot pin 切换介质; k3_ds.md V1.8 §1.2 未给出具体引脚或寄存器。
- 禁止推断: 不由 K3 SoC boot pin 默认值推定; 不由 SoC 概述层引脚分配(AA~AT + 1~20)推定具体 boot 复用。
- 解除条件: 目标 DTS 中 `boot-pins`、SoC TRM 寄存器字段或 com260_hw_resources.md 下载制品中原理图直接观察。
- 影响主题: MS04(平台资源)、MS07(介质归属)。

### 8.4 eFuse 字段布局与启动身份

- 当前证据: docs-buildroot boot.md 未提供 eFuse 字段; k3_ds.md V1.8 §1.2 未给出 eFuse 字段布局。
- 禁止推断: 不由其他 RISC-V SoC 通用 eFuse 字段推定; 不由 K1 或 Pico 推定。
- 解除条件: SoC TRM 或 com260_hw_resources.md 下载制品中 eFuse map 直接观察。
- 影响主题: MS05(中断与时间)、MS07(安全/启动身份)。

### 8.5 SDK 通用示例与 CoM260 Kit 实际刷机命令的差异

- 当前证据: docs-buildroot boot.md 给出的 `fastboot flash gpt partition_universal.json` 等命令针对 K3 Buildroot SDK, 不区分 Kit。
- 禁止推断: 不由 SDK 通用示例推定 CoM260 Kit 的实际刷机命令或分区; 不由命令文件名前缀推定 Kit 身份。
- 解除条件: CoM260 Kit release notes 或目标 DTS 中 `partition_*.json` 的实际引用直接观察。
- 影响主题: T11(镜像与 DTS 文档)、MS07(介质落点)。

### 8.6 OpenSBI 与 Buildroot 在 CoM260 Kit 上的实际版本

- 当前证据: docs-buildroot boot.md 未给出 OpenSBI 与 Buildroot 的版本字段; docs-buildroot image.md 只说明镜像格式 zip; SDK baseline 来自 MS02 交叉验证。
- 禁止推断: 不由 SDK baseline 推定 CoM260 Kit 默认使用同一 OpenSBI 1.6 / Buildroot 2025.02.6; 不由 Linux 6.18 + U-Boot 2022.10 推定。
- 解除条件: CoM260 Kit release notes 或镜像包中版本字段直接观察。
- 影响主题: MS05(中断)、MS07(网络与存储介质)。

## 9. 修订快照

| 文档来源 | 已观察修订 | 观察日期 | 状态 |
| --- | --- | --- | --- |
| docs-buildroot boot.md | 页面未提供版本字段; 已读出 1740 行正文 | 2026-09-07 | observed |
| docs-buildroot image.md | 页面未提供版本字段; 已读出 6 行正文 | 2026-09-07 | observed |
| k3_ds.md V1.8 | 2026-08-25, §1.2 补充 DPU0/DPU1 显示接口说明 | 2026-09-07 | observed |
| MS02 SDK baseline | OpenSBI 1.6, U-Boot 2022.10, Linux 6.18, Buildroot 2025.02.6 | 2026-09-05 | supporting(不刷新) |

## 10. 边界声明

1. 本文档不写 load address、DRAM 保留区、FSBL/OpenSBI/U-Boot handoff 寄存器的具体值, 不从默认值或相邻板型推定。
2. "K3 SoC 支持"不等于"CoM260 Kit 可用"; 任一介质/接口在文档中只在该 Kit 直接证据出现时标 `官方事实` 或 `交叉验证`, 否则按 §8 闭包。
3. 产品版本(com260_user_guide.md V2.0 中的 `K3-CoM260_P1_LP5315B_32X2_v03_20260312`)、文档修订(用户指南 V2.0/2026-03-19、k3_ds.md V1.8/2026-08-25)、SDK baseline(OpenSBI 1.6 / U-Boot 2022.10 / Linux 6.18 / Buildroot 2025.02.6)、DTS 分支(`k3-br-v1.0.y`)与观察日期五个概念不互换。
4. docs-buildroot boot.md / image.md 仅作 `交叉验证` 等级使用; 官网 boot.md / image.md 仍是唯一 `官方事实` 来源, 但正文为 SPA 壳, 实际可见身份为 Vue SPA title="SpacemiT", `partially-observed` 状态保持。
5. 本文档只覆盖启动阶段、介质与版本边界; 镜像类型、写入方式与目标 DTS 映射见 [`com260-image-and-dts.md`](com260-image-and-dts.md)。
6. SDK baseline 不在本 Cycle 刷新; 任何对 OpenSBI/U-Boot/Linux/Buildroot 版本的更新应作为独立 refresh change。
7. SoC 启动模式只有 download boot 与 local boot 两种; 进入 U-Boot Fastboot 的三种操作路径(FEL+RESET / ADB / 长按 `s` 键)不是 SoC 启动模式的分类, 而是刷机交互界面的三种入口; 不应混用术语。
8. ESOS 是 K3 特有的 SoC 固件分区, 在 FSBL 之后、OpenSBI 之前运行; `bootinfo_*.json` 是构建配置, 实际刷入介质的是由其生成的 `bootinfo_*.bin`。
