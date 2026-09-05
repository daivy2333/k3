# Tasks

> 维护者: `openspec-milestone-planner` 负责 `MSxx` 路线; `openspec-docs-maintainer` 负责状态同步。
> 当前状态: MS01 已完成, MS02-MS07 待办, 无进行中任务, 无已承诺待办。

## Milestone Roadmap (MSxx)

### MS01 — 来源覆盖与主题结构基线

- Status: completed
- Outcome: 建立 K3 官网资料的来源覆盖表、主题目录、文档模板和术语种子表, 使每个已发现页面都有明确职责、优先级和预期落点。
- Rationale: 来源清点、主题归类和写作约束共同决定后续所有聚合文档的可追溯性; 单独完成后即可作为各主题 change 的稳定入口。
- Dependencies: None
- Scope: 清点 R01、R04-R08; 区分首批、未来候选、交叉验证和暂不聚合资料; 建立 `docs/index.md`、`docs/reference/` 以及 platform、boot、interrupts、serial、dma、network、storage、buses、peripherals 等主题位置; 定义事实、推论、未知项、版本和术语的表达规则。
- Non-goals: 不完成全部硬件正文; 不选择 K3 之外的平台; 不创建抓取或校验脚本。
- Workload: 中; 需要人工核对至少 52 篇已观察主题页及尚未完整展开的目录, 并解决来源页到主题文档的多对多映射。
- Stable baseline: 所有已发现资料都能从覆盖表定位到一个主题职责; 新文档可直接使用统一模板, 无需重新决定目录和术语风格。
- Verification boundary: R04-R08 的每个 URL 均被覆盖表引用; 目录符合 D02; 模板满足 M02、M03; 每个文档职责唯一; 仓库内没有新增可执行内容。
- Diagnostic boundary: 失败范围限制为来源漏记、主题归类冲突、模板不合规或链接失效, 不进入具体硬件事实争议。
- Split signals: 若新发现的未展开目录显著扩大清点工作, 将补充清点放入独立 change, 但不拆分本 milestone 的结构基线成果。
- Related changes: `establish-k3-doc-foundation` (2026-09-05 收尾, archived)
- Related references: R01, R03, R04, R05, R06, R07, R08

### MS02 — 来源追踪与人工刷新基线

- Status: planned
- Outcome: 建立适用于纯 Markdown 仓库的版本追踪、链接迁移和人工刷新机制, 使来源变化不会形成静默差异。
- Rationale: 大量正文聚合前必须固定观察日期、版本和失效处理规则; 否则后续无法判断差异来自官网更新还是整理错误。
- Dependencies: MS01
- Scope: 记录页面观察日期、文档版本和 SDK 基线; 定义 changed、moved、removed、unreachable 状态; 明确官网正文与官方 GitHub 交叉验证边界; 建立 refresh change 的人工检查清单; 以至少一组 MS01 来源完成实际复核。
- Non-goals: 不实现自动抓取、页面 diff 或链接检查程序; 不把 GitHub 仓库提升为正文权威源; 不聚合新的设备主题。
- Workload: 中; 需要设计并实际演练跨页面、跨版本的人工追踪闭环。
- Stable baseline: 后续文档均可回答依据哪个来源版本整理、何时复核、链接变化后如何处置。
- Verification boundary: 至少一次 refresh change 达到 `accepted`; R01 的最近观察日期与被复核文档一致; 页面迁移和不可访问情形都有可执行处理步骤。
- Diagnostic boundary: 失败范围限制为页面变化、版本遗漏、链接迁移、访问异常或交叉验证边界错误。
- Split signals: 若官网长期要求认证、阻断访问或无法稳定定位页面, 停止刷新并转 Incident, 不扩大本 milestone。
- Related changes: None
- Related references: R01, R07, R08

### MS03 — K3 CoM260 板级与启动事实基线

