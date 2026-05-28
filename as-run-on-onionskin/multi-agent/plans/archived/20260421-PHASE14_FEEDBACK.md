# Phase 14 — FEEDBACK FOR BRAINSTORMING AND SPEC ENGINEERING

**Last updated:** 2026-04-21


## Open Questions Round 1 for Triage / Phase 14 planning

1. **`--bootstrap-origins`**: Currently in RMS section. Does it apply to growth or HMM too?
   If RMS-only, rename to `--rms-bootstrap-origins` for consistency.

2. **`--min-shape-score`**: Currently in RMS section (no pipeline prefix). Should it also exist
   for growth? (Growth has `--growth-shape-score-threshold` for the multistage filter.) If
   `--min-shape-score` is RMS-only, rename to `--rms-min-shape-score`.

3. **`--strict-bic`**: Currently in RMS section. Is this also relevant to growth? If RMS-only,
   rename to `--rms-strict-bic`.

4. **`--dedup-peak-dist-kb`**: In Overlap section. Applies to all pipelines or just some?
   Help string says "Applied before overlap resolution and after detection." Appears universal.
   Consider renaming to `--dedup-dist-kb` or keeping as-is with a clarified help string.

5. **`--growth-rcn-smooth-bins`**: Already properly prefixed in the Growth section. No change needed.

6. **`--norm-mode` position**: Moving to Universal means it loses the explicit note about being
   the growth/rcn fallback. The help string needs to be updated to explain all three pipelines.

7. **`--pipelines` help string**: Says "Default: all (growth + rcn-mean-shift)". HMM is opt-in.
   Update to say "Available: growth, rcn-mean-shift (rms), hmm."


**Author:** Claude Code 2.1.98 (claude-sonnet-4-6)


## Answers to Open Questions Round 1 for Triage / Phase 14 planning

1. **`--bootstrap-origins`**: 

This is an example of a CLI flag that might currently be RMS-only, but that can likely be generalized to all pipelines. So it should go to the universal section, and have a note in its help string that at the moment it only applies to the RMS pipeline. Then in the RMS section we can have the flag `--rms-bootstrap-origins` that defaults to `--bootstrap-origins` unless it is specified in the RMS section.

2. **`--min-shape-score`**: 

This is an example of a flag that should be in the universal section as is, and have pipeline-specific versions that can override the universal default as explained for --bootstrap-origins. `--min-shape-score` should be renamed to `--shape-score-threshold` and should be the universal default flag that `--rms-shape-score-threshold` and `--growth-shape-score-threshold` default to. There absolutely will be an HMM flag for this as well so we can add it now as a placeholder: `--hmm-shape-score-threshold` 

3. **`--strict-bic`**: 

This should be relevant to the universal shape score parameters. Its name shuld be updated to reflect it being part of the shape scoring: `--shape-score-strict-bic`. It should be a two option flag (on vs off): `--shape-score-strict-bic [on|off]` and it should default to "off" for now.


Shape scoring should get its own universal section (in addition to APS, Timing, and Overlap Resolution), which will at minimum include:
`--shape-score-threshold`
`--shape-score-strict-bic`


Each pipeline can have its own valve to turn this on or off regardless of the universal - e.g.:
`--hmm-shape-score-strict-bic [on|off] `
`--growth-shape-score-strict-bic [on|off]`
`--rms-shape-score-strict-bic [on|off]`
These all default to "None" which defaults to `--shape-score-strict-bic [on|off]`

4. **`--dedup-peak-dist-kb`**: 

Anything in the overlap section should be considered as potential to be universal even if it applies to only one pipeline right now. Same goes for timing and APS sections. I imagine this applies to both RMS and Growth pipelines, each of which uses mean shift for calls, correct?

As a catchall, the Overlap section could be renamed to: "Overlap resolution / Deduplication / twin peak decomposition" 

The rename can just be "--dedup-dist"
- you removed peak which is fine
- "kb" is reundant with the help string.


5. **`--growth-rcn-smooth-bins`**: 

- Already properly prefixed in the Growth section. No change needed **to prefix**.
- We should consider making the name more like similar CLI flags for the HMM 
- e.g. `--hmm-smooth-halfwidth` suggests this flag should be `--growth-rcn-smooth-halfwidth`
- `-rcn-` should be kept to differentiate it from the growth track, if this is for smoothing RCN files and not the growth track file.



6. **`--norm-mode` position**: 

This one is more complicated. I believe the current defaults for each mode are:
- chrom-median for growth
- chrom-median for rms
- ref-stage for hmm

The way this should act:
- each stage's norm mode default is used if --norm-mode is left None
    - each stage is set to default to None, but if None is detected it is updated to our planned default via processing the argument with a little bit of if/else code.
- if --norm-mode is specified it becomes the default for each stage
    - if --norm-mode is not None, then each stage inherits it by default if it is still set to None
- if a given stage norm mode is specified, it overrides --norm-mode
    - if a stage norm-mode flag is used, then its value is not None, and the user-selected norm mode for that stage is used regardless of the status of --norm-mode

Does that all make sense?

