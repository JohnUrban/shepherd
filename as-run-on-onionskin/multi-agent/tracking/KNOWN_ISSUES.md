# KNOWN ISSUES

Actionable near-term issues that the project intends to address soon, either directly,
by promoting them into the active phase spec, or by explicitly dropping them.

**What belongs here:** Concrete bugs, incomplete capabilities, architectural cleanup items,
inspection/regression gaps, and near-term side quests that are real enough to act on soon.

**What does NOT belong here:** Speculative ideas, long-range future directions, or naming
brainstorms that are not yet actionable. Those belong in `multi-agent/tracking/BRAINSTORM.md`.

**How this file is used:** Agents should consult this file when suggesting next steps,
especially when closing out a Priority or a Phase, because those are natural moments to pick
up short-horizon side quests.

**Lifecycle rules:**
- If a known issue is solved directly, remove it from this file and record the issue summary
	plus the solution/work in `CHANGELOG.md`.
- If a known issue is promoted into the active phase spec, remove it from this file and record
	the move in `CHANGELOG.md` so the new home is documented.
- If a known issue is solved indirectly by other work, remove it from this file and record that
	it was resolved indirectly in `CHANGELOG.md`, citing the work or prior changelog entry that
	obviated it.
- If a known issue is dropped, record the decision and rationale in
	`multi-agent/project_context/DECISIONS.md`, and also record the drop in `CHANGELOG.md`.
- Keep entries concise but actionable. The goal is to preserve enough structure that an agent
	can pick one up and start work without having to reconstruct the problem from memory.

**Suggested per-issue fields:**
- Status
- Urgency
- Why it matters
- Current understanding
- Likely landing zone
- Exit condition

**Ordering convention:** Entries are reverse chronological (newest first). New
top-level `[ISSUE:YYYY-MM-DD:N]` entries go at the top of the file (immediately
after this preamble), matching the convention used by `CHANGELOG.md`,
`multi-agent/DEVLOG.md`, `multi-agent/AUDIT_HISTORY.md`, the per-phase
`PHASE<N>_AUDIT_LOG.md` files, and `PHASE<N>_SURPRISE_LOG.md`. In-place expansions
of an existing entry stay where the entry sits — they are not "new entries" for
this rule's purpose. Out-of-order entries pre-dating this convention will be
re-sorted at cycle 15.10a SPEC15.20 closeout sweep; do not re-sort retroactively
outside that sweep. See `multi-agent/AGENT_CONVENTIONS.md § KNOWN_ISSUES.md` for
the canonical rule.


---

## [ISSUE:2026-05-05:4] `make test` runtime trending toward developer-experience threshold (~265 s post-cycle-15.10a-S2; redistribution into `make test-slow` gate recommended)

**Status:** Active. Surfaced 2026-05-05 by Phase 15 Final Overseer Pass 1 audit (Cat 4 / Recommendation 7). Replaces the older `[ISSUE:2026-04-21:2]` framing — both flag the same underlying concern but the cycle-15.10a-S2 closeout numbers are the load-bearing data point.

**Urgency:** Medium. Not blocking; degrading developer-experience cadence over time.

**Why it matters:** `make test` runtime per cycle 15.10a-S2 closeout (CHANGELOG `[v0.14.97]`): 265.94s (276 passed / 1 skipped) under xdist parallelization. Ramp from cycle-15.10a baseline of ~250s. Crossing the developer-experience threshold defined as "comfortable inner-loop iteration cadence" — at ~5 minutes per pass, agents iterating at the gate boundary spend material time waiting. Test-suite growth across Phase 14 (~127 tests) → Phase 15 (~276 tests) is roughly 2.2x in not-much-time; runtime will continue to grow as future phases add cross-pipeline integration tests unless the gate structure is redistributed.

**Likely root cause:** mix of unit + integration tests on a single `make test` gate. F6-style integration tests in cycle 15.10a-S2 (sampled 5 of 204 strategy-pipeline combinations) are appropriately scoped, but earlier cycles added full-pipeline integration tests on `make test` that could plausibly move to a slower gate.

**Recommended remediation (post-Phase-15):**

- **Redistribute into a `make test-slow` (or analog) gate.** Move slow integration tests (full-pipeline invocations, large-fixture E2E) to the slower gate; keep `make test` as the fast unit-test inner-loop gate.
- **Audit recently-added tests.** For each test added during Phases 13–15, classify as unit-vs-integration and route to the appropriate gate. The Cycle 15.10a-S2 R2 self-pushback notes mention sampling F6 integration tests precisely to avoid bloating `make test` further; that pattern can extend retroactively.
- **Verify pytest-xdist parallelization is enabled.** Cycle 15.10a-S2 R2 verified xdist is in use (`pytest -n auto -q` per make-target wiring); confirm and tune `n auto` if newer machines expose more cores.
- **Set realistic exit criteria.** Older `[ISSUE:2026-04-21:2]` had an "under 90 seconds" exit condition that was infeasible at filing time + is more infeasible now. Realistic post-redistribution target: `make test` (fast gate) under 60–90 seconds; `make test-slow` (separate gate) tolerates 5–10 minutes since it's not in inner-loop.

**Cross-references:**
- `[ISSUE:2026-04-21:2]` — older entry on the same theme; this entry supersedes it. R7 in Final Overseer Pass 1 recommended either close `[ISSUE:2026-04-21:2]` as superseded or amend exit condition; at orchestrator triage 2026-05-05 the choice was to keep both Active so the older entry's history isn't lost; future cleanup can close `[ISSUE:2026-04-21:2]` after this entry's redistribution lands.
- CHANGELOG `[v0.14.97]` cycle 15.10a-S2 closeout — 265.94s baseline.
- CHANGELOG `[v0.14.95]` cycle 15.10a closeout — ~250s baseline.

**Likely landing zone:** Post-Phase-15 dedicated test-suite-hygiene cycle, OR opportunistic redistribution at the start of Phase 16 if a Phase 16 close gate would otherwise inherit the runtime overhead.

**Exit condition:** `make test` (fast inner-loop gate) under 60–90 seconds; slow integration tests routed to a separate `make test-slow` (or analog) gate; classification rationale documented at gate-definition time.

---

## [ISSUE:2026-05-05:3] PuffStep parity gate (`make puff-compare`) is fragile against fresh regenerate: `#`-header skipping missing in comparison script + `--hmm-statepath-base` Makefile/reference snapshot mismatch

