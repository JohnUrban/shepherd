# Phase Development System (PDS) — v2
**Last updated:** 2026-04-23

**Revision marker:** v2 of the Phase Development System. v1 remains at
`multi-agent/workflows/phase-development-system_PDS-v1.md` for reference. Key v2
changes:

1. **Four files, not three.** During SPEC engineering stage: BRAINSTORM + FEEDBACK + SPEC.
   During audit/implement/re-audit stage: SPEC + `AUDIT_LOG.md` (new sibling). The
   SPEC stays clean as the authoritative contract; AUDIT_LOG carries round-by-round
   history.

   **CHANGELOG vs DEVLOG note (from v0.14.64):** Throughout this file and its
   companion workflow files, "the cycle-closeout LOG entry" means either
   `CHANGELOG.md` (product-scope cycles) or `multi-agent/DEVLOG.md` (dev-system-scope
   cycles), routed by primary content. Shared `v0.X.YY` version stream. See
   `AGENT_CONVENTIONS.md` for the routing rule.
2. **Substantive-priorities rule.** Priorities must justify a full audit-implement-reaudit
   cycle. Thin priorities (single rename, single help-string edit) must be bundled into
   batched cycles. See the new "Substantive priorities" section.
3. **Cycle granularity.** Adopted from `spec_plan_three_role_audit_loop-v2.md`: a cycle
   is either one substantive priority or one named batched group of thin priorities.
4. **CHANGELOG batching at cycle closeout.** Per-role-round CHANGELOG writes are
   replaced with one per-cycle entry using consolidated authorship surveyed from
   AUDIT_LOG.
5. **Phase-group SPEC structure** (optional). When a phase has many priorities,
   organize them into phase-groups in the SPEC with dependency-ordered headings (like
   Phase 14 Supplemental was reorganized into Phase 0 / 1 / 2 / 3 / 4 / 5).
6. **Compaction awareness.** If session is compacted since last re-read, treat as cold
   start and re-read required files. Re-reads stay mandatory at the designated
   checkpoints; they are cheap insurance.
7. **Role 2 may propose skip-reaudit** on mechanical cycles (orchestrator approves).
   See `spec_plan_three_role_audit_loop-v2.md` for criteria.

## What PDS is

This is the multi-agent system by which Phases are brainstormed (often from soup files), planned / spec'd,
audited, implemented, and finalized.


## How the Phase Development System Works

### Four important files (v2)

During the SPEC engineering stage:

- `multi-agent/plans/next/<THEME>_SOUP.md` : where various ideas around a theme are
  collected in an informal way by both human and agent developers — a **primordial
  soup** of ideas around a theme. Lives in `plans/next/` (not `plans/`) until promoted.
  Filename uses a vague / broad / lightly-evocative theme prefix + the `_SOUP.md`
  suffix (e.g., `HMM_SOUP.md`, `SUMMIT_SOUP.md`, `FWD-ARCH_SOUP.md`); never
  scope-anchoring (`HMM_SOUP`, not `HMM_COMPLETENESS_SOUP`) and never numbered
  (`HMM_SOUP`, not `PHASE15_SOUP`). The SOUP body **MUST NOT contain
  SELF-references to phase numbers** while in `next/` — i.e., no "this is the
  Phase X plan", no "Phase X follow-up", no rename-history narrative that names
  the SOUP's own past `PHASE<N>_*.md` filenames (rename history is preserved by
  `git log --follow` — doesn't need to be in the body). Cross-references to
  actually-closed phases ARE allowed as historical anchors. The self-reference
  ban is what makes "rename to renumber" a zero-content-edit operation. See
  `multi-agent/AGENT_CONVENTIONS.md § Future-phase planning surfaces` for the
  canonical rules + the four-tier hierarchy that distinguishes SOUP from
  per-phase BRAINSTORM staging files and from the long-lived
  `tracking/BRAINSTORM.md` reservoir.

- `multi-agent/plans/PHASE<N>_<THEME>_SOUP.md` : the **live, post-promotion**
  location of the SOUP. The same file is moved here (via `git mv`) at the
  moment the orchestrator decides to promote the theme. The phase number `<N>`
  is committed at this move. The SOUP receives a one-time **SOUP ID labeling
  pass** (every discrete entry tagged with `SOUP<N>.<idx>`) at this same moment —
  this is the **only** allowed body modification. After the labeling pass the
  SOUP body is read-only; the labeled file serves as the immutable
  source-of-truth-for-provenance during BRAINSTORM construction. The SOUP
  stays live in `plans/` (alongside the BRAINSTORM that's being authored from
  it) until the soup-to-brainstorm transfer is verified complete, at which
  point the orchestrator archives it. See § Identifier system in
  `AGENT_CONVENTIONS.md` for the SOUP ID + BRAIN ID + SPEC ID format details.

- `multi-agent/plans/PHASE<N>_BRAINSTORM.md` : authored from the labeled SOUP
  (lives in `plans/`, sibling to the SOUP during the transfer). Each entry
  receives a `BRAIN<N>.<idx>` BRAIN ID and a `Source:` field citing the SOUP IDs it covers,
  plus any other source citations (`tracking/BRAINSTORM.md` long-lived
  entries, `tracking/KNOWN_ISSUES.md` `[ISSUE:YYYY-MM-DD:N]` IDs, prior
  archived phase plans, etc.). BRAINSTORM is more organized than SOUP:
  themed sections, identified candidate priorities, open questions hoisted
  into a `## Open questions` block. Not necessarily in final-implementation
  order; that's the SPEC's job. Phase number is fully committed at this
  point.

- `multi-agent/plans/PHASE<N>_FEEDBACK.md` : where human and agent developers leave
  feedback **first on the SOUP-to-BRAINSTORM transition** (the BRAINSTORM-from-SOUP
  authoring step **triggers the first FEEDBACK entry** — that's the canonical
  starting point for FEEDBACK; typically the first entry contains agent
  open-questions, line-items-for-approval, and a transfer-status block), then
  on BRAINSTORM as it develops, and later on SPEC as it develops from
  BRAINSTORM. FEEDBACK is the audit-log analog for the engineering stage.
  FEEDBACK uses `Q<idx>` / `JQ<idx>` for question identifiers; it does not
  introduce its own provenance-IDs.

- `multi-agent/plans/PHASE<N>_SPEC.md` : the culmination of BRAINSTORM and FEEDBACK —
  the ideas mapped to the most sensible ordering and organized into
  **substantive Priorities** with `SPEC<N>.<idx>` IDs (e.g., `SPEC15.1`, `SPEC15.2`).
  Each Priority has a `Source:` / `Covers:` field citing the BRAIN IDs it covers
  (one, many, or none — Priorities authored from new SPEC-stage thinking
  cite the FEEDBACK discussions that produced them). SPEC IDs are
  reorderable during SPEC engineering; once the SPEC is locked at the start
  of the audit/implement stage, IDs freeze and inserts get the
  next-available number even if it appears "out of order" in the file's
  reading sequence.

**Promotion (SOUP → live `plans/` SOUP → BRAINSTORM):** When the
orchestrator decides a SOUP is the next thing to formalize, the agent:

1. Moves the SOUP via `git mv multi-agent/plans/next/<THEME>_SOUP.md multi-agent/plans/PHASE<N>_<THEME>_SOUP.md` (phase number `<N>` is committed here).
2. Performs the one-time SOUP ID labeling pass on the SOUP body — adds `SOUP<N>.<idx>` tags to each discrete entry. After this pass, the SOUP body is read-only forever.
3. Authors `PHASE<N>_BRAINSTORM.md` from the labeled SOUP, assigning `BRAIN<N>.<idx>` BRAIN IDs to each BRAINSTORM entry and citing SOUP IDs in `Source:` fields.
4. Seeds `PHASE<N>_FEEDBACK.md` with the first entry (open questions, line-items-for-approval, transfer-status block).
5. (Iterative) Multi-pass refinement via FEEDBACK: agent flags questions, user answers, agent does a follow-up pass updating BRAINSTORM with back-references in FEEDBACK. Repeats until transfer is faithful and complete.
6. Auditor (Agent 2) verifies the transfer per the auditor prompt below: every SOUP ID is referenced in at least one BRAIN ID's `Source:`, no silent scope drift, etc. Verdict appended to FEEDBACK.
7. **Once the auditor declares CLEAN**, the SOUP file is archived: `git mv multi-agent/plans/PHASE<N>_<THEME>_SOUP.md multi-agent/plans/archived/<YYYYMMDD>-PHASE<N>_<THEME>_SOUP.md`. The archive filename carries both lineages: `PHASE<N>` links to the phase identity, `<THEME>` preserves the SOUP's theme name from its `next/` era, `<YYYYMMDD>` is the archival date. SOUP archival happens AFTER the soup-to-brainstorm transfer is verified, not at the start of BRAINSTORM authoring — this preserves SOUP read-accessibility during the (often iterative) transfer.

