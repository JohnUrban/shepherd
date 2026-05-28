# GC-Aware Chrom-Median Normalization — SOUP

**Stage:** SOUP (pre-BRAINSTORM scratchpad in `multi-agent/plans/next/`).
**Theme:** GC-aware extension to chrom-median normalization — adds a per-GC-class
dimension (integer 0-100) to the chrom-median normalization machinery so each
genomic bin is normalized to the median of bins in its OWN GC class on the same
chromosome rather than the whole-chromosome median.
**Lifecycle:** Promotes to a per-phase BRAINSTORM in `multi-agent/plans/`
when the orchestrator brings it onto the stage and assigns a phase number.
See `multi-agent/AGENT_CONVENTIONS.md § Future-phase planning surfaces`
and `multi-agent/workflows/phase-development-system_PDS-v2.md` for the
SOUP → BRAINSTORM → SPEC lifecycle.

**Created:** 2026-04-30 by Claude Code 2.1.117 (claude-opus-4-7), per user
direction during Phase 15 cycle 15.4a-S2 amendment authorization. Triggered
by user observation that cycle 15.4a SPEC15.6's flat-detector + reliability
scorer defects on real data (`[ISSUE:2026-04-30:3]`) point at deeper
normalization-strategy questions: ref-stage normalization can be polluted by
amplified samples in the stage-1 manifest; chrom-median normalization avoids
that pollution but loses the benefit of subtracting out technical biases (GC
bias, sequence-prep bias, etc.). GC-aware chrom-median is a candidate
synthesis that may capture the best of both.

**Status:** SOUP-stage. Not started. Out of scope for Phase 15 (cycle
15.4a-S2 explicitly defers GC-aware work to this SOUP per the cycle's scope
boundary lock; cycle 15.4a-S2 tunes existing SPEC15.6 deliverables to ground
truth, not normalization-strategy redesign).

---

## Why a dedicated GC-aware-chrom-median phase

The user surfaced the underlying tension at cycle 15.4a-S2 amendment time
(2026-04-30, verbatim):

> *"The thing we miss out on when we don't do ref-stage norm is accounting
> for all the technical biases that change the shape of the data — various
> GC biases and sequence prep biases, etc. Ref-stage normalization helps
> 'subtract' those out, but chrom-median does not. Perhaps we could add a
> GC-aware mode for chrom-median that uses the chromosome-specific median
> for each GC class. We can do integer GC classes from 0-100. Then instead
> of one chromosome-specific median there are 101, 1 for each GC class.
> Then bins are normalized to the GC class they fit into..."*

The two normalization paths in the live tree have complementary weaknesses:

- **Ref-stage normalization** subtracts technical biases by dividing each
  sample's stage-K signal by the per-bin median across stage-1 samples
  (the "reference stage"). PROBLEM: if any stage-1 sample is already
  amplified (e.g., SAG40 in `[ISSUE:2026-04-30:3]`), the stage-1 median is
  polluted. Other stage-1 samples appear below baseline at amplified
  positions (the SAG41 negative-bump artifact), and posterior-phase
  ref-stage selection inherits the contamination.
- **Chrom-median normalization** divides each bin by the per-chromosome
  median of that sample's signal. PROBLEM: it doesn't account for GC bias,
  sequence-prep bias, or other technical biases that change signal shape
  across bins of different GC content. Bins with high or low GC content
  systematically deviate from the chrom-wide median in ways that have
  nothing to do with biology.

GC-aware chrom-median is a candidate hybrid: keep chrom-median's resilience
to stage-1 pollution while restoring per-GC-class normalization that
captures the technical bias structure ref-stage normalization gets for
free.

---

## What this SOUP is about

A new normalization mode (CLI flag, output schema, validation) that:

1. Bins each sample's signal into 101 GC classes (integer 0-100, one
   per percentage point of GC content) per chromosome.
2. Computes per-chromosome per-GC-class medians (101 medians per
   chromosome).
3. Normalizes each bin by dividing by the median of its GC class on the
   same chromosome, rather than by the whole-chromosome median.

Output: the same RCN-style (Read Copy Number) output schema as existing
chrom-median, with the per-GC-class denominator substitution invisible to
downstream consumers (APS, summit refinement, timing, etc.).

The phase covers algorithm, CLI surface, output schema, validation against
ground-truth data, and a probable harvest from the external
`geckocorrection` codebase (loaded at the top level of onionskin, gitignored
parallel to PuffStep — see Item 2).

---

## Items parked here

### Item 1 — GC-aware chrom-median mode design

**Background.** Cycle 15.3a SPEC15.4 landed Bayesian ODW System with chrom-median
support; cycle 15.7a SPEC15.16 covers HMM `--norm-mode chrom-median`
validation-driven default switch. The existing chrom-median path is
GC-blind — it computes a single per-chromosome median per sample.
GC-aware extension adds a 101-bin GC-class dimension.

**Open design questions:**

- **Bin GC-class assignment.** Each bin needs a GC class. Computed from the
  reference genome's GC content over the bin's bp range (rounded to integer
  percentage). Cache per-chromosome bin-to-GC-class map at run start.
