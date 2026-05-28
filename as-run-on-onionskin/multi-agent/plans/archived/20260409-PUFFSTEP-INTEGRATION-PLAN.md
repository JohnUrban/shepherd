# PUFFSTEP INTEGRATION PLAN

**Status:** Phase 7 ✓ COMPLETE. Phase 8 ✓ COMPLETE. Phase 9 outlined.
**PuffStep audit:** COMPLETE AND CLOSED (v0.8.02). Nothing further to port.
**PuffStep runtime:** RETIRED (v0.8.03). Zero runtime dependency on PuffStep.
**Created:** 2026-04-08
**Last updated:** 2026-04-09 (post-v0.8.03)
**Authors:** John M. Urban, Claude Code (claude-sonnet-4-6), GitHub Copilot (GPT-5.3-Codex)
**Companion ROADMAP section:** Phase 7 — HMM integration; Phase 8 — HMM feature parity; Phase 9 — HMM synthesis

---

## Vision and scope

**The goal is for onionskin to fully replace PuffStep.** A user should be able to do
anything they could do with PuffStep by running `onionskin.py --pipelines hmm`. Onionskin
is *enveloping* PuffStep — not just calling it.

PuffStep's default parameters (optimized for DS2 5 kb data) are the onionskin defaults, but
that is not the limit of what onionskin should support. Onionskin incorporates all of
PuffStep's flexibility (emission models, training/re-estimation, parameter optimization) and
extends beyond it (true Baum-Welch, additional signal transforms).

The integration is structured in phases:
- **Phase 7:** Full port of PuffStep's validated default pipeline. Goal: exact parity with gold standard.
- **Phase 8:** Full feature parity with PuffStep's complete CLI surface (all emission models, training, etc.).
- **Phase 9:** HMM-derived metrics integrated with onionskin's other modules (APS, timing, etc.).

---

## Guiding principles

1. **PuffStep is a fork, not a dependency.** `PuffStep/` stays untracked, never modified. It is a living reference for porting. Once Phase 8.4 is complete, onionskin will have zero runtime dependency on PuffStep.

2. **Parity-first.** Each migration step is validated against the gold-standard reference before proceeding. `make puff-compare` is the acceptance gate for all Phase 7 work.

3. **Full feature parity is the end state, not optional.** Code we implement but don't yet exercise by default goes into `hmm_core.py`. "Unused" means unused regardless of how the user parameterizes the CLI — not "not the default."

4. **Defaults are validated; extensions are additive.** The DS2 5 kb parameters are the defaults and the parity baseline. New emission models, training options, and normalization protocols are additive — they do not change defaults.

5. **No PuffStep at runtime (Phase 8.4).** When 8.4 is complete, `ONIONSKIN_HMM_STEP6_BACKEND` and `_find_puffstep_root()` become dead code and are removed.

6. **Full genome, not chr II only.** Validation targets the full-genome DS2 5 kb dataset.

---

## Scope decisions (from Phase 7 review, 2026-04-08)

- Steps 9/10 (`extract_summits_like_puffstep`) used `pybedtools`. Replaced with pure-Python
  `onionskin_core/hmm_summits.py` ported from `PuffStep/puffStep_core/findsummits.py`. ✓ DONE
- Sort ordering after protocol 31 output fixed (explicit sort by chrom, start). ✓ DONE
- `_forward_normal` and `_backward_normal` generalized; posterior decoding wired via
  `--hmm-decode-path`. ✓ DONE (v0.7.24)
- All Tier 3 HMM options fully native — no PuffStep required for any user-facing feature. ✓ DONE

### Warning policy (Phase 9+)
- When HMM runs alongside other pipelines, emit warnings if HMM parameters break
  onionskin's nested replication bubble assumptions.
- Suppress warnings when `--pipelines hmm` is used alone.

---

## Reference data and gold standard

| Item | Path | Purpose |
|------|------|---------|
| Gold-standard results | `dev/puffstep/automate/5kb/` | Known-good PuffStep outputs; primary comparison target |
| Full-genome manifest (5 kb) | `dev/datasets/full_genome/batch/manifest.batch-5kb.fofn` | Used by `make puffstep-py`, `make puff-compare` |
| Chr II relative manifest | `tests/full_chrom_training_data/manifest.dataset2.5000bp.tsv` | Used by existing `make full` targets |