After SOUP archival, the official BRAINSTORM iteration stage begins —
BRAINSTORM is open for expansion (new ideas, deeper analysis, additional
sourcing from `tracking/BRAINSTORM.md` and `tracking/KNOWN_ISSUES.md`). The
prompt for the official brainstorming-stage iteration is the next section
("after soup-to-brainstorm transition") below in this file.

When SPEC engineering closes, BRAINSTORM and FEEDBACK are archived. The SPEC stays live
going into implementation.

During the audit/implement/re-audit stage:

- `multi-agent/plans/PHASE<N>_SPEC.md` : the authoritative contract from engineering.
  Edited only for (a) priority status table updates at cycle closeout, and (b)
  user-approved scope changes. Does NOT receive audit findings or implementation reports.
- `multi-agent/plans/PHASE<N>_AUDIT_LOG.md` : NEW in v2. Sibling to SPEC. Round-by-round
  history of the audit-implement-reaudit loop. Every Role 1 audit round, Role 2
  implementation round, Role 1 re-audit or skip-reaudit closeout, Role 3 wrap-up audit,
  post-wrap-up triage / implementation / closeout — all go here under dated, role-labeled
  subsections per cycle.

When the Phase closes out, SPEC and AUDIT_LOG are archived side-by-side with the same
date prefix.


### Three main stages

**1. Brainstorm Stage:**
- In this stage, ideas are collected and generated. In short:
	- Start BRAINSTORM file
    - Start BRAINSTORM from SOUP file when a SOUP file to kick of the Phase is available
	- Have other agents give feedback in FEEDBACK
	- Update BRAINSTORM from feedback
	- Iterate over feedback and updates until it feels done
- For a given phase a `multi-agent/plans/PHASE<N>_BRAINSTORM.md` is created.
	- That file is where the aims and goals of the phase are discussed, where related
	  ideas from KNOWN_ISSUES.md and BRAINSTORM.md are collected, and where new ideas
	  are formed.
	- That file is not necessarily in most sensible order of development.
	- The focus is on completeness of ideas surrounding the aims and goals of the phase.
	- This file gets updated based on feedback added to
	  `multi-agent/plans/PHASE<N>_FEEDBACK.md`.


**2. SPEC engineering stage:**
- When the brainstorm stage is done, an agent makes an attempt to convert it into a SPEC
  in the most sensible order, accounting for dependencies and organized into
  **substantive Priorities** (see below).
- That first attempt is put in `multi-agent/plans/PHASE<N>_SPEC.md`.
- Another agent gives feedback, added to the bottom of
  `multi-agent/plans/PHASE<N>_FEEDBACK.md` with an appropriate new section title.
	- Feedback is on whether the SPEC includes everything from the brainstorm, in the
	  most sensible order, with substantive priorities that justify the audit loop.
- Agents iterate over FEEDBACK and SPEC updates until the SPEC is considered done.
- When the SPEC file is finished:
	- the BRAINSTORM file is moved to
	  `multi-agent/plans/archived/YYYYMMDD-PHASE<N>_BRAINSTORM.md`.
	- the FEEDBACK file is moved to
	  `multi-agent/plans/archived/YYYYMMDD-PHASE<N>_FEEDBACK.md`.
	- for both, the YYYYMMDD prefix is the date of the day the SPEC is finished.
- The sibling `multi-agent/plans/PHASE<N>_AUDIT_LOG.md` is created at this point (or at
  the start of the first cycle under v2) with a skeleton header:

  ```markdown
  # PHASE <N> AUDIT LOG

  Sibling to `multi-agent/plans/PHASE<N>_SPEC.md`. Captures round-by-round history of
  the audit-implement-reaudit loop for this phase under
  `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md`.

  ## Cycle: <first cycle name> — OPEN (awaiting Role 1 initial audit)
  ```

**3. SPEC Audit / Implement stage:**
- Agents carry out the SPEC using the system defined in
  `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md`.
- The SPEC stays clean. All audit findings, implementation reports, closeout judgments,
  and wrap-up audits go into `PHASE<N>_AUDIT_LOG.md`.
- One CHANGELOG/DEVLOG entry per cycle closeout with consolidated authorship (routed by primary content per the file-routing rule in `AGENT_CONVENTIONS.md`; brainstorming-stage and SPEC-engineering-stage cycles are typically dev-system → DEVLOG, while audit-implement-reaudit cycles that touch product code typically → CHANGELOG). **No CHANGELOG/DEVLOG entry is written during stage iteration — only at cycle closeouts.** **Intermediate git commits ARE welcome (and encouraged) at logical checkpoints during iteration** — the rule is about LOG-entry cadence, not commit cadence. Intermediate commits use the same `changelog-entry-<topic>.txt` repo-root scratch-file convention as closeout commits (see `AGENT_CONVENTIONS.md § Git commit convention`), but the scratch file does NOT get inserted into `CHANGELOG.md`/`DEVLOG.md` — only the cycle-closeout commit fossilizes a LOG entry. The intermediate scratch file is just a verbose commit message for git history.
- When the Phase is closed out:
	- SPEC → `multi-agent/plans/archived/YYYYMMDD-PHASE<N>_SPEC.md`
	- AUDIT_LOG → `multi-agent/plans/archived/YYYYMMDD-PHASE<N>_AUDIT_LOG.md`
	- Same date prefix on both.


## Substantive priorities (new in v2)

Each priority in a SPEC must be **substantive** — it must justify the cost of a full
audit-implement-reaudit cycle. The test: would a Role 1 audit of this priority in
isolation produce 2+ concrete findings, or require design judgment across multiple
surfaces? If yes, it is substantive. If no, it is thin — and thin priorities must be
**bundled into a batched cycle** with related work, not run individually through the
three-role loop.

### Characteristics of a substantive priority

- Multi-file implementation across 3+ surfaces.
- Design judgment (help-string quality bar, terminology harmonization, cross-pipeline
  framing, Universal vs pipeline-specific promotion).
- Runtime wiring (not just parser surface).
- A Role 1 audit of this priority would reasonably produce 2+ concrete findings.

### Characteristics of a thin priority

