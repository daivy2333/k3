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
