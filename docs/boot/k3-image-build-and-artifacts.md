> 来源: https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/boot.md（源端修订: unknown；观察日期: 2026-09-02；partially-observed）；https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/boot.md（源端修订: unknown；观察日期: 2026-09-07）；https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/image.md（源端修订: unknown；观察日期: 2026-09-02；partially-observed）；https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/image.md（源端修订: unknown；观察日期: 2026-09-07）；https://github.com/spacemit-com/K3-Ubuntu-Images/blob/main/README.rst（源端修订: branch main；观察日期: 2026-09-14；K3 Pico-ITX）；https://github.com/spacemit-com/K3-Ubuntu-Images/blob/main/gadget.in/gadget.yaml（源端修订: branch main；观察日期: 2026-09-14；K3 Pico-ITX）；https://github.com/spacemit-com/K3-Ubuntu-Images/blob/main/image_flash.py（源端修订: branch main；观察日期: 2026-09-14；K3 Pico-ITX）；https://github.com/spacemit-com/K3-Ubuntu-Images/blob/main/partition_universal.json（源端修订: branch main；观察日期: 2026-09-14；K3 Pico-ITX）；https://github.com/spacemit-com/K3-Ubuntu-Images/blob/main/partition_4M.json（源端修订: branch main；观察日期: 2026-09-14；K3 Pico-ITX）

# K3 镜像构建与产物关系

本文把 K3 SDK、K3 Pico-ITX 官方仓库和固定 revision 第三方实现中的产物放进同一套字段，但不把三条构建链视为等价实现。读者可以据此判断一个文件由谁生成、装了什么、交给哪个启动阶段，以及它属于 RAM、分区还是文件系统；不能据此认定某个示例适用于 CoM260。

## 1. 范围与证据等级

| 等级 | 本文用途 | 不能证明 |
| --- | --- | --- |
| 官方事实 | SpacemiT 社区 K3 boot/image 入口 | SPA 壳未展示的正文、CoM260 实际分区与运行结果 |
| 官方仓库行为 | docs-buildroot 与 K3-Ubuntu-Images 中直接读取的文件 | 仓库示例已在 CoM260 执行；Pico-ITX 布局适用于 CoM260 |
| 固定 revision 第三方 | [R22](../../.claude/analysis/k3-image-flashing-and-real-board-bringup.md) 捕获的 Rt-Async-AMP `ccb1ff0b`、tgoskits `19219411d` 及当前本地对应路径 | 官方支持、真板成功、当前 CoM260 的唯一地址或 DTB |
| 推论 | 已知字段之间的关系 | 未出现的值或兼容性 |
| 未知项 | 当前证据不足的字段 | 可以用相邻板型、文件名或通用默认值补齐 |

项目未构建、下载或验证本节列出的任何镜像。既有启动阶段与介质事实以 [CoM260 启动链](com260-boot-chain.md) 为准，DTS 候选与内存边界以 [镜像与 DTS](com260-image-and-dts.md) 为准，存储控制器责任以 [K3 存储主题](../storage/k3-qspi-spi-sdhci.md) 和 [K3 UFS](../storage/k3-ufs.md) 为准。

## 2. 七种位置不是同一地址

| 名称空间 | 含义 | 示例来源 | 使用边界 |
| --- | --- | --- | --- |
| FIT `load` | U-Boot 解包后把某个 image 放入的内存地址 | 第三方 StarryOS ITS | 不等于 FIT 文件上传处 |
| FIT `entry` | 控制权交给 payload 时的入口地址 | 第三方 StarryOS ITS | 不等于 DTB 地址或存储偏移 |
| 上传缓冲区 | Host 通过 Fastboot stage 发送整个容器时占用的 RAM | 第三方 README 的 `0x180000000` | 只属固定第三方示例，不是通用 CoM260 值 |
| BootROM 传输内存 | download boot 时 BootROM 接收临时 FSBL/U-Boot 的 RAM | K3-Ubuntu-Images Fastboot 流程 | 当前文件未给出可移植地址，不与上传缓冲区合并 |
| 存储偏移 | 原始 MTD 或介质上的字节位置 | Pico-ITX `partition_4M.json` | 示例只属该仓库目标板和布局 |
| GPT/MTD 分区 | 由名称、偏移和大小描述的持久区 | K3 SDK / Pico-ITX JSON | 同名分区的内容仍可因构建链不同而不同 |
| 文件系统路径 | ESP、bootfs 或 rootfs 内的文件位置 | Pico-ITX README / gadget | 不等于分区偏移，也不代表 BootROM 可直接读取 |

