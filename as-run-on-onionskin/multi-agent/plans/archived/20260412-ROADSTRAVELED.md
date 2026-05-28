# ROADSTRAVELED

This file is an archive of ROADMAP entries **as they were written**, preserved verbatim at the moment they were collapsed or replaced with a condensed one-liner in `ROADMAP.md`.

## Purpose

When a ROADMAP entry is collapsed (marked `✓ DONE` and reduced to a summary line to keep ROADMAP readable), the original full entry — including design notes, rationale, and implementation details — is prepended here so it remains accessible for future reference.

Entries are in reverse-chronological order: the most recently archived entry appears first.

---

<!-- New entries go here, above this line -->

---

## Archived: Phase 6 — APS feature evolution (closed v0.6.00)

> **Why archived:** Phase 6 closed. Priority 6.1 (shape clustering) done. Priorities 6.2
> (dimensionality reduction) and 6.3 (shape-aware clustering) deferred post-HMM to BRAINSTORM.md.

**Priority 6.1 — Bin-wise APS feature vectors ✓ DONE (v0.3.32)**
`--aps-feature shape` implemented in `build_aps_shape_matrix()`. Per-amplicon normalization
via `1/√n` weights so every locus contributes equally to Euclidean distance. Exposed via
`--aps-shape-no-normalize`. Shape matrices written to `aps_matrix_shape_raw.tsv` /
`aps_matrix_shape_zscore.tsv`.

**Priority 6.2 — Dimensionality reduction / embedding → BRAINSTORM**
PCA, UMAP, diffusion-like developmental embeddings. Full notes in BRAINSTORM.md [2026-04-07].

**Priority 6.3 — Shape-aware clustering → BRAINSTORM**
Cluster using amplicon morphology, not just integrated magnitude. Full notes in BRAINSTORM.md.

---

## Archived: Phase 5 — Posterior reruns and APS evolution (closed v0.6.00)

> **Why archived:** Phase 5 closed. All core items done or deferred. Summary of outcomes:

**Completed in Phase 5 (v0.5.01–v0.5.66):**
- 5.1 Posterior reruns ✓ (v0.5.01)
- 5.2 APS weighting strategies ✓ (v0.5.49)
- 5.5 Cluster-k selection (`elbow`, `sweep`, `keep`) ✓ (v0.5.52)
- 5.7 `--no-aps-singleton-guard` ✓ (v0.5.08)
- 5.8 Summit estimation grouping-invariance fix ✓ (v0.5.05–v0.5.07)
- 5.9.1 Double-smoothing audit ✓ (v0.5.17 — smooth_mean kept; rolling_median worse)
- 5.9.2 Sub-bin resolution philosophy ✓ (v0.5.54–v0.5.56 — both levels reported)
- 5.9.5 Summit benchmark infrastructure ✓ (v0.5.09–v0.5.16 — `make summit` + posterior eval)
- 5.9.7 Switch `final_origin_bp` to parabola_mean ✓ (v0.5.12)
- 5.9.8 Posterior eval in summit test suite ✓ (v0.5.13–v0.5.16)
- 5.10 Summit output files schema ✓ (v0.5.07)
- 5.11 Sliding-offset sub-bin refinement ✓ (v0.5.54–v0.5.56); parabola portability ✓ (v0.5.56)
- 5.13 Shape filter for multistage calls ✓ (v0.5.66)

**Deferred to BRAINSTORM (post-HMM):**
- 5.0.2 timing.py fold-change via refined summit
- 5.3 APS order stability diagnostics
- 5.4 Better posterior ordering logic
- 5.9.3 Timing-guided stage weights
- 5.9.4 Iterative summit ↔ timing EM convergence
- 5.9.6 Prior/posterior profile similarity investigation
- 5.12 Robustness dataset expansion (user-driven)

Full ROADMAP entries for all items retained (abbreviated) in ROADMAP.md Phase 5 section.

---

## Archived: Phase 6.5 — Code Unification Refactor (collapsed v0.5.62)

> **Why archived:** Phase 6.5 fully complete as of v0.5.62. All six priorities done.
> Full plan file preserved at `multi-agent/plans/archived/20260407-CODE-UNIFICATION-REFACTOR.md`.

---

**Authors:** John M. Urban, Claude Code 2.1.85 (claude-sonnet-4-6)

# Phase 6.5 — Code Unification Refactor (ACTIVE — pre-HMM prerequisite)

