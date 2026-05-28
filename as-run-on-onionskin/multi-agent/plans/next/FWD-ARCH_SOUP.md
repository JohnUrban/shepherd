# Forward Architecture — SOUP

**Stage:** SOUP (pre-BRAINSTORM scratchpad in `multi-agent/plans/next/`).
**Theme:** Forward architecture — explicit cross-pipeline synthesis layer
above pipeline-local prior outputs; capability matrix for pipeline
admissibility; downstream analysis recomputation from synthesized call
sets. Carries forward unresolved architecture-design questions from the
Phase 11 era (pipeline-local ownership work, of which Priority 11.1
landed at v0.10.32; the rest of the architectural agenda — synthesis
boundary, etc. — never made it to a formal SPEC and lives here as raw
material).
**Lifecycle:** Promotes to a per-phase BRAINSTORM in `multi-agent/plans/`
when the orchestrator brings it onto the stage and assigns a phase
number — IF the SOUP is judged still relevant at that point. See the
audit findings at the bottom of this file (added 2026-04-27) for the
current state of that judgement.

**Origin:** Created circa 2026-04-14. Has accumulated rename history
through several earlier filenames; that history is preserved by
`git log --follow` rather than narrated here, per the SOUP convention
(no self-references to past phase-numbered identities in the body). Body
cleanup during the DEVPLAN-ROADMAP-AND-NEXT-DIRECTORY-SYSTEM cycle was
limited to title + framing; internal cross-references to actually-closed
phases (Phase 10/11/13) are preserved as historical anchors per the
SOUP convention's allowance for cross-references.

> **Maintenance:** SOUP files are intentionally early-stage and
> disorganized. Update freely as ideas surface; do not introduce
> SELF-references to phase numbers (cross-references to actually-closed
> phases ARE allowed as historical anchors — see `AGENT_CONVENTIONS.md
> § Future-phase planning surfaces`).
>
> **Standing rule (carried from earlier framing):** keep `make summit`,
> `make summit-smoke`, and `make summit-baseline` as the canonical
> summit regression / baseline surface for HMM summit quality until
> the relevant work lands (this is repeated in HMM_SOUP.md too; the
> rule is shared).

---

## What this SOUP is about

Updating cross-pipeline synthesis architecture: defining the explicit
synthesis layer that sits above pipeline-local prior outputs, the
capability matrix for pipeline admissibility, and how downstream
analyses get recomputed from synthesized call sets when synthesis
exists.

---

## Core design rules

1. Pipeline-derived outputs belong under the pipeline that computes them.
2. Shared computation belongs in modules, not in shared emitted-analysis directories.
3. Controller/run-level artifacts must remain analysis-neutral.
4. No pipeline is the default, canonical, authoritative, or main pipeline.
5. Until explicit synthesis exists, each pipeline remains independent in both `01-prior/` and
   `02-posterior/`.
6. Cross-pipeline synthesis remains a separate explicit layer above pipeline-local products.
7. Completeness work must respect pipeline-local ownership rather than flattening differences
   into centralized intermediate files.
8. Any change that alters emitted paths, output schemas, or inspection-relevant
   analysis surfaces must update the affected utilities, inspectors, and regression helpers in
   the same change rather than leaving compatibility drift for later.
9. As new pipeline-local features come online, extend the relevant inspection and regression
   surfaces (for example inspectors, evaluation harnesses, and helper tests) where those
   features are now expected to be visible or testable.
10. `multistage` remains an input-data property, not the name of the growth pipeline.
11. `--pipelines all` means all three pipelines: `growth`, `per-stage`, and `hmm`.

---

## Active carry-over decisions from late Phase 10-13

- The long-term pipeline term remains `growth-model`, while `multistage` remains the data/property term.
- The retained grouped pipeline order/names are `01-hmm/`, `02-growth-model/`, `03-rcn-mean-shift/`.
- The aligned engine/module naming direction is `hmm_engine.py`, `growth_model_engine.py`, `rcn_mean_shift_engine.py` (batch engine), and `rcn_mean_shift_singlefile_engine.py` (single-file CLI), all under `onionskin_core/engines/`. The shared RCN helpers live in `onionskin_core/rcn_mean_shift_helpers.py`.
- Until explicit cross-pipeline synthesis exists, each pipeline remains independent in prior/posterior space and should produce its own posterior manifest from its own prior outputs.
- The first synthesis target is call-level synthesis across pipeline-local prior outputs; downstream analyses can then be recomputed from the synthesized call set rather than treated as the first synthesis problem.
- The capability matrix for pipeline admissibility still needs an explicit written home in the live planning surface rather than remaining only implicit in controller behavior.

---

## Non-goals

- Do not revive deprecated shared emitted-output ownership patterns.
- Do not approximate synthesis through centralized intermediate files.
- Do not treat growth as the owner of common analysis surfaces.

