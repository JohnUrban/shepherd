
# ORCHESTRATOR GUIDE FOR `spec_plan_three_role_audit_loop.md`

This file is for the human orchestrator.

Use it alongside:

- `multi-agent/workflows/spec_plan_three_role_audit_loop.md`

That file defines the workflow rules for the agents.
This file shows you what to send, when to send it, and how to move from one role to the next.

---

## Core Rule

Do not paste only a tiny fragment of the workflow unless you already know the agent has read the
whole workflow file.

Best practice:

1. Tell the agent to read `multi-agent/workflows/spec_plan_three_role_audit_loop.md`
2. Tell it which role it is playing
3. Tell it which template behavior to follow
4. Tell it the target file
5. Tell it the target section when relevant
6. Tell it the current round

In practice, your prompt should be:

- whole workflow file by reference
- plus the pertinent template behavior
- plus the current target file and section

---

## Quick Launcher Pattern

Use this shell for nearly every round:

```text
Read multi-agent/workflows/spec_plan_three_role_audit_loop.md and follow it.

You are acting as {ROLE_NAME}.
Use {TEMPLATE_NAME}.

Target file: {TARGET_FILE}
Target section: {TARGET_SECTION}
Current round: {ROUND_TYPE}

{ROUND_SPECIFIC_INSTRUCTIONS}
```

If a round is full-file rather than section-specific, omit `Target section`.

---

## Start-To-Finish Pipeline

This is the standard 3-role sequence.

1. Auditor performs initial audit
2. Implementer executes the written instructions
3. Auditor re-audits and either closes the section or writes follow-up findings
4. Repeat steps 2-3 until the section closes
5. Final Overseer audits the fully worked spec/plan at the end
6. If Final Overseer finds actionable work, bounce back through:
	- Auditor triage
	- Implementer execution
	- Auditor final closeout

The Final Overseer normally does one pass only.

---

## Standard Copy-Paste Prompts

These are the prompts I recommend using in practice.

### 1. Auditor — Initial Audit

Use when opening a new section.

```text
Read multi-agent/workflows/spec_plan_three_role_audit_loop.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template A — Role 1 Initial Audit.

Target file: path/to/TARGET_FILE
Target section: path/to/SECTION_HEADING
Current round: initial audit

Audit this section deeply against the live codebase. Determine whether it is closed or open.
If it is open, append audit findings and exact implementation instructions into the same section,
including file references, function references, and concrete repair notes. Update CHANGELOG.md for
the audit. Read actual code rather than trusting prior reports. Include scripts/ and tests/ when
relevant.


After finishing up, before writing your CHANGELOG entry:
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop.md
- ensure that you have completed all that is expected of you from your role and template.


```

If you want to speed through a narrow round, you can add:
```text
If this is very narrow, then write your audit and plan as Role 1, template A. Then implement as Role 2, Template B. Then re-audit as Role 1, Template C, all in on session. What do you think?
```


### 2. Implementer — Implementation Round

Use after the Auditor has written findings and instructions.

```text
Read multi-agent/workflows/spec_plan_three_role_audit_loop.md and follow it.

You are acting as Role 2 — Implementer / Supplemental Auditor.
Use Template B — Role 2 Implementation.

Target file: path/to/TARGET_FILE
Target section: path/to/SECTION_HEADING
Current round: implementation

Read the audited findings and implementation instructions in the target section, then implement
them as closely as possible. While working, inspect nearby code for audit misses in the same
surface area. If you need to diverge, document the reason in your implementation report under the
same section. Update CHANGELOG.md, HANDOFF.md, and TASK.md. Report the tests you ran.


After finishing up, before writing your CHANGELOG entry:
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop.md
- ensure that you have completed all that is expected of you from your role and template.


```

### 3. Auditor — Re-Audit Round

Use after the Implementer finishes an implementation round.

```text
Read multi-agent/workflows/spec_plan_three_role_audit_loop.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template C — Role 1 Re-Audit.

Target file: path/to/TARGET_FILE
Target section: path/to/SECTION_HEADING
Current round: re-audit

Role 2 has finished an implementation round. Re-audit the live code and determine whether this
section is now closed or still open. Read the actual changed code rather than trusting the
implementation report. If it is still open, append new findings and exact repair instructions into
the same section and update CHANGELOG.md. Include scripts/ and tests/ when relevant.


After finishing up, before writing your CHANGELOG entry:
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop.md
- ensure that you have completed all that is expected of you from your role and template.

```

### 4. Final Overseer — Wrap-Up Audit

Use after all intended sections are nominally closed.

