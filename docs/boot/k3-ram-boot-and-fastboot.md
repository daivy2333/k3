> 来源: https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/boot.md（源端修订: unknown；观察日期: 2026-09-02；partially-observed）；https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/boot.md（源端修订: unknown；观察日期: 2026-09-07）；https://github.com/spacemit-com/K3-Ubuntu-Images/blob/main/README.rst（源端修订: branch main；观察日期: 2026-09-14；K3 Pico-ITX）；https://github.com/spacemit-com/K3-Ubuntu-Images/blob/main/fastboot.yaml（源端修订: branch main；观察日期: 2026-09-14；K3 Pico-ITX）

# K3 RAM 引导与 Fastboot 暂存

本文是决策基线，不是可直接执行的 Runbook。所有命令均来自官方 K3 通用资料、Pico-ITX 官方仓库或固定 revision 第三方实现，本项目未执行命令、未连接设备，也未验证 CoM260 的语法、地址、DTB 或启动结果。缺少任一前置证据时，流程停在当前层，不写持久介质。

产物身份和七种位置名称空间见[镜像构建与产物关系](k3-image-build-and-artifacts.md)，download/local boot 与 Fastboot 入口见[CoM260 启动链](com260-boot-chain.md)，DTB 和内存未知项见[镜像与 DTS](com260-image-and-dts.md)。固定第三方序列及历史环境限制来自 [R22](../../.claude/analysis/k3-image-flashing-and-real-board-bringup.md)。

## 1. 证据与状态边界

| 类别 | 本文可用结论 | 不能证明 |
| --- | --- | --- |
| 官方 K3 文档 | download/local boot 的区别；BROM-Fastboot 只负责加载 U-Boot Fastboot；U-Boot shell 可进入 Fastboot | CoM260 按键位置、当前固件语法、目标 RAM 范围或 StarryOS 可启动 |
| 官方 Pico-ITX 仓库 | BootROM 临时加载 FSBL/U-Boot 后由 U-Boot Fastboot 服务接管的机制 | Pico-ITX 临时固件、动作文件或设备选择适用于 CoM260 |
| 固定 revision 第三方 | `starryos.uimg` 的 stage/`bootm` 序列及示例地址关系 | 官方支持、当前板型匹配、命令成功或地址安全 |
| 历史板上证据 | R18 曾到达 Bianbu root shell 并观察到串口输出 | 当前 local boot、Fastboot、reset 恢复或新 payload 已验证 |
| 未知项 | 把缺少的板型、命令、地址、DTB 与恢复事实显式阻塞 | 可以用示例值补齐 CoM260 操作参数 |

本文中的“临时”只表示预期状态位于 RAM。BootROM 传输内存、U-Boot Fastboot 上传缓冲区、FIT `load`、FIT `entry` 是不同名称空间；RAM 操作也必须先证明范围和生命周期安全。

## 2. 前置条件与停止矩阵

| 检查 | 继续所需证据 | 缺失时停止位置 | 原因 |
| --- | --- | --- | --- |
| 板与恢复基线 | 精确板型/变体、稳定串口、当前 local boot 可恢复、复位路径已确认 | 进入任何 Fastboot 前 | 失去控制台或基线后无法区分传输故障与板级故障 |
| 入口能力 | 当前固件实际支持所选入口；download boot 所需按键/USB 路径已由目标板资料确认 | 切换启动状态前 | K3 通用入口不证明 CoM260 物理实现 |
| 主机设备身份 | 主机只枚举一个经板上信息交叉确认的目标 | Host 传输前 | 设备列表为空或不唯一时不能选择目标 |
| U-Boot 命令能力 | 当前 U-Boot help/输出证明 Fastboot stage、参数和 FIT 检查/启动命令存在 | 输入 stage 服务命令前 | 官方通用命令与第三方语法可能不匹配当前固件 |
| 镜像身份 | FIT 来源、大小、配置、kernel/FDT、hash、`load`、`entry` 已离板检查 | Host 传输前 | 文件存在不证明 FIT 可解析或属于目标板 |
| RAM 安全 | 上传区间和每个 FIT 子镜像最终区间均在已证 DRAM 内，避开 U-Boot、自身容器、reserved-memory、CMA、设备窗口及彼此 | stage 或 `bootm` 前 | 重叠可在解析或搬运时覆盖容器、固件或运行内存 |
| DTB 适用性 | model/compatible、内存、reserved-memory、UART、PLIC/APLIC/IMSIC 与目标变体一致 | `bootm` 前 | FIT 可解析不等于板级匹配 |
| 回退 | 无持久修改，且复位后能回到原 local boot；失败时的串口观察点已定义 | 跳转前 | 无恢复证据时不能把复位当作已验证回退 |

