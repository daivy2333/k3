# source-refresh Specification

## Purpose
TBD - created by archiving change establish-k3-source-tracking-baseline. Update Purpose after archive.
## Requirements
### Requirement: 选定来源组接受真实人工刷新

维护流程 SHALL 对 Gate 1 批准的 R01、R07 和 R08 共 7 个 URL 执行一次真实人工复核，并 SHALL 为每个 URL 记录一个且仅一个刷新结果。

#### Scenario: 选定来源均完成复核

- **WHEN** 维护者可以取得选定 URL 的当前页面、官方目录、响应或官方仓库状态
- **THEN** 每个 URL 分别获得 `unchanged`、`changed`、`moved`、`removed` 或 `unreachable` 之一，并记录已检查清单和下一恢复点

#### Scenario: 未检查来源保持旧状态

- **WHEN** 本 change 只检查批准的 7 个 URL
- **THEN** 其余已登记 URL 的观察日期、源端修订、访问状态和备注保持不变，且不获得本轮刷新结论

#### Scenario: 执行期来源集合改变

- **WHEN** 执行前或执行中发现 R01、R07 或 R08 的登记集合与批准基线不一致
- **THEN** 当前 Cycle 停止并返回 Plan，不静默扩大检查集合或把新来源标记为已检查

#### Scenario: 首次刷新只比较已持久化字段

- **WHEN** 2026-09-02 baseline 没有保存导航、K3 路径、默认分支或 release 正文的上次值
- **THEN** 首次 refresh 只用已持久化的 URL、来源身份和访问状态判断结果；新观察到的字段作为当前 baseline 记录，不参与 `unchanged` 或 `changed` 判定

#### Scenario: SPA 壳不能提供正文

- **WHEN** R01 或 R07 的 HTTP 请求成功，但响应只有 SPA 壳且无法取得所需正文
- **THEN** 该 URL 记录为 `unreachable`，不得从路由或 supporting source 推断官网正文

#### Scenario: 已持久化字段仍不足以比较

- **WHEN** 当前观察既不能确认已持久化的 URL 和来源身份，也不符合 `unreachable`、`moved` 或 `removed` 的证据条件
- **THEN** 当前 Cycle 停止并返回 Plan，不把证据缺失归类为 `unchanged` 或 `changed`

### Requirement: 刷新结果驱动受限的行级更新

维护流程 SHALL 按 `unchanged`、`changed`、`moved`、`removed` 和 `unreachable` 的既有语义更新来源覆盖记录，并 SHALL 保留历史 URL、未检查状态和证据边界。

#### Scenario: 来源未变化

- **WHEN** 页面修订与相关正文相对仓库基线均未变化
- **THEN** 该 URL 记录为 `unchanged`，只更新观察日期，不改写源端修订、访问状态、备注或技术正文

#### Scenario: 来源发生实质变化

- **WHEN** 页面修订、SDK 版本线索或相关正文发生变化
- **THEN** 该 URL 记录为 `changed`，更新获批范围内的观察元数据并列出受影响主题；若处理需要技术聚合或扩大批准范围则停止并返回 Plan

#### Scenario: 来源移动

- **WHEN** R01 入口、重定向或官方仓库信息确认旧 URL 已迁移到新 URL
- **THEN** 该 URL 记录为 `moved`，保留旧 URL 行及其聚合和访问状态，在备注中记录迁移，并为确认的新 URL 增加完整覆盖记录

#### Scenario: 来源永久下架

- **WHEN** 官方入口或明确响应证明页面已经永久下架
- **THEN** 该 URL 记录为 `removed`，保留旧 URL 行及其聚合和访问状态，并按获批范围登记无法替代的来源缺口

#### Scenario: 来源暂时不可访问

- **WHEN** 网络超时、临时错误或当前访问条件无法取得正文，且没有永久下架证据
- **THEN** 该 URL 记录为 `unreachable`，不得推断为 `removed`，不得静默替换来源，访问状态只在新证据支持时改变

