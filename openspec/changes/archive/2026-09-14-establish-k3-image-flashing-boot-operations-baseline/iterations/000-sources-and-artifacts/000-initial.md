# Iteration 000 / Cycle 000: Source and image-artifact baseline

## Plan Context

- Status: ready
- Iteration: 000-sources-and-artifacts
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: None

**Iteration Scope**

- Change tasks: 1.1, 1.2
- Depends on: None
- Stable baseline: MS12 sources have explicit current observation and board scope; artifact production, packaging, consumption, namespaces, and unknowns are searchable in one document.
- Verification boundary: sources are observed or explicitly limited, URL keys are unique, required artifact fields and board/evidence boundaries are present, and relative links resolve.
- Diagnostic boundary: source reachability and identity; artifact names, producers, formats, payloads, consumers, address/partition namespaces, and board scope.
- Deferred tasks: 2.1, 3.1, 4.1-4.3

**Cycle Scope**

- Trigger: initial
- Acceptance gaps: None
- Repair items: None
- Inherited scope: proposal R1, R2, R6 and D1-D3, D6-D8; M01-M04; existing MS03/MS09 baselines.
- Excluded scope: RAM-boot procedure, persistent flashing/recovery procedure, final navigation/terms/gaps, product execution, MS11 closeout, and global status updates.

**Objective**

Create a directly sourced artifact baseline that later Iterations can use without guessing whether a name, address, partition, or workflow applies to generic K3, CoM260, Pico-ITX, or fixed third-party code.

**Background**

Existing boot documents explain stages and selected image/DTS facts but do not connect all MS12 artifacts to producers, consumers, namespaces, and deployment paths. R21 contains deferred candidate URLs. R22 contains a 2026-09-11 fixed-revision investigation. The user approved documentation-only MS12 planning and prohibited execution of build, download, flashing, partitioning, formatting, and board actions.

**Investigation Facts**

- Current Baseline: repository documentation is the only product; M04 prohibits executable code. `docs/boot/com260-boot-chain.md` and `com260-image-and-dts.md` establish boot stages, existing SDK image facts, and G7 DTS uncertainty. `docs/reference/source-coverage.md` currently has 70 unique URLs before MS12 implementation.
- Current-State Evidence: R22 traces local fixed revisions through `others/Rt-Async-AMP/envs/k3-com260.toml` → `xtask/src/build.rs` → OpenSBI/ESOS/StarryOS packaging, including `starryos.uimg` upload `0x180000000`, kernel load/entry `0x140000000`, and DTB load `0x138000000`; these are third-party implementation facts, not CoM260 defaults. Its tool/build/board checks are historical and must not be presented as current execution evidence.
- Current-State Evidence: direct read on 2026-09-14 of `https://github.com/spacemit-com/K3-Ubuntu-Images` shows a K3 Pico-ITX UEFI image with GPT firmware partitions (`env`, `bootinfo`, `fsbl`, `esos`, `opensbi`, `uboot`), ESP/CIDATA/ext4 rootfs, temporary RAM-only `u-boot.itb` Fastboot service, Fastboot and Titan paths, and explicit Pico-ITX scope. It does not establish CoM260 applicability.
- Current-State Evidence: official `spacemit-com/uboot-2022.10`, `spacemit-com/opensbi`, and `spacemit-com/buildroot-ext` repository entry points are reachable, but repository landing pages alone do not define a CoM260 artifact or partition contract. Exact files used for product facts must be opened directly.
- Code and Critical Path: source entry → direct observation and scope classification → unique coverage row → artifact matrix → later RAM/persistent documents. There is no runtime call graph, concurrent state, cancellation, or executable test fixture; lifecycle is documentation state only.
- Existing verification entry points: structured `rg`/parsing assertions over Markdown, relative-link resolution, URL uniqueness, forbidden-overclaim scans, `git diff --check`, and `openspec validate establish-k3-image-flashing-boot-operations-baseline --strict`.
- Concurrent boundary: MS11 is being closed independently. Before modifying shared reference files, Act must read the current diff and preserve all concurrent content. A semantic overlap or unexpected count baseline invalidates this Cycle and requires a Plan decision.