- Status: planned
- Outcome: 形成面向 StarryOS 目标硬件 K3 CoM260 Kit 的 SoC、模组、底板和启动事实包, 支撑其目标板 bring-up 规划。
- Rationale: StarryOS 已确定使用 K3 CoM260; 后续 platform、IRQ、DMA 和 GMAC 工作都依赖统一且可追溯的板级事实。
- Dependencies: MS01, MS02
- Scope: 聚合 K3 product brief 与 datasheet; CoM260 overview、datasheet、hardware resources 和 user guide; 区分 SoC、CoM260 模组与 Kit 底板资源; 整理 CPU/hart、DRAM、UFS、SPI Flash、TF Card、debug UART、GMAC/PHY 和连接器; 整理 boot media、OpenSBI/U-Boot handoff、镜像与 DTS 使用方式; 登记缺失的寄存器级资料。
- Non-goals: 不聚合 Pico、RV2768 或 Shelf 正文; 不推定未由官网、DTS 或实机证据确认的装载地址、DRAM 布局、GMAC 实例或 PHY 参数; 不修改 StarryOS。
- Workload: 中到大; 需要协调芯片、模组、底板、SDK 和 DTS 多层资料并消除同名资源歧义。
- Stable baseline: StarryOS 可以直接引用 CoM260 的目标板事实规划 platform change, 不必重新遍历 K3 官网或比较其他板型。
- Verification boundary: SoC、模组和底板事实明确分层; boot、memory、console、network 和 storage 均有来源与未知项; CoM260 相关结论不混入其他 K3 板卡事实。
- Diagnostic boundary: 失败范围限制为 SoC/板级混淆、版本差异、启动链缺口、DTS 缺失或模组与底板资源归属不清。
- Split signals: 若单一 CoM260 主题超过 M02 的 500 行建议上限, 按 boot、memory、board-resources 等文档拆分, 但保持同一 milestone。
- Related changes: None
- Related references: R01, R03, R04, R05, R08

### MS04 — CoM260 平台资源与串口基线

- Status: planned
- Outcome: 形成 K3 CoM260 early console 和设备上电前置条件的完整资料包。
- Rationale: pinctrl、clock、reset 和 UART 共同构成最小可观测 bring-up 链路; 单独完成后可隔离平台资源问题与通用中断、DMA 或网络问题。
- Dependencies: MS02, MS03
- Scope: 聚合 CoM260 pinctrl、clock、reset 和 UART; 整理 MMIO width/stride、FIFO、threshold、clock/reset 顺序、引脚复用、DTS 字段以及 AP UART 与 RCPU UART 的差异。
- Non-goals: 不展开通用 AIA 实现、timer、DMA 或 GMAC 数据面; 不将 Linux `8250_of` 行为直接等同于 StarryOS 实现。
- Workload: 中; 需要跨 datasheet、板卡资源、Buildroot 指南和 DTS 对齐同一 console 实例。
- Stable baseline: 后续可据此规划 CoM260 platform descriptor、early console 以及 polling/IRQ UART bring-up。
- Verification boundary: 目标 UART 的 MMIO、width/stride、clock、reset、pinctrl、IRQ 引用和未知项闭环; AP/RCPU 域不会被混用。
- Diagnostic boundary: 失败范围限制为 pinmux、clock、reset、MMIO access、UART 配置或资源域选择。
- Split signals: 若 AP UART 与 RCPU UART 的时钟、安全域或编程模型无法共享验证边界, 再拆为两个 milestone。
- Related changes: None
- Related references: R03, R04, R05, R08

### MS05 — CoM260 中断与时间基线

- Status: planned
- Outcome: 形成 CoM260 有线中断、MSI 和时间源的统一资料基线。
- Rationale: UART、GMAC、PCIe 和异步设备都依赖正确的 hart routing 与完成语义; 首次证明中断模型可独立建立高价值故障边界。
- Dependencies: MS02, MS03
- Scope: 整理 AIA、APLIC、IMSIC 的角色与拓扑; hart/context、IRQ domain、routing、mask、ack/complete; wired IRQ 与 MSI/MSI-X 边界; timer、deadline、timeout 及 suspend/resume 相关事实。
- Non-goals: 不实现 StarryOS interrupt controller; 不声称 Linux 中断行为等同于裸机行为; 不展开具体设备数据面。
- Workload: 中到大; 需要协调 datasheet、CoM260 DTS、SDK release note 与官方内核来源, 并保留公开资料缺口。
- Stable baseline: 后续 UART、GMAC、PCIe 和异步设备文档可引用同一套 CoM260 中断术语、拓扑和生命周期。
- Verification boundary: 控制器能力、DTS 路由和 Linux 行为明确分层; wired IRQ 与 MSI 可区分; 缺失地址和 delivery 规则被显式登记而非推测。
- Diagnostic boundary: 失败范围限制为控制器拓扑、hart routing、mask/ack、MSI delivery 或 timer 五类。
- Split signals: 若 wired IRQ 与 IMSIC Multi MSI 由独立资料和验证路径支撑且无法形成共同基线, 拆为两个 milestone。
- Related changes: None
- Related references: R03, R05, R07, R08

### MS06 — CoM260 DMA、IOMMU 与内存一致性基线

