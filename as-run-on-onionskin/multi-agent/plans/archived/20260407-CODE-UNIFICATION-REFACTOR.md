# Code Unification Refactor Plan

**Status:** Phases A–D DONE (v0.5.57–v0.5.60). Phase E pending.

**Strategic goal:** Consolidate duplicated code across the three engine files
(`engines/single.py`, `engines/multistage.py`, `single_engine.py`) so that any
logic developed for one is immediately available to all. Required before HMM
integration to avoid integrating into three separate code paths.

**Final vision (after Phase E):** All three engines share signal utilities,
detection algorithms, and refinement estimators from shared modules.
`engines/single.py` and `engines/multistage.py` become thin wrappers with only
their unique logic. Adding any new estimator (e.g., HMM-based) requires
writing it once and wiring it into both engines' output schemas.

---

## Background: What exists today

### Files involved

| File | Role |
|------|------|
| `onionskin_core/engines/single.py` | Original standalone v2 caller. Has its own copies of ALL signal utilities plus `stage2_score`, `boundaries_and_peak`, `blockiness_metrics`. Has a `main()` for CLI. Self-contained — no imports from onionskin_core. |
| `onionskin_core/engines/multistage.py` | Original multistage monolith. Also has its own copies of all signal utilities PLUS evidence track scoring (linear_growth_z, isotonic, step, unimodal). Has `main()`. |
| `onionskin_core/single_engine.py` | Multi-sample single-stage pipeline. Wraps `engines/single.py` via `detection.py → common.py → engines/single.main()`. Has its own copies of `_smooth_mean`, `_dedup_by_peak_proximity`, `_dBIC_flat_vs_tri`, `_shape_filter_calls`. |
| `onionskin_core/detection.py` | Currently 12 lines — just calls `load_reference_single()` to invoke `engines/single.main()`. |
| `onionskin_core/refinement.py` | Shared refinement utilities. Contains `read_bedgraph`, `detect_bin_size`, `per_chrom_median`, `add_norm_log2` (wrapper via `load_reference_single`), `fit_local_parabola`, `refine_summit_parabola`, `stage2_refine` (wrapper via `load_reference_single`), `sliding_offset_profile`, `refine_origin_sliding_offset`. |
| `onionskin_core/common.py` | Dispatch layer + shared utilities. Contains `smooth_rcn_median`, `genomic_sort_key`, `load_reference_single()`, `load_reference_multistage()`. Module-level imports from both engines cause circular import. |
| `onionskin_core/signal_utils.py` | Does NOT exist yet. Will be created in Phase B. |
| `onionskin_core/io.py` | `Sample` dataclass + `read_manifest`. Good already. |
| `onionskin_core/modeling.py` | `logbf_from_delta_bic`, `bed_score_from_logbf`. Good already. |

### The circular import chain (pre-Phase A)

```
refinement.py → common.py → engines/multistage.py (module-level import)
                                   ↑
                      Cannot import from refinement.py
```

Root cause: `common.py` imports both engine modules at module level. When
`engines/multistage.py` tried to import `refinement.py`, Python detected a
cycle. Workaround was deferred (inside-function) imports in `multistage.py`
for `refine_origin_sliding_offset` (added in v0.5.53–v0.5.54).

### Key duplicated functions

