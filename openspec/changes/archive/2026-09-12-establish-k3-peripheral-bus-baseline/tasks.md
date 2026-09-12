## 1. 来源与 I2C/USB

- [x] 1.1 更新 `docs/reference/source-coverage.md` 中五个 MS10 官方入口的当前职责和可达边界，保持 URL 唯一；验证五行不再是 `future / deferred`，USB 目录未展开及源端修订 `unknown` 仍被准确记录。
- [x] 1.2 创建 `docs/buses/k3-i2c-and-usb.md`，记录 I2C 实例/client、I²C 数量冲突、USB controller/PHY/Host/DRD/role switch/Hub、共享 PHY、板级映射和未知项；验证首行来源、对象分层、变体差异、四字段未知项及相对链接有效。

## 2. PCIe 与 CAN

- [x] 2.1 创建 `docs/buses/k3-pcie-and-can.md`，记录 PCIe controller/lane/PHY/RC/EP/插槽与 AP/RP FlexCAN/CAN-FD/收发器边界；验证静态拓扑不被表述为枚举或通信成功，CAN 板级冲突完整保留，四字段未知项齐全。

## 3. EtherCAT

- [x] 3.1 创建 `docs/buses/k3-ethercat.md`，记录 `ec_master → eth1` 静态依赖、软件与协议边界并引用 MS07；验证正文不重复 GMAC/DMA 数据路径，不声明未经验证的周期、同步精度、恢复或真板结果。

## 4. 导航与汇总

- [x] 4.1 更新 `docs/reference/known-gaps.md`，把三篇正文未知项映射到 G7 和既有相关缺口，并仅在存在独立总线缺口时新增递增 G 条目；验证四字段、来源对应、状态汇总和影响主题一致。
- [x] 4.2 更新 `docs/reference/terminology.md`，增加正文实际使用且现有表缺失的总线术语；验证主写法无重复、定义标明对象范围和正文位置。
- [x] 4.3 更新 `docs/index.md`，加入三篇总线正文入口并同步 `docs/buses/` 状态、来源数量和缺口计数；验证全部相对链接有效，计数与覆盖表及 known-gaps 一致，并运行 `git diff --check` 与 `openspec validate establish-k3-peripheral-bus-baseline --strict`。

## Iteration Plan

### Iteration 000: 来源与 I2C/USB 基线

- Tasks: 1.1, 1.2
- Depends on: None
- Stable baseline: 五个 MS10 来源具有当前职责，I2C 与 USB 的控制器、设备、PHY、角色和板级关系有独立正文。
- Verification boundary: 覆盖表五行状态和 URL 唯一性正确；`k3-i2c-and-usb.md` 覆盖资源分层、数量冲突、共享 PHY、变体及未知项；相对链接有效。
- Diagnostic boundary: 来源状态、I2C controller/client、USB controller/PHY/role/Hub、I²C 数量冲突和 USB/PCIe PHY 互斥。
- Non-goals: 不写 PCIe/CAN/EtherCAT 正文，不更新最终术语、缺口和索引。
- Balance audit: 来源责任是首篇正文的前置条件，二者合并形成可独立复用的低速控制与 USB 基线；单独执行来源会留下无正文状态，纳入 PCIe/CAN 会跨越独立故障域。

### Iteration 001: PCIe 与 CAN 基线

- Tasks: 2.1
- Depends on: Iteration 000
- Stable baseline: PCIe 静态拓扑和 CAN 控制器/协议/物理层具备独立正文，并复用上一轮的共享 PHY 边界。
- Verification boundary: `k3-pcie-and-can.md` 覆盖 PCIe lane/PHY/RC/EP/插槽、AP/RP FlexCAN 和 CAN 板级冲突；静态能力与运行结果分离。
- Diagnostic boundary: PCIe 资源与枚举边界、USB/PCIe PHY 互斥、FlexCAN 实例、CAN-FD 和收发器可用性。
- Non-goals: 不写 EtherCAT，不执行 NVMe/CAN 真板验证，不更新最终导航。
- Balance audit: PCIe 与 CAN 各自材料有限，但共同承担“控制器—物理接口—运行边界”成果；再拆会产生过细 Iteration，合入 USB 或 EtherCAT 会混合依赖和诊断边界。

