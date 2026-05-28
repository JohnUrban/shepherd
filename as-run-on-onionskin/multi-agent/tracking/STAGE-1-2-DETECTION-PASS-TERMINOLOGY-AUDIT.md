# Stage-1 / Stage-2 Detection-Pass Terminology Audit

**Created:** 2026-04-25 (Phase 14 Supplemental cycle 14S.2a, priority 14-S14)
**Authors:** John M. Urban, Claude Code 2.1.109 (claude-sonnet-4-6), Codex (GPT-5)

## Purpose

This file is the canonical decision surface for a future renaming phase that
would migrate internal "Stage-1" / "Stage-2" detection-pass terminology to
unambiguous names (e.g., `pass-1` / `pass-2`, `passone_*` / `passtwo_*`). It
classifies every `Stage-1` / `Stage-2` / `stage_1` / `stage_2` / `stage1` /
`stage2` token in the live codebase as one of:

- **detection-pass** — refers to the two-pass detection algorithm internal to
  the rcn-mean-shift / refinement code (Stage-1 = mean-shift candidate
  detection; Stage-2 = scoring / refinement of those candidates). RENAME
  CANDIDATE.
- **biological-stage** — refers to developmental stage 1, stage 2, etc. — the
  per-replicate developmental ordering. **MUST NOT BE RENAMED.**
- **ambiguous** — needs case-by-case review.

## Critical invariant

**Biological-stage terminology is NEVER renamed.** It correctly reflects the
developmental biology of the experimental system (per-replicate stage 1, stage
2, etc., across developmental time). Renaming biological-stage tokens would
silently invalidate the scientific meaning of variable names, output filenames,
and user-facing messages. Any future renaming phase must touch only
`detection-pass` rows in this audit, and must re-confirm `ambiguous` rows
against the live code at the time of that phase.

## Scope of this audit

- **Classify only — no code changes in this priority.** This file is produced
  as part of Phase 14 Supplemental cycle 14S.2a, which is an audit-only cycle
  (no runtime code touches).
- Future renaming work, if approved, would consume this file as its work-list.

## How this audit was produced

1. Role 1 (initial audit) harvested tokens via:
   ```bash
   grep -rn -E '[Ss]tage[ _-]?[12]\b|[Ss]tage1|[Ss]tage2' onionskin.py onionskin_core/ tests/ \
     | grep -v '\.pyc\|__pycache__\|stage_1[0-9]\|stage_2[0-9]\|stage1[0-9]\|stage2[0-9]'
   ```
   plus equivalent greps on `multi-agent/full_instructions/PIPELINE_SPEC.md`
   and `multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md`.
2. Role 2 (this round) verified Role 1's classifications against the
   surrounding live-code context and added supplemental sites that the initial
   harvest underrepresented (notably `tests/test_pipeline.py` per-stage fixture
   filenames and additional pipeline-diagram occurrences in PIPELINE_SPEC.md
   and ONIONSKIN_FULL_HANDOFF.md). The grep returns ~257 lines total — many
   are repeated occurrences of the same token in the same file. Only the
   distinct contextual sites worth a future rename decision are tabulated
   below.

## Detection-pass tokens (rename candidates)

These are internal references to the **two-pass detection algorithm**. They
are the targets of any future rename pass.

### Source code — `onionskin_core/`

