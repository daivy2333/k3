# Iteration 002 / Cycle 000: Persistent Deployment and Recovery Baseline

## Plan Context

- Status: ready
- Iteration: 002-persistent-recovery
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: ../001-ram-boot/000-initial.md

**Iteration Scope**

- Change tasks: 3.1
- Depends on: Iteration 000, Iteration 001
- Stable baseline: Fastboot partition writes, Titan package flows, SD whole-disk images, and component updates are distinguishable and remain non-executable until target, package, layout, capacity, backup, recovery, interruption, and post-write evidence are complete.
- Verification boundary: each path records actors, inputs, persistent targets, prerequisites, source, unexecuted status, success/failure, interruption, stop, and recovery; no Pico-ITX or fixed-third-party layout is promoted to CoM260.
- Diagnostic boundary: board/download identity, package scope, partition/GPT/MTD mapping, storage capacity, Titan inputs, Host block-device identity, component sequencing, post-write checks, interruption, and recovery.
- Deferred tasks: 4.1-4.3

**Cycle Scope**

- Trigger: initial
- Acceptance gaps: None
- Repair items: None
- Inherited scope: R4-R7; S3-S5; D1-D8; accepted Iteration 000 artifact and namespace baseline; accepted Iteration 001 entry, command, layered-result, reset, and unknown boundaries.
- Excluded scope: executing a write; installing tools; connecting/resetting hardware; producing images; selecting a CoM260 layout; executable Runbook; final index, terminology, and gap synchronization; global status.

**Objective**

Create a source-headed comparison and decision document for four persistent deployment classes, where no path can appear executable for CoM260 until its destructive prerequisites and recovery route are explicitly evidenced.

**Background**

The accepted artifact baseline identifies firmware, filesystem, GPT/MTD, and whole-image objects. The accepted RAM-boot baseline establishes entry, staging, layered success, and non-escalation. Persistent deployment adds irreversible or hard-to-recover state changes and therefore needs its own target-identity, backup, interruption, sequencing, verification, and recovery contract.

**Investigation Facts**

- Current Baseline: `docs/boot/k3-image-build-and-artifacts.md` separates SDK, Pico-ITX, and fixed-third-party artifact chains and leaves CoM260 package, media, partition, FIT/DTB, and compatibility facts in U1-U4. These unknowns remain blocking for persistent action.
- Current Baseline: `docs/boot/k3-ram-boot-and-fastboot.md` defines a volatile prerequisite and layered failure model. RAM success can support payload evaluation but does not prove a persistent package, partition, component, or recovery path.
- Current-State Evidence: official docs-buildroot material distinguishes K3 firmware layouts and documents generic Fastboot flashing plus Buildroot ZIP use by Titan. Existing accepted docs do not establish a CoM260-specific package name, partition map, storage capacity, Titan version, backup method, or recovery image.
- Current-State Evidence: official `K3-Ubuntu-Images` behavior targets Pico-ITX. Its README, gadget, `fastboot.yaml`, `image_flash.py`, `partition_universal.json`, and `partition_4M.json` expose GPT/MTD, Fastboot, Titan, and whole-image mechanisms. Their offsets, sizes, payload choices, temporary boot files, and target selection are `non-target-board` evidence only.
- Current-State Evidence: fixed-revision `others/Rt-Async-AMP/README.md` stages `esos.itb` or `opensbi.itb` into RAM and then describes MTD erase/write of same-named partitions. The local source does not prove the current board's partition names, media, capacity, backup, recovery entry, or component compatibility; those commands must not be copied into product documentation as executable steps.
- Current-State Evidence: R22 recommends preserving FSBL, bootinfo, GPT, and U-Boot during first bring-up and changing at most one component after RAM boot. It records no successful backup, write, reset recovery, Titan run, SD write, or post-write boot.
- Current-State Evidence: SD deployment has two distinct objects: a whole-disk image written to a Host-selected block device and partition/filesystem payloads extracted or assembled within an image. Neither proves K3 boot selection, firmware partition completeness, or CoM260 media compatibility.
- Code and Critical Path: evidence class and target identity → select exactly one path → identify package/artifact and persistent target namespace → prove layout/capacity → prove backup and recovery → model interruption → show sourced unexecuted command/action attributes → perform no action in this change → define post-write checks and earliest-layer rollback. The product remains Markdown only.
- Existing verification entry points: absent-file RED; structured assertions over four paths, common safety fields, command/action attributes, one-component sequencing, success/failure/interruption/recovery, unknown closure, and prohibited claims; source coverage and relative links; destructive-command presentation scan; line count; `git diff --check`; strict OpenSpec validation.