因此，第三方 `starryos.uimg` 上传到 `0x180000000` 后，其 kernel 的 `load/entry = 0x140000000`、FDT 的 `load = 0x138000000`。这三个地址来自固定第三方 ITS/README，分别属于容器缓冲区和两个 FIT image 字段。它们不能反推 CoM260 官方镜像布局。

## 3. K3 SDK 启动产物

| 产物 | 生产者或来源 | 格式 / payload | 消费阶段 | 目标名称空间 | 板型范围 | 等级与未知项 |
| --- | --- | --- | --- | --- | --- | --- |
| `bootinfo_*.json` | K3 SDK board 配置 | 构建配置，保存介质和 FSBL 地址/长度字段 | 构建步骤 | SDK 源码路径 | K3-common 示例 | 官方仓库行为；不是刷入介质的二进制 |
| `bootinfo_block.bin` / `bootinfo_spinor.bin` / `bootinfo_spinand.bin` | SDK 由对应 JSON 生成 | BootROM 可消费的 bootinfo 二进制 | BootROM | 固定介质位置或 `bootinfo` 分区 | K3-common；CoM260 实际介质未知 | 官方仓库行为；目标偏移必须按板型和介质确认 |
| `FSBL.bin` | K3 U-Boot/SPL 构建 | FSBL/SPL | BootROM | `fsbl` 分区或 bootinfo 指定位置 | K3-common | 官方仓库行为；CoM260 重建与恢复路径未知 |
| `esos.itb` | K3 SDK ESOS 构建 | K3 ESOS FIT | FSBL/SPL | `esos` 分区、偏移或配置的文件路径 | K3-common | 官方仓库行为；不能与第三方精简 ESOS 容器等同 |
| `fw_dynamic.itb` | K3 OpenSBI 构建 | M-mode SBI 固件 FIT | FSBL/SPL/ESOS | `opensbi` 分区 | K3-common | 官方仓库行为；版本和 CoM260 现场内容未知 |
| `u-boot.itb` | K3 U-Boot 构建 | U-Boot FIT | OpenSBI 或 download boot 临时服务 | `uboot` 分区或 RAM | K3-common | 同名文件的 RAM/持久角色必须由具体流程决定 |
| `bootfs` / `bootfs.img` | K3 SDK 镜像构建 | 内核、DTB、启动配置所在文件系统镜像 | U-Boot | GPT/文件系统 | K3-common 示例 | 内容、容量及 CoM260 对应关系未知 |
| `rootfs` / `rootfs.ext4` | K3 SDK 镜像构建 | Linux 用户空间 | Linux | GPT/文件系统 | K3-common 示例 | 不等于 AP kernel FIT；容量和布局未知 |

bootinfo 配置、生成的 bootinfo 二进制和 `fsbl` 是三种对象。文件名相近或处于同一工具目录，不能改变其生产/消费关系。

## 4. K3 Pico-ITX 官方 UEFI 镜像链

K3-Ubuntu-Images README 明确将目标写为 **SpacemiT K3 Pico-ITX**。以下内容是 SpacemiT 官方仓库行为，但目标范围仍是 `non-target-board`，不能替代 CoM260 资料。

