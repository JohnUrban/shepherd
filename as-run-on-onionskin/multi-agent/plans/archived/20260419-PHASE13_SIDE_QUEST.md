# Phase 13 Side Quest: RMS Threshold + Step-08 Final Classification

## Purpose

This document defines a tightly scoped RMS side quest that should be executed
before any broader Phase 13 drift. It has two linked goals:

1. Revisit the effective RMS default detection sensitivity now that the
   stage-local shape filter exists, with the current leading candidate being
   `--z-thresh-single = 2.5`.
2. Fix the step-08 correctness bug where the same locus can currently survive
   both as a unified amplicon and as a unified collapsed repeat because accepted
   stage-local calls and shape-filter rejects are unified on independent tracks.

The intended outcome is not a directory-layout rewrite. The intended outcome is
one shared step-08 interval universe, one explicit cross-stage classification
pass, and one true non-overlapping final partition into unified amplicons versus
unified collapsed repeats, with enough breadcrumb metadata that later agents can
audit or extend the behavior without reverse engineering it from artifacts.

## Scope Decisions

These decisions are part of the plan, not open-ended brainstorming:

1. Keep this work inside the existing `08-multistage-unification/` surface for
   this round.
2. Do not start by renumbering RMS steps or creating a new final-classification
   directory.
3. Preserve the step-06 shape filter as a real stage-local filter. Cross-stage
   rescue happens after that local decision, not by turning step 06 into a
   no-op annotation pass.
4. Prefer additive schema changes over path moves.
5. Bundle the threshold change only if the user still wants it in the same
   implementation round after the step-08 classifier design is stable.

## Current Verified Contract

These facts were re-audited against the live repo before writing this plan:

1. [onionskin.py](onionskin.py) exposes the user-facing `--z-thresh-single`
   default at `4.0` and passes it into RMS execution.
2. [onionskin_core/detection.py](onionskin_core/detection.py) contains the
   shared `stage1_mean_shift()` threshold logic where that z-threshold actually
   takes effect.
3. [onionskin_core/rcn_mean_shift_engine.py](onionskin_core/rcn_mean_shift_engine.py)
   runs per-stage detection, deduplication, shape filtering, and summit
   refinement, then writes unified calls and unified collapsed repeats to
   `03-rcn-mean-shift/08-multistage-unification/`.
4. [onionskin_core/per_stage_mean_shift_engine.py](onionskin_core/per_stage_mean_shift_engine.py)
   owns the shared `_unify_stage_calls()` and `_calls_to_bed_domains()` helpers
   that define most of the current step-08 behavior.
5. [onionskin_core/output_layout.py](onionskin_core/output_layout.py) and
   [multi-agent/full_instructions/PIPELINE_SPEC.md](multi-agent/full_instructions/PIPELINE_SPEC.md)
   already describe `08-multistage-unification/` as the home of unified
   cross-stage calls and unified collapsed repeats.
6. [tests/test_pipeline.py](tests/test_pipeline.py),
   [tests/test_rcn_mean_shift_selector.py](tests/test_rcn_mean_shift_selector.py),
   [tests/test_eval_summit_precision_v2.py](tests/test_eval_summit_precision_v2.py),
   [tests/eval_summit_precision_v2.py](tests/eval_summit_precision_v2.py), and
   [tests/run_summit_precision_test.sh](tests/run_summit_precision_test.sh)
   all directly know about the RMS unified-call surface.
7. There is no standalone `onionskin_core/gap_analysis.py` module in this
   workspace; notebook/gap-analysis behavior is currently handled inline via
   [onionskin_core/notebooks.py](onionskin_core/notebooks.py).

## Implementation Plan

### Phase A: Lock the Contract Before Editing

1. Reconfirm the live code contract in:
   - [onionskin.py](onionskin.py)
   - [onionskin_core/detection.py](onionskin_core/detection.py)
   - [onionskin_core/per_stage_mean_shift_engine.py](onionskin_core/per_stage_mean_shift_engine.py)
   - [onionskin_core/rcn_mean_shift_engine.py](onionskin_core/rcn_mean_shift_engine.py)
   - [onionskin_core/output_layout.py](onionskin_core/output_layout.py)
