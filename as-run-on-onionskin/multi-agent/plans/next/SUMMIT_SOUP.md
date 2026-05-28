# Summit Methodology — SOUP

**Stage:** SOUP (pre-BRAINSTORM scratchpad in `multi-agent/plans/next/`).
**Theme:** Summit methodology cluster — bootstrap-extension of the
parabola estimator, dynamic origin-detection window, RMS pre-smoothing,
HMM summit accuracy + sliding-offset, cross-pipeline estimator audit,
optional schema rationalization.
**Lifecycle:** Promotes to a per-phase BRAINSTORM in `multi-agent/plans/`
when the orchestrator brings it onto the stage and assigns a phase number.
See `multi-agent/AGENT_CONVENTIONS.md § Future-phase planning surfaces`
and `multi-agent/workflows/phase-development-system_PDS-v2.md` for the
SOUP → BRAINSTORM → SPEC lifecycle.

**Created:** 2026-04-23 by Claude Code 2.1.104 (claude-opus-4-7), per user
direction at end of Phase 14 Supplemental SPEC engineering. Rename
history is preserved by `git log --follow` rather than narrated here, per
the SOUP convention (no self-references to past phase-numbered identities
in the body).

**Status:** SOUP-stage. Not started. Planned to land after the HMM
completeness phase.

---

## Relationship to `CROSS_PIPELINE_UNIFICATION_SOUP.md` (added 2026-04-30)

This SOUP is one specific instance of the broader cross-pipeline structural
divergence pattern captured in
`multi-agent/plans/next/CROSS_PIPELINE_UNIFICATION_SOUP.md`. Specifically, this
SOUP's Item 5 expansion (cross-pipeline summit-refinement-algorithm parity:
porting `refine_summit_parabola` to Growth + HMM; HMM `_estimate_parabola_bp`
unification; HMM per-stage parabola summit emission) sits alongside the
broader unification SOUP's Items 1-3 (HMM step-number leakage; HMM-thinner-APS-path
gaps). The two SOUPs may graduate to phases on independent timelines, or merge
into a single structural-unification phase depending on scope-sizing at
graduation time. Cross-references between the two should be maintained as
each item's status changes.

---

## Why a dedicated Summit Methodology phase

The onionskin codebase has accumulated a genuine summit-estimator cluster that spans
pipelines and touches both CLI, output schemas, and algorithmic design. During Phase 14
Supplemental scope engineering (Rounds 8–9), a CI-semantics ambiguity was surfaced that
cannot be cleanly fixed in isolation — it is one piece of a larger coherent bundle.
Rather than patch the symptom in Phase 14 Supplemental and do half-a-solution, the entire
bundle is being parked here for a dedicated pass.

**The bundle has five recognizable pieces. Solving them together is cleaner than doing
any one in isolation.**

---

## What this SOUP is about

A coherent, cross-pipeline rationalization of how onionskin estimates amplicon summits
(as proxies for re-replication origins / initiation zones), how the estimator choice is
disclosed, and how uncertainty around each estimate is reported.

The phase covers algorithm, CLI, output schema, and documentation together.

---

## Items parked here

### Item 1 — Bootstrap the parabola estimator (originally Phase 14 Supplemental Q39/JQ3)

**Background.** `_origins.tsv` currently emits `final_origin_bp` (renamed to
`final_summit_bp` in Phase 14 Supplemental 14-S28), which is the parabola_mean estimate
when `n_parabola_valid ≥ 1` and argmax_mean as fallback. The CI columns
(`origin_ci_low/high_bp`, renamed to `final_summit_low/high_bp`) have **mixed semantics**:

- When parabola wins: `final_summit_low/high_bp` = `min` / `max` across per-stage
  parabola vertices (a range, not a confidence interval).
- When argmax wins: `final_summit_low/high_bp` = bootstrap CI on argmax_mean (resampling
  replicates).

`origin_boot_sd_bp` (renamed to `argmax_mean_bootstrap_sd_bp` in 14-S28) is always
argmax-based, regardless of which estimator won. Phase 14 Supplemental renamed it
honestly to reflect that.

**Fix.** Implement parabola-specific bootstrap. Proposed structure (mirror the existing
argmax bootstrap in `refine_origin_for_call`):

1. Per-stage replicate resampling with replacement (same as current argmax bootstrap).
2. In each iteration: recompute per-stage stage-median RCN profile from resampled
   replicates → run per-stage parabola fit → aggregate valid per-stage vertices to one
   `parabola_mean_bp` per iteration.
3. Distribution of N iteration-level aggregates → bootstrap CI on parabola_mean.
4. Stages resampled together (one aggregate per iteration), not independently — same
   convention as current argmax bootstrap.

**After the fix:**
- `final_summit_low/high_bp` has clean semantics: always bootstrap CI on the winning
  estimator (parabola when valid, argmax fallback). The Phase 14 Supplemental band-aid
  help text explaining the mixed semantics can be removed.
- `argmax_mean_bootstrap_sd_bp` remains honestly named; a new `parabola_mean_bootstrap_sd_bp`
  (or equivalent) joins it.
- Or: emit `final_summit_bootstrap_sd_bp` as the SD of whichever estimator won per row.
  Decision belongs to this phase's spec engineering.

**Dependencies.** None. Can be implemented in isolation OR bundled with other items here.

