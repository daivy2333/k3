> 来源: https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs（源端修订: unknown；观察日期: 2026-09-02）

# 来源覆盖表

> 产品文档通过获批 OpenSpec change 修改；change accepted 后，docs-maintainer 仅按实际结果同步 SNAPSHOT、tasks、M/D/K/R/I 中适用的状态。
> 字段定义: 唯一键为 `URL`；所有 38 个 URL 由 R01、R04-R08 转录，无新增无遗漏。
> 修订规则: 行级状态变更需创建 refresh change；本表不记录 refresh 运行历史，只记录当前覆盖。

## 字段说明

| 字段 | 取值 |
| --- | --- |
| URL | 唯一键，原文保留 |
| 来源职责 | authority / official-doc / official-product / supporting-source |
| 目标范围 | CoM260 / K3-common / non-target-board / workflow-support |
| 主题位置 | 当前或未来 `docs/` 主题职责（platform / boot / interrupts / serial / dma / network / storage / buses / peripherals / 总入口） |
| 优先级 | current / future / supporting |
| 聚合状态 | active / deferred / out-of-scope / supporting |
| 源端修订 | 页面明确给出时记录，否则写 `unknown` |
| 观察日期 | 当前统一为 `2026-09-02` |
| 访问状态 | `observed`: exact URL 的正文已直接取得 / `partially-observed`: 官方目录或检索片段确认来源但未完整取得正文 / `unverified`: 尚无直接或官方目录证据 |
| 备注 | 缺口、迁移、跨板归属、用途限制 |

## 覆盖表（38 行）

