
# ORCHESTRATOR GUIDE FOR `spec_plan_three_role_audit_loop-v3.md`

This file is a working guide for **the Orchestrator** — the role that drives
the audit-loop cycles by sending prompts to R1/R2/R3 agents and deciding
when to move between rounds.

**Audience under v3.** Under v2, this file was framed as a guide for "the
human orchestrator." Under v3, the Orchestrator role has been formalized as
a phase-spanning AI agent (typically Claude Code Opus Max) that works in
deliberation with the Principal (human; phase owner / scope authority). The
templates and decision guides here are now used by:

- The **AI Orchestrator** as a concrete reference for what prompts to draft,
  in what order, with what payload — when shaping launchers and mid-flight
  pushback notes.
- The **Principal** as a reference for what the AI Orchestrator should be
  doing + what's expected at each stage, to verify the AI Orchestrator is
  on track.

The full Orchestrator role definition lives in
`multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md § The
Orchestrator (phase-spanning AI role; v3)`. This file is the
**concrete-prompt companion** to that role definition.

Use this file alongside:

- `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md` — the
  workflow rules + Templates A-J for the agents
- `multi-agent/workflows/phase-development-system_PDS-v3.md` — the broader
  phase development system covering SOUP → BRAINSTORM → SPEC engineering
  → handoff into the audit/implement cycles this file covers

That first file defines the workflow rules for the agents.
This file shows what to send, when to send it, and how to move from one
role to the next.

**Revision markers:**

- **v3 (current).** Adds the formal Orchestrator role context — file
  audience updated to acknowledge AI Orchestrator + Principal partnership
  (deliberation pattern in audit-loop-v3 § Orchestrator). Cross-references
  updated from v2 to v3 throughout. Strategist template (section 0) reframed
  as the Orchestrator's kickoff act, not a separate Strategist role. v4
  consolidation plan: this file's content + audit-loop-v3 + PDS-v3 will
  merge into a unified `phase_workflow-v4.md` after v3 stabilizes through
  one or two phases. v2 remains at
  `multi-agent/workflows/orchestrator.spec_plan_three_role_audit_loop-v2.md`
  for reference.

- **v2.** Paralleled the v2 changes in `spec_plan_three_role_audit_loop-v2.md`:
  sibling `AUDIT_LOG.md` file, per-cycle CHANGELOG batching,
  substantive-priorities rule, cycle granularity, authorship consolidation at
  cycle closeout, and Role 2 skip-reaudit declaration with
  orchestrator-approved closeout. v1 remains at
  `multi-agent/workflows/orchestrator.spec_plan_three_role_audit_loop-v1.md`.

**CHANGELOG vs DEVLOG note (from v0.14.64):** Throughout this file, every
reference to "the CHANGELOG entry at cycle closeout" should be read as "the
CHANGELOG entry **or** the DEVLOG entry at cycle closeout, by cycle scope."
Cycles whose primary work was onionskin product close to `CHANGELOG.md`. Cycles
whose primary work was dev-system (workflow, conventions, agent files, SPEC
engineering with no code changes) close to `multi-agent/DEVLOG.md`. The two
files share a single `v0.X.YY` version stream. See `AGENT_CONVENTIONS.md` for
the full routing rule.

---

## Core Rule

Do not paste only a tiny fragment of the workflow unless you already know the agent has read the
whole workflow file.

Best practice:

1. Tell the agent to read `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md`
2. Tell it which role it is playing
3. Tell it which template behavior to follow
4. Tell it the target file AND the sibling audit-log file
5. Tell it the target cycle (a substantive priority or a named batched group)
6. Tell it the current round

In practice, your prompt should be:

- whole workflow file by reference
- plus the pertinent template behavior
- plus the current target file + audit-log file
- plus the current target cycle

---

## Key v2 concepts to remember as orchestrator

- **SPEC stays clean.** During the audit-implement-reaudit loop the SPEC is edited only
  for (a) priority status table updates at cycle closeout, and (b) scope changes that
  you approve. All round-by-round audit findings, implementation reports, and closeout
  judgments live in the sibling `AUDIT_LOG.md`.
- **CHANGELOG entries are batched.** One entry per cycle closeout, not per role-round.
  The closing agent consolidates authorship from `AUDIT_LOG.md`.
- **Cycles, not priorities.** A cycle is either one substantive priority OR one named
  batched group of thin priorities. Don't run a full three-role loop on a single flag
  rename — bundle such work into a batched cycle (e.g., "Phase 1 — structural parser
  changes" covering a whole phase of renames).
- **Role 2 may recommend skip-reaudit.** On truly mechanical batched cycles, Role 2 can
  declare the skip criteria met and hand back both a skip-accepted prompt and a
  force-reaudit prompt. **You pick.** Default to full re-audit if any uncertainty.
- **R3 may be deferred to the next cycle's R3.** Light cycles (audit-only, no
  runtime code) may defer their R3 scope to the next cycle's R3 rather than run
  a standalone R3 session. STRATEGY.md names the absorbing cycle; the absorbing
  R3 verifies both cycles' surfaces at closeout. See "Deferred R3" in the
  Cycle Granularity section of `spec_plan_three_role_audit_loop-v3.md`. No
  chaining — one-cycle deferral max.
