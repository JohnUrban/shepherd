# Multi-Agent Shared Behavior and Memory Infrastructure

This document is a manager-facing map of the file network used to coordinate multiple agents
(Claude Code, Codex, Gemini CLI, Gemini Code Assist, GitHub Copilot, etc.) on the onionskin project.
It is not a rules document — see `multi-agent/AGENT_CONVENTIONS.md` for conventions agents must follow.

---

## Individual agent instruction files

Each agent auto-detects its own bootstrap file. These files are structurally identical
(same required reading list, same dev rules, same workflow) with only agent-specific
sections varying (e.g., model selection table in CLAUDE.md is Claude Code–specific).
Consistency across all files is a critical invariant — if one changes, all must change.

| File | Location | Summary | Use it for | Do not use it for | Must not duplicate | Required reading? | Update frequency |
| ---: | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| CLAUDE.md | repo root | Auto-detected bootstrap for Claude Code | Directing Claude Code; Claude-specific model selection and token efficiency rules | Directing other agents; project context | See `multi-agent/AGENT_CONVENTIONS.md` | Claude Code reads automatically | When conventions change or Claude-specific rules change |
| GEMINI.md | repo root | Auto-detected bootstrap for **Gemini CLI** — CLI reads it hierarchically from working directory upward on every invocation | Directing Gemini CLI; CLI-specific tool vocabulary notes | Directing other agents | See `multi-agent/AGENT_CONVENTIONS.md` | Gemini CLI reads automatically | When conventions change |
| GEMINI.md | repo root | Auto-detected bootstrap for **Gemini Code Assist** (VS Code agent mode) — same file as Gemini CLI; both tools share one bootstrap | Directing Gemini Code Assist in agent mode | Directing other agents; chat-only mode (limited tool access) | See `multi-agent/AGENT_CONVENTIONS.md` | Gemini Code Assist reads automatically in agent mode | When conventions change |
| AGENTS.md | repo root | Auto-detected bootstrap for Codex | Directing Codex | Directing other agents | See `multi-agent/AGENT_CONVENTIONS.md` | Codex reads automatically | When conventions change |
| .github/copilot-instructions.md | `.github/` | Auto-detected bootstrap for GitHub Copilot (agent mode and standard mode) — full file read/write, shell execution, and subagent spawning available in agent mode | Directing Copilot; Copilot-specific notes | Directing other agents | See `multi-agent/AGENT_CONVENTIONS.md` | Copilot reads automatically | When conventions change |

---

## Multi-agent shared infrastructure

Grouped by time horizon and purpose.

### Long-term / Permanent — context and rules

These files change rarely and provide stable grounding for any agent on any session.

| File | Location | Summary | Use it for | Do not use it for | Must not duplicate | Required reading? | Update frequency |
| ---: | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| ONIONSKIN_FULL_HANDOFF.md | `multi-agent/full_instructions/` | Canonical **context** document: architecture, module list, design decisions, biological background, test matrix; §8 "Important Context and Gotchas" for stable codebase pitfalls | Onboarding new agents/sessions; biological and architectural background; understanding how the system works; stable codebase pitfalls | Editorial rules (→ AGENT_CONVENTIONS.md); future plans (→ ROADMAP.md); recent history (→ CHANGELOG.md) | CHANGELOG.md, ROADMAP.md, AGENT_CONVENTIONS.md | Yes — required reading #1 | When significant architecture changes, new modules, or major design decisions occur |
| AGENT_CONVENTIONS.md | `multi-agent/` | Canonical **rules** document: CHANGELOG/ROADMAP format, authorship, timestamps, git commit convention, project_context usage, AGENT_SWOT maintenance | Understanding and enforcing all editorial and process conventions; authorship format; CHANGELOG timestamp rules | Project context, architecture, biology (→ ONIONSKIN_FULL_HANDOFF.md) | ONIONSKIN_FULL_HANDOFF.md | Yes — required reading #4 | When new conventions are established or existing ones strengthened |
| PIPELINE_SPEC.md | `multi-agent/full_instructions/` | Technical specification of all three pipelines: detection steps, output schemas, CLI flags, defaults | Verifying code matches spec; understanding output file schemas; knowing what CLI flags do | Biological background (→ ONIONSKIN_FULL_HANDOFF.md); future plans (→ ROADMAP.md) | ONIONSKIN_FULL_HANDOFF.md (high-level architecture) | Conditional — required when working on pipeline steps, outputs, or CLI flags | Any time pipeline steps, outputs, CLI flags, or defaults change |
| AGENT_SWOT.md | `multi-agent/` | Per-agent Strengths / Weaknesses / Opportunities / Threats assessments; tracks agent capabilities and failure patterns over time | Calibrating trust in agents; diagnosing recurring patterns; manager-level oversight of agent performance | Project design decisions (→ DECISIONS.md); codebase knowledge (→ ONIONSKIN_FULL_HANDOFF.md) | AGENT_CONVENTIONS.md, DECISIONS.md | Not required — manager reference; agents may read when relevant | When agents exhibit new or pattern-establishing behavior |
| BRAINSTORM.md | `multi-agent/` | Conceptual scratchpad: speculative ideas, naming candidates, unconventional directions not yet ready for ROADMAP | Exploring naming questions; capturing preliminary ideas before they are committed; architectural concepts under consideration | Committed plans (→ ROADMAP); settled rationale (→ DECISIONS.md); completed work (→ CHANGELOG.md) | ROADMAP.md, DECISIONS.md | Not required — consulted when exploring future directions or naming questions | When ideas are added, committed elsewhere, or abandoned |