**Estimated scope.** ~1–2 days of careful work for an experienced developer, including
tests and regression runs.

---

### Item 2 — Dynamic origin-detection window — **MIGRATED to Phase 15 SPEC15.4 (2026-04-29)**

**Migration provenance:** This item has been promoted into the Phase 15 SPEC as the **Bayesian Origin Detection Window (ODW) System** at `multi-agent/plans/PHASE15_SPEC.md § SPEC15.4`. The Phase 15 form covers the original IBM-C5 cross-pipeline ODW design + a 4-metric naming framework (`onset_stage` / `last_active_stage` / `max_rcn_stage` / `regression_stage`) + a parameterized statistical detector framework (3 detector modes: `hardthreshold` / `gaussian_overlap` / `bayesian_posterior`) + new dedicated `--odw-*` argparse group + cross-pipeline reach (HMM + Growth already inherit; RMS gets explicit timing wiring per SPEC15.4 deliverable 9). The original "design sketch" framing here was replaced by an explicit cross-pipeline implementation contract. **Formerly lived in `SUMMIT_SOUP.md` as Item 2; promoted at Phase 15 brainstorming Round 4 (2026-04-28) and locked into SPEC15.4 at SPEC engineering Round 1 (2026-04-28).**

**Original content (preserved for historical reference):**

**Background.** `KNOWN_ISSUES.md` `[ISSUE:2026-04-18:3]`. Also IBM-C5 in
`multi-agent/tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md`. Each amplicon should have its
own "origin-detection stage window" (from onset stage to last-summit-informative active
stage) rather than using a fixed early-stage set. Current `early-parabola-mean` RMS
selector is a coarse first pass; biologically, summit/origin refinement should be
confined to per-amplicon active window because late stages have "elongation interference"
that shifts the summit peak laterally.

**What's needed.** A design sketch identifying:
- Per-amplicon candidate signals for onset-stage detection.
- Per-amplicon candidate signals for last-active-stage detection.
- Per-pipeline differences in how this gets implemented.
- Plausible implementation order across pipelines.

**Dependencies.** Likely helps from Item 4 (HMM peak_rcn_stage + state-path signals).
Interacts with Item 1 (bootstrap should run within the active window, not across all
stages).

---

### Item 3 — Investigate RMS summit parabola pre-smoothing — **FOLDED into Phase 15 SPEC15.7 dlv 8 (2026-04-29)**

**Migration provenance:** This item has been folded into Phase 15 SPEC15.7 deliverable 8 ("Parabola-smoothing methodology audit") at `multi-agent/plans/PHASE15_SPEC.md § SPEC15.7`. The Phase 15 form covers the audit of growth's smoothing choice + comparison to RMS + HMM parabola-summit computation + alignment if growth's choice is provably better. **Formerly lived in `SUMMIT_SOUP.md` as Item 3; folded into Phase 15 SPEC at SPEC engineering Round 1 (2026-04-28) per BRAIN15.33 user note.**

**Original content (preserved for historical reference):**

**Background.** From Phase 14 Supplemental Q36 user feedback:
> "RMS does not pre-smooth RCN before its summit parabola. I believe we found that
> helped in an earlier testing round."

Growth pipeline smooths per-sample RCN via `--growth-rcn-smooth-halfwidth` before summit
parabola fitting AND APS computation. RMS does NOT smooth RCN before its
`refine_summit_parabola` call. HMM has its own RCN smoothing via `--hmm-smooth-halfwidth`
(step-5) and `--hmm-aps-smooth-halfwidth` (step-14, added in 14-S23).

**What's needed.**
- Find the prior testing round the user referenced; document the evidence.
- If RMS benefits from pre-smoothing, add `--rms-rcn-smooth-halfwidth` with analogous
  semantics.
- Consider promoting to Universal with per-pipeline overrides (would require HMM to
  opt-out to avoid double-smoothing since HMM has its own smoothing already).

**Dependencies.** Ties in with Item 5 (cross-pipeline summit harmonization). Low risk to
implement in isolation first if prior testing evidence is convincing.

---

### Item 4 — HMM summit accuracy work (peak_rcn_stage + sliding-offset sub-bin) — **MIGRATED to Phase 15 SPEC15.7 (2026-04-29)**

**Migration provenance:** This item has been promoted into Phase 15 SPEC at `multi-agent/plans/PHASE15_SPEC.md § SPEC15.7`. The Phase 15 form covers BOTH sub-pieces (`peak_rcn_stage` → renamed to `max_rcn_stage` per design-decisions rule 2; sliding-offset sub-bin port for HMM at SPEC15.7 deliverable 3) PLUS substantial expansion: per-amplicon stage-selection strategy menu (5 cross-pipeline strategies + 1 HMM-only `odw_narrowest_state`); ODW-confined refinement cross-pipeline; cross-pipeline summit-algorithm parity considerations. **Formerly lived in `SUMMIT_SOUP.md` as Item 4; promoted at Phase 15 brainstorming Round 5 (2026-04-28) and locked into SPEC15.7 at SPEC engineering Round 1 (2026-04-28); cross-pipeline reframing applied at Round 3 (2026-04-29).**

**Original content (preserved for historical reference):**

