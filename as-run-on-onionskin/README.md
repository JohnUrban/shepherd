
# onionskin
**Onionskin** is a toolkit for detecting, modeling, and comparing developmental DNA amplification domains from genome-wide bedGraph coverage data.

It is designed for biological systems in which localized re-replication or intrachromosomal gene amplification produces **broad, summit-centered copy-number gradients** rather than simple flat CNV segments. The current development has been strongly motivated by **Sciara coprophila** and related developmental amplification systems, but the framework is intended to be useful more generally for broad amplification peak detection and analysis.

![logo](assets/logo/onionskin_logo_01.png)


## What onionskin does

Onionskin works with one or more binned genomic coverage tracks and can:

- detect broad amplification domains across three complementary detection pipelines (Growth-model, RMS = RCN-mean-shift, HMM state-path)
- estimate amplicon summits and boundaries with cross-pipeline strategy menus + ODW-confined refinement + sliding-offset sub-bin localization
- analyze multi-stage developmental progression
- compute timing metrics (onset stage, last-active stage, max-RCN stage, regression stage) using a Bayesian Origin Detection Window (ODW) framework
- generate per-sample and per-stage RCN summaries (with variance-aware MAD bedGraphs)
- compute **APS (Amplification Progression Score)** with multiple feature modes including 3-layer PCA
- score per-amplicon reliability across pipelines + classify per-locus flat samples (Bayesian-ODW-consistent)
- run summit↔timing one-pass 4-step convergence cross-pipeline
- cluster samples using APS-derived features
- support posterior sample ordering and grouping
- refine loci with higher-resolution inputs
- decompose neighboring or "twin" peaks within broad loci
- suppress false positives from collapsed repeats, gap regions, and constitutional signals

## Biological motivation

In systems such as **Drosophila chorion amplification** and **Sciara DNA puff amplification**, local re-replication can generate broad genomic domains where:

- copy number increases toward a replication origin
- reaches a summit near the origin
- decreases away from it on both sides
- broadens over developmental time as forks travel farther
- may show nested or doubling-like amplification structure

This means the signal of interest is often a **broad hill-shaped profile**, not a simple stepwise CNV segment. Onionskin is built specifically around that kind of signal.

## Current capabilities (Phase 15+ era; v0.14.84+)

Onionskin has evolved well beyond its original Phase 9 single-sample focus. It now supports three complementary detection pipelines (Growth-model, RMS, HMM) plus a unified analysis surface (APS, timing, summit refinement, reliability scoring, gap analysis, clustering) intentionally shared across all three. Phase 15 is currently mid-execution (HMM completeness + cross-pipeline generalization theme).

### Detection and refinement
- robust major-bin-size detection
- multistage and single-sample workflows
- summit and interval refinement using higher-resolution manifests
- summit-stage selection menu (`--summit-stage-selection`) — 68 curated opt-in strategy IDs spanning selector x estimator x aggregation, with per-pipeline overrides and diagnostic TSV/BED emission under each summit-refinement step
- ODW-confined summit refinement cross-pipeline + HMM sliding-offset sub-bin port
- summit↔timing one-pass 4-step convergence (across Growth + RMS + HMM)
- overlap resolution and twin-peak decomposition for broad neighboring calls
- peak-proximity deduplication across all call paths
- shape filter: suppresses false positives from constitutional / collapsed-repeat signals (cross-pipeline)

### Growth track modeling (multistage)
- five stage-progression methods: `step` (default), `linear`, `isotonic`, `unimodal`, `ensemble`
- `step` is robust to plateau and late-stage decline; does not require monotonic growth across all stages
- empirically validated on both single-animal (9-stage) and pooled (5-stage) datasets

### RMS (RCN-mean-shift) pipeline
- per-stage chrom-normalized signal-driven detection
- shape-filter + multistage unification
- timing reach (per-pipeline timing wired post-SPEC15.4)
- cross-pipeline parity with Growth-model on most analysis surfaces

