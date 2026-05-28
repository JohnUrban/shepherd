# DECISIONS

<!-- RULES
- Append-only (no edits to past entries)
- No duplication of CHANGELOG
- Must include rationale
- Must NOT imply global applicability unless stated
- If reasoning matters → DECISIONS.md. If it's a one-time implementation note → CHANGELOG.md.
-->

---

## [2026-05-04] Phase 15 cycle 15.10a Stage B — `--onset-*` vs `--timing-*` two-prefix scheme documented (SPEC15.19 d1 closes [ISSUE:2026-04-25:1])

- Context:
  - The Timing argparse group at `onionskin.py:1647` mixes flags with two prefixes: `--onset-*` (4 flags: `--onset-rcn-threshold`, `--onset-span-rcn-threshold`, `--onset-method`, `--onset-z-threshold`) and `--timing-*` (2 flags: `--timing-onset-quantile`, `--timing-exclude-loci`). `[ISSUE:2026-04-25:1]` flagged the inconsistency as a deferred naming-cleanup item from Phase 14 Supplemental 14-S13.
  - Cycle 15.10a R1 audit OQ2 raised the scoping decision: SINGLE PREFIX (rename `--onset-*` to `--timing-onset-*` with deprecation aliases) vs DOCUMENTED TWO-PREFIX SCHEME.

- Decision:
  - DOCUMENTED TWO-PREFIX SCHEME locked. No code rename. No deprecation aliases. Add a one-sentence preamble to the Timing argparse group help text describing the scheme; close `[ISSUE:2026-04-25:1]` per file lifecycle.

- Rationale:
  - The semantic distinction is real and worth preserving:
    - `--onset-*` flags govern the **onset-stage detection algorithm** (the detector primitive that decides which stage a locus first becomes active). These configure the detector itself.
    - `--timing-*` flags govern **post-detection timing calculations** (quantile mapping for the global timing reference, exclusion filtering). These configure what happens after the detector has run.
  - Renaming `--onset-*` → `--timing-onset-*` would cascade into deprecation churn for cosmetic gain — the user-facing distinction between detector primitive flags and post-detection-calculation flags would be lost rather than improved.
  - This decision parallels DECISIONS.md `[2026-04-29] Phase 15 SPEC engineering naked --onset-* prefix retention` which established the same retain-don't-rename precedent for the broader cross-pipeline `--onset-*` flag family.

- Scope:
  - Applies to the Timing argparse group flag-naming convention. Does not apply to other argparse groups; future groups should establish their own naming conventions per their semantic structure.

- Authors:
  - John M. Urban, Claude Code 2.1.126 (claude-opus-4-7 ; Effort: Max).

---

## [2026-05-04] Phase 15 cycle 15.10a Stage A — HMM `--norm-mode` default reverted from `chrom-median` back to `ref-stage` (SPEC15.16 partial reversal mid-phase amendment)

- Context:
  - Cycle 15.7a SPEC15.16 (~2026-05-04 earlier) flipped the HMM `--norm-mode` resolved default from `ref-stage` to `chrom-median` for reference-backed 2+ stage inputs at `onionskin.py:_default_pipeline_norm_mode`. Rationale at the time: chrom-median normalization avoids the early-amplified-stage-1 anchor pollution problem; planning intent treated chrom-median as the more biologically defensible default for HMM detection.
  - Post-flip by-eye comparison against the established ref-stage production baseline (full run at `dev/share/res20260504-1/`, comparing `02/hires_results/` ref-stage vs `03/hmm_chrom_med/` chrom-median) exposed substantive regressions documented in `KNOWN_ISSUES.md [ISSUE:2026-05-04:3]`.

- Decision:
  - Cycle 15.10a Stage A flips `_default_pipeline_norm_mode("hmm", n_stages>=2, has_reference_stage=True)` back to `"ref-stage"` (the long-standing production default before cycle 15.7a). The `chrom-median` MODE itself is preserved as a non-default option via explicit `--hmm-norm-mode chrom-median` override.
  - The cycle-15.7a-introduced `tests/test_hmm_chrom_median_default.py` is renamed to `tests/test_hmm_chrom_median_explicit.py` and re-pinned with explicit `--hmm-norm-mode chrom-median` (per OQ1 Principal lock at cycle 15.10a R1 audit time). Preserves a non-default-mode regression gate; chrom-median validation surface stays usable for future HMM tuning work.
  - `Makefile` `puff-compare` targets retain their explicit `--hmm-norm-mode ref-stage` flags (defensive pinning; insulates PuffStep gold-standard parity gate from any future re-flip churn).

- Rationale:
  - Quantitative impact (same input data, same run): chrom-median produces 506 amplicon trajectories vs ref-stage's 57 (8.9× more) genome-wide. chrom-median stage_1 produces 138 summit-state regions where ref-stage detects 0; stage_2 produces 439 vs 11 (40× more). Pattern is genome-wide, not locus-specific.
  - At the two best-characterized amplicons (II/9A and II/2B), ref-stage matches by-eye HMM gold-standard expectations bp-perfectly (8/8 across 2 loci × 4 stages); chrom-median produces spurious distal summit-state regions at amplicon edges and over-detects at stage_1.
  - PuffStep parity gate has historically only exercised ref-stage mode; chrom-median was a new default without equivalent gold-standard validation coverage. The regression slipped through automated testing because there was no parity gate exercising the new default.
  - Returning to the production-validated baseline restores the guarantee that the HMM default behaves in agreement with by-eye expectations + the gold-standard parity gate.