**Background.** IBM-C4 in `multi-agent/tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md`.
BRAINSTORM.md `[2026-04-14]` entries:
- "HMM summit estimation — use stage of maximum summit RCN (`peak_rcn_stage`)": stages
  2–4 give better origin localization than later stages because elongation interference
  shifts the summit peak laterally in later stages. Adds `peak_rcn_stage` column to
  `hmm_summit_estimates.tsv`.
- "HMM summit refinement — sliding-offset sub-bin localization": port growth's
  `sliding_offset_profile` / `refine_origin_sliding_offset` approach (v0.5.54–56) to HMM.

**Dependency.** Both require per-sample per-stage step-5 outputs, which depends on IBM-C2
(HMM parallel child pipeline). IBM-C2 is a future-phase priority (the HMM-completeness SOUP); Item 4 here rides after it.

**Alternative ordering:** if HMM parallel child pipeline landed in the HMM-completeness phase, Item 4
here can land immediately in Summit Methodology.

---

### Item 5 — Cross-pipeline summit estimator audit and unification

**Background.** Five recognizable estimator families exist across pipelines:

1. **Argmax** — bin-center of peak-RCN bin (growth, RMS).
2. **Parabola** — sub-bin parabola fit at the peak (growth via
   `refine_origin_for_call`, RMS via `refine_summit_parabola`).
3. **Sliding-offset sub-bin** — `refine_origin_sliding_offset`, v0.5.54–56. Growth
   currently; proposed for HMM (Item 4).
4. **RMS selector policies** — `--rms-summit-policy` with options like
   `early-parabola-mean`. A composition layer over the underlying methods.
5. **HMM state-path-based summit** — `_step9_summit_states` / `_step10_summit_bins` in
   `hmm_engine.py`. Different mechanism entirely — extracted from the Viterbi/posterior
   state sequence.

**What's needed.**
- Audit every estimator, its scope, its current output columns, its known strengths/
  weaknesses.
- Decide which remain, which are unified, which are retired.
- Cross-pipeline harmonization of output column names where the underlying concept is
  the same.
- Deprecation of any that do not survive.

**Dependencies.** Interacts with Items 1 (bootstrap), 2 (dynamic window), 3 (pre-smoothing),
4 (HMM improvements). This item is the "integration layer" for the others.

#### Item 5 expansion (2026-04-29) — corrected cross-pipeline summit-algorithm landscape from Phase 15 code dive

A detailed per-pipeline code-dive verification was performed during Phase 15 SPEC engineering Round 3 (post-commit correction 2026-04-29) at user request. The result: the cross-pipeline summit-algorithm landscape has gaps in BOTH directions (RMS has algorithms Growth + HMM lack; HMM lacks algorithms Growth + RMS have). This expansion captures the actual landscape so Item 5's audit + unification work has a concrete starting point when this SOUP eventually graduates to BRAINSTORM.

**Cross-pipeline summit-refinement-algorithm landscape (verified 2026-04-29):**

| Algorithm | Source | Growth | RMS | HMM |
|---|---|---|---|---|
| `refine_origin_sliding_offset` (sub-bin sliding-offset) | `onionskin_core/refinement.py:208` | ✓ direct (`growth_model_engine.py:504, 527`) | ✓ via `rcn_mean_shift_helpers.py:527, 547` | ✗ — Phase 15 SPEC15.7 dlv 3 ports it (gated on SPEC15.3 per-sample data) |
| `refine_summit_parabola` (multi-resolution + adaptive-window parabola; "Hybrid hierarchical + weighted-mean parabola summit refinement") | `onionskin_core/refinement.py:96` | ✗ | ✓ via `rcn_mean_shift_engine.py:287` + `rcn_mean_shift_helpers.py:498` | ✗ (HMM has its own simpler `_estimate_parabola_bp` single-window parabola at `hmm_ported_analyses.py:265`) |
| `_estimate_parabola_bp` (single-window parabola, HMM-local) | `onionskin_core/hmm_ported_analyses.py:265` | ✗ | ✗ | ✓ (HMM step-13 / step-15 outputs) |
| `hmm_summit_refinement.py` (multi-stage-aware filtering + Phase 9.3 summit refinement) | `onionskin_core/hmm_summit_refinement.py` | ✗ | ✗ | ✓ (HMM-specific; consumes step-11 + step-12 outputs; likely state-path-required so probably non-generalizable) |
| `stage2_score` / `stage2_refine` (stage-2 detection scoring + boundaries — NOT a summit-refinement algorithm) | `onionskin_core/refinement.py:263, 356` | ✓ | ✗ | ✗ (different concept: stage-2 detection vs summit refinement) |

