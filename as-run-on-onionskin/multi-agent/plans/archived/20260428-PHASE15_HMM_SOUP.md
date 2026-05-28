# HMM Completeness (and Growth/RMS Pipeline updates when applicable) — SOUP

**Stage:** SOUP (pre-BRAINSTORM scratchpad in `multi-agent/plans/next/`).
**Theme:** HMM completeness + enrichment.
**Lifecycle:** Promotes to a per-phase BRAINSTORM in `multi-agent/plans/`
when the orchestrator brings it onto the stage and assigns a phase number.
See `multi-agent/AGENT_CONVENTIONS.md § Future-phase planning surfaces`
and `multi-agent/workflows/phase-development-system_PDS-v2.md` for the
SOUP → BRAINSTORM → SPEC lifecycle.

> **Maintenance:** SOUP files are intentionally early-stage and
> disorganized. Update freely as ideas surface; do not introduce
> SELF-references to phase numbers (cross-references to actually-closed
> phases ARE allowed as historical anchors — see `AGENT_CONVENTIONS.md
> § Future-phase planning surfaces`).
>
> **Standing rule (carried from earlier framing):** keep `make summit`,
> `make summit-smoke`, and `make summit-baseline` as the canonical
> summit regression / baseline surface for HMM summit quality until the
> HMM-completeness work lands.

---

## SOUP15.1 — What this SOUP is about

Finishing up HMM pipeline completeness and enriching it with the many ideas we brainstorm.
- It is to make it capable of producing its own analyses downstream of amplicon calling, including summit detection, summit refinement, APS, posterior ordering/grouping/manifest, timing, fork tracking, fork age, etc.
- It is to increase the accuracy and reliability of creating HMM posterior manifests (and corresponding growth pipeline updates when applicable)
- It is to increase the accuracy of HMM summit estimation (and corresponding growth pipeline updates when applicable)
- It is to expand analyses, plots, and notebooks in rich and useful ways for the HMM pipeline (and corresponding growth pipeline updates when applicable)

---

## SOUP15.2 — Core design rules

1. Pipeline-derived outputs belong under the pipeline that computes them.
2. Shared computation belongs in modules, not in shared emitted-analysis directories.
3. Controller/run-level artifacts must remain analysis-neutral.
4. No pipeline is the default, canonical, authoritative, or main pipeline.
5. Until explicit synthesis exists, each pipeline remains independent in both `01-prior/` and
   `02-posterior/`.
6. Cross-pipeline synthesis remains a separate explicit layer above pipeline-local products.
7. Completeness work must respect pipeline-local ownership rather than flattening differences
   into centralized intermediate files.
8. Any Phase 11 change that alters emitted paths, output schemas, or inspection-relevant
   analysis surfaces must update the affected utilities, inspectors, and regression helpers in
   the same change rather than leaving compatibility drift for later.
9. As new pipeline-local features come online, extend the relevant inspection and regression
   surfaces (for example inspectors, evaluation harnesses, and helper tests) where those
   features are now expected to be visible or testable.
10. `multistage` remains an input-data property, not the name of the growth pipeline.
11. `--pipelines all` means all three pipelines: `growth`, `per-stage`, and `hmm`.

---

## SOUP15.3 — Active carry-over decisions from late Phase 10-11

- The long-term pipeline term remains `growth-model`, while `multistage` remains the data/property term.
- The retained grouped pipeline order/names are `01-hmm/`, `02-growth-model/`, `03-rcn-mean-shift/`.
- The aligned engine/module naming direction is `hmm_engine.py`, `growth_model_engine.py`, `rcn_mean_shift_engine.py` (batch engine), and `rcn_mean_shift_singlefile_engine.py` (single-file CLI), all under `onionskin_core/engines/`. The shared RCN helpers live in `onionskin_core/rcn_mean_shift_helpers.py`.
- Until explicit cross-pipeline synthesis exists, each pipeline remains independent in prior/posterior space and should produce its own posterior manifest from its own prior outputs.
- The first synthesis target is call-level synthesis across pipeline-local prior outputs; downstream analyses can then be recomputed from the synthesized call set rather than treated as the first synthesis problem.
- The capability matrix for pipeline admissibility still needs an explicit written home in the live planning surface rather than remaining only implicit in controller behavior.

---

## SOUP15.4 — Non-goals

- Do not revive deprecated shared emitted-output ownership patterns.
- Do not approximate synthesis through centralized intermediate files.
- Do not treat growth as the owner of common analysis surfaces.

---

## SOUP15.5 — Priority 12.1 — HMM completeness

### Goal

Make HMM the first major forward implementation priority within the clean architecture.

### Priority note

HMM completeness remains ahead of per-stage completeness in the live Phase 11 queue.

### Scope statement

Complete missing HMM analysis families inside the HMM subtree only.

### Required outcome

- HMM has a concrete completeness matrix across the already-recognized analysis families.
- Missing HMM analyses are classified as reusable or HMM-specific.
- HMM completeness advances without reintroducing deprecated ownership patterns.
- The old late-Phase-10 HMM carry-over items that are still near-term priorities have an active
   home here rather than living only in historical or brainstorm surfaces.

### Near-term carry-over queue

- ~~Add HMM summit-refinement BED outputs so refinement is not TSV-only.~~ **resolved v0.10.41**
- ~~Evaluate a trimmed-mean smoothing alternative for HMM~~ — **resolved v0.10.39** (`--hmm-trim-halfwidth`).
- ~~Audit `--hmm-bin-size` auto-detection~~ — **resolved v0.10.38** (auto-detect from manifest, abort on mismatch).
- ~~Rename `--rcn-smooth-bins` → `--growth-rcn-smooth-bins`~~ — **resolved v0.10.40**.
- Potential split of `--hmm-smooth-halfwidth` into separate APS halfwidth flag (low priority; diagnose need before doing).
- **HMM summit accuracy — use stage of maximum summit RCN (`peak_rcn_stage`):** Empirically
  stages 2–4 give better origin localization than later stages because "elongation interference"
  shifts the summit peak laterally in later stages. The stage of maximum summit RCN should be
  identified per amplicon and used as the authoritative summit position source for step-14/15.
  `peak_rcn_stage` column to be added to `hmm_summit_estimates.tsv`. See BRAINSTORM entry
  "[2026-04-14] HMM summit estimation — use stage of maximum summit RCN, not last stage."
  Prerequisite: Priority 11.4 (per-sample per-stage step-5 outputs).
