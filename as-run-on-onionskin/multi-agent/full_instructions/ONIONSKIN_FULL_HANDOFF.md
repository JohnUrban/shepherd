# Onionskin — Full Expanded Project Handoff

**Last updated:** 2026-04-01 | **Corresponds to:** v0.5.39 (Phase 5)

## Purpose of this document

This document is a comprehensive handoff for a new AI coding assistant or human developer joining the **onionskin** project with **zero prior context**.

It is intentionally long and detailed. It is designed to:

- explain the scientific problem
- explain the software architecture
- explain the mathematical models and outputs
- record major design decisions and tradeoffs
- identify what is implemented vs incomplete
- define future directions and priorities
- make continuation in a new thread / Codex / Claude Code / Copilot feasible

This document should be treated as the best current narrative specification of the project.

---

## Reading and efficiency guide

This document is intentionally comprehensive. Use it efficiently:

- **Read it once per session** (or when the repository changes significantly). Do not re-read it in full across iterations.
- **After initial understanding, search or re-read only the relevant section** for the task at hand.
- **Do not re-read the entire document to answer a narrow question** — use grep/search to find the relevant section.

### Agent model selection

| Task | Recommended model |
|------|-------------------|
| Most coding, editing, spec updates | sonnet (default) |
| Deep cross-module reasoning, math correctness, ambiguous behavior | opus |
| Lightweight checks, verification passes, applying known fixes | haiku |

Escalation ladder: **haiku → sonnet → opus**. Do not escalate unless confidence is genuinely low. Do not make changes when uncertain — escalate first.

### Code reading strategy for agents

- **Perform an initial structural scan**, then read only files relevant to the current task.
- **Do not read all source files repeatedly** across iterations — track what you have already read.
- **Scope reads explicitly**: for a single-engine task, read `single_engine.py` + `refinement.py`, not the entire codebase.
- **Prefer targeted grep/search** over full-file reads when looking for a specific function or value.

### Iteration policy

- Prefer 1–2 high-quality passes over many shallow ones.
- Fix all findings in a single pass.
- Stop early if no high-confidence findings remain.

---

# 1. Project Identity

## What is onionskin?

**Onionskin** is a Python package / command-line pipeline for detecting, characterizing, and analyzing **developmental DNA amplification domains** from genome-wide sequencing coverage data.

It operates on **binned bedGraph coverage tracks** and is designed for both:

1. **single-sample / single-stage analysis**
2. **multi-sample staged developmental analysis**

The software is specifically motivated by **intrachromosomal gene amplification** and **re-replication** systems such as:

- **Drosophila melanogaster** chorion gene amplification
- **Sciara coprophila** DNA puffs and developmental amplification domains

but the framework is also intended to be useful for more general **copy-number increase detection**, provided the signal resembles broad, structured peak-like domains rather than purely stepwise CNV segments.

---

## What scientific problem does it solve?

The motivating biological problem is:

> detect genomic regions undergoing localized DNA amplification, estimate their boundaries and summits, track their progression over developmental time, and infer how samples relate to one another through the amplification program.

The specific biological signal of interest is **not** just a flat elevated copy-number segment. Instead, the signal often has a characteristic **peak / hill / onion-skin geometry**:

- copy number rises as one approaches a replication origin
- reaches a summit near the origin
- then falls away on both sides

So instead of a simple CNV step, the signal is often closer to a **broad mound or layered peak**.

This distinguishes onionskin from many cancer CNV tools, which are usually optimized for:

- long stepwise segments
- relatively flat gain / loss plateaus
- integer or near-integer copy state changes

In onionskin’s target domain, the signal often looks like:

- gradual increase
- central summit
- gradual decrease
- evolving width across time
- potentially nested or doubled structure in log space

---

## Who is the intended user?

Primary intended users are:

- computational biologists
- genome scientists
- developmental biologists
- anyone studying **DNA amplification domains** across development

Especially users working with:

- staged samples
- replicated samples per stage
- genome-wide coverage data in bedGraph format
- developmental systems where amplification changes across time

The current project has been driven by **Sciara coprophila** developmental amplification data, but the abstractions are meant to generalize.

---

## Biological background: re-replication and intrachromosomal amplification

### Re-replication
Re-replication means a genomic region is replicated more than once within a cell cycle or developmental context, often due to localized re-initiation of DNA replication.

This can produce:

- elevated local DNA copy number
- replication forks moving outward from origins
- gradients of copy number centered on origins
- layered amplification consistent with repeated replication rounds

### Intrachromosomal gene amplification
In systems like **Drosophila chorion amplification** and **Sciara DNA puffs**, a locus can become amplified within a chromosome arm during development.

This often produces domains where:

- central bins near an origin have highest copy number
- flanking bins are lower
- width expands as forks travel farther over developmental time
- later stages show broader and higher amplification

### Drosophila vs Sciara scale
A rough heuristic from this project has been:

- in **Drosophila**, meaningful amplification peaks may be on the order of **50 kb or more**
- in **Sciara**, the minimum interesting domains may often be even broader, though smaller low-amplitude domains can also exist

A key interest is in:

- high-amplitude large loci
- lower-amplitude late or subtle loci
- multi-stage progression
- broad developmental trajectories

---

## Why “onionskin”?

The name comes from the image of repeated layers of replication emanating from an origin, producing nested amplification structure — like layers of an onion skin.

The software is intended to capture:

- the broad hill shape
- temporal outward growth
- summit-centered amplification
- ultimately mechanistic progression through re-replication rounds

---

# 2. Inputs and Outputs

## Input file formats

### A. bedGraph
Primary raw input format.

Expected columns:

```text
chrom    start    end    value
```

Where:
- `chrom` = chromosome / scaffold name
- `start`, `end` = genomic coordinates
- `value` = per-bin normalized or semi-normalized abundance / coverage / TPM-like quantity

The package has worked with bedGraphs produced from pipelines such as:
- alignment + bedtools counting
- Sturgeon-derived quantification

The current expectation is that values can be interpreted as proportional to DNA copy signal and can be transformed into **RCN**.

---

### B. Manifest file
For multi-sample analysis, onionskin uses a tab-delimited manifest mapping samples to developmental stages.

Typical schema:

```text
stage<TAB>path
```

Some older or toy manifests may instead contain:

```text
file<TAB>stage
```

Manifest parsing has needed care because relative paths can be tricky, especially in toy data.

---

### C. High-resolution manifests
Optional manifests containing the same sample set at higher genomic resolution, for example:

- 5 kb base detection data
- 1 kb high-resolution refinement data
- 500 bp very high-resolution refinement data

These are used to refine:
- summit positions
- local peak structure
- overlap / twin-peak resolution

---

## Key input scenarios

### Scenario 1 — single file, no reference
Use one bedGraph only.

Goal:
- detect amplification-like domains in a single sample using within-sample normalization and shape-aware logic

### Scenario 2 — single amplified sample + one or more reference controls
Use one or more “amplified” files and optionally one or more reference files.

Goal:
- compute RCN relative to a cleaner early / non-amplified baseline

### Scenario 3 — multi-stage developmental series
Use multiple samples assigned to ordered developmental stages.

Goal:
- detect broad amplification program across development
- estimate timing
- compare samples
- compute APS
- cluster samples into posterior groups

---

## Current outputs (high-level categories)

Outputs are structured under an output directory. The exact layout depends on mode:

**Single-file / single-stage / ratio-of-ratios (Pipeline 1/2):**
```text
outdir/
├── README.md
├── 01-prior/
│   ├── aps/
│   ├── notebook/
│   ├── others/
│   ├── plots/
│   ├── samples/
│   ├── stage_median_within_calls/
│   ├── stage_medians/
│   ├── summary_bedgraphs/
│   ├── summits/
│   ├── onionskin_origins.tsv
│   ├── onionskin_single_calls.tsv
│   └── onionskin_amplicons_recommended_to_keep.bed
└── 02-posterior/   (written by default; pass --skip-posterior to suppress)
    └── (same structure as 01-prior/)
```

**Multistage (Pipeline 3):**
```text
outdir/
├── README.md
├── prior/
│   ├── aps/
│   ├── notebook/
│   ├── others/
│   ├── plots/
│   ├── samples/
│   ├── stage_median_within_calls/
│   ├── stage_medians/
│   ├── summary_bedgraphs/
│   └── summits/
├── *_origins.tsv
├── *_calls.tsv
├── *_timing.tsv
├── *_stage_summary.tsv
└── *_unified_onionskin_calls*.bed
```

---

## Important current / planned output types

### Core calls
- `*_unified_onionskin_calls.bed`
- resolved / postprocessed call files (varies by release)
- raw and resolved intervals

### Timing
- `*_timing.tsv`

Includes:
- onset stage
- last active stage (`last_active_stage`; renamed from the older `latest_activity_stage`)
- ODW boundary aliases (`odw_start_stage`, `odw_end_stage`, `odw_duration_stages`)
- max stage
- max RCN
- doublings
- lag from earliest
- onset evidence columns

The Origin Detection Window (ODW) detector is shared by growth timing,
HMM timing, RMS timing, and APS regression diagnostics. `--odw-active-stage-detector`
selects `hardthreshold`, `gaussian_overlap`, or `bayesian_posterior` (default);
`--odw-active-prob-threshold` / `--odw-active-fold-threshold` tune the
growth-direction timing and summit-selection consumers, while
`--odw-regression-prob-threshold` / `--odw-regression-fold-threshold` tune APS
regression diagnostics in the reverse biological direction. Stage 1 is tested
against background RCN=1.0 rather than against a nonexistent previous stage.

