# SPEC / PLAN THREE-ROLE AUDIT LOOP

Reusable coordination prompt for 2-3 agents working through a spec or plan file one target
section at a time.

This workflow is agent-agnostic. Any capable agent may fill any role.

---

## Purpose

Use this workflow when the user wants to close out an active spec or plan by repeating a
disciplined loop:

1. audit one section deeply
2. write exact findings and instructions into that same section
3. implement only those audited open items
4. re-audit the implementation against live code
5. repeat until the section is closed
6. perform a final wrap-up audit across the full spec or plan
7. if the wrap-up audit finds actionable work, route it back through Auditor -> Implementer ->
   Auditor for final remediation and closeout

The workflow is designed for files such as active phase specs, implementation plans, audit
plans, or other structured markdown files under `multi-agent/plans/` or another user-specified
path.

---

## When To Use

Use this workflow when all of the following are true:

- The target work is organized into named sections such as phases, priorities, tasks, or workstreams.
- The user wants one section audited at a time rather than broad simultaneous implementation.
- The user wants findings, repair instructions, implementation notes, and closeout status written
  back into the same spec or plan file.
- The user wants every audit and implementation round recorded in `CHANGELOG.md`.

Do not use this workflow for:

- small one-off bug fixes that do not need a spec-driven loop
- purely read-only audits with no implementing role
- large exploratory design work that is not yet organized into auditable sections

---

## Required Inputs

Before any role starts, the user should provide or confirm:

1. `TARGET_FILE`: the spec or plan file to use as the authoritative working contract
2. `TARGET_SECTION`: the specific section to audit or implement in the current round
3. `UPDATE_FILES`: files that must be synced after each round
   Default set for onionskin:
   - `CHANGELOG.md`
   - `multi-agent/project_context/HANDOFF.md`
   - `multi-agent/project_context/TASK.md`
4. whether the current round is:
   - initial audit
   - implementation
   - re-audit
   - final wrap-up audit
   - post-wrap-up triage
   - post-wrap-up implementation
   - post-wrap-up closeout
5. whether the workflow is operating in:
   - 2-role mode
   - 3-role mode

Suggested assignment line:

```text
You are acting as {ROLE_NAME} from multi-agent/workflows/spec_plan_three_role_audit_loop.md.
Target file: {TARGET_FILE}
Target section: {TARGET_SECTION}
Current round: {ROUND_TYPE}
```

---

## Target File Structure And Editing Conventions

The target spec or plan file is the authoritative audit trail. Edit it conservatively.

1. Keep the existing section structure intact unless the user explicitly asks for a redesign.
2. Write audit and implementation history as appended dated subsections; do not erase prior audit
   history.
3. Role 1 writes findings and instructions in the relevant target section that owns the work.
4. Role 2 writes implementation reports in that same target section unless the user explicitly
   chooses a different reporting surface.
5. Role 1 writes re-audit findings and closeout judgments in that same target section.
6. Role 3 writes the wrap-up audit at the bottom of the file under:

   `## Wrap-up audit with Final Overseer`

   If the file already has a designated wrap-up heading, reuse it instead of inventing another one.
7. If Role 3 finds actionable issues, leave the wrap-up audit intact and route the actionable work
   back into the authoritative execution location:
   - section-specific findings should be translated by Role 1 into the relevant section(s)
   - cross-cutting findings may be triaged in a short auditor response below the wrap-up audit,
     but implementation instructions should still point back to the section(s) that own the work
8. Distinguish clearly between:
   - verified complete work
   - actionable follow-up findings
   - future-looking ideas or next-phase suggestions

---

## Rules for updating other files

Before writing in the CHANGELOG:

1. Read your agent file (CLAUDE.md, AGENTS.md, GEMINI.md, .github/copilot-instructions.md)
2. Read AGENT_CONVENTIONS.md

---


## Roles

### Role 1 — Auditor / Instruction Writer

Role 1 performs deep audits and writes the repair contract.

Primary responsibilities:

1. Read the target section and the surrounding context needed to understand its intent.
2. Audit the live code, tests, scripts, and documentation relevant to that section.
3. Determine whether the section is closed, partially complete, or still open.
4. If open, append audit findings and exact implementation instructions into the same section of
   `TARGET_FILE`.
5. Update `CHANGELOG.md` for each audit round.
6. Re-audit Role 2's implementation by reading the actual changed code rather than trusting the
   implementation report.
