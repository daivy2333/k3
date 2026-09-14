# Iteration 001 / Cycle 000: Volatile RAM-Boot Baseline

## Plan Context

- Status: ready
- Iteration: 001-ram-boot
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: ../000-sources-and-artifacts/000-initial.md

**Iteration Scope**

- Change tasks: 2.1
- Depends on: Iteration 000
- Stable baseline: readers can evaluate a non-persistent K3 RAM/Fastboot path through actors, state transitions, memory checks, layered success signals, reset recovery, and explicit unknowns.
- Verification boundary: the document distinguishes BootROM, temporary U-Boot, local U-Boot, host Fastboot, RAM staging, FIT parsing/load/entry, and DTB roles; every command is sourced, structured, and marked unexecuted; unresolved safety prerequisites stop the path.
- Diagnostic boundary: entry and device discovery, temporary bootloader service, staging, FIT parse, buffer/load/entry ranges, DTB applicability, jump, first byte, reset, and local-boot recovery.
- Deferred tasks: 3.1, 4.1-4.3

**Cycle Scope**

- Trigger: initial
- Acceptance gaps: None
- Repair items: None
- Inherited scope: R3, R5-R7; S2, S5; D1-D6 and D8; accepted Iteration 000 artifact identities, namespaces, evidence boundaries, and U1-U4.
- Excluded scope: partition, MTD, GPT, Titan, or SD writes; build/download execution; hardware access; persistent recovery procedure; final navigation, terminology, gaps, and global status.

**Objective**

Create a source-headed RAM-boot decision document that shows how a candidate FIT can move from a host into temporary K3 memory and reach a layered observation point without implying a persistent write or a verified CoM260 procedure.

**Background**

Iteration 000 established artifact identities and separated the FIT upload buffer from internal `load`/`entry`, storage offsets, partitions, and filesystem paths. Existing boot documentation separates K3 `download boot` and `local boot`, but no single document currently combines entry routes, actor transitions, command provenance, address and DTB prerequisites, layered success, failure containment, and reset recovery for volatile staging.

**Investigation Facts**

- Current Baseline: accepted Iteration 000 adds `docs/boot/k3-image-build-and-artifacts.md`; its U3 leaves CoM260 FIT load/entry, reserved-memory safety, and DTB mapping unresolved. It records the fixed-third-party example only: FIT upload buffer `0x180000000`, kernel load/entry `0x140000000`, and FDT load `0x138000000`.
- Current-State Evidence: `docs/boot/com260-boot-chain.md` distinguishes two SoC modes from three U-Boot Fastboot entry routes. BROM-Fastboot is a download-boot path that loads a temporary U-Boot service; `adb reboot bootloader` begins from a running local OS and may be unavailable; serial long-press `s` reaches a local U-Boot shell where the official generic command is `fastboot 0`. CoM260 FEL/RESET placement and actual command support remain unknown.
- Current-State Evidence: fixed-revision `others/Rt-Async-AMP/README.md` records a third-party volatile sequence: U-Boot `fastboot -l 0x180000000 -s 0x04000000 usb 0`, host `fastboot stage .../starryos.uimg`, U-Boot `Ctrl+C`, then `bootm 0x180000000`. These values and syntax are implementation examples, not official CoM260 defaults and not execution evidence.
- Current-State Evidence: the same fixed-revision FIT declares kernel load/entry `0x140000000` and FDT load `0x138000000`; the upload buffer must not overlap the FIT container while U-Boot relocates subimages. `os = "linux"` is FIT metadata and does not identify the StarryOS payload as Linux.
- Current-State Evidence: R22 records that the host lacked `fastboot`, USB enumeration failed, no build artifact existed, and no board command was run. Those historical tool results must not be promoted to a current prerequisite check or runtime result; this Iteration documents decisions only.
- Current-State Evidence: R18 proves a historical factory local-boot path reached a Bianbu root shell and that serial was usable. It does not prove present Fastboot support, address safety, DTB applicability, reset recovery, or StarryOS output.
- Code and Critical Path: source header and evidence scope → prerequisite/stop matrix → entry route → actor/state transition → host staging into an upload buffer → return to U-Boot → FIT inspection → buffer versus load/entry range check → DTB gate → `bootm` decision → layered observation → reset to the pre-existing local-boot baseline. There is no executable code, device interaction, concurrency, timeout implementation, or persistent media mutation.
- Existing verification entry points: absent-file RED; structured Markdown assertions for actors, states, command fields, namespaces, stop conditions, recovery, success layers, and unknowns; source-to-coverage comparison; relative-link resolution; forbidden persistent-write and overclaim scans; line count; `git diff --check`; strict OpenSpec validation.

