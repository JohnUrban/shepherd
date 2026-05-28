# Upstream Flat-Sample Detection via Single-Pipeline Prior — SOUP

**Stage:** SOUP (pre-BRAINSTORM scratchpad in `multi-agent/plans/next/`).
**Theme:** Architectural decoupling of flat-sample detection from each pipeline's
own normalization machinery. A single pipeline runs first as a "flat-detection
prior" (using chrom-median or GC-aware chrom-median normalization, NOT ref-stage),
emits a shared flat-samples sidecar, and ALL active pipelines (Growth, RMS, HMM)
then consume that sidecar for stage-1 anchoring + auto-switch decisions before
their own ref-stage-normalized work runs.
**Lifecycle:** Promotes to a per-phase BRAINSTORM in `multi-agent/plans/`
when the orchestrator brings it onto the stage and assigns a phase number.
See `multi-agent/AGENT_CONVENTIONS.md § Future-phase planning surfaces`
and `multi-agent/workflows/phase-development-system_PDS-v2.md` for the
SOUP → BRAINSTORM → SPEC lifecycle.

**Created:** 2026-04-30 by Claude Code 2.1.117 (claude-opus-4-7), per user
direction during Phase 15 cycle 15.4a-S2 corrections appendix authorization.
Triggered by user observation that fixing flat-detector defects in-cycle
(`[ISSUE:2026-04-30:3]`) is a tactical patch on a structural problem:
flat-sample detection runs INSIDE each pipeline's own normalization
context, which means each pipeline can be tricked by its own
ref-stage-normalization pollution into mis-classifying samples. The
structural fix is to detect flat samples ONCE upstream, before any
pipeline picks a normalization mode, and feed that decision down.

**Status:** SOUP-stage. Not started. Out of scope for Phase 15. Probably
graduates AFTER `multi-agent/plans/next/GECKO_SOUP.md` (GC-aware
chrom-median normalization) so the upstream flat-detection prior can
inherit GC-aware chrom-median as its normalization mode of choice. See
"When to start" below.

---

## Why a dedicated upstream-flat-detection phase

The user surfaced the underlying architectural mismatch at cycle 15.4a-S2
corrections-appendix time (2026-04-30, verbatim):

> *"this does probably call for a future phase to basically require 'prior'
> work to be chrom-median to determine flat samples before ref-stage is
> allowed -- that could be FLAT_SOUP --> it would be something we would want
> to do for all pipelines --> and if more than 1 pipeline is picked, it
> would be done by a single pipeline upstream of running the pipelines
> with a flag like `--flat-detection-method` (hmm or rms or growth, default
> hmm or rms)."*

The structural problem the SOUP addresses:

- **Today (Phase 15):** each active pipeline runs its own
  `_check_flat_sample_auto_switch` AFTER its own normalization pass. Growth
  flat-detects against Growth's ref-stage-normalized signal; RMS against
  RMS's; HMM against HMM's. If any of those normalizations are polluted
  (because flat-detection determines which samples are valid stage-1
  anchors, but the normalization step chose anchors BEFORE flat-detection
  ran — circular dependency), the flat-classification is unreliable. The
  cycle 15.4a-S2 in-cycle fix (Stage B.1: flat-detection on chrom-median
  signal) is a tactical patch within each pipeline; it does not address
  the architectural circularity.
- **Future (this SOUP):** flat-sample detection runs ONCE upstream, in a
  dedicated "prior" phase, using a normalization mode that does NOT depend
  on stage-1 anchor selection (chrom-median; ideally GC-aware chrom-median
  per `GECKO_SOUP.md` if that has graduated first). The upstream prior
  emits `flat_samples.tsv` (and any companion sidecars). All active
  pipelines THEN run, with stage-1-anchor decisions informed by the
  upstream prior's flat-classification. Each pipeline's normalization
  mode is chosen with knowledge of which samples are reliable anchors.

The circular dependency is broken by adding a phase BEFORE the per-pipeline
runs that doesn't require stage-1 anchor decisions to operate.

---

## What this SOUP is about

A new architectural phase + CLI surface + cross-pipeline contract that:

1. Runs ONE selected "flat-detection prior" pipeline FIRST, using a
   normalization mode that doesn't depend on stage-1 anchor selection
   (chrom-median or GC-aware chrom-median). The prior pipeline's only job
   is to emit a flat-samples sidecar; its own analysis outputs are
   discarded or kept as diagnostic-only.
