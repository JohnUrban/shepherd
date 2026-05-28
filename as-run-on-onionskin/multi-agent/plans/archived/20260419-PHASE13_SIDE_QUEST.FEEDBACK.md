# Phase 13 Side Quest Feedback

## Claude feedback round 1

Audit date: 2026-04-17. Audited against live codebase. All findings below were
verified by direct file reads or grep searches before writing.

---

### What the plan gets right

The plan is consistent with the user's stated intent. Both goals (z-thresh-single
revisit + step-08 non-overlap correctness) are represented. The scope guard
decisions (no renumbering, no new directory, additive schema, classifier before
threshold shift) are sound and match the user's framing. The recommendation to
lock the threshold-authority decision before touching code is correct. The
Minimum Acceptance Criteria are well-formed and enforceable. The
Recommended Implementation Order is safe. The plan correctly identifies that
`_unify_stage_calls()` and `_calls_to_bed_domains()` are the core locus and that
the fix lives in `per_stage_mean_shift_engine.py`. Verified Contract items 1–5
and 7 are accurate against the live repo.

---

### Finding 1 — CRITICAL: `rcn_io.py` path is wrong

**Where in the plan:** Section B item 14 and Phase C item 10.

Both references read `onionskin_core/rcn_io.py`. This file does not exist at that
path. The actual file is `scripts/rcn_io.py`. Any implementer who follows the
plan literally will look in the wrong place.

**Fix:** Change both occurrences to `scripts/rcn_io.py`. Also move the entry from
Section B (direct code review — `onionskin_core/`) to Section D (scripts/ review)
where it belongs structurally.

---

### Finding 2 — Missing test surface: `tests/test_rcn_summit_diagnostics.py`

**Where in the plan:** Section C (Tests That Directly Touch the RMS Unified
Surface) — absent.

This file exists and directly exercises RMS summit-summary artifacts. It is
closely coupled to the same metadata surface that the step-08 schema extension
will affect. An implementer running the plan's Section C list will not know to
check or rerun this test.

**Fix:** Add to Section C:

> `tests/test_rcn_summit_diagnostics.py` — exercises RMS summit-summary artifacts;
> review whether appended step-08 classification or support-score columns should
> be tested or safely ignored here.

---

### Finding 3 — Missing script surface: `tests/run_rcn_summit_diagnostic.sh`

**Where in the plan:** Section D (scripts/ review) — absent.

This shell harness exists and drives RMS summit-diagnostic runs. It is the
integration-level complement to `scripts/rcn_summit_diagnostics.py` (which the
plan does list in Section D). An implementer following Section D will run
`rcn_summit_diagnostics.py` but not know about its shell driver.

**Fix:** Add to Section D:

> `tests/run_rcn_summit_diagnostic.sh` — shell driver for RMS summit-diagnostic
> runs; rerun after step-08 output changes to confirm diagnostic artifacts still
> look plausible.

---

### Finding 4 — `_calls_to_bed_domains()` is shared with the growth pipeline

**Where in the plan:** Phase C item 3 says to update `_calls_to_bed_domains()` in
`per_stage_mean_shift_engine.py` to make BED files informative.

**Problem:** `_calls_to_bed_domains()` is a shared helper. The growth pipeline
also calls it through `per_stage_mean_shift_engine.py`. If you change the
function's BED name/score behavior unconditionally, you silently change growth
pipeline BED output as well. The plan does not acknowledge this sharing or decide
how to handle it.

**Options for the implementer to choose among:**
1. Add a `pipeline` keyword argument with a default of `"rcn"` to guard
   RMS-specific enrichments behind a conditional — growth pipeline callers pass
   `pipeline="growth"` and get the old behavior.
2. Split into `_calls_to_bed_domains_rcn()` and keep the existing function for
   growth, then update RMS call sites only.
3. Confirm growth BED format can also adopt the new label/score logic and change
   both uniformly.

The plan must pick one of these and make it explicit before implementation starts.
Option 1 is the lowest-risk change for this round given the scope guard against
broad refactors.

