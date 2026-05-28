# Phase 14 Supplemental — Strategy (plan of attack)

**Generated:** 2026-04-24 ~00:45 EST
**Author:** Claude Code 2.1.104 (claude-opus-4-7)
**SPEC revision:** PHASE14_SUPPLEMENTAL-SPEC.md @ v0.14.64 (2026-04-24)
**Phase title:** Phase 14 Supplemental — CLI/parser/help architecture closeout

## Phase summary

25 active priorities remain (3 closed in Phase 0). The SPEC is organized into 5 execution phases with hard dependencies: Phase 1 (structural parser changes, 10 priorities) gates Phase 3 (help-string polish, 7) and Phase 4 (polish + tooling, 2) on its final CLI surface; Phase 2 (audit-only, 5) is independent; Phase 5 (closeout, 1 active) gates on everything else. Dominant themes: flag renames, Universal-tier promotions, runtime wiring of placeholder flags, help-string quality uplift, closeout doc sweep with tracking-directory reorg. Mix: ~40% mechanical (Phase 1a, parts of Phase 4), ~50% substantive runtime/semantic work (Phase 1b, Phase 3b, Phase 5), ~10% pure audit. Main risk: Phase 1b's SPEC-mandated pairing of S27+S28 (Universal promotion + column renames with honest mixed-semantics labels from Q39) — auditor error here propagates to Phase 3b S8 global help pass and to Phase 5 doc sweep.

## Cycle table

| Cycle index | Cycle name | Priorities | R1 (audit) | R2 (implement) | R3 (closeout) | Notes |
|---|---|---|---|---|---|---|
| **1** | **14S.1a** | S10, S4, S5, S29, S30 | Primary: Claude Code — Sonnet<br>Alt: Copilot — Claude Sonnet; or Copilot + GPT-5.5 High | Primary: Copilot + GPT-5.5 Medium<br>Alt: Codex GPT-5.5 Medium; or Copilot + GPT-5.5 High | Primary: Claude Code — Sonnet (different session)<br>Alt: Copilot — Claude Sonnet | 5 mechanical parser changes + 1 default flip. **Skip-reaudit eligible** if R2 declares criteria met. S10 removes standalone engine CLIs — R2 must also delete/convert associated help-regression tests. |
| **2** | **14S.1b** | S27+S28 (paired), S22, S23, S26 | Primary: Claude Code — **Opus**<br>Alt: Copilot — Claude Opus; or Gemini 3.1 Pro Preview | Primary: Codex GPT-5.5 **High**<br>Alt: Copilot + GPT-5.5 High | Primary: Claude Code — **Opus** (different session)<br>Alt: Copilot — Claude Opus | Substantive runtime/semantic. **S27+S28 MUST land in the same R2 round** (SPEC directive). Full re-audit; no skip-reaudit. |
| **3** | **14S.2a** | S12, S25, S14, S15, S21 | Primary: Claude Code — Sonnet<br>Alt: Copilot — Claude Sonnet | Primary: Copilot + GPT-5.5 Medium<br>Alt: Codex GPT-5.5 Medium | **Defer to 14S.3a R3 (index 4)** | Pure audit-only; no runtime code. S12 + S25 audit tables already produced 2026-04-23. Remaining R2 work: produce `STAGE-1-2-DETECTION-PASS-TERMINOLOGY-AUDIT.md`, append HMM PuffStep audit + `--hmm-0-based-statepath` audit to `PHASE15_BRAINSTORM.md`. Cycle closes via Template H lightweight closeout by R1. |
| **4** | **14S.3a** | S6, S7, S9, S13, S19, S20 | Primary: Claude Code — Sonnet<br>Alt: Copilot — Claude Sonnet | Primary: Copilot + GPT-5.5 Medium<br>Alt: Codex GPT-5.5 Medium | Primary: Claude Code — Sonnet (different session)<br>Alt: Copilot — Claude Sonnet | 6 targeted help-text passes. S7 depends on S26 (closed in index 2 / 14S.1b). S20 depends on S22 (closed in index 2 / 14S.1b). **R3 also verifies 14S.2a (index 3) deferred deliverables** (4 audit/tracking files exist, cover stated scope, wire correctly with 3a help-string direction). |
| **5** | **14S.3b** | S8 solo | Primary: Claude Code — **Opus**<br>Alt: Gemini 3.1 Pro Preview; or Copilot — Claude Opus | Primary: Codex GPT-5.5 **High**<br>Alt: Copilot + GPT-5.5 High | Primary: Claude Code — **Opus** (different session)<br>Alt: Gemini 3.1 Pro Preview | Broadest single priority in the phase. Touches **every parser group**. Integrates S25 help-string direction, S28 summit-as-origin-proxy language, multi-pipeline step mentions. |
| **6** | **14S.4a** | S11, S24 | Primary: Claude Code — Sonnet<br>Alt: Copilot — Claude Sonnet | Primary: Copilot + GPT-5.5 Medium<br>Alt: Codex GPT-5.5 Medium | Primary: Claude Code — Sonnet<br>Alt: Copilot — Claude Sonnet | S11 = mechanical reorder of `add_argument` calls per pipeline step order. S24 = new standalone `scripts/validate_onionskin_flags.py` + `make validate-flags` + README/HANDOFF mentions. Independent; bundled for cycle efficiency. |
| **7** | **14S.5a** | S16 | Primary: Claude Code — **Opus**<br>Alt: Gemini 3.1 Pro Preview; or Copilot — Claude Opus | Primary: Codex GPT-5.5 **High**<br>Alt: Copilot + GPT-5.5 High | Primary: Claude Code — **Opus** (different session)<br>Alt: Gemini 3.1 Pro Preview | Closeout doc sweep. `git mv BRAINSTORM.md KNOWN_ISSUES.md → tracking/`. Reference propagation across 4 agent files + AGENT_CONVENTIONS + workflows + full_instructions + audits + live plan files in one atomic commit. Inline 14-S18 drift audit bundled here. Depends on all prior cycles CLOSED. |