- **Compaction voids "I already read it."** If the agent's session has been compacted
  since it last read AGENT_CONVENTIONS or its bootstrap file, the re-read is mandatory.
  Re-reads are cheap quality insurance — never prune them.

---

## Quick Launcher Pattern

Use this shell for nearly every round:

```text
Below N and <N> = <phase number to be filled in by user>

Read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md and follow it.

You are acting as {ROLE_NAME}.
Use {TEMPLATE_NAME}.

Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md
Target cycle: <cycle name> or <cycle index>
Current round: {ROUND_TYPE}

{ROUND_SPECIFIC_INSTRUCTIONS}
```

If a round is full-file rather than cycle-specific (e.g., wrap-up audit), omit `Target cycle`.

For supplemental phases (e.g., Phase 14 Supplemental) or other phases whose file names
deviate from the plain `PHASE<N>_*.md` pattern, substitute the concrete filenames
directly (e.g., `multi-agent/plans/PHASE14_SUPPLEMENTAL-SPEC.md`).

---

## Start-To-Finish Pipeline

This is the standard sequence with the v3 Orchestrator role spanning the
whole phase.

**Pre-cycles (Orchestrator already in role; SOUP → BRAINSTORM → SPEC):**
The Orchestrator has been driving the brainstorm + SPEC engineering stages
per `phase-development-system_PDS-v3.md` before reaching this audit-loop
file's territory. By the time the steps below begin, the SPEC is locked
and the Orchestrator is preparing to kickoff cycle execution.

0. **(Once per phase, before the first cycle) Orchestrator's kickoff act —
   Strategist function.** The Orchestrator produces
   `multi-agent/plans/PHASE<N>_STRATEGY.md` per Template I in
   `spec_plan_three_role_audit_loop-v3.md`. Under v3, this is no longer a
   separate Strategist role — it is the Orchestrator's kickoff act when
   transitioning from SPEC engineering to cycle execution. Splits the
   phase into execution-ordered cycles, assigns primary + alternative
   agents to each role per cycle, and names a Final Overseer. See the
   launcher prompt below.
1. Auditor performs initial audit on a cycle → writes findings to AUDIT_LOG_FILE
2. Implementer executes the written instructions → writes implementation report to
   AUDIT_LOG_FILE, declares re-audit-needed OR skip-reaudit-recommended
3. If re-audit: Auditor re-audits and either closes the cycle (SPEC status table update
   + CHANGELOG entry with consolidated authorship) or writes follow-up findings
4. If skip-reaudit accepted by Orchestrator/Principal: Auditor performs
   lightweight spot-check and closes the cycle (Template H)
5. Repeat steps 2-4 until the cycle closes, then move to the next cycle
6. Final Overseer audits the fully worked spec/plan at the end → writes wrap-up audit
   to AUDIT_LOG_FILE. **Final Overseer must be a fresh chat, structurally
   independent from the Orchestrator's accumulated frame.**
7. If Final Overseer finds actionable work, bounce back through:
	- Auditor triage → opens post-wrap-up remediation cycle in AUDIT_LOG_FILE
	- Implementer execution
	- Auditor final closeout (Template G)

The Final Overseer normally does one pass only.

**Post-cycles (Orchestrator wind-down; new in v3):**

8. **Outgoing Orchestrator writes the Succession Briefing** at
   `multi-agent/plans/PHASE<N>_SUCCESSION_BRIEFING.md` per Template J in
   `spec_plan_three_role_audit_loop-v3.md`. Wisdom transfer to the
   next-phase Orchestrator (cross-cycle pattern memory, recurring blind
   spots, deliberation patterns that worked, things that didn't, brainstorm
   seeds, recommended working style). Written AFTER Final Overseer signs
   off, BEFORE phase archive, so Final Overseer's findings can be
   incorporated.
9. **Phase archive.** Principal moves SPEC + AUDIT_LOG + STRATEGY +
   BACKGROUND + SURPRISE_LOG + SUCCESSION_BRIEFING to
   `multi-agent/plans/archived/<date>-PHASE<N>_*.md`.
10. **Outgoing Orchestrator chat closes.** Next-phase Orchestrator
    bootstraps from the archived prior-phase artifacts + Succession
    Briefing.

---

## Standard Copy-Paste Prompts

These are the prompts I recommend using in practice.

### 0. Strategist (Orchestrator's kickoff act under v3) — Phase Plan of Attack (once per phase, before any cycle)

Under v3, Strategist is no longer a separate one-shot role. The Strategist
function is **the Orchestrator's kickoff act** when transitioning from SPEC
engineering to cycle execution. The Orchestrator (typically the same chat
that drove SPEC engineering, or a fresh Orchestrator chat that bootstraps
from artifacts if a handoff has occurred) produces
`multi-agent/plans/PHASE<N>_STRATEGY.md` — a cycle table with primary +
alternative agents for each role, deviation rationale, and a Final Overseer
entry. Drives the rest of the phase.

The launcher prompt below is the form to use whether the Orchestrator is
the same chat continuing through phase, or a fresh chat picking up the
Orchestrator role at this stage.

Assign to a Tier-1 reasoning agent (Claude Code — Opus, Gemini 3.1 Pro
Preview, or Codex — GPT-5.5 (Reasoning: High)). It needs to read the full SPEC and reason
about both cycle granularity and agent fit.