**Implementation Guidance**

Organize the document as a decision path rather than a copyable Runbook. Start from prerequisites and selectable entry routes, then show one common volatile state machine. Put every command in a table with source, actor/environment, state-changing class, execution status, prerequisites, success signal, failure signal, and stop condition. Keep the generic official entry commands and fixed-third-party stage/boot example visibly separate.

**Behavioral Change**

Current documentation exposes boot stages and scattered RAM/Fastboot examples. Required behavior adds one conditional, explicitly unexecuted knowledge surface where readers can tell which actor owns each step, which state is volatile, what evidence permits progression, what each success layer proves, and when to stop and reset without changing persistent storage.

**Task Contracts**

### 2.1: Document volatile RAM boot and Fastboot staging

- Requirement/Scenario: R3 volatile path; R5 early-layer failure; R6 board/source scope; R7 command attribution; S2 and S5.
- Depends on: accepted tasks 1.1 and 1.2.
- Targets: `docs/boot/k3-ram-boot-and-fastboot.md::entire new document`
- Current behavior: `com260-boot-chain.md` documents entry modes; the artifact document separates namespaces; R22 and fixed-revision third-party files contain an unexecuted StarryOS example. No document joins them into a guarded volatile state machine.
- Required behavior: create a Chinese-first, source-headed document that separates download-boot temporary U-Boot from local U-Boot entry, identifies host/BootROM/U-Boot/RAM/FIT/DTB actors and states, and gates stage, parse, jump, first byte, platform facts, minimal function, and workload progress. Reset returns to the already working local-boot baseline; failure never authorizes a persistent update.
- Required changes: include a prerequisite and stop matrix; entry-route comparison; volatile state diagram; upload-buffer versus FIT load/entry and image-size/range checks; FIT metadata and DTB applicability checks; one structured row for every shown command; success/failure/rollback matrix; four-field unknown closure for unsupported syntax, device identity, safe memory ranges, DTB mapping, and recovery entry. Link the accepted artifact, boot-chain, image/DTS, UART, DMA/memory, and R22 evidence surfaces without copying their full tables.
- Preserve: Iteration 000 evidence classes and namespaces; MS03 local/download distinction; G7 and U1-U4 unknowns; M01-M04; ≤500-line recommendation; source metadata; Chinese-first terminology; all existing documents and concurrent user changes.
- Forbidden: executing or presenting any command as verified; claiming fixed-third-party addresses, size, DTB, syntax, or success for CoM260; using `fastboot flash`, `mtd erase/write`, Titan, `dd`, partitioning, formatting, or any persistent mutation; treating device enumeration, FIT parse, jump, first byte, platform initialization, or workload as interchangeable success; adding tools, scripts, artifacts, or a Runbook.
- Test witness: before creation, `test ! -e docs/boot/k3-ram-boot-and-fastboot.md` succeeds; required actor/state/command/safety assertions therefore fail RED.
- GREEN condition: the file exists, begins with compliant sources, contains the required actors, state transitions, command fields, namespace/range checks, layered signals, stop/reset behavior, scope labels, and four-field unknowns; all relative links resolve and no persistent command is present.
- Verification: assert required headings/fields and actor/state terms; ensure each displayed command has all eight command attributes; compare every external URL with `source-coverage.md`; resolve relative links; scan for forbidden persistent commands and unsupported CoM260 claims; enforce ≤500 lines; run `git diff --check` and strict OpenSpec validation. A missing field/link, unscoped example, persistent mutation, or collapsed success layer fails.
- Stop when: a safe RAM range, supported U-Boot/Fastboot syntax, unique device identity, target DTB, or reset recovery would need to be invented; a source changes board scope; the document requires a persistent operation; or current accepted boot/artifact facts conflict.

**Invariants**

- Documentation only; commands remain sourced and unexecuted.
- RAM staging changes no persistent medium and creates no authority for later writes.
- Official K3, official Pico-ITX, fixed-revision third-party, inference, and unknown evidence remain separate.
- Upload buffer, FIT load/entry, BootROM transfer RAM, storage offsets, partitions, and filesystem paths never collapse.
- Existing MS03/MS09 behavior, G7, U1-U4, and concurrent user changes remain intact.