2. Reconfirm the documentation contract in:
   - [README.md](README.md)
   - [multi-agent/full_instructions/PIPELINE_SPEC.md](multi-agent/full_instructions/PIPELINE_SPEC.md)
   - [multi-agent/plans/PHASE13_SPEC.md](multi-agent/plans/PHASE13_SPEC.md)
3. Make one explicit decision on threshold authority before implementation:
   - either change only the CLI default in [onionskin.py](onionskin.py), or
   - align the CLI default and any RMS engine-internal defaults so direct engine
     entry points behave consistently with controller-driven runs.
4. Reconfirm that `--z-thresh-single` and `--halfwidths-kb-single` are the live
   calibrated pair for RMS detection, then make this round's scope explicit:
   this side quest may lower the z-threshold default, but it does not retune
   the halfwidth array unless a broader follow-up is intentionally approved.
   The current halfwidth default remains `40,80,120,160,220,300` unless changed
   deliberately.
5. Record those threshold-authority and halfwidth-scope decisions in the
   implementation PR/session notes so later agents
   do not have to infer whether `2.5` is the whole-program default or only the
   controller default.

### Phase B: Fix Step-08 Final Classification

1. Refactor the RMS unification flow in
   [onionskin_core/per_stage_mean_shift_engine.py](onionskin_core/per_stage_mean_shift_engine.py)
   so step 08 no longer treats accepted stage-local calls and collapsed-repeat
   rejects as independent unification lanes.
2. Build one shared interval universe by concatenating:
   - stage-local accepted calls, labeled with provisional state `1`, and
   - stage-local shape-filter rejects written as putative collapsed repeats,
     labeled with provisional state `0`.
3. Construct canonical interval groups using the current `_unify_stage_calls()`
   overlap rule: sort by `chrom,start,end`, then merge by any-overlap into one
   cluster per overlap component. For each cluster, set `start=min(start)` and
   `end=max(end)` and retain the full contributor list.
4. Compute one state per stage within each canonical interval group using
   explicit precedence:
   - `1` if any contributor from that stage is an accepted call,
   - else `0` if any contributor from that stage is a shape-filter reject,
   - else `-1` if that stage has no contributor in the cluster.
5. Immediately after building that shared interval universe, introduce a
   dedicated helper such as `_classify_unified_amplicons()` whose input includes
   all relevant stage-local contributors, not just the accepted-call subset.
6. Preserve the full sorted run-stage order in the emitted breadcrumb trail,
   including `-1` states for absent stages.
7. Use a small explicit decision table for the first-pass final classifier.
   Recommended rules for this round:
   - all-same labels stay as-is;
   - absent/flat that transitions to triangle and stays triangle becomes final
     `amplicon`;
   - mixtures containing triangle but later reverting to flat or absent become
     final `low-confidence-amplicon`;
   - pure absent/flat mixtures become final `collapsed-repeat`.
8. Single-stage edge case: when only one stage exists, there is no temporal
   transition logic. A single-stage interval is `amplicon` if its sole stage
   state is `1`, else `collapsed-repeat`. Do not emit
   `low-confidence-amplicon` for single-stage runs in this round.
9. Emit `final_classification` as a concrete string label with exactly three
   values in this round:
   - `amplicon`
   - `low-confidence-amplicon`
   - `collapsed-repeat`
10. Low-confidence rows stay in the unified amplicon TSV/BED with
    `final_classification=low-confidence-amplicon`. Do not create a fourth
    output file for this round.
11. Return two non-overlapping DataFrames from the final classifier:
   - final unified amplicons
   - final unified collapsed repeats
12. Add an explicit invariant check that fails loudly if any interval lands in
   both final sets.

### Phase C: Extend Step-08 Output Semantics

1. Extend the unified TSV schema so both step-08 outputs carry concrete final
   classification evidence while preserving current columns for backward
   compatibility where possible.