HMM now has a native multistage-unification sink before APS/timing:
`01-hmm/14-multistage-unification/` reports triangle shape scores,
state-path growth credibility, the selected unification mode, and retained
HMM loci. Downstream HMM late steps are `15-aps/`, `16-saps/` (reserved),
`17-timing/`, `18-clustering/`, and `19-gap-analysis/`.

### Stage summary
- `*_stage_summary.tsv`

Per amplicon × stage summaries, including:
- median peak RCN
- mean peak RCN
- fraction of samples exceeding thresholds
- BIC-related or model-support columns
- in later versions, max vs median clarity

### Summits
- BED-like files for summit positions, base and hires variants

### Summary tracks
- bedGraphs summarizing median / model / peak style tracks
- in both `RCN` and `log2RCN` spaces in more recent releases

### Per-sample tracks
Fully implemented and emitted in `samples/`:
- `sampleXXX.RCN.bedGraph`
- `sampleXXX.log2RCN.bedGraph`

### Stage medians
Fully implemented and emitted in `stage_medians/`:
- genome-wide per-stage median `RCN`
- genome-wide per-stage median `log2RCN`

### APS outputs (fully implemented, Phase 3+)
- scalar APS per sample (area, summit, width variants)
- APS per locus contributions
- per-amplicon reliability sidecar and flat-sample diagnostic sidecar
- APS raw matrix
- APS z-scored matrix
- hierarchical clustering outputs (Ward/Euclidean)
- dendrogram ordering (raw and z-scored)
- hybrid cluster-then-APS ordering
- comparison table
- `--aps-focus-loci` restricts computation to named amplicons
- `--aps-preamp-threshold` controls flat-sample flagging (`auto` reports without filtering)
- `--keep-amplicons-bed` retains specified loci despite reliability flags
- `--aps-singleton-guard off` disables reassignment of lone-member APS clusters to their nearest centroid
- `--aps-cluster-k 0` (default) auto-selects k = max(2, n_stages)

### Visualization outputs (Phase 4+)
Diagnostic plots in `plots/`:
- profile plots (per-stage median RCN/log2 bedGraph tracks with call overlays)
- summit plots (summit position distribution, parabola fits)
- shape-filter plots (dBIC score distributions, pass/fail visualization)
- overlap/twin-peak plots (split evidence, valley depths)
- QC plots (sample-level QC, normalization diagnostics)

Pre-built Jupyter notebooks in `notebooks/` (Phase 4+):
- APS analysis
- timing / growth curves
- fork progression
- gap analysis

### Posterior reruns (fully implemented, Phase 5)
- `--skip-posterior`: suppresses the posterior pass. By default the posterior manifest is built from APS cluster assignments and the full pipeline reruns under `02-posterior/`. (Legacy `--posterior` flag accepted with deprecation warning.)
- `_write_run_meta`, `_check_prior_complete`, `build_posterior_manifest` in `posterior.py`
- `02-posterior/` has the same directory structure as `01-prior/`

---

## Toy data and tests

### Toy dataset (`tests/toy_dataset/`)
Synthetic data validating:
- stage progression
- broad large amplicon growth
- late-appearing smaller amplicons
- multi-resolution behavior

Toy tests expect TP=2 (two synthetic amplicons), FP=0, FN=0.

---

### Twin-peak training data (`tests/twin_peak_training_data/`)
Small chr II subsets used by `make twin` (fast).  Superceded for comprehensive testing by `full_chrom_training_data/` but retained as a fast regression check.

**Dataset 1** — single-animal data, 9 stages, 2 replicates per stage.  **Dataset 2** — batch approach, 5 stages, 2 replicates.  Both available at 5 kb, 1 kb, and 500 bp.

Two training spots with ground truth:
- **Spot 1**: large amplicon (left) + shorter amplicon (right) at chr II ~5.9 Mb + ~7.2 Mb
- **Spot 2**: two short amplicons of roughly equal height at chr II ~22.1 Mb + ~22.7 Mb

---

### Full chromosome II training data (`tests/full_chrom_training_data/`)
Comprehensive chr II dataset used by `make full` and related targets.  Same two datasets (DS1 and DS2) but covering the **entire chromosome II**.  Includes multi-tier ground truth.

**Ground truth files:**

| File | Description |
|------|-------------|
| `amplicons.by-eye.bed` | 13 amplicons manually identified in IGV (most authoritative); 4 marked low-confidence in the name column ("`-low-confidence`") |
| `amplicons.HMM.bed` | 12 amplicons from the automated Puff Step HMM pipeline; `high_confidence` / `low_confidence` in name column (confidence based on width > 100 kb) |
| `example-collapsed-repeats.bed` | Two chr II collapsed-repeat / constitutional regions: point estimate at ~14.5 Mb (the known single-file FP) and full region at ~36.5 Mb |
| `example-known-negative-region.bed` | One negative-control interval (chr II 10.5–12.3 Mb) |
| `single-amplicons.for-no-split-test.bed` | The 4 HIGH-confidence single amplicons (II-3, II-4, II-5, II-7) used to test for spurious splitting |

**HIGH-confidence by-eye amplicons** (9 total — filter: no "`low-confidence`" in name):

| Name | Coordinates | Confirmation |
|------|-------------|-------------|
| II-1 | 5,910,689–6,501,954 | qPCR, FISH |
| II.7.2 | 7,180,767–7,561,088 | — |
| II-2.1 | 22,125,449–22,534,503 | — |
| II-2.2 | 22,678,925–23,275,379 | — |
| II-3 | 33,055,654–33,911,430 | qPCR, FISH, Southern, multiple methods (best-characterized) |
| II-4 | 41,413,288–41,906,860 | qPCR, FISH |
| II-5 | 46,060,415–46,458,919 | — |
| II-6 | 47,143,836–48,353,675 | qPCR |
| II-7 | 54,625,426–55,089,998 | qPCR |

**LOW-confidence by-eye amplicons** (4): II.17 (~17.4 Mb), II.19 (~19.4 Mb), II.26 (~26.5 Mb), II.31 (~31.2 Mb)

**14 amplicons have been experimentally validated** (across all chromosomes, not just chr II).

**Multi-peak training subdirectories:**
- `twin_peak_training/` — Ground truth for twin-peak spots 1, 2, and putative spot 3 (chr II II-6 region: II:47345000-47430000 low-conf + II:47565000-47925000 high-conf).  Putative spot 3 is informational.
- `triple_peak_training/` — Ground truth for a possible triple-peak at spot 1: Amplicon-1 (very tall), Amplicon-1.5 (very short, HMM only, lowest confidence, sits between Amp-1 and Amp-2), Amplicon-2 (short).  Under investigation.

**Trust rule:** Any coordinates the user provides as ground truth are authoritative.  Multiple ground truth sources (e.g., `dev/some-ground-truths.txt`, BED files, twin/triple training data) are subsets that can be unioned — apparent inconsistencies are sub-sampling, not contradictions.

---

### Dataset characteristics and known analytical challenges

Understanding why DS1 and DS2 behave differently is essential for interpreting test results and guiding future development.

#### Experimental design

| | Dataset 1 | Dataset 2 |
|-|-----------|-----------|
| Animal source | Single animals | Batches of 20–30 animals |
| Replicates/stage | 2 individual animals | 2 pooled batches |
| Stages | 9 (wider developmental window) | 5 (initiation window only) |
| Stage coverage | Initiation **and** elongation/plateau/decline | Initiation window only |

#### Why DS1 is analytically harder — and why this motivated the default method choice

Three compounding factors make the growth track approach less effective on DS1 when using slope-based methods like `linear`:

1. **Single-animal variance.**  With n=2 animals per morphological stage, real developmental timing differences between individuals create high within-stage noise.  The growth track fits per-stage medians; with small n and biologically meaningful animal-to-animal variation in amplification progression, the slope estimate is unreliable.  DS2's pooling suppresses this: each replicate averages over a population, yielding a stable stage estimate.

2. **Late-stage plateau and possible decline.**  DS2 covers the amplification *initiation* window, where copy number increases monotonically — the growth track's core assumption.  DS1 extends into later stages where initiation has largely stopped but elongation continues: amplicons get wider but not necessarily taller.  The growth slope across all 9 stages is therefore diluted by plateau stages, reducing z-scores for real amplicons.  The very latest stages may also show slight copy-number decline, possibly reflecting nuclease activity as salivary glands begin autolytic breakdown (uniform DNA loss; the apparent relative copy number of amplified loci may drift back toward 1).

3. **Normalization interaction.**  The chrom-median normalization is computed over all chr II bins.  In late stages with wide flat amplicons, the chrom-median itself shifts, potentially compressing the apparent relative copy number further.

These factors explain why the `linear` method (OLS slope averaged across all stages) yielded only 4/9 sensitivity on DS1: the plateau and decline stages dilute the slope signal.  A systematic scan of all five growth-track methods (Phase 3.23) showed that `step` — which finds the single stage boundary maximizing `mean(later) − mean(earlier)` rather than averaging across all stages — achieves 9/9 on DS1 and 9/9 on DS2 simultaneously with zero false positives.  This is why **`step` is now the default method**.  See the "Growth track methods" section below for the full empirical results table.

#### Why DS1 is the more important dataset

DS1 — single-animal resolution with a wide developmental window — is scientifically more informative for studying inter-animal variation, precise amplification timing, and late-stage dynamics.  Both DS1 and DS2 are now part of the full test suite (`make full`); both pass 9/9 with `step` at default parameters.  DS2 remains useful as a simpler validation case (fewer stages, pooled animals, monotonic initiation signal).

#### Future analytical directions for DS1

The core sensitivity problem with DS1 is solved by the `step` default.  The following directions remain open for further improvement or new research questions:

- **Add more replicates per stage** to DS1 to reduce single-animal variance, which would benefit all methods including `step`.
- **Regroup biologically equivalent stages** (e.g. collapse plateau stages) if finer stage resolution is not needed for a particular analysis.
- **Ratio-of-ratios approach**: compare RCN(stage N) / RCN(stage N−1) rather than absolute amplitude.  Invariant to baseline drift and initiation timing; more interpretable under plateau dynamics, though noisier near RCN=1.
- **HMM integration**: the Puff Step HMM detects a structural step-up pattern relative to flanks rather than a growth slope — an orthogonal detection approach.  Planned as an optional refinement layer (see HMM integration memory entry).
- **Reconciliation**: combine growth track calls and HMM calls into a single unified call set.

---

### Growth track methods and when to use them

The multistage pipeline builds a per-bin "growth evidence" track before calling amplicons.  The method is selected with `--growth-fit-method` (default: `step`).  All five methods are available and fully wired; choose based on the expected stage-progression shape of your data.

| Method | Core idea | Best for |
|--------|-----------|----------|
| `step` (**default**) | Finds the single stage boundary that maximizes `mean(later) − mean(earlier)`; takes the maximum over all possible split points | Any data — robust to plateau and late-stage decline; does not require monotonic growth across all stages |
| `linear` | OLS regression slope t-statistic across all samples | Data where amplification increases steadily across all stages (strict monotonic growth assumed) |
| `isotonic` | PAVA monotone fit improvement over flat model | Data that is mostly monotonically increasing but may have flat plateaus |
| `unimodal` | Rise-then-fall isotonic fit | Data with a clear peak stage and symmetric decline afterward |
| `ensemble` | Element-wise max of all four scores | Exploratory use or highly heterogeneous data; most permissive, highest FP risk |

**Empirical results on full chr II (Phase 3.23 scan):**

| Method | DS1 sensitivity | DS1 FP | DS2 sensitivity | DS2 FP |
|--------|----------------|--------|----------------|--------|
| linear | 4/9 | 0 | 9/9 | 0 |
| **step** | **9/9** | **0** | **9/9** | **0** |
| isotonic | 9/9 | 1 (col-repeat) | 9/9 | 1 (col-repeat) |
| unimodal | 9/9 | 1 neg + 1 crep | 9/9 | 2 neg + 1 crep |
| ensemble | 9/9 | 1 neg + 1 crep | 9/9 | 2 neg + 1 crep |

`step` is the only method achieving 9/9 with zero FPs on both datasets simultaneously.  `linear`'s 4/9 on Dataset 1 is explained by the plateau and possible late-stage decline in the 9-stage single-animal data diluting the slope estimate.

**Why `step` works on both datasets:**
The `step` scoring function is `max over all k of mean(stages > k) − mean(stages ≤ k)`.  Even if only 3 of 9 stages show rising signal and the remaining 6 are flat or declining, the contrast between "before onset" and "after onset" bins remains strong at those 3 stages.  `linear` averages the slope across all 9 stages, so the plateau dilutes the signal.

**z-threshold note:** `step` is more sensitive than `linear` at the same z-threshold.  If switching from `linear` to `step` and the test data is a small genomic extract (rather than full-chromosome), a higher z-threshold may be needed to suppress low-confidence FPs.  On full-chromosome data with chrom-median normalization, the default z-thresh (4.5 for multistage) is well-calibrated for `step`.

**Future ensemble use:** If `step` ever misses amplicons that another method would catch, the right next step is `--growth-fit-method ensemble --growth-ensemble-methods step,linear` (max of two clean methods) rather than the full four-way ensemble, which has higher FP rates.

---

### Standard test suite

**Fast (run every implementation):**
```bash
make test        # pytest unit tests
make toy         # multistage toy dataset smoke test
make single      # single-file + single-stage smoke test (alias: make toy-single)
make twin        # twin-peak decomposition on chr II toy subsets
```

**Medium (run when relevant logic changes or before releases):**
```bash
make full                  # comprehensive chr II: sensitivity + no-spurious-splits + twin peaks
make full-no-split         # targeted spurious-split regression for known single amplicons
make full-twin             # twin-peak resolution on all of chr II (spots 1, 2; putative spot 3 informational)
make full-collapsed-repeat # single-file shape-filter test; also discovers novel FP candidates
```

**Slow (run periodically or before major releases):**
```bash
make test-single # T1-T12 full single-mode regression matrix
```

`make full` runs both datasets (DS1 and DS2) on all of chr II at 5 kb.  Pass conditions: sensitivity ≥ 7/9 HIGH-confidence amplicons per dataset, zero FP in negative-control region, zero FP in collapsed-repeat region, no spurious splits of known single amplicons.  Also checks twin peaks at spots 1 and 2 (hard requirements) and putative spot 3 (informational).

`make full-collapsed-repeat` runs single-file mode twice — once with shape filter enabled (pass = no collapsed-repeat FP) and once with it disabled (informational output listing candidate novel constitutional/collapsed-repeat regions).

### Single-mode regression matrix (T1–T12, `tests/run_single_tests.sh`)
Covers all sub-modes of `run_single_stage` and `run_singlefile_caller`:

| Test | Mode | Data | Expected TP | Notes |
|------|------|------|-------------|-------|
| T1 | single-file (`--onionskin`), 5 kb | toy | 2 | baseline |
| T2 | single-file, 1 kb | toy | 2 | hires resolution |
| T3 | single-stage 2-rep, 5 kb | toy | 2 | replicate aggregation |
| T4 | single-stage 3-rep, 5 kb | toy | 2 | larger replicate set |
| T5 | ratio-of-ratios, 5 kb | toy | 2 | reference normalization |
| T6 | single-file, 5 kb | DS1 stage9 rep1 | 4 | FP=0 (shape filter removes 14.5 Mb) |
| T7 | single-file, 5 kb | DS2 stage5 rep1 | 1 | FP=0, FN=3 — documented limitation: stage5 is earliest, only strongest amplicon detectable from one replicate |
| T8 | single-stage 2-rep, 5 kb | DS1 stage9 | 4 | FP≤1 (17.3 Mb low-conf retained) |
| T9 | ratio-of-ratios, 5 kb | DS1 stage9/stage1 | 4 | FP≤1 |
| T10 | ratio-of-ratios, 5 kb | DS2 stage5/stage1 | 4 | FP≤1 (19.5 Mb low-conf retained) |
| T11 | single-stage + 1 kb hires | DS1 stage9 | 4 | FP≤1 |
| T12 | ratio + full hires (5kb+1kb+500bp) | DS1 stage9/stage1 | 4 | FP≤1 |

Pass/fail uses summit-based scoring: a prediction is TP if its peak position falls inside a truth interval.

### Scoring utility (`tests/score_calls.py`)
Unified scoring script used by all regression tests.
- Auto-detects TSV (`_single_calls.tsv`, has `peak` column) vs BED (midpoint as proxy summit)
- Summit-based scoring (primary pass/fail), reciprocal-overlap scoring, any-overlap scoring, Jaccard
- CLI: `--pred`, `--summits`, `--truth`, `--tag`, `--assert-tp/fp/fn`, `--json-out`, `--quiet`
- Exit 0 = pass, 1 = assertion failure, 2 = file error

### Parameter optimizer (`tests/optimize_single_params.py`)
Staged grid search over single-mode detection parameters using the real training datasets.
- Stage 1: z-thresh × trend-kb × smooth-kb (60 combos)
- Stage 2: peak-search-kb × window-kb (16 combos)
- Stage 3: halfwidths-kb variants (4 combos)
- Weighted F1 across toy + real datasets (real weight=3, toy weight=1)
- Parallel execution via `ProcessPoolExecutor`
- `make optimize-single` target

---

# 3. Architecture Overview

The project is structured around a main CLI and a set of core modules.

## Main entry point

### `onionskin.py`
This is the main user-facing CLI.

Responsibilities:
- parse arguments
- select single vs multi-sample flow
- coordinate outputs
- call into core modules
- increasingly become the single source of user-facing options

A recurring TODO has been:
> expose all relevant options at the main CLI level rather than hiding them only in underlying reference engines.

---

## `onionskin_core/`

Below is the conceptual role of each module that has been discussed / intended.

---

### `aps.py`
Purpose:
- compute APS scalar metrics (area, summit, width)
- compute per-locus APS contributions
- build APS matrices (raw and z-score)
- perform hierarchical clustering (Ward/Euclidean)
- derive multiple posterior orderings (scalar, dendrogram raw, dendrogram zscore, hybrid)
- write APS outputs

Key ideas:
- `APS_area`, `APS_summit`, `APS_width`
- raw vs zscore clustering
- Ward + Euclidean clustering
- dendrogram order
- hybrid cluster-then-APS ordering
- singleton guard (min stages with amplification evidence)
- `--aps-focus-loci`, `--aps-cluster-k 0` (auto k = max(2, n_stages))

Status: **fully implemented** (Phase 3+)

---

### `timing_diagnostics.py`
Purpose:
- shared APS timing-domain diagnostics extracted out of `aps.py`
- compute summit-window post-onset dip/regression diagnostics
- compute global-degradation summaries from those transition probabilities

Key ideas:
- reuse the ODW detector family in the regression direction
- keep APS table/output writing separate from timing-domain interpretation
- expose regression-direction defaults independently from timing's active-direction defaults

---

### `aps_plots.py`
Purpose:
- diagnostic plots for APS outputs
- APS matrix heatmaps, dendrogram visualizations, ordering comparison tables

---

### `engines/` (subdirectory of `onionskin_core/`)
Contains the refactored engine implementations:
- `engines/multistage.py` — refactored multistage engine functions (called by `multistage_engine.py`)
- `engines/single.py` — refactored single-mode engine functions (called by `single_engine.py`)

The top-level `multistage_engine.py` and `single_engine.py` serve as orchestrators that import from `engines/`.

---

