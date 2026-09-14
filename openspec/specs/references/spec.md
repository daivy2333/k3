# References

## Purpose

索引项目依赖的内部产物和外部资料。条目使用 `Rxx`, 只记录检索元数据
(类型, 路径或 URL, 版本或日期, 用途, 状态), 不复制目标正文。

Change Evidence 位于所属 change 内, 由 change 提供索引, 不登记 R。

## Requirements

### Requirement: 参考可定位

参考 SHALL 记录类型、路径或 URL、版本或日期、用途和状态。

#### Scenario: 登记持久化产物

- **WHEN** 新分析、Runbook 或 Incident 需要跨会话复用
- **THEN** 使用递增 R 编号登记检索元数据

---

## R01 — SpacemiT K3 官方文档 (权威源)

- 类型: external-doc
- URL: https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs
- 语言: zh-CN
- 最近观察修订日期: 2026-09-05 (initial observation baseline 2026-09-02 由 change `establish-k3-doc-foundation` 记录; 2026-09-05 由 refresh change `establish-k3-source-tracking-baseline` 更新; R01 仍为 SPA 壳, 实际可见身份为 Vue SPA title="SpacemiT", `partially-observed` 状态保持)
- 用途: 本仓库 M01 指定的唯一权威源; 任何 K3 相关信息变更首先在此确认。
- 状态: active

## R02 — OpenSpec 工作流规范

- 类型: schema
- 路径: 仓库根 `openspec/`, `CLAUDE.md`, `.claude/`
- 版本: OpenSpec CLI 1.6.0
- 用途: 定义本仓库内的 change, spec, evidence, cycle 结构和验证流程。
- 状态: active

## R03 — K3 官方资料面向 StarryOS 异步驱动的聚合分析

- 类型: analysis
- 路径: `.claude/analysis/k3-official-docs-for-starryos-async-drivers.md`
- 版本: captured 2026-09-02, k3 `8a97785ce339d8298421ba4a398709db5d65ca42`, StarryOS `b83e800aa937568eff3a11c32e840b0b8730eade`
- 用途: 记录 K3 官网资料规模下界、StarryOS MS09-MS12 依赖、首批 15 个聚合主题和未来驱动资料分级。
- 状态: active

## R04 — K3 芯片与目标板资料 (当前有用)

- 类型: external-doc-set
- URLs:
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs/k3_ds.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs/root_overview.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/eco/k3_com260/root_overview.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/eco/k3_com260/com260_hw_resources.md
  - https://www.spacemit.com/community/development-kit/k3-pico-itx
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/eco/k3_pico/pico_hw_resources.md
- 最近观察日期: 2026-09-02
- 用途: 确认 SoC 能力，并在 StarryOS MS09 前比较目标板、板载 MAC/PHY、存储、debug UART 和可用连接器。
- 状态: active

## R05 — K3 平台与异步驱动基础页 (当前有用)

- 类型: external-doc-set
- URLs:
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/boot.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/image.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/device_management.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/01-PINCTRL.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/05-UART.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/09-GMAC.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/16-Clock.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/21-DMA.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/Reset.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/Timer.md
- 最近观察日期: 2026-09-02
- 用途: 为 StarryOS K3 platform descriptor、启动、pinctrl、clock/reset、UART、IRQ、GMAC、DMA/cache 与 timeout 设计提供首轮聚合入口。
- 状态: active

## R06 — K3 后续异步设备驱动资料 (未来有用)

- 类型: external-doc-set
- URLs:
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/02-GPIO.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/03-PWM.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/04-IR-RX.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/06-I2C.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/07-QSPI.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/08-SDHC.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/10-USB
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/11-PCIe.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/15-CAN.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/17-Audio.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/22-EtherCAT.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/23-WDT.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/24-RTC.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/SPI.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/ufs.md
- 最近观察日期: 2026-09-02
- 用途: 在 GPIO、PWM、I2C、SPI/QSPI、storage、USB、PCIe、CAN、Audio、EtherCAT、WDT 或 RTC milestone 获批后定位对应资料。
- 状态: deferred

