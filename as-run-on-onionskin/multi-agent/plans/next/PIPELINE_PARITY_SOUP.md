# PIPELINE_PARITY_SOUP

**Theme:** cross-pipeline parity gaps that don't have a better thematic SOUP home.

**Project intent (re-stated):** "the only true difference between pipelines is how they call amplicons. Then almost everything thereafter is game for each pipeline at least by analogy — sometimes a pipeline might have to do something a little differently, but has the same thing in spirit." — Principal, chat 2026-04-29.

This file is the catch-all for cross-pipeline parity gaps where one pipeline has a feature/code path/data product the others don't (or analogous-but-divergent implementations exist where parity should hold), AND no themed SOUP is a better home.

## Filing convention

Before adding a new entry here, check whether an existing themed SOUP fits the gap naturally:
- `SUMMIT_SOUP.md` — summit-related concepts (estimators, refinement, canonical chosen-summit handoff).
- `FLAT_SOUP.md` — flat-sample / amplicon-flat-shape concepts.
- `APS_SOUP.md` — APS catalog + clustering concepts.
- `UNIFIED-RCN_SOUP.md` — unified RCN aggregation + cross-sample concepts.
- `TRAJECTORY_SOUP.md` — trajectory / cross-pipeline cluster concepts.

If the gap fits one of those naturally, file there with a `[PARITY]` tag in the entry heading or body. Otherwise, file here.

## Entries

### Item 1 (2026-05-05) — Triangle FIT for asymmetric fork elongation: Growth-only, RMS + HMM lack it

**Background.** Growth has triangle FIT code (`fit_asym_triangle()` at `onionskin_core/engines/growth_model_engine.py:649`; `tri_asym_basis()` helper at `:636`; `call_stagewise_model_for_call()` caller at `:680`) intended for asymmetric-fork-elongation analysis. The function takes `mu_bp` (the canonical upstream summit) as input + grid-searches over shape parameters `(wL, wR)`; returns best-fit shape (`b`, `h`, `wL`, `wR`, `sse`). This is a fit-with-fixed-apex characterizing fork-elongation asymmetry per stage; it is NOT a summit estimator (does not search over apex positions; the "summit" used is the upstream canonical mu).

RMS engine (`rcn_mean_shift_engine.py`, `rcn_mean_shift_helpers.py`) and HMM engine (`hmm_engine.py`) have ZERO triangle code (verified by grep 2026-05-05). Cross-pipeline parity gap: when the fork-elongation downstream work fires, RMS + HMM should have analogous triangle-fit code (consuming each pipeline's per-stage data + canonical summit) so all three pipelines produce comparable fork-asymmetry characterizations.

**Why it matters.** The fork-elongation application likely uses or will eventually use the project's best summit estimate as input. With Growth-only triangle fit code, fork-asymmetry analysis can only be performed on Growth-detected amplicons; RMS- and HMM-detected amplicons are excluded from this analysis. Per the project's cross-pipeline parity principle, this is a real systemic gap.

**What this item adds.**
- Port `fit_asym_triangle` + `tri_asym_basis` from `growth_model_engine.py` to a shared module (e.g., `onionskin_core/refinement.py` or a new `onionskin_core/triangle_fit.py`); same defaults across all 3 pipelines.
- Wire cross-pipeline integration so RMS + HMM emit per-stage triangle-fit results analogous to Growth's.
- Update the `--asym-tri-model-*` flag family + per-pipeline overrides (`--growth-asym-tri-model-*` already exist; add `--rms-asym-tri-model-*` + `--hmm-asym-tri-model-*` overrides at port time, with universal `--asym-tri-model-*` defaults shared).
- Per-pipeline integration tests verifying fork-asymmetry shape parameters land at expected output paths for each pipeline.

**Dependencies.**
- Canonical chosen-summit boundary established per pipeline (which cycle 15.10a-S2 is wiring via the 68-strategy menu's override path; F4 contract). Triangle FIT consumes the canonical summit per pipeline; needs that boundary stable before porting.
- Empirical motivation for which pipelines need fork-asymmetry analysis first (Principal-led decision, not surfaced by audit).

**Estimated scope.** Substantive port + cross-pipeline integration: ~1-2 days for an experienced developer including tests + cross-pipeline parity validation. Consume Growth's existing implementation as reference; not greenfield design.

**Status.** Deferred. No near-term Phase 15 supplemental cycle planned. Would land as part of a future fork-elongation-themed phase or supplemental cycle.

**Cross-references.**
- `fit_asym_triangle` at `onionskin_core/engines/growth_model_engine.py:649` (existing Growth implementation reference).
- `--asym-tri-model-halfwidth`, `--asym-tri-model-smooth`, `--asym-tri-model-halfwidth-grid` flags + per-pipeline `--growth-asym-tri-model-*` overrides at `onionskin.py:1999, 2009, 2019, 3310, 3318, 3325` (existing argparse surface; universal defaults are already in place; per-pipeline override pattern is already established for Growth).
- `multi-agent/plans/next/SUMMIT_SOUP.md` Item 5 expansion 2026-05-05 (triangle-apex SUMMIT ESTIMATOR — distinct concept; a NEW summit estimator using triangle-model-vs-RCN error minimization with grid-search over apex position; not implemented in any pipeline today; SEPARATE from this triangle-FIT-for-fork-elongation parity gap).
- Memory `feedback_cross_pipeline_parity_blindspot.md` (project intent — "the only true difference between pipelines is how they call amplicons; everything thereafter is game for each pipeline").
- Memory `feedback_inverse_parity_invention.md` 2026-05-05 (the corrigendum incident that surfaced this gap — orchestrator initially mis-attributed triangle code location to RMS; code dive corrected to Growth; the parity gap is RMS + HMM lack it, NOT Growth + HMM as initially stated).
