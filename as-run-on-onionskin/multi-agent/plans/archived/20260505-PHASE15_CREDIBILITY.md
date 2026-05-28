# State-path-aware credibility deliberation: SPEC15.13 d7 metric-definitions hammer-out

**Created:** 2026-05-03 EDT — to capture the Principal-orchestrator hammer-out conversation locking concrete metric definitions for SPEC15.13 d7's HMM-specific state-path-aware reliability extensions ("state-path persistence, ghost-stage count weighting, multi-step state-path evolution scoring, fork-travel monotonic-growth integration"). Cycle 15.7a R1 (commit `ca896f6`) surfaced 8 hammer-out questions Q1-Q8 in `PHASE15_AUDIT_LOG.md` cycle 15.7a section § Principal hammer-out subsection. The file's purpose is to avoid the failure that motivated this hammer-out: a prior agent had an extensive d7-design conversation with the Principal that left only breadcrumbs in planning files (per Principal observation 2026-05-03). This file captures the conversation + the orchestrator's research as it happens, so successor agents have a substantive trail.

**Lifecycle:** Per-phase planning artifact. Tracked. Archives alongside SPEC / AUDIT_LOG / STRATEGY / FEEDBACK / BACKGROUND / SURPRISE_LOG at phase close (`multi-agent/plans/archived/<YYYYMMDD>-PHASE15_CREDIBILITY.md`). Do NOT delete. Scope: SPEC15.13 d7 metric-definitions hammer-out only — NOT a general phase synthesis surface; cycle narratives belong in AUDIT_LOG.

**Sibling-file pattern:** mirrors `PHASE15_BACKGROUND.md` — topic-specific scratchpad created mid-phase, archived with phase. Per Principal direction 2026-05-03.

**See also:**
- `multi-agent/plans/PHASE15_AUDIT_LOG.md` cycle 15.7a § Principal hammer-out subsection — the formal Q1-Q8 surface.
- `multi-agent/plans/PHASE15_SPEC.md` SPEC15.13 d7 (line 1018-1024) — placeholder-pending-Principal-deliberation contract being hammered out.
- `multi-agent/plans/PHASE15_SPEC.md` SPEC15.5 (lines 373-437) — the predecessor priority that landed the per-amplicon state-path-evolution checks (cycle 15.4a, closed v0.14.80) that d7 is supposed to consume / layer on top of.
- `multi-agent/plans/archived/20260428-PHASE15_FEEDBACK.md:268-269` — Principal's original verbatim discussion of state-path evolution + the 4 checks (the "long rich conversation" the Principal referenced — IT IS RECORDED, not lost).
- `multi-agent/plans/archived/20260428-PHASE15_BRAINSTORM.md:899` — pyramid-shape check anchor.
- `multi-agent/tracking/BRAINSTORM.md` `[2026-04-18] SAPS` entry (lines 1752+) — BRAIN15.20 SAPS source material.
- `multi-agent/tracking/KNOWN_ISSUES.md` `[ISSUE:2026-05-03:1]` — reliability + importance scoring redesign + relocation; post-Phase-15 scope.
- `onionskin_core/hmm_multistage_unification.py:357-396` — `_state_path_checks` function (already implemented at cycle 15.4a).
- `onionskin_core/hmm_fork_travel.py:188, 234-243` — `ghost_level_flag` per-stage primitive.

---

## CRITICAL ORCHESTRATOR FINDING (2026-05-03 — surfaced via search after Principal Q1-Q3 responses)

**Q1-Q3 concepts ARE NOT new.** They were extensively designed by the Principal in 2026-04-28 FEEDBACK and IMPLEMENTED in cycle 15.4a (closed v0.14.80) as part of SPEC15.5(c) + (e).

The "long rich conversation" the Principal referenced IS recorded — at `multi-agent/plans/archived/20260428-PHASE15_FEEDBACK.md:268-269` — and the resulting design landed in code at `onionskin_core/hmm_multistage_unification.py:_state_path_checks` (lines 357-396).

### Existing per-amplicon (per-trajectory) columns — already computed, emitted, and used internally by HMM cycle 15.4a

Verified from `onionskin_core/hmm_multistage_unification.py` 2026-05-04. The `_state_path_checks(trajectory_id, fork_df, nlevels_by_stage, stages)` function at lines 374-416 operates per-trajectory across stages (NOT per-sample). It returns 8 columns:

| Column | Concept | Type | Implementation |
|---|---|---|---|
| `state_path_persistence_after_popup` | Q1's persistence | bool | `_persistence_after_popup` at line 357 |
| `state_path_width_grows` | Q3 width evolution | bool | inline at line 393 (`_nondecreasing` + `_has_positive_delta` over per-stage widths) |
| `state_path_more_than_one_step` | Q3 multi-step evolution | bool | inline at line 394 (`max(levels) > 1`) |
| `state_path_pyramid_shaped` | Q3 pyramid structure | bool | `_pyramid_shaped` at line 337 |
| `state_path_level_growth_monotonic` | height-growth | bool | inline at line 396 |
| `state_path_fork_travel_monotonic` | Q4 fork-travel monotonicity | bool | inline at line 397-399 |
| `ghost_level_count` | Q2 ghost-stage count | int | sums `ghost_level_flag` per-stage values from fork data |
| `state_path_growth_credibility_score` | aggregate score | float [0,1] | `(positives_count / 6) - min(0.35, 0.05 × ghost_count)` |

**Output files (already written):**
- `<hmm-out>/14-multistage-unification/all_trajectories.multistage_unification.tsv` (full schema; 23 columns; line 583).
- `<hmm-out>/14-multistage-unification/all_trajectories.state_path_credibility.tsv` (focused; 11 columns; line 600).
- `<hmm-out>/14-multistage-unification/all_trajectories.multistage_unification.bed` (browser track; line 620).

**Internal uses (the score IS used):**
- `state_path_growth_evidence` — derived boolean indicating "did state-path checks classify this as a real amplicon."
- `state_path_unification_pass` — feeds the `--multistage-unification-mode` flag's mode 2 (state-path-only) / mode 3 (both-required) / mode 4 (either-suffices, default).
- The `filter_keep` final amplicon pass/fail decision incorporates `state_path_unification_pass`.

**What does NOT yet consume these (the gap for "surfacing"):**
- `onionskin_core/reliability.py` `_compute_reliability(amplicon_class)` at line 176 builds `reliability_score` from a `scores` array that does NOT include any state-path-credibility column. The cross-pipeline reliability sidecar `amplicon_reliability.tsv` does not carry the state-path columns.
- The `<aps-out>/tracks/` IGV-track emission added cycle 15.6a-S1 doesn't include state-path credibility tracks.

### Reframing implication for SPEC15.13 d7

SPEC15.13 d7 is NOT being asked to define these from scratch. d7's job is to layer **per-sample** state-path-aware reliability extensions on top of these existing **per-amplicon** outputs, where "per-sample" = consuming SPEC15.13 SAPS data substrate (per-sample HMM state-path decoding from SPEC15.3) — hence d7's first sentence: *"consume SPEC15.3 per-sample state-path outputs + SPEC15.13 SAPS data."*