### HMM state-path pipeline
- native multi-step HMM pipeline under grouped `01-prior/01-hmm/` outputs (and `02-posterior/01-hmm/` for posterior regrouping); ported from PuffStep with parity checks
- enabled by including `hmm` in `--pipelines` (HMM is explicit opt-in); `--pipelines all` runs growth + rms + hmm
- gain step layout (post-cycle-15.4b reorder; post-cycle-15.5a additions): mednorm → stage-medians → removeZeroBins → chrom-renorm → medianSmoothed-RCN → HMM → collapsedHMM → aboveBackground → summitStates → summitBins → amplicon-metrics → fork-travel → summit-refinement → multistage-unification → **gap-analysis** → APS → SAPS → timing → clustering → active-stage-summit → timing-updated
- HMM state-path bedGraphs are emitted with `--hmm-statepath-base 0` by default; use `--hmm-statepath-base 1` for legacy PuffStep-compatible labels. State-path outputs carry self-describing headers, and HMM runs emit `00-pipeline-metadata/pipeline_info.json`.
- HMM normalization now defaults to `chrom-median`; pass `--hmm-norm-mode ref-stage` to preserve legacy reference-stage behavior on old analysis recipes.
- single-file / single-stage HMM runs without a denominator use a no-ratio preprocessing branch
- reference-backed HMM runs use the ratio-backed branch when `--hmm-norm-mode ref-stage` is selected
- HMM step16 APS emits MAD bedGraphs alongside Growth (13-aps) and RMS (12-aps), feeding variance-aware sigmas into the cross-pipeline timing + flat-sample tests
- outputs state-path bedGraphs, collapsed regions, above-background regions, summit-state regions, summit bins, per-amplicon metrics, fork-travel tables, gap-suspect filter outputs, HMM APS / timing / clustering outputs, and HMM notebooks
- step-6 fallback backend is debug-only via env var: `ONIONSKIN_HMM_STEP6_BACKEND=native|puffstep`

### Timing
- onset-stage estimation
- max-stage / max-RCN reporting
- **Bayesian Origin Detection Window (ODW) framework** (`onionskin_core/odw.py`) with three detector modes:
  - `hardthreshold` — legacy 1.25× rule
  - `gaussian_overlap` — variance-aware Gaussian-overlap test
  - `bayesian_posterior` (default) — Bayesian posterior over active vs background
- 4-metric framework: `onset_stage`, `last_active_stage`, `max_rcn_stage`, `regression_stage`
- ODW boundary metadata columns (`odw_start_stage`, `odw_end_stage`, `odw_duration_stages`)
- `--odw-active-stage-detector`, `--odw-prob-threshold`, `--odw-fold-threshold` CLI controls
- variance-aware sigma resolution from per-stage MAD bedGraphs (cross-pipeline)
- timing reach across all three pipelines (Growth + RMS + HMM all emit timing post-SPEC15.4)
- summit↔timing 4-step convergence: initial summit → initial timing → active-stage-confined refined summit → final timing on refined summits

### Reliability + flat-sample detection
- cross-pipeline reliability scorer (`onionskin_core/reliability.py`) emits `amplicon_reliability.tsv`
- five evidence axes; `keep_override` escape hatch via `--keep-amplicons-bed` / `--known-reliable-amplicons`
- `gap_suspect` hard-exclude axis from gap-analysis cross-pipeline (cycle 15.4b SPEC15.22)
- per-locus flat-sample test (`onionskin_core/flat_sample.py`) reuses the Bayesian ODW transition primitive against the chrom-median baseline
- posterior stage-1 anchoring of flat samples; auto-switch-to-chrom-median controller when zero flat samples detected
- `--flat-sample-prob-threshold`, `--flat-sample-fold-threshold`, `--flat-sample-min-flat-fraction`, `--allow-nonflat-ref-stage` CLI controls