## Goals
Consolidate duplicated code across `engines/single.py`, `engines/multistage.py`, and
`single_engine.py` so that any logic developed for one engine is immediately portable
to the others. Required before HMM integration to avoid integrating into three
separate code paths. After this phase, adding a new estimator, summit method, or
detection approach means writing it once and wiring it into a shared module.

**Full plan:** `multi-agent/plans/archived/20260407-CODE-UNIFICATION-REFACTOR.md`

---

## Priority 6.5.A — Break the circular import chain ✓ DONE (v0.5.57)
- `common.py` module-level engine imports → lazy (inside `load_reference_*` functions)
- Unused `_multistage_main` import removed
- `engines/multistage.py` deferred refinement imports → module-level

## Priority 6.5.B — Create `signal_utils.py` ✓ DONE (v0.5.58)
- Extract `rolling_median`, `smooth_mean`, `robust_z` (canonical: array API), `conv1d`,
  `boxcar_kernel`, `find_local_maxima`, `bic`, `r2_from_xy`, `nice_name`
- Resolves `robust_z` API divergence between engines

## Priority 6.5.C — Expand `detection.py` ✓ DONE (v0.5.59)
- Move shared detection algorithms from both engine monoliths: `Candidate`,
  `stage1_mean_shift`, `fit_block`, `fit_bump_triangle_basis`, `dedup_calls_by_peak_proximity`
- Remove inline copies from both engines

## Priority 6.5.D — Expand `refinement.py` ✓ DONE (v0.5.60)
- Move `boundaries_and_peak`, `blockiness_metrics`, `stage2_score` from `engines/single.py`
- Move `_dBIC_flat_vs_tri`, `_shape_filter_calls` from `single_engine.py`
- `stage2_refine` calls these directly instead of via `load_reference_single()`
- Also: `io.read_manifest` canonicalized as sole manifest reader; `parse_manifest` removed
  from `engines/multistage.py`; `load_tracks_from_manifest` accepts `List[Sample]`

## Priority 6.5.E.0 — Output directory numbered-prefix refactor ✓ DONE (v0.5.61)
Implements the hierarchical pipeline-arm prefix scheme. See DECISIONS.md [2026-04-07].

**Final scheme** (inside each grouping dir, e.g., `01-prior/`):
- `00-INDEX.md` — flat file guide, sorted first
- `00-signal-files/` — shared: per-sample bedGraphs + genome stage medians
- `01a-stage_median_within_calls/` — multistage internal (kept; see DECISIONS.md [2026-04-07])
- `01b-summary_bedgraphs/`
- `01c-summits/`
- `01d-summit_refinement/` — summit estimate TSVs/BEDs (replaces `others/`)
- `01e-aps/`
- `01f-plots/`
- `01g-notebook/`
- `02-per-stage-mean-shift/` — Phase E (created in Phase E)
- **No `others/`** — permanently eliminated
- `03-per-stage-HMM/`, `04-unified-results/` — created when those phases are implemented

**Rationale for `01a-`/`01b-`/… notation:** groups current multistage subdirs under arm `01-`
without requiring a `01-multistage-growth/` wrapper refactor until Phase 7. Pipeline arm
dirs `02-`/`03-`/`04-` will be stable at outer level forever.

**Files changed:** `output_layout.py`, `onionskin.py`, `aps.py`, `timing.py`, `notebooks.py`, `tests/test_aps.py`
**CHANGELOG:** v0.5.61

## Priority 6.5.E — Per-stage mean-shift outputs ✓ DONE (v0.5.62)
Run single-mode detection/scoring on each stage's aggregate track within multistage,
with full shape filtering, BED outputs, and cross-stage unification.

**Architecture (settled 2026-04-07; see BRAINSTORM and CODE-UNIFICATION-REFACTOR.md):**
- Output location: `02-per-stage-mean-shift/` (created by Phase E; Phase E.0 registers the dir)
- Posterior re-run: automatic (engine called again with posterior manifest)
- HiRes: base resolution only for Phase E; hires summit refinement is Phase E.2
- Flag: `--skip-per-stage` skips the pipeline; default is to run it

**Per-stage outputs** (for each stageN):
- `stageN_aggregate.bedGraph`, `stageN_calls.tsv`, `stageN_calls.bed`, `stageN_summits.bed`
- `stageN_putative_collapsed_repeats.tsv`, `stageN_putative_collapsed_repeats.bed`