The R1 audit's Q1-Q3 framing missed this nuance. It asked the Principal to define concepts that were already locked in cycle 15.4a, treating the SPEC body's d7 placeholder language as if it introduced new concepts. The placeholder language was actually paraphrasing the 2026-04-28 FEEDBACK conversation, not introducing new concepts — but the paraphrase lost the "per-sample analog of existing per-amplicon checks" intent.

### Three open framings for d7's actual scope (Principal picks)

**(i) Per-sample analogs of the cycle-15.4a per-amplicon checks (consuming SAPS data).**

Why this exists as an option: cycle-15.4a's `_state_path_checks` operates on the **JOINT HMM run** — fork/state-path data aggregated per stage (one consensus state path per stage from the joint HMM decode of all samples in that stage). SAPS (SPEC15.13 d1-d6) gives **per-sample** state-path decoding — each sample s gets its own state path per stage. So we CAN ask: "for sample s, does s's individual state path at amplicon a show persistence-after-popup across stages?" using SAPS per-sample data. The cross-stage evaluation is the same; the new dimension is per-sample variability. Aggregate across samples per amplicon: e.g., `state_path_persistence_per_sample_aggregate = fraction_of_samples_where_individual_path_persists`.

What "more science" means concretely:
- **Catches cross-sample heterogeneity hidden by joint consensus.** Joint HMM might say "persistent" because 70% of samples agree; the 30% disagreement is meaningful signal a per-sample column would surface.
- **Distinguishes confident from ambiguous calls.** Per-sample-aggregate near 0 or 1 = clear; near 0.5 = ambiguous → useful for downstream filtering decisions.
- **Cross-sample coherence as its own credibility signal.** A real amplicon should show coherent evidence across most samples; spurious calls may show evidence in only outlier samples.

What "the SPEC body's per-sample framing" means: SPEC15.13 d7 paragraph 1 (`PHASE15_SPEC.md:1020`) reads *"consume SPEC15.3 per-sample state-path outputs + SPEC15.13 SAPS data."* That input contract implies per-sample dimension is intended; the placeholder language paraphrasing the 2026-04-28 FEEDBACK lost the explicit "per-sample analog of per-amplicon checks" wording, which is the pattern that makes "consume per-sample state-path outputs" actually meaningful work.

Cost: each sample × amplicon × 4 checks = ~50 × 50 × 4 = ~10,000 boolean computations + aggregation per dataset (small but non-trivial; SAPS data must land first per Stage A).

Caveat (honest pitch): the existing joint checks may already capture most of the signal. Per-sample analogs could be a marginal improvement, not transformative. Worth doing if cross-sample heterogeneity matters; not worth doing if (ii) consumption + width-threshold is enough.

**(ii) Pure consumption of existing per-amplicon checks** — no per-sample variant. Surface the cycle-15.4a `state_path_growth_credibility_score` (and its 6 boolean components + ghost_count) as new columns in the SPEC15.6 cross-pipeline reliability sidecar (`amplicon_reliability.tsv`) for HMM rows. Optionally add IGV tracks. Lighter implementation; the scores already exist — just need joining onto the reliability sidecar.

Gap even for (ii): the new width-threshold credibility prior (your 2026-05-04 Q3 supplement — < 50-100 kb gets reduced trust regardless of stability pattern) is NOT implemented. Would need a new column (e.g., `state_path_latest_stage_width_bp` + a soft-trust score column) added to the existing `_state_path_checks` outputs OR computed at consumption time.

**(iii) Something else** — cross-references `[ISSUE:2026-05-03:1]` for the post-Phase-15 redesign discussion. **Hybrid is also possible**: do (ii) now in cycle 15.7a + defer (i) to a follow-up cycle once cycle-15.7a SAPS data is in hand to evaluate whether per-sample analogs add real value.

**Hammer-out re-anchor:** Q1-Q4 should be re-read as "per-sample variants of the existing per-amplicon checks" rather than "define these concepts from scratch." The Principal's Q1-Q3 responses are consistent with this re-anchoring (your gut on Q1 matched `state_path_persistence_after_popup`'s logic; Q2 ghost ↔ persistence complement; Q3 the 4 checks).

---

## Historical record — prior Principal discussions + design source material

This section preserves verbatim source material so successor agents have the full context, not just the orchestrator's distillation.

### A. Principal's 2026-04-28 FEEDBACK discussion of state-path evolution + collapsed-repeat discriminator (verbatim)

Source: `multi-agent/plans/archived/20260428-PHASE15_FEEDBACK.md:265-269`. Context: Q&A round during Phase 15 BRAINSTORM-to-SPEC engineering. The Principal's verbatim exposition of state-path evolution that became SPEC15.5(c) + (e) and was implemented in cycle 15.4a (closed v0.14.80).

> "[265] In that case, there are more false positive classifications for collapsed repeats, which is okay and the expected behavior and result.
> [266] The multistage unification process actually allows us to 'rescue' amplicons that were falsely labeled as a collapsed repeat in one or more stages because we are using the evolution of the amplicon across multiple stages to update our view on it.
> [267] The same exact thing needs to be part of the HMM pipeline
> [268] In addition, the HMM pipeline allows us to monitor state path evolution across the stages for the final multistage unification of amplicon calls vs collapsed repeat calls. Both ideas should be part of the HMM pipeline, and multistage unification can default to one of a few behaviors: (1) use only the triangle filter based results, (2) use only the state path evolution based results, (3) only classify candidates as amplicons if they are classified as amplicons in both methods, (4) include any candidate that is classified as an amplicon in either method. It can default to 4 (least stringent filter). We can change that default if it turns out to be too permissive. Regardless of classification decision, both the multistage triangle analysis and multistage state path evolution analysis should be run and reported on (e.g. results in columns of a table for each amplicon)
> [269] The state path evolution idea to better classify candidate regions marked as above background in any stage is elaborated on elsewhere, but involves a very similar logic. When multiple stages are present, above background amplicon candidates are not excluded prior to evaluating them with respect to all stages. If there is only one stage, then filters for things like 'length' (for example) might be needed to help eliminate false positives (e.g. anything < 50 kb with only one step up) at the expense of false negatives. But with multiple stages, we can more confidently say which regions are truly amplicons. Then we would not want to exclude any state path information from those regions. A very early stage might, for example, have a single step interval that is < 50 kb -- it might only be 10 kb -- but we would know that is really summit (origin) location information. In contrast, a region that is single-step and the same length (whether it is 10 kb or 400 kb) across all stages would more likely be a collapsed repeat. BUT if a single step pops up in the latest stage (and is not present in the stages before it) - we again would not treat that like a collapsed repeat because it was not present across all stages. Anything that pops up late and persists looks real, especially if it also passes the triangle filter. So with the statepath evolution, for each candidate we are looking to see: (1) does it persist after it pops up? (2) if it is present in multiple stages, does it get wider across them? (3) is it is present in multiple stages, does it evolve to have more than one step? (4) if it has more than one step, is it pyramid-shaped (with the summit interval approximately centered between steps on both sides)? For amplicons that start early and grow tall, all checks will pass (and it will survive the triangle multistage process as well). For amplicons that start early (or any stage except the last stage), but do not grow in height, they would need to at least pass checks 1 and 2, and pass the multistage triangle analysis (if triangle results were instructed to be used in the CLI flag discussed above). For amplicons that appear in only one stage, it would have to be the last stage to satisfy check 1, and it would have to pass the multistage triangle analysis (if triangle results were instructed to be used in the CLI flag discussed above)."

