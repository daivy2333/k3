## Context

仓库当前没有 `docs/` 目录。可复用的输入只有项目约束 M01-M04、决策 D01-D02、R01 与 R04-R08 登记的 38 个唯一 URL，以及 2026-09-02 捕获的 K3 官网分析。后续目标硬件已确定为 K3 CoM260 Kit。

当前 `openspec/config.yaml` 第 51 行把包含 `status: pending` 的规则写成未引用的 YAML plain scalar。OpenSpec CLI 返回成功，但报告 parse warning 并忽略整份项目配置，导致 artifact instructions 不包含本仓库的来源、文档形态和 task 规则。

本 change 是纯 Markdown 基础建设。它不需要运行时数据模型、并发控制、安全接口或性能设计；主要状态是来源的覆盖、访问和刷新状态。

## Goals / Non-Goals

**Goals:**

- 用一个覆盖表收纳 R01、R04-R08 当前登记的 38 个唯一 URL。
- 建立无需空目录占位文件的主题导航和职责边界。
- 固定后续主题文档的首行来源、证据强度、术语和缺口表达。
- 建立可中断、可继续的人工刷新流程。
- 以最小 quoting 修复使 OpenSpec CLI 重新加载既有项目规则。

**Non-Goals:**

- 不整理 CoM260 的 datasheet、boot、UART、interrupt、DMA、GMAC 或 PHY 技术正文。
- 不修改 references、SNAPSHOT、milestone 状态或 StarryOS。
- 不建立自动抓取、链接检测、页面 diff 或渲染工具。
- 不把初始来源观察伪装为一次真实 source refresh。

## Decisions

### D1：使用六个实际文档承载基础职责

新增以下产品文档：

- `docs/index.md`：总入口、主题职责和当前范围；
- `docs/reference/source-coverage.md`：唯一来源覆盖表；
- `docs/reference/document-template.md`：主题文档格式和多来源示例；
- `docs/reference/terminology.md`：主写法、英文原词和别名；
- `docs/reference/known-gaps.md`：已确认资料缺口和解除条件；
- `docs/reference/source-refresh.md`：人工刷新状态和操作流程。

`platform/`、`boot/`、`interrupts/`、`serial/`、`dma/`、`network/`、`storage/`、`buses/` 和 `peripherals/` 在 `docs/index.md` 中先记录职责和计划路径，不创建空目录或没有技术内容的占位 overview。后续 change 在产生真实主题内容时创建目录。

替代方案是为每个主题建立空 overview。该方案会产生无法提供稳定事实的占位文档，并让相对链接检查失去意义，因此不采用。

### D2：URL 是覆盖记录的唯一键

`source-coverage.md` 每个 URL 一行，使用以下字段：

| 字段 | 含义 |
| --- | --- |
| URL | 当前登记地址，也是唯一键 |
| 来源职责 | authority、official-doc、official-product 或 supporting-source |
| 目标范围 | CoM260、K3-common、non-target-board 或 workflow-support |
| 主题位置 | 当前或未来 `docs/` 主题职责 |
| 优先级 | current、future 或 supporting |
| 聚合状态 | active、deferred、out-of-scope 或 supporting |
| 源端修订 | 页面明确给出时记录，否则写 unknown |
| 观察日期 | 使用现有证据的 2026-09-02 |
| 访问状态 | observed、partially-observed 或 unverified |
| 备注 | 缺口、迁移或使用限制 |

当前 38 个 URL 是验收基线。若 Act 开始前 R01、R04-R08 已改变，属于 baseline change，必须停止并返回 Plan，而不是自行扩大覆盖范围。

替代方案是为 URL 另建编号。URL 已能稳定去重，额外身份会增加维护成本，因此不采用。

### D3：主题文档首行同时表达修订和观察时间

单来源文档首行采用：

```markdown
> 来源: <URL>（源端修订: YYYY-MM-DD 或 unknown；观察日期: YYYY-MM-DD）
```

多来源文档仍把全部直接来源放在第一行，以分号分隔；正文再说明每项事实归属。若源端未提供修订日期，必须写 `unknown`，不能用本地观察日期冒充源端修订。

