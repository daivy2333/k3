# Iteration 000 / Cycle 001: ownership 事实与链接修复

## Plan Context

- Status: ready
- Iteration: 000-dma-ownership-baseline
- Cycle: 001-rework
- Cycle Type: rework
- Parent cycle: `000-initial.md`

**Iteration Scope**

- Change tasks: T1-T2
- Depends on: `000-initial.md` 的实际实施结果
- Stable baseline: Iteration 001 可引用来源可追溯、设备语义正确且链接有效的 DMA/ownership 文档
- Verification boundary: UFS、GMAC、共享内存和 PMA 的描述与固定 revision 源码一致；全部本地链接可解析；T1-T2 状态与实际完成情况一致
- Diagnostic boundary: 来源支持、descriptor/doorbell/OCS、GMAC OWN、PMA 作用域、本地链接和 change task 状态
- Deferred tasks: T3-T5

**Cycle Scope**

- Trigger: rework-required
- Acceptance gaps: A2-A5；对象分层存在，但 UFS/PMA/GMAC 事实错误、前向链接失效且 task 状态未同步
- Repair items: T1-R1、T2-R1、T2-R2、T2-R3
- Inherited scope: proposal R1-R5、design D1-D5、原 T1-T2、M01-M04、D01-D07
- Excluded scope: T3-T5、创建 Iteration 001 正文、修改 known-gaps/index、驱动实现、全局状态和项目记忆

**Objective**

修正 `k3-dma-and-memory-ownership.md` 中与可读固定 revision 源码冲突的 UFS、GMAC、共享内存和 PMA 事实，删除或降级不存在的本地链接，复核来源支持，并在全部验证通过后同步 T1-T2 checkbox。

**Background**

Act 在 `000-initial.md` 中越权把 Plan Review 写成 `accepted`，但独立 Review 发现多个阻断 Acceptance 的问题。终态区域按规则不可覆写，因此保留父 Cycle 原文作为历史现场；本 rework Cycle 取代其未经独立审计的接受结论，不改变 Iteration Map。

**Current Baseline**

- `docs/dma/k3-dma-and-memory-ownership.md` 已创建，154 行；`docs/reference/source-coverage.md` 有 65 个唯一表内 URL。
- 原 T1-T2 checkbox 仍为未完成，`openspec list` 显示 0/5 tasks。
- 父 Cycle `Act Response: reported`，但 Plan Review 由 Act 填为 `accepted`；这不是独立 Plan Review。
- Persisted Evidence 仍为 `none`；所有问题可从仓库内容和固定 revision 源码重跑。

**Current-State Evidence**

- UFS `desc.rs` 的 UTRD 只有 DW0-DW3、UCD 地址、response offset/length 和 PRDT offset/length；没有 `OWN` 字段。`transfer.rs` 以 doorbell bit 清零作为完成条件，随后执行 `dma_rmb()`、`complete_for_cpu()` 并读取 OCS/response。
- 正文第 29、61-63、87-95、103 行把 UFS 写成“置/清 UTRD OWN”或用 OWN 统一描述完成，和上述源码冲突。
- GMAC `core.rs::reclaim_tx/reclaim_rx` 在 DMA 已清 OWN 后读取 error，随后调用 `desc::clear` 清整个 descriptor；正文第 101-103 行写成 error 时 CPU 主动清 OWN 或重新置 OWN，扩大了可证行为。
- RP `chip-k3-rt24/src/lib.rs::K3Rt24::init` 明确记录 rcpu1 不能通过 custom CSR 控制 cache/PMA；正文第 70 行称 AP/RP 双方分别维护 PMA 窗口，第 72 行把 PBMT、fence 和 cache maintenance 组成必需链，和可读代码及 R10 边界冲突。
- `docs/dma/k3-cache-pma-address-translation.md` 尚不存在，但正文第 6、73、96、113、127、134、145 行链接该文件；`test -f` 返回 1。父 Act Response 的“11 个相对链接全部 OK”不可复现。
- `docs/reference/source-coverage.md` 表内 65 个 URL 均唯一；OpenSpec strict validation 与 diff check 通过，但它们不能替代事实和链接验收。

