## Context

MS03 已记录 K3/CoM260 启动阶段，MS05 已记录 mailbox 双向通知，MS06 已记录共享内存与 PMA/PBMT/cache 边界。现有正文没有串联 AP/RP 生命周期、共享窗口初始化、ring/RPC 状态和对端复位恢复；`docs/amp/` 尚不存在。

R09–R12 捕获了 Rt-Async-AMP `ccb1ff0b`、tgoskits `19219411d`、OpenSBI `7a2df083` 的固定 revision 调查。当前本地 checkout 仍匹配这些提交且各自工作树无修改；`rt-async/modules/platform` 与 `modules/ov-channels` 仍缺失，根 workspace 的 offline metadata 因前者不存在而退出 101。可读代码足以确认 `wait_ready`、watchdog re-init、BUSY/doorbell、RPC error/Deferred 和 AP `/dev/rt_shm` 的调用位置，但不能确认 ring layout、跨核原子、IrqLatch 或真板效果。

当前 `source-coverage.md` 有 70 行且 70 个唯一 URL，`known-gaps.md` 有 G1–G10，`docs/index.md` 没有 AMP 主题入口。工作区带有 MS06/MS07 的既有 staged 修改和未跟踪 `others/`；本 change 必须只追加自身内容，不覆盖或提交这些既有状态。

## Goals / Non-Goals

**Goals**

- 让 AP/RP 镜像交接、共享窗口地址、初始化所有权和恢复阶段形成可独立查询的生命周期模型。
- 让请求、响应、urgent ring、BUSY、doorbell、ISR、等待和 RPC 完成形成可追踪的数据/通知路径。
- 用状态转换表达 timeout、取消、错误响应、对端 reset 和 re-init 的数据损失边界。
- 让来源、术语、缺口和总入口与两篇正文一致。

**Non-Goals**

- 不补写缺失子仓，不修改或构建第三方工程。
- 不设计 StarryOS AMP/RPC API，不实现驱动或跨核协议。
- 不刷新 SNAPSHOT、全局 tasks 或 M/D/K/R/I。
- 不把第三方 revision、nonce、时间戳或服务发现字段用于运行身份验证。

## Decisions

### D1：拆分生命周期与消息路径两篇文档

`docs/amp/k3-amp-shared-memory-lifecycle.md` 承载镜像/握手、地址 alias、窗口布局、PMA/PBMT、初始化所有权与 reset/re-init；`docs/amp/k3-rpc-ring-notification.md` 承载请求/响应/urgent、ring、BUSY、doorbell、等待、RPC 错误与完成。

两篇内容依赖不同的证据和诊断边界。生命周期基线不依赖缺失的 ring 实现即可成立；消息路径必须把缺失 `ov-channels` 和 `rt-async` 明示为边界。合并成单篇会让 ring/原子缺口阻塞地址与初始化基线，也容易超过 M02 的建议行数，因此不采用。

### D2：官方事实、官方交叉验证和固定 revision 第三方行为分层

官网 R01 及 K3 相关页面保持唯一权威入口；SpacemiT GitHub 文档和 Linux DTS 只作交叉验证；R09–R12 及其指向的 checkout 只说明固定 revision 第三方行为。正文从第三方多处控制流归纳出的路径标为推论，未有真板或缺失源码支持的部分标为未知项。

替代方案是把能在本地编译或能找到测试的第三方内容升级为 K3 事实。构建与 host test 不能证明 K3 硬件行为，且当前 workspace 不能完整解析，因此不采用。

### D3：生命周期按“最后破坏者—初始化者—发布—在线—失效—重建”组织

第一篇文档按阶段记录 SPL/U-Boot/bootm 等潜在窗口破坏者、AP probe 与 RP fallback/watchdog、`SHM_BASE` 发布和 magic 失效。每个阶段同时记录地址空间、状态所有者、允许操作和失败后果。

替代方案是按 AP、RP 两侧分别罗列。该结构会隐藏两侧初始化竞争和 U-Boot 清窗与晚期写回的时序关系，因此不采用。

### D4：共享数据是真值，doorbell 只触发重检

第二篇文档把共享 ring 与 mailbox hardware channel 分成两个编号空间。请求/响应/urgent、BUSY 和响应关联属于共享数据；mailbox FIFO、pending、IRQ 与 waker 只负责促使接收方重新检查共享状态。一次门铃不映射为一条消息，通知丢失、合并和重复均按状态重检边界描述。

