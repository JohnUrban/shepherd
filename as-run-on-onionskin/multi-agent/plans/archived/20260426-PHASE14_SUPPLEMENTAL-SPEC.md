# PHASE 14 SUPPLEMENTAL SPEC

**Authors:** John M. Urban, Claude Code 2.1.104 (claude-sonnet-4-6), Claude Code (claude-opus-4-7)
**Revision:** REOPENED 2026-04-22 ~17:45 EDT. Priorities 14-S1/S2/S3 are CLOSED. The spec
is now expanded to its full intended scope: all CLI naming debt, help-string quality, flag
ordering, Universal-promotion candidates, and the standalone engine CLI architectural question
that were improperly relegated to `KNOWN_ISSUES.md` without user approval. `KNOWN_ISSUES.md`
entries 2026-04-22:1 through 2026-04-22:7 are hereby promoted back to active priorities here.

**Second expansion (2026-04-22 ~20:00 EDT):** A recon audit of Phase 14 + Phase 14 Supplemental
intended scope (see `PHASE14_SUPPLEMENTAL-FEEDBACK.md` "RECON AUDIT" section) surfaced
additional items from user feedback that were only partially promoted into priorities.
Priorities **14-S13 through 14-S18** are added below, each blocked on an OPEN QUESTION in
`PHASE14_SUPPLEMENTAL-FEEDBACK.md` (Q12–Q18). Priorities **14-S8, 14-S9, and 14-S12** have
expansion notes appended reflecting the same audit's findings.

**Third expansion (2026-04-22 ~20:40 EDT):** After user pushback ("Ignore the SPEC. Think of
the SPECs as the products we know are too narrow"), a second-pass audit of PHASE14_BRAINSTORM,
PHASE14_FEEDBACK (user-voice sections), KNOWN_ISSUES.md, and BRAINSTORM.md produced Findings
5–16 in `PHASE14_SUPPLEMENTAL-FEEDBACK.md`. Priorities **14-S19 through 14-S24** are added
below, each blocked on OPEN QUESTIONS Q19–Q26.

**Context:** Phase 14 proper closed at v0.14.46. Phase 14 Supplemental originally addressed 12
Growth model CLI flags, but the full intended scope — per `PHASE14_SUPPLEMENTAL-FEEDBACK.md` —
is a comprehensive CLI quality pass across all three pipelines: correct naming, Universal
promotion of shared concepts, step-mention help strings for ALL flag groups, placeholder flag
clarification, HMM help improvements, and resolution of the standalone engine CLI question.

All naming decisions are in `PHASE14_SUPPLEMENTAL-FEEDBACK.md`.

**All open questions resolved (Q9/Q10/Q11 answered in `PHASE14_SUPPLEMENTAL-FEEDBACK.md`):**
- 14-S4: `--growth-scan-halfwidth` / `--rms-scan-halfwidth`; "Second pass (pass 2)" / "pass 1"
- 14-S8: Expanded to full help-quality pass (step mentions + content rewrite for terse strings)
- 14-S10: Remove standalone engine CLIs

---

## Help string conventions (locked)

- Write "asymmetric triangle model fitting" with a brief inline context phrase — never bare.
  Example: "per-stage asymmetric triangle model fitting (independent left/right slopes per
  stage, used for fork travel rate and direction)"
- Use "halfwidth" in flag names where the flag's value IS a halfwidth (search radius, not a
  full window width). Smoothing spans are NOT halfwidths; "smooth" suffix is correct for those.
- Drop `-kb` from flag names; put units in help strings.
- Each help string must include a step mention in brackets using the full output directory
  step name: `[growth: 09-summit-refinement]`. Pipeline group headers may carry the step
  mention when all flags in the group share the same step.
- Universal flags planned for future pipelines: `[growth: 10-timing; rms: planned; hmm: planned]`.

---

---

## Reorder note (2026-04-23, post-v0.14.63)

Priorities below are ordered in **implementation-dependency sequence** rather than
the chronological Q-round order in which they were conceived. Priority identifiers
(`14-S1` … `14-S30`) are stable and unchanged — only the block order in this file
was reshuffled. The prior chronological layout is preserved verbatim at
`multi-agent/plans/PHASE14_SUPPLEMENTAL-SPEC.UNORDERED.md` for reference.

**Phase groups in this file:**

- **Phase 0 — Closed priorities** (14-S1, 14-S2, 14-S3). Retained at top with
  existing Role 2 implementation report and Role 1 re-audit blocks intact.
- **Phase 1 — Structural parser changes** (14-S10, 14-S4, 14-S5, 14-S29, 14-S30,
  14-S27, 14-S28, 14-S22, 14-S23, 14-S26). All flag renames, new groups, new flags,
  and runtime wiring land here so the CLI surface is final before later passes.
- **Phase 2 — Audit-only deliverables** (14-S12, 14-S25, 14-S14, 14-S15, 14-S21).
  Produce tracking docs and `PHASE15_BRAINSTORM.md` appendices. No code changes.
- **Phase 3 — Help-string polish** (14-S6, 14-S7, 14-S9, 14-S13, 14-S19, 14-S20,
  14-S8). Depends on final CLI surface from Phase 1. 14-S8 runs LAST within Phase 3
  as the integrating full-parser pass.
- **Phase 4 — Structural polish + tooling** (14-S11, 14-S24). Flag ordering and
  validator script land after everything else is final.
- **Phase 5 — Closeout** (14-S17, 14-S18, 14-S16). 14-S17 and 14-S18 are retained
  here as metadata pointers (FOLDED / NOT A PRIORITY); 14-S16 is the final
  implementation step (doc sweep + tracking-dir `git mv`).

**What changed structurally in this reorder:**

- Old section `## Expanded scope — priorities 14-S4 through 14-S12` (prior
  divider before 14-S4 in the chronological file) is dropped in favor of the
  phase-group headings below.
- The "Note — Findings 13, 14, 15, 16 routing" block stays alongside 14-S26 at
  the end of Phase 1 (its natural semantic partner).
- No priority content was edited, added, or removed. Every block from the
  UNORDERED file is preserved verbatim.

---

## v2 workflow migration (2026-04-23, mid-phase)

Phase 14 Supplemental was initiated under the v1 audit-loop workflow
(`multi-agent/workflows/spec_plan_three_role_audit_loop-v1.md`) where audit
findings, implementation reports, and re-audit closeouts lived embedded in this
SPEC file. At mid-stream v2 adoption, those embedded blocks for the 14-S1/S2/S3
cycle were moved verbatim to the sibling audit log
`multi-agent/plans/PHASE14_SUPPLEMENTAL-AUDIT_LOG.md`. This SPEC is now clean as
the authoritative contract.

Going forward (14-S4 onwards), all audit rounds, implementation reports, and
closeouts go into `PHASE14_SUPPLEMENTAL-AUDIT_LOG.md`, and each cycle closeout
produces ONE CHANGELOG entry with consolidated authorship per the v2 workflow
(`multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md`).

Historical CHANGELOG entries for 14-S1/S2/S3 (v0.14.48, v0.14.49) are NOT
rewritten — they remain per-round under v1 conventions. The priority status
table below reflects post-migration state.

---

## Priority status table

Updated at each cycle closeout. Phase numbers reference the implementation
phase-group structure (Phase 0 / 1 / 2 / 3 / 4 / 5) from the Reorder note above.

| Priority | Phase | Status | Short description |
|---|---|---|---|
| 14-S1 | 0 | CLOSED v0.14.49 | Rename 6 growth-specific flags |
| 14-S2 | 0 | CLOSED v0.14.49 | Peak summary → Universal with growth overrides |
| 14-S3 | 0 | CLOSED v0.14.49 | Asymmetric triangle model → new Universal group |
| 14-S10 | 1 | CLOSED v0.14.68 | Remove standalone engine CLIs |
| 14-S4 | 1 | CLOSED v0.14.68 | `--*-window` → `--*-scan-halfwidth`; Stage-2 → Pass-2 |
| 14-S5 | 1 | CLOSED v0.14.68 | Add `--hmm-emission-model`, deprecate `--hmm-emodel` |
| 14-S29 | 1 | CLOSED v0.14.68 | `--*-peak-search` → `--*-peak-search-halfwidth` |
| 14-S30 | 1 | CLOSED v0.14.68 | `--growth-shape-filter` on/off (default off) |
| 14-S27 | 1 | CLOSED v0.14.70 | Summit Refinement Universal promotion (new group, 3+3 flags) |
| 14-S28 | 1 | CLOSED v0.14.70 | Summit/origin terminology harmonization + column renames |
| 14-S22 | 1 | CLOSED v0.14.70 | Add `--aps-area-excess-floor on/off`, wire 3 pipelines |
| 14-S23 | 1 | CLOSED v0.14.70 | Split `--hmm-smooth-halfwidth` into step-5 + step-14 flags |
| 14-S26 | 1 | CLOSED v0.14.70 | Wire `--rms-shape-score-strict-bic` at runtime |
| 14-S12 | 2 | CLOSED v0.14.70.3 | RCN-profile flag survey — triaged into 14-S27 |
| 14-S25 | 2 | CLOSED v0.14.70.3 | `-rcn-` prefix audit — codification rides 14-S16 |
| 14-S14 | 2 | CLOSED v0.14.70.3 | Internal Stage-1/Stage-2 terminology audit (tracking doc) |
| 14-S15 | 2 | CLOSED v0.14.70.3 | HMM PuffStep synonym audit (PHASE15_BRAINSTORM appendix) |
| 14-S21 | 2 | CLOSED v0.14.70.3 | `--hmm-0-based-statepath` impact audit (PHASE15_BRAINSTORM) |
| 14-S6 | 3 | CLOSED v0.14.71 | `--aps-rank-by` help expansion |
| 14-S7 | 3 | CLOSED v0.14.71 | Placeholder flag help — post-14-S26 |
| 14-S9 | 3 | CLOSED v0.14.71 | HMM help improvements, PuffStep consolidation |
| 14-S13 | 3 | CLOSED v0.14.71 | Timing parser group step-mention pass |
| 14-S19 | 3 | CLOSED v0.14.71 | `--pipelines` two-block help + fix `all` semantics |
| 14-S20 | 3 | CLOSED v0.14.71 | APS group Universal-in-spirit re-framing + `--dedup-dist` verify |
| 14-S8 | 3 | CLOSED v0.14.72 | Full help-string quality pass across ALL parser groups |
| 14-S11 | 4 | CLOSED v0.14.73 | Flag ordering within pipeline groups |
| 14-S24 | 4 | CLOSED v0.14.73 | `validate-flags` script + Makefile target |
| 14-S17 | 5 | CLOSED v0.14.74 (folded into 14-S16) | (AGENT_CONVENTIONS CLI conventions update) |
| 14-S18 | 5 | RESOLVED v0.14.74 (inline drift audit clean) | (CLAUDE.md stale pointer — handled inline) |
| 14-S16 | 5 | CLOSED v0.14.74 | Closeout doc sweep + tracking-dir move |