**Implementation Guidance**

Inspect only sources necessary for the artifact matrix. Prefer exact official files over repository roots; record current branch/version or observation date. Reuse R22 fixed-revision facts only after checking that the local referenced paths still match their captured scope. Organize the artifact document by producer/chain and then use a common field table so similarly named files remain distinguishable.

**Behavioral Change**

Current behavior provides boot-stage and image/DTS facts but no unified artifact contract, and R21 URLs are deferred candidates. Required behavior adds directly observed or explicitly limited source rows and a document where each artifact exposes producer, format, payload, consumer, destination namespace, board scope, evidence class, and unknowns. No command becomes authorized or verified.

**Task Contracts**

### 1.1: Establish MS12 source coverage

- Requirement/Scenario: R1 shared names/unknown destination; R6 Pico-ITX/inaccessible source.
- Depends on: None
- Targets: `docs/reference/source-coverage.md::coverage table and metadata`
- Current behavior: boot/image official pages and GitHub mirrors are active, while R21 holds deferred repository candidates not yet qualified for MS12; K3-Ubuntu-Images is absent from coverage.
- Required behavior: directly inspect the exact official pages/files needed by tasks 1.2-3.1; add or update unique rows with type, applicability, topic, state, revision/branch or observation date, reachability, purpose, and limitation. Explicitly label K3-Ubuntu-Images as Pico-ITX behavior and preserve inaccessible sources without deriving facts.
- Required changes: cover at minimum the directly used K3-Ubuntu-Images README and exact layout/flow files, relevant SpacemiT boot/image docs already present, and any exact official U-Boot/OpenSBI/buildroot-ext files actually cited by the artifact document; reuse existing rows instead of duplicating URLs.
- Preserve: M01 source hierarchy; existing URL keys, observation dates, evidence classes, counts, MS11 concurrent content, and unrelated topic responsibilities.
- Forbidden: refreshing unrelated rows; recording a search result/snippet as observed; treating a repository root as proof of a file-level fact; labelling Pico-ITX or third-party behavior as CoM260; adding downloaded source or executable tools.
- Test witness: before change, exact URL search shows the MS12-specific K3-Ubuntu-Images file rows are absent and R21 is `deferred`; parse the coverage URL column to confirm the current count and uniqueness immediately before editing.
- GREEN condition: every artifact-document external URL has one coverage row or an explicit pre-existing-row mapping; all new rows carry accurate scope/status; URL count equals unique count.
- Verification: parse URL column for total/uniqueness; compare artifact header/body URLs to coverage; assert Pico-ITX and unreachable limitations; inspect diff for unrelated refreshes. Any duplicate, missing cited URL, false observation, or overwritten concurrent content fails.
- Stop when: source content changes the approved scope, an exact source cannot support a required contract field, the current coverage baseline conflicts semantically with MS11 closeout, or resolving applicability requires choosing a CoM260 partition/DTS/layout without evidence.

### 1.2: Create the image build and artifact map