**Relevant Code**

- `docs/dma/k3-dma-and-memory-ownership.md`：本 Cycle 的主要修复目标。
- `docs/reference/source-coverage.md`：T1 来源支持和唯一性复核；只在实际来源职责错误时精准修正。
- `openspec/changes/establish-k3-com260-dma-memory-ownership-baseline/tasks.md`：全部修复和验证通过后只勾选 T1-T2。
- `others/Rt-Async-AMP/tgoskits/drivers/ax-driver/src/block/k3_ufs/{desc,transfer}.rs`：UFS doorbell/OCS 与 DMA 同步依据。
- `others/Rt-Async-AMP/tgoskits/drivers/ax-driver/src/net/k3_gmac/{desc,core}.rs`：GMAC OWN、error 与回收依据。
- `others/Rt-Async-AMP/modules/chip-k3-rt24/src/lib.rs`、`modules/ov-shm/src/shm.rs`、`opensbi-k3/.../spacemit_k3.c`：RP cache/PMA、fence 与 AP OpenSBI PMA 作用域依据。

**Critical Path**

1. 重新核对正文实际依赖的官方 URL；无法直接打开时保留既有观察状态，不新增“本轮确认”事实。
2. 用 doorbell-clear、`dma_rmb`、`complete_for_cpu`、OCS/response 重写 UFS ownership 路径，删除全部 UTRD OWN 断言。
3. 把 GMAC error 回收收紧为 DMA 清 OWN 后 CPU 读取 error 并清 descriptor，不泛化 reset/重新置 OWN。
4. 把 PMA 设置限定为 AP OpenSBI X100 hart 的固定 revision 路径；RP 只保留不能通过 custom CSR 控制 cache/PMA 与 fence 边界。
5. 将未来 Iteration 001 文件引用改为无链接文本，或仅在目标实际存在后链接；校验全部本地 Markdown 链接。
6. 全部 GREEN 后勾选 T1-T2，保持 T3-T5 未完成，并执行完整 diff Review。

**Implementation Guidance**

保留三类传输对象和五阶段抽象，但每个设备的 completion token 必须使用各自实际机制。抽象段落涉及 UFS 时写 doorbell bit/OCS，涉及 GMAC 时写 OWN；无法同时成立的句子拆开。证据标签按直接支持层级降级，不把 roadmap 决策写成官方事实。

**Behavioral Change**

修复后，文档不再声称 UFS 存在 OWN 位，不再声称 RP 能设置 PMA，也不再承诺尚未创建文件的链接可达；GMAC error/recovery 与源码一致。change task 状态将反映 Iteration 000 的实际完成结果。

**Change Surface**

| Repair | Requirement/Scenario | File/Symbol | Current Responsibility | Planned Change |
| --- | --- | --- | --- | --- |
| T1-R1 | R1/R3/R4/R5 来源场景 | `docs/reference/source-coverage.md` | 65 URL 来源表 | 复核三条新增 URL 与备注，只修不受来源支持的职责 |
| T2-R1 | R1/R2/R3 | `docs/dma/k3-dma-and-memory-ownership.md` §3、§4、§6、§7 | GMAC/UFS ownership | 修正 UFS completion 和 GMAC error/recovery |
| T2-R2 | R1/R2/R3/R4 | 同文件 §1、§5、§6、§8 | 共享内存与 PMA 边界 | 修正 RP/PMA、fence/cache 和证据标签 |
| T2-R3 | R5 | 同文件全部相对链接；`tasks.md` T1-T2 | 导航与状态 | 消除失效链接，GREEN 后勾选 T1-T2 |

**Task Contracts**

### T1-R1：来源职责复核