任一检查失败只允许补证或回到原基线。RAM boot 失败不授权 Fastboot 分区写入、MTD 擦写、整盘写入或固件升级。

## 3. 三种入口不是一条路径

| 入口 | 初始状态 | 参与者与状态变化 | 持久状态 | 适用边界 |
| --- | --- | --- | --- | --- |
| BROM-Fastboot/download boot | 目标进入 BootROM 下载模式 | Host → BootROM 传输内存 → 临时 FSBL/U-Boot → U-Boot Fastboot 服务 | 不应写分区；临时固件驻 RAM | 官方 K3/Pico-ITX 机制；CoM260 按键、临时镜像与命令仍未知 |
| 已运行 OS 请求 bootloader | local boot 已到 OS | OS 发出重启请求 → U-Boot Fastboot | 原有固件仍在持久介质 | 某些固件没有 ADB；不是 SoC download boot |
| 串口进入本地 U-Boot | local boot 启动中且串口可控 | 串口中断 autoboot → 本地 U-Boot shell → U-Boot Fastboot | 原有固件仍在持久介质 | 官方 K3 通用入口；按键时机和命令支持需现场确认 |

BROM-Fastboot 的 BootROM 传输内存不等于稍后 `fastboot stage` 使用的上传缓冲区。临时 U-Boot 也不等于 local boot 固件分区里的 U-Boot 或 Pico-ITX `uboot` 分区中的 EDK2。

## 4. 共同的易失状态机

```text
已证 local boot 或已证 download-boot 恢复入口
  → 进入 U-Boot Fastboot 服务
  → Host 唯一识别目标
  → FIT 暂存到已证上传缓冲区
  → 退出 Fastboot，回到 U-Boot shell
  → 检查 FIT 配置、大小、hash、load、entry 与 FDT
  → 检查上传区和全部最终区间不重叠
  → 在 DTB 与复位门通过后决定是否跳转
  → 分层观察解析、跳转、首字节、平台、最小功能和 workload
  → 无论成功或失败，复位并验证原 local boot 基线
```

状态所有者依次是 Host、BootROM 或既有 U-Boot、U-Boot Fastboot 服务、目标 RAM、FIT 解析器、payload 和复位后的原启动链。Host 的传输成功只证明数据交给 Fastboot；U-Boot 的 FIT 解析成功只证明容器可读；两者都不证明已执行 payload。

## 5. 命令来源与属性

下表是来源对照，不是执行清单。每行都标记为“未执行”；变量、设备选择和示例地址不得直接替换为猜测值。