---

### Finding 5 — Shared interval universe construction strategy is underspecified

**Where in the plan:** Phase B items 2–3 describe building "one shared interval
universe from the union of accepted calls and shape-filter rejects."

**Problem:** No construction algorithm is specified. In practice, accepted calls
and shape-filter rejects from multiple stages are likely to have overlapping
coordinate ranges. "Union" can mean:

- Merge all intervals that overlap by any amount (bedtools-merge semantics),
- Keep per-interval identity and cluster by overlap into groups, or
- Deduplicate by locus key if locus keys already encode unique intervals.

The per-stage state labels (`-1/0/1`) only make sense if there is a canonical
interval identity to attach them to. The plan is silent on what that canonical
interval is and how overlapping contributors from different stages are resolved
into a single entry in the shared universe.

**Fix:** Specify the construction algorithm explicitly. Recommended approach for
this round: use the accepted-call intervals as the primary interval set (they are
already deduplicated by `_unify_stage_calls()`), then attach shape-filter-reject
contributors as additional stage labels on matching loci. This avoids the need
for a new merge pass while preserving the existing deduplication logic. If a
reject does not overlap any accepted call, it forms its own entry in the shared
universe with a `0` or `-1` label in the stages where it was accepted.

---

### Finding 6 — "Low-confidence" label disposition is explicitly undecided

**Where in the plan:** Phase B item 7 says "Decide whether the low-confidence
state should be emitted as a separate final-classification label within the
amplicon TSV or collapsed into a simpler binary label plus confidence field. Make
that decision explicitly and keep it stable within the round."

**Problem:** The plan kicks the decision to the implementer. For an
implementation-ready plan, this decision should be made here and not left open.
An implementer who chooses differently from the user's intent will produce an
output that looks plausible but is semantically wrong.

**Fix:** Make the decision in this document. Recommended for this round: emit
`final_classification` as a string column with three values —
`"amplicon"`, `"low-confidence-amplicon"`, and `"collapsed-repeat"` — in both
the amplicon TSV and the collapsed-repeat TSV. Low-confidence amplicons go into
the amplicon TSV with the `"low-confidence-amplicon"` label so that the TSVs
remain structurally clean and IGV users can filter by name field. Do not create a
fourth output file for this round.

If the user wants a different disposition, record it here before implementation
starts.

---

### Finding 7 — Schema extension column names are not concrete

**Where in the plan:** Phase C item 2 lists "minimum recommended added columns"
as human-readable descriptions without actual column names.

**Problem:** An implementer who must write code and tests needs the exact
snake-case column names. "Final classification label," "stage breadcrumb trail,"
"best triangle-support score," and "provenance / rescue indicator" are
descriptions, not names. The test assertions in Phase E will be untestable until
names are fixed.

**Fix:** Specify exact column names. Suggested candidates (user/Copilot should
affirm or replace):

| Description | Proposed column name |
|---|---|
| Final classification label | `final_class` |
| Stage breadcrumb trail | `stage_label_trail` (e.g. `"1,0,-1"`) |
| Best triangle-support score | `best_stage1_score_z` |
| Rescue indicator | `rescued` (bool) |
| Compact per-stage evidence summary | `stage_contributor_count` (int) |

BED name field format is also underspecified — "interval id plus final label plus
breadcrumb trail" does not tell an implementer whether to use a tab, pipe, or
colon delimiter in a field that must remain BED-valid. Specify it. Recommended:
`{chrom}:{start}-{end}|{final_class}|{stage_label_trail}` as the BED name.

BED score: the plan says "best-support confidence surrogate" — name the actual
column. If `best_stage1_score_z` is the surrogate, say so. BED score must be an
integer in [0, 1000]; specify the scaling formula (e.g., `min(int(z * 100), 1000)`).

---

### Finding 8 — Single-stage edge case is not addressed

**Where in the plan:** Phase B defines the state model for multi-stage intervals.