```text
Read multi-agent/workflows/spec_plan_three_role_audit_loop.md and follow it.

You are acting as Role 3 — Final Overseer / Wrap-Up Auditor.
Use Template D — Role 3 Final Wrap-Up Audit.

Target file: path/to/TARGET_FILE
Current round: final wrap-up audit

All intended sections are nominally closed. Perform a full wrap-up audit of the spec or plan,
the audit trail, and the actual codebase. Do not edit code. Report any missed issues, audit-trail
inconsistencies, or adjacent issues that fit the spirit of the target plan. Write your report in
the designated wrap-up section at the bottom of the target file, and update CHANGELOG.md for the
audit. Use the heading `## Wrap-up audit with Final Overseer` unless the target file already has
an explicit wrap-up heading.


After finishing up, before writing your CHANGELOG entry:
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop.md
- ensure that you have completed all that is expected of you from your role and template.

```

### 5. Auditor — Post-Wrap-Up Triage

Use only if the Final Overseer found actionable work.

```text
Read multi-agent/workflows/spec_plan_three_role_audit_loop.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template E — Role 1 Post-Wrap-Up Triage.

Target file: path/to/TARGET_FILE
Current round: post-wrap-up triage

Role 3 has finished the wrap-up audit and found additional actionable items. Review the wrap-up
audit against the live code, decide which findings should become active repair work now, and
translate them into concrete implementation instructions in the authoritative target section(s)
of the spec or plan. Do not rewrite the Final Overseer report. Update CHANGELOG.md and sync
HANDOFF.md plus TASK.md to point to the new next action.


After finishing up, before writing your CHANGELOG entry:
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop.md
- ensure that you have completed all that is expected of you from your role and template.

```

### 6. Implementer — Post-Wrap-Up Implementation

Use after the Auditor has triaged the Final Overseer findings into concrete repair work.

```text
Read multi-agent/workflows/spec_plan_three_role_audit_loop.md and follow it.

You are acting as Role 2 — Implementer / Supplemental Auditor.
Use Template F — Role 2 Post-Wrap-Up Implementation.

Target file: path/to/TARGET_FILE
Target section: path/to/SECTION_HEADING
Current round: post-wrap-up implementation

Role 1 has translated actionable wrap-up findings into repair instructions. Implement only that
triaged subset, perform the justified validation, append your implementation report in the
authoritative target section(s), and update CHANGELOG.md, HANDOFF.md, and TASK.md.


After finishing up, before writing your CHANGELOG entry:
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop.md
- ensure that you have completed all that is expected of you from your role and template.

```

### 7. Auditor — Final Closeout After Remediation

Use after the Implementer finishes the post-wrap-up implementation round.

```text
Read multi-agent/workflows/spec_plan_three_role_audit_loop.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template G — Role 1 Final Closeout After Wrap-Up Remediation.

Target file: path/to/TARGET_FILE
Target section: path/to/SECTION_HEADING
Current round: post-wrap-up closeout

Role 2 has completed the post-wrap-up remediation round. Re-audit the live code, determine
whether the actionable findings from the Final Overseer audit are now resolved, and append the
final closeout judgment in the relevant section(s). Update CHANGELOG.md and sync the project
context files if the target file is now fully closed.


After finishing up, before writing your CHANGELOG entry:
- Read one agent file (e.g. choose from CLAUDE.md, AGENTS.md, GEMINI.md, or .github/copilot-instructions.md).
- Then read AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop.md
- ensure that you have completed all that is expected of you from your role and template.

```

---

## Concrete Example: Phase 12 Start

This example assumes:

- Claude = Auditor
- GitHub Copilot = Implementer
- Gemini = Final Overseer

### Claude — open Priority 12.1

```text
Read multi-agent/workflows/spec_plan_three_role_audit_loop.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template A — Role 1 Initial Audit.

Target file: multi-agent/plans/PHASE12_SPEC.md
Target section: ## Priority 12.1 — <replace with actual title>
Current round: initial audit

Audit this section deeply against the live codebase. Determine whether it is closed or open.
If it is open, append audit findings and exact implementation instructions into the same section,
including file references, function references, and concrete repair notes. Update CHANGELOG.md for
the audit. Read actual code rather than trusting prior reports. Include scripts/ and tests/ when
relevant.
```

### Copilot — implement Priority 12.1

```text
Read multi-agent/workflows/spec_plan_three_role_audit_loop.md and follow it.

You are acting as Role 2 — Implementer / Supplemental Auditor.
Use Template B — Role 2 Implementation.

Target file: multi-agent/plans/PHASE12_SPEC.md
Target section: ## Priority 12.1 — <replace with actual title>
Current round: implementation

Claude has finished the audit and written implementation instructions in the target section.
Read the audited findings and implement them as closely as possible. While working, inspect nearby
code for audit misses in the same surface area. If you need to diverge, document the reason in
your implementation report under the same section. Update CHANGELOG.md, HANDOFF.md, and TASK.md.
Report the tests you ran.
```

### Claude — re-audit Priority 12.1

```text
Read multi-agent/workflows/spec_plan_three_role_audit_loop.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template C — Role 1 Re-Audit.

