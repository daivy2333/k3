> 来源: https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/ufs.md（源端修订: unknown；观察日期: 2026-09-02）; https://raw.githubusercontent.com/spacemit-com/docs-buildroot/main/zh/k3_buildroot/device/peripheral_driver/ufs.md（源端修订: unknown；观察日期: 2026-09-09）; https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs/k3_ds.md（源端修订: unknown；观察日期: 2026-09-02）; https://github.com/spacemit-com/docs-chip/blob/main/zh/key_stone/k3/k3_docs/k3_ds.md（源端修订: 2026-08-25 V1.8；观察日期: 2026-09-07）; https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/boot.md（源端修订: unknown；观察日期: 2026-09-02）; https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/boot.md（源端修订: unknown；观察日期: 2026-09-07）; https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/image.md（源端修订: unknown；观察日期: 2026-09-02）; https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/image.md（源端修订: unknown；观察日期: 2026-09-07）; https://github.com/PlaticaIt/StarryOS/blob/19219411d5dc1515496f910d04c93da12ee95be4/drivers/ax-driver/src/block/k3_ufs/mod.rs（源端修订: 19219411d5dc1515496f910d04c93da12ee95be4；观察日期: 2026-09-10）; https://github.com/PlaticaIt/StarryOS/blob/19219411d5dc1515496f910d04c93da12ee95be4/os/StarryOS/configs/board/spacemit-k3-com260-ifx.dts（源端修订: 19219411d5dc1515496f910d04c93da12ee95be4；观察日期: 2026-09-10）

# K3 / CoM260 UFS 主机与同步块设备事实包

> 范围: 按 D1–D6 把 K3 UFS 拆为官方 SoC/板级正文、官方 GitHub 交叉验证、固定 revision 第三方 UFS 同步块 driver 与 K3 专属 binding 缺失事实包；按 D3 区分 MPHY / UniPro / Link startup、UTP/SCSI descriptor（UTRD / UTMRD / UCD / PRDT / UPIU）、DMA/cache/doorbell、轮询完成、单次重试 + 失败 fatal 五阶段状态机；按 D4 区分静态资源（SoC 能力、DTS 字段）与运行时行为（提交 / 完成 / 恢复）；按 D5 把 timeout / OCS / controller fatal 显式列入可观察错误边界并明确恢复上限。