| Function | engines/single.py | engines/multistage.py | single_engine.py |
|---|---|---|---|
| `read_bedgraph` | ✓ | ✓ | via refinement.py |
| `detect_bin_size` | ✓ | ✓ | via refinement.py |
| `per_chrom_median` | ✓ | ✓ | via refinement.py |
| `add_norm_log2` | ✓ | ✓ (as `normalize_log2_chrom_median`) | via refinement.py |
| `rolling_median` | ✓ | ✓ | — |
| `smooth_mean` | ✓ | ✓ | `_smooth_mean` (same code) |
| `robust_z` | ✓ **returns (z, med, sigma)** | ✓ **returns array only** | — |
| `conv1d` | ✓ | ✓ | — |
| `boxcar_kernel` | ✓ | ✓ | — |
| `find_local_maxima` | ✓ (no length guard) | ✓ (has `len(x)<3` guard) | — |
| `bic` | ✓ | ✓ | — |
| `r2_from_xy` | — | ✓ | — |
| `nice_name` | — | ✓ | — |
| `stage1_mean_shift` | ✓ | ✓ | — |
| `fit_block` | ✓ | ✓ | — |
| `fit_bump_triangle_basis` | ✓ | ✓ | — |
| `_dedup_by_peak_proximity` | — | ✓ | ✓ (different name) |
| `parse_manifest` | — | ✓ (large impl) | via io.py |
| `_dBIC_flat_vs_tri` | — | — | ✓ single_engine.py only |
| `_shape_filter_calls` | — | — | ✓ single_engine.py only |

**Critical API divergence:** `robust_z` returns different types in the two engines:
- `engines/single.py`: `(z, med, sigma)` — 3-tuple; callers do `z, _, _ = robust_z(s)`
- `engines/multistage.py`: just `z` — array; callers do `z = robust_z(s)`
Must be reconciled before sharing call sites (see Phase B).

---

## Phase A — Break the circular import chain

**Status: DONE** (v0.5.57)

**What was done:**
1. In `common.py`: removed unused top-level `from onionskin_core.engines.multistage import main as _multistage_main` import.
2. In `common.py`: converted `load_reference_single()` and `load_reference_multistage()` from returning pre-imported module-level references to using lazy imports inside the function bodies. This eliminates all module-level imports from `common.py` to either engine, breaking the circular chain.
3. In `engines/multistage.py`: promoted deferred `from onionskin_core.refinement import refine_origin_sliding_offset` imports (which were workarounds for the circular import) to proper module-level imports.
4. Ran `make test`, `make toy`, `make single` — all pass.

**Result:** `engines/multistage.py` can now import from `refinement.py` at module level. The lazy-import pattern in `common.py` is the permanent design going forward.

---

## Phase B — Create `signal_utils.py` (shared signal processing)

**Status: DONE** (v0.5.58)

**Goal:** Extract pure signal processing utilities into a single shared module
with no imports from within onionskin_core (only numpy, pandas, math).

### Step B1: Create `onionskin_core/signal_utils.py`

Canonical implementations (resolving all divergences):

