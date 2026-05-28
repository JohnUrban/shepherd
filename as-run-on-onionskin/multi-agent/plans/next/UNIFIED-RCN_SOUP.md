# UNIFIED-RCN — SOUP

**Stage:** SOUP (pre-BRAINSTORM scratchpad in `multi-agent/plans/next/`).
**Theme:** Unified per-sample smoothed RCN intermediates shared between
pipelines (long-horizon idea; pipelines currently differ in how they
compute and consume RCN-like signals, so unification was previously
audited as not-yet-feasible).
**Lifecycle:** Promotes to a per-phase BRAINSTORM in `multi-agent/plans/`
when (and if) the orchestrator brings it onto the stage. The user has
flagged this as **future-phase, not Phase 15** — needs more groundwork
before unification becomes the right choice.
See `multi-agent/AGENT_CONVENTIONS.md § Future-phase planning surfaces`
and `multi-agent/workflows/phase-development-system_PDS-v2.md` for the
SOUP → BRAINSTORM → SPEC lifecycle.

> **Maintenance:** SOUP files are intentionally early-stage and
> disorganized. Update freely as ideas surface; do not introduce
> SELF-references to phase numbers (cross-references to actually-closed
> phases ARE allowed as historical anchors — see `AGENT_CONVENTIONS.md
> § Future-phase planning surfaces`).
>
> **Provenance:** Created during Phase 15 soup-to-brainstorm transfer
> (Round 1 follow-up, 2026-04-28) in response to user direction in
> `multi-agent/plans/PHASE15_FEEDBACK.md` Q10 answer. The user noted
> that an earlier audit concluded unification "would not make sense
> because the pipelines are doing things differently," and that if
> unification is pursued at all, it should be in a future phase. This
> SOUP captures the idea so it doesn't get lost.

---

## Source idea (from `tracking/BRAINSTORM.md [2026-04-16]`)

> **`tracking/BRAINSTORM.md [2026-04-16] Long-horizon: unified per-sample smoothed RCN intermediates shared between pipelines`**
>
> The HMM step-5 emits per-stage aggregate smoothed RCN bedGraphs
> (median algorithm). The idea was to have growth, RMS, and HMM
> all share a unified per-sample smoothed RCN intermediate that any
> pipeline could consume.

The full content of the [2026-04-16] entry in `tracking/BRAINSTORM.md`
identifies three blockers (lines ~30–55 of `tracking/BRAINSTORM.md`,
verified during Phase 15 transfer 2026-04-28):

1. **Shape mismatch (hard blocker):** HMM steps 2–5 operate on per-stage group aggregates (median across samples per stage), not on per-sample data. The current HMM step-5 does not emit per-sample smoothed outputs.
2. **Masking semantic difference (hard blocker):** HMM step-3 removes zero bins from the bedGraph (math constraint of the ratio step in ref-stage normalization). Other pipelines do not remove zero bins this way.
3. **Algorithm difference (medium concern):** HMM uses `local_median_smooth_bedgraph()` for step-5 smoothing; growth and RMS use different smoothing algorithms. A unified intermediate would need to settle on one or expose multiple variants.

The proposed direction was: HMM step-5 (or a new step-5b) would need
to also emit per-sample smoothed intermediates for growth/RMS to
inherit. This would close the shape-mismatch blocker but leave the
masking-semantic and algorithm-difference blockers open.

## Audit conclusion (per user note in Phase 15 Q10, 2026-04-28)

The user noted: *"I think we actually concluded in an audit at one
point that this would not make sense because the pipelines are doing
things differently. Nonetheless, if we pursue this it would be in a
future phase. This idea can be added to its own SOUP file with all
appropriate references to other files."*

The audit conclusion is consistent with the three blockers above —
unification is hard precisely because the pipelines have legitimate
divergent needs (HMM's ratio-step zero-bin removal is a math
constraint, not an arbitrary design choice; growth and RMS need
different smoothing surfaces for their downstream analyses).

## Why this SOUP exists

Even though unification was audited as not-yet-feasible, the idea is
worth preserving because:

- Some Phase 15 work (BRAIN15.6 HMM parallel child pipeline, with
  per-sample step-3-through-11 outputs under `indiv_samples/`) will
  produce per-sample smoothed RCN intermediates for HMM. After
  Phase 15, the shape-mismatch blocker (1) is partially relaxed for
  the HMM side.
- BRAIN15.21 (lift shared gap mask to pre-pipeline) addresses the
  masking-semantic concern — at least for gap-bin removal, all three
  pipelines can inherit a shared mask. The math-constraint zero-bin
  removal in HMM ref-stage stays separate.
- The algorithm-difference blocker (3) is the residual. A future
  phase that addresses it (e.g., by allowing a configurable
  smoothing-algorithm flag at controller level, or by accepting
  multiple smoothing variants in parallel) would unblock unification.

So the right time to revisit this idea is **after** Phase 15 lands
its parallel-child-pipeline + shared-gap-mask work and the residual
blocker becomes the primary one.

## Cross-references

- `tracking/BRAINSTORM.md [2026-04-16] Long-horizon: unified per-sample smoothed RCN intermediates shared between pipelines` — the originating idea.
- Phase 15 BRAIN15.6 — HMM parallel child pipeline (per-sample steps 3–11 indiv_samples/ outputs).
- Phase 15 BRAIN15.21 — Lift shared gap mask + missingness diagnostic to pre-pipeline step.
- Earlier audit concluding "doesn't make sense yet" — user-recalled in `PHASE15_FEEDBACK.md` Q10 answer (2026-04-28); exact audit location TBD if needed.

## Open questions for whoever picks this up later

1. Has the residual algorithm-difference blocker (3) become tractable since the original audit? E.g., are growth and RMS now using compatible-enough smoothing approaches that a single unified surface could serve both?
2. Does HMM's per-sample step-5 output (post-BRAIN15.6) get consumed by anything outside HMM, or does HMM still use only the per-stage aggregate? If outside consumers exist, that's evidence the unification door is opening.
3. Is there a smaller-scope unification that DOES make sense — e.g., a unified per-sample smoothed-RCN cache used only at the post-call annotation stage (not at pipeline-internal computation)?
