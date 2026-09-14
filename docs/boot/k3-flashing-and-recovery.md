> 来源: https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/boot.md（源端修订: unknown；观察日期: 2026-09-02；partially-observed）；https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/boot.md（源端修订: unknown；观察日期: 2026-09-07）；https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/image.md（源端修订: unknown；观察日期: 2026-09-02；partially-observed）；https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/image.md（源端修订: unknown；观察日期: 2026-09-07）；https://github.com/spacemit-com/K3-Ubuntu-Images/blob/main/README.rst（源端修订: branch main；观察日期: 2026-09-14；K3 Pico-ITX）；https://github.com/spacemit-com/K3-Ubuntu-Images/blob/main/gadget.in/gadget.yaml（源端修订: branch main；观察日期: 2026-09-14；K3 Pico-ITX）；https://github.com/spacemit-com/K3-Ubuntu-Images/blob/main/image_flash.py（源端修订: branch main；观察日期: 2026-09-14；K3 Pico-ITX）；https://github.com/spacemit-com/K3-Ubuntu-Images/blob/main/fastboot.yaml（源端修订: branch main；观察日期: 2026-09-14；K3 Pico-ITX）；https://github.com/spacemit-com/K3-Ubuntu-Images/blob/main/partition_universal.json（源端修订: branch main；观察日期: 2026-09-14；K3 Pico-ITX）；https://github.com/spacemit-com/K3-Ubuntu-Images/blob/main/partition_4M.json（源端修订: branch main；观察日期: 2026-09-14；K3 Pico-ITX）

# K3 持久部署与恢复边界

本文比较四类会改变持久状态的机制，不是可执行 Runbook。本项目未安装或运行刷写工具，未连接目标板，未读取现场分区，也未写入、擦除、分区或格式化任何介质。当前证据不足以形成 CoM260 刷写步骤；表中的动作只说明在证据齐全时必须满足的合同。

产物和目标名称空间见[镜像构建与产物关系](k3-image-build-and-artifacts.md)，易失入口、暂存和分层失败见 [RAM 引导与 Fastboot](k3-ram-boot-and-fastboot.md)，启动阶段见 [CoM260 启动链](com260-boot-chain.md)，DTS 边界见[镜像与 DTS](com260-image-and-dts.md)。介质责任由 [K3 QSPI/SPI/SDHCI](../storage/k3-qspi-spi-sdhci.md) 与 [K3 UFS](../storage/k3-ufs.md) 承担，现有未决项见[已知缺口](../reference/known-gaps.md)，固定第三方调查见 [R22](../../.claude/analysis/k3-image-flashing-and-real-board-bringup.md)。

## 1. 证据等级与适用边界

| 证据 | 可说明 | 不能说明 |
| --- | --- | --- |
| 官方 K3 文档 | 通用固件布局、Fastboot 与 Titan/ZIP 机制 | 当前 CoM260 包名、介质、分区、容量、工具版本或恢复包 |
| 官方 Pico-ITX 仓库 | GPT/MTD、Fastboot、Titan 和整盘镜像如何组合 | Pico-ITX 的 JSON、gadget、偏移、大小或固件适用于 CoM260 |
| 固定 revision 第三方 | OpenSBI/ESOS 先暂存再写同名 MTD 分区的实现意图 | 当前板上存在同名分区、介质可写、组件兼容或操作成功 |
| 已接受 RAM 基线 | 如何先验证 AP payload 而不改持久介质 | RAM 成功等于某个持久组件可以替换 |
| 未知项 | 缺失证据必须阻止动作 | 可以由通用 K3、相邻板或文件名补值 |

同一文件名不建立同一二进制身份；同一分区名不建立同一 payload 类型；同一工具名不建立板型或包格式兼容性。

## 2. 共同破坏性操作门

以下条件必须同时成立，缺一项就停止在写入之前。

