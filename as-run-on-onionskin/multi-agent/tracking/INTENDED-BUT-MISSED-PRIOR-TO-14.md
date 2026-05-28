# INTENDED-BUT-MISSED — inventory of work that was intended for phases prior to Phase 14 but was silently narrowed, deferred, or dropped without user approval (or was user-approved-deferred-post-HMM but never revisited)

**Authors:** John M. Urban (audit manager), Claude Code 2.1.104 (claude-opus-4-7)
**Created:** 2026-04-22 ~22:00 EDT
**Scope:** Phases 4 through 13 (and pre-phase refactor plans). Phase 14 and its Supplemental
are tracked in `multi-agent/plans/archived/20260426-PHASE14_SUPPLEMENTAL-FEEDBACK.md` /
`multi-agent/plans/archived/20260426-PHASE14_SUPPLEMENTAL-SPEC.md`
and are out of scope here. Phases 15+ are also out of scope. Phases 15+ live files are in `plans/next/` at the time of writing this, but may also now be in `plans/archived/` if since completed.

**Purpose:** Same spirit as the Phase 14 recon audit. Inventory every item that was intended
for one of the pre-14 phases (by its letter or spirit) but was silently narrowed out, moved
to `KNOWN_ISSUES.md` / `BRAINSTORM.md` without user approval, or user-approved-deferred
"post-HMM" and never revisited. Err on the side of listing — this is a triage surface for
the user, not a pre-decided plan.

**Sources dug through:**

- `ROADMAP.md` — phase 4–11 markers, especially `◑ PARTIAL`, `✗ DEFERRED`, `✗ DROPPED`,
  `✗ CLOSED`, `✓ SUBSTANTIALLY COMPLETE`
- `multi-agent/plans/archived/20260407-CODE-UNIFICATION-REFACTOR.md` (pre-phase)
- `multi-agent/plans/archived/20260409-PUFFSTEP-INTEGRATION-PLAN.md` (pre-phase)
- `multi-agent/plans/archived/20260412-ROADSTRAVELED.md` (retired; historical only)
- `multi-agent/plans/archived/20260411-PHASE9_SPEC.md` and `*PHASE9_COPILOT_NOTES.md`
- `multi-agent/plans/archived/20260413-PHASE10_SPEC.md`
- `multi-agent/plans/archived/20260415-PHASE11_SPEC.md`
- `multi-agent/plans/archived/20260416-PHASE12_BRAINSTORM.md` and `*PHASE12_SPEC.md`
- `multi-agent/plans/archived/20260417-PHASE13_BRAINSTORM.md` and `*PHASE13_SPEC.md`
- `multi-agent/plans/archived/20260419-PHASE13_SIDE_QUEST.md` and `*SIDE_QUEST.FEEDBACK.md`
- `multi-agent/tracking/KNOWN_ISSUES.md`
- `multi-agent/tracking/BRAINSTORM.md`
- `CHANGELOG.md` phase closure notes (sampled)

**Terminology mapping used in this document (most-recent form only):**

| Most recent term | Earlier names seen in archives |
|---|---|
| RMS / rcn-mean-shift | per-stage, per-stage-mean-shift, single-engine, single-file, single-mode, single-stage, single_engine |
| growth / growth-model | multistage-growth, multistage (deprecated-as-pipeline-name in Phase 11; multistage is now only a *property* of a multi-stage dataset) |
| HMM | PuffStep-ported, PuffStep (referring to predecessor tool) |
| step dir (e.g. `09-summit-refinement`) | sometimes called `04-summit_refinement/` pre-Phase 11; grouped layout landed in Phase 7.4 and was consolidated in Phase 11 |

---

## Phase 15 closeout sweep — IBM-C* dispositions (cycle 15.10a Stage D F11; SPEC15.20 d3)

Per cycle 15.10a Stage D R2 gap-closure iteration 3 (2026-05-05; SPEC15.20 d3 directive
to mark RESOLVED status on IBM-C* items addressed by Phase 15). Detailed entry bodies
below remain unchanged for historical context; this banner records the closeout-state
disposition per IBM in one place.

