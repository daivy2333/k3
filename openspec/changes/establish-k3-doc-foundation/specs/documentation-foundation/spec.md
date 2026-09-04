## ADDED Requirements

### Requirement: 已发现来源具有完整覆盖记录

文档体系 SHALL 为 R01、R04-R08 中每个已发现 URL 保存唯一的覆盖记录，记录来源类别、URL、观察日期或已知版本、目标主题、聚合优先级和状态。

#### Scenario: 正常登记可访问来源

- **WHEN** 官方页面可以定位且职责明确
- **THEN** 覆盖表记录其元数据、主题位置和 active 或 deferred 状态

#### Scenario: 页面不可访问或版本未知

- **WHEN** 页面暂时不可访问或无法确认版本
- **THEN** 覆盖表保留该 URL 并将未确认字段显式标记为 unknown 或 unreachable，不得删除或猜测

#### Scenario: 非当前目标板资料被发现

- **WHEN** 来源属于 Pico、RV2768、Shelf 或其他非 CoM260 板卡
- **THEN** 覆盖表将其标记为 deferred 或 out-of-scope，且不创建对应聚合正文

### Requirement: 聚合内容使用主题驱动导航

仓库 SHALL 在 `docs/` 下提供主题驱动的入口和目录职责，使来源页面与聚合文档可以多对一或一对多映射，而不镜像官网 URL 树。

#### Scenario: 读者从总入口定位资料

- **WHEN** 读者打开 `docs/index.md`
- **THEN** 可以通过相对链接定位 source coverage、terminology、known gaps 以及 platform、boot、interrupts、serial、dma、network、storage、buses 和 peripherals 主题职责

#### Scenario: 新建聚合文档

- **WHEN** 后续 change 在 `docs/` 创建主题文档
- **THEN** 文档使用 kebab-case 文件名，首行 `> 来源:` 同时记录源页面 URL 和观察到的修订日期，并遵守 500 行拆分建议

#### Scenario: 一个主题依赖多个来源

- **WHEN** 一个主题需要芯片、板卡和 SDK 页面共同佐证
- **THEN** 文档首部保留所有直接来源及各自观察日期，并在正文中维持事实归属

### Requirement: 术语和证据强度可区分

文档体系 SHALL 统一 K3 CoM260 相关术语，并 SHALL 区分官方事实、交叉验证信息、推论和未知项。

#### Scenario: 同一术语存在多种写法

- **WHEN** 来源或既有材料对同一概念使用不同中文或英文名称
- **THEN** terminology 文档选择一个主写法、保留必要英文原词并记录别名，不得在标题层并列同义名称

#### Scenario: 资料不足以支持硬件结论

- **WHEN** product brief、Linux 指南或源码只能提供部分证据
- **THEN** 文档将结论标记为交叉验证、推论或未知项，不得提升为已确认的寄存器级事实

### Requirement: 来源变化可以人工刷新

文档体系 SHALL 提供不依赖仓库内脚本的人工刷新流程，覆盖 unchanged、changed、moved、removed 和 unreachable 状态，并允许中断后从已记录状态继续。

#### Scenario: 来源内容没有变化

- **WHEN** 维护者按刷新指南复核页面且内容与版本均未变化
- **THEN** 记录新的观察日期和 unchanged 结论，不改写无关聚合正文

#### Scenario: 来源内容或版本发生变化

- **WHEN** 页面修订日期、版本或相关正文发生变化
- **THEN** 维护者创建 refresh change，列出受影响主题并在获批后更新对应文档来源行和内容

#### Scenario: 来源移动或删除

- **WHEN** 页面 URL 移动、删除或长期不可访问
- **THEN** 维护者保留旧 URL 和状态，尝试从 R01 重新定位，并在无法确认替代来源时记录缺口而不静默切换

#### Scenario: 人工刷新被中断

- **WHEN** 维护者只完成部分 URL 的复核
- **THEN** 已复核与未复核条目保持可区分状态，后续执行可以继续而不把未检查页面标为 unchanged

### Requirement: OpenSpec 项目规则可被 CLI 加载

`openspec/config.yaml` SHALL 是有效 YAML，现有 artifact rules 的语义 SHALL 在语法修复后保持不变。

#### Scenario: 请求 change artifact instructions

- **WHEN** 维护者运行 OpenSpec instructions 或 status 命令
- **THEN** CLI 不报告 config parse warning，并加载 proposal、tasks 和 spec 的项目级规则

#### Scenario: 修复配置语法

- **WHEN** 为消除当前第 51 行的 YAML 隐式映射解析错误而修改配置
- **THEN** 仅改变必要的 YAML quoting，不改变规则文本表达的约束