**Unified outputs** (cross-stage non-redundant union with `stages_present` column):
- `unified_stage_calls.tsv/.bed`, `unified_collapsed_repeats.tsv/.bed`

**Implementation:** `run_per_stage_mean_shift()` in `single_engine.py`; wired into
`engines/multistage.py` main() reusing in-memory `base_tracks`. New per-stage
detection params: `--per-stage-z-thresh`, `--per-stage-halfwidths-kb`, etc.
**CHANGELOG:** v0.5.62

---

## Archived: Priority 4.9 — Integrated final amplicon keep/exclude recommendations (collapsed v0.5.50)

> **Why archived:** Priority 4.9 was marked ◑ PARTIAL since v0.4.13 because the "Future
> development" bullet list was open-ended. In v0.5.50 the entry was reviewed and all four
> core output files are confirmed present and working; the integration logic is implemented.
> The future-development items are aspirational enhancements, not blocking functionality.
> Closed as DONE; enhancements ported to BRAINSTORM.md [2026-04-07] for future work.

---

**Authors:** John M. Urban, Claude Code ~2.1.81 (claude-sonnet-4-6) — original implementation v0.4.13; reviewed and closed v0.5.50

### Priority 4.9 — Integrated final amplicon keep/exclude recommendations ◑ PARTIAL (v0.4.13)

At the end of Phase 2 post-processing, after all analytical modules have run, produce a single
integrated decision per amplicon: **exclude** or **keep** (with caveats).

#### Current outputs (v0.4.13)

| File | Contents |
|---|---|
| `candidate_amplicons_recommended_for_exclusion.bed` | Intermediate; accumulated during pipeline from gap analysis (and potentially other future sources); append-mode |
| `amplicons_recommended_for_exclusion.bed` | **Final**; integrated across gap + timing + width growth; see logic below |
| `amplicons_recommended_to_keep.bed` | **Final**; all kept amplicons, annotated with gap metrics and growth signal |
| `amplicons_actively_rejected.bed` | Shape filter hard rejects (separate, not included in final calls) |

#### Integration logic (v0.4.13)

**`summit_dense`** (gaps AT the origin/summit window):
- → Always reject. Hard veto. `integrated_reason=gap_summit_dense` (or `gap_summit_and_timing_excluded`)
- No rescue possible.

**`high_density`** (gaps throughout amplicon but summit is clean):
- → Reject by default. `integrated_reason=high_density_no_growth_evidence`
- **Rescue**: if width growth signal is present (`width_slope_kb_per_stage > 0`, `width_r2 ≥ 0.40`) AND `timing_excluded=False` → retain with `rescue=width_growth_rescued_from_high_density_reject`
- If `timing_excluded=True`, reject even with growth. `integrated_reason=high_density_and_timing_excluded`
- If width growth present but rejected anyway: `note=width_growth_present_consider_manual_review`

**`timing_excluded=True`** (any gap_concern):
- → Always reject. `integrated_reason=timing_excluded`

**Review-level concerns** (`dispersed`, `summit_proximal`):
- → Retain with `note=review_level_gap_concern_not_sufficient_for_rejection`

**Clean**:
- → Retain with `note=no_concerns`

#### BED name annotations (all amplicons in both output files)

- `gap_concern=<value>` — from gap analysis
- `gap_fraction=<f>` — fraction of amplicon covered by gaps
- `gap_count=<n>` — number of distinct gaps (separate from fraction; 1 long optical map gap ≠ 3 short gaps)
- `gap_fraction_summit_window=<f>` — gap fraction in summit ±half-width window
- `gap_count_near_summit=<n>` — gap count near summit
- `width_growth_kb_per_stage=<v>` — from `_origins.tsv` (multistage only)
- `width_growth_r2=<v>` — R² of width vs stage regression
- `timing_excluded=true` — if timing outlier flag set
- `integrated_reason=<value>` / `integrated_decision=retained` / `rescue=<value>` / `note=<value>`

#### Rescue thresholds (current)

```python
_WIDTH_RESCUE_MIN_SLOPE_KB = 0.0   # any positive width growth qualifies
_WIDTH_RESCUE_MIN_R2       = 0.40  # R² ≥ 0.40 required
```

#### Future development (ported to BRAINSTORM)

See BRAINSTORM.md [2026-04-07] for the full list of aspirational enhancements.

---

## Archived: Priority 4.10 — Post-hoc summit refinement `--refine-summits` (dropped v0.5.50)

