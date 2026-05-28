# DEVPLAN — ROADMAP role + `plans/next/` directory mechanics

**Authors:** John M. Urban, Claude Code 2.1.112 (claude-opus-4-7 ; Effort: Extra High)
**Created:** 2026-04-26 EDT (drafted at end of Phase 14 Supplemental Final Overseer wrap-up second pass)
**Type:** dev-system maintenance plan — DEVLOG-routed when implemented; no
product code touched. Filename prefix `DEVPLAN-` (alternative
`DEVLOGPLAN-`) marks this as dev-system work, NOT a product phase. Such
files do not get phase numbers and are not part of the onionskin phases
or product ROADMAP. They are usually impromptu updates that often do not
even have formal pre-implementation plans (a quick DEVLOG entry can stand
alone). This particular DEVPLAN was written down because the scope is
multi-part and the user wanted the ideas preserved against context loss.
**Status:** PROPOSED — awaiting orchestrator scheduling. Lives in
`plans/next/` as a placeholder until implementation fires.

> This file is itself a dev-system planning document, not a product phase.
> It carries no phase number in its filename (per the
> `DEVPLAN-<topic>.md` convention for dev-system work) and (deliberately)
> mentions no specific destination phase number internally. When this DEVPLAN
> is implemented, it does NOT need to be promoted to `multi-agent/plans/`
> the way a product phase SOUP file would be — it can fire directly as an
> impromptu DEVLOG-routed cycle. The file in `plans/next/` is a record of
> the agreed scope, not a phase-formalization staging point.

---

## 1. Purpose

Codify the role-shift that happened to `ROADMAP.md` over the past several
phases, and codify the `plans/next/` ↔ `plans/` promotion mechanics that
emerged organically. Both are real conventions today but neither is written
down in `AGENT_CONVENTIONS.md` or the workflow files, so new agents (and
future-you-with-no-context) re-derive them imperfectly each time.

