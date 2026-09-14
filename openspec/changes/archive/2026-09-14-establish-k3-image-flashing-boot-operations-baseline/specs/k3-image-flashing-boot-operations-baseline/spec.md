## Purpose

Define the required K3/CoM260 image artifact, volatile boot, persistent deployment, safety, recovery, source, and navigation knowledge baseline.

## ADDED Requirements

### Requirement: Image artifacts are traceable across production and consumption

The documentation SHALL distinguish each artifact's producer or source, format/container, payload role, consuming boot stage, destination namespace, board applicability, evidence class, and unknowns.

#### Scenario: Artifact name is shared by different build chains

- **WHEN** official SDK, official repository, and fixed-revision third-party material use overlapping or different names
- **THEN** the documentation keeps their identities and responsibilities separate and does not infer equivalence from filenames

#### Scenario: Artifact destination is not proven

- **WHEN** a load address, partition, offset, or filesystem path is absent or only shown for another board
- **THEN** the CoM260 destination remains unknown and records the evidence needed to resolve it

### Requirement: Address and partition namespaces remain distinct

The documentation SHALL distinguish FIT load/entry addresses, upload buffers, BootROM transfer memory, storage offsets, GPT partitions, MTD partitions, and filesystem paths.

#### Scenario: A staged FIT contains internal load addresses

- **WHEN** an image is uploaded to a temporary RAM buffer and parsed by U-Boot
- **THEN** the upload address is not described as the kernel entry, DTB load address, or persistent storage location

### Requirement: Volatile RAM boot is separated from persistent deployment

The documentation SHALL provide a non-persistent K3 RAM-boot decision path that preserves working firmware and identifies host, BootROM/U-Boot, RAM, FIT, DTB, success, failure, reset, and recovery boundaries.

#### Scenario: RAM boot prerequisites are satisfied

- **WHEN** board identity, serial recovery baseline, command support, device selection, image size, memory ranges, FIT metadata, and DTB applicability are confirmed
- **THEN** the documented flow can progress through staging, FIT inspection, jump, and layered observation without claiming persistent writes

#### Scenario: A RAM boot prerequisite is missing

- **WHEN** device identity, U-Boot syntax, address safety, FIT size, DTB mapping, or recovery baseline cannot be confirmed
- **THEN** the documentation requires stopping before staging or jumping and identifies the missing evidence

### Requirement: Persistent paths require destructive-action safeguards

The documentation SHALL separate Fastboot partition writes, Titan packages, SD whole-disk images, and component updates, and SHALL pair each state-changing path with target identity, applicability, backup/recovery, interruption, success, failure, and stop conditions.

#### Scenario: Persistent operation is fully evidenced

- **WHEN** board/media identity, package scope, partition map, recovery route, capacity, command provenance, and post-write criteria are all known
- **THEN** the document may present the sourced procedure as unexecuted and conditional, with one-component-at-a-time sequencing

#### Scenario: Persistent operation lacks recovery evidence

- **WHEN** a partition, device, package, download mode, backup, or recovery route is unknown
- **THEN** the document does not present the operation as executable for CoM260 and forbids substituting example values

### Requirement: Failure diagnosis does not escalate destructiveness

The documentation SHALL organize failures by discovery, transfer, parse, jump, first-byte, platform, device, interrupt, minimal-function, and workload layers.

#### Scenario: An early layer fails

- **WHEN** a flow fails before its current layer's success signal
- **THEN** diagnosis remains within that layer and later or more destructive changes are not proposed as a shortcut

### Requirement: Source and board scope are explicit

The documentation SHALL distinguish official K3 documents, official repository behavior, fixed-revision third-party behavior, inference, and unknowns, with board scope and observation metadata.

#### Scenario: Official repository targets Pico-ITX

- **WHEN** a repository documents a K3 Pico-ITX image or flashing layout
- **THEN** it may support mechanism understanding but SHALL NOT establish CoM260 partition, DTB, firmware, or command applicability

#### Scenario: Candidate source is inaccessible

- **WHEN** the source cannot be directly read
- **THEN** its status and limitation are recorded without deriving product facts from snippets or neighboring sources

### Requirement: Commands remain attributable and unexecuted

Every documented command SHALL identify its source, actor/environment, state-changing class, execution status, prerequisites, success and failure signals, and stop condition.

#### Scenario: Command can alter persistent media

- **WHEN** a command flashes, erases, writes, partitions, formats, or targets a host block device
- **THEN** target confirmation and recovery requirements are included and the command is explicitly marked not executed by this project

### Requirement: Navigation and summaries remain consistent

The three MS12 documents SHALL be reachable from the main index and SHALL remain consistent with source coverage, terminology, known gaps, and existing boot/storage baselines.

#### Scenario: MS12 baseline is complete

- **WHEN** artifact, volatile, and persistent documents satisfy their requirements
- **THEN** links resolve, source URLs and terms are unique, gap numbering and counts agree, and no existing behavior baseline is silently redefined
