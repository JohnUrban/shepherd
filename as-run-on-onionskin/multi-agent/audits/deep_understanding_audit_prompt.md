# DEEP UNDERSTANDING AUDIT PROMPT

You are working with the onionskin codebase.

Your task is to demonstrate a deep, structural understanding of the system based on the current repository snapshot.

Do NOT modify any files (code, spec, or documentation).

---

## MODEL GUIDANCE

Use **opus** for this audit — it requires cross-document reasoning across PIPELINE_SPEC.md,
ONIONSKIN_FULL_HANDOFF.md, and code simultaneously.

Key file paths:
- `multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md`
- `multi-agent/full_instructions/PIPELINE_SPEC.md`
- `CLAUDE.md` (bootstrap; agent conventions)

---

# GOAL

Provide a comprehensive audit of your understanding of the system, including architecture, data flow, and alignment between documentation and implementation.

---

# SECTION 1 — HIGH-LEVEL ARCHITECTURE

Describe:

1. The overall purpose of onionskin
2. The major components of the system
3. The distinction between:
   - onionskin_core
   - engines (if present)
   - planned future integrations (e.g., PuffStep / HMM integration — ROADMAP Phase 7)

---

# SECTION 2 — PIPELINE FLOW

Describe BOTH:

## A. Single mode pipeline
## B. Multistage mode pipeline

For each:

- Step-by-step flow
- Inputs and outputs at each step
- Key modules involved

---

# SECTION 3 — DATA FLOW

Explain:

- What data structures are passed between modules
- How signal transforms across the pipeline
- Where normalization, detection, and scoring occur

---

# SECTION 4 — KEY MODULE RESPONSIBILITIES

For each important module (e.g., aps, timing, detection, engines):

- What it does
- What inputs it expects
- What outputs it produces

---

# SECTION 5 — SPEC VS IMPLEMENTATION

Compare:

- PIPELINE_SPEC.md
- ONIONSKIN_FULL_HANDOFF.md
- Actual code behavior

Identify:

- mismatches
- ambiguities
- missing documentation
- places where code is more advanced than spec (or vice versa)

---

# SECTION 6 — POTENTIAL RISKS / TECHNICAL DEBT

Identify:

- fragile areas
- confusing structure
- areas likely to cause bugs
- opportunities for simplification

---

# SECTION 7 — READINESS FOR HMM / PUFFSTEP INTEGRATION (PHASE 7)

The user has an external HMM-based amplicon caller (PuffStep) that is a planned integration
target (ROADMAP Phase 7). Explain:

- where HMM/puffstep output would plug into the current pipeline
- what inputs puffstep would need from onionskin (or share with it)
- what outputs it should produce (call set, summit state intervals, fork distance estimates)
- any blockers or prerequisites in the current architecture (e.g., unified RCN processing)

---

# RULES

- Be precise and concrete
- Reference actual modules/files when possible
- Do NOT speculate beyond available evidence
- If unsure, say so explicitly

---

# OUTPUT FORMAT

Return structured sections exactly as above.

