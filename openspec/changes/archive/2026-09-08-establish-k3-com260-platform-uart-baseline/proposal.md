## Why

MS03 已记录 CoM260 Kit 的 UART0 接口和静态 `chosen` 线索，但平台控制、UART 实例、AP/RCPU 资源域及 console 路径仍散落在 datasheet、SDK 文档、DTS、官方驱动和第三方工程中。MS04 需要先建立分层、可追溯的事实基线，避免后续驱动设计混用资源域、版本或证据等级。

## Source Baseline and Change Type

- 权威入口: https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs
- 官网主题页: PINCTRL、UART、Clock、Reset，覆盖表观察日期为 2026-09-02，状态为 `partially-observed`，源端修订未知
- 板级基线: MS03 已归档的 CoM260 资源、启动、镜像与 DTS 文档，观察基线截至 2026-09-07
- 交叉验证来源: SpacemiT `docs-chip`、`docs-buildroot` 和 `linux-6.18` 官方仓库；仅证明对应文档或源码行为
- 第三方经验: `others/Rt-Async-AMP` 与 `others/tgoskits` 的固定 commit；只提取实现经验和待验证假设
- Change 类型: aggregation；整理已登记来源并补充本主题需要的精确 URL，不预设官网发生 refresh
- 对应 milestone: MS04，一个 milestone 对应本 change

## What Changes

- 创建 `docs/platform/k3-platform-control.md`，聚合 pinctrl、clock、reset、APBC/CCU、AP/RCPU 资源域及 UART 初始化依赖。
- 创建 `docs/serial/com260-uart.md`，聚合 UART 实例、MMIO width/stride、FIFO/threshold、IRQ、DTS 字段、console 路径和板级 UART0 线索。
- 修正 MS03 文档中已能由 `k3.dtsi` 静态解析的 `serial0` 映射，并把 console 未知项的后续归属从 MS07 调整到 MS04；运行时 bootloader 覆盖仍保留为未知项。
- 在引用前登记本 change 直接读取的精确官方仓库文档、DTS/DTSI、binding 或 driver URL；只在发现现有 G1-G7 未覆盖的新缺口时更新 `known-gaps.md`。
- 更新 `docs/index.md`，使 platform 与 serial 主题入口链接到交付文档。
- 分层整理 Rt-Async-AMP/tgoskits 中的 UUE/OUT2、THRE/TEMT、FIFO、IRQ budget、divisor errata 和初始化顺序，不把第三方行为写成硬件规范。

## Capabilities

### New Capabilities

- `k3-com260-platform-uart-baseline`: 定义 CoM260 平台控制、UART/console、来源分层、冲突保存和未知项闭包的聚合行为。

### Modified Capabilities

- None.

## Scope Decisions

- 默认交付两篇主题文档；若 AP UART 与 RCPU UART 的来源或编程模型无法共享一个清晰边界，可在 `docs/serial/` 内拆分，但不扩展 milestone。
- 官网是唯一 `官方事实` 来源；SpacemiT 官方 GitHub 内容标为 `交叉验证` 或 `官方源码行为`，Rt-Async-AMP/tgoskits 标为 `第三方经验`。
- `k3.dtsi` 的静态 aliases 可以证明 `serial0` 指向的控制器节点；它不能证明 bootloader 最终 cmdline、实际 console 或目标 Kit 的唯一顶层 DTS。
- K3 DTS 使用含 `k1` 的 compatible 时，文档记录字符串和实际绑定/驱动路径，不按名称推定为 K1 平台。
- FIFO 深度等来源冲突必须并列保留适用来源、版本和层级。当前官方 K3 DTS/SDK 的 256 与 tgoskits 的 64 不自行裁决。
- 第三方固定 clock/divisor、pinmux 自修复及寄存器初始化序列是实现经验；只有直接硬件资料能将其提升为硬件约束。

## Scenario Gaps and Defaults