**Non-goals**

No build, image download, device discovery, USB/serial access, board reset, boot attempt, persistent deployment, Runbook, global status update, or final navigation synchronization.

**Acceptance**

- A1 (R3/S2/D1/D3/T2.1): the document separates entry routes and all host/BootROM/U-Boot/RAM/FIT/DTB transitions, and explicitly identifies every state as volatile or pre-existing.
- A2 (R3/R7/S2/D4/T2.1): every displayed command has source, actor/environment, state-changing class, unexecuted status, prerequisites, success signal, failure signal, and stop condition.
- A3 (R3/R6/S2/D2-D3/D6/T2.1): fixed-third-party addresses, syntax, size, and DTB remain examples; unresolved CoM260 facts use four-field unknown closures and block progression.
- A4 (R5/S5/D5/T2.1): success and failure remain layered from discovery through workload; reset returns to the local-boot baseline, and an early failure never escalates to a persistent action.
- A5 (R6/R8/D7-D8/T2.1): every external URL maps to source coverage, relative links resolve, existing baselines are linked rather than redefined, and the document stays within the split boundary.

**Verification**

- Capture RED for the absent target document and absent required structures.
- After creation, assert actors, state classes, route distinctions, command attributes, address/range checks, success layers, reset path, stop conditions, evidence labels, and four-field unknowns.
- Compare source URLs to coverage and resolve all relative links.
- Reject persistent-write commands and claims that third-party/Pico-ITX values are verified CoM260 defaults.
- Check line count, `git diff --check`, and `openspec validate establish-k3-image-flashing-boot-operations-baseline --strict`.
- Record commands, decisive outputs, exit codes, changed paths, and conclusions in Act Response; do not create an evidence identity mechanism.

**Gate 2 Readiness**

- Requirements coverage: PASS — R3, R5-R7 and S2/S5 map to T2.1, A1-A5, the target file, and direct Markdown checks.
- Investigation completeness: PASS — accepted artifact namespaces, boot entry modes, fixed-third-party sequence, historical execution limits, state path, target surface, and verification entries are identified.
- Design closure: PASS — decision-path structure, actor/state ownership, command schema, volatile boundary, layered results, rollback, evidence classes, and forbidden persistent actions are fixed.
- Task executability: PASS — T2.1 defines its target, behavior, changes, preservation, prohibitions, RED/GREEN, verification, and stop conditions.
- Iteration balance: PASS — one volatile state machine forms an independent result; persistent recovery remains a separate risk domain.
- Traceability: PASS — requirements, scenarios, decisions, task, Iteration, target, witnesses, and Acceptance form a complete chain.
- Verification sufficiency: PASS — checks observe the document's required behavior, sources, links, safety boundaries, and OpenSpec validity directly.
- Identity-evidence prohibition: PASS — no run identity, hash, manifest, revision handshake, or validation framework is planned.
- Substantive unknowns: PASS — unknown command support, memory safety, device identity, DTB mapping, and recovery become stop conditions rather than Act choices.
- Artifact consistency: PASS — this Cycle follows the approved proposal, spec, design, tasks, and accepted Iteration 000 baseline.
- Persisted Evidence: PASS — Mode `none`; all checks are cheap to rerun and Act Response is sufficient.
- User plan approval: PASS — user approved implementation with “批准实施” on 2026-09-14 after receiving the Iteration 001 plan summary.

**Persisted Evidence**

- Mode: none
- Act Response will hold decisive outputs and exit codes; no result requires durable raw evidence.
- Budget: not applicable.

**Risks and Notes**

- Source commands can appear actionable despite warnings; the eight command attributes and decision-path layout are mandatory.
- The fixed-third-party `0x180000000`/`0x04000000` values are evidence for namespace reasoning only until target memory and FIT ranges are confirmed.
- Historical R18/R22 results constrain claims but are not current execution evidence.
- Gate 2 is complete. This Cycle is ready for `openspec-act`; this approval does not authorize Plan to perform implementation itself.

## Act Response

- Status: reported

**Implemented**