| 产物 | 生产者或来源 | 格式 / payload | 消费阶段 | 目标名称空间 | 板型范围 | 等级与未知项 |
| --- | --- | --- | --- | --- | --- | --- |
| `ubuntu-26.04-...img` / `.img.zst` | K3-Ubuntu-Images / ubuntu-image | 单个 GPT 整盘镜像或其压缩包 | 提取/刷写工具 | Host 文件；展开后写目标盘 | Pico-ITX | 官方仓库行为；版本随 release 变化，不是 CoM260 镜像 |
| `.tar.gz` Titan 包 | `image_flash.py --titan` 或 release | Titan 目录归档 | Titantools | Host 归档 → 多个持久目标 | Pico-ITX | 官方仓库行为；CoM260 包格式与适用工具未知 |
| `env.bin` | gadget 固件输入 | U-Boot 环境 | FSBL/后续固件 | `env` 分区 | Pico-ITX 布局 | 不得使用示例偏移补齐 CoM260 |
| `bootinfo_spinor.bin` | gadget 的 U-Boot 固件输入 | bootinfo 二进制 | BootROM | `bootinfo` 分区 | Pico-ITX 布局 | `partition_universal` 与 `partition_4M` 的偏移不同 |
| `FSBL.bin` | gadget 的 U-Boot 固件输入 | U-Boot SPL | BootROM | `fsbl` 分区 | Pico-ITX 布局 | 与 K3 SDK 同名不证明二进制相同 |
| `esos.itb` | gadget 的 ESOS 输入 | ESOS FIT | FSBL | `esos` 分区 | Pico-ITX 布局 | 不等同第三方 Rt-Async-AMP ESOS |
| `fw_dynamic.itb` | gadget 的 OpenSBI 输入 | OpenSBI FIT | FSBL/ESOS | `opensbi` 分区 | Pico-ITX 布局 | 现场版本与 CoM260 适用性未知 |
| `edk2.itb` | gadget 中的 UEFI 固件 | EDK2 UEFI FIT | OpenSBI | 名为 `uboot` 的历史分区 | Pico-ITX | 分区名不是 payload 类型；运行时不是 U-Boot |
| `u-boot.itb` | PPA 提供的临时刷机固件 | 含 Fastboot server 的 U-Boot FIT | USB download BootROM | RAM，不写入任何分区 | Pico-ITX 流程 | 只作临时刷机服务；不可与 `uboot` 分区中的 `edk2.itb` 混用 |
| `esp.vfat` | gadget 构建 | ESP，含 GRUB EFI 和 stub 配置 | EDK2 | `esp` GPT 分区 | Pico-ITX | CoM260 是否采用 UEFI/ESP 未确认 |
| `cidata.vfat` | gadget 构建 | cloud-init NoCloud 数据 | Ubuntu 首启 | `cidata` GPT 分区 | Pico-ITX | 不属于通用 K3 固件阶段 |
| `writable.ext4` | ubuntu-image/gadget | Ubuntu rootfs 与 `/boot/grub/grub.cfg` | GRUB/Linux | `writable` GPT 分区 | Pico-ITX | 不等同 K3 SDK `rootfs.ext4`，也不是 CoM260 默认 rootfs |

该链的运行时路径为 BootROM → FSBL → OpenSBI → EDK2 → GRUB → Linux，ESOS 在独立管理核上运行。`uboot` 分区存放 `edk2.itb` 是历史命名；用于刷机的 `u-boot.itb` 只驻留 RAM。名称空间和 payload 类型必须分别读取。

## 5. 固定 revision 第三方 StarryOS / Rt-Async-AMP 链

本节复用 R22 捕获的第三方 revision，并对照当前本地路径。它描述实现，不构成官方或真板验证结论。

生产者证据入口为 [K3 环境配置](../../others/Rt-Async-AMP/envs/k3-com260.toml)、[xtask 构建逻辑](../../others/Rt-Async-AMP/xtask/src/build.rs)、[K3 FIT 打包脚本](../../others/Rt-Async-AMP/scripts/flash/k3-pack-itb.sh)、[ESOS ITS](../../others/Rt-Async-AMP/scripts/flash/esos_k3_com260_ifx.its) 和 [StarryOS ITS](../../others/Rt-Async-AMP/tgoskits/os/StarryOS/configs/board/spacemitk3-com260kit.its)。这些链接只定位 R22 固定 revision 工作树中的实现证据。

| 产物 | 生产者或来源 | 格式 / payload | 消费阶段 | 目标名称空间 | 板型范围 | 等级与未知项 |
| --- | --- | --- | --- | --- | --- | --- |
| `opensbi.itb` | `xtask::opensbi_k3` 复制 `fw_dynamic.itb` | OpenSBI FIT | FSBL/ESOS | 构建目录；第三方说明另有 `opensbi` MTD 写入路径 | 固定第三方 CoM260 配置 | 第三方行为；未在本项目构建或刷写 |
| `esos.itb` | `build_rt_async` + `k3-pack-itb.sh` | rcpu0 占位固件、rcpu1 rt-async、两个 DTB、交互 blob | K3 FSBL/ESOS 装载链 | 构建目录；第三方说明另有 `esos` MTD 写入路径 | 固定第三方 `com260_ifx` 配置 | 不是官方 ESOS；固定 payload 与目标板映射未由官方资料确认 |
| `starryos.uimg` | tg-xtask + `spacemitk3-com260kit.its` | AP kernel + 固定 IFX DTB 的 FIT | U-Boot `bootm` | Host 文件 → 上传缓冲区 → FIT load/entry | 固定第三方 CoM260 配置 | DTB 与当前 Kit 未唯一映射；未在本项目构建或引导 |
| `rt-async-k3-*.elf` | Rust build/xtask | RCPU1 ELF | ESOS 打包脚本 | 构建目录 → `esos.itb` payload | 固定第三方 | 不是 AP payload，也不是可直接刷写的完整镜像 |

