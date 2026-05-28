# Phase 15 Final Overseer — Pass 1 audit report

**Date:** 2026-05-05 EDT
**Authors:** Claude Code 2.1.126 (claude-opus-4-7 ; Effort: Max) — Final Overseer Pass 1, fresh-chat session
**Workflow:** v2 audit/implement/reaudit loop (`multi-agent/workflows/archived/phase-development-system_PDS-v2.md`); Final Overseer 2-pass pattern; Pass 1 (audit-only; no code/doc/test edits; no git operations).
**Phase:** 15 (HMM completeness + cross-pipeline enrichment) — close gate satisfied per `CHANGELOG.md [v0.14.97]` (cycles 15.10a + 15.10a-S1 + 15.10a-S2 all CLOSED).

---

## Verdict

**OPEN-with-5-tracking-drift-findings (1 substantive ROADMAP staleness; 4 in-situ tracking-doc cross-reference gaps). No code/test/runtime defects identified. No remediation cycle required.**

The substantive Phase 15 deliverables are clean: code state matches R3 closeout judgments across all 19 cycle-rounds; cross-pipeline parity is real (Growth + RMS + HMM diagnostic-summits BED/TSV verified emitted at runtime); cycle 15.10a-S2's curated 68-strategy menu + corrigendum-aligned triangle-apex NaN-emit is implemented exactly as documented (`onionskin_core/summit_strategies.py:578-585`); deleted modules from cycle 15.10a R2 (`summit_aggregation.py`, `summit_diagnostics.py`, `summit_diagnostic_runner.py`) are gone from the live tree; per-pipeline posterior architecture from cycle 15.10a-S1 is wired correctly. The cycle 15.10a-S2 orchestrator-as-R3 cycle-final exception was justified — its F1–F8 disposition stands up to fresh-eyes review.

The findings are entirely tracking-surface drift introduced by the supplemental cycles 15.10a-S1 + 15.10a-S2 landing AFTER cycle 15.10a's SPEC15.20 d1 closeout sweep ran. The closeout sweep was correct as-of-cycle-15.10a; the two supplemental cycles should have triggered ROADMAP + IBM + SUMMIT_SOUP + AUDIT_LOG updates at their R3/closeout but didn't.

## Summary