**Cycle granularity suggestion** (final call is orchestrator's):
- Phase 1 can run as ONE batched cycle (10 priorities) OR split into a mechanical
  sub-cycle (14-S10, 14-S4, 14-S5, 14-S29, 14-S30) and substantive sub-cycles
  (14-S27+14-S28 together, 14-S22, 14-S23, 14-S26 each on its own). The
  substantive ones likely benefit from full audit-implement-reaudit; the
  mechanical ones are skip-reaudit candidates.
- Phase 2 is audit-only (no code changes) — can run as one batched cycle producing
  5 deliverable docs in parallel.
- Phase 3's 14-S8 is a substantive integrating pass — recommend its own cycle.
  14-S6, 14-S13, 14-S19 are thin and can be batched.
- Phase 4 is 2 priorities — one batched cycle.
- Phase 5 is 3 priorities with 14-S16 as the substantive one; its own cycle.

---

## Phase 0 — Closed priorities (14-S1, 14-S2, 14-S3)

Historical closed priorities from the original Phase 14 Supplemental (pre-v2
migration). Their scope blocks follow verbatim. Audit log history for these
priorities lives in `PHASE14_SUPPLEMENTAL-AUDIT_LOG.md`.

## Priority 14-S1 — Rename 6 growth-specific flags

**Goal:** Add `--growth-` prefix to 6 unprefixed flags in the Growth model parser group.
Update help strings with context, step mentions, and halfwidth/window naming corrections.
Add 6 deprecated-flag entries for the old names.

**Status: IMPLEMENTED — pending Role 1 re-audit.**

**Flags in scope:**

| Old name | New name | Notes |
|---|---|---|
| `--method` | `--growth-fit-method` | Q1 resolved |
| `--ensemble-methods` | `--growth-ensemble-methods` | Q1 resolved; active when `--growth-fit-method ensemble` |
| `--stage-weight-mode` | `--growth-stage-weight-mode` | Q2 resolved |
| `--refine-window-kb` | `--growth-refine-halfwidth` | Drop `-kb`; it IS a halfwidth (search radius); Q8 confirmed |
| `--refine-smooth-kb` | `--growth-refine-smooth` | Drop `-kb`; full-span smoothing window, NOT a halfwidth |
| `--stage-median-resolution` | `--growth-stage-median-resolution` | Q6: growth-only for now |

**Note on `--refine-*` flags:** Both parameterize Step 9 (`09-summit-refinement`).
`--refine-window-kb` is a search **halfwidth** (radius around the initial peak, not a full span)
→ renamed `--growth-refine-halfwidth` per Q7/Q8. `--refine-smooth-kb` is a full-span
smoothing window (code: `sm_w = smooth_kb * 1000 // bin_size` applied symmetrically) → NOT a
halfwidth, remains `--growth-refine-smooth`. Summit refinement is a shared downstream goal and
may be promoted to Universal in a future phase.

**After renaming, `args.*` attributes change automatically:**

| Old attribute | New attribute |
|---|---|
| `args.method` | `args.growth_fit_method` |
| `args.ensemble_methods` | `args.growth_ensemble_methods` |
| `args.stage_weight_mode` | `args.growth_stage_weight_mode` |
| `args.refine_window_kb` | `args.growth_refine_halfwidth` |
| `args.refine_smooth_kb` | `args.growth_refine_smooth` |
| `args.stage_median_resolution` | `args.growth_stage_median_resolution` |

**Improved help strings:**

```
--growth-fit-method    Trend-fitting method for the stage-progression growth evidence track.
                       linear: OLS slope across stages. isotonic: monotone regression.
                       step: best step change across stages (default). unimodal: single-peak
                       rise/fall. ensemble: weighted average of all four.
                       [growth: 01-growth-track]

--growth-ensemble-methods
                       Comma-separated base methods to combine when --growth-fit-method is
                       ensemble (choices: linear,isotonic,step,unimodal).
                       [growth: 01-growth-track]

--growth-stage-weight-mode
                       How replicate counts weight stages in per-stage asymmetric triangle
                       model fitting (independent left/right slopes per stage, used for fork
                       travel rate and direction) and in origin refinement. replicate: weight
                       by count (default). stage: equal weight per stage. equal: all
                       replicates equal.
                       [growth: 09-summit-refinement, 10-timing]

--growth-refine-halfwidth
                       Search halfwidth in kb around the initial amplicon peak estimate
                       within which the origin is refined by finding per-stage argmax
                       positions. Default 200 kb. [growth: 09-summit-refinement]

--growth-refine-smooth Smoothing span in kb applied to stage-median profiles during origin
                       refinement peak finding. Default 10 kb.
                       [growth: 09-summit-refinement]

--growth-stage-median-resolution
                       Resolution of stage-median bedGraphs used for within-calls tracks.
                       best: finest available hires resolution (default). base: base
                       resolution only. hires: hires resolution only.
                       [growth: 06-stage-medians]
```

### Implementation scope — Priority 14-S1

**1. `onionskin.py:build_parser()` — Growth model group**

For each flag: change the flag string, update the help string. Keep `choices=`, `type=`,
`default=`, `metavar=` unchanged. Do NOT add `dest=` (argparse auto-derives from flag name).

**2. `onionskin.py:_build_ms_argv()` — update `args.*` reads**

Engine-internal flag strings are unchanged (translation boundary rule):
```python
ms += ["--method", args.growth_fit_method]           # was args.method
ms += ["--ensemble-methods", args.growth_ensemble_methods]  # was args.ensemble_methods
ms += ["--stage-weight-mode", args.growth_stage_weight_mode]  # was args.stage_weight_mode
ms += ["--refine-window-kb", str(args.growth_refine_halfwidth)]  # was args.refine_window_kb
ms += ["--refine-smooth-kb", str(args.growth_refine_smooth)]  # was args.refine_smooth_kb
ms += ["--stage-median-resolution", args.growth_stage_median_resolution]  # was args.stage_median_resolution
```

Grep to confirm no remaining old-attr reads in `onionskin.py`:
```
grep -n "args\.method\b\|args\.ensemble_methods\|args\.stage_weight_mode\|args\.refine_window_kb\|args\.refine_smooth_kb\|args\.stage_median_resolution" onionskin.py
```

**3. `onionskin.py:_DEPRECATED_FLAGS` — add 6 entries**

Append after the existing 23 entries:
```python
"--method":                  "--growth-fit-method",
"--ensemble-methods":        "--growth-ensemble-methods",
"--stage-weight-mode":       "--growth-stage-weight-mode",
"--refine-window-kb":        "--growth-refine-halfwidth",
"--refine-smooth-kb":        "--growth-refine-smooth",
"--stage-median-resolution": "--growth-stage-median-resolution",
```

**4. Tests — `tests/test_pipeline.py`**

- `test_all_deprecated_flags_exit_with_redirect` covers all `_DEPRECATED_FLAGS` entries
  automatically; adding the 6 entries is sufficient for deprecation coverage.
- Add/update a focused growth help regression:
  - `--growth-fit-method` present in `python onionskin.py -h`
  - `--method` absent as a standalone Growth group flag
  - `--growth-refine-halfwidth` present
  - `--refine-window-kb` absent

**5. Documentation**

- `multi-agent/full_instructions/PIPELINE_SPEC.md` — update growth pipeline CLI flag table
  and any narrative references.
- `multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md` — update any CLI examples or
  flag-name references in growth discussion sections.

---

## Priority 14-S2 — Peak summary flags → Universal

**Goal:** Move `--peak-summary`, `--peak-quantile`, `--peak-topk` from the Growth model parser
group to the Universal parser group. Keep the same flag strings (no rename). Add
growth-specific overrides (`--growth-peak-*`) following the `_resolve_override_value()`
pattern. Update help strings with Universal framing and step mentions. No deprecated-flag
entries needed (the flag strings are unchanged).

**Status: IMPLEMENTED — pending Role 1 re-audit.**

**Architectural decision (Q5):** These flags control how per-call peak signal heights are
summarized across replicate stage medians for the peak profile output (`07-signal-tracks`).
They operate on RCN profiles, not on the growth evidence track — conceptually universal to
any pipeline that emits per-call signal profiles.

**Flags in scope:**

| Flag | Current group | New group | Growth override |
|---|---|---|---|
| `--peak-summary` | Growth model | Universal | `--growth-peak-summary` |
| `--peak-quantile` | Growth model | Universal | `--growth-peak-quantile` |
| `--peak-topk` | Growth model | Universal | `--growth-peak-topk` |

**Attribute names:**

| Attribute | Role | Default |
|---|---|---|
| `args.peak_summary` | Universal (unchanged; move parser group) | `"quantile"` |
| `args.peak_quantile` | Universal (unchanged) | `0.9` |
| `args.peak_topk` | Universal (unchanged) | `2` |
| `args.growth_peak_summary` | Growth override (new) | `None` |
| `args.growth_peak_quantile` | Growth override (new) | `None` |
| `args.growth_peak_topk` | Growth override (new) | `None` |

**Improved help strings for Universal flags:**

```
--peak-summary         Method to summarize per-call peak signal height across replicate stage
                       medians for the peak profile output. quantile: fractile across all
                       stage medians (see --peak-quantile). topk: mean of the top-k stage
                       medians (see --peak-topk). max: single highest stage median.
                       Currently applied only by the growth pipeline; planned for RMS and
                       HMM in a future phase. [growth: 07-signal-tracks]

--peak-quantile        Fractile (0–1) used when --peak-summary quantile (default 0.9).
                       [growth: 07-signal-tracks]

--peak-topk            Number of top stage medians averaged when --peak-summary topk
                       (default 2). [growth: 07-signal-tracks]
```

**Help strings for growth override flags:**

```
--growth-peak-summary  Override --peak-summary for the growth pipeline only.
--growth-peak-quantile Override --peak-quantile for the growth pipeline only.
--growth-peak-topk     Override --peak-topk for the growth pipeline only.
```

### Implementation scope — Priority 14-S2

**1. `onionskin.py:build_parser()` — move flags and add overrides**

- Remove `--peak-summary`, `--peak-quantile`, `--peak-topk` from the Growth model group.
- Add them to the Universal group (`uni.add_argument(...)`) with the updated help strings
  above. Keep `choices=`, `type=`, `default=` unchanged.
- Add `--growth-peak-summary`, `--growth-peak-quantile`, `--growth-peak-topk` to the Growth
  model group with `default=None` and the override help strings above.

**2. `onionskin.py:_build_ms_argv()` — use `_resolve_override_value()`**

```python
ms += ["--peak-summary",  _resolve_override_value(args, 'growth_peak_summary',  'peak_summary')]
ms += ["--peak-quantile", str(_resolve_override_value(args, 'growth_peak_quantile', 'peak_quantile'))]
ms += ["--peak-topk",     str(_resolve_override_value(args, 'growth_peak_topk',     'peak_topk'))]
```

Grep to confirm no remaining un-resolved reads:
```
grep -n "args\.peak_summary\|args\.peak_quantile\|args\.peak_topk" onionskin.py
```

**3. `onionskin.py:_DEPRECATED_FLAGS` — no new entries**

Flag strings are unchanged; no deprecation needed.

**4. Tests — `tests/test_pipeline.py`**

- Add a help regression:
  - `--peak-summary` appears in Universal section of `python onionskin.py -h`
  - `--growth-peak-summary` appears in Growth model override section

**5. Documentation**

Update PIPELINE_SPEC.md and ONIONSKIN_FULL_HANDOFF.md to reflect Universal promotion.

---

## Priority 14-S3 — Asymmetric triangle model flags → Universal

**Goal:** Move `--model-window-kb`, `--model-smooth-kb`, `--w-grid-kb` from the Growth model
parser group to a new Universal-tier "Asymmetric Triangle Model" parser group. Rename them
with `--asym-tri-model-*` names. Add growth-specific overrides. Add 3 deprecated-flag entries.
Update `_build_ms_argv()` to use `_resolve_override_value()`.

**Status: IMPLEMENTED — pending Role 1 re-audit.**

**Architectural decision:** Per-stage asymmetric triangle model fitting for fork travel rate and
direction is a shared downstream goal applicable to all three pipelines. These flags become
Universal now; the growth pipeline is the only current consumer but help strings note the
planned future scope.

**Naming rationale (Q7/Q8):**
- `--model-window-kb` → `--asym-tri-model-halfwidth` — the "window" IS a halfwidth (search
  radius around the refined origin, not a full span); Q8 confirmed "halfwidth" naming.
- `--model-smooth-kb` → `--asym-tri-model-smooth` — the smoothing span is a FULL span
  (code: `sm_w = smooth_kb * 1000 // bin_size` applied symmetrically); NOT a halfwidth.
- `--w-grid-kb` → `--asym-tri-model-halfwidth-grid` — `w` = `wL`/`wR` = halfwidths.

**Flags in scope:**

| Old name | Universal flag | Growth override |
|---|---|---|
| `--model-window-kb` | `--asym-tri-model-halfwidth` | `--growth-asym-tri-model-halfwidth` |
| `--model-smooth-kb` | `--asym-tri-model-smooth` | `--growth-asym-tri-model-smooth` |
| `--w-grid-kb` | `--asym-tri-model-halfwidth-grid` | `--growth-asym-tri-model-halfwidth-grid` |

**Attribute names:**

| Attribute | Role | Default |
|---|---|---|
| `args.asym_tri_model_halfwidth` | Universal | `300` |
| `args.asym_tri_model_smooth` | Universal | `10` |
| `args.asym_tri_model_halfwidth_grid` | Universal | `"20,40,80,120,160,220,300"` |
| `args.growth_asym_tri_model_halfwidth` | Growth override | `None` |
| `args.growth_asym_tri_model_smooth` | Growth override | `None` |
| `args.growth_asym_tri_model_halfwidth_grid` | Growth override | `None` |

**New parser group — "Asymmetric Triangle Model":**

```python
atm = p.add_argument_group(
    "Asymmetric Triangle Model",
    "Per-stage asymmetric triangle model fitting (independent left/right slopes per stage, "
    "used for fork travel rate and direction). Currently applied by the growth pipeline "
    "(10-timing); planned for RMS and HMM in a future phase."
)
```

Place this group AFTER the existing Timing group and BEFORE the APS group in `build_parser()`.

**Help strings for Universal flags:**

```
--asym-tri-model-halfwidth
                       Search halfwidth in kb around the refined amplicon origin used to fit
                       the per-stage asymmetric triangle model (independent left/right slopes
                       per stage, used for fork travel rate and direction). Default 300 kb.
                       [growth: 10-timing; rms: planned; hmm: planned]

--asym-tri-model-smooth
                       Smoothing span in kb applied to the stage-median profile before
                       curvature estimation in the per-stage asymmetric triangle model fit.
                       Full-span window; does not affect the final fit itself. Default 10 kb.
                       [growth: 10-timing; rms: planned; hmm: planned]

--asym-tri-model-halfwidth-grid
                       Comma-separated grid of half-width values in kb searched for the
                       left (wL) and right (wR) sides of the asymmetric triangle model.
                       Each wL/wR pair evaluated by least-squares; best BIC winner kept.
                       Default 20,40,80,120,160,220,300.
                       [growth: 10-timing; rms: planned; hmm: planned]
```

**Help strings for growth override flags:**

```
--growth-asym-tri-model-halfwidth
                       Override --asym-tri-model-halfwidth for the growth pipeline only.
--growth-asym-tri-model-smooth
                       Override --asym-tri-model-smooth for the growth pipeline only.
--growth-asym-tri-model-halfwidth-grid
                       Override --asym-tri-model-halfwidth-grid for the growth pipeline only.
```

### Implementation scope — Priority 14-S3

**1. `onionskin.py:build_parser()` — add Universal group, remove from Growth group**

- Remove `--model-window-kb`, `--model-smooth-kb`, `--w-grid-kb` from the Growth model group.
- Add the new `atm` group (see above) with the 3 Universal flags. Keep `type=`, `default=`
  unchanged from the originals (`type=int` for halfwidth and smooth; `type=str` for grid).
- Add `--growth-asym-tri-model-halfwidth`, `--growth-asym-tri-model-smooth`,
  `--growth-asym-tri-model-halfwidth-grid` to the Growth model group with `default=None`.

**2. `onionskin.py:_build_ms_argv()` — use `_resolve_override_value()`**

Engine-internal flag strings are unchanged (translation boundary rule):
```python
ms += ["--model-window-kb", str(_resolve_override_value(args, 'growth_asym_tri_model_halfwidth', 'asym_tri_model_halfwidth'))]
ms += ["--model-smooth-kb", str(_resolve_override_value(args, 'growth_asym_tri_model_smooth',    'asym_tri_model_smooth'))]
ms += ["--w-grid-kb", _resolve_override_value(args, 'growth_asym_tri_model_halfwidth_grid', 'asym_tri_model_halfwidth_grid')]
```

Grep to confirm no remaining old-attr reads:
```
grep -n "args\.model_window_kb\|args\.model_smooth_kb\|args\.w_grid_kb" onionskin.py
```

**3. `onionskin.py:_DEPRECATED_FLAGS` — add 3 entries**

```python
"--model-window-kb":         "--asym-tri-model-halfwidth",
"--model-smooth-kb":         "--asym-tri-model-smooth",
"--w-grid-kb":               "--asym-tri-model-halfwidth-grid",
```

**4. Tests — `tests/test_pipeline.py`**

- `test_all_deprecated_flags_exit_with_redirect` covers the 3 new entries automatically.
- Add a help regression:
  - `--asym-tri-model-halfwidth` present in `python onionskin.py -h`
  - `--model-window-kb` absent
  - `--asym-tri-model-halfwidth-grid` present
  - `--w-grid-kb` absent

**5. Documentation**

Update PIPELINE_SPEC.md and ONIONSKIN_FULL_HANDOFF.md to reflect the new Universal group
and renamed flags.

---

## Scope boundary — do NOT change

- `growth_model_engine.py` standalone parser flag strings — the engine keeps `--method`,
  `--model-window-kb`, `--w-grid-kb`, `--peak-summary`, etc. internally (translation boundary
  rule from Phase 14.2 / 14.6).
- `onionskin_core/` internal Python parameter names (`method`, `w_grid_kb`, `peak_summary`,
  etc. in function signatures) — internal APIs, not user-facing CLI.
- `ROADMAP.md` historical phase records.
- `CHANGELOG.md` entries already written.
- Items identified in the Copilot audit but outside current scope: `--growth-window`/
  `--rms-window` halfwidth rename, HMM naming cleanup, placeholder flag help, standalone
  engine CLI architectural question, repo-wide step-mention harmonization. These are tracked
  in `KNOWN_ISSUES.md`.

---

## Phase 1 — Structural parser changes (post-S1/S2/S3)

*Implementation order:* **14-S10 → 14-S4 → 14-S5 → 14-S29 → 14-S30 → 14-S27 →
14-S28 → 14-S22 → 14-S23 → 14-S26.**

*Rationale:* all flag renames, new groups, new flags, and runtime wiring land
first so the CLI surface is final before help-string polish (Phase 3) and
flag-ordering (Phase 4) touch it. 14-S10 runs first because removing the
standalone engine CLIs frees subsequent passes from dual-CLI sync burden. 14-S27
and 14-S28 land in the same Role 2 round per the existing "Coordination with
14-S28" directive in the 14-S27 block.

## Priority 14-S10 — Standalone engine CLI: keep vs. remove/deprecate

**Goal:** Make an explicit architectural decision on the standalone argparse CLIs in
`growth_model_engine.py` and `rcn_mean_shift_engine.py`, then implement accordingly.

**Status: OPEN QUESTION — awaiting user decision.**

**User question (from feedback):** "Why do we still have user-facing CLI flags in the engines
themselves if they are never expected to be directly used by us or a user? Do we have a need to
use them directly in development? Or is it sufficient to access them through the onionskin CLI?"

**Options:**

| Option | Implication |
|---|---|
| Keep + help-string parity pass | Maintain both CLIs; requires ongoing sync with top-level CLI; do a help-quality pass now (Priority 14-S8 step mentions would apply here too) |
| Remove engine argparse parsers | Engines become pure Python APIs with no standalone `-h`; `onionskin.py` is the sole entry point; simplifies long-term maintenance |
| Deprecate (keep but warn) | Add a deprecation notice to the engine `main()` output; remove in a future phase |

**Decision (Q11 resolved in `PHASE14_SUPPLEMENTAL-FEEDBACK.md`): REMOVE.**

User: "Let's move forward with simplifying onionskin here since all pipelines are accessed from
onionskin.py. I do not foresee directly using the engines, especially if they are not maintained
in parallel, which has proven burdensome."

**Status: READY.**

**Implementation scope:**

1. `onionskin_core/engines/growth_model_engine.py`:
   - Remove the `argparse` parser construction and `main()` entry point entirely
   - The engine remains a fully functional Python module; only the standalone CLI is removed
   - Remove the `if __name__ == "__main__": main()` guard
   - Remove any imports used exclusively by `main()` (e.g., `argparse`)
2. `onionskin_core/engines/rcn_mean_shift_engine.py`:
   - Same: remove `argparse` parser, `main()`, `if __name__ == "__main__"` guard, and
     any `main()`-only imports
3. `setup.py` / `pyproject.toml` — remove any `console_scripts` entry points for the engine
   CLIs if present
4. `tests/test_pipeline.py` — remove any tests that invoke the engine CLIs standalone via
   `subprocess` or `-h`; replace with a note that engine-level help is no longer available
   (these were the `growth_engine_help` and `rms_engine_help` focused regressions added in
   Phase 14.6 — those tests must be removed or converted to import-level checks)
5. `PIPELINE_SPEC.md` and `ONIONSKIN_FULL_HANDOFF.md` — remove any mention of standalone engine
   CLI invocation; add a note that both engines are invoked exclusively via `onionskin.py`
6. `multi-agent/project_context/DECISIONS.md` — record the removal decision and rationale

---

## Priority 14-S4 — `--growth-window` / `--rms-window` halfwidth rename + Stage-2→Pass-2 terminology

**Goal:** Apply the Q7 halfwidth-naming rule to `--growth-window` and `--rms-window`, both of
which have help text describing a "half-size" value (i.e. a halfwidth). Add 1-2 words to each
new name for intuition. Update "Stage-2" → "Pass-2" (or agreed equivalent) in help strings.
Update "half-size" → "halfwidth" in help strings. Add deprecated-flag entries.

**Status: READY.**

**Decisions (Q9/Q10 resolved in `PHASE14_SUPPLEMENTAL-FEEDBACK.md`):**

- `--growth-window` → **`--growth-scan-halfwidth`**
- `--rms-window` → **`--rms-scan-halfwidth`**
- "Stage-2" → **"Second pass (pass 2)"** on first use in a help string; **"pass 2"** on
  subsequent uses within the same string. Same pattern for "Stage-1" / "pass 1".
- Note: `--rms-halfwidths` / `--growth-halfwidths` are the pass-1 counterparts; help strings
  for the new scan-halfwidth flags should cross-reference this relationship so users understand
  the two-pass structure. Help strings should not mention "pass 2" in isolation without
  contextualizing that there is a "pass 1."
- Note for DECISIONS.md: "ms" is reserved as shorthand for both "mean-shift" and "multi-stage"
  in this codebase — avoid it in new flag names to prevent ambiguity.

**Implementation scope:**

1. `onionskin.py:build_parser()` — rename both flags; update help strings:
   - `--growth-window` → `--growth-scan-halfwidth`
   - `--rms-window` → `--rms-scan-halfwidth`
   - Update help: "Stage-2" → "Second pass (pass 2)"; "half-size" → "halfwidth"
   - Cross-reference the pass-1 counterpart (`--growth-halfwidths` / `--rms-halfwidths`)
   - Add step mentions (coordinate with 14-S8)
2. `onionskin.py:_build_ms_argv()` — update `args.growth_window` → `args.growth_scan_halfwidth`
   and `args.rms_window` → `args.rms_scan_halfwidth`
3. `onionskin.py:_DEPRECATED_FLAGS` — add:
   ```python
   "--growth-window": "--growth-scan-halfwidth",
   "--rms-window":    "--rms-scan-halfwidth",
   ```
4. `tests/test_pipeline.py` — add help regression: new names present, old names absent
5. `PIPELINE_SPEC.md` — update any references to `--growth-window` / `--rms-window`
6. `multi-agent/project_context/DECISIONS.md` — record the "ms" ambiguity note

---

## Priority 14-S5 — Add `--hmm-emission-model`; deprecate `--hmm-emodel`

**Goal:** Add `--hmm-emission-model` as the new canonical HMM emission model flag. Keep
`--hmm-emodel` as a deprecated PuffStep synonym (redirect to `--hmm-emission-model` via
`_DEPRECATED_FLAGS`). Do NOT simplify the help string — keep the full model jargon.

**Status: READY.**

**User decision (from feedback):** "We can add `--hmm-emission-model` but keep `--hmm-emodel`
as a remnant PuffStep synonym." "I disagree with `--hmm-emodel` being too full of stuff — I
want it to be."

**Implementation scope:**

1. `onionskin.py:build_parser()` — HMM group:
   - Rename `--hmm-emodel` → `--hmm-emission-model` (same choices, same help text, same default)
   - Add a note in the help string that `--hmm-emodel` is the PuffStep synonym (now deprecated)
2. `onionskin.py:_DEPRECATED_FLAGS` — add:
   ```python
   "--hmm-emodel": "--hmm-emission-model",
   ```
3. `tests/test_pipeline.py` — add help regression: `--hmm-emission-model` present, verify
   deprecated-flag test covers `--hmm-emodel` (covered automatically by
   `test_all_deprecated_flags_exit_with_redirect` once the entry is added)
4. `PIPELINE_SPEC.md` — update any references from `--hmm-emodel` to `--hmm-emission-model`

---

## Priority 14-S29 — Detection-flag halfwidth review (Q33 resolved)

**Goal (Q33 resolved):** User accepted auditor pushback. Rename only
`--growth-peak-search` (and RMS parallel `--rms-peak-search`) to add the `-halfwidth`
suffix. Leave `--growth-trend` / `--rms-trend` and `--growth-smooth` / `--rms-smooth`
unchanged (they are full-width windows, not halfwidths).

**Status: READY.**

**Preliminary code check (Role 2 will verify more carefully):**
- `--growth-trend` (help: "Rolling-median baseline window in kb") — rolling-median
  windows are typically **full-width** spans, not halfwidths. Likely NO rename.
- `--growth-smooth` (help: "Smoothing window for residual signal in kb") — likely
  full-width span. Likely NO rename.
- `--growth-peak-search` (help: "Stage-2 peak search radius in kb") — "radius" IS a
  halfwidth. **Rename candidate**: `--growth-peak-search` → `--growth-peak-search-halfwidth`
  OR → `--growth-scan-peak-search-halfwidth` (incorporating the pass-2 "scan" prefix
  introduced in 14-S4).

**Implementation scope:**
1. Role 2 verifies `--growth-peak-search` / `--rms-peak-search` really is a halfwidth
   via code check (`onionskin_core/detection.py` and the mean-shift engines).
2. Rename:
   - `--growth-peak-search` → `--growth-peak-search-halfwidth`
   - `--rms-peak-search` → `--rms-peak-search-halfwidth`
3. Add `_DEPRECATED_FLAGS` redirects for both old names.
4. Update help strings: keep the "Stage-2" context (or "pass 2" per 14-S4 terminology
   harmonization) in the wording.
5. Leave `--growth-trend`, `--rms-trend`, `--growth-smooth`, `--rms-smooth` untouched
   (they are full-width windows per the help-text semantics).
6. Tests: help regression + deprecated-flag coverage for the 2 renames.

---

## Priority 14-S30 — `--growth-shape-filter` on/off semantics with safe default (Q34 resolved)

**Goal (Q34 resolved):** Flip `--growth-shape-filter` from `action="store_true"` to
`choices=["on", "off"]`. Keep **default=off** for now (user softened from the original
`default=on` proposal to minimize risk). Add a KNOWN_ISSUE entry scheduling the
default-flip test for a future phase.

**Status: READY.**

**User direction (Q34):**
> "I approve this change. I am not sure what to look out for. If you want, we can use
> default off for now until it is further tested. That might make most sense. Then put a
> KNOWN_ISSUE that we need to test keeping it on. Does it change any results? I believe
> we ran this once and found out it does not. So it could be harmless to have on."

**Code-investigation finding (growth-track vs RCN signal question):**

Filter operates on `shape_score_raw` (dBIC_flat_vs_tri) computed in `stage2_score`
during the multistage engine run. `stage2_score` reads per-stage profiles after
`add_norm_log2()` — i.e., log2-RCN profiles. So the growth shape-filter operates on
**RCN signal**, not the growth evidence track.

Role 2 verifies this during implementation and writes it into the updated help string.

**Implementation scope:**
1. `onionskin.py:build_parser()` — change `--growth-shape-filter` from
   `action="store_true"` to `choices=["on", "off"], default="off"`.
2. Update help string to say: "Apply shape filter to multistage calls (operates on
   log2-RCN profiles via the dBIC_flat_vs_tri metric). Default: off. Setting to `on`
   removes calls below `--growth-shape-score-threshold` from `_calls.tsv`."
