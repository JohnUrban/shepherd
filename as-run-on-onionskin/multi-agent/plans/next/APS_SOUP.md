# APS — SOUP

**Stage:** SOUP (pre-BRAINSTORM scratchpad in `multi-agent/plans/next/`).
**Theme:** APS (Amplicon Posterior Score) — clustering, features, plots, notebooks, parity across pipelines, deferred follow-ups.
**Lifecycle:** Promotes to a per-phase BRAINSTORM in `multi-agent/plans/`
when the orchestrator brings it onto the stage and assigns a phase number.
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
> `multi-agent/plans/PHASE15_FEEDBACK.md` Q11 answer ("we should start
> making an APS_SOUP.md to start tracking all APS related ideas"). The
> Phase 15 BRAINSTORM also has its own APS-related entries (BRAIN15.14
> `--aps-area-excess-floor`, BRAIN15.16 APS Universal-in-spirit,
> BRAIN15.17 APS plots/parity, BRAIN15.19 clustering defaults, BRAIN15.20
> SAPS) — those are Phase 15 scope; this SOUP catches APS-related ideas
> that are NOT in Phase 15 scope (deferred IBM items, candidate future
> work, longer-horizon ideas).

---

## What this SOUP is about

Catalogue of APS-related ideas that span beyond the current phase's
scope. APS work touches multiple analysis surfaces (cluster scoring,
posterior ordering, shape features, dimensionality reduction,
external feature ingestion, plots/notebooks) and accumulates ideas
that don't all fit in any single phase. This SOUP is the home for
those ideas while they wait for the right phase to formalize them.

## Deferred items from `INTENDED-BUT-MISSED-PRIOR-TO-14.md` (per Phase 15 Q11 disposition, 2026-04-28)

The user's Q11 disposition for the IBM-C6 / IBM-C7 cluster (Phase 15
FEEDBACK round 1) was: most are deferred from Phase 15. Captured here
for future-phase planning. Each entry preserves the IBM-file confidence
rating and the source citation.

### IBM-C6A — APS order stability (bootstrap diagnostics)

- **Origin:** ROADMAP Priority 5.3 ✗ DEFERRED (post-HMM); design notes in `tracking/BRAINSTORM.md [2026-04-07]`.
- **Status:** Deferred from Phase 15.
- **Confidence:** medium (per IBM file).
- **Concept (from IBM file):** bootstrap-style diagnostics for the stability of APS-derived cluster ordering — i.e., does shuffling/resampling produce consistent orderings or noisy ones?
- **Phase placement:** future phase (not Phase 15).

### IBM-C6D — Timing-guided active-stage weights for summit estimation

- **Origin:** ROADMAP Priority 5.9.3 ✗ DEFERRED (post-HMM).
- **Status:** Deferred from Phase 15.
- **Confidence:** medium (per IBM file).
- **Concept (from IBM file):** timing-guided `--stage-weight-mode timing`, center-weighted active window, per-stage parabola validity as data-driven weight proxy. Depends on timing module stability and HMM integration for active-window estimates.
- **Phase placement:** future phase. May depend on Phase 15 outcomes (HMM completeness work + timing module stability).

### IBM-C6E — Iterative summit ↔ timing convergence (EM-like)

- **Origin:** ROADMAP Priority 5.9.4 ✗ DEFERRED (post-HMM).
- **Status:** Deferred from Phase 15.
- **Confidence:** medium (per IBM file).
- **Concept (from IBM file):** EM-like iteration between summit estimation and timing active-window. Depends on IBM-C6D.
- **Phase placement:** future phase, gated on IBM-C6D landing first.

### IBM-C6F — Prior/posterior profile similarity investigation

- **Origin:** ROADMAP Priority 5.9.6 ✗ DEFERRED (post-HMM).
- **Status:** Deferred from Phase 15. **User note:** "If something like this comes up during our work, it can be a side quest, and we can update its status."
- **Confidence:** medium (per IBM file).
- **Concept (from IBM file):** investigate similarity between prior (morphology-derived) and posterior (DNA-amplification-derived) developmental orderings. Needs a dataset with differing prior/posterior groupings.
- **Phase placement:** opportunistic side-quest if relevant data appears during Phase 15; otherwise future phase.

