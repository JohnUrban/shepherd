# Clustering defaults

Onionskin clusters samples and amplicons in three places. This document covers
**what each clustering surface does**, **the default for every user-facing knob**,
**how to override a default**, and **what to look for in the output to verify
the override took effect**.

For the underlying machinery + step structure, see
[`PIPELINE_SPEC.md`](./PIPELINE_SPEC.md).

---

## Quick orientation

| Clustering surface | What it groups | When you'd want to override defaults |
|--------------------|---------------|--------------------------------------|
| APS sample clustering | Samples grouped by their amplicon-amplification profile | You want to inspect cluster structure on raw amplitudes (no z-score) or vice versa, or you want to disable the reliability/importance weighting to see which amplicons dominate the unweighted distance. |
| HMM fork-trajectory clustering | Per-fork trajectories (one row per fork-stage) grouped by trajectory shape | You want a different cluster count `k`, you want amplitude-preserving rather than shape-only clustering, or you want to keep singleton clusters that the default suppresses. |
| Combined APS + state-path (SAPS) clustering | Samples grouped using BOTH amplitude (APS) and state-path-shape (SAPS) features simultaneously | You want to compare a hybrid amplitude+shape clustering against pure APS or pure SAPS results. |

---

## APS sample clustering

Onionskin computes an Amplification Progression Score (APS) per sample × amplicon
and clusters samples based on similarity of their amplicon profiles. Three
defaults govern how the per-sample APS aggregation is built up before the
clustering distance is computed.

### Defaults

| Knob | Default | What it controls (plain English) | Override flag |
|------|---------|----------------------------------|---------------|
| Per-bin area-excess floor | `on` | Whether bins with RCN below 1 (below diploid baseline) are clipped to zero before being summed into per-locus APS. On = clip to zero (treats below-baseline as "no amplification signal"). Off = keep signed per-bin contributions (negative bins reduce the per-locus APS). | `--aps-area-excess-floor on\|off\|post_sum` |
| Aggregation mode | `weight` | How per-amplicon scores (reliability + importance) shape the sample-level aggregate. `weight` keeps every amplicon in the aggregate but multiplies each amplicon's contribution by its score. `filter` drops low-score amplicons entirely. `both` filters then weights. `none` is the legacy unweighted aggregate (no score awareness). | `--aps-aggregation-mode weight\|filter\|both\|none` |
| Aggregation score | `joint` | Which per-amplicon score to apply (reliability = "how trustworthy is this call?", importance = "how discriminating is this amplicon between samples?"). `joint` multiplies the two — an amplicon must be both trustworthy AND discriminating to weigh heavily. | `--aps-aggregation-score reliability\|importance\|joint` |
| Aggregation score floor | `0.01` | Score threshold for `filter`/`both` modes. Defaults to a tiny underflow floor (effectively keep everything). Raise to `0.5` to make `filter` mode bite seriously. | `--aps-aggregation-score-min FLOAT` |
| Cluster scaling | `both` | Whether the APS feature matrix is z-scored before computing distances. `none` = raw amplitudes. `zscore` = per-feature z-score (every amplicon contributes equally regardless of amplitude). `both` (default) runs both and writes both ordering tables for inspection. | `--aps-scale none\|zscore\|both` |
| Cluster method | `hierarchical` | The clustering algorithm. Currently only `hierarchical` is supported. | `--aps-cluster-method hierarchical` |

### Plain-English rationale per default

- **Per-bin area-excess floor on**: bins below diploid baseline don't represent
  amplification — they're either non-amplifying regions or signal-loss artifacts.
  Letting them subtract from the per-locus APS conflates "no amplification" with
  "anti-amplification."
- **Aggregation mode `weight` + score `joint`**: every amplicon contributes to
  the per-sample aggregate, but contributions scale by reliability (don't trust
  a noisy call) AND importance (don't be dominated by an amplicon that's the
  same in every sample). Independently max-normalized so the joint score stays
  in `[0, 1]`.
- **Score floor `0.01`**: the `weight` default doesn't filter by score so the
  floor only matters in `filter`/`both` modes; the small default is just an
  underflow safety net.
- **Scaling `both`**: clustering on raw amplitudes lets high-amplitude amplicons
  dominate (often the right answer if amplitude is a stage marker). Clustering
  on z-scored features makes every amplicon contribute equally (often the right
  answer if you want shape-based clustering). The default emits both so you can
  compare.

### Worked example

```
onionskin --pipelines all --manifest manifest.tsv --out-dir my-run \
  --aps-aggregation-mode filter \
  --aps-aggregation-score reliability \
  --aps-aggregation-score-min 0.5
```

This switches APS aggregation from "weight every amplicon" to "drop low-reliability
amplicons entirely before summing." After the run, look at:

- `my-run/01-prior/02-growth-model/13-aps/onionskin_aps.tsv` — the per-sample
  aggregate. Compared to a default run, this should have the same number of
  rows (one per sample) but different values, because low-reliability amplicons
  are no longer contributing.
- `my-run/01-prior/02-growth-model/13-aps/locus_contributions.tsv` — per-(sample,
  locus) contributions. Filter rows by `is_filtered_out = True` to see which
  loci were dropped under the new mode.