### `log.py`
Purpose:
- centralized logging configuration
- structured log output with optional verbosity levels

---

### `notebooks.py`
Purpose:
- generate pre-built Jupyter notebooks in `notebooks/` output directory
- notebooks cover APS, timing, growth curves, fork progression, gap analysis

---

### `posterior.py`
Purpose:
- posterior rerun logic
- `build_posterior_manifest`: constructs a posterior sample manifest from APS cluster assignments
- `_check_prior_complete`: validates that a prior run is ready for posterior analysis
- `_write_run_meta`: writes run metadata for posterior tracking
- Powers the posterior pass (default-on; suppressed by `--skip-posterior`)

---

### `profile_plots.py`, `qc_plots.py`, `shape_filter_plots.py`, `overlap_plots.py`, `summit_plots.py`
Purpose:
- specialized diagnostic plot modules for each pipeline phase
- written to `plots/` subdirectory of every run

---

### `autodetect.py`
Purpose:
- auto-detect operation mode based on inputs
- infer single-sample vs multi-sample logic
- possibly infer resolution / bin behavior

Status:
- part of the architecture in some full releases
- one of the modules helping unify earlier separate programs

---

### `binning.py`
Purpose:
- detect the “major” bin size of a file robustly

Important design decision:
- do **not** use the first bin alone
- instead use the **mode of sampled bin widths**
- robust to:
  - truncated first bins
  - short terminal bins
  - filtered gaps
  - N-gap influenced odd bins

Status:
- this was an important bugfix / robustness improvement
- considered foundational

---

### `common.py`
Purpose:
- shared helper functions
- basic reusable utilities across engines

Status:
- part of the architecture in prior releases
- not usually the main focus of design discussion

---

### `detection.py`
Purpose:
- invokes the reference v2 single-file engine (`onionskin_call_autobin_v2.py`) via `common.load_reference_single()`
- `run_singlefile_caller(bedgraph_path, calls_path, extra_args)` is the main entry point
- writes a `_single_calls.tsv` with per-call detection metrics including `delta_BIC_block_minus_bump`

Important historical context:
- the v2 engine is strong at detecting gradient-shaped peaks and computing per-call shape metrics
- future decomposition may reuse some of its twin-peak detection ideas post hoc

---

### `io.py`
Purpose:
- read manifests
- read bedGraphs
- normalize path handling
- guard against relative-path bugs

Important gotcha:
- at one point toy data manifests caused `toy_data/toy_data/...` path duplication due to relative-path joining behavior

---

### `modeling.py`
Purpose:
- convert raw shape model statistics from the v2 engine into calibrated confidence scores
- `logbf_from_delta_bic(delta_bic)`: maps `delta_BIC_block_minus_bump` to a log Bayes Factor (`≈ 0.5 × delta_BIC`)
- `bed_score_from_logbf(logbf)`: maps logBF to a BED score 0–1000 (saturating transform, saturates around logBF ≈ 20)
- also contains `ci_width_to_conf_log10` for bootstrap CI confidence mapping

Note: the raw shape-fitting functions (`fit_bump_triangle_basis`, `fit_block`, `blockiness_metrics`) live in the reference engines, not here.  See design decision K for the shape detection architecture.

---

### `multistage_engine.py`
Purpose:
- primary orchestrator for staged multi-sample analysis
- imports core functions from `engines/multistage.py`

Responsibilities:
- load samples by stage
- compute RCN
- summarize by stage
- detect candidate amplicons
- track progression across stages
- coordinate timing, APS, and visualization layers
- use hires manifests for refinement

This is the main engine for the biologically rich developmental use case.

Note: The actual engine functions live in `onionskin_core/engines/multistage.py`; `multistage_engine.py` is the top-level orchestrator that imports from there.

---

### `output_layout.py`
Purpose:
- create structured output directories
- manage subdirectories such as:
  - `core/`
  - `samples/`
  - `stage_medians/`
  - `summaries/`
  - `summits/`
  - `aps/`

Status:
- introduced to solve the “too many flat output files” problem
- necessary for future `prior/` vs `posterior/` reruns

---

### `overlap.py`
Purpose:
- postprocess overlapping / duplicated amplicon calls
- decompose broad loci into multiple neighboring real amplicons (twin-peak splitting)

Current implementation (Phase 3.5+, fully working for 2-peak case):
- valley-depth-based splitting using stage-median-within-calls bedGraphs
- NMS (non-maximum suppression) dominant peak selection with `min_sep_bins` derived from `_MIN_SEP_BP = 200_000` (bin-size-agnostic since Phase 3.6)
- uses **last 3 stages** (highest stage numbers) for split evidence — these show the clearest separation
- local-outlier masking: artifact bins (fold < 25% of local 9-bin median) set to NaN before valley computation
- 500 kb clustering window collapses stage-to-stage positional drift in split candidates
- gap-bin masking: bins with log2 < −1.0 (fold < 0.5) set to NaN to prevent repeat regions from acting as artificial valleys
- single-call spans (no overlap) are also processed so internal twin peaks within one broad call are found

Peak-proximity deduplication:
- `apply_peak_proximity_dedup_tsv(tsv_path, peak_dist_kb)`: post-detection deduplication that merges calls whose summit positions are within `--dedup-dist` of each other (default 100 kb). Applied in `run_single_stage` after shape filtering. Complements the `(chrom, start, end)` exact dedup that removes duplicate halfwidth detections.

Known limitations:
- `max_peaks=2` limits to at most one split per span; three neighboring amplicons would require `max_peaks` increase
- 500 kb clustering window: closely spaced real valleys (< 500 kb) in the same span could be merged

---

### `rcn_io.py`
Purpose:
- write genome-wide RCN and log2RCN tracks cleanly
- centralize proper bedGraph writing
- avoid literal escaped tab/newline bugs

Historical note:
- there were bugs where output BED/BEDGraph files contained literal `\t` and `\n` text instead of actual delimiters/newlines
- centralized writers were a direct response to this

---

### `readme.py`
Purpose:
- auto-generate README.md in output directories
- document:
  - output structure
  - file meanings
  - parameters
  - possibly column schemas

Status:
- this has been requested and partially implemented conceptually
- should become stronger over time

---

### `refinement.py`
Purpose:
- refine candidate peaks / summits
- use hires manifests where available
- improve summit position estimates
- support overlap decomposition

This is especially relevant at:
- 1 kb
- 500 bp

and for subtle low-amplitude neighboring peaks.

Key functions (Phase 3.25+):
- `stage2_refine()` — applies the v2 Stage 2 window fitting at any resolution to obtain a refined peak position (used in the hires loop in `single_engine.py`)
- `fit_local_parabola(x_bp, y, center_bp, window_bp)` — fits a quadratic `f(x) = Ax² + Bx + C` to log2-fold signal in a window around the current summit; returns vertex `−B/(2A)`, curvature `A`, and residual variance.  Guards: ≥5 points, `A < 0` (downward curve), vertex within window.
- `refine_summit_parabola(resolutions, initial_summit_bp, wL_bp, wR_bp)` — hybrid hierarchical + weighted-mean refinement across all available resolutions.  Iterates coarsest-first; uses curvature from the previous fit to set an adaptive window for the next finer resolution (`w = min(w₀, sqrt(0.1/|A|))`).  Combines valid vertex estimates via inverse-residual-variance weighted mean.  See design decision M for full rationale.

---

### `single_engine.py`
Purpose:
- single-file and single-stage analysis path
- orchestrates the full single-mode pipeline from raw bedGraphs to final outputs

Key functions:
- `run_single_stage(samples, out_prefix, ...)`: the main entry point for manifest-driven single/ratio-of-ratios modes
  - aggregates replicates (chromosome-median-normalized fold track)
  - invokes v2 caller for detection
  - deduplicates calls by `(chrom, start, end)` — v2 detects each locus at multiple halfwidths
  - applies **shape filter** (`_shape_filter_calls` via `dBIC_flat_vs_tri`) to remove constitutional/repeat FPs
  - refines summits at base and hires resolutions via `stage2_refine`
  - bootstraps summit CI across replicates
  - writes `_origins.tsv`, `_metrics.tsv`, `_domains.bed`, summit BEDs, summary bedGraphs
- `apply_shape_filter_tsv(tsv_path, bg_path, min_score)`: applies the shape filter to a raw `_single_calls.tsv` written by `run_singlefile_caller` (used in the `--onionskin` single-file code path)
- `_dBIC_flat_vs_tri(seg, idx_s, idx_p, idx_e, baseline)`: computes the new shape metric (see design decision K)

Key outputs:
- `_origins.tsv`: per-call summit/origin position, interval bounds, logBF, BED score, `delta_BIC_block_minus_bump_base`, `shape_score_raw`, `summit_parabola_bp`, `summit_parabola_uncertainty_bp`; multistage additionally includes `final_summit_bp`, `final_summit_low/high/width_bp`, `final_summit_bp_median`, `final_summit_low/high_bp_median`, `summit_estimator_used`, `argmax_mean_bp`, `argmax_mean_ci_low/high_bp`, `argmax_mean_bootstrap_sd_bp`, `parabola_mean_bp`, `parabola_median_bp`, `n_parabola_valid`
- Cycle 15.10a-S2 summit strategy note: `--summit-stage-selection` is now a curated 68-ID opt-in menu. Growth overwrites `_origins.tsv:final_summit_bp` when a strategy is selected, RMS emits `08-multistage-unification/{out_prefix}_origins.tsv` for APS consumption, HMM carries `final_summit_bp` through `14-multistage-unification/all_trajectories.multistage_unification.tsv`, and all three pipelines emit `diagnostic-summits/` TSV+BED surfaces under their summit-refinement steps for by-eye comparison.
- `_metrics.tsv`: per-call geometry (width, asymmetry, summit height, shape scores, `summit_parabola_bp`, `summit_parabola_uncertainty_bp`)
- `_domains.bed`: interval calls
- `_summits.{base,hires_N,final}.bed`: summit positions at each resolution
- `_amplicons_actively_rejected.bed`: calls rejected by shape filter (6-column BED, append mode)

