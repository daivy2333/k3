## Why

MS03、MS04、MS05 和 MS07 已记录 K3/CoM260 的部分板级连接、平台资源、中断与 GMAC 边界，但 I2C、USB、PCIe、CAN 和 EtherCAT 仍只有来源入口或散落事实。MS10 需要把五类总线的控制器能力、DTS 资源、板级可达性和资料缺口整理为可检索知识，同时防止把 SoC 能力、静态节点或连接器存在误写为运行时可用。

本 change 由 SpacemiT K3 官方社区文档的 I2C、USB、PCIe、CAN 和 EtherCAT 章节驱动；覆盖表最近观察日期为 2026-09-02，源端修订均为 `unknown`。本 change 是既有资料的聚合与重组，不是已确认源端变化后的 refresh。

## What Changes

- 建立 I2C 与 USB 主题知识，区分控制器、从属设备、PHY、Host、DRD、role switch、Hub 和 BootROM 下载用途。
- 建立 PCIe 与 CAN 主题知识，整理实例、lane/PHY、RC/EP、pinctrl、clock/reset、IRQ、DMA、板级插槽或收发器以及静态与运行边界。
- 建立 EtherCAT 主题知识，说明 EtherCAT master 到 GMAC1 的静态依赖、软件组成和证据边界，不重复 MS07 的 GMAC/DMA 正文。
- 更新来源覆盖、总索引、术语和已知缺口，使五个官方入口、主题正文和未确认事实具有唯一落点。
- 来源不可访问、USB 子树未展开、DTS 变体不唯一、资料冲突或运行结果未知时，保留未知项和解除条件，不补写未经证实的值。

### Approved Planning Assumptions

- 正文初步拆为 `k3-i2c-and-usb.md`、`k3-pcie-and-can.md` 和 `k3-ethercat.md`；调查若证明任一文件会超过 M02 的 500 行建议上限，可在不改变需求范围的前提下继续拆分。
- 只聚合官方资料、官方 GitHub/DTS 的可审计行为和必要推论，不执行真板枚举、I/O、性能、实时性或故障注入测试。
- USB 官方入口是目录；必须先展开子页。当前环境仍不可访问时，按既有来源规则记录不可达和 supporting evidence，不以通用 Linux 或第三方材料替代 K3 官方事实。
- EtherCAT 只补充 `ec_master → eth1` 以上的软件和协议边界，复用 MS07 的 GMAC1/PHY/DMA 事实；静态节点不构成周期、同步精度或实时运行保证。
- Persisted Evidence 默认 `none`；可重跑的 Markdown、链接和 OpenSpec 校验结果写入 Act Response。

### Gate 1 Approval

- Status: PASS
- User instruction: `批准`
- Approved scope: MS10 的 I2C、USB、PCIe、CAN 和 EtherCAT 知识基线，以及本 proposal 的场景、默认假设和 Non-goals。
- Approved at: 2026-09-11

### Non-goals

- 不实现或修改 I2C、USB、PCIe、CAN、EtherCAT、GMAC、DMA、IRQ 或网络驱动。
- 不设计 Rust 总线 API，不修改 StarryOS、Linux、Buildroot 或第三方仓库。
- 不执行 USB gadget/Host、PCIe/NVMe、CAN-FD 或 EtherCAT 真板操作和性能测试。
- 不选择 G7 尚未唯一映射的 CoM260 顶层 DTS 作为默认目标。
- 不把 SoC 控制器数量、DTS 节点、连接器或收发器存在解释为当前产品版本已经可用。
- 不扩展到 K3 之外的芯片、板卡或 SpacemiT 文档章节。

## Scenario Sketch

### S1：查询控制器资源和板级可达性

- 前置状态：官方资料或 DTS 提供 I2C、USB、PCIe、CAN 或 EtherCAT 入口。
- 动作：读者查询控制器实例、MMIO、clock/reset、pinctrl、IRQ、DMA、PHY 或 CoM260 连接。
- 可观察结果：正文按 SoC、控制器、模组和 Kit 分层列出已确认事实及证据等级。
- 失败边界：字段缺失、DTS 变体不唯一或板级连接不能确认时，记录未知项，不推定默认值或可用性。

### S2：查询 I2C 与 USB 设备链

- 前置状态：DTS 出现 I2C 从属设备、USB PHY、Host、DRD、Type-C 控制器、role switch 或 Hub。
- 动作：读者查询设备从控制器到板级接口的依赖关系。
- 可观察结果：正文区分 I2C controller/client 与 USB controller/PHY/role/Hub，并记录基础款和 Kit V02 的差异。
- 失败边界：USB 子页不可访问或设备运行状态未知时，不用通用 Linux 行为补齐 K3 专属语义。

### S3：查询 PCIe 链路和端口用途

- 前置状态：SoC、DTS 或板级资料描述 PCIe lane、PHY、RC/EP 或 M.2 接口。
- 动作：读者查询端口、lane/PHY、复位/时钟请求和插槽之间的映射。
- 可观察结果：正文列出已证静态映射，并把 RC/EP 能力、NVMe/Wi-Fi 用途和运行时枚举分开。
- 失败边界：没有枚举、MSI/MSI-X、热插拔或 link-state 证据时，不声明相应功能已运行。

### S4：查询 CAN 控制器和收发器边界

- 前置状态：SoC 声明 CAN 能力，DTS 或 Kit 资料出现 FlexCAN 和板载 CAN FD 收发器。
- 动作：读者查询 AP/RP 实例、时钟、pinctrl、IRQ、协议能力和连接器可用性。
- 可观察结果：正文分开控制器、CAN-FD 协议和物理收发器，并保留 Kit 文档中的可用性冲突。
- 失败边界：控制器节点或收发器存在不能证明 OS 已启用或真板收发成功。

### S5：查询 EtherCAT 依赖与运行边界

- 前置状态：DTS 中 `ec_master` 绑定 `eth1`，既有 MS07 已描述 GMAC1/PHY/DMA。
- 动作：读者查询 EtherCAT master 的静态依赖、软件组成和运行条件。
- 可观察结果：正文引用既有 GMAC 基线，只补 EtherCAT 专属关系、来源和未知项。
- 失败边界：没有周期、同步、错误恢复或真板日志时，不声明实时性能或协议运行成功。

### S6：来源、导航和兼容性收敛

- 前置状态：五类总线正文完成，或官方页面不可访问、相互冲突。
- 动作：聚合者更新来源覆盖、术语、缺口和总索引。
- 可观察结果：URL 唯一、正文入口有效、术语和缺口计数一致，官方事实、交叉验证、推论和未知项分级明确。
- 失败边界：新材料与既有基线冲突时停止静默覆盖，返回 Plan 判断是否属于本 change 或独立 refresh。

## Capabilities

### New Capabilities

- `k3-peripheral-bus-baseline`: 规定 K3 I2C、USB、PCIe、CAN 和 EtherCAT 的控制器资源、板级连接、依赖关系、证据边界及导航要求。

### Modified Capabilities

- 无。

## Impact

- 预计新增 `docs/buses/` 下三篇主题文档，并修改 `docs/index.md`、`docs/reference/source-coverage.md`、`docs/reference/known-gaps.md` 与 `docs/reference/terminology.md`。
- 新增 change delta spec；不修改可执行代码、API、构建系统或运行时依赖。
- 复用 R04、R05、R08、MS03、MS04、MS05、MS07 和 G7；M01 指向的官方来源仍是硬件事实权威。
