> 来源: https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/22-EtherCAT.md（源端修订: unknown；观察日期: 2026-09-02）；supporting: https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260.dtsi（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-07）

# K3 EtherCAT 静态依赖与运行边界

CoM260 共享 DTS 把 `ec_master` 的 `master0` 关联到 `eth1`。该 phandle 只证明静态依赖，不证明 EtherCAT master 已 probe、网口已被独占、slave 已发现或帧已经交换。

## 1. 范围与证据

- **官方事实**：22-EtherCAT 官网入口承担 K3 EtherCAT 驱动说明的权威职责；正文尚未直接取得，源端修订保持 `unknown`。
- **交叉验证**：SpacemiT Linux `k3-br-v1.0.y` 的 `k3_com260.dtsi` 提供 `ec_master → master0 → main-device = <&eth1>` 静态绑定。
- **推论**：只描述静态对象之间的依赖方向，不补写具体 EtherCAT master 实现、协议参数或运行结果。
- **未知项**：软件组成、配置来源、slave 发现、周期、同步、错误处理和真板状态均缺少直接证据。

官网正文访问失败不表示页面不存在，也不能用页面标题或通用 EtherCAT 文档补写 K3 命令、配置和恢复语义。

## 2. CoM260 共享 DTS 的静态绑定

已观察到的结构可缩写为：

```dts
&ec_master {
    master0 {
        main-device = <&eth1>;
    };
};
```

这段配置给出以下静态关系：

| 对象 | 已观察关系 | 证据等级 | 不能推出 |
| --- | --- | --- | --- |
| `ec_master` | 共享 DTS 引用的 EtherCAT master 节点 | 交叉验证 | driver 已匹配或 probe 成功 |
| `master0` | `ec_master` 下的 master 子节点 | 交叉验证 | master 数量、实例 ID 或用户态接口 |
| `main-device` | phandle 指向 `eth1` | 交叉验证 | `eth1` 已被独占、link up 或正在交换 EtherCAT 帧 |
| `eth1` | EtherCAT master 的静态 Ethernet 设备依赖 | 交叉验证 | PHY、DMA、IRQ 和协议栈均已完成初始化 |

`k3_com260.dtsi` 是多个顶层 DTS 可继承的共享配置。该关系不选择 CoM260 Kit 的默认目标 DTB，也不证明任一具体板型已启用完整 EtherCAT 运行路径；目标 DTS 的候选与映射边界见[镜像与 DTS](../boot/com260-image-and-dts.md)。

## 3. 对象与职责边界

| 层级 | 本文确认的关系 | 详细责任位置 | 当前边界 |
| --- | --- | --- | --- |
| GMAC/PHY/RGMII | `eth1` 是静态依赖的 Ethernet 设备 | [GMAC、MDIO、PHY 与 RGMII](../network/com260-gmac-phy.md) | 不在本文重复 PHY reset、delay 或 link 初始化 |
| MAC/MTL/DMA/IRQ | EtherCAT 流量若经 `eth1` 传输，会依赖既有网络数据面 | [GMAC DMA ring 与 IRQ](../network/k3-gmac-dma-irq.md) | 不在本文复制 descriptor、doorbell、ring 或 IRQ handler 路径 |
| EtherCAT master | `ec_master/master0` 静态引用 `eth1` | 本文 | 具体 driver、probe 顺序和接口未知 |
| 协议配置 | 应决定周期、从站拓扑、过程数据和同步方式 | 未知项 | 当前没有 K3 配置格式或参数来源 |
| 应用与 slave | 应消费过程数据并对应实际从站 | 未知项 | 当前没有 slave 清单、发现结果或真板日志 |

静态 phandle、Ethernet link 与 EtherCAT 协议运行是不同层级。任何一层存在都不能代替其他层的可用性证据。

## 4. 软件与协议边界

当前材料没有确认下列内容：

- K3 使用的 EtherCAT master 实现、内核模块、用户态组件及版本；
- master 配置从 DTS、文件、命令行还是应用接口取得；
- `eth1` 与普通网络栈的独占、共享或切换策略；
- slave 扫描、地址分配、PDO/SDO、状态机和 Distributed Clocks 配置；
- 应用如何提交输出、读取输入、接收错误或取消操作。