```python
# signal_utils.py

def rolling_median(arr: np.ndarray, window: int) -> np.ndarray:
    # identical in both engines — use either
    s = pd.Series(arr)
    return s.rolling(window=window, center=True, min_periods=max(5, window//5)).median().to_numpy()

def smooth_mean(arr: np.ndarray, window: int) -> np.ndarray:
    # identical in both engines and single_engine._smooth_mean
    s = pd.Series(arr)
    return s.rolling(window=window, center=True, min_periods=max(3, window//5)).mean().to_numpy()

def robust_z(x: np.ndarray) -> np.ndarray:
    # CANONICAL: returns array only (multistage.py version)
    # engines/single.py returns 3-tuple — callers in single.py need updating (see Phase C)
    x = np.asarray(x, dtype=float)
    med = np.nanmedian(x)
    mad = np.nanmedian(np.abs(x - med))
    sigma = 1.4826 * mad if mad > 0 else np.nanstd(x)
    if not np.isfinite(sigma) or sigma <= 0:
        sigma = np.nanstd(x) + 1e-9
    z = (x - med) / sigma
    z[~np.isfinite(z)] = 0.0
    return z

def conv1d(arr: np.ndarray, kernel: np.ndarray) -> np.ndarray:
    return np.convolve(arr, kernel, mode="same")

def boxcar_kernel(halfwidth: int) -> np.ndarray:
    w = np.ones(2 * halfwidth + 1, dtype=float)
    w /= w.sum()
    return w

def find_local_maxima(x: np.ndarray, min_distance: int = 10) -> np.ndarray:
    # Use multistage.py version (has len(x)<3 guard)
    x = np.asarray(x)
    if len(x) < 3:
        return np.array([], dtype=int)
    peaks = np.where((x[1:-1] > x[:-2]) & (x[1:-1] >= x[2:]))[0] + 1
    if len(peaks) == 0:
        return np.array([], dtype=int)
    order = np.argsort(x[peaks])[::-1]
    selected = []
    taken = np.zeros(len(x), dtype=bool)
    for idx in order:
        p = peaks[idx]
        lo = max(0, p - min_distance); hi = min(len(x), p + min_distance + 1)
        if taken[lo:hi].any(): continue
        selected.append(p)
        taken[lo:hi] = True
    return np.array(sorted(selected), dtype=int)

def bic(sse: float, n: int, k: int) -> float:
    sse = max(float(sse), 1e-12)
    return n * math.log(sse / n) + k * math.log(n)

def r2_from_xy(x: np.ndarray, y: np.ndarray) -> float:
    # Currently only in multistage.py
    mask = np.isfinite(x) & np.isfinite(y)
    if mask.sum() < 3: return float("nan")
    x = x[mask]; y = y[mask]
    x0 = x - x.mean()
    denom = float(x0 @ x0)
    if denom <= 0: return float("nan")
    beta = float((x0 @ (y - y.mean())) / denom)
    yhat = y.mean() + beta * x0
    ssr = float(np.sum((yhat - y.mean())**2))
    sst = float(np.sum((y - y.mean())**2))
    return float(ssr / sst) if sst > 0 else float("nan")

def nice_name(chrom: str, start: int, end: int) -> str:
    # Currently only in multistage.py
    a = round(start / 1e6, 1); b = round(end / 1e6, 1)
    return f"{chrom}:{a:.1f}-{b:.1f}"
```

### Step B2: Update `single_engine.py`
- Replace `_smooth_mean` with `from .signal_utils import smooth_mean`

### Step B3: Update `refinement.py`
- It doesn't directly use most of these, but confirm no redundant local copies.

### Step B4 (optional, defer to Phase C): Update both engine monoliths to import from `signal_utils`
- In `engines/single.py`: add `from onionskin_core.signal_utils import ...` at top; keep local definitions for safety until Phase C removes them.
- In `engines/multistage.py`: same.

**Do NOT remove local copies from engines yet** — that's Phase C. This phase only creates the canonical home.

**Tests to run:** `make test`, `make toy`, `make single`

---

## Phase C — Expand `detection.py` with shared detection algorithms

**Status: DONE** (v0.5.59)

**Goal:** Move the shared detection-layer algorithms out of both engine
monoliths and into `detection.py`, so adding a new detection method means
writing it once.

### Step C1: Add to `detection.py`

Move from both engines (they are currently identical or near-identical):

```python
# detection.py additions

from .signal_utils import rolling_median, smooth_mean, robust_z, conv1d, boxcar_kernel, find_local_maxima, bic
from dataclasses import dataclass

@dataclass
class Candidate:
    chrom: str; start: int; end: int; peak: int; score_z: float; scale_bins: int

def stage1_mean_shift(df_chr, bin_size, halfwidths_kb, trend_kb, z_thresh) -> List[Candidate]:
    # Use multistage.py version (robust_z returns array, not 3-tuple)
    # ... (copy from multistage.py)

def fit_block(y, idx_s, idx_e) -> Tuple[float, int, Dict]:
    # identical in both engines — copy either

def fit_bump_triangle_basis(y, idx_s, idx_p, idx_e) -> Tuple[float, int, Dict]:
    # identical in both engines — copy either

def dedup_calls_by_peak_proximity(df, peak_col, score_col, peak_dist_bp) -> pd.DataFrame:
    # Canonical; currently exists as _dedup_calls_by_peak_proximity (multistage)
    # and _dedup_by_peak_proximity (single_engine). Same logic, different names.
```