**Gold-standard structure:** Steps 06–10, groups 2–5.
**Parity baseline:** `make puff-compare` → 24 files match, 4 skipped (empty raw summit bins).

---

## Output architecture

```
<out-dir>/03-hmm/
    01-mednorm/
    02-unionStats/
    03-removeZeroBins/
    04-chromMedRatioNorm-RCN/
    05-medianSmoothed-RCN/
    06-HMM/
    07-collapsedHMM/
    08-aboveBackground/
    09-summitStates/
    10-summitBins/
```

Invoked via: `onionskin.py --pipelines hmm --manifest <fofn> --out-dir <dir>`

---

## Phase 7 — PuffStep default pipeline integration ✓ COMPLETE

**Goal:** Exact parity with gold-standard PuffStep outputs on DS2 5 kb full-genome data.
All 10 pipeline steps run natively in Python by default. PuffStep is not required at runtime
for the default configuration.

### Phase 7.0 — Reference baseline ✓ COMPLETE
Validated that bash pipeline reproduces gold standard. Established parity baseline.

### Phase 7.1 — Python engine: orchestration wrapper ✓ COMPLETE
`onionskin_core/engines/hmm_engine.py` created. All 10 steps orchestrated in Python
(initially delegating to PuffStep CLI tools via subprocess). Wired into `onionskin.py` via
`--pipelines hmm`. `make puffstep-py` added.

### Phase 7.2 — Validation: `make puff-compare` ✓ COMPLETE
`scripts/compare_puffstep_outputs.py` + `make puff-compare`. Validates steps 06–10 against
gold standard. Exact match on 06–09, float-tolerant on raw step 10 RCN col 6.

### Phase 7.3 — Port PuffStep internals into onionskin_core ✓ COMPLETE

All 10 steps now run natively:

| Step(s) | Ported from | Target | Version |
|---------|------------|--------|---------|
| 8 | `awk '$4>1'` + `mergeBed` | `signal_utils.merge_bed_intervals()` | v0.7.05 |
| 7 | `collapseBedGraphRuns.py` | `signal_utils.collapse_bedgraph_runs()` | v0.7.05 |
| 3 | `removeZeroBins.py` | `signal_utils.remove_zero_bins()` | v0.7.06 |
| 2 | `unionBedGraphStats.py` | `signal_utils.union_bedgraph_stats()` | v0.7.07 |
| 1, 4, 5 | `puffStep.py normalize` (p1, p31, p32) | `signal_utils.py` | v0.7.08 |
| 9, 10 | `puffStep.py summits` | `onionskin_core/hmm_summits.py` | v0.7.22 |
| 6 | `puffStep.py puffcn` | `onionskin_core/hmm_core.py` | v0.7.11 |

### Phase 7.4 — Output layout + CLI polish ✓ COMPLETE
`output_layout.py`, `00-INDEX.md`, `onionskin.py` help text, `README.md`, `PIPELINE_SPEC.md` addendum.

### Phase 7.4.4 — HMM tunability surface (Tier 1 + 2 + 3 surface) ✓ COMPLETE
**Tier 1 (v0.7.16):** `--hmm-special-idx`, `--hmm-init-special`, `--hmm-leave-special-state`,
`--hmm-leave-other`, `--hmm-exp-decay-scale` — wired end-to-end.

**Tier 2 (v0.7.17):** `--hmm-mu-scale`, `--hmm-initialprobs` — wired end-to-end.

**Tier 3 (v0.7.18):** `--hmm-emodel`, `--hmm-kmeans`, `--hmm-iters`, `--hmm-converge`,
`--hmm-constrain-emit`, `--hmm-emitpseudo` — CLI flags surfaced. Native implementation
completed in Phase 8.1. See Phase 8 below.

### Phase 7.4.5 — Parity-first cleanup ✓ COMPLETE

1. **Replaced `pybedtools` in steps 9/10** ✓ — pure-Python `onionskin_core/hmm_summits.py`,
   ported from `PuffStep/puffStep_core/findsummits.py`. Zero external binary dependency.