### Requirement: SDK baseline 与来源权威边界保持可追溯

刷新结果 SHALL 区分 R01 权威入口、R07 SDK 页面和 R08 官方 supporting source，并 SHALL 记录能够直接观察到的 SDK 版本或修订线索而不提升证据等级。R07 正文不可读取时，流程 MAY 使用与该页面对应的 SpacemiT 官方 GitHub 文档建立交叉验证级 SDK baseline。

#### Scenario: R07 提供 SDK 版本线索

- **WHEN** `source.md` 或 `bl-v1.0.y.md` 当前内容明确给出 SDK、OpenSBI、U-Boot、Linux 或 Buildroot 的版本或修订信息
- **THEN** 刷新结果按官网事实记录该线索及观察日期，不用观察日期冒充源端修订

#### Scenario: R08 与官网信息一致

- **WHEN** 官方 GitHub 仓库能够佐证官网目录、版本或历史
- **THEN** 该信息只标记为 supporting 或交叉验证，R01 继续作为唯一权威正文来源

#### Scenario: R08 与官网信息不一致

- **WHEN** 官方 GitHub 仓库与 R01 或 R07 的可观察信息存在差异
- **THEN** 刷新结果明确记录差异和待解除条件，不以 R08 静默覆盖官网结论

#### Scenario: R07 不可读取时使用 supporting fallback

- **WHEN** R07 URL 因 SPA 壳无法取得正文，但对应 SpacemiT 官方 GitHub 文档可读取并明确给出 SDK 或 release 信息
- **THEN** R07 仍记录为 `unreachable`，SDK baseline 使用该官方 GitHub 文档并在 Act Response 明确标记为 `交叉验证`，不得改写为 R01 或 R07 官网事实

### Requirement: 人工刷新可以中断并恢复

维护流程 SHALL 允许在任意 URL 后取消或中断，并 SHALL 通过当前 Cycle 的已检查清单和 `next` 恢复，而不重写已完成结论。

#### Scenario: 部分 URL 完成后中断

- **WHEN** 维护者因取消、网络问题或外部条件只完成部分 URL
- **THEN** 已检查 URL 保留逐项结论，未检查 URL 保持原值，当前 Cycle 保持未完成并记录下一 URL

#### Scenario: 从中断点恢复

- **WHEN** 维护者随后恢复同一 Cycle
- **THEN** 执行从 `next` 指向的 URL 继续，既有逐项结论不被聚合结论或默认 `unchanged` 覆盖

#### Scenario: 同一 URL 连续不可访问

- **WHEN** 同一 URL 连续三次无法访问且仍无替代来源或永久下架证据
- **THEN** 当前执行停止并返回 Plan，不开始第四次同类尝试

### Requirement: MS02 验收直接检查目标状态

MS02 的验证 SHALL 检查逐 URL 结果、覆盖表行级差异、SDK baseline、来源权威边界和恢复状态，并 SHALL 不依赖仓库内自动化工具或身份型 Evidence 工程。

#### Scenario: 首次真实 refresh 完成

- **WHEN** 批准集合中的全部 URL 已复核且计划内文档更新完成
- **THEN** 每个 URL 都有可审计的独立结论，已检查行与结论一致，未检查行无变化，SDK baseline 标明官网事实或交叉验证等级，OpenSpec change 验证通过

#### Scenario: 验证发现越界修改

- **WHEN** diff 包含未批准 URL、技术正文、全局项目状态或可执行工具的修改
- **THEN** 验证失败，当前 Cycle 不得报告完成

#### Scenario: 不需要持久化 Evidence

- **WHEN** 命令、决定性输出、退出码和逐 URL 结论可以在 Act Response 中完整表达
- **THEN** Persisted Evidence 使用 `none`，不创建 Evidence 目录或运行身份材料

