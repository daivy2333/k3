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
- 最近观察修订日期: (待首次聚合时由 change 记录)
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
- 最近观察日期: 2026-09-02
- 用途: 锁定 OpenSBI、U-Boot、Linux 与 Buildroot 版本，并跟踪 IMSIC、PCIe、GMAC、NVMe、SD/SDIO、suspend 和外部中断修复。
- 状态: watch

## R08 — SpacemiT 官方文档与内核仓库 (交叉验证)

- 类型: external-source-set
- URLs:
  - https://github.com/spacemit-com/docs-chip
  - https://github.com/spacemit-com/docs-product
  - https://github.com/spacemit-com/docs-buildroot
  - https://github.com/spacemit-com/linux-6.18
- 最近观察日期: 2026-09-02
- 用途: 发现官网目录、追踪历史与核对 DTS/compatible/驱动入口；不替代 R01 指定的官网权威正文。
- 状态: supporting