```text
Read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md and follow
Template I — Strategist: Phase Plan of Attack.

Target SPEC: multi-agent/plans/PHASE<N>_SPEC.md
Target audit log: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Output: multi-agent/plans/PHASE<N>_STRATEGY.md

Produce the STRATEGY.md draft in chat. Each role cell (R1, R2, R3) must list
a Primary agent AND at least one Alternative. Wait for my approval before
writing the file to disk.

After the file is written, your closing chat message must also include a
ready-to-paste Role 1 initial-audit launcher prompt (Template A format) for
cycle index 1, with the concrete phase number N filled in — per "Final chat
output — next-action handoff prompt" in Template I.
```

### 1. Auditor — Initial Audit

Use when opening a new cycle.

```text
Below N and <N> = <phase number to be filled in by user>

Read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template A — Role 1 Initial Audit.

Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md
Target cycle: <cycle name> or <cycle index>
Current round: initial audit

Audit this cycle deeply against the live codebase. Determine whether it is closed or open.
If it is open, append audit findings and exact implementation instructions into the cycle
section of AUDIT_LOG_FILE (NOT the SPEC), including file references, function references,
and concrete repair notes. Do NOT write to CHANGELOG.md in this round — CHANGELOG batching
happens at cycle closeout. Read actual code rather than trusting prior reports. Include
scripts/ and tests/ when relevant.


Required checkpoint reads (quality guardrail — do not skip, especially post-compaction):
- If you have not read your agent file (CLAUDE.md, AGENTS.md, GEMINI.md, or
  .github/copilot-instructions.md) this session, or if the session has been compacted
  since you last read it, read it now.
- Same for AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md
- ensure that you have completed all that is expected of you from your role and template.
- emit a git commit block per the "Convention for agent-produced git commit blocks" section in spec_plan_three_role_audit_loop-v3.md (mid-cycle form for non-closeout rounds; closeout form when you write the CHANGELOG/DEVLOG entry), placed before your handoff prompt(s).


```

If you want to speed through a narrow round (single agent doing A→B→C in one session), you
can add:
```text
If this is a narrow, thin-priority batched cycle, then write your audit and plan as Role 1,
Template A. Then implement as Role 2, Template B. Then decide: re-audit as Role 1 Template
C, or skip-reaudit closeout as Role 1 Template H. All in one session. What do you think?
```

### 2. Implementer — Implementation Round

Use after the Auditor has written findings and instructions.

```text
Below N and <N> = <phase number to be filled in by user>

Read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md and follow it.

You are acting as Role 2 — Implementer / Supplemental Auditor.
Use Template B — Role 2 Implementation.

Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md
Target cycle: <cycle name> or <cycle index>
Current round: implementation

Read the audited findings and implementation instructions in the cycle section of
AUDIT_LOG_FILE, then implement them as closely as possible. While working, inspect nearby
code for audit misses in the same surface area. If you need to diverge, document the reason
in your implementation report in the same cycle section. Append your report to
AUDIT_LOG_FILE. Update HANDOFF.md and TASK.md. Report the tests you ran.

CHANGELOG rule for this round: Do NOT write to CHANGELOG.md unless you are recommending
skip-reaudit. If you are recommending skip-reaudit, write the cycle's CHANGELOG/DEVLOG
entry now — you are the de facto closing agent if the skip is accepted. If re-audit
proceeds instead, Role 3 will update your entry at closeout. If re-audit is declared
needed, do NOT write a CHANGELOG entry.

At round end, declare either "re-audit needed" or "skip-reaudit recommended" per the
Skip-Reaudit Criteria in the workflow file. Then emit handoff prompts as follows:

- **If skip-reaudit is recommended:** provide both handoff prompts (re-audit variant and
  skip-reaudit-accepted variant) so the orchestrator can pick.
- **If re-audit is declared needed:** provide only the re-audit handoff prompt (Template C).
  Do not emit the skip-reaudit variant — the orchestrator may override to skip, but that
  decision belongs to the orchestrator, not the implementer.


Required checkpoint reads (quality guardrail):
- If you have not read your agent file this session, or if the session has been compacted
  since you last read it, read it now.
- Same for AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md
- ensure that you have completed all that is expected of you from your role and template.
- emit a git commit block per the "Convention for agent-produced git commit blocks" section in spec_plan_three_role_audit_loop-v3.md (mid-cycle form for non-closeout rounds; closeout form when you write the CHANGELOG/DEVLOG entry), placed before your handoff prompt(s).


```

### 3. Auditor — Re-Audit Round

Use after the Implementer finishes an implementation round AND either declared "re-audit
needed" OR declared "skip-reaudit recommended" that you have decided to OVERRIDE with a
full re-audit.

```text
Below N and <N> = <phase number to be filled in by user>

Read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template C — Role 1 Re-Audit.

Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md
Target cycle: <cycle name> or <cycle index>
Current round: re-audit

Role 2 has finished an implementation round. Re-audit the live code and determine whether
this cycle is now closed or still open. Read the actual changed code rather than trusting
the implementation report. If still open, append new findings and exact repair instructions
into the cycle section of AUDIT_LOG_FILE. If closed: update the SPEC's priority status
table and write the cycle-closeout CHANGELOG entry with consolidated authorship (survey
AUDIT_LOG_FILE's cycle section for all `**Authors:**` lines, deduplicate, user first).
Include scripts/ and tests/ when relevant.


Required checkpoint reads (quality guardrail):
- If you have not read your agent file this session, or if the session has been compacted
  since you last read it, read it now.
- Same for AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md
- ensure that you have completed all that is expected of you from your role and template.
- emit a git commit block per the "Convention for agent-produced git commit blocks" section in spec_plan_three_role_audit_loop-v3.md (mid-cycle form for non-closeout rounds; closeout form when you write the CHANGELOG/DEVLOG entry), placed before your handoff prompt(s).

```