Key CLI flags added in Phase 5:
- `--shape-score-strict-bic on`: use k=0 for flat model in shape filter BIC (see Design Decision K)
- posterior pass: default-on; rerun pipeline under posterior manifest in `02-posterior/` (pass `--skip-posterior` to suppress)
- `--rcn-smooth-bins N`: apply rolling-median smoothing to RCN track before detection
- `--aps-focus-loci LOCI`: restrict APS computation to named amplicons
- `--aps-singleton-guard off`: disable reassignment of lone-member APS clusters to their nearest centroid
- `--threads N`: parallelism for CPU-intensive steps
- `--dedup-dist FLOAT`: summit proximity threshold for post-detection deduplication

Important architectural relationship:
- onionskin originally began as separate programs for single-file and multistage calling
- these were later merged under one umbrella CLI; `single_engine` preserves the simpler mode

---

### `summaries.py`
Purpose:
- generate output summary tracks and summary tables
- produce stage summaries
- prepare:
  - summary median tracks
  - model tracks
  - peak tracks

Desired outputs include:
- genome-wide stage medians
- within-call medians
- clear RCN/log2RCN summary tracks

---

### `timing.py`
Purpose:
- infer per-amplicon onset timing and progression metrics
- report stage of onset
- report max stage / max RCN
- report onset evidence columns
- support alternative onset methods:
  - RCN threshold
  - zscore
  - hybrid

Timing has been an active area of tuning.

---

## Pipeline Specification Document

`multi-agent/full_instructions/PIPELINE_SPEC.md` is a companion document that translates every pipeline in onionskin into precise plain-English prose and mathematics, step by step.  It is designed so that a human developer or AI agent can understand, verify, or re-implement any pipeline without reading the code.

**What it contains:**
- All four pipelines (single-file, single-stage, ratio-of-ratios, multistage) as numbered steps
- Mathematical definitions for every normalization, detection, shape-fitting, scoring, and output step
- Part 0 flowcharts giving an overview of all pipelines and Phase 2
- Per-pipeline detailed flowcharts
- "Where the code lives" subsections on every step, listing the relevant files and functions
- Appendix A: complete default parameter table for all CLI flags
- Appendix B: complete output file index for all pipelines and Phase 2

**Maintenance rules — read this before changing code:**

1. **Any time a pipeline step changes** (new normalization, new detection logic, new output column, changed default, changed formula), PIPELINE_SPEC.md must be updated to match.

2. **Any time a new CLI flag is added** that affects pipeline behavior (not just cosmetic), add it to Appendix A.

3. **Any time a new output file is added**, add it to Appendix B and the relevant step's output description.

4. **Any time CHANGELOG.md is updated**, check whether it merits updating PIPELINE_SPEC.md.  Produce an update plan first and report it to the user before making changes.

5. **Do not assume the spec is correct** — it has been developed through iterative audits and may lag behind code changes.  When in doubt, read the code.

6. **"Where the code lives" sections do not include line numbers** (too brittle to maintain).  File paths and function names are sufficient.

7. **The spec is not authoritative on defaults** — verify defaults against `onionskin.py` argparse definitions and the reference engines.  The code is always the ground truth.

**Key things to know when reading the spec:**
- Pipeline 1 = `run_singlefile_caller` + `apply_shape_filter_tsv`
- Pipeline 2 = `run_single_stage` in `onionskin_core/single_engine.py`
- Pipeline 3 = `multistage_engine.run_multistage()` → core functions in `onionskin_core/engines/multistage.py` (the legacy `reference_engines/onionskin_multistage_v5_clean.py` is now a reference only)
- Phase 2 = `_phase2_postprocess()` in `onionskin.py`
- Mode dispatch for `--manifest` path uses `autodetect.decide_mode()`; the `--onionskin` path dispatches inline in `main()`

---

## Reference engines

The package still includes older reference implementations in:

### `reference_engines/onionskin_call_autobin_v2.py`
The older single-file / single-sample engine.

Historically strong at:
- direct single-file peak logic
- seeing twin-peak structure in some regions

### `reference_engines/onionskin_multistage_v5_clean.py`
The older multistage engine.

Historically important because:
- many current capabilities were built by wrapping / modernizing this logic
- some options still live there and are not yet fully re-exposed through main `onionskin.py`

These reference engines matter because:
- some current behavior still depends on them
- future cleanup may refactor away direct reliance on them
- option exposure is still incomplete because of this history

---

## Overall pipeline flow

### High-level current flow

1. load inputs
2. detect / validate bin size
3. compute RCN / log2RCN
4. aggregate by stage if multistage
5. detect broad amplification loci
6. refine summits and boundaries
7. compute timing
8. resolve overlaps / duplicate broad calls
9. compute APS and APS-based clustering
10. write structured outputs

### Current extended flow (Phase 5+)

When the posterior pass runs (default-on unless `--skip-posterior` is passed):

1. prior run → `01-prior/` (full pipeline through APS clustering)
2. APS / clustering / posterior ordering
3. posterior rerun → `02-posterior/` (pipeline rerun under posterior sample manifest)
4. optional HMM / Bayesian interpretation layer (future)

---

# 4. Key Design Decisions

This section records major choices and why they were made.

## A. Use RCN language explicitly
There was discussion about whether outputs should say:
- `fold`
- `log2`
or instead:
- `RCN`
- `log2RCN`

Decision trend:
- favor **RCN** language because it reflects the biological interpretation more naturally for this project

Reason:
- these signals are interpreted as **relative copy number**
- `fold` is mathematically fine, but `RCN` is more domain-specific and intuitive here

---

## B. Robust bin-size detection by mode
Earlier approaches could mistakenly use the first bin width.

Decision:
- detect the **major / modal bin size** from sampled bins

Reason:
- first bins can be atypical
- terminal bins can be truncated
- bins near gaps can vary

Tradeoff:
- slightly more computation, but vastly safer

---

## C. Separate detection from interpretation
Core philosophy:

- **Detection** = find candidate amplified loci
- **Interpretation** = timing, APS, clustering, HMM, posterior ordering

This separation is important because:
- it keeps the core deterministic and debuggable
- it allows richer downstream inference without destabilizing locus calling

---

## D. Multistage first, single-sample still supported
The multistage developmental use case is the biologically richest and primary target.

But single-sample / single-stage still matters:
- useful when only one sample exists
- useful for post-hoc decomposition
- useful for regions where high-resolution individual shapes matter

So the architecture keeps both modes.

---

## E. APS as a post-core layer
APS was intentionally designed **not** to affect initial detection directly.

Instead:
- first define final loci
- then compute APS from those loci

Reason:
- APS depends on stable loci
- unstable broad / overlapping calls would make APS unstable

---

## F. Ward + Euclidean clustering for APS vectors
This came from earlier successful prototype work.

Decision:
- use hierarchical clustering:
  - `linkage(..., method='ward', metric='euclidean')`

Reason:
- biologically validated in earlier notebooks
- captures multivariate amplification state
- better fit than simplistic clustering defaults

Tradeoff:
- requires careful thinking about scaling
- leaf order is not uniquely meaningful in every tied case

---

## G. Compute both raw and z-scored APS clustering
Important design choice for APS.

Decision:
- compute both:
  - raw APS vectors
  - per-locus z-scored APS vectors

Reason:
- raw preserves magnitude, which may reflect biology
- zscore equalizes loci and reveals relative-pattern structure

Tradeoff:
- more outputs
- more choices for users
- but prevents premature overcommitment to one scaling scheme

---

## H. Output multiple posterior orderings
Decision:
- do not collapse posterior ordering into one single definitive rank too early

Instead compute:
- scalar APS_area ordering
- dendrogram leaf ordering (raw)
- dendrogram leaf ordering (zscore)
- hybrid cluster-then-APS ordering

Reason:
- scalar APS and multivariate clustering capture different things
- one can contradict the other
- exposing both is scientifically honest

---

## I. Twin-peak resolution should be post-hoc and adaptive
Decision trend:
- use broad multistage detection to identify candidate large regions
- then use a dedicated decomposition step to decide whether they contain 1, 2, or more peaks

Reason:
- multistage broad calls are good at identifying the program
- single/high-res logic is better at seeing local multiple summits
- earlier stages may separate neighboring peaks more cleanly than late merged profiles

This is not yet solved, but the design direction is clear.

---

## J. Always full release bundles, never patches only
This became a strict best practice.

Reason:
- patch bundles were confusing
- users should never have to guess how to merge files
- every deliverable should unzip into a self-contained `onionskin/` directory

Also:
- normal permissions
- zip root should be `onionskin/`
- release should be tested before delivery

---

## K. Shape detection: two complementary metrics, each for a different job

Two distinct shape metrics coexist in the pipeline.  They answer different questions and should not be conflated.

### Old metric: `delta_BIC_block_minus_bump`
Computed by the v2 reference engine (`stage2_score` in `onionskin_call_autobin_v2.py`).

- Compares a **fitted triangle** vs a **fitted flat block**, both with their own free intercepts (k=2)
- Operates on the **smoothed log2 residual** (after rolling-median local trend subtraction)
- Answers: *"Of the detrended residual signal, does this locus look more like a gradient peak or a flat plateau?"*
- Used for: **confidence scoring** (`logbf_from_delta_bic` → `summit_conf_logBF` → `bed_score` in origins.tsv / metrics.tsv) and **multistage score boosting** (`score = stage1_z + max(0, delta_bic)` in v5_clean)

