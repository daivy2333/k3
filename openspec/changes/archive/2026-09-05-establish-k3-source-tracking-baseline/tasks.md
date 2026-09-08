## 1. Iteration 000 — 首次来源刷新

- [x] 1.1 [T1] 修订 `docs/reference/source-refresh.md`：首次 refresh 只比较已持久化字段，新字段建立当前 baseline，SPA 壳归为 `unreachable`。
- [x] 1.2 [T2] 重新复核 R01 K3 文档入口，替换无效结论并校正对应覆盖行。
- [x] 1.3 [T3] 重新复核 R07 `source.md`；官网不可读时用对应官方 GitHub 文档建立交叉验证级 SDK 来源 baseline。
- [x] 1.4 [T4] 重新复核 R07 `bl-v1.0.y.md`；官网不可读时用对应官方 GitHub release notes 建立交叉验证级 release baseline。
- [x] 1.5 [T5] 直接复核 R08 `docs-buildroot/tree/main/zh/k3_buildroot`，记录 supporting 结果并校正覆盖行。
- [x] 1.6 [T6] 直接复核 R08 `docs-chip/tree/main/zh/key_stone/k3`，记录 supporting 结果并校正覆盖行。
- [x] 1.7 [T7] 直接复核 R08 `docs-product/tree/main/zh/k3_com260`，记录 supporting 结果并校正覆盖行。
- [x] 1.8 [T8] 直接复核 R08 `linux-6.18/tree/k3-br-v1.0.y`，记录 supporting 结果并校正覆盖行。
- [x] 1.9 [T9] 仅在 `removed` 或符合既有长期不可访问条件且无替代来源时更新 `docs/reference/known-gaps.md`；否则以 `SKIPPED` 和实际原因完成条件分支。

## Task Contracts

### T1：首次 baseline 比较规则可执行

- Requirement/Scenario: R1 / 首次刷新只比较已持久化字段、SPA 壳不能提供正文、已持久化字段仍不足以比较；R5 / 首次真实 refresh 完成。
- Depends on: None.
- Targets: `docs/reference/source-refresh.md`。
- Current behavior: 初始实现要求任一职责字段缺少上次值就停止，导致首次 refresh 无法为新观察字段建立 baseline。
- Required behavior: 指南只用 2026-09-02 已持久化字段判定结果；导航、路径、分支和 release 内容作为当前 baseline；R01/R07 只有 SPA 壳时归为 `unreachable`。
- Required changes: 改写比较字段、证据顺序和停止条件；保持五种结果、长期状态与刷新结果分离、中断恢复规则不变。
- Preserve: M01-M04、D01-D08；R08 只作 supporting；首行来源和现有章节职责。
- Forbidden: 不增加第六种结果，不保存正文快照，不引入脚本、Hash、manifest、run ID 或 Evidence 要求。
- Test witness: 当前指南第 112–124 行会在新字段没有上次值时停止，已由 Cycle 000 Review 证明会使 T2 起的刷新结论失效。
- GREEN condition: 指南可直接回答哪些字段参与首次比较、哪些字段只建立 baseline、SPA 壳如何分类以及何时真正停止。
- Verification: 检查新增字段表和停止条件；`git diff --check` 与 OpenSpec strict validate 均通过。
- Stop when: 需要增加第六种结果、改变 M01 权威边界或扩大已批准 7 URL 范围。

### T2：R01 权威入口获得独立刷新结果

