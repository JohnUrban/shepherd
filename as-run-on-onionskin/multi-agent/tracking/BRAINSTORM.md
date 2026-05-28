# BRAINSTORM

Free-form scratchpad for speculative ideas, naming candidates, and unconventional directions
that are not yet ready for ROADMAP or DECISIONS.md.

**What belongs here:** Ideas under consideration but not committed. Naming proposals.
Architectural concepts being explored. Questions to revisit. Anything too speculative or
preliminary for ROADMAP.

**What does NOT belong here:** Committed plans (→ ROADMAP), settled decisions (→ DECISIONS.md),
completed work (→ CHANGELOG.md), session tasks (→ TASK.md).

**Rules for agents:**
- Read freely when exploring future directions or naming questions
- Do NOT promote entries to ROADMAP or code without explicit user direction
- When an idea moves to ROADMAP: update or remove its entry here, note where it landed
- When an idea is abandoned: note the reason briefly, then remove the entry

---

## [2026-04-16] Long-horizon: unified per-sample smoothed RCN intermediates shared between pipelines

**Status:** Long-horizon architecture idea. NOT feasible in current architecture (assessed in
Priority 12.4 feasibility audit, v0.12.18). Document here for future consideration.

### Idea

Growth pipeline currently reads raw per-sample bedGraphs from the manifest and applies its
own normalization (`add_norm_log2`) + smoothing (`smooth_mean`, mean algorithm) independently.
HMM step-5 smooths per-stage aggregate RCN bedGraphs (median algorithm). The idea is to have
a single shared pre-processing intermediate — per-sample smoothed RCN bedGraphs — that both
pipelines could consume instead of re-deriving from scratch.

### Why it is not currently feasible

1. **Shape mismatch (hard blocker):** HMM steps 2-5 operate on per-stage group aggregates
   (step 2 collapses per-sample tracks to group medians; steps 3-5 process only those
   stage-level files). Step-5 outputs are one smoothed RCN bedGraph per non-reference stage.
   Growth `load_tracks_from_manifest()` requires one bedGraph per individual sample.
   The current HMM step-5 does not emit per-sample smoothed outputs.

2. **Masking semantic difference (hard blocker):** HMM step-3 removes zero bins from
   per-stage group median bedGraphs before normalization. Growth applies inline per-sample
   zero-masking on the Y matrix (≥90% threshold). These are not equivalent; bridging them
   would require non-trivial redesign.

3. **Algorithm difference (medium concern):** HMM uses `local_median_smooth_bedgraph()`
   (median), growth uses `smooth_mean()` (mean). Same units and defaults; different behavior
   at outlier bins.

### What would need to change

- HMM step-5 (or a new step-5b) would need to also emit per-sample smoothed intermediates
  in addition to per-stage aggregates (analogous to the planned `indiv_samples/` subdir
  described in KNOWN_ISSUES [ISSUE:2026-04-19:1] for step-14 APS).
- Both pipelines would need to agree on a single smoothing algorithm (median vs mean).
- Growth would need a new entry-point that accepts pre-computed per-sample smoothed
  bedGraphs instead of raw manifest paths.

### When to revisit

If step-14 APS is ever refactored to read per-sample step-5 intermediates (KNOWN_ISSUES
[ISSUE:2026-04-19:1]), that creates the infrastructure needed for this idea. Revisit after
the parallel child pipeline (Priority 11.4) is implemented and the step-5 per-sample
emission question is resolved.

---

## [2026-04-14] HMM summit estimation — use stage of maximum summit RCN, not last stage

**Status:** RESOLVED — addressed by Phase 15's HMM summit refinement work. SPEC15.7
(cycle 15.5a, v0.14.83) shipped cross-pipeline summit refinement with the active-stage
strategy menu; SPEC15.24 (cycle 15.7b, v0.14.92) added HMM per-stage parabola summit
emission for cross-pipeline parity. See IBM-C4 RESOLVED marker in
`multi-agent/tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` for the full Phase-15 trail.
Original entry preserved below per migration-with-provenance convention.

**Original status (pre-RESOLVED):** Open — not yet implemented; relevant to Priority 11.3 (HMM completeness) and
future HMM summit accuracy work.

### Observation

When computing per-stage HMM summit estimates, the stage with the **maximum summit RCN**
(origin bin RCN at peak) gives the best spatial localization — NOT the last stage or an
average across all stages.

Empirically on ds2 chr II: stages 2-4 have summit bins that directly overlap the known
origin region. Stage 4 summit from raw RCN is already shifted laterally. Stage 5 summits
are shifted substantially further. This degradation is caused by **elongation interference**:
in later stages, forks launched from the origin have traveled outward and their cumulative
RCN contribution inflates bins flanking the origin more than the origin bin itself, shifting
the parabola center toward flank signal rather than origin signal.

### Why area_excess is wrong for stage selection (same logic as best_onset_stage)

Area grows monotonically as fork fronts advance — you cannot use area to identify the best
stage. Summit RCN reflects origin-firing intensity and peaks at the stage of maximum
initiation before elongation becomes dominant.

### What this means for HMM step-14 APS and step-15 timing

HMM step-14 reads per-stage summit positions. If those summit positions are derived from
a global parabola fit or from the latest stage, they will be systematically shifted away
from the true origin. A per-stage or best-stage summit strategy is needed.