### New metric: `dBIC_flat_vs_tri` (Phase 3.10)
Implemented in `single_engine._dBIC_flat_vs_tri`.

- Compares a **flat model** vs a **triangle model**, both with a **fixed chromosome-median baseline** (k=1)
- Operates on **raw log2** signal
- Answers: *"Is there a gradient structure above the chromosome-wide background at all?"*
- Used for: **pre-filtering constitutional/collapsed-repeat FPs** before confidence scoring

### Why the old metric fails as a constitutional-FP filter
The rolling-median trend subtraction absorbs constitutional elevation before the old metric sees the signal.  A collapsed-repeat FP and a weak true amplicon both produce small residual gradients — indistinguishable.  The key failure: the 14.5 Mb constitutional FP scored 8.9, the weakest TP scored 8.6.

### Why the new metric works for that job
The chromosome median is a **global reference** unaffected by local elevation.  A constitutionally elevated region is a flat plateau above that median — the triangle adds nothing (score ≈ 0).  A true gradient amplicon has peaked structure — the triangle wins by a large margin (score 68–900+).  Threshold of 50 validated to remove all constitutional FPs without losing any TP.

### Why both are kept
After the new filter removes constitutional FPs, the old metric is used for **quality ranking among surviving calls** — how confident is this a well-shaped gradient amplicon?  For this purpose, the old metric's use of stage2-refined boundaries (proper window geometry) makes it *more* accurate than the new metric, which uses coarse call coordinates.  In multistage mode, the ratio-of-ratios track already removes constitutional background, so the old metric works correctly there too.

### CLI exposure
`--shape-score-threshold FLOAT` (default 50, 0 disables) controls the new filter threshold.  Set in `onionskin.py`, passes through `_refine_kwargs` to `run_single_stage`.

`--shape-score-strict-bic on`: use k=0 for the flat model instead of k=1 (so the triangle must overcome a log(n_bins) penalty per call). Default mode uses `--shape-score-strict-bic off` (or omits the flag), keeping k=1 for both models so the penalty cancels, equivalent to a likelihood-ratio test. Strict mode reduces each call's score by log(n_bins) where n_bins is the number of finite signal bins spanning the call (~4.6 for 500 kb at 5 kb bins). Testing on DS2 chr II shows no calls change status at threshold=50 — real amplicons score 230–570 points above threshold, making the correction negligible in practice. Intended for future calibration.

Rejected calls: calls that fail the shape filter are written to `_amplicons_actively_rejected.bed` (6-column BED) in append mode in the run output directory.

---

## L. Single-mode parameter calibration: z-thresh=4.0, halfwidths starting at 40 kb

The default single-mode detection parameters were calibrated against real training data (Phase 3.9) after toy-data defaults produced excessive false positives.

**Key findings:**
- All FPs are wide (>305 kb) — a minimum width floor does not help; the issue is detection scale
- FPs are predominantly detected at halfwidths ≤ 20 kb (scale_bins ≤ 4)
- True amplicons are detected at halfwidths ≥ 40 kb (scale_bins ≥ 8)
- Raising z-thresh from 3.0 → 4.0 eliminates additional FPs without losing weak TPs
- `trend-kb=2000` is retained; larger values hurt by narrowing the baseline window

**Resulting defaults:**
- `--rms-z-thresh`: 3.0 → **4.0**
- `--rms-halfwidths`: `10,20,40,80,120,160,220,300` → **`40,80,120,160,220,300`** (remove <40 kb scales)

**Deduplication:**
The v2 caller detects each locus at multiple halfwidths.  `run_single_stage` deduplicates calls by `(chrom, start, end)` immediately after loading the TSV, keeping the row with the highest stage1 z-score.  Without this, the same region appeared multiple times and inflated FP counts.

**Separate single vs multistage defaults:**
CLI flags are split: `--rms-z-thresh` / `--growth-z-thresh`, `--rms-halfwidths` / `--growth-halfwidths`, etc.  Each mode has its own calibrated defaults.

---

## M. Local parabola summit refinement — design rationale and biological justification

### Previous summit logic and its limitations

The pipeline uses two complementary shape models, both inherited from the v2 reference engine:

**Stage 1 detection:** The v2 caller scans the genome at multiple halfwidths and assigns a z-score (`stage1_score_z`) to each candidate locus based on a circular-mean-shift bump score.  The rough peak position from stage 1 is the argmax bin within the detected domain.

**Stage 2 refinement (`stage2_refine`):** Applied per call, this refines the summit by fitting a scoring function (`rs`, a local detrended residual) in a large window (±300–1200 kb) around the stage 1 peak.  The refined peak is the position that maximizes `rs` within this window.

**Asymmetric triangle fit (`fit_asym_triangle`):** Separately, the multistage engine fits an asymmetric triangle model `f(x) = A(1 − |x − μ| / w)` with different halfwidths `wL` and `wR` to the per-stage median log2 profile in the same large window.  The triangle apex (`mu_bp`) is the summit estimate; `wL`/`wR` characterize domain shape.

**Limitations:**
- *Quantization:* `stage2_refine` returns the bin with the highest score — the result is quantized to the bin boundary.  At 5 kb resolution, summit positions can be off by up to 2.5 kb; even at 500 bp, errors up to 250 bp remain.  Sub-bin precision is never achieved by argmax alone.
- *Wide window pulls the apex:* The large fitting window is appropriate for characterizing overall domain shape and width, but means the triangle apex can be pulled toward noisy flanks.  The flanks have more bins than the apex, giving them disproportionate leverage on the fit.
- *Multi-resolution "finest wins" (Bug 5, fixed Phase 3.24):* The hires loop originally iterated finest-first and overwrote `final_peak` at each step, so the *coarsest* hires resolution silently won.  Fixed by iterating `reversed(hires_samples_by_res)` — coarsest first, finest last.  But even after the fix, "finest wins" is still a "one resolution wins, rest discarded" strategy that wastes information from all other resolutions.

### Why we moved to a local parabola (Phase 3.25)

### Why a local parabola works near the apex

Near any smooth maximum, the Taylor expansion is:

```
f(x) ≈ f(μ) + f''(μ)(x − μ)²/2 + f'''(μ)(x − μ)³/6 + …
```

The linear term is zero at the maximum by definition.  The quadratic (parabola) captures the dominant second-order term.  The parabola vertex `−B/(2A)` gives a least-squares estimate of `μ` with sub-bin precision — this is the classical sub-pixel peak estimation technique used in image processing and signal analysis.

### Biological justification for local symmetry

The RCN profile near the amplification origin arises from bi-directional replication forks that both traveled outward from the same initiation site.  At the very apex — immediately flanking the origin — both fork directions have covered similar distances and encountered similar chromatin.  Systematic fork-speed asymmetry exists at the scale of the full amplicon (captured by `wL`/`wR` and `symmetry_ratio`), but the *local* apex region is much more symmetric because there has been less time and distance for fork obstacles or speed differences to accumulate.

The only scenario that would strongly violate local apex symmetry is an immediate fork barrier within a few kilobases of the origin — such as is seen in rDNA where a polar replication fork barrier (Fob1) sits adjacent to the rDNA intergenic spacer origin.  No such structure has been observed in the Sciara or Drosophila amplicon loci studied here.  Even if a mild barrier existed, the parabola fit would not catastrophically fail — it would widen slightly and the residual variance would increase, which down-weights that resolution in the weighted mean.

### Why not asymmetric models

Asymmetric alternatives were considered and rejected:

- **Skew-normal / asymmetric Gaussian**: adds a skew parameter `α` (4 free parameters total).  In a narrow apex window with few bins, this is prone to overfitting noise rather than signal.
- **Cubic correction `Ax² + Bx + Cx³`**: captures first-order asymmetry, but the maximum is no longer `−B/2A`; solving `f'(x) = 0` for a cubic yields two candidate roots and is numerically unstable near a flat apex.
- **Bi-quadratic (separate left/right parabola with shared apex)**: requires knowing the apex location first (circular), and the continuity constraint complicates fitting.

The global flank asymmetry is already measured separately by `symmetry_ratio` and the `wL`/`wR` columns.  The local parabola is intentionally restricted to the apex zone (`min(wL, wR) / 3`) where the quadratic approximation is most valid.

### Hybrid approach: hierarchical window sizing + weighted-mean combination

Two multi-resolution strategies were evaluated:

**Pure hierarchical (vertex chaining):** use the 5 kb vertex to constrain the window for 1 kb, then the 1 kb vertex for 500 bp.  Intuitive, but error propagates — a biased coarse vertex pulls all subsequent windows.

**Pure weighted mean (fully independent):** fit each resolution in the same fixed window, combine via inverse-residual-variance weighting.  Robust, but the fixed window may be too wide for fine resolution (including linear slopes that pull the vertex) or too narrow for coarse (insufficient curvature sampled).

**Chosen hybrid:** use hierarchical structure *only for window sizing* — the curvature `A` from the coarser fit sets the adaptive window for the next finer resolution (`w_adaptive = sqrt(δ/|A|)`, δ = 0.1 log2 units, meaning the window spans the apex region where the parabola deviates by less than 0.1 log2 from the vertex).  Then fit each resolution *independently* within its window, and combine the vertices via weighted mean.  This gets:
- Curvature-adaptive windows (hierarchical benefit — sharper peak → narrower window)
- No error propagation in vertex estimates (weighted mean benefit)
- Automatic down-weighting of poor fits via residual variance

### Why all resolutions contribute roughly equally

The precision of the parabola vertex estimate scales as `σ²/(N·A²)` where `σ` = per-bin noise and `N` = number of points in the window.  Going from 5 kb to 500 bp bins: `N` increases ~10× but `σ` also increases (fewer reads averaged per bin, ~1/√bin_size noise scaling).  These effects roughly cancel — no single resolution dominates, and the weighted mean genuinely pools information across all of them.