| URL | 来源职责 | 目标范围 | 主题位置 | 优先级 | 聚合状态 | 源端修订 | 观察日期 | 访问状态 | 备注 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs | authority | K3-common | 总入口 | current | active | unknown | 2026-09-02 | partially-observed | M01 单一权威源；K3 全部资料的入口 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs/k3_ds.md | official-doc | K3-common | platform | current | active | unknown | 2026-09-02 | partially-observed | K3 SoC datasheet |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs/root_overview.md | official-doc | K3-common | platform | current | active | unknown | 2026-09-02 | partially-observed | K3 SoC root overview |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/eco/k3_com260/root_overview.md | official-doc | CoM260 | platform | current | active | unknown | 2026-09-02 | partially-observed | K3 CoM260 模组 root overview |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/eco/k3_com260/com260_hw_resources.md | official-doc | CoM260 | platform | current | active | unknown | 2026-09-02 | partially-observed | K3 CoM260 模组硬件资源 |
| https://www.spacemit.com/community/development-kit/k3-pico-itx | official-product | non-target-board | platform | future | out-of-scope | unknown | 2026-09-02 | partially-observed | K3 Pico-ITX 开发板；非 CoM260 目标板，仅作 cross-validation |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/eco/k3_pico/pico_hw_resources.md | official-doc | non-target-board | platform | future | out-of-scope | unknown | 2026-09-02 | partially-observed | K3 Pico 模组硬件资源；非 CoM260 目标板 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/boot.md | official-doc | K3-common | boot | current | active | unknown | 2026-09-02 | partially-observed | K3 SDK buildroot 启动流程 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/image.md | official-doc | K3-common | boot | current | active | unknown | 2026-09-02 | partially-observed | K3 SDK buildroot 镜像构建 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/device_management.md | official-doc | K3-common | platform | current | active | unknown | 2026-09-02 | partially-observed | K3 SDK buildroot 设备管理 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/01-PINCTRL.md | official-doc | K3-common | platform | current | active | unknown | 2026-09-02 | partially-observed | K3 pinctrl 驱动说明 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/05-UART.md | official-doc | K3-common | serial | current | active | unknown | 2026-09-02 | partially-observed | K3 UART 驱动说明 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/09-GMAC.md | official-doc | K3-common | network | current | active | unknown | 2026-09-02 | partially-observed | K3 GMAC 驱动说明 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/16-Clock.md | official-doc | K3-common | platform | current | active | unknown | 2026-09-02 | partially-observed | K3 clock 驱动说明 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/21-DMA.md | official-doc | K3-common | dma | current | active | unknown | 2026-09-02 | partially-observed | K3 DMA 驱动说明 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/Reset.md | official-doc | K3-common | platform | current | active | unknown | 2026-09-02 | partially-observed | K3 reset 驱动说明 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/Timer.md | official-doc | K3-common | interrupts | current | active | unknown | 2026-09-02 | partially-observed | K3 timer 驱动说明 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/02-GPIO.md | official-doc | K3-common | peripherals | future | deferred | unknown | 2026-09-02 | partially-observed | K3 GPIO 驱动说明；非当前目标 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/03-PWM.md | official-doc | K3-common | peripherals | future | deferred | unknown | 2026-09-02 | partially-observed | K3 PWM 驱动说明；非当前目标 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/04-IR-RX.md | official-doc | K3-common | peripherals | future | deferred | unknown | 2026-09-02 | partially-observed | K3 IR-RX 驱动说明；非当前目标 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/06-I2C.md | official-doc | K3-common | buses | future | deferred | unknown | 2026-09-02 | partially-observed | K3 I2C 驱动说明；非当前目标 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/07-QSPI.md | official-doc | K3-common | storage | future | deferred | unknown | 2026-09-02 | partially-observed | K3 QSPI 驱动说明；非当前目标 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/08-SDHC.md | official-doc | K3-common | storage | future | deferred | unknown | 2026-09-02 | partially-observed | K3 SDHC 驱动说明；非当前目标 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/10-USB | official-doc | K3-common | buses | future | deferred | unknown | 2026-09-02 | partially-observed | K3 USB 驱动说明；非当前目标 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/11-PCIe.md | official-doc | K3-common | buses | future | deferred | unknown | 2026-09-02 | partially-observed | K3 PCIe 驱动说明；非当前目标 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/15-CAN.md | official-doc | K3-common | buses | future | deferred | unknown | 2026-09-02 | partially-observed | K3 CAN 驱动说明；非当前目标 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/17-Audio.md | official-doc | K3-common | peripherals | future | deferred | unknown | 2026-09-02 | partially-observed | K3 Audio 驱动说明；非当前目标 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/22-EtherCAT.md | official-doc | K3-common | buses | future | deferred | unknown | 2026-09-02 | partially-observed | K3 EtherCAT 驱动说明；非当前目标 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/23-WDT.md | official-doc | K3-common | peripherals | future | deferred | unknown | 2026-09-02 | partially-observed | K3 WDT 驱动说明；非当前目标 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/24-RTC.md | official-doc | K3-common | peripherals | future | deferred | unknown | 2026-09-02 | partially-observed | K3 RTC 驱动说明；非当前目标 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/SPI.md | official-doc | K3-common | storage | future | deferred | unknown | 2026-09-02 | partially-observed | K3 SPI 驱动说明；非当前目标 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/ufs.md | official-doc | K3-common | storage | future | deferred | unknown | 2026-09-02 | partially-observed | K3 UFS 驱动说明；非当前目标 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/source.md | official-doc | K3-common | workflow-support | current | supporting | unknown | 2026-09-02 | partially-observed | K3 SDK buildroot 源码与构建；作 supporting，不作正文权威 |
| https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/release_notes/bl-v1.0.y.md | official-doc | K3-common | workflow-support | current | supporting | unknown | 2026-09-02 | partially-observed | K3 SDK buildroot bl-v1.0.y release notes；作 supporting |
| https://github.com/spacemit-com/docs-buildroot | supporting-source | K3-common | workflow-support | supporting | supporting | unknown | 2026-09-02 | observed | Spacemit 官方 buildroot 文档仓库；只作 cross-validation |
| https://github.com/spacemit-com/docs-chip | supporting-source | K3-common | workflow-support | supporting | supporting | unknown | 2026-09-02 | observed | Spacemit 官方 chip 文档仓库；只作 cross-validation |
| https://github.com/spacemit-com/docs-product | supporting-source | K3-common | workflow-support | supporting | supporting | unknown | 2026-09-02 | observed | Spacemit 官方 product 文档仓库；只作 cross-validation |
| https://github.com/spacemit-com/linux-6.18 | supporting-source | K3-common | workflow-support | supporting | supporting | unknown | 2026-09-02 | observed | Spacemit 官方 linux-6.18 仓库；只作 cross-validation |