- The dendrogram + heatmap PNGs in the same directory: cluster structure may
  shift when low-reliability noise is removed.

To compare against the default unweighted aggregate during the same run, also pass:

```
  --aps-emit-unfiltered-aggregates
```

This adds `onionskin_aps_unfiltered.tsv` alongside the score-aware
`onionskin_aps.tsv` so you can spot-check the difference.

---

## HMM fork-trajectory clustering

Onionskin's HMM pipeline can cluster per-fork trajectories — each trajectory is
the time-series of a single fork's amplification level across stages. This is
distinct from APS sample clustering: the rows being clustered are forks (per
amplicon × stage), not samples.

### Defaults

| Knob | Default | What it controls (plain English) | Override flag |
|------|---------|----------------------------------|---------------|
| Cluster count `k` | `auto` | How many trajectory clusters to form. `auto` selects via silhouette score across a small range. Pass an integer to force a specific count. | `--hmm-trajectory-clustering-k auto\|N` |
| Feature scaling | `zscore` | Whether per-fork trajectory features are z-scored before clustering. `zscore` (default) emphasizes shape over amplitude. `none` preserves amplitude differences (so high-amplitude forks group together regardless of shape). | `--hmm-trajectory-clustering-scaling none\|zscore` |
| Singleton-cluster guard | `auto` | Whether to suppress singleton clusters (clusters with one fork). `auto` (default) suppresses when `k` is set explicitly to a value that would produce singletons; off when `k=auto`. Force on or off if you have a strong preference. | `--hmm-trajectory-clustering-singleton-guard on\|off\|auto` |

### Plain-English rationale per default

- **`k=auto`**: the right cluster count is data-dependent (different datasets
  have different fork-trajectory diversity). Silhouette-score selection picks
  a defensible default; force a value if you have prior knowledge.
- **`zscore` scaling**: trajectory shape is usually more informative than absolute
  amplitude for fork-classification. If your forks span a wide amplitude range
  and amplitude IS the meaningful axis, switch to `none`.
- **`auto` singleton guard**: singleton clusters are usually noise. Suppressing
  them aggregates outlier forks into the nearest large cluster, producing
  cleaner downstream interpretation.

### Worked example

```
onionskin --pipelines hmm --manifest manifest.tsv --out-dir my-run \
  --hmm-trajectory-clustering-k 5 \
  --hmm-trajectory-clustering-scaling none
```

This forces 5 clusters using raw (non-z-scored) trajectory features.
After the run, look at:

- `my-run/01-prior/01-hmm/19-clustering/hmm_trajectory_clusters.tsv` — one row
  per fork with assigned cluster label. Should have exactly 5 distinct cluster
  labels.
- `my-run/01-prior/01-hmm/19-clustering/hmm_trajectory_dendrogram.png` —
  visual dendrogram. Cuts at 5 clusters.

---

## Combined APS + state-path (SAPS) clustering

SAPS = State-path APS = APS-style features computed from HMM state-path data
rather than raw RCN amplitudes. SAPS is computed independently of regular APS;
the two feature spaces can also be combined for joint clustering when both are
available (HMM pipeline only).

### Defaults

| Knob | Default | What it controls (plain English) | Override flag |
|------|---------|----------------------------------|---------------|
| SAPS computation | enabled | Whether SAPS is computed during HMM analysis. SAPS produces per-sample × per-amplicon state-path-derived scores at HMM step 17. | `--saps-disable` (suppresses SAPS) |
| HMM state-path label base | `0` | Whether HMM state labels start at 0 (background = state 0) or 1 (background = state 1). 0 (default) is the new convention; 1 is PuffStep-compatible legacy. | `--hmm-statepath-base 0\|1` |

### Plain-English rationale per default

- **SAPS enabled by default**: the cost of computing SAPS is small relative to
  the HMM run as a whole, and the additional feature space is useful for
  combined-clustering inspection.
- **Label base 0**: matches modern convention (background = state 0). Use
  `--hmm-statepath-base 1` only if you need PuffStep-compatible labels for
  downstream tooling that expects the legacy convention.

### Worked example

To skip SAPS computation entirely (useful for time-constrained runs):

```
onionskin --pipelines hmm --manifest manifest.tsv --out-dir my-run \
  --saps-disable
```

After the run, verify:

- `my-run/01-prior/01-hmm/17-saps/` — should NOT exist (or should be empty)
  when `--saps-disable` is passed.
- HMM step 18 timing + step 19 clustering still run; only the SAPS step is
  skipped.

---

## Cross-references

- [`PIPELINE_SPEC.md`](./PIPELINE_SPEC.md) — full pipeline step inventory + the
  underlying machinery for each clustering surface.
- [`APS_CATALOG.md`](./APS_CATALOG.md) — catalog of all APS analysis surfaces
  cross-pipeline (Universal vs pipeline-specific).

---

*Audience: external users running onionskin who want to understand the
clustering knobs available to them. For deliberation history, design
discussions, or implementation details, see the audit log + DECISIONS.md
in `multi-agent/`.*