**Implementation Guidance**

Organize the document around a common persistent-operation contract followed by four path sections. Prefer action schemas and blocked examples over copyable destructive command blocks. Where a source contains a destructive command, describe its role and provenance without reproducing a ready-to-run command unless all required attributes and explicit non-execution/stop text remain in the same row. Keep target namespaces and board scopes visible in every path.

**Behavioral Change**

Current documents explain artifacts and volatile staging but do not provide a unified persistent safety contract. Required behavior adds a comparison surface that distinguishes Fastboot partition, Titan archive/directory, SD whole-disk, and individual firmware-component paths and makes missing identity, capacity, backup, recovery, interruption, or post-write facts a visible blocker.

**Task Contracts**

### 3.1: Document persistent flashing and recovery boundaries

- Requirement/Scenario: R4 persistent safeguards; R5 non-escalating failure; R6 source/board scope; R7 command attribution; S3-S5.
- Depends on: accepted tasks 1.1, 1.2, and 2.1.
- Targets: `docs/boot/k3-flashing-and-recovery.md::entire new document`
- Current behavior: artifact and RAM documents identify objects, namespaces, entry paths, and unknowns, while official Pico-ITX and fixed-third-party sources contain incompatible persistent mechanisms. No common document prevents those mechanisms from being mistaken for a CoM260 procedure.
- Required behavior: create a Chinese-first, source-headed decision document that separately models Fastboot partition writes, Titan archives/directories, SD whole-disk images, and one-component OpenSBI/ESOS or other firmware updates. Each path must expose source/board scope, actors, input/package, persistent target, prerequisites, target identification, capacity/layout checks, backup/recovery, interruption behavior, success/failure, post-write check, stop condition, and unexecuted status.
- Required changes: include a shared destructive-action gate; a four-path comparison; per-path state transitions and responsibility boundaries; a structured action/command table using the eight inherited command attributes plus target confirmation and recovery; one-component sequencing that preserves FSBL/bootinfo/GPT/U-Boot unless independently justified; layered post-write diagnosis; four-field unknown closures for CoM260 package/layout, Titan/tool compatibility, Host block-device/boot selection, and component backup/recovery. Link the artifact, RAM, boot-chain, image/DTS, storage, gaps, and R22 surfaces without duplicating their full tables.
- Preserve: official/Pico-ITX/fixed-third-party evidence separation; Iteration 000 namespaces and U1-U4; Iteration 001 layered success and non-escalation; MS03/MS09 facts; G7; M01-M04; source metadata; Chinese-first terminology; ≤500-line recommendation; all concurrent user changes.
- Forbidden: executing or claiming validation of any command; selecting a target device, partition, offset, package, image, DTB, capacity, or recovery path by assumption; presenting Pico-ITX JSON/gadget/Titan flow or fixed-third-party MTD commands as CoM260 instructions; combining component changes; treating a successful transfer/write as boot success; adding tools, scripts, manifests, images, Runbooks, or identity-evidence mechanisms.
- Test witness: before creation, `test ! -e docs/boot/k3-flashing-and-recovery.md` succeeds; four-path, safety-field, interruption, recovery, and command/action assertions fail RED.
- GREEN condition: the file exists, begins with compliant sources, includes all four paths and common fields, marks all actions unexecuted and conditional, blocks every unresolved destructive prerequisite, defines one-component sequencing and post-write/recovery layers, resolves links, and remains free of an actionable unsupported CoM260 procedure.
- Verification: assert four path headings and common fields; parse action rows for all attributes; verify explicit non-execution and board scope; compare all external URLs to source coverage; resolve relative links; scan for raw destructive command blocks and unsupported CoM260 defaults; enforce ≤500 lines; run `git diff --check` and strict OpenSpec validation. Missing recovery/interruption/identity fields, a collapsed namespace, an unscoped example, or an actionable unsupported command fails.
- Stop when: any path needs an invented CoM260 package, partition, capacity, device, boot selection, tool version, backup, or recovery fact; a source changes scope; a destructive command cannot be presented safely under the approved schema; or accepted artifact/RAM/storage facts conflict.

**Invariants**

- Documentation only; no Host or target state is changed.
- Every persistent action remains sourced, explicitly unexecuted, conditional, and paired with target and recovery gates.
- A successful transfer or write proves only its own layer, not boot or workload success.
- Only one component may be considered at a time; RAM baseline and original local boot remain prerequisites.
- Board scope, artifact identity, and all storage/address namespaces remain distinct.
- Existing MS03/MS09 behavior, G7, U1-U4, and concurrent user changes remain intact.