因此，本文不把通用 EtherCAT 栈或其他平台的操作方式写成 K3 行为，也不提供未观察的 Buildroot 命令。

## 5. 运行、错误与恢复边界

当前没有周期、抖动、同步精度、吞吐或帧丢失测量，也没有 master probe、slave discovery、OP 状态或真板通信日志。以下路径同样未观察：

- `eth1` link-down、slave disappearance 或拓扑变化；
- 帧超时、取消、重试、watchdog 和状态降级；
- master、GMAC、PHY 或 slave reset 后的恢复顺序；
- suspend/resume、普通网络共存或接口重新绑定。

`main-device = <&eth1>` 不能证明上述路径已实现或成功，也不能据此声明实时周期和同步精度达到任何数值。

## 6. 未知项

### U1：EtherCAT master 软件组成与配置契约

- 当前证据：官网入口已登记但正文未直接取得；共享 DTS 只给出 `ec_master/master0/main-device`。
- 禁止推断：不得用节点名或通用 EtherCAT 栈补写 driver、工具、版本、命令、配置格式或 probe 顺序。
- 解除条件：直接取得适用 K3 SDK 修订的官方 EtherCAT 正文及对应源码、配置，并核对目标板启动日志。
- 影响主题：软件部署、master 生命周期、接口所有权、配置与诊断入口。

### U2：目标 DTB、网口所有权与 slave 拓扑

- 当前证据：绑定位于 CoM260 共享 DTS，`main-device` 指向 `eth1`；默认 Kit DTB、网口独占策略和 slave 清单未知。
- 禁止推断：不得由 shared DTS 选择默认 DTB，也不得声明 `eth1` 已被 EtherCAT 独占、可与普通网络共存或已发现 slave。
- 解除条件：确认目标板修订与最终 DTB，核对 master 配置、接口绑定状态和 slave 扫描结果。
- 影响主题：板型映射、网络接口分配、协议拓扑和 bring-up。

### U3：周期、同步与完成语义

- 当前证据：没有 K3 周期、抖动、Distributed Clocks、同步精度、帧完成或超时数据。
- 禁止推断：不得从静态绑定、GMAC 能力或 TSN 名称推导 EtherCAT 实时性能和同步保证。
- 解除条件：在明确软件版本、目标 DTB、slave 拓扑和负载下取得配置及可重复的周期、抖动和同步测量。
- 影响主题：实时调度、过程数据时限、时钟同步和性能验收。

### U4：错误、取消与恢复路径

- 当前证据：既有 GMAC 文档只承担 Ethernet 数据面边界；没有 EtherCAT 的 link-down、slave disappearance、超时、取消或 reset 恢复记录。
- 禁止推断：不得把 GMAC 错误处理等同于 EtherCAT master 或 slave 状态恢复。
- 解除条件：取得官方错误语义和恢复流程，并在目标系统观察故障注入后的状态变化、错误结果与恢复判据。
- 影响主题：错误传播、状态机、watchdog、重连和运维诊断。

### U5：真板可用性

- 当前证据：本项目没有连接开发板，也没有 EtherCAT master、slave、链路或应用日志。
- 禁止推断：不得把 DTS 节点存在、`status` 或 phandle 写成真板功能验证结果。
- 解除条件：在已确认板型、镜像、DTB、接口和从站配置下完成端到端通信，并记录明确的成功与失败判据。
- 影响主题：板级支持状态、部署说明、兼容性和验收结论。

## 7. 相邻主题

- DTS 候选与 `ec_master → eth1` 摘要：[镜像与 DTS](../boot/com260-image-and-dts.md)。
- GMAC、MDIO、PHY 与 RGMII 静态链：[GMAC/PHY](../network/com260-gmac-phy.md)。
- MAC/MTL/DMA、完成与 IRQ 边界：[GMAC DMA/IRQ](../network/k3-gmac-dma-irq.md)。
- 来源状态与唯一 URL：[来源覆盖](../reference/source-coverage.md)。

术语、已知缺口和总索引由后续 Iteration 统一收敛，本轮不提前修改。