- Implementation is a single rename, single help-string edit, or single trivial wire.
- No ambiguity in what the final state should be.
- A Role 1 audit would produce ≤1 concrete finding (essentially just "do what the
  priority says").
- Testing surface is covered by the existing test suite without new fixtures.

### Rule

When engineering a SPEC:

- Aim for **5–10 substantive priorities per phase**, not 30 thin ones.
- If the natural atomization produces many thin priorities, **group them into phases**
  (a v2 SPEC structural convention — see "Phase-group SPEC structure" below) and treat
  each phase as one batched cycle in the implementation stage.
- "Atomize steps" language still applies to **pipeline step separation** (each new
  output-producing step is its own step in the pipeline). It does NOT mean atomize
  every priority into its smallest possible form.

### What to do with an existing SPEC that has too many thin priorities

Two options, in order of preference:

1. **Reorganize into phase-groups** (like Phase 14 Supplemental was reorganized into
   Phase 0 / 1 / 2 / 3 / 4 / 5 with implementation-dependency ordering). Each phase
   becomes one batched cycle in the audit loop. Priority identifiers stay stable.
2. **Collapse related thin priorities into single substantive priorities** with sub-items.
   This requires user approval (priority identifier changes are scope changes).

### Phase-group SPEC structure (new in v2 — optional but recommended)

When a phase has >10 priorities, organize the SPEC into phase-groups under H2 headings:

```markdown
## Phase 0 — Closed priorities

... (closed priorities)

## Phase 1 — Structural parser changes

*Implementation order: P1 → P2 → P3 → ...*
*Rationale: all structural changes land first so the CLI surface is final before
help-string polish (Phase 3) touches it.*

... (priorities in dependency order)

## Phase 2 — Audit-only deliverables

... (priorities)

## Phase 3 — Help-string polish

... (priorities)

## Phase 4 — Tooling

... (priorities)

## Phase 5 — Closeout

... (priorities)
```

Each phase becomes a natural batched cycle in the implementation stage. The cycle name
in AUDIT_LOG_FILE can literally be "Phase 1 — Structural parser changes" covering all
the priorities in that phase.


## Phase Development Rules

You are in a repo under active development by multiple agents. Assume that in the time
between each of our interactions, other agents might have already made changes to the
codebase, making your context window stale. To combat stale knowledge, do all of the
following:

**Before making any decisions:**
- re-read all files you will modify
- do not rely on prior context or memory
- assume the codebase may have changed since your last interaction

**For each file you modify:**
- show the current relevant snippet
- confirm it matches your assumptions
- then apply changes

**Do not rely on:**
- previous audits
- earlier session context
- remembered file structure

**Only trust the current file contents.**

**Before inserting into CHANGELOG.md or DEVLOG.md (only at cycle closeout — not during stage iteration):**
- locate the most recent entry
- confirm insertion point explicitly
- do not assume location from memory
- route by primary content per `AGENT_CONVENTIONS.md` (brainstorming + SPEC engineering cycles are typically DEVLOG)

**After making changes:**
- re-read the modified files
- confirm they follow conventions exactly
- check against AGENT_CONVENTIONS.md

**On cold starts**
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or
  .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.


**After finishing up:**
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or
  .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

**Before giving your wrap-up summary comments to the user:**
- ensure that you have completed all that was asked of you from the prompt and from this
  file.

**Compaction awareness (new in v2):**
- If the session has been compacted since you last read a required file, treat yourself as
  a cold start and re-read. Compaction voids the "I already read it this session"
  assumption. Required re-reads are cheap insurance against format drift and convention
  drift that costs far more to fix.

**Be thorough during brainstorming and SPEC engineering phases:**
- Identify all code that touches or is touched by changes we are proposing, and that will
  need to be updated in parallel.
- Identify CLI flags that need updates:
	- If a CLI flag was previously restricted to one pipeline, but now applies to more,
	  it needs to have a CLI flag in the universal section as well as pipeline-specific
	  flags that can override the universal.
	- help strings may need to be updated
- If a new CLI flag is needed:
	- If it will eventually apply to multiple pipelines, then there needs to be one in
	  the universal section, and pipeline-specific versions in the pipeline sections
	  that can override the universal.
	- try to put the CLI flag in the organization category that makes most sense
		- Applies to all pipelines -> Universal, with possible pipeline-specific overrides
		- APS-related -> APS section
		- Timing-related -> Timing section
		- HMM-related -> HMM section
		- Etc.
- Search through markdown files as well as code.
	- Look for overlapping ideas in KNOWN_ISSUES.md and BRAINSTORM.md
	- Include updates needed for README.md and similar documentation files

**Aim for substantive priorities (new in v2):**
- A Priority that amounts to a single flag rename, a single help-string tweak, or a
  single trivial wire is NOT substantive enough to stand alone.
- Bundle related thin work into substantive priorities OR into phase-groups that become
  batched cycles in the audit loop.
- Target: 5–10 substantive priorities per phase, not 30 thin ones.
- "Atomize steps" still applies to pipeline-step separation, not to priorities.


## Typical form of the Opening brainstorm prompt to Agent 1 from Orchestrator, soup-to-brainstorm optional
```text
Below:
- "N" and "<N>" = N, for Phase N.
- "THEME" and "<THEME>" = THEME, for <THEME>_SOUP.md

If this is a cold start, orient yourself first by reading:
- one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md)
- then AGENT_CONVENTIONS.md

We are embarking on development of Phase <N> using the Phase Development System defined here: multi-agent/workflows/phase-development-system_PDS-v2.md

You are acting as Agent 1. 

We are in the "soup-to-brainstorm" transfer stage of phase development. 
- For canonical rules on SOUP → BRAINSTORM lifecycle (filename conventions, phase-number rules, four-tier hierarchy distinguishing SOUP from tracking/BRAINSTORM.md, the SOUP ID + BRAIN ID + SPEC ID identifier system, and provenance-citation conventions), see multi-agent/AGENT_CONVENTIONS.md § Future-phase planning surfaces (especially the § Identifier system across phase-development stages sub-section).

You will be helping to construct the initial versions of or continue building:
- multi-agent/plans/PHASE<N>_BRAINSTORM.md
- multi-agent/plans/PHASE<N>_FEEDBACK.md

The initial set of ideas for Phase N are in the following SOUP file in one of these locations:
- multi-agent/plans/next/<THEME>_SOUP.md (pre-promotion location)
- multi-agent/plans/PHASE<N>_<THEME>_SOUP.md (live, post-promotion location — file is in `plans/` alongside the BRAINSTORM during transfer)

If the SOUP file is still `multi-agent/plans/next/<THEME>_SOUP.md`, your first step is to **move it** (via `git mv`) to `multi-agent/plans/PHASE<N>_<THEME>_SOUP.md`. The phase number IS committed at this point — the move signals the SOUP has been promoted out of the candidate-queue.

Once the SOUP is at its `plans/PHASE<N>_<THEME>_SOUP.md` location, perform a **one-time SOUP ID labeling pass** on the SOUP body. This is the **single allowed modification** to the SOUP body — adding `SOUP<N>.<idx>` IDs at the start of each discrete idea/entry. Format: `SOUP15.1`, `SOUP15.2`, ... sequential per file, starting at 1. After this single labeling pass is complete, the SOUP body is read-only forever (it gets archived as-is once the transfer is verified complete). No other body modifications are allowed.

Use your judgement on what counts as a discrete entry. If a multi-line scratch contains genuinely-distinct sub-ideas, you can either (a) assign one ID for the cluster and flag the ambiguity in FEEDBACK for user resolution, OR (b) escalate to the user via FEEDBACK before assigning. Splitting one scratch into multiple SOUP IDs is interpretive and requires user approval. When in doubt, prefer one ID + FEEDBACK clarification.

We are now "staging" the contents of the SOUP file by translating them into a proper set of PHASE<N>_BRAINSTORM.md and PHASE<N>_FEEDBACK.md files. The SOUP file is not typically exhaustive. It is just the starting material for these two files. The SOUP file is likely to be a mixture of entries from human and agent developers. It will be a mixture of good and bad formatting. It is not likely to be very well formatted where human developers entered scratch-pad-style ideas. The BRAINSTORM file will start out as an accurate well-formatted and better-organized reflection of the SOUP file. When that is complete, the BRAINSTORM file can be expanded in the official "brainstorming stage". The overall goal is to transfer all SOUP information to the BRAINSTORM file with all intentions and ideas preserved. This may not take a single pass. The agent may need to make a partial transfer to the BRAINSTORM file at first, and then give feedback to the user in the FEEDBACK file. The agent can provide line items for approval as well as interview questions to the user when clarification is needed or when a thin idea needs expansion (see below). The transfer process may involve an audit from a separate agent to ensure it went well.

**Each BRAINSTORM entry receives a `BRAIN<N>.<idx>` BRAIN ID** at authoring time (modification type 3 below). Format: `BRAIN15.1`, `BRAIN15.2`, ... sequential per BRAINSTORM file. Each BRAIN ID's body must include a `Source:` field citing the `SOUP<N>.<idx>` IDs it covers, plus any other source citations from `tracking/BRAINSTORM.md` long-lived entries, `tracking/KNOWN_ISSUES.md` `[ISSUE:YYYY-MM-DD:N]` IDs, or other phase plans. Examples:
- `Source: SOUP15.3, SOUP15.7`
- `Source: SOUP15.3, [ISSUE:2026-04-19:1], tracking/BRAINSTORM.md [2026-04-14] entry on HMM peak_rcn_stage`

The transfer of information to the BRAINSTORM is required to:
- preserve the SOUP body as-is after the SOUP ID labeling pass — no other edits
- ensure every SOUP ID is referenced in at least one BRAIN ID's `Source:` field by the time transfer is declared complete (mechanically verifiable)

The only modifications to ideas in SOUP during the transfer process to BRAINSTORM fall into a few types:
1. Clean-up — Cleaning up the formatting in the BRAINSTORM transcription (NOT in SOUP; SOUP body is read-only after labeling). Ensure all BRAINSTORM entries are well-formatted and follow any BRAINSTORM formatting conventions. Human developer entries from SOUP are most likely to need this. Agent entries are typically well-formatted but might need tweaks to follow BRAINSTORM-specific conventions. Moreover, if the SOUP body contains any phase-number self-references — accumulated cruft from earlier renames in `next/` — those should NOT be carried over into BRAINSTORM verbatim; the BRAINSTORM-stage file CAN have its own phase number (since the number is committed at this point), but inherited cruft from `next/` era stale numbers should be reviewed and cleaned during transcription.
2. Reorganization — Presenting the information in a more useful order in BRAINSTORM if and when possible. SOUP body ordering is preserved (read-only); only BRAINSTORM ordering is the agent's call.
3. ID assignments — Two layers, both required during this transfer:
   - **SOUP ID labeling pass** on the SOUP file (one-time, described above) — every discrete SOUP entry gets a `SOUP<N>.<idx>`.
   - **BRAIN ID assignment** in the BRAINSTORM file — every BRAINSTORM entry gets a `BRAIN<N>.<idx>` and a `Source:` field citing the SOUP IDs it covers.
4. Cross-referencing — if ideas reference each other either implicitly or explicitly, the cross-references should be made explicit by using the IDs from step 3 in BRAINSTORM entry bodies. Mutual cross-references go in both/all entries involved.
5. Expanding upon a thinly laid out idea — Some SOUP ideas, especially scratch entries from the human developer, may need more information to turn into a proper BRAINSTORM entry. The agent is not allowed to decide alone on its expansion. All agent intentions should be presented to the human developer for approval in the FEEDBACK file as "Line items for approval" or "Open questions for the user". The user updates the FEEDBACK file, and the feedback drives further updates to the BRAINSTORM file. This may require multiple iterative passes; that is expected, not a failure.
6. Splitting / merging — when a SOUP entry contains two distinct ideas that should be separate BRAINSTORM entries, or when two SOUP entries are actually fragments of one idea. Always require user approval via FEEDBACK before acting. Splits in BRAINSTORM ARE allowed — multiple BRAIN IDs may cite the same SOUP ID in their `Source:` fields, with a FEEDBACK trail documenting the split rationale.
7. Any other edits not mentioned here should be first discussed with the user either through the chat interface or FEEDBACK file. The best practice is using the FEEDBACK file for most questions/answers and decisions so all agents can gain the context of how and why decisions were made. Things like scope narrowing and scope broadening need user feedback. Scope narrowing especially needs user approval. Test all assumptions against the user's judgement. Do not silently assume you are right about things you inferred.

Do NOT:
- narrow the scope of ideas unless expressly permitted by the user
- leave any SOUP ID unreferenced in the final BRAINSTORM unless expressly permitted by the user
- make unpermitted edits to the SOUP body beyond the one-time SOUP ID labeling pass
- silently change ideas during transcription
- silently make assumptions

DO:
- develop ideas with the user
- tell the user about assumptions
- ask the user for feedback in the FEEDBACK file
- ensure your comprehension is aligned with the user

Note that this is the canonical starting point for PHASE<N>_FEEDBACK.md. The first FEEDBACK entry — typically your open questions and proposals from the transfer plus a transfer-status block (partial / complete / needs-review-then-iterate) — is created during this transfer step, not later. The transfer-status block should explicitly list which SOUP IDs have been transferred to BRAINSTORM and which (if any) are still pending.

It is entirely possible that a SOUP file is already ready to be transcribed to BRAINSTORM in one pass, especially if agents were used to help build it all along and formatted the entries well. In that case, the SOUP ID labeling pass + the one-pass BRAINSTORM transcription with `BRAIN<N>.<idx>` IDs and `Source:` citations may finish in a single session, with FEEDBACK containing only a "transfer COMPLETE; awaiting auditor review" status entry.

When the soup-to-brainstorm transfer process is verified complete (every SOUP ID is referenced in at least one BRAIN ID's `Source:`, AND any open FEEDBACK questions about transfer scope have been resolved), the SOUP file is retired to:
- multi-agent/plans/archived/<YYYYMMDD>-PHASE<N>_<THEME>_SOUP.md

That archival is performed by the orchestrator (you propose the move via `git mv` in your closeout commit block; the orchestrator runs it). DO NOT archive the SOUP yourself before the transfer is verified — premature archival breaks the read-against-source-during-transfer pattern.

In summary:
- you are Agent 1
- your job is to (a) move SOUP from `next/` to `plans/` if not already done, (b) do the one-time SOUP ID labeling pass, (c) transcribe SOUP entries to BRAINSTORM with BRAIN IDs + provenance Source: citations, (d) seed FEEDBACK with the first entry covering open questions and transfer status, (e) iterate via FEEDBACK as needed
- for any decision or point of confusion, consult the user via FEEDBACK or chat

Does that all make sense? Are you ready to go? 

If you have no questions, then proceed.
```


## Typical form of the soup-to-brainstorm AUDITOR prompt to Agent 2 from Orchestrator

Use this prompt after Agent 1 has done at least one pass at the soup-to-brainstorm transfer and reports it ready for review. The auditor (Agent 2) verifies the transfer is faithful, complete, and well-formed before the SOUP file gets archived.

```text
Below:
- "N" and "<N>" = N, for Phase N.
- "THEME" and "<THEME>" = THEME, for the SOUP file's theme prefix.

If this is a cold start, orient yourself first by reading:
- one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md)
- then AGENT_CONVENTIONS.md (especially § Future-phase planning surfaces and § Identifier system across phase-development stages)
- then multi-agent/workflows/phase-development-system_PDS-v2.md (the soup-to-brainstorm transfer prompt above + the conventions referenced therein)

We are in the "soup-to-brainstorm" transfer stage of phase development.

You are acting as Agent 2 — the AUDITOR for the soup-to-brainstorm transfer that Agent 1 has performed.

Files involved:
- multi-agent/plans/PHASE<N>_<THEME>_SOUP.md — the live, post-promotion SOUP (read-only after the SOUP ID labeling pass)
- multi-agent/plans/PHASE<N>_BRAINSTORM.md — what Agent 1 has authored from the SOUP
- multi-agent/plans/PHASE<N>_FEEDBACK.md — Agent 1's open questions, line-items-for-approval, and transfer-status notes; plus any user responses

Your job is to audit the transfer for faithfulness, completeness, and conformance to the conventions. Do NOT edit any of those three files (Agent 1 owns BRAINSTORM and FEEDBACK; SOUP is read-only). You write findings into a NEW section at the bottom of FEEDBACK with the heading `## Auditor findings — round <N>` (round number sequential; first audit pass is round 1, subsequent re-audits after Agent 1 follow-up passes are round 2, 3, etc.).

Specifically verify:

1. **SOUP ID coverage (mechanically auditable).** Every `SOUP<N>.<idx>` ID in the SOUP body must be referenced in at least one BRAINSTORM entry's `Source:` field. Use a grep-style approach: enumerate SOUP IDs from SOUP body, enumerate Source: citations from BRAINSTORM, identify any unreferenced SOUP IDs. Each unreferenced SOUP ID is potentially a missed entry — flag it explicitly. (If the user has approved certain SOUP IDs as intentionally not transferred — e.g., out-of-scope items the user wants left behind — the FEEDBACK file should document that approval; cross-check FEEDBACK for the approval before flagging.)

2. **BRAIN ID assignment.** Every BRAINSTORM entry must have a `BRAIN<N>.<idx>` BRAIN ID. Sequential, unique, no gaps unless explicitly explained. No untagged entries.

3. **Source: provenance citations.** Every BRAIN ID must have a `Source:` field. Most should cite at least one SOUP ID; some may cite tracking/BRAINSTORM.md entries, KNOWN_ISSUES IDs, or other phase plans (allowed and expected for cross-tier sourcing). A BRAIN ID with NO Source: at all is a red flag — it represents a brand-new idea introduced during transfer without source-of-truth, which suggests scope-broadening that may not have been approved.

4. **No silent scope narrowing.** Compare the spirit and breadth of SOUP entries to their BRAINSTORM transcriptions. If BRAINSTORM entries have narrower scope than their SOUP sources, check FEEDBACK for explicit user approval of that narrowing. If no approval is recorded, flag the narrowing as a finding.

5. **No silent scope broadening or new-idea-injection.** If a BRAIN ID has no Source: pointing back to SOUP, tracking-tier, or prior-phase plans, that BRAIN ID may be a new idea introduced by Agent 1. New ideas during transfer require user approval via FEEDBACK; flag any without recorded approval.

6. **Splits / merges.** If multiple BRAIN IDs cite the same SOUP ID (split) OR if one BRAIN ID's Source: lists multiple SOUP IDs (merge), check FEEDBACK for the user-approval trail documenting the split/merge rationale. Splits especially require explicit approval per the convention.

7. **SOUP body integrity (post-labeling).** Verify the SOUP body has been modified ONLY by the one-time SOUP ID labeling pass. No other body edits should have occurred. Use git history if needed (`git log -p` on the SOUP file) to confirm only the labeling pass appears in the diff.

8. **Open FEEDBACK questions.** Enumerate any open questions or line-items-for-approval that Agent 1 raised but the user has not yet answered. These must be resolved before transfer can be declared complete; flag them explicitly so the orchestrator knows what remains.

9. **Transfer-status block.** Verify FEEDBACK contains a transfer-status block from Agent 1 stating whether the transfer is partial, complete, or needs-review. If the status is "complete" but your audit finds issues from items 1-8 above, the status is wrong; flag the discrepancy.

10. **General formatting and convention compliance.** BRAINSTORM entries follow consistent formatting; phase-number self-references from the SOUP's `next/`-era are not carried over; cross-references between BRAINSTORM entries use BRAIN IDs not vague prose; the file is well-organized.

Output format for your audit findings (appended to FEEDBACK as a new `## Auditor findings — round <N>` section):

- **Verdict:** one of `CLEAN — transfer ready for archive` / `OPEN — Agent 1 follow-up pass needed` / `BLOCKED — orchestrator decision required`
- **Per-criterion findings:** brief ✓ / ✗ / ⚠ status + notes for each of the 10 verifications above
- **Specific issues:** itemized list of any findings (unreferenced SOUP IDs, missing BRAIN IDs, scope drift, etc.) with exact citations (file:line where possible, or quoted strings)
- **Recommendations:** concrete next steps for Agent 1 (or the orchestrator) to resolve any flagged issues. If verdict is CLEAN, recommend the orchestrator-driven SOUP archival via `git mv multi-agent/plans/PHASE<N>_<THEME>_SOUP.md multi-agent/plans/archived/<YYYYMMDD>-PHASE<N>_<THEME>_SOUP.md` and the start of the official brainstorming-stage iteration.

Do NOT edit BRAINSTORM, SOUP, or any non-FEEDBACK file during this audit. Even if you spot small typos or formatting issues — flag them, don't fix. Agent 1 owns those files at this stage.

In summary:
- you are Agent 2, the auditor of Agent 1's soup-to-brainstorm transfer
- you read SOUP, BRAINSTORM, and FEEDBACK
- you write a findings section at the bottom of FEEDBACK
- you do not edit any other file
- you give a verdict (CLEAN / OPEN / BLOCKED) plus per-criterion findings + recommendations

Does that make sense? Are you ready to go?

If you have no questions, then proceed.
```


## Typical form of the soup-to-brainstorm FOLLOW-UP prompt to Agent 1 from Orchestrator (post-audit or post-FEEDBACK-response next pass)

Use this prompt when Agent 1 needs to do another transfer pass — typically after the auditor (Agent 2) has flagged findings, OR after the user has answered open questions in FEEDBACK and Agent 1 needs to incorporate those answers into BRAINSTORM updates.

```text
Below:
- "N" and "<N>" = N, for Phase N.
- "THEME" and "<THEME>" = THEME, for the SOUP file's theme prefix.

If this is a cold start, orient yourself first by reading:
- one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md)
- then AGENT_CONVENTIONS.md (especially § Future-phase planning surfaces and § Identifier system across phase-development stages)
- then multi-agent/workflows/phase-development-system_PDS-v2.md (the soup-to-brainstorm prompt + auditor prompt sections above)

