## Why

MS03、MS05 和 MS06 已分别记录 K3 的启动、mailbox 与共享内存属性，但 AP/RP 生命周期、共享窗口、ring、RPC、通知和恢复仍分散在多篇文档与第三方分析中。MS08 需要建立独立、可追溯的跨核通信知识基线，使这些机制能按资源所有权、内存序和证据等级统一检索。

本 change 由 SpacemiT K3 官方社区文档及其 K3 子章节驱动，最近观察到的源端修订日期为 2026-09-08。它属于基于已有资料的内容聚合与重组，不是源端更新后的 refresh。

## What Changes

- 新增 K3 AMP 与跨核通信主题文档，整理 AP/RP 镜像和握手、共享窗口与地址 alias、初始化所有权、PMA/PBMT 边界，以及 mailbox doorbell、ring 和 RPC 的分层关系。
- 记录请求、响应、urgent 通道、BUSY、错误响应、多块发布、通知重检和等待者模型，并区分源码确认、作者声明、推论与未知项。
- 整理超时、取消、对端复位、watchdog re-init、未读消息丢失和恢复期间通信边界，不把 magic 恢复表述为无损恢复。
- 更新总索引、来源覆盖和已知缺口，使新增主题及未确认材料具有唯一落点。
- 补充 AMP、RPC、ring、doorbell 和共享窗口所需术语，避免与 mailbox hardware channel、DMA ring 或一般进程间通信混用。
- 对缺失的 `rt-async`、`ov-channels`、U-Boot K3 分支、手册、原理图和真板日志保留明确缺口，不以未读取的实现补全 ring、原子或启动语义。

### Non-goals

- 不实现或修改 AMP、RPC、ring、mailbox、共享内存或操作系统组件。
- 不运行刷写或真板实验，不把第三方源码、注释或历史测量升级为 K3 官方事实。
- 不扩展到 K3 之外的芯片、板卡或 SpacemiT 文档章节。
- 不重新定义 MS03、MS05、MS06 已建立的启动、中断、DMA 或内存属性事实；需要时使用交叉引用。
- 不把服务发现、版本字段、commit、run-id、nonce 或时间顺序设计成运行身份或验证机制。

## Capabilities

### New Capabilities

- `k3-amp-rpc-baseline`: 规定 K3 AP/RP 生命周期、共享内存、通知、ring、RPC、错误与恢复知识基线的内容、证据边界和导航要求。

### Modified Capabilities

- 无。

## Impact

- 预计新增 `docs/amp/` 下的主题文档，并修改 `docs/index.md`、`docs/reference/source-coverage.md`、`docs/reference/known-gaps.md` 和 `docs/reference/terminology.md`。
- 新增 change delta spec；不修改可执行代码、API、构建系统或运行时依赖。
- 复用 R09–R12 的固定 revision 分析作为第三方调查输入；M01 指向的官方来源仍是硬件事实权威。
