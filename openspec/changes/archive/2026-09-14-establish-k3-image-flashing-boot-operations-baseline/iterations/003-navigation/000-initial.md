# Iteration 003 / Cycle 000: Navigation and Consistency Convergence

## Plan Context

- Status: ready
- Iteration: 003-navigation
- Cycle: 000-initial
- Cycle Type: initial
- Parent cycle: ../002-persistent-recovery/000-initial.md

**Iteration Scope**

- Change tasks: 4.1, 4.2, 4.3
- Depends on: Iteration 000, Iteration 001, Iteration 002
- Stable baseline: all three MS12 knowledge documents are reachable, and source, terminology, gap, index, and change task summaries agree.
- Verification boundary: links resolve; source URLs, primary terms, and gap IDs are unique; exact counts and boot status agree; existing MS03/MS09 and MS11 content is preserved; strict OpenSpec validation passes.
- Diagnostic boundary: `known-gaps.md`, `terminology.md`, `index.md`, the three MS12 documents, `source-coverage.md`, and change tasks.
- Deferred tasks: None

**Cycle Scope**

- Trigger: initial
- Acceptance gaps: None
- Repair items: None
- Inherited scope: R8/S6/D7-D8; accepted Iterations 000-002; T4.1-T4.3; M01-M04; existing reference/index maintenance rules.
- Excluded scope: new technical investigation, source refresh, product execution, Runbook/Issue creation, global SNAPSHOT/tasks/M/R/I updates, MS11 lifecycle work, and change archival.

**Objective**

Integrate the accepted MS12 documents into the repository's authoritative navigation, terminology, and gap surfaces without changing their technical behavior or overwriting concurrent closeout content.

**Background**

Iterations 000-002 produced an artifact map, volatile RAM/Fastboot decision path, and persistent deployment/recovery boundary. They are not yet listed in `docs/index.md`; their primary boot/deployment terms are absent from `terminology.md`; and their unresolved package/layout/tool/recovery domain is not represented as one aggregate gap.

**Investigation Facts**

- Current Baseline: `source-coverage.md` has 76 unique URL rows after Iteration 000. Iterations 001-002 introduced no new source URL; all 14 external URLs used across their two documents map to existing rows.
- Current Baseline: `known-gaps.md` contains G1-G14, with G3-G5 `partial` and the rest `open`. G7 owns default target DTS mapping; G12 owns storage-controller board mapping and runtime/recovery details. Neither owns the cross-path CoM260 image package, deployment tool, partition contract, download/recovery entry, and post-write recovery closure established by MS12.
- Current-State Evidence: add exactly one aggregate G15 for the independent MS12 deployment/recovery closure. It must reference all three MS12 documents and explicitly remain distinct from G7 and G12; it does not close or renumber existing gaps.
- Current Baseline: `terminology.md` contains 63 data rows with unique primary spellings. The MS12 documents use nine new primary terms that need stable definitions and authoritative links: `BootROM`, `FSBL`, `ESOS`, `OpenSBI`, `FIT`, `Fastboot`, `Titan`, `GPT`, and `MTD`. “整盘镜像” remains descriptive prose rather than a separate primary term.
- Current Baseline: `docs/index.md` still reports 70 source URLs, 63 terms, and G1-G14/14 gaps; its boot list contains only `com260-boot-chain.md` and `com260-image-and-dts.md`, and the boot responsibility row still ends at Iteration 001.
- Current-State Evidence: after the planned updates, the authoritative counts are 76 source URLs, 72 terminology rows, and G1-G15/15 gaps. The three MS12 documents must appear under `docs/boot/`, and the boot responsibility/status must include the MS12 artifact, RAM, and persistent baselines without rewriting MS03 facts.
- Concurrent Boundary: the worktree already contains user/MS11 changes in all three target files. Act must edit current content surgically, preserve G14 and the peripheral terms/index entries, and stop on semantic conflict rather than restoring an earlier snapshot.
- Code and Critical Path: three accepted docs → aggregate unknown ownership → G15 and summary row → nine primary term rows → index links/status/counts → cross-document uniqueness/link/count checks. There is no runtime code, state machine, external execution, or concurrency beyond shared Markdown editing.
- Existing verification entry points: RED searches for absent G15/terms/index links; gap ID/table count checks; primary term uniqueness/count checks; source URL count; index literal/count/status checks; all modified-document relative-link resolution; forbidden removal checks for G14/MS11 entries; `git diff --check`; strict OpenSpec validation.

**Implementation Guidance**

Edit from the current worktree, one shared file at a time. Add G15 after G14 and before the summary, then add its summary and source-coverage correspondence. Add the nine boot terms in dependency order near the base boot/platform terms. Finally update the index counts, boot list, boot responsibility row, and maintenance summary. Re-read each target's diff before moving to the next task.