We are in the "soup-to-brainstorm" transfer stage of phase development. This is a FOLLOW-UP pass — you (Agent 1) have already done at least one transfer pass; new input has arrived that you need to incorporate.

You are acting as Agent 1, continuing the transfer.

Files involved:
- multi-agent/plans/PHASE<N>_<THEME>_SOUP.md — read-only post-labeling-pass (do NOT modify)
- multi-agent/plans/PHASE<N>_BRAINSTORM.md — your authoring surface
- multi-agent/plans/PHASE<N>_FEEDBACK.md — contains your prior open questions, the user's answers, and possibly auditor findings from Agent 2

Read FEEDBACK in full first. Look for:
1. **User answers** to your prior open questions / line-items-for-approval. Each answer is your green light to update the relevant BRAIN ID(s) in BRAINSTORM accordingly.
2. **Auditor findings** (from Agent 2's `## Auditor findings — round <N>` section, if present). Each flagged issue is something to address: missing SOUP ID coverage, missing BRAIN IDs, scope drift, new-idea-without-source, etc.
3. **New questions or directions** from the user that arrived since your last pass.

For each item you address, update BRAINSTORM appropriately AND update FEEDBACK with a brief note on what you did (e.g., "Q3 answered → BRAIN15.5's `Source:` updated; BRAIN15.5 expanded per user direction in Q3").

If new questions or ambiguities arise during this follow-up pass, append them to FEEDBACK as new line-items-for-approval / open questions for the next round — same convention as the first pass.

End the round by updating the FEEDBACK transfer-status block to reflect the new state: still partial (more rounds expected), or complete-pending-audit (ready for Agent 2 re-audit), or complete (if no audit is requested by the orchestrator and all findings are resolved).

Do NOT:
- modify the SOUP body (read-only after the SOUP ID labeling pass, which already happened in the first round)
- silently make changes that weren't approved in FEEDBACK or by the user directly
- archive the SOUP yourself (orchestrator owns that operation; you propose via commit block when transfer is verified complete, but that's only after re-audit clears)

DO:
- read FEEDBACK fully before touching BRAINSTORM
- address each item explicitly with a back-reference in FEEDBACK ("Q3 answered → BRAIN15.5 updated to ...")
- raise new questions in FEEDBACK as they arise during this pass

In summary:
- you are Agent 1 doing a follow-up transfer pass
- read FEEDBACK first; act on user answers and auditor findings
- update BRAINSTORM accordingly with clear back-references in FEEDBACK
- update the transfer-status block at the end
- raise new questions for the next round if needed

Does that make sense? Are you ready to go?

If you have no questions, then proceed.
```


## Typical form of the soup-to-brainstorm CLOSEOUT prompt to Orchestrator's chosen agent, used by the orchestrator

Use this prompt when the auditor has declared `CLEAN — transfer ready for archive` and the orchestrator is ready to retire the SOUP file. The closeout agent finalizes the transfer by archiving the SOUP, updating any project-context bookmarks, and signaling that the official brainstorming stage can begin.

```text
Below:
- "N" and "<N>" = N, for Phase N.
- "THEME" and "<THEME>" = THEME, for the SOUP file's theme prefix.
- "YYYYMMDD" = today's date in that format (the date of SOUP archival).

If this is a cold start, orient yourself first by reading:
- one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md)
- then AGENT_CONVENTIONS.md (especially § Future-phase planning surfaces)
- then multi-agent/workflows/phase-development-system_PDS-v2.md (the soup-to-brainstorm transfer + auditor + follow-up prompts above)

We are at the CLOSEOUT of the soup-to-brainstorm transfer stage for Phase <N>. The auditor (Agent 2) has declared the transfer CLEAN — every SOUP ID is referenced in BRAINSTORM, every BRAIN ID is well-sourced, no silent scope drift, all open FEEDBACK questions are resolved.

Your job is to:

1. **Archive the SOUP file.** Propose this `git mv` in your closeout commit block (the orchestrator runs the actual command):
   ```
   git mv multi-agent/plans/PHASE<N>_<THEME>_SOUP.md multi-agent/plans/archived/<YYYYMMDD>-PHASE<N>_<THEME>_SOUP.md
   ```

2. **Append a closeout entry to FEEDBACK** under the heading `## Soup-to-brainstorm transfer CLOSED — <YYYYMMDD>`. The entry should:
   - Confirm the auditor's CLEAN verdict and reference the audit round where it was issued.
   - State that the SOUP is being archived to `multi-agent/plans/archived/<YYYYMMDD>-PHASE<N>_<THEME>_SOUP.md`.
   - State that the official brainstorming stage now begins — BRAINSTORM is open for expansion (new ideas, deeper analysis, additional sourcing from `tracking/BRAINSTORM.md` and `tracking/KNOWN_ISSUES.md`) per the standard "Brainstorm Stage" rules in this PDS-v2 file.
   - List the BRAIN ID-count of the BRAINSTORM at closeout (e.g., "BRAINSTORM contains 23 BRAIN IDs covering all 18 SOUP IDs plus 5 cross-tier-sourced ideas") — useful baseline for tracking BRAINSTORM evolution during the official brainstorming stage.

3. **Optionally update HANDOFF.md and TASK.md** to reflect the new state: "Phase <N> soup-to-brainstorm transfer CLOSED; BRAINSTORM iteration now active." If those files have specific entries for the prior soup-to-brainstorm work, update them accordingly.

4. **Emit a closeout commit block** per the "Convention for agent-produced git commit blocks" in `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md` (or the equivalent convention applicable in this workflow file). The commit block includes: the `git mv` for SOUP archival, `git add` for FEEDBACK + HANDOFF + TASK if updated, and a `git commit` line. CHANGELOG/DEVLOG routing: this work is dev-system, so route to DEVLOG.md (one entry capturing the closeout). The DEVLOG entry follows the per-anchor `.Z` versioning rule (next available `v0.X.YY.Z` per `AGENT_CONVENTIONS.md`).

5. **Hand off to the orchestrator** with a note that the official brainstorming stage can now begin via the next prompt in this PDS-v2 file (the "after soup-to-brainstorm transition" opening brainstorm prompt).

Do NOT:
- modify BRAINSTORM during this closeout (BRAINSTORM is now stable as the soup-to-brainstorm transfer's output; further changes happen during the official brainstorming stage)
- modify the SOUP body (it has been read-only since the labeling pass; archive moves the file as-is)
- skip the FEEDBACK closeout entry (the audit trail is canonical)

DO:
- propose the `git mv` for SOUP archival in the commit block
- append the closeout FEEDBACK entry
- update HANDOFF / TASK as appropriate
- emit the closeout commit block per convention

In summary:
- propose SOUP archival (`git mv` to `archived/<YYYYMMDD>-PHASE<N>_<THEME>_SOUP.md`)
- append FEEDBACK closeout entry
- update project-context files
- emit DEVLOG-routed closeout commit block

Does that make sense? Are you ready to go?

If you have no questions, then proceed.
```



## Typical form of the Opening brainstorm prompt to Agent 1 from Orchestrator, after soup-to-brainstorm transition or when there is no soup
```text
Below "N" and "<N>" = N, for Phase N.

If this is a cold start, orient yourself first by reading:
- one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md)
- then AGENT_CONVENTIONS.md

We are embarking on development of Phase <N> using the Phase Development System defined here: multi-agent/workflows/phase-development-system_PDS-v2.md

You are acting as Agent 1. This is the first pass in the brainstorming stage, and will be helping to construct the initial versions of or continue building:
- multi-agent/plans/PHASE<N>_BRAINSTORM.md
- multi-agent/plans/PHASE<N>_FEEDBACK.md

Phase <N> will be focused on the following:
1.
2.
...
N.

My motivation for this phase is:
- ...
- ...

First I want to brainstorm for Phase <N>.
- To do so, I would like for you to do a deep dive audit of the code base to come up with ideas for the Priorities in this Phase.
- Let's also make sure to audit scripts in tests/ and scripts/ for filepaths and other things that would need to change along with the Phase <N> changes.
- In addition to auditing the code base, search BRAINSTORM.md and KNOWN_ISSUES.md for ideas, goals, and projects we have previously thought up that are in the same spirit of how we defined the overarching goals for Phase <N>.
- Can you find ideas, concepts, designs or other things that harmonize with what Phase <N> is shaping up to be?
- Do you have any new ideas to add to this BRAINSTORM?

When questions arise, you should ask me for input, but also recognize the following aims:
- **Atomize steps (pipeline-step granularity, not priority granularity):** If it is a new process to update the output, it is a new step in the pipeline.
- **No backwards cramming:** The pipeline is a chain of steps and a chain of outputs. The output from a step further down the chain goes into the output directory for that step, not to a directory upstream in the chain. If it is a question of merging outputs, merge them, but put the updated file in the later step. We can potentially have a final step directory that contains all the final results.
- **Do it the same:** If the other pipelines do it one way, the per-stage pipeline should do it the same way or follow the same logic.
- **Substantive priorities (v2):** When we eventually convert BRAINSTORM to SPEC, priorities must be substantive enough to justify a full audit-implement-reaudit cycle. Aim for 5–10 substantive priorities per phase, not 30 thin ones. Thin work gets bundled into phase-groups / batched cycles. Do not atomize priorities into single-line renames; bundle related work.

To start:
- We can first collect those ideas and create a relatively unordered / arbitrarily ordered way in multi-agent/plans/PHASE<N>_BRAINSTORM.md.
- You can add open questions for me in multi-agent/plans/PHASE<N>_FEEDBACK.md
- I will update multi-agent/plans/PHASE<N>_FEEDBACK.md with my answers
- When we are done with open questions, I will ask other agents for feedback and updates, which they will place in multi-agent/plans/PHASE<N>_FEEDBACK.md
- Then you will read their new FEEDBACK in multi-agent/plans/PHASE<N>_FEEDBACK.md and update multi-agent/plans/PHASE<N>_BRAINSTORM.md accordingly
- After we are satisfied with the BRAINSTORM for this phase, you will then take the contents of multi-agent/plans/PHASE<N>_BRAINSTORM.md and make a proper SPEC in multi-agent/plans/PHASE<N>_SPEC.md.
- Once the SPEC for this phase is made, we can begin the audit+implement volleying between you and another agent as described in multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md

Examples of how to structure files:
- examples from previous phases can be found in the archive here: multi-agent/plans/archived/

Does that sound good?

**After finishing up:**
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

**If this session has been compacted since the last read of those files, re-read them. Compaction voids the "already read" assumption.**

**Before giving your wrap-up summary comments to the user:**
- re-read multi-agent/workflows/phase-development-system_PDS-v2.md
- ensure that you have completed all that was asked of you from the prompt and from this file regarding your role in the brainstorming stage.
```


## Typical form of the follow-up opening brainstorm prompts to Agents 2+ from Orchestrator
```text
Below "N" and "<N>" = N, for Phase N.

If this is a cold start, orient yourself first by reading:
- one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md)
- then AGENT_CONVENTIONS.md

