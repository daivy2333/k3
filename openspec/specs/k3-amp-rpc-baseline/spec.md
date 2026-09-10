# k3-amp-rpc-baseline Specification

## Purpose

K3 AP/RP 生命周期、共享内存、mailbox、ring 与 RPC 的可追溯知识契约, 使读者能够区分官方事实、第三方实现、推论和仍待真板或缺失源码解除的边界; 是 MS08 主题文档与 source-coverage / known-gaps / terminology 同步的可追溯契约。

## Requirements

### Requirement: AP/RP 生命周期与资源所有权可追溯

知识基线 SHALL 描述 AP/RP 镜像、启动交接、握手、共享窗口初始化和恢复阶段，并为地址、资源所有者和阶段顺序标明来源与证据等级。

#### Scenario: 查询正常启动与初始化路径

- **WHEN** 读者查询 AP/RP 从镜像装载到共享通道可用的正常路径
- **THEN** 文档给出各阶段的参与者、可观察状态、共享窗口所有者和已确认的交接关系

#### Scenario: 启动材料不完整或相互冲突

- **WHEN** 固定 DTB、内嵌 DTB、源码注释或缺失的 U-Boot 实现不能支持唯一启动结论
- **THEN** 文档分别记录可确认行为、冲突、未知项和解除条件，不选择未经证实的唯一解释

### Requirement: 共享窗口与内存属性边界明确

知识基线 SHALL 分开描述 AP 物理地址、RCPU 本地 alias、窗口布局、PMA/PBMT、barrier、cache 和访问权限，并禁止把共享 SRAM 结论外推为所有 DDR 或设备 DMA 的一致性结论。

#### Scenario: 查询共享窗口数据路径

- **WHEN** 读者查询 AP 与 RP 如何定位并访问共享窗口
- **THEN** 文档给出两侧地址、已知大小、布局责任、可见性动作和仍需验证的 alias 或属性

#### Scenario: 缺少 ring 或原子实现

- **WHEN** `ov-channels` 或 `rt-async` 源码不可用，无法确认布局、原子序或唤醒细节
- **THEN** 文档保留缺失实现及其阻断结论，不用注释、估算或相邻模块替代

### Requirement: 数据通道与通知通道分层

知识基线 SHALL 区分共享内存中的请求、响应和 urgent 通道与 mailbox 硬件通道, 并说明 doorbell 仅触发接收端重检队列, 不能替代队列状态; BUSY 仅为服务端弹性轮询窗口提示, 不充当锁或并发门禁, 客户端仅在 BUSY=0 时发送 NOTIFY。

#### Scenario: 请求获得正常响应

- **WHEN** AP 发布请求且 RP 处理后发布响应
- **THEN** 文档描述发布、内存序、通知、ISR 清除、队列重检和响应关联的完整路径及证据边界

#### Scenario: 通知丢失、合并或资源繁忙

- **WHEN** doorbell 未发送、FIFO 满、BUSY 置位、IRQ 被合并或等待者注册发生竞态
- **THEN** 文档说明现有实现的推进条件、可能失效方式和需要补充的验证，不把一次通知等同于一次消息

### Requirement: RPC 行为和失败边界可查询

知识基线 SHALL 描述请求、响应、urgent、Deferred、多块消息、反序列化失败和未知方法的可观察行为, 并记录等待、超时、取消、并发等待者不均与多等待者限制。

#### Scenario: RPC 正常完成

- **WHEN** 请求可被服务端处理且响应可发布
- **THEN** 文档说明请求与响应的关联、完成条件、通知策略和已读测试入口

#### Scenario: RPC 产生错误或调用方停止等待

- **WHEN** 解码失败、方法未知、响应过长、对端无响应、调用超时或取消
- **THEN** 文档区分已实现行为、潜在不唤醒或资源滞留风险，以及尚未定义的取消和恢复语义

### Requirement: 复位与恢复不隐瞒数据损失

知识基线 SHALL 描述 AP 初始化、RP fallback、watchdog re-init 和对端复位之间的状态竞争，并明确重新初始化可能清除未读消息。

#### Scenario: 共享窗口保持有效

- **WHEN** 后启动的一侧发现窗口 magic 和布局仍有效
- **THEN** 文档说明现有实现是否保留队列、如何恢复通知以及哪些条件仍需真板确认

#### Scenario: 窗口无效或对端复位

- **WHEN** 任一侧发现 magic 无效、启动阶段清空 SRAM 或对端重启
- **THEN** 文档标明初始化责任、通信暂停、未读消息影响、错误传播和重新建立服务所需条件

### Requirement: 来源覆盖与导航保持一致

AMP/RPC 主题 SHALL 接入总索引、来源覆盖与已知缺口; 官方来源、固定 revision 第三方分析和未知项 SHALL 使用一致的主题落点与证据等级; 新增术语 (AMP / RPC / ring / doorbell / 共享窗口) SHALL 有唯一主写法且不与 mailbox hardware channel 或 DMA ring 混用。

#### Scenario: 新增主题完成聚合

- **WHEN** AMP/RPC 主题文档加入仓库
- **THEN** 总索引可进入该主题，相关来源有唯一覆盖记录，缺口包含当前证据和解除条件，交叉引用不复制既有主题正文，术语表新增五项

#### Scenario: 来源不能直接支持结论

- **WHEN** 结论只来自第三方源码、作者声明、缺失仓库线索或尚未运行的测试入口
- **THEN** 文档按实际证据等级表述并保留验证边界，不标记为官方事实或本项目运行结论