> 边界: 本文档只整合官方 `ufs.md`、官方 `k3_ds.md V1.8`、官方 `boot.md` / `image.md` 与 [`spacemit-k3-com260-ifx.dts`](https://github.com/PlaticaIt/StarryOS/blob/19219411d5dc1515496f910d04c93da12ee95be4/os/StarryOS/configs/board/spacemit-k3-com260-ifx.dts) 直接打开的字段；运行时行为仅引用 tgoskits `k3_ufs/` 固定 revision 第三方源码（Rt-Async-AMP `ccb1ff0b` / tgoskits `19219411`），不提升为 K3 官方硬件规范；不裁决 com260_user_guide §2 / §3.3.3 与 com260_ds §1.4 之间的 128 GB / 256 GB 容量冲突，保留原文；不补写不存在的 K3 UFS 专属 binding 或 IRQ 路径；不修改 UFS 第三方 driver、`docs/reference/source-coverage.md`、`docs/reference/known-gaps.md`、其它 6 篇产品文档或 OpenSpec 体系文件。

- [1. 范围与证据等级](#1-范围与证据等级)
- [2. SoC 能力与板级可达性](#2-soc-能力与板级可达性)
- [3. 静态资源（SoC DTS 字段）](#3-静态资源soc-dts-字段)
- [4. 启动关系（UFS 作为启动介质）](#4-启动关系ufs-作为启动介质)
- [5. MPHY / UniPro / Link startup](#5-mphy--unipro--link-startup)
- [6. UTP / SCSI descriptor（UTRD / UTMRD / UCD / PRDT / UPIU）](#6-utp--scsi-descriptorutrd--utmrd--ucd--prdt--upiu)
- [7. DMA / cache / doorbell](#7-dma--cache--doorbell)
- [8. SCSI / LUN scan 与 sync block 边界](#8-scsi--lun-scan-与-sync-block-边界)
- [9. 完成模型（轮询，不注册 IRQ）](#9-完成模型轮询不注册-irq)
- [10. 错误、超时、controller fatal 与恢复](#10-错误超时controller-fatal-与恢复)
- [11. 未知项](#11-未知项)
- [12. 主题边界与导航](#12-主题边界与导航)

## 1. 范围与证据等级

- 事实: UFS 官网入口 ([`software/SDK/buildroot/k3_buildroot/device/peripheral_driver/ufs.md`](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/ufs.md)) 在来源覆盖表登记为 `current / active / partially-observed`；2026-09-02 观察只取得 SPA 壳，不携带 UTRD / UCD / PRDT 队列字段。R06 是包含该 URL 的外部文档集合，carrier 状态仍为 `deferred`，不改变覆盖表中该 UFS 行的当前责任。本仓库未观察到 K3 UFS 专属 binding。（官方来源入口）
- 事实: 官方 GitHub 对应页（[`docs-buildroot ufs.md`](https://raw.githubusercontent.com/spacemit-com/docs-buildroot/main/zh/k3_buildroot/device/peripheral_driver/ufs.md)）说明 K3 SDK buildroot UFS 驱动描述、UTRD / UCD / PRDT 与 doorbell 行为、CoM260 UFS 节点对应；本 Cycle 2026-09-09 直接打开，仅作交叉验证，不替代官网。（交叉验证）
- 事实: 官方 [`k3_ds.md V1.8 §2.2.6`](https://github.com/spacemit-com/docs-chip/blob/main/zh/key_stone/k3/k3_docs/k3_ds.md) 把 K3 SoC UFS 主机描述为 UFS 2.2 主机（符合 JEDEC UFS 2.2、MIPI UniPro v1.6、MIPI M-PHY v3.0；HS-GEAR3 + PWM-GEAR1；支持从 UFS 直接启动）；本仓库直接打开，证据状态 `confirmed`。（官方 SoC 能力）
- 事实: tgoskits [`k3_ufs/`](https://github.com/PlaticaIt/StarryOS/blob/19219411d5dc1515496f910d04c93da12ee95be4/drivers/ax-driver/src/block/k3_ufs/mod.rs) 固定 revision 第三方源码（`19219411`）将 K3 UFS 注册为同步块设备，不在 driver 中注册 IRQ handler；UFS 路径以 `prepare_slot` → `ring_doorbell` → `poll_completion` 三阶段做 UTRD 提交与完成观察，并把 timeout / OCS error / controller fatal 与单次重试 / fatal latch 分支独立处理。（固定 revision 第三方）
- 推论: UFS 的 K3 专属 doorbell、UTRD / UTMRD / UCD / PRDT / UPIU 字段语义、MPHY gear 协商与 link startup 仍受 K3 programmer reference 缺失限制；本仓库未观察到 K3 UFS 专属 binding。不得为该路径补写不存在的字段或 IRQ 路径。（边界）
- 边界: UFS 协议层（MPHY / UniPro / UTP / SCSI 命令）由本文档 §5–§8 承担；MS06 ([`docs/dma/k3-dma-and-memory-ownership.md`](../dma/k3-dma-and-memory-ownership.md)) 保留 UFS 内建 DMA 与 ownership 五阶段；MS03 ([`docs/boot/com260-boot-chain.md`](../boot/com260-boot-chain.md)) 保留 BROM / FSBL / 启动介质切换与镜像布局；UFS 容量冲突 C1 保持原文，不裁决。

## 2. SoC 能力与板级可达性

- 事实: SoC 控制器层能力为 UFS 2.2 主机（符合 JEDEC UFS 2.2、MIPI UniPro v1.6、MIPI M-PHY v3.0 规范；HS-GEAR3 + PWM-GEAR1；支持从 UFS 直接启动）。该条目仅证明 SoC 控制器层能力，不证明 CoM260 引出或 Kit 启用（依据 [`k3_ds.md V1.8 §2.2.6`](https://github.com/spacemit-com/docs-chip/blob/main/zh/key_stone/k3/k3_docs/k3_ds.md)，直接打开）。（官方 SoC 能力）
- 事实: CoM260 板级资源在 [`com260-board-resources.md §2.3`](../platform/com260-board-resources.md) 登记为 UFS 2.2 板载、默认容量 128 GB；三个公开订货型号（COM3K308128 / COM3K316128 / COM3K332128）UFS 容量均为 128 GB，DDR 容量分别为 8 / 16 / 32 GB（依据 `com260_ds.md §1.4`）。（板级引出）
- 事实: Kit / 底板通过金手指使用模组 UFS（依据 `com260_user_guide.md §3.3.1`）。（Kit 启用）
- 边界: SoC 能力 ≠ 板级引出 / 启用。本节不重写 [`com260-board-resources.md §2.3`](../platform/com260-board-resources.md) 已登记字段，仅引用其结论；冲突 C1（128 GB vs 256 GB，com260_user_guide §2 / §3.3.3 与 com260_ds §1.4）由 [`com260-board-resources.md §10`](../platform/com260-board-resources.md) 承担。

## 3. 静态资源（SoC DTS 字段）

> 本节事实均来自 [`spacemit-k3-com260-ifx.dts`](https://github.com/PlaticaIt/StarryOS/blob/19219411d5dc1515496f910d04c93da12ee95be4/os/StarryOS/configs/board/spacemit-k3-com260-ifx.dts) `ufshc@0xc0e00000` 节点（固定 revision `19219411`），本仓库未观察到 K3 UFS 专属 binding schema。

- 事实: 控制器节点 `ufshc@0xc0e00000` `compatible = "spacemit,k3-ufshcd"`，`status = "okay"`；`reg = <0x00 0xc0e00000 0x00 0x40000>`（MMIO 基址 0xC0E00000，长度 0x40000）。（固定 revision 第三方 DTS）
- 事实: `clock-names = "ufs-aclk"`，`clocks = <0x05 0x37>`，`clock-freq = <0x1d4c0000>`（490 MHz）；`resets = <0x05 0x3b>`，`reset-names = "ufs-aclk-rst"`。（固定 revision 第三方 DTS）
- 事实: `freq-table-hz = <0x1d4c0000 0x1d4c0000>`；`ref-clk-freq = <0x124f800>`（19.2 MHz）。（固定 revision 第三方 DTS）
- 事实: `interrupts = <0x87 0x04>`，`interrupt-parent = <0x82>`；本仓库第三方 UFS driver（tgoskits `k3_ufs/mod.rs:probe`）解码该字段用于日志打印（`IRQ: 135`），但**不在 driver 中注册 IRQ handler**——UFS 完成路径采用 doorbell bit 轮询；IRQ 仅作信息输出，不进入完成判定。（固定 revision 第三方）
- 事实: `lanes-per-direction = <0x02>`；tgoskits `k3_ufs/mod.rs:probe` 把它作为 power-mode 协商无连接 lane 时的 fallback，driver 默认值同样为 2 lane / direction。（固定 revision 第三方）
- 推论: 该 DTS 不在 [`k3-br-v1.0.y`](https://github.com/spacemit-com/linux-6.18/tree/k3-br-v1.0.y) 官方候选内；K3 UFS 专属 binding 在官方候选内未直接登记；本仓库只引用 `spacemit-k3-com260-ifx.dts` 字段，不升级为 K3 官方硬件规范。（边界）
- 边界: 本节不改写 `k3_ds.md V1.8 §2.2.6` 已登记的 SoC 能力字段；不重写 [`docs/dma/k3-dma-and-memory-ownership.md §4`](../dma/k3-dma-and-memory-ownership.md) 已登记的 UFS 内建 DMA 字段。

## 4. 启动关系（UFS 作为启动介质）

- 事实: BROM 可以根据 boot pin 切换不同启动介质（NOR / NAND / eMMC / UFS），具体取决于硬件设计（依据 `docs-buildroot boot.md §刷机流程`）。（交叉验证）
- 事实: eMMC / SD 卡 / UFS 共用同一固件布局，使用 GPT 索引分区表；bootinfo 偏移固定 0x100000 或 0x110000 byte（依据 `docs-buildroot boot.md §固件布局` + `k3_ds.md V1.8 §2.2.4/§2.2.5/§2.2.6`）。（交叉验证）
- 事实: UFS 启动参数为 UFS 2.2、MIPI UniPro v1.6、MIPI M-PHY v3.0、HS-GEAR3 + PWM-GEAR1，并支持从 UFS 直接启动（依据 `k3_ds.md V1.8 §2.2.6`）。（官方 SoC 能力）
- 边界: 启动介质切换、固件布局、bootinfo 偏移与 BROM / FSBL / ESOS / OpenSBI / U-Boot 阶段由 [`com260-boot-chain.md`](../boot/com260-boot-chain.md) 承担；本节不重写 MS03 已登记字段。
- 边界: 镜像布局（GPT、bootinfo、payload）由 [`com260-image-and-dts.md`](../boot/com260-image-and-dts.md) 承担；本节不重写 MS03 镜像字段。

## 5. MPHY / UniPro / Link startup

> 本节运行时行为来自 tgoskits [`k3_ufs/mod.rs:probe`](https://github.com/PlaticaIt/StarryOS/blob/19219411d5dc1515496f910d04c93da12ee95be4/drivers/ax-driver/src/block/k3_ufs/mod.rs) 与 [`k3_ufs/init.rs`](https://github.com/PlaticaIt/StarryOS/blob/19219411d5dc1515496f910d04c93da12ee95be4/drivers/ax-driver/src/block/k3_ufs/init.rs)，固定 revision `19219411`；本仓库未观察到 K3 UFS 专属 binding 内的 MPHY / UniPro 寄存器字段。

- 事实: 第三方 driver `probe` 阶段按 Linux `ufshcd` 流程执行 `host_init` → `mphy_init` → `unipro_init` → `link_startup_pre` → `link_startup` → `link_startup_post` 六步，再 `dump_regs` 才进入 transfer-list 设置。（固定 revision 第三方）
- 事实: 第三方 driver 先以默认 PWM 完成 link startup，再执行 NOP OUT、fDeviceInit 和 device quirks；随后 `upgrade_link_to_hs` 读取双方能力，在 host 的 HS-G3 / 2 lane / Rate B 上限内选择候选并尝试切换。能力不可读、无 HS gear、候选切换失败时保留或恢复 PWM。lane 数依次取 connected、available，二者均为 0 时才使用 DTS `lanes-per-direction` fallback。（固定 revision 第三方）
- 边界: 启动关系（BROM / boot pin / 介质切换 / 固件布局）由 §4 承担；DMA / cache / doorbell 由 §7 承担；不重写 [`com260-boot-chain.md`](../boot/com260-boot-chain.md) 已登记字段。

## 6. UTP / SCSI descriptor（UTRD / UTMRD / UCD / PRDT / UPIU）

> 本节运行时结构来自 tgoskits [`k3_ufs/desc.rs`](https://github.com/PlaticaIt/StarryOS/blob/19219411d5dc1515496f910d04c93da12ee95be4/drivers/ax-driver/src/block/k3_ufs/desc.rs) 与 [`k3_ufs/transfer.rs`](https://github.com/PlaticaIt/StarryOS/blob/19219411d5dc1515496f910d04c93da12ee95be4/drivers/ax-driver/src/block/k3_ufs/transfer.rs) 固定 revision `19219411`；本仓库未观察到 K3 UFS 专属 binding 内的 UTRD / UCD / PRDT 字段语义。

- 事实: transfer list 使用 UTRD（UTP Transfer Request Descriptor）和 UTMRD（UTP Task Management Request Descriptor），每个 transfer slot 的 UTRD 指向对应 UCD（UTP Command Descriptor）。固定第三方实现为 UCD 的 command UPIU 与 response UPIU 各预留 512-byte 对齐区域；这些区域承载 COMMAND / RESPONSE / QUERY / NOP OUT 等报文，不能据此断言所有线上的 UPIU 长度固定为 512 字节。（固定 revision 第三方）
- 事实: PRDT（Physical Region Descriptor Table）由 UCD 引用，描述 data buffer 的物理地址与长度；tgoskits `prepare_slot` 在 data 传输时把 PRDT 链接到 `data_buf`，并对 data buffer 执行 `prepare_for_device`（cache clean + DMA fence）。（固定 revision 第三方）
- 事实: UTRD 保存 command type / data direction / interrupt bit、OCS、UCD base、response UPIU offset/length 和 PRDT offset/length。tgoskits `prepare_slot` 把 command UPIU 复制到 UCD 的 command 区域，并把该 UPIU byte 3 的 task tag 设为 slot；task tag 不在 UTRD 字段中。（固定 revision 第三方）
- 推论: K3 UFS 第三方 driver 把 UTP 报文、descriptor 与 data buffer 的 owner 转换显式拆为三段（CPU 准备 / doorbell 提交 / 轮询完成）；每段有唯一所有者，避免与 GMAC ring 那种 OWN 位混淆（依据 [`docs/dma/k3-dma-and-memory-ownership.md §4`](../dma/k3-dma-and-memory-ownership.md) 已有事实）。（跨主题）
- 边界: UFS 内建 DMA 与 ownership 五阶段由 [`docs/dma/k3-dma-and-memory-ownership.md §4`](../dma/k3-dma-and-memory-ownership.md) 承担；本节不重写 ownership 状态机。

## 7. DMA / cache / doorbell

> 本节运行时行为来自 tgoskits `k3_ufs/transfer.rs` 固定 revision `19219411`；本仓库未观察到 K3 UFS 专属 binding 内的 doorbell / cache 操作规范。

- 事实: tgoskits UFS 路径的 CPU 准备阶段执行 `prepare_for_device`（cache clean + DMA fence），doorbell 提交前执行 `dma_wmb`；轮询 doorbell bit 清零后执行 `dma_rmb` 与 `complete_for_cpu`（cache invalidate），再读取 UTRD OCS 与 response UPIU。（固定 revision 第三方）
- 事实: tgoskits UFS 路径在 `prepare_slot` 阶段把 data buffer 链入 PRDT；data 传输完成后由 `poll_completion` 步骤执行 `complete_for_cpu`；该过程没有 GMAC 式 OWN 位（依据 [`docs/dma/k3-dma-and-memory-ownership.md §4`](../dma/k3-dma-and-memory-ownership.md) 已登记事实）。（固定 revision 第三方 + 跨主题）
- 边界: DMA / cache / memory ownership 五阶段（CPU 准备 / CPU 提交 / device in flight / 完成观察 / 错误与恢复）由 [`docs/dma/k3-dma-and-memory-ownership.md §3`](../dma/k3-dma-and-memory-ownership.md) 承担；本节不重写 ownership 状态机，不补写不存在的字段。

## 8. SCSI / LUN scan 与 sync block 边界

> 本节运行时行为来自 tgoskits [`k3_ufs/scsi.rs`](https://github.com/PlaticaIt/StarryOS/blob/19219411d5dc1515496f910d04c93da12ee95be4/drivers/ax-driver/src/block/k3_ufs/scsi.rs) 与 [`k3_ufs/mod.rs:probe`](https://github.com/PlaticaIt/StarryOS/blob/19219411d5dc1515496f910d04c93da12ee95be4/drivers/ax-driver/src/block/k3_ufs/mod.rs) 固定 revision `19219411`；本仓库未观察到 K3 UFS 专属 binding 内的 LUN scan 流程。

- 事实: 第三方 driver `probe` 阶段在 `setup_transfer_lists` 之后按"启动 → 设备初始化链"步骤完成 NOP OUT、LUN 容量 / UNIT READY / flag 探测；通过后调用 `register_sync_block(plat_dev, host)`，把 K3 UFS 注册为同步块设备（依赖 `axklib` 块设备层）。（固定 revision 第三方）
- 事实: 完成注册后 K3 UFS 暴露给上层的接口是同步块 I/O（read / write 512 字节扇区）；本仓库不证明该同步块 I/O 在 RISC-V SMP / RTOS 场景下的可扩展性或实时性（见 §11 未知项 U4）。（固定 revision 第三方 + 未知项）
- 边界: 块设备层接口（`register_sync_block`）由 `ax-driver/block` 固定 revision 第三方源码承担；本节不重写 `axklib` 块设备层字段。

## 9. 完成模型（轮询，不注册 IRQ）

- 事实: 第三方 K3 UFS driver（tgoskits `k3_ufs/mod.rs:probe`）通过 `info.interrupts()` 解析 IRQ 号并打印日志（`[k3-ufs] IRQ: 135`），**不在 driver 中注册 IRQ handler**；UFS 完成路径在 `poll_completion` 中以 doorbell bit 清零为完成条件。（固定 revision 第三方）
- 推论: K3 UFS 第三方 driver 把 UFS 完成路径与 IRQ 路径解耦：IRQ 解析仅用于信息输出（硬件拓扑存在但未启用），完成判定完全靠轮询 doorbell；这与 GMAC 路径中 IRQ / NAPI / OWN 协同的完成模型不同。（跨主题）
- 边界: 中断拓扑、IMSIC / MSI / PLIC 由 [`docs/interrupts/`](../interrupts/) 承担；本节不重写中断分发字段。

## 10. 错误、超时、controller fatal 与恢复

> 本节运行时行为来自 tgoskits [`k3_ufs/transfer.rs:submit_upiu`](https://github.com/PlaticaIt/StarryOS/blob/19219411d5dc1515496f910d04c93da12ee95be4/drivers/ax-driver/src/block/k3_ufs/transfer.rs) 与 [`k3_ufs/error.rs`](https://github.com/PlaticaIt/StarryOS/blob/19219411d5dc1515496f910d04c93da12ee95be4/drivers/ax-driver/src/block/k3_ufs/error.rs) 固定 revision `19219411`；本仓库未观察到 K3 UFS 专属 binding 内的 controller fatal 处理规范。

- 事实: 第三方 driver 在 `submit_upiu` 中把 UTRD 提交与完成观察拆为三步（`prepare_slot` → `ring_doorbell` → `poll_completion`），并以 `submit_upiu_once` 包装；`poll_completion` 在超时或 OCS error 时返回 `UfsError::Timeout` / `UfsError::OcsError`。（固定 revision 第三方）
- 事实: `submit_upiu` 捕获 `Timeout` / `OcsError` / `ControllerFatal` 三类错误时调用 `recover_controller()`：停止 list / 重置 HCE / 重启 link / 重写 transfer-list 基址，再用 NOP OUT 验证 link 成功后才把同一命令 `submit_upiu_once` 重试一次。（固定 revision 第三方）
- 事实: 恢复失败时（`recover_controller` 自身返回 `Err`），driver 设置 `self.fatal = true`，并把后续所有 `submit_upiu` 立刻以 `UfsError::ControllerFatal` 拒绝，避免反复进入恢复路径；`fatal` 是 latch 状态，直到 driver 重建（不在本路径内）。（固定 revision 第三方）
- 边界: 单次重试、fatal latch 是 tgoskits 第三方实现选择，**不证明** K3 官方硬件规范或 Linux `ufshcd` 行为；本仓库不把该行为升级为 K3 硬件保证。
- 边界: 第三方 driver 的 `recover_controller` 内部步骤（list clear / HCE reset / link restart / NOP OUT verify）由 tgoskits `k3_ufs/transfer.rs:recover_controller` 固定 revision 源码承担；本节不复制源码、不重写错误分类。

## 11. 未知项

### U1 — K3 UFS 专属 binding 缺失

- 当前证据: linux-6.18 [`k3-br-v1.0.y`](https://github.com/spacemit-com/linux-6.18/tree/k3-br-v1.0.y) 分支 `Documentation/devicetree/bindings/ufs/` 目录不包含 `spacemit,k3-ufshcd.yaml` 或同义 schema；本仓库只观察到固定 revision 第三方 DTS `ufshc@0xc0e00000` 节点字段。
- 禁止推断: 不得在缺 binding 情况下补写不存在的 `spacemit,k3-ufshcd.yaml` 字段（interrupts / doorbell / UTRD / UCD / PRDT / OCS 寄存器偏移等）。
- 解除条件: 官方 `k3-br-v1.0.y` 分支或 `docs-buildroot` 提交 `spacemit,k3-ufshcd.yaml` 后，刷新 `docs/reference/source-coverage.md` 中已激活的 UFS 入口及新增 binding 来源；R06 carrier 是否继续 `deferred` 由其文档集合整体责任另行判断。
- 影响: K3 UFS 静态资源（§3）、link startup 字段（§5）、UTRD / UCD / PRDT 字段语义（§6）目前只能引用第三方固定 revision 字段；不得作为 K3 官方硬件规范引用。

### U2 — IRQ 路径未启用

- 当前证据: 第三方 driver `probe` 阶段 `info!("[k3-ufs] IRQ: {}", irq);` 仅打印 IRQ 号；本仓库未观察到 `register_irq` 或 `irq_set_handler` 调用。
- 禁止推断: 不得补写"IRQ 已注册但未在轮询中触发"等推断；不得把"IRQ 解析"等同于"IRQ 启用"。
- 解除条件: tgoskits 第三方 driver 新增 `register_irq` + IRQ 完成路径并由 fixed revision 承载；或 K3 官方 Linux 仓库 [`k3-br-v1.0.y`](https://github.com/spacemit-com/linux-6.18/tree/k3-br-v1.0.y) 提交 `spacemit,k3-ufshcd` driver。
- 影响: K3 UFS 完成模型（§9）、错误恢复超时检测（§10）目前仅靠轮询；IRQ 启用后才能讨论 ISR / NAPI 协同完成。

### U3 — UFS 容量冲突 C1

- 当前证据: `com260_user_guide.md §2` "本地存储: UFS 2.2, 可选 128 GB / 256 GB 容量"；`com260_user_guide.md §3.3.3` 功能接口表只列 `UFS (128GB)`；`com260_ds.md §1.4` 订货型号表只列 8 / 16 / 32 GB DDR × 128 GB UFS 三个组合。
- 禁止推断: 不得裁决"当前套件固定 128 GB"或"256 GB 仅规划"等结论；不得自行补全容量表。
- 解除条件: 官方 `com260_user_guide` 或 `com260_ds` 新版本对齐 §2 / §3.3.3 / §1.4 三处描述后，由 [`com260-board-resources.md §10`](../platform/com260-board-resources.md) 收敛冲突；或第三方实测确认 256 GB 模组存在并登记。
- 影响: 存储镜像规划（`com260-image-and-dts.md` 未来维护）和 bring-up 镜像不能从现有资料推定目标模组容量；任何依赖容量的布局或操作都需要先确认实际目标，256 GB 模组仍需单独证据。

### U4 — 同步块 I/O 在 RTOS / SMP 场景下的可扩展性

- 当前证据: tgoskits 第三方 driver 把 K3 UFS 注册为 `register_sync_block(plat_dev, host)`（同步块设备），但本仓库未观察到多核并发请求队列、IO 调度或 per-CPU buffer cache 的设计。
- 禁止推断: 不得声明"K3 UFS 同步块路径可支撑 SMP 高并发 / 实时性"或"与 Linux block layer 性能等价"。
- 解除条件: 第三方 driver 引入异步 / 多队列路径并由 fixed revision 承载；或提供 throughput / latency 在多核场景的实测数据。
- 影响: MS08 ([`docs/amp/`](../amp/)) RTOS 路径、RISC-V SMP 多核场景下 K3 UFS 块 I/O 的行为不能由本文档支撑；评估 RTOS / SMP 镜像烧录吞吐时必须先建立该路径的实测证据。

### U5 — controller fatal 重建路径

- 当前证据: tgoskits 第三方 driver 在 `recover_controller` 失败时设置 `self.fatal = true`，后续 `submit_upiu` 立刻拒绝；但本仓库未观察到 `fatal = true` 之后 driver 自我重建（如再次 `host_init` / `setup_transfer_lists`）的路径。
- 禁止推断: 不得声明"fatal latch 后 driver 仍可恢复"或"fatal 状态由硬件 reset 唯一解除"等结论。
- 解除条件: tgoskits 第三方 driver 新增 fatal 重建路径并由 fixed revision 承载；或上层 axklib 块设备层提供 block-device re-register 接口。
- 影响: recovery 失败并置位 fatal 后，当前 host 的后续提交会持续失败，直到重新 probe 或重建 host；本仓库未观察到可达的自动重建入口，也不能据此指定模块 reload 或系统重启为唯一恢复手段（见 §10 边界）。

## 12. 主题边界与导航

- 上游: [`com260-board-resources.md §2.3`](../platform/com260-board-resources.md)（板级 UFS 资源矩阵 + 容量冲突 C1）、[`k3-soc-overview.md §2.2.6`](../platform/k3-soc-overview.md)（SoC 能力登记）、[`com260-boot-chain.md`](../boot/com260-boot-chain.md)（BROM / FSBL / 启动介质切换）、[`com260-image-and-dts.md`](../boot/com260-image-and-dts.md)（GPT 镜像布局）、[`k3-dma-and-memory-ownership.md §4`](../dma/k3-dma-and-memory-ownership.md)（UFS 内建 DMA 与 ownership 五阶段）。
- 平行存储主题: [`k3-qspi-spi-sdhci.md`](k3-qspi-spi-sdhci.md)（QSPI / SPI / SDHCI 同步串行存储；不引入 UFS / eMMC 内容）。
- 下游: [`docs/reference/known-gaps.md`](../reference/known-gaps.md)（未来 Iteration 002 收敛 G6 等与 UFS 协议层相关的已知缺口），[`docs/reference/source-coverage.md`](../reference/source-coverage.md)（UFS 主题入口覆盖表权威源）；不重写两者已登记字段。
- 跨主题引用: [`docs/interrupts/`](../interrupts/)（IRQ 拓扑 + IMSIC / MSI / PLIC）、[`docs/amp/`](../amp/)（RTOS 同步块 I/O 上下文）。
- 不变项: 容量冲突 C1（U3 承担）；K3 UFS 专属 binding 缺失（U1 承担）；K3 UFS driver 不注册 IRQ（§9 + U2 承担）；单次重试 / fatal latch（§10 + U5 承担）；MS03 / MS06 / source-coverage 表权威性。
- 禁止项: 不提升第三方 tgoskits 实现为 K3 官方硬件规范；不修改 [`others/Rt-Async-AMP/`](https://github.com/PlaticaIt/StarryOS) 第三方源码；不修改其它 6 篇产品文档、`docs/reference/source-coverage.md`、`docs/reference/known-gaps.md`、OpenSpec 体系文件。
- 官方来源: [`ufs.md`](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/ufs.md)（SPA 壳 `partially-observed`） / [`docs-buildroot ufs.md`](https://raw.githubusercontent.com/spacemit-com/docs-buildroot/main/zh/k3_buildroot/device/peripheral_driver/ufs.md)（cross-validation） / [`k3_ds.md V1.8 §2.2.6`](https://github.com/spacemit-com/docs-chip/blob/main/zh/key_stone/k3/k3_docs/k3_ds.md)（SoC 能力） / [`boot.md`](https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/boot.md) + [`image.md`](https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/image.md)（启动 + 镜像）。