- Conditions for future re-flip (post-Phase-15):
  - Post-GECKO_SOUP GC-aware normalization phase that addresses the per-bin technical bias that chrom-median normalization preserves but ref-stage subtracts via its baseline step.
  - OR a dedicated HMM-tuning cycle that addresses chrom-median noise-floor (Concerns 2 + 3 of `[ISSUE:2026-05-04:3]` — fork-tracking algorithm robustness when fed noisy upstream data; HMM transition probabilities, state priors, cleanup-step filtering for chrom-median's noisier input).

- Scope:
  - Applies to Phase 15 cycle 15.10a Stage A only (the partial reversal). Does not retract the cycle-15.7a SPEC15.16 design intent; chrom-median remains a supported mode that may become the default again in a future phase once the pre-conditions above are met.

- Authors:
  - John M. Urban, Claude Code 2.1.126 (claude-opus-4-7 ; Effort: Max).

---

## [2026-05-04] Phase 15 cycle 15.9a SPEC15.21 implementation locks

- Context:
  - Cycle 15.9a R1 raised five open questions around the ODW threshold split, the Growth engine bridge, and the second-pass background control surface.
  - Principal direction during the R2 implementation session locked the behavioral/default-preservation choices before the code landed.

- Decision:
  - `run_multistage(...)` is completed as a Namespace-based controller-to-engine bridge. `onionskin.py` builds a Namespace and passes it directly into `onionskin_core/engines/growth_model_engine.py`; no maintained engine argparse shim or argv transport remains in the live path.
  - The ODW split preserves the live-tree defaults: active-direction probability stays `0.9`, regression-direction probability stays `0.8`, and both directions keep a fold threshold of `1.25` unless the user overrides them.
  - The second-pass background rename/split is locked to `--prior-second-pass-background-estimation`, `--posterior-second-pass-background-estimation`, and `--posterior-inherited-mask {local,union,intersect}`. `--second-pass-background-floor` is intentionally unchanged in this cycle.

- Rationale:
  - The Namespace bridge completes the 2026-04-24 engine-CLI simplification decision instead of extending the transitional Growth parser yet again.
  - The ODW split was meant to expose two biologically distinct directions without silently tightening APS regression defaults. Keeping active at `0.9` and regression at `0.8` preserves live behavior while still making the asymmetry explicit and user-configurable.
  - Prior/posterior second-pass controls are distinct concepts. Keeping the floor flag unchanged avoids unnecessary deprecation churn on a parameter whose meaning did not change.

- Scope:
  - Applies to Phase 15 cycle 15.9a SPEC15.21 implementation and closeout only.
  - Does not decide any future-phase detector redesign, posterior consensus policy beyond `local|union|intersect`, or any rename of `--second-pass-background-floor`.

- Authors:
  - John M. Urban, GitHub Copilot (GPT-5.4 ; Reasoning: Medium)

---

## [2026-04-30] Phase 15 cycle 15.4a-S2 cross-pipeline norm-mode default audit

- Context:
  - Cycle 15.4a-S2 Stage A.4 required a lightweight audit of the live
    cross-pipeline normalization defaults before implementing flat-detector
    and posterior auto-switch repairs.
  - The user-evidence run at `dev/share/res20260430-1/` used
    `--growth-norm-mode chrom-median`, `--rms-norm-mode chrom-median`, and
    `--hmm-norm-mode ref-stage`, which matches the defect scenario: HMM's
    ref-stage normalization can be polluted by early amplified stage-1
    samples, while chrom-median paths avoid that specific anchor dependency.

- Decision:
  - Treat this as evidence for cycle 15.4a-S2 only. Do not flip durable
    pipeline normalization defaults in this supplemental defect-fix cycle.
  - Growth's built-in default is `chrom-median`.
  - RMS's built-in default is `chrom-median` for single-stage and multistage
    with a reference stage, but `ref-stage` for the two-stage-with-reference
    contract.
  - HMM's built-in default is `chrom-median` for single-stage and `ref-stage`
    for two-or-more stages when a reference stage exists.
  - Cycle 15.4a-S2 may use chrom-median as the flat-detection signal source
    and posterior auto-switch fallback, but the HMM durable default-flip
    decision remains owned by SPEC15.16 / cycle 15.7a.

- Rationale:
  - The user-visible defect is not "all pipelines have the wrong durable
    default"; it is that HMM ref-stage-normalized flat detection can consume a
    polluted anchor and then feed posterior anchoring decisions.
  - The narrow fix is to decouple flat detection from the polluted ref-stage
    signal and to make posterior auto-switch state per-pipeline. That solves
    this cycle's defect without preempting SPEC15.16's validation-driven HMM
    default-flip protocol.

- Scope:
  - Applies to Phase 15 cycle 15.4a-S2 Stage A/B only.
  - Does not decide SPEC15.16, Growth/RMS future default policy, GC-aware
    chrom-median normalization, or dual-mode output UX.

- Authors:
  - John M. Urban, Codex-cli 0.126.0-alpha.8 (GPT-5.5 ; Reasoning: Extra High)

---

## [2026-04-30] Phase 15 cycle 15.6b sort-order audit and shared-grid invariant hardening

- Context:
  - Phase 15 cycle 15.6b SPEC15.23 required a B-Common audit over
    `os.listdir(...)`, `glob.glob(...)`, and `sample_ids[0]`-style canonical
    sample selection surfaces after the Stage A determinism diagnostics.
  - Stage A.2 found no reproduced run-to-run variance on the current tree for
    the 5x toy recipe, but Stage A.3 verified that several code paths were still
    implicitly relying on the first sample's bedGraph grid and on filesystem
    enumeration order remaining stable.
  - The sweep identified same-surface deterministic-contract sites in
    `onionskin_core/aps.py`, `onionskin_core/aps_pca.py`,
    `onionskin_core/engines/growth_model_engine.py`,
    `onionskin_core/engines/rcn_mean_shift_engine.py`,
    `onionskin_core/rcn_mean_shift_helpers.py`, and
    `onionskin_core/summit_convergence.py`.

- Decision:
  - **Treat the shared bedGraph grid as an explicit contract, not an implicit
    alphabetical-first convention.** Validate the grid once at a shared helper
    boundary and reuse the validated reference grid rather than silently letting
    `sample_ids[0]` choose the canonical bins.
  - **Treat filesystem enumeration order as unspecified.** Any multi-file
    reduction on determinism-sensitive surfaces must use `sorted(...)` over
    `os.listdir(...)` results before concatenation or downstream aggregation.
  - **Prefer loud failure over silent fallback on determinism-critical paths.**
    If the shared-grid contract or a stage-worker result breaks, raise loudly
    and diagnose it instead of auto-healing with union-grid synthesis or empty
    DataFrame substitution in this cycle.

- Rationale:
  - Stage A.3 verified the shared-grid invariant on the toy manifest inputs plus
    transformed RMS and HMM per-sample tracks, so union-grid reconstruction
    would add complexity without solving an observed bug on the audited path.
  - Explicit sorting removes a latent APFS/POSIX ordering dependency at near-zero
    runtime cost and makes the deterministic contract obvious in code review.
  - The old silent behavior allowed identical inputs to depend on whichever
    sample happened to occupy the first position or on whether a failing worker
    was quietly replaced with empty outputs. For Phase 15's clustering and
    strategy-selection work, loud invariant failure is the safer contract.

- Scope:
  - Applies to the Phase 15 cycle 15.6b determinism-fix surfaces in
    `onionskin_core/common.py`, `onionskin_core/aps.py`,
    `onionskin_core/aps_pca.py`,
    `onionskin_core/engines/growth_model_engine.py`,
    `onionskin_core/engines/rcn_mean_shift_engine.py`,
    `onionskin_core/rcn_mean_shift_helpers.py`,
    `onionskin_core/summit_convergence.py`, and the same-cycle regression test
    coverage in `tests/test_determinism.py`.
  - Does NOT authorize a broader HMM step-number refactor, cross-pipeline
    union-grid normalization policy, or Stage D defaults work from cycle 15.6a-S1.

- Cross-references:
  - `multi-agent/plans/PHASE15_AUDIT_LOG.md` cycle 15.6b Role 1 + Role 2 entries
  - `[SURPRISE:15:15.6a:1:A:5]` in `multi-agent/plans/PHASE15_SURPRISE_LOG.md`
  - SPEC15.23 Stage A.3 / Stage B Common

- Authors:
  - John M. Urban, GitHub Copilot (GPT-5.4 ; Reasoning: Xhigh)

---

## [2026-04-30] Phase 15 cycle 15.6a — empty-locus `mean_rcn` and `summit_rcn` default = 1.0 (diploid baseline)

- Context:
  - Phase 15 cycle 15.6a SPEC15.10 introduced the new APS column schema where `_locus_metrics()` (`onionskin_core/aps.py:242-303`) emits `summit_rcn` + `mean_rcn` per locus replacing the legacy `summit_excess` + `mean_excess` + `peak_rcn` columns.
  - The empty-locus edge case (no bedGraph bins overlap the locus interval) needs an explicit return value for both new columns. Codex chose `summit_rcn=1.0` and `mean_rcn=1.0` in the cycle 15.6a Role 2 implementation (`aps.py:255-262`). Cycle 15.6a R3 re-audit verified the choice was implemented but flagged that the cycle's `[2026-04-30] Phase 15 cycle 15.6a SPEC15.11 morphology feature definitions` DECISIONS entry did not enumerate this specific choice, leaving it implicit.
  - `[SURPRISE:15:15.6a:1:A:6]` (filed during R1 audit) explicitly flagged this as a SPEC engineering decision that should be locked rather than left unilateral.

- Decision:
  - **Lock empty-locus `mean_rcn = 1.0` and `summit_rcn = 1.0`** (diploid baseline; RCN = 1.0 represents no amplification). This reflects what Codex implemented in cycle 15.6a Stage A and is consistent with the rest of the new schema's RCN-as-fold-over-baseline semantics.

- Rationale:
  - **Mathematically consistent.** `summit_rcn = 1.0` was Codex's choice for empty loci in the same return path; `mean_rcn = 1.0` matches it. Both represent "no signal observed at this locus → treat as baseline."
  - **Backward-compatible under floor-on aggregation (default).** Under `--aps-sample-aggregation-floor on` (default), the per-locus `mean_excess` derivation `max(mean_rcn - 1.0, 0.0) = 0.0` reproduces the legacy `mean_excess = 0.0` for empty loci byte-identically. No regression on legacy outputs.
  - **Honest under floor-off aggregation (the new path SPEC15.10 d6 introduced).** Under `--aps-sample-aggregation-floor off`, empty loci contribute `(1.0 - 1.0) = 0.0` to the per-locus signed sum → `APS_sum_of_means += 0.0` for empty loci. Empty loci contribute nothing, which matches the biological semantic ("no data, no contribution").
  - **The alternative (`mean_rcn = 0.0`)** would give empty loci `(0.0 - 1.0) = -1.0` per locus under floor-off → `APS_sum_of_means -= 1.0 × n_empty_loci`. Defensible (preserves legacy `mean_excess = 0.0` byte-parity even under floor-off via clamping) but biologically weirder — empty loci would actively reduce APS_sum_of_means under floor-off, which is not what the floor-off path's intent is.
  - Empty loci are rare in practice (sparse coverage manifests with chrom-specific bedGraph dropout) so the choice is mostly invisible on toy + production data, but the decision is real and should be honest.

- Scope:
  - Applies to Phase 15 cycle 15.6a SPEC15.10 schema and its downstream consumers (sample-level APS aggregation, post-sum-floor mode, reliability-filtered aggregate path that lands in cycle 15.6a-S1).
  - Does NOT alter the legacy `summit_excess`/`mean_excess`/`peak_rcn` empty-locus return path (those columns are retired post-cycle-15.6a).
  - Cycle 15.6a-S1 + SPEC15.20 (cycle 15.10a closeout sweep) inherit this lock; reliability-filtered aggregate work (Option A/B/C in cycle 15.6a-S1) consumes the same `mean_rcn = 1.0` empty-locus convention.

- Cross-references:
  - `[SURPRISE:15:15.6a:1:A:6]` in `multi-agent/plans/PHASE15_SURPRISE_LOG.md` — the surprise that flagged this as a needed lock; status now flips to Resolved (v0.14.84) per this DECISIONS entry.
  - `[2026-04-30] Phase 15 cycle 15.6a SPEC15.11 morphology feature definitions` (above in this file) — sibling cycle 15.6a Stage A schema lock; this entry completes the schema-decisions set for that cycle.
  - `onionskin_core/aps.py:255-262` — the live implementation.

- Authors:
  - John M. Urban, Claude Code 2.1.117 (claude-opus-4-7 ; Effort: Max)

---

## [2026-03-27] Smoothing: smooth_mean retained over rolling_median for summit/APS

- Context:
  - `refine_origin_for_call` used `smooth_mean` for profile smoothing before parabola fitting and argmax. Rolling_median was proposed as a replacement under the "primary-bin-cap" hires smoothing design.

- Decision:
  - Retain `smooth_mean` as the default. Rolling_median added as a future option via `primary_bin_size` parameter (currently unused).

- Rationale:
  - Testing showed rolling_median is neutral-to-worse: it produces step-like plateau artifacts that degrade parabola vertex fits. Benchmark results: ds2_1000bp ORC-Core dist degraded 1,340→1,868 bp; ds2_500bp L4 miss degraded 220→710 bp.

- Alternatives Considered:
  - `rolling_median` (scipy median_filter) — rejected; step artifacts bias parabola vertex position toward plateau edges

- Scope:
  - `refinement.py` `refine_origin_for_call`; multistage engine summit estimation
  - Raw (unsmoothed) RCN is still used for shape filter and twin-peak valley detection where sharp-feature sensitivity matters

- Notes:
  - Primary-bin-cap rolling_median design deferred to ROADMAP Priority 5.9.1 for potential future calibration; threshold re-tuning would be required

---

## [2026-03-27] Output directory naming: 01-prior / 02-posterior over prior / posterior

- Context:
  - Output directories were originally `prior/` and `posterior/`. Renamed in v0.5.09.

- Decision:
  - Use `01-prior/` and `02-posterior/` as canonical output directory names.

- Rationale:
  - Numeric prefixes make directories sort naturally in file browsers and make pipeline stage ordering explicit at a glance.

- Alternatives Considered:
  - `prior/` / `posterior/` — rejected; no natural sort order when mixed with other output files

- Scope:
  - All run modes (single-mode, multistage, posterior reruns). Affects `output_layout.py`, `onionskin.py`, `readme.py`, all tests.

---

## [2026-05-03] Phase 15 cycle 15.6a-S1 APS aggregation replaces `--aps-weight-loci`

- Context:
  - The cycle 15.6a-S1 orchestrator clarification on F5 replaced the old
    Option A/B/C framing with the new APS aggregation surface:
    `--aps-aggregation-mode`, `--aps-aggregation-score`,
    `--aps-aggregation-score-min`, and `--aps-emit-unfiltered-aggregates`.
  - The older `--aps-weight-loci` surface only weighted `area_excess` into
    side columns such as `weighted_area_excess` and `APS_area_weighted`, which
    no longer matches the locked cross-pipeline contract for canonical APS
    aggregation.

- Decision:
  - Retire `--aps-weight-loci` as a live user-facing APS control.
  - Canonical APS aggregation now flows only through the `--aps-aggregation-*`
    surface.
  - Score-aware weighting/filtering applies uniformly through the new
    aggregation path rather than through legacy side columns.

- Rationale:
  - The locked F5 contract requires one coherent aggregation surface across
    Growth, RMS, and HMM step-16, including weighting/filtering semantics,
    score-source selection, score floors, and optional unfiltered sidecar
    emission.
  - Keeping `--aps-weight-loci` alive in parallel would preserve an obsolete,
    narrower behavior that weights only one signal family and invites user
    confusion about which output is canonical.
  - The retirement also matches the schema cleanup: `weighted_area_excess` and
    `APS_area_weighted` are removed in favor of the aggregation-aware canonical
    APS outputs.

- Scope:
  - Applies to Phase 15 cycle 15.6a-S1 and later trees descended from this
    cycle.
  - Historical references to `--aps-weight-loci` in older CHANGELOG or audit
    history entries remain as provenance only.

- Cross-references:
  - `multi-agent/plans/PHASE15_AUDIT_LOG.md` cycle 15.6a-S1 F5/F6 material.
  - `multi-agent/full_instructions/CLUSTERING_DEFAULTS.md` Track 1 notes.

- Authors:
  - John M. Urban, GitHub Copilot (GPT-5.4 ; Reasoning: High)

---

## [2026-03-26] Shape filter BIC: default keeps k=1 for both models (likelihood-ratio mode)

- Context:
  - The shape filter compares flat vs triangle BIC. The flat model is truly k=0 (baseline externally fixed); the triangle model is k=1. Using k=1 for both (default) makes penalty terms cancel, giving a pure log-ratio of SSEs. Using k=0/k=1 (strict mode) correctly penalizes triangle model complexity.

- Decision:
  - Default: k=1 for both (penalty terms cancel → pure SSE ratio). Strict mode (`--shape-score-strict-bic on`): k=0 for flat, k=1 for triangle.

- Rationale:
  - Current threshold (50) was calibrated under the default mode. Strict mode lowers scores by ~log(n_bins) ≈ 6.0; at threshold 50 no calls change status on toy or DS2 real data. Strict mode is intended for future recalibration, not current production use.

- Alternatives Considered:
  - Always-strict — rejected; would require threshold re-calibration before production use and changes no calls at current threshold

- Scope:
  - `modeling.py` `_dBIC_flat_vs_tri`; `detection.py` `_shape_filter_calls` / `apply_shape_filter_tsv`; `single_engine.py` `run_single_stage`

---

## [2026-03-26] APS weights must be post-onset conditioned, not overall prevalence

- Context:
  - Priority 5.2 design: should per-locus APS weights penalize loci that are absent in many stages?

- Decision:
  - Weight components must be conditioned on post-onset behavior. A locus absent before onset is not a weak locus — it is a late-onset locus. Only absence *after* the estimated onset stage should reduce confidence.

- Rationale:
  - Penalizing overall prevalence would systematically down-weight real late-onset amplicons, distorting the APS ordering for samples near the boundary between amplification program stages.

- Alternatives Considered:
  - Overall detection rate across all stages — rejected; conflates "not yet started" with "unreliable"

- Scope:
  - ROADMAP Priority 5.2 design; future `aps.py` weighting implementation

- Notes:
  - Key formula: `best_onset_stage = argmax_s(mean_{t≥s}(p_{tℓ}) − mean_{t<s}(p_{tℓ}))`; `dip_rate` = fraction of stages after onset where evidence drops below threshold

---

## [2026-03-29] ONIONSKIN_FULL_HANDOFF.md is context; AGENT_CONVENTIONS.md is rules

- Context:
  - Prior to v0.5.25, ONIONSKIN_FULL_HANDOFF.md was described as the "canonical instruction source" for all agents.

- Decision:
  - ONIONSKIN_FULL_HANDOFF.md = canonical **context** document (architecture, biology, design decisions, gotchas). AGENT_CONVENTIONS.md = canonical **rules** document (editorial conventions, maintenance rules).

- Rationale:
  - Mixing context and rules in one document made it impossible to add process rules without cluttering the onboarding document. Separating them lets each document be updated independently.

- Alternatives Considered:
  - Keep rules in ONIONSKIN_FULL_HANDOFF.md — rejected; that document is already large and architecturally focused

- Scope:
  - All agent bootstrap files (CLAUDE.md, AGENTS.md, GEMINI.md, copilot-instructions.md) updated to reflect the distinction

---

## [2026-03-31] No commit IDs in CHANGELOG.md

- Context:
  - Proposal to embed git commit hashes in CHANGELOG entries for cross-referencing.

- Decision:
  - Do not include commit IDs in CHANGELOG.md. Use `git log` to cross-reference.

- Rationale:
  - To embed a commit ID, you'd need to know the hash before committing (impossible cleanly), or amend the commit afterward (creates a second commit or perpetually dirty working tree). The overhead outweighs the benefit given that `git log --oneline` with the version-tagged commit message already serves as the cross-reference.

- Alternatives Considered:
  - Commit then amend CHANGELOG with hash — rejected; creates a second commit or dirty state after every session
  - Post-commit hook to update CHANGELOG — rejected; adds tooling complexity for marginal benefit

- Scope:
  - CHANGELOG.md editorial convention; all agents

---

## [2026-03-31] PROJECT_STATE.md not added; blockers fold into TASK.md

- Context:
  - ChatGPT proposed PROJECT_STATE.md as a "current reality" bridge between ROADMAP and CHANGELOG. ROADMAP already has a "Current state snapshot" section.

- Decision:
  - Do not create PROJECT_STATE.md. Fold the "blockers" concept into TASK.md. ROADMAP snapshot section stays as-is.

- Rationale:
  - Two documents both claiming to represent "current reality" will inevitably diverge. The only unique value in PROJECT_STATE.md was a blockers section, which belongs in the tactical task queue (TASK.md) rather than a strategic document.

- Alternatives Considered:
  - Move ROADMAP snapshot into PROJECT_STATE.md — rejected; adds maintenance churn to an already-working infrastructure without clear benefit

- Scope:
  - `handoffs/project_context/` file inventory

---

## [2026-03-31] One GEMINI.md covers both Gemini CLI and Gemini Code Assist

- Context:
  - Two distinct Google agents are used: Gemini CLI (command-line tool) and Gemini Code Assist
    (VS Code extension, agent mode). Both auto-read `GEMINI.md` from the repo root.

- Decision:
  - One `GEMINI.md` file covers both agents. Authorship format differs per variant:
    `Gemini CLI <version> (<model-id>)` vs. `Gemini Code Assist (<model-id>)` or
    `Gemini Code Assist (model auto-selected)`.

- Rationale:
  - Both tools read the same file; maintaining two files would create convention drift risk.
    Agent-specific callouts (CLI hierarchy behavior, GAC agent mode requirement) live in a
    `## Gemini-specific notes` section within GEMINI.md. Authorship distinction is sufficient
    to tell them apart in CHANGELOG entries.

- Alternatives Considered:
  - Separate GEMINI_CLI.md and GEMINI_GAC.md — rejected; unnecessary duplication for tools
    that share the same project conventions

- Scope:
  - `GEMINI.md`; `multi-agent/AGENT_CONVENTIONS.md` authorship table; `multi-agent/AGENT_SWOT.md`

---

## [2026-03-31] MEMORY.md not added; pitfalls split between DECISIONS.md and ONIONSKIN_FULL_HANDOFF.md

- Context:
  - ChatGPT proposed a MEMORY.md file for "lessons, patterns, pitfalls."

- Decision:
  - Do not create MEMORY.md. Pitfalls involving design decisions go to DECISIONS.md. Stable codebase gotchas go to a `## Gotchas and known pitfalls` section in ONIONSKIN_FULL_HANDOFF.md.

- Rationale:
  - DECISIONS.md absorbs pitfalls-as-decisions naturally (e.g., rolling_median rejection). Stable gotchas belong in the onboarding document. A third document for overlapping content adds maintenance overhead without clear incremental value.

- Alternatives Considered:
  - MEMORY.md with Lessons / Patterns / Pitfalls sections — rejected; redundant with DECISIONS.md and ONIONSKIN_FULL_HANDOFF.md

- Scope:
  - `handoffs/project_context/` file inventory

---

## [2026-04-07] stage_median_within_calls/ bedGraphs — keep emitting permanently

- Context:
  - These bedGraphs (one per stage, restricted to called intervals) were questioned as potentially redundant with the full genome-wide stage median bedGraphs (genome_stage_medians/). The concern was that users would prefer full-genome bedGraphs for IGV inspection and that these restricted files add clutter.

- Decision:
  - Keep emitting `stage_median_within_calls/` bedGraphs permanently. Do not remove them.

- Rationale:
  - Three downstream algorithmic uses inside the pipeline itself — not just for visualization:
    1. `resolve_overlaps_bed` reads them to compute within-call valley depth for twin-peak detection (overlap resolution algorithm).
    2. `overlap_plots.write_overlap_diagnostic_plots` reads them to render within-call profile shapes.
    3. `timing.py` `_load_within_calls_track` reads them to locate the onset stage for timing curve computation.
  - Removing them would break overlap resolution and timing without a non-trivial refactor of those modules to read from `genome_stage_medians/` and crop in memory.
  - Labeled as "internal pipeline intermediate" in `00-INDEX.md` so users understand they are not for manual IGV inspection (use `genome_stage_medians/` for that).

- Alternatives Considered:
  - Remove and crop in memory during overlap resolution — possible but deferred; requires refactoring three separate modules and testing
  - Move to a hidden `.internal/` directory — rejected; complicates path handling; the INDEX label is sufficient

- Scope:
  - `onionskin_core/output_layout.py` (emission); `onionskin_core/timing.py`, `onionskin_core/overlap_plots.py`, `onionskin.py` (consumption)

---

## [2026-04-07] Output subdirectory numbering scheme — hierarchical pipeline-arm prefixes

- Historical note:
  - The `00-signal-files/` shared-foundation portion of this decision is superseded by the Phase 10 cleanup contract completed in `v0.10.24`.
  - `00-signal-files/` is now deprecated rather than active target architecture. The retained APS-derived track directories now live under `01-multistage-growth/05-aps/`, and HMM filtered-input copies are no longer emitted.

- Context:
  - Output subdirectories within `01-prior/` and `02-posterior/` sorted alphabetically with no indication of pipeline order or importance. Extension of the 2026-03-27 `01-prior`/`02-posterior` numbering decision.

- Decision:
  - Use two-digit zero-padded prefixes where `00-` means "shared foundation" and `01-`–`04-` are reserved for the four pipeline arms. Multistage-specific outputs now live inside a real `01-multistage-growth/` container, where the inner subdirectories are renumbered `01-`/`02-`/…/`07-`.
  - Final scheme:
    - `00-INDEX.md` — flat file guide (not a directory)
    - `00-signal-files/` — shared per-sample bedGraphs + genome stage medians
    - `01-multistage-growth/01-stage_median_within_calls/` — multistage internal intermediate
    - `01-multistage-growth/02-summary_bedgraphs/`
    - `01-multistage-growth/03-summits/`
    - `01-multistage-growth/04-summit_refinement/` — summit estimate TSVs/BEDs (replaces `others/`)
    - `01-multistage-growth/05-aps/`
    - `01-multistage-growth/06-plots/`
    - `01-multistage-growth/07-notebook/`
    - `02-per-stage-mean-shift/` — Phase E; eventual three-pipeline arm #2
    - `03-hmm/` — Phase 7 HMM pipeline arm
    - `04-unified-results/` — Phase 8+ (created then)
  - No generic `others/` catch-all. All outputs have explicit named homes.

- Rationale:
  - Historical note: `01a-`/`01b-` notation was the temporary bridge used before the grouped-layout refactor landed. That transitional flat layout has now been retired in favor of the real `01-multistage-growth/01-...07-` container while pipeline arm dirs `02-`/`03-`/`04-` stay stable at the outer level.
  - `00-signal-files/`: `00-` signals "shared foundation preceding all pipeline arms." Contains outputs shared across all three pipelines (per-sample bedGraphs, genome-wide stage medians). Mirrors the `00-INDEX.md` sentinel concept.
  - Eliminates `others/` permanently — no generic dump directory. All file types routed explicitly.

- Alternatives Considered:
  - Simple sequential numbering `01-`–`10-`: rejected; would require renumbering when three-pipeline structure lands
  - `01.01-`/`01.02-` dot notation: rejected; shell globbing awkward; letter suffixes `01a-` are cleaner
  - `00-signal-files/` as `01-signal-files/` (bumping pipeline arms to `02-`–`05-`): both work; `00-` chosen to emphasize shared-foundation status

- Scope:
  - `onionskin_core/output_layout.py` (primary); `onionskin.py`, `onionskin_core/timing.py`, `onionskin_core/aps.py`, `onionskin_core/readme.py` (path consumers)

---

## [2026-04-09] PuffStep protocols p16 and p17: will not be ported

- Context:
  - During Phase 8.3 signal utilities expansion, all 34 PuffStep normalization protocols
    were categorized and systematically ported. Two were identified as not worth porting.

- Decision:
  - **p16** (glocal median ratio norm): **will not be ported. Ever.**
  - **p17** (median norm → scale to target coverage level → ratio → re-median norm): **will not be ported. Ever.**

- Rationale:
  - p16: marked `INDEV` with `print("INDEV")` in PuffStep source (`normalize.py`). Never
    reached production status in PuffStep itself. No documented expected behaviour.
  - p17: an obscure multi-step pipeline that requires a `scale_data` primitive (scale all
    values to a target median coverage level, e.g. 1000). That primitive is not needed by
    any other protocol or workflow. The pipeline is not part of any standard replication
    analysis and has no clear biological motivation beyond p1 (median norm + ratio).

- Alternatives Considered:
  - Implementing `scale_data` as an additional primitive and composing p17 from it: rejected.
    Adds complexity with no identified use case; p17 adds little over p1 for the genomic
    replication context onionskin targets.

- Scope:
  - `onionskin_core/signal_utils.py` (not implemented there)
  - Recorded in ROADMAP.md (Phase 8.3), PUFFSTEP-INTEGRATION-PLAN.md (Phase 8.3 + open items),
    and this file. Do not revisit without a concrete use case.

---

## [2026-04-17] APS clustering feature and k-selection for posterior pipeline — investigation in progress

- Context:
  - The posterior pipeline re-runs the full growth model after APS clustering reorders/groups
    samples by developmental progression. Its purpose is to correct for sample mix-ups,
    within-stage heterogeneity, and ambiguous stage assignments — improving the growth track
    over the prior rather than just replicating it.
  - The original default (area + elbow k-selection) was found to produce biologically wrong
    posterior growth tracks: the elbow criterion selects k=2 on data spanning orders of
    magnitude in APS area, collapsing stages 1–N-1 into one group. The growth model then
    measures group2/group1 per bin, which is highest at the flanks (where early-stage forks
    have not arrived, so the denominator is near-background) rather than the summit. This
    produces an inverted M-shaped growth track — mathematically correct for k=2 but
    biologically wrong for developmental amplification data.
  - An investigation was run to identify better clustering features and k-selection strategies.

- Experiments run (2026-04-17):
  - Toy dataset (chr II, 5 stages, 3 reps/stage — synthetic, perfectly ordered):
    - 11 experiments covering all combinations of feature × k-selection
    - Features tested: area, shape, summit, width, log10area, log2area, log2summit, log2shape
    - k-selection: elbow (default), keep (= n_unique_stages), keep+no-singleton-guard
  - ds2+II (chr II, 5 stages, 2 reps/stage — real developmental amplification data,
    known biological heterogeneity and suspected stage-2/3 sample ambiguity):
    - Same 11 experiments (excluding redundant log2area)
    - Five-criterion evaluation: (1) monotonic growth, (2) amplicon detection, (3) summit
      quality at II-3, (4) no M-shape, (5) stage-1 pre-amplification status

- Toy data findings:

  | Experiment | k | Stage-5 splits | Track quality |
  |---|---|---|---|
  | area + elbow (baseline) | 2 | n/a (1-4 pooled) | M-shape (flanks 96.5 vs summit 14.4) |
  | shape + elbow | 2 | n/a (same) | M-shape |
  | area/shape/summit + keep + no-guard | 5 | YES — rep split | Flanks still highest |
  | log10area + elbow | 2 | n/a (1-3 pooled) | Not M-shaped, but wrong grouping |
  | log2area + elbow | 2 | same as log10area | Same |
  | log2shape + elbow | 2 | n/a (1-3 pooled) | Not M-shaped, but wrong grouping |
  | width + keep | 5 | No | Identical to prior (correct) |
  | log10area + keep + no-guard | 5 | No | Identical to prior (correct) |
  | log2area + keep + no-guard | 5 | No | Identical to prior (correct) |
  | log2summit + keep + no-guard | 5 | No | Identical to prior (correct) |
  | log2shape + keep + no-guard | 5 | No | Identical to prior (correct) |

  Key toy-data conclusions:
  - The elbow criterion is structurally broken for area-scale APS data: one dominant Ward gap
    drives k=2 regardless of feature transformation. Even log-compressed features pick k=2.
  - Raw area/shape/summit + keep creates singletons within stage 5 (replicates of the same
    biological stage split across posterior groups) due to large absolute within-stage variance.
  - Log transformation (log10area, log2summit, log2shape) combined with keep eliminates the
    singleton problem: within-stage variance becomes negligible in log space.
  - width + keep also avoids singletons (width grows more linearly with development).
  - On toy data (where stages are perfectly ordered), all `keep` + log-transform methods
    produce posterior = prior, confirming correct behavior when no reordering is needed.

- ds2+II findings:

  | Experiment | k | Regressions | Calls vs prior | M-shape | Notes |
  |---|---|---|---|---|---|
  | PRIOR (reference) | 5 | 6 | 8 (ref) | None | Stage-3 regressions across most amplicons |
  | area + elbow | 2 | — | 8 | Likely | Top bins outside known high-conf amplicons |
  | area + keep + no-guard | 5 | many splits | 9 | None | Stages 2,3,4 all inter-mixed |
  | shape + elbow | 2 | — | 8 | None | Top bins in II-6, not II-3/II-4 |
  | shape + keep + no-guard | 5 | 6 | 9 | None | Fixed II-1/II-2.1/II-3 regressions; added II-6/II-7 regressions |
  | summit + keep + no-guard | 5 | many | 8 | None | Stages 2,3,4 inter-mixed |
  | width + keep | 4 | many | 8 | None | k collapsed; worst k≥4 result |
  | log10area + elbow | 3 | 0 | 6 | None | All 6 amplicons monotonic; lost II.7 and II-5 |
  | log10area + keep + no-guard | 5 | 6 | 9 | None | Identical to shape+keep |
  | log2summit + keep + no-guard | 5 | 4 | 7 | None | Lost II-5; only method recovering II-4 in top-3 |
  | log2shape + keep + no-guard | 5 | 6 | 9 | None | Identical to shape+keep and log10area+keep |

  Five-criterion detail for top candidates (ds2+II chr II):

  Criterion 1 — Prior regressions (stage-2/3 issue confirmed):
  - Prior has stage-3 < stage-2 across II-1, II-2, II-3, II-4 and additional stage-4
    regressions at II.7, II-4, II-5. Confirms that stage-2 and stage-3 samples are
    ambiguously ordered in the prior for this dataset.
  - shape+keep / log10area+keep: fixed the three most important regressions (II-1, II-2.1,
    II-3) by merging prior stages 2+3 into a single posterior group. However, introduced
    new regressions in II-6 and II-7 (which were clean in the prior). Net count: 6 → 6.
  - log10area+elbow (k=3): eliminated all regressions — 6 → 0. All detected amplicons
    monotonically increasing across 3 posterior stages. Best on this criterion.
  - log2summit+keep: 6 → 4 regressions. Partial improvement.

  Criterion 2 — Amplicon detection:
  - shape+keep / log10area+keep: 9 calls — correctly split II-2 into II-2.1 and II-2.2
    (both are real separately confirmed amplicons). All prior calls preserved. Best.
  - log10area+elbow: 6 calls — lost II.7 (low confidence) and II-5. Sensitivity cost of k=3.
  - log2summit+keep: 7 calls — lost II-5.

  Criterion 3 — Summit quality (II-3, best-characterized amplicon):
  - All methods place the posterior summit within the II-3 amplicon boundary. No method
    displaces the summit. Prior peak: 33380000; posterior peaks: 33335000–33470000. ✓

  Criterion 4 — No intra-amplicon M-shape (revised):
  - The original check compared summit zone vs external flanks (±300 kb outside the amplicon),
    which missed the subtler intra-amplicon pattern the user reported: a "telephone poles with
    wire" shape where the shoulders WITHIN the amplicon are higher than the summit zone itself.
    A more precise check was performed by extracting the full 750-bin profile across II-3
    (chr II, 33100k–33850k, 5 kb bins) and identifying the global maximum within the amplicon
    vs the summit zone maximum.

  Reference: prior peak at II-3 = 9.96 at 33370k (summit zone = 33330k–33420k).

  | Method              | Summit zone peak        | Amplicon max            | Assessment               |
  |---------------------|------------------------|------------------------|--------------------------|
  | PRIOR               | 9.96 at 33370k          | 9.96 at 33370k          | Centered ✓               |
  | area + elbow        | 4.1–5.2 (flat)          | 6.85 at 33605k (shoulder)| M-shape confirmed ✗     |
  | shape + keep        | 21.68 at 33365k         | 26.83 at 33570k         | Right-shifted ~200kb ⚠   |
  | log10area + keep    | 21.68 at 33365k         | 26.83 at 33570k         | Right-shifted ~200kb ⚠   |
  | log2shape + keep    | 21.68 at 33365k         | 26.83 at 33570k         | Right-shifted ~200kb ⚠   |
  | log10area + elbow   | 9.77 at 33420k          | 11.45 at 33500k         | Mildly right-shifted ⚠   |
  | log2summit + keep   | 14.61 at 33385k         | 15.50 at 33620k         | Near-centered ✓          |

  Summary:
  - area+elbow: confirmed intra-amplicon M-shape. Summit zone (4.1–5.2) is suppressed below
    both shoulders (left 6.75 at 33225k, right 6.85 at 33605k). This is the telephone-pole
    pattern on real data, not just toy data.
  - shape+keep / log10area+keep / log2shape+keep: identical results on this dataset. The summit
    zone does have high values (max 21.68 at 33365k, close to the summit), but the global
    amplicon maximum is ~200 kb right of the summit (26.83 at 33570k). Not a classic M-shape
    but the summit is not the global maximum within the amplicon.
  - log10area+elbow: picks k=3, mildly right-shifted peak (33500k, 120 kb from summit).
  - log2summit+keep: summit zone max at 33385k (15 kb from summit), with right shoulder only
    slightly higher (15.50 at 33620k, 250 kb away). Most summit-faithful of all methods tested.

  Criterion 5 — Stage-1 pre-amplification:
  - All methods keep both stage-1 replicates (SAG41 APS=1.15M, SAG40 APS=1.76M) in posterior
    stage 1. The 1.5× difference between them does not cause separation. One rep is likely
    near pre-amplification; the other already shows early amplification signal. No method
    tested separates them into different posterior stages.

  Additional observations:
  - shape+keep, log10area+keep, and log2shape+keep produce identical cluster maps and
    identical posterior tracks on ds2+II chr II. These three features are equivalent for
    this dataset.
  - width+keep collapsed to k=4 on real data (worse than k=5), with stage4/5 merged —
    contrary to toy data where it produced a clean k=5. Width is not a reliable feature
    for real data with genuine stage overlap.
  - log10area+elbow picks k=3 on real data (not k=2 as on toy data). The elbow is not
    completely inert on real data — it responds to the more complex gap structure. k=3
    achieves the best monotonicity but at sensitivity cost.
  - The "splits" in k=5 keep+no-guard results are not noise: they reflect genuine biological
    ambiguity in ds2. Stages 2 and 3 are ambiguously ordered in APS space, consistent with
    the user's observation that their averaged prior signal shows stage-3 < stage-2
    regressions across most amplicons. The posterior correctly identifies this and merges
    them into the same group.

  ds1+II experiment table (9 stages × 2 reps/stage = 18 samples; chr II only):

  Dataset context: ds1 is a single-animal dataset with very high APS disorder. The prior
  has 8 regressions in 3 amplicons. Stages 5–9 are all crowded into 3.09M–5.57M APS range
  with no monotonic ordering. Stage 2 is split 10×: H52=298K vs H72=3.09M. Prior detects
  only 3 chr II amplicons (II-1, II.7 overlap, II-6). II-2 and II-3 are not in the prior.

  | Experiment        |  k | amps |  prior lost | post_regs | s1_n | s1_min_APS | II-3 summit |
  |-------------------|----|------|-------------|-----------|------|------------|-------------|
  | PRIOR             |  9 |  3   | —           |  8        | 2    | 100016     | not detected|
  | baseline          |  3 |  3   | II-6        |  0        | 7    | 100016     | 33460kb     |
  | area+keep         |  9 |  3   | II-6        |  4        | 3    | 100016     | 33461kb     |
  | shape+elbow       |  2 |  2   | II-6+II-2   |  0        | 7    | 100016     | N/A         |
  | shape+keep        |  9 |  3   | II-6        |  4        | 3    | 100016     | 33461kb     |
  | summit+keep       |  9 |  4   | none        |  5        | 3    | 100016     | 33461kb     |
  | width+keep        |  9 |  4   | none        |  5        | 3    | 100016     | 33461kb     |
  | log10area+elbow   |  2 |  5*  | none        |  0        | 4    | 100016     | 33460kb     |
  | log10area+keep    |  9 |  4   | none        |  5        | 1    | 100016     | 33456kb     |
  | log2summit+keep   |  9 |  4   | none        |  6        | 1    | 100016     | 33454kb     |
  | log2shape+keep    |  9 |  4   | none        |  6        | 3    | 100016     | 33461kb     |
  | log2shape+elbow   |  2 |  5*  | none        |  0        | 4    | 100016     | 33460kb     |
  * 5 amps includes a new detection at ~41.6M (II-4 region), not present in prior.

  Interpretation notes for ds1+II:
  - Elbow methods (k=2 on ds1) catastrophically collapse 9 stages into 2 groups. Even though
    they find more amplicons (5 vs 4) and show 0 regressions, they destroy all developmental
    granularity. Not viable despite better detection surface.
  - baseline (k=3) drops II-6, gains II-2 and II-3 (same count of 3 amps, different set).
    Dropping a known prior amplicon is disqualifying.
  - area+keep, shape+keep: also drop II-6. Disqualifying on detection criterion.
  - summit+keep, width+keep: detect all 4 expected amps, 5 regressions. Viable candidates
    but 5 regressions is high (though prior had 8, so monotonicity improved overall).
  - log10area+keep: detect 4 amps, 5 regressions, only 1 sample in stage 1 (S38; most pure
    pre-amplification isolation). Regression count is same as summit/width+keep.
  - log2summit+keep: detect 4 amps, 6 regressions, only 1 sample in stage 1 (most pure).
    Highest regression count among keep methods. Summit most faithful on ds2 but more regs
    on ds1 — tradeoff.
  - log2shape+keep: detect 4 amps, 6 regressions, 3 stage-1 samples. Same regression count
    as log2summit+keep but groups S38+H71+H52 into stage 1 (broader, less pure but more
    statistically stable).
  - II-3 summit is very consistent (33454–33461kb) across all methods that detect it.
    Summit location is not a differentiator for ds1.
  - M-shape analysis for ds1+II is deferred pending full-genome experiments.
  - Key contrast with ds2: on ds2, log10area+keep/log2shape+keep/shape+keep produced
    identical results. On ds1 they diverge — log10area+keep gives singleton stage-1 (S38
    alone at 100K APS) while log2shape+keep groups S38+H71+H52 together. This is because
    in log10 space, 100K vs 194K is a larger gap (Δ=0.29 log10) than in shape space where
    all three have similar early-amplification profiles.

### ds2 + 4-chromosome (X, II, III, IV) experiment — full-genome APS scope

Run on the full 4-chromosome manifest (`dev/datasets/full_genome/batch/manifest.batch-5kb.fofn
--chromosomes X,II,III,IV`). APS is computed across all loci on all 4 chromosomes, so the APS
range is wider (46 amplicons vs 7–8 on chr II alone). All 11 experiments run in
`dev/runs/aps_clust_ds2_4chr/`. Reproducible via `make clust-ds2-4chr`, `make clust-report-ds2-4chr`.

**Chr reliability ordering** (critical context for interpreting results):
`II > III > X > IV`
- Chr II: clearest, best-characterized, strongest signal — most reliable baseline
- Chr III: reliable secondary chromosome
- Chr X: noisier due to X/X' animal heterogeneity in this dataset
- Chr IV: substantial marginal amplicons with uncertain truth — false positives likely among the 20 prior calls

**The key discriminator: chr IV**
Chr IV has the widest APS range and the largest number of prior calls (20), many of which are
marginal. This makes it the hardest test for the clustering step. Feature methods that over-separate
mid-range samples (log10area, log2summit, log2shape) fail catastrophically here.

**Chr IV table (20 prior amps, 10 prior regs):**

| Experiment | k | amps | dropped | post_regs | s1_n |
|---|---|---|---|---|---|
| baseline/elbow | 2 | 18 | 2 | 0 | 8 |
| summit + keep | 5 | 15 | 5 | 0 | 2 |
| shape + keep | 5 | 13 | 7 | 4 | 4 |
| area + keep | 5 | 13 | 7 | 2 | 2 |
| log10area + elbow | 3 | 12 | 8 | 0 | 5 |
| log2shape + elbow | 3 | 12 | 8 | 0 | 5 |
| log10area + keep | 5 | **6** | **14** | 1 | 2 |
| log2summit + keep | 5 | 6 | 14 | 1 | 2 |
| log2shape + keep | 5 | 6 | 14 | 1 | 2 |
| width + keep | 5 | 6 | 14 | 1 | 2 |

**Chr II table (8 prior amps, 1 prior reg) — 4chr APS scope:**

| Experiment | k | amps | dropped | post_regs |
|---|---|---|---|---|
| baseline/elbow | 2 | 9 | 0 | 0 |
| summit + keep | 5 | 7 | 1 | 1 |
| shape + keep | 5 | 7 | 1 | 3 |
| area + keep | 5 | 6 | 2 | 3 |
| log10area + elbow | 3 | 6 | 2 | 0 |
| log10area + keep | 5 | **4** | **4** | 2 |

**Chr X table (10 prior amps, 4 prior regs):**

| Experiment | k | amps | dropped | post_regs |
|---|---|---|---|---|
| summit + keep | 5 | 8 | 2 | 1 |
| log10area + keep | 5 | 6 | 4 | 1 |

**Chr III table (8 prior amps, 4 prior regs):**

| Experiment | k | amps | dropped | post_regs |
|---|---|---|---|---|
| summit + keep | 5 | 8 | 1 | 0 |
| log10area + keep | 5 | 4 | 4 | 3 |

**Root cause of log10area+keep failure (the singleton problem):**

In 4-chromosome APS space, SAG47 (prior stage 4 rep2, APS=8.9M area) is an outlier.
In log10 space:
- SAG47: log10(8.9M) ≈ 6.95
- SAG45 (stage 3 rep2): log10(9.7M) ≈ 6.99 — only 0.04 log10 units away
- SAG43 (stage 2 rep2): log10(10.8M) ≈ 7.03
- SAG44 (stage 3 rep1): log10(13.5M) ≈ 7.13
- SAG42 (stage 2 rep1): log10(14.9M) ≈ 7.17
- SAG46 (stage 4 rep1): log10(20.0M) ≈ 7.30

Ward linkage on these 10 values creates a large gap between SAG47 (6.95) and the next point
(SAG45 at 6.99). Surprise: with k=keep=5, SAG47 ends up alone in posterior stage 2 — a
**singleton stage**. A single-replicate stage median = SAG47's raw RCN values. Marginal chr IV
amplicons that fall below the BIC detection threshold on a noisy single-replicate stage fail to
be called. This drops 13–14 of 20 chr IV amplicons.

Confirmed from cluster map comparison:
- log10area+keep cluster map: SAG47 alone at cluster_raw=5 → posterior stage 2 (singleton)
- summit+keep cluster map: SAG47 grouped with SAG45, SAG43 at cluster_raw=4 → posterior stage 2 (3 samples)

**Why summit_keep avoids the singleton:**
In summit space, SAG47's per-locus peak RCN values cluster naturally with other early-amplification
samples (SAG45, SAG43) because they have similar spatial profiles at this development stage. The
singleton does not form. SAG47 ends up at posterior stage 2 with 3 samples — a more stable stage
median.

**Why chr-II-only tests were insufficient to catch this:**
On chr II alone, APS is dominated by the large ~II-3 amplicon. The spread in area values is less
extreme. In the chr-II-only log10area+keep test, log10area+keep appeared tied with summit+keep
as a "best candidate." The 4-chr test exposed the fragility — when the APS range widens (more
chromosomes, more amplicons, more varied sizes), log10 over-separates mid-range samples.

**Revised recommendation and conclusion:**

**summit+keep is the correct new default.** It is the only method that:
1. Avoids singleton stages across both narrow (chr II only) and wide (4-chr) APS ranges
2. Retains the most amplicons on the most challenging chromosome (chr IV: 15 vs 6 for log10area+keep)
3. Reduces regressions on chr II and III (1 and 0 vs baseline's 0 but k=2 overcounting issue)
4. Keeps 2 samples in stage 1 (correct pre-amp isolation on ds2 where stage 1 should be the two lowest-APS samples)

**Previous conclusion retracted:** The earlier assessment that `log10area+keep` and `summit+keep`
were joint best candidates was based only on chr-II single-chromosome data. The 4-chr test
definitively broke `log10area+keep`.

**CLI defaults changed to:**
- `--aps-feature summit` (was `area`)
- `--aps-cluster-k keep` (was `elbow`)
- Singleton guard auto-disabled for `keep` mode (guard only meaningful for elbow/N-cluster methods)

- Status: DECISION REACHED (v0.10.37) — APS defaults changed to summit+keep. Investigation
  complete. Future experiments (chr-specific APS bootstrapping, guard-ON for log10area tests,
  RCN floor removal) are recorded in BRAINSTORM.md as optional follow-ups.

- Scope:
  - `onionskin_core/aps.py` — new features: log10area, log2area, log2summit, log2shape;
    log transform in build_aps_shape_matrix; updated _FEATURE_COL
  - `onionskin.py` — `--aps-feature` default changed to `summit`; `--aps-cluster-k` default
    changed to `keep`; `_aps_fix_singletons()` helper disables guard automatically for keep mode;
    `_run_posterior_qc()` implements 5 automated posterior QC checks; `--aps-feature` and
    `--aps-cluster-k` help text updated to reflect new defaults and reasoning
  - `scripts/aps_cluster_experiments.py` — runs all 11 APS clustering experiments for a
    given dataset+chromosome
  - `scripts/aps_cluster_report.py` — generates comparison table from experiment runs
  - `Makefile` — added clust-ds1-II, clust-ds2-II, clust-report-ds1-II, clust-report-ds2-II,
    clust-ds2-4chr, clust-report-ds2-4chr

## [2026-04-15] Bin size detection must use mode-of-sizes, not first-bin heuristic

- **Decision:** Always use the statistical mode of observed bin sizes (across the first
  N rows of the bedGraph) as the canonical bin size estimate. Never rely solely on the
  first bin.

- **Reason:** Early development revealed that some input bedGraphs have a non-representative
  first bin — the first record in the file had a different size from all remaining bins.
  Using the first bin alone produced an incorrect bin size estimate for those files. The
  mode-of-sizes approach is robust against this edge case: a single anomalous first bin is
  outvoted by the dominant size across the rest of the file.

- **Canonical implementation:** `onionskin_core/refinement.py::detect_bin_size(path_or_df)`.
  For a DataFrame input: compute mode of `(end - start)` across the first `max_lines` rows.
  For a file-path input: delegate to `onionskin_core.binning.detect_major_bin_size()`, which
  uses the same mode approach.

- **Do not regress to first-bin heuristic** anywhere in the codebase.


## [2026-04-15] Keep `onionskin_core/dead_code.py` as a PuffStep code reservoir

- **Decision:** `onionskin_core/dead_code.py` must not be deleted. It is an intentional
  archive of PuffStep functions that are not yet active in the codebase but are preserved
  so that future port work does not require reconstructing them from the original repository.

- **Reason:** The original agreement (Phase 3) was to retain PuffStep code in a reservoir
  rather than discard it outright. The file currently holds `write_summit_beds`, which
  captures a coordinate convention (`[origin, origin+1]`) and per-label summit output
  logic that diverges from the active `_write_summit_bed()`. Keeping it means the option
  to unify or extend summit BED output remains open without re-deriving the PuffStep approach.

- **Policy:** Do not delete the file. Do not remove functions from it silently. The module
  docstring must describe the file as a PuffStep reservoir and must not suggest deletion.
  If the reservoir is genuinely no longer needed, the decision to remove it should be
  made explicitly and recorded here.

---

## [2026-04-23] "ms" is overloaded in this codebase — avoid in new flag names

- Context:
  - During Phase 14 Supplemental Q9 (14-S4), we were naming a new flag for the
    pass-2 mean-shift scan halfwidth. Candidate names included variants with "ms"
    (e.g., `--growth-ms-halfwidth`).
  - "ms" is already used as shorthand for BOTH "mean-shift" (the algorithm) AND
    "multi-stage" (a property of the input data, also historically used as the pipeline
    name before Phase 11 clarified it) in the codebase.

- Decision:
  - Do not introduce new flag names containing "ms". Use explicit, non-overloaded
    words: "scan" (for pass-2 scanning-window semantics), "mean-shift" when truly
    necessary.
  - Chosen name in 14-S4: `--growth-scan-halfwidth` / `--rms-scan-halfwidth`.
  - "multistage" as a term remains valid only when referring to the input-data property,
    NEVER as a pipeline name (Phase 11 decision).

- Rationale:
  - Single-character / two-character abbreviations with two legitimate expansions in
    the same repo are a durable source of reader confusion. Avoiding them up front is
    cheaper than documenting the ambiguity later.

---

## [2026-04-24] RMS shape-score strict-BIC wired now; HMM shape-score deferred to Phase 15

- Context:
  - Phase 14 Supplemental 14-S26 revisited the RMS and HMM shape-score parser surfaces.
  - RMS already had mature shape-filter sinks (`shape_filter_calls()` /
    `apply_shape_filter_tsv()`) and live runtime wiring for `--rms-shape-score-threshold`
    and `--rms-shape-score-strict-bic`.
  - HMM has placeholder flags but no equivalent state-path-aware shape-score sink yet.

- Decision:
  - Keep RMS shape-score override flags wired in Phase 14 Supplemental and update their
    help/tests to reflect that they are live.
  - Keep HMM shape-score flags as placeholders and explicitly defer their wiring to Phase
    15 HMM completeness work.

- Rationale:
  - RMS shape filtering is a direct extension of already-mature RMS/growth BIC shape
    logic.
  - HMM shape filtering needs design work that accounts for state-path growth as a
    credibility signal and cross-stage amplicon-shape meta-analysis, so it belongs with
    Phase 15 HMM development.

---

## [2026-04-24] Standalone engine CLIs removed as maintained user surfaces

- Context:
  - Phase 14 Supplemental 14-S10 revisited why `growth_model_engine.py` and
    `rcn_mean_shift_engine.py` still exposed separate argparse CLIs when all supported
    user access flows through `onionskin.py`.
  - Keeping engine-level help surfaces in parity with the top-level CLI repeatedly added
    maintenance burden without a clear user or development need.

- Decision:
  - `onionskin.py` is the sole maintained user-facing CLI for growth and RMS pipelines.
  - The growth engine keeps a private `_run_argv()` parser only as the internal
    `run_multistage(argv)` implementation detail used by `onionskin.py`.
  - The RMS engine no longer exposes a standalone `main()` or module `__main__` path.

- Rationale:
  - User direction from Q11: "Let's move forward with simplifying onionskin here since all
    pipelines are accessed from onionskin.py. I do not foresee directly using the engines,
    especially if they are not maintained in parallel, which has proven burdensome."
  - A single public parser avoids stale crosswalk/help tests and keeps Phase 14
    Supplemental parser terminology work concentrated in one CLI surface.

---

## [2026-04-23] Phase 14 Supplemental column-rename band-aid (14-S28): accept mixed semantics, defer structural fix

- Context:
  - Phase 14 Supplemental 14-S28 renamed `_origins.tsv` output columns to use "summit"
    terminology consistent with the rest of the Phase 14 public vocabulary.
  - Investigation during Round 9 (Q39/JQ3) surfaced that the current
    `origin_ci_low/high_bp` columns have MIXED semantics — range across per-stage
    parabola vertices when parabola wins, bootstrap CI on argmax_mean when argmax
    fallback wins. `origin_boot_sd_bp` is always argmax-based regardless of winner.
    We do not bootstrap the parabola today.
  - Three options considered: (A) preserve mixed semantics in rename, (B) rename with
    honest labels (no "ci" or "bootstrap" where they would be inaccurate), (C) do the
    structural fix — implement parabola-specific bootstrap — in Phase 14 Supplemental
    itself.

- Decision:
  - Option B: band-aid with honest labels.
    - `origin_ci_low/high/width_bp` → `final_summit_low/high/width_bp` (dropped "ci"
      because "ci" would be inaccurate for parabola-wins rows).
    - `origin_boot_sd_bp` → `argmax_mean_bootstrap_sd_bp` (honest: the SD is always
      argmax-based, not the SD of `final_summit_bp` when parabola wins).
  - Add a parser-group description block to the new Summit Refinement group (14-S27)
    that makes the mixed semantics visible from `onionskin -h` directly.
  - Defer the structural fix to a dedicated future phase — the Summit
    Methodology cluster — parked at
    `multi-agent/plans/next/SUMMIT_SOUP.md` (formerly the Phase 16
    brainstorm at the time of decision; renamed during the
    DEVPLAN-ROADMAP-AND-NEXT-DIRECTORY-SYSTEM cycle to drop the
    arbitrary phase-number filename in favor of a theme-only SOUP name).

- Rationale:
  - Doing the structural fix (parabola bootstrap) in isolation is half a solution. The
    full fix is tangled with: dynamic origin-detection window (IBM-C5); RMS summit
    pre-smoothing investigation (Q36); HMM `peak_rcn_stage` + sliding-offset
    (IBM-C4); cross-pipeline summit estimator audit across 5 known estimator families
    (argmax, parabola, sliding-offset, RMS selector policies, HMM state-path).
  - Phase 15 is HMM-centric and already bloating; forcing summit-methodology work
    into it would blur its scope.
  - Band-aid with honest labels + documentation is proportional to the problem for now.
    The dedicated phase is the right home for the structural fix.

- Scope boundary for this decision:
  - Internal `onionskin_core/` function names (`refine_origin_for_call`,
    `refine_origin_sliding_offset`, etc.) remain "origin" per the 14.2 internal-boundary
    rule. Only public output columns, flag names, and help strings standardize on
    "summit."

---

## [2026-04-23] Summit Methodology as a dedicated future phase (originally framed as "Phase 16")

- Context:
  - Multiple deferrals across Phase 14 Supplemental Rounds 4–9 accumulated a coherent
    summit-estimator / summit-refinement bundle that spans pipelines, CLI, output
    schema, and algorithmic design.
  - Solving any single piece in isolation (e.g., parabola bootstrap in 14-S28) leaves
    the other pieces inconsistent with it.

- Decision:
  - Park the entire bundle in `multi-agent/plans/next/SUMMIT_SOUP.md`
    (formerly the Phase 16 brainstorm at the time of decision; renamed
    during the DEVPLAN-ROADMAP-AND-NEXT-DIRECTORY-SYSTEM cycle to drop
    the arbitrary phase-number filename in favor of a theme-only SOUP
    name). Six items:
    (1) parabola bootstrap; (2) dynamic origin-detection window; (3) RMS summit
    pre-smoothing investigation; (4) HMM `peak_rcn_stage` + sliding-offset; (5)
    cross-pipeline summit estimator audit; (6) optional `_origins.tsv` schema
    rationalization.
  - Phase order: after Phase 15 (HMM completeness), before any later analytical phase.
  - The prior `PHASE16_BRAINSTORM.md` scaffold (per-stage completeness) was confirmed
    obsolete — its scope was consumed by Phase 13 — and was archived as
    `multi-agent/plans/archived/20260423-PERSTAGE_BRAINSTORM.md`. The new Summit
    Methodology brainstorm replaces it at that filename.

- Rationale:
  - Dedicated phase lets the spec-engineering loop treat the bundle coherently rather
    than piecemeal.
  - Phase 15 stays HMM-centric.
  - Phase 14 Supplemental stays CLI-centric (with the band-aid column rename for the
    immediate naming consistency win).

---

## [2026-04-28] APS feature mode candidates — accepted/rejected catalog + RCN-vs-excess design decision

- Context:
  - Phase 15 transfer-stage Q13 follow-up (in `multi-agent/plans/PHASE15_FEEDBACK.md` Round 1
    follow-up section) evaluated candidate APS clustering feature modes. APS clustering
    operates on samples (not amplicons) — its purpose is posterior sample grouping /
    posterior manifests / posterior analysis. Features must therefore be quantities that
    vary across samples for the same amplicon. The user requested a catalog of which
    candidate features were merged into the new design (BRAIN15.27 — Composite multi-feature
    APS clustering modes with three-layer PCA composition) and which were rejected with
    rationale, so future phases don't re-investigate already-rejected ideas.
  - Investigation also surfaced that the existing `summit_excess` / `mean_excess` / floored-
    `area_excess` framing — introduced by an agent during earlier development — drops
    information vs using raw RCN values directly. The user requested migration back to raw
    RCN as the default while preserving the floor semantics as an opt-in for sample-level
    APS aggregation reasoning. That migration is captured in BRAIN15.28
    (`summit_excess` → `summit_rcn` rename + RCN-as-default migration).

- Decision:
  - **Accepted candidate features** (folded into BRAIN15.27's `metrics` mode + Layer-1 + Layer-2 PCA components):
    - Per-amplicon: `summit_rcn` (raw, post-rename per BRAIN15.28), `width_above_threshold_bp`, `log10area`, `asymmetry_ratio`, `shape_score_raw` (= `dBIC_flat_vs_tri`).
    - Per-sample scalars: sample-level summit-APS, sample-level width-APS, sample-level log10-area-APS.
    - Layer-1 amplicon-level shape PCA scores (per-amplicon, X PCs each, X controlled by `--aps-num-amp-pc`).
    - Layer-2 sample-level shape PCA scores (X scalars per sample, X controlled by `--aps-num-sample-pc`; two computation variants — 2a `_raw` and 2b `_reduced`).
    - Layer-3 matrix-level PCA pre-projection (`--aps-pca-matrix` toggle, applies after any `--aps-feature` mode).
  - **Rejected candidate features:**
    - **Fourier descriptors of the RCN profile** — designed for periodic / closed contours (image-processing of closed shapes); RCN profiles are linear, asymmetric, single-peaked. Structural mismatch — there's no rotation invariance to exploit, no closed boundary to traverse. Trying to use Fourier here is solving a different problem than ours, with all the interpretability cost and none of the benefits the technique was designed for.
    - **Curvature features (second derivative of the RCN profile)** — second derivatives amplify noise; even smoothed profiles produce flaky curvature outside the immediate summit neighborhood. Redundant with what `--aps-feature shape` and the existing parabola-fit curvature coefficient `A` (already used per-summit; mentioned at [onionskin_core/notebooks.py:1859](onionskin_core/notebooks.py#L1859)) already capture. Adding a "curvature features" plural channel would add noise without adding information.
    - **Multi-family BIC vectors as a sample-clustering feature** — for amplicon classification (amplicon vs collapsed-repeat vs odd region), this is exactly what `dBIC_flat_vs_tri` already provides and what BRAIN15.18's multistage unification work expands on (the 4-mode classification flag + state-path evolution checks). For sample clustering specifically, multi-family BIC vectors are unusual — BIC measures model-fit quality (a per-call quantity), and the overlap with BRAIN15.18 means the most useful application is already in scope. The single existing `dBIC_flat_vs_tri` value (= `shape_score_raw`) IS sample-resolving and IS included in BRAIN15.27's feature vector — that's where the BIC information enters APS clustering.
    - **`peak_rcn` as a distinct feature alongside `summit_rcn`** — `peak_rcn` and `summit_rcn` are the same quantity post-rename (BRAIN15.28). Including the legacy `summit_excess = max(peak_rcn - 1.0, 0.0)` as a feature alongside `summit_rcn` adds dimension without adding information (offset by 1, optionally floored). With the floor: actively destroys information by collapsing all below-background samples to 0. Without the floor: arbitrary translation by a constant.
  - **RCN-vs-excess design decision** (BRAIN15.28 captures the implementation work):
    - **Migrate APS feature computation back to raw RCN values as the default.** Rename `peak_rcn` → `summit_rcn`. Deprecate `summit_excess` and `mean_excess` as per-locus columns; the excess+floor semantics are derived at sample-level aggregation time when the user opts into the `--aps-sample-aggregation-floor` (or similar) flag.
    - **Behavior change for `--aps-feature summit`:** under the rename, `--aps-feature summit` reads raw `summit_rcn` instead of floored `summit_excess`. Flag in CHANGELOG when it lands.
    - **Area is handled separately by BRAIN15.14** (`--aps-area-excess-floor` analytical testing) — this DECISIONS entry does not touch area; whichever form (excess-floored vs RCN-summed vs post-sum-floor) wins the BRAIN15.14 testing is what BRAIN15.27's `log10area` consumes.

- Rationale:
  - The "excess" framing without the floor is a translation by a constant (`peak_rcn - 1.0`), which is invariant under any clustering metric that's translation-invariant and silently shifts the reference frame for any metric that isn't. With the floor (`max(peak_rcn - 1.0, 0.0)`) it actively destroys information — every below-background sample at a locus gets the same value (0), regardless of how far below background the actual signal was. For sample clustering, this collapses developmental-stage variation in the lower-RCN region.
  - The user's reasoning (verbatim, Phase 15 transfer-stage chat 2026-04-28): *"At one point, an agent decided to go the 'excess' route. I questioned why RCN was not enough as is (it is the same as 'fold change'), but the agent thought the excess route made more sense. I just went with it. But honestly, it seems like a pointless move. The profile is in the shape of 'fold change'. So RCN=1 is no difference and becomes 0 with log. The excess computation without the floor is an arbitrary subtraction of a constant, and with the floor I believe it damages information we have about the amplicon (just arbitrarily setting things to 0 or 1)."*
  - The floor behavior IS preserved as an opt-in for sample-level APS aggregation, where treating below-background samples as "0 contribution from that amplicon" is more interpretable than letting them contribute negative values. Per user note (same chat): *"I think the 0-floor behavior made sense / makes sense when thinking about how much an individual amplicon contributes to a sample-level APS score."*
  - Rejection rationales (Fourier, curvature, multi-family BIC, peak_rcn-as-distinct-feature) are about either structural mismatch with the data, redundancy with existing features, or coverage by a different in-scope BRAIN entry. Recording the rationales prevents future-phase re-investigation.

- Alternatives Considered:
  - **Keep `summit_excess` as canonical and add `summit_rcn` as a parallel column** — rejected; doubles column surface for no semantic gain. The migration to raw RCN as default is cleaner with a rename.
  - **Add Fourier descriptors as an experimental feature mode for testing** — rejected per structural-mismatch reasoning above; testing would burn cycles on a method known not to fit the data.
  - **Add curvature features as a separate `--aps-feature curvature` mode** — rejected per redundancy with `--aps-feature shape` + existing parabola-fit curvature.
  - **Compute multi-family BIC vectors (flat / triangle / parabola / double-triangle / plateau) per amplicon and use as a sample-clustering feature** — rejected for sample clustering; partially folded into BRAIN15.18 for amplicon classification.
  - **Defer the new feature mode work to a post-Phase-15 phase** — rejected; user explicitly authorized adding 1-2 feature-mode candidates to Phase 15 scope (BRAIN15.27 + BRAIN15.28).

- Scope:
  - This decision applies to APS feature computation (`onionskin_core/aps.py`) and downstream APS clustering (`hierarchical_cluster()`, `--aps-feature` parser surface, posterior manifest construction).
  - Implementation work captured in `multi-agent/plans/PHASE15_BRAINSTORM.md` BRAIN15.27 (new feature modes) and BRAIN15.28 (rename + migration).
  - Future-phase candidates (UMAP / diffusion maps for dimensionality reduction; amplicon-level clustering complementary to sample clustering; multi-family parametric shape fits beyond the existing triangle/parabola) are captured in `multi-agent/plans/next/APS_SOUP.md`.

- Notes:
  - This DECISIONS entry is the canonical home for the rejection rationales. `multi-agent/plans/PHASE15_FEEDBACK.md` Round 1 follow-up section back-references this entry; future agents asking "should we try X?" for any rejected idea should be pointed here.
  - The CHANGELOG entry that lands BRAIN15.28's rename should explicitly call out the behavior change for `--aps-feature summit` (raw RCN vs floored excess) so users can validate against prior runs.

- Authors:
  - John M. Urban, Claude Code 2.1.x (claude-opus-4-7 ; Effort: High)

---

## [2026-04-29] `--onset-*` flags retain naked-prefix form (no `--timing-` prefix harmonization)

- Context:
  - The Timing argparse group at `onionskin.py:build_parser()` mixes prefix conventions: `--onset-rcn-threshold`, `--onset-span-rcn-threshold`, `--onset-method`, and `--onset-z-threshold` use a naked `--onset-` prefix, while peers `--timing-onset-quantile` and `--timing-exclude-loci` use the full `--timing-` prefix. Phase 14 Supplemental's CLI cleanup (cycle 14S.1a, v0.14.68; cycles 14S.3a/b, v0.14.71/v0.14.72) harmonized prefixes elsewhere but did not touch the Timing group.
  - The inconsistency was surfaced again during Phase 15 cycle 15.3a re-audit (CLOSED v0.14.79) when discussing scope for SPEC15.21 (architectural refactor + ODW threshold split). User asked whether to fold prefix harmonization into SPEC15.21.

- Decision:
  - Leave `--onset-*` flags in their current naked-prefix form. Do NOT rename to `--timing-onset-*` in SPEC15.21 or any current-phase work. This decision has now been made twice (Phase 14 Supplemental implicitly by omission; Phase 15 cycle 15.3a closeout explicitly). Reversible at any time — recording here so future agents don't re-litigate without new context.

- Rationale:
  - The `--onset-*` flags are conceptually onset-detector-level, distinct from the timing-pass orchestration knobs (`--timing-onset-quantile` is the global-reference quantile for `lag_from_earliest`; `--timing-exclude-loci` is the BED of excluded loci; both are timing-consumer-level). Forcing the `--timing-` prefix on the onset-detector knobs would conflate two abstraction layers (detector vs consumer) that are semantically distinct.
  - Renaming would create deprecation churn for cosmetic gain only — no behavioral or architectural improvement.
  - Onset-detection is a different detector concern from ODW active-stage / regression detection. If onset-detection ever gets its own dedicated detector framework (parallel to ODW), the natural argparse group home would be a new `--onset-*` group, not the `--timing-*` group.

- Alternatives Considered:
  - **Rename `--onset-*` to `--timing-onset-*` for prefix consistency in the Timing argparse group** — rejected; conflates abstraction layers + deprecation churn for cosmetic uniformity.
  - **Move `--onset-*` flags to a new `--onset-*` argparse group separate from Timing** — rejected for now; insufficient critical mass (4 flags) and no immediate architectural need. Could be revisited in a future onset-detector overhaul if onset-detection itself gets a SPEC.

- Scope:
  - Applies to `onionskin.py:build_parser()` Timing group flags. The four `--onset-*` flags (`--onset-rcn-threshold`, `--onset-span-rcn-threshold`, `--onset-method`, `--onset-z-threshold`) keep their current names.
  - Out of scope for SPEC15.21 + any current Phase 15 cycle.

- Notes:
  - This decision is reversible. If onset-detection becomes a separate detector framework in a future phase, OR if a focused Timing-group prefix harmonization SPEC is opened, the rename can be reconsidered then.
  - During the same SPEC15.21 design discussion, three other CLI/architectural decisions were locked: (1) split `--odw-prob-threshold` into per-direction `--odw-active-prob-threshold` (default 0.9) and `--odw-regression-prob-threshold` (default 0.8); (2) split `--odw-fold-threshold` into per-direction `--odw-active-fold-threshold` (default 1.25) and `--odw-regression-fold-threshold` (default 1.25); (3) relocate `--near-gap-bp` from the Timing group to a new Gap-analysis argparse group alongside `--gap-mask-strategy` and `--gap-mask-bin-threshold`. Those decisions are documented in SPEC15.21's priority body, not duplicated here.

- Authors:
  - John M. Urban, Claude Code 2.1.116 (claude-opus-4-7 ; Effort: Max)

---

## [2026-04-30] Phase 15 cycle 15.6a SPEC15.11 morphology feature definitions

- Context:
  - SPEC15.11 adds composite APS sample-clustering modes that combine per-amplicon morphology, sample-level APS scalars, and PCA-derived shape features.
  - The Role 1 audit correctly flagged three implementation-time schema choices that needed to be locked before coding: the `asymmetry_ratio` formula, whether to rename `width_above_threshold_bp`, and whether `log10area` should be emitted as a per-locus column.

- Decision:
  1. **`asymmetry_ratio` formula:** `abs(summit_bp - midpoint) / (locus_width / 2)`, clipped to `[0, 1]`. `0` means centered; `1` means the summit sits at an interval edge. If no refined summit is available, use the max-RCN bin midpoint.
  2. **Width column:** keep `width_above_threshold_bp` as the canonical per-locus column because it preserves the threshold-dependent meaning of the value. The `--aps-feature width` mode remains the user-facing alias.
  3. **`log10area` emission:** emit `log10area = log10(max(area_excess, 1.0))` as a per-locus contribution column so Layer-1/composite feature builders do not need to infer it from a transformed matrix later.
  4. **Shape score attachment:** attach `shape_score_raw` to APS contribution rows when an upstream calls/trajectory TSV exposes it; otherwise leave `NaN` and have composite feature matrices fill it to `0` for clustering.

- Rationale:
  - The asymmetry formula is deterministic, scale-normalized, cross-pipeline, and computable from columns already present in APS loci plus summit maps.
  - Keeping `width_above_threshold_bp` avoids hiding the biologically meaningful `--aps-width-threshold` dependency behind a vague `width` column.
  - Emitting `log10area` in the contribution schema makes the composite feature contract auditable and keeps HMM/Growth/RMS feature resolution shared.

- Scope:
  - Applies to Phase 15 cycle 15.6a Stages A-C only: APS schema migration, post-sum-floor mechanism, PCA/composite APS sample-clustering feature implementation, and harness extension.
  - Does not decide SPEC15.9 default flips or SPEC15.12 quantitative clustering defaults; those are deferred to cycle 15.6a-S1 after the authorized cycle 15.6b determinism work closes.

- Authors:
  - John M. Urban, Codex-cli 0.125.0 (GPT-5.5 ; Reasoning: Extra High)

---

## [2026-04-30] Phase 15 cycle 15.5a: active-stage summit refinement and one-pass summit-to-timing convergence

- Context:
  - SPEC15.7 and SPEC15.8 require cross-pipeline summit refinement that uses the timing/ODW state already inferred for each amplicon, plus a one-pass feedback loop where refined summit positions regenerate timing summaries.
  - The cycle explicitly excludes SUMMIT_SOUP-deferred estimator portability work: do not port growth's parabola estimator into RMS/HMM and do not unify HMM's `_estimate_parabola_bp` with shared `refinement.py` in this round.

- Decision:
  1. **Default strategy:** `--summit-stage-selection odw_best_shape`, with per-pipeline overrides (`--growth-summit-stage-selection`, `--rms-summit-stage-selection`, `--hmm-summit-stage-selection`). If per-stage shape evidence is absent, `odw_best_shape` falls back to all ODW-active stages rather than inventing a shape ranking.
  2. **Strategy menu:** expose `max_rcn`, `onset`, `last_active`, `odw_all`, `odw_best_shape`, and HMM-only `odw_narrowest_state`; reject the HMM-only strategy for growth/RMS at argument validation.
  3. **Output layout:** preserve original timing/summit artifacts and write explicit convergence audit files:
     - Growth: `09b-active-stage-summit/` and `10b-timing-updated/`.
     - RMS: `15b-active-stage-summit/` and `15c-timing-updated/`.
     - HMM: `20-active-stage-summit/` and `21-timing-updated/`.
  4. **Canonical promotion:** after convergence, promote step-4 timing back to each pipeline's canonical timing path. For HMM, promote step-3 summit estimates back to `18-timing/hmm_summit_estimates.tsv` and step-4 timing back to `18-timing/hmm_timing.tsv` while retaining step-numbered audit copies.
  5. **HMM sliding offset:** use step-5 per-sample `indiv_samples/sample_tracks.tsv` when present and aggregate HMM tracks as a fallback; expose `--hmm-sliding-offset-step` for the bp grid.
  6. **Evaluation harness:** `scripts/summit_strategy_evaluation.py` owns the 6-strategy × ODW-knob matrix. It now writes report columns for by-eye/summit-eval truth counts and nearest-truth summit-distance summaries when real matrix outputs exist.

- Rationale:
  - ODW-constrained stages are the biologically relevant stage subset for summit refinement; using every stage can let inactive/noisy samples pull a summit away from the developmental signal being timed.
  - `odw_best_shape` is the best default contract because it prefers stage evidence with stronger amplicon-like shape while preserving a conservative all-active fallback where a pipeline lacks per-stage shape scores.
  - Canonical promotion keeps downstream consumers on stable paths while step-numbered files make the behavioral change audit-friendly.
  - HMM step-20/21 directories are placed after step 19 in the physical layout because the active-stage masks depend on timing generated at step 18; the canonical promoted files keep `18-timing/` as the user-facing timing surface.

- In-situ caveats for re-audit:
  - HMM APS currently still runs before step-21 timing promotion. Today that APS path does not consume `hmm_timing.tsv`, but the cycle 15.5a re-auditor should verify this remains harmless before closeout and before SPEC15.6-style downstream reliability work assumes final timing is universally upstream of APS.
  - Growth/RMS `max_rcn_stage` parity is schema-complete but not mathematically identical to HMM's per-trajectory windowed RCN argmax in every path. Re-audit should decide whether this is acceptable for the current strategy selector or whether a follow-up should share a true windowed `max_rcn_stage` helper across all pipelines.
  - The harness dry-run was validated in Role 2. A full 36-cell benchmark matrix over the full-chromosome truth fixtures is implemented but was not executed in this round because it is much larger than the standard validation suite; closeout should decide whether to run it before writing the CHANGELOG entry.

- Scope:
  - Applies to Phase 15 cycle 15.5a implementation and closeout only.
  - Does not decide the future SUMMIT_SOUP estimator-portability work or the out-of-scope IBM-C6E iterative EM-style summit↔timing convergence.

- Authors:
  - John M. Urban, Codex (codex-cli 0.126.0-alpha.8, GPT-5.5, Reasoning: Extra High)

### [2026-04-30] R2-J round resolutions to the in-situ caveats above

- **Caveat 1 (HMM APS ordering vs canonical timing promotion):** **DEFERRED to cycle 15.6a R1 audit.** The R3 Round 2 re-audit confirmed the ordering is currently harmless because HMM APS does not consume `hmm_timing.tsv`. Cycle 15.6a R1 must verify this remains harmless before any APS code starts reading timing in a way that would break the ordering. Captured in cycle 15.5a section of `multi-agent/plans/PHASE15_AUDIT_LOG.md` and in cycle 15.6a notes column of `PHASE15_STRATEGY.md`.
- **Caveat 2 (Growth/RMS `max_rcn_stage` parity — Option A vs Option B):** **OPTION A LANDED in R2-J.** User's directive: cross-pipeline parity is the right call. Implementation: new shared `onionskin_core/summit_metrics.py` module with `compute_max_rcn_stage()` (factored from HMM's `hmm_ported_analyses.py:_max_rcn_stage`); HMM now calls it via a thin wrapper preserving the trajectory-based call site; Growth + RMS now call it post-write so the per-stage RCN bedGraph dependency is available. Behavior is now mathematically identical across all three pipelines (windowed argmax of per-stage peak fold over a half-width clamped to `[1 kb, min(width // 10, 50 kb)]`). The prior buggy Growth implementation (max of stage NUMBERS present) and RMS implementation (`int(stage)` per row, trivially equal to `stage`) are retired.
- **Caveat 3 (full 36-cell benchmark matrix on full-chromosome truth fixtures):** **DEFERRED to a stand-alone `dev/runs/` task post-closeout.** R3 Round 2 confirmed this is a benchmark report rather than a contract gate, and the existing default-strategy lock at `odw_best_shape` was made on design-contract grounds (not benchmark winner). The harness at `scripts/summit_strategy_evaluation.py` is operational; the user (or a future agent) can run it any time without blocking cycle closeout.
- **R2-J round outcome — RMS step-3 active-stage refinement (R3 finding F-R2-1):** **FIXED.** `summit_convergence.run_rms_active_stage_convergence` now reads RMS per-stage `_stage<N>_summit_estimates.tsv` files post-loop, spatial-matches each timing-row's call to the per-stage rows by `(chrom, start, end)` overlap, filters to strategy-selected stages, and takes the median of `parabola_summit_bp` over the surviving rows (`peak_raw_bp` fallback if no parabola data; `unified_calls.peak`/`call_peak` fallback if no per-stage rows match). The prior implementation hard-wired `step3_summit_bp == step1_summit_bp` for every call, making RMS step-3 active-stage refinement a no-op. After the fix, RMS produces shifted step-3 summits on real toy data when strategy-selected stages have parabola summits at a different bp than the unified call peak.
- **R2-J round outcome — programmatic shift regression tests (R3 finding F-R2-3):** Added `test_rms_active_stage_convergence_shifts_step3_summit` (mirrors Growth's existing `test_growth_active_stage_convergence_writes_step_files` pattern) and `test_hmm_active_stage_mask_shifts_summit` (asserts `_compute_summit_estimates` produces different mask-sensitive output when an active-stages mask excludes a stage whose peak sits at a different bp than the surviving stages). Both tests pass post-R2-J fix.

---

## [2026-04-29] Phase 15 cycle 15.4a — SPEC15.6 Round 4 design refinements (post-R1-audit, between R2 and R3)

- Context:
  - SPEC15.6 was locked at SPEC engineering close (2026-04-28) with deliverable 3 (flat-sample detection) and deliverable 4 (`--aps-preamp-threshold` flag with `auto` mode) in their original Round 3 cross-pipeline form. Cycle 15.4a Role 2 (Codex GPT-5.5 Extra High) implemented the original contract; commits `a905388` through `3c721de` landed Stages A-L with Stage I implementing `--aps-preamp-threshold` per the original SPEC text.
  - Between R2 implementation and R3 re-audit, user surfaced sharp design questions: should the per-locus flat-sample test be Bayesian-ODW-consistent? Why is `--aps-preamp-threshold` an APS-side flag when flat-sample detection is conceptually a multistage-unification + reliability output? Was posterior stage-1 anchoring deferred — and if so, why, given it's the whole motivation for flat-sample detection?
  - Discussion (chat 2026-04-29) produced four locked design refinements that revise the SPEC15.6 contract before R3 re-audit fires.

- Decision:
  1. **Bayesian-ODW-consistent per-locus flat test (SPEC15.6 deliverable 3 reworked).** Per-locus "is this sample showing growth at this locus?" reuses the SPEC15.4 Bayesian ODW transition primitive with `previous_rcn = chrom_median = 1.0` and `previous_log2 = 0.0` (same primitive that powers stage-1-vs-background detection). Defaults inherit from `--odw-active-prob-threshold` (0.9) and `--odw-active-fold-threshold` (1.25, post-SPEC15.21). NEW per-locus override flags `--flat-sample-prob-threshold` and `--flat-sample-fold-threshold` allow tuning the flat-vs-amplified test independently of stage-to-stage growth. NEW sample-level rollup flag `--flat-sample-min-flat-fraction` (default 1.0). The hard-threshold "Summit RCN < 1.5×" form from BRAINSTORM 2026-04-19 is RETIRED.
  2. **`--aps-preamp-threshold` retired (SPEC15.6 deliverable 4 reworked).** The original `--aps-preamp-threshold` flag (with `auto` gap-detection mode) is dropped from SPEC scope. The principled per-locus Bayesian test makes both the raw-threshold form and the gap-detection heuristic redundant. Flat-sample logic relocates to a new `onionskin_core/flat_sample.py` module (sister to `reliability.py`) — flat-sample detection is a multistage-unification + reliability *consumer*, not APS-owned.
  3. **Auto-switch-to-chrom-median when zero flats detected, with `--allow-nonflat-ref-stage` override.** Pipeline prints flat-sample count to stderr; if 0 flats are detected, emits a loud warning AND auto-switches any posterior analyses that would otherwise use ref-stage normalization to chrom-median. `--allow-nonflat-ref-stage` (default off) lets the user opt OUT.
  4. **Posterior stage-1 anchoring promoted from "deferred" to in-cycle-15.4a (SPEC15.6 deliverable 6 extended).** Each pipeline's posterior/clustering surface anchors flat-flagged samples to its stage-1 / pre-amp slot BEFORE clustering remaining samples into k-1 stages. Cross-pipeline contract; per-pipeline implementation. Closes IBM-2026-04-14:1's "Exit condition: Stage 1 of the posterior contains only pre-amplification samples".
  5. **`--known-reliable-amplicons` synonym for `--keep-amplicons-bed` (SPEC15.6 deliverable 9 extended).** Both flag spellings accepted. Trim to one in a later phase if desired.
  6. **HMM-specific state-path-aware reliability extensions moved from SPEC15.6 OPTIONAL to SPEC15.13 deliverable 7.** Phase 15's theme is HMM completeness; SAPS provides the natural data substrate for state-path-aware reliability metrics. Cycle 15.4a (SPEC15.6) commits only to the cross-pipeline shared schema for HMM; HMM extras land in cycle 15.7a alongside SAPS.

- Rationale:
  - Decision 1: harmonizes flat-sample-detection's per-locus mechanism with what we already call "growth" everywhere else (SPEC15.4 Bayesian ODW). One canonical primitive instead of two parallel hard thresholds.
  - Decision 2: IBM-2026-04-14:1's "Deprecated solution ideas" section already established the raw-threshold + gap-detection paths as fragile. The principled Bayesian test makes the flag redundant. Module ownership reflects logical ownership (multistage-unification + reliability *produce* flat-sample state; APS *consumes* it via the sidecar).
  - Decision 3: no flats detected → no valid stage-1 anchor → ref-stage normalization is biologically untrustworthy → auto-switch to chrom-median (which doesn't depend on a stage-1 anchor) is the safe default. Override flag for explicit-opt-in cases. Complements the project's existing chrom-median-defaults direction.
  - Decision 4: posterior stage-1 anchoring is the WHOLE POINT of having flat-sample detection per IBM-2026-04-14:1's exit condition. Round-3 cross-pipeline reframing inadvertently dropped the anchoring step in narrowing to "shared sidecar contract"; catching it before R3 re-audit fires is the right time. User explicitly framed: "the whole point of having a flat sample detection system was to be able to define posterior stage 1."
  - Decision 5: better anchors the flag to the reliability-test escape-hatch use case; coexists with the more general original name; deferred trim avoids premature lock-in.
  - Decision 6: matches phase theme (HMM completeness); SAPS data substrate makes HMM extras a clean fit alongside SAPS; cycle 15.4a focus stays on the cross-pipeline shared contract.

- Alternatives Considered:
  - Decision 1: keep the hard 1.5× form. Rejected — creates a parallel-threshold-system inconsistency with the rest of the codebase and is no easier to set a priori than the Bayesian form.
  - Decision 2: keep `--aps-preamp-threshold` as a manual user-override knob. Rejected — module-ownership argument made the cleaner break preferable; the Bayesian test serves the tuning use case via the new `--flat-sample-*` flags.
  - Decision 3: no auto-switch (just warn). Rejected — silent biological failure mode is too easy to overlook; loud warning + safe default + override flag is the right risk balance.
  - Decision 4: defer anchoring to a future SOUP / new SPEC priority. Rejected — anchoring is the user-visible biological fix and lives at the same surface as detection; deferring duplicates effort and breaks the "one cycle = one coherent contract" pattern.
  - Decision 5: pick one name (either `--keep-amplicons-bed` or `--known-reliable-amplicons`). Deferred — both serve different communication needs; deciding which is canonical is a phase-15-closeout matter at earliest.
  - Decision 6: keep HMM extras in SPEC15.6 OPTIONAL. Rejected per phase-theme argument + SAPS-data-substrate argument.

- Scope:
  - Applies to Phase 15 cycle 15.4a Role 1 re-audit (R3) and any post-R3 remediation. The R3 auditor verifies the live code (post-R2 commits) against the refined contract; gaps become Stage I.5 / J.5 / extended Stage I / etc. as needed.
  - Applies to Phase 15 cycle 15.7a Role 1 audit (SPEC15.13 deliverable 7).
  - Out of scope for any other phase or priority.

- Notes:
  - Codex's Stage I implementation (commit `3fabb05` for the flag wiring; flat-sample sidecar emission in `aps.py:1370` + `hmm_ported_analyses.py:173-174`) consumed the original SPEC15.6 deliverable 3 + 4 contract. The R3 re-audit + any Stage I.5 / J.5 remediation cycle implements the refined contract from this DECISIONS entry. The user authorized this mid-cycle SPEC refinement explicitly per scope-authority rule (broadening is encouraged when surfaced to the user).
  - Cross-references: SPEC15.6 deliverables 3, 4, 6, 7, 9; SPEC15.13 deliverable 7; STRATEGY cycle 15.4a + 15.7a row notes; cycle 15.4a section in `PHASE15_AUDIT_LOG.md`.

- Authors:
  - John M. Urban, Claude Code 2.1.116 (claude-opus-4-7 ; Effort: Max)

---

## [2026-04-29] Phase 15 mid-phase amendment: insert SPEC15.22 + cycle 15.4b for gap-analysis architectural cleanup; concretize landing slots for cycle 15.4a-S1 surfaced issues

- Context:
  - Cycle 15.4a closed at v0.14.80 (cross-pipeline reliability scorer + Bayesian flat-sample contract + posterior stage-1 anchoring). Cycle 15.4a-S1 closed at v0.14.81 (R3-O4 follow-up: HMM step15-aps emits MAD bedGraphs; flat_sample.py consumes them). Both cycles surfaced multiple architectural defects in adjacent surfaces that the planning routine had not anticipated.
  - User explicit framing (chat 2026-04-29): "the only true difference between pipelines is how they call amplicons. Then almost everything thereafter is game for each pipeline at least by analogy." Multiple "X-pipeline-only" defects keep getting exposed; the framework reconciliation work needed to fix them is now substantial.
  - Five issues surfaced during cycle 15.4a + 15.4a-S1:
    1. `[ISSUE:2026-04-29:3]` `--compute-aps` / `--posterior` should be `--skip-aps` / `--skip-posterior` (semantic inversion). Code-dive verified `--compute-aps` only suppresses growth APS; RMS + HMM run their APS regardless.
    2. `[ISSUE:2026-04-29:4]` Posterior-regression tracking gap (PuffStep covers detection only).
    3. `[ISSUE:2026-04-29:5]` Gap-analysis annotation-only across all 3 pipelines + HMM placement wrong (step 19) + parallel keep/exclude frameworks need reconciliation. `_finalize_amplicon_recommendations` (Priority 4.9 from v0.4.13) is GROWTH-PIPELINE-ONLY despite being a clearly cross-pipeline concept.
    4. `[ISSUE:2026-04-29:6]` Sample-level APS aggregation does NOT use reliability-filtered loci.
    5. `[ISSUE:2026-04-29:7]` Systemic cross-pipeline-parity audit gap (process improvement).
  - Plus a new BRAINSTORM entry: `[2026-04-29] Bayesian-weighted amplicon reliability` (generalize binary keep/reject to a weight-in-[0,1] degree-of-belief framework).

- Decision:
  1. **Insert SPEC15.22 + cycle 15.4b into Phase 15** as a mid-phase amendment, in execution order between cycle 15.4a-S1 (closed) and cycle 15.5a (queued). SPEC15.22 covers `[ISSUE:2026-04-29:5]` PARTIAL reconciliation: HMM gap-analysis step relocation (step 19 → between multistage-unification and APS, second HMM step renumber); cross-pipeline `gap_suspect` filter contract integrated into the cycle-15.4a reliability scorer; cross-pipeline port of `_finalize_amplicon_recommendations` to RMS + HMM. Belongs to Phase 4 (HMM shape-filter / multistage unification / amplicon reliability) since gap-analysis-as-filter fits the reliability framework theme.
  2. **Light-touch deliverable references added to existing planned cycles** so the issues stick without bloating the SPEC with new priority bodies:
     - SPEC15.12 (cycle 15.6a): mid-phase amendment paragraph references `[ISSUE:2026-04-29:6]`.
     - SPEC15.19 (cycle 15.10a): mid-phase amendment paragraph references `[ISSUE:2026-04-29:3]`.
     - SPEC15.20 (cycle 15.10a): mid-phase amendment paragraph references `[ISSUE:2026-04-29:7]` + IBM-C14 closeout + `[ISSUE:2026-04-29:1]` closeout-state verification.
  3. **Defer to a future phase** (NOT addressed in Phase 15):
     - `[ISSUE:2026-04-29:4]` (posterior-regression tracking — requires test-infrastructure work too large for closeout cycle).
     - `[ISSUE:2026-04-29:5]` part-b (full Priority-4.9-vs-reliability-scorer-vs-gap-analysis framework reconciliation — design questions still open; user direction is "merge smartly, retain all non-redundant analyses, do not eliminate quality analyses").
     - `multi-agent/tracking/BRAINSTORM.md [2026-04-29] Bayesian-weighted amplicon reliability` (full re-derivation; subset lands in cycle 15.6a per the SPEC15.12 light-touch).
     - `--timing-exclude-loci` folding into the unified user-input keep/exclude flag space.

- Rationale:
  - **Decision 1 (insert SPEC15.22 + cycle 15.4b):** if cycle 15.5a's cross-pipeline summit refinement runs on the unfiltered call set, refinement decisions get committed on noisy gap-suspect amplicons that should have been filtered. The cost of doing the partial fix now (one cycle, ~200-300 line surface, well-scoped) is much lower than the cost of doing 5+ downstream cycles on partially-correct upstream call sets and then re-running them. Theme alignment with Phase 4 (HMM shape-filter / multistage unification / amplicon reliability) is strong.
  - **Decision 2 (light-touch references):** the four issues #3, #6, #7, and IBM-C14 closeout fit naturally into existing planned priorities (15.12, 15.19, 15.20). Adding new SPEC priorities for each would balloon Phase 15 into 25+ priorities; the light-touch approach pins the issues to the right cycles without that bloat.
  - **Decision 3 (defer to future phase):** the deferred items are too big for Phase 15 closeout cycle without disrupting the 15.10a editorial scope. They will benefit from being designed as a coherent unit (the Bayesian-weighted reliability framing influences how the keep/exclude reconciliation should be designed, which influences the posterior-regression-tracking strategy, etc.). Rolling them into a future phase together is cleaner than scattering them across 15.10a's housekeeping.

- Alternatives Considered:
  - Insert SPEC15.22 in Phase 9 (architectural cleanup, alongside SPEC15.21). Rejected — Phase 9's natural execution order is AFTER Phases 5-8, which contradicts the "before cycle 15.5a" requirement.
  - Skip SPEC15.22 entirely in Phase 15; defer all gap-analysis cleanup to a future phase. Rejected — cycles 15.5a–15.10a all benefit from running on the gap-filtered call set, and the HMM step-19-placement defect is severe enough that leaving it for a future phase risks compounding rework.
  - Add new SPEC priorities for issues #3 / #6 / #7 instead of light-touch references. Rejected — bloats Phase 15 priority count; light-touch keeps the planning surface manageable while preserving traceability.
  - Promote the Bayesian-weighted reliability brainstorm to a SOUP file with a SOUP ID. Considered; deferred — it's still an idea, not a candidate phase. If/when it graduates to candidate-phase status, it can move to a SOUP file with a SOUP ID per the AGENT_CONVENTIONS § Future-phase planning surfaces lifecycle.

- Scope:
  - Applies to Phase 15 audit/implement stage (`PHASE15_SPEC.md`, `PHASE15_STRATEGY.md`, the live cycle-15.4b R1 audit + downstream cycles).
  - Does NOT apply to any future phase's SPEC engineering — the deferred items are in `multi-agent/tracking/KNOWN_ISSUES.md` and `multi-agent/tracking/BRAINSTORM.md` for the future phase's brainstorming-stage to pick up.

- Notes:
  - User's strict convention reminder (chat 2026-04-29): "nothing gets given a Phase number according to our conventions. They are either a KNOWN_ISSUE with ID, a tracked BRAINSTORM.md ID, or in a SOUP file -- or sometimes an IBS-style file with ID." This decision honors that — only SPEC15.22 + cycle 15.4b receive Phase numbers (they're now formally in PHASE15_SPEC.md + PHASE15_STRATEGY.md). The deferred items remain as KNOWN_ISSUE / BRAINSTORM IDs.
  - User's strict scope-authority rule: agents do not narrow scope without user approval. This decision is a USER-DRIVEN broadening (per cycle 15.4a-S1 chat 2026-04-29 explicit greenlight) — not an agent-driven narrowing. Specifically the user said: "Let's do everything the way you leaned and recommended."
  - Cross-references: SPEC15.22 priority body in `PHASE15_SPEC.md`; cycle 15.4b row in `PHASE15_STRATEGY.md`; the four KNOWN_ISSUES entries `[ISSUE:2026-04-29:3..7]`; BRAINSTORM `[2026-04-29] Bayesian-weighted amplicon reliability`; `multi-agent/plans/archived/20260412-ROADSTRAVELED.md:158-217` (Priority 4.9 archived design notes).

- Authors:
  - John M. Urban, Claude Code 2.1.116 (claude-opus-4-7 ; Effort: Max)

---

## [2026-05-03] Per-pipeline + run-level JSON sidecar convention (cross-pipeline; project-wide)

**Decision:** every onionskin run emits a two-tier JSON sidecar pair documenting all parameters + computed values used to produce the run's outputs. Provides the canonical anchor for parameter audits, downstream-consumer parameter lookups, and reproducibility.

**Tier 1 — Run-level sidecar at `<run-out>/00-run-metadata/run_info.json`** (controller writes ONCE at startup; static run-wide fields):
- `manifest_path` — input manifest path.
- `version` — onionskin version.
- `timestamp_start_iso8601` — controller startup time.
- `cli_args_resolved` — full argparse Namespace as JSON (every CLI flag's resolved value).
- `pipelines_selected` — value of `--pipelines` (e.g., `["growth", "rcn-mean-shift", "hmm"]` or subset).
- `pipelines_run` — actual pipelines that fired.
- `chromosomes_run` — value of `--chromosomes`.
- `out_dir` — root output directory.
- Schema-stable across cycles; new fields added by appending, not by reshaping.

**Tier 2 — Per-pipeline sidecar at `<run-out>/01-prior/<pipeline>/00-pipeline-metadata/pipeline_info.json`** (each pipeline writes its own AFTER running; dynamic computed fields):
- `pipeline_name` — `hmm`/`growth`/`rms`.
- `cross_reference: run_metadata_path` — relative path to the Tier-1 run-level sidecar (e.g., `"../../00-run-metadata/run_info.json"`).
- `pipeline_completed_at` — ISO-8601 timestamp.
- Pipeline-specific computed fields (per-pipeline section schema documented in respective SPEC priorities — e.g., HMM's section in SPEC15.14 deliverable 10 includes `statepath_base`, `state_label_map`, `state_count`, `emission_params`, `gap_mask_hash`, `chrom_renorm_stats`, `pass_metadata`).

**Why this convention exists:** (i) single canonical place to look up "what parameter was used for run X" — answers any reproducibility question; (ii) downstream consumers (SAPS, trajectory clustering, posterior-phase inheritance, plots, future-phase analyses) can extract the labeling regime + emission parameters + computed values without parsing the entire output set; (iii) cross-cycle schema stability so future-phase additions don't fragment the convention.

**Precedence rule for downstream consumers (locked):**
- The per-pipeline JSON sidecar is the canonical anchor — read it first.
- If the file being processed also has self-describing headers (e.g., bedGraph emissions; per SPEC15.14 deliverable 9), cross-check header values against sidecar values. If they DISAGREE, warn loudly + prefer the sidecar values (sidecar is authoritative).
- If the JSON sidecar is missing (e.g., processing older or partial output set) but headers are present, fall back to header values + warn that the sidecar is missing.
- If NEITHER the JSON sidecar NOR the self-describing header is present, fail hard with a clear error.

**Where the per-tier convention is implemented:**
- **Cycle 15.7a (SPEC15.14 deliverables 9 + 10):** lands the controller-level Tier-1 `run_info.json` + the HMM-specific Tier-2 `pipeline_info.json` + the `read_state_label_header` parser helper in new `onionskin_core/state_path_io.py` module. Self-describing file headers also land for HMM state-value-bearing emissions.
- **Cycle 15.8a (SPEC15.17 + future):** Growth + RMS Tier-2 `pipeline_info.json` extensions inherit the schema from this DECISIONS entry. Cross-pipeline schema parity is the SPEC15.17 deliverable.
- **Future cycles / phases:** any new pipeline-specific computed values + new pipelines (cross-pipeline trajectory tracking per `multi-agent/plans/next/TRAJECTORY_SOUP.md`; SAPS clustering per `multi-agent/plans/next/APS_SOUP.md` Track 3 future-phase work; etc.) extend the convention by adding fields, never by reshaping the two-tier structure.

**Notes:**
- The two-tier structure is the locked design (chosen 2026-05-03 over the alternative of a single top-level JSON containing everything). Reasoning: clean separation of static run-wide fields (Tier 1; written once at controller startup) from dynamic computed fields (Tier 2; written per-pipeline after each pipeline runs); avoids parallel-write race conditions; preserves locality (pipeline-specific consumers stay in pipeline directory); cross-reference field provides the "all-in-one-place" anchor when desired.
- Self-describing file headers (per SPEC15.14 deliverable 9) and the JSON sidecar pair are complementary, not redundant. Headers carry inline provenance for self-containment; sidecars carry the canonical run-wide + pipeline-specific record.
- `run_info.json` schema follows the cycle-15.4a-S4 first-pass-mask-metadata.json pattern (precedent for JSON sidecars in `00-`-prefixed metadata directories).

**Cross-references:**
- SPEC15.14 deliverables 9 + 10 (HMM-specific instance + controller-level sidecar; cycle 15.7a).
- SPEC15.17 (cross-pipeline analysis-surface parity; cycle 15.8a) — extends to Growth + RMS.
- Cycle 15.4a-S4 first-pass-mask-metadata.json convention precedent.
- `feedback_no_backwards_compat.md` — sidecar-schema additions are permitted freely without backwards-compat concerns since onionskin has no external users yet.

**Authors:**
- John M. Urban (Principal direction 2026-05-03 chat: "save all parameters there such that any question of what parameter was used could be answered by the json file, and I'd argue all pipelines should have such a json file"; "headers keep it but the json file becomes the default, potentially checked against the headers if/when they are present as a sanity check"; "the latter [two-tier] means it can all land in one step").
- Claude Code 2.1.126 (claude-opus-4-7 ; Effort: Max).

---

## [2026-05-04] SPEC15.24 d1 HMM per-stage parabola summit emission placement lock (Phase 15 cycle 15.7b)

**Decision:** Q1 is LOCKED to Option A for SPEC15.24 d1. HMM per-stage parabola summit emission lands as a satellite function named `run_hmm_per_stage_parabola_summit_estimates(step4_dir, step11_dir, out_dir)` in `onionskin_core/hmm_ported_analyses.py`, and emits one TSV per stage into `steps["step13"]` (`13-summit-refinement/`) using the filename pattern `hmm_stage{N}_summit_estimates.tsv`.

**Schema (locked):**
- `trajectory_id`
- `chrom`
- `start`
- `end`
- `peak_raw_bp`
- `parabola_summit_bp`
- `parabola_uncertainty_bp`
- `stage`
- `max_rcn_stage`

**Locked implementation details:**
- Data source split is intentional: stage bedGraphs come from `steps["step4"]`; per-stage HMM trajectory windows come from step-11 amplicon metrics, accessed through the renamed helpers in `hmm_summit_refinement.py`.
- `parabola_uncertainty_bp` is populated as `NaN` because HMM's local `_estimate_parabola_bp` primitive returns only the bp estimate, not an uncertainty interval.
- The step-20 convergence consumer follows the same per-stage glob-discovery pattern as RMS, reading `hmm_stage*.tsv` from `steps["step13"]`.

**Why this is locked:**
- Strongest cross-pipeline parity. The HMM consumer `run_hmm_active_stage_convergence` can mirror `run_rms_active_stage_convergence` in file discovery and median-over-selected-stages behavior instead of diverging into a single-file step-18-specific path.
- Avoids changing the existing step-18 / step-20 combined-profile timing path, which remains in place per SPEC15.24 d2 coexistence language.

**Explicit non-decision / rejected alternative:**
- Option B (single concatenated per-stage summit TSV emitted inside `run_hmm_timing` at step 18) is **NOT** being implemented in cycle 15.7b.

**Cross-references:**
- SPEC15.24 d1 + d2.
- `multi-agent/plans/PHASE15_AUDIT_LOG.md` cycle 15.7b R1 finding F1 / Q1.
- `feedback_cross_pipeline_parity_blindspot.md` parity principle.

**Authors:**
- John M. Urban (Principal lock conveyed in cycle 15.7b Role 2 instruction, 2026-05-04).