We are continuing development of Phase <N> using the Phase Development System defined here: multi-agent/workflows/phase-development-system_PDS-v2.md

You are acting as Agent 2. This is an audit and feedback session in the brainstorming stage, and you will be providing feedback in:
- multi-agent/plans/PHASE<N>_FEEDBACK.md


Phase <N> will be focused on the following:
1.
2.
...
N.

My motivation for this phase is:
- ...
- ...

First I want to brainstorm for Phase <N>.
- I have already started this process with Agent 1 (and Agent 2, ..., ).
- Agent 1 did a deep dive audit of the codebase, and through a few iterations of back and forth, we constructed multi-agent/plans/PHASE<N>_BRAINSTORM.md.
- Other agents may have already given feedback as well. If so, it will be in multi-agent/plans/PHASE<N>_FEEDBACK.md.
- The brainstorm started out arbitrarily ordered, but it is shaping up.
- I would like to continue brainstorming Phase <N> with you to get more perspective.
- To do so, I would like for you to do a deep dive audit of the code base to come up with ideas for the Priorities in this Phase.
- Let's also make sure to audit scripts in tests/ and scripts/ for filepaths and other things that would need to change along with the changes made during this Phase.
- Give feedback on what we do already have in multi-agent/plans/PHASE<N>_BRAINSTORM.md.
- Importantly, focus on things we missed or do not yet discuss in multi-agent/plans/PHASE<N>_BRAINSTORM.md.
- In addition to auditing the code base, search BRAINSTORM.md and KNOWN_ISSUES.md for ideas, goals, and projects we have previously thought up that are in the same spirit of how we defined the overarching goals for this Phase.
- Can you find ideas, concepts, designs or other things that harmonize with what Phase <N> is shaping up to be?
- Do you have any new ideas to add to this BRAINSTORM?
- Add your findings and ideas to multi-agent/plans/PHASE<N>_FEEDBACK.md
- Never delete anything from multi-agent/plans/PHASE<N>_FEEDBACK.md, only append your report to the end of it.