7. **`--pipelines` help string**: 

   Update to say:
   
   Available: hmm, growth, rms, all. Any one or more can be used with a comma-separated list. all = hmm,growth,rms. hmm = hidden markov model. growth = multiple stage growth modeling. rms = RCN mean shift. RCN = relative copy number. HMM and RMS can handle a single file, single stage with multiple replicates, a single stage versus a reference stage (each with 1 or more replicates), and multiple stages with or without a reference stage (--norm-mode set to chrom-median or ref-stage, respectively). The growth modeling pipeline is designed for 3 or more stages without a reference stage (--norm-mode chrom-median) or 4 or more stages when one is a reference stage (--norm-mode ref-stage). To use different normalization modes across pipelines, see their pipeline-specific --norm-mode flags.


**Author:** John M. Urban


---


## Open Questions Round 2 — from codebase audit

1. **`--pipelines` value normalization**: The new `--pipelines` help string uses `rms` as the
   short form for `rcn-mean-shift`. Tests and existing scripts currently pass `rcn-mean-shift`
   as the pipeline value (e.g. `--pipelines rcn-mean-shift`). Should `--pipelines rms` be
   accepted as a synonym, or does the canonical internal value remain `rcn-mean-shift` and
   `rms` is display-only? Accepting both is backward compatible but means two valid strings.
   Requiring migration to `rms` is cleaner but breaks existing tests and scripts.

2. **`--bootstrap-origins` for growth**: Audit confirmed `args.bootstrap_origins` is passed
   in `_build_ms_argv()` (line 2095) so the growth pipeline subprocess already receives it.
   Is this intentional? The flag was in the RMS section but has silently affected growth too.
   Moving it to Universal formalizes that — just confirming this is desired.

3. **`--growth-norm-mode` default**: Currently `default=argparse.SUPPRESS` (treated as not set).
   After the `--norm-mode` default changes to `None`, the `_resolve_growth_norm_mode()` logic
   must produce `chrom-median` when both `--norm-mode` and `--growth-norm-mode` are unset.
   This is already what `_default_pipeline_norm_mode()` returns for growth, but worth
   confirming the expected behavior is: unset → chrom-median, not unset → error.


**Author:** Claude Code 2.1.98 (claude-sonnet-4-6)


---


## Answers to "Open Questions Round 2 — from codebase audit"

1. **`--pipelines` value normalization**: 

- Lets allow both rcn-mean-shift and rms.
- we do not have to advertise "rcn-mean-shift" but we can continue using it in our tests, and it can be allowable in general.


2. **`--bootstrap-origins` for growth**: 

- Yes moving it to Universal and formalizes that is desired.
- We have not always been principled in how we added and/or updated CLI flags
- That is why I am focused on it in this phase
- Should we add a rule to AGENT_CONVENTIONS.md regarding rules for adding and/or updating CLI flags and things that affect CLI flags?
    - Maybe that rule should be made explicit in the BRAINSTORM and SPEC files as well.
    - I added a section to `multi-agent/workflows/phase-development-system_PDS-v1.md` regarding this

3. **`--growth-norm-mode` default**: 

Exactly how we implement the idea I outlined in a previous answer is up to you. When we use a "None" strategy or a "argparse.SUPPRESS" strategy is not important to me, only the behavior and results are important.

- If the user does not do anything with --norm-mode or --growth-norm-mode, then the norm mode for growth should default to chrom-median.
- Else if the user specifies something with --norm-mode, but not with --growth-norm-mode, then the latter defaults to the former.
- Else if the user specifies something with --norm-mode but also something with --growth-norm mode, then the latter is what is used for the growth pipeline.

I think that covers all bases.

