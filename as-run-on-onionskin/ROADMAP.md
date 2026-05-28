# ROADMAP

`ROADMAP.md` is a **retrospective + light-reading bird's-eye echo** of the
onionskin project's phase plans — not a forward-looking compass. The phase
system itself (the SPEC + AUDIT_LOG + STRATEGY + FEEDBACK files in
`multi-agent/plans/` and `multi-agent/plans/archived/`) is the actual
roadmap; CHANGELOG `**Roadmap:**` lines that say "Phase X — Phase Y" are
pointing at the phase plans themselves, not at sections of this file.

This file collects light-reading retrospective entries for each phase
once that phase's SPEC has been formalized, with `✓ DONE (v0.x.xx)` /
`◑ PARTIAL (v0.x.xx)` markers added as priorities close. Earlier phases
(Phase 4 through Phase 11) carry more detailed historical entries because
those entries pre-date the current convention; from Phase 12 onward, the
new light-reading + completion-markers pattern applies.

For the canonical role / update-trigger / forbidden-pattern rules, see
`multi-agent/AGENT_CONVENTIONS.md § ROADMAP.md`. For candidate future
phases (in early-stage SOUP form), see `multi-agent/plans/next/`.

---

## How to read this file

**Phases** are major development clusters (Phase 4, Phase 5, …). Phase
numbers align with the middle digit in changelog versions: Phase 4 work
lands in `v0.4.xx` entries.

**Status markers:**
- ✓ DONE (v0.x.xx) — fully implemented and tested; entry may be collapsed to a one-liner + tag
- ◑ PARTIAL (v0.x.xx) — partially implemented; remaining work described below
- *(unmarked)* — not yet started (rare in this file under the new
  retrospective convention; new phases get an entry only once their
  SPEC formalizes, so unmarked priorities here are historical artifacts
  from the older forward-looking framing)

**Cross-references:** Each CHANGELOG entry has a `**Roadmap:**` line.
Under the new convention, that line names the phase plan itself
("Phase X — Phase Y" titles in `multi-agent/plans/` or
`multi-agent/plans/archived/`), not a section of this file. Each
`✓ DONE (v0.x.xx)` / `◑ PARTIAL (v0.x.xx)` marker here points at the
CHANGELOG entry where it landed.

---

## Historical preamble (preserved)

The remainder of this file's earlier sections (Phase 4 through Phase 11)
were authored under the older forward-looking convention and contain
substantial design detail rather than light-reading summaries. They are
preserved as-is for historical context. From Phase 12 onward (added
during the DEVPLAN-ROADMAP-AND-NEXT-DIRECTORY-SYSTEM cycle, DEVLOG
v0.14.75.3 + v0.14.75.4), entries follow the new light-reading +
completion-markers pattern.

---

# Phase 4 — Output completeness and correctness ✓ SUBSTANTIALLY COMPLETE

## Goals
Phase 4 is about making onionskin more biologically faithful and easier to use on real datasets.

## Phase 4 closure note (v0.4.31)

All core Phase 4 priorities are done or closed. Two items are carried forward to Phase 5 as incremental improvements rather than blockers:
- **4.9 remaining**: graduated gap thresholds, gap-count weighting, amplicon_class integration, shape-score soft signal, `--keep-amplicons-bed` override.
- **4.11 remaining**: stage-activity fold-change in `timing.py` should use `final_origin_bp` from `_origins.tsv` rather than `median_peak_fold` from the bump model (larger refactor; defer).
- **4.10** (`--refine-summits`): gated on summit quality improvements; defer to Phase 5 or later.

---

## Priority 4.0 — APS correctness audit ✓ DONE (v0.3.28)

Full audit of `aps.py` against PIPELINE_SPEC.md completed in v0.3.28. No logic bugs found in APS computation, clustering, or ordering. Three spec/docstring corrections made (clamped `summit_excess` formula, `total_locus_bp` zero-guard, `compute_and_write_aps` docstring). APS rank direction confirmed intentional (rank 1 = least amplified).

---

## Priority 4.1 — CLI exposure and cleanup ✓ DONE (v0.3.07)

All engine options exposed as named CLI flags in v0.3.07. The `--extra` pass-through was retained for legacy compatibility but removed in v0.5.65 — all parameters now have named flags. Argument groups organized: Input/Output, Detection—shared, Single-sample specific, Multistage specific, Timing, Overlap resolution, APS, Advanced. Subsequent sessions added incremental flags (`--dedup-peak-dist-kb` v0.3.18, `--aps-feature` v0.3.31, etc.) organically without breaking the structure.

---

## Priority 4.2 — Complete sample/stage track outputs ✓ DONE (v0.4.00)

`samples/` and `stage_medians/` confirmed populated with both `RCN.bedGraph` and `log2RCN.bedGraph` per sample/stage (verified v0.4.00). The `others/` directory — previously created unconditionally — is now lazy: only created when a file is actually swept there, so it no longer appears as an empty artifact in typical runs.

---

## Priority 4.3 — Max-vs-median reporting clarity ✗ CLOSED

`_stage_summary.tsv` already reports `median_peak_fold`, `mean_peak_fold`, and `max_peak_fold` as distinct
columns. No specific downstream computation or output column was identified as ambiguously labeled or using
the wrong statistic. Closed without action — reopen if a concrete problem surfaces.

---

## Priority 4.4 — Twin-peak decomposition ✓ DONE (v0.3.18–v0.3.22)

Twin-peak decomposition is complete. `make twin` and `make full` both pass for all known twin-peak loci (spots 1 and 2 hard requirements; spot 3 informational). The implementation uses multi-stage consensus valley detection with NMS-based peak selection, local-outlier masking, and separate split parameters for single vs. overlapping calls.

---

## Priority 4.5 — Twin-peak tests in standard suite ✓ DONE (v0.3.21)

Twin-peak training data is included in the release. Standard validation now uses make targets:

```bash
make test    # pytest unit tests
make toy     # multistage smoke test
make single  # single-file / single-stage smoke test
make twin    # twin-peak decomposition
make full    # full chr II: sensitivity + no-split + twin (both datasets)
```

---

## Priority 4.6 — Known output metric bugs ✓ DONE (v0.3.24)

All five bugs from the Phase 3.16 spec audit were fixed in v0.3.24: hires loop sort order, `n_supporting_stages` column lookup, `symmetry_ratio`/`curvature_metric` aggregation, `model_bic` key, and dead code removal.

*(Original bug list preserved below for reference.)*

### Bugs fixed (from Phase 3.16 spec audit)
1. **Hires loop "coarsest wins"** (`single_engine.py`): resolutions are iterated finest-first but results are overwritten, so the coarsest resolution silently wins. Fix: reverse the sort order. *Highest priority — directly degrades summit accuracy.*
2. **`n_supporting_stages` always 0**: column lookup uses wrong column name (`median_peak_log2` vs. actual `median_peak_fold`).
3. **`symmetry_ratio` and `curvature_metric` always NaN** in `_origins.tsv`: per-stage values are never aggregated from the progression table into the origins row.
4. **`model_bic` always NaN** in `_progression.tsv`: looks for key `"bic"` but the model returns `"sse"`.
5. **Dead code / silent ignores**: `write_summit_beds()` never called; `--profile-space` flag silently ignored; unreachable block in `timing.py`.

---

# Phase 4.7 — Automated visualization outputs

## Goals
Generate human-interpretable figures at each pipeline stage that can be dropped directly into presentations or papers.  All figures saved as both PDF (vector, for Keynote / Illustrator label editing) and PNG (preview).

---

## Priority 4.7.1 — APS dendrogram and heatmap ✓ DONE (v0.3.33)

Publication-ready APS clustering figures are now generated automatically in `onionskin_core/aps_plots.py` and called from `compute_and_write_aps`:

- **`aps_dendrogram_raw.pdf/.png`** — Ward/Euclidean dendrogram, branches dark gray, leaf labels colored by cluster.  Both raw and z-scored variants.
- **`aps_heatmap_raw.pdf/.png`** — Rows = samples (leaf order), columns = loci; left cluster color strip; colorbar. Both raw and z-scored variants.
- Labels are separate vector text objects in PDF — selectable and hideable in Keynote/Illustrator.

---

## Priority 4.7.2 — Prior-ordered and scalar-ordered APS heatmaps ✓ DONE (v0.4.02–v0.4.03)

Four alternative heatmap orderings (v0.4.02): prior-stage groups (within-group APS-descending), and three global scalar orderings (area / summit / width APS descending). Per-amplicon-ordered heatmaps completed in v0.4.03 (unblocked by Priority 4.7.11).

---

## Priority 4.7.11 — Amplicon importance ranking and per-amplicon-ordered heatmaps ✓ DONE (v0.4.03–v0.4.05)

Amplicon importance ranking implemented in v0.4.03; revised to 4-component score in v0.4.04 (33/33/16.5/16.5%: onset earliness, max_RCN, n_active_stages, activity_breadth). Timing module extended in v0.4.04 with `n_active_stages`, `latest_activity_stage`, `activity_breadth`, and `amplicon_class` (Founder, Finisher, Constitutive_Prolonged, Facultative_Prolonged, Intermediary). `*_timing_report.md` written automatically alongside each `*_timing.tsv`. Timing robustness completed in v0.4.05: `--timing-exclude-loci BED` (removes outlier false-positive regions from global reference), `--timing-onset-quantile Q` (default 0.10; quantile-based reference for `lag_from_earliest`, which is now signed and can be negative for amplicons earlier than the reference). Importance score fixed to use `_norm01(-lag)` (handles negative lag correctly). Importance ranking is the canonical amplicon priority list for the run — informs growth curves and profile snapshots in future phases. **Timing-specific plots: not yet planned — add to a future priority.**

---

## Priority 4.7.2-orig — Per-amplicon developmental growth curves ✓ DONE (v0.4.14–v0.4.15)

4-panel per-amplicon figure: summit excess (linear), log₂(peak RCN), width above 1.5× (kb),
area excess vs stage. Stage median thick blue line; individual samples thin + alpha=0.35.
Cluster coloring from APS clusters when available (see 4.7.7). Output to `aps/growth_curves/`;
multi-page `aps/growth_curves_all.pdf`.

---

## Priority 4.7.3 — Stage progression heatmap ✓ DONE (v0.4.14–v0.4.15)

Rows = amplicons, columns = stages, cells = median peak RCN. `YlOrRd` color scale, vmin=1.
Output: `aps/stage_heatmap.pdf/png`.

---

## Priority 4.7.4 — Per-amplicon RCN profile snapshots ✓ DONE (v0.4.16)

Genomic-position RCN profile per amplicon: stage-median thick line; per-sample thin alpha=0.18 lines colored by stage (viridis); summit dashed vertical; split-point verticals; 20% pad. Output to `plots/profiles/profile_<call_id>.pdf/png`.

---

## Priority 4.7.5 — Shape filter diagnostic ✓ DONE (v0.4.19)

Per-amplicon triangle fit vs actual RCN, PASS/FAIL verdict with dBIC score.
Works in single-stage and `--onionskin` modes; multistage mode skips (shape filter
not applied there). Output: `plots/shape_filter/shape_{call_id}.pdf/.png`.

---

## Priority 4.7.6 — Summit diagnostic plot ✓ DONE (v0.4.19)

Per-amplicon zoomed summit region with per-stage RCN traces, weighted-average
combined profile, parabola fit, and bootstrap CI shading. Works with base-resolution
only (no hires manifest required). Handles both multistage and single-mode column
naming. Output: `plots/summit_fits/summit_{call_id}.pdf/.png`.

---

## Priority 4.7.7 — Growth curve per amplicon: APS ranking context ✓ DONE (v0.4.14)

Implemented as part of 4.7.2-orig: individual sample lines in growth curves are colored by
APS cluster assignment when `clusters_for_strip` is available. Cluster legend shown on panel 1.

---

## Priority 4.7.8 — Per-overlap / twin-peak valley diagnostic ✓ DONE (v0.4.18)