### Promotion status (Phase 5.12+)

`summit_parabola_bp` and `summit_parabola_uncertainty_bp` are written alongside the existing `final_summit_bp`.

In **multistage mode** (Pipeline 3): the parabola estimate has been promoted — `final_summit_bp` uses `parabola_mean_bp` when at least one stage has a valid parabola fit and falls back to `argmax_mean_bp` otherwise.  The `_origins.tsv` now includes: `final_summit_low/high/width_bp`, `final_summit_bp_median`, `final_summit_low/high_bp_median`, `summit_estimator_used`, `argmax_mean_bp`, `argmax_mean_ci_low/high_bp`, `argmax_mean_bootstrap_sd_bp`, `parabola_mean_bp`, `parabola_median_bp`, `n_parabola_valid`.  `final_summit_low/high_bp` have mixed semantics: per-stage parabola min/max when `summit_estimator_used=parabola_mean`, bootstrap CI on argmax mean when `summit_estimator_used=argmax_mean`; the median low/high columns are always bootstrap CIs.

In **single-mode** (Pipelines 1/2): `final_summit_bp` remains the stage2-refine result; `summit_parabola_bp` is additive alongside it.

`summit_parabola_uncertainty_bp` is NaN when fewer than 2 valid fits are available (e.g., base-only runs with no hires data).

### Future: PuffStep integration

