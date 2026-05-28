# Phase 9 Copilot Notes

Archived working notes and decisions from the active Phase 9 implementation period.
This file is preserved for historical context after Phase 9 closeout on 2026-04-11.

## Status

- Historical phase authority: `multi-agent/plans/archived/20260411-PHASE9_SPEC.md`.
- Alignment source: `ROADMAP.md` (should align, less detailed than spec).
- Legacy plan status: `multi-agent/plans/archived/20260409-PUFFSTEP-INTEGRATION-PLAN.md` is archived and not active.
- Final phase state: 9.1 complete, 9.2 complete, 9.3 complete, 9.4 complete; 9.5 and 9.6 deferred out of Phase 9.

## Closeout review — 2026-04-11

- Phase 9 is closed as the standalone-HMM phase.
- The previously listed step-10 summit-bin follow-ups were stale by closeout time: the full-bin summit semantics and focused regression coverage had already been delivered in the later v0.9.08 work recorded in project context.
- `--hmm-optimize` and other cross-pipeline-dependent follow-on work were intentionally deferred until after the Phase 10 shared-contract/unification phase.

## Active Decision Notes

### 2026-04-09: Phase 9.2 table-first MVP completed

- Implemented `onionskin_core/hmm_fork_travel.py` and wired step 12 in
  `onionskin_core/engines/hmm_engine.py`.
- Step 12 now emits `12-fork-travel/fork_travel_metrics.tsv`.
- Cross-stage tracking currently uses overlap-based `trajectory_id` grouping to
  handle stage-varying amplicon boundaries.

### 2026-04-09: Phase 9.2 completion pass finished

- Step 12 now emits full 9.2 outputs:
  - `12-fork-travel/fork_travel_metrics.tsv`
  - `12-fork-travel/plots/*.png`
- Plots implemented:
  - trajectory plots
  - asymmetry scatter
  - level-emergence heatmap
  - nested-domain diagrams
- 9.2 status is now complete; work shifts to 9.3.

### 2026-04-09: Phase 9.3 core step-13 outputs implemented

- Added `onionskin_core/hmm_summit_refinement.py`.
- Added step-13 wiring in `onionskin_core/engines/hmm_engine.py`.
- Step 13 currently emits:
  - `13-summit-refinement/all_trajectories.summit_refinement.tsv`
  - `13-summit-refinement/{trajectory_id}.summit_refinement.tsv`
- Implemented strategies and metrics:
  - permissive union, narrowest, coverage, combined summit intervals
  - origin stability (`origin_stddev_bp`)
  - multistage-aware filter flags (`filter_keep`, `multistage_growth_evidence`)

### 2026-04-09: Phase 9.3 completion pass finished

- Exposed step-13 policy controls via HMM CLI:
  - `--hmm-ms-min-width`
  - `--hmm-require-multistage-growth`
- Wired controls end-to-end through `onionskin.py` and `hmm_engine.py`.
- Added policy-focused unit test coverage in `tests/test_hmm_summit_refinement.py`.
- 9.3 status is now complete.

### 2026-04-09: Phase 9.4 core outputs wired (partial completion)

- Added `onionskin_core/hmm_ported_analyses.py`.
- Added HMM step runners:
  - `run_step14_hmm_aps(...)` -> `14-aps/`
  - `run_step15_hmm_timing(...)` -> `15-timing/`
  - `run_step16_hmm_clustering(...)` -> `16-clustering/`
- Wired steps 14-16 in `onionskin_core/engines/hmm_engine.py` and threaded
  step-14/15 controls from `onionskin.py`.
- Added `tests/test_hmm_ported_analyses.py`.
- Validation:
  - focused pytest PASS
  - `make test` PASS
  - HMM smoke run PASS (`dev/runs/phase94_smoke_batch/03-hmm/14-aps`,
    `15-timing`, `16-clustering` emitted)
  - `make puff-compare` PASS
- Remaining 9.4 scope:
  - additional summit-estimation strategies
  - required HMM notebooks

### 2026-04-09: Phase 9.4 completion pass finished

- Added side-by-side summit estimate outputs in step 15:
  - `summit_state_center_bp`
  - `parabola_summit_state_bp`
  - `parabola_active_window_bp`
  emitted in `15-timing/hmm_summit_estimates.tsv`.
- Added `onionskin_core/hmm_notebooks.py` and wired notebook generation in HMM engine.
- HMM now emits required notebooks in `03-hmm/notebooks/`:
  - `fork_travel_explorer.ipynb`
  - `hmm_state_path_qc.ipynb`
  - `amplicon_atlas.ipynb`
  - `parameter_characterization.ipynb`
- Expanded tests (`tests/test_hmm_ported_analyses.py`) for summit-estimate table
  and notebook generation.
- Validation:
  - focused pytest PASS
  - `make test` PASS
  - HMM smoke run PASS (`phase94_complete_smoke`)
  - `make puff-compare` PASS

### Step 10 summit bin semantics (temporary parity behavior)

- Current behavior: step 10 emits 1-bp point-style intervals in point-intersection cases.
- Reason: preserve `make puff-compare` parity baseline while Phase 9.2 work proceeds.
- Intended behavior: emit the full bedGraph bin interval (for example, a 5 kb bin)
  for the max-col4 summit bin within each summit-state region.
- Planned transition: switch step 10 to full-bin output, then update/rebaseline parity
  comparison expectations accordingly.

## Historical carry-forward items

- Priority 9.5 and 9.6 did not stay in Phase 9; they were deferred into the post-Phase-9 planning surface.
- The right-side asymmetry question and other HMM-specific open questions remain historical context for later design work, not unfinished active Phase 9 tasks.

## Maintenance

- When adding a new note, prefer short bullet points with date tags like `2026-04-09`.
- Move resolved notes into Phase 9 archive at Phase 9 close-out.