When questions arise, you should ask me for input, but also recognize the following aims:
**Atomize steps (pipeline-step granularity, not priority granularity):** If it is a new process to update the output, it is a new step.
**No backwards cramming:** The pipeline is a chain of steps and a chain of outputs. The output from a step further down the chain goes into the output directory for that step, not to a directory upstream in the chain. If it is a question of merging outputs, merge them, but put the updated file in the later step. We can potentially have a final step directory that contains all the final results.
**Do it the same:** If the other pipelines do it one way, the per-stage pipeline should do it the same way or follow the same logic.
**Substantive priorities (v2):** Priorities must be substantive. Bundle thin work into phase-groups / batched cycles. Aim for 5–10 substantive priorities per phase.


Whatever your findings are, add them near the bottom of multi-agent/plans/PHASE<N>_FEEDBACK.md in a section named by you, and sign off on your audit. Do not edit anything in multi-agent/plans/PHASE<N>_BRAINSTORM.md directly unless given express permission by me. Do not delete anything in any file unless given permission by me. The contributions you make will be reviewed by myself and other agents for further integration into the plans. **No CHANGELOG/DEVLOG entry should be written for brainstorming-stage iteration audits — those are intermediate work; LOG entries are written only at cycle closeouts (per § How the Phase Development System Works → "One CHANGELOG/DEVLOG entry per cycle closeout"; routed by primary content per `AGENT_CONVENTIONS.md`). However, an intermediate git commit IS welcome at the end of your audit pass — use the standard `changelog-entry-<topic>.txt` repo-root scratch-file commit-message convention, but do NOT insert into `CHANGELOG.md`/`DEVLOG.md`; the scratch file is just the commit message body. Whether to commit is the orchestrator's call; the agent proposes the commit block.** You will possibly want to update TASK.md and/or HANDOFF.md when you are done.

Does that sound good?

**After finishing up:**
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

**If this session has been compacted since the last read of those files, re-read them.**

**Before giving your wrap-up summary comments to the user:**
- re-read multi-agent/workflows/phase-development-system_PDS-v2.md
- ensure that you have completed all that was asked of you from the prompt and from this file regarding your role in the brainstorming stage.
```

## Typical form of the returning BRAINSTORM prompts to Agent 1 from Orchestrator
```text
Below "N" and "<N>" = N, for Phase N.

If this is a cold start, orient yourself first by reading:
- one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md)
- then AGENT_CONVENTIONS.md

We are continuing the development of Phase <N> using the Phase Development System defined here: multi-agent/workflows/phase-development-system_PDS-v2.md



You are acting as Agent 1, and we are still in the brainstorming stage. In this session, you will be responding to feedback found in:
- multi-agent/plans/PHASE<N>_FEEDBACK.md

And you will be incorporating feedback into:
- multi-agent/plans/PHASE<N>_BRAINSTORM.md

Other agents have reviewed multi-agent/plans/PHASE<N>_BRAINSTORM.md and left their feedback and ideas in multi-agent/plans/PHASE<N>_FEEDBACK.md. Can you review the FEEDBACK file and update the BRAINSTORM file accordingly? Please also update the FEEDBACK file with your Triage decisions based on feedback. What feedback did you accept and incoporate? What feedback did you reject and why? Add your triage section to the bottom of the FEEDBACK file.
- Never delete anything from multi-agent/plans/PHASE<N>_FEEDBACK.md, only append your report to the end of it.

**After finishing up:**
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

**If this session has been compacted since the last read of those files, re-read them.**