2. Use exact column names in this round. Minimum required additions:
   - `final_classification`
   - `stage_state_trail`
   - `rescued_from_shape_reject`
   - `best_shape_score_raw`
   - `support_stage_count`
   - `reject_stage_count`
   - `contributor_stage_count`
   Define `best_shape_score_raw` as the maximum contributor `shape_score_raw`
   within the final cluster, regardless of whether that contributor was kept or
   rejected at the stage-local shape-filter step. `best_stage1_score_z` is not
   a decision-driving field for this side quest. If it is already naturally
   available in the final unified TSV, it may remain as ancillary upstream
   detection provenance, but do not add new implementation complexity solely to
   preserve it and do not use it as the step-08 BED score source.
3. `stage_state_trail` must encode one integer state per stage in the run's
   sorted stage order, for example `1,0,-1`.
4. `rescued_from_shape_reject` is `true` when a final amplicon cluster contains
   at least one stage-local reject contributor and still resolves to either
   `amplicon` or `low-confidence-amplicon`.
5. `shape_score_raw` and `best_shape_score_raw` refer to the triangle shape
   score from `dBIC_flat_vs_tri`, i.e. `BIC(flat) - BIC(triangle)`. Positive
   values favor the triangle model; lower or negative values indicate flatter /
   repeat-like structure.
6. Do not mutate `_calls_to_bed_domains()` globally for this side quest. That
   helper is used outside final RMS step-08 emission, including growth-model
   paths in the same module. Instead, add a dedicated step-08 BED writer for
   the final classified RMS outputs, or add an explicit RMS-only branch that
   leaves existing generic behavior unchanged for non-step-08 callers.
7. Step-08 BED format is concrete in this round:
   - BED name = `{chrom}:{start}-{end}|{final_classification}|{stage_state_trail}`
   - BED score source = `best_shape_score_raw` for both amplicon and
     collapsed-repeat outputs
   - BED score field = the raw unmodified `best_shape_score_raw` value; do not
       clip, rescale, or otherwise transform it for this workflow
8. Keep BED formatting compact enough for IGV and downstream BED readers.
9. Route the final classified outputs through the existing write sites in
   [onionskin_core/rcn_mean_shift_engine.py](onionskin_core/rcn_mean_shift_engine.py)
   and the legacy/per-stage write site in
   [onionskin_core/per_stage_mean_shift_engine.py](onionskin_core/per_stage_mean_shift_engine.py).
10. Do not introduce a new output directory in this round.
11. Review the step-08 to step-09 handoff in [onionskin.py](onionskin.py) so gap
   analysis continues to consume the final RMS amplicon partition rather than a
   stale pre-classification interpretation.
12. Review [onionskin_core/notebooks.py](onionskin_core/notebooks.py) so the gap
   analysis notebook remains correct if RMS TSV inputs gain breadcrumb or final
   classification columns.
13. Verify that downstream consumers either ignore the new RMS metadata columns
    or intentionally surface them without breaking:
    - [onionskin_core/aps.py](onionskin_core/aps.py)
    - [onionskin_core/profile_plots.py](onionskin_core/profile_plots.py)
    - [onionskin_core/summit_plots.py](onionskin_core/summit_plots.py)
    - [onionskin_core/shape_filter_plots.py](onionskin_core/shape_filter_plots.py)
    - [onionskin_core/qc_plots.py](onionskin_core/qc_plots.py)
    - [onionskin_core/summaries.py](onionskin_core/summaries.py)
    - [onionskin_core/io.py](onionskin_core/io.py)
    - [onionskin_core/rcn_io.py](onionskin_core/rcn_io.py)

### Phase D: Threshold Shift, If Bundled

1. Do not lower the threshold first. Stabilize the step-08 final classifier
   before changing candidate volume.
2. If the threshold shift remains in scope, apply it only after the classifier
   produces a correct non-overlapping partition.
3. If the chosen decision is to move the default to `2.5`, review all places
   where `4.0` is treated as the calibrated RMS/single default and update only
   the places that are intended to reflect the live default rather than a
   historical search baseline.

### Phase E: Verification and Closeout