## R07 — K3 SDK 基线与更新追踪 (持续观察)

- 类型: external-doc-set
- URLs:
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/source.md
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/release_notes/bl-v1.0.y.md
- 最近观察日期: 2026-09-05 (由 refresh change `establish-k3-source-tracking-baseline` 更新; 官网两个 URL 仍为 SPA 壳, 实际 SDK baseline 由对应 SpacemiT 官方 GitHub `docs-buildroot` 文档建立, 等级 `交叉验证`: K3 Buildroot SDK v1.0.0-v1.0.7, 核心组件 OpenSBI 1.6 / U-Boot 2022.10 / Linux 6.18 / buildroot 2025.02.6, manifest `k3-br-v1.0.y.xml` → 分支 `k3-br-v1.0.y`)
- 状态: watch

## R08 — SpacemiT 官方文档与内核仓库 (交叉验证)

- 类型: external-source-set
- URLs:
  - https://github.com/spacemit-com/docs-chip
  - https://github.com/spacemit-com/docs-product
  - https://github.com/spacemit-com/docs-buildroot
  - https://github.com/spacemit-com/linux-6.18
- 最近观察日期: 2026-09-05 (由 refresh change `establish-k3-source-tracking-baseline` 更新; 四个仓库均已定位 K3 相关路径或分支: `docs-buildroot/tree/main/zh/k3_buildroot`, `docs-chip/tree/main/zh/key_stone/k3`, `docs-product/tree/main/zh/k3_com260`, `linux-6.18/tree/k3-br-v1.0.y`)
- 状态: supporting

## R09 — Rt-Async-AMP 的 K3 启动与板级适配分析

- 类型: analysis
- 路径: `.claude/analysis/rt-async-amp-k3-boot-platform.md`
- 版本: captured 2026-09-08, k3 `573162934e6ebdb6fe5d254c09cd29a922231e15`, Rt-Async-AMP `ccb1ff0b487e4f49ea570c41f330741eecece935`, tgoskits `19219411d5dc1515496f910d04c93da12ee95be4`
- 用途: 检索第三方 K3 AP/RP 镜像链、FIT 装载地址、RT24 握手、保留内存和刷写边界；不替代 R01 官方事实。
- 状态: active

## R10 — Rt-Async-AMP 的 K3 共享内存与通知链分析

- 类型: analysis
- 路径: `.claude/analysis/rt-async-amp-k3-shared-memory.md`
- 版本: captured 2026-09-08, Rt-Async-AMP `ccb1ff0b487e4f49ea570c41f330741eecece935`, tgoskits `19219411d5dc1515496f910d04c93da12ee95be4`, OpenSBI `7a2df083ed06373c506e2e6f4e09bbd168202f2d`
- 用途: 检索 AP/RP SRAM 布局、PMA 非缓存窗口、mailbox/APLIC/IMSIC 门铃、`/dev/rt_shm` 以及初始化与唤醒风险。
- 状态: active

## R11 — Rt-Async-AMP 的 K3 驱动与 StarryOS 异步边界分析

- 类型: analysis
- 路径: `.claude/analysis/rt-async-amp-k3-drivers.md`
- 版本: captured 2026-09-08, Rt-Async-AMP `ccb1ff0b487e4f49ea570c41f330741eecece935`, tgoskits `19219411d5dc1515496f910d04c93da12ee95be4`, StarryOS `6fcc602de48217a125d40f1f34635138464a2b9e`
- 用途: 检索 K3 APLIC/IMSIC、PXA UART、pinctrl、GMAC、UFS、RT24 平台驱动的可复用层次、忙等和 SMP/DMA 边界。
- 状态: active

## R12 — Rt-Async-AMP 面向 StarryOS 的 K3 复用清单

