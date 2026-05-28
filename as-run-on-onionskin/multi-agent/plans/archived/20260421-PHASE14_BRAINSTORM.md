# Phase 14 — CLI Restructuring and Cleanup

**Last updated:** 2026-04-21 (Round 4 — Copilot + Gemini audit incorporated)
**Corresponds to:** ROADMAP Phase 14 (future `v0.14.xx`)
**Status:** Brainstorm / planning. Not started.

---

## What Phase 14 is

Phase 14 is a ground-up cleanup of the onionskin CLI (`build_parser()` in `onionskin.py`).
Goals:
1. Make each flag's pipeline ownership unambiguous from its name alone.
2. Eliminate legacy naming that predates the current "growth / hmm / rms" vocabulary.
3. Group flags by their conceptual category, not by implementation accident.
4. Replace silent legacy aliases with explicit deprecation errors that point to the correct flag.
5. Make `onionskin --help` readable end-to-end without needing prior knowledge of the codebase.

No behavioral changes are planned unless a flag name was genuinely misleading about its effect.

---

## Agreed section order (settled in planning)

```
1.  Input / Output
2.  Universal
3.  Shape scoring          ← NEW section
4.  Overlap resolution / Deduplication / twin-peak decomposition
5.  Timing
6.  APS — Amplification Progression Score
7.  HMM engine
8.  Growth model
9.  RCN Mean Shift (RMS)
```

The "Advanced" group is dissolved entirely. Every flag in it either belongs to a pipeline,
belongs in Universal, or belongs in Overlap.

---

## Priority 14.1 — Flag renames: "single/multi" → "rms/growth"

### RMS detection flags (currently `--*-single`)

| Current name | Proposed name | Notes |
|---|---|---|
| `--z-thresh-single` | `--rms-z-thresh` | |
| `--halfwidths-kb-single` | `--rms-halfwidths` | drop `-kb` per user preference |
| `--trend-kb-single` | `--rms-trend` | drop `-kb` |
| `--smooth-kb-single` | `--rms-smooth` | drop `-kb` |
| `--peak-search-kb-single` | `--rms-peak-search` | drop `-kb` |
| `--window-kb-single` | `--rms-window` | drop `-kb` |

> Units (kb) stay in the help string rather than the flag name.

### Growth detection flags (currently `--*-multi`)

| Current name | Proposed name | Notes |
|---|---|---|
| `--z-thresh-multi` | `--growth-z-thresh` | |
| `--halfwidths-kb-multi` | `--growth-halfwidths` | |
| `--trend-kb-multi` | `--growth-trend` | |
| `--smooth-kb-multi` | `--growth-smooth` | |
| `--peak-search-kb-multi` | `--growth-peak-search` | |
| `--window-kb-multi` | `--growth-window` | |

### Other direct renames

| Current name | Proposed name | Location change |
|---|---|---|
| `--rcn-mean-shift-norm-mode` | `--rms-norm-mode` | Advanced → RMS section |
| `--shape-filter-multistage` | `--growth-shape-filter` | Advanced → Growth section |
| `--ms-shape-score-threshold` | `--growth-shape-score-threshold` | Advanced → Growth section (overrides universal) |
| `--growth-rcn-smooth-bins` | `--growth-rcn-smooth-halfwidth` | Growth section (name only) |
| `--min-shape-score` | `--shape-score-threshold` | RMS section → Shape scoring section (universal) |
| `--strict-bic` | `--shape-score-strict-bic` | RMS section → Shape scoring section (see below) |
| `--dedup-peak-dist-kb` | `--dedup-dist` | Overlap section (stays, renamed) |

### `args.*` attribute renames (implementation work)

Every argparse rename changes the `args.xxx` attribute name accessed downstream.
A full grep of `onionskin.py` is required before implementation; known affected attributes:

```
args.z_thresh_single          → args.rms_z_thresh
args.halfwidths_kb_single     → args.rms_halfwidths
args.trend_kb_single          → args.rms_trend
args.smooth_kb_single         → args.rms_smooth
args.peak_search_kb_single    → args.rms_peak_search
args.window_kb_single         → args.rms_window
args.z_thresh_multi           → args.growth_z_thresh
args.halfwidths_kb_multi      → args.growth_halfwidths
args.trend_kb_multi           → args.growth_trend
args.smooth_kb_multi          → args.growth_smooth
args.peak_search_kb_multi     → args.growth_peak_search
args.window_kb_multi          → args.growth_window
args.rcn_mean_shift_norm_mode → args.rms_norm_mode
args.shape_filter_multistage  → args.growth_shape_filter
args.ms_shape_score_threshold → args.growth_shape_score_threshold
args.growth_rcn_smooth_bins   → args.growth_rcn_smooth_halfwidth
args.min_shape_score          → args.shape_score_threshold
args.strict_bic               → args.shape_score_strict_bic
args.dedup_peak_dist_kb       → args.dedup_dist
```

All call sites in `onionskin.py` must be updated in the same pass.
Key locations identified from audit:
- `onionskin.py:2088-2093` — `_build_ms_argv()` builds growth subprocess argv (`args.z_thresh_multi`, etc.)
- `onionskin.py:2131-2144` — `_build_detect_extra()` builds single-sample detection argv (`args.z_thresh_single`, etc.) — **separate function from `_build_ms_argv()`**
- `onionskin.py:2147-2161` — `_refine_kwargs()` builds kwargs for run_single_stage refinement: reads `args.z_thresh_single`, `args.min_shape_score`, `args.strict_bic`, `args.dedup_peak_dist_kb`, `args.growth_rcn_smooth_bins` — all need updating
- `onionskin.py:2150-2159` — `_run_rcn_mean_shift_controller()` direct call
- `onionskin.py:2354-2372` — RCN controller call
- `onionskin.py:3381-3384` — standalone RMS block shape filter / dedup
- `onionskin.py:3478-3481` — another apply_shape_filter / dedup call
- `onionskin.py:2236` — error message string `"--rcn-mean-shift-norm-mode ref-stage requires..."` → update to `--rms-norm-mode`

---

## Priority 14.2 — Legacy alias removal with deprecation errors

Current silent aliases to replace with explicit error catches:

| Removed flag | Error message / redirect |
|---|---|
| `--rcn-summit-policy` | "Unknown flag. Use `--rms-summit-policy` instead." |
| `--rcn-early-summit-stages` | "Unknown flag. Use `--rms-early-summit-stages` instead." |
| `--per-stage-norm-mode` | "Unknown flag. Use `--rms-norm-mode` instead." |
| `--rcn-mean-shift-norm-mode` | "Unknown flag. Use `--rms-norm-mode` instead." |
| `--min-shape-score` | "Unknown flag. Use `--shape-score-threshold` instead." |
| `--strict-bic` | "Unknown flag. Use `--shape-score-strict-bic on` instead." |
| `--dedup-peak-dist-kb` | "Unknown flag. Use `--dedup-dist` instead." |
| `--shape-filter-multistage` | "Unknown flag. Use `--growth-shape-filter` instead." |
| `--ms-shape-score-threshold` | "Unknown flag. Use `--growth-shape-score-threshold` instead." |
| `--no-aps-singleton-guard` | "Unknown flag. Use `--aps-singleton-guard off` instead." |

Also add deprecation catches for each `--*-single` / `--*-multi` rename:

| Removed flag | Redirect |
|---|---|
| `--z-thresh-single` | `--rms-z-thresh` |
| `--halfwidths-kb-single` | `--rms-halfwidths` |
| `--trend-kb-single` | `--rms-trend` |
| `--smooth-kb-single` | `--rms-smooth` |
| `--peak-search-kb-single` | `--rms-peak-search` |
| `--window-kb-single` | `--rms-window` |
| `--z-thresh-multi` | `--growth-z-thresh` |
| `--halfwidths-kb-multi` | `--growth-halfwidths` |
| `--trend-kb-multi` | `--growth-trend` |
| `--smooth-kb-multi` | `--growth-smooth` |
| `--peak-search-kb-multi` | `--growth-peak-search` |
| `--window-kb-multi` | `--growth-window` |
| `--growth-rcn-smooth-bins` | `--growth-rcn-smooth-halfwidth` |

