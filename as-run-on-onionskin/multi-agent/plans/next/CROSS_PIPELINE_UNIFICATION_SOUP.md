# Cross-Pipeline Structural Unification — SOUP

**Stage:** SOUP (pre-BRAINSTORM scratchpad in `multi-agent/plans/next/`).
**Theme:** Structural unification across the three onionskin pipelines (Growth,
RMS, HMM) for the analysis surface downstream of amplicon-calling — APS, summit
refinement, timing, reliability, gap analysis, clustering, plots, notebooks,
plus the supporting code-identifier conventions (function names, parameter
names, dict keys) that make the unification durable.
**Lifecycle:** Promotes to a per-phase BRAINSTORM in `multi-agent/plans/`
when the orchestrator brings it onto the stage and assigns a phase number.
See `multi-agent/AGENT_CONVENTIONS.md § Future-phase planning surfaces`
and `multi-agent/workflows/phase-development-system_PDS-v2.md` for the
SOUP → BRAINSTORM → SPEC lifecycle.

**Created:** 2026-04-30 by Claude Code 2.1.117 (claude-opus-4-7), per user
direction following Phase 15 cycle 15.6a R1 audit's `[SURPRISE:15:15.6a:1:A:8]`
high-weight meta-observation that the cross-pipeline structural divergence
pattern keeps recurring as discrete cross-pipeline-port priorities despite
explicit user direction.

**Status:** SOUP-stage. Not started. Phase 15 in flight is closing many
discrete cross-pipeline-port priorities (SPEC15.4 ODW, SPEC15.6 reliability,
SPEC15.7 summit refinement, SPEC15.8 summit↔timing convergence, SPEC15.18
`--peak-summary` extension, SPEC15.22 gap-analysis); each closes one
microcosm of the pattern this SOUP captures. This SOUP will graduate to a
dedicated phase when the recurring-port work has accumulated enough that
a structural fix is more efficient than continuing per-cycle ports.

---

## Why a dedicated structural-unification phase

The user's stated project intent (sourced from `CLAUDE.md`, multiple memory
files including `feedback_cross_pipeline_parity_blindspot.md`, and recurring
direction at brainstorming-stage rounds): **"the only true difference between
pipelines is how they call amplicons."** Everything downstream of
amplicon-calling — APS, timing, summit refinement, reliability, gap analysis,
clustering, plots, notebooks — should be a uniform analysis surface across
Growth, RMS, and HMM, with pipeline-specific exceptions explicitly documented.

What the live tree shows as of 2026-04-30 (HEAD `25834ec`, post-cycle-15.6a
closeout) is a different reality:

1. **HMM has its own parallel-but-thinner copies of multiple analysis modules.**
   `run_step16_hmm_aps` is a thinner version of `compute_and_write_aps`
   (Growth + RMS go through). Trajectory clustering at
   `hmm_ported_analyses.py` has no Growth/RMS analog (and arguably should
   not — fork-trajectory is HMM-native — but the boundary between
   "HMM-specific" and "should-be-universal-but-was-only-built-for-HMM" is
   blurry).

2. **Cross-pipeline ports keep getting filed as discrete cycles.** Phase 15
   alone has at least 5 cross-pipeline-port priorities. Each closes one gap
   and reveals (or creates) the next one. The audit-implement-reaudit loop
   catches each instance but does not catch the pattern. Cycle 15.6a R1's
   `[SURPRISE:15:15.6a:1:A:8]` named this explicitly as a high-weight
   meta-observation.

3. **HMM-only step-number leakage in code identifiers.** The HMM pipeline
   uses step-numbered function names (`run_step15_hmm_gap_analysis`,
   `run_step16_hmm_aps`, etc.), step-numbered dict keys
   (`build_hmm_steps()` returns `{"step1": ..., "step2": ..., ...}`), and
   step-numbered parameter/variable names (`step11_dir`, `step12_dir`,
   `step11_rows`). Growth and RMS use semantic identifiers. Each HMM
   pipeline step renumber (Phase 15 alone has had two — cycle 15.2a SAPS
   placeholder insertion, cycle 15.4b gap-analysis-before-APS reorder)
   forces sweeping renames across ~250+ touch points and the renumber
   sweep is never quite complete (Final Overseer cycles routinely catch
   missed sites). See `[ISSUE:2026-04-30:2]` for the concrete inventory.