- 类型: analysis
- 路径: `.claude/analysis/rt-async-amp-k3-starryos-reuse.md`
- 版本: captured 2026-09-08, k3 `573162934e6ebdb6fe5d254c09cd29a922231e15`, Rt-Async-AMP `ccb1ff0b487e4f49ea570c41f330741eecece935`, tgoskits `19219411d5dc1515496f910d04c93da12ee95be4`, StarryOS `6fcc602de48217a125d40f1f34635138464a2b9e`
- 用途: 按优先级检索十组可复用材料、milestone 映射、验证缺口以及尚需补齐的 `rt-async`、`ov-channels`、U-Boot 和板级材料。
- 状态: active

## R13 — K3 CoM260 平台控制与 UART 基线需求规范

- 类型: delta-spec
- 路径: `openspec/specs/k3-com260-platform-uart-baseline/spec.md`
- 版本: 2026-09-08 由 change `establish-k3-com260-platform-uart-baseline` 归档时同步（5 added, 0 removed, 0 modified）
- 用途: 收录 K3 AP / APBC2 secure / RCPU 三域的 pinctrl / clock / reset / APBC / CCU provider 依赖, UART 17 实例的 MMIO / IRQ / FIFO / threshold / compatible 字段, 静态与运行时 console 路径分层, PXA UART 工程经验适用边界, 以及来源覆盖与未知项导航规则; 是 MS04 主题文档与 source-coverage / known-gaps 同步的可追溯契约。
- 状态: active

## R14 — K3 CoM260 中断、时间与通知机制基线需求规范

- 类型: delta-spec
- 路径: `openspec/specs/k3-com260-interrupt-time-notification-baseline/spec.md`
- 版本: 2026-09-09 由 change `establish-k3-com260-interrupt-time-notification-baseline` 归档时同步（5 added, 0 removed, 0 modified）
- 用途: 收录 K3 AP AIA/CLINT/IMSIC/APLIC 与 RCPU PLIC/SysTimer/MSIP/AON timer 的分域拓扑, wired IRQ 与 MSI 路径分层, timer 与软件通知的域边界, AP↔RP mailbox 双向通知链 (mailbox4 ch0/ch1、APLIC source 217、PLIC source 69、IMSIC EID 保留为未知项), 通知与数据状态分离原则, 来源覆盖与未知项四字段导航规则; 是 MS05 主题文档与 source-coverage / known-gaps 同步的可追溯契约。
- 状态: active

## R15 — K3 CoM260 DMA、cache、PMA 与内存所有权基线需求规范

- 类型: delta-spec
- 路径: `openspec/specs/k3-com260-dma-memory-ownership-baseline/spec.md`
- 版本: 2026-09-09 由 change `establish-k3-com260-dma-memory-ownership-baseline` 归档时同步（added, removed, modified 数待归档时由 OpenSpec 集成确认）
- 用途: 收录 K3 DMA 类型与能力按对象分层（通用 DMA controller / 设备内建 DMA / 共享内存通道不互相补值）、CPU 与设备所有权转换可验证、barrier / cache maintenance / 地址宽度与边界规则、PMA 16 entries + Svpbmt K3 silicon 忽略 + AMP window IO 翻转、PMA/PBMT/IOMMU/地址转换四对象不互相替代、来源覆盖与未知项四字段导航规则; 是 MS06 主题文档与 source-coverage / known-gaps 同步的可追溯契约。
- 状态: active

## R16 — K3 CoM260 GMAC、MDIO、PHY 与网络硬件基线需求规范

- 类型: delta-spec
- 路径: `openspec/specs/k3-com260-gmac-network-baseline/spec.md`
- 版本: 2026-09-10 由 change `establish-k3-com260-gmac-network-baseline` 归档时同步（5 added, 0 removed, 0 modified）
- 用途: 收录 K3 SoC GMAC 能力、CoM260 模组引出、Kit 板级连接与 DTS 变体四层分层的板级事实, MDIO/PHY/RGMII 静态链与 PHY ID 边界, DWMAC5 MAC/MTL/DMA 与 TX/RX descriptor/data buffer ownership、cache/doorbell、IRQ/reclaim、设备专有错误（descriptor error、TBU/RBU、FBE、reset、link-down）和 `try_lock` 锁竞争分层, 官方/固定 revision 第三方证据等级, 不指定默认 Kit DTS、不由 PHY ID 推定完整器件、不宣称真板运行时 link/IRQ delivery/coherency/IOMMU/reset 恢复已验证; 是 MS07 主题文档与 source-coverage / known-gaps 同步的可追溯契约。
- 状态: active