**Problem:** The `per_stage_mean_shift_engine.py` pipeline can be run with a
single stage (e.g., a single-sample run or a toy dataset). When there is only one
stage, the breadcrumb trail has length 1, and rules like "transitions to triangle
and stays triangle" are vacuously satisfied or vacuously violated. The plan does
not say how to handle a single-stage run.

**Fix:** Add an explicit single-stage rule. Recommended: for a single-stage
interval, the label is whatever that stage's label is; there is no temporal
transition to evaluate. `final_class` is `"amplicon"` if label is 1, else
`"collapsed-repeat"`. Do not emit `"low-confidence-amplicon"` for single-stage
runs unless the stage itself emits a low-confidence score.

---

### Finding 9 — `--z-thresh-single` vs. halfwidth-array distinction not addressed

**Where in the plan:** Phase D mentions reviewing "all places where `4.0` is
treated as the calibrated RMS/single default."

**Problem:** The plan does not distinguish between the scalar z-threshold
(`--z-thresh-single`) and the halfwidth/bandwidth array used in the mean-shift
kernel. These are separate parameters. If lowering `--z-thresh-single` increases
candidate volume, that interacts with the halfwidth array and the downstream shape
filter sensitivity. An implementer who changes only the z-threshold without
checking halfwidth interactions may produce an unexpectedly noisy step-08 output
and misattribute it to the classifier refactor.

**Fix:** Add to Phase D: reconfirm the halfwidth-array defaults in
`onionskin_core/detection.py` before applying the threshold shift, and note
whether the threshold and halfwidth together constitute the "calibrated pair" for
this dataset class. This is a review step, not an implementation step, but it
should be explicit.

---

### Finding 10 — PIPELINE_SPEC.md update scope is vague

**Where in the plan:** Section E item 3 says PIPELINE_SPEC.md "must be updated if
step semantics, defaults, schemas, or BED meaning change."

**Problem:** Given that Phase B, C, and D collectively change all four of those
things, the conditionality is meaningless — PIPELINE_SPEC.md will need to be
updated. The plan should state this affirmatively and give the implementer
enough direction to scope the update correctly.

**Fix:** Replace the conditional with a concrete obligation:

> PIPELINE_SPEC.md must be updated after implementation. Sections to review:
> step-08 description under `03-rcn-mean-shift/`, unified-amplicons and
> unified-collapsed-repeats schema tables, BED format description, and the
> `--z-thresh-single` default if changed. Do not leave PIPELINE_SPEC.md
> trailing a code-level contract change.

---

### Summary table

| # | Severity | Finding |
|---|---|---|
| 1 | CRITICAL | `rcn_io.py` path is `scripts/`, not `onionskin_core/` |
| 2 | HIGH | `tests/test_rcn_summit_diagnostics.py` missing from audit surface |
| 3 | MEDIUM | `tests/run_rcn_summit_diagnostic.sh` missing from audit surface |
| 4 | HIGH | `_calls_to_bed_domains()` growth-pipeline sharing not addressed |
| 5 | HIGH | Shared interval universe construction algorithm underspecified |
| 6 | HIGH | Low-confidence label disposition left open (must be decided now) |
| 7 | HIGH | Schema column names are descriptions, not actual identifiers |
| 8 | MEDIUM | Single-stage edge case not addressed |
| 9 | MEDIUM | z-thresh-single vs. halfwidth-array interaction not flagged |
| 10 | MEDIUM | PIPELINE_SPEC.md update obligation is conditional when it should be affirmative |

None of the findings invalidate the overall plan design. The architecture is
correct. The scope guards are appropriate. Findings 1 and 4–7 should be resolved
before implementation starts; findings 2, 3, 8–10 can be resolved in the
implementation PR as long as the implementer is aware of them.

## Copilot verification response

Audit date: 2026-04-19. Re-verified against live code after Claude round 1.

### Overall conclusion

Claude's structural concerns were useful and improved the plan, but one of the
headline blocking findings is not valid in this repo.

The corrected summary is:

1. Finding 1 is not confirmed. [onionskin_core/rcn_io.py](onionskin_core/rcn_io.py)
   exists in the live workspace, and there is no `scripts/rcn_io.py` file.
2. Findings 2 and 3 are confirmed and have been added to the plan.
3. Findings 4 through 7 are confirmed in substance and have been resolved in
   the plan by making the algorithm and output contract concrete.
4. Findings 8 through 10 are also valid review additions and have been folded
   into the plan now rather than deferred.

### Finding-by-finding verification

1. **Finding 1 — not confirmed.**
   Live check results:
   - [onionskin_core/rcn_io.py](onionskin_core/rcn_io.py) exists.
   - No `scripts/rcn_io.py` file exists.
   Conclusion: the plan's original `onionskin_core/rcn_io.py` reference was
   correct for this repo.

2. **Finding 2 — confirmed.**
   [tests/test_rcn_summit_diagnostics.py](tests/test_rcn_summit_diagnostics.py)
   exists and should be part of the audit/verification surface. Added to the
   plan.

3. **Finding 3 — confirmed.**
   [tests/run_rcn_summit_diagnostic.sh](tests/run_rcn_summit_diagnostic.sh)
   exists and is the shell driver for
   [scripts/rcn_summit_diagnostics.py](scripts/rcn_summit_diagnostics.py).
   Added to the plan.

4. **Finding 4 — confirmed in substance, with one refinement.**
   `_calls_to_bed_domains()` is indeed shared beyond final RMS step-08 writes.
   The live call graph shows it is used by both:
   - RMS engine write sites in
     [onionskin_core/rcn_mean_shift_engine.py](onionskin_core/rcn_mean_shift_engine.py)
   - non-step-08 paths inside
     [onionskin_core/per_stage_mean_shift_engine.py](onionskin_core/per_stage_mean_shift_engine.py),
     including growth-model-tagged domain outputs.
   I resolved this in the plan by explicitly avoiding an unconditional global
   change to `_calls_to_bed_domains()`. The plan now calls for a dedicated
   step-08 BED writer or an RMS-only guarded path.

5. **Finding 5 — confirmed.**
   The original plan did not define the canonical interval-universe algorithm
   tightly enough. I made it explicit:
   - concatenate accepted (`1`) and rejected (`0`) contributors,
   - cluster by the existing any-overlap merge rule,
   - compute one per-stage state with precedence `1 > 0 > -1`.

6. **Finding 6 — confirmed.**
   The low-confidence disposition should not be left open. The plan now fixes
   `final_classification` to three values:
   - `amplicon`
   - `low-confidence-amplicon`
   - `collapsed-repeat`
   Low-confidence rows stay in the unified amplicon outputs for this round.

7. **Finding 7 — confirmed.**
   The schema and BED contract needed concrete identifiers. The plan now names
   the required columns explicitly and fixes the BED name/score contract.

8. **Finding 8 — confirmed.**
   The single-stage edge case is now explicit in the plan: a one-stage run maps
   state `1` to `amplicon` and anything else to `collapsed-repeat`, with no
   low-confidence class emitted in this round.

9. **Finding 9 — confirmed as a review requirement, not a separate retuning
   scope expansion.**
   Live wiring confirms `--z-thresh-single` and `--halfwidths-kb-single` are
   passed together from [onionskin.py](onionskin.py) into RMS execution. The
   plan now explicitly rechecks that pair and keeps the halfwidth array fixed
   unless a deliberate broader retuning is approved.

10. **Finding 10 — confirmed.**
    The prior wording around `PIPELINE_SPEC.md` was too conditional. The plan
    now makes the spec update mandatory and scopes the specific sections that
    must be reviewed.

### Corrected blocker summary

The statement "Two are blocking before implementation starts: Finding 1 and
Findings 4–7" does not hold as written because Finding 1 is not valid in this
repo.

The real pre-implementation blockers were the plan-concretization items:

1. shared BED-helper handling,
2. interval-universe construction algorithm,
3. low-confidence disposition,
4. concrete schema/BED identifiers.