One figure per examined span (cluster or single call): per-stage RCN profiles with peaks (▲),
valleys (▼ + depth score), split decision + accepted split lines; call-bar showing before/after
boundaries. Output: `plots/overlap_diagnostics/span_{chrom}_{start}_{end}.pdf/png`.

---

## Priority 4.7.9 — Genome-wide chromosome overview ✓ DONE (v0.4.16)

Two-panel figure per chromosome: stage-median RCN filled-area traces (top, 5:1 ratio) + amplicon bars with call_id labels (bottom). Output to `plots/genome_overview/chrom_<chr>.pdf/png`.

---

## Priority 4.7.10 — QC plots ✓ DONE (v0.4.17)

Stacked-density (ridgeline) figure: genome-wide vs within-calls RCN per sample, sorted
by stage, bin-width-weighted KDE, viridis stage coloring. Bin-size histogram dropped
(developer-only interest, not user-facing). Output: `plots/qc/rcn_distributions.pdf/png`.

---

## Priority 4.7.12 — Jupyter notebook output directory (`notebook/`) ✓ DONE (v0.4.24)

Add a `notebook/` subdirectory adjacent to `prior/` in each run's output layout.  Each notebook
is a self-contained Jupyter `.ipynb` with relative paths to that run's outputs and starter cells
for loading + plotting.

### Planned notebooks

| Filename | Contents |
|---|---|
| `aps_exploration.ipynb` | Load APS TSVs and matrices; re-run clustering at user-chosen k; scatter plots of scalar APS |
| `timing_analysis.ipynb` | Load timing TSV; plot onset lag distribution; re-rank amplicons interactively |
| `growth_curves.ipynb` | RCN progression curves per amplicon across stages; overlay multiple amplicons |
| `fork_progression.ipynb` | Width left/right vs stage; symmetry ratio; velocity estimates |
| `gap_analysis.ipynb` | Load gap_analysis TSVs; interactive filter by concern level; re-generate BED exclusion lists |

### Layout

```text
outdir/
├── prior/
│   └── notebook/
│       ├── aps_exploration.ipynb
│       ├── timing_analysis.ipynb
│       ├── growth_curves.ipynb
│       ├── fork_progression.ipynb
│       └── gap_analysis.ipynb
```

Notebooks reference results using relative paths (e.g., `../aps/…`, `../timing.tsv`).

---

## Priority 4.8 — Parallelization ✓ DONE (v0.4.10, v0.4.28)

Embarrassingly parallel improvements to reduce wall time during interactive development.

### Implemented (v0.4.10)

- `make fast` — runs `test + toy + single + twin` in parallel via `make -j4`
- `write_aps_plots` — dispatches 8–12 independent matplotlib renders to a `ProcessPoolExecutor`
- Multistage per-call origin refinement loop — `ThreadPoolExecutor` with per-call RNG seeds; eliminates long hang after "[multistage] after dedup: N call(s)"
- `_bootstrap_origin` (single_engine) — 200 bootstrap replicates dispatched to `ThreadPoolExecutor` with per-replicate RNG seeds

### Implemented (v0.4.28)

- `write_summit_plots` — per-amplicon summit diagnostic renders dispatched to `ProcessPoolExecutor`
- `write_shape_filter_plots` — per-amplicon shape filter renders dispatched to `ProcessPoolExecutor`
- `write_profile_plots` — per-amplicon profile snapshot renders dispatched to `ProcessPoolExecutor`; chromosome overviews remain sequential (single figure per chrom)
- All three functions accept `max_workers` and are threaded from `onionskin.py` via `--threads`

### Not parallelized (deferred or not worth it)

- `write_overlap_diagnostic_plots` — uses pre-loaded DataFrames in `bg_cache`; refactor needed; typically few spans
- `compute_aps_tables` — sequential over samples × loci; fast enough at current scale
- Gap annotation, APS matrix construction — fast at current scale

---

## Priority 4.9 — Integrated final amplicon keep/exclude recommendations ✓ DONE (v0.4.13)

All four output files implemented and integration logic working (gap_concern → keep/exclude
with width-growth rescue). Future enhancements (graduated thresholds, amplicon_class
integration, user override `--keep-amplicons-bed`) ported to BRAINSTORM.md.
Full entry archived in ROADSTRAVELED.md (collapsed v0.5.50).

---

## Priority 4.10 — Post-hoc summit refinement (`--refine-summits`) ✗ DROPPED (v0.5.50)

Never implemented (placeholder only). Dropped to focus on HMM integration (Phase 7).
Full design preserved in ROADSTRAVELED.md and BRAINSTORM.md; revisit after Phase 7 if needed.

---

## Priority 4.11 — Unified summit position for all summit-related computations ✓ DONE (v0.5.08 / v0.5.50)

APS_summit fix implemented: `final_origin_bp` from `_origins.tsv` threaded through
`summit_bp_map` into `aps.py`; APS_summit now samples RCN at refined origin ± window.
Stage-activity fold-change fix in `timing.py` remains deferred (larger refactor; future
item if needed). Full entry archived in ROADSTRAVELED.md (collapsed v0.5.50).

---

## Priority 4.12 — APS clustering: auto-k selection and singleton guard ✓ DONE (v0.4.31)

### Problem

The previous auto-k logic (`cluster_k=0 → max(2, n_stages)`) was a flat cap, not a real selection. It produced a fixed k equal to the number of prior stages regardless of the actual cluster structure in the APS matrix, often yielding singletons (clusters with a single sample). Singleton clusters are meaningless for posterior grouping and make dendrogram coloring confusing.

### Design

**Auto-k selection via linkage acceleration (elbow criterion):**
- After computing the Ward linkage matrix `Z`, examine the gaps between consecutive merge distances (second differences of `Z[:, 2]`).
- The merge step with the largest acceleration marks the most natural cut point in the dendrogram.
- `n_stages` is retained as the upper bound for the search; the elbow selects the best k ≤ n_stages.

**Singleton guard (post-processing reassignment):**
- After `fcluster`, any cluster containing exactly one sample is a singleton.
- For each singleton sample, compute Euclidean distance to each non-singleton cluster centroid in APS feature space.
- Reassign the singleton to the nearest centroid.
- Applied after auto-k selection and after explicit-k selection alike.
- If all clusters are singletons (n_samples ≤ k), no reassignment is possible; clusters returned unchanged.

### Example

10 samples, 5 prior stages, auto-k=5 (old behavior): clusters of sizes 2, 1, 4, 1, 2 — two singletons.
After singleton guard: singletons merge into nearest neighbors → 3 clusters with sizes 3, 5, 3 (approximately). The elbow criterion may also select k=3 directly from the linkage structure.

---


# Phase 5 — Posterior reruns and APS evolution

## Goals
Move from APS as a descriptive layer to APS as an inferential engine. Also carry forward incremental quality improvements from Phase 4 that were not blockers.

---

## Priority 5.0 — Carried forward from Phase 4 ◑ PARTIAL

### 5.0.1 — Keep/exclude recommendation refinements ✓ RESOLVED (v0.5.50)
Ported to BRAINSTORM.md [2026-04-07] as a possible side quest. Not blocking.

### 5.0.2 — Stage-activity fold-change using refined summit ✗ DEFERRED (post-HMM → see BRAINSTORM.md)
- Non-trivial refactor of `timing.py`; deferred until after Phase 7 HMM integration.
- Full design notes in BRAINSTORM.md [2026-04-07].

### 5.0.3 — `--refine-summits` standalone re-run ✗ DROPPED (v0.5.50)
Dropped to prioritize HMM integration. Design preserved in BRAINSTORM.md [2026-04-07] and ROADSTRAVELED.md.

---

## Priority 5.1 — Posterior reruns ✓ DONE (v0.5.01)

### Motivation

The prior manifest assigns samples to stages based on external biology (morphological stage, developmental time point, etc.). After APS clustering the samples are grouped by their actual DNA amplification profile. These groups may differ meaningfully from the prior groups — e.g. 5 prior stages → 3 APS-based posterior groups. The posterior groups should be ordered earliest → latest by mean APS (or cluster dendrogram position), then the detection + timing + visualization modules should be re-run with the posterior manifest so that users can directly compare morphology-driven versus DNA-driven analyses.

### Concrete example

10 samples, 5 prior stages (2 each), APS clustering (after singleton guard) → 3 posterior groups sized ~3, ~5, ~3. Posterior groups ordered by mean APS_area → early / mid / late. Re-run onionskin on the posterior manifest → outputs land in `posterior/`.

### Design

**Step 1 — Build posterior manifest**
- Read `aps/aps_clusters_raw.tsv` (sample_id → cluster).
- Order clusters by mean APS_area from `aps/aps_order_area.tsv` → cluster rank = posterior stage.
- Write `posterior_manifest.tsv` with columns `[sample_id, bedgraph_path, stage, prior_stage, prior_cluster]`.

**Step 2 — Selective pipeline re-run**
Modules to re-run on posterior manifest:
- detection (call amplicons using posterior stage groupings)
- timing (re-compute timing metrics with posterior stages)
- profile plots, stage progression heatmap, growth curves
- APS (re-run on posterior manifest → `posterior/aps/`)

Modules to **skip**:
- APS clustering itself (the posterior groups ARE the clustering result; no need to re-cluster)
- Hires refinement (expensive; summit positions from prior refinement can be re-used)
- Shape filter (no new bedGraphs; same per-sample shape scores apply)

**Step 3 — Output layout**
```text
outdir/
├── prior/          ← existing outputs, unchanged
└── posterior/
    ├── posterior_manifest.tsv
    ├── detections/ ← re-called amplicons using posterior stage assignments
    ├── timing/     ← re-computed timing using posterior stages
    ├── aps/        ← APS re-run on posterior manifest
    ├── plots/
    │   ├── profiles/
    │   ├── growth_curves/
    │   └── stage_heatmap/
    └── summary/
```

**Step 4 — CLI integration**
- `--posterior` flag triggers Step 1–3 automatically after main pipeline completes.
- `--posterior-manifest PATH` skips Step 1 and uses a user-supplied manifest directly (advanced).
- Both write to `posterior/` adjacent to `prior/`.

### Desired result

Users can compare morphology-driven prior analyses against DNA-driven posterior analyses side by side. The posterior re-run uses the same detection and visualization logic, so differences in outputs are attributable to the grouping alone.

### Dependencies

- Priority 4.12 (singleton guard) must be complete — posterior groups must be meaningful (no singletons).
- APS order tables (`aps_order_area.tsv`, `aps_clusters_raw.tsv`) must be written — already done.

### Status

Not started. Implement after Phase 4 is substantially complete.

---

## Priority 5.2 — APS weighting strategies ✓ DONE (v0.5.49)

Per-stage MAD tracks, probabilistic oscillation test, oscillation annotation, and global
degradation report implemented (v0.5.47–v0.5.49).  Scope changed from APS weighting to
oscillation annotation: confirmed real amplicons all showed statistically credible dips
under the Gaussian overlap test — no reliable discriminator between spurious and real
oscillation was found with available data.  `locus_weight` = 1.0; `--aps-weight-loci`
retained as experimental no-op.  APS weighting goal preserved in BRAINSTORM.md.
Full original entry archived in ROADSTRAVELED.md.

---

## Priority 5.3 — APS order stability ✗ DEFERRED (post-HMM → see BRAINSTORM.md)
Bootstrap diagnostics for APS ordering robustness (loci, samples, raw vs zscore clustering stability).
Full design notes preserved in BRAINSTORM.md [2026-04-07].

---

## Priority 5.4 — Better posterior ordering logic ✗ DEFERRED (post-HMM → see BRAINSTORM.md)
Refine scalar/dendrogram/hybrid ordering relationship for posterior grouping.
Full design notes preserved in BRAINSTORM.md [2026-04-07].

---

## Priority 5.5 — Cluster-k selection methods (`--aps-cluster-k`) ✓ DONE (v0.3.30, v0.4.12, v0.5.52)

### Current state

Three conceptually distinct k-selection strategies now exist:

| Method | CLI value | Status | Description |
|--------|-----------|--------|-------------|
| `keep` | (was sentinel `0`) | ✓ Implemented (v0.3.30) | Use n_stages from the input manifest — the original default |
| `elbow` | (current default) | ✓ Implemented (v0.4.12) | Linkage acceleration: second difference of Ward merge distances; elbow = most natural cut |
| `sweep` | `auto` (proposed) | ✗ Not yet implemented | Silhouette sweep over k=2..min(n−1,10); pick k with highest average silhouette score |

`auto` is currently unimplemented; if added, propose making it an alias for `elbow` until
empirical comparison on real data determines which method generalizes better.
`elbow` is the recommended default: it reflects actual dendrogram structure, is
computationally free (uses the already-computed linkage matrix), and is less sensitive
to cluster geometry than silhouette.

### Implementation (v0.5.52)
All three modes implemented and wired to `--aps-cluster-k` (type: str):
- `'elbow'` (default): existing `_optimal_k_from_linkage`, no extra dependencies
- `'sweep'`: new `_optimal_k_silhouette`; requires scikit-learn
- `'keep'`: resolves to k=n_stages explicitly
- Integer strings also accepted (e.g., `--aps-cluster-k 3`)
- Legacy `0` accepted as alias for `'elbow'`
- Verbose output logs chosen k and per-k silhouette scores for sweep mode

Empirical comparison of elbow vs sweep on DS1/DS2 is a suggested follow-up to determine
whether the default should change.

---

## Priority 5.6 — External feature matrix input ✗ DROPPED

Dropped: the alignment problem (user cannot know `call_id`s or resolved amplicon coordinates
in advance) makes a raw feature matrix input fragile and error-prone. Better design: onionskin
ingests additional signal tracks (HMM bedGraph, chromatin accessibility, RNA-seq) and computes
features itself from its own call coordinates. See BRAINSTORM.md [2026-04-07] for that vision.

---

## Priority 5.7 — `--no-aps-singleton-guard` CLI flag ✓ DONE (v0.5.08)

`--no-aps-singleton-guard` flag added.  When present, passes `fix_singletons=False` to
`hierarchical_cluster`, preserving exact k groups even when some contain only one sample.
Use with `--aps-cluster-k N` to force a specific number of posterior groups for comparison.

---

## Priority 5.8 — Fix summit estimation: replicate-weighted combined profile is grouping-invariant ✓ DONE (v0.5.05–v0.5.07)

### Problem (resolved in v0.5.05)

`refine_origin_for_call` used `stage_weight_mode="replicate"` by default, making the combined
profile ≈ unweighted grand mean of all samples, invariant to stage regrouping.  Prior and posterior
runs produced identical `final_origin_bp`.

### What was done (v0.5.05)

- **Option B implemented:** Per-stage argmaxes — smooth each stage's median profile independently,
  take argmax → one position per stage.  Aggregate as both weighted mean (`final_origin_bp`) and
  unweighted median (`final_origin_bp_median`).  Both written to `_origins.tsv` with separate
  bootstrap CIs.
- Both are now meaningfully different between prior and posterior runs (confirmed experimentally).

### Remaining (all resolved)

- Default weighting changed to `"equal"` in v0.5.07.  ✓
- Comprehensive stage weighting improvements (5.9.3) — deferred post-HMM. See BRAINSTORM.md.
- Summit strategy validation (5.9.4) — deferred post-HMM. See BRAINSTORM.md.
- Option D (HMM-intersection) deferred to Phase 7.

---

## Priority 5.9 — Summit estimation: comprehensive algorithm improvements

### Context

Summit estimation affects origin reporting (`_origins.tsv`), APS summit scoring, timing
stage-activity thresholds, and all summit visualizations.  Several unresolved questions have
accumulated.  This priority consolidates them.

---

### 5.9.1 — Double-smoothing audit ✓ DONE (v0.5.17)

Resolved empirically. v0.5.17 tested replacing `smooth_mean` with `rolling_median` in
`refine_origin_for_call`; results were neutral-to-worse (ds2_1000bp ORC-Core dist 1,340→1,868 bp;
ds2_500bp missed L4). Reverted to `smooth_mean`. Code audit confirms `smooth_rcn_median` is NOT
applied to sample_tracks in the multistage argmax path — profiles entering `refine_origin_for_call`
are raw log2 values; only one `smooth_mean` is applied. Double-smoothing is not occurring.

---

### 5.9.2 — Sub-bin resolution philosophy ✓ DONE (v0.5.54–v0.5.56)

Both levels of precision are reported in all modes:
- Multistage: `argmax_bp` (bin-center), `argmax_bin` (IGV interval), `parabola_vertex_bp`,
  `parabola_vertex_bin` in `_stage_summit_estimates.tsv`; aggregate estimates in
  `_aggregate_summit_estimates.tsv`; all as BED entries in `_summit_estimates.bed`.
- Single-file/single-stage: `summit_parabola_bp`, `sliding_offset_bp`, `sliding_offset_hires_bp`
  in `_origins.tsv` and `_metrics.tsv`.
- Sliding-offset refinement (5.11) also addresses bin-boundary bias: sweeps all partition offsets
  to find the 5 kb bin that best centers the origin, providing a bin-level estimate that is not
  biased by arbitrary binning phase.
- Guidance: do not over-interpret single-bp positions from coarse bins. Use bin intervals as
  honest precision; parabola/sliding-offset as best point estimates.

---

### 5.9.3 — Stage weighting: timing-guided active-stage weights ✗ DEFERRED (post-HMM → see BRAINSTORM.md)

Full design preserved in BRAINSTORM.md [2026-04-07]: timing-guided `--stage-weight-mode timing`,
center-weighted active window, per-stage parabola validity as data-driven weight proxy.
Depends on timing module stability and HMM integration (Phase 7) for active-window estimates.

---

### 5.9.4 — Iterative summit ↔ timing convergence (EM-like) ✗ DEFERRED (post-HMM → see BRAINSTORM.md)

EM-like iteration between summit estimation and timing active-window. Depends on 5.9.3.
Full design preserved in BRAINSTORM.md [2026-04-07].

---

### 5.9.5 — Summit strategy validation ✓ DONE (v0.5.09–v0.5.16)

Benchmark infrastructure complete. `make summit`, `make summit-smoke`, `make summit-baseline`
exist and run prior + posterior evaluations (`_eval_one_posterior` in test script).
Parabola outperforms argmax in 7/8 II/9A runs; `final_origin_bp` updated to `parabola_mean_bp`
in v0.5.12 (Priority 5.9.7). Posterior summit eval added to suite in v0.5.13–v0.5.16 (Priority 5.9.8).
II/2B has insufficient experimental ground truth for rigorous benchmarking; other chr II amplicons
have no validated origin coordinates. The benchmark scope is appropriately limited to II/9A.

Benchmark results archived below (v0.5.09 — II/9A, DS1 + DS2):

| Run | argmax_mean | parabola_mean | Best |
|-----|-------------|---------------|------|
| DS1 5000bp | 33,454,412 | 33,467,891 ✓L4 | parabola_median (2,531 bp) |
| DS1 500bp  | 33,457,324 | **33,470,571 ✓L4** | **parabola_mean (108 bp)** |
| DS1 hires  | 33,457,324 | **33,470,246 ✓L4** | **parabola_mean (217 bp)** |
| DS2 5000bp | 33,471,111 ✓L4 | 33,466,765 ✓L3 | argmax_mean (648 bp) |
| DS2 hires  | 33,464,500 | 33,468,007 | parabola_median (2,327 bp) |

Key: parabola outperforms argmax in 7/8 runs, often by 3–8 kb. See CHANGELOG v0.5.09 for full table.

---

### 5.9.6 — Prior/posterior profile similarity investigation ✗ DEFERRED (post-HMM → see BRAINSTORM.md)

Observation: prior and posterior RCN profile panels looked nearly identical after switching to
per-stage argmax aggregation. Two hypotheses: (1) biological — prior staging was already close
to APS-derived staging; (2) bug — inspector pooling all samples regardless of manifest grouping.
Action requires a dataset where prior/posterior groupings differ substantially.
Deferred until HMM integration reshapes the profile structure and groupings. See BRAINSTORM.md.

---

## Priority 5.10 — Summit output files: schemas and documentation ✓ DONE (v0.5.07)

Four summit-related output files are written by the multistage engine for each run:

---

### `_stage_summit_estimates.tsv`

One row per (amplicon × stage). Captures per-stage summit estimates before aggregation.

| Column | Meaning |
|--------|---------|
| `call_id` | Amplicon identifier (chrom:start-end) |
| `chrom` | Chromosome |
| `stage` | Developmental stage number |
| `argmax_bp` | The bin-start coordinate of the highest-RCN bin in this stage's smoothed median profile. This is the primary per-stage estimate. It is the start of a bin, not a sub-bin interpolation. |
| `argmax_bin` | The full bin interval containing `argmax_bp`, in IGV format: `chrX:start-end`. Load this into IGV to see the bin-level estimate. |
| `parabola_vertex_bp` | Sub-bin estimate: the fitted vertex of a downward-opening parabola fit to the smoothed log2 profile near the peak. NaN if the fit was invalid (not downward-opening, or vertex outside window). |
| `parabola_vertex_bin` | The bin interval containing `parabola_vertex_bp`, in IGV format. Empty if `parabola_vertex_bp` is NaN. |

---

### `_aggregate_summit_estimates.tsv`

One row per amplicon. Summaries across all stages.

| Column | Meaning |
|--------|---------|
| `call_id` | Amplicon identifier |
| `chrom` | Chromosome |
| `n_stages` | How many stages had enough data to contribute a summit estimate |
| `argmax_mean_bp` | Average of per-stage argmax positions (equal weight per stage). The canonical `final_origin_bp` in `_origins.tsv`. |
| `argmax_mean_ci_low` / `argmax_mean_ci_high` | 95% bootstrap CI for `argmax_mean_bp`. Bootstraps by resampling within each stage and recomputing the mean each time. |
| `argmax_median_bp` | Median of per-stage argmax positions. More robust to a single outlier stage. |
| `argmax_median_ci_low` / `argmax_median_ci_high` | 95% bootstrap CI for `argmax_median_bp`. |
| `parabola_mean_bp` | Average of per-stage parabola vertex positions (only counting stages where the parabola fit was valid). NaN if no valid fits. |
| `parabola_median_bp` | Median of valid per-stage parabola vertices. NaN if fewer than 1 valid fit. |
| `n_parabola_valid` | Number of stages that produced a valid parabola vertex. Useful diagnostic: high value means the peak is well-developed and symmetric across most stages; low value means the signal is weak, flat, or noisy. |

---

### `_all_summit_estimates.tsv`

One row per amplicon. Wide format — all per-stage estimates pivoted to columns, plus all aggregate columns. Designed for notebook analysis where you want to compare estimators and plot them together.

Columns: all columns from `_aggregate_summit_estimates.tsv` plus `stage_N_argmax_bp` and `stage_N_parabola_vertex_bp` for each stage N present in the data.

---

### `_summit_estimates.bed`

BED6 file containing all summit position estimates as genomic intervals. Load into IGV to visually inspect and compare estimators side-by-side with the RCN profile.

Rows included per amplicon:
- `{call_id}:stage{N}:argmax` — single-bp point at the per-stage argmax position
- `{call_id}:stage{N}:argmax_bin` — the full bin interval for the argmax
- `{call_id}:stage{N}:parabola` — single-bp parabola vertex (if valid)
- `{call_id}:stage{N}:parabola_bin` — full bin interval for the parabola vertex (if valid)
- `{call_id}:aggregate:argmax_mean` — single-bp at the equal-weighted mean argmax position
- `{call_id}:aggregate:argmax_mean_ci` — CI interval (argmax_mean_ci_low to argmax_mean_ci_high)
- `{call_id}:aggregate:argmax_median` — single-bp at the median argmax position
- `{call_id}:aggregate:argmax_median_ci` — CI interval for the median estimator
- `{call_id}:aggregate:parabola_mean` — single-bp parabola mean (if any valid stage fits)
- `{call_id}:aggregate:parabola_median` — single-bp parabola median (if any valid stage fits)