## R17 — K3 AMP、共享内存、RPC 与跨核通信基线需求规范

- 类型: delta-spec
- 路径: `openspec/specs/k3-amp-rpc-baseline/spec.md`
- 版本: 2026-09-10 由 change `establish-k3-amp-rpc-baseline` 归档时同步（6 added, 0 removed, 0 modified）
- 用途: 收录 K3 AP/RP 镜像与握手、共享窗口地址与 alias、初始化所有权与 PMA/PBMT/cache 边界、生命周期状态表、reset/re-init 与未读消息可能丢失、共享 ring + 通知 + 等待者分层、RPC 正常/错误/超时/取消/reset 路径、BUSY 提示语义（不充当锁/互斥门禁）、官方/固定 revision 第三方证据等级, 缺失的 `rt-async` / `ov-channels` / U-Boot K3 分支 / 手册 / 原理图 / 真板日志保留为缺口; 是 MS08 主题文档与 source-coverage / known-gaps / terminology 同步的可追溯契约。
- 状态: active

## R18 — K3 CoM260 串口启动观察手册

- 类型: runbook
- 路径: `.claude/runbooks/k3-com260-uart-boot.md`
- 版本: captured 2026-09-11（本次会话用户实跑观察, 后续更新需重跑或新增实跑证据）
- 用途: 记录 K3 CoM260 Kit 出厂预装 Bianbu Linux 4.0.1 镜像的 USB-TTL 串口启动观察流程, 包括 12V/6A 电源、3 根杜邦线（不接 VCC/5V）接线、`115200 8N1` 串口参数、BootROM→SPL/FSBL→OpenSBI→U-Boot→Linux Kernel→rootfs→systemd→Bianbu→ttyS0 login 启动链、`Bianbu 4.0.1 k3 ttyS0` + `k3 login:` 成功判据、官方预装镜像默认 root 凭据 `root` / `bianbu`、以及密码修改安全提示; 适用范围为 K3 CoM260 Kit 预装 Bianbu 4.0.1 镜像的 local boot 路径, 不覆盖 download boot / BROM-Fastboot / 其它 K3 板卡 / 自编译或非 Bianbu 发行版。
- 状态: active

## R19 — K3 CoM260 网络开发与文件传输流程 (草稿)

- 类型: runbook (draft)
- 路径: `.claude/runbooks/k3-com260-network-dev-and-file-transfer.md`
- 版本: captured 2026-09-11（本次会话用户描述, **未在真板执行**, 全部具体动作 / IP 网段 / U 盘分区路径 / scp 命令均为典型流程整理, 25 处 `⚠️` 标记 + 显式 "采集缺口" 清单）
- 用途: 描述 K3 CoM260 Kit 通过以太网直连 + SSH/SCP/SFTP + U 盘备用的网络开发与文件传输流程, 涵盖网线直连、同网段 IP (静态 / ICS 两种方案)、SSH 登录、双向 `scp` 传输、U 盘备用 (含 `lsblk` / `mount /dev/sda1 /mnt/usb` 示例, 设备节点仅为示例)、最终开发结构图; 依赖 R18 完成串口启动后 root Shell 可用; 假设根凭据来自 R18; 当前为 draft 状态, 待真板实跑补齐采集缺口后可升级为 active; 不覆盖路由器 / VLAN / 无线 / 跨网段 / 防火墙深入配置 / VS Code Remote 接入。
- 状态: draft

## R20 — K3 链接缺口与存储来源评估