### Long-term / Permanent — history and plans

The authoritative records of what has been done and where things are going.

| File | Location | Summary | Use it for | Do not use it for | Must not duplicate | Required reading? | Update frequency |
| ---: | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| CHANGELOG.md | repo root | Authoritative record of all completed work; reverse chronological; version-tagged, timestamped, with authorship | Understanding what changed and when; version history; cross-referencing with ROADMAP | Future plans (→ ROADMAP.md); design rationale (→ DECISIONS.md); current session state (→ HANDOFF.md) | ROADMAP.md, DECISIONS.md | Yes — required reading #3 (last 2–3 entries sufficient) | Every development session |
| ROADMAP.md | repo root | Bird's-eye plan across phases; summary-level current and historical phase view | Strategic planning; understanding phase structure; checking what is next at a high level; tracking completion | Detailed current-phase planning (→ active SPEC/PLAN under `multi-agent/plans/`); completed history (→ CHANGELOG.md) | Active phase SPEC/PLAN, CHANGELOG.md | Yes — required reading #2 | Update only the active phase block when priorities are added, completed, substantially expanded, or status changes |
| Active phase SPEC/PLAN | `multi-agent/plans/` | Detailed current-phase planning source of truth; verbose implementation queue and scope notes for the active phase | Current-phase planning, scope control, detailed design/ordering notes, marking current work partial/done | Bird's-eye multi-phase summary (→ ROADMAP.md); speculative future ideas (→ BRAINSTORM.md) | ROADMAP.md, BRAINSTORM.md | Conditional — required when working in that active phase | Edit in place during the active phase; archive intact when the phase ends |
| Archived phase SPEC/PLAN files | `multi-agent/plans/archived/` | Immutable detailed provenance for completed phases; archived copies of the former active SPEC/PLAN files | Recovering original detailed design rationale and implementation notes for completed phases | Live planning for the current phase | Active phase SPEC/PLAN | No — consult when tracing older completed work | Created when a phase ends; should not be routinely edited afterward |

### Audit tools

Reusable invocation prompts for specific audit tasks. Not reference documents and not required
reading. Paste or reference them when you want to run the corresponding audit.

| File | Location | Purpose |
| ---: | :--- | :--- |
| `pipeline_spec_audit_prompt.md` | `multi-agent/audits/` | Audit `PIPELINE_SPEC.md` for drift from the codebase — formulas, defaults, output schemas, CLI flags |
| `onionskin_full_handoff_audit_prompt.md` | `multi-agent/audits/` | Audit `ONIONSKIN_FULL_HANDOFF.md` for drift from code, spec, and agent instruction files |
| `deep_understanding_audit_prompt.md` | `multi-agent/audits/` | Structured deep-understanding assessment of the full system — architecture, data flow, spec vs implementation, HMM integration readiness |

### Workflow tools

Reusable multi-agent coordination prompts for structured audit / implement / closeout loops.
Unlike the files in `multi-agent/audits/`, these are not one-shot audit prompts; they define a
role-based collaboration protocol that can be reused across different specs and plans.

| File | Location | Purpose |
| ---: | :--- | :--- |
| `spec_plan_three_role_audit_loop.md` | `multi-agent/workflows/` | Reusable 2-role or 3-role workflow for auditing one spec/plan section at a time, implementing against the written findings, re-auditing, and finishing with a wrap-up audit |

---

### Short-term / Session-level — active state

These files are updated frequently and reflect current or near-current state.

