# Tasks

> 维护者: `openspec-milestone-planner` 负责 `MSxx` 路线; `openspec-docs-maintainer` 负责状态同步。
> 当前状态: 初始化完成, 无进行中任务, 无已承诺待办。

## Milestone Roadmap (MSxx)

### MS01 — 首次聚合 K3 公开文档骨架

- 状态: planned
- 成果: 把源端 (R01) 当前可访问的 K3 顶层章节镜像为 `docs/` 下的主题化 Markdown; 每篇文档含 `> 来源:` 行。
- 工作量: 中 (需要逐章人工核对)
- 稳定基线: `docs/` 下存在可被读者按主题检索的 K3 概述/硬件/软件/外设分类文件; 源端 URL 与观察修订日期记录在每篇文档与对应 change。
- 验证边界: `openspec validate --specs --changes` 通过; 至少 1 个 change 标记 `accepted`; `docs/` 内每篇文档都带 `> 来源:` 行且行内 URL 可解析。
- 诊断边界: 源页面访问失败或结构变更时, 仅限 MS01 范围内重排; 不引入其它板子。
- 依赖: 无
- 关联 R: R01

### MS02 — 主题细化与术语稳定

- 状态: planned
- 成果: 在 MS01 基础上按 SoC 子系统 (CPU/内存/启动/外设/调试) 完成主题细化, 解决跨页重复描述, 锁定 M03 术语表。
- 工作量: 中
- 稳定基线: `docs/<topic>/` 下的二级主题文件存在; M03 术语被一致使用。
- 验证边界: 同一术语在 `docs/` 全文检索中只出现一种拼写; references 至少包含一份术语表 Rxx。
- 诊断边界: 当源页面出现新术语或冲突术语时, 仅在 MS02 范围内更新; 不重新拆 MS01。
- 依赖: MS01
- 关联 R: R01

### MS03 — 源端刷新机制

- 状态: planned
- 成果: 建立源端修订日期检查节奏与 change 模板, 每次源端更新产生对应 change, 不留静默差异。
- 工作量: 小
- 稳定基线: docs-maintainer 维护的源端 revision 时间线可追溯; 至少一次刷新 change `accepted`。
- 验证边界: 至少 1 个标 `refresh` 的 change 完成 `accepted`; `R01` 记录的"最近观察修订日期"被相应 change 刷新。
- 诊断边界: 源端下线或权限变更时, 停止刷新并开 incident。
- 依赖: MS01, MS02
- 关联 R: R01

## 进行中

(无)

## 已承诺待办

(无)

## 阻塞

(无)

## 最近完成

- 项目初始化: 完成 OpenSpec 结构, specs, SNAPSHOT, tasks, change-cycle 模板, CLAUDE.md。

## 与 OpenSpec Changes 的同步

- 每个 milestone 对应一个或多个 OpenSpec change (数量不绑定)。
- change `accepted` 时, 由 `openspec-docs-maintainer` 把对应 milestone 状态从 `planned` 推进到 `in-progress` 或 `done`, 并在"最近完成"追加引用。
- 未批准的想法不进入 tasks。