7. Close the section only after verifying that the findings are actually resolved.
8. If Role 3 identifies actionable wrap-up findings, review that wrap-up audit, decide which
   findings become active repair work, and translate them into concrete implementation
   instructions in the authoritative section(s) of `TARGET_FILE`.
9. After Role 2 completes any post-wrap-up remediation, perform the final closeout audit.

Role 1 must not:

- trust the implementation report without inspecting the code
- skip `scripts/` or `tests/` when they are plausibly affected
- silently narrow or broaden scope — scope changes belong in an explicit audit finding for the
  user to approve; never relegate in-scope work to KNOWN_ISSUES without asking the user first

Expected output from Role 1:

- an audit subsection in `TARGET_FILE`
- exact repair instructions with file/function references when possible
- a matching `CHANGELOG.md` entry
- an explicit close / still-open judgment after each re-audit
- when needed, an auditor triage of Final Overseer findings and a final post-remediation closeout
   judgment


Expected at the end of each audit round when more implementation is needed:
- At the end of your chat message to the user, provide the following prompt to give to the Role 2 Implementer Agent:
```text
Use this prompt for the Implementer Agent:

Read multi-agent/workflows/spec_plan_three_role_audit_loop.md and follow it.

You are acting as Role 2 — Implementer / Supplemental Auditor.
Use Template B — Role 2 Implementation.

Target file: path/to/TARGET_FILE
Target section: path/to/SECTION_HEADING
Current round: implementation

Read the audited findings and implementation instructions in the target section, then implement them as closely as possible. While working, inspect nearby code for audit misses in the same surface area. If you need to diverge, document the reason in your implementation report under the same section. Update CHANGELOG.md, HANDOFF.md, and TASK.md. Report the tests you ran.
```
- where path/to/TARGET_FILE = `TARGET_FILE` you were working on
- where path/to/SECTION_HEADING = `TARGET_SECTION` you were working in


Expected after closing up a priority, and there are still priorities to address in this Phase Spec:
- At the end of your chat message to the user, provide the following prompt to give to back to you, the Role 1 Auditor, for prompting the first pass audit on the next Priority:
```text
Feed this prompt back to me, the Role 1 Auditor agent, to move on to the next Priority in this Phase:

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
- where path/to/TARGET_FILE = `TARGET_FILE` you were working on
- where path/to/SECTION_HEADING = the `TARGET_SECTION` after the one you just closed.



Expected after all intended sections are nominally closed when it is time to pass it off to the Role 3 Final Overseer for the Wrap-Up Audit:
- At the end of your chat message to the user, provide the following prompt to give to the Role 3 Final Overseer Agent:
```text
Use this prompt for the Final Overseer Agent:

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
- where path/to/TARGET_FILE = `TARGET_FILE` you were working on
- where path/to/SECTION_HEADING = `TARGET_SECTION` you were working in


Expected after the triage of the Final Overseer findings into concrete repair work ready for the Role 2 Implementer Agent.
- At the end of your chat message to the user, provide the following prompt to give to the Role 2 Implementer Agent:
```text
Use this prompt for the Implementer Agent:

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
- where path/to/TARGET_FILE = `TARGET_FILE` you were working on
- where path/to/SECTION_HEADING = `TARGET_SECTION` you were working in

### Role 2 — Implementer / Supplemental Auditor

Role 2 implements the audited instructions and performs in-situ auditing while working.

Primary responsibilities:

1. Read the current audited section in `TARGET_FILE` and the latest relevant `CHANGELOG.md` entry.
2. Implement only the audited open items unless code reality forces a documented deviation.
3. While editing, inspect surrounding code carefully enough to catch audit misses in the same
   neighborhood.
4. If a miss is found:
   - fix it if it is clearly in scope and low-risk, then document it
   - otherwise flag it clearly in the implementation report
5. Run the targeted validation justified by the changed surface.
6. Append an implementation report into the same target section of `TARGET_FILE`.
7. Update `CHANGELOG.md`, `HANDOFF.md`, and `TASK.md` to reflect the new state.
8. If the round comes from Role 3 follow-up findings, implement only the actionable subset that
   Role 1 has triaged into concrete instructions.

Role 2 must not:

- start coding before explicit user approval for the implementation round
- silently diverge from the audit instructions
- assume the audit was exhaustive
- close the section without a verifying audit from Role 1 or Role 3

Expected output from Role 2:

- the code changes themselves
- an implementation report in `TARGET_FILE`
- a matching `CHANGELOG.md` entry
- synced `HANDOFF.md` and `TASK.md`
- clear note of tests run and any deviation from the audit instructions

Expected at the end of each implementatin round when ready for the re-audit:
- At the end of your chat message to the user, provide the following prompt to give to the Role 1 Auditor Agent:
```text
Use this prompt for the Auditor Agent:

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
- where path/to/TARGET_FILE = `TARGET_FILE` you were working on
- where path/to/SECTION_HEADING = `TARGET_SECTION` you were working in