| 命令或动作 | 来源 | Actor / 环境 | 状态变更类别 | 执行状态 | 前置条件 | 成功信号 | 失败信号 | 停止条件 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `adb reboot bootloader` | 官方 docs-buildroot boot.md | Host + 已运行且启用 ADB 的目标 OS | 易失重启，进入既有 U-Boot Fastboot | 未执行 | 唯一设备、ADB 可用、串口与 local boot 回退已证 | 目标重启并在串口/Host 两侧显示 U-Boot Fastboot | ADB 不存在、目标不唯一、重启后无控制台 | 任一前置条件或 Fastboot 身份不成立 |
| `fastboot 0` | 官方 docs-buildroot boot.md | 目标串口中的本地 U-Boot shell | 启动易失 Fastboot 服务 | 未执行 | 已中断 autoboot；当前 help 支持该语法；USB 与回退已证 | U-Boot 显示 Fastboot 服务且 Host 唯一发现目标 | 未知命令、USB 无枚举、控制台丢失 | 不改用第三方参数猜测；复位回原 local boot |
| `fastboot -l 0x180000000 -s 0x04000000 usb 0` | 固定 revision `others/Rt-Async-AMP/README.md` | 目标 U-Boot shell | 建立固定第三方示例的 RAM stage 窗口 | 未执行 | 当前 U-Boot 支持 `-l/-s`；实际 FIT 大小和完整 RAM 图证明该范围安全 | U-Boot 进入等待 stage 状态 | 语法拒绝、范围无证据、与最终 `load` 区间冲突 | 地址、大小或语法未由目标板验证；不得把示例当默认值 |
| `fastboot stage build/k3-com260/starryos.uimg` | 固定 revision `others/Rt-Async-AMP/README.md` | Host Fastboot 客户端 | Host 文件传入目标 RAM，不写持久介质 | 未执行 | 唯一目标；文件身份、大小、FIT 元数据和 U-Boot stage 窗口已证 | Host 报告 stage 传输成功，U-Boot 仍可控 | 设备消失、超时、传输错误、大小超限 | 不唯一、失败或 U-Boot 输出异常时停止；不改用 flash 子命令 |
| `Ctrl+C` 退出 Fastboot | 固定 revision `others/Rt-Async-AMP/README.md` | 目标串口 / U-Boot Fastboot 服务 | 结束易失服务，返回 shell | 未执行 | stage 已明确结束且串口仍连接 | 出现 U-Boot prompt，暂存 RAM 未被重用 | 无 prompt、复位、hang 或内存状态未知 | 未返回受控 shell 时不得解析或跳转 |
| `bootm 0x180000000` | 固定 revision `others/Rt-Async-AMP/README.md` | 目标 U-Boot shell | 解析暂存 FIT、搬运子镜像并跳转 | 未执行 | FIT/hash/config、上传区、所有 `load`/`entry`、DTB 和复位门全部通过 | U-Boot 依次显示所选 config、子镜像校验/搬运和跳转；随后单独观察首字节 | FIT/校验/重叠错误、错误 config、无跳转或异常 | 任一元数据、地址、DTB 或恢复事实未知；该地址只属固定第三方示例 |

官方 Pico-ITX `fastboot.yaml` 还展示 BootROM/临时 FSBL/U-Boot 与后续 U-Boot Fastboot 的分层，但本文不复制其目标板动作，也不把 Pico-ITX 临时镜像用于 CoM260。

## 6. 地址、大小与 FIT 门

固定第三方示例包含三组不同的 RAM 语义：上传缓冲区起点 `0x180000000`、容量示例 `0x04000000`；kernel `load/entry = 0x140000000`；FDT `load = 0x138000000`。这些值只用来说明检查方法。

执行判断必须基于实际 FIT 和目标板内存图，至少形成以下区间：

- 上传容器 `[stage_start, stage_start + actual_fit_size)`，其中 `actual_fit_size <= stage_capacity`；
- 每个 FIT image 的 `[load, load + decompressed_or_loaded_size)`；压缩 payload 不能只用压缩文件大小估算最终范围；
- U-Boot、栈、malloc、临时解压区、DTB、reserved-memory、CMA 和其他活动固件范围。

所有区间必须位于已确认 DRAM 内，且容器在 U-Boot 读取和搬运完成前不能被任何目标区间覆盖。上传起点不同于 kernel `entry`；FDT 没有可类比 kernel 的执行入口。相关 cache、PMA 与地址转换边界见 [K3 DMA 与内存所有权](../dma/k3-dma-and-memory-ownership.md) 和 [cache/PMA/地址转换](../dma/k3-cache-pma-address-translation.md)，它们不替代 bootloader 实际内存图。

FIT 门还需核对 image 类型、架构、压缩、hash、配置选择、kernel/FDT 引用、`load` 和 `entry`。固定示例中的 `os = "linux"` 只是 FIT 元数据；不得据此改变 payload 身份。

## 7. DTB 与第一字节门

固定第三方 FIT 使用 IFX 命名 DTB，而当前 Kit 与 CoM260 候选 DTS 尚未唯一映射。跳转前必须核对 model/compatible、DRAM、reserved-memory、console/UART、PLIC/APLIC/IMSIC、chosen/bootargs 和启动所需设备；详情保持由[镜像与 DTS](com260-image-and-dts.md)的 G7 负责。

串口物理与参数见 [CoM260 UART](../serial/com260-uart.md)。串口无输出可能位于 console/波特率、入口、异常、DTB 或 payload 层，不能单凭 Host stage 成功归因。

## 8. 分层成功、失败与回退