3. Runtime: update `args.growth_shape_filter` reads (currently `getattr(...False)` per
   `onionskin.py:3494`) to use the choices comparison `args.growth_shape_filter == "on"`.
4. Breaking change: `--growth-shape-filter` (bare, no arg) is no longer valid argparse.
   Users invoking it bare will get an argparse error. Acceptable per scope-authority
   (user approved). Document in CHANGELOG.
5. Tests: update any test that sets the boolean flag to use `--growth-shape-filter off`
   (explicitly); add regression for the new choices-based form.
6. `multi-agent/KNOWN_ISSUES.md`: add entry scheduling the default-flip test for a
   future phase (Phase 15 or later).
## Priority 14-S27 — Universal promotion of summit-refinement flags (Q28 partially resolved; Q31/Q32 pending)

**Goal (Q28 resolved for promotion):** Promote four currently-growth-group flags to
Universal where all three pipelines will eventually use them (or already could). The
user's feedback explicitly directed that "promote next supplemental" decisions land in
THIS supplemental since the next phase is HMM development.

**Status: PARTIAL READY — blocked on Q31 (group placement) and Q32 (final flag names).**

**Flags to promote (3 of originally 4; Q31/Q32 resolved):**

| Current growth-group flag | Universal name | Scope (pre-Phase-15) |
|---|---|---|
| `--growth-stage-weight-mode` | `--stage-weight-mode` | Growth only today; RMS/HMM summit-refinement wiring is Phase 15 |
| `--growth-refine-halfwidth` | `--refine-summit-halfwidth` | Growth only today; applies to per-stage argmax during refinement |
| `--growth-refine-smooth` | `--refine-summit-smooth` | Growth only today; smoothing span before per-stage argmax |