- **Bin-count thresholds per GC class.** Some GC classes will have very
  few bins on a given chromosome (extreme GC values). What's the minimum
  bin count per class before the median becomes meaningful? Fall back to
  class-pooling (e.g., merge GC classes within ±k of underpopulated bins)
  or to whole-chromosome median for sparse classes? Validation-driven
  default.
- **Cross-chromosome behavior.** Should the GC-class median be computed
  per-chromosome (current SOUP framing) or genome-wide (pool across
  chromosomes within a GC class)? Per-chromosome preserves chromosome-
  specific behavior; genome-wide gives more bins per class for stability.
  Hybrid possible: per-chromosome for well-populated GC classes,
  genome-wide pooling for sparse classes.
- **CLI surface.** New flag like `--norm-mode chrom-median-gc` or extend
  existing `--norm-mode chrom-median` with a `--gc-aware on|off` modifier.
  Pipeline-specific overrides (`--hmm-norm-mode`, `--growth-norm-mode`,
  `--rms-norm-mode`) inherit the same option set.
- **Output schema.** Internal bin-to-GC-class map saved as a sidecar TSV?
  Per-GC-class median tracks emitted? Or is the GC-aware normalization
  fully transparent to downstream consumers (ref-stage-equivalent output
  schema)?

**Estimated scope.** Multi-cycle theme. Could justify a phase-group
structure when it graduates: Phase X cycle 1 = mode design + per-chromosome
implementation; Phase X cycle 2 = sparse-class fallback + cross-chromosome
pooling; Phase X cycle 3 = validation against ground-truth + default-flip
evaluation.

---

### Item 2 — Harvest from external `geckocorrection` repo

**Background.** User has loaded the external `geckocorrection` codebase at
the top level of onionskin (gitignored, parallel to how PuffStep was
handled before its useful bits got harvested into onionskin proper). The
geckocorrection repo implements GC-aware correction patterns the user has
already validated; this Item harvests what's useful and lets the external
repo go.

**Pattern (parallel to PuffStep handling):**

1. External repo loaded at top level of onionskin (gitignored).
2. Future-phase cycles dive into `geckocorrection/` and identify reusable
   algorithms / parameter tunings / test fixtures.
3. Useful bits get incorporated into `onionskin_core/` with provenance
   notes citing the geckocorrection origin.
4. Once enough has been harvested, the external repo is no longer needed —
   onionskin's GC-aware chrom-median is the canonical home.

**Probable harvest targets** (R1 audit at graduation time will enumerate):

- Bin GC-content computation from reference genome.
- Per-GC-class median computation + sparse-class fallback strategies.
- Validation fixtures + test data showing GC-aware correction's effect
  vs naive correction on real data.
- Any plot / diagnostic surface that visualizes per-GC-class behavior.

**Estimated scope.** Half-cycle of audit work + half-cycle of integration
work, per useful-bit. Total scope depends on what geckocorrection contains
that survives the audit's "is this useful in onionskin's context" filter.

---

### Item 3 — Optional dual-mode (ref-stage AND chrom-median AND chrom-median-GC-aware) outputs for diagnostic comparison

**Background.** Cycle 15.4a-S2's diagnostic-question (e) asks whether the
controller should emit BOTH ref-stage AND chrom-median outputs side-by-side
for comparison. This SOUP's GC-aware extension adds a third option.
Diagnostic surfaces benefit from being able to compare all three on the
same data — the user-supplied evidence at `dev/share/res20260430-1`
showed how the choice between normalization modes affects the SAG40 / SAG41
II/9A artifact pattern.

**Possible designs:**

- **Tri-mode flag** — `--norm-modes ref-stage,chrom-median,chrom-median-gc`
  emits all three normalized output trees side-by-side. Each downstream
  step (APS, summit refinement, timing) consumes the user-selected mode
  via the existing `--norm-mode` flag, but the alternative trees stay on
  disk for comparison.
- **Diagnostic-only mode** — primary normalization picks one; the others
  emit lightweight diagnostic-tracks-only (no full APS/summit/timing
  recompute). Cheaper, less complete.
- **Switch + sidecar comparison report** — one canonical normalization
  used; sidecar report shows what would have happened under the other
  modes on key positions (e.g., reliability-flagged amplicons).

**Dependencies.** Requires Item 1 to be substantially complete (the
GC-aware mode must exist before it can be one of the three).

**Estimated scope.** Sub-cycle scoped — fits cleanly inside Item 1's
graduation phase.

---

### Item 4 — Cross-pipeline reach for GC-aware normalization

**Background.** Cycle 15.7a SPEC15.16 covers HMM-only `--norm-mode chrom-median`
validation-driven default switch. Growth + RMS have their own normalization
paths. GC-aware extension needs to apply cross-pipeline (Growth + RMS + HMM)
per the recurring `feedback_cross_pipeline_parity_blindspot.md` direction.

**What's needed.** Audit each pipeline's normalization code path and
identify where the GC-aware substitution lands. Likely in:

- `onionskin_core/refinement.py` and/or `onionskin_core/odw.py` for the
  shared normalization primitive (if one exists post-Phase-15).
- Each pipeline's engine boundary for the per-pipeline-specific norm-mode
  flag wiring.
- New shared module like `onionskin_core/normalization.py` if the
  refactor consolidates currently-scattered normalization logic.

**Connection to other future-phase work.** This Item touches the same
cross-pipeline structural divergence pattern captured at
`multi-agent/plans/next/CROSS_PIPELINE_UNIFICATION_SOUP.md`. The GC-aware
implementation could either land alongside the cross-pipeline structural
unification work (single phase) or as a sibling phase. Decision belongs
to graduation time.

---

## Known interactions with other phases / SOUPs

- **Phase 15 cycle 15.4a-S2** (mid-phase amendment 2026-04-30) tunes
  existing SPEC15.6 flat-detector + reliability-scorer to ground-truth
  data. The cycle's diagnostic question (d) — *"Should chrom-median be
  default for all pipelines?"* — partially overlaps with this SOUP's
  scope but cycle 15.4a-S2 explicitly DEFERS GC-aware work here. If
  cycle 15.4a-S2 surfaces a chrom-median-as-default change for all
  pipelines, this SOUP's eventual phase consumes that as its starting
  default + adds the GC-aware extension.

- **Phase 15 cycle 15.7a SPEC15.16** — HMM `--norm-mode chrom-median`
  validation-driven default switch. Lands HMM-only chrom-median as a
  durable default if validation passes. This SOUP's eventual phase builds
  on that.

- **`multi-agent/plans/next/CROSS_PIPELINE_UNIFICATION_SOUP.md`** —
  cross-pipeline structural unification. Item 4 of this SOUP (cross-pipeline
  GC-aware reach) overlaps with that SOUP's broader unification theme.
  May graduate as siblings or merge.

- **External `geckocorrection` repo** at top-level of onionskin
  (gitignored, parallel to PuffStep). Item 2 harvests; eventually the
  external repo is parted with.

- **`multi-agent/tracking/KNOWN_ISSUES.md` `[ISSUE:2026-04-30:3]`** —
  the trigger for cycle 15.4a-S2 + indirectly the trigger for this SOUP.
  When this SOUP graduates and lands GC-aware chrom-median, that issue's
  exit condition (or future analogous issues) may be resolved by the
  better normalization rather than by tuning the existing flat-detector
  thresholds.

---

## When to start

After Phase 15 closes (cycle 15.10a + Final Overseer wrap-up). Concrete
triggers that would justify graduation:

- Cycle 15.4a-S2's tuning + threshold calibration of existing SPEC15.6
  deliverables produces unsatisfactory results on ground truth, suggesting
  the underlying normalization strategy needs upgrading rather than the
  detector/scorer thresholds.
- Cycle 15.7a SPEC15.16 validation flips HMM default to chrom-median, and
  user direction is to extend the same flip to Growth + RMS.
- User explicitly directs that chrom-median's GC-blindness is the limiting
  factor for cross-pipeline parity.
- A new dataset or chromosome with extreme GC content variability surfaces
  GC-bias artifacts the existing chrom-median can't compensate for.

Until any of those triggers, Phase 15's discrete work + cycle 15.4a-S2's
threshold calibration are doing the right work. This SOUP captures the
deeper normalization-strategy redesign for when it makes sense to commit.

---

## Open brainstorm questions (not yet answered)

1. **GC class granularity.** Integer percentage classes (0-100) is the
   user-proposed default. Could fewer classes (e.g., 20 buckets at 5%
   width) be more robust on sparse data? Could finer (decimal classes)
   help on extreme-GC chromosomes? Validation-driven.

2. **Sparse-class fallback strategy.** Fold sparse GC classes into
   neighbors? Pool across chromosomes? Fall back to chrom-wide median?
   Each has tradeoffs — empirical question.

3. **Per-chromosome vs genome-wide GC-class medians.** Per-chromosome
   captures chromosome-specific bias; genome-wide gives more bins per
   class for stability. Hybrid possible. Need ground-truth comparison.

4. **Should the SOUP merge with `CROSS_PIPELINE_UNIFICATION_SOUP.md`?**
   Item 4 (cross-pipeline reach) overlaps. The two SOUPs may graduate
   as siblings or merge into a single normalization-strategy-unification
   phase depending on scope-sizing at graduation time.

5. **Validation harness.** What ground-truth fixtures show GC-aware
   improvement most clearly? Likely the same `dev/share/res20260430-1` /
   `tests/full_chrom_training_data/amplicons.by-eye.bed` surfaces cycle
   15.4a-S2 uses, plus targeted GC-bias fixtures from `geckocorrection`'s
   test suite.

6. **Backward-compatibility for existing CLI users.** GC-aware should be
   opt-in initially (new flag), validated against existing outputs, then
   considered as a default-flip candidate per the same protocol that
   SPEC15.16 uses for the HMM chrom-median default switch.

These are starter questions. More will emerge during brainstorming
rounds.
