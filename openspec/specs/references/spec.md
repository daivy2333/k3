# References

## Purpose

索引项目依赖的内部产物和外部资料。条目使用 `Rxx`, 只记录检索元数据
(类型, 路径或 URL, 版本或日期, 用途, 状态), 不复制目标正文。

Change Evidence 位于所属 change 内, 由 change 提供索引, 不登记 R。

## Requirements

### Requirement: 参考可定位

参考 SHALL 记录类型、路径或 URL、版本或日期、用途和状态。

#### Scenario: 登记持久化产物

- **WHEN** 新分析、Runbook 或 Incident 需要跨会话复用
- **THEN** 使用递增 R 编号登记检索元数据

---

## R01 — SpacemiT K3 官方文档 (权威源)

- 类型: external-doc
- URL: https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs
- 语言: zh-CN
- 最近观察修订日期: (待首次聚合时由 change 记录)
- 用途: 本仓库 M01 指定的唯一权威源; 任何 K3 相关信息变更首先在此确认。
- 状态: active

## R02 — OpenSpec 工作流规范

- 类型: schema
- 路径: 仓库根 `openspec/`, `CLAUDE.md`, `.claude/`
- 版本: OpenSpec CLI 1.6.0
- 用途: 定义本仓库内的 change, spec, evidence, cycle 结构和验证流程。
- 状态: active