完成 T2.1。新增 K3 RAM/Fastboot 决策文档，分开 download boot 临时 U-Boot、OS 重启入口和 local U-Boot 串口入口，建立 Host → BootROM/U-Boot → RAM → FIT → DTB → payload → reset 的易失状态机，并把命令属性、地址门、DTB 门、分层结果和 U1-U4 停止条件放入同一知识表面。没有执行命令或访问设备。

**Changed Files and Symbols**

- `docs/boot/k3-ram-boot-and-fastboot.md`：新增 149 行；包含证据边界、8 项前置/停止矩阵、3 种入口、易失状态机、6 条完整属性命令记录、地址/FIT/DTB 门、9 层结果和 4 个未知项。
- `openspec/changes/establish-k3-image-flashing-boot-operations-baseline/tasks.md`：T2.1 标记完成。
- 当前 Cycle `Act Response`：记录 TDD 见证、Review 和验证结果。

**Deviations from Plan**

None. 目标、范围、证据模式和验证合同均按 Plan Context 执行。

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
| T2.1 RED | `test ! -e docs/boot/k3-ram-boot-and-fastboot.md`；逐项搜索要求结构 | `RED target_absent=PASS`；9 个要求项均 `RED missing` | 修改前目标与 Gate 3 | PASS（预期 RED） |
| 必填结构 | 对 actor、状态、命令字段、namespace、0-8 层和 U1-U4 执行固定字符串断言 | `required_structure=PASS` | A1-A4、T2.1 | PASS |
| 来源覆盖 | 提取首行外部 URL 并查询 `source-coverage.md` | `document_urls=4 uncovered=0` | A5、T2.1 | PASS |
| 相对链接 | 提取 Markdown 相对链接并检查目标存在 | `relative_links=8 broken=0` | A5、T2.1 | PASS |
| 命令属性 | 解析命令表列数并检查每行执行状态 | `command_rows=6 malformed=0` | A2、T2.1 | PASS |
| 持久命令边界 | 扫描 Fastboot/MTD/整盘写入命令字面量 | `persistent_command_literals=0` | A3-A4、T2.1 | PASS |
| 文档长度 | `wc -l docs/boot/k3-ram-boot-and-fastboot.md` | `149` | A5、M02 | PASS |
| Markdown diff | `git diff --check --` 当前 Cycle 文件 | 无输出，退出 0 | 产品与反馈变更 | PASS |
| OpenSpec | `openspec validate establish-k3-image-flashing-boot-operations-baseline --strict` | `Change 'establish-k3-image-flashing-boot-operations-baseline' is valid`，退出 0 | change 结构 | PASS |

**Persisted Evidence**

None required.

**Experience Candidates**

None — 本轮只创建文档并运行可低成本重跑的结构检查，没有端到端硬件操作或范围外实质缺陷。

**Remaining Issues**

None within Iteration 001。U1-U4 是计划要求保留的产品未知项，会阻止未经证据的实际操作，不是实施缺口。

**Commit or Diff Reference**

未提交。当前 Cycle 产品修改为新增 `docs/boot/k3-ram-boot-and-fastboot.md`；并更新 T2.1 与本 Act Response。未修改用户的 MS11 收尾内容或 `others/`。

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

- 独立解析命令表：6 行均有 source、actor/environment、state-changing class、未执行状态、prerequisite、success、failure 和 stop 字段，`malformed=0`。
- 独立检查入口与分层矩阵：3 种入口、0-8 层结果和 U1-U4 均存在；上传缓冲区、FIT `load`/`entry`、BootROM 传输内存和持久介质保持分离。
- 首行 4 个外部 URL 均映射到来源覆盖表；8 个相对链接目标存在；未发现持久写命令字面量或把第三方/Pico-ITX 值提升为 CoM260 默认值。
- 采信 Act Response 的 RED、结构、链接和 149 行结论；独立复核后覆盖范围未变化。
- `git diff --check` 无输出、退出 0；严格 OpenSpec 校验输出 `Change 'establish-k3-image-flashing-boot-operations-baseline' is valid`、退出 0。

**Follow-up Decision**

接受当前 Cycle 和 Iteration 001。既有 Iteration Map 仍适用，已展开 Iteration 002 的初始 Cycle；详细计划保持 `draft`，等待用户审计批准后才能实施。

**Iteration Plan Update**

None.

**Next Cycle**

None.

**Next Iteration**

`../002-persistent-recovery/000-initial.md`