| IBM | Phase 15 disposition | Cycle / version |
|-----|---------------------|-----------------|
| IBM-C2 (HMM parallel child pipeline + SAPS + step-14 APS fix) | **RESOLVED** — HMM parallel child pipeline shipped via SPEC15.3 (cycle 15.3a, v0.14.79); step-14 APS bug fix bundled. SAPS shipped via SPEC15.13 (cycle 15.7a, v0.14.91). | 15.3a/15.7a |
| IBM-C3 (Amplicon reliability scoring + flat-sample detection) | **RESOLVED** — Cross-pipeline reliability scoring + flat-sample detection + classifier shipped via SPEC15.6 (cycle 15.4a + supplemental cycles 15.4a-S1/S2/S3/S4, v0.14.80–v0.14.89). New `onionskin_core/flat_sample.py` + `--flat-sample-*` flags + `--known-reliable-amplicons` synonym + `--allow-nonflat-ref-stage` + auto-switch logic. | 15.4a + S1–S4 |
| IBM-C4 (HMM summit accuracy: `peak_rcn_stage`, sliding-offset sub-bin) | **RESOLVED** — HMM summit refinement shipped via SPEC15.7 (cycle 15.5a, v0.14.83); HMM per-stage parabola summit emission via SPEC15.24 (cycle 15.7b, v0.14.92); cross-pipeline summit-aggregation menu via SPEC15.19 d3 (cycle 15.10a, v0.14.95) — initial 9-strategy implementation REVERTED + redesigned via cycle 15.10a-S2 v0.14.97 (curated 68-strategy single-flag menu replacing the reverted 9-strategy 2-flag scaffold). | 15.5a/15.7b/15.10a/15.10a-S2 |
| IBM-C5 (Dynamic origin-detection window cross-pipeline) | **RESOLVED** — Bayesian ODW System shipped via SPEC15.4 (cycle 15.4a, v0.14.80) as the cross-pipeline 4-metric dynamic origin-detection-window framework. New `onionskin_core/odw.py` detector primitive + `--odw-*` argparse group. Architectural cleanup + per-direction threshold split via SPEC15.21 (cycle 15.9a, v0.14.94). | 15.4a/15.9a |
| IBM-C6C (Stage-activity fold-change using refined summit; timing.py refactor) | **RESOLVED** — Cross-pipeline summit↔timing one-pass 4-step convergence shipped via SPEC15.8 (cycle 15.5a, v0.14.83). | 15.5a |
| IBM-C6D (Timing-guided active-stage weights for summit estimation) | **RESOLVED** — Active-stage strategy menu (`--summit-stage-selection`) shipped via SPEC15.7 (cycle 15.5a, v0.14.83) with `odw_*` strategies that consume timing-derived ODW boundaries to drive summit estimation. | 15.5a |
| IBM-C7A (APS dimensionality reduction / embedding) | **RESOLVED** — 3-layer APS PCA shipped via SPEC15.11 (cycle 15.6a, v0.14.84) covering amplicon Layer 1 + sample Layer 2 raw + sample Layer 2 zscore. Per-pipeline cross-pipeline parity via SPEC15.17 (cycle 15.8a, v0.14.93). Deferred bits (UMAP/diffusion) tracked at the APS_SOUP for future-phase work. | 15.6a/15.8a |
| IBM-C7B (Shape-aware APS clustering) | **RESOLVED** — Shape-aware APS feature surfaces shipped via SPEC15.11 (composite multi-feature APS clustering modes) cycle 15.6a, v0.14.84. Aggregation-mode/score machinery via SPEC15.12 cycle 15.6a-S1, v0.14.90. | 15.6a/15.6a-S1 |
| IBM-C8A (`--aps-area-excess-floor on\|off` flag) | **RESOLVED** — Already captured as Phase 14 Supplemental Finding 9 / Q22 / 14-S22 (closed v0.14.70). Phase 15 SPEC15.9 (cycle 15.6a, v0.14.84) finalized the analytical testing + default behavior. | Phase 14-S22 + 15.6a |
| IBM-C8B (Stage-1 pre-amp isolation / principled flat-sample detection) | **RESOLVED** — Subsumed by IBM-C3 resolution: SPEC15.6 cross-pipeline flat-sample detection (cycle 15.4a + S1/S2/S3/S4, v0.14.80–v0.14.89) is the principled flat-sample-detection mechanism for the project. | 15.4a + S1–S4 |
| IBM-C12 (APS locus diagnostics) | **RESOLVED** — APS locus diagnostics relocated + cleaned up architecturally via SPEC15.21 cycle 15.9a (v0.14.94) — the per-locus regression/dip/oscillation surface moved to new `onionskin_core/timing_diagnostics.py` module; `aps.py` consumes via import. Diagnostic columns (`regression_stage`, `max_rcn_stage`, `dip_rate`, `n_real_dips`, `n_transitions`, `transition_ps`, `oscillating_transitions`, `locus_weight`) preserved cross-cycle. | 15.9a |
| IBM-C14 (Keep/exclude recommendation enhancements) | **PARTIALLY-RESOLVED** — `--known-reliable-amplicons` synonym + reliability-scoring framework absorbed shape-score-proximity + timing-flag concepts via SPEC15.6 cycle 15.4a (v0.14.80). Further reconciliation tracked at `KNOWN_ISSUES.md [ISSUE:2026-04-29:5]` part-b deferred to future phase. (Closeout-state previously verified at cycle 15.10a Stage D F11.) | 15.4a (partial) |

**IBMs NOT addressed by Phase 15 (intentionally — out of scope):** IBM-C1 (terminology
internal-code rename — Phase 14 Supplemental scope), IBM-C6A/B/E/F (Phase 5 deferrals
deeper than what Phase 15 absorbed; remain candidates for future phases), IBM-C7
non-A/B parts (UMAP / diffusion embeddings), IBM-C9 (`--feature-track` ingestion;
user-gated), IBM-C10 (robustness dataset expansion; user-gated), IBM-C11 (cross-pipeline
synthesis / `04-unified-results/` — planned future phase), IBM-C13 (CHANGELOG/DEVLOG split
mechanics — covered by Phase 14 Supplemental dev-system work).

---

## TOP-LEVEL QUICK-PERUSE SECTION (for triage)

Priority ranks are the auditor's best judgment of "how likely the user would want this picked
up soon." They are not commitments. Each item has a one-line hook and a pointer to its
detailed entry below.

### High-priority clusters (strong candidates to pick up)

| Rank | ID | Title | Origin | Status hook |
|---|---|---|---|---|
| 1 | **IBM-C2** | HMM parallel child pipeline + SAPS + step-14 APS raw-file fix | Phases 9/12 (scaffold-era "12.2") | Fully designed in BRAINSTORM; blocks biological-correctness bug in HMM step-14; never implemented |
| 2 | **IBM-C3** | Amplicon reliability scoring + flat-sample detection before APS/clustering | "11.5" promotion record (never landed) → scaffold-era "12.3" | Labeled "PROMOTED to PHASE11_SPEC 11.5" in KNOWN_ISSUES, but Phase 11 SPEC only ever had 11.1–11.3; orphaned |
| 3 | **IBM-C1** | Internal-code "Stage-1/Stage-2" → "pass 1/pass 2" (detection pass terminology) | Phase 13 Deferred Loose End #1 | Public CLI/help partially done in Phase 14; internal labels (`stage1_best_peak`, `stage1_score_z`, etc.) and prose still stale |
| 4 | **IBM-C4** | HMM summit accuracy — `peak_rcn_stage` column; sliding-offset sub-bin refinement for HMM | Phase 9 follow-up → scaffold-era "12.1" carry-over | Both fully designed in BRAINSTORM; depend on IBM-C2; never implemented |
| 5 | **IBM-C5** | Dynamic origin-detection window (cross-pipeline summit refinement) | Phase 13 Priority 13.3 II/9A follow-up; [ISSUE:2026-04-18:3] | Design sketch pending; user explicitly flagged this as needing a design pass before more selector work |