2. **Fixed sort ordering in protocol 31** ✓ — explicit `sort(key=lambda r: (r[0], r[1]))` added.
3. `pybedtools` import fully removed from `signal_utils.py`.

---

## Phase 8 — Full PuffStep feature parity ◑ IN PROGRESS (8.4 pending)

### 8.1 — Tier 3 HMM tunables: native implementation ✓ COMPLETE

All Tier 3 features are native. No PuffStep required for any user-facing configuration.

| Feature | PuffStep flag | Onionskin status |
|---------|--------------|-----------------|
| Alternative emission models (exponential, poisson, geometric, gamma) | `--emodel` | ✓ NATIVE (v0.7.23) |
| K-means initialization | `--kmeans` | ✓ NATIVE (v0.7.25) |
| Viterbi training (hard EM) | `--iters`, `--converge` | ✓ NATIVE (v0.7.26) |
| Constrained emission re-estimation | `--constrain-emit` | ✓ NATIVE (v0.7.26) |
| Emission pseudocount | `--emitpseudo` | ✓ NATIVE (v0.7.26) |
| Posterior decoding | `--hmm-decode-path` | ✓ NATIVE (v0.7.24) |
| True Baum-Welch (soft EM) | `--hmm-training baum_welch` | ✓ NATIVE (v0.7.27) — **new beyond PuffStep** |

**Implementation notes (for cold start):**
- Emission dispatch: `_log_emit_seq(data, eprobs, emodel)` in `hmm_core.py` — returns vectorized
  (nstates×T) log-emission matrix. `eprobs[0]` = primary param, `eprobs[1]` = secondary.
- K-means: `_kmeans_init_emissions(data, nstates)` — scipy k-means; sorted by mean; sigma floor 1e-6.
- Viterbi training: `_log10_prob_path`, `_update_eprobs/tprobs/iprobs` — hard EM loop matching
  PuffStep's `do_hmm_iter_steps()`.
- Baum-Welch: `_forward_loglik`, `_bw_estep_one` (γ + ξ_sum per chrom), `_bw_mstep_*`,
  `_baum_welch_train` — soft EM; convergence on total log-likelihood. Final decode is separate.
- `--hmm-training viterbi` (default, PuffStep-compatible) vs. `--hmm-training baum_welch` (new).
- In practice on broad amplification domains, both training modes produce nearly identical results.
  Viterbi training is faster; BW is theoretically optimal.
- `make puff-compare` remains OVERALL PASS (24 match, 4 skipped) throughout all additions.

### 8.2 — Posterior decoding CLI exposure ✓ COMPLETE (v0.7.24)
`--hmm-decode-path viterbi|posterior`. Wired: `onionskin.py` → `run_hmm()` → `_step6_hmm()` →
`write_hmm_states_bedgraph(decode_path=...)`. Default: `viterbi`.

### 8.3 — Signal utilities expansion ✓ COMPLETE (v0.8.00)

Comprehensive port of PuffStep normalization protocols and biological signal transforms.
All opt-in functions; no default behavior change.

**`onionskin_core/signal_utils.py` — new functions:**

| Function | PuffStep protocol | Description |
|----------|-----------------|-------------|
| `local_median_normalize_bedgraph` | p30 | Divide by local-window median; `_safe_local_median` fallback (median → trimmed mean → mean → 1.0) |
| `local_mean_smooth_bedgraph` | p34 | Replace with local-window mean |
| `local_trimmed_mean_smooth_bedgraph` | p33 | Replace with trimmed window mean |
| `spmr_normalize_bedgraph` | p22 | Divide by total sum × 1e6 (signal per million reads) |
| `robust_z_normalize_bedgraph` | p18 | `(x - median) / MAD`; falls back to std if MAD == 0 |
| `rank_normalize_bedgraph` | p19 | Fractional rank (average ties) |
| `rank_standardize_bedgraph` | p23 | `(rank - midpoint) / midpoint`; centered 0, ~bounded ±1 |

Internal helpers: `_safe_local_median(x, play_it_safe, extreme)`, `_local_window_apply(values, halfwidth, fn)`.