| 门 | 必须取得的证据 | 失败时处理 |
| --- | --- | --- |
| 目标身份 | 板型、模组变体、序列/物理标识、当前启动模式与 Host 枚举相互一致且目标唯一 | 不选择设备，不传输写入数据 |
| 介质身份 | 现场只读信息证明 UFS/eMMC/SD/SPI-NOR/SPI-NAND/SSD 的实际角色与容量 | 不用 SoC 支持列表推定板载介质 |
| 包适用性 | 官方 release/manifest 把包名、版本、板型、介质和布局绑定到同一目标 | 不用 Pico-ITX 或第三方包替代 |
| 布局与容量 | 现场分区/GPT/MTD 与包内描述逐项匹配；每个输入不超过目标边界 | 不按同名分区或示例偏移写入 |
| 备份 | 旧内容可以完整读出、离板保存、验证可读，并能映射回原目标 | 没有可恢复备份时不覆盖现有内容 |
| 恢复入口 | 已证明可重新进入 download/recovery，Host 可识别，且兼容恢复包和工具可用 | 不把按键说明或历史启动当作恢复测试 |
| 供电与中断 | 电源、线缆、Host 睡眠/重启、超时和工具取消行为已明确 | 无法保证操作窗口时不开始 |
| 单一变化 | 本轮只改变一个经批准的组件或一个完整且自洽的整盘包 | 不把多个独立组件更新合并为一次诊断 |
| 写后判据 | 预先定义写入层、重启层、首字节、平台、最小功能和 workload 的独立成功信号 | 不以工具显示完成代替启动成功 |

目标确认必须由可观察的板上与 Host 信息共同完成，不能只依赖设备路径、分区名称、文件名或列表顺序。本文不规定具体身份字段，因为当前 CoM260 证据未闭合。

## 3. 四类路径对比

| 路径 | Actor 与输入 | 持久目标 | 适用前提 | 中断边界 | 写后与恢复边界 |
| --- | --- | --- | --- | --- | --- |
| Fastboot 分区写入 | Host Fastboot + 单个分区镜像 | U-Boot Fastboot 暴露的 GPT/MTD/逻辑分区 | 目标唯一；当前命令、分区、介质、大小和包绑定已证 | 传输成功前无写入结论；写入开始后的断连可能留下部分内容 | 先验证该分区，再复位分层观察；失败只恢复该目标 |
| Titan 包/目录 | Host Titan 工具 + 官方 ZIP、归档或展开目录 | 包描述的多个固件/文件系统目标 | 工具版本、驱动、板型、下载入口、包清单和覆盖范围已证 | 工具的原子性、重试和断电语义必须由对应版本说明给出 | 需要整包恢复路径；不能把 UI 完成视为各分区或启动成功 |
| SD 整盘镜像 | Host 镜像写入工具 + 单个 whole-disk image | 经人工确认的可移除块设备全盘 | 镜像含完整布局；容量足够；CoM260 可选择该 SD 启动；Host 设备唯一 | 中断会使 GPT、固件或文件系统不完整 | 写后先离板检查布局，再以明确 boot selection 测试；原板载介质保持不变 |
| 单组件更新 | Host stage + U-Boot/目标写入动作 + 一个 OpenSBI、ESOS 或其他组件 | 一个已证 MTD/GPT 分区 | RAM baseline 已通过；组件生产链、依赖、分区、备份与恢复均已证 | 擦除后到完整写入前是不可启动窗口 | 只验证该组件及原 local boot/RAM baseline；失败不连带更新其他固件 |

四类路径不能混用成功信号。Fastboot 传输完成、Titan UI 完成、Host 块写完成或目标分区写完，都只证明各自当前层的状态。

## 4. 持久动作属性

下表没有可复制命令，只记录来源中的动作类别。所有动作均为“未执行”，当前 CoM260 未满足共同门。