- Requirement/Scenario: R1-S1-S3；R3-S1-S3；R4-S1-S3；R5-S1。
- Depends on: None.
- Targets: `docs/reference/source-coverage.md`、正文首行和来源导航。
- Current behavior: 65 个表内 URL 唯一，但三条新增 docs-buildroot 行及正文声称“直接打开”的具体内容尚需在本 rework 重新核对。
- Required behavior: 每个精确来源只承载可直接确认的职责；不可访问时不得制造新观察结论，保留既有状态并把无法支持的正文陈述降为未知。
- Required changes: 逐项核对 21-DMA、09-GMAC、ufs 和正文实际引用 DTS；只修不受支持的备注或正文。
- Preserve: URL 唯一键、既有观察日期、M01/D04-D07、65 行基线（除非发现实际重复或无效新增）。
- Forbidden: 不批量 refresh，不以第三方源码替代官方 URL，不修改无关来源行。
- Test witness: 本 Review 环境无法解析 raw.githubusercontent.com；父 Act 的直接打开结果不能由本次网络复核重现。
- GREEN condition: Act 记录每个直接来源的可读结果或明确不可访问边界；正文陈述不超过可证内容；表内 URL 无重复。
- Verification: URL 表行计数、唯一性、正文首行一对一和逐来源决定性输出。
- Stop when: 来源变化要求修改 requirement、设计或 M01 边界。

### T2-R1：设备 ownership 事实修正

- Requirement/Scenario: R1-S2；R2-S1-S4；R3-S1-S2。
- Depends on: T1-R1.
- Targets: `docs/dma/k3-dma-and-memory-ownership.md` §1、§3、§4、§6、§7。
- Current behavior: UFS 被错误描述为 UTRD OWN 置/清；GMAC error 被描述为 CPU 主动清或重新置 OWN。
- Required behavior: UFS 使用 prepare/doorbell/doorbell-clear/`dma_rmb`/`complete_for_cpu`/OCS-response；GMAC 使用 DMA 清 OWN、CPU invalidate、error 检查和 `desc::clear`。
- Required changes: 删除所有 UFS OWN 断言；拆分通用抽象中的设备专有完成条件；收紧 error/timeout/reset 推论，不把未知恢复策略写成事实。
- Preserve: 三类对象、descriptor/data buffer 分离、固定 revisions、无真板结论。
- Forbidden: 不虚构 UFS OWN、IRQ completion 或通用 reset 序列，不把 GMAC规则外推 UFS。
- Test witness: `rg -n 'UTRD.*OWN|OWN.*UTRD|设备清 UTRD|置 UTRD'` 当前命中正文第 61-63 行附近，而 UFS源码无对应字段。
- GREEN condition: 上述禁止模式零命中；UFS 和 GMAC 路径分别与固定 revision 源码一致。
- Verification: 源码符号/字段对照、正文模式检查、状态链人工审查、strict validate。
- Stop when: 固定 revision 变化或源码无法支持既有 Acceptance。

### T2-R2：共享内存与 PMA 作用域修正

- Requirement/Scenario: R1-S3；R2-S3；R3-S2-S3；R4-S1-S2。
- Depends on: T1-R1.
- Targets: `docs/dma/k3-dma-and-memory-ownership.md` §1、§5、§6、§8。
- Current behavior: 正文称 AP/RP 双方维护 PMA，并把 PMA、PBMT、fence、cache maintenance 组合成必须链。
- Required behavior: AP OpenSBI PMA 固定 revision 路径与 RP custom CSR 不可用事实分开；`fence iorw,iorw` 只记录排序且不等于 cache clean；PBMT/真板效果保持未知。
- Required changes: 删除“AP/RP 双方分别维护 PMA”和未经证实的必需链；把抽象/设计判断从“官方事实”降为推论或边界。
- Preserve: 地址 alias、通知不等于数据可见、G5/G7 和 Iteration 001 责任边界。
- Forbidden: 不声称本项目验证 PMA/PBMT 生效，不声称 RP 可写 PMA CSR，不外推到设备 DMA。
- Test witness: 正文第 70、72 行与 `chip-k3-rt24/src/lib.rs::K3Rt24::init` 第 79-83 行冲突。
- GREEN condition: AP/RP 能力分层与 R10/固定源码一致；fence/cache/PMA/PBMT 不互相替代。
- Verification: 冲突措辞零命中、固定 revision 路径核对、证据等级人工审查。
- Stop when: 新证据改变平台属性模型或需要调整 T3 契约。