## Deviation rationale

- **14S.1b R1/R3 = Opus (vs. baseline Sonnet):** S28's column renames use honest mixed-semantics labels per Q39's band-aid path — e.g., dropping `_ci_` from `final_summit_low/high/width_bp` because the interval is no longer purely bootstrap-CI, but keeping `argmax_mean_bootstrap_sd_bp` because that SD really is argmax-based. Sonnet tends to accept internally-consistent drift without catching this nuance. Cost of missing it: bad column schema propagates into Phase 3b S8 help strings and Phase 5 doc sweep.
- **14S.1b R2 = Codex High (vs. baseline Copilot Medium):** S27+S28 mandatory pairing + S22 wiring across all 3 pipelines + S23 flag split + S26 placeholder-to-wired transition is a large coordinated R2 round. High reasoning pays for itself.
- **14S.2a R3 deferred to 14S.3a R3:** Cycle 2a is pure audit-only with no runtime code changes; running a standalone R3 session would be overkill. Absorbed into 14S.3a's R3 closeout per the Deferred-R3 rule in `spec_plan_three_role_audit_loop-v2.md` § Cycle Granularity. Cycle 2a still closes with its own CHANGELOG/DEVLOG entry via Template H lightweight closeout by R1.
- **14S.3b R1/R3 = Opus (vs. baseline Sonnet):** S8 is a full-parser-surface pass across every group, integrating S25's help-string direction and S28's terminology decisions. Semantic integration across ~9 parser groups exceeds Sonnet's comfortable audit scope.
- **14S.3b R2 = Codex High:** Broadest rewrite in the phase; high reasoning reduces drift risk.
- **14S.5a R1/R3 = Opus:** File moves + cross-file reference propagation in a single atomic commit across ~10 files. An audit miss here leaves stale pointers across the dev system that agents will re-propagate for months.
- **14S.5a R2 = Codex High:** Atomic multi-file commit with AGENT_CONVENTIONS updates demands precision, not speed.

## Execution summary