1. Add direct regression coverage for:
   - a rescued interval that is stage-local reject early but final amplicon
   - a pure absent/flat interval that lands only in final collapsed repeats
   - breadcrumb trail presence and contents
   - explicit non-overlap between final unified amplicons and final unified
     collapsed repeats
2. Run the minimum risk-based validation suite:
   - `make test`
   - `make toy`
3. Add `make single` if the threshold change lands in the same round or if
   single-stage expectations materially change.
4. Add `make twin` only if interval consolidation affects overlap/twin logic.
5. Spot-check at least one chr-II dev artifact and confirm:
   - breadcrumb labels are readable
   - BED score is populated
   - rescued amplicons no longer coexist in the final collapsed-repeat output
   - final partition looks biologically plausible
6. Smoke-review APS and notebook/report surfaces if step-08 metadata grows:
   - [tests/test_aps.py](tests/test_aps.py)
   - [onionskin_core/notebooks.py](onionskin_core/notebooks.py)
   - the relevant plot modules listed in Phase C
7. After behavior is validated, update:
   - [CHANGELOG.md](CHANGELOG.md)
   - [multi-agent/project_context/HANDOFF.md](multi-agent/project_context/HANDOFF.md)
   - [multi-agent/project_context/TASK.md](multi-agent/project_context/TASK.md)

## Codebase Audit

This section distinguishes direct edit targets from review-only consumers and
generated surfaces.

### A. Direct Code Changes Required

These files are the core implementation surface and should be expected to change.

1. [onionskin.py](onionskin.py)
   - threshold default authority
   - RMS controller wiring
2. [onionskin_core/per_stage_mean_shift_engine.py](onionskin_core/per_stage_mean_shift_engine.py)
   - shared interval universe construction
   - final classifier helper
   - unified TSV schema emission
   - BED naming/score behavior
3. [onionskin_core/rcn_mean_shift_engine.py](onionskin_core/rcn_mean_shift_engine.py)
   - routed step-08 write sites
   - final non-overlapping amplicon vs collapsed-repeat emission

### B. Direct Code Review Required, Even If No Edit Is Needed

These files touch the side quest semantically and must be re-read during
implementation.

1. [onionskin_core/detection.py](onionskin_core/detection.py)
   - shared `stage1_mean_shift()` threshold behavior
2. [scripts/summit_inspector.py](scripts/summit_inspector.py)
   - review whether richer RMS step-08 metadata should be surfaced or whether
     it remains safely ignored
3. [tests/eval_summit_precision_v2.py](tests/eval_summit_precision_v2.py)
   - review how RMS unified calls are normalized and whether appended metadata
     should remain column-agnostic
4. [tests/eval_summit_precision.py](tests/eval_summit_precision.py)
   - legacy heuristic based on unified-stage-call tables; likely safe, but it
     should be smoke-reviewed if schema columns are appended
5. [tests/eval_summit_precision_v1.py](tests/eval_summit_precision_v1.py)
   - same reason as the legacy evaluator above
6. [onionskin_core/notebooks.py](onionskin_core/notebooks.py)
    - gap-analysis notebook generation is inline here rather than in a separate
       gap-analysis module
    - new RMS classification/breadcrumb columns may need to be displayed or
       safely ignored
7. [onionskin_core/aps.py](onionskin_core/aps.py)
    - review whether APS table generation stays column-agnostic when fed richer
       RMS unified-call inputs
8. [onionskin_core/profile_plots.py](onionskin_core/profile_plots.py)
    - verify plotting code tolerates richer RMS unified-call metadata
9. [onionskin_core/summit_plots.py](onionskin_core/summit_plots.py)
    - same reason as profile plots
10. [onionskin_core/shape_filter_plots.py](onionskin_core/shape_filter_plots.py)
      - same reason as the other plot consumers
11. [onionskin_core/qc_plots.py](onionskin_core/qc_plots.py)
      - same reason as the other plot consumers
12. [onionskin_core/summaries.py](onionskin_core/summaries.py)
      - review for any implicit assumptions about RMS unified-call columns