**Note on `robust_z` divergence:** `engines/single.py`'s `stage1_mean_shift` calls
`z, _, _ = robust_z(s)` (3-tuple unpack). After using the canonical `robust_z`
(array return), update the `stage1_mean_shift` in `detection.py` to `z = robust_z(s)`.
Then update `engines/single.py`'s local `stage1_mean_shift` similarly (or import from detection.py).

### Step C2: Update `engines/multistage.py`
- Import `Candidate`, `stage1_mean_shift`, `fit_block`, `fit_bump_triangle_basis`, `dedup_calls_by_peak_proximity` from `detection.py`
- Remove local copies of those functions

### Step C3: Update `engines/single.py`
- Same imports; update `stage1_mean_shift` for the `robust_z` API change
- Remove local copies

### Step C4: Update `single_engine.py`
- Replace `_dedup_by_peak_proximity` with `from .detection import dedup_calls_by_peak_proximity`

**Tests to run:** `make test`, `make toy`, `make single`, `make twin`, `make full`

---

## Phase D — Expand `refinement.py` with scoring and merge `parse_manifest`

**Status: DONE** (v0.5.60)

**Goal:** Move the shape-scoring and boundary functions that currently live only
in `engines/single.py` into `refinement.py` so multistage can use them directly.
Also merge the duplicate manifest parsing.

### Step D1: Move from `engines/single.py` to `refinement.py`

```
boundaries_and_peak(df_chr, peak_pos, bin_size, trend_kb, smooth_kb, peak_search_kb, window_kb) -> Dict
blockiness_metrics(rs_seg) -> Dict
stage2_score(df_chr, peak_pos, bin_size, ...) -> Dict
```

These are currently called by `stage2_refine` in `refinement.py` via `load_reference_single()`. After this move, `stage2_refine` calls them directly. Remove the `load_reference_single()` call in `stage2_refine`.

**Impact on `detection.py`:** `run_singlefile_caller` currently calls `engines/single.main()` which orchestrates disk I/O + calling. After this phase, consider whether `run_singlefile_caller` should instead call the individual detection functions directly. This is the largest architectural decision in the refactor — whether to keep `engines/single.main()` as the disk-based entry point or replace it with a function-call API. **Recommendation: keep `engines/single.main()` as the disk-based entry point** for now; `run_singlefile_caller` remains unchanged. This bounds risk.

### Step D2: Move `_dBIC_flat_vs_tri` and `_shape_filter_calls` from `single_engine.py` to `refinement.py`

These are shape scoring functions that should be available to multistage too:
- `_dBIC_flat_vs_tri(seg, idx_s, idx_p, idx_e, baseline, strict_bic)` → make public
- `_shape_filter_calls(df_calls, bg_path, min_score, ctx_bins, strict_bic)` → make public

### Step D3: Unify `parse_manifest`

`engines/multistage.py` has a large `parse_manifest()` and `io.py` has `read_manifest()` — both parse the same manifest format but with different implementations. After reading both:
- `io.py`'s `read_manifest` is simpler and more tested (used by io.Sample objects).
- `engines/multistage.py`'s `parse_manifest` returns a DataFrame (different API).
- Multistage's version is called internally by `main()`.
- Resolution: keep both for now but note the redundancy in a TODO comment.
  Full merge is a later cleanup.

**Tests to run:** `make test`, `make toy`, `make single`, `make twin`, `make full`

---

## Phase E.0 — Output directory numbered-prefix refactor (prerequisite to Phase E)

**Status: NOT STARTED**

**Goal:** Implement the finalized hierarchical pipeline-arm prefix scheme in
`build_output_layout`, add `00-INDEX.md`, add `00-signal-files/`, eliminate `others/`,
and fix the ambiguous `stage_medians/` name. See DECISIONS.md [2026-04-07] for rationale.

