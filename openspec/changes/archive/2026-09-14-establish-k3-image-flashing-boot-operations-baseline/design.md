## Context

MS12 extends the existing documentation-only repository. MS03 already establishes the K3 local/download boot distinction, BootROM → FSBL/SPL → ESOS → OpenSBI → U-Boot → payload chain, image/DTS facts, and unresolved CoM260 DTS mapping. MS09 establishes storage-controller and media boundaries. R22 supplies a still-applicable investigation of fixed-revision Rt-Async-AMP/StarryOS packaging and a layered real-board safety model, but it records no executed build or board operation.

On 2026-09-14 the official `spacemit-com/K3-Ubuntu-Images` repository was directly readable. It describes a UEFI Ubuntu image and flashing workflow for the K3 Pico-ITX board, including GPT firmware partitions, ESP/CIDATA/rootfs, a temporary RAM-resident `u-boot.itb` Fastboot service, and Titan packaging. This is useful official repository behavior but is not proof of CoM260 applicability. The official U-Boot, OpenSBI, and buildroot-ext repository entry points are discoverable; their default branches and generic repository summaries do not by themselves establish a CoM260 workflow.

## Goals / Non-Goals

**Goals**

- Connect artifact production, packaging, load/entry, destination, consumer, and recovery boundaries.
- Separate volatile RAM boot from persistent Fastboot/Titan/SD deployment.
- Make every documented command attributable and explicitly unexecuted.
- Make destructive paths conditional on target identity, board/media applicability, backup, recovery, and success/stop criteria.
- Preserve evidence-class and board-variant boundaries across official docs, official repositories, third-party fixed revisions, inference, and unknowns.

**Non-goals**

- No build, download, flashing, board access, storage write, or runtime proof.
- No executable Runbook and no claim that examples are safe to copy to CoM260.
- No adoption of Pico-ITX UEFI, partition, DTB, or firmware layout as CoM260 truth.
- No revision of existing MS03/MS09 behavior contracts.

## Decisions

### D1: Three documents follow risk and state-transition boundaries

Use one artifact/build document, one volatile RAM/Fastboot document, and one persistent flashing/recovery document. Artifact identity is a data-model problem, RAM boot changes only volatile state, and persistent deployment can make devices unbootable; combining them would obscure different stop and verification boundaries.

### D2: Board applicability is a first-class field

Each workflow fact identifies K3-common, CoM260, Pico-ITX, or fixed third-party scope. Pico-ITX repository behavior may explain mechanisms, but only direct CoM260 evidence can produce a CoM260-specific actionable step.

### D3: Address and partition namespaces remain separate

FIT internal `load`/`entry`, host upload buffers, BootROM transfer locations, storage offsets, GPT partitions, MTD partitions, and filesystem paths are distinct fields. Values from one namespace never fill another.

### D4: Documented commands are evidence-bearing examples, not authorization

Every command records source, actor/environment, state-changing class, execution status, prerequisites, success signal, failure signal, and stop condition. Persistent or host-block-device commands also state target confirmation and recovery requirements. The documents never instruct this change's Act to execute them.

### D5: Recovery precedes persistent write documentation

A persistent path is complete only when the same section identifies entry into download/recovery mode, target/package compatibility, backup or official recovery package, interruption behavior, and post-write boot criteria. Missing prerequisites are an explicit blocker, not a prose footnote.

### D6: Source activation requires direct observation

R21 is a candidate set, not evidence. A URL enters `source-coverage.md` as observed only after its content is directly read and its board scope, branch/version or observation date, purpose, and limitations are recorded. Unreachable or repository-level pages retain limited status; snippets and search summaries cannot supply product facts.

### D7: Existing documents remain factual prerequisites

The new documents link to existing boot chain, image/DTS, storage, UART, DMA, gaps, R18, and R22 material. They do not duplicate full tables or rewrite those baselines. MS11's concurrent closeout is preserved and does not gate the semantic design; any actual overlap in `docs/index.md` or reference files must be reconciled without overwriting user changes.

### D8: Verification checks knowledge behavior directly

RED witnesses are missing target documents, absent source rows, absent navigation, and missing safety/field assertions. GREEN verifies required sections and fields, source URL uniqueness, terminology uniqueness, gap numbering/counts, relative links, forbidden overclaims, `git diff --check`, and strict OpenSpec validation. No identity or evidence framework is introduced.

## Risks / Trade-offs

- Official sources can change after planning. Act must record the observation date and stop if the current content changes board scope or invalidates a Task Contract.
- K3-Ubuntu-Images currently targets Pico-ITX. Treating it as mechanism-only leaves CoM260 partition and UEFI applicability unresolved, but avoids an unsafe extrapolation.
- Commands can look actionable even when labelled unexecuted. Structured provenance, prerequisites, stop conditions, and explicit non-validation language are therefore acceptance requirements.
- Source/reference/index files overlap with the concurrent MS11 closeout. Act must inspect the current diff immediately before each edit and stop on semantic conflict; it may not restore the planning-time file snapshot.
- The work spans at least seven product files and several evidence domains, so light mode is not applicable.

## Migration Plan

No data or runtime migration exists. Implement Iteration 000 first, then expand later Iterations only after Plan Review accepts the preceding stable baseline. Normal change closeout will merge the delta spec and update global status; this Plan does neither.