### Medium-priority clusters (post-HMM items the ROADMAP said "revisit after Phase 7", which is long done)

| Rank | ID | Title | Origin | Status hook |
|---|---|---|---|---|
| 6 | **IBM-C6A** | APS order stability (bootstrap diagnostics) | ROADMAP Priority 5.3 `✗ DEFERRED (post-HMM)` | Never revisited; design notes in BRAINSTORM [2026-04-07] |
| 7 | **IBM-C6B** | Better posterior ordering logic (scalar/dendrogram/hybrid) | ROADMAP Priority 5.4 `✗ DEFERRED (post-HMM)` | Never revisited; design in BRAINSTORM [2026-04-07] |
| 8 | **IBM-C6C** | Stage-activity fold-change using refined summit (timing.py refactor) | ROADMAP Priority 5.0.2 `✗ DEFERRED (post-HMM)` | Design in BRAINSTORM [2026-04-07] |
| 9 | **IBM-C6D** | Timing-guided active-stage weights for summit estimation | ROADMAP Priority 5.9.3 `✗ DEFERRED (post-HMM)` | Design in BRAINSTORM [2026-04-07] |
| 10 | **IBM-C6E** | Iterative summit ↔ timing convergence (EM-like) | ROADMAP Priority 5.9.4 `✗ DEFERRED (post-HMM)` | Depends on IBM-C6D |
| 11 | **IBM-C6F** | Prior/posterior profile similarity investigation | ROADMAP Priority 5.9.6 `✗ DEFERRED (post-HMM)` | Needs a dataset with differing prior/posterior groupings |
| 12 | **IBM-C7A** | APS dimensionality reduction / embedding (PCA, UMAP, diffusion) | ROADMAP Priority 6.2 `✗ DEFERRED (post-HMM)` | Phase 6 closure note moves this to BRAINSTORM |
| 13 | **IBM-C7B** | Shape-aware APS clustering (cluster on amplicon morphology) | ROADMAP Priority 6.3 `✗ DEFERRED (post-HMM)` | Same closure note |
| 14 | **IBM-C8A** | RCN floor removal experiment + `--aps-area-excess-floor on\|off` flag | KNOWN_ISSUES [2026-04-18:1]; early Phase 4–5 origin | Already captured as Phase 14 Supplemental Finding 9 / Q22 — listed here for cross-reference |
| 15 | **IBM-C8B** | Stage-1 pre-amp isolation (principled "flat sample" detection in APS) | KNOWN_ISSUES [2026-04-14:1]; APS work during Phase 5 | Overlaps with IBM-C3; design already substantial |

### Lower-priority / open-design / user-gated clusters

| Rank | ID | Title | Origin | Status hook |
|---|---|---|---|---|
| 16 | **IBM-C9** | External signal-track ingestion for APS features (`--feature-track NAME:PATH`) | ROADMAP 5.6 was `✗ DROPPED`; successor design preserved in BRAINSTORM [2026-04-07] | User-gated — dropped with successor design kept; revive only on user ask |
| 17 | **IBM-C10** | Robustness dataset expansion | ROADMAP 5.12 `✗ DEFERRED (post-HMM)` | User-gated — "Do not add datasets without user confirmation" |
| 18 | **IBM-C11** | Cross-pipeline synthesis / `04-unified-results/` | Phase 7.4 architectural note; Phase 11.6 scope statement | Planned as a dedicated future phase, not missed-per-se; listed for completeness |
| 19 | **IBM-C12** | APS locus diagnostics (`best_onset_stage`, `post_support`, `dip_rate`) | BRAINSTORM [2026-04-01] long discussion | Phase 5 moved these to BRAINSTORM as "future: per-locus APS reliability weighting"; partial work landed as oscillation annotation v0.5.47–49 |
| 20 | **IBM-C13** | Per-chromosome APS clustering bootstrapping + APS_mean_of_means clustering test | BRAINSTORM [2026-04-18] × 2 entries | Small analytical experiments; user-gated |
| 21 | **IBM-C14** | Keep/exclude recommendation enhancements (post-Priority 4.9) | ROADMAP Priority 4.9 successor; BRAINSTORM [2026-04-07] | Original 4.9 done v0.4.13; additional refinements moved to BRAINSTORM |

### Cross-reference items already captured in the Phase 14 recon audit

These are *not* new findings — they are listed here only so this inventory is complete.

| ID | Title | Already tracked in |
|---|---|---|
| IBM-X1 | `--hmm-0-based-statepath` opt-in flag | Phase 14 Supplemental FEEDBACK file (live; planned for archive) Finding 8 / Q21 / priority 14-S21 |
| IBM-X2 | `--validate-flags` developer scanner (agent-rejected without user input) | Phase 14 Supplemental FEEDBACK file (live; planned for archive) Finding 11 / Q24 / priority 14-S24 |
| IBM-X3 | `--hmm-smooth-halfwidth` step-5 / step-14 APS split | Phase 14 Supplemental FEEDBACK file (live; planned for archive) Finding 10 / Q23 / priority 14-S23 |
| IBM-X4 | `--aps-rank-by` future choices (summit, width, shape) | PHASE14_SUPPLEMENTAL 14-S6 / KNOWN_ISSUES [2026-04-22:1] |