| File:line | Context | Suggested rename | Justification |
|---|---|---|---|
| `onionskin_core/detection.py:16` | `from .refinement import ... stage2_score` | `passtwo_score` | Function name in detection-pass scoring |
| `onionskin_core/detection.py:40` | `candidates = stage1_mean_shift(...)` | `passone_mean_shift` | Call site for pass-1 candidate detection |
| `onionskin_core/detection.py:42` | `score = stage2_score(...)` | `passtwo_score` | Call site for pass-2 scoring |
| `onionskin_core/detection.py:53` | `"stage1_score_z": candidate.score_z` | `passone_score_z` | Output column key (internal); pass-1 z-score |
| `onionskin_core/detection.py:54` | `"stage1_scale_bins": candidate.scale_bins` | `passone_scale_bins` | Output column key (internal); pass-1 scale |
| `onionskin_core/detection.py:60` | `sort_values([..., "stage1_score_z"], ...)` | `passone_score_z` | Mirror upstream column rename |
| `onionskin_core/detection.py:81` | `def stage1_mean_shift(...)` | `def passone_mean_shift(...)` | Function definition for pass-1 mean-shift |
| `onionskin_core/refinement.py:117` | docstring `(from stage2_refine)` | `passtwo_refine` | Docstring naming pass-2 refinement |
| `onionskin_core/refinement.py:263` | `def stage2_refine(...)` | `def passtwo_refine(...)` | Function definition for pass-2 refinement |
| `onionskin_core/refinement.py:278` | `return stage2_score(...)` | `passtwo_score` | Call site |
| `onionskin_core/refinement.py:282` | `# Stage-2 scoring and shape filter` | `# Pass-2 scoring and shape filter` | Comment |
| `onionskin_core/refinement.py:356` | `def stage2_score(...)` | `def passtwo_score(...)` | Function definition for pass-2 scoring |
| `onionskin_core/rcn_mean_shift_helpers.py:13–14` | imports of `stage1_mean_shift`, `stage2_refine`, `stage2_score` | mirror upstream | Import-side mirror |
| `onionskin_core/rcn_mean_shift_helpers.py:35,196,308,346,348,357,455,684,695,710,717,718,720,806,807,810,811,818,819,824,893,894,897,996,1003,1082` | `stage1_score_z` / `stage1_scale_bins` / `stage2_refine` / `stage2_score` / `peak_default_stage1_best` / `peak_default_stage1_best_parabola` / `best_stage1_score_z` / `summit_estimator_used = "stage1_best_parabola"` etc. | mirror upstream renames; `stage1_best` column-key pattern → `passone_best` | All detection-pass; many repeated column keys |
| `onionskin_core/engines/growth_model_engine.py:102,109,140,141,312,822,1287,1289,1301,1302,1311,1344` | imports + `def bed_score(stage1_z, ...)` + `stage1_z` / `stage2_score` references | mirror upstream | Engine consumes detection-pass values |

### Source code — top-level `onionskin.py`

| File:line | Context | Suggested rename | Justification |
|---|---|---|---|
| `onionskin.py:1998` | help text: `"default keeps the highest-stage1 contributing peak. early-parabola-mean replaces ..."` (`--rms-summit-policy` help) | "highest-pass-1 contributing peak" | User-visible help; refers to pass-1 detection score |

### Tests

| File:line | Context | Suggested rename | Justification |
|---|---|---|---|
| `tests/test_pipeline.py:764` | `assert "peak_default_stage1_best" in text, out` | `peak_default_passone_best` | Detection-pass column key check (column emitted by rcn-mean-shift unification) |
| `tests/test_pipeline.py:765` | `assert "peak_default_stage1_best_parabola" in text, out` | `peak_default_passone_best_parabola` | Detection-pass column key check |
| `tests/eval_summit_precision_v2.py:456,457,461,462,464,465,488,489,490,491,519` | `peak_default_stage1_best`, `stage1_best_parabola`, `stage1_best_peak`, `best_stage1_score_z` | mirror upstream | Detection-pass column readers in eval harness |
| `tests/test_eval_summit_precision_v2.py:25,91,139,167,180,181,201,203,204,217` | `best_stage1_score_z`, `stage1_best_peak_bp`, `stage1_best_parabola_bp`, `summit_estimator_used == "stage1_best_parabola"` | mirror upstream | Same |

### User-facing docs — pipeline diagrams and prose

