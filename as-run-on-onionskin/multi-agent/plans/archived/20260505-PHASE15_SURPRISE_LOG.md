# PHASE 15 SURPRISE LOG

Supplemental to `multi-agent/plans/PHASE15_AUDIT_LOG.md`. Captures fresh-eyes observations
that an agent noticed during a phase activity but deliberately did NOT include in the
formal audit / implementation / closeout report because the observation was either
(a) out of the cycle's scope, (b) a hygiene drift adjacent to the work but not load-bearing
for the cycle's deliverables, (c) an opinion or "smell" rather than a concrete finding,
or (d) a cross-cycle pattern that doesn't belong inside any single cycle's section.

## Why this file exists

The audit-implement-reaudit loop in
`multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md` is disciplined about scope
— each cycle's audit log entries answer concrete questions about that cycle's SPEC
deliverables. The discipline keeps the audit log focused, but it also creates a
filtering pressure: anything an agent notices that is "out of scope" is implicitly
asked to either be silently dropped or be filed in `multi-agent/tracking/KNOWN_ISSUES.md`
or `multi-agent/tracking/BRAINSTORM.md`. Both of those targets have their own scope
constraints (KNOWN_ISSUES is for actionable near-term issues; BRAINSTORM is for
speculative future-direction ideas). Neither cleanly accommodates the *peripheral
vision* category: things an agent saw that seemed off but did not justify a
finding-with-repair-contract right now.

This file is that home. It exists so that the user can ask agents *"did you notice
anything off that did not make it into your formal report?"* and get a written record
back, accumulated across cycles + phases + roles. It accepts:

- **Adjacent drift** — code, docs, or planning-surface inconsistencies that fell
  next to the work but did not block it.
- **Cross-cycle patterns** — observations that only become visible when standing back
  from any single cycle's narrow lens.
- **Smells without concrete repair** — things that look wrong but where the agent
  cannot articulate the right fix without more user input.
- **Opinions on drift from user intent** — fresh-eyes assessments of whether the
  project's evolving artifacts (SPEC, code, planning surfaces) still match what the
  user appears to want from the program. Speculative; wrong assessments are expected
  and fine.
- **Heads-up notes for future cycles** — things the agent thinks the next cycle's
  auditor / implementer should look for, even though the agent did not make it a
  formal finding.

## Conventions

### File naming + scope

This file is **per-phase** under the same convention as `PHASE<N>_SPEC.md`,
`PHASE<N>_AUDIT_LOG.md`, `PHASE<N>_STRATEGY.md`, and `PHASE<N>_FEEDBACK.md`.
Future phases get their own `PHASE<N>_SURPRISE_LOG.md`. At phase archive
(post-Final-Overseer per workflow v2 § Step 8), this file archives alongside
its siblings into `multi-agent/plans/archived/`.

If an entry transcends the phase (e.g., a cross-phase pattern observation), it
still lives in the per-phase file of the phase whose work surfaced it. Future
cross-phase synthesis happens by reading multiple `PHASE<N>_SURPRISE_LOG.md`
files in chronological order.

### Entry ID format

Every entry gets a stable ID of the form:

```
[SURPRISE:<N>:<C>:<R>:<T>:<X>]
```

where:
- **N** — Phase number (e.g., `15`).
- **C** — Cycle name (e.g., `15.6a`, `15.4b`, `wrap-up` for Role 3 wrap-up audits).
  Use the same cycle name that appears in the cycle's `## Cycle:` heading in
  `PHASE<N>_AUDIT_LOG.md` so the cross-reference is mechanical.
- **R** — Role number — `1` for Role 1 (auditor), `2` for Role 2 (implementer),
  `3` for Role 3 (final overseer / wrap-up).
- **T** — Template letter from the workflow file's Prompt Templates section
  (`A` initial audit, `B` implementation, `C` re-audit, `D` final wrap-up,
  `E` post-wrap-up triage, `F` post-wrap-up implementation, `G` final closeout
  after wrap-up remediation, `H` cycle closeout after skip-reaudit, `I`
  strategist).
- **X** — index that starts at `1` and increments by 1 for each new entry in
  the same `(N, C, R, T)` tuple. Each role+template invocation gets its own
  fresh `X=1`, so a single agent contributing 4 entries during one Role 1
  Template A initial audit produces `X=1, 2, 3, 4`.

