# APS Catalog

Master catalog of APS outputs across Growth, RMS, and HMM as of Phase 15 cycle 15.8a.

## Universal outputs

- `onionskin_aps.tsv`
- per-locus contribution TSVs (`locus_contributions.tsv` / equivalent)
- APS matrix TSVs:
  `aps_matrix_raw.tsv`, `aps_matrix_zscore.tsv`, plus feature-specific matrix names
- `aps_amplicon_importance.tsv`
- scalar ordering TSVs and APS order tables
- APS cluster/order TSVs:
  `aps_clusters.tsv`, `aps_clusters_raw.tsv`, `aps_clusters_zscore.tsv`,
  `aps_dendrogram_order_raw.tsv`, `aps_dendrogram_order_zscore.tsv`
- APS heatmap plots under `plots/aps/`
- APS score tracks / aggregation-score sidecars

## Pipeline-specific exceptions

- Growth:
  includes overlap-resolution plots and growth-curve surfaces that do not have direct RMS/HMM analogs in this cycle.
- RMS:
  peak-summary outputs now live in `03-rcn-mean-shift/peak-summary/`.
- HMM:
  APS parity now includes amplicon importance, scalar orderings, APS score tracks, and the heatmap subset of APS plots.
  Dendrogram plots remain intentionally absent until the linkage-matrix capture work lands.

## Canonical summit inputs used by APS

- Growth APS reads refined summit positions from `*_origins.tsv`; curated summit strategies replace `final_summit_bp` in place when enabled.
- RMS APS reads `08-multistage-unification/{out_prefix}_origins.tsv` when present; otherwise it falls back to the unified-call peak surface.
- HMM APS reads `14-multistage-unification/all_trajectories.multistage_unification.tsv` and prefers `final_summit_bp` over the combined-interval midpoint.
- The `diagnostic-summits/` TSV+BED files under each summit-refinement step are comparison/evaluation surfaces, not APS outputs.

## Cross-pipeline parity

| Output | Growth | RMS | HMM | Notes |
|---|---|---|---|---|
| `onionskin_aps.tsv` | ✓ | ✓ | ✓ | Canonical APS table |
| Per-locus contributions TSV | ✓ | ✓ | ✓ | Naming varies slightly by pipeline |
| APS raw/zscore matrices | ✓ | ✓ | ✓ | Feature-specific names may also be emitted |
| `aps_amplicon_importance.tsv` | ✓ | ✓ | ✓ | HMM landed in cycle 15.8a |
| Scalar ordering TSVs | ✓ | ✓ | ✓ | HMM landed in cycle 15.8a |
| APS score tracks | ✓ | ✓ | ✓ | HMM parity verified in cycle 15.8a |
| APS raw/zscore heatmaps | ✓ | ✓ | ✓ | HMM landed via the no-dendrogram subset |
| APS scalar-ordered heatmaps | ✓ | ✓ | ✓ | HMM landed in cycle 15.8a |
| APS focus-locus heatmaps | ✓ | ✓ | ✓ | HMM landed in cycle 15.8a |
| APS dendrogram plots | ✓ | ✓ | ✗ | HMM deferred pending linkage-matrix capture |

## HMM by-eye evaluation entry points

- `tests/full_chrom_training_data/amplicons.HMM.bed`
- `tests/summit_training_data/II9A.HMM.bed`
- `tests/summit_training_data/II2B.HMM.bed`
- `tests/summit_training_data/grab-hmm-information.sh`
- `make summit-hmm`

## Structural gaps still documented

- HMM APS dendrograms are the remaining intentional parity gap.
- The missing HMM dendrogram surface is tracked under `CROSS_PIPELINE_UNIFICATION_SOUP` Item 3.