4. **The duplicated `_FEATURE_COL` dict was a microcosm.** Two dispatch
   dicts, two places to update on every column rename. Cycle 15.6a Stage C
   collapsed this into a shared `build_aps_feature_matrix()` resolver as
   a partial mitigation, but the pattern persists in adjacent surfaces:
   - `build_amplicon_importance()` at `aps.py:1537-1541` — Growth + RMS
     emit `aps_amplicon_importance.tsv`; HMM does not. (See
     `[SURPRISE:15:15.6a:1:A:7]`.)
   - `build_scalar_orderings()` at `aps.py:618-663` — Growth + RMS only.
   - `write_aps_plots()` at `aps.py:1547-1560` — Growth + RMS only; HMM
     has its own structurally different plot path.

5. **`feedback_cross_pipeline_parity_blindspot.md` exists because the
   pattern keeps recurring.** Memory: *"User has emphasized multiple times
   that 'X-pipeline-only' defects systematically slip through audits despite
   his repeated efforts to expose them."* The memory is dated and explicit;
   it has been an active feedback signal for multiple cycles. Yet each
   cycle's audit catches the issue at the in-cycle remediation level rather
   than questioning whether the structural decoupling itself is the root
   cause of the recurring pattern.

The cumulative observation: **the project has accumulated structural
decoupling between pipelines that no one cycle is sized to address, so it
keeps surfacing as the same class of finding under different names.** A
dedicated phase is the right home.

---

## What this SOUP is about

A coherent, cross-pipeline rationalization of the analysis-surface code
structure such that:

1. The "only true difference between pipelines is how they call amplicons"
   intent is **enforced at the type / interface level**, not just by
   per-cycle audit vigilance.
2. Code identifiers (function names, dict keys, parameters, variables) are
   semantic ("aps", "timing", "summit-refinement"), not positional ("step15",
   "step16"). Step numbers belong on output directory names + user-facing
   diagnostic strings + intentional algorithm-step artifacts only.
3. Every analysis-surface module that exists for one pipeline either exists
   for all three (with documented pipeline-specific implementations where
   needed) or has an explicit DECISIONS-locked rationale for why it is
   pipeline-specific (e.g., HMM trajectory clustering has no Growth/RMS
   analog because fork-trajectory is HMM-native).
4. Future cross-pipeline drift is structurally prevented (e.g., via shared
   abstract-base interfaces that all three implementations must satisfy,
   or via lint/grep guards that flag asymmetric helpers).

The phase covers algorithm structure, code organization, naming conventions,
and structural test coverage together.

---

## Items parked here

### Item 1 — `build_hmm_steps()` semantic-key refactor

**Background.** `onionskin_core/output_layout.py:301-336` returns step-numbered
keys for the HMM pipeline; Growth + RMS use semantic keys. ~242 call sites in
`onionskin_core/engines/hmm_engine.py` consume `steps["step15"]`,
`steps["step16"]`, etc. — encoding pipeline step positions into engine logic.
When step numbers shift (twice in Phase 15 alone), every call site has to be
re-examined.

**Fix.** Flip dict keys to semantic names matching Growth + RMS conventions:
`steps["mednorm"]`, `steps["stage-medians"]`, `steps["hmm"]`,
`steps["collapsed-hmm"]`, `steps["above-background"]`, `steps["summit-states"]`,
`steps["summit-bins"]`, `steps["amplicon-metrics"]`, `steps["fork-travel"]`,
`steps["summit-refinement"]`, `steps["multistage-unification"]`,
`steps["gap-analysis"]`, `steps["aps"]`, `steps["saps"]`, `steps["timing"]`,
`steps["clustering"]`, `steps["active-stage-summit"]`, `steps["timing-updated"]`.
Update all 242 engine call sites. Update PIPELINE_SPEC.md cross-references.
PuffStep parity gate is the safety net.

**Dependencies.** None hard. Cross-references with cycles touching engine
internals — schedule away from cycles that are heavily editing
`hmm_engine.py` to minimize merge cost.

**Estimated scope.** Substantive single-cycle work; ~1-2 days for an
experienced developer including tests and grep verification. Not as small as
Phase 15 cycle 15.9a's "mechanical-with-care" frame would accommodate.

---

### Item 2 — Function / parameter / variable renames cascading from Item 1

**Background.** Once `steps["step15"]` becomes `steps["gap-analysis"]`, the
surrounding function names like `run_step15_hmm_gap_analysis` look obviously
redundant — the function takes `steps["gap-analysis"]` as input and the
function name already says "gap_analysis"; the "step15" prefix is pure
positional cruft.

