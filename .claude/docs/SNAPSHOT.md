# SNAPSHOT

> 当前项目状态: `current`
> 最后同步: 初始化 (Wed Sep 02 2026 22:38:25 GMT+0800)
> 同步者: `openspec-init`

## 项目身份

- 名称: K3 板子信息汇总
- 英文名: SpacemiT K3 Board Documentation Aggregation
- 用途: 把 SpacemiT 官方社区的 K3 板子文档整理为结构化 Markdown, 便于离线检索与跨会话复用。
- 范围: 仅 K3 (key_stone/k3); 不覆盖 SpacemiT 其它板子或 K3 之外的章节。

## 信息源 (单源, M01)

- URL: https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs
- 语言: 简体中文
- 权威性: Spacemit 官方社区文档
- 最近观察修订日期: 待首次聚合时由首个 change 记录

## 形态与目录

- 类型: 纯文档 / Markdown 聚合
- 仓库内不存放可执行代码 (M04)
- 目录:
  - `docs/` — K3 聚合产出 (主题驱动, 非镜像源页面树, D02)
  - `openspec/` — OpenSpec 配置, specs, changes
  - `.claude/` — Claude Code 入口 (skills, commands, docs)
  - `CLAUDE.md` — 项目公共规则
  - `AGENTS.md` — 不适用 (本项目仅支持 Claude Code)

## 技术栈与依赖

- 无运行时依赖
- 无构建工具链
- 无包管理文件
- 唯一工具: OpenSpec CLI 1.6.0 (用于变更管理和规范验证)

## 支持范围

- Claude Code: 已配置 (`.claude/skills/`, `.claude/commands/`)
- Codex: 未配置
- OpenCode: 未配置

## 工作区与分支

- 工作区: 初始化完成, 无未提交修改
- Git 分支: `main`
- 初始 commit: 待 Phase 8 收尾时生成

## 关键约束摘要 (指针)

- M01 单一权威源 — `openspec/specs/project-model/spec.md`
- M02 输出形态 — `openspec/specs/project-model/spec.md`
- M03 语言与术语 — `openspec/specs/project-model/spec.md`
- M04 不存放可执行代码 — `openspec/specs/project-model/spec.md`
- D01 纯 Markdown 聚合 — `openspec/specs/decisions/spec.md`
- D02 主题驱动目录 — `openspec/specs/decisions/spec.md`
- R01 权威源 URL — `openspec/specs/references/spec.md`

## 同步状态

- `current`: 本次初始化建立的结构与上述描述一致。
- 任何字段变更需要刷新本文件并把状态从 `current` 保留, 或在新版本失效时标 `stale`。
