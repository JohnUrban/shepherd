# PIPELINE_SPEC COMPLETENESS AUDIT PROMPT

Audit `multi-agent/full_instructions/PIPELINE_SPEC.md` for completeness and accuracy against the current codebase.

---

## PRIMARY RULE

The code is the source of truth. The spec is the artifact that must be made complete.

- Start from code, not from existing spec headings
- Edits go to `PIPELINE_SPEC.md` only
- Never fix code bugs during this audit
- If code behavior appears wrong, document it accurately in the spec only if that behavior is already user-facing and intentional; otherwise report it as a code bug in the audit summary

This audit must be **full-scope** across the entire spec, not a partial pass focused only on the largest known gap.

---

## PURPOSE OF THIS AUDIT

This is **not** a narrow consistency pass over claims already present in the spec.

This audit must answer both questions:

1. Is every major pipeline behavior currently described in `PIPELINE_SPEC.md` accurate?
2. Are there major code paths, pipeline steps, outputs, flags, guards, or emitted artifacts that are missing or too underspecified in `PIPELINE_SPEC.md` to be operationally useful?

If the answer to (2) is yes, expand the spec.

Stopping after fixing only one obvious area is not acceptable unless the auditor has explicitly shown that all other major areas were checked and found accurate or already sufficiently documented.

---

## SCOPE RULES

- You may read any relevant code needed to determine actual current behavior
- You may read companion planning docs only when needed to disambiguate intended active behavior
- You may update only:
  - `multi-agent/full_instructions/PIPELINE_SPEC.md`
- Do not edit code, tests, README, ROADMAP, or Phase plans during the audit itself
- Audit bookkeeping (`CHANGELOG.md`, `multi-agent/AUDIT_HISTORY.md`, `multi-agent/project_context/HANDOFF.md`, `multi-agent/project_context/TASK.md`) is updated after the audit is complete

---

## TOKEN EFFICIENCY RULES

- Read each file at most once per session
- Use targeted search before opening files
- Prioritize entry points, orchestrators, and output writers first
- Build a temporary coverage checklist from code modules before reading the corresponding spec sections
- Fix all genuine findings in a single pass when practical
- Default to 1 audit iteration; only run a second verification iteration if the first pass produces broad or ambiguous findings

---

## REQUIRED AUDIT METHOD

### Phase 0 — Learn the document architecture before editing

Before proposing any edits, read enough of `PIPELINE_SPEC.md` to identify its organizing principles.

At minimum, determine:

1. What belongs in the top addendum versus the detailed body
2. Where overview flowcharts live
3. Where full pipeline walkthroughs live
4. Where Phase 2 / posterior / appendices live
5. Whether a new feature belongs in an existing section, a new peer section, or an appendix only

Do not insert new major sections without respecting the existing document architecture.

If the spec does not explicitly state its placement rules, infer them from the stable existing structure before editing.

### Phase A — Build the code-driven checklist first

Before assessing the spec, inventory the active code paths and organize them into a checklist.

At minimum, inventory:

1. Top-level pipeline selection and CLI dispatch in `onionskin.py`
2. Single-file pipeline implementation and outputs
3. Single-stage / ratio pipeline implementation and outputs
4. Multistage pipeline implementation and outputs
5. Phase 2 post-processing, APS, plots, notebooks, timing, overlap handling, and output organization
6. HMM pipeline orchestration and outputs

This checklist is mandatory and must cover the **entire** document scope. Do not stop after one category.

For each checklist item, note:

- entry point file/function
- downstream modules called
- major outputs emitted
- user-facing flags/defaults that materially change behavior
- whether the current spec covers it fully, partially, inaccurately, or not at all

Before applying edits, produce an internal coverage ledger for all checklist items. Every item must be classified before the audit can be considered complete.

### Phase B — Compare checklist to the spec

For each checklist item, classify the spec coverage as one of:

- `accurate`
- `stale`
- `missing`
- `underspecified`

`underspecified` means the feature is mentioned, but not with enough detail to let a reader understand the operational behavior, file outputs, or parameter effects.

### Phase C — Expand the spec where needed

When a feature is `missing` or `underspecified`, expand `PIPELINE_SPEC.md` so it becomes operationally useful.

Do not treat broad omissions as out of scope simply because the spec does not already contain a section heading for them.

Respect placement:

- Overview material belongs with the overview sections
- Full step-by-step pipeline descriptions belong with the detailed pipeline body
- Flags/default tables belong in Appendix A unless the behavior also needs narrative explanation in the main body
- Output file inventories belong in Appendix B plus the relevant pipeline/step section when operational understanding requires it

Respect depth and style:

- New detailed pipeline sections should mirror the existing peer sections in `PIPELINE_SPEC.md`
- When a pipeline is described in the main body, the expectation is usually: title, flowchart, step-by-step subsections, operational explanation, outputs/files written, and "Where the code lives"
- For algorithmic steps that are defined by formulas, thresholds, transforms, or explicit scoring rules, include the relevant math or rule statements rather than replacing them with a prose-only summary
- A bare step list or directory list is not sufficient when the peer pipeline sections provide real operational detail
- For HMM or other newly expanded pipeline sections, explicitly check whether normalization, filtering, decoding, thresholding, or aggregation steps should be written with equations or formal rule statements to match the standard used elsewhere in the spec

Do not place detailed pipeline walkthroughs in the top addendum unless the addendum is explicitly where the document stores detailed walkthroughs. The addendum is for current behavioral overrides and concise status notes, not for replacing the main pipeline body structure.

---

## REQUIRED HMM RULE

HMM coverage must be audited step-by-step from code, not treated as a high-level addendum.

Specifically verify whether `PIPELINE_SPEC.md` adequately covers:

1. HMM pipeline entry and dispatch
2. Sequence filtering / chromosome gating before HMM execution
3. Steps 1 through 10 in `onionskin_core/engines/hmm_engine.py`
4. Steps 11 through 16 and their helper modules
5. HMM-specific outputs, tables, plots, notebooks, and directory structure
6. HMM-specific CLI flags and defaults in `onionskin.py`
7. Any environment-variable or backend-selection behavior that affects HMM execution

If HMM coverage is only a compact status block or output list, that is not sufficient. Expand the spec.

When expanding HMM coverage, place it where a peer pipeline section belongs in the document architecture:

- add or update overview material in the overview/flowchart region
- add or update the detailed HMM pipeline section in the main pipeline body
- use appendices only for defaults and complete output inventories

Do not satisfy HMM completeness by appending all details to the top addendum alone.

---

## SOURCE FILES TO PRIORITIZE

All under `/Users/johnurban/searchPaths/github/onionskin/` unless otherwise noted.

### Required code entry points

```
onionskin.py
onionskin_core/engines/single.py
onionskin_core/single_engine.py
onionskin_core/engines/multistage.py
onionskin_core/multistage_engine.py
onionskin_core/engines/hmm_engine.py
```

### Required supporting modules

```
onionskin_core/aps.py
onionskin_core/aps_plots.py
onionskin_core/autodetect.py
onionskin_core/binning.py
onionskin_core/common.py
onionskin_core/detection.py
onionskin_core/hmm_core.py
onionskin_core/hmm_metrics.py
onionskin_core/hmm_fork_travel.py
onionskin_core/hmm_summit_refinement.py
onionskin_core/hmm_ported_analyses.py
onionskin_core/hmm_notebooks.py
onionskin_core/io.py
onionskin_core/modeling.py
onionskin_core/notebooks.py
onionskin_core/output_layout.py
onionskin_core/overlap.py
onionskin_core/posterior.py
onionskin_core/rcn_io.py
onionskin_core/readme.py
onionskin_core/refinement.py
onionskin_core/summaries.py
onionskin_core/timing.py
multi-agent/full_instructions/PIPELINE_SPEC.md
```