- Requirement/Scenario: R1 / 选定来源完成复核、集合改变、首次持久字段比较、SPA 壳；R2 / 五种结果；R3 / 权威边界；R4 / 中断恢复。
- Depends on: T1.
- Targets: `docs/reference/source-coverage.md` 中 R01 K3 文档入口行。
- Current behavior: 行 28 已被无效的 `unchanged` 结论改为 `2026-09-05`；该结论缺少导航上次值且违反当时停止规则。
- Required behavior: 重新访问原 URL，只用持久化 URL、来源身份和访问状态分类；导航观察只建立当前 baseline，并记录覆盖变化、已检查清单和 `next: R07 source.md`。
- Required changes: 只按结果矩阵更新该行；moved 时保留旧行并新增已确认新行；结果本身只写 Act Response。
- Preserve: 未检查 37 行、旧 URL、M01 权威身份和技术正文。
- Forbidden: 不把 R08 可访问性当作 R01 可访问，不遍历或聚合全部 K3 子树。
- Test witness: Cycle 000 Review 已证明当前 `2026-09-05` 日期来自无效结果；新 Act Response 尚无替代该结论的有效证据。
- GREEN condition: R01 有独立、证据支持的结果；覆盖行变化与结果一致；失败或中断时 `next` 明确。
- Verification: 检查该行 diff、旧 URL 保留、URL 唯一性和 Act Response 结果。
- Stop when: 登记集合变化、持久字段也无法确认且不符合已有结果分支，或处理必须扩展到技术正文。

### T3：R07 source 页面获得 SDK 来源结果

- Requirement/Scenario: R1 / 选定来源完成复核、SPA 壳；R2 / 五种结果；R3 / R07 SDK 线索、supporting fallback；R4 / 中断恢复。
- Depends on: T2.
- Targets: `docs/reference/source-coverage.md` 中 R07 `source.md` 行。
- Current behavior: 行 60 已基于 SPA 壳结果改为 `2026-09-05`，但 Cycle 000 没有建立 SDK baseline。
- Required behavior: 复核官网并如实分类；正文不可读时，从对应官方 GitHub `docs-buildroot/blob/main/zh/k3_buildroot/source.md` 记录交叉验证级 SDK 信息和 `next: R07 bl-v1.0.y.md`。
- Required changes: 只按结果矩阵修改该行；不把观察日期写成源端修订。
- Preserve: 官网事实等级、其他覆盖行和 R08 supporting 边界。
- Forbidden: 不从 URL、分支名或通用 SDK 知识推断版本，不把 GitHub 交叉验证改写为官网事实。
- Test witness: Cycle 000 Review 已证明当前行日期没有对应 SDK baseline；新 Act Response 尚无官方 GitHub 正文证据。
- GREEN condition: 该 URL 有独立结果；官网不可读时如实记录 `unreachable`，并从对应官方 GitHub 正文建立明确标级的 SDK baseline。
- Verification: 检查页面观察、行级 diff、Act Response 和未检查行。
- Stop when: 对应 GitHub 文档也不可读或身份不匹配，或需要聚合技术正文。

### T4：R07 release notes 获得 release baseline

- Requirement/Scenario: R1 / 选定来源完成复核、SPA 壳；R2 / 五种结果；R3 / R07 SDK 线索、supporting fallback；R4 / 中断恢复。
- Depends on: T3.
- Targets: `docs/reference/source-coverage.md` 中 R07 `bl-v1.0.y.md` 行。
- Current behavior: 行 61 已基于 SPA 壳结果改为 `2026-09-05`，但 Cycle 000 没有建立 release baseline。
- Required behavior: 复核官网并如实分类；正文不可读时，从对应官方 GitHub `docs-buildroot/blob/main/zh/k3_buildroot/release_notes/bl-v1.0.y.md` 记录交叉验证级 release baseline 和 `next: R08 docs-buildroot`。
- Required changes: 只按结果矩阵修改该行；与 T3 共同形成 R07 SDK baseline。
- Preserve: 源端修订与观察日期分离、其他覆盖行和官网权威性。
- Forbidden: 不把路径中的 `bl-v1.0.y` 自动当作当前已验证版本，不把 GitHub 交叉验证改写为官网事实。
- Test witness: Cycle 000 Review 已证明当前行日期没有对应 release baseline；新 Act Response 尚无官方 GitHub 正文证据。
- GREEN condition: 该 URL 有独立结果；T3/T4 共同形成标明官网事实或交叉验证等级的可复用 SDK baseline。
- Verification: 检查页面观察、行级 diff、T3/T4 合并结论和 Act Response。
- Stop when: T3、T4 及其对应官方 GitHub 文档都无法提供 SDK baseline，或来源身份不匹配。