### Iteration 002: EtherCAT 基线

- Tasks: 3.1
- Depends on: Iteration 001
- Stable baseline: EtherCAT master 的静态依赖、软件边界和未知运行条件可独立检索。
- Verification boundary: `k3-ethercat.md` 引用 MS07 且不重复 GMAC/DMA；静态绑定不被写成协议或实时性能结论。
- Diagnostic boundary: `ec_master`、`eth1` 依赖、软件组成、周期/同步/恢复证据缺口。
- Non-goals: 不修改既有 GMAC 正文，不实现或验证 EtherCAT 栈。
- Balance audit: 虽然只有一个任务，但 EtherCAT 有独立协议和实时性故障域，且依赖前两轮稳定的总线/网络边界；与收尾汇总合并会混合技术正文与计数导航。

### Iteration 003: 导航、术语与缺口收敛

- Tasks: 4.1, 4.2, 4.3
- Depends on: Iteration 000, Iteration 001, Iteration 002
- Stable baseline: 三篇总线正文能从总索引进入，来源、术语、缺口和计数一致，MS10 可进入最终 Review。
- Verification boundary: 链接有效、术语无重复、缺口正文与汇总一致、索引计数准确、change 严格校验通过。
- Diagnostic boundary: `known-gaps.md`、`terminology.md`、`index.md` 三个汇总表面。
- Non-goals: 不新增总线技术事实，不刷新 SNAPSHOT/tasks，不修复 change 外问题。
- Balance audit: 三项共享全部正文完成后的收敛输入；提前执行会迫使未完成主题决定术语和计数，分别拆分又不能形成稳定交付。

## Requirements Traceability Matrix

| Requirement | Scenario | Design | Task | Iteration | Code Surface | Test Witness | Simplification | Status |
|---|---|---|---|---|---|---|---|---|
| R1 资源与板级分层 | 资源可证；映射不唯一 | D2, D3 | 1.1-3.1 | 000-002 | coverage；三篇 buses 正文 | deferred rows；目标文件缺失；资源/证据断言 | None | Covered |
| R2 I2C/USB 对象边界 | 复合依赖；子树不可见 | D2-D4 | 1.1, 1.2 | 000 | coverage；`k3-i2c-and-usb.md` | I2C 数量、USB 对象、共享 PHY、unknown 断言 | None | Covered |
| R3 PCIe 静态/运行分离 | 静态映射；只有能力 | D4 | 2.1 | 001 | `k3-pcie-and-can.md` | lane/PHY/RC/EP 与非枚举断言 | None | Covered |
| R4 CAN 三层分离 | 节点与收发器；资料冲突 | D5 | 2.1 | 001 | `k3-pcie-and-can.md` | AP/RP、CAN-FD、冲突与非运行断言 | None | Covered |
| R5 EtherCAT 边界 | 静态绑定；实时证据缺失 | D5 | 3.1 | 002 | `k3-ethercat.md`；network links | 依赖链接、非重复、unknown 断言 | None | Covered |
| R6 来源可追溯 | 可读；不可访问 | D2 | 1.1, 4.1 | 000, 003 | coverage；known-gaps | URL 唯一、状态、四字段检查 | None | Covered |
| R7 导航一致 | 聚合完成；基线冲突 | D3, D6 | 4.1-4.3 | 003 | gaps、terminology、index | 链接、计数、术语、严格校验 | None | Covered |

## Task Status

- Completed Iterations: 000, 001, 002, 003
- Current Iteration: None
- Current Cycle: None
- Deferred Iterations: None
- Persisted Evidence: none