---

## Candidate priority — Explicit synthesis boundary

### Goal

Define the future synthesis interface above pipeline-local outputs without approximating it
through centralized intermediates.

### Scope statement

Synthesis must remain an explicit layer above completed pipeline-local outputs.

### Required outcome

- The valid prior inputs to synthesis are explicit.
- The valid posterior behaviors with and without synthesis are explicit.
- The future default synthesis behavior and the future independent-pipeline fallback behavior are
  explicit.
- Until synthesis exists, or when it is intentionally disabled, pipeline-local posterior behavior
   remains independent.
- Call-level synthesis is the first required synthesis surface.

### Implementation plan

1. Define which pipeline-local prior outputs are valid inputs to future synthesis.
2. Define which posterior behaviors remain independent when synthesis is absent.
3. Define the call-level synthesis contract before broader downstream synthesis ambitions.
4. Define the default synthesis behavior for the future synthesis-enabled state.
5. Define the independent-pipeline fallback behavior for the future `--no-synthesis` state.
6. Clarify how downstream analyses should be recomputed from synthesized call sets once the
   call-level synthesis layer exists.
7. Keep synthesis as an explicit layer above completed pipeline-local outputs.
8. Do not use centralized intermediate files as a substitute for synthesis design.

---

## Audit findings (added 2026-04-27 by Claude Code 2.1.112 / claude-opus-4-7 / Effort: Extra High)

Audited as part of the DEVPLAN-ROADMAP-AND-NEXT-DIRECTORY-SYSTEM cycle's
deliverable #4 (rename + clean SOUP files). User direction was to KEEP
this file in place and record findings here for future agents/orchestrator
to act on (or not) when the time comes to evaluate promotion.

### Is the content still meaningful?

**Yes, partially.** The forward-architecture content is the only written
place where the cross-pipeline synthesis design thinking is captured. The
specifics:

- **Synthesis layer (Candidate priority — Explicit synthesis boundary):**
  Still entirely forward-looking. No code or planning surface today
  defines the call-level synthesis contract or the
  `--no-synthesis` fallback behavior. Worth preserving as raw material.
- **Capability matrix for pipeline admissibility** (carry-over decision
  bullet 6): Still unresolved. The matrix is implicit in controller
  behavior today; never written down explicitly anywhere. This is real
  design debt and worth elevating into a future phase.
- **Core design rules 1–7 (pipeline-local ownership, no centralized
  intermediates, etc.):** Still operative as the project's architectural
  posture. Most are restatements of principles that landed in Phase 11.1
  (v0.10.32). Useful as a one-stop reference, but largely redundant with
  what's already codified in `multi-agent/AGENT_CONVENTIONS.md` +
  ONIONSKIN_FULL_HANDOFF.md. Could be pruned to just the
  not-yet-codified ones if/when this SOUP gets BRAINSTORM-ified.
- **Carry-over decisions from late Phase 10-13:** Dated — some items
  are now settled (engine module naming converged), others still apply.
  Worth re-auditing against current code reality before promotion.

### What's stale and should not survive promotion to BRAINSTORM

- The original "Phase 14 is about cross-pipeline synthesis" framing was
  wrong (Phase 14 became CLI cleanup). The corrected SOUP framing at
  the top of this file replaces it.
- The `Priority 14.1` numbered heading was renamed to "Candidate priority"
  during this audit since the file never represented an actual Phase 14
  plan and the number was misleading.
- Anything in the "Active carry-over decisions" section that has been
  settled by Phase 12, 13, or 14 work should be removed or marked
  "RESOLVED v0.X.YY" during the BRAINSTORM-ification step.

### Recommendations for future agents / orchestrator

1. **Defer promotion until after the HMM-completeness phase closes.**
   The synthesis layer's first concrete need will probably emerge from
   HMM posterior outputs landing alongside growth and RMS posteriors.
   Premature promotion risks designing the synthesis contract against
   incomplete pipeline outputs.
2. **Re-audit the carry-over decisions list against live code at promotion
   time.** Several bullets will be obsolete by then (engine module naming,
   for example, is already converged).
3. **Consider folding the synthesis design thinking into the
   HMM-completeness phase BRAINSTORM** rather than a standalone phase
   — the dependencies are tight enough that splitting them may create
   coordination friction. Decision belongs to the orchestrator at
   promotion time.
4. **Capability matrix for pipeline admissibility** is the most
   actionable single item here. It could land as a small standalone
   doc-only deliverable during a quiet cycle even before this SOUP
   gets fully promoted, since it's just writing down what already
   exists implicitly in controller behavior.
5. **Consider archiving instead of promoting** if the next phase or two
   reveals that the synthesis-layer thinking has been overtaken by
   simpler approaches that emerged organically. The audit-time check
   should ask "do we still need this?" before assuming promotion.