**Fix.** Cascade renames:
- `run_step15_hmm_gap_analysis` → `run_hmm_gap_analysis`
- `run_step16_hmm_aps` → `run_hmm_aps`
- `run_step17_hmm_saps` → `run_hmm_saps`
- `run_step18_hmm_timing` → `run_hmm_timing`
- `run_step19_hmm_clustering` → `run_hmm_trajectory_clustering`
- `run_step13_summit_refinement` → `run_hmm_summit_refinement` (or similar)
- Internal helpers: `_load_step11_rows` → `_load_amplicon_metrics_rows`;
  `_discover_step11_stage_files` → `_discover_amplicon_metrics_stage_files`;
  `_load_step12_growth` → `_load_fork_travel_growth`.
- Parameter renames: `step4_dir` → `mednorm_dir` or `chrom_renorm_dir`;
  `step11_dir` → `amplicon_metrics_dir`; `step12_dir` → `fork_travel_dir`;
  `step14_dir` → `multistage_unification_dir`.

**Dependencies.** Best done immediately after Item 1 lands so the call-site
context already references semantic names.

**Estimated scope.** Smaller than Item 1 once the call sites are already
updated; ~half a day.

**Postscript (2026-05-04):** Cycle 15.7b absorbed the HMM-side function renames
from this item for the summit-refinement slice via SPEC15.24 d5:
`run_step13_summit_refinement` → `run_hmm_summit_refinement`,
`_load_step11_rows` → `_load_amplicon_metrics_rows`,
`_discover_step11_stage_files` → `_discover_amplicon_metrics_stage_files`, and
`_load_step12_growth` → `_load_fork_travel_growth`. The per-stage HMM summit
emission gap that motivated this nearby cleanup also landed in-cycle via
SPEC15.24 d1 (`hmm_stage{N}_summit_estimates.tsv` in `13-summit-refinement/`).
The bulk `build_hmm_steps()` semantic-key refactor and its ~242 call sites
remain future-phase work.

---

### Item 3 — HMM-thinner-APS-path surfaces audit + port

**Background.** Per `[SURPRISE:15:15.6a:1:A:7]` and Stage C of cycle 15.6a's
implementation report:
- HMM does not emit `aps_amplicon_importance.tsv` (Growth + RMS do via
  `build_amplicon_importance` at `aps.py:1537-1541`).
- `build_scalar_orderings()` at `aps.py:618-663` is Growth + RMS only.
- `write_aps_plots()` at `aps.py:1547-1560` is Growth + RMS only; HMM has
  its own structurally different plot path under `hmm_notebooks.py`.

SPEC15.17 / cycle 15.8a is supposed to catch this via the master APS catalog
(`multi-agent/full_instructions/APS_CATALOG.md`), but the gap inventory may
be longer than SPEC15.17's wording captures. This Item parks the "audit
+ port the parallel-but-thinner HMM analysis surfaces to genuine cross-pipeline
parity" work that is too broad for any single Phase 15 cycle.

**Fix.** Audit every analysis-surface module for "Growth + RMS only" or
"HMM-only" patterns; for each, decide: port to all three / decompose into
shared core + pipeline-specific orchestration / explicitly document as
pipeline-specific with a DECISIONS-locked rationale.

**Dependencies.** Best done after SPEC15.17 / cycle 15.8a's APS catalog
deliverable lands — that catalog is the audit's starting inventory.

**Estimated scope.** Multi-cycle theme; could justify a phase-group structure
when it graduates.

---

### Item 4 — Abstract-base-class scaffold to prevent future drift

**Background.** Even if Items 1-3 land cleanly, future work can re-introduce
asymmetric helpers because nothing in the type system enforces parity. A
`BasePipelineAPS` abstract base class (or Protocol) that all three
implementations must satisfy would catch parity drift at type-check time
rather than at audit time.

**Possible designs.**
- **Abstract base class** — `class BasePipelineAPS(ABC): ...` with abstract
  methods for every cross-pipeline analysis-surface helper. Pipeline-specific
  classes inherit + implement. Pros: type-checkable parity. Cons: invasive
  refactor; may not match Python idioms in this research codebase.
- **Protocol-based** — `class APSPipeline(Protocol): ...` with required
  method signatures. Pros: less invasive than ABCs. Cons: still requires
  refactor of three current implementations to fit the protocol.
- **Lint/grep guards** — a `scripts/check_cross_pipeline_parity.py` script
  that grep-scans for asymmetric helpers + fails CI if found. Pros: minimal
  refactor; catches drift at PR time. Cons: less rigorous than type system;
  may miss subtle asymmetries.