Target file: multi-agent/plans/PHASE12_SPEC.md
Target section: ## Priority 12.1 — <replace with actual title>
Current round: re-audit

Copilot has finished an implementation round. Re-audit the live code and determine whether this
section is now closed or still open. Read the actual changed code rather than trusting the
implementation report. If it is still open, append new findings and exact repair instructions into
the same section and update CHANGELOG.md. Include scripts/ and tests/ when relevant.
```

Loop Copilot and Claude until the section is closed, then move to the next section.

### Gemini — wrap-up audit after all sections close

```text
Read multi-agent/workflows/spec_plan_three_role_audit_loop.md and follow it.

You are acting as Role 3 — Final Overseer / Wrap-Up Auditor.
Use Template D — Role 3 Final Wrap-Up Audit.

Target file: multi-agent/plans/PHASE12_SPEC.md
Current round: final wrap-up audit

All intended sections are nominally closed. Perform a full wrap-up audit of the spec or plan,
the audit trail, and the actual codebase. Do not edit code. Report any missed issues, audit-trail
inconsistencies, or adjacent issues that fit the spirit of the target plan. Write your report in
the designated wrap-up section at the bottom of the target file, and update CHANGELOG.md for the
audit. Use the heading `## Wrap-up audit with Final Overseer` unless the target file already has
an explicit wrap-up heading.
```

---

## Concrete Example: After Final Overseer Finishes

Use this when the Final Overseer finds additional actionable work.

### Role-language version

```text
Role 3 finished the wrap-up audit and found actionable follow-up.
Return to Role 1 for triage and instructions.
Then send Role 2 for implementation.
Then send Role 1 for final closeout.
```

### Claude — triage Gemini findings in Phase 11

```text
Read multi-agent/workflows/spec_plan_three_role_audit_loop.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template E — Role 1 Post-Wrap-Up Triage.

Target file: multi-agent/plans/PHASE11_SPEC.md
Target section: ## Wrap-up audit with Gemini
Current round: post-wrap-up triage

Gemini has completed the Final Overseer audit at the bottom of PHASE11_SPEC.md and identified
additional findings in the spirit of Phase 11. Review that wrap-up audit against the live
codebase and decide which findings should become active repair work now versus future-phase
backlog. Translate the actionable items into concrete implementation instructions in the
authoritative section(s) of PHASE11_SPEC.md. Do not rewrite or remove the Final Overseer report.
Update CHANGELOG.md for your audit/triage. Include scripts/ and tests/ when relevant.
```

### Copilot — implement the triaged Final Overseer follow-up

```text
Read multi-agent/workflows/spec_plan_three_role_audit_loop.md and follow it.

You are acting as Role 2 — Implementer / Supplemental Auditor.
Use Template F — Role 2 Post-Wrap-Up Implementation.

Target file: multi-agent/plans/PHASE11_SPEC.md
Target section: <insert the section Claude updated with actionable repair instructions>
Current round: post-wrap-up implementation

Claude has triaged the actionable findings from the Final Overseer audit and translated them into
repair instructions in PHASE11_SPEC.md. Implement only that triaged subset, perform the justified
validation, append your implementation report in the authoritative target section(s), and update
CHANGELOG.md, HANDOFF.md, and TASK.md.
```

### Claude — final closeout after post-wrap-up remediation

```text
Read multi-agent/workflows/spec_plan_three_role_audit_loop.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template G — Role 1 Final Closeout After Wrap-Up Remediation.

Target file: multi-agent/plans/PHASE11_SPEC.md
Target section: <insert the section Claude used for the actionable repair instructions>
Current round: post-wrap-up closeout

Copilot has completed the post-wrap-up remediation round. Re-audit the live code, determine
whether the actionable findings from the Final Overseer audit are now resolved, and append the
final closeout judgment in the relevant section(s). Update CHANGELOG.md and sync the project
context files if the target file is now fully closed.
```

---

## Decision Guide For The Orchestrator

If you are unsure what to send next, use this rule:

1. New section starts: send Auditor Template A
2. Auditor wrote instructions: send Implementer Template B
3. Implementer finished: send Auditor Template C
4. All sections closed: send Final Overseer Template D
5. Final Overseer found actionable work: send Auditor Template E
6. Auditor translated those findings into repair instructions: send Implementer Template F
7. Implementer finished the repair: send Auditor Template G

---

## Notes

1. You do not need to paste the entire workflow file into the prompt. It is enough to tell the
	agent to read `multi-agent/workflows/spec_plan_three_role_audit_loop.md` and follow it.
2. You should still include the pertinent template behavior in the prompt so the immediate task is
	unambiguous.
3. Use actual agent names if helpful, but the prompts also work if you refer only to role names.
4. The Final Overseer audit should normally be a single pass. If it finds actionable work, the
	usual next sequence is Auditor -> Implementer -> Auditor, not a second Final Overseer pass.