### APS (Amplification Progression Score)
- scalar APS metrics + per-locus APS contributions
- new column schema: `summit_rcn`, `mean_rcn`, `area_excess`, `width_above_threshold_bp`, `log10area`, `asymmetry_ratio` (cycle 15.6a SPEC15.10 retired the legacy `summit_excess` / `mean_excess` / `peak_rcn` columns)
- area-excess floor modes: `--aps-area-excess-floor on|off|post_sum` (per-bin floor, no floor, or amplicon-total post-sum floor)
- sample-level aggregation floor: `--aps-sample-aggregation-floor on|off`
- multiple feature modes via `--aps-feature`: `summit`, `log2summit`, `area`, `log10area`, `log2area`, `shape`, `log2shape`, `shape_pca`, `log2shape_pca`, `sample_shape_pca_raw`, `sample_shape_pca_reduced`, `amp_metrics`, `sample_metrics`, `metrics`
- 3-layer deterministic PCA (`onionskin_core/aps_pca.py`): per-amplicon PCA → per-sample PCA → optional matrix-level final PCA
- PCA controls: `--aps-num-amp-pc`, `--aps-num-sample-pc`, `--aps-pca-matrix`, `--aps-final-pc`
- APS matrices for clustering (raw and z-scored)
- hierarchical clustering with Ward linkage + Euclidean distance
- multiple posterior ordering outputs (scalar, dendrogram, hybrid)
- amplicon importance ranking output (`aps_amplicon_importance.tsv`) on Growth + RMS; cross-pipeline parity with HMM is queued for Phase 15 cycle 15.8a (SPEC15.17 expansion)
- score-aware APS aggregation via `--aps-aggregation-mode`, `--aps-aggregation-score`, `--aps-aggregation-score-min`, and `--aps-emit-unfiltered-aggregates`
- per-pipeline `_amplicons_recommended_to_keep.bed` / `_amplicons_recommended_for_exclusion*.bed` advisory BEDs (cross-pipeline post-cycle-15.4b)

### Output organization
- structured output directories (`01-prior/` for prior run outputs; `02-posterior/` runs by default — pass `--skip-posterior` to suppress)
- numbered grouped outputs (`01-hmm/`, `02-growth-model/`, `03-rcn-mean-shift/`)
- canonical final multistage result files nested under each pipeline's grouped dir, not directly under `01-prior/` / `02-posterior/`
- stage summaries, timing tables, APS tables, reliability sidecars, gap-annotated calls, posterior manifests
- summary bedGraphs and BED outputs
- amplicon recommendation files cross-pipeline
- rejected-call logs (`_amplicons_actively_rejected.bed`, `_amplicons_actively_rejected_multistage.bed`)
- diagnostic plots (per-pipeline `*-plots/` directories)
- pre-built Jupyter notebooks (per-pipeline `*-notebook/` or `notebooks/` directories)
- shared pre-pipeline gap mask + missingness diagnostic at `01-prior/00-gap-analysis/`
- robust `--verbose` / `--debug` logging

## Input data

### Primary input

Onionskin expects **bedGraph** files with four columns:

```text
chrom    start    end    value
```

### Typical workflows

#### 1. Single-sample mode
Use one bedGraph, optionally with one or more reference controls.

#### 2. Multistage mode
Use a manifest mapping samples to stages:

```text
stage<TAB>path
```

Optional high-resolution manifests can also be provided (e.g. 1 kb or 500 bp bedGraphs alongside the base 5 kb data).

> **Note — hires manifests are experimental and low-priority on a first pass.**
> Currently, hires data only affects summit position estimation; it does not improve
> call detection, boundary accuracy, timing, or APS. The summit module is still under
> active development and hires data does not reliably improve accuracy in its current
> form. **Recommendation: omit `--hires-manifest` on your first pass.** A dedicated
> post-hoc summit refinement command (`--refine-summits`) is planned for a future
> release, allowing you to re-run just the summit module on an existing output directory
> if finer summit precision is later needed.