**Behavioral Change**

Current navigation omits the completed MS12 surfaces and reports stale counts. Required behavior makes the three documents reachable, defines their new primary vocabulary once, records one non-overlapping aggregate deployment gap, and synchronizes all public counts and boot status.

**Task Contracts**

### 4.1: Converge MS12 unknowns into one aggregate gap

- Requirement/Scenario: R8 navigation consistency; S6 complete baseline; D7-D8.
- Depends on: accepted tasks 1.1-3.1.
- Targets: `docs/reference/known-gaps.md::G15, summary table, source-coverage correspondence`
- Current behavior: G1-G14 exist; G7 and G12 cover DTS and storage-controller domains, but no gap owns the cross-path image/package/tool/download/recovery closure.
- Required behavior: add exactly G15 with current evidence, forbidden inference, resolution conditions, affected topics, status record, and explicit G7/G12 responsibility separation; add one summary row and one correspondence bullet. Keep status `open` and date 2026-09-14.
- Required changes: map CoM260 official package/artifact applicability, boot/download identity, partition/media contract, Titan/tool compatibility, SD boot selection, component backup/recovery, and post-write evidence to the three MS12 documents; preserve their internal U1-U4 as detail rather than duplicating every field.
- Preserve: G1-G14 text, order, IDs, states, dates, MS11 G14 content, and all existing correspondence bullets.
- Forbidden: renumbering or closing gaps; merging G15 into G7/G12; claiming an operation was executed; adding more than one new gap.
- Test witness: `rg '^## G15\.' docs/reference/known-gaps.md` and its summary-row search fail before modification while G1-G14 count is 14.
- GREEN condition: gap headings and summary IDs both equal unique G1-G15; G15 has all four fields, `open` state, one summary row, one correspondence entry, and links to all three MS12 documents.
- Verification: parse heading/summary counts and uniqueness; assert four fields, links, date/state and G7/G12 separation; diff review proves G1-G14 unchanged.
- Stop when: MS12 unknowns cannot be separated from an existing gap without redefining its ownership, or concurrent changes have already allocated G15.

### 4.2: Add only MS12 primary terminology

- Requirement/Scenario: R8/S6/D7-D8.
- Depends on: 4.1.
- Targets: `docs/reference/terminology.md::基础术语 table`
- Current behavior: 63 unique terms; BootROM, FSBL, ESOS, OpenSBI, FIT, Fastboot, Titan, GPT, and MTD have no primary rows.
- Required behavior: add those nine terms once, with English expansion/name, restrained aliases, a usage boundary, and relative links to their authoritative MS12 sections. Result: 72 unique primary terms.
- Required changes: distinguish BootROM from U-Boot; FIT container from payload; Fastboot transport/service from persistent target; GPT from MTD; Titan tool/package flow from generic archive; FSBL/ESOS/OpenSBI roles from filenames.
- Preserve: all 63 existing rows, primary spellings, aliases, MS11 terms, headings, and usage rules.
- Forbidden: adding descriptive prose such as whole-disk image as a term; redefining existing boot/storage terms; duplicate primary spellings; adding unsupported aliases.
- Test witness: exact primary-row searches for all nine terms return zero before modification; current data-row count is 63 and unique count is 63.
- GREEN condition: all nine rows exist exactly once, resolve to authoritative sections, and total/unique primary rows equal 72.
- Verification: parse row count/unique count; inspect nine links and definitions; diff review proves existing rows unchanged.
- Stop when: a term collides with an existing primary spelling/alias or requires a new technical decision not present in accepted documents.

### 4.3: Synchronize index and final consistency

- Requirement/Scenario: R8/S6/D7-D8.
- Depends on: 4.1, 4.2.
- Targets: `docs/index.md::reference counts, docs/boot list, boot responsibility row, maintenance summary`
- Current behavior: index reports 70 URLs, 63 terms, and 14 gaps; omits three MS12 docs; boot responsibility ends at Iteration 001.
- Required behavior: report 76 URLs, 72 terms, and G1-G15/15 gaps; list all three MS12 documents under `docs/boot/`; update boot responsibility/status to include artifact, volatile RAM/Fastboot, persistent deployment/recovery, safety and unknown boundaries; preserve existing entries and current ten-topic responsibility structure.
- Required changes: add concise descriptions for the artifact, RAM, and persistent documents; synchronize both reference lines and maintenance-summary counts; verify all repository-relative links in the three target files and three MS12 documents.
- Preserve: MS03 boot entries, MS09 storage entries, MS11 peripheral entries, all other topic rows, source semantics, and global-state boundaries.
- Forbidden: changing SNAPSHOT/global tasks/M/R/I; marking actual build/boot/flash success; removing or rewriting unrelated index content; adding technical facts not already accepted.
- Test witness: exact link searches for the three MS12 docs fail; stale index literals `70 URL`, `63 术语`, and `14 gaps` are present before modification.
- GREEN condition: three links each occur once in the boot list; all count/status literals agree with 76/72/15; relative links resolve; no stale count remains; T4.1-T4.3 can be marked complete.
- Verification: parse canonical counts from source/term/gap files and compare index; assert link occurrence/status; resolve modified/MS12 document links; check unique sources/terms/gaps; inspect full change diff; run `git diff --check` and strict OpenSpec validation.
- Stop when: canonical counts disagree internally, concurrent edits change the expected totals or boot ownership, or reconciliation would overwrite user content.

