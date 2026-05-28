# PIPELINE_SPEC AUDIT PROMPT

Audit `multi-agent/full_instructions/PIPELINE_SPEC.md` for consistency with the current codebase.

---

## RULE

The spec must describe what the code actually does.

- Edits go to `PIPELINE_SPEC.md` only
- Never fix code bugs — document them accurately as known issues instead

---

## TOKEN EFFICIENCY RULES (FOLLOW THESE)

- **Read each file at most once per session.** Track what you have already read. Do not re-read.
- **Scope reads to relevant files only.** Only read a file if it is needed to verify a specific spec claim.
- **First pass reads everything** (spec + source files). Subsequent passes re-read only files involved in findings or changes.
- **Fix all findings in a single pass.** Do not launch a second iteration unless the first pass produced ≥5 genuine findings that require cross-checking.
- **Stop early** if a pass produces zero or only trivial findings.
- **Default to 1 iteration.** Only escalate to 2 if the first iteration had significant findings that warrant a verification pass.

---

## MODEL SELECTION RULES

```
Default model:
- Use sonnet for the audit agent (primary workhorse; sufficient for pattern-matching discrepancies)

Escalate to opus only if:
- Cross-module reasoning is complex and the behavior is genuinely ambiguous
- Mathematical correctness is uncertain and formula derivation is required
- Conflicting implementations exist and you cannot determine ground truth by reading alone

Follow-up / verification passes:
- Use haiku for:
  - Lightweight checks ("does this line still match after the fix?")
  - Applying already-identified straightforward edits
  - Verification of simple default values or file paths

Escalation ladder: haiku → sonnet → opus
Safety rule: Do NOT make changes when uncertain. Escalate before modifying anything.
```

---

## PER-ITERATION PROCESS

For each iteration:

1. Launch an Explore agent (**model=sonnet** by default; escalate to opus only if the checklist hits genuine ambiguity)
2. **First iteration:** read ALL source files and the full spec
3. **Subsequent iterations:** read only files involved in changes from the previous pass
4. Return numbered findings with:
   - category (Math / Defaults / OutputFiles / PipelineLogic / Flowcharts / CodeLocations / AppendixA / AppendixB)
   - spec quote
   - code quote
   - exact fix

5. Implement every genuine finding using Edit tool calls (all in one pass)

6. Update `CHANGELOG.md` under the current phase

7. Re-launch only if ≥5 significant findings were found (max 2 total iterations)

---

## SOURCE FILES TO READ (first iteration only)

All under `/Users/johnurban/searchPaths/github/onionskin/`:

```
onionskin.py
onionskin_core/aps.py
onionskin_core/autodetect.py
onionskin_core/binning.py
onionskin_core/common.py
onionskin_core/detection.py
onionskin_core/io.py
onionskin_core/modeling.py
onionskin_core/multistage_engine.py
onionskin_core/output_layout.py
onionskin_core/overlap.py
onionskin_core/rcn_io.py
onionskin_core/readme.py
onionskin_core/refinement.py
onionskin_core/single_engine.py
onionskin_core/summaries.py
onionskin_core/timing.py
onionskin_core/engines/multistage.py
onionskin_core/engines/single.py
onionskin_core/notebooks.py
onionskin_core/posterior.py
multi-agent/full_instructions/PIPELINE_SPEC.md
```

---

## AUDIT CHECKLIST (hit all on first pass)

- Every formula: denominators, signs, thresholds, clamp bounds, NaN/non-finite handling
- Every default value: cross-check argparse in `onionskin.py` AND the engine files
- Every output file: column schemas, exact file names, `others/` vs top-level, `keep_suffixes` in `output_layout.py`
- Every "Where the code lives" entry: function names, file paths, verify existence and behavior
- Flowcharts: node sequences, decision conditions, consistency with detailed sections
- Appendix A: missing CLI flags, incorrect flag names or defaults
- Appendix B: missing files, mislabeled files, incorrect location annotations
- Phase 2 conditions: exact guards in `onionskin.py` for P1/P2/P3/P4

---

## FINAL REPORT

After all iterations, return:

1. Summary of all spec changes made
2. List of any code bugs found (file/function, what's wrong, what the fix would be)
3. Ask the user: fix bugs now, or save as todo for later?

---

## PROMPT HISTORY

- Original prompt designed by Claude Code for itself.
- Revised by ChatGPT for structure.
- Further revised (Phase 3.26) to add model selection rules, token efficiency rules, reduce iteration cap to 2, update source file paths to reflect engine migration from `reference_engines/` to `onionskin_core/engines/`.