## Example use cases

### Single-sample analysis
- broad amplification detection in one sample
- optional normalization against reference control(s)
- useful when only one stage or one file exists

### Multistage developmental analysis
- identify candidate amplicons across time
- refine summits and boundaries via active-stage summit refinement + summit↔timing convergence
- estimate onset timing + ODW boundaries
- compute APS with chosen feature mode
- score amplicon reliability + identify flat samples
- cluster and reorder samples by amplification state via APS-derived features
- run posterior reruns by default (pass `--skip-posterior` to suppress) under refined sample groupings

## APS (Amplification Progression Score) — primary defaults

APS is a post-core analysis layer intended to summarize how far a sample has progressed through the amplification program. The default scalar APS is based on **integrated excess amplification** across final resolved amplicon loci:

```text
APS_area = sum(max(RCN - 1, 0)) across all bins in all final loci
```

Default feature mode (`--aps-feature summit`) reads raw `summit_rcn` (windowed at the refined origin position) post-cycle-15.6a; the legacy floored `summit_excess` columns are retired. Multiple feature modes are available for clustering experiments — see the **APS** subsection of "Current capabilities" above for the full list. After Phase 15 closes, a master APS catalog (`multi-agent/full_instructions/APS_CATALOG.md`) will document Universal vs Pipeline-specific surfaces in detail; until then, refer to the SPEC at `multi-agent/plans/PHASE15_SPEC.md` for the canonical contract.

## Current output categories

Onionskin emits outputs including:

- unified and resolved amplicon BED files
- amplicon recommendation BED files (`*_amplicons_recommended_to_keep.bed` + `*_amplicons_recommended_for_exclusion*.bed`) cross-pipeline
- rejected-call BED files (`_amplicons_actively_rejected.bed`, `_amplicons_actively_rejected_multistage.bed`)
- timing TSVs (per-pipeline)
- stage summary TSVs
- ODW boundary + amplicon-class metadata
- origins TSVs (summit positions, CIs, shape scores, parabola estimates, sliding-offset estimates, max_rcn_stage)
- final multistage calls / timing / origins / progression files under `02-growth-model/`
- summit BED files (base, hires, final)
- step-numbered active-stage-summit + timing-updated TSVs (cross-pipeline 4-step convergence outputs)
- per-stage within-call tracks (Growth `06-stage-medians/`)
- per-pipeline genome-wide per-stage RCN tracks + MAD bedGraphs in each pipeline's APS step (`13-aps/genome_stage_medians/` for Growth, `12-aps/genome_stage_medians/` for RMS, `16-aps/genome_stage_medians/` for HMM)
- per-sample RCN tracks (HMM `05-medianSmoothed-RCN/indiv_samples/`, RMS `02-chrom-norm/indiv_samples/`)
- gap-annotated calls + per-pipeline gap analysis outputs
- shared pre-pipeline gap mask + missingness diagnostic (`01-prior/00-gap-analysis/`)
- amplicon reliability sidecars (`amplicon_reliability.tsv` cross-pipeline)
- flat-sample sidecars (`flat_samples.tsv`)
- APS tables, clustering outputs, dendrogram and posterior orderings
- amplicon-importance + scalar-ordering tables (Growth + RMS; HMM parity queued for cycle 15.8a)
- 3-layer PCA artifacts (`aps_matrix_layer1_pca.tsv`, `aps_matrix_layer2_pca.tsv`, optional `aps_matrix_layer3_pca.tsv`)
- posterior manifests and cluster maps
- HMM outputs under grouped `01-prior/01-hmm/`
- diagnostic plots in per-pipeline `*-plots/` directories
- pre-built Jupyter notebooks in per-pipeline `*-notebook/` or `notebooks/` directories

**Output directory layout (current — derived from `onionskin_core/output_layout.py` at HEAD; subject to phase-mid drift in HMM step layout — consult the live tree if the README and code disagree):**