Implementation pattern: pre-parse `sys.argv` scan + `sys.exit(1)` with redirect message. This
catches stale test/script invocations before argparse parses anything.

HMM PuffStep synonyms (`--hmm-expected-special-length`, `--hmm-leave-special-state`, etc.):
**keep these** — they are compatibility bridges to a predecessor tool, not internal naming drift.

---

## Priority 14.3 — Section migration

### New: Universal section

Move these flags to Universal (affects all pipelines or the whole run):

- `--pipelines` (currently Advanced)
- `--norm-mode` (currently Growth — see Priority 14.6 for behavior change)
- `--ref-stage` (currently Advanced)
- `--threads` (currently Advanced)
- `--rng-seed` (currently RMS — affects bootstrap and stochastic steps globally)
- `--min-seq-length` (currently HMM — help string already says it applies to all pipelines)
- `--min-bin-count-per-seq` (currently HMM — same)
- `--verbose` (currently Advanced)
- `--debug` (currently Advanced)
- `--bootstrap-origins` (currently RMS — generalizable; note in help: "currently applies to RMS
  pipeline only; `--rms-bootstrap-origins` overrides this for the RMS pipeline specifically")

Update `--min-seq-length` and `--min-bin-count-per-seq` help strings: remove "HMM:" prefix,
say "Applied to all pipelines when sequence/bin filtering is active."

### New: Shape scoring section (universal)

A new universal section between Universal and Overlap:

```
--shape-score-threshold      (renamed from --min-shape-score; universal default)
--shape-score-strict-bic     (renamed from --strict-bic; on|off, default off)
```

Each pipeline also gets per-pipeline override flags in its own section (all default None → inherits
from `--shape-score-threshold` / `--shape-score-strict-bic`):

```
--rms-shape-score-threshold      (in RMS section)
--rms-shape-score-strict-bic     (in RMS section)
--growth-shape-score-threshold   (in Growth section, renamed from --ms-shape-score-threshold)
--growth-shape-score-strict-bic  (in Growth section)
--hmm-shape-score-threshold      (placeholder in HMM section — not yet wired)
--hmm-shape-score-strict-bic     (placeholder in HMM section — not yet wired)
```

`--strict-bic` was `action="store_true"` (boolean). Changing to `choices=["on", "off"]` means
a behavior change at the argparse level. The deprecation catch `--strict-bic` should emit a clear
message: "Use `--shape-score-strict-bic on` instead."

### Flags moving to Overlap section (from Advanced)

- `--overlap-smooth-bp`
- `--overlap-outlier-window-bp`

### Flags moving to Growth section (from Advanced)

- `--growth-norm-mode` (already prefixed; move only)
- `--growth-shape-filter` (renamed from `--shape-filter-multistage`)
- `--growth-shape-score-threshold` (renamed from `--ms-shape-score-threshold`)

New flags added to Growth section (per universal/override pattern):
- `--growth-shape-score-strict-bic [on|off]` (override of universal `--shape-score-strict-bic`)
- `--growth-bootstrap-origins` (override of universal `--bootstrap-origins`, default None)

### Flags moving to RMS section (from Advanced)

- `--rms-norm-mode` (renamed from `--rcn-mean-shift-norm-mode`)

New flags added to RMS section (per universal/override pattern):
- `--rms-shape-score-threshold` (override)
- `--rms-shape-score-strict-bic [on|off]` (override)
- `--rms-bootstrap-origins` (override, default None inherits from `--bootstrap-origins`)

### Flags moving to HMM section (from Advanced)

- `--hmm-norm-mode` (already prefixed; move only)

New placeholder flags added to HMM section:
- `--hmm-shape-score-threshold` (placeholder; not yet wired)
- `--hmm-shape-score-strict-bic [on|off]` (placeholder; not yet wired)

### Dissolve the Advanced section

After migration, Advanced should be empty and removed. No "Diagnostic / development flags"
stub needed — nothing qualifies.

---

## Priority 14.4 — `--norm-mode` behavior change

Currently `--norm-mode` defaults to `"chrom-median"`. The agreed behavior (confirmed):

- If neither `--norm-mode` nor `--<pipeline>-norm-mode` is set → pipeline uses its built-in default:
  - growth: `chrom-median`
  - rms: `chrom-median`
  - hmm: `ref-stage`
- If `--norm-mode` is set but not `--<pipeline>-norm-mode` → pipeline inherits `--norm-mode`
- If `--<pipeline>-norm-mode` is set → that value is used regardless of `--norm-mode`

The exact implementation strategy (None default vs argparse.SUPPRESS) is flexible; only the
behavior matters. A clean approach:

1. Change `--norm-mode` default from `"chrom-median"` to `None`
2. Update `_resolve_pipeline_norm_mode()` at `onionskin.py:1989`:
   - when `shared` (the `--norm-mode` value) is `None`, `_default_pipeline_norm_mode()` returns
     the pipeline's built-in default instead of forwarding `None`
3. Update `getattr(args, "norm_mode", "chrom-median")` at `onionskin.py:1996` →
   `getattr(args, "norm_mode", None)`
4. Update `getattr(args, "norm_mode", "chrom-median")` at `onionskin.py:2215` similarly
5. Update `onionskin.py:512`: `norm_mode = norm_mode_override or getattr(args, "norm_mode", "chrom-median")`

Note on HMM default: HMM currently defaults to `ref-stage` (not `chrom-median`). In a future
phase, HMM may be updated to default to `chrom-median` as well, but that is out of scope for
Phase 14. See the new KNOWN_ISSUES entry added by this phase.

---

## Priority 14.5 — Section header and description updates

### RMS section

Rename from:
```
"rcn-mean-shift / single-stage detection"
```
To:
```
"RCN Mean Shift (RMS)"
```
Description: "Options for the stage-local per-sample detection lane (step-03 pipeline).
Applies to single-file, single-stage, and standalone --pipelines rms runs."

Remove "single-stage" framing entirely — stale terminology.

### Growth section

Rename from:
```
"Growth-model stage-structured"
"Options for the growth-model lane on stage-structured inputs; multistage is an input
property, not the pipeline name."
```
To:
```
"Growth model"
"Options for the growth-model stage-progression pipeline (step-02)."
```

### HMM section

Current header: `"HMM engine (Phase 7)"` → strip to just `"HMM engine"`

Current description references:
- implementation detail about "native implementation is the default execution path"
- "Step-6 rollback backend remains env-only via ONIONSKIN_HMM_STEP6_BACKEND=..."
- "Ported from PuffStep (puffStep.py puffcn)"

Strip all three. New description: "Parameters for the HMM state-path pipeline. Enable with
--pipelines hmm (or a list that includes hmm). PuffStep flag synonyms are accepted alongside
the onionskin-native names."

### Overlap section

Rename from:
```
"Overlap resolution / twin-peak decomposition"
```
To:
```
"Overlap resolution / Deduplication / twin-peak decomposition"
```

### Help strings for pipeline-ownership flags

For every flag that moves from a pipeline-specific section to Universal, update the help
string to explain which pipelines it affects. For example:
- `--bootstrap-origins`: "Bootstrap replicates for origin confidence interval (0 disables).
  Currently applies to the RMS pipeline. `--rms-bootstrap-origins` overrides this for RMS."
- `--rng-seed`: "Random seed for bootstrap and stochastic steps across all pipelines."

---

## Priority 14.6 — Test updates

Every test that uses a renamed flag must be updated. Confirmed from audit:

### `tests/test_pipeline.py`
- **Line 41–60: `test_help_runs()`** — this test currently asserts the exact old section header
  strings (line 57: `"rcn-mean-shift / single-stage detection"`, line 60:
  `"Growth-model stage-structured"`). It is a full snapshot-style help-text validator, not just
  a few targeted checks. Phase 14 must treat `test_help_runs()` as a **complete snapshot refresh**
  — all asserted header strings and normalization wording must be updated to match the new parser.
  This is the biggest single-test blast radius in the suite.
- Line 48: `assert "--rcn-mean-shift-norm-mode" in out` → change to `assert "--rms-norm-mode" in out`
- Line 370: `"--rcn-mean-shift-norm-mode"` → `"--rms-norm-mode"`
- Line 546: test name `"--per-stage-norm-mode must be accepted as an alias for --rcn-mean-shift-norm-mode"` → convert to a deprecation error test (expect sys.exit / error)
- Line 652: `"--rcn-mean-shift-norm-mode"` → `"--rms-norm-mode"`
- Line 724: `"--rcn-mean-shift-norm-mode"` → `"--rms-norm-mode"`
- Line 1015: `"--rcn-mean-shift-norm-mode"` → `"--rms-norm-mode"`

### `tests/optimize_single_params.py`
Search grids are keyed by old flag names (lines ~295–315):
`"z-thresh-single"`, `"trend-kb-single"`, `"smooth-kb-single"`, `"halfwidths-kb-single"`.
All key names must be updated to the new `rms-*` equivalents:
`"rms-z-thresh"`, `"rms-trend"`, `"rms-smooth"`, `"rms-halfwidths"`.

### `tests/run_twin_peak_test.sh`
- Lines 128, 135: `--z-thresh-multi` → `--growth-z-thresh`

### `tests/run_full_chrII_test.sh`
- Lines 257, 292: `--min-shape-score` → `--shape-score-threshold`

### `tests/test_strict_bic.sh`
- `--strict-bic` → `--shape-score-strict-bic on`

### `tests/compare_strict_bic.py`
- All references to `--strict-bic` string → `--shape-score-strict-bic on`

### `tests/scan_growth_methods_ds1.sh`
- Line 71: `--z-thresh-multi` → `--growth-z-thresh`

### `tests/full_chrom_training_data/README.txt`
- Mentions `--z-thresh-multi 4.0` in example/baseline notes → update to `--growth-z-thresh 4.0`

### Makefile
- Line 103 (`strict-bic` target): Update help text description from `--strict-bic` to `--shape-score-strict-bic on`

### `onionskin_core/aps.py`
- Line 514: docstring mentions `--no-aps-singleton-guard` → update to `--aps-singleton-guard off`

### Add new deprecation error tests

Add tests that confirm old flags now produce sys.exit(1) with a redirect message:
- One test per removed flag name in the tables in Priority 14.2
- These replace the alias-acceptance tests

---

## Priority 14.7 — Single-file `main()` flag consistency

`rcn_mean_shift_engine.py:main()` has its own argparse (standalone per-file CLI):
`--z-thresh`, `--halfwidths-kb`, `--trend-kb`, `--smooth-kb`, `--peak-search-kb`, `--window-kb`.

These are intentionally shorter (no prefix needed since `main()` is RMS-only by definition).
**No rename needed** for those flags.

Add a note to `main()` help or description: "These flags correspond to `--rms-*` in the full
onionskin CLI."

---

## Priority 14.8 — `scripts/` and auxiliary tool cleanup

### `scripts/aps_cluster_experiments.py`
Lines 29–38 use `--no-aps-singleton-guard`, which is NOT a current CLI flag.
The current flag is `--aps-singleton-guard off`. Update all uses from
`"--no-aps-singleton-guard"` to `"--aps-singleton-guard", "off"`.

### `scripts/summit_inspector.py`
This tool has its own `--pipelines` selector and pipeline vocabulary. It currently
exposes `rcn-mean-shift` to its users and has no `rms` synonym. Updates needed:

- `PIPELINE_ORDER` at line 76: add `"rms"` as a recognized token (or handle via normalization)
- `PIPELINE_LABELS` at line 77–80: add `"rms": "RCN mean-shift"` entry
- `_parse_pipeline_list()` at line 239: normalize `"rms"` → `"rcn-mean-shift"` before
  comparing against `PIPELINE_ORDER`, so `--pipelines rms` is accepted
- `--pipelines` help text in `summit_inspector.py`: add `rms` to the list of accepted values

The `rcn-mean-shift` token can remain valid internally (the `PIPELINE_ORDER` tuple and all
directory-path logic uses it). `rms` is the user-facing alias only.

### `scripts/generate_hmm_v2_references.py`

Verified: this script contains no legacy flag names. No changes needed. Noting explicitly so
the implementation pass does not have to re-verify it.

---

## Priority 14.9 — `--pipelines rms` synonym in the main CLI

Accept both `"rms"` and `"rcn-mean-shift"` as valid values in `--pipelines`. The help string
advertises `"rms"` as the canonical short form; `"rcn-mean-shift"` remains silently accepted for
backward compatibility. Tests may continue using `"rcn-mean-shift"` without updating. No
deprecation error needed — both are valid permanently.

Implementation: wherever `--pipelines` is parsed (the `_pipelines_include()` helper and related
functions), normalize `"rms"` → `"rcn-mean-shift"` before any comparison logic, or make all
comparisons accept either string.

---

## Priority 14.10 — `growth_model_engine.py:main()` standalone CLI scope

`growth_model_engine.py:main()` is a standalone argparse entry point (parallel to the
`rcn_mean_shift_engine.py:main()` discussed in Priority 14.7). It exposes engine-native flag
names (e.g. `--dedup-peak-dist-kb`) that are independent of the top-level `onionskin.py` CLI.

**Explicit scope decision (matching Priority 14.7):** Phase 14 does NOT rename the standalone
engine CLI flags in `growth_model_engine.py:main()`. Rationale: engine entry points are
lower-level tools; their shorter un-prefixed names are appropriate for single-engine use.

**What Phase 14 does do:**
- Add a note to `growth_model_engine.py:main()` help or description: "These flags correspond
  to `--growth-*` in the full onionskin CLI."
- Document the intentional boundary: `_build_ms_argv()` in `onionskin.py` translates the new
  top-level `--growth-*` names back to engine-native names when calling the growth engine.
  This translation layer is deliberate — it keeps the engine's own interface stable.

---

## Priority 14.11 — Developer-helper surface naming decision

Some developer-facing file and target names embed the old flag vocabulary:
- `tests/test_strict_bic.sh` — shell test script
- `tests/compare_strict_bic.py` — Python comparison utility
- `make strict-bic` — Makefile target

Phase 14 renames the runtime flags (`--strict-bic` → `--shape-score-strict-bic`) and updates
all flag invocations inside these surfaces. The question is whether the surface names themselves
should also change.

**Explicit decision:**
- **Keep** `tests/test_strict_bic.sh` and `tests/compare_strict_bic.py` as-is.
  Rationale: renaming files disrupts `git log` / `git blame` history without meaningful
  user-facing benefit; these are internal dev tools, not public CLI.
- **Rename** the Makefile target from `make strict-bic` → `make shape-score-bic`.
  Rationale: Makefile targets ARE user-facing (shown in `make help`); keeping `strict-bic`
  as the visible target name after the CLI vocabulary changes creates a vocabulary split
  between the runtime CLI and the developer entry point. The rename cost is trivial.

Both files' interiors must still be updated to use `--shape-score-strict-bic on` instead of
`--strict-bic` (covered under Priority 14.6).

---

## Priority 14.12 — CLI flag conventions rule in AGENT_CONVENTIONS.md

Add an explicit rule to `multi-agent/AGENT_CONVENTIONS.md` (and reference it from the
SPEC/brainstorm files) covering how to add and update CLI flags. Core rules (drawn from
PDS-v1.md and this phase's decisions):

- If a flag was previously restricted to one pipeline but now applies to more, it must have
  a canonical flag in the Universal section plus pipeline-specific override flags.
- If a new flag will eventually apply to multiple pipelines, add it to Universal immediately
  (even if only one pipeline implements it yet), with a help-string note on current coverage.
- Pipeline-specific override flags default to None and inherit from the universal unless set.
- Use the `--<pipeline>-<concept>` naming pattern for pipeline-specific flags.
- All pipeline-specific flags that change the behavior of a universal flag must be listed
  in the help string of the universal flag.

---

## Priority 14.13 — Live documentation full migration scope

The following documents are **full live-doc migration surfaces** — old flag names appear
throughout, not just in one appendix or one CLI summary block. Phase 14 must treat each as a
complete pass, not a spot-fix.

### `multi-agent/full_instructions/PIPELINE_SPEC.md`

Old names appear in:
- Appendix A (CLI flags) — already planned
- Step narratives and algorithm descriptions (`--min-shape-score`, `--strict-bic`, `--dedup-peak-dist-kb`)
- Default and calibration sections (`--z-thresh-single`, `--z-thresh-multi` default values)
- Parameter tables (`--no-aps-singleton-guard`, `--rcn-mean-shift-norm-mode`, etc.)

**Required action:** full search-and-replace pass with a reviewed list of all old → new name
mappings. Only intentionally historical content (e.g. "as of v0.13.xx this was called X")
should retain old names.

### `multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md`

Old names appear across multiple conceptual sections:
- Shape-filter discussion
- Deduplication discussion
- APS singleton-guard discussion
- Historical default/calibration notes (example `make strict-bic` usage)
- Any CLI overview or example invocations

**Required action:** same full pass as PIPELINE_SPEC.md.

### `ROADMAP.md`

The Phase 14 block in ROADMAP.md should use the new vocabulary when referencing planned flag
names, section names, and pipeline tokens (e.g. "rms" not "rcn-mean-shift" where the short
form is now canonical).

**Required action:** update the Phase 14 priority descriptions to use new names; leave all
older phase content untouched (historical).

### Scope boundary

Only the following should intentionally retain old flag names:
- `CHANGELOG.md` entries written before Phase 14 (historical record)
- Archived plan files in `multi-agent/plans/archived/`
- Comments in code marked as "legacy compatibility note"

---

## Implementation notes

- All renames require adding deprecation-catching code (pre-parse `sys.argv` scan) to catch
  old flag names and print a clear message, since we are the only users and want to catch
  stale test/script invocations.
- Do all renames in a single pass to avoid half-renamed states.
- Run `make test`, `make toy`, `make single`, `make twin` after implementation.
- `--strict-bic` changing from `action="store_true"` to `choices=["on","off"]` is a
  behavioral change at the argparse level; the deprecation catch handles old invocations.
- `test_help_runs()` is a full parser snapshot test — expect it to fail immediately on
  every section header and wording change. Treat updating this test as a required part of
  the implementation, not a follow-up.
- `PIPELINE_SPEC.md` and `ONIONSKIN_FULL_HANDOFF.md` require **full migration passes** (see
  Priority 14.13) — old names appear in step narratives, parameter tables, and calibration
  sections, not just the CLI appendix. Treat them as complete passes, not spot-fixes.
- `ROADMAP.md`: update Phase 14 block to use new vocabulary; leave all earlier phases as-is.
- Update `multi-agent/KNOWN_ISSUES.md` if any known issue references old flag names.
- Deprecation error mechanism: the pre-parse `sys.argv` scan approach is recommended (simple
  and catches errors before argparse runs). An `argparse.Action`-based approach is a valid
  alternative; pick one and use it consistently across all deprecated flags.
- The translation layer in `_build_ms_argv()` and `_build_detect_extra()` (top-level
  `--rms-*` / `--growth-*` → engine-native short names) is intentional; do not remove or
  bypass it. `_refine_kwargs()` at `onionskin.py:2147` is a third translation call site.