---

## DETAILED FINDINGS (by cluster)

Each cluster: goal, originating phase(s) and evidence, current state (verified), and a
recommendation. Confidence levels: **high** = strong direct user-voice evidence +
design-complete; **medium** = user-voice evidence with more scoping needed; **low** =
inferred from phase markers only.

---

### IBM-C1 — Internal-code "Stage-1/Stage-2" → "pass 1/pass 2" terminology (detection passes only)

**Origin:** Phase 13 SPEC `## Deferred Loose Ends After Phase 13 Closeout` (Item 1), archived
at `multi-agent/plans/archived/20260420-PHASE13_SPEC.md:4084–4100`.

**Evidence:** The Phase 13 SPEC states explicitly:
> "help strings and user-facing labels that currently say `Stage-1` / `Stage-2` when they
> actually mean algorithm passes should be rewritten as `first pass` / `second pass` …
> internal labels such as `stage1_best_peak` and `stage1_score_z` should eventually be
> renamed to pass-based terms such as `firstpass_best_peak` / `firstpass_score_z` or
> equivalent clearer names"

**What happened:** Phase 14 and Phase 14 Supplemental (Q10 resolved) addressed help-string
terminology for the scan-halfwidth flags — but only for those flags. The broader internal
pass-based terminology rewrite across `onionskin_core/` and test code was never done. Live
code still contains `"Stage-2 peak search radius in kb"` and `"Stage-2 fit window half-size
in kb"` in help text for flags other than the renamed ones, and the internal column names
`stage1_best_peak` / `stage1_score_z` are still present.

**Critical invariant:** Do not rename biological "stage 1/2" (as in posterior stage-1 samples,
stage=1 in replicate stages, `_stage_weight_mode`, etc.). Only the detection-pass meaning
gets rewritten.

**Overlap with Phase 14 recon:** Phase 14 Supplemental Finding 6 / Q13 / priority 14-S14 —
same item, fully captured there. Listed here only because it originated in Phase 13.

**Confidence:** high (explicit user-facing directive in Phase 13 SPEC).

**Recommendation:** Close via 14-S14 decision (already queued).

---

### IBM-C2 — HMM parallel child pipeline + SAPS + step-14 APS raw-file fix

**Origin:** BRAINSTORM.md `[2026-04-19] HMM parallel child pipeline (per-sample individual
decoding) — HIGH PRIORITY` and `[2026-04-18] SAPS — State-APS from individual-sample HMM
decoding`. Overlapping with `[ISSUE:2026-04-19:1]` in KNOWN_ISSUES.md. Scheduled in
`plans/next/HMM_SOUP.md` (HMM-completeness SOUP).

**The three linked items:**

1. **HMM parallel child pipeline** — Extend HMM steps 3 through 11 to emit per-sample outputs
   under `indiv_samples/` subdirectories alongside the existing joint/merged outputs. This
   gives downstream analyses access to correctly normalized per-sample tracks consistent with
   HMM preprocessing.
2. **SAPS (State-APS)** — New analysis that computes APS-like scores directly from decoded
   HMM state paths per sample, not from signal amplitude. Depends on per-sample state paths
   emitted by #1.