**Single primary file:** `onionskin_core/output_layout.py`

### Final directory scheme

```
<grouping_dir>/
  00-INDEX.md                        flat file guide; not a directory
  00-signal-files/                   shared: per-sample bedGraphs + genome stage medians
  01a-stage_median_within_calls/     multistage internal (overlap resolution + timing)
  01b-summary_bedgraphs/             multistage summary tracks
  01c-summits/                       multistage summit BEDs from engine
  01d-summit_refinement/             summit estimate TSVs/BEDs (replaces others/)
  01e-aps/                           APS results
  01f-plots/                         all diagnostic plots
  01g-notebook/                      Jupyter notebooks
  02-per-stage-mean-shift/           Phase E (created in Phase E, not here)
```

No `others/` — no generic dump directory, ever.

### Step E.0.1: Update `build_output_layout` in `output_layout.py`

Layout dict key renames:

| Old key | New key | Old path | New path |
|---|---|---|---|
| `stage_medians_dir` | `stage_medians_dir` (keep key) | `stage_median_within_calls/` | `01a-stage_median_within_calls/` |
| `summary_dir` | `summary_dir` | `summary_bedgraphs/` | `01b-summary_bedgraphs/` |
| `summits_dir` | `summits_dir` | `summits/` | `01c-summits/` |
| (new) | `summit_refinement_dir` | — | `01d-summit_refinement/` |
| `aps_dir` | `aps_dir` | `aps/` | `01e-aps/` |
| `plots_dir` | `plots_dir` | `plots/` | `01f-plots/` |
| `samples_dir` | `samples_dir` | `samples/` | `00-signal-files/samples/` |
| `genome_stage_medians_dir` | `genome_stage_medians_dir` | `stage_medians/` | `00-signal-files/genome_stage_medians/` |
| `notebook_dir` | `notebook_dir` | `notebook/` | `01g-notebook/` |
| `others_dir` | *removed* | `others/` | *gone* |

Note: `samples/` and `genome_stage_medians/` move inside `00-signal-files/`. The layout
dict values change but the key names can stay the same (less churn at call sites).

### Step E.0.2: Generate `00-INDEX.md` from `build_output_layout`

Write once at layout creation time. Template (static, not dynamic):
```markdown
# Output directory index

| Directory | Contents | Note |
|---|---|---|
| `00-signal-files/` | Per-sample bedGraphs; genome-wide stage medians | Shared across all pipelines |
| `01a-stage_median_within_calls/` | Stage median bedGraphs within called intervals | Internal: used by overlap resolution + timing |
| `01b-summary_bedgraphs/` | Summary bedGraphs (model peak, base fold, log2) | |
| `01c-summits/` | Summit BED files from multistage engine | |
| `01d-summit_refinement/` | Summit estimate TSVs and BEDs | |
| `01e-aps/` | APS matrices, cluster assignments, ordering | |
| `01f-plots/` | All diagnostic plots (profile, shape, summit, QC, overlap) | |
| `01g-notebook/` | Jupyter notebooks | |
| `02-per-stage-mean-shift/` | Per-stage independent mean-shift calls (if run) | Phase E |
```

### Step E.0.3: Update `organize_outputs` routing

- Route `*_summit_estimates.bed` and `*_*_summit_estimates.tsv` to `summit_refinement_dir`
- Remove all `others_dir` references
- Update `stage_median_within_calls` glob pattern to match new `01a-` path

### Step E.0.4: Update all layout key consumers

Grep for every layout key reference (path strings change; key names mostly stable):
- `onionskin.py` — `stage_medians_dir`, `genome_stage_medians_dir`, `samples_dir`, `aps_dir`, `plots_dir`, `notebook_dir`, `others_dir`
- `onionskin_core/aps.py` — hardcoded `'stage_medians'` path at line 1150
- `onionskin_core/timing.py` — hardcoded `'stage_median_within_calls'` at line ~876; must match new `01a-` name
- `onionskin_core/readme.py` — any hardcoded subdir names