### T2-R3：链接和任务状态闭合

- Requirement/Scenario: R5-S3；原 A5。
- Depends on: T2-R1、T2-R2.
- Targets: `docs/dma/k3-dma-and-memory-ownership.md` 全部相对链接；`tasks.md` T1-T2 checkbox。
- Current behavior: 未创建的 Iteration 001 文件被链接 7 次；父 Act Response 错报链接全通过；T1-T2 仍未勾选。
- Required behavior: 当前 Cycle 结束时全部本地链接目标存在；未来文件使用无链接文本；仅在 T1/T2 全部 GREEN 后勾选对应 checkbox。
- Required changes: 替换 7 个失效前向链接并扫描全部 Markdown links；更新 T1-T2 为 `[x]`，保持 T3-T5 `[ ]`。
- Preserve: 后续文档名称和 Iteration Map；不提前创建 T3 文件。
- Forbidden: 不用占位文件让链接测试通过，不勾选未验证任务，不修改 T3-T5。
- Test witness: `test -f docs/dma/k3-cache-pma-address-translation.md` 返回 1；`openspec list` 显示 0/5 tasks。
- GREEN condition: 本地链接全部可解析；`openspec list` 显示 2/5 tasks；T3-T5 仍未完成。
- Verification: 相对链接目标扫描、checkbox 精确计数、`openspec list`、diff check。
- Stop when: T1-R1、T2-R1 或 T2-R2 未 GREEN。

**Invariants**

- 父 Cycle 的 Plan Context、Act Response 和终态 Plan Review 不覆写；本 Cycle 形成后继审计链。
- 官方、官方 GitHub、第三方固定 revision、推论和未知项分层不变。
- `others/` 只读；不修改 T3-T5、SNAPSHOT、全局 tasks、M/D/K/R/I。
- 不创建占位产品文档、Evidence 目录或身份型验证机制。

**Non-goals**

- 不实现 Iteration 001，不更新 G5 或 index。
- 不补充新硬件结论，不运行 QEMU 或真板测试。

**Acceptance**

- A1 / T1-R1：正文来源职责不超过可直接确认内容，表内 URL 唯一且元数据一致。
- A2 / T2-R1：UFS 无 OWN 断言；doorbell/OCS completion 与源码一致；GMAC error/reclaim 与源码一致。
- A3 / T2-R2：AP OpenSBI PMA 与 RP CSR 能力分层；fence、cache、PMA/PBMT 不互相替代。
- A4 / T2-R3：全部本地链接可解析且不创建占位；T1-T2 状态与验证一致。
- A5：文档少于 450 行，四级证据和未知项四字段保持，strict validation 与 diff check 通过；T3-T5 未实施。

**Verification**

- 来源：首行 URL 与覆盖表一对一，表行总数与唯一数一致；记录直接打开或不可访问结果。
- 事实：UFS OWN 禁止模式零命中；doorbell-clear/OCS、GMAC OWN/error/clear、AP/RP PMA 边界与源码逐项对照。
- 链接：解析所有 `./`、`../` Markdown 目标；不得忽略未来文件。
- 状态：T1-T2 `[x]`、T3-T5 `[ ]`，`openspec list` 为 2/5。
- 边界和质量：scoped diff、`wc -l < 450`、`git diff --check`、`git diff --cached --check`、OpenSpec strict validation。

**Gate 2 Readiness**