2. Emits a SHARED `flat_samples.tsv` (and possibly a SHARED reliability
   sidecar) at a path discoverable by all downstream pipelines.
3. THEN runs all selected pipelines (Growth, RMS, HMM) with stage-1
   anchoring + auto-switch decisions informed by the upstream prior.
   Pipelines may use ref-stage normalization (now safe because flat
   samples are reliably identified) OR chrom-median per existing
   `--*-norm-mode` flags.
4. Provides a new CLI flag `--flat-detection-method {hmm,rms,growth,all}`
   to select which pipeline acts as the prior. Default: TBD (likely
   `hmm` or `rms`, see Open brainstorm questions).
5. Cross-pipeline contract: identical `flat_samples.tsv` schema regardless
   of which pipeline ran the prior. The schema is whatever Phase 15 lands
   per SPEC15.6 deliverable 5; this SOUP commits to consuming that schema
   unchanged across all pipeline-as-prior configurations.

---

## Items parked here

### Item 1 — `--flat-detection-method` CLI flag + prior-phase orchestration

**Background.** Add a new flag to onionskin's argparse surface
(`onionskin.py:build_parser`) that selects which pipeline runs as the
upstream flat-detection prior. Likely value set: `{hmm, rms, growth, all,
auto, off}`. Default value depends on Item 2's pipeline-quality decision.

**Orchestration changes** in `onionskin.py` controller:
- After parsing CLI args + resolving active pipelines, but BEFORE entering
  any per-pipeline branch: if `--flat-detection-method` is not `off`,
  run the selected pipeline in "flat-detection-prior mode" with
  normalization forced to chrom-median (or GC-aware chrom-median if
  GECKO has graduated).
- Capture the prior's `flat_samples.tsv` at a controller-resolved path
  (e.g., `<out-dir>/01-prior-flat-detection/flat_samples.tsv`).
- Skip `_check_flat_sample_auto_switch` calls inside individual pipelines
  (set per-pipeline auto-switch flags from the upstream sidecar instead).
- Per-pipeline calls to `_run_*_engine` proceed normally, with
  stage-1-anchoring decisions informed by the upstream prior.

**Edge case:** if the prior pipeline is the same as the only active
pipeline (e.g., user runs `--pipelines hmm --flat-detection-method hmm`),
the upstream prior run + the main HMM run can either be (a) the same
single execution (cache flat-samples between the upstream prior phase
and the main analysis), or (b) two separate runs (simpler but wastes
compute). Decision deferred to brainstorm rounds.

**Effort sizing.** Medium. New CLI flag + ~50-100 lines of controller
orchestration + per-pipeline integration to consume the upstream sidecar
instead of running its own flat-detection. New end-to-end regression
tests. Documentation updates.

### Item 2 — Pipeline-quality decision: which is the best "flat-detection prior"?

**Background.** Three candidate pipelines (Growth, RMS, HMM); each has
different sensitivity / specificity tradeoffs for flat-sample detection.
The SOUP's brainstorm + SPEC engineering rounds need to decide:

- Default value for `--flat-detection-method` when the user doesn't
  specify.
- Whether `--flat-detection-method=all` (run all three priors, combine
  classifications) is a meaningful option.
- Whether `--flat-detection-method=auto` (heuristic-driven selection
  based on dataset characteristics — number of stages, sample counts,
  coverage depth, etc.) is feasible.

**Approach.** Empirical — run a benchmarking exercise across diverse
fixtures (toy, twin-peak, full-twin, real DS1, user-supplied
`dev/share/res20260430-1/`) using each pipeline as prior, measure
flat-classification accuracy + downstream posterior quality. Decide
default + supported value set based on results.

**Cross-reference:** SPEC15.16 (cycle 15.7a) covers HMM `--norm-mode
chrom-median` validation-driven default switch. Item 2's empirical
benchmarking can leverage Phase 15's SPEC15.16 evaluation infrastructure
if reusable.

### Item 3 — Prior-phase output layout + file naming

**Background.** Where does the upstream prior write its outputs? Options:

- (a) Dedicated subdirectory `<out-dir>/01-prior-flat-detection/`
  (parallel to existing `<out-dir>/01-prior/` Phase 14 phasing
  convention). Outputs include `flat_samples.tsv`, possibly
  `amplicon_reliability.tsv` if reliability is also done upstream,
  diagnostic plots if any.