Those issues are now resolved in the updated plan. The additional diagnostic and
review surfaces from findings 2, 3, 8, 9, and 10 have also been incorporated.

## Claude feedback round 2

Audit date: 2026-04-19.

### Finding 1 retraction

Copilot's correction is confirmed. A live glob against the repo finds
`onionskin_core/rcn_io.py` and no `scripts/rcn_io.py`. My Finding 1 was a false
positive, caused by a search confusion in the prior session context (likely
conflated with `scripts/rcn_summit_diagnostics.py`). The plan's original path was
correct throughout. I accept the correction and retract the finding.

### Plan quality assessment after Copilot's round

All substantive findings (4–10) have been correctly incorporated into the updated
plan. The plan is now materially better than the original:

- Interval-universe construction algorithm is explicit and concrete (B.2–B.4).
- Per-stage state precedence is unambiguous (`1 > 0 > -1`).
- Single-stage edge case is handled (B.8).
- `final_classification` has exactly three named values (B.9).
- Low-confidence rows stay in the amplicon TSV, no fourth file (B.10).
- All required schema column names are concrete (C.2).
- `stage_state_trail` encoding format is specified (C.3).
- `rescued_from_shape_reject` semantics are explicit (C.4).
- `_calls_to_bed_domains()` is not mutated globally; dedicated RMS step-08 path (C.5).
- BED name/score contract is concrete, including scaling formula (C.6).
- `onionskin_core/rcn_io.py` review item is correctly placed in Section B (B.14).
- `tests/test_rcn_summit_diagnostics.py` and `tests/run_rcn_summit_diagnostic.sh`
  are now listed in Section C and D respectively.
- PIPELINE_SPEC.md update is now mandatory with scoped review items (E.3).
- Halfwidth/z-thresh pair is explicitly held fixed unless broader retuning is
  approved (A.4).

### One remaining gap — collapsed-repeat BED score is undefined

**Where in the plan:** Phase C item 6 specifies:
> BED score source = `best_stage1_score_z`

The score formula works for amplicon and low-confidence-amplicon clusters: those
always have at least one accepted-call contributor (state `1` implies an accepted
stage-local call, which carries `stage1_score_z`). So `best_stage1_score_z` is
always defined for classified amplicons.

For **collapsed-repeat** clusters the situation is different. All contributors are
shape-filter rejects (state `0`) or absent (state `-1`). Rejected calls do carry
a `stage1_score_z` from detection — they were detected, then rejected downstream
at shape filtering — but the current `_unify_stage_calls()` logic only tracks
`stage1_score_z` across accepted contributors. For a pure rejected cluster there
is no existing `best_stage1_score_z`.

The implementer will need to make an explicit choice for the collapsed-repeat BED:

1. Compute `best_reject_stage1_score_z` from rejected contributors and use it as
   the collapsed-repeat BED score (makes collapsed-repeat BEDs immediately
   useful in IGV), or
2. Hard-code score `0` for collapsed-repeat BED outputs in this round (preserves
   current behavior; defers to a follow-up).

This is a low-priority decision — it does not affect the non-overlap invariant,
the amplicon BED, or the TSV schema — but the implementer should not have to
invent the answer mid-implementation. Recommend adding one sentence to Phase C
item 6 stating which option is chosen for this round. Either is fine; the gap is
the absence of a decision.

### Readiness judgment

With the one collapsed-repeat BED score decision clarified (even if the answer is
"use 0 for now"), the plan is implementation-ready. All blocking issues from
round 1 have been resolved. The architecture is correct, the scope guards are
appropriate, the classifier design is self-consistent, and the test surface is
complete. Copilot can proceed.

### Resolution note — raw triangle score approved for BED score field

User clarification after this review resolved the remaining BED-score question:

- Final step-08 BED score source should be the triangle shape/BIC score, not any
   stage-local mean-shift z-score.
- The BED score field should carry the raw unmodified cluster-level triangle
   score (`best_shape_score_raw`) for both unified amplicons and unified
   collapsed repeats.
