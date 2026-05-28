# TRAJECTORY — SOUP

**Stage:** SOUP (pre-BRAINSTORM scratchpad in `multi-agent/plans/next/`).
**Theme:** Cross-pipeline trajectory tracking — generalizing HMM's principled fork-tracking (state-path transitions) to Growth and RMS pipelines via heuristic RCN doubling-threshold scans.
**Lifecycle:** Promotes to a per-phase BRAINSTORM in `multi-agent/plans/`
when the orchestrator brings it onto the stage and assigns a phase number.
See `multi-agent/AGENT_CONVENTIONS.md § Future-phase planning surfaces`
and `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md` for the
SOUP → BRAINSTORM → SPEC lifecycle.

> **Maintenance:** SOUP files are intentionally early-stage and disorganized. Update freely as ideas surface; do not introduce SELF-references to phase numbers (cross-references to actually-closed phases ARE allowed as historical anchors — see `AGENT_CONVENTIONS.md § Future-phase planning surfaces`).
>
> **Provenance:** Created during Phase 15 cycle 15.6a-S1 R2 round (2026-05-03 chat) as Principal-direction follow-up to the trajectory clustering redesign in that cycle. The redesign locked HMM's per-fork preserved feature engineering (max_layers × 3 + 1 per stage; per-amplicon concatenation across stages; cluster amplicons on full feature matrix). This SOUP captures the parallel future-phase question: can Growth and RMS pipelines do an analogous fork-tracking analysis without HMM's state-path machinery?

---

## What this SOUP is about

HMM uses decoded state paths (state 1 = background; state 2 = first amplification; state 3 = second amplification; state N = (N-1)-th amplification) to track each fork's progression across stages. For each (amplicon, stage), HMM emits per-level fork edges (`level_<L>_left_bp`, `level_<L>_right_bp`) corresponding to the state-N → state-(N-1) transition points. The current Phase 15 cycle 15.6a-S1 redesign clusters amplicons on these per-fork-preserved features.