**Before giving your wrap-up summary comments to the user:**
- re-read multi-agent/workflows/phase-development-system_PDS-v2.md
- ensure that you have completed all that was asked of you from the prompt and from this file regarding your role in the brainstorming stage.
```

## Typical form of the returning BRAINSTORM prompts to Agent 2+ from Orchestrator
```text
Below "N" and "<N>" = 14, for Phase 14.

If this is a cold start, orient yourself first by reading:
- one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md)
- then AGENT_CONVENTIONS.md

We are continuing development of Phase <N> using the Phase Development System defined here: multi-agent/workflows/phase-development-system_PDS-v2.md

You are acting as Agent 2. This is an audit and feedback session in the brainstorming stage, and you will be providing feedback in:
- multi-agent/plans/PHASE<N>_FEEDBACK.md


Continued Brainstorming:
- I would like to continue brainstorming Phase <N> with you to get more perspective.
- To do so, I would like for you to do a deep dive audit of the code base to come up with ideas for the Priorities in this Phase.
- Let's also make sure to audit scripts in tests/ and scripts/ for filepaths and other things that would need to change along with the changes made during this Phase.
- Give feedback on what we do already have in multi-agent/plans/PHASE<N>_BRAINSTORM.md.
- Importantly, focus on things we missed or do not yet discuss in multi-agent/plans/PHASE<N>_BRAINSTORM.md.
- In addition to auditing the code base, search BRAINSTORM.md and KNOWN_ISSUES.md for ideas, goals, and projects we have previously thought up that are in the same spirit of how we defined the overarching goals for this Phase.
- Can you find ideas, concepts, designs or other things that harmonize with what Phase <N> is shaping up to be?
- Do you have any new ideas to add to this BRAINSTORM?
- Add your findings and ideas to multi-agent/plans/PHASE<N>_FEEDBACK.md.
- Never delete anything from multi-agent/plans/PHASE<N>_FEEDBACK.md, only append your report to the end of it.

When questions arise, you should ask me for input, but also recognize the following aims:
**Atomize steps (pipeline-step granularity, not priority granularity):** If it is a new process to update the output, it is a new step.
**No backwards cramming:** The pipeline is a chain of steps and a chain of outputs. The output from a step further down the chain goes into the output directory for that step, not to a directory upstream in the chain. If it is a question of merging outputs, merge them, but put the updated file in the later step. We can potentially have a final step directory that contains all the final results.
**Do it the same:** If the other pipelines do it one way, the per-stage pipeline should do it the same way or follow the same logic.
**Substantive priorities (v2):** Priorities must be substantive. Bundle thin work into phase-groups / batched cycles.

Whatever your findings are, add them near the bottom of multi-agent/plans/PHASE<N>_FEEDBACK.md in a section named by you, and sign off on your audit. Do not edit anything in multi-agent/plans/PHASE<N>_BRAINSTORM.md directly unless given express permission by me. Do not delete anything in any file unless given permission by me. The contributions you make will be reviewed by myself and other agents for further integration into the plans. **No CHANGELOG/DEVLOG entry should be written for brainstorming-stage iteration audits — those are intermediate work; LOG entries are written only at cycle closeouts (per § How the Phase Development System Works → "One CHANGELOG/DEVLOG entry per cycle closeout"; routed by primary content per `AGENT_CONVENTIONS.md`). However, an intermediate git commit IS welcome at the end of your audit pass — use the standard `changelog-entry-<topic>.txt` repo-root scratch-file commit-message convention, but do NOT insert into `CHANGELOG.md`/`DEVLOG.md`; the scratch file is just the commit message body. Whether to commit is the orchestrator's call; the agent proposes the commit block.** You will possibly want to update TASK.md and/or HANDOFF.md when you are done.

Does that sound good?

**After finishing up:**
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

**If this session has been compacted since the last read of those files, re-read them.**

**Before giving your wrap-up summary comments to the user:**
- re-read multi-agent/workflows/phase-development-system_PDS-v2.md
- ensure that you have completed all that was asked of you from the prompt and from this file regarding your role in the brainstorming stage.
```

## Typical form of opening SPEC ENGINEERING stage prompt to Agent 1 from Orchestrator
```text
Below "N" and "<N>" = N, for Phase N.

If this is a cold start, orient yourself first by reading:
- one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md)
- then AGENT_CONVENTIONS.md

We are continuing the development of Phase <N> using the Phase Development System defined here: multi-agent/workflows/phase-development-system_PDS-v2.md

You are acting as Agent 1, and we are entering the phase SPEC engineering stage. In this session, you will be:
- constructing multi-agent/plans/PHASE<N>_SPEC.md
- using the ideas in multi-agent/plans/PHASE<N>_BRAINSTORM.md
- and providing any questions or notes in multi-agent/plans/PHASE<N>_FEEDBACK.md

It is time for the first pass of spec engineering. Go ahead and create multi-agent/plans/PHASE<N>_SPEC.md from multi-agent/plans/PHASE<N>_BRAINSTORM.md.

Follow the v2 substantive-priorities rule:
- Organize the work into SUBSTANTIVE priorities that each justify a full audit-implement-reaudit cycle.
- Aim for 5–10 substantive priorities per phase, not 30 thin ones.
- If the natural set has many thin renames/help-edits, group them into phase-groups (e.g., "Phase 1 — structural parser changes", "Phase 2 — audit-only deliverables", etc.) with an implementation-dependency ordering. Each phase-group becomes one batched cycle in the implementation stage.
- The SPEC's highest-level headings may be `## Phase <M> — <theme>` with the priorities nested underneath in dependency order.
- Any dependencies for a given priority need to be in an earlier priority or an earlier phase-group.

When you are finished making the SPEC, audit it again against multi-agent/plans/PHASE<N>_BRAINSTORM.md to ensure it is complete and there are no gaps. Your audit is a time to challenge and pressure test the weaknesses present. Update the SPEC file with the results of your own audit, if any changes are still needed. When you are done, another agent will attempt to find things to add or improve, and will place that feedback at the bottom of multi-agent/plans/PHASE<N>_FEEDBACK.md. After their feedback, you will be given an opportunity to triage it and implement the ideas you agree with into multi-agent/plans/PHASE<N>_SPEC.md. Other agents will have a chance to make final judgements on multi-agent/plans/PHASE<N>_SPEC.md, and to give the final go-ahead into the subsequent "audit / implement" cycle. We are not done with SPEC engineering until I say so. We are not ready to move on to implementation until I say so.

**After finishing up:**
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

**If this session has been compacted since the last read of those files, re-read them.**

**Before giving your wrap-up summary comments to the user:**
- re-read multi-agent/workflows/phase-development-system_PDS-v2.md
- ensure that you have completed all that was asked of you from the prompt and from this file regarding your role in the brainstorming stage.
```

## Typical form of subsequent SPEC ENGINEERING stage prompt to Agent 2 from Orchestrator
```text
Below "N" and "<N>" = N, for Phase N.

If this is a cold start, orient yourself first by reading:
- one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md)
- then AGENT_CONVENTIONS.md

We are continuing the development of Phase <N> using the Phase Development System defined here: multi-agent/workflows/phase-development-system_PDS-v2.md

You are acting as Agent 2, and we have begun the phase SPEC engineering stage. In this session, you will be:
- evaluating multi-agent/plans/PHASE<N>_SPEC.md
- against multi-agent/plans/PHASE<N>_BRAINSTORM.md
- and providing any feedback, questions, or notes in multi-agent/plans/PHASE<N>_FEEDBACK.md
- Never delete anything from multi-agent/plans/PHASE<N>_FEEDBACK.md, only append your report to the end of it.

We are done with the phase <N> brainstorm stage. We have moved on the spec engineering stage and are using the brainstorm file as guide for engineering the spec file here: multi-agent/plans/PHASE<N>_SPEC.md

Agent 1 has gone through multi-agent/plans/PHASE<N>_BRAINSTORM.md and organized it into substantive priorities (v2 rule) with the goal of the most sensible order possible. Any dependencies for a given priority should be in an earlier priority or phase-group, and part of your job is to make sure that is true.

You will now audit multi-agent/plans/PHASE<N>_SPEC.md against multi-agent/plans/PHASE<N>_BRAINSTORM.md. Look for:
- gaps (missing BRAINSTORM items)
- ordering errors (dependencies that appear AFTER their dependents)
- inconsistencies
- atomization errors (thin priorities that should be bundled into substantive priorities or phase-groups — v2 rule)
- anything that can be improved

Is anything missing? Are the priorities substantive? Are they ordered correctly? Is the phase-group structure (if used) sensible? Is it complete and ready to go? Report your findings of this audit at the bottom of multi-agent/plans/PHASE<N>_FEEDBACK.md. Unless given express permission by me, do not directly edit anything in multi-agent/plans/PHASE<N>_BRAINSTORM.md or multi-agent/plans/PHASE<N>_SPEC.md, and never delete anything from multi-agent/plans/PHASE<N>_FEEDBACK.md (only add to the end of it). If the SPEC is ready to go, I will move the multi-agent/plans/PHASE<N>_BRAINSTORM.md file to the archive multi-agent/plans/archived. If there were issues, we can discuss implementing changes to the SPEC based on your FEEDBACK, or I will have Agent 1 review your feedback. We are not done with SPEC engineering until I say so. We are not ready to move on to implementation until I say so.