> **Why archived:** Priority 4.10 was never implemented — only a placeholder comment in
> `onionskin.py`. Dropped in v0.5.50 to focus development on HMM integration (Phase 7),
> which is the highest-priority direction. The `--refine-summits` design is fully
> preserved here and in BRAINSTORM.md [2026-04-07]. Revisit after HMM integration
> is in place, if finer per-sample summit resolution becomes important again.

---

**Authors:** John M. Urban — original ROADMAP entry; dropped v0.5.50 by John M. Urban, Claude Code 2.1.85 (claude-sonnet-4-6)

### Priority 4.10 — Post-hoc summit refinement (`--refine-summits`)

Hires manifests currently only affect summit position estimation and do not improve
call detection, timing, APS, or any other pipeline output.  Because the summit module
is still under active development, the cost:benefit ratio of including hires data in
a first-pass run is poor.  The recommended pattern is:

1. Run onionskin without `--hires-manifest` (fast, base-resolution).
2. Optionally re-run summit refinement later with hires data if finer precision is needed.

#### Planned design

A `--refine-summits` flag (or subcommand) that:
- Accepts an existing onionskin output directory + one or more hires manifest paths.
- Reads the resolved BED and any existing run parameters from that directory.
- Re-runs only the summit estimation module with hires bedGraph data.
- Updates summit-related outputs in-place (`_origins.tsv`, `_summits.*.bed`) without
  re-running detection, timing, APS, or overlap resolution.

#### Also needed

- `--hires-manifest` help text and README should clearly state the experimental
  status and low first-pass value. ✓ Done (v0.4.18 / README).
- Once the summit module is meaningfully improved (see project_summit_todo.md),
  benchmark the hires benefit using the chr II ground truth BEDs and update the
  user guidance accordingly.

---

## Archived: Priority 4.11 — Unified summit position for all summit-related computations (collapsed v0.5.50)