- (b) Existing `<out-dir>/01-prior/` directory with prior-flat-detection
  outputs co-mingled. Risk: confusion with the main prior-phase
  analysis outputs.
- (c) `<out-dir>/00-shared-prior/` — emphasizes that the upstream prior
  is shared across all downstream pipelines. Possibly clearest naming.

Decision deferred to brainstorm rounds. The output_layout.py + Finding 5
of SOUP Item 6 (CROSS_PIPELINE_UNIFICATION_SOUP.md) overlap here — the
layout dict's HMM-specific keys gap (Finding 5) interacts with this
SOUP's prior-phase layout design.

### Item 4 — Reliability scoring upstream too?

**Background.** SPEC15.6 deliverable 5 specifies a cross-pipeline
identical reliability-scoring sidecar (`amplicon_reliability.tsv`).
The same architectural argument that flat-sample detection should be
upstream applies to amplicon reliability: each pipeline's reliability
scores depend on its own normalization context, and that context can
be biased by stage-1 anchor pollution.

**Scope question for graduation time:** does this SOUP cover both
flat-sample detection AND amplicon reliability scoring as upstream
work, or just flat-sample detection?

**R1 prior on the question (orchestrator note 2026-04-30):** likely
both should be upstream, since they share inputs (per-locus tests
informed by `summit_rcn`) and downstream consumers (posterior
stage-anchoring decisions consume both). Defer to brainstorm rounds
for confirmation.

### Item 5 — Backward-compatibility + opt-out semantics

**Background.** Existing users run onionskin without
`--flat-detection-method`. The SOUP needs to decide:

- Does the upstream prior become the DEFAULT, with `--flat-detection-method=off`
  as opt-out (user opts out of upstream flat-detection)?
- Or does the upstream prior require an EXPLICIT opt-in
  (`--flat-detection-method=hmm` etc.), with default behavior unchanged
  from current per-pipeline flat-detection?

Likely default-on with opt-out IF the empirical benchmarking (Item 2)
shows unambiguous improvement; otherwise opt-in. Defer to brainstorm
rounds.

---

## Known interactions with other phases / SOUPs

- **`multi-agent/plans/next/GECKO_SOUP.md`** — GC-aware chrom-median
  normalization. **FLAT_SOUP probably graduates AFTER GECKO_SOUP** so
  the upstream flat-detection prior can use GC-aware chrom-median as
  its normalization mode. If GECKO graduates first, the upstream
  prior gets the better normalization for free; if FLAT_SOUP graduates
  first, the upstream prior uses plain chrom-median and a future GECKO
  graduation would upgrade it. Either order is valid; the GECKO-first
  order is preferred for reusing GC-aware infrastructure.

- **`multi-agent/plans/next/CROSS_PIPELINE_UNIFICATION_SOUP.md`** —
  Item 6 (cross-pipeline feature parity / silent no-ops reckoning)
  has overlap with this SOUP at:
  - **Finding 5** (output layout dict Growth-anchored keys; SOUP Item 6b)
    — the upstream-prior phase needs HMM-specific layout keys to write
    its outputs cleanly; resolving Finding 5 helps this SOUP.
  - **Finding 3** (per-pipeline flat-sample state) — Phase 15 cycle
    15.4a-S2 is fixing this in-cycle, but the upstream-prior architecture
    obviates the per-pipeline-state question entirely (one upstream
    decision feeds all pipelines).
  - **Item 6c meta-deliverable** (recurring parity audit) — flat-sample
    upstream-detection is the kind of architectural fix that the
    recurring sweep will surface as "the right way to do this" once
    the per-pipeline-state hack accumulates enough friction.

- **`multi-agent/plans/next/SUMMIT_SOUP.md`** — limited overlap. Summit
  refinement methodology is orthogonal to flat-sample detection.

- **`multi-agent/tracking/KNOWN_ISSUES.md` `[ISSUE:2026-04-30:3]`** —
  Phase 15 cycle 15.4a-S2 in-cycle defect-fix that this SOUP
  architecturally subsumes when it graduates.