本 change 的流程性文档引用 R01 权威入口，并在正文注明其结构规则来自 M01-M04 与 D01-D02。这样既满足 `docs/` 首行约束，也不把仓库规则伪装成官方硬件事实。

### D4：证据强度采用四级标记

- `官方事实`：官网正文直接陈述；
- `交叉验证`：官方 GitHub、DTS 或驱动用于定位和核对；
- `推论`：由多个事实推出但源页面未直接陈述；
- `未知项`：当前证据不足，列出解除条件。

术语表至少固定 K3、CoM260 Kit、SoC、AP、RCPU、AIA、APLIC、IMSIC、MMIO、IRQ、DMA、IOMMU、GMAC、PHY、MDIO、RGMII、polling、async、waker 和 coherency 的主写法。`known-gaps.md` 至少记录官网总页数、未展开目录、GMAC/PHY、AIA routing、DMA coherency/IOMMU 和 programmer reference 六类缺口。

### D5：刷新状态与聚合状态分离

覆盖表保存长期聚合状态；刷新指南定义一次检查的结果：`unchanged`、`changed`、`moved`、`removed`、`unreachable`。部分检查通过逐行更新观察日期和访问状态体现，未检查行保持原值，不设置全局“全部完成”标志。

`changed` 必须先创建 refresh change，再修改受影响主题正文；`moved` 保留旧 URL；`removed` 和 `unreachable` 不触发静默替换。初始建表只建立 baseline，不记作真实 refresh。

替代方案是维护每次检查的运行日志或 manifest。当前流程可由 change 和行级元数据追踪，无需建立身份型证据系统，因此不采用。

### D6：配置修复只增加必要引用

把第 51 行整个规则标量加引号，文字内容和其他配置保持不变。验证同时要求 CLI 不再出现 parse warning，并且 instructions 输出包含现有 project rules。

该任务是本 change 唯一不以 `docs/` 主题文件为目标的前置修复，已在 Gate 1 范围中明确批准；其余每个实施任务只负责一个 topic-level Markdown 文档。

## Iteration Design

### Iteration 000：覆盖与结构

先修复配置，再创建入口、覆盖表、模板、术语和缺口文档。完成后形成可供所有后续聚合使用的结构基线，并可独立验证 38 个 URL 是否完整、唯一。

### Iteration 001：刷新与一致性

在 Iteration 000 的覆盖字段和模板稳定后创建人工刷新指南，用当前 baseline 演练 unchanged、unreachable 和 partial-resume 三类路径，再把已存在的指南接入总入口。演练结果写入 Act Response，不创建运行身份或持久化 Evidence。

两个 Iteration 不按 milestone 数量划分：Iteration 000 完成 MS01；Iteration 001 建立 MS02 的机制基础，但首次真实 source refresh 仍由未来来源变更 change 验收。

## Risks / Trade-offs

- [官网目录仍可能不完整] → 覆盖表声明当前 38 个 URL 和至少 52 篇主题页只是已观察下界，未知子树保留为 gap。
- [流程文档引用官方入口可能被误解为官方原文] → 首行满足 M02，正文明确流程规则来自仓库 M/D，不把流程要求标成官方事实。
- [网页没有修订日期] → 使用 `unknown` 并单列观察日期，不合并两个概念。
- [未来 references 新增 URL] → 视为后续 change 输入；若在 Act 前已发生则停止并返回 Plan。
- [配置修复与内容任务职责不同] → 保持为单一、最小、先行任务，不借机调整规则语义。

## Migration and Rollback

仓库当前没有 `docs/`，无需迁移旧正文。实施顺序为 config quoting、Iteration 000 文档、Iteration 001 指南。若变更回退，删除本 change 新增的六个文档并恢复该行 quoting 即可；references、SNAPSHOT 和 StarryOS 不受影响。

## Open Questions

None. CoM260 的寄存器、IRQ、DMA 和 PHY 未知项属于后续技术聚合内容，不影响本 change 的结构契约。
