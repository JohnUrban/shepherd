# Background-region thinking: per-phase deliberation scratchpad

**Created:** 2026-05-01 EDT — to preserve the multi-turn deliberation between user + Claude Code 2.1.118 (claude-opus-4-7) about how chrom-median + chrom-MAD background-region estimation should work across cycles 15.4a-S3 + SPEC15.15 + cycle 15.7a + downstream consumers (flat-sample test sigma source, ODW transition test sigma source).

**Lifecycle:** Per-phase planning artifact. Tracked. Archives alongside SPEC / AUDIT_LOG / STRATEGY / FEEDBACK at phase close (`multi-agent/plans/archived/<YYYYMMDD>-PHASE15_BACKGROUND.md`). Do NOT delete. Scope: background-region thinking only — not a general phase synthesis surface; cycle narratives live in AUDIT_LOG.

**See also:** `background.txt` (user's parallel raw-notes file — read but DO NOT edit). The user's most recent thinking lives below the `MOST RECENT NOTES START BELOW` line in that file.

---

## What's locked / already in planning (pre-this-conversation)

- **SPEC15.2** — Shared gap mask + missingness diagnostic (controller pre-pipeline step). DONE. Status: gap mask is BED-based; not currently applied to RCN tracks at the per-pipeline emission step.
- **SPEC15.15** — HMM two-pass background-region-anchored chrom-renorm. READY (queued at cycle 15.7a). HMM-only by current design. Pass-1 = run HMM steps 1–8 normally; pass-2 = use background regions = complement of HMM-classified above-background (using widest representation across stages). Layout: pass-1 archived under `pass1/` after pass-2 transitions; sentinel `.pass_metadata.json` for atomicity.
- **Cycle 15.4a-S3** (current, R1 audit complete 2026-05-01) — sigma-source dispatch via new `--flat-sample-sigma-source` flag. Default TBD post-Stage-B.6 calibration; safe-fallback `within-stage-mad` if calibration ambiguous. **R1 audit pushback (F4):** chrom-MAD-on-whole-chrom (sigma_log2 ≈ 0.371 for SAG40 stage 1 chr-II) is LARGER than within-stage MAD (0.309) at the SAG40 II:33175 worked-example locus on the live `dev/share/res20260430-1/` fixture. Under cycle-15.4a-S2 tuned thresholds (0.7/1.10), BOTH sigma sources already pass that case. The `chrom-mad` default is NOT a foregone conclusion empirically.
- **`--chrom-estimation-mode whole|background`** — proposed earlier this session as a master escape hatch. User pushed back during background-tmp deliberation: "but this was using more principled background regions wasn't it?" — open question whether this is redundant with granular flags or is the right master switch.

## What's new (proposed this session, not yet locked)

### From last response (user's notes file lines 8–32)
- Pass-1 gets STRUCTURAL exclusions before any HMM has run: gap-mask, chrom-trim
- Pass-1 RCN-bounds heuristic `[0.25, 1.75]` via `--first-pass-background-bounds LO,HI` (default 0.25,1.75; legacy `0,inf`)
- Pass-2 broadens beyond SPEC15.15's complement-of-above-background: also exclude gap, chrom-trim, 50kb flanks of above-background
- `--second-pass-background-floor FLOOR` (default 0)
- `--second-pass-background-flank-bp BP` (default 50000)
- First-pass + second-pass chrom-medians + chrom-MADs reported to screen like SPEC15.2 missingness; pass-1 vs pass-2 comparison surfaced
- Cross-pipeline gap-bin → NaN handling for median/MAD computations (NOT for bedGraph outputs — user clarified this in MOST RECENT NOTES line 106)

### From assistant pushback (last assistant response)
- F2 cleanup: separate `--flat-sample-detector {inherit, hardthreshold, gaussian_overlap, bayesian_posterior}` flag (default `inherit`), reserves `--flat-sample-sigma-source` for sigma-source-only with 4 values (`within-stage-mad`, `chrom-mad`, `contrib-spread`, `fixed`)
- Q1: Pass-1 bounds applied to chrom-median-normalized RCN (compute naive chrom-median → divide → keep [0.25, 1.75] → recompute on kept set). User confirmed (notes line 90, 127).
- Q2: Cross-pipeline second-pass — user confirmed yes (notes line 95, 129) but flagged Growth Track may not make sense (it IS the amplicon definer for that pipeline)
- Q3: Pass-1 background = controller pre-pipeline (like SPEC15.2) vs per-pipeline applied to shared mask (option iii). User uncertain (notes line 132–133).

### From user's MOST RECENT NOTES (background.txt lines 49–134)
- **Pass-1 bounds applied to chrom-median-normalized RCN with INTERSECTION across samples** (notes line 100): single shared background mask, not per-sample masks. Amplicon regions excluded from all samples by virtue of being above-cutoff in any.
- **Underestimate of MAD acceptable** (notes line 103) — but user is questioning whether to keep the pass-1 heuristic concept at all. Quote: *"not even sure I want to keep the idea — the hmm was itself the best background estimator. Same with the other pipelines to different extents..."*
- **Gap-bin → NaN** (notes line 106): for median/MAD computations only; NOT for bedGraph outputs (no NaN in output files).
- Three candidate methods user enumerated for first-pass background estimation (notes lines 49–53):
  - (i) Median-norm → RCN-bounds filter → redo median-norm
  - (ii) Median-norm → mixture model isolate Gaussian centered at 1 → cutoff → exclude
  - (iii) Median-norm → above-background classification → merge → new background

### The "square 1" realization (user, this turn)
User has identified that all three first-pass methods are essentially heuristic reinventions of what the HMM already does principled-ly. Quote (paraphrased from user message): *"you realize that another way you could estimate the background bins is to run the HMM like we do, then take a union of all the above background bins determined by the state path. And that brings us back much closer or exactly to our original idea — first pass as-is, second pass using background regions informed by the HMM. And then you ask yourself — ok, but do we want a HMM-naive version for the first pass. And now we are at square 1 again."*

**This is actually a clarifying insight, not a regression.** The implicit question now is: **do we need a HMM-naive first-pass background concept at all?**

## Three deliberation outcomes possible

### Outcome A — Drop the heuristic pass-1 entirely; keep current behavior + SPEC15.15-as-designed
- Pass-1 = whatever each pipeline does today (whole-chrom median + within-stage MAD)
- Pass-2 = SPEC15.15-as-designed (HMM-informed, cycle 15.7a)
- Cycle 15.4a-S3 narrows back to original scope (sigma-source dispatch + F2 cleanup + gap-bin → NaN)
- `chrom-mad` sigma source uses naive whole-chrom MAD (the F4 empirical pushback says this may not improve over within-stage; default-flip safe-fallback to `within-stage-mad`)
- `chrom-mad-background` sigma source becomes available AFTER SPEC15.15 cycle 15.7a lands; could be a follow-up cycle 15.4a-S4 or opt-in extension at that time
- **Pros:** simplest; honors the "HMM is the best background estimator" insight; no re-deliberation needed
- **Cons:** SAG40 case stays as-resolved-by-cycle-15.4a-S2 (passes at tuned thresholds); doesn't deliver the "tighten thresholds via stable noise estimate" benefit until SPEC15.15 lands

### Outcome B — Keep heuristic pass-1 as opt-in but minimal
- Pass-1 default = legacy whole-chrom (current behavior)
- Add `--first-pass-background-bounds LO,HI` as opt-in for users who want pre-HMM heuristic background estimation
- No reporting/diagnostic surface for pass-1 medians (drop that scope per user's notes line 78–80)
- SPEC15.15 cycle 15.7a delivers the principled pass-2
- **Pros:** preserves the option for users who want a quick chrom-median refinement before HMM; doesn't bloat default behavior
- **Cons:** extra flag for marginal benefit; user is leaning toward "the HMM does this better"

### Outcome C — Full unified architecture per the consolidated flag set in last assistant response
- Pass-1 + Pass-2 + cross-pipeline + diagnostic surface + intersection-across-samples mask + median-norm-iterate
- Big scope expansion; requires SPEC15.15 cycle 15.7a re-scoping
- **Pros:** most thorough; principled cross-pipeline framework
- **Cons:** doesn't deliver HMM-pipeline-actually-working ASAP; reinvents HMM via heuristics for the pass-1 step; user's "square 1" insight argues against this

**Current user lean:** likely Outcome A or B. User has not yet decided. Will resume conversation later.

## Outstanding clarifying questions for user

1. **Do we even want a HMM-naive first-pass background concept?** (User's "square 1" question.)
2. **If yes (Outcomes B or C):** which of the three first-pass methods — RCN-bounds filter, mixture model, or HMM-then-union? User's notes lean toward (i) RCN-bounds for simplicity OR back to current = no first-pass refinement.
3. **`--chrom-estimation-mode whole|background`** — keep as master switch or retire in favor of granular flags? User pushed back on retiring; needs adjudication.
4. **Cross-pipeline second-pass** — yes per user but Growth Track concern: Growth pipeline's amplicon-detection IS the second-pass equivalent already. Need to think about what second-pass means for Growth specifically.
5. **Q3 (controller pre-pipeline vs per-pipeline w/ shared mask)** — user uncertain.

## Consolidated flag-set proposal (still on the table; subject to outcome A/B/C decision)

**New flags (added by cycle 15.4a-S3 expanded scope):**
- `--first-pass-background-bounds LO,HI` (default `0.25,1.75`; legacy `0,inf`) — applied to chrom-median-normalized RCN, intersection across samples, two-iteration converge.
- `--chrom-trim FRACTION` (default `0.001`; max `0.01`) — excludes first/last X% bins per chromosome.
- `--second-pass-background-estimation off|on` (default `off`; future default `on` per BRAIN15.25). Triggers SPEC15.15-style two-pass with refined background.
- `--second-pass-background-floor FLOOR` (default `0`).
- `--second-pass-background-flank-bp BP` (default `50000`).
- `--flat-sample-sigma-source {within-stage-mad, chrom-mad, contrib-spread, fixed}` (default TBD post-Stage-B.6 calibration).
- `--flat-sample-detector {inherit, hardthreshold, gaussian_overlap, bayesian_posterior}` (default `inherit`) — F2 cleanup.
- `--flat-sample-rcn-sigma`, `--flat-sample-log2-sigma` (for `fixed` mode).

**Retired (proposed; user pushback on first one):**
- `--chrom-estimation-mode whole|background` — replaced by granular flags. **USER PUSHBACK** — reconsider.
- `--hmm-bkgrd-renorm` (SPEC15.15 deliverable 1 candidate) — replaced cross-pipeline by `--second-pass-background-estimation`. Worth confirming.

## Concrete F1-F11 status (from cycle 15.4a-S3 R1 audit)

- **F1** (CLI flag + dispatch needed) — confirmed; load-bearing for cycle scope; not band-aid.
- **F2** (`hardthreshold` is detector value not sigma source) — RESOLVED via separate `--flat-sample-detector` flag per assistant proposal; user agreed.
- **F3** (`fixed` mode needs `--flat-sample-rcn-sigma` + `--flat-sample-log2-sigma` flags) — confirmed; necessary completion.
- **F4** (chrom-MAD-on-whole-chrom is LARGER than within-stage MAD at SAG40 stage 1 chr-II) — empirical pushback stands; default selection is open question; depends on Outcome A/B/C decision.
- **F5–F11** — out of scope for this deliberation; per R1 audit body.

## Other Phase-15 cycles that can advance in parallel

User asked: *"are there other parts of the SPEC that we can do cycles on while deliberating on this?"*

**Available now (independent of background-region deliberation):**

- **Cycle 15.6a-S1** — Stage D APS analytics carry-forward (SPEC15.9 d1/d3/d4 default-flip + SPEC15.12 quantitative clustering defaults). HARD prereq (cycle 15.6b determinism) satisfied. Has been waiting since v0.14.84. R1+R3 primary Claude Code Opus Max; R2 primary Codex GPT-5.5 Extra High. **Highest-value parallel work.**
- **Cycle 15.7b** — HMM per-stage parabola summit emission for cross-pipeline parity (SPEC15.24). Smaller scope. R1 Opus High; R2 Copilot GPT-5.4 Extra High; R3 Opus High.
- **Cycle 15.8a** — Cross-pipeline analysis-surface parity + plot ports (SPEC15.17 expanded + SPEC15.18). R1 Opus High; R2 Copilot Xhigh.
- **Cycle 15.9a** — Architectural cleanup (SPEC15.21 timing-domain consolidation + ODW threshold split). R1 Opus High; R2 Codex High.

**Coupled to background-region deliberation:**

- **Cycle 15.4a-S3** — current cycle; outcome depends on A/B/C decision.
- **Cycle 15.7a** — HMM-specific late-phase work; INCLUDES SPEC15.15 (the prior-art pass-2 design). May need re-scoping if background-region framework expands cross-pipeline.

**Recommendation:** fire cycle 15.6a-S1 next while deliberating. It's been queued the longest, has no coupling to the background-region work, and clears valuable APS analytics + clustering defaults work that subsequent cycles depend on.

## Late-evening user insight (2026-05-01, before driving home) — KEY UNIFIER

**Quote:** *"the posterior run can actually inherit the background regions learned in prior. The entire 'prior' is like a first pass for the posterior. So gaps and background can both be inherited. Maybe updated - but probably not necessary."*

**Why this is a major simplification:**

1. **Prior→Posterior already IS the two-pass architecture.** The user's prior/posterior framing in onionskin (cycle 15.4a + cycle 15.4a-S2 closeout) is structurally identical to SPEC15.15's pass-1/pass-2 framing — they just had different vocabulary. Prior phase = pass-1 (compute chrom-median + run HMM steps 1-8 + identify above-background regions). Posterior phase = pass-2 (re-run with refined manifest informed by APS clustering, AND now: inherit background regions from prior for chrom-renorm + sigma-source).
2. **No need for a separate first-pass-vs-second-pass flag system inside a single run.** SPEC15.15's `--hmm-bkgrd-renorm on|off` + the proposed `--second-pass-background-estimation` etc. become redundant with the existing prior/posterior structure. The "two pass" happens NATURALLY across `--posterior` invocations.
3. **Gap-mask inheritance already free** via SPEC15.2's pre-pipeline shared gap-mask infrastructure (computed once, all pipelines + phases inherit).
4. **Background-region inheritance is the new addition.** Prior's HMM step 8 above-background regions get serialized as a sidecar (e.g., `<prior-out>/01-hmm/08-summitStates/.../background_regions.bed` or similar). Posterior controller reads that sidecar at startup and threads it into the posterior chrom-renorm + sigma-source computations.

**What this simplifies in the deliberation:**

- **No heuristic pass-1 needed.** Pass-1 = prior run = current behavior. Pass-2 = posterior run = inherits prior's HMM-derived backgrounds. Outcome A (drop heuristic pass-1) wins by construction.
- **SPEC15.15 cycle 15.7a re-scopes** from "two-pass within an HMM run" to "posterior run inherits prior's HMM-derived background regions for chrom-renorm." Smaller scope; uses the prior/posterior structure already in place.
- **The proposed `--second-pass-background-estimation` flag** essentially becomes "does the posterior inherit prior's background regions?" — could default `on` (inherit by default; opt-out with `off` for legacy posterior behavior).
- **Cross-pipeline naturally** — the prior/posterior architecture spans Growth + RMS + HMM. Each pipeline's prior produces above-background regions (HMM step 8 for HMM; analogous calls for Growth/RMS); posterior phase consumes them.

**For Growth Track concern (the recursion problem the user flagged at notes line 130):** Growth's prior produces growth-track-defined amplicons. Growth's posterior inherits those for chrom-renorm. No recursion — the prior's amplicon calls are stable inputs to posterior's renorm; posterior doesn't need to re-classify amplicons before renorm.

**For chrom-MAD-background sigma source (cycle 15.4a-S3):** the posterior phase's flat-test can use `chrom-mad-background` where "background" = prior's HMM-derived background regions. This unifies cycle 15.4a-S3 + SPEC15.15 cleanly: cycle 15.4a-S3 adds the dispatch knob; SPEC15.15 cycle 15.7a (now reframed as "prior→posterior background-region inheritance") delivers the background regions; the two land in sequence and the sigma source becomes useful at SPEC15.15 close.

**Updated outcome-A-prime sketch (resume here):**

- Cycle 15.4a-S3 narrows: sigma-source dispatch + F2 detector cleanup + F3 fixed-mode flags + gap-bin → NaN cleanup. `chrom-mad` sigma source value uses naive whole-chrom MAD (acknowledged limitation per F4); `chrom-mad-background` value is added but raises an error pre-SPEC15.15 (or falls back to whole-chrom with a deprecation-style warning).
- SPEC15.15 cycle 15.7a re-scopes to "prior→posterior background-region inheritance" — possibly cross-pipeline at design time. Reads prior's background regions sidecar (HMM step 8 for HMM; analogous for Growth/RMS); threads into posterior chrom-renorm + sigma-source.
- After SPEC15.15 lands, `chrom-mad-background` sigma source becomes functional in cycle 15.4a-S3's dispatch.

This sequence is cleaner than the heuristic-pass-1 path AND cleaner than the deferred-add-later path I sketched in Outcome A. **My updated lean: Outcome A-prime as outlined above, treating SPEC15.15 cycle 15.7a as the natural delivery vehicle for the background-region inheritance.**

## Resume-conversation update (2026-05-01, late evening) — user RE-OPENS the heuristic pass-1 question

User pushed back on Outcome A-prime. Wants the optional within-prior second pass after all. Key reasoning:

1. **Better background estimation → better prior results.** Even if posterior eventually inherits background regions, the prior's own results benefit from a tighter chrom-renorm.
2. **Prior runs on the user's input manifest** (which the user may want to keep, rather than the posterior-derived manifest). Posterior-with-same-manifest defeats the point of running posterior.
3. **People may not run `--posterior` at all.** Background-quality improvements should work in single-pass prior runs.
4. **Per-pipeline differentiation:**
   - **HMM:** estimates background well; can bootstrap itself (run HMM → identify above-background → re-renorm using complement). Heuristic pass-1 useful but not strictly needed.
   - **RMS:** also catalogs above-background regions; can self-bootstrap similarly.
   - **Growth:** **special case** — growth track itself is NOT good at spotting stable collapsed repeats. Growth pipeline benefits MOST from the heuristic pass-1 forms because it cannot self-bootstrap collapsed-repeat detection.
5. **Two heuristic pass-1 forms user proposed:**
   - **Form (1):** median-norm → clip RCN to [0.25, 1.75] → re-estimate median and MAD on kept bins.
   - **Form (2):** median-norm → learn cutoffs from mixture model (instead of assuming 0.25/1.75) → take union merge across samples for shared mask → optionally: merge intervals within X kb (e.g., 15 kb) → retain intervals at least Y kb (e.g., 25 kb).
6. **Within-prior second pass (per pipeline):** union-merge heuristic-pass-1 background with pipeline-called above-background regions (HMM step 8 above-background for HMM; RMS calls for RMS; growth calls for Growth — which is recursive for growth and that's why heuristic-pass-1 matters for growth).
7. **Posterior:** simply inherits per-pipeline background regions from prior. Cross-pipeline unification is FUTURE work.
8. **Cross-pipeline unification approaches** for "the" background mask, when we get there:
   - (i) intersect — only bins that are background in all pipelines (most conservative).
   - (ii) union — bins that are background in any pipeline (most permissive).
   - (iii) something in between (e.g., majority-vote, weighted-by-pipeline-confidence).

## Outcome D (new; supersedes A-prime per user resume) — proposed

**Recommended split:**

**Cycle 15.4a-S3 (current; minimal scope):**
- Sigma-source dispatch + F2 detector cleanup + F3 fixed-mode flags + cross-pipeline gap-bin → NaN (the original scope).
- Adds `chrom-mad` sigma source value using **naive whole-chrom MAD** for now (the F4 limitation acknowledged).
- Default safe-fallback `within-stage-mad`.
- Cycle stays small; ships quickly; clears the dispatch architecture.

**New Cycle 15.4a-S4 (queue immediately after 15.4a-S3 closes):**
- **NEW:** Controller-level pre-pipeline shared "naive background mask" computation (analogous to SPEC15.2's gap-mask pre-pipeline step). All pipelines inherit. Form (1) only — chrom-median-norm → clip [0.25, 1.75] → intersect across samples → use as mask for chrom-MAD recomputation.
- New flags: `--first-pass-background-bounds LO,HI` (default 0.25,1.75; legacy `0,inf`); `--chrom-trim FRACTION` (default 0.001).
- `chrom-mad-background` becomes available as new sigma source value (sister to `chrom-mad`); uses the new mask-derived chrom-MAD.
- Cross-pipeline emission of per-sample chrom-MAD-background bedGraphs (sister to per-sample chrom-MAD bedGraphs from cycle 15.4a-S3).
- This is what delivers the user's "growth pipeline benefits from heuristic pass-1" goal.

**SPEC15.15 cycle 15.7a (re-scoped):**
- Within-prior second pass (HMM-bootstrap / RMS-bootstrap / growth-amplicon-union-merge).
- Posterior background-region inheritance from prior.
- Form (2) heuristic pass-1 (mixture model + interval merge + length filter) deferred or optional here.
- Cross-pipeline unification approaches (intersect / union / hybrid) deferred to a future phase.

**Future work / not in any current cycle:**
- Form (2) heuristic pass-1 (mixture model approach).
- Cross-pipeline unification of background masks.
- Auto-tuning of HMM transition probabilities to compensate for tighter background estimates.

## Pushbacks worth raising before committing to Outcome D

1. **Scope: cycle 15.4a-S3 stays minimal (good for fast ship); cycle 15.4a-S4 absorbs the heuristic pass-1 work.** Splitting protects the sigma-source dispatch from getting blocked on background-region deliberation. Confirm split or override?

2. **Form (1) only in cycle 15.4a-S4; Form (2) deferred.** Form (1) is simpler + ships faster + delivers most of the value. Form (2)'s mixture-model + interval-merge + length-filter is sophisticated and probably warrants its own cycle (or fits naturally into SPEC15.15 cycle 15.7a as an enhancement). Confirm or override?

3. **For Growth specifically: heuristic pass-1 IS the only viable background detector pre-amplicon-call.** This is the load-bearing argument for keeping the heuristic. If user runs `--pipelines growth` only, no HMM is available, so heuristic pass-1 is the only path to refined background estimation. Confirm this matches user intent?

4. **Within-prior second pass for HMM:** structurally the same as SPEC15.15's existing pass-2 design (re-run HMM steps 1-8 with background-informed renorm). Does cycle 15.7a SPEC15.15 also need to absorb the heuristic pass-1 (so HMM users get pre-HMM heuristic pass-1 + post-HMM bootstrap pass-2)? Or is heuristic pass-1 only useful for growth/RMS pre-amplicon-call?

5. **Posterior inheritance:** the prior's per-pipeline background-region sidecars (HMM step 8 above-background BED for HMM; RMS calls BED for RMS; growth calls BED for Growth) get serialized at known paths. Posterior controller reads them at startup. Cross-pipeline unification is future. Confirm staging.

6. **`--chrom-estimation-mode whole|background`** — under Outcome D this becomes meaningful again: `whole` = legacy (no heuristic pass-1, no within-prior second pass); `background` = use heuristic pass-1 + within-prior second pass + posterior inheritance per the cycle 15.4a-S4 + SPEC15.15 cycle 15.7a delivery. Master switch with sub-flags layered underneath. Adopt this as the user-facing UX; granular flags for finer control.

## LOCKED PLAN (2026-05-01 evening) — ready to execute

**Status:** All key decisions confirmed by user 2026-05-01. Doc updates in flight. Cycle 15.4a-S3 R2 launcher pending after corrections appendix lands.

### Locked decisions (all confirmed by user)

1. **`--background-flank-bp BP`** (default 50000) — single flag for gap flanks Pass-1; gap + above-background flanks Pass-2.
2. **Growth Pass-2 retains "bins > HI" exclusion** (in addition to growth-called amplicons + flanks). HMM/RMS Pass-2 REPLACES "bins > HI" with above-background-regions ± flanks.
3. **No screen reporting of medians/MADs per pass.** Replaced by % stats per chromosome: `% trim`, `% below-LO`, `% above-HI`, `% gap+flanks`, `total % masked`.
4. **Keyword aliases** for `--first-pass-background-bounds`: `legacy`/`none`/`whole` ↔ `0,inf`; `naive`/`background` ↔ `0.25,1.75`. ~5 lines of parser code.
5. **`[ISSUE:2026-05-01:1]` deferred** — see narrowed F10 framing below. Broad claim withdrawn; narrow case (stage-1 anchor detection) may warrant filing after Stage A verification in cycle 15.4a-S4.
6. **3-cycle split confirmed:** 15.4a-S3 (current; minimal) → 15.4a-S4 (NEW; pre-pipeline shared naive background mask) → 15.7a SPEC15.15 (re-scoped: cross-pipeline within-prior second pass + posterior inheritance).

### NARROWED F10 framing (orchestrator pushback against orchestrator's earlier overgeneralization)

Earlier claim was that within-stage between-sample MAD systematic error applies broadly to (a) flat-sample test, (b) ODW active-stage transition test, (c) regression/dip detection. **User correctly pushed back: this conflates two structurally different questions.**

- **Flat-sample test** asks a within-sample-vs-background question but uses between-sample-at-this-bin variance as noise. Mismatch — wrong tool for the question. Cycle 15.4a-S3 + 15.4a-S4 fixes.
- **ODW active-stage transition test for stages N > 1** asks a between-stages-at-this-bin differential-expression question. **Per-bin within-stage between-sample MAD IS the right input** — it's the per-condition replicate variance that DEseq2/EdgeR/limma frameworks require. Same data quantity, different question, correct match. **NOT a systematic error.** Reduced power at high-heterogeneity bins is the right behavior for differential-expression tests.
- **Regression / dip detection** has the same between-stages structure as the transition test — same MAD input is correct. **NOT a systematic error.**
- **Stage-1 active-stage detection** (no previous stage to compare to — Stage 1 in chrom-median norm mode, Stage 2 in ref-stage norm mode) IS structurally a within-stage-vs-background question. **If the current implementation uses per-bin within-stage MAD as noise estimate here, that IS the same systematic error.** Need Stage A verification in cycle 15.4a-S4. Narrow scope, not broad.

**R1's F10 in cycle 15.4a-S3 audit is narrowed.** The vulnerability is real ONLY for the stage-1 anchor detection slice. Stages N>1 transition + regression/dip detection are correctly using the right MAD input for their differential-expression question structure.

### Per-pipeline plan (locked)

**HMM:**
- Default: current behavior.
- With `--first-pass-background-bounds 0.25,1.75`: controller computes shared naive mask before HMM runs; HMM consumes mask for chrom-renorm + chrom-MAD.
- With `--second-pass-background-estimation on`: after HMM step 8 above-background, refined mask = Pass-1 mask but with "bins > HI" replaced by above-background ± `--background-flank-bp` flanks. Re-run HMM steps 2-8 with refined mask. Pass-1 archived under `pass1/`; pass-2 at top-level.
- With `--posterior`: inherits prior's HMM-derived background regions.

**RMS:** Same structural pattern as HMM. Pass-2 trigger uses RMS-called amplicons. With `--posterior`: inherits prior's RMS-derived background regions.

**Growth:** Same structural pattern, with one critical difference — Pass-2 RETAINS "bins > HI" exclusion (in addition to growth-called amplicons + flanks) because growth pipeline does NOT catch stable collapsed repeats by itself. With `--posterior`: inherits prior's growth-derived background regions.

### Universal pre-pipeline steps (controller-level)

- Per-sample chrom-median normalize.
- Per-sample mask: keep bins where RCN ∈ [LO, HI] AND NOT in gap mask AND NOT in gap-flank-bp halo AND NOT in first/last `--chrom-trim` of chromosome.
- Cross-sample reduce: union-of-excluded-bins across samples = shared naive background mask. (Equivalently: intersection-of-included-bins.)
- Pipelines start at "step 2 equivalent" if Stage A confirms median-norm-is-step-1 across all three pipelines.
- Diagnostic: report % stats per chromosome to screen.

### Cycle scoping (locked)

**Cycle 15.4a-S3 (current; awaiting R2 with NARROWED scope):**
- `--flat-sample-sigma-source {within-stage-mad, chrom-mad, contrib-spread, fixed}` (default `within-stage-mad`).
- `--flat-sample-detector {inherit, hardthreshold, gaussian_overlap, bayesian_posterior}` (default `inherit`) — F2 cleanup.
- `--flat-sample-rcn-sigma`, `--flat-sample-log2-sigma` for `fixed` mode (F3).
- Cross-pipeline gap-bin → NaN handling (in-memory only; not in bedGraph outputs).
- `chrom-mad` semantic: pipeline's current chrom-MAD (naive whole-chrom this cycle).
- **Background-mask machinery DEFERRED to cycle 15.4a-S4.** R1's Stage B.1 + B.2 (chrom-MAD-per-sample bedGraph emission) ALSO deferred — without the mask, per-sample chrom-MAD adds little value vs the existing per-stage between-sample MAD bedGraphs.

**Cycle 15.4a-S4 (NEW; queue immediately after 15.4a-S3 closes):**
- Controller-level pre-pipeline shared naive background mask (Form 1).
- New flags: `--first-pass-background-bounds`, `--chrom-trim`, `--background-flank-bp`.
- Per-sample chrom-MAD bedGraph emission across all three pipelines; chrom-MAD computed over the naive mask.
- `chrom-mad` sigma-source value upgrades to mask-derived; cycle 15.4a-S3's dispatch absorbs it transparently.
- Stage A verification: median-norm-is-step-1 cross-pipeline?; stage-1 anchor detection MAD source?
- Diagnostic: % stats per chromosome.

**SPEC15.15 cycle 15.7a (re-scoped to cross-pipeline):**
- Within-prior second pass per pipeline.
- New flags: `--second-pass-background-estimation`, `--second-pass-background-floor`.
- Posterior background-region inheritance from prior (per-pipeline).
- Pass-1 outputs archived under `pass1/`; pass-2 at top-level.

**Future:**
- Form 2 heuristic pass-1 (mixture model + interval merge + length filter).
- Cross-pipeline mask unification (intersect / union / hybrid).
- Stage-1 anchor detection MAD source fix (narrow F10 case).
- HMM transition-probability auto-tuning.

### Flag semantics reference (locked)

| Flag | Where it acts | Default |
|---|---|---|
| `--flat-sample-sigma-source {within-stage-mad, chrom-mad, contrib-spread, fixed}` | flat-sample test sigma source dispatch | `within-stage-mad` (cycle 15.4a-S3 safe-fallback) |
| `--flat-sample-detector {inherit, hardthreshold, gaussian_overlap, bayesian_posterior}` | flat-sample test detector independent of ODW choice | `inherit` (= `--odw-active-stage-detector`) |
| `--flat-sample-rcn-sigma FLOAT`, `--flat-sample-log2-sigma FLOAT` | only for `--flat-sample-sigma-source fixed` | None each → falls back to module constants 0.1 / 0.15 |
| `--first-pass-background-bounds LO,HI` | controller-level Pass-1 mask (cycle 15.4a-S4) | `0.25,1.75` |
| `--chrom-trim FRACTION` | controller-level Pass-1 mask, first/last X% per chromosome (cycle 15.4a-S4) | `0.001` (max 0.01) |
| `--background-flank-bp BP` | flank halo around gaps (Pass-1) + above-background (Pass-2) | `50000` |
| `--second-pass-background-estimation off\|on` | enables within-prior second pass per pipeline (SPEC15.15 cycle 15.7a) | `off` (future `on` per BRAIN15.25) |
| `--second-pass-background-floor FLOOR` | minimum RCN in Pass-2 background (default keeps non-gap 0-bins) | `0` |

**Retired:**
- `--chrom-estimation-mode whole|background` — superseded by granular flags.
- `--hmm-bkgrd-renorm` — superseded by `--second-pass-background-estimation`.

### Flat-sample sigma-source values reference

- **`within-stage-mad`** — per-bin MAD across replicate samples within a single stage (legacy `aps.py:_build_stage_stats:142-170`). NOT chrom-wide. Kept for reproducibility / comparison; the legacy buggy one for flat-sample test purposes. Default in cycle 15.4a-S3 as safe-fallback.
- **`chrom-mad`** — per-sample chrom-MAD over current pass's background mask. Cycle 15.4a-S3: naive whole-chrom MAD (pre-mask). Cycle 15.4a-S4: chrom-MAD over Pass-1 naive mask. Cycle 15.7a SPEC15.15 with `--second-pass-background-estimation on`: chrom-MAD over Pass-2 refined mask.
- **`contrib-spread`** — existing fallback heuristic at `flat_sample.py:_resolve_default_sigma:168-193`. `np.median(np.abs(summit_rcn_values - 1.0))` over the contribution table — MAD-style spread vs baseline 1.0. Per-amplicon-call-derived, not per-bin or per-chrom.
- **`fixed`** — user-supplied sigma values. Use case: deterministic tests, ablation studies.

## Key user direction signals (memorized)

- *"the hmm was itself the best background estimator"* — user is converging toward "trust the HMM, don't reinvent it" framing
- *"first pass as-is, second pass using background regions informed by the HMM. And then you ask yourself — ok, but do we want a HMM-naive version for the first pass."* — explicit articulation that the heuristic pass-1 may be unnecessary
- *"Yes - I would not be surprised if one pipeline does it that way while the others don't. Probably should convert gap bins (not all 0 bins) to NaN"* — gap-bin → NaN cross-pipeline cleanup is wanted; not coupled to pass-1/pass-2 deliberation
- *"shouldn't have NaN in a bedGraph"* — output-format constraint: NaN is for in-memory computations only

## What I think the right next moves are

1. **User resumes deliberation when ready.** No need to force a decision under pressure.
2. **Fire cycle 15.6a-S1 in parallel.** Independent + queued + high-value.
3. **When user resumes:** answer the 5 outstanding clarifying questions above, then pick Outcome A/B/C, then I write the cycle 15.4a-S3 corrections appendix accordingly (or recommend narrowing back to original scope if Outcome A wins).
4. **For SPEC15.15:** likely stays as-is for now (HMM-only, cycle 15.7a), with possible cross-pipeline expansion deferred to a follow-up cycle if Outcome C is chosen.

---

## Mid-cycle decision (2026-05-03) — cycle 15.4a-S4 Stage A.5 GMM refit in log2 space (post-empirical-evidence)

**Status:** Decision recorded for future incorporation into PHASE15_AUDIT_LOG.md cycle 15.4a-S4 section as a SECOND orchestrator clarification, after Copilot completes Stage A.7 authorization-gate pause. Mid-flight pushback note has been injected to Copilot directly; this entry documents the decision history + rationale for the eventual audit-log entry.

**Trigger.** Copilot's first run of the Stage A sweep helper, now tracked at `scripts/fit_first_pass_background_defaults.py`, fitted GMM in raw RCN space and surfaced three symptoms that diagnostic-confirmed the wrong fit space:

1. **V_L undefined in all 9 manifests.** Background was always the leftmost component because deletion-region bins (RCN ∈ [0, 0.5]) absorbed into the background component's left tail. No "left distribution" to compute V_L against.
2. **σ-floor sanity check fired in 4 of 9 manifests.** The widened background σ pushed `μ_bg − 2σ_bg` below 0.25, failing the LO σ-floor.
3. **Third component degenerate** with σ values 9, 11, 13, and 446 — the GMM parked long-tail amplification + outliers in a "bag" component rather than fitting a coherent amplification distribution.

**Decision (orchestrator-direction; user-confirmed 2026-05-03):** refit GMM in log2(RCN) space.

**Rationale.**

- RCN is a multiplicative quantity (2× amplification = +1 in log2; 0.5× deletion = −1 in log2). Biological signals on multiplicative scales are typically lognormal in raw space, i.e., Gaussian in log space. Fit-space alignment with biological ground truth.
- log2(0) = −∞; combined with cycle-15.4a-S3 CR3's `RCN ≤ 0` filter (already established for `_chrom_mad_sigmas_from_track:200-202`), the 0-floor disappears naturally from the input population. No half-Gaussian at the left edge.
- Symmetric treatment of amplification and deletion: 2× amp at +1, 0.5× del at −1. Valley between background and amplification has the same structure as valley between deletion and background.
- Long-tail amplification doesn't hijack a degenerate component — log space compresses the right tail (4× = +2, 8× = +3) into a region with reasonable σ.
- σ-floor sanity check operates in log space (the fit's natural geometry); converted to raw space only for the user-facing cutoff comparison + hard guard rail application.

**Algorithm (replaces F8(4)-(7) for the Stage A.5 sweep + Stage B.3 `learn` mode implementation):**

1. Population construction: gap+flank+chrom-trim filter (unchanged) → additionally filter `RCN ≤ 0` (consistent with cycle 15.4a-S3 CR3) → `values_log2 = np.log2(values)` → filter non-finite (catches log2(0)=−∞ defensively).
2. Fit 2-3 component GMM in log2 space; BIC selects k (per F8(8)).
3. Background component = `argmin(|μ_log − 0.0|)` (log2(1) = 0 is background).
4. Compute valley points V_L_log, V_R_log in log2 space using existing joint-PDF argmin / PDF crossing primitives.
5. Convert to raw space: `V_L_raw = 2^V_L_log`, `V_R_raw = 2^V_R_log`.
6. Apply hard guard rails in raw space (unchanged): `LO = max(V_L_raw, G_L_hard=0.25)`, `HI = min(V_R_raw, G_R_hard=1.75)`.
7. σ-floor sanity check in log space: `LO_log = log2(LO)`, `HI_log = log2(HI)`; `LO_log ≤ μ_bg_log − N·σ_bg_log` AND `HI_log ≥ μ_bg_log + N·σ_bg_log`. Failure logs warning + flags for R3 review; does NOT auto-widen the cutoffs (guard rails are intentional).

**TSV schema (extends F9 schema in PHASE15_AUDIT_LOG.md cycle 15.4a-S4 corrections appendix):** add `_log2`-suffixed sister columns for component μ/σ and valley points; keep raw-space columns (`gmm_component_*_mu`, `valley_point_*`, `LO_final`, `HI_final`) populated with `exp2(log_value)` for backward compatibility with the v1 TSV.

**Hard guard rails stay locked.** G_L_hard=0.25, G_R_hard=1.75 unchanged. The point of refitting in log space is to make the fit ITSELF well-behaved + give us a real V_L for diagnostic visibility — not to change the (LO, HI) values that get baked as static defaults. On most manifests the guard rails will still win; the difference is that V_L will be defined (not "undefined; LO fell back to G_L_hard") and σ-floor failures should mostly resolve.

**Predicted post-refit outcomes** (for sanity-check confirmation when Copilot reruns):
- V_L_raw lands in [0.4, 0.7] across most manifests — no longer "undefined".
- V_R_raw lands in [1.4, 2.0] across most manifests.
- LO_final = max(V_L_raw, 0.25) — likely > 0.25 on most manifests.
- HI_final = min(V_R_raw, 1.75) — guard rail wins on high-amplification manifests; valley wins on cleaner ones.
- σ-floor failures (4 of 9 in raw space) should mostly resolve.

**Forward-pointers.**
- Mid-flight pushback note delivered to Copilot 2026-05-03; awaiting refit + updated Stage A diagnostic report at A.7 authorization gate.
- After Copilot completes Stage A.7, orchestrator appends formal clarification to PHASE15_AUDIT_LOG.md cycle 15.4a-S4 section (SECOND orchestrator clarification, post-Stage-A.5-results; same pattern as the 2026-05-03 first clarification on F8(5) cutoff derivation rule).
- F9 TSV schema extension (log-space sister columns) gets recorded in the formal audit-log clarification.







#####################################################################################################################################
## User-only section (no edits by other agents)
## No agent work below this line
#####################################################################################################################################

RE: F1 - Got it. Makes more sense now.

RE: F2 - Yes I like this.

RE: F3 - Got it.

RE: F4 and background regions, pass 2, etc
- Pass 1 background definition 
	- it sounds like we are converging on more intelligently using background regions for pass 1 as well.
	- we might not have identified the amplicons yet, but we can still mask gaps and chromosome ends as planned
	- Moreover, we can use some type of heuristic similar to RCN ∈ [0.7, 1.3] in the first pass, but I would propose it be more permissive like RCN ∈ [0.25, 1.75], perhaps tunable with `--first-pass-background-bounds 0.25,1.75` (`0,inf`) could be legacy bounds, i.e. no bounds (explain in help text). Although now we might be stepping on the toes of "--chrom-estimation-mode whole|background" so this all needs to be sorted out.
		--> RCN ∈ [0.25, 1.75] probably would be sufficient for all those conditions, but we can enforce all the conditions to be thorough.
	- So first pass background estimation can be something like all bins except: gap bins, bins within first or last X% of chromosome (default 0.1%), bins with RCN <0.25, bins with RCN >1.75.
	- That should give a better background estimation even on the first pass, even if it is  little forced; my only concern is it would underestimate the MAD.
	- BUT it would be more consistent across the stages that have different levels of amplification, so it is a trade-off
	- A "risk" is this might change the results of the HMM a little, but it might be overall for the better, and we could tweak things like transition probabilities if needed.
	- This would give first pass chromosome-specific medians and MADs --> these should be reported to screen the same way "missingness" is.
	- these can be used for normalization and for the flatness stuff later; it would already be pre-computed
		- but if second pass is enabled, I think the pipeline goes back to the start directly after getting the "above background" regions.
		- so if some type of "--second-pass-background-estimation" flag is used, then those are the chromosome-specific medians and MADs passed downstream to subsequent analyses like flatness detection
		- making sense?
- Pass 2 background definition is now too narrow
	- current definition was just the complement of the called regions (amplicons and collapsed repeats)
		--> Background regions = intervals NOT classified as above-background. For all things above background (amplicons AND collapsed repeats), use their widest representation found across stages (most conservative: bins are background ONLY if they are never seen above background in any stage).
	- But the new definition for background would broaden to be that but also exclude: gap bins, bins within first or last X% of chromosome (default 0.1%)
		-> NOT above background (and NOT flanking above background within tunable 50 kb default), NOT gap, NOT first or last X% of chromosome
	- Anything above background as determined by the HMM takes the place of RCN ∈ [0.25, 1.75]
		- only weakness might be the 2nd pass having no floor : could have a sibling flag to --first-pass-background-bounds for second pass, --second-pass-background-floor (default = 0, keeps all >0 that are otherwise considered background bins)
	- This would give second pass (updated) chromosome-specific medians and MADs --> these should be reported to screen the same way "missingness" is, and should compare 1st pass to 2nd pass to easily see the difference(s) if any.
	
RE: "aps.py:_build_stage_stats (lines 159–162) uses np.nanmedian for both median + MAD, so NaN bins are correctly treated as missing when the upstream pipeline emits NaN for gap bins."
- Yes - I would not be surprised if one pipeline does it that way while the others don't. Probably should convert gap bins (not all 0 bins) to NaN for this to work across pipelines in general.

Can you help package up everything I said with what we were already planning. I have introduced some new things that may or may not work. Push back as needed. We also need to avoid redundancy, settle on flag names, etc.
Then we can move on to something like Option A, B, or C -- updated how you see fit.


Is that all clear? Ask clarifying questions as needed. Instruct me where I was thinking about something different or in the wrong way here. Point out the new ideas, the older ideas, and things that would change based on the new ideas. 


=======================================================================================================================


MOST RECENT NOTES START BELOW
--> just raw notes
--> I have not processed them all yet
--> I have not formed strong conclusions on everything yet

First pass --> gap mask and structural for median
- could median norm, then RCN filter
- could median norm, combine, mixture model to isolate mean closest to 1 and its associated variance -- and cutoffs ... then exclude all higher than cutoff...
- could med norm ---> above background --> merge --> new background

==============

--flat-sample-sigma-source
HMM two-pass: pass-1 = run normally, pass-2 = use background regions for chrom-renorm
- Pass-1 background definition (per SPEC15.15) — just complement of HMM-classified above-background, no structural exclusions, HMM-only
	- Locked (pre-this-discussion)
	- need to unlock
gap mask + missingness
--chrom-trim FRACTION flag (default 0.1%)

-------
New
- Pass-1 gets structural exclusions (gap-mask, chrom-trim) BEFORE any HMM has run
- Pass-1 RCN-bounds heuristic [0.25, 1.75] via --first-pass-background-bounds LO,HI (default 0.25,1.75; legacy 0,inf)
	- RCN bounds for MAD learning
		- but dont need MAD until after knowing true background regions...? if so, no MAD needed
	- RCN bounds also for median updating
		- this might be worthwhile
		- global median norm
			- then redo with values inside 0.25,1.75
- Pass-2 broadens beyond SPEC15.15's complement-of-above-background — also excludes gap, chrom-trim, 50kb flanks of above-background
- --second-pass-background-floor FLOOR, def 0
	= Sister to --first-pass-background-bounds
- First-pass + second-pass chrom-medians + chrom-MADs reported to screen like SPEC15.2 missingness; pass-1 vs pass-2 comparison surfaced
	- may not be necessary if only need MAD later
	- and may not be interesting if median is 1 in both cases after norm
		- if it is global medians of raw counts and how they change, fine, but that is a lot of output, every file diff
- Cross-pipeline gap-bin → NaN handling (so nanmedian works uniformly)
	- yes gap bins should not be part of median norm
	- this might change some results but for the better one hopes, especially for future unknown situations

RE: PUSHBACK
1. --first-pass-background-bounds [0.25, 1.75] — applied to WHICH signal? 
	- Chrom-median-normalized RCN gives a distribution centered ~1.0 by construction, so [0.25, 1.75] = "factor of 4 around chrom-median" is sensible there.
		- Ref-stage-normalized RCN is centered on the polluted stage-1 median; [0.25, 1.75] could carve out the wrong region.
	- 
YES -> This is the idea -> compute naive chrom-median → divide → keep bins with normalized RCN ∈ [0.25, 1.75] → recompute chrom-median + chrom-MAD on the kept set

2. --chrom-estimation-mode whole|background is now redundant.
	- pushback -- but this was using more principled background regions wasn't it?

3. YES: Growth + RMS get second-pass-background-estimation too
	- but this could be a future endeavor
	- certainly they can get second pass after getting all the above background stuff
	- and then it depends on what they do upfront -- if it is simple to do the chrom-med -> cutoffs -> redo chrom-med + mad, then fine

4. minimally per-pipeline it should not just be "0.25,1.75" unique to each sample... samples should share the same background region - so it could be the intersection "0.25,1.75" in every sample.... 
	- thus amplicon regions will be excluded from all samples

5. underestimate ok
	- not even sure I want to keep the idea -- the hmm was itself the best background estimator
	- same with the other pipelines to different extents...
6. Gap-bin → NaN cross-pipeline. Yes - for things like median and mad estimation, but not for outputs (shouldn't have NaN in a bedGraph, for example)



New flags (added by cycle 15.4a-S3 expanded scope):
--first-pass-background-bounds LO,HI (default 0.25,1.75; legacy 0,inf) — applied to chrom-median-normalized RCN per Q1.
--chrom-trim FRACTION (default 0.001; max 0.01) — excludes first/last X% bins per chromosome.
--second-pass-background-estimation off|on (default off; future default on per BRAIN15.25). Triggers SPEC15.15-style two-pass with refined background.
--second-pass-background-floor FLOOR (default 0).
--second-pass-background-flank-bp BP (default 50000).
--flat-sample-sigma-source {within-stage-mad, chrom-mad, contrib-spread, fixed} (default TBD post-Stage-B.6 calibration).
--flat-sample-detector {inherit, hardthreshold, gaussian_overlap, bayesian_posterior} (default inherit) — F2 cleanup.
--flat-sample-rcn-sigma, --flat-sample-log2-sigma (for fixed mode).


Retired:
--chrom-estimation-mode whole|background — replaced by granular flags above.
--hmm-bkgrd-renorm (SPEC15.15 deliverable 1 candidate) — replaced cross-pipeline by --second-pass-background-estimation. (Worth confirming with you.)



Q1: what you said is what I had in mind: compute naive chrom-median → divide → keep bins with normalized RCN ∈ [0.25, 1.75] → recompute chrom-median + chrom-MAD on the kept set

Q2: Yes, Cross-pipeline second-pass-background-estimation - but it might need to be done different in each 
	-- and we need to ask if it even makes sense for the Growth Track which is what defines amplicons for that pipeline

Q3: Pass-1 background computation = controller pre-pipeline (like SPEC15.2) or per-pipeline applied to a shared mask (option iii)?
	-- not sure yet





=================

You have the right pushback and the right questions.

---------------
Response to give for continuing this conversation and/or need more deliberating.
"""
Can you store all the factoids pertinent to this conversation and these decisions in a temporary file -- maybe `background.tmp` -- so we don't lose this thread. I am sorting through the ideas. You have the right pushback and questions, and I think we need to pause before we jump in to re-evaluate exactly what we want to do and why, and what is the best ways. We have some ideas that overlap, but our new ones may not necessarily be better than our old ones. For example, we are essentially trying to turn the first pass into a naive version of the second pass. It presents us with needing a way to estimate the background regions. There are several ways we can do that... we can do median-norm --> then get the median and mad inside bounds (0.25-1.75) and redo the median norm step. We could do median norm followed by mixture models to try to isolate the gaussian that is centered around 1 to update what we consider the median and variance. That could be done per sample, or just combine all median normalized values from all samples to get a single mixture model, single gaussian centered on 1, a single set of mean and variance, and a single cutoff for RCN values. Then apply the cutoff to each sample. But it strikes me that we would probably want to then take the union of excluded bins across all samples so we have a single "background mask" -- not different "background masks" for each sample. But there will be a lot of background bins that happen to be above the cutoff that are excluded - so we can try to do filtering where there needs to be X above-background bins in a row with a tolerance of Y "background" bins in between each above-background bin.... and only keep merged intervals longer than some length. But then it occurs to you that... you're basically doing what the HMM does only sloppier with heuristics rather than principles. And you realize that another way you could estimate the background bins is to run the HMM like we do, then take a union of all the above background bins determined by the state path. And that brings us back much closer or exactly to our original idea -- first pass as-is, second pass using background regions informed by the HMM. And then you ask yourself -- ok, but do we want a HMM-naive version for the first pass. And now we are at square 1 again. Some of these thoughts are also captured in "background.txt" -- and my notes to your last response start below the line: "MOST RECENT NOTES START BELOW" -- above that is my last response. I want to come back to this conversation.  You can read "background.txt", but don't edit that file.  I want to come back to this conversation.   It is not clear to me how to proceed actually. Maybe you have a clearer picture give my feedback and notes.

Feel free to respond and continue this conversation. We may also want to consider if there are other parts of the SPEC that we can do cycles on while deliberating on this.
"""


---------------------------


I still think we should go for the original idea of the optional second pass in the prior stage. The background will have been estimated better, and it can lead to better "prior" results. Moreover, it will be on the manifest the user input, which might be what they want rather than our posterior manifest. One could potentially do an option for forcing the posterior to use the same manifest, but then it makes the posterior run almost pointless -- the only point would be the better background estimation. There is also the possibility that people don't run --posterior. I think HMM estimates background regions well, and can bootstrap itself to perform a little bit better. RMS can do this as well since it catalogs all the above background regions. Growth is interesting though. The growth track itself is not necessarily good at spotting stable collapsed repeats. Yet the growth track is affected by the initial normalizations, and could also potentially benefit from a better background estimate. The growth pipeline itself might benefit from one of the "pass1 forms" I presented. Either (1) median-norm -> clip RCN within 0.25-1.75 -> re-estimate median and MAD, or (2) median-norm -> learn cutoff(s) from mixture model (instead of assuming 0.25 and 1.75. EThen take the union merge across samples to have all the same bins. One could stop there or go a few steps further --> merge intervals that are within X kb (e.g. 15 kb) --> of the new set of intervals, retain those that are at least Y kb (e.g. 25 kb). The second pass could union merge with called regions. For all pipelines, posterior simple starts with the background regions learned for each pipeline --> until we have a way to unify the background estimates across pipelines. Approaches could be (i) to take the intersect -- only bins that are background in all pipelines, (ii) take the union, bins that were background in any pipeline, (iii) something in between. Anyway -- getting ahead of myself. Just letting the ideas flow. But the main thing is that I still think it is worth developing second pass background estimation within "prior" itself, although I agree that the posterior is the ultimate second pass but it is updating more than the background region estimates.
-------------

NEW NOTES: 8:48 PM - 2026-05-01

Copying notes from conversation - with additional notes, modifications, annotations, etc:
"""
	Candidate amplicon or collapsed repeat regions: structurally above-background; correct exclusion
	Gap bins (0 in all samples): missing data; correct exclusion
	50kb flanks around gaps + amplicons + collapsed repeats: captures the spurious-signal halo; sound
	Optional chromosome-end trim (--chrom-trim 0.1 default 0.1%, max ~1%): captures telomeric/sub-telomeric noise; conservative + flagged
	
	RCN ∈ [0.25, 1.75]
	
	Pass-1 gets structural exclusions (gap-mask, chrom-trim) BEFORE any HMM has run
	
	Pass-1 RCN-bounds heuristic [0.25, 1.75] via --first-pass-background-bounds LO,HI (default 0.25,1.75; legacy 0,inf)
	
	Pass-2 broadens beyond SPEC15.15's complement-of-above-background — also excludes gap, chrom-trim, 50kb flanks of above-background
	
	--second-pass-background-floor FLOOR (default 0)
	
	Cross-pipeline gap-bin → NaN handling (so nanmedian works uniformly)


	--first-pass-background-bounds LO,HI (default 0.25,1.75; legacy 0,inf)
		--first-pass-background-bounds [0.25, 1.75] --> median-norm -> filter --> re-estimate median and mad, redo median norm
		-- MAD may not actually be needed upfront because we don't need the MAD until after the amplicon and collapsed repeats are known at which point background is updated, and new median and MAD learned
			- but can grab it anyway just in case
	--chrom-trim FRACTION (default 0.001; max 0.01) 
	--second-pass-background-estimation off|on (default off; future default on)
	--second-pass-background-floor FLOOR (default 0).
	
	--background-flank-bp BP (default 50000) structural consideration applied to gaps (in both passes), amplicons, and collapsed repeats in second pass
		- first pass: LO|HI, gaps, gap flanks, trim
		- second pass: opt LO | amplicons/collapsed_repeats replace HI, gaps, gap flanks, trim
"""





Pass 1 - all pipelines inherit information: 
--first-pass-background-bounds LO,HI (default 0.25,1.75; legacy 0,inf): controller computes a naive background mask BEFORE any pipeline runs. 
	-> the keywords "legacy" and "none" and "whole" could also be used for "0,inf"  (in addition to "0,inf")
		-> 'legacy' since that is the legacy behavior 
		-> 'none' since it specifies no bounds on RCN (besides the fact that RCN is confined to 0,inf naturally) 
		-> 'whole' as a nod to --chrom-estimation-mode whole
	-> the keywords 'naive' and 'background' could specify "0.25,1.75" (in addition to "0.25,1.75")
		-> these are just nods to our intended defaults and purposes
	-> keywords are not necessary per se -- just thought to mention it as a possibility
--chrom-estimation-mode whole|background
Per-sample chrom-median-normalize 
→ keep bins where RCN ∈ [LO,HI] 
→ exclude bins < LO, bins > HI, gap bins, bins in gap flanks, bins in first/last --chrom-trim of chromosome 
→ intersect across all samples → shared naive background mask.
	- Intersect has to be of "intersect of included bins across samples", or alternatively it might be easier to talk about as "union of excluded bins across samples"
		- It is either the union of "excluded bins" --> union of bins from gaps, flanks, trimming, < LO, >HI
		- Or the intersection of "included bins" --> bins that are "NOT excluded bins" in any sample 
			--> bins where the following is true for all samples: "NOT gap bins, NOT flanks, NOT trims, NOT < LO, NOT > HI" 
			--> since all samples arbitrarily share gap bins, gap flank bins, and trim bins, the real difference is only for the RCNs
--chrom-trim FRACTION (default 0.001)
--background-flank-bp (default 50000) used for gap flanks only in pass 1
Gap bins always excluded (via SPEC15.2's existing pre-pipeline gap-mask)

One question is -- how does each pipeline compute their first set of RCN values? do all do median normalization like step 1 in HMM?
	- if so, then that first step can be done outside of the pipelines before they begin, do all the mask work, each pipeline inherits the same gap mask, background mask, etc 
		- pipelines can start at subsequent step
		- for example, if the controller does everything we would do in HMM step 1, then the HMM pipeline can technically start at step 2, right?
	- if the pipelines do normalization differently in the first step(s), how different? 


Pass 2:
--background-flank-bp (default 50000) used for gap flanks and above background flanks in pass 2

HMM Pass 2:
- "bins > HI" replaced by above-background regions per stage (using the widest representation across stages — most conservative
- "bins < LO" replaced by --second-pass-background-floor FLOOR (default 0)
- still using gap bins, bins in gap flanks, bins in first/last --chrom-trim of chromosome
- flanks also added to above background regions (in addition to gap flanks) to capture the spurious-signal halo
--> This should start out as the complete Pass 1 mask updated by the above background regions
	-> It starts out as the gaps/flanks/trim - i.e. everything but bins > HI and bins < LO -- which are now replaced as described above
	-> is that what you (Opus) also intended? or did you want to do the entire first pass updated with widest representation above-background regions (plus flanks perhaps)?
- Pass-1 outputs archived under pass1/; pass-2 outputs at top-level (per SPEC15.15's locked layout discipline).
- Pass-2 step 1 uses updated background for global median; step 4 uses updated background from chromosome-specifc renorm ; global and chromosome-specific medians and MADs computed on updated background where relevant
- With --posterior: posterior phase inherits prior's HMM-derived background regions (pass-2 mask if enabled, else pass-1 naive mask). 
- Posterior runs its own HMM with chrom-renorm anchored to inherited background.
	- posterior pass 2 probably not necessary, but may yield some additional benefit -- unclear

RMS Pass 2:
- Same structural pattern as HMM, but pass-2 trigger uses RMS-derived above-background regions (RMS calls).
- First pass is inherited from "controller" same as for HMM
- Second pass is updated same as HMM but using RMS above-background calls.
- Pass-1 outputs archived under pass1/; pass-2 outputs at top-level (per SPEC15.15's locked layout discipline).
- Pass-2 - global and chromosome-specific median and MAD computed on updated background where relevant
--posterior: inherits prior's RMS-derived background regions.

Growth Pass 2:
- First pass is inherited from "controller" same as for HMM and RMS
- Unlike RMS and HMM, "bins > HI" is retained in Pass 2 in addition to growth-called amplicons + flanks
	- Growth needs to retain it since it does not include the majority of collapsed repeats in its calls since they are stable across stages (don't grow)
- however for pipeline parity "bins < LO" is still replaced by --second-pass-background-floor FLOOR (default 0) -- i.e. all is included, even non-gap 0 bins by default
- --posterior inherits Growth's 2nd pass (similar to HMM and RMS inheriting their own second passes)


Agreed these are retired:
--chrom-estimation-mode whole|background 
	--> this was about MAD if there was no second pass... 
	--> if second pass is not on... and if this flag does not exist... then this inherits whatever first pass was
	--> this flag concept (w/ different name) could still be useful for single pass if user wants to toggle things differently -- but its overkill
	--> I'd argue the Median and MAD should always try to come from background regions shared by all samples that exclude the amplicons/repeats/gaps/etc
		-> so even if the first pass was legacy mode, we should still do default background mask stuff for the flat sample detection step to inherit
		-> but the same argument could be applied to the first pass in general -> if it is a single pass, the median for median norm should come from a better approximated set of shared background regions
		-> so then the recommendation would just be that a single pass should try to do better than legacy mode for background estimation
		-> and really the recommendaiton will eventually be second-pass=on
	--> so this flag can be retired because its use case would basically be something we would not recommend

--hmm-bkgrd-renorm
	--> agreed that it is replaced by --second-pass-background-estimation


Ideas we covered that I now reject - and replacement ideas where relevant:
- NO to this: First-pass + second-pass chrom-medians + chrom-MADs reported to screen like SPEC15.2 missingness; pass-1 vs pass-2 comparison surfaced
	--> would have to do per sample which would be annoying --> TOO MUCH output to screen
	--> REPLACEMENT IDEA --> like missingness %, report trim % (arbitrary reflecting the input parameter), report % masked for too low, % masked for too high, % mask flanks gaps/low/high, total % masked --> all share the same mask



Whats going on with these?
--flat-sample-sigma-source 
	within-stage-mad --> is this the global MAD computed from background regions (as opposed to chromosome-specific MAD)?
	chrom-mad --> This will eventually be MAD computed from PASS2 background regions if two passes done or PASS1 background regions otherwise, correct?
	contrib-spread --> can you explain this one in simple terms? where does it come from? what is it? how was it computed?
	fixed --> what are the fixed values? what use case is this option good for? Not trying to get rid of it per se, just curious. 
--flat-sample-detector {inherit, hardthreshold, gaussian_overlap, bayesian_posterior} (default inherit) 
	- this inherits from what again? the active stage one?
--flat-sample-rcn-sigma, --flat-sample-log2-sigma (for fixed mode).
	- this is for "--flat-sample-detector fixed"?
	- does the user need to specify both? when is rcn sigma used over log2 sigma? why not use rcn-sigm to compute log2-sigma? <- not pushback; genuine question

Thinking about related stuff
	- A lot of this got kicked off because SAG40 was has a bump at II/9a but is considered flat
	- This made us realize that using the between-sample bin MADs was a systematic error
	- Then we started estimated within-sample chromosome-specific MADs, and got into deeper thinking about background estimation
	- But coming back to the original systematic error --> are we making the same type of error in active stage estimation and regression / dip detection stuff?
	- I can sense that it might be alright in this case -- and that would be why I originally went down that road
		- but I can also sense a need to think it through and make sure
	- Are you following? 

Let me know what you think of everything I said. I believe we are converging on a concrete plan. In general I believe I agreed with most of your plan, and 100% of the spirit of your plan. My notes above may modify some of your plan slightly. Integrate my directions with yours, and update the scoping and planning.




---------------------

NEW NOTES 2026-05-01 11:45 PM
I confirm all of this, and I think we should move forward with updating all the documents that need updating, and getting to the point where we can launch a cycle.

Let's get to the point of launching a cycle, and while it is running, you and I can further discuss whether "ODW active-stage transition test" and "Regression / dip detection" is truly suffering from the same pathology. They are actually different than the pathology we just dealt with. What we just dealt with was basically using per-bin MAD to see if a bin in a sample significantly exceeded its own stage's MAD -- right? something like that. It was using the median and MAD in place to test the same bin (or region) in the samples the median and MAD were computed from - and asking, "is this samples bin >1.25x of the bin's median and MAD with >0.9 prob?" but that was a stupid way to ask if it was flat. The way to ask if it is flat is, "is there a >= 90% chance that this bin is >1.25x than the background median and MAD across this chromosome?" (or whatever prob and cutoff; not necessarily 0.9 and 1.25). That is the right frame. For the active stage question, we are asking if there is a true difference between stages at the same bin or region. So to know that, we do need to know something about the variance from both stages that may explain the difference we see between the same bin across those stages. The question is, what two sets of variances do we need? Is it the chromosome-specific background variance for each stage? Well, no probably not because by definition we know we are not testing a background region. The median and MAD we care about, I think, truly might be the per-bin median and per-bin MAD for each stage -- because we want to compare the same bin across stages like a differential expression question. We are asking if the bin is differentially expressed -- or here, differentially amplified. And specifically we are asking if the bin at stage N+1 is significantly "upregulated" (significantly more amplified) than it is in stage N. But we don't want to just know "significantly up" -- we want to frame it as a Bayesian question: what is the probability that this bin in stage N+1 is at least X-fold more amplified than it is in stage N? and is that probability > probability cutoff? So this is an official pushback. I understand how, after finding this systematic error in flatness detection, we might start wondering if the error was made in other places that the bayesian framework is used -- but for each question we need to think clearly about what the correct median and MADs are. For a within-sample question where we want to know the probability a bin is at least X-fold higher than the flat background, then we want to use the flat background median and MAD. For a between-stage question where we want to know the probability that bin B is at least X-fold higher in sample N+1 than it is in sample N, then we need to know the per-bin median and MAD of bin B in both samples, just like differential expression analyses (think EdgeR and DEseq2). Does that all make sense? I am pretty sure it does. But test it against your vast knowledge. Note that in the active stage test, we actually have two conditions: in the first stage where there is no previous stage (Stage 1 in chrom-median norm mode, stage 2 in ref-stage norm mode), we have to ask the within-stage question where bin B is compared to the chromosome-specific background median and MAD. But then for all subsequent stages, we are comparing bin B between stage N and N-1, and it needs the differential expression form of the question. So this is nuanced, and we need to be careful! Push back as needed.

Let's continue this discussion - but also, let's start launching stuff that is now settled.
