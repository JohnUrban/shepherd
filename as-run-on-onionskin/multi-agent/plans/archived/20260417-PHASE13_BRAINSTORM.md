# PHASE 13 BRAINSTORM

**Status:** Pre-spec brainstorm. Not committed to ROADMAP yet.  
**Purpose:** Synthesize codebase audit into candidate Phase 13 priorities, unordered.
This doc feeds PHASE13_SPEC.md after user review and Copilot/Gemini addenda.

**User-stated Phase 13 goals:**
1. Restructure the per-stage pipeline output from a flat `03-per-stage-mean-shift/` directory
   into a numbered step-by-step layout — matching how HMM and growth pipelines are organized.
2. Three guiding principles:
   - **Atomize steps**: Each new process producing output = a new step directory.
   - **No backwards cramming**: Output from a step goes in that step's dir, not upstream.
   - **Do it the same**: Follow the same logic as HMM and growth pipelines.

---

## BRAINSTORM-A: Per-stage step formalization (numbered steps, no root files)

### Current state

The per-stage pipeline uses a single flat output directory:
```
03-per-stage-mean-shift/
  {prefix}_stage1_aggregate.bedGraph
  {prefix}_stage1_aggregate.hires_{bin}.bedGraph
  {prefix}_stage1_calls.tsv
  {prefix}_stage1_calls.bed
  {prefix}_stage1_summits.bed
  {prefix}_stage1_summits.hires_{bin}.bed
  {prefix}_stage1_putative_collapsed_repeats.tsv
  {prefix}_stage1_putative_collapsed_repeats.bed
  {prefix}_stage2_aggregate.bedGraph        (if ≥2 non-ref stages)
  ... (same set for each stage)
  {prefix}_unified_stage_calls.tsv
  {prefix}_unified_stage_calls.bed
  {prefix}_unified_collapsed_repeats.tsv
  {prefix}_unified_collapsed_repeats.bed
```

All files land flat in `03-per-stage-mean-shift/` with no step subdirectory structure.

The engine entry point is `run_per_stage_mean_shift()` in
`onionskin_core/per_stage_mean_shift_engine.py`. It receives `out_dir` (= the flat
`03-per-stage-mean-shift/` path) and writes all outputs directly there. The logical
pipeline steps inside the engine are:

| Internal step | Function(s) | What it produces |
|---|---|---|
| Aggregation | `_aggregate_stage_tracks()` | Per-stage median bedGraph from replicates |
| Normalization | `_chrom_median_norm()` / `_single_stage_fold_track()` | Normalized aggregate tracks |
| Detection | `stage1_mean_shift()` + `stage2_score()` | Per-stage calls + summit positions |
| Dedup | `dedup_calls_by_peak_proximity()` | No separate file output (in-memory filter) |
| Shape filter | `_shape_filter_calls()` | Rejected BEDs, putative collapsed-repeats |
| Hires refinement | hires loop | Hires summit BEDs per stage |
| Unification | `_unify_stage_calls()` | Unified cross-stage calls + collapsed-repeats |

`run_single_stage()` — a separate function in the same file — handles the 1-2 stage paths
(reference-backed single-stage controller). It also writes flat to `out_dir`.

### Target state (user intent: atomized steps, same-as-others discipline)

Proposed step layout under `03-per-stage-mean-shift/`:

| Step | Dir | Contents | HMM analog | Growth analog |
|------|-----|----------|------------|--------------|
| 01 | `01-aggregate/` | Per-stage aggregate bedGraphs (base + hires) | `05-medianSmoothed-RCN` | `01-growth-track/` |
| 02 | `02-calls/` | Per-stage calls TSV + BED, summits BED | `06-HMM` → `09-summitStates` | `02-calls/` + `08-summits/` |
| 03 | `03-shape-filter/` | Shape scores, putative collapsed repeats TSV+BED, rejected BED | `08-aboveBackground` | `03-shape-filter/` |
| 04 | `04-summit-refinement/` | Hires summit BEDs per stage | `13-summit-refinement` | `09-summit-refinement/` |
| 05 | `05-unified/` | `unified_stage_calls.tsv/bed`, `unified_collapsed_repeats.tsv/bed` | _(no HMM analog)_ | `11-overlapResolution/` |

**5 steps — simpler than growth's 15, appropriate for per-stage's narrower scope.**

#### Open question A1: Should summits be in step 02 or a separate step?
Currently per-stage summits (`_stage{st}_summits.bed`) are base detection outputs computed
during the same pass as calls. If we put them in `02-calls/` we avoid an extra directory.
But growth separates summits from calls (step 08 vs step 02). Principle: "Do it the same"
argues for a separate `03-summits/` dir, shifting shape-filter to `04-` and refinement to `05-`.
Counterargument: per-stage summits are just the peak coordinate from each call — much less
machinery than growth's summit pipeline. Flat inclusion in `02-calls/` may be cleaner.

**Candidate A: 5-step layout (summits in 02-calls/)**
```
01-aggregate/  02-calls/  03-shape-filter/  04-summit-refinement/  05-unified/
```

**Candidate B: 6-step layout (summits separated)**
```
01-aggregate/  02-calls/  03-summits/  04-shape-filter/  05-summit-refinement/  06-unified/
```

#### Open question A2: Where do hires aggregate tracks go?
The hires aggregate bedGraphs (`_stage{st}_aggregate.hires_{bin}.bedGraph`) are produced
by `_aggregate_stage_tracks()` — same function as the base aggregate. They should go in
`01-aggregate/` alongside base aggregate tracks. Alternatively they could go in
`04-summit-refinement/` since they are only used during the hires refinement loop.
Principle: "No backwards cramming" — they are produced upstream, so `01-aggregate/`.

#### Open question A3: What about dedup?
`dedup_calls_by_peak_proximity()` runs after detection but produces no separate file output —
it modifies the in-memory call list. No step dir needed. Document in spec that this is an
invisible in-engine filter between steps 02 and 03.

### Implementation scope

Changes required across the codebase:

1. **`onionskin_core/output_layout.py`**
   - Add `build_per_stage_steps(per_stage_dir: str) -> Dict[str, str]` function analogous
     to `build_growth_steps()` — returns mapping of step names to numbered step paths
   - Update `build_output_layout()` to expand `per_stage_mean_shift_dir` into step-level keys
     (or leave `per_stage_mean_shift_dir` as the container and pass `per_stage_steps` separately)
   - Update `ensure_layout_dirs()` `include_per_stage` branch to create all step subdirs
   - Update `_INDEX_MD_TEMPLATE` to document the step layout under `03-per-stage-mean-shift/`

2. **`onionskin_core/per_stage_mean_shift_engine.py`**
   - Refactor `run_per_stage_mean_shift()` to accept a `per_stage_steps` dict (analogous to
     HMM engine receiving `steps`) instead of a flat `out_dir` string
   - Route each output file to its appropriate step dir
   - Same refactor for `run_single_stage()` if it uses the same output dirs
   - `per_stage_steps` dict lives in `output_layout.py`, not here (same as growth)

3. **`onionskin.py`** (`_run_standalone_per_stage_controller`, lines 2293-2312)
   - Build `per_stage_steps` from `build_per_stage_steps(layout["per_stage_mean_shift_dir"])`
   - Pass `per_stage_steps` to `run_per_stage_mean_shift()` instead of `per_stage_dir`
   - Update any path references for per-stage outputs in downstream logic

4. **`tests/test_pipeline.py`** (lines 348-349, 381-382)
   - Update assertions from flat `03-per-stage-mean-shift/{prefix}_unified_stage_calls.tsv`
     to the new step-dir path (e.g., `03-per-stage-mean-shift/05-unified/{prefix}_unified_stage_calls.tsv`)
   - Add assertions for other step dirs

5. **`scripts/summit_inspector.py`**
   - Line 85: `"per-stage": "03-per-stage-mean-shift"` — this is the container dir name, may not
     need to change if inspector does its own file search within the dir
   - Line 363: placeholder comment — the step layout may enable future inspector support;
     document updated step paths so inspector can eventually look in `05-unified/`

---

## BRAINSTORM-B: run_single_stage() scope question

`run_single_stage()` in `onionskin_core/per_stage_mean_shift_engine.py` handles 1-2 stage
reference-backed runs. It currently writes to the same flat `out_dir`. The Phase 13 step
layout applies to the full multi-sample batch engine. Two options:

**Option B1: Shared step layout** — `run_single_stage()` also writes to step dirs.
Pro: consistent layout regardless of stage count.
Con: more work; single-stage is simpler and the step dirs might look sparse.

**Option B2: Step layout only for run_per_stage_mean_shift()** — `run_single_stage()`
stays flat or gets its own simpler layout.
Con: inconsistency; `03-per-stage-mean-shift/` layout depends on which path was taken.

Recommendation: Option B1 — same layout. Sparse step dirs are fine; they match the HMM's
approach (some steps produce minimal output on short runs).

---

## BRAINSTORM-C: `build_per_stage_steps()` interface design

The growth pipeline uses `build_growth_steps(growth_model_dir)` in `output_layout.py`
and passes a `growth_steps` dict to the engine. HMM uses a local `steps` dict inside
`hmm_engine.py` built from the `hmm_out_dir` argument. The cleaner Phase-12 approach
(growth) puts the step-dir definitions in `output_layout.py`.

For per-stage, follow the growth pattern:
```python
def build_per_stage_steps(per_stage_dir: str) -> Dict[str, str]:
    return {
        "aggregate": os.path.join(per_stage_dir, "01-aggregate"),
        "calls": os.path.join(per_stage_dir, "02-calls"),
        "shape-filter": os.path.join(per_stage_dir, "03-shape-filter"),
        "summit-refinement": os.path.join(per_stage_dir, "04-summit-refinement"),
        "unified": os.path.join(per_stage_dir, "05-unified"),
    }
```
(Adjust numbering based on resolution of Q-A1.)

---

## BRAINSTORM-D: BRAINSTORM.md per-stage items

From the 2026-04-11 BRAINSTORM.md entry `"02-per-stage-mean-shift/ completeness phase"`:
> Desired end state: `per-stage` should be able to stand on its own when run alone and
> should provide artifacts generated within `02-per-stage-mean-shift/` rather than borrowing
> from other pipelines.

Phase 13 directly advances this goal — step dirs are a prerequisite for adding future
per-stage-specific outputs (genome-wide stage median tracks, APS-equivalent, timing analog).
No new per-stage outputs are in scope for Phase 13 beyond the structural reorganization.

The BRAINSTORM.md note about `indiv_samples/` subdir (per-sample per-stage smoothed
bedGraphs — Phase 11.4 analog) is not in scope for Phase 13.

---

## Open Questions for John M. Urban

1. **Summits in 02-calls/ or separate 03-summits/?**
   - Candidate A (5-step): summits live in `02-calls/` — simpler, fewer dirs
   - Candidate B (6-step): summits get their own dir to match growth more closely
   - Lean: Candidate A — per-stage summits are the peak coord of each call, not a separate
     pipeline stage worth isolating, especially without a growth-equivalent 03→08 gap

2. **run_single_stage() included?**
   - Confirmed recommendation: yes — both code paths use the same step layout.
   - Override if you see a reason for different treatment.

3. **summit_inspector.py placeholder (line 363): fix in Phase 13 or defer?**
   - Phase 13 creates the structural prerequisite (step dirs + known unified path).
   - Wiring the inspector to use `05-unified/` could be a Phase 13 stretch goal.
   - Defer recommendation: document updated paths in the placeholder comment; full inspector
     support is a separate follow-on task (per-stage still lacks genome-wide stage medians
     and origins needed for full inspector overlays).

4. **`_INDEX_MD_TEMPLATE` update**: should the index table gain a per-stage step row
   block matching the growth pipeline's row block? Or keep a single summary row?
   - Recommendation: add a row block (same discipline as growth).

---

## Feedback from John M. Urban

**Note** - I moved the target file to "multi-agent/plans/PHASE13_BRAINSTORM.md". 

#### Question A1:
- We should do 02-calls and 03-summits. I believe summits is a separate process. The amplicon call is the massive 100-500+ kb breadth. The summit is an extra step. Plus, it is nice to have separated in my opinion.
- Regarding, 01-aggregate -- it seems like there is more than one thing going on there. Can you list all files and all processes going into that dir? Is it both per--stage medians from replicates as well as then normalization? Because that would be two steps similar to HMM pipeline's 02-unionStats and 04-chromMedianNorm-RCN (or 01-mednorm). 
- 01-aggregate and 05-unified are sort of confusing as single words. In fact, can you try to make each directory here two words so I can better predict what is in it? Is "aggregate" the best term here? I see the function has that name, but we could change both if we come up with a better term. 
- Based on the steps you laid out in the first table, it seems like the directories should be:
    - 01-aggregation (or other name)
    - 02-normalization
    - 03-detection
    - 04-summits
    - 05-deduplication
    - 06-shape-filtration
    - 07-summit-refinement
    - 08-multistage-unification
- 8 steps -- more atomized, easier logic to follow for a human user in my opinion.

