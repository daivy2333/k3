## Why

K3 文档仓库已经识别出至少 52 篇官方主题页，但 `docs/` 尚无稳定的来源覆盖表、主题导航、写作模板和刷新规则。若直接开始整理 CoM260 硬件事实，页面会被重复归类，来源版本与未知项也无法一致追踪。

本 change 先建立后续聚合所需的文档基础，同时覆盖 MS01 和 MS02 的基础部分；不把 milestone 与 change 绑定为一一对应关系。

## Source Baseline and Change Type

- 权威入口: https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs
- 来源集合: R01、R04-R08
- 最近完成的来源观察: 2026-09-02
- Change 类型: initial aggregation + restructure
- 说明: 当前仓库还没有可比较的旧版 `docs/` 正文，因此本 change 建立刷新机制并记录初始观察基线，但不虚构一次实际的源版本变更。MS02 要求的首次真实 refresh 可由后续任一来源更新 change 完成。

## What Changes

- 建立 K3 官方资料覆盖表，记录每个已发现 URL 的来源类别、观察日期或版本、聚合优先级、目标主题和当前状态。
- 建立主题驱动的 `docs/` 导航和目录职责，当前目标板固定为 K3 CoM260 Kit；其他 K3 板卡只登记为 deferred，不聚合正文。
- 建立统一的文档模板、术语表和已知缺口表，明确区分官方事实、交叉验证信息、推论和未知项。
- 建立人工来源刷新指南，覆盖 unchanged、changed、moved、removed 和 unreachable 情形，以及变更前后的核对步骤。
- 对 `openspec/config.yaml` 做最小 YAML 语法修复，使 OpenSpec CLI 能加载现有 artifact rules；规则语义不变。

## Capabilities

### New Capabilities

- `documentation-foundation`: 定义来源覆盖、主题导航、文档格式、术语与未知项表达，以及人工刷新行为。

### Modified Capabilities

- None.

## Scope Decisions

- 默认把 R04-R08 中的全部 URL 纳入覆盖表，即使页面 deferred、暂时不可访问或仅用于交叉验证，也不得静默遗漏。
- 首个 change 分为两个逻辑 Iteration：先建立覆盖与结构，再建立刷新与一致性基线。
- `docs/` 是聚合内容的唯一产品输出位置；OpenSpec change 目录只保存需求、设计、任务和 Cycle。
- 错误处理表现为显式状态和缺口记录，不以猜测内容填补页面访问失败或官方资料缺失。
- 本仓库没有运行时并发、取消、安全或性能路径；相关场景不适用。需处理的是人工操作中断后的可恢复性和来源兼容性。

## Non-goals

- 不聚合 CoM260 datasheet、boot、UART、interrupt、DMA 或 GMAC 的技术正文；这些属于 MS03 及以后。
- 不聚合 Pico、RV2768、Shelf 或 K3 之外的板卡正文。
- 不创建 scraper、link checker、renderer 或其他可执行脚本。
- 不修改 StarryOS。
- 不在本 change 中宣称 MS02 的首次真实 source refresh 已完成，也不同步 milestone 状态或 SNAPSHOT。

## Impact

- 新增 `docs/` 下的入口、来源覆盖、术语、缺口、模板和刷新指南文档。
- 最小修复 `openspec/config.yaml` 的现有 YAML 解析错误，使既有 proposal、tasks 和 spec rules 生效。
- 不影响应用代码、API、运行时依赖或构建工具链。
- 后续 CoM260 聚合 change 将依赖本 change 的目录职责、来源元数据和写作约束。