**`onionskin_core/signal_transforms.py` (new module):**
Self-contained (own I/O helpers; no circular imports). Biological transforms:

| Function | Origin | Description |
|----------|--------|-------------|
| `compute_skew_bedgraph` | CovBedClass.computeSkew | RFD: `(V[i]-V[i-1])/(V[i]+V[i-1])*100`. First bin = 0 per chrom. |
| `compute_percent_change_bedgraph` | CovBedClass.computePercentChange | `(V[i]-V[i-1])/V[i-1]*100`. First bin = 0. |
| `compute_skew_change_bedgraph` | CovBedClass.computeSkewChange | Derivative of RFD/200. Positive peaks = origins, negative = termini. |
| `log2_bedgraph` | PuffStep finalize | Log₂-transform with optional pseudocount |
| `log10_bedgraph` | PuffStep finalize | Log₁₀-transform with optional pseudocount |
| `ratio_bedgraph` | normalize_to_other | `(test+pseudo)/(control+pseudo)` |
| `subtract_bedgraph` | subtract_other | `test - control` |
| `pct_diff_bedgraph` | pct_diff_from_other | `100*(test-control)/control` — asymmetric |
| `pct_skew_bedgraph` | pct_skew_given_other | `100*(test-control)/(|test|+|control|)` — symmetric, bounded ±100 |

**Kernel smoothing (p2/3/4/5/6/13/14/15) ✓ COMPLETE (v0.8.01):**
`gaussian_smooth_bedgraph(input, output, bandwidth)` in `signal_utils.py`. `bandwidth` in bp
(same units as PuffStep `--bandwidth`; default 10000). Converts to bin-index σ via
`sigma = bandwidth / bin_size`, then `scipy.ndimage.gaussian_filter1d(mode="reflect")`.
All 8 ksmooth protocols are fully composable from existing primitives.

**Explicitly skipped — final decisions, do not revisit:**
- **p16** (glocal median ratio norm): marked `INDEV` in PuffStep source; not production-ready.
- **p17** (median norm + scale-to-target-coverage + ratio + re-median norm): obscure pipeline;
  `scale_data` primitive not needed elsewhere; not part of any standard workflow.

### 8.4 — Retire PuffStep runtime dependency ✓ COMPLETE (v0.8.03)

Removed from `hmm_engine.py`:
- `_find_puffstep_root()` and `_puffstep_tools()`
- `ONIONSKIN_HMM_STEP6_BACKEND` env-var check
- `step6_backend` parameter from `run_hmm()` and `_step6_hmm()`
- `tools: Dict[str, str]` parameter from `_step6_hmm()`
- Entire `elif backend == "puffstep":` subprocess block

Also completed as part of 8.4 close-out — full HMM CLI parity:
- `--hmm-exp-decay on|off` (default: on): expose exp_decay as CLI flag.
  Adds `_get_transition_probs_uniform()` to `hmm_core.py` for the off path.
- `--hmm-transprobs MATRIX`: full custom transition matrix; overrides leave_* and exp_decay.
- `--hmm-learn-pseudo FLOAT`: transition pseudocount for Viterbi training. Port of PuffStep --learnpseudo.

PuffStep is reference-only. No runtime dependency anywhere in onionskin.

---

## Phase 9 — HMM synthesis with onionskin modules (OUTLINED)

**Goal:** HMM state paths inform and constrain APS, timing, fork asymmetry, and unified results.

**Note — Step 10 summit bin semantics:** Current step 10 output uses 1-bp point-style intervals only to preserve `make puff-compare` parity. Intended behavior is to emit the full bedGraph bin interval (for example, a 5 kb bin) for the max-col4 summit bin within each summit-state region. Planned fix: switch step 10 to full-bin interval output and then update/rebaseline parity comparison expectations accordingly.

- **9.1:** Per-amplicon doubling boundaries, summit-state refinement, active-window estimates
- **9.2:** APS + HMM integration (state-path timing weights, HMM-boundary constrained fork gradient)
- **9.3:** `--hmm-optimize` — automated model/parameter selection (BIC/cross-validation)
- **9.4:** All-stage HMM with collapsed-repeat exclusion
- **9.5:** Unified results (`04-unified-results/`) aggregating HMM, multistage, per-stage outputs