### T5：R08 docs-buildroot 保持 supporting 身份

- Requirement/Scenario: R1 / 选定来源完成复核；R2 / 五种结果；R3 / R08 一致或不一致；R4 / 中断恢复。
- Depends on: T4.
- Targets: `docs/reference/source-coverage.md` 中 `https://github.com/spacemit-com/docs-buildroot` 行。
- Current behavior: 行 62 为 `unknown | 2026-09-02 | observed`，职责和聚合状态均为 supporting。
- Required behavior: 复核仓库公开性和身份，直接检查 `main/zh/k3_buildroot`，记录该路径作为当前 baseline、与 R07 的关系以及 `next: R08 docs-chip`。
- Required changes: 只按结果矩阵修改该行；差异写入 Act Response 和待解除条件。
- Preserve: supporting-source、workflow-support 和 supporting 状态。
- Forbidden: 不把仓库提交内容提升为官网事实，不记录 commit Hash 作为验收身份。
- Test witness: Cycle 000 Act 声称未见 K3 路径，但 Review 已直接定位该路径；新 Act Response 尚未纠正观察。
- GREEN condition: 仓库有独立结果，角色未被提升，R07 差异已显式记录。
- Verification: 检查仓库观察、行级 diff、Act Response 和权威字段。
- Stop when: 仓库身份变化要求重定向，或与官网差异影响已批准需求语义。

### T6：R08 docs-chip 保持 supporting 身份

- Requirement/Scenario: R1 / 选定来源完成复核；R2 / 五种结果；R3 / R08 一致或不一致；R4 / 中断恢复。
- Depends on: T5.
- Targets: `docs/reference/source-coverage.md` 中 `https://github.com/spacemit-com/docs-chip` 行。
- Current behavior: 行 63 为 `unknown | 2026-09-02 | observed`，职责和聚合状态均为 supporting。
- Required behavior: 复核仓库公开性和身份，直接检查 `main/zh/key_stone/k3`，记录该路径作为当前 baseline 和 `next: R08 docs-product`。
- Required changes: 只按结果矩阵修改该行；与 R01 的差异显式记录。
- Preserve: supporting-source、workflow-support 和 supporting 状态。
- Forbidden: 不从芯片文档仓库聚合 K3 技术事实，不记录 commit Hash 作为验收身份。
- Test witness: Cycle 000 Act 声称未见 K3 路径，但 Review 已直接定位该路径；新 Act Response 尚未纠正观察。
- GREEN condition: 仓库有独立结果且 M01 权威边界保持不变。
- Verification: 检查仓库观察、行级 diff、Act Response 和权威字段。
- Stop when: 仓库身份变化或差异要求修改技术正文。

### T7：R08 docs-product 保持 supporting 身份

- Requirement/Scenario: R1 / 选定来源完成复核；R2 / 五种结果；R3 / R08 一致或不一致；R4 / 中断恢复。
- Depends on: T6.
- Targets: `docs/reference/source-coverage.md` 中 `https://github.com/spacemit-com/docs-product` 行。
- Current behavior: 行 64 为 `unknown | 2026-09-02 | observed`，职责和聚合状态均为 supporting。
- Required behavior: 复核仓库公开性和身份，直接检查 `main/zh/k3_com260`，记录该路径作为当前 baseline 和 `next: R08 linux-6.18`。
- Required changes: 只按结果矩阵修改该行；与官网产品资料的差异显式记录。
- Preserve: supporting-source、workflow-support 和 supporting 状态。
- Forbidden: 不聚合 CoM260 板级正文，不记录 commit Hash 作为验收身份。
- Test witness: Cycle 000 Act 声称未见 K3/CoM260 路径，但 Review 已直接定位该路径；新 Act Response 尚未纠正观察。
- GREEN condition: 仓库有独立结果且没有越过 MS02 进入 MS03。
- Verification: 检查仓库观察、行级 diff、Act Response 和范围边界。
- Stop when: 仓库身份变化或发现必须进入板级技术聚合的变化。

