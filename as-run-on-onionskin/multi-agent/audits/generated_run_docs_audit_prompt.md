# GENERATED RUN DOCS AUDIT PROMPT

Audit the generated run-folder documentation path for accuracy and usefulness.

Primary targets:

- `onionskin_core/readme.py`
- `onionskin_core/output_layout.py`

Generated artifacts in scope:

- run-level `README.md` files written into output directories
- `00-INDEX.md` files written into grouped run directories

---

## PRIMARY RULE

The code that generates run-folder documentation is the source of truth. The audit must ensure that generated run docs match the actual emitted layout and help a user navigate a results directory correctly.

- Start from generator code and output-layout code, not from the current prose alone
- Edits go to the run-doc generator code and templates only
- Never fix unrelated pipeline logic during this audit
- If the generated docs expose a real code/output mismatch, either update the generated docs to describe the current truth or report the code bug explicitly if the generator cannot honestly describe the current behavior cleanly

---

## PURPOSE OF THIS AUDIT

This audit is about **generated run-folder usability and correctness**.

It must answer:

1. Does the generated run README describe the outputs a user will actually see in a run directory?
2. Does `00-INDEX.md` correctly describe the numbered directory structure and the distinction between shared outputs, grouped outputs, optional side pipelines, and top-level HMM outputs?
3. If a user opens a completed run folder for the first time, can they quickly tell what is final, what is intermediate, and where the main results live?

The goal is navigability and correctness, not full algorithm-spec detail.

---

## SCOPE RULES

- You may read any relevant code needed to determine emitted output structure and generator behavior
- You may inspect output-writing code across the pipeline when needed to verify that generated docs mention the right files/directories
- You may update only the code/templates that generate run-folder documentation and indexes
- Do not edit `README.md`, `PIPELINE_SPEC.md`, or roadmap/spec files during the audit itself unless the user explicitly expands scope
- Audit bookkeeping is updated after the audit completes

---

## TOKEN EFFICIENCY RULES

- Read each file at most once per session
- Use targeted search before opening large files
- Prioritize layout builders and documentation writers first
- Build a checklist of emitted directories/files before editing generator prose
- Fix all genuine findings in a single pass when practical

---

## REQUIRED AUDIT METHOD

### Phase A — Build the emitted-layout checklist first

Before editing generated-doc code, inventory the current emitted run structure.

At minimum, check:

1. Grouped output layout in `onionskin_core/output_layout.py`
2. Which files stay at grouping top level versus moved into subdirectories
3. What `00-INDEX.md` currently claims
4. What run-level README generation in `onionskin_core/readme.py` currently claims
5. Posterior-run helper files and where they land
6. Top-level HMM output location and whether it is grouped or not
7. Optional per-stage side-pipeline outputs
8. Which directories are explicitly internal intermediates versus user-consumable outputs

For each category, classify generated-doc coverage as:

- `accurate`
- `stale`
- `missing`
- `misleading`

### Phase B — Audit by user task

Check whether a user can answer these questions from the generated docs:

1. Where are the main call tables?
2. Where are the main BED outputs?
3. Which directories are shared foundation vs pipeline-specific?
4. Which outputs are optional and appear only under certain flags?
5. Where do posterior outputs live?
6. Where do HMM outputs live?
7. Which directories are internal intermediates that most users can ignore?

If the answer is "no" for any question, that is a real finding.

### Phase C — Edit generated-doc logic/templates

When fixing the run docs:

- Keep prose short and practical
- Prefer navigational clarity over exhaustive file inventories
- Explicitly identify internal intermediates when that reduces confusion
- Distinguish top-level HMM outputs from grouped outputs
- Distinguish prior vs posterior outputs clearly
- Preserve deterministic output naming and stable templates

---

## SOURCE FILES TO PRIORITIZE

All under `/Users/johnurban/searchPaths/github/onionskin/` unless otherwise noted.

### Required first-pass files

```
onionskin_core/output_layout.py
onionskin_core/readme.py
onionskin.py
```

### Read as needed

```
onionskin_core/engines/hmm_engine.py
onionskin_core/notebooks.py
onionskin_core/posterior.py
onionskin_core/aps.py
onionskin_core/overlap.py
onionskin_core/timing.py
onionskin_core/single_engine.py
onionskin_core/engines/multistage_engine.py
multi-agent/AGENT_CONVENTIONS.md
```

---

## REQUIRED CHECKLIST

### Output-layout correctness

- Grouped directories match `build_output_layout()`
- `00-INDEX.md` matches the actual grouped layout
- Top-level HMM placement is described correctly
- Posterior helper files are described correctly
- Optional per-stage side-pipeline outputs are not mislocated
- Top-level kept files are not described as if they were moved into subdirectories

### User navigability

- Generated docs point users to main results first
- Internal intermediates are labeled as such when appropriate
- Shared vs optional branches are explained clearly
- HMM outputs are not conflated with prior/posterior grouped outputs

### Drift detection

- No stale directory names remain
- No stale numbering scheme remains
- No stale assumptions about plots/notebooks locations remain

---

## FINDING FORMAT

Return numbered findings with:

1. category (`RunREADME`, `OutputIndex`, `OutputLayout`, `Posterior`, `HMMPlacement`, `SidePipeline`, `Navigability`, `Terminology`)
2. coverage status (`stale`, `missing`, `misleading`)
3. code reference
4. generated-doc/template reference
5. exact fix to apply

Do not report trivial wording nits.

---

## IMPLEMENTATION RULE

After identifying all genuine findings, update the generated-doc code/templates in a single pass when practical.

Prefer changes that improve first-run usability without bloating the generated docs.

---

## POST-AUDIT DELIVERABLES

After the generator/template edits are complete:

1. Update `CHANGELOG.md`
2. Update `multi-agent/AUDIT_HISTORY.md`
3. Update `multi-agent/project_context/HANDOFF.md`
4. Update `multi-agent/project_context/TASK.md`

The changelog entry should explicitly say this was a generated run-doc/output-index audit and summarize the user-facing navigation issues fixed.

---

## FINAL REPORT

Return:

1. Summary of generated-doc behavior changed
2. Any code/output bugs found but not fixed
3. Any intentional remaining simplifications in the generated docs
4. Brief checklist summary proving both the run README and the grouped output index were reviewed against the real emitted layout