- **`multi-agent/tracking/KNOWN_ISSUES.md` `[ISSUE:2026-04-30:4]`** —
  Phase 15 cycle 15.4a-S2 in-cycle HMM-posterior unit work, which
  also gets simplified once the upstream-prior architecture lands
  (the per-pipeline-flat-sample-auto-switch refactor in cycle 15.4a-S2
  Stage B.5 is a tactical patch; the upstream-prior makes that refactor
  obsolete).

- **`multi-agent/plans/PHASE15_SPEC.md` SPEC15.16** (cycle 15.7a) —
  HMM chrom-median default switch. SPEC15.16's empirical benchmarking
  infrastructure is reusable for FLAT_SOUP Item 2's pipeline-quality
  decision.

---

## When to start

When the per-pipeline flat-detection patches accumulate enough friction
that the upstream-prior architecture is more efficient than continuing
patch-by-cycle. Concrete graduation triggers:

- A future phase's audit-implement-reaudit cycle catches a third
  flat-detection-related defect after Phase 15's `[ISSUE:2026-04-30:3]`
  + `[ISSUE:2026-04-30:4]` are closed.
- The per-pipeline `_flat_sample_auto_switch_{growth,rms,hmm}` refactor
  from cycle 15.4a-S2 Stage B.5 develops additional per-pipeline state
  divergences, signaling that the per-pipeline approach is fundamentally
  unstable.
- `GECKO_SOUP.md` graduates (GC-aware chrom-median is implemented) and
  the user wants to leverage GC-aware normalization for the upstream
  flat-detection prior.
- The user explicitly directs that flat-detection should move upstream
  for cleaner architecture.

**Order preference:** GECKO_SOUP first (GC-aware chrom-median lands), then
FLAT_SOUP (upstream-prior uses GC-aware chrom-median as default
normalization for the prior phase). Reverse order works but loses the
GC-aware normalization benefit on the prior phase.

Until any of those triggers, Phase 15's discrete in-cycle flat-detection
fixes (cycle 15.4a-S2 Stage B.1's chrom-median path; Stage B.5's
per-pipeline auto-switch refactor) are doing the right work — closing
the most user-visible defects. This SOUP captures the broader architectural
fix for when it makes sense to go bigger.

---

## Open brainstorm questions (not yet answered)

1. **Default value for `--flat-detection-method`.** Three candidates: `hmm`
   (theme-aligned with Phase 15's HMM-completeness focus), `rms` (computationally
   cheaper than HMM; less sensitive but possibly sufficient for flat-detection
   purposes), `auto` (heuristic-driven). Empirical benchmarking (Item 2) decides.

2. **Pipeline-as-prior accuracy comparison.** Which pipeline produces the
   most accurate flat-classifications? HMM has the richest state-path
   information but is computationally expensive; RMS has summit-aware
   detection; Growth has the simplest stage-progression model. Run
   benchmarks across diverse fixtures.

3. **`--flat-detection-method=all` semantic.** When all three pipelines
   run as priors and produce different flat-classifications, what's the
   combination rule? Majority vote (2 of 3 pipelines agree)? Conservative
   (any pipeline classifies as flat → flat)? Weighted by per-pipeline
   accuracy from Item 2 benchmarking?

4. **Cache vs re-run for same-pipeline-as-prior-and-main case.** When
   user runs `--pipelines hmm --flat-detection-method hmm`, can the
   upstream prior + main HMM run share a single execution by caching
   flat-samples mid-run? Or are they kept as separate runs for
   architectural cleanliness?

5. **Reliability scoring upstream too?** (Item 4 above.) Does this
   SOUP's scope expand to include upstream amplicon-reliability
   scoring, or stay flat-sample-detection-only?

6. **Output layout naming.** (Item 3 above.) `01-prior-flat-detection/`,
   co-mingled with `01-prior/`, or `00-shared-prior/`?

7. **Backward-compatibility default.** (Item 5 above.) Default-on with
   `--flat-detection-method=off` opt-out, or default-off with explicit
   opt-in? Empirical benchmarking outcomes inform.

8. **GECKO_SOUP graduation order coupling.** This SOUP prefers GECKO
   graduating first. If GECKO_SOUP graduates much later than this SOUP
   is ready to graduate, do we proceed with plain chrom-median for the
   upstream prior and upgrade later, or hold this SOUP for GECKO?
   User direction needed at graduation time.

These are starter questions. More will emerge during brainstorming rounds.