13. [onionskin_core/io.py](onionskin_core/io.py)
      - review for generic TSV/BED writing assumptions that could affect the new
         step-08 metadata
14. [onionskin_core/rcn_io.py](onionskin_core/rcn_io.py)
      - review for RMS-specific I/O assumptions about output structure
15. [onionskin_core/refinement.py](onionskin_core/refinement.py)
      - parabola-refined summit metadata feeds the unified classifier inputs and
         should be sanity-checked during implementation

### C. Tests That Directly Touch the RMS Unified Surface

These tests or harnesses know about step-08 paths, unified-call files, or RMS
selector behavior and should be considered first-class audit surfaces.

1. [tests/test_pipeline.py](tests/test_pipeline.py)
   - currently asserts existence of `08-multistage-unification/*_unified_stage_calls.tsv`
   - should gain non-overlap and metadata-aware assertions
2. [tests/test_rcn_mean_shift_selector.py](tests/test_rcn_mean_shift_selector.py)
   - directly exercises `_unify_stage_calls()`
   - likely needs extension or partial rewrite once final classification becomes
     part of unification behavior
3. [tests/test_eval_summit_precision_v2.py](tests/test_eval_summit_precision_v2.py)
   - directly fabricates RMS unified-call inputs
   - must prove appended columns do not break evaluation
4. [tests/test_rcn_summit_diagnostics.py](tests/test_rcn_summit_diagnostics.py)
    - exercises RMS summit-summary artifacts and should be reviewed if step-08
       classification or support-score metadata becomes more explicit downstream
5. [tests/run_summit_precision_test.sh](tests/run_summit_precision_test.sh)
   - drives RMS summit evaluation and selector policy sweeps
   - may need no path change, but should be rerun because RMS outputs and
     threshold defaults can shift behavior
6. [tests/optimize_single_params.py](tests/optimize_single_params.py)
   - currently encodes `z-thresh-single = 4.0` in fixed defaults and grids
   - review whether those values should remain historical search anchors or be
     realigned to the live default
7. [tests/run_full_chrII_test.sh](tests/run_full_chrII_test.sh)
   - collapsed-repeat regression is single-file focused, not step-08 focused
   - still review if the threshold change lands because candidate volume and
     collapsed-repeat exposure are part of the rationale for the side quest
8. [tests/test_aps.py](tests/test_aps.py)
    - review if APS input assumptions change when step-08 RMS TSVs gain richer
       metadata or classification columns

### D. Scripts in scripts/ That Must Be Reviewed

1. [scripts/rcn_summit_diagnostics.py](scripts/rcn_summit_diagnostics.py)
   - consumes RMS summit-summary artifacts rather than raw step-08 TSVs
   - likely does not need a path change
   - should still be reviewed so any new final-classification metadata or
     support-score naming does not silently confuse downstream interpretation
2. [tests/run_rcn_summit_diagnostic.sh](tests/run_rcn_summit_diagnostic.sh)
   - shell driver for RMS summit-diagnostic runs; rerun after step-08 output
     changes to confirm the diagnostic artifact still looks plausible
3. [scripts/summit_inspector.py](scripts/summit_inspector.py)
   - review whether new step-08 metadata should be displayed or ignored
4. [scripts/compare_puffstep_outputs.py](scripts/compare_puffstep_outputs.py)
   - explicitly out of scope for edits in this side quest
   - audit confirmed it is HMM-only and should not be changed unless a later
     separate task broadens its pipeline scope
5. [Makefile](Makefile)
    - review only if validation targets or documentation snippets need to change
       to cover the new RMS regression expectations

### E. Documentation and Contract Files That Must Be Updated or Reviewed

1. [onionskin_core/output_layout.py](onionskin_core/output_layout.py)
   - manual edit target if the textual description of step 08 becomes more
     specific about final classification, breadcrumb metadata, or BED semantics
2. [README.md](README.md)
   - review whether user-facing output-tree language needs clarification once
     step-08 semantics change