The ID lets a reader trace back to exactly which cycle round produced the
observation, which is useful for sanity-checking the perspective (an
implementer's surprise during Stage B has a different epistemic position
from an auditor's surprise at initial-audit time).

### Entry format

Each entry is a Markdown subsection with the following structure (modeled
loosely on `multi-agent/tracking/KNOWN_ISSUES.md`):

```markdown
## [SURPRISE:N:C:R:T:X] <natural-language title>

**Authors:** <consolidated authorship per AGENT_CONVENTIONS authorship section>

**Status:** <Active | Resolved (vX.Y.ZZ) | Superseded by [SURPRISE:...] | Logged-only>

**Surprise weight:** <Low | Medium | High>
  — Low: aesthetic / hygiene; Medium: real drift but not blocking; High: looks like a real defect or systemic issue worth a closer look.

**What I noticed:** <concrete observation, with file/line references when applicable>

**Why it stood out:** <the agent's epistemic reasoning — what made this look "off">

**Likely landing zone:** <which cycle / phase / SPEC ID is the natural home, or
"no obvious home" if the agent thinks it's homeless>

**Cross-references:** <pointers to related KNOWN_ISSUES, BRAINSTORM, audit-log
entries, code locations, prior CHANGELOG/DEVLOG versions>
```

The `Authors:` line follows the `AGENT_CONVENTIONS.md § Authorship conventions`
rules — same format as CHANGELOG entries (model + runtime toggle).

### Status values

Status reflects both *whether* an entry is being addressed AND whether closure
has been verified. The taxonomy is deliberately not just "open vs closed" because
many entries pass through an intermediate "actively being addressed but not yet
verified closed" phase that needs its own state to keep the log honest.

- **Active** — observation stands; no formal action taken yet. Initial state.
- **In-flight (cycle X.Y stage Z | [ISSUE:YYYY-MM-DD:N] | DECISIONS [date])** —
  actively being addressed by named work elsewhere (a cycle stage, a
  KNOWN_ISSUES entry, a DECISIONS lock, a SPEC amendment) but the work has not
  yet closed. Status text MUST cite the addressing work concretely so a reader
  can find it. Verification gate: when the addressing work closes, the entry
  transitions to **Resolved** or back to **Active** (if the work didn't actually
  address it on closer look).
- **Promoted to <destination + ID>** — the entry's substantive home is now
  elsewhere (`KNOWN_ISSUES [ID]`, `BRAINSTORM [date entry title]`,
  `DECISIONS [date entry title]`, or `SPEC <SPEC ID>`). The SURPRISE entry stays
  for provenance. Verification gate: when the destination resolves, this status
  transitions to **Resolved (vX.Y.ZZ)** citing the closing version.
- **Resolved (vX.Y.ZZ)** — verified closed at a cycle re-audit or by a
  CHANGELOG/DEVLOG version commit. Cite the version that closed it. This is the
  ONLY terminal state that requires re-audit or version-commit verification —
  do not flip to Resolved on the implementer's word alone.
- **Superseded by [SURPRISE:...]** — a later, sharper observation in this same
  log captures the same phenomenon better. Point to it.
- **Logged-only** — the agent recorded it but does not believe action is
  warranted. Someone else (user, future agent) may disagree later and reopen by
  flipping to Active.

**No entry should remain Active forever.** If a re-auditor reviews surprises
tagged with their cycle (per Template C's surprise-review gate; see
`multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md`) and concludes the
observation is real but not addressed by this cycle, they MUST add a one-line
postscript explaining why no action this cycle (e.g., *"Out of scope for cycle
15.6a; landing zone is SPEC15.20 cycle 15.10a closeout sweep"*) and may flip to
**Promoted** if filing the entry as a KNOWN_ISSUES / BRAINSTORM entry is the
right disposition. Active-forever is a smell — explicit deferral with a landing
zone is the discipline.

### When to flip status (which agent role does what)

- **Authoring agent (any role)** — files entries as **Active** by default. Files
  entries as **Logged-only** if confidently not actionable. Files entries as
  **Promoted** if filing the substantive home (KNOWN_ISSUES / BRAINSTORM /
  DECISIONS / SPEC) at the same time.
- **Implementing agent (Role 2 / Template B)** — when picking up a surprise as
  in-cycle work, flip the entry's status to **In-flight (cycle X.Y stage Z)** at
  the time of pickup. Cite the stage so a reader can find the implementation.
  When the implementer's round closes pending re-audit, the In-flight tag stays;
  the re-auditor decides whether to flip to Resolved.
- **Re-auditing agent (Role 1 / Template C at closeout)** — REQUIRED to scan
  `[SURPRISE:<N>:<this cycle>:*:*:*]` entries plus any Active entries whose
  Likely landing zone tags this cycle, and decide for each: flip to **Resolved
  (vX.Y.ZZ)** if verified closed, leave/restore **Active** with a postscript
  if not addressed, flip to **Promoted to <ID>** if filing the substantive home
  is the right disposition, or flip to **Logged-only** if reassessment concludes
  no action warranted.
- **Final Overseer (Role 3 / Template D)** — at phase wrap-up, surfaces any
  remaining Active entries that should have been resolved during the phase as
  wrap-up findings (Template D produces its own audit; Active surprises feed it
  the same way orphaned KNOWN_ISSUES would).

### Authoring discipline

- **Be honest about epistemic confidence.** It is fine to write "I am not sure
  whether this is real" or "this looks wrong but I cannot articulate the fix."
  This file rewards uncertainty surfacing more than it rewards polished claims.
- **Prefer concrete over vague.** Even when the observation is fuzzy, point at
  specific files / lines / SPEC sections / cycle references. A vague surprise
  with no cross-references is not useful.
- **Append-only by default.** Do not delete entries; if the surprise turns out
  to be wrong on later examination, change the status to `Logged-only` (or
  `Resolved` with a note) and add a brief postscript explaining why. Provenance
  matters more than tidiness.
- **No CHANGELOG entry for adding to this file.** Entries are observational
  notes, not deliverables. They do not increment the version. If a SURPRISE
  entry catalyzes actual repair work in a later cycle, that cycle's CHANGELOG
  closeout entry can reference the SURPRISE ID for context.
- **Adding entries does NOT require user approval the way scope changes do.**
  This file is explicitly for the agent's voice; the user reads it as input,
  not as a contract. (Compare: SPEC scope is user-controlled per
  `AGENT_CONVENTIONS.md § Scope authority`. SURPRISE_LOG is agent-controlled.)

### Relationship to other planning surfaces

| Surface | What it holds | Versus this file |
|---|---|---|
| `PHASE<N>_AUDIT_LOG.md` | Round-by-round audit findings + implementation reports + closeout judgments — strictly in-scope per the cycle's SPEC contract | SURPRISE_LOG is for *out-of-scope* observations from the same cycle round |
| `multi-agent/tracking/KNOWN_ISSUES.md` | Long-lived **actionable** near-term issues with concrete exit conditions | SURPRISE_LOG is for observations that are NOT yet actionable, may never be, or are smells without a clear repair |
| `multi-agent/tracking/BRAINSTORM.md` | Speculative future-direction ideas (the long-lived idea reservoir) | SURPRISE_LOG is for "this looks off in the *current* tree", not "here is something we could do later" |
| `PHASE<N>_FEEDBACK.md` | Brainstorming-stage cross-agent audit feedback during SPEC engineering | SURPRISE_LOG accumulates across the entire phase, not just brainstorming |
| `multi-agent/project_context/DECISIONS.md` | Locked design decisions with rationale | SURPRISE_LOG entries can be promoted into DECISIONS entries if the user adjudicates them |

### Promotion paths (when a SURPRISE entry stops being just a smell)

- **Smell → KNOWN_ISSUE:** if a later session decides the observation is real
  + actionable + has a clear exit condition, file a sibling
  `[ISSUE:YYYY-MM-DD:N]` entry in `multi-agent/tracking/KNOWN_ISSUES.md` and
  cross-reference the SURPRISE ID. Update the SURPRISE entry's `Status:` to
  point at the new issue.
- **Smell → SPEC amendment:** if the user authorizes scope expansion to address
  the observation in the current phase, the SPEC body gets a new priority +
  the SURPRISE entry's status flips to `Resolved (vX.Y.ZZ)` at closeout.
- **Smell → DECISIONS lock:** if the observation is a definitional ambiguity
  the user resolves with a design call, write a DECISIONS entry citing the
  SURPRISE ID and flip the status to `Resolved`.
- **Smell → BRAINSTORM entry:** if the observation is genuinely speculative
  future-phase work, file a BRAIN entry in `multi-agent/tracking/BRAINSTORM.md`
  and flip the SURPRISE status to `Logged-only` (still preserved here for
  provenance).

### What does NOT go in this file

- Concrete findings WITH a repair contract — those are AUDIT_LOG findings.
- Re-derivable observations (e.g., "I noticed file X has 100 lines"). Only
  observations that an agent actually had to *think* to surface count.
- Routine restatements of locked DECISIONS. The point is NEW peripheral-vision
  observation, not summarizing what is already known.
- Emotional venting / blame / model-vs-model commentary. Surprise about *the
  code or planning surfaces*, not about other agents' choices. (Critique of a
  prior agent's reasoning IS allowed if it surfaces a real drift; criticism
  for criticism's sake is not.)

---

## Entries (newest first)

## [SURPRISE:15:15.9a:1:A:1] SPEC15.21 d2/d3 default-preservation premise is stale — live tree already runs regression detection at `--odw-prob-threshold` (0.9), not the legacy hardcoded `_DIP_PROB_THRESHOLD = 0.8`

**Authors:** John M. Urban, Claude Code 2.1.126 (claude-opus-4-7 ; Effort: Max)

**Status:** Resolved (v0.14.94) — cycle 15.9a R2 implementation locked OQ4 to **path (b) SPEC-literal `--odw-regression-prob-threshold = 0.8`** per Principal direction (DECISIONS.md `[2026-05-04] Phase 15 cycle 15.9a SPEC15.21 implementation locks` — "active-direction probability stays 0.9, regression-direction probability stays 0.8"). Restored the original SPEC15.21 design intent of asymmetric directional defaults (active 0.9 / regression 0.8); R1's recommended path (a) was overridden in favor of the SPEC-literal asymmetry. The "ZERO net change at default invocation" guarantee was restated relative to the *intended* legacy `_DIP_PROB_THRESHOLD = 0.8` rather than the live-tree-as-found 0.9 wiring. The 0.9 wiring at `aps.py:1942, 1948` (legacy callsite) was rewritten by R2's Stage B implementation to thread the new `--odw-regression-prob-threshold` (default 0.8) explicitly via `aps.py:1584, 1592` (post-relocation in `timing_diagnostics.py`). Calibration revalidation: `make puff-compare` 28/28 PASS at R3 closeout — HMM detection bit-for-bit unchanged; `make test` 262/262 + 1 skipped; `make toy` PASS. The "untraced prior cycle" referenced in this entry's body was likely cycle 15.3a's Stage D wiring; future-phase work tracking that history is not required since the asymmetry is now the locked + user-visible design.

**Original status:** Active — blocked cycle 15.9a R2 Stage B (F2/F3) until Principal locked OQ4.

**Surprise weight:** High (blocks an implementation stage until resolved).

**What I noticed:** SPEC15.21 deliverable 3 ([PHASE15_SPEC.md:1582-1591](/Users/johnurban/searchPaths/github/onionskin/multi-agent/plans/PHASE15_SPEC.md#L1582)) sets `--odw-regression-prob-threshold` default to `0.8` and notes that this "preserves" the legacy `_DIP_PROB_THRESHOLD = 0.8` constant. The "Implementation notes — Behavioral change at default invocation: ZERO net change" guarantee at `PHASE15_SPEC.md:1621` rests on the same premise. The live tree contradicts both: at [`onionskin_core/aps.py:1942`](/Users/johnurban/searchPaths/github/onionskin/onionskin_core/aps.py#L1942) `_compute_locus_diagnostics(..., dip_prob_threshold=odw_prob_threshold, ...)` and at [`aps.py:1948`](/Users/johnurban/searchPaths/github/onionskin/onionskin_core/aps.py#L1948) `_compute_global_degradation(transition_ps, dip_prob_threshold=odw_prob_threshold)` both override the function-default `_DIP_PROB_THRESHOLD = 0.8` ([`aps.py:1287`](/Users/johnurban/searchPaths/github/onionskin/onionskin_core/aps.py#L1287)) with the value of `--odw-prob-threshold` (default `0.9`). So regression detection currently runs at **0.9 not 0.8** at default invocation. A SPEC-literal `--odw-regression-prob-threshold = 0.8` would be a behavioral default change (regression P threshold 0.9 → 0.8), violating the SPEC's "ZERO net change" guarantee.

**Why it stood out:** Cycle 15.9a's whole premise rests on the per-direction split being structurally byte-identical at default invocation (the safety story for the cleanup). The SPEC text and the live tree disagree on what the regression-direction default actually IS today. This is the same class of architectural-debt-vs-spec-text drift surfaced repeatedly in the cross-pipeline-parity-blindspot pattern (`feedback_cross_pipeline_parity_blindspot.md`) — except here it's between SPEC text and live code rather than between pipelines. The drift was not caught by `[ISSUE:2026-04-29:2]` either: that issue's body still describes the asymmetry as "regression preserves the legacy `_DIP_PROB_THRESHOLD = 0.8` constant." Some prior cycle (untraced — likely cycle 15.3a R2 or a follow-up cycle that wired `dip_prob_threshold=odw_prob_threshold`) silently closed the asymmetry without updating the SPEC or KNOWN_ISSUES descriptions.

**Promotion path:** Principal locks OQ4 (cycle 15.9a R1 audit, STEP 6) by choosing path (a) live-tree-preserving 0.9 default + SPEC text revision, or path (b) SPEC-literal 0.8 default + DS1/DS2 calibration check. R1 recommends path (a). Once locked, the F2/F3 repair contract proceeds; this entry flips to Resolved (vX.Y.ZZ) at cycle 15.9a closeout with a postscript noting which path the Principal chose and the SPEC-text revision summary.

**Cross-references:**
- `multi-agent/plans/PHASE15_AUDIT_LOG.md` cycle 15.9a R1, F2 + F3 + OQ4.
- `multi-agent/tracking/KNOWN_ISSUES.md` `[ISSUE:2026-04-29:2]` (description body is itself stale per this entry).
- `multi-agent/plans/PHASE15_SPEC.md § SPEC15.21` deliverable 3 + implementation note "ZERO net change at default invocation."
- `feedback_cross_pipeline_parity_blindspot.md` (the broader pattern this is a SPEC-vs-code instance of).

## [SURPRISE:15:15.6a-S1:1:A:1] HMM step-16 APS call site silently bypasses `--aps-shape-no-normalize` and the new aggregation-choice surface that should replace `--aps-weight-loci`

**Authors:** John M. Urban, GitHub Copilot (GPT-5.4 ; Reasoning: High)

**Status:** Resolved (v0.14.90) — cycle 15.6a-S1 R2 Stage B repaired the parity gap at `onionskin_core/hmm_ported_analyses.py:236, 462-475`: `run_step16_hmm_aps` now declares `shape_normalize` + `aggregation_mode/score/score-min/emit_unfiltered_aggregates` as parameters, and `build_aps_feature_matrix(...)` is called with `shape_normalize=shape_normalize` + `aggregation_mode=aggregation_mode` (replacing the prior hard-coded `shape_normalize=True` + `weight_loci=False`). End-to-end thread verified at `onionskin.py:4212-4216` → `engines/hmm_engine.py:1640-1643` → `run_step16_hmm_aps`. New regression test `test_hmm_step16_threads_shape_and_weight_flags` at `tests/test_hmm_ported_analyses.py:214-289` regression-locks the threading. `--aps-weight-loci` is retired entirely; the cross-pipeline-thinner-HMM concern at this call site is closed.

**Surprise weight:** Medium

**What I noticed:** The HMM APS path in [onionskin_core/hmm_ported_analyses.py](/Users/johnurban/searchPaths/github/onionskin/onionskin_core/hmm_ported_analyses.py#L431) hard-codes `shape_normalize=True` and `weight_loci=False` when it calls `build_aps_feature_matrix(...)`, while the Growth and RMS APS path in [onionskin_core/aps.py](/Users/johnurban/searchPaths/github/onionskin/onionskin_core/aps.py#L1910) and [onionskin_core/aps.py](/Users/johnurban/searchPaths/github/onionskin/onionskin_core/aps.py#L1912) threads the resolved CLI values. That means HMM silently ignores `--aps-shape-no-normalize` today. It also means the legacy `--aps-weight-loci` pathway is already drifting on HMM, and the cycle-15.6a-S1 orchestrator clarification's replacement surface (`--aps-aggregation-mode` plus `--aps-aggregation-score`) would be equally vulnerable if R2 only refactors Growth/RMS and forgets this HMM call site.

**Why it stood out:** This is the same cross-pipeline thinner-HMM APS pattern previously captured at `[SURPRISE:15:15.6a:1:A:7]` and `[SURPRISE:15:15.6a:1:A:8]`, but now at a more local and testable abstraction boundary. Cycle 15.6a's shared APS resolver work reduced one earlier divergence, but the owning HMM call site still wires different behavior than Growth/RMS. For Stage A of cycle 15.6a-S1 this matters immediately because the Track 1 defaults inventory and evidence grid would otherwise compare non-equivalent feature matrices across pipelines without any visible error.

**Likely landing zone:** Cycle 15.6a-S1 itself. Stage A should inventory it explicitly, and Stage B should resolve it by threading the HMM path through the same resolved APS clustering / aggregation knobs used by Growth and RMS, then verifying the fix in the cross-pipeline parity table.

**Cross-references:**
- [multi-agent/plans/PHASE15_AUDIT_LOG.md](/Users/johnurban/searchPaths/github/onionskin/multi-agent/plans/PHASE15_AUDIT_LOG.md#L180) F2.4.
- [multi-agent/plans/PHASE15_AUDIT_LOG.md](/Users/johnurban/searchPaths/github/onionskin/multi-agent/plans/PHASE15_AUDIT_LOG.md#L268) F8.
- [onionskin_core/hmm_ported_analyses.py](/Users/johnurban/searchPaths/github/onionskin/onionskin_core/hmm_ported_analyses.py#L431).
- [onionskin_core/aps.py](/Users/johnurban/searchPaths/github/onionskin/onionskin_core/aps.py#L1910).
- `[SURPRISE:15:15.6a:1:A:7]` and `[SURPRISE:15:15.6a:1:A:8]`.

## [SURPRISE:15:15.4a-S2:1:A:3] HMM `shape_score_raw` is computed differently from Growth/RMS — different stage source + different window

**Authors:** John M. Urban (pushback), Claude Code 2.1.117 (claude-opus-4-7 ; Effort: Max) (code dive + transcription).

**Status:** Resolved (v0.14.87) — cycle 15.4a-S2 Stage B.7 implemented the structural harmonization (rejected the threshold-0 band-aid). `_shape_score_for_row` at `onionskin_core/hmm_multistage_unification.py:196-214` now consumes the latest-stage profile via `_latest_stage_context_profile(...)` with `ctx_bins=100`, matching Growth/RMS's `shape_filter_calls` (`onionskin_core/refinement.py:473-526`). HMM `step14_shape_score_threshold` default restored to `50.0` cross-pipeline (`onionskin_core/engines/hmm_engine.py:1246`; `_effective_hmm_shape_score_threshold` at `onionskin.py:3260-3265`). User-eyes rerun (`dev/runs/15-4a-S2-user-eyes-flagfix/`) keeps 9/12 chr-II step13 trajectories at threshold 50 (was 3/12). The 3 dropped trajectories (`II:6675000-6740000`, `II:47345000-47430000`, `II:55640000-55670000`) all have very low shape scores (1.55, 5.86, 26.53 — all below threshold 50) AND fail state-path evidence (state_path_growth_credibility_score=0.17), making them HMM-detection false-positives rather than by-eye-confirmed amplicons. Cross-pipeline context confirms the 9 chr-II count is in plausible biological-real-data variance range (Growth 10 final / 8 APS reliability pass; RMS 13 pass; original HMM 3 pass). Codex's R2 report flagged a factual nit (the SURPRISE entry's claim about `dBIC_flat_vs_tri` having a `ctx_bins` parameter is slightly off; the parameter is on the caller `shape_filter_calls`); the substance of the SURPRISE finding holds.

**Surprise weight:** High — this is a structural cross-pipeline asymmetry that R2's cycle 15.4a-S2 Stage B.7 fix (flip HMM `step14_shape_score_threshold` default from 50 → 0) papers over rather than resolving. The threshold=50 default was carefully calibrated against Growth/RMS-style shape-score computation; flipping it to 0 for HMM ships HMM with a structurally different test that effectively disables the shape filter.

**What I noticed:** Two different implementations of the BIC-based triangle-vs-flat shape filter exist in the codebase:

- **Growth/RMS path** uses `dBIC_flat_vs_tri` defined at `onionskin_core/refinement.py:477`. It accepts a `ctx_bins: int = 100` parameter and constructs the BIC comparison window as `lo = max(0, idx_s - ctx_bins); hi = min(len(ya) - 1, idx_e + ctx_bins)` — the call range plus 100 bins of genomic context on each side. The shape-filter-plots module documents (`onionskin_core/shape_filter_plots.py:11`) that this is *"fitted via least-squares against the **last-stage** log2RCN profile in a context window ±ctx_bins."* So Growth/RMS use the LAST stage (typically the most amplified) plus ±100-bin context.

- **HMM path** uses `_shape_score_for_row` at `onionskin_core/hmm_multistage_unification.py:181-197`. It calls `dBIC_flat_vs_tri(vals, 0, idx_p, vals.size - 1, baseline, strict_bic=strict_bic)` with `lo=0` and `hi=vals.size-1` — the **entire amplicon range only, no context window**. The `vals` array comes from `_mean_shape_profile` at `hmm_multistage_unification.py:157-178`, which **averages across ALL stages** (`vals = np.array([np.nanmean(keyed[mid]) for mid in mids])`). So HMM uses the across-stage MEAN profile + no context.

**Two structural differences:**

1. **Stage source.** HMM averages across all stages; Growth/RMS use last-stage only. Averaging dilutes the peak shape — a stage-1 "no signal" profile averaged with a stage-5 "big peak" profile gives a moderate shape that doesn't fit a triangle as well as the last-stage peak alone. Lower BIC-difference scores result.

2. **Context window.** HMM has none (lo=0, hi=full amplicon); Growth/RMS has ±100 bins of surrounding genomic baseline. Without the context window, the BIC comparison has weaker discriminating power between "this is a peak above baseline" vs "this is just bumpy noise" — both fit similarly poorly. Growth/RMS's context window provides the strong "this is what flat looks like outside the call" signal that makes the triangle-fit advantage stand out.

**Why it stood out:** R2's cycle 15.4a-S2 Stage B.7 fix flipped HMM `step14_shape_score_threshold` default from 50 → 0 to keep the 9 missing II amplicons that fail at threshold=50. The user pushed back (verbatim 2026-04-30): *"`shape_score_threshold=0` doesn't sound good to me ... I am not sure it is the threshold that needed changing. The threshold was carefully selected long ago when testing for RMS or Growth. If it is not working for the HMM pipeline, the question is why not? what is the HMM pipeline doing differently? Is it doing it in a different window length? stuff like that should be investigated. ... `shape_score_threshold=0` sounds like a band-aid that can stop working easily."*

The user is correct. The threshold=50 default was calibrated for Growth/RMS-style scores; HMM-style scores are systematically lower because of the two structural differences above. The right fix is harmonizing HMM's `_shape_score_for_row` to match the Growth/RMS computation method (last-stage profile + ±100-bin context window), not lowering the threshold.

**Likely landing zone:**

- **Cycle 15.7a (HMM-specific late-phase work, SPEC15.13/14/15/16) is the natural home** — that cycle already touches HMM internals. Add a deliverable: harmonize `_shape_score_for_row` to use `(a)` last-stage profile (not across-stage mean), `(b)` add ctx_bins padding via `dBIC_flat_vs_tri`'s existing parameter. Validate that HMM step14 output at threshold=50 matches the Growth/RMS-style expectations on user's `dev/share/res20260430-1/` and toy fixtures. Once harmonized, restore `step14_shape_score_threshold=50` default cross-pipeline.
- **Alternative:** new mid-Phase-15 amendment cycle (e.g., 15.4a-S3) specifically for shape-score computation harmonization, sized small enough to land before cycle 15.6a-S1.
- **Decision pending user direction:** whether R2's threshold=0 default flip lands in cycle 15.4a-S2 as a temporary band-aid (with explicit "until 15.7a or 15.4a-S3 lands the structural fix" note) OR is REVERTED to 50 and the threshold-tuning portion of Stage B.7 is held until structural fix arrives.

**R1 recommendation: REVERT R2's threshold=0 default flip; keep threshold=50 cross-pipeline; flag as known-issue with landing zone cycle 15.7a or new 15.4a-S3.** Threshold=0 effectively disables the shape filter for HMM, which is a regression from current behavior and produces 23 chr-II amplicons in chrom-median diagnostic mode (vs by-eye 10-13). The threshold-tuning portion of R2's Stage B.8 (prob_threshold 0.9→0.7, fold_threshold 1.25→1.10 in `flat_sample.py`) should still land — those are flat-test-specific defaults that were inappropriately inheriting from ODW active-stage defaults; that's a legitimate fix.

**Cross-references:**

- `multi-agent/plans/PHASE15_AUDIT_LOG.md` cycle 15.4a-S2 Role 2 Stage A Diagnostic Report (Codex's A.5: "Threshold sweep on the existing step14 table shows the current default `shape_score_threshold=50` keeps 8 genome-wide rows and 3 chr-II rows; threshold `0` would keep 56 genome-wide rows and all 12 chr-II step13 rows.") — this is the data that surfaced the threshold-vs-method question
- `onionskin_core/refinement.py:477` (`dBIC_flat_vs_tri` definition; ctx_bins parameter; the function HMM also calls but with the wrong arguments)
- `onionskin_core/single_engine.py` (Growth caller — uses ctx_bins + last-stage)
- `onionskin_core/rcn_mean_shift_helpers.py` (RMS caller — uses ctx_bins + last-stage)
- `onionskin_core/hmm_multistage_unification.py:157-197` (HMM caller — uses across-stage mean + no ctx_bins)
- `onionskin_core/shape_filter_plots.py:11` (documentation of intended Growth/RMS contract)
- Code: R2's working-tree change at `onionskin_core/engines/hmm_engine.py:1246` (`step14_shape_score_threshold: float = 0.0` — the band-aid to revisit)

---

## [SURPRISE:15:15.4a-S2:1:A:1] Within-stage between-sample MAD as the per-locus flat-test sigma is a systematic-error design flaw

**Authors:** John M. Urban (analytic insight + framework), Claude Code 2.1.117 (claude-opus-4-7 ; Effort: Max) (data + code dive + transcription).

**Status:** Resolved (v0.14.88) for the dispatch-architecture portion. Cycle 15.4a-S2 closed at v0.14.87 with the in-scope tactical patches (Stage B.1 chrom-median flat-sample signal source + Stage B.8 tuned thresholds 0.7/1.10). Cycle 15.4a-S3 then landed the structural sigma-source dispatch architecture and CLOSED at v0.14.88 (Role 2: GitHub Copilot GPT-5.4 Reasoning: Extra High; Role 1 R3 re-audit: Claude Code 2.1.126 claude-opus-4-7 Effort: High): new CLI flag `--flat-sample-sigma-source {within-stage-mad, chrom-mad, contrib-spread, fixed}`, separate `--flat-sample-detector` plus fixed-sigma overrides, cross-pipeline Growth/RMS/HMM plumbing, `sigma_source` emission in `flat_samples.tsv`, placeholder `chrom-mad` whole-chrom computation from chrom-median sample tracks (NOT HMM step-5 indiv_samples — Stage A.3 asymmetry bypassed by sourcing from cross-pipeline-shared `compute_sample_rcn_tracks(norm_mode="chrom-median")`), in-memory gap-bin `0 -> NaN` handling at the chrom-MAD computation site (no NaN written to any bedGraph), and a default-flip from `within-stage-mad` to `chrom-mad` as a user-authorized principles-default. Real-fixture validation was redirected from Google Drive paths to workspace-local manifests under `dev/datasets/full_genome/batch/` per user direction; no chr-II reliability-pass-count regression observed on that fixture. **Carry-forward postscript:** the structural noise-source rework continues into cycle 15.4a-S4 (full mask-restricted `chrom-mad` formulation: pre-pipeline shared naive background mask + per-sample chrom-MAD bedGraph emission cross-pipeline) and SPEC15.15 cycle 15.7a re-scoped (within-prior second-pass + posterior background-region inheritance from prior). Cycle 15.4a-S3 ships the half-formulation; the multi-cycle plan locked in `multi-agent/plans/PHASE15_BACKGROUND.md` lands the rest.

**Cycle 15.4a-S4 completion postscript (2026-05-03; appended at v0.14.89 closeout by Role 1 R3 re-audit Claude Code 2.1.126 (claude-opus-4-7 ; Effort: Max)):** Cycle 15.4a-S4 closed at v0.14.89 with the structural mask machinery, per-sample chrom-MAD bedGraph emission cross-pipeline, Path B.2.A (HMM gains `<hmm-out>/16-aps/samples/`), and the new `chrom-background-mad` default. The cycle followed a R1-audit → first-orchestrator-clarification → R2-Stage-A-empirical-evidence → SECOND-orchestrator-clarification → R2-Stage-B trajectory; the SECOND clarification dropped `learn` mode + GMM machinery + guard rails entirely in favor of biology-grounded static defaults (LO=0, HI=1.75) after R2's Stage A.5 sweep showed GMM-derived valleys produced artifact-tight cutoffs (σ-floor sanity check fired on 7 of 9 manifests). Calibration evidence at `dev/runs/15-4a-S4-calibration/sigma_source_comparison.tsv`: `chrom-background-mad` sigma_log2 = 0.290 at SAG40 stage-1 chr-II vs `whole-chrom-mad` 0.371 (~22% reduction); HMM `II:33175000-33795000` clears tuned 0.7 gate under the new default (P=0.787); HMM `II/9A` still fails (P=0.623). The F5/B.9/C.8 acceptance criterion was reframed from hard pass-fail to evidence-only per the SECOND clarification — `chrom-background-mad` is preferred for being PRINCIPLED (excludes ≥2× amplification from MAD computation by construction), NOT for producing a measurably smaller sigma at every worked-example locus. The truly principled formulation lands at cycle 15.7a (re-scoped SPEC15.15) when the prior's HMM-derived above-background regions inform the chrom-background mask. Status field stays "Resolved (v0.14.88) for the dispatch-architecture portion" — the dispatch-architecture portion is genuinely resolved at v0.14.88; the structural mask portion landed cleanly at v0.14.89 (this postscript); the carry-forward continues into cycle 15.7a for posterior background-region inheritance. Multi-cycle plan locked in `multi-agent/plans/PHASE15_BACKGROUND.md` is now 2 of 3 cycles complete.

**Cycle 15.7a Stage G R3 cycle-completion postscript (2026-05-04; appended at v0.14.91 closeout by Role 1 R3 re-re-audit Claude Code 2.1.126 (claude-opus-4-7 ; Effort: Max); SUPERSEDES the prior PARTIAL/deferral framing). The R3-original closeout-attempt (commit `e301afd` pre-override) proposed PARTIAL + new `[ISSUE:2026-05-04:1]` for d3/d4/d6/d8 deferral; the Principal-orchestrator override 2026-05-04 reopened the cycle and issued a Stage G repair contract; R2 (GitHub Copilot — Claude Sonnet 4.6 ; Effort: High + Medium) landed all six Stage G deliverables at commit `c38654e`; this R3 round verified Stage G clean and closes the cycle.** Cycle 15.7a CLOSED v0.14.91 with **SPEC15.15 fully DONE** (NOT PARTIAL):

- **d3 within-prior re-run with refined mask** — landed cross-pipeline. HMM re-runs steps 3-8 with refined mask at `hmm_engine.py:1704-1796` (joint + indiv); Growth re-runs full pipeline via `run_multistage(_p2_ms_argv + ["--background-mask-bed", _bed_path])` at `onionskin.py:947-959`; RMS re-runs via `run_rcn_mean_shift(...)` with `_prepare_stage_structured_tracks(..., mask_intervals=_mask_intervals_rms)` at `onionskin.py:4176-4217`. Locked retain_above_hi asymmetry preserved: HMM/RMS replace (`False`); Growth retain (`True`) per BACKGROUND.md.
- **d4 atomic pass-2 promotion + `pass1/` archive** — landed cross-pipeline. `pass2_in_progress=True` sentinel BEFORE archive; archive via `shutil.copy2` to per-step `pass1/` subdirectory; pass-2 re-run; `pass2_completed=True` sentinel finalized after. Verified in all three pipelines (HMM `hmm_engine.py:1681-1789`; Growth `onionskin.py:927-972`; RMS `onionskin.py:4154-4230`).
- **d6 posterior chrom-renorm/MAD inheritance from prior pass-2 mask sidecar** — landed via pure resolver. `_resolve_prior_mask` at `onionskin.py:5063-5095` (NEVER mutates `args._first_pass_background_mask`; per-pipeline independent resolution). All three posterior helpers call it independently with their own prior pipeline dirs: `_run_rcn_posterior:5117-5118`, `_run_hmm_posterior:5211`, `_run_posterior:5328-5329`. Threading via `first_pass_background_mask_override` (HMM), `mask_intervals_override` (RMS), `--background-mask-bed <path>` argv (Growth, given Growth engine's argv-only interface). Per-pipeline mask independence (Option A) preserved; cross-pipeline synthesis modes (intersect/union/hybrid) deferred to SPEC15.21 d12 (cycle 15.9a) `--posterior-inherited-mask` flag.
- **d8 full pipeline plumbing** — landed in all three engines + the controller. HMM engine receives `second_pass_background_estimation` kwarg at `hmm_engine.py:1355` with default `off`; Growth engine accepts `--background-mask-bed BED` argv at `growth_model_engine.py:1158-1166`; RMS controller orchestrates pass-2 at `onionskin.py:4118-4251`. Default-off path BYTE-IDENTICAL to v0.14.90 baseline (verified `make puff-compare` 28/28 + `make test` 255 passed/1 skipped + dedicated regression test at `tests/test_second_pass_background_default_off_byte_identical.py` 8/8 PASS).
- **F4.5 / `[ISSUE:2026-05-01:1]` narrow stage-1 anchor sigma-source repair** — landed via Stage G G.5. `--odw-stage1-sigma-source` flag at `onionskin.py:1761-1773` (default `chrom-background-mad`); dispatch at `onionskin_core/timing.py:_median_mad_sigma:215, 453` (stage-1 only; stages N>1 untouched per NARROWED F10 framing); regression test at `tests/test_stage1_anchor_sigma_source.py` 4/4 PASS.

**Multi-cycle plan in `PHASE15_BACKGROUND.md` is now substantively 3-of-3 complete:**
1. **Cycle 15.4a-S3 (v0.14.88)** — sigma-source dispatch architecture: `--flat-sample-sigma-source {within-stage-mad, chrom-mad, contrib-spread, fixed}` + per-pipeline plumbing.
2. **Cycle 15.4a-S4 (v0.14.89)** — full mask-restricted formulation: pre-pipeline shared naive background mask + per-sample chrom-MAD bedGraph emission cross-pipeline + new `chrom-background-mad` sigma-source value.
3. **Cycle 15.7a (v0.14.91)** — within-prior pass-2 background-region renorm + posterior background-region inheritance from prior pass-2 mask sidecar cross-pipeline + stage-1 anchor sigma-source repair.

The principled `chrom-background-mad` formulation is now cross-pipeline complete for both prior pass-2 + posterior inheritance contexts under user-enabled `--second-pass-background-estimation on`. The dispatch-architecture portion remains genuinely Resolved (v0.14.88); the structural mask portion remains Resolved (v0.14.89); the within-prior pass-2 + posterior-inheritance portion lands cleanly at v0.14.91 (this postscript supersedes the prior PARTIAL framing). All three resolved.

**Surprise weight:** High — the user has independently identified this as a structural bug that will not be fully addressed by Phase 15's threshold-tuning fixes (cycle 15.4a-S2 Stage B.8). Likely candidate for the next phase — possibly absorbed by `multi-agent/plans/next/FLAT_SOUP.md` upstream-prior architecture, or a dedicated successor SOUP for the noise-source rework.

**What I noticed:** The MAD bedGraphs that `flat_sample.compute_flat_sample_sidecar` reads as the noise sigma for the per-locus Bayesian growth test (`onionskin_core/flat_sample.py:380-411`) are computed as **between-sample variance WITHIN A STAGE at each genomic bin**. The writer is `_build_stage_stats` at `onionskin_core/aps.py:142-170`:

```python
sids = [m['sample_id'] for m in metas if m['stage'] == stage]
stack_rcn  = np.column_stack([sample_tracks[sid]['RCN'].to_numpy(...) for sid in sids])
med_rcn  = np.nanmedian(stack_rcn,  axis=1)
mad_rcn  = np.nanmedian(np.abs(stack_rcn  - med_rcn[:, None]),  axis=1)
```

So `mad_rcn[bin_i]` = median absolute deviation of RCN values across replicate samples sharing a stage at that specific bin. When samples within a stage diverge biologically (e.g., SAG40 has growth at II:33175000-33795000 with `summit_rcn=1.378` while SAG41 has the negative-bump artifact `summit_rcn=0.621` at the same locus — `dev/share/res20260430-1/01-prior/01-hmm/16-aps/locus_contributions.tsv`), the within-stage between-sample MAD is HIGH at those bins. **The MAD is high precisely BECAUSE the biological signal is real — flat sample vs growing sample disagree.** That same high MAD is then fed into the per-locus Bayesian flat-sample test as the noise sigma, which inflates the posterior `denom` and squashes the probability that growth exceeds `fold_threshold` even when the residual signal is well above threshold.

Concrete arithmetic for SAG40 stage 1 at II:33175000-33795000 (data + math verified 2026-04-30):

- summit_rcn = 1.378 → log2 = +0.462
- baseline = 1.0 → log2 = 0.0
- log2(fold_threshold=1.25) = +0.322
- MAD_log2 at locus from `01-hmm/16-aps/genome_stage_medians/stage1.MAD_log2RCN.bedGraph` ≈ 0.254 → sigma = 0.254 × 1.4826 ≈ 0.377
- denom = sqrt(0.377² + 0.377²) = 0.377 × √2 ≈ 0.534
- Posterior P(growth > 1.25×) = 1 − Φ((0.322 − 0.462)/0.534) = 1 − Φ(−0.264) = **0.604**
- prob_threshold = 0.9 → 0.604 < 0.9 → not active → flat

The signal IS above the fold threshold (1.378 > 1.25), but the high MAD-derived sigma (driven by SAG40-vs-SAG41 disagreement at that bin) prevents the test from confidently calling it growth. The detector is consuming the very signal it should detect as if it were noise.

**Why it stood out:** The user's pushback during cycle 15.4a-S2 R2 flight (verbatim 2026-04-30): *"if I only had 1 sample, and I wanted to test if the II/9A summit was growing, I would use the chrom-median and the chrom-MAD from the single sample itself to test the II/9A site... if there is a flat sample and a growing sample, the variance in the 'same bin' across samples will be high — but it is not randomly high, it is a systematic true difference and the design is the error... systematic error."*

The user is correct. The design choice to use within-stage between-sample MAD as the noise estimate conflates two semantically distinct quantities:

1. **True per-sample noise** — technical / sequencing / GC / replicate-handling variance. This is what a sigma in a Bayesian growth test should represent.
2. **Inter-sample biological difference** — one sample is amplified, another is not. This is the SIGNAL the flat-detector is trying to find.

The current code estimates (1) using a quantity that includes both (1) and (2). When (2) is large, the sigma estimate is inflated, and the very test designed to detect (2) loses sensitivity. Self-defeating.

**Proposed alternative (per user direction 2026-04-30):** Use **within-sample chrom-MAD** — for each sample, compute the MAD of RCN values across all bins on the chromosome (relative to that sample's chrom-median). This represents true per-sample noise (technical variance across the chromosome's bins) and is INDEPENDENT of whether other samples in the same stage are amplified or flat. A single-sample test makes biological sense: *"this sample's bin at II/9A is N MADs above its own chrom-median; what's the probability that's real growth vs noise?"*

This is structural, not threshold-tuning. R2's cycle 15.4a-S2 fix (lower fold_threshold 1.25→1.10 + lower prob_threshold 0.9→0.7) is a tactical patch within the existing within-stage-MAD framework. It will catch SAG40 at the moderate-MAD II/9A locus (summit_rcn=1.20, MAD_log2≈0.14, prob ≈ 0.65 with new defaults) but will still struggle at the high-MAD II:33175000-33795000 locus (MAD_log2≈0.25; even with looser thresholds the inflated sigma fights against detection). Worse, the loosened thresholds at low-MAD loci may produce false-positive growth calls in genuinely-noisy datasets — the patch shifts the failure mode rather than fixing it.

**Likely landing zone:** `multi-agent/plans/next/FLAT_SOUP.md` graduation phase, OR a dedicated successor SOUP for noise-source rework. Specifically the work would:

1. Add `chrom_MAD` per-sample bedGraph emission (sister to per-stage MAD bedGraphs).
2. Wire `flat_sample.compute_flat_sample_sidecar` to consume the per-sample chrom-MAD as sigma source (with the existing per-stage between-sample MAD path retained as fallback for compatibility).
3. Possibly extend to the broader ODW transition test (`onionskin_core/timing.py:_median_mad_sigma`) with the same chrom-MAD-per-sample option. The ODW test has the same systematic-error vulnerability when between-sample MAD is used as a stage-to-stage transition test sigma.
4. Validate cross-pipeline: this surface is in shared `aps.py`, so a fix benefits all three pipelines.

This is structural enough that landing it in a dedicated phase (alongside FLAT_SOUP's upstream-prior-pipeline architecture) makes more sense than mid-Phase-15 amendment. The Phase 15 cycle 15.4a-S2 Stage B.1 (chrom-median signal source) + B.8 (threshold tuning) are the right Phase-15 patches; the noise-source rework is the right next-phase work.

**Cross-references:**
- `multi-agent/plans/next/FLAT_SOUP.md` — natural future-phase home; this surprise is a graduation-trigger consideration
- `multi-agent/plans/next/CROSS_PIPELINE_UNIFICATION_SOUP.md` Item 6 — feature-parity reckoning; cross-pipeline noise-source consistency fits the broader cross-pipeline-1st-class theme
- `multi-agent/tracking/KNOWN_ISSUES.md [ISSUE:2026-04-30:3]` — the cycle 15.4a-S2 Driver 1 issue; Stage B.8's threshold tuning addresses the proximate symptom but not this structural cause
- Code: `onionskin_core/aps.py:142-170` (writer); `onionskin_core/flat_sample.py:340-411` (consumer); `onionskin_core/timing.py:480-510` (analogous ODW consumer with same vulnerability)
- Data evidence: `dev/share/res20260430-1/01-prior/01-hmm/16-aps/genome_stage_medians/stage1.MAD_log2RCN.bedGraph` (high MAD at II:33175000-33795000 bins)

---

## [SURPRISE:15:15.4a-S2:1:A:2] `locus_contributions.tsv` `flat_sample_flag` column is stale-False even when the sample-level rollup says True

**Authors:** John M. Urban (observation), Claude Code 2.1.117 (claude-opus-4-7 ; Effort: Max) (verification + transcription).

**Status:** Resolved (v0.14.87) — cycle 15.4a-S2 R2 picked this up as adjacent in-cycle work and folded the per-(sample, prior_stage) backfill into both Growth/RMS APS (`onionskin_core/aps.py:compute_and_write_aps` calls `attach_flat_sample_flags_to_contributions(contrib_df, flat_df)` after reliability attachment) and HMM APS (`onionskin_core/hmm_ported_analyses.py:run_step16_hmm_aps` mirrors). Verified at re-audit by inspecting `dev/runs/toy_out/01-prior/01-hmm/16-aps/locus_contributions.tsv`: stage-1 samples (flagged True in `flat_samples.tsv`) now show `flat_sample_flag=True` per row; stage-2 through stage-5 samples (flagged False) show False. `[ISSUE:2026-04-30:5]` flipped to Resolved (v0.14.87) at this same closeout.

**Surprise weight:** Medium — small bug with downstream-correctness implications. Anyone consuming `locus_contributions.tsv` directly (rather than joining against `flat_samples.tsv`) gets stale flat-classification data per row.

**What I noticed:** `dev/share/res20260430-1/01-prior/01-hmm/16-aps/locus_contributions.tsv` has every SAG40 stage-1 row showing `flat_sample_flag = False`, even though `flat_samples.tsv` correctly reports SAG40 stage 1 with `flat_sample_flag = True` (n_flat=7, n_reliable=7, flat_fraction=1.0). The two TSVs disagree on the same fact about the same sample. The `flat_sample_flag` column in `locus_contributions.tsv` appears to be either (a) never populated from the flat-samples sidecar, or (b) populated from a stale snapshot before the sidecar is finalized, or (c) populated from a different per-locus quantity that happens to share the same column name.

The reliability scorer at `onionskin_core/reliability.py:180` consumes `flat_sample_ids` from the flat sidecar correctly (`flat_flag = bool(sample_ids and sample_ids.issubset(flat_ids))`), so `amplicon_reliability.tsv`'s `flat_sample_flag` column reflects the correct sidecar value. The bug is specifically in `locus_contributions.tsv` writes.

**Why it stood out:** During the deep-dive on cycle 15.4a-S2 R1 audit's chicken-and-egg story, I pulled all SAG40 + SAG41 rows from `locus_contributions.tsv` to verify the residual-bump claim. The flat_sample_flag column showed False everywhere for SAG40, which conflicted with the n_flat=7 / 7 reading in `flat_samples.tsv`. Confirmed by reading the relevant code paths — the column is in the schema (locus_contributions header includes `flat_sample_flag keep_override`) but the value being written is not coming from the post-flat-sidecar consumer chain.

**Likely landing zone:**
- **If R3 catches it during cycle 15.4a-S2 re-audit:** in-cycle small fix (likely a one-line population in `compute_and_write_aps` after the flat sidecar lands). Worth absorbing if R3 has cycle-budget.
- **Otherwise:** filed at cycle 15.10a SPEC15.20 closeout sweep (KNOWN_ISSUES carry-over), OR promoted to `multi-agent/tracking/KNOWN_ISSUES.md` as a separate small-bug entry the user can decide whether to fix in a future cycle.

**Cross-references:**
- `onionskin_core/aps.py:compute_and_write_aps` — the writer of `locus_contributions.tsv`; flag-population logic is somewhere in this function
- `onionskin_core/flat_sample.py:write_flat_sample_sidecar` — produces the authoritative `flat_samples.tsv`
- `onionskin_core/reliability.py:180` — correct consumer pattern; the locus_contributions writer should mirror this
- Data evidence: `dev/share/res20260430-1/01-prior/01-hmm/16-aps/locus_contributions.tsv` (column always False for SAG40 + SAG41 stage-1 rows) vs `flat_samples.tsv` (correctly reports True)

---

## [SURPRISE:15:15.6a:2:B:1] `HANDOFF.md` top state is current, but deeper orientation sections still carry stale Phase 14 / Phase 16-era cues

**Authors:** John M. Urban, Codex-cli 0.126.0-alpha.8 (GPT-5.5 ; Reasoning: Extra High)

**Status:** Active — landing zone unchanged at SPEC15.20 / cycle 15.10a closeout sweep.

**Re-audit postscript [cycle 15.7a R3, 2026-05-04]:** R2 mid-cycle attached a "cycle 15.7a stage E in-flight" line to this entry, but the work R2 actually performed (SPEC15.13/15.14/15.16 line-number-annotation sweep) is the substantive home of `[SURPRISE:15:15.6a:1:A:3]`, not this HANDOFF-deeper-section drift. R3 reverts this entry's Status to `Active` and re-routes the in-flight credit to `[SURPRISE:15:15.6a:1:A:3]` where it belongs. The HANDOFF drift originally observed here remains untouched — still landing-zoned at SPEC15.20 / cycle 15.10a closeout sweep.

**Surprise weight:** Low

**What I noticed:** During the cycle 15.6a Role 2 wrap-up I updated the top
cold-start state, `## Last Action`, active `Next planned work`, and bottom
`## Next Action` pointers in `multi-agent/project_context/HANDOFF.md` so a
cold-start agent lands on the correct next event: cycle 15.6a Round 2 re-audit.
But a quick grep still shows deeper historical/current-state sections carrying
older orientation cues, including Phase 14 Supplemental "Final Overseer is the
NEXT event" wording, a "Phase 16 planned" pointer, an old `make test` count, and
preserved historical launchers below the active next-action block.

**Why it stood out:** The top of `HANDOFF.md` is now correct and load-bearing, so
this did not belong in the formal cycle 15.6a Role 2 implementation report. But
the file has accumulated enough historical strata that a cold-start reader who
skims below the fresh top section could receive mixed signals about what is live
versus provenance. This is exactly the kind of adjacent planning-surface hygiene
drift that is too small for a repair contract but worth not silently forgetting.

**Likely landing zone:** SPEC15.20 / cycle 15.10a closeout sweep, or any
dedicated dev-system cleanup pass that refreshes `HANDOFF.md` by separating
active cold-start pointers from preserved historical launcher provenance.

**Cross-references:**
- `multi-agent/project_context/HANDOFF.md` top cold-start state and `## Last Action`
  (currently correct for cycle 15.6a Role 2 complete / re-audit queued).
- `multi-agent/project_context/HANDOFF.md` deeper historical/current-state
  sections containing old Phase 14 Supplemental / Phase 16-era cues.
- Cycle 15.6a Role 2 implementation report in `PHASE15_AUDIT_LOG.md`.

**Re-audit postscript [cycle 15.6a R3 closeout, 2026-04-30]:** Active. R2 author's
own observation; the cold-start surface fix was in scope for R2 wrap-up but the
deep historical-strata refresh is hygiene drift outside cycle 15.6a's contract.
Landing zone: SPEC15.20 / cycle 15.10a closeout sweep.

---

## [SURPRISE:15:15.6a:1:A:1] `--aps-weight-loci` is documented as a no-op but cycle 15.4a Stage J wired it up

**Authors:** John M. Urban, Claude Code 2.1.117 (claude-opus-4-7 ; Effort: Max)

**Status:** Resolved (v0.14.90) — cycle 15.6a-S1 retired `--aps-weight-loci` entirely in favor of the new `--aps-aggregation-{mode,score,score-min}` flag matrix + `--aps-emit-unfiltered-aggregates` side-car. The drift no longer exists because the flag itself no longer exists. Retirement locked at `multi-agent/project_context/DECISIONS.md` 2026-05-03 entry (`APS aggregation replaces --aps-weight-loci`). Stale references swept from `README.md` + `aps.py` docstring; help text gone. Cross-pipeline parity carried by the new flag matrix verified per cycle 15.6a-S1 F8 parity table.

**Surprise weight:** Medium

**What I noticed:** The CLI help text at `onionskin.py:1845-1862` and the docstring inside `compute_and_write_aps` at `aps.py:1199-1215` both still describe `--aps-weight-loci` as a placeholder — verbatim from the help text: *"NOTE: locus_weight is currently always 1.0 (placeholder). The weighting machinery is in place but the formula is not yet implemented... This flag is a no-op in the current release but enables the weighted output columns for inspection."* But the actual code at `aps.py:1422-1433` (cycle 15.4a Stage J) now reads `weight_col = 'reliability_score' if 'reliability_score' in contrib_df.columns else 'locus_weight'` — so when reliability scoring is active (which is the default post-cycle-15.4a), `--aps-weight-loci` *does* something real now: each locus's `area_excess` gets multiplied by its `reliability_score` before summing into `APS_area_weighted`.

**Why it stood out:** SPEC15.12's mid-phase amendment paragraph (`PHASE15_SPEC.md:889`) explicitly leans on this flag as the *"partial precursor"* for `[ISSUE:2026-04-29:6]` — meaning cycle 15.6a's design is built on top of the cycle-15.4a wiring. But a user reading the help text would conclude the flag is still inert. There is no version-stamped note in the help text saying "as of v0.14.80 (cycle 15.4a) this flag is live when reliability_score is present." The drift is small, but `--aps-weight-loci` is now load-bearing for SPEC15.12's downstream design and the help-text reality gap could mislead the user picking between Option A (promote to default) vs Option B (side-by-side outputs).

**Likely landing zone:** Stage A of cycle 15.6a's Role 2 implementation (the column-rename sweep already touches this region of `aps.py`); a one-line help-text refresh is in scope as drive-by hygiene. Alternatively SPEC15.20 closeout sweep (cycle 15.10a) if Stage A is too narrowly scoped. The flag's *semantics* should be locked in DECISIONS.md alongside the F4.5 Option A/B/C decision.

**Cross-references:**
- F4.5 in `PHASE15_AUDIT_LOG.md` cycle 15.6a section (the `[ISSUE:2026-04-29:6]` finding).
- `aps.py:1422-1433` (live wiring).
- `onionskin.py:1845-1862` (stale help text).
- `aps.py:1199-1215` (stale docstring).
- `PHASE15_SPEC.md:889` (SPEC15.12 mid-phase amendment paragraph that depends on the live wiring).

**Re-audit postscript [cycle 15.6a R3 closeout, 2026-04-30]:** Active. R2's Stage A
addressed schema migration only and did not refresh the `--aps-weight-loci`
help text or DECISIONS-lock its semantics. The semantics decision is naturally
co-located with cycle 15.6a-S1's reliability-filtered aggregate Option A/B/C
adjudication for SPEC15.12 d3-d4. Primary landing zone: cycle 15.6a-S1 Stage D.
Fallback landing zone if 15.6a-S1 does not sweep it: SPEC15.20 / cycle 15.10a
closeout sweep.

---

## [SURPRISE:15:15.6a:1:A:2] `PHASE15_STRATEGY.md` execution summary still says "21 priorities" post-SPEC15.22-amendment

**Authors:** John M. Urban, Claude Code 2.1.117 (claude-opus-4-7 ; Effort: Max)

**Status:** Active

**Surprise weight:** Low

**What I noticed:** `PHASE15_STRATEGY.md:40` execution summary line reads *"10 cycles total covering 21 priorities (SPEC15.1–SPEC15.21)."* The phase summary at line 10 (and the SPEC15.22 amendment subsection at lines 78-111) says 22 priorities post-amendment. The execution summary line was not updated when SPEC15.22 + cycle 15.4b were inserted on 2026-04-29.

**Why it stood out:** Self-inconsistency within a planning file. Trivial to fix; a reader skimming the execution summary first gets the wrong count. The amendment subsection is correctly appended (per the "Append only — never rewrite earlier rows" rule), but the execution summary apparently does not count as an "earlier row" for that rule's purpose — it is a recap that should evolve with amendments.

**Likely landing zone:** SPEC15.20 closeout sweep (cycle 15.10a) — the tracking-file cleanup pass naturally catches stale priority counts. Or any future amendment can sweep this incidentally.

**Cross-references:**
- `PHASE15_STRATEGY.md:40` (stale execution-summary line).
- `PHASE15_STRATEGY.md:10` (correct post-amendment phase summary).
- `PHASE15_STRATEGY.md:78-111` (the 2026-04-29 amendment subsections).

**Re-audit postscript [cycle 15.6a R3 closeout, 2026-04-30]:** Active. Out of
scope for cycle 15.6a (planning-surface hygiene drift, not in any of the
SPEC15.9-15.12 deliverables). Landing zone: SPEC15.20 / cycle 15.10a closeout
sweep.

---

## [SURPRISE:15:15.6a:1:A:3] Cycle 15.4b's HMM step renumber broke SPEC line-number annotations across multiple SPEC15.X entries

**Authors:** John M. Urban, Claude Code 2.1.117 (claude-opus-4-7 ; Effort: Max)

**Status:** Resolved (v0.14.91) — cycle 15.7a Stage E sweep updated SPEC15.13 d1 (`15-saps/` → `17-saps/` per post-cycle-15.4b layout; verified at `PHASE15_SPEC.md:1001`), SPEC15.13 d3 stale step reference (`PHASE15_SPEC.md:261`), SPEC15.22 d1 step rename map (`PHASE15_SPEC.md:563`), plus the SPEC15.14/15.15/15.16 line annotations in scope for cycle 15.7a. R2's mid-cycle attempt to flip `[SURPRISE:15:15.6a:2:B:1]` was a misattribution; the substantive work belongs here. Drift outside SPEC15.13-16 is **out of scope** for this cycle and remains for cycle 15.10a SPEC15.20 closeout sweep — landing zone path (a) per the original entry. Convention guard path (b) — `multi-agent/AGENT_CONVENTIONS.md` blast-radius checklist amendment — remains a future-phase candidate; deferred-not-resolved is the right framing for it.

**Surprise weight:** Medium

**What I noticed:** I flagged in F2.2 of cycle 15.6a's audit that SPEC15.10 d5 references `hmm_ported_analyses.py:149-151` while live is `304-305` (post-cycle-15.4b SPEC15.22 HMM step renumber + HMM APS reorganization). But the same staleness exists at multiple other SPEC15.X line-number annotations:
- SPEC15.11 d8 references `scripts/aps_cluster_experiments.py:26-45`; live is `26-39` (the 11-experiment grid).
- SPEC15.12 d5 references `hmm_ported_analyses.py:380-439`; live trajectory clustering is at `726-786`.
- SPEC15.11 d1 references "BRAIN15.27 lines 1389-1396" — I did not verify this against the BRAINSTORM file (it was archived during SPEC engineering closeout) but the precision suggests the same staleness risk.

**Why it stood out:** Cycle 15.4b SPEC15.22 had a documented *runtime* blast radius (HMM step relocation; renumber 15→16/16→17/etc.; engine + output_layout + tests + notebooks + PIPELINE_SPEC). The Final Overseer caught five doc-and-template misses (FO-F1 through FO-F5) and the remediation round closed them. But the SPEC body itself — the file that was supposedly "static during implementation" per the v2 contract — has line-number references inside SPEC15.10 + SPEC15.11 + SPEC15.12 priority blocks that the renumber silently invalidated. Nobody updated the SPEC's downstream cross-references because the SPEC was nominally not being touched.

**Why this matters:** The SPEC's line-number annotations are how Role 2 implementers locate the surfaces they need to edit. Cycle 15.6a Role 2 will get pointed at `hmm_ported_analyses.py:149-151` for the HMM-side feature resolver (which is now somebody else's code, or empty space) unless they read the live tree first. My audit caught the SPEC15.10 d5 case explicitly; I did NOT enumerate the same drift class for SPEC15.11 + SPEC15.12.

**Likely landing zone:** Two paths: (a) cycle 15.6a Role 2 sweeps SPEC line-number annotations as a pre-implementation step (and `git diff`s the SPEC at closeout to confirm the sweep is honest about what it changed), OR (b) a future-phase amendment-hygiene rule codified in `multi-agent/AGENT_CONVENTIONS.md` saying *"any cycle that renumbers code structure (steps, paths, line numbers) MUST also sweep SPEC line-number annotations as part of its blast-radius checklist."* The latter is more durable — applies to future renumber cycles too.

**Cross-references:**
- F2.2 in `PHASE15_AUDIT_LOG.md` cycle 15.6a section (the SPEC15.10 d5 case I caught).
- Cycle 15.4b SPEC15.22 in `PHASE15_AUDIT_LOG.md:836` (the renumber that caused the staleness).
- `PHASE15_SPEC.md:755` (SPEC15.10 d5 stale ref).
- `PHASE15_SPEC.md:811` and `821` (SPEC15.11 d8 / Implementation notes references).
- `PHASE15_SPEC.md:860` (SPEC15.12 d5 stale ref).

**Re-audit postscript [cycle 15.6a R3 closeout, 2026-04-30]:** Active. Cycle 15.6a
R2 used the audit's live line numbers (per F2.2 repair instruction) so the
SPEC15.10 d5 case did not bite at implementation time, but R2 did not sweep
the SPEC body to refresh stale annotations themselves. Landing zone for the
SPEC body sweep: SPEC15.20 / cycle 15.10a closeout sweep. The durable
structural fix (an `AGENT_CONVENTIONS.md` rule for renumber-cycle blast-radius)
is a dev-system concern outside Phase 15's product scope; if pursued, it lands
as a `DEVPLAN-*` plan in `plans/next/` rather than as a Phase 15 priority.

---

## [SURPRISE:15:15.6a:1:A:4] Bare `except Exception` swallows step-21 timing-promotion errors silently

**Authors:** John M. Urban, Claude Code 2.1.117 (claude-opus-4-7 ; Effort: Max)

**Status:** Resolved (v0.14.86) — cycle 15.6b R3 re-audit (2026-04-30) verified the narrowing landed correctly at `onionskin_core/engines/hmm_engine.py:1665-1734`: outer broad `except Exception` is removed; only a local `_safe_copy(src, dst)` helper wraps the file-finalization `shutil.copy2(...)` calls, catching only `OSError`. All non-FS operations (`masks_from_timing`, `run_step18_hmm_timing`, `csv.DictReader`, `int(float(raw))`) now propagate up. Stage A.1 evidence: 0/5 toy runs swallowed exceptions (`dev/runs/15-6b-stageA1-run{1..5}.stderr.log`).

**Surprise weight:** Medium

**What I noticed:** `onionskin_core/engines/hmm_engine.py:1721-1722` — the entire active-stage summit refinement block (cycle 15.5a's introduction; lines 1655-1722) is wrapped in `try: ... except Exception as exc: print(f"[hmm] active-stage summit refinement warning: {exc}", file=sys.stderr)`. The step-21 timing promotion at line 1719 (`shutil.copy2(step21_timing, step18_timing)`) sits inside this try-block. If the copy fails — or any of the surrounding code raises (`masks_from_timing` parse failure, file-IO race, csv reader hiccup, the `int(float(raw))` conversion at line 1699 raising `ValueError` for unexpected sliding_offset_bp values) — the user gets a one-line stderr warning and the run continues with stale step18 timing.

**Why it stood out:** This is exactly the kind of error path that a non-deterministic environment (the very thing `[ISSUE:2026-04-30:1]` is investigating) could plausibly trigger. If two `make toy` runs differ in `selected_stages` because one of them silently caught an exception inside this block while the other did not, the variance attribution would point at "strategy selection" when the root cause is actually "swallowed exception." The `try/except` boundary is wide enough (~70 lines of orchestration + 4 nested function calls) that a fault at any line inside it produces the same one-line stderr.

This is pre-existing code from cycle 15.5a, not introduced by anything in cycle 15.6a; my audit did not flag it because it sits outside the SPEC15.9-15.12 surface area. But it is adjacent to C2's determinism investigation in a way that could matter: if cycle 15.6b runs the diagnostic recipe and the variance turns out to be artifact-of-swallowed-exception rather than artifact-of-floating-point-reduction-order, narrowing the try/except to just the operations that genuinely need it (and letting the rest crash loudly) is a much smaller fix than reseeding the whole pipeline.

**Likely landing zone:** Cycle 15.6b (proposed determinism cycle) as a Stage A diagnostic step — *before* assuming the variance is in floating-point reductions or timing/ODW posteriors, instrument this try/except to log every exception trace + verify nothing is being swallowed during normal runs. If no exceptions fire, narrow scope. If exceptions DO fire, the determinism question is separately answered.

Alternatively: a separate small "exception-discipline pass" cycle in the Phase 15 closeout sweep that reviews every bare `except Exception` in `onionskin_core/engines/` and tightens scope. This is more general than C2.

**Cross-references:**
- C2 in `PHASE15_AUDIT_LOG.md` cycle 15.6a section.
- `[ISSUE:2026-04-30:1]` in `multi-agent/tracking/KNOWN_ISSUES.md`.
- `onionskin_core/engines/hmm_engine.py:1655-1722` (the full try/except block).
- Cycle 15.5a section in `PHASE15_AUDIT_LOG.md:7` (cycle that introduced this code).

**Re-audit postscript [cycle 15.6a R3 closeout, 2026-04-30]:** Active. The
orchestrator-locked Stage-A-C-only scope for cycle 15.6a explicitly excluded
this; explicitly tagged for cycle 15.6b (proposed determinism cycle, separately
authorized) Stage A diagnostic per the C2 escalation in cycle 15.6a R1 audit.
Not flipped to In-flight because cycle 15.6b R1 has not yet fired.

---

## [SURPRISE:15:15.6a:1:A:5] `build_aps_shape_matrix` uses alphabetically-first sample as canonical bin grid

**Authors:** John M. Urban, Claude Code 2.1.117 (claude-opus-4-7 ; Effort: Max)

**Status:** Resolved (v0.14.86) — cycle 15.6b R3 re-audit (2026-04-30) verified the B-Branch (v) assertion path landed correctly. Stage A.3 confirmed the invariant holds empirically (`dev/runs/15-6b-stageA3/grid_invariant_checks.json`: 42/42 checks `same_grid: true`). New `assert_shared_track_grid` + `assert_shared_chrom_track_grid` helpers in `onionskin_core/common.py` are wired across `aps.py:457` (build_aps_shape_matrix), `aps_pca.py` (compute_amplicon_layer1_pca + compute_sample_layer2_pca_raw), `engines/growth_model_engine.py` (5 sites), `engines/rcn_mean_shift_engine.py` (2 sites including the audit-named `:241`), and `rcn_mean_shift_helpers.py` (2 sites). Silent-determinism-risk converted into loud `AssertionError` on any future grid divergence.

**Surprise weight:** Low

**What I noticed:** `onionskin_core/aps.py:371` — `ref_sid = sample_ids[0]`. The shape-matrix's canonical bin grid (which determines the column structure of the M×bins shape matrix that feeds clustering) is whatever sample lands first after `sample_ids = [m['sample_id'] for m in metas]` resolves. Sample order comes from the manifest's row order (`compute_sample_rcn_tracks` iterates the manifest in file order); in practice this is alphabetical because the manifest is alphabetical, but the dependency on "alphabetically-first" is implicit, not enforced.

**Why it stood out:** Two related concerns:
1. The SPEC asserts samples share the same bedGraph grid (which justifies the merge at line 405 falling back to `RCN=1.0` for missing bins as defensive). If that assumption were ever to break (e.g., per-sample post-processing diverging the grid), the shape matrix's column structure would silently depend on which sample sorts first. Renaming a sample could change cluster results without anything about the experiment changing.
2. This is the same flavor of "alphabetical-sort-is-load-bearing" pattern that `[ISSUE:2026-04-30:1]` C2 is investigating. Strategy selection might or might not have a similar implicit ordering dependency — and even if it does not, it is worth a "what other places assume sort order?" sweep across the codebase for determinism robustness.

**Likely landing zone:** Cycle 15.6b (proposed determinism cycle) Stage A diagnostic — when surveying determinism risks, audit for sort-order load-bearing patterns. Or: a stand-alone code-hygiene cycle in Phase 15 closeout sweep. If the shared-bedgraph-grid invariant is genuinely guaranteed, the risk is theoretical and a comment explaining the invariant suffices.

**Cross-references:**
- C2 in `PHASE15_AUDIT_LOG.md` cycle 15.6a section.
- `aps.py:371` (the implicit dependency).
- `aps.py:399-406` (the merge that defends against grid divergence).
- `[ISSUE:2026-04-30:1]` in `multi-agent/tracking/KNOWN_ISSUES.md`.

**Re-audit postscript [cycle 15.6a R3 closeout, 2026-04-30]:** Active. Stage A-C
did not address determinism-adjacent sort-order audits; this entry remains
tagged for cycle 15.6b (proposed determinism cycle, separately authorized)
Stage A diagnostic survey. Not flipped to In-flight because cycle 15.6b R1 has
not yet fired.

---

## [SURPRISE:15:15.6a:1:A:6] Empty-locus `mean_rcn` default — back-compat ripple SPEC engineering should lock

**Authors:** John M. Urban, Claude Code 2.1.117 (claude-opus-4-7 ; Effort: Max)

**Status:** Resolved (v0.14.84) by `multi-agent/project_context/DECISIONS.md [2026-04-30] Phase 15 cycle 15.6a — empty-locus mean_rcn and summit_rcn default = 1.0 (diploid baseline)`. Codex implemented `mean_rcn=1.0` in cycle 15.6a Stage A; the DECISIONS entry now locks the choice with full rationale (mathematical consistency + backward-compat under floor-on default + honest semantics under floor-off). The original surprise's ask was for an explicit lock, not a different choice — the lock is now in place.

**Surprise weight:** Medium

**What I noticed:** Today's `_locus_metrics()` at `aps.py:202` returns `mean_excess=0.0` for empty loci (no bins overlap the locus interval). My SPEC15.10 d2 repair instructions in F2.1 of cycle 15.6a's audit propose `mean_rcn=1.0` for the post-rename empty-locus default (matching `summit_rcn=1.0` — diploid baseline). But the choice has a back-compat ripple I did not surface explicitly in the audit:

- **Under floor-on sample-aggregation** (default): `max(mean_rcn - 1.0, 0.0) = 0.0` either way; no behavior change.
- **Under floor-off sample-aggregation** (the new path SPEC15.10 d6 opens up): `mean_rcn=1.0` → `(1.0 - 1.0) = 0.0` per locus → `APS_sum_of_means += 0.0` for empty loci (good). But if SPEC engineering picks `mean_rcn=0.0` instead → `(0.0 - 1.0) = -1.0` per locus → `APS_sum_of_means -= 1.0 × n_empty_loci` (potentially significant).

Either default is defensible:
- `1.0` is consistent with `summit_rcn=1.0` and matches "empty = baseline".
- `0.0` is consistent with the legacy `mean_excess=0.0` and preserves byte-for-byte aggregate parity for runs that contain empty loci (rare but possible — e.g., sparse coverage manifests).

**Why it stood out:** I picked `1.0` in the audit's F2.1 repair instructions but realized later this should be a SPEC engineering decision lock, not a unilateral Role 2 call. Empty loci are rare enough that the ripple may be invisible in test fixtures + toy data, but the decision is real and should be honest.

**Likely landing zone:** A `[2026-XX-XX] Phase 15 cycle 15.6a SPEC15.10 empty-locus mean_rcn default` entry in `multi-agent/project_context/DECISIONS.md`, locked alongside the F3.1 morphology-feature-definitions DECISIONS entry that Stage A already produces. Either choice is fine; the lock is what matters.

**Cross-references:**
- F2.1 in `PHASE15_AUDIT_LOG.md` cycle 15.6a section.
- `aps.py:198-233` (`_locus_metrics()` function).
- SPEC15.10 d2 + d6 in `PHASE15_SPEC.md:749-760`.

**Re-audit postscript [cycle 15.6a R3 closeout, 2026-04-30]:** Active. The live
`_locus_metrics()` empty-locus return at `onionskin_core/aps.py:255-262`
chose `mean_rcn=1.0` (matching this surprise's "diploid baseline"
recommendation). The `[2026-04-30] Phase 15 cycle 15.6a SPEC15.11 morphology
feature definitions` DECISIONS entry locks the morphology + `log10area`
schema but does not enumerate the empty-locus `mean_rcn` choice explicitly.
Landing zone for an explicit DECISIONS lock (if user wants the choice
formalized rather than implicit-via-diploid-baseline framing): cycle
15.6a-S1 alongside the SPEC15.9 default-flip evaluation, OR SPEC15.20 /
cycle 15.10a closeout sweep.

---

## [SURPRISE:15:15.6a:1:A:7] HMM does not emit `aps_amplicon_importance.tsv` — quiet cross-pipeline parity gap

**Authors:** John M. Urban, Claude Code 2.1.117 (claude-opus-4-7 ; Effort: Max)

**Status:** Resolved v0.14.93 (cycle 15.8a). `build_amplicon_importance(...)` ported into `run_hmm_aps()` in `hmm_ported_analyses.py` (SPEC15.17 d12). `APS_CATALOG.md` documents the remaining structural gap (HMM dendrogram plots — deferred to CROSS_PIPELINE_UNIFICATION_SOUP Item 3). R3 independently verified Stage D + Stage J both clean at cycle 15.8a closeout 2026-05-04.

**Surprise weight:** Medium

**What I noticed:** Growth + RMS go through `compute_and_write_aps()` at `aps.py:1133-1563`, which calls `build_amplicon_importance()` at line 1537-1541 to produce `aps_amplicon_importance.tsv` (rank-ordered amplicons by composite onset-earliness + total-growth score). HMM goes through `run_step16_hmm_aps()` at `hmm_ported_analyses.py:180-348`, which is a thinner parallel implementation that does NOT call `build_amplicon_importance` — anywhere. HMM users have no `aps_amplicon_importance.tsv` analog.

**Why it stood out:** This sits squarely inside the user's repeatedly-stated cross-pipeline-parity blind-spot pattern (memory `feedback_cross_pipeline_parity_blindspot.md`). The user has emphasized multiple times that *"the only true difference between pipelines is how they call amplicons"* — and importance ranking is squarely an amplicon-level analysis, not a calling step. The omission has been hiding in plain sight at minimum since cycle 15.4b (when the HMM APS step got its current shape) and likely longer.

It is not the only HMM-side parallel-but-thinner gap I noticed:
- `build_scalar_orderings()` at `aps.py:618-663` — Growth + RMS only.
- `write_aps_plots()` at `aps.py:1547-1560` — Growth + RMS only (HMM has its own plot path under `hmm_notebooks.py` which is structurally different).
- The 8-mode `_FEATURE_COL` dict at `aps.py:608-615` — HMM has a 2-mode duplicate at `hmm_ported_analyses.py:304-305`.

These are all individually small but the pattern is real: HMM's APS analysis surface is structurally smaller than Growth + RMS's, and each gap got there by some prior cycle deciding "we'll port this to HMM later." SPEC15.17 / cycle 15.8a (APS analysis-surface parity + master APS catalog) is supposed to catch all of this — but if I am noticing it from cycle 15.6a's perspective without trying, the pattern is more pervasive than the SPEC15.17 wording captures.

**Likely landing zone:** SPEC15.17 / cycle 15.8a — the master APS catalog file deliverable (`multi-agent/full_instructions/APS_CATALOG.md` per JQ30 lock) is the right home for systematically enumerating "Universal vs HMM-only vs Growth+RMS-only" surfaces. Worth flagging to that cycle's R1 auditor that the gap inventory is likely longer than the SPEC's language implies.

**Cross-references:**
- F7 in `PHASE15_AUDIT_LOG.md` cycle 15.6a section (cross-pipeline parity blind-spot guard).
- `feedback_cross_pipeline_parity_blindspot.md` in user memory.
- SPEC15.17 in `PHASE15_SPEC.md:1083` (cycle 15.8a — APS analysis-surface parity).
- `aps.py:1133-1563` (full Growth+RMS APS path).
- `hmm_ported_analyses.py:180-348` (HMM thin APS path).

**Re-audit postscript [cycle 15.6a R3 closeout, 2026-04-30]:** Active. Out of
scope for cycle 15.6a (cross-pipeline analysis-surface parity is structural,
not per-cycle). Landing zone: SPEC15.17 / cycle 15.8a (master APS catalog).
Surface to that cycle's R1 auditor: the gap inventory likely runs broader than
SPEC15.17's current language captures (also missing on HMM:
`build_scalar_orderings`, `write_aps_plots`, the formerly-duplicated
`_FEATURE_COL` dict — the last of which cycle 15.6a Stage C resolved by
factoring `build_aps_feature_matrix` into a shared resolver).

---

## [SURPRISE:15:15.6a:1:A:8] Drift assessment — cross-pipeline structural divergence keeps slipping past audits despite explicit user direction

**Authors:** John M. Urban, Claude Code 2.1.117 (claude-opus-4-7 ; Effort: Max)

**Status:** Promoted to `multi-agent/plans/next/CROSS_PIPELINE_UNIFICATION_SOUP.md` + `[ISSUE:2026-04-30:2]` (2026-04-30)

**Postscript (2026-04-30, post-cycle-15.6a closeout):** This high-weight meta-observation has been **promoted to a dedicated SOUP file** (`multi-agent/plans/next/CROSS_PIPELINE_UNIFICATION_SOUP.md`) that captures the cross-pipeline structural unification theme as a future-phase candidate. The SOUP catalogs four specific items: (1) `build_hmm_steps()` semantic-key refactor + 242 engine call site updates; (2) function/parameter/variable renames cascading from Item 1; (3) HMM-thinner-APS-path surfaces audit + port (`build_amplicon_importance`, `build_scalar_orderings`, `write_aps_plots`); (4) abstract-base-class scaffold to prevent future drift. Plus Item 5: convention guards in `AGENT_CONVENTIONS.md`.

The concrete actionable subset (HMM step-number leakage in code identifiers across function names, dict keys, parameters, variables — Categories 1-3 of the SOUP's Items 1-2) has been filed as **`[ISSUE:2026-04-30:2]`** in `multi-agent/tracking/KNOWN_ISSUES.md` with explicit per-cycle in-phase fix guidance for Phase 15's remaining cycles (cycle 15.7a: mandatory in-flight rename of `run_step17_hmm_saps` → `run_hmm_saps`; cycle 15.9a: bulk of HMM-pipeline-specific Category 2+3 renames as SPEC15.21 sub-deliverable; future phase: bulk Category 1 refactor).

A new `AGENT_CONVENTIONS.md` § Naming conventions rule was added 2026-04-30 to prevent future step-number-in-code-identifier drift.

The original four-hypothesis interpretation analysis (a/b/c/d below) is preserved for provenance — the user's effective direction was a hybrid of (b) and (c): the workflow needed a cross-cycle drift-detection mechanism (Template C closeout review gates + this SURPRISE_LOG file are the workflow-level fix; the structural-unification SOUP is the codebase-level fix).

**Surprise weight:** High

**What I noticed:** This entry is broader than a single observation — it is a fresh-eyes opinion on a pattern that emerges when the cycle 15.6a audit perspective is held against the entire onionskin program intent.

The user's stated intent (sourced from `CLAUDE.md`, multiple memory files including `feedback_cross_pipeline_parity_blindspot.md`, and recurring direction at brainstorming-stage rounds): *"the only true difference between pipelines is how they call amplicons."* Everything downstream of amplicon-calling — APS, timing, summit refinement, reliability, gap analysis, clustering, plots, notebooks — should be a uniform analysis surface across Growth, RMS, and HMM, with pipeline-specific exceptions explicitly documented.

What I observe in the live tree (HEAD `4e09f4f`) is a different reality:

1. **HMM has its own parallel-but-thinner copies of multiple analysis modules.** `run_step16_hmm_aps` is a thinner version of `compute_and_write_aps`. The two `_FEATURE_COL` dicts. The trajectory-clustering machinery at `hmm_ported_analyses.py:726-786` has no Growth/RMS analog (and arguably should not — fork-trajectory is HMM-native — but the boundary between "HMM-specific" and "should-be-universal-but-was-only-built-for-HMM" is blurry).

2. **Cross-pipeline ports keep getting filed as discrete cycles.** Cycle 15.4b's SPEC15.22 ported `_finalize_amplicon_recommendations` from growth-only to all three pipelines. Cycle 15.5a's SPEC15.7 ported summit-refinement strategy menu cross-pipeline. Cycle 15.5a's SPEC15.8 ported summit↔timing convergence cross-pipeline. Cycle 15.4a's SPEC15.6 ported reliability scoring cross-pipeline. SPEC15.18 (cycle 15.8a) is queued to port `--peak-summary` to RMS + HMM. **Each of these is filed as a discrete cross-pipeline port priority, not as remediation of a deeper structural decoupling.** The audit-implement-reaudit loop catches each instance but does not catch the pattern.

3. **The duplicated `_FEATURE_COL` dict in `hmm_ported_analyses.py:304-305` is a microcosm.** Two dicts. Two dispatch logic paths. Two places to update when SPEC15.10 renames `summit_excess` → `summit_rcn`. Two places that my F2.2 finding caught as an audit miss in SPEC15.10's deliverables (the SPEC said *"Verify no other pipeline-side resolver lags via grep for `summit_excess` / `peak_rcn` / `mean_excess` after the rename lands"* — anticipating exactly the drift, but treating it as an after-the-rename verification step rather than an upstream "why are there two dicts?" question).

4. **The `feedback_cross_pipeline_parity_blindspot.md` memory entry exists because the pattern keeps recurring.** Memory `feedback_cross_pipeline_parity_blindspot.md`: *"User has emphasized multiple times that 'X-pipeline-only' defects systematically slip through audits despite his repeated efforts to expose them."* The memory is dated and explicit; it has been an active feedback signal for multiple cycles. And yet cycle 15.6a's audit (which I just wrote) caught the issue at the F2.2 + F3.5 + F7 level — i.e., as in-cycle remediation steps for SPEC15.10 + SPEC15.11 — rather than questioning whether the structural decoupling itself is the root cause of the recurring pattern.

**Why it stood out:** Standing back from the immediate cycle, this looks less like "a sequence of independent audits caught discrete defects" and more like "the project has accumulated structural decoupling between pipelines that no one cycle is sized to address, so it keeps surfacing as the same class of finding under different names." Each cycle that adds a cross-pipeline port closes one gap and reveals (or creates) the next one. Phase 15 alone has at least 5 cross-pipeline-port priorities (SPEC15.4 ODW, SPEC15.6 reliability, SPEC15.7 summit refinement, SPEC15.8 summit↔timing convergence, SPEC15.18 `--peak-summary` extension); Phase 14 had similar. The remediation cadence is roughly one cross-pipeline port per audit cycle.

**Possible drift interpretations** (any of these could be wrong; surfacing them for the user's reaction, not asserting any of them):

- **(a) Hypothesis: HMM was built to a different spec than Growth + RMS, and the resulting structural decoupling is being slowly reconciled cycle-by-cycle.** If true, the recurring pattern is expected and the project is on track — each cycle moves the needle. Phase 15's "HMM completeness" theme is consistent with this reading.
- **(b) Hypothesis: The audit-implement-reaudit workflow's per-cycle scope keeps the lens too narrow to see the structural decoupling. Each finding gets filed under "this specific surface is X-pipeline-only" and never escalates to "the X-pipeline-onlyness itself is the defect."** If true, the workflow needs a *cross-cycle* drift-detection mechanism — perhaps a periodic Role 3 wrap-up audit that explicitly looks for the recurring-finding-class pattern.
- **(c) Hypothesis: The codebase's module structure has not been refactored to enforce cross-pipeline parity at the type level. There is no `BasePipelineAPS` interface that all three implementations must satisfy. Without that scaffold, parity has to be checked at audit time — which is exactly the workflow's recurring failure mode.** If true, a Phase 16 (or later) "cross-pipeline base-class refactor" cycle could collapse the recurring port priorities into a one-time structural fix. But that is invasive and probably not what the user wants for a research codebase.
- **(d) Hypothesis: The recurring pattern is fine and the user knows it.** The `feedback_cross_pipeline_parity_blindspot.md` entry is a heads-up for me, not a complaint. Each cycle's port priority IS the remediation; the user is tracking the shrinking-gap progress. The "blind-spot" framing is about agents missing it during audits, not about the project being structurally off-course.

I cannot tell from inside one cycle which of (a)/(b)/(c)/(d) is closest to the truth. My weak prior is some mix of (b) and (d): the workflow does keep audits narrow, and the user is actively tracking the recurring-port progress. But (c) is real too — a `BasePipelineAPS` shaped interface would prevent future drift even if it does not resolve the current backlog.

**Likely landing zone:** None obvious. This is a meta-observation that does not fit cleanly in any cycle's repair contract. Possible homes if the user wants to engage with it:
- A note added to `multi-agent/AGENT_CONVENTIONS.md` formalizing a "cross-pipeline-parity sweep" expectation for any cycle that touches APS / timing / clustering / plots / notebooks — would give the workflow a built-in reminder to look for X-pipeline-only patterns at audit time.
- A Phase 16+ entry in `multi-agent/tracking/BRAINSTORM.md` for "cross-pipeline structural unification" if (c) resonates with the user.
- Acknowledged-and-logged-only if (a) or (d) is the correct read.

**Cross-references:**
- `feedback_cross_pipeline_parity_blindspot.md` in user memory.
- F7 in `PHASE15_AUDIT_LOG.md` cycle 15.6a section.
- SPEC15.17 in `PHASE15_SPEC.md:1083` (the only Phase 15 priority that frames cross-pipeline parity systematically — and it is a catalog deliverable, not a structural fix).
- Cycle 15.4a + 15.4b + 15.5a + 15.6a + 15.8a in `PHASE15_STRATEGY.md` (the cross-pipeline-port priority chain).

**Re-audit postscript [cycle 15.6a R3 closeout, 2026-04-30]:** Active. Per the
orchestrator's explicit instruction at re-audit time, this meta-observation is
**surfaced to the orchestrator for adjudication** rather than being unilaterally
flipped to Logged-only or Promoted. The four hypotheses (a)/(b)/(c)/(d) need
user reaction. Possible dispositions: (1) **Logged-only** if (a) or (d) is the
correct read (existing per-cycle cross-pipeline-port priorities collectively
suffice; user is tracking the recurring-port progress as expected); (2)
**Promoted to a Phase 16+ BRAINSTORM entry** (cross-pipeline structural
unification phase; e.g., `BasePipelineAPS` shaped interface) if (c) resonates;
(3) **Promoted to an `AGENT_CONVENTIONS.md` amendment / `DEVPLAN-*` plan** if
(b) resonates (workflow needs cross-cycle drift-detection for the
"X-pipeline-only finding-class" pattern). The flip to a terminal status is
deferred to the user's adjudication at orchestrator triage. Adjacent partial
mitigations already landed in cycle 15.6a (the shared
`build_aps_feature_matrix` resolver collapsed the duplicated `_FEATURE_COL`
dict — one microcosm of the pattern this surprise flagged).

---