**Status:** Active. Filed 2026-05-05 by R3 of cycle 15.10a after a fresh `make puff-parity` (regenerate + compare) returned 20/28 PASS / 8 FAIL while the launcher-prescribed lighter-gate `make puff-compare` against the existing stale-but-consistent run dir returns 28/28 PASS (R2's reported result). The cycle 15.10a deliverables do not touch HMM detection, state-labeling, self-describing headers, or the gate itself — the 8 failures are pre-existing latent issues inherited from earlier cycles, exposed only by fresh regenerate.

**Urgency:** Medium. The gate continues to function correctly against the stale-snapshot pattern that R3 audits have used for several cycles (15.7a/15.7b/15.8a/15.9a/15.10a). The fragility surfaces only on regenerate. No biological/correctness regression in cycle 15.10a; the issue is gate-infrastructure-only.

**Two component defects:**

1. **`scripts/compare_puffstep_outputs.py:_read_sorted_lines` (lines 75–88) does not skip `#`-prefixed comment lines.** SPEC15.14 (cycle 15.7a) introduced self-describing file headers on HMM state-output bedGraph files (`# statepath_base = …`, `# background_state = …`, `# emission_params = …`, `# state_label_map = …`, `# pipeline_metadata_path = …`, etc., 7 lines on `06-HMM/*.states.bedGraph`). The reference at `dev/puffstep/automate/5kb-v2` predates SPEC15.14 and lacks these headers. After header strip, run and reference both have exactly 70681 data lines.
2. **`--hmm-statepath-base` Makefile flag value disagrees with the gold-standard reference at `dev/puffstep/automate/5kb-v2`.** Makefile target `puffstep-py` passes `--hmm-statepath-base 1` (the help text claims this is "PuffStep-compatible"). On `06-HMM/2.*.states.bedGraph`, the run emits state `1` for `~5 associated_contig_*` rows (e.g., `associated_contig_107:15000-20000`) where the reference emits state `0`. The reference therefore appears to use 0-based labels for those background-state rows, contradicting the help-text framing. This collapses into the `07-collapsedHMM/*.collapsed.bedGraph` value differences (`0.0` vs `1.0`) for the same auxiliary contigs.

**Why it matters:** The puff-compare gate is the load-bearing HMM detection bit-for-bit-unchanged regression gate. As currently configured it operates correctly only against a stale-but-consistent existing `dev/runs/puffstep-py/01-prior/01-hmm` directory. Any agent who runs `make puff-parity` (or `make puffstep-py` followed by `make puff-compare`) hits 8 failures that are NOT a real regression. This is a footgun for future R3 audits.

**Cross-references:**
- SPEC15.14 (cycle 15.7a, v0.14.91) — self-describing-file-headers feature that introduced the `#`-line skipping requirement.
- Cycle 15.10a R3 closeout (this entry's filing context). 15.10a R2 + earlier R3s used the lighter-gate `make puff-compare` against the stale snapshot.

**Additional note (2026-05-05, cycle 15.10a-S1 R2):** A later R2 run for the multi-pipeline posterior controller fix also hit the same 8-file `make puff-compare` failure pattern under the existing `make puff-compare` target, not only under fresh regenerate. The observed signals were the same two components already described here: `#`-prefixed metadata headers at the top of `06-HMM/*.states.bedGraph` (`# background_state`, `# emission_params`, `# pipeline_metadata_path`, etc.) and `07-collapsedHMM/*.collapsed.bedGraph` value mismatches on auxiliary contigs such as `associated_contig_107`, `associated_contig_132`, and `associated_contig_133`. No HMM-engine, state-labeling, or PuffStep-parity code was touched in that run; the edited slice was limited to multi-pipeline posterior controller routing, tests, and docs.

**Fix sketch (future cycle):**
- Patch `scripts/compare_puffstep_outputs.py:_read_sorted_lines` to skip `#`-prefixed comment lines (one-line filter inside the comprehension).
- Resolve the `--hmm-statepath-base` Makefile/help-text/reference snapshot mismatch: either regenerate the gold-standard reference under `--hmm-statepath-base 1`, or flip the Makefile to `--hmm-statepath-base 0`, or revisit the help-text framing.
- Add a regression check that flags missing `#`-header skip + statepath-base coherence at gate-run time.

**Likely landing zone:** Post-Phase-15 dedicated gate-hardening cycle (carried forward as a near-term post-archive priority — first dedicated cycle after Phase 15 archive that touches PuffStep parity infrastructure should absorb this). NOT a Phase 15 priority. Cross-confirmed by Phase 15 Final Overseer Pass 1 audit 2026-05-05 + cycle 15.10a R3 + cycle 15.10a-S1 R3 + cycle 15.10a-S2 orchestrator-as-R3 closeout (all four reproduced the same 8-file failure family + dispositioned identically; this is verified-not-a-Phase-15-regression).

**Exit condition:** Fresh `make puff-parity` (regenerate + compare) returns 28/28 PASS deterministically, with no reliance on stale-snapshot consistency.

---

## [ISSUE:2026-05-04:3] HMM `--norm-mode chrom-median` regression: spurious distal summit-state regions and degraded fork-tracking output vs the established `ref-stage` baseline

**Status:** Concern 1 LANDED (v0.14.95) — cycle 15.10a Stage A reverted the HMM `--norm-mode` default from `chrom-median` back to `ref-stage` per the SPEC15.16 partial reversal mid-phase amendment 2026-05-04. See DECISIONS.md `[2026-05-04] Phase 15 cycle 15.10a Stage A` entry. Concerns 2 (fork-tracking algorithm robustness) and 3 (HMM tuning for chrom-median to be production-quality) remain Active at unchanged post-Phase-15 landing zones; future re-flip conditions documented in the DECISIONS entry. Originally filed 2026-05-04 by Principal-orchestrator after by-eye comparison of cycle 15.7a SPEC15.16 default-flip output against established ref-stage baseline.

**Urgency:** Concern 1 RESOLVED via cycle 15.10a Stage A workaround. Concerns 2 + 3 stay Low (post-Phase-15 algorithm robustness + tuning work).

**Why it matters:** Cycle 15.7a SPEC15.16 (~5 days ago) flipped the HMM default from `ref-stage` to `chrom-median`. ref-stage has been the production HMM normalization mode for years (PuffStep parity gold-standard, primary training/validation analyses). Looking at downstream output post-flip exposed a substantive regression. The bug went undetected by automated testing because PuffStep parity has historically only exercised ref-stage mode; chrom-median was a new default without equivalent gold-standard validation coverage.

Quantitative impact (same input data, same run): chrom-median produces **506 amplicon trajectories vs ref-stage's 57 (8.9× more)** genome-wide. chrom-median stage_1 produces 138 summit-state regions where ref-stage detects 0; chrom-median stage_2 produces 439 where ref-stage produces 11 (40× more); the pattern is genome-wide, not locus-specific.

At the two best-characterized amplicons in the system (II/9A and II/2B, with by-eye HMM analysis defining expected `summit_state_interval` bounds at each stage in `tests/summit_training_data/01-II9A-...` and `02-II2B-...`):
- **ref-stage matches by-eye expectations EXACTLY** at every stage — 8/8 bp-perfect matches across 2 loci × 4 stages (stages 2-5).
- **chrom-median produces:** main amplicon bounds slightly tighter than expected at every stage; spurious distal summit-state regions at amplicon edges; spurious stage_1 detections where ref-stage and by-eye agree no detection should occur yet.

**Current understanding:**

This issue conflates three logically-distinct concerns that share a common observed symptom. Future fix work should treat them separately.

*Concern 1 — Upstream HMM regression in chrom-median mode (proximate cause).* chrom-median normalization (log2 of bin signal / chrom-median) does NOT remove per-bin technical bias. ref-stage normalization additionally subtracts the per-bin median across reference-stage samples on top of chrom-median (via `apply_ref_stage_baseline` in `growth_model_engine.py:376` — the equivalent step lives in the HMM normalization pipeline as well). The HMM's state-transition model in chrom-median mode appears to interpret per-bin technical bias as biological signal, producing spurious active-state regions away from real amplicon centers and over-detecting at stage_1 where signal is genuinely weak.

*Concern 2 — Fork-tracking algorithm robustness when fed noisy upstream data.* Internal inconsistency in `fork_age_metrics.tsv`: at II/9A in chrom-median mode, the tracker assigns `fork_age_emergence_stage=stage_1` to stage_4's outermost level (claiming continuity back to stage_1) but does NOT make the same claim for stages 2-3, which would be the intermediate steps in that trajectory. The same fork cannot have emerged at stage_1, "not exist" at stages 2-3, then exist again at stage_4 traced back to stage_1. May be a genuine bug in the matcher or an intentional design constraint that breaks down under noisy input — code inspection needed. ref-stage data shows the tracker handles legitimate level-count growth (2→3→4 levels across stages) cleanly when input is clean, so this concern may not manifest absent upstream noise.

*Concern 3 — Tune HMM for chrom-median to make it production-quality.* Even if Concern 2 is fully resolved, chrom-median's upstream output would still be noisier than ref-stage's. To make chrom-median a viable alternative mode (or eventually a viable default), the HMM itself needs tuning — transition probabilities, state priors, the cleanup step that filters spurious distal summit regions, or all of the above.

**Three-layer evidence chain (mechanism):**

1. **Upstream (`09-summitStates/*.bed`):** chrom-median calls spurious distal summit-state regions at amplicon edges. Per-stage genome-wide counts above. At II/2B, a spurious left summit at ~5840000-5885000 appears at every stage 2-5 (~330kb to the left of the real summit center at ~6.2M).

2. **Mid-pipeline (`12-fork-travel/fork_age_metrics.tsv`):** the spurious distal summits propagate as separate amplicon trajectories — each spurious region gets its own `trajectory_id`. This is the proximate cause of the 506-vs-57 trajectory count gap.

3. **Downstream (`12-fork-travel/plots/*.fork_age_trajectory.png`):** biologically-implausible patterns emerge. Symptom signatures include:
   - Oldest fork (Age 0) appears at stage_1, disappears at stages 2-3, reappears at stage_4 (II/9A and II/2B both show this pattern in chrom-median mode; ref-stage shows clean monotonic Age 0 progression at the same loci).
   - New "old" fork ages (Age 4, Age 5) appear only at the last stage with no precursor stages.
   - Asymmetric outlier forks present on one side without a symmetric counterpart on the other.
   - Disconnected single-stage points (no across-stage continuity).

**Reproduction:**
- Run dir: `dev/share/res20260504-1/`
- chrom-median run: `03/hmm_chrom_med/`
- ref-stage run: `02/hires_results/`
- Reference plots: `01-prior/01-hmm/12-fork-travel/plots/II_*.fork_age_trajectory.png`
- Gold-standard expectation files: `tests/summit_training_data/01-II9A-evidence-based-confinement-boundaries-and-rules.bed`, `tests/summit_training_data/02-II2B-evidence-based-confinement-boundaries-and-rules.bed`

**Likely landing zone:**
- **Phase 15 cycle 15.9b (proposed):** addresses Concern 1 via workaround — flips the HMM default back to ref-stage. Restores the historical production behavior validated against PuffStep parity and by-eye gold-standard expectations. Does NOT fix Concerns 2 and 3.
- **Future HMM-tuning cycle (post-Phase-15):** addresses Concern 3 — HMM parameter tuning + cleanup-step improvements so chrom-median is competitive with ref-stage on the same data. Likely sequenced after GECKO_SOUP-driven phase if GC-aware normalization reduces the technical bias that's confusing chrom-median.
- **Future fork-tracking robustness cycle (post-Phase-15):** addresses Concern 2 — read the fork-matching code; determine whether the `fork_age_emergence_stage` inconsistency is a bug or intentional; if bug, improve spatial-continuity heuristics or relax to consistency-over-completeness ("drop the stage_1 emergence claim if intermediate stages don't support it"). May be testable independently of Concern 1+3 by injecting synthetic noisy upstream input.

**Exit condition:**
- Concern 1: cycle 15.9b lands the default-flip reversal; ref-stage is restored as the HMM default; this entry's "workaround active" sub-state moves to "workaround landed."
- Concern 2: independent investigation cycle determines whether this is a bug or intentional design; if bug, algorithm fix lands and re-running on the captured chrom-median data produces consistent `fork_age_emergence_stage` columns across all stages (including stages 2-3 in the II/9A example).
- Concern 3: independent HMM-tuning cycle produces chrom-median output where genome-wide trajectory counts and per-stage summit-state counts are within ~2× of ref-stage's on the same input, and bp-level matches to by-eye gold-standard expectations land within ±5kb at II/9A and II/2B at every stage.
- Issue is fully retired only when all three concerns have been adjudicated (resolved or explicitly dropped via DECISIONS.md).

**Cross-references:**
- Cycle 15.7a SPEC15.16 (the chrom-median default flip that exposed this regression).
- Cycle 15.9b (proposed, post-15.9a; reverts the default).
- DECISIONS.md `[2026-05-04]` (the reversal rationale; to be written when 15.9b is formalized).
- `multi-agent/plans/next/GECKO_SOUP.md` (GC-aware normalization for future revisit of chrom-median competitiveness).
- `multi-agent/plans/next/CROSS_PIPELINE_UNIFICATION_SOUP.md` (potential home for the fork-tracking robustness work if it's framed as cross-pipeline parity).
- `tests/summit_training_data/01-II9A-evidence-based-confinement-boundaries-and-rules.bed`, `tests/summit_training_data/02-II2B-evidence-based-confinement-boundaries-and-rules.bed` (gold-standard expectation files used to validate ref-stage's bp-perfect match).
- PuffStep parity gate historical context: always run against ref-stage; chrom-median was never tested under this gate, which is why the regression slipped through automated testing.

## [ISSUE:2026-05-04:2] Growth pipeline emits peak-summary outputs via two independent code paths (redundant)

**Status:** Active. Known tech debt introduced intentionally in cycle 15.8a.

**What happened:** When `write_peak_summary_outputs()` from `summaries.py` was ported to RMS and HMM in cycle 15.8a, the growth pipeline was not migrated to the shared function — it had a long-standing inline implementation in `growth_model_engine.py` (`build_summary_profiles_per_chrom` + downstream write logic) that predates the cross-pipeline unification work. Rather than refactor mid-cycle, cycle 15.8a wired `write_peak_summary_outputs()` as an *additional* output pass for growth, writing to the new step-less `peak-summary/` directory. The old code continues to write to `07-signal-tracks/` as before.

**Impact:**

- The growth pipeline now runs the peak-summary computation twice per run (once via the legacy path → `07-signal-tracks/`, once via `summaries.py` → `peak-summary/`). For most runs the extra cost is small but it is still wasteful and confusing.
- The old `07-signal-tracks/` files remain: `_summary_peak.{label}.fold.bedGraph`, `_summary_peak.{label}.log2.bedGraph` (fold + log2, model-centric naming). The new `peak-summary/` files match the RMS/HMM surface: `_peak_summary_{mode}.bedGraph`, `_max_projection.bedGraph`, `_max_projection.chrom_median_norm.bedGraph`, `_max_projection.smoothed.bedGraph` (RCN space, cross-pipeline naming).
- The two code paths use the same underlying max/topk/quantile math but the smoothed variant in `peak-summary/` uses `smooth_halfwidth_bins=0` (no smoothing) because the growth context does not expose a per-bin smooth parameter to the onionskin.py call site.

**Intended resolution (future phase):**

1. Port the growth signal-tracks peak-summary output fully to `write_peak_summary_outputs()`, eliminating the inline `build_summary_profiles_per_chrom` peak path.
2. Expose a smooth_kb parameter to the growth call site so the smoothed `_max_projection.smoothed.bedGraph` is meaningful.
3. Decide whether `07-signal-tracks/_summary_peak.*` files should be removed (breaking change) or kept as legacy aliases.
4. Update test coverage to cover both the unified and legacy surfaces during the transition.

**Files involved:**

- `onionskin_core/engines/growth_model_engine.py` — legacy inline peak-summary path
- `onionskin.py` — new `write_peak_summary_outputs` call for growth (added cycle 15.8a)
- `onionskin_core/output_layout.py` — `build_growth_steps()` now has both `signal-tracks` and `peak-summary` keys
- `onionskin_core/summaries.py` — shared `write_peak_summary_outputs()`

**Likely landing zone:** Cross-pipeline unification phase (co-resolved with CROSS_PIPELINE_UNIFICATION_SOUP / future signal-surface parity work).

---

## [ISSUE:2026-05-03:4] Gap-rejection should be integrated with reliability + importance to enable rescue of incorrectly gap-rejected amplicons

**Status:** Active. Concrete + actionable. Surfaced during cycle 15.6a-S1 R1 deliberation (2026-05-03 chat) when reviewing empirical filter rates in `amplicon_reliability.tsv` outputs and observing that the X:9555000-9900000 amplicon was filtered solely because `gap_suspect=True` despite a `reliability_score` of 0.845 (well above the 0.50 default class threshold for `Unclassified`).

**Why it matters:**

- The `gap_suspect` axis in `compute_reliability_sidecar` is currently a **hard-exclude axis**: `gap_suspect=True` → `reliability_flag=filter` regardless of the underlying `reliability_score` (`onionskin_core/reliability.py:184-185`). The only override is `keep_override=True` from a user-provided `--keep-amplicons-bed` BED — which requires the user to know in advance which gap-rejected amplicons to rescue.
- Empirical observation (user, 2026-05-03): the X:9555000-9900000 amplicon "is real but is also interrupted by a gap in the way that we would want to flag it. An edge case for sure." This is a class of amplicons where the gap-analysis correctly flags a structural concern, but the underlying signal (per the reliability and importance scorers) suggests the amplicon is real and informative.
- A principled framework would let the reliability + importance scores inform the gap-rejection decision rather than the gap-rejection being a unilateral hard-filter. For example: an amplicon with high reliability + high importance + gap-suspect should be flagged for review, possibly resurrected with caveats, rather than silently dropped.
- This is adjacent to `[ISSUE:2026-05-03:1]` (reliability + importance scoring redesign + relocation) and may be best resolved in the same phase.

**Proposed framework directions (for design exploration in a future cycle, not lockedyet):**

- **Soft gap-rejection:** instead of binary `gap_suspect` → filter, treat `gap_suspect` as a sub-score (0 if gap-suspect, 1 otherwise) that participates in the reliability mean alongside the 5 monotone-trend axes. Loci with high reliability + high importance can survive a gap-suspect strike if their other axes compensate.
- **Resurrection rule:** keep `gap_suspect` as a hard-exclude default but add an automatic resurrection path when reliability_score AND importance_score both exceed configurable thresholds. Surface the resurrected loci as a separate column / output for user review.
- **Tiered flag:** replace binary `reliability_flag {pass, filter}` with a tiered classification `{pass, conditional, filter}` where `conditional` is "gap-suspect but otherwise high-quality" — preserved in the canonical aggregation but flagged for inspection.

**Action items (post-Phase-15, likely co-resolved with [ISSUE:2026-05-03:1]):**

1. Audit current gap-rejection cases in real runs to understand the rescue-vs-filter tradeoff empirically. How many gap-suspect loci have high reliability + high importance? What fraction of those would be rescuable on inspection?
2. Decide on framework direction (soft / resurrection / tiered).
3. Implement chosen framework as part of the reliability + importance scoring redesign in `[ISSUE:2026-05-03:1]`.
4. Update `--keep-amplicons-bed` semantics if needed — if soft gap-rejection lands, the explicit user-provided keep BED becomes a sharper override layer rather than the only rescue mechanism.

**Cross-references:**

- `[ISSUE:2026-05-03:1]` (reliability + importance scoring redesign + relocation — likely co-resolved).
- `[ISSUE:2026-04-29:5]` (gap-analysis architectural cleanup — closed in cycle 15.4b SPEC15.22; landed the gap_suspect annotation that this issue would soften).
- Cycle 15.6a-S1 R1 audit F5 + empirical inspection of `dev/runs/15-4a-S3-fixture-check/chrom-mad/01-prior/01-hmm/16-aps/amplicon_reliability.tsv` (X:9555000-9900000 case).
- `onionskin_core/reliability.py:184-185` (current hard-exclude implementation).

**Likely landing zone:** Same phase as `[ISSUE:2026-05-03:1]` (post-Phase-15 unification phase or scoring redesign phase — TBD).


---

## [ISSUE:2026-05-03:3] Test-generalization audit: review all test scripts to identify which need to be extended cross-pipeline

**Status:** Active. Process improvement, not a single bug. Surfaced during cycle 15.6a-S1 R1 deliberation (2026-05-03 chat) — user observed that twin peaks test runs Growth-only and asked whether other tests have the same single-pipeline scope drift.

**Why it matters:**

Many test scripts exercise only one pipeline (e.g., `tests/run_twin_peak_test.sh` runs `--pipelines growth` only). This systematically:

- Hides per-pipeline regressions ("test passes" → false sense of cross-pipeline correctness).
- Hides "tests pass but feature doesn't work in pipeline X" situations (e.g., `[SURPRISE:15:15.6a:1:A:7]` — HMM doesn't emit `aps_amplicon_importance.tsv` but no test caught it because no test checked HMM for that emission).
- Doesn't enforce cross-pipeline parity at the test layer, even when project intent is cross-pipeline equivalence per the user direction "the only true difference between pipelines is how they call amplicons."

This is the test-layer analog of `[ISSUE:2026-04-29:7]` — that issue tracks the SPEC + audit-layer gap; this one tracks the test-layer gap.

**Action items (post-Phase-15):**

1. Audit `tests/` + `Makefile` targets — which test scripts run `--pipelines <one>` vs `--pipelines <all>`? Produce a matrix: test × pipelines-exercised.
2. For each pipeline-specific test, decide: is the underlying behavior pipeline-agnostic (should be tested cross-pipeline) or pipeline-specific (correctly scoped)?
3. Generalize where the underlying capability is a project goal (twin peaks resolution; gap-handling; reliability scoring; APS aggregation; clustering; summit refinement; etc.).
4. Lock per-pipeline test scope explicitly in test docstrings / Makefile help text so the scope is intentional, not accidental.
5. Add cross-pipeline parity tests where missing (e.g., a single test that runs the same data through Growth + RMS + HMM and verifies analogous outputs).

**Cross-references:**

- `[ISSUE:2026-05-03:2]` (twin peaks is one concrete instance of this broader pattern).
- `[ISSUE:2026-04-29:7]` (systemic cross-pipeline-parity audit gap at SPEC + audit layer — this issue is the test-layer analog).
- Phase 15 cycle 15.8a SPEC15.17 (cross-pipeline analysis-surface parity — emissions; this issue is about tests, not emissions).

**Likely landing zone:** Post-Phase-15 cleanup phase. NOT a Phase 15 cycle.

---

## [ISSUE:2026-05-03:2] Twin peaks test should run all three pipelines; Growth uses non-default growth-z-thresh in test; defaults likely fail twin peaks in the wild

**Status:** Active. Concrete + actionable. Surfaced during cycle 15.6a-S1 R1 deliberation (2026-05-03 chat) — user observed that Growth pipeline does not consistently pass twin peaks resolution under default parameters in the wild on the same datasets used for development.

**Why it matters:**

- `tests/run_twin_peak_test.sh:44` runs `--pipelines growth` ONLY. RMS and HMM are not exercised by `make twin` or `make full-twin`. Twin peaks resolution is a project performance goal that should hold cross-pipeline.
- The test script invokes Growth with `--growth-z-thresh 5.0` (dataset1) and `5.5` (dataset2) — both pumped higher than the production default `4.0`. The script comment explicitly notes: "*5.0 eliminates low-confidence FPs in the toy extract while preserving all 4 known twin-peak detections.*"
- This means **the test passes by design with tuned z-thresholds, not at production defaults**. When the user runs Growth in the wild with default `--growth-z-thresh 4.0`, twin peaks resolution fails — but the test reports PASS, masking the real-world failure mode.
- Note on `--norm-mode`: the test uses `--norm-mode chrom-median`, which IS the production default for Growth and RMS (HMM defaults to `ref-stage`, which is being worked toward unifying to `chrom-median` as a Phase 15 deliverable). So the `--norm-mode` choice in the test is correct; the parameter mismatch is in `--growth-z-thresh` only.
- Suspected mechanism: `--growth-z-thresh 5.0–5.5` is high enough to suppress true positives that share the noise floor with twin peaks. Lowering to default `4.0` recovers true positives but introduces noise that breaks twin peak resolution. So Growth at production defaults may have a fundamental sensitivity-vs-resolution tradeoff that the test obscures.
- An analogous example is RMS: it used to have more stringent z-thresholds, but those could be relaxed after the triangle filtering and multistage unification step landed. For Growth, the analogous structural improvement (triangle filter / multistage unification feeding back into z-thresh tuning) may not yet be in place. Parameter tuning alone may not suffice; a structural improvement may be needed.

**Action items (exploratory, post-Phase-15):**

1. Audit twin peaks test parameters vs current production defaults. Reconcile: should the test reflect production defaults, or should production defaults be pushed up to match what works in the test? (Note: pushing defaults up sacrifices sensitivity for resolution; this is a real tradeoff that needs deliberation.)
2. Extend test harness to run twin peaks against Growth, RMS, and HMM. Each pipeline reports pass/fail independently. Twin peaks resolution is a project performance goal that should hold cross-pipeline.
3. Parameter sweeps on Growth at production-default `--growth-z-thresh` to identify what changes recover twin peaks reliably. Surface findings; potentially update Growth defaults.
4. Investigate whether structural improvements (triangle filter / multistage unification analog for Growth, by analogy with RMS's z-thresh relaxation history) would resolve the sensitivity-vs-resolution tension without requiring a tuned z-thresh.
5. Diagnostic-first; defaults update is a separate downstream cycle.

**Cross-references:**

- `[ISSUE:2026-05-03:3]` (test-generalization audit — twin peaks is one concrete instance of the broader cross-pipeline test coverage gap).
- Phase 15 cycle 15.5a (SPEC15.7 + SPEC15.8 — cross-pipeline summit refinement + summit↔timing convergence — closed v0.14.83; relevant to twin peaks resolution mechanics).
- Phase 15 cycle 15.7a SPEC15.13 (HMM-specific late-phase work; if Growth needs analog structural improvements, an analogous "growth-completeness" arc may be warranted in a future phase).

**Likely landing zone:** Post-Phase-15 cleanup phase. NOT a Phase 15 cycle. Sweeps + diagnosis only — no own cycle needed.

---

## [ISSUE:2026-05-03:1] Reliability + importance scoring need principled redesign + relocation; verify post-cycle-15.8a

**Status:** Active. Concrete + actionable. Surfaced during cycle 15.6a-S1 R1 deliberation (2026-05-03 chat) when the user reviewed the live `_monotone_score` formula and observed that the implementation is shallow relative to the documented criteria for amplicon reliability. Empirical observation: reliability scores cluster in [0.8, 1.0] for most amplicons; binary `reliability_flag` filtering is dominated by `gap_suspect` (the hard-exclude axis), not by score thresholds.

**Why it matters:**

- **`amplicon_reliability.tsv`** — `reliability_score` is computed as a simple arithmetic mean of 5 monotone-trend axes (`rcn_increase_score`, `width_increase_score`, `area_increase_score`, `bic_increase_score`, `parabola_height_increase_score`). Each axis is itself a heuristic blend `0.7 × nonnegative + 0.3 × positive` where `nonnegative` is the fraction of stage-to-stage deltas that are non-decreasing and `positive` is `1.0` if any delta strictly increases else `0.75` — with the `0.75` softener producing an effective `0.225` floor (rather than `0.0`) for a fully-failing trajectory. Implementation: `onionskin_core/reliability.py:_monotone_score:53-62`. The formula is ad-hoc — produced by an agent at some point, not based on a principled model of "what makes an amplicon real." The user has spent significant time describing reliability criteria; the current implementation is shallow.
- **`aps_amplicon_importance.tsv`** — `importance_score = 0.33 × norm(-lag) + 0.33 × norm(max_RCN) + 0.165 × norm(n_active_stages) + 0.165 × norm(activity_breadth)`. Long-standing (pre-Phase-15; `aps.py:973-1080` `build_amplicon_importance`). Empirically has bigger spread than reliability and correlates better with discriminating power for sample ordering and clustering — a barely-active amplicon has near-zero discriminating power even if it's "reliable."
- The two scorers solve adjacent problems and may be best designed jointly / integrated rather than continuing to evolve in parallel. Reliability answers "do we believe this is a real amplicon?"; importance answers "how informative is this amplicon for ordering and discrimination?". In cycle 15.6a-S1, a `joint = reliability × importance` weighting mode was added to APS aggregation as a partial remedy, but the underlying scorers themselves remain shallow.
- Both scorers currently live downstream of the multistage-unification step (in APS directory under `onionskin_core/aps.py` and `onionskin_core/reliability.py`). Conceptually they belong AT or BEFORE multistage unification — same level as gap-analysis (cycle 15.4b SPEC15.22 relocated gap-analysis to its own step before APS for similar reasons). APS would be a consumer of reliability + importance, not the producer.
- The `0.225` floor in `_monotone_score` (the `positive=0.75` softener when no delta is strictly positive) could be moved to `0.0` once a principled scoring design is in place. Don't change it unilaterally now — current floor is acceptable as "uncertainty in the scorer" rather than as a real lower bound on amplicon quality.

**Action items for future cycle (likely post-Phase-15 unification phase):**

1. Audit reliability scorer against documented criteria; replace `_monotone_score` with a principled formulation. Inspect each of the 5 sub-scores to determine which are working as intended and which need redesign.
2. Audit importance scorer; consider whether the 4-component weighting (33/33/16.5/16.5) is the right structure; revisit weights against ground truth.
3. Decide whether to integrate the two scorers or keep them separate but co-located (e.g., a unified `amplicon_quality.tsv` with both columns plus a recommended combined metric).
4. Relocate emission to multistage-unification step (or sister step like gap-analysis); APS becomes a consumer.
5. Once redesigned, revisit the `_monotone_score` `0.225` floor (move to `0.0` if scoring math supports it).
6. Reconcile with `[ISSUE:2026-05-03:4]` (gap-rejection integration with reliability + importance to allow rescue of incorrectly gap-rejected amplicons).
7. **AFTER cycle 15.8a closes** (where HMM gets the importance port + step-less `plots/` + APS_CATALOG.md), verify:
   - HMM emits `aps_amplicon_importance.tsv` at parity with Growth/RMS.
   - The chrom/start/end schema change introduced in cycle 15.6a-S1 propagated to all three pipelines.
   - The `joint` score bedGraph/BED tracks emit cleanly cross-pipeline.
   - The track emissions aren't duplicated/triplicated unnecessarily across pipeline output dirs.

**Cross-references:**

- Cycle 15.6a-S1 R1 audit F5 + F6 (immediate cycle that surfaced this issue + landed the partial remedy via `--aps-aggregation-mode`/`--aps-aggregation-score`/`--aps-aggregation-score-min` flags).
- `[SURPRISE:15:15.6a:1:A:7]` (HMM thinner APS path).
- `[ISSUE:2026-04-29:6]` (sample-level APS reliability filtering — narrow fix landed in cycle 15.6a-S1; principled redesign deferred here).
- `[ISSUE:2026-04-30:3]` (flat-sample + amplicon reliability tuning — closed at v0.14.87; this issue is the broader principled-redesign continuation).
- `[ISSUE:2026-05-03:4]` (gap-rejection integration with reliability + importance to enable rescue of incorrectly gap-rejected amplicons — adjacent issue, likely co-resolved).
- DECISIONS.md `[2026-XX-XX]` Phase 15 cycle 15.6a-S1 — `--aps-aggregation-*` semantics + score-weighted aggregation framing (drops "Bayesian shrinkage" terminology from prior framing).

---

## [ISSUE:2026-05-01:1] Stage-1 active-stage anchor detection still uses per-bin within-stage MAD instead of a background-region noise estimate

**Status:** Resolved (v0.14.91) — cycle 15.7a Stage G G.5 landed the narrow stage-1 active-stage anchor sigma-source repair per the NARROWED F10 framing (stage-1 only; stages N>1 untouched). New CLI flag `--odw-stage1-sigma-source {chrom-background-mad, within-stage-mad}` at `onionskin.py:1761-1773` with default `chrom-background-mad`. Dispatch at `onionskin_core/timing.py:_median_mad_sigma`: parameter `stage1_sigma_source: str = "chrom-background-mad"` at `:215`; stage-1 dispatch at `:453` `if int(stage) == 1 and str(stage1_sigma_source).strip().lower() == "chrom-background-mad":` reads the per-chrom chrom-background-MAD from cycle-15.4a-S4 surfaces. Stages N>1 continue using per-bin within-stage MAD via the existing helper logic per the NARROWED F10 framing — correct match for stage-to-stage differential-expression questions. Threading at `onionskin.py:793, 808, 4090, 4216, 4588` (controller + HMM engine + RMS controller + Growth call sites). New regression test at `tests/test_stage1_anchor_sigma_source.py` (4 tests, 4 pass) covers default + opt-in paths + threading. `make puff-compare` 28/28 PASS held (HMM detection bit-for-bit unchanged — change is to the timing module, not to HMM detection). Closed in this file at cycle 15.7a R3 closeout.

**Original status:** Active. Surfaced 2026-05-03 during cycle 15.4a-S4 Stage A verification.

**Urgency:** Medium. Not a release blocker for cycle 15.4a-S4 because the current cycle is already scoped to first-pass background-mask machinery + chrom-MAD emission, but it is the next near-term consumer that wants the same background-aware sigma surface.

**Why it matters:** The stage-1 active-stage anchor decision is structurally a within-stage-vs-background question, not a stage-vs-stage differential-expression question. Using per-bin within-stage replicate MAD at that boundary imports the same kind of biological-variance-as-noise failure mode that cycles 15.4a-S2 and 15.4a-S3 just corrected for the flat-sample test. Stages N>1 are different: there the question really is stage-to-stage change, so within-stage MAD remains the right variance surface.

**Current understanding:** `onionskin_core/timing.py:_median_mad_sigma` loads `stageN.MAD_RCN.bedGraph` / `stageN.MAD_log2RCN.bedGraph` from the per-pipeline APS directories and returns a window-level sigma from those per-bin between-sample MAD tracks (`onionskin_core/timing.py:363-384`). `compute_timing_from_stage_summary(...)` then consumes that helper for **all** stages, including the stage-1 anchor comparison against background (`onionskin_core/timing.py:488-504`). There is no stage-1 special case. The cycle 15.4a-S4 Stage A code read confirms the narrow case from R1's F4 is real on the live tree.

**Likely landing zone:** Follow-up cycle or cycle 15.7a SPEC15.15 cross-pipeline ODW timing rework. The natural repair is to thread the new chrom-background-MAD surfaces / background-region sidecars into the stage-1 anchor path without disturbing the stages N>1 transition tests.

**Exit condition:** Stage-1 anchor detection reads a background-region-aware sigma source (for example the new per-sample chrom-background-MAD surfaces or an equivalent background-region sidecar) while stages N>1 continue to read per-bin within-stage MAD, and DS1/DS2 calibration confirms acceptable `onset_stage` / `last_active_stage` drift.

**Cross-references:**
- `onionskin_core/timing.py:363-384` — `_median_mad_sigma`
- `onionskin_core/timing.py:488-504` — stage-loop call site that includes stage 1
- `multi-agent/plans/PHASE15_AUDIT_LOG.md` cycle 15.4a-S4, Finding F4 + Role 2 Stage A diagnostic

---

## [ISSUE:2026-04-30:6] Shape-filter window is "amplicon bounds + ctx_bins" — could be more principled with summit-centered or amplicon-bounds-with-25-50kb-flanks instead

**Status:** Active. Surfaced 2026-04-30 by orchestrator-outside-cycle code dive during cycle 15.4a-S2 R2 flight, while answering user pushback on R2's `step14_shape_score_threshold=0` default flip.

**Urgency:** Low. The current Growth/RMS shape-filter window (call.start to call.end + ctx_bins=100 each side, ≈ 1.5Mb total for typical 5kb-bin amplicons of width ~500kb) works in practice and most current calling results would not change with a different window choice. This is a "more principled" cleanup item, not a defect-fix.

**Why it matters:** The current shape-filter window is conceptually awkward: it scales linearly with amplicon width AND adds a fixed bin-count context. For a wide amplicon (1Mb call range), the scored segment becomes 2Mb (1Mb call + 500kb each side); for a narrow amplicon (50kb call), 1.05Mb (50kb call + 500kb each side). The triangle vs flat BIC test is then comparing very different geometric regimes across calls. Two more principled alternatives the user has previously considered:

(a) **Summit-centered fixed window** — e.g., `summit_bp ± 500kb` regardless of amplicon width. Captures the majority of typical amplicons (most are <1Mb wide) with a uniform geometric scoring regime. Was apparently the user's design intent at one point; current implementation is the "amplicon bounds + 100-bin context" form which approximates this for typical-width amplicons but diverges at the extremes.

(b) **Amplicon-bounds + small fixed flank** — e.g., `call.start - 25kb` to `call.end + 25kb`. Provides modest context without the wide-amplicon-explosion problem. The 25-50kb flank is the same range the user has flagged as the "absolute widest we should be checking" for summit-area metrics generally (per the discussion of mean_rcn vs summit_rcn dilution).

**Current understanding:** No principled justification for the current "ctx_bins=100" choice over either alternative. Likely a default carried forward from early shape-filter prototyping. Most current results won't change because typical amplicons cluster in the 100kb-1Mb-wide regime where the three options give similar windows. Edge cases (very narrow or very wide amplicons) might benefit from either alternative.

**Likely landing zone:** Future-phase consideration. Not in any current Phase 15 cycle's scope. Could land alongside any future shape-filter rework cycle. If the cycle 15.4a-S3 shape-score harmonization (per `[SURPRISE:15:15.4a-S2:1:A:3]`) needs to make a window-choice decision anyway, weighing alternative (a) or (b) at that time would be cheap.

**Exit condition:** Decision recorded in DECISIONS.md (or future SPEC) about which window strategy to use; if changed from current, regression validation on representative datasets shows no significant call-set drift.

**Cross-references:**
- `multi-agent/plans/PHASE15_SURPRISE_LOG.md [SURPRISE:15:15.4a-S2:1:A:3]` — shape-score harmonization SURPRISE that surfaced this question
- `onionskin_core/refinement.py:473-533` — `shape_filter_calls` with current `ctx_bins=100` default
- User's framing 2026-04-30 — *"if it is aimed at using summit +/- 500kb to capture any amplicon, I wonder why it does not just use the amplicon boundaries ...? or amplicon boundaries with 25-50 kb flanks..."*

---

## [ISSUE:2026-04-30:5] `locus_contributions.tsv` `flat_sample_flag` column is stale-False even when the sample-level rollup says True

**Status:** Resolved (v0.14.87) — cycle 15.4a-S2 R3 re-audit verified the in-cycle fold-in. `attach_flat_sample_flags_to_contributions(contrib_df, flat_df)` is now called in both `onionskin_core/aps.py:compute_and_write_aps` (Growth + RMS APS) and `onionskin_core/hmm_ported_analyses.py:run_step16_hmm_aps` (HMM APS) after reliability attachment, mapping per-(sample_id, prior_stage) `flat_sample_flag` from `flat_samples.tsv` onto every contribution row. Verified at re-audit by inspecting `dev/runs/toy_out/01-prior/01-hmm/16-aps/locus_contributions.tsv`: stage-1 samples flagged True in `flat_samples.tsv` now show `flat_sample_flag=True` per row; stage-2-5 samples flagged False show False. Cross-pipeline applicable (single helper in `onionskin_core/flat_sample.py:508-541`). Surfaced 2026-04-30 by orchestrator-outside-cycle data-deep-dive during cycle 15.4a-S2 R2 flight per `[SURPRISE:15:15.4a-S2:1:A:2]`.

**Urgency:** Low-medium. Affects downstream consumers that read `locus_contributions.tsv` directly rather than joining against `flat_samples.tsv`. The reliability scorer at `onionskin_core/reliability.py:180` correctly reads the flat sidecar so `amplicon_reliability.tsv`'s flat_sample_flag is accurate; the bug is specifically in `locus_contributions.tsv` writes.

**Why it matters:** Two output files (`locus_contributions.tsv` + `flat_samples.tsv`) report the same fact (per-sample-stage flat verdict) but disagree on at least one user-supplied dataset (`dev/share/res20260430-1/` — every SAG40 stage-1 row in `locus_contributions.tsv` shows `flat_sample_flag=False` while `flat_samples.tsv` correctly reports `flat_sample_flag=True` with `n_flat=7/7`). Anyone consuming `locus_contributions.tsv` for downstream analysis gets the wrong flat-classification per row. Cross-pipeline applicability — same writer code path serves all three pipelines, so likely affects Growth + RMS too.

**Current understanding:** The column is in the schema (locus_contributions header includes `flat_sample_flag keep_override`) but the value being written is not coming from the post-flat-sidecar consumer chain. Likely causes (in order of likelihood): (a) the column is populated from a stale snapshot before the flat sidecar is finalized; (b) the column is populated from a different per-locus quantity that happens to share the same column name; (c) the column is never populated and defaults to False. Investigation should grep `compute_and_write_aps` in `onionskin_core/aps.py` for the population logic.

**Likely landing zone:**
- **First preference:** flag to cycle 15.4a-S2 R3 (Template C re-audit by Claude Code — Opus, Effort: High) for in-cycle small fix during re-audit. R3 has a natural blast radius into the flat-detection surfaces this cycle is touching; one-line population fix in `compute_and_write_aps` likely sufficient.
- **Fallback if R3 doesn't catch / fix:** cycle 15.10a SPEC15.20 closeout sweep absorbs.

**Exit condition:** `locus_contributions.tsv` rows for samples flagged flat in `flat_samples.tsv` show `flat_sample_flag=True`. Consistency check across pipelines — Growth + RMS + HMM all emit consistent flat_sample_flag values across both files.

**Cross-references:**
- `multi-agent/plans/PHASE15_SURPRISE_LOG.md [SURPRISE:15:15.4a-S2:1:A:2]` — the source observation
- `onionskin_core/aps.py:compute_and_write_aps` — the writer
- `onionskin_core/flat_sample.py:write_flat_sample_sidecar` — the authoritative source
- `onionskin_core/reliability.py:180` — correct consumer pattern (mirror this in the locus_contributions writer)

---

## [ISSUE:2026-04-30:4] HMM-only `--posterior` + `--compute-aps` silently no-op + flat-sample per-pipeline state divergence

**Status:** Resolved (v0.14.87) — cycle 15.4a-S2 closed all four findings. Stage B.4 added standalone `_run_hmm_posterior(...)` at `onionskin.py:4455-4520` (parallel to `_run_rcn_posterior` and `_run_posterior`); HMM-only short-circuit branches at `onionskin.py:4818` (multistage) and `:5090` (single/two-stage) now invoke it under `if getattr(args, "posterior", False)`. HMM APS now emits `aps_clusters_raw.tsv`, `aps_clusters_zscore.tsv`, and order files (`aps_dendrogram_order_raw.tsv`, `aps_dendrogram_order_zscore.tsv`, plus the order-comparison files via `write_order_tables`) needed by `build_posterior_manifest`'s `_POSTERIOR_REQUIRED` contract; the legacy `aps_clusters.tsv` is preserved. Stage B.5 refactored the shared-flag pattern into per-pipeline `args._flat_sample_auto_switch_<pipeline>` attributes plus a unified `_apply_flat_sample_auto_switches(args, pipeline_names)` consumer applied separately at each posterior helper (`_run_posterior` covers all three; `_run_rcn_posterior` covers RMS+HMM addressing the new F11 RMS-posterior gap; `_run_hmm_posterior` covers HMM). The compatibility shim `args._flat_sample_auto_switch` is still set for any older readers but is no longer consulted by new code paths. Stage B.6 added `hmm_aps_dir` layout key at `onionskin_core/output_layout.py:293`. New regression tests at `tests/test_hmm_only_posterior.py` (E2E `--pipelines hmm --posterior --compute-aps` against toy fixture) and `tests/test_per_pipeline_flat_sample_state.py` (per-pipeline auto-switch isolation + pipeline-aware norm-mode resolver). `make puff-compare` PASS 28/28 on freshly re-run post-R2 tree (HMM detection bit-for-bit unchanged). Surfaced 2026-04-30 by orchestrator-outside-cycle audit prompted by direct user observation `--pipelines hmm --posterior` produces no posterior outputs and no error.

**Urgency:** High — user-flagged 2026-04-30 ("I need this done ASAP"). Cycle 15.4a-S2 broadens its original `[ISSUE:2026-04-30:3]` flat-detector + reliability-scorer-tuning scope to absorb this HMM-posterior-unit fix in the same cycle.

**Why it matters:** `--pipelines hmm --posterior` invocations silently produce no posterior outputs and no error. The HMM-only short-circuits in `onionskin.py` return before `args.posterior` and `args.compute_aps` are checked. This is the recurring "X-pipeline-only" defect pattern (per `feedback_cross_pipeline_parity_blindspot.md`) — Growth and RMS (when run alone) both honor `--posterior`; only HMM-alone fails. Surfaced 2026-04-30 by orchestrator-outside-cycle audit prompted by direct user observation.

**Current understanding:** Four findings, all in scope for cycle 15.4a-S2 (severity S2 = silent feature no-op or silent miscalculation risk):

1. **Finding 1 — `--posterior` silently no-ops for HMM-only.** `onionskin.py:4672-4691` (multistage controller) and `onionskin.py:4941-4959` (single/two-stage controller) HMM-only short-circuit branches `return` before `args.posterior` is checked. The two `_run_posterior(...)` call sites at `onionskin.py:4712` and `onionskin.py:4770` live inside the multistage+growth branch gated by `if dec.mode == "multistage" and _pipelines_include(args, "growth")`. The multistage RMS branch at `onionskin.py:4773-4806` correctly honors `--posterior` via `_run_rcn_posterior`; HMM has no equivalent.

2. **Finding 2 — `--compute-aps` not invoked for HMM-only prior runs.** Same HMM-only short-circuits return before `_phase2_postprocess` (where Growth's APS computation lives at `onionskin.py:795-830`) and before the RMS engine's APS step (at `onionskin.py:3647-3670`). HMM-only runs produce no APS outputs — which then makes Finding 1's posterior fix unimplementable, since `_run_posterior` requires APS cluster files (gates at `onionskin.py:4296-4302` + `onionskin.py:4385-4390`). Findings 1 and 2 are an inseparable unit.

3. **Finding 3 — Flat-sample auto-switch is per-pipeline-set but consumed as a single shared flag in `_run_posterior`.** `onionskin.py:4402-4411` reads `args._flat_sample_auto_switch` and applies the same norm-mode override across `growth_norm_mode` / `rms_norm_mode` / `hmm_norm_mode`. The `_check_flat_sample_auto_switch(...)` helper at `onionskin.py:97-147` IS called separately for each pipeline (sites at `onionskin.py:852` growth, `:3697` rms, `:3962` hmm) and each call sets `args._flat_sample_auto_switch = True` independently — meaning whichever pipeline runs LAST wins. If pipelines' flat-sample status diverges (one pipeline detects 0 flats but another detects ≥1), the posterior norm-mode override is wrong for at least one pipeline. Per-pipeline state must be preserved per-pipeline (e.g., `args._flat_sample_auto_switch_growth`, `_rms`, `_hmm`) and consumed per-pipeline.

4. **Finding 6 — APS not emitted by HMM at all.** Growth and RMS have explicit APS code paths; HMM has none. Coupled with Findings 1+2 — fixing posterior wiring requires APS emission first. The `--compute-aps` help text at `onionskin.py:1716-1718` documents this flag as universal across all three pipelines, but the implementation is Growth+RMS-only.

**Likely landing zone:** Cycle 15.4a-S2 (mid-phase amendment 2026-04-30; broadened scope per user direction).

**Exit conditions:**
- `--pipelines hmm --posterior --compute-aps` produces a `posterior/` directory with HMM-pipeline outputs.
- New end-to-end regression test (e.g., `tests/test_hmm_only_posterior.py`) verifies E2E HMM-only-posterior pathway against toy fixture; PASSES at cycle close.
- Flat-sample auto-switch decision is applied per-pipeline (each pipeline's posterior norm-mode override consults its OWN prior-run flat-sample state, not a shared flag).
- `make puff-compare` 28/28 CRITICAL gate held (HMM detection bit-for-bit unchanged by posterior wiring).

**Out of scope (deferred to other landing zones):**
- Finding 4 (Growth-only `--posterior` re-run shortcut at `onionskin.py:4708-4713`) — `multi-agent/plans/next/CROSS_PIPELINE_UNIFICATION_SOUP.md` Item 6a.
- Finding 5 (Growth-anchored output layout dict keys at `onionskin_core/output_layout.py:243-294`) — same SOUP Item 6b.
- Finding 7 (HMM-missing diagnostic plots — APS, summit, profile, shape-filter, QC) — Phase 15 cycle 15.8a, SPEC15.17 scope expansion (4 new deliverables).

**Status flips at closeout:** Active → Resolved (v0.X.YY) when cycle 15.4a-S2 closes.

---

## [ISSUE:2026-04-30:2] HMM pipeline step-number leakage into function names, dict keys, parameters, and variables

**Status:** Active. Surfaced at Phase 15 cycle 15.6a R3 closeout discussion (2026-04-30) when reviewing `[SURPRISE:15:15.6a:1:A:8]` (cross-pipeline structural divergence meta-observation).

**Urgency:** Medium-high — **opportunistic in-phase fixes mandatory for any cycle that touches the named surfaces.** The bulk refactor (HMM engine's 242 step-numbered call sites + `build_hmm_steps` semantic-key flip) is too large for any single Phase 15 cycle and is deferred to a future phase per `multi-agent/plans/next/CROSS_PIPELINE_UNIFICATION_SOUP.md`. But cycles that touch step-numbered functions/params/variables MUST fix-as-they-go rather than perpetuate the pattern.

**Why it matters:** Phase 15 alone has renumbered HMM steps twice (cycle 15.2a SAPS placeholder insertion, cycle 15.4b gap-analysis-before-APS reorder). Each renumber forced renames across function names, parameter names, local variables, log message prefixes, dict keys, output filename prefixes, AUDIT_LOG cross-references, SPEC line annotations, and tests. The cycle 15.4b Final-Overseer caught 5 missed sites; cycle 15.6a R1's `[SURPRISE:15:15.6a:1:A:3]` caught additional SPEC line-number staleness. Every renumber to come will cost the same audit attention. The function/param/variable names being step-numbered is the load-bearing cause of much of that maintenance — they encode WHERE a function sits in the pipeline sequence rather than WHAT the function does.

**Current understanding:** Three distinct categories of step-number leakage exist in the codebase, each with different scope + leverage:

**Category 1 — `build_hmm_steps()` step-numbered DICT KEYS (HIGHEST LEVERAGE).** `onionskin_core/output_layout.py:301-336` returns `{"step1": "01-mednorm", "step2": "02-stage-medians", ..., "step21": "21-timing-updated"}`. Every engine call site does `steps["step15"]`, `steps["step16"]`, etc. — encoding pipeline step numbers into engine logic. Compare to Growth (`growth_steps["origins"]`, `growth_steps["stage-medians"]`) and RMS (`rcn_ms_steps["chrom-norm"]`, `rcn_ms_steps["summit-refinement"]`) which use semantic keys. HMM is the asymmetry. ~242 engine call sites in `onionskin_core/engines/hmm_engine.py` alone consume these keys. Flipping to semantic keys is the single highest-leverage change because it forces all call sites to update + creates natural pressure to rename surrounding functions/params.

**Category 2 — Step-numbered FUNCTION NAMES.** Concrete sites:
- `onionskin_core/hmm_summit_refinement.py`: `_discover_step11_stage_files`, `_load_step11_rows`, `_load_step12_growth`, `run_step13_summit_refinement`.
- `onionskin_core/hmm_ported_analyses.py`: `run_step15_hmm_gap_analysis`, `run_step16_hmm_aps`, `run_step17_hmm_saps`, `run_step18_hmm_timing`, `run_step19_hmm_clustering`.

**Category 3 — Step-numbered PARAMETER NAMES + LOCAL VARIABLES.** Parameters like `step4_dir`, `step11_dir`, `step12_dir`, `step14_dir` leak step numbers into call sites. Local variables like `step11_rows` propagate the leakage further. Log message prefixes like `[hmm] step15: ...` are user-facing diagnostic strings — those can keep step numbers (step numbers help users navigate output dirs) OR move to a hybrid like `[hmm] gap-analysis (step 15): ...`.

**What stays step-numbered (intentional, do NOT rename):**
- **Output directory names** (`15-gap-analysis/`, `16-aps/`) — user-facing layout where ordering matters; step numbers are the layout's primary signal.
- **`_summits_step1.tsv`, `_timing_step2.tsv`** in `summit_convergence.py` — these refer to the **4-step convergence algorithm** (step1=initial, step2=initial timing, step3=refined, step4=final), NOT pipeline step positions. Different concept entirely.

**Per-cycle in-phase fix guidance (Phase 15 remaining cycles):**

- **Cycle 15.6b** (determinism diagnostic): unlikely to touch step-numbered surfaces. Fix in-flight if naturally adjacent.
- **Cycle 15.6a-S1** (Stage D APS analytics): touches APS analytical code; unlikely to touch engine step-naming. No in-flight rename mandatory.
- **Cycle 15.7a** (HMM-specific late-phase work, including SAPS implementation): **MANDATORY in-flight rename** — `run_step17_hmm_saps` becomes `run_hmm_saps` as part of the SAPS net-new implementation. Don't perpetuate the pattern in net-new code.
- **Cycle 15.8a** (cross-pipeline analysis-surface parity): likely touches HMM APS surface. Opportunistic in-flight rename of any step-numbered surface the cycle modifies.
- **Cycle 15.9a** (architectural cleanup, SPEC15.21): **picks up the bulk of HMM-pipeline-specific function/param/variable renames as a sub-deliverable of SPEC15.21.** Scope: function rename for the 9 step-numbered functions named in Category 2 + their parameter renames in Category 3. **Does NOT include** the `build_hmm_steps` semantic-key refactor + 242 engine call sites — that's too large for cycle 15.9a's "mechanical-with-care, ZERO net change at default invocation" scope frame.

**Future-phase scope (deferred, NOT cycle-15.9a work):**

- `build_hmm_steps()` semantic-key refactor (Category 1) + 242 engine call site updates.
- Any remaining Category 2/3 sites not picked up by in-phase fixes.
- Cross-references in `multi-agent/full_instructions/PIPELINE_SPEC.md` + tests + planning files that propagate step numbers from these surfaces.

Substantive home for the future-phase work: `multi-agent/plans/next/CROSS_PIPELINE_UNIFICATION_SOUP.md`. The step-number leakage is one specific instance of the broader cross-pipeline structural divergence pattern that SOUP captures.

**Convention guard added 2026-04-30:** `multi-agent/AGENT_CONVENTIONS.md` § Naming conventions gains a new rule (cycle-15.6a-fallout dev-system change committed alongside this issue): *"Function names, parameter names, local variables, and dict keys MUST NOT embed pipeline step numbers. Step numbers belong on output directory names + user-facing diagnostic strings + intentional algorithm-step artifacts (e.g., the 4-step convergence's _summits_step1.tsv). They do NOT belong in code identifiers."* Future code reviews + audits must enforce.

**Cycle 15.7a Stage G postscript (2026-05-04; appended at v0.14.91 closeout):** Cycle 15.7a applied the in-flight opportunistic rename to FIVE Category 2 functions per Stage A (SAPS) + Stage G G.6 (gap-analysis + APS + timing + clustering): `run_step15_hmm_gap_analysis` → `run_hmm_gap_analysis` at `onionskin_core/hmm_ported_analyses.py:161`; `run_step16_hmm_aps` → `run_hmm_aps` at `:224`; `run_step17_hmm_saps` → `run_hmm_saps` at `:556` (with thin compatibility shim at `:551-553` for stale-import safety); `run_step18_hmm_timing` → `run_hmm_timing` at `:830`; `run_step19_hmm_clustering` → `run_hmm_clustering` at `:952`. Engine call sites updated at `onionskin_core/engines/hmm_engine.py:1842-1846, 1853, 1865, 1915, 1922, 1965, 1991, 2011`. Tests updated at `tests/test_hmm_ported_analyses.py` + `tests/test_per_sample_chrom_mad_emission.py`. Step-numbered output directory keys (`steps["step15"]`, etc.) intentionally retained per § "What stays step-numbered." Remaining Category 2 surfaces (`hmm_summit_refinement.py` 4 functions: `_discover_step11_stage_files`, `_load_step11_rows`, `_load_step12_growth`, `run_step13_summit_refinement`) + Category 1 (`build_hmm_steps` semantic-key flip + 242 engine call sites) + Category 3 (parameter + local-variable renames) all UNCHANGED — landing zone unchanged at cycle 15.9a SPEC15.21 + future-phase per CROSS_PIPELINE_UNIFICATION_SOUP. Issue stays Active.

**Cycle 15.7b postscript (2026-05-04; appended at v0.14.92 closeout): Category 2 COMPLETE.** Cycle 15.7b applied the in-flight opportunistic rename to the remaining FOUR Category 2 functions in `onionskin_core/hmm_summit_refinement.py`: `_discover_step11_stage_files` → `_discover_amplicon_metrics_stage_files`; `_load_step11_rows` → `_load_amplicon_metrics_rows`; `_load_step12_growth` → `_load_fork_travel_growth`; `run_step13_summit_refinement` → `run_hmm_summit_refinement`. Public call site updated in `onionskin_core/engines/hmm_engine.py`. Combined with cycle 15.7a's 5 renames, all 9/9 Category 2 step-numbered function names are now retired from the HMM codebase. Category 2 is **COMPLETE**. Categories 1 + 3 remain open at their unchanged landing zone (cycle 15.9a SPEC15.21 for bulk Category 3; future-phase via CROSS_PIPELINE_UNIFICATION_SOUP for Category 1). Issue stays Active (Categories 1+3 outstanding).

**Cycle 15.9a Stage A.5 postscript (2026-05-04; appended at v0.14.94 closeout): Category 3 NARROW SCOPE RESOLVED.** Cycle 15.9a Stage A.5 (per OQ1 INCLUDE lock) applied the opportunistic Category 3 rename pass to the in-scope surface: parameter names + local variables within the 9 already-renamed Category 2 functions in `onionskin_core/hmm_summit_refinement.py` and `onionskin_core/hmm_ported_analyses.py`. Both files are now grep-clean of `step\d+_dir` / `step\d+_rows` / `step\d+_growth` identifiers (verified at R3 closeout via `grep -nE 'step[0-9]+_dir|step[0-9]+_rows' onionskin_core/hmm_summit_refinement.py onionskin_core/hmm_ported_analyses.py` → zero hits). Category 3 in the **renamed Cat 2 function surface** is RESOLVED. Category 3 in `onionskin_core/engines/hmm_engine.py` (~30+ remaining `step\d+_dir` parameters/locals) remains UNRESOLVED — that surface is structurally Category 1 territory (parameters consume `steps["step1"]`, etc., dict keys), and was deferred per OQ1's narrow scope framing. Category 1 + remaining engine-internal Category 3 remain at their unchanged landing zone (`multi-agent/plans/next/CROSS_PIPELINE_UNIFICATION_SOUP.md`). Issue stays Active (Category 1 + engine-internal Category 3 outstanding).

**Principal-bless postscript (2026-05-04, post-closeout):** OQ1 as discussed pre-cycle did NOT have a "narrow scope framing" — that phrasing in the postscript above was R3's closeout-time interpretation. R3 narrowed Cat 3 at closeout from the originally agreed "renames in functions + their callers" to "renames in function bodies only" (skipping the ~30+ engine-internal callers in `hmm_engine.py`). R3's architectural defense — that engine-internal `step\d+_dir` parameters consume `steps["stepN"]` dict keys and so are coupled to Cat 1's dict-key flip, making a half-rename (`amplicon_metrics_dir = steps["step11"]`) strictly worse than the current state — is sound. Principal accepted the narrowing post-closeout 2026-05-04 rather than re-opening for a separate engine-rename pass. Engine-internal Cat 3 will land alongside Cat 1's dict-key refactor in the future cross-pipeline unification phase, not as a separate Phase 15 sub-deliverable.

**Likely landing zone:**
- **In-phase opportunistic** (Phase 15 remaining cycles): cycles 15.7a (mandatory for SAPS) + 15.8a (opportunistic) + 15.9a (bulk of Category 2 + 3 in-phase scope).
- **Future phase**: `multi-agent/plans/next/CROSS_PIPELINE_UNIFICATION_SOUP.md` graduation owns Category 1 (`build_hmm_steps` semantic keys + 242 engine call sites).

**Exit condition:** Either (1) the bulk refactor lands in a future-phase structural-unification cycle and the codebase is grep-clean of step-numbered code identifiers (excepting the intentional surfaces enumerated above); OR (2) per-cycle in-phase fixes accumulate to grep-clean over time, in which case the future-phase scope shrinks proportionally and the issue closes when grep-clean is achieved.

**Cross-references:**
- `[SURPRISE:15:15.6a:1:A:8]` in `multi-agent/plans/PHASE15_SURPRISE_LOG.md` — meta-observation that this issue is a microcosm of.
- `multi-agent/plans/next/CROSS_PIPELINE_UNIFICATION_SOUP.md` — future-phase substantive home for the bulk refactor.
- `multi-agent/plans/next/SUMMIT_SOUP.md` § Item 5 expansion — adjacent cross-pipeline parity work (summit-algorithm parity); the two SOUPs cross-reference each other.
- `multi-agent/AGENT_CONVENTIONS.md` § Naming conventions — convention guard preventing future leakage.
- Cycle 15.4b SPEC15.22 (`PHASE15_AUDIT_LOG.md`) — the renumber that surfaced the maintenance cost most acutely.
- Cycle 15.4b Final-Overseer findings FO-F1 through FO-F5 — concrete examples of renumber-sweep miss patterns.

---

## [ISSUE:2026-04-30:1] Strategy-selection determinism — `selected_stages` varies between runs of identical code on identical toy data

**Status:** Resolved (v0.14.86) — cycle 15.6b CLOSED 2026-04-30 via Path 1 (source identified + made deterministic). Stage A.1 (HMM bare-`except Exception` adjacency probe) returned 0/5 swallowed exceptions across 5 toy runs and 0/5 RMS worker-failure warnings. Stage A.2 (broader 5×-toy targeted TSV diff) returned 58/58 identical files across 5 runs (`dev/runs/15-6b-stageA2/targeted_tsv_digest_summary.tsv`). Stage A.3 (canonical-grid invariant audit) returned 42/42 same-grid checks true. Stage B prophylactic hardening landed across `common.py` (new `assert_shared_track_grid` + `assert_shared_chrom_track_grid` helpers), `aps.py`, `aps_pca.py`, `engines/growth_model_engine.py` (5 sites + sort-order lock at `:1968`), `engines/rcn_mean_shift_engine.py` (2 sites + worker-failure re-raise + active-stage-convergence wrapper removal), `rcn_mean_shift_helpers.py` (2 sites), `engines/hmm_engine.py:1665-1734` (active-stage refinement broad-catch narrowed to `_safe_copy` OSError-only), and `summit_convergence.py:295` (sort-order lock for per-stage `_stage<N>_summit_estimates.tsv` enumeration). Stage C added `tests/test_determinism.py` (3 parametrized cases — Growth/RMS/HMM each — guarding reintroduction). Cross-pipeline parity validated. Validation gates at closeout: `pytest tests/test_determinism.py -q` PASS (3/3); `make test` PASS (176/177 + 1 skipped); explicit 2-run toy diff PASS (58 files, 0 diffs); `make twin` PASS; `make full-twin` PASS (4/0/2); `make puff-compare` PASS (28/28 — CRITICAL HMM detection bit-for-bit unchanged). The original observation (cycle 15.5a R3 vs R2-J `II:3680000-4345000` `selected_stages=4` vs `4,5`) was not directly reproduced in the post-cycle-15.6a 5+ identical-input runs at the diagnostic stage; the determinism critical-path silent-failure + implicit-order hazards that were prophylactically hardened ARE real on the code path and any of them could plausibly have been the cause. Empirical evidence of post-fix stability is strong via the regression-test gate. See `multi-agent/plans/PHASE15_AUDIT_LOG.md` cycle 15.6b R1 + R2 + R3 sections for full diagnostic + repair contract + closeout judgment.

**Urgency:** Medium-high — **must be picked up by cycle 15.6a R1 audit.** Cycle 15.6a (SPEC15.9-15.12 — APS analytical work + clustering defaults) consumes the same upstream timing/ODW data that this non-determinism sits on top of. If APS analytical computations turn out to depend on the affected surfaces, cycle 15.6a's audit needs to either scope a fix into the cycle or explicitly defer with rationale per the scope-authority rule.

**Why it matters:** Two runs of `make toy` on the same tree at the same HEAD produced different `selected_stages` for the same call. The per-stage parabola summit values were stable across runs, but the **strategy machinery** (specifically the `odw_best_shape` strategy in `onionskin_core/summit_strategies.py`) selected different stages each time. Concrete observation:

| Run | call | selected_stages | step3_summit_bp |
|-----|------|----------------|-----------------|
| Cycle 15.5a R2-J 2026-04-30 | `II:3680000-4345000` | `4` (1 stage) | 4095535 |
| Cycle 15.5a R3 re-run 2026-04-30 | `II:3680000-4345000` | `4,5` (2 stages) | 4096689 |

The cycle 15.5a R3 prescribed contract — that step3 ≠ step1 — held in both runs, so cycle 15.5a closed cleanly at v0.14.83. But the underlying observation is real: something upstream of strategy selection is non-deterministic on identical input.

**Current understanding:** The variance is upstream of the strategy selector itself (per-stage parabola summits were identical across both runs). Likely candidates:
1. **Per-stage shape-confidence values** (the `odw_best_shape` strategy reads these to decide which ODW-active stages survive). If shape-confidence computation has a stochastic component, the strategy selector reading it is doing the right thing — it's the *input* that's noisy.
2. **Upstream timing / ODW boundary computation** (`onset_stage`, `last_active_stage`, `odw_start_stage`, `odw_end_stage` from `onionskin_core/timing.py`). If these vary between runs, the ODW window the strategy selector receives varies → different stages selected.
3. **Parallel reduction order in `_per_stage_one`'s `ProcessPoolExecutor`** (`onionskin_core/engines/rcn_mean_shift_engine.py`). RMS runs per-stage detection concurrently; floating-point reduction order across concurrent stages may produce slightly different aggregate values.
4. **Cached state leak** despite `rm -rf dev/runs/toy_out` (lower probability since the user-invoked R3 re-audit also cleaned the toy dir).

**Diagnostic recipe (cycle 15.6a R1 audit should run this):**
1. Run `make toy` 5 times in a row from a clean tree (`rm -rf dev/runs/toy_out` before each).
2. Diff the timing.tsv (Growth + RMS + HMM) and ODW boundary columns between runs. If those differ → the noise is upstream of strategy selection (timing or ODW). If they're identical but `selected_stages` still differs → noise is in the strategy selector itself.
3. Diff per-stage shape-confidence values between runs (if exposed).
4. If `ProcessPoolExecutor` reduction order is suspected: re-run with `concurrent.futures.ThreadPoolExecutor` or serial execution to compare.

**Likely landing zone:** Cycle 15.6a R1 audit picks this up as a scoping decision: either (a) scope a deterministic-fix into cycle 15.6a if APS computations are sensitive to the variance; (b) defer to a later cycle if APS is robust to the variance; (c) escalate to a dedicated cycle (15.6b or similar mid-phase amendment) if the fix is large and APS robustness can't be assumed. Cycle 15.6a R1 must explicitly address one of (a)/(b)/(c) per the scope-authority rule (no silent deferral).

**Exit condition:** Either (1) the source of non-determinism is identified and made deterministic in cycle 15.6a (or a downstream cycle); or (2) cycle 15.6a R1 explicitly verifies APS analytical work is robust to the strategy-selection variance and documents the acceptable tolerance bounds in DECISIONS.md or PIPELINE_SPEC.md; or (3) the variance is identified as expected statistical behavior of an inherently stochastic upstream component (e.g., bootstrap), in which case the contract becomes "step3 ≠ step1 holds, magnitude is statistical" and a confidence-interval column may be appropriate.

**Cross-references:**
- `multi-agent/plans/PHASE15_AUDIT_LOG.md` cycle 15.5a R3 closeout subsection — original observation.
- `multi-agent/plans/next/SUMMIT_SOUP.md` Open brainstorm question 11 (cross-pipeline `step3_n_stages_used` evidence-count column parity) — adjacent design question that becomes more interesting if the variance turns out to be real biological signal vs noise.

---

## [ISSUE:2026-04-21:2] `make test` runtime is slow for inner-loop development (~3:20, dominated by integration tests)

**Status:** Active. Noticed during Phase 14 where `make test` is run every few minutes.

**Urgency:** Medium. Not a correctness issue, but meaningfully slows the audit/implement loop.

**Why it matters:** With the current multi-agent development rhythm (audit → implement → re-audit, each requiring a `make test` pass), 3+ minutes per cycle adds up quickly. The total of 74 tests is not large; the issue is that most of `test_pipeline.py` runs the full pipeline on real bedGraph data.

**Current understanding (profiled 2026-04-21):**

| Group | Time | % of total | Notes |
|---|---|---|---|
| 4 posterior tests | ~94s | 47% | Run full prior + posterior pipeline each; hard to make cheap |
| 7 RMS standalone norm-mode / alias tests | ~31s | 15% | Each runs full 4-stage RMS just to verify a norm-mode value or flag alias |
| `test_toy_multistage` + divergent-norm-modes test | ~19s | 9% | Core multistage integration — legitimately needed |
| `test_multistage_growth_shape_score_strict_bic_reaches_live_sink` | ~11s | 5% | Phase 14 addition; runs full pipeline to check a column exists |

The 7 RMS standalone tests are the most actionable target: they test things (norm-mode resolution, flag alias acceptance) that are now also covered by fast resolver/parser unit tests added in Phase 14. The full-pipeline run is redundant coverage for those specific assertions.

**Likely landing zone:**

1. **RMS standalone tests (31s savings possible):** Replace the norm-mode verification and alias acceptance assertions in the 7 `test_multistage_rcn_mean_shift_*` tests with resolver/parser-level unit tests. Keep one representative full-pipeline RMS test for integration smoke. Affected tests:
   - `test_multistage_rcn_mean_shift_default_uses_shared_chrom_median_semantics`
   - `test_multistage_rcn_mean_shift_default_can_follow_shared_ref_stage_semantics`
   - `test_multistage_rcn_mean_shift_can_force_ref_stage_override`
   - `test_legacy_per_stage_norm_mode_flag_is_accepted_as_alias`
   - `test_legacy_per_stage_pipeline_token_is_accepted_as_alias`
   - `test_multistage_rcn_mean_shift_can_emit_early_parabola_selector_candidate`
   - `test_multistage_rcn_mean_shift_only_runs_standalone_controller`

2. **Posterior tests (94s):** Consolidate assertions across the 4 tests into fewer runs. `test_force_manifest_remaps_primary_and_hires_and_supports_posterior`, `test_posterior_output_files_use_user_out_prefix`, `test_posterior_rcn_mean_shift_output_exists`, and `test_posterior_rcn_ms_standalone` could potentially be merged so the pipeline runs once and all four assert blocks check the same output.

3. **`test_multistage_growth_shape_score_strict_bic_reaches_live_sink` (11s):** Could be replaced by a unit test that calls `_apply_multistage_shape_filter()` directly on a fixture TSV, checking that `shape_score_raw` is computed with the right BIC variant.

**Exit condition:** `make test` completes in under 90 seconds without dropping real coverage. Or: the inner-loop validation is restructured so Phase 14 implementation rounds use `make test -k "not posterior"` as the default smoke target, reserving the full suite for phase-boundary validation.

---

## [ISSUE:2026-04-18:4] RMS chr-II by-eye interval evaluation needs the full chr-II surface, not summit-run proxy labels

**Status:** Active known issue. Near-term analytical follow-up from Priority 13.3.

**Urgency:** High if more RMS selector/call work is pursued.

**Why it matters:** A selector or interval-partition change that improves II/9A still needs to be
checked against the by-eye amplicon BED on the correct interval-emitting lane. Using the wrong
surface can make a step-08 interval improvement look like a regression.

**Current understanding:**
- `dev/runs/phase13_byeye_amplicon_compare_20260418/summary.tsv` is only a lightweight proxy
  built from representative summit-run labels under `dev/runs/summit_precision/`.
- It is useful for that summit-run surface, but it is not the authoritative surface for judging
  the step-08 final interval partition or threshold changes.
- The corrected full chr-II by-eye interval check on the live resolved RMS outputs shows much
  stronger behavior than that proxy implied:
  - dataset1 resolved: `overlap50_f1 0.7826`, `summit_f1 0.6957`, `summit_fn 5`
  - dataset2 resolved: `overlap50_f1 0.7500`, `summit_f1 0.6667`, `summit_fn 5`
- Resolved vs unresolved on the full chr-II lane: resolved outputs improve summit hit accounting
  and reduce summit false negatives, while unresolved outputs retain slightly higher overlap50 F1;
  Jaccard is unchanged within each dataset.
- The open issue is therefore not simply “RMS still trails HMM on by-eye.” The real issue is that
  future interval-side RMS work needs a dedicated full-chr-II by-eye comparison surface so the
  project stops mixing summit-only and interval-partition interpretations.

**Likely landing zone:**
- `tests/run_full_chrII_test.sh` and the `dev/runs/full_chrII/` outputs
- `tests/score_calls.py` plus a dedicated full-chr-II by-eye wrapper/artifact
- `scripts/rcn_summit_diagnostics.py` only for summit-side comparisons, not for step-08 interval judgment

**Exit condition:** A future RMS change is evaluated on the full chr-II by-eye interval surface,
and the project either adopts that surface as the standing interval-side check or explicitly stops
using by-eye interval behavior as a live tuning target for Priority 13.3.

## [ISSUE:2026-04-18:3] Dynamic onset / last-active-stage origin-detection window needs an explicit cross-pipeline design sketch

**Status:** Active known issue. Planning-mode follow-up for Phase 13 summit/origin work.

**Urgency:** Near-term. Important before more selector heuristics are implemented.

**Why it matters:** The current fixed-stage `early-parabola-mean` selector is only a coarse
first pass. The intended biological model is amplicon-specific: each amplicon should eventually
use its own onset stage and last summit-informative active stage when doing final summit/origin
refinement.

**Current understanding:**
- The updated BRAINSTORM entry now states the correct conceptual direction: summit localization is
  really origin detection, and the informative stage window should be defined separately for each
  amplicon.
- This needs to be made more explicit as a design sketch for all three pipelines: growth, HMM,
  and rcn-mean-shift.
- The near-term instruction is: stay in planning mode and extend the new BRAINSTORM entry into a
  more explicit dynamic onset / last-active-stage design sketch for all three pipelines.

**Likely landing zone:**
- `multi-agent/tracking/BRAINSTORM.md` first
- then the active phase spec or a follow-on phase spec once the design is explicit enough to
  implement

**Exit condition:** The BRAINSTORM entry is expanded into a concrete cross-pipeline design sketch
that identifies candidate signals, per-pipeline differences, and a plausible implementation order
for dynamic origin-detection windows.


## [ISSUE:2026-04-14:1] Stage-1 pre-amplification isolation: no mechanism to anchor non-amplified samples

**Status:** Active known issue. Design not yet started. Priority High.

**Urgency:** High — without stage-1 pre-amp anchoring, APS clustering may place partly-
amplified samples in stage 1, making the growth model's "baseline" stage biologically wrong.

**Why it matters:** The growth model measures amplification relative to a reference group.
If stage 1 contains already-amplified samples (as seen in ds2 where one stage-1 rep had
APS=1.76M vs the other at 1.15M), the baseline is inflated and downstream fold-change
estimates are underestimated. Ideally, stage 1 should contain ONLY pre-amplification
samples with APS near background.

**Current understanding:**
- The APS-based k=keep clustering cannot force pre-amp samples into a specific stage;
  it only orders by APS similarity and creates k groups of similar size.
- On ds1, `log10area+keep` and `log2summit+keep` both isolate S38 (APS=100K) alone in
  stage 1. This is because in log10 space, 100K is sufficiently far from 194K that they
  separate. This is a useful property, but it depends on APS spacing, not on any explicit
  pre-amp constraint.

**Ideas:**
- 1. Principled Thresholding for Area Excess to determine "flat" amplicons and "flat" samples:
  - For a principled way of defining what non-amplified APS is
    - We need to consider how we are computing APS (area excess, RCN, summit, whole amplicon, etc)
    - We need to consider the number of amplicons and even the number of bins within amplicons. 
      - APS threshold cannot be known in advance for a blind sum of area excess without knowing the number of bins
      - If the number of bins was known, we know area excess should be close to 0 for all bins, and defintely < 0.5 for each bin.
      - We could pick a threshold between 0-0.5.
      - Then we could say the sum should not exceed num_bins*threshold.
      - But we can also just say the average should not exceed the threshold.
  - Average Area Excess
    - If APS is area excess with a floor of zero, then non-amplified APS should be close to 0.
    - The mean bin score across each amplicon should be close to 0, and the mean of the mean bin scores across all amplicons should be close to 0.
    - Since area excess (AE) = RCN - 1, AE of 0.5 is equivalent of 1.5, so the mean and median should definitely be <0.5.
    - We could define amplicons as not amplified if their average AE < 0.5
    - Similarly, we could define the sample as not-amplified if the average of amplicon averages is < 0.5
  - Average Area Excess of Summit
    - Can choose to just look at the summit or local region around the summit for Average Area Excess.
      - local region = the summit bin +/- X bins (where X is in the 3-5 range).
    - This might be more sensitive to early amplification
    - This might be more robust against outliers (fewer bins, fewer expected outliers)
- 2. Add a step prior to APS that uses principled method to classify "flat" (background) samples vs amplified samples.
  - Simplest: if we called an amplicon in the sample, then it has amplification.
    - However, this might be too simple without first curating a set of confident amplicons (removing false positives).
  - Triangularity:
    - (1) Initial amplicon set: a union set of amplicons is defined across all stages as it is already done.
    - (2) Amplicon set refining: 
      - for each stage, each amplicon obtains a "triangle vs flat BIC score" and is classified as triangle or flat.
      - if an amplicon is classified as flat across all stages, it is removed from set (added to rejected)
      - for amplicons that grow across >= 2 stages, there should be a general increase in the triangle score 
          - Risk: we need to see if triangle scores plateua and reverse when amplicon enters elongation-only phase
    - (3) Sample set refinement:
      - for each sample, each amplicon from the refined set obtains a "triangle vs flat BIC score" and is classified as triangle or flat.
      - if >= X amplicons in a sample are classifed as flat, the sample is classified as flat.
        - else the sample is classifed as containing amplification.
        - where X is defined as 1-N, where N = number of amplicons
        - X defaults to N, meaning all amplicons are expected to be flat
          - Risk: Requiring all amplicons to be classified as flat might be too conservative and increase false negatives -- flat samples not classified as flat. It could be lowered to allow a few to be classified as triangular, but a second test can enforce a second BIC threshold. This would test if they were weakly or strongly triangular; so it would be >= X need to pass threshold 1, and of the 1-(N-X) that do not, they need to pass threshold 2.


**Observations:**
- Looking at SAG41 across all 4 chromosomes as an example - but this used summit+keep:
  - range = 0-1.8; Q10-Q90 = 0-0.7; Q25-Q75 = 0.2-0.6; Median = 0.4 ; MAD = 0.2 ; Mean = 0.4; StDev = 0.3.
  - there may be some low-level amplification present, but not likely.
  - the only two above 1 are likely spurious false positives X.
    - X:0.0-2.0 -- this is collapsed repeat rDNA collapsed repeat stuff that seems higher copy number here when looking at sample-specific chrom-median normalized RCN
    - X:57.5-58.3 -- this is a region with a bump inside the part of X with a match to X' 

**Candidate design:**
1. Add CLI flag for ____.
2. In `compute_and_write_aps()`, split samples into pre-amp (APS < threshold) and
   amplified (APS ≥ threshold) sets before clustering.
3. Pre-amp samples are assigned posterior stage 1. Amplified samples are clustered into
   k-1 stages (or k_keep stages = n_unique_amplified_stages).
4. The cluster map sidecar records which samples were pre-amp anchored vs clustered.

**Likely landing zone:** Implementation in `onionskin_core/aps.py` and `onionskin.py`,
coordinated with the APS clustering fix.

**Exit condition:** Stage 1 of the posterior contains only pre-amplification samples when
a threshold is provided or detectable, and the pipeline warns when stage 1 appears to
contain already-amplified samples.

#### Deprecated solution ideas:
- **APS Thresholds** 
  - Use an explicit `--aps-preamp-threshold` flag that forces all samples below the threshold into stage 1 before clustering the rest. For example, `--aps-preamp-threshold 500000` would place all samples with APS < 500K into a fixed stage-1 group, then cluster the remaining samples into k-1 additional stages. Add `--aps-preamp-threshold <APS_value>` flag (or `auto` to use gap detection).
    - This would not work because it is impossible to pick an APS threshold a priori on a new genome and/or new dataset
    - One would need another way to prove samples are pre-amplification (or non-amplified) and find a threshold between the highest APS of non-amplified samples and the lowest APS of amplified samples.
    - But if we have a way to separate amplified from non-amplified samples, then we would not need this also.
    - This idea COULD WORK, IF we had a principled way to determine a priori what a non-amplified APS (or something correlated with it) looks like.
    - See "Principled Thresholding for Area Excess" above:
      - Thresholding could work if it was looking at the **average** area excess for each amplicon and average of averages across all ; both should be <0.5 (i.e. close to 0)
      - It could also work if just looking at the RCN or area excess of the summit bin or the average RCN / average area excess of the summit bin +/- X bins (where X is in the 3-5 range).
  - Alternative: detect pre-amp samples automatically using a fold-change or APS ratio threshold (e.g., APS < 1.5× the median APS of the lowest-APS sample).
    - This does not work because it assumes the lowest-APS sample is non-amplified, which may be untrue.
    - Thus, there still needs to be a way to define samples as non-amplified to learn what the APS boundary is between non-amplified and amplified.


---


## [ISSUE:2026-04-18:2] `early-parabola-mean` runtime selector — extend to amplicon-specific dynamic onset / last-active-stage window

**Status:** Active — DEFERRED to cycle 15.10a-S2 F5 strategy-menu redesign (Principal direction 2026-05-05). Cycle 15.10a R2 landed a 9-strategy 2-flag `--summit-aggregation` menu intended to provide alternative selectors; that implementation is being reverted + redesigned in cycle 15.10a-S2 (curated 68-strategy single-flag design). Closure of this entry will land with cycle 15.10a-S2's redesigned menu.

**Urgency:** Near-term. Important before more selector heuristics are implemented.

**Why it matters:** The current fixed-stage `early-parabola-mean` selector is a coarse first pass. The intended biological model is amplicon-specific: each amplicon should use its own onset stage and last summit-informative active stage when doing final summit/origin refinement. Cross-references the broader cross-pipeline summit-aggregation design space tracked at `multi-agent/plans/next/SUMMIT_SOUP.md`.

**Cross-references:**
- Cycle 15.10a R2 9-strategy `--summit-aggregation` implementation (scaffold-only; superseded by 15.10a-S2 redesign per Principal direction 2026-05-05).
- Cycle 15.10a-S2 redesign in progress (Form A2 amendment formalization).
- `[ISSUE:2026-04-18:3]` (dynamic onset / last-active-stage origin-detection window cross-pipeline design sketch — companion design item).

**Exit condition:** cycle 15.10a-S2 R3 closeout lands the redesigned strategy menu with explicit amplicon-specific dynamic-window machinery; this entry closes with that closeout.

---


## [ISSUE:2026-04-18:1] APS area-excess floor default and clustering experiment

**Status:** Partially resolved by Phase 14 Supplemental 14-S22; analytical experiment remains.

**Urgency:** Near-term analytical priority.

**Why it matters:** `_locus_metrics()` in `onionskin_core/aps.py` historically floored
per-bin excess at 0 via `excess = np.maximum(loc['RCN'] - 1.0, 0.0)`. This discards all
signal below background, which may destroy information at early amplification stages where
RCN hovers near 1.0 and slight undershoot bins are meaningful context.

**Current understanding:**
- Phase 14 Supplemental added and wired `--aps-area-excess-floor on|off` across growth,
  RMS, and HMM APS. Default remains `on`, preserving historical behavior.
- The floor was introduced to avoid negative APS contributions from bins with RCN < 1
  (under-replicated or noise). This was a reasonable conservative choice.
- However, early-stage APS values may be too compressed to separate cleanly from background
  with the floor active. Removing the floor (or using plain `loc['RCN'] - 1.0`) might
  improve clustering separation at early stages.
- The experiment: re-run `clust-ds1-II`, `clust-ds2-II`, and `clust-ds2-4chr` with the
  floor removed and compare stage separation quality.

**Likely landing zone:** Phase 15 analytical comparison using the wired
`--aps-area-excess-floor off` path, including a post-sum-floor variant. See
`multi-agent/plans/next/HMM_SOUP.md` 14-S22 follow-ups.

**Exit condition:** Floor vs no floor tested and reported on. Better stage separation and
better posterior manifests demonstrated by analyzing posterior amplicons similar to
`make clust-ds2-II` and related tests that use `python scripts/aps_cluster_report.py`.
CLI flag default set to better choice.


---

## [ISSUE:2026-04-19:1] HMM step-14 APS reads raw manifest bedGraphs, not 01-mednorm files

**Status:** Active known issue. High priority — addressed by parallel child pipeline.

**Urgency:** High — step-14 APS normalization is inconsistent with steps 1-13.

**Diagnosed behavior:** `run_step14_hmm_aps()` calls `compute_sample_rcn_tracks(manifest, ...)`
which reads the **original raw bedGraph files** from the manifest and applies its own internal
chrom-median normalization (`norm_mode='chrom-median'`). It does NOT use the `01-mednorm` files
from step 1 of the HMM pipeline, and does NOT use the step-5 smoothed files. The `smooth_bins`
parameter in step-14 applies a second, independent smoothing pass on top of already-raw input.

**Why it matters:**
- Steps 1-13 operate on properly zero-bin-filtered, ratio-normalized (or chrom-median re-norm)
  data. Step-14 APS operates on a different, independently-normalized version of the same data.
- The step-5 smoothing (median or trimmed-mean) chosen by the user is NOT used by step-14.
- When the parallel child pipeline is implemented, step-14 should read from
  `05-medianSmoothedRCN/indiv_samples/` bedGraphs instead — these will already be zero-bin-
  filtered, properly normalized, and smoothed with the user's chosen halfwidth/trim setting.

**Current workaround:** None. Step-14 APS is still useful but uses slightly different data.

**Fix:** Implement the parallel child pipeline (BRAINSTORM). Then update `run_step14_hmm_aps()`
to accept a `step5_indiv_dir` path and read from there instead of the raw manifest.

**Exit condition:** Step-14 APS reads from `05-medianSmoothedRCN/indiv_samples/` bedGraphs,
consistent with all other HMM pipeline steps.

---

## [ISSUE:2026-04-19:2] Amplicon reliability scoring and flat-sample detection — PROMOTED to PHASE11_SPEC 11.5

Design is in BRAINSTORM. Now also specified as Priority 11.5 in `multi-agent/plans/PHASE11_SPEC.md`.
Implementation pending. See PHASE11_SPEC for exit conditions and implementation plan.


---


# LOW PRIORITY KNOWN ISSUES
## [ISSUE:2026-04-26:1] `--hmm-mu-scale` silently scales emission means instead of sigmas (PuffStep translation bug)

**Status:** Deferred to Phase 15. Substantive design entry lives in
`multi-agent/plans/next/HMM_SOUP.md` § "CORRECTNESS BUG:
--hmm-mu-scale translation from PuffStep is wrong + full HMM
translation re-audit required" (lines ~668–712).

**Urgency:** Low for the project (deferred); medium for any user
actively using the flag (silently incorrect).

**Why it matters:** In PuffStep, `--mu_scale` derived sigmas
(`sigma = mean * mu_scale`) and was mutually exclusive with
`--sigma`. In onionskin, the translation reversed this:
`--hmm-mu-scale` instead scales emission means
(`scaled_means = [m * mu_scale for m in emission_means]`) and does
nothing to sigmas. The help text further claims "Also rescales the
sigmas by the same factor" — which is doubly wrong (means are
scaled, sigmas are not touched). Users passing
`--hmm-mu-scale 0.5` today receive means-scaled behavior, not
PuffStep-equivalent sigma behavior.

**Current understanding:** Bug surfaced during Phase 14 Supplemental
cycle 14S.4a re-audit (per STRATEGY § 45+ correctness advisories).
Phase 15 BRAINSTORM entry mandates a full PuffStep → onionskin HMM
translation re-audit at Opus/Max Effort before fixing — the bug may
not be isolated to `--hmm-mu-scale`.

**Suggested user-side workaround until Phase 15 fix lands:** Do not
pass `--hmm-mu-scale`. Instead, set sigmas explicitly via
`--hmm-emission-sigmas` if non-default sigmas are needed. The
`--hmm-emission-sigmas` help text's cross-reference to
`--hmm-mu-scale` is also wrong and should be ignored.

**Likely landing zone:** Phase 15 (HMM completeness + correctness
pass). Fix requires (1) code correction in `onionskin.py` argparse
wiring + downstream HMM engine call site, (2) help-text correction
on both `--hmm-mu-scale` and `--hmm-emission-sigmas`, and (3) a
design decision on override priority since onionskin's
`--hmm-emission-sigmas` always has a hardcoded default
(PuffStep's `mu_scale` was reachable only when `sigma` was None).

**Exit condition:** Phase 15 HMM re-audit cycle closes; `--hmm-mu-scale`
either correctly scales sigmas (PuffStep-equivalent) or is removed
in favor of `--hmm-emission-sigmas`-only configuration; both flags'
help texts are accurate; entry is removed from this file with a
CHANGELOG cross-reference per the file's lifecycle rules.

---

## [ISSUE:2026-04-24:1] Test future `--growth-shape-filter` default flip to `on`

**Status:** Deferred validation item from Phase 14 Supplemental 14-S30.

**Why it matters:** `--growth-shape-filter` now uses explicit `on|off` semantics with
`default=off`. User direction in Q34 noted that prior experience suggested enabling the
filter by default may not change outputs, but asked to keep `off` until tested.

**Current behavior:** Users can opt in with `--growth-shape-filter on`; the diagnostic TSV
is written regardless. Bare `--growth-shape-filter` is intentionally invalid.

**Exit condition:** Run representative multistage outputs with default off versus default on.
If outputs are unchanged or the changes are accepted, flip the default to `on`, update tests,
and remove this issue.

---

## [ISSUE:2026-04-21:1] HMM `--norm-mode` default: consider switching to `chrom-median` in a future phase

**Status:** Superseded-and-reverted (v0.14.95) — cycle 15.7a Stage E (SPEC15.16) flipped the HMM `--norm-mode` resolved default from `ref-stage` to `chrom-median` for 2+-stage reference-backed inputs (Resolved v0.14.91 status was correct at that closeout). However, post-flip by-eye comparison exposed a substantive regression in chrom-median output — see the supersession entry `[ISSUE:2026-05-04:3]` HMM `--norm-mode chrom-median` regression. Cycle 15.10a Stage A (SPEC15.16 partial reversal mid-phase amendment 2026-05-04) reverted the default back to `ref-stage` (DECISIONS.md `[2026-05-04] Phase 15 cycle 15.10a Stage A` entry). The chrom-median MODE remains supported as a non-default option; chrom-median validation surface preserved via renamed `tests/test_hmm_chrom_median_explicit.py` (Cycle 15.10a Stage A renamed from `tests/test_hmm_chrom_median_default.py` + re-pinned with explicit `--hmm-norm-mode chrom-median`). The original `[ISSUE:2026-04-21:1]` framing ("consider switching to chrom-median in a future phase") remains valid for a post-Phase-15 re-flip once Concerns 2 + 3 of `[ISSUE:2026-05-04:3]` are addressed (post-GECKO_SOUP GC-aware normalization OR HMM-tuning cycle for chrom-median noise-floor).

**Original status:** Deferred design note. Phase 14 scope does not change HMM's built-in norm-mode default.

**Urgency:** Low. Current default (`ref-stage`) is correct for current HMM usage patterns.

**Why it matters:** After Phase 14's `--norm-mode` behavior change (universal default None →
per-pipeline defaults), HMM will continue to default to `ref-stage` when neither `--norm-mode`
nor `--hmm-norm-mode` is set. In a future phase, if HMM support for chrom-median normalization
is verified and validated on real data, the HMM default could be switched to match growth and
RMS (both default to `chrom-median`). This would make all three pipelines behave consistently
under `--norm-mode chrom-median` without any extra flags.

**Current understanding:** No behavioral work needed now. Phase 14 preserves HMM's `ref-stage`
default by design.

**Exit condition:** HMM chrom-median behavior validated on representative datasets; default
changed and documented in a follow-on phase spec.

---

## [ISSUE:2026-04-19:3] `--hmm-smooth-halfwidth` APS split — RESOLVED in Phase 14 Supplemental 14-S23

`--hmm-aps-smooth-halfwidth` now controls HMM step-14 APS per-locus smoothing independently
from `--hmm-smooth-halfwidth` step-5 whole-genome smoothing. When unset, it inherits
`--hmm-smooth-halfwidth`, preserving the old default behavior.

**Exit condition:** Satisfied; remove this resolved entry during the next KNOWN_ISSUES cleanup.

---

## [ISSUE:2026-04-24:2] RMS summit parabola pre-smoothing investigation

**Status:** Active follow-up from Phase 14 Supplemental 14-S27 / Q36.

**Why it matters:** Growth pre-smooths per-sample RCN before summit parabola fitting via
`--growth-rcn-smooth-halfwidth`. RMS currently calls `refine_summit_parabola` without an
analogous pre-smoothing pass, and earlier testing reportedly found pre-smoothing helpful.

**Current understanding:** HMM has separate smoothing controls (`--hmm-smooth-halfwidth`,
`--hmm-aps-smooth-halfwidth`), so promoting `--growth-rcn-smooth-halfwidth` to Universal
would double-smooth HMM. RMS needs its own evidence-driven investigation instead.

**Exit condition:** Earlier testing notes reviewed; RMS summit accuracy tested with and
without RCN pre-smoothing; if beneficial, add an RMS-specific
`--rms-rcn-smooth-halfwidth` flag.

---

## [ISSUE:2026-04-28:1] plots and notebooks should be the last directories in each pipeline

**Status:** RESOLVED v0.14.93 (cycle 15.8a). Step-less `plots/` and `notebooks/` directories adopted across all three pipelines in `onionskin_core/output_layout.py`; all Python references to `14-plots`, `15-notebook`, `11-plots`, `14-notebook` eliminated. Exit condition met: no step numbers on these directories.

---

## [ISSUE:2026-04-28:2] Setting HMM state path to be 0-based --hmm-0-based-statepath should be done as default

**Status:** Resolved (v0.14.91) — cycle 15.7a Stage B (SPEC15.14) landed the 0-based default via single value-flag form (re-scoped 2026-05-03 per Principal direction; cleaner than the original `--hmm-1-based-statepath` boolean proposal). New flag `--hmm-statepath-base {0,1}` default `0` at `onionskin.py:2327`; legacy 1-based behavior preserved via explicit `--hmm-statepath-base 1` (used for PuffStep gold-standard parity at `Makefile:124-125, 137-138, 150-151`). Internal HMM math is 0-based throughout (`onionskin_core/hmm_core.py:_viterbi:309`, `:_posterior_path:382`); user-facing labels translate at I/O boundaries via the `state_label_map` dictionary constructed at `onionskin_core/state_path_io.py:make_state_label_map`. Adaptive defaults for companion threshold flags via `argparse.SUPPRESS` + `_resolve_hmm_statepath_defaults:4437-4454`: base 0 → `thresh_state=0` + `max_state_thresh=-1`; base 1 → `thresh_state=1` + `max_state_thresh=0`. Round-trip equivalence regression test at `tests/test_hmm_statepath_base_round_trip.py` + adaptive-defaults test at `tests/test_hmm_statepath_base_defaults.py` + state-label-header round-trip test at `tests/test_state_path_io.py` + JSON sidecar schema test at `tests/test_run_pipeline_metadata_sidecars.py`. `make puff-compare` 28/28 PASS under explicit legacy flags. Self-describing `#`-prefixed canonical headers emitted at every state-value-bearing output (state-path bedGraphs at `<hmm-out>/06-HMM/*.expdecay.states.bedGraph`); Tier-1 `run_info.json` + Tier-2 `pipeline_info.json` JSON sidecars per DECISIONS.md 2026-05-03 sidecar convention. Closed in this file at cycle 15.7a R3 closeout.

**Original status:** Active. 

**Why it matters:** It is more intuitive that the background state is 0 and the first step up is 1, second is 2, third is 3, and so on. 
Moreover, the expected RCN of the state becomes 2^STATE when states are 0-based rather than 2^(STATE-1) when states are 1-based.

**What to do:** Instead of `--hmm-0-based-statepath` as an option, it should be default.
Instead, the replacement option would be `--hmm-1-based-statepath` with a help string that describes how it changes the behavior back to legacy PuffStep statepaths.
All results should be identical other than state labels.
If someone does two runs, one with 0-based states and one with 1-based states, the structure of the state path, all intervals, all stats, and everything except the arbitrary state labels should match.
The state label choice should have NO effect on actual biological, technical, and statistical results.

**Considerations:** This will affect regression tests that make sure our HMM results still match the gold standard PuffStep results. 
Regression tests already in place could use `--hmm-1-based-statepath` to compare results.
To extrapolate those results to the 0-based state paths, a new regression test could be designed to ensure equality between 0-based and 1-based results (including posterior results) on all aspects of the output except state labels.

**Exit condition:** 0-based state path is default, equality with 1-based state path results ensured, regression tests updated as described, new regression test added.


---

## [ISSUE:2026-04-29:1] outputs are not standardized across pipelines where they can be : where they can be, they should be (or should they?)

**Cycle 15.10a Stage D postscript (2026-05-05; SPEC15.20 mid-phase amendment 2026-04-29):** Status framed explicitly as **maintenance-mode** (per the SPEC15.20 amendment). Cross-pipeline output standardization is an ongoing maintenance goal with no exit condition; Phase 15 substantively advanced it (Bayesian ODW cross-pipeline, summit refinement strategy menu, APS cross-pipeline analysis-surface parity, gap-analysis architectural cleanup, summit-aggregation strategy cross-pipeline emission), but the goal itself is structurally open-ended. Future cycles continue to push toward standardization opportunistically; the cross-pipeline-parity audit checklist (per `[ISSUE:2026-04-29:7]` resolution) makes the discipline auditable on a per-cycle basis. Entry stays Active per its maintenance-mode nature; do not close.

***What:** 
- The major difference in each pipeline is how amplicons are called
- Then they all do similar things such as summit refinement, filtering, timing, aps, etc
- There is a long-term goal of having each pipeline have as high overlap in their output menus as possible, examples include:
	- Same summit refinement approaches where possible
	- Same timing analyses where possible
	- Same asymmetry analyses where possible
	- Etc
- The outputs can be the same "by analogy" or "in spirit", and do not have to take identical paths to get to the analogous output
	- They can have or be made to tolerate pipeline-specific nuances and differences
	- For example, RCN may have been computed differently, so the source of RCN for a subsequent analysis might be different
	- As another example, there may need to be various tweaks to the analysis to accomodate the pipeline architecture or anything
- If an output is truly pipeline-specific, that is okay
	- For examples, 
		- the HMM pipeline can do analyses on its statepath that other pipelines cannot do, and will therefore have HMM pipeline-specific outputs
		- the Growth pipeline can do analyses on its growth track that other pipelines cannot do, and will therefore have Growth pipeline-specific outputs
- One aim is to keep an inventory of analyses and outputs, and which pipelines generate them
	- This could be in the form of a table with 4 columns: output / analysis, HMM pipeline, Growth pipeline, RMS pipeline.
		- Checks or X's to signify which pipeline has what.
- Another aim is to fill the gaps given the inventory table
- A third aim is to keep the inventory table up-to-date and well-maintained: likely to be something for AGENT_CONVENTIONS
- A fourth aim is to keep the pipelines up-to-date with the table, to fill gaps as they arise
- All of the above is mostly focused on the inventory of analyses and outputs themselves, but another thing to standardize is the architecture of each output
	- for example, analogous output tables should have the same columns, with the same column names, in the same column order 
		- and rows should represent the same type of thing as well (e.g. each row is an amplicon, or each row is a sample, or a stage, etc)
	- for analogous tables, it should not be such that, for example, one has "chr, start, end, amplicon" as the first 4 columns and column names and the other has "amplicon_name, chrom, begin, end" as the first 4 columns and column names
		- both should have the same, either "chr, start, end, amplicon" or "amplicon_name, chrom, begin, end"
		- and the preference should be toward what is more canonical in bioinformatics
			- for example, when possible, the first three columns can be BED-like (tab-sep): "chr\tstart\tend"
	- there should be a push to standardize outputs in this way as well
		- not just ensuring analogous outputs, but ensuring that analogous outputs have shared architectural conventions
		- this is better than just allowing everything to be different by making the code catch multiple conventions
			- in cases where it is necessary for the code to tolerate multiple conventions, that is ok; but if it is just evolutionary drift between the pipelines and it can be standardized, then it should be
	- the inventory table can therefore be expanded to include a column that lists the standardized columns in order for all to follow
	- it is ok if a pipeline does not produce one of the values for a given column in an analogous output
		- it can just put "N/A" or "." or "None" or whatever makes most sense there.
			- The decision for the missing data marker should be standardized and our code should know how to deal with it if it is read in
			- The missing data marker should be a bioinformatics standard if the file is to be read by bioinformatics tools such as IGV
**Status:** Active. 

**Why it matters:** To meet user expectations, and to make it more user friendly. To prevent the burden of tolerating multiple conventions when only one is needed.

**What to do:** Deal with this in an ongoing way. Each phase can contribute to this goal. 

**Considerations:** Keep this issue in mind in brainstorming, spec engineering, audits, and even in regular chats with the user.

**Exit condition:** This is an ongoing maintenance goal and convention that basically has no exit condition. It is "done" only in a temporary way until more development happens at which point, it is active again.



---

## [ISSUE:2026-04-29:4] Posterior-regression tracking — posterior should perform as well or better than prior across tests

**Status:** Active known issue. Proposed testing-strategy upgrade.

**Why it matters:** The posterior pipeline exists because posterior results are expected to improve upon prior results — that's the biological premise. If a posterior result is *worse* than the corresponding prior result on any quality dimension, that's a bug or a calibration issue worth flagging at test time, not later. Currently, the test suite doesn't systematically compare posterior vs prior on the same dataset for quality metrics; only smoke-level "posterior runs and produces output" tests exist.

**Proposed change:** Build a posterior-regression-tracking test layer that:
1. For tests where posterior is being exercised, captures a small set of quality metrics on both prior and posterior outputs (e.g., calibration-amplicon recall, gold-standard summit precision, BIC pass-rate on known amplicons, twin-peak resolution accuracy).
2. Asserts posterior ≥ prior on each metric (or within tolerance for ties), and **flags any regression** where posterior < prior. A regression doesn't necessarily fail the test outright — it could emit a warning sidecar that the orchestrator/user reviews — but it surfaces in test output instead of going silent.
3. Becomes the default expectation for any new posterior-exercising test: prove posterior is at least as good as prior.

**Direction this aligns with:** Posterior is becoming the default test target because the posterior pipeline is where the biology-correctness payoffs land. Cycle 15.4a-S1 (this cycle) introduces posterior stage-1 anchoring + auto-switch-to-chrom-median + MAD-aware sigmas — all designed to make posterior more biologically faithful than prior. Without posterior-regression tracking, those improvements ship without a feedback loop that detects when they fail to improve outcomes (or worse, regress them).

**Risks:**
- Some quality metrics may legitimately differ between prior and posterior in either direction without indicating a bug (e.g., timing onset detection on small datasets has noise floor). Test design needs tolerance bands or flag-not-fail semantics for noisy metrics.
- Adds compute time to the test suite — every posterior-exercising test now also runs prior-side comparison metrics. Optional via opt-in flag if needed.

**Likely landing zone:** Phase 15 closeout cycle 15.10a, OR a dedicated future-phase SOUP entry if scope expands. Could also pair with `[ISSUE:2026-04-29:3]` since the flag rename naturally surfaces this question (which tests exercise posterior, and should they also assert no-regression?).

**Cross-references:** `[ISSUE:2026-04-29:3]` (`--skip-posterior` flag rename — dovetails with reviewing tests for posterior coverage), `[ISSUE:2026-04-14:1]` (the IBM origin of flat-sample detection, whose payoff is precisely posterior-vs-prior improvement).

---

## [ISSUE:2026-04-29:5] Gap-analysis is annotation-only across all 3 pipelines + HMM placement is wrong + multiple parallel keep/exclude frameworks need reconciliation

**Status:** Active known issue. **Partial resolution formalized as SPEC15.22 + cycle 15.4b in Phase 15** (HMM placement + cross-pipeline `gap_suspect` filter contract + cross-pipeline `_finalize_amplicon_recommendations` port). Full Priority-4.9-vs-reliability-scorer-vs-gap-analysis framework reconciliation is deferred to a future phase per the mid-phase amendment 2026-04-29 (see `multi-agent/project_context/DECISIONS.md [2026-04-29] Phase 15 mid-phase amendment`).

**Why it matters — three coupled architectural defects:**

1. **HMM placement defect.** HMM gap-analysis is at `01-hmm/19-gap-analysis/` — the LAST step. APS (15), SAPS (16), timing (17), clustering (18) all run BEFORE gap-analysis. Growth (`02-growth-model/12-gap-analysis/` before APS at 13) and RMS (`03-rcn-mean-shift/09-gap-analysis/` before APS at 12) have correct ordering. Only HMM is wrong.

2. **Filter defect across all 3 pipelines.** `onionskin_core/gap_analysis.py:annotate_calls_with_gap_distance()` writes `gap_distance_bp` and `near_gap` columns to `_calls.gap_annotated.tsv` sidecars but does NOT modify the active call set. No downstream consumer (APS, timing, clustering, flat-sample, reliability scorer) reads `near_gap` to filter. Gap-suspect amplicons get full downstream computation across all 3 pipelines.

3. **Multiple parallel keep/exclude frameworks exist with no integration AND `_finalize_amplicon_recommendations` is GROWTH-PIPELINE-ONLY.** Cycle 15.4a (v0.14.80) introduced `onionskin_core/reliability.py` (cross-pipeline reliability scorer + `amplicon_reliability.tsv` with `reliability_flag` that DOES filter the APS clustering matrix). Priority 4.9 (v0.4.13, archived v0.5.50) introduced `onionskin.py:269-486 _finalize_amplicon_recommendations` (growth-only integrated keep/exclude reckoning + `_amplicons_recommended_{for_exclusion,to_keep}.bed` advisory BEDs). Shape filter has its own `_amplicons_actively_rejected*.bed` (growth + RMS hard-reject). Gap analysis is its own annotation-only thing across all 3 pipelines. **None of these talk to each other.** A user reading the BED files cannot tell which framework's verdict is canonical for which downstream consumer.

**The two conflated concepts inside "recommendation-only" (per user clarification 2026-04-29):**

The current "recommendation" framing conflates two separable concerns that actually want different policies:

- **(a) User-facing recommendation:** we run analyses, produce verdicts, and surface them to the user via human-readable BEDs (`_amplicons_recommended_{for_exclusion,to_keep}.bed`) so the user can review, accept, or override. The user's refinement input back to the system is `--keep-amplicons-bed` / `--known-reliable-amplicons` (both spellings target the same `keep_amplicons_bed` argparse dest; both flags landed in cycle 15.4a, commits `40878ae` + `7ceebf3` — NOT redundant with any pre-existing flag, just two spellings).
- **(b) Internal-use filtering:** we believe our own analyses unless the user has explicitly overridden via the keep BED. Some downstream analyses tolerate noise on unreliable amplicons (per-amplicon outputs are individually inspectable; false-positive amplicons just produce ignorable rows). Other downstream analyses are *harmed* by including unreliable amplicons (sample-level APS aggregates noise into the sum; clustering on noisy features distorts cluster geometry and posterior stage assignment).

The current implementation only partially honors (b): cycle 15.4a's reliability scorer + flat-sample machinery filter the APS clustering matrix and the posterior cluster-map step. Per-amplicon outputs (timing, classifier, individual call rows) intentionally don't filter — they include reliability annotations alongside. **What's missing:** sample-level APS aggregation, cross-pipeline `_finalize_amplicon_recommendations` parity, gap-analysis integration into the filter contract.

**Cross-pipeline parity is a recurring blind spot (broader meta-finding).** "X-pipeline-only" defects keep getting exposed mid-cycle when an audit was supposed to catch them. The growth-only Priority 4.9 wiring is one example caught here. The HMM step15-aps no-MAD-emit defect (cycle 15.4a-S1) is another. The gap-analysis step-19-not-step-15 defect (this issue) is another. Project intent (per user 2026-04-29 chat): "the only true difference between pipelines is how they call amplicons. Then almost everything thereafter is game for each pipeline at least by analogy." See `[ISSUE:2026-04-29:7]` for the systemic-audit follow-up.

**Existing machinery this issue must reconcile (full cross-reference list):**

- `onionskin.py:269-486` — `_finalize_amplicon_recommendations()` (Priority 4.9; **growth-only**; advisory BEDs; integrates gap + timing + width-growth + rescue logic).
- `onionskin.py:489-599` — `_apply_multistage_shape_filter()` (growth multistage shape rejection; hard-filters; emits `_amplicons_actively_rejected_multistage.bed`).
- `onionskin_core/rcn_mean_shift_helpers.py:224-369` — RMS shape-filter rejection BED helpers (hard-filters; cross-stage parity with growth shape filter).
- `onionskin_core/gap_analysis.py:678+` — `annotate_calls_with_gap_distance()` (annotation-only across 3 pipelines).
- `onionskin_core/gap_analysis.py:788` — emits `candidate_amplicons_recommended_for_exclusion.bed` intermediate BED (consumed by `_finalize_amplicon_recommendations` in growth only).
- `onionskin_core/reliability.py` — cycle-15.4a reliability scorer (`reliability_flag`, `keep_override`, class-aware thresholds; filters APS matrix cross-pipeline). Conceptually parallel to Priority 4.9.
- `onionskin_core/flat_sample.py` — cycle-15.4a Bayesian flat-sample detector (filters samples; SPEC15.6 Round 4).
- `--keep-amplicons-bed` / `--known-reliable-amplicons` (cycle 15.4a; both new, both spellings of same dest) — user-override escape hatch via reliability `keep_override` column.
- `--timing-exclude-loci` (`onionskin.py:1391`; older flag, narrow scope) — user-input BED of regions to EXCLUDE from global timing reference computation. Functionally OPPOSITE direction of `--keep-amplicons-bed` (exclude vs keep) AND narrower scope (timing-only vs cross-pipeline). NOT redundant with `--keep-amplicons-bed` but a related fragmented user-input mechanism worth folding into the unified keep/exclude framework when the parallel-frameworks reconciliation lands. Consumed by `compute_timing_from_stage_summary(exclude_loci_bed=...)` only; not honored by reliability scorer, APS clustering, or other downstream consumers.
- IBM-C14 (Phase 14 INTENDED-BUT-MISSED) — explicit "Priority 4.9 enhancements: shape-score proximity, timing-flag, `--keep-amplicons-bed`". Cycle 15.4a partially landed it (`--keep-amplicons-bed`, class-aware reliability) but did NOT integrate with Priority 4.9 or gap-analysis.
- ROADSTRAVELED archive (`multi-agent/plans/archived/20260412-ROADSTRAVELED.md:158-217`) — full Priority 4.9 design notes; closed DONE at v0.5.50.

**Architectural design questions (LOCKED HOLD — needs explicit user discussion before any cycle implements):**

User direction (2026-04-29 chat): the parallel frameworks need to be **reconciled smartly; merge overlapping concepts; retain all non-redundant stuff; do not eliminate quality analyses; just unify what is present, not remove anything**. Decision points still to consider:

- Should the reliability scorer **subsume** Priority 4.9 (gap-analysis becomes one of the reliability evidence axes; `_finalize_amplicon_recommendations` becomes a derived view of `amplicon_reliability.tsv`; recommended-keep/exclude BEDs become a render of the reliability sidecar)? — cleanest cross-pipeline path.
- Or **port Priority 4.9 cross-pipeline** as a separate user-facing layer (two layers: reliability is APS-input filtering, Priority 4.9 is downstream user-facing recommendation-with-rescue-logic) — preserves Priority 4.9's semantically-rich rescue logic verbatim.
- Or **unify both** into a new framework that emits both `amplicon_reliability.tsv` (machine-readable) and the `_amplicons_recommended_{for_exclusion,to_keep}.bed` (user-facing summary with rescue logic) — biggest lift but most correct.
- Should gap-analysis emit a **`gap_suspect` flag** that the reliability scorer consumes as a hard-exclude axis, OR as a soft evidence axis combined with class-aware thresholds (compatible with the Bayesian-weight scheme — see `multi-agent/tracking/BRAINSTORM.md [2026-04-29] Bayesian-weighted amplicon reliability`)?
- For the HMM placement: HMM gap-analysis natural slot is right after multistage-unification at step 14, before APS at step 15. Step renumber yet again.

**Proposed near-term resolution path:**

1. **Phase 15 partial fix candidate (SPEC15.22 or 15.10a closeout work):** relocate HMM gap-analysis to between multistage-unification and APS; make gap-analysis filter the active call set OR feed `gap_suspect` into reliability; cross-pipeline harmonize so all 3 pipelines have the same gap-analysis order + filter contract.
2. **Phase 15+ or Phase 16:** full keep/reject reconciliation per user direction — merge the parallel frameworks; cross-pipeline port `_finalize_amplicon_recommendations`; preserve all non-redundant analyses; address the user-facing-vs-internal-use conceptual split explicitly.

**Risk of in-Phase-15 partial fix:** gap-analysis filtering is a real behavioral change for downstream outputs — flat-sample anchoring, APS clustering, timing classifier all start operating on a smaller call set when gap-suspects get filtered. PuffStep parity unaffected (HMM detection upstream of gap-analysis). `make twin` / `make full-twin` may shift if twin-peak amplicons happen to be near gaps; needs validation runs on calibration datasets.

**Landing zone (formalized 2026-04-29 mid-phase amendment):** SPEC15.22 + cycle 15.4b in Phase 15 — partial reconciliation only (HMM placement + cross-pipeline `gap_suspect` filter + cross-pipeline `_finalize_amplicon_recommendations` port). Full Priority-4.9-vs-reliability-scorer-vs-gap-analysis framework reconciliation is deferred to a future phase.

**Cross-references:**
- `[ISSUE:2026-04-29:1]` (cross-pipeline output standardization).
- `[ISSUE:2026-04-29:3]` (`--compute-aps`/`--posterior` rename).
- `[ISSUE:2026-04-29:4]` (posterior-regression tracking).
- `[ISSUE:2026-04-29:6]` (sample-level APS does not use reliability filtering — actionable subset of this issue).
- `[ISSUE:2026-04-29:7]` (systemic cross-pipeline parity audit gap — meta).
- `multi-agent/tracking/BRAINSTORM.md [2026-04-29] Bayesian-weighted amplicon reliability` (more sophisticated reliability framing).
- IBM-C14 archived (Phase 14 INTENDED-BUT-MISSED; partially resolved by cycle 15.4a).

---

## [ISSUE:2026-04-29:6] Sample-level APS aggregation does not use reliability-filtered loci (potentially harmed by unreliable amplicons)

**Status:** Resolved (v0.14.90) — cycle 15.6a-S1 landed the `--aps-aggregation-{mode,score,score-min}` flag matrix + `--aps-emit-unfiltered-aggregates` side-car, replacing the legacy `--aps-weight-loci` (now retired). Default `--aps-aggregation-mode=weight` + `--aps-aggregation-score=joint` makes sample-level APS aggregation reliability-and-importance-aware out of the box; `filter` mode supports the original binary-filter framing. Independent max-normalization preserves `joint ∈ [0, 1]`. Cross-pipeline parity verified at cycle close per cycle 15.6a-S1 F8 parity table. Principled redesign + relocation of the underlying scorers continues at `[ISSUE:2026-05-03:1]` (post-cycle-15.8a verification scope). The narrower `[ISSUE:2026-04-29:6]` ask — give sample-level APS reliability-aware aggregation — is closed.

**Why it matters:** Per user direction 2026-04-29: "sample-level APS is potentially harmed" if it aggregates over unreliable amplicons, because false-positive amplicons contribute their (noisy) area to the sum. The user explicitly called out APS aggregation + clustering as the major examples where reliability filtering matters — clustering is already filtered (cycle 15.4a Stage J), but sample-level APS aggregation is NOT.

**Current behavior:** `aps.py:236+ compute_aps_tables()` builds `aps_df` (per-sample sums of `area_excess`, `summit_excess`, `width_above_threshold_bp`, etc. across ALL loci passed in). The reliability scorer + flat-sample machinery (cycle 15.4a Stage J) filter the CLUSTERING matrix downstream — but the original `aps_df` written to `onionskin_aps.tsv` aggregates over the unfiltered locus set. So `APS_area`, `APS_summit`, `APS_width_1p5`, `APS_global_mean`, `APS_sum_of_means`, `APS_mean_of_means` all include unreliable amplicons' contributions.

**Where the gap is:**
- `onionskin_core/aps.py:1402` — `aps_df.to_csv(os.path.join(out_dir, 'onionskin_aps.tsv'), sep='\t', index=False)` writes the unfiltered aggregates.
- `onionskin_core/aps.py:1403` — `contrib_df.to_csv(...)` ALSO writes unfiltered (per-locus annotated with reliability columns, so users can re-aggregate, but the TSV header doesn't make that obvious).

**Proposed fix (binary version — simplest):**

1. Compute `aps_df_reliable` from reliability-filtered `contrib_df` (same code path as the matrix filter).
2. Either replace `aps_df` with `aps_df_reliable` (cleaner; behavioral change at default invocation) OR emit BOTH `onionskin_aps.tsv` (unfiltered, current) + `onionskin_aps_reliable.tsv` (filtered, new) so users can compare.
3. Add columns to `aps_df` capturing the filter decision: `n_loci_used_in_aps`, `n_loci_reliable`, etc. — so the audit trail is in the file.
4. Apply `--keep-amplicons-bed` / `--known-reliable-amplicons` overrides via the existing `keep_override` mechanism.

**Proposed fix (Bayesian-weighted version — more sophisticated):**

Per user direction 2026-04-29 + see `multi-agent/tracking/BRAINSTORM.md [2026-04-29] Bayesian-weighted amplicon reliability`: instead of binary filter, weight each locus's contribution by `reliability_score` (already exists in `amplicon_reliability.tsv` at the per-call level). Sample-level APS becomes `sum(area_excess × reliability_weight)` instead of `sum(area_excess)`. This is the more sophisticated Bayesian-shrinkage version — locked-out amplicons (weight ≈ 0) contribute nothing; high-confidence amplicons (weight ≈ 1) contribute their full area; ambiguous ones contribute fractionally.

**Existing partial machinery that supports either fix:**

- `--aps-weight-loci` flag exists at `aps.py:1152` — currently weights `area_excess × locus_weight` and `locus_weight = 1.0` always (placeholder per ROADMAP Priority 5.2). Cycle 15.4a wired `weight_col = 'reliability_score' if 'reliability_score' in contrib_df.columns else 'locus_weight'` at `aps.py:1386`. **So `--aps-weight-loci` IS already a soft form of the Bayesian-weighted fix** — but it's opt-in (default off) and only affects `weighted_area_excess` / `APS_area_weighted` columns, not the canonical APS columns. Promoting `--aps-weight-loci` to default on (or making the canonical APS columns use weighted aggregation) is a small step.
- `attach_reliability_to_contributions()` and `filter_contributions_for_reliability()` in `reliability.py` are the primitives that already exist for the binary filter case.

**Risk:** changing default APS aggregation is a behavioral change for any existing analysis pipeline that consumes `onionskin_aps.tsv`. Probably want side-by-side outputs (`onionskin_aps.tsv` unfiltered + `onionskin_aps_reliable.tsv` reliability-filtered or weighted) at first, then flip the default in a later cycle once external consumers are updated.

**Likely landing zone:** Phase 15 cycle 15.6a (SPEC15.9-12 APS analytical work) is the natural home — that cycle's clustering-defaults finalization is exactly the place to address aggregation + weighting decisions in concert. Could also land partial fix earlier as a small follow-up cycle.

**Cross-references:**
- `[ISSUE:2026-04-29:5]` (parallel keep/exclude frameworks need reconciliation — this is a concrete subset).
- `multi-agent/tracking/BRAINSTORM.md [2026-04-29] Bayesian-weighted amplicon reliability` (sophisticated form of the fix).
- ROADMAP Priority 5.2 (mentioned in `aps.py:1198-1212` — `--aps-weight-loci` flag is the partial precursor).

---

## [ISSUE:2026-04-29:8] "Flat" terminology is overloaded — disambiguate flat-sample vs flat-shape across code + docs

**Status:** Active known issue. Language clarity / API hygiene.

**Why it matters:** The word "flat" is used for two semantically distinct concepts in the onionskin codebase, which creates confusion in code, docstrings, CLI help text, output column names, and user-facing documentation:

1. **Flat sample (RCN ≈ 1 throughout)** — a SAMPLE that has NO amplicon growth. Truly flat — flat to the ground / at chrom-median baseline. Cycle 15.4a's `flat_sample.py` + `flat_sample_flag` + `--flat-sample-*` flags + `flat_samples.tsv` sidecar are all this concept. The flat-sample-detection contract is "is this sample showing amplification at any reliable locus?"
2. **Flat shape (rectangular / plateau / tabletop)** — an AMPLICON SHAPE that is constant ABOVE background but does NOT have triangular gradient structure. The shape filter's `dBIC_flat_vs_tri` discriminant compares "flat" (= rectangular tabletop) vs "triangle" (= peak-and-shoulders gradient). Flat amplicons typically indicate collapsed repeats or other constitutional copy-number features rather than developmental amplification. The `--shape-score-threshold` machinery is this concept.

**Examples of overloading in current code:**
- `onionskin_core/flat_sample.py` — the "flat sample" concept (#1).
- `onionskin_core/rcn_mean_shift_helpers.py:18` — `dBIC_flat_vs_tri` — the "flat shape" concept (#2).
- `tests/compare_strict_bic.py` — likely the shape-flat concept (#2).
- `aps.py:888-890` docstrings — discusses MAD-of-medians "approximately Gaussian by the CLT" — neither sense, but the word "flat" appears nearby in sigma discussions.
- BRAINSTORM 2026-04-19 design discussion — the entry uses "flat" for both senses without disambiguation: "Flat-sample detection: A sample is 'flat' if it is flat at essentially all reliable amplicon loci" (#1) AND "triangle vs flat BIC score" (#2).

**Why this matters in practice:**
- Future agents reading the code (or future user reading help text) can misinterpret which "flat" is meant — especially since both concepts intersect in the per-locus flat-sample test (which uses the per-locus RCN baseline, NOT the shape filter, but a confused reader might assume "flat-sample test" uses the dBIC_flat_vs_tri).
- Cross-pipeline harmonization gets harder when two distinct concepts share a word.
- Bayesian-weighted reliability brainstorm (`multi-agent/tracking/BRAINSTORM.md [2026-04-29] Bayesian-weighted amplicon reliability`) discusses both concepts; using disambiguated terms would improve clarity.

**Proposed disambiguation:**
- **Concept #1 (truly flat at baseline):** keep "flat sample" / "flat at locus" as the canonical phrasing. Optionally promote to "baseline-flat" or "preamp-flat" in future-facing language to make the "RCN ≈ 1 / no growth" meaning explicit.
- **Concept #2 (rectangular plateau above background):** retire "flat" in favor of "rectangular" or "tabletop" or "plateau" or "non-triangular" or "constant-above-background." Or keep "flat" but always pair it with "vs triangle" or "shape" for context (e.g., `dBIC_flat_vs_tri` is OK because the `_vs_tri` makes the shape-comparison context explicit).

**Likely landing zone:** Phase 15 cycle 15.10a closeout (SPEC15.20 housekeeping sweep) — language hygiene fits naturally with the tracking-file cleanup pass. Could also be addressed earlier within any cycle that touches the affected modules — should not block any cycle, but each touched file is a natural opportunity to apply the disambiguation. Could also justify a small dedicated future-phase task ("language hygiene sweep") if the user prefers a single coordinated rename.

**Risks:**
- Renaming `dBIC_flat_vs_tri` (or any column / function / flag with "flat" in the shape sense) is a breaking change for tests, docstrings, help text. Coordinate carefully.
- "Flat sample" itself probably stays — it's the more obviously-correct sense (truly flat at baseline).
- Cross-pipeline impact: shape-filter "flat vs triangle" terminology is used in growth + RMS today; HMM gains it in cycle 15.4a. Consistent rename across all 3.

**Cross-references:**
- `[ISSUE:2026-04-29:5]` (parallel keep/exclude framework reconciliation — terminology hygiene fits the reconciliation discussion).
- `[ISSUE:2026-04-29:7]` (systemic cross-pipeline parity audit — language consistency is one of the audit dimensions).
- `multi-agent/tracking/BRAINSTORM.md [2026-04-29] Bayesian-weighted amplicon reliability` (uses both senses of "flat" inline; would benefit from the disambiguation).




---


## [ISSUE:2026-04-30:3] flat sample detection and amplicon reliability may not be working as expected

**Status:** Resolved (v0.14.87) — cycle 15.4a-S2 closed at v0.14.87. Stage B.1 added `--flat-sample-norm-mode {chrom-median, ref-stage}` (default `chrom-median`) so the per-locus flat-sample test consumes a chrom-median-normalized signal independent of the (polluted) ref-stage anchor; `compute_flat_sample_sidecar(...)` and `write_flat_sample_sidecar(...)` accept a new `signal_contrib_df` kwarg, and Growth/RMS APS (`onionskin_core/aps.py:compute_and_write_aps`) + HMM APS (`onionskin_core/hmm_ported_analyses.py:run_step16_hmm_aps`) build the alternate chrom-median contrib_df + pass it through. Stage B.8 split flat-sample defaults from ODW active-stage defaults: `DEFAULT_FLAT_SAMPLE_PROB_THRESHOLD=0.7` and `DEFAULT_FLAT_SAMPLE_FOLD_THRESHOLD=1.10` (`onionskin_core/flat_sample.py:49-50`); explicit `--flat-sample-prob-threshold`/`--flat-sample-fold-threshold` overrides preserved. Stage B.7 (HMM step14 shape-score harmonization to last-stage profile + 100-bin context, threshold restored to 50 cross-pipeline; `onionskin_core/hmm_multistage_unification.py:_shape_score_for_row` + `_latest_stage_context_profile`) lifted HMM chr-II keep count from 3/12 to 9/12 step13 trajectories on the user fixture; the 3 remaining are HMM-detection false-positives (very low shape scores 1.55/5.86/26.53 + state-path-evidence failures), not by-eye-confirmed amplicons. User-eyes verification at `dev/runs/15-4a-S2-user-eyes-flagfix/`: SAG40 `flat_sample_flag=False` (was True), `n_reliable_loci=9 / n_flat_loci=4 / flat_fraction=0.444`, II/9A locus `summit_rcn=1.20` (chrom-median signal). Cross-pipeline counts on the same fixture: Growth 10 final chr-II (8 APS reliability pass) / RMS 13 pass / HMM 9 pass. Validation gates: targeted pytest suites PASS, `make test` PASS (183 passed, 1 skipped), `make toy/twin/full-twin` PASS, `make puff-compare` PASS 28/28 (HMM detection bit-for-bit unchanged on freshly re-run post-R2 tree). The systematic-error noise-source design flaw described in `[SURPRISE:15:15.4a-S2:1:A:1]` (within-stage between-sample MAD as the per-locus flat-test sigma) is OUT OF SCOPE for this cycle and lands in cycle 15.4a-S3 per user direction 2026-05-01 (new `--flat-sample-sigma-source` flag with chrom-MAD per-sample bedGraph emission). The reliability-scorer was NOT the false-negative bottleneck per R1's audit — the chr-II amplicon count gap surfaces upstream of `run_step16_hmm_aps` (HMM step14 multistage-unification rejection class), not at reliability scoring; this cycle's structural shape-score fix moved the count from 3 → 9 / 12 chr-II step13 trajectories.

**Landing zone (historical):** Phase 15 cycle 15.4a-S2 — supplemental defect-fix to cycle 15.4a SPEC15.6 deliverables. Mid-phase amendment formalized 2026-04-30 (post-cycle-15.6a-R3-closeout discussion). Cycle 15.4a-S2 fired AFTER cycle 15.6b closed (cycle 15.6b's strategy-selection determinism fix is load-bearing for cluster-stability across runs which the flat-detector tuning + reliability-scorer threshold calibration consume — without determinism, threshold calibration to ground-truth is confounded by run-to-run noise). See `multi-agent/plans/PHASE15_AUDIT_LOG.md` cycle 15.4a-S2 section for the concrete data evidence + diagnostic questions captured at amendment-authorization time. See `multi-agent/plans/PHASE15_STRATEGY.md` amendment-log dated 2026-04-30 for scope-boundary lock (tune existing SPEC15.6 deliverables to ground-truth data; do NOT redesign the detector — that's future-phase work; GC-aware chrom-median normalization deferred to `multi-agent/plans/next/GECKO_SOUP.md`).

### What:
- sample with known light amplification is being called flat
	- At least 1 sample where II/9A, the earliest and eventually largest amplicon, has started amplifying (obvious at the summit), but this sample is called flat
	- The effect if used as ref-stage in the posterior run is continued distortion of summit region --> the exact opposite of what we want
- looking at 01-prior/01-hmm/16-aps/amplicon_reliability.tsv
	- there are only 8 amplicons there
	- is that expected?
	- that is way lower than expected based on by-eye amplicon analysis --> 10-13 on II vs 3 on II, for example. There are for sure 5-7 really good ones on II.

### When onionskin was run:
- 2026-04-30 10:10 AM EST
- Phase 15 mid-phase amendment 2026-04-30 (post-cycle-15.6a-R3-closeout)

### Command
```
python "${SCRIPT}" \
  --threads 0 \
  --manifest "${BATCH_5KB}" \
  --hires-manifest "${BATCH_1500BP},${BATCH_500BP}" \
  --pipelines hmm,growth,rms \
  --chromosomes "${CHROMS}" \
  --min-seq-length 50000 \
  --min-bin-count-per-seq 10 \
  --out-dir "${OUTDIR}" \
  --out-prefix "${OUTPRE}" \
  --stage-weight-mode "${STAGEWEIGHT}" \
  --growth-fit-method "${GROWTHMODEL}" \
  --norm-mode "${NORMMODE}" \
  --growth-norm-mode chrom-median \
  --rms-norm-mode chrom-median \
  --hmm-norm-mode ref-stage \
  --compute-aps \
  --aps-cluster-k "${APSK}" \
  --aps-feature "${APSFTR}" \
  --aps-rank-by area \
  --aps-scale both \
  --aps-cluster-method hierarchical \
  --aps-width-threshold 1.5 \
  --aps-focus-loci 1 \
  --onset-rcn-threshold 1.5 \
  --onset-span-rcn-threshold 1.5 \
  --onset-method rcn \
  --onset-z-threshold 3.0 \
  --timing-onset-quantile 0.10 \
  --growth-rcn-smooth-halfwidth 3 \
  --near-gap-bp 10000 \
  --ref-stage 1 \
  --debug \
  --hmm-bin-size 5000 \
  --hmm-smooth-halfwidth 3 \
  --hmm-emission-means 1,2,4,8,16,32,64,128 \
  --hmm-emission-sigmas 0.25,0.5,1,2,4,6,8,24 \
  --hmm-expected-background-length 1000000 \
  --hmm-expected-amp-step-length 75000 \
  --hmm-background-idx 0 \
  --hmm-init-background 0.997 \
  --hmm-exp-decay on \
  --hmm-exp-decay-scale 1.0 \
  --hmm-emission-model normal \
  --hmm-decode-path viterbi \
  --hmm-training viterbi \
  --hmm-iters 1 \
  --hmm-converge 1e-9 \
  --hmm-emitpseudo 1e-7 \
  --hmm-learn-pseudo 1e-300 \
  --hmm-thresh-state 1 \
  --hmm-merge1 0 \
  --hmm-min-width 0 \
  --hmm-merge2 0 \
  --hmm-max-state-thresh 0 \
  --hmm-ms-min-width 0 \
  --hmm-pseudo 0 
```

**Results:** 
- See here for results:
--> /Users/johnurban/searchPaths/github/onionskin/dev/share/res20260430-1
- The user only did a limited search, looking mostly in: 01-prior/01-hmm/16-aps/
- results on II are worth comparing to tests/full_chrom_training_data/amplicons.by-eye.bed 

**Status:** Active. 

**Why it matters:** These need to work right - otherwise, what's the point? 

**What to do:** 
### Flat Sample Stuff
- User needs to provide examples of truly flat samples, and examples that should not be called flat, but are being called flat.
- Tweaks need to be made to match real data
	- Could be tweaks to default
	- Could be tweaks to code 
		--> is it checking the summit area for growth or the entire amplicon? 
		--> is it being too permissive or too strict (depending on what we are looking at?)

### Amplicon reliability
- needs to be better tuned to ground truth data
- need to identify which reliability analyses are working as intended, and which are not
	- which are most permissive in calling an amplicon reliable? and which ones are most conservative (causing a lot of false negatives)?
- potentially update what analyses are used or how we weight them or how they are tuned

**Exit condition:** These things work to user's satisfaction.

---