- 类型: analysis
- 路径: `.claude/analysis/k3-link-gap-and-storage-source-assessment.md`
- 版本: captured 2026-09-11, k3 `820535c0bab7b2e58df3c1c01bc6a9e2689ba4a9`
- 用途: 对照临时候选清单与 `source-coverage.md`，区分已逐 URL 登记、已有语义覆盖、当前 MS09 存储 change 的直接来源缺口，以及仅适用于未来镜像、烧录、OpenSBI 和 StarryOS/ArceOS 移植调查的链接；记录 2026-09-11 GitHub raw DNS 不可达边界。
- 状态: active

## R21 — K3 镜像、启动与 StarryOS 移植候选网站

- 类型: external-source-set
- URLs:
  - https://www.spacemit.com/community/document/info?lang=zh&nodepath=tools%2Fuser_guide%2Fflasher_user_guide
  - https://github.com/spacemit-com/K3-Ubuntu-Images
  - https://github.com/spacemit-com/K3-Ubuntu-Images/blob/main/README.rst
  - https://github.com/spacemit-com/uboot-2022.10
  - https://github.com/spacemit-com/opensbi
  - https://github.com/riscv-software-src/opensbi/blob/master/docs/platform/generic.md
  - https://github.com/riscv-software-src/opensbi/blob/master/docs/platform_guide.md
  - https://github.com/riscv-software-src/opensbi/blob/master/docs/platform_requirements.md
  - https://github.com/Starry-OS/StarryOS
  - https://github.com/arceos-org/arceos
  - https://github.com/arceos-org/app-helloworld
  - https://github.com/spacemit-com/manifests
  - https://github.com/spacemit-com/buildroot
  - https://github.com/spacemit-com/buildroot-ext
  - https://github.com/spacemit-com/archlinux-spacemit
  - https://github.com/spacemit-com/.github/blob/main/upstream-status/toolchain.md
- 版本: candidate set captured 2026-09-11；本次未直接观察，GitHub raw 访问因 DNS 解析失败退出 6
- 用途: 为未来 K3 镜像封装、Titan/Fastboot 烧录、厂商启动组件、OpenSBI 平台责任、StarryOS/ArceOS 移植、SDK 组成和工具链调查提供检索入口；不属于当前 MS09 Iteration 000 的直接来源，不支持当前硬件事实或可用性结论。
- 状态: deferred

## R22 — K3 CoM260 镜像、烧录与真板启动路径分析

- 类型: analysis
- 路径: `.claude/analysis/k3-image-flashing-and-real-board-bringup.md`
- 版本: captured 2026-09-11, k3 `820535c0bab7b2e58df3c1c01bc6a9e2689ba4a9`, Rt-Async-AMP `ccb1ff0b487e4f49ea570c41f330741eecece935`, tgoskits `19219411d5dc1515496f910d04c93da12ee95be4`
- 用途: 检索 K3 官方启动产物、StarryOS/RT24 FIT 构建链、U-Boot RAM 临时引导、ESOS/OpenSBI 持久更新边界、Titan/SD 卡待补条件，以及从出厂基线到 workload 的真板分层 Gate；所有未实跑命令和破坏性写入均有明确边界。
- 状态: active

## R23 — K3 存储控制器（QSPI、SPI、SDHC、UFS）基线需求规范

- 类型: delta-spec
- 路径: `openspec/specs/k3-storage-controller-baseline/spec.md`
- 版本: 2026-09-11 由 change `establish-k3-storage-controller-baseline` 归档时同步（7 added, 0 removed, 0 modified）
- 用途: 收录 K3 SoC 与 CoM260 模组的 QSPI / 普通 SPI / SDHC / UFS 四类存储控制器的资源与板级可达性、启动介质关系、数据路径与资源所有权（按控制器分层）、UFS 协议栈与设备边界（MPHY/UniPro/UTP/SCSI/descriptor/DMA/cache）、错误超时与恢复语义、来源冲突与不可达状态处理、存储主题导航与既有基线一致性；明确官方事实、官方软件行为、固定 revision 第三方经验、推论与未知项的等级边界，不把 UFS 行为外推到 QSPI / SPI / SDHC，不把静态 IRQ 描述等同于运行路径使用 IRQ；是 MS09 主题文档与 source-coverage / known-gaps / terminology 同步的可追溯契约。
- 状态: active

