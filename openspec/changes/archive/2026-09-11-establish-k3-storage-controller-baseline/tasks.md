## Tasks

- [x] T1 — 激活并补齐存储来源覆盖：更新 `docs/reference/source-coverage.md` 中 QSPI、SDHC、SPI、UFS 官网入口的当前职责，保留本次不可达边界；仅为已经实际直接读取的官方 GitHub 页面、K3 DTS/binding/driver 增加或复用 supporting rows，并保持 URL 唯一。
- [x] T2 — 创建 `docs/storage/k3-qspi-spi-sdhci.md`：按 QSPI、普通 SPI、SD/eMMC/SDHCI 分层记录 SoC 能力、CoM260 可达性、DTS 资源、启动关系、数据路径、DMA/IRQ/tuning、错误边界和未知项；明确 QSPI 与通用 SPI、SD 与 eMMC、静态节点与运行时路径的差异。
- [x] T3 — 创建 `docs/storage/k3-ufs.md`：记录 K3/CoM260 UFS 能力与资源、MPHY/UniPro/link startup、UTRD/UTMRD/UCD/PRDT、SCSI/LUN、DMA/cache、轮询完成、timeout/fatal recovery 和同步访问边界；按证据等级区分官方材料与固定第三方实现。
- [x] T4 — 更新 `docs/reference/known-gaps.md`：把两篇存储正文的未知项映射到既有 G5/G7，并在确有独立存储缺口时新增递增 G 条目；同步状态汇总、来源对应和影响主题，不借本 change 裁决 UFS 容量或默认 DTS。
- [x] T5 — 更新 `docs/reference/terminology.md`：增加正文实际使用且现有表中缺失的存储术语，至少核对 QSPI、SDHCI、eMMC、UFS、MPHY、UniPro、UTP、UPIU、UTRD/UCD/PRDT；避免同义重复并说明设备适用范围。
- [x] T6 — 更新 `docs/index.md`：加入两篇存储正文入口，更新 `docs/storage/` 职责、来源数量和缺口计数，使其与覆盖表和 known-gaps 一致。

## Iteration Plan

### Iteration 000: 来源与 QSPI/SPI/SDHCI 基线

- Tasks: T1, T2
- Depends on: None
- Stable baseline: 存储来源具有当前职责，QSPI、普通 SPI、SD/eMMC/SDHCI 的资源、启动和数据路径有独立正文及未知项。
- Verification boundary: 四个官网入口不再 deferred 且准确记录可达状态；新增 supporting rows（若有）均已直接读取且 URL 唯一；`k3-qspi-spi-sdhci.md` 覆盖 requirement 1-3、6 的相关场景，首行来源和相对链接有效。
- Diagnostic boundary: 来源访问/登记、QSPI 与普通 SPI 区分、SD/eMMC 板级映射、SDHCI 静态资源、第三方 core 证据等级。
- Non-goals: 不写 UFS 协议/恢复正文，不更新全局导航和最终缺口汇总。
- Balance audit: T1 为 T2 和后续 UFS 正文提供统一来源责任；T2 形成非 UFS 存储的完整稳定成果。拆开会留下无正文来源或无登记正文，合并 UFS 则跨越独立协议和恢复故障域，当前粒度合适。

### Iteration 001: UFS 协议、数据路径与恢复基线

- Tasks: T3
- Depends on: Iteration 000
- Stable baseline: UFS 从板级资源、MPHY/UniPro 到 UTP/SCSI、DMA/cache、完成和 recovery 的路径可独立检索，并保留同步轮询和第三方证据边界。
- Verification boundary: `k3-ufs.md` 覆盖 requirement 1-6 的 UFS 场景，明确 IRQ 解析但不注册、轮询完成、单次 recovery retry 与 fatal latch；相对链接和来源元数据有效。
- Diagnostic boundary: UFS platform resource、link initialization、descriptor/doorbell、LUN、DMA/cache、timeout/fatal/recovery。
- Non-goals: 不修改 QSPI/SPI/SDHCI 正文，不执行 UFS I/O 或真板验证，不接入索引最终计数。
- Balance audit: UFS 单篇正文内部各层构成一条端到端路径，拆成更小 Iteration 会产生不能独立解释的半成品；与 Iteration 000 合并会混合不同故障域并使 Review 过重。

### Iteration 002: 导航、术语与缺口收敛

- Tasks: T4, T5, T6
- Depends on: Iteration 000, Iteration 001
- Stable baseline: 所有存储正文能从总索引进入，术语、来源数量、缺口状态和影响主题一致，MS09 change 可进入最终 Review。
- Verification boundary: 两篇正文入口有效；术语无重复；known-gaps 正文与汇总一致；索引计数与覆盖表/缺口表相等；change 严格校验通过。
- Diagnostic boundary: `known-gaps.md`、`terminology.md`、`index.md` 三个导航与汇总表面。
- Non-goals: 不新增技术结论，不修复本 change 之外的 OpenSpec spec 或 SNAPSHOT/tasks 状态。
- Balance audit: 三个任务共享两篇正文完成后的收敛输入，单独分轮均不足以形成稳定成果；合并到前两轮会迫使未完成正文提前决定计数和缺口，因此保留独立收尾 Iteration。

## Requirements Traceability Matrix

| Requirement | Scenario | Design | Task | Iteration | Code Surface | Test Witness | Simplification | Status |
|---|---|---|---|---|---|---|---|---|
| R1 资源与板级可达性 | 静态资源；连接未知 | D2, D4 | T1, T2, T3 | 000, 001 | `source-coverage.md`; 两篇 storage 正文 | deferred rows；目标文件缺失；资源/证据断言 | None | Covered |
| R2 启动介质关系 | 已确认路径；目标不唯一 | D3 | T2, T3 | 000, 001 | 两篇 storage 正文；boot 相对链接 | 启动/候选/非默认断言与链接检查 | None | Covered |
| R3 数据路径与所有权 | 正常路径；仅静态能力 | D3, D4 | T2, T3 | 000, 001 | 两篇 storage 正文；DMA 相对链接 | ownership/DMA/IRQ 能力边界断言 | None | Covered |
| R4 UFS 独立边界 | 分层路径；硬件细节缺失 | D1, D4 | T3 | 001 | `k3-ufs.md` | 目标文件缺失；MPHY/UniPro/UTP/SCSI 章节断言 | None | Covered |
| R5 错误与恢复 | 已审计实现；完整性未知 | D5 | T2, T3 | 000, 001 | 两篇 storage 正文 | timeout/error/recovery/unknown 断言 | None | Covered |
| R6 来源冲突与不可达 | 可交叉验证；冲突/不可达 | D2, D4 | T1-T4 | 000-002 | coverage、正文、known-gaps | URL 唯一、证据等级、四字段缺口检查 | None | Covered |
| R7 导航一致性 | 完成聚合；与旧基线冲突 | D3, D6 | T4-T6 | 002 | gaps、terminology、index | 链接、计数、术语、严格校验 | None | Covered |

## Task Status

- Completed Iterations: 000 (`accepted`)
- Current Iteration: 001
- Current Cycle: `iterations/001-ufs-protocol-data-recovery/000-initial.md`
- Deferred Iterations: 002
- Persisted Evidence: none