- **HMM sliding-offset sub-bin summit refinement:** The growth pipeline uses
  `sliding_offset_profile` / `refine_origin_sliding_offset` in `onionskin_core/refinement.py`
  (v0.5.54–v0.5.56) for sub-bin localization without needing hires data. HMM does not yet
  have this. Should be applied at the `peak_rcn_stage` step-5 smoothed bedGraphs after 11.4
  introduces per-sample outputs. See BRAINSTORM entry "[2026-04-14] HMM summit refinement —
  sliding-offset sub-bin localization." Low–medium priority; after 11.4.

### Implementation plan

1. Build the HMM completeness matrix across existing analysis families already recognized in
   planning:
   - APS
   - posterior grouping/order/manifest
   - timing
   - fork tracking
   - fork age
   - elongation asymmetry
   - refinement
   - any already-committed HMM-relevant downstream family
2. For each family, classify the missing work as:
   - already complete
   - missing but reusable from shared logic
   - missing and requiring HMM-specific implementation
3. Implement missing HMM analyses inside the HMM subtree only.
4. Reuse shared modules from 11.2 where appropriate.
5. Do not block HMM completeness on per-stage completeness, growth parity, or synthesis design.
6. Treat summit-refinement BED outputs as part of the active HMM completeness surface.
7. Treat the trimmed-mean smoothing follow-up and smoothing-flag semantics audit as near-term
   HMM analytical work rather than as distant backlog.

---

## SOUP15.6 — Priority 12.2 — HMM parallel child pipeline

**Priority: HIGH.** Prerequisite for the step-14 APS raw-file bug fix, SAPS, and amplicon
reliability scoring. Must precede 11.5.

### Goal

Run per-sample HMM steps 3–11 in independent `indiv_samples/` subdirectories so that
downstream APS and future SAPS operate on correctly normalized, per-sample track files rather
than on raw manifest bedGraph files.

### Scope statement

Steps 03-removeZeroBins through 11-ampliconMetrics currently produce only merged/joint outputs.
Adding `indiv_samples/` subdirectory outputs alongside joint outputs gives step-14 APS and the
future SAPS step access to per-sample normalized tracks that are consistent with the HMM
preprocessing pipeline (steps 1–13).

### Required outcome

- Steps 03 through 11 each emit per-sample outputs under `<step>/indiv_samples/<sample_id>.*`
  alongside existing joint outputs.
- Step-14 APS reads from `05-medianSmoothedRCN/indiv_samples/` instead of re-reading raw
  manifest bedGraph files and re-normalizing independently.
- A new step-15 SAPS (State-APS) reads decoded HMM state paths from
  `06-HMM/indiv_samples/` for per-sample state-level scoring.
- Step renumbering to accommodate SAPS: timing → `16-timing/`, clustering → `17-clustering/`.
- The step-14 APS raw-file re-normalization bug (see KNOWN_ISSUES) is closed by this change.

### Background: step-14 APS normalization bug

`run_step14_hmm_aps()` currently calls `compute_sample_rcn_tracks(manifest, ...)` which reads
raw bedGraph files from the manifest and re-normalizes them independently of the HMM pipeline
preprocessing. This is inconsistent with steps 1–13: the APS operates on un-preprocessed
signal while everything else uses the HMM-normalized, smoothed, zero-bin-removed tracks.
Fixing this requires the `indiv_samples/` outputs from step-5 to exist at APS invocation time.

### Implementation plan

1. Extend steps 03 through 11 to write per-sample outputs under `indiv_samples/` subdirectories
   alongside existing joint outputs.
2. Update step-14 APS to read from `05-medianSmoothedRCN/indiv_samples/` instead of raw
   manifest files.
3. Add step-15 SAPS using per-sample HMM state paths from `06-HMM/indiv_samples/`.
4. Renumber timing → `16-timing/` and clustering → `17-clustering/`.
5. Update all path references, helpers, and inspector utilities affected by step renumbering.
6. Add regression tests covering the `indiv_samples/` output contract.

---

## SOUP15.7 — Priority 12.3 — Amplicon reliability scoring and flat-sample detection

**Priority: HIGH.** Pre-APS diagnostic pass. Required before continued APS cluster validation
and before SAPS can be interpreted meaningfully.

### Goal

Score each called amplicon on evidence strength across stages before APS clustering, and
detect pre-amplification (flat) samples before they distort clustering.

### Scope statement

APS clustering currently receives amplicons without any pre-flight evidence filter.
Amplicons with weak or unreliable multi-stage signal, or runs that include unamplified
control-like samples, produce misleading APS cluster structure. A pre-APS reliability
pass fixes this before cluster scores are computed.

### Required outcome

- Each amplicon is scored on five evidence axes prior to APS:
  - RCN increase across stages
  - Width increase across stages
  - Area increase across stages
  - Triangle BIC increase across stages
  - Parabola height increase across stages
- A composite reliability flag is derived from these axes per amplicon.
- Flat (pre-amplification / unamplified) samples are detected and flagged before APS
  clustering runs, so that APS scores are not distorted by samples that show no
  amplification trajectory.
- A `--aps-preamp-threshold` (or equivalent) controls the flat-sample detection cutoff.
- Scores and flags are emitted as a sidecar TSV alongside APS outputs.
- APS clustering operates only on samples and amplicons passing the reliability threshold.

### Implementation plan

1. Implement a pre-APS evidence-scoring pass operating on per-amplicon stage-structured RCN,
   width, area, BIC, and parabola-height data.
2. Derive a per-amplicon reliability composite flag.
3. Implement flat-sample detection using the same evidence axes.
4. Expose `--aps-preamp-threshold` to control the flat-sample detection cutoff.
5. Emit reliability scores and flat-sample flags as sidecar outputs.
6. Wire APS clustering to use only amplicons and samples passing reliability thresholds.
7. Validate on real multi-stage data that cluster structure improves vs. unfiltered APS.




###### SOUP15.8 — OTHER
- APS needs fleshing out -- plots, for example
--hmm-0-based-statepath flag	HMM output convention change; no connection to this SOUP's goals; stays in BRAINSTORM.md
indiv_samples/ path pre-definition	Dead code for unimplemented Phase 13 feature; add when the feature lands


---


## Appended 2026-04-23 from Phase 14 Supplemental Q answers

Phase 14 Supplemental scope-engineering (Q12–Q30) surfaced several items that belong in
this future phase rather than Phase 14. Captured below for this SOUP-stage review.

Authors: Claude Code 2.1.104 (claude-opus-4-7), integrating user decisions from
`multi-agent/plans/PHASE14_SUPPLEMENTAL-FEEDBACK.md`.

---

### SOUP15.9 — 14-S15 audit findings — HMM PuffStep synonym coverage (Q15 result)

**Scope:** User directed (Q15): leave HMM PuffStep-synonym help text as-is in Phase 14
Supplemental. Audit whether help strings mention the synonym on each canonical flag; any
changes land in this future phase.