| File | Location | Summary | Use it for | Do not use it for | Must not duplicate | Required reading? | Update frequency |
| ---: | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| HANDOFF.md | `multi-agent/project_context/` | Session bookmark (CRITICAL): last action, current state, mandatory next action; precise bridge between sessions | Resuming unfinished tasks; knowing exactly where the last agent left off; cross-agent handoffs | Large context information; task lists (→ TASK.md); broad scope | TASK.md (may reference but must not restate task lists) | Yes — required reading #5; **MUST be updated at end of every session** | Every session end — no exceptions |
| TASK.md | `multi-agent/project_context/` | Immediate task queue (≤5 tasks) and blockers; tactical, not strategic | Immediate (1–3 step horizon) actionable tasks; blockers; short context notes from recent discussion | Long-term planning; copying from ROADMAP | ROADMAP.md (beyond extracting immediate next steps) | Yes — required reading #6; update as tasks complete or are added | Frequently |
| DECISIONS.md | `multi-agent/project_context/` | Rationale layer: growing list of design decisions, alternatives considered, and why choices were made | Preventing re-litigation of settled questions; understanding why something works the way it does | Automatically repeating past decisions without re-evaluating; duplicating CHANGELOG notes | CHANGELOG.md | No — consulted at decision points; not required startup reading | When design decisions are made or alternatives are rejected |

---

## Notes

### How these files relate to each other
```
Entry points for individual agents:
Individual bootstrap files (CLAUDE.md etc.) ──► each agent reads its own
       │
       ▼
Multi-agent shared infrastructure:

  Rules layer:
  AGENT_CONVENTIONS.md ──► rules all agents must follow governing all files below

  Context and spec (stable):
  ONIONSKIN_FULL_HANDOFF.md  ──► stable context (rarely changes)
  PIPELINE_SPEC.md           ──► technical spec (changes with code)

  History and plans (append/update):
  ROADMAP.md                 ──► bird's-eye what is planned
  active SPEC/PLAN           ──► detailed current-phase plan
  archived SPEC/PLAN files   ──► detailed completed-phase provenance
  CHANGELOG.md               ──► what was done

  Active state (session-level):
  HANDOFF.md  ──► where we are right now (session boundary)
  TASK.md     ──► what to do right now (tactical queue)

  Reference (consulted, not required startup reading):
  AGENT_SWOT.md    ──► how each agent performs: strengths, weaknesses, opportunities, and threats.
  DECISIONS.md     ──► why things are the way they are (decision rationale)
  BRAINSTORM.md    ──► speculative ideas, naming candidates, future directions under consideration

  Audit tools (invocation prompts, not reference documents):
  multi-agent/audits/pipeline_spec_audit_prompt.md         ──► PIPELINE_SPEC drift audit
  multi-agent/audits/onionskin_full_handoff_audit_prompt.md ──► ONIONSKIN_FULL_HANDOFF drift audit
  multi-agent/audits/deep_understanding_audit_prompt.md    ──► full system deep-understanding audit

  Workflow tools (reusable collaboration protocols):
  multi-agent/workflows/spec_plan_three_role_audit_loop.md ──► section-by-section audit / implement / re-audit / wrap-up loop

```


### What agents read on every session startup (required reading list)
1. ONIONSKIN_FULL_HANDOFF.md
2. ROADMAP.md
3. CHANGELOG.md (last 2–3 entries)
4. AGENT_CONVENTIONS.md
5. HANDOFF.md
6. TASK.md
7. PIPELINE_SPEC.md *(conditional — only for pipeline/output/CLI work)*

### The "must not duplicate" principle
Each file owns its domain. The most common violations to watch for:
- Explaining completed work in ROADMAP → that belongs in CHANGELOG
- Writing design rationale in CHANGELOG → that belongs in DECISIONS.md
- Copying ROADMAP priorities into TASK.md → TASK.md extracts immediate steps only
- Restating TASK.md content in HANDOFF.md → HANDOFF references TASK, does not restate

### Temporal structure of the system
| Time horizon | Files |
|---|---|
| Permanent / foundational | ONIONSKIN_FULL_HANDOFF.md, AGENT_CONVENTIONS.md, PIPELINE_SPEC.md |
| Long-term history | CHANGELOG.md, archived phase SPEC/PLAN files |
| Long-term future | ROADMAP.md, BRAINSTORM.md |
| Meta / management | AGENT_SWOT.md, DECISIONS.md, BRAINSTORM.md |
| Audit tools | `multi-agent/audits/` (task-specific audit prompts) |
| Workflow tools | `multi-agent/workflows/` (reusable multi-agent coordination prompts) |
| Session / short-term | HANDOFF.md, TASK.md |
