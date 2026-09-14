## 1. Sources and artifact model

- [x] 1.1 Directly inspect the MS12 official/candidate source set needed for artifact and deployment behavior, then update `docs/reference/source-coverage.md` with unique URLs, observation metadata, board scope, evidence class, and limitations; preserve inaccessible entries as limited and do not refresh unrelated rows.
- [x] 1.2 Create `docs/boot/k3-image-build-and-artifacts.md` mapping official SDK, official repository, and fixed-revision third-party artifact chains across producer, format, payload, consumer, destination namespace, board scope, evidence class, and unknowns; distinguish FIT load/entry, upload buffers, offsets, partitions, and filesystem paths.

## 2. Volatile RAM boot and Fastboot staging

- [x] 2.1 Create `docs/boot/k3-ram-boot-and-fastboot.md` separating K3 download/local boot, BootROM temporary U-Boot, U-Boot Fastboot, host staging, RAM buffer, FIT parsing/load/entry, DTB applicability, layered success criteria, reset recovery, and stop conditions; mark every command sourced and unexecuted.

## 3. Persistent flashing and recovery

- [x] 3.1 Create `docs/boot/k3-flashing-and-recovery.md` separating Fastboot partition writes, Titan archive/directory flows, SD whole-disk images, and OpenSBI/ESOS or other component updates; require target identity, board/media/package applicability, capacity, backup/recovery, interruption handling, one-component sequencing, post-write checks, and explicit destructive stop conditions.

## 4. Navigation and consistency

- [x] 4.1 Update `docs/reference/known-gaps.md` to map unresolved CoM260 image/package/DTS/partition/download/recovery questions to existing gaps and add exactly one new aggregate gap only if the three documents leave an independent MS12 closure domain.
- [x] 4.2 Update `docs/reference/terminology.md` with only new primary terms actually used by the MS12 documents, preserving unique primary spellings and linking each term to its authoritative section.
- [x] 4.3 Update `docs/index.md` with the three MS12 entries and synchronize boot status, source/term/gap counts, links, and maintenance summaries; verify the full change with direct content assertions, relative-link checks, `git diff --check`, and strict OpenSpec validation.

## Iteration Plan

### Iteration 000: Source and image-artifact baseline

- Tasks: 1.1, 1.2
- Depends on: None
- Stable baseline: MS12 sources have explicit current observation and board scope; artifact production, packaging, consumption, address/partition namespaces, and unknowns are independently searchable.
- Verification boundary: required sources are directly observed or explicitly limited; URL keys remain unique; the artifact document contains all required fields, preserves Pico-ITX/CoM260 separation, and has valid relative links.
- Diagnostic boundary: source reachability/identity, artifact naming and producers, FIT/container fields, boot consumers, address namespaces, partition/filesystem destinations, and board applicability.
- Non-goals: no RAM-boot procedure, persistent flashing procedure, final gap/term/index synchronization, build, download, or hardware action.
- Balance audit: source activation and artifact modeling are mutually dependent and together create the prerequisite vocabulary for later paths; separating them would leave sources without a knowledge surface, while adding RAM/persistent operations would combine distinct safety domains.

### Iteration 001: Volatile RAM-boot baseline

- Tasks: 2.1
- Depends on: Iteration 000
- Stable baseline: readers can evaluate a non-persistent K3 RAM/Fastboot path with actors, state transitions, address checks, layered success signals, rollback, and explicit unknowns.
- Verification boundary: the document distinguishes BootROM/U-Boot/host/RAM/FIT/DTB roles, includes command provenance and unexecuted status, and stops on every unresolved safety prerequisite.
- Diagnostic boundary: device discovery, temporary bootloader service, staging, FIT parse, address/load/entry, DTB, first-byte, reset, and local-boot recovery.
- Non-goals: no partition/MTD/SD writes, no Titan procedure, no build or board execution, and no claim of StarryOS boot success.
- Balance audit: one document is justified because volatile staging and jump form one state machine; it remains separate from persistent writes, which have materially different recovery and destructive-risk boundaries.