Score column: stage number for per-stage rows, 0 for aggregate rows. This allows IGV color-by-score to highlight stage identity.

Note: single-bp intervals use half-open BED coordinates (start = pos-1, end = pos). The CI intervals use (ci_low-1 to ci_high), so the interval spans the full bootstrap CI range.

---

### Design notes

- `final_origin_bp` in `_origins.tsv` uses `parabola_mean_bp` when `n_parabola_valid ≥ 1`,
  falling back to `argmax_mean_bp` (updated in Priority 5.9.7, v0.5.12). `summit_estimator_used`
  column indicates which estimator was used.
- `_stage_summit_estimates.tsv` is the primary file for per-stage exploration.
- `_aggregate_summit_estimates.tsv` is the primary file for per-amplicon summary reporting.
- `_all_summit_estimates.tsv` is designed for Jupyter notebooks.
- `_summit_estimates.bed` is designed for IGV visual inspection.

---

## Priority 5.9.7 — Switch `final_origin_bp` to parabola_mean with argmax fallback ✓ DONE (v0.5.12)

**Motivation:** The Priority 5.9.5 benchmark establishes that `parabola_mean_bp` is the best
estimator in 7/8 test cases, with median errors 3–8 kb smaller than `argmax_mean_bp`.  The
current `final_origin_bp = argmax_mean_bp` convention is a historical default, not an evidence-
based choice.

**Implementation:**
- In `_process_one_call`, set `final_origin_bp = parabola_mean_bp` when `n_parabola_valid >= 1`.
  Fall back to `argmax_mean_bp` when `n_parabola_valid == 0`.
- Add a `summit_estimator_used` column to `_origins.tsv`: `"parabola_mean"` or `"argmax_mean"`.
- Update `origin_ci_low_bp` / `origin_ci_high_bp` to reflect parabola spread (IQR or bootstrap
  across valid per-stage parabola vertices), not argmax CI.
- Update eval pass/fail thresholds in `run_summit_precision_test.sh` once CI is parabola-based.
- Update `_summit_estimates.bed` documentation to mark `parabola_mean` as the recommended estimate.

**User guidance (for `_origins.tsv` docs and output README):**
- `parabola_mean_bp` is the recommended estimate when `n_parabola_valid ≥ 1`.
- `argmax_mean_bp` is the fallback and is reported separately for comparison.
- The 5 kb bin interval (`stage_summit_estimates.tsv:argmax_bin`) is the most interpretable
  region-level report; it can clearly overlap the ground-truth zone even when the single-bp
  estimate is slightly off.
- Do not over-interpret single-bp positions from 5 kb bins — precision depends on resolution
  and data quality.  Use the bin interval as the "honest" precision, the parabola as the "best
  point estimate."

---

## Priority 5.11 — Bin partition optimization: sliding-offset sub-bin refinement ✓ DONE (v0.5.54–v0.5.56)

**Core idea:** A 5 kb bin partition is arbitrary — the true origin may fall anywhere within a
bin.  The sliding-offset approach finds the partition offset where the summit signal is best
centered, producing a more accurate bin-level estimate of the origin without requiring raw reads.

**Broader framing:** This is a *pre-processing / re-binning step*, not a parabola-specific
technique.  A shifted profile can be fed into any summit estimator: argmax, parabola, Gaussian
fit, or others not yet implemented.  The goal is to find the W-bp bin that best brackets the
true origin, regardless of which estimator is then applied to that profile.  The resulting
`sliding_offset_bp` column is a new estimator candidate in the same framework as existing ones —
it competes in the summit benchmark (`eval_summit_precision.py`) against II/9A and II/2B ground
truth alongside `argmax_mean_bp`, `parabola_mean_bp`, etc.

**Applicability:** The concept is not limited to multistage mode.  Single-file and single-stage
modes have the same bin-boundary bias problem.  Implementation should be written as a general
utility function applicable to any profile (one bedGraph, stage median, or multi-stage weighted
average).  The initial implementation will be in the multistage engine because that is where the
full summit refinement infrastructure lives, but it should be designed for portability.

**Key insight on re-binning without raw reads:**

For offset `δ`, each new "offset bin" `[s+δ, s+δ+W)` overlaps the existing fixed bins in
proportion to their intersection lengths.  New RCN = coverage-weighted mean of overlapping bins.
This can be computed entirely from the existing bin table — no raw read access required.

**Algorithm:**
1. For each amplicon, try offsets `δ ∈ {0, 100, 200, ..., W-100}` (where W = bin size).
2. For each offset, compute the new binned profile as a weighted mean interpolation.
3. Apply one or more estimators (argmax, parabola) to the shifted profile.
4. The offset where the summit bin most closely centers the peak signal is the
   "bin-partition-optimal" estimate.
5. Add `sliding_offset_bp` (argmax on best-centered shifted profile) as a new estimator column.

**Convergence check:** If the best-centered offset's bin center coincides with the parabola
vertex on the same shifted profile, that convergence is strong evidence for the true summit.
Divergence flags cases where the two estimators disagree and warrants further investigation.

**Computational cost:** O(W/step × n_stages × n_bins_per_amplicon).  Trivial at 5 kb / 100 bp step.

**Phased implementation:**
1. ~~Implement `_sliding_offset_profile` + `_refine_origin_sliding_offset` as standalone utilities.~~ **DONE (v0.5.54)**
2. ~~Integrate into multistage engine: `sliding_offset_bp` column in `_aggregate_summit_estimates.tsv`;~~ **DONE (v0.5.54)**
   ~~BED entry `{call_id}:aggregate:sliding_offset` in `_summit_estimates.bed`.~~
3. ~~Port to single-file and single-stage engines~~ **DONE (v0.5.56)** — `sliding_offset_bp`
   and `sliding_offset_hires_bp` now appear in single-engine `_origins.tsv` and `_metrics.tsv`.
   Utility functions `sliding_offset_profile` and `refine_origin_sliding_offset` live in
   `refinement.py` and are shared across all engines.
4. ~~(Phase 2) Hires-informed mode~~ **DONE (v0.5.55)** — see below.

### Hires manifest enhancement

When a hires manifest is available (e.g., 500 bp bins alongside 5 kb bins), the sliding-offset
refinement can use the hires bins directly instead of interpolating from the coarse bins:

**Without hires (default):** For each offset δ, approximate the new binned profile as a
coverage-weighted mean of the existing 5 kb bins.  This is a proportional interpolation —
accurate in expectation but limited by the information content of the 5 kb bins themselves.

**With hires:** For each offset δ, rebin directly from the 500 bp bedGraph at the candidate
offset positions.  Each new "offset bin" `[s+δ, s+δ+W)` maps to exactly 10 contiguous 500 bp
bins; RCN is their simple mean (or median).  This is exact — no interpolation error.

The hires approach is strictly better when the data are available.  It resolves sub-bin signal
structure that proportional interpolation cannot reconstruct.  The coarse-only approach remains
the default when no hires manifest is provided; hires engagement is automatic when present.

**Implementation note:** Hires bedGraphs are already loaded into the pipeline when `--hires-manifest`
is supplied.  Extending `refine_origin_sliding_offset()` to accept an optional hires track dict
(`{call_id: hires_df}`) would be the natural integration point.

### Parabola estimator portability ✓ DONE (v0.5.56)

`refine_summit_parabola` is imported and called in `single_engine.py`; `summit_parabola_bp` and
`summit_parabola_uncertainty_bp` appear in single-engine `_origins.tsv` and `_metrics.tsv`.

---

## Priority 5.12 — Robustness dataset expansion ✗ DEFERRED (post-HMM → see BRAINSTORM.md)

User-driven: requires additional datasets with validated origin coordinates.
Full design (Tier 1 replicates, Tier 2 organisms) preserved in BRAINSTORM.md [2026-04-07].
Do not add datasets without user confirmation.

---

## Priority 5.13 — Shape filter for multistage calls ✓ DONE (v0.5.66)

`_apply_multistage_shape_filter()` added to `onionskin.py`, runs after the multistage engine
completes on every run:
- **Always:** reads `_calls.tsv`, uses `delta_bic` column as shape metric, writes
  `_shape_scores_multistage.tsv` with per-call `delta_bic` and `would_reject` columns,
  reports to stderr via `log.verbose`.
- **`--shape-filter-multistage`** (default off): when enabled, removes calls with
  `delta_bic < --ms-shape-score-threshold` (default 50.0) from `_calls.tsv`, rewrites
  `_unified_onionskin_calls.bed` to match, writes rejected calls to
  `_amplicons_actively_rejected_multistage.bed`.

Toy dataset: both amplicons pass (delta_bic = 644 and 175 — well above threshold).
Threshold of 50.0 carried from single-file calibration; calibrate empirically on full runs
before enabling `--shape-filter-multistage` in production.

`make test`, `make toy`, `make single`, `make twin` — all pass.

---

## Priority 5.9.8 — Add posterior eval to the summit test suite ✓ DONE (v0.5.13–v0.5.16)

**Motivation:** The current `make summit` test evaluates only `01-prior/01-multistage-growth/04-summit_refinement/*_summit_estimates.bed`.
Posterior re-estimation (`02-posterior/`) runs the same pipeline with APS-informed groupings and
may produce tighter or more accurate summit positions, particularly for estimators that benefit from
a reduced, stage-purified subset of samples.

**Plan:**

### Step 1 — Enable posterior output in DS2 hires run

Modify `_run_pipeline_hires_bg` in `run_summit_precision_test.sh` (or add a dedicated variant) to
pass `--compute-aps` and any posterior trigger flags to `onionskin.py`.  Check current CLI for the
flags that activate posterior re-estimation.

### Step 2 — Discover posterior summit BED

Extend `_eval_one` (or add `_eval_one_posterior`) to look for:
```
$DATASET_OUT/02-posterior/01-multistage-growth/04-summit_refinement/*_summit_estimates.bed
```
alongside the existing `01-prior/01-multistage-growth/04-summit_refinement/*_summit_estimates.bed`.

### Step 3 — Extend `eval_summit_precision.py` to accept `--prior-summit-bed` / `--posterior-summit-bed`

Currently only `--summit-bed` is supported.  Add:
- `--prior-summit-bed` — path to prior `_summit_estimates.bed`
- `--posterior-summit-bed` — path to posterior `_summit_estimates.bed`
- When both are supplied, run `eval_II9A_multiestimator` for each, prepend `prior_` / `posterior_`
  to estimator labels in the summary JSON

### Step 4 — Extend `consolidate_summit_reports.py`

Add a "Prior vs Posterior Comparison" section to the full-run markdown, showing:
- For each run: best prior estimator vs best posterior estimator, ORC-Core distances for each
- Whether posterior is better, equal, or worse (and for which estimator types)

### Step 5 — Test and document

Run `make summit-smoke` first (DS2 hires only), verify posterior BED is discovered and evaluated,
compare results.  If posterior consistently improves accuracy, note that in ROADMAP 5.9.7 guidance
and in the user-facing output README.

**Expected outcome:** Posterior eval surfaces systematically in the benchmark, allowing us to
quantify how much APS-guided grouping improves (or does not improve) summit placement.

---

## Phase 5 closure note (v0.5.66)

All core Phase 5 priorities are done or formally closed. Deferred items moved to BRAINSTORM.md:
- 5.0.2 (timing.py refined summit fold-change), 5.3 (APS order stability), 5.4 (posterior
  ordering), 5.9.3 (timing-guided stage weights), 5.9.4 (summit↔timing EM convergence),
  5.9.6 (prior/posterior profile similarity), 5.12 (dataset expansion) — all post-HMM.

---

# Phase 6 — APS feature evolution

## Goals
Move beyond one APS value per amplicon.

---

## Priority 6.1 — Bin-wise APS feature vectors ("shape" clustering) ✓ DONE (v0.3.32)