### Read only when needed for ambiguity resolution

```
multi-agent/plans/PHASE11_SPEC.md
multi-agent/plans/archived/20260411-PHASE9_SPEC.md
multi-agent/project_context/HANDOFF.md
multi-agent/project_context/TASK.md
CHANGELOG.md
```

---

## REQUIRED CHECKLIST

Hit all of these on the first pass.

### Coverage completeness

- Every active pipeline mode has operationally useful documentation
- Every major orchestrated step has matching spec coverage
- Every user-facing output family is represented
- Every emitted notebook family is represented where relevant
- HMM has more than a directory/status summary; it has real behavioral coverage
- The auditor has explicitly checked all major spec regions, not just one newly expanded area
- New or repaired pipeline sections are comparable in depth to the existing peer pipeline sections where the code warrants that level of detail

### Accuracy

- Formulas, thresholds, clamp bounds, defaults, and NaN/non-finite handling
- Flowchart order and decision branches
- File names, directory locations, and output schemas
- "Where the code lives" paths and function names
- Phase 2 guards and conditional execution in `onionskin.py`
- Appendix A flags and defaults
- Appendix B output files and locations
- Whether mathematical / decision-rule content has been omitted in places where peer sections would normally include it

### Code-to-spec gap detection

- Features present in code but absent from the spec
- Features present in spec but too shallow to explain the code path
- Recently added HMM/Phase 9 outputs that only appear in plans or changelog but not in the spec
- Misplaced content that is factually correct but violates the document's organizing principles

### Structural placement audit

- New overview content is placed in the overview region
- New detailed pipeline content is placed in the main pipeline body
- The addendum remains concise and override-oriented unless the existing structure explicitly says otherwise
- Appendices remain appendices rather than absorbing core narrative pipeline explanation
- Newly added sections match the document's local writing pattern rather than introducing a completely different level of detail or organization without cause

---

## FINDING FORMAT

Return numbered findings with:

1. category (`Completeness`, `Defaults`, `OutputFiles`, `PipelineLogic`, `Flowcharts`, `CodeLocations`, `AppendixA`, `AppendixB`, `HMM`, `Phase2`, `Structure`)
2. coverage status (`stale`, `missing`, `underspecified`)
3. code reference
4. spec reference or note that no corresponding spec section exists
5. exact fix to apply

Do not return trivial wording nits.

---

## IMPLEMENTATION RULE

After identifying all genuine findings, update `PIPELINE_SPEC.md` in a single edit pass when practical.

Prefer expansions that preserve the existing structure and style of the document. If the current structure is insufficient for HMM completeness, add new subsections rather than forcing everything into the existing compact status block.

Before editing, verify that the planned changes cover all checklist categories. If one category receives major expansion, that does not excuse skipping verification of the other categories.

If a change is structurally invasive, prefer moving or rewriting misplaced content rather than layering more content on top of a misplacement.

---

## POST-AUDIT DELIVERABLES

After the spec edits are complete:

1. Update `CHANGELOG.md`
2. Update `multi-agent/AUDIT_HISTORY.md`
3. Update `multi-agent/project_context/HANDOFF.md`
4. Update `multi-agent/project_context/TASK.md`

The changelog entry must clearly state that this was a completeness-oriented, code-first audit and summarize any major newly documented coverage areas, especially if HMM sections were expanded.

---

## FINAL REPORT

Return:

1. Summary of spec coverage that was added or corrected
2. Any code bugs found but not fixed
3. Residual known gaps, if broad expansion is still needed after this pass
4. A brief checklist summary proving that every major pipeline family and major document region was reviewed

---

## PROMPT INTENT

This prompt exists because a consistency-only audit can miss entire regions of the codebase that are merely summarized or omitted in the spec. The goal here is completeness first, then accuracy.