**Growth and RMS lack state-path data.** They have:
- Amplicon boundaries (the leading edge of the OUTERMOST = oldest fork — where the amplicon's RCN profile first crosses background).
- The RCN profile itself across the amplicon's span.
- Various derived metrics (summit, width above threshold, area excess, shape).

The leading edge of the oldest fork is trivially available from amplicon boundaries. **The interesting question:** can Growth/RMS approximate the inner forks (level_1, level_2, ...) by scanning the RCN profile for points where it crosses doubling thresholds (RCN ≥ 2, RCN ≥ 4, RCN ≥ 8, RCN ≥ 16, ...)? Each crossing point would be a heuristic estimate of where the corresponding fork generation reached.

If the heuristic produces useful approximations, Growth and RMS could each emit a `fork_travel_metrics.tsv` analog and feed it into the same trajectory-clustering machinery the HMM pipeline uses (cycle 15.6a-S1 deliverable). That would close another HMM-thinner gap.

If the heuristic is too noisy / too biased / too lossy, the conclusion is "trajectory tracking is HMM-only by data-availability constraint," and Growth/RMS keep the simpler "amplicon-boundary = oldest-fork-leading-edge" picture without inner-fork tracking.

This SOUP is the design space for that question. It is NOT an HMM-pipeline change — HMM keeps its principled state-path machinery. It is a heuristic generalization for the other two pipelines.

---

## Core design questions (open)

### 1. Heuristic fork-position scan — what's the algorithm?

The naive approach: walk along the RCN profile from the amplicon boundary inward, mark every position where RCN first crosses a doubling threshold (2, 4, 8, 16, 32, 64). Treat those crossing points as left/right fork edges for level_1, level_2, level_3, level_4, level_5, level_6.

**Naive-approach problems:**
- **RCN noise.** The raw RCN profile is bin-level signal with stochastic noise. A bin near threshold could fluctuate above/below repeatedly within a small interval; scanning for "first crossing" is sensitive to which side of a noise blip you sample.
- **Systematic offset from HMM transitions.** HMM finds the most likely point where states transition; that's not the same as where the RCN profile crosses the doubling threshold. The HMM transition is typically EARLIER (or LATER, depending on direction) than the RCN crossing because the HMM accounts for prior + likelihood + smoothness penalties, while the RCN crossing is just a hard threshold.
- **Multi-modal profiles.** Some amplicons have plateaus, secondary peaks, or asymmetric ramps. A simple monotonic scan from boundary to summit may misidentify the threshold crossing.

**Candidate fixes (each warrants design exploration):**
- (a) **Trend-line smoothing.** Fit a smooth trend line (LOWESS, kernel regression, monotonic regression) to the RCN profile within the amplicon, then scan the trend line for threshold crossings. Less noise-sensitive but introduces a smoothing-bandwidth choice.
- (b) **Triangle model crossings.** Fit the existing onionskin triangle shape model to the RCN profile within the amplicon's span (start to end). Use the fitted triangle's analytical form to compute exact threshold-crossing points. Robust to noise; assumes triangle shape; consistent with existing shape-filter machinery.
- (c) **Parabola model crossings.** Same as (b) but with the existing parabola shape model. Two analytical solutions per threshold (inner + outer crossing on each side); pick the relevant one per fork edge.
- (d) **Hybrid.** Combine: use trend-line for the broad scan, fall back to triangle/parabola model when the profile fits the model well.

The Phase 15 cycle 15.6a-S1 redesign already settled HMM's design ("use state-path transitions, no heuristics, no model-fitting"). Growth/RMS's design is open.

### 2. Doubling-threshold list — fixed or data-driven?

**Fixed thresholds:** 2, 4, 8, 16, 32, 64 (literal RCN doubling). Easy to specify; matches the binary "each fork doubles the local copy count" framing.

**Data-driven thresholds:** infer the set of "natural" RCN levels from the per-stage RCN profile distributions (mode-finding; KMM; HMM-on-RCN). More principled but introduces design complexity.

**Hybrid:** start with fixed thresholds for the design/implementation phase; allow `--trajectory-thresholds CSV` override if user wants per-dataset tuning.

### 3. Output schema parity with HMM

The HMM pipeline emits `fork_travel_metrics.tsv` with columns `(trajectory_id, stage, level_idx, state_value, left_bp, right_bp, left_travel_bp, right_travel_bp, asymmetry_bp, asymmetry_fraction, ...)`. For Growth/RMS to feed the same trajectory-clustering machinery, they'd need to emit the same schema.

**Open questions:**
- Growth/RMS don't have state values (no HMM = no decoded state path). Either fill `state_value = level_idx + 1` as a synthetic mapping, or omit the column. Schema-uniformity argues for synthetic fill; data-honesty argues for omit + a `data_source = heuristic` column to mark the difference.
- `level_idx = 0` convention: HMM uses level_0 = innermost = oldest = first-to-amplify (per-fork-age semantics). Growth/RMS heuristic should use the same convention so cross-pipeline clustering doesn't mix axis conventions.
- What about `incremental_left_bp` / `incremental_right_bp`? HMM computes these as `level[L].left_bp - level[L-1].left_bp`. Heuristic version: same arithmetic on heuristic crossings.
- Cross-stage annotation (`level_emergence_stage`, `ghost_level_flag`, monotonicity): same logic applies; trivially portable.

### 4. Cross-pipeline parity — should Growth/RMS feed the SAME trajectory clustering?

If the heuristic produces honest enough approximations, yes. Then trajectory clustering becomes cross-pipeline, with `--pipelines all` producing equivalent (or at least comparable) clustering across all three pipelines.

If the heuristic is too noisy, NO — Growth/RMS get their own (simpler) trajectory representation that's documented as "oldest-fork only" and doesn't feed cross-pipeline clustering. Trajectory clustering stays HMM-only by data-availability constraint.

**Validation gate:** before promoting this SOUP to a phase, run a comparison experiment — compute heuristic fork-positions on Growth/RMS for the same amplicons HMM tracked, compare to HMM's principled positions. If heuristic ↔ HMM agreement is ≥80% within ±5kb, heuristic is honest enough; <60% suggests it's too lossy.

---

## Relationship to other SOUPs

- **`SUMMIT_SOUP.md`** — summit-refinement algorithm parity is adjacent but distinct. Summit refinement is "where exactly is the peak"; this SOUP is "where exactly are the inner fork edges." Summit refinement has cross-pipeline parity work in Phase 15 cycle 15.7b (SPEC15.24 HMM per-stage parabola summit emission). This SOUP would be the inner-fork-edge analog.
- **`CROSS_PIPELINE_UNIFICATION_SOUP.md`** — Item 3 (HMM-thinner-APS-path surfaces audit + port: `build_amplicon_importance`, `build_scalar_orderings`, `write_aps_plots`). This SOUP is similar in spirit (closing an HMM-only emission gap) but at a different surface (fork-tracking vs APS analysis surfaces).
- **`UNIFIED-RCN_SOUP.md`** — RCN-related ideas. This SOUP is one specific RCN-derived heuristic (doubling-threshold scan).

---

## Provisional phase placement

**Not Phase 15.** Phase 15 is HMM-completeness focused; this is a Growth/RMS-completion follow-up. Earliest plausible home: a future cross-pipeline-parity-completion phase (potentially the same phase that consolidates other "HMM-thinner" surfaces from `CROSS_PIPELINE_UNIFICATION_SOUP.md`).

**Dependencies on Phase 15:**
- The HMM trajectory clustering redesign in cycle 15.6a-S1 sets the canonical feature-engineering convention (max_layers × 3 + 1 per stage; per-amplicon concatenation; 0-fill missing). Growth/RMS heuristic must produce the same feature schema once cross-pipeline trajectory clustering is wired.
- Cycle 15.7b's HMM per-stage parabola summit emission (SPEC15.24) is loosely related — Growth/RMS already have parabola/triangle fits via `summit_refinement` machinery. The same fits could be reused for fork-position estimation.
- Cycle 15.4a-S4's first-pass background mask (closed v0.14.89) defines what "background" means for the doubling-threshold scan — RCN above 1.0 or above some threshold derived from the mask?

---

## Out-of-scope for this SOUP (worth flagging)

- **Re-designing HMM's fork-tracking.** HMM's state-path-derived approach is principled and Phase 15 has locked the trajectory clustering on top of it.
- **Inventing new biological models for fork dynamics.** This SOUP is about closing a cross-pipeline gap with a heuristic, not about reinterpreting fork biology.
- **Fork-age semantics.** The `fork_age_idx = stage_level_count - 1 - level_idx` mapping is established in HMM's `hmm_fork_travel.py`; the heuristic version of Growth/RMS would inherit the same mapping. Not a SOUP-level question.

---

*SOUP files are append-only at the SOUP stage. Add new ideas as they surface; cross-reference adjacent SOUPs; defer organization to BRAINSTORM-stage formalization when the orchestrator promotes this SOUP to a phase.*