`--aps-feature shape` is implemented in `build_aps_shape_matrix()`. Each bin's RCN-1 excess is a feature; bins are weighted by `1/√n_bins_in_locus` so every locus contributes equally (in mean-squared-difference terms) to Euclidean distance regardless of width. The correct weight is `1/√n` (not `1/n`): setting `w² × Σ(x−y)² = (1/n) × Σ(x−y)²` yields `w = 1/√n`. Exposed via `--aps-shape-no-normalize` to disable weighting. Shape matrices written to `aps_matrix_shape_raw.tsv` / `aps_matrix_shape_zscore.tsv`.

*(Original design notes below for reference.)*

### Original design
Instead of representing each amplicon by one summary score, represent each sample by all bins within all final amplicons. Exposed as `--aps-feature shape` (alongside `area`, `summit`, `width`). Per-amplicon normalized feature blocks preferred — equalizes amplicon influence.

---

## Priority 6.2 — Dimensionality reduction / embedding ✗ DEFERRED (post-HMM → see BRAINSTORM.md)

PCA, UMAP, diffusion-like developmental embeddings. Better visualization of posterior trajectories.
Deferred until HMM integration (Phase 7) stabilizes the pipeline architecture.

---

## Priority 6.3 — Shape-aware clustering ✗ DEFERRED (post-HMM → see BRAINSTORM.md)

Cluster using amplicon morphology, not just integrated magnitude. Post-HMM.

---

## Phase 6 closure note (v0.5.66)

Priority 6.1 done. Priorities 6.2 and 6.3 deferred post-HMM to BRAINSTORM.md. Phase 6 is closed.

---

# Phase 6.5 — Code Unification Refactor ✓ DONE (v0.5.57–v0.5.62)

Consolidated duplicated code across all three engines into shared modules.
Full plan and design notes archived in ROADSTRAVELED.md and
`multi-agent/plans/archived/20260407-CODE-UNIFICATION-REFACTOR.md`.

- Priority 6.5.A ✓ DONE (v0.5.57) — Break circular import chain (`common.py` lazy imports)
- Priority 6.5.B ✓ DONE (v0.5.58) — Create `signal_utils.py` (canonical signal utilities)
- Priority 6.5.C ✓ DONE (v0.5.59) — Expand `detection.py` (shared detection algorithms)
- Priority 6.5.D ✓ DONE (v0.5.60) — Expand `refinement.py` (scoring + manifest canonicalization)
- Priority 6.5.E.0 ✓ DONE (v0.5.61) — Output directory numbered-prefix scheme
- Priority 6.5.E ✓ DONE (v0.5.62) — Per-stage mean-shift pipeline (`02-per-stage-mean-shift/`)
- Priority 6.5.E.2 ✓ DONE (v0.5.63) — Hires summit refinement + parallelization for per-stage

---

# Phase 7 — HMM integration ✓ SUBSTANTIALLY COMPLETE (v0.7.20)

## Goals
Integrate PuffStep's HMM pipeline into onionskin as a first-class engine. All 10 pipeline
steps now run natively in Python by default. PuffStep is not required at runtime for the
default configuration. See `multi-agent/plans/PUFFSTEP-INTEGRATION-PLAN.md` for full detail.

**The broader vision:** Onionskin will fully replace PuffStep. Phase 7 achieves parity with
PuffStep's validated default. Phase 8 completes full feature parity. Phase 9 integrates HMM
evidence into onionskin's other modules.

See BRAINSTORM.md [2026-04-07] for the three-pipeline architecture vision and `04-unified-results`.

---

## Priority 7.1 — Unified RCN and smoothed-RCN processing ◑ PARTIAL (v0.7.08)

Before combining puffstep and onionskin evidence, both pipelines must agree on signal.
Onionskin uses ±3-bin running median; puffstep may use a different smoothing approach.
Differences in smoothing affect summit calls, APS_summit, and HMM emission computation.

**Plan:**
1. Run both pipelines on the same dataset; overlay RCN/smoothed-RCN in the summit inspector
2. Identify normalization, bin boundary, and smoothing kernel discrepancies
3. Converge on a shared preprocessing spec; consider a single shared preprocessing module

**Implemented so far:**
- Native preprocessing ports in `hmm_engine.py` + `signal_utils.py` for steps 1/2/3/4/5/7/8.
- Parity gate repeatedly validated with `make puff-compare` (24 match, 4 skipped).
- Remaining preprocessing semantics to finalize are concentrated in summit extraction
  behavior (steps 9/10), especially boundary/point-like interval edge cases.

---

## Priority 7.2 — HMM as optional refinement/validation layer ◑ PARTIAL (v0.7.01–v0.7.11)

Add HMM as an optional mode running puffstep after detection and refinement. Use HMM
summit-state intervals to constrain or validate `final_origin_bp`. Expose HMM output tracks
alongside onionskin tracks in the standard output layout.

Depends on Priority 7.1 (shared signal) being complete.

**Implemented so far:**
- HMM engine is wired as an optional pipeline and runs in standard output layout.
- Steps 1-10 run natively by default.
- Step-6 fallback backend remains available as env-only rollback/debug path:
  `ONIONSKIN_HMM_STEP6_BACKEND=puffstep`.
- Validation workflow (`make puffstep-py` + `make puff-compare`) is established as
  the integration gate for every migration step.

---

## Priority 7.3 — HMM-derived metrics

Candidate outputs: per-sample HMM state paths, per-amplicon doubling boundaries, inferred fork
distances by round, “recent round extent” summaries. May later become alternative APS features
or additional posterior-ordering features.

---

## Priority 7.4 — Three-pipeline architecture and `01-multistage-growth` refactor

After HMM integration, refactor output layout so the three pipelines (multistage growth,
per-stage mean-shift, HMM) each occupy their own numbered directory, with `04-unified-results`
as the downstream synthesis layer. Unified results aggregate timing, APS, fork gradient, etc.
across whichever pipelines were run — single-pipeline runs use only that pipeline's outputs.

Target grouped layout after the refactor:

```
<out_dir>/
  01-prior/
    01-multistage-growth/
    02-per-stage-mean-shift/
    03-hmm/
    04-unified-results/
  02-posterior/
    (mirrors 01-prior structure when posterior regrouping is run)
```

The grouped-layout refactor has now landed: multistage outputs live under
`01-multistage-growth/`, and HMM outputs live under grouped `03-hmm/` inside both
`01-prior/` and `02-posterior/` when those pipeline arms are run.

See BRAINSTORM.md [2026-04-07] for detailed architecture notes.

---

## Priority 7.4.4 — HMM tunability surface expansion ✓ COMPLETE (v0.7.15–v0.7.18)

**Authors:** John M. Urban, GitHub Copilot (GPT-5.3-Codex)

- Tier 1 complete (v0.7.16): `--hmm-special-idx`, `--hmm-init-special`, `--hmm-leave-special-state`, `--hmm-leave-other`, `--hmm-exp-decay-scale`
- Tier 2 complete (v0.7.17): `--hmm-mu-scale`, `--hmm-initialprobs`
- Tier 3 CLI surfaced (v0.7.18): `--hmm-emodel`, `--hmm-kmeans`, `--hmm-iters`, `--hmm-converge`, `--hmm-constrain-emit`, `--hmm-emitpseudo`; native backend blocks non-default Tier 3 (requires `ONIONSKIN_HMM_STEP6_BACKEND=puffstep`)
- **Decision (post-v0.7.20):** Tier 3 native implementation is Phase 8.

---

## Priority 7.4.5 — Parity-first cleanup ✓ COMPLETE (v0.7.22)

1. **pybedtools replaced** (v0.7.22): `onionskin_core/hmm_summits.py` — pure-Python port of PuffStep findsummits. Steps 9/10 use it; `signal_utils.py` has no pybedtools.
2. **Protocol-31 sort fixed** (v0.7.22): explicit sort in `chromosome_ratio_normalize_bedgraph`.
3. Validated: `make puff-compare` OVERALL PASS (24 match, 4 skipped).

---

# Phase 8 — Full PuffStep feature parity

**Goal:** Onionskin implements everything PuffStep's CLI offers natively. PuffStep is not
required at runtime for any user-requested configuration.

## Priority 8.1 — Tier 3 HMM tunables: native implementation ✓ COMPLETE

**8.1a ✓ COMPLETE (v0.7.23):** Alternative emission models native in `hmm_core.py`.
- `SUPPORTED_EMODELS = {normal, exponential, poisson, geometric, gamma}`
- `_log_emit_seq()` dispatch; `_viterbi`, `_forward`, `_backward`, `_posterior_path` generalized.
- `--hmm-emodel` fully functional.

**8.1b ✓ COMPLETE (v0.7.25):** K-means initialization native.
- `_kmeans_init_emissions(data, nstates)` in `hmm_core.py` using `scipy.cluster.vq`.
- `--hmm-kmeans` wired end-to-end; overrides emission_means/sigmas from data.

**8.1c ✓ COMPLETE (v0.7.26):** Iterative parameter re-estimation native.
- Viterbi training loop: `_log10_prob_path`, `_update_eprobs`, `_update_tprobs`, `_update_iprobs`.
- `--hmm-iters`, `--hmm-converge`, `--hmm-constrain-emit`, `--hmm-emitpseudo` fully native.
- This is Viterbi training (hard EM), matching PuffStep's `do_hmm_iter_steps()`.

**8.1d ✓ COMPLETE (v0.7.27):** True Baum-Welch (soft EM) — new beyond PuffStep.
- PuffStep used Viterbi training only; onionskin now also supports true Baum-Welch.
- `_forward_loglik`, `_bw_estep_one`, `_bw_mstep_*`, `_baum_welch_train` in `hmm_core.py`.
- `--hmm-training viterbi|baum_welch` CLI flag. Default: `viterbi` (PuffStep-compatible).
- Convergence tracked on total log-likelihood (sum of log scale factors = exact log P(obs|model)).
- In practice on broad amplification domains, BW and Viterbi training produce nearly identical
  results. BW is theoretically optimal; Viterbi training is faster.

## Priority 8.2 — Posterior decoding CLI ✓ COMPLETE (v0.7.24)
`--hmm-decode-path viterbi|posterior` added. Wired through `run_hmm()` → `_step6_hmm()` → `write_hmm_states_bedgraph`. Default: `viterbi`.

## Priority 8.3 — Signal utilities expansion ✓ COMPLETE (v0.8.00)

Additional normalization protocols ported from PuffStep and a new signal transforms module.
All opt-in; no default behavior change.

**`onionskin_core/signal_utils.py` additions:**
- `local_median_normalize_bedgraph` (p30) — divide by local-window median (play_it_safe fallback)
- `local_mean_smooth_bedgraph` (p34) — replace with local-window mean
- `local_trimmed_mean_smooth_bedgraph` (p33) — replace with trimmed window mean
- `spmr_normalize_bedgraph` (p22) — signal per million reads (divide by total × 1e6)
- `robust_z_normalize_bedgraph` (p18) — `(x - median) / MAD`
- `rank_normalize_bedgraph` (p19) — fractional rank
- `rank_standardize_bedgraph` (p23) — `(rank - midpoint) / midpoint`
- `_safe_local_median`, `_local_window_apply` — shared internal helpers

**`onionskin_core/signal_transforms.py` (new module):**
Biological signal transforms (not normalization — these change the representation):
- `compute_skew_bedgraph` — RFD: `(V[i]-V[i-1])/(V[i]+V[i-1])*100`
- `compute_percent_change_bedgraph` — `(V[i]-V[i-1])/V[i-1]*100`
- `compute_skew_change_bedgraph` — derivative of RFD/200; positive peaks = origins
- `log2_bedgraph`, `log10_bedgraph`
- `ratio_bedgraph`, `subtract_bedgraph`, `pct_diff_bedgraph`, `pct_skew_bedgraph`