**Pattern:** promote to Universal under new "Summit Refinement" parser group (Q31
resolved → option 1), add `--growth-*` overrides following the `_resolve_override_value()`
pattern used by 14-S2/S3. Help strings say "Currently applied by the growth pipeline;
planned for RMS and HMM in Phase 15 HMM completeness work."

**Decision: `--growth-rcn-smooth-halfwidth` stays GROWTH-SPECIFIC (Q36 confirmed).**

Code investigation rationale:
- The flag controls per-sample RCN smoothing before BOTH summit parabola fitting AND APS
  computation — a dual-use upstream smoothing pass unique to the growth pipeline's
  architecture.
- HMM has its own RCN smoothing via `--hmm-smooth-halfwidth` (step-5 whole-genome) and
  `--hmm-aps-smooth-halfwidth` (step-14 APS, per 14-S23). Forcing HMM to use a promoted
  universal flag would cause double-smoothing.
- RMS does not currently smooth RCN before its summit parabola path (`refine_summit_parabola`);
  no analogous pre-smoothing pass exists.
- The flag remains `--growth-rcn-smooth-halfwidth` in the Growth model parser group.

**KNOWN_ISSUES entry to create during Role 2 implementation** (per user Q36 direction):
> "It is concerning that RMS does not pre-smooth RCN before its summit parabola. I believe
> we found that helped in an earlier testing round."
> — John M. Urban

Add `multi-agent/KNOWN_ISSUES.md` (or `multi-agent/tracking/KNOWN_ISSUES.md` if the
closeout-time move has happened) entry: investigate whether RMS summit parabola
(`refine_summit_parabola` in `rcn_mean_shift_helpers.py`) would benefit from pre-smoothing
RCN profiles like the growth pipeline does via `--growth-rcn-smooth-halfwidth`. Check
earlier testing notes for prior-round evidence. If beneficial, add an RMS equivalent flag
(`--rms-rcn-smooth-halfwidth`).

**Summit Refinement group placement:** new parser group between Timing and Asymmetric
Triangle Model (Q31 option 1).

**Implementation scope (Q31/Q32/Q36 resolved; 14-S27 READY):**
1. `onionskin.py:build_parser()`:
   - Add new "Summit Refinement" parser group between Timing and Asymmetric Triangle
     Model. Group **description** includes the band-aid help text from 14-S28 explaining
     the mixed semantics of `final_summit_low_bp`/`final_summit_high_bp` and pointing to
     `PHASE16_BRAINSTORM.md` for the future unification pass.
   - Add the 3 Universal flags (`--stage-weight-mode`, `--refine-summit-halfwidth`,
     `--refine-summit-smooth`) with help strings that carry multi-pipeline step mentions
     (e.g., `[growth: 09-summit-refinement; rms: planned; hmm: planned]`) and note
     "currently applied by the growth pipeline."
   - Add the 3 growth overrides (`--growth-stage-weight-mode`,
     `--growth-refine-summit-halfwidth`, `--growth-refine-summit-smooth`) with
     `default=None` per the override pattern.
   - Leave `--growth-rcn-smooth-halfwidth` unchanged in the Growth model group.
2. `onionskin.py:_build_ms_argv()` — update to use `_resolve_override_value()` for each
   of the 3 universal/override pairs.
3. `onionskin.py:_DEPRECATED_FLAGS` — add 3 redirect entries so explicit invocations of
   the old names hard-fail with the new names.
4. Tests — help regression confirming the 3 new universal + 3 new override flags appear;
   deprecated-flag regression covers the old-name hard-fails automatically.
5. `PIPELINE_SPEC.md`, `ONIONSKIN_FULL_HANDOFF.md` — document the new Summit Refinement
   group (fold into 14-S16 closeout doc sweep).

**Coordination with 14-S28:** Terminology-harmonization decisions determine the exact
wording used in the new help strings. Both priorities land in the same Role 2 round.

---

## Priority 14-S28 — Summit/origin terminology harmonization (public-facing) (raised in Q28)

**Goal:** The user noted that "origin" and "summit" are used interchangeably in
user-facing flags and help strings. Live code investigation confirms this:
- Flag names that use "summit": `--rms-summit-policy`, `--rms-early-summit-stages`
- Flag names / help that use "origin": `--bootstrap-origins`, "refined amplicon origin",
  "origin refinement peak finding"
- Function names split similarly: `refine_summit_parabola` (RMS) vs
  `refine_origin_for_call` + `refine_origin_sliding_offset` (growth).

User decision (Q28): standardize on "summit" for public-facing content (flag names,
help strings). Internal code function/variable names stay as-is (internal API boundary).
Help strings for summit-refinement flags explicitly explain that summit is a proxy for
the re-replication initiation zone / biological origin.

**Status: READY.**

**Scope boundary:**
- Flag names: use "summit" everywhere "origin" currently is used interchangeably.
- Help strings: use "summit" consistently; add one-line clarification that summit is a
  proxy for the origin / initiation zone.
- Internal code (function names, docstrings, variable names in `onionskin_core/`):
  NOT renamed in this priority per user direction "at least for the public facing
  stuff." Internal naming audit could be a Phase 15+ item.

**Flags / help strings affected (Q35 + Q37 resolved):**

Code investigation of `--bootstrap-origins` confirmed it is the SAME conflation of
origin/summit found elsewhere. `_bootstrap_peak_ci` (RMS) and `refine_origin_for_call`
(growth) both bootstrap over replicates to produce confidence intervals on the DETECTED
PEAK POSITION — which is the summit. "Origin" here is not a narrower technical term;
it is legacy naming for the summit.

**Flag renames (Q37 approved):**

| Old | New |
|---|---|
| `--bootstrap-origins` | `--bootstrap-summits` |
| `--growth-bootstrap-origins` | `--growth-bootstrap-summits` |
| `--rms-bootstrap-origins` | `--rms-bootstrap-summits` |

- `--rms-summit-policy`, `--rms-early-summit-stages`: already use "summit"; no change.
- New flags from 14-S27: `--refine-summit-halfwidth`, `--refine-summit-smooth` use
  "summit" (decided in 14-S27 naming).
- Help strings across the parser that say "origin refinement", "refined amplicon origin",
  etc. — rewrite to "summit refinement", "refined amplicon summit", with an inline note
  that summit is the proxy for the biological origin / initiation zone.

**Output column renames (Q38 option 1 + Q39 resolved; band-aid path chosen):**

User approved renaming output columns (not just flags) for consistency. Blast radius:
`notebooks.py`, `summit_plots.py`, test fixtures, any downstream consumer of
`_origins.tsv`.

**Code-investigation finding surfaced during Q39 (critical context):** the current
`origin_ci_low/high_bp` columns have MIXED semantics — they carry the range across
per-stage parabola vertices when parabola wins, and bootstrap CI on argmax_mean when
argmax wins (fallback). `origin_boot_sd_bp` always takes the argmax-based bootstrap SD
regardless of winner. See `growth_model_engine.py:1510–1543`.

**Band-aid path (chosen):** rename columns with honest labels that do NOT claim "CI" or
"bootstrap" in ways that would be inaccurate. Defer the structural fix (bootstrap the
parabola directly) to a dedicated future Summit Methodology phase — see
`multi-agent/plans/next/PHASE16_BRAINSTORM.md`.

Final column rename table:

| Old | New | Meaning (post-rename) |
|---|---|---|
| `final_origin_bp` | `final_summit_bp` | Winner summit estimate (parabola when `n_parabola_valid ≥ 1`, else argmax). Disambiguated by `summit_estimator_used` column. |
| `origin_ci_low_bp` | `final_summit_low_bp` | Interval lower bound. Semantics depend on winner: range across per-stage parabola vertices (parabola wins) OR bootstrap CI on argmax_mean (argmax wins). |
| `origin_ci_high_bp` | `final_summit_high_bp` | Interval upper bound. Same mixed semantics. |
| `origin_ci_width_bp` | `final_summit_width_bp` | `high − low`. Dropped "ci" because the width inherits the same mixed semantics as low/high. |
| `origin_boot_sd_bp` | `argmax_mean_bootstrap_sd_bp` | Honest renaming — the SD is always argmax-based (bootstrap over replicates), regardless of which estimator won. Not the SD of `final_summit_bp` when parabola wins. |

**Band-aid parser-group help text (added to 14-S27 new Summit Refinement group):**

Add a short description block visible in `onionskin -h` so users do not need to dig
through PIPELINE_SPEC.md to understand the mixed semantics:

> "The `final_summit_bp` column in `_origins.tsv` is the winner between the parabola
> estimator (when at least one stage produced a valid fit) and the argmax estimator
> (fallback). See the `summit_estimator_used` column per row. The `final_summit_low_bp` /
> `final_summit_high_bp` columns carry interval bounds whose semantics depend on the
> winner: range across per-stage parabola vertices when parabola wins; bootstrap
> confidence interval on argmax_mean when argmax wins. A dedicated Summit Methodology
> pass is planned (see `PHASE16_BRAINSTORM.md`) to unify these semantics
> (bootstrap the parabola directly; extend to cross-pipeline summit harmonization)."

**Scope boundary:**
- Internal function/variable names in `onionskin_core/` (`refine_origin_for_call`,
  `refine_origin_sliding_offset`, `origin_bp` locals, etc.) stay unchanged per the 14.2
  internal-boundary rule.
- Parabola bootstrap NOT implemented in this priority — deferred to Summit Methodology
  phase.
- Cross-pipeline summit harmonization, dynamic origin-detection window, HMM peak_rcn_stage,
  RMS pre-smoothing investigation — all ALSO deferred to Summit Methodology phase.

**Implementation scope (Q37 + Q38 resolved; pending Q39):**

1. Rename the 3 bootstrap flags in `onionskin.py:build_parser()`. Add 3
   `_DEPRECATED_FLAGS` redirect entries.
2. Rename the 5 output columns in the growth engine (`growth_model_engine.py`) — the
   writer sites where `final_origin_bp` and its CI columns are emitted to `_origins.tsv`.
3. Update all consumer read-sites:
   - `onionskin_core/notebooks.py` — `origin_ci_lo`, `origin_ci_hi`, `origin_ci_low_bp`,
     `origin_ci_high_bp` references.
   - `onionskin_core/summit_plots.py` — same + `origin_boot_sd` references.
   - Any test fixtures that reference these columns.
4. Update help strings for `origin`→`summit` across the parser.
5. Update documentation: `PIPELINE_SPEC.md` (column schema section),
   `ONIONSKIN_FULL_HANDOFF.md` (any column-reference prose), README (if applicable).
6. Tests: add a focused regression that a fresh run emits the new column names and none
   of the old names.

**Scope boundary:** internal function/variable names in `onionskin_core/`
(`refine_origin_for_call`, `refine_origin_sliding_offset`, per-stage local
`origin` variables) stay unchanged. Only emitted columns, flag names, and help strings
change. This matches the Phase 14.2 internal-boundary rule.

**Implementation scope:**
1. `onionskin.py:build_parser()`: rename flags where "origin" is used interchangeably
   with "summit" (Q32 confirms per-flag list). Update all help strings to use "summit."
   Add an inline clarification note (single sentence) to the help for the newly-Universal
   summit-refinement flags explaining the summit-as-origin-proxy relationship.
2. `_DEPRECATED_FLAGS`: add redirect entries for any renamed flags.
3. Docs (`PIPELINE_SPEC.md`, `ONIONSKIN_FULL_HANDOFF.md`): standardize user-facing "summit"
   terminology; preserve internal "origin" references where they refer to function names
   or variable names.
4. Tests: help regression covers the new wording.

**Out of scope (Phase 15 or later):** internal `refine_origin_*` function renames,
`onionskin_core/` variable names, internal docstrings. These follow the Phase 14.2
scope-boundary rule that stops renames at the `onionskin.py` call boundary.

---

## Priority 14-S22 — Add `--aps-area-excess-floor on|off` flag, wired across all three pipelines (Q22 resolved)

**Goal (Q22 resolved):** Add the CLI flag NOW and wire it into all three pipelines' APS
modules (HMM step-14, Growth step-13, RMS step-12). Default is `on` to preserve current
behavior. A future Phase 15 priority will test and possibly flip the default.

**Status: READY.**

**User direction (Q22):**
> "We should add `--aps-area-excess-floor` AND wire it into all three pipeline APS modules.
> Default should be to whatever the current behavior is (default=on seems to be correct).
> We likely need to handle two separate issues
> 1. Area excess for APS scores : subtracting 1 and setting negative bins to 0 was a somewhat
>    clever way to force each amplicon to contribute >= 0 to APS; however, since RCN values
>    oscillate around 1, when 1 is subtracted and floored at 0, it systematically makes the
>    APS higher than 0 across all bins. It should only do the 0 operation at the end after
>    the area is summed, if it is less than 0, set it to 0. This allows the negative bins to
>    keep the APS of an amplicon closer to 0 in general, and adds a slight correction after
>    seeing the sum, if needed.
> 2. Area excess for shape-based clustering: Setting the floor to 0 probably does the most
>    damage here, which is what the original hypothesis was. We were trying to find the best
>    way(s) to cluster to find the posterior groupings. We realized setting the 0 floor (or
>    RCN=1 floor) is possibly resulting in information loss with respect to shape similarities
>    when many bins are set to 0 or 1."
> …
> "For this phase, add it and wire it so it obeys the current behavior as default. In a future
> Phase, probably the end of Phase 15, we can add a Priority to finish testing this and
> finding the best default cluster conditions for posterior groups in general."