```text
outdir/
├── 01-prior/
│   ├── 00-INDEX.md
│   ├── 00-gap-analysis/                   (shared pre-pipeline gap mask + missingness diagnostic)
│   ├── 01-hmm/                            (when `--pipelines` includes `hmm`)
│   │   ├── 01-mednorm/
│   │   ├── 02-stage-medians/
│   │   ├── 03-removeZeroBins/
│   │   ├── 04-chrom-renorm/
│   │   ├── 05-medianSmoothed-RCN/
│   │   │   └── indiv_samples/
│   │   ├── 06-HMM/
│   │   ├── 07-collapsedHMM/
│   │   ├── 08-aboveBackground/
│   │   ├── 09-summitStates/
│   │   ├── 10-summitBins/
│   │   ├── 11-amplicon-metrics/
│   │   ├── 12-fork-travel/
│   │   ├── 13-summit-refinement/
│   │   ├── 14-multistage-unification/
│   │   ├── 15-gap-analysis/
│   │   ├── 16-aps/
│   │   │   └── genome_stage_medians/      (per-stage RCN + MAD bedGraphs)
│   │   ├── 17-saps/                       (SAPS state-path APS tables)
│   │   ├── 18-timing/
│   │   ├── 19-clustering/
│   │   ├── 20-active-stage-summit/
│   │   ├── 21-timing-updated/
│   │   └── notebooks/
│   ├── 02-growth-model/
│   │   ├── onionskin*.{tsv,bed,bedGraph}    (canonical final multistage result files)
│   │   ├── 01-growth-track/
│   │   ├── 02-calls/
│   │   ├── 03-shape-filter/
│   │   ├── 04-origins/
│   │   ├── 05-progression/
│   │   ├── 06-stage-medians/
│   │   ├── 07-signal-tracks/
│   │   ├── 08-summits/
│   │   ├── 09-summit-refinement/
│   │   ├── 09b-active-stage-summit/
│   │   ├── 10-timing/
│   │   ├── 10b-timing-updated/
│   │   ├── 11-overlapResolution/
│   │   ├── 12-gap-analysis/
│   │   ├── 13-aps/
│   │   │   ├── samples/
│   │   │   └── genome_stage_medians/
│   │   ├── 14-plots/
│   │   └── 15-notebook/
│   ├── 03-rcn-mean-shift/
│   │   ├── 01-stage-medians/
│   │   ├── 02-chrom-norm/
│   │   │   └── indiv_samples/
│   │   ├── 03-detection/
│   │   ├── 04-summits/
│   │   ├── 05-deduplication/
│   │   ├── 06-shape-filter/
│   │   ├── 07-summit-refinement/
│   │   ├── 08-multistage-unification/
│   │   ├── 09-gap-analysis/
│   │   ├── 10-amplicon-metrics/
│   │   ├── 11-plots/
│   │   ├── 12-aps/
│   │   ├── 13-clustering/
│   │   ├── 14-notebook/
│   │   ├── 15-timing/
│   │   ├── 15b-active-stage-summit/
│   │   └── 15c-timing-updated/
├── 02-posterior/                            (written by default; pass `--skip-posterior` to suppress)
│   ├── 01-hmm/
│   │   ├── posterior_manifest.tsv
│   │   ├── posterior_manifest_cluster_map.tsv
│   │   └── posterior_hires_manifest_{i}.tsv  (if hires manifests were provided)
│   ├── 02-growth-model/
│   │   ├── posterior_manifest.tsv
│   │   ├── posterior_manifest_cluster_map.tsv
│   │   └── posterior_hires_manifest_{i}.tsv  (if hires manifests were provided)
│   └── 03-rcn-mean-shift/
│       ├── posterior_manifest.tsv
│       ├── posterior_manifest_cluster_map.tsv
│       └── posterior_hires_manifest_{i}.tsv  (if hires manifests were provided)
```