3. **Step-14 APS bug fix** — `run_step14_hmm_aps()` currently calls
   `compute_sample_rcn_tracks(manifest, ...)` which re-reads raw bedGraphs and
   re-normalizes them independently, ignoring the step-1–13 preprocessing. It should read
   from `05-medianSmoothedRCN/indiv_samples/` once that exists (requires #1). This is a real
   analytical inconsistency, not just a code-cleanup item.

**Evidence of deferral without user approval:**
- `KNOWN_ISSUES.md [ISSUE:2026-04-19:1]` explicitly says the fix is "Implement the parallel
  child pipeline".
- Phase 12 SPEC explicitly declared this out-of-scope: `"| indiv_samples/ path pre-definition
  | Dead code for unimplemented Phase 13 feature; add when the feature lands |"`.
- Phase 13 SPEC did not pick it up (Priority 13.9 did implement per-stage posterior; HMM
  parallel child pipeline remained untouched).
- Phase 14 was CLI-only; out of scope.
- User feedback that exists: the BRAINSTORM entry describes the design as user-confirmed
  and says **HIGH PRIORITY**. No evidence of user-approved defer.

**Prerequisites for downstream work (what this blocks):**
- HMM summit refinement via `peak_rcn_stage` (IBM-C4) needs per-sample per-stage step-5 outputs.
- HMM sliding-offset sub-bin refinement (IBM-C4) same.
- SAPS analysis (part of this cluster).
- Amplicon reliability scoring prerequisites overlap with this cluster's per-sample data
  access.

**Confidence:** high (user-voice HIGH PRIORITY; design already detailed).

**Recommendation:** Strong candidate for the next phase spec. Substantial work (per-sample
outputs across ~8 HMM steps, plus a new step-15 SAPS with step renumbering timing→16,
clustering→17). Scope carefully before committing.

---

### IBM-C3 — Amplicon reliability scoring + flat-sample detection before APS/clustering

**Origin:** BRAINSTORM.md `[2026-04-19] Amplicon reliability scoring and flat-sample
detection before APS/clustering`. `KNOWN_ISSUES.md [ISSUE:2026-04-19:2]` explicitly claims it
was "PROMOTED to PHASE11_SPEC 11.5" — **but the archived Phase 11 SPEC only ever contained
priorities 11.1–11.3.** This is a classic orphaned promotion: an agent wrote "promoted to
Phase 11.5" into KNOWN_ISSUES, but the actual Phase 11 SPEC was never expanded to hold it.
Later appears in `plans/next/HMM_SOUP.md` (HMM-completeness SOUP).

**Scope (from BRAINSTORM.md and KNOWN_ISSUES.md):**
- Pre-APS per-amplicon reliability scoring on 5 axes: RCN increase across stages, width
  increase, area increase, triangle BIC increase, parabola height increase.
- Composite reliability flag per amplicon.
- Flat-sample (pre-amplification / unamplified) detection on the same evidence axes.
- `--aps-preamp-threshold` (or equivalent) to control flat-sample cutoff.
- APS clustering operates only on amplicons and samples passing reliability.

**Overlap:** IBM-C8B (Stage-1 pre-amp isolation, `[ISSUE:2026-04-14:1]`) is the same
underlying biological problem viewed from the APS-input side. The user's later framing
(BRAINSTORM [2026-04-19]) supersedes the earlier Stage-1-only framing.

**Confidence:** high (explicit user-promoted promotion record; orphaned; still causes APS
cluster distortion).

**Recommendation:** Strong candidate for the next phase. Useful without IBM-C2 (operates on
existing per-amplicon data); even stronger with IBM-C2 in place.

---

### IBM-C4 — HMM summit accuracy improvements (`peak_rcn_stage` column + sliding-offset sub-bin)

**Origin:** BRAINSTORM.md `[2026-04-14] HMM summit estimation — use stage of maximum summit
RCN, not last stage` and `[2026-04-14] HMM summit refinement — sliding-offset sub-bin
localization`. Carry-over in `plans/next/HMM_SOUP.md` (HMM-completeness SOUP).

**Two items:**

1. **`peak_rcn_stage`:** Empirically, stages 2–4 give better origin localization than later
   stages because "elongation interference" shifts the summit peak laterally. The stage of
   maximum summit RCN should be identified per amplicon and used as the authoritative summit
   position source for step-14/15. Adds `peak_rcn_stage` column to
   `hmm_summit_estimates.tsv`.

2. **Sliding-offset sub-bin summit refinement for HMM:** The growth pipeline uses
   `sliding_offset_profile` / `refine_origin_sliding_offset` in `onionskin_core/refinement.py`
   (v0.5.54–v0.5.56) for sub-bin localization without needing hires data. HMM does not have
   this. Should be applied at the `peak_rcn_stage` step-5 smoothed bedGraphs.

**Dependency:** Both items require per-sample per-stage step-5 outputs → depends on **IBM-C2**
(HMM parallel child pipeline).

**Confidence:** high (explicit user observations and design in BRAINSTORM; clearly motivated
by real pipeline accuracy evidence).

**Recommendation:** Bundle with IBM-C2 in the next HMM-completeness phase.

---

### IBM-C5 — Dynamic origin-detection window (cross-pipeline summit refinement)

**Origin:** KNOWN_ISSUES.md `[ISSUE:2026-04-18:3]` and BRAINSTORM.md `[2026-04-18]
Cross-pipeline origin detection windows — dynamic onset / last-active-stage summit
refinement`. Explicitly deferred from Phase 13 Priority 13.3 (see
`archived/20260419-PHASE13_SIDE_QUEST.md:460` — "Additional summit-selector improvement ideas
discovered after closeout are intentionally deferred").

**Scope:** Each amplicon should have its own "origin-detection stage window" (from onset stage
to last-summit-informative active stage) rather than using a fixed early-stage set. Current
`early-parabola-mean` selector is a coarse first pass. Biologically, summit/origin refinement
should be confined to the per-amplicon active window because late stages have
"elongation interference" that shifts the summit peak laterally.

**Status of the deferral:** The user's own voice in BRAINSTORM says "stay in planning mode
and extend the new BRAINSTORM entry into a more explicit dynamic onset / last-active-stage
design sketch for all three pipelines." So the user **explicitly wanted design work to
continue**, not for the item to be parked. This is a design-mode deferral that was never
followed through with a design sketch.

**Exit condition (from KNOWN_ISSUES):** "The BRAINSTORM entry is expanded into a concrete
cross-pipeline design sketch that identifies candidate signals, per-pipeline differences,
and a plausible implementation order for dynamic origin-detection windows."

**Confidence:** high (user-voice explicit design-mode direction; substantial existing
discussion in BRAINSTORM).

**Recommendation:** Next step is a design session (spec-engineering, not implementation). The
design informs later summit-optimization phases.

---

### IBM-C6 — Post-HMM deferrals from Phase 5 (APS, posterior, summit ↔ timing)

**Overall framing:** Phase 5 closed with several priorities explicitly marked `✗ DEFERRED
(post-HMM → see BRAINSTORM.md)`. Phase 7 (HMM integration) closed substantially complete in
v0.7.20. Phases 8–13 did not revisit these deferrals. They are now legitimately post-HMM and
should be triaged rather than left as silent backlog.

#### IBM-C6A — APS order stability (bootstrap diagnostics)

**Origin:** ROADMAP Priority 5.3 `✗ DEFERRED (post-HMM → see BRAINSTORM.md)`.

**Scope:** Bootstrap diagnostics for APS ordering robustness — loci resampling, samples
resampling, raw vs z-score clustering stability. Full design in `BRAINSTORM.md [2026-04-07]`.

**Confidence:** medium (user-approved-deferred; design preserved; easy to revive; no explicit
re-approval yet).

#### IBM-C6B — Better posterior ordering logic

**Origin:** ROADMAP Priority 5.4 `✗ DEFERRED (post-HMM → see BRAINSTORM.md)`.

**Scope:** Refine scalar / dendrogram / hybrid ordering relationship for posterior grouping.

**Confidence:** medium.

#### IBM-C6C — Stage-activity fold-change using refined summit (`timing.py` refactor)

**Origin:** ROADMAP Priority 5.0.2 (carried forward from Phase 4) `✗ DEFERRED (post-HMM)`.

**Scope:** Non-trivial refactor of `timing.py` to recompute stage-activity fold-change using
refined summit positions. Explicitly "deferred until after Phase 7 HMM integration."

**Confidence:** medium.

#### IBM-C6D — Timing-guided active-stage weights for summit estimation

**Origin:** ROADMAP Priority 5.9.3 `✗ DEFERRED (post-HMM)`.

**Scope:** Timing-guided `--stage-weight-mode timing`, center-weighted active window,
per-stage parabola validity as data-driven weight proxy. Depends on timing module stability
and HMM integration for active-window estimates.

**Confidence:** medium.

#### IBM-C6E — Iterative summit ↔ timing convergence (EM-like)

**Origin:** ROADMAP Priority 5.9.4 `✗ DEFERRED (post-HMM)`.

**Scope:** EM-like iteration between summit estimation and timing active-window. Depends on
IBM-C6D.

**Confidence:** medium (design speculative; dependency chain).

#### IBM-C6F — Prior/posterior profile similarity investigation

**Origin:** ROADMAP Priority 5.9.6 `✗ DEFERRED (post-HMM)`.

**Scope:** Investigate whether prior and posterior RCN profile panels being nearly identical
is (a) biological, (b) inspector pooling bug. Requires a dataset where prior/posterior
groupings differ substantially. User-gated.

**Confidence:** low–medium (needs specific dataset; may no longer be a concern if HMM
posterior has since resolved the similarity).

---

### IBM-C7 — Post-HMM deferrals from Phase 6 (APS feature evolution)

**Overall framing:** Phase 6 closure note explicitly moves two priorities to BRAINSTORM as
"post-HMM" deferrals. HMM has landed; these are due for triage.

#### IBM-C7A — APS dimensionality reduction / embedding

**Origin:** ROADMAP Priority 6.2 `✗ DEFERRED (post-HMM → see BRAINSTORM.md)`.

**Scope:** PCA, UMAP, diffusion-like developmental embeddings. Better visualization of
posterior trajectories.

**Confidence:** medium.

#### IBM-C7B — Shape-aware APS clustering

**Origin:** ROADMAP Priority 6.3 `✗ DEFERRED (post-HMM → see BRAINSTORM.md)`.

**Scope:** Cluster using amplicon morphology, not just integrated magnitude. Post-HMM.

**Confidence:** medium.

---

### IBM-C8 — APS cluster quality / RCN floor / pre-amplification

#### IBM-C8A — `--aps-area-excess-floor on|off` flag (RCN floor removal experiment)

**Origin:** `KNOWN_ISSUES.md [ISSUE:2026-04-18:1]` proposes this flag. Underlying issue
originates in Phase 4–5 APS design.

**Already captured:** Phase 14 Supplemental FEEDBACK file (live; planned for archive)
Finding 9 / Q22 / priority 14-S22. Listed here for cross-reference only.

**Confidence:** high (concrete design + clear CLI flag proposal).

#### IBM-C8B — Stage-1 pre-amp isolation (principled flat-sample detection)

**Origin:** `KNOWN_ISSUES.md [ISSUE:2026-04-14:1]`. Originates in Phase 4/5 APS
implementation context.

**Scope:** APS clustering currently cannot force pre-amp samples into a specific stage. User
spelled out a principled approach based on area-excess thresholds and/or triangularity
scoring (amplicon-set refinement + sample-set refinement). Extensive discussion in the
KNOWN_ISSUES entry.

**Overlap:** IBM-C3 (amplicon reliability scoring) is the same biological problem viewed from
a complementary angle. Implement both together or decide one supersedes the other.

**Confidence:** high (concrete algorithmic design in KNOWN_ISSUES; clear biological
motivation).

---

### IBM-C9 — External signal-track ingestion for APS features (`--feature-track NAME:PATH`)

**Origin:** ROADMAP Priority 5.6 `✗ DROPPED`. Successor design preserved in BRAINSTORM.md
`[2026-04-07] Additional signal tracks for APS clustering features (from Priority 5.6)`.

**Scope (from BRAINSTORM):** Generalized `--feature-track NAME:PATH` interface. Replaces the
earlier "external feature matrix input" design (which required the user to pre-compute the
matrix). Onionskin ingests bedGraph-format signal tracks, computes per-amplicon features
within each amplicon window or at summit ± X kb, adds them to the APS feature matrix.

**Status:** Priority 5.6 was user-approved-dropped, but the successor `--feature-track`
design in BRAINSTORM was not explicitly dropped. Ambiguous user position.

**Confidence:** low (user-gated; revive only on explicit ask).

---

### IBM-C10 — Robustness dataset expansion

**Origin:** ROADMAP Priority 5.12 `✗ DEFERRED (post-HMM → see BRAINSTORM.md)`. Explicit
gate: "Do not add datasets without user confirmation."

**Scope:** Tier 1 (replicates) and Tier 2 (other organisms) benchmarking. User-gated.

**Confidence:** low (explicitly user-gated).

---

### IBM-C11 — Cross-pipeline synthesis / `04-unified-results/`

**Origin:** Originally described in Phase 7.4 architecture note. Formally scoped as
**Phase 11 Priority 11.6 "Explicit synthesis boundary"** in the archived Phase 11 SPEC. The
Phase 11 SPEC closed with only 11.1–11.3 implemented; 11.6 was carried forward in the live
scaffold (`plans/next/FWD-ARCH_SOUP.md` — Forward Architecture SOUP).

**Scope:** Define the future synthesis interface above pipeline-local outputs without
approximating it through centralized intermediates. First target is call-level synthesis
across pipeline-local prior outputs.

**Status:** This is a *planned future phase*, not a missed item. Listed here only for
completeness, because the phrase "Priority 11.6" appears in the archived Phase 11 SPEC but
was not delivered within Phase 11 proper.

**Confidence:** not applicable (planned future phase, not a deferral-without-approval).

**Recommendation:** No action from this inventory. Track in `plans/next/` as the user sees
fit.

---

### IBM-C12 — APS locus diagnostics (`best_onset_stage`, `post_support`, `dip_rate`)

**Origin:** BRAINSTORM.md `[2026-04-01] APS locus diagnostics — design discussion and open
questions` — very long user-voice design thread from Phase 5 era.

**Scope:** Three per-locus diagnostic signals that were designed but left as
placeholders/partial implementations:
- `best_onset_stage` — the stage at which a locus becomes credibly amplified (partially done
  via v0.5.47–49 oscillation annotation)
- `post_support` — per-locus posterior support for amplicon credibility
- `dip_rate` — signal for summit RCN declining in late stages (potential elongation-only
  phase marker)

**Status:** Some related work landed (oscillation annotation). The per-locus reliability
weighting goal was explicitly marked "off ROADMAP as of v0.5.49" — but this is an
agent-flagged move not necessarily a user approval.

**Confidence:** medium (complex design, partial implementation, unclear user position on
remaining scope).

---

### IBM-C13 — Per-chromosome APS clustering bootstrapping + APS_mean_of_means clustering test

**Origin:** BRAINSTORM.md `[2026-04-18]` × 2 entries.

**Scope:** Two small analytical experiments:
1. Per-chromosome APS clustering robustness bootstrapping.
2. `APS_mean_of_means` (aggregate across chromosome means rather than across samples
   directly) as a clustering input alternative.

**Status:** Experimental proposals; never picked up. User-gated.

**Confidence:** low (speculative; low-risk to defer; low-risk to pick up).

---

### IBM-C14 — Keep/exclude recommendation enhancements (post-Priority 4.9)

**Origin:** ROADMAP Priority 4.9 was DONE v0.4.13. Additional refinements moved to
BRAINSTORM.md `[2026-04-07] Keep/exclude recommendation enhancements (from Priority 4.9)`.

**Scope (from BRAINSTORM):**
- Shape-score proximity to threshold as a soft signal
- Timing flag (late + fragmented = suspicious)
- `--keep-amplicons-bed` user override for coordinates that are always retained

**Status:** Original 4.9 done. Enhancements partially-resolved via Phase 15: SPEC15.6 cycle 15.4a landed `--known-reliable-amplicons` (synonym of `--keep-amplicons-bed`) for the user-override portion; reliability-scoring framework (Phase 15 SPEC15.6) absorbed shape-score-proximity and timing-flag concepts into the broader cross-pipeline reliability + flat-sample classifier work. Further reconciliation tracked at `KNOWN_ISSUES.md [ISSUE:2026-04-29:5]` part-b (gap-analysis ↔ reliability framework reconciliation; Phase 15 partial-resolution per SPEC15.22 cycle 15.4b; full reconciliation deferred to a future phase per the mid-phase amendment 2026-04-29). Cycle 15.10a Stage D F11 closeout-state verification: confirmed.

**Confidence:** medium (concrete enhancements; clear user motivation; low-risk additions).

---

## APPENDIX A — Per-phase index (brief)

Compact view of where each item originated. Use this when asking "what was missed in Phase
X?"

### Pre-phase refactor plans

- `20260407-CODE-UNIFICATION-REFACTOR.md` — became Phase 6.5. No material deferrals found.
- `20260409-PUFFSTEP-INTEGRATION-PLAN.md` — became Phase 7. No material deferrals found
  outside what ROADMAP 7.1/7.2 already capture.
- `20260412-ROADSTRAVELED.md` — retired; historical only. Content folded into BRAINSTORM
  already.

### Phase 4 — Output completeness (✓ SUBSTANTIALLY COMPLETE)

- IBM-C6C: 5.0.2 (Stage-activity fold-change) carried forward from Phase 4, then deferred.
- IBM-C14: Priority 4.9 keep/exclude enhancements.

### Phase 5 — Posterior reruns and APS evolution

- IBM-C6A: 5.3 APS order stability (DEFERRED post-HMM).
- IBM-C6B: 5.4 Better posterior ordering (DEFERRED post-HMM).
- IBM-C6D: 5.9.3 Timing-guided stage weights (DEFERRED post-HMM).
- IBM-C6E: 5.9.4 Summit ↔ timing EM (DEFERRED post-HMM).
- IBM-C6F: 5.9.6 Prior/posterior similarity (DEFERRED post-HMM).
- IBM-C9: 5.6 External feature matrix (DROPPED; successor in BRAINSTORM).
- IBM-C10: 5.12 Robustness datasets (DEFERRED post-HMM; user-gated).
- IBM-C12: APS locus diagnostics (partial; moved off ROADMAP).

### Phase 6 — APS feature evolution

- IBM-C7A: 6.2 Dim reduction (DEFERRED post-HMM).
- IBM-C7B: 6.3 Shape-aware clustering (DEFERRED post-HMM).

### Phase 6.5 — Code unification

- No material deferrals. All priorities delivered.

### Phase 7 — HMM integration (✓ SUBSTANTIALLY COMPLETE)

- Priority 7.1 (◑ PARTIAL) and 7.2 (◑ PARTIAL) — the ROADMAP marks both PARTIAL, but the
  notes show substantial completion via Phases 8 and 9. The PARTIAL marker is stale rather
  than indicating genuine missed work. No new finding.

### Phase 8 — Full PuffStep parity

- No material deferrals. All priorities delivered.

### Phase 9 — HMM as standalone pipeline

- Priority 9.5 (All-stage HMM) — explicitly **deferred to post-Phase-10** with user approval.
  Not missed; awaiting cross-pipeline contract work.
- Priority 9.6 (`--hmm-optimize`) — explicitly **deferred until post-unification** with user
  approval. Not missed.
- IBM-C4 (`peak_rcn_stage`, sliding-offset sub-bin for HMM) — raised in BRAINSTORM after
  Phase 9 closed, tagged as a Phase 12 carry-over in live scaffold, never implemented.

### Phase 10 — Cleanup closeout

- No material deferrals remain after Priorities 10.1–10.6 closed. Standalone per-stage
  posterior gap, noted as 10.4 issue, was resolved in Phase 13 Priority 13.9.

### Phase 11 — Forward architecture (closed at 11.1–11.3)

- IBM-C11 (11.6 Explicit synthesis boundary) — carried forward as a planned future phase.
- "CLI terminology harmonization" — deferred from Phase 11 Gemini wrap-up Category 4 →
  became Phase 14. Resolved.
- IBM-C3 — `[ISSUE:2026-04-19:2]` claims "PROMOTED to PHASE11_SPEC 11.5" but no 11.5 ever
  existed in the archived Phase 11 SPEC. Orphaned promotion.

### Phase 12 — HMM/growth layout restructuring

- Phase 12 explicitly declared out-of-scope:
  - Per-stage internal stepification (later done in Phase 13).
  - IBM-X1 `--hmm-0-based-statepath` flag (captured in Phase 14 recon).
  - `indiv_samples/` path pre-definition (tagged as "Phase 13 feature; add when lands" —
    never landed → IBM-C2).
- No other deferrals.

### Phase 13 — Per-stage (now RMS) completeness

- IBM-C1: Phase 13 "Deferred Loose Ends" Item 1 — internal `stage1_*` → `firstpass_*`.
- IBM-C5: Dynamic origin-detection window (deferred from 13.3 II/9A investigation).
- Phase 13 "Even More Loose Ends" (Loose Ends A and B) — were **resolved** in v0.13.73–76.
  Not missed.
- Note: "per-stage posterior blocker" comment that Phase 13 started with **was** resolved
  by Priority 13.9. Good.

### Side quest (archived `20260419-PHASE13_SIDE_QUEST.md`)

- Post-side-quest deferred items (summit-selector runtime feasibility work) are captured in
  KNOWN_ISSUES.md entries `[2026-04-18:2]` and `[2026-04-18:4]`. Those are currently analytical
  follow-up items rather than Phase-13-era misses.

---

## APPENDIX B — Cross-references to Phase 14 recon audit

The Phase 14 Supplemental recon audit produced Findings 1–16 (two passes). Several of those
map to pre-14 originating items:

| Phase 14 recon finding | Originating phase | Inventory ID |
|---|---|---|
| Finding 6 / Q13 — internal Stage-1/2 code terminology | Phase 13 Deferred Loose Ends #1 | IBM-C1 |
| Finding 8 / Q21 — `--hmm-0-based-statepath` | Phase 12 explicit out-of-scope; BRAINSTORM [2026-04-18] | IBM-X1 |
| Finding 9 / Q22 — `--aps-area-excess-floor` | Phase 4/5 APS design lineage; KNOWN_ISSUES [2026-04-18:1] | IBM-C8A / IBM-X4 |
| Finding 10 / Q23 — `--hmm-smooth-halfwidth` split | Phase 12 HMM work; KNOWN_ISSUES [2026-04-19:3] | IBM-X3 |
| Finding 11 / Q24 — `--validate-flags` | Phase 14 agent-deferred; originated in Phase 14 | IBM-X2 |

---

## APPENDIX C — What this inventory deliberately excludes

- **Phase 14 and its Supplemental.** Tracked in
  `multi-agent/plans/archived/20260426-PHASE14_SUPPLEMENTAL-FEEDBACK.md` and
  `multi-agent/plans/archived/20260426-PHASE14_SUPPLEMENTAL-SPEC.md`
  (planned for archive after Final Overseer per workflow v2 § Step 8).
- **Phases 15+.** Live scaffold files in `plans/next/` (filenames 15/16/17; internal bodies
  describe themselves as 12/13/14 due to stale numbering from an earlier planning pass).
  Out of scope per user direction.
- **User-approved drops.** Priority 4.10 (`--refine-summits` post-hoc) and Priority 5.0.3
  (same, standalone re-run variant) were explicitly user-approved drops. Not revived.
- **Purely analytical follow-up work** with no CLI/architecture impact unless the user
  specifically wanted it captured. (A small number of these were included in IBM-C13 / IBM-C14
  because they carried concrete design content; most were not.)
- **Housekeeping / stale-reference cleanup.** Not catalogued as missed work.

---

## APPENDIX D — How to use this inventory

- **Triage pass.** Read the top quick-peruse table. Mark each row "pick up now / pick up
  later / drop / already done (please verify)."
- **To add to a future SPEC.** The detailed entries are sized to be promoted into a phase
  SPEC priority with minor reformatting. Each entry names the originating phase, evidence
  source, and confidence level so SPEC-engineering can weigh them.
- **To drop explicitly.** Record the drop in `DECISIONS.md` with rationale (per
  AGENT_CONVENTIONS.md), then remove the entry from this inventory.
- **To convert to KNOWN_ISSUES.** If an item is concrete and near-term but not yet SPEC-ready,
  promote to `KNOWN_ISSUES.md` and note the landing in this inventory.

This inventory itself is not a plan. It is a detection surface.