### B. SPEC15.5(c) + (e) — what got promoted from FEEDBACK into the SPEC

Source: `multi-agent/plans/PHASE15_SPEC.md:392-420` (live, not archived — SPEC15.5 already DONE at v0.14.80).

> **(c) HMM-specific state-path growth credibility ("ghost levels").** Detect growing state paths (nesting depth increases across stages → high credibility) vs flickering state paths (level appears/disappears → low credibility / "ghost levels"). HMM-native signal not available from RMS or growth. New HMM credibility feature feeding the final amplicon set.
>
> **(e) State-path-evolution checks for amplicon-vs-collapsed-repeat.** Run these checks against multistage state-path data:
> 1. Does the candidate persist after it pops up?
> 2. If present in multiple stages, does it get wider across them?
> 3. If present in multiple stages, does it evolve to have more than one step?
> 4. If multi-step, is it pyramid-shaped (summit interval approximately centered between steps on both sides)?
>
> Decision rules (verbatim user spec, BRAIN15.18):
> - Amplicons that start early and grow tall: all checks pass + passes multistage triangle process. → amplicon.
> - Amplicons that start early but don't grow in height: must pass at least checks 1 + 2 AND pass multistage triangle analysis (if triangle results selected by combined flag). → amplicon.
> - Amplicons that appear in only one stage: must be the LAST stage AND must pass multistage triangle analysis. → amplicon (single-stage, late-onset).
> - Anything single-stage that isn't the last stage AND doesn't grow: collapsed-repeat-likely.
> - Single-step regions of the same length across all stages: collapsed-repeat-likely.
>
> **Within-ODW non-monotonicity (BRAIN15.18 Round 5 update):** all checks above operate across the full ODW (BRAIN15.24 framework), allowing for plateau-mid-window patterns. The within-ODW per-transition active-growth test consumed here is the **post-SPEC15.4 framework** (Bayesian ODW System), NOT the legacy 1.25× rule.
>
> **Single-stage vs multistage filtering (BRAIN15.18 user-quoted Q9):** in multistage runs, do NOT exclude short/single-step regions before running multistage analysis — the analysis itself classifies them. In single-stage runs, fall back to length / step-count heuristic filters because the multistage signal isn't available.

### C. BRAIN15.20 SAPS source — `tracking/BRAINSTORM.md` [2026-04-18] (verbatim)

Source: `multi-agent/tracking/BRAINSTORM.md:1752-1779`.

> ## [2026-04-18] SAPS — State-APS from individual-sample HMM decoding
>
> **Status:** Vision / exploratory. Prerequisite = parallel child pipeline (see above).
> **Updated priority: HIGH** — now a near-term deliverable after the child pipeline is implemented.
>
> **What it is:** SAPS (State APS) computes APS on each sample's decoded HMM state path instead of on raw RCN values. This gives an amplification score that is HMM-posterior-denoised rather than raw-coverage-based.
>
> **Pipeline concept (updated):**
> 1. Use `03-removeZeroBins/indiv_samples/` per-sample bedGraphs.
> 2. Use `04-medNormOrRCN/indiv_samples/` chrom-median renorm.
> 3. Use `05-medianSmoothedRCN/indiv_samples/` (already smoothed — skip step-14 re-smoothing).
> 4. Run HMM steps 6→7→8→9→10→11 per sample (state paths in `06-HMM/indiv_samples/`).
> 5. SAPS at `15-saps/` uses state paths from step 6.
>
> **SAPS metrics per locus:**
> - `saps_area` = Σ (state_value − 1) × bin_width, where state_value is the decoded HMM state
> - `saps_summit` = summit_state + summit_length / 1,000,000 (for tie-breaking)
> - `saps_width` = total bp in state > 1 (above-background)
> - `saps_shape` = per-bin state value vector (profile shape)
>
> **HMM state encoding reminder:** States are 1-based. State 1 = background. State ≥ 2 = amplified. `--hmm-background-idx 0` means background is at index 0 internally but output is 1-based.
>
> **Output location:** `15-saps/` (timing bumped to 16, clustering to 17).

**Note on state encoding:** the BRAIN15.20 quote above predates SPEC15.14's 0-based default flip (locked 2026-05-03). Once SPEC15.14 lands at cycle 15.7a, the "States are 1-based" reminder updates to "States are 0-based by default; state 0 = background; legacy 1-based via `--hmm-statepath-base 1`." SAPS implementation must be label-dictionary-aware from the start per F1 step 6 in cycle 15.7a R1 audit.

### D. BRAINSTORM "HMM-native amplicon quality criteria" [2026-04-18] (verbatim)

Source: `multi-agent/tracking/BRAINSTORM.md:1782-1803`. Same date as BRAIN15.20; sister entry that names the SAME 4-check concepts the FEEDBACK 269 quote elaborates on.

> ## [2026-04-18] HMM-native amplicon quality criteria
>
> **Status:** Vision / exploratory. Not implemented; possibly replaces or augments growth-model BIC chain for HMM-only calls.
>
> **Motivation:** The current HMM pipeline borrows growth-model BIC/shape criteria for amplicon filtering. For amplicons that are HMM-detected but growth-dark, or for improving HMM-only confidence, HMM-native quality signals would be more principled.
>
> **Proposed criteria:**
> 1. **Widening**: above-background region widens across developmental stages
> 2. **Summit increase**: highest decoded state ever increases (not just appears)
> 3. **Hill shape**: HMM state profile shows rise→peak→fall structure (2→3→2+, etc.)
> 4. **Persistence**: amplicon is detected continuously from first-observed stage onward
> 5. **Shape filter**: triangle vs flat classification in maximal-amplification stage (outputs a BED file marking flat amplicons as lower-quality or unreliable)
>
> **Output ideas:**
> - `hmm_amplicon_quality.tsv` — per-amplicon flag matrix for each criterion
> - `hmm_flat_amplicons.bed` — amplicons that fail triangle filter in maximal stage
> - `hmm_persistent_amplicons.bed` — subset that satisfy persistence from first detection

**Mapping to cycle-15.4a implementation:** criterion (1) Widening = `state_path_width_grows`; (2) Summit increase = `level_growth`; (3) Hill shape = `state_path_pyramid_shaped`; (4) Persistence = `state_path_persistence_after_popup`; (5) Shape filter = SPEC15.5(a) HMM shape-filter sink (separate deliverable). All landed at cycle 15.4a (v0.14.80) per `_state_path_checks` aggregator at `hmm_multistage_unification.py:357-396`.

### E. BRAINSTORM "Amplicon reliability scoring and flat-sample detection before APS/clustering" [2026-04-19] (relevant excerpt)

Source: `multi-agent/tracking/BRAINSTORM.md:1855-1907`. Context: this entry became BRAIN15.7 → SPEC15.6 (cross-pipeline reliability scoring; closed cycle 15.4a v0.14.80 + supplemental 15.4a-S2/S3/S4). The SAPS-relevant + HMM-state-path-aware-extension lines:

> **Application to both pipelines:**
> - Growth pipeline: apply the triangle BIC, parabola height, width/area metrics directly to calls from `_origins.tsv` and stage-median bedGraph profiles.
> - HMM pipeline: same metrics plus state-path–based QC (state occupancy across stages, summit state persistence, etc.) in a second pass. **HMM-specific metrics designed separately.**
>
> **Scope and priority:** First pass implements the pipeline-agnostic metrics (1–5 above plus flat-sample assignment). Second pass adds HMM-specific state-path metrics. This is a significant feature addition — likely its own Priority under Phase 11 or Phase 12.

**Critical context:** the 2026-04-19 BRAINSTORM explicitly anticipated a "second pass" of HMM-specific state-path metrics that **come AFTER the cross-pipeline shared scorer**. The shared scorer landed at SPEC15.6 / cycle 15.4a (v0.14.80). The "HMM-specific metrics designed separately" is what SPEC15.13 d7 is supposed to be — the deferred second pass. So d7's role is to consume SAPS data + add HMM-specific state-path-aware reliability columns ON TOP OF the shared scorer's columns. This further supports the (i) per-sample-analogs framing in the open question above.

---

## Q1 — `state_path_persistence`

### R1's framing (audit-log lines 306-316)

Per-amplicon definition. Candidate framings:
- (a) Per-sample run-length-derived. `r_max(s, a) / K`. Higher = locally stable.
- (b) Per-amplicon state-stay-probability under posterior. Mean posterior confidence at each bin within amplicon.
- (c) Cross-stage state-path consistency. Distinct state values at amplicon's central bin across samples.
- (d) Hybrid OR Principal-supplied alternative.

R1 made no recommendation; flagged as the "most-needs-input" question.

### Principal's response (2026-05-03 chat)

> "I actually have no idea what this state_path_persistence thing is for or what it is trying to measure, and for what purpose... This is the first time I've seen this term... and the first time I've seen this phrase, 'state path is locally stable for this amplicon in this sample'."

> Best guess: persistence = "once an amplicon is detected in stage s, a real amplicon is expected to persist through all subsequent stages. Amplicons do not just disappear. So if a certain stage (or sample) has an amplicon candidate, but it is a oneoff unique to that stage or sample compared to the stages that come after it, then it was likely a false positive."

Two candidate scoring schemes the Principal raised:
- **Percent**: "percent of subsequent stages that also identify an amplicon at that spot. The expectation for a true amplicon is 100%." Weakness: a 100% score for 1 subsequent stage is the same as 100% for 4 stages.
- **Count**: "count of stages it persists in after its first appearance." Opposite problem: gives early amplicons more points than late amplicons.

Edge cases the Principal called out:
- Amplicon arises in last stage → 0 subsequent stages → can't measure persistence.
- Amplicon arises with 1 stage after → if present, gets 100%.
- Most useful for early amplicons.

### Search findings

- **2026-04-28 FEEDBACK** (`archived/20260428-PHASE15_FEEDBACK.md:268-269`): Principal's verbatim discussion. Quote: *"for each candidate we are looking to see: (1) does it persist after it pops up?"* This is Check 1 of the 4 state-path-evolution checks.
- **SPEC15.5(e)** (`PHASE15_SPEC.md:398-417`): Check 1 = persistence-after-pop-up.
- **Implementation**: `onionskin_core/hmm_multistage_unification.py:_persistence_after_popup` (line 340), output column `state_path_persistence_after_popup`. Cycle 15.4a R3 verified clean (audit-log line 6248-6256, 6253).
- **AUDIT_LOG cycle 15.4a F5/Stage E**: per-amplicon Check 1 was implemented using `last_active_stage - onset_stage + 1` from `timing.py:541-543` (the `odw_duration_stages` column added in cycle 15.3a).

### Tentative direction (orchestrator note)

The Principal's "best guess" maps EXACTLY onto cycle 15.4a's existing `state_path_persistence_after_popup` per-amplicon check. The d7 work for Q1 is most likely a per-sample analog: **does sample s individually show the amplicon persisting in its own decoded state path across stages after first detection?** Aggregate across samples per amplicon (mean OR reliability-weighted mean) → new column `state_path_persistence_per_sample_aggregate` or similar.

Both Principal-raised scoring schemes (percent, count) have known existing handling in `odw_duration_stages` framing — the existing per-amplicon implementation uses ODW-window-based duration which sidesteps the early/late bias. The per-sample variant could mirror that.

### Status

**Awaiting Principal lock** after seeing critical finding above. Re-anchored framing for Q1: per-sample analog of existing per-amplicon `state_path_persistence_after_popup`, NOT a from-scratch definition.

---

## Q2 — `ghost_stage_count_weighting`

### R1's framing (audit-log lines 318-328)

What is a "ghost stage"? Candidate framings:
- (a) Sample-level ghost: sample's state path ≤ thresh_state everywhere in amplicon AND sample NOT in `flat_samples.tsv`. Per-amplicon HMM false negatives.
- (b) Stage-level ghost: stage where per-stage majority state-path call is ≤ thresh_state but stage IS within amplicon's [onset_stage, last_active_stage] ODW window.
- (c) Sample-stage-pair ghost: cross-pipeline detector says active but HMM says background.
- (d) Principal-supplied alternative.

### Principal's response (2026-05-03 chat)

> "Was I working with you when we discussed Ghost stages? It in fact was not my term. The Claude Opus I was discussing this stuff with coined it. And I assumed would write more extensively about it."

> Gut: "A ghost stage would be a stage where the amplicon 'ghosted' --> disappeared, after its initial appearance. So ghost stage count would be the complement of persistence count for a given amplicon, if my understanding is correct. And if that is true, it could be redundant. Unless the two concepts were used in a special way... maybe 1 is percent, 1 is count."

Principal's mapping:
- (b) stage-level ghost matches the Principal's understanding.
- (c) sample-stage-pair sounds like cross-pipeline synthesis (future-phase, not d7 scope).
- (a) — Principal didn't follow; asked for re-explanation if I think it's useful.

### Search findings

- **`hmm_fork_travel.py:188, 234-243`**: `ghost_level_flag` per-stage primitive — flag set when a stage has a ghost level (per-row computation in fork-travel data).
- **AUDIT_LOG cycle 15.4a R1 F3** (`PHASE15_AUDIT_LOG.md:6248`): "HMM-specific state-path growth credibility / ghost levels" — verified in `hmm_multistage_unification.py:_state_path_checks` aggregating various credibility components into `state_path_growth_credibility_score`. NOTE: ghost-stage count was supposed to be a per-trajectory aggregate of `ghost_level_flag` per Stage C (audit-log line 6463): *"Lift `ghost_level_flag` (per-stage flag) into a per-trajectory credibility score combining: (i) ghost-stage count, (ii) monotonic-growth of `n_levels`, (iii) monotonic-growth of fork-travel widths."*
- **AUDIT_LOG cycle 15.4a Stage C** (`PHASE15_AUDIT_LOG.md:6535`): *"Lifted step-12 ghost-level and fork-width primitives into per-trajectory state-path credibility outputs with `ghost_stage_count`, `state_path_growth_credibility_score`, and related pass/fail evidence columns."* So a `ghost_stage_count` column was emitted at cycle 15.4a.
- **2026-04-28 FEEDBACK**: ghost-level concept does NOT appear in this archive. The term "ghost" surfaces in the SPEC15.5(c) sub-priority text via a different agent's transcription, then gets implemented as `ghost_level_flag` + `ghost_stage_count` in cycle 15.4a.

