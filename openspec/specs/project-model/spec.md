# Project Model

## Purpose

记录 K3 板子信息汇总项目的当前有效跨模块约束。条目使用 `Mxx`，记录
分类、范围、不变量、证据和状态。本文档不保存历史选择过程。

## Requirements

### Requirement: 项目模型可验证

长期有效的跨模块约束 SHALL 记录范围、不变量、证据和状态。

#### Scenario: 确认稳定约束

- **WHEN** 已验证某项约束会影响多个模块或后续变更
- **THEN** 使用递增 M 编号记录分类、范围、不变量、证据和状态

---

## M01 — 单一权威信息源

- 分类: domain
- 范围: 本仓库所有 K3 相关内容的来源边界
- 不变量:
  - 所有 K3 信息必须可追溯到 M01 指向的源页面。
  - 不得引入 SpacemiT 其它板子、其它章节或其它厂商的素材。
  - 源页面 URL 改变时, 不得静默切换; 必须创建 change 并刷新所有相关文档的 `> 来源:` 行。
- 证据: `R01` (references/spec.md)
- 状态: accepted

## M02 — 输出形态与目录边界

- 分类: architecture
- 范围: `docs/` 目录与 Markdown 文档的物理与结构约束
- 不变量:
  - 所有聚合产出只允许存在于 `docs/` 顶层或子目录。
  - 单个文档长度建议 ≤ 500 行; 超过时按主题拆分为同目录子文件。
  - 文档命名使用 kebab-case, 反映主题而非源页面 slug。
  - 每个文档首行使用 `> 来源:` 引用 (a) 源页面 URL, (b) 最近一次观察到的源端修订日期。
- 证据: 仓库布局 (`docs/` 目录), config.yaml `Output conventions`
- 状态: accepted

## M03 — 主要语言与术语稳定

- 分类: domain
- 范围: 文档语言, 术语, 中英混排策略
- 不变量:
  - 默认语言为简体中文 (zh-CN)。
  - 专有技术名词 (如 RISC-V, SoC, BSP) 保留英文原拼写。
  - 标题层不出现中英混排的同义并列; 选用一个并在文内保持一致。
- 证据: config.yaml `Source of truth (single)` 与 `Output conventions`
- 状态: accepted

## M04 — 不存放可执行代码

- 分类: runtime
- 范围: 仓库内可执行内容的边界
- 不变量:
  - 本仓库不包含应用程序代码, 脚本或构建工具链。
  - 任何辅助工具 (抓取, 校验, 渲染) 必须放在独立仓库并通过 `references/spec.md` 登记为 `dependency`。
- 证据: config.yaml `Workflow expectations`
- 状态: accepted
