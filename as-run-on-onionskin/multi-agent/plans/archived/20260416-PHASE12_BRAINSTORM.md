# PHASE 12 BRAINSTORM

**Status:** Pre-spec brainstorm. Not committed to ROADMAP yet.  
**Purpose:** Synthesize codebase audit + BRAINSTORM.md + KNOWN_ISSUES.md into candidate
Phase 12 priorities, unordered. This doc feeds PHASE12_SPEC.md after user review.

**User-stated Phase 12 goals:**
1. Restructure growth pipeline output into numbered step-by-step subdirectories (like HMM's
   01-mednorm through 16-clustering). **No output files at the `02-growth-model/` root** —
   all output files move into numbered step dirs.
2. Restructure pipeline execution order in `onionskin.py` to HMM → Growth → Per-Stage
   (matching directory order 01-hmm, 02-growth-model, 03-per-stage-mean-shift).
3. Audit whether growth pipeline can reuse HMM-generated smoothed RCN files (may stay or
   become its own future phase depending on complexity).

---

## BRAINSTORM-A: Growth pipeline step formalization (numbered steps, no root files)

### Current state

The HMM pipeline has a clean `steps` dict in `onionskin_core/engines/hmm_engine.py`:
```python
steps = {
    "step1": os.path.join(hmm_out_dir, "01-mednorm"),
    "step2": os.path.join(hmm_out_dir, "02-unionStats"),
    ...
    "step16": os.path.join(hmm_out_dir, "16-clustering"),
}
```
All HMM output lives in numbered step dirs. No canonical result files sit at the `01-hmm/` root.

The growth pipeline has numbered subdirs but puts its canonical detection outputs at the
container root instead of inside a step dir:
```
02-growth-model/
  onionskin_calls.tsv           <- at root — needs to move
  onionskin_origins.tsv         <- at root — needs to move
  onionskin_stage_summary.tsv   <- at root — needs to move
  onionskin_timing.tsv          <- at root — needs to move
  onionskin_calls.gap_annotated.tsv   <- at root — needs to move
  onionskin_unified_onionskin_calls.bed        <- at root — needs to move
  onionskin_unified_onionskin_calls.resolved.bed   <- at root — needs to move
  onionskin_amplicons_recommended_to_keep.bed  <- at root — needs to move
  onionskin_amplicons_actively_rejected*.bed   <- at root — needs to move
  onionskin__shape_scores_growth_model.tsv     <- at root — needs to move
  01-stage_median_within_calls/
  02-summary_bedgraphs/
  03-summits/
  04-summit_refinement/
  05-aps/
  06-plots/
  07-notebook/
```

### Target state (user intent: no root files, atomized steps)

`01-detection` is split into 5 discrete engine sub-steps, matching the HMM's granularity
(HMM has 10 pre-analysis steps; growth gets 5 engine steps + downstream analysis steps).
All output moves into numbered step dirs — nothing at the `02-growth-model/` root.

| Step | Dir | Contents | HMM analog | Source (current) |
|------|-----|----------|------------|-----------------|
| 01 | `01-growth-track` | `growth_track.bedGraph` | `05-medianSmoothed-RCN` | `02-growth-model/` root |
| 02 | `02-calls` | `calls.tsv`, `stage_summary.tsv`, `per_sample_amplitudes.tsv` | `06-HMM` | `02-growth-model/` root |
| 03 | `03-shape-filter` | `shape_scores_multistage.tsv`, `amplicons_actively_rejected*.bed` | `08-aboveBackground` | `02-growth-model/` root |
| 04 | `04-origins` | `origins.tsv` | `09-summitStates` / `10-summitBins` | `02-growth-model/` root |
| 05 | `05-progression` | `progression.tsv` | _(no HMM analog)_ | `02-growth-model/` root |
| 06 | `06-stage-medians` | `stage_median_within_calls.stage*.bedGraph` | _(internal intermediate)_ | `01-stage_median_within_calls/` |
| 07 | `07-signal-tracks` | `summary_model_peak.base.fold.bedGraph`, `.log2.bedGraph` | _(no HMM analog)_ | `02-summary_bedgraphs/` |
| 08 | `08-summits` | `summits.final.bed` | _(engine output)_ | `03-summits/` |
| 09 | `09-summit-refinement` | stage/aggregate/all summit estimate TSVs + BED | `13-summit-refinement` | `04-summit_refinement/` |
| 10 | `10-timing` | `timing.tsv`, `timing_report.md` | `15-timing` | `02-growth-model/` root |
| 11 | `11-overlapResolution` | `unified_calls.bed`, `unified_calls.resolved.bed` | _(no HMM analog)_ | `02-growth-model/` root |
| 12 | `12-gap-analysis` | `calls.gap_annotated.tsv`, `calls.gap_analysis.tsv`, `calls.gap_report.md`, `amplicons_recommended_to_keep.bed`, `amplicons_recommended_for_exclusion.bed` | _(no HMM analog)_ | `02-growth-model/` root |
| 13 | `13-aps` | APS matrices, cluster maps, `genome_stage_medians/`, `samples/` | `14-aps` | `05-aps/` |
| 14 | `14-plots` | All diagnostic plots (profile, shape, APS, overlap, genome-overview) | _(no HMM analog)_ | `06-plots/` |
| 15 | `15-notebook` | Jupyter notebooks | _(no HMM analog)_ | `07-notebook/` |

**15 steps — mirrors HMM's 16.**

**File-to-step mapping for all previously root-level files:**
- `onion_growth_track.bedGraph` → `01-growth-track/`
- `onion_calls.tsv`, `onion_stage_summary.tsv`, `onion_per_sample_amplitudes.tsv` → `02-calls/`
- `onion_shape_scores_multistage.tsv`, `onion_amplicons_actively_rejected*.bed` → `03-shape-filter/`
- `onion_origins.tsv` → `04-origins/`
- `onion_progression.tsv` → `05-progression/`
- `onion_timing.tsv`, `onion_timing_report.md` → `10-timing/`
- `onion_unified_onionskin_calls.bed`, `onion_unified_onionskin_calls.resolved.bed` → `11-overlapResolution/`
- `onion_calls.gap_annotated.tsv`, `onion_calls.gap_analysis.tsv`, `onion_calls.gap_report.md`, `onion_amplicons_recommended_to_keep.bed`, `onion_amplicons_recommended_for_exclusion.bed` → `12-gap-analysis/`