**Scope decisions for this priority:**

1. Flag is added and wired across all three pipelines.
2. Default is `on` (current behavior preserved). Do NOT implement the subtle behavior change
   the user described in concern #1 (post-sum correction vs per-bin floor) — that is
   explicitly deferred to Phase 15.
3. The flag affects both APS score computation (`_locus_metrics()`-analog paths) and any
   shape-based clustering feature computation that currently uses the floored excess.

**Implementation scope:**

1. `onionskin.py:build_parser()` — APS group — add:
   ```
   --aps-area-excess-floor [on|off]
       Whether to floor per-bin area excess at 0 (discarding bins with RCN < 1) when
       computing APS. on (default): preserves current per-bin-floor behavior. off: retains
       signed per-bin contributions; sum-level floor may still apply. May improve early-stage
       cluster separation on some datasets.
       [hmm: 14-aps; growth: 13-aps; rms: 12-aps]
   ```
2. `onionskin_core/aps.py:_locus_metrics()` — wire the flag into the locus-metrics function;
   branch on `floor_enabled`.
3. `onionskin.py` APS call paths for all three pipelines — pass the flag value through.
   Locate the APS invocation for:
   - Growth APS (step-13)
   - RMS APS (step-12)
   - HMM APS (step-14; `run_step14_hmm_aps`)
4. Tests — add a behavior regression confirming the flag switches per-bin floor on/off.
5. Update `KNOWN_ISSUES.md [ISSUE:2026-04-18:1]` — CLI part becomes resolved; remaining
   analytical / experimental work (default flip, shape-clustering implications) moves to a
   new Phase 15 BRAINSTORM entry.

**Append to `PHASE15_BRAINSTORM.md`:**
- Phase 15 follow-up priority: test `--aps-area-excess-floor off` on live datasets; decide
  whether to flip the default.