替代方案是按通知次数推导消息完成。现有代码明确以 ring pending 作为就绪条件，该方案会误述完成语义，因此不采用。

### D5：恢复作为可能丢数据的状态转换

magic 无效、对端 reset 或启动链清窗触发 re-init 时，文档必须写明通信暂停、未读消息可能清除、旧 request ID/Deferred 操作的状态未知，以及重新上线的条件。固定等待 3 秒、最多 8 次自愈等数值只记录为第三方策略，不提升为 K3 安全时序。

替代方案是把 watchdog 归类为透明自愈。源码注释确认 re-init 会清 ring，无法承诺无损恢复，因此不采用。

### D6：已有缺口继续归属 G4/G5/G7，新增一个 AMP/RPC 独立缺口

APLIC/IMSIC delivery 继续由 G4 持有，cache/PMA/IOMMU 由 G5 持有，默认 CoM260 DTS 由 G7 持有。新增 G11 承载共享窗口官方布局、ring/原子、初始化所有权、RPC 失败/取消和 reset 恢复的独立缺口；不得复制 G4/G5/G7 的解除条件。

替代方案是为每个缺失仓库或错误分支分别建立 G 条目。它会把同一协议闭包拆成难以同步的小项，因此不采用。

### D7：来源覆盖只登记正文实际引用的官方 URL

两篇正文使用的现有官方 URL 先复用原行并扩充 `amp` 主题职责；只有实际打开、正文实际引用且尚未登记的官方 URL 才新增。R09–R12 已在 references 登记，第三方仓库路径不重复进入官方来源覆盖表。

替代方案是把所有第三方源码文件分别登记为 URL。固定 revision 调查已有 R09–R12 作为检索入口，重复登记会混淆来源权威性和覆盖计数，因此不采用。

### D8：验证只检查文档行为和可追溯性

测试见证覆盖目标文件不存在、URL 唯一性、缺口数量、索引入口和第三方 workspace 的已知解析失败。GREEN 验证检查首行来源、证据标签、状态模型、相对链接、计数、scoped diff、Markdown whitespace 和 strict OpenSpec validation；不要求构建第三方工程或创建持久 Evidence。

替代方案是把第三方构建或真板运行作为本 change 验收。它超出纯文档 change 的授权和可用环境，也会把缺失依赖变成实现阻塞，因此不采用。

## Risks / Trade-offs

- [第三方分析捕获于 2026-09-08] → Act 在引用前核对本地 HEAD、工作树和目标符号；若 revision 或控制流变化影响 D1–D8，停止并返回 Plan。
- [`rt-async` 与 `ov-channels` 缺失] → 只记录调用点、缺失能力和解除条件，不补全 ring layout、原子序或唤醒保证。
- [启动注释包含历史真板结论] → 区分源码控制流、作者声明和本项目未运行状态，不用时间值关闭未知项。
- [与 MS03/MS05/MS06 重复] → 第一篇只串联生命周期，第二篇只串联消息与 RPC；既有启动、中断和内存属性正文使用相对链接。
- [工作区已有 staged 内容] → 每项验证限定本 change 文件和精确 diff，不重排、不提交、不清理既有修改。
- [正文规模增长] → 每篇目标不超过 450 行；预计超过 500 行或需要第三篇时返回 Plan。

## Migration Plan

Iteration 000 先核对来源并创建生命周期文档，形成后续可引用的地址、初始化和恢复状态基线。其 Review Result 为 `accepted` 后，Iteration 001 再创建 ring/RPC 文档并更新 G4/G5/G7/G11、术语、来源覆盖和总入口。

回滚时删除本 change 新建的 `docs/amp/` 文档，并精准还原本 change 对 reference/index 的修改；不得覆盖 MS06/MS07 或 `others/` 的既有状态。

## Open Questions

没有阻塞 Gate 2 的实质问题。共享 SRAM alias 的真板一致性、PMA 每 hart 实效、ring layout/原子序、IrqLatch、多等待者、RPC poison 唤醒、取消与 reset 后恢复均作为 G4/G5/G7/G11 下的未知项，不交给 Act 决定契约语义。