### Tentative direction (orchestrator note)

The Principal's intuition that ghost ≈ "stage where the amplicon disappeared after initial appearance" is the inverse of persistence. The cycle-15.4a implementation already emits `ghost_stage_count` per amplicon. The d7 work for Q2 is most likely:
- Either a per-sample variant (count of stages where sample s lost its above-background state at the amplicon, weighted by reliability).
- Or pure consumption of the existing per-amplicon `ghost_stage_count` as a column in the d7 reliability extensions.

Re: (a) per-sample-level ghost framing — R1's (a) is "sample never showed the amplicon at all but should have per `flat_samples.tsv`." That's a different concept from "amplicon disappeared after appearing in this sample" (the Principal's framing). (a) catches false-negative samples; the Principal's framing catches mid-trajectory fade-out within a sample. Both could be useful; they measure different things.

### Status

**Awaiting Principal lock** after critical finding above. Likely d7 scope: per-sample variant OR pure consumption. The Principal's gut "ghost = complement of persistence" matches the cycle 15.4a `ghost_stage_count` column already emitted.

---

## Q3 — `multi_step_state_path_evolution_scoring`

### R1's framing (audit-log lines 330-340)

Candidate framings:
- (a) Monotonic-state-increase fraction across adjacent stage pairs.
- (b) State-path ↔ fork-travel correlation (Q4 territory).
- (c) Distinct state values traversed.
- (d) Principal-supplied alternative.

### Principal's response (2026-05-03 chat)

> "I really can't believe the agent did not write more extensively about this. We talked about it extensively. Is it in any of our planning files?!"

> "Fortunately it is not too hard to think about. From stage to stage, the state path defining an amplicon is expected to evolve in at least a few ways:
> 1. For amplicons that amplified higher than 2-fold, the first stage might see just a 2-fold state, but at least one or more subsequent stages will show nested layers pop out from the new doubling round.
> 2. These new layers are expected to arise approximately centered within the layer (state) beneath it.
> 3. So that is saying --> amplicons are expected to grow taller emanating from the summit (origin) area, and that they form pyramid structures (a triangle with steps).
> 4. The amplicon is expected to get wider in subsequent stages. And all layers are expected to get wider with each stage. This is like our fork trajectory tracking / fork age plots."

Edge cases:
- Amplicons starting early have multiple stages to evaluate.
- 2-fold-only amplicons: if ≥1 stage after, expect to grow wider even if not taller.
- Last-stage-only amplicons: can really only get a triangle score; absence in prior stages argues against spurious state-path-transition origin.

Principal summary: "state path evolution is quite related to the types of things we measure for reliability. Does it get taller? Does it get wider? Does it get more triangular / pyramid-shaped?"

**Q3 supplement (2026-05-04 — what an amplicon should NOT have):**

> "It should not, for example, just be a high copy number state jump that stays the exact same across all stages (like a collapsed repeat). It should probably be longer than 50 kb in its latest stage -- really that is probably too permissive. Even 100 kb is probably permissive based on what we are seeing, but other genomes may be different. Even Drosophila has just 50-100 kb amplicons, not these super wide ones we have. Suffice to say -- short 10-50 kb things are probably not amplicons... we would trust them much less."

Two distinct discriminators surfaced here:
- **Stability across stages** = collapsed-repeat signature (high-copy state jump that does not evolve). Already a major Phase 15 design pillar (see SPEC15.15 d3 RMS pre-filtered amplicon trigger; Growth Pass-2 retention of "bins > HI" for collapsed-repeat handling; SPEC15.5 multistage unification).
- **Latest-stage width threshold** = a per-amplicon credibility prior. Default trust for short (< 50-100 kb) amplicons is low; longer amplicons get higher prior weight. This is a NEW d7 angle not yet implemented in the cycle-15.4a per-amplicon checks, though SPEC15.5(e) edge case already mentions "in single-stage runs, fall back to length/step-count heuristics."

### Search findings — THIS IS WHERE THE SEARCH PAID OFF

**It IS in our planning files. Extensively. Implemented since cycle 15.4a (v0.14.80).**

- **2026-04-28 FEEDBACK** (`archived/20260428-PHASE15_FEEDBACK.md:269`): Principal's verbatim version of the same explanation, quoted in full:

  > *"So with the statepath evolution, for each candidate we are looking to see: (1) does it persist after it pops up? (2) if it is present in multiple stages, does it get wider across them? (3) is it is present in multiple stages, does it evolve to have more than one step? (4) if it has more than one step, is it pyramid-shaped (with the summit interval approximately centered between steps on both sides)? For amplicons that start early and grow tall, all checks will pass... For amplicons that start early (or any stage except the last stage), but do not grow in height, they would need to at least pass checks 1 and 2... For amplicons that appear in only one stage, it would have to be the last stage to satisfy check 1, and it would have to pass the multistage triangle analysis."*

  This is essentially the same explanation the Principal gave today, ~ a week earlier, in a different chat with a different agent. It got promoted into SPEC15.5(e) and then implemented in cycle 15.4a.

- **SPEC15.5(e)** (`PHASE15_SPEC.md:398-417`): the 4 checks codified verbatim as Checks 1-4 + 5 decision rules per amplicon profile.
- **`onionskin_core/hmm_multistage_unification.py:_state_path_checks`** (lines 357-396): aggregates Checks 1-4 + level_growth + fork_growth into `state_path_growth_credibility_score`.
  - Check 1 — `_persistence_after_popup:340`.
  - Check 2 — `_width_grows` (per cycle 15.4a R3 verification).
  - Check 3 — `_trajectory_stage_nlevels:204` + `more_than_one_step` aggregation.
  - Check 4 — `_pyramid_shaped:320`.
- **AUDIT_LOG cycle 15.4a R3** (`PHASE15_AUDIT_LOG.md:6248-6256`, `:6535-6537`): verifies each check is implemented + uses Bayesian ODW window detector primitive (NOT 1.25× retired hard rule).

**Search findings for Q3 supplement (collapsed-repeat / width-threshold; surfaced 2026-05-04):**
- **2026-04-28 FEEDBACK** (`archived/20260428-PHASE15_FEEDBACK.md:269`): same fundamental concept — *"a region that is single-step and the same length (whether it is 10 kb or 400 kb) across all stages would more likely be a collapsed repeat."* The 2026-04-28 framing emphasized **stability across stages** as the discriminator (NOT width directly — even 400 kb gets called collapsed-repeat if stable). The 2026-05-04 supplement adds **width threshold as a separate trust prior** (< 50-100 kb = lower trust regardless of stability pattern). Subtly different angle.
- **SPEC15.15 d3 + Growth asymmetry** (`PHASE15_SPEC.md:1191-1193`): Pass-2 background mask machinery uses RMS pre-filtered amplicon calls (which include collapsed repeats) as the masking trigger; Growth Pass-2 RETAINS "bins > HI" because growth pipeline does NOT call collapsed repeats. So the collapsed-repeat-vs-amplicon discrimination is structurally embedded across the pipelines.
- **No existing per-amplicon `min_width_bp` trust threshold flag** in the codebase. The "fall back to length/step-count heuristics" SPEC15.5(e) edge case is for single-stage runs only.

The Q3 supplement's width-threshold discriminator is partially novel — it's a credibility prior the d7 reliability extensions could carry as a column (e.g., `state_path_latest_stage_width_bp` + a soft-trust gate at 50-100 kb), separate from the existing 4-check framework.

### Tentative direction (orchestrator note)

**The d7 Q3 work is most likely**: per-sample variants of the existing per-amplicon checks. For a given amplicon `a` and sample `s`:
- Per-sample Check 1: does sample `s`'s decoded state path show persistence-after-pop-up at `a`?
- Per-sample Check 2: does sample `s`'s decoded state path show width-grows-across-stages at `a`?
- Per-sample Check 3: does sample `s`'s decoded state path show multi-step evolution at `a`?
- Per-sample Check 4: does sample `s`'s decoded state path show pyramid-shape at `a`?

Aggregate across samples per amplicon (mean OR reliability-weighted mean) → new HMM-only reliability column(s).

OR: pure consumption — surface the existing `state_path_growth_credibility_score` (per-amplicon, cross-sample-aggregated already) as a column in the SPEC15.6 reliability sidecar for HMM rows.

The (a)/(b)/(c) candidates from R1 are partly redundant with this re-anchoring. (b) state-path↔fork-travel correlation IS Q4 territory and remains useful. (a) monotonic-state-increase and (c) distinct-states-traversed are simpler than the existing 4-check framework — possibly subsumed by the existing implementation.

### Status

**Awaiting Principal lock** after critical finding above. The d7 work for Q3 most likely = per-sample analogs of the cycle-15.4a 4-check framework, OR pure consumption of `state_path_growth_credibility_score`. NOT a redefinition of the concepts.

---

## Q4 — `fork_travel_monotonic_growth_integration`

### R1's framing (audit-log lines 342-352)

How does fork-travel fold into the credibility score? Candidate framings:
- (a) Direct integration via correlation (= Q3(b)).
- (b) Threshold gate.
- (c) Continuous slope component.
- (d) Principal-supplied alternative.

### Principal's response (2026-05-04)

> "Oh I am not deferring Q4 per se. It just seemed like I answered it already. It seems like part of the state path evolution answer -- specifically fork travel, trajectory, age stuff; general widening growth. It does not seem mysterious."

Principal considers Q4 effectively answered by Q3 — fork-travel is part of the state-path-evolution answer (widening / fork trajectory / fork age territory). Treat it like the other 3 checks, not as a special threshold gate or continuous slope component. The cycle-15.4a `fork_growth` boolean monotonicity check already implements this per-amplicon. No separate per-amplicon question remains; the per-sample-analog-vs-consumption framing question still applies (same as Q1-Q3).

### Search findings

- **`fork_growth` column** already exists in the cycle-15.4a `_state_path_checks` aggregate at `hmm_multistage_unification.py:357-396`.
- **`hmm_fork_travel.py:188, 234-243`** is the per-stage `ghost_level_flag` + fork-travel data source.
- **AUDIT_LOG cycle 15.4a Stage C** (`PHASE15_AUDIT_LOG.md:6463`): *"monotonic-growth of fork-travel widths across stages"* explicitly listed as the third component of the per-trajectory ghost-level credibility score combining ghost-stage count + n_levels monotonic growth + fork-travel width monotonic growth.
- **AUDIT_LOG cycle 15.4a Stage E** (`PHASE15_AUDIT_LOG.md:6537`): *"fork-travel monotonicity checks"* implemented.

So fork-travel monotonic-growth integration ALREADY exists per-amplicon; the d7 question reduces to: per-sample analog (does sample s individually show fork-travel monotonic growth?) OR pure consumption of existing per-amplicon `fork_growth`.

### Status

**Awaiting Principal — deferred pending critical-finding digestion.**

---

## Q5 — Aggregation across the four metrics

### R1's framing (audit-log lines 354-364)

Candidate framings:
- (a) Additive columns only. **R1 RECOMMENDED.**
- (b) HMM-only aggregate score.
- (c) Replace cross-pipeline `reliability_score` for HMM rows. **R1 RECOMMENDED NOT.**
- (d) Principal-supplied alternative.

### Principal's response

*[Awaiting.]*

### Search findings

*[Will populate after Principal response.]*

Pre-search note: cycle 15.4a already emits `state_path_growth_credibility_score` as an aggregate of the 4 checks + level_growth + fork_growth. So an aggregate already exists; the Q5 question is really "additive new columns ONTO the existing aggregate" vs "extend the existing aggregate."

### Status

**Awaiting Principal response.**

---

## Q6 — Schema column naming convention

### R1's framing (audit-log line 366)

`state_path_*` prefix vs `hmm_credibility_*` prefix. R1 recommends `state_path_*`.

### Principal's response

*[Awaiting.]*

### Search findings

*[Will populate after Principal response.]*

Pre-search note: existing columns from cycle 15.4a use `state_path_*` prefix (`state_path_persistence_after_popup`, `state_path_width_grows`, etc.) — establishing precedent.

### Status

**Awaiting Principal response.**

---

## Q7 — Classifier integration

### R1's framing (audit-log line 368)

Should the six-label `_classify_amplicon` get HMM-only label variants? R1 recommends NO.

### Principal's response

*[Awaiting.]*

### Search findings

*[Will populate after Principal response.]*

### Status

**Awaiting Principal response.**

---

## Q8 — Validation surface

### R1's framing (audit-log line 370)

New HMM-only columns are PURELY ADDITIVE; existing `reliability_score` for HMM rows must be byte-identical to v0.14.90 value on DS1 chr II calibration set. R1 confirms read.

### Principal's response

*[Awaiting.]*

### Search findings

*[Will populate after Principal response.]*

### Status

**Awaiting Principal response.**

---

## Provisional locks

### LOCKED 2026-05-04 — d7 scope for cycle 15.7a is the hybrid path

**(ii) Pure consumption + width-threshold supplement** lands in cycle 15.7a Stage F. Per-sample analogs (option (i)) are DEFERRED to end-of-phase consideration (see § Deferred work below).

**Cycle-15.7a Stage F deliverable contract — FULLY LOCKED 2026-05-04 (late evening):**

R2 implements the following 11-column contract. All columns land on `amplicon_reliability.tsv` (the SPEC15.6 cross-pipeline reliability sidecar) for HMM rows; NULL/empty for Growth + RMS rows. Verbose names preserved (Q6 lock — Principal direction "verbose names you had are fine").

**Existing 8 columns surfaced from `<hmm-out>/14-multistage-unification/all_trajectories.state_path_credibility.tsv` (computed by `_state_path_checks` at `hmm_multistage_unification.py:407-415`):**

1. `state_path_persistence_after_popup` (bool)
2. `state_path_width_grows` (bool)
3. `state_path_more_than_one_step` (bool)
4. `state_path_pyramid_shaped` (bool)
5. `state_path_level_growth_monotonic` (bool)
6. `state_path_fork_travel_monotonic` (bool)
7. `ghost_level_count` (int)
8. `state_path_growth_credibility_score` (float [0, 1]) — `(positives_count / 6) - min(0.35, 0.05 × ghost_count)`, clamped.

**3 new columns computed in `_state_path_checks` (or downstream surfacing layer):**

9. `state_path_latest_stage_width_bp` (int) — raw width of amplicon at its latest active stage. Always emitted; lets downstream consumers apply their own thresholds. Computed from the amplicon's bin range at the stage matching `last_active_stage`.

10. `state_path_width_credibility` (float [0, 1]) — soft-ramp trust factor based on width:
    ```
    state_path_width_credibility = clamp((latest_stage_width_bp - low) / (high - low), 0, 1)
    ```
    Defaults: `low = 10_000`, `high = 100_000`. CLI flags: `--state-path-width-trust-low-bp` (default 10000) + `--state-path-width-trust-high-bp` (default 100000) for organism-dependent tuning.

11. `state_path_growth_credibility_score_width_adjusted` (float [0, 1]) — multiplicative width-adjusted aggregate:
    ```
    state_path_growth_credibility_score_width_adjusted =
        state_path_growth_credibility_score × state_path_width_credibility
    ```
    Principal direction 2026-05-04 (late evening): "testing the water for including width_credibility as a multiplier" — exploratory column; no downstream consumer wired to it this cycle. The base `state_path_growth_credibility_score` (col 8) stays intact for downstream consumers that already use it (multistage unification mode 2/3/4 logic). The width-adjusted form (col 11) is purely additive surfacing for empirical evaluation.

**IGV-track emission:**
- `<aps-out>/tracks/state_path_growth_credibility.{bedGraph, bed}` for col 8 (HMM-only this cycle; cross-pipeline rollout future-phase since the score itself is HMM-native).
- Optional: track for col 11 if straightforward.

**Validation gates:**
- `reliability_score` for HMM rows stays byte-identical to v0.14.90 value on the DS1 chr II calibration set (Q5(a) + Q8 lock). The 11 new columns are PURELY ADDITIVE — no change to existing scores or aggregates.
- New `tests/test_hmm_state_path_aware_reliability.py` (R1-prescribed in cycle 15.7a R1 audit F2 / F3): cover the surfacing join + width-credibility soft-ramp shape + multiplicative aggregate formula + no-regression on existing `reliability_score`.
- `make puff-compare` 28/28 PASS unchanged.

**Stage F gating (R2 reads this and confirms):** Stage F runs after Stage A (SAPS lands, since Stage F columns join the HMM step-14 multistage unification output which already exists, but downstream HMM consumers may benefit from SAPS data being available). Per cycle 15.7a R1 audit, Stage F holds until Principal hammer-out resolves. **Hammer-out is now resolved** (locks above); R2 may fire Stage F when ready.

### Q3 supplement sub-questions — LOCKED 2026-05-04

**Q3.S1 — width-credibility soft form. LOCKED: (b) Soft ramp.**

Form: `state_path_width_credibility = clamp((width - low_floor) / (high_floor - low_floor), 0, 1)` with default `low=10_000`, `high=100_000`. Continuous; below 10kb = 0; above 100kb = 1; linear ramp between. CLI flags `--state-path-width-trust-low-bp` (default 10000) + `--state-path-width-trust-high-bp` (default 100000) for organism-dependent tuning.

**Q3.S2 — integration into aggregate `state_path_growth_credibility_score`. LOCKED: (a) Independent column for cycle 15.7a.**

`state_path_width_credibility` lands as a standalone column; the existing `state_path_growth_credibility_score` aggregate stays byte-identical at v0.14.90 calibration. Multiplicative integration (form (b)) is acceptable per Principal direction but reserved for a post-empirical-data revisit; not in cycle 15.7a scope.

**Principal direction 2026-05-04 (governing rationale):**

> "Yes the width threshold is something to note but probably shouldnt exclude things that pass other tests. It is just one test to consider - we should implement, but use it softly. The exceptions might be hard-to-detect amplicons that fire in a small proportion of cells such that their signal doesn't rise over the background well, and they appear short."

This is THE governing reason for picking soft-ramp + independent-column over hard-floor + multiplicative: real amplicons firing in only a small proportion of cells may have width-suppressed signal but pass the other state-path checks. A hard floor would exclude them; a multiplicative penalty would suppress their aggregate score. Soft-ramp + independent column lets downstream consumers see both signals without one occluding the other. Cross-sample heterogeneity at low-cell-fraction loci is also exactly the regime where the deferred Option (i) per-sample analogs would shine — orthogonal future-work synergy.

---

## Deferred work — Option (i) per-sample analogs (end-of-phase or follow-up cycle)

**Status:** Deferred per Principal direction 2026-05-04. NOT in cycle 15.7a Stage F scope. To be revisited at end-of-phase or via mid-phase amendment after empirical SAPS data is available.

**Principal motivation for keeping the idea alive (2026-05-04):**

> "I think we should take everything you said about 'i' and then consider doing it at the end of the phase somewhere. I still need to wrap my head around it, but it might be useful, especially for when we apply this to the full genome dataset 1 where there are 8-9 stages with up to several samples per stage."

The full-genome DS1 dataset (~8-9 stages × multiple samples per stage) is exactly the regime where per-sample heterogeneity becomes meaningful — small-sample-count datasets like the toy fixture or DS1-chr-II calibration set may not show enough cross-sample variation for per-sample analogs to matter, but the full-genome multi-stage dataset has the dimensionality where joint-HMM-consensus may hide real sample-level disagreement.

**Concept summary (preserved for re-surfacing):**

- After SAPS (SPEC15.13 d1-d6) lands at cycle 15.7a Stage A, each sample s has its own decoded state path per stage at every chromosome.
- Apply the SAME 4 cycle-15.4a checks (persistence-after-popup, width-grows, multi-step-evolution, pyramid-shape) to each (sample s, amplicon a) pair using s's individual decoded path instead of the joint consensus.
- Aggregate across samples per amplicon: `state_path_<check>_per_sample_aggregate = fraction_of_samples_passing_check`.
- New per-amplicon columns surfacing cross-sample agreement vs disagreement:
  - `state_path_persistence_per_sample_agreement_fraction` (float [0, 1])
  - `state_path_width_grows_per_sample_agreement_fraction`
  - `state_path_multi_step_per_sample_agreement_fraction`
  - `state_path_pyramid_per_sample_agreement_fraction`
  - Aggregate: `state_path_per_sample_coherence_score` = mean of above fractions.

**Why valuable (worth re-surfacing):**

- **Detects cross-sample heterogeneity hidden by joint consensus.** Joint says "persistent" because 70% agree → per-sample column surfaces the 30% disagreement.
- **Distinguishes confident from ambiguous calls.** Per-sample agreement near 1.0 = clear amplicon; near 0.5 = ambiguous → useful filter.
- **Cross-sample coherence as its own credibility signal.** Real amplicons should show coherent evidence across samples; spurious calls cluster in outlier samples.

**Why deferred (vs cycle 15.7a Stage F):**

- Substantive R2 work depending on SAPS landing first (Stage A → wait → Stage F-or-later).
- Speculative value — joint checks may already capture most signal on small datasets.
- Empirical SAPS data needed to evaluate before committing implementation effort.
- Cycle 15.7a is already a 4-SPEC-priority cycle; adding (i) substantively expands scope.

**Likely landing zones:**

- **Cycle 15.10a SPEC15.20** — could fold into the closeout-housekeeping cycle as a new sub-deliverable IF empirical SAPS data shows per-sample analogs add value.
- **New mid-phase amendment cycle (15.7a-S1 or similar)** — if the value is clear post-SAPS landing, a focused supplemental cycle just for per-sample analogs.
- **Post-Phase-15 follow-up phase** — if the value emerges only on full-genome DS1 + the principled redesign discussion in `[ISSUE:2026-05-03:1]` ends up requiring a broader reliability/importance scoring overhaul.

**Action item at cycle 15.7a closeout:** R3 should surface this deferred work in the closeout report so it stays on the orchestrator's radar.

**Cross-references:**
- `[ISSUE:2026-05-03:1]` — reliability + importance scoring redesign + relocation; post-Phase-15 scope.
- SPEC15.13 d7 (`PHASE15_SPEC.md:1018-1024`) — the placeholder language whose per-sample framing originally implied this scope.
- `multi-agent/tracking/BRAINSTORM.md:1900-1906` — 2026-04-19 entry that explicitly anticipated "HMM-specific metrics designed separately" as a "second pass."

---

## Discussion log

*Free-form notes during the Principal-orchestrator deliberation. Verbatim Principal quotes preserved where relevant. Orchestrator pushback / re-explanations recorded.*

### 2026-05-03

- Q1-Q3 surfaced by R1 audit (commit `ca896f6`).
- Principal began responding in sequence; Q1, Q2, Q3 answered.
- Orchestrator search after Q3 surfaced the critical finding above (Q1-Q3 concepts already implemented in cycle 15.4a). Principal alerted.
- File created at this point per Principal direction to avoid the prior-conversation-not-recorded failure mode.

### 2026-05-04 (late evening) — full hammer-out resolution

- **All sub-questions LOCKED**:
  - Scope: (ii) consumption + width-threshold supplement (cycle 15.7a Stage F); (i) per-sample analogs deferred to end-of-phase consideration with full-genome DS1 motivation preserved.
  - Q3.S1 (width-credibility form): soft ramp, `clamp((width - 10000) / 90000, 0, 1)`, CLI-tunable.
  - Q3.S2 (width integration into existing aggregate): multiplicative, but as a NEW column (`state_path_growth_credibility_score_width_adjusted`), not modifying the existing `state_path_growth_credibility_score`. Principal framing: "testing the water for including width_credibility as a multiplier."
  - Q5: additive columns + new HMM-only width-adjusted aggregate. Cross-pipeline `reliability_score` byte-identical at v0.14.90.
  - Q6: verbose `state_path_*` names retained for surfacing (no rename to short forms).
  - Q7: do NOT extend `_classify_amplicon` six-label classifier with HMM-only labels.
  - Q8: purely additive validation gate.
- **11-column contract** locked in § Provisional locks above.
- Path forward: orchestrator-outside-cycle SPEC engineering replaces SPEC15.13 d7 placeholder with the locked contract; commit; launch R2 with full Stage A-F contract.

### 2026-05-04 — Q3 supplement (afternoon)

- Q3 supplement from Principal: collapsed-repeat discriminator + latest-stage width threshold (50-100 kb floor for credibility prior).
- **Orchestrator analysis of remaining open questions** (post-search; post-Q1-Q4 responses):
  - **The single open framing question** — what is d7's actual scope? (i) per-sample analogs of cycle-15.4a per-amplicon checks consuming SAPS data; (ii) pure consumption of existing per-amplicon `state_path_growth_credibility_score` as new sidecar columns; (iii) something else (post-Phase-15 redesign / hybrid). Q1-Q4 mechanics fall out of this single decision.
  - **The new width-threshold supplement** — orthogonal to (i)/(ii); independent decision. Add a `state_path_latest_stage_width_bp` column + soft-trust gate? Threshold value? Continuous credibility weight vs hard pass-fail?
  - **Q5 (aggregation)** — falls out of the framing decision: under (i) → per-sample variants get aggregated like `state_path_growth_credibility_score` did at cycle 15.4a; under (ii) → existing aggregate gets surfaced. R1's recommended (a) "additive columns only" is consistent with both; cross-pipeline `reliability_score` stays stable.
  - **Q6 (column naming)** — rubber-stamp; existing columns use `state_path_*` prefix.
  - **Q7 (classifier integration)** — likely rubber-stamp; cross-pipeline parity blindspot lens argues against extending six-label `_classify_amplicon` with HMM-only labels.
  - **Q8 (validation surface)** — falls out of Q5; "purely additive new columns; existing `reliability_score` byte-identical at v0.14.90 calibration" works under (i) and (ii).
- Going one-at-a-time: present the framing question first, then the width-threshold supplement, then bundle Q6/Q7/Q8 as confirmations.
- Orchestrator memory-comparison check (Principal asked "How was my memory? Was I defining things similar to how I defined them in the past?"):
  - **Q1 persistence**: 2026-04-28 → "(1) does it persist after it pops up?" / 2026-05-03 → "amplicons expected to persist through all subsequent stages... percent of subsequent stages... 100% expected." **VERY similar**; same core concept; today added explicit scoring-scheme thinking (percent vs count).
  - **Q3 multi-step evolution**: 2026-04-28 → 4 checks (persistence; widens; multi-step; pyramid). 2026-05-03 → same 4 elements expressed more vividly (nested layers from doubling rounds; centered new layers; pyramid structures; widens with each stage; fork trajectory analog). **STRONG match**; same framework.
  - **Last-stage-only edge case**: 2026-04-28 → *"For amplicons that appear in only one stage, it would have to be the last stage to satisfy check 1, and it would have to pass the multistage triangle analysis."* / 2026-05-03 → *"For amplicons that arise in the last stage present in the data, there is no way to measure the evolution. I believe the recommendation back then that I gave was that we can really only get a triangle score from such an amplicon."* **EXACT match — Principal even self-cited the prior recommendation.**
  - **Collapsed-repeat discriminator**: 2026-04-28 emphasized **stability across stages** (even 400 kb stable = collapsed repeat). 2026-05-04 added **width threshold as a separate trust prior** (< 50-100 kb = lower trust regardless of stability). **Slightly new angle today**; mostly compatible with prior framing.
  - **Q2 ghost stage**: not in 2026-04-28 FEEDBACK explicitly — concept came from a different prior agent's transcription that became SPEC15.5(c) text. The Principal's gut today ("ghost = complement of persistence") matches the cycle-15.4a `ghost_stage_count` column already emitted.
  - **Overall**: Memory was very good. Reconstructed the 4-check framework from first principles, exactly as it was defined ~ a week ago. The recording gap was on the prior agent's side (paraphrased into d7 placeholder language without preserving the per-amplicon-vs-per-sample distinction or the 4-check anchoring), not on the Principal's side.
