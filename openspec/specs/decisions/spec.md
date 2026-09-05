# Decisions

## Purpose

记录 K3 信息汇总项目重要选择的原因、替代方案、影响和状态。条目使用
`Dxx`, 被替代后保留并标记 `superseded`。

## Requirements

### Requirement: 决策可追溯

重要选择 SHALL 记录决定、原因、替代方案、影响、状态和关联模型。

#### Scenario: 接受长期选择

- **WHEN** 开发者确认跨模块、兼容性或长期设计选择
- **THEN** 使用递增 D 编号记录 accepted 决策

#### Scenario: 替代旧决策

- **WHEN** 新决策替代已有选择
- **THEN** 保留旧条目并标记 superseded 和替代编号

---

## D01 — 纯 Markdown 聚合, 不内置抓取工具

- 决定: 仓库只承载人工整理后的 Markdown, 不附带任何抓取, 解析或渲染脚本。
- 原因:
  - 用户明确选择"纯文档/Markdown 汇总"形态。
  - 源页面结构变更概率不可忽略; 自动化抓取需要持续维护, 短期收益不抵维护成本。
  - 仓库保持纯文本便于跨平台协作, 无运行时依赖。
- 替代方案:
  - 使用 Python (requests + BeautifulSoup) 或 Node.js (cheerio) 自动抓取并生成。
  - 使用 MkDocs / Docusaurus 直接从源构建静态站点。
  - 将页面 PDF 整篇入库。
- 影响:
  - 每次源端更新需要人工开 change, 不能由脚本自动重生成。
  - 文档质量取决于人工核对, 但保留了术语与结构的可控性。
- 关联模型: M01, M04
- 状态: accepted

## D02 — 主题驱动目录, 不镜像源页面树

- 决定: `docs/` 下的目录与文件按主题划分, 而非按源页面 URL slug 复制。
- 原因:
  - 源页面导航树与读者关心的主题并非一一对应, 例如同一 SoC 概述可能跨多个源页面。
  - 主题化目录便于读者按板子功能模块 (CPU, 内存, 外设, 启动, 调试) 检索。
  - 源端改版时, 主题目录结构保持稳定, 减少对读者的引用失效。
- 替代方案:
  - 1:1 镜像源页面树, 路径与 slug 一致。
  - 完全平铺到 `docs/` 顶层。
- 影响:
  - 每个文档必须显式声明来源页 (M02 规定 `> 来源:` 行), 否则无法回溯。
  - 重构主题分类时, 旧路径需要保留 redirect 或在 `references/spec.md` 登记迁移记录。
- 关联模型: M02
- 状态: accepted

## D03 — 六个实际文档承载基础职责

- 决定: 文档基础由 `docs/index.md`、`docs/reference/source-coverage.md`、`docs/reference/document-template.md`、`docs/reference/terminology.md`、`docs/reference/known-gaps.md` 和 `docs/reference/source-refresh.md` 六个产品文档承载, 不为空主题建立占位 overview。
- 原因:
  - 来源覆盖、写作模板、术语、缺口和刷新规则相互独立, 不在单一文档内并列可避免职责重叠。
  - `docs/index.md` 只导航到真实文件, 避免出现无法稳定提供事实的占位文档。
  - 后续主题 (platform、boot、interrupts、serial、dma、network、storage、buses、peripherals) 在产生真实技术内容时才创建子目录, 维持 D02 主题驱动约束。
- 替代方案:
  - 为每个未来主题建立空 overview 占位文件。
  - 把术语、缺口和模板合并为单一份 reference。
- 影响:
  - `docs/index.md` 中未创建的主题位置只列职责, 不写占位正文, 保持相对链接的稳定可达性。
  - 任何新增 reference 文档必须显式被 `index.md` 引用, 否则视为未交付。
- 关联模型: M02, D02
- 状态: accepted

## D04 — URL 是覆盖记录的唯一键

- 决定: `source-coverage.md` 以源页面 URL 作为覆盖记录的唯一键, 每个 URL 占一行, 不为同一 URL 引入额外编号或身份。
- 原因:
  - URL 本身已能稳定去重, 额外身份会增加维护成本并引入同步风险。
  - 一行一字段结构便于按 URL、目标范围、聚合状态、访问状态、观察日期等维度检索和筛选。
  - 字段语义 (来源职责、目标范围、主题位置、优先级、聚合状态、源端修订、观察日期、访问状态、备注) 共同表达一次 refresh change 所需的元数据。
- 替代方案:
  - 为每个 URL 分配内部编号 (R-1, R-2, …) 作为唯一键。
  - 在 R04-R08 之外另建覆盖标识。
- 影响:
  - 当源页面 URL 变化时, 旧 URL 行保留并标记 `备注: 已迁移至 <新 URL>`, 不静默替换。
  - 主题文档首行的 `> 来源:` 必须与覆盖表某行完全一致, 否则视为未登记来源。
- 关联模型: M01, M02
- 状态: accepted

## D05 — 主题文档首行同时表达修订与观察时间