> Phase 15 is currently mid-execution. The `plots/` and `notebooks/` paths are slated to become step-less (drop the numeric prefix) at cycle 15.8a (SPEC15.17). Some HMM-side analysis-surface emissions (`aps_amplicon_importance.tsv`, scalar orderings) are queued to land in HMM at the same cycle. Until then, the live tree is the source of truth.

## Project structure

Core code lives in:

```text
onionskin.py
onionskin_core/
tests/
scripts/
multi-agent/
```

### Main modules in `onionskin_core/`

#### Cross-pipeline analysis surface
- `aps.py` — APS computation, matrices, clustering, ordering, importance ranking
- `aps_pca.py` — 3-layer deterministic PCA (per-amplicon → per-sample → matrix-level)
- `aps_plots.py` — APS diagnostic plots
- `flat_sample.py` — Bayesian-ODW-consistent per-locus flat-sample detection
- `gap_analysis.py` — gap-distance annotation + `gap_suspect` filter derivation cross-pipeline
- `odw.py` — Bayesian Origin Detection Window framework (3 detector modes)
- `reliability.py` — cross-pipeline amplicon reliability scoring
- `summit_convergence.py` — summit↔timing one-pass 4-step convergence orchestration
- `summit_metrics.py` — shared `compute_max_rcn_stage` helper (cross-pipeline)
- `summit_strategies.py` — summit-stage selection menu (6 strategies)
- `timing.py` — onset and progression timing metrics; consumes ODW + MAD bedGraphs

#### Pipeline orchestration
- `engines/growth_model_engine.py` — growth-model pipeline orchestrator
- `engines/hmm_engine.py` — HMM state-path pipeline orchestrator
- `engines/rcn_mean_shift_engine.py` — RMS pipeline orchestrator
- `growth_model_engine.py` — top-level growth-model dispatcher
- `rcn_mean_shift_engine.py` — top-level rcn-mean-shift dispatcher
- `rcn_mean_shift_helpers.py` — RMS shared helpers
- `posterior.py` — posterior rerun logic (default-on; `--skip-posterior` suppresses)

#### HMM-specific
- `hmm_core.py` — HMM state-path core
- `hmm_metrics.py` — HMM amplicon metrics
- `hmm_summits.py` — HMM summit-state derivation
- `hmm_summit_refinement.py` — HMM filtering and summit refinement
- `hmm_multistage_unification.py` — HMM shape/state-path multistage unification
- `hmm_fork_travel.py` — HMM fork-travel/asymmetry metrics and plots
- `hmm_ported_analyses.py` — HMM-side gap analysis, APS, SAPS dispatch, timing, and trajectory clustering outputs
- `hmm_saps.py` — HMM State-APS scores from per-sample decoded state paths
- `state_path_io.py` — HMM state-label header and label-map helpers
- `hmm_notebooks.py` — HMM notebook generation

#### Detection + refinement primitives
- `detection.py` — detection logic
- `refinement.py` — summit and interval refinement (parabola + sliding-offset)
- `modeling.py` — peak-shape modeling
- `overlap.py` — overlap resolution and twin-peak decomposition
- `overlap_plots.py` — overlap/twin-peak diagnostic plots
- `binning.py` — major-bin-size detection
- `signal_transforms.py` / `signal_utils.py` — shared signal-processing utilities

#### Plots + notebooks
- `notebooks.py` — pre-built Jupyter notebook generation (Growth + RMS)
- `profile_plots.py` — per-stage profile diagnostic plots
- `qc_plots.py` — sample QC diagnostic plots
- `shape_filter_plots.py` — shape-filter diagnostic plots
- `summit_plots.py` — summit position diagnostic plots

#### I/O + utility
- `autodetect.py` — mode autodetection
- `common.py` — shared helpers
- `io.py` — manifests and bedGraph I/O
- `log.py` — centralized logging
- `output_layout.py` — output directory management
- `rcn_io.py` — clean RCN/log2RCN writers
- `readme.py` — output README generation
- `summaries.py` — summary tracks and tables

