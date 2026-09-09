## Context

本 change 是纯 Markdown 聚合。当前产品文档只声明 K3 支持 AIA、UART 等 consumer 接 `&saplic`，G4 仍缺地址、domain 与 delivery。2026-09-08 直接复核的官方 `k3.dtsi` 已提供 AP CLINT `0xe081c000`、IMSIC `0xe0400000`、APLIC `0xe0804000`、16 hart interrupt files、511 IDs、63 guest IDs 与 512 wired sources。Timer 官网页只能观察 SPA 壳。R10-R12 和固定 revision 第三方源码提供 RCPU PLIC/SysTimer/AON timer、AP `stopei`、mailbox4 与 self-test 行为，但没有本项目真板证据。

## Goals / Non-goals

**Goals**

- 建立 AP AIA/CLINT 与 RCPU PLIC/SysTimer/AON timer 的分域事实包。
- 建立 mailbox 通知链，区分数据通道、硬件 channel、IRQ source 与 EID。
- 保存官方事实、官方源码行为、第三方经验、推论和未知项边界。
- 更新精确来源、G4/新增缺口判断和 interrupts 导航。

**Non-goals**

- 不设计或实现驱动、IRQ domain、timer、mailbox、waker 或 async API。
- 不用第三方数值证明 CoM260 官方或真板行为，不展开设备数据面。
- 不创建脚本、构建、仿真、上板测试或 Evidence 身份机制。

## Decisions

### D1：两篇主题文档

`docs/interrupts/k3-interrupt-and-time.md` 承载 AP/RP 控制器与 timer；`docs/interrupts/com260-mailbox-notification.md` 承载 mailbox 和通知。两者故障域、术语和来源不同，拆分后均预计低于 450 行。

### D2：来源层按域和职责分开

官网正文只标 `官方事实`；SpacemiT GitHub/DTS/driver 标 `交叉验证`；R10-R12 与 `others/` 固定 revision 标 `第三方经验`。AP 与 RCPU 不互相补地址、hart、source、claim 或 timer 写序。

### D3：AP 拓扑以官方 DTS 为主

AP 文档从 `k3.dtsi` 记录 CLINT、IMSIC、APLIC 节点和连接；Linux 通用 binding/driver只解释字段或软件行为，不提升为 K3 硬件规范。`stopei` 与 affinity 由第三方实现单列。

### D4：通知不是数据真值

mailbox 只提示接收方重查队列。文档分别记录 AP/RP user、硬件 channel、FIFO/pending、source/EID、handler 和 self-test；共享内存 CH0-CH2 不与 mailbox channel 对号。

### D5：缺口条件更新

官方 DTS 能部分解除 G4 的地址和静态 topology，但不能证明运行时 delivery、claim/complete、SMP affinity 或真板行为。只有新对象且四字段不重复时新增缺口。

## Risks / Trade-offs

- 官方分支可能变化：Act 重新直接打开实际引用 URL；变化影响域模型时停止返回 Plan。
- `k3.dtsi` 是 K3 公共 SoC 配置，不证明唯一 Kit 顶层 DTS；文档保留 G7。
- 第三方注释包含上板声称：只记录可读代码行为，历史测量不写成本项目结论。
- index 当前部分计数仍显示 MS03 值；本 change 更新 interrupts 入口时按实际覆盖表和 gaps 同步，避免继续传播旧计数。

## Verification Strategy

- RED：两篇目标正文和 interrupts 入口不存在；精确新 URL 未登记；G4 仍为 open。
- 内容：检查 AP/RP 分域、节点字段、wired/MSI、timer、mailbox 清除、自测、证据标签和未知项四字段。
- 一致性：正文 URL 各在覆盖表一次；G4/新 G 状态与正文一致；index 链接可解析且计数等于权威文件。
- 边界：只修改计划列出的 Markdown；`others/` 只读。
- 质量：行数、相对链接、`git diff --check` 与 OpenSpec strict validation。