`cargo xtask build k3-com260` 在当前第三方源码中组织 OpenSBI、全部 K3 rt-async ELF、ESOS 打包和 StarryOS 构建。本文没有执行该命令。打包脚本需要 `mkimage`、`lzop`、固定 payload 和目标 ELF；文件存在或脚本可读不能证明产物可构建。

## 6. 产物关系速查

```text
构建配置/源码
  ├─ bootinfo_*.json ─生成→ bootinfo_*.bin ─BootROM读取→ FSBL位置
  ├─ U-Boot/SPL ─生成→ FSBL.bin ─BootROM加载→ DDR与后续固件初始化
  ├─ ESOS源码或第三方payload ─打包→ esos.itb ─FSBL加载→ 管理核/RCPU
  ├─ OpenSBI ─打包→ fw_dynamic.itb 或 opensbi.itb ─加载→ SBI runtime
  ├─ U-Boot ─打包→ u-boot.itb ─加载→ U-Boot或临时Fastboot服务
  ├─ EDK2 ─打包→ edk2.itb ─Pico-ITX的`uboot`分区→ UEFI
  └─ OS构建 ─生成→ bootfs/rootfs、GPT镜像或starryos.uimg
```

箭头只表达对应来源已经证明的生产或消费关系，不表示所有板型共享同一文件、地址、分区或验证状态。

## 7. 未知项闭包

### U1. CoM260 的正式镜像包与产物清单

- 当前证据: K3 SDK 文档给出通用产物；K3-Ubuntu-Images 明确面向 Pico-ITX；第三方链使用固定 CoM260/IFX 配置。
- 禁止推断: 不由 K3 通用或 Pico-ITX release 名推定 CoM260 官方镜像名、内容或固件组合。
- 解除条件: CoM260 官方下载页或与产品版本绑定的 manifest 直接列出包名、产物、校验信息和适用介质。
- 影响主题: 本文、后续 RAM boot、Fastboot/Titan/SD 与恢复文档。

### U2. CoM260 分区、介质和固件内容映射

- 当前证据: K3 SDK 有多种介质布局；Pico-ITX JSON 给出 GPT/4M MTD 示例；CoM260 默认启动介质和分区现场未确认。
- 禁止推断: 不把 Pico-ITX 偏移、大小、分区名或 EDK2 payload 写成 CoM260 默认值。
- 解除条件: 目标板官方分区配置、板上只读分区枚举与对应官方恢复包三者对齐。
- 影响主题: 持久刷写、恢复、存储控制器和 G7。

### U3. CoM260 FIT、DTB、load 与 entry

- 当前证据: 第三方 ITS 给出 AP kernel、FDT 与上传缓冲区三组地址，但固定 IFX DTB 未与当前 Kit 唯一映射。
- 禁止推断: 不由第三方文件名、注释或成功生成 FIT 推定官方地址安全、DTB 匹配或可启动。
- 解除条件: 目标 CoM260 变体的官方 DTS/内存保留区、镜像 ITS/FIT 元数据和真板启动日志一致。
- 影响主题: 本文、RAM boot、[镜像与 DTS](com260-image-and-dts.md) G7。

### U4. 同名固件的版本和兼容关系

- 当前证据: SDK、Pico-ITX gadget 和第三方链都出现 FSBL/ESOS/OpenSBI/U-Boot 相关名称，但生产源、payload 和目标布局不同。
- 禁止推断: 不按文件名或分区名认定二进制兼容；不把 `uboot` 分区等同 U-Boot payload。
- 解除条件: 对应 release/manifest 明确绑定生产版本、目标板、介质、加载者和升级/回退关系。
- 影响主题: 全部部署与恢复路径。

## 8. 使用边界

- 本文是知识索引，不是 Runbook；所有命令、构建和板上操作均未在本项目执行。
- RAM 引导的前置状态、命令字段和失败边界由后续 `k3-ram-boot-and-fastboot.md` 承担。
- 持久 Fastboot、Titan、SD 与恢复要求由后续 `k3-flashing-and-recovery.md` 承担。
- 当前证据不足时停止在未知项，不以 Pico-ITX、固定第三方实现或相邻板型补值。