## Installation

Installation details are still evolving with the package. The project currently assumes a standard scientific Python stack including at least:

- Python 3.10+
- numpy
- pandas
- scipy
- matplotlib
- scikit-learn (for PCA + hierarchical clustering)
- pytest

Notebook generation also requires `nbformat` (typically pulled in via `jupyter` or as a transitive dep).

A formal `pyproject.toml` / `requirements.txt` is not yet wired; the project can be run directly from the repo. Set up an environment manually (conda/mamba/venv all work) with the dependencies above.

The optional `geckocorrection` external repo (gitignored at top level, like `PuffStep`) is not required for normal use. It is reserved for future-phase GC-aware chrom-median normalization work captured at `multi-agent/plans/next/GECKO_SOUP.md`.

## Validation

A release should not be considered acceptable unless it passes the standard validation workflow:

```bash
make test          # pytest unit tests (fast slice; recommended for inner-loop)
make toy           # multistage toy dataset smoke test
make single        # single-file / single-stage smoke test
make twin          # twin-peak decomposition on chr II toy subsets
make full          # full chr II: sensitivity + no-split + twin
make full-no-split # spurious-split regression, both datasets
make full-twin     # full-genome twin-peak resolution
make shape-score-bic   # shape-filter BIC regression
make summit        # canonical summit regression (Growth + HMM + RMS)
make summit-smoke  # quick summit regression
make puff-compare  # CRITICAL gate: HMM detection bit-for-bit vs PuffStep gold-standard
make test-single   # T1-T12 single-mode regression slice (slow; pre-release only)
```

`make full` covers sensitivity against known ground-truth amplicons, false-positive controls (negative region, collapsed-repeat region), spurious-split regression, and twin-peak resolution.

`make puff-compare` is the **critical gate** for any change touching HMM detection (steps 06-10): it verifies bit-for-bit parity against the PuffStep gold-standard outputs.

For summit quality specifically, `make summit` is the canonical summit regression surface across Growth, HMM, and RMS. Use `make summit-smoke` for the quick version and `make summit-baseline` to refresh the stored summit baseline when a deliberate behavioral change is accepted.

Helper/test utility roles are documented in [tests/UTILITY_SCOPE.md](tests/UTILITY_SCOPE.md).

## Developer tooling

**`make validate-flags [file ...]`** scans files for deprecated onionskin CLI flags and reports their current redirects. It exits 1 if any deprecated flags are found, so it can be used as a CI gate.

Example: `make validate-flags my_analysis_pipeline.sh`

## Current limitations

The package is already useful, but several active areas remain in development:

- twin-peak decomposition limited to 2-peak case (`max_peaks=2`); ≥3 neighboring amplicons not yet handled
- joint DNA + RNA developmental inference not implemented
- hires summit refinement remains useful mainly for summit-position inspection; it does not yet materially improve detection, timing, or APS on a first pass
- HMM analysis-surface parity vs Growth + RMS is not yet complete (e.g., HMM does not currently emit `aps_amplicon_importance.tsv`); cross-pipeline parity ports queued for Phase 15 cycle 15.8a (SPEC15.17)
- second-pass background estimation currently defaults off and emits per-pipeline mask sidecars when enabled; full two-pass rerun behavior remains under active Phase 15 validation
- some normalization-strategy choices (for example ODW threshold split into per-direction tunable knobs) are queued Phase 15 work

## Future directions

### Coming in Phase 15 (mid-execution)
- HMM two-pass renormalization full rerun + posterior inheritance hardening (SPEC15.15 follow-through)
- Cross-pipeline analysis-surface parity catalog (`APS_CATALOG.md` per SPEC15.17)
- HMM-thinner-APS-path explicit ports (`aps_amplicon_importance.tsv`, scalar orderings on HMM)
- HMM per-stage parabola summit emission for cross-pipeline parity (SPEC15.24, cycle 15.7b)
- Strategy-selection determinism diagnostic + fix (SPEC15.23, cycle 15.6b)
- APS clustering defaults + reliability-filtered aggregate decision (cycle 15.6a-S1)
- Architectural cleanup: timing-domain consolidation + ODW threshold split into per-direction tunables (SPEC15.21, cycle 15.9a)