7 execution cycles + 1 Final Overseer pass, covering 22 substantive-or-mechanical priorities across 25 total (S17 folded into S16, S18 handled inline within any agent-file touch). **3 Opus-tier upgrades** (1b, 3b, 5a) on Role 1 and Role 3; all others baseline Sonnet-tier. **1 skip-reaudit-eligible cycle** (1a, if R2 declares) and **1 deferred-R3 cycle** (2a → 3a). Final Overseer fires after 5a closes and before archival. Known risks: (a) auditor drift on S28 column renames if R1 downgrades to Sonnet against the Opus recommendation; (b) 1b is a genuinely large Role 2 round — if Codex High struggles, split into 1b.i (S27+S28) and 1b.ii (S22, S23, S26) as a mid-phase amendment; (c) 5a's atomic-commit requirement is unforgiving — any intermediate write will fragment the reference-propagation audit trail.

## Final Overseer

- **Primary:** Gemini 3.1 Pro Preview
- **Alt:** Codex GPT-5.5 High reasoning; or Gemini 2.5 Pro
- **Trigger:** after cycle **14S.5a** closes, before SPEC + AUDIT_LOG + STRATEGY archival to `multi-agent/plans/archived/`
- **Scope:** full Phase 14 Supplemental audit across live code + docs + agent files; cross-cycle drift check (did 3b's help rewrites contradict 1b's flag semantics? did 5a's tracking-dir move leave orphaned references?); comparison against `multi-agent/tracking/KNOWN_ISSUES.md` (post-move) and `multi-agent/tracking/BRAINSTORM.md` (post-move); verify `INTENDED-BUT-MISSED-PRIOR-TO-14.md` items routed correctly. Any actionable findings → post-wrap-up triage (Template E) with R1 = Claude Code — Opus.

### Correctness advisories for Final Overseer (added 2026-04-26, cycle 14S.4a re-audit)

The following HMM engine help-string and code correctness issues were found during the
14S.4a audit discussion. Items 1–2 were corrected within the cycle; item 3 is deferred
to Phase 15 and flagged here for the Final Overseer to verify the deferral was properly
recorded and that no downstream help strings contradict the intended correction.

1. **`--hmm-pseudo` step tag was wrong** (`[hmm: 01-mednorm]` → corrected to
   `[hmm: 04-chrom-renorm]`). Code trace shows `pseudo` is not passed to
   `_step1_mednorm` at all; it is only used at step 4 (`_step4_rcn_norm` /
   `_step4_chr_median_norm`). Computationally active only in ref-stage (ratio) mode;
   in chrom-median mode, `pseudo` is embedded in output filenames only. Help text body
   corrected to reflect step 04 and mode-conditionality. **Fixed in v0.14.73.**

2. **`--hmm-pseudo` ordering in HMM group** — original audit placed `--hmm-pseudo` at
   position 2 and `--hmm-norm-mode` at position 3 (incorrect since `--hmm-pseudo` is
   step-04, not step-01). Corrected to: `--hmm-norm-mode` [01+04] at position 2,
   `--hmm-pseudo` [04] at position 3. **Fixed in v0.14.73.**

3. **`--hmm-mu-scale` behavior inverted from PuffStep (DEFERRED — Phase 15).**
   In PuffStep, `--mu_scale` derived sigmas from means (`sigma = mean × mu_scale`);
   means were never touched. In onionskin, the translation reversed this: `--hmm-mu-scale`
   scales the means (`mean = mean × mu_scale`) and never touches sigmas. The help text
   claim "Also rescales the sigmas by the same factor" is therefore wrong. The
   `--hmm-emission-sigmas` help text cross-reference ("use `--hmm-mu-scale 0.5` sets
   sigma = 0.5 × mean") is also wrong. Additionally, because `--hmm-emission-sigmas`
   always has a hardcoded default in onionskin, a corrected implementation would need
   to resolve priority logic (explicit sigmas vs. `mu_scale`-derived sigmas).
   **Fix scope and design: Phase 15 — see `PHASE15_BRAINSTORM.md` §
   "CORRECTNESS BUG: --hmm-mu-scale translation from PuffStep is wrong".**
   The Final Overseer should verify this bug is explicitly captured in the Phase 15
   planning surface and that no other Phase 14 Supplemental work accidentally
   depended on the incorrect `mu_scale` behavior.

## Amendment log

*No amendments yet.*