### T8：R08 linux-6.18 保持 supporting 身份

- Requirement/Scenario: R1 / 选定来源完成复核；R2 / 五种结果；R3 / R08 一致或不一致；R4 / 中断恢复。
- Depends on: T7.
- Targets: `docs/reference/source-coverage.md` 中 `https://github.com/spacemit-com/linux-6.18` 行。
- Current behavior: 行 65 为 `unknown | 2026-09-02 | observed`，职责和聚合状态均为 supporting。
- Required behavior: 复核仓库公开性和身份，直接检查 `k3-br-v1.0.y` 分支，记录该分支作为当前 baseline 并在完成时写 `next: none`。
- Required changes: 只按结果矩阵修改该行；与官网或 R07 的差异显式记录。
- Preserve: supporting-source、workflow-support 和 supporting 状态。
- Forbidden: 不用 Linux 行为替代 K3 硬件规范，不记录 commit Hash 作为验收身份。
- Test witness: Cycle 000 Act 没有给出 K3 分支证据，但 Review 已直接定位该分支；新 Act Response 尚未纠正观察。
- GREEN condition: 仓库有独立结果，7 URL 已检查清单完整，`next: none`。
- Verification: 检查仓库观察、行级 diff、七项结果、顺序和范围边界。
- Stop when: 仓库身份变化或差异要求修改技术正文。

### T9：符合门槛的来源缺口得到登记

- Requirement/Scenario: R2 / 来源永久下架、来源暂时不可访问；R5 / 越界修改。
- Depends on: T2-T8.
- Targets: `docs/reference/known-gaps.md`。
- Current behavior: 现有 G1-G6 不记录本 change 尚未发生的 removed 或长期 unreachable 结果。
- Required behavior: 只有官方证据支持 removed，或符合既有长期不可访问条件且无替代来源时，新增含当前证据、禁止推断、解除条件和影响主题的缺口；否则写 `SKIPPED: no eligible source gap`。
- Required changes: 满足门槛时精准追加一个或多个 G 条目并更新汇总；不满足时不修改文件。
- Preserve: G1-G6、历史 URL、缺口状态枚举和 R01/R08 权威边界。
- Forbidden: 不因单次 timeout、cache miss 或工具 DNS 失败创建缺口，不删除旧条目。
- Test witness: 触发时，目标 URL 尚无对应缺口的检查为 RED；未触发时按条件记录 SKIPPED，不建立虚假测试。
- GREEN condition: 每个符合门槛的结果都有唯一缺口，或全部不符合门槛且文件无修改。
- Verification: 对照七项结果检查 G 条目和汇总；运行 Markdown 与 diff 检查。
- Stop when: 缺口需要技术调查或改变 G1-G6 的既有事实。

## Iteration Plan

### Iteration 000: 首次来源刷新

- Tasks: T1, T2, T3, T4, T5, T6, T7, T8, T9
- Depends on: None
- Stable baseline: 维护者拥有一次覆盖 R01、R07、R08 的真实 refresh 结果、标明官网事实或交叉验证等级的可复用 SDK baseline、明确 supporting 边界和可执行的首次比较规则。
- Verification boundary: 7 个 URL 各有独立结果；实际覆盖行变化与结果一致；未检查行保持不变；T3/T4 形成有证据等级的 SDK baseline；完整交付 diff 排除 `.omo/`；URL 唯一、Markdown diff 和 OpenSpec strict validate 通过。
- Diagnostic boundary: T1 隔离比较规则；T2-T8 各隔离一个来源访问或分类问题；T9 隔离 removed/长期 unreachable 的缺口登记。
- Non-goals: 不刷新其他 31 个 URL，不聚合技术正文，不同步全局状态，不创建自动化或持久化 Evidence。
- Balance audit: 九个任务共享一次 refresh 的顺序、结果表和验收边界，任何来源子集都不能单独完成 MS02；每个 URL 又由独立 task 隔离访问和分类失败。拆分 Iteration 会产生不能作为后续 milestone 依赖的部分 baseline，因此保持单 Iteration。