- Status: planned
- Outcome: 形成异步网络、存储和流式设备可共同依赖的 DMA buffer ownership 与 coherency 资料。
- Rationale: 在接入设备 IRQ 或 async wakeup 前, 必须先能解释 descriptor 与数据缓冲区何时对 CPU 和设备可见。
- Dependencies: MS02, MS03
- Scope: 整理 DMA controller、channel、descriptor、burst、地址宽度、scatter-gather 和 cyclic DMA; 区分通用 DMA 与设备内建 DMA; 整理 cache coherency、cache line、barrier、map/unmap、IOMMU 和地址转换; 建立 CPU/device 所有权转换与完成可见性问题表。
- Non-goals: 不把 Linux DMA API 当作 K3 硬件规范; 不设计 StarryOS Rust API; 不实现 GMAC 或 storage driver。
- Workload: 中到大; 需要从能力说明、DTS 和官方驱动行为中分离硬件事实、软件约定与待实机验证项。
- Stable baseline: GMAC、storage、PCIe 和 Audio 资料可引用统一的 CoM260 DMA、地址与所有权模型。
- Verification boundary: 每项结论标明 datasheet 明示、官方驱动行为或待实机验证; descriptor/data buffer、CPU/device 与 coherent/non-coherent 边界完整。
- Diagnostic boundary: 失败范围限制为寻址、descriptor、cache、barrier、IOMMU 或完成可见性。
- Split signals: 若通用 DMA 与 GMAC/PCIe 内建 DMA 无法共享所有权和验证模型, 再拆独立 milestone。
- Related changes: None
- Related references: R03, R04, R05, R08

### MS07 — CoM260 GMAC、PHY 与异步网络资料基线

- Status: planned
- Outcome: 形成直接服务 StarryOS 目标板 polling NIC、IRQ NIC 和 async NIC 规划的 CoM260 网络驱动资料包。
- Rationale: GMAC/PHY 的可用基线同时依赖平台资源、中断和 DMA; 在前三项稳定后聚合, 可以保留清晰的 link、MAC、DMA、IRQ 与 async 故障边界。
- Dependencies: MS04, MS05, MS06
- Scope: 整理 CoM260 实际 GMAC 实例、MMIO、clock/reset 和 pinctrl; MDIO、PHY 型号与地址、link、RGMII delay、reset 与 ref clock; RX/TX descriptor ring、ownership 与 reclaim; interrupt cause、mask、ack 与 budget; polling 到 IRQ 再到 async 的分层关系; 与 StarryOS `NetDriverOps`、`NetQueueControl`、waker 和 recovery contract 的概念映射; timeout、link loss、ring stall 和 recovery 问题清单。
- Non-goals: 不实现 StarryOS 驱动; 不用 QEMU VirtIO 结果证明 CoM260 硬件行为; 不在缺少证据时预选通用 DWMAC 寄存器模型。
- Workload: 大; 需要把板级 link、MAC、DMA、IRQ 和异步通知五层资料整合为一个可独立使用的网络基线。
- Stable baseline: 后续 OpenSpec change 可以直接规划 CoM260 polling、IRQ 和 async NIC, 无需重新调查基础硬件与来源边界。
- Verification boundary: 平台、PHY、MAC、DMA、IRQ 和异步通知分层明确; 寄存器级未知项显式保留; polling、IRQ、async 和 recovery 均有独立验证入口。
- Diagnostic boundary: 失败范围可限制为板级 link、MAC 数据面、DMA/cache、IRQ delivery 或异步唤醒。
- Split signals: 若 GMAC 寄存器和 descriptor 资料足够完整且单篇超过 500 行, 拆为 hardware、DMA 和 async-integration 文档; 若 polling 与 IRQ/async 形成可独立验收的大型资料集, 再拆 milestone。
- Related changes: None
- Related references: R03, R04, R05, R07, R08

## 进行中

(无)

## 已承诺待办

(无)

## 阻塞

(无)

## 最近完成

- 项目初始化: 完成 OpenSpec 结构, specs, SNAPSHOT, tasks, change-cycle 模板, CLAUDE.md。
- MS01 / change `establish-k3-doc-foundation` (2026-09-05 收尾, archived): 完成 R01、R04-R08 的 38 个唯一 URL 来源覆盖、主题目录、文档模板、术语种子、6 类已知缺口与人工刷新流程; 修正 `openspec/config.yaml` 的 YAML quoting, 使 OpenSpec CLI 加载既有 artifact rules; 同时建立 MS02 的机制基础 (refresh change 工作流), 但首次真实 source refresh 仍由未来来源变更 change 验收。

## 与 OpenSpec Changes 的同步

- 每个 milestone 对应一个或多个 OpenSpec change (数量不绑定)。
- change `accepted` 时, 由 `openspec-docs-maintainer` 按完成范围把对应 milestone 推进到 `active` 或 `completed`, 并在"最近完成"追加引用。
- 未批准的想法不进入 tasks。