**Cross-pipeline summit-algorithm parity gaps (work for this SOUP's eventual phase):**

1. **`refine_summit_parabola` port to Growth + HMM.** RMS has it; Growth + HMM don't. RMS's multi-resolution + adaptive-window parabola is a higher-quality algorithm than HMM's existing single-window `_estimate_parabola_bp`. Growth currently has no parabola summit refinement at all (uses sliding-offset only).
2. **HMM `_estimate_parabola_bp` unification with shared `refine_summit_parabola`.** If point 1 lands, HMM's local single-window parabola becomes redundant (or stays as a simpler-and-faster alternative — design decision).
3. **HMM sliding-offset port** — covered by Phase 15 SPEC15.7 dlv 3. NOT this SOUP's work; just cross-referenced for completeness.
4. **`hmm_summit_refinement.py` (multi-stage-aware filtering) generalization.** HMM-only currently; question is whether the multi-stage-aware filtering CONCEPT can be generalized to Growth + RMS via their multistage outputs (post-Phase-15 SPEC15.5 multistage unification work). Likely answer: HMM-specific in current form; could be re-architected as a shared multi-stage-aware filter that takes any pipeline's per-stage data — substantial refactor.

**Phase 15 cross-pipeline summit work that COMPLETES at Phase-15 close (NOT this SOUP's work):**

- HMM sliding-offset port (SPEC15.7 dlv 3).
- Cross-pipeline `--summit-stage-selection` strategy menu (1-5 cross-pipeline; 6 HMM-only) (SPEC15.7 dlv 1).
- ODW-confined summit refinement on each pipeline's existing summit-refinement code path (SPEC15.7 dlv 2).
- Cross-pipeline summit↔timing 4-step convergence (SPEC15.8).
- Cross-pipeline reliability scoring + classifier (SPEC15.6).
- Bayesian Origin Detection Window (ODW) System + 4-metric framework (SPEC15.4).

**Provisional deliverables for this SOUP's eventual phase (when it graduates to BRAINSTORM/SPEC):**

These extend the original Item 5 audit + unification framing with the concrete gaps identified by the 2026-04-29 code dive:

a. **Port `refine_summit_parabola` to Growth.** Wire the multi-resolution + adaptive-window parabola refinement into Growth's summit-refinement step. Emit `parabola_summit_bp` column alongside Growth's existing `sliding_offset_bp` columns.

b. **Port `refine_summit_parabola` to HMM.** Wire shared into HMM's summit-refinement step. Decide HMM's local `_estimate_parabola_bp` fate: replace / keep both (different columns) / flag-selectable.

c. **Cross-pipeline summit-algorithm column schema harmonization.** After (a) + (b), all 3 pipelines have both `sliding_offset_bp` and `parabola_summit_bp` columns. Verify shared column naming + semantics.

d. **Cross-pipeline strategy-menu wiring extension.** Each pipeline's `--summit-stage-selection` strategies can consume EITHER summit algorithm at the chosen stage(s).

e. **Validation against by-eye + PuffStep gold-standard surfaces.** Cross-pipeline summit-algorithm consistency must be demonstrated; per-pipeline summit-accuracy improvements must be measured.

f. **Documentation updates.** APS_CATALOG.md (created at Phase 15 SPEC15.17 dlv 4) gets new entries; PIPELINE_SPEC.md + ONIONSKIN_FULL_HANDOFF.md updated.

g. **Optional: generalize `hmm_summit_refinement.py` multi-stage-aware filtering** to a shared module that takes any pipeline's per-stage data (only if discussion item 4 in the Open brainstorm questions resolves to "yes, generalize").

**Provenance:** This expansion was originally drafted as `multi-agent/plans/PHASE16_SPEC.md` during Phase 15 SPEC engineering Round 3 (2026-04-29) under a "bypass SOUP/BRAINSTORM" deviation, but the corrected post-Round-3 code dive revealed the content was more brainstorm-like than SPEC-like (multiple open discussion items + scope TBD). The user's instinct (chat 2026-04-29) was that this content belongs in `SUMMIT_SOUP.md` rather than a premature Phase 16 SPEC. **Formerly lived in `multi-agent/plans/PHASE16_SPEC.md` and was referred to as `SPEC16.1` (RMS sliding-offset port; later corrected to "Cross-pipeline summit-refinement-algorithm parity completion"). PHASE16_SPEC.md was deleted at the port (2026-04-29).** The full audit history for the deviation + correction lives in `multi-agent/plans/PHASE15_FEEDBACK.md` § "SPEC engineering Round 3" + post-commit correction notes.

#### Item 5 expansion (2026-04-30) — cross-pipeline per-stage summit data emissions parity

The Phase 15 cycle 15.5a R3 re-audit (Round 2) surfaced a new cross-pipeline parity layer that the original Item 5 expansion (2026-04-29) didn't enumerate: **output-schema parity for per-stage summit data emissions.** This layer sits one level below the algorithm-parity layer covered above — it's about what each pipeline writes to disk per-stage, not about what algorithms each runs.

**Why it surfaced now.** Cycle 15.5a's RMS step-3 active-stage-refinement repair (R3 finding F-R2-1) requires reading per-stage parabola summit estimates from `_stage<N>_summit_estimates.tsv` and medianing over the strategy-selected stages. The discussion that followed considered an evidence-count column (`step3_n_stages_used`, "how many stages contributed parabola fits to the median") for downstream confidence weighting. The cross-pipeline question — "should all 3 pipelines emit it?" — exposed that the underlying per-stage data emissions are themselves asymmetric, so the column would carry semantically different meanings per pipeline (the same trap as cycle 15.5a R3 finding F-R2-2 on `max_rcn_stage`).

**Cross-pipeline per-stage summit data emissions landscape (verified 2026-04-30):**

| Per-stage data emission | Growth | RMS | HMM |
|---|---|---|---|
| Per-stage parabola summit table (one row per amplicon × stage with `parabola_summit_bp`) | ✗ direct table; per-stage data is reachable through `stage_median_within_calls.stage<N>.bedGraph` re-reads | ✓ (`_stage<N>_summit_estimates.tsv` post-cycle-15.5a R2 implementation) | ✗ — HMM has no per-stage parabola summit table; HMM's machinery is masked sliding-offset over per-sample step-5 bedGraphs |
| Per-stage sliding-offset summit table | ✗ | ✗ | ✗ — sliding-offset is computed on aggregated input, not per-stage |
| Per-amplicon "evidence count" column on step-3 / final-summit outputs (`step3_n_stages_used` or analog) | natural to emit (median-over-stages pattern) | natural to emit post-R2-J (median-over-stages pattern) | structurally different — HMM doesn't median over per-stage parabolas, runs single sliding-offset over masked input |

**What this means.** A clean cross-pipeline `step3_n_stages_used` (or any evidence-count column) requires either:

a. **Per-stage parabola summit emission cross-pipeline.** Growth + HMM gain per-stage parabola summit tables analogous to RMS's `_stage<N>_summit_estimates.tsv`. Strategy machinery medianizes per-stage parabolas uniformly across all 3 pipelines. **Tied directly to Open brainstorm question 6 (porting `refine_summit_parabola` to Growth + HMM)** — without that algorithm port, Growth has no parabola summit refinement at all and HMM has only the simpler `_estimate_parabola_bp` single-window output (which doesn't currently emit per-stage tables but could).

b. **Different evidence-count semantics per pipeline, with explicit documentation.** Growth + RMS get `step3_n_stages_used = "count of selected stages with valid parabola fits"`; HMM gets `step3_active_stages_n = "count of stages active in the mask after _compute_summit_estimates filtered"` — same conceptual idea ("evidence backing this summit"), different name + different calculation. Avoids forcing structural parity but ships divergent column names cross-pipeline (the kind of asymmetric output schema cycle 15.5a R3 F-R2-2 just exposed as a parity hazard).

c. **HMM machinery refactor to median-of-per-stage-summits pattern.** Replace HMM's masked-sliding-offset-over-aggregated-input with a per-stage-parabola-then-median pattern that matches Growth + RMS post-(a). Largest design change; couples to Open brainstorm question 7 (HMM `_estimate_parabola_bp` fate post-`refine_summit_parabola` port). Highest cost but produces the strongest cross-pipeline uniformity.

**Phase 15 scope question (orchestrator decision).** Cycle 15.5a's HMM sliding-offset port + stage-selection strategy menu landed without per-stage parabola summit emissions on HMM — the HMM lane uses masked sliding-offset over the per-sample step-5 input, structurally different from the RMS lane's per-stage-parabola-then-median pattern. The user's chat 2026-04-30 question — *"HMM needs the per-stage parabola summit -- doesn't it? are we not adding that in Phase 15? It seems like Phase 15 material → HMM pipeline completeness and enrichment"* — flags whether option (a) above is actually Phase 15 scope material. **Status: open question for the orchestrator.** Pragmatic considerations:

- Phase 15's theme IS HMM completeness + cross-pipeline enrichment, so HMM gaining per-stage parabola summit emission fits the theme.
- BUT cycle 15.5a is mid-execution; cycle 15.5a's R2-J round (Stages A/B/C for findings F-R2-1, F-R2-2, F-R2-3) is the immediate work. Adding HMM per-stage parabola summit emission mid-cycle expands cycle 15.5a's scope.
- Cleaner alternative: insert a new cycle (15.5b or similar mid-phase amendment, parallel to how SPEC15.22 + cycle 15.4b were inserted 2026-04-29) that scopes HMM per-stage parabola summit emission cross-pipeline if the user wants it in Phase 15.
- OR: leave it as SUMMIT_SOUP work for the eventual summit-methodology phase — accepted defer means cycle 15.5a closes with HMM's machinery structurally different from Growth + RMS, the parity gap is documented here, and the eventual phase fills it.

**Recommendation (deferred to user):** if Phase 15 scope expansion is on the table, this is a clean candidate (theme-aligned, well-bounded) and mirrors the SPEC15.22 mid-phase amendment pattern. If Phase 15 is being kept tight, this stays here and graduates with the rest of Item 5.

#### Item 5 expansion (2026-05-05) — triangle-apex SUMMIT ESTIMATOR (new estimator concept; not implemented in any pipeline)

**Background.** During Principal-orchestrator design exchange 2026-05-05 enumerating the curated 68-strategy summit-aggregation menu for cycle 15.10a-S2, a fourth estimator family beyond `argmax`/`parabola`/`sliding-window-parabola` was identified: **triangle-apex**, intended as a true summit-position estimator. This concept was meant to be added a long time ago but was missed.

**CORRIGENDUM 2026-05-05** (added during cycle 15.10a-S2 R1-audit-equivalent code dive while planning a then-proposed cycle 15.10a-S3 that was subsequently dropped): the original Background statement that "RMS already implements triangle-apex via the `--asym-tri-model-*` flag family" was an orchestrator transcription error. Code dive surfaced two distinct concepts that were conflated:

1. **Triangle FIT (existing in Growth; not a summit estimator).** `fit_asym_triangle()` at `onionskin_core/engines/growth_model_engine.py:649` is a fit-with-fixed-apex pattern intended for asymmetric-fork-elongation analysis. It takes `mu_bp` (the canonical upstream summit) as input and grid-searches over shape `(wL, wR)`; does NOT estimate apex from a profile. RMS + HMM lack this code entirely. **The cross-pipeline parity gap for this FIT code (Growth-only; RMS + HMM would benefit when fork-elongation downstream work fires) is filed at `multi-agent/plans/next/PIPELINE_PARITY_SOUP.md` Item 1.**
2. **Triangle-apex SUMMIT ESTIMATOR (this entry; not implemented in any pipeline).** A true summit-position estimator with the algorithm spec'd below.

**Algorithm spec (Principal direction 2026-05-05).** Starting at the approximate summit position, fix a window of length L (open parameter — empirically tune from candidates like 50 kb / 100 kb / 150 kb). Grid-search over candidate apex positions within the window. For each candidate apex, fit a triangle model to the RCN signal in the window (with the apex fixed at the candidate position; shape params either fixed or jointly fit). Return the apex position that minimizes the triangle-model-vs-RCN error.

**Hypothesis:** the apex would be positioned close to the summit (origin) location. Empirically tuning the window length L (and the loss function — SSE? weighted SSE? robust loss?) is part of the implementation.

**What this estimator adds to the curated 68-strategy menu:** a 4th genuinely-distinct summit estimator beyond argmax/parabola/sliding-window-parabola. When implemented + wired cross-pipeline (Growth + RMS + HMM), 17 triangle-apex strategies × 3 pipelines = 51 cells flip from NaN-emit to valid-emit (modulo the structural overlap with `odw-narrowest-state-*` × non-HMM pipelines, which stay NaN by HMM-only-selector rule).

**Status:** deferred to a future cycle. No near-term Phase 15 supplemental cycle planned for this. The curated 68-strategy menu in cycle 15.10a-S2 emits NaN for all 17 triangle-apex strategies across all 3 pipelines until this SOUP item is acted on.

**Dependencies.** Cycle 15.10a-S2 lands first (curated 68-strategy menu structure; menu emits NaN for triangle-apex strategies pending this item). Empirical tuning of window length L is part of this item's implementation work.

**Estimated scope.** Substantive new estimator: ~1-2 days for design + implementation + cross-pipeline integration + empirical window-length calibration + tests. Single shared module (e.g., `onionskin_core/summit_estimators.py`) consumed by Growth + RMS + HMM compute paths; same defaults across all 3 pipelines per cross-pipeline parity principle.

**Cross-references:**
- Cycle 15.10a-S2 mid-phase amendment 2026-05-05 + R1 audit equivalent (`multi-agent/plans/PHASE15_AUDIT_LOG.md` cycle 15.10a-S2 section) — corrigendum embedded; cycle 15.10a-S2 emits NaN for triangle-apex strategies pending this SOUP item.
- `[ISSUE:2026-05-05:2]` (cycle 15.10a-S2 landing zone for the curated 68-strategy redesign; corrigendum embedded).
- `multi-agent/plans/next/PIPELINE_PARITY_SOUP.md` Item 1 (the SEPARATE cross-pipeline parity gap for the existing Growth triangle FIT code, which is downstream-of-summit fork-elongation work — distinct from this summit-estimator concept).
- `fit_asym_triangle` at `onionskin_core/engines/growth_model_engine.py:649` (existing fit-with-fixed-apex code; NOT this estimator; reference only for "this is what's NOT here today").
- Memory `feedback_inverse_parity_invention.md` 2026-05-05 (the corrigendum incident — orchestrator made a pipeline-attribution claim without code-verifying; got it backwards).

#### Item 5 expansion (2026-05-05) — pre-saturation strategies deferred from cycle 15.10a-S2 F5 redesign

**Background.** During Principal-orchestrator design exchange 2026-05-05 enumerating the curated 68-strategy summit-aggregation menu for cycle 15.10a-S2, a `pre-saturation` stage selector was discussed: use ODW-active stages BUT exclude stages where the amplicon has saturated (signal flat-tops because all DNA molecules at that locus have been re-replicated). Per-stage parabola fits on saturated stages wash out summit information; pre-saturation excludes them.

**Why deferred:** the saturation criterion needs more careful definition. Options considered:
- "Stage where peak RCN doesn't significantly exceed prior stage's peak RCN by ≥ X%" (X needs calibration)
- "Stage where peak signal value plateau is detected via curvature/derivative criterion"
- "Stage where total amplicon-region copy-number reaches a saturation threshold"

Not enough thought has been put into this to define a rigorous saturation criterion without empirical calibration against by-eye + gold-standard surfaces. Cycle 15.10a-S2 deferred this strategy to SUMMIT_SOUP per Principal direction.

**What this item needs.**
- Define saturation criterion rigorously, with calibration against II/9A and II/2B (where saturation behavior is empirically observable).
- Add `odw-pre-saturation` selector to the 68-strategy curated menu (would expand to 68 + 4 estimators × 4 patterns = 16 additional strategies = 84 total cross-pipeline; same HMM-only triangle-apex caveat applies).
- Cross-pipeline implementation parallel to other selectors.

**Dependencies.** Cycle 15.10a-S2's curated 68-strategy menu lands first; this item adds 16 strategies on top.

**Estimated scope.** Calibration + design work: ~2-3 days including saturation-criterion empirical study. Implementation alongside any other Item 5 expansion.

**Cross-references:**
- Cycle 15.10a-S2 mid-phase amendment 2026-05-05 + R1 audit equivalent.
- `[ISSUE:2026-05-05:2]` (where the deferral was made).

---

### Item 6 (optional, pending evidence) — Summit output schema rationalization

**Background.** Phase 14 Supplemental 14-S28 did a band-aid column rename for the
`_origins.tsv` schema (`origin_*` → `final_summit_*`, honest naming on
`argmax_mean_bootstrap_sd_bp`). The schema has grown organically and carries redundant /
partially-overlapping columns (`argmax_mean_bp`, `argmax_median_bp`, `argmax_mean_ci_*`,
`parabola_mean_bp`, `parabola_median_bp`, `final_summit_bp`, `summit_estimator_used`,
etc.). After Items 1–5 land, a schema rationalization pass would decide what stays, what
merges, and what gets removed.

**Dependencies.** Best done after Items 1–5 are substantially complete.

**Out-of-scope for the first wave of Summit Methodology** — add only if time permits.

---

## Design principles for this phase (draft)

1. **"Summit" is the user-facing term.** Internal `refine_origin_*` function names stay
   as-is (14.2 internal-boundary rule), but all public output columns, CLI flag names,
   and help strings use "summit" per Phase 14 Supplemental 14-S28 harmonization.
2. **Disclose estimator choice per row.** `summit_estimator_used` stays; any future
   added estimators extend this column's value set.
3. **Clean bootstrap semantics.** After this phase, `final_summit_low/high_bp` has one
   well-defined meaning: bootstrap CI on the winning estimator per row. No mixed
   semantics. Band-aid help text from 14-S27/S28 is removed.
4. **One bootstrap approach shared across pipelines where applicable.** Per-stage
   replicate resampling is the natural shared primitive.
5. **Respect the 14.2 internal-boundary rule.** Code renames inside `onionskin_core/`
   do not propagate unless they map to a public surface change.
6. **Respect the scope-authority rule from v0.14.50.** No narrowing without user
   approval; raise broadening opportunities explicitly.

---

## Known interactions with other phases

- **the HMM-completeness phase:** Item 4 here depends on IBM-C2 (HMM parallel child
  pipeline) which is HMM-completeness-phase work. If IBM-C2 lands in the HMM-completeness phase, Item 4 here becomes
  free-standing.
- **Phase 14 Supplemental band-aid 14-S28:** the mixed-semantics column rename in
  Phase 14 Supp is intentionally transitional. This phase removes the band-aid by
  implementing Item 1.
- **Phase 13 side-quest closeout notes:** referenced "summit optimization as its own
  future phase." This is that phase.

---

## When to start

After the HMM-completeness phase lands. User direction: "the HMM-completeness phase is supposed to be
HMM-centric, and it is already bloating well beyond that" — so push summit-methodology
work to a dedicated follow-on.

---

## Open brainstorm questions (not yet answered)

1. **Parabola bootstrap — should bootstrap operate on onset-to-last-active window
   (Item 2) or all stages?** Current argmax bootstrap uses all stages. Item 2 lives in
   this phase; tempting to do them together.
2. **`argmax_mean_bootstrap_sd_bp` fate** post-parabola-bootstrap: keep alongside new
   `parabola_mean_bootstrap_sd_bp`, or replace with a single `final_summit_bootstrap_sd_bp`
   that picks based on `summit_estimator_used`? Clean option depends on whether users
   want both estimators' uncertainties side-by-side or just the winner's.
3. **Cross-pipeline naming unification** — Item 5 — how deep? Output column names only,
   or internal function names too?
4. **RMS selector policies** — should `early-parabola-mean` and siblings survive or be
   retired in favor of a unified estimator interface?
5. **HMM state-path summit** — how does it compose with parabola / sliding-offset when a
   state-path summit disagrees with them? Is the state-path summit a separate first-class
   estimator or a precondition / window-narrower for the others?

These are starter questions. More will emerge during brainstorming rounds.

#### Open brainstorm questions added 2026-04-29 (from corrected cross-pipeline parity audit; ported from former `PHASE16_SPEC.md`)

6. **`refine_summit_parabola` (multi-resolution + adaptive-window parabola) port to Growth + HMM?** RMS has it; Growth + HMM don't. RMS's algorithm is a higher-quality parabola summit-refinement than HMM's existing single-window `_estimate_parabola_bp`. Growth doesn't currently use ANY parabola summit refinement (uses sliding-offset only). Should Growth + HMM both gain `refine_summit_parabola` so all 3 pipelines have BOTH algorithms (sliding-offset + multi-resolution parabola) available? **User's stated principle (chat 2026-04-29):** "all pipelines should use all summit strategies (except where not possible like HMM-specific stuff)" — strongly suggests YES.
7. **HMM `_estimate_parabola_bp` unification with shared `refine_summit_parabola`?** If question 6 resolves YES, HMM's local `_estimate_parabola_bp` becomes either redundant (replace) or a simpler-and-faster alternative (keep both / flag-selectable). Decision matters for HMM step-13 / step-15 output column schema.
8. **HMM sliding-offset port placement** — already covered by Phase 15 SPEC15.7 dlv 3 (gated on SPEC15.3 per-sample step-5 data). NOT this SOUP's work; cross-referenced for completeness so this SOUP's eventual phase doesn't duplicate.
9. **`hmm_summit_refinement.py` (multi-stage-aware filtering, Phase 9.3) generalization to Growth + RMS?** Currently HMM-only; consumes step-11 + step-12 (HMM-step-derived) outputs. Question is whether the multi-stage-aware filtering CONCEPT can be generalized to Growth + RMS via their multistage outputs (post-Phase-15 SPEC15.5 multistage unification work). Likely answer: HMM-specific in current form; could be re-architected as a shared multi-stage-aware filter that takes any pipeline's per-stage data — substantial refactor, only worth doing if the generalized form has clear benefits over the per-pipeline shape-filter / multistage-unification machinery already in growth/RMS.
10. **Cross-pipeline summit-algorithm parity completion scope sizing.** If questions 6 + 7 + 9 all resolve toward generalize, this SOUP's eventual phase grows from "audit + minor unification" to "comprehensive cross-pipeline summit-algorithm parity completion" — could justify multiple priorities + a phase-group structure when it graduates. If only question 6 resolves YES, the eventual phase stays small (1-2 priorities). If all questions resolve toward "no, current per-pipeline algorithm choices are intentional," the eventual phase shrinks to mostly schema rationalization (Item 6).

#### Open brainstorm questions added 2026-04-30 (from cycle 15.5a R3 Round 2 re-audit fallout)

11. **Cross-pipeline evidence-count column parity (`step3_n_stages_used` / `step3_active_stages_n` / equivalent).** Should step-3 summit outputs cross-pipeline carry an evidence-count column indicating how many stages of evidence backed the step-3 summit? See "Item 5 expansion (2026-04-30) — cross-pipeline per-stage summit data emissions parity" above for the per-pipeline matrix + the three options (a/b/c) for how to make the column cross-pipeline-uniform. The question is conditional on whether downstream consumers (plots, filters, QC reports, future APS feature modes) actually want such a column — if no consumer wants it, defer indefinitely. Tied to questions 6 + 7 (without `refine_summit_parabola` ported to HMM, option (a) — the cleanest cross-pipeline-uniform answer — is blocked by HMM's structural lack of per-stage parabola summit emissions).
12. **Phase 15 scope expansion — HMM per-stage parabola summit emission?** User flagged at chat 2026-04-30 that HMM's lack of per-stage parabola summit emission is theme-aligned with Phase 15's HMM completeness + cross-pipeline enrichment goal. Three resolution paths: (a) expand Phase 15 scope mid-execution by inserting a new cycle (15.5b or similar mid-phase amendment, parallel to SPEC15.22 + cycle 15.4b inserted 2026-04-29); (b) keep Phase 15 tight, defer to this SOUP's eventual phase; (c) split — Phase 15 scopes the OUTPUT (HMM emits per-stage parabola summit table), this SOUP scopes the ALGORITHM port (`refine_summit_parabola` → HMM). Question is open — requires user decision at orchestrator triage time. **Status as of 2026-04-30: parked here pending user direction.** **Status (2026-05-04): Resolved — absorbed into Phase 15 as SPEC15.24, cycle 15.7b.**

#### Open brainstorm questions added 2026-05-05 (from cycle 15.10a Stage C F5 implementation fallout)

13. **[SUPERSEDED 2026-05-05 by cycle 15.10a-S2 redesign — original premise no longer holds.]** Question originally asked about default-pick decisions for the 9-strategy summit-aggregation menu landed by cycle 15.10a Stage C F5 (SPEC15.19 d3). That 9-strategy 2-flag menu was REVERTED at cycle 15.10a-S2 (v0.14.97); the deprecated modules (`summit_aggregation.py`, `summit_diagnostics.py`, `summit_diagnostic_runner.py`) were deleted. The replacement is the curated 68-strategy single-flag menu in `onionskin_core/summit_strategies.py`. Defaults still stay UNCHANGED in cycle 15.10a-S2 (each pipeline's current default summit behavior is preserved); strategy menu is opt-in via `--summit-stage-selection` + 3 pipeline-specific overrides. Default-pick decision for any of the 68 strategies (or a different-than-current-default per pipeline) remains a post-Phase-15 Principal-driven activity informed by the diagnostic BED + IGV inspection on a representative full-genome run. Tied to question 4 (RMS selector policies — `--rms-summit-policy early-parabola-mean` overlaps semantically with curated `early-N-non-ref-*-mean` strategies; future-phase consolidation candidate).

14. **[SUPERSEDED 2026-05-05 by cycle 15.10a-S2 redesign — original premise no longer holds.]** Question originally asked about incremental existing-method-capture wiring into `summit_diagnostic_runner.py` (deleted by cycle 15.10a-S2 Stage A revert). The cycle 15.10a R2 9-strategy 2-flag F5 implementation deferred ~20 pipeline-specific existing-method captures (Growth ~4, RMS ~8, HMM ~12) as future-cycle scope. Cycle 15.10a-S2's curated 68-strategy single-flag menu replaces that approach with 60 cross-pipeline strategies + 8 HMM-only strategies built from selector × estimator × aggregator deterministic enumeration; existing-method captures are no longer the conceptual frame. The 68-strategy menu is the canonical surface; if specific legacy method captures (e.g., HMM step-18-timing-summit, RMS refine-summit-parabola) prove useful in IGV inspection beyond what the 68 expose, they can be filed as targeted future-phase items individually. Cadence question (one-per-pipeline vs one-per-method-family vs opportunistic) is no longer relevant.