Expected after finishing the post-wrap-up implementation round (after implementation of the triage of the Final Overseer Wrap-up audit).
- At the end of your chat message to the user, provide the following prompt to give to the Role 1 Auditor Agent:
```text
Use this prompt for the Auditor Agent:

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

### Role 3 — Final Overseer / Wrap-Up Auditor

Role 3 is optional in 2-role mode and recommended in 3-role mode.

Role 3 performs the final broad audit after all target sections are nominally closed.

Primary responsibilities:

1. Read the full target spec or plan, including the audit trail and implementation reports.
2. Audit the actual codebase against the claims made by Role 1 and Role 2.
3. Look for:
   - missed issues inside already-closed sections
   - inconsistencies between audits and implementation
   - adjacent issues that fit the spirit of the target phase or plan
4. When the target is a phase spec, active plan, or comparable project-planning file, also
   compare it against the live near-term and speculative backlog surfaces to identify overlap,
   already-parked work, and items that still fit the spirit of the current phase or plan.
   For onionskin, this usually means checking `multi-agent/tracking/KNOWN_ISSUES.md` and
   `multi-agent/tracking/BRAINSTORM.md`.
5. Write a wrap-up audit section at the end of `TARGET_FILE`, or in a user-designated wrap-up
   section if one already exists.
6. Update `CHANGELOG.md` for the wrap-up audit.
7. Update `HANDOFF.md` and `TASK.md` only if the wrap-up audit creates concrete next actions.
8. Separate actionable follow-up findings from future-looking observations so Role 1 can triage
   the implementation-worthy items cleanly.

Role 3 must not:

- edit code unless the user explicitly converts the round into an implementation round
- trust either audit reports or implementation reports without checking live code

Expected output from Role 3:

- a wrap-up audit section in `TARGET_FILE`
- a `CHANGELOG.md` entry describing the wrap-up audit
- a clear statement of whether the target file is fully closed or has remaining findings
- when relevant, a short overlap assessment against `KNOWN_ISSUES.md` and `BRAINSTORM.md`
   identifying which findings are already parked versus newly surfaced
- when relevant, a clearly labeled actionable-follow-up block that can be handed back to Role 1

Expected at the end of the Final Overseer Wraup-Up audit:
- At the end of your chat message to the user, provide the following prompt to give to the Role 1 Auditor Agent:
```text
Use this prompt for the Auditor Agent:

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
- where path/to/TARGET_FILE = `TARGET_FILE` you were working on
- where path/to/SECTION_HEADING = `TARGET_SECTION` you were working in


---

## Shared Rules

These rules apply to every role.

1. Work one target section at a time.
2. Inspect live code rather than trusting markdown reports.
3. Include `scripts/` and `tests/` in audits and re-audits whenever they are plausibly affected.
4. Keep the audit trail in the target spec or plan file; do not scatter it across unrelated files.
5. Record every audit and every implementation round in `CHANGELOG.md`.
6. Update `HANDOFF.md` and `TASK.md` whenever a round changes the next actionable state.
7. Follow `multi-agent/AGENT_CONVENTIONS.md` for authorship, timestamps, and approval rules.
8. Do not rewrite history. Append new audit or implementation sections instead of erasing prior
   findings.
9. When code reality differs from the written audit, prefer correctness, then document the
   divergence explicitly.
10. The Final Overseer audit is normally a single pass. If it finds actionable work, the loop
    returns to Role 1 -> Role 2 -> Role 1. Role 3 does not usually re-enter unless the user asks.
11. Surface scope questions explicitly. If adjacent work clearly falls within the spirit of the
    current phase, flag it as a broadening opportunity in the audit output — do not silently add
    it, but also do not relegate it to KNOWN_ISSUES without asking. The user is the sole authority
    on all scope decisions. See `multi-agent/AGENT_CONVENTIONS.md` for the full scope authority
    rule.

---

## Standard Execution Loop

### Step 1 — Initial Audit

