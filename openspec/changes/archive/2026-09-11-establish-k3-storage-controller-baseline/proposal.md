## Why

MS03 已建立 K3 启动链和镜像基线，MS06 已建立 DMA、cache、PMA 与内存所有权边界，但 QSPI、SPI、SDHC 和 UFS 仍只有来源入口或分散事实，缺少按控制器区分的资源、启动用途、数据路径和恢复说明。MS09 需要建立存储控制器知识基线，使读者能够查询每类控制器的已证行为，并识别资料缺失、设备边界和不能外推的第三方经验。

本 change 由 SpacemiT K3 官方社区文档的 QSPI、SPI、SDHC、UFS、启动和 CoM260 相关章节驱动，最近观察到的源端修订日期为 2026-09-08。它属于基于已有资料的内容聚合与重组，不是源端更新后的 refresh。

## What Changes

- 建立 QSPI、SPI 和 SDHC 主题知识，分别整理控制器实例、DTS、clock/reset、pinctrl、IRQ、DMA、板级可达性、启动关系和错误边界。
- 建立独立的 UFS 主题知识，整理 MPHY、UniPro、UTP/SCSI、descriptor、DMA、cache maintenance、IRQ、fatal error、timeout 和 recovery，并区分官方资料、官方软件行为与固定 revision 第三方实现。
- 把启动介质与运行时数据路径分开说明；存在控制器或 IRQ 节点不等于该路径已启用或采用中断推进。
- 更新总索引、来源覆盖、术语和已知缺口，使每个存储来源、正文主题及未确认事实具有唯一落点。
- 对来源不可访问、资料冲突、CoM260 未引出、运行时状态未知或恢复语义不足的情况记录当前证据、影响和解除条件，不补写未经证实的行为。

### Planning Assumptions Pending Approval

- “下一个 change”按 roadmap 顺序指 MS09，不跳到 MS10 或 MS11。
- QSPI、SPI 和 SDHC 形成一个非 UFS 存储主题；UFS 因协议栈、DMA 和恢复边界不同而独立成文。实际调查若显示前一主题过重，只能在不改变需求范围的前提下拆文档。
- 只记录静态资料和可审计源码行为，不执行真板刷写、破坏性介质操作或运行时性能测试。
- 无法直接访问的官方页面按既有来源规则记录不可达和 supporting evidence，不以第三方材料替代官方硬件事实。
- Persisted Evidence 默认 `none`；可重跑的 Markdown、链接和 OpenSpec 校验结果写入 Act Response。

### Gate 1 Approval

- Status: PASS
- User instruction: `批准`
- Approved scope: MS09 的 QSPI、SPI、SDHC 和 UFS 知识基线，以及本 proposal 的场景、默认假设和 Non-goals。
- Approved at: 2026-09-10

### Non-goals

- 不实现或修改 QSPI、SPI、SDHC、UFS、DMA、IRQ、块设备或文件系统驱动。
- 不设计 Rust 存储 API，不评估吞吐、时延、功耗或介质寿命。
- 不执行刷写、擦除、分区、格式化或其他可能改变介质内容的操作。
- 不把 UFS 的 MPHY、UniPro、UTP/SCSI、cache 或恢复经验外推到 QSPI、SPI 或 SDHC。
- 不因 DTS 中存在节点、IRQ 或 DMA 属性而声明 CoM260 Kit 已连接、已启用或运行时采用该路径。
- 不扩展到 K3 之外的芯片、板卡或 SpacemiT 文档章节。

## Scenario Sketch

### S1：查询控制器静态资源与板级边界

- 前置状态：官方资料或 DTS 提供一个 QSPI、SPI、SDHC 或 UFS 控制器入口。
- 动作：读者查询实例、地址、clock/reset、pinctrl、IRQ、DMA 或 CoM260 可达性。
- 可观察结果：文档按控制器列出已确认资源、来源和证据等级，并区分 SoC 集成与板级连接。
- 失败边界：字段缺失、来源冲突或板级连接不可确认时，记录未知项及解除条件，不推定可用性。

### S2：查询启动介质关系

- 前置状态：启动文档或软件配置提到 QSPI、SPI、SDHC 或 UFS。
- 动作：读者查询该介质在 Boot ROM、SPL/U-Boot 或 OS 阶段的作用。
- 可观察结果：文档说明已证的阶段、参与者和交接关系，并链接既有启动基线。
- 失败边界：介质候选、默认启动目标或板级配置不能唯一确定时，保留多个候选及证据边界。

### S3：查询正常数据路径

- 前置状态：资料足以识别某控制器的数据传输对象及软件路径。
- 动作：读者查询命令、descriptor/buffer、DMA、cache、完成通知和资源所有权。
- 可观察结果：文档按设备描述 CPU 与控制器之间的准备、提交、完成和回收路径，并引用 MS06 的通用所有权模型。
- 失败边界：只发现 IRQ 或 DMA 描述而未确认运行路径时，不把能力声明为实际使用行为。

### S4：查询超时、取消与错误恢复

- 前置状态：官方软件或固定 revision 第三方实现含错误、轮询、超时或复位路径。
- 动作：读者查询命令超时、fatal error、链路失败、取消或重新初始化的结果。
- 可观察结果：文档记录可观察错误、资源状态、恢复动作、返回语义和证据适用范围。
- 失败边界：恢复是否保留未完成请求、是否要求硬件复位或是否可重试无法确认时，明确未知，不声称无损恢复。

### S5：来源不可访问或相互冲突

- 前置状态：官方 SPA 页面不可直接读取，或手册、DTS、驱动和第三方实现给出不同能力或参数。
- 动作：聚合者建立存储知识条目。
- 可观察结果：正文分别标注官方事实、官方软件行为、第三方经验、推论和未知项，来源覆盖表保持唯一记录。
- 失败边界：没有足够证据时停止形成硬件结论；缺口进入已有 known-gaps 体系，而非选择一个未经证实的值。

### S6：导航和兼容性检查

- 前置状态：存储主题文档加入仓库。
- 动作：读者从总索引或来源覆盖表进入主题，并沿相对链接访问相关启动、DMA 和平台文档。
- 可观察结果：链接有效，术语、来源日期、主题职责和缺口计数一致；既有 MS03/MS06 内容不被重复定义。
- 失败边界：发现旧文档与新证据冲突时，停止静默覆盖并返回 Plan 判断是本 change 内修正还是另建 refresh change。

## Capabilities

### New Capabilities

- `k3-storage-controller-baseline`: 规定 K3 QSPI、SPI、SDHC 和 UFS 的控制器资源、启动关系、数据路径、错误恢复、证据边界及导航要求。

### Modified Capabilities

- 无。

## Impact

- 预计新增 `docs/storage/` 下的非 UFS 存储主题和 UFS 主题文档，并修改 `docs/index.md`、`docs/reference/source-coverage.md`、`docs/reference/known-gaps.md` 与 `docs/reference/terminology.md`。
- 新增 change delta spec；不修改可执行代码、API、构建系统或运行时依赖。
- 复用 R04、R05、R08、R10、R12 及 MS03/MS06 行为规格作为调查输入；M01 指向的官方来源仍是硬件事实权威。