The same type of behavior is expected for HMM and RMS norm modes too.
- Exactly the same for RMS
- Similar but with the first default being ref-stage (when the user doesn't attempt to use either flag)

Note that in the future when the HMM pipeline is updated, I plan to possibly let it default to chrom-median as well, but we are not there yet. Nonetheless, we can add that note to somewhere such as KNOWN_ISSUES.md.

**Author:** John M. Urban


---


## Copilot Audit Round 3 — missed blast radius and scope clarifications

This pass focused on what still looks under-specified after comparing the current
Phase 14 brainstorm against the live parser, tests, helper scripts, and a few
developer-facing docs. The main brainstorm is already strong; the items below are
meant to sharpen scope and reduce avoidable stale-surface cleanup after the rename
pass.

1. **Top-level help regression surface is broader than the current rename checklist.**

    `tests/test_pipeline.py::test_help_runs()` is not just checking a few flag names.
    It currently asserts the existing section headers and help phrasing, including:
    - `"rcn-mean-shift / single-stage detection"`
    - `"Growth-model stage-structured"`
    - the current normalization wording around `--norm-mode`

    Since Priority 14.5 intentionally rewrites those headers and descriptions,
    Phase 14 should explicitly treat the top-level parser help test as a full
    snapshot-style refresh, not only a few targeted string substitutions.

    **Why it matters:** otherwise the implementation checklist underestimates how much
    of the parser-help surface will fail immediately even if the flag renames are correct.

2. **Auxiliary `--pipelines` CLIs need an explicit Phase 14 decision, not just the main parser.**

    `scripts/summit_inspector.py` has its own `--pipelines` selector and currently
    advertises `growth,hmm,rcn-mean-shift,all` in its help text.

    Phase 14 already settles the main onionskin CLI direction as:
    - advertise `rms`
    - continue accepting `rcn-mean-shift`

    What is still not explicit is whether auxiliary tools should:
    - also accept `rms` as a synonym, or
    - intentionally keep the filesystem-facing token `rcn-mean-shift`

    **Why it matters:** without an explicit decision, the repo will end Phase 14 with
    two different user-facing pipeline vocabularies depending on whether the user is
    launching the main CLI or a helper tool.

    **Suggested routing:** add this as a small subtask under Priority 14.8 or 14.9.

3. **Standalone growth-engine CLI scope needs the same treatment already planned for standalone RMS.**

    `onionskin_core/engines/growth_model_engine.py:main()` remains a public argparse
    surface and still exposes engine-native names such as `--dedup-peak-dist-kb`.
    The current brainstorm explicitly discusses the standalone RMS CLI (`Priority 14.7`),
    but not the standalone growth-engine CLI.

    I think the missing decision is:
    - Phase 14 renames only the top-level `onionskin.py` CLI and keeps engine CLIs
       engine-native, with a brief help note/crosswalk, **or**
    - Phase 14 also harmonizes standalone engine entry points where they are treated
       as public user tools.

    **Why it matters:** `_build_ms_argv()` will otherwise translate new top-level names
    back into older engine-native ones, which is fine technically, but it should be a
    deliberate boundary and documented as such.

4. **Developer docs and utilities still hardcode old names outside the currently listed Phase 14 files.**

    The brainstorm already names `PIPELINE_SPEC.md`, `ONIONSKIN_FULL_HANDOFF.md`,
    `tests/test_pipeline.py`, `tests/test_strict_bic.sh`, `tests/run_full_chrII_test.sh`,
    `tests/run_twin_peak_test.sh`, `tests/scan_growth_methods_ds1.sh`, `Makefile`, and
    `scripts/aps_cluster_experiments.py`. In addition, the live tree still has older
    names in at least these surfaces:

    - `tests/optimize_single_params.py` — search grids keyed by `z-thresh-single`,
       `trend-kb-single`, `smooth-kb-single`, `halfwidths-kb-single`, etc.
    - `tests/full_chrom_training_data/README.txt` — example baseline wording still cites
       `--z-thresh-multi 4.0`
    - `onionskin_core/aps.py` — docstring still mentions `--no-aps-singleton-guard`
    - `Makefile` help text still describes the `strict-bic` target in terms of
       `--strict-bic`

    **Why it matters:** these are exactly the kinds of low-visibility surfaces that keep
    old vocabulary alive after a supposedly complete CLI cleanup.

    **Suggested routing:** either expand Priority 14.6 / 14.8 to include a short
    "developer-doc and utility surface" checklist, or explicitly defer these so the
    implementation phase does not silently leave them stale.

Overall judgment: `PHASE14_BRAINSTORM.md` already captures the main architecture of the
rename/migration. The remaining risk is not missing a major conceptual rename; it is
ending the phase with inconsistent vocabulary across the main CLI, auxiliary CLIs,
tests, utilities, and developer-facing documentation.

**Author:** GitHub Copilot (GPT-5.4)


## Phase 14 Deep Dive Audit — Gemini CLI (2026-04-21)

I have performed a comprehensive audit of the codebase, scripts, and tests to identify additional Priorities and "blast radius" items for Phase 14.

### 1. Auxiliary CLI Harmonization (Refinement for Priority 14.5/14.9)
The brainstorm already mentions `rms` synonyms, but my audit identified specific locations where this must be enforced to ensure a consistent developer/user experience:
- **`scripts/summit_inspector.py`**:
    - Add `rms` to `PIPELINE_ORDER` and `PIPELINE_LABELS`.
    - Update `--pipelines` help text to include `rms`.
    - Ensure `_parse_pipeline_list` handles the synonym before validation.
- **`scripts/aps_cluster_experiments.py`**:
    - This script currently hardcodes `"--no-aps-singleton-guard"`. It must be updated to use the new `"--aps-singleton-guard", "off"` syntax.
- **Standalone Engine CLIs**:
    - `onionskin_core/engines/growth_model_engine.py` and `onionskin_core/engines/rcn_mean_shift_engine.py` have their own `main()` functions. While internal, their flags (e.g., `--z-thresh`, `--halfwidths-kb`) should either remain simple for standalone use or be aliased to the new `rms-*`/`growth-*` names to avoid "vocabulary drift" when developers use them directly.

### 2. Universal Flag Consolidation (New Priority Candidate)
Beyond the "Universal" detection flags already identified (`--shape-score-threshold`, etc.), several execution/meta flags should be moved to a dedicated **Universal / Execution** help group:
- `--threads`, `--rng-seed`, `--verbose`, `--debug`, `--chromosomes`, `--out-dir`, `--out-prefix`.
- These are currently in "Input / Output", "Advanced", or scattered. Moving them to a group at the top or bottom of the help output (as discussed in BRAINSTORM "Flag architecture") improves the scannability of the detection groups.

### 3. Test Suite Blast Radius (The "Big One")
The impact on `tests/` is significant and needs explicit sub-tasks:
- **`tests/test_pipeline.py`**: Many tests assert specific help strings or use the old flags in `check_run` calls. These will all fail.
- **Bash Integration Tests**: The following scripts use flags targeted for renaming:
    - `tests/run_twin_peak_test.sh` (uses `--z-thresh-multi`)
    - `tests/scan_growth_methods_ds1.sh` (uses `--z-thresh-multi`)
    - `tests/run_full_chrII_test.sh` (uses `--min-shape-score`)
    - `tests/test_strict_bic.sh` (uses `--strict-bic`)
- **Utility Scripts**: `tests/compare_strict_bic.py` and `tests/optimize_single_params.py` (which likely uses `onionskin.py` via subprocess) need updates.

### 4. Documentation & Metadata Gaps
- **`Makefile`**: The `make strict-bic` target description in the `help` section needs to be updated to reference `--shape-score-strict-bic`.
- **`tests/full_chrom_training_data/README.txt`**: Mentions `--z-thresh-multi`.
- **`onionskin_core/aps.py`**: Docstrings (e.g., line 514) still reference `--no-aps-singleton-guard`. These should be updated to reflect the new API.

### 5. HMM Shape Filter "Placeholder" logic
I support the idea of adding `--hmm-shape-score-threshold` and `--hmm-shape-score-strict-bic` as placeholders in the HMM section. Even if the HMM engine doesn't yet implement a shape filter, exposing the flags (and having them default to the universal values) makes the CLI feel "complete" and ready for the Phase 11.5/11.7 analytical landing.

### Principles Check:
- **Atomize steps**: The transition from `onionskin.py` to auxiliary scripts is a clear "next step" for vocabulary consistency.
- **No backwards cramming**: Flag updates in `posterior` runs should correctly use the new names in the generated `command` metadata in READMEs.
- **Do it the same**: Supporting `rms` as a synonym across *all* scripts (including `summit_inspector.py`) is critical for this principle.

**Sign-off:** Gemini CLI (v0.14.xx) — Audit complete. Ready for integration into Phase 14 SPEC.


---


## Agent 1 Triage — Round 3 (Copilot + Gemini feedback)

### Summary

Both Copilot and Gemini provided substantive audit findings. The vast majority were accepted and
incorporated into `PHASE14_BRAINSTORM.md`. Two points were rejected on design grounds.

---

### Accepted findings

**From Copilot:**

1. **`test_help_runs()` is a full snapshot test, not a targeted check** — ACCEPTED.
   Confirmed at `tests/test_pipeline.py:41–60`: lines 57 and 60 assert the exact old
   section header strings. Added an explicit note to Priority 14.6 that this test requires
   a complete snapshot refresh, not just a few string substitutions. This is the largest
   single-test blast radius item in the suite.

2. **`scripts/summit_inspector.py` `--pipelines` vocabulary** — ACCEPTED.
   Confirmed that `summit_inspector.py` has its own `PIPELINE_ORDER`, `PIPELINE_LABELS`,
   and `_parse_pipeline_list()`. Added specific update instructions to Priority 14.8 covering
   what each structure needs: add `"rms"` normalization in `_parse_pipeline_list()`, add
   `"rms"` entry to `PIPELINE_LABELS`, update `--pipelines` help text. The `rcn-mean-shift`
   internal token is preserved; `rms` is the user-facing alias only.

3. **`growth_model_engine.py:main()` standalone CLI scope** — ACCEPTED.
   The RMS standalone CLI was already covered in Priority 14.7, but the parallel growth
   standalone CLI was not. Added Priority 14.10 (growth standalone scope decision). Explicit
   decision: Phase 14 does NOT rename engine-native flags; adds a crosswalk help note;
   documents that `_build_ms_argv()` is the intentional translation layer between top-level
   and engine-native names.

4. **Additional low-visibility blast radius surfaces** — ACCEPTED.
   Confirmed and added to Priority 14.6:
   - `tests/optimize_single_params.py`: search grids keyed by old `z-thresh-single`,
     `trend-kb-single`, `smooth-kb-single`, `halfwidths-kb-single` flag names
   - `tests/full_chrom_training_data/README.txt`: mentions `--z-thresh-multi 4.0`
   - `onionskin_core/aps.py:514`: docstring mentions `--no-aps-singleton-guard`
   - `Makefile` help text for `strict-bic` target: references `--strict-bic` by name

**From Gemini:**

1. **`summit_inspector.py` specifics (PIPELINE_ORDER, PIPELINE_LABELS, `_parse_pipeline_list`)** — ACCEPTED.
   Gemini identified the same surfaces as Copilot plus the specific structures to update.
   Incorporated the detailed structure names into Priority 14.8.

2. **Test suite blast radius** — ACCEPTED.
   Gemini's point about test suite scope confirmed and consolidates with existing Priority 14.6.
   `tests/optimize_single_params.py` (also flagged by Copilot) added explicitly.

3. **Documentation and metadata gaps (`Makefile`, `README.txt`, `aps.py`)** — ACCEPTED.
   Confirms and duplicates Copilot's finding #4 above; incorporated into Priority 14.6.

4. **HMM shape filter placeholder support** — ACCEPTED (already in brainstorm).
   Gemini's endorsement of the `--hmm-shape-score-threshold` and `--hmm-shape-score-strict-bic`
   placeholders is noted. No change needed — already reflected in Priority 14.3.

---

### Rejected findings

**From Gemini:**

5. **Move `--chromosomes`, `--out-dir`, `--out-prefix` to a Universal / Execution group** —
   REJECTED.
   These are genuinely I/O flags — they control input sources and output destinations, not
   pipeline behavior. The project's section order places "Input / Output" as section #1
   deliberately, as a user-facing concept separate from the behavioral "Universal" section.
   Moving these would blur that distinction without user benefit. The `--threads`, `--verbose`,
   `--debug` etc. in Advanced are already handled by the existing migration plan.

6. **Standalone engine CLI flags aliasing to `rms-*`/`growth-*`** — REJECTED.
   Gemini suggested aliasing `rcn_mean_shift_engine.py:main()` and `growth_model_engine.py:main()`
   flags (e.g. `--z-thresh`, `--dedup-peak-dist-kb`) to the new top-level names. This would
   add unnecessary complexity at the engine level. The explicit design (Priority 14.7 and 14.10)
   is to keep engine CLIs shorter and standalone, with a crosswalk note, and let `_build_ms_argv()`
   be the translation layer in the top-level CLI. This is the correct boundary.

---

**Author:** Claude Code 2.1.98 (claude-sonnet-4-6)


---


## Copilot Audit Round 4 — authoritative doc scope and developer-tool naming boundary

This pass focused on what still looks under-specified after Agent 1 incorporated the
earlier Copilot and Gemini findings into `PHASE14_BRAINSTORM.md`. The main rename plan
is now fairly complete. The remaining gaps are mostly about **where Phase 14's vocabulary
cleanup is supposed to stop**.

1. **The live documentation migration scope is broader than the current implementation notes say.**

   The brainstorm currently says to update `PIPELINE_SPEC.md` Appendix A and update
   `ONIONSKIN_FULL_HANDOFF.md` if the CLI section references old names. The live tree is
   broader than that.

   In `multi-agent/full_instructions/PIPELINE_SPEC.md`, the old names are not confined to
   Appendix A. They also appear in:
   - step narratives and formulas (`--min-shape-score`, `--strict-bic`, `--dedup-peak-dist-kb`)
   - default/calibration sections (`--z-thresh-single`, `--z-thresh-multi`)
   - parameter tables (`--no-aps-singleton-guard`, `--rcn-mean-shift-norm-mode`, etc.)

   In `multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md`, they also appear across
   multiple conceptual sections, not just one CLI summary block:
   - shape-filter discussion
   - dedup discussion
   - APS singleton-guard discussion
   - historical default/calibration notes
   - example `make strict-bic` usage text

   **Why it matters:** these are still live, authoritative instruction surfaces used by agents
   and by future planning/implementation passes. If Phase 14 updates only one appendix and one
   nominal CLI section, the repo will still keep teaching the old vocabulary in its main internal
   reference docs.

   **Suggested routing:** make the brainstorm explicit that both
   `multi-agent/full_instructions/PIPELINE_SPEC.md` and
   `multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md` are **full live-doc migration
   surfaces** for Phase 14, not just spot-fix targets. Only historical records (old changelog,
   archived plans, old handoff history) should intentionally retain the old flag names.

2. **Phase 14 still needs an explicit keep-or-rename decision for developer-facing helper names that embed old flag terms.**

   The brainstorm already covers updating command strings and help text, but it does not yet say
   whether developer-facing helper names themselves should stay historical or be renamed when the
   CLI flag changes. The strongest live examples are:
   - `tests/test_strict_bic.sh`
   - `tests/compare_strict_bic.py`
   - `make strict-bic`

   Right now the plan updates the flag spelling inside those surfaces, but not the surface names.

   **Why it matters:** without an explicit decision, Phase 14 may end with a split vocabulary where
   user-facing CLI is fully renamed to `shape-score-*`, but the most obvious developer/test entry
   points still advertise `strict-bic`. That may be acceptable, but it should be a deliberate
   boundary rather than an accident.

   **Suggested routing:** add a small Phase 14 note or subtask that decides one of the following:
   - keep developer-helper names like `strict-bic` as historical shorthand while updating only the
     runtime flags and help text, or
   - rename those helper surfaces too and document any compatibility aliasing / transition plan.

Overall judgment: the remaining risk is less about missing another flag rename and more about
finishing Phase 14 with the top-level CLI cleaned up while the repo's live reference docs and
developer-facing helper surfaces still preserve the previous naming scheme in uncontrolled ways.

**Author:** GitHub Copilot (GPT-5.4)

---


## Gemini CLI Phase 14 Deep Dive Audit — documentation, auxiliary tools, and internal parity

**Date:** 2026-04-21  
**Status:** Audit complete.

### 1. Documentation & Specification Blast Radius
The transition to a unified CLI requires a comprehensive documentation migration.
- **`PIPELINE_SPEC.md` & `ONIONSKIN_FULL_HANDOFF.md`**: These files contain nearly 50 references to legacy flags (e.g., `--z-thresh-single`, `--strict-bic`). A full migration pass is required for these live-doc surfaces.
- **`ROADMAP.md`**: Phase descriptions should be updated to use the new vocabulary to maintain consistency with the v0.14.xx implementation.

### 2. Auxiliary Tool & Test Synchronization
- **`scripts/summit_inspector.py`**: Requires updates to `PIPELINE_ORDER`, `PIPELINE_LABELS`, and `_parse_pipeline_list` to support the `rms` synonym.
- **`scripts/aps_cluster_experiments.py`**: Transition hardcoded `--no-aps-singleton-guard` to `--aps-singleton-guard off`.
- **`tests/test_pipeline.py`**: `test_help_runs()` is a major breaking point. Recommend switching to a snapshot-style assertion for help-text headers and group structures to handle broad layout changes gracefully.
- **`scripts/generate_hmm_v2_references.py`**: Check for hardcoded `onionskin.py` calls using legacy flags.

### 3. Internal Engine Parity & Translation
- **Translation Layer**: `_build_ms_argv()` and `_build_detect_extra()` in `onionskin.py` must be meticulously updated to map new `args.rms_*` and `args.growth_*` attributes back to the `--z-thresh` / `--halfwidths-kb` / etc. flags expected by the sub-engines.
- **HMM Parity**: Add `--hmm-shape-score-threshold` and `--hmm-shape-score-strict-bic` as opt-in/placeholder flags to maintain symmetry in the "Shape scoring" help group across all three pipelines.

### 4. CLI Policy: "Hard Break" Redirections
- Implement custom `argparse.Action` classes for all deprecated flags. Instead of silent aliasing or generic "unrecognized argument" errors, provide a specific "Hard Break" message: `Error: --z-thresh-single is deprecated. Use --rms-z-thresh instead.`

### 5. New Idea: `--validate-flags`
- Add a developer-only `--validate-flags` check that scans `sys.argv` for legacy strings and warns if internal scripts are still using old vocabulary. This helps catch drift in `tests/` and `scripts/` that aren't covered by automated help-checks.

**Author:** Gemini CLI (Pro 1.5)


---


## Agent 1 Triage — Round 4 (Copilot Round 4 + Gemini Round 4 feedback)

### Summary

Both Round 4 audits focused on scope clarification — particularly the live documentation
migration surface and developer-helper naming. All substantive new findings were accepted.
One speculative feature idea was deferred. One call-site correction was discovered independently
during verification.

---

### Accepted findings

**From Copilot Round 4:**

1. **PIPELINE_SPEC.md and ONIONSKIN_FULL_HANDOFF.md are full migration surfaces, not spot-fixes** — ACCEPTED.
   Confirmed that old flag names appear in step narratives, parameter tables, calibration sections,
   and conceptual discussions — not just Appendix A and one CLI block. Added Priority 14.13 with
   explicit full-pass instructions for both documents plus ROADMAP.md. The scope boundary is now
   documented: only CHANGELOG.md (pre-Phase 14 entries), archived plans, and legacy compatibility
   comments in code should intentionally retain old names.

2. **Developer-helper surface naming decision (`test_strict_bic.sh`, `compare_strict_bic.py`, `make strict-bic`)** — ACCEPTED.
   Added Priority 14.11 with an explicit decision:
   - **Keep** `test_strict_bic.sh` and `compare_strict_bic.py` file names (git history cost
     outweighs benefit for internal dev tools).
   - **Rename** `make strict-bic` → `make shape-score-bic` (Makefile targets are user-facing
     via `make help`; keeping `strict-bic` there after the CLI vocabulary shift creates an
     uncontrolled split).

**From Gemini Round 4:**

3. **`_build_detect_extra()` and `_build_ms_argv()` as separate translation layer sites** — ACCEPTED.
   The brainstorm previously mislabeled lines 2134-2139 as "same function" as `_build_ms_argv()`.
   Corrected in Priority 14.1: `_build_detect_extra()` at lines 2131-2144 is a separate function
   that also reads old attribute names. Additionally, `_refine_kwargs()` at lines 2147-2161 was
   found to be a third translation call site (not flagged by any audit; found during independent
   code verification). Added all three to the key-locations list.

4. **`scripts/generate_hmm_v2_references.py` — check for legacy flag usage** — ACCEPTED (as a
   verification task). Verified: the script contains no legacy flag names. Added to Priority 14.8
   as an explicitly pre-cleared surface so implementors do not need to re-verify.

5. **Confirmation that `PIPELINE_SPEC.md` and `ONIONSKIN_FULL_HANDOFF.md` need full passes** —
   ACCEPTED. Aligns with and reinforces Copilot's finding #1 above; incorporated into Priority 14.13.

6. **"Hard Break" deprecation error messaging** — ACCEPTED as a style clarification. Updated
   implementation notes to note that both the pre-parse `sys.argv` scan and the `argparse.Action`
   approach are valid; the key requirement is that all deprecated flags produce a specific redirect
   message rather than a generic argparse error. Pick one mechanism and use it consistently.

7. **HMM shape filter placeholders (`--hmm-shape-score-threshold`, `--hmm-shape-score-strict-bic`)** —
   already in brainstorm (Priority 14.3). Gemini's endorsement noted; no changes needed.

---

### Rejected findings

**From Gemini Round 4:**

8. **`--validate-flags` developer mode** — REJECTED for Phase 14 scope.
   This would add a public CLI surface (`--validate-flags`) whose only purpose is to catch
   developer-side drift in test/script invocations. The pre-parse deprecation error mechanism
   already serves this purpose at zero public API cost — any old flag in `sys.argv` immediately
   produces a named redirect error. Adding `--validate-flags` increases public CLI surface area
   without meaningful benefit over the existing deprecation-catch approach. Deferring to
   `multi-agent/BRAINSTORM.md` as a speculative developer-tooling idea.

---

### Independent finding (during verification)

- `_refine_kwargs()` at `onionskin.py:2147-2161` reads `args.z_thresh_single`,
  `args.min_shape_score`, `args.strict_bic`, `args.dedup_peak_dist_kb`, and
  `args.growth_rcn_smooth_bins` — all of which are renamed in Phase 14. This function was
  not flagged by any audit pass. Added to Priority 14.1 key-locations list.

---

**Author:** Claude Code 2.1.98 (claude-sonnet-4-6)


---


## Copilot SPEC Audit — Phase 14 SPEC vs Brainstorm

### Verdict

The condensation from 13 brainstorm priorities to 11 implementation-ordered SPEC priorities is
mostly sound, and I do not see a brainstorm priority that was dropped outright. The remaining
problems are narrower: the live SPEC still contains unresolved self-audit TODOs that were not
folded back into the operative priority bodies, and one dependency edge is still underspecified.
Because of that, I do not consider the SPEC fully implementation-ready yet.

### Findings

1. **The self-audit resolutions were not actually integrated into the operative priorities.**

   The self-audit section still contains live instructions of the form "Resolution: Add to
   Priority ..." rather than already-resolved priority text. The biggest examples are:
   - explicit override-inheritance wiring for new per-pipeline flags in Priority 14.2
   - explicit boolean-to-string migration guidance for `args.shape_score_strict_bic` in Priority 14.2
   - explicit "no effect yet" help-text wording for the HMM placeholder flags in Priority 14.1
   - explicit `multi-agent/KNOWN_ISSUES.md` coverage in Priority 14.10

   This matters because the surrounding framing and recent tracking imply those six self-audit
   gaps were already resolved in-place. Right now the SPEC's operative instructions and its own
   self-audit notes still disagree.

   **Recommendation:** fold each of those resolution lines into the actual target priority body,
   then trim or rewrite the self-audit so it no longer contains live "Add to Priority ..." TODO
   text.

2. **Priority 14.10 has an incomplete dependency chain for the doc migration pass.**

   Priority 14.10 currently depends on 14.2 only, but its own scope explicitly includes doc
   references such as historical `make strict-bic` examples. That naming decision is not settled
   until Priority 14.9 (`strict-bic` -> `shape-score-bic`). In practice, the live-doc pass should
   happen only after the helper-surface naming decision is fixed.

   **Recommendation:** make 14.10 depend on 14.9 as well, or explicitly state that the live-doc
   migration is the final naming pass after 14.2, 14.5, and 14.9 are settled.

3. **Readiness is slightly overstated even though coverage is otherwise good.**

   I do not see a missing brainstorm priority, and the implementation order is generally better
   than the brainstorm order for execution. The blocker is not missing scope; it is that a few
   important pressure-test findings are still parked in the audit section instead of the main
   priority text. Once those are folded in, the SPEC should be ready to hand off.

**Author:** GitHub Copilot (GPT-5.4)


---


## Agent 1 Triage Round 1 — SPEC Stage (v0.14.09)

### Accepted findings

**From Copilot SPEC Audit:**

1. **Self-audit resolutions not folded into operative priority bodies** — ACCEPTED.
   All six "Resolution: Add to Priority X" items from the v0.14.07 self-audit have been folded
   directly into the relevant priority bodies. Priority 14.1 now explicitly states that HMM
   placeholder flags must have "no effect yet" help strings with required wording. Priority 14.2
   now contains the override-inheritance wiring pattern (with code example), the `strict_bic`
   boolean conversion requirement at all call boundaries, and a scope boundary statement
   clarifying that `onionskin_core/` internal function parameter names are not renamed.
   Priority 14.10 now includes `KNOWN_ISSUES.md` in scope. The self-audit section has been
   converted from a live-TODO list to a resolved audit log table.

2. **Priority 14.10 dependency on 14.9 missing** — ACCEPTED.
   Dependency chain updated: 14.10 now depends on 14.2 AND 14.9. Rationale: live docs
   (`ONIONSKIN_FULL_HANDOFF.md`) contain examples referencing `make strict-bic`, and that name
   is not settled until Priority 14.9 renames the Makefile target to `make shape-score-bic`.

### Independent audit findings (all accepted and folded in)

3. **`optimize_single_params.py` had 6 old RMS keys; SPEC listed only 4** — found during
   code grep. Lines 299–315 also contain `"peak-search-kb-single"` and `"window-kb-single"`.
   Priority 14.7 updated to list all 6 keys.

4. **4 missed `getattr(args, ...)` call sites** — `getattr(args, "dedup_peak_dist_kb", ...)` at
   line 462, `getattr(args, "growth_rcn_smooth_bins", ...)` at lines 532 and 2429, and
   `getattr(args, "min_shape_score", ...)` at line 605 — all in `onionskin.py` but using
   `getattr` syntax, not dot notation. The existing grep command would have missed them.
   Added explicitly to Priority 14.2 call-site list and added a supplemental `getattr` grep
   command.

5. **Makefile `.PHONY` declaration (line 1) also lists `strict-bic`** — not mentioned in
   Priority 14.9. Added.

6. **All 4 agent instruction files reference `make strict-bic`** — `CLAUDE.md:76`,
   `AGENTS.md:76`, `GEMINI.md:81`, `.github/copilot-instructions.md:76` all have
   `| Shape filter / BIC | \`make strict-bic\` |`. Per AGENT_CONVENTIONS.md all four must be
   updated together. Added to Priority 14.9 with explicit file/line table.

7. **`multi-agent/project_context/DECISIONS.md` line 61 references `--strict-bic`** — BIC
   decision entry text says `"Strict mode (--strict-bic): k=0 for flat, k=1 for triangle."`.
   Added to Priority 14.10 scope.

8. **`dev/` directory and `onionskin_core/` scope boundary** — neither was explicitly excluded
   from the Phase 14 migration scope. Added explicit scope boundary notes to Priority 14.2
   (onionskin_core/ internal parameters not renamed) and Priority 14.10 (dev/ out of scope).

9. **`run_twin_peak_test.sh` line values corrected** — line 128 uses `5.0` not `4.0`, line 135
   uses `5.5`. Priority 14.7 updated to show the actual values.

---

**Author:** Claude Code 2.1.98 (claude-sonnet-4-6)


---


## Copilot SPEC Audit Round 2 — Phase 14 re-audit after v0.14.09

### Verdict

The main readiness blockers from my prior audit do appear to be resolved in the live SPEC. I do
not see a dropped brainstorm priority, the dependency chain is now materially cleaner, and the
current SPEC is very close to implementation-ready. I found one remaining live-doc scope gap that
should be addressed or explicitly declared out of scope before closing SPEC engineering.

### Findings

1. **`README.md` is still missing from the live-doc migration scope.**

   Priority 14.10 now covers the major live documentation surfaces (`PIPELINE_SPEC.md`,
   `ONIONSKIN_FULL_HANDOFF.md`, `ROADMAP.md`, `DECISIONS.md`, `KNOWN_ISSUES.md`), but it still
   omits the top-level `README.md`. That is a real gap against repo conventions and the live
   user-facing surface.

   Evidence from the current repo:
   - `README.md` says `--pipelines all` runs `growth + rcn-mean-shift + hmm`
   - Phase 14 makes `rms` the canonical short-form user-facing token for the same pipeline
   - `multi-agent/AGENT_CONVENTIONS.md` explicitly treats `README.md` as a maintained user-facing
     surface when major user-facing CLI capability changes land

   I am not arguing that README must rename internal directory names like `03-rcn-mean-shift/` or
   module names like `rcn_mean_shift_engine.py`. The issue is narrower: user-facing CLI wording in
   README should either adopt the canonical `rms` vocabulary where appropriate or explicitly note
   the `rms` / `rcn-mean-shift` equivalence.

   **Recommendation:** add `README.md` to Priority 14.10 with a scoped instruction such as:
   review top-level user-facing CLI text for the new canonical `rms` vocabulary, especially
   `--pipelines` descriptions, while preserving internal path/module names that intentionally stay
   `rcn-mean-shift`.

### Non-blocking note

I also checked auxiliary evaluator CLIs for adjacent vocabulary drift. `tests/eval_summit_precision_v2.py`
still exposes suppressed legacy alias flags (`--rcn-summit-policy`, `--rcn-early-stages`) on its
own local argparse surface. I do **not** currently consider that a Phase 14 blocker because this
phase is scoped around the onionskin CLI plus explicitly named supporting surfaces, and this tool
uses a tool-local API rather than the top-level onionskin parser. If you want Phase 14 to fully
normalize auxiliary evaluation CLIs too, that should be added explicitly to Priority 14.8 as a
scope expansion rather than assumed implicitly.

### Closeout read

If `README.md` is added to Priority 14.10, I think the SPEC is ready to close the SPEC
engineering stage and wait for your approval to move into the audit / implement loop. If you
prefer to keep README out of scope, I recommend documenting that scope boundary explicitly in the
SPEC before calling it final.

**Author:** GitHub Copilot (GPT-5.4)

---

## Agent 1 Triage Round 2 — SPEC Stage (v0.14.11)

### Findings evaluated

| # | Source | Finding | Decision |
|---|--------|---------|----------|
| 1 | v0.14.10 Copilot audit | `README.md` missing from Priority 14.10 — line 56 uses `rcn-mean-shift` in user-facing `--pipelines` description | **Accepted.** Added `README.md` section to Priority 14.10 with scoped instruction: update user-facing CLI vocabulary (`--pipelines` descriptions and pipeline token examples), preserve internal names (`03-rcn-mean-shift/`, `rcn_mean_shift_engine.py`). |
| 2 | v0.14.10 Copilot audit | `tests/eval_summit_precision_v2.py` has `--rcn-summit-policy`/`--rcn-early-stages` SUPPRESS aliases with `rms_` dest names — Copilot flagged as potential Phase 14 scope gap | **Accepted as non-blocking, explicit scope boundary added.** Verified: aliases already map to `rms_summit_policy`/`rms_early_stages` destinations — the `rms_` vocabulary is already in place. This is a tool-local argparse API, not the main onionskin CLI. Added explicit out-of-scope declaration to the Priority 14.10 scope boundary section. |

### Independent review

No additional gaps found. After Round 2 additions:
- All user-facing CLI surfaces (main parser, README, live docs) are covered.
- All internal scope boundaries (onionskin_core/, dev/, eval scripts, historical entries) are explicitly declared.
- Dependency chain and implementation order remain correct.

### Assessment

SPEC is implementation-ready. No further SPEC engineering rounds needed.

**Author:** Claude (claude-sonnet-4-6)