> **Why archived:** Priority 4.11 was marked ◑ PARTIAL since v0.5.08. Reviewed in v0.5.50:
> the APS_summit fix (threading `final_origin_bp` from `_origins.tsv` through `summit_bp_map`
> into `aps.py`) is confirmed implemented. The stage-activity fold-change fix was explicitly
> deferred within the original ROADMAP entry itself ("defer to Phase 5 or a dedicated
> sub-priority") and remains deferred. Closing the APS_summit portion as DONE; the
> stage-activity fix is a separate future item if needed.

---

**Authors:** John M. Urban, Claude Code ~2.1.81 (claude-sonnet-4-6) — original entry and APS_summit implementation v0.5.08; reviewed and closed v0.5.50

### Priority 4.11 — Unified summit position for all summit-related computations ◑ PARTIAL (v0.5.08)

#### Problem

Summit estimation in the refinement module (`refine_summit_parabola`) carefully computes `final_origin_bp` for each amplicon and writes it to `_origins.tsv`. However, downstream modules that report "summit" quantities recompute the summit position independently using cruder methods:

- **APS_summit** (`aps.py`): `max(RCN - 1)` over the entire amplicon BED window. Can grab a bin far from the true origin if there is a local artifact or asymmetric shoulder anywhere in the interval.
- **Stage-activity fold-change** (`timing.py`, `_ACTIVE_FOLD_THRESHOLD = 1.25`): uses `median_peak_fold` from `stage_summary`, which is the bump-model fitted height at the *detection-phase* peak position — not the refined `final_origin_bp`.
- **`max_RCN`** in timing outputs: same source as above.

The consequence is that all three concepts — "what is the amplitude of this amplicon?", "is it actively growing this stage?", and "where does it peak?" — are computed from different positions using different heuristics.

#### Design

The authoritative summit position is `final_origin_bp` in `_origins.tsv` (one row per `call_id`). All summit-magnitude computations should read from the smoothed RCN at or near that position:

**APS_summit fix (tractable now):**
- `Locus` already has `call_id`; `_origins.tsv` already has `final_origin_bp` per `call_id`
- Add optional `origins_tsv` parameter to `compute_and_write_aps` → `compute_aps_tables`
- Pass `summit_bp: Optional[int]` into `_locus_metrics` per locus
- Compute `summit_excess = RCN_at_summit - 1` where `RCN_at_summit` = smoothed RCN value at the bin containing `summit_bp` (or median of ±1 bin window), instead of `max(RCN over whole window)`

**Stage activity fold-change fix (larger scope):**
- `timing.py` would need to read the per-sample bedGraph tracks at `final_origin_bp` rather than using `median_peak_fold` from `stage_summary`
- OR: add a `summit_RCN` column to `stage_summary` by having the multistage engine sample the bedGraph at the refined summit position after refinement completes
- This is a more significant refactor; defer to Phase 5 or a dedicated sub-priority

#### Implementation status at time of collapse (v0.5.50)

**Done:**
- `summit_bp_map` threaded into `aps.py` via `origins_tsv` parameter (v0.5.08)
- `APS_summit` now samples RCN at `final_origin_bp` ± window instead of `max(RCN)` over window

**Deferred (not part of 4.11 scope):**
- Stage-activity fold-change fix in `timing.py` — larger refactor, future item if needed

---

## Archived: Priority 5.2 — APS weighting strategies (collapsed v0.5.49)

> **Why archived:** Priority 5.2 was closed in v0.5.49.  The original objective
> (per-locus APS weighting) was reconsidered: confirmed real amplicons all showed
> statistically credible oscillations under the probabilistic test, so no
> discriminator between "real oscillator" and "spurious detection" was found.
> The scope was changed to oscillation annotation + global degradation detection
> (which are fully implemented).  The APS weighting goal is preserved in
> BRAINSTORM.md for future work.
>
> **Note on ROADSTRAVELED backfill:** ROADSTRAVELED was empty prior to v0.5.49
> because the convention was established (v0.5.29) after most Phase 4 entries were
> already collapsed.  Those entries are recoverable from CHANGELOG.md and git history.
> Going forward all collapses must be archived here first.

---

**Authors:** John M. Urban, Claude Code ~2.1.81 (claude-sonnet-4-6) — weighting design expanded from original sparse entry in v0.5.29; purpose and credibility-discount framing clarified in v0.5.42; diagnostic column definitions corrected and placeholder limitations documented in v0.5.44

### Objectives
Add per-locus reliability weighting to APS computation. The weight is a credibility
discount — not a size or amplitude adjustment. Locus size and amplitude are already
naturally encoded in `area_excess = sum((RCN−1) × bin_width)`: a large, high-amplitude
amplicon already contributes far more to APS than a small, weak one without any explicit
weighting. `locus_weight` adjusts only for **confidence** that a detected locus is a
true, consistent amplification event.

### Scientific reason
APS is a sum of `area_excess` across detected loci. Not all detected loci are equally
trustworthy: some flicker across stages, some were produced by uncertain twin-peak splits,
some have high gap density near the summit. A locus that flickers or was poorly resolved
still contributes its full `area_excess` to every sample's APS — even though it is a
noisier indicator of developmental progression than a locus that is rock-solid across
all post-onset stages.

`locus_weight` is a per-locus reliability discount `w_ℓ ∈ [0, 1]` applied to each
locus's `area_excess` contribution. Reliable loci receive `w_ℓ ≈ 1`; noisy, flickery,
or poorly-resolved loci receive `w_ℓ < 1`. This produces a **reliability-weighted APS**
that is a better signal for developmental ordering when some detected loci are dubious.

This is conceptually a posterior APS: after seeing all stage data, the full dataset is
used to assess each locus's credibility, and that credibility is folded back into the
APS sum. It is distinct from posterior ordering/grouping (which reorders samples
using APS clusters) — this operates at the locus level, not the sample level.

### Important caution: late-onset loci
Do not penalize true late-onset loci merely because they are absent early. A locus that
first appears only in late samples is still a real and important part of the amplification
program if it is detected confidently in those stages. Weighting must be conditioned on
**post-onset** behavior, not overall prevalence across all stages.

### Important caution: area_excess vs summit RCN
`area_excess` is the WRONG signal for onset detection and dip detection. It grows with
fork travel distance and compounds across stages — the biggest area jump is almost always
the last stage regardless of when the origin fired. All onset- and consistency-related
diagnostics must use **summit RCN** (`peak_rcn` at the replication origin), not area.
Summit RCN reflects origin-firing directly; area reflects cumulative fork elongation.

### Important caution: late-stage nuclease degradation
In datasets with many developmental stages (e.g., DS1 with 9 stages), late stages may
show declining summit RCN even though the locus was genuinely amplified earlier. Salivary
gland breakdown involves nonspecific nucleases that preferentially remove amplified
sequence (more copies = more absolute loss), potentially reducing apparent summit RCN.
The stage of maximum summit RCN is not necessarily the last stage and should not be
assumed to be. `dip_rate` is designed to flag this, but its definition is not yet final.

### Weighting machinery (implemented v0.5.44, formula pending)

CLI: `--aps-weight-loci` (off by default).

When active:
- `contrib_df` gains `weighted_area_excess = locus_weight × area_excess`
- `aps_df` gains `APS_area_weighted = sum(weighted_area_excess per sample)`
- When `--aps-feature area`, clustering uses `weighted_area_excess` instead of
  `area_excess` so unreliable loci influence grouping less
- No effect on `summit`, `width`, or `shape` clustering features

The weighted APS form is:
```
APS_weighted(s) = Σ_ℓ  w_ℓ × Σ_{x ∈ ℓ}  max(RCN_s(x) − 1, 0)
```

**locus_weight is currently always 1.0 (placeholder).** The machinery is in place;
the formula is not yet implemented. See diagnostic column definitions below and
BRAINSTORM.md [2026-04-01] "APS locus diagnostics — design discussion."

### Diagnostic columns (implemented, definitions partially correct)

These columns appear in `onionskin_aps_locus_contributions.tsv`.  All are computed
post-hoc from the full `contrib_df` after `compute_aps_tables()` returns.

**`best_onset_stage`** — stage when amplification was first detected.
- CURRENT IMPL: pulls `onset_stage` from timing TSV (preferred); falls back to first
  stage with mean `peak_rcn >= 2.0`. Uses summit RCN — correct signal.
- KNOWN LIMITATION: if stage 1 already shows amplification, onset = stage 1, but
  we cannot infer that amplification was absent before stage 1.

**`post_support`** — PLACEHOLDER, definition needs rethinking.
- CURRENT IMPL: mean `peak_rcn` across ALL stages (not conditioned on onset).
- INTENT: some measure of "how consistently and strongly is this locus detected?"
  after onset. Candidates: fraction of post-onset stages above threshold; mean
  post-onset peak_rcn; coefficient of variation of post-onset peak_rcn.
- Do not rely on current values for biological interpretation.

**`dip_rate`** — fraction of stages where mean `peak_rcn < 0.5 × max(peak_rcn)`.
- Uses summit RCN — correct signal (area would never dip even during degradation).
- KNOWN LIMITATION: currently computed over ALL stages, not post-onset only.
  Pre-onset low values are expected and should not count as dips. Fix: restrict to
  post-onset stages once `best_onset_stage` definition is finalized.
- The 0.5 threshold is arbitrary and uncalibrated.

**`locus_weight`** — always 1.0. Placeholder for the eventual reliability weight.

### Implementation status at time of collapse (v0.5.49)

**Done:**
- `--aps-weight-loci` CLI flag, `weighted_area_excess`, `APS_area_weighted` output columns
- `best_onset_stage` from timing TSV with coord-based fallback for call_id format mismatches
- `post_support` and `dip_rate` diagnostic columns (definitions still imperfect)
- Per-stage MAD tracks: `stageN.MAD_RCN.bedGraph`, `stageN.MAD_log2RCN.bedGraph` (v0.5.47)
- Probabilistic oscillation test: P(RCN_s > RCN_{s+1}) = Φ(z), σ = MAD × 1.4826 (v0.5.48)
- Oscillation annotation columns: `max_rcn_stage`, `regression_stage`,
  `oscillating_transitions`, `is_oscillator` (v0.5.49)
- Global degradation report: `aps_stage_regression_report.tsv` (v0.5.49)

**Calibration amplicons (DS1 chr II):**
- `chrII:45755000-46620000` — genuine weak amplicon, confirmed by IGV + HMM in DS2
- `chrII:54380000-55152500` — real
- `chrII:21440000-22737500` — real, first peak of twin-peak training pair
- `chrII:40660000-42530000` — real, high variance from small N per stage

**Key design constraints (user-confirmed):**
- Must NOT penalize low absolute RCN — detecting weak, broad, low-level amplicons is
  a founding goal of onionskin
- Must work on prior groupings, not just posterior
- Must be variance-aware — point estimates of per-stage means misfire at N=2 per stage

**Why the weighting goal was deferred:** All four calibration amplicons showed
statistically confirmed oscillations under the probabilistic test — the drops are real
biology (nuclease degradation, twin-peak measurement, sample-level variance), not noise.
No discriminator between "spurious oscillation" and "real oscillating amplicon" was
found with the available data.  Scope changed to annotation-only; weighting deferred.

