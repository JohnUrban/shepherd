Run 1–3 autonomous iterations of auditing `multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md` for alignment with:

- current codebase
- PIPELINE_SPEC.md
- agent instruction files (CLAUDE.md / AGENTS.md)

The goal is NOT to re-explain the system.

The goal is to detect:
- drift
- contradictions
- outdated statements
- misleading guidance
- redundant or conflicting instruction layers

Edits go to ONIONSKIN_FULL_HANDOFF.md only.
Do NOT modify code.
Do NOT modify PIPELINE_SPEC.md.

---

# FOR EACH ITERATION

1. Use model=sonnet by default (escalate if needed)

2. Read:
   - ONIONSKIN_FULL_HANDOFF.md (full in iteration 1, targeted after)
   - PIPELINE_SPEC.md (relevant sections only)
   - CHANGELOG.md (recent changes)
   - CLAUDE.md / AGENTS.md / GEMINI.md / `.github/copilot-instructions.md` (agent behavior expectations)
   - Relevant code (targeted)

3. Return numbered findings with:

   - Category:
     (Drift / SpecMismatch / CodeMismatch / AgentMismatch / Redundancy / Ambiguity / Misleading)

   - Handoff quote

   - Source of truth:
     (code / PIPELINE_SPEC / agent files)

   - Issue explanation

   - Exact fix

---

# AUDIT CHECKLIST

## 1. Drift vs Code
- Does the handoff describe behavior that is no longer true?
- Are any modules described incorrectly or incompletely?

## 2. Drift vs PIPELINE_SPEC
- Any contradictions between handoff and spec?
- Any duplicated logic that has diverged?

## 3. Agent Instruction Consistency
- Do model selection rules match CLAUDE.md / AGENTS.md?
- Do token-efficiency rules match current best practices?
- Any conflicting guidance across files?

## 4. Pipeline Description Accuracy
- High-level flow must match real execution
- No need for full mathematical precision (that belongs in spec)
- But must not be wrong or misleading

## 5. Redundancy Control
- Identify duplicated sections that risk divergence
- Suggest consolidation or referencing instead of duplication

## 6. Clarity vs Truth
- Flag statements that are:
  - technically incorrect
  - oversimplified in a misleading way
  - ambiguous in important areas

## 7. Future Directions / Architecture Plans
- Are described plans still accurate?
- Any outdated assumptions about architecture?

---

# ITERATION RULES

- Full read only in iteration 1
- Later iterations:
  - only re-read affected sections
- Limit to 1–3 iterations (default 2)
- Stop early if no high-confidence findings remain
- Fix ALL findings in one pass

---

# COST CONTROL

- Do NOT read entire repo unless necessary
- Use CHANGELOG-driven targeting
- Avoid re-checking validated sections

---

# AFTER COMPLETION

Report:

1. Summary of all handoff updates
2. List of:
   - code vs spec mismatches
   - spec vs handoff mismatches
   (do NOT fix outside handoff)

3. Ask:
   - Should we update PIPELINE_SPEC?
   - Should we fix code?
   - Or accept divergence?