| Dimension | Status | Evidence |
| --- | --- | --- |
| Investigation | PASS | 独立读取产品文档、父 Act Response、UFS/GMAC/RP/OpenSBI 固定 revision 源码并复现链接失败 |
| Design | PASS | 仅恢复原 R1-R5 语义，不改变目标、范围或验收边界 |
| Iteration Plan | PASS | 仍为 Iteration 000 的 T1-T2；不修改 Map |
| Cycle Scope | PASS | 四个 repair item 只关闭既有 A1-A5 gap |
| Task Contracts | PASS | 每项含目标、来源、见证、GREEN、验证和停止条件 |
| Traceability | PASS | T1-R1/T2-R1/R2/R3 映射原 requirement、task 和 Acceptance |
| Verification | PASS | 直接检查事实、链接、任务状态和错误边界，无身份型证据工程 |
| User Plan Approval | PASS | 用户原话：`授权你进行实施修复并自行审计接受`；明确批准 rework 实施并授权串联 Plan Review |

**Persisted Evidence**

- Mode: none

Act Response 足以记录命令、决定性输出和退出码；所有审计证据可低成本重跑。

**Risks and Notes**

- 本 Review 环境无法解析 raw.githubusercontent.com；T1-R1 必须记录可访问结果，不得静默沿用“已直接打开”。
- 父 Cycle 的越权终态保留为历史现场；本 rework 被接受后才构成独立 Plan Review 的有效收敛结果。

## Act Response

- Status: reported

**Implemented**

- T1-R1：复核覆盖表 65 个表内 URL 全部唯一，正文首行 8 个 URL 均在表内恰好一行。当前环境无法解析 GitHub raw host，因此保留父 Cycle 已登记的 2026-09-09 观察记录，不新增刷新结论；官网 SPA URL 只称主题入口，不再让不可读页面承担具体字段。
- T2-R1：删除 UFS UTRD OWN 模型，改为 `prepare_for_device → dma_wmb → doorbell → doorbell bit clear → dma_rmb → complete_for_cpu → OCS/response`；GMAC error 路径改为 DMA 清 OWN 后 CPU invalidate、读取 error、`desc::clear` 和归还 buffer。
- T2-R2：把共享 SRAM 的 PMA 路径拆为 AP OpenSBI X100 hart 的固定 revision 修改与 RP custom CSR 不可用事实；删除 PBMT、PMA、fence 和 cache maintenance 必须共同成立的断言；把计划抽象从“官方事实”降为计划约束或推论。
- T2-R3：将 7 个尚未创建的 Iteration 001 文件链接改为无链接文件名；全部本地 Markdown 目标可解析；T1-T2 改为完成，T3-T5 保持未完成。

**Changed Files and Symbols**

- `docs/dma/k3-dma-and-memory-ownership.md`：修正 §1、§2-§7、U1/U3/U4 和主题导航中的事实层级、设备 completion、PMA 作用域及前向链接。
- `openspec/changes/establish-k3-com260-dma-memory-ownership-baseline/tasks.md`：T1-T2 `[ ] → [x]`；T3-T5 未变。
- 当前 Cycle：Gate 2 获批，填写本 Act Response；未修改父 Cycle、Plan Review 或 Iteration Map。

**Deviations from Plan**

- T1-R1 当前网络环境无法解析 `raw.githubusercontent.com`，`curl` 退出 6。按契约保留既有观察日期，不把本轮失败写成来源刷新；正文只保留本地可验证事实或既有覆盖记录。
- 其他偏差：None。

**Blocker Handoff**

None.

**Blocker Resolution**

None.

**Self-Review**

- Plan compliance: PASS
- Full diff reviewed: PASS
- Critical findings unresolved: 0
- Important findings unresolved: 0
- Minor findings unresolved: 0

Spec review 先确认 A1-A5 与四个 repair item 均满足；随后 code-quality review 确认修改只涉及计划文件、没有占位文档、没有扩大 Iteration 001，且事实标签不再用不可读官网或计划决策替代硬件证据。

**Verification Evidence**