When PuffStep (the user's HMM amplicon caller) is integrated, the HMM-defined summit state interval will replace the `min(wL, wR) / 3` heuristic window entirely.  The parabola is fit within the structurally defined apex interval — a principled boundary rather than a geometry heuristic.  The two tools serve complementary roles: HMM for structural segmentation and fork travel estimation; parabola for continuous sub-bin apex localization within the segmented interval.

---

# 5. Current State

This section tries to separate what is working, what is partial, and what remains unresolved.

## Fully implemented / broadly working
As of Phase 5.41 (v0.5.41):

- main CLI (`onionskin.py`) fully exposes all engine options with split single/multi defaults
- multistage pipeline runs and finishes
- APS module with Ward/Euclidean clustering, z-scored matrices, multiple posterior orderings, per-locus contributions, scalar/summit/width APS, singleton guard, `--aps-focus-loci`
- timing outputs (onset stage, max stage, doublings, onset evidence columns)
- twin-peak decomposition working for the 2-peak case (NMS-based, bin-size-agnostic, passes training data TP=4 FP=0 FN=0)
- shape filter (`dBIC_flat_vs_tri`, `--shape-score-strict-bic on`) removes constitutional/collapsed-repeat FPs in single-mode; rejected calls written to `_amplicons_actively_rejected.bed`
- full single-mode regression suite T1–T12 all pass
- `make test`, `make toy`, `make single`, `make twin`, `make full`, `make test-single` all pass
- output directory structure fully organized under `01-prior/` (single-mode) and `prior/` (multistage)
- `plots/` and `notebook/` subdirectories populated in every run
- bootstrap summit CI for single-stage mode; per-stage argmax aggregation (weighted mean + median) for multistage
- posterior reruns fully implemented + default-on: posterior manifest built from APS clusters and pipeline reruns under `02-posterior/` (suppressed by `--skip-posterior`)
- per-sample tracks (`samples/`) and per-stage median tracks (`stage_medians/`) fully emitted
- `_amplicons_recommended_*.bed` recommendations output
- visualization outputs: diagnostic plots (profile plots, summit plots, shape-filter plots, overlap plots, QC plots) in `plots/`; pre-built Jupyter notebooks in `notebook/`
- output README auto-generated in every run (`readme.py`)

## Partially implemented / needing cleanup

### 1. Timing sensitivity
Timing outputs work, but onset-stage sensitivity thresholds remain tunable and not fully settled.

### 2. Max vs median ambiguity
Outputs need explicit labeling to clarify whether a reported peak value is:
- true maximum sample value
- stage median peak
- something else

### 3. Twin-peak decomposition — ≥3 peaks not yet handled
The current resolver is limited to `max_peaks=2` (at most one split per span).  Spans with three or more neighboring real amplicons would require extending this logic.

---

## Known incomplete or future-critical areas

### Bayesian posterior updating
Not implemented.
Desired in future:
- blend morphology prior with APS
- maybe later with RNA

### HMM integration
HMM is integrated as an optional pipeline (`--pipelines hmm`) with native
preprocessing, HMM state calling, summit-state extraction, fork-travel metrics,
multistage unification, APS, timing, clustering, notebooks, and gap analysis.
Cycle 15.7a adds SAPS at `17-saps/`, `--hmm-statepath-base` label metadata,
HMM pipeline metadata sidecars, and a chrom-median HMM normalization default.
Remaining HMM-specific future work includes richer trajectory-clustering
analysis surfaces and deeper empirical evaluation of posterior inherited-mask
synthesis modes beyond the new `local` / `union` / `intersect` control surface.

### Rich APS clustering using all bins
Not implemented.
Current APS clustering uses per-locus summaries, not bin-wise feature vectors.

### Twin-peak training-data-driven tests
Desired to be included permanently in the release and test suite.

---

## What tests cover

Routine targets (run on every implementation — all fast):

```bash
make test        # pytest unit tests (test_pipeline.py, test_aps.py)
make toy         # multistage toy run, validates full pipeline end-to-end
make single      # single-file + single-stage smoke test (alias: make toy-single)
make twin        # twin-peak decomposition on both real training datasets
make shape-score-bic  # compare default vs --shape-score-strict-bic on toy + DS2 chr II stage 5
```

Medium (run when relevant logic changes or before releases):

```bash
make full                   # comprehensive chr II: sensitivity + no-spurious-splits + twin peaks (both DS1 and DS2)
make full-no-split          # targeted spurious-split regression for known single amplicons
make full-twin              # twin-peak resolution on all of chr II (spots 1, 2; putative spot 3 informational)
make full-collapsed-repeat  # single-file shape-filter test; also discovers novel FP candidates
```

Slow (run periodically or before major releases):

```bash
make test-single # T1-T12 regression matrix for single-file / single-stage modes
```

See Section 2 (Toy data and tests) for the full T1–T12 description.

### Developer tooling

- `scripts/validate_onionskin_flags.py` — scan files for deprecated onionskin CLI
  flags; prints flag-to-redirect pairs; exit code 1 if any found. Run via
  `make validate-flags`.


## Development environment rules
For testing during development:
- NEVER write outputs into the repository root or tracked directories.
- ALWAYS run onionskin in a sandbox directory under:

    dev/runs/<run_name>/

- Example:

    python onionskin.py \
      --manifest tests/toy_dataset/manifest.tsv \
      --out-dir dev/runs/test1

- Temporary files, debugging outputs, and experiments must go under:
    dev/

- Do NOT modify tracked test data or manifests unless explicitly instructed.

---

# 6. Conventions and Style

## Language / environment
- Python
- expected modern scientific Python stack

## Likely dependencies
- `numpy`
- `pandas`
- `scipy`
- optionally `matplotlib` for diagnostics or clustering visuals
- possibly `sklearn` later if clustering / dimensionality reduction expands

## Style conventions
- snake_case
- modular helper functions
- explicit outputs
- avoid hidden state
- deterministic where possible

## Preferred engineering style
- explicit, readable code
- modular utilities
- clear file naming
- biologically interpretable metrics
- no unnecessary black-box steps

## Release and packaging style
Must follow these rules:
1. full release bundles only
2. zip root must be `onionskin/`
3. normal permissions
4. test before release
5. no patch-only deliverables

---

# 7. Future Directions

This is the long-term roadmap as discussed.

## Near-term priorities
1. max-vs-median clarity in output column naming (explicit labeling of max vs median peak values)
2. extend twin-peak decomposition to ≥ 3 neighboring amplicons (`max_peaks > 2`)
3. stronger non-overlapping final amplicon partitioning after splitting
4. primary-bin-cap and rolling_median hires smoothing design
5. APS notebook validation: change k=3 default to n_stages; optimal-k selection

## APS follow-on work
6. APS weighting experiments
7. richer APS clustering using all bins within amplicons
8. order-comparison-based interpretation utilities

## Mid-term
10. better overlap / decomposition logic
11. more robust timing evidence
12. gap-aware penalties / annotation
13. sample QC based on APS and RCN sanity

## Longer-term
14. Bayesian posterior ordering and grouping
15. RNA + DNA joint ordering
16. HMM post-hoc module
17. fork-distance / doubling-structure analysis
18. publication-grade report generation

---

## Rich APS clustering (important future idea)
A future extension is to cluster samples not using one APS value per amplicon, but using **all bins inside final amplicon intervals** as features.

Current:
\[
x_s = [APS_{A1}, APS_{A2}, \dots, APS_{AN}]
\]

Future:
\[
x_s = [RCN_{A1,b1}, RCN_{A1,b2}, \dots, RCN_{AN,bX_N}]
\]

Benefits:
- preserves shape
- preserves width
- preserves asymmetry
- can distinguish samples by full amplicon morphology

Caveat:
- long amplicons contribute more features and thus more influence

Possible solutions:
- no correction
- per-amplicon normalization
- hybrid weighting

This is a very strong future direction but not the immediate next implementation.

---

# 8. Important Context and Gotchas

This section records things likely to trip up a new developer or AI.

## 1. This project evolved from two separate programs
Historically:
- single-sample logic
- multistage logic

were separate and later merged.

This means some option exposure and architecture is still transitional.

## 2. The broad detector is not the final biological truth
It may detect broad overlapping loci that actually contain multiple neighboring amplicons.

Do not assume broad initial calls are the final units.

## 3. Timing outputs are tunable, not final truth
Onset stage depends on thresholds and evidence definitions. The current logic works, but is not settled.

## 4. Median vs max matters a lot
Many biological questions depend on whether a value reflects:
- median stage signal
- mean stage signal
- max sample signal

Always be explicit.

## 5. Gaps / masked bins matter
N-gap / zero-bin masking is necessary, but regions near gaps may still be biologically tricky because:
- repeats
- collapsed repeats
- shared kmers / mapping ambiguity

Gap proximity should be tracked, not ignored.

## 6. Twin peaks are a central real problem, not a corner case
The project has at least two known real-data training spots where one broad region contains two separate amplicons.  The 2-peak case is now solved (Phase 3.5+): `overlap.py` correctly splits both spots in both datasets (TP=4, FP=0, FN=0).

The ≥3-peak case is not yet handled (`max_peaks=2` limit).  Do not assume all multi-peak situations are resolved.

## 7. APS scalar ordering and clustering are different
Scalar APS can contradict posterior grouping if used naively.
Do not collapse these into one concept without care.

## 8. Packaging matters
It was explicitly important that:
- permissions are normal
- unzip creates `onionskin/`
- release is self-contained

This is not cosmetic; it is a user requirement.

---

## 9. Constitutional / collapsed-repeat regions are systematic FPs in single-file mode

In **single-file mode** (one bedGraph, no reference), genomic regions that are constitutively elevated across all developmental stages — such as collapsed repeats or regions with shared kmers — appear as false positives because there is no reference track to cancel them out.

**Why they pass detection:** These regions are genuinely elevated above the chromosome-wide median and may have a large z-score.  There is no way to distinguish them from real amplicons using signal strength alone.

**How they are filtered:** The `dBIC_flat_vs_tri` shape metric (Phase 3.10) catches most of them.  A constitutional elevation appears as a flat plateau above the chromosome median — the triangle model earns no BIC gain — score ≈ 0.  Real gradient amplicons score 68–900+.  Default threshold of 50 validated against all 12 regression tests.  CLI: `--shape-score-threshold` (default 50, 0 disables).

**Mode-specific behavior:**
- **Single-file mode**: Most susceptible — no reference cancellation.  Shape filter is the main defense.
- **Ratio-of-ratios mode**: Constitutional elevations largely cancel in the ratio (amplified/reference).  Most such FPs disappear.
- **Multistage mode**: Also uses ratio/normalization; constitutional background is similarly suppressed.

**Known example:** chr II ~14.5 Mb (DS1).  Constitutional elevation, z ≈ 5.8 at 40 kb scale.  `dBIC_flat_vs_tri` score ≈ 0.1 — correctly removed by shape filter.

**Low-confidence amplicons** that look identical to TPs but are not in the primary truth set are documented in `tests/twin_peak_training_data/low_confidence_amplicons.bed`.  These should not be treated as errors — they are biologically real but weakly amplified.

---

## 10. T7 single-file limitation: early developmental stage cannot recover all amplicons

**T7** (single-file, Dataset 2, stage 5, one replicate) expects TP=1, FN=3.  This is **documented behavior, not a bug.**

Stage 5 is the earliest developmental stage in Dataset 2.  Amplification signal is very weak in this stage; only the strongest amplicon (spot 1 amplicon 1, ~6.2 Mb) is reliably detectable from a single replicate.  Recovering all 4 amplicons would require:
- a later stage with stronger signal, or
- replicate averaging (single-stage mode), or
- ratio-of-ratios against an earlier reference

This is a fundamental biological and statistical limitation of single-file mode at early stages.

---

## 11. Chr II ground truth catalog

The user has comprehensive ground truth for all amplicons in the genome, established by:
- Visual inspection in IGV
- Cross-validated with the PuffStep HMM (finds all or most of the same amplicons)
- 14 amplicons experimentally validated across all chromosomes (qPCR, FISH, Southern, etc.)

**Trust rule:** Any coordinates the user provides as ground truth are authoritative. Multiple
source files (BED files, `dev/some-ground-truths.txt`, training data) are subsets that can be
unioned — apparent inconsistencies are sub-sampling artifacts, not contradictions.

### Full-chromosome chr II ground truth

Training data: `tests/full_chrom_training_data/`

**HIGH-confidence by-eye amplicons (9):**
- II-1 (II/7.1): 5,910,689–6,501,954 (qPCR, FISH)
- II/7.2: 7,180,767–7,561,088
- II-2.1 (II/2A): 22,125,449–22,534,503
- II-2.2 (II/2B): 22,678,925–23,275,379 (twin pair with II-2.1)
- II-3 (II/9A): 33,055,654–33,911,430 (best-characterized; qPCR, FISH, Southern, multiple methods; contains the ORC-Core at ~33,470,421–33,470,505 bp)
- II-4: 41,413,288–41,906,860 (qPCR, FISH)
- II-5: 46,060,415–46,458,919
- II-6: 47,143,836–48,353,675 (qPCR; contains putative spot-3 twin pair)
- II-7: 54,625,426–55,089,998 (qPCR)

**LOW-confidence by-eye amplicons (4):** II.17 (~17.4 Mb), II.19 (~19.4 Mb), II.26 (~26.5 Mb), II.31 (~31.2 Mb)

**Known collapsed repeats (chr II — systematic FPs in single-file mode):**
- II:14,500,000 (point estimate; shape filter removes it)
- II:36,478,040–36,583,911 (full region by eye)

**Negative control:** II:10,526,201–12,331,192

**Twin-peak spots:**
- Spot 1: II-1 + II/7.2 (confirmed)
- Spot 2: II-2.1 + II-2.2 (confirmed)
- Putative Spot 3: inside II-6 at II:47,345,000–47,430,000 (low-conf, HMM only) + II:47,565,000–47,925,000 (high-conf); detected in both datasets on first run

**Triple-peak candidate:** Spot 1 region — Amplicon-1.5 at II:6,666,390–6,752,454 sits between II-1 and II/7.2 (HMM only, very short, lowest confidence). Under investigation.

**Known single amplicons (no-split regression test):**
II-3, II-4, II-5, II-7 (not II-6 — it contains putative twin peaks).
File: `tests/full_chrom_training_data/single-amplicons.for-no-split-test.bed`

### Summit precision training data

`tests/summit_training_data/`
- `01-II9A-evidence-based-confinement-boundaries-and-rules.bed` — primary: hierarchical Levels 1–7 for II/9A (II-3). Hard asserts: Level 1–4 at 500 bp, 1–3 at 1000 bp, 1–2 at 5000 bp.
- `02-II2B-evidence-based-confinement-boundaries-and-rules.bed` — supplemental: Level 7 informational for II/2B (II-1)
- `summit_inspector.py` — visualization script for summit estimates vs confinement zones

**Current benchmark baseline (v0.5.12+):**
`make summit` runs DS1 + DS2 × 500/1000/5000 bp (6 manifests). DS2 hires also evaluated.
Best estimator: parabola_mean (parabola over argmax in 7/8 test cases; DS1 500bp parabola_mean = 108 bp from ORC-Core). See ROADMAP Priority 5.9.5 for full results.

---

## 12. Dataset 1 vs Dataset 2 — experimental design and analytical implications

**Dataset 1 (DS1):** Single-animal data. 9 stages, 2 replicates per stage (2 individual animals per stage).

**Dataset 2 (DS2):** Batch approach. 5 stages, 2 replicates per stage (each replicate = 20–30 pooled animals).

### Why DS1 is analytically harder

Three compounding factors explain why DS1 can underperform DS2 with naive parameter choices:

1. **Single-animal variance:** With n=2 per stage and biologically meaningful animal-to-animal
   progression differences, within-stage noise is high. DS2's pooling suppresses this.

2. **Stage coverage extends beyond the amplification initiation window:** DS2 covers stages
   during active initiation (copy number increases monotonically → growth track works well).
   DS1 extends into later stages where amplicons widen but don't get taller, and may show slight
   copy-number decline (possible nuclease degradation as salivary glands autolyze). This dilutes
   growth slope estimates and reduces z-scores for real amplicons.

3. **Normalization interaction:** In late stages with wide flat amplicons, the chrom-median
   normalization can shift, compressing apparent relative copy numbers.

### Current test expectations (v0.5.xx+, `step` default)

`make full` with both datasets, default parameters: **DS1 9/9, DS2 9/9, zero FPs.**
The old 4/9 DS1 result (when `linear` was the default) was caused by the plateau/decline dynamics above — not a bug.

**Diagnostic:** If DS1 ever underperforms again, first check: (1) is `step` still the default method? (2) are the failing amplicons in late stages where plateau/decline would suppress `linear`?

### DS1 vs DS2 reliability

- **DS2** is the primary validation anchor for pipeline development. Reliable, clean, biologically well-behaved.
- **DS1** is the scientifically more important dataset (single-animal resolution enables developmental variation studies), but analytically harder to tune.

### Future directions

1. Restrict growth track fitting to the initiation window in DS1 (first 5–6 stages)
2. Add more replicates per stage to reduce single-animal variance
3. Ratio-of-ratios approach: compare RCN(stage N) / RCN(stage N-1) — more robust under plateau dynamics
4. HMM integration: PuffStep detects structural "step up" relative to flanks rather than growth slope

---

# 9. Agent Bootstrap Instructions

Agent bootstrap instructions are maintained in the authoritative per-agent files at the
repository root. Read those files directly — this section is no longer the source of truth:

- `CLAUDE.md` — Claude Code instructions (primary; all agents should read this)
- `AGENTS.md` — shared conventions for all agents
- `GEMINI.md` — Gemini CLI / GCA-specific conventions
- `.github/copilot-instructions.md` — GitHub Copilot instructions