**Audit produced during Phase 14 Supplemental cycle 14S.2a** (priority 14-S14 / 14-S15).
Live-code source: `onionskin.py:build_parser()` HMM group, lines 1278–1706 (verified
2026-04-25).

| Canonical flag | PuffStep synonym | Help mentions synonym? | Synonym still registered as argparse alias? | Future-phase follow-up |
|---|---|---|---|---|
| `--hmm-expected-background-length` (line 1360, default=1_000_000) | `--hmm-expected-special-length` | YES — line 1370: "Also accepted as `--hmm-expected-special-length` (PuffStep synonym)." | YES — argparse alias on line 1360 | Working as intended; this future phase decides whether to retire the synonym (less biologically informative than `--hmm-expected-background-length`) or keep it |
| `--hmm-expected-amp-step-length` (line 1373, default=75_000) | `--hmm-expected-other-length` | YES — line 1382: "Also accepted as `--hmm-expected-other-length` (PuffStep synonym)." | YES — argparse alias on line 1373 | Same — `expected-other-length` is generic; this future phase decides |
| `--hmm-background-idx` (line 1386, default=0) | `--hmm-special-idx` | YES — line 1399: "Also accepted as `--hmm-special-idx` (PuffStep synonym)." | YES — argparse alias on line 1386 | Same |
| `--hmm-init-background` (line 1402, default=0.997) | `--hmm-init-special` | YES — line 1411: "Also accepted as `--hmm-init-special` (PuffStep synonym)." | YES — argparse alias on line 1402 | Same |
| `--hmm-leave-background-state` (line 1414, default=None) | `--hmm-leave-special-state` | YES — line 1427: "Also accepted as `--hmm-leave-special-state` (PuffStep synonym)." | YES — argparse alias on line 1414 | Same |
| `--hmm-leave-amp-step` (line 1430, default=None) | `--hmm-leave-other` | YES — line 1444: "Also accepted as `--hmm-leave-other` (PuffStep synonym)." | YES — argparse alias on line 1430 | Same |
| `--hmm-emission-model` (line 1497, default="normal") | `--hmm-emodel` | YES — line 1506: "The PuffStep synonym `--hmm-emodel` is now deprecated and redirects here." | NO — already retired (handled by `_DEPRECATED_FLAGS` pre-parse gate in `onionskin.py`) | Already retired (Phase 14 / cycle 14S.1a v0.14.68) WHICH WAS A MISTAKE. An agent made this decision unilaterally, and it was unintended by the user. No synonyms should be retired. All synonyms should be available |

**Summary of audit findings:**

- **6 of 7 canonical pairs** (the first six rows): the PuffStep synonym is BOTH still
  registered as a working argparse alias AND mentioned in the canonical flag's help text.
  This is consistent with user direction Q15 (leave as-is for Phase 14 Supplemental).
- **1 of 7 pairs** (`--hmm-emodel`): already retired via the `_DEPRECATED_FLAGS` pre-parse
  gate during cycle 14S.1a (v0.14.68). The canonical flag's help text describes the
  deprecation explicitly. This pair is the closeout pattern this future phase may follow if it
  retires further synonyms.
- **No "missing mention" cases found.** Every active synonym is documented in the
  canonical flag's help text.
- **No stale or undocumented aliases found.** Every argparse alias has a corresponding
  help-text mention.

**Future-phase decision surface:**

1. **Keep all synonyms.** Lowest-friction; preserves PuffStep continuity. Retrieve any that have been "deprecated" including `--hmm-emodel`. Synonyms should NOT be deprecated. They do not have to be our main advertised flag, but we should keep them to make historical comparisons to PuffStep results and commands easier to figure out.

2. **Help-text rewrites.** Tighten the wording of the synonym mention without
   actually retiring the alias. 

The `--hmm-emodel` retirement (Phase 14 Supplemental cycle 14S.1a v0.14.68) was a mistake not authorized by the user. An agent decided to do this on its own.

---

### SOUP15.10 — 14-S21 audit — `--hmm-0-based-statepath` + adaptive-default design

**Scope:** User directed (Q21): defer implementation to this future phase; produce audit findings
now to save this future phase's time.

**User-added design detail:** When `--hmm-0-based-statepath` is set AND the user has NOT
explicitly set `--hmm-thresh-state` or `--hmm-max-state-thresh`, shift their defaults
accordingly. If user explicitly sets them, honor what they give. Update both flags'
docstrings to explain this adaptability.

**Current live state (audited in `onionskin.py:build_parser()`, 2026-04-23):**

| Flag | Current default (1-based) | 0-based equivalent |
|---|---|---|
| `--hmm-thresh-state` | `1` ("states > 1 are amplified; state 1 is CN=1 background") | `0` ("states > 0 are amplified; state 0 is CN=1 background") |
| `--hmm-max-state-thresh` | `0` (sentinel: no filter; discard regions whose max state ≤ 0) | `-1` (sentinel: no filter; discard regions whose max state ≤ -1) |

*Note:* user's Q21 answer mentioned `--hmm-max-state-thres` without the final `h`; the live
flag is `--hmm-max-state-thresh`. Use the live spelling.

**Files that would need to change when flag is added + wired:**

1. `onionskin.py:build_parser()` HMM group:
   - Add `--hmm-0-based-statepath` (store_true, default False). Help text explains the
     2^state copy-number mapping advantage.
   - Update `--hmm-thresh-state` and `--hmm-max-state-thresh` help strings to explain
     the adaptive-default behavior under `--hmm-0-based-statepath`.
   - Consider: use `argparse.SUPPRESS` as the default for the two threshold flags so we
     can distinguish "user omitted" from "user set to the numeric default." This is the
     same pattern used elsewhere in the parser for "unset inherits override" behavior.
2. `onionskin.py:main()` (post-parse argv):
   - After `parse_args()`, detect the `args.hmm_0_based_statepath` boolean.
   - If True AND `args.hmm_thresh_state` is unset (SUPPRESS sentinel), assign 0.
   - If True AND `args.hmm_max_state_thresh` is unset, assign -1 (or None with downstream
     handling).
   - If False, assign the existing defaults 1 and 0 respectively.
3. HMM engine state-path emit code paths:
   - `onionskin_core/engines/hmm_engine.py` — where state paths are written to disk
     (`_step6_hmm()` bedgraph emit; any other writer).
   - `onionskin_core/hmm_core.py` — `_posterior_path()`, `_viterbi()`, any state-index
     returner. These can stay 1-based internally; the shift applies at the write boundary.
   - Preferred implementation: shift at the final write step, not internally. Core stays
     1-based; only output files carry 0-based indexing when the flag is set.