**Kernel smoothing (p2/3/4/5/6/13/14/15) ✓ COMPLETE (v0.8.01):**
`gaussian_smooth_bedgraph(input, output, bandwidth)` in `signal_utils.py`. Bandwidth in bp
(same units as PuffStep's `--bandwidth` flag; default 10000 bp). Converts to bin-index σ via
`sigma = bandwidth / bin_size`. Uses `scipy.ndimage.gaussian_filter1d(mode="reflect")`.
All 8 protocols are now composable from existing primitives.

**Explicitly skipped (final decisions — do not revisit):**
- p16 (glocal median ratio norm): marked `INDEV` in PuffStep source; not production-ready.
- p17 (median norm + scale-to-target-coverage + ratio + re-median norm): obscure pipeline;
  `scale_data` primitive not needed elsewhere; not part of any standard workflow.

## Priority 8.4 — Retire PuffStep runtime dependency + full CLI parity ✓ COMPLETE (v0.8.03)
- Removed `_find_puffstep_root()`, `_puffstep_tools()`, `ONIONSKIN_HMM_STEP6_BACKEND` env-var,
  `step6_backend` param, and the entire PuffStep subprocess block from `hmm_engine.py`.
- `--hmm-exp-decay on|off` (default: on): added `_get_transition_probs_uniform()` for off path.
- `--hmm-transprobs MATRIX`: full custom transition matrix; overrides leave_* and exp_decay.
- `--hmm-learn-pseudo FLOAT`: transition pseudocount for Viterbi training. Port of PuffStep --learnpseudo.
- `make puff-compare` OVERALL PASS. No PuffStep runtime dependency anywhere.

---

# Phase 9 — HMM as standalone pipeline

**Goal:** Treat the HMM result set as a self-sufficient pipeline — the PuffStep successor
mode. When the user runs `--pipeline hmm`, every analysis onionskin performs for multistage
growth mode is also available from HMM results alone. Phase 9 also adds HMM-specific analyses
that the state path uniquely enables: fork travel trajectories, nesting-level geometry,
origin stability, and multi-stage-aware filtering. The HMM output directory will mirror the
multistage growth directory structure plus all new HMM-native analyses.

**Design principle:** The nested HMM state path directly encodes the cumulative replication
fork displacement per doubling round. State-path step-down boundaries, read relative to the
estimated origin/summit position, give left and right fork travel distances for each nesting
level. Cross-stage tracking turns per-stage snapshots into a developmental trajectory for
each amplicon.

---

## Priority 9.1 — HMM-derived replication metrics (per amplicon, per stage)

Extract the fundamental geometric quantities from each amplicon's state path:

- **Nesting-level inventory:** enumerate all distinct state levels (background → summit state)
  and their boundary coordinates.
- **Doubling boundaries:** left and right boundary positions for each nesting level.
- **Active-window estimate:** span of states > background (outermost level boundary pair).
- **Summit state interval:** coordinates of the highest-state interval; use as alternative
  `final_origin_bp` and as tighter parabola-fitting window for summit position estimation.
- **State-path complexity score:** number of distinct nesting levels per amplicon per stage.
  Single number summarizing developmental depth captured in that stage.
- **Origin position estimate:** center of summit state interval; used as anchor for all fork
  travel calculations. Multiple estimation strategies supported (center, parabola fit within
  summit interval, etc.).

Output: per-amplicon per-stage metrics table analogous to multistage growth outputs.

---

## Priority 9.2 — Fork travel and asymmetry (per amplicon, per stage, cross-stage)

**Per-stage fork travel (single amplicon, single stage):**
For each nesting level L (from summit/innermost down to background/outermost):
- Left fork travel = origin position − left boundary of level L
- Right fork travel = right boundary of level L − origin position
- Total domain length = right boundary − left boundary
- Left/right asymmetry = right travel − left travel (signed); |asymmetry| / total = asymmetry
  fraction
- **Incremental travel** (for levels below the innermost): additional left/right distance
  traveled relative to the level immediately above. Proxy for time between re-replication
  rounds if fork speed is constant.

**Cross-stage fork travel trajectory (per amplicon, all stages):**
For each set of replication forks (identified by nesting level), track left and right travel
distances across all stages in which they are detectable:
- Map fork positions per stage relative to the origin (negative = leftward, positive =
  rightward)
- Build a per-amplicon, per-fork-set trajectory: stage × fork position table
- Detect level emergence (which stage a nesting level first appears = when that re-replication
  round began)
- Detect fork travel growth: does a given fork set continue traveling between stages?
- Flag "ghost levels": a nesting level present in stage N but absent in stage N+1 (model
  artifact vs. biology)

**Flagship visualization — fork travel trajectory plot:**
- Y-axis: stages (stage 1 top to stage N bottom)
- X-axis: genomic position relative to origin (0 = origin; negative = left; positive = right)
- Each fork set plotted in a distinct color; left/right fork positions as points or error bars
- Variant: both forks plotted as positive distances, different symbols, so left/right distance
  from origin can be visually compared per stage

**Additional visualizations:**
- Asymmetry scatter: left fork travel vs. right fork travel per level per stage, all amplicons.
  Diagonal = symmetric; deviation = directional bias.
- Nested domain diagram: stacked nested rectangles per amplicon per stage; each level a
  different color. Shows nesting structure at a glance.
- Level emergence heatmap: amplicons × stages, colored by number of active nesting levels.
  Developmental progression at a glance.
- State path browser: per amplicon, raw RCN + smoothed RCN + state path, all stages stacked.
  Mini genome browser for the amplicon.
- Replication pulse width per level: domain length at each nesting level plotted across stages.
  Shows how wide each replication pulse was that round.

---

## Priority 9.3 — Multi-stage-aware filtering and summit refinement

**Key principle:** The full multi-stage trajectory is the unit of evidence, not any single
stage in isolation. Width and complexity filters must be applied after examining all stages.

**Multi-stage-aware filtering:**
- Collect per-amplicon, per-stage state path evidence before applying any width or complexity
  filter.
- An amplicon that has only a single doubling (oldest forks only, ≤2× signal) in an early
  stage should NOT be filtered on width alone if later stages show expected widening from
  fork travel growth, or if nested levels appear in later stages.
- An amplicon seen only in a single stage and never growing is a weaker candidate; width
  filter applies there.
- "Multi-stage growth" evidence within the HMM pipeline (nested levels growing or fork
  travel increasing across stages) is a high-confidence tier, analogous to the independent
  evidence from multistage growth mode.
- The step-8 width and state filters (`--hmm-min-width`, `--hmm-max-state-thresh`, etc.) 
  should ultimately be evaluated in this multi-stage context, not applied greedily per stage.

**Summit refinement across stages:**
- Permissive summit: union of summit-state intervals across all stages (merge across stages).
- High-confidence summit estimates:
  1. Narrowest summit-state interval across all stages.
  2. Coverage-based: compute "coverage depth" of summit-state intervals across stages;
     central region with highest cross-stage coverage.
  3. Combination: intersection of methods 1 and 2.
- Use summit-state interval as the fitting window for parabola-based summit estimation;
  provides tighter biological constraints vs. using the full amplicon window.
- Origin stability metric: does the inferred summit position shift between stages? Shift
  may indicate noisy estimation or genuine origin zone shift (biology).

---

## Priority 9.4 — Multistage growth analyses ported to HMM pipeline

Adapt the full multistage growth output directory for the HMM pipeline. The HMM directory
should mirror the multistage growth directory plus HMM-native analyses.

**Direct ports (same analysis, HMM-defined amplicon regions):**
- APS computation using HMM-derived amplicon boundaries and summit estimates
- Timing computation using HMM state-path boundary-constrained fork gradient
- All summit estimation strategies applied within HMM-defined amplicon coordinates
- Clustering using RCN signal within HMM amplicon windows (same methods as multistage growth)
- Sample comparison and grouping using HMM-derived metrics

**HMM-native extensions:**
- Clustering on state path profiles directly (not just RCN signal):
  state-path vectors as features for clustering/dimensionality reduction
- APS and timing weights informed by per-level fork travel distances rather than just signal
  gradient
- Level-based APS: compute APS per nesting level, not just per amplicon

**Jupyter notebooks:**
- **Fork travel explorer:** one amplicon, all stages, interactive origin position slider,
  all fork sets with level colors, fork travel trajectory plot.
- **HMM state path QC:** RCN signal + state path + expected nested structure per amplicon;
  flags suspicious HMM fits (single-bin state islands, inverted nesting, ghost levels).
- **Amplicon atlas:** all amplicons × all stages: state path heatmaps, fork travel summary,
  clustering dendrograms in one notebook.
- **Parameter characterization (pre-optimization):** RCN distribution properties (background
  level, signal-to-noise, dynamic range) vs. HMM parameters that worked on gold-standard
  data. Diagnostic notebook for new datasets before parameter tuning.

---

## Priority 9.5 — All-stage HMM

Deferred out of Phase 9. The remaining meaningful "all-stage HMM" questions now depend on
cross-pipeline contracts and shared file/filter semantics, so they belong after the Phase 10
unification work rather than inside standalone-HMM closeout.

---

## Priority 9.6 — Parameter optimization (`--hmm-optimize`)

Deferred until the post-unification phase. `--hmm-optimize` should not be designed or
implemented on top of moving preprocessing, file-ownership, and pipeline-composition
contracts.

- Characterize relationships between gold-standard RCN distributions and the HMM parameters
  that worked (ground from Priority 9.4 parameter characterization notebook).
- New data will have different RCN distributions; differences should guide parameter tweaks
  (e.g., log2-transformed RCN may require different emission means/sigmas).
- BIC/cross-validation scoring within nested replication bubble constraints.
- `--hmm-optimize` CLI flag; routine runs as a pre-analysis diagnostic and proposes updated
  defaults.

---

# Phase 10 — Cleanup closeout and de-entangling

**Goal:** Close the old Phase 10 local optimum cleanly by removing deprecated shared-output
ownership assumptions, preserving the valid 10.1/10.2 contract work, and leaving the repo ready
for a forward architecture phase. Phase 10 is cleanup only: migration, ownership correction, and
removal of deprecated structures are allowed; new analysis capabilities, completeness work, and
forward-facing features are out of scope.

---

## Priority 10.1 — Cross-pipeline contract audit ✓ DONE (v0.10.00)

**Authors:** John M. Urban, GitHub Copilot (GPT-5.4)

Audited the controller and engine contract surface. The concrete findings were: duplicated
HMM-owned sequence-scan/filter helpers, mixed multistage+HMM runs not sharing one effective
filtered sequence universe, a duplicate posterior HMM invocation that could fall back to the
wrong manifest, and an unresolved `--pipelines all` composition decision that remains deferred
to Priority 10.5.

---

## Priority 10.2 — Shared filtering semantics ✓ DONE (v0.10.20)

**Authors:** John M. Urban, GitHub Copilot (GPT-5.4)

Shared filtering semantics are now explicitly controller-owned for `--chromosomes`, `--min-seq-length`, and `--min-bin-count-per-seq`, so one requested input universe is chosen and shared across participating pipelines. The nearby controller/manifest/normalization side quests discovered while landing that contract have also been closed. Later pipeline-local APS/posterior completeness and cross-pipeline synthesis questions were explicitly reclassified into Priorities 10.4 and 10.5 rather than kept as 10.2 spillover.

---

## Priority 10.3 — Planning-surface invalidation cleanup ✓ DONE (v0.10.24)

**Authors:** John M. Urban, GitHub Copilot (GPT-5.4)

Remove the invalid active planning assumptions inherited from the deprecated shared-output
architecture. This priority rewrites the live planning surface so pipeline-derived outputs are no
longer treated as candidates for shared ownership above pipelines.

The active planning/doc surface is now aligned to the cleanup-only contract: `00-signal-files`
is deprecated, not retained architecture, and not a prerequisite for later completeness.

---

## Priority 10.4 — Deprecated-structure inventory and disposition ✓ DONE (v0.10.24)

**Authors:** John M. Urban, GitHub Copilot (GPT-5.4)

Enumerate every currently emitted deprecated shared-output artifact and assign each one exactly
one disposition: move to owning pipeline, move to controller/run-level metadata, or remove
entirely.

The authoritative inventory is now closed:
- `00-signal-files/samples/` → move to `01-multistage-growth/05-aps/samples/`
- `00-signal-files/genome_stage_medians/` → move to `01-multistage-growth/05-aps/genome_stage_medians/`
- `00-signal-files/filtered_inputs/` → remove entirely; HMM filtering is now applied directly during step 1

---

## Priority 10.5 — Ownership correction and migration ✓ DONE (v0.10.24)

**Authors:** John M. Urban, GitHub Copilot (GPT-5.4)

Execute the migration and removal work required by the new ownership contract: move pipeline-
derived outputs back to the pipeline that computes them, move retained analysis-neutral metadata
to controller-owned locations, and remove deprecated owner/view patterns.

APS-local track ownership is now corrected under `01-multistage-growth/05-aps/`, and the HMM
filtered-input owner/view pattern has been removed rather than relocated.

---

## Priority 10.6 — Cleanup closeout gate ✓ DONE (v0.10.25)

**Authors:** John M. Urban, GitHub Copilot (GPT-5.4)

Close Phase 10 only after deprecated shared-output assumptions are absent from planning,
queueing, and active architecture, and after the next implementation path clearly points into
Phase 11 rather than back into cleanup-only work.

The Phase 10 closeout gate is now satisfied. The remaining active path is Phase 11 forward
architecture, beginning with pipeline-local ownership architecture and shared-module extraction.

---

# Phase 11 — Forward architecture

**Goal:** Build the forward architecture around exactly four layers: pipeline-local outputs,
shared code/modules, controller/run-level metadata, and future explicit cross-pipeline
synthesis. This phase is where completeness and forward implementation resume.

---

## Priority 11.1 — Pipeline-local ownership architecture ✓ DONE (v0.10.32)

**Authors:** John M. Urban, GitHub Copilot (GPT-5.4)

Define and enforce pipeline-local emitted ownership for `growth`, `per-stage`, and `hmm`, with
pipeline-derived outputs emitted only under the subtree of the pipeline that computes them and
the remaining supported composition/capability surface defined against that ownership model.

- Growth-owned prior/posterior outputs now emit directly under grouped `01-multistage-growth/` roots instead of writing flat files and relocating them later.
- The grouped growth subtree now owns its secondary families directly under `01-stage_median_within_calls/`, `02-summary_bedgraphs/`, `03-summits/`, and `04-summit_refinement/`.
- Fresh grouping roots are now controller-only surfaces rather than mixed controller-plus-growth output surfaces.

---

## Priority 11.2 — Shared module extraction

**Authors:** John M. Urban, GitHub Copilot (GPT-5.4)

Extract duplicated computation into shared modules without creating any shared emitted-analysis
layer above pipelines.

---

## Priority 11.3 — HMM completeness

**Authors:** John M. Urban, GitHub Copilot (GPT-5.4)

Make HMM the first major forward implementation priority by completing the missing HMM analysis
families inside the HMM pipeline and reusing shared modules where appropriate.

---

## Priority 11.4 — Per-stage completeness

**Authors:** John M. Urban, GitHub Copilot (GPT-5.4)

Extend the same pipeline-local completeness standard to the `per-stage` pipeline.

---

## Priority 11.5 — Growth peer-pipeline cleanup

**Authors:** John M. Urban, GitHub Copilot (GPT-5.4)

Remove residual architectural special status from `growth` and align it fully with the
peer-pipeline ownership model.

---

## Priority 11.6 — Explicit synthesis boundary

**Authors:** John M. Urban, GitHub Copilot (GPT-5.4)

Define the future synthesis interface above pipeline-local outputs without approximating
synthesis through centralized intermediate files.

---

# Phase 12 — HMM step layout migration + pipeline execution order ✓ DONE (v0.12.25)

**Closed:** 2026-04-16. SPEC archived at
`multi-agent/plans/archived/20260416-PHASE12_SPEC.md`; BRAINSTORM at
`multi-agent/plans/archived/20260416-PHASE12_BRAINSTORM.md`.

Restructured pipeline architecture: HMM step dictionary moved into the
canonical `output_layout.py`, growth pipeline expanded to a 15-step layout,
and pipeline execution order standardized as HMM → Growth → Per-Stage. Plus
an investigation-only audit (12.4) of HMM-smoothed-RCN reuse across pipelines
and a compatibility-matrix audit.

## Priority 12.1 — HMM steps dict migration to `output_layout.py` ✓ DONE (v0.12.25)
## Priority 12.2 — Growth pipeline 15-step layout ✓ DONE (v0.12.25)
## Priority 12.3 — Pipeline execution order (HMM → Growth → Per-Stage) ✓ DONE (v0.12.25)
## Priority 12.4 — HMM smoothed RCN reuse feasibility audit ✓ DONE (v0.12.25)
## Priority 12.4 (investigation-only) — Compatibility Matrix ✓ DONE (v0.12.25)

---

# Phase 13 — Per-stage completeness + posterior infrastructure ✓ DONE (v0.13.80)

**Closed:** 2026-04-21 (final wrap-up at v0.13.81). SPEC archived at
`multi-agent/plans/archived/20260420-PHASE13_SPEC.md`; BRAINSTORM at
`multi-agent/plans/archived/20260417-PHASE13_BRAINSTORM.md`.

The per-stage / RCN-mean-shift pipeline received its completeness pass:
14-step layout adoption, summit refinement (parabola estimator + II/9A
diagnostic), gap analysis + amplicon metrics, per-sample normalization +
APS + clustering, plots + notebooks + QC, and posterior groups + ordering
+ manifests. Plus HMM prettification + name harmonization carried over.

## Priority 13.1 — Rename + 14-step layout + unified controller + doc/guidance cleanup ✓ DONE (v0.13.80)
## Priority 13.2 — HMM prettification + name harmonization ✓ DONE (v0.13.80)
## Priority 13.3 — Summit refinement (parabola estimator) + II/9A diagnostic ✓ DONE (v0.13.80)
## Priority 13.4 — Gap analysis + amplicon metrics ✓ DONE (v0.13.80)
## Priority 13.5 — Per-sample normalization + APS + clustering ✓ DONE (v0.13.80)
## Priority 13.6 — Plots ✓ DONE (v0.13.80)
## Priority 13.7 — rcn-mean-shift notebooks ✓ DONE (v0.13.80)
## Priority 13.8 — Per-stage QC / parameter tuning ✓ DONE (v0.13.80)
## Priority 13.9 — Posterior groups, ordering, manifests ✓ DONE (v0.13.80)

---

# Phase 14 — CLI cleanup + parser overhaul ✓ DONE (v0.14.46)

**Closed:** 2026-04-22. SPEC archived at
`multi-agent/plans/archived/20260422-PHASE14_SPEC.md`; BRAINSTORM at
`multi-agent/plans/archived/20260421-PHASE14_BRAINSTORM.md`; FEEDBACK at
`multi-agent/plans/archived/20260421-PHASE14_FEEDBACK.md`.

Parser section structure overhaul + atomic flag-rename pass with `args.*`
call-site updates + `--norm-mode` behavior change + deprecation error
catches + `--pipelines rms` synonym + engine-CLI crosswalk + test/scripts/
docs migration + AGENT_CONVENTIONS CLI flag conventions rule. Established
the canonical CLI surface that Phase 14 Supplemental then refined further.

## Priority 14.1 — Parser section structure overhaul ✓ DONE (v0.14.46)
## Priority 14.2 — Flag renames + `args.*` call-site updates (atomic pass) ✓ DONE (v0.14.46)
## Priority 14.3 — `--norm-mode` behavior change ✓ DONE (v0.14.46)
## Priority 14.4 — Deprecation error catches ✓ DONE (v0.14.46)
## Priority 14.5 — `--pipelines rms` synonym in the main CLI ✓ DONE (v0.14.46)
## Priority 14.6 — Engine CLI crosswalk notes ✓ DONE (v0.14.46)
## Priority 14.7 — Test suite updates ✓ DONE (v0.14.46)
## Priority 14.8 — Scripts and auxiliary tools ✓ DONE (v0.14.46)
## Priority 14.9 — Developer-helper surface naming ✓ DONE (v0.14.46)
## Priority 14.10 — Live documentation migration ✓ DONE (v0.14.46)
## Priority 14.11 — AGENT_CONVENTIONS.md CLI flag conventions rule ✓ DONE (v0.14.46)

---

# Phase 14 Supplemental — Comprehensive CLI quality pass ✓ DONE (v0.14.75)

**Closed:** 2026-04-26 (post-wrap-up remediation closeout at v0.14.75). SPEC
archived at `multi-agent/plans/archived/20260426-PHASE14_SUPPLEMENTAL-SPEC.md`;
sibling files (`-AUDIT_LOG.md`, `-STRATEGY.md`, `-FEEDBACK.md`,
`-SPEC.UNORDERED.md`) at the same `20260426-PHASE14_SUPPLEMENTAL-*.md`
pattern.

Full CLI quality pass picking up where Phase 14 left off: 30 priorities
(14-S1 through 14-S30) covering naming-debt cleanup, Universal-promotion
of shared concepts, step-mention help-string coverage, placeholder-flag
clarification, HMM help improvements, structural polish + flag ordering,
the standalone-engine-CLI architectural decision, the
`validate_onionskin_flags.py` developer tool, and a closeout doc sweep
(tracking-directory reorganization + `AGENT_CONVENTIONS` CLI conventions
codification + agent-file drift audit). Organized into 7 execution cycles
(14S.1a through 14S.5a) plus a Final Overseer wrap-up audit (two passes:
Gemini 3.1 Pro Preview first; Opus second) plus a post-wrap-up remediation
cycle.

This was also the first phase fully run under the v2 audit-implement-reaudit
workflow (mid-stream migrated from v1 at v0.14.64), introducing sibling
`AUDIT_LOG.md` files, per-cycle CHANGELOG batching, the substantive-
priorities rule, the cycle-granularity distinction, the
phase-archive-after-Final-Overseer rule (codified in DEVLOG v0.14.73.1),
and the conversational-restoration of in-situ audit + push-back affordances
across role descriptions and Templates B/C/F/G (DEVLOG v0.14.75.1).

## Priorities 14-S1, 14-S2, 14-S3 (Phase 0 — historical) ✓ DONE (v0.14.49)
## Priorities 14-S10, 14-S4, 14-S5, 14-S29, 14-S30 (Phase 1 — mechanical) ✓ DONE (v0.14.68)
## Priorities 14-S27, 14-S28, 14-S22, 14-S23, 14-S26 (Phase 1 — substantive) ✓ DONE (v0.14.70)
## Priorities 14-S12, 14-S25, 14-S14, 14-S15, 14-S21 (Phase 2 — audit-only) ✓ DONE (v0.14.70.3)
## Priorities 14-S6, 14-S7, 14-S9, 14-S13, 14-S19, 14-S20 (Phase 3 — help-string polish) ✓ DONE (v0.14.71)
## Priority 14-S8 (Phase 3 — full cross-group help-string quality pass) ✓ DONE (v0.14.72)
## Priorities 14-S11, 14-S24 (Phase 4 — structural polish + tooling) ✓ DONE (v0.14.73)
## Priorities 14-S16, 14-S17 (folded), 14-S18 (resolved inline) (Phase 5 — closeout) ✓ DONE (v0.14.74)
## Post-wrap-up remediation cycle (Findings A+B+C+Gemini-1) ✓ DONE (v0.14.75)

---

# Phase 15 — HMM completeness + cross-pipeline enrichment ✓ DONE (v0.14.97)

**Closed:** 2026-05-05 (cycles 15.10a + 15.10a-S1 + 15.10a-S2 closure sequence at v0.14.95 / v0.14.96 / v0.14.97). SPEC + AUDIT_LOG + SURPRISE_LOG + STRATEGY + FEEDBACK + BRAINSTORM + Final Overseer Pass 1 + Pass 2 reports all archive as `multi-agent/plans/archived/2026MMDD-PHASE15_*.md` post-Final-Overseer wrap-up.

Phase 15 spanned 19 cycle-rounds (15.1a, 15.2a, 15.3a, 15.4a + 4 supplemental cycles 15.4a-S1/S2/S3/S4, 15.4b, 15.5a, 15.6a + supplemental 15.6a-S1, 15.6b, 15.7a, 15.7b, 15.8a, 15.9a, 15.10a + 2 supplemental cycles 15.10a-S1/S2) covering HMM correctness + the Bayesian ODW system + cross-pipeline analysis-surface parity + reliability + flat-sample detection + summit refinement strategy menu + summit↔timing convergence + APS analytical work + clustering defaults finalization + SAPS + label-dictionary architecture + within-prior second-pass background + posterior architecture (per-pipeline manifests) + APS catalog + step-less plots/notebooks + peak-summary cross-pipeline extension + architectural cleanup (timing-diagnostics module + ODW per-direction threshold split + Gap-analysis argparse group + Growth engine argparse elimination + posterior second-pass controls) + HMM `--norm-mode chrom-median` default-flip-and-revert + housekeeping bundle + closeout sweep + curated 68-strategy summit-stage-selection menu cross-pipeline (replacing an initial 9-strategy 2-flag implementation that was reverted at cycle 15.10a-S2).

Notable structural changes shipped during Phase 15:
- **Bayesian ODW System** (SPEC15.4) — cross-pipeline directional active-stage detector primitive in `onionskin_core/odw.py`.
- **`onionskin_core/timing_diagnostics.py`** (SPEC15.21) — new module consolidating stage-progression diagnostics out of `aps.py`.
- **`onionskin_core/summit_strategies.py`** (SPEC15.19 d3 cycle 15.10a Stage C F5; redesigned via cycle 15.10a-S2) — curated 68-strategy single-flag summit-stage-selection menu cross-pipeline (5 single-stage selectors × 4 estimators + 3 multi-stage selectors × 4 estimators × 4 patterns = 68 strategy IDs deterministically derived from selector × estimator × aggregation; 60 cross-pipeline + 8 HMM-only). Per CORRIGENDUM 2026-05-05, all 17 triangle-apex strategies emit NaN cross-pipeline pending future SUMMIT_SOUP item realization. Per-pipeline diagnostic BED + TSV emit at `<pipeline>/<summit-refinement-step>/diagnostic-summits/` with all 68 strategies per amplicon (TSV; NaN-allowed) + valid-summit-only rows (BED). Cycle 15.10a R2's initial 9-strategy 2-flag scaffold (`summit_aggregation.py` + `summit_diagnostics.py` + `summit_diagnostic_runner.py`) was REVERTED at cycle 15.10a-S2 Stage A.
- **HMM CLI surface** — `--hmm-statepath-base` + label-dictionary architecture (SPEC15.14); `--hmm-norm-mode` chrom-median default flip then partial reversal (SPEC15.16 + cycle 15.10a Stage A); SAPS cross-pipeline (SPEC15.13).
- **APS cross-pipeline parity** — analysis-surface parity (SPEC15.17) + master APS catalog at `multi-agent/full_instructions/APS_CATALOG.md` + step-less `plots/` + `notebooks/` directories cross-pipeline.
- **Per-pipeline posterior architecture** (cycle 15.10a-S1) — `_run_posterior` controller flow rewritten so each pipeline (Growth + RMS + HMM) builds its own posterior manifest from its own APS clustering output. Per-pipeline manifests + cluster maps + hires manifests at `02-posterior/02-growth-model/` + `02-posterior/01-hmm/` + `02-posterior/03-rcn-mean-shift/`. Grouping-level `02-posterior/posterior_manifest.tsv` + `posterior_manifest_cluster_map.tsv` dropped entirely. HMM `shutil.copy2` controller-mode hack removed. New `prior_layout["rms_aps_dir"]` alias.
- **Engine argparse elimination** — Growth engine's transitional `_run_argv()` parser + `_build_ms_argv()` argv-translation removed (SPEC15.21 d10); `run_multistage()` now consumes `argparse.Namespace` directly. AGENT_CONVENTIONS item 12 elevated: "no engine modules have argparse" (was "no future engine modules add argparse").
- **CLI semantic-inversion** — `--compute-aps` → `--skip-aps`, `--posterior` → `--skip-posterior` (cycle 15.10a Stage B F4). APS + posterior now run by default; legacy flags accepted with deprecation warning.

## Priority SPEC15.1 — HMM PuffStep re-audit + flag fixes ✓ DONE (cycle 15.1a, v0.14.77)
## Priority SPEC15.2 — Shared pre-pipeline gap mask ✓ DONE (cycle 15.2a, v0.14.78)
## Priority SPEC15.3 — HMM parallel child pipeline + step renumbering ✓ DONE (cycle 15.3a, v0.14.79)
## Priority SPEC15.4 — Bayesian ODW System ✓ DONE (cycle 15.4a, v0.14.80)
## Priority SPEC15.5 — HMM shape-score wiring + multistage unification ✓ DONE (cycle 15.4a, v0.14.80)
## Priority SPEC15.6 — Cross-pipeline reliability scoring + flat-sample detection ✓ DONE (cycle 15.4a, v0.14.80; refinement 15.4a-S1/S2/S3/S4 v0.14.81–v0.14.89)
## Priority SPEC15.22 — Gap-analysis architectural cleanup ✓ DONE (cycle 15.4b, v0.14.82)
## Priority SPEC15.7 — Cross-pipeline summit refinement strategy menu ✓ DONE (cycle 15.5a, v0.14.83)
## Priority SPEC15.8 — Cross-pipeline summit↔timing convergence ✓ DONE (cycle 15.5a, v0.14.83)
## Priority SPEC15.9 — APS area-excess-floor analytical testing ✓ DONE (cycle 15.6a, v0.14.84)
## Priority SPEC15.10 — APS column rename + raw-RCN migration ✓ DONE (cycle 15.6a, v0.14.84)
## Priority SPEC15.11 — Composite multi-feature APS clustering modes ✓ DONE (cycle 15.6a, v0.14.84)
## Priority SPEC15.23 — Strategy-selection determinism diagnostic + fix ✓ DONE (cycle 15.6b, v0.14.86)
## Priority SPEC15.12 — Clustering defaults finalization across pipelines ✓ DONE (cycle 15.6a-S1, v0.14.90)
## Priority SPEC15.13 — SAPS implementation ✓ DONE (cycle 15.7a, v0.14.91)
## Priority SPEC15.14 — `--hmm-statepath-base` flag + label-dictionary architecture + JSON sidecars ✓ DONE (cycle 15.7a, v0.14.91)
## Priority SPEC15.15 — Cross-pipeline within-prior second-pass background renorm + posterior inheritance ✓ DONE (cycle 15.7a, v0.14.91)
## Priority SPEC15.16 — HMM `--norm-mode chrom-median` default flip ✓ DONE (cycle 15.7a, v0.14.91); ✓ partial reversal DONE (cycle 15.10a Stage A, v0.14.95)
## Priority SPEC15.24 — HMM per-stage parabola summit emission ✓ DONE (cycle 15.7b, v0.14.92)
## Priority SPEC15.17 — APS analysis-surface parity + master APS catalog + step-less plots/notebooks + HMM plot ports ✓ DONE (cycle 15.8a, v0.14.93)
## Priority SPEC15.18 — `--peak-summary` extension to RMS + HMM + max-projection variants ✓ DONE (cycle 15.8a, v0.14.93)
## Priority SPEC15.21 — Architectural cleanup (timing-diagnostics + ODW per-direction split + Gap-analysis group + Growth engine argparse elimination + posterior second-pass controls) ✓ DONE (cycle 15.9a, v0.14.94)
## Priority SPEC15.19 — Phase 15 housekeeping bundle (incl. summit-aggregation strategy menu cross-pipeline) ✓ DONE (cycle 15.10a, v0.14.95) — initial 9-strategy 2-flag F5 implementation REVERTED + redesigned via cycle 15.10a-S2 v0.14.97 (curated 68-strategy single-flag menu in `onionskin_core/summit_strategies.py` replacing the deprecated `summit_aggregation.py` + `summit_diagnostics.py` + `summit_diagnostic_runner.py` modules)
## Priority SPEC15.20 — Phase 15 closeout sweep (tracking-file cleanup + CLUSTERING_DEFAULTS.md + user-facing doc cross-alignment + cross-pipeline-parity audit checklist) ✓ DONE (cycle 15.10a, v0.14.95) — supplemental tracking-doc coherence repairs landed via Final Overseer Pass 1 + orchestrator triage 2026-05-05 (T1 ROADMAP staleness, T2 SUMMIT_SOUP question 13+14 supersession markers, T4 KNOWN_ISSUES newest-first re-sort, T5 IBM-C4 disposition update)
## Supplemental cycle 15.10a-S1 — Multi-pipeline controller posterior architecture defect-fix ✓ DONE (cycle 15.10a-S1, v0.14.96) — `_run_posterior` rewritten to build per-pipeline posterior manifests from per-pipeline APS dirs (Growth + RMS + HMM each writes its own); per-pipeline `posterior_manifest_cluster_map.tsv` + per-pipeline `posterior_hires_manifest_<i>.tsv` siblings; grouping-level `02-posterior/posterior_manifest.tsv` + cluster_map dropped entirely; HMM `shutil.copy2` hack removed; new `prior_layout["rms_aps_dir"]` alias in `output_layout.py`. F8 regression test (`tests/test_pipeline.py`) locks per-pipeline-APS-source correctness against future regression. README + PIPELINE_SPEC posterior layout documentation updated for per-pipeline parity. `[ISSUE:2026-05-05:1]` removed at closeout.
## Supplemental cycle 15.10a-S2 — F5 strategy-menu redesign + revert ✓ DONE (cycle 15.10a-S2, v0.14.97) — Curated 68-strategy single-flag `--summit-stage-selection` menu cross-pipeline (Growth + RMS + HMM) implemented in `onionskin_core/summit_strategies.py` (~809 lines: registry + selector dispatch + aggregator dispatch + internal selector-compatibility shim for legacy bare tokens). Cycle 15.10a R2's deprecated 9-strategy 2-flag scaffold modules (`summit_aggregation.py`, `summit_diagnostics.py`, `summit_diagnostic_runner.py`, `tests/test_summit_strategies.py`) deleted entirely. CORRIGENDUM 2026-05-05 mid-flight: all 17 triangle-apex strategies emit NaN cross-pipeline pending future SUMMIT_SOUP summit-estimator item realization (orchestrator transcription error in original locked design surfaced + fixed; two distinct concepts disentangled: existing Growth-only triangle FIT for asymmetric fork elongation filed at `multi-agent/plans/next/PIPELINE_PARITY_SOUP.md` Item 1; new triangle-apex SUMMIT ESTIMATOR concept filed at `multi-agent/plans/next/SUMMIT_SOUP.md` Item 5 expansion 2026-05-05). F4 override path wired through engines per pipeline canonical-producer/consumer boundaries (Growth `_origins.tsv` patched; RMS unified-calls `peak` → timing `call_peak` handoff patched; HMM step 11 metrics patched upstream of step 13 consumption); defaults preserved (HMM detection bit-for-bit unchanged at canonical/lighter-gate stale-snapshot pattern). F5 diagnostic emit per-pipeline at `<pipeline>/<summit-refinement-step>/diagnostic-summits/<pipeline>_diagnostic_summits.{bed,tsv}` (TSV all 68 rows per amplicon; BED valid-summit-only). F6 sampled per-pipeline integration tests at new `tests/test_summit_strategies_integrated.py`. User-facing docs (README + PIPELINE_SPEC + ONIONSKIN_FULL_HANDOFF + APS_CATALOG) updated. Closed via orchestrator-as-R3 cycle-final exception per Principal direction 2026-05-05 (Final Overseer absorbs independent-eyes verification at phase level). `[ISSUE:2026-05-05:2]` removed at closeout.
