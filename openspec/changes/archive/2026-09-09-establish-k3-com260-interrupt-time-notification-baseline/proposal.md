## Why

MS04 已建立 UART consumer 到 `&saplic` 的静态连接，但 K3 CoM260 的 AP AIA、APLIC、IMSIC、RCPU PLIC、timer 与 mailbox 通知仍分散在 SoC 概述、DTS、SDK 文档和第三方工程中。MS05 需要按 AP/RP 域和证据层整理这些机制，避免把 wired IRQ、MSI、timer 或 mailbox 的实现经验互相补值。

## Source Baseline and Change Type

- 权威入口：<https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs>
- 现有官网主题：Timer 页面已登记，观察日期 2026-09-02，状态 `partially-observed`；官网正文仍为 SPA 壳。
- 官方交叉验证：SpacemiT `docs-chip`、`docs-buildroot` 与 `linux-6.18` 的 K3 路径或分支。
- 第三方经验：R10-R12 所登记的 Rt-Async-AMP/tgoskits 固定 revision，只记录源码行为、风险和待验证假设。
- Change 类型：aggregation；建立 MS05 知识基线，不实现中断控制器、timer 或 mailbox 驱动。

## What Changes

- 创建 `docs/interrupts/` 下的 K3 CoM260 中断与时间主题文档，分开说明 AP AIA/APLIC/IMSIC、RCPU PLIC/SysTimer/MSIP/AON timer 的适用域。
- 创建或拆分通知机制正文，整理 mailbox channel/user、wired source/EID、FIFO/pending 清除、通知与数据分离、自测及工作预算。
- 在引用前登记本 change 实际直接打开的 Timer、DTS、binding、driver 或官方源码精确 URL。
- 复核 G4 与本 change 新未知项；只有对象和解除条件不重复时才新增缺口，不用第三方实现关闭 G4。
- 更新 `docs/index.md` 的 interrupts 入口与状态，并同步本 change 实际改变的来源或缺口计数。

## Capabilities

### New Capabilities

- `k3-com260-interrupt-time-notification-baseline`：定义 K3 CoM260 AP/RP 中断域、timer 与 mailbox 通知机制的来源分层、关系图、冲突保存和未知项闭包。

### Modified Capabilities

- None.

## Scope Decisions

- 默认按“中断与时间”“mailbox 与通知”拆成两篇正文；若调查证明来源和术语可以在 450 行内保持清晰，可合并，但不能删除任一验收主题。
- AP AIA/APLIC/IMSIC 与 RCPU PLIC/SysTimer/MSIP/AON timer 分域记录；相同 IRQ、hart 或 timer 术语不构成互用依据。
- wired IRQ、MSI/MSI-X、EID 与 mailbox 硬件 channel 分层记录，不把 channel 编号等同 IRQ source 或 EID。
- `stopei`、affinity、重复 IRQ、FIFO/pending 清除和 self-test 只作为固定 revision 第三方经验；标准语义、官方 DTS 行为与第三方代码不互相提升。
- timer 的频率、deadline、mtimecmp 写序、suspend/resume 和 timeout 只记录直接来源支持的范围；不由 `timebase-frequency` 或 RP 实现推定 AP 运行行为。

## Scenario Gaps and Defaults

- 官网正文仍不可读时，保持官网行 `partially-observed`，只用官方 GitHub/DTS/driver 作 `交叉验证`，不代填官网修订。
- CoM260 Kit 默认顶层 DTS 未唯一映射时，只使用共享 K3/CoM260 字段；变体专有 routing 不写成默认板值。
- 缺少 K3 programmer reference 时，保留 G4；RISC-V AIA 标准可以解释术语，但不能补写 K3 MMIO、source 数、EID 数或 delivery 行为。
- 第三方 AP DTS 中的 IMSIC `0xe0400000`、APLIC `0xe0804000`、512 sources/511 EID，以及 mailbox source217 仅作为该固定 revision 配置，不自动成为 CoM260 官方事实。
- RCPU PLIC/SysTimer/mailbox 的地址、IRQ 和清除顺序只属于已读 RP 配置；不得外推 AP，也不得由变量名 `MBX3` 推定物理 mailbox3。
- mailbox FIFO 满、ISR 工作预算、SMP affinity、重复通知或缺失实现无法闭合时，记录未知项和解除条件，不设计驱动修复。
- 本仓库没有运行时并发或超时执行；取消、超时、恢复仅作为来源中已有的软件行为与未知边界记录，不创建可执行验证程序。

## Scenario Sketch

- **AP wired IRQ**：给定官方 DTS 或源码可定位 APLIC/IMSIC 拓扑，整理 source → target hart/EID → claim/complete 关系；缺字段时保留未知，不由第三方数值补齐。
- **MSI/MSI-X**：给定 binding/driver 明示 domain 或 delivery，分开记录 wired 与 MSI 路径；只有第三方实现时标为经验并保留 affinity 边界。
- **RCPU 中断与时间**：给定 RP DTS/源码，记录 PLIC、SysTimer、MSIP 与 AON timer 的固定 revision 行为；禁止替代 AP AIA 或推定所有 hart。
- **timer deadline**：来源明确频率、counter 与 compare 时，记录读写和 deadline 边界；写序、瞬态或 suspend/resume 不明时保持未知。
- **mailbox 通知**：来源可闭合时，记录 AP/RP user、channel、FIFO、pending、IRQ 与队列重查关系；channel、source、EID 不逐号等同。
- **重复或残留通知**：记录先排 FIFO、再清 pending、重读和预算边界；缺少真板证据时不声明不会丢中断或重复触发。
- **self-test**：记录第三方 `TEST_MBOX` 的注入路径及其能证明和不能证明的范围；不把源码存在当作测试已通过。
- **来源变化或中止**：官方分支、节点或字段变化影响域模型时停止使用旧调查结论，返回 Plan；普通不可访问按既有 fallback 保存未知项。

## Non-goals

- 不设计或实现 OS 中断子系统、IRQ domain、timer、mailbox、waker 或 async notification API。
- 不修改 Linux、DTS、bootloader、StarryOS、Rt-Async-AMP 或 tgoskits 产品代码。
- 不用 QEMU、第三方板卡或通用 RISC-V 行为证明 CoM260 真板行为。
- 不展开 UART、DMA、GMAC、UFS 等设备数据面；只记录其 IRQ 或通知连接作为边界示例。
- 不建立抓取器、专用验证器、运行身份、manifest、Hash 或 Evidence 占位。

## Impact

- 预计新增两篇 `docs/interrupts/` 主题文档，修改来源覆盖、known-gaps（条件性）和总入口。
- OpenSpec change 保存需求、调查、设计、任务、Cycle 与验证结果；仓库继续保持纯 Markdown。
- MS06-MS09 可引用本 change 的 IRQ、timer 与通知术语和域边界，不必重新混合 AP/RP 来源。

## Gate 1

- Status: approved
- User approval: `批准`

| Check | Status | Evidence |
| --- | --- | --- |
| BDD gap scan | PASS | 覆盖 AP/RP 分域、wired IRQ、MSI、timer、mailbox、残留通知、自测、来源变化和不可访问 |
| Gap decisions | PASS | 用户接受 Scenario Gaps and Defaults |
| Scenario sketch | PASS | 本 proposal 给出正常、冲突、失败和停止路径 |
| OpenSpec change | PASS | `establish-k3-com260-interrupt-time-notification-baseline` 已创建 |
| Requirements and scope approval | PASS | 用户原话：`批准` |