### 3b. Auditor — Skip-Reaudit Cycle Closeout (new in v2)

Use after the Implementer recommends skip-reaudit AND you accept the recommendation.

```text
Below N and <N> = <phase number to be filled in by user>

Read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template H — Role 1 Cycle Closeout After Skip-Reaudit.

Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md
Closed cycle: <cycle just finished (index or name)>
Next target cycle: <next cycle (index or name) or "none — final cycle in plan">
Current round: cycle closeout (skip-reaudit accepted)

Role 2 recommended skip-reaudit for the closed cycle and I (orchestrator) accepted.
Perform these steps in order:
1. Lightweight verification: spot-check 2-3 surfaces Role 2 claims to have changed. Not a
   full re-audit — just a sanity pass for obvious drift.
2. Update the SPEC's priority status table for the closed cycle.
3. Write the cycle-closeout CHANGELOG entry with consolidated authorship. Survey
   AUDIT_LOG_FILE's cycle section for all `**Authors:**` lines, deduplicate (user first),
   emit one consolidated line.
4. Update HANDOFF.md and TASK.md.
5. (Optional, if next cycle is queued) Proceed to initial audit of the next cycle using
   Template A behavior. Otherwise stop.


Required checkpoint reads (quality guardrail):
- If you have not read your agent file this session, or if the session has been compacted
  since you last read it, read it now.
- Same for AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md
- ensure that you have completed all that is expected of you from your role and template.
- emit a git commit block per the "Convention for agent-produced git commit blocks" section in spec_plan_three_role_audit_loop-v3.md (mid-cycle form for non-closeout rounds; closeout form when you write the CHANGELOG/DEVLOG entry), placed before your handoff prompt(s).

```

### 4. Final Overseer — Wrap-Up Audit

Use after all intended cycles are nominally closed.

```text
Below N and <N> = <phase number to be filled in by user>

Read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md and follow it.

You are acting as Role 3 — Final Overseer / Wrap-Up Auditor.
Use Template D — Role 3 Final Wrap-Up Audit.

Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md
Current round: final wrap-up audit

All intended cycles are nominally closed. Perform a full wrap-up audit of the spec or plan,
the audit log, and the actual codebase. Do not edit code. Report any missed issues,
audit-trail inconsistencies, or adjacent issues that fit the spirit of the target plan.
When relevant, compare the target spec against `multi-agent/tracking/KNOWN_ISSUES.md` and `multi-agent/tracking/BRAINSTORM.md`. Write
your report as a new "Wrap-up audit" cycle section at the end of AUDIT_LOG_FILE. At cycle
closeout (if no actionable follow-up), write one CHANGELOG entry with consolidated
authorship.


Required checkpoint reads (quality guardrail):
- If you have not read your agent file this session, or if the session has been compacted
  since you last read it, read it now.
- Same for AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md
- ensure that you have completed all that is expected of you from your role and template.
- emit a git commit block per the "Convention for agent-produced git commit blocks" section in spec_plan_three_role_audit_loop-v3.md (mid-cycle form for non-closeout rounds; closeout form when you write the CHANGELOG/DEVLOG entry), placed before your handoff prompt(s).

```

### 5. Auditor — Post-Wrap-Up Triage

Use only if the Final Overseer found actionable work.

```text
Below N and <N> = <phase number to be filled in by user>

Read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template E — Role 1 Post-Wrap-Up Triage.

Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md
Current round: post-wrap-up triage

Role 3 has finished the wrap-up audit in AUDIT_LOG_FILE and found additional actionable
items. Review the wrap-up audit against the live code, decide which findings should become
active repair work now, and translate them into concrete implementation instructions in a
new "post-wrap-up remediation" cycle section of AUDIT_LOG_FILE. Do not rewrite the Final
Overseer report. Do NOT write to CHANGELOG.md in this round. Sync HANDOFF.md plus TASK.md
to point to the new next action.


Required checkpoint reads (quality guardrail):
- If you have not read your agent file this session, or if the session has been compacted
  since you last read it, read it now.
- Same for AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md
- ensure that you have completed all that is expected of you from your role and template.
- emit a git commit block per the "Convention for agent-produced git commit blocks" section in spec_plan_three_role_audit_loop-v3.md (mid-cycle form for non-closeout rounds; closeout form when you write the CHANGELOG/DEVLOG entry), placed before your handoff prompt(s).

```

### 6. Implementer — Post-Wrap-Up Implementation

Use after the Auditor has triaged the Final Overseer findings into concrete repair work.