| File:line | Context | Suggested rename | Justification |
|---|---|---|---|
| `multi-agent/full_instructions/PIPELINE_SPEC.md:219–220,235–236,261,280–281,359–360` | mermaid diagram nodes: `Stage 1: Mean-shift scan → candidates`, `Stage 2: Refine boundaries...`, etc. | `Pass 1: Mean-shift scan → candidates`, `Pass 2: Refine boundaries...` | Pipeline-flow diagrams describing the two-pass detection algorithm |
| `multi-agent/full_instructions/PIPELINE_SPEC.md:415` | heading `### Step 3 — Stage 1: Mean-shift scan (candidate detection)` | `### Step 3 — Pass 1: Mean-shift scan (candidate detection)` | Pipeline-step heading |
| `multi-agent/full_instructions/PIPELINE_SPEC.md:421` | `peak_default_stage1_best`, `peak_default_stage1_best_parabola`, `--rms-early-summit-stages` description | mirror upstream column renames; help text rephrasing | Detection-pass column references in user-facing prose |
| `multi-agent/full_instructions/PIPELINE_SPEC.md:459` | `stage1_score_z, stage1_scale_bins` (output schema bullet) | mirror upstream | Output schema documentation |
| `multi-agent/full_instructions/PIPELINE_SPEC.md:462` | `onionskin_core/detection.py → stage1_mean_shift()` | mirror upstream | Code-pointer reference |
| `multi-agent/full_instructions/PIPELINE_SPEC.md:1585` | `"the same mean-shift and Stage-2 scoring logic used by the single/ratio pipeline"` | "Pass-2 scoring" | Cross-reference prose |
| `multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md:581` | "Stage 1: z-thresh × trend-kb × smooth-kb (60 combos)" | "Pass 1: z-thresh × trend-kb × smooth-kb" | Parameter-sweep description for two-pass detection |
| `multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md:582` | "Stage 2: peak-search-kb × window-kb (16 combos)" | "Pass 2: peak-search-kb × window-kb" | Same |
| `multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md:864` | `stage2_refine()` — applies the v2 Stage 2 window fitting at any resolution | `passtwo_refine()` — applies the v2 Pass-2 window fitting | Function-pointer description |
| `multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md:881` | `refines summits at base and hires resolutions via stage2_refine` | `via passtwo_refine` | Mirror |
| `multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md:1191` | `stage2_score in onionskin_call_autobin_v2.py` | `passtwo_score in ...` | Engine-pointer description |
| `multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md:1196` | `score = stage1_z + max(0, delta_bic)` | `score = passone_z + max(0, delta_bic)` | Multistage scoring formula prose |
| `multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md:1213` | "stage2-refined boundaries (proper window geometry)" | "pass-2-refined boundaries" | Quality-ranking discussion |
| `multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md:1240` | "keeping the row with the highest stage1 z-score" | "highest pass-1 z-score" | Dedup discussion |
| `multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md:1253` | "**Stage 1 detection:**" + `stage1_score_z` | "**Pass 1 detection:**" + `passone_score_z` | Section heading + column ref |
| `multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md:1255` | "**Stage 2 refinement (`stage2_refine`):**" | "**Pass 2 refinement (`passtwo_refine`):**" | Section heading + function ref |
| `multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md:1260` | `stage2_refine returns the bin with the highest score` | `passtwo_refine returns the bin...` | Function-behavior description |
| `multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md:1315` | `final_summit_bp` remains the stage2-refine result | `pass-2-refine result` | Output-schema discussion |

## Biological-stage tokens (MUST NOT rename)

These reflect developmental biology — per-replicate developmental stage 1, 2,
etc. They appear in output filenames, manifest definitions, log messages, and
test fixture filenames. They MUST NOT be renamed in any future rename pass.

### Source code

| File:line | Context | Reason |
|---|---|---|
| `onionskin.py:604,607,611,619` | `stage1.RCN.bedGraph` (output filename for first developmental stage) | Per-stage output filename naming convention |
| `onionskin.py:808` | "manifest stages are numbered 1..N in developmental order, with stage 1 ..." | Help / documentation prose about developmental ordering |
| `onionskin.py:1270` | "stage 1 = earliest" (posterior-stage assignment help) | Posterior assignment of developmental stages |
| `onionskin.py:2988,3128–3154` | "Stage-1 contamination" check (posterior stage-1 samples QC) | Biological QC: posterior stage-1 = pre-amp samples |
| `onionskin.py:3768` | "reference samples are stage 1" | Biological — reference samples in developmental stage 1 |
| `onionskin_core/hmm_fork_travel.py:505` | `# stage_1 at top` (developmental stage axis label) | HMM trajectory developmental axis |
| `onionskin_core/posterior.py:103` | "posterior stage 1" (cluster ordering) | Cluster ordering by developmental position |
| `onionskin_core/hmm_metrics.py:67,128,249` | `stage_label`, e.g., `"stage_2"` parsed from filenames | Per-stage metrics keyed by developmental stage |

### Tests — manifest / fixture filenames