### Iteration 002: Persistent deployment and recovery baseline

- Tasks: 3.1
- Depends on: Iteration 000, Iteration 001
- Stable baseline: Fastboot, Titan, SD, and component-update paths are distinguishable and cannot be presented as executable without target, package, partition, backup, and recovery evidence.
- Verification boundary: each path records actor, inputs, persistent targets, prerequisites, source, unexecuted status, success/failure, interruption, stop, and recovery; Pico-ITX behavior is not promoted to CoM260.
- Diagnostic boundary: board/download identity, package format, partition/GPT/MTD mapping, storage capacity, Titan inputs, SD device identity, component sequencing, and post-write recovery.
- Non-goals: no destructive execution, no Runbook, no generic host setup tutorial, and no decision that CoM260 uses a Pico-ITX or third-party layout.
- Balance audit: four persistent mechanisms share the same target-identity and recovery invariant and belong in one comparison surface; splitting by tool would repeat safeguards, while merging with RAM boot would erase the volatile/persistent boundary.

### Iteration 003: Navigation, terminology, and gap convergence

- Tasks: 4.1, 4.2, 4.3
- Depends on: Iteration 000, Iteration 001, Iteration 002
- Stable baseline: all MS12 knowledge is reachable and the authoritative source, terminology, gap, and index surfaces agree.
- Verification boundary: links resolve; URL/term/gap identifiers are unique; counts and statuses agree; existing MS03/MS09 and concurrent MS11 content is preserved; strict OpenSpec validation passes.
- Diagnostic boundary: `known-gaps.md`, `terminology.md`, `index.md`, and final cross-document consistency.
- Non-goals: no new technical facts, source refresh, product execution, SNAPSHOT/tasks/M/R/I update, or MS11 closeout work.
- Balance audit: these summaries require all three documents and form one final consistency result; performing them earlier creates unstable counts, while splitting them yields no independently useful baseline.

## Requirements Traceability Matrix

| Requirement | Scenario | Design | Task | Iteration | Code Surface | Test Witness | Simplification | Status |
|---|---|---|---|---|---|---|---|---|
| R1 artifact traceability | shared names; unknown destination | D1-D3, D6 | 1.1, 1.2 | 000 | coverage; artifact doc | missing doc/rows; required-field assertions | None | Covered |
| R2 namespace separation | staged FIT | D3 | 1.2 | 000 | artifact doc | upload/load/entry/partition distinction assertions | None | Covered |
| R3 volatile path | ready; prerequisite missing | D1, D3-D5 | 2.1 | 001 | RAM/Fastboot doc | actor/state/stop/rollback assertions | None | Covered |
| R4 persistent safeguards | evidenced; recovery missing | D1-D5 | 3.1 | 002 | flashing/recovery doc | four-path field and forbidden-overclaim assertions | None | Covered |
| R5 layered failure | early failure | D1, D5 | 2.1, 3.1 | 001-002 | RAM and recovery docs | layer/rollback/no-escalation assertions | None | Covered |
| R6 source/board scope | Pico-ITX; inaccessible | D2, D6 | 1.1-3.1 | 000-002 | coverage; three docs | scope/evidence-status assertions | None | Covered |
| R7 command attribution | persistent command | D4, D5 | 2.1, 3.1 | 001-002 | RAM and recovery docs | provenance/unexecuted/safety-field assertions | None | Covered |
| R8 navigation consistency | baseline complete | D7, D8 | 4.1-4.3 | 003 | gaps; terms; index | links/counts/uniqueness/strict validate | None | Covered |

## Task Status

- Completed Iterations: 000-sources-and-artifacts, 001-ram-boot, 002-persistent-recovery
- Current Iteration: 003-navigation
- Current Cycle: 000-initial (ready)
- Deferred Iterations: None
- Persisted Evidence: none
