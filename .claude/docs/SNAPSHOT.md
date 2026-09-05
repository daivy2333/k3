# SNAPSHOT

> 当前项目状态: `current`
> 最后同步: 2026-09-05 (Sat Sep 05 2026 13:38:00 GMT+0800) 增量刷新
> 同步者: `openspec-docs-maintainer`
> 同步 revision: `1f572bb` + 未提交 `docs/index.md` / `docs/reference/source-refresh.md` / `openspec/specs/decisions/spec.md` / `openspec/specs/references/spec.md` / `.claude/docs/tasks.md`

## 项目身份

- 名称: K3 板子信息汇总
- 英文名: SpacemiT K3 Board Documentation Aggregation
- 用途: 把 SpacemiT 官方社区的 K3 板子文档整理为结构化 Markdown, 便于离线检索与跨会话复用。
- 范围: 仅 K3 (key_stone/k3); 不覆盖 SpacemiT 其它板子或 K3 之外的章节。

## 信息源 (单源, M01)

- URL: https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs
- 语言: 简体中文
- 权威性: Spacemit 官方社区文档
- 最近观察修订日期: 2026-09-02 (initial observation baseline; 源端未公开修订日期, 由 change `establish-k3-doc-foundation` 记录; 后续由 refresh change 更新)

## 形态与目录

- 类型: 纯文档 / Markdown 聚合
- 仓库内不存放可执行代码 (M04)
- 目录:
  - `docs/` — K3 聚合产出 (主题驱动, 非镜像源页面树, D02)
  - `docs/index.md` — 总入口与主题职责表
  - `docs/reference/` — 来源覆盖、文档模板、术语、已知缺口与人工刷新指南
  - `openspec/` — OpenSpec 配置, specs, changes (active 与 archive)
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
- Codex: 已配置 (`.agents/skills/` 软链, `AGENTS.md` 入口)
- OpenCode: 已配置 (`.agents/skills/` 软链, `AGENTS.md` 入口)
- skill 副本: 单一权威在 `.claude/skills/`, `.agents/skills/` 为软链, 不维护多份
- skill frontmatter: 统一精简为 `name` + `description` 两字段 (三端共同)

## 工作区与分支

- 工作区: change `establish-k3-doc-foundation` 已于 2026-09-05 收尾归档; 产品代码当前改动为 `docs/index.md` (T8 入口) 与 `docs/reference/source-refresh.md` (T7 指南, 含 G1-G4 修复), 均待常规 commit。
- Git 分支: `main`
- 归档目录: `openspec/changes/archive/2026-09-05-establish-k3-doc-foundation/`

## 已建立文档

- `docs/index.md`: 总入口、CoM260 范围、五个 reference 链接与九类主题职责 (T2, T8)
- `docs/reference/source-coverage.md`: R01、R04-R08 当前登记的 38 个唯一 URL 唯一覆盖表 (T3)
- `docs/reference/document-template.md`: 首行来源、源端修订、观察日期与证据强度格式 (T4)
- `docs/reference/terminology.md`: K3、CoM260 Kit、SoC、AP、RCPU、AIA、APLIC、IMSIC、MMIO、IRQ、DMA、IOMMU、GMAC、PHY、MDIO、RGMII、polling、async、waker、coherency 二十项主写法与别名 (T5)
- `docs/reference/known-gaps.md`: 官网总页数、未展开目录、GMAC/PHY、AIA routing、DMA coherency/IOMMU、programmer reference 六类缺口 (T6)
- `docs/reference/source-refresh.md`: 184 行, 五种一次检查结果、三个独立维度、change/gap 边界、三个文字演练与停止条件 (T7)

## Milestone 状态指针

- MS01 (来源覆盖与主题结构基线) — `completed` (2026-09-05 收尾, 见 `.claude/docs/tasks.md`)
- MS02 (来源追踪与人工刷新基线) — 机制基础已建立, 首次真实 source refresh 待未来来源变更 change
- MS03-MS07 — `planned`

## 关键约束摘要 (指针)

- M01 单一权威源 — `openspec/specs/project-model/spec.md`
- M02 输出形态 — `openspec/specs/project-model/spec.md`
- M03 语言与术语 — `openspec/specs/project-model/spec.md`
- M04 不存放可执行代码 — `openspec/specs/project-model/spec.md`
- D01 纯 Markdown 聚合 — `openspec/specs/decisions/spec.md`
- D02 主题驱动目录 — `openspec/specs/decisions/spec.md`
- D03 六个实际文档承载基础职责 — `openspec/specs/decisions/spec.md`
- D04 URL 是覆盖记录的唯一键 — `openspec/specs/decisions/spec.md`
- D05 主题文档首行同时表达修订与观察时间 — `openspec/specs/decisions/spec.md`
- D06 证据强度采用四级标记 — `openspec/specs/decisions/spec.md`
- D07 刷新状态与聚合状态分离 — `openspec/specs/decisions/spec.md`
- D08 配置修复只增加必要引用 — `openspec/specs/decisions/spec.md`
- R01 权威源 URL — `openspec/specs/references/spec.md`
- R04-R08 当前有用与持续观察 URL 集合 — `openspec/specs/references/spec.md`

## 同步状态

- `current`: 本次增量刷新同步了 change `establish-k3-doc-foundation` 收尾后的项目状态; 工作区未提交改动属于该 change 的产品交付, 提交后本描述仍 `current`。
- 任何字段变更需要刷新本文件并把状态从 `current` 保留, 或在新版本失效时标 `stale`。