```text
Below N and <N> = <phase number to be filled in by user>

Read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md and follow it.

You are acting as Role 2 — Implementer / Supplemental Auditor.
Use Template F — Role 2 Post-Wrap-Up Implementation.

Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md
Target cycle: post-wrap-up remediation
Current round: post-wrap-up implementation

Role 1 has translated actionable wrap-up findings into repair instructions in the
post-wrap-up remediation cycle of AUDIT_LOG_FILE. Implement only that triaged subset,
perform the justified validation, append your implementation report in the same cycle
section, and update HANDOFF.md and TASK.md. Do NOT write to CHANGELOG.md until the
post-wrap-up cycle closes.

At round end, declare either "re-audit needed" or "skip-reaudit recommended" per the
Skip-Reaudit Criteria.


Required checkpoint reads (quality guardrail):
- If you have not read your agent file this session, or if the session has been compacted
  since you last read it, read it now.
- Same for AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md
- ensure that you have completed all that is expected of you from your role and template.
- emit a git commit block per the "Convention for agent-produced git commit blocks" section in spec_plan_three_role_audit_loop-v3.md (mid-cycle form for non-closeout rounds; closeout form when you write the CHANGELOG/DEVLOG entry), placed before your handoff prompt(s).

```

### 7. Auditor — Final Closeout After Remediation

Use after the Implementer finishes the post-wrap-up implementation round.

```text
Below N and <N> = <phase number to be filled in by user>

Read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template G — Role 1 Final Closeout After Wrap-Up Remediation.

Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md
Target cycle: post-wrap-up remediation
Current round: post-wrap-up closeout

Role 2 has completed the post-wrap-up remediation round. Re-audit the live code, determine
whether the actionable findings from the Final Overseer audit are now resolved, and append
the final closeout judgment to the post-wrap-up remediation cycle in AUDIT_LOG_FILE. If
closed: update SPEC priority status table, write the cycle-closeout CHANGELOG entry with
consolidated authorship, and sync HANDOFF.md + TASK.md if the target file is now fully
closed.


Required checkpoint reads (quality guardrail):
- If you have not read your agent file this session, or if the session has been compacted
  since you last read it, read it now.
- Same for AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md
- ensure that you have completed all that is expected of you from your role and template.
- emit a git commit block per the "Convention for agent-produced git commit blocks" section in spec_plan_three_role_audit_loop-v3.md (mid-cycle form for non-closeout rounds; closeout form when you write the CHANGELOG/DEVLOG entry), placed before your handoff prompt(s).

```

---

## Concrete Example: Phase 15 Start — under v2

This example assumes:

- Claude Opus = Auditor (Role 1)
- GitHub Copilot = Implementer (Role 2)
- Gemini = Final Overseer (Role 3)
- Phase number `N = 15` (filled in below per the "Below N" convention)
- Cycle being worked: Priority 15.1 (substantive) — assume this is
  `PHASE15_STRATEGY.md` cycle index 1, cycle name `Phase15.1`

### Claude — open Cycle for Priority 15.1 (substantive)

```text
Below N and <N> = 12

Read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template A — Role 1 Initial Audit.

Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md
Target cycle: Phase15.1 (index 1) — Priority 15.1
Current round: initial audit

Audit this cycle deeply against the live codebase. Determine whether it is closed or open.
If it is open, append audit findings and exact implementation instructions into a new
cycle section of the audit log file, including file references, function references, and
concrete repair notes. Do NOT write to CHANGELOG.md. Read actual code rather than trusting
prior reports. Include scripts/ and tests/ when relevant.
```

### Copilot — implement the cycle

```text
Below N and <N> = 12

Read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md and follow it.

You are acting as Role 2 — Implementer / Supplemental Auditor.
Use Template B — Role 2 Implementation.

Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md
Target cycle: Phase15.1 (index 1) — Priority 15.1
Current round: implementation

Claude has finished the audit and written implementation instructions in the cycle section
of the audit log file. Read the audited findings and implement them as closely as
possible. While working, inspect nearby code for audit misses. If you need to diverge,
document the reason in your implementation report in the same cycle section. Append your
report to the audit log file. Do NOT write to CHANGELOG.md. Update HANDOFF.md and TASK.md.
Report the tests you ran.

At round end, declare either "re-audit needed" or "skip-reaudit recommended" per the
Skip-Reaudit Criteria in the workflow file.
```

### Claude — re-audit and close the cycle

```text
Below N and <N> = 12

Read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template C — Role 1 Re-Audit.

Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md
Target cycle: Phase15.1 (index 1) — Priority 15.1
Current round: re-audit

Copilot has finished an implementation round. Re-audit the live code and determine whether
this cycle is now closed or still open. Read the actual changed code rather than trusting
the implementation report. If still open, append new findings and exact repair
instructions to the cycle section. If closed: update the SPEC's priority status table and
write the cycle-closeout CHANGELOG entry with consolidated authorship (survey the audit
log file's cycle section for all `**Authors:**` lines, deduplicate, John first). Include
scripts/ and tests/ when relevant.
```

Loop Copilot and Claude until the cycle is closed, then move to the next cycle (referenced
by index + name in PHASE15_STRATEGY.md).

### Gemini — wrap-up audit after all cycles close

```text
Below N and <N> = 12

Read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md and follow it.

You are acting as Role 3 — Final Overseer / Wrap-Up Auditor.
Use Template D — Role 3 Final Wrap-Up Audit.

Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md
Current round: final wrap-up audit

All intended cycles are nominally closed. Perform a full wrap-up audit. Do not edit code.
Report any missed issues, audit-trail inconsistencies, or adjacent issues that fit the
spirit of the target plan. Compare against `multi-agent/tracking/KNOWN_ISSUES.md` and
`multi-agent/tracking/BRAINSTORM.md`. Write your report as a new "Wrap-up audit" cycle section at
the end of the audit log file. At cycle closeout (if no actionable follow-up), write one
CHANGELOG entry with consolidated authorship.
```