**Non-goals**

No build, download, flashing, erase, partition, format, block-device write, board access, recovery execution, tool installation, Runbook, source refresh, global status update, or navigation synchronization.

**Acceptance**

- A1 (R4/S3-S4/D1-D5/T3.1): Fastboot partition, Titan, SD whole-disk, and component-update paths are separate and each includes actors, input, persistent target, identity, applicability, capacity/layout, backup/recovery, interruption, success/failure, post-write, stop, and unexecuted status.
- A2 (R4/R7/S3-S4/D4-D6/T3.1): every displayed action or command has source, actor/environment, state-changing class, execution status, prerequisites, success signal, failure signal, stop condition, target confirmation, and recovery requirement.
- A3 (R4/R6/S3-S4/D2-D3/D6/T3.1): Pico-ITX and fixed-third-party examples remain scoped evidence; no CoM260 package, layout, device, capacity, tool, DTB, or recovery fact is inferred.
- A4 (R5/S5/D5/T3.1): interruption and post-write outcomes remain layered, one-component sequencing is explicit, and failures never escalate to broader firmware/GPT changes.
- A5 (R6/R8/D7-D8/T3.1): sources map to coverage, relative links resolve, accepted baselines remain authoritative, and the document stays within the split boundary.

**Verification**

- Capture RED for the absent target and absent four-path/safety structures.
- Assert four path classes, common persistent-operation fields, action attributes, target and recovery gates, one-component sequencing, interruption handling, layered post-write results, evidence scope, and four-field unknowns.
- Compare source URLs with coverage and resolve relative links.
- Reject raw unsupported destructive command blocks and claims that Pico-ITX or third-party values are CoM260 defaults.
- Check line count, `git diff --check`, and `openspec validate establish-k3-image-flashing-boot-operations-baseline --strict`.
- Record commands, decisive output, exit codes, changed paths, and conclusions in Act Response; create no Evidence identity mechanism.

**Gate 2 Readiness**

- Requirements coverage: PASS — R4-R7 and S3-S5 map to T3.1, A1-A5, the target, and direct document checks.
- Investigation completeness: PASS — accepted artifact/RAM baselines, four mechanisms, evidence scopes, destructive state path, recovery gaps, and validation entries are identified.
- Design closure: PASS — shared gate, per-path structure, action schema, one-component ordering, interruption, layered post-write results, and unknown treatment are fixed.
- Task executability: PASS — T3.1 defines target, behavior, changes, preservation, prohibitions, RED/GREEN, verification, and stop conditions.
- Iteration balance: PASS — the four mechanisms share the same target/recovery invariant and form one comparison surface; they remain separate from volatile staging and final navigation.
- Traceability: PASS — requirements, scenarios, decisions, task, Iteration, file, witness, and Acceptance form a complete chain.
- Verification sufficiency: PASS — checks observe path separation, action safety, source scope, links, unsupported-command absence, and OpenSpec validity.
- Identity-evidence prohibition: PASS — no hash chain, run identity, manifest, target pin, or validation framework is planned.
- Substantive unknowns: PASS — missing CoM260 package/layout/tool/device/recovery facts are document stop conditions, not Act decisions.
- Artifact consistency: PASS — this Cycle follows the approved proposal, spec, design, tasks, and accepted Iterations 000-001.
- Persisted Evidence: PASS — Mode `none`; all results are cheap to rerun and Act Response is sufficient.
- User plan approval: PASS — user approved implementation with “批准实施” on 2026-09-14 after receiving the Iteration 002 plan summary.

**Persisted Evidence**

- Mode: none
- Act Response will record decisive outputs and exit codes; durable raw logs are unnecessary because no product operation occurs.
- Budget: not applicable.

**Risks and Notes**

- Destructive source commands can remain unsafe even with warnings; the common gate and per-action attributes must stay adjacent to any command representation.
- Pico-ITX layouts and fixed-third-party MTD names are useful only for mechanism comparison.
- A recovery description is incomplete without a separately evidenced entry path, compatible recovery package, target identity, and post-recovery success criterion.
- Gate 2 is complete. This Cycle is ready for `openspec-act`; this approval does not authorize Plan to perform implementation itself.

## Act Response

- Status: reported

**Implemented**

完成 T3.1。新增 K3 持久部署与恢复决策文档，以共同破坏性操作门约束 Fastboot 分区、Titan 包/目录、SD 整盘和单组件更新四条路径；所有动作仅作来源对照并标记未执行，没有提供可复制的破坏性命令。

