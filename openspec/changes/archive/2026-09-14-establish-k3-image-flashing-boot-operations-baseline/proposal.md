## Why

MS03 documents K3 boot stages and image/DTS facts, while MS09 documents storage-controller boundaries. The repository still lacks one traceable baseline connecting SDK or third-party artifacts to packaging, volatile RAM boot, persistent deployment, success criteria, stop conditions, and recovery requirements. Without that separation, examples for Pico-ITX, generic K3, or fixed-revision StarryOS can be mistaken for a verified CoM260 procedure.

## What Changes

- Establish a source baseline for official K3 image/build/flashing repositories and the existing fixed-revision StarryOS/Rt-Async-AMP implementation.
- Add a document that maps boot artifacts, FIT/ITB structure, build origins, load/entry addresses, partitions, and payload responsibility without claiming that an artifact was built or tested here.
- Add a document for U-Boot RAM boot and Fastboot staging that separates host, BootROM, temporary U-Boot, target RAM, FIT load/entry, and persistent media.
- Add a document for Fastboot partition writes, Titan archives, SD-card images, destructive-action boundaries, stop conditions, and recovery prerequisites.
- Update source coverage, terminology, known gaps, and the main index after the three documents form stable baselines.

### Approved Requirements and Scope

- User instruction: “收尾我在进行，你只管下一个change的计划就好，开始吧批准”.
- The approved scope is the complete MS12 scope in `.claude/docs/tasks.md`.
- This change writes documentation only. It does not install tools, clone or update repositories, build or download images, connect or reset hardware, run Fastboot or Titan, write storage, partition or format devices, or claim real-board validation.
- Commands may be documented only with provenance and an explicit unexecuted status. Destructive commands additionally require prerequisites, target-identification checks, stop conditions, and recovery requirements.
- Official K3 material, official repository behavior, fixed-revision third-party behavior, inference, and unknowns remain separate evidence classes.
- The concurrent MS11 closeout is external to this change. Planning does not alter MS11 artifacts or global state; implementation must preserve its changes and stop on overlap that invalidates the baseline.

### Gate 1 Approval

- Status: PASS
- Evidence: the user approved the complete MS12 scope and the stated non-execution/destructive-operation boundaries on 2026-09-14.
- Scenario gaps resolved by approved defaults: documentation-only execution; no destructive commands are run; Pico-ITX material is reference behavior rather than CoM260 truth; unknown board/media/address/partition values remain unknown.

### Non-goals

- No executable code, build system, image artifact, downloaded source tree, or generated manifest is added to this repository.
- No installation, build, packaging, flashing, partitioning, formatting, USB enumeration, board reset, or boot test is performed.
- No K3-Ubuntu-Images Pico-ITX layout, fixed StarryOS DTB, generic SDK partition JSON, example address, or example command is promoted to a CoM260 default.
- No Runbook or Issue is created; this change creates product knowledge documents, not an executed operating procedure.
- No existing boot, storage, board, UART, or DMA behavior baseline is redefined.

## Scenario Sketch

### S1: Trace an image artifact from source to boot stage

- Given an operator encounters a K3 artifact such as `bootinfo_*.bin`, `FSBL.bin`, `esos.itb`, `fw_dynamic.itb`, `u-boot.itb`, `edk2.itb`, `bootfs`, `rootfs`, or `starryos.uimg`.
- When they consult the artifact document.
- Then they can identify its producer/source, container or payload role, consumer boot stage, volatile or persistent destination, and evidence class.
- Failure boundary: ambiguous names or conflicting build chains remain separate and are not mapped to a CoM260 partition without direct evidence.

### S2: Evaluate a volatile RAM-boot path

- Given a working serial/local-boot baseline and a candidate FIT image.
- When an operator evaluates BootROM/U-Boot/Fastboot staging and `bootm` flow.
- Then host actions, temporary bootloader service, upload buffer, FIT load/entry, DTB checks, success criteria, and reset-based recovery are distinguishable.
- Failure boundary: unsupported command syntax, unidentified device, address overlap, oversized image, unverified DTB, or lost recovery baseline causes a documented stop before boot or write actions.

### S3: Evaluate a persistent Fastboot or Titan operation

- Given a candidate package and a target K3 board.
- When an operator evaluates partition flashing or Titan.
- Then the document requires exact board/media identification, package applicability, partition map, backup/recovery route, one-component-at-a-time scope, and post-write boot criteria.
- Failure boundary: Pico-ITX-only evidence, unknown CoM260 partition mapping, missing recovery image, or unavailable download mode prevents the procedure from being presented as executable for CoM260.

### S4: Evaluate an SD-card image path

- Given an image or prospective image layout and removable media.
- When an operator evaluates SD deployment.
- Then whole-disk images, extracted partitions, boot selection, firmware partitions, payload filesystems, and host block-device risk are separated.
- Failure boundary: an example `dd`, partition, or format command is not supplied as an executable CoM260 procedure while the image format, boot selection, or device identity remains unknown.

### S5: Diagnose failure without escalating destructiveness

- Given failure at discovery, stage, FIT parse, jump, first byte, platform facts, or later workload.
- When the failure matrix is consulted.
- Then diagnosis remains at the earliest failed layer and identifies a safe rollback or stop condition.
- Failure boundary: RAM-boot failure never authorizes combined OpenSBI/ESOS/U-Boot/GPT updates.

### S6: Navigate a consistent knowledge baseline

- Given all three documents are complete.
- When a reader enters through `docs/index.md`, terminology, gaps, or source coverage.
- Then links, counts, evidence labels, and topic ownership agree.
- Failure boundary: inaccessible sources are recorded as such and do not silently become observed facts.

## Capabilities

### New Capabilities

- `k3-image-flashing-boot-operations-baseline`: traceable artifact, volatile boot, persistent deployment, safety, and recovery documentation for K3/CoM260.

### Modified Capabilities

- None. Existing domain requirements remain authoritative and are referenced rather than rewritten.

## Impact

- Planned product surfaces: `docs/boot/k3-image-build-and-artifacts.md`, `docs/boot/k3-ram-boot-and-fastboot.md`, `docs/boot/k3-flashing-and-recovery.md`, `docs/reference/source-coverage.md`, `docs/reference/known-gaps.md`, `docs/reference/terminology.md`, and `docs/index.md`.
- Planning surfaces: this change only.
- Concurrent MS11 closeout remains out of scope.