**Naming convention (Q4 resolved: hyphens, readable short names):**
- Engine steps use hyphen-separated words: `01-growth-track`, `02-calls`, `03-shape-filter`, `04-origins`, `05-progression`
- `06-stage-medians` — renamed from `01-stage_median_within_calls` (shorter, underscores removed)
- `07-signal-tracks` — renamed from `02-summary_bedgraphs` (describes content not file type; `03-signal` was too short per user feedback)
- `08-summits` — renamed from `03-summits` (number change only)
- `09-summit-refinement` — renamed from `04-summit_refinement` (hyphens, "summit" kept per user direction)
- `10-timing`, `11-overlapResolution`, `12-gap-analysis`, `13-aps`, `14-plots`, `15-notebook` — new or renumbered
- `11-overlapResolution` uses camelCase matching HMM style (`07-collapsedHMM`, `08-aboveBackground`)

**Note on `--resolve-overlaps` removal (Q2 resolved):**
The `--resolve-overlaps` flag currently gates whether overlap resolution runs. User decision:
always run it — no use case for skipping it; it is required for a non-redundant unique amplicon
set. For Phase 12, remove the flag and make `11-overlapResolution/` a mandatory step. All
`if getattr(args, "resolve_overlaps", False):` guards in `_phase2_postprocess()` go away.

**Note on `06-stage-medians/` (resolved: keep it):**
These are stage-median bedGraphs restricted to called intervals — **not** genome-wide tracks.
Currently used internally by `resolve_overlaps_bed()` (valley-depth signal) and
`write_profile_plots()`. Keeping them on disk is consistent with HMM retaining intermediate
step outputs. Renamed from `01-stage_median_within_calls/` to `06-stage-medians/` (shorter,
hyphenated). The engine writes these during step 02-calls; they are kept for downstream use.

**Note on engine refactoring (steps 01–05):**
Currently the engine writes all steps 01–05 outputs to the flat `out_prefix` in a single
monolithic pass. For Phase 12, the engine must be refactored to write each step's outputs
to its numbered step dir. This parallels how `hmm_engine.py` writes each step function's
outputs to the corresponding `steps["stepN"]` path. The spec must explicitly call this out
as a required engine-level change, not just a file-move reorganization.

### Implementation scope

Changes required across the codebase:

1. **`onionskin_core/output_layout.py`**
   - Add a `growth_steps` dict (analogous to HMM's `steps` dict) mapping step names to paths
     under `growth_model_dir`: `"growth-track"`, `"calls"`, `"shape-filter"`, `"origins"`,
     `"progression"`, `"stage-medians"`, `"signal-tracks"`, `"summits"`, `"summit-refinement"`,
     `"timing"`, `"overlapResolution"`, `"gap-analysis"`, `"aps"`, `"plots"`, `"notebook"`
   - Update all layout dict keys to point to new numbered dirs
   - Update `ensure_layout_dirs()` to create the 15 numbered dirs
   - Update `_INDEX_MD_TEMPLATE` to document the new 15-step layout

2. **`onionskin_core/engines/growth_model_engine.py`**
   - **Refactor engine to write each step's outputs to its numbered step dir directly** (steps 01–05)
   - Receive the `growth_steps` dict as a parameter (same pattern as HMM receiving its `steps` dict)
   - Remove all flat-prefix output writes; replace with step-dir writes
   - `steps` dict lives in `output_layout.py`, not here

3. **`onionskin.py`**
   - `_layout_growth_model_path()` / `_layout_growth_model_prefix()`: redirect to `02-calls/` step dir
     (formerly the "detection" root; calls.tsv and origins.tsv are now in `02-calls/` and `04-origins/`)
   - `_phase2_postprocess()`: update all layout key references to new step dirs; remove all
     `if getattr(args, "resolve_overlaps", False):` guards (overlap always runs)
   - `_apply_multistage_shape_filter()`: write to `03-shape-filter/` step dir
   - `_run_posterior()`: update all growth path references for posterior context
   - Build `growth_steps` from `output_layout` and pass to engine

4. **`tests/test_pipeline.py`**
   - `_growth_model_dir()` helper: container path stays `02-growth-model/`
   - Detection output assertions: move from root to `02-calls/`, `03-shape-filter/`, `04-origins/`, `05-progression/`
   - `"01-stage_median_within_calls"` → `"06-stage-medians/"`
   - `"02-summary_bedgraphs"` → `"07-signal-tracks/"`
   - `"03-summits"` → `"08-summits/"`
   - `"04-summit_refinement"` → `"09-summit-refinement/"`
   - `"05-aps"` → `"13-aps/"`
   - Add assertions for `"11-overlapResolution/"` (always present post-flag-removal)
   - Add assertions for `"12-gap-analysis/"` outputs
   - Phase 11.1 contract assertions for secondary growth families → update all numbers

5. **`scripts/summit_inspector.py`**
   - `PIPELINE_DIR_NAMES["growth"] = "02-growth-model"` — container stays same
   - `_find_origins_tsv()`: root glob → `04-origins/`
   - `_find_stage_medians_dir()`: `"05-aps/genome_stage_medians"` → `"13-aps/genome_stage_medians"`
   - `'output_dir': os.path.join(pipeline_dir, '06-plots', 'summit_fits')` → `"14-plots/summit_fits"`

6. **`scripts/aps_cluster_report.py`**
   - `"01-stage_median_within_calls"` → `"06-stage-medians"`
   - `"04-summit_refinement"` → `"09-summit-refinement"`
   - Root-level `_calls.tsv` → `"02-calls"` subdir
   - Root-level `_stage_summary.tsv` → `"02-calls"` subdir
   - All prior and posterior variants updated accordingly

7. **`scripts/generate_hmm_v2_references.py`** — audit for growth path assumptions  
   **`scripts/compare_puffstep_outputs.py`** — audit for growth path assumptions  
   **`scripts/aps_cluster_experiments.py`** — audit for growth path assumptions  
   **`onionskin_core/readme.py`** — audit generated run docs for hardcoded growth subdir paths

---

## BRAINSTORM-B: Pipeline execution order (HMM → Growth → Per-Stage)

### Observation

The target grouped directory order is:
```
01-prior/
  01-hmm/
  02-growth-model/
  03-per-stage-mean-shift/
```

The current execution order in `onionskin.py` for a combined `--pipelines all` run is:
```
1. Growth → run_multistage() + _phase2_postprocess()
2. Per-stage → _run_standalone_per_stage_controller()
3. HMM → _run_hmm_engine()    ← runs LAST, despite living in 01-hmm/
```

This inverts the directory naming. Per the BRAINSTORM [2026-04-11] note: "HMM-first
ordering has a strong historical argument because it matches the long-lived pufferfish /
PuffStep mental model and places the most established lane at the left edge of the continuum."

### Dependency analysis

- **Does HMM depend on growth outputs?** No. `_run_hmm_engine()` reads directly from the
  manifest and writes under `01-hmm/`. It has no dependency on `02-growth-model/` files.
- **Does growth depend on HMM outputs?** Currently no. Exception: if BRAINSTORM-C
  (HMM RCN reuse) is implemented, then yes — growth would read step-5 files from `01-hmm/`.
  This makes BRAINSTORM-B a prerequisite for BRAINSTORM-C.
- **Does per-stage depend on growth or HMM?** No. Per-stage reads from the manifest
  independently.

### What changes in `onionskin.py`

The main execution branches for the `--manifest` path (lines ~2938–3118) each currently
call growth → per-stage → HMM. Each branch needs reordering to HMM → growth → per-stage.

Key branches to reorder:
1. `dec.mode == "multistage" and _pipelines_include(args, "growth")` (line ~2938):
   - Current: `run_multistage()` → `_phase2_postprocess()` → per-stage → `_run_hmm_engine()`
   - Target: `_run_hmm_engine()` → `run_multistage()` → `_phase2_postprocess()` → per-stage
2. `dec.mode == "multistage" and _pipelines_include(args, "per-stage")` (line ~3013): per-stage only, HMM at end — reorder HMM to run before per-stage
3. `dec.mode == "single-stage"` (line ~3033): growth → HMM; reorder to HMM → growth
4. Single-file caller path (line ~3076): growth → HMM; reorder to HMM → growth
5. `--onionskin` path (line ~3120+): check same ordering issue there
6. `_run_posterior()`: check whether the posterior branch also sequences growth before HMM

**Pre-run setup:** `shared_chromosomes` and `ensure_layout_dirs()` are currently called
as part of setting up the growth run. If HMM runs first, these must be resolved before
the first pipeline call. This is already partially separated (chromosomes are resolved
via `_resolve_shared_chromosomes()`, `ensure_layout_dirs()` is called before the growth
engine), but the spec should verify that both are clean to call before HMM.

### Files that need changes

| File | Change |
|------|--------|
| `onionskin.py` | Reorder pipeline calls in each multistage + single-stage branch |
| `onionskin.py` | Verify `shared_chromosomes` + `ensure_layout_dirs` precede all pipeline calls |
| `onionskin.py` | Reorder posterior branch |

Validation: `make test`, `make toy`, `make single`, `make twin` — no result changes expected
since pipelines are independent.

---

## BRAINSTORM-C: HMM smoothed RCN reuse by growth pipeline

### Observation

In a combined `--pipelines all` run (with BRAINSTORM-B's order change: HMM runs first),
the HMM pipeline has already produced per-sample, per-stage smoothed RCN bedGraphs in
`01-hmm/05-medianSmoothed-RCN/`. The growth model engine runs its own chrom-median
normalization and smoothing independently.

If HMM step-5 outputs are available, the growth pipeline could potentially skip its own
normalization and read the pre-normalized, pre-smoothed files instead — avoiding redundant
computation and ensuring consistent normalization across pipelines.

### What this would require

1. **BRAINSTORM-B must land first**: HMM must complete before growth starts.
2. Growth engine needs a new optional code path: when `01-hmm/05-medianSmoothed-RCN/`
   files are available, read from there instead of re-normalizing.
3. CLI flag (e.g., `--growth-use-hmm-rcn on|off`, default `off` until validated).
4. Fallback: if HMM was not run or step-5 files are absent, growth uses its own normalization.
5. Validation: compare growth results self-computed vs HMM-derived RCN to confirm consistency.

### Risk / complexity

- HMM step-5 uses `--hmm-smooth-halfwidth` bins; growth uses `--growth-rcn-smooth-bins`.
  If window sizes differ, the smoothed profiles differ.
- HMM step-5 applies zero-bin filtering (step 3); growth's normalization may differ
  slightly. Need to verify compatibility.
- HMM step-5 outputs individual per-sample files; growth reads per-stage aggregates.
  The growth engine would need to aggregate from step-5 indiv files or use a new HMM
  per-stage median output.

**Spec recommendation:** scope as feasibility investigation first (read the code, identify
the exact normalization paths, determine compatibility). Implement only if the audit shows
a clean drop-in path. Otherwise defer to a future phase.

### Relationship to KNOWN_ISSUES

- KNOWN_ISSUES [ISSUE:2026-04-19:1] (step-14 APS reads raw manifest, not step-5) is the
  HMM-internal analog. Fixing step-14 to read from step-5 (after the parallel child
  pipeline lands) would be the HMM-side fix; this Priority is the growth-side fix.

---

## BRAINSTORM-D: Scripts and tests consumer audit

### Hardcoded paths inventory (must-fix for BRAINSTORM-A)

**`scripts/summit_inspector.py`** (~330 lines):
- `PIPELINE_DIR_NAMES["growth"] = "02-growth-model"` — container name stays.
- `_find_origins_tsv()` (line ~137–141): root-level globs → `"04-origins/*_origins.tsv"` (and grouped variants)
- `_find_stage_medians_dir()` (line ~180–183): `"05-aps/genome_stage_medians"` → `"13-aps/genome_stage_medians"` (and all path variants)
- `'output_dir': os.path.join(pipeline_dir, '06-plots', 'summit_fits')` (lines 315, 323) → `"14-plots/summit_fits"`

**`scripts/aps_cluster_report.py`** (~320 lines):
- Line 104: `"02-growth-model" / "01-stage_median_within_calls"` → `"02-growth-model" / "06-stage-medians"`
- Lines 133–134, 228–229: `"04-summit_refinement"` → `"09-summit-refinement"`
- Lines 205, 314: root-level `_calls.tsv` → `"02-growth-model" / "02-calls" / f"{prefix}_calls.tsv"`
- Line 217: root-level `_stage_summary.tsv` → `"02-growth-model" / "02-calls" / ...`
- Lines 275–276: prior root-level `_calls.tsv`, `_stage_summary.tsv` → `"02-calls"` subdir

**`tests/test_pipeline.py`** (~350+ lines):
- Lines 96–108: root-level detection outputs → step dirs (`02-calls/`, `03-shape-filter/`, `04-origins/`, `05-progression/`)
- `"01-stage_median_within_calls"` → `"06-stage-medians"`
- `"02-summary_bedgraphs"` → `"07-signal-tracks"`
- `"03-summits"` → `"08-summits"`
- `"04-summit_refinement"` → `"09-summit-refinement"`
- `"05-aps"` → `"13-aps"`
- Phase 11.1 contract assertions for growth secondary families → all step numbers updated
- Add assertions for `"11-overlapResolution/"` and `"12-gap-analysis/"`

**`onionskin_core/readme.py`** (generated run docs):
- Audit `write_run_readme()` for hardcoded growth subdir paths; update to new step numbering.

**Other scripts to audit:**
- `scripts/aps_cluster_experiments.py` — may reference old growth subdir paths
- `scripts/generate_hmm_v2_references.py` — may reference old growth subdir paths
- `scripts/compare_puffstep_outputs.py` — may reference old growth subdir paths

---

## BRAINSTORM-F: HMM steps dict migration to output_layout.py (Q3 corollary)

### Observation

The user's answer to Q3 is: "They should be doing the same thing — and the same thing should be
`output_layout.py`." Currently the HMM pipeline manages its step paths via a `steps` dict built
inside `run_hmm()` in `hmm_engine.py`, NOT via `output_layout.py`. This is inconsistent with the
decided direction.

### What this means for Phase 12

If BRAINSTORM-A adds a `steps` dict for the growth pipeline in `output_layout.py`, the HMM
pipeline should be updated in the same pass to also expose its step paths through `output_layout.py`.
This creates a single canonical path authority for both pipelines — external scripts and tests can
query `output_layout.build_output_layout()` for any step path rather than needing to hardcode
step dir names or import from engine modules.

### Scope of HMM change

`hmm_engine.py` currently builds its `steps` dict with:
```python
steps = {
    "step1": os.path.join(hmm_out_dir, "01-mednorm"),
    ...
    "step16": os.path.join(hmm_out_dir, "16-clustering"),
}
```
This dict is referenced internally by all 16 step functions via `steps["stepN"]`.

The migration would:
1. Move the `steps` dict construction into `output_layout.py` (a new `build_hmm_steps()` function
   or an extension of `build_output_layout()` that returns both the layout dict and the hmm steps).
2. `hmm_engine.py` receives the steps dict as a parameter (passed in from `_run_hmm_engine()` in
   `onionskin.py`) rather than building it internally.
3. External scripts (e.g., `summit_inspector.py`, `generate_hmm_v2_references.py`,
   `compare_puffstep_outputs.py`) can get canonical HMM step paths from `output_layout` instead
   of hardcoding `"01-hmm/01-mednorm/"` etc.

### Files involved

| File | Change |
|------|--------|
| `onionskin_core/output_layout.py` | Add growth `steps` dict + HMM `steps` dict construction |
| `onionskin_core/engines/hmm_engine.py` | Receive `steps` dict as parameter; remove internal construction |
| `onionskin.py` | Build both dicts from `output_layout` and pass to engines |
| `scripts/generate_hmm_v2_references.py` | Use canonical paths from layout |
| `scripts/compare_puffstep_outputs.py` | Use canonical paths from layout |
| Any script with hardcoded HMM step paths | Update to use layout dict |

### Note

This is a refactor with no behavioral change. HMM step paths do not change — only where the
path construction lives. The spec should include a validation pass confirming HMM step paths
are identical before and after.

---

## BRAINSTORM-E: Opportunistic items in the spirit of Phase 12

### [E-1] Grouped-layout path invariant tests (from BRAINSTORM.md [2026-04-11])

Phase 12 is the natural time to add narrow `test_output_layout.py` assertions that verify
major growth output paths land in their expected numbered dirs after a toy run. Locks in
the Phase 12 layout contract without a full golden-output regime.

### [E-2] `00-INDEX.md` template update

`output_layout.py`'s `_INDEX_MD_TEMPLATE` describes the current `01-`–`07-` growth layout.
It must be updated atomically with the dir renaming to describe the new `01-detection/`
through `09-notebook/` (or whatever final numbering the spec settles on).

### [E-3] Execution order note in `00-INDEX.md`

If BRAINSTORM-B lands, add a sentence to `_INDEX_MD_TEMPLATE` documenting the execution
order (HMM → Growth → Per-Stage) and why it matches the directory numbering.

---

## Open questions for spec design session

1. **Timing step placement:** ✅ RESOLVED — `06-timing/` (its own step, atomize principle).

2. **Unified calls and overlap resolution:** ✅ RESOLVED — `02-overlap/`; `--resolve-overlaps`
   flag removed; overlap resolution is mandatory.

3. **`steps` dict location:** ✅ RESOLVED — `output_layout.py` for both growth and HMM
   (see BRAINSTORM-F for HMM migration scope).

4. **Naming convention:** ✅ RESOLVED — hyphen-only, short names. See target state table for
   final short names. Open sub-question: can `01-stage_median_within_calls` data be computed
   in-memory so no directory is needed? Needs code audit in the spec.

5. **BRAINSTORM-C scope in Phase 12:** ✅ RESOLVED — feasibility audit only; report on
   compatibility and trade-offs; do not implement.

6. **Posterior mirroring:** ✅ RESOLVED — yes; posterior branch mirrors prior structure.


## John Urban Answers to Open questions for spec design session

The thems to all my answers are:
- **Atomize steps:** If it is a new process to update the output, it is a new step. Example: overlap resolution is processing the calls to create a new output, so it is a new step.
- **No backwards cramming:** The pipeline is a chain of steps and a chain of outputs. The output from a step further down the chain goes into the output directory for that step, not to a directory upstream in the chain. Example: timing results should be placed in timing step directory, not upstream in 01-detection. If it is a question of merging outputs, merge them, but put the updated file in the later step. We can potentially have a final step directory that contains all the final results.

- **If HMM does it, Growth does it:** 

1. **Timing step placement:**  `06-timing/` (its own step) - If it is a question of merging outputs, merge them, but put the updated file in the later step.

2. **Unified calls and overlap resolution:** a separate `02-overlap-resolution/` We should get rid of "--resolve-overlaps" as part of Phase 12. There is no use case where we would not want to do that. It is a major and necessary part of defining unique amplicons. Is it not? This is clear to me - give pushback if you want - but I think we should get rid of "--resolve-overlaps" since there is no use case for not doing it, and then the answer becomes clear - put it in 02-overlap-resolution/.

3. **`steps` dict location:** Growth and HMM should do the same thing probably, but that might not be what HMM currently does. Claude recommended output_layout.py here, and I agree. But it also sounds to me like the HMM actually needs to be updated to also use output_layout.py. So I think they should be doing the same thing - but the same thing looks like it should be output_layout.py. So updating HMM in this way can clearly be part of Phase 12.

4. **Hyphen vs. underscore naming:** We can do hyphens. We can also change the names in this step to shorter names. Open to suggestions. Also we can get rid of the bedGraphs that only output bins within the calls. I am only interested in the full bedGraphs for signal and the BED files for intervals. Is there a use case for bedGraphs that contain only the bins within the BED intervals? If so, that would be trivial for the user to make on their own. This can also be part of Phase 12.

5. **BRAINSTORM-C scope in Phase 12:** Let's limit this to feasability audit. You already pointed out that it would basically break the independence each pipeline has for giving it parameters. We can look into it and just report on whether it would be worth doing, or would actually only limit onionskin's flexibility. 

6. **Posterior mirroring:** yes — the posterior branch mirrors the prior structure.

## Copilot addendum — hidden consumers, transition mechanics, and scope control (2026-04-15)

This addendum focuses on gaps or under-emphasized items relative to the current brainstorm,
not on restating the core A/B/C/F structure.

### 1. Hidden consumers not yet called out strongly enough

The current consumer audit already catches the biggest runtime scripts, but there are a few
additional high-value consumers that Phase 12 should explicitly track:

- `tests/eval_summit_precision.py`, `tests/eval_summit_precision_v1.py`, and
   `tests/eval_summit_precision_v2.py` each auto-discover growth summit BEDs by taking an
   `_origins.tsv` path and then looking for the BED in a sibling `04-summit_refinement/`
   directory. Once `_origins.tsv` moves into `04-origins/` and summit estimates move into
   `09-summit-refinement/`, that discovery logic breaks unless updated.
- `tests/test_aps.py` currently hardcodes `01-prior/02-growth-model/05-aps/` and still invokes
   `--resolve-overlaps`. That makes it a direct consumer of both the APS step renumber and the
   planned CLI removal / always-run overlap behavior.
- `onionskin_core/notebooks.py` has notebook discovery cells that currently glob for root-level
   `*_calls.tsv`, `*_calls.gap_annotated.tsv`, `*_calls.gap_analysis.tsv`, and keep/exclude BEDs
   under the prior directory. A step-oriented growth layout will break those globs unless the
   notebook discovery is updated to search the new numbered step directories.

### 2. Transition mechanics in `output_layout.py` need their own explicit sub-bullet

Two pieces of `output_layout.py` look small but are actually central to the Phase 12 migration:

- `_INDEX_MD_TEMPLATE` still documents the current 7-subdir growth layout plus root-level final
   files. This is a user-facing contract and should be treated as a required Phase 12 deliverable,
   not just a cleanup note.
- `organize_outputs()` still encodes the old root-file ownership model by moving canonical growth
   outputs from a flat prefix into the current grouped subdirs. If Phase 12 refactors the growth
   engine to write directly into numbered step dirs, the spec should explicitly decide whether
   `organize_outputs()` is updated as a backward-compatibility bridge or retired from the main
   growth path entirely.

Related feedback: `onionskin_core/readme.py` looks lower risk than I initially expected. Its
README generation is mostly file-inventory and suffix driven, so the index template is the more
important documentation surface to prioritize.

### 3. Posterior-specific path rules need to be spelled out, not assumed

The brainstorm already says posterior should mirror prior, but there are two concrete controller
details worth elevating:

- `_run_posterior()` currently writes `posterior_manifest.tsv` and `posterior_hires_manifest_*.tsv`
   directly into the growth pipeline root under `02-posterior/02-growth-model/`. If the Phase 12
   rule is truly "no root files in the growth arm," then the spec needs an explicit exception or a
   new metadata/home for these controller-owned manifest files.
- `_run_posterior()` also still runs the posterior growth pipeline before the posterior HMM arm.
   Phase 12 execution-order work should explicitly include the posterior branch, not only the prior
   `main()` branches.

### 4. Scope-control feedback: keep Phase 12 focused on your stated goals

I would recommend two explicit scope boundaries in the eventual spec:

- Do **not** silently expand Phase 12 into per-stage internal stepification unless you decide you
   want that on purpose. Your stated goals are growth step formalization, global pipeline-order
   alignment, and the HMM-reuse feasibility audit. Per-stage can benefit from the new execution
   order without also needing a full internal numbered-step redesign in this same phase.
- Be careful about removing the within-calls stage-median bedGraphs from disk. Right now the code
   uses them for overlap resolution and profile plotting. If the user-facing desire is to hide or
   de-emphasize them, that is feasible, but it is broader than a simple rename and should be framed
   as either a separate sub-priority or a deliberate "keep as internal step output" decision.

### 5. Consumer triage: not all downstream files are equally fragile

One useful distinction for Phase 12 implementation planning:

- Exact-path consumers are high risk and should be updated in the same phase as the layout change.
   Examples: `tests/test_pipeline.py`, `tests/test_aps.py`, the summit-precision helpers, and the
   notebook discovery code.
- Wildcard or `find`-based consumers are lower risk. Several shell wrappers in `tests/` discover
   `*_origins.tsv` by searching recursively rather than hardcoding a growth subdir, so they may need
   little or no change. That means the spec can prioritize the brittle exact-path consumers first
   instead of over-budgeting for every test script equally.

### 6. Feedback on BRAINSTORM-C: make the feasibility audit a contract-matrix exercise

The current feasibility framing is directionally right. One thing I would add is that the audit
deliverable should probably be a short compatibility matrix, not just a prose conclusion. At a
minimum it should compare:

- smoothing parameter ownership (`--hmm-smooth-halfwidth` vs `--growth-rcn-smooth-bins`)
- zero-bin filtering / missing-bin semantics
- producer shape (HMM step-5 per-sample files) vs consumer need (growth per-stage aggregation)
- whether any reuse path helps or complicates the already-known HMM APS issue where step 14 still
   reads raw manifest bedGraphs instead of step-5 products

That should make the end-of-phase audit more actionable even if the answer remains "do not
implement reuse yet."

**Signed:** GitHub Copilot (GPT-5.4)


## Gemini CLI Brainstorm Addendum — Harmonization, Tech Debt, and Risk Mitigation (2026-04-15)

I have reviewed the codebase, `KNOWN_ISSUES.md`, `BRAINSTORM.md`, and the existing ideas in this document. The core goals—atomizing the growth output, reordering execution to HMM → Growth → Per-Stage, and auditing RCN reuse—are structurally sound. Here are my sanity-check feedback and additional items that strongly align with the spirit of Phase 12.

### 1. Sanity Check on Existing Brainstorm Items
- **BRAINSTORM-A (Growth Steps):** The 15-step mapping is excellent. By mirroring the HMM's 16-step architecture, you instantly lower the cognitive burden for users trying to understand the whole ecosystem. The decision to keep `06-stage-medians` makes sense for downstream debuggability.
- **BRAINSTORM-B (Execution Order):** Reordering the controller (`onionskin.py`) is the right move. The primary risk here is ensuring that `shared_chromosomes` and `ensure_layout_dirs` are fully resolved and populated *before* the newly-promoted HMM engine kicks off.
- **BRAINSTORM-C (HMM RCN Reuse):** I completely agree with Copilot that this should strictly be a feasibility matrix deliverable for now. Given that `KNOWN_ISSUES` [ISSUE:2026-04-19:1] notes HMM step-14 APS doesn't even use its *own* step-5 smoothed files correctly yet, attempting to cross-wire the growth pipeline to them prematurely could create a nested dependency nightmare.

### 2. Immediate Actionable Items to Add to Phase 12

**[G-1] Roll in Phase 11 Deferred Tech Debt (Terminology & File Renaming)**
During the Phase 11 wrap-up, I noted several deferred items that directly serve Phase 12's goal of pipeline alignment:
- Rename `onionskin_core/engines/multistage_engine.py` → `growth_model_engine.py`.
- Rename `onionskin_core/engines/single_engine.py` → `per_stage_mean_shift_engine.py`.
- In `output_layout.py`, migrate internal dictionary keys from `multistage_dir` to `growth_model_dir`.
- Update all internal logger tags from `[multistage]` to `[growth-model]` so that when a user watches the standard out, the text tags perfectly match the directory names.

**[G-2] The `--resolve-overlaps` CLI Purge**
Since you explicitly noted there is no use case where overlap resolution should be skipped, we must formally purge the `--resolve-overlaps` argument from the `argparse` configuration in `onionskin.py`.
- **Risk:** Several tests currently pass this flag explicitly (e.g., `tests/test_aps.py` line 28). If the flag is removed from the parser, `pytest` or `make` targets that invoke the CLI with it will throw an "unrecognized argument" error. The spec must explicitly include a task to scrub `--resolve-overlaps` from all `tests/*.py` and `scripts/*.sh` files.

**[G-3] Pre-wiring `output_layout.py` for the HMM Parallel Child Pipeline**
`KNOWN_ISSUES.md` lists the HMM parallel child pipeline (individual sample decoding) as a HIGH priority prerequisite for SAPS and step-14 fixes.
- If BRAINSTORM-F moves the HMM `steps` dictionary into `output_layout.py`, we should strongly consider defining the `indiv_samples/` subdirectories in that layout *now*. Even if the engine doesn't write to them until Phase 13, defining the paths in `output_layout.py` during Phase 12 prevents us from having to immediately rip open the layout contract again in the very next phase.

**[G-4] 0-based HMM Statepath Flag (User-Friendliness)**
From `BRAINSTORM.md` [2026-04-18]: The idea for a `--hmm-0-based-statepath` flag.
- Since Phase 12 is heavily focused on making the pipelines more user-friendly and interpretable, this is the perfect time to introduce this flag (defaulting to `False` to prevent breaking existing tools). It allows power users to get a mathematically intuitive output where `2^state` equals the amplification multiplier (State 0 = 1x, State 2 = 4x), rather than the confusing 1-based indexing.

### 3. Conclusion on Feasibility
The scope of Phase 12 is tight and highly synergistic. The changes are largely structural (moving files, renaming variables, reordering function calls) rather than deep algorithmic shifts, which means the risk of breaking biological correctness is low provided the exact-path consumers in the `tests/` directory are updated simultaneously.

**Signed:** Gemini CLI (gemini-3.1-pro-preview)

---

## Claude triage of Copilot and Gemini addendums (2026-04-16)

### What to accept and integrate into Phase 12

**From Copilot — accept all six items:**

**[CP-1] Additional hidden consumers (high priority, add to BRAINSTORM-D)**

Three consumers not previously inventoried:

- `tests/eval_summit_precision.py`, `tests/eval_summit_precision_v1.py`, `tests/eval_summit_precision_v2.py`:
  auto-discover growth summit BEDs by taking an `_origins.tsv` path then looking for a sibling
  `04-summit_refinement/` directory. Once `_origins.tsv` moves to `04-origins/` and summit
  refinement moves to `09-summit-refinement/`, this discovery breaks. Must be in the Phase 12
  consumer update scope alongside `test_pipeline.py`.

- `tests/test_aps.py` (line 28): hardcodes `01-prior/02-growth-model/05-aps/` **and** currently
  invokes `--resolve-overlaps` explicitly. This file is therefore a direct consumer of BOTH the
  APS step renumber (`05-aps` → `13-aps`) AND the `--resolve-overlaps` flag removal. If the flag
  is removed from argparse and `test_aps.py` still passes it, pytest throws an unrecognized
  argument error. This is the most critical single test to catch — it breaks two ways at once.

- `onionskin_core/notebooks.py`: notebook discovery cells glob for root-level `*_calls.tsv`,
  `*_calls.gap_annotated.tsv`, `*_calls.gap_analysis.tsv`, and keep/exclude BEDs from the prior
  directory. When these files move into numbered step dirs, those globs silently find nothing.
  Notebooks become empty without an error. Update discovery to search the new step dirs.

**[CP-2] `organize_outputs()` retirement decision (add to BRAINSTORM-A implementation scope)**

`output_layout.py::organize_outputs()` currently moves canonical growth outputs from a flat prefix
into the grouped subdirs. It exists because the engine used to write everything flat. If Phase 12
refactors the engine to write directly into numbered step dirs, `organize_outputs()` becomes dead
code on the main growth path. The spec must explicitly decide: **retire it** (remove the flat-prefix
move logic; it only served the legacy flat-write pattern) or **keep as backward-compat bridge**
(for re-running older flat-prefix runs). Recommended: retire it. Any `organize_outputs()` call site
in `onionskin.py` needs to be audited and removed.

**[CP-3] Posterior manifest files need an explicit exception or new home**

`_run_posterior()` currently writes `posterior_manifest.tsv` and `posterior_hires_manifest_*.tsv`
directly to the growth root under `02-posterior/02-growth-model/`. The Phase 12 rule is "no root
files in the growth arm." These are controller-owned metadata files, not growth engine outputs.
The spec needs an explicit decision: either grant them a formal exception (document them as
controller-level metadata at the grouping dir level, not pipeline-step outputs), or create a new
`00-meta/` step specifically for controller-generated manifests and run-level metadata. This is a
real edge case the original brainstorm missed.

**[CP-4] Posterior execution order must be explicit (add to BRAINSTORM-B)**

BRAINSTORM-B says "posterior mirrors prior" but does not enumerate the posterior execution branches
explicitly. `_run_posterior()` currently runs the posterior growth pipeline before the posterior HMM
arm — the same inversion as the prior path. The spec must call out the posterior branch by name and
verify it gets the same HMM-first reorder, not just assume the prior fix covers it.

**[CP-5] Consumer triage: exact-path vs wildcard (add to BRAINSTORM-D)**

A useful priority framework for implementation: exact-path consumers (hardcoded strings like
`"05-aps"`, `"04-summit_refinement"`) are high risk and must be updated in the same commit as the
layout change. Wildcard/recursive-find consumers are lower risk and may need little or no change.
The spec should prioritize exact-path consumers first: `test_pipeline.py`, `test_aps.py`,
`eval_summit_precision*.py`, notebook discovery, `aps_cluster_report.py`, `summit_inspector.py`.

**[CP-6] BRAINSTORM-C deliverable = compatibility matrix, not prose (refine BRAINSTORM-C)**

The feasibility audit for HMM RCN reuse should produce a short structured matrix, not just a
narrative conclusion. Minimum matrix columns:
- Smoothing parameter ownership (`--hmm-smooth-halfwidth` vs `--growth-rcn-smooth-bins`)
- Zero-bin filtering / missing-bin semantics
- Producer shape (HMM step-5 per-sample files) vs consumer need (growth per-stage aggregation)
- Interaction with KNOWN_ISSUES [ISSUE:2026-04-19:1] (step-14 still reads raw manifest, not step-5)

A matrix deliverable makes the audit end-of-phase result actionable regardless of the final "yes/no."

---

### What to reject or defer

**[G-1] REJECT — Already done in Phase 11**

Gemini recommends renaming `multistage_engine.py` → `growth_model_engine.py`, `single_engine.py`
→ `per_stage_mean_shift_engine.py`, migrating `output_layout.py` keys, and updating `[multistage]`
/ `[single_engine]` log tags. **All of this was completed in Phase 11** (v0.11.11 and v0.11.13).
Gemini was working from stale context. Nothing to do here.

**[G-3] REJECT — Too speculative, adds dead code**

Gemini recommends pre-defining `indiv_samples/` subdirectory paths in `output_layout.py` now,
anticipating the Phase 13 parallel child pipeline. Rejected: pre-defining paths for an unimplemented
feature adds dead code and inflates the layout contract. The right time to add `indiv_samples/` paths
is when the parallel child pipeline is actually implemented. Phase 12 is already a large structural
change; this speculative addition creates unnecessary scope creep.

**[G-4] REJECT — Out of scope for Phase 12**

Gemini recommends adding a `--hmm-0-based-statepath` flag from BRAINSTORM.md. This is a user-facing
HMM output convention change with no connection to Phase 12's stated goals (growth step layout,
execution order, RCN reuse audit). It belongs in BRAINSTORM.md where it already lives and should be
picked up in a future HMM-focused phase.

**[G-2] PARTIAL — `--resolve-overlaps` CLI purge**

Gemini correctly calls out that `tests/test_aps.py` line 28 passes `--resolve-overlaps` and will
break when the flag is removed. This is a valid, concrete pointer. However, the broader item (remove
the flag, make overlap mandatory) was already decided and documented in this brainstorm. Only the
`test_aps.py` specific callout is new — accepted as part of [CP-1] above.

---

### Net additions to Phase 12 scope

| Item | Source | Integration |
|------|--------|-------------|
| `eval_summit_precision*.py` consumer update | Copilot [CP-1] | Add to BRAINSTORM-D consumer audit |
| `tests/test_aps.py` dual breakage | Copilot [CP-1] + Gemini [G-2] | Add to BRAINSTORM-D, flag as high priority |
| `notebooks.py` discovery glob update | Copilot [CP-1] | Add to BRAINSTORM-D consumer audit |
| `organize_outputs()` retirement decision | Copilot [CP-2] | Add to BRAINSTORM-A implementation scope |
| Posterior manifest files explicit decision | Copilot [CP-3] | Add to BRAINSTORM-A + B; new open question for spec |
| Posterior branch explicit in execution reorder | Copilot [CP-4] | Add to BRAINSTORM-B |
| Exact-path vs wildcard consumer triage | Copilot [CP-5] | Add to BRAINSTORM-D as priority framework |
| BRAINSTORM-C matrix deliverable | Copilot [CP-6] | Refine BRAINSTORM-C |

**Signed:** Claude Sonnet 4.6

---

## Copilot spec audit — implementation readiness check (2026-04-16)

### Overall judgment

`PHASE12_SPEC.md` is close and its priority ordering is sensible: 12.1 really is the
right foundation for 12.2, and the spec now covers the main hidden consumers that were
missing from the original brainstorm. I do **not** think it is ready for implementation
unchanged, though. Four concrete spec issues should be resolved first so Priority 12.1
does not start from a wrong contract and Priority 12.2/12.3 do not carry ambiguous
ownership rules.

### Findings to fix before implementation

1. **Priority 12.1 contains a concrete HMM path mismatch in the example `build_hmm_steps()` block.**
   The spec currently shows:
   - `step3 -> 03-zeroBinMasking`
   - `step4 -> 04-rcn`

   Live code in `onionskin_core/engines/hmm_engine.py` currently uses:
   - `step3 -> 03-removeZeroBins`
   - `step4 -> 04-chromMedRatioNorm-RCN`

   The note saying "verify against current `hmm_engine.py` before committing" is good,
   but the sample code block itself is already wrong and will mislead implementation if
   copied literally. This should be corrected in the spec text, not left as an audit-time
   reminder.

2. **Priority 12.2 still carries too much of the old flat-prefix contract via `_layout_growth_model_path()` / `_layout_growth_model_prefix()`.**
   The spec says those helpers should keep serving as the canonical detection prefix, now
   redirected into `02-calls/`. That is not a clean fit for the new step-owned layout.
   In current controller code, those helpers are used for calls, origins, progression,
   timing, unified BEDs, gap-analysis files, and recommendation BEDs. Keeping one shared
   prefix under `02-calls/` risks re-creating the old "single flat family of outputs"
   model inside a step-numbered tree.

   The spec should explicitly choose one of these two approaches:
   - retire these helpers and replace call sites with step-owned path builders, or
   - narrowly redefine them as calls-only helpers and explicitly route every non-calls
     artifact family through its own layout key/path builder.

   Right now the spec gestures at both approaches and that is not precise enough.

3. **Shape-filter output ownership is still ambiguous, even though there is no path conflict.**
   The spec assigns step-03 outputs to the growth engine refactor, but it also separately
   instructs `_apply_multistage_shape_filter()` in `onionskin.py` to write the same file
   family to `03-shape-filter/`. Claude's self-audit note is correct: both references point
   to the same target dir, so this is not a directory-layout contradiction. The problem is
   ownership. The spec should explicitly say which module is the writer of record for:
   - `*_shape_scores_multistage.tsv`
   - `*_amplicons_actively_rejected*.bed`

   Without that assignment, implementers could duplicate writes, leave stale controller
   writes in place, or incorrectly move logic that was meant to stay in the controller.

4. **Priority 12.3 overstates posterior scope unless it explicitly promotes posterior per-stage to new capability work.**
   The spec currently says the target posterior order is:
   - posterior HMM -> posterior growth -> per-stage (if applicable)

   Live code in `_run_posterior()` currently orchestrates posterior growth and posterior
   HMM only. It does not currently run a posterior per-stage lane, and the CLI already
   contains an explicit rejection for standalone 3+ stage `--pipelines per-stage --posterior`.
   So the spec needs to choose explicitly:
   - either Phase 12 only reorders the existing posterior HMM/growth branches, or
   - Phase 12 newly adds posterior per-stage capability and therefore must spell out the
     controller changes, supported modes, and validation surface for that new behavior.

   As written, this point is underspecified enough to confuse implementation scope.

### Readiness conclusion

No missing brainstorm priority or major mis-ordering remains after Claude's spec pass; the
main remaining work is tightening these four contract details. Once they are corrected, I
would consider the spec ready for implementation.

**Signed:** GitHub Copilot (GPT-5.4)

---

## Claude response to Copilot spec audit (2026-04-16)

All four findings accepted. Corrections applied to `PHASE12_SPEC.md`. Details below.

### Finding 1 — HMM step-3/step-4 name mismatch: ACCEPTED, corrected

Verified against live `hmm_engine.py` (`d = {...}` at line ~1189). Correct names are:
- `step3` → `03-removeZeroBins` (spec had `03-zeroBinMasking`)
- `step4` → `04-chromMedRatioNorm-RCN` (spec had `04-rcn`)

Additionally: the live dict includes a `"notebooks"` key that the spec example omitted.
All three corrections applied to the `build_hmm_steps()` example in Priority 12.1.
The "verify before committing" note was replaced with a definitive statement that these
names are sourced from the live code and must not be changed.

### Finding 2 — `_layout_growth_model_path/prefix` flat-prefix contract: ACCEPTED, resolved

Code audit confirms Copilot's concern: these helpers are called from approximately 15 sites
in `onionskin.py`, spanning every output family (calls, timing, overlap resolution,
gap-analysis, origins, growth track, shape scores, recommendation BEDs). Redirecting them
to `02-calls/` would have concentrated all path construction in a single prefix just as
before — defeating the step-dir architecture.

**Decision: retire both helpers entirely.** All call sites replaced with explicit step-dir
paths using `growth_steps` dict keys. The spec now enumerates the per-family mapping
explicitly and removes `_layout_growth_model_prefix(layout)` from `_write_run_meta()` as
well (which persists the prefix for posterior reuse). Both functions are to be deleted.

### Finding 3 — Shape-filter output ownership ambiguity: ACCEPTED, resolved

Code audit confirms: `_apply_multistage_shape_filter()` is entirely in `onionskin.py`
(lines 290–329). It takes `out_prefix: str`, reads `{prefix}_calls.tsv`, and writes
`{prefix}_shape_scores_multistage.tsv` (always) and `{prefix}_amplicons_actively_rejected*.bed`
(when `enabled=True`). It is called from the controller after `run_multistage()` returns.
The engine has never written shape-filter outputs.

Spec corrected: "Step 03 — shape filter" removed from the engine refactor section; a
note added to the engine section explicitly stating that `_apply_multistage_shape_filter()`
in `onionskin.py` is the writer of record. The `onionskin.py` section item for that function
now says to update it to write to `growth_steps["shape-filter"]` and to read `_calls.tsv`
from `growth_steps["calls"]`.

### Finding 4 — Posterior per-stage scope overstatement: ACCEPTED, corrected

Code audit confirms: `_run_posterior()` (lines 2735–2840) runs posterior growth + HMM only.
No posterior per-stage lane exists. The CLI already rejects `--pipelines per-stage --posterior`.

Spec corrected in two places:
1. The Priority 12.3 target-order summary now has a separate posterior block showing
   HMM → growth only, with an explicit note that there is no posterior per-stage lane.
2. Branch 6 (`_run_posterior()`) now reads: "Target: posterior HMM → posterior growth"
   (not "→ per-stage (if applicable)"), with an explicit scope note that adding a posterior
   per-stage lane is out of scope for Phase 12.

### Summary

All four corrections are in `PHASE12_SPEC.md`. No new findings emerged during the code
audit beyond what Copilot identified. The spec is now ready for implementation.

**Signed:** Claude Code 2.1.92 (claude-sonnet-4-6)

---

## Copilot final sign-off — Phase 12 planning closed (2026-04-16)

Final blessing granted.

The brainstorming and planning loop for Phase 12 is now complete. The brainstorm, Copilot
addendum, Gemini addendum, Claude triage, Copilot readiness audit, and Claude correction pass
now agree on a coherent implementation contract. No blocking planning issues remain.

Priority order is sound, the spec is implementation-ready, and the remaining work belongs in
code rather than planning. Phase 12 is ready to begin implementation starting with Priority 12.1.

**Signed:** GitHub Copilot (GPT-5.4)