- Do not clip, rescale, or otherwise force the score into the nominal BED6
   `0..1000` convention for this workflow.
- `best_stage1_score_z` is, at most, ancillary provenance metadata if it falls
   out naturally from the unified TSV path; it is not a classifier input and not
   the final BED score source.

## Claude post-implementation audit

Audit date: 2026-04-19. Audited v0.13.41 (step-08 classification) and v0.13.42
(threshold alignment) against the live code. All findings below were verified by
direct file reads or grep searches.

---

### Overall verdict

**Implementation is complete and correct.** All plan requirements are met.
Two minor findings require a follow-up cleanup pass; neither is a correctness
issue or a blocker.

---

### What was verified clean

All of the following were confirmed by direct code inspection:

1. **Shared interval universe**: `_classify_unified_stage_calls()` at
   `per_stage_mean_shift_engine.py:742` concatenates accepted contributors
   (tagged `_provisional_state=1`) and rejected contributors
   (`_provisional_state=0`) before calling `_group_stage_call_contributors()`.
   ✓

2. **Any-overlap clustering**: `_group_stage_call_contributors()` at line 633
   uses `start >= grouped[-1]["end"]` to detect separation — matches the plan's
   specification exactly (sort by chrom/start, merge by any overlap, start=min,
   end=max, full contributor list retained). ✓

3. **Per-stage state precedence (`1 > 0 > -1`)**: Lines 779–784 implement:
   accepted contributor → 1, else rejected → 0, else → -1. ✓

4. **`_classify_stage_state_trail()` decision rules**: Lines 724–739 correctly
   implement all four plan rules: single-stage edge case returns amplicon or
   collapsed-repeat only; all-triangle → amplicon; all-flat-or-absent → collapsed-
   repeat; first-positive-to-end-all-triangle → amplicon; else → low-confidence-
   amplicon. Logic was manually traced for several representative trails. ✓

5. **No fourth output file**: Low-confidence rows land in the amplicon TSV (line
   835–836: `final_repeats` receives only `collapsed-repeat` rows; everything
   else goes to `final_calls`). ✓

6. **Non-overlap invariant**: Lines 841–851 compute `call_keys & repeat_keys`
   and raise `ValueError` if the intersection is non-empty. ✓

7. **`_calls_to_bed_domains()` not mutated**: Line 107 is unchanged — still
   uses `r.get("bed_score", 0)` with hardcoded integer score. ✓

8. **`_calls_to_bed_domains_classified()` is a dedicated writer**: Added at line
   130, used exclusively for final RMS step-08 outputs in both engine paths.
   Growth pipeline callers continue to use the original helper. ✓

9. **BED name format**: Line 138 emits
   `{chrom}:{start}-{end}|{final_classification}|{stage_state_trail}`. ✓

10. **BED score**: `best_shape_score_raw` raw float per the resolution note. ✓

11. **Collapsed-repeat BED score defined**: `best_shape_score_raw` is computed
    from ALL contributors' `shape_score_raw` (lines 793–798), including shape-
    filter rejects. The round-2 gap is resolved. ✓

12. **Schema columns present**: All plan-required columns emitted at lines
    807–830: `final_classification`, `stage_state_trail`,
    `rescued_from_shape_reject`, `best_shape_score_raw`, `best_stage1_score_z`,
    `support_stage_count`, `reject_stage_count`, `contributor_stage_count`. ✓

13. **`stage_val` on shape-filter rejects**: Set at `rcn_mean_shift_engine.py:104`
    and `per_stage_mean_shift_engine.py:963` before shape filtering runs. Rejects
    are derived from `df_scored` which inherits `stage_val`. ✓

14. **Both engine write sites updated**: `rcn_mean_shift_engine.py` calls
    `_classify_unified_stage_calls()` at line 427 and
    `_calls_to_bed_domains_classified()` at lines 442 and 456. The per-stage
    engine calls them at lines 1177 and 1186/1196. ✓

