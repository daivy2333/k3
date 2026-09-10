## 1. Iteration 000 — DMA 与 ownership 来源基线

- [x] 1.1 [T1] 在 `docs/reference/source-coverage.md` 登记正文实际引用且已直接打开的 DMA、K3 DTS/binding/driver、GMAC/UFS 与 OpenSBI 精确官方 URL，并同步唯一 URL 数。
- [x] 1.2 [T2] 创建 `docs/dma/k3-dma-and-memory-ownership.md`，按对象整理通用 DMA、GMAC/UFS 内建 DMA、descriptor/data buffer 和 CPU/device ownership 生命周期。

## 2. Iteration 001 — 内存属性、地址转换与收尾

- [x] 2.1 [T3] 创建 `docs/dma/k3-cache-pma-address-translation.md`，分层整理 cache maintenance、barrier/fence、PMA/PBMT、IOMMU、CPU/device 地址与 AP/RP alias。
- [x] 2.2 [T4] 复核 `docs/reference/known-gaps.md` G5；按实际官方证据保持 `open` 或更新为 `partial`，仅在对象和解除条件不重复时新增缺口。
- [x] 2.3 [T5] 更新 `docs/index.md` 的 DMA 入口和状态，并按权威文件同步实际来源与缺口计数。

## Task Contracts

### T1：精确来源登记

- Requirement/Scenario: R1-S1-S3；R3-S1-S3；R4-S1-S3；R5-S1。
- Depends on: None.
- Targets: `docs/reference/source-coverage.md`。
- Current behavior: 21-DMA、GMAC、UFS 和部分 K3 DTS URL 已登记；DMA binding/driver、IOMMU、cache/PMA 等正文候选未按 MS06 的直接引用职责闭合。
- Required behavior: 只登记可直接打开且两篇正文实际引用的官方 URL；同步总数、唯一数、身份、分支和观察日期；既有 URL 只扩充职责，不重复建行。
- Preserve: 既有 62 行及其身份、观察日期和状态；官网 `partially-observed`；M01、D04-D07。
- Forbidden: 不登记未引用候选，不批量刷新既有来源，不把第三方 URL 登记为官方交叉验证，不修改 `others/`。
- Test witness: `docs/dma/` 不存在，现有覆盖表没有完整 MS06 来源职责。
- GREEN condition: 两篇正文全部直接官方 URL 各登记一次，总数与唯一数一致，无重复。
- Verification: URL 精确计数、表格计数、字段检查、正文首行反向核对、`git diff --check`。
- Stop when: 来源变化影响 D1-D5，或必须引入 M01/R08 之外的新权威来源。

### T2：DMA 与 ownership 事实包

- Requirement/Scenario: R1-S1-S3；R2-S1-S4；R3-S1-S3；R5-S1-S2。
- Depends on: T1.
- Targets: `docs/dma/k3-dma-and-memory-ownership.md`。
- Current behavior: 文件和目录不存在；DMA、GMAC、UFS 与共享内存行为散落在平台文档、G5/G6 和 R09-R12 分析中。
- Required behavior: 区分通用 DMA controller、GMAC/UFS 内建 DMA 和共享内存；对 descriptor/data buffer 分别记录准备、发布、in-flight、完成、回收、错误、timeout 和 reset 边界。
- Required changes: 建立能力/对象矩阵和 ownership 状态表；官方资料、官方源码、第三方固定 revision、推论与未知项分层；通知/doorbell/IRQ 不替代数据可见性。
- Preserve: G3/G5/G6/G7；设备边界；四级证据；少于 450 行。
- Forbidden: 不设计 DMA API，不展开 PHY/UFS 协议，不把 GMAC/UFS/共享 SRAM 的规则外推为全局硬件规范。
- Test witness: `test ! -e docs/dma/k3-dma-and-memory-ownership.md` 返回 0。
- GREEN condition: 三类对象、descriptor/data buffer 和双向 ownership 转换均可追溯，错误恢复与未知项完整。
- Verification: 首行来源、对象矩阵、状态转换、错误分支、证据标签、未知项四字段、相对链接、行数、strict validate。
- Stop when: 传输对象无法分离，或来源冲突要求改变 requirement/Acceptance。

### T3：cache、PMA/PBMT 与地址转换

- Requirement/Scenario: R3-S1-S3；R4-S1-S3；R2-S3-S4；R5-S1-S2。
- Depends on: Iteration 000 accepted.
- Targets: `docs/dma/k3-cache-pma-address-translation.md`。
- Current behavior: 文件不存在；`k3-soc-overview.md` 只声明 IOMMU 能力，R10 只保存固定 revision PMA/PBMT 与 SRAM alias 调查。
- Required behavior: 分开 cache clean/invalidate/flush、barrier/I/O fence、PMA、PBMT、页表属性、IOMMU、CPU VA/PA、device address/IOVA 和 AP/RP alias。
- Required changes: 记录 GMAC/UFS 与共享内存各自的 cache/地址路径；OpenSBI PMA 调用阶段和 per-hart 边界；IOMMU 节点/绑定不足时保留未知；明确 fence 不等于 cache maintenance。
- Preserve: G5/G7；固定 revisions；无真板结论；少于 450 行。
- Forbidden: 不声称 X100、PMA/PBMT 或 IOMMU 硬件效果已由本项目验证，不把共享内存 PMA 规则外推设备 DMA。
- Test witness: `test ! -e docs/dma/k3-cache-pma-address-translation.md` 返回 0。
- GREEN condition: 各内存属性机制和地址空间边界可独立检索，冲突、适用对象和解除条件完整。
- Verification: 首行来源、机制矩阵、地址关系、revision、四字段未知项、链接、行数、strict validate。
- Stop when: 官方 IOMMU/DMA 模型与当前调查实质冲突，或无法分离平台属性和设备策略。