- Requirement/Scenario: R1 shared names/unknown destination; R2 staged FIT; R6 board scope.
- Depends on: 1.1
- Targets: `docs/boot/k3-image-build-and-artifacts.md::entire new document`
- Current behavior: artifact facts are distributed across two boot documents and R22; there is no common producer/format/payload/consumer/destination/applicability matrix, and address namespaces can only be reconstructed across sources.
- Required behavior: create a source-headed document covering official SDK artifacts, official K3-Ubuntu-Images/Pico-ITX behavior, and fixed-revision Rt-Async-AMP/StarryOS behavior. For every artifact, state producer/source, format/container, payload role, consuming stage, volatile/persistent destination namespace, board scope, evidence class, and unknowns. Explicitly distinguish FIT internal load/entry, upload buffer, BootROM transfer memory, storage offset, GPT/MTD partition, and filesystem path.
- Required changes: include bootinfo JSON versus generated BIN, FSBL/SPL, ESOS, OpenSBI names, U-Boot versus EDK2 in an inherited `uboot` partition, bootfs/rootfs/ESP/CIDATA, StarryOS AP/RP FITs, DTB selection, build/package entry points, and a four-field unknown section for unresolved CoM260 applicability.
- Preserve: existing MS03/MS09 facts, G7, evidence grades, source metadata format, ≤500-line recommendation, Chinese-first terminology, and relative navigation.
- Forbidden: claiming any artifact was built/downloaded/validated; equating filename variants; assigning Pico-ITX/third-party partitions or DTBs to CoM260; presenting example sizes/addresses as universal; copying full existing boot/storage tables.
- Test witness: `test ! -e docs/boot/k3-image-build-and-artifacts.md` must succeed before creation; required-field and namespace searches therefore fail RED.
- GREEN condition: the new file exists, begins with compliant sources, contains all common artifact fields and namespace distinctions, marks scope/evidence/unknowns, and all relative links resolve.
- Verification: structured assertions for required artifacts/fields/namespaces and forbidden overclaims; compare every source URL to task 1.1 coverage; resolve relative links; verify line count ≤500 or return to Plan for split decision.
- Stop when: one artifact identity requires an unverified equivalence, CoM260-specific destination must be invented, source evidence contradicts existing accepted specs, or the document exceeds the split boundary without a coherent decomposition.

**Invariants**

- Documentation only; no product execution or source downloads.
- M01-M04 and accepted MS03/MS09 specs remain authoritative.
- Board variants, evidence grades, address namespaces, and artifact identities never silently collapse.
- Concurrent MS11/user changes are preserved.

**Non-goals**

No RAM boot, persistent write procedure, final navigation synchronization, global state update, Runbook, tool installation, build, device access, or media mutation.

**Acceptance**

- A1 (R1/R6/D2/D6/T1.1): every external fact used by the artifact document maps to a unique, accurately scoped coverage row; Pico-ITX and inaccessible sources are explicit.
- A2 (R1/R2/D1-D3/T1.2): the artifact document supplies producer, format, payload, consumer, destination namespace, board scope, evidence class, and unknowns, and distinguishes all address/partition namespaces.
- A3 (R1/R6/D7/T1.2): existing boot/storage/DTS uncertainty remains authoritative and no CoM260 applicability is inferred from Pico-ITX, generic K3, or fixed third-party material.
- A4 (R8/D8/T1.1-T1.2): sources are unique, cited URLs are covered, relative links resolve, and no unrelated/concurrent content is overwritten.

**Verification**

- Capture RED for absent exact source rows and absent target document.
- After changes, parse coverage URLs and compare total with unique count.
- Assert source header, artifact fields, namespaces, scope/evidence labels, required unknown fields, and forbidden claims.
- Resolve all relative links in both modified/new documents.
- Run `git diff --check` and `openspec validate establish-k3-image-flashing-boot-operations-baseline --strict`.
- Record commands, decisive output, exit codes, changed paths, and conclusions in Act Response; do not create an evidence identity mechanism.

**Gate 2 Readiness**

- Requirements coverage: PASS — RTM has no Missing or Simplified entry.
- Investigation completeness: PASS — existing boot/storage surfaces, R21/R22, current official repository scope, critical source-to-document path, and validation entry points are identified.
- Design closure: PASS — board applicability, namespaces, evidence classes, command status, safety, and document split are decided.
- Task executability: PASS — tasks identify targets, current/required behavior, tests, GREEN, preservation, forbidden scope, verification, and stop conditions.
- Iteration balance: PASS — all tasks are assigned once across four dependency-ordered, independently verifiable domains.
- Traceability: PASS — requirements, scenarios, decisions, tasks, Iteration, files, witnesses, and Acceptance are mapped.
- Verification sufficiency: PASS — checks directly observe Markdown behavior, links, uniqueness, scope labels, forbidden overclaims, diff validity, and OpenSpec validity.
- Identity-evidence prohibition: PASS — no hash, revision handshake, manifest, run identity, or tool-validation framework is planned.
- Substantive unknowns: PASS — unknown product facts remain documented unknowns; no contract choice is delegated to Act.
- Artifact consistency: PASS — proposal, spec, design, tasks, and this Cycle use the same scope and boundaries.
- Persisted Evidence: PASS — Mode `none`; results are cheap to rerun and Act Response is sufficient.
- User plan approval: PASS — user approved implementation with “批准实施” on 2026-09-14 after receiving the completed plan summary.