| 动作类别 | 来源 | Actor / 环境 | 状态变更类别 | 执行状态 | 前置条件 | 成功信号 | 失败信号 | 停止条件 | 目标确认 | 恢复要求 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 向一个命名分区写入对应镜像 | 官方 docs-buildroot；Pico-ITX `fastboot.yaml` | Host Fastboot + U-Boot Fastboot | 覆盖一个持久分区 | 未执行 | 共同门全部通过；命令语法、镜像和分区绑定已证 | Host 与 U-Boot 均报告完整写入，且只影响预期分区 | 设备消失、短写、容量/分区错误或目标重启 | 任一身份、布局、备份或恢复字段未知 | 板型、介质、分区枚举、大小和目标唯一性共同确认 | 原分区备份与相同入口下的单分区恢复已验证 |
| 应用 Titan ZIP、归档或目录 | 官方 image 文档；Pico-ITX README/`image_flash.py` | Host Titan 工具 + download-mode 目标 | 可能覆盖多个固件和文件系统 | 未执行 | 精确工具版本、驱动、包格式、manifest、目标板和覆盖清单已证 | 工具逐目标报告完成，写后布局与包清单一致 | 包拒绝、设备断连、任一目标失败或状态不明 | 工具/包/目标/中断语义任一未知 | 工具显示身份必须与板上身份和官方包范围一致 | 同版本工具、官方完整恢复包和 download 入口已验证 |
| 把 whole-disk image 写入 SD | 官方 K3/Pico-ITX 整盘镜像机制 | Host + 可移除块设备 | 重写 GPT、固件区与文件系统 | 未执行 | 镜像格式、容量、完整布局、Host 设备和目标板 SD boot 已证 | Host 完整写入且离板重读的布局/容量符合镜像 | 设备选择变化、空间不足、I/O 错误或中断 | 设备不唯一、不可丢弃、挂载中或 boot selection 未知 | 人工核对物理介质、设备路径、容量、挂载与可丢弃性 | 不改板载介质；保留原卡镜像或可重建的官方包 |
| 暂存并替换一个 OpenSBI/ESOS 类组件 | 固定 revision `others/Rt-Async-AMP/README.md` | Host Fastboot + U-Boot shell | 擦除并重写一个固件分区 | 未执行 | RAM boot 已过目标层；FIT、依赖、分区、容量、备份和恢复已证 | 单分区写入完成，复位后原 local boot 与 RAM baseline 均达到预定层 | 擦除后失败、写入不完整、无首字节或平台事实改变 | 任何字段未知；不得同时替换第二组件 | 现场 MTD/GPT 枚举、介质、边界和组件名共同确认 | 可从 download mode 恢复原分区；原镜像已离板可读 |

## 5. Fastboot 分区写入

Fastboot 是传输与目标写入接口，不是分区事实来源。只有目标 U-Boot 的实际分区暴露、官方目标包和介质布局三者一致时，命名分区才可进入候选。Pico-ITX `fastboot.yaml` 和 JSON 只证明该板型的动作与布局可以关联，不证明 CoM260 存在相同名称、大小或 payload。

状态链为：唯一设备 → 进入已证 U-Boot Fastboot → 选择一个已证镜像/分区对 → 传输 → 目标写入 → 读取状态/布局 → 复位 → 分层启动检查。任一层失败停在该层；不把失败归因于另一个尚未改变的固件。

## 6. Titan 包与目录

官方资料把 K3 Buildroot ZIP 与 Titan 关联，Pico-ITX 仓库还能生成 Titan 目录归档。两者不建立共同的 CoM260 包合同。开始前必须知道 Titan 的精确版本和主机要求、包入口形态、manifest/动作文件、目标板与介质、覆盖清单、容量检查、断线/断电恢复语义和官方恢复包。

Titan 通常可能跨多个目标，不能套用“单组件”诊断。若包本身是完整且不可拆分的官方恢复单元，必须按整包验收；否则不能为了方便把多个独立固件组合为一次更新。

## 7. SD 整盘镜像

whole-disk image、单分区镜像和文件系统内文件是三种对象。整盘写入会替换目标 SD 的分区表与内容；Host 块设备误选会破坏无关数据。写入前必须人工确认可移除介质、设备路径、容量、挂载状态和可丢弃性，并确认镜像包含目标板启动所需的固件、分区、DTB、环境和 payload。

K3 支持 SD 启动不等于 CoM260 当前 boot pin/优先级会选择该卡。写后先离板重读布局，再单独验证 boot selection；板载介质必须保持原状，才能作为失败时的独立回退。

## 8. OpenSBI、ESOS 与其他单组件更新

固定第三方流程把 `opensbi.itb` 或 `esos.itb` 暂存到 RAM 后写入同名 MTD 分区。这只说明实现意图。组件更新前还需证明生产 revision、FIT 内容、加载者/消费者、ABI/共享内存依赖、当前分区内容、目标大小、备份和恢复。

顺序约束是一次只改变一个组件：先保留 FSBL、bootinfo、GPT 和 U-Boot；用已接受的 RAM 路径证明 AP payload；只有实质需求和完整恢复证据同时存在时，才评估一个 OpenSBI 或 ESOS 组件。写后必须同时复核原 local boot 和 RAM baseline。任一退化先恢复该组件，不继续修改第二个组件。

## 9. 中断处理与恢复状态