| File:line | Context | Reason |
|---|---|---|
| `tests/test_pipeline.py:284,330` | `f"{out_prefix}_stage_median_within_calls.stage1.bedGraph"` (per-stage output filename test) | Per-developmental-stage output check |
| `tests/test_pipeline.py:460–462` | `Sample(stage=1, path="stage1_rep1.bedGraph", ...)`, `Sample(stage=2, ...)` | Replicate dataset; stage = developmental stage |
| `tests/test_pipeline.py:544,545,588,589,630,631,1042,1043,1081,1082,1202,1268,1269,1342,1343,1413,1414` | `toy / "stage1_rep1.bedGraph"`, `toy / "stage1_rep2.bedGraph"` | Toy fixture filenames for first developmental stage |
| `tests/test_pipeline.py:695` | `f"{out_prefix}_stage1_dedup_report.tsv"` | Per-developmental-stage report filename |
| `tests/test_pipeline.py:696` | `f"{out_prefix}_stage1_calls.tsv"` | Per-developmental-stage calls filename |
| `tests/test_pipeline.py:1116,1160` | `tmp_path / "stage1_manifest.tsv"`, `"stage1_manifest_unsupported.tsv"` | Test manifest naming for first developmental stage |
| `tests/run_hmm_fork_age_smoke.py:94,95,100,101` | `stage_1.hmm_amplicon_metrics.tsv`, `stage_2.hmm_amplicon_metrics.tsv` | Per-developmental-stage HMM metric files |
| `tests/optimize_single_params.py:134,138,145,156,189` | dataset descriptors `stage9`, `stage5`, `stage1` | Biological dataset identifiers |
| `tests/test_hmm_ported_analyses.py:91` | `"1.TPM.medNorm.stage_1_median.zeroBinsRemoved.RCN_pseudo0.bedGraph"` | Input bedGraph filename for first developmental stage |
| `tests/eval_summit_precision_v2.py:1428,1433` | `stage2b = _eval_II2B_stage_estimators_from_map(...)`; `summary["II2B_stage_estimators"]` | II/2B is the dataset stage label; the `stage2b` here is a Python variable name for an evaluation result on that dataset |

## Ambiguous tokens (case-by-case review at rename time)

| File:line | Context | Reason ambiguous |
|---|---|---|
| `tests/optimize_single_params.py:12–13,26` | banner: `"Stage 2 — fix Stage-1 winners"`, etc. | These describe the `optimize_single_params.py` tool's own three-stage workflow, not the detection algorithm. Probably keep (this script names its own internal steps), but consider renaming to avoid confusion with the detection-pass terminology |
| `tests/test_eval_summit_precision_v2.py:96,144,185,222` | filenames like `demo_stage2_summit_estimates.tsv` (test fixture filenames) | "stage2" here likely refers to detection-algorithm pass-2 output. Classify case-by-case at rename time by reading what the fixture exercises |
| `multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md:559,564,565,566,568` | T-test descriptions like "single-stage 2-rep, 5 kb", "ratio-of-ratios, 5 kb / DS1 stage9/stage1" | "single-stage" / "2-rep" describes a single-developmental-stage dataset with two replicates (biological), but appears in a detection-method test matrix. Conservative classification: biological |

## Notes on the harvest

- Role 1's initial harvest captured the canonical detection-pass surfaces. Role
  2's verification (this round) added entries from `tests/test_pipeline.py`
  per-stage fixture filenames (biological, not detection-pass) and expanded the
  user-facing docs section to cover all the pipeline-diagram and prose
  occurrences in PIPELINE_SPEC.md and ONIONSKIN_FULL_HANDOFF.md.
- The `stage_state_trail` token in `onionskin_core/rcn_mean_shift_helpers.py:730`
  (`_classify_stage_state_trail`) is a per-developmental-stage classification
  trail, NOT detection-pass terminology. It is correctly biological.
- This audit intentionally does **not** propose renames for help text inside
  user-facing CLI flags whose name itself is `--*-stage-*` (e.g.,
  `--rms-early-summit-stages`); those are user-facing flag-name questions
  separate from the internal-code rename discussion.

## Future-phase notes

When a future renaming phase consumes this file:

1. Re-run the harvesting greps; classifications may need updating if the
   codebase has shifted.
2. Run renames module-by-module rather than as a global sed — column-name
   renames change wire-format (TSV headers), which propagates into eval
   harnesses, dashboards, and any user scripts. Plan the migration path before
   touching code.
3. Hold the `biological-stage` rows fixed throughout. Only `detection-pass`
   rows are eligible for renaming.