**Changed Files and Symbols**

- `docs/boot/k3-flashing-and-recovery.md`：新增 144 行；包含 9 项共同门、四路径比较、4 条完整动作记录、路径责任边界、中断/恢复矩阵、7 层写后判据和 U1-U4。
- `openspec/changes/establish-k3-image-flashing-boot-operations-baseline/tasks.md`：T3.1 标记完成。
- 当前 Cycle `Act Response`：记录 TDD 见证、两阶段 Review 和最终验证。

**Deviations from Plan**

None. 目标、证据边界、禁止项和验证合同均按 Plan Context 执行。

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

**Verification Evidence**

| 验证项 | 命令或操作 | 输出摘录 | 覆盖范围 | 结论 |
|---|---|---|---|---|
| T3.1 RED | `test ! -e docs/boot/k3-flashing-and-recovery.md`；搜索四路径与安全字段 | `RED target_absent=PASS`；8 项均 `RED missing` | 修改前目标与 Gate 3 | PASS（预期 RED） |
| 必填结构 | 固定字符串检查共同门、四路径、动作字段、单组件顺序和 U1-U4 | `required_structure=PASS` | A1-A4、T3.1 | PASS |
| 来源覆盖 | 提取首行 URL 并查询 `source-coverage.md` | `document_urls=10 uncovered=0` | A3/A5、T3.1 | PASS |
| 相对链接 | 提取 Markdown 相对链接并检查目标存在 | `relative_links=8 broken=0` | A5、T3.1 | PASS |
| 动作属性 | 解析 4 行动作的 11 个合同字段 | `action_rows=4 malformed=0` | A1/A2、T3.1 | PASS |
| 破坏性命令边界 | 扫描 shell 代码块及 Fastboot/MTD/整盘写命令字面量 | `raw_destructive_commands=0` | A2-A4、T3.1 | PASS |
| 文档长度 | `wc -l docs/boot/k3-flashing-and-recovery.md` | `144` | A5、M02 | PASS |
| Markdown diff | `git diff --check --` 当前 Cycle 文件 | 无输出，退出 0 | 产品与反馈变更 | PASS |
| OpenSpec | `openspec validate establish-k3-image-flashing-boot-operations-baseline --strict` | `Change 'establish-k3-image-flashing-boot-operations-baseline' is valid`，退出 0 | change 结构 | PASS |

**Persisted Evidence**

None required.

**Experience Candidates**

None — 本轮未执行持久操作，只运行可低成本重跑的 Markdown 检查，没有范围外实质缺陷。

**Remaining Issues**

None within Iteration 002。U1-U4 是计划要求保留的产品未知项，它们使实际持久操作保持不可执行，不是当前实施缺口。

**Commit or Diff Reference**

未提交。当前 Cycle 产品修改为新增 `docs/boot/k3-flashing-and-recovery.md`；并更新 T3.1 与本 Act Response。未修改用户的 MS11 收尾内容或 `others/`。

## Plan Review

- Review Result: accepted

**Findings**

None. 未发现阻塞 Acceptance 或非阻塞 Minor finding。

**Deviation Classification**

None. 实现与 Plan Context 一致。

**Acceptance Gaps**

None. A1-A5 均满足。

**Convergence**

N/A：首次 Review，无父 Cycle gap。

**Evidence**

- 独立解析动作表：4 行均包含 source、actor/environment、state-changing class、未执行状态、prerequisite、success、failure、stop、target confirmation 和 recovery requirement，`malformed=0`。
- 独立检查四路径、九项共同门、中断/恢复、七层写后判据与 U1-U4；单组件顺序和失败不扩大修改范围均明确。
- 首行 10 个外部 URL 均映射到来源覆盖表；8 个相对链接目标存在；未发现 shell 命令块或可复制的 Fastboot/MTD/整盘写入命令。
- 采信 Act Response 的 RED、结构和 144 行结论；独立复核后覆盖范围未变化。
- `git diff --check` 无输出、退出 0；严格 OpenSpec 校验输出 `Change 'establish-k3-image-flashing-boot-operations-baseline' is valid`、退出 0。

**Follow-up Decision**

接受当前 Cycle 和 Iteration 002。既有 Iteration Map 仍适用，已展开最终 Iteration 003 的初始 Cycle；详细计划保持 `draft`，等待用户审计批准后才能实施。

**Iteration Plan Update**

None.

**Next Cycle**

None.

**Next Iteration**

`../003-navigation/000-initial.md`
