# README AUDIT PROMPT

Audit `README.md` for user-facing accuracy against the current codebase.

---

## PRIMARY RULE

The code is the source of truth. The README is the user-facing artifact that must remain accurate, current, and navigable.

- Start from code and emitted behavior, not from existing README headings
- Edits go to `README.md` only during the audit itself
- Never fix code bugs during the audit
- If code behavior appears wrong, do not "correct" the README to match intended behavior unless that intended behavior is actually implemented; either document the real current behavior or report a code bug in the audit summary

---

## PURPOSE OF THIS AUDIT

This is a **user-facing documentation audit**, not a deep algorithm-spec rewrite.

The audit must answer:

1. Does `README.md` accurately describe the project's current user-visible capabilities?
2. Does it accurately describe the current output layout, key modes, and active module structure?
3. Does it still list completed work as a limitation or future direction?
4. Would a new user get a materially correct picture of how to run and interpret onionskin from the README alone?

The goal is accuracy, navigability, and honesty. The README should not try to mirror `PIPELINE_SPEC.md` in algorithmic depth.

---

## SCOPE RULES

- You may read any relevant code needed to determine active user-facing behavior
- You may read project-planning docs only when needed to determine whether a README claim is stale or superseded
- You may update only:
  - `README.md`
- Do not edit code, tests, ROADMAP, or spec files during the audit itself
- Audit bookkeeping (`CHANGELOG.md`, `multi-agent/AUDIT_HISTORY.md`, `multi-agent/project_context/HANDOFF.md`, `multi-agent/project_context/TASK.md`) is updated after the audit is complete

---

## REQUIRED MAINTENANCE RULES

Apply the README rules from `multi-agent/AGENT_CONVENTIONS.md`:

1. When a capability listed under "Current limitations" or "Future directions" is completed, remove or reclassify it
2. When a major new capability is added, add it to the relevant README section
3. The output directory layout tree must reflect the actual emitted structure
4. The module list must include all active modules
5. The phase/version marker in "Current capabilities" should reflect the current release
6. Keep README user-facing; avoid overloading it with low-level internal implementation detail

---

## TOKEN EFFICIENCY RULES

- Read each file at most once per session
- Use targeted search before opening files
- Prioritize top-level entry points, output layout helpers, and user-facing generators first
- Build a temporary checklist from active user-facing code paths before editing the README
- Fix all genuine findings in a single pass when practical

---

## REQUIRED AUDIT METHOD

### Phase A — Build a user-facing checklist from code

Before editing `README.md`, inventory the major user-facing surfaces.

At minimum, check:

1. Top-level CLI capabilities and major modes in `onionskin.py`
2. Current output layout in `onionskin_core/output_layout.py`
3. Generated run-README behavior in `onionskin_core/readme.py` (only for alignment of terminology/output names)
4. Active major modules under `onionskin_core/`
5. Active HMM pipeline modules and output families
6. Current testing/validation entry points in `Makefile`
7. Current roadmap-state items only where needed to determine whether a README limitation/future-direction bullet is stale

For each category, classify README coverage as:

- `accurate`
- `stale`
- `missing`
- `misleading`

### Phase B — Audit the README by section

At minimum, review:

1. Title / project framing
2. "What onionskin does"
3. "Current capabilities"
4. HMM summary
5. Output organization / output categories
6. Output directory layout tree
7. Main module list
8. Validation section
9. Current limitations
10. Future directions
11. Status / development notes

### Phase C — Edit for user-facing correctness

When fixing the README:

- Prefer short, accurate user-facing explanations over internal implementation detail
- Remove completed work from limitations/future directions
- Update stale phase/version markers
- Update stale directory names and output-tree structure
- Add active user-visible modules only when they help orientation
- Do not turn the README into a second `PIPELINE_SPEC.md`

---

## SOURCE FILES TO PRIORITIZE

All under `/Users/johnurban/searchPaths/github/onionskin/` unless otherwise noted.

### Required first-pass files

```
README.md
onionskin.py
onionskin_core/output_layout.py
onionskin_core/readme.py
Makefile
multi-agent/AGENT_CONVENTIONS.md
```

### Read as needed

```
onionskin_core/engines/hmm_engine.py
onionskin_core/hmm_metrics.py
onionskin_core/hmm_fork_travel.py
onionskin_core/hmm_summit_refinement.py
onionskin_core/hmm_ported_analyses.py
onionskin_core/hmm_notebooks.py
ROADMAP.md
CHANGELOG.md
```

---

## REQUIRED CHECKLIST

### Accuracy

- Major modes and capabilities are described correctly
- Current phase/version marker is not stale
- HMM summary reflects current active outputs, not an obsolete early-stage snapshot
- Output directory tree reflects actual emitted structure
- Directory names match current numbered layout
- Posterior behavior and output placement are described correctly
- Module list includes current active major modules

### Drift removal

- Completed work is not still listed as a limitation
- Reclassified or dropped work is not still listed as an active near-term plan
- Fixed metric bugs are not still described as open bugs
- Historical directory names are removed if no longer current

### User usefulness

- The README tells a new user where key outputs live
- The README distinguishes grouped outputs from top-level HMM outputs when relevant
- The validation section still reflects current recommended checks
- The README remains readable and concise rather than becoming a low-level implementation dump

---

## FINDING FORMAT

Return numbered findings with:

1. category (`Capabilities`, `OutputLayout`, `Modules`, `Limitations`, `FutureDirections`, `Validation`, `Versioning`, `HMM`, `Terminology`)
2. coverage status (`stale`, `missing`, `misleading`)
3. code/source reference
4. README reference
5. exact fix to apply

Do not report trivial wording nits.

---

## IMPLEMENTATION RULE

After identifying all genuine findings, update `README.md` in a single edit pass when practical.

Prefer the smallest set of user-facing edits that restores accuracy.

---

## POST-AUDIT DELIVERABLES

After the README edits are complete:

1. Update `CHANGELOG.md`
2. Update `multi-agent/AUDIT_HISTORY.md`
3. Update `multi-agent/project_context/HANDOFF.md`
4. Update `multi-agent/project_context/TASK.md`

The changelog entry should explicitly say this was a README user-facing accuracy audit and summarize the major drift areas repaired.

---

## FINAL REPORT

Return:

1. Summary of README sections updated
2. Any code bugs found but not fixed
3. Any README areas intentionally left high-level
4. Brief checklist summary proving the major user-facing sections were reviewed