Role 1 audits `TARGET_SECTION`, determines whether it is open or closed, writes findings and exact
instructions into `TARGET_FILE`, and records the audit in `CHANGELOG.md`.

### Step 2 — Implementation

After explicit user approval, Role 2 implements the audited open items, runs the justified
validation, appends an implementation report into the same section, and updates the required
project-context files.

### Step 3 — Re-Audit

Role 1 reads the actual changed code and decides whether the section is now closed or still open.
If still open, Role 1 writes new findings and instructions into the same section and records the
re-audit in `CHANGELOG.md`.

### Step 4 — Iterate Until Closed

Repeat Steps 2 and 3 until the section is explicitly closed by an auditing role.

### Step 5 — Section Transition

When one section is closed, move to the next user-designated section and repeat the same loop.

### Step 6 — Final Wrap-Up Audit

When all intended sections are nominally closed, Role 3 performs a full-file wrap-up audit.
In 2-role mode, Role 1 performs this step.

### Step 7 — Post-Wrap-Up Remediation Loop

If the wrap-up audit finds actionable issues:

1. Role 1 reviews the wrap-up audit and decides which findings become active repair work.
2. Role 1 translates those actionable findings into concrete instructions in the authoritative
   target section(s) of `TARGET_FILE`.
3. After explicit user approval, Role 2 implements that post-wrap-up repair set and reports the
   validation.
4. Role 1 performs the final closeout audit.
5. Role 3 usually does not re-audit this second pass unless the user explicitly requests it.

---

## Required File Updates

Minimum required artifacts for onionskin:

1. `TARGET_FILE`
   - Role 1 writes audit findings and closeout judgments
   - Role 2 writes implementation reports
   - Role 3 writes the wrap-up audit
2. `CHANGELOG.md`
   - every audit round
   - every implementation round
   - the final wrap-up audit
3. `multi-agent/project_context/HANDOFF.md`
   - update when the round changes what the next agent should do
4. `multi-agent/project_context/TASK.md`
   - update when the round changes the immediate tactical queue

If the wrap-up audit opens new actionable work, `HANDOFF.md` and `TASK.md` should move back from
"closed" language to the specific next repair action owned by Role 1 or Role 2.

Optional additional artifacts when relevant:

- `ROADMAP.md` if a phase or priority status changes
- `DECISIONS.md` if the work settles a reusable design decision
- other docs/specs if the audited section explicitly requires synchronization

---

## Acceptance And Closeout Rules

A target section is closed only when all of the following are true:

1. An auditing role has inspected the actual changed code.
2. The findings in the latest audit are either resolved or explicitly reclassified.
3. The implementation round recorded its validation surface.
4. `TARGET_FILE`, `CHANGELOG.md`, `HANDOFF.md`, and `TASK.md` are consistent with the latest state.
5. No high-confidence open findings remain in the audited section.

The full spec or plan is closed only when all targeted sections are closed and the wrap-up audit
does not identify remaining blocking findings, or when any actionable wrap-up findings have gone
through the post-wrap-up remediation loop and been closed by Role 1.

---

## Common Failure Modes

Watch for these repeatedly:

1. Role 1 trusts the implementation report instead of reading the changed code.
2. Role 2 trusts the audit blindly and misses obvious nearby drift.
3. One role audits only core modules and forgets `scripts/` or `tests/`.
4. A role updates the spec or plan but forgets `CHANGELOG.md`.
5. `HANDOFF.md` or `TASK.md` lag behind the actual state.
6. A role silently narrows or broadens scope. The failure is the silence, not the scope change:
   raising a scope expansion as an explicit finding is correct behavior; quietly omitting work or
   relegating it to KNOWN_ISSUES without user approval is not.
7. A role edits old audit text instead of appending a new audit or implementation subsection.
8. Role 3 mixes actionable findings and future-phase suggestions without labeling which is which.
9. Role 1 leaves Final Overseer findings only in the wrap-up section instead of translating the
   actionable ones into the authoritative execution section(s).
10. Role 3 checks only the live code and spec text, but forgets to compare the target plan
   against `KNOWN_ISSUES.md` and `BRAINSTORM.md` for overlap and in-scope follow-up.

---

## Prompt Templates

Use these as compact launch prompts.

### Template A — Role 1 Initial Audit