---

## Current implementation status (v0.8.00)

| Phase | Status |
|-------|--------|
| 7.0 Reference baseline | ✓ COMPLETE |
| 7.1 Python orchestration wrapper | ✓ COMPLETE |
| 7.2 Validation (`make puff-compare`) | ✓ COMPLETE |
| 7.3 Native port (all 10 steps) | ✓ COMPLETE |
| 7.4 Output layout + CLI polish | ✓ COMPLETE |
| 7.4.4 Tunability surface (Tier 1 + 2 + 3) | ✓ COMPLETE |
| 7.4.5 Parity cleanup (pybedtools removed, sort fixed) | ✓ COMPLETE |
| 8.1 Tier 3 HMM native (all emission models, k-means, training) | ✓ COMPLETE |
| 8.1d True Baum-Welch (new beyond PuffStep) | ✓ COMPLETE |
| 8.2 Posterior decoding CLI | ✓ COMPLETE |
| 8.3 Signal utilities + signal transforms module | ✓ COMPLETE |
| 8.4 Retire PuffStep runtime + full CLI parity | ✓ COMPLETE (v0.8.03) |
| 9.1–9.5 HMM synthesis | OUTLINED |

**Ongoing validation:** `make puff-compare` OVERALL PASS (24 match, 4 skipped) throughout.

---

## Make target summary

| Target | Description |
|--------|-------------|
| `make puffstep-II` | Run bash pipeline on full-genome DS2 5kb manifest (requires PuffStep + bedtools) |
| `make puffstep-py` | Run Python HMM engine on same manifest |
| `make puff-compare` | Compare run against gold standard at steps 06–10 |

`make puff-compare RUN=<path>` accepts an explicit run directory.

---

## Open items

| ID | Item | Status |
|----|------|--------|
| O4 | pybedtools dependency in steps 9/10 | ✓ RESOLVED (v0.7.22 — hmm_summits.py) |
| O5 | Sort ordering in protocol 31 output | ✓ RESOLVED (v0.7.25) |
| O6 | Tier 3 native implementation | ✓ RESOLVED (v0.7.23–v0.7.27) |
| O7 | Kernel smoothing port (R ksmooth → scipy) | ✓ RESOLVED (v0.8.01 — gaussian_smooth_bedgraph) |
| O10 | p16 glocal median ratio norm | SKIPPED — INDEV in PuffStep, will not port |
| O11 | p17 scale-to-target-coverage pipeline | SKIPPED — obscure, not needed, will not port |
| O12 | winsorize_bedgraph | ✓ RESOLVED (v0.8.02) |
| O13 | chromosome_median_normalize_bedgraph (single-sample primitive) | ✓ RESOLVED (v0.8.02) |
| O14 | Final comprehensive PuffStep audit (all files) | ✓ COMPLETE (v0.8.02) — nothing further to port |
| O8 | Phase 8.4 PuffStep retirement | USER-GATED |
| O9 | Empirical Viterbi vs. BW comparison on real data | USER-CURIOSITY — pending |

---

## What is NOT happening

- PuffStep is not modified.
- PuffStep is not a tracked dependency (no `requirements.txt` reference).
- The bash pipeline (`puffStepPipeline.parallel.sh`) stays as a reference.
- We do not try to keep onionskin's HMM engine and PuffStep in sync after porting.
- We do not port everything at once — each step is validated before proceeding.
- Phase 9 does not begin before Phase 8 is complete.

---

## Dependency policy

**Do not introduce dependencies that require external binaries or non-Python system tools.**

Permitted:
- `numpy`, `pandas`, `scipy`, `matplotlib`, `seaborn`, standard library — always fine

Not permitted:
- `pybedtools` — wraps `bedtools`; requires an external binary
- `pysam` — wraps `samtools`/`htslib`; requires an external binary
- Any library that shells out to `bedtools`, `samtools`, `tabix`, `bgzip`, `bedops`, etc.

**Rationale:** Onionskin is a pure-Python package. Users should not need to install
bioinformatics command-line tools to use it. PuffStep's own source (`findsummits.py`,
`CovBedClass.py`, etc.) is pure Python and is the correct reference for any porting work.