| 中断点 | 可知状态 | 不得声称 | 恢复方向 |
| --- | --- | --- | --- |
| 写入前 | 只完成只读确认 | 目标已改变 | 安全退出，保留原状态 |
| 传输中 | 数据可能未完整到达 | 目标一定未开始写入 | 依具体工具协议判断；状态未知则进入恢复模式，不重试其他路径 |
| 擦除后、写完前 | 目标组件或布局可能不可用 | 复位必然恢复 | 保持供电时按工具文档恢复；断电后从已证 download/recovery 入口恢复原内容 |
| 工具报告完成后 | 当前工具层报告成功 | 分区内容、启动或 workload 已成功 | 只读核对目标与布局，再进入分层复位检查 |
| 首次复位失败 | 最早失败层已观察 | 应更新更多组件 | 恢复本次唯一变化，验证原基线，再分析该层 |

自动重试、断点续传、回滚或原子性只有在当前工具版本和目标协议明确保证时才能写入操作合同；本文没有这类证据。

## 10. 写后分层判据

| 层 | 成功信号 | 失败时边界 |
| --- | --- | --- |
| 0 工具/写入 | 工具和目标都报告预期目标完成 | 只处理连接、目标、容量和写入状态 |
| 1 布局/内容 | 只读枚举与预期分区、大小、类型和组件一致 | 恢复唯一变化，不重写其他目标 |
| 2 BootROM/FSBL | 出现原有早期启动阶段 | 排查 bootinfo、FSBL、介质与本次覆盖范围 |
| 3 ESOS/OpenSBI/U-Boot | 各阶段按原顺序出现并保持兼容 | 只定位本次组件及其直接依赖 |
| 4 payload 首字节 | 预期 payload 在已证 UART 输出 | 回到 RAM/DTB/入口边界，不升级其他固件 |
| 5 平台与最小功能 | hart、内存、中断、timer、console、存储与目标事实一致 | 停在最早失败子系统 |
| 6 workload | 明确定义的 workload 和重启/掉电恢复通过 | 不把单次 boot 当作持久稳定性结论 |

## 11. 未知项闭包

### U1. CoM260 官方包、介质与分区布局

- 当前证据: K3 SDK 提供通用布局，Pico-ITX 仓库提供另一板型的 JSON/gadget；当前 CoM260 官方包和现场布局未对齐。
- 禁止推断: 不由分区同名、通用 K3 支持或 Pico-ITX 示例补出 CoM260 偏移、大小、payload 或介质。
- 解除条件: 官方 CoM260 release/manifest、包内布局和目标板只读枚举逐项一致。
- 影响主题: Fastboot、Titan、SD、单组件更新与全部恢复路径。

### U2. Titan 版本、包合同与中断语义

- 当前证据: 官方资料说明 Titan/ZIP 机制；Pico-ITX 工具能生成 Titan 目录归档。本项目没有当前 CoM260 工具版本或运行结果。
- 禁止推断: 不把任意 ZIP、目录、JSON 或 UI 选项视为兼容；不假定重试、回滚或断点续传。
- 解除条件: 当前 Titan 官方手册、精确工具版本、CoM260 包说明、覆盖清单和中断恢复测试形成同一合同。
- 影响主题: Titan 路径、整包恢复和量产判断。

### U3. SD 镜像与启动选择

- 当前证据: K3 支持 SD，Pico-ITX 有整盘 Ubuntu 镜像；当前没有 CoM260 StarryOS/Buildroot 完整 SD 镜像或 boot selection 证据。
- 禁止推断: 不由 SoC 能力推定目标板会从 SD 启动；不把单分区 payload 当作 whole-disk image。
- 解除条件: 官方或经批准生成的完整 CoM260 镜像、布局检查、目标板 boot pin/优先级证据和可恢复启动测试一致。
- 影响主题: Host 块设备门、SD 写后检查和板载介质回退。

### U4. 组件备份、兼容与恢复入口

- 当前证据: 固定第三方描述 OpenSBI/ESOS 写入；R22 未执行备份、写入或恢复，当前目标分区和固件依赖未知。
- 禁止推断: 不由 FIT 可解析、RAM boot 成功或分区同名推定组件可替换；不把历史 FEL/RESET 说明当作恢复验证。
- 解除条件: 当前组件可读备份、生产/消费兼容关系、目标分区、精确恢复包、download 入口和恢复后的分层启动均已验证。
- 影响主题: 单组件顺序、写后失败、OpenSBI/ESOS 与原 local boot。

## 12. 使用边界

- 本文不授权任何写入；共同门未全部闭合时，四条路径都保持不可执行。
- 任何成功只覆盖实际观察到的当前层，不能替代后续启动或 workload 证据。
- 持久失败只恢复本次唯一变化，不以扩大固件或布局改动作为捷径。
- 实际执行若获单独批准，应由基于已验证现场事实的 Runbook 承担；本 change 不创建 Runbook。
