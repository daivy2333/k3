# Improvements

## Purpose

记录有证据但尚未承诺实施的改进机会。条目使用 `Ixx`, 包含分类、问题、
证据、影响、建议和状态。被批准后创建 change, 并把原条目标记 `promoted`。

## Requirements

### Requirement: 改进项可评估

改进项 SHALL 包含分类、问题、证据、影响、建议和状态。

#### Scenario: 发现未排期问题

- **WHEN** 已有证据表明存在改进机会但尚未批准实施
- **THEN** 使用递增 I 编号记录

#### Scenario: 批准实施

- **WHEN** 用户批准实施改进项
- **THEN** 创建 OpenSpec change 并把原条目标记 promoted

---

<!-- 改进条目将在实际运行中产生 (如: 源页面发现矛盾术语, 主题目录需要重分类,
     源端更新后某文档需要刷新等)。初始化阶段不创建占位条目。 -->