- Phase 15 follow-up priority: test post-sum-floor behavior (user's concern #1) as an
  alternative wiring; gate on comparative evidence.
- Pre-amplification sample detection / "flat" sample classification remains a late Phase 15
  priority (user said: "Part of that will also be identifying truly 'flat' non-amplification
  samples for 'stage 1', then clustering the rest into automated groups"; "no M-shaped
  posterior amplicons, seeing summit evaluations improve or at least stay the same, no
  regressions").

---

## Priority 14-S23 — Split `--hmm-smooth-halfwidth` into step-5 and step-14 APS flags (Q23 resolved)

**Goal (Q23 resolved):** Do the split now. Wire both flags. Defaults identical. Help
strings cross-reference each other.

**Status: READY.**

**User direction (Q23):**
> "Do this split now and wire it: `--hmm-smooth-halfwidth` and `--hmm-aps-smooth-halfwidth`.
> Both can have the same defaults, but the flexibility is worth having for later testing.
> Make sure the help strings explain what each will apply to. Help strings should recognize
> the existence of the other similar flag, and note that defaults are currently the same."

**Implementation scope:**
1. `onionskin.py:build_parser()` — HMM group:
   - Keep `--hmm-smooth-halfwidth` as step-5 whole-genome bedGraph smoothing (original role,
     unchanged). Help string cross-references `--hmm-aps-smooth-halfwidth` and notes that
     defaults are currently the same.
   - Add `--hmm-aps-smooth-halfwidth` as step-14 APS per-locus smoothing. Default is same
     as `--hmm-smooth-halfwidth` default. If `--hmm-aps-smooth-halfwidth` is explicitly set,
     use it. If unset AND `--hmm-smooth-halfwidth` is explicitly set, fall back to that.
     Otherwise use the numeric default.
   - Step mentions in help: `[hmm: 05-medianSmoothedRCN]` on `--hmm-smooth-halfwidth`;
     `[hmm: 14-aps]` on `--hmm-aps-smooth-halfwidth`.
2. `onionskin.py` step-14 APS call path — read the resolved APS halfwidth via a helper
   mirroring `_resolve_override_value()`. Two-level resolution:
   - explicit `--hmm-aps-smooth-halfwidth` wins;
   - else explicit `--hmm-smooth-halfwidth`;
   - else default.
3. Tests — confirm all three resolution paths.
4. Update `KNOWN_ISSUES.md [ISSUE:2026-04-19:3]` — mark resolved (remove entry; note the
   resolution in CHANGELOG).

---

## Priority 14-S26 — Wire `--rms-shape-score-strict-bic` (and `--rms-shape-score-threshold` if needed) (Q26 resolved)

**Goal (Q26 resolved):** The RMS shape-filter sink exists (`shape_filter_calls()` /
`apply_shape_filter_tsv()` already use `strict_bic` per the Phase 14.2 wiring). Wire
`--rms-shape-score-strict-bic` so it takes effect when a user sets it (overriding the
universal `--shape-score-strict-bic`). Same pattern for `--rms-shape-score-threshold` if not
already wired. `--hmm-shape-score-*` remain placeholders — HMM shape-score wiring moves to
Phase 15.

**Status: READY.**

**User direction (Q26):**
> "If RMS pipeline is already set up to use this, which it is, then we should wire it now.
> That was the intention. Same for HMM pipeline. If we have shape filtering there already,
> then wire it. For HMM, either way, I assume we will also develop this aspect more in
> Phase15. … **Shorter answer based on your options: Wire RMS now, keep HMM placeholder
> (and append this to PHASE15 file where we are collecting HMM development ideas).**"

**Implementation scope:**

1. `onionskin.py` — verify current state:
   - Check that `--rms-shape-score-strict-bic` and `--rms-shape-score-threshold` are
     parser-registered with `default=None` so the `_resolve_override_value()` pattern can
     treat unset as "inherit from universal."
   - Audit the RMS call paths (`_run_rcn_mean_shift_controller`, `_refine_kwargs`, standalone
     RMS block, single-file block) for where `strict_bic` and `min_shape_score` (→
     `shape_score_threshold`) are currently read. Phase 14.2 wired growth; verify RMS
     mirror exists or add it.
   - For each call site that currently reads the universal value, wrap with
     `_resolve_override_value(args, 'rms_shape_score_strict_bic', 'shape_score_strict_bic')`
     and similarly for `rms_shape_score_threshold`.
   - At the final core-call boundary, convert `"on"/"off"` string to bool.
2. Update `--rms-shape-score-strict-bic` help text to reflect it is now wired (remove
   "Reserved" / "placeholder" language).
3. Tests — add a focused regression confirming an explicit `--rms-shape-score-strict-bic on`
   CLI argument reaches the RMS shape-filter sink and behaves differently from default
   `--shape-score-strict-bic off`.
4. `--hmm-shape-score-*` — NO WIRING. Leave placeholder. Update help to say "HMM shape-score
   wiring scheduled for Phase 15 HMM completeness work."
5. `multi-agent/project_context/DECISIONS.md` — add a short entry documenting the
   intentional asymmetry: RMS wired in Phase 14 Supplemental, HMM wired in Phase 15.

**Append to `PHASE15_BRAINSTORM.md`:**
- HMM shape-score filter wiring (`--hmm-shape-score-threshold`,
  `--hmm-shape-score-strict-bic`) — design and implementation.
- Add HMM equivalent of RMS's meta-analysis of amplicon shapes across stages: "we should add
  to HMM what we do for RMS with the meta-analysis of amplicon shapes across stages to get a
  final set of amplicons and collapsed repeats." (User quote from Q26 answer.)
- HMM-specific related concept: detecting state-path growth as a shape/credibility signal.

---

## Note — Findings 13, 14, 15, 16 routing

- **Finding 13** (HMM shape-score wiring plan): Phase 15 item. Appended to
  `PHASE15_BRAINSTORM.md` via 14-S26.
- **Finding 14** (`--rms-shape-score-strict-bic` partial wiring): resolved by new
  **Priority 14-S26** above.
- **Finding 15** (`-rcn-` prefix generalization): resolved by new **Priority 14-S25** above
  (audit-only; user-decided next step).
- **Finding 16** (obsolete BRAINSTORM.md entries): fold into 14-S16 closeout doc sweep.

---

## Phase 2 — Audit-only deliverables

*Implementation order:* **14-S12 → 14-S25 → 14-S14 → 14-S15 → 14-S21** (any order
works — they produce independent tracking docs / `PHASE15_BRAINSTORM.md`
appendices with no code-path dependencies). 14-S12 and 14-S25 already have their
audit tables produced (2026-04-23); remaining Role 2 work is minimal or folded
into later priorities.

## Priority 14-S12 — RCN-profile flag survey: identify remaining Universal promotion candidates

**Goal:** Survey the growth pipeline flag group for flags that operate on RCN profiles (not on
the growth evidence track itself) and are therefore candidates for Universal promotion in a
current or near-future wave.

**Status: READY (investigation priority — produces a report, not implementation).**

**User direction (from feedback):** "Anything in the growth pipeline that is concerned with RCN
profiles (as opposed to growth track profiles) should be considered universal. We should look
for this now so we can see what we want to pull out of growth later to apply to RMS and/or HMM
pipelines."

**Already promoted:**
- `--peak-summary`, `--peak-quantile`, `--peak-topk` → Universal (14-S2) — RCN profile summary
- `--asym-tri-model-*` → Universal (14-S3) — operates on stage-median RCN profiles for timing

**Flags to survey (growth group, remaining after 14-S1/S2/S3):**

| Flag | Question |
|---|---|
| `--growth-refine-halfwidth` | Operates on RCN profiles for origin refinement — Universal candidate? |
| `--growth-refine-smooth` | Same step — Universal candidate? |
| `--growth-stage-median-resolution` | Controls which RCN resolution is used — Q6 decided growth-only for now; revisit? |
| `--growth-stage-weight-mode` | Weights stages by replicate count — applies to RCN-profile computations too |
| `--growth-fit-method` | Pure growth-track fitting — likely NOT a Universal candidate |
| `--growth-ensemble-methods` | Same — NOT a Universal candidate |

**Status (Q16 + Q28 resolved):** Survey produced and triaged. User directed that all
"promote next supplemental" items are promoted INTO the current supplemental (there is
no next supplemental; Phase 15 is HMM development and user wants CLI organizational work
done before HMM work begins). Approved promotions become concrete mini-priorities inside
this supplemental — see **Priority 14-S27** below.

**Survey outcomes:**
- `--growth-fit-method`, `--growth-ensemble-methods`: keep growth-only.
- `--growth-stage-weight-mode`: **promote to Universal** (new priority 14-S27).
- `--growth-refine-halfwidth`, `--growth-refine-smooth`: **promote to Universal**
  (new priority 14-S27), with summit/origin terminology resolution (14-S28).
- `--growth-rcn-smooth-halfwidth`: **promote to Universal** (new priority 14-S27).
- `--growth-stage-median-resolution`: keep growth-only (per Q6 earlier decision).
- `--growth-z-thresh`, `--growth-halfwidths`, `--growth-peak-search`,
  `--growth-window` (→ `--growth-scan-halfwidth`): keep pipeline-specific parallel-naming
  pattern; detection-flag halfwidth naming review is separate and proposed as
  **Priority 14-S29**.
- `--growth-shape-filter`: user raised a semantics/default question — separate
  **Priority 14-S30**.
- Remaining growth flags: already in correct state.

**Where the survey lives:** `PHASE14_SUPPLEMENTAL-FEEDBACK.md` end-of-file
`## 14-S12 Survey` section (produced 2026-04-23).

---

## Priority 14-S25 — `-rcn-` prefix convention audit + help-string precision updates (Q25 + Q29 resolved)

**Goal (Q25 resolved):** Produce a 4-column audit table covering every flag that operates
on RCN profiles (not on growth evidence tracks or unrelated data). User reviews table and
decides whether to rename any to carry `-rcn-` in the name. No renames in this priority.

**Status: READY.**

**User direction (Q25):**
> "This would need a full audit with a table of each flag, what group it currently sits
> in, what the suggested rename would be, and what the help text currently says (so 4
> columns). I can then make decisions on it."

**Audit table columns:**
| Flag | Current parser group | Current help text (excerpt) | Suggested rename (if any) |

**Scope:** Audit all flags in `onionskin.py:build_parser()` that operate on RCN profiles.
Candidates to review:
- Universal / Shape scoring / Overlap / APS / Timing / Asymmetric Triangle Model groups:
  any flag whose help describes operating on RCN bedGraphs, stage-median RCN profiles, or
  per-sample RCN tracks.
- HMM group: smoothing flags over RCN tracks (`--hmm-smooth-halfwidth`,
  `--hmm-trim-halfwidth`, future `--hmm-aps-smooth-halfwidth`).
- Growth group: flags operating on RCN profiles (e.g., `--growth-refine-halfwidth`,
  `--growth-refine-smooth`, `--growth-rcn-smooth-halfwidth` — already has `-rcn-`).
- RMS group: equivalents (e.g., `--rms-trend`, `--rms-smooth`).
- Asymmetric Triangle Model group: all three flags operate on stage-median RCN profiles.

**Deliverable (audit):** Appended to `PHASE14_SUPPLEMENTAL-FEEDBACK.md`
`## 14-S25 Audit — -rcn- prefix convention candidates` (produced 2026-04-23).

**User decisions (Q29 resolved):**
- Approved renames:
  - `--growth-refine-halfwidth` → see 14-S27/14-S28 for final name
    (`--refine-summit-halfwidth` or similar; summit/origin terminology decision applies)
  - `--growth-refine-smooth` → same treatment via 14-S27/14-S28
- All other audit candidates: **no rename** (narrow `-rcn-` convention stands; only use
  when a flag's name would otherwise be ambiguous with a non-RCN surface in the same
  group or at the same prefix).
- **Help-string direction:** user approves the narrow rename decisions but wants help
  strings for RCN-profile-operating flags updated to more specifically say they operate
  on RCN signal, in simpler plain language. Example rewrite the user provided:
  > Old: "Method to summarize per-call peak signal height across replicate stage medians
  > for the peak profile output."
  > New: "Method to summarize RCN signal height for each peak call across stages for the
  > peak summary profile output in <step-directory-location>. Each stage is represented
  > as the per-bin median across its replicates."
- **Peak vs amplicon terminology:** user approves keeping "peak" in flag names, but wants
  help strings to clarify that "peak call" = the entire breadth and shape of the
  mountain-shaped amplicon (the amplicon interval). This applies more broadly than just
  RCN-flag help — bundle into 14-S8 full help-quality pass.

**Implementation scope (Phase 14 Supplemental Role 2):**
1. Produce the audit table — DONE.
2. No flag renames in this priority directly — the two approved renames are
   subsumed by Priority 14-S27 (summit-refinement Universal promotion) where they are
   renamed with the summit/origin terminology decision.
3. Update help strings per user's direction for RCN-profile-operating flags: more
   specific, simpler, explicit "RCN signal" language where currently vague. This is a
   subset of the 14-S8 help-quality pass — the RCN-specific wording becomes part of the
   S8 quality bar.
4. Codify the narrow `-rcn-` convention in `AGENT_CONVENTIONS.md` (14-S16 closeout doc
   sweep will pick this up): "Use `-rcn-` in a flag name only when the name would
   otherwise be ambiguous with a non-RCN surface in the same group or at the same prefix."

---

## Priority 14-S14 — Internal-code "Stage-1"/"Stage-2" detection-pass terminology: audit-only (Q13 resolved)

**Goal (narrowed per Q13):** Produce an audit-only classification table of every
"Stage-1"/"Stage-2"/`stage_1`/`stage_2`/"stage-1"/"stage-2" token in the codebase.
Classify each as `detection-pass` / `biological-stage` / `ambiguous`. **No renames in this
priority.** Save findings for later decision.

**Status: READY** (simplified from blocked-on-Q13).

**User direction (Q13):**
> "Let's leave this at audit-only for now. Put the audit findings and output in KNOWN_ISSUES
> for later, or in its own file under `multi-agent/tracking/`."

**Critical invariant:** Biological stage terminology (e.g., "posterior stage 1 samples",
`stage=1` in replicate stages, `stage_medians`, `_stage_weight_mode`, etc.) is NOT renamed
— it is correctly biological. Audit classifies; does not mutate.

**Deliverable:** New file
`multi-agent/tracking/STAGE-1-2-DETECTION-PASS-TERMINOLOGY-AUDIT.md` with a classified table.
Columns:
- File:line
- Context (one line of source or help-string excerpt)
- Classification: `detection-pass` / `biological-stage` / `ambiguous`
- Suggested rename (if detection-pass): e.g., `stage1_best_peak` → `firstpass_best_peak`
- Justification for classification

Audit covers:
- `onionskin.py` (help strings, log prefixes, docstrings, comments, variable names)
- `onionskin_core/*.py` and `onionskin_core/engines/*.py` (docstrings, comments, variable
  and parameter names)
- `tests/` (test names, fixture names, docstring references)
- `multi-agent/full_instructions/*.md` and `multi-agent/plans/*.md` where the term appears
  in live (non-archived) content

**Implementation scope:** Produce and save the audit file. No code changes. No renames.
Tracking file becomes the canonical decision surface for a future renaming phase.

---

## Priority 14-S15 — HMM PuffStep-synonym audit-only pass (Q15 resolved)

**Goal (narrowed per Q15):** Help text stays as-is for now. Produce an audit table of every
HMM flag with a PuffStep synonym. Append findings to
`multi-agent/plans/next/PHASE15_BRAINSTORM.md` for use during Phase 15 HMM work. **No code
or help-text changes in this priority.**

**Status: READY** (simplified from blocked-on-Q15).

**User direction (Q15):**
> "Let's leave the help text as is with regard to PuffStep mentions for now. They can be
> audited to see if changes are needed. The audit findings and output can be added to
> `multi-agent/plans/next/PHASE15_BRAINSTORM.md`. This will be for updating the HMM
> pipeline."

**Flags to audit:**
- `--hmm-expected-background-length` / `--hmm-expected-special-length`
- `--hmm-background-idx` / `--hmm-special-idx`
- `--hmm-init-background` / `--hmm-init-special`
- `--hmm-leave-background-state` / `--hmm-leave-special-state`
- `--hmm-leave-amp-step` / `--hmm-leave-other`
- Any other pair discovered in `onionskin.py:build_parser()` HMM group.

**Per-pair audit columns:**
- Canonical flag name
- PuffStep synonym
- Current help string mentions synonym? (yes / no / partial)
- Is synonym still registered as argparse alias? (yes / no)
- Suggested follow-up for Phase 15 (e.g., "add synonym mention"; "leave as-is"; "tighten
  wording")

**Deliverable:** Append audit findings as a new section to
`multi-agent/plans/next/PHASE15_BRAINSTORM.md` under a heading like `## 14-S15 audit findings
— HMM PuffStep synonym coverage`. Phase 15 Role 2 picks this up as part of HMM pipeline
development.

**Implementation scope:** Produce and save the audit file appendix. No code changes.

---

## Priority 14-S21 — `--hmm-0-based-statepath` — DEFERRED to Phase 15 with audit appended (Q21 resolved)

**Goal (Q21 resolved):** Deferred to Phase 15 HMM completeness work. This supplemental
priority produces an audit of what would need to change when the flag lands, plus captures
the user's additional design detail about adaptive defaults for companion flags. Audit
findings appended to `multi-agent/plans/next/PHASE15_BRAINSTORM.md`.

**Status: READY (audit-and-defer).**

**User direction (Q21):**
> "I believe we should add the `--hmm-0-based-statepath` flag. However, I believe when
> `--hmm-0-based-statepath` is used, it should change the defaults of `--hmm-thresh-state`
> and `--hmm-max-state-thres` to 0 and -1 (None) or something. So there needs to be an
> argument catch that sees this and updates the defaults if those flags are not specifically
> set by the user. If those flags are specifically set by the user, then just use what they
> give. And their docstrings should be updated to reflect their adaptability to using
> `--hmm-0-based-statepath`. This is to be done, and can be done in this phase or the entire
> idea including my feedback can be appended to the end of
> `multi-agent/plans/next/PHASE15_BRAINSTORM.md` where we are collecting ideas for HMM
> development. So it will be either done right now or very soon. if you defer it to
> `multi-agent/plans/next/PHASE15_BRAINSTORM.md`, which is fine, then also audit what would
> need to be done, and report audit findings there as well to save us work in the future."

**Implementation scope in this priority (audit-only, no code changes):**

1. Audit `onionskin.py:build_parser()` HMM group — identify `--hmm-thresh-state`,
   `--hmm-max-state-thres`, and any other state-index-aware flags. Record current defaults,
   current help strings, and what those defaults would need to become under 0-based
   indexing.
2. Audit HMM engine code paths that emit state-path values
   (`onionskin_core/engines/hmm_engine.py`, `onionskin_core/hmm_core.py`,
   `onionskin_core/hmm_summits.py`, `onionskin_core/hmm_metrics.py`, and any step that
   writes state paths to disk). Record every call-site where a `-1` shift would need to
   apply if the flag is active, and every downstream consumer that reads state-path values.
3. Audit tests (`tests/test_pipeline.py`) for state-value assertions that would need to
   gate on the flag.
4. Audit docs (`PIPELINE_SPEC.md`, `ONIONSKIN_FULL_HANDOFF.md`) for state-indexing
   descriptions that would need a conditional note.
5. Capture the adaptive-default logic the user described: when `--hmm-0-based-statepath` is
   set AND the user has NOT explicitly set `--hmm-thresh-state` or `--hmm-max-state-thres`,
   shift their defaults by -1. Require an `argparse`-post-parse catch (e.g.,
   `argparse.SUPPRESS` on the two flags so the catch can distinguish unset vs user-set).

**Deliverables:**
- Audit findings appended to `multi-agent/plans/next/PHASE15_BRAINSTORM.md` under a new
  heading like `## 14-S21 audit — --hmm-0-based-statepath impact + adaptive-default design`.
- No changes to `onionskin.py` or anything else in this priority.
- The `BRAINSTORM.md [2026-04-18] --hmm-0-based-statepath future CLI flag` entry stays as
  the original design doc; Phase 15 implementation phase will supersede it.

---

## Phase 3 — Help-string polish (depends on Phase 1 final CLI surface)

*Implementation order:* **14-S6 → 14-S7 → 14-S9 → 14-S13 → 14-S19 → 14-S20 →
14-S8.**

*Rationale:* 14-S8 is the integrating full-parser help-quality pass and runs
LAST so it can absorb the help-string direction from 14-S25, cover every
flag/group added or renamed in Phase 1, and verify against final multi-pipeline
step mentions. 14-S7 runs after 14-S26 (Phase 1) because S26 moves the RMS
shape-score pair out of placeholder status. 14-S20 runs after 14-S22 (Phase 1)
because `--aps-area-excess-floor` needs to exist before the APS group is
re-framed.

## Priority 14-S6 — `--aps-rank-by` help expansion

**Goal:** Expand `--aps-rank-by` help text to explain what ranking means and note that
additional choices are planned. Do NOT hide the flag.

**Status: READY.**

**User decision (from feedback):** "Do not hide `--aps-rank-by`. Expand its help text. Put a
note in KNOWN_ISSUES that this likely was intended to have more options (perhaps: summit, width,
shape)."

**Implementation scope:**

1. `onionskin.py:build_parser()` — APS group:
   - Expand `--aps-rank-by` help: explain that `area` ranks calls by total area excess
     (the current default and only active choice); note that `summit`, `width`, and `shape`
     are planned for a future release
2. `tests/test_pipeline.py` — no new help regression needed beyond confirming help renders

*Note: a `KNOWN_ISSUES.md` entry for adding future choices (`summit`, `width`, `shape`) should
be kept; this priority only addresses the help string.*

---

## Priority 14-S7 — Placeholder flag help: clarify wiring status

**Goal:** For each placeholder flag currently in the parser, update its help string to honestly
describe its current wiring status: true no-op, partial wiring hook, or inheritance placeholder.
Do NOT hide these flags.

**Status: READY.**

**User decision (from feedback):** "Do not remove or hide these. It is okay to 'make the help
more explicit that they are reserved parser surfaces with no current behavioral effect in the
present release' and to 'explain whether they are true no-ops, partial wiring hooks, or
inheritance placeholders'."

**Flags in scope and their current wiring status (verify against live code):**

| Flag | Status at SPEC writing | Post-v0.14.70 live state (updated at cycle 14S.3a closeout v0.14.71) |
|---|---|---|
| `--hmm-shape-score-threshold` | Inheritance placeholder | **Still placeholder** — HMM shape-score wiring deferred to Phase 15 per Q26. Help updated to "Inheritance placeholder" in cycle 14S.3a. |
| `--hmm-shape-score-strict-bic` | Inheritance placeholder | **Still placeholder** — same. Help updated in cycle 14S.3a. |
| `--rms-shape-score-threshold` | Inheritance placeholder | **WIRED** in Priority 14-S26 v0.14.70. Help reflects wired state ("Inherits from --shape-score-threshold when unset."). |
| `--rms-shape-score-strict-bic` | Inheritance placeholder | **WIRED** in Priority 14-S26 v0.14.70. Help reflects wired state. |
| `--rms-bootstrap-summits` (renamed from `--rms-bootstrap-origins` in 14-S28 v0.14.70) | True no-op (no bootstrap impl) at SPEC write time | **WORKING OVERRIDE** — RMS bootstrap implemented at `rcn_mean_shift_helpers.py:_bootstrap_origin()` (line 148); override resolves via `_effective_rms_bootstrap_summits()`. Help is honest: "Unset inherits from --bootstrap-summits." |
| `--growth-bootstrap-summits` (renamed from `--growth-bootstrap-origins` in 14-S28 v0.14.70) | True no-op (no bootstrap impl) at SPEC write time | **WORKING OVERRIDE** — Growth bootstrap implemented in `growth_model_engine.py:refine_origin_for_call()`. Help is honest. |

**Implementation scope:**

1. `onionskin.py:build_parser()` — for the 4 flags that remain placeholders after 14-S26:
   - Verify wiring status against live code (before writing help)
   - For `--hmm-shape-score-threshold` / `--hmm-shape-score-strict-bic`: note
     "HMM shape-score wiring scheduled for Phase 15 HMM completeness; currently parser-only."
   - For `--rms-bootstrap-origins` / `--growth-bootstrap-origins`: note
     "Bootstrap origin confidence interval is not yet implemented. Reserved parser surface."
2. No `_DEPRECATED_FLAGS` changes
3. Test changes: optional help regression confirming the new wording
4. Coordinate with 14-S26: the RMS pair moves out of the placeholder list once wired

---

## Priority 14-S9 — HMM help improvements

**Goal:** Improve help strings for `--hmm-decode-path`, consolidate PuffStep compatibility
notes to the HMM group header where currently repeated per-flag, and optionally add practical
decision context to `--hmm-training`.

**Status: READY.**

**User decisions (from feedback):**
- `--hmm-decode-path`: add user-facing decision context. User note: "viterbi is almost always
  preferred at the scale the HMM is working on since it is much faster and gives accurate
  results." Add that to the help string.
- `--hmm-training`: "It is okay that `--hmm-training` overexplains algorithms. Do not simplify.
  It would be fine to add more user-facing decision context though, only if we have something
  useful to say there."
- PuffStep compatibility notes: "I somewhat agree that a section-level compatibility note would
  likely be cleaner than repeating 'Port of PuffStep ...' inside many individual help strings,
  except where it is specific to the helpstring."

**Implementation scope:**

1. `onionskin.py:build_parser()` — HMM group:
   - `--hmm-decode-path` help: add "viterbi is strongly recommended at typical amplification
     detection scales — faster and equivalent accuracy; posterior is available for research use"
   - HMM group description/header: add a single PuffStep compatibility note (e.g., "HMM pipeline
     (ported from PuffStep). See individual flags for PuffStep-specific synonyms.")
   - Remove redundant "Port of PuffStep" text from individual flag help strings where the per-flag
     note adds nothing beyond the group-level note; keep it where the flag is PuffStep-specific
   - `--hmm-training`: add practical decision context if something genuinely useful can be said;
     do not simplify existing content
2. `tests/test_pipeline.py` — help regression: confirm `--hmm-decode-path` contains "viterbi"
   recommendation language

### Expansion note (recon audit 2026-04-22) — per-flag PuffStep synonym preservation

Per user feedback: "I do not agree that we should retire 'special' terminology for HMM. It is
a remnant synonym from PuffStep, which I believe we mention in the help text already. If not,
mention it." 14-S9 already consolidates redundant PuffStep notes at the group level, but it
does NOT explicitly require verifying that every PuffStep synonym continues to carry its
per-flag identification.

**Additional requirement (conditional on Q15 resolution):** For each HMM flag with a PuffStep
synonym (e.g., `--hmm-special-idx`, `--hmm-leave-special-state`, `--hmm-init-special`,
`--hmm-expected-special-length`, `--hmm-leave-other`), the help text must still say "Also
accepted as `--hmm-*` (PuffStep synonym)." Remove redundant generic PuffStep prose only where
the per-flag note adds nothing beyond the new group-level note — but keep the synonym
identification.

*Note:* If Q15 is answered "separate priority", this requirement becomes **14-S15** instead
of a 14-S9 sub-task.

---

## Priority 14-S13 — Timing parser group: step-mention pass (Q12 resolved)

**Goal (narrowed per Q12):** Apply multi-pipeline step mentions to Timing flags in the
`[growth: 10-timing; rms: planned; hmm: planned]` format. Do NOT add `--growth-*` prefixes
(user does not currently suspect a need for different values across pipelines). Do NOT
re-structure Timing as a Universal section member. Defer the `--onset-*` vs `--timing-*`
prefix inconsistency to `KNOWN_ISSUES.md`.

**Status: READY** (simplified from blocked-on-Q12).

**User direction (Q12):**
> "Leave as-is. Do not add `--growth-*` stuff at this time since I do not currently suspect
> a need for different values for different pipelines. Yes write step mentions in 'pipeline
> step' format discussed elsewhere. Defer adding `--timing-*` prefixes for now. Can be a
> KNOWN_ISSUE."

**Implementation scope:**

1. `onionskin.py:build_parser()` — Timing group flags:
   - `--onset-rcn-threshold`, `--onset-span-rcn-threshold`, `--onset-method`,
     `--onset-z-threshold`, `--timing-onset-quantile`, `--timing-exclude-loci`,
     `--near-gap-bp`
   - Add `[growth: 10-timing; rms: planned; hmm: planned]` step mention to each help string
     (either on each flag or on the group description — whichever reads cleaner).
   - No flag renames. No new flags.
2. `multi-agent/KNOWN_ISSUES.md` — add a new low-priority entry noting the `--onset-*` vs
   `--timing-*` prefix inconsistency for a future harmonization pass.
3. Tests — optional help regression confirming representative step-mention strings appear.

**Scope boundary:** No parser restructure, no overrides, no renames. This priority is a
single-group help-text pass plus one KNOWN_ISSUES entry.

---

## Priority 14-S19 — `--pipelines` help: quick-start + verbose; fix `all` semantics (Q19 resolved)

**Goal (Q19 resolved):** Keep the existing short help as a "quick start" block at the top of
the `--pipelines` help string. Append the user-written verbose educational content after it
as a "Context" block. Fix the factual contradiction so `all` is described as running all
three pipelines (HMM included).

**Status: READY** (simplified from blocked-on-Q19).

**User direction (Q19):**
> "Add my help text below what is there. What is there might need some fixing for accuracy,
> but serves as a nice 'Quick start' for the flag. What I wrote could come after to give
> more context. … Factual contradiction: fix help text. … all means all: fix help text.
> Restore user's verbose --pipelines help text; resolve the all semantics contradiction."

**Multistage alias (Finding 12, Q19 secondary):** **REJECTED.** `--pipelines multistage`
will NOT be accepted as an alias for `--pipelines growth`. User direction: "See reasoning
in my above feedback section."

**Target help-text structure (two-block layout):**

1. **Quick start** (corrected version of current short help):
   > Comma-separated list of pipelines to run, or `all`. Available: growth, rms (or
   > rcn-mean-shift), hmm. Examples: `--pipelines growth,rms`  `--pipelines hmm`
   > `--pipelines all`. Default: `all` = growth + rms + hmm.

2. **Context** (user's verbose text from PHASE14_FEEDBACK Q7, verbatim with canonical
   tokens):
   > `all = hmm,growth,rms`. `hmm` = hidden markov model. `growth` = multiple stage growth
   > modeling. `rms` = RCN mean shift. RCN = relative copy number. HMM and RMS can handle
   > a single file, single stage with multiple replicates, a single stage versus a
   > reference stage (each with 1 or more replicates), and multiple stages with or without
   > a reference stage (`--norm-mode` set to `chrom-median` or `ref-stage`, respectively).
   > The growth modeling pipeline is designed for 3 or more stages without a reference
   > stage (`--norm-mode chrom-median`) or 4 or more stages when one is a reference stage
   > (`--norm-mode ref-stage`). To use different normalization modes across pipelines, see
   > their pipeline-specific `--norm-mode` flags.

The "HMM is opt-in" sentence must be removed — it contradicts
`_resolve_requested_effective_pipelines()` behavior where `all` expands to all three.

**Implementation scope:**
1. `onionskin.py:build_parser()` — rewrite the `--pipelines` help string with the two-block
   structure above.
2. `tests/test_pipeline.py` — update / add help regression: confirm the help contains both
   the quick-start sentence and the "multiple stages with or without a reference stage"
   phrase from the context block.
3. `_normalize_pipeline_token()` — leave unchanged. Do NOT add `multistage` as an alias.
4. No runtime behavior changes (runtime already correct; only help text was misleading).

---

## Priority 14-S20 — APS group re-framed as Universal-in-spirit; `--dedup-dist` verification (Q20 resolved)

**Goal (Q20 resolved):** APS stays in its own parser group (do NOT merge into the "Universal"
parser section) but is re-framed as Universal-in-spirit: it applies to all pipelines. Audit
each APS flag for runtime-agnostic vs help-string-narrow behavior. Verify `--dedup-dist`
semantics across pipelines. Produce a report; update help strings where they falsely claim
pipeline-specific scope.

**Status: READY** (simplified from blocked-on-Q20).

**Terminology clarification (from user Q20):**
> "'promotion to universal' needs the right scope here. The term 'universal' might be
> overloaded since there is a 'Universal' section and a 'universal' concept in the overlap,
> timing, and aps sections. For APS, it is the concept of being universal, not to be moved
> to the section called 'universal'. ... APS is universal as in, it will be in all
> pipelines. if this means the section needs to be re-framed, then yes. Do not merge it
> with the 'universal' section though."

**Implementation scope:**

### Part A — APS audit + help-string re-framing

1. Identify every flag in the APS parser group. Expected list:
   - `--aps-feature`, `--aps-feature-scale`, `--aps-feature-clip`, `--aps-feature-log-offset`
   - `--aps-cluster-method`, `--aps-cluster-k`
   - `--aps-singleton-guard`, `--aps-rank-by`
   - any additional APS flags discovered in the live parser.

2. For each flag, determine via code inspection:
   - Which pipelines' APS call paths the flag currently reaches (HMM step-14 APS, Growth
     APS, RMS APS).
   - Whether the help string claims pipeline-specific scope incorrectly.

3. Update the APS group header and individual flag help strings to reflect the
   Universal-in-spirit framing. Add multi-pipeline step mentions using the established
   convention: `[hmm: 14-aps; growth: 13-aps; rms: 12-aps]`.

4. Do NOT move these flags into the `Universal` parser section. APS remains its own group.

5. No flag renames in this priority. No new per-pipeline overrides unless runtime behavior
   demonstrably differs across pipelines.

### Part B — `--dedup-dist` verification (Finding 7)

1. Verify which pipelines currently use `--dedup-dist` at runtime. User-voice expectation:
   applies to RMS and Growth (both use mean-shift for calls).

2. Update `--dedup-dist` help text to explicitly state which pipelines and which steps it
   affects. Remove vague "Applied before overlap resolution and after detection." phrasing.

3. If pipelines implement dedup via different code paths (user-flagged concern: "we have an
   ongoing attempt to unify code and make it non-redundant"), **surface that finding for
   Phase 15 code-unification work** — do not attempt to unify in this priority. Report
   findings to `multi-agent/plans/next/PHASE15_BRAINSTORM.md` under a new subheading.

4. `--dedup-dist` stays in the Overlap group. No move.

### Deliverables

- Updated APS group help strings in `onionskin.py:build_parser()`.
- Updated `--dedup-dist` help string in `onionskin.py:build_parser()`.
- APS audit findings appended to `PHASE14_SUPPLEMENTAL-FEEDBACK.md` end-of-file for user
  review.
- Cross-pipeline dedup implementation differences (if any) appended to
  `PHASE15_BRAINSTORM.md`.
- Tests: help regression confirming key step-mention strings appear.

---

## Priority 14-S8 — Full help-string quality pass: ALL parser groups

**Goal:** Two-part improvement across every flag group in `onionskin.py`:

1. **Step mentions** — add `[pipeline: XX-step-name]` context to every flag group, matching
   the pattern established for Growth flags. Every user should be able to read the help output
   and know which output directory step each flag affects.

2. **Content quality** — while making the step-mention pass, any help string that is terse,
   incomplete, doesn't explain its default, or doesn't say when/why a user would actually
   change the flag must be rewritten. Adding a step tag to a weak string is not sufficient;
   the string itself must be good.

This is a full help-quality pass, not a step-mention append.

**Status: READY.**

**Step-mention conventions (from feedback):**
- Use full output directory step name (e.g., `[hmm: 02-stage-medians]`), not shorthand
- For Universal flags: list all pipelines, e.g., `[hmm: 14-aps; growth: 13-aps; rms: 12-aps]`
- Section headers may carry the step mention when all flags in the group share a step,
  to avoid repetition in each individual flag's help
- Style anchors for good existing help strings: `--timing-onset-quantile`, `--aps-cluster-k`,
  `--shape-score-strict-bic` — these explain defaults, intended use, and tradeoffs; use them
  as the quality bar every rewritten string should meet

**Help-string quality bar — a flag's help must:**
- Say what the flag controls (not just restate the flag name)
- State the default value and what happens when left at default
- Explain when and why a user would change it from the default
- Include step mention(s) in brackets
- For flags with choices: describe what each choice does in plain terms

**Target groups (priority order):**

1. **Timing group** — step mentions `[growth: 10-timing]`; expand any terse timing flags
2. **Overlap/dedup group** — step mentions for overlap-resolution and deduplication steps;
   expand any terse overlap flags
3. **HMM modeling and decoding flags** — map each flag to its HMM step (01–16); rewrite
   terse or jargon-heavy strings (covered jointly with 14-S9 for `--hmm-decode-path` and
   PuffStep consolidation)
4. **RMS/HMM override groups** — step mentions per pipeline; rewrite terse overrides
5. **Universal group** — multi-pipeline step mentions; rewrite any strings that don't explain
   the universal vs. pipeline-override relationship
6. **APS group** — step mentions `[hmm: 14-aps; growth: 13-aps; rms: 12-aps]`; expand terse
   APS flags
7. **Shape scoring group** — step mentions; expand placeholder-adjacent strings (coordinated
   with 14-S7)
8. **Single-mode group** — step mentions where applicable; expand terse single-mode flags
9. **Growth model group** — verify the already-improved Growth strings meet the quality bar;
   patch any gaps found

**Implementation scope:**

1. `onionskin.py:build_parser()` — for every group above:
   - Verify owning pipeline steps using `onionskin_core/output_layout.py` step dicts
   - Add step mentions to flag help strings or group descriptions/headers
   - Rewrite any help string that does not meet the quality bar above
2. `tests/test_pipeline.py` — help regression: confirm representative step-mention strings
   appear in rendered help (spot-check at least one flag per group)
3. `PIPELINE_SPEC.md` — no structural changes needed; this is an `onionskin.py` pass only

**Reference step directories:**
- HMM: `build_hmm_steps()` in `output_layout.py`
- Growth: `build_growth_steps()` in `output_layout.py`
- RMS: `build_rcn_mean_shift_steps()` in `output_layout.py`

### Expansion note (recon audit 2026-04-22) — multi-pipeline step mentions on shared flags

Per user feedback and the recon audit, Universal, Shape scoring, Overlap/dedup, APS, and
Asymmetric Triangle Model groups hold flags that affect multiple pipelines. Their help
strings must carry **multi-pipeline step mentions** (example: `[hmm: 14-aps; growth: 13-aps;
rms: 12-aps]`), not a single-pipeline step mention. Where a flag is currently Universal-
shaped but only one pipeline implements it, the help must say `[growth: 10-timing; rms:
planned; hmm: planned]` to surface future scope explicitly. This is the same style already
used by the 14-S3 Asymmetric Triangle Model group help strings — apply it consistently
everywhere a shared flag lives.

### Expansion note (2026-04-23, Q28/Q29) — plain-language help + peak/amplicon clarification

Additional quality-bar items for the help-string pass, per user direction in Q28/Q29:

**Plain-language rewrite bar.** Where current help uses dense jargon ("per-call peak
signal height across replicate stage medians"), rewrite in simpler plain language without
losing precision. Example from Q29:

> Old: "Method to summarize per-call peak signal height across replicate stage medians
> for the peak profile output."
> New: "Method to summarize RCN signal height for each peak call across stages for the
> peak summary profile output. Each stage is represented as the per-bin median across its
> replicates."

**"Peak" vs "amplicon" clarification.** "Peak" stays as the canonical term in flag names
(user confirmed in Q29). But help strings should, where relevant, clarify that in this
codebase "peak call" refers to the entire breadth and shape of the mountain-shaped
amplicon — the amplicon interval, not a ChIP-seq-style narrow peak. Add this clarification
to help on at least: `--peak-summary`, `--peak-quantile`, `--peak-topk`, and
`--growth-peak-search` (and its RMS counterpart).

**"Summit" as proxy for origin.** Per 14-S28, help strings for summit-refinement flags
(the new Universal flags from 14-S27) should include a one-sentence clarification that
summit is a proxy for the re-replication origin / initiation zone.

**"RCN signal" explicitness.** Per Q29, where a flag operates on RCN signal, the help
string should say so explicitly rather than using vague language like "signal."

**Target groups with multi-pipeline step mentions required:**
- Universal group — per-pipeline step for each flag that actually touches a step; explicit
  "planned" markers where future-intended
- Shape scoring group — all three pipelines once shape scoring is fully wired
- Overlap/Dedup group — all three pipelines
- APS group — all three pipelines (APS steps differ: hmm=14, growth=13, rms=12)
- Asymmetric Triangle Model group — already done; verify remains consistent

---

## Phase 4 — Structural polish + tooling

*Implementation order:* **14-S11 → 14-S24.**

*Rationale:* reorder-within-groups (14-S11) needs the final flag set from Phase 1
to be correctly ordered. The validator script (14-S24) reads `_DEPRECATED_FLAGS`;
its test fixtures benefit from the final deprecation set.

## Priority 14-S11 — CLI flag ordering: flags follow pipeline execution order

**Goal:** Within each pipeline flag group in `build_parser()`, reorder `add_argument` calls
so that flags appear in the same order as the pipeline steps they affect. A user scanning the
help output should see flags in the order they would encounter them during a run.

**Status: READY.**

**User direction (from feedback):** "Order of CLI flags should follow order in which they are
used across the pipeline. Let's make sure they are being presented in this logical ordering."

**Implementation scope:**

1. `onionskin.py:build_parser()` — for each pipeline group (Growth, RMS, HMM) and Universal:
   - Map each flag to its owning step number using the `build_*_steps()` output layout dicts
   - Reorder `add_argument` calls within each group to match step order
   - Where a flag affects multiple steps, place it at the first step it affects
2. No behavior change; pure parser organization pass
3. `tests/test_pipeline.py` — no new assertions needed (help regression already covers presence;
   ordering is a style convention not verified by automated tests)
4. After reordering, perform a visual review of `python onionskin.py -h` output to confirm
   the ordering reads naturally

---

## Priority 14-S24 — Deprecated-flag validator: `scripts/validate_onionskin_flags.py` + `make validate-flags` (Q27 resolved)

**Goal (Q27 resolved):** Build a developer-facing deprecated-flag scanner as a standalone
script (not an onionskin subcommand, not a top-level flag). Wrap with a Makefile target for
discoverability. No dispatch wrapper in `onionskin.py`.

**Status: READY.**

**User direction (Q27):**
> "After talking with Claude (opus), the design we liked was not a flag but a separate
> script: (1) `scripts/validate_onionskin_flags.py` — standalone script with its own
> argparse, imports `_DEPRECATED_FLAGS` from onionskin (or from a small shared module if
> you want to decouple). (2) `make validate-flags` Makefile target → one-word developer
> entry point; wraps `python scripts/validate_onionskin_flags.py "$@"`. This solves the
> discoverability concern without adding Python dispatch complexity. (3) Mention the
> script in `README.md` and `ONIONSKIN_FULL_HANDOFF.md` under developer tooling. (4) No
> dispatch wrapper in `onionskin.py`."

**Implementation scope:**

1. **`scripts/validate_onionskin_flags.py`** — standalone Python script with its own
   argparse. Accepts one or more paths (positional args). Paths may be `.sh`, `.py`, or
   any text file. Scans each for occurrences of any key in the onionskin
   `_DEPRECATED_FLAGS` dict and reports `file:line: <flag> → <redirect>` for each hit.
   Exit code 0 if no hits, 1 if any hits found (makes it usable as a CI gate).
   - Import `_DEPRECATED_FLAGS` from `onionskin` (or from a tiny
     `onionskin_core/deprecated_flags.py` shared module if decoupling feels cleaner; that
     is a minor implementation decision — either works, pick the lighter option).
   - Script has its own `-h` help text describing its purpose.

2. **`Makefile` target `validate-flags`** — wraps
   `python scripts/validate_onionskin_flags.py "$@"`. Accepts additional arguments via
   `$(filter-out $@,$(MAKECMDGOALS))` or equivalent. Add to `help` section.

3. **Docs:**
   - `README.md` — add a short "Developer tooling" mention of `make validate-flags`.
   - `ONIONSKIN_FULL_HANDOFF.md` — same under developer-tooling section.

4. **Test:** `tests/` fixture file with known deprecated flag occurrences; unit test
   asserts scanner reports them with correct redirects and exits 1.

**Scope boundary:** no changes to `onionskin.py` public CLI. No subcommand. No dispatch
wrapper. The pre-parse `_check_deprecated_flags()` runtime gate stays as the safety net;
the script is a pre-flight tool that supplements, not replaces, the gate.

---

## Phase 5 — Closeout

*Implementation order:* **14-S17 metadata → 14-S18 metadata → 14-S16.**

*14-S17 (FOLDED) and 14-S18 (NOT A PRIORITY) are retained here as metadata
pointers for traceability; their actual scope has been absorbed elsewhere (S17
into S16 step 4; S18 into inline drift-audit work during any Role 2 round that
touches an agent file). 14-S16 is the final implementation step — doc sweep
across `PIPELINE_SPEC.md` / `ONIONSKIN_FULL_HANDOFF.md` / `README.md` /
`AGENT_CONVENTIONS.md` / `DECISIONS.md`, plus the tracking-directory `git mv` of
`multi-agent/BRAINSTORM.md` and `multi-agent/KNOWN_ISSUES.md` with reference
propagation across all four agent instruction files.*

## Priority 14-S17 — AGENT_CONVENTIONS.md CLI conventions section update — FOLDED into 14-S16

**Status (Q17 resolved): FOLDED INTO 14-S16** — user accepted the proposed 14-S16 scope
which already includes AGENT_CONVENTIONS.md updates. 14-S17 is not a standalone priority.

The AGENT_CONVENTIONS.md updates belong in the 14-S16 closeout doc sweep, step 4 of that
priority's scope. Items to cover there:

1. Add the `halfwidth`-vs-`window` naming rule from Q7/Q8 (halfwidth when the value is a
   search radius, not a full span).
2. Codify the Universal-tier group pattern beyond `--<pipeline>-<concept>`: Universal-in-
   spirit groups like "Asymmetric Triangle Model", "Shape scoring", "APS", "Timing",
   "Overlap" can hold shared flags with optional pipeline-specific overrides, and stay as
   their own parser groups (they do NOT merge into the "Universal" parser section). Note
   the terminology distinction from Q20: "universal" is overloaded between the
   parser-section name and the cross-pipeline concept.
3. Document the step-mention convention: full output directory step name in brackets;
   multi-pipeline format for Universal-in-spirit flags
   (e.g., `[hmm: 14-aps; growth: 13-aps; rms: 12-aps]`).
4. Document the standalone-engine-CLI-removal decision (from 14-S10) as a scope boundary
   rule: future engine files do not add argparse.
5. Ensure live code references (`onionskin.py:build_parser()`, `_resolve_override_value()`,
   `_DEPRECATED_FLAGS`, `_check_deprecated_flags()`) remain accurate.

---

## Priority 14-S18 — CLAUDE.md stale pointer + agent file drift — NOT A PRIORITY (Q18 resolved)

**Status (Q18 resolved): NOT A PRIORITY.** User direction: "This does not need a priority.
It is just something that can be done. A broader agent file audit for drift can be also be
done without making it a priority. Just report the results to chat screen about drift, ask
user if they want it fixed."

**Converted to a handled-inline task.** When any Role 2 implementation round touches an
agent file (e.g., as part of 14-S16 closeout doc sweep), the implementer performs a quick
drift audit across `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `.github/copilot-instructions.md`
and reports findings to the user in the chat. If the user approves, fix in the same round.

**Known items to surface when this runs:**
1. `CLAUDE.md:28` references stale `PHASE11_SPEC.md` — active phase plan now lives at
   `multi-agent/plans/PHASE14_SUPPLEMENTAL-SPEC.md` (or archive once Phase 14 closes).
2. Any parallel references in the other three agent files.
3. Any other archived-phase references that should point to the current active spec.

**No standalone audit/implement/re-audit loop.** This remains a small follow-through item.

---

## Priority 14-S16 — Phase 14 Supplemental closeout documentation sweep

**Goal:** Run a unified Role-1 documentation audit after all other supplemental priorities
are implemented and re-audited. This is the Phase 14 Supplemental equivalent of Phase 14
proper's Priority 14.10 (live documentation migration).

**Status: BLOCKED ON Q17** (see `PHASE14_SUPPLEMENTAL-FEEDBACK.md`). Scope confirmation
needed.

**Depends on:** 14-S1 through 14-S12 (plus 14-S13, 14-S14, 14-S15 if approved) all CLOSED.

**Proposed scope (subject to Q17):**

1. **`multi-agent/full_instructions/PIPELINE_SPEC.md`** — audit for:
   - Stale references to pre-supplemental Growth flag names (`--method`, `--ensemble-methods`,
     `--stage-weight-mode`, `--refine-window-kb`, `--refine-smooth-kb`,
     `--stage-median-resolution`, `--model-window-kb`, `--model-smooth-kb`, `--w-grid-kb`,
     `--peak-summary`, `--peak-quantile`, `--peak-topk`, `--growth-window`, `--rms-window`)
   - References to "Stage-2"/"Stage-1" in a detection-pass context (per 14-S4 terminology)
   - Missing coverage of the new Asymmetric Triangle Model group
   - Missing coverage of the Universal-promoted peak-summary flags

2. **`multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md`** — same audit targets.

3. **`README.md`** — user-facing flag examples; any "Universal section" overview text that
   should mention the new Asymmetric Triangle Model group or the Universal-promoted peak
   flags.

4. **`multi-agent/AGENT_CONVENTIONS.md`** CLI conventions section — verify still accurately
   describes the live parser; add halfwidth-vs-window rule; codify Universal-tier-with-
   growth-overrides pattern beyond the pipeline-specific pattern (Asymmetric Triangle Model
   group is an example). If Q17 splits this, the AGENT_CONVENTIONS update becomes 14-S17.

5. **`multi-agent/project_context/DECISIONS.md`** — add the "ms" ambiguity note from 14-S4;
   add terminology note from 14-S14 if approved; add engine-CLI-removal rationale from
   14-S10.

6. **Repo-wide stale-attribute grep sweep** — verify no lingering old attribute names
   (`args.method`, `args.peak_summary`, `args.w_grid_kb`, etc.) in code, tests, scripts, or
   documentation. Include `getattr(args, ...)` patterns per Phase 14.2 precedent.

7. **Tracking-directory reorg** — at Phase 14 Supplemental closeout, move long-lived tracking
   surfaces into `multi-agent/tracking/` (created during the recon-audit work in v0.14.55;
   currently holds `INTENDED-BUT-MISSED-PRIOR-TO-14.md`).
   - `git mv multi-agent/BRAINSTORM.md multi-agent/tracking/BRAINSTORM.md`
   - `git mv multi-agent/KNOWN_ISSUES.md multi-agent/tracking/KNOWN_ISSUES.md`
   - Update all references in the four agent instruction files
     (`CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `.github/copilot-instructions.md`) in the
     **same commit** per AGENT_CONVENTIONS.md consistency rule.
   - Update `multi-agent/AGENT_CONVENTIONS.md` — routing table, dedicated BRAINSTORM.md
     and KNOWN_ISSUES.md sections, planning-content routing table.
   - Update any other live references in `multi-agent/workflows/`,
     `multi-agent/full_instructions/`, `multi-agent/audits/`, and active phase-plan files.
   - **Do NOT update** references inside `multi-agent/plans/archived/` (historical record)
     or inside CHANGELOG.md entries written before the move (historical record).
   - Validation: `grep -rn "multi-agent/BRAINSTORM.md\|multi-agent/KNOWN_ISSUES.md"` should
     return only archival/historical hits after the move.

**Validation:**
- After edits: run each rename/old-name grep from prior priorities; confirm only intentional
  historical exceptions remain
- `make test` must still pass at 104+

---