### IBM-C7A — APS dimensionality reduction / embedding (PCA, UMAP, diffusion)

- **Origin:** ROADMAP Priority 6.2 ✗ DEFERRED (post-HMM).
- **Status:** Deferred from Phase 15.
- **Confidence:** medium (per IBM file).
- **Concept (from IBM file):** dimensionality reduction over APS-feature vectors (PCA, UMAP, diffusion maps) to support visualization and embedding-based clustering.
- **Phase placement:** future phase.

### IBM-C9 — External signal-track ingestion for APS features (`--feature-track NAME:PATH`)

- **Origin:** ROADMAP Priority 5.6 ✗ DROPPED (replaced by `--feature-track` design); successor design preserved in `tracking/BRAINSTORM.md [2026-04-07] Additional signal tracks for APS clustering features (from Priority 5.6)`.
- **Status:** Deferred from Phase 15 (NOT in user's Q11 list but cross-referenced for completeness — IBM-C9 is the natural sibling of IBM-C7A for input-side APS feature engineering).
- **Concept:** generalized `--feature-track NAME:PATH` interface that lets users supply additional bedGraph/signal tracks as APS clustering features (RNA-seq, replication timing, mappability, etc.).
- **Phase placement:** future phase.

## Items requiring more investigation (Phase 15 follow-up promised in Q11)

### IBM-C6B — Better posterior ordering logic (scalar/dendrogram/hybrid)

- **Origin:** ROADMAP Priority 5.4 ✗ DEFERRED (post-HMM); `tracking/BRAINSTORM.md [2026-04-07]` reference (no dedicated detailed entry — IBM file's one-line description is the substantive scope statement).
- **Status:** **User said "might be of interest"** in Phase 15 Q11 (2026-04-28); promotion-to-Phase-15 decision pending. Summarized in Phase 15 Round 1 follow-up FEEDBACK.
- **Confidence:** medium (per IBM file).
- **Concept (from IBM file):** "Refine scalar / dendrogram / hybrid ordering relationship for posterior grouping." This refers to how APS posterior ordering is derived — currently a mix of scalar APS values and dendrogram-style hierarchical clustering, with a "hybrid" option that blends. The 5.4 deferred item proposed refining this relationship (e.g., principled weighting, alternative dendrogram metrics, ordering-stability checks).
- **Decision needed:** Whether to promote into Phase 15 BRAINSTORM as a new BRAIN entry, or leave in this SOUP for a later phase. The IBM file's one-line description is the only substantive content; deeper design work would be needed to spec.

### IBM-C6C — Stage-activity fold-change using refined summit (`timing.py` refactor) — PROMOTED to Phase 15 BRAIN15.29

- **Origin:** ROADMAP Priority 5.0.2 (carried forward from Phase 4) ✗ DEFERRED (post-HMM); `tracking/BRAINSTORM.md [2026-04-07]` reference; IBM file's two-line description.
- **Status:** **PROMOTED to Phase 15** as `BRAIN15.29` per Phase 15 FEEDBACK Q12 user answer (2026-04-28). The user's Q12 answer included a substantial new design (the four-step one-pass summit-then-timing-then-updated-summit-then-updated-timing convergence, distinct from EM-iterative IBM-C6E) that goes well beyond the IBM file's two-line scope. See `multi-agent/plans/PHASE15_BRAINSTORM.md` BRAIN15.29 for the full Phase 15 design.
- **Action taken:** No further work in this SOUP; future-phase planning surface for this item is now BRAIN15.29 in Phase 15 BRAINSTORM. Closeout cleanup in `tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` IBM-C6C entry — flag in a separate cleanup pass as RESOLVED-via-promotion-to-BRAIN15.29.

### IBM-C7B — Shape-aware APS clustering — RESOLVED-by-existing-and-by-BRAIN15.27

- **Origin:** ROADMAP Priority 6.3 ✗ DEFERRED (post-HMM); `tracking/BRAINSTORM.md [2026-04-07]` reference; IBM file's one-line description: "Cluster using amplicon morphology, not just integrated magnitude. Post-HMM."
- **Status:** **RESOLVED.** The original Priority 6.3 one-line scope is fulfilled by the existing `--aps-feature shape` (Priority 6.1, v0.3.32; per-bin RCN-1 profile feature matrix with `1/√n_bins_in_locus` weighting). Investigation (Phase 15 Q13 follow-up, 2026-04-28) found no documented "richer" plans for 6.3 anywhere in `tracking/BRAINSTORM.md`, ROADMAP, or archived planning surfaces — the "see BRAINSTORM.md" pointer in the deferred 6.3 entry never resolved to a substantive design. The richer interpretations the user surfaced during Phase 15 transfer follow-up — parametric shape fits / composite morphology / sample-level integrators / Layer-1 amplicon-PCA / Layer-2 sample-PCA / Layer-3 matrix-PCA — are now captured in **BRAIN15.27** (Composite multi-feature APS clustering modes with three-layer PCA composition), so any "richer than `--aps-feature shape`" intent that the original 6.3 may have implied is in scope for Phase 15 implementation under BRAIN15.27.
- **Why RESOLVED:** The one-line scope was matched by 6.1's implementation; Phase 15's BRAIN15.27 covers the richer extensions that anyone could have plausibly meant by "shape-aware APS clustering."
- **Cleanup actions taken (Phase 15 transfer follow-up, 2026-04-28):** ROADMAP.md Priority 6.3 marker flipped to ✓ DONE per `multi-agent/project_context/DECISIONS.md` 2026-04-28 entry. IBM-C7B entry in `tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` to be marked RESOLVED with closeout pointer to 6.1 + BRAIN15.27 (deferred to a separate cleanup pass — flagged in Phase 15 FEEDBACK).

### `--aps-rank-by` future choices — summit, width, shape (migrated from KNOWN_ISSUES.md 2026-04-28 per user direction)

**Formerly lived in `KNOWN_ISSUES.md` and was referred to as `[ISSUE:2026-04-22:1]`.**

- **Origin:** Help string for `--aps-rank-by` was expanded by Phase 14 Supplemental cycle 14-S6 (2026-04). The flag currently accepts only `area`. The flag was designed to support multiple ranking strategies; `summit`, `width`, and `shape` are the planned additions.
- **Status:** Migrated to APS_SOUP.md per user direction in Phase 15 FEEDBACK Round 3 (2026-04-28): *"[ISSUE:2026-04-22:1] may be important for posterior ordering, but not necessarily posterior groups / posterior manifests. If that is the case, it can go in APS_SOUP.md. If it does affect posterior manifests, then it should be part of Phase 15."* User confirmed in chat that posterior-ordering-only effect → APS_SOUP placement.
- **Why APS_SOUP, not Phase 15:** The flag affects APS-derived sample ordering (which affects posterior ordering displays + analyses) but does NOT affect posterior amplicon groupings/manifests (which are determined by clustering on the APS feature matrix, independent of the ranking choice for displays). Per the user's "if it affects manifests → Phase 15; if it affects ordering only → APS_SOUP" rule, this is APS_SOUP material.
- **Verbatim from KNOWN_ISSUES (preserved here for content-completeness):** *"`--aps-rank-by` currently accepts only `area`. The flag was designed to support multiple ranking strategies; `summit`, `width`, and `shape` are the planned additions. Exit condition: Additional choices implemented and wired; KNOWN_ISSUES entry removed at that time."*
- **Phase placement:** future phase. Could pair naturally with BRAIN15.27 (composite multi-feature APS clustering modes — same set of features: summit, width, shape) but the ranking-vs-clustering distinction matters: BRAIN15.27 changes what feature matrix clustering operates on; this `--aps-rank-by` extension changes which feature drives the post-clustering ranking display. They could land separately or be coordinated; SPEC engineering at promotion-time will decide.
- **Open questions for whoever picks this up later:**
  1. Does `--aps-rank-by shape` need a single scalar score per sample (since ranking is 1-D), or does it operate on a derived quantity (e.g., shape PCA PC1)?
  2. Should `--aps-rank-by` choices coordinate with `--aps-feature` choices (e.g., warn if user picks `--aps-feature area --aps-rank-by shape`)?
  3. Are there pipeline-specific override flags needed (`--growth-aps-rank-by`, etc.)?
- **Cross-reference:** the post-prompted "post-prefixed-flag with `--<pipeline>-` override pattern" decision is already established (see existing `--growth-peak-summary` pattern). Any new ranking choice should follow that pattern.

---

## Convention applied for the migration above

This is the first application of the **KNOWN_ISSUES → new home migration with provenance annotation** convention codified in `multi-agent/AGENT_CONVENTIONS.md § KNOWN_ISSUES.md` (added 2026-04-28). The pattern: (1) restructure issue content for the new home; (2) add `**Formerly lived in `KNOWN_ISSUES.md` and was referred to as `[ISSUE:YYYY-MM-DD:N]`.**` provenance line; (3) delete from KNOWN_ISSUES.md (no stub).

## Existing APS-related Phase 15 BRAIN entries (cross-reference, not in this SOUP's scope)

For completeness, APS-related ideas that ARE in Phase 15 scope live in `multi-agent/plans/PHASE15_BRAINSTORM.md`:

- BRAIN15.14 — `--aps-area-excess-floor` analytical testing
- BRAIN15.16 — APS Universal-in-spirit re-framing forward-extension
- BRAIN15.17 — APS plots / parity catalog (HMM should offer all analyses/outputs/plots/notebooks of any other pipeline; create a master APS catalog file)
- BRAIN15.19 — Clustering defaults finalization
- BRAIN15.20 — SAPS implementation (state-path APS)
- BRAIN15.27 — Composite multi-feature APS clustering modes with three-layer PCA composition (added Phase 15 Q13 follow-up 2026-04-28)
- BRAIN15.28 — `summit_excess` → `summit_rcn` rename + RCN-as-default migration (added Phase 15 Q13 follow-up 2026-04-28; sibling to 15.27)
- BRAIN15.29 — Summit-then-timing-then-updated-summit-then-updated-timing convergence one-pass (added Phase 15 Q12 follow-up 2026-04-28; promotes IBM-C6C)

Those are the in-scope Phase 15 APS items. This SOUP catches everything else.

## Future-phase candidates added during Phase 15 transfer follow-up (2026-04-28)

### Amplicon-level clustering (complementary to sample-level APS clustering)

- **Source:** Phase 15 transfer-stage user note (chat 2026-04-28): *"Note that clustering is primarily on samples, not amplicons. Amplicon similarity based on clustering is still something we can explore in the future (perhaps add it to APS_SOUP.md)."*
- **Concept:** Cluster AMPLICONS by their cross-sample feature profiles — i.e., for each amplicon build a vector of per-sample values (summit, width, area, shape PCA scores, etc.), then cluster amplicons by similarity. Complementary to BRAIN15.27 / sample APS clustering, which clusters samples by per-amplicon feature vectors.
- **Use cases (speculative):** identify groups of amplicons with similar developmental-stage activation patterns; discover "amplicon families" that respond synchronously across stages; compare amplicon families across chromosomes / genomic contexts.
- **Phase placement:** future phase (post-Phase-15). Implementation leverages most of BRAIN15.27's machinery — same per-amplicon features, same Layer-1 amplicon PCA. The distance computation flips axis (sample-distance → amplicon-distance), and clustering operates on the transposed matrix.
- **Open questions for whoever picks this up later:**
  1. What's the right normalization for amplicon-level features? Per-amplicon z-score? Per-feature z-score? Both?
  2. How does amplicon clustering interact with the BRAIN15.7 reliability-filtered amplicon set — should reliability filtering apply before amplicon clustering, or is unreliable-amplicon membership in a cluster diagnostic information?
  3. Does this need its own `--aps-feature` family, or does it slot into a separate `--aps-amplicon-clustering` flag with the existing feature modes as inputs?
  4. Are there amplicon-level features that ONLY make sense in the cross-sample direction (e.g., variance across samples, max-min range, "how stage-discriminating is this amplicon" scores)? If so, those become amplicon-clustering-specific features distinct from BRAIN15.27's sample-clustering features.

#### 2026-05-03 user-direction refinement (Phase 15 cycle 15.6a-S1 chat)

User riffed (paraphrased — concretization needed):

- **Start simple — single-feature amplicon clustering first.** Cluster amplicons on a single per-amplicon scalar (e.g., `summit_rcn`, or `width_above_threshold_bp`, or `area_excess`, or `importance_score`, or `reliability_score` — pick one). One score per amplicon → easy distance computation; cluster by similarity in that single dimension.
- **Then graduate to multi-feature amplicon clustering.** Use a feature LIST (summit, width, area, log10_area, asymmetry, etc.) per amplicon — same set the existing APS clustering machinery already builds, but with axes flipped: rows = amplicons, columns = (cross-sample-summary) features.
- **Avoid PCA + shape modes for the first pass** — those are designed for sample-level clustering where there are many bins per amplicon. Amplicon-level clustering wants single-score-per-amplicon-per-feature semantics. PCA stays available as a future complication once single + multi-feature variants are honest.
- **Output location question:**
  - Option (a) — same `<aps-out>/` step, different sub-directory (e.g., `<aps-out>/amplicon-clustering/`). Co-located with sample-level APS clustering.
  - Option (b) — its own top-level pipeline-step directory (e.g., a new step number). More discoverable; cleaner mental model.
  - User leans toward (b) for user-friendliness. Open until concretization-stage formalization.
- **Implementation hint:** the existing APS clustering machinery (BRAIN15.27 / SPEC15.11/SPEC15.12) builds a (sample × feature) matrix. Transposing that matrix gives a (feature × sample)-as-(amplicon × sample-summary)-shaped structure. The "feature" axis becomes amplicons, the "sample-summary-stat" axis becomes the new feature axis. Substantial machinery reuse possible.
- **Concretization gap:** what cross-sample summary statistic per amplicon? Mean across samples, median, max, max-min range, variance, "stage-discriminating" score (e.g., max-stage-mean − min-stage-mean), etc. Each has different biological semantics. Worth explicit deliberation when this graduates to BRAINSTORM.

This refines but does not supersede the parent entry above. The "future phase" placement still holds.

### "Shape-aware" candidate features evaluated and rejected during Phase 15 (provenance)

The following candidate APS clustering feature modes were evaluated during Phase 15 transfer follow-up (2026-04-28) and **rejected** with rationale documented in `multi-agent/project_context/DECISIONS.md` 2026-04-28 entry:

- **Fourier descriptors of the RCN profile** — structural mismatch (designed for periodic / closed contours; RCN profiles are linear single-peaked).
- **Curvature features (second derivative of the RCN profile)** — noise amplification + redundancy with `--aps-feature shape` and existing parabola-fit curvature.
- **Multi-family BIC vectors as a sample-clustering feature** — the most useful application is amplicon classification, already covered by BRAIN15.18; for sample clustering specifically, the single existing `dBIC_flat_vs_tri` (= `shape_score_raw`) is included in BRAIN15.27's feature vector already.
- **`peak_rcn` as a distinct feature alongside `summit_rcn`** — redundant (post-rename per BRAIN15.28 they're the same column).

If a future phase wants to revisit any of these, the DECISIONS.md entry should be the starting point.

## Side-quest awareness items (per-chromosome APS clustering bootstrap + APS_mean_of_means)

Per Phase 15 Q10 user direction (2026-04-28), the following `tracking/BRAINSTORM.md` entries are **NOT formal Phase 15 objectives** but should be kept in mind during APS clustering optimization, with progress folded back into the original BRAINSTORM entries if it occurs:

- `tracking/BRAINSTORM.md [2026-04-18] Per-chromosome APS clustering bootstrapping experiments`
- `tracking/BRAINSTORM.md [2026-04-18] APS_mean_of_means clustering test`

User-quoted disposition: *"This is a very interesting issue that we should keep in mind when optimizing APS clustering. But we should not raise it to a formal objective of this Phase. If we happen to make progress on this in this phase or side quests in this phase, then we should update that BRAINSTORM idea with relevant information we learn. So we might make some progress here, but it should not be a blocker for phase completion, and it should not be formalized into an objective; just mention it as something to keep in mind at the appropriate time, and to update if progress is made."*

These could be promoted to formal BRAIN entries in a future APS-focused phase.