4. Downstream consumers of state-path values:
   - `onionskin_core/hmm_summits.py` (`extract_summits`, `_pick_summit`,
     `_merge_and_pick`) — reads state values to decide summit state.
   - `onionskin_core/hmm_metrics.py` — reads state values for nesting-level metrics.
   - `onionskin_core/hmm_fork_travel.py` — reads state values for fork-travel metrics.
   - Each of these must either read from the internal 1-based path OR apply the shift in
     the same consistent boundary.
   - Decision for this future phase: pick whether the flag affects ONLY the emitted bedgraph, OR
     also all metric outputs. User likely wants the emitted bedgraph to match the metrics
     tables — so both surfaces shift together.
5. Tests (`tests/test_pipeline.py`):
   - Any assertion on a specific state-value in an emitted bedgraph or metrics table needs
     to gate on the flag.
6. Docs:
   - `multi-agent/full_instructions/PIPELINE_SPEC.md` — state-path description section.
   - `multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md` — same.
   - README if it describes state-indexing.

**Dependencies:** None for adding the flag. Implementation touches the HMM engine, metrics,
and fork-travel modules — ideally bundled with other this SOUP's HMM completeness work.

**Potential complications:**
- `_shape_filter` or APS calls that use HMM state values need the shift too.
- Existing test baselines (snapshots of bedgraph values) will diverge by -1 when the flag
  is active. Tests need conditional baselines or two baseline sets.
- PuffStep-compatible outputs (gold-standard comparisons via `make puff-compare`) must
  remain 1-based unless PuffStep is also updated — so the `--hmm-0-based-statepath` flag
  must NOT be active during `make puff-compare` runs.

**Remove from `BRAINSTORM.md` when this future phase lands this:** the `[2026-04-18]
--hmm-0-based-statepath future CLI flag` entry.

---

#### Test enumeration addendum (added 2026-04-25, cycle 14S.2a Role 2)

The original 14-S21 audit (2026-04-23) noted that `tests/test_pipeline.py` would need
state-value assertions gated on `--hmm-0-based-statepath` but did not enumerate specific
sites. Role 2 ran the audit's suggested grep and the broader sweep below to enumerate all
test sites that touch state-path or state-value semantics:

```bash
grep -n "state" tests/test_pipeline.py | grep -iE "assert|expect|value"
grep -rn "hmm.*state\|state_path\|posterior.*path\|viterbi.*state" tests/ | grep -iE "assert|==|!="
grep -rn "hmm_thresh_state\|hmm_max_state_thresh\|hmm_state_path\|stage_state_trail\|0-based\|1-based" tests/
```

**Findings:**

| File:line | Assertion | Affected by `--hmm-0-based-statepath`? |
|---|---|---|
| `tests/test_pipeline.py:759` | `assert "stage_state_trail" in text, out` | NO. Checks column-name presence only; `stage_state_trail` is a per-developmental-stage classification trail (RCN-mean-shift output, computed in `onionskin_core/rcn_mean_shift_helpers.py:_classify_stage_state_trail()`) — NOT an HMM state-path. Unaffected by HMM 0-based indexing |
| `tests/test_rcn_final_classification.py:85` | `assert rescued["stage_state_trail"] == "0,1,1"` | NO — same reason; this is RCN classification state, not HMM state-path |
| `tests/test_rcn_final_classification.py:96` | `assert collapsed["stage_state_trail"] == "0,-1,0"` | NO — same |
| `tests/test_rcn_final_classification.py:150` | `assert row["stage_state_trail"] == "1,-1,0"` | NO — same |
| `tests/test_rcn_final_classification.py:196` | `assert row["stage_state_trail"] == "1,1"` | NO — same |
| `tests/test_hmm_ported_analyses.py:134` | `qc_nb = nb_dir / "hmm_state_path_qc.ipynb"` | NO — filename existence check only; no value assertion |

**Conclusion of enumeration:** No existing test in `tests/` directly asserts on raw HMM
state-path integer values. The only "state" assertions in `tests/test_pipeline.py` and
`tests/test_rcn_final_classification.py` reference `stage_state_trail`, which is RCN-mean-shift
classification state and is NOT affected by `--hmm-0-based-statepath`. This means the
0-based-statepath flag, when implemented, would primarily impact:

1. Output bedgraphs from `_step6_hmm()` (the emit boundary).
2. Metrics tables that consume HMM state values (`hmm_metrics.py`, `hmm_summits.py`,
   `hmm_fork_travel.py`).
3. `make puff-compare` baseline comparisons (gold-standard PuffStep outputs are 1-based;
   the audit's rule "flag must NOT be active during `make puff-compare` runs" is correct).

When this future phase lands `--hmm-0-based-statepath`, **new tests will be needed** rather than
existing tests being gated. Suggested additions:

- A focused test under `tests/test_hmm_ported_analyses.py` (or a new
  `tests/test_hmm_zero_based_statepath.py`) that runs an HMM bedgraph emission with the
  flag set and asserts the emitted state values are shifted by -1 vs the 1-based default.
- A test verifying the adaptive default for `--hmm-thresh-state` (1 → 0) and
  `--hmm-max-state-thresh` (0 → -1) when the flag is set without explicitly providing
  these companion flag values.
- A test verifying that explicitly-set `--hmm-thresh-state N` is honored as-is regardless
  of `--hmm-0-based-statepath` (the user-set-overrides-default semantic).

**Live-code verification 2026-04-25:** All file/function references in the original
14-S21 audit (above this addendum) were re-verified against live code by Role 2 during
cycle 14S.2a:

- `--hmm-thresh-state` and `--hmm-max-state-thresh` defaults and help strings: verified
  at `onionskin.py:1606,1610` and `onionskin.py:1648,1652` (live spelling
  `--hmm-max-state-thresh` with final `h` confirmed; user's Q21 typo `--max-state-thres`
  noted).
- `_step6_hmm()`: verified at `onionskin_core/engines/hmm_engine.py:551`.
- `_posterior_path()`: verified at `onionskin_core/hmm_core.py:337`.
- `_viterbi()`: verified at `onionskin_core/hmm_core.py:244`.
- `extract_summits` / `_pick_summit` / `_merge_and_pick`: verified at
  `onionskin_core/hmm_summits.py:256, 183, 212`.
- `onionskin_core/hmm_metrics.py`, `onionskin_core/hmm_fork_travel.py`: both files exist.

The audit's design recommendations (use `argparse.SUPPRESS` to detect "user omitted",
shift at the write boundary not internally, gate test baselines, exclude flag from
`make puff-compare` runs) all remain valid. The 14-S21 audit is current and ready for
this future phase implementation.