3. [multi-agent/full_instructions/PIPELINE_SPEC.md](multi-agent/full_instructions/PIPELINE_SPEC.md)
    - must be updated after implementation because this side quest changes step-08
       semantics and output contracts, and may also change the `--z-thresh-single`
       default
    - minimum review/update scope: step-08 description under
       `03-rcn-mean-shift/`, unified-call and unified-collapsed-repeat schema
       tables, BED name/score semantics, and the `--z-thresh-single` default if
       changed
4. [multi-agent/plans/PHASE13_SPEC.md](multi-agent/plans/PHASE13_SPEC.md)
   - review/update only if this side quest is promoted into active phase scope
5. [CHANGELOG.md](CHANGELOG.md)
   - required only after implementation is complete and validated
6. [multi-agent/project_context/HANDOFF.md](multi-agent/project_context/HANDOFF.md)
   - required only after implementation is complete and validated
7. [multi-agent/project_context/TASK.md](multi-agent/project_context/TASK.md)
   - required only after implementation is complete and validated

### F. Generated Documentation and Artifact Surfaces

These are not primary manual edit targets for the planning round, but their
content will change when the implementation is rerun.

1. Run-local `00-INDEX.md` files generated from
   [onionskin_core/output_layout.py](onionskin_core/output_layout.py)
2. Run-local README outputs generated under `dev/runs/...`
3. Existing checked-in toy artifacts such as [toy_out/README.md](toy_out/README.md)
   if the repo intentionally refreshes example output after implementation

### G. Verified Non-Issues / Explicit Exclusions

1. There is no live `onionskin_core/gap_analysis.py` in this workspace, so it is
   not a valid implementation surface for this side quest. Gap-analysis notebook
   behavior is currently inline in
   [onionskin_core/notebooks.py](onionskin_core/notebooks.py).
2. This plan does not include a broad RMS parameter sweep.
3. This plan does not include dynamic onset / last-active-stage origin-window
   biology.
4. This plan does not include a new final-classification directory or RMS step
   renumbering.
5. This plan does not include HMM comparator-script changes.

## Recommended Implementation Order

If the work is split across agents, this is the safest ordering:

1. Lock threshold-authority decision and final-classifier label semantics.
2. Refactor shared interval-universe construction and final step-08 classifier.
3. Add non-overlap invariant and expanded step-08 schema/BED metadata.
4. Update direct tests for the new classifier contract.
5. Apply the threshold change only if it remains in scope.
6. Update docs/specs.
7. Run validation and artifact spot-checks.
8. Update changelog/handoff/task after validation.

## Minimum Acceptance Criteria

The side quest should not be considered complete unless all of the following are
true:

1. A locus cannot appear in both final unified amplicons and final unified
   collapsed repeats.
2. Step-08 TSV outputs expose enough metadata to explain why a locus ended in
   its final class.
3. Step-08 BED outputs carry nontrivial label/score information.
4. The evaluator and summit-diagnostic surfaces still run without schema drift.
5. The chosen threshold-authority rule is explicit and documented.
6. Documentation matches the code for step-08 semantics and any default changes.

## Notes for Future Follow-Up

If this side quest succeeds, the next discussion can revisit whether a separate
final-classification step is actually warranted. That question should be driven
by real downstream pressure from gap analysis, APS, or notebook/reporting needs,
not by aesthetics alone.

## Closeout Status

Closed at `v0.13.46`.

- Minimum Acceptance Criteria were met in live code and validated in the Phase 13 implementation
   rounds recorded in `CHANGELOG.md`.
- Claude's post-implementation audit found two low-severity cleanup items only:
   an orphaned import in `onionskin_core/rcn_mean_shift_engine.py` and an optional missing pure-
   amplicon test in `tests/test_rcn_final_classification.py`.
- Both cleanup items were resolved in the closeout pass for this plan.
- Additional summit-selector improvement ideas discovered after closeout are intentionally deferred
   out of this side quest and out of Priority 13.3 closeout. The current code already computes
   parabola-refined RMS summit estimates; any future work on using those estimates more directly at
   runtime belongs to a later summit-optimization phase, not to this side quest.