**Persisted Evidence**

- Mode: none
- Act Response will record decisive command output and exit codes. No result meets the necessity threshold for persisted evidence.
- Budget: not applicable; no `evidence/` directory is planned.

**Risks and Notes**

- Official source content is temporally unstable; observation dates and exact file scope are required.
- The planning-time 70-URL baseline may change during MS11 closeout. Act must use the then-current total and preserve additions rather than enforcing 70 as a handshake value.
- Repository-root accessibility does not imply exact file suitability.
- Gate 2 is complete. This Cycle is ready for `openspec-act`; this approval does not authorize Plan to perform implementation itself.

## Act Response

- Status: reported

**Implemented**

完成 T1.1-T1.2 及 Plan Review 指定的有限修复。直接读取 6 个 K3-Ubuntu-Images 精确文件 URL，在来源覆盖表中以 `non-target-board` / `supporting-source` 登记；创建 K3 镜像构建与产物关系正文，分开 K3 SDK、Pico-ITX 官方仓库和固定 revision 第三方链，建立七种地址/位置名称空间和四组未知项，并补齐官方打包脚本、R22 与固定 revision 生产者文件的直接链接。

**Changed Files and Symbols**

- `docs/reference/source-coverage.md`：覆盖总数 70 → 76；新增 README、gadget、image_flash、fastboot 与两份 partition JSON 行；保留既有 70 行、历史说明和 MS11 并行内容。
- `docs/boot/k3-image-build-and-artifacts.md`：新增 134 行；包含证据等级、七种名称空间、三条产物链、关系图、U1-U4 和使用边界；首行加入 `image_flash.py`，第三方章节链接 R22 及 5 个生产者文件。
- `openspec/changes/establish-k3-image-flashing-boot-operations-baseline/tasks.md`：T1.1、T1.2 标记完成。
- 当前 Cycle `Act Response`：写入本次实现、Review 和验证结果。

**Deviations from Plan**

非实质定位差异：R22 描述的 ESOS ITS 实际位于 `others/Rt-Async-AMP/scripts/flash/esos_k3_com260_ifx.its`，不是仓库顶层 `its/`。正文只引用构建链与产物语义，不依赖错误路径；契约、范围和 Acceptance 未改变。

**Blocker Handoff**

None.

**Blocker Resolution**

None.

**Self-Review**

- Plan compliance: PASS
- Full diff reviewed: PASS
- Important finding resolved: 1 — 已补齐 `image_flash.py`、R22 与 5 个固定 revision 生产者文件的直接链接。
- Critical findings unresolved: 0
- Important findings unresolved: 0
- Minor findings unresolved: 0

**Verification Evidence**

| 验证项 | 命令或操作 | 输出摘录 | 覆盖范围 | 结论 |
|---|---|---|---|---|
| T1.1 RED | 逐 URL `rg -F`；解析覆盖表 URL 列 | 6 个精确 URL 均 `MISSING`；`coverage_total=70 coverage_unique=70` | 来源修改前状态 | PASS（预期 RED） |
| T1.2 RED | `test ! -e docs/boot/k3-image-build-and-artifacts.md` | `artifact_doc_absent=0` | 新文档修改前状态 | PASS（预期 RED） |
| Review 修复 RED | 检查 `image_flash.py` 首行来源及 R22/5 个本地源码链接 | `RED missing ...` 共 7 项 | A1/A2/A4 来源闭环 | PASS（预期 RED） |
| 来源唯一性 | 解析覆盖表 URL 列 | `coverage_total=76 coverage_unique=76` | A1/A4、T1.1 | PASS |
| 精确来源行 | 对 6 个 K3-Ubuntu-Images URL 逐项计数 | 每个 URL 均为 `1` | A1、T1.1 | PASS |
| Review 修复 GREEN | 检查新增来源、R22/5 个本地源码链接及目标存在性 | `GREEN source_and_local_links=7/7` | A1/A2/A4 来源闭环 | PASS |
| 正文来源覆盖 | 提取正文首行 URL 并查询覆盖表 | `document_urls=9 uncovered=0` | A1/A4、T1.1-T1.2 | PASS |
| 产物字段与名称空间 | 检查 6 个公共字段、7 个名称空间、U1-U4 四字段 | 无 `MISSING` 输出 | A2/A3、T1.2 | PASS |
| 相对链接 | 解析正文相对链接并检查目标存在 | `relative_links=11 broken=0` | A4、T1.2 | PASS |
| 文档长度 | `wc -l docs/boot/k3-image-build-and-artifacts.md` | `134 docs/boot/k3-image-build-and-artifacts.md` | M02、T1.2 | PASS |
| Markdown diff | `git diff --check --` 当前 Cycle 文件 | 无输出，退出 0 | 当前 Cycle 产品与反馈变更 | PASS |
| OpenSpec | `openspec validate establish-k3-image-flashing-boot-operations-baseline --strict` | `Change 'establish-k3-image-flashing-boot-operations-baseline' is valid`，退出 0 | change 结构 | PASS |

**Persisted Evidence**

None required.

**Experience Candidates**

None — 本轮只产生可低成本重跑的 Markdown 检查，没有已实跑的高风险操作或范围外实质缺陷。

**Remaining Issues**

None within Iteration 000。U1-U4 是计划要求保留的产品未知项，不是当前 Cycle 的实施缺口。

**Commit or Diff Reference**

未提交。当前 Cycle 产品修改为 `docs/reference/source-coverage.md` 和新增 `docs/boot/k3-image-build-and-artifacts.md`；MS11 收尾及 `others/` 均未修改。

## Plan Review

- Review Result: accepted

**Findings**

None. 上轮来源闭环问题已修复；未发现阻塞 Acceptance 或非阻塞 Minor finding。

**Deviation Classification**

ACT-DEVIATION resolved：`image_flash.py`、R22 和 5 个固定 revision 生产者文件均已形成直接链接；原计划与规格保持有效。

**Acceptance Gaps**

None. A1-A4 均满足。

**Convergence**

reduced：上轮 2 组来源追溯缺口已全部关闭，剩余 0 项。

**Evidence**

- 独立检查 `docs/boot/k3-image-build-and-artifacts.md`：首行 9 个 URL 均映射到 `source-coverage.md`，覆盖表 URL 无重复；R22 和 5 个生产者链接存在且目标可达。
- 独立结构与边界扫描：公共产物字段、七种名称空间和 U1-U4 均存在；未发现把 Pico-ITX 或固定第三方事实提升为 CoM260 已验证结论。
- 采信 Act Response 的修复 RED/GREEN、11 个相对链接和 76 个唯一来源结论；复核后的覆盖范围未变化。
- `git diff --check` 无输出、退出 0；严格 OpenSpec 校验输出 `Change 'establish-k3-image-flashing-boot-operations-baseline' is valid`、退出 0。

**Follow-up Decision**

接受当前 Cycle 和 Iteration 000。既有 Iteration Map 仍适用，已展开 Iteration 001 的初始 Cycle；其详细计划保持 `draft`，等待用户审计批准后才能实施。

**Iteration Plan Update**

None.

**Next Cycle**

None.

**Next Iteration**

`../001-ram-boot/000-initial.md`