### T4：G5 与新缺口不重复

- Requirement/Scenario: R3-S3；R4-S3；R5-S2。
- Depends on: T2-T3.
- Targets: `docs/reference/known-gaps.md`。
- Current behavior: G5 为 `open`，只记录概述级缺失；G6 单独覆盖 GMAC programmer reference。
- Required behavior: 按新增官方证据更新 G5 当前证据、未解字段和状态；新缺口须有独立对象及四字段，且不重复 G5/G6。
- Preserve: G1-G10 其他状态和事实，无证据不改；已解除历史不删除。
- Forbidden: 不凭第三方代码关闭 G5，不把每个设备的相同 coherency 问题拆成重复 G 项。
- Test witness: G5 未列本 change 新来源与对象边界。
- GREEN condition: G5 状态与两篇正文一致；新增项不重复；汇总状态和计数同步。
- Verification: G 编号、状态、四字段、正文交叉引用、语义比较、`git diff --check`。
- Stop when: 新证据改变 milestone 范围或需要长期模型/决策变更。

### T5：入口与计数一致

- Requirement/Scenario: R5-S3。
- Depends on: T2-T4.
- Targets: `docs/index.md`。
- Current behavior: DMA 为待聚合且无正文链接；来源和缺口计数反映 MS05 基线。
- Required behavior: 链接两篇实际正文并标为已聚合；来源和缺口计数与覆盖表/known-gaps 一致。
- Preserve: 其他主题职责、链接和状态。
- Forbidden: 不复制技术事实，不改变 MS07-MS11 状态，不同步 SNAPSHOT/tasks。
- Test witness: 两个 DMA 链接不存在，主题表仍标待聚合。
- GREEN condition: 链接可解析，DMA 状态和计数一致，无关行不变。
- Verification: 链接解析、计数对照、scoped diff、`git diff --check`。
- Stop when: T2-T4 未 GREEN。

## Iteration Plan

### Iteration 000：DMA 与 ownership 来源基线

- Tasks: T1-T2.
- Depends on: MS03-MS05 已归档。
- Stable baseline: 后续内存属性文档可引用稳定的传输对象、descriptor/data buffer 与 ownership 状态术语。
- Verification boundary: 精确来源唯一；三类传输对象和 CPU/device ownership 生命周期按证据层可追溯。
- Diagnostic boundary: 来源身份、DMA 对象、descriptor/data buffer、发布/完成/回收。
- Non-goals: cache/PMA/PBMT/IOMMU 机制正文、G5 更新和 index 收尾。
- Balance audit: T1 与 T2 强依赖且共同形成可独立引用的 ownership 基线；不含内存属性故障域，工作量适中。

### Iteration 001：内存属性、地址转换与收尾

- Tasks: T3-T5.
- Depends on: Iteration 000 accepted.
- Stable baseline: MS06 的 DMA、ownership、coherency 和地址转换边界可供 MS07-MS09 引用。
- Verification boundary: cache/barrier/PMA/PBMT/IOMMU 分层完整，G5、来源、入口与计数一致。
- Diagnostic boundary: cache/ordering、平台属性、地址转换、缺口和导航。
- Non-goals: 驱动实现、真板验证、下游 GMAC/UFS/AMP 正文。
- Balance audit: T3 形成第二个独立技术结果；T4-T5 与其共享最终一致性边界，不足以单独成为 Iteration，合并合理。

## Requirements Traceability Matrix

| Requirement | Scenario | Design | Task | Iteration | Code Surface | Test Witness | Simplification | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| R1 | 通用/设备内建/共享内存分层 | D1,D2 | T1,T2 | 000 | coverage; ownership doc | `docs/dma/` 不存在 | None | Covered |
| R2 | 交付/完成/通知/错误恢复 | D3 | T2,T3 | 000,001 | ownership; attribute docs | 状态表不存在 | None | Covered |
| R3 | cache maintenance 与 fence | D3,D4 | T2,T3,T4 | 000,001 | ownership; attribute; gaps | G5 概述级 | None | Covered |
| R4 | PMA/PBMT/IOMMU/地址 alias | D4 | T3,T4 | 001 | attribute doc; gaps | 专题文档不存在 | None | Covered |
| R5 | 来源、G5、入口 | D2,D5 | T1,T4,T5 | 000,001 | reference; index | URL/状态/链接检查 | None | Covered |

## Plan Completeness Review

- TBD/TODO: None.
- Requirement simplification: None.
- 五项 requirement 的全部 scenarios 均映射到 design、task、文件和验证。
- 两个 Iteration 各形成稳定基线，依赖有序；后续 Iteration 尚未创建目录或 Cycle。
- Persisted Evidence: `none`；所有验证均可从仓库 Markdown 与只读来源低成本重跑。