---

## Concrete Example: Batched Cycle (new in v2)

This example shows running a batched cycle — multiple thin priorities bundled into one
audit-implement-reaudit loop.

Assume Phase 14 Supplemental Phase 1a (the mechanical structural-parser-changes
sub-cycle) covering: `14-S10, 14-S4, 14-S5, 14-S29, 14-S30`. Per
`PHASE14_SUPPLEMENTAL-STRATEGY.md`, this is cycle **index 1**, cycle name **14S.1a**.

Note: Phase 14 Supplemental uses the supplemental naming `14_SUPPLEMENTAL-*.md` rather
than the plain `14_*.md` pattern, so the file paths below are written concretely rather
than using the generic `PHASE<N>_*.md` template.

### Claude — open the batched cycle

```text
Read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template A — Role 1 Initial Audit.

Spec file: multi-agent/plans/PHASE14_SUPPLEMENTAL-SPEC.md
Audit log file: multi-agent/plans/PHASE14_SUPPLEMENTAL-AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE14_SUPPLEMENTAL-STRATEGY.md
Target cycle: 14S.1a (index 1) — mechanical structural parser changes (14-S10, 14-S4, 14-S5, 14-S29, 14-S30)
Current round: initial audit

Audit this batched cycle deeply against the live codebase. Each priority in the batch has
its own scope block in the SPEC. Produce one consolidated set of findings and exact
implementation instructions covering all 5 priorities, appended to a new cycle section of
the audit log file. Do NOT write to CHANGELOG.md in this round. Read actual code.
```

### Copilot — implement the batched cycle

Role 2's implementation round covers all 10 priorities. One implementation report covers
the whole batch.

### Closeout

At closeout, one CHANGELOG entry covers all 10 priorities under a single cycle-name like:

```
## [v0.X.YY] — YYYY-MM-DD HH:MM EST

### Phase 14 Supplemental Phase 1 — structural parser changes (batched cycle)

Closed priorities: 14-S10, 14-S4, 14-S5, 14-S29, 14-S30, 14-S27, 14-S28, 14-S22, 14-S23, 14-S26.
[summary of work...]

**Authors:** John M. Urban, Claude Code 2.1.X (claude-opus-4-7), GitHub Copilot (gpt-5-codex)

**Roadmap:** Phase 14 Supplemental Phase 1
```

Note that the SPEC's priority status table gets 10 rows updated to `CLOSED (v0.X.YY)` in
this single closeout.

---

## Concrete Example: After Final Overseer Finishes

Use this when the Final Overseer finds additional actionable work.

### Role-language version

```text
Role 3 finished the wrap-up audit in AUDIT_LOG_FILE and found actionable follow-up.
Return to Role 1 for triage (Template E) — triage opens a new post-wrap-up remediation
cycle in AUDIT_LOG_FILE.
Then send Role 2 for implementation (Template F).
Then send Role 1 for final closeout (Template G).
```

### Claude — triage Gemini findings in Phase 15

```text
Below N and <N> = 15

Read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template E — Role 1 Post-Wrap-Up Triage.

Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md
Current round: post-wrap-up triage

Gemini has completed the Final Overseer audit at the end of the audit log file and
identified additional findings in the spirit of Phase 15. Review that wrap-up audit against
the live codebase and decide which findings should become active repair work now versus
future-phase backlog. Translate the actionable items into concrete implementation
instructions in a new "post-wrap-up remediation" cycle section of the audit log file. Do
not rewrite or remove the Final Overseer report. Do NOT write to CHANGELOG.md in this
round. Include scripts/ and tests/ when relevant.
```

### Copilot — implement the triaged Final Overseer follow-up

```text
Below N and <N> = 15

Read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md and follow it.

You are acting as Role 2 — Implementer / Supplemental Auditor.
Use Template F — Role 2 Post-Wrap-Up Implementation.

Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md
Target cycle: post-wrap-up remediation
Current round: post-wrap-up implementation

Claude has triaged the actionable findings from the Final Overseer audit and translated
them into repair instructions in the post-wrap-up remediation cycle of the audit log file.
Implement only that triaged subset, perform the justified validation, append your report
in the same cycle section, and update HANDOFF.md + TASK.md. Do NOT write to CHANGELOG.md
until the post-wrap-up cycle closes.

At round end, declare either "re-audit needed" or "skip-reaudit recommended" per the
Skip-Reaudit Criteria.
```

### Claude — final closeout after post-wrap-up remediation

```text
Below N and <N> = 15

Read multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template G — Role 1 Final Closeout After Wrap-Up Remediation.

Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md
Target cycle: post-wrap-up remediation
Current round: post-wrap-up closeout

Copilot has completed the post-wrap-up remediation round. Re-audit the live code,
determine whether the actionable findings from the Final Overseer audit are now resolved,
and append the final closeout judgment to the post-wrap-up remediation cycle section. If
closed: update SPEC priority status table, write the cycle-closeout CHANGELOG entry with
consolidated authorship, and sync HANDOFF.md + TASK.md if the target file is now fully
closed.
```

---

## Decision Guide For The Orchestrator — v2

If you are unsure what to send next, use this rule:

1. **New cycle starts**: send Auditor Template A (Role 1 initial audit).
2. **Auditor wrote instructions**: send Implementer Template B (Role 2 implementation).
3. **Implementer finished with "re-audit needed" declaration**: send Auditor Template C
   (Role 1 re-audit).
4. **Implementer finished with "skip-reaudit recommended"**:
   - If you ACCEPT the skip: send Auditor Template H (Role 1 cycle closeout after
     skip-reaudit).
   - If you REJECT the skip (force full re-audit): send Auditor Template C instead.
5. **Cycle closed, next cycle queued**: send Auditor Template A for the next cycle (or
   Template H already covers this transition if skip-reaudit was accepted).
6. **All cycles closed**: send Final Overseer Template D.
7. **Final Overseer found actionable work**: send Auditor Template E (triage into
   post-wrap-up remediation cycle).
8. **Auditor triaged findings**: send Implementer Template F.
9. **Post-wrap-up implementation finished**: send Auditor Template G (final closeout).

### Deciding whether to accept a Role 2 skip-reaudit

Default to **NO skip** (force full re-audit) when any of these apply:
- The cycle involves design judgment or help-string quality.
- The cycle is a substantive single-priority cycle (not a thin-priority batched cycle).
- The cycle touches runtime wiring, not just parser surface.
- Tests did not run green on the first try.
- This is the first time you're using v2 — build trust in the skip criteria before using
  them liberally.

Default to **YES, accept skip** only when ALL skip criteria are met AND the cycle is a
batched mechanical cycle (renames, help-string appends with no judgment, flag additions
with clear spec, etc.).

Even when accepting a skip, Role 1 still does a lightweight spot-check at closeout
(Template H). That's the backstop.

---

## Notes

1. You do not need to paste the entire workflow file into the prompt. It is enough to tell
   the agent to read `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md` and
   follow it.
2. You should still include the pertinent template behavior in the prompt so the immediate
   task is unambiguous.
3. Use actual agent names if helpful, but the prompts also work if you refer only to role
   names.
4. The Final Overseer audit should normally be a single pass. If it finds actionable work,
   the usual next sequence is Auditor → Implementer → Auditor, not a second Final Overseer
   pass.
5. **v2 opt-in**: the v1 system is still available at
   `multi-agent/workflows/orchestrator.spec_plan_three_role_audit_loop-v1.md`. Pick one
   system per target SPEC; do not mix.
6. **Create AUDIT_LOG_FILE at cycle start**: when opening the first cycle under v2, the
   orchestrator (or Role 1 at initial audit) creates the sibling AUDIT_LOG_FILE with a
   skeleton header. See the v2 workflow file's "Notes For The User" for the template.

---

## Agent selection and performance tuning

This section is the human orchestrator's reference for choosing agents and configuring
their performance settings. It is self-contained — no need to cross-reference the
workflow file for this information.

### Performance toggle convention (parallel across vendors)

All three frontier vendors expose a comparable runtime toggle that trades tokens and
latency for deeper reasoning. The model name selects the model; the toggle selects how
hard it thinks. Notation is parallel:

| Vendor | Harnesses | Model examples | Toggle name | Toggle values |
|---|---|---|---|---|
| Anthropic | Claude Code, Github Copilot — Claude | Sonnet, Opus, Haiku | **Effort** | Low \| Medium \| **High** (default) \| Extra High \| Max |
| OpenAI | Codex, Github Copilot — GPT | GPT-5.5, GPT-5.4 | **Reasoning** | Low \| Medium \| **High** \| Extra High |
| Google | Gemini terminal | Gemini 3.1 Pro Preview, 2.5 Pro | **Thinking** | low / medium / high (CLI flag value, varies by Gemini version) |

Always write agent specs in the form `<harness> — <model> (<Toggle>: <value>)`. Examples:
- `Claude Code — Sonnet (Effort: Extra High)`
- `Claude Code — Opus (Effort: High)`
- `Codex — GPT-5.5 (Reasoning: High)`
- `Github Copilot — GPT-5.5 (Reasoning: Medium)`
- `Github Copilot — Claude Sonnet (Effort: Extra High)`
- `Gemini 3.1 Pro Preview (Thinking: high)`

Higher toggle values pay off most on judgment-heavy tasks (auditing, re-auditing, phase
closeout); implementation from a clear spec is more mechanical and rarely needs above the
default.

### Agent tier roster

**Tier 1 — deep reasoning** (use for audit/re-audit/closeout on semantically tricky or
phase-ending cycles):

| Agent | Default toggle for this tier |
|---|---|
| Claude Code — Opus | Effort: High; Extra High for phase-ending / Final Overseer |
| Github Copilot — Claude Opus | Effort: High; Extra High for phase-ending / Final Overseer |
| Github Copilot — GPT-5.5 | Reasoning: High or Extra High |
| Codex — GPT-5.5 | Reasoning: High or Extra High |
| Gemini 3.1 Pro Preview | Thinking: high (recommended for audit/overseer work) |
| Gemini 2.5 Pro | Thinking: high (recommended for audit/overseer work) |

**Tier 2 — balanced / default** (standard audit cycles; R2 implementation):

| Agent | Default toggle for this tier |
|---|---|
| Claude Code — Sonnet | Effort: **Extra High** for audit/re-audit; **High** for implementation |
| Github Copilot — Claude Sonnet | Effort: **Extra High** for audit/re-audit; **High** for implementation |
| Github Copilot — GPT-5.5 | Reasoning: Medium |
| Codex — GPT-5.5 | Reasoning: Medium |

**Tier 3 — fast / cheap** (mechanical spot-checks, simple verification):

| Agent | Default toggle for this tier |
|---|---|
| Claude Code — Haiku | Effort: High default; Medium/Low sufficient for simple spot-checks |
| Github Copilot — Claude Haiku | Effort: High default |
| Github Copilot — GPT-5.5 | Reasoning: Low |
| Codex — GPT-5.5 | Reasoning: Low |
| Gemini 3 Flash Preview | — |
| Gemini 3.1 Flash Lite Preview | — |

### Baseline agent assignments

**Claude Code available (preferred):**

| Role | Primary | Alt |
|---|---|---|
| R1 — Auditor | Claude Code — Sonnet (Effort: Extra High) | Github Copilot — Claude Sonnet (Effort: Extra High); or Github Copilot — GPT-5.5 (Reasoning: High) |
| R2 — Implementer | Github Copilot — GPT-5.5 (Reasoning: Medium or High) | Codex — GPT-5.5 (Reasoning: High); or Github Copilot — Claude Sonnet (Effort: High) |
| R3 — Re-auditor/Closeout | Claude Code — Sonnet (Effort: Extra High; **different session from R2**) | Github Copilot — Claude Sonnet (Effort: Extra High); upgrade to Opus-tier for phase-ending cycles |
| Final Overseer | Gemini 3.1 Pro Preview (Thinking: high) | Codex — GPT-5.5 (Reasoning: High); or Gemini 2.5 Pro (Thinking: high) |

**Claude Code unavailable:**

| Role | Primary | Alt |
|---|---|---|
| R1 — Auditor | Github Copilot — GPT-5.5 (Reasoning: High) | Github Copilot — Claude Opus (Effort: High) / Claude Sonnet (Effort: Extra High) |
| R2 — Implementer | Codex — GPT-5.5 (Reasoning: Medium or High) | Github Copilot — GPT-5.5 (Reasoning: Medium) |
| R3 — Re-auditor/Closeout | Github Copilot — GPT-5.5 (Reasoning: High; **different session**) | Gemini 3.1 Pro Preview (Thinking: high) |
| Final Overseer | Gemini 3.1 Pro Preview (Thinking: high) | Codex — GPT-5.5 (Reasoning: High) |

### When to upgrade or downgrade tier

**Upgrade R1 (and usually R3) to Tier 1** when the cycle involves:
- Semantic nuance (mixed-meaning labels, subtle invariants, nontrivial math or biology).
- Full-surface cross-cutting passes (help-string rewrites, cross-file doc sweeps).
- Phase-ending closeouts (file moves, reference propagation, archival).
- Decisions where auditor drift would cost significant downstream rework.

**Keep R1 at Tier 2** when the cycle is:
- Mechanical renames, deprecations, default flips, single-flag additions.
- Contained parser-group changes with no cross-pipeline impact.
- Runtime wiring of a flag whose semantics are already fully spec'd.

**R2 upgrade path:** Medium → High for multi-file substantive cycles → High/Extra High for
runtime correctness, test-harness wiring, or SPEC-mandated paired priorities.

### Per-role toggle recommendations

| Role | Task | Claude Effort | GPT Reasoning | Gemini Thinking |
|---|---|---|---|---|
| R1 — Auditor (Sonnet/GPT-5.5) | Single-surface audit | **Extra High** | **High** | high |
| R1 — Auditor (Sonnet/GPT-5.5) | Paired/complex audit (multiple priorities) | **Max** | **Extra High** | high |
| R2 — Implementer | Mechanical implementation of clear spec | **High** (default) | **Medium** | medium |
| R2 — Implementer | Complex cross-module repair / dependency fix | **Extra High** | **High** | high |
| R3 — Re-auditor / Closeout | Verify implementation matches findings | **Extra High** | **High** | high |
| R1/R3 — Auditor (Opus) | Any audit | **High** — Opus reasons deeply at High | n/a | n/a |
| Final Overseer | Cross-cycle phase closeout | **Extra High** (Opus) | **High** (Codex) | high (Gemini) |

**Key principle:** Higher toggle values pay off most on judgment and verification
(auditing, re-auditing, closeout). Implementation from a clear spec is more mechanical
— the default value is usually sufficient. For Opus, High is already strong; use Extra
High only for consequential phase-ending judgments. Max effort is reserved for the most
complex multi-priority Sonnet audits where missing a cross-module interaction would cost
downstream rework.

### How toggle values appear in STRATEGY.md and advisory blocks

STRATEGY.md bundles the toggle value with the agent name in each role cell:
```
Primary: Claude Code — Sonnet (Effort: Extra High)
Alt: Codex — GPT-5.5 (Reasoning: High)
```
Advisory blocks copy verbatim from STRATEGY.md, so the full assignment (agent + toggle)
flows through to every handoff prompt automatically. When writing advisory blocks by hand,
always include the level alongside the agent name.