**Invariants**

- Documentation only; no source refresh or product execution.
- G1-G14, 63 existing term rows, MS03/MS09 baselines, and MS11 closeout content remain intact.
- Each URL, primary term, and gap ID remains unique.
- Counts derive from canonical tables, not planning-time assumptions; any baseline change is reconciled before editing.
- No global SNAPSHOT/tasks/M/R/I or lifecycle state is modified.

**Non-goals**

No new technical facts, source observation, behavior change, build, hardware/storage action, Runbook, Issue, global status update, archive, or branch cleanup.

**Acceptance**

- A1 (R8/S6/D7/T4.1): exactly one G15 owns the independent MS12 deployment/recovery closure, includes all required fields and links, and remains distinct from G7/G12 without modifying G1-G14.
- A2 (R8/S6/D7/T4.2): exactly nine accepted MS12 primary terms are added; total and unique rows equal 72; each row points to its authoritative section.
- A3 (R8/S6/D7-D8/T4.3): the index lists all three MS12 docs once and consistently reports 76 sources, 72 terms, and 15 gaps in reference and maintenance summaries.
- A4 (R8/S6/D8/T4.1-T4.3): source URLs, terms, gaps, and links are unique/resolvable; MS03/MS09/MS11 content is preserved; no execution or CoM260 applicability claim is introduced.
- A5 (R8/S6/D8/T4.3): all change tasks and four Iterations can reach a consistent final diff with strict OpenSpec validation.

**Verification**

- Capture RED for absent G15, nine missing term rows, three absent index links, and stale index counts.
- Parse and compare source URL total/unique, term total/unique, gap heading/summary total/unique, and index literals.
- Resolve relative links in the three targets and three MS12 documents.
- Assert preservation of G14, all ten MS11 peripheral terms/entries, MS03 boot entries, and MS09 storage entries.
- Review the full change diff, run `git diff --check`, and run `openspec validate establish-k3-image-flashing-boot-operations-baseline --strict`.
- Record decisive output and exit codes in Act Response; Persisted Evidence remains unnecessary.

**Gate 2 Readiness**

- Requirements coverage: PASS — R8/S6/D7-D8 map to T4.1-T4.3, A1-A5, six product surfaces, and direct consistency checks.
- Investigation completeness: PASS — current source/term/gap counts, missing links/terms, gap ownership, shared-file concurrency, and validation entries are known.
- Design closure: PASS — G15 ownership, exact nine terms, expected canonical counts, index placement, preservation, and stop boundaries are fixed.
- Task executability: PASS — each task defines target, behavior, edits, preservation, prohibitions, RED/GREEN, verification, and stop conditions.
- Iteration balance: PASS — gap, term, and index updates depend on all three documents and together form one final consistency result.
- Traceability: PASS — requirement, scenario, decisions, tasks, Iteration, targets, witnesses, and Acceptance form a complete chain.
- Verification sufficiency: PASS — canonical parsing and link checks directly observe all counts, uniqueness, navigation, and preservation behavior.
- Identity-evidence prohibition: PASS — no manifest, revision handshake, run identity, hash chain, or validation framework is planned.
- Substantive unknowns: PASS — no product unknown must be resolved; G15 records them without changing technical claims.
- Artifact consistency: PASS — the plan follows accepted Iterations 000-002 and the existing change Map.
- Persisted Evidence: PASS — Mode `none`; outputs are short and cheap to reproduce.
- User plan approval: PASS — user approved implementation with “批准实施” on 2026-09-14 after receiving the Iteration 003 plan summary.

**Persisted Evidence**

- Mode: none
- Act Response will record canonical counts, preservation checks, links, diff, and validation results.
- Budget: not applicable.

**Risks and Notes**

- All three target files contain concurrent MS11/user changes; Act must edit their current text and stop on semantic overlap.
- Planning-time counts are execution expectations, not handshake identities. Act must derive totals from the current canonical tables before each edit.
- G15 aggregates an independent deployment/recovery closure; it must not absorb G7 target-DTS or G12 storage-controller responsibilities.
- Gate 2 is complete. This Cycle is ready for `openspec-act`; this approval does not authorize Plan to perform implementation itself.