## R24 — K3 外设总线（I2C、USB、PCIe、CAN、EtherCAT）基线需求规范

- 类型: delta-spec
- 路径: `openspec/specs/k3-peripheral-bus-baseline/spec.md`
- 版本: 2026-09-12 由 change `establish-k3-peripheral-bus-baseline` 归档时同步（7 added, 0 removed, 0 modified）
- 用途: 收录 K3 I2C / USB / PCIe / CAN / EtherCAT 五类总线的 SoC 能力、控制器资源、对象边界（I2C controller/client、USB PHY/Host/DRD/role switch/Hub、PCIe RC/EP/PCIe PHY/lane、FlexCAN 与 CAN-FD、收发器、EtherCAT master）与板级可达性分层、静态拓扑与运行能力分离（PCIe 静态 vs 枚举/枚举/MSI、CAN 三层分离、EtherCAT master 复用 MS07 GMAC 基线）、来源冲突与不可达状态处理、总线导航与既有 MS03/MS04/MS05/MS07 基线的一致性；明确官方事实、官方软件行为、推论与未知项的等级边界，不把 SoC 控制器存在、节点、phandle、收发器或连接器声明为枚举/通信/协议运行成功；是 MS10 主题文档与 source-coverage / known-gaps / terminology 同步的可追溯契约。
- 状态: active

## R25 — K3 通用外设（GPIO、PWM、IR-RX、Audio、WDT、RTC）基线需求规范

- 类型: delta-spec
- 路径: `openspec/specs/k3-general-peripheral-baseline/spec.md`
- 版本: 2026-09-12 由 change `establish-k3-general-peripheral-baseline` 归档时同步（8 added, 0 removed, 0 modified）
- 用途: 收录 K3 GPIO / PWM / IR-RX / Audio / WDT / RTC 六类通用外设的 SoC 能力、控制器资源、对象边界（GPIO controller/pinctrl mux/IRQ 责任分离、PWM channel/pinmux/consumer 与 30/20 数量冲突并列保留、IR-RX controller 与 input 边界、Audio I2S/SSPA/DAI/sound card/codec 或 display endpoint/DMA/power domain 静态链与 codec/stream 边界分离、WDT 计数/复位与 MMIO RTC / RPMI RTC 双路径生命周期）与板级可达性分层、来源冲突与不可达状态处理、通用外设导航与既有 MS03-MS06 / G7 基线的一致性；明确官方事实、官方软件行为、推论与未知项的等级边界，不把 SoC 控制器存在、节点、复用引脚、电源输入或 disabled 状态解释为真板 GPIO/PWM/IR/Audio/WDT/RTC 功能可用；DMA/cache 边界通过 MS06 引用而不重复定义；是 MS11 主题文档与 source-coverage / known-gaps / terminology 同步的可追溯契约。
- 状态: active

## R26 — K3 镜像制作、烧录与启动操作基线需求规范

- 类型: delta-spec
- 路径: `openspec/specs/k3-image-flashing-boot-operations-baseline/spec.md`
- 版本: 2026-09-14 由 change `establish-k3-image-flashing-boot-operations-baseline` 归档时同步（8 added, 0 removed, 0 modified）
- 用途: 收录 K3/CoM260 镜像产物可追溯契约（producer/source、format/container、payload role、consuming stage、destination namespace、board scope、evidence class、unknowns）、地址/分区名称空间分离（FIT load/entry、upload buffer、BootROM transfer memory、storage offset、GPT/MTD partition、filesystem path）、易失 RAM 引导与持久部署分离、破坏性操作安全防护（Fastboot 分区、Titan 包、SD 整盘、单组件更新）、失败诊断不升级破坏性、来源与板级范围显式、命令可归属且未执行、导航与汇总一致；是 MS12 主题文档（`k3-image-build-and-artifacts.md` / `k3-ram-boot-and-fastboot.md` / `k3-flashing-and-recovery.md`）与 source-coverage / known-gaps（G15） / terminology / index 同步的可追溯契约。
- 状态: active