| 层 | 成功信号 | 失败边界 | 安全回退 |
| --- | --- | --- | --- |
| 0 原基线 | 原 local boot 和串口可恢复 | 原系统已不能稳定启动 | 停止新镜像评估，先恢复基线 |
| 1 入口/发现 | 进入预期 U-Boot Fastboot；Host 唯一识别同一目标 | 入口错误、无设备、多设备或身份不一致 | 退出或复位，不传输 |
| 2 stage | Host 与 U-Boot 均确认完整传输，返回受控 shell | 超时、短传、大小错误、prompt 未恢复 | 复位；不调用任何持久写命令 |
| 3 FIT parse | config、hash、kernel/FDT、load/entry 与区间检查通过 | 格式、hash、config、重叠或范围失败 | 停在 U-Boot 或复位；不跳转 |
| 4 jump | U-Boot 明确交接到预期 entry | `bootm` 拒绝、异常或跳转目标不明 | 复位回原 local boot |
| 5 首字节 | 预期 UART 出现 payload 独有首条输出 | 无输出、乱码、早期 trap | 只排查入口、console、异常与 DTB，不更新固件 |
| 6 平台事实 | hart、内存、DTB、时钟和中断控制器与目标一致 | 任一平台事实不符 | 复位；返回对应平台/DTB 文档补证 |
| 7 最小功能 | timer、异常、console、内存等最小集合按预期工作 | 设备、IRQ、DMA 或内存错误 | 停在最早失败子系统，不扩大 workload |
| 8 workload | 明确定义的 workload 通过 | 功能、稳定性或资源错误 | 保留原持久介质，复位回 local boot |

后一层成功不能反向证明前一层之外的板级兼容性。任何层失败都不能以更新 OpenSBI、ESOS、U-Boot、分区表或 rootfs 作为捷径。

## 9. 未知项闭包

### U1. 当前 CoM260 的入口与命令语法

- 当前证据: 官方资料给出 K3 通用入口，固定第三方给出另一组 U-Boot Fastboot 参数；本项目未读取目标板当前 U-Boot help。
- 禁止推断: 不把 `fastboot 0`、`-l/-s` 参数、ADB 或 Pico-ITX download 动作视为当前 CoM260 已支持。
- 解除条件: 目标板串口记录、当前 U-Boot help/版本和 Host 枚举结果共同确认入口、语法与唯一设备。
- 影响主题: stage 前置门、命令表、persistent recovery 的下载入口。

### U2. 安全 RAM 区间与 FIT 实际大小

- 当前证据: 固定第三方示例给出上传、kernel 和 FDT 地址；现有 DTS 候选与目标 Kit 未唯一映射，U-Boot 活动区和实际 FIT 大小未知。
- 禁止推断: 不把 `0x180000000`、`0x04000000`、`0x140000000` 或 `0x138000000` 写成 CoM260 默认值；不以压缩大小代替加载大小。
- 解除条件: 目标板 DRAM/保留区、U-Boot 内存使用、实际 FIT 元数据和各 payload 加载大小形成无重叠区间证明。
- 影响主题: stage、FIT parse、jump 与首字节。

### U3. FIT 配置和 DTB 对应目标变体

- 当前证据: 固定 FIT 指向 IFX 命名 DTB；官方 CoM260 候选集合尚无当前 Kit 的唯一映射。
- 禁止推断: 不由文件名、FIT hash 通过或 `bootm` 能解析推定板型、内存、UART 或中断拓扑正确。
- 解除条件: 板上产品身份、官方目标 DTS/DTB、FIT config 和启动日志中的 model/compatible 一致。
- 影响主题: FIT 门、第一字节、平台事实与后续最小功能。

### U4. 复位后的 local-boot 恢复

- 当前证据: R18 是历史启动记录；本项目未在本次 change 中复位或验证当前持久固件。
- 禁止推断: 不因 RAM boot 理论上不写盘就声称复位必然恢复；不把历史 root shell 当作当前恢复测试。
- 解除条件: RAM 操作前后使用同一串口和电源/复位路径，观察原固件按既定阶段回到已知 local-boot 成功点。
- 影响主题: 跳转授权、每层安全回退和 persistent recovery 前置条件。

## 10. 使用边界

- 本文不执行或授权命令，不提供持久写流程。
- 未知项未解除时，只能补充只读证据，不能继续 stage 或跳转。
- RAM boot 的成功范围止于实际通过的最高层；它不证明持久部署安全。
- 后续持久路径必须重新检查目标身份、包/介质/分区、备份、恢复和中断处理，不能继承本节的易失假设。