---

### SOUP15.11 — 14-S22 follow-ups — `--aps-area-excess-floor` analytical testing

**Deferred from Phase 14 Supplemental 14-S22** (CLI flag added in Phase 14 Supplemental
with default=on; the analytical work deferred to this future phase per user Q22 answer).

**future-phase priorities to spec:**

1. **Default-flip test:** Test `--aps-area-excess-floor off` on live datasets. Evaluate
   impact on APS cluster separation, posterior groupings, and summit evaluation (II/9A,
   II/2B). Decide whether to flip the default to `off`.
2. **Post-sum-floor variant:** User's concern #1 in Q22 answer: the per-bin floor at 0
   systematically biases APS high when RCN oscillates around 1. A better implementation:
   allow negative per-bin contributions during the sum, then floor the per-amplicon total
   at 0 if it ends up negative. This is an alternative wiring for
   `--aps-area-excess-floor off` — not a new flag, but a different internal implementation.
   Test both wirings side-by-side.
3. **Shape-based clustering implications:** User's concern #2: flooring hurts
   shape-similarity feature computation more than APS score computation. When
   `--aps-feature shape` is active, the floor may have an outsized effect on cluster
   recovery. Test `--aps-feature shape` with floor on vs off on datasets where posterior
   groupings are known.
4. **Validation criteria for any default change:** "No M-shaped posterior amplicons;
   summit evaluations improve or at least stay the same; no regressions." (User quote.)

---

### SOUP15.12 — 14-S22 related — Pre-amplification sample detection / "flat" sample classification

User-directed late-Phase-15 priority (from Q22 answer):

> "Part of that will also be identifying truly 'flat' non-amplification samples for 'stage
> 1', then clustering the rest into automated groups."

**Overlaps with:**
- KNOWN_ISSUES.md `[ISSUE:2026-04-14:1]` Stage-1 pre-amp isolation (already in
  `multi-agent/tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` as IBM-C8B)
- BRAINSTORM.md `[2026-04-19]` Amplicon reliability scoring and flat-sample detection
  before APS/clustering (IBM-C3)

These three items merge into a single future-phase priority: pre-APS reliability scoring,
flat-sample detection, stage-1 anchoring.

---

### SOUP15.13 — 14-S20 related — cross-pipeline dedup code unification

