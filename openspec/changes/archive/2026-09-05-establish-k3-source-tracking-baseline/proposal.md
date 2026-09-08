## Why

MS01 已建立来源覆盖与人工刷新规则，但全部观察日期仍来自 2026-09-02 的初始 baseline，尚未执行一次真实 refresh。MS02 需要用一个边界明确、可中断恢复的来源组完成实际复核，证明后续主题文档能够追踪官网、SDK 和交叉验证来源的变化。

## Source Baseline and Change Type

- 权威入口: https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs
- 当前仓库观察基线: 2026-09-02
- 源端修订: unknown
- Change 类型: refresh；不预设来源已经发生变化
- 对应 milestone: MS02，一个 milestone 对应本 change

## What Changes

- 对 R01、R07 和 R08 当前登记的 7 个 URL 执行首次真实人工 refresh，并为每个 URL 单独记录 `unchanged`、`changed`、`moved`、`removed` 或 `unreachable` 结论。
- 按实际结论更新 `docs/reference/source-coverage.md` 的行级观察元数据；未检查行保持原值。
- 核对 R07 的 SDK 来源与版本线索，保持官网正文和 R08 supporting source 的权威边界。
- 验证中断恢复、超时或不可访问、URL 迁移、永久下架和执行期 baseline 变化的停止条件。
- 为 change 收尾后的 R01、R07、R08 引用同步提供逐 URL 结果；本 change 不直接维护全局 references 或 milestone 状态。

## Capabilities

### New Capabilities

- `source-refresh`: 定义选定来源组的真实人工刷新、逐 URL 结果、行级状态更新、权威边界和中断恢复行为。

### Modified Capabilities

- None.

## Scope Decisions

- Replan 决定（用户于 2026-09-05 回复“同意”）：首次 refresh 只比较 2026-09-02 已持久化字段；本轮首次观察到的导航、路径和分支作为新 baseline，不要求不存在的上次值。R01/R07 正文无法渲染时记为 `unreachable`；SDK baseline 可以使用对应 SpacemiT 官方 GitHub 文档，但必须标为交叉验证，不提升为 R01 官方事实。

- 默认检查集合为 7 个 URL：R01 权威入口、R07 的 `source.md` 与 `bl-v1.0.y.md`、R08 的 `docs-chip`、`docs-product`、`docs-buildroot` 与 `linux-6.18`。
- 该集合同时覆盖权威入口、SDK baseline 和官方交叉验证来源；不以全量 38 URL refresh 作为 MS02 的完成条件。
- 逐 URL 实际结论只能在 Act 执行时产生。计划和验证不得预设所有来源均为 `unchanged`。
- 若复核发现会改变 K3 技术正文的新事实，只记录受影响主题并停止扩大范围；技术聚合留给 MS03 及后续 change。
- 本仓库不存在运行时并发。并发编辑不进入需求；人工取消、网络超时和部分完成按中断恢复场景处理。

## Scenario Gaps and Defaults

- 检查范围采用上述 7 个 URL；用户可以在 Gate 1 改为其他不超过 10 个 URL 的单一来源组。
- 一次或暂时性访问失败默认为 `unreachable`，不能推断为 `removed`。
- 执行期若 R01、R07、R08 登记集合与计划基线不一致，默认停止并返回 Plan，不静默扩大检查集合。
- 未检查 URL 不更新观察日期，也不得获得任何刷新结论。

## Non-goals

- 不刷新全部 38 个已登记 URL。
- 不聚合 CoM260 board、boot、UART、interrupt、DMA、GMAC、PHY 或其他技术正文。
- 不把 R08 的 GitHub 仓库提升为 M01 权威正文来源。
- 不创建 scraper、页面 diff、链接检查器、manifest、run ID、Hash 账本或 Evidence 占位目录。
- 不修改 StarryOS，不处理 K3 之外的板卡或官网章节。
- 不同步 SNAPSHOT、tasks、references 或 milestone 状态；这些由 change accepted 后的 `openspec-docs-maintainer` 处理。

## Impact

- 计划中的产品修改以 `docs/reference/source-coverage.md` 为主；只有实际结果要求且仍在获批范围内时，才修改 `docs/reference/known-gaps.md` 或 `docs/reference/source-refresh.md`。
- OpenSpec change 保存需求、设计、任务、Cycle 和逐 URL Act Response，不新增运行时依赖或可执行内容。
- 后续 MS03-MS07 可以引用本 change 形成的最近观察日期、SDK baseline 和来源处置边界。

## Gate 1

- Status: approved
- User approval: `同意`
- Approved scope: 7 URL 检查集合、Scenario Gaps and Defaults 和 Non-goals 保持草案内容不变。

| Check | Status | Evidence |
| --- | --- | --- |
| BDD gap scan | PASS | Scope Decisions 与 Scenario Gaps 覆盖正常、变化、移动、删除、不可访问、中断、baseline 变化、取消/超时和兼容性 |
| Gap decisions | PASS | 用户接受 7 URL 默认范围及全部默认处理 |
| Scenario sketch | PASS | `source-refresh` delta spec 的 R1-R5 共 18 个场景 |
| OpenSpec change | PASS | `establish-k3-source-tracking-baseline` 已创建且 strict validate 通过 |
| Requirements and scope approval | PASS | 用户原话：`同意` |