**Dependencies.** Hard prerequisite: Items 1-3 substantially complete (or
at least a clear-enough analysis-surface inventory that the abstract
interface is well-defined).

**Estimated scope.** Could be small (lint script) or large (full ABC
refactor) depending on chosen design.

---

### Item 5 — Convention guards in AGENT_CONVENTIONS.md

**Background.** Even with the structural fixes, future code reviews + audits
need explicit conventions to enforce. The 2026-04-30 update to
`multi-agent/AGENT_CONVENTIONS.md` § Naming conventions (committed alongside
this SOUP's creation) added the rule about no-step-numbers-in-code-identifiers.
Additional guards may be useful:

- "When adding a new helper to one pipeline, audit + document whether it
  belongs on the other two."
- "When renumbering pipeline steps, run a grep-based blast-radius check
  before declaring the renumber complete; the script lives at
  `scripts/check_step_renumber_blast_radius.py` (proposed)."
- "Cross-pipeline analysis-surface helpers MUST live in shared modules
  (`onionskin_core/aps.py`, `onionskin_core/timing.py`, etc.) with
  pipeline-specific orchestration only at engine boundaries."

**Dependencies.** None — these can land incrementally as conventions even
before the structural items.

---

### Item 6 — Cross-pipeline feature parity / silent no-ops reckoning (added 2026-04-30)

**Background.** Items 1–5 capture cross-pipeline STRUCTURAL parity (function names, dict keys, base classes — the "shape of the code" issues). Item 6 captures cross-pipeline FEATURE parity — orchestration-layer silent no-ops where Growth gets treated as 1st-class while HMM/RMS are 2nd-class. Surfaced 2026-04-30 by orchestrator-outside-cycle audit triggered by user observation that `--posterior` does not work with `--pipelines hmm`. The 7-finding sweep that surfaced this item also found `[ISSUE:2026-04-30:4]` (HMM-only `--posterior` + `--compute-aps` no-op + flat-sample state divergence — being fixed in cycle 15.4a-S2) plus several plot-parity gaps (being fixed in cycle 15.8a via SPEC15.17 expansion).

**6a — Growth-only `--posterior` re-run shortcut.** `onionskin.py:4708-4713` — if a prior run is already complete on disk, the shortcut skips re-running the full prior and goes straight to `_run_posterior(...)`. This optimization lives ONLY in the multistage Growth branch. RMS-only multistage `--posterior` runs (which use `_run_rcn_posterior` at `onionskin.py:4805`) and HMM-only `--posterior` runs (post the `[ISSUE:2026-04-30:4]` cycle 15.4a-S2 fix) re-execute the full prior every time. **S2 (silent feature no-op — optimization just doesn't apply).** No fix in Phase 15 because the analogous shortcuts for RMS-only and HMM-only need parallel `_check_prior_complete` analogues (RMS- and HMM-aware completeness probes), which is non-trivial design work.

**6b — Output layout dict keys are Growth-anchored; no HMM-specific equivalents.** `onionskin_core/output_layout.py:243-294` — the layout dict always includes `calls_dir`, `shape_filter_dir`, `aps_dir`, `plots_dir` keyed to `02-growth-model/`. RMS has parallel `rcn_*` keys keyed to `03-rcn-mean-shift/`. HMM has only `hmm_dir`; per-step output management is internal to HMM engine. Functionally OK today (no runtime KeyError because `_phase2_postprocess` only runs for Growth and RMS, never for HMM-only), but the design asymmetry resurfaces every time HMM gets a new feature that Growth/RMS got via `_phase2_postprocess`. **S3 (asymmetric but no current bug).** Future work: introduce HMM-specific layout keys (e.g., `hmm_calls_dir`, `hmm_aps_dir`) so HMM engine consumes layout-resolved paths the same way Growth/RMS do, OR document the asymmetry as intentional with an "HMM manages its own paths" architectural rationale.

**6c — Meta-deliverable: recurring parity audit.** The 7-finding sweep that surfaced 6a + 6b was likely incomplete — the user has surfaced that "stuff keeps coming up" across multiple phases despite explicit universalization efforts. Two recurring guardrails are proposed (both confirmed by user direction 2026-04-30):

- **Cycle 15.10a (Phase 15 closeout) responsibility.** Phase closeout sweep for remaining 1st-class-Growth patterns; cheap fixes land in Phase 15, others get filed as `[ISSUE:...]` for next-phase remediation. Sweep targets: silent feature no-ops on single-pipeline configurations (analogous to Findings 1+2 but for other flags); shared-flag-vs-per-pipeline-state bugs (analogous to Finding 3); Growth-specific orchestration fast-paths (analogous to Finding 4); Growth-anchored layout / state / config (analogous to Finding 5).
- **AGENT_CONVENTIONS rule + workflow Template A guardrail.** Every cycle's R1 audit includes a brief "is anything Growth-anchored that should be universal?" sweep as part of standard audit protocol. Codified in `multi-agent/AGENT_CONVENTIONS.md` (when Item 6 graduates to BRAINSTORM) and in Template A of `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md`.

**6d — Finding 3 cross-reference (representative of broader pipeline-state-divergence pattern).** `[ISSUE:2026-04-30:4]` Finding 3 (flat-sample auto-switch is per-pipeline-set but consumed as a single shared flag in `_run_posterior` at `onionskin.py:4402-4411`) is being **fixed in cycle 15.4a-S2** (per-pipeline state preserved per-pipeline; per-pipeline consumption). The specific defect is in-flight; THIS sub-item parks the broader theme: are there other places where per-pipeline state collapses into a shared flag? Examples to audit during 6c's recurring sweep: any `args._<state>` attribute that's set in multiple per-pipeline call sites (race / last-write-wins hazards); any controller-level decision that consults a single state value when per-pipeline state is the actually-relevant signal.

**Dependencies.** Independent of Items 1–5 — feature parity and structural parity are different dimensions of the same problem and can be addressed independently.

**Effort sizing.** Items 6a + 6b are concrete deferred-fix work (~1-2 cycles each in a future phase). Item 6c is recurring meta-work (no end-state — just a discipline). Item 6d is a brainstorm seed for further audit.

---

### Item 7 — Posterior-inherited background mask: `specific` vs `union` vs hybrid (added 2026-05-02)

**Context.** Phase 15 cycle 15.4a-S4 lands the controller-level **first-pass** background mask: per-sample chrom-median normalize → keep RCN ∈ [LO,HI] AND NOT in gap mask AND NOT in gap-flank halo AND NOT in `--chrom-trim` of chromosome; cross-sample union-of-excluded-bins computed once at the controller level and shared by all pipelines that run. The first-pass mask is **cross-pipeline shared by virtue of being computed once pre-pipeline**.

Phase 15 cycle 15.7a (re-scoped SPEC15.15) lands the **second-pass** background-region inheritance for posterior runs: each pipeline does its own pass-2 background detection during its prior run (= pipeline-specific refinement based on detected amplicons + above-background classifications); when a posterior is run, it inherits the **prior's** pass-2 mask rather than recomputing.

This SOUP item captures the **future-looking design space** for *which* prior's mask the posterior should inherit when multiple pipelines have been run in the same prior — i.e., what does `--posterior-inherited-background` mean architecturally.

**Design space.**

- **`specific`** — posterior of pipeline X inherits pipeline X's pass-2 mask. Default for now (Phase 15 cycle 15.7a). Simplest contract. Reflects each pipeline's internally consistent view.
- **`union`** — posterior of pipeline X inherits the union of all available pipelines' pass-2 masks (`hmm ∪ rcn-mean-shift ∪ growth` for whatever pipelines were actually run in the prior). Conservative — treats anything any pipeline excluded as excluded. Likely better for users who want maximal exclusion of biologically-active regions before posterior MAD/median estimation.
- **Hybrid (3rd option, name TBD).** Possibilities:
  - **`intersection`** — bins all pipelines agreed are background. Most permissive; very few bins. Probably too aggressive on real fixtures.
  - **`majority`** — bins ≥2 of 3 pipelines marked as background. Compromise between union and intersection.
  - **`weighted-union`** — union but weighted by per-pipeline confidence at each bin.
  - **`pipeline-confidence-derived`** — choose which pipeline's mask to use per chromosome based on which pipeline has the most by-eye-validated calls on that chromosome (data-driven; expensive).

**Anticipated user trajectory.** Default starts at `specific` (cycle 15.7a) for simplicity + same-pipeline internal consistency. As real-fixture experience accumulates and the cross-pipeline-divergence pattern becomes better characterized, the default may flip to `union` (conservative) or to a hybrid option once empirical evidence supports the third-option semantic.

**Why this lives in CROSS_PIPELINE_UNIFICATION_SOUP and not in `FLAT_SOUP.md`.** FLAT_SOUP captures the upstream-prior architecture — i.e., the shape of the prior pass itself. This item is about *cross-pipeline composition* of pass-2 results in the posterior — structurally cross-pipeline-mask-unification, which is the core theme of this SOUP. It's also explicitly cited as "future-phase work" in the cycle 15.4a-S4 STRATEGY row's out-of-scope guardrail (`cross-pipeline mask unification (intersect/union/hybrid for --pipelines all) — future phase`).

**Dependencies.** Hard prerequisite: cycle 15.4a-S4 (controller-level first-pass mask machinery) + cycle 15.7a (per-pipeline pass-2 background-region detection in prior + posterior-inheritance plumbing under `specific` default). This SOUP item is the next-phase evolution of cycle 15.7a's `specific`-default decision into a richer cross-pipeline-aware design.

**Effort sizing.** ~1-2 cycles in a future phase. Code surface is the `--posterior-inherited-background` flag's resolver + per-bin mask combination logic. The hard work is the empirical evaluation of which option produces materially better posterior MAD estimates on the user's real fixtures.

**Open brainstorm questions.**

- Does `union` actually improve posterior chrom-MAD quality on real fixtures vs `specific`? Or is it conservative-without-benefit? Need empirical evidence on `dev/datasets/full_genome/batch/` and `tests/full_chrom_training_data/`.
- For the hybrid option: is `majority` (≥2 of 3) a meaningful default candidate, or does it collapse to ~`union` whenever 2 pipelines agree (which they usually will on real data)?
- Is the "third option" actually a different *aggregation* (intersection/majority/weighted) or a different *semantic* (pipeline-confidence-derived per chromosome)? The latter is a much larger surface and may belong in its own SOUP.
- Does this interact with GC-aware chrom-median (`GECKO_SOUP.md`)? GC-aware normalization changes which bins land in [LO,HI] — but the posterior-inheritance choice operates on the result of pass-2, which is downstream of normalization. Probably independent.

---

## Known interactions with other phases / SOUPs

- **`multi-agent/plans/next/SUMMIT_SOUP.md`** — adjacent cross-pipeline
  parity work specifically for summit-algorithm methodology. SUMMIT_SOUP's
  Item 5 expansion captures cross-pipeline summit-refinement-algorithm parity
  (port `refine_summit_parabola` to Growth + HMM; HMM `_estimate_parabola_bp`
  unification; HMM per-stage parabola summit emission). Those items are
  one specific instance of the broader pattern this SOUP captures. The two
  SOUPs cross-reference each other; they may graduate to phases on their
  own timelines or merge into a single structural-unification phase
  depending on scope sizing at graduation time.

- **Phase 15 cycles touching this SOUP's surfaces:**
  - Cycle 15.6a Stage C (closed v0.14.84) collapsed the duplicated
    `_FEATURE_COL` dict into a shared `build_aps_feature_matrix()` resolver
    — partial mitigation of one microcosm.
  - Cycle 15.7a (queued) MUST opportunistically rename `run_step17_hmm_saps`
    → `run_hmm_saps` since SAPS is net-new code (per
    `[ISSUE:2026-04-30:2]`'s in-phase fix guidance).
  - Cycle 15.8a (queued, SPEC15.17) lands the master APS catalog — the
    audit-inventory deliverable that informs Item 3's port scope.
  - Cycle 15.9a (queued, SPEC15.21) takes the bulk of HMM-pipeline-specific
    Category 2 + 3 function/param/variable renames as a sub-deliverable
    (per `[ISSUE:2026-04-30:2]` landing zone). NOT Item 1 (`build_hmm_steps`
    refactor) — that's too large for cycle 15.9a's "mechanical-with-care"
    frame.

- **`multi-agent/tracking/KNOWN_ISSUES.md`:**
  - `[ISSUE:2026-04-30:2]` HMM step-number leakage — concrete actionable
    inventory; this SOUP is the future-phase substantive home for the
    bulk refactor.
  - `[ISSUE:2026-04-29:7]` cross-pipeline parity audit gap — closeout-tracking
    issue from cycle 15.4a-S1 closeout; this SOUP graduates to the phase
    that satisfies that issue's exit condition.

---

## When to start

When the per-cycle cross-pipeline-port priorities accumulate enough that a
structural fix is more efficient than continuing port-by-cycle. Concrete
triggers that would justify graduation:

- A future Phase N audit's R1 catches the same pattern as
  `[SURPRISE:15:15.6a:1:A:8]` — i.e., the recurring-port pattern is still
  surfacing despite Phase 15's discrete-port closures.
- The HMM pipeline undergoes a third step renumber and the renumber-sweep
  cost again exceeds expectations.
- The user explicitly directs that the analysis-surface decoupling has
  reached "diminishing returns on per-cycle ports."

Until any of those triggers, Phase 15's discrete cross-pipeline-port priorities
are doing the right work — closing one microcosm at a time. This SOUP captures
the pattern + the deferred-bulk for when it makes sense to go bigger.

---

## Open brainstorm questions (not yet answered)

1. **Phase boundaries.** If this SOUP graduates, does it become one
   structural-unification phase or a phase-group with multiple phases (one
   per item or item-cluster)? Item 1 (build_hmm_steps refactor) plus Item 2
   (cascading function/param renames) is probably one phase. Item 3
   (HMM-thinner-APS-path port) plus Item 4 (abstract-base scaffold) might
   need their own phase.

2. **SUMMIT_SOUP merger?** Should this SOUP and SUMMIT_SOUP merge into a
   single "cross-pipeline structural unification" phase at graduation, given
   substantial overlap? Or stay separate with cross-references? Decision
   depends on how the items partition cleanly.

3. **Abstract-base-class vs Protocol vs lint-script for Item 4** —
   philosophical question about how strict to make parity enforcement.
   Research codebases often resist heavy ABC scaffolding. The lint-script
   approach may fit the project's culture better; needs user input.

4. **Naming consistency for HMM-only-by-design surfaces.** `hmm_summit_refinement.py`
   and `hmm_fork_travel.py` are named with the `hmm_` prefix appropriately
   because they're genuinely HMM-specific. But should the file naming
   convention extend to *function* prefixes? E.g., `run_hmm_aps` (this
   SOUP's Item 2 proposal) vs `run_aps` in an HMM context. The prefix
   approach loses meaning if all three pipelines have a `run_aps` named
   the same way; the prefix preserves clarity if they don't.

5. **Backward-compat requirements for the public CLI surface.** The CLI flag
   names (e.g., `--hmm-saps-*`) are user-facing and probably stable. But
   the internal function/parameter renames have no backward-compat constraint
   — they're internal. Confirm this assumption when graduation time comes.

6. **Topic-via-different-ingredients framework (per user direction 2026-04-30).**
   Pipeline-specific plots ARE legitimate when they're based on
   pipeline-specific INGREDIENTS — state path is HMM-only, mean-shift
   trajectory is RMS-only, growth track is Growth-only. But when a TOPIC is
   addressed by a pipeline-specific plot in one pipeline (e.g.,
   asymmetry-in-state-path for HMM), is there an analogous topic-via-different-
   ingredients plot for the other pipelines (e.g., asymmetry-in-RCN-signal
   for Growth + RMS)? Light scan performed 2026-04-30 of HMM-native plots:
   - **Asymmetry scatter** (`onionskin_core/hmm_fork_travel.py:656`) — HAS clear
     RCN analogue: left-flank vs right-flank slope/integral measurable in any
     RCN profile. Cross-pipeline candidate.
   - **Asymmetry scatter by stage** (`onionskin_core/hmm_fork_travel.py:717`) —
     same concept per stage; cross-pipeline candidate.
   - **Level emergence heatmaps** (`onionskin_core/hmm_fork_travel.py:805`) —
     possible analogue: per-stage threshold-crossing pattern in RCN (when does
     locus exceed threshold per stage). Needs design discussion.
   - **Nested domains** (`onionskin_core/hmm_fork_travel.py:901`) — possible
     analogue via nested peak structure (summit + flanking-boundary). Substantial
     new analysis surface.
   - **Fork-travel trajectories** (`onionskin_core/hmm_fork_travel.py:611`) — no
     clean analogue; concept is HMM/state-machine specific. Keep HMM-native.
   - **Fork-age trajectories** (`onionskin_core/hmm_fork_travel.py:983`) — weak
     analogue (onset-stage-as-age in timing data); concept blur.
   Per user direction 2026-04-30: **HMM-enriching work is Phase 15 (already on
   track via SPEC15.17 expansion); HMM-exporting work (porting HMM-native
   concepts to Growth/RMS) is deferred to this SOUP, with a full audit when
   this SOUP graduates to BRAINSTORM stage.** Light-scan results above are
   the starting inventory; full audit at graduation time.

7. **Other unsurfaced 1st-class-Growth patterns.** The 7-finding sweep that
   surfaced Item 6 was a starting inventory, not exhaustive. The user has
   noted that "stuff keeps coming up" across multiple phases despite explicit
   universalization. What's still hiding? This is the audit-target for Item
   6c's recurring sweep. Categories to audit when Item 6c fires:
   - Single-pipeline shortcut branches (`only_X` patterns) — do they all honor
     the universal flags they should?
   - Shared `args._<state>` attributes that get set per-pipeline but consumed
     as one — Finding 3 was one example; how many more?
   - Output layout / file naming asymmetries — Finding 5 was one; others?
   - Help text vs implementation mismatches — flags documented as universal
     but only wired to one pipeline.

8. **`--skip-posterior` / `--skip-aps` default-flip (parked, low priority).**
   User-suggested 2026-04-30: should `--posterior` and `--compute-aps` flip to
   default-on with `--skip-posterior` / `--skip-aps` opt-out flags? Significant
   blast radius (run-time roughly doubles when posterior is implicit; every
   test that doesn't expect APS+posterior outputs needs updating; potential
   breaking change for downstream users). User direction 2026-04-30: skip for
   now, super low priority. Parked here for rediscovery if user revisits the
   defaults question. No further work attached.

9. **Posterior-inherited-first-pass-mask synthesis strategy — which mode is
   empirically best (`local` / `union` / `intersect` / other)?** Surfaced
   2026-05-04 during cycle 15.7a Stage G implementation discussion. The
   architectural question: when the posterior pass runs and inherits the
   first-pass background mask from the prior pass, should each pipeline
   inherit *only its own* prior-pass mask (`local`), or should the three
   pipelines synthesize a unified mask (e.g., `union` of the three
   prior-pass masks, or `intersect` of the three prior-pass masks)?

   **What landed in cycle 15.7a Stage G (commit `c38654e`):** Per-pipeline
   inheritance — each pipeline's posterior pass uses its own prior pass's
   first-pass background mask. This is effectively the `local` mode and is
   preserved as the default going forward (it is the implementation the
   user converged on after mid-flight pushback against R2's earlier
   single-shared-mask architectural error).

   **Why a flag is being added (cycle 15.9a SPEC15.21 amendment 2026-05-04):**
   The user explicitly wants to keep the per-pipeline inheritance work as
   the default but expose `union` / `intersect` synthesis modes via flag so
   that future-phase empirical comparison is cheap. Flag name:
   `--posterior-inherited-mask local|union|intersect` (default `local`).

   **What `union` and `intersect` would mean conceptually:**
   - `union` — take the union of bins masked by each prior-pass pipeline's
     first-pass background mask. More-aggressive mask (more bins excluded
     from background estimation in posterior pass).
   - `intersect` — take the intersection. Less-aggressive mask (only bins
     all three pipelines agreed to mask).
   - `local` — pipeline-specific (what cycle 15.7a Stage G implements).

   **Suggested experimental design for future-phase exploration:**
   - Run posterior with each mode on representative datasets (DS1
     chr-II calibration; full-genome DS1; the user's `dev/share/res...`
     sets if appropriate).
   - Compare downstream calls (call counts, summit positions,
     reliability scores) across the three modes.
   - Compare chrom-MAD stability of the posterior pass-2 background
     estimates across the three modes (to the extent posterior pass-2
     ever runs — currently SPEC15.15 d6 default-locks it OFF).
   - Default-flip if data warrants. Otherwise `local` stays default.

   **Why this is future-phase, not Phase 15:** SPEC15.13/14/15/16 in Phase
   15 already locked the per-pipeline-inheritance design as the cycle
   15.7a-bound Stage G implementation; the synthesis-mode flag exists
   primarily so the question stays empirically askable. Active comparison
   work belongs in a structural-unification phase (this SOUP) where the
   broader cross-pipeline-mask machinery is already on the table.

   **Cross-references:**
   - Cycle 15.7a R2 Stage G commit `c38654e` (per-pipeline inheritance
     = `local` mode; posterior helpers
     `_run_hmm_posterior` / `_run_rcn_posterior` / `_run_posterior` and
     resolver `_resolve_prior_mask` in `onionskin.py`).
   - Cycle 15.9a SPEC15.21 scope amendment 2026-05-04 (5-item amendment;
     this SOUP entry's flag is item 5 of that amendment).
   - User chat 2026-05-04 — *"I think the second pass intersect or union
     approach for masking in the posterior would be fine."* + *"Let's not
     get rid of the pipeline-specific inheritance work — keep it as a
     mode."*
   - `multi-agent/plans/PHASE15_SPEC.md § SPEC15.15` (cross-pipeline
     pass-2 background-mask machinery) and `§ SPEC15.21` post-amendment
     (where the flag itself lands).

These are starter questions. More will emerge during brainstorming rounds.
