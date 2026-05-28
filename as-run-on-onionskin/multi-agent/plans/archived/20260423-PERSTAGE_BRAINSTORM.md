## DEPRECATED
### This was a planned phase that other phases ended up consuming before we got there. 
- Intended: completeness to make it capable of producing its own analyses downstream of amplicon calling, including summit detection, summit refinement, APS, posterior ordering/grouping/manifest, timing, fork tracking, fork age, etc.
- Phase 13 delivered (8-step layout, APS, clustering, posterior manifests, plots, gap analysis, amplicon metrics, per-stage QC — all landed in Phase 13).
	- Not done
		-  timing, fork tracking, fork age



# Phase 11 Implementation Specification — Forward architecture

**Last updated:** 2026-04-14
**Corresponds to:** ROADMAP Phase 11 (future `v0.11.xx`)
**Status:** Priority 11.1 complete (`v0.10.32`); Priorities 11.2 through 11.8 not started.
Some 11.3 carry-over items resolved: trimmed-mean smoothing (`v0.10.39`), `--hmm-bin-size` auto-detect (`v0.10.38`).

> **Maintenance rule:** This file is now the active Phase 11 plan. Update it whenever a Phase 11
> priority is completed or materially re-scoped.
>
> **Standing rule:** During Phase 11, keep `make summit`, `make summit-smoke`, and
> `make summit-baseline` as the canonical summit regression / baseline surface for HMM summit
> quality.

---

## What Phase 13 is

Phase 13 is the finishing up Per-Stage completeness to make it capable of producing its own analyses downstream of amplicon calling, including summit detection, summit refinement, APS, posterior ordering/grouping/manifest, timing, fork tracking, fork age, etc.

---

## Core design rules

1. Pipeline-derived outputs belong under the pipeline that computes them.
2. Shared computation belongs in modules, not in shared emitted-analysis directories.
3. Controller/run-level artifacts must remain analysis-neutral.
4. No pipeline is the default, canonical, authoritative, or main pipeline.
5. Until explicit synthesis exists, each pipeline remains independent in both `01-prior/` and
   `02-posterior/`.
6. Cross-pipeline synthesis remains a separate explicit layer above pipeline-local products.
7. Completeness work must respect pipeline-local ownership rather than flattening differences
   into centralized intermediate files.
8. Any Phase 11 change that alters emitted paths, output schemas, or inspection-relevant
   analysis surfaces must update the affected utilities, inspectors, and regression helpers in
   the same change rather than leaving compatibility drift for later.
9. As new pipeline-local features come online, extend the relevant inspection and regression
   surfaces (for example inspectors, evaluation harnesses, and helper tests) where those
   features are now expected to be visible or testable.
10. `multistage` remains an input-data property, not the name of the growth pipeline.
11. `--pipelines all` means all three pipelines: `growth`, `per-stage`, and `hmm`.

---

## Active carry-over decisions from late Phase 10-12

- The long-term pipeline term remains `growth-model`, while `multistage` remains the data/property term.
- The retained grouped pipeline order/names are `01-hmm/`, `02-growth-model/`, `03-rcn-mean-shift/`.
- The aligned engine/module naming direction is `hmm_engine.py`, `growth_model_engine.py`, `rcn_mean_shift_engine.py` (batch engine), and `rcn_mean_shift_singlefile_engine.py` (single-file CLI), all under `onionskin_core/engines/`. The shared RCN helpers live in `onionskin_core/rcn_mean_shift_helpers.py`.
- Until explicit cross-pipeline synthesis exists, each pipeline remains independent in prior/posterior space and should produce its own posterior manifest from its own prior outputs.
- The first synthesis target is call-level synthesis across pipeline-local prior outputs; downstream analyses can then be recomputed from the synthesized call set rather than treated as the first synthesis problem.
- The capability matrix for pipeline admissibility still needs an explicit written home in the live planning surface rather than remaining only implicit in controller behavior.

---

## Non-goals

- Do not revive deprecated shared emitted-output ownership patterns.
- Do not approximate synthesis through centralized intermediate files.
- Do not treat growth as the owner of common analysis surfaces.

---

## Priority 13.1 — Per-stage completeness

**Priority note:** Keep this active in Phase 11, but below HMM completeness unless a blocking
per-stage issue forces earlier attention.

### Goal

Extend the same pipeline-local completeness standard to the `per-stage` pipeline.

### Scope statement

Per-stage completeness must use the same ownership rules as HMM and growth.

### Required outcome

- The `per-stage` lane has a completeness matrix parallel to HMM.
- Biologically appropriate downstream families for `per-stage` are explicitly identified.
- Missing `per-stage` outputs remain pipeline-local.

### Implementation plan

1. Build the same completeness matrix for `per-stage`.
2. Identify which downstream analysis families are biologically appropriate for `per-stage`.
3. Reuse shared modules where applicable.
4. Implement missing per-stage outputs inside the per-stage subtree only.
5. Keep posterior behavior pipeline-local unless and until explicit synthesis exists.

---