- 决定: 主题文档首行采用 `> 来源: <URL>（源端修订: YYYY-MM-DD 或 unknown；观察日期: YYYY-MM-DD）`, 多来源文档把全部直接来源放在同一行, 用分号分隔。
- 原因:
  - 源页面未必公开修订日期, 必须区分源端修订与本仓库观察日期, 不混用。
  - 单行表达所有直接来源便于在编辑器侧栏、生成器与 grep 检索中快速识别。
  - 流程性文档 (如 `source-refresh.md`) 也遵守同一格式, 把 R01 作为来源, 不把仓库规则伪装成官方硬件事实。
- 替代方案:
  - 仅记录源端修订, 缺省时用本地观察日期冒充。
  - 在文档正文嵌入完整来源表, 不写在首行。
- 影响:
  - 任何新主题文档首行必须同时含 URL 和观察日期, 否则视为不合规交付。
  - 源页面 URL 变化时, 旧行 `备注` 注明迁移事实, `> 来源:` 行由独立 change 改写。
- 关联模型: M02, D04
- 状态: accepted

## D06 — 证据强度采用四级标记

- 决定: 文档事实分为四级: `官方事实` (官网正文直接陈述)、`交叉验证` (官方 GitHub、DTS 或驱动用于定位和核对)、`推论` (由多个事实推出但源页面未直接陈述)、`未知项` (当前证据不足, 列出解除条件)。
- 原因:
  - 仅靠"已确认"或"未确认"二分法无法表达 K3 资料在多源交叉时的实际差异。
  - 术语表至少固定 K3、CoM260 Kit、SoC、AP、RCPU、AIA、APLIC、IMSIC、MMIO、IRQ、DMA、IOMMU、GMAC、PHY、MDIO、RGMII、polling、async、waker、coherency 的主写法, 减少后续正文同义并列。
  - `known-gaps.md` 至少记录官网总页数、未展开目录、GMAC/PHY、AIA routing、DMA coherency/IOMMU 和 programmer reference 六类缺口, 避免后续 change 重复声明同一缺口。
- 替代方案:
  - 单一"已确认"与"待核实"两分标记。
  - 完全依赖自然语言修饰, 不建立显式分级。
- 影响:
  - 任何技术结论必须标注证据等级, 不得把推论或交叉验证结果当作官方事实。
  - 术语新增或主写法变更需经 `openspec-plan` 评审, 在 `terminology.md` 同步。
- 关联模型: M03
- 状态: accepted

## D07 — 刷新状态与聚合状态分离

- 决定: 覆盖表保存长期聚合状态; 人工刷新流程定义一次检查的结果 (`unchanged`、`changed`、`moved`、`removed`、`unreachable`), 两者由不同字段承载, 不允许把一次检查结果写入长期状态枚举, 也不在覆盖表增加"本轮刷新完成"全局列。
- 原因:
  - 长期状态 (active / deferred / out-of-scope / supporting) 和一次刷新结果互相不可替代: 前者描述文档范围, 后者描述本次核对结论。
  - 部分完成检查只更新已检查行的观察日期与访问状态, 未检查行保持原值, 由行级元数据决定恢复点, 避免全局完成标志导致的误判。
  - 维护每次检查的运行日志或 manifest 属于身份型证据工程, 与本仓库的纯 Markdown 流程不匹配。
- 替代方案:
  - 在覆盖表增加"本轮刷新"列或全局标志位。
  - 维护每次检查的运行 ID、manifest 和 hash 账本。
- 影响:
  - `changed` 必须先创建 refresh change, 获批后再修改主题正文; `moved` 保留旧 URL; `removed` 与 `unreachable` 在 R01 无法定位替代来源时进入 `known-gaps.md`, 不静默替换。
  - 任何修改覆盖表行级字段的 refresh change 必须随附 Act Response 中的逐 URL 结论与已检查清单, 否则视为不完整交付。
- 关联模型: M01, M02
- 状态: accepted

## D08 — 配置修复只增加必要引用

- 决定: `openspec/config.yaml` 第 51 行使用未引用 YAML plain scalar 时, 修复只对该行加引号, 规则文字与其他配置保持不变。
- 原因:
  - 该行原本表达 `status: pending` 等 artifact rule, 修复目的是让 OpenSpec CLI 加载既有规则, 不重新设计规则。
  - 修改范围控制在 YAML quoting 层面, 不改变规则语义、字段或顺序, 便于审计时定位差异。
  - 该修复是后续 `docs/` 文档被 OpenSpec 规则覆盖的前提, 必须在创建文档前完成。
- 替代方案:
  - 重新设计整份 `config.yaml` 的 artifact rules。
  - 关闭现有 parse warning 而不修配置, 例如通过 CLI 选项禁用检查。
- 影响:
  - 后续 OpenSpec change 直接继承现有 artifact rules, 不再出现 parse warning。
  - 任何对 `config.yaml` 的进一步修改必须保留规则文字与字段顺序, 仅允许最小 quoting 调整。
- 关联模型: 无
- 状态: accepted
