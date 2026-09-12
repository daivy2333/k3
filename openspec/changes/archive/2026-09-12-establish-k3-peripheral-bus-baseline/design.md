## Context

见 [proposal.md](proposal.md) 的动机与批准范围。仓库目前没有 `docs/buses/` 正文；[总索引](../../../docs/index.md)把五个目标入口标为待聚合，[来源覆盖表](../../../docs/reference/source-coverage.md)中的对应 URL 均为 `future / deferred / partially-observed`，源端修订为 `unknown`。

当前可审计材料分为三层：K3 datasheet 提供控制器能力；官方 Linux `k3-br-v1.0.y` DTS 提供实例、资源和 CoM260 变体；既有 MS03/MS04/MS05/MS07 文档提供板级连接、中断和 GMAC 边界。五个 Buildroot 操作页及 USB 子树在当前环境不可直接读取，因此不能提供驱动命令或运行结果。

调查还发现一项来源冲突：既有中文聚合记录 9 路 I²C，当前官方英文 K3 datasheet 写“最多 10 路”。本 change 保留冲突，不静默刷新既有 SoC 概述。

## Goals / Non-Goals

**Goals:**

- 用统一链路表达 `SoC 能力 → 控制器资源 → DTS 状态 → 模组/Kit 连接 → 软件或运行边界`。
- 让每类总线的静态事实、官方软件行为、推论和未知项可独立检索。
- 复用既有平台、中断、DMA 和网络基线，避免重复定义。

**Non-Goals:**

- 不补写无法直接观察的 Buildroot 操作步骤或真板行为。
- 不通过修改既有 MS03-MS09 正文裁决来源冲突。
- 不为文档验证新增脚本、抓取器、manifest 或 Evidence 目录。

## Decisions

### D1：三篇正文按依赖和故障域拆分

创建 `k3-i2c-and-usb.md`、`k3-pcie-and-can.md`、`k3-ethercat.md`。I2C 与 USB 共享 Type-C 控制器和板载设备依赖；PCIe 与 CAN 都需要把控制器、引脚/PHY 或收发器、连接器和运行状态分层；EtherCAT 直接依赖 MS07，单独成文可避免把 GMAC 静态链提升为协议运行结论。替代方案是五篇单总线文档，但当前直接来源不足，容易形成短小空壳；全部合并则跨越三个独立故障域。

### D2：来源不可达是正文边界，不是实施阻塞

Iteration 000 把五个官方入口调整为当前 MS10 职责，同时保留 `partially-observed`、`unknown` 和 USB 子树未展开事实。正文只使用已直接观察的官方 datasheet、Linux DTS、产品资料和既有主题；Buildroot 专属命令、测试和恢复语义进入未知项。若 Act 能直接打开页面，可在同一证据规则下补充；若内容与计划契约冲突则停止返回 Plan。

### D3：I²C 数量冲突并列保留

正文同时记录既有中文资料的 9 路和当前英文 datasheet 的“最多 10 路”，再用 DTS 实例表说明已观察节点；三者不互相补值。替代方案是选择最新英文数值并修改 SoC 概述，但那属于 source refresh，超出本 change。

### D4：USB/PCIe 共享 PHY 必须显式建模

K3 datasheet 说明 USB Port B/C/D 的 SuperSpeed PHY 与 PCIe 共享、同一时刻只能选择一种功能。USB 和 PCIe 正文必须交叉引用该互斥关系，并区分 Port A DRD、Host port、PHY、role switch 与板级连接。只列两个独立控制器表会隐藏关键资源冲突，因此不采用。

### D5：CAN 和 EtherCAT 不以静态节点证明运行

CAN 正文分开 AP/RP FlexCAN、CAN-FD 能力和 Kit 收发器冲突。EtherCAT 正文只把 `ec_master → eth1` 作为静态依赖，引用 MS07 的 GMAC1/PHY/DMA 事实；周期、同步、错误恢复和真板状态保持未知。

### D6：验证直接检查知识产物

测试见证使用目标文件缺失、来源仍 deferred、索引无入口等当前 RED。GREEN 检查首行来源、必需章节、证据等级、四字段未知项、相对链接、URL 唯一、术语与计数一致，以及 OpenSpec 严格校验。验证结果写入 Act Response，Persisted Evidence 为 `none`。

## Risks / Trade-offs

- [Buildroot 入口不可达] → 保留权威 URL 和访问状态，仅使用已核对 supporting source，不编造操作行为。
- [英文 datasheet 与既有中文数量冲突] → 本 change 并列记录；若要更新全局 SoC 基线，另建 refresh change。
- [CoM260 DTS 变体不唯一] → 每项资源标明变体，沿用 G7，不选择默认板型。
- [三篇文档仍可能过长] → 仅在正文超过 M02 的 500 行建议上限时按单总线继续拆分，requirement 和 Iteration 边界不变。
- [EtherCAT 内容较少] → 保留独立文档，因为其协议证据、实时性和错误域与 GMAC 静态基线不同。

## Migration Plan

按 Iteration 依次建立来源与 I2C/USB、PCIe/CAN、EtherCAT，最后统一更新缺口、术语和索引。任一 Iteration 未通过 Review 时不展开下一轮。回滚只需撤销本 change 新增或修改的 Markdown；不涉及数据迁移或运行状态。