### Beyond Phase 15
- non-trivial twin-peak decomposition (≥3 neighboring amplicons)
- joint DNA + RNA developmental inference
- publication-grade reporting and visualization layers
- Bayesian-weighted amplicon reliability framework (parked at `multi-agent/tracking/BRAINSTORM.md`)
- GC-aware chrom-median normalization (parked at `multi-agent/plans/next/GECKO_SOUP.md`)
- Cross-pipeline structural unification — base-class scaffolding to enforce parity at the type level (parked at `multi-agent/plans/next/CROSS_PIPELINE_UNIFICATION_SOUP.md`)
- Summit methodology phase — bootstrap of parabola estimator, schema rationalization, cross-pipeline summit-algorithm parity completion (parked at `multi-agent/plans/next/SUMMIT_SOUP.md`)
- Continued primary-bin-cap / rolling_median hires smoothing design

## Important note

Onionskin is **not** intended as a generic cancer CNV caller. Its central design assumption is that the signal of interest often resembles a **broad summit-centered amplification domain with developmental progression**, not just flat gain/loss segments.

## Handoff and AI-agent docs

This repo includes onboarding and AI-agent-oriented documents under a `multi-agent/` directory. The canonical long-form context document is:

```text
multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md
```

The pipeline contract reference is:

```text
multi-agent/full_instructions/PIPELINE_SPEC.md
```

Active phase planning lives under `multi-agent/plans/`, and tracking-tier reservoirs (KNOWN_ISSUES, BRAINSTORM, DECISIONS) live under `multi-agent/tracking/` and `multi-agent/project_context/`. See `multi-agent/AGENT_CONVENTIONS.md` for the full convention set.

This is useful for:
- new ChatGPT threads
- Claude Code
- Codex
- Copilot
- human onboarding

## Acknowledgements and Lineage

Onionskin's HMM integration work is deeply informed by **PuffStep**, which itself descends from **pufferfish**.

- pufferfish lineage repository:
    https://github.com/JohnUrban/pufferfish
- early pufferfish development context (historical subdirectory):
    https://github.com/JohnUrban/sciara-project-tools

PuffStep's own README describes the modernization path from pufferfish to PuffStep, including migration to Python 3, removal of R/rpy2 dependencies, and streamlined core HMM tooling.

Timeline highlights from PuffStep documentation:
- pufferfish 0.0.0 (02/15/2016)
- pufferfish 0.0.0b (06/01/2020)
- pufferfish 0.1.20200925 (09/25/2020)
- PuffStep 1.1.20260218 (02/18/2026)

Please cite the historical PuffStep/pufferfish origin as requested by PuffStep:

- John Urban (2016), PhD Thesis, Brown University:
    "The genome and DNA puff sequences of the fungus fly, Sciara coprophila, and
    genome-wide methods for studying DNA replication."
    https://repository.library.brown.edu/studio/item/bdr:733543/ (see Chapter 4)

## Status

This project is under active development and is evolving quickly. Phase 15 (HMM completeness + cross-pipeline generalization) is currently mid-execution. Expect active refinement of:
- HMM-side analysis surfaces (SAPS, normalization defaults, statepath conventions)
- ODW + summit-stage selection defaults
- APS feature modes + clustering defaults
- Cross-pipeline parity surfaces (catalogs, plot-surface unification)
- Strategy-selection determinism + reproducibility guarantees

The core biological and architectural direction is well established. The detailed contract for Phase 15 work lives at `multi-agent/plans/PHASE15_SPEC.md`.