### Step E.0.5: Remove `others_dir` everywhere

Search `others_dir` across all files; delete dict entry and all references.

**Tests after Phase E.0:** `make test`, `make toy`, `make single`, `make twin`

---

## Phase E — Per-stage mean-shift outputs (new feature)

**Status: DONE (v0.5.62)**

**Goal:** Run single-mode detection/scoring on each stage's aggregate track within the
multistage pipeline, with full shape filtering, BED outputs for IGV, and cross-stage
unified outputs. This creates the `02-per-stage-mean-shift/` pipeline arm.

**Architecture decisions (settled 2026-04-07):**
- Output location: `02-per-stage-mean-shift/` (added in Phase E.0 layout)
- No changes to existing multistage output schema — fully additive
- Posterior re-run: automatic (multistage engine called again; stage aggregates regenerated)
- HiRes: base resolution only for Phase E; hires refinement deferred to Phase E.2
- Shape filter: YES — apply `shape_filter_calls`; rejected calls are emitted separately
- CLI flag: `--pipelines growth,per-stage,hmm` (with `all` shortcut); `--skip-per-stage` is convenience alias for `--pipelines growth`
- Three-pipeline full restructuring (`01-multistage-growth/`, etc.) deferred to Phase 7 entry
- Unified results only generated when 2+ pipelines run
- Naming note: `10-independent_stage_calls` saved as alternative name for future reference

**Dependencies:** Phases A–D (done); Phase F (output layout refactor)

### Step E1: Add `run_per_stage_mean_shift()` to `single_engine.py`

New function, not modifying existing paths. Signature:
```python
def run_per_stage_mean_shift(
    sample_tracks: Dict[str, Dict[str, pd.DataFrame]],   # {sid: {chrom: df}}
    sample_ids: List[str],
    stages: np.ndarray,                                   # integer stage per sample
    chroms: List[str],
    bin_size: int,
    out_dir: str,                                         # 10-per-stage-mean-shift/ path
    out_prefix: str,                                      # base prefix for file names
    # single-mode parameters (mirror CLI defaults)
    trend_kb: int = 2000,
    smooth_kb: int = 80,
    z_thresh: float = 3.0,
    halfwidths_kb: List[int] = None,
    peak_search_kb: int = 250,
    window_kb: int = 700,
) -> None
```

For each unique stage value:
1. Compute stage median track across all samples at that stage (reuse `_stage_medians`)
2. Run `stage1_mean_shift` on the aggregate
3. Run `stage2_score` per candidate
4. Run `shape_filter_calls` → split into passing / rejected / collapsed_repeats
5. Write per-stage outputs (TSV + BED)

After all stages:
6. Dedup passing calls across stages → `unified_stage_calls.tsv/.bed`
7. Dedup rejected calls → `unified_rejected_calls.tsv`
8. Dedup collapsed repeats → `unified_collapsed_repeats.tsv/.bed`

Dedup strategy: overlap-based merge; `stages_present` column lists which stage integers
contributed to each interval.

### Step E2: Wire into `engines/multistage.py` main()

After the stage median tracks are computed (currently line ~1037 area), call:
```python
if not args.skip_per_stage:
    run_per_stage_mean_shift(
        sample_tracks=base_tracks,
        sample_ids=sample_ids,
        stages=stages,
        chroms=chroms,
        bin_size=base_bin,
        out_dir=layout["per_stage_mean_shift_dir"],
        out_prefix=os.path.basename(layout["out_prefix"]),
        ...
    )
```

### Step E3: Add `--skip-per-stage` CLI flag