#### Question A2:
- If they are produced at the same time in the 01-aggregate step, then they go there. 
- I propose making sub-directories under 01-aggregate for bin size though OR keep the primary on the top level under 01-aggregate, but put the others in subdirectories named hires-<binsize>/.

#### Question A3:
- I see deduplication is invisible.
- I actually propose making it "visible" - wouldn't that be more parallel with the growth pipeline? I believe growth has that as two outputs and two steps.

#### Question: BRAINSTORM-B layout
- I go for Option B1, Shared Step Layout
- the single file / single stage / single stage + reference versions of the per-stage pipeline should feel like part of the continuum of what the pipeline does in general

#### RE: open questions

**Open Q1:** 
- See answer to Q A1 above -- I think 8 steps.

**Open Q2:** 
- Same lay out, part of the same continuum of the per-stage pipeline. If the code treats the single file / single stage per-stage run as a separate pipeline, then we should expand the scope to better unify the per-stage pipeline into something that does not look like two distinct things that we pretend is one thing, but something on a continuum. If that makes sense to do - we can try. If not, then not.

**Open Q3:**
- Yes we should expand the scope of Phase 13 to include upgrading anything that touches or is touched by the per-stage pipeline. This phase should not break anything or make anything stale. It should be about making the per-stage pipeline feel more like part of the pipeline family, and not some discarded side project.

**Open Q4:**
- Add a row block (same as growth). If I am honest, I did not understand this question - I'd have to dig into the code more to see what you mean. Feel free to explain it in simple terms for my benefit.

#### Add-ons for Phase 13:

Let's expand the scope of Phase 13 to also including:
- (i) prettying up miscellaneous areas of the pipeline and unifying some terms across pipelines, and 
- (ii) making the per-stage pipeline a little meatier in places that would be pretty trivial to do like APS, clustering stuff, plots, gap analysis, and amplicon metrics.

**Examples of "expansion i": Misc Prettification**
- for the HMM pipeline, 02-unionStats directory should probably be renamed. We output that union stats file, but it is not used, and not totally interesting, although we can keep it. The bedGraph is really the thing we want. It should probably have a name that is parallel to the same step in per-stage and/or growth pipelines. We should try to harmonize names more in general - if the same conceptual or functional thing is happening (even if slightly differently in each pipeline).
- for HMM pipeline, 04-chromMedRatioNorm-RCN might beed a more generic name because some times it won't be ratio norm. Perhaps just 04-chromMedReNorm (for re-normalizing since it is the second norm step). 

**Examples of "expansion ii": making it meatier and more robust.**
- APS / clustering and posterior manifest building, posterior runs
- Summit refinement should include all the bits we use for growth pipeline, including the parabola estimator, which I believe is doing the best. I looked at the per-stage summits, and they are bizarrely right shifted at II/9A -- seems like some systematic artifact...
- I noticed on a recent run that there were very few per-stage calls and it did not look to good in general, so we should do some comparisons to our gold standard results (hmm) or the "by-eye" amplicons, all in addition to summit optimization with the II/9A summit evaluations (and II/2B).
- catchall: as said above: APS, clustering stuff, plots, gap analysis, and amplicon metrics, other things that area not "new" analyses.

---

## Claude Triage (Round 1 — after user feedback)

### Resolved: 8-step layout (confirmed)

User confirmed 8 steps. Resolved step layout under `03-per-stage-mean-shift/`:

| Step | Dir | Contents | Notes |
|------|-----|----------|-------|
| 01 | `01-stage-aggregates/` | Per-stage median tracks from replicates | Base: top-level; hires: `hires-<binsize>/` subdirs |
| 02 | `02-normalization/` | Chrom-median-normalized or fold-change aggregate tracks | Parallels HMM `04-chromMedRatioNorm-RCN` (see renaming below) |
| 03 | `03-detection/` | Per-stage calls TSV + BED from mean-shift detection | Raw calls before dedup/shape-filter |
| 04 | `04-summits/` | Per-stage summit BEDs | Separated from calls per user direction; summit = extra step |
| 05 | `05-deduplication/` | Dedup report (pre/post counts per stage, removed calls) | Made visible; see dedup visibility discussion below |
| 06 | `06-shape-filter/` | Shape scores TSV, putative collapsed repeats TSV+BED, rejected BED | Matches growth naming (`06-shape-filter`, not `shape-filtration`) |
| 07 | `07-summit-refinement/` | Hires summit BEDs per stage | Parallels growth `09-summit-refinement/` |
| 08 | `08-stage-unification/` | `unified_stage_calls.tsv/bed`, `unified_collapsed_repeats.tsv/bed` | **NOT** "multistage-unification" — "multistage" is a deprecated term in this codebase (Phase 11 rename); "stage-unification" is the correct parallel |

**Step name rationale:**
- `01-stage-aggregates`: output-describing, two words, avoids vague "aggregation" noun. HMM
  analog is `01-mednorm` (process-describing); growth analog is `01-growth-track`
  (output-describing) — we follow growth's pattern here.
- `02-normalization`: one word but sufficiently descriptive; could be `02-chrom-renorm` for
  stronger HMM parallelism (see HMM renaming below) — left open for spec.
- `03-detection`: one word, clear; `03-ms-detection` is an alternative if two-word is desired.
- `04-summits`: user-confirmed; one word, clear.
- `05-deduplication`: user-confirmed.
- `06-shape-filter`: match growth naming exactly (user's "shape-filtration" is acceptable but
  inconsistent with growth; prefer `shape-filter` for cross-pipeline name harmonization).
- `08-stage-unification`: avoids deprecated "multistage" term. Alternatives: `08-unified-calls`,
  `08-cross-stage`. "stage-unification" is most parallel to growth's `11-overlapResolution`.

---

### Resolved: Aggregation vs Normalization (two steps)

Yes — these are two distinct steps already in the engine:
1. `_aggregate_stage_tracks()` — computes the per-stage median bedGraph across replicates.
   Input: per-sample per-stage individual bedGraphs. Output: one aggregate bedGraph per stage.
2. `_chrom_median_norm()` / `_single_stage_fold_track()` — normalizes the aggregate track
   by chromosome median (or computes fold-change relative to a reference stage).
   Input: aggregate bedGraphs. Output: normalized aggregate bedGraphs.

These parallel HMM steps `01-mednorm` (initial normalization of individual samples) and
`04-chromMedRatioNorm-RCN` (second normalization after union stats). They should be two
separate step dirs. **8-step layout confirmed.**

---

### Resolved: Hires aggregates layout

Hires aggregate bedGraphs go in `01-stage-aggregates/` (produced upstream, no backwards
cramming). Subdirectory layout:
```
01-stage-aggregates/
  {prefix}_stage1_aggregate.bedGraph        (base resolution)
  {prefix}_stage2_aggregate.bedGraph
  hires-{bin}/
    {prefix}_stage1_aggregate.hires_{bin}.bedGraph
    {prefix}_stage2_aggregate.hires_{bin}.bedGraph
```
If multiple hires bin sizes exist, each gets its own `hires-{bin}/` subdir.

---

### Resolved: Dedup visibility

User wants dedup visible. Two options discussed:

**Option D1 (lighter):** Write a per-stage dedup summary report in `05-deduplication/`:
`{prefix}_stage{st}_dedup_report.tsv` — columns: stage, n_before, n_after, n_removed.
No pre-dedup call files needed; just the accounting record.

**Option D2 (heavier):** Write pre-dedup calls to `05-deduplication/` and post-dedup to `03-detection/`
(or vice versa), making both states inspectable.

**Recommendation D1** — a dedup report is the least invasive way to make dedup visible without
duplicating the full call table. But this is an open question for the spec: does user want the
full pre-dedup call table as well, or just the report?

Note: In the growth pipeline, dedup is also in-memory with no separate file output — so making
per-stage dedup visible would actually put per-stage *ahead* of growth in transparency.

---

### Resolved: run_single_stage() unification

Option B1 confirmed — `run_single_stage()` uses the same 8-step layout. If the code currently
treats single-stage as a separate path (writing flat to `out_dir`), Phase 13 should refactor it
to share the same step-dir routing logic. Whether this requires a full merge of the two code
paths or just a shared `per_stage_steps` dict parameter depends on code inspection during spec
authoring.

---

### Resolved: Open Q4 — what _INDEX_MD_TEMPLATE is

`_INDEX_MD_TEMPLATE` is the content of the `00-INDEX.md` file written to each grouping directory
(`01-prior/` and `02-posterior/`). It contains a reader's guide table listing every subdirectory
and its contents, so users opening the output directory in Finder or `ls` know what each folder holds.

Currently the growth pipeline has a detailed block of rows (one per step dir). Currently the
per-stage line is a single summary row:
```
| `03-per-stage-mean-shift/` | Per-stage independent mean-shift calls | ... |
```

After Phase 13, this single row should expand to a block of 8 rows — one per step dir — just
like the growth block. That's all "add a row block" means.

---

### New Priority: HMM directory renaming (Expansion i)

User-confirmed HMM directory renaming as part of Phase 13 prettification:

| Current HMM dir | Proposed rename | Rationale |
|---|---|---|
| `02-unionStats` | `02-union-tracks` | "union-tracks" describes the output (union bedGraph); "Stats" is misleading since the key output is the bedGraph, not stats. Parallels per-stage `01-stage-aggregates` in describing the output. |
| `04-chromMedRatioNorm-RCN` | `04-chrom-renorm` | "chrom-renorm" is generic (doesn't assume ratio); parallels per-stage `02-normalization`. User's suggestion. |

**Implementation note:** HMM directory names are hardcoded in `build_hmm_steps()` in
`onionskin_core/output_layout.py` and must match what `hmm_engine.py` expects.
Tests that check HMM step dirs (`tests/test_pipeline.py`) must also be updated.
The `_INDEX_MD_TEMPLATE` HMM section must be updated.

Additional cross-pipeline name harmonization: audit whether step descriptions in
`_INDEX_MD_TEMPLATE` use consistent vocabulary across HMM, growth, and per-stage sections
(e.g., "normalization", "detection", "summit refinement"). This is documentation-only and
can be batched with the template update.

---

### New Priority: Per-stage pipeline "meatier" additions (Expansion ii)

Ordered by estimated complexity (easier → harder):

**ii-A. Profile plots** (easy)
Existing `write_profile_plots()` or equivalent can be wired to per-stage. Each stage's
aggregate track + calls can be plotted. Output → `07-summit-refinement/plots/` or a new
`09-plots/` step dir (if added).

**ii-B. Parabola estimator for per-stage summits** (medium)
Per-stage summits are currently the raw peak coordinate from the mean-shift output. The
growth pipeline uses a parabola fit for summit localization. User notes the per-stage summit
at II/9A is systematically right-shifted — likely a symptom of using raw peak rather than
parabola. Wire the existing parabola estimator to `07-summit-refinement/`.

**ii-C. Summit right-shift bug at II/9A** (investigation)
User observed systematic right-shift in per-stage summits at II/9A. This needs a comparison
against growth pipeline summit output and/or HMM step-9/13 summit estimates at the same locus.
Resolution may be (1) the parabola fix above, (2) a halfwidth or window parameter artifact,
or (3) a normalization asymmetry. Add to spec as a diagnostic subtask.

**ii-D. Gap analysis** (easy — existing code)
`gap_analysis.py` functions are already used by the growth pipeline. They can be applied to
per-stage `08-stage-unification/` unified calls. Output → new `09-gap-analysis/` step dir
(if added). This requires knowing the chromosome sizes (already available in the engine context).

**ii-E. Amplicon metrics** (medium — adapting HMM step-11)
HMM step-11 (`11-amplicon-metrics/`) emits per-amplicon metrics. An analog for per-stage
could compute per-stage call metrics (width, amplitude, shape score summary). Output format
should be compatible with the other pipelines' metric tables.

**ii-F. APS and clustering** (large)
APS requires defining a per-stage analog of the growth-pipeline APS score (currently based
on stage-median amplitude at each locus). For per-stage, the natural input is the per-stage
aggregate track amplitude at each called locus. This is a non-trivial new analysis.
Posterior manifest building and posterior reruns depend on APS/clustering — these are also
large and should probably be a sub-priority within Phase 13 or deferred to Phase 14.

**ii-G. Quality comparison vs HMM** (investigation + tooling)
User notes per-stage calls "did not look too good" on a recent run and wants comparison
against HMM or by-eye amplicons. This is a diagnostic/validation task, not a pipeline
feature. Could be a notebook or a standalone comparison script. Suggests a call-level
agreement metric (overlap fraction, summit distance) between per-stage and HMM outputs.

**ii-H. Posterior support** (largest — deferred recommendation)
`_run_standalone_per_stage_controller` already has a block explicitly blocking posterior:
```
"standalone 3+ stage --pipelines per-stage does not yet support --posterior"
```
Posterior for per-stage requires APS + clustering (ii-F) as a prerequisite. Recommend
deferring to Phase 14.

**Phase 13 ii-scope recommendation:** Target ii-A through ii-E as Phase 13. Treat ii-F (APS
+ clustering) as a Phase 13 stretch goal if ii-A–E are clean. Defer ii-G (comparison), 
ii-H (posterior) to Phase 14 as they are investigative/large.

---

### Revised scope for Phase 13

**Core (step-dir restructuring):**
- 8-step `03-per-stage-mean-shift/` layout
- `build_per_stage_steps()` in `output_layout.py`
- `run_per_stage_mean_shift()` and `run_single_stage()` refactored to route to step dirs
- `_run_standalone_per_stage_controller` updated in `onionskin.py`
- `tests/test_pipeline.py` updated
- `_INDEX_MD_TEMPLATE` updated (per-stage 8-row block added)
- Dedup report (make dedup visible)

**Prettification (expansion i):**
- HMM `02-unionStats` → `02-union-tracks`
- HMM `04-chromMedRatioNorm-RCN` → `04-chrom-renorm`
- `build_hmm_steps()` updated in `output_layout.py`
- `_INDEX_MD_TEMPLATE` HMM section updated
- `tests/test_pipeline.py` HMM step assertions updated

**Meatier per-stage (expansion ii, Phase 13 scope):**
- Profile plots per stage (ii-A)
- Parabola summit estimator wired to per-stage `07-summit-refinement/` (ii-B)
- Summit right-shift diagnostic at II/9A (ii-C)
- Gap analysis for per-stage unified calls (ii-D)
- Amplicon metrics (ii-E)

**Stretch goal (if time allows):**
- APS and clustering for per-stage (ii-F)

**Deferred to Phase 14:**
- Posterior support for per-stage (ii-H)
- Cross-pipeline quality comparison tooling (ii-G)

---

## Feedback from John M. Urban (Round 2)

#### RE: naming correction to flag
- `08-stage-unification` is fine
- `multistage` was deprecated as a reference to the growth pipeline, not as a reference to the stages
- we now treat `multistage` as a property of the input dataset
- if the input dataset has multple stages, then `multistage` can still be a reference to the multiple stages
- `08-multistage-unification` makes more sense if unification is happening across multiple stages, not within a stage

#### RE: 06-shape-filter
- Yes, that name makes more sense. Good point.

#### RE: Dedup visibility level
- Let's go with both options actually, in the following way
- the pre-dedup BED files should be in 
03-detection
- the light weight dedup report TSV should go with the dedup BED files in `05-deduplication`
- my interest in the pre-dedup BED files is to be able to monitor what the pipeline is seeing and doing

#### RE Open Question 1 - name for 01
- `01-stage-aggregates` is better than before but the file description you use is literally "per-stage medians", so why not `01-stage-medians`?
- I vote for `01-stage-medians`

#### RE Open Question 2 - 02-chrom-renorm
- I like `04-chrom-renorm` for HMM
- I like `02-chrom-renorm`, but I would need to know two things: (1) is it chromosome-specific normalization?, and (2) is it the second time some form of median normalization is happening to earn the "re" in "renorm"?
- if it is chromosome-specific it earns the "chrom-norm"
- if it is also a second normalization, it earns "chrom-renorm"
- so perhaps `02-chrom-norm` or `02-chrom-renorm`

#### RE HMM renames
- RE: "02-unionStats → 02-union-tracks" - the only reason to hang on to the word "union" is that I imagine the function in the code is called that because the script that PuffStep used was called that. But I don't think it is super intuitive for the bedGraphs in there, which are per-stage medians. Since they are per-stage medians, we should give this the same type of name we gave per-stage medians in the per-stage pipeline. Here it would be `02-stage-medians`. So "02-unionStats → 02-stage-medians". And the text files in that directory that end in "unionBedGraphStats.txt" should be renamed to end with "bin-stats.txt or "positional-bin-stats.txt" because those are much more fitting descriptions of what is in the file. The "unionBedGraphStats" thing is sort of a vestige of using BEDtools "unionbedg".
- RE: 04-chrom-renorm -- yes I like this.

#### RE Expansion ii Phase 13 Scope
- I agree with adding Profile plots, parabola summit estimator + II/9A diagnostic, gap analysis, amplicon metrics.
- I think APS/clustering is a regular goal, not a stretch goal. We have as much time as we want, so time does allow. We have the amplicon intervals. We have the ability to get the sample-specific RCN. We have all the machinery to compute Area Excess, Summit, Width, etc. Everything is basically ready to go. We are not limited by time or anything. APS can be computed if there is any number of stages including only 1 (so >= 1). But clustering does not make sense to do unless there are 2+ stages with at least 3-4 samples total. The samples might have better groupings even when there are 2 input stages. This same logic should also be at play in the HMM pipeline. Is it?
- Let's keep posterior groups, ordering, and manifests in Phase 13, not Phase 14. If we compute APS and do clustering, a product of that is literally the posterior grouping and therefore the posterior manifest. Then the whole pipeline just runs again on the updated manifest. It makes sense with the new goal of Phase 13 to make the per-stage pipeline meatier, and it we are 99% of the way there if we compute APS and clustering, which I think we should. This could be the last Priority of Phase 13. This should be an analysis when there are 2+ stages with at least 3-4 samples total (same as for clustering). The same caveat should hold true for the HMM pipeline. 
    - There is no blocker to this in my mind. It just needs to come after the APS/clustering Priority, or implemented at the end of the same Priority. Any blockers written might just be stale.
- To the plots we generate, I think we can easily add the aps plots (if we compute APS anyway) and the summit_fit plots analogous to those in the growth pipeline. We could probably also do the "genome_overview" plots.
- Cross-pipeline quality comparison -- cross-pipeline synthesis should not be part of Phase 13. But it would be perfectly fine to compare the BED intervals from per-stage to the various level of ground truth we have for amplicons across the genome. For example, `tests/full_chrom_training_data/amplicons.by-eye.bed` for chromosome II, or `dev/puffstep/automate/5kb-corechrom-v2/08-aboveBackground/5.TPM.medNorm.stage_5_median.zeroBinsRemoved.RCN_pseudo0.medSmoothedHW3.expdecay.states.collapsed.aboveBackgroundRegions.bed` which is consistent with by-eye identification in IGV. This Priority would not be about comparing pipelines as much as doing some QC of the per-stage pipeline. Summit evaluations at II/9A would be even more important for Phase 13, but both are okay to look at. It will give us an idea of how to tune the per-stage parameters. Are the current set giving us too few calls? too many? we should find out and adjust it.


---

## Claude Triage (Round 2 — after user Round 2 feedback)

### Resolved: Final 8-step directory names

| Step | Dir | Status | Changed from Round 1? |
|------|-----|--------|-----------------------|
| 01 | `01-stage-medians/` | **Confirmed** | `01-stage-aggregates` → `01-stage-medians` (user: "the file description you use is literally per-stage medians, so why not 01-stage-medians?") |
| 02 | `02-chrom-norm/` | **Confirmed** | `02-normalization` → `02-chrom-norm` — see chrom-norm vs chrom-renorm resolution below |
| 03 | `03-detection/` | **Confirmed** | unchanged; now explicitly holds pre-dedup calls (see dedup layout below) |
| 04 | `04-summits/` | **Confirmed** | unchanged |
| 05 | `05-deduplication/` | **Confirmed** | now holds dedup report TSV + post-dedup calls; pre-dedup calls stay in `03-detection/` |
| 06 | `06-shape-filter/` | **Confirmed** | unchanged |
| 07 | `07-summit-refinement/` | **Confirmed** | unchanged |
| 08 | `08-multistage-unification/` | **Confirmed** | `08-stage-unification` → `08-multistage-unification`. User clarifies: "multistage" was deprecated only as a name for the growth pipeline. As a property of a multi-stage dataset, it is not deprecated and remains the right adjective here — unification is across multiple stages. |

---

### Resolved: `02-chrom-norm` vs `02-chrom-renorm`

User asked two questions:
1. **Is it chromosome-specific?** YES — `_chrom_median_norm()` divides each bin by the
   chromosome median of the aggregate track. This earns the `chrom-` prefix.
2. **Is it the second normalization, earning the `re-`?** NO — the preceding step
   (`01-stage-medians`) is a *aggregation* (median of replicates per bin per stage), not a
   normalization. Step 02 is the first (and only) normalization applied to the aggregate.
   Therefore `re-` is not earned.

**Resolution: `02-chrom-norm`.**

Contrast with HMM `04-chrom-renorm` — there, `01-mednorm` IS a prior normalization step
(per-sample median normalization), so the chromosome-median step at step 04 IS a
*re*-normalization. HMM `04-chrom-renorm` earns both `chrom-` and `re-`.

---

### Resolved: Dedup layout (combined D1 + D2)

User confirmed: use both approaches together.

```
03-detection/
  {prefix}_stage{st}_calls.tsv           ← pre-dedup calls (TSV + BED)
  {prefix}_stage{st}_calls.bed           ← pre-dedup calls BED

05-deduplication/
  {prefix}_stage{st}_dedup_report.tsv    ← lightweight report (n_before, n_after, n_removed)
  {prefix}_stage{st}_calls.bed           ← post-dedup calls BED (canonical filtered set)
```

`04-summits/` summit BEDs correspond to the pre-dedup calls (they are point coordinates
extracted from detected calls before the proximity-based dedup pass). The shape-filter
step (`06-shape-filter/`) operates on the post-dedup calls from `05-deduplication/`.

**Motivation** (user): "my interest in the pre-dedup BED files is to be able to monitor
what the pipeline is seeing and doing." The pre-dedup calls in `03-detection/` provide
full transparency into what the detector found before any collapsing step is applied.

---

### Resolved: HMM `02-stage-medians` rename + file rename

User confirms: `02-unionStats` → `02-stage-medians` (same concept as per-stage `01-stage-medians`).
The "union" terminology is a PuffStep/BEDtools vestige; the content is per-stage median bedGraphs.

File rename in the same step:
- `*_unionBedGraphStats.txt` → `*_positional-bin-stats.txt`
  (user proposed "bin-stats.txt" or "positional-bin-stats.txt"; `positional-bin-stats` is more
  informative — the file records per-bin coverage statistics at each genomic position)

Updated HMM prettification table:

| Current | Proposed | Rationale |
|---|---|---|
| `02-unionStats` | `02-stage-medians` | Content is per-stage median bedGraphs; "union" is a PuffStep artifact |
| `04-chromMedRatioNorm-RCN` | `04-chrom-renorm` | Second normalization (re-), chromosome-specific; avoids assuming ratio mode |
| `*_unionBedGraphStats.txt` | `*_positional-bin-stats.txt` | File holds per-bin positional statistics; "union" and "BedGraph" are misleading |

---

### Resolved: APS/clustering — regular Phase 13 goal, not stretch

User confirms: APS and clustering are regular Phase 13 goals. Rationale: all machinery
(amplicon intervals, per-sample RCN, Area Excess, Summit, Width, etc.) is already available;
the analysis is ready to implement.

**APS rule (≥1 stages):** APS can be computed for any run with at least 1 non-reference stage.
**Clustering rule (2+ stages, 3-4+ samples):** Clustering only runs when ≥2 stages and at
least 3-4 samples total are available (point estimates with fewer samples are unstable).

**Open audit question:** Does the HMM pipeline currently enforce the same APS/clustering
rules (APS for ≥1, clustering for 2+ stages + 3-4+ samples)? This must be verified during
spec authoring and harmonized if not. Flag as a spec audit item.

---

### Resolved: Posterior groups/ordering/manifests — Phase 13 (not Phase 14)

User confirms: if APS + clustering are implemented, posterior grouping and manifest building
follow naturally as a Phase 13 deliverable. "If we compute APS and do clustering, a product
of that is literally the posterior grouping and therefore the posterior manifest."

The current blocker comment in `_run_standalone_per_stage_controller` (lines 2266-2272):
```
"standalone 3+ stage --pipelines per-stage does not yet support --posterior"
```
User: "There is no blocker to this in my mind. Any blockers written might just be stale."
This must be audited during spec to determine whether the blocker is still technically valid
or whether it was written when APS/clustering were not yet planned for per-stage.

**Sequencing:** Posterior must come after APS+clustering are implemented; it should be the
final Priority in Phase 13.

**Same rule for HMM:** Posterior clustering logic (2+ stages + 3-4+ samples) should apply
equally to the HMM pipeline. Verify during spec.

---

### Resolved: Plots scope expanded

Updated `ii-A. Profile plots` scope: in addition to basic profile plots, if APS is computed:
- APS plots (per-locus APS score visualization, analogous to growth `14-plots/aps/`)
- Summit-fit plots (parabola fit diagnostics, analogous to growth summit-fit plots)
- Genome-overview plots (amplicon-level overview across genome, analogous to growth
  `14-plots/genome_overview/`)

All three are natural outputs of the APS + parabola summit refinement work and should be
produced by the same plot-writing pass.

---

### Resolved: Cross-pipeline QC reframed as per-stage QC

Renamed from "cross-pipeline quality comparison" (ii-G) to **per-stage QC / parameter tuning**.
Scope: compare per-stage BED calls against known ground truth, not against other pipelines.

Ground truth sources available:
- `tests/full_chrom_training_data/amplicons.by-eye.bed` — chromosome II by-eye amplicons
- `dev/puffstep/automate/5kb-corechrom-v2/08-aboveBackground/5.TPM.medNorm.stage_5_median.zeroBinsRemoved.RCN_pseudo0.medSmoothedHW3.expdecay.states.collapsed.aboveBackgroundRegions.bed` — HMM-consistent chr II calls

Key questions the QC will answer:
- Are per-stage calls systematically too few or too many?
- Is the II/9A summit right-shift resolved by parabola estimator, or is there a deeper cause?
- Summit evaluation at II/2B as a second locus.
- Are parameters (z-thresh, halfwidth, etc.) well-tuned for the full genome?

This is a **diagnostic and parameter-tuning Priority**, not a cross-pipeline synthesis task.
Output: a recommendation report on parameter adjustments if needed, plus summit evaluation
plots for II/9A and II/2B.

**Scope change from Round 1:** QC is now a regular Phase 13 goal (was deferred to Phase 14).

---

### Updated Phase 13 scope (supersedes Round 1 scope summary)

**Priority 13.1 — 8-step per-stage layout (core restructuring)**
- `build_per_stage_steps()` in `output_layout.py` with 8 step dirs
- `run_per_stage_mean_shift()` and `run_single_stage()` accept `per_stage_steps` dict
- `_run_standalone_per_stage_controller` in `onionskin.py` updated
- `_INDEX_MD_TEMPLATE` updated: per-stage 8-row block
- `tests/test_pipeline.py` updated
- Pre-dedup calls in `03-detection/`; dedup report + post-dedup calls in `05-deduplication/`

**Priority 13.2 — HMM prettification + name harmonization**
- `02-unionStats` → `02-stage-medians` in `build_hmm_steps()`
- `04-chromMedRatioNorm-RCN` → `04-chrom-renorm` in `build_hmm_steps()`
- `*_unionBedGraphStats.txt` → `*_positional-bin-stats.txt` (file rename in HMM engine)
- `_INDEX_MD_TEMPLATE` HMM section updated
- `tests/test_pipeline.py` HMM step assertions updated
- Audit `_INDEX_MD_TEMPLATE` vocabulary across all three pipelines for consistency

**Priority 13.3 — Per-stage summit refinement (parabola estimator) + diagnostics**
- Wire existing parabola estimator to per-stage `07-summit-refinement/`
- Summit evaluation at II/9A (right-shift investigation) and II/2B
- Compare against growth pipeline + HMM summit estimates at same loci
- Hires summit BEDs produced here (currently bundled with base detection)

**Priority 13.4 — Per-stage expansion: gap analysis + amplicon metrics + profile plots**
- Gap analysis for unified calls → new `09-gap-analysis/` step dir (or appended to `08-multistage-unification/`)
- Amplicon metrics (width, amplitude, shape score summary per call) — analog of HMM step-11
- Profile plots per stage (aggregate track + calls)

**Priority 13.5 — Per-stage APS + clustering**
- Compute APS for ≥1 stages using per-stage aggregate track amplitude at called loci
- Cluster when 2+ stages + 3-4+ samples (same rule as growth pipeline; verify same rule in HMM)
- APS plots + summit-fit plots + genome-overview plots
- Audit whether HMM APS/clustering enforces the same ≥1/≥2+3-4 rule

**Priority 13.6 — Per-stage QC / parameter tuning**
- Overlap per-stage calls against `amplicons.by-eye.bed` and PuffStep HMM BED for chr II
- Report: too few / too many? sensitivity/specificity at known loci?
- Summit evaluations at II/9A and II/2B (tie-in with Priority 13.3 diagnostics)
- Parameter adjustment recommendation if warranted

**Priority 13.7 — Per-stage posterior (groups, ordering, manifests)**
- Requires Priorities 13.5 (APS+clustering) as prerequisite
- Audit blocker comment in `_run_standalone_per_stage_controller` — is it still valid?
- Posterior grouping and manifest building following growth pipeline pattern
- Apply same 2+ stages + 3-4+ samples rule for clustering before posterior
- Wire `--posterior` support for per-stage

---

## Copilot Addendum

### Audit focus

Read-only audit of the live per-stage pipeline surfaces, adjacent pipeline-family layout
surfaces, and nearby backlog/docs with emphasis on things not yet captured clearly in this
brainstorm. Audited surfaces included:

- `onionskin_core/per_stage_mean_shift_engine.py`
- `onionskin.py` per-stage controller paths
- `onionskin_core/output_layout.py`
- `onionskin_core/notebooks.py`
- `tests/test_pipeline.py`
- `scripts/summit_inspector.py`
- `multi-agent/BRAINSTORM.md`
- `multi-agent/KNOWN_ISSUES.md`

### What looks strong already

- The 8-step layout direction is strong and consistent with the user's stated goals.
- The renamed per-stage steps are materially clearer than the earlier flat layout.
- The decision to make dedup visible is good and improves user transparency.
- The separation between structural Phase 13 work and later/meatier per-stage feature work is
  conceptually right, even if a few later priorities still need sharper boundaries.
- The HMM prettification / naming harmonization work fits the "pipeline family" goal and does
  not feel like arbitrary scope creep.

### Missing or under-specified items

- **Dedup file routing needs an explicit example.** The brainstorm now says pre-dedup calls stay
  in `03-detection/` and post-dedup outputs plus report go in `05-deduplication/`, but it still
  needs a concrete file-level example so the spec can be unambiguous about TSV/BED placement and
  which file is canonical after dedup.

- **Hires subdirectory layout should be frozen explicitly.** The current text says hires
  aggregates go under `01-stage-medians/` in `hires-{bin}/` subdirs, but a concrete mini-tree
  should be added so later spec writing does not drift.

- **APS input source for per-stage is not yet explicit.** If Priority 13.5 computes APS, the
  brainstorm should say whether APS reads from `02-chrom-norm/` base-resolution tracks,
  `01-stage-medians/`, or any hires-derived surface. My recommendation is base-resolution
  normalized tracks from `02-chrom-norm/`; summit refinement is about position, not amplitude.

- **Gap-analysis placement is still unresolved.** The current text leaves open whether gap
  analysis should append to `08-multistage-unification/` or become its own later step. Under the
  user's atomization rule, if gap analysis writes new outputs, it should probably be its own step.
  This likely argues for a later `09-gap-analysis/` and for plots to become `10-plots/` if plots
  are promoted to first-class outputs.

- **Plot output placement is under-specified.** The brainstorm adds profile plots, APS plots,
  summit-fit plots, and genome-overview plots, but does not decide whether plots should be a
  dedicated step or live inside producer steps. Under the user's "new process = new step" rule,
  a dedicated plots step is more consistent.

- **Posterior output placement for per-stage should be stated explicitly.** If Priority 13.7
  lands, the natural parallel is `02-posterior/03-per-stage-mean-shift/` with the same internal
  step layout. This should be stated early so the spec can stay parallel with growth and HMM.

### Additional Priority candidates / refinements

- **Priority candidate: per-stage layout contract hardening.** The brainstorm mentions test
  updates, but I would explicitly call out a narrow invariant test pass analogous to the Phase 12
  grouped-layout hardening work. This is not a separate large priority, but it should be a named
  deliverable inside the structural layout priority so the new per-stage step contract is locked
  in immediately.

- **Priority candidate: per-stage docs and generated-guidance sync.** Beyond `_INDEX_MD_TEMPLATE`,
  Phase 13 should explicitly audit `README.md`, generated notebook guidance, and any output-guide
  text for stale flat per-stage assumptions. This is in the spirit of "do not break anything or
  make anything stale" and is broader than only updating tests.

- **Priority candidate: per-stage inspector-readiness, not necessarily full inspector support.**
  `scripts/summit_inspector.py` does not yet meaningfully support per-stage overlays, but Phase 13
  creates the structural prerequisite by making `08-multistage-unification/` and other step paths
  predictable. Even if full support is deferred, the placeholder/comment path knowledge should be
  updated in Phase 13 so the inspector is not left stale.

### Risks / sequencing feedback

- **The brainstorm may still be slightly overscoped if all expansion-ii items are treated as the
  same class of work as the structural re-layout.** The structural 8-step refactor plus HMM
  harmonization already form a substantial phase. APS/clustering/posterior and the summit QC work
  are each big enough to deserve explicit dependency language and likely should not begin until the
  step layout is stable and tested.

- **The clearest critical path is:** structural 8-step layout first, then HMM naming/template
  harmonization, then summit-refinement diagnostic work, then gap-analysis/metrics/plots, then
  APS+clustering, and only then posterior. The current brainstorm is mostly aligned with this, but
  the dependencies should be made even more explicit in the eventual spec.

- **Posterior blocker audit should be an explicit audit checkpoint before promising Phase 13
  posterior delivery.** The current blocker in `_run_standalone_per_stage_controller()` may be
  stale, but that needs verification rather than assumption.

### Backlog items that harmonize with Phase 13

- From `multi-agent/BRAINSTORM.md`, the older "per-stage completeness" direction is clearly part
  of this phase's spirit and can be treated as conceptually absorbed into Phase 13.

- From `multi-agent/KNOWN_ISSUES.md`, two items look especially relevant once per-stage APS work
  starts:
  - `ISSUE:2026-04-14:1` stage-1 pre-amplification isolation
  - `ISSUE:2026-04-18:1` RCN floor removal experiment in `_locus_metrics()`

  These are not core Phase 13 structural blockers, but they are very much in the same spirit as
  the per-stage APS/clustering and QC priorities and should at least be mentioned as audit inputs
  when the spec is written.

### Concrete add-on bullets to consider integrating

- Add a concrete dedup-routing example showing pre-dedup TSV/BED in `03-detection/` and post-
  dedup BED + dedup report TSV in `05-deduplication/`.

- Add a concrete `01-stage-medians/` tree showing base tracks at top level and hires tracks in
  `hires-{bin}/` subdirectories.

- Add an explicit note that if APS is implemented for per-stage, the default input should be the
  normalized aggregate tracks from `02-chrom-norm/` unless a later audit proves hires or another
  surface is more appropriate.

- Add an explicit spec-authoring checkpoint to audit whether HMM currently enforces the same APS /
  clustering gating rules being proposed for per-stage.

- Add an explicit Phase 13 deliverable to update any user-facing/generated guidance that still
  assumes flat per-stage outputs.

- Consider making plots a dedicated later step if Phase 13 really follows the atomization rule to
  its logical conclusion.

### Questions for John M. Urban

1. For the dedup visibility design, do you want the post-dedup canonical call TSV in
   `05-deduplication/` too, or only the BED plus report there?
2. For gap analysis and plots, do you want to keep the 8-step layout fixed for Phase 13 core and
   add those as later steps only when implemented, or should the eventual larger step count be
   planned now even if some directories stay empty until later?
3. For per-stage APS, are you comfortable treating `02-chrom-norm/` as the authoritative numeric
   input surface by default?
4. Do you want Phase 13 to explicitly include doc/guidance/inspector stale-path cleanup as a named
   deliverable, not just an implicit expectation?

### Sign-off

Audit completed by GitHub Copilot (GPT-5.4), based on live-code inspection plus review of
`BRAINSTORM.md` and `KNOWN_ISSUES.md`. Main conclusion: the current Phase 13 brainstorm is strong,
but it would benefit from a few more explicit file-routing examples, dependency checkpoints, and a
clearer decision on whether plots and gap analysis become first-class later steps.

---

## Feedback for Copilot from John M. Urban 

1. All canonical things as well as BED.
2. Good catch. The eventual larger step count should be planned now even if some directories stay empty until later.
3. I don't think treating `02-chrom-norm/` as the authoritative numeric input surface by default makes sense for APS, which is a "per sample" analysis. Each sample needs to be median normalized and processed as if a "single file" - then those can be inputs to APS. The individual samples can be processed in `02-chrom-norm/indiv_samples` or some similar subdirectory name. 
4. Yes! Thanks for asking. I do want Phase 13 to explicitly include doc/guidance/inspector stale-path cleanup as a named deliverable, not just an implicit expectation.

#### Other Feedback
- The way you phrased question 2 made me realize that "per-stage" is not the right name for this pipeline anymore because for all pipeline we talk in terms of "per stage" and "per sample". So the term "per-stage" is overloaded. Please generate new candidate names for this pipeline that are focused on the dominant feature. For example, a candidate might be `rcn-mean-shift`. A name like that would differentiate it a bit more and make it more obvious what it does similar to "hmm" for the HMM pipeline.

## Copilot Addendum 2 — targeted follow-up after John's feedback

### Resolved from John's answers

- **Dedup canonical outputs:** `05-deduplication/` should hold the full canonical post-dedup call
  set, not just BED plus report. That means the spec should plan for canonical TSV as well as BED
  in `05-deduplication/`, alongside the lightweight dedup report. This keeps the user-facing
  canonical call table aligned with the canonical interval file.

- **Plan the larger step count now:** even if some later directories remain empty in early Phase 13
  implementation rounds, the eventual larger pipeline shape should be planned explicitly now. This
  argues against treating gap analysis, plots, APS/clustering, or other later deliverables as
  vague appendices to the 8-step structure. If they are real downstream processes, they should be
  anticipated as later numbered steps in the eventual pipeline family layout.

- **APS should be fed by per-sample normalized surfaces, not only stage-level tracks:** this is a
  strong correction. APS is fundamentally sample-level, so the brainstorm/spec should stop framing
  `02-chrom-norm/` stage-level outputs as the likely default APS input. Instead, Phase 13 should
  plan for per-sample normalized outputs under something like `02-chrom-norm/indiv_samples/` so
  APS, clustering, posterior grouping, and posterior manifest building can use properly aligned
  sample-level inputs.

- **Doc/guidance/inspector cleanup should be a named deliverable:** agreed and now explicit. This
  should be treated as a real Priority-level deliverable within the structural layout work, not an
  implied side effect.

### Revised structural implication from the APS correction

John's APS clarification implies that `02-chrom-norm/` may need to own **two related but distinct
normalized surfaces**:

1. **Stage-level normalized tracks** used by the main mean-shift calling continuum.
2. **Per-sample normalized tracks** under an `indiv_samples/` subdirectory used for APS /
   clustering / posterior-family work.

That is worth stating explicitly because it changes the meaning of step 02 from "normalize the
aggregate track" to "normalize the aggregate track and also emit the sample-level normalized
surface needed by downstream sample-level analyses." This feels architecturally coherent and also
brings this pipeline closer to the rest of the family, where later analyses often depend on a
child/per-sample surface rather than only the stage-level summary surface.

### Naming direction: replacing the overloaded "per-stage" pipeline label

I agree that "per-stage" is now too overloaded as a pipeline name. It describes one aspect of the
data organization, but it does not name the dominant analysis identity. A better pipeline name
should point at the defining method or signal logic, the same way `hmm` does.

#### Strong candidate names

1. **`rcn-mean-shift`**
   - Best balance of precision and clarity.
   - Names both the signal surface (`RCN`) and the dominant caller (`mean-shift`).
   - Distinguishes this pipeline from both `hmm` and `growth` without relying on stage/sample
     wording.

2. **`mean-shift-rcn`**
   - Same ingredients, slightly more method-forward than signal-forward.
   - A little less natural than `rcn-mean-shift`, but still strong.

3. **`rcn-calling`**
   - Cleaner and broader, but less distinctive because it hides the specific mean-shift identity.
   - Better if you want a more future-proof umbrella name that could absorb non-mean-shift
     internals later.

4. **`broad-rcn`**
   - Emphasizes the broad-domain nature of the caller.
   - More descriptive of biological scale than algorithm, but also more ambiguous.

5. **`domain-rcn`**
   - Signals broad amplicon/domain detection using RCN.
   - Reasonable if the user-facing identity should emphasize domains rather than the algorithm.

#### Weaker candidates

- **`stage-mean-shift`**: still keeps the overloaded stage terminology in the pipeline name.
- **`multistage-mean-shift`**: too input-configuration-specific; weak for single-stage or
  single-file runs.
- **`aggregate-mean-shift`**: overemphasizes the aggregation surface rather than the analysis.
- **`independent-mean-shift`**: too vague; does not tell the user what signal is being analyzed.

#### Recommendation

My recommendation is to treat **`rcn-mean-shift`** as the leading candidate. It is specific,
compact, method-identifying, and works across single-file, single-stage, and multi-stage inputs.
It also makes future path naming more legible, for example:

- `03-rcn-mean-shift/`
- `02-posterior/03-rcn-mean-shift/`

If you want a slightly broader umbrella that could survive algorithmic changes later,
`rcn-calling` is the main alternative.

### Additional spec consequences to consider

- If the pipeline is renamed, the Phase 13 spec should explicitly decide whether this phase does
  only the **internal step-layout refactor** or also includes the **public pipeline renaming** of
  directory labels, CLI vocabulary, docs, templates, tests, and inspector assumptions.

- If renaming is included now, there should be a deliberate migration pass for all user-facing
  strings still saying `per-stage`, reserving that phrase for descriptive context only rather than
  as the pipeline's proper name.

- If renaming is deferred, the brainstorm/spec should still note the naming decision path now, so
  Phase 13 structural work does not harden a label you already think is semantically wrong.

### Suggested refinement to the eventual step plan

Because John wants the larger step count planned now, I would suggest the future-facing plan start
thinking in two layers:

1. **Core implemented-now steps** for the immediate structural refactor.
2. **Reserved later numbered steps** for outputs already expected to become first-class in this
   pipeline family, such as gap analysis, plots, APS, clustering, and posterior-related surfaces.

That gives the spec room to stay honest about what is implemented now while still preventing the
layout from having to be renumbered again as the pipeline gets meatier.

### Sign-off

Follow-up addendum completed by GitHub Copilot (GPT-5.4) in direct response to John's feedback.
Main conclusion: the APS clarification materially changes the downstream architecture, and
`rcn-mean-shift` is the strongest current candidate for replacing the overloaded `per-stage`
pipeline name.

---

## Feedback regarding Copilot's Second Addendum from John M. Urban 
- `rcn-mean-shift` is the clear winner so far.
- `rcn-mean-shift` (or other new name) should be renamed internally and public facing

---

## Gemini Addendum

Based on a deep-dive audit of the codebase, historical planning documents (`HANDOFF.md`, `PHASE11_SPEC.md`, `PHASE12_SPEC.md`), and the current Phase 13 brainstorm, I have synthesized several high-impact priorities that harmonize with the goal of making the `rcn-mean-shift` (formerly `per-stage`) pipeline a first-class citizen of the onionskin family.

### 1. Structural Alignment: Renaming & Reordering
The "per-stage" nomenclature is indeed overloaded. I support the transition to **`rcn-mean-shift`** as the canonical name for this pipeline. Furthermore, my audit of `HANDOFF.md` (v0.12.02) and `output_layout.py` reveals a discrepancy: the target grouped order is explicitly intended to be **HMM → RCN-Mean-Shift → Growth**, but `output_layout.py` currently places growth at `02-` and per-stage at `03-`.

**Proposed Action:**
- **Rename internally and externally**: Change `--pipelines per-stage` to `--pipelines rcn-mean-shift` (maintaining a legacy alias for a grace period).
- **Correct the Grouped Order**: Swap the numbering in `output_layout.py` and `onionskin.py` to:
    - `01-hmm/`
    - `02-rcn-mean-shift/`
    - `03-growth-model/`
- **Controller Reorder**: Update `onionskin.py` to invoke the `rcn-mean-shift` controller *before* the growth model.

### 2. The Atomic 13-Step Layout
To follow the user's mandate of "atomize steps" and "do it the same," I propose the following full 13-step layout for `02-rcn-mean-shift/`. This includes "reserved" steps for the meatier features planned for this phase.

| Step | Directory | Content / Purpose |
|------|-----------|-------------------|
| 01 | `01-stage-medians/` | Per-stage aggregate medians (base + `hires-{bin}/` subdirs). |
| 02 | `02-chrom-norm/` | Stage-level normalized tracks + `indiv_samples/` (per-sample normalized tracks for APS). |
| 03 | `03-detection/` | Raw (pre-dedup) mean-shift calls (TSV + BED). |
| 04 | `04-summits/` | Per-stage raw peak positions (extracted from detections). |
| 05 | `05-deduplication/` | Dedup report + post-dedup canonical calls (TSV + BED). |
| 06 | `06-shape-filter/` | Shape scores, collapsed repeats, and rejected BEDs. |
| 07 | `07-summit-refinement/` | **Parabola fit refinement** (Step-7) + hires refinement (Step-13 analog). |
| 08 | `08-multistage-unification/` | Cross-stage unified calls and unified repeats. |
| 09 | `09-gap-analysis/` | Amplicon gap analysis and recommendation BEDs. |
| 10 | `10-plots/` | Consolidated plots (Profile, APS, Summit-fit, Genome-overview). |
| 11 | `11-aps/` | APS matrices and metrics (Area, Summit, Width). |
| 12 | `12-clustering/` | Sample clustering and ordering based on APS. |
| 13 | `13-posterior/` | Posterior manifest generation and grouping (if 2+ stages + 3-4+ samples). |

### 3. HMM Prettification & Family Harmonization
To further the "pipeline family" goal, I recommend these specific renames in the HMM engine and `output_layout.py`:
- **`02-unionStats` → `02-stage-medians`**: Harmonizes with the "aggregate of replicates" concept.
- **`04-chromMedRatioNorm-RCN` → `04-chrom-renorm`**: Accurate and generic.
- **Internal File Renames**: Change `*_unionBedGraphStats.txt` to `*_positional-bin-stats.txt` (or similar) to remove the "union" vestige.

### 4. Expansion ii: Technical Implementation Details
- **Parabola Summit Estimator**: My audit shows `per_stage_mean_shift_engine.py` is currently under-utilizing the hybrid hierarchical parabola fit in `refinement.py`. Wiring this to `07-summit-refinement/` is the direct fix for the II/9A "right-shift" artifact noted by the user.
- **Per-Sample Normalization**: I propose adding a dedicated pass in Step 02 (`02-chrom-norm/indiv_samples/`) using `aps.compute_sample_rcn_tracks()`. This ensures APS and clustering have the correct per-sample inputs without "backwards cramming" or borrowing from Growth.
- **Unified Controller**: Unify `run_single_stage()` and `run_per_stage_mean_shift()` into a single `run_rcn_mean_shift()` engine that handles any number of stages (1 to N) using the same step-routing logic.

### 5. Tooling & Test Integrity
Renaming and reordering are high-impact changes that will break hardcoded assumptions.
- **`scripts/summit_inspector.py`**: Must update `PIPELINE_ORDER`, `PIPELINE_LABELS`, and `PIPELINE_DIR_NAMES`.
- **`tests/test_pipeline.py`**: Comprehensive update to all path assertions (especially replacing `03-per-stage-mean-shift` with `02-rcn-mean-shift`).
- **`Makefile`**: Update targets like `make test-single` or others that reference the old directory name.

### Sign-off
Audit and brainstorm expansion completed by Gemini CLI. The proposed 13-step layout and structural reordering (01-HMM, 02-RCN-MS, 03-Growth) align exactly with the "retained target" stated in the handoff history while fulfilling the user's "atomize" and "meatier" mandates.

---

## Feedback regarding Gemini's Addendum from John M. Urban 
- In Gemini's CHANGELOG, I saw this:
```
 - **Structural Reordering:** Identified a discrepancy between target intent and current code; proposed corrective swap of directory numbering to `01-hmm/`, `02-rcn-mean-shift/, and `03-growth-model/` to match execution order and historical handoff mandates.
```
- Since Phase 12, we agreed on: `01-hmm`, `02-growth-model`, `03-per-stage` (or now `03-rcn-mean-shift`).
    - It might be that there are still stale references to the other way in the code or other active files that still need to be purged.
    - If it is only historical discrepancies in things like audits and changelog (etc), then we can either ignore or leave "update notes" near them so future agents do not raise red flags.


---

## Claude Triage (Round 3 — after Copilot/Gemini addenda)

### Resolved: Pipeline rename — `per-stage` → `rcn-mean-shift`

John confirmed: **`rcn-mean-shift` is the winner.** Rename is internal and public-facing.

What this touches:
- **Directory label**: `03-per-stage-mean-shift/` → `03-rcn-mean-shift/`
- **CLI token**: `--pipelines per-stage` → `--pipelines rcn-mean-shift`
  (add a grace-period alias `per-stage` that maps to `rcn-mean-shift` with a deprecation warning)
- **Engine public API**: `run_per_stage_mean_shift()` + `run_single_stage()` → `run_rcn_mean_shift()` (unified; see below)
- **Engine file**: `onionskin_core/per_stage_mean_shift_engine.py` → `onionskin_core/rcn_mean_shift_engine.py`
- **`output_layout.py`**: `build_per_stage_steps()` → `build_rcn_mean_shift_steps()`; layout dict key `per_stage_mean_shift_dir` → `rcn_mean_shift_dir`; `ensure_layout_dirs` param `include_per_stage` → `include_rcn_mean_shift`
- **`onionskin.py`**: controller renamed `_run_standalone_per_stage_controller` → `_run_rcn_mean_shift_controller`; all `per_stage` variable names updated; `_pipelines_include(args, "per-stage")` → `"rcn-mean-shift"`
- **Log tags**: `[per-stage]` → `[rcn-mean-shift]` throughout
- **`scripts/summit_inspector.py`**: `PIPELINE_ORDER`, `PIPELINE_LABELS`, `PIPELINE_DIR_NAMES` all updated
- **`tests/test_pipeline.py`**: all `03-per-stage-mean-shift` path assertions → `03-rcn-mean-shift`
- **`README.md`, `PIPELINE_SPEC.md`, `_INDEX_MD_TEMPLATE`**: all `per-stage` references updated

The rename is mechanical but broad. It is the first task of Priority 13.1 so all subsequent work uses the new name from the start.

---

### Resolved: Gemini's reorder finding — FALSE POSITIVE

Gemini proposed swapping directories to `01-hmm/`, `02-rcn-mean-shift/`, `03-growth-model/` citing a discrepancy with HANDOFF history. **This reorder is incorrect.** The Phase 12 decision (Priority 12.3) was explicitly: **HMM → Growth → Per-Stage** = `01-hmm/`, `02-growth-model/`, `03-per-stage-mean-shift/`. The current code in `output_layout.py` already reflects this correctly:
- `hmm_dir = 01-hmm/` ✓
- `growth_model_dir = 02-growth-model/` ✓
- `per_stage_mean_shift_dir = 03-per-stage-mean-shift/` ✓

What Gemini found is stale pre-Phase-12 text in historical HANDOFF entries — not a code defect. No directory swap is needed. The doc-cleanup Priority (see below) should add "update notes" near those historical stale references, per John's guidance.

---

### Resolved: Full 13-step layout (planning the final shape now)

John confirmed: plan the full step count now even if some directories are initially empty. Gemini's 13-step proposal aligns with this. Final layout under `03-rcn-mean-shift/`:

| Step | Dir | Contents | Status |
|------|-----|----------|--------|
| 01 | `01-stage-medians/` | Per-stage aggregate medians (base + `hires-{bin}/` subdirs) | Priority 13.1 |
| 02 | `02-chrom-norm/` | Stage-level normalized tracks + `indiv_samples/` (per-sample normalized) | P13.1 (stage) + P13.5 (indiv) |
| 03 | `03-detection/` | Pre-dedup calls TSV + BED | Priority 13.1 |
| 04 | `04-summits/` | Per-stage raw summit BEDs | Priority 13.1 |
| 05 | `05-deduplication/` | Dedup report TSV + post-dedup canonical calls TSV + BED | Priority 13.1 |
| 06 | `06-shape-filter/` | Shape scores TSV, collapsed repeats TSV+BED, rejected BED | Priority 13.1 |
| 07 | `07-summit-refinement/` | Parabola-fit refined summits + hires refinement | Priority 13.3 |
| 08 | `08-multistage-unification/` | Unified cross-stage calls TSV + BED, unified collapsed repeats | Priority 13.1 |
| 09 | `09-gap-analysis/` | Gap-annotated calls, gap analysis TSV, recommendation BEDs | Priority 13.4 |
| 10 | `10-plots/` | All plots (profile, APS, summit-fit, genome-overview) | Priority 13.6 |
| 11 | `11-aps/` | APS matrices, per-locus APS scores | Priority 13.5 |
| 12 | `12-clustering/` | Cluster assignments, ordering tables | Priority 13.5 |
| 13 | `13-posterior/` | Posterior manifest, group assignments | Priority 13.8 |

Steps 09–13 will be empty directories until their implementing Priority runs. This matches how HMM handles sparse runs (some step dirs are always created but may be minimal).

---

### Resolved: Unified controller (`run_rcn_mean_shift()`)

Gemini proposes — and John's "continuum" direction confirms — merging `run_single_stage()` and `run_per_stage_mean_shift()` into a single `run_rcn_mean_shift()` that handles 1-to-N stages with the same 13-step routing logic. The distinction between the two current functions is an implementation artifact; the user-facing model is a single pipeline that happens to have more or fewer stages.

This is part of Priority 13.1. The merged function signature should accept `n_stages` and route accordingly, with gating rules (e.g., `08-multistage-unification/` only populated when N≥2 stages) handled internally.

---

### Resolved: `02-chrom-norm/indiv_samples/` — per-sample normalized tracks for APS

John's correction (via Copilot Q3): APS is a per-sample analysis. Stage-level tracks from `02-chrom-norm/` are NOT the right input. Per-sample normalized tracks must be computed and emitted to `02-chrom-norm/indiv_samples/`.

Step 02 output structure:
```
02-chrom-norm/
  {prefix}_stage1_chrom_norm.bedGraph          ← stage-level (for detection calling)
  {prefix}_stage2_chrom_norm.bedGraph
  indiv_samples/
    {prefix}_{sample_id}_stage1_chrom_norm.bedGraph  ← per-sample (for APS)
    {prefix}_{sample_id}_stage2_chrom_norm.bedGraph
```

The per-sample tracks require applying `_chrom_median_norm()` (or equivalent) to each replicate's track before aggregation, rather than only to the aggregate. This is new computation not currently in the engine — must be specced carefully in Priority 13.5.

---

### Resolved: Dedup final canonical output (Copilot Q1)

`05-deduplication/` holds **all** canonical post-dedup outputs:
```
05-deduplication/
  {prefix}_stage{st}_dedup_report.tsv    ← lightweight report (n_before, n_after, n_removed)
  {prefix}_stage{st}_calls.tsv           ← post-dedup canonical calls TSV
  {prefix}_stage{st}_calls.bed           ← post-dedup canonical calls BED
```

`03-detection/` holds pre-dedup:
```
03-detection/
  {prefix}_stage{st}_calls.tsv           ← pre-dedup raw calls TSV
  {prefix}_stage{st}_calls.bed           ← pre-dedup raw calls BED
```

---

### Resolved: Doc/guidance/inspector cleanup — explicit named Priority

John confirmed this should be an explicit named deliverable, not an implied side effect of the rename. Scope: `README.md`, `PIPELINE_SPEC.md`, generated notebook guidance, `_INDEX_MD_TEMPLATE`, `summit_inspector.py` stale path comments, and any historical HANDOFF/BRAINSTORM/KNOWN_ISSUES references that use `per-stage` as a pipeline label. Also: "update notes" near stale pre-Phase-12 ordering references (Gemini false positive — these are historical artifacts, not bugs, but should be annotated so future agents don't re-flag them).

---

### Resolved: Amplicon metrics (Priority 13.4 scope)

Copilot and Gemini both note the gap-analysis Priority. The amplicon metrics step (width, amplitude, shape score summary per call) is a natural companion. Confirmed as part of Priority 13.4 scope alongside gap analysis. Does NOT get its own step directory — amplicon metrics output lives in the calling step (`05-deduplication/` or `06-shape-filter/`) depending on where the canonical post-filter calls reside. *Open for spec:* whether a separate `09b-amplicon-metrics/` or `10b-amplicon-metrics/` is needed, or whether amplicon metrics TSV lives in `08-multistage-unification/` alongside unified calls.

---

### Resolved: HMM APS/clustering guard audit — deferred to spec

The user asked: does HMM currently enforce the same rule (APS for ≥1, clustering for 2+ stages + 3-4+ samples)? This requires reading `hmm_engine.py` and `hmm_ported_analyses.py`. Not resolvable from brainstorm context. Flagged as a **mandatory spec audit item** before Priority 13.5 can be written.

---

### Updated Phase 13 scope — final (supersedes all previous summaries)

**Priority 13.1 — `rcn-mean-shift` rename + 13-step layout + unified controller**
- Rename `per-stage` → `rcn-mean-shift` everywhere (CLI, dirs, engine, layout, tests, inspector, docs)
- `03-per-stage-mean-shift/` → `03-rcn-mean-shift/`
- `run_per_stage_mean_shift()` + `run_single_stage()` → `run_rcn_mean_shift()` (unified)
- `build_rcn_mean_shift_steps()` with all 13 dirs in `output_layout.py`
- Pre-dedup calls in `03-detection/`; dedup report + canonical calls (TSV+BED) in `05-deduplication/`
- `_INDEX_MD_TEMPLATE` updated (13-row block for rcn-mean-shift)
- `tests/test_pipeline.py` updated; test invariant pass for new step contract
- `summit_inspector.py` stale path update

**Priority 13.2 — HMM prettification + name harmonization**
- `02-unionStats` → `02-stage-medians` in `build_hmm_steps()`
- `04-chromMedRatioNorm-RCN` → `04-chrom-renorm` in `build_hmm_steps()`
- `*_unionBedGraphStats.txt` → `*_positional-bin-stats.txt` in HMM engine
- `_INDEX_MD_TEMPLATE` HMM section updated; vocabulary audit across all three pipelines
- `tests/test_pipeline.py` HMM step assertions updated

**Priority 13.3 — Summit refinement (parabola estimator) + II/9A diagnostic**
- Wire parabola estimator to `07-summit-refinement/` in `rcn_mean_shift_engine.py`
- Summit evaluation at II/9A (right-shift root cause) and II/2B
- Compare against growth pipeline + HMM summit estimates at same loci

**Priority 13.4 — Gap analysis + amplicon metrics**
- Gap analysis for unified calls → `09-gap-analysis/`
- Amplicon metrics (width, amplitude, shape score) — output placement to be decided in spec

**Priority 13.5 — Per-sample normalization + APS + clustering**
- Per-sample chrom-norm pass → `02-chrom-norm/indiv_samples/`
- APS for ≥1 non-reference stages → `11-aps/`
- Clustering for 2+ stages + 3-4+ samples → `12-clustering/`
- APS plots; summit-fit plots; genome-overview plots → `10-plots/`
- Audit HMM APS/clustering gating rules; harmonize if different

**Priority 13.6 — Plots step (`10-plots/`)**
- Profile plots per stage (aggregate track + calls)
- APS plots (if 13.5 complete), summit-fit plots, genome-overview plots
- May overlap with 13.5 implementation; keep separate Priority for clarity

**Priority 13.7 — Doc/guidance/inspector cleanup**
- `README.md`, `PIPELINE_SPEC.md`, generated notebook guidance: all `per-stage` → `rcn-mean-shift`
- `summit_inspector.py`: full stale-path/label update (PIPELINE_ORDER, labels, dir names)
- Historical stale-ordering references in HANDOFF/docs: annotate with "update note" (not errors)
- Verify no user-facing guidance still assumes flat per-stage outputs

**Priority 13.8 — Per-stage QC / parameter tuning**
- Overlap rcn-mean-shift calls against `amplicons.by-eye.bed` and PuffStep HMM BED (chr II)
- Summit evaluations at II/9A and II/2B (complements Priority 13.3 diagnostics)
- Report: too few / too many calls? Parameter adjustment recommendation if warranted

**Priority 13.9 — Posterior groups, ordering, manifests**
- Requires Priority 13.5 (APS + clustering) as prerequisite
- Audit blocker in `_run_rcn_mean_shift_controller` — verify if technically stale
- Posterior manifest + grouping → `13-posterior/`
- Same 2+ stages + 3-4+ samples gating as clustering
- Wire `--posterior` support for rcn-mean-shift
- Verify HMM posterior follows same gating rule


---

## Feedback regarding Claude's Final Triage (and discussion with Claude) from John M. Urban 

#### RE: One open question
Claude asked:
```
One open question I'd like your call on before spec authoring:

The amplicon metrics output location is unresolved — does it get its own step dir (e.g., 10b-amplicon-metrics/ between gap analysis and plots), or does it live inside an existing step? Under the atomization rule it should probably be its own step, which would push plots to 11-plots/ and APS/clustering to 12-aps//13-clustering/. That makes the layout 14+ steps. Or we can keep amplicon metrics as an output inside 08-multistage-unification/ and keep 13 steps. Your call.
```
- Agreed on atomization: it should be its own step. But I dont like "10b". We need to give it a regular integer and bump others up by +1 as needed.

#### RE: One thing I want to confirm before writing the spec
Claude said:
```
Priority 13.7 (doc/guidance cleanup) will partially overlap with 13.1 (rename), since the rename itself touches README, PIPELINE_SPEC, etc. Should 13.7 be a cleanup sweep that runs after the rename lands, catching anything 13.1 missed — or should it be merged into 13.1 as one big rename+cleanup Priority?
```
- Claude is in charge of designing the first pass of SPEC and finding the most sensible order of operations. I yield to Claude's engineering decisions on this. If you want me to give you my leaning, I am thinking that if it makes sense to merge into 13.1, and it would be trivial to do so, then merge it in. The only reason to keep it as 13.7 would be if it had a dependency it needed to wait on before being implemented, which does not sound like the case here.

#### Do we need to anticipate hires individual samples?
- In `01-stage-medians/` we are making `hires-{bin}/` subdirectories.
- In `02-chrom-norm/` we are making `indiv_samples/` subdirectories.
- Will we need `02-chrom-norm/indiv_samples/hires-{bin}/` directories as well?
- Will individual hires samples be needed anywhere in the pipelin?

### Sign Off: John M. Urban
- This Brainstorm phase is done.
- Claude is permitted to make the first pass at PHASE13_SPEC.md
- Then Copilot audits the Spec
- Then Claude responds to and implements based on Copilot’s audit
- Finally Copilot gets another chance to close out the SPEC engineering phase
- Then we will begin "audit / implement" volleying as defined in `multi-agent/workflows/spec_plan_three_role_audit_loop.md`

---

## Claude — Final Sign-off Responses (for spec authoring record)

### Amplicon metrics step number
Amplicon metrics gets its own regular integer step, bumping all subsequent steps +1.
**Final 14-step layout under `03-rcn-mean-shift/`:**

| Step | Dir |
|------|-----|
| 01 | `01-stage-medians/` |
| 02 | `02-chrom-norm/` (+ `indiv_samples/`) |
| 03 | `03-detection/` |
| 04 | `04-summits/` |
| 05 | `05-deduplication/` |
| 06 | `06-shape-filter/` |
| 07 | `07-summit-refinement/` |
| 08 | `08-multistage-unification/` |
| 09 | `09-gap-analysis/` |
| 10 | `10-amplicon-metrics/` |
| 11 | `11-plots/` |
| 12 | `12-aps/` |
| 13 | `13-clustering/` |
| 14 | `14-posterior/` |

### Doc/guidance cleanup — merged into Priority 13.1
Merging into Priority 13.1 is the right call. The rename already touches README,
PIPELINE_SPEC, INDEX_MD_TEMPLATE, and inspector. Keeping a separate Priority 13.7 only
makes sense if there is a dependency — there is none. Priority 13.1 scope now includes:
rename + 14-step layout + unified controller + doc/guidance/inspector sweep.

### Hires individual samples — NOT anticipated
`02-chrom-norm/indiv_samples/hires-{bin}/` directories are **not needed**.

Reasoning:
- Hires tracks exist for summit position refinement only. The hires refinement loop
  (`07-summit-refinement/`) works on the **aggregate** hires track (already in
  `01-stage-medians/hires-{bin}/`), not per-sample tracks.
- APS computes scalar metrics (Area Excess, Summit, Width) at called loci by querying
  per-sample signal at those positions. This query works fine at **base resolution** —
  the per-sample bedGraphs in `02-chrom-norm/indiv_samples/` are base-resolution and
  sufficient for APS amplitude extraction.
- There is no downstream analysis that requires per-sample hires coverage. The parabola
  summit fit uses the aggregate hires signal; APS uses per-sample base-resolution signal.

**Conclusion:** `01-stage-medians/hires-{bin}/` (aggregate hires) ✓ needed.
`02-chrom-norm/indiv_samples/` (per-sample base) ✓ needed.
`02-chrom-norm/indiv_samples/hires-{bin}/` ✗ not needed — do not create.

---

## Copilot Audit of `PHASE13_SPEC.md`

### Verdict

**Not quite ready to archive yet.** The spec is close and the overall Priority ordering is mostly
sound, but I found a small number of material issues that should be resolved in the spec before
implementation begins.

### What looks good

- The core dependency order is mostly sensible: structural refactor first, then HMM harmonization,
  then summit refinement, then downstream analytics, then posterior.
- The rename to `rcn-mean-shift` is carried through consistently at the planning level.
- The 14-step layout incorporates John's later feedback correctly: reserved later steps are planned
  now, `10-amplicon-metrics/` is its own integer step, and `02-chrom-norm/indiv_samples/hires-{bin}/`
  is correctly excluded.
- Merging doc/guidance/inspector cleanup into Priority 13.1 is a good engineering decision.

### Findings

#### FIND-S1 — Posterior placement is inconsistent with pipeline-family logic

The spec currently places posterior output inside the main prior pipeline tree as
`03-rcn-mean-shift/14-posterior/`. That is the largest remaining issue.

Why this is a problem:
- The project's existing family pattern is that posterior products live under the top-level
  `02-posterior/` grouping, not as a late step inside the prior pipeline tree.
- Priority 13.8 itself says `--posterior` should run "following the same pattern as the growth
  pipeline's `_run_posterior()`". That points toward `02-posterior/03-rcn-mean-shift/`, not a
  `14-posterior/` directory inside `03-rcn-mean-shift/`.
- The brainstorm/addenda explicitly raised this as the natural parallel and asked for it to be
  stated clearly. The current spec instead mixes two different concepts: prior step layout vs
  posterior rerun ownership.

Recommended repair:
- Move posterior outputs out of the `03-rcn-mean-shift/` step tree and define their home under the
  top-level posterior grouping, parallel to the other pipelines.
- If you still want a numbered internal step sequence for the posterior rerun itself, define that
  inside the posterior pipeline's own layout rather than inside prior `03-rcn-mean-shift/`.

Severity: **High**

#### FIND-S2 — Priority 13.5 still underspecifies the per-sample APS input contract

The spec improved this substantially by adding `02-chrom-norm/indiv_samples/`, but it still says:
"apply `_chrom_median_norm()` (or equivalent) to the individual sample's bedGraph track." That is
not yet precise enough relative to John's guidance that each sample should be median-normalized and
processed as if a single file.

Why this matters:
- "Or equivalent" leaves room for an implementation that only chrom-normalizes the raw per-sample
  track and stops there.
- John's correction suggests a fuller per-sample preprocessing contract may be intended before APS
  consumes those tracks.
- If the spec does not pin this down now, the implementation can drift and later audits will have
  to reopen what "single-file-like" processing was supposed to mean.

Recommended repair:
- In Priority 13.5, explicitly define the preprocessing contract for `indiv_samples/` inputs.
- Either name the exact helper / pipeline surface to reuse, or state clearly that the intended
  contract is only chrom-median normalization and not any additional single-file transforms.
- Remove the phrase "or equivalent" unless the equivalence class is spelled out.

Severity: **Medium**

#### FIND-S3 — Priority 13.4 is internally inconsistent about what step 10 metrics describe

Priority 13.4 says gap analysis and amplicon metrics are both downstream of unified calls in
`08-multistage-unification/`, but the implementation scope then says amplicon metrics should be
computed after shape filtering (step 06) from per-stage call data already in memory.

Why this matters:
- These are two different metric identities: per-stage post-filter call metrics vs unified
  cross-stage amplicon metrics.
- The output filename `{prefix}_amplicon_metrics.tsv` and the suggested `stage` column imply a
  per-stage table, while the step numbering after unification implies a downstream unified product.
- John explicitly chose to make amplicon metrics its own step; that step should have a clear data
  contract before implementation starts.

Recommended repair:
- Decide whether `10-amplicon-metrics/` is fed from post-filter per-stage calls or from unified
  multistage calls.
- Make the overview, target state, implementation scope, and TSV schema all agree.
- If both views are wanted, say so explicitly and name the two outputs separately.

Severity: **Medium**

### Ordering assessment

The Priority order is otherwise reasonable. I do **not** see a major dependency inversion in the
current 13.1 → 13.8 sequence. The only substantial architectural problem is the posterior home,
which is more about output-model consistency than numerical ordering.

One minor note: placing plots after APS/clustering is conservative but acceptable. It means all
possible plot dependencies are satisfied before Priority 13.6 starts, even though some plot types
could technically land earlier.

### Readiness assessment

My assessment is:

- **Close, but not ready to archive yet.**
- If FIND-S1 through FIND-S3 are repaired in `PHASE13_SPEC.md`, the spec should be ready to go.
- I do not currently see evidence that the whole Priority order needs to be redesigned; this looks
  like targeted spec cleanup rather than a Phase 13 replanning exercise.

### Sign-off

Audit completed by GitHub Copilot (GPT-5.4) against the current `PHASE13_SPEC.md` and the final
brainstorm record in this file. Main conclusion: the spec is strong overall, but posterior output
placement and two remaining data-contract ambiguities should be fixed before implementation.

---

## Copilot Audit of `PHASE13_SPEC.md` — Round 2 / codebase-conformance pass

### Verdict

**Still not quite ready for implementation sign-off.** Claude fixed the three findings from the
first audit, and those repairs were the right ones, but this second pass found a handful of new
issues when the spec was checked against live codebase conventions and the broader pipeline family.

### What improved since Round 1

- Posterior placement now conforms to the established grouped-output pattern:
  `02-posterior/03-rcn-mean-shift/` rather than a fake late step under the prior tree.
- Priority 13.5 is much sharper about the per-sample APS input surface.
- Priority 13.4 now resolves the internal ambiguity over what `10-amplicon-metrics/` means.
- The overall Priority order still looks sensible. I still do **not** see a major dependency
  inversion in the 13.1 → 13.8 sequence.

### Findings

#### FIND-R2-1 — The spec still contains stale 14-step/14-dir language after the switch back to 13 steps

The current spec correctly defines a **13-step** prior tree, but several places still say 14.

Examples from the current spec:
- `build_rcn_mean_shift_steps()` section says it returns all 14 step dirs
- `_INDEX_MD_TEMPLATE` update says "14-row block"
- `tests/test_pipeline.py` validation says verify all 14 dirs exist
- README / PIPELINE_SPEC update bullets still say "14-step layout"
- validation text still says "all 14 step dirs exist"

Why this matters:
- This is the kind of inconsistency that turns directly into implementation churn, especially in
  `output_layout.py`, tests, and docs.
- The spec is now using the correct 13-step model conceptually, but the stale 14-count wording
  leaves the implementation contract ambiguous.

Recommended repair:
- Sweep the spec for every stale `14-step`, `14-row`, `all 14 dirs`, and `steps 09–14` reference
  that should now be `13-step`, `13-row`, `all 13 dirs`, or `steps 09–13`.

Severity: **Medium**

#### FIND-R2-2 — Priority 13.1 still omits several live rename/convention touchpoints in core code

The spec now covers many important rename surfaces, but the live codebase shows additional core
files that are clearly touched by this work and are not currently named in Priority 13.1.

Most important omissions:
- `onionskin_core/autodetect.py`
  - `eligible_pipelines` still returns tuples containing `per-stage`
  - this affects CLI behavior and existing tests in `tests/test_pipeline.py`
- `onionskin_core/readme.py`
  - run-local `README.md` generation is pipeline-family-facing and already knows about posterior
    grouped outputs and manifest sidecars
  - the rename / layout work will affect the wording it emits
- `onionskin_core/__init__.py`
  - if `build_rcn_mean_shift_steps()` becomes part of the public helper surface, exports need audit
- `onionskin_core/detection.py`
  - imports the lazy loader from `common.py`; if loader names move, this file is touched too

Why this matters:
- These are not optional polish files. They are active code surfaces that influence pipeline
  resolution, user-facing output guidance, and import stability.
- The rename/layout Priority should own them explicitly so the implementation does not stop at the
  more obvious engine/controller files and leave stale behavior behind.

Recommended repair:
- Add these files explicitly to Priority 13.1 scope.

Severity: **Medium**

#### FIND-R2-3 — The spec's test/tooling sweep is still narrower than the real rename surface

The current spec names `tests/test_pipeline.py` and `scripts/summit_inspector.py`, but there are
additional live test/helper surfaces that clearly reference the old pipeline token or old behavior.

Examples:
- `tests/run_toy_single.sh`
  - uses `--pipelines per-stage`
  - comments describe the pipeline as "per-stage"
- `tests/eval_summit_precision.py`
- `tests/eval_summit_precision_v1.py`
- `tests/eval_summit_precision_v2.py`
  - all define `SUPPORTED_PIPELINES = {"auto", "growth", "hmm", "per-stage"}`
  - all contain explicit `per-stage` unsupported-path messaging
- `tests/UTILITY_SCOPE.md`
  - still refers to "Growth / Per-Stage Utilities"

Why this matters:
- The spec is making the rename internal **and** public-facing.
- That means rename-sensitive test/helper scripts need to be part of the planned sweep, not left to
  be discovered mid-implementation.

Recommended repair:
- Expand Priority 13.1's audit/update list to include these test/helper scripts.
- Where legacy alias support is intentional, say so explicitly for the tests.

Severity: **Medium**

#### FIND-R2-4 — Priority 13.8 still understates the existing posterior artifact contract

The spec now fixes the posterior directory location, but it still describes posterior outputs too
narrowly relative to the current posterior machinery.

Live-code convention:
- `onionskin_core/posterior.py` `build_posterior_manifest()` writes:
  - `posterior_manifest.tsv`
  - `posterior_manifest_cluster_map.tsv` alongside it
- `_run_posterior()` in `onionskin.py` also writes:
  - `posterior_hires_manifest_{i}.tsv` when hires manifests are present
- `onionskin_core/readme.py` already knows about these files as first-class posterior artifacts

Why this matters:
- If Phase 13 says rcn-mean-shift posterior support follows the existing family pattern, the spec
  should acknowledge the same posterior manifest sidecars the rest of that machinery already emits.
- Otherwise the implementation may technically work but still fall short of the codebase's current
  posterior-output expectations.

Recommended repair:
- Expand Priority 13.8 target state to include:
  - `02-posterior/posterior_manifest_cluster_map.tsv`
  - `02-posterior/posterior_hires_manifest_{i}.tsv` when hires inputs are present
- Make clear that rcn-mean-shift posterior support reuses the same posterior manifest builder
  contract rather than inventing a reduced one.

Severity: **Medium**

### Advisory

#### ADVISORY-R2-1 — The spec is silent on whether `rcn-mean-shift` intentionally has no notebook step

This is not a blocker, but it is a pipeline-family conformance question worth answering in the spec.

Context:
- Growth has a notebook step (`15-notebook/`)
- HMM has a grouped `notebooks/` directory
- The new `rcn-mean-shift` layout does not currently plan any notebook output step

Recommendation:
- Add one sentence in the spec stating either:
  - notebook generation is intentionally out of scope for Phase 13, or
  - a future notebook step is anticipated but not part of the current phase

That will prevent future agents from flagging this as an unexplained family inconsistency.

### Readiness assessment

My assessment after this second pass is:

- The spec is materially better than in Round 1.
- The Priority order still looks sound.
- The remaining issues are mostly **scope completeness / codebase-conformance cleanup**, not
  architectural rework.
- I would still **not** call it implementation-ready until FIND-R2-1 through FIND-R2-4 are fixed.

### Sign-off

Second-pass audit completed by GitHub Copilot (GPT-5.4), with additional checking against live
pipeline-family conventions in the codebase. Main conclusion: the updated spec fixed the big
architectural issues from Round 1, but it still needs one more completeness/conformance cleanup
pass before implementation should start.


## John's Feedback for Claude
- Determine exactly whether our final number of steps was 13 or 14, and make sure everything across the SPEC is consistent with that.
- Follow recommendation for posterior artifact contract. conform to the existing family pattern.
- No notebook for rcn-mean-shift was an oversight. Let’s add notebooks that are counterparts to notebooks found in the Growth and/or HMM pipelines, when it is feasible for RCN-Mean-Shift pipeline.
- Verify other issues, and update SPEC accordingly.

---

## Copilot Audit of `PHASE13_SPEC.md` — Round 3 / final sign-off pass

### Verdict

**Not quite ready for implementation sign-off yet.** The current spec is close: the 14-step
layout is now internally consistent, posterior placement matches the live family pattern, and I
do not see a priority-order or dependency-order problem. But one final pass against the live
codebase still found three concrete conformance gaps that should be resolved before Priority 13.1
starts.

### What I checked in this pass

- Public-facing rename surfaces in `onionskin.py`, `tests/test_pipeline.py`, `README.md`, and
  `PIPELINE_SPEC.md`
- Plot helper/module conventions across `onionskin_core/*_plots.py`
- Notebook helper and output-file conventions across `onionskin_core/notebooks.py` and
  `onionskin_core/hmm_notebooks.py`

### Findings

#### FIND-R3-1 — Public-facing rename scope is still underspecified beyond `--pipelines per-stage`

Priority 13.1 says the rename to `rcn-mean-shift` is internal **and public-facing**, but the spec
still only explicitly owns the `--pipelines per-stage` token change. The live codebase shows a
broader public surface that still carries the old name and needs an explicit decision.

Examples from live code:
- `onionskin.py` still exposes `--per-stage-norm-mode`
- `onionskin.py` help text still says `Available: growth, per-stage, hmm`
- the argparse section label is still `Per-stage / single-stage detection`
- runtime/status text still prints `per-stage=...`
- `PIPELINE_SPEC.md` currently documents `--per-stage-norm-mode` as the pipeline-specific flag
- `tests/test_pipeline.py` asserts these exact strings today

Why this matters:
- If the rename is truly public-facing, leaving these surfaces unnamed in the spec makes the
  implementation contract incomplete.
- If the intent is to preserve `--per-stage-norm-mode` as a legacy public flag, that also needs
  to be stated explicitly, because right now the spec implies a more complete rename than it
  actually defines.

Recommended repair:
- In Priority 13.1, add an explicit sub-decision for **pipeline-specific public rename surfaces**.
- Decide whether to rename `--per-stage-norm-mode` to `--rcn-mean-shift-norm-mode` with a legacy
  alias, or intentionally retain the old flag name as a documented compatibility exception.
- Include associated help-group titles, runtime/status strings, `PIPELINE_SPEC.md` Appendix A,
  README wording, and tests in that same decision.

Severity: **High**

#### FIND-R3-2 — Priority 13.6 still points at a non-existent `onionskin_core/plots.py`

The plot priority is directionally right, but it still names a catch-all module that does not
exist in the repo.

Current spec text:
- `write_profile_plots()` in `onionskin_core/plots.py` (or equivalent)
- `Reuse existing plot functions from onionskin_core/plots.py`

Live code layout:
- `onionskin_core/profile_plots.py` → `write_profile_plots()`
- `onionskin_core/summit_plots.py` → `write_summit_plots()`
- `onionskin_core/aps_plots.py` → `write_aps_plots()`
- plus `shape_filter_plots.py`, `qc_plots.py`, `overlap_plots.py`

Why this matters:
- The current wording is ambiguous enough that an implementer could waste time looking for a
  nonexistent abstraction or create a new wrapper module that the codebase does not otherwise use.
- This phase is specifically about family conformance, so the spec should point to the concrete
  module split the repo already uses.

Recommended repair:
- Rewrite Priority 13.6 to reference the real plotting modules/functions explicitly.
- If Claude wants to allow a future wrapper, say that separately; do not use `plots.py` as if it
  were already the concrete target.

Severity: **Medium**

#### FIND-R3-3 — Priority 13.7 notebook output contract diverges from existing notebook-family conventions

The new notebook priority correctly identifies the relevant growth notebook counterparts, but its
output contract does not match how notebooks are currently emitted elsewhere in the repo.

Live notebook convention:
- Growth notebooks are written as stable filenames like `aps_exploration.ipynb` and
  `gap_analysis.ipynb`
- HMM notebooks are also stable filenames like `fork_travel_explorer.ipynb` and
  `amplicon_atlas.ipynb`
- Existing helper functions take `notebook_dir` and write fixed notebook names into that directory

Current spec divergence:
- Target state says `14-notebook/{prefix}_aps_exploration.ipynb`
- Target state says `14-notebook/{prefix}_gap_analysis.ipynb`
- Proposed helper signatures are centered on `(rcn_ms_steps, out_prefix, ...)`, which departs from
  the current family pattern of `write_*_notebook(notebook_dir)` style entry points

Why this matters:
- This is not a correctness bug, but it is a family-conformance issue in a phase whose core goal is
  to make rcn-mean-shift feel like part of the same pipeline family.
- Prefixed filenames are a new convention not used by the existing growth or HMM notebook outputs.

Recommended repair:
- Either align Priority 13.7 to the existing notebook contract (`aps_exploration.ipynb`,
  `gap_analysis.ipynb` written into `14-notebook/`), or explicitly justify why rcn-mean-shift
  notebooks should intentionally break that pattern.
- If helper signatures differ for good reasons, separate the **helper API** decision from the
  **emitted filename** decision so the public output contract stays family-consistent.

Severity: **Medium**

### Readiness assessment

My assessment after this final pass is:

- The spec is close.
- I do not see any new missing Priority or mis-ordered Priority.
- The remaining issues are concentrated in **public rename completeness** and **concrete
  codebase-conformance details**, not architecture.
- After FIND-R3-1 through FIND-R3-3 are resolved, I would consider the spec ready to implement.

### Sign-off

Final-pass audit completed by GitHub Copilot (GPT-5.4) against the updated `PHASE13_SPEC.md` and
live codebase conventions. Main conclusion: one last spec revision is still needed, but the
remaining work is narrow and should not require another major redesign.

---

## Copilot Audit of `PHASE13_SPEC.md` — Round 4 / post-v0.13.13 conformance pass

### Verdict

**Still not quite ready to move on.** Claude's v0.13.13 revision did close the three Round 3
findings, and the spec is materially stronger than before. I still do not see a missing Priority
or a broken Priority order. But one more high-reasoning pass against live code and helper
contracts surfaced three remaining spec-level issues that should be fixed before implementation
starts.

### What I checked in this pass

- Whether the updated test sweep in Priority 13.1 actually matches the live rename-sensitive
  assertions in `tests/test_pipeline.py` and `tests/eval_summit_precision*.py`
- Whether Priority 13.6 now matches the real plot-helper output contracts in
  `profile_plots.py` and `summit_plots.py`
- Whether Priority 13.7's notebook validation is consistent with the spec's own gating rules for
  gap analysis and multistage unification

### Findings

#### FIND-R4-1 — Priority 13.1 still under-specifies the live test/helper rename surface

The spec improved the rename coverage for `tests/test_pipeline.py`, but the explicit checklist is
still incomplete and now includes one incorrect code reference.

Concrete problems:
- The spec says the import to update is `from onionskin_core import _resolve_per_stage_norm_mode`,
  but the live file actually imports from `onionskin`, not `onionskin_core`.
- The spec names only a subset of the live assertions that will change under the public rename.
  The current file also asserts:
  - help-group label text (`"Per-stage / single-stage detection"`)
  - `eligible_pipelines` tuples from `autodetect.py`
  - `_resolve_requested_effective_pipelines(args) == {"growth", "per-stage", "hmm"}`
  - `_pipelines_include(args, "per-stage") is True`
  - runtime status strings like `per-stage=chrom-median` / `per-stage=ref-stage`
  - the error string `per-stage normalization mode ref-stage requires ...`
- The summit-precision helper sweep is also still narrower than the live surface. The spec names
  `SUPPORTED_PIPELINES` and unsupported-path logic, but the live files also expose
  `--pipeline auto|growth|hmm|per-stage` in usage/help text.

Why this matters:
- This phase is explicitly renaming the pipeline publicly. These tests are contract checks, not
  incidental strings.
- If the implementation follows the current spec literally, it is likely to miss several failing
  assertions and one wrong import rewrite.

Recommended repair:
- Expand Priority 13.1 item 6 to explicitly include the remaining `tests/test_pipeline.py`
  assertions above and correct the import source to `from onionskin import ...`.
- In item 18, explicitly include the CLI usage/help string updates in
  `tests/eval_summit_precision*.py`, not just `SUPPORTED_PIPELINES` and the unsupported-path
  message.
- State clearly which `--pipelines per-stage` test cases remain as intentional legacy-alias
  coverage versus which should switch to canonical `rcn-mean-shift` usage.

Severity: **High**

#### FIND-R4-2 — Priority 13.6 still does not conform to the real plot-helper output contract

Round 3 fixed the module names, but the actual output-path contract in Priority 13.6 is still out
of sync with the helpers the spec says to reuse.

Live helper behavior:
- `write_profile_plots()` writes per-call profiles into `plots_dir/profiles/` with names like
  `profile_{call_id}.png`
- `write_profile_plots()` writes chromosome overviews into `plots_dir/genome_overview/` with
  names like `chrom_{chrom}.png`
- `write_summit_plots()` writes into `plots_dir/summit_fits/`, not `summit_fit/`

Current spec mismatches:
- Target state still shows `11-plots/{prefix}_stage{st}_profile.png`
- Validation still expects `11-plots/{prefix}_stage1_profile.png`
- Validation still expects `11-plots/genome_overview/{prefix}_genome_overview.png`
- The implementation snippet passes `plots_dir=os.path.join(rcn_ms_steps["plots"], "summit_fit")`
  into `write_summit_plots()`, which would not match the helper's own `summit_fits/` convention
- The `calls_tsv=<post-dedup unified calls from rcn_ms_steps["deduplication"]>` example is
  internally contradictory: `05-deduplication/` holds post-dedup canonical stage calls, not
  unified calls

Why this matters:
- This is now a concrete code-conformance problem, not just a naming preference.
- The spec either needs to align to the existing helper outputs or explicitly say that Phase 13.6
  will add adapter/wrapper logic that changes those outputs.

Recommended repair:
- Decide whether Priority 13.6 is reusing the existing helper output contract as-is or wrapping it.
- If reusing as-is, update the target state and validation to match `profiles/`,
  `genome_overview/chrom_{chrom}.png`, and `summit_fits/summit_{call_id}.png`.
- If wrapping, say so explicitly and describe the wrapper-owned output contract.
- Fix the `calls_tsv` example to reference either post-dedup canonical stage calls or the actual
  unified-calls file from `08-multistage-unification/`, but not both at once.

Severity: **Medium**

#### FIND-R4-3 — Priority 13.7 notebook validation conflicts with the spec's own gating rules

The notebook naming/signature issue is fixed, but the validation section still contains one
internal contradiction.

Spec facts already established elsewhere:
- Priority 13.4 defines gap analysis as downstream of unified calls in
  `08-multistage-unification/`
- Priority 13.1 says `08-multistage-unification/` is only populated when `N≥2` stages
- Priority 13.7 says notebooks are gated on whether prerequisite outputs are present

Current contradiction:
- Validation says: `For single-stage run (no APS): only gap notebook emitted; 14-notebook/ not empty`

Why this matters:
- On the spec's own rules, a true single-stage run has no step-08 unified output, so gap analysis
  would not obviously run, and the gap notebook would not obviously exist.
- That makes the validation contract internally inconsistent and could cause unnecessary audit or
  test churn later.

Recommended repair:
- Either explicitly broaden gap analysis to support single-stage inputs, or
- change the notebook validation so single-stage runs are allowed to emit no notebooks when neither
  APS nor gap-analysis prerequisites are present.

Severity: **Medium**

### Readiness assessment

My assessment after this Round 4 pass is:

- The spec is close and the remaining issues are narrower than in Round 3.
- I still do not see any missing Priority or bad execution order.
- The remaining problems are about **concrete contract accuracy**: tests, helper output shapes,
  and one gating inconsistency.
- After FIND-R4-1 through FIND-R4-3 are fixed, I would consider the spec ready to move on.

### Sign-off

Round 4 audit completed by GitHub Copilot (GPT-5.4) against the updated `PHASE13_SPEC.md` and
live codebase helper contracts. Main conclusion: the spec is very close, but it still needs one
more narrow correction pass before implementation should begin.

