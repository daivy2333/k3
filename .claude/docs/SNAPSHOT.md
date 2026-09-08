# SNAPSHOT

> 当前项目状态: `current`
> 最后同步: 2026-09-08 (Tue Sep 08 2026 14:05:00 GMT+0800) 增量刷新
> 同步者: `openspec-docs-maintainer`
> 同步 revision: `1f572bb` + 未提交 change `establish-k3-com260-board-boot-baseline` 收尾同步 (MS03 全部产品交付 + 项目级状态)

## 项目身份

- 名称: K3 板子信息汇总
- 英文名: SpacemiT K3 Board Documentation Aggregation
- 用途: 把 SpacemiT 官方社区的 K3 板子文档整理为结构化 Markdown, 便于离线检索与跨会话复用。
- 范围: 仅 K3 (key_stone/k3); 不覆盖 SpacemiT 其它板子或 K3 之外的章节。

## 信息源 (单源, M01)

- URL: https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs
- 语言: 简体中文
- 权威性: Spacemit 官方社区文档
- 最近观察修订日期: 2026-09-07 (initial observation baseline 2026-09-02 由 change `establish-k3-doc-foundation` 记录; 2026-09-05 由 refresh change `establish-k3-source-tracking-baseline` 更新; 2026-09-08 由 change `establish-k3-com260-board-boot-baseline` 在 §6.6 重新观察确认; R01 仍为 SPA 壳, 实际可见身份为 Vue SPA title="SpacemiT", `partially-observed` 状态保持)

## 形态与目录

- 类型: 纯文档 / Markdown 聚合
- 仓库内不存放可执行代码 (M04)
- 目录:
  - `docs/` — K3 聚合产出 (主题驱动, 非镜像源页面树, D02)
  - `docs/index.md` — 总入口与主题职责表
  - `docs/reference/` — 来源覆盖、文档模板、术语、已知缺口与人工刷新指南
  - `docs/boot/` — K3 启动链、镜像与 DTS 主题文档
  - `docs/platform/` — K3 SoC 概述与 CoM260 板级资源主题文档
  - `openspec/` — OpenSpec 配置, specs, changes (active 与 archive)
  - `.claude/` — Claude Code 入口 (skills, commands, docs, analysis, runbooks, incidents)
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

- 工作区: change `establish-k3-doc-foundation` 与 `establish-k3-source-tracking-baseline` 已于 2026-09-05 收尾归档; change `establish-k3-com260-board-boot-baseline` 于 2026-09-08 收尾归档 (MS03); 工作区待常规 commit 的产品代码包括 `docs/index.md` / `docs/boot/com260-boot-chain.md` / `docs/boot/com260-image-and-dts.md` / `docs/platform/com260-board-resources.md` / `docs/platform/k3-soc-overview.md` / `docs/reference/source-coverage.md` / `docs/reference/known-gaps.md` / `docs/reference/source-refresh.md` 与 `.claude/analysis/rt-async-amp-k3-{boot-platform,drivers,shared-memory,starryos-reuse}.md`。
- Git 分支: `main`
- 归档目录: `openspec/changes/archive/2026-09-05-establish-k3-doc-foundation/`, `openspec/changes/archive/2026-09-05-establish-k3-source-tracking-baseline/`, `openspec/changes/archive/2026-09-05-establish-k3-com260-board-boot-baseline/`

## 已建立文档

- `docs/index.md`: 总入口、CoM260 范围、九个 reference / boot / platform 链接与十类主题职责 (T2, T8, MS03 T13)
- `docs/reference/source-coverage.md`: 51 个唯一 URL 唯一覆盖表 (MS01 baseline 38 + MS03 Iteration 000 新增 6 + Iteration 001 第一轮新增 3 + Iteration 001 修复轮新增 4 个 raw DTS 文件), R01/R04-R08 与两个 linux-6.18 raw DTS URL 已逐项登记 (T3, MS03 T7-T9)
- `docs/reference/document-template.md`: 首行来源、源端修订、观察日期与证据强度格式 (T4)
- `docs/reference/terminology.md`: K3、CoM260 Kit、SoC、AP、RCPU、AIA、APLIC、IMSIC、MMIO、IRQ、DMA、IOMMU、GMAC、PHY、MDIO、RGMII、polling、async、waker、coherency 二十项主写法与别名 (T5)
- `docs/reference/known-gaps.md`: 7 个缺口 (G1-G7) 含 G3 partial, G7 新增 (CoM260 Kit 默认目标 DTS 未唯一映射), G8 已删除 (镜像 release management 超出 T12 范围); 含镜像内部组成与 aliases 节点的局部未知项交叉引用 (T6, MS03 T12)
- `docs/reference/source-refresh.md`: 持久字段 / 新 baseline 字段 / SPA 壳 / 真正的停止条件 四节, 以及 MS02 建立的 refresh change 工作流 (T7, MS02)
- `docs/platform/k3-soc-overview.md`: K3 SoC 能力概述, 涵盖 CPU/内存/外设/连接/启动能力总览 (MS03 Iteration 000 交付)
- `docs/platform/com260-board-resources.md`: K3 CoM260 模组 + Kit 板级资源, 涵盖 SoC 集成接口、模组引脚、Kit 底板连接器、PHY、UART、debug 等 (MS03 Iteration 000 交付)
- `docs/boot/com260-boot-chain.md`: K3 SoC 启动能力与 K3 CoM260 Kit 已观察到的启动链路 (local boot: Boot ROM → FSBL/SPL → ESOS → OpenSBI → U-Boot → payload/OS; download boot: Boot ROM → U-Boot Fastboot), 介质、SDK 版本边界、未知闭包 (MS03 T10, 202 行)
- `docs/boot/com260-image-and-dts.md`: K3 CoM260 镜像类型、写入方式、DTS 候选集合 (6 个 .dts + 1 个 .dtsi base, 直接打开 4 个文件), CMA 0x140000000 + DRAM 0x102000000 解码, 缺口闭包 (MS03 T11, 210 行)

## 已建立分析

- `.claude/analysis/k3-official-docs-for-starryos-async-drivers.md`: K3 官方资料规模下界与首批聚合主题分析 (R03, MS01)
- `.claude/analysis/rt-async-amp-k3-boot-platform.md`: Rt-Async-AMP 的 K3 启动与板级适配分析 (R09, MS03-MS08)
- `.claude/analysis/rt-async-amp-k3-shared-memory.md`: Rt-Async-AMP 的 K3 共享内存与通知链分析 (R10, MS05/MS06/MS08)
- `.claude/analysis/rt-async-amp-k3-drivers.md`: Rt-Async-AMP 的 K3 驱动与 StarryOS 异步边界分析 (R11, MS04-MS07)
- `.claude/analysis/rt-async-amp-k3-starryos-reuse.md`: Rt-Async-AMP 面向 StarryOS 的 K3 复用清单 (R12, MS03-MS11)

## Milestone 状态指针

- MS01 (来源覆盖与主题结构基线) — `completed` (2026-09-05 收尾, 见 `.claude/docs/tasks.md`)
- MS02 (来源追踪与人工刷新基线) — `completed` (2026-09-05 收尾, change `establish-k3-source-tracking-baseline` 首次真实 refresh 完成 7 URL 逐 URL 结论与 `交叉验证` 等级 SDK baseline)
- MS03 (K3 CoM260 板级与启动事实基线) — `completed` (2026-09-08 收尾, change `establish-k3-com260-board-boot-baseline` 完成 platform/boot 主题文档 + 51 URL 覆盖 + 7 个缺口闭包 + 4 个 raw DTS 文件直接打开)
- MS04-MS11 — `planned`

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
- R03-R12 当前有用、持续观察、第三方分析 URL/文档集合 — `openspec/specs/references/spec.md`

## 同步状态

- `current`: 本次增量刷新同步了 change `establish-k3-com260-board-boot-baseline` 收尾后的项目状态; MS03 已完成, 4 篇产品文档 + 51 URL 覆盖 + 7 缺口闭包 + 4 raw DTS 文件直接打开; 4 份 Rt-Async-AMP 第三方分析已登记 R09-R12。
- 任何字段变更需要刷新本文件并把状态从 `current` 保留, 或在新版本失效时标 `stale`。
