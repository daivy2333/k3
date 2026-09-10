## ADDED Requirements

### Requirement: DMA 类型与能力必须按对象分层

文档 SHALL 分开记录通用 DMA controller、设备内建 DMA 和共享内存通道的实例、channel、descriptor、data buffer、burst、地址宽度、scatter-gather、cyclic 与完成机制，并标明来源、版本、适用设备和证据等级。

#### Scenario: 通用 DMA controller 字段可追溯

- **WHEN** 官方文档、DTS、binding 或 driver 提供 DMA controller、channel 或能力字段
- **THEN** 文档记录对应实例、字段和限制，并保留未公开的 descriptor 或完成语义

#### Scenario: 设备内建 DMA 不外推为通用规则

- **WHEN** GMAC、UFS 或其他设备源码提供专有 ring、UTP、descriptor 或完成路径
- **THEN** 文档把行为限定到该设备和固定来源，不补写通用 DMA controller 的能力

#### Scenario: 共享内存与设备 DMA 不互相补值

- **WHEN** AP/RP 共享内存和设备 DMA 都涉及 buffer、barrier 或 cache 属性
- **THEN** 文档按传输对象分别记录，不用一侧地址别名、PMA 或同步方式填补另一侧

### Requirement: CPU 与设备所有权转换必须可验证

文档 SHALL 区分 descriptor 与 data buffer、CPU ownership 与 device ownership、提交与完成可见性，并仅在直接来源支持时记录 cache maintenance、barrier、状态位、IRQ/poll 和回收顺序。

#### Scenario: CPU 将对象交给设备

- **WHEN** 来源明确 descriptor 或 buffer 的填充、clean/flush、barrier 和提交步骤
- **THEN** 文档记录步骤顺序、状态变化、适用对象和设备可见的完成边界

#### Scenario: 设备将对象归还 CPU

- **WHEN** 来源明确 IRQ、poll、状态位或 descriptor ownership 表示传输完成
- **THEN** 文档记录 CPU 重新读取前的 invalidate、barrier 或状态检查及其证据边界

#### Scenario: 通知不等同于数据可见

- **WHEN** IRQ、mailbox 或其他通知到达但来源未说明数据同步关系
- **THEN** 文档不得声明数据已对 CPU 可见，并登记缺失的完成或内存序条件

#### Scenario: 错误超时或 reset 中断所有权转换

- **WHEN** 传输出现 DMA error、timeout、取消、设备 reset 或对端重启
- **THEN** 文档记录已知的停止、回收和恢复条件；缺少安全回收依据时保留未知项

### Requirement: cache 与 barrier 语义必须区分

文档 SHALL 分开记录 cache line、clean、invalidate、flush、CBO、内存 barrier 与 I/O fence 的对象、方向、执行时机和适用域，不把任一操作描述为其他操作的替代。

#### Scenario: cache maintenance 有直接实现依据

- **WHEN** 官方或固定 revision 源码对 descriptor 或 data buffer 执行 cache maintenance
- **THEN** 文档记录操作对象、范围、方向、调用位置与来源等级，不推定未覆盖设备采用相同策略

#### Scenario: fence 只有排序作用

- **WHEN** 来源只执行 memory barrier 或 `fence iorw,iorw` 而没有 cache clean/invalidate
- **THEN** 文档只记录可证的排序语义，不称其完成 cache maintenance

#### Scenario: cache line 或 coherency 未公开

- **WHEN** K3 资料未给出 cache line 大小、coherent interconnect 或设备 snoop 能力
- **THEN** 文档保留 G5 或独立未知项，不用 Linux API 行为或通用架构默认值补齐

### Requirement: PMA、PBMT、IOMMU 与地址转换必须分层

文档 SHALL 区分 PMA、PBMT、页表属性、IOMMU、CPU 虚拟/物理地址、设备地址、IOVA 与 AP/RP 地址别名，并记录设置阶段、作用范围、适用 hart、验证边界和缺失条件。

#### Scenario: PMA 或 PBMT 设置可定位

- **WHEN** OpenSBI、DTS 或固定 revision 源码可定位属性设置、窗口边界和调用阶段
- **THEN** 文档记录实际代码行为及适用范围，不因设置代码存在而声明真板属性已经生效

#### Scenario: 地址别名缺少同一存储证明

- **WHEN** AP 与 RP 使用不同地址访问声称相同的 SRAM 或 buffer
- **THEN** 文档记录两侧配置与推论，并把同一物理存储和双向可见性保留为待验证条件

#### Scenario: IOMMU 状态不明

- **WHEN** 无法定位目标设备的 IOMMU 节点、驱动、domain 或 map/unmap 路径
- **THEN** 文档不得推定 IOMMU 不存在、bypass 或 identity mapping，并登记解除条件

### Requirement: 来源、缺口和入口必须一致

MS06 文档 SHALL 遵守首行来源、URL 唯一登记、四级证据、未知项四字段和主题导航规则，并保持 G5 与实际调查结论一致。

#### Scenario: 官网正文不可访问

- **WHEN** 官方社区页只能观察到 SPA 壳
- **THEN** 保持 `partially-observed`，使用官方 GitHub、DTS、binding 或 driver 交叉验证且不代填官网修订

#### Scenario: G5 仍未完全解除

- **WHEN** 官方源码只能补充部分 DMA、cache、PMA/PBMT 或 IOMMU 字段，仍缺 programmer reference 或真板证据
- **THEN** 按实际证据把 G5 保持 `open` 或调整为 `partial`，并保留所有未解字段

#### Scenario: DMA 主题交付

- **WHEN** DMA 与内存所有权正文达到验收条件
- **THEN** `docs/index.md` 提供可解析入口，并反映实际聚合状态、来源计数和缺口计数