Phase 15 ran 19 cycle-rounds across 24 SPEC priorities (SPEC15.1–SPEC15.24). All 24 priorities have DONE markers in `PHASE15_SPEC.md` cycle table + corresponding ROADMAP entries + AUDIT_LOG cycle headings. Each cycle's R3 closeout matches its R1 audit's F-findings repair contract. Code state at HEAD (commit `d0d1eb9`) matches R3 closeout claims for cycle 15.10a-S2 (verified by reading [summit_strategies.py:560-585](onionskin_core/summit_strategies.py#L560-L585) for triangle-apex NaN-emit; runtime check that `dev/runs/toy_out/01-prior/01-hmm/13-summit-refinement/diagnostic-summits/hmm_diagnostic_summits.{bed,tsv}` + Growth + RMS analogues all exist; verified deleted modules from cycle 15.10a R2 are gone).

The ROADMAP was last updated at cycle 15.10a closeout (v0.14.95) and reflects the 9-strategy 2-flag F5 implementation + claims Phase 15 closed at v0.14.95. Cycle 15.10a-S2 reverted that 9-strategy implementation entirely + replaced with curated 68-strategy single-flag menu in `summit_strategies.py` + closed Phase 15 at v0.14.97. Cycle 15.10a-S1 (v0.14.96) is also entirely absent from ROADMAP. Several other tracking surfaces (SUMMIT_SOUP Open brainstorm questions 13+14; INTENDED-BUT-MISSED-PRIOR-TO-14.md IBM-C4 disposition; KNOWN_ISSUES.md newest-first sort; SUMMIT_SOUP corrigendum + PHASE15_AUDIT_LOG.md anti-deferral reminder) reference "cycle 15.10a-S3" or "9-strategy menu" or stale module names. None of these affect runtime behavior — they are documentation/tracking-surface coherence only.

The cycle 15.10a-S2 orchestrator-as-R3 cycle-final exception (per Principal direction 2026-05-05 — Final Overseer absorbs independent-eyes verification at phase level) is verified justified: the documented F1–F8 dispositions hold up under independent code inspection; the corrigendum's documentation across AUDIT_LOG + KNOWN_ISSUES + SUMMIT_SOUP + PIPELINE_PARITY_SOUP is consistent; runtime triangle-apex NaN-emit is implemented at the documented code location. No defect this fresh-eyes pass would have caught was missed by orchestrator-as-R3.

Cross-pipeline parity was applied uniformly: every Phase 15 deliverable that landed cross-pipeline (Bayesian ODW, summit refinement strategy menu, ODW-confined refinement, summit↔timing convergence, reliability/flat-sample machinery, APS analysis-surface parity, master APS catalog, plot ports, peak-summary extension, gap-analysis architectural cleanup, posterior architecture, summit-aggregation 68-strategy menu) lands at all 3 pipelines; intentional asymmetries (HMM-only `odw-narrowest-state-*` selectors; Growth-retains-bins-`>HI` Pass-2; Growth-only overlap-resolution plots; Growth-only triangle FIT for fork elongation flagged in `PIPELINE_PARITY_SOUP` Item 1) are documented + cross-referenced.

## Findings — categorized for orchestrator triage

### Category 1: Code/test defects (orchestrator in-situ repair acceptable; no new cycle needed)

**None.** Live code at HEAD (`d0d1eb9`) matches all R3 closeout claims I spot-checked. The cycle 15.10a-S2 R2 deletions of deprecated F5 scaffold modules + curated 68-strategy registry + canonical-producer/consumer override path + per-pipeline integration tests are all present + correct.

### Category 2: Tracking/doc drift (orchestrator in-situ repair acceptable)

**T1. [SUBSTANTIVE] `ROADMAP.md` Phase 15 section is stale at cycle 15.10a-S2 (v0.14.97) close.** Severity: moderate.

Symptoms (multiple, all on the same surface):

- [ROADMAP.md:1553](ROADMAP.md#L1553): `Phase 15 — HMM completeness + cross-pipeline enrichment ✓ DONE (v0.14.95)` — should be `v0.14.97` (Phase 15 close gate is `15.10a + 15.10a-S1 + 15.10a-S2`; close-version is v0.14.97 per [CHANGELOG.md:64](CHANGELOG.md#L64) `[v0.14.97]`).
- [ROADMAP.md:1555](ROADMAP.md#L1555): `Closed: 2026-05-05 (cycle 15.10a closeout at v0.14.95)` — should mention 15.10a-S1 + 15.10a-S2 closure pattern + v0.14.97.
- [ROADMAP.md:1559-1560](ROADMAP.md#L1559-L1560): `Phase 15 spanned 16 cycles (15.1a through 15.10a + 4 supplemental sub-cycles S1/S2/S3/S4 under 15.4a + 15.4b + 15.6b + 15.6a-S1 + 15.7b)` — missing 15.10a-S1 + 15.10a-S2 (actual final count is 18 cycle-rounds, not 16).
- [ROADMAP.md:1577-1579](ROADMAP.md#L1577-L1579): `**onionskin_core/summit_aggregation.py + summit_diagnostics.py** (SPEC15.19 d3 cycle 15.10a Stage C F5) — 9 cross-pipeline summit-position aggregation strategies emit-only diagnostic BED+TSV per pipeline for IGV inspection.` — these modules were DELETED by cycle 15.10a-S2 Stage A (verified `ls onionskin_core/summit_aggregation.py` → no such file). Live tree has `onionskin_core/summit_strategies.py` (~809 lines) implementing curated 68-strategy single-flag menu. Description should be rewritten to reflect the post-15.10a-S2 design.
- [ROADMAP.md:1617](ROADMAP.md#L1617): `## Priority SPEC15.19 — Phase 15 housekeeping bundle (incl. summit-aggregation strategy menu cross-pipeline) ✓ DONE (cycle 15.10a, v0.14.95)` — should reference cycle 15.10a-S2 v0.14.97 redesign as the final state of the SPEC15.19 d3 strategy menu deliverable, OR add a separate ROADMAP entry for cycle 15.10a-S2's redesign + corrigendum.
- **Cycle 15.10a-S1 (v0.14.96) per-pipeline posterior architecture defect-fix is entirely absent from ROADMAP** — should appear as a ROADMAP entry analogous to other supplemental cycles, since it materially changed user-visible posterior output paths cross-pipeline.

Suggested fix scope (in-situ): rewrite `# Phase 15 — ...` section header from `v0.14.95` → `v0.14.97`, update closure paragraph, replace stale module reference + 9-strategy text with curated 68-strategy single-flag menu description, add ROADMAP entries for cycle 15.10a-S1 (v0.14.96) per-pipeline posterior architecture + cycle 15.10a-S2 (v0.14.97) F5 strategy-menu redesign.

**T2. `multi-agent/plans/next/SUMMIT_SOUP.md` Open brainstorm questions 13 + 14 (added 2026-05-05) describe the OBSOLETE 9-strategy menu and reference deleted modules.** Severity: minor.

[multi-agent/plans/next/SUMMIT_SOUP.md:437](multi-agent/plans/next/SUMMIT_SOUP.md#L437): question 13 says `Cycle 15.10a Stage C F5 (SPEC15.19 d3) landed a 9-strategy summit-aggregation menu cross-pipeline (onionskin_core/summit_aggregation.py + summit_diagnostics.py + per-pipeline diagnostic BED+TSV emission at each pipeline's summit-refinement step)`. The 9-strategy menu was reverted at cycle 15.10a-S2; the modules referenced no longer exist; the curated 68-strategy menu replaced them. Question 13's premise no longer holds.

[multi-agent/plans/next/SUMMIT_SOUP.md:439](multi-agent/plans/next/SUMMIT_SOUP.md#L439): question 14 similarly references "9 NEW aggregation strategies" + `summit_diagnostic_runner.py` (deleted by cycle 15.10a-S2 Stage A) for incremental method-capture wiring.

Suggested fix scope (in-situ): either rewrite questions 13 + 14 to reflect the post-15.10a-S2 curated 68-strategy menu state (default-pick decision still relevant — the menu is opt-in; existing-method captures still relevant), OR mark them RESOLVED-by-supersession with pointers to the curated 68-strategy menu in `summit_strategies.py`.

**T3. `cycle 15.10a-S3` references in two surfaces — never-existed cycle.** Severity: trivial.

- [multi-agent/plans/next/SUMMIT_SOUP.md:297](multi-agent/plans/next/SUMMIT_SOUP.md#L297): `**CORRIGENDUM 2026-05-05** (added during cycle 15.10a-S3 R1 audit code dive)` — corrigendum was actually added during cycle 15.10a-S2 R1-audit-equivalent (collapsed orchestrator-Principal exchange).
- [multi-agent/plans/PHASE15_AUDIT_LOG.md:187](multi-agent/plans/PHASE15_AUDIT_LOG.md#L187): `triangle-apex emits NaN on all 3 pipelines pending cycle 15.10a-S3` — should be "pending future SUMMIT_SOUP item realization" (no cycle 15.10a-S3 was ever planned or created).

Suggested fix scope (in-situ): two-line edit each. Replace `cycle 15.10a-S3` → `cycle 15.10a-S2` (SUMMIT_SOUP) and `pending cycle 15.10a-S3` → `pending future SUMMIT_SOUP item realization` (AUDIT_LOG).

**T4. `multi-agent/tracking/KNOWN_ISSUES.md` newest-first sort is partially broken.** Severity: minor.

Cycle 15.10a Stage D F11 (per `PHASE15_SPEC.md` SPEC15.20 d1) claimed a newest-first re-sort. Live state has multiple chronological out-of-order entries:

| Line | Entry | Date | Issue |
|---|---|---|---|
| [395](multi-agent/tracking/KNOWN_ISSUES.md#L395) | `[ISSUE:2026-04-21:2]` | 2026-04-21 | Should be after 2026-04-28 entries below it |
| [600](multi-agent/tracking/KNOWN_ISSUES.md#L600) | `[ISSUE:2026-04-18:2]` | 2026-04-18 | Out of order vs 2026-04-14:1 above it |
| [692](multi-agent/tracking/KNOWN_ISSUES.md#L692) | `[ISSUE:2026-04-26:1]` | 2026-04-26 | Out of order — should be near top (above 2026-04-21) |
| [791](multi-agent/tracking/KNOWN_ISSUES.md#L791) | `[ISSUE:2026-04-24:2]` | 2026-04-24 | Out of order |
| [809](multi-agent/tracking/KNOWN_ISSUES.md#L809) | `[ISSUE:2026-04-28:1]` | 2026-04-28 | Far out of order — should be among 2026-04-30 / 2026-04-29 cluster near top |
| [815](multi-agent/tracking/KNOWN_ISSUES.md#L815) | `[ISSUE:2026-04-28:2]` | 2026-04-28 | Same as above |
| [1176](multi-agent/tracking/KNOWN_ISSUES.md#L1176) | `[ISSUE:2026-05-03:1]` | 2026-05-03 | At BOTTOM of file — should be near top (4th newest after 2026-05-05/04 cluster) |
| [1213](multi-agent/tracking/KNOWN_ISSUES.md#L1213) | `[ISSUE:2026-05-03:2]` | 2026-05-03 | Same as above |
| [1244](multi-agent/tracking/KNOWN_ISSUES.md#L1244) | `[ISSUE:2026-05-03:3]` | 2026-05-03 | Same as above |
| [1276](multi-agent/tracking/KNOWN_ISSUES.md#L1276) | `[ISSUE:2026-05-03:4]` | 2026-05-03 | Same as above |

The 4 `2026-05-03` entries at the END of the file are the most egregious — they post-date the 2026-04-26/24/21/19/18/14 cluster that follows them in the linear file ordering. Cycle 15.10a Stage D F11 was either applied incompletely or these entries were appended afterward without re-sorting.

Suggested fix scope (in-situ): re-sort `KNOWN_ISSUES.md` strictly by `[ISSUE:DATE:N]` newest-first. ~10 entries to reposition. Sectioning preserved (preamble at top, ordering convention notes preserved).

**T5. `multi-agent/tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` IBM-C4 disposition entry is stale at cycle 15.10a-S2 close.** Severity: trivial.

[multi-agent/tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md:55](multi-agent/tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md#L55): `IBM-C4 ... cross-pipeline summit-aggregation menu via SPEC15.19 d3 (cycle 15.10a, v0.14.95)` — the SPEC15.19 d3 strategy menu was redesigned at cycle 15.10a-S2 v0.14.97. Disposition should reference both cycles or only the final 15.10a-S2 v0.14.97.

Suggested fix scope (in-situ): one-line edit. Append `; redesigned via cycle 15.10a-S2 v0.14.97 (curated 68-strategy single-flag menu replacing the reverted 9-strategy 2-flag scaffold)` to the IBM-C4 disposition cell.

### Category 3: Substantive findings warranting a post-wrap-up remediation cycle

**None.** All Category 2 findings are surgical doc/tracking edits suitable for orchestrator in-situ repair. No code/test/runtime defects. No silent narrowing of substantive scope. No deferred-by-stealth work that should have landed in Phase 15.

### Category 4: Future-phase carry-forwards (no Phase 15 action; verify SOUP/KNOWN_ISSUES landing)

**C1. `make test` runtime trending toward developer-experience threshold (~265.94s post-cycle-15.10a-S2 per `[v0.14.97]` CHANGELOG entry).** Properly NOT filed during Phase 15. Recommend orchestrator triage at Phase 15 archive — file a fresh `[ISSUE:2026-05-05:N]` entry capturing the runtime trend + suggested redistribution into a `make test-slow` gate. Existing `[ISSUE:2026-04-21:2]` (line 395, dated 2026-04-21) is older + carries an exit condition of "under 90 seconds" that was already infeasible at filing time; a fresh entry with realistic exit criteria is cleaner than amending the old one.

**C2. `[ISSUE:2026-05-05:3]` PuffStep parity gate fragility (8/28 fail signature on fresh regenerate).** Properly Active in `KNOWN_ISSUES.md` line 51. Cross-referenced by both cycle 15.10a-S1 R3 + cycle 15.10a-S2 orchestrator-as-R3 closeouts as not-a-regression. Landing zone is implicit "post-Phase-15 dedicated gate-hardening cycle"; orchestrator should make the landing zone explicit at Phase 15 archive (reference Phase 16 backlog or similar).

**C3. SUMMIT_SOUP triangle-apex SUMMIT ESTIMATOR (Item 5 expansion 2026-05-05) + PIPELINE_PARITY_SOUP triangle FIT for fork elongation (Item 1).** Both correctly staged with corrigendum context, algorithm specs, and cross-references to each other (verified — neither claims to be the other's implementation; both note distinct concepts). Cycle 15.10a-S2 NaN-emit at runtime aligns with deferral. No Pass 1 action.

**C4. `[ISSUE:2026-04-30:2]` HMM step-number leakage Categories 1 + 3 (engine internal).** Properly Active; Categories 2 + Cat 3 narrow scope completed in Phase 15; remaining surface deferred to `multi-agent/plans/next/CROSS_PIPELINE_UNIFICATION_SOUP.md`. No Pass 1 action.

**C5. Other Active KNOWN_ISSUES entries** (`[ISSUE:2026-05-04:2]` Growth peak-summary redundant paths; `[ISSUE:2026-05-01:1]` is actually marked Resolved v0.14.91; `[ISSUE:2026-04-30:6]` shape-filter window principled cleanup; `[ISSUE:2026-04-29:1]` Phase 15 closeout verify gap-mask migration — see note below; `[ISSUE:2026-04-29:4]`/`:5`/`:7`/`:8`; `[ISSUE:2026-04-28:1]`; `[ISSUE:2026-04-24:1]`/`:2`; `[ISSUE:2026-04-21:1]` superseded-and-reverted; `[ISSUE:2026-04-18:1]`/`:2`/`:3`/`:4`; `[ISSUE:2026-04-19:1]`; `[ISSUE:2026-04-14:1]`; `[ISSUE:2026-05-03:*]`). All correctly Active with realistic landing zones.

**Sub-finding inside C5:** `[ISSUE:2026-04-29:1]` `Phase 15 closeout: verify growth/RMS shared gap-mask migration` is itself a Phase 15 closeout-tracking issue. Pass 1 verification: SPEC15.2 (cycle 15.2a v0.14.78) is the shared pre-pipeline gap-mask deliverable; SPEC15.22 (cycle 15.4b v0.14.82) reconciled gap-analysis architectural placement. Verification work is implicitly done by virtue of those cycles closing. Recommend orchestrator Pass 1 → triage either close this entry as Resolved (v0.14.78 + v0.14.82) or flip to a Phase-15-archive verification task.

## Per-cycle audit summary

(Pass/Fail/Open per cycle's R3 closeout against R1 audit's F-findings; spot-check live code matches R3 claims.)

| Cycle | Status | Notes |
|---|---|---|
| 15.1a (SPEC15.1 v0.14.77) | ✓ PASS | HMM PuffStep re-audit + `--hmm-mu-scale` fix + missing synonyms + `--hmm-emodel` restored. R3 closeout consistent with R1; `make puff-compare` 28/28 baseline established. |
| 15.2a (SPEC15.2+15.3 v0.14.78) | ✓ PASS | Shared pre-pipeline gap mask + HMM parallel child pipeline + step-14 APS bug fix + step renumbering. Step-renumber blast-radius captured in cycle's R3. |
| 15.3a (SPEC15.4 v0.14.79) | ✓ PASS | Bayesian ODW System (`onionskin_core/odw.py`); 4-metric framework; `last_active_stage` rename; `--odw-*` argparse group. `[SURPRISE:15:15.9a:1:A:1]` (later: cycle 15.9a flagged that 15.3a's wiring landed `_DIP_PROB_THRESHOLD` at 0.9 not 0.8) was Resolved at cycle 15.9a v0.14.94 with SPEC-literal asymmetric directional defaults. |
| 15.4a (SPEC15.5+15.6 v0.14.80) | ✓ PASS | HMM shape-score + multistage unification + cross-pipeline reliability + flat-sample classifier. Foundation for the supplemental S1/S2/S3/S4 cycles. |
| 15.4a-S1 (v0.14.81) | ✓ PASS | Sigma-derivation gap (Path B) post-closeout follow-up. |
| 15.4b (SPEC15.22 v0.14.82) | ✓ PASS | Gap-analysis architectural cleanup; HMM placement fix; cross-pipeline filter contract; `_finalize_amplicon_recommendations` ported to RMS + HMM. |
| 15.5a (SPEC15.7+15.8 v0.14.83) | ✓ PASS | Cross-pipeline summit refinement strategy menu (cycle 5 strategies cross-pipeline + 1 HMM-only) + ODW-confined refinement + HMM sliding-offset port + summit↔timing 4-step convergence. R3 surfaced two findings (F-R2-1 RMS step-3 active-stage refinement; F-R2-2 max_rcn_stage parity) addressed in-cycle via R2-J. |
| 15.6a (SPEC15.9+15.10+15.11+15.12 v0.14.84) | ✓ PASS | APS area-excess-floor + column rename + composite multi-feature clustering modes. Stage A-C only; SPEC15.12 deferred to cycle 15.6a-S1. |
| 15.6b (SPEC15.23 v0.14.86) | ✓ PASS | Strategy-selection determinism diagnostic + fix; bare `except Exception` narrowing; canonical-grid invariant audit + assertions. `tests/test_determinism.py` regression-locked. |
| 15.6a-S1 (SPEC15.9d1+SPEC15.12 v0.14.90) | ✓ PASS | Stage D follow-up to 15.6a; `--aps-aggregation-{mode,score,score-min}` flag matrix + `--aps-emit-unfiltered-aggregates` side-car; `--aps-weight-loci` retired; HMM trajectory clustering refactored. |
| 15.4a-S2 (v0.14.87) | ✓ PASS | Flat-detector + amplicon reliability tuning + `[ISSUE:2026-04-30:4]` HMM-posterior unit fix. Stage B.1/B.4/B.5/B.6/B.7/B.8 all closed. R3 verified. |
| 15.4a-S3 (v0.14.88) | ✓ PASS | Flat-sample sigma-source dispatch architecture; `--flat-sample-sigma-source` flag with 4 values; default flipped to `chrom-mad` (placeholder for cycle 15.4a-S4 mask-derived). |
| 15.4a-S4 (v0.14.89) | ✓ PASS | Pre-pipeline shared naive background mask + per-sample chrom-MAD bedGraph emission cross-pipeline; default flipped to `chrom-background-mad`. |
| 15.7a (SPEC15.13+15.14+15.15+15.16 v0.14.91) | ✓ PASS | SAPS implementation + `--hmm-statepath-base` + label-dictionary architecture + JSON sidecars + within-prior second-pass cross-pipeline + `--norm-mode chrom-median` HMM default flip. (Note: SPEC15.16 partial reversal absorbed into cycle 15.10a Stage A.) |
| 15.7b (SPEC15.24 v0.14.92) | ✓ PASS | HMM per-stage parabola summit emission for cross-pipeline parity. |
| 15.8a (SPEC15.17+15.18 v0.14.93) | ✓ PASS | APS analysis-surface parity + master APS catalog + step-less plots/notebooks + HMM plot ports + `--peak-summary` extension to RMS+HMM + max-projection variants. |
| 15.9a (SPEC15.21 v0.14.94) | ✓ PASS | Architectural cleanup — `timing_diagnostics.py` module + ODW per-direction threshold split + Gap-analysis argparse group + Growth engine argparse elimination + posterior second-pass controls. |
| 15.10a (SPEC15.19+15.20 v0.14.95) | ✓ PASS (R2 narrowed F5; reverted by 15.10a-S2) | Stage A SPEC15.16 partial reversal + Stage B mechanical CLI cleanup + Stage C 9-strategy 2-flag F5 (later REVERTED by 15.10a-S2) + Stage D closeout sweep. |
| 15.10a-S1 (v0.14.96) | ✓ PASS | Multi-pipeline controller posterior architecture defect-fix; per-pipeline manifests; grouping-level paths dropped; HMM `shutil.copy2` hack removed. |
| 15.10a-S2 (v0.14.97) | ✓ PASS (orchestrator-as-R3 cycle-final exception verified) | F5 strategy-menu redesign + revert. Curated 68-strategy single-flag menu live cross-pipeline. CORRIGENDUM 2026-05-05 implemented in code at [summit_strategies.py:578-585](onionskin_core/summit_strategies.py#L578-L585) with explicit corrigendum-aligned NaN-emit for triangle-apex × all pipelines. F1–F8 all verified clean by orchestrator-as-R3 + spot-check by Pass 1 fresh-eyes. Diagnostic emissions verified at runtime: Growth + RMS + HMM all emit `<pipeline>_diagnostic_summits.{bed,tsv}` at the documented paths. |

## SPEC priorities verification (SPEC15.1 — SPEC15.24)

| Priority | DONE marker (SPEC) | DONE marker (ROADMAP) | Code state spot-check | Status |
|---|---|---|---|---|
| SPEC15.1 | DONE v0.14.77 | DONE v0.14.77 | `--hmm-mu-scale`/`--hmm-emodel`/synonyms in onionskin.py | ✓ PASS |
| SPEC15.2 | DONE v0.14.78 | DONE v0.14.78 | shared pre-pipeline gap mask in controller | ✓ PASS |
| SPEC15.3 | DONE v0.14.78 | DONE v0.14.79 | HMM parallel child pipeline; step-14 APS fix | ✓ PASS (ROADMAP version differs slightly from SPEC; both refer to cycle 15.2a/15.3a sequence; non-issue) |
| SPEC15.4 | DONE v0.14.79 | DONE v0.14.79 | `onionskin_core/odw.py`; `--odw-*` group | ✓ PASS |
| SPEC15.5 | DONE v0.14.80 | DONE v0.14.80 | HMM shape-score + multistage unification | ✓ PASS |
| SPEC15.6 | DONE v0.14.80–v0.14.89 | DONE v0.14.80–v0.14.89 | flat_sample.py + reliability + classifier | ✓ PASS |
| SPEC15.7 | DONE v0.14.83 | DONE v0.14.83 | summit_convergence.py + summit_strategies.py | ✓ PASS |
| SPEC15.8 | DONE v0.14.83 | DONE v0.14.83 | summit↔timing 4-step convergence | ✓ PASS |
| SPEC15.9 | DONE v0.14.84/v0.14.90 | DONE v0.14.84 | aps.py post-sum-floor variant | ✓ PASS |
| SPEC15.10 | DONE v0.14.84 | DONE v0.14.84 | summit_excess→summit_rcn rename | ✓ PASS |
| SPEC15.11 | DONE v0.14.84 | DONE v0.14.84 | composite multi-feature clustering | ✓ PASS |
| SPEC15.12 | DONE v0.14.90 | DONE v0.14.90 | --aps-aggregation-* flag matrix; HMM trajectory clustering | ✓ PASS |
| SPEC15.13 | DONE v0.14.91 | DONE v0.14.91 | onionskin_core/hmm_saps.py | ✓ PASS |
| SPEC15.14 | DONE v0.14.91 | DONE v0.14.91 | --hmm-statepath-base + state_path_io.py | ✓ PASS |
| SPEC15.15 | DONE v0.14.91 | DONE v0.14.91 | within-prior pass-2 cross-pipeline | ✓ PASS |
| SPEC15.16 | DONE v0.14.91 + reversed v0.14.95 | DONE v0.14.91 + partial reversal v0.14.95 | HMM --norm-mode default = ref-stage | ✓ PASS |
| SPEC15.17 | DONE v0.14.93 | DONE v0.14.93 | APS_CATALOG.md + HMM plot ports | ✓ PASS |
| SPEC15.18 | DONE v0.14.93 | DONE v0.14.93 | --peak-summary extended cross-pipeline | ✓ PASS |
| SPEC15.19 | DONE v0.14.95 (cycle 15.10a) | DONE v0.14.95 | Stage A reversal + Stage B/C; Stage C F5 9-strategy was REVERTED by 15.10a-S2 (curated 68-strategy is the final state) | ◑ ROADMAP STALE per T1; SPEC15.19 final state is curated 68-strategy via cycle 15.10a-S2 v0.14.97 |
| SPEC15.20 | DONE v0.14.95 (cycle 15.10a Stage D F11) | DONE v0.14.95 | tracking-file cleanup + CLUSTERING_DEFAULTS.md + cross-pipeline-parity audit checklist | ◑ KNOWN_ISSUES sort imperfect per T4 |
| SPEC15.21 | DONE v0.14.94 | DONE v0.14.94 | timing_diagnostics.py + ODW per-direction split + Gap-analysis group + Growth engine argparse elimination | ✓ PASS |
| SPEC15.22 | DONE v0.14.82 | DONE v0.14.82 | gap-analysis HMM placement + cross-pipeline filter contract | ✓ PASS |
| SPEC15.23 | DONE v0.14.86 | DONE v0.14.86 | strategy-selection determinism diagnostic + fix | ✓ PASS |
| SPEC15.24 | DONE v0.14.92 | DONE v0.14.92 | HMM per-stage parabola summit emission | ✓ PASS |

24/24 priorities have DONE markers at both SPEC and ROADMAP level. SPEC15.19 + SPEC15.20 are the two with downstream tracking drift (per T1 + T4). All 24 priorities have substantive code state matching their deliverables.

## Cross-pipeline parity audit

Lens applied per `feedback_cross_pipeline_parity_blindspot.md` (project intent: "the only true difference between pipelines is how they call amplicons; almost everything thereafter is game for each pipeline at least by analogy"). Verified by spot-checking all 3 pipelines emit at the documented paths for each cross-pipeline deliverable.

| Concept | Growth | RMS | HMM | Status |
|---|---|---|---|---|
| Bayesian ODW System (SPEC15.4) | ✓ | ✓ | ✓ | Cross-pipeline; `onionskin_core/odw.py` shared primitive |
| Reliability scoring + flat-sample classifier (SPEC15.6) | ✓ | ✓ | ✓ | Cross-pipeline; `flat_sample.py` shared |
| Per-sample chrom-MAD bedGraph emission (cycle 15.4a-S4) | ✓ | ✓ | ✓ | Cross-pipeline; `<pipeline>/<aps-step>/samples/` |
| Pre-pipeline shared naive background mask (cycle 15.4a-S4) | ✓ | ✓ | ✓ | Computed once at controller before pipelines start |
| Gap-analysis architectural placement (SPEC15.22) | ✓ | ✓ | ✓ | HMM placement fixed; cross-pipeline filter contract |
| Cross-pipeline summit refinement strategy menu (SPEC15.7) | ✓ | ✓ | ✓ | `--summit-stage-selection` curated 68-ID menu (cycle 15.10a-S2 final state); HMM-only `odw-narrowest-state-*` selectors documented as intentional asymmetry |
| Summit↔timing 4-step convergence (SPEC15.8) | ✓ | ✓ | ✓ | Cross-pipeline; `summit_convergence.py` shared |
| HMM per-stage parabola summit emission (SPEC15.24) | ✓ (existing) | ✓ (existing post-15.5a) | ✓ (added 15.7b) | Parity reached |
| APS column rename (SPEC15.10) | ✓ | ✓ | ✓ | summit_excess→summit_rcn; mean_excess→mean_rcn |
| Composite APS clustering modes (SPEC15.11) | ✓ | ✓ | ✓ | Cross-pipeline shared infrastructure |
| APS aggregation flag matrix (SPEC15.12) | ✓ | ✓ | ✓ | Cross-pipeline parity verified per cycle 15.6a-S1 F8 table |
| Within-prior pass-2 cross-pipeline (SPEC15.15) | ✓ | ✓ | ✓ | Cross-pipeline; pass-2 archive + sentinel pattern shared. Intentional Growth asymmetry: retains "bins > HI" exclusion in Pass-2 (locked design per cycle 15.4a-S3 corrections appendix) — documented + cross-referenced |
| APS analysis-surface parity (SPEC15.17) | ✓ | ✓ | ✓ | Cross-pipeline; HMM gained `aps_amplicon_importance.tsv` + summit refinement plots + profile/genome-overview plots + shape-filter diagnostic plots + QC plots. Overlap-resolution plots correctly remain Growth-only (intentional per pipeline-specific concept) |
| `--peak-summary` extension (SPEC15.18) | ✓ | ✓ | ✓ | Cross-pipeline + max-projection variants |
| Plot/notebook step-less directory layout (SPEC15.17) | ✓ | ✓ | ✓ | Cross-pipeline structural change |
| HMM CLI surface — statepath-base (SPEC15.14) | n/a | n/a | ✓ | HMM-specific; canonical headers + JSON sidecars cross-pipeline patterns |
| Per-pipeline posterior architecture (cycle 15.10a-S1) | ✓ | ✓ | ✓ | Per-pipeline manifests + cluster maps + hires manifests; grouping-level paths dropped |
| Diagnostic-summits BED/TSV emission (cycle 15.10a-S2) | ✓ | ✓ | ✓ | All 3 pipelines emit `<pipeline>_diagnostic_summits.{bed,tsv}` at runtime (verified via `dev/runs/toy_out/` inspection); 68 rows per amplicon per pipeline |
| Triangle-apex strategies (cycle 15.10a-S2 corrigendum) | NaN | NaN | NaN | Intentional per CORRIGENDUM 2026-05-05; deferred to SUMMIT_SOUP item realization. Code at `summit_strategies.py:578-585`. |
| Triangle FIT for fork-elongation (Growth-only) | ✓ | ✗ | ✗ | Intentional cross-pipeline parity gap; filed at `PIPELINE_PARITY_SOUP.md` Item 1 (a separate concept from triangle-apex SUMMIT ESTIMATOR per corrigendum) |

**Verdict: Cross-pipeline parity is clean.** No "X-pipeline-only" defects identified at Pass 1. Intentional asymmetries are all documented + cross-referenced.

## Test suite hygiene assessment

`make test` runtime per cycle 15.10a-S2 closeout: ~265.94s (276 passed / 1 skipped). Ramp from cycle-15.10a baseline of ~250s. Crossing the developer-experience threshold defined as "comfortable inner-loop iteration cadence" — at ~5 min per pass, agents iterating at the gate boundary spend material time waiting.

**Recommendation for orchestrator triage (NOT a Pass 1 action):**
- File a fresh `[ISSUE:2026-05-05:N]` entry in `KNOWN_ISSUES.md` (after the T4 re-sort lands) capturing the runtime trend + suggested redistribution: move slow integration tests to a `make test-slow` gate; audit recently-added tests for unit-vs-integration appropriateness; consider parallelization via pytest-xdist if not already enabled.
- The existing `[ISSUE:2026-04-21:2]` (line 395, dated 2026-04-21) `make test` runtime is slow entry has a "under 90 seconds" exit condition that was infeasible at filing time + is more infeasible now. Either close it as superseded-by-the-new-entry, or amend its exit condition to a realistic threshold + landing zone.

`make puff-compare` `[ISSUE:2026-05-05:3]`: 8/28 fail signature pre-existing per cycle 15.10a R3. Not a regression introduced by Phase 15 cycles. Properly Active in `KNOWN_ISSUES.md`. Landing zone implicit (post-Phase-15 dedicated gate-hardening cycle); recommend orchestrator make landing zone explicit at Phase 15 archive.

## Phase charter vs deliverables

Phase 15's stated theme: **HMM completeness + cross-pipeline enrichment**.

**HMM completeness items — all landed:**
- HMM parallel child pipeline (SPEC15.3 cycle 15.2a/15.3a)
- HMM step-14 APS fix bundled
- SAPS implementation (SPEC15.13 cycle 15.7a)
- `--hmm-statepath-base` + label dictionary + JSON sidecars (SPEC15.14 cycle 15.7a)
- HMM summit refinement (SPEC15.7 cycle 15.5a) + per-stage parabola emission (SPEC15.24 cycle 15.7b)
- HMM posterior anchoring + posterior background-region inheritance (SPEC15.6 + SPEC15.15 cycles 15.4a/15.4a-S2/15.7a)
- HMM CLI correctness audit (SPEC15.1 cycle 15.1a)
- HMM `--norm-mode chrom-median` default flip (SPEC15.16 cycle 15.7a) + partial reversal (cycle 15.10a Stage A)
- HMM trajectory clustering refactor (SPEC15.12 cycle 15.6a-S1)

**Cross-pipeline enrichment items — all landed:**
- Bayesian ODW System cross-pipeline (SPEC15.4 cycle 15.4a)
- Cross-pipeline reliability scoring + flat-sample classifier (SPEC15.6 cycle 15.4a + S1/S2/S3/S4)
- Cross-pipeline summit refinement strategy menu (SPEC15.7 cycle 15.5a; final form cycle 15.10a-S2)
- Cross-pipeline summit↔timing convergence (SPEC15.8 cycle 15.5a)
- Cross-pipeline gap-analysis architectural cleanup (SPEC15.22 cycle 15.4b)
- APS analysis-surface parity + master APS catalog (SPEC15.17 cycle 15.8a)
- `--peak-summary` extension to RMS + HMM + max-projection variants (SPEC15.18 cycle 15.8a)
- Cross-pipeline plot ports to HMM (SPEC15.17 expansion 2026-04-30)
- Cross-pipeline-parity audit checklist (SPEC15.20 cycle 15.10a Stage D)
- Per-pipeline posterior architecture (cycle 15.10a-S1)
- Per-sample chrom-MAD bedGraph emission cross-pipeline (cycle 15.4a-S4)

**Charter-aligned closeout work — all landed:**
- Architectural cleanup (SPEC15.21 cycle 15.9a)
- Phase 15 housekeeping bundle (SPEC15.19 cycle 15.10a)
- Phase 15 closeout sweep (SPEC15.20 cycle 15.10a Stage D — partial drift per T4 but substantively complete)

**Anything intended but missed:** No. All 24 priorities + all amendment-driven scope expansions landed. IBM-C2/C3/C4/C5/C6C/C6D/C7B/C8A/C8B/C12 RESOLVED; IBM-C14 PARTIALLY-RESOLVED with explicit deferral landing zone. BRAINSTORM RESOLVED markers consistent with cycle/version closures.

**Verdict: phase deliverables match charter; no scope shortfall.**

## Recommendations for orchestrator (Pass 1 → triage)

1. **In-situ ROADMAP fix (T1)** — rewrite Phase 15 section header v0.14.95 → v0.14.97; update closure paragraph to reference cycles 15.10a-S1 + 15.10a-S2; replace stale `summit_aggregation.py + summit_diagnostics.py` description with curated 68-strategy menu in `summit_strategies.py`; add ROADMAP entries for cycle 15.10a-S1 (v0.14.96) per-pipeline posterior architecture defect-fix and cycle 15.10a-S2 (v0.14.97) F5 strategy-menu redesign + corrigendum.
2. **In-situ SUMMIT_SOUP cleanup (T2)** — rewrite Open brainstorm questions 13 + 14 to reflect post-15.10a-S2 curated 68-strategy menu state, OR mark them RESOLVED-by-supersession with pointers to `summit_strategies.py`.
3. **In-situ cycle 15.10a-S3 → cycle 15.10a-S2 corrigendum-source cleanup (T3)** — two-line edit: SUMMIT_SOUP.md:297 + PHASE15_AUDIT_LOG.md:187.
4. **In-situ KNOWN_ISSUES.md re-sort (T4)** — strict newest-first by `[ISSUE:DATE:N]`; ~10 entries to reposition; the 4 `[ISSUE:2026-05-03:*]` entries at the bottom are the highest-priority repositions.
5. **In-situ IBM-C4 disposition update (T5)** — append cycle 15.10a-S2 v0.14.97 reference to the disposition cell.
6. **Consider closing `[ISSUE:2026-04-29:1]`** — Phase 15 closeout-tracking issue for shared gap-mask migration was implicitly delivered by SPEC15.2 (v0.14.78) + SPEC15.22 (v0.14.82). Either flip to Resolved or convert to a Phase 15 archive verification task.
7. **File new `[ISSUE:2026-05-05:N]`** for `make test` runtime trend (after T4 re-sort lands) with realistic exit criteria + landing zone.
8. **Make landing zone explicit** for `[ISSUE:2026-05-05:3]` PuffStep parity gate fragility (post-Phase-15 dedicated gate-hardening cycle).

Recommendation 1 is the largest in-situ surface (~30-50 line ROADMAP edits). Recommendations 2-8 are surgical edits. Estimated total orchestrator-triage scope: **moderate** (~1-2 hours of focused ROADMAP + tracking-file cleanup).

## Pass 2 readiness criteria

Pass 2 fires fresh after orchestrator-driven remediation lands. Pass 2 verifies:

- Recommendations 1-5 (in-situ ROADMAP + SUMMIT_SOUP + AUDIT_LOG + KNOWN_ISSUES + IBM tracking-doc cleanup) are landed cleanly.
- Recommendation 6 (`[ISSUE:2026-04-29:1]` disposition) is decided + applied.
- Recommendation 7 (new `make test` runtime KNOWN_ISSUES entry) is filed if accepted.
- Recommendation 8 (PuffStep parity gate landing zone explicit) is recorded.
- No new drift introduced by remediation.
- Working tree clean; remediation committed.

After Pass 2 verifies, Pass 2 writes the formal Phase 15 close + triggers the orchestrator-driven phase archive operation per `feedback_archive_operations_orchestrator_only.md` (orchestrator runs the `git mv` for SPEC + AUDIT_LOG + STRATEGY + SURPRISE_LOG + BACKGROUND + CREDIBILITY + this Pass 1 + Pass 2 reports → `multi-agent/plans/archived/<YYYYMMDD>-PHASE15_*.md`).