This plan is the dev-system follow-up identified as Finding D in the second
Final Overseer wrap-up audit for Phase 14 Supplemental
(`multi-agent/plans/PHASE14_SUPPLEMENTAL-AUDIT_LOG.md` § "Wrap-up audit —
second pass"). Both clarifications received from the user during that audit
session are reflected here.

## 2. Background — the two clarifications

**Clarification A (received 2026-04-26 evening, mid-audit):** ROADMAP's role
has changed. It used to be a forward-looking compass that constantly needed
re-editing as the project evolved. It is now a **retrospective, light-reading
bird's-eye echo** of what the active phase plans are doing. The phase plans
themselves (the SPEC + AUDIT_LOG + STRATEGY + FEEDBACK files in
`multi-agent/plans/` and `multi-agent/plans/archived/`) are the actual
near-term planning surface. Past versions of ROADMAP that tried to look far
ahead were "way too wiley" and required constant churn.

**Clarification B (same evening, follow-up):** The phase system *itself* is
technically what is now called "the ROADMAP" in this project. CHANGELOG
entries' `**Roadmap:**` lines that say `Phase X — Phase Y` are pointing at
the actual roadmap (the phase plans), **not** at the `ROADMAP.md` summary
file. They have been correct in spirit all along; the convention text just
hasn't caught up.

**Promotion mechanic:** Candidate future phases live as files in
`multi-agent/plans/next/`. They are intentionally early-stage and
disorganized. Phase numbers can be reassigned by renaming files (e.g., what
was `PHASE17_*.md` could become `PHASE15_*.md` if the queue is reordered).
Any candidate can be "brought onto the stage" by promoting it to
`multi-agent/plans/` — at that point real planning structure (BRAINSTORM →
SPEC → AUDIT_LOG → STRATEGY → FEEDBACK) replaces the early-stage scratchpad.

## 3. Empirical motivation (why this is not just hypothetical)

The current state of `multi-agent/plans/next/`:

```
multi-agent/plans/next/PHASE15_BRAINSTORM.md
multi-agent/plans/next/PHASE16_BRAINSTORM.md
multi-agent/plans/next/PHASE17_BRAINSTORM.md
multi-agent/plans/next/PHASES.txt
```

- `PHASE16_BRAINSTORM.md` opens with a paragraph documenting its own rename
  history (was `PHASE-TBD_SUMMIT_BRAINSTORM.md`, then renamed to
  `PHASE16_BRAINSTORM.md` after a prior Phase 16 scaffold was archived). The
  phase-number string `Phase 16` appears in the body and would need to be
  edited again on the next renumber.
- `PHASE17_BRAINSTORM.md` opens with the title `# Phase 11 Implementation
  Specification — Forward architecture`. The file was renamed without
  cleaning up the body. The body still describes itself as Phase 11 work.
  This is exactly the failure mode the proposed convention prevents.

If the queue had been kept clean of internal phase numbers, both renames
would have been atomic file-rename operations with zero content edits.

## 4. Proposed design

### 4.1. ROADMAP role codification

Add to `multi-agent/AGENT_CONVENTIONS.md § ROADMAP.md` (replacing or
expanding the current section, currently at lines 228–243):

- ROADMAP is **retrospective and light-reading**. It is not the
  forward-looking compass it once was.
- The phase system itself (the SPEC files + plan files + STRATEGY files in
  `multi-agent/plans/`) is the actual roadmap. CHANGELOG `**Roadmap:**`
  lines that point at "Phase X — Phase Y" titles are pointing at the
  phase plans, NOT at sections in `ROADMAP.md`.
- A new phase gets a `ROADMAP.md` entry **once its SPEC is fully formalized**
  (not at brainstorm stage; not at SOUP stage). The entry is a short,
  reader-friendly overview of what the SPEC plans to do.
- As priorities within that phase close, ROADMAP gets `✓ DONE (v0.x.xx)` or
  `◑ PARTIAL (v0.x.xx)` markers added.
- ROADMAP MUST NOT speculate about future phases that have not yet hit
  formalized-SPEC stage. Speculative future thinking belongs in
  `multi-agent/plans/next/` (see § 4.2).
- When a phase closes, its ROADMAP entry stays as a permanent retrospective
  record (the entry is updated to reflect final state, not deleted).

### 4.2. `plans/next/` directory mechanics + the SOUP convention

Add a new section `## Future-phase planning surfaces — `plans/next/`` to
`multi-agent/AGENT_CONVENTIONS.md` (location: after the existing
"Active phase SPEC/PLAN files" section).

**Key rules:**

1. `multi-agent/plans/next/` holds candidate future phases in their earliest,
   most disorganized form. These files are NOT the formal phase plan;
   they are the raw material from which a formal phase plan is built when
   the user is ready to bring the candidate onto the stage.
2. **No phase numbers in file contents.** The only place a phase number
   appears for a `next/` file is the **filename itself** (and even that is
   optional — see § 4.3 for the alternative thematic-naming option). The
   body must NEVER name a phase number. This makes phase-number renames
   atomic file-rename operations with zero content edits, supporting the
   user's "rename to renumber" mechanic.
3. **Audit-time check.** Any agent reading or modifying a file in
   `plans/next/` MUST check whether the contents mention a phase number
   and flag it for cleanup (or clean it up if obvious). Workflow v2's
   wrap-up audit prescription should include a `next/`-directory phase-
   number scan as part of cross-cycle drift checks.
4. **Promotion from `next/` to `plans/`** is a deliberate orchestrator
   action, not a routine cycle deliverable. The promotion is when the
   first BRAINSTORM-tier formalization happens (see § 4.3).

### 4.3. The SOUP → BRAINSTORM → SPEC progression

**Recommendation: yes, adopt the SOUP terminology.** Reasoning:

- The current overload — using `BRAINSTORM` for both the
  `multi-agent/tracking/BRAINSTORM.md` general speculative-ideas reservoir
  AND for the early-stage per-future-phase files in `plans/next/` — is
  conceptually confusing. The two play different roles: the tracking-dir
  BRAINSTORM is a long-lived idea reservoir; a per-phase BRAINSTORM is a
  formalization staging point for a specific upcoming phase.
- Calling the disorganized early-stage per-phase files SOUP captures their
  state honestly. They are primordial material, not a structured plan.
- Mirrors the existing `BRAINSTORM → SPEC` translation pattern: when
  BRAINSTORM hardens into SPEC, we don't rename the file — we author a new
  SPEC file and the BRAINSTORM gets archived. Doing the same at the
  `SOUP → BRAINSTORM` boundary is consistent.
- Creates a meaningful event point for `SOUP → BRAINSTORM` translation,
  which can naturally trigger the first entry in the new sibling FEEDBACK
  file (since FEEDBACK is the Q&A surface during SPEC formalization, and
  BRAINSTORM is where the "what should the SPEC become?" questions start
  to crystallize).

**Proposed lifecycle for a future phase:**

1. **SOUP stage** — file lives in `multi-agent/plans/next/`. Filename
   convention is `<THEME>_SOUP.md` where `<THEME>` is a vague /
   broad / lightly-evocative noun phrase (e.g., `HMM_SOUP.md`,
   `SUMMIT_SOUP.md`, `FWD-ARCH_SOUP.md`) — NOT a specific scope-anchoring
   name like `HMM_COMPLETENESS_SOUP.md` and NOT `PHASE<N>_SOUP.md`. Per
   user resolution of § 7 question 1: vague names because specific names
   pre-suppose final scope and create rename pressure when reality drifts
   from the original framing; numbered names create renumbering pressure
   when the queue reorders. Vague thematic names are stable. The `_SOUP`
   suffix is retained on every `next/` file (per § 7 question 1a) so the
   lifecycle marker stays explicit. Body contents are unstructured: rough
   ideas, open questions, links to related code, design fragments, things
   the user wants to think about later. The body NEVER mentions a phase
   number (per § 4.2 rule 2). Multiple SOUP files for unrelated future
   themes coexist freely in `next/`.
2. **Promotion → BRAINSTORM stage.** When the orchestrator decides the
   phase is the next one to formalize, an agent (likely the orchestrator's
   designated planning agent) authors a sibling `PHASE<N>_BRAINSTORM.md`
   in `multi-agent/plans/` (NOT in `next/`) using the SOUP file as raw
   material. The BRAINSTORM is more organized: themed sections, identified
   priority candidates, open questions hoisted into a `## Open questions`
   block. Phase number IS now committed for the BRAINSTORM/SPEC artifacts
   (those live at the live planning path under the assigned number). The
   original SOUP file is archived to
   `multi-agent/plans/archived/<YYYYMMDD>-PHASE<N>_<THEME>_SOUP.md` —
   the archive filename carries **both lineages**: the `PHASE<N>` prefix
   links the archived SOUP to its newly-assigned phase identity (and to
   the BRAINSTORM/SPEC siblings that now live under that number), and
   the `<THEME>` middle preserves the theme name the SOUP carried during
   its `next/` lifetime (so anyone browsing the archive can trace the
   pre-promotion identity). **The BRAINSTORM-from-SOUP step triggers
   the first entry in `PHASE<N>_FEEDBACK.md`** (the Q&A surface that
   runs alongside SPEC engineering).
3. **BRAINSTORM → SPEC stage.** Standard existing pattern: SPEC gets
   authored from BRAINSTORM, BRAINSTORM gets archived alongside SPEC at
   phase close.
4. **SPEC active.** Standard existing pattern: SPEC + AUDIT_LOG + STRATEGY
   + FEEDBACK live at `multi-agent/plans/`; ROADMAP.md gets a retrospective
   entry at SPEC formalization (see § 4.1).
5. **Phase close + archive.** Standard pattern (workflow v2 § Step 8 —
   Phase Archive).

### 4.4. Rename existing `next/` files to SOUP convention

Concrete rename mapping (per § 4.3 vague-thematic + `_SOUP` suffix):

- `multi-agent/plans/next/PHASE15_BRAINSTORM.md` →
  `multi-agent/plans/next/HMM_SOUP.md`
  (theme: HMM completeness + enrichment; current contents already focus
  on HMM. Body cleanup: replace any "Phase 15" mention with phase-
  number-free language; the file's structure can stay as-is.)
- `multi-agent/plans/next/PHASE16_BRAINSTORM.md` →
  `multi-agent/plans/next/SUMMIT_SOUP.md`
  (theme: Summit Methodology; the opening rename-history paragraph plus
  any `Phase 16` mention should be cleaned up. The substantive 6-item
  bundle stays.)
- `multi-agent/plans/next/PHASE17_BRAINSTORM.md` —
  **EVALUATE BEFORE RENAMING.** This file is the oldest of the three
  (last updated 2026-04-14) and is genuinely stale: its title still
  says "Phase 11 Implementation Specification — Forward architecture",
  its body self-describes as "the active Phase 11 plan", it contains
  the typo `"Phase 14 is the about updating cross-pipeline synthesis"`
  (Phase 14 was actually CLI cleanup, not synthesis), and many of its
  ideas (Phase 11 architecture, Phase 13 per-stage completeness) have
  been subsumed by closed phases. Implementer step:
  - **First**, read the file end-to-end and ask the user whether it's
    still meaningful or should be archived as obsolete.
  - **If KEEP:** rename to `multi-agent/plans/next/FWD-ARCH_SOUP.md`,
    rewrite title to a phase-number-free thematic header (e.g.,
    `# Forward Architecture — early-stage notes`), prune the
    Phase 11 / Phase 14 stale framing, and clean any other internal
    phase-number mentions.
  - **If ARCHIVE:** `git mv` to a phase-number-free archive filename
    such as `multi-agent/plans/archived/<YYYYMMDD>-FORWARD_ARCH_SOUP.md`
    or whatever theme name the user prefers. Do NOT preserve any prior
    `PHASE<N>_*` filename in the archive name — the phase numbers in
    `plans/next/` files were assigned somewhat arbitrarily through
    multiple renames and don't correspond to actual phase plans, so
    reusing them would create misleading historical context. Update
    `PHASES.txt` if needed.

`PHASES.txt` (the index file) stays as-is — it IS the orchestrator's
free-form scratchpad and is expected to mention phase numbers explicitly
where the orchestrator finds them useful.

### 4.5. Workflow file updates

The dev system spans **two** workflow files (the "two-cassette" model):

- `multi-agent/workflows/phase-development-system_PDS-v2.md` — the
  brainstorming + SPEC engineering "early leg" of the pipeline.
- `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md` +
  `multi-agent/workflows/orchestrator.spec_plan_three_role_audit_loop-v2.md`
  — the audit/implement/re-audit "later leg".

The SOUP convention and the SOUP → BRAINSTORM promotion mechanic primarily
belong to the **early leg** (PDS-v2), since that is where SOUP files are
born and where promotion happens. The audit-loop-v2 file gets the smaller
share of the changes.

**`phase-development-system_PDS-v2.md` (primary changes):**

- Add a section codifying the SOUP stage and the SOUP → BRAINSTORM
  promotion mechanic per § 4.3 (lifecycle stages 1 → 2). Include the
  `<THEME>_SOUP.md` filename convention, the no-internal-phase-numbers
  rule, the multiple-coexistent-SOUP-files allowance, and the explicit
  rule that **the BRAINSTORM-from-SOUP step triggers the first entry in
  the new sibling `PHASE<N>_FEEDBACK.md` file**.
- Add a **pre-promotion check**: when the orchestrator decides to
  promote a SOUP file to a per-phase BRAINSTORM, the planning agent
  scans the SOUP body for any phase-number mentions and flags them
  before authoring the BRAINSTORM. (Catches accumulated cruft.)
- Cross-reference `AGENT_CONVENTIONS.md § Future-phase planning surfaces`
  as the canonical rules home (per the rules-in-conventions /
  ops-in-workflows split).
- Find any prescription that says "update ROADMAP" with forward-looking
  intent and rewrite per § 4.1.

**`spec_plan_three_role_audit_loop-v2.md` + orchestrator-v2 (smaller
changes):**

- Find any prescription that says "update ROADMAP" with forward-looking
  intent and rewrite per § 4.1: "ROADMAP gets a retrospective entry when
  the SPEC is fully formalized; closeout adds completion markers."
- Add a **wrap-up audit-time check** to Template D: scan
  `multi-agent/plans/next/` for any internal phase-number mentions in
  SOUP-stage files and flag them. (Wrap-up audits already do
  cross-cycle drift checks; this fits the same surface.) Do **NOT** add
  the scan to Template C (per-cycle re-audit) — that operates on
  `plans/`, not `next/`, and adding the scan there would be
  out-of-scope noise.
- Cross-reference both `AGENT_CONVENTIONS.md § Future-phase planning
  surfaces` and `phase-development-system_PDS-v2.md` for the SOUP
  convention.

**Workflow v1** (`spec_plan_three_role_audit_loop-v1.md`): may be left
alone — deprecated; preserved as historical sibling; no new phases are
expected to opt into it. Document the decision to leave v1 untouched
in this DEVPLAN's implementation report for clarity. (Optional: add a
single one-line cross-reference at the top of v1 saying "for new phases,
see v2; ROADMAP/SOUP conventions live in AGENT_CONVENTIONS § ...".)

**Future-merge question (informational, not in scope):** the user has
asked whether `phase-development-system_PDS-v2.md` and
`spec_plan_three_role_audit_loop-v2.md` should eventually be merged
into a single workflow file. Recommendation: **keep them separate.**
The "cassette" framing the user invoked is well-founded: the two files
serve genuinely different stages of the dev cycle (SPEC engineering vs.
implementation oversight). Keeping them separate allows opting into a
future v3 audit-loop while keeping the same v2 PDS (or vice versa) and
keeps each file small enough to navigate. The cost of keeping them
separate is just disciplined cross-references — which is what this
DEVPLAN is adding anyway. Revisit the merge question only if the
cross-references start to feel noisy.

### 4.6. Agent-file tier-list updates

`CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `.github/copilot-instructions.md`:

- The ROADMAP tier-list entry (currently entry 3) should reflect ROADMAP's
  retrospective + light-reading role. Update wording so new agents do not
  treat ROADMAP as a forward-looking compass.
- A new tier-list entry for `multi-agent/plans/next/` may be useful (rank
  it appropriately — probably alongside or near the BRAINSTORM entry). The
  purpose: "candidate future phases in early-stage SOUP form;
  agent-readable when planning the next phase to promote."

### 4.7. Backfill ROADMAP.md retrospective entries

Add light-reading retrospective entries for each closed phase since the
ROADMAP last updated (mtime currently `Apr 13`):

- **Phase 12** — closed at **v0.12.25** (2026-04-16) — short overview of
  what landed. SPEC archived at `multi-agent/plans/archived/20260416-PHASE12_SPEC.md`
  and BRAINSTORM at `multi-agent/plans/archived/20260416-PHASE12_BRAINSTORM.md`.
- **Phase 13** — closed at **v0.13.80** (2026-04-21; final wrap-up at
  v0.13.81) — short overview. SPEC archived at
  `multi-agent/plans/archived/20260420-PHASE13_SPEC.md` (and BRAINSTORM
  at `archived/20260417-PHASE13_BRAINSTORM.md`).
- **Phase 14** — closed at **v0.14.46** (date per CHANGELOG) — short
  overview. SPEC archived at `multi-agent/plans/archived/20260422-PHASE14_SPEC.md`
  (BRAINSTORM at `archived/20260421-PHASE14_BRAINSTORM.md`,
  FEEDBACK at `archived/20260421-PHASE14_FEEDBACK.md`).
- **Phase 14 Supplemental** — closed at **v0.14.75** (2026-04-26;
  post-wrap-up remediation closeout) — short overview. SPEC archived at
  `multi-agent/plans/archived/20260426-PHASE14_SUPPLEMENTAL-SPEC.md`
  (sibling files at the same `20260426-PHASE14_SUPPLEMENTAL-*.md` pattern).

Each entry follows the new pattern (see § 4.1): light-reading, retrospective,
with `✓ DONE (v0.x.xx)` markers on each substantive priority. Implementer
should `grep` the archived SPEC files for the canonical priority list to
populate each entry's bullets — don't re-derive from CHANGELOG noise.

## 5. Scope deliverables (seven parts)

These seven are the concrete deliverables when this DEVPLAN is implemented:

1. **`AGENT_CONVENTIONS.md` § ROADMAP.md** — codify the retrospective +
   light-reading + SPEC-formalization-trigger role per § 4.1.
2. **`AGENT_CONVENTIONS.md` § Future-phase planning surfaces** — new
   section codifying `plans/next/` rules + SOUP convention + promotion
   mechanic per § 4.2 + § 4.3. Also note PHASES.txt's role as the
   orchestrator's free-form scratchpad (per § 7 question 2 user
   resolution: "stays flat; not authoritative; not auto-maintained by
   agents").
3. **`AGENT_CONVENTIONS.md` § DEVPLAN naming convention** — new short
   section codifying `DEVPLAN-<topic>.md` (or `DEVLOGPLAN-<topic>.md`)
   as the naming pattern for dev-system work plans. Such files: never
   carry phase numbers; are NOT product phases; are usually impromptu;
   may live in `plans/next/` if a formal pre-implementation plan is
   written, or may fire directly as DEVLOG-routed work without any
   pre-plan. (THIS file is the canonical example.)
4. **Rename + clean SOUP files** — execute § 4.4. Two straightforward
   renames (PHASE15 → `HMM_SOUP.md`; PHASE16 → `SUMMIT_SOUP.md`); one
   conditional rename pending user evaluation (PHASE17: archive vs.
   `FWD-ARCH_SOUP.md` rename + title rewrite). All retained files must
   pass the no-internal-phase-numbers grep at acceptance time (§ 9).
5. **Workflow file updates (both cassettes)** — per § 4.5.
   `phase-development-system_PDS-v2.md` (primary: SOUP convention +
   promotion mechanic + pre-promotion check + ROADMAP-update rewording)
   and `spec_plan_three_role_audit_loop-v2.md` + orchestrator-v2
   (smaller: ROADMAP-update rewording + `next/`-directory phase-number
   scan added to wrap-up Template D only). Both files cross-reference
   `AGENT_CONVENTIONS.md § Future-phase planning surfaces` as the
   canonical home. Workflow v1 left untouched per § 4.5.
6. **Agent-file tier-list updates** — per § 4.6.
7. **`ROADMAP.md` backfill** — per § 4.7. Light-reading retrospective
   entries for Phase 12, 13, 14, 14 Supplemental in the new pattern.

## 6. Sequencing notes

- **Best handled AFTER the post-wrap-up remediation cycle for Phase 14
  Supplemental closes** (i.e., after Findings A + B + C + Gemini-1 land at
  v0.14.75). That keeps this DEVPLAN's work cleanly separated from the
  product-side remediation.
- **Could fold into Phase 15 cold-start work** as a prerequisite step
  (since Phase 15's SOUP → BRAINSTORM promotion would benefit from having
  the new convention in place), but only if Phase 15 is genuinely the next
  thing to fire.
- **Alternative ordering:** if phase archive happens immediately after the
  v0.14.75 remediation closes, this DEVPLAN can fire concurrently with
  whatever's next (it's pure dev-system, no product code, so it doesn't
  block product cycles). Most natural: dedicate one dev-system cycle to
  this DEVPLAN, then move on.
- **Cycle structure suggestion:** single cycle, Template-I-driven STRATEGY
  → Template B implementation → Template C re-audit. Could even be
  skip-reaudit-eligible since most deliverables are mechanical (renames +
  conventions text), but the workflow + agent-file updates benefit from a
  re-audit pass.

## 7. Open design questions for the user

1. ~~**SOUP filename convention preference?**~~ **Resolved by user 2026-04-26
   late evening:** thematic-name style (Option B), with the explicit caveat
   that names should be **vague/broad/lightly-evocative**, not specific
   enough to presuppose final scope. Reasoning: `HMM_COMPLETENESS_SOUP`
   pre-supposes too much about what will be inside the file and might need
   a rename anyway; `PHASE<N>_SOUP` doesn't presuppose anything about
   contents but creates annoying phase-number renumber-bumping. Best of
   both: vague thematic names like `HMM_SOUP.md` rather than
   `HMM_COMPLETENESS_SOUP.md`. Name changes under this scheme are low
   burden and don't cascade across files (since no internal phase-number
   references per § 4.2 rule 2). When this DEVPLAN is implemented, the
   three existing PHASE15/16/17 SOUP-stage files should be renamed to
   vague-thematic forms; concrete suggestions per file are listed in
   the implementation deliverable § 4.4. **Sub-question pending user
   confirmation** (see § 7 question 1a below) about whether the `_SOUP`
   suffix is retained or dropped in favor of unsuffixed thematic names.

1a. ~~**Retain `_SOUP` suffix on thematic-named files?**~~ **Resolved by
    user 2026-04-26 late evening:** YES — retain `_SOUP` on every file in
    `plans/next/`. The earlier `SUMMIT_PHASE` example was a typo for
    `SUMMIT_SOUP`. Convention is now: vague thematic prefix +
    `_SOUP.md` suffix (e.g., `HMM_SOUP.md`, `SUMMIT_SOUP.md`,
    `FORWARD_ARCHITECTURE_SOUP.md`). Lifecycle marker is explicit in
    every filename in `plans/next/`.
2. ~~**Should the `multi-agent/plans/next/PHASES.txt` index file evolve?**~~
   **Resolved by user 2026-04-26 evening:** PHASES.txt is intentionally the
   orchestrator's basic scratch file for thinking about phases and how to
   organize them. Stays flat and free-form. NOT structured per-file. The
   filename and free-form bullet style are deliberate. AGENT_CONVENTIONS
   should note its role as "orchestrator scratchpad — not authoritative;
   not auto-maintained by agents."
3. ~~**Where does "the BRAINSTORM-from-SOUP step triggers the first FEEDBACK
   entry" rule actually live?**~~ **Resolved:** **canonical home is
   `AGENT_CONVENTIONS.md § Future-phase planning surfaces`** (rules belong
   in conventions); **operational cross-reference lives in
   `multi-agent/workflows/phase-development-system_PDS-v2.md`** at the
   SPEC-engineering-kickoff prescription point (operations belong in
   workflows). The "rules-in-conventions, ops-in-workflows" split per
   user clarification: `AGENT_CONVENTIONS` is "hard-coded" general rules;
   workflow files are interchangeable "cassettes" describing specific
   pipeline stages — keeping conventions out of workflow files preserves
   the cassette swappability.
4. ~~**What about `multi-agent/tracking/BRAINSTORM.md` vs the per-phase
   BRAINSTORM files?**~~ **Resolved by user 2026-04-26 late evening:**
   make the distinction explicit in `AGENT_CONVENTIONS.md`. The
   four-tier hierarchy:
   - **`multi-agent/tracking/BRAINSTORM.md`** — the **ultimate scratch
     pad / note pad of ideas** jotted down freely. Long-lived, general
     reservoir. Used as a **SOURCE** when authoring per-phase BRAINSTORM
     files (helps make them meatier).
   - **`multi-agent/tracking/KNOWN_ISSUES.md`** — long-lived registry
     focused on **near-term actionable known issues**, not broad
     speculative ideas. Also used as a **SOURCE** during SPEC engineering.
   - **`multi-agent/plans/<phase>/PHASE<N>_BRAINSTORM.md`** (during SPEC
     engineering) — per-phase formalization staging file. Created by
     SOURCING from `tracking/BRAINSTORM.md` + `tracking/KNOWN_ISSUES.md`
     + the phase's own SOUP file. Has structure (themed sections, open
     questions, candidate priorities). Becomes the input to the SPEC.
   - **`multi-agent/plans/next/<THEME>_SOUP.md`** — pre-BRAINSTORM
     scratchpad, owned by the orchestrator + occasional contributor.
     Promotes to a per-phase BRAINSTORM in `plans/` when the orchestrator
     decides the theme is the next thing to formalize.
   These distinctions are part of deliverable #2 (the new
   `AGENT_CONVENTIONS.md § Future-phase planning surfaces` section).

## 8. Out of scope for this DEVPLAN

- Touching anything in `multi-agent/plans/` (active SPEC files for the
  current phase). All changes here are dev-system + planning-surface only.
- Touching `multi-agent/plans/archived/` (the historical record). Past
  archived files keep their phase numbers in contents because they are
  immutable provenance.
- Renaming `multi-agent/tracking/BRAINSTORM.md` (the general reservoir).
  It keeps its name; only per-phase staging files in `plans/next/` adopt
  the SOUP terminology.
- ONIONSKIN_FULL_HANDOFF.md header refresh (Phase 14 Supplemental
  wrap-up audit Finding E). That is a separate maintenance item.
- **The conversational/pushback restoration shipped at DEVLOG v0.14.75.1**
  (commit `a981665`) is COMPLETE and out-of-scope for this DEVPLAN. That
  work added "Further clarification" sections to the Role 1 and Role 2
  descriptions in `spec_plan_three_role_audit_loop-v2.md` plus echoes in
  Templates B/C/F/G to recapture the in-situ-fix + push-back + praise
  dynamic. Deliverable #5 of THIS DEVPLAN covers ONLY the ROADMAP-related
  and SOUP-related workflow prescriptions; do NOT re-do or undo the
  v0.14.75.1 conversational additions.
- Workflow v1 (`spec_plan_three_role_audit_loop-v1.md`): may be left
  alone per § 4.5. Only an optional one-line cross-reference at the top
  for navigation.

## 9. Acceptance criteria

A subsequent agent implementing this DEVPLAN should be able to verify all
of the following before declaring it CLOSED:

- `grep -nE "Phase [0-9]" multi-agent/plans/next/*_SOUP.md` returns 0
  hits in the body of any SOUP file (PHASES.txt is excluded — it's the
  orchestrator's free-form scratchpad and may mention numbers freely).
- `multi-agent/plans/next/` contains exactly the SOUP files agreed upon
  during implementation (`HMM_SOUP.md`, `SUMMIT_SOUP.md`, plus either
  `FWD-ARCH_SOUP.md` if user opted to keep the Forward Architecture
  file OR no FWD-ARCH file at all if it was archived) plus `PHASES.txt`
  and this DEVPLAN file. No `PHASE<N>_BRAINSTORM.md` files remain.
- `multi-agent/AGENT_CONVENTIONS.md` contains:
  - new ROADMAP role section (deliverable #1);
  - new `## Future-phase planning surfaces` section (deliverable #2)
    explicitly distinguishing the four-tier hierarchy from § 7 question 4
    resolution (`tracking/BRAINSTORM` vs `tracking/KNOWN_ISSUES` vs
    per-phase BRAINSTORM in `plans/` vs SOUP in `plans/next/`);
  - new `## DEVPLAN naming convention` section (deliverable #3).
- All four agent files (`CLAUDE.md`, `AGENTS.md`, `GEMINI.md`,
  `.github/copilot-instructions.md`) have the updated ROADMAP tier-list
  entry reflecting the retrospective role + a new `plans/next/`
  tier-list entry.
- `ROADMAP.md` has retrospective entries for Phase 12 (v0.12.25), Phase 13
  (v0.13.80), Phase 14 (v0.14.46), and Phase 14 Supplemental (v0.14.75)
  in the new light-reading + `✓ DONE (v0.x.xx)`-markers format.
- `multi-agent/workflows/phase-development-system_PDS-v2.md` documents the
  SOUP convention + SOUP→BRAINSTORM promotion mechanic + the
  "BRAINSTORM-from-SOUP triggers first FEEDBACK entry" rule + the
  pre-promotion phase-number-mention scan + ROADMAP-update rewording.
- `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md` +
  `orchestrator.spec_plan_three_role_audit_loop-v2.md` have the
  ROADMAP-update rewording + the wrap-up Template D `next/`-directory
  scan addition.
- The DEVLOG v0.14.75.1 conversational-restoration content in
  `spec_plan_three_role_audit_loop-v2.md` is **untouched** (verified
  unchanged by the implementer in their report).
- A `make test` run is green (sanity check; no product code should be
  touched).

## 10. Risk + reversibility notes

- **Reversible.** All renames + content edits are git-trackable; an undo
  is `git revert` plus reverse-renames.
- **Low risk to product.** No `onionskin.py`, `onionskin_core/`, `tests/`,
  or `scripts/` files touched. Pure planning-surface + convention work.
- **Coordination risk:** if a future agent picks up Phase 15 SPEC
  engineering before this DEVPLAN is implemented, that agent will work
  under the OLD convention. Recommend the orchestrator schedule this
  DEVPLAN before any SOUP → BRAINSTORM promotion fires, OR explicitly
  flag the cross-cutting state in the Phase 15 launcher.

---

**Provenance:** This plan was drafted at the end of the Phase 14
Supplemental Final Overseer wrap-up audit second pass on 2026-04-26
evening EDT. The rationale and the user clarifications are recorded in
`multi-agent/plans/PHASE14_SUPPLEMENTAL-AUDIT_LOG.md` § "Wrap-up audit —
second pass" Finding D, and in the chat transcript leading up to that
audit. This file lives at `multi-agent/plans/next/` per its dev-system,
not-yet-scheduled status.