The `peak_rcn_stage` column (noted in the `dip_rate` section below as "should probably be
added") is the foundation: identify the stage of maximum summit RCN per amplicon, then
use that stage's summit position as the authoritative summit estimate.

### Relationship to sliding-offset / hires refinement

Even within the best stage, the 5 kb bin grid introduces ±2.5 kb localization error.
The sliding-offset approach (already implemented for the growth pipeline in
`onionskin_core/refinement.py` as `sliding_offset_profile` / `refine_origin_sliding_offset`)
gives sub-bin resolution without needing hires data. See the separate BRAINSTORM entry
on HMM sliding-offset refinement below.

### When to resolve

Resolve together with or after Priority 11.4 (parallel child pipeline), since the
per-sample per-stage step-5 outputs produced by 11.4 are the natural input for per-stage
summit estimation. The `peak_rcn_stage` column should be added to `hmm_summit_estimates.tsv`
when this is implemented.

---

## [2026-04-14] HMM summit refinement — sliding-offset sub-bin localization

**Status:** RESOLVED — HMM summit refinement (including sliding-offset port) shipped
via SPEC15.7 cycle 15.5a (v0.14.83) as part of the cross-pipeline summit-refinement
strategy menu. The hires-based `--refine-summits` portion was effectively superseded
by the Phase 15 cross-pipeline summit-refinement architecture which operates on the
existing per-pipeline outputs without requiring a separate post-hoc refinement flag.
See IBM-C4 RESOLVED marker for the full Phase-15 trail.

**Original status (pre-RESOLVED):** Open — sliding-offset implemented for growth pipeline only; HMM not yet
covered; hires-based `--refine-summits` also deferred.

### Background

The growth pipeline uses `sliding_offset_profile` and `refine_origin_sliding_offset` in
`onionskin_core/refinement.py` (implemented v0.5.54–v0.5.56) to achieve sub-bin summit
resolution. The approach: slide a 5 kb window across multiple bin-phase offsets and find
the offset whose window center best aligns with the true summit. This gives ±0 to ±1 bin
accuracy without requiring a hires manifest.

The HMM pipeline does NOT currently use this. HMM summit positions come from per-stage
bedGraph signal without sliding-offset refinement.

### Why HMM needs its own version

HMM step-5 emits per-sample per-stage smoothed bedGraphs. The summit localization step
(used in steps 9/10 and inherited by step-14 APS and step-15 timing) bins at 5 kb.
Without sliding-offset refinement, summit positions are limited to 5 kb granularity
(±2.5 kb error). For chr II origins known to within ~5-10 kb from the literature, this
is a non-trivial fraction of the total uncertainty budget.

### Two refinement approaches available

1. **Sliding-offset (no hires required):** extend `sliding_offset_profile` /
   `refine_origin_sliding_offset` to work on HMM step-5 smoothed bedGraphs. Apply at the
   stage identified as `peak_rcn_stage` (stage of maximum summit RCN). This should be
   the first approach since it requires no additional data.

2. **Hires bedGraph refinement (`--refine-summits`):** the `--refine-summits` planned
   design in the existing BRAINSTORM entry (post-hoc summit refinement) was drafted for
   the growth pipeline and explicitly deferred pending HMM integration. Once HMM summit
   estimation is stabilized, the hires approach can be benchmarked against the
   sliding-offset approach using the chr II ground truth BEDs.

### Prerequisite

Priority 11.4 (parallel child pipeline) produces per-sample step-5 smoothed outputs in
`indiv_samples/` subdirs — this is the natural input for per-sample sliding-offset
refinement before per-sample summit positions are aggregated.

### When to resolve

After Priority 11.4. Cross-reference: `--refine-summits` BRAINSTORM entry; growth
`onionskin_core/refinement.py`; `dip_rate` / `peak_rcn_stage` section in this file.

---

## [2026-04-18] Cross-pipeline origin detection windows — dynamic onset / last-active-stage summit refinement

**Status:** Open. Conceptual next-generation summit/origin refinement idea. Relevant to all three
pipelines, not just rcn-mean-shift.

### Core idea

The biologically relevant "early stages" are not the same for every amplicon. They should be
defined **relative to that amplicon's own re-replication trajectory**, especially the stages where
the summit region is still gaining **height** from active initiation rather than merely becoming
wider because forks continue to elongate.

In that framing, summit localization is really a form of **origin detection**. The goal is
to refine the final summit/origin estimate using only the stage window where initiation is still
informative of **origin location** for that amplicon.

### Why a fixed early-stage set is only a coarse first pass

The current `early-parabola-mean` runtime selector candidate for the rcn-mean-shift pipeline uses
a fixed stage set (`2,3`) for every merged amplicon. That is acceptable as a coarse first-pass
runtime approximation, but it is not the intended end state. It will work best on amplicons that
have most active re-replication activity between stages 1 and 2, and stages 2 and 3.

Different amplicons can:
- turn on at different onset stages,
- stop actively increasing at the summit at different later stages,
- and enter an elongation-dominated regime at different times.

So a single global "early stages" definition will inevitably be too early for some amplicons and
too late for others. "Early stages" needs to be defined separately for each amplicon.

### Preferred long-term design

For each amplicon, infer a dynamic **origin-detection window** bounded by:
- an **onset stage**: first stage where the summit region clearly begins increasing,
- a **last active stage**: last stage where additional summit-region height increase is still
  detected before elongation dominates.

Final summit/origin refinement should then be confined to that per-amplicon stage window.
Stages within that window can be further evaluatated on whether they have strong origin signal
or not based on determining if there was significant height increase compared to width increase.
First pass and default can be to include all stages within the **origin-detection window**.

### Candidate signals for the window

Possible inputs for detecting the onset / last-active-stage window include:
- General idea:
  - explicit detection of when the summit region starts increasing in height and when it stops increasing in height even if the domain keeps widening.
- local summit-window averages rather than whole-amplicon width/area,
  - the entire amplicon width is informative of primarily elongation
  - the <= 50 kb region around the approximated origin position is far more informative of origin activity
  - the <= 5-10kb around the origin (if we knew where it was) is the most informative
  - the closer to the origin, the more origin activity information
  - the farther away from the origin (the broader the zone studied), the more dilute origin signal becomes compared to elongation signal.
- summit-region RCN or area-excess trajectories across stages,
  - for stage 1 (or stage 2 when 1 is the ref-stage), RCN is relative to chromosome median
  - this is to capture the global earliest stage as onset stage when amplification already exists there
  - subsequent stages can define activity based on the height growth relative to the previous stage.
- a "peak_rcn_stage"-style summary where useful,
- per-stage parabola-refined summit heights,
  - onset when the parabola height suggests a parabola vertex higher than background
  - Monitor parabola shape across stages to learn how to best define a last active parabola stage.
- triangle metrics
  - first stage where triangle vs flat BIC score weighs in favor of triangle as onset stage. 
  - Monitor triangularity score increase, and triangle height vs base width across stages to learn how to best define a last active triangle stage.
- for the HMM, onset stage, last active stage, and origin zones are easier to define.
  - HMM onset stage is the first stage the state path has an interval above state 1. 
  - HMM last stage is the last stage the summit state is higher than previous stages. 
  - HMM origin zone can be refined to:
    - the narrowest summit state interval within the origin detection window
    - the central region across all summit state intervals in the origin detection window with the most "coverage".
    - we can assume the origin is within the summit state interval(s), and even that it is in the narrowest one, especially if the narrowest is in one of the earlier stages of the origin detection window.
    - this is why the HMM should not filter out amplicons that are simple short state-2 intervals in individual samples without first looking at all stages or samples to determine if it is actually a very short summit state interval within a true amplicon.
      - How the HMM should handle filtering only when considering all stages or samples is discussed elsewhere.

Width growth alone should **not** be treated as evidence that a stage remains origin-informative.
Re-replication forks continue to travel (elongate) long after the origin stops firing. Different datasets will have different numbers of post-firing stages. For example, DS1 with 9 stages has 3-5 post-firing stages where as DS2 as 0-2.

### Cross-pipeline implication

This should eventually be a shared conceptual behavior across all pipelines:
- growth,
- HMM,
- and rcn-mean-shift.

The exact implementation details may differ by pipeline, but the governing idea should be the
same: do summit/origin refinement inside an amplicon-specific stage window rather than using a
single hard-coded early-stage set for all amplicons.

### Near-term interpretation

Treat the current rcn-mean-shift `early-parabola-mean` work as a **coarse exploratory bridge** to
this broader design, not as the final summit-selection model.

Also, the 2026-04-18 selector follow-up argues against treating a global robust median as the
main solution. An explicit parabola-median aggregate is already a meaningful comparator and was
only middling on the current RMS diagnostic set, while a naive all-stage median performed poorly.
That points back to the same core design lesson: the hard part is selecting the informative stage
window, not just averaging harder across mixed stage-local positions.

## Recap:
The concept of using early stages for the summit is something we plann to implement on all pipelines. The amplicons do not all necessarily have the same "early stages" for origin firing -- "early stages" is a relative term, relative to when a given amplicon starts growing taller. These are the early stages of re-replication for that amplicon. At some point initiation stops, and there is only continued fork elongation which can confound summit detection. Note that "summit detection" in this context is an attempt at "replication origin detection", which is the biological motivation. So having the same set of early stages for all amplicons is an okay coarse first-pass, but not expected to be accurate across all amplicons. What we need to do is to detect the onset stage for each amplicon and to detect the last stage an increase at the summit region is detected. Then confine any final summit refinement to within those stages for a given amplicon. This type of behavior should eventually be part of all pipelines.

---

## [2026-04-11] Future regression coverage ideas after parity stabilization

**Status:** Active future-consideration list. Not committed to ROADMAP yet.

The current regression surface is good enough to resume feature development, but there are
still several targeted tests worth considering later if recent work patterns turn into
recurring failure modes.

**Candidate additions:**

- **Grouped-layout path invariants.** When Priority 7.4 output-layout refactor work starts,
  add focused tests that assert where major pipeline outputs land and that generated README /
  index files describe the same structure. This would catch path drift without needing broad
  end-to-end golden-output snapshots.
- **Dirty-run-directory HMM rerun regression.** Preserve coverage for the v0.9.22 stale-file
  rerun bug by testing that rerunning HMM in an existing output directory does not mix current
  filtered files with stale unfiltered intermediates.
- **Authoritative parity smoke wrapper.** Keep `make puff-parity` as the exact full-genome v2
  gate, but consider a lightweight automated smoke target or CI-style wrapper that fails fast
  if the default parity lane ever stops matching `5kb-v2`.
- **Legacy zero-width defensive unit coverage.** Add a direct unit test around defensive
  handling of zero-width / 1 bp legacy-region edge cases in summit-intersection logic so the
  old PuffStep artifact class does not silently regress while refactors continue.
- **Notebook / generated-doc output contract tests.** Recent sessions exposed drift between
  emitted output layout and generated README / notebook guidance. A narrow contract test could
  assert the presence of key expected files and directories after representative HMM-only and
  grouped-output runs.
- **Core-only exploratory compare fixture.** Not as a release gate, but as a recorded expected-
  drift fixture so future agents can quickly confirm that the exploratory `puffstep-py-core`
  lane is still showing the same methodology-driven class of differences rather than a new bug.

**Non-goal for now:** do not expand this into a large golden-file regime unless repeated
breakage shows that the narrower targeted tests are insufficient.

## [2026-04-11] Post-7.4 code unification phase across all three pipelines

**Status:** Historical note only. The active committed plan now lives in the Phase 10 cleanup spec and the Phase 11 forward-architecture scaffold.

**Trigger:** The grouped-layout hardening work exposed that the HMM pipeline had retained a
private manifest parser even though multistage and the rest of onionskin already used the
shared manifest model. That suggests there may still be other HMM-specific or pipeline-
specific duplicates that should be retired now that `growth`, `per-stage`, and `hmm` are
all first-class pipeline arms.

**Historical takeaway:** The shared manifest/sample model, shared filtering semantics, and
duplicate-code audit all remained valid directions. The specific shared emitted-output branch
that emerged from this brainstorm was later abandoned because it conflicted with pipeline-local
ownership.

**What still matters from this note:**

- keep shared filtering semantics as controller-owned contract work
- keep duplicate-code reduction as shared-module extraction work
- keep future synthesis as a later layer above pipeline-local outputs

**What was abandoned on 2026-04-13:**

- treating deprecated shared-output structures as part of the target architecture
- treating pipeline-derived outputs as candidates for canonical shared ownership above pipelines
- using centralized intermediate files as the path to cross-pipeline convergence

## [2026-04-11] Abandoned shared-output architecture idea

**Status:** Abandoned on 2026-04-13.

**Abandoned idea:** Treating deprecated shared-output structures as a canonical owner layer for
pipeline-derived analysis outputs.

**Reason for abandonment:** This conflicted with the pipeline-local ownership contract by
creating a fifth architectural layer above pipelines and by reintroducing growth-adjacent output
surfaces as de facto common owners.

**Do not revive:**

- canonical-owner plus pipeline-view patterns for pipeline-derived outputs
- centralized intermediate-file ownership as a substitute for shared code reuse
- any future design argument that treats deprecated shared-output structures as architectural
  foundation rather than as migration-removal targets

**Replacement direction:** pipeline-local outputs, shared code/modules, analysis-neutral
controller metadata, and future explicit synthesis.

## [2026-04-11] Shared filtering semantics across pipelines

**Status:** Promoted to ROADMAP Phase 10 and formalized in the Phase 10 spec now archived at `multi-agent/plans/archived/20260413-PHASE10_SPEC.md`.

**Observation:** The current `03-hmm/00-filtered-input/` naming makes the HMM pipeline's input
filtering visible, but it also raises a correctness question: if a run requests chromosome or
sequence filtering, all pipelines in the run should usually start from the same filtered input
universe rather than having HMM filter independently while other pipelines still see the full
dataset.

**Design principle:** In a multi-pipeline run, input-selection flags should define a shared run
contract unless there is a clearly documented, biologically justified special case.

**Flags explicitly called out for harmonization:**

- `--chromosomes`
- `--min-seq-length`
- `--min-bin-count-per-seq`

**Implication:** Filtering probably belongs conceptually above the three pipelines, not as an
HMM-only preprocessing quirk. The user-facing meaning of those flags should be the same whether
the run uses `growth`, `per-stage`, `hmm`, or any combination.

**Open question:** Are there any legitimate cases where one pipeline should bypass a requested
global filter while another respects it? Default assumption should be no unless concrete cases
appear.

**Clarification added 2026-04-11:** Shared filtering semantics are not the same thing as shared
normalization semantics. The HMM should not be treated as if ratio normalization is inherently
required. The design direction now is:

- ratio-backed HMM remains valid for reference-backed multi-stage/two-stage inputs
- single-file and true single-stage inputs without a denominator should support a no-ratio HMM path
- `--norm-mode` must eventually be clarified as a cross-pipeline contract because it is not
  currently shared in meaning across multistage, per-stage, and HMM lanes

**Further clarification added 2026-04-11:** the controller contract cannot be driven only by the
number of stages. It should be described from a fuller input-capability surface:

- number of distinct stages
- replicate count per stage
- whether a denominator/reference stage exists explicitly

This matters because some analyses are mathematically runnable with two stages, but their
stability and interpretability still depend on the replicate structure within those stages.

**No-ratio HMM motivation:** ratio normalization can suppress real early-stage amplicons when the
denominator already contains weak amplification signal. The no-ratio branch avoids that at the
cost of more collapsed-repeat exposure, which can then be mitigated post hoc via the existing
triangle-vs-flat / non-growth filtering machinery.

**Implementation note added 2026-04-12:** the no-ratio HMM branch now runs a dedicated step 3
instead of skipping step 3 entirely. Current rule: remove a bin only when it is zero-valued
across all step-2 group-median bedGraphs, using the existing gap-mask logic conservatively as a
shared assembly-gap detector rather than as a file-specific zero-bin filter.

**Implementation note added 2026-04-12:** stage numbers are now enforced as onionskin's canonical
`1..N` developmental-order contract. Synthetic manifests from convenience inputs now emit visibly
at the grouping level, and `--ref-stage` is now treated as an advanced override rather than a
normal stage-definition mechanism.

**Implementation note added 2026-04-12:** the first pipeline-specific normalization split is now
live at the CLI level via `--growth-norm-mode`, `--per-stage-norm-mode`, and `--hmm-norm-mode`.
These currently inherit the shared `--norm-mode` by default, but the important architecture shift
is that HMM preprocessing and controller semantics no longer have to pretend one normalization
choice automatically describes every pipeline equally well.

**Implementation note added 2026-04-12:** the top-level controller now treats per-stage as its own
pipeline arm even when growth is also requested. In `onionskin.py`, growth+per-stage runs now use
the multistage engine with `--skip-per-stage`, then launch standalone per-stage separately. This is
the right direction for the continuum model and should eventually make it easier to keep per-stage
defaults, outputs, and downstream contracts independent from growth.

**Implementation note added 2026-04-12:** the pipeline-specific normalization flags no longer just
shadow the global flag. Growth now defaults to `chrom-median`, while HMM and per-stage now default
to `ref-stage` on reference-backed 2+ stage inputs and otherwise fall back to the shared
`--norm-mode`. The explicit `inherit` option survives as a compatibility escape hatch.

**Implementation note added 2026-04-12:** the first dependency-graph cleanup step is now live.
`multistage_engine.py` no longer defines `--skip-per-stage` and no longer launches per-stage
internally. Mixed runs are still sequenced conservatively in `onionskin.py`, but the ownership edge
is now in the controller rather than hidden inside the growth engine.

## [2026-04-11] `02-per-stage-mean-shift/` completeness phase

**Status:** Promoted to ROADMAP Phase 10 and formalized in the Phase 10 spec now archived at `multi-agent/plans/archived/20260413-PHASE10_SPEC.md`.

**Observation:** `02-per-stage-mean-shift/` currently exists as a pipeline arm, but it is not yet
as analytically complete or self-sufficient as the current growth lane or HMM lane.

**Desired end state:** `per-stage` should be able to stand on its own when run alone and should
produce its own downstream analyses derived only from its own calls and signals, including items
analogous to timing, APS, clustering, and other later-stage analysis products where appropriate.

**Planning requirement:** This likely deserves its own multi-step phase after the shared-
architecture / file-contract work is settled, because those later analyses should depend only on
artifacts generated within `02-per-stage-mean-shift/` rather than borrowing from other pipelines.

**Questions to answer in a later design session:**

- Which downstream products should exist for `per-stage` parity with the other pipelines?
- Which analyses can be reused from shared modules immediately, and which require new
  per-stage-specific transforms?
- What are the minimum outputs needed so `--pipelines per-stage` feels complete and not just a
  helper lane?

## [2026-04-11] Pipeline execution contracts and `--pipelines` completeness

**Status:** Promoted to ROADMAP Phase 10 and formalized in the Phase 10 spec now archived at `multi-agent/plans/archived/20260413-PHASE10_SPEC.md`.

**Expectation:** All supported pipeline combinations should work cleanly and predictably.

**Required cases to verify / enforce:**

- `--pipelines all` should mean all three pipelines are functional together.
- Each individual pipeline should also run cleanly on its own.
- Naming/alias expectations should be clarified, especially whether the user-facing term should
  remain `growth` or whether `multistage` should become an accepted alias if that is more natural.

**Representative cases to treat as first-class contracts:**

- `--pipelines growth`
- `--pipelines per-stage`
- `--pipelines hmm`
- `--pipelines all`
- mixed subsets such as `growth,hmm` and `growth,per-stage`

**Additional composition question raised 2026-04-11:** single-mode inputs do not map cleanly
onto the current `growth` label. In practice, single-file and single-stage contexts primarily
want the `02-per-stage-mean-shift/` lane and the HMM lane, with the HMM itself needing a
different preprocessing contract depending on whether a denominator stage exists.

**Current planning direction:** a two-stage manifest can remain on the single-stage/
reference-backed path when one stage is explicitly designated as `--ref-stage` or when
`--norm-mode ref-stage` makes that denominator relationship explicit. This needs a formal
controller contract before Priority 10.5 is considered complete.

**Updated design direction late 2026-04-11:** the above is still too tied to the older
single-stage versus multistage vocabulary. The cleaner contract is:

- `multistage` should describe the input/data situation when multiple stages are present, not just
  the growth-model lane
- HMM and per-stage are also multistage analyses when they operate across multiple stages and
  extract stage-structured evidence
- the pipeline-specific label should be `growth-model`, not `multistage`
- grouped pipeline ordering may need to move away from the current legacy growth-first layout
  toward `01-hmm/`, `02-per-stage-mean-shift/`, and `03-growth-model/`
- HMM-first ordering has a strong historical argument because it matches the long-lived
  pufferfish / PuffStep mental model and places the most established lane at the left edge of the
  continuum
- engine/module names should eventually stop encoding the older single-versus-multistage split;
  the working names are `per_stage_mean_shift_engine.py` and `growth_model_engine.py`

**Why this matters:** Shared architecture and shared filtering are only valuable if the actual
pipeline composition semantics are reliable. This is partly code correctness and partly CLI / UX
contract design.

---

## [2026-04-09] HMM right-side asymmetry bias — artifact vs. biology

**Status:** Active open question. Observed in first real-data outputs at v0.9.07.
See full diagnosis in `multi-agent/plans/archived/20260411-PHASE9_SPEC.md` → "Open questions and known artifacts."

**Observation:** `right_travel_bp > left_travel_bp` systematically across amplicons.
The HMM consistently places the right domain boundary slightly further from the origin
than the left boundary.

**Leading hypothesis:** Directional Viterbi artifact. Left-to-right processing means the
decoder has "inertia" (accumulated high-state history) when it reaches the right drop-off,
causing it to linger in the high state slightly longer than on the left entry. This is an
inherent property of the Viterbi algorithm — reversing direction would flip the bias, not
remove it. A bidirectional fix is non-trivial.

**Alternative hypothesis:** Real biology. Replication forks don't necessarily travel equal
distances on each side. Genomic orientation, local sequence context (GC, accessibility),
or strand-specific fork speed could produce genuine asymmetry. Leading vs. lagging strand
differences in replication timing have been reported at some origins.

**Distinguishing test:** Phase 10 cross-pipeline comparison. If the per-stage mean-shift
and multistage-gradient pipelines show the same asymmetry pattern for the same loci, the
asymmetry is likely biological. If it is HMM-specific, it is likely algorithmic.

**Short-term mitigation ideas (not committed):**
- Add dataset-wide mean asymmetry annotation to fork travel plots and QC notebook warning
- Report `asymmetry_fraction` statistics in the step-12 `fork_travel_metrics.tsv` summary
- For Phase 10 synthesis: use the other pipeline boundary estimates as a correction reference

---

## [2026-04-09] HMM fork travel and analysis plots — review COMPLETE

**Status:** Review completed 2026-04-09. Full findings in `multi-agent/plans/archived/20260411-PHASE9_SPEC.md`
→ "Plot review — findings." Summary of decisions:

- **Fork asymmetry scatter:** Keep; swap axes (right→X, left→Y); add per-stage grid variant.
- **Level emergence heatmap:** Keep; add sequence-length/bin-count filters; add chr-level heatmaps.
- **Fork travel trajectory:** Keep as-is; add new "fork age tracking" plot (see below).
- **Nested domain diagrams:** Keep as-is, user liked them.

---

## [2026-04-09] Fork age tracking — new visualization concept

**Status:** RESOLVED — promoted to Phase 15 BRAIN15.32 and shipped earlier in Phase 15
(per `multi-agent/plans/PHASE15_BRAINSTORM.md` BRAIN15.32 RESOLVED marker; cycle 15.10a
SPEC15.20 d4 staleness sweep). The fork age tracking concept landed as part of HMM
fork-trajectory plotting + clustering work; this entry kept here as historical
provenance per the migration-with-provenance convention.

**Original status (pre-RESOLVED):** Design agreed, implementation deferred. Do NOT assign to Copilot without a
more detailed spec (write that spec in a design session with the user first).

**Concept:** The existing fork travel trajectory plots track nesting levels by their rank
within each stage (L0 = newest/innermost in that stage, L1 = second-newest, etc.). This
means the same level label refers to different biological fork sets across stages, which
is not what was originally intended.

The intended visualization tracks the SAME biological fork set across stages, identified
by their firing order (age):
- **Fork age 0 (oldest):** the first set of re-replication forks ever fired, always at
  the outermost boundaries of the amplicon in every stage.
- **Fork age 1 (second oldest):** the next set inward, visible from the second doubling
  onward.
- **Fork age k (newest):** always the innermost/summit-state interval in the current stage.

**Key mapping:** In a stage with N nesting levels indexed 0 (innermost) to N-1 (outermost),
fork age `a` corresponds to level index `N-1-a`. This mapping is stable and biologically
meaningful because the outermost level always represents the oldest forks.

**Rendering rule:** Only fork ages present in all (or most) stages can be plotted as
continuous trajectories. Younger fork ages (small values of age, e.g., age 0 = newest)
may not be present in early stages. Render missing stages as gaps in the line, not errors.

**Biological interpretation:**
- A fork age that is visible only in late stages indicates the corresponding re-replication
  round happened late developmentally.
- A fork age whose left/right positions are stable across stages means that fork set stopped
  traveling (or traveled very little after its initial firing).
- A fork age whose positions grow monotonically across stages means those forks were still
  traveling during the observation window.

**Implementation notes:**
- This requires a post-processing step on the step-12 data (or a new sub-step) to re-index
  levels by age rather than by rank within stage.
- The underlying data is already in step-12 outputs; this is a plotting/indexing transform,
  not a new metric.
- File naming: `{trajectory_id}.fork_age_trajectory.png` to distinguish from the existing
  `{trajectory_id}.fork_travel_trajectory.png`.

---

## [2026-04-09] HMM sequence filtering — CLI flags

**Status:** RESOLVED — all three flags (`--chromosomes` HMM passthrough, `--min-seq-length`,
`--min-bin-count-per-seq`) are implemented in `onionskin_core/engines/hmm_engine.py` per
Phase 15 BRAIN15.30 RESOLVED marker (cycle 15.10a SPEC15.20 d4 staleness sweep). This
entry kept here as historical provenance per the migration-with-provenance convention.

**Original status (pre-RESOLVED):** Design agreed, implementation deferred. Should be added before or alongside
9.5 work, since the unfiltered heatmap is cluttered with small contigs.

**Problem:** The HMM currently runs on all input sequences, including hundreds of small
associated contigs enriched for unplaced repeats. These dominate the level emergence
heatmap and produce uninteresting signal. The `--chromosomes` flag is also currently
not passed through to the HMM engine.

**Proposed flags** (to be added to `hmm_engine.py` / `run_hmm()`):
- `--chromosomes` passthrough (bug fix, not a new feature — the flag already exists
  in the CLI but is not honored by the HMM step)
- `--min-seq-length INT` — skip sequences shorter than this (default: 50000 bp).
  Excludes most small associated contigs without requiring the user to know scaffold names.
- `--min-bin-count-per-seq INT` — skip sequences with fewer bins than this (default: 10).
  Redundant with `--min-seq-length` at standard bin sizes, but gives users an alternative
  intuitive control. Both can be set independently.

**Chromosome-level heatmaps:**
- When `--chromosomes` is specified: emit one per-chromosome heatmap for each chromosome.
- When not specified: emit per-sequence heatmaps for sequences meeting: length > 1 Mb AND
  >= 2 amplicons detected. This provides a clean chromosome-scale view without requiring
  the user to enumerate chromosomes explicitly.
- Global heatmap still emitted as a diagnostic overview (with filter applied).

**Implementation location:** Filter sequences at the top of `run_hmm()` before any step
executes, so all downstream steps automatically operate on the filtered set.

---

## [2026-04-07] Multi-engine integration and HMM meta-analysis vision

**Status:** Active design — code unification (ROADMAP 6.5) is the prerequisite.
Updated 2026-04-07 with three-pipeline architecture decisions.

### Three-pipeline architecture (decided 2026-04-07)

After full integration, the onionskin output tree will be structured as follows:

```
<out_dir>/
  01-prior/
   01-hmm/                    ← chosen continuum-left anchor; strongest historical/default lane
   02-per-stage-mean-shift/   ← stage-structured but simpler/nonparametric lane
   03-growth-model/           ← chosen rename for current growth-model lane
    04-unified-results/        ← meta-analysis across all three pipelines
  02-posterior/
    (mirrors 01-prior structure with posterior stage assignments)
```

Three independent approaches to amplicon calling, summit estimation, and staging:
1. **HMM state-path pipeline** (`01-hmm/`) — historically primary lane and still the
  most established left-edge/default analysis family
2. **Per-stage mean-shift** (`02-per-stage-mean-shift/`) — each stage's aggregate track
  analyzed independently using stage1_mean_shift + stage2_score + shape_filter
3. **Growth-model** (`03-growth-model/`) — joint
  cross-stage model-fitting lane formerly labeled `multistage-growth`

### Implementation order
- Historical outcome: the grouped-layout refactor has now landed with the legacy growth-first
  ordering, so `01-prior/` and `02-posterior/` both currently use grouped pipeline arms with
  `01-multistage-growth/`, `02-per-stage-mean-shift/`, and grouped `03-hmm/`.
- Remaining future work from this architecture thread is no longer the basic directory move;
  it is deeper contract work: shared file ownership, shared filtering semantics, pipeline
  completeness, eventual `04-unified-results/`, and now an explicit reconsideration of pipeline
  naming/order under the newer continuum model.

### Flag architecture (decided 2026-04-07)
Historical design note: this predates the current `--pipelines` implementation. The repo now
uses `--pipelines` directly, but there is still an open contract question about whether
`--pipelines all` should expand to all three pipelines or remain `growth + per-stage` with HMM
explicitly opt-in. See the 2026-04-11 pipeline-contract brainstorming section above.

### Per-stage mean-shift outputs (Phase 6.5.E spec)
Each stage emits:
- `stageN_calls.tsv` — passing calls (stage1 + stage2 + shape filter)
- `stageN_calls.bed` — BED for IGV
- `stageN_summits.bed` — summit positions for IGV
- `stageN_rejected_calls.tsv` — gap/quality rejects
- `stageN_putative_collapsed_repeats.tsv` — shape filter rejects
- `stageN_putative_collapsed_repeats.bed` — BED for IGV

Unified (cross-stage dedup, non-redundant union with stage membership columns):
- `unified_stage_calls.tsv` / `.bed`
- `unified_rejected_calls.tsv`
- `unified_collapsed_repeats.tsv` / `.bed`

Future: cross-stage presence/absence, timing, oscillation filtering (Phase 8+).
Current approach: simple deduplication to non-redundant union with `stages_present` column.

### HMM inputs (decided 2026-04-07)
The HMM (PuffStep integration) takes per-stage median RCN profiles (median-smoothed),
NOT the mean-shift or multistage call outputs. It generates its own per-stage calls,
per-stage summit state intervals, and per-stage summit bin estimates independently.
The three pipelines converge only in `04-unified-results/`.

### HiRes in Phase 6.5.E (decided 2026-04-07)
Phase E uses base resolution only for detection. Hires summit refinement (analogous
to what single_engine does in step 3) is noted as Phase E.2 enhancement — not blocking.

### Meta-analysis layer (future, Phase 8+)
- Compare summit estimates from all three approaches per locus; flag disagreements
- Multi-approach agreement as confidence for timing, fork elongation, APS
- Report disagreement explicitly — disagreement is scientifically informative
- Per-stage call integration: consensus or weighted averaging

### Code unification plan
See `multi-agent/plans/CODE-UNIFICATION-REFACTOR.md` for the detailed phased plan.
The key insight: unifying into shared modules BEFORE HMM integration means we
integrate HMM into one architecture rather than three. This is the primary
motivation for prioritizing Phase 6.5 before Phase 7.

---

## [2026-03-31] PuffStep fork / integration project name

**Status:** Open — naming undecided

The planned HMM integration (ROADMAP Phase 7) will start with PuffStep as-is and then be
developed further to blend more deeply with onionskin. Because it will diverge significantly
from the original PuffStep, it may eventually warrant its own name.

**Candidates discussed:**
- `onionlayers` — evokes the onionskin layering metaphor; implies the HMM analysis "layers"
  structure onto the onionskin DNA signal. Not yet committed.
- PuffStep (keep current name) — simpler if the fork stays recognizable as PuffStep-derived.
- Other ideas: TBD — revisit when Phase 7 implementation begins.

**When to resolve:** When Phase 7 integration work begins. At that point, update:
- ROADMAP Phase 7 header
- `multi-agent/MULTI-AGENT-INFRASTRUCTURE.md`
- `multi-agent/audits/deep_understanding_audit_prompt.md` (Section 7)
- Any other references to "PuffStep / HMM integration"

## [2026-04-01] APS locus diagnostics — design discussion and open questions

**Status:** Open — placeholder implementations in place; definitions need future refinement

This entry documents the design discussion that took place in v0.5.44 around the three
locus diagnostic columns added to `onionskin_aps_locus_contributions.tsv` and the
biological intent behind them.  These columns feed into ROADMAP Priority 5.2 (locus
reliability weighting).  The implementations are placeholders with known limitations.

### What we are trying to accomplish

APS = `sum(area_excess across loci)`.  Not all detected loci are equally trustworthy.
`locus_weight` is a **credibility discount** `w ∈ [0,1]` — NOT a size/amplitude
adjustment (those are already encoded in `area_excess = sum((RCN-1) × bin_width)`).
A large, strong amplicon already dominates APS by its area; `locus_weight` only adjusts
for *confidence that the detection is real and consistent*.

This is conceptually a **posterior APS**: after seeing all stage data, assess each
locus's credibility, then fold that back into the APS sum.  Distinct from posterior
ordering/grouping (which reorders samples) — this operates at the locus level.

### best_onset_stage

**Biological intent:** when did the replication origin first fire?  This is the first
stage where the amplicon shows meaningful signal — NOT the stage of biggest area jump.

**Why area_excess is wrong for onset detection:**
Area_excess = integral of (RCN-1) × bin_width across the locus.  This grows with
fork travel distance, which continues to increase every stage as forks move further
from the origin.  The absolute area jump is therefore almost always largest at the
last stage, regardless of when the origin actually started firing.  Using area_excess
to find onset would misassign `best_onset_stage = last_stage` for nearly every locus.

**What the right signal is:** summit RCN (RCN at the replication origin itself).
Summit RCN reflects origin-firing directly — it rises when new initiation rounds begin
and plateaus or declines when they stop.  Fork elongation adds area but does not
substantially increase summit RCN.

**Current implementation:** pulls `onset_stage` from the timing module's output TSV
(which uses RCN-threshold + shape-evidence logic on summit RCN).  Falls back to first
stage where mean `peak_rcn >= 2.0`.  This is correct in direction but has a known
limitation: if stage 1 already shows amplification (peak_rcn >= 2 and triangle shape),
onset is stage 1 — we do know that amplification was absent at some point before stage 1:
we just did not get an early enough sample to detect it.
Single-stage mode detects amplification relative to background, not relative to a
pre-amplification state.  Multistage mode adds temporal context but to detect if
stage 1 is the earliest onset_stage among the samples, we need to test stage 1 similar to
single-stage mode: relative to background (e.g. relative to chrom-median).

**Open question:** should `best_onset_stage` remain a simple pointer to the timing
module's `onset_stage`, or should it be independently derived here from the
cross-stage `contrib_df`?  Using the timing module's value is preferred for consistency
but creates a dependency on the timing TSV being present.

### post_support

**Biological intent (original, unclear):** some notion of "degree of belief" that
the locus is a real amplicon after seeing all stage data — a posterior credibility score.

**Why the current implementation is a placeholder:**
Currently = mean `peak_rcn` across ALL stages.  This is a rough proxy at best.
It does not condition on onset (so early-absent loci are penalized), and it does not
distinguish between "consistently detected" vs "barely detected everywhere."

**Better future definition candidates:**
- Fraction of post-onset stages where `peak_rcn >= threshold` (e.g., 1.5 or 2.0)
- Mean `peak_rcn` in post-onset stages only (conditions on onset correctly)
- Coefficient of variation of `peak_rcn` in post-onset stages (low CV = consistent)
The name `post_support` is retained for schema stability; interpretation should not
be relied upon until the definition is settled.

### dip_rate

**Biological intent:** flag loci whose summit RCN is inconsistent after onset — either
flickering (suggesting a spurious or marginal detection) or declining in late stages
(suggesting nuclease degradation of the DNA in late salivary gland stages).

**Why this matters biologically:**
In DS1 (9 stages), stages after ~5 are predominantly elongation-phase stages — new
initiation rounds have largely stopped and existing forks are traveling outward.  In
the latest stages, salivary gland cells may begin to be broken down; nucleases
nonspecifically digest the genome.  Because amplified regions have more copies, they
initially lose more absolute DNA, but the *ratio* (RCN) can decline — summit RCN
may drop in the latest stages even though the locus was genuinely amplified earlier.

This means the stage of **maximum summit RCN** is NOT necessarily the last stage.
We want to identify that peak stage per locus, and `dip_rate` is a step toward that.

**Why summit RCN, not area_excess:**
Area grows monotonically as forks travel further from the origin.  Even when summit
RCN is declining (nuclease degradation), the total area may still be increasing
because the fork front is still advancing.  A locus would never show an area "dip"
even when biologically declining.  Summit RCN is the correct signal.

**Current implementation:** fraction of stages where mean `peak_rcn < 0.5 × max(peak_rcn)`.
This is reasonable in direction but the threshold (0.5) is arbitrary and not calibrated.
The stage of maximum mean `peak_rcn` is implicitly defined by this, but not explicitly
reported — it should probably be added as its own column (`peak_rcn_stage`).

**Open question:** should `dip_rate` be computed only over post-onset stages (ignoring
pre-onset low values which are expected and not a "dip")?  Currently it uses all stages,
which would count pre-onset stages as dips even for late-onset loci.

### Prior grouping belief strength (related idea)

The user's prior grouping (experimental staging, morphology, etc.) carries real biological
information.  APS-derived ordering should not completely override it.  A future
`--prior-weight` or `--aps-prior-strength` parameter could blend prior staging with
APS-derived ordering.  The user has explicitly stated that for their datasets, the prior
grouping should be treated with merit — especially given that late-stage degradation
could distort APS in ways that move samples relative to each other artifactually.

**Status update (v0.5.46):** The absolute-threshold formula implemented in v0.5.45
was REVERTED.  See "Why the absolute-threshold formula failed" below.
`locus_weight` is back to 1.0 placeholder.  The correct next step is implementing
per-stage variance outputs (MAD/stdev per bin per stage), then a probabilistic
oscillation test.  See "Next steps" below.

### Why the absolute-threshold formula failed (v0.5.45 → v0.5.46 revert)

The formula `locus_weight = post_support × (1 − dip_rate)` with absolute thresholds
(`post_support` = fraction of post-onset stages with `peak_rcn >= 2.0`;
`dip_rate` = fraction with `peak_rcn < 1.5`) was tested against DS1 chr II and
found to produce wrong results on all four ambiguous loci:

- **II:45.8-46.6** (weight=0.00): genuine low-signal amplicon confirmed by IGV and HMM
  detection in DS2.  RCN never reaches 2.0 because only a subset of cells amplify it,
  or it amplifies to a low level.  The domain is broad (865 kb), triangle-shaped,
  and shows fork-front dynamics across stages.  Discounting it was wrong.
- **II:54.4-56.1** (weight=0.61): real.  Should be 1.0.
- **II:21.4-23.5** (weight=0.83): real, first peak of a known twin-peak training pair.  Should be 1.0.
- **II:40.7-42.5** (weight=1.00 but high variance): real.  Weight was accidentally
  correct for the wrong reason.

**Root cause:** With only 2 replicates per stage, per-stage mean `peak_rcn` has high
sampling variance.  Stage-to-stage oscillations in the metrics are dominated by
within-stage replicate noise, not biological signal.  Any formula using point estimates
of per-stage means to detect "dips" will misfire on real data at typical sample sizes.

**Secondary root cause:** Absolute thresholds (2.0, 1.5) conflate signal strength with
credibility.  A consistently weak locus (peak_rcn = 1.4 every stage) should have
high credibility — it's reliably there, just at low level.  The formula penalizes
signal strength, which is already captured by `area_excess`.  This double-penalizes
weak-but-real amplicons.

**Important design principle (user-stated):**
One of the founding goals of onionskin is to detect late, small, hard-to-detect
amplicons — broad triangular domains that are highly blended with background, possibly
amplified in only a subset of cells.  The locus credibility weight must NOT penalize
low absolute RCN.  It should only penalize inconsistency, and only when that
inconsistency is statistically supported given the within-stage variance.

### Prior groupings and posterior groupings — both must work

The probabilistic oscillation test must work on **prior groupings**, not just posterior.
Users may reject posterior groupings entirely and work only with prior staging.  Both
analyses must be robust.  Posterior groupings help by increasing replicates per group
and reducing variance, but they are not a prerequisite for the prior analysis.

### Growth track — useful but partially circular for credibility

The growth track (robust-z of stage-wise evidence: linear, isotonic, step, or unimodal
fit to RCN across stages per bin) is the primary detection signal — amplicons are
*found* because their growth track z-score is high.  Using the growth track as a
credibility signal is therefore partially circular (a found amplicon by definition had
a high growth track somewhere).

However, the **fork-front shape** in the growth track (two symmetric peaks flanking
the origin, broadening across stages) is structurally more specific than the detection
z-score and carries genuinely independent credibility information.  Shape score
(`dBIC_flat_vs_tri`) captures something similar in RCN space.  Whether to use
growth track shape for credibility is worth exploring, but needs care to avoid
double-counting with detection evidence.  Not pursued yet.

### Per-stage variance — the prerequisite for a correct formula

The correct formula requires per-stage variance.  Currently onionskin writes:
- per-stage median tracks (stage medians per bin) ✓
- per-sample tracks ✓

**Per-stage MAD tracks: ✓ DONE (v0.5.47)** — `stageN.MAD_RCN.bedGraph` and
`stageN.MAD_log2RCN.bedGraph` now written alongside existing median tracks.
N=1 → MAD = 0.0 (safe).  Verified sensible on DS1 chr II.

### Oscillation annotation: DONE (v0.5.47–v0.5.49)

1. ~~**Add per-stage MAD (or stdev) tracks** to the output.~~ **DONE (v0.5.47)**
2. ~~**Probabilistic oscillation test**~~ **DONE (v0.5.48)** — Gaussian overlap
   P(RCN_s > RCN_{s+1}) = Φ(z), σ = MAD × 1.4826, multi-bin summit window.
3. ~~**Oscillation annotation + global degradation report**~~ **DONE (v0.5.49)** —
   `aps_stage_regression_report.tsv`, `is_oscillator`, `max_rcn_stage`,
   `regression_stage`, `oscillating_transitions` columns in locus_contributions.tsv.

**Outcome:** Priority 5.2 closed as annotation-only.  All four calibration amplicons
showed statistically confirmed oscillations — the probabilistic test correctly identifies
real biology (not noise), but no discriminator between "spurious oscillator" and "real
oscillating amplicon" was found with available data.  `locus_weight` = 1.0.

### Future: per-locus APS reliability weighting (off ROADMAP as of v0.5.49)

This goal is preserved here because the machinery exists and the need may resurface.

**The problem:** APS is a sum of `area_excess` across loci.  If some detected loci are
spurious or poorly resolved, they add noise to every sample's APS.  A credibility weight
`w_ℓ ∈ [0,1]` per locus could reduce their influence without removing them.

**Why deferred:** At this stage of the project, we do not have a reliable discriminator
between a spurious detection and a real amplicon with genuine stage-to-stage variation.
The oscillation test correctly flags real biology; applying it as a weight would penalize
real amplicons.  The number of spurious detections in practice appears to be small.

**Key constraints for any future formula:**
- Must NOT penalize low absolute RCN (weak, broad, low-level amplicons are a founding goal)
- Must work on prior groupings independently (users may reject posterior groupings)
- Must be variance-aware (N=2 per stage → point estimates are noisy)
- Calibration: the four DS1 chr II amplicons below must all receive weight ≈ 1.0

**Calibration amplicons (DS1 chr II, manifest.dataset1.5000bp.tsv):**
- `chrII:45755000-46620000` (II:45.8-46.6, 865 kb) — genuine weak amplicon,
  confirmed by IGV + DS2 HMM; fork fronts visible; must not be penalized for low RCN
- `chrII:54380000-55152500` (II:54.4-56.1, 772 kb) — real
- `chrII:21440000-22737500` (II:21.4-23.5, 1297 kb) — real, twin-peak training pair
- `chrII:40660000-42530000` (II:40.7-42.5, 1870 kb) — real, genuinely high variance

**Possible future approaches:**
- Multi-dataset consensus: a locus detected in multiple independent datasets with
  consistent oscillation pattern is more credibly a "variable" locus; one detected
  only in a single dataset with erratic behavior is more suspect
- Shape-based discriminator: use fork-front gradient symmetry or shape score as a
  more independent credibility signal (growth-track z-score is partially circular)
- More replicates per stage: with N≥4, per-stage variance estimates are stable enough
  to evaluate oscillation without grossly inflating P values

---

## [2026-04-07] Keep/exclude recommendation enhancements (from Priority 4.9)

**Status:** PARTIALLY-RESOLVED — Phase 15 cross-pipeline reliability scoring + flat-sample
detection (SPEC15.6 cycle 15.4a + supplemental cycles, v0.14.80–v0.14.89) absorbed the
shape-score-proximity + timing-flag concepts. `--known-reliable-amplicons` synonym
(`--keep-amplicons-bed`) shipped cross-pipeline. Further reconciliation tracked at
`KNOWN_ISSUES.md [ISSUE:2026-04-29:5]` part-b deferred to a future phase. See IBM-C14
PARTIALLY-RESOLVED marker for the full trail.

**Original status (pre-PARTIALLY-RESOLVED):** Open — possible side quest at any point; core feature is done

Priority 4.9 (integrated keep/exclude recommendations) was closed as DONE in v0.5.50 because
all four output files are produced and the integration logic is working. The following
enhancements from the original ROADMAP entry are preserved here for future consideration.
They are not tied to any ROADMAP phase — tackle whenever relevant.

### Context: what Priority 4.9 implemented

At the end of post-processing, onionskin produces:
- `amplicons_recommended_to_keep.bed` — all kept amplicons, annotated
- `amplicons_recommended_for_exclusion.bed` — integrated reject decision
- `candidate_amplicons_recommended_for_exclusion.bed` — intermediate accumulator
- `amplicons_actively_rejected.bed` — shape filter hard rejects

Integration logic uses `gap_concern` (summit_dense, high_density, timing_excluded,
dispersed, summit_proximal, clean) combined with width growth rescue thresholds:
```python
_WIDTH_RESCUE_MIN_SLOPE_KB = 0.0   # any positive slope qualifies
_WIDTH_RESCUE_MIN_R2       = 0.40  # R² ≥ 0.40
```

### Possible enhancements (not committed)

- **Graduated gap thresholds**: `>0.20` warn, `>0.35` reject unless rescued (currently binary at 0.20)
- **Gap count weighting**: 3–5 short gaps more suspicious than 1 long optical map gap
- **Tighter rescue thresholds**: raise `_WIDTH_RESCUE_MIN_R2` to 0.6 or require minimum slope if false rescues observed
- **Incorporate `amplicon_class`** from timing: Founder/Constitutive should require stronger evidence to exclude
- **Incorporate `origin_confidence`** from multistage refinement
- **Shape-score proximity** to threshold as a soft signal
- **Confidence-weighted exclusion**: probabilistic rather than binary
- **Timing flag**: amplicons with large positive lag AND high gap fraction (late + fragmented = suspicious)
- **User override**: `--keep-amplicons-bed` (coordinates that are always retained despite flags)

---

## [2026-04-07] Additional signal tracks for APS clustering features (from Priority 5.6)

**Status:** Open — vision only; not ready to implement; possible future side quest

Priority 5.6 proposed a `--aps-feature-matrix` flag for users to supply their own sample ×
amplicon feature matrix. Dropped (v0.5.50) because the alignment problem makes it fragile:
users cannot know `call_id`s or resolved amplicon coordinates in advance, leading to silent
misalignment errors.

### Better design: onionskin ingests signal tracks and computes features itself

Instead of a raw matrix, onionskin should accept additional input tracks and compute features
from them using its own resolved call coordinates. The user supplies data; onionskin handles
feature extraction. Examples:

- **HMM bedGraph**: once HMM is integrated (Phase 7), HMM state-path features (fraction of
  bins in state X per amplicon, mean posterior probability, etc.) are computed internally.
- **Chromatin accessibility bedGraph** (e.g., ATAC-seq): onionskin computes mean accessibility
  within each amplicon window or at the summit ± some window.
- **RNA-seq**: pass in a coverage or expression track; onionskin computes expression per
  amplicon or per gene overlapping each amplicon.
- **Any bedGraph-format signal**: a generalized `--feature-track NAME:PATH` interface that
  adds per-amplicon summary statistics (mean, median, max) to the APS feature matrix.

This design has no alignment problem — onionskin maps its own call coordinates onto whatever
tracks are provided. The resulting augmented feature matrix feeds into the same Ward/Euclidean
APS clustering machinery.

**Not a near-term priority.** Revisit after HMM integration is complete.

---

## [2026-04-07] Post-hoc summit refinement — `--refine-summits` (from Priority 4.10)

**Status:** RESOLVED-by-supersession — Phase 15's cross-pipeline summit-refinement strategy
menu (SPEC15.7 cycle 15.5a, v0.14.83 + SPEC15.8 summit↔timing convergence + SPEC15.24
HMM per-stage parabola summit emission cycle 15.7b) provides cross-pipeline summit
refinement architecture that obviates the need for a separate post-hoc `--refine-summits`
flag. The original concept is now woven into the standard pipeline summit-refinement step
across all 3 pipelines.

**Original status (pre-RESOLVED):** Open — possible side quest; never implemented; deferred pending HMM integration

Priority 4.10 was dropped in v0.5.50 to focus on HMM integration. The design is
preserved here in case it becomes relevant after HMM is in place.

### What this is

Hires manifests currently only affect summit position estimation; they do not improve
call detection, timing, APS, or any other pipeline output. The recommended pattern is:

1. Run onionskin without `--hires-manifest` (fast, base-resolution).
2. Optionally re-run summit refinement later with hires data if finer precision is needed.

### Planned design (never implemented)

A `--refine-summits` flag (or subcommand) that:
- Accepts an existing onionskin output directory + one or more hires manifest paths.
- Reads the resolved BED and any existing run parameters from that directory.
- Re-runs only the summit estimation module with hires bedGraph data.
- Updates summit-related outputs in-place (`_origins.tsv`, `_summits.*.bed`) without
  re-running detection, timing, APS, or overlap resolution.

**Also noted:** Once the summit module is meaningfully improved, benchmark the hires
benefit using the chr II ground truth BEDs and update user guidance accordingly.

**Note:** `--hires-manifest` help text and README already state the experimental status
and low first-pass value (done v0.4.18).

---

## [2026-03-31] Audit conventions for agents

**Status:** Open — still need to do
- All audits should be reported to CHANGELOG even if nothing is found to change, and especially if there are changes.
- This will help create a timestamp trail of audits.
- We will also make an AUDIT_HISTORY.md file that just lists audits, one line per audit.
- Newest audit will either be added to the top or bottom of the list.
- Agents can quickly see when the last audit was for each audit time by scanning the top (or bottom) of the list.


**When to resolve:** ASAP


---

## [2026-04-07] Future phases moved from ROADMAP

*Moved from ROADMAP.md during Phase 5/6 close-out. These are post-HMM design notes.*

# Phase 8 — Bayesian and multimodal inference

## Goals
Move from heuristic/post-hoc ordering toward formal posterior developmental inference.

---

## Priority 8.1 — Bayesian developmental ordering
### Concept
Treat morphology as a prior and APS/DNA as evidence.

### Desired direction
Blend:
- morphology prior
- APS likelihood proxy
- possibly RNA information

---

## Priority 8.2 — RNA integration
### Objective
Allow optional RNA-seq-derived sample ordering and grouping to be combined with DNA amplification state.

### Long-term vision
A unified developmental coordinate informed by:
- morphology
- DNA amplification
- transcriptome state

---

## Priority 8.3 — Joint posterior groups
### Desired result
Posterior groupings that may better reflect true developmental states than morphology alone.

---

# Phase 8.4 — Continuous developmental time axis

## Goals
Replace discrete stage indices with a continuous developmental time coordinate, enabling regression-based growth modeling, fork velocity estimation, and latent trajectory inference.

---

## Priority 8.4.1 — `--stage-values` continuous column support

**Authors:** John M. Urban, Claude Code ~2.1.81 (claude-sonnet-4-6) — added v0.5.29

### Concept

Currently, samples are assigned to integer stage bins (1, 2, 3, …). This works for grouped analyses but discards the continuous ordering information available from morphological measurements (e.g., continuous eyespot size scores, time post-eclosion, morphology composite scores).

Proposed flag:

```
--stage-values continuous_column
```

Where `continuous_column` names a column in the manifest TSV containing a numeric developmental position for each sample. This upgrades the model class from:

> ordinal/bin model → continuous latent time model

### What it enables

- **Regression-based growth curves:** fit RCN ~ f(t) instead of stage-mean steps; enables smooth growth-curve estimation
- **Fork velocity estimates:** if amplicon width grows linearly with time, the slope is a fork velocity in kb/unit_time
- **Onset time estimation:** onset = the t value where f(t) first exceeds the amplification threshold (more precise than "first stage where median ≥ threshold")
- **Comparison of prior vs APS-derived time:** use morphology-derived continuous scores as the prior, APS ranking as the posterior coordinate; correlation tells you how well morphology tracks amplification state

### Design notes

- Manifest format: add optional third column `stage_value` (numeric); `stage` column remains for grouping/display labels
- Alternatively, accept a separate `--stage-values-file PATH` TSV mapping `sample_id → continuous_value`
- All existing discrete-stage modules continue to work unchanged; continuous mode is an additional analysis layer
- Growth curve fitting in continuous mode: use isotonic regression or monotone spline (natural extension of existing `isotonic` and `unimodal` methods)
- APS ordering can be overlaid directly against the continuous prior coordinate for visual comparison

### Relationship to APS posterior ordering

The continuous stage value is the **prior** developmental coordinate (based on morphology). APS rank is the **likelihood-based** coordinate (based on DNA). Comparing them visually (scatter: continuous_stage vs APS_rank) is the natural follow-on to the current morphology-group vs APS-cluster comparison.

### Dependencies

- Requires clean per-sample bedGraph loading (already done)
- Requires manifest parser to accept optional numeric column (minor extension to `io.py`)
- Regression fitting is already partially present (`linear`, `isotonic` methods in modeling)

### Phase assignment

This is Phase 8+ work — requires stable APS infrastructure and is most useful once posterior reruns are routine. Lower priority than HMM integration (Phase 7) but higher priority than formal Bayesian modeling (Priority 8.1).

---

# Phase 8.5 — Fork progression geometry and asymmetry analysis

## Goals
Characterize the spatial and directional properties of replication fork progression within amplification domains.  These analyses go beyond scalar APS to describe *how* amplification spreads — not just how much.

---

## Priority 8.5.1 — Bidirectional fork asymmetry plots

Each amplification domain is initiated at an origin and forks travel outward in both directions.  This analysis characterizes asymmetry between the left and right flanks.

### Approach
- For each amplicon, split the RCN profile at the summit into left flank and right flank
- Compute per-stage:
  - Half-width left vs half-width right (measured at a fixed RCN threshold, e.g. 1.5)
  - Integrated area left vs right (area_excess for each half)
  - Flank gradient (slope of RCN dropoff from summit toward each edge)
- Asymmetry metric = left / right ratio for each measure
- Plot asymmetry vs stage to see if forks travel preferentially in one direction and whether that changes developmentally

### Output
- Per-amplicon asymmetry figure: left/right half-width, area, and gradient ratio vs stage
- Summary table of asymmetry metrics per amplicon per stage
- Amplicons with strong consistent asymmetry flagged for interpretation

### Biological interpretation
Persistent asymmetry may reflect:
- One fork consistently traveling further (roadblock on one side?)
- Asymmetric chromatin or sequence context slowing one fork
- One-sided amplification initiation bias

---

## Priority 8.5.2 — Fork gradient and velocity analysis

The slope of the RCN profile moving away from the origin (the "gradient") reflects how rapidly coverage drops off — which is a proxy for how far forks have traveled in the cell population.

### Approach
- Fit a model to each flank of the RCN profile (linear, exponential, or piecewise)
- Extract the gradient at multiple positions along the flank
- Track how the gradient changes across developmental stages
- Identify positions where the gradient changes abruptly — potential fork stalling zones

### Features to look for
- **Fork stalling zones**: positions where the gradient steepens unexpectedly — forks from the population are stopping disproportionately here
- **Fork acceleration zones**: positions where the gradient shallows — more DNA is replicated than expected from a simple linear fork model; possible secondary origins or fork acceleration
- **Plateau transitions**: regions where RCN approaches a local flat value mid-flank — could indicate a zone where a fixed fraction of cells replicate to that point

### Future connections
These gradient features can be compared to:
- Underlying DNA sequence (repetitive elements, G-quadruplexes, high-GC regions)
- Chromatin state tracks (if available): histone marks, ATAC-seq, ChIP-seq
- Transcription unit positions and orientations (co-directional vs head-on replication-transcription collisions)
- RNA-seq expression levels if available

---

## Priority 8.5.3 — Developmental fork trajectory visualization

Combine asymmetry and gradient into a single per-amplicon "fork trajectory" figure:
- Stage-ordered polar or Cartesian plot showing left/right fork travel distance vs stage
- Background lines = individual samples; thick line = stage median
- Highlights whether fork travel is symmetric, accelerating, decelerating, or stalling

---

# Phase 9 — Packaging, usability, and publication readiness

## Goals
Make onionskin easier to distribute, use, and cite.

---

## Priority 9.1 — Packaging
- formal installation path
- clearer dependency management
- release versioning

## Priority 9.2 — Documentation
- contributor docs
- architecture diagrams
- richer README
- output schema docs

## Priority 9.3 — CI
- standard validation in CI
- toy + twin-peak tests in CI

## Priority 9.4 — Publication support
- citation info
- reproducible example outputs
- figure-generation helpers
- release-tagged snapshots

## Priority 9.5 — Library / Jupyter API usage
### Objectives
Make onionskin modules importable directly in notebooks and scripts without going through the CLI.

### Requirements
- Core functions must have no implicit `sys.exit()`, no argparse inside module functions
- Clean module boundaries: `from onionskin_core.aps import compute_and_write_aps` should work
- Provide a minimal example notebook showing how to load results and call clustering/ordering interactively
- Document the public API surface (which functions are stable entry points)

### Use cases
- Interactive exploration of APS matrices in Jupyter
- Custom post-processing pipelines that call onionskin functions programmatically
- Integration with external tools without subprocess wrappers

---

# Phase 10 — Underreplication detection (future / low priority)

## Goals
Detect stable underreplication domains — the inverse of amplification — as observed in
*Drosophila* salivary glands and related polytene-chromosome systems. These are broad
copy-number dips (50–500 kb) that fall stably below the diploid baseline across
developmental stages.

---

## Priority 10.1 — Inverted domain detection (`--detect-underreplication`)

The mean-shift / z-score framework used for amplification detection should be
re-purposable with minimal changes:

- Flip the signal direction: look for bins with sustained **negative** z-scores
  (RCN stably below 1.0) rather than positive.
- The existing halfwidth, z-threshold, and stage-vote logic should carry over
  directly; only the sign of the detection criterion changes.
- Scope: single-file and single-sample modes initially (these are the modes where
  per-sample RCN is available without multi-stage normalization complications).
- Output schema mirrors the amplification output but with an `underreplication`
  call type tag.
- APS equivalents (e.g. "URS" — underreplication score) are optional and can
  be deferred.

**Implementation note:** The core change is likely a single sign-flip in the
detection threshold check, plus a flag to enable the mode. Should be low effort
if the mean-shift detector is well-modularized.

---

# Phase 6 — HMM (puffstep) integration and unified processing pipeline

## Goals

Integrate the user's HMM-based amplicon caller (puffstep) into onionskin as a complementary track, and work out a unified RCN processing pipeline so both tools agree on their input signal.

---

## Priority 6.1 — Unified RCN and smoothed-RCN processing

### Problem

Onionskin and puffstep independently compute RCN tracks and may apply different normalization, smoothing, or binning strategies. Before comparing calls or combining evidence, the two pipelines must agree on signal.

When puffstep is brought into this codebase, the first task is to compare:
- Raw RCN tracks (per-sample and stage-median bedGraphs)
- Smoothed RCN tracks (onionskin uses ±3-bin running median; puffstep may use a different approach)

Differences in smoothing will affect where each pipeline sees the summit, what it calls "active", and what amplitude it reports. The II/9A summit diagnostic plots already showed that onionskin's smoothed signal may look different from what puffstep sees — this needs to be resolved before combining outputs.

### Plan

1. **Track comparison**: run both pipelines on the same dataset, overlay their RCN and smoothed-RCN bedGraphs in the summit inspector (or a dedicated comparison plot)
2. **Identify discrepancies**: normalization differences (chrom-median vs. other), bin boundaries, smoothing kernel, edge handling
3. **Converge on a shared processing spec**: one definition of "smoothed RCN" used by both for summit estimation, APS, and HMM emission computation
4. **Consider**: write a single shared preprocessing module that both pipelines call, rather than duplicating logic

### Why this matters

Summit accuracy and APS_summit correctness both depend on the smoothed signal. If puffstep and onionskin smooth differently, APS_summit (now anchored at `final_origin_bp`) and the HMM's own amplitude estimates will disagree even on the same data. Convergence here unlocks reliable cross-pipeline comparison and eventual integration.

---

## Priority 6.2 — HMM as optional refinement/validation layer

### Problem

Onionskin is designed for unguided discovery (no prior knowledge of where amplicons are). Puffstep HMM requires user-defined emission and transition parameters but may outperform onionskin on specific tasks:
- Distinguishing single amplicons from spurious splits
- Confirming low-confidence calls
- Providing summit state intervals that constrain where the max-RCN bin can fall

### Plan

- Add HMM as an optional `--hmm` mode that runs puffstep after detection and refinement
- Use HMM summit-state intervals to constrain APS_summit bin selection (as an alternative to or validation of `final_origin_bp`)
- Provide sensible default parameters; allow user override
- Expose HMM output tracks alongside onionskin tracks in the standard output layout

### Deferred until

Priority 6.1 (unified processing) is complete, so both pipelines are working from the same smoothed signal.

---

# Phase 11 — Genomic sequence and annotation context

## Goals

Integrate gene annotation (GTF) and genome sequence (FASTA) as optional inputs to improve
initiation zone precision, produce sequence-level outputs for motif discovery tools, and lay
the groundwork for expression-informed analysis.  These are lofty, long-horizon features;
implementation is deferred until summit accuracy and posterior grouping are well-established.

---

## Priority 11.1 — Gene annotation (GTF) integration

### Motivation

Origin placement is constrained by gene structure: replication origins frequently occupy
intergenic regions between divergently transcribed genes.  If a summit estimator's distribution
is concentrated within a known intergenic gap, the full gap becomes a biologically plausible
initiation zone — not just the point estimate.

### Inputs

- `--gtf <file>` — optional GTF/GFF3 annotation file
- Gene records parsed to extract strand, TSS positions, and gene body extents per chromosome

### Logic

For each amplicon's summit region:
1. Identify the nearest genes flanking the summit estimate (left and right).
2. Compute the intergenic interval between the upstream gene's TES and the downstream gene's TSS
   (or between two converging/diverging gene boundaries, as appropriate).
3. If the summit estimate (point or CI) overlaps this intergenic gap, promote the gap as the
   **gene-boundary-constrained initiation zone**: the narrowest biologically plausible window
   consistent with both the data and the annotation.
4. If no clear intergenic context is found (summit in gene body, or no GTF provided), fall back
   to the CI-based initiation zone.

### Outputs

- `initiation_zone_bp_lo` / `initiation_zone_bp_hi` columns in `_origins.tsv` — the refined
  zone (gene-boundary-constrained when available, CI-based otherwise)
- `initiation_zone_source` — `"gene_boundary"`, `"ci"`, or `"fallback"`
- `_initiation_zones.bed` — BED4 of all amplicon initiation zones, name = call_id

### Design notes

- Gene-boundary logic applies only when `--gtf` is provided; all outputs are populated
  unconditionally (CI-based fallback when no GTF).
- Divergently transcribed gene pairs (common at *Drosophila* origins) should be detected
  and handled as a distinct high-confidence case.
- The constrained zone must be ≥ 1 bp and ≤ the amplicon width; no zone should extend
  outside the called amplicon boundaries.

---

## Priority 11.2 — Genome FASTA integration and initiation zone sequence output

### Motivation

Once initiation zones are defined (Priority 11.1), the genome sequence of those zones can
be extracted and exported for downstream motif analysis.  This enables users to run tools
like MEME, DREME, or FIMO directly on the candidate origin sequences without manual extraction.

### Inputs

- `--genome <fasta>` — optional genome FASTA (indexed with `.fai`, or auto-indexed on first use)
- Requires `_initiation_zones.bed` (produced by Priority 11.1, or CI-based if no GTF)

### Outputs

- `sequence/initiation_zones.bed` — BED6 copy of the initiation zones, suitable for `bedtools getfasta`
- `sequence/initiation_zones.fa` — FASTA of extracted sequences, one entry per amplicon;
  header format: `>call_id:chrom:start-end (source: gene_boundary|ci|fallback)`
- `sequence/summit_windows.fa` — FASTA of fixed ±N kb windows around `final_origin_bp`
  (default ±2.5 kb; user-configurable via `--summit-fasta-window-kb`); useful for narrow
  motif scans that do not require the full initiation zone

### Design notes

- FASTA extraction implemented via `pyfaidx` or `BioPython`; avoid subprocess calls to
  `bedtools` to keep the dependency footprint predictable.
- Both output files are omitted (with a warning) if `--genome` is not provided.
- Both files should be ready to pass directly to MEME: `meme sequence/initiation_zones.fa -dna -mod anr`

---

## Priority 11.3 — K-mer analysis and origin-relevant motif discovery

### Motivation

With sequences extracted, we can go beyond exporting to MEME and perform internal k-mer
analysis to identify sequence features that are enriched near confirmed origins and depleted
in control regions.  This provides a built-in, interpretation-level complement to external
motif tools.

### Approach

1. **K-mer frequency tables** — compute k-mer (k = 4–8) frequencies in each initiation zone
   sequence and in flanking negative-control windows of equal size.
2. **Enrichment scoring** — log-odds or chi-square enrichment of each k-mer in initiation zones
   vs controls across all amplicons; output a ranked k-mer table.
3. **Optional ML step** — train a logistic regression or random forest classifier on k-mer
   feature vectors to distinguish initiation zone sequences from controls; report feature
   importances as a proxy for origin-relevant motifs.
4. **Cross-amplicon consensus** — cluster amplicons by k-mer profile; amplicons with similar
   sequence environments may share regulatory logic.

### Outputs

- `sequence/kmer_enrichment.tsv` — ranked k-mer table (k-mer, frequency in zones, frequency
  in controls, log-odds, p-value)
- `sequence/kmer_classifier_importances.tsv` — ML feature importances (optional, only when
  `--kmer-classifier` flag is set)
- `sequence/kmer_amplicon_clusters.tsv` — amplicon-by-k-mer matrix and cluster assignments
  (optional)

### Design notes

- sklearn is the natural dependency; import is optional and guarded (warn if not installed).
- This analysis is purely additive — it does not alter any existing output or summit estimate.
- The negative control windows should be drawn from the same chromosomes and similar GC content
  to avoid composition bias.

---

## Priority 11.4 — Stage-specific expression integration

### Motivation

Two distinct use cases motivate adding stage-specific RNA-seq expression data:

1. **Posterior grouping and ordering (connects to Priority 8.2):** Expression state provides
   an orthogonal axis to DNA amplification for assigning samples to developmental stages.
   A joint APS + expression model can produce posterior groupings that better reflect true
   developmental transitions, especially in datasets where amplification alone is ambiguous.

2. **Summit and initiation zone refinement (connects to Priority 11.1):** Gene expression
   state near an amplicon's summit provides biological context: a divergently transcribed gene
   pair where both genes are actively expressed in the stage(s) where amplification initiates
   is stronger evidence for an intergenic origin than a pair where both genes are silent.
   Expression-weighted gene-boundary constraints can sharpen the initiation zone.

### Inputs

- `--expression <file>` — optional RNA-seq expression table (TSV: gene_id × sample, TPM or
  normalized counts); or a directory of per-stage expression files
- Matched to GTF gene identifiers; samples matched to amplification manifest samples by ID

### Stage-specific expression outputs

- `expression/active_genes_near_summits.tsv` — for each amplicon × stage: genes within
  `--expression-window-kb` of the summit, their expression level, and active/inactive call
  (threshold user-configurable)
- `expression/expression_constrained_zones.tsv` — initiation zones further narrowed by
  expression evidence (e.g., confirmed active divergent pair narrows zone to their gap)

### Design notes

- Expression integration is strictly additive: no existing output changes unless
  `--expression` is provided.
- The joint APS + expression posterior ordering (Priority 8.2) should share the expression
  loading module developed here.
- Stage-to-sample mapping must tolerate partial coverage: not all expression stages need to
  match amplification stages exactly; nearest-stage fallback with a warning.

---

# HMM step 9 — summit state robustness

## Known weakness: spurious high-state summit extraction

**Context (recorded 2026-04-09, Phase 8 close-out):**

Step 9 identifies the summit state interval within each above-background region by finding
the bin(s) with the highest HMM state value. This works correctly when the HMM behaves
well — i.e., when the highest state genuinely sits at the replication origin and the
state path descends monotonically to both sides (the expected "replication bubble" profile).

However, if the HMM has a spurious jump to a high state anywhere in the region — caused
by an outlier bin that escaped median smoothing, or a data artifact — that bin will be
extracted as the summit regardless of its biological plausibility. The result is a summit
estimate that may be positioned at a spike rather than the true origin.

**Mitigations already in place:**
- Median smoothing of the RCN track before HMM (step 5, `--hmm-smooth-halfwidth`) removes
  most outliers before the state path is computed.
- Exponential decay in transition probabilities (`--hmm-exp-decay on`) penalises transient
  state jumps because they require two unlikely transitions (up and then down).
- `--hmm-thresh-state`, `--hmm-merge1`, `--hmm-min-width`, etc. (step 8) can filter out
  small spurious regions before summit extraction.

**Potential future solutions:**

1. **Structural validation of the summit state interval**: after extracting the peak-state
   bin, check whether it is (a) approximately centred in the enclosing region and (b) flanked
   by monotonically descending states on both sides. If not, flag it or fall back to the
   centroid of the highest-contiguous-state run. This is analogous to the shape filter used
   in the multistage pipeline.

2. **Cross-pipeline summit consensus**: if the multistage or per-stage mean-shift pipeline
   also ran, compare HMM summit positions against those estimates. Large disagreements (e.g.,
   > half the region width) are a signal that the HMM summit is a spurious outlier. Could be
   a soft flag or a hard fallback depending on user preference.

3. **Light median smoothing of the state path itself before summit extraction**: apply a
   narrow median smooth (e.g., ±1 or ±2 bins) to the collapsed state-path bedGraph before
   step 9. This would suppress single-bin or two-bin state spikes without altering broad
   genuine amplification structure. Importantly, this would be smoothing the *state values*
   (integers 1–7), not the RCN signal — so it is a post-HMM correction rather than a
   pre-HMM input change. The smoothed state path would only be used for summit extraction;
   the original state path would still be written to disk for all other purposes.
   **This is probably the simplest and most robust single fix if the problem proves common.**

4. **Constrain the window considered for summit extraction**: rather than searching the
   entire above-background region for the max state, restrict search to the central N% of
   the region (e.g., middle 80%). This prevents edge-of-region outliers from being selected
   as the summit, at the cost of potentially missing real off-centre summits.

**Priority:** Low — when HMM is well-tuned to the data (which is the design intent),
this issue does not arise in practice. Revisit if HMM summit positions diverge from
multistage estimates on real datasets, or when Phase 9.2 (cross-pipeline integration) lands.

---

# Future cross-pipeline synthesis

**Scope:** Unify the results of all pipelines (multistage growth, per-stage, HMM) into a
single final set of amplicon calls. This is future work after the cleanup-only closeout and the
forward architecture phase have made the pipeline-local products explicit and well-characterized.

**Core goal:** Derive a final set of amplicon calls that is more accurate and more complete
than any single pipeline alone. Amplicons supported by multiple pipelines are elevated in
confidence; amplicons unique to one pipeline are scrutinized but not automatically discarded.

---

## Amplicon call unification

- For each amplicon candidate, record which pipelines detected it and with what evidence.
- Assign a confidence tier:
  - **Tier 1:** detected by HMM + multistage growth + per-stage (strongest)
  - **Tier 2:** detected by two pipelines
  - **Tier 3:** detected by one pipeline only (weakest; subject to secondary inspection)
- Eliminate calls that appear artifactual in only one pipeline (e.g., a spurious HMM fit
  that has no RCN support in the other pipelines).
- Preserve calls that look real even if from only one pipeline, provided secondary evidence
  supports them (e.g., consistent RCN shape, multi-stage growth pattern).

## Final boundary estimation

- For each accepted amplicon, derive a final set of boundary estimates by combining pipeline
  outputs:
  - Lean heavily on HMM state-path boundaries when available — they are spatially precise
    and biologically interpretable as fork travel endpoints.
  - Use multistage growth boundaries as corroboration or fallback.
  - Derive a final summit/origin position estimate from the intersection of summit-state
    intervals (HMM), parabola fits (multistage), and per-stage estimates.

## Secondary HMM inspection for HMM-missed amplicons

- Flag amplicons detected by other pipelines but absent from the HMM output.
- Known cause: broad, low-fold (e.g., 1.5×) triangular domains are detectable by gradient-
  based methods but may be missed by the default HMM emission parameters.
- For flagged amplicons: re-run HMM locally on that genomic region with updated emission
  parameters tuned to detect step-up/step-down hill shapes at lower fold-change.
  If the local HMM succeeds, incorporate its state-path boundaries into the unified result.
- This gives us HMM-derived fork travel estimates for amplicons that were otherwise HMM-dark.

## Unified output layer (`04-unified-results/`)

- `unified_amplicon_calls.tsv` / `.bed` — final amplicon set with confidence tiers
- `unified_boundaries.tsv` — final left/right boundary estimates per amplicon, with
  provenance (which pipeline(s) contributed to each estimate)
- `unified_summit_calls.tsv` — final summit/origin position estimates, method-of-origin noted
- `unified_rejected_calls.tsv` — calls considered and rejected, with rejection reason
- `unified_stage_calls.tsv` — per-stage amplicon presence, boundaries, and metrics
- Per-amplicon pipeline support summary (presence/absence matrix across pipelines)

## Pipeline-local posterior baseline before synthesis

- Until cross-pipeline synthesis exists, each pipeline's posterior grouping / ordering /
  manifest should descend from that same pipeline's `01-prior/` results.
- This means pipeline-local APS can drive pipeline-local posterior grouping/order/manifest
  without waiting for `04-unified-results/`.
- If synthesis is intentionally disabled in the future (for example with a `--no-synthesis`
  flag), these pipeline-local prior/posterior products should remain first-class outputs rather
  than fallback leftovers.

## Meta-analyses on unified calls

- Fork travel trajectory analysis (Phase 9 methods) applied to the unified amplicon set
  using the best available HMM state paths (including locally re-run ones).
- APS, timing, and clustering computed on the final unified amplicon set.
- Cross-pipeline concordance metrics: how often do boundaries agree? How large are
  discrepancies? Informs future pipeline improvements.

## Open posterior synthesis questions

- If `01-prior/04-unified-results/` exists, should `02-posterior/` synthesis descend from that
  prior unified synthesis, from a new synthesis performed after the posterior reruns, or both?
- Should all posterior pipelines eventually run on one shared posterior manifest produced by the
  prior unified synthesis, or should pipeline-local posterior manifests remain first-class even
  after unified synthesis exists?
- Should unified APS be synthesized from the per-pipeline APS outputs, or recomputed after calls,
  summits, and boundaries are synthesized into one unified result set?
- If unified synthesis is active, should the per-pipeline posterior outputs remain emitted by
  default, become optional, or be suppressible with dedicated flags?

## Design notes

- Unification logic should be modular and independently testable.
- Confidence tiers and provenance tracking must be explicit in all output files — never
  silently merge pipeline outputs.
- Shared APS math should not be mistaken for growth-pipeline ownership of APS. Pipeline-local APS
  outputs are expected, and unified APS is a later synthesis-layer question.
- The secondary HMM inspection step is optional and may be gated behind a CLI flag
  (e.g., `--hmm-reinspect-missing`) to allow faster runs.
- Later forward-architecture work may reveal that some pipelines are systematically biased for
  certain amplicon types. Those findings should feed back into upstream pipeline parameter
  tuning.

---

---

## [2026-04-19] HMM parallel child pipeline (per-sample individual decoding) — HIGH PRIORITY

**Status:** Design agreed. Escalated to HIGH priority — prerequisite for SAPS, step-14 APS
improvement, amplicon reliability scoring, and pre-amp flat-sample detection.

**Problem it solves:** The current HMM pipeline runs jointly on all samples (stage-median
bedGraphs). Individual per-sample decoding is needed for: (1) SAPS (state-based APS),
(2) per-sample amplicon QC, (3) flat-sample detection, (4) step-14 APS using
properly-normalized per-sample bedGraphs instead of raw files.

**Diagnosed current step-14 problem:** `run_step14_hmm_aps()` reads raw manifest bedGraphs
and re-normalizes via `compute_sample_rcn_tracks()` (chrom-median applied in `aps.py`). It
does NOT use the `01-mednorm` files from the HMM pipeline. This means: (a) HMM step-14 APS
operates on differently-normalized data than the state calling steps, and (b) the smoothing
applied in step 5 is NOT used by step 14. Both are problems. The parallel child pipeline
addresses both.

**Design:**
Starting at step 03-removeZeroBins, create an `indiv_samples/` subdirectory in each step
directory so that individual samples are processed in parallel with the joint pipeline:
- `03-removeZeroBins/indiv_samples/` — same zero-bin mask as joint run applied per sample
- `04-medNormOrRCN/indiv_samples/` — per-sample chrom-median renorm (no ratio; zero bins excluded)
- `05-medianSmoothedRCN/indiv_samples/` — step-5 smoothing (median or trimmed mean) per sample
- `06-HMM/indiv_samples/` — HMM state calling per sample (transition probs from joint run or fresh)
- `07-collapse/indiv_samples/`, `08-aboveBackground/indiv_samples/`
- `09-summitStates/indiv_samples/`, `10-summitBins/indiv_samples/`
- `11-ampliconMetrics/indiv_samples/`

Then:
- `14-aps/` uses `05-medianSmoothedRCN/indiv_samples/` instead of raw manifest bedGraphs
- `15-saps/` uses `06-HMM/indiv_samples/` state paths to compute state-based APS
- Amplicon reliability scoring uses `08-aboveBackground/indiv_samples/` and `11-ampliconMetrics/indiv_samples/`

**Step numbering:** `15-timing` → `16-timing`, `16-clustering` → `17-clustering`. SAPS at `15-saps/`.
Step numbering change requires coordinated update across `hmm_engine.py`, `hmm_ported_analyses.py`,
`hmm_notebooks.py`, `summit_inspector.py`, `timing.py`, `onionskin.py`, and test paths.

**Priority note:** Should be promoted into PHASE11_SPEC under a new Priority (11.4 or similar).
Amplicon reliability scoring and pre-amp detection are downstream of this.

---

## [2026-04-18] SAPS — State-APS from individual-sample HMM decoding

**Status:** Vision / exploratory. Prerequisite = parallel child pipeline (see above).
**Updated priority: HIGH** — now a near-term deliverable after the child pipeline is implemented.

**What it is:** SAPS (State APS) computes APS on each sample's decoded HMM state path instead
of on raw RCN values. This gives an amplification score that is HMM-posterior-denoised rather
than raw-coverage-based.

**Pipeline concept (updated):**
1. Use `03-removeZeroBins/indiv_samples/` per-sample bedGraphs.
2. Use `04-medNormOrRCN/indiv_samples/` chrom-median renorm.
3. Use `05-medianSmoothedRCN/indiv_samples/` (already smoothed — skip step-14 re-smoothing).
4. Run HMM steps 6→7→8→9→10→11 per sample (state paths in `06-HMM/indiv_samples/`).
5. SAPS at `15-saps/` uses state paths from step 6.

**SAPS metrics per locus:**
- `saps_area` = Σ (state_value − 1) × bin_width, where state_value is the decoded HMM state
- `saps_summit` = summit_state + summit_length / 1,000,000 (for tie-breaking)
- `saps_width` = total bp in state > 1 (above-background)
- `saps_shape` = per-bin state value vector (profile shape)

**HMM state encoding reminder:** States are 1-based. State 1 = background. State ≥ 2 =
amplified. `--hmm-background-idx 0` means background is at index 0 internally but output is
1-based.

**Output location:** `15-saps/` (timing bumped to 16, clustering to 17).

---

## [2026-04-18] HMM-native amplicon quality criteria

**Status:** Vision / exploratory. Not implemented; possibly replaces or augments growth-model BIC chain for HMM-only calls.

**Motivation:** The current HMM pipeline borrows growth-model BIC/shape criteria for amplicon
filtering. For amplicons that are HMM-detected but growth-dark, or for improving HMM-only
confidence, HMM-native quality signals would be more principled.

**Proposed criteria:**
1. **Widening**: above-background region widens across developmental stages
2. **Summit increase**: highest decoded state ever increases (not just appears)
3. **Hill shape**: HMM state profile shows rise→peak→fall structure (2→3→2+, etc.)
4. **Persistence**: amplicon is detected continuously from first-observed stage onward
5. **Shape filter**: triangle vs flat classification in maximal-amplification stage
   (outputs a BED file marking flat amplicons as lower-quality or unreliable)

**Output ideas:**
- `hmm_amplicon_quality.tsv` — per-amplicon flag matrix for each criterion
- `hmm_flat_amplicons.bed` — amplicons that fail triangle filter in maximal stage
- `hmm_persistent_amplicons.bed` — subset that satisfy persistence from first detection

**Not a near-term priority.** Revisit as HMM completeness work (Priority 11.3) matures.

---

## [2026-04-18] Per-chromosome APS clustering bootstrapping experiments

**Status:** Exploratory — understand how each chromosome's signal quality affects APS clustering.

**Motivation:** ds2+4chr experiments revealed that chr IV (small amplicons, possible false
positives) and chr X (X/X' heterogeneity) can degrade clustering that works well on chr II
and III. Systematically testing single-chromosome and pairwise combinations would map
which chromosomes are signal contributors vs noise contributors.

**Proposed experiments:**
- Single chromosome: II alone, III alone, X alone, IV alone
- Pairs: II+III, II+X, II+IV, III+IV, III+X
- Triple: II+III+X, II+III+IV
- Full: II+III+X+IV (already done)

**Makefile targets to add:** `clust-ds2-chrII`, `clust-ds2-chrIII`, `clust-ds2-chrX`,
`clust-ds2-chrIV`, `clust-ds2-chrII-III`, etc. — or a single `clust-ds2-bootstrap` target
that runs all combinations.

**Expected findings:**
- Chr II is the most reliable ground truth (large, unambiguous amplicons).
- Chr III is reliable secondary signal.
- Chr X may add noise due to hemizygosity / X/X' heterogeneity.
- Chr IV may add confusion due to small amplicons near BIC threshold.

**Not a near-term priority.** The `summit+keep` default already handles the 4-chr case
well. Run these experiments when investigating a dataset where clustering fails.

---

## [2026-04-18] APS_mean_of_means clustering test

**Status:** Exploratory. `APS_mean_of_means` already exists in the APS output but is unused.

**Motivation:** Current APS (area metric) is size-weighted by construction — large amplicons
dominate. `APS_mean_of_means` weights each locus equally regardless of size. Testing whether
equal-locus-weight clustering produces better stage separation on datasets where a few large
amplicons swamp the signal from many small ones.

**Experiment:** Re-run `clust-ds1-II` and `clust-ds2-II` with a modified feature that uses
`mean_of_means` per locus instead of summed area, and compare cluster quality.

**Not a near-term priority.** The `summit+keep` default already avoids the size-domination
problem (summit feature is inherently per-locus equal-weight). This experiment is only
worth running if summit clustering fails on a new dataset.

---

## [2026-04-19] Amplicon reliability scoring and flat-sample detection before APS/clustering

**Status:** Design discussion complete. Not yet scoped into active phase spec.

**Motivation:** APS clustering and posterior stage ordering assume that stage 1 contains
pre-amplification samples. If partly-amplified samples land in stage 1 ("baseline contamination"),
the growth model's reference is inflated and downstream fold-change estimates are underestimated.
The underlying problem is that we currently have no formal degree-of-belief scoring for candidate
amplicons, and no way to identify samples that are flat at confirmed-amplicon loci.

**Agreed design direction:**

Step 1 — Amplicon reliability scoring (before APS)
A pre-APS analysis pass that evaluates each candidate amplicon across all stages on a set of
metrics. Amplicons present in more stages get more metrics evaluated.

Metrics applicable when amplicon is present in ≥2 stages:
1. Does the summit RCN increase across stages? (by ≥1.5× fold minimum)
2. Does the amplicon width increase across stages? (by ≥50 kb or some threshold)
3. Does the total area (sum of excess RCN across bins) increase across stages?
4. Does the triangle BIC score increase across stages? (More triangular with time = real gradient)
5. Does the parabola summit height increase across stages?

Metrics applicable when amplicon appears in only 1 stage, or for 1-doubling amplicons:
- Triangle vs flat BIC score (is the amplicon shape distinguishable from noise in that stage?)

For amplicons present in ≥2 stages but showing only 1 doubling:
- Width and area increase (fork elongation without RCN increase)
- Persistent same-width / same-area across stages → likely collapsed repeat, not amplicon

**Step 2 — Flat-sample detection (using curated amplicons)**
Once a reliable set of amplicons is identified, for each sample, determine whether it is flat
at all (or >X%) of the reliable amplicon loci. Checks per sample per locus:
- Is the summit bin RCN < 1.5× chromosome median?
- Does the amplicon pass a "flat" BIC score at that locus in that sample?

A sample is "flat" if it is flat at essentially all reliable amplicon loci. That sample becomes
a candidate for posterior stage 1 anchoring.

**Handling ambiguity:** Samples that fail a few tests at a few loci get a separate label
("ambiguous"). The hard flat-sample assignment only goes to samples clean across the board.
Downstream warnings (or new posterior QC check) can flag when stage 1 contains ambiguous samples.

**Application to both pipelines:**
- Growth pipeline: apply the triangle BIC, parabola height, width/area metrics directly to
  calls from `_origins.tsv` and stage-median bedGraph profiles.
- HMM pipeline: same metrics plus state-path–based QC (state occupancy across stages,
  summit state persistence, etc.) in a second pass. HMM-specific metrics designed separately.

**Scope and priority:** First pass implements the pipeline-agnostic metrics (1–5 above plus
flat-sample assignment). Second pass adds HMM-specific state-path metrics. This is a
significant feature addition — likely its own Priority under Phase 11 or Phase 12.

**Open questions:**
- Which thresholds are data-driven vs user-settable? Probably most should be hard defaults
  with expert-override flags, not required user input.
- How to handle amplicons that appear in only the last few stages (late-starting)? May need
  special treatment since they cannot show early-stage increase.
- Does this produce a new output file (e.g., `amplicon_reliability.tsv`) or annotate an
  existing calls TSV? Likely new file plus annotation columns in the calls TSV.

---

## [2026-04-18] `--hmm-0-based-statepath` future CLI flag

**Status:** Future option. Breaking change if made the default; safest as an opt-in flag.

**What it does:** Makes HMM state-path outputs use 0-based indexing (state 0 = background,
state 1 = first amplification step, etc.) instead of the current 1-based convention (state
1 = background, state 2 = first amplification step).

**Why consider it:** With 0-based states, the expected amplification level at a bin is
directly `2^state` (e.g., state 0 = 2^0 = 1× background, state 2 = 4× amplification). This
makes the mathematical relationship between state values and copy number explicit.

**Why it is not the default:** The current 1-based convention is established in all existing
outputs, scripts, and documentation. Changing it silently would break downstream use.

**Design:** A `--hmm-0-based-statepath` flag (default False) that subtracts 1 from all
state-path output values. Internal computation remains unchanged; only output encoding shifts.

---

# General development principles

Across all future phases:

1. keep the core detector stable
2. treat interpretation as layered modules
3. preserve deterministic, interpretable outputs
4. prefer explicit schemas and output contracts
5. add tests whenever adding a feature
6. always ship full release bundles, not patches
7. keep release archives extracting into a single `onionskin/` directory
8. normalize permissions before release
9. maintain changelog and roadmap as first-class project documents

---

# Immediate next recommendation

The most practical next steps from the current Phase 3.23+ state:

1. **APS correctness audit** — verify the APS computation, clustering, and ordering logic before building further on it
2. **Output metric bug fixes** — fix hires coarsest-wins (summit accuracy) and the always-NaN/always-0 output columns
3. **Finalize CLI exposure** — complete option pass-through so all major parameters are accessible from `onionskin.py`
4. **Wire sample/stage track outputs** — ensure `samples/` and `stage_medians/` directories are populated
5. **Then move into posterior reruns**

---

## [2026-04-15] CLI flag terminology harmonization — deferred from Phase 11 wrap-up triage

**Status:** Deferred. Requires a dedicated scoped CLI harmonization pass.

**Context:** Phase 11 wrap-up audits (Gemini + Copilot supplemental) flagged that `--z-thresh-single`
and `--z-thresh-multi` still encode the old input-property vocabulary (`single` vs. `multi`) rather
than the pipeline vocabulary (`growth-model`, `per-stage`) established in Phase 11.

The codebase has already partially moved to pipeline-prefixed flags (`--growth-norm-mode`,
`--per-stage-norm-mode`, etc.), so the inconsistency is real. However:

- CLI flag renames are user-facing and represent a breaking API change.
- A complete harmonization requires a full CLI audit to identify all flags that still use
  the old vocabulary and decide their pipeline-prefixed equivalents.
- The scope can expand quickly without a tight spec.

**Recommendation when opening this pass:** Start by inventorying all `--z-thresh-*`, `--growth-*`,
`--per-stage-*`, and `--hmm-*` flags in `onionskin.py` and write the full rename mapping before
any implementation. Decide whether to add deprecated aliases or make clean breaks.

**Do not bundle with the output-layout key migration (Phase 11 post-wrap-up FIND-I).** They are
different risk levels.


---

## [2026-04-21] Developer-tooling: `--validate-flags` legacy-flag scanner

**Status:** Speculative. Deferred from Phase 14 (rejected as out-of-scope; pre-parse deprecation
errors already serve this purpose for Phase 14's needs).

### Idea

Add a developer-only `--validate-flags` mode that scans `sys.argv` (or a provided script)
for any known legacy flag strings and reports them with their current equivalents. This would
help catch drift in `tests/` and `scripts/` that isn't caught by CI or by running a test that
happens to use the old flag.

### Why it was deferred

Phase 14 already adds hard-break pre-parse deprecation errors for all removed flags. Any
invocation of an old flag immediately produces a named redirect message. This makes
`--validate-flags` redundant for Phase 14's specific goals.

The idea becomes more interesting in a larger codebase where scripts are run non-interactively
and the deprecation catch may not be surfaced. If onionskin grows a larger external user base
or if long-lived automation scripts become a concern, revisit this.

### Potential form

```
onionskin --validate-flags -- --old-flag-name arg --another-old-flag ...
# or as a standalone sub-command
onionskin validate-flags path/to/script.sh
```

---

## [2026-04-29] Bayesian-weighted amplicon reliability — generalize binary keep/reject to a weighted "degree of belief" framework

**Status:** Brainstorm-stage idea. Surfaced during cycle 15.4a-S1 keep/exclude framework reconciliation discussion (user direction 2026-04-29).

**Motivation:** The current keep/reject framework (cycle 15.4a `reliability_flag`, Priority 4.9 keep/exclude BEDs, shape filter rejected.bed, gap-analysis annotation) treats amplicon reliability as binary: either an amplicon passes the test set and is kept, or it doesn't and is rejected. The user's intuition (from `--aps-weight-loci`'s long-deferred design space, and 2026-04-29 chat): a **per-amplicon weight in [0, 1]** representing degree of belief that the amplicon is real would be a cleaner generalization.

**The framing:**

- **Start at 0** (assume false) — the Bayesian prior is "this is noise" until evidence proves otherwise.
- **Each reliability test increases the weight** as evidence accumulates: triangle-vs-flat BIC pass, triangle BIC increases across stages, parabola height increases, area increases, width grows, no gap concerns, etc. Each independent test adds belief.
- **Final weight in [0, 1]:**
  - 0 = collapsed repeat / clear false positive.
  - 1 = II9A-tier high-confidence amplicon.
  - In between = ambiguous / partial-evidence amplicons.
- **The binary keep/reject machinery becomes a degenerate case** where the weight is thresholded into 1 or 0.

**Interpretation of "weight" depends on consumer:**

- **Some analyses can ignore weight entirely** and run on the full candidate set (per-amplicon outputs are individually inspectable; false positives just produce ignorable rows). Examples: per-amplicon timing classifier, individual call rows, plots showing all candidates with their classification.
- **Some analyses are HARMED by including unreliable amplicons** and should consume the weight as either a binary filter OR a probabilistic weight in their aggregation:
  - **Sample-level APS:** `APS_area = sum(area_excess × weight)` instead of `sum(area_excess)`. Direct extension of `--aps-weight-loci` to use `reliability_score` instead of placeholder 1.0. See `[ISSUE:2026-04-29:6]`.
  - **APS clustering:** today (cycle 15.4a Stage J) uses a binary filter (`reliability_flag != "filter"`). Could become weighted feature contributions in the clustering matrix (each locus contributes its `feature × weight` instead of `feature`). Probably more interpretable + robust than binary filtering.
  - **Posterior cluster-map / posterior manifest:** weight-aware posterior stage assignment? Probably overkill — sample-level weight doesn't generalize cleanly. Stick with binary filter at this layer.

**Why this matters:** the binary-filter framework loses information at the boundary. An amplicon scoring 0.55 on the reliability formula (mostly positive evidence but one or two test failures) is treated identically to one scoring 0.05 (clear false positive). Both are filtered out under the binary `reliability_flag != "filter"` rule, but their actual evidential support differs by an order of magnitude. The weight framework preserves that gradient, lets sample-level aggregations downweight (rather than discard) ambiguous amplicons, and gives users a cleaner audit signal ("this amplicon contributed 0.55 × area to the sample APS — moderately confident").

**Implementation sketch (rough; not a SPEC):**

1. **Reuse `reliability_score`.** Cycle 15.4a's reliability scorer already produces a per-call `reliability_score` in `[0, 1]` from the 5-axis evidence framework. This IS the weight in the framing above (or close to it — the formula may need re-tuning for explicit Bayesian-shrinkage semantics).
2. **Promote `--aps-weight-loci` to default-on (or replace canonical aggregation with weighted form).** Today the flag is opt-in and uses `locus_weight = 1.0` (placeholder) unless `reliability_score` is present (cycle 15.4a wired this conditionally). Making the canonical APS columns use weighted aggregation closes `[ISSUE:2026-04-29:6]` directly.
3. **Generalize the gap-analysis defect fix to use weights.** Instead of "gap-suspect → hard filter," gap evidence becomes one of the axes that subtracts from the weight (still in `[0, 1]`). Compatible with `[ISSUE:2026-04-29:5]`'s reconciliation discussion.
4. **Class-aware weighting still applies.** SPEC15.6 deliverable 8's locked rule (Founder + Constitutive_Prolonged need stronger evidence to exclude) becomes class-aware-prior in the Bayesian framing — the prior weight might be class-dependent (already-known-class amplicons start at higher weight; ambiguous ones start at lower weight). The 4 DS1 chr II calibration amplicons MUST remain weight ≈ 1.0 — same regression gate as today.

**Connection to the current reliability_score design:**

Cycle 15.4a's `reliability_score` is already in `[0, 1]` and aggregates 5 evidence axes. The current binary-filter is `reliability_flag = "filter"` when `reliability_score < class_threshold`. The Bayesian-weighted framing essentially says: drop the threshold, use the score directly as a weight, and let downstream aggregations consume the gradient. The plumbing is largely in place; what's missing is (a) consumer-side adoption of the weight (only `--aps-weight-loci` honors it today, opt-in only) and (b) the Bayesian semantics on the formula (current scorer is heuristic-aggregated, not explicitly Bayesian).

**Risks / open questions:**

- **Does the current 5-axis `reliability_score` actually behave like a Bayesian posterior?** The formula at `reliability.py:_class_threshold + composite mean of axis scores` is heuristic — the evidence-axis sub-scores aren't true posteriors and the composite isn't a Bayesian update. To make the Bayesian framing rigorous, the formula would need re-derivation. Probably acceptable to ship the existing score as a "weight" in `[0, 1]` while deferring the rigorous Bayesian re-derivation to a future cycle.
- **Backwards-compatibility for any analysis pipeline currently consuming binary filter outputs.** Switching default APS aggregation from sum to weighted sum is a behavioral change; needs versioning + side-by-side outputs at first.
- **Weight propagation through clustering algorithms.** Hierarchical clustering with weighted features is a real thing but not all clustering implementations support it cleanly. May require feature-matrix pre-multiplication (each row × sqrt of locus weight) or a clustering-method choice that accepts weights natively.
- **Class-aware prior calibration.** The 4 DS1 chr II calibration amplicons regression gate must remain a hard contract.

**Likely landing zone:** Phase 16+ if scope is limited to "extend reliability framework with explicit weighted semantics + class-aware prior + Bayesian-shrinkage formula re-derivation." Smaller subset (promote `--aps-weight-loci` to default + use `reliability_score` as the weight) could land in Phase 15 cycle 15.6a (SPEC15.9-12 APS analytical work) — see `[ISSUE:2026-04-29:6]`.

**Cross-references:**
- `[ISSUE:2026-04-29:5]` (parallel keep/exclude framework reconciliation — this brainstorm is one possible architecture for the unified framework).
- `[ISSUE:2026-04-29:6]` (sample-level APS reliability gap — direct subset of this brainstorm's APS-aggregation idea).
- `aps.py:--aps-weight-loci` flag (long-existing partial precursor; ROADMAP Priority 5.2).
- IBM-C14 (Phase 14; partial-resolved; same conceptual neighborhood).
- ROADSTRAVELED archive Priority 4.9.