## Act Response

- Status: reported

**Implemented**

- Added exactly one independent aggregate gap, G15, for CoM260 image-package, tool, persistent-write, and recovery closure while retaining G7 target-DTS and G12 storage-controller ownership.
- Added the nine approved MS12 primary terms with links to the documents that define their operational boundaries.
- Repaired T4.2 after Review: all nine rows now identify the authoritative MS12 sections, and the unsupported ESOS acronym expansion was replaced with the evidenced name `ESOS`.
- Added the three MS12 boot documents to the index and synchronized boot responsibility plus the 76-source, 72-term, and 15-gap summaries.
- Marked T4.1-T4.3 complete after all content and validation gates passed.

**Changed Files and Symbols**

- `docs/reference/known-gaps.md`: G15 detail, summary row, and source-coverage correspondence.
- `docs/reference/terminology.md`: BootROM, FSBL, ESOS, OpenSBI, FIT, Fastboot, Titan, GPT, and MTD rows.
- `docs/index.md`: reference counts, three boot entries, boot responsibility, and maintenance summary.
- `tasks.md`: T4.1-T4.3 completion state.

**Deviations from Plan**

- The initial RED helper misclassified the source and terminology tables; it was corrected against the canonical table structures before any completion decision or product edit depended on those counts.
- The first implementation linked terms only at document level and supplied an unsupported ESOS expansion. The current-Cycle repair followed Plan Review without widening scope; both findings are resolved.

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

- Spec compliance review: PASS — all nine term rows identify accepted MS12 sections and use evidenced terminology.
- Diff quality review: PASS — the repair changes only the nine planned rows and preserves their primary spellings and cardinality.

**Verification Evidence**

- RED: G15, all nine primary terms, and all three MS12 index links were absent; the index still contained the 70/63/14 summaries.
- Content assertions: source URL rows/unique URLs `76/76`; terminology rows/unique primary terms `72/72`; gap headings/summary rows `15/15`.
- Cardinality: each of the nine primary terms occurs exactly once; each MS12 boot link occurs exactly once in the index; G15 has exactly one heading, summary row, and correspondence entry.
- Repair RED: explicit section references `0/9`; unsupported ESOS expansion `1`.
- Repair GREEN: explicit section references `9/9`; unsupported ESOS expansion `0`; terminology rows/unique primary terms remain `72/72`.
- Relative-link check: all Markdown document links in the three edited product documents and all three MS12 boot documents resolve.
- Preservation review: the product diff only appends G15 and the nine terms, and updates the planned index literals/boot entries/responsibility; existing MS03/MS09/MS11 content is retained.
- `git diff --check`: PASS.
- `openspec validate establish-k3-image-flashing-boot-operations-baseline --type change --strict --no-interactive`: PASS (`Change 'establish-k3-image-flashing-boot-operations-baseline' is valid`).

**Persisted Evidence**

None required.

**Experience Candidates**

None.

**Remaining Issues**

- None within this Cycle. The repaired Act Response awaits Plan Review.

**Commit or Diff Reference**

- Working-tree diff; no commit created.

## Plan Review

- Review Result: accepted

**Findings**

- None blocking. The T4.2 repair adds authoritative section references to all nine term rows and removes the unsupported ESOS acronym expansion. No new Critical, Important, or Minor finding remains.

**Deviation Classification**

- ACT-DEVIATION — resolved in the current Cycle.

**Acceptance Gaps**

- None. A1-A5 pass.

**Convergence**

- reduced — the two prior T4.2 gaps are closed.

**Evidence**

- Independent parsers: source rows/unique URLs `76/76`, term rows/unique primary terms `72/72`, gap headings/unique IDs `15/15`, and gap summary rows/unique IDs `15/15`.
- Repair checks: all nine new term rows contain explicit `§` references; “Embedded SoC Operating System” occurs zero times; the ESOS English-name field uses evidenced wording.
- All relative Markdown document links resolve in the three edited product files and all three MS12 boot documents; all seven change tasks are complete.
- Full product diff review confirms G1-G14, the original 63 terms, and MS03/MS09/MS11 content remain intact; changes stay within the approved navigation scope.
- Fresh gates: `git diff --check` exited 0; strict OpenSpec validation exited 0 and reported the change valid.

**Follow-up Decision**

- Accept Iteration 003. All MS12 Iterations and tasks are complete; no further Act work is required. The change is ready for `openspec-docs-maintainer` closeout when authorized.

**Iteration Plan Update**

None.

**Next Cycle**

None.

**Next Iteration**

None.