| 验证项 | 命令或操作 | 输出摘录 | 结论 |
| --- | --- | --- | --- |
| RED / UFS | `rg 'UTRD.*OWN\|OWN.*UTRD\|设备清 UTRD\|置 UTRD'` | 原正文 5 个相关命中 | PASS：错误基线已见证 |
| RED / PMA | `rg 'AP / RP 双方分别维护 PMA'` | 原正文第 70 行命中 | PASS：错误基线已见证 |
| RED / 链接 | `test -f docs/dma/k3-cache-pma-address-translation.md` | exit 1 | PASS：失效前向链接已见证 |
| UFS GREEN | 禁止模式检查 + 固定源码对照 | false UFS OWN = 0；源码 UFS OWN = 0；正文含 doorbell-clear/OCS 链 | PASS |
| PMA/GMAC GREEN | 冲突模式检查 + 固定源码对照 | 错误 PMA/GMAC 恢复措辞 = 0；RP 源码明确不能通过 CSR 控制 cache/PMA | PASS |
| 本地链接 | 解析正文全部 `./`、`../` Markdown 目标并 `test -e` | `broken_links=0` | PASS |
| 来源映射 | 覆盖表 URL 计数和首行逐 URL 精确匹配 | `table_rows=65`；`unique_table_urls=65`；8/8 各命中一次 | PASS |
| 来源访问边界 | `curl ...raw.githubusercontent.com...` | `Could not resolve host`；exit 6 | PASS：未刷新，只保留既有观察记录 |
| task 状态 | `openspec list` | `2/5 tasks` | PASS：T1-T2 完成，T3-T5 未完成 |
| 文档行数 | `wc -l docs/dma/k3-dma-and-memory-ownership.md` | `154` | PASS：小于 450 |
| diff | `git diff --check && git diff --cached --check` | 无输出；exit 0 | PASS |
| OpenSpec | `openspec validate establish-k3-com260-dma-memory-ownership-baseline --strict` | `Change ... is valid` | PASS |

**Persisted Evidence**

None required.

**Experience Candidates**

None。此次修复是当前 change 的事实校正，不形成独立 Runbook 或 Incident。

**Remaining Issues**

- T3-T5 属 Iteration 001，仍未实施。
- G5、PMA/PBMT/IOMMU 真板效果和默认 Kit DTS 映射继续保持未知；本 rework 不关闭。

**Commit or Diff Reference**

- 工作区包含父 Cycle 的 staged 实施和本 rework 的 unstaged 修复；未创建 commit。

## Plan Review

- Review Result: accepted

**Findings**

None blocking. T1-R1、T2-R1、T2-R2 与 T2-R3 均已按契约闭合；当前网络仍无法解析 `raw.githubusercontent.com`，但 Act 未据此刷新来源观察，也未新增依赖该访问结果的事实，因此不构成 Acceptance gap。

**Deviation Classification**

ACT-DEVIATION resolved.

**Acceptance Gaps**

None.

**Convergence**

reduced：父 Cycle 独立审计发现的 UFS、GMAC、PMA、链接与任务状态缺口已全部关闭。

**Evidence**

- 独立检查正文禁止模式：false UFS OWN、错误 PMA 双方维护和错误 GMAC recovery 措辞均为 0。
- 解析正文全部本地 Markdown 目标：`broken_links=0`。
- 覆盖表：65 个表内 URL、65 个唯一 URL；正文首行 8 个 URL 均各命中一行。
- `openspec list`：2/5 tasks；T1-T2 完成，T3-T5 未完成。
- `wc -l docs/dma/k3-dma-and-memory-ownership.md`：154，小于 450。
- `git diff HEAD --check` 与 `openspec validate establish-k3-com260-dma-memory-ownership-baseline --strict` 均通过。

**Follow-up Decision**

接受 Iteration 000。返工恢复了原 R1-R5 与 A1-A5 的事实、导航和状态边界；无须继续创建同 Iteration Cycle。按既有 Iteration Map 展开 Iteration 001 的 initial Cycle，但不在本 Review 中实施 T3-T5。

**Iteration Plan Update**

None.

**Next Cycle**

None.

**Next Iteration**

`../001-memory-attributes-finish/000-initial.md`.
