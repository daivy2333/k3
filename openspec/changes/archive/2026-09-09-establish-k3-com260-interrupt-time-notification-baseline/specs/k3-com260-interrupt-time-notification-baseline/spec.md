## ADDED Requirements

### Requirement: AP 与 RCPU 中断域必须分层聚合

文档 SHALL 分开记录 AP AIA/APLIC/IMSIC 与 RCPU PLIC 的控制器角色、地址或节点、hart/context、IRQ domain、routing、mask、claim/complete 或 ack/EOI，并标明来源、版本、适用域和证据等级。

#### Scenario: AP wired IRQ 路由可追溯

- **WHEN** 官方 DTS、binding 或 driver 提供 APLIC/IMSIC 节点和路由字段
- **THEN** 文档记录 wired source 到目标 hart/EID/leaf IRQ 的可证路径，并保留缺失字段

#### Scenario: AP 与 RCPU 控制器不互相补值

- **WHEN** AP AIA 与 RCPU PLIC 使用相同的 IRQ、hart 或 claim 术语
- **THEN** 文档按资源域分别记录，不用一侧地址、stride、寄存器或行为填补另一侧

#### Scenario: claim 与 complete 语义来源不足

- **WHEN** 只有 RISC-V 标准或第三方 `stopei` 实现而缺少 K3 官方说明
- **THEN** 文档只解释对应来源层的语义，并把 K3 硬件行为保留为未知项

### Requirement: wired IRQ 与 MSI 路径必须区分

文档 SHALL 分开描述 wired IRQ、MSI/MSI-X、EID 分配、target hart 和 affinity，禁止把 APLIC source、mailbox channel 或 EID 视为同一编号空间。

#### Scenario: MSI delivery 有官方依据

- **WHEN** 官方 binding、DTS 或 driver 明示 MSI domain 或 delivery 关系
- **THEN** 文档记录对应节点、父子 domain 与限制

#### Scenario: 只有第三方 MSI 实现

- **WHEN** MSI/EID/affinity 仅能从固定 revision 第三方代码观察
- **THEN** 文档标为第三方经验，并保留 SMP affinity、迁移和资源释放边界

### Requirement: timer 与软件通知必须按域记录

文档 SHALL 整理 AP/RCPU 可证的 timer、counter、compare、deadline、timeout、MSIP 与 suspend/resume 边界，不得由一个域的频率、地址或写序推定另一域。

#### Scenario: timer 字段完整

- **WHEN** 来源给出 counter、compare、频率、hart 或中断连接
- **THEN** 文档记录字段及适用执行级别，并说明它能支持的 deadline 结论

#### Scenario: timer 写序或恢复未知

- **WHEN** mtimecmp 瞬态、hart 编号、suspend/resume 或 reset 后状态缺少直接依据
- **THEN** 文档记录当前证据、禁止推断、解除条件和影响主题

### Requirement: mailbox 通知链必须与数据状态分离

文档 SHALL 记录 AP/RP mailbox 的物理实例、user/channel、FIFO/pending、IRQ source/EID、清除顺序、自测和工作预算，并把通知解释为促使接收方重查数据状态的提示。

#### Scenario: AP 到 RP 通知

- **WHEN** AP 写 mailbox channel 并触发 RP IRQ
- **THEN** 文档记录发送 user、接收 user、channel、IRQ 与 FIFO/pending 处理关系，不把共享内存通道号等同硬件 channel

#### Scenario: RP 到 AP 通知

- **WHEN** RP 发门铃并经 APLIC/IMSIC 到达 AP
- **THEN** 文档记录可证 source/EID/handler 路径和固定 revision 边界

#### Scenario: 残留或重复通知

- **WHEN** FIFO、pending 或电平状态在使能 IRQ 前已有残留，或 handler 达到工作预算
- **THEN** 文档记录排空、清除、重读、截断和错误传播的已知行为与未知项

#### Scenario: self-test 未实际运行

- **WHEN** 第三方源码包含 mailbox 自测入口但没有本项目运行证据
- **THEN** 文档说明该入口的路径与覆盖范围，不声明自测通过或防丢通知成立

### Requirement: 来源、缺口和入口必须一致

MS05 文档 SHALL 遵守首行来源、URL 唯一登记、四级证据、未知项四字段和主题导航规则。

#### Scenario: 官网正文不可访问

- **WHEN** 官方社区页只能观察到 SPA 壳
- **THEN** 保持 `partially-observed`，使用官方 GitHub/DTS/driver 交叉验证且不代填官网修订

#### Scenario: G4 仍未完全解除

- **WHEN** 官方源码只能补充部分地址或 routing，仍缺 programmer reference 或真板 delivery 证据
- **THEN** 更新 G4 为实际 `partial` 或保持 `open`，并保留未解字段；不得以第三方代码关闭

#### Scenario: interrupts 主题交付

- **WHEN** 中断、时间和通知正文达到验收条件
- **THEN** `docs/index.md` 提供可解析入口并反映实际聚合状态、来源计数和缺口计数