**If** 14-S20's `--dedup-dist` verification reveals that different pipelines implement dedup
via different code paths, the unification belongs to this future phase (user Q20: "we have an
ongoing attempt to unify code and make it non-redundant").

this SOUP to include:
- Dedup code path inventory.
- Shared dedup module (candidate: `onionskin_core/dedup.py`).
- Per-pipeline call-site migration plan.

---

### SOUP15.14 — 14-S26 follow-ups — HMM shape-score wiring

Deferred from Phase 14 Supplemental 14-S26 (RMS wired, HMM stays placeholder per Q26).

**HMM shape-score wiring items:**

1. **Implement the HMM shape-filter sink.** The RMS pipeline has
   `shape_filter_calls()` / `apply_shape_filter_tsv()` producing `shape_score_raw` from
   `dBIC_flat_vs_tri`. HMM does not have an equivalent call-time sink. Build one:
   - Decide the input: per-amplicon stage-median RCN profile within the HMM amplicon
     interval.
   - Implement `dBIC_flat_vs_tri` equivalent for HMM data (reuse RMS/growth code where
     possible).
   - Emit `shape_score_raw` column in an HMM output file (new or existing).
   - Wire `--hmm-shape-score-threshold` to consume it.
   - Wire `--hmm-shape-score-strict-bic` to toggle strict-BIC mode.

2. **HMM meta-analysis of amplicon shapes across stages** (user Q26 quote):
   > "Add to HMM what we do for RMS with the meta-analysis of amplicon shapes across stages
   > to get a final set of amplicons and collapsed repeats."
   - Port RMS's cross-stage shape meta-analysis to HMM.
   - Emit a final set of amplicons (passing the meta-filter) and putative collapsed
     repeats (failing).

3. **HMM-specific: detect state-path growth as shape credibility signal** (user Q26):
   - HMM state paths encode nesting levels directly.
   - Growing state paths (nesting depth increases across stages) → high credibility.
   - Flickering state paths (level appears/disappears across stages) → low credibility
     ("ghost levels" from BRAINSTORM.md [2026-04-18]).
   - This is a genuinely HMM-native signal not available from RMS or growth pipelines.
   - Becomes a new HMM credibility feature that feeds the final amplicon set.

---

### SOUP15.15 — 14-S20 related — APS Universal-in-spirit re-framing summary

Per user Q20: APS stays in its own parser group; header re-framed to make it explicit that
APS is applied by all three pipelines. This is a Phase 14 Supplemental item (14-S20), not a
future-phase item, but the user noted that as APS work develops in this future phase, the same framing
logic should extend to:
- Timing group (14-S13 already does a step-mention-only pass)
- Shape scoring group (already Universal)
- Overlap group (already Universal-in-spirit per 14-S20 + Finding 7)
- Asymmetric Triangle Model group (already Universal-in-spirit per 14-S3)

future work on HMM may add new HMM-specific APS flags or sub-APS (SAPS) flags. When that
happens, apply the same re-framing to any new APS-like parser group.

---

### SOUP15.16 — Other HMM development items carried over from pre-14 recon audit

Cross-referenced in `multi-agent/tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md`:

- **IBM-C2** — HMM parallel child pipeline (steps 3–11 `indiv_samples/` outputs) + SAPS +
  step-14 APS raw-file fix. HIGH priority from BRAINSTORM [2026-04-19].
- **IBM-C4** — HMM `peak_rcn_stage` column + sliding-offset sub-bin summit refinement.
  Depends on IBM-C2.
- **IBM-C5** — Cross-pipeline dynamic origin-detection window. Design-mode work.

These three are the obvious HMM-completeness anchor items; 14-S21, 14-S22 follow-ups,
14-S26 follow-ups, and 14-S15 audit all fold in alongside them.





## SOUP15.17 — True state means
At the time of writing this `--hmm-mu-scale` seems to be wrong in its help string.
It says:

```
  --hmm-mu-scale FLOAT  Uniform scale factor applied to all emission means
                        before HMM decoding (step 6). Useful for adjusting the
                        emission model when your RCN values differ in overall
                        magnitude from the default means (1,2,4,8,...).
                        Default: 1.0 (no scaling). Also rescales the sigmas by
                        the same factor. [hmm: 06-HMM] (default: 1.0)

## For convenience (corresponds to --mu and --sigma below)
  --hmm-emission-means FLOATS
                        Comma-separated HMM emission distribution means, one
                        per state. Default: 1,2,4,8,16,32,64,128 (8 states
                        covering CN=1 through 128-fold amplification in powers
                        of 2, typical for developmental re-replication). The
                        number of states is inferred from this list. State 1
                        (index 0) represents un-amplified background (CN=1);
                        higher states represent increasing copy number. For a
                        normal emission model these are the Gaussian means;
                        for Poisson/exponential they are lambda/B
                        respectively. [hmm: 06-HMM] (default:
                        1,2,4,8,16,32,64,128)
  --hmm-emission-sigmas FLOATS
                        Comma-separated HMM emission distribution sigmas
                        (standard deviations), one per state. Default:
                        0.25,0.5,1,2,4,6,8,24. Must have the same number of
                        entries as --hmm-emission-means. Only used by the
                        normal and gamma emission models; ignored by
                        Poisson/exponential/geometric. Alternative: use --hmm-
                        mu-scale to set sigmas as a fraction of the means
                        (e.g. --hmm-mu-scale 0.5 sets sigma = 0.5 * mean for
                        every state). [hmm: 06-HMM] (default:
                        0.25,0.5,1,2,4,6,8,24)

```



- I like the concept of ensuring the means actually make sense with the RCN values but that is not what that flag did in PuffStep.
- It descends from this PuffStep help string (--mu_scale), and I do not think its behavior changed:
```
  --mu_scale MU_SCALE   See --sigma for more details on sigmas. Use this to
                        scale means (--mu) to use as stdevs (sigma) instead of
                        taking square roots of means. For example, --mu_scale
                        0.5 will use mu*0.5 as the stdev.

## For convenience:
  --mu MU, --discreteEmat MU
                        PuffCN has been optimized for mapping DNA puffs in the
                        fungus fly. The default state means were previously
                        hard-coded. This option allows some flexibility from
                        the command-line to change the state means. Default:
                        1,2,4,8,16,32,64 To change: Provide comma-seprated
                        list of state means. The number of states will be
                        calculated from this list. If changing state sigmas
                        (used in normal model), it must have same number of
                        states represented. NOTE: If using exponential or
                        geometric distribution, provide the expected mean RCN
                        values of the states as you would for normal or
                        poisson models. This script will automatically take
                        their inverses to work in the exponential and
                        geometric models. NOTE2: For "--emodel discrete" can
                        call --mu as --discreteEmat for better-readability at
                        commandline. Instead of a comma-sep list of means,
                        provide comma-/semicolon-separated values to make up a
                        nState X nSymbol matrix. Example of a 3-state x 4
                        symbol matrix: "0.97,0.01,0.01,0.01;0.01,0.97,0.01,0.0
                        1;0.01,0.01,0.01,0.97". That can be thought of as 3
                        4-sided dice.
  --sigma SIGMA         PuffCN has been optimized for mapping DNA puffs in the
                        fungus fly. The default state sigmas (stdevs) were
                        previously hard-coded. This option allows some
                        flexibility from the command-line to change the state
                        sigmas. Default: if not changed, defaults to square
                        root of state means (Poisson-like). To change: Provide
                        comma-seprated list of state sigmas. Alternatively:
                        Use --mu_scale (default False) with a scaling factor
                        multiplied against the MUs. The number of states is
                        calculated from this state mean list, which defaults
                        to 7. If changing state sigmas (used in normal model),
                        it must have same number of states represented as
                        state means.
```

- The "mu scale" flag only ever scaled means by a constant to create sigmas. That's it. 
- It is not updating the means to match RCN values
- Having said that, there could be something we do to learn the RCN mean values of what is determined to be background regions on each chromosome, and redoing the chromosome-specific normalization such that the median and/or mean of background regions = 1 as expected.
	- the median norm and chromosome-specific median norm (ref-stage or not) steps should have already gotten really close to this

---

## SOUP15.18 — CORRECTNESS BUG: --hmm-mu-scale translation from PuffStep is wrong + full HMM translation re-audit required

### The bug

`--hmm-mu-scale` was translated from PuffStep incorrectly. In PuffStep, `--mu_scale`
**never touched the means** — it was an alternative method for deriving sigmas:
`sigma = mean * mu_scale` for each state (used when neither `--sigma` nor the
sqrt-of-means default was desired). It was mutually exclusive with `--sigma`.

In onionskin, the translation agent reversed this: `--hmm-mu-scale` now **scales the
means** (`scaled_means = [m * mu_scale for m in emission_means]`) and does nothing to
sigmas. The sigmas are always taken from `--hmm-emission-sigmas` unchanged. The help
text even claims "Also rescales the sigmas by the same factor" — which is doubly wrong
(means are scaled, not sigmas; and sigmas are not touched at all).

Additionally, because `--hmm-emission-sigmas` always has a hardcoded default in
onionskin (`0.25,0.5,1,2,4,6,8,24`), even a corrected implementation would need to
rethink priority logic: in PuffStep, `mu_scale` was reachable only when `sigma` was
`None`; in onionskin, sigma is never `None`. The fix requires both code correction and
a design decision on override priority (`--hmm-emission-sigmas` explicit vs.
`--hmm-mu-scale` derived).

The `--hmm-emission-sigmas` help text is also wrong: it cross-references `--hmm-mu-scale`
as "sets sigma = 0.5 * mean for every state" — which has never been true in onionskin.

### Required action: full PuffStep → onionskin HMM translation re-audit (Opus, Max Effort)

This bug was introduced silently by an agent during the PuffStep → onionskin translation.
It raises the question of whether other HMM flags were similarly mistranslated. A
dedicated re-audit is required:

1. **Code correctness audit** — for every HMM engine flag in `onionskin.py:build_parser()`
   and every corresponding parameter in `onionskin_core/engines/hmm_engine.py`, verify
   that what the code actually does matches what PuffStep did. Pay particular attention
   to any flag whose behavior was described in terms of another flag (e.g., mu_scale
   described in terms of sigma), flags whose defaults changed, and flags whose types
   changed. Use `PuffStep/puffStep_core/hmm_fxns.py` and `PuffStep/puffStep.py` as the
   authoritative reference.
2. **Fix code first** — correct any mistranslated behavior before touching help strings.
3. **Help string audit** — after code is confirmed correct, audit all HMM help strings
   for accuracy. This includes removing false cross-references (e.g., the `--hmm-emission-sigmas`
   claim about `--hmm-mu-scale`).

Assigned agent: **Opus, Max Effort**. This is a correctness-sensitive audit that must
not be delegated to a lighter model.

### Missing PuffStep synonyms (to add as part of this phase's CLI flag work)

PuffStep flags that were significantly renamed in onionskin currently have no synonym
aliases. These should be added so that users migrating from PuffStep do not need to
translate their command lines:

| PuffStep flag | Onionskin flag | Missing synonym to add |
|---|---|---|
| `--mu` / `--discreteEmat` | `--hmm-emission-means` | `--hmm-mu` and `--hmm-discreteEmat` |
| `--sigma` | `--hmm-emission-sigmas` | `--hmm-sigma` |
| `--path` / `-p` | `--hmm-decode-path` | `--hmm-path` |
| `--constrainEmit` | `--hmm-constrain-emit` | `--hmm-constrainEmit` |

Flags already covered by existing synonyms (no action needed):
`--hmm-special-idx`, `--hmm-init-special`, `--hmm-leave-special-state`,
`--hmm-leave-other`, `--hmm-expected-special-length`, `--hmm-expected-other-length`,
`--hmm-emodel` (deprecated → redirects to `--hmm-emission-model`).

Flags that were just `--hmm-`-prefixed with no other rename (kmeans, iters, converge,
emitpseudo, learnpseudo, transprobs, exp-decay, initialprobs) do not need synonyms —
the `--hmm-` prefix is predictable enough.
	- but it could be better matched by updating perhaps








## SOUP15.19 — Clustering defaults
- A while ago, we started optimizing for what works best for clustering
	- summit vs width vs area vs shape vs log2 and log10 versions of some of those
	- elbow vs keep 
	- singleton guard or not
-	- etc
- we need to finish that in this phase, especially as it relates to HMM results
- last I saw, HMM is still using elbow and area, and still being collapsed into 2 groups


## SOUP15.20 — SAPS
- Statepath APS score analyses as described elsewhere


---

## SOUP15.21 — Lift shared gap mask + missingness diagnostic to a pre-pipeline step (HMM-motivated; benefits all three pipelines via inheritance)

**Context (verified in current code, 2026-04-27):**

Two pieces of "where is there no usable data" logic currently live in
two places by historical accident — they're effectively the same
concept:

- **Growth-pipeline diagnostic + inline mask**, in
  `onionskin_core/engines/growth_model_engine.py`:
  - `_report_missingness()` (lines 77–94) emits per-chrom
    `[onionskin] missingness chrom=II: bins_all_nan=0.0081` log
    lines. Diagnostic only — does not produce a reusable artifact.
  - Inline `frac_bad >= 0.9` masking (lines 1259–1262) sets bins
    to NaN where ≥90% of samples have either NaN-from-transform or
    original-zero values. Used internally by growth; not surfaced
    as a shared artifact.
- **Gap-mask machinery** in `onionskin_core/gap_analysis.py`:
  - `build_gap_mask_from_bedgraph()` — **single-file** variant.
    Treats a bin as "gap" when one bedGraph's value is 0 there.
  - `build_shared_gap_mask_from_bedgraphs()` — **multi-file
    intersection** variant. Treats a bin as "gap" only when ALL
    files are 0 there.
  - `annotate_calls_with_gap_distance()` — post-call annotation,
    emits `_calls.gap_annotated.tsv` + `_calls.gap_analysis.tsv` +
    `.gap_report.md`. Currently calls
    `build_gap_mask_from_bedgraph()` (single-file variant)
    internally.

Important framing note: bins start as numbers in bedGraphs (the user
confirmed). NaNs only appear during analysis (`log2(0)`, division-by-
zero, etc.). Growth's `frac_bad` check at line 1261 is
`~np.isfinite(Y) | zero_mask` — it covers both transform-introduced
NaNs and original zeros with the same logic. So "missingness" and
"gap" are mostly **the same concept**; the split into two
pieces of code with different terminology is a historical artifact.

**Current cross-pipeline state:**

- **Growth pipeline:** runs the inline `frac_bad` masking + the
  diagnostic missingness log + a post-call
  `annotate_calls_with_gap_distance()` (in `onionskin.py:520`),
  which internally calls `build_gap_mask_from_bedgraph()`
  (single-file) on `_mask_bg` (manifest sample 0 by default).
- **RMS pipeline:** runs `annotate_calls_with_gap_distance()` (in
  `onionskin.py:2965`) on its unified calls. Internally re-builds
  the same single-file gap mask from `_mask_bg` from scratch,
  independent of any work growth already did. **So when both
  pipelines run, the same single-file gap mask is built twice on
  the same input.**
- **HMM pipeline:** has two paths.
  - **ref-stage normalization mode (main path):** removes zero
    bins from the bedGraph before HMM runs, so the HMM never sees
    them. This addresses a math constraint of the ratio step (zero
    denominators break the ratio); zero-bin removal has proven
    more accurate in practice than pseudocount workarounds. **This
    is the right approach for HMM in ref-stage mode and stays
    as-is.**
  - **no-ratio Step-3 alternate branch** (`_step3_remove_gap_bins`
    in `hmm_engine.py:325`): uses
    `build_shared_gap_mask_from_bedgraphs()` — the multi-file
    intersection variant — to identify and remove gap bins.
- HMM does NOT currently use `annotate_calls_with_gap_distance()`
  for any post-call gap-aware annotation of HMM calls.

### The principled approach is the multi-file intersection

`build_shared_gap_mask_from_bedgraphs()` (intersection across all
input files) is the principled way to identify gaps: a bin is a gap
only when every file agrees it's zero, which is what makes a gap a
gap (systematic missing data — assembly issue, mappability black
hole, etc.). Per-file zero intervals from a single bedGraph can be
sample-specific dropouts that have nothing to do with the assembly.

So the multi-file/shared/intersection approach **is not "more
conservative" in a hedging sense** — it's just better. It avoids
treating per-file dropouts as gaps. HMM's no-ratio branch already
does it the right way; growth and RMS do not.

This means growth and RMS can benefit from the same approach: pick
up the multi-file intersection mask instead of the single-file mask
they currently use, and they get more accurate gap classifications
in their post-call annotations.

### Pre-pipeline placement is the right home

Once we agree the multi-file intersection mask is the right gap
definition, building it once at controller level — after bin-size
detection, before any pipeline runs — falls out for free as the
clean architecture:

- All three pipelines that want gap data can inherit from one
  computation.
- No risk of pipelines diverging on what counts as a gap.
- The diagnostic missingness log (currently growth-pipeline-internal)
  can also lift pre-pipeline since it's purely informational —
  there's no reason to surface it from inside one pipeline only.

**General architectural principle:** anything shared by two or more
pipelines that doesn't depend on pipeline-internal upstream work
can come out in front, run once, and be inherited. Bin-size
detection already follows this pattern. Gap-mask construction +
missingness diagnostic should join it.

**Performance is not the motivation.** Gap-mask construction is
fast even when it runs twice per multi-pipeline invocation. The
wins are (a) consistency across pipelines, (b) no redundant
computation, (c) one place to reason about and change
gap-classification behavior, (d) growth and RMS get more accurate
gap classifications by virtue of switching from the single-file to
the multi-file approach.

### Design — keep pipelines distinct where they need to be

The goal is **NOT** to force all three pipelines to do the same
thing. Each pipeline keeps its own paths where its needs differ:

- **HMM ref-stage mode keeps zero-bin removal.** Math constraint
  (zero denominators in the ratio step) + proven accuracy
  advantage over pseudocounts. This is not gap classification —
  it's a different need that happens to consume zero bins.
  Untouched by this change.
- **Growth's inline `frac_bad` masking stays** because it is genuinely
  pipeline-specific: it operates on the post-transform `Y` matrix
  (after `log2`, stage operations, etc.) — i.e., on growth's own
  intermediate, not on raw bedGraph data. Per the architectural
  principle ("anything shared by two or more pipelines and not
  dependent on pipeline-internal upstream work can come out in
  front"), `frac_bad` is on the wrong side of that line — it
  depends on pipeline-internal upstream work. So it stays inside
  growth. If a later audit determines its post-transform check is
  in fact redundant with the shared missingness artifact, retire it
  then.
- **RMS's bedGraph-level processing internals** (signal extraction,
  per-chrom stage operations) are unchanged. What changes for RMS
  in this work is its **post-call** flow:
  `annotate_calls_with_gap_distance()` will now use the multi-file
  shared mask by default (so RMS stops doing single-file gap
  classification), but everything before that step in RMS is
  untouched.

What does change:

- **Diagnostic missingness logging** moves pre-pipeline. Lifted
  from `_report_missingness()` in growth and emitted by the
  controller before any pipeline runs.
- **Gap-mask construction** moves pre-pipeline using the multi-file
  intersection approach
  (`build_shared_gap_mask_from_bedgraphs()`). The result becomes a
  controller-owned artifact (location TBD per `output_layout.py`
  conventions).
- **`annotate_calls_with_gap_distance()` is updated** to call
  `build_shared_gap_mask_from_bedgraphs()` by default instead of
  `build_gap_mask_from_bedgraph()`. A legacy option preserves the
  single-file path in case any pipeline needs to revert (e.g., a
  `strategy="shared"|"single"` parameter, default `"shared"`).
  Whether to remove the legacy code is a later-cycle decision.
- **Growth and RMS inherit the change automatically** since they
  call `annotate_calls_with_gap_distance()` directly. They may need
  small tweaks to pass the bedGraph list (currently a single string;
  the function may take a list when targeting the shared variant,
  or sniff based on the strategy parameter).
- **HMM gains `annotate_calls_with_gap_distance()`** for its own
  post-call gap-aware annotation. This is cross-pipeline parity —
  HMM currently has no gap-aware call metrics. **Separate need
  from zero-bin removal.** The two coexist: ref-stage zero-bin
  removal stays (math constraint); post-call gap annotation is
  added (parity with growth and RMS).
- **HMM chrom-median mode consumes the pre-pipeline shared gap
  mask to remove gap bins from the bedGraph before HMM runs.** Not
  masking, not annotation — actual removal so the HMM operates as
  if those bins don't exist. The HMM statepath forms across the
  remaining bins; the collapse step puts the gap regions back as
  overlapped intervals in the output coordinates. This is what
  serves chrom-median's need to keep the same set of gap bins
  excluded across all files (no inconsistency from per-file
  zero-drop).

**Concrete deliverables when this fires:**

1. A pre-pipeline gap-analysis step in the controller, run after
   bin-size detection, before any pipeline runs. Uses
   `build_shared_gap_mask_from_bedgraphs()` against the input
   bedGraphs.
2. Emitted artifacts:
   - A shared gap-mask BED at e.g.
     `<out_dir>/01-prior/00-gap-analysis/<out_prefix>_shared_gap_mask.bed`.
   - A missingness diagnostic log (chrom-level summary, current
     `_report_missingness` content, lifted out of growth).
3. `annotate_calls_with_gap_distance()` updated:
   - Default behavior calls
     `build_shared_gap_mask_from_bedgraphs()` (multi-file intersection).
   - Legacy parameter (e.g., `strategy="single"`) preserves
     `build_gap_mask_from_bedgraph()` behavior for revertability.
4. HMM gains a post-call `annotate_calls_with_gap_distance()`
   invocation for cross-pipeline parity.
5. HMM chrom-median path consumes the pre-pipeline shared mask to
   **remove** gap bins from the bedGraph fed to HMM.
6. Growth and RMS inherit the updated mask default automatically.
   Verify they produce reasonable gap-distance metrics on existing
   test data (will differ slightly from current single-file results
   — that's the intended improvement).
7. CLI surface (TBD; likely controller-level): a way to override
   the gap-analysis source / strategy / chrom-median bin-removal
   threshold.
8. Tests:
   - Shared mask artifact produced and correct.
   - HMM ref-stage path produces accurate results (no regression
     from the unchanged zero-bin removal).
   - HMM chrom-median path correctly removes pre-computed gap bins
     before HMM sees the bedGraph.
   - HMM produces gap-annotated call output that matches growth/RMS
     in format/columns.

**Counterpoints / things to weigh:**

- **Switching the default produces slightly different growth/RMS
  gap-distance results.** Per-file dropouts no longer count as
  gaps. That's the intended improvement, but worth flagging in
  CHANGELOG when the change lands and verifying no downstream
  consumer is surprised.
- **Legacy code retention.** Keep `build_gap_mask_from_bedgraph()`
  available alongside the new default. Decide later (separate
  cycle) whether to remove it.
- **Performance is not the motivation.** Don't lean on speed
  arguments in the SPEC; lean on consistency and correctness.

**Why this lives in HMM_SOUP rather than waiting:** the HMM
pipeline's need for both (a) chrom-median consuming a precomputed
shared gap mask to remove bins consistently across files and (b)
post-call gap-aware annotation parity with growth/RMS — those
needs are specific to HMM completeness work. The fact that
growth and RMS also benefit (more accurate gap classification +
no redundant mask computation) is downstream gravy. But the work
touches all three pipelines, so when this is formalized into a
SPEC priority the cross-pipeline rewiring should be explicit
deliverable scope.

**Cross-reference:** add a short entry in
`multi-agent/tracking/KNOWN_ISSUES.md` pointing here. The
redundant single-file mask computation between growth and RMS,
and the fact that growth/RMS use the less-principled single-file
approach today, both warrant a near-term-issue cross-reference
even though the full fix is HMM-phase work.