- 官网 SPA 正文不可直接取得时，保持官网行 `partially-observed`，使用相应官方 GitHub 页面或源码交叉验证，不代填官网修订。
- CoM260 Kit 默认目标 DTS 仍未唯一映射；共享 `k3_com260.dtsi` 可用于静态共同字段，变体专有 UART/pinctrl 事实不得写成目标 Kit 默认值。
- 运行时 bootloader 对 `stdout-path`、bootargs 或 console 的覆盖缺少启动日志和镜像证据，保留未知项并列出解除条件。
- 完整 UART 寄存器语义、电气映射或 programmer manual 无直接来源时，只记录未知项，不用 Linux 默认值、PXA 通用知识或第三方代码补齐。
- 来源在调查期间发生变化时停止使用旧观察结论，重新登记版本并回到受影响的规划步骤；工作区中的用户修改不得覆盖。
- 本仓库没有运行时并发语义；并发、取消和恢复只适用于来源调查流程，不为文档需求虚构运行时场景。

## Scenario Sketch

- **正常聚合**: 来源可读且资源域明确时，事实进入对应主题文档，并记录 URL、版本/分支、板型、资源域和证据等级。
- **AP/RCPU 分域**: 相同 UART 编号或寄存器模型出现在不同资源域时，分别记录实例、clock/reset、pinctrl 和 IRQ，不互相补值。
- **静态 console 解析**: DTS aliases 与 `chosen` 可闭合时，记录 `serial0` 的静态指向；缺少运行时证据时不得声称最终 console 已确认。
- **SPA fallback**: 官网只能读取 SPA 壳时，GitHub 对应页和官方源码仅作交叉验证，官网正文状态不变。
- **来源冲突**: FIFO、clock/divisor 或初始化顺序不一致时，并列记录来源与适用层，不选择未经证明的统一值。
- **compatible 边界**: K3 节点复用 `spacemit,k1-uart` 等字符串时，说明绑定关系，不由命名推导 SoC 身份或寄存器全集。
- **第三方经验**: UUE/OUT2、THRE/TEMT、IRQ budget 和 errata 可形成带出处的工程经验，不成为 K3 硬件事实。
- **来源变化或中止**: 保留已验证结果，未完成项保持待办并记录恢复点；不使用过期观察补齐结论。

## Non-goals

- 不设计或实现异步 UART、copier、ring、waker、poll/select、flush/tcdrain、VFS/TTY 或 StarryOS 接口。
- 不修改 StarryOS、Linux、DTS、bootloader、Rt-Async-AMP 或 tgoskits 产品代码。
- 不提前展开 MS05 的 AIA/PLIC/timer、MS06 的 DMA/IOMMU 或 MS07 的 GMAC/PHY 数据面。
- 不把 Linux、官方 DTS、binding 或第三方驱动行为提升为 datasheet/TRM 级硬件规范。
- 不用 K1、Pico、其他 CoM260 变体或通用 8250/PXA 默认值填补 K3 CoM260 未知项。
- 不创建 scraper、验证器、运行时 Evidence 占位或可执行代码，不直接维护 SNAPSHOT、全局 tasks 或 M/D/K/R/I。

## Impact

- 计划中的产品修改为两篇新主题文档、`docs/index.md`、`docs/reference/source-coverage.md`，以及必要时修正 MS03 文档和 `docs/reference/known-gaps.md`。
- OpenSpec change 保存需求、调查设计、任务、Cycle 和验证结果；仓库继续保持纯 Markdown。
- 后续 UART 驱动计划可以引用实例、资源域、平台控制依赖、console 路径、来源冲突和未知项，不必重新混合遍历全部来源。

## Gate 1

- Status: approved
- User approval: `批准`
- Proposed scope: 两篇主题文档、必要的 MS03 勘误、覆盖登记、索引更新，以及条件性的缺口更新

| Check | Status | Evidence |
| --- | --- | --- |
| BDD gap scan | PASS | 覆盖正常路径、AP/RCPU 分域、SPA fallback、静态/运行时 console 边界、来源冲突、compatible 边界、第三方经验和调查中止 |
| Gap decisions | PASS | 用户接受 Scenario Gaps and Defaults |
| Scenario sketch | PASS | 本 proposal 给出正常、失败、冲突和边界路径 |
| OpenSpec change | PASS | `establish-k3-com260-platform-uart-baseline` 已创建 |
| Requirements and scope approval | PASS | 用户原话：`批准` |