In `onionskin.py`'s `build_parser()`, under "Advanced" group:
```python
ap.add_argument("--skip-per-stage", action="store_true",
    help="Skip per-stage mean-shift pipeline (10-per-stage-mean-shift/). "
         "Implies no unified results.")
```

### Step E4: Per-stage output schema

All files in `10-per-stage-mean-shift/`:

```
stage1_calls.tsv                       passing calls (stage2_score metrics)
stage1_calls.bed                       BED for IGV
stage1_summits.bed                     summit positions
stage1_rejected_calls.tsv             gap/quality rejects
stage1_putative_collapsed_repeats.tsv  shape filter rejects
stage1_putative_collapsed_repeats.bed
stage2_calls.tsv
... (per stage)
unified_stage_calls.tsv               cross-stage deduplicated union + stages_present
unified_stage_calls.bed
unified_rejected_calls.tsv
unified_collapsed_repeats.tsv
unified_collapsed_repeats.bed
```

Columns in `stageN_calls.tsv`:
`chrom`, `start`, `end`, `peak_pos`, `stage_val`, `stage1_score_z`, `stage1_scale_bins`,
`peak_value`, `delta_BIC_block_minus_bump`, `blockiness_*`, `bump_vs_block`

### Step E5: Add to `00-INDEX.md` template

Document `10-per-stage-mean-shift/` with a brief description of the approach and output file list.

**Tests after Phase E:** `make test`, `make toy`, `make single`, `make twin`, `make full`, `make full-twin`

---

## Testing checkpoints

| After phase | Required tests |
|---|---|
| A | `make test`, `make toy`, `make single` |
| B | `make test`, `make toy`, `make single` |
| C | `make test`, `make toy`, `make single`, `make twin`, `make full` |
| D | `make test`, `make toy`, `make single`, `make twin`, `make full` |
| E.0 | `make test`, `make toy`, `make single`, `make twin` |
| E | `make test`, `make toy`, `make single`, `make twin`, `make full`, `make full-twin` |
| E.2 | `make test`, `make toy`, `make single`, `make twin`, `make full`, `make full-twin` |

---

## File ownership after all phases

| Module | Owns |
|---|---|
| `signal_utils.py` | `rolling_median`, `smooth_mean`, `robust_z` (array API), `conv1d`, `boxcar_kernel`, `find_local_maxima`, `bic`, `r2_from_xy`, `nice_name` |
| `detection.py` | `Candidate`, `stage1_mean_shift`, `fit_block`, `fit_bump_triangle_basis`, `dedup_calls_by_peak_proximity`, `run_singlefile_caller` |
| `refinement.py` | `read_bedgraph`, `detect_bin_size`, `per_chrom_median`, `add_norm_log2`, `fit_local_parabola`, `refine_summit_parabola`, `stage2_refine`, `boundaries_and_peak`, `blockiness_metrics`, `stage2_score`, `sliding_offset_profile`, `refine_origin_sliding_offset`, `dBIC_flat_vs_tri`, `shape_filter_calls` |
| `common.py` | `smooth_rcn_median`, `genomic_sort_key`, `load_reference_single()` (lazy), `load_reference_multistage()` (lazy) |
| `io.py` | `Sample`, `read_manifest` |
| `modeling.py` | `logbf_from_delta_bic`, `bed_score_from_logbf` |
| `engines/single.py` | `main()` CLI entry point, unique v2 shape metrics only |
| `engines/multistage.py` | `main()` CLI entry point, evidence track scoring (`linear_growth_z`, `isotonic_growth_score`, etc.), width progression modeling, BIC likelihood stats |
| `single_engine.py` | Multi-sample aggregation, bootstrap, orchestration |

---

## Cross-references
- ROADMAP.md: Phase 6 (Code Unification Refactor)
- BRAINSTORM.md: `[2026-04-07] Multi-engine integration and HMM meta-analysis vision`
- CHANGELOG.md: entries starting at v0.5.57