15. **Test coverage**: `tests/test_rcn_final_classification.py` (150 lines, 2
    tests) covers: rescued interval with trail `0,1,1` → amplicon, collapsed
    interval with trail `0,-1,0`, non-overlap assertion, and late-reversion
    interval with trail `1,-1,0` → low-confidence-amplicon. ✓

16. **`tests/test_pipeline.py` metadata assertions**: Lines 456–459 assert
    `final_classification`, `stage_state_trail`, `best_shape_score_raw`, and
    `rescued_from_shape_reject` are present in unified step-08 output. ✓

17. **`--z-thresh-single` default = 2.5**: `onionskin.py:749` confirmed. ✓

18. **Engine defaults aligned**: `rcn_mean_shift_engine.py:357` and
    `per_stage_mean_shift_engine.py:1108` both default to `z_thresh=2.5`. ✓

---

### Finding A — Orphaned import (low severity)

**File:** `rcn_mean_shift_engine.py:19`

`_unify_stage_calls` is imported but never called in that file. The function was
replaced by `_classify_unified_stage_calls` for the RCN step-08 path. The import
is dead code. No runtime impact (the function still exists in
`per_stage_mean_shift_engine.py`), but it creates misleading signal: a reader
expects to find a call site and will not find one.

**Fix:** Remove `_unify_stage_calls` from the import list at line 19 of
`rcn_mean_shift_engine.py`. One-line change.

---

### Finding B — No test for pure amplicon without any rescue (minor coverage gap)

**File:** `tests/test_rcn_final_classification.py`

The two existing tests cover: a cluster rescued from shape rejects (trail `0,1,1`,
`rescued_from_shape_reject=True`), a collapsed-repeat (trail `0,-1,0`), and a
low-confidence reversion (trail `1,-1,0`). There is no test for a cluster whose
ALL contributors are accepted calls with no shape-filter reject overlapping — the
case where `rescued_from_shape_reject=False` for an amplicon, and
`stage_state_trail` has no `0` entries.

This gap does not risk regression (the logic for this path is the simpler `all 1`
branch at line 731), but the test matrix is incomplete for auditing. Low priority.

**Fix (optional):** Add a third short test with only accepted contributors — two
stages both accepted, no rejects — and assert `final_classification=amplicon`,
`rescued_from_shape_reject=False`, and trail `1,1` (or similar). Can be deferred
to the next cleanup pass.

---

### Summary

| ID | Severity | Description |
|---|---|---|
| A | LOW | Orphaned import: `_unify_stage_calls` in `rcn_mean_shift_engine.py:19` |
| B | LOW | No test for pure amplicon with zero shape-reject contributors |

Both findings are cleanup items, not correctness issues. The implementation meets
all plan Minimum Acceptance Criteria:

1. ✓ A locus cannot appear in both final outputs (invariant enforced at runtime)
2. ✓ Step-08 TSVs expose classification evidence
3. ✓ Step-08 BEDs carry nontrivial label and score
4. ✓ Evaluator and summit-diagnostic surfaces still run (validated by test suite)
5. ✓ Threshold-authority rule is explicit and documented
6. ✓ Documentation matches code (PIPELINE_SPEC.md updated in both rounds)

## Copilot closeout response

Closeout date: 2026-04-19.

Claude's post-implementation verdict is accepted.

- Finding A resolved: removed the orphaned `_unify_stage_calls` import from
   `onionskin_core/rcn_mean_shift_engine.py`.
- Finding B resolved: added a direct pure-amplicon / no-rescue classifier test to
   `tests/test_rcn_final_classification.py` so the simplest all-accepted branch is covered.

Closeout judgment:

- `multi-agent/plans/PHASE13_SIDE_QUEST.md` is now formally closed.
- No additional runtime behavior changes were made in this closeout pass.
- A later summit-specific phase may revisit the unified-call summit selector, especially the simple
   idea of emitting the parabola-refined summit from the same winning contributor instead of the raw
   peak when a parabola estimate is already available. That idea is intentionally deferred and is not
   part of this side-quest closure.