**After finishing up:**
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

**If this session has been compacted since the last read of those files, re-read them.**

**Before giving your wrap-up summary comments to the user:**
- re-read multi-agent/workflows/phase-development-system_PDS-v2.md
- ensure that you have completed all that was asked of you from the prompt and from this file regarding your role in the brainstorming stage.
```


## Typical form of subsequent SPEC ENGINEERING stage prompt to Agent 1 from Orchestrator
```
Below "N" and "<N>" = N, for Phase N.

If this is a cold start, orient yourself first by reading:
- one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md)
- then AGENT_CONVENTIONS.md

We are continuing the development of Phase <N> using the Phase Development System defined here: multi-agent/workflows/phase-development-system_PDS-v2.md

You are acting as Agent 1, and we are still in the phase SPEC engineering stage. In this session, you will be:
- evaluating the feedback, questions, or notes placed in multi-agent/plans/PHASE<N>_FEEDBACK.md by other agent(s) (e.g. Agent 2, Agent 3, etc)

Other agent(s) (e.g. Agent 2, Agent 3, etc) have audited multi-agent/plans/PHASE<N>_SPEC.md against multi-agent/plans/PHASE<N>_BRAINSTORM.md, and made some suggestions for improvements that are appended to the bottom of multi-agent/plans/PHASE<N>_FEEDBACK.md.

They thought these issues were most important:
1. ..
2. ...
X. ...

Please read the audits and evaluate their findings and feedback, think of how to improve them further, and implement your plan for updating `multi-agent/plans/PHASE<N>_SPEC.md`.

Audit the updated SPEC against the conventions used in the codebase and by the pipelines in general. Ensure that the SPEC conforms to expectations set by other pipelines. Ensure that everything across the entire repo that is affected by this work is anticipated in our SPEC. Is there anything across the codebase that touches or is touched by the work we are about to do that we did not include as part of the SPEC? If so, we need to update the SPEC to include instructions on how to handle each. Ask yourself, what else do I need to check for to ensure the SPEC is complete and valid and ready to go? And when you answer your own question, then check for that stuff as well. Update the spec file with the results of your own audit, if any changes are still needed. It will then might go to a follow-up audit round with another agent.

Also verify the v2 substantive-priorities rule: priorities are substantive or bundled into phase-groups. Thin priorities standing alone are a failure mode.

Ultimately, we are trying to move on to the "audit / implement" cycles between 2 agents, following instructions here: multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md. So, those rounds will catch stuff as well. Please let us know if we are ready to move on and can close out the SPEC engineering stage, and move on to the implementation-oriented stage of Phase <N>. Nebertheless, we are not done with SPEC engineering until I say so. We are not ready to move on to implementation until I say so. If the SPEC is ready to go, I will let you know, and I will move the multi-agent/plans/PHASE<N>_BRAINSTORM.md file to the archive in multi-agent/plans/archived.

**After finishing up:**
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

**If this session has been compacted since the last read of those files, re-read them.**

**Before giving your wrap-up summary comments to the user:**
- re-read multi-agent/workflows/phase-development-system_PDS-v2.md
- ensure that you have completed all that was asked of you from the prompt and from this file regarding your role in the brainstorming stage.
```

## Typical form of additional rounds of SPEC ENGINEERING stage prompts to Agent 2 from Orchestrator
```
Below "N" and "<N>" = N, for Phase N. X = X for Round X.

If this is a cold start, orient yourself first by reading:
- one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md)
- then AGENT_CONVENTIONS.md

We are continuing the development of Phase <N> using the Phase Development System defined here: multi-agent/workflows/phase-development-system_PDS-v2.md

You are acting as Agent 2, and we are still in the phase SPEC engineering stage. In this session, you will continue to:
- evaluate multi-agent/plans/PHASE<N>_SPEC.md
- against multi-agent/plans/PHASE<N>_BRAINSTORM.md
- and provide any feedback, questions, or notes in multi-agent/plans/PHASE<N>_FEEDBACK.md
- Never delete anything from multi-agent/plans/PHASE<N>_FEEDBACK.md, only append your report to the end of it.

It is time for round <X>. Agent 1 implemented fixes based on the previous audit.

You will now audit multi-agent/plans/PHASE<N>_SPEC.md against multi-agent/plans/PHASE<N>_BRAINSTORM.md. Look for gaps, errors, inconsistencies, atomization errors (thin priorities that should be bundled — v2 rule), and anything that can be improved. Is anything missing? Are the priorities substantive? Are they ordered correctly? Is it complete and ready to go?

Also audit the updated SPEC against the conventions used in the codebase and by the pipelines in general. Ensure that the SPEC conforms to expectations set by other pipelines. Ensure that everything across the entire repo that is affected by this work is anticipated in our SPEC. Is there anything across the codebase that touches or is touched by the work we are about to do that we did not include as part of the SPEC? If so, we need to update the SPEC to include instructions on how to handle each. Ask yourself, what else do I need to check for to ensure the SPEC is complete and valid and ready to go? And when you answer your own question, then check for that stuff as well. Update the spec file with the results of your own audit, if any changes are still needed.

Report your findings of this audit at the bottom of multi-agent/plans/PHASE<N>_FEEDBACK.md.

Unless given express permission by me, do not directly edit anything in multi-agent/plans/PHASE<N>_BRAINSTORM.md or multi-agent/plans/PHASE<N>_SPEC.md, and never delete anything from multi-agent/plans/PHASE<N>_FEEDBACK.md (only add to the end of it).

Ultimately, we are trying to move on to the "audit / implement" cycles between 2 agents, following instructions here: multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md. So, those rounds will catch stuff as well. Please let us know if we are ready to move on and can close out the SPEC engineering stage, and move on to the implementation-oriented stage of Phase <N>. Nebertheless, we are not done with SPEC engineering until I say so. We are not ready to move on to implementation until I say so.

If the SPEC is ready to go, I will move the multi-agent/plans/PHASE<N>_BRAINSTORM.md file to the archive in multi-agent/plans/archived. If there were issues, we can discuss implementing changes to the SPEC based on your FEEDBACK, or I will have Agent 1 review your feedback.

**After finishing up:**
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

**If this session has been compacted since the last read of those files, re-read them.**

**Before giving your wrap-up summary comments to the user:**
- re-read multi-agent/workflows/phase-development-system_PDS-v2.md
- ensure that you have completed all that was asked of you from the prompt and from this file regarding your role in the brainstorming stage.
```

## MOVING ON TO THE "AUDIT / IMPLEMENT" CYCLES

See the following files for more on how to enter this stage:
- `multi-agent/workflows/orchestrator.spec_plan_three_role_audit_loop-v2.md`
- `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md`

The example prompts below is from those files, and are here for convenience.

The first move is to have "the strategist" come up with a "Phase Plan of Attack" to come up with agent recommendations for each role in each cycle.

## Typical form of Opening Prompt to "The Strategist" before kicking off the "AUDIT / IMPLEMENT" cycles -- from Orchestrator

```text
Read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md and follow
Template I — Strategist: Phase Plan of Attack.

Target SPEC: multi-agent/plans/PHASE<N>_SPEC.md
Target audit log: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Output: multi-agent/plans/PHASE<N>_STRATEGY.md

Produce the STRATEGY.md draft in chat. Each role cell (R1, R2, R3) must list
a Primary agent AND at least one Alternative. Wait for my approval before
writing the file to disk.

After the file is written, your closing chat message must also include a
ready-to-paste Role 1 initial-audit launcher prompt (Template A format) for
cycle index 1, with the concrete phase number N filled in — per "Final chat
output — next-action handoff prompt" in Template I.
```

## Typical form of Opening Prompt for "AUDIT / IMPLEMENT" cycles to Agent 1 from Orchestrator

```text
Read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template A — Role 1 Initial Audit.

Target file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Target cycle: <cycle name — either a substantive priority like "Priority <N>.1" or a named batched group like "Phase 1 — structural parser changes">
Current round: initial audit (audit round 1)

Audit this cycle deeply against the live codebase. Determine whether it is closed or open.
If it is open, append audit findings and exact implementation instructions into the cycle section of AUDIT_LOG_FILE (NOT the SPEC),
including file references, function references, and concrete repair notes. Do NOT write to CHANGELOG.md in this round — that
happens at cycle closeout.
Read actual code rather than trusting prior reports. Include scripts/ and tests/ when relevant.

If AUDIT_LOG_FILE does not yet exist, create it with the skeleton header (see the v2 workflow file's "Notes For The User"
section for the template).

Required checkpoint reads (quality guardrail — do not skip, especially post-compaction):
- If you have not read your agent file (CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md)
  this session, or if the session has been compacted since you last read it, read it now.
- Same for AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md
- ensure that you have completed all that is expected of you from your role and template.
```
