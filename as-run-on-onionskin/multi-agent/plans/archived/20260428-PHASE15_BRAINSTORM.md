# Phase 15 BRAINSTORM — HMM completeness + enrichment (with cross-pipeline updates where applicable)

**Stage:** BRAINSTORM (per-phase formalization staging in `multi-agent/plans/`).
**Theme:** HMM completeness + enrichment.
**Source SOUP:** `multi-agent/plans/archived/20260428-PHASE15_HMM_SOUP.md` (archived 2026-04-28 at brainstorming-stage cycle closeout v0.14.76.2; SOUP body is read-only and was the seed for this BRAINSTORM's 21 transcribed-from-SOUP entries).
**Lifecycle position:** brainstorming stage active (SOUP labeled `SOUP15.1 .. SOUP15.21` and archived; BRAINSTORM iterated through Round 1–Round 6 follow-ups + Codex Agent 2 brainstorming-stage code/tracking audit (2026-04-28) + Codex Agent 2 narrow re-audit after Round 6 (2026-04-28; verdict SUBSTANTIVELY CLEAN); ratchets to SPEC engineering when user signals Q24 = "we're done; go SPEC").
**Companion file:** `multi-agent/plans/PHASE15_FEEDBACK.md` (transfer-stage Q&A, open questions, transfer-status block).

> **Maintenance:** SOUP self-references that survived in the SOUP body
> from earlier `next/`-era drafts (e.g., `Priority 12.1`, `Priority 12.2`,
> `Priority 12.3` headings; `**Stage:** SOUP (pre-BRAINSTORM scratchpad
> in multi-agent/plans/next/)` framing) are NOT carried forward into
> this BRAINSTORM verbatim. The BRAINSTORM is Phase 15's authoritative
> framing surface; the SOUP body is preserved as-is in archive.

---

## How to read this BRAINSTORM

Each BRAIN entry has:
- A heading `## BRAIN15.<idx> — <title>`.
- A `**Source:**` line citing the SOUP IDs (and any tier-1/tier-2 cross-tier sources) that this BRAIN entry covers.
- The body: faithful transcription of the source idea, cleaned up for formatting and re-organized for narrative flow. Sub-bullets, tables, and code refs from SOUP are preserved when meaningful.

Splits and consolidations across SOUP → BRAINSTORM are documented in `PHASE15_FEEDBACK.md` § "Splits and consolidations across SOUP → BRAINSTORM (Agent 1 interpretive choices needing user approval)".

---

## Phase scope and framing

### BRAIN15.1 — Phase scope: HMM completeness + enrichment

**Source:** SOUP15.1.

Finishing up HMM pipeline completeness and enriching it with the many ideas brainstormed across late Phase 10–14:

- Make HMM capable of producing its own analyses downstream of amplicon calling, including summit detection, summit refinement, APS, posterior ordering/grouping/manifest, timing, fork tracking, fork age, etc.
- Increase the accuracy and reliability of creating HMM posterior manifests (and corresponding growth pipeline updates when applicable).
- Increase the accuracy of HMM summit estimation (and corresponding growth pipeline updates when applicable).
- Expand analyses, plots, and notebooks in rich and useful ways for the HMM pipeline (and corresponding growth pipeline updates when applicable).

The phase is HMM-centric but explicitly cross-pipeline-aware: changes that touch the controller-level architecture (gap mask, missingness diagnostic) cascade to growth and RMS by inheritance per the architectural rule "anything shared by two or more pipelines that doesn't depend on pipeline-internal upstream work can come out in front, run once, and be inherited."

### BRAIN15.2 — Core design rules

**Source:** SOUP15.2.

These rules were established across Phase 10–11 and remain in force for Phase 15. They constrain how new HMM-completeness work is allowed to land architecturally:

1. Pipeline-derived outputs belong under the pipeline that computes them.
2. Shared computation belongs in modules, not in shared emitted-analysis directories.
3. Controller/run-level artifacts must remain analysis-neutral.
4. No pipeline is the default, canonical, authoritative, or main pipeline.
5. Until explicit synthesis exists, each pipeline remains independent in both `01-prior/` and `02-posterior/`.
6. Cross-pipeline synthesis remains a separate explicit layer above pipeline-local products.
7. Completeness work must respect pipeline-local ownership rather than flattening differences into centralized intermediate files.
8. Any change that alters emitted paths, output schemas, or inspection-relevant analysis surfaces must update the affected utilities, inspectors, and regression helpers in the same change rather than leaving compatibility drift for later.
9. As new pipeline-local features come online, extend the relevant inspection and regression surfaces (inspectors, evaluation harnesses, helper tests) where those features are now expected to be visible or testable.
10. `multistage` remains an input-data property, not the name of the growth pipeline.
11. `--pipelines all` means all three pipelines: `growth`, `per-stage`, and `hmm`.

### BRAIN15.3 — Active carry-over decisions from late Phase 10–11

**Source:** SOUP15.3.

These are decisions already settled at the end of Phase 10–11 and carried into Phase 15 as standing facts:

- The long-term pipeline term remains `growth-model`; `multistage` remains the data/property term.
- The retained grouped pipeline order/names are `01-hmm/`, `02-growth-model/`, `03-rcn-mean-shift/`.
- The aligned engine/module naming direction is `hmm_engine.py`, `growth_model_engine.py`, `rcn_mean_shift_engine.py` (batch engine), and `rcn_mean_shift_singlefile_engine.py` (single-file CLI), all under `onionskin_core/engines/`. The shared RCN helpers live in `onionskin_core/rcn_mean_shift_helpers.py`.
- Until explicit cross-pipeline synthesis exists, each pipeline remains independent in prior/posterior space and produces its own posterior manifest from its own prior outputs.
- The first synthesis target is call-level synthesis across pipeline-local prior outputs; downstream analyses can then be recomputed from the synthesized call set rather than treated as the first synthesis problem.
- The capability matrix for pipeline admissibility still needs an explicit written home in the live planning surface rather than remaining only implicit in controller behavior.

### BRAIN15.4 — Non-goals

**Source:** SOUP15.4.

- Do not revive deprecated shared emitted-output ownership patterns.
- Do not approximate synthesis through centralized intermediate files.
- Do not treat growth as the owner of common analysis surfaces.

---

## HMM completeness — the major work

### BRAIN15.5 — HMM completeness matrix across analysis families

**Source:** SOUP15.5 (umbrella; the carry-over queue items in SOUP15.5 are split out to BRAIN15.8 and BRAIN15.13). Round 4 expansion (2026-04-28) adds the gap-audit table from the brainstorming-stage code audit + elevates ODW (Origin Detection Window — see BRAIN15.24 for the framework) as the central organizing concept.

**Goal:** Make HMM the first major forward implementation priority within the clean architecture established in Phase 10–14. Complete missing HMM analysis families inside the HMM subtree only. Bring HMM to **parity** with Growth and RMS post-amplicon-calling analyses where parity is meaningful, while also adding **HMM-specific analyses** that only HMM can offer (state path, summit state intervals, nesting-level structure).

**Required outcome:**

- HMM has a concrete completeness matrix across the already-recognized analysis families.
- Missing HMM analyses are classified as reusable or HMM-specific.
- HMM completeness advances without reintroducing deprecated ownership patterns.
- The old late-Phase-10 HMM carry-over items that are still near-term priorities have an active home here (see BRAIN15.8, BRAIN15.13) rather than living only in historical or brainstorm surfaces.
- ODW (Origin Detection Window — onset_stage to last_active_stage; see BRAIN15.24) is the unifying lens for any analysis whose accuracy depends on confining inputs to active-firing stages: summit refinement, summit estimation, sliding-offset, fork-travel boundary detection, timing, etc.

#### Analysis families × HMM step inventory (Round 4 gap audit)

The Round 3 brainstorming-stage code audit compared HMM step inventory ([onionskin_core/output_layout.py:281-301](onionskin_core/output_layout.py#L281-L301)) against growth-model and RMS step inventories. **HMM has 5–6 gaps relative to growth/RMS that this BRAIN entry's matrix tracks; many are covered by other Phase 15 BRAIN entries already.**

| Analysis family | Growth has | RMS has | HMM has | Gap status & coverage |
|---|---|---|---|---|
| **Shape filter / annotation** | `02-growth-model/03-shape-filter/` | `03-rcn-mean-shift/06-shape-filter/` | NO (shape filter not wired for HMM) | **Gap.** Covered by BRAIN15.18 (HMM shape-score wiring + multistage unification + state-path evolution). |
| **Multistage unification** | (overlap-resolution at growth step 11) | `03-rcn-mean-shift/08-multistage-unification/` | NO (parts in HMM steps 7-8 collapsedHMM + aboveBackground but no unified step) | **Gap.** Covered by BRAIN15.18 sub-priorities (a)+(b)+(c)+(d)+(e). |
| **Signal-tracks / peak-summary** | `02-growth-model/07-signal-tracks/` (with `--peak-summary`) | (planned per `--peak-summary` help) | NO | **Gap.** Covered by BRAIN15.31 (Round 4 — `--peak-summary` extension to RMS + HMM + max-projection variants, per `tracking/KNOWN_ISSUES.md [ISSUE:2026-04-28:3]`). |
| **Width progression** | `02-growth-model/05-progression/` | NO | Partial — `12-fork-travel/` and `15-timing/` capture some progression-like data, but no dedicated progression file matching growth's | **Gap.** Per Q19 user direction: produce HMM-equivalent both as state-path-derived (HMM-specific, leading-edge of state 1; per-doubling-round forks) AND as growth-analogous where it produces meaningfully different results. Folded into this entry's "fork progression" family. |
| **Gap-analysis post-call annotation** | `02-growth-model/12-gap-analysis/` | `03-rcn-mean-shift/09-gap-analysis/` | NO (HMM step-3 `removeZeroBins/` is the HMM-equivalent for ratio constraint; not the same as post-call gap distance annotation) | **Gap.** Covered by BRAIN15.21 (Lift shared gap mask + missingness diagnostic to pre-pipeline step). |
| **Plots step (dedicated)** | `02-growth-model/14-plots/` (`aps/`, `growth_curves/`, `genome_overview/`) | `03-rcn-mean-shift/11-plots/` | NO (only `notebooks/` subdir exists under HMM) | **Gap.** Covered by BRAIN15.17 (APS plots/parity catalog), with structural fix per `[ISSUE:2026-04-28:1]`: **all pipelines remove step-numbers from plots/ + notebooks/ — they're "step-less" directories** (Q18 user direction). |
| **APS analyses & sub-features** | `02-growth-model/13-aps/` (matrices, cluster assignments, ordering tables; samples/, genome_stage_medians/) | `03-rcn-mean-shift/12-aps/` (same shape) | YES — HMM step 14 `aps/` exists, but with the step-14 raw-manifest bug ([ISSUE:2026-04-19:1]) | **Partial gap.** Bug covered by BRAIN15.6; analysis-surface parity covered by BRAIN15.17 + BRAIN15.20 (SAPS) + BRAIN15.27 (composite multi-feature APS modes); area-form decision by BRAIN15.14. |
| **Timing analyses** | `02-growth-model/10-timing/` | (likely planned) | YES — HMM step 15 `timing/` | Largely present; refinement covered by BRAIN15.29 (summit↔timing one-pass). |
| **Summit refinement** | `02-growth-model/09-summit-refinement/` | `03-rcn-mean-shift/07-summit-refinement/` | YES — HMM step 13 `summit-refinement/` | Present; sliding-offset adoption + ODW-confined input covered by BRAIN15.8 (peak_rcn_stage + sliding-offset; retitle pending — see BRAIN15.24 Round 4 update). |
| **Amplicon metrics** | (in growth `02-calls/` summary) | `03-rcn-mean-shift/10-amplicon-metrics/` | YES — HMM step 11 `amplicon-metrics/` | Present in HMM's structure. |
| **Fork tracking** | NO (growth doesn't track fork age/level structure) | NO | YES — HMM step 12 `12-fork-travel/` (with `fork_age_metrics.tsv`, `level_emergence_stage`, `ghost_level_flag`, etc.) | **HMM-specific.** Already implemented per [tracking/BRAINSTORM.md [2026-04-09] HMM fork travel and analysis plots — review COMPLETE](#); audit for Phase 15 spirit (parity + completeness) per Q19 + tracking note. |
| **Fork age** (alias for fork-tracking) | NO | NO | YES — same dir as fork tracking | Already implemented; BRAIN15.32 was proposed in Round 3 but **AUDIT-DISPROVED in Round 4** — see BRAIN15.32 RESOLVED marker. Audit only for enhancements per Q20 user direction. |
| **Elongation asymmetry** | NO | NO | YES — HMM step 12 emits asymmetry metrics; `tracking/BRAINSTORM.md [2026-04-09] HMM right-side asymmetry bias` is a known design-question | **HMM-specific.** Active question on the diagnosis — Viterbi-direction artifact vs real biology. Possibly fold into BRAIN15.18 fork-travel completeness work as a diagnostic axis. |
| **Posterior grouping / order / manifest** | YES (via APS clustering) | YES (via APS clustering) | Partial — HMM has step 14 `aps/` + step 16 `clustering/`, but uses `elbow` + `area` clustering defaults that may not be optimal | Covered by BRAIN15.19 (clustering defaults finalization) + BRAIN15.27 (composite multi-feature APS clustering modes) + BRAIN15.18 (multistage unification → final amplicon set that drives posterior groupings). |
| **HMM-specific quality criteria** (state-path-evolution; ghost levels) | N/A | N/A | YES — designed in `tracking/BRAINSTORM.md [2026-04-18] HMM-native amplicon quality criteria` | **HMM-specific.** Covered by BRAIN15.18 sub-priority (c). |
| **HMM-specific SAPS** (state-path APS) | N/A | N/A | NO (per-sample state-path outputs not yet emitted; SAPS not implemented) | **HMM-specific gap.** Covered by BRAIN15.20 (SAPS implementation, gated on BRAIN15.6 parallel child pipeline). |

#### Phase 15 deliverable surface (the matrix as a checklist)

When Phase 15 closes, **HMM should be self-sufficient when run alone** (per the SOUP-stage user direction "the HMM pipeline should offer all analyses, outputs, plots, and notebook analyses offered by any other pipeline"). The matrix above defines the deliverable surface: each row is a deliverable target, with the BRAIN entry that owns that piece of the work.

**Cross-pipeline-rewiring scope:** Phase 15 doesn't only add HMM-specific work — some completeness gaps (`--peak-summary`, plots/ + notebooks/ step-less convention, gap-mask shared pre-pipeline lift) require parallel changes in growth and RMS to maintain consistency. BRAIN15.21 + BRAIN15.31 + the step-less plots/notebooks rule are explicitly cross-pipeline.

#### ODW (Origin Detection Window) as the central organizing concept

For any HMM analysis whose accuracy depends on confining inputs to active-firing stages, the ODW framework (BRAIN15.24) is the unifying lens. This includes:

- **Summit refinement** (BRAIN15.8): use ODW-confined input, not all-stages-average.
- **Summit-then-timing-then-updated-summit one-pass convergence** (BRAIN15.29): step 3 "active stages" = ODW.
- **Cross-pipeline ODW design** (BRAIN15.22 / IBM-C5): brings the same window concept to growth and RMS.
- **Fork-travel boundary detection** (this entry / `12-fork-travel/`): ODW boundaries inform what's biologically meaningful in the trajectory.
- **Timing `onset_stage`** (timing module): `onset_stage` IS the ODW start.

Future-phase note: the ODW concept may extend to per-pipeline timing module refactor (per `tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` IBM-C6D: timing-guided active-stage weights) — folded into BRAIN15.29 design space.

#### Implementation plan

1. **Use the gap-audit table above as the Phase 15 deliverable checklist.** Each row's BRAIN entry owns that piece; this entry serves as the umbrella + cross-reference index.
2. **For each family classified "Gap" or "Partial gap":** the owning BRAIN entry's deliverables fill the gap. Classification (already complete / missing-but-reusable / missing-and-HMM-specific) is per the table above.
3. **Implement missing HMM analyses inside the HMM subtree** (per BRAIN15.2 design rule 1). Cross-pipeline-rewiring items (gap-mask shared mask, --peak-summary cross-pipeline, plots/+notebooks/ step-less) are explicit cross-pipeline scope and touch growth + RMS too.
4. **Reuse shared modules where appropriate** — `dBIC_flat_vs_tri` (shape filter), `dedup_calls_by_peak_proximity()` (already shared), `annotate_calls_with_gap_distance()` (will become shared via BRAIN15.21), etc.
5. **Do not block HMM completeness on per-stage completeness, growth parity, or synthesis design** — Phase 15 produces HMM-completeness; cross-pipeline synthesis (`04-unified-results/`) is a separate later phase.
6. **Treat summit-refinement BED outputs as part of the active HMM completeness surface** (already resolved at v0.10.41 — preserve regression coverage).
7. **Treat the trimmed-mean smoothing follow-up** (resolved v0.10.39) and smoothing-flag semantics audit as near-term HMM analytical work rather than as distant backlog.
8. **Apply the step-less plots/+notebooks/ convention cross-pipeline** per Q18 user direction + `[ISSUE:2026-04-28:1]`.

### BRAIN15.6 — HMM parallel child pipeline + SAPS + step-14 APS raw-file fix (HIGH)

**Source:** SOUP15.6, IBM-C2, [ISSUE:2026-04-19:1] HMM step-14 APS reads raw manifest bedGraphs, tracking/BRAINSTORM.md [2026-04-19] HMM parallel child pipeline (per-sample individual decoding) — HIGH PRIORITY.

**Priority: HIGH.** Prerequisite for the step-14 APS raw-file bug fix, SAPS, and amplicon reliability scoring (BRAIN15.7).

**Goal:** Run per-sample HMM steps 3–11 in independent `indiv_samples/` subdirectories so that downstream APS and future SAPS operate on correctly normalized, per-sample track files rather than on raw manifest bedGraph files.

**Background — step-14 APS normalization bug:** `run_step14_hmm_aps()` currently calls `compute_sample_rcn_tracks(manifest, ...)` which reads raw bedGraph files from the manifest and re-normalizes them independently of the HMM pipeline preprocessing. This is inconsistent with HMM steps 1–13: APS operates on un-preprocessed signal while everything else uses the HMM-normalized, smoothed, zero-bin-removed tracks. Fixing this requires the `indiv_samples/` outputs from step-5 to exist at APS invocation time. See [ISSUE:2026-04-19:1] for the full diagnosis.

**Required outcome:**

- Steps 03 through 11 each emit per-sample outputs under `<step>/indiv_samples/<sample_id>.*` alongside existing joint outputs.
- Step-14 APS reads from `05-medianSmoothed-RCN/indiv_samples/` instead of re-reading raw manifest bedGraph files and re-normalizing independently.
- A new step-15 SAPS (State-APS) reads decoded HMM state paths from `06-HMM/indiv_samples/` for per-sample state-level scoring (see BRAIN15.20 for SAPS specifics).
- Step renumbering to accommodate SAPS: timing → `16-timing/`, clustering → `17-clustering/`.
- The step-14 APS raw-file re-normalization bug is closed by this change.

**Implementation plan:**

1. Extend steps 03 through 11 to write per-sample outputs under `indiv_samples/` subdirectories alongside existing joint outputs.
2. Update step-14 APS to read from `05-medianSmoothed-RCN/indiv_samples/` instead of raw manifest files.
3. Add step-15 SAPS using per-sample HMM state paths from `06-HMM/indiv_samples/` (full design in BRAIN15.20).
4. Renumber timing → `16-timing/` and clustering → `17-clustering/`.
5. Update all path references, helpers, and inspector utilities affected by step renumbering.
6. Add regression tests covering the `indiv_samples/` output contract.

#### Round 6 update (2026-04-28) — concrete file targets for the step-renumbering sweep (per Codex audit)

The Codex Agent 2 brainstorming-stage code/tracking audit (2026-04-28; see FEEDBACK § "Agent 2 brainstorming-stage code/tracking audit") enumerated specific live code surfaces the SPEC engineering stage will need to touch. Captured here so SPEC engineering inherits the concrete file checklist:

- **`onionskin_core/output_layout.py:281-300`** — currently maps HMM step14/15/16 to `14-aps`, `15-timing`, `16-clustering`. After BRAIN15.6 + BRAIN15.20 (SAPS at step 15), renumber to `14-aps` (unchanged), **new** `15-saps/`, `16-timing/`, `17-clustering/`, plus the step-less `plots/` and `notebooks/` per BRAIN15.17.
- **`onionskin_core/engines/hmm_engine.py:1236-1268`** — imports and runs `run_step14_hmm_aps`, `run_step15_hmm_timing`, and `run_step16_hmm_clustering`. After renumbering, function names update: `run_step14_hmm_aps` (unchanged), **new** `run_step15_hmm_saps`, `run_step16_hmm_timing`, `run_step17_hmm_clustering`. Internal call-site references update.
- **`onionskin_core/hmm_notebooks.py:84, :111, :122`** — hardcodes `12-fork-travel`, `14-aps`, `15-timing`, `16-clustering`. After renumbering: `15-timing` → `16-timing`, `16-clustering` → `17-clustering`. Per BRAIN15.17, also drop step-numbers from `plots/` and `notebooks/` and migrate notebook generator outputs to step-less paths. The function name itself (`write_hmm_phase9_notebooks` per `hmm_engine.py:1269-1270` callsite) is stale — see BRAIN15.33 for the rename + modernization deliverable.
- **`tests/test_pipeline.py:179`** — asserts the help string includes `[hmm: 14-aps; growth: 13-aps; rms: 12-aps]`. After renumbering, the assertion stays at `14-aps` for HMM (step-14 APS doesn't move; only step-15+ does). No test change unless help-string text is also updated for the SAPS step insertion.
- **`scripts/summit_inspector.py:153-158`** — searches `hmm_steps['step15']/hmm_summit_estimates.tsv`. After renumbering: `step15` → `step16` (since timing moves from step 15 to step 16 per the SAPS insertion). This script is also flagged separately in BRAIN15.33 for a ModuleNotFoundError fix; bundle the step-renumber update into that fix.

**Typo fixed (2026-04-28 Round 6):** earlier rounds wrote `05-medianSmoothedRCN` (no hyphen between "Smoothed" and "RCN"); the actual HMM step-5 directory is `05-medianSmoothed-RCN` (with hyphen) per `onionskin_core/output_layout.py:288`. Both occurrences in BRAIN15.6 corrected.

**Downstream dependencies (work blocked until BRAIN15.6 lands):**

- BRAIN15.7 (amplicon reliability scoring) wants per-sample per-stage step-5 outputs.
- BRAIN15.8 (`peak_rcn_stage` + sliding-offset for HMM) wants per-sample per-stage step-5 outputs.
- BRAIN15.20 (SAPS) is gated on per-sample state-path outputs from step-6.

### BRAIN15.7 — Amplicon reliability scoring + flat-sample detection (pre-APS) (HIGH)

**Source:** SOUP15.7, SOUP15.12, IBM-C3, [ISSUE:2026-04-14:1] Stage-1 pre-amplification isolation, [ISSUE:2026-04-19:2] Amplicon reliability scoring (PROMOTED to PHASE11_SPEC 11.5 but never landed), tracking/BRAINSTORM.md [2026-04-19] Amplicon reliability scoring + flat-sample detection.

**Priority: HIGH.** Pre-APS diagnostic pass. Required before continued APS cluster validation and before SAPS (BRAIN15.20) can be interpreted meaningfully.

**This BRAIN entry consolidates SOUP15.7 and SOUP15.12** because both describe the same pre-APS reliability + flat-sample detection priority — the SOUP itself notes "These three items merge into a single future-phase priority: pre-APS reliability scoring, flat-sample detection, stage-1 anchoring."

**Goal:** Score each called amplicon on evidence strength across stages before APS clustering, and detect pre-amplification (flat) samples before they distort clustering.

**Background:** APS clustering currently receives amplicons without any pre-flight evidence filter. Amplicons with weak or unreliable multi-stage signal, or runs that include unamplified control-like samples, produce misleading APS cluster structure. A pre-APS reliability pass fixes this before cluster scores are computed.

User-quoted context for the late-Phase-15 priority (from Phase 14 Supplemental Q22 answer): *"Part of that will also be identifying truly 'flat' non-amplification samples for 'stage 1', then clustering the rest into automated groups."*

**Required outcome:**

- Each amplicon is scored on five evidence axes prior to APS:
  - RCN increase across stages
  - Width increase across stages
  - Area increase across stages
  - Triangle BIC increase across stages
  - Parabola height increase across stages
- A composite reliability flag is derived from these axes per amplicon.
- Flat (pre-amplification / unamplified) samples are detected and flagged before APS clustering runs, so APS scores are not distorted by samples that show no amplification trajectory.
- A `--aps-preamp-threshold` (or equivalent) controls the flat-sample detection cutoff. (See [ISSUE:2026-04-14:1] for the user-proposed `auto` mode using gap detection.)
- Scores and flags are emitted as a sidecar TSV alongside APS outputs.
- APS clustering operates only on samples and amplicons passing the reliability threshold.

**Implementation plan:**

1. Implement a pre-APS evidence-scoring pass operating on per-amplicon stage-structured RCN, width, area, BIC, and parabola-height data.
2. Derive a per-amplicon reliability composite flag.
3. Implement flat-sample detection using the same evidence axes.
4. Expose `--aps-preamp-threshold` to control the flat-sample detection cutoff.
5. Emit reliability scores and flat-sample flags as sidecar outputs.
6. Wire APS clustering to use only amplicons and samples passing reliability thresholds.
7. Validate on real multi-stage data that cluster structure improves vs. unfiltered APS.

**Dependency:** Wants per-sample per-stage step-5 outputs from BRAIN15.6 to compute the evidence axes correctly.

#### Round 4 expansion (2026-04-28) — fold IBM-C8B + IBM-C12 + IBM-C14 + tracking/BRAINSTORM.md [2026-04-01] design content

Per Q16 user direction: *"yes, absolutely merge the ideas and enrich them with all prior planning notes. try to use verbatim notes when possible to avoid drifting away from the ideas, especially notes written by the human."*

**Sources folded in:**

- `tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` IBM-C8B (Stage-1 pre-amp isolation — principled flat-sample detection)
- `tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` IBM-C12 (APS locus diagnostics — `best_onset_stage` / `post_support` / `dip_rate`)
- `tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` IBM-C14 (Keep/exclude recommendation enhancements — post-Priority 4.9)
- `tracking/BRAINSTORM.md [2026-04-01] APS locus diagnostics — design discussion and open questions` (the substantive Phase-5-era user-voice design thread that anchors all three IBM items above)
- `tracking/BRAINSTORM.md [2026-04-07] Keep/exclude recommendation enhancements (from Priority 4.9)`

**IBM-C8B framing (verbatim from `tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md`):** *"APS clustering currently cannot force pre-amp samples into a specific stage. User spelled out a principled approach based on area-excess thresholds and/or triangularity scoring (amplicon-set refinement + sample-set refinement). [...] IBM-C3 (amplicon reliability scoring) is the same biological problem viewed from a complementary angle. Implement both together or decide one supersedes the other. Confidence: high (concrete algorithmic design in KNOWN_ISSUES; clear biological motivation)."*

The IBM-C8B / IBM-C3 overlap means **BRAIN15.7 covers both angles of the same biological problem**: amplicon-set refinement (which amplicons are reliable) AND sample-set refinement (which samples are non-pre-amplification). Both refinements run before APS clustering.

**IBM-C12 framing (verbatim from `tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md`):** *"Three per-locus diagnostic signals that were designed but left as placeholders/partial implementations: `best_onset_stage` — the stage at which a locus becomes credibly amplified (partially done via v0.5.47–49 oscillation annotation); `post_support` — per-locus posterior support for amplicon credibility; `dip_rate` — signal for summit RCN declining in late stages (potential elongation-only phase marker). Some related work landed (oscillation annotation). The per-locus reliability weighting goal was explicitly marked 'off ROADMAP as of v0.5.49' — but this is an agent-flagged move not necessarily a user approval."*

**IBM-C14 framing (verbatim from `tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md`):** *"Priority 4.9 (integrated keep/exclude recommendations) was DONE v0.4.13. Additional refinements moved to BRAINSTORM.md `[2026-04-07] Keep/exclude recommendation enhancements (from Priority 4.9)`. Scope: Shape-score proximity to threshold as a soft signal; Timing flag (late + fragmented = suspicious); `--keep-amplicons-bed` user override for coordinates that are always retained. Status: Original 4.9 done; enhancements never picked up. Confidence: medium (concrete enhancements; clear user motivation; low-risk additions)."*

User's Round 3 Q15 framing puts this all in the ODW context — see BRAIN15.24 Round 4 update for the four-metric framework and the elongation-interference phenomenon. The ODW is the natural domain for "credibility" reasoning: the further outside the ODW a signal lies, the less reliably it represents origin firing.

#### Verbatim user content from `tracking/BRAINSTORM.md [2026-04-01] APS locus diagnostics`

**On `best_onset_stage` (verbatim):** *"Biological intent: when did the replication origin first fire? This is the first stage where the amplicon shows meaningful signal — NOT the stage of biggest area jump. Why area_excess is wrong for onset detection: Area_excess = integral of (RCN-1) × bin_width across the locus. This grows with fork travel distance, which continues to increase every stage as forks move further from the origin. The absolute area jump is therefore almost always largest at the last stage, regardless of when the origin actually started firing. Using area_excess to find onset would misassign `best_onset_stage = last_stage` for nearly every locus. What the right signal is: summit RCN (RCN at the replication origin itself). Summit RCN reflects origin-firing directly — it rises when new initiation rounds begin and plateaus or declines when they stop. Fork elongation adds area but does not substantially increase summit RCN."*

**On stage-1 detection (verbatim, Round 4-relevant):** *"Single-stage mode detects amplification relative to background, not relative to a pre-amplification state. Multistage mode adds temporal context but to detect if stage 1 is the earliest onset_stage among the samples, we need to test stage 1 similar to single-stage mode: relative to background (e.g. relative to chrom-median)."* — This is the same "stage 1 growth IS detectable against background" rule that's central to BRAIN15.24 + BRAIN15.29.

**On `dip_rate` (verbatim):** *"Biological intent: flag loci whose summit RCN is inconsistent after onset — either flickering (suggesting a spurious or marginal detection) or declining in late stages (suggesting nuclease degradation of the DNA in late salivary gland stages). Why this matters biologically: In DS1 (9 stages), stages after ~5 are predominantly elongation-phase stages — new initiation rounds have largely stopped and existing forks are traveling outward. In the latest stages, salivary gland cells may begin to be broken down; nucleases nonspecifically digest the genome. Because amplified regions have more copies, they initially lose more absolute DNA, but the *ratio* (RCN) can decline — summit RCN may drop in the latest stages even though the locus was genuinely amplified earlier."*

**On the design principle (verbatim, user-stated):** *"One of the founding goals of onionskin is to detect late, small, hard-to-detect amplicons — broad triangular domains that are highly blended with background, possibly amplified in only a subset of cells. The locus credibility weight must NOT penalize low absolute RCN. It should only penalize inconsistency, and only when that inconsistency is statistically supported given the within-stage variance."*

**On variance-aware credibility (verbatim):** *"With only 2 replicates per stage, per-stage mean `peak_rcn` has high sampling variance. Stage-to-stage oscillations in the metrics are dominated by within-stage replicate noise, not biological signal. Any formula using point estimates of per-stage means to detect 'dips' will misfire on real data at typical sample sizes. Absolute thresholds (2.0, 1.5) conflate signal strength with credibility. A consistently weak locus (peak_rcn = 1.4 every stage) should have high credibility — it's reliably there, just at low level. The formula penalizes signal strength, which is already captured by `area_excess`. This double-penalizes weak-but-real amplicons."*

**On per-stage variance prerequisites (verbatim, RESOLVED v0.5.47):** *"Per-stage MAD tracks: ✓ DONE (v0.5.47) — `stageN.MAD_RCN.bedGraph` and `stageN.MAD_log2RCN.bedGraph` now written alongside existing median tracks. N=1 → MAD = 0.0 (safe). Verified sensible on DS1 chr II."* The MAD machinery is already there for variance-aware reliability formulas.

**Calibration amplicons (verbatim):**

> *"Calibration amplicons (DS1 chr II, manifest.dataset1.5000bp.tsv):*
> *- chrII:45755000-46620000 (II:45.8-46.6, 865 kb) — genuine weak amplicon, confirmed by IGV + DS2 HMM; fork fronts visible; must not be penalized for low RCN*
> *- chrII:54380000-55152500 (II:54.4-56.1, 772 kb) — real*
> *- chrII:21440000-22737500 (II:21.4-23.5, 1297 kb) — real, twin-peak training pair*
> *- chrII:40660000-42530000 (II:40.7-42.5, 1870 kb) — real, genuinely high variance"*

These amplicons are the validation set for any reliability-scoring formula in BRAIN15.7 — all four must receive weight ≈ 1.0. A formula that penalizes any of them is broken.

#### Already-implemented machinery (do NOT duplicate; build on top)

- **Per-stage MAD tracks** (`stageN.MAD_RCN.bedGraph`, `stageN.MAD_log2RCN.bedGraph`) — implemented v0.5.47.
- **Probabilistic oscillation test** — implemented v0.5.48 (Gaussian overlap P(RCN_s > RCN_{s+1}) = Φ(z), σ = MAD × 1.4826, multi-bin summit window).
- **Oscillation annotation columns** — implemented v0.5.49 in `aps_stage_regression_report.tsv`: `is_oscillator`, `max_rcn_stage`, `regression_stage`, `oscillating_transitions`. (See BRAIN15.24 for details on `max_rcn_stage` and `regression_stage` semantics.)
- **`locus_weight = 1.0` placeholder** — Priority 5.2 closed as annotation-only at v0.5.49 because *"no discriminator between 'spurious oscillator' and 'real oscillating amplicon' was found with available data."* All four calibration amplicons showed statistically confirmed oscillations — the test correctly identifies real biology, not noise.

#### Multi-stage unification overlap (cross-reference to BRAIN15.18)

User-quoted from FEEDBACK Round 3 IBM-C3 disposition: *"Amplicon reliability scoring is the same in spirit to the multistage unification final shape classification in the RMS pipeline, which should be ported to the HMM pipeline, as well as the HMM-specific variant I described elsewhere that follows state path evolution."*

So BRAIN15.7 and BRAIN15.18 (HMM shape-score wiring + multistage unification + state-path evolution) are tightly coupled:

- **BRAIN15.18** does the per-amplicon classification (amplicon vs collapsed-repeat) using triangle-based + state-path-based methods + the 4-mode combination flag.
- **BRAIN15.7** does the per-amplicon reliability scoring on top of the BRAIN15.18 final classification.

Implementation order: BRAIN15.18 lands first (final amplicon set); BRAIN15.7 scores reliability on that final set.

#### Keep/exclude recommendation enhancement candidates (from IBM-C14 + tracking/BRAINSTORM.md [2026-04-07])

Verbatim list of possible enhancements (from `tracking/BRAINSTORM.md [2026-04-07] Keep/exclude recommendation enhancements`, all flagged "not committed"):

- **Graduated gap thresholds:** `>0.20` warn, `>0.35` reject unless rescued (currently binary at 0.20)
- **Gap count weighting:** 3–5 short gaps more suspicious than 1 long optical map gap
- **Tighter rescue thresholds:** raise `_WIDTH_RESCUE_MIN_R2` to 0.6 or require minimum slope if false rescues observed
- **Incorporate `amplicon_class` from timing:** Founder/Constitutive should require stronger evidence to exclude
- **Incorporate `origin_confidence` from multistage refinement**
- **Shape-score proximity** to threshold as a soft signal
- **Confidence-weighted exclusion:** probabilistic rather than binary
- **Timing flag:** amplicons with large positive lag AND high gap fraction (late + fragmented = suspicious)
- **User override:** `--keep-amplicons-bed` (coordinates that are always retained despite flags)

These are the IBM-C14 candidates. Phase 15 implementation can pick which to land based on what the BRAIN15.7 reliability formula actually needs vs what's belt-and-suspenders.

#### Updated implementation plan (Round 4)

The earlier 7-step plan still holds. Phase 15 implementation now also:

1. **Lands the ODW concept (BRAIN15.24) first** — `last_active_stage` column + `onset_stage`-as-ODW-start metadata + ODW-confined inputs feed into BRAIN15.7's evidence axes.
2. **Builds on the existing oscillation/MAD machinery** rather than reimplementing.
3. **Tests against the 4 DS1 chr II calibration amplicons** as a regression gate — all must come out with weight ≈ 1.0 in any new formula.
4. **Honors the founding design principle** that the formula must NOT penalize low absolute RCN — only inconsistency, and only when statistically supported given within-stage variance.
5. **Coordinates with BRAIN15.18 multistage unification** — operates on its final amplicon set.
6. **Adds `--keep-amplicons-bed`** user override (from IBM-C14) so the user can pin loci that the reliability formula would otherwise exclude.

#### Round 5 expansion (2026-04-28) — amplicon-class taxonomy: known live timing.py implementation to audit + harmonize (per Codex Round 6 audit, was originally framed as code-archaeology)

Per Q23 user clarification (Phase 15 FEEDBACK Round 4 answers): *"we previously developed methods to classify amplicons as things like 'pioneers' or something like that, and 'constitutive' vs 'faculatative'... I forget all the terms, but it was definitely related to or a precursor to the ODW concept, and we already had developed methods to determine whether there was growth between stages or not WITHIN the ODW. That is to say that it is possible to have stages within the ODW that do not show summit height since the last stage, but summit growth continues later in the window. You might want to find this code, and these ideas and outputs to see what is useful, and to make sure we don't redevelop the same ideas again, etc."*

**Round 6 update (per Codex Agent 2 brainstorming-stage code/tracking audit, 2026-04-28):** the amplicon-class machinery is **already implemented and live** in `onionskin_core/timing.py`. This is no longer an "archaeology / does it exist?" question — Codex's audit located the implementation. BRAIN15.7's deliverable shifts from "find existing classification machinery" to "audit the LIVE timing.py classification machinery; port to HMM where needed; decide harmonization with ODW terminology."

**Live timing.py implementation (verbatim findings from Codex audit):**

- `onionskin_core/timing.py:300-317` — output schema includes `latest_activity_stage`, `activity_breadth`, and `amplicon_class` columns.
- `onionskin_core/timing.py:370-393` — stage-over-stage active-growth detection uses a **1.25 fold-change threshold after onset**. This IS the within-ODW growth detection method the user described (Q23: *"we already had developed methods to determine whether there was growth between stages or not WITHIN the ODW"*). The 1.25 threshold is the magic number used today.
- `onionskin_core/timing.py:466-468` — output rows write `latest_activity_stage` and `activity_breadth` columns.
- `onionskin_core/timing.py:511-527` — `_classify_amplicon()` emits the six class labels: `Constitutive_Prolonged`, `Facultative_Prolonged`, `Founder`, `Finisher`, `Intermediary`, and `Excluded`. **These are the user-recalled "pioneers / constitutive / facultative" terms** (with refinement: Constitutive and Facultative both have `_Prolonged` suffix; "pioneer"-style class is split into `Founder` and `Finisher`; `Intermediary` is a fourth phenotype).

**Implication for BRAIN15.7's deliverable:**

Phase 15 implementation should:

1. **Audit the existing timing.py classifier** — does the 1.25 fold-change threshold work robustly across datasets? Does the six-class taxonomy capture the developmental phenotypes well? Are there known false-positives or false-negatives that would benefit from refinement during Phase 15?
2. **Port to HMM** — the timing.py classifier currently runs on growth/RMS pipeline outputs. Phase 15 BRAIN15.6's HMM parallel child pipeline + BRAIN15.20 SAPS should provide the per-sample data needed for HMM-side classification. Decide whether to port the existing classifier to HMM as-is, or whether HMM's state-path data enables a richer classifier.
3. **Harmonize with ODW terminology (per BRAIN15.24)** — `latest_activity_stage` is the timing.py predecessor name for what the four-metric framework calls `last_active_stage`. **Decision (per user 2026-04-28 chat): rename `latest_activity_stage` → `last_active_stage` cross-pipeline.** Single canonical name; no synonyms; no "two names for the same thing" maintenance burden. The rename is contained to `onionskin_core/timing.py` (column emit + classifier inputs at lines 300-317, 466-468, 511-527) + downstream callers that read the column. Tracking-file mentions of `latest_activity_stage` get cleaned up as part of BRAIN15.34's Phase 15 closeout sweep.
4. **Per-class reliability scoring** — the BRAIN15.7 reliability formula can be class-aware: e.g., `Founder` and `Finisher` may have different evidence-axis weighting than `Constitutive_Prolonged`. Per `tracking/BRAINSTORM.md [2026-04-07] Keep/exclude recommendation enhancements`: *"Incorporate `amplicon_class` from timing: Founder/Constitutive should require stronger evidence to exclude."*
5. **Cross-pipeline harmonization check** — timing.py is shared across pipelines. Phase 15 confirms HMM uses the same classifier (or an HMM-specific extension) and emits the same column schema for cross-pipeline-comparable outputs.

6. **Bayesian Origin Detection Window (ODW) System — replace the 1.25× hard-threshold within-ODW growth-detection test with a variance-aware statistical framework with tunable knobs.** **Decision (per user 2026-04-28 chat): the 1.25× hard rule is officially being retired in favor of a probabilistic / variance-aware test.** All three candidate formulations are exposed as flag options; default is the Bayesian posterior. **The framework outputs feed multiple ODW metrics — `onset_stage`, `last_active_stage`, the per-transition active mask, amplicon-class classification — so the flag and its tunable knobs are part of a system, not just a single-metric detector.**

   **Background — the two existing tests in code:**

   - **1.25× fold-change** (`onionskin_core/timing.py:370-393`): asks *"did stage `s` show ACTIVE GROWTH compared to stage `s-1`?"* — binary mask using a fixed 1.25 fold-change threshold; defines `latest_activity_stage` (post-rename: `last_active_stage`) + amplicon class. **This is the test being retired.**
   - **Gaussian-overlap dip probability** (`onionskin_core/aps.py:748-1041`, v0.5.48 — uses `scipy.stats.norm.cdf` at line 993; sigma = MAD × 1.4826 from per-stage MAD bedGraphs at v0.5.47): asks *"is stage `s` STATISTICALLY HIGHER than stage `s+1`?"* — variance-aware probability per transition; feeds `regression_stage`, `is_oscillator`, `oscillating_transitions`. The variance-aware machinery already exists; the upward-direction variant is a one-line flip.

   **Candidate formulation A — Gaussian-overlap framework (test "any directional growth"):**

   - Reuse the existing `aps.py:991-994` machinery, flipping direction: `p_growth = norm.cdf((med_{s+1} - med_s) / denom)` where `denom = sqrt(sigma_s² + sigma_{s+1}²)` and `sigma = MAD × 1.4826`.
   - A transition counts as "active growth" if `p_growth > growth_prob_threshold` (e.g., 0.5, 0.9, 0.95).
   - Tests `H0: med_{s+1} = med_s` (no difference). Asks: "given the data, how likely is it that stage s+1 actually exceeds stage s in the underlying distribution?"
   - **Pro:** zero new machinery — direct reuse of the existing dip-probability infrastructure with reversed direction. Same MAD-derived sigma, same test family for both `last_active_stage` (upward) and `regression_stage` (downward).
   - **Con:** doesn't preserve the 1.25× biological-meaningfulness anchor — a tiny but statistically-clean growth signal (e.g., 5% with very low replicate variance) would register as "active." May be too permissive depending on threshold.

   **Candidate formulation B — Bayesian posterior P(ratio ≥ 1.25) (test "biologically meaningful growth"):**

   - Per-transition: model the log-ratio of medians as approximately Normal: `log(med_{s+1} / med_s) ~ N(observed_log_ratio, sqrt(sigma_log_s² + sigma_log_{s+1}²))` using per-stage `MAD_log2RCN` (already emitted at v0.5.47).
   - With a flat prior (or any reasonable conjugate prior), the posterior P(ratio ≥ 1.25 | data) = `1 - norm.cdf((log(1.25) - observed_log_ratio_mean) / observed_log_ratio_sigma)`.
   - A transition counts as "active growth" if posterior P(ratio ≥ 1.25) > posterior_threshold (e.g., 0.9, 0.95, 0.99 per user suggestion).
   - **Pro:** preserves the 1.25× biological-meaningfulness anchor (the threshold IS the meaningful effect size we care about); adds variance-awareness on top; user-stated preference for the high posterior thresholds (0.9 / 0.95 / 0.99) gives strong evidence requirements.
   - **Pro:** computationally similar to formulation A — single `norm.cdf` call per transition, just with a non-zero hypothesized log-ratio (`log(1.25)` instead of 0).
   - **Con:** the 1.25 threshold itself is somewhat arbitrary; a more principled approach might be to test multiple effect-size thresholds (1.25, 1.5, 2.0, etc.) and report the posterior at each. But for a single-threshold test, 1.25 is the historical anchor.

   **All three formulations are clean upgrades-or-fallbacks** with respect to the hard 1.25× threshold (B and the legacy hard-threshold are special cases of the same family — B = hard-threshold-with-variance-awareness; A is a variance-aware test against a different null). All leverage the existing MAD-derived sigma machinery from v0.5.47 + v0.5.48.

   **The Bayesian ODW System — concrete CLI surface (per user direction 2026-04-28):**

   **CLI architecture: new "Origin Detection Window" argparse group (decision per user 2026-04-28 chat).** The ODW flags form a **new dedicated argparse group**, NOT additions to the existing timing group. Rationale: ODW boundaries affect multiple downstream analyses (timing, APS, summit refinement, amplicon-reliability scoring, multistage unification, summit↔timing convergence) — not just timing. Adding the flags to timing would mis-suggest that they only affect timing outputs. The new ODW group follows the **APS Universal-in-spirit pattern** (per BRAIN15.16; Phase 14 Supplemental 14-S20): the group's header text explicitly enumerates the downstream analysis surfaces affected by these flags so users (and future agents) immediately understand the cross-cutting scope. Suggested header text:

   > *"Origin Detection Window (ODW) controls. The ODW is the per-amplicon active-firing stage window (`onset_stage` to `last_active_stage`) where summit signal is most reliable. These flags configure the within-ODW per-transition growth-detection test that defines the ODW boundaries. **Outputs feed timing analyses (onset_stage, last_active_stage, amplicon_class), summit refinement (BRAIN15.8 stage-selection strategies), amplicon reliability scoring, and multistage unification (within-ODW growth/no-growth detection).** Tightening the thresholds narrows the ODW to higher-confidence stages (better summit signal); loosening widens (better amplicon recall)."*

   The framework is exposed as **one detector-selection flag plus two tunable-knob sibling flags** that work across multiple detectors:

   | Flag | Default | Type | Description |
   |---|---|---|---|
   | `--odw-active-stage-detector` (or similar — name TBD by SPEC engineering; user suggested `--active-stage-detector` / `--odw-active-stage-detector`; the test isn't only for `last_active_stage` so the name shouldn't anchor on that single metric) | `bayesian_posterior` | choice: `{hardthreshold, gaussian_overlap, bayesian_posterior}` | Selects the within-ODW per-transition growth-detection test. |
   | `--odw-prob-threshold` (or similar) | `0.9` | float in `[0, 1]` | Probability threshold above which a transition counts as "active growth." Used by `gaussian_overlap` (as `p_growth > threshold`) and `bayesian_posterior` (as `posterior > threshold`). User-suggested values: 0.9, 0.95, 0.99. Ignored by `hardthreshold`. |
   | `--odw-fold-threshold` (or similar) | `1.25` | float | Min effect-size cutoff (fold-change). Used by `hardthreshold` (as `ratio >= threshold`) and `bayesian_posterior` (as `posterior P(ratio >= threshold) > prob_threshold`). Ignored by `gaussian_overlap` (which tests against the null of equality, not against an effect-size threshold). |

   The flags compose as expected:

   - `bayesian_posterior` (default) uses BOTH thresholds: posterior probability `>= prob_threshold` that effect size `>= fold_threshold`. Tightest combo.
   - `gaussian_overlap` uses ONLY `prob_threshold`: probability of any directional growth `>= prob_threshold`.
   - `hardthreshold` uses ONLY `fold_threshold`: deterministic check that `ratio >= fold_threshold`. Ignores variance.

   **Why both knobs exist (per user direction):** raising either knob makes the detector more conservative — useful for **confining the ODW to the most active stages with the best summit signal**, which directly affects BRAIN15.8 stage-selection performance. Loosening makes the detector more inclusive — useful for capturing weaker amplicons that wouldn't otherwise register as having an ODW. The two knobs are biologically and statistically distinct:

   - `prob_threshold` controls *evidence strength* (how confident must we be before we believe the growth is real?).
   - `fold_threshold` controls *effect-size meaningfulness* (how much growth must there be to count as biologically meaningful?).

   Tuning them independently lets users optimize for different downstream tasks (summit accuracy vs amplicon recall vs cluster discrimination, etc.).

   **Connection to BRAIN15.8 stage-selection (summit optimization):**

   BRAIN15.8's per-amplicon stage-selection menu (max_rcn_stage / onset_stage / last_active_stage / all-ODW / subset-ODW / narrowest-summit-state) consumes ODW boundaries that are themselves a function of these thresholds. Tightening the ODW (raise `prob_threshold` or `fold_threshold`) yields fewer-but-higher-confidence stages → typically better summit signal (less elongation interference). Loosening yields more stages → typically better summit recall but more interference. **The Bayesian ODW System is the primary tuning surface for summit optimization in Phase 15.**

   **Phase 15 SPEC engineering decision items (deferred to SPEC stage):**

   - Pick the canonical flag names (the table above uses placeholders; SPEC engineering finalizes — recommend prefix `--odw-` to group them).
   - Decide whether `regression_stage` should also be controlled by the same flag (currently uses Gaussian-overlap dip in `aps.py:993`; could be unified under the same `--odw-active-stage-detector` framework with reversed direction). Suggestion: yes — keep `regression_stage` and `last_active_stage` symmetric under the same statistical machinery, just opposite directions.
   - Cross-pipeline: timing.py is shared; the rewrite happens once and applies cross-pipeline (HMM, growth, RMS).
   - Test the impact of `--odw-prob-threshold` / `--odw-fold-threshold` tuning on summit accuracy (BRAIN15.8 evaluation) and amplicon-class distributions (BRAIN15.7).

**This is a code-archaeology task** — find what the project has already implemented for amplicon classification (pioneer / constitutive / facultative or similarly-named classes) + within-ODW growth-vs-flat detection, audit it, and decide per-implementation whether to:

- **Adopt:** existing implementation matches our needs; integrate it into BRAIN15.7's reliability framework + BRAIN15.18's multistage unification + BRAIN15.24's ODW machinery + BRAIN15.8's stage-selection strategies. Reuse rather than re-implement.
- **Refine:** existing implementation captures the spirit but needs improvement. Update the existing code; propagate to all callers.
- **Supersede:** existing implementation is legacy / suboptimal; new ODW-based methods replace it; deprecate the legacy code with a migration path.

**Audit scope (search candidates):**

- **Code:** grep `pioneer`, `constitutive`, `facultative`, `amplicon_class`, `amplicon_type`, `class_label`, `category` across `onionskin_core/`, `onionskin.py`, `tests/`, `scripts/`. Specifically check:
  - `onionskin_core/timing.py` (per `tracking/BRAINSTORM.md [2026-04-01]` mention of timing-derived `amplicon_class`).
  - `onionskin_core/aps.py` (oscillation annotation machinery v0.5.47-49 may include classification).
  - `onionskin_core/refinement.py` and `onionskin_core/rcn_mean_shift_helpers.py` (cross-stage growth detection).
- **Planning surfaces:** grep for the same terms in `tracking/BRAINSTORM.md`, `tracking/KNOWN_ISSUES.md`, `tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md`, archived phase plans. The `[2026-04-07] Keep/exclude recommendation enhancements` entry mentions *"Incorporate `amplicon_class` from timing: Founder/Constitutive should require stronger evidence to exclude"* — that's an existing-classification reference.
- **Per-stage growth/no-growth detection:** find the methods the user mentioned for detecting whether summit growth occurred between stages within the ODW. Likely candidates: `growth-curve` infrastructure, per-transition statistical tests in the timing module, oscillation annotation logic in `aps.py`.

**Deliverables of the audit (Phase 15 sub-priority of BRAIN15.7):**

1. **Code-archaeology report** — what classifications + growth-detection methods exist; where they live; their definitions; what they output.
2. **Decision per finding:** Adopt / Refine / Supersede (with rationale per item).
3. **Integration plan** — how each adopted/refined piece connects to the four-metric framework + ODW + multistage unification + reliability scoring.
4. **Cross-pipeline harmonization check** — if amplicon-class taxonomy lives in some pipelines but not others, decide whether to lift to controller-level or keep pipeline-specific.

**Connection to other BRAIN entries:**

- **BRAIN15.18 (HMM shape-score wiring + multistage unification):** within-ODW growth/no-growth detection feeds the multistage-unification classification. If existing detection methods are already wired for growth/RMS, port to HMM.
- **BRAIN15.24 (peak_rcn_stage audit / four-metric framework):** the existing `last_active_stage` detection (if any exists in code) IS the audit's step-1 finding under a different name. Cross-reference back.
- **BRAIN15.8 (HMM summit refinement / stage-selection strategy menu):** if existing classification or detection identifies which stages are "active" per amplicon (ODW boundaries), the stage-selection strategies should consume those instead of re-deriving them.

**Why this is a BRAIN15.7 sub-priority rather than its own BRAIN entry:** the audit's output (classification machinery + reliability machinery + ODW-aware filtering) all live inside BRAIN15.7's amplicon-reliability scope. Adding a new BRAIN ID just for "find existing classification code" would proliferate thin entries; folding into BRAIN15.7 keeps the substantive work consolidated. The audit itself is one of the items in BRAIN15.7's expanded SPEC priority.

**Updated Round 4-and-Round 5 implementation plan (consolidated):**

The earlier 7-step plan still holds. Phase 15 implementation now also (combining Round 4 + Round 5 expansions):

1. **First:** code-archaeology audit (Round 5 above) — find existing amplicon-class taxonomy + within-ODW growth/no-growth detection. Avoid re-developing what's already there.
2. **Then:** Lands the ODW concept (BRAIN15.24) — `last_active_stage` column + `onset_stage`-as-ODW-start metadata + ODW-confined inputs feed into BRAIN15.7's evidence axes.
3. Builds on the existing oscillation/MAD machinery (v0.5.47–49) rather than reimplementing.
4. Tests against the 4 DS1 chr II calibration amplicons as a regression gate — all must come out with weight ≈ 1.0.
5. Honors the founding design principle (must NOT penalize low absolute RCN; only inconsistency given within-stage variance).
6. Coordinates with BRAIN15.18 multistage unification (operates on its final amplicon set).
7. Adds `--keep-amplicons-bed` user override (from IBM-C14).

### BRAIN15.8 — HMM summit accuracy: per-amplicon stage-selection (incl. `max_rcn_stage`) + ODW-confined refinement + sliding-offset sub-bin

> **Title note (Round 5):** Originally framed as "`peak_rcn_stage` column + sliding-offset sub-bin refinement." Per Q23 user clarification, `peak_rcn_stage` is a retired synonym for `max_rcn_stage` (existing code column at `onionskin_core/aps.py:1016`). Title reframed to reflect the broader scope: per-amplicon stage-selection with multiple testable strategies (BRAIN15.8 Round 5 menu), of which `max_rcn_stage`-based selection is one. ODW-confined refinement (BRAIN15.24 framework) is the umbrella concept; sliding-offset sub-bin refinement is the resolution-improvement step that runs at whatever stage the selection strategy identifies.

**Source:** SOUP15.5 (carry-over queue items, split out from the umbrella), IBM-C4, tracking/BRAINSTORM.md [2026-04-14] HMM summit estimation — use stage of maximum summit RCN, not last stage; tracking/BRAINSTORM.md [2026-04-14] HMM summit refinement — sliding-offset sub-bin localization.

**Cross-reference:** **gated on BRAIN15.24 (`peak_rcn_stage` audit)** — adoption into HMM only proceeds after the audit clears or after any naming/semantics correction is propagated everywhere the term is currently used. See BRAIN15.24 for the audit's scope.

**Two sub-priorities, packaged together because they share the same dependency on per-sample per-stage step-5 outputs (BRAIN15.6):**

**(a) `peak_rcn_stage` column for HMM summit estimation.** Empirically, stages 2–4 give better origin localization than later stages because "elongation interference" shifts the summit peak laterally in later stages. The stage of maximum summit RCN should be identified per amplicon and used as the authoritative summit position source for step-14/15. A `peak_rcn_stage` column is to be added to `hmm_summit_estimates.tsv`.

**(b) HMM sliding-offset sub-bin summit refinement.** The growth pipeline uses `sliding_offset_profile` / `refine_origin_sliding_offset` in `onionskin_core/refinement.py` (v0.5.54–v0.5.56) for sub-bin localization without needing hires data. HMM does not yet have this. Should be applied at the `peak_rcn_stage` step-5 smoothed bedGraphs after BRAIN15.6 introduces per-sample outputs.

**Implementation order:**

1. **First:** BRAIN15.24 audit clears the term + semantics + name.
2. **Second:** BRAIN15.6 per-sample step-5 outputs land.
3. **Then:** add the `peak_rcn_stage` column (with whatever name the audit settles on).
4. **Then:** wire sliding-offset refinement at the per-sample-per-stage step-5 bedGraphs identified by `peak_rcn_stage`.

**Note from carry-over:** Potential split of `--hmm-smooth-halfwidth` into a separate APS halfwidth flag was a near-term carry-over item; this is **already RESOLVED in Phase 14 Supplemental cycle 14S.23 (v0.14.x)** as `--hmm-aps-smooth-halfwidth`. Tracked as [ISSUE:2026-04-19:3] (RESOLVED). See BRAIN15.13 for the resolution note.

#### Round 5 expansion (2026-04-28) — menu of per-amplicon stage-selection strategies for testing

Per Q23 user direction (Phase 15 FEEDBACK Round 4 answers): summit modeling for HMM should test **multiple per-amplicon stage-selection strategies**, not just a single one. The four-metric framework + ODW concept (BRAIN15.24) provides the strategy candidates. Each strategy answers the question *"for amplicon A, which stage(s) should drive the summit estimate?"* differently. Phase 15 implementation should make this configurable so all strategies can be evaluated against the by-eye + PuffStep gold-standard surfaces (per BRAIN15.17 by-eye eval references).

**Strategy menu (each becomes a configurable flag value or testable variant):**

1. **`max_rcn_stage`** (argmax of summit-window-median RCN across post-onset stages — already an existing column per BRAIN15.24) — *"clear instruction of how to choose a stage for summit modeling for each amplicon"* (user Q23). Trivial to test: just use the existing column. Bundles the [2026-04-14] HMM summit estimation idea — "use stage of maximum summit RCN, not last stage" — as one strategy on the menu.
2. **`onset_stage`** (first stage where summit growth is detected — already exists in timing module) — *"for each amplicon, see if the stage of earliest activity typically produces the best summit results with the various summit estimators (argmax, parabola, etc)"* (user Q23). Testable; expected to work well for amplicons whose origin firing is concentrated in early stages.
3. **`last_active_stage`** (last stage where growth is detected — new column per BRAIN15.24) — *"for each amplicon, see if the last stage growth is detected is a good stage"* (user Q23). Testable; a candidate where the summit profile may be most fully developed before elongation interference takes over.
4. **All stages within ODW** (use `onset_stage` to `last_active_stage` as the input window) — *"we can apply statistical approaches across the stages for summit estimation"* (user Q23). The default ODW-confined approach. Statistical aggregator candidates: per-stage parabola fits + median-of-vertices, weighted-mean by per-stage shape-confidence, etc.
5. **Subset of stages within ODW** — pick a subset based on per-stage shape quality. *"we can try to find a stage within that has the best triangle or parabola shape — that is the most pointy or peak-like, then use that one"* (user Q23). Quantitative metrics: dBIC_flat_vs_tri (already computed per stage); parabola curvature coefficient; goodness-of-fit residuals.
6. **HMM-specific: stage with narrowest summit-state interval** — *"for HMM, it could be the stage with narrowest summit state interval too"* (user Q23). HMM provides the summit-state-interval data natively (`onionskin_core/hmm_summits.py` machinery); stages where the summit-state interval is narrowest typically correspond to the most-pointy-summit stages. Distinct from the bin-resolution shape-quality metrics in strategy 5; the summit-state-interval metric is HMM-state-path-derived.

**Phase 15 deliverables for BRAIN15.8 (revised post-Round-5):**

1. **Implement the stage-selection strategy machinery** — likely a flag like `--hmm-summit-stage-selection {max_rcn,onset,last_active,odw_all,odw_best_shape,odw_narrowest_state}` (default TBD; suggestion: `odw_best_shape` once tested, with `max_rcn` as a documented fallback for legacy comparability).
2. **Evaluate all strategies** against the by-eye + PuffStep gold-standard surfaces (per BRAIN15.17 by-eye eval files: `tests/full_chrom_training_data/amplicons.by-eye.bed`, `tests/summit_training_data/01-II9A-evidence-based-confinement-boundaries-and-rules.bed`, `dev/puffstep/automate/`).
3. **Wire sliding-offset refinement** at whatever stage(s) the chosen strategy identifies — sliding-offset refines a chosen stage's profile to sub-bin resolution; it's downstream of stage selection.
4. **Report results across strategies** in the comparison framework so the user can pick the best default per dataset (or accept a single default if one strategy dominates across datasets).

**Implementation order updated post-Round-5:**

1. **First:** BRAIN15.24 audit clears the term + semantics + name (mostly done at framework level; SPEC engineering will fill in `last_active_stage` detection mechanics).
2. **Second:** BRAIN15.6 per-sample step-5 outputs land.
3. **Third:** Implement the stage-selection strategy machinery (per the menu above).
4. **Fourth:** Implement summit-stage selection per chosen strategy; wire sliding-offset refinement at the selected stage(s) per-sample-per-stage step-5 bedGraphs (BRAIN15.6).
5. **Fifth:** Evaluate strategies against by-eye + PuffStep eval surfaces; pick default(s).

**Bayesian ODW System tunables as summit-optimization knobs (per Round 6 user direction 2026-04-28):**

The ODW boundaries that strategies 2–6 consume are themselves a function of the Bayesian ODW System's tunable knobs (BRAIN15.7 deliverable item 6 + BRAIN15.24 four-metric framework). Specifically:

- `--odw-prob-threshold` (default 0.9) — evidence-strength knob.
- `--odw-fold-threshold` (default 1.25) — effect-size meaningfulness knob.

**Tightening either knob** (raise from defaults) → narrower ODW → fewer-but-higher-confidence stages enter strategies 4 (all-ODW), 5 (subset-best-shape), 6 (narrowest-state). Typically improves summit signal because elongation-interference-affected stages are excluded.

**Loosening either knob** → wider ODW → more stages → potentially better amplicon recall but more elongation interference in summit estimates.

**For BRAIN15.8's evaluation matrix:** Phase 15 should test **both** the per-amplicon stage-selection menu (strategies 1–6) AND a sweep over `--odw-prob-threshold` × `--odw-fold-threshold` values. The combined experimental matrix gives the best summit-accuracy-vs-recall tradeoff for the user's datasets. The Bayesian ODW System's knobs are the primary tuning surface for summit optimization in Phase 15.

**Important within-ODW non-monotonicity note (per Q23 user clarification):**

> *"we already had developed methods to determine whether there was growth between stages or not WITHIN the ODW. That is to say that it is possible to have stages within the ODW that do not show summit height since the last stage, but summit growth continues later in the window."*

This means **the ODW is NOT strictly monotonic growth.** Some stages within `[onset_stage, last_active_stage]` may show no growth from the previous stage but growth resumes later. The `last_active_stage` detector (per BRAIN15.24) must allow this — a single non-growth transition mid-window does NOT mean the window has ended; only sustained-flat-or-declining behavior closes the window. This connects to the within-ODW per-transition test design space discussed in BRAIN15.24's audit step 4-5 plan.

**Cross-reference to existing amplicon-class taxonomy code (audit task surfaced by Q23 — folded into BRAIN15.7):**

User Q23 noted: *"we previously developed methods to classify amplicons as things like 'pioneers' or something like that, and 'constitutive' vs 'faculatative'... I forget all the terms, but it was definitely related to or a precursor to the ODW concept, and we already had developed methods to determine whether there was growth between stages or not WITHIN the ODW. [...] You might want to find this code, and these ideas and outputs to see what is useful, and to make sure we don't redevelop the same ideas again."*

**Action:** the amplicon-class-taxonomy code-archaeology audit lives in BRAIN15.7 (amplicon reliability) Round 5 expansion — see that entry. Findings from the audit feed back into BRAIN15.8 strategy menu (e.g., if existing classification machinery already implements within-ODW growth detection, BRAIN15.8 strategy 4 / 5 should reuse it rather than re-implement).

---

## HMM CLI correctness

### BRAIN15.9 — CORRECTNESS BUG: `--hmm-mu-scale` + full PuffStep → onionskin HMM translation re-audit

**Source:** SOUP15.17, SOUP15.18, [ISSUE:2026-04-26:1] `--hmm-mu-scale` silently scales emission means instead of sigmas (PuffStep translation bug).

**This BRAIN entry consolidates SOUP15.17 ("True state means" — user musings about what mu-scale should/shouldn't do) and SOUP15.18 ("CORRECTNESS BUG" — formal bug report + re-audit demand).** They cover the same problem from two angles; consolidation is interpretive — see FEEDBACK for the consolidation note.

**The bug:** `--hmm-mu-scale` was translated from PuffStep incorrectly. In PuffStep, `--mu_scale` **never touched the means** — it was an alternative method for deriving sigmas: `sigma = mean * mu_scale` for each state (used when neither `--sigma` nor the sqrt-of-means default was desired). It was mutually exclusive with `--sigma`.

In onionskin, the translation agent reversed this: `--hmm-mu-scale` now **scales the means** (`scaled_means = [m * mu_scale for m in emission_means]`) and does nothing to sigmas. The sigmas are always taken from `--hmm-emission-sigmas` unchanged. The help text even claims "Also rescales the sigmas by the same factor" — which is doubly wrong (means are scaled, not sigmas; and sigmas are not touched at all).

Additionally, because `--hmm-emission-sigmas` always has a hardcoded default in onionskin (`0.25,0.5,1,2,4,6,8,24`), even a corrected implementation needs to rethink priority logic: in PuffStep, `mu_scale` was reachable only when `sigma` was `None`; in onionskin, sigma is never `None`. The fix requires both code correction and a design decision on override priority (`--hmm-emission-sigmas` explicit vs. `--hmm-mu-scale` derived).

The `--hmm-emission-sigmas` help text is also wrong: it cross-references `--hmm-mu-scale` as "sets sigma = 0.5 * mean for every state" — which has never been true in onionskin.

**User-side framing (from SOUP15.17):**

- The concept of ensuring the means actually make sense with the RCN values is a separate (and potentially valid) idea, but it is NOT what `--mu_scale` did in PuffStep.
- The PuffStep-descended behavior was strictly "scale means by a constant to create sigmas" — nothing more.
- A potentially valuable separate feature: learn the RCN mean values of background regions per chromosome and redo chromosome-specific normalization such that the median and/or mean of background regions = 1 as expected. The median norm and chromosome-specific median norm steps should already get close to this, but a verification pass might surface gaps. **This is a separate idea from fixing the mu-scale bug** — flagged for separate consideration; do not bundle into the bug fix.

**Required action: full PuffStep → onionskin HMM translation re-audit (Opus, Max Effort).**

This bug was introduced silently by an agent during the PuffStep → onionskin translation. It raises the question of whether other HMM flags were similarly mistranslated. A dedicated re-audit is required:

1. **Code correctness audit** — for every HMM engine flag in `onionskin.py:build_parser()` and every corresponding parameter in `onionskin_core/engines/hmm_engine.py`, verify that what the code actually does matches what PuffStep did. Pay particular attention to any flag whose behavior was described in terms of another flag (e.g., mu_scale described in terms of sigma), flags whose defaults changed, and flags whose types changed. Use `PuffStep/puffStep_core/hmm_fxns.py` and `PuffStep/puffStep.py` as the authoritative reference.
2. **Fix code first** — correct any mistranslated behavior before touching help strings.
3. **Help string audit** — after code is confirmed correct, audit all HMM help strings for accuracy. This includes removing false cross-references (e.g., the `--hmm-emission-sigmas` claim about `--hmm-mu-scale`).

**Assigned agent profile:** Opus, Max Effort. This is a correctness-sensitive audit that must not be delegated to a lighter model.

**Exit condition:** [ISSUE:2026-04-26:1] closes; HMM CLI semantics match PuffStep where they were intended to; documentation is accurate.

### BRAIN15.10 — Missing PuffStep flag synonyms to add

**Source:** SOUP15.18 (split out from the CORRECTNESS BUG entry — this is a separable CLI aliasing task).

PuffStep flags that were significantly renamed in onionskin currently have no synonym aliases. These should be added so users migrating from PuffStep do not need to translate their command lines:

| PuffStep flag | Onionskin flag | Missing synonym to add |
|---|---|---|
| `--mu` / `--discreteEmat` | `--hmm-emission-means` | `--hmm-mu` and `--hmm-discreteEmat` |
| `--sigma` | `--hmm-emission-sigmas` | `--hmm-sigma` |
| `--path` / `-p` | `--hmm-decode-path` | `--hmm-path` |
| `--constrainEmit` | `--hmm-constrain-emit` | `--hmm-constrainEmit` |

Flags already covered by existing synonyms (no action needed): `--hmm-special-idx`, `--hmm-init-special`, `--hmm-leave-special-state`, `--hmm-leave-other`, `--hmm-expected-special-length`, `--hmm-expected-other-length`, `--hmm-emodel` (currently deprecated → see BRAIN15.11 — must be RESTORED, not retired).

Flags that were just `--hmm-`-prefixed with no other rename (`kmeans`, `iters`, `converge`, `emitpseudo`, `learnpseudo`, `transprobs`, `exp-decay`, `initialprobs`) do not need synonym additions — they ARE the PuffStep names with the `--hmm-` prefix tacked on.

**Standing rule (per Phase 15 FEEDBACK Q6, 2026-04-28):** The `--hmm-` prefix is **invisible** for the purpose of comparing onionskin flag names to PuffStep flag names. Synonym status is **assumed true** for any flag whose post-prefix name is unchanged from PuffStep — no explicit synonym registration is needed for those flags. This rule should be stated explicitly in the header section of the HMM argparse group. Synonym registrations + help-text synonym notes are required ONLY when the post-prefix onionskin flag name was changed from the PuffStep name (i.e., when we made a rename decision); in those cases, we still offer the prefixed-PuffStep-name version as a synonym + a help-string note pointing at the synonym.

### BRAIN15.11 — HMM PuffStep synonym audit results + restore `--hmm-emodel` + standing rule "no synonyms retired"

**Source:** SOUP15.9 (Phase 14 Supplemental 14-S15 audit findings).

**Audit summary (from SOUP15.9):** A complete audit of HMM PuffStep synonym coverage was produced during Phase 14 Supplemental cycle 14S.2a. Key findings:

| Canonical flag | PuffStep synonym | Help mentions synonym? | Synonym registered as alias? |
|---|---|---|---|
| `--hmm-expected-background-length` | `--hmm-expected-special-length` | YES | YES |
| `--hmm-expected-amp-step-length` | `--hmm-expected-other-length` | YES | YES |
| `--hmm-background-idx` | `--hmm-special-idx` | YES | YES |
| `--hmm-init-background` | `--hmm-init-special` | YES | YES |
| `--hmm-leave-background-state` | `--hmm-leave-special-state` | YES | YES |
| `--hmm-leave-amp-step` | `--hmm-leave-other` | YES | YES |
| `--hmm-emission-model` | `--hmm-emodel` | YES (deprecation note) | NO — **retired** in cycle 14S.1a v0.14.68 |

**Critical user finding (verbatim from SOUP15.9):** *"The `--hmm-emodel` retirement (Phase 14 Supplemental cycle 14S.1a v0.14.68) was a mistake not authorized by the user. An agent decided to do this on its own."*

**Required action:**

1. **Restore `--hmm-emodel` as an active argparse alias** for `--hmm-emission-model`. Remove it from the `_DEPRECATED_FLAGS` pre-parse gate. Update the canonical flag's help text accordingly.
2. **Standing rule going forward:** **No PuffStep synonyms are to be retired.** Synonyms do not have to be the main advertised flag, but they MUST remain available so historical PuffStep commands and command-history archeology still works. This rule applies to all synonyms, not just `--hmm-emodel`.
3. **Help-text rewrites (separate, lower-priority):** Tighten the wording of synonym mentions where helpful, without retiring any alias.

**Lessons from the incident:** Catalog this as a documented case of unilateral agent narrowing — a "scope authority" violation. Cross-reference to `multi-agent/AGENT_CONVENTIONS.md § Scope authority` and `feedback_scope_authority.md` (auto-memory).

---

## HMM CLI features (new flags + behaviors)

### BRAIN15.12 — `--hmm-0-based-statepath` flag + adaptive defaults for companion threshold flags

**Source:** SOUP15.10 (the 14-S21 audit), SOUP15.8 item 2 (an older `######OTHER` scratch note — vestigial; resolved per Phase 15 FEEDBACK Q1, 2026-04-28), tracking/BRAINSTORM.md [2026-04-18] `--hmm-0-based-statepath` future CLI flag.

**Resolution of the older "stays in BRAINSTORM.md" note (per Phase 15 FEEDBACK Q1, 2026-04-28):** That note was vestigial — `--hmm-0-based-statepath` is firmly in Phase 15 scope. The user has also noted (Q1) that during the brainstorming stage Phase 15 will likely update its plans here: switch the **default state path to 0-based** and treat the legacy state path as 1-based. That update is captured in `tracking/KNOWN_ISSUES.md` (added by user on 2026-04-28); it does not need to be re-engineered here at the soup-to-brainstorm transfer stage — the brainstorming stage will integrate it into BRAIN15.12.

**Goal:** Add a `--hmm-0-based-statepath` flag that emits HMM state paths using 0-based state indexing (state 0 = CN=1 background, state 1 = first amplification level, etc.) rather than the current 1-based default (state 1 = CN=1 background, state 2 = first amplification level, etc.). The 0-based indexing makes the `2^state` copy-number mapping arithmetically clean. Plus: when the flag is set AND the user has NOT explicitly set the companion threshold flags, shift their defaults; if the user explicitly sets them, honor the user-provided values.

**Adaptive defaults:**

| Flag | Default when 1-based (current) | Default when `--hmm-0-based-statepath` is set |
|---|---|---|
| `--hmm-thresh-state` | `1` ("states > 1 are amplified; state 1 is CN=1 background") | `0` ("states > 0 are amplified; state 0 is CN=1 background") |
| `--hmm-max-state-thresh` | `0` (sentinel: no filter) | `-1` (sentinel: no filter; discard regions whose max state ≤ -1) |

Implementation pattern: use `argparse.SUPPRESS` as the default for the two threshold flags so we can distinguish "user omitted" from "user set to the numeric default." Detect `args.hmm_0_based_statepath` post-parse, assign defaults accordingly.

**Files that change when this lands** (from SOUP15.10 audit, verified live 2026-04-25):

1. `onionskin.py:build_parser()` HMM group — add the flag, update the two threshold flags' help strings.
2. `onionskin.py:main()` post-parse argv — detect the flag, apply adaptive defaults.
3. `onionskin_core/engines/hmm_engine.py:_step6_hmm()` (line 551) — state-path emit.
4. `onionskin_core/hmm_core.py:_posterior_path()` (line 337), `_viterbi()` (line 244) — internal stays 1-based; shift applies at write boundary.
5. Downstream consumers: `onionskin_core/hmm_summits.py` (`extract_summits`, `_pick_summit`, `_merge_and_pick` at lines 256, 183, 212), `onionskin_core/hmm_metrics.py`, `onionskin_core/hmm_fork_travel.py`.
6. `tests/test_pipeline.py` and friends — new tests needed (none of the existing state-value assertions are affected per the 2026-04-25 enumeration; suggested additions in SOUP15.10's "Test enumeration addendum" sub-section).
7. Docs: `multi-agent/full_instructions/PIPELINE_SPEC.md`, `multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md`, README.

**Constraints / cautions:**

- PuffStep gold-standard outputs (`make puff-compare`) are 1-based; the `--hmm-0-based-statepath` flag MUST NOT be active during `make puff-compare` runs, or comparisons will be off-by-one.
- Decision for SPEC engineering: pick whether the flag affects ONLY the emitted bedgraph, OR also all metric outputs. User likely wants the emitted bedgraph to match the metrics tables — so both surfaces shift together. This is a SPEC-engineering decision, not a BRAINSTORM one; flagged for SPEC.

**Cross-ref:** the `--hmm-0-based-statepath` future CLI flag entry in tracking/BRAINSTORM.md [2026-04-18] should be updated/closed when this priority lands.

### BRAIN15.13 — `--hmm-smooth-halfwidth` APS split (RESOLVED in Phase 14 Supplemental 14S.23)

**Source:** SOUP15.5 (carry-over queue item: "Potential split of `--hmm-smooth-halfwidth` into separate APS halfwidth flag (low priority; diagnose need before doing)").

**Status:** **Already RESOLVED.** Tracked as [ISSUE:2026-04-19:3] which closed in Phase 14 Supplemental cycle 14S.23. The new `--hmm-aps-smooth-halfwidth` flag controls HMM step-14 APS per-locus smoothing independently from the general `--hmm-smooth-halfwidth`.

**Action for Phase 15:** Mention in Phase 15 closeout that this carry-over item is already done; remove from carry-over tracking surfaces. No implementation work in Phase 15.

---

## APS / clustering / shape

### BRAIN15.14 — `--aps-area-excess-floor` analytical testing

**Source:** SOUP15.11 (14-S22 follow-ups), [ISSUE:2026-04-18:1] APS area-excess floor default and clustering experiment.

**Context:** The CLI flag was added in Phase 14 Supplemental with `default=on`; the analytical work was deferred to Phase 15 per user Q22 answer. Two user-flagged concerns drive the analytical agenda:

**Concern #1:** The per-bin floor at 0 systematically biases APS high when RCN oscillates around 1. A better implementation would allow negative per-bin contributions during the sum, then floor the per-amplicon total at 0 if it ends up negative. This is an alternative wiring for `--aps-area-excess-floor off`, NOT a new flag — it's a different internal implementation. Test both wirings side-by-side.

**Concern #2:** Flooring may hurt shape-similarity feature computation more than APS score computation. When `--aps-feature shape` is active, the floor may have an outsized effect on cluster recovery. Test `--aps-feature shape` with floor on vs off on datasets where posterior groupings are known.

**Phase 15 work:**

1. **Default-flip test:** Test `--aps-area-excess-floor off` on live datasets. Evaluate impact on APS cluster separation, posterior groupings, and summit evaluation (II/9A, II/2B). Decide whether to flip the default to `off`.
2. **Post-sum-floor variant:** Implement and test the post-sum-floor wiring as an alternative for `--aps-area-excess-floor off`. Test alongside the simple-off wiring.
3. **Shape-based clustering implications:** Test `--aps-feature shape` with floor on vs off on datasets where posterior groupings are known.
4. **Validation criteria for any default change:** "No M-shaped posterior amplicons; summit evaluations improve or at least stay the same; no regressions." (User quote, Q22 answer.)

### BRAIN15.15 — Cross-pipeline dedup code unification — **RESOLVED (already done; closes as N/A in Phase 15)**

**Source:** SOUP15.13 (14-S20 related — cross-pipeline dedup code unification).

**Status:** **RESOLVED** by Phase 14 Supplemental cycle 14-S20 work. No Phase 15 implementation work; this BRAIN entry exists as a closeout marker (analogous to BRAIN15.13).

**Investigation result (Phase 15 FEEDBACK Q7 follow-up, 2026-04-28):**

The 14-S20 work already verified `--dedup-dist` pipeline applicability and explicitly concluded **no cross-pipeline dedup unification work is needed**:

- **Growth and RMS already share** the dedup implementation via `dedup_calls_by_peak_proximity()` in `onionskin_core/rcn_mean_shift_helpers.py`. Growth calls it from `onionskin_core/engines/growth_model_engine.py:1382`; RMS calls it from `onionskin_core/engines/rcn_mean_shift_engine.py:227` (and from `onionskin.py:4042 / 4142` via `apply_peak_proximity_dedup_tsv()`).
- **HMM uses its own step-8 merging** by design (`--hmm-merge2`); HMM intentionally does NOT use `--dedup-dist` and the `--dedup-dist` help text was rewritten in 14-S20 to make this scope explicit (verified at `onionskin.py:1001-1012`).
- The 14-S20 closeout entry in `CHANGELOG.md` (around v0.14.x) explicitly states: *"no PHASE15_BRAINSTORM cross-pipeline dedup note added (RMS+Growth already share `dedup_calls_by_peak_proximity()`; HMM uses its own step-8 merging by design)."* So 14-S20 explicitly tried to AVOID even creating this BRAINSTORM note — its mere existence here is overhead introduced during Phase 15 transfer that we now close.

**Action taken:** This entry is marked RESOLVED. No DEDUP_SOUP.md was created (would have been if 14-S20 had revealed unaddressed divergence; it didn't).

**No further Phase 15 work needed for this BRAIN entry.**

### BRAIN15.16 — APS Universal-in-spirit re-framing — forward extension to new HMM-specific APS/SAPS flags

**Source:** SOUP15.15 (14-S20 related — APS Universal-in-spirit re-framing summary).

**Context:** Per user Q20 (Phase 14 Supplemental), APS stays in its own parser group with the header re-framed to make explicit that APS is applied by all three pipelines. That re-framing was Phase 14 Supplemental work (14-S20), not Phase 15. But as APS work develops in Phase 15, the same framing logic should extend to:

- Timing group (14-S13 already does a step-mention-only pass)
- Shape scoring group (already Universal)
- Overlap group (already Universal-in-spirit per 14-S20 + Finding 7)
- Asymmetric Triangle Model group (already Universal-in-spirit per 14-S3)

**Phase 15 work:** When Phase 15 adds new HMM-specific APS flags or sub-APS (SAPS) flags (BRAIN15.20), apply the same re-framing pattern to any new APS-like parser group. Maintain consistency with the Universal-in-spirit pattern established in Phase 14 Supplemental.

#### Round 6 update (2026-04-28) — extend the Universal-in-spirit pattern to the NEW Origin Detection Window argparse group

The Bayesian ODW System (BRAIN15.7 Round 6 deliverable item 6) introduces three new CLI flags (`--odw-active-stage-detector`, `--odw-prob-threshold`, `--odw-fold-threshold` — names TBD by SPEC engineering) that don't fit any existing argparse group cleanly. Per user direction (chat 2026-04-28), Phase 15 creates a **new "Origin Detection Window" (ODW) argparse group** rather than adding the flags to the existing timing group.

Rationale (cross-cutting scope):

- ODW boundaries affect **timing analyses** (onset_stage, last_active_stage, activity_breadth, amplicon_class — all in `onionskin_core/timing.py`).
- ODW boundaries affect **summit refinement** (BRAIN15.8 stage-selection strategies — max_rcn_stage / onset_stage / last_active_stage / all-ODW / subset-best-shape / narrowest-summit-state all consume ODW boundaries).
- ODW boundaries affect **amplicon reliability scoring** (BRAIN15.7 — uses ODW for credibility evaluation per the founding design principle).
- ODW boundaries affect **multistage unification** (BRAIN15.18 sub-priorities d + e — within-ODW non-monotonic-growth detection consumes the same per-transition test outputs).
- ODW boundaries affect **summit↔timing convergence** (BRAIN15.29 — the "active stages" referenced in step 3 of the four-step convergence are the ODW).
- ODW boundaries may affect **APS analyses** (BRAIN15.27 — if APS feature computation chooses to use ODW-confined inputs for stages-rich features; SPEC engineering decides).

Adding the flags to the timing group would mis-suggest single-domain scope. The new ODW group with cross-cutting header text (see BRAIN15.7 item 6) follows the same Universal-in-spirit pattern this BRAIN15.16 entry promotes — group-header text explicitly enumerates downstream surfaces affected.

**Phase 15 list of cross-cutting argparse groups (post-Round-6):**

- APS group (Phase 14 Supplemental 14-S20 — already Universal-in-spirit framed).
- Timing group (14-S13 step-mention-only pass).
- Shape scoring group (already Universal).
- Overlap group (already Universal-in-spirit per 14-S20 + Finding 7).
- Asymmetric Triangle Model group (already Universal-in-spirit per 14-S3).
- **NEW: Origin Detection Window (ODW) group (Phase 15 — per BRAIN15.7 item 6 + this Round 6 update).** Apply same Universal-in-spirit framing.

When SPEC engineering finalizes the ODW group's flags + header text, BRAIN15.16's forward-extension list extends to include the ODW group.

### BRAIN15.17 — APS analysis-surface parity across pipelines + master APS catalog file

**Source:** SOUP15.8 item 1 ("APS needs fleshing out -- plots, for example"). Concrete scope confirmed by user in Phase 15 FEEDBACK Q5 (2026-04-28).

**Goal:** Bring the HMM pipeline's APS analysis surface — plots, output tables, notebook templates, etc. — to **parity** with what Growth and RMS already offer. The HMM pipeline should offer all analyses, outputs, plots, and notebook analyses offered by any other pipeline. As part of this work, create a **master APS catalog file** that lists every APS analysis/output/plot/notebook available across the three pipelines, used to drive ongoing completeness audits.

**Phase 15 deliverables:**

1. **Audit the analogous APS steps in Growth and RMS pipelines** to enumerate every APS analysis, output, plot, and notebook those pipelines emit today.
2. **Audit the HMM pipeline's APS analysis surface** for what currently exists vs what Growth/RMS has.
3. **Implement the gaps** in HMM so the HMM APS analysis surface matches Growth/RMS parity. (Pipeline-specific exceptions are allowed and noted in the catalog.)
4. **Create a master APS catalog file** (location TBD; candidate: `multi-agent/full_instructions/APS_CATALOG.md` or a tracking surface) that:
   - Lists every APS-related analysis, output file, plot, and notebook.
   - Marks each entry as "Universal" (all three pipelines should have it) vs "Pipeline-specific" (e.g., HMM-only state-path features, RMS-only shape-filter outputs).
   - Is updated any time something is added to the APS analysis surface in any pipeline.
   - Serves as a look-up table for what is supposed to be present at minimum in each pipeline's APS surface.
   - Is auditable in both directions: audit each pipeline against the catalog (does the pipeline emit everything the catalog says it should?), and audit the catalog against the three pipelines in code (does the catalog reflect everything the code emits?).

**Implementation approach:**

- Step 1 starts with a code-level grep + manifest-level inspection of each pipeline's APS-emitting modules and helper scripts.
- Step 2 produces a gap list (what HMM doesn't yet emit but Growth/RMS does).
- Step 3 implements the gaps inside the HMM subtree only (per BRAIN15.2 design rule 1: pipeline-derived outputs belong under the pipeline that computes them).
- Step 4 produces the catalog as a maintained living document — its maintenance becomes a standing rule going forward.

**Cross-pipeline scope:** This BRAIN entry is primarily about HMM-APS parity (HMM is the gap-bearing pipeline today), but the catalog itself spans all three pipelines and stays useful for future audits.

#### Round 4 expansion (2026-04-28) — step-less plots/notebooks convention + by-eye eval references + HMM plot review

##### Step-less `plots/` and `notebooks/` directories — cross-pipeline structural change (per Q18 + [ISSUE:2026-04-28:1])

User direction Q18 (Phase 15 FEEDBACK Round 3, 2026-04-28): *"HMM pipeline should absolutely get a `plots` directory for the 'completeness' goal, but see KNOWN_ISSUES.md [ISSUE:2026-04-28:1]. all pipelines should remove the step from the plots and notebooks directories because as development continues, and steps are added or subtracted, the plots and notebooks directories can sit in the wrong place... so might as well put them outside of the steps, and allow plots and notebooks to be written to them at any time. And as new steps or analyses are added, they can just write to those 'step-less' directories as needed."*

**[ISSUE:2026-04-28:1] (Plots and notebooks should be the last directories in each pipeline) — verbatim:** *"Plots and Notebooks are supposed to reach back into previous step directories, not forward into subsequent step directories. [...] Partial solution: Do not add step numbers to these directories since they will always be changing if steps are added after them."*

**Cross-pipeline structural change:**

- **Growth-model pipeline** currently uses `02-growth-model/14-plots/` and `02-growth-model/15-notebook/` — drop the step numbers → `02-growth-model/plots/` and `02-growth-model/notebooks/`.
- **RMS pipeline** currently uses `03-rcn-mean-shift/11-plots/` and `03-rcn-mean-shift/14-notebook/` — drop the step numbers → `03-rcn-mean-shift/plots/` and `03-rcn-mean-shift/notebooks/`.
- **HMM pipeline** currently uses `01-hmm/notebooks/` (step-less by accident) and lacks a `plots/` dir entirely — add `01-hmm/plots/` (step-less); keep `01-hmm/notebooks/` step-less.

**Why this is a structural fix, not just a rename:**

- The current step-numbered scheme breaks every time a new pipeline step is inserted (e.g., BRAIN15.6 inserts SAPS at step 15, renumbering everything beyond — the `14-plots/` and `15-notebook/` directories get shifted out from under any code that referenced them by step number).
- Step-less directories sit at a stable path regardless of step-renumbering. Any analysis code can write to them without coupling to step numbering.
- This frees plots/notebooks to "reach back" into ANY step's outputs without competing with the step-numbering scheme.

**Update to `onionskin_core/output_layout.py`:** the `build_growth_steps()`, `build_rcn_mean_shift_steps()`, and `build_hmm_steps()` functions all need updating. The `step15`/`step16` keys in `build_hmm_steps()` should be replaced or augmented with `plots`/`notebooks` keys (already partially there for `notebooks` in HMM). Same for the other two pipelines.

**Tests + scripts that reference step-numbered plots/notebooks paths:** sweep needed — scripts in `scripts/` (e.g., `aps_cluster_report.py`, `summit_inspector.py`) and tests in `tests/` may reference `14-plots/` or `15-notebook/` by step number.

##### HMM plot review — already-substantial work (per `tracking/BRAINSTORM.md [2026-04-09] HMM fork travel and analysis plots — review COMPLETE`)

The HMM-side plot review was completed 2026-04-09. Per-plot decisions captured in the (now archived) `multi-agent/plans/archived/20260411-PHASE9_SPEC.md` "Plot review — findings" section. Summary of decisions (verbatim):

- **Fork asymmetry scatter:** Keep; swap axes (right→X, left→Y); add per-stage grid variant.
- **Level emergence heatmap:** Keep; add sequence-length/bin-count filters; add chr-level heatmaps.
- **Fork travel trajectory:** Keep as-is; add new "fork age tracking" plot.
- **Nested domain diagrams:** Keep as-is, user liked them.

User Q20 follow-up note on the [2026-04-09] entry: *"It says complete, but make sure it is complete in the spirit of Phase 15 (e.g. compared to other pipelines, and using all its own HMM-specific outputs)."*

**Phase 15 audit task for BRAIN15.17:** verify each of the 4 plot decisions above actually landed in code; verify the new plots (per-stage grid variant of fork asymmetry; chr-level heatmaps; fork age tracking plot) exist; close any gaps. Cross-check against the cross-pipeline parity catalog this entry produces.

##### Fork age tracking — already implemented (BRAIN15.32 audit-disproved Round 3)

User Q20 follow-up note: *"BRAIN15.32 may also already be implemented. I am starting to see the pattern that BRAINSTORM.md may be stale and need an audit and clean up. Nonetheless, BRAIN15.32 is a very important topic - it just might already be done."*

**Round 4 audit confirmed: fork age tracking is implemented** in [onionskin_core/hmm_fork_travel.py:12, 242-243](onionskin_core/hmm_fork_travel.py) — `12-fork-travel/fork_age_metrics.tsv`, `level_emergence_stage`, `ghost_level_flag`. See BRAIN15.32 RESOLVED marker (this BRAINSTORM file) for verification details.

**Phase 15 task for BRAIN15.17:** include fork-age-tracking plots in the cross-pipeline catalog as HMM-specific (no analog in growth/RMS). Audit whether the [2026-04-09] design's "rendering rule" (continuous trajectories with gaps for missing stages) is honored in code; if not, that's an enhancement opportunity within Phase 15 scope.

##### By-eye evaluation surface for HMM (per Q17)

User Q17 answer (Phase 15 FEEDBACK Round 3, 2026-04-28): *"we can certainly use the 'by eye' file `tests/full_chrom_training_data/amplicons.by-eye.bed` for evaluations. we can and should also use the PuffStep results - various results here: `dev/puffstep/automate/`. we can and certainly should use our various summit evaluation tests for II/9A as well, codified in various make tests, for example, and all based on this file: `tests/summit_training_data/01-II9A-evidence-based-confinement-boundaries-and-rules.bed`, and to a much lesser extent this file `tests/summit_training_data/02-II2B-evidence-based-confinement-boundaries-and-rules.bed`. so yes - we can use the machinery and files already in place to evaluate HMM performance. We have made a ton of tests in that direction. There may or may not need to be some adaptations to HMM run development. I will let you audit the code for that."*

**Phase 15 task for BRAIN15.17:** audit whether existing by-eye eval machinery already runs against HMM outputs cleanly. If yes, document the established pattern and add HMM-specific eval entry points to the catalog (`make summit-hmm` style targets, etc.). If gaps exist, capture them as Phase 15 deliverables.

**Reference files (verbatim from Q17 answer):**

- `tests/full_chrom_training_data/amplicons.by-eye.bed` — full chr-by-eye amplicon coordinates
- `tests/summit_training_data/01-II9A-evidence-based-confinement-boundaries-and-rules.bed` — II/9A summit eval (primary)
- `tests/summit_training_data/02-II2B-evidence-based-confinement-boundaries-and-rules.bed` — II/2B summit eval (secondary)
- `dev/puffstep/automate/` — PuffStep gold-standard outputs for cross-validation

##### Cross-pipeline plot inventory expansion candidates

The [2026-04-09] HMM plot review's decisions list isn't an exhaustive cross-pipeline catalog. The Round 4 catalog work should also include:

- Profile plots (per-amplicon RCN profiles per stage) — exist in `02-growth-model/14-plots/profile/` (or similar); HMM should have analog.
- Genome overview plots — exist in growth as `genome_overview/`; HMM analog.
- Summit fit plots — exist in RMS as `summit_fits/`; HMM analog.
- Shape-filter diagnostic plots — exist in growth/RMS (with `shape_filter_plots.py` shared module); HMM should reuse the shared module after BRAIN15.18 wires the HMM shape filter.
- Growth-curves plots — growth-pipeline-specific; HMM analog uses state path or per-stage RCN series.

Bidirectional fork analysis priorities (8.5.1 / 8.5.2 / 8.5.3 from `tracking/BRAINSTORM.md`): user Q20 note flagged these as needing audit-then-include. Defer to brainstorming-stage code audit; if not implemented, add to BRAIN15.17 catalog as HMM-related plot/analysis deliverables.

##### Updated implementation plan (Round 4)

The earlier 4-step plan still holds. Phase 15 implementation now also:

1. **Lands the step-less plots/notebooks structural change** across all three pipelines first (it touches `output_layout.py` and may break tests/scripts that reference step-numbered paths — fix in same change).
2. **Audits HMM plots against the [2026-04-09] decisions list** + the cross-pipeline catalog to identify missing plots.
3. **Audits HMM by-eye eval machinery** for adaptations needed.
4. **Authors the master APS catalog file** — covers Universal vs Pipeline-specific outputs across all three pipelines + maintenance discipline.
5. **Cross-references with BRAIN15.31** (--peak-summary extension) for any peak-summary plots emitted by the new flag.

##### Round 6 update (2026-04-28) — concrete HMM notebook generator target (per Codex audit)

The Codex Agent 2 brainstorming-stage code/tracking audit (2026-04-28) named `onionskin_core/hmm_notebooks.py` as the existing HMM notebook generator that needs the step-less treatment + a name modernization. Captured here as concrete implementation target:

- **`onionskin_core/hmm_notebooks.py:84, :111, :122`** — currently writes notebooks that hardcode step-numbered HMM directory paths (`12-fork-travel`, `14-aps`, `15-timing`, `16-clustering`). Phase 15 changes step numbering (BRAIN15.6: SAPS at step 15, timing → 16, clustering → 17) AND drops step-numbers from `notebooks/` per the step-less convention. The notebook generator must update both: paths to step-numbered directories shift to the new numbers; output of the generator itself moves to step-less `<pipeline>/notebooks/`.
- **Function name `write_hmm_phase9_notebooks`** (called from `onionskin_core/engines/hmm_engine.py:1269-1270`) — stale relic from Phase 9 development. Should be renamed during the BRAIN15.17 expansion or as part of BRAIN15.33 (Phase 15 housekeeping bundle). Recommended new name: `write_hmm_notebooks` (drop the phase number; matches the step-less convention's spirit).
- **Cross-reference:** the function rename is also called out in BRAIN15.33 Round 6 update.

This is a small concrete deliverable that prevents BRAIN15.17's "audit HMM plots/notebooks" abstract task from missing the actual generator file at SPEC engineering time.

### BRAIN15.18 — HMM shape-score wiring + meta-analysis + state-path growth credibility

**Source:** SOUP15.14 (14-S26 follow-ups — HMM shape-score wiring), tracking/BRAINSTORM.md [2026-04-18] HMM-native amplicon quality criteria (the "ghost levels" entry referenced in SOUP15.14).

**Context:** Phase 14 Supplemental wired the RMS shape-score sink (`shape_filter_calls()` / `apply_shape_filter_tsv()` producing `shape_score_raw` from `dBIC_flat_vs_tri`); HMM was left as a placeholder per Q26 answer. Phase 15 lands the HMM shape-score work.

**Three sub-priorities, packaged together:**

**(a) Implement the HMM shape-filter sink.**

- Decide the input: per-amplicon stage-median RCN profile within the HMM amplicon interval.
- Implement `dBIC_flat_vs_tri` equivalent for HMM data (reuse RMS/growth code where possible).
- Emit `shape_score_raw` column in an HMM output file (new or existing).
- Wire `--hmm-shape-score-threshold` to consume it.
- Wire `--hmm-shape-score-strict-bic` to toggle strict-BIC mode.

**(b) HMM meta-analysis of amplicon shapes across stages** (user Q26 quote: *"Add to HMM what we do for RMS with the meta-analysis of amplicon shapes across stages to get a final set of amplicons and collapsed repeats."*)

- Port RMS's cross-stage shape meta-analysis to HMM.
- Emit a final set of amplicons (passing the meta-filter) and putative collapsed repeats (failing).

**(c) HMM-specific: detect state-path growth as shape credibility signal.**

HMM state paths encode nesting levels directly. Growing state paths (nesting depth increases across stages) → high credibility. Flickering state paths (level appears/disappears across stages) → low credibility ("ghost levels" from tracking/BRAINSTORM.md [2026-04-18]). This is a genuinely HMM-native signal not available from RMS or growth pipelines. Becomes a new HMM credibility feature that feeds the final amplicon set.

**(d) Combined multistage unification flag with 4 modes (substantially expanded per Phase 15 FEEDBACK Q9, 2026-04-28).**

Multistage unification can default to one of four behaviors via a flag (default = mode 4, least stringent; can be tightened later if 4 turns out too permissive):

1. Use only the triangle filter–based results.
2. Use only the state-path evolution–based results.
3. Only classify candidates as amplicons if they are classified as amplicons in BOTH methods (most stringent).
4. Include any candidate that is classified as an amplicon in EITHER method (least stringent — default).

Regardless of the classification decision, BOTH the multistage triangle analysis and the multistage state-path-evolution analysis should be **run and reported** (e.g., results in columns of a per-amplicon table) so the user can see both signals.

**(e) State-path evolution checks for amplicon-vs-collapsed-repeat classification (substantially expanded per Phase 15 FEEDBACK Q9, 2026-04-28).**

For each candidate region (any region marked above-background in any stage), run these checks against the multistage state-path data:

1. Does it persist after it pops up?
2. If it is present in multiple stages, does it get wider across them?
3. If it is present in multiple stages, does it evolve to have more than one step?
4. If it has more than one step, is it pyramid-shaped (with the summit interval approximately centered between steps on both sides)?

**Decision rules from these checks (user-explained):**

- Amplicons that start early and grow tall: all checks pass; also passes the multistage triangle process. → **amplicon**.
- Amplicons that start early (or any stage except the last), but do not grow in height: must pass at least checks 1 and 2 AND pass the multistage triangle analysis (if triangle results are instructed to be used in the combined-flag mode). → **amplicon**.
- Amplicons that appear in only one stage: must be the LAST stage (to satisfy check 1 — "does it persist after it pops up" is trivially-yes for last-stage-only) AND must pass the multistage triangle analysis (if triangle results are instructed). → **amplicon (single-stage, late-onset)**.
- Anything single-stage that ISN'T the last stage and doesn't grow: collapsed-repeat-likely.
- Single-step regions of the same length across all stages: collapsed-repeat-likely (they don't show the cross-stage evolution amplicons exhibit).

**Why this matters for filtering decisions in single-stage runs vs multistage runs:**

> User-quoted (Q9): *"When multiple stages are present, above background amplicon candidates are not excluded prior to evaluating them with respect to all stages. If there is only one stage, then filters for things like 'length' (for example) might be needed to help eliminate false positives (e.g. anything < 50 kb with only one step up) at the expense of false negatives. But with multiple stages, we can more confidently say which regions are truly amplicons. Then we would not want to exclude any state path information from those regions."*

So: in **multistage runs**, do NOT exclude short/single-step regions before running the multistage analysis — the analysis itself is what classifies them. In **single-stage runs**, fall back to length / step-count heuristic filters because the multistage signal isn't available.

**Cross-reference:** This `(d)` and `(e)` content is the prerequisite for BRAIN15.26 (chrom-median default switch). BRAIN15.18 + BRAIN15.21 must land before BRAIN15.26 fires.

**Brainstorming-stage note:** Sub-priorities (d) and (e) above were substantially expanded during the Phase 15 transfer follow-up based on the user's Q9 answer. The user explicitly noted (Q10) that aggregating ideas from `tracking/BRAINSTORM.md`, `tracking/KNOWN_ISSUES.md`, `tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` is technically a brainstorming-stage activity (after the SOUP transfer closes) — so additional enrichment of (d) and (e) from those tracking files is expected during the brainstorming stage.

#### Round 5 update (2026-04-28) — within-ODW non-monotonic growth handling

Per Q23 user clarification (Phase 15 FEEDBACK Round 4 answers): within-ODW growth is **NOT strictly monotonic**. The user notes: *"it is possible to have stages within the ODW that do not show summit height since the last stage, but summit growth continues later in the window."*

**Implication for sub-priorities (d) + (e):**

- The "growth detected w.r.t. previous stage" check that distinguishes amplicon vs collapsed-repeat under sub-priority (e) must allow a **single non-growth transition mid-window** without closing the window. The window only closes when sustained-flat-or-declining behavior begins (which is what `last_active_stage` per BRAIN15.24 captures).
- The 4-mode classification flag's logic (sub-priority d) needs to accommodate the within-ODW non-monotonicity. Specifically:
  - "Persistence after pop-up" (check 1 in sub-priority e) — measured across the whole ODW, not just contiguous monotonic-growth runs.
  - "Width grows across stages" (check 2) — measured across the whole ODW, allowing for plateau-mid-window patterns.
  - "Multi-step state-path evolution" (check 3) — same.
  - "Pyramid-shape" (check 4) — symmetric assessment across ODW, robust to within-window plateaus.
- **Existing within-ODW growth-detection methods are already in the codebase** (per Q23 + Round 6 Codex audit finding 2026-04-28): `onionskin_core/timing.py:370-393` implements the 1.25× fold-change test today (defines `latest_activity_stage`, post-rename: `last_active_stage`). However, the 1.25× hard-threshold test is officially being **retired in Phase 15** in favor of a probabilistic / variance-aware test (Gaussian-overlap or Bayesian posterior P(ratio ≥ 1.25); see BRAIN15.7 Round 6 deliverable item 6 + BRAIN15.24 four-metric naming-summary table for the framework decision). BRAIN15.18 sub-priorities (d) + (e) should consume the **post-rewrite** within-ODW growth-detection test, not the legacy 1.25× rule.
- This non-monotonicity tolerance is also the reason `last_active_stage` (BRAIN15.24) is distinct from `regression_stage` — `regression_stage` is the FIRST confirmed dip post-peak; `last_active_stage` is the LAST stage where growth was detected (can be after a plateau). Both are computed by variance-aware tests after the BRAIN15.7-item-6 framework rewrite (Gaussian-overlap or Bayesian posterior).

### BRAIN15.19 — Clustering defaults finalization

**Source:** SOUP15.19 (Clustering defaults).

**Context:** A while ago, optimization began for what works best for clustering: summit vs width vs area vs shape (and log2 / log10 versions of some), elbow vs keep, singleton guard or not, etc. The work was not finished. Phase 15 finishes it, especially as it relates to HMM results.

**Last known state (per SOUP15.19):** HMM is still using `elbow` and `area`, and is still being collapsed into 2 groups.

**Phase 15 work:**

- Inventory the current clustering defaults across the three pipelines.
- Run the optimization experiments that were started but not finished.
- Settle the defaults — at minimum for HMM, ideally cross-pipeline.
- Document the rationale for the chosen defaults so future audits can re-validate against new datasets.

**Dependency on BRAIN15.7 + BRAIN15.14 (confirmed per Phase 15 FEEDBACK Q8, 2026-04-28):**

Both prerequisites land first; BRAIN15.19 follows them.

- **BRAIN15.14 (`--aps-area-excess-floor` analytical testing):** One of BRAIN15.19's tests will be on which `--aps-area-excess-floor` behavior works better — the floor-on, floor-off, and post-sum-floor variants from BRAIN15.14 are direct inputs to BRAIN15.19's clustering experiments. Strict dependency: BRAIN15.14 results inform BRAIN15.19's experimental matrix.
- **BRAIN15.7 (amplicon reliability scoring + flat-sample detection):** Reliability-filtered amplicon sets change the clustering inputs. Testing BRAIN15.19 conclusions before vs after reliability filtering would force re-running every test if the filtered set changes the conclusions. Cleaner to wait until the reliability-filtered amplicon set is stable, then run clustering experiments on that set. Per the user's note: *"testing would not strictly be dependent on this, but it would require us testing whether our conclusions changed; thus, it makes more sense to just wait until the set of amplicons is refined."*

**SPEC-engineering note:** SPEC priority for BRAIN15.19 should explicitly list BRAIN15.7 and BRAIN15.14 as prerequisites in its `Source:` / `Depends-on:` field.

#### Round 6 update (2026-04-28) — terminology split: APS sample posterior clustering vs HMM trajectory/fork clustering (per Codex audit)

The Codex Agent 2 brainstorming-stage code/tracking audit (2026-04-28) found that **HMM currently has TWO different "clustering" concepts at different step directories** that the SPEC must keep distinctly named to avoid silent confusion:

- **Sample posterior clustering** (HMM step-14 `aps/`): produces sample-level APS feature matrices and `aps_clusters.tsv` from APS feature columns. Implementation: `onionskin_core/hmm_ported_analyses.py:149-184`. **This is what BRAIN15.19 (clustering defaults finalization) and BRAIN15.27 (composite multi-feature APS clustering modes with three-layer PCA) are about.** Drives **posterior sample groupings / posterior manifests**.
- **HMM trajectory / fork clustering** (HMM step-16 `clustering/` — post-renumbering will be step-17): produces `trajectory_feature_matrix.tsv` and `trajectory_clusters.tsv` from fork-travel feature vectors. Implementation: `onionskin_core/hmm_ported_analyses.py:380-439`. **NOT the same as APS sample clustering.** This is HMM-specific (only HMM has fork-travel data); operates on amplicon-level fork-trajectory features, not sample-level APS features.

**SPEC-engineering rule:** keep these two concepts named distinctly across SPEC priorities, output paths, and help-string text. **"Step 17 clustering" alone is ambiguous** — it could refer to either concept. Disambiguate with explicit modifiers:

- `aps_clusters.tsv` / "APS sample posterior clustering" / "step-14 APS clustering" / "posterior-grouping clustering" — for the sample-level posterior-grouping concept.
- `trajectory_clusters.tsv` / "HMM trajectory clustering" / "step-17 trajectory clustering" / "fork-trajectory clustering" — for the HMM-specific fork-trajectory concept.

Both are legitimate Phase 15 deliverables. They live at different step directories, consume different feature vectors, and produce different output files — they should never be conflated.

**Connection to BRAIN15.27:** BRAIN15.27 is correctly about APS-driven posterior sample grouping / posterior manifests (the first concept). The composite multi-feature APS modes (composite morphology + 3-layer PCA) feed the APS feature matrix; clustering on that matrix produces `aps_clusters.tsv`. BRAIN15.27 does NOT touch the trajectory-clustering concept.

**Connection to BRAIN15.5 (HMM completeness matrix):** the analysis-families table should distinguish the two concepts as separate rows: "APS sample posterior clustering" (Universal — all three pipelines) vs "HMM trajectory/fork clustering" (HMM-specific — no growth/RMS analog).

---

## SAPS / pre-pipeline gap mask / IBM bundle / misc

### BRAIN15.20 — SAPS implementation (state-path APS)

**Source:** SOUP15.20 (SAPS), IBM-C2 SAPS sub-piece, tracking/BRAINSTORM.md [2026-04-18] SAPS — State-APS from individual-sample HMM decoding.

**Goal:** Compute APS on each sample's decoded HMM state path instead of (or alongside) raw signal amplitude. Per-locus SAPS gives state-based scoring that is HMM-native.

**Dependency:** SAPS is gated on BRAIN15.6 (HMM parallel child pipeline) — needs per-sample state-path outputs from HMM step-6 (`06-HMM/indiv_samples/`). Cannot land before BRAIN15.6.

**Step placement (from BRAIN15.6 + SOUP15.20):** SAPS at `15-saps/` (with the renumbering: timing → `16-timing/`, clustering → `17-clustering/`).

**Detailed design:** See tracking/BRAINSTORM.md [2026-04-18] SAPS entry for the per-locus metric definitions. The SOUP entry in SOUP15.20 is a stub ("Statepath APS score analyses as described elsewhere"); the elsewhere is the [2026-04-18] BRAINSTORM entry. Phase 15 SPEC engineering should pull the design-level details from that BRAINSTORM entry into the SPEC.

### BRAIN15.21 — Lift shared gap mask + missingness diagnostic to a pre-pipeline step

**Source:** SOUP15.21 (Lift shared gap mask + missingness diagnostic to a pre-pipeline step — HMM-motivated; benefits all three pipelines via inheritance).

**Architectural framing:** "Anything shared by two or more pipelines that doesn't depend on pipeline-internal upstream work can come out in front, run once, and be inherited" — the same principle that already governs bin-size detection. Gap-mask construction + missingness diagnostic should join it.

**Why this lives in HMM_SOUP rather than waiting:** The HMM pipeline's needs for both (a) chrom-median consuming a precomputed shared gap mask to remove bins consistently across files, and (b) post-call gap-aware annotation parity with growth/RMS — those needs are specific to HMM completeness work. The fact that growth and RMS also benefit (more accurate gap classification + no redundant mask computation) is downstream gravy.

**The principled approach is the multi-file intersection.** `build_shared_gap_mask_from_bedgraphs()` (intersection across all input files) is the principled way to identify gaps: a bin is a gap only when every file agrees it's zero. Per-file zero intervals from a single bedGraph can be sample-specific dropouts that have nothing to do with the assembly. The shared mask is **better, not just "more conservative"** — it avoids treating per-file dropouts as gaps.

**Current state (verified 2026-04-27):**

- **Growth:** runs inline `frac_bad >= 0.9` masking + `_report_missingness()` diagnostic + post-call `annotate_calls_with_gap_distance()` (in `onionskin.py:520`) which uses the **single-file** mask via `build_gap_mask_from_bedgraph()`.
- **RMS:** runs `annotate_calls_with_gap_distance()` (in `onionskin.py:2965`); rebuilds the same single-file gap mask from scratch — **so growth and RMS both build the single-file mask, twice on the same input.**
- **HMM ref-stage:** removes zero bins from the bedGraph before HMM runs (math constraint of the ratio step). Stays as-is.
- **HMM no-ratio Step-3 (`_step3_remove_gap_bins` in `hmm_engine.py:325`):** uses `build_shared_gap_mask_from_bedgraphs()` — **the multi-file intersection variant — already does it the right way.**
- HMM does NOT currently use `annotate_calls_with_gap_distance()` for any post-call gap-aware annotation.

**Concrete deliverables:**

1. A pre-pipeline gap-analysis step in the controller, run after bin-size detection, before any pipeline runs. Uses `build_shared_gap_mask_from_bedgraphs()` against the input bedGraphs.
2. Emitted artifacts:
   - A shared gap-mask BED at e.g. `<out_dir>/01-prior/00-gap-analysis/<out_prefix>_shared_gap_mask.bed`.
   - A missingness diagnostic log (chrom-level summary, current `_report_missingness` content, lifted out of growth).
3. `annotate_calls_with_gap_distance()` updated:
   - Default behavior calls `build_shared_gap_mask_from_bedgraphs()` (multi-file intersection).
   - Legacy parameter (e.g., `strategy="single"`) preserves `build_gap_mask_from_bedgraph()` behavior for revertability.
4. HMM gains a post-call `annotate_calls_with_gap_distance()` invocation for cross-pipeline parity.
5. HMM chrom-median path consumes the pre-pipeline shared mask to **remove** gap bins from the bedGraph fed to HMM (separate need from zero-bin removal in ref-stage; the two coexist).
6. Growth and RMS inherit the updated mask default automatically. Verify they produce reasonable gap-distance metrics on existing test data (will differ slightly from current single-file results — that's the intended improvement).
7. CLI surface (TBD; likely controller-level): a way to override the gap-analysis source / strategy / chrom-median bin-removal threshold.
8. Tests:
   - Shared mask artifact produced and correct.
   - HMM ref-stage path produces accurate results (no regression from the unchanged zero-bin removal).
   - HMM chrom-median path correctly removes pre-computed gap bins before HMM sees the bedGraph.
   - HMM produces gap-annotated call output that matches growth/RMS in format/columns.

**Design constraints (preserve pipeline distinctness where it matters):**

- **HMM ref-stage mode keeps zero-bin removal.** Math constraint (zero denominators in the ratio step) + proven accuracy advantage over pseudocounts. Untouched by this change.
- **Growth's inline `frac_bad` masking stays** because it operates on the post-transform `Y` matrix (after `log2`, stage operations, etc.) — pipeline-internal, not on raw bedGraph data. If a later audit determines its post-transform check is in fact redundant with the shared missingness artifact, retire it then.
- **RMS's bedGraph-level processing internals** (signal extraction, per-chrom stage operations) are unchanged. What changes for RMS is its post-call flow: `annotate_calls_with_gap_distance()` will now use the multi-file shared mask by default.

**Counterpoints / things to weigh:**

- **Switching the default produces slightly different growth/RMS gap-distance results.** Per-file dropouts no longer count as gaps. That's the intended improvement; flag in CHANGELOG when the change lands and verify no downstream consumer is surprised.
- **Legacy code retention.** Keep `build_gap_mask_from_bedgraph()` available alongside the new default. Decide later (separate cycle) whether to remove it.
- **Performance is not the motivation.** Don't lean on speed arguments in the SPEC; lean on consistency and correctness.

**Cross-reference:** Add a short entry in `multi-agent/tracking/KNOWN_ISSUES.md` pointing here. The redundant single-file mask computation between growth and RMS, and the fact that growth/RMS use the less-principled single-file approach today, both warrant a near-term-issue cross-reference even though the full fix is Phase 15 work.

### BRAIN15.22 — Other HMM development items carried over from pre-14 recon audit

**Source:** SOUP15.16 (Other HMM development items carried over from pre-14 recon audit), tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md IBM-C2 / IBM-C4 / IBM-C5.

**Context:** The recon audit produced in late Phase 13 / pre-Phase-14 catalogued HMM-relevant items that should anchor a future HMM-completeness phase. Phase 15 IS that phase. The IBM-C anchor items are:

- **IBM-C2** — HMM parallel child pipeline + SAPS + step-14 APS raw-file fix → covered by **BRAIN15.6 + BRAIN15.20**.
- **IBM-C4** — HMM `peak_rcn_stage` column + sliding-offset sub-bin → covered by **BRAIN15.8** (gated on **BRAIN15.24**).
- **IBM-C5** — Cross-pipeline dynamic origin-detection window. **Design-mode work; not directly covered by another BRAIN entry.**

**IBM-C5 detail (from INTENDED-BUT-MISSED-PRIOR-TO-14.md and tracking/BRAINSTORM.md [2026-04-18] Cross-pipeline origin detection windows):** Design sketch for all three pipelines on dynamic onset / last-active-stage origin-detection. HMM has natural anchors (onset stage = first state-path-above-1 stage; last stage = last summit-state-higher-than-previous stage). Cross-pipeline equivalents need explicit definition.

**Phase 15 staged plan for IBM-C5 (per Phase 15 FEEDBACK Q3, 2026-04-28):** Full Phase 15 lifecycle — design + SPEC + implementation:

1. **Brainstorming stage:** Author a design sketch for all three pipelines (HMM, growth, RMS) covering dynamic onset and last-active-stage origin-detection windows. Flesh out the design fully during brainstorming, settling the cross-pipeline definitions and the per-pipeline mechanics.
2. **SPEC engineering stage:** Promote IBM-C5 to its own `SPEC15.<idx>` priority in `PHASE15_SPEC.md`, sourced from this BRAIN15.22 entry. The SPEC priority captures the agreed-on design + the implementation deliverables.
3. **Implementation stage:** Implement IBM-C5 inside the Phase 15 implementation cycle.

So IBM-C5 lands as **full concrete code in Phase 15**, not deferred — the staged plan ensures design quality before implementation.

**IBM-C5 background context (from user, Phase 15 FEEDBACK Q3, 2026-04-28):** Phase 13 Priority 13.3 II/9A follow-up; tracking/KNOWN_ISSUES.md `[ISSUE:2026-04-18:3]`. Design sketch was pending; user explicitly flagged this as "needing a design pass before more selector work."

**Phase 15 work for the bundle:** This BRAIN entry primarily serves as a cross-reference index — "the IBM-C anchor items live in these BRAIN IDs." IBM-C2 and IBM-C4 are covered by BRAIN15.6 / BRAIN15.20 / BRAIN15.8; IBM-C5 is the residual that this BRAIN entry now anchors with its own staged Phase 15 plan above.

### BRAIN15.23 — `indiv_samples/` path pre-definition cleanup

**Source:** SOUP15.8 item 3 ("indiv_samples/ path pre-definition	Dead code for unimplemented Phase 13 feature; add when the feature lands").

**Context:** Phase 13 had path pre-definition code (`indiv_samples/` references in path constants or layout helpers) for an unimplemented feature. Phase 15's BRAIN15.6 IS implementing that feature, so the dead code becomes live code at that point.

**Phase 15 work:**

- During BRAIN15.6 implementation, audit all `indiv_samples/`-related path constants and layout helpers.
- Remove any pre-definition that doesn't match the BRAIN15.6 design.
- Make sure the BRAIN15.6 implementation hooks into whatever pre-definition was already there (if it was correct) or replaces it cleanly (if it wasn't).

**Status:** Folded into BRAIN15.6 execution; this BRAIN entry exists as a tickler so the audit doesn't get missed.

### BRAIN15.24 — `peak_rcn_stage` term audit (definition + cross-onionskin uses + naming verification + correctness check before HMM adoption)

**Source:** Phase 15 transfer-stage user directive (chat 2026-04-27) — see `PHASE15_FEEDBACK.md` § "User-introduced BRAIN entries (no SOUP source)" for the approval log. This BRAIN entry has no SOUP source by design — it was added at BRAINSTORM authoring time at user direction to gate adoption of `peak_rcn_stage` semantics into the HMM pipeline.

**Why this audit is needed:** The user has, on multiple occasions during development, noted head-scratching about `peak_rcn_stage` — the exact intended definition has not been pinned down with the user, and there is concern that an agent may have implemented (or designed) the concept under a name or semantics that the user did not fully approve. Whether the implementation is correct, whether the name is right, and whether existing uses are aligned with intent — all are open questions.

**Current state (partially known as of 2026-04-27):**

- **The literal string `peak_rcn_stage` does not appear in `onionskin.py`, `onionskin_core/`, `scripts/`, or `tests/`** (verified via grep 2026-04-27). It appears only in planning surfaces — `multi-agent/tracking/BRAINSTORM.md`, `multi-agent/tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md`, and the Phase 15 SOUP/BRAINSTORM.
- **The literal-string negative raises confidence that the term itself is not implemented**, but the CONCEPT may still exist under a different name. Synonymous terms may exist scattered across BRAINSTORM, KNOWN_ISSUES, and possibly code — referring to the same concept under different names. Finding those synonyms (in planning surfaces AND code) is part of the audit.
- The audit therefore covers **both** unimplemented design intent (planning-surface term to be implemented) AND the possibility that the concept lives under a different name in code or planning surfaces. We do not know yet which case we are in.

**User-recalled tentative original meaning (chat 2026-04-27, captured for the audit's starting point — to be confirmed during step 2 below):**

`peak_rcn_stage` = **the last stage at which summit growth is detected relative to the previous stage**. After that stage, the summit is either flat or shows a small downward drift; the small downward drift is to be treated as still flat (the user notes the rationale is discussed elsewhere — find that during the audit).

This pairs naturally with `onset_stage` (the earliest stage at which growth is detected). The two together bracket the active-growth window for an amplicon's summit.

**Adjacent contested area — stage 1 growth detection (relevant to BRAIN15.24's scope and to any agent doing this audit):**

In stage 1, growth must be detected **against background**, NOT against a previous stage (there is no previous stage). In stages 2+, growth is detected **relative to the previous stage**. Agents have repeatedly insisted "stage 1 growth is unknowable" — this is **wrong** and the user has had to correct it multiple times. Any audit, design, or implementation work touching `peak_rcn_stage`, `onset_stage`, or related growth-detection-by-stage concepts must operate under the rule: stage 1 growth IS detectable (against background), and the agent must not silently revert to "unknowable" framing.

**Audit timing (per Phase 15 FEEDBACK Q2, 2026-04-28):** The audit runs as part of the **Phase 15 brainstorming stage** (i.e., after the soup-to-brainstorm transfer closes out and the official brainstorming stage begins). User direction: *"It is already 'tomorrow' but yes let's figure out peak_rcn_stage as part of brainstorming. That will help us brainstorm correctly."* The audit's findings then feed into BRAIN15.8 + any other BRAINSTORM entries that touch the term, which propagate into SPEC engineering.

**Audit scope:**

1. **Code + planning-surface search: find any existing patterns that implement (or describe) the same concept under a different name.** Two complementary searches:
   - **Planning surfaces:** grep `multi-agent/tracking/BRAINSTORM.md`, `multi-agent/tracking/KNOWN_ISSUES.md`, `multi-agent/tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md`, archived phase plans for synonymous terms naming the same concept under a different name. The user has flagged that synonymous terms likely exist scattered across our writings.
   - **Code:** grep `argmax` (in stage-axis contexts), `peak_stage`, `best_stage`, `summit_stage`, `max_summit`, `peak_idx`, `best_idx`, `last_growth_stage`, `last_active_stage` (the last one explicitly mentioned in tracking/BRAINSTORM.md [2026-04-18]), `last_summit_stage` — across `onionskin_core/refinement.py`, `onionskin_core/engines/growth_model_engine.py`, `onionskin_core/rcn_mean_shift_helpers.py`, `onionskin_core/engines/rcn_mean_shift_engine.py`, `onionskin_core/hmm_summits.py`, `onionskin_core/hmm_metrics.py`, `onionskin_core/hmm_fork_travel.py`.
   - For each hit (planning or code), document: (a) the term used, (b) the operational definition (what it actually computes or describes), (c) where it appears, (d) whether it matches the user's tentative recalled meaning (above) — match, partial-match, or different concept.
   - Pay particular attention to `last_active_stage` (mentioned in tracking/BRAINSTORM.md [2026-04-18] cross-pipeline origin detection windows entry) — this term specifically suggests "the last stage at which X is observed," which is structurally close to the user's tentative recall.
2. **Pin down the exact intended semantics of "peak RCN stage" with the user.** Walk through:
   - What value is being computed? (Per-amplicon? Per-bin? Per-summit-position?)
   - What "peak" means here — maximum of summit-bin RCN across stages? Maximum of summit interval mean? Something else?
   - What "stage" means here — developmental stage index? Per-sample stage? Per-bin stage assignment?
   - Why the user was head-scratching: is there an alternative interpretation that would be more biologically meaningful?
3. **Compare the intended semantics (step 2) to existing implementations (step 1).** Three possibilities:
   - **(a) Match.** Existing implementation matches intended semantics; just needs the agreed-on name and adoption into HMM.
   - **(b) Partial match — needs correction.** Existing implementation captures the intent but has subtle issues (wrong axis, wrong reduction, edge cases, etc.). Correct the implementation; propagate to all callers.
   - **(c) No existing implementation.** The concept is unimplemented; design and implement it from scratch under the agreed-on name.
4. **Validate the name itself.** Is `peak_rcn_stage` the right name for the agreed-on semantics, or does a clearer term emerge? Candidate alternatives to consider: `summit_peak_stage`, `max_rcn_stage`, `best_summit_stage`, etc. If existing code uses a name that's actually clearer than `peak_rcn_stage`, prefer the existing name.
5. **Lock in the corrected definition + name + propagation plan.** Document the agreed-on semantics, final name, column placement, and (if step 3 found existing code) the correction + rename + caller-update plan. If a rename is decided, propagate to:
   - All existing live code identified in step 1.
   - `multi-agent/tracking/BRAINSTORM.md` planning entries that mention `peak_rcn_stage`.
   - `multi-agent/tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` IBM-C4 entry.
   - Phase 15 BRAINSTORM (BRAIN15.8 here, plus any other entry that references the term — check this file for completeness).
   - Phase 15 SOUP (read-only after labeling pass — document the rename decision in FEEDBACK rather than editing the SOUP body).
   - Help text and docs for any flag that consumes the column.
6. **Gate BRAIN15.8 (HMM `peak_rcn_stage` + sliding-offset adoption) on this audit.** HMM does not adopt the term/column until BRAIN15.24 clears AND any cross-onionskin correction (case b above) lands first. If case (a) — clean match — HMM adoption is unblocked immediately. If case (c) — implement first, then adopt.

**Output of the audit:** A short audit report (could be a FEEDBACK entry, a separate BRAINSTORM update, or a tracking/BRAINSTORM.md note) documenting the agreed-on semantics + name + cross-onionskin propagation plan. After the audit, BRAIN15.8 unblocks.

**Cross-reference:** This audit pattern (term audit before propagating to a new pipeline) is generalizable. Worth noting in `multi-agent/AGENT_CONVENTIONS.md` as a recommended practice for any term that has accumulated head-scratching during development — flag for SPEC engineering or post-Phase-15 closeout.

#### Audit step 1 partial-result — found in aggregation pass (2026-04-28, brainstorming-stage Round 3)

The brainstorming-stage aggregation pass surfaced two existing columns in `onionskin_core/aps.py` that implement closely-related (but not identical) versions of the `peak_rcn_stage` concept:

**Existing column 1: `max_rcn_stage`** ([onionskin_core/aps.py:1016](onionskin_core/aps.py#L1016), computed at [onionskin_core/aps.py:978](onionskin_core/aps.py#L978)).

- Definition (verbatim from doc-string at [onionskin_core/aps.py:876-878](onionskin_core/aps.py#L876)): *"Post-onset stage at which the summit-window median RCN is highest. NaN when stage_stats is absent or the window is empty."*
- Computation: `max_rcn_stage = max(post_win_stages, key=lambda s: win[s][0])` — i.e., **argmax of summit-window-median RCN across post-onset stages.**
- Provenance: implemented at v0.5.49 as part of the oscillation-annotation work (see `tracking/BRAINSTORM.md [2026-04-01] APS locus diagnostics — design discussion and open questions` lines 925–933 — the planning-surface entry that motivated this column suggested the name `peak_rcn_stage` but it landed in code as `max_rcn_stage`).

**Existing column 2: `regression_stage`** ([onionskin_core/aps.py:1017](onionskin_core/aps.py#L1017), computed at [onionskin_core/aps.py:1003](onionskin_core/aps.py#L1003)).

- Definition (verbatim from doc-string at [onionskin_core/aps.py:880-884](onionskin_core/aps.py#L880)): *"Stage s+1 of the first confirmed dip at or after max_rcn_stage. NaN if no confirmed dip occurs after the peak stage. NOTE: includes dips at global degradation stages."*
- Adjacent / complementary to `max_rcn_stage` — defines the FIRST stage post-peak where a confirmed dip is detected.

**Comparison to the user's tentative recalled semantics for `peak_rcn_stage`:**

User's recall (verbatim 2026-04-28): *"`peak_rcn_stage` is the last stage at which summit growth is detected with respect to the previous stage; after that it is either flat or goes down a little bit (the going down is basically to be treated as still flat though for reasons we can discuss)."*

| Aspect | User's recall | `max_rcn_stage` (live code) | `regression_stage` (live code) |
|---|---|---|---|
| **Quantity computed per locus** | "Last stage of growth w.r.t. previous stage" | "Stage where summit-window-median RCN is highest" (argmax) | "First stage post-peak with confirmed dip" |
| **Comparison axis** | Per-transition: RCN_s vs RCN_{s-1} (growth detection) | Global: argmax across stages | Hybrid: post-peak transition + dip-confirmation test |
| **Treatment of post-peak slight decline** | "Treated as still flat" — implies a tolerance for small drops | Argmax ignores; if a drop is followed by a plateau-tied-with-peak, behavior is well-defined; if a drop is followed by a slightly-lower value, argmax still picks the original peak stage | Confirmed-dip threshold determines what counts as a "dip"; sub-threshold drops are not flagged |
| **Match assessment** | — | **Partial match.** Returns the same answer as user's recall in monotonic-growth-then-flat scenarios. Diverges in scenarios where: (a) growth pauses then resumes (argmax picks the global highest, user's "last growth stage" depends on whether the pause counts as growth), (b) a slightly-lower stage interrupts (argmax picks the original peak; user's "still flat" rule may pick the slightly-lower stage as still-growing). | **Different concept.** Tracks the START of regression after peak, not the LAST growth stage. But the user's "stages 1..regression_stage-1 = growth-active" framing IS consistent with regression_stage's definition (subject to the dip-confirmation threshold matching the user's "treated as still flat" tolerance). |

**Provisional case classification (from the audit's three-case matrix in step 3):**

This is **case (b) Partial match — needs correction or refinement.** The existing `max_rcn_stage` captures the spirit of the concept but the semantics don't exactly match the user's tentative recall. The adjacent `regression_stage` column captures a related but distinct quantity. The user-walkthrough audit step needs to settle:

1. Which of the three exact-semantic candidates is the desired definition: argmax-based (`max_rcn_stage`-style), per-transition-growth-based (user's recall), or first-confirmed-dip-based (`regression_stage`-style)?
2. Whether the "going down a little bit / treated as still flat" tolerance is best implemented as a confirmed-dip threshold (already done in `regression_stage`) or as some other tolerance mechanism applied to the argmax variant.
3. Whether the right answer is one column or three columns (e.g., keep `max_rcn_stage` + `regression_stage` AND add a new `last_growth_stage` column for the user's recalled semantics — three different per-locus quantities, all biologically meaningful).

**Naming considerations:**

- `max_rcn_stage` is a precise name for what the code does (argmax-based).
- `regression_stage` is precise for its definition.
- `peak_rcn_stage` (the planning-surface name) is ambiguous — could mean any of the three semantics above; matches `max_rcn_stage`'s spirit but isn't tied to a specific definition.
- A new column for the user's recalled semantics could be named `last_growth_stage` (more precise) or `growth_end_stage` (similar) or kept as a refined `peak_rcn_stage` that explicitly redefines the semantics.

**Audit step 4-5 propagation considerations:** If the user's intended semantics differ from `max_rcn_stage`, the audit must decide: do we (a) rename / redefine `max_rcn_stage` to match user intent (risks breaking downstream that already consumes `max_rcn_stage`), (b) add a new column with the user's semantics alongside the existing `max_rcn_stage` (no breakage; clearer), or (c) keep `max_rcn_stage` as-is and use a NEW column under a different name in HMM (silos but avoids touching live code).

**Other planning-surface mentions of `peak_rcn_stage`:**

- `tracking/BRAINSTORM.md [2026-04-01] APS locus diagnostics` lines 833–835: *"The stage of maximum mean `peak_rcn` is implicitly defined by this, but not explicitly reported — it should probably be added as its own column (`peak_rcn_stage`)."* — This is the seed planning note that led to `max_rcn_stage` (renamed in implementation).
- `tracking/BRAINSTORM.md [2026-04-14] HMM summit estimation` (already cited at BRAIN15.8): proposes `peak_rcn_stage` for HMM specifically, expecting it to identify the stage of maximum summit RCN per amplicon.
- `tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` IBM-C4: re-flags the term in a Phase-9 follow-up framing.

**Recommendation for the audit's user-walkthrough step:** Open the audit by walking through one or two example amplicons in DS1 chr II using the existing `max_rcn_stage` and `regression_stage` outputs alongside the user's recalled-semantics computation. The discrepancy (or agreement) on real data will inform whether case (a) clean-match, case (b) correction-needed, or case (c) need-new-column.

**Holding bucket — synonymous-concept code search results (aggregation pass, 2026-04-28):**

The literal grep for `peak_rcn_stage` returns only planning-surface mentions (no code). Concept-search via `argmax`, `max_rcn_stage`, etc. yields:

- `onionskin_core/aps.py:976-978, 1016` — `max_rcn_stage` argmax computation (described above).
- `onionskin_core/aps.py:1003, 1017` — `regression_stage` first-post-peak-confirmed-dip (described above).
- `onionskin_core/aps.py:1271-1285` — `is_oscillator` flag derived from `oscillating_transitions`; marks loci where post-onset stages show confirmed dips that aren't at global degradation stages.
- `onionskin_core/timing.py:312` — `max_stage_max_peak_RCN` column emitted by timing pipeline.
- `onionskin_core/timing.py:408` — `max_idx = int(np.nanargmax(peak_rcn))` — internal use of argmax-style stage selection.
- `onionskin_core/summit_plots.py:200` — *"Per-stage summit positions: argmax of each stage's already-smoothed RCN profile"* — different concept (per-stage argmax, not across-stage argmax) but related machinery.

**Audit step 1 status: PARTIALLY COMPLETE.** The code-search portion of step 1 is now done (above). The user-walkthrough portion of step 2 (pin down exact intended semantics) is still pending — that's the activity the user described as "we can audit it even tonight - later on as part of brainstorming." This audit-step partial result feeds into that walkthrough.

#### Round 4 update (2026-04-28) — Origin Detection Window (ODW) framework, 4-metric system, elongation interference

The user's Q15 answer (Phase 15 FEEDBACK Round 3, 2026-04-28) settled the audit's user-walkthrough step (step 2) at framework level. The semantics are no longer "which one of three candidates" — instead, **all four metrics are kept; each gives different information; their relationship is unified by the Origin Detection Window (ODW) concept.**

**The four metrics (all keep, none retire):**

| Metric | What it computes | Status |
|---|---|---|
| `onset_stage` | First stage where summit growth is detected (vs. background in stage 1; vs. previous stage in stage 2+). The start of the ODW. | Already exists in timing module; semantics already match user intent. |
| `last_active_stage` | Last stage where summit growth is detected w.r.t. previous stage (with "still flat" tolerance for slight dips that aren't confirmed regression). The end of the ODW. **Was previously conflated with `max_rcn_stage` by an agent — they are SEPARATE concepts.** | Not yet a dedicated column; user's recalled semantics for the historical `peak_rcn_stage` planning concept best match this metric. **Needs implementation** as a new column under whatever name we settle on (`last_active_stage` is a strong candidate). |
| `max_rcn_stage` | Argmax of summit-window-median RCN across post-onset stages. | Already exists in `onionskin_core/aps.py:1016`. May or may not coincide with `last_active_stage`. |
| `regression_stage` | Stage s+1 of the first confirmed dip at or after `max_rcn_stage`. | Already exists in `onionskin_core/aps.py:1017`. Captures regression-onset, NOT the same as `last_active_stage` (a flat period before regression would be inside `last_active_stage` but `regression_stage` is the stage AFTER regression begins). |

**The unifying concept — ODW (Origin Detection Window):**

ODW = **stages from `onset_stage` to `last_active_stage`** = the active-firing window where the replication origin is firing and the summit profile is most reliably peak-shaped. Outside the ODW (specifically: in stages after `last_active_stage`), **"elongation interference"** sets in — replication forks that have traveled outward from the origin inflate flanking bins more than the origin bin itself, shifting the parabola center toward flank signal rather than origin signal. The summit gets flatter, less peak-like; what looks like "the highest stage" by argmax may have the worst signal-to-noise on origin location.

**Why this matters for HMM summit estimation (BRAIN15.8 + BRAIN15.29):**

User-quoted summary (verbatim from Phase 15 FEEDBACK Round 3 Q15 answer, 2026-04-28):

> *"The newer idea would be to identify the 'origin detection window (ODW)' which starts at the stage where summit growth is first detected and ends at the last stage summit growth is detected — this are setting boundaries on stages where the origin is actively firing, and the summit is expected to be more pointy and peak-like. In stages after the ODW, the origin is no longer active or is much less active, and the 'summit' region is extended by the last set of replication forks to something more like a flat table top over a broader region — and where the origin was gets less and less obvious the longer this flat top is."*

> *"This actually also extends to the stage of maximum summit RCN. Sometimes this will work if the max captured the pointiness well. In others where the max had replication forks that traveled far, the signal:noise of origin location will be much less. Even if it is at the end of the ODW, other stages in the ODW might have better signal."*

**Implication:** HMM summit estimation should **confine its input to stages within the ODW**, not just pick the argmax stage and not just average across all stages. The right-active-stage selection is per-amplicon — different amplicons have different ODWs depending on their developmental firing timing.

**How `last_active_stage` should be detected (from user's Round 3 Q15 chat):**

> *"Stage 1 growth (or stage 2 when 1 is the ref-stage) is detected relative to background since there is no prior stage. Stages 2-N use some statistical test to compare stage S to stage S-1 to see if there was growth relative to the previous stage. At some point we did come up with a bunch of tests for this — they might be in the timing module."*

> *"For comparing stage S to S-1, it does not need to be a ratio between them... we can just look at the series of summit RCN values from stage 1 to N. We already do growth curves — we can look at the input data for the growth curve plots. We could fit a loess line or something so a random oscillation (high outlier) does not necessarily screw up the analysis."*

So `last_active_stage` detection has multiple candidate implementations: per-transition statistical test (timing-module style) OR fit-line-based (loess on summit RCN series across stages). The "still flat tolerance" naturally handles the case where the summit-RCN curve plateaus — the plateau IS active stages by user's definition (origin is still firing but at saturation; not regressing).

**Important distinction from `regression_stage`:**

- `regression_stage` = first stage post-peak where a CONFIRMED DIP is detected.
- `last_active_stage` = last stage where GROWTH IS DETECTED (with "still flat" tolerance).
- These can differ: a flat period (no growth, no confirmed dip) sits BEFORE `regression_stage` but AFTER `last_active_stage`. Both metrics carry useful information.

**Relationship to existing planning surfaces:**

- `tracking/KNOWN_ISSUES.md [ISSUE:2026-04-18:3]` (Dynamic onset / last-active-stage origin-detection window — needs explicit cross-pipeline design sketch) is the substantive prior thread on ODW. This issue was already cited at BRAIN15.22 (IBM-C5 cross-pipeline ODW design sketch).
- `tracking/BRAINSTORM.md [2026-04-18] Cross-pipeline origin detection windows — dynamic onset / last-active-stage summit refinement` is the corresponding BRAINSTORM entry (already cited at BRAIN15.22).
- `tracking/BRAINSTORM.md [2026-04-14] HMM summit estimation — use stage of maximum summit RCN, not last stage` was the precursor; user noted this is partially STALE — superseded by the ODW framework. The "use max stage" framing is correct in spirit (don't use last stage; don't average all stages) but doesn't capture the more principled ODW approach. The [2026-04-14] entry's observations about "elongation interference" on ds2 chr II (stages 2–4 give origin overlap; stage 4 already shifted; stage 5 substantially shifted) are STILL useful as evidence for why ODW matters — that's the dataset-side reasoning.

**Audit step 4-5 propagation plan (now well-defined post-Round-4):**

1. **Implement `last_active_stage` as a new per-locus column** (in HMM, growth, RMS where the per-stage summit-RCN data is available). The exact statistical test for "growth detected w.r.t. previous stage" + "still flat tolerance" is BRAIN-level open question for SPEC engineering; candidate approaches: per-transition statistical test, loess-fit-based, growth-curve-input-style.
2. **Keep `max_rcn_stage` and `regression_stage` as-is** (they are valid and useful as separate columns).
3. **Add ODW boundary metadata** to per-amplicon outputs: `odw_start_stage` (alias of `onset_stage`), `odw_end_stage` (alias of `last_active_stage`), `odw_duration_stages` (= `odw_end_stage` - `odw_start_stage` + 1).
4. **Confine summit-estimation inputs to the ODW** for BRAIN15.8 (HMM peak_rcn_stage + sliding-offset) and BRAIN15.29 (summit-then-timing one-pass). The "stage of maximum summit RCN within ODW" replaces "stage of maximum summit RCN across all stages" as the basis for summit refinement.
5. **Document "elongation interference"** as the project-wide term for the post-ODW phenomenon. Useful in code comments, plot annotations, and SPEC priorities.
6. **Cross-pipeline harmonization** — IBM-C5 / BRAIN15.22 is the design sketch for ODW across all three pipelines. HMM has the most leverage (state-path data), but growth and RMS need analogous ODW machinery.

**Audit step 4 (validate the name) — recommendation post-Round-4:**

- Keep `onset_stage` (already-correct name).
- Add `last_active_stage` as the new column for the user's recalled "peak_rcn_stage"-conflated semantic. Strong-recommend this name over `peak_rcn_stage` because:
  - It accurately describes what the metric computes (last stage of growth activity).
  - "peak_rcn_stage" is too easily conflated with `max_rcn_stage` (which IS argmax-of-RCN).
  - "last_active_stage" is already used in `tracking/BRAINSTORM.md [2026-04-18] Cross-pipeline ODW` as the established planning-surface term for this concept.
- Keep `max_rcn_stage` (already-correct name; precise definition).
- Keep `regression_stage` (already-correct name; precise definition).
- **Retire the term `peak_rcn_stage`; keep the argmax-based stage-selection IDEA under the canonical name `max_rcn_stage` (per Q23 user clarification, Round 5 update):**

  - The TERM `peak_rcn_stage` is **retired**. It only ever lived in planning surfaces (`tracking/BRAINSTORM.md`, IBM file, Phase 15 SOUP/BRAINSTORM); it never landed in code. The audit confirmed: where `peak_rcn_stage` was used in planning surfaces, the intended concept was the same as the existing `max_rcn_stage` column in `onionskin_core/aps.py:1016` (= argmax of summit-window-median RCN across post-onset stages). So `peak_rcn_stage` and `max_rcn_stage` are **synonyms for the same concept**; consolidate to `max_rcn_stage` (the code-canonical name).
  - The IDEA of argmax-based stage selection for summit modeling — i.e., *"use the stage where summit RCN is highest"* — is **kept**, NOT retired. Per Q23 user direction: *"don't eliminate it, just make it a parenthetical to the ODW stuff in general, and we can actually implement the concept of summit modeling using the max_rcn_stage to test it out."* The IDEA becomes one strategy on the per-amplicon stage-selection menu (see BRAIN15.8 Round 5 expansion).
  - Where `peak_rcn_stage` appears in tracking-file planning entries (e.g., `tracking/BRAINSTORM.md [2026-04-14] HMM summit estimation — use stage of maximum summit RCN, not last stage`; IBM-C4), treat it as a synonym for `max_rcn_stage`. Cleanup of those tracking-file mentions is part of BRAIN15.34's Phase 15 closeout sweep (replace `peak_rcn_stage` references with `max_rcn_stage` + add provenance annotation).
  - **Naming summary, post-Q23 + Round 6 update for `latest_activity_stage` finding:**
    - `onset_stage` — first stage where summit growth is detected (vs. background in stage 1; vs. previous stage in stage 2+). ODW start.
    - `last_active_stage` — last stage where growth is detected (with "still flat" tolerance; non-monotonic-OK per Q23). ODW end. **Round 6 finding (per Codex audit, 2026-04-28):** **THIS CONCEPT ALREADY EXISTS IN CODE** as `latest_activity_stage` in `onionskin_core/timing.py:300-317, :466-468`. The 1.25 fold-change threshold for stage-over-stage active-growth detection lives at `timing.py:370-393`. **Decision (per user 2026-04-28 chat): rename `latest_activity_stage` → `last_active_stage` cross-pipeline.** Single canonical name; no synonyms; no "two names for the same thing" maintenance burden. The rename is contained to `onionskin_core/timing.py` (column emit + classifier) + downstream callers that read the column. Tracking-file mentions of `latest_activity_stage` get cleaned up as part of BRAIN15.34's Phase 15 closeout sweep.
    - `max_rcn_stage` — argmax of summit-window-median RCN across post-onset stages. Existing column at `onionskin_core/aps.py:1016`. **`peak_rcn_stage` is the retired synonym for this.**
    - `regression_stage` — first stage post-peak with confirmed dip. Existing column at `onionskin_core/aps.py:1017`.

  - **Round 6 connection to BRAIN15.7 (amplicon-class taxonomy):** the `latest_activity_stage` column lives in the same timing.py module as `amplicon_class`, `Constitutive_Prolonged` / `Facultative_Prolonged` / `Founder` / `Finisher` / `Intermediary` / `Excluded` classifications. These are tightly coupled — the classifier uses `latest_activity_stage` as one of its inputs. Phase 15's ODW-vs-`latest_activity_stage` rename decision is therefore also a decision about the timing.py classifier's column schema. Cross-reference BRAIN15.7 Round 5 expansion + Round 6 update. **Per user direction 2026-04-28: rename to `last_active_stage` everywhere** — single canonical name; no synonyms.

  - **Round 6 connection to the Gaussian-overlap probabilistic dip test** (`onionskin_core/aps.py:748-1041`, v0.5.48): the `regression_stage` metric (from `aps.py:1003, :1017`) is computed using a more sophisticated variance-aware statistical test (`scipy.stats.norm.cdf` of dip probability with sigma = MAD × 1.4826, using per-stage MAD bedGraphs from v0.5.47) than the 1.25× fold-change test that defines `last_active_stage` (per timing.py:370-393). The two tests are complementary: 1.25× tests the UPWARD direction (active growth between stages); Gaussian-overlap tests the DOWNWARD direction (confirmed dip between stages).

  - **Decision (per user 2026-04-28 chat): the 1.25× hard rule is OFFICIALLY being retired** in favor of the **Bayesian Origin Detection Window (ODW) System** — a probabilistic / variance-aware test framework with three configurable detectors (`hardthreshold`, `gaussian_overlap`, `bayesian_posterior`; default `bayesian_posterior`) and two tunable-knob sibling CLI flags (`--odw-prob-threshold` default 0.9; `--odw-fold-threshold` default 1.25). The system is broader than `last_active_stage` alone: its outputs feed `onset_stage`, `last_active_stage`, the per-transition active mask, amplicon-class classification, and (potentially) `regression_stage`. The tunable knobs let users tighten the ODW (raise thresholds → fewer-but-higher-confidence stages → better summit signal for BRAIN15.8) or loosen (more inclusive → better amplicon recall). Full system spec lives in BRAIN15.7 Round 6 deliverable item 6.

**Audit step 6 (gate on BRAIN15.8 / BRAIN15.29):**

- BRAIN15.8 (HMM summit refinement) — Round 5 update extends the entry with a **menu of per-amplicon stage-selection strategies for testing** (see BRAIN15.8 Round 5 expansion). The `max_rcn_stage` selection IS one of the strategies on the menu; ODW-confined methods are others. Both get evaluated.
- BRAIN15.29 (summit-then-timing-then-updated-summit-then-updated-timing one-pass convergence) — already aligned with the ODW concept. The "active stages" referenced in BRAIN15.29 step 3 are exactly the ODW. Update BRAIN15.29 to use ODW terminology directly during SPEC engineering.

**Audit step 1 status: COMPLETE post-Round-4.** Step 2 (user-walkthrough) is settled at framework level via Q15 answer + Q23 follow-up. Steps 3-6 (compare-to-existing, validate-name, lock-in, gate-BRAIN15.8/29) are now well-defined. Implementation-time SPEC engineering will fill in the remaining details (which statistical test for `last_active_stage` detection; how to expose ODW boundaries in output schemas; menu of per-amplicon stage-selection strategies to test in BRAIN15.8).

### BRAIN15.25 — HMM two-pass background-region-anchored chromosome-specific renormalization

**Source:** Phase 15 FEEDBACK Q4 user directive (2026-04-28) — separated out from the SOUP15.17 / BRAIN15.9 `--hmm-mu-scale` correctness work as an independent idea per user direction *"this should be a separate idea — do not bundle into the mu-scale bug fix."* This BRAIN entry has no SOUP source by design — added at user direction during BRAINSTORM authoring per the same convention as BRAIN15.24.

**Goal:** Implement a two-pass HMM run where the first pass identifies background regions per chromosome, and the second pass uses ONLY those background regions to compute the chromosome-specific normalization. The motivation: ensure the background-region median/mean genuinely equals 1 (as expected) by excluding amplicon and collapsed-repeat regions from the normalization pool. The current chromosome-specific median norm (ref-stage or not) gets close to this but does not principally exclude above-background regions from its background pool.

**Design (verbatim user spec, Phase 15 FEEDBACK Q4, 2026-04-28):**

The new flag controls behavior. Candidate flag: `--hmm-bkgrd-renorm on|off` (default: `off` initially; future default to `on`).

When the flag is `on`:

1. **Pass 1:** Run steps 1–8 of the HMM pipeline normally.
   - Pass-1 results go in subdirectories under each step directory called `pass1/`.
2. **Define background regions** on each chromosome at step 8 (pass 1):
   - Background regions = intervals NOT classified as above-background.
   - For all things above background (amplicons AND collapsed repeats), use their **widest** representation found across stages.
     - This excludes as much possibly-above-background material as possible from the background pool.
     - This is the **most conservative** classification: bins are classified as background ONLY if they are never seen to be above background in any stage.
3. **Pass 2:** Re-run the pipeline using the background regions to drive normalization:
   - Restart at step 1.
   - Step 1 (re-run): only use background regions to determine the median.
     - Pass-2 results go in the **top level** of step directory (or under `pass2/` — see directory-layout decision below).
   - Steps 2–3 (re-run): proceed normally.
   - Step 4 (re-run): only use chromosome-specific background regions for renormalization.
   - Steps 5–8 (re-run): proceed normally; pass-2 results store as above.
   - Steps 9+: continue with pass-2 inputs; results live at top level.

**Concrete deliverables:**

1. The `--hmm-bkgrd-renorm` flag (or equivalent name TBD during SPEC engineering).
2. A "widest representation across stages" reduction for amplicon + collapsed-repeat intervals at step 8 pass 1.
3. A background-region-aware variant of the step-1 median computation.
4. A background-region-aware variant of the step-4 chromosome-specific renormalization.
5. Pipeline plumbing to do the second pass — re-running steps 1–8 with the new normalization inputs.
6. Directory layout for pass1/pass2 outputs (see open question below).
7. Tests verifying the two-pass run produces background-region medians close to 1 on representative datasets.

**Open question — directory layout (agent opinion solicited per user note):**

Two candidate directory layouts:

- **(A) `pass1/` and `pass2/` subdirectories under each step directory:**
  - Pro: explicit and unambiguous which results came from which pass.
  - Pro: side-by-side comparison is trivial — both passes' outputs persist for downstream inspection.
  - Con: every code path that reads from a previous step's directory now needs to know which pass it's reading from — significant code change surface.
- **(B) Top-level results in each step directory at all times; pass1 results moved to `pass1/` only when pass 2 begins:**
  - Pro: minimal code change. For steps 1–8, pass-1 results live at top level until pass 2 starts (so all downstream-of-1 code reads from the same well-known path through pass 1). At pass-2 boundary, current top-level results move to `pass1/`, then pass 2 runs and writes new top-level results.
  - Pro: "final" results are always at top level — useful both for human inspection and for downstream / post-hoc scripts that don't care about pass provenance.
  - Con: intermediate state (during the move + re-run) is more fragile to interruption; need atomic-rename or a marker file to indicate which state the directory is in.

**Agent recommendation:** Option **(B)** is more pragmatic. The "final results always at top level" property is genuinely useful for downstream consumers, and the migration cost in (A) is large. The interruption-fragility concern in (B) can be mitigated by writing a sentinel file (`.pass1_archived` or similar) that pass 2 checks and respects. If the user wants explicit pass provenance throughout, (B) can be extended with a small `pass1_archived_at: <timestamp>` marker file at top level that points at the `pass1/` subdirectory.

**Final layout decision deferred to SPEC engineering** — this is an architectural call that affects how downstream code reads files, so SPEC engineering should validate the choice against the actual code paths that will be touched.

**Phase placement:** Phase 15. The default is `off` initially; once the implementation is validated on real data, a future phase can flip the default to `on`.

### BRAIN15.26 — HMM `--norm-mode` chrom-median default switch (gated on filters + multistage unification)

**Source:** [ISSUE:2026-04-21:1] HMM `--norm-mode` chrom-median consideration (deferred design note); user directive in Phase 15 FEEDBACK Q9 (2026-04-28). This BRAIN entry promotes the deferred consideration into Phase 15 scope at user direction.

**Goal:** Switch the HMM `--norm-mode` default from `ref-stage` to `chrom-median`. This is **late-Phase-15 work** — it cannot land until the prerequisites (filters + multistage unification) are in place so chrom-median can be used safely without ref-stage's ability to filter out collapsed repeats etc. via reference comparison.

**Prerequisites (per user Q9, 2026-04-28):** Switching to chrom-median by default depends on:

1. **HMM shape filter / triangle-based multistage unification** (BRAIN15.18 covers this — HMM gains a shape-filter sink + cross-stage shape meta-analysis for collapsed-repeat detection) — must land first.
2. **HMM state-path-evolution-based multistage unification** (also part of BRAIN15.18 sub-priority (c) — "detect state-path growth as shape credibility signal" — substantially expanded by the user's Q9 answer; see BRAIN15.18 follow-up note below) — must land first.
3. **The combined multistage unification flag** that lets the user pick which classification mode to use (triangle-only / state-path-only / both-required / either-suffices) — must be wired before chrom-median can become the default safely.
4. **BRAIN15.21** (lift shared gap mask + missingness diagnostic to pre-pipeline step) — chrom-median consumes the pre-pipeline shared gap mask to remove gap bins consistently across files. Already in BRAIN15.21's deliverables; this is a hard prerequisite.

**Why these prerequisites matter (user-explained, Q9, 2026-04-28):**

> *"This will be near the end of PHASE15. It will depend on putting filters in place to remove things like collapsed repeats."*

> *"We just got done with using the shape filter in the RMS pipeline as part of multistage unification (step 8, 08-multistage-unification). This worked very well, and allows RMS to not depend on ref-stage to eliminate things like collapsed repeats. Multistage unification involves creating a union set of calls across all candidate amplicons and collapsed repeats, then running the shape filter on each candidate location across all stages, and using the results to give a final multistage classification of each candidate, splitting the union of candidates into a partition of amplicons and collapsed repeats. This is the final authoritative set of calls across all data that the pipeline can move forward with."*

> *"The same exact thing needs to be part of the HMM pipeline."*

> *"In addition, the HMM pipeline allows us to monitor state path evolution across the stages for the final multistage unification of amplicon calls vs collapsed repeat calls."*

> *"So yes, chrom-median should be the default after all the right filters and multistage unification results are in place. Then downstream results would depend only on candidates with 'final' classifications as amplicons."*

**Phase 15 deliverable:** The default switch itself is a small change — just flipping the default value of `--hmm-norm-mode` in `onionskin.py:build_parser()`. The bulk of the work is the prerequisite landings (BRAIN15.18 expansion + BRAIN15.21). This BRAIN entry tracks the actual default-switch as a separate priority because it's gated on those prerequisites and should land near end-of-Phase-15.

**Validation criteria for the default flip:**

- Background-region medians near 1 across chromosomes on representative datasets.
- No regression in summit accuracy (II/9A, II/2B) compared to ref-stage on the same datasets.
- Multistage-unified amplicon set is stable under the switch.
- PuffStep gold-standard `make puff-compare` results either continue to match (if PuffStep agrees with chrom-median outputs at this point) OR `make puff-compare` runs explicitly use `--hmm-norm-mode ref-stage` to preserve gold-standard comparability.

### BRAIN15.27 — Composite multi-feature APS clustering modes with three-layer PCA composition

**Source:** Phase 15 transfer-stage user directives (chat 2026-04-28; Q13 follow-up; multi-round design discussion). No SOUP source; logged in `PHASE15_FEEDBACK.md` Round 2 follow-up under "User-introduced BRAIN entries (no SOUP source)" per the same convention as BRAIN15.24, BRAIN15.25, BRAIN15.26. Rejected feature-mode candidates (Fourier descriptors, curvature features, multi-family-BIC vectors, `peak_rcn` as a distinct feature) and the RCN-vs-excess design decision are documented in `multi-agent/project_context/DECISIONS.md` under the 2026-04-28 APS feature mode entry; the Round 2 follow-up FEEDBACK section back-references that DECISIONS entry.

**Goal:** Add a family of new `--aps-feature` modes that build per-sample feature vectors from already-computed per-amplicon morphology + per-sample APS aggregates + amplicon-level shape PCA scores, with optional matrix-level PCA preprocessing before clustering. Purpose: improve **posterior sample grouping / posterior manifests / posterior analysis** via richer feature representations than any single-feature mode supports today.

**Clustering target reminder:** APS clustering operates on **samples**, not amplicons. The per-sample feature vector is built from quantities that genuinely vary across samples for the same amplicon (so they resolve developmental-stage axes). Quantities that are constant across samples for a given amplicon (or trivially redundant with another included feature) are excluded — see DECISIONS.md for the rejection rationale.

**Per-amplicon features (the 5 morphology dimensions + Layer-1 PCA scores):**

| # | Feature | Description / column | Provenance |
|---|---|---|---|
| 1 | `summit_rcn` | Raw peak RCN within amplicon (per-sample) | Existing column (post-rename — see BRAIN15.28; computed locally if 15.28 not yet landed) |
| 2 | `width` | Width above threshold (`width_above_threshold_bp`) per-sample-per-amplicon | Existing column |
| 3 | `log10area` | log10-transformed area; "excess area" vs RCN-summed area decided by BRAIN15.14 testing | Existing column (form decided by BRAIN15.14) |
| 4 | `asymmetry_ratio` | Per-sample-per-amplicon, from asymmetric triangle fit | Existing column |
| 5 | `shape_score_raw` | Per-sample-per-amplicon `dBIC_flat_vs_tri` value | Existing column ([refinement.py:415](onionskin_core/refinement.py#L415)) |
| 6+ | Layer-1 amplicon-level shape PCA scores (PC1..PCX) | New computation; X controlled by `--aps-num-amp-pc` (default 2) | New (this BRAIN entry) |

**Sample-level features (3 scalars + Layer-2 PCA scores):**

| # | Feature | Description |
|---|---|---|
| 1 | sample-level summit-APS | sum or average of per-amplicon `summit_rcn` |
| 2 | sample-level width-APS | sum or average of per-amplicon `width` |
| 3 | sample-level log10-area-APS | sum or average of per-amplicon `log10area` |
| 4+ | Layer-2 sample-level shape PCA scores (PC1..PCX) | New computation; X controlled by `--aps-num-sample-pc` (default 2). Variant 2a or 2b — see "Three PCA layers" below |

**Three PCA layers (orthogonal, composable):**

- **Layer 1 — amplicon-level shape PCA.** For each amplicon separately, fit PCA on the (n_samples × n_bins_in_amplicon) shape sub-matrix; take top X PCs per amplicon. Per-sample feature contribution: X scores per amplicon. Total Layer-1 dim per sample = n_amplicons × X. Toggle: `--aps-num-amp-pc <int>` (default 2).
- **Layer 2 — sample-level shape PCA**, two variants:
  - **2a (`sample_shape_pca_raw`):** PCA across the samples × all-bins-concatenated matrix. Caveat: bins of wide amplicons dominate by sheer dimension count.
  - **2b (`sample_shape_pca_reduced`):** PCA across the Layer-1-reduced amplicon scores matrix (samples × (n_amplicons × Layer1_X)). Each amplicon contributes equally regardless of bin count — pre-equalized.
  - Both produce M (samples) × X output. Toggle: `--aps-num-sample-pc <int>` (default 2).
- **Layer 3 — matrix-level PCA.** Generic post-feature-construction projection: whatever feature matrix `--aps-feature` produced (M × any-N), PCA-project to M × X before clustering. Independent toggle that composes with any Layer 1/Layer 2 / metrics mode.

The three layers compose. Example: `--aps-feature shape_pca --aps-pca-matrix --aps-final-pc 2` does Layer 1 (per-amplicon PCA on shape) → Layer 3 (PCA on the resulting matrix). Or `--aps-feature metrics --aps-pca-matrix` does the all-metrics composite then Layer-3 PCA-projects it before clustering.

**New `--aps-feature` modes (additive — existing single-feature modes stay):**

| Mode | Composition | Per-sample dim | Layer dependencies |
|---|---|---|---|
| `shape_pca` | Layer 1 amplicon PCA on per-bin shape | n_amplicons × X | Layer 1 (`--aps-num-amp-pc`) |
| `log2shape_pca` | Layer 1 on log2-transformed shape | n_amplicons × X | Layer 1 (`--aps-num-amp-pc`) |
| `sample_shape_pca_raw` | Layer 2a (PCA across all-bins-concatenated) | X | Layer 2 (`--aps-num-sample-pc`) |
| `sample_shape_pca_reduced` | Layer 2b (PCA across Layer-1-reduced amplicon scores) | X | Layer 1 + Layer 2 |
| `amp_metrics` | The 5 amplicon-level features + Layer-1 amplicon PCA scores | n_amplicons × (5 + X) | Layer 1 (`--aps-num-amp-pc`) |
| `sample_metrics` | The 3 sample-level scalars + Layer-2 PCs | 3 + X | Layer 2 (`--aps-num-sample-pc`) — variant 2a or 2b is itself a sub-toggle (default TBD in SPEC engineering; recommendation 2b for the equalization advantage) |
| `metrics` | `amp_metrics` ∪ `sample_metrics` | n_amplicons × (5 + X_amp) + 3 + X_sample | Layer 1 + Layer 2 |

**Naming convention:** `log2-` prefix for shape per existing convention (`log2shape`, `log2shape_pca`); `log10-` prefix for area per BRAIN15.14 testing convention (`log10area`); summit stays raw per the user-confirmed approach. No `log10shape_*` modes — that's a typo trap; if needed, can add a separate naming-convention decision.

**New flags:**

| Flag | Default | Affects |
|---|---|---|
| `--aps-num-amp-pc <int>` | 2 | Layer 1 (`shape_pca`, `log2shape_pca`, `sample_shape_pca_reduced`, `amp_metrics`, `metrics`) |
| `--aps-num-sample-pc <int>` | 2 | Layer 2 (`sample_shape_pca_raw`, `sample_shape_pca_reduced`, `sample_metrics`, `metrics`) |
| `--aps-pca-matrix` | off | Layer 3 toggle; composable with any `--aps-feature` |
| `--aps-final-pc <int>` | 2 | Layer 3 PC count |

**Optional sub-toggle (for SPEC engineering):** `sample_shape_pca` variant choice (2a `_raw` vs 2b `_reduced`) is exposed as two distinct mode names today, but SPEC engineering may consolidate to a single mode + a `--aps-sample-pca-strategy` flag.

**Soft dependencies on other Phase 15 BRAIN entries:**

- **BRAIN15.14** (`--aps-area-excess-floor` analytical testing) decides what `area` means for `log10area`. Whichever wins (excess vs RCN-based) is what BRAIN15.27 consumes.
- **BRAIN15.28** (`summit_excess` → `summit_rcn` rename + RCN-as-default migration). If 15.28 lands first, BRAIN15.27 reads the renamed column directly. If 15.27 lands first, it computes raw RCN locally pending the rename. Either ordering works.
- **BRAIN15.7** (amplicon reliability / flat-sample detection) and **BRAIN15.19** (clustering defaults finalization) — BRAIN15.27 produces inputs to BRAIN15.19's experimental matrix; the two interlock during testing. BRAIN15.7's reliability filtering should run before BRAIN15.27's feature computation so flat samples don't distort PCA.

**Implementation staging suggestion (for SPEC engineering, not BRAIN-stage):**

Phase 15 doesn't need to land all 7 new modes simultaneously. A reasonable staging:

1. **First wave:** `metrics` (all 5 amplicon-level + 3 sample-level features + Layer-1 + Layer-2 PCs) + `--aps-pca-matrix` (Layer 3). Smallest implementation surface, biggest immediate gain.
2. **Second wave:** `shape_pca` / `log2shape_pca` (Layer 1 standalone — for users who want only amplicon-level shape PCA without the metrics).
3. **Third wave:** `sample_shape_pca_raw` / `sample_shape_pca_reduced` / `amp_metrics` / `sample_metrics` (carve-outs for testing isolation).

But staging is a SPEC-engineering decision; BRAIN15.27 spec's the full design and lets SPEC engineering decide what to land in which cycle.

**Future-separability hooks (per user note about possibly separating feature variants later):** The new feature modes' implementation should expose the feature list as a configurable enumeration in `onionskin_core/aps.py`, so adding/removing individual features later — or testing per-amplicon-only vs sample-scalars-included variants — is a one-flag change rather than a refactor.

**Brainstorming-stage notes for follow-on enrichment:**

- Default for X (PCs per amplicon, PCs per sample, PCs final) of 2 is a recommendation, not a decision. Brainstorming/SPEC engineering should decide whether default X=1 (more conservative, PC1 only — captures dominant axis without PC2 noise) or X=2 (PC1 + PC2). Test on real data during BRAIN15.19 clustering experiments.
- `sample_shape_pca_reduced` (variant 2b) is recommended over `_raw` (variant 2a) by default because of the bin-count-equalization advantage user-noted. Confirm during testing.
- Cluster validation: BRAIN15.27 mode results should be evaluated against the reliability-filtered amplicon set from BRAIN15.7 and the area-form-decision from BRAIN15.14; the combined experimental matrix lives in BRAIN15.19.

**Future candidate (deferred to APS_SOUP.md):** Amplicon-level clustering — clustering AMPLICONS by their cross-sample feature profiles (complementary to the sample-clustering target of BRAIN15.27). User-noted as a future-phase candidate.

#### Round 6 update (2026-04-28) — clustering-concept disambiguation + APS experiment harness folded in (per Codex audit)

**Clustering-concept disambiguation (cross-reference to BRAIN15.19 Round 6 update):** BRAIN15.27 is **strictly about APS sample posterior clustering** — the sample-level posterior-grouping concept implemented at HMM step-14 `aps/` (`onionskin_core/hmm_ported_analyses.py:149-184`), which produces `aps_clusters.tsv` and drives posterior sample groupings / posterior manifests. **It is NOT about HMM trajectory/fork clustering** (HMM step-16 → 17 post-renumbering, `trajectory_feature_matrix.tsv` / `trajectory_clusters.tsv`, `onionskin_core/hmm_ported_analyses.py:380-439`), which is a separate HMM-specific concept. The composite multi-feature APS modes (composite morphology + 3-layer PCA, `--aps-feature shape_pca` etc.) feed the APS feature matrix; BRAIN15.27's clustering is sample-level posterior grouping using that matrix. SPEC engineering must keep these two clustering concepts distinctly named in priority titles, output paths, and help-string text.

**APS experiment harness (`scripts/aps_cluster_experiments.py`) folded in as a Phase 15 BRAIN15.27 + BRAIN15.19 deliverable:** the existing experiment harness is currently growth-only (`scripts/aps_cluster_experiments.py:26-45` enumerates only the older `--aps-feature` modes) and uses `--pipelines growth` hard-coded. Phase 15 work expands it:

- **Make the harness pipeline-selectable** — accept `--pipelines growth|rms|hmm` (or any combination) instead of hard-coded growth-only.
- **Include all new feature modes** from BRAIN15.27 — `metrics`, `amp_metrics`, `sample_metrics`, `shape_pca`, `log2shape_pca`, `sample_shape_pca_raw`, `sample_shape_pca_reduced`. Plus the existing modes (`area`, `summit`, `width`, `shape`, `log2*`, `log10*`).
- **Honor the BRAIN15.21 + BRAIN15.31 cross-pipeline-rewiring scope** — the harness should respect `--aps-pca-matrix` (Layer 3 toggle), `--aps-num-amp-pc`, `--aps-num-sample-pc`, `--aps-final-pc`.
- **Outputs go to `dev/runs/<run_name>/`** by default per the dev-rule in `CLAUDE.md` (no outputs at repo root).
- **Cross-reference BRAIN15.19's experimental matrix:** the harness IS BRAIN15.19's primary tool for evaluating which clustering defaults work best across pipelines + feature modes + datasets.

This is a concrete script-level deliverable that prevents BRAIN15.27's "implement the new feature modes" + BRAIN15.19's "run the optimization experiments that were started but not finished" abstract tasks from missing the actual harness file at SPEC engineering time.

### BRAIN15.28 — `summit_excess` → `summit_rcn` rename; migrate APS to raw-RCN as default; preserve excess + floor as opt-in for sample-level aggregation

**Source:** Phase 15 transfer-stage user directive (chat 2026-04-28). No SOUP source; sibling to BRAIN15.27 (which depends on this rename to read the canonical column name); logged in `PHASE15_FEEDBACK.md` Round 2 follow-up. Decision rationale documented in `multi-agent/project_context/DECISIONS.md` under the 2026-04-28 APS feature mode entry (excess-vs-RCN design decision section).

**Goal:** Migrate APS feature computation away from the "excess" framing (`max(peak_rcn - 1.0, 0.0)` with floor, or `peak_rcn - 1.0` without floor) back to using **raw RCN values** as the default. Rename the existing `peak_rcn` column to `summit_rcn` for naming consistency with the new `summit_*`-style feature names. Preserve the floor-excess behavior as an opt-in for sample-level APS aggregation reasoning where an individual amplicon's contribution-to-sample-APS framing benefits from the floor semantics.

**Background — why "excess" was introduced and why we're migrating away:**

The current code uses `summit_excess` (and `area_excess`, `mean_excess`) computed in [`onionskin_core/aps.py:_locus_metrics`](onionskin_core/aps.py#L163) as `max(peak_rcn - 1.0, 0.0)` with the `--aps-area-excess-floor` flag controlling whether the floor is applied. The "excess" framing was introduced by an agent during earlier development; the user notes (Phase 15 FEEDBACK chat 2026-04-28):

> *"At one point, an agent decided to go the 'excess' route. I questioned why RCN was not enough as is (it is the same as 'fold change'), but the agent thought the excess route made more sense. I just went with it. But honestly, it seems like a pointless move. The profile is in the shape of 'fold change'. So RCN=1 is no difference and becomes 0 with log. The excess computation without the floor is an arbitrary subtraction of a constant, and with the floor I believe it damages information we have about the amplicon (just arbitrarily setting things to 0 or 1)."*

Specifically:
- **Without floor:** `summit_excess = peak_rcn - 1.0` is a translation by a constant. Any clustering / distance computation invariant under translation is unaffected; any computation that isn't invariant (e.g., log scaling, ratio comparisons) is silently shifted to a different reference frame.
- **With floor:** `summit_excess = max(peak_rcn - 1.0, 0.0)` actively destroys information — every below-background sample at a locus gets the same value (0), regardless of how far below background the actual signal was. For sample clustering, this collapses developmental-stage variation in the lower-RCN region.

The user did note that **the floor behavior makes some sense for sample-level APS aggregation**: when computing how much each amplicon contributes to a sample-level APS score, treating below-background samples as "0 contribution from that amplicon" is more interpretable than letting them contribute negative values. So we don't fully retire the excess+floor behavior; we keep it available as an opt-in for sample-level aggregation specifically.

**Required changes:**

1. **Column rename:** `peak_rcn` → `summit_rcn` in [`_locus_metrics()`](onionskin_core/aps.py#L163), `compute_aps_tables()`, and any downstream caller. The renamed column carries the **raw peak RCN value** (no excess subtraction, no floor).
2. **Deprecate `summit_excess` as a per-locus column:** remove `summit_excess` from `_locus_metrics`'s output dict. The current behavior of `summit_excess = max(peak_rcn - 1.0, 0.0)` becomes a derived value computed at sample-level aggregation time when the user opts into the floor semantics.
3. **Behavior change for `--aps-feature summit`:** under the rename, `--aps-feature summit` reads the **raw `summit_rcn`** column instead of the floored `summit_excess`. This is a clustering-output behavior change. Document in CHANGELOG; flag as "v0.X.YY default flip from excess-floored summit to raw-RCN summit" so users can validate against prior runs.
4. **Sample-level APS aggregation flag:** add `--aps-sample-aggregation-floor on|off` (or similar; name TBD) to control whether sample-level APS aggregates apply the `max(x-1, 0)` floor at sample-summing time. Default: TBD by SPEC engineering. Initial recommendation: **default on** (preserves current sample-level APS interpretability) but expose the off path so testing can validate.
5. **Same pattern for `mean_excess`:** rename to `mean_rcn` (raw mean within amplicon); deprecate `mean_excess` as a per-locus column; expose floor semantics at sample-aggregation time.
6. **Area is handled by BRAIN15.14, not here:** BRAIN15.14 already tests `--aps-area-excess-floor on|off` and the post-sum-floor variant. BRAIN15.28 does NOT touch `area_excess` — the area-form decision is BRAIN15.14's. Once BRAIN15.14 settles, BRAIN15.27's `log10area` feature reads whatever the area decision selected.
7. **Help-text + docs updates:** flag rename in `onionskin.py:build_parser()`; update `PIPELINE_SPEC.md` and `ONIONSKIN_FULL_HANDOFF.md` references to `summit_excess` / `mean_excess` / `peak_rcn` columns.

**Backward compatibility:**

- Column-rename callers (downstream consumers of `peak_rcn` / `summit_excess` / `mean_excess`): each call site needs auditing during implementation. Deprecation period: TBD by SPEC engineering. Recommendation: **immediate rename with no deprecation period**, since the project is pre-1.0 and the user has explicitly stated the migration intent.
- The `--aps-feature summit` behavior change: flag in CHANGELOG; if needed, add `--aps-feature summit-excess-legacy` for users who explicitly want the prior behavior. Recommendation: **don't add a legacy flag** — let users pin to a prior version if they need the old behavior.

**Soft dependencies / interactions:**

- **BRAIN15.27** consumes `summit_rcn` directly. Either ordering of 15.27 and 15.28 works (15.27 computes locally if 15.28 not yet landed); cleanest is 15.28 lands first.
- **BRAIN15.14** decides area form; this entry leaves area alone.
- **`--aps-area-excess-floor` flag** (Phase 14 Supplemental work): retained for area; not affected by the summit rename.

**Test expectations:**

- Existing tests that check column names need updates: any test reading `peak_rcn`, `summit_excess`, or `mean_excess` columns from APS output gets updated to read the renamed columns.
- Existing tests that check APS clustering behavior under `--aps-feature summit` need their baseline regenerated (the clustering result will differ — raw RCN vs floored excess).
- New test: verify `summit_rcn` raw values are emitted correctly for both above-background and below-background amplicons.

**Why a separate BRAIN entry from 15.27:** The rename + behavior change has surface area beyond the new feature mode (touches existing `--aps-feature summit` semantics, downstream callers, tests, docs). Splitting it out keeps BRAIN15.27 focused on the new feature design and lets BRAIN15.28 own its own SPEC priority + audit cycle.

#### Round 6 update (2026-04-28) — name HMM-side `--aps-feature summit` resolver explicitly (per Codex audit)

The Codex Agent 2 brainstorming-stage code/tracking audit (2026-04-28) found that the HMM port has its own LOCAL `--aps-feature` resolver that maps `summit` → `summit_excess` independently of the shared APS module. Captured here as a concrete implementation target so the HMM-side rename doesn't lag behind the cross-pipeline rename:

- **`onionskin_core/hmm_ported_analyses.py:149-151`** — currently maps `--aps-feature summit` → `summit_excess` in the HMM-side feature resolver. **This is in addition to the shared APS module's resolver.** The rename in BRAIN15.28 must touch BOTH: the shared APS module resolver AND the HMM-side `hmm_ported_analyses.py:149-151` resolver. Otherwise HMM continues consuming `summit_excess` after the cross-pipeline rename, producing inconsistent output across pipelines.
- **Implication for BRAIN15.28's deliverables list:** add an explicit step "**Audit and update the HMM-side feature resolver in `onionskin_core/hmm_ported_analyses.py:149-151` alongside the shared APS module change.** Verify no other pipeline-side resolver lags behind via grep for `summit_excess` / `peak_rcn` / `mean_excess` after the rename lands."
- **Cross-reference to BRAIN15.20 (SAPS):** SAPS will introduce its own state-path-based feature resolution. Phase 15 should harmonize SAPS feature naming with the post-rename APS schema (use `summit_rcn` etc., not `summit_excess`), so SAPS doesn't ship with the legacy naming.

### BRAIN15.29 — Summit-then-timing-then-updated-summit-then-updated-timing convergence (one-pass; promotes IBM-C6C with richer design)

**Source:** Phase 15 FEEDBACK Q12 user directive (2026-04-28) promoting **IBM-C6C** (Stage-activity fold-change using refined summit; `timing.py` refactor) into Phase 15 scope with substantial new design detail. The IBM file's two-line scope is the seed; the user's Q12 answer in `PHASE15_FEEDBACK.md` adds the four-step convergence design + the rationale for confining summit estimation to active stages. No SOUP source; logged in `PHASE15_FEEDBACK.md` Round 2 follow-up under "User-introduced BRAIN entries" per the same convention as BRAIN15.24, BRAIN15.25, BRAIN15.26, BRAIN15.27, BRAIN15.28.

**Note on IBM-C6C resolution:** This BRAIN entry promotes IBM-C6C from `tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` into Phase 15 scope. APS_SOUP.md's "Items requiring more investigation" entry for IBM-C6C should be updated to point at BRAIN15.29 as the promoted home.

**Goal:** Implement a **one-pass** summit ↔ timing convergence pattern that produces accurate summit positions confined to amplicon-active stages, then re-runs timing using those refined summit positions. This is **NOT** an EM-iterative convergence (per user explicit direction); it's a single pass through 4 ordered steps.

**Why this matters (the circular-dependency problem the user surfaced):**

The existing summit-estimation pipeline and the existing timing pipeline have a circular dependency:

- Summit estimation works best when confined to stages where the origin is actively firing (the "early" stages for a given amplicon, before elongation interference flattens the summit profile).
- Timing analyses estimate WHICH stages are active for each amplicon — but timing's calculations depend on having a summit position.
- So timing depends on summit, and the BEST summit depends on knowing which stages are active (which is what timing estimates).

User-quoted explanation (Phase 15 FEEDBACK Q12, 2026-04-28):

> *"This is a broader goal to consistently use the same summit. Final summit refinement and selection should happen after multistage unification. That will mean we have a final set of amplicon candidates. The summit refinement should use information from across all stages to get the best estimate. This is why we want to confine summit estimation to only stages there is origin firing activity — the 'early' stages relative to a given amplicon's activity / timing. The later stages include only elongation, not replication initiation, so the summit region is expected to become flatter like a table top rather than pointy like a peak. It is most pointy when the origin is actively firing, and the tip of the point is our best guess of where the origin is. The 'timing' analyses get a little screwed up because we are trying to estimate stages of activity using the summit, but we want to estimate the summit using information about timing: what stages the origins are firing in. So it creates a circular problem."*

**The four-step one-pass fix (user-specified design):**

1. **Best initial summit estimate** (post-multistage-unification) — use the existing summit refinement pipeline (BRAIN15.8 sliding-offset refinement, parabola fit, etc.) on the multistage-unified final amplicon set. This produces a "best summit estimator position" without yet using timing information.
2. **Estimate timing** ("timing" step) — run the existing timing analysis using the step-1 summit positions as input. This produces per-amplicon active-stage estimates.
3. **Refine summit estimators based on active stages** ("active_stage_summit_refinement" step) — re-run summit estimation, but **confine the input data to the stages flagged as active by step 2**. Stages flagged as elongation-only (post-firing) are excluded from the summit fit. This produces refined summit positions that aren't biased by elongation-flattened post-firing stages.
4. **Update timing info** ("updated_timing" step) — re-run timing using the step-3 refined summit positions. This produces the final timing estimates.

**Stop after step 4 — no further iteration.** Per user explicit direction:

> *"This is similar to the 'summit estimates ↔ timing' convergence idea, but here I am not suggesting any further iterations. Best summit → timing → update summit → update timing. Done. The changes to timing results in the timing update are likely to be marginal for most amplicons — presumably even a coarse estimate of the summit position would work. But the updates to summit positions could be quite meaningful."*

This explicitly differentiates BRAIN15.29 from IBM-C6E (Iterative summit ↔ timing convergence — EM-like) which IS an EM-iterative pattern and is **deferred** per APS_SOUP.md.

**Required outcome:**

- Onionskin emits 4 sets of summit + timing artifacts under the multistage-unified amplicon set:
  - Step-1 outputs: initial-summit estimates (current summit refinement pipeline output, post-multistage-unification).
  - Step-2 outputs: initial-timing estimates (current timing pipeline output, using step-1 summits).
  - Step-3 outputs: active-stage-refined summit estimates (NEW; uses step-2 active-stage flags to confine the summit fit input data).
  - Step-4 outputs: final timing estimates (uses step-3 refined summits).
- The "final" summit positions for any downstream consumer (APS computation, posterior manifest, plots, notebooks) come from step 3.
- The "final" timing estimates come from step 4.
- Step 1 + Step 2 outputs are preserved for transparency / diagnostic comparison, not as the canonical results.
- Output schemas for each step are explicit (TBD during SPEC engineering — likely two new step directories or a sub-step structure).

**Implementation scope:**

1. **New summit-refinement code path: "active_stage_summit_refinement"** — operates on the multistage-unified amplicon set + active-stage flags from step 2. The code probably refactors / extends the existing summit-refinement modules (`onionskin_core/refinement.py`, `onionskin_core/hmm_summits.py`, etc.) to take an "active stages mask" parameter that confines which stages contribute to the summit fit.
2. **Timing module re-run support** — the timing module needs to be able to run twice (step 2 and step 4) with different summit inputs. Likely a parameter to point at either step-1 or step-3 summit positions; output goes to a different sub-directory. May or may not require a `timing.py` refactor (the IBM-C6C scope of "non-trivial refactor of timing.py to recompute stage-activity fold-change using refined summit positions" enters here — depends on whether the existing timing module is already clean enough to re-run with different summit inputs, or whether structural changes are needed).
3. **Pipeline orchestration** — controller-level wiring to run the 4 steps in order, plumbing intermediate outputs to subsequent steps.
4. **Output schema decisions** — decide whether step-1 / step-3 are emitted as separate files (`<sample>_summits_step1.tsv`, `<sample>_summits_step3.tsv`) or as a single file with a step-marker column. Same for timing. Likely separate files for clarity.
5. **CLI surface** — likely no new flag; this becomes the default summit + timing flow when multi-stage data is available. May want a `--no-active-stage-summit-refinement` opt-out for users who want the simpler step-1 summits as the canonical output (e.g., for backward-comparability runs).
6. **Tests** — new tests for the active-stage-summit-refinement step; baseline regeneration for tests that check summit positions (the canonical summits will shift slightly for many amplicons).
7. **Plots / notebooks** — diagnostic notebooks should show the step-1 vs step-3 summit positions side-by-side for a few example amplicons; plots should default to showing step-3 (final) but allow toggling to step-1.

**Soft dependencies on other Phase 15 BRAIN entries:**

- **BRAIN15.18** (HMM shape-score wiring + multistage unification + state-path evolution) — BRAIN15.29 operates on the multistage-unified amplicon set; BRAIN15.18 must produce that set first. Hard dependency.
- **BRAIN15.8** (HMM `peak_rcn_stage` + sliding-offset refinement) — BRAIN15.8's per-amplicon-per-sample-per-stage step-5 outputs are the natural input for step-1 summit refinement (and for step-3's confined-to-active-stages variant). Likely hard dependency.
- **BRAIN15.6** (HMM parallel child pipeline) — provides the per-sample step-5 outputs that BRAIN15.8 depends on; transitive hard dependency.
- **BRAIN15.24** (`peak_rcn_stage` audit) — BRAIN15.29's "active stages" concept is closely related to `peak_rcn_stage`'s definition. The audit should resolve any naming / semantics ambiguity before BRAIN15.29 implementation begins. Hard dependency on the audit clearing.

**Pipeline-applicability:**

The four-step pattern naturally applies to **all three pipelines** (HMM, growth, RMS), since each pipeline has its own summit-estimation + timing flow. But the user's framing emphasizes HMM (which has the richest active-stage information from state-path evolution + sliding-offset refinement). Initial implementation: HMM. Cross-pipeline extension: deferred follow-on if HMM proves the pattern works.

**Diagnostic value and testing:**

The user noted that the timing update (step 4 vs step 2) is *"likely to be marginal for most amplicons"*, but the summit updates (step 3 vs step 1) *"could be quite meaningful."* So testing should focus on:

- How much do summit positions shift between step 1 and step 3? Distribution analysis across the test datasets.
- Where shifts are large, is the step-3 position the visually-correct one (per ground truth where available)?
- How much does the timing update (step 4 vs step 2) actually change downstream APS / posterior groupings? If marginal, the cost of implementing step 4 vs cost of skipping it (using step-2 timing as canonical despite step-3 summit being canonical) is worth a SPEC-engineering decision.

**Why this is in Phase 15 rather than deferred:**

- The HMM-completeness focus of Phase 15 + BRAIN15.6 + BRAIN15.8 + BRAIN15.18 produce the prerequisites for this work to land cleanly.
- The summit ↔ timing circularity is a long-standing problem (originally flagged as ROADMAP Priority 5.0.2 in Phase 4, deferred post-HMM as IBM-C6C); Phase 15 IS post-HMM in the relevant sense.
- The user explicitly promoted this from "deferred" to in-scope per Q12 (2026-04-28).

**What's deferred (NOT in BRAIN15.29):**

- **IBM-C6E (Iterative summit ↔ timing EM-like convergence)** — explicitly deferred per user note that BRAIN15.29 is one-pass, NOT EM-iterative. IBM-C6E stays in APS_SOUP.md.
- **IBM-C6D (Timing-guided active-stage weights for summit estimation)** — Round 4 update: per Q20 user direction in Phase 15 FEEDBACK Round 3, IBM-C6D is **promoted** because it relates to the ODW concept. Folded into BRAIN15.29's body as part of the active-stage-weighting design space. *"IBM-C6D ... This sounds related to the Origin Detection Window (ODW) concept, so yes we should explore this and integrate good ideas surrounding this theme from this section or any section it arises in."* (User Q20 IBM bundle answer.)

### BRAIN15.30 — HMM sequence filtering CLI flags — **RESOLVED (already implemented; closes as N/A in Phase 15)**

**Source:** Round 3 Brainstorming-stage aggregation pass proposal in `PHASE15_FEEDBACK.md` Round 3 section, originating from `tracking/BRAINSTORM.md [2026-04-09] HMM sequence filtering — CLI flags` and `tracking/BRAINSTORM.md [2026-04-11] Shared filtering semantics across pipelines`.

**Status:** **RESOLVED.** Round 4 code audit (2026-04-28) confirmed all three flags are already implemented and wired through the HMM engine. Per user Q20 answer: *"BRAIN15.30 - this is supposed to already be implemented. The --chromosomes flag has always worked on the HMM pipeline, and `--min-seq-length` + `--min-bin-count-per-seq` were originally ONLY in the HMM pipeline and had to be ported out to generalize to all pipelines, if I recall correctly. At minimum, they were developed during HMM pipeline development, and for the HMM pipeline, but may have always applied to all 3 on purpose."*

**Code audit verification (2026-04-28):**

- `--chromosomes` passthrough: `chromosomes` parameter is plumbed into HMM engine at [onionskin_core/engines/hmm_engine.py:121](onionskin_core/engines/hmm_engine.py#L121) and consumed at line 128 + 1000.
- `--min-seq-length`: `min_seq_length_bp` parameter at [onionskin_core/engines/hmm_engine.py:122](onionskin_core/engines/hmm_engine.py#L122), default 50_000 at line 1001.
- `--min-bin-count-per-seq`: `min_bin_count_per_seq` parameter at [onionskin_core/engines/hmm_engine.py:123](onionskin_core/engines/hmm_engine.py#L123), default 10 at line 1002.

**Implication for `tracking/BRAINSTORM.md [2026-04-09] HMM sequence filtering — CLI flags`:** the entry is stale — the design proposal it captures has already been implemented. This entry exists as a closeout marker; the [2026-04-09] tracking entry should be marked DONE during a brainstorming-stage cleanup of `tracking/BRAINSTORM.md`. User Q20 hint: *"I am starting to see the pattern that BRAINSTORM.md may be stale and need an audit and clean up."* — separate cleanup task; not Phase 15 priority.

**No further Phase 15 work needed for this BRAIN entry.**

### BRAIN15.31 — `--peak-summary` extension to RMS + HMM + max-projection variants (raw / normalized / smoothed)

**Source:** Phase 15 FEEDBACK Q20 user directive (2026-04-28) approving the proposed Round 3 BRAIN15.31; substantive Phase-15-inclusion direction in `tracking/KNOWN_ISSUES.md [ISSUE:2026-04-28:3] Max projection RCN profiles`. User-quoted Q20 answer: *"Yes to BRAIN15.31 - I added [ISSUE:2026-04-28:3] to known issues today and intended for it to be part of Phase 15. Yes we should extend this out to all 3 pipelines. Whatever the issue said in KNOWN_ISSUES is fodder for including in Phase 15."* No SOUP source by design — added at user direction during brainstorming stage (same convention as BRAIN15.24/25/26/27/28/29).

**Goal:** Extend the existing `--peak-summary` flag (currently growth-only, per the flag's own help-string note *"Currently applied only by the growth pipeline; planned for RMS and HMM in a future phase."*) to RMS and HMM. Add raw / normalized / smoothed variants when `--peak-summary max` is chosen, plus possibly an `all` option. Phase 15 IS the future phase that delivers this work.

**Current state (verified Round 4 audit, 2026-04-28):**

- `--peak-summary {quantile,topk,max}` exists at [onionskin_core/engines/growth_model_engine.py:1127](onionskin_core/engines/growth_model_engine.py#L1127), default `quantile`.
- Help-string in `onionskin.py` already declares the planned RMS+HMM extension: *"Currently applied only by the growth pipeline; planned for RMS and HMM in a future phase."*
- Pipeline-specific override `--growth-peak-summary` already exists.

**Verbatim user content from `tracking/KNOWN_ISSUES.md [ISSUE:2026-04-28:3]`:**

> *"This is in reference to all the peak summary stuff originally from the growth pipeline, now universal. [...] The original intention was just max projection. This may already be accomplished with the 'max' option in --peak-summary. The original goal of max projection was as follows:*
> *- For each bin across the genome, use the max RCN value seen across all stages.*
> *- It would be analogous to 'max projection' for z-stacks in microscopy.*
>
> *Although 'max' is an option for --peak-summary and this might already be available, the following additions should be considered:*
> *- raw max projection: likely what `--peak-summary max` already does*
> *- normalized max projection: following the max projection procedure, perform chromosome-specific median normalization to ensure the results are interpretable.*
> *- smoothed max projection: following normalization, apply the median- or trimmed-mean- smoothing procedure performed on RCN profiles earlier in the pipeline to wrangle outlier bins.*
>
> *The consideration should be to include all 3 outputs for the user to view when `--peak-summary max` is chosen. A further consideration would be to do the normalization and smoothing operations on `--peak-summary quantile` and `--peak-summary topk` as well. Another consideration still would be to add an 'all' option to `--peak-summary` for `--peak-summary all` which would be analogous to `--peak-summary quantile,topk,max` and output all outputs for each option.*
>
> *A question is whether these operations happen only on amplicon intervals or all bins across the genome, and if not the latter, what is done for those?"*

**Phase 15 deliverables:**

1. **Extend `--peak-summary` to RMS and HMM pipelines.** Apply the same logic that growth currently has, in pipeline-specific output directories (e.g., `01-hmm/<step>-peak-summary/` and `03-rcn-mean-shift/<step>-peak-summary/`). Pipeline-specific overrides: `--rms-peak-summary` and `--hmm-peak-summary` (analogous to `--growth-peak-summary`).
2. **Add raw / normalized / smoothed variants for `--peak-summary max`** — three output files when max is chosen:
   - Raw max projection: `<prefix>_max_projection.bedGraph` (likely existing behavior).
   - Normalized max projection: `<prefix>_max_projection.chrom_median_norm.bedGraph` — chromosome-specific median-normalized so the result is interpretable as fold-change.
   - Smoothed max projection: `<prefix>_max_projection.smoothed.bedGraph` — median- or trimmed-mean-smoothed (per existing pipeline smoothing convention) to wrangle outlier bins.
3. **Consider applying normalization + smoothing to quantile + topk** — possibly emit normalized + smoothed variants for all three modes (quantile, topk, max), not just max.
4. **Add `--peak-summary all` option** — equivalent to running quantile + topk + max with all variants. Useful for comparison and selection.
5. **Decide whether operations apply only to amplicon intervals or all bins across the genome** — open SPEC-engineering question. User-flagged: *"A question is whether these operations happen only on amplicon intervals or all bins across the genome, and if not the latter, what is done for those?"* Default: emit both (whole-genome track AND per-amplicon summary).

**Output schemas:**

Each pipeline emits its own peak-summary directory under its step-numbered output:

- Growth: `02-growth-model/07-signal-tracks/` (existing) — extend to include normalized + smoothed variants.
- RMS: new step or sub-dir under `03-rcn-mean-shift/` — e.g., `03-rcn-mean-shift/<step>-peak-summary/`.
- HMM: new step or sub-dir under `01-hmm/` — e.g., `01-hmm/<step>-peak-summary/`.

Step numbering decisions defer to SPEC engineering. The step-less plots/notebooks structural change (per BRAIN15.17) does NOT apply to peak-summary outputs — those are pipeline-derived analytical outputs that belong in step-numbered dirs per BRAIN15.2 design rule 1.

**Soft dependencies:**

- **BRAIN15.6** (HMM parallel child pipeline) — provides per-sample step-5 outputs that HMM peak-summary can consume.
- **BRAIN15.21** (shared gap mask + missingness diagnostic to pre-pipeline) — peak-summary output should ideally respect the shared gap mask to avoid spurious gap-bin contributions.
- **BRAIN15.5** (HMM completeness matrix) — peak-summary is one of the gap-audit-table rows ("Signal-tracks / peak-summary" — `02-growth-model/07-signal-tracks/`).

**Implementation staging suggestion:**

Phase 15 doesn't need to land all variants at once. Reasonable staging:

1. **First:** extend `--peak-summary` (existing 3 modes) to RMS + HMM. Largest user-visible win; smallest implementation surface.
2. **Second:** add raw / normalized / smoothed variants for `--peak-summary max`.
3. **Third:** extend variants to quantile + topk modes if testing shows value.
4. **Fourth:** add `--peak-summary all`.

But staging is SPEC-engineering's call.

### BRAIN15.32 — HMM fork age tracking visualization — **RESOLVED (already implemented; closes as N/A in Phase 15) + enhancement audit recommended**

**Source:** Round 3 Brainstorming-stage aggregation pass proposal in `PHASE15_FEEDBACK.md` Round 3 section, originating from `tracking/BRAINSTORM.md [2026-04-09] Fork age tracking — new visualization concept`.

**Status:** **RESOLVED.** Round 4 code audit (2026-04-28) confirmed fork age tracking is implemented in `onionskin_core/hmm_fork_travel.py`. Per user Q20 answer: *"BRAIN15.32 may also already be implemented. I am starting to see the pattern that BRAINSTORM.md may be stale and need an audit and clean up. Nonetheless, BRAIN15.32 is a very important topic - it just might already be done. Certainly we have some version of it already. So it is just a matter of whether this is proposing something new or a fix to it or making it better. Needs a code audit."*

**Code audit verification (2026-04-28):**

- Fork age metrics emitted: [onionskin_core/hmm_fork_travel.py:12](onionskin_core/hmm_fork_travel.py#L12) — *"`12-fork-travel/fork_age_metrics.tsv`"*
- Per-level emergence stage tracking: [onionskin_core/hmm_fork_travel.py:242](onionskin_core/hmm_fork_travel.py#L242) — `level_emergence_stage`, `ghost_level_flag`
- Module docstring: *"with one row per (amplicon, level, stage)"* — supports the level-vs-age re-indexing the [2026-04-09] design called for.

**Open Phase-15 question — does the implementation match the [2026-04-09] design's "rendering rule"?**

The [2026-04-09] design specified (verbatim): *"Only fork ages present in all (or most) stages can be plotted as continuous trajectories. Younger fork ages (small values of age, e.g., age 0 = newest) may not be present in early stages. Render missing stages as gaps in the line, not errors."*

**Phase 15 audit task (folded into BRAIN15.17 plot-review work):** verify the implementation honors this rendering rule. If gaps in early-stage trajectories are rendered as gaps (correct) vs as zeros or as errors (incorrect), close as DONE. If the rendering is incorrect, that's a small enhancement task within Phase 15 scope.

**Phase 15 audit task (folded into BRAIN15.5 completeness matrix):** verify the fork-age-tracking output schema matches the [2026-04-09] design's filename convention (`{trajectory_id}.fork_age_trajectory.png`). Filename convention drift is a small fix.

**Implication for `tracking/BRAINSTORM.md [2026-04-09] Fork age tracking — new visualization concept`:** mostly DONE; the entry is stale at the design level (the concept is implemented). User-quoted hint about staleness: *"I am starting to see the pattern that BRAINSTORM.md may be stale and need an audit and clean up."* — separate cleanup task; not blocking Phase 15.

**No new BRAIN entry needed.** Audit-driven enhancements (if any) fold into BRAIN15.17 (plot review/parity) or BRAIN15.5 (completeness matrix) rather than living as their own entry.

### BRAIN15.33 — Phase 15 housekeeping bundle (timing prefix harmonization + summit_inspector fix + RMS summit-selector follow-up)

**Source:** Phase 15 FEEDBACK Round 3 user direction (2026-04-28) marking three `tracking/KNOWN_ISSUES.md` items as Phase-15-intended that don't fit cleanly under any other BRAIN entry. No SOUP source by design — added at user direction during brainstorming stage.

**This bundle exists to satisfy the substantive-priorities-v2 rule** by combining three thin-but-related housekeeping items into one BRAIN entry with one audit-implement-reaudit cycle, rather than spawning three separate thin BRAIN entries.

#### Bundled item 1: `--onset-*` vs `--timing-*` prefix harmonization in Timing parser group ([ISSUE:2026-04-25:1])

**Verbatim from `tracking/KNOWN_ISSUES.md [ISSUE:2026-04-25:1]`:** *"Deferred naming cleanup from Phase 14 Supplemental 14-S13. The Timing parser group currently mixes two flag prefixes: `--onset-*` (4 flags: `--onset-rcn-threshold`, `--onset-span-rcn-threshold`, `--onset-method`, `--onset-z-threshold`) and `--timing-*` (2 flags: `--timing-onset-quantile`, `--timing-exclude-loci`) plus `--near-gap-bp`. Q12 deferred prefix harmonization. Exit condition: Future-phase work consolidates these flags under a single prefix or adopts a clearer naming convention, with backwards-compatible aliases if needed."*

**Phase 15 deliverable:** Decide single prefix (`--timing-*` is more conventional given the parser group is the Timing group) or document the two-prefix scheme deliberately. If renaming, add backwards-compat aliases. Connects to BRAIN15.10 (PuffStep synonym work) and BRAIN15.11 (HMM PuffStep synonym audit) only loosely — those are HMM-specific; this is timing-pipeline-specific.

#### Bundled item 2: `summit_inspector.py` ModuleNotFoundError ([ISSUE:2026-04-28:4])

**Verbatim from `tracking/KNOWN_ISSUES.md [ISSUE:2026-04-28:4]`:** Error output: *"ModuleNotFoundError: No module named 'onionskin_core'"* when running `scripts/summit_inspector.py --run-dir ${ONIONDIR} --region II:33000000-34000000 ...`. *"This is an important script to the user. What to do: Reproduce and fix. Considerations: Is the user using anything outdated here?"*

**Phase 15 deliverable:** Fix the module-import path or the script's invocation pattern so `summit_inspector.py` runs cleanly. Likely fix: ensure the script honors the `onionskin_core` package path (PYTHONPATH-aware invocation, or import-shim, or proper invocation via `python -m`). Audit similar tooling in `scripts/` for the same issue.

**Cross-reference:** the script may also need step-numbering updates per BRAIN15.6 (SAPS step insertion) and the step-less plots/notebooks structural change per BRAIN15.17. Bundle the fix-up sweep here.

#### Bundled item 3: `early-parabola-mean` runtime selector follow-up ([ISSUE:2026-04-18:2])

**Verbatim from `tracking/KNOWN_ISSUES.md [ISSUE:2026-04-18:2]`:** *"Active known issue. Near-term follow-up from Priority 13.3 runtime selector work. [...] `early-parabola-mean` is now fully wired through runtime, unified-call outputs, the canonical summit evaluator, and the dedicated rcn diagnostic lane. Validation passed: `make test`, `make toy`, and `RMS_SUMMIT_POLICY=early-parabola-mean SUMMIT_CHROMOSOMES=II bash tests/run_summit_precision_test.sh smoke`. The smoke run emitted `early_parabola_mean` as the actual runtime rcn selector, but the ds2 hires II/9A result was still only `3/10` with signed ORC distance `+1979 bp`. [...] If more selector work happens, it should focus on choosing the informative stage window rather than averaging across mixed stage states."*

**Connection to ODW (BRAIN15.24):** the issue's own recommendation — *"choosing the informative stage window rather than averaging across mixed stage states"* — IS the ODW-confined-input pattern that BRAIN15.8 + BRAIN15.29 implement. This issue is partially superseded by the ODW framework: confining the summit-selector input to ODW stages should give better selector results than averaging across all stages.

**Phase 15 deliverable:** as part of the BRAIN15.8 ODW-confined summit refinement work, validate that the new ODW-confined approach improves on `early-parabola-mean`'s smoke-test results. If it does, this issue closes. If it doesn't, capture the residual gap as a Phase 15 deliverable to land a runtime-feasible selector improvement that DOES improve II/9A on the diagnostic surface.

**Note this item is RMS-side primarily** but per user direction *"in spirit is something we should pay attention to when computing parabola summits in the HMM pipeline."* So Phase 15's HMM summit-refinement work (BRAIN15.8 + BRAIN15.29) should heed the lessons from the RMS selector exploration.

#### Out of scope for this bundle

- `[ISSUE:2026-04-24:2]` (RMS summit parabola pre-smoothing investigation) — user note: *"in spirit is something we should pay attention to when computing parabola summits in the HMM pipeline. When designing it with the Growth pipeline, smoothing was important. And actually, the type of smoothing it does specifically. I recall we tried changing it to median smoothing, and the summit evaluation results got poorer so we reverted it back to mean smoothing or trimmed mean smoothing that it does — check the growth code for this, describe it and compare it to how parabola summits are computed for the HMM pipeline."* This is a smoothing-choice audit relevant to BRAIN15.8 (HMM summit refinement) — folded there as a "parabola-smoothing methodology audit" deliverable, not bundled here.

#### Round 6 update (2026-04-28) — bundled item 4: HMM notebook generator rename + modernize (per Codex audit)

The Codex Agent 2 brainstorming-stage code/tracking audit (2026-04-28) found a fourth small housekeeping item that fits this bundle's "thin Phase-15-marked items not fitting cleanly under other entries" pattern. Adding as a fourth bundled item:

**Bundled item 4: HMM notebook generator function rename + modernization**

- **Current state:** `onionskin_core/hmm_notebooks.py` writes notebooks via a function still called `write_hmm_phase9_notebooks` (called from `onionskin_core/engines/hmm_engine.py:1269-1270`). The function name is a stale relic from Phase 9 development.
- **Phase 15 deliverable:** rename `write_hmm_phase9_notebooks` → `write_hmm_notebooks` (drop the phase number; matches the step-less convention's spirit per `[ISSUE:2026-04-28:1]` + BRAIN15.17 step-less plots/notebooks structural change). Update the callsite at `hmm_engine.py:1269-1270`. The function body itself doesn't necessarily need changes here — the modernization is the rename + cleanup of any stale Phase-9-era hardcoded paths inside the function (cross-reference BRAIN15.17 Round 6 update for the step-numbered-path fixes inside the notebook generator).
- **Cross-reference:** BRAIN15.17 (APS plots/parity) Round 6 update names `hmm_notebooks.py` as the implementation target for the step-less notebooks structural change. This BRAIN15.33 bundled item adds the function-rename housekeeping deliverable on top.
- **Why bundled here:** thin housekeeping item (rename + a few stale-path fixes); doesn't justify its own BRAIN entry; fits BRAIN15.33's "Phase 15 housekeeping bundle" theme.

### BRAIN15.34 — Phase 15 closeout: tracking-file cleanup pass (apply migration-with-provenance convention everywhere)

**Source:** Phase 15 FEEDBACK Round 4 user direction (2026-04-28). User-quoted: *"As part of one of the last things to add to SPEC, we should have a stage where we make sure we clean up all the tracking files. In general, I think all should follow the concept we just laid out for KNOWN_ISSUES. If we port it somewhere else for active development or that puts it in line for active development, then it should be deleted from the tracking file completely (if the whole idea was ported over, otherwise partially deleted with reference to the new location of the piece that was moved), ported over to the new home, and the new home annotated with its provenance (where it used to live and what it used to be called). So at the end of Phase 15, everything that is done in Phase 15 should be cleaned up. And we might as well expand it to try to clean up the files in general so we don't have to keep encountering ideas that have already been implemented."*

No SOUP source by design — added at user direction during Phase 15 brainstorming stage (same convention as BRAIN15.24/25/26/27/28/29/31/33).

**Goal:** As one of the last priorities in the Phase 15 SPEC (or as the very last priority before phase closeout), apply the **KNOWN_ISSUES → new home with provenance annotation** convention (see `multi-agent/AGENT_CONVENTIONS.md § KNOWN_ISSUES.md`, added Round 4 2026-04-28) **across all tracking files**, not just `KNOWN_ISSUES.md`. Plus a general staleness sweep so we stop encountering already-implemented ideas as if they were open.

**Scope of files cleaned up:**

- `multi-agent/tracking/KNOWN_ISSUES.md` — delete entries that landed in Phase 15 implementation; partial-delete entries where only part of the issue landed (with explicit pointer to where the implemented piece moved to).
- `multi-agent/tracking/BRAINSTORM.md` — mark stale entries as DONE with v0.X.YY pointers + brief implementation note (e.g., the 2026-04-09 HMM sequence filtering CLI entry — already implemented per BRAIN15.30 RESOLVED marker; the 2026-04-09 fork age tracking entry — already implemented per BRAIN15.32 RESOLVED marker; etc.). Decide on a per-entry basis: full delete + provenance pointer at the new home, OR keep the entry but mark DONE with implementation note.
- `multi-agent/tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` — close out entries that were promoted into Phase 15 BRAIN entries. IBM items already covered: IBM-C2 (BRAIN15.6) / IBM-C3 (BRAIN15.7) / IBM-C4 (BRAIN15.8) / IBM-C5 (BRAIN15.22) / IBM-C6C (BRAIN15.29) / IBM-C6D (BRAIN15.29 deferred-list) / IBM-C7B (RESOLVED-by-existing + BRAIN15.27) / IBM-C8A (BRAIN15.14) / IBM-C8B (BRAIN15.7) / IBM-C12 (BRAIN15.7) / IBM-C14 (BRAIN15.7) / IBM-C7A (BRAIN15.27 deferred bits + APS_SOUP).
- `ROADMAP.md` — update completion markers for Phase 15 priorities once they ship: `✓ DONE (v0.X.YY)`.

**Cleanup convention (apply uniformly):**

For each entry that's been ported / promoted / implemented:

1. **Identify the new home** — which Phase 15 BRAIN entry, which DECISIONS entry, which ROADMAP marker, which archived phase plan, etc.
2. **Add provenance annotation at the new home** if not already present: e.g., `**Formerly lived in `tracking/BRAINSTORM.md` and was referred to as `[2026-04-09] HMM sequence filtering — CLI flags`.**` (paraphrased per file's natural style). The exact wording can vary; the principle is: future agents grepping for the old name find the new home.
3. **Remove from the source tracking file** — full delete if the whole entry was ported; partial delete with a pointer if only part was ported (e.g., *"The summit-window-shape sub-idea moved to BRAIN15.X; the post-call gap-distance annotation sub-idea remains here"*).
4. **Verify with `git log` history** that the rename history is preserved by `git log --follow` so deletion doesn't break historical traceability.

**Two operational modes:**

- **Phase-15-specific cleanup (mandatory at phase close):** entries that Phase 15 explicitly implemented, ported, promoted, or RESOLVED. The cleanup is the final step before declaring Phase 15 closed. Tied to specific Phase 15 priorities.
- **General staleness cleanup (broader, opportunistic):** entries that were silently implemented in earlier phases but never closed out in their tracking-file home. The Round 4 audit found at least two examples (BRAIN15.30 = `[2026-04-09]` HMM sequence filtering — already in code; BRAIN15.32 = `[2026-04-09]` Fork age tracking — already in code). User-quoted observation: *"I am starting to see the pattern that BRAINSTORM.md may be stale and need an audit and clean up."* This broader cleanup is opportunistic — do as much as time allows during the BRAIN15.34 cycle, but don't block phase-15 closeout on it.

**Deliverables:**

1. **Closeout sweep** of `KNOWN_ISSUES.md` — every issue marked Phase-15-included in the FEEDBACK Round 3 user note (2026-04-28) gets deleted (or partial-deleted with pointer) once the corresponding Phase 15 work ships.
2. **Closeout sweep** of `tracking/BRAINSTORM.md` — every entry that maps to a Phase 15 BRAIN entry gets a RESOLVED note pointing at the BRAIN entry + the v0.X.YY where it landed.
3. **Closeout sweep** of `tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` — every IBM-C* item resolved by Phase 15 gets RESOLVED status + closeout pointer.
4. **General staleness sweep** of `tracking/BRAINSTORM.md` — entries that describe work that landed in earlier phases (Phase 9–14) should also be marked DONE / closed if found. This is opportunistic — flag candidates during the closeout sweep but don't expand scope indefinitely.
5. **`ROADMAP.md` update** — Phase 15 priorities get `✓ DONE (v0.X.YY)` markers in their ROADMAP entries (per existing ROADMAP convention).
6. **Audit-pass verification** — after the cleanup, an auditor agent verifies that no Phase-15-implemented work is still listed as "open" or "active" in tracking files. The Round 4 follow-up FEEDBACK section's inclusion-list cross-references make this auditable mechanically.

**Phase placement:**

- BRAIN15.34 is **the last priority in Phase 15 SPEC** (executes after all other Phase 15 implementation lands).
- It is itself a SPEC priority (not just a wrap-up checklist) because it has substantive deliverables: file edits, provenance annotations, audit-pass verification.
- After BRAIN15.34 closes, Phase 15 itself can close cleanly with no stale references to "open" Phase 15 work in tracking files.

**Cross-reference to AGENT_CONVENTIONS:**

The migration-with-provenance convention codified at `multi-agent/AGENT_CONVENTIONS.md § KNOWN_ISSUES.md` (added Round 4 2026-04-28) is the operational template. BRAIN15.34's cleanup applies that convention systematically. The convention may itself be expanded post-Phase-15 to cover non-KNOWN_ISSUES tracking files (BRAINSTORM, INTENDED-BUT-MISSED) — that expansion can happen as part of BRAIN15.34's execution if useful refinements emerge.

**Why this lives in BRAIN15.34, not in BRAIN15.5 (HMM completeness matrix):**

BRAIN15.5 tracks the HMM-side analysis-family completeness; BRAIN15.34 tracks the *planning-file* hygiene that closes Phase 15 cleanly. Different concerns, different surfaces. BRAIN15.5 closes when HMM gains parity with growth/RMS; BRAIN15.34 closes when planning files no longer reference Phase 15 work as open.

**Soft dependencies:**

- All other Phase 15 BRAIN entries land first. BRAIN15.34 is the last priority in the Phase 15 SPEC's execution order.
- The `KNOWN_ISSUES` migration convention (already codified in AGENT_CONVENTIONS Round 4) is the prerequisite — BRAIN15.34 applies that convention.

**Open question for SPEC engineering (defer to that stage):**

When porting an entry from `tracking/BRAINSTORM.md` whose work landed inline in code (no dedicated BRAIN entry; just shipped), where does the provenance annotation go? Options: (a) annotate in CHANGELOG (already does this for code-level changes); (b) annotate in ROADMAP completion marker; (c) keep a brief RESOLVED note in the original tracking-file entry rather than full-delete. SPEC engineering picks the per-case convention.

---

## Coverage check

The following SOUP IDs are referenced by at least one BRAIN entry's `Source:` field (mechanically auditable):

- SOUP15.1 → BRAIN15.1
- SOUP15.2 → BRAIN15.2
- SOUP15.3 → BRAIN15.3
- SOUP15.4 → BRAIN15.4
- SOUP15.5 → BRAIN15.5 (umbrella) + BRAIN15.8 (carry-over: peak_rcn_stage + sliding-offset) + BRAIN15.13 (carry-over: --hmm-smooth-halfwidth split, RESOLVED) — split documented in FEEDBACK
- SOUP15.6 → BRAIN15.6
- SOUP15.7 → BRAIN15.7 (consolidated with SOUP15.12)
- SOUP15.8 → BRAIN15.12 (item 2: --hmm-0-based) + BRAIN15.17 (item 1: APS plots) + BRAIN15.23 (item 3: indiv_samples/ path) — split documented in FEEDBACK
- SOUP15.9 → BRAIN15.11
- SOUP15.10 → BRAIN15.12
- SOUP15.11 → BRAIN15.14
- SOUP15.12 → BRAIN15.7 (consolidated with SOUP15.7)
- SOUP15.13 → BRAIN15.15
- SOUP15.14 → BRAIN15.18
- SOUP15.15 → BRAIN15.16
- SOUP15.16 → BRAIN15.22
- SOUP15.17 → BRAIN15.9 (consolidated with SOUP15.18)
- SOUP15.18 → BRAIN15.9 (mu-scale + re-audit) + BRAIN15.10 (missing PuffStep synonyms split out)
- SOUP15.19 → BRAIN15.19
- SOUP15.20 → BRAIN15.20
- SOUP15.21 → BRAIN15.21

**All 21 SOUP IDs covered.** Splits and consolidations are documented in `PHASE15_FEEDBACK.md`.

**BRAIN-IDs that have no direct SOUP source (user-approved, see FEEDBACK):**

- BRAIN15.24 — `peak_rcn_stage` term audit (user directive 2026-04-27).
- BRAIN15.25 — HMM two-pass background-region-anchored chromosome-specific renormalization (user directive Phase 15 FEEDBACK Q4, 2026-04-28; separated out from BRAIN15.9).
- BRAIN15.26 — HMM `--norm-mode` chrom-median default switch (user directive Phase 15 FEEDBACK Q9, 2026-04-28; promotes [ISSUE:2026-04-21:1] into Phase 15 scope).
- BRAIN15.27 — Composite multi-feature APS clustering modes with three-layer PCA composition (user directive Phase 15 FEEDBACK Q13 follow-up multi-round design discussion, 2026-04-28).
- BRAIN15.28 — `summit_excess` → `summit_rcn` rename + RCN-as-default migration (user directive Phase 15 FEEDBACK same discussion, 2026-04-28; sibling to BRAIN15.27).
- BRAIN15.29 — Summit-then-timing-then-updated-summit-then-updated-timing convergence (one-pass; user directive Phase 15 FEEDBACK Q12 promoting IBM-C6C with richer design, 2026-04-28).
- BRAIN15.31 — `--peak-summary` extension to RMS + HMM + max-projection variants (user directive Phase 15 FEEDBACK Q20 + `[ISSUE:2026-04-28:3]`, 2026-04-28).
- BRAIN15.33 — Phase 15 housekeeping bundle (timing prefix harmonization + summit_inspector fix + RMS summit-selector follow-up — user direction in Phase 15 FEEDBACK Round 3 marking `[ISSUE:2026-04-25:1]`, `[ISSUE:2026-04-28:4]`, `[ISSUE:2026-04-18:2]` as Phase-15-intended, 2026-04-28).
- BRAIN15.34 — Phase 15 closeout: tracking-file cleanup pass (apply migration-with-provenance convention everywhere — user direction in Phase 15 FEEDBACK Round 4, 2026-04-28).

**BRAIN-IDs marked RESOLVED (no Phase 15 implementation work):**

- BRAIN15.13 — `--hmm-smooth-halfwidth` APS split (resolved in Phase 14 Supplemental cycle 14S.23 — closeout marker).
- BRAIN15.15 — Cross-pipeline dedup code unification (resolved by Phase 14 Supplemental cycle 14-S20 — closeout marker; Phase 15 FEEDBACK Q7 investigation 2026-04-28).
- BRAIN15.30 — HMM sequence filtering CLI flags (`--chromosomes` / `--min-seq-length` / `--min-bin-count-per-seq`) — already implemented in HMM engine; Round 4 code audit 2026-04-28 verified. Closeout marker only.
- BRAIN15.32 — HMM fork age tracking visualization — already implemented in `hmm_fork_travel.py`; Round 4 code audit 2026-04-28 verified. Closeout marker only; any audit-driven enhancements fold into BRAIN15.17 / BRAIN15.5.