```text
You are acting as Role 1 — Auditor / Instruction Writer from
multi-agent/workflows/spec_plan_three_role_audit_loop.md.

Target file: {TARGET_FILE}
Target section: {TARGET_SECTION}

Audit this section deeply against the live codebase. Determine whether it is closed or open.
If it is open, append audit findings and exact implementation instructions into the same section,
including file references, function references, and concrete repair notes. Update CHANGELOG.md for
the audit. Read actual code rather than trusting prior reports. Include scripts/ and tests/ when
relevant.
```

### Template B — Role 2 Implementation

```text
You are acting as Role 2 — Implementer / Supplemental Auditor from
multi-agent/workflows/spec_plan_three_role_audit_loop.md.

Target file: {TARGET_FILE}
Target section: {TARGET_SECTION}

Read the audited findings and implementation instructions in the target section, then implement
them as closely as possible. While working, inspect nearby code for audit misses in the same
surface area. If you need to diverge, document the reason in your implementation report under the
same section. Update CHANGELOG.md, HANDOFF.md, and TASK.md. Report the tests you ran.
```

### Template C — Role 1 Re-Audit

```text
You are acting as Role 1 — Auditor / Instruction Writer from
multi-agent/workflows/spec_plan_three_role_audit_loop.md.

Target file: {TARGET_FILE}
Target section: {TARGET_SECTION}

Role 2 has finished an implementation round. Re-audit the live code and determine whether this
section is now closed or still open. Read the actual changed code rather than trusting the
implementation report. If it is still open, append new findings and exact repair instructions into
the same section and update CHANGELOG.md. Include scripts/ and tests/ when relevant.
```

### Template D — Role 3 Final Wrap-Up Audit

```text
You are acting as Role 3 — Final Overseer / Wrap-Up Auditor from
multi-agent/workflows/spec_plan_three_role_audit_loop.md.

Target file: {TARGET_FILE}

All intended sections are nominally closed. Perform a full wrap-up audit of the spec or plan,
the audit trail, and the actual codebase. Do not edit code. Report any missed issues, audit-trail
inconsistencies, or adjacent issues that fit the spirit of the target plan. When relevant, also
compare the target spec or plan against `multi-agent/tracking/KNOWN_ISSUES.md` and
`multi-agent/tracking/BRAINSTORM.md` to report overlap, already-parked items, and any findings that still
belong in the spirit of the current phase or project. Write your report in the designated wrap-up
section at the bottom of the target file, and update CHANGELOG.md for the audit. Use the heading
`## Wrap-up audit with Final Overseer` unless the target file already has an explicit wrap-up
heading.
```

### Template E — Role 1 Post-Wrap-Up Triage

```text
You are acting as Role 1 — Auditor / Instruction Writer from
multi-agent/workflows/spec_plan_three_role_audit_loop.md.

Target file: {TARGET_FILE}

Role 3 has finished the wrap-up audit and found additional actionable items. Review the wrap-up
audit against the live code, decide which findings should become active repair work now, and
translate them into concrete implementation instructions in the authoritative target section(s)
of the spec or plan. Do not rewrite the Final Overseer report. Update CHANGELOG.md and sync
HANDOFF.md plus TASK.md to point to the new next action.
```

### Template F — Role 2 Post-Wrap-Up Implementation

```text
You are acting as Role 2 — Implementer / Supplemental Auditor from
multi-agent/workflows/spec_plan_three_role_audit_loop.md.

Target file: {TARGET_FILE}

Role 1 has translated actionable wrap-up findings into repair instructions. Implement only that
triaged subset, perform the justified validation, append your implementation report in the
authoritative target section(s), and update CHANGELOG.md, HANDOFF.md, and TASK.md.
```

### Template G — Role 1 Final Closeout After Wrap-Up Remediation

```text
You are acting as Role 1 — Auditor / Instruction Writer from
multi-agent/workflows/spec_plan_three_role_audit_loop.md.

Target file: {TARGET_FILE}

Role 2 has completed the post-wrap-up remediation round. Re-audit the live code, determine
whether the actionable findings from the Final Overseer audit are now resolved, and append the
final closeout judgment in the relevant section(s). Update CHANGELOG.md and sync the project
context files if the target file is now fully closed.
```

---

## 2-Role Mode

If only two agents are available:

1. Role 1 performs both the initial audits and the final wrap-up audit.
2. Role 2 performs implementation plus in-situ supplemental auditing.
3. Keep the same file-update and closeout rules.

---

## Notes For The User

This file works best when you tell each agent its role before asking it to read the file.
The workflow is intentionally more structured and less verbose than a raw historical prompt,
but the prompt templates are designed to preserve the behavior of a successful human-tested
multi-agent loop.