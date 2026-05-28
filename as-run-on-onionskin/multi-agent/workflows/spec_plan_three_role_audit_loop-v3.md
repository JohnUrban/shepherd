# SPEC / PLAN THREE-ROLE AUDIT LOOP — v3

Reusable coordination prompt for **a Principal (human) + an Orchestrator (AI;
phase-spanning) + 2-3 audit-loop agents (R1/R2/R3)** working through a spec or
plan file one **cycle** at a time. A cycle covers one substantive priority or
one batched group of thin priorities.

This workflow is agent-agnostic at the audit-loop level. Any capable agent may
fill R1/R2/R3. The Orchestrator role is recommended to be a single AI agent
(typically Claude Code Opus Max) that lives across the whole phase as an
orchestration layer between the Principal and the audit-loop agents. See the
Orchestrator section below.

**Revision markers:**

- **v3 (current).** Adds the Orchestrator role formally — a phase-spanning AI
  agent that absorbs the prior Strategist function (Template I) as its kickoff
  act and continues across cycle execution as an orchestration layer between
  the Principal and the audit-loop agents. Adds Succession Briefing
  (Template J) as the outgoing Orchestrator's wind-down deliverable.
  Acknowledges multi-chat continuity within an Orchestrator role (state
  transfer via memory files + planning surfaces). Documents the bidirectional
  deliberation pattern between Principal and Orchestrator that shapes scope
  evolution mid-phase. v2 remains available at
  `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md` for reference.
- **v2.** Introduced: (1) a sibling `AUDIT_LOG.md` file so the SPEC stays clean
  during implementation, (2) per-cycle CHANGELOG batching, (3) a
  substantive-priorities rule, (4) cycle granularity (substantive vs batched),
  (5) authorship consolidation at cycle closeout, and (6) a safety-gated Role 2
  skip-reaudit declaration. v1 remains available at
  `multi-agent/workflows/spec_plan_three_role_audit_loop-v1.md`.

**CHANGELOG vs DEVLOG note (from v0.14.64):** Throughout this file, every
reference to "the CHANGELOG entry at cycle closeout" should be read as "the
CHANGELOG entry **or** the DEVLOG entry at cycle closeout, by cycle scope."
Cycles whose primary work was onionskin product (code, CLI, tests, user-facing
docs) close to `CHANGELOG.md`. Cycles whose primary work was dev-system
(workflow files, `AGENT_CONVENTIONS.md`, agent bootstrap files, SPEC/BRAINSTORM/
FEEDBACK engineering with no code changes) close to `multi-agent/DEVLOG.md`.
See `AGENT_CONVENTIONS.md` for the full routing rule. The two files share a
single `v0.X.YY` version stream; a version appears in exactly one file.

---

## Purpose

Use this workflow when the user wants to close out an active spec or plan by repeating a
disciplined loop:

1. audit one cycle deeply
2. write exact findings and instructions into that cycle's `AUDIT_LOG.md` section
3. implement only those audited open items
4. re-audit the implementation against live code (OR Role 2 declares safety-gated skip)
5. repeat until the cycle is closed
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
- The user wants one cycle audited at a time rather than broad simultaneous implementation.
- The user wants findings and implementation history captured for review.
- The user wants cycle closeouts recorded in `CHANGELOG.md`.

Do not use this workflow for:

- small one-off bug fixes that do not need a spec-driven loop
- purely read-only audits with no implementing role
- large exploratory design work that is not yet organized into auditable sections

---

## Required Inputs

Before any role starts, the user should provide or confirm:

1. `TARGET_FILE`: the spec or plan file to use as the authoritative working contract
2. `AUDIT_LOG_FILE`: the sibling audit-log file
   Default: if TARGET_FILE is `multi-agent/plans/PHASE<N>_SPEC.md` (or
   `PHASE<N>_SUPPLEMENTAL-SPEC.md`), then AUDIT_LOG_FILE is the sibling
   `multi-agent/plans/PHASE<N>_AUDIT_LOG.md` (or
   `PHASE<N>_SUPPLEMENTAL-AUDIT_LOG.md`).
3. `TARGET_CYCLE`: the cycle being worked in the current round. A cycle is
   either:
   - one substantive priority (see Cycle Granularity below), OR
   - a named batched group of thin priorities (e.g., "Phase 1 renames" covering
     14-S4 + 14-S5 + 14-S29 + 14-S30)
4. `UPDATE_FILES`: files that must be synced at cycle boundaries
   Default set for onionskin:
   - `CHANGELOG.md` — at cycle closeout only
   - `multi-agent/project_context/HANDOFF.md` — at any state change
   - `multi-agent/project_context/TASK.md` — at any state change
   - `AUDIT_LOG_FILE` — every role-round
5. whether the current round is:
   - initial audit
   - implementation
   - re-audit
   - skip-reaudit declaration (Role 2 only)
   - final wrap-up audit
   - post-wrap-up triage
   - post-wrap-up implementation
   - post-wrap-up closeout
6. whether the workflow is operating in:
   - 2-role mode
   - 3-role mode

Suggested assignment line (substitute `<N>` with the current phase number; use concrete
filenames for supplemental phases whose paths deviate from `PHASE<N>_*.md`):

```text
Below N and <N> = <phase number to be filled in by user>

You are acting as {ROLE_NAME} from multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md.
Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md
Target cycle: <cycle name> or <cycle index>
Current round: {ROUND_TYPE}
```

---

## The Orchestrator (phase-spanning AI role; v3)

**One-line summary:** the Orchestrator is an AI agent (recommended: Claude
Code Opus Max) that initializes via the Strategist function (Template I; phase
strategy doc) at phase kickoff and continues across the whole phase as an
orchestration layer between the Principal (human; phase owner / scope
authority) and the audit-loop agents (R1/R2/R3). The Orchestrator role
**absorbs the prior Strategist function** — Strategist is no longer a
one-shot role but the kickoff act of the Orchestrator's lifecycle.

The Orchestrator does NOT replace the Principal's role as scope authority.
Per `multi-agent/AGENT_CONVENTIONS.md § Scope authority`, the Principal is
the sole decision-maker on scope. The Orchestrator surfaces options + drafts
artifacts; the Principal authorizes, refines, or redirects.

### Why the role exists

- **Cross-cycle pattern memory.** Phase 15's GAP-1 pattern (cycle 15.4a-S3
  Copilot R2 missed the structural parity table; R3 had to remediate inline)
  is exactly the kind of recurring blind spot a phase-spanning Orchestrator
  flags pre-emptively. Without the Orchestrator, each cycle's R1/R3 must
  rediscover patterns the prior cycle already surfaced.
- **Buffer the Principal.** R1/R2/R3 are typically launched in fresh chat
  sessions. The Orchestrator carries phase context so the Principal does not
  have to re-bootstrap context with every audit-loop agent.
- **Catch what falls between roles.** Authorship/Effort attribution drift,
  scope drift, help-text omissions, mid-cycle scope shifts — these don't
  belong inside any single audit-loop role's deliverable but accumulate
  across the phase. The Orchestrator owns them.
- **Translate empirical results back to design.** When R2's implementation
  produces unexpected results (e.g., cycle 15.4a-S4's GMM `learn` mode
  artifacts), the Orchestrator helps interpret + recommends pivots before
  R3 begins, often via orchestrator-outside-cycle clarifications appended
  to the audit log under append-only discipline.
- **Reduce per-agent context cost.** Audit-loop agents in fresh chats can
  start clean without inheriting earlier-cycle deliberation; the
  Orchestrator carries that and feeds only what's relevant via prompts +
  audit-log clarifications.

### Lifecycle

The Orchestrator role spans the full phase across three lifecycle stages:

**KICKOFF (Strategist function).** At phase start, the Orchestrator
initializes by producing the phase strategy doc at
`multi-agent/plans/PHASE<N>_STRATEGY.md` per **Template I — Strategist:
Phase Plan of Attack** below. STRATEGY.md splits the phase into
execution-ordered cycles, assigns primary + alternative agents per role per
cycle, and states deviations from the baseline strategy. This is the
Orchestrator's first deliverable. Run once per phase; amend (never rewrite)
as the phase progresses.

**ONGOING (cycle execution).** Across all cycles in the phase, the
Orchestrator:

- Drafts R1/R2/R3 launcher prompts and mid-flight pushback notes (rich
  prompts with locked-contract checklists, validation gate lists, scope
  boundaries, pushback discipline). The Principal launches the prompts in
  fresh chats.
- Pre-checks R2 output before R3 launches (catches issues cheaply; cycle
  15.4a-S3 GAP-1 was caught this way).
- Authors orchestrator-outside-cycle clarifications when scope shifts
  mid-cycle. These are appended to the cycle's audit-log section under
  append-only discipline (e.g., cycle 15.4a-S3 corrections appendix; cycle
  15.4a-S4 FIRST + SECOND clarifications).
- Captures decision history in `multi-agent/plans/PHASE<N>_BACKGROUND.md`
  (per-phase planning scratchpad; promoted from `<theme>.tmp.md` when
  deliberation captures non-trivial decision history worth preserving).
- Updates STRATEGY.md with mid-phase amendments + amendment-log entries
  when cycle ordering, scope, or assignments change.
- Updates SPEC.md when scope decisions affect priorities (rare; typically
  via SPEC priority status table changes at cycle closeout).
- Triages SURPRISE_LOG entries at cross-cycle level (deciding whether a
  surprise authorizes a supplemental cycle or stays Active for future-phase
  pickup).
- Authors DEVLOG entries for planning-surface engineering and
  orchestrator-housekeeping commits (e.g., Effort: Max sweeps, SOUP
  additions, mid-cycle clarifications). DEVLOG entries follow the routing
  rule in `AGENT_CONVENTIONS.md`.
- Drafts commit messages per `feedback_commit_message_file_convention.md`
  (scratch file at repo root as `changelog-entry-<version>.txt`).
- Maintains authorship/Effort consistency across artifacts (catches drift;
  applies retroactive corrections as a discrete commit).
- Adds SOUP entries (`multi-agent/plans/next/<THEME>_SOUP.md`) for deferred
  future-phase work surfaced during cycle execution.
- Translates empirical results back to design decisions through deliberation
  with the Principal (see Deliberation pattern below).

**WIND-DOWN (phase close).** After the final R3 closes the last cycle:

1. Final Overseer (separate fresh-chat role) does the end-of-phase
   independent review. **The Orchestrator does NOT become the Final
   Overseer** — Final Overseer's value is structural independence from the
   Orchestrator's accumulated frame.
2. After Final Overseer signs off, the outgoing Orchestrator writes the
   **Succession Briefing** at `multi-agent/plans/PHASE<N>_SUCCESSION_BRIEFING.md`
   per **Template J — Succession Briefing** below. The briefing transfers
   wisdom (cross-cycle pattern memory, recurring blind spots, deliberation
   patterns that worked, things that didn't, brainstorm seeds, open
   cross-phase entries, recommended working style) to the next-phase
   Orchestrator. The briefing is NOT a recreation of the formal artifacts
   (SPEC/AUDIT_LOG/STRATEGY/BACKGROUND/SURPRISE_LOG); those archive intact.
3. Phase archive: SPEC + AUDIT_LOG + STRATEGY + BACKGROUND + SURPRISE_LOG +
   SUCCESSION_BRIEFING all move to `multi-agent/plans/archived/<date>-PHASE<N>_*.md`.
4. Outgoing Orchestrator's chat closes.
5. Next-phase Orchestrator (recommended: fresh chat, possibly different AI
   instance) bootstraps by reading the archived prior-phase artifacts +
   Succession Briefing.

### Entry point: starts at SOUP

**The Orchestrator role starts at SOUP — when a candidate future-phase
SOUP file (`multi-agent/plans/next/<THEME>_SOUP.md`) is being shaped or
when the Principal is bringing a SOUP onto the stage as the next-phase
BRAINSTORM.** The role then continues through:

- SOUP → BRAINSTORM transition (per-phase BRAINSTORM file generation)
- BRAINSTORM iteration rounds (Agent 1 writer + Agent 2 auditor +
  Orchestrator deliberating with Principal on convergence)
- SPEC engineering (deriving SPEC priorities from BRAINSTORM)
- STRATEGY kickoff (Template I; phase strategy doc — the Orchestrator's
  formal first deliverable as it transitions into cycle-execution mode)
- Cycle execution (R1/R2/R3 rounds across all cycles in the phase)
- Wind-down (Final Overseer pass + Succession Briefing emission +
  phase archive)

This is the **default entry point**. Starting at SOUP gives the
Orchestrator the strongest cross-cycle pattern memory because the
deliberation patterns that emerge in brainstorming + SPEC engineering
carry through to cycle execution.

**Late entry is acceptable.** If circumstances require an Orchestrator
to join at the Strategist stage (or even mid-cycle-execution), the
incoming chat bootstraps from the existing artifacts (SPEC + BRAINSTORM
+ SOUP + recent audit-log entries + feedback memories). Some pre-cycle
deliberation context is lost; the SPEC contract is inherited cleanly.
This is suboptimal but workable when the long form isn't feasible.

The Principal chooses the entry point. Phase 15 used a hybrid (some
Orchestrator-like activity from BRAINSTORM stage onward across multiple
chats; formal Strategist-onward activity from v0.14.76.5 SPEC closeout).

### Multi-chat continuity within a single Orchestrator role

**The Orchestrator role is conceptually always there, but is NOT
required to be chat-continuous.** A single Orchestrator role typically
spans many cycle rounds and may exceed a single chat session's
practical context window. The role is therefore **role-continuous, not
chat-continuous**. The Principal may hand off to a new Orchestrator
chat as needed — and SHOULD do so when the existing chat's accumulated
context starts to be a hindrance rather than a help (e.g., context
window approaching limits, conversational drift, or the Principal
simply prefers a fresh chat for a clean pass).

Handoff between successive Orchestrator chats is normal + expected, not
a fallback. State transfer happens via:

1. **Memory files** at `~/.claude/projects/<project>/memory/` (auto-memory)
   plus project-specific memory in `multi-agent/project_context/` and
   feedback memories under the agent's persistent memory store.
2. **Planning surfaces** — STRATEGY.md, BACKGROUND.md, AUDIT_LOG.md,
   SPEC.md, SURPRISE_LOG.md — all read on cold-start by a successor
   Orchestrator chat.
3. **The Principal's continuous through-line.** The Principal is the
   sole entity with literal continuity across all chat sessions; they
   feed forward whatever isn't yet captured in artifacts.

A successor Orchestrator chat should perform a cold-start read of the
phase artifacts + recent feedback memories before participating in
deliberation. The Principal flags specific artifacts as load-bearing
for the current cycle. If the prior Orchestrator chat captured
non-trivial decision history that didn't make it into formal artifacts,
the Principal can paste relevant excerpts into the new chat as
bootstrap.

**Heuristics for when to hand off to a fresh Orchestrator chat:**

- Context window noticeably approaching capacity (compactions starting
  to lose load-bearing detail).
- Conversational drift — recent turns feel inefficient compared to
  earlier in the chat.
- Stage transition (e.g., BRAINSTORM closing → SPEC engineering opening
  → STRATEGY kickoff). Natural reset points.
- Mid-phase amendment that substantially re-scopes the cycle queue.
  Fresh chat helps re-bootstrap with the new scope clear.
- Principal preference. If the Principal wants a fresh perspective for
  the next deliberation round, that's sufficient reason.

**Heuristics for keeping the same Orchestrator chat:**

- Mid-cycle deliberation that depends on accumulated context from
  earlier in the chat (e.g., interpreting Stage A.5 results in light
  of R1's audit + first orchestrator clarification).
- Quick-turn artifact emission where bootstrapping cost > continuation
  cost.
- Active mid-flight pushback note authoring.
- The Principal-Orchestrator deliberation has built up vocabulary +
  shorthand that would be expensive to rebuild.

The choice is per-handoff judgment, not a hard rule. The role
continues; the chat instance is the implementation detail.

### Deliberation pattern (Principal ↔ Orchestrator)

The Orchestrator is not a passive context buffer. The role works through
**bidirectional deliberation** with the Principal that actively shapes
scope evolution mid-phase.

Concrete patterns observed in Phase 15:

- **Orchestrator surfaces options.** Orchestrator drafts artifacts +
  surfaces design options (e.g., "Option A: in-place upgrade; Option B:
  two values; Option C: hybrid").
- **Principal pushes back, refines, redirects.** The Principal selects,
  refines naming, adjusts defaults, sometimes rejects all options and
  redirects entirely (e.g., "drop learn mode entirely"). The
  Orchestrator's first proposal is rarely the final landing.
- **Orchestrator pushes back on Principal's first instinct.** When
  Orchestrator sees a deferred risk the Principal hasn't surfaced,
  Orchestrator flags it (e.g., "the empirical defaults are dataset-overfit;
  biology priors are more robust"). The Principal may accept, reject, or
  refine.
- **Mid-flight pushback notes.** When an audit-loop agent is hitting
  unexpected results, the Orchestrator drafts a pushback note + the
  Principal injects it into the running agent's chat.
- **Empirical results trigger pivots.** When Stage A diagnostic results
  surface that the locked algorithm doesn't fit reality, the Orchestrator
  helps interpret + drafts orchestrator-outside-cycle clarifications. The
  Principal authorizes the pivot.
- **Scope decisions ALWAYS authorized by Principal.** Per
  `multi-agent/AGENT_CONVENTIONS.md § Scope authority`, narrowing is
  prohibited without Principal approval; broadening is encouraged but
  surfaced and acknowledged. The Orchestrator does not silently descope
  or expand.

This deliberation pattern produces scope evolution mid-phase that's
disciplined (recorded in BACKGROUND.md + audit-log clarifications) but
responsive (not locked by R1's initial audit alone). The role only works
if the deliberation is bidirectional — a passive context-buffer would
not catch what's surfaced through these exchanges.

### Boundaries (what the Orchestrator does NOT do)

- **Does not run git commands.** Per `feedback_git_control.md`, the
  Principal owns all git operations. Orchestrator emits commit blocks +
  scratch files (per `feedback_commit_message_file_convention.md`) for
  the Principal to run.
- **Does not write production code.** That's R2's job. Orchestrator may
  draft code-shaped pseudocode in audit-log clarifications when
  specifying behavior, but does not commit code.
- **Does not narrow scope without Principal approval.** Per Scope
  Authority. Broadening may be surfaced and recommended; the Principal
  authorizes.
- **Does not act as Final Overseer.** Final Overseer must be structurally
  independent from the Orchestrator's accumulated frame. The wind-down
  Succession Briefing is the Orchestrator's last act; Final Overseer
  precedes it as a fresh-chat independent review.
- **Does not override audit-log append-only discipline.** Corrections
  appendices APPEND; never edit prior R1/R2/R3 entries. The trajectory
  (R1 audit → corrections appendix → R2 implementation → R3 closeout) is
  preserved as decision history.
- **Does not bypass the Principal's authorization for design pivots.**
  Even when the Orchestrator identifies a clear flaw (e.g., GMM `learn`
  mode artifacts), the pivot lands only after Principal authorization.

### Interaction model

**With Principal (human; scope authority):**

- Principal feeds Orchestrator the chat output + audit log entries from
  audit-loop agent sessions.
- Orchestrator integrates, identifies what landed/missed, flags drift.
- Principal asks Orchestrator for pre-checks before R3 launches to catch
  issues cheaply.
- Principal + Orchestrator deliberate strategy + scope; Orchestrator
  drafts artifacts; Principal approves or pushes back.
- Mid-flight: Principal tells Orchestrator what an agent is hitting in
  real time; Orchestrator drafts pushback notes; Principal injects.
- Orchestrator emits commit blocks; Principal runs them.

**With audit-loop agents (R1/R2/R3) — indirect:**

- Through prompts the Orchestrator drafts and the Principal launches in
  fresh chats.
- Through orchestrator-outside-cycle clarifications appended to the
  cycle's audit-log section, which the audit-loop agents read on
  cold-start.
- Through pre-checks by the Orchestrator that catch issues before R3
  begins, sometimes triggering inline-R2 surgical-fix prompts the
  Principal can deliver.
- Through commit messages the Orchestrator drafts that carry the
  framing the Orchestrator shaped.

**With Final Overseer — indirect:**

- The Orchestrator does not directly brief the Final Overseer.
- Final Overseer reads the archived phase artifacts + reaches its own
  conclusions independently.
- After Final Overseer signs off, Orchestrator writes the Succession
  Briefing incorporating Final Overseer's findings.

### Recommended model + agent choice

- **Claude Code Opus 4.7 — Effort: Max** is the recommended primary.
  Reasoning quality + long-context handling matter more than speed.
- **Codex GPT-5.5 (Reasoning: Extra High)** is acceptable as alternative.
- **Gemini is excluded** per project memory `feedback_role_selection_priors.md`
  (historical thin/shallow audits; once destroyed the codebase during
  implementation).
- **Github Copilot** is acceptable for narrow R2 implementation roles but
  is NOT recommended for the Orchestrator role — its working style
  (target-and-go) doesn't match the orchestration role's cross-cycle
  pattern recognition + bidirectional deliberation requirements.

### Decision guide for cycle transitions (migrated from orchestrator companion file under v3)

When uncertain what prompt to draft next during cycle execution, use this
decision tree (Principal-Orchestrator joint decision; Orchestrator drafts,
Principal authorizes):

1. **New cycle starts** → draft Auditor Template A launcher (Role 1
   initial audit).
2. **Auditor wrote instructions** → draft Implementer Template B launcher
   (Role 2 implementation).
3. **Implementer finished with "re-audit needed" declaration** → draft
   Auditor Template C launcher (Role 1 re-audit).
4. **Implementer finished with "skip-reaudit recommended"**:
   - If Principal ACCEPTS the skip → draft Auditor Template H launcher
     (Role 1 cycle closeout after skip-reaudit).
   - If Principal REJECTS the skip (force full re-audit) → draft Auditor
     Template C launcher instead.
5. **Cycle closed, next cycle queued** → draft Auditor Template A launcher
   for the next cycle (or Template H already covers this transition if
   skip-reaudit was accepted).
6. **All cycles closed** → draft Final Overseer Template D launcher.
7. **Final Overseer found actionable work** → draft Auditor Template E
   launcher (triage into post-wrap-up remediation cycle).
8. **Auditor triaged findings** → draft Implementer Template F launcher.
9. **Post-wrap-up implementation finished** → draft Auditor Template G
   launcher (final closeout).
10. **Final Overseer signed off + post-wrap-up remediation closed** →
    Outgoing Orchestrator writes the Succession Briefing per Template J.
    Then phase archive (Principal runs `git mv`).

#### Deciding whether to accept a Role 2 skip-reaudit

Default to **NO skip** (force full re-audit) when any of these apply:

- The cycle involves design judgment or help-string quality.
- The cycle is a substantive single-priority cycle (not a thin-priority
  batched cycle).
- The cycle touches runtime wiring, not just parser surface.
- Tests did not run green on the first try.
- The Principal-Orchestrator pair is new to v3; build trust in the skip
  criteria before using them liberally.

Default to **YES, accept skip** only when ALL skip criteria are met AND
the cycle is a batched mechanical cycle (renames, help-string appends
with no judgment, flag additions with clear spec, etc.).

Even when accepting a skip, Role 1 still does a lightweight spot-check at
closeout (Template H). That's the backstop.

### What gets archived at phase close

At phase close, the following move to `multi-agent/plans/archived/<date>-PHASE<N>_*`:

- `PHASE<N>_SPEC.md`
- `PHASE<N>_AUDIT_LOG.md`
- `PHASE<N>_STRATEGY.md`
- `PHASE<N>_BACKGROUND.md`
- `PHASE<N>_SURPRISE_LOG.md`
- `PHASE<N>_SUCCESSION_BRIEFING.md` (NEW under v3)
- Any per-phase `BRAINSTORM.md` or `FEEDBACK.md` if not already archived

Archive operations are **orchestrator-only** per project memory
`feedback_archive_operations_orchestrator_only.md` (i.e., the Principal
runs the `git mv` operations; audit-loop agents propose but do not
execute).

---

## Pre-cycle: Phase Strategy (STRATEGY.md) — produced by the Orchestrator at kickoff

Before running the first audit cycle of a new or reopened phase, the
**Orchestrator** produces a plan-of-attack document at
`multi-agent/plans/PHASE<N>_STRATEGY.md` (sibling to `PHASE<N>_SPEC.md` and
`PHASE<N>_AUDIT_LOG.md`) using **Template I — Strategist: Phase Plan of
Attack** below. STRATEGY.md splits the phase into execution-ordered cycles,
assigns primary + alternative agents per role per cycle, and states
deviations from the baseline strategy.

This is the Orchestrator's kickoff deliverable. Run once per phase; the
output drives cycle execution and is amended (never rewritten) as the phase
progresses with mid-phase amendment-log entries. At phase close, archive
STRATEGY.md alongside SPEC, AUDIT_LOG, BACKGROUND, SURPRISE_LOG, and the
Succession Briefing.

Under v3, the Strategist is no longer a separate one-shot role; the
Strategist function is the kickoff act of the Orchestrator's lifecycle.

---

## Cycle Granularity — substantive vs batched

This is a new v2 concept. **Do not run a full audit-implement-reaudit loop on
single-line flag renames or trivial help-string tweaks.** Bundle thin work into
substantive cycles.

### Substantive priority

A priority is substantive when any one of these is true:
- It plausibly requires multi-file implementation across 3+ surfaces.
- It involves design judgment (help-string quality bar, terminology harmonization,
  cross-pipeline framing).
- It touches runtime wiring (not just parser surface).
- A Role 1 audit would reasonably produce 2+ concrete findings.

Substantive priorities get their **own cycle**: one Role 1 audit → one Role 2 implement
→ one Role 1 re-audit (or Role 2 skip-reaudit declaration) → closeout.

### Thin priority

A priority is thin when all of these are true:
- Implementation is a single rename, single help-string edit, or single trivial wire.
- No ambiguity in what the final state should be.
- A Role 1 audit would produce ≤1 concrete finding (essentially just "do what the
  priority says").
- Testing surface is covered by the existing test suite without new fixtures.

Thin priorities **must be batched** into a named batched cycle with related work.
Common batch units:
- A phase group (e.g., Phase 1 of a reordered SPEC).
- A theme group (e.g., "all the audit-only deliverables").
- A single Role 2 round spans the whole batched cycle.

Rule of thumb: **prefer 5-10 substantive cycles per phase over 30 thin cycles.** The
three-role loop is expensive; every cycle should justify the cost.

### Who decides substantive vs thin

- When the SPEC is engineered, the SPEC's phase/group structure (per PDS-v2)
  usually implies the cycle granularity. See PDS-v2 on substantive priorities.
- At Role 1 initial audit, Role 1 may propose consolidation ("these three
  priorities are thin — recommend batching"). Orchestrator approves.
- Never batch priorities across phase boundaries. Phase structure encodes hard
  dependencies.

### Deferred R3 (bundled into next cycle's R3)

Some cycles do not merit a standalone Role 3 closeout round — typically
audit-only cycles that produced tracking documents or appendices with no
runtime code changes. Rather than run a full R3 session that spot-checks
trivial deliverables, the cycle may **defer its R3 scope to the next cycle's
R3 auditor**.

Rules for deferring R3:

1. **Eligibility.** A cycle may defer R3 only when its primary output is
   documentation or audit artifacts and it contains no runtime code changes
   (or only trivial help-string edits already covered by R1 initial audit).
   Substantive runtime-wiring cycles MUST run their own R3.
2. **Closeout still happens.** The deferring cycle still closes with its own
   CHANGELOG/DEVLOG entry and its own priority-status-table update. R1
   performs a lightweight closeout (Template H-style skip-reaudit flow), then
   records in the cycle-closeout entry that "R3 scope deferred to cycle
   <absorbing cycle>."
3. **Absorbing cycle's R3 scope grows.** The next cycle's R3 auditor adds
   "Deferred verification from cycle <X>: <one-line scope>" to their closeout
   scope. They re-check the deferred cycle's deliverables before signing off.
4. **Naming in STRATEGY.md.** The deferring cycle's R3 cell must explicitly
   name the absorbing cycle (e.g., `R3: Defer to 14S.3a R3`). The absorbing
   cycle's Notes column must mirror the deferral (e.g., `R3 also verifies
   14S.2a deferred deliverables`).
5. **No chaining.** A cycle may only defer to the immediately-next R3. The
   absorbing cycle's R3 itself cannot be deferred.
6. **Final-cycle edge case.** If a deferring cycle is the last cycle before
   the Final Overseer pass, the Final Overseer absorbs the deferred scope
   instead. Call this out explicitly in STRATEGY.md.

---

## Target File Structure And Editing Conventions — v2

The target SPEC is the **authoritative contract**; it stays clean once engineering
closes. The sibling `AUDIT_LOG.md` is the **round-by-round history**; it grows as
the cycle runs.

### What goes in the SPEC file

1. Priority scope, goals, decisions (from engineering stage).
2. A small **priority status table** at the top listing each priority and its state:
   `OPEN`, `IN PROGRESS`, `CLOSED (v0.X.YY)`, `BATCHED WITH <cycle>`. The closing
   agent updates this row at cycle closeout.
3. Phase/group structure headings (if applicable).
4. Scope boundary blocks, help-string conventions, decisions references.

### What does NOT go in the SPEC file

- Role 1 audit findings (go to `AUDIT_LOG.md`).
- Role 2 implementation reports (go to `AUDIT_LOG.md`).
- Role 1 re-audit closeouts (go to `AUDIT_LOG.md`).
- Role 3 wrap-up audit (goes to `AUDIT_LOG.md`).
- Post-wrap-up triage, implementation, closeout (go to `AUDIT_LOG.md`).

The SPEC is edited only when **scope changes**, which requires explicit user approval
per the Scope Authority rule in `AGENT_CONVENTIONS.md`. Role 1 cannot silently narrow
or broaden; scope changes go through the user and are reflected in the SPEC.

### What goes in AUDIT_LOG.md

1. Dated, role-labeled subsections per round, newest-first throughout —
   both rounds within a cycle and cycles across the file. 'Newest' =
   timestamp of the most recent round in that cycle.
2. Each round-entry must include a `**Authors:**` line per AGENT_CONVENTIONS.
3. Structure:

   ```markdown
   # PHASE <N> AUDIT LOG

   ## Cycle: <cycle name> — CLOSED v0.X.YY
     or
   ## Cycle: <cycle name> — OPEN (Role <R> round N)

   ### <timestamp> — Role 1 Re-Audit (or Role 2 Skip-Reaudit Declaration)
   **Authors:** ...
   Closeout judgment: ...

   ### <timestamp> — Role 2 Implementation
   **Authors:** ...
   Implemented: ...
   Tests: ...
   Deviations: ...

   ### <timestamp> — Role 1 Initial Audit
   **Authors:** ...
   Findings: ...
   Repair instructions: ...

   ---
   ```

4. When a cycle closes, update the Cycle heading to `CLOSED v0.X.YY` and update the
   priority status table in the SPEC.

### Role writes per file

- Role 1 writes audit findings + exact instructions → `AUDIT_LOG.md`
- Role 2 writes implementation reports → `AUDIT_LOG.md`
- Role 1 writes re-audit closeouts → `AUDIT_LOG.md`
- Role 2 writes skip-reaudit declarations (when applicable) → `AUDIT_LOG.md`
- Role 3 writes the wrap-up audit → `AUDIT_LOG.md` under a clearly-marked wrap-up heading
- All roles MAY append peripheral-vision observations → `PHASE<N>_SURPRISE_LOG.md`
- Role 1 (Template C re-audit) MUST review and update statuses on this cycle's SURPRISE entries → `PHASE<N>_SURPRISE_LOG.md`
- SPEC receives only: priority status table updates, scope-change edits (user-approved)
- CHANGELOG receives: one entry per cycle at closeout

### What goes in PHASE<N>_SURPRISE_LOG.md

Per-phase sibling to SPEC + AUDIT_LOG + STRATEGY + FEEDBACK. Holds fresh-eyes
observations that did NOT make it into the formal AUDIT_LOG report because the
observation was out-of-scope, hygiene-drift-adjacent, a smell without concrete
repair, or a cross-cycle pattern. See `AGENT_CONVENTIONS.md § SURPRISE_LOG.md`
for the full convention. Key invariants:

- Status taxonomy prevents Active-forever drift: every entry passes through
  `Active` → `In-flight` → `Resolved (vX.Y.ZZ)`, with side states for
  promotion / supersession / logged-only.
- Re-audit closeouts (Template C) MUST review this cycle's SURPRISE entries
  and flip statuses appropriately — see Template C's SURPRISE review gate.
- Adding entries does NOT increment the version (no CHANGELOG/DEVLOG entry
  for SURPRISE_LOG additions).

### Phase startup — creating the per-phase sibling files

When a new phase begins (after SPEC engineering closes and the SPEC + STRATEGY
files are in place), the orchestrator (or the first cycle's Role 1 auditor)
ensures these per-phase sibling files exist before the first cycle fires:

- `PHASE<N>_AUDIT_LOG.md` — created with the standard header (see § What goes
  in AUDIT_LOG.md above). Entries section starts empty.
- `PHASE<N>_SURPRISE_LOG.md` — created by copying the preamble + Conventions
  sections verbatim from the latest live `PHASE<N-1>_SURPRISE_LOG.md` (or, if
  this is the first phase using the convention, from
  `multi-agent/plans/PHASE15_SURPRISE_LOG.md` as the canonical reference).
  Update the title heading to `PHASE <N>` and the Entries section starts empty.

The SPEC and STRATEGY files are produced earlier (during SPEC engineering and
strategist passes per Templates F-engineering and Template I respectively).

---

## Rules for updating CHANGELOG — v2

### CHANGELOG batching rule

**One CHANGELOG entry per CYCLE closeout, not per role-round.** Mid-cycle notes go in
`AUDIT_LOG.md`, not CHANGELOG.

This cuts CHANGELOG entries from ~3 per priority (Role 1 audit + Role 2 implement + Role 1
re-audit) to 1 per cycle. For batched cycles, 1 entry covers several priorities.

### Who writes the CHANGELOG entry

The **closing agent** writes the CHANGELOG entry. This is usually:
- Role 1 at re-audit closeout (normal path)
- Role 2 at skip-reaudit declaration (when orchestrator accepts the skip)
- Role 3 at wrap-up (for wrap-up audit entries)
- Role 1 at post-wrap-up closeout (for post-wrap-up remediation cycles)

### Authorship consolidation at cycle closeout

**When the closing agent writes the CHANGELOG entry, it MUST:**

1. Survey all rounds logged in the relevant cycle section of `AUDIT_LOG.md`.
2. Collect every `**Authors:**` line from those rounds.
3. Deduplicate and union them into one consolidated `**Authors:**` line.
4. Order: user first, then other contributors in order of first contribution to the cycle.
5. Emit the consolidated `**Authors:**` line in the CHANGELOG entry.

**Example.** A cycle has three rounds:
- Round 1 Role 1 audit by `John M. Urban, Claude Code 2.1.104 (claude-opus-4-7)`
- Round 2 Role 2 implement by `John M. Urban, GitHub Copilot (gpt-5-codex)`
- Round 3 Role 1 re-audit by `John M. Urban, Claude Code 2.1.104 (claude-opus-4-7)`

The cycle-closeout CHANGELOG entry `**Authors:**` line is:
```
**Authors:** John M. Urban, Claude Code 2.1.104 (claude-opus-4-7), GitHub Copilot (gpt-5-codex)
```

### Required checkpoint reads before writing CHANGELOG

These reads stay mandatory (quality guardrail against format drift, especially
post-compaction):

1. Read your agent file (CLAUDE.md, AGENTS.md, GEMINI.md, .github/copilot-instructions.md)
   — for authorship format for your agent class
2. Read AGENT_CONVENTIONS.md — for CHANGELOG format, timestamp format, authorship rules

**Compaction awareness (new in v2):** If this session has been compacted since you
last read either file, treat yourself as a cold start and re-read. Never skip these
reads on the assumption "I already read it this session" if compaction has occurred.

---

## Roles

### Role 1 — Auditor / Instruction Writer

Role 1 performs deep audits and writes the repair contract.

Primary responsibilities:

1. Read the target cycle scope from `TARGET_FILE` and the surrounding context needed to
   understand its intent.
2. Audit the live code, tests, scripts, and documentation relevant to that cycle.
3. Determine whether the cycle is closed, partially complete, or still open.
4. If open, append audit findings and exact implementation instructions into the cycle's
   section of `AUDIT_LOG_FILE` (NOT the SPEC).
5. At cycle closeout: update the SPEC's priority status table. Write the cycle's
   CHANGELOG entry using the authorship consolidation rule above.
6. Re-audit Role 2's implementation by reading the actual changed code rather than trusting the
   implementation report.
7. Close the cycle only after verifying that the findings are actually resolved.
8. If Role 3 identifies actionable wrap-up findings, review that wrap-up audit, decide which
   findings become active repair work, and translate them into concrete implementation
   instructions in a new cycle section of `AUDIT_LOG_FILE`.
9. After Role 2 completes any post-wrap-up remediation, perform the final closeout audit.

Role 1 must not:

- trust the implementation report without inspecting the code
- skip `scripts/` or `tests/` or documentation when they are plausibly affected
- silently narrow or broaden scope — scope changes belong in an explicit audit finding for the
  user to approve; never relegate in-scope work to KNOWN_ISSUES without asking the user first
- edit the SPEC body with audit findings; findings go to `AUDIT_LOG_FILE`
- **decide deferrals during initial audit.** Per `multi-agent/AGENT_CONVENTIONS.md § Scope authority §
  Deferral is NOT an ordinary disposition (critical — strict pathology-only rule)`, R1 never proposes
  "defer this to a future cycle" while writing the repair contract. If R1 sees something that looks
  pathological, raise `[PATHOLOGY_CANDIDATE]` mid-flight (real-time chat + audit log) and STOP writing
  the R2 contract for that flagged item until block lifts. R1 produces no R2 instructions for
  pathology-flagged items. The default disposition is "implement what was scoped"; deferral is
  pathology-only and requires the 4-step protocol (raise → independent audit → Principal decision).

Further clarification:
- looking for ways to broaden the scope or improve the phase is allowable, but it must be raised to the user's attention for approval
- it is best to assume the implementation was NOT exhaustive and that you can add value by looking for gaps and pushing back when needed
- it is encouraged to leave praise to the implementer when the implementer finds gaps in the audit or identifies clever ways to solve problems encountered in the code reality.

Expected output from Role 1:

- an audit subsection in `AUDIT_LOG_FILE` under the current cycle
- exact repair instructions with file/function references when possible
- at cycle closeout: a `CHANGELOG.md` entry with consolidated authorship + a SPEC
  priority status-table update
- an explicit close / still-open judgment after each re-audit
- when needed, an auditor triage of Final Overseer findings and a final post-remediation closeout
  judgment


Expected at the end of each audit round when more implementation is needed:
- At the end of your chat message to the user, provide the following prompt to give to the Role 2 Implementer Agent:
```text
Use this prompt for the Implementer Agent:

Read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md and follow it.

You are acting as Role 2 — Implementer / Supplemental Auditor.
Use Template B — Role 2 Implementation.

Target file: path/to/TARGET_FILE
Audit log file: path/to/AUDIT_LOG_FILE
Target cycle: <cycle name>
Current round: implementation

Read the audited findings and implementation instructions in the relevant cycle
section of the audit log file, then implement them as closely as possible. While
working, inspect nearby code for audit misses in the same surface area. If you need
to diverge, document the reason in your implementation report in the same audit log
cycle section. Append your implementation report to AUDIT_LOG_FILE. Do NOT write to
CHANGELOG.md in this round — CHANGELOG batching happens at cycle closeout. Update
HANDOFF.md and TASK.md to reflect the new state. Report the tests you ran.

At closeout of your round, decide whether re-audit is needed or whether skip-reaudit
is safe. See the Role 2 skip-reaudit criteria in the workflow file.
```
- where path/to/TARGET_FILE = `TARGET_FILE` you were working on
- where path/to/AUDIT_LOG_FILE = the sibling audit-log file
- where <cycle name> = `TARGET_CYCLE` you were working in


Expected after closing up a cycle, and there are still cycles to address in this Phase Spec:
- At the end of your chat message to the user, provide the following prompt to give to back to you, the Role 1 Auditor, for prompting the first pass audit on the next cycle:
```text
Feed this prompt back to me, the Role 1 Auditor agent, to move on to the next cycle in this Phase:

Read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template A — Role 1 Initial Audit.

Target file: path/to/TARGET_FILE
Audit log file: path/to/AUDIT_LOG_FILE
Target cycle: <next cycle>
Current round: initial audit

Audit this cycle deeply against the live codebase. Determine whether it is closed or open.
If it is open, append audit findings and exact implementation instructions into the cycle's
section of AUDIT_LOG_FILE (NOT the SPEC), including file references, function references, and
concrete repair notes. Do NOT write to CHANGELOG.md in this round — that happens at cycle
closeout. Read actual code rather than trusting prior reports. Include scripts/ and tests/
when relevant.


Required checkpoint reads (quality guardrail — do not skip, especially post-compaction):
- If you have not read your agent file (CLAUDE.md, AGENTS.md, GEMINI.md, or
  .github/copilot-instructions.md) this session, or if the session has been compacted
  since you last read it, read it now.
- Same for AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md
- ensure that you have completed all that is expected of you from your role and template.
- emit a git commit block per the "Convention for agent-produced git commit blocks" section in this workflow file (mid-cycle form for non-closeout rounds; closeout form when you write the CHANGELOG/DEVLOG entry), placed before your handoff prompt(s).

```
- where path/to/TARGET_FILE = `TARGET_FILE` you were working on
- where path/to/AUDIT_LOG_FILE = the sibling audit-log file
- where <next cycle> = the `TARGET_CYCLE` after the one you just closed.



Expected after all intended cycles are nominally closed when it is time to pass it off to the Role 3 Final Overseer for the Wrap-Up Audit:
- At the end of your chat message to the user, provide the following prompt to give to the Role 3 Final Overseer Agent:
```text
Use this prompt for the Final Overseer Agent:

Read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md and follow it.

You are acting as Role 3 — Final Overseer / Wrap-Up Auditor.
Use Template D — Role 3 Final Wrap-Up Audit.

Target file: path/to/TARGET_FILE
Audit log file: path/to/AUDIT_LOG_FILE
Current round: final wrap-up audit

All intended cycles are nominally closed. Perform a full wrap-up audit of the spec or plan,
the audit log, and the actual codebase. Do not edit code. Report any missed issues,
audit-trail inconsistencies, or adjacent issues that fit the spirit of the target plan.
Write your report in a new "Wrap-up audit" cycle section at the end of AUDIT_LOG_FILE. At
cycle closeout, write a single CHANGELOG entry for the wrap-up audit with consolidated
authorship.


Required checkpoint reads (quality guardrail):
- If you have not read your agent file this session, or if the session has been compacted
  since you last read it, read it now.
- Same for AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md
- ensure that you have completed all that is expected of you from your role and template.
- emit a git commit block per the "Convention for agent-produced git commit blocks" section in this workflow file (mid-cycle form for non-closeout rounds; closeout form when you write the CHANGELOG/DEVLOG entry), placed before your handoff prompt(s).
```
- where path/to/TARGET_FILE = `TARGET_FILE` you were working on
- where path/to/AUDIT_LOG_FILE = the sibling audit-log file


Expected after the triage of the Final Overseer findings into concrete repair work ready for the Role 2 Implementer Agent.
- At the end of your chat message to the user, provide the following prompt to give to the Role 2 Implementer Agent:
```text
Use this prompt for the Implementer Agent:

Read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md and follow it.

You are acting as Role 2 — Implementer / Supplemental Auditor.
Use Template F — Role 2 Post-Wrap-Up Implementation.

Target file: path/to/TARGET_FILE
Audit log file: path/to/AUDIT_LOG_FILE
Target cycle: post-wrap-up remediation
Current round: post-wrap-up implementation

Role 1 has translated actionable wrap-up findings into repair instructions in AUDIT_LOG_FILE.
Implement only that triaged subset, perform the justified validation, append your
implementation report under the post-wrap-up remediation cycle in AUDIT_LOG_FILE, and update
HANDOFF.md plus TASK.md. Do NOT write to CHANGELOG.md until the post-wrap-up cycle closes.


Required checkpoint reads (quality guardrail):
- If you have not read your agent file this session, or if the session has been compacted
  since you last read it, read it now.
- Same for AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md
- ensure that you have completed all that is expected of you from your role and template.
- emit a git commit block per the "Convention for agent-produced git commit blocks" section in this workflow file (mid-cycle form for non-closeout rounds; closeout form when you write the CHANGELOG/DEVLOG entry), placed before your handoff prompt(s).
```
- where path/to/TARGET_FILE = `TARGET_FILE` you were working on
- where path/to/AUDIT_LOG_FILE = the sibling audit-log file

### Role 2 — Implementer / Supplemental Auditor

Role 2 implements the audited instructions and performs in-situ auditing while working.

Primary responsibilities:

1. Read the current audited cycle section in `AUDIT_LOG_FILE` and the related
   priority block(s) in the SPEC.
2. Implement only the audited open items unless code reality forces a documented deviation.
3. While editing, inspect surrounding code carefully enough to catch audit misses in the same neighborhood.
4. If a miss is found:
   - fix it if it is clearly in scope and low-risk, then document it in the implementation
     report
   - otherwise flag it clearly in the implementation report
5. Run the targeted validation justified by the changed surface.
6. Append an implementation report into the same cycle section of `AUDIT_LOG_FILE`.
7. Do **not** write to `CHANGELOG.md` in mid-cycle rounds. CHANGELOG is written once at
   cycle closeout by the closing agent (per the CHANGELOG batching rule).
8. Update `HANDOFF.md` and `TASK.md` to reflect the new state.
9. If the round comes from Role 3 follow-up findings, implement only the actionable subset
   that Role 1 has triaged into concrete instructions in `AUDIT_LOG_FILE`.
10. **At the end of the implementation round, decide whether re-audit is needed or
    whether skip-reaudit is safe.** See "Skip-Reaudit Criteria" below.

Role 2 must not:

- start coding before explicit user approval for the implementation round
- silently diverge from the audit instructions
- assume the audit was exhaustive
- close the cycle without either a verifying re-audit from Role 1 or an
  orchestrator-approved skip-reaudit declaration
- write to CHANGELOG.md in mid-cycle rounds (batching rule)
- **silently defer scoped deliverables.** Per `multi-agent/AGENT_CONVENTIONS.md § Scope authority §
  Deferral is NOT an ordinary disposition (critical — strict pathology-only rule)`, R2 never emits
  sidecar/placeholder/TODO substitutes as a deferral mechanism, never files follow-up
  `[ISSUE:YYYY-MM-DD:N]` entries to track work that R1 specified, and never silently scope-narrows.
  If R2 mid-implementation suspects pathology, STOP work on that deliverable, raise
  `[PATHOLOGY_CANDIDATE]` mid-flight (real-time chat + audit log) with the required fields, and
  wait for independent audit + Principal decision. The cycle's other deliverables continue. The
  default expectation is "implement what was scoped"; the only legitimate non-implementation is
  pathology raised explicitly through the 4-step protocol.

Further clarification:
- it is allowable to diverge from audit instructions when code reality forces a deviation, but it must be reported
- it is best to assume the audit was NOT exhaustive and that you can add value by looking for gaps and pushing back when needed
- your secondary role is a supplemental auditor, so report your audit findings that were not addressed by the auditor, and whether or not you already implemented fixes based on your own findings, for the auditor to incorporate in their re-audit
- it is encouraged to leave praise to the auditor when the instructions left are elegant and/or when the auditor identifies clever ways to solve problems encountered in the code reality
- it is also encouraged to leave praise to the re-auditor when a re-audit finds gaps in the first pass implementation.

Expected output from Role 2:

- the code changes themselves
- an implementation report appended to the cycle section in `AUDIT_LOG_FILE`
- synced `HANDOFF.md` and `TASK.md`
- clear note of tests run and any deviation from the audit instructions
- an explicit decision at round end: "re-audit needed" OR "skip-reaudit recommended"
  (with reasoning, for orchestrator to approve)

#### Skip-Reaudit Criteria (new in v2)

Role 2 may **recommend** skipping re-audit only if **ALL** of the following are true:

1. Every item in Role 1's audit instructions was implemented exactly as specified.
2. Zero divergences from the audit instructions.
3. All justified tests passed green (e.g., `make test` fully green; targeted regressions
   covering the changed surface all pass).
4. No in-situ audit miss was found that required structural or design decisions. (Trivial
   fixes handled inline are OK; decisions are not.)
5. No ambiguity was encountered during implementation.
6. The cycle is mechanical (thin-priority batched cycle or straightforward substantive
   priority without design judgment).

Role 2 **must request re-audit** (never skip) if ANY of these are true:
- Any divergence from instructions, even justified.
- Any test failed at any point, even if later fixed.
- Any audit miss that required judgment to handle.
- The cycle involves design judgment (help-string quality bar, terminology harmonization,
  cross-pipeline framing, Universal promotion, closeout doc sweeps).
- Any uncertainty about whether the implementation fully satisfied the instructions.

**The orchestrator (user) has final approval on any skip-reaudit.** Role 2's declaration
is a recommendation, not a decision. If the orchestrator overrides, re-audit happens.

Expected at the end of each implementation round when re-audit IS needed:
- At the end of your chat message to the user, provide the following prompt to give to the Role 1 Auditor Agent:
```text
Use this prompt for the Auditor Agent:

Read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template C — Role 1 Re-Audit.

Target file: path/to/TARGET_FILE
Audit log file: path/to/AUDIT_LOG_FILE
Target cycle: <cycle name>
Current round: re-audit

Role 2 has finished an implementation round. Re-audit the live code and determine whether
this cycle is now closed or still open. Read the actual changed code rather than trusting
the implementation report. If it is still open, append new findings and exact repair
instructions into the cycle section of AUDIT_LOG_FILE. If it is now closed, update the
SPEC's priority status table and write the cycle-closeout CHANGELOG entry with
consolidated authorship. Include scripts/ and tests/ when relevant.


Required checkpoint reads (quality guardrail):
- If you have not read your agent file this session, or if the session has been compacted
  since you last read it, read it now.
- Same for AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md
- ensure that you have completed all that is expected of you from your role and template.
- emit a git commit block per the "Convention for agent-produced git commit blocks" section in this workflow file (mid-cycle form for non-closeout rounds; closeout form when you write the CHANGELOG/DEVLOG entry), placed before your handoff prompt(s).
```
- where path/to/TARGET_FILE = `TARGET_FILE` you were working on
- where path/to/AUDIT_LOG_FILE = the sibling audit-log file
- where <cycle name> = `TARGET_CYCLE` you were working in


Expected at the end of each implementation round when Role 2 RECOMMENDS SKIP-REAUDIT:
- At the end of your chat message to the user, provide a **two-option prompt** covering
  both paths (accept skip OR force re-audit). The orchestrator (user) picks one.

```text
Skip-reaudit recommended for <cycle name>.

Skip criteria satisfied:
- [list each criterion that is true]

If you ACCEPT the skip and want to move to the next cycle, use this prompt for the Role 1 Auditor Agent:

Read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template H — Role 1 Cycle Closeout After Skip-Reaudit.

Target file: path/to/TARGET_FILE
Audit log file: path/to/AUDIT_LOG_FILE
Closed cycle: <cycle just finished>
Next target cycle: <next cycle>
Current round: cycle closeout (skip-reaudit accepted) + next-cycle initial audit

Role 2 recommended skip-reaudit for the closed cycle and the orchestrator accepted.
Perform these steps in order:
1. Lightweight verification: spot-check 2-3 surfaces that Role 2 claims to have changed
   (not a full re-audit — just sanity check for obvious drift).
2. Update the SPEC's priority status table for the closed cycle.
3. Write the cycle-closeout CHANGELOG entry with consolidated authorship (survey
   AUDIT_LOG_FILE for all Authors lines in the closed cycle's rounds).
4. Update HANDOFF.md and TASK.md.
5. Proceed to initial audit of the next cycle. Use Template A behavior for that part.


Required checkpoint reads (quality guardrail):
- If you have not read your agent file this session, or if the session has been compacted
  since you last read it, read it now.
- Same for AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md
- ensure that you have completed all that is expected of you from your role and template.
- emit a git commit block per the "Convention for agent-produced git commit blocks" section in this workflow file (mid-cycle form for non-closeout rounds; closeout form when you write the CHANGELOG/DEVLOG entry), placed before your handoff prompt(s).

---

If you REJECT the skip and want full re-audit, use this prompt for the Role 1 Auditor Agent:

Read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template C — Role 1 Re-Audit.

Target file: path/to/TARGET_FILE
Audit log file: path/to/AUDIT_LOG_FILE
Target cycle: <cycle just finished>
Current round: re-audit (orchestrator overrode skip-reaudit)

Role 2 finished an implementation round and recommended skip-reaudit, but the orchestrator
requested full re-audit. Re-audit the live code and determine whether the cycle is now
closed or still open. Read actual changed code.


Required checkpoint reads (quality guardrail):
- If you have not read your agent file this session, or if the session has been compacted
  since you last read it, read it now.
- Same for AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md
- ensure that you have completed all that is expected of you from your role and template.
- emit a git commit block per the "Convention for agent-produced git commit blocks" section in this workflow file (mid-cycle form for non-closeout rounds; closeout form when you write the CHANGELOG/DEVLOG entry), placed before your handoff prompt(s).
```
- where path/to/TARGET_FILE = `TARGET_FILE` you were working on
- where path/to/AUDIT_LOG_FILE = the sibling audit-log file
- where <cycle just finished> = the cycle Role 2 just implemented
- where <next cycle> = the cycle queued after this one


Expected after finishing the post-wrap-up implementation round (after implementation of the triage of the Final Overseer Wrap-up audit):
- At the end of your chat message to the user, provide the following prompt to give to the Role 1 Auditor Agent:
```text
Use this prompt for the Auditor Agent:

Read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template G — Role 1 Final Closeout After Wrap-Up Remediation.

Target file: path/to/TARGET_FILE
Audit log file: path/to/AUDIT_LOG_FILE
Target cycle: post-wrap-up remediation
Current round: post-wrap-up closeout

Role 2 has completed the post-wrap-up remediation round. Re-audit the live code, determine
whether the actionable findings from the Final Overseer audit are now resolved, and append
the final closeout judgment to the post-wrap-up remediation cycle in AUDIT_LOG_FILE. If
closed: update SPEC priority status table, write the cycle-closeout CHANGELOG entry with
consolidated authorship, and sync HANDOFF.md and TASK.md if the target file is now fully
closed.


Required checkpoint reads (quality guardrail):
- If you have not read your agent file this session, or if the session has been compacted
  since you last read it, read it now.
- Same for AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md
- ensure that you have completed all that is expected of you from your role and template.
- emit a git commit block per the "Convention for agent-produced git commit blocks" section in this workflow file (mid-cycle form for non-closeout rounds; closeout form when you write the CHANGELOG/DEVLOG entry), placed before your handoff prompt(s).
```

### Role 3 — Final Overseer / Wrap-Up Auditor

Role 3 is optional in 2-role mode and recommended in 3-role mode.

Role 3 performs the final broad audit after all target cycles are nominally closed.

Primary responsibilities:

1. Read the full target spec, the full AUDIT_LOG file, and sample the codebase against
   both.
2. Audit the actual codebase against the claims made by Role 1 and Role 2 in AUDIT_LOG.
3. Look for:
   - missed issues inside already-closed cycles
   - inconsistencies between audits and implementation
   - adjacent issues that fit the spirit of the target phase or plan
4. When the target is a phase spec, active plan, or comparable project-planning file, also
   compare it against the live near-term and speculative backlog surfaces to identify overlap,
   already-parked work, and items that still fit the spirit of the current phase or plan.
   For onionskin, this usually means checking `multi-agent/tracking/KNOWN_ISSUES.md` and `multi-agent/tracking/BRAINSTORM.md`.
5. Write a wrap-up audit cycle section in `AUDIT_LOG_FILE`.
6. At wrap-up cycle closeout (no follow-up actionable work found): update SPEC priority
   status table if applicable, and write the wrap-up audit CHANGELOG entry with
   consolidated authorship.
7. Update `HANDOFF.md` and `TASK.md` only if the wrap-up audit creates concrete next actions.
8. Separate actionable follow-up findings from future-looking observations so Role 1 can
   triage the implementation-worthy items cleanly.

Role 3 must not:

- edit code unless the user explicitly converts the round into an implementation round
- trust either audit reports or implementation reports without checking live code
- write audit content to the SPEC body; audit content goes to `AUDIT_LOG_FILE`
- **accept R2 deferral requests on R2's reasoning alone.** Per `multi-agent/AGENT_CONVENTIONS.md §
  Scope authority § Deferral is NOT an ordinary disposition (critical — strict pathology-only rule)`,
  R3 errs strongly toward REJECTING any deferral request. R3 confirms a deferral only if (a) R2
  explicitly raised `[PATHOLOGY_CANDIDATE]` AND (b) R3 independently verifies pathology through code
  auditing — NOT by accepting R2's reasoning or being persuaded by logic. If R3 finds R2 silently
  deferred without raising the candidate (sidecar-only intermediates; PARTIAL closeout proposals;
  follow-up ISSUE filings as deferral mechanisms), R3 REJECTS the cycle close and instructs R2 to
  either implement OR raise `[PATHOLOGY_CANDIDATE]` properly. R3 does NOT close cycles PARTIAL with
  deferred deliverables; PARTIAL + new ISSUE filing IS silent narrowing. R3 launcher prompts
  authored by the orchestrator MUST NOT offer deferral framings as menu options — default-frame is
  "cycle stays OPEN if any R1-scoped deliverable didn't land."

Expected output from Role 3:

- a wrap-up audit cycle section in `AUDIT_LOG_FILE`
- a `CHANGELOG.md` entry (one, at wrap-up cycle closeout) with consolidated authorship
- a clear statement of whether the target file is fully closed or has remaining findings
- when relevant, a short overlap assessment against `KNOWN_ISSUES.md` and `BRAINSTORM.md`
   identifying which findings are already parked versus newly surfaced
- when relevant, a clearly labeled actionable-follow-up block that can be handed back to Role 1

Expected at the end of the Final Overseer Wrap-Up audit:
- At the end of your chat message to the user, provide the following prompt to give to the Role 1 Auditor Agent:
```text
Use this prompt for the Auditor Agent:

Read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md and follow it.

You are acting as Role 1 — Auditor / Instruction Writer.
Use Template E — Role 1 Post-Wrap-Up Triage.

Target file: path/to/TARGET_FILE
Audit log file: path/to/AUDIT_LOG_FILE
Current round: post-wrap-up triage

Role 3 has finished the wrap-up audit in AUDIT_LOG_FILE and found additional actionable
items. Review the wrap-up audit against the live code, decide which findings should become
active repair work now, and translate them into concrete implementation instructions in a
new post-wrap-up remediation cycle section of AUDIT_LOG_FILE. Do not rewrite the Final
Overseer report. Do NOT write to CHANGELOG.md in this round — CHANGELOG for the
post-wrap-up remediation will be written at its cycle closeout. Sync HANDOFF.md plus
TASK.md to point to the new next action.


Required checkpoint reads (quality guardrail):
- If you have not read your agent file this session, or if the session has been compacted
  since you last read it, read it now.
- Same for AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md
- ensure that you have completed all that is expected of you from your role and template.
- emit a git commit block per the "Convention for agent-produced git commit blocks" section in this workflow file (mid-cycle form for non-closeout rounds; closeout form when you write the CHANGELOG/DEVLOG entry), placed before your handoff prompt(s).
```
- where path/to/TARGET_FILE = `TARGET_FILE` you were working on
- where path/to/AUDIT_LOG_FILE = the sibling audit-log file


---

## Shared Rules — v2

These rules apply to every role.

1. Work one cycle at a time.
2. Inspect live code rather than trusting markdown reports.
3. Include `scripts/` and `tests/` in audits and re-audits whenever they are plausibly affected.
4. Keep the round-by-round audit trail in `AUDIT_LOG_FILE`. Keep the SPEC clean.
5. Record each **cycle closeout** as a single CHANGELOG entry with consolidated authorship.
   Do not write CHANGELOG entries per role-round.
6. Update `HANDOFF.md` and `TASK.md` whenever a round changes the next actionable state.
7. Follow `multi-agent/AGENT_CONVENTIONS.md` for authorship, timestamps, approval rules, and
   scope authority.
8. Do not rewrite history in `AUDIT_LOG_FILE`. Append new rounds; do not erase prior rounds.
9. When code reality differs from the written audit, prefer correctness, then document the
   divergence explicitly in AUDIT_LOG_FILE.
10. The Final Overseer audit is normally a single pass. If it finds actionable work, the loop
    returns to Role 1 -> Role 2 -> Role 1. Role 3 does not usually re-enter unless the user asks.
11. Surface scope questions explicitly. If adjacent work clearly falls within the spirit of the
    current phase, flag it as a broadening opportunity in the audit output — do not silently add
    it, but also do not relegate it to KNOWN_ISSUES without asking. The user is the sole authority
    on all scope decisions. See `multi-agent/AGENT_CONVENTIONS.md` for the full scope authority
    rule.
12. **Compaction awareness (v2)**: If the session has been compacted since you last read a
    required file, treat yourself as a cold start and re-read. Re-reads are cheap insurance
    against format drift; skipping them after compaction risks drift that costs far more to
    fix.
13. **Substantive-cycle rule (v2)**: Do not run a full audit-implement-reaudit loop on thin
    priorities. Bundle thin priorities into batched cycles. See "Cycle Granularity" above.
14. **Skip-reaudit is a Role 2 recommendation, not a decision (v2)**: Only the orchestrator
    (user) can accept a skip-reaudit. All skip criteria must be met or re-audit is required.
15. **Authorship consolidation at cycle closeout (v2)**: The closing agent MUST survey
    AUDIT_LOG_FILE's cycle section and consolidate all `**Authors:**` lines into one
    deduplicated line in the CHANGELOG entry.

---

## Standard Execution Loop — v2

### Step 1 — Initial Audit

Role 1 audits `TARGET_CYCLE`, determines whether it is open or closed, writes findings and exact
instructions into the cycle section of `AUDIT_LOG_FILE` (not the SPEC). Role 1 does NOT write
to CHANGELOG in this round.

### Step 2 — Implementation

After explicit user approval, Role 2 implements the audited open items, runs the justified
validation, appends an implementation report to the cycle section of `AUDIT_LOG_FILE`, and
updates HANDOFF.md + TASK.md. Role 2 does NOT write to CHANGELOG. At round end, Role 2 declares
either "re-audit needed" or "skip-reaudit recommended (pending orchestrator approval)".

### Step 3 — Re-Audit OR Skip-Reaudit Closeout

**If re-audit needed**: Role 1 reads the actual changed code and decides whether the cycle is
now closed or still open. If still open, Role 1 writes new findings and instructions into the
cycle section of AUDIT_LOG_FILE. If closed, Role 1 updates the SPEC priority status table and
writes the cycle-closeout CHANGELOG entry with consolidated authorship.

**If skip-reaudit accepted by orchestrator**: Role 1 performs lightweight spot-check (2-3
surfaces), then closes the cycle: SPEC priority status table update + CHANGELOG entry with
consolidated authorship. Skip-reaudit lightweight closeout is Template H.

### Step 4 — Iterate Until Closed

Repeat Steps 2 and 3 until the cycle is explicitly closed by an auditing role (or by Role 1
at skip-reaudit closeout).

### Step 5 — Cycle Transition

When one cycle is closed, move to the next user-designated cycle and repeat the loop.

### Step 6 — Final Wrap-Up Audit

When all intended cycles are nominally closed, Role 3 performs a full-file wrap-up audit,
writing its report as a wrap-up cycle section in `AUDIT_LOG_FILE`. In 2-role mode, Role 1
performs this step.

At wrap-up cycle closeout (no actionable follow-up): CHANGELOG entry written by Role 3 (or
Role 1 in 2-role mode) with consolidated authorship.

### Step 7 — Post-Wrap-Up Remediation Loop

If the wrap-up audit finds actionable issues:

1. Role 1 reviews the wrap-up audit and decides which findings become active repair work.
2. Role 1 opens a new "post-wrap-up remediation" cycle section in `AUDIT_LOG_FILE` and
   translates the actionable findings into concrete instructions.
3. After explicit user approval, Role 2 implements that post-wrap-up repair set, reports
   the validation in AUDIT_LOG_FILE, declares re-audit or skip-reaudit.
4. Role 1 performs the final closeout (full re-audit or lightweight spot-check depending on
   Role 2 declaration + orchestrator decision). At closeout: single CHANGELOG entry for the
   post-wrap-up remediation cycle.
5. Role 3 usually does not re-audit this second pass unless the user explicitly requests it.

### Step 8 — Phase Archive (separate from cycle structure)

The phase archive — moving `PHASE<N>_SPEC.md`, `PHASE<N>_AUDIT_LOG.md`,
`PHASE<N>_STRATEGY.md`, `PHASE<N>_FEEDBACK.md`, and any chronological backup files
into `multi-agent/plans/archived/` with `YYYYMMDD-` prefix — is **NOT a cycle
deliverable**. It is a separate operation that happens AFTER:

1. The last execution cycle of the phase closes (e.g., Phase 14 Supplemental's
   cycle 14S.5a).
2. Final Overseer's wrap-up audit closes (Step 6).
3. Any post-wrap-up triage / remediation / closeout cycles close (Step 7).

The orchestrator (user) performs the archive as an operation outside the cycle
structure, after confirming all wrap-up work is complete and no further edits to
the AUDIT_LOG are pending.

**Why archive-last matters.** Final Overseer reads SPEC + AUDIT_LOG + STRATEGY at
their live paths and writes its wrap-up findings INTO the AUDIT_LOG. Writing into
an archived file violates the AGENT_CONVENTIONS.md rule "Once archived, a phase
SPEC/PLAN file should be treated as immutable historical provenance." Post-wrap-up
remediation cycles (Templates E/F/G) also write to the live AUDIT_LOG. Archiving
early forces those writes into archived paths, which breaks immutability and
confuses provenance.

**What cycle-scoped closeout work IS in scope** (e.g., a Phase 14 Supplemental
14-S16-style "closeout doc sweep" cycle): reference propagation across agent files /
workflows / live plan files; tracking-directory moves (e.g.,
`multi-agent/BRAINSTORM.md` → `multi-agent/tracking/BRAINSTORM.md`); CLI conventions
codification in AGENT_CONVENTIONS.md. These are doc-sweep-style edits that ship
within a cycle's atomic commit. The actual `git mv` of the
SPEC/AUDIT_LOG/STRATEGY/FEEDBACK files into `archived/` does NOT.

**What the cycle-scoped closeout work MAY include** (transitional accommodations
for the eventual archive): updating agent-file active-phase pointers to flag that
the phase will close soon, and pre-staging any archive-related rationale in
DECISIONS.md. The pointer can name the *current* live SPEC path (still active) and
note the planned post-Overseer archival; the actual rewrite to "archived at
`<date>-<file>`" happens AFTER the archive operation.

**Audit-time check.** Any Role 1 audit whose repair instructions propose archiving
SPEC/AUDIT_LOG/STRATEGY/FEEDBACK files into `archived/` MUST stop and re-check.
The archive is a post-Final-Overseer event; if the audit is for a within-phase
cycle (including the last cycle of the phase), the archive is out of scope.

---

## Required File Updates — v2

Minimum required artifacts for onionskin:

1. `TARGET_FILE` (the SPEC)
   - Priority status table updated at cycle closeout (by the closing agent)
   - Scope-change edits only (user-approved)
   - NOT the place for audit findings, implementation reports, or closeout judgments

2. `AUDIT_LOG_FILE`
   - Role 1 audit findings and exact instructions (every initial audit, every re-audit)
   - Role 2 implementation reports (every implementation round)
   - Role 1 closeout judgments (at cycle closeout)
   - Role 3 wrap-up audit
   - Role 1 post-wrap-up triage + closeout
   - Every round carries a `**Authors:**` line

3. `CHANGELOG.md`
   - **One entry per cycle closeout**, written by the closing agent
   - Consolidated authorship (survey AUDIT_LOG_FILE cycle section for all `**Authors:**`
     lines, deduplicate, user first)
   - Not per role-round

4. `multi-agent/project_context/HANDOFF.md`
   - update at any state change between rounds

5. `multi-agent/project_context/TASK.md`
   - update at any state change between rounds

If the wrap-up audit opens new actionable work, `HANDOFF.md` and `TASK.md` should move back from
"closed" language to the specific next repair action owned by Role 1 or Role 2.

Optional additional artifacts when relevant:

- `ROADMAP.md` if a priority within an already-formalized-SPEC phase closes
  (add a `✓ DONE (v0.x.xx)` or `◑ PARTIAL (v0.x.xx)` marker to the existing
  retrospective entry). Do NOT add ROADMAP entries for phases whose SPEC has
  not yet formalized — see `multi-agent/AGENT_CONVENTIONS.md § ROADMAP.md`
  for the retrospective + light-reading + SPEC-formalization-trigger role.
- `DECISIONS.md` if the work settles a reusable design decision
- other docs/specs if the audited cycle explicitly requires synchronization

---

## Acceptance And Closeout Rules — v2

A target cycle is closed only when all of the following are true:

1. An auditing role has inspected the actual changed code (OR Role 2 skip-reaudit was
   accepted by orchestrator AND Role 1 has done lightweight spot-check at closeout).
2. The findings in the latest audit round are either resolved or explicitly reclassified.
3. The implementation round(s) recorded their validation surface in AUDIT_LOG_FILE.
4. `AUDIT_LOG_FILE`, `CHANGELOG.md`, `HANDOFF.md`, `TASK.md`, and the SPEC priority status
   table are consistent with the latest state.
5. No high-confidence open findings remain in the audited cycle.

The full spec or plan is closed only when all targeted cycles are closed and the wrap-up audit
does not identify remaining blocking findings, or when any actionable wrap-up findings have gone
through the post-wrap-up remediation cycle and been closed by Role 1.

---

## Common Failure Modes

Watch for these repeatedly:

1. Role 1 trusts the implementation report instead of reading the changed code.
2. Role 2 trusts the audit blindly and misses obvious nearby drift.
3. One role audits only core modules and forgets `scripts/` or `tests/`.
4. A role writes audit content into the SPEC instead of `AUDIT_LOG_FILE` (v2 failure mode).
5. A role writes a CHANGELOG entry mid-cycle instead of at cycle closeout (v2 failure mode).
6. The closing agent forgets to consolidate authorship and writes a CHANGELOG entry with
   only its own author line, missing contributors from prior rounds (v2 failure mode).
7. `HANDOFF.md` or `TASK.md` lag behind the actual state.
8. A role silently narrows or broadens scope. The failure is the silence, not the scope change:
   raising a scope expansion as an explicit finding is correct behavior; quietly omitting work or
   relegating it to KNOWN_ISSUES without user approval is not.
9. A role rewrites prior audit/implementation content in AUDIT_LOG_FILE instead of appending a
   new round entry.
10. Role 3 mixes actionable findings and future-phase suggestions without labeling which is which.
11. Role 1 leaves Final Overseer findings only in the wrap-up section instead of opening a
    post-wrap-up remediation cycle in AUDIT_LOG_FILE.
12. Role 3 checks only the live code and spec text, but forgets to compare the target plan
    against `KNOWN_ISSUES.md` and `BRAINSTORM.md` for overlap and in-scope follow-up.
13. Role 2 recommends skip-reaudit when criteria are not all met (v2 failure mode). Role 2
    must be honest about its own uncertainty; better to request re-audit than to push past a
    cycle with a silent drift.
14. Orchestrator accepts a skip-reaudit without Role 1 performing the lightweight spot-check
    at closeout (v2 failure mode).
15. An agent running thin priorities as individual cycles instead of batching them (v2 failure
    mode — violates substantive-cycle rule).
16. After a compaction event, an agent skips the required checkpoint reads on the assumption
    "I already read it this session" (v2 failure mode — compaction voids that assumption).
17. **Authorship/Effort attribution drift — agents fabricate plausible-looking reasoning/effort
    values in authorship lines regardless of the actual setting** (recurring across all agents:
    Copilot, Codex, Claude, Gemini). Surfaced explicitly Phase 15 cycle 15.6a-S1 R2 round
    (2026-05-03): Copilot wrote `Reasoning: Extra High` when the user-selected level was High.
    The fix has two parts: (a) when drafting a launcher prompt for an agent, name the
    reasoning/effort level explicitly in the prompt body so the agent has nothing to fabricate;
    (b) when the launcher does not name the level, agent must ask the user to confirm as the
    FIRST IMMEDIATE ACTION of the round, BEFORE any substantive work — catch the user at the
    keyboard before they walk away. Authoritative source for the toggle value is the user's
    explicit confirmation in the current chat; STRATEGY-row primary/alt assignments are
    suggestive only; agent personal-memory entries are stale snapshots and NOT authoritative.
    See `multi-agent/AGENT_CONVENTIONS.md § Authorship conventions § Per-agent format § Rules`
    for the full rule.
18. **Deferral as ordinary disposition — agents treat "close PARTIAL + file follow-up `[ISSUE:YYYY-MM-DD:N]`"
    as a default-available menu option for cycle closeout** (recurring across all agents).
    Surfaced explicitly Phase 15 cycle 15.7a R3 round (2026-05-04): R3 proposed framing-(a)+(c)-hybrid
    closeout with SPEC15.15 d3/d4/d6/d8 deferred to a new `[ISSUE:2026-05-04:1]`; Principal-orchestrator
    overrode 2026-05-04 because the deferral pattern is silent narrowing dressed up as orchestrator
    process. The same pattern surfaced earlier at cycle 15.6a-S1 (R2's "all-levels-mean" trajectory
    clustering — Principal mental-simulation pushback forced redesign in-cycle). The fix is the strict
    pathology-only rule per `multi-agent/AGENT_CONVENTIONS.md § Scope authority § Deferral is NOT an
    ordinary disposition (critical — strict pathology-only rule)`. Summary:
    - Default expectation for ALL agents (R1/R2/R3/orchestrator): implement what was scoped.
    - Sole carve-out is PATHOLOGY (impossible OR genuinely harmful implementation; concrete examples
      in the AGENT_CONVENTIONS rule). "Big" / "complex" / "out of scope estimate" / "could be cleaner
      later" / "low priority for the phase" are NOT pathology.
    - 4-step protocol when pathology suspected: (1) any agent raises `[PATHOLOGY_CANDIDATE]`
      mid-flight (real-time chat + audit log) with required fields; (2) block affected deliverable
      immediately; (3) independent audit by a different role produces `[PATHOLOGY_AUDIT_VERIFIED]` or
      `[PATHOLOGY_AUDIT_REJECTED]` (independent code-grounded verification, NOT acceptance of the
      raising agent's reasoning); (4) Principal decides — `[PATHOLOGY_REJECTED:IMPLEMENT]` /
      `[PATHOLOGY_CONFIRMED_DEFER:<landing-zone>]` / `[PATHOLOGY_CONFIRMED_RESTRUCTURE]`.
    - Cross-agent skepticism: all agents err toward REJECTING other agents' deferral plans;
      independent investigation required; default toward rejection.
    - Per-role flavor: R1 never decides deferrals during initial audit (raises `[PATHOLOGY_CANDIDATE]`
      and stops writing R2 contract for the flagged item); R2 never silently emits
      sidecar/placeholder/TODO substitutes or files follow-up ISSUEs as deferral mechanisms; R3 errs
      strongly toward rejecting deferrals — confirms only on independent code-grounded pathology
      verification, never on R2's reasoning alone.
    - Distinct concept: `[ALTERNATIVE_APPROACH_PROPOSAL]` is a SPEC amendment request mid-cycle
      (markedly better runtime/memory/parallelization/structural cleanliness — concrete reasoning),
      NOT a deferral. Different protocol (no independent audit needed; Principal decides directly).
      See AGENT_CONVENTIONS § Alternative-approach proposals — distinct from pathology.

---

## Prompt Templates

Use these as compact launch prompts.

**Convention for agent-produced handoff prompts.** The `Below N and <N> = <phase
number...>` header and the `PHASE<N>_*.md` path forms in these templates exist
SOLELY for the user's convenience when cold-pasting a launcher from the
orchestrator file — they let the user set N once at the top and have every
downstream path resolve.

When an agent emits a handoff prompt for the next role (per Templates B / C /
H / I), the agent already knows the phase number, so:

1. **Omit the `Below N and <N> = ...` line entirely.** The concrete paths
   below self-identify the phase; the line is vestigial in a hard-coded
   handoff.
2. **Hard-code `N` throughout the rest of the prompt.** Substitute every
   `PHASE<N>_*.md` path with the concrete filename (e.g.,
   `PHASE15_SPEC.md`), and use concrete supplemental-phase filenames where
   applicable (e.g., `PHASE14_SUPPLEMENTAL-SPEC.md`). No uninstantiated
   `<N>` placeholders.
3. **Include the two standard guardrail blocks verbatim.** Agent-emitted
   handoffs MUST carry the "Required checkpoint reads" block (agent file +
   AGENT_CONVENTIONS.md re-read guardrail) and the "Before giving your
   wrap-up summary comments to the user" block (workflow-file re-read + role
   completeness check). These are what make cold-paste launchers and
   agent-emitted handoffs identical in effect. Copy them from the relevant
   template (A/B/C/D/E/F/G/H) verbatim.
4. **Precede the launcher code block with an Orchestrator advisory block.**
   Each agent-emitted handoff must be introduced by a short, structured
   advisory naming the next assignee, role, template, and cycle — pulled
   from `PHASE<N>_STRATEGY.md`. This saves the orchestrator from looking up
   the strategy file each time. The advisory sits OUTSIDE the ``` ```text
   code block so the code block stays clean for paste. Required format:

   ```markdown
   **Orchestrator advisory (who runs this next):**
   - **Cycle:** <index + name> — <one-line priority summary>
   - **Role & template:** <Role 1|2|3> <round type> → use Template <A|B|C|D|E|F|G|H>
   - **Assigned agent (per PHASE<N>_STRATEGY.md):**
     - Primary: <primary agent + effort/reasoning/thinking level from the relevant role column>
     - Alt: <alt agent(s) + level from the relevant role column>
     (Copy verbatim from STRATEGY.md — the level is bundled with the agent name in that file.)
   - **Notes:** <execution-relevant notes from STRATEGY Notes column — e.g.,
     "different session from R2", "skip-reaudit eligible", "R3 defers to
     cycle X R3 (index Y)", "S27+S28 paired per SPEC directive" — or
     "none" if there are no special notes>

   Paste the following into the next agent's fresh chat:
   ```

   Mapping from STRATEGY.md role columns to workflow role + template:

   | STRATEGY column | Next workflow role + template |
   |---|---|
   | R1 (audit)      | Role 1 initial audit, Template A |
   | R2 (implement)  | Role 2 implementation, Template B |
   | R3 (closeout)   | Role 1 re-audit, Template C (or Template H if skip-reaudit accepted) |

   The Final Overseer pass (Template D) uses the "Final Overseer" row of
   STRATEGY.md, not a per-cycle role column.

   When an agent emits BOTH a re-audit handoff AND a skip-reaudit-accepted
   handoff (Role 2 implementation closeout), each gets its own advisory.
   The advisory for the skip-reaudit-accepted variant uses Template H; the
   re-audit variant uses Template C. Both draw the assignee from the
   cycle's STRATEGY R3 column.

5. **Output each launcher prompt in a `\`\`\`text` fenced code block.** Never emit the
   prompt body as inline chat prose. VS Code and similar markdown environments render
   fenced code blocks with a copy button that preserves indentation and structure on
   paste. The Orchestrator advisory (rule 4) sits OUTSIDE the fence so the paste
   target is clean. This requirement mirrors the general "Copy-paste output formatting"
   rule in `AGENT_CONVENTIONS.md`.

**Convention for agent-produced git commit blocks.** Every agent role, when wrapping
up its session, MUST emit a `git` command block in addition to its handoff prompt(s).
The block sits between the role's wrap-up summary and the Orchestrator advisory +
handoff prompt. It tells the user exactly which commands to run to commit the role's
work. Applies to all role-templates (A, B, C, D, E, F, G, H, I).

The agent does NOT run any of these commands itself — git control belongs to the user.
The agent only outputs the commands.

**The block MUST be a fenced code block (triple backticks), not inline prose or inline
code.** VS Code renders fenced blocks with a copy button; inline-formatted commands lose
that. Never write git commands as plain sentences or single-backtick inline code.

There are two forms depending on whether the role writes a CHANGELOG/DEVLOG entry:

**A. Mid-cycle commit** (most rounds — R1 audit, R2 implementation when re-audit is
declared needed, etc.). No CHANGELOG/DEVLOG entry is written this round; the commit
message is a one-line summary of the role's work:

```
git add <explicit list of files; never `git add .` or `-A`>
git commit -m "Phase <N>, cycle <name> (index <i>), Role <r> <round type> (Template <X>): <one-line summary>. | Authors: <consolidated authorship including toggle, per AGENT_CONVENTIONS authorship section>"
```

**B. Closeout commit** (the role that writes the CHANGELOG or DEVLOG entry — typically
Role 1 at re-audit closeout, Role 2 at skip-reaudit acceptance, Role 3 at wrap-up audit
closeout, etc., per the "Who writes the CHANGELOG entry" rules above).

The closing agent (1) writes the CHANGELOG/DEVLOG entry into the appropriate file,
(2) writes a verbatim copy of that entry to a top-level scratch file named
`changelog-entry-<version>.txt` (with the actual version number; the file is
gitignored via `changelog-entry-*.txt` so it cannot be accidentally committed),
and (3) emits THREE commands for the user:

```
git add <explicit list of files; include CHANGELOG.md or DEVLOG.md and any other files modified>
git commit --file=changelog-entry-<version>.txt
rm changelog-entry-<version>.txt
```

Why `--file=` and not `-m`: the closeout commit fossilizes the CHANGELOG/DEVLOG entry
verbatim in git history, so `git log` becomes directly searchable for the full entry
text and the original wording is preserved regardless of whether the
CHANGELOG/DEVLOG file is ever later reformatted, migrated, or split. The
version-tagged filename eliminates "did I pick the wrong txt file?" risk.

`<version>` is the full version string of the entry being committed: `v0.X.YY` for a
CHANGELOG entry (e.g., `v0.14.70`) or `v0.X.YY.Z` for a DEVLOG entry under the new
split-stream scheme (e.g., `v0.14.70.1`). See `AGENT_CONVENTIONS.md § CHANGELOG.md
and DEVLOG.md` for the version-format rules. The gitignore pattern
`changelog-entry-*.txt` covers both forms.

**Appending to an already-committed entry.** If the user requests appending content
to an existing CHANGELOG/DEVLOG entry whose original commit has already happened,
edit the entry in-place to add the appended subsection, then create a NEW scratch
file `changelog-entry-<version>-followup<N>.txt` (suffix `-followup1`, `-followup2`
for subsequent follow-ups) containing just the appended subsection content. Use the
new follow-up file in the three-command closeout block. If the original commit has
NOT yet happened, edit the entry in-place and rewrite the existing
`changelog-entry-<version>.txt` with the full updated entry — the user's existing
commands still work. The agent picks the scenario by checking `git log --oneline`
for the version commit, or by asking the user.

Rules common to both forms:
- Output the block as a fenced code block (triple backticks) — never inline prose or
  inline code. See `AGENT_CONVENTIONS.md` § "Copy-paste output formatting".
- List explicit files in `git add`. Never use `git add .` or `git add -A`.
- Include the full authorship line with runtime toggle values (Effort / Reasoning /
  Thinking) per the AGENT_CONVENTIONS authorship rules.
- The commit block goes BEFORE the Orchestrator advisory + handoff prompt in the
  agent's wrap-up output, so the user runs git commands first, then pastes the
  handoff into the next chat.

**Git-discipline principles (apply to every block emitted under this convention).**

1. **No interactive git in copy-paste blocks — ever.** Every command in the block
   must run cleanly in a non-interactive shell (the user often runs the block via
   paste, or as a `bash -c` script chunk). Forbidden: `git add -p`, `git add -i`,
   `git rebase -i`, `git commit` without `-m`/`--file=` (which opens `$EDITOR`),
   `git mergetool`, any `--edit` flag. Use explicit per-file `git add <path>`
   commands. If a hunk-level split is genuinely needed, instruct the user
   separately in prose ("after the block runs, please review and stage the
   remaining hunks of `<file>` interactively"); never embed the interactive step
   inside the fenced block. Reason: pasted interactive commands either hang
   silently waiting for input or skip files entirely; both have caused
   recoverable-but-painful incidents (cycle 14S.5a Codex `git add -p` skipped
   four agent files + TASK.md).

2. **One concept per commit.** Each commit block represents a single coherent
   change: one cycle round, one role's output, or one named operation (e.g., a
   phase archive). Do not bundle "implementation + closeout doc sweep + phase
   archive" into one commit; do not bundle two roles' output. If a round produces
   work in two distinct conceptual buckets (e.g., tracking-dir reorg AND a
   convention codification), the agent emitting the block MAY split into two
   sequential commits within the same fenced block — but each commit message must
   stand alone and the file lists must not overlap. The reviewer (next-role audit,
   wrap-up overseer, or the user via `git log`) should be able to revert any one
   commit without untangling unrelated work.

3. **Implementer-splits-stages pattern for multi-step implementations.** When a
   Role 2 (or Role 3 wrap-up implementer) executes instructions that decompose
   into ordered stages (e.g., "Stage 1: move tracking files; Stage 2: propagate
   references; Stage 3: add new convention text"), the implementer SHOULD emit
   one commit block per stage rather than a single end-of-round mega-block. Each
   stage commit gets its own one-line message describing just that stage's
   concept; the implementation report in the AUDIT_LOG enumerates all stages.
   This makes mid-implementation surgical reverts cheap (cycle 14S.5a needed
   exactly this — the archive stage was reverted while keeping the prior stages
   intact). The implementer does NOT need to invent stages where the work is
   genuinely atomic; this rule applies when the Role 1 instructions explicitly
   number stages, when the work spans clearly distinct subsystems, or when the
   implementer's own judgment says the round is decomposable. When in doubt, ask
   the orchestrator before commit-time, not after.

### Template A — Role 1 Initial Audit

```text
Below N and <N> = <phase number to be filled in by user>

You are acting as Role 1 — Auditor / Instruction Writer from
multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md.

Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md
Surprise log file: multi-agent/plans/PHASE<N>_SURPRISE_LOG.md
Target cycle: <cycle name> or <cycle index>

Audit this cycle deeply against the live codebase. Determine whether it is closed or open.
If it is open, append audit findings and exact implementation instructions into the cycle
section of the audit log file (NOT the SPEC), including file references, function
references, and concrete repair notes. Do NOT write to CHANGELOG.md in this round — that
is done at cycle closeout. Read actual code rather than trusting prior reports. Include
scripts/ and tests/ when relevant.

**KNOWN_ISSUES carry-over scan (do not skip):** Before producing audit findings, scan
multi-agent/tracking/KNOWN_ISSUES.md for entries whose "Likely landing zone" tags this
cycle, this cycle's SPEC IDs, or any of this cycle's deliverables. Issues so tagged are
explicit instructions from prior cycles' closeouts that this audit should incorporate
into the cycle's repair contract. For each, treat as a mandatory carry-over requiring
explicit response in the audit log: scope-fix into this cycle / defer-with-rationale /
escalate to dedicated cycle (e.g., a mid-phase amendment). NO silent deferral per the
scope-authority rule.

**SURPRISE_LOG carry-over scan (do not skip):** Scan the surprise log file for
`Active` entries whose "Likely landing zone" tags this cycle, this cycle's SPEC IDs,
or any of this cycle's deliverables. Active surprises so tagged are open observations
from prior rounds that this audit should triage. Triage outcomes per entry — pick ONE,
write the disposition into the audit log AND update the SURPRISE entry's status:
  (a) **Integrate into the cycle's repair contract** — fold the surprise into the audit
      findings as a concrete repair instruction; flip the SURPRISE status to
      `In-flight (cycle <this cycle> stage <X>)` citing the stage that addresses it.
      Re-audit at closeout will flip to Resolved.
  (b) **Elevate / promote** — if the surprise is real + actionable but doesn't fit this
      cycle's scope, file a `[ISSUE:YYYY-MM-DD:N]` in tracking/KNOWN_ISSUES.md or a
      BRAIN entry in tracking/BRAINSTORM.md (whichever fits) and flip the SURPRISE
      status to `Promoted to <destination + ID>`.
  (c) **Dismiss with reasoning** — if the surprise is no longer real (code evolved, or
      reassessment changes the read), flip the SURPRISE status to `Logged-only` and
      add a one-line postscript explaining why.
  (d) **Defer with explicit landing zone** — if the surprise is real but neither this
      cycle nor a near-term home is the right place, leave status `Active` and add a
      one-line postscript naming the specific future cycle / SPEC ID / phase-closeout
      milestone where it should be picked up. Active-without-postscript is the smell;
      Active-with-explicit-deferral is the discipline.

**SURPRISE_LOG (optional new entries):** If you notice observations during this audit
that did not make it into the formal findings — adjacent drift, smells, cross-cycle
patterns, or fresh-eyes opinions on user intent — append SURPRISE entries to the
surprise log file per the conventions in its preamble. Quality > coverage: do not write
filler. The file rewards concreteness and uncertainty surfacing, not entry count. New
entries default to status `Active`; cite specific file/line references; assign IDs of
the form `[SURPRISE:<N>:<cycle>:1:A:<X>]` with X starting at 1 and incrementing for
each entry you file in this round.

**Archive-timing check (do not skip).** If your audit's repair instructions propose
`git mv` of `PHASE<N>_SPEC.md`, `PHASE<N>_AUDIT_LOG.md`, `PHASE<N>_STRATEGY.md`, or
`PHASE<N>_FEEDBACK.md` files into `multi-agent/plans/archived/`, STOP and re-check
§ Standard Execution Loop / Step 8 — Phase Archive. The phase archive is a
post-Final-Overseer event performed by the orchestrator outside the cycle structure,
not a Role 2 deliverable inside any execution cycle (including the last cycle of
the phase). Cycle-scoped closeout work (e.g., a 14-S16-style "closeout doc sweep"
cycle) covers reference propagation, tracking-directory moves, and convention
codification — but never the SPEC/AUDIT_LOG/STRATEGY/FEEDBACK file moves
themselves.


Required checkpoint reads (quality guardrail — do not skip, especially post-compaction):
- If you have not read your agent file (CLAUDE.md, AGENTS.md, GEMINI.md, or
  .github/copilot-instructions.md) this session, or if the session has been compacted
  since you last read it, read it now.
- Same for AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md
- ensure that you have completed all that is expected of you from your role and template.
- emit a git commit block per the "Convention for agent-produced git commit blocks" section in this workflow file (mid-cycle form for non-closeout rounds; closeout form when you write the CHANGELOG/DEVLOG entry), placed before your handoff prompt(s).
```

### Template B — Role 2 Implementation

```text
Below N and <N> = <phase number to be filled in by user>

You are acting as Role 2 — Implementer / Supplemental Auditor from
multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md.

Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md
Surprise log file: multi-agent/plans/PHASE<N>_SURPRISE_LOG.md
Target cycle: <cycle name> or <cycle index>

Read the audited findings and implementation instructions in the cycle section of the
audit log file, then implement them as closely as possible. 
Per the Role 2 description: assume the audit may not be exhaustive and that it may contain errors; document any in-situ catches; flag observations or push back when the audit or instructions have erred, drifted from reality, got something wrong, or where auditor judgment is otherwise needed; praise the auditor's clever instruction-writing where warranted.
While working, inspect nearby
code for audit misses in the same surface area. If you need to diverge, document the
reason in your implementation report under the same cycle section. Append your report to
the audit log file. 
Update HANDOFF.md and TASK.md. Report the tests you ran.

**CHANGELOG rule for this round:** Do NOT write to CHANGELOG.md unless you are recommending
skip-reaudit. If you are recommending skip-reaudit, write the cycle's CHANGELOG/DEVLOG
entry now — you are the de facto closing agent if the skip is accepted. If re-audit
proceeds instead, Role 3 (or Role 1 at re-audit closeout) will update your entry at
closeout. If re-audit is declared needed, do NOT write a CHANGELOG entry; Role 3 writes
it at closeout.

**SURPRISE_LOG status updates (do as you go):** When you pick up a SURPRISE entry as
in-cycle work — i.e., the audit's repair instructions reference the surprise OR you
discover during implementation that a Stage you are working on addresses one — flip the
entry's status to `In-flight (cycle <this cycle> stage <X>)` at pickup time and cite
the stage in the status text. The re-auditor will flip to Resolved at closeout if
verification succeeds; leaving it Active during your round implies you didn't pick it up.

**SURPRISE_LOG (optional new entries):** Filing is opt-in, not required. If you notice
observations during implementation that did not make it into your formal report —
adjacent drift, smells, cross-cycle patterns, in-situ catches that don't justify a
finding-level pushback — append SURPRISE entries to the surprise log file per the
conventions in its preamble. Quality > coverage. Assign IDs of the form
`[SURPRISE:<N>:<cycle>:2:B:<X>]` with X starting at 1 and incrementing for each entry
you file in this round.

At round end, declare either "re-audit needed" or "skip-reaudit recommended" per the
Skip-Reaudit Criteria in the workflow file. Then emit handoff prompts as follows:

- **If skip-reaudit is recommended:** provide both handoff prompts (re-audit variant and
  skip-reaudit-accepted variant) so the orchestrator can pick.
- **If re-audit is declared needed:** provide only the re-audit handoff prompt (Template C).
  Do not emit the skip-reaudit variant — the orchestrator may override to skip, but that
  decision belongs to the orchestrator, not the implementer.

Handoff prompts follow the four "Convention for agent-produced handoff prompts" rules at
the top of the Prompt Templates section: **omit the `Below N and <N> = ...` line**,
hard-code every `PHASE<N>_*.md` path with the concrete filename (including
supplemental-phase filenames where applicable), include the two guardrail blocks verbatim,
**and precede each launcher code block with an Orchestrator advisory** naming the next
assignee from `PHASE<N>_STRATEGY.md`. Each handoff gets its own advisory — the
skip-reaudit variant maps to Template H and the re-audit variant maps to Template C; both
draw the assignee from the cycle's STRATEGY R3 column.


Required checkpoint reads (quality guardrail):
- If you have not read your agent file this session, or if the session has been compacted
  since you last read it, read it now.
- Same for AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md
- ensure that you have completed all that is expected of you from your role and template.
- emit a git commit block per the "Convention for agent-produced git commit blocks" section in this workflow file (mid-cycle form for non-closeout rounds; closeout form when you write the CHANGELOG/DEVLOG entry), placed before your handoff prompt(s).
```

### Template C — Role 1 Re-Audit

```text
Below N and <N> = <phase number to be filled in by user>

You are acting as Role 1 — Auditor / Instruction Writer from
multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md.

Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md
Surprise log file: multi-agent/plans/PHASE<N>_SURPRISE_LOG.md
Target cycle: <cycle name> or <cycle index>

Role 2 has finished an implementation round. Re-audit the live code and determine whether
this cycle is now closed or still open. Read the actual changed code rather than trusting
the implementation report. 
Per the Role 1 description: Leave praise in the cycle section of the audit log file to the implementer when the implementer finds gaps in the audit or identifies clever ways to solve problems encountered in the code reality; push back against deviations from instructions when appropriate: push back when the implementer's deviations, fixes, or observations don't hold up under re-audit.
If still open, append new findings and exact repair instructions
to the cycle section of the audit log file. 
If closed: update the SPEC's priority status
table and write the cycle-closeout CHANGELOG entry with consolidated authorship (survey
the audit log file's cycle section for all `**Authors:**` lines, deduplicate, user first).
Include scripts/ and tests/ when relevant. If closed and another cycle is queued in
the strategy file, produce a handoff prompt for the next cycle's Role 1 initial audit
in Template A format, subject to the four "Convention for agent-produced handoff
prompts" rules at the top of the Prompt Templates section: omit the `Below N` line,
hard-code concrete paths, include the two guardrail blocks verbatim, and precede the
launcher with an Orchestrator advisory naming the next cycle's R1 assignee from
`PHASE<N>_STRATEGY.md`. Set `Target cycle:` to that cycle's name and index from the
strategy file.

**SURPRISE_LOG closeout review gate (do not skip):** Before declaring closeout, scan
the surprise log file for two populations of entries that this cycle's re-audit must
disposition:

  (1) Entries tagged with this cycle as the AUTHORING site —
      `[SURPRISE:<N>:<this cycle>:*:*:*]` — i.e., entries filed by the R1, R2, or
      this R3 round during this cycle.
  (2) Entries with status `Active` or `In-flight` whose "Likely landing zone" tags
      this cycle, this cycle's SPEC IDs, or any of this cycle's deliverables —
      regardless of which cycle authored them. (R1's Template A scan should have
      triaged these at audit time; closeout verifies that R2 actually addressed
      whatever R1 flipped to In-flight.)

For each entry, decide ONE disposition and update the entry's `**Status:**` line
in-place:

  - **Resolved (vX.Y.ZZ)** — verified addressed by this cycle's work, where
    "verified" means the live code / docs / tests reflect the resolution. Cite
    the closeout version. This is the only state that requires hard verification.
  - **In-flight (cycle X.Y stage Z | [ISSUE:...] | DECISIONS [date])** — addressed
    elsewhere; flip status with destination reference. Common case: the surprise
    was promoted to KNOWN_ISSUES or BRAINSTORM mid-cycle and now lives there.
  - **Promoted to <destination + ID>** — the surprise's substantive home is now
    a tracking-tier file; create the destination entry if R1/R2 didn't already.
  - **Active (with postscript)** — observation still stands AND no in-cycle action
    addressed it. Add a one-line postscript explaining why no action this cycle
    AND naming a specific future cycle / SPEC ID / phase-closeout milestone where
    it should land. Active-without-postscript is a smell at re-audit time —
    either explicit deferral with landing zone, or one of the other dispositions.
  - **Logged-only** — confirmed not actionable on reassessment.

If the cycle is NOT closing (still open with new findings), still triage population
(1) entries — the implementer's pickups should be reflected in the In-flight statuses
even when the cycle continues. Population (2) entries can roll forward to the next
re-audit if R1's audit flagged them but R2 didn't get to them yet.

**Archive-timing check (do not skip).** If your re-audit finds that Role 2 has
performed `git mv` of `PHASE<N>_SPEC.md`, `PHASE<N>_AUDIT_LOG.md`,
`PHASE<N>_STRATEGY.md`, or `PHASE<N>_FEEDBACK.md` files into
`multi-agent/plans/archived/`, STOP and re-check § Standard Execution Loop /
Step 8 — Phase Archive. The phase archive is a post-Final-Overseer event
performed by the orchestrator outside the cycle structure. If the cycle being
re-audited is a regular execution cycle (including the last cycle of the
phase), the archive is out of scope and must be reverted. The cycle still
closes once the in-scope work (reference propagation, tracking-dir moves,
convention codification) is verified clean.


Required checkpoint reads (quality guardrail):
- If you have not read your agent file this session, or if the session has been compacted
  since you last read it, read it now.
- Same for AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md
- ensure that you have completed all that is expected of you from your role and template.
- emit a git commit block per the "Convention for agent-produced git commit blocks" section in this workflow file (mid-cycle form for non-closeout rounds; closeout form when you write the CHANGELOG/DEVLOG entry), placed before your handoff prompt(s).
```

### Template D — Role 3 Final Wrap-Up Audit

```text
Below N and <N> = <phase number to be filled in by user>

You are acting as Role 3 — Final Overseer / Wrap-Up Auditor from
multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md.

Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md
Surprise log file: multi-agent/plans/PHASE<N>_SURPRISE_LOG.md

All intended cycles are nominally closed. Perform a full wrap-up audit of the spec or
plan, the audit log, and the actual codebase. Do not edit code. Report any missed issues,
audit-trail inconsistencies, or adjacent issues that fit the spirit of the target plan.
When relevant, also compare the target spec or plan against `multi-agent/tracking/KNOWN_ISSUES.md` and `multi-agent/tracking/BRAINSTORM.md` to
report overlap, already-parked items, and any findings that still belong in the spirit of
the current phase or project.

**SURPRISE_LOG end-of-phase sweep (do not skip):** scan the surprise log file for any
entries still marked `Active` or `In-flight` at phase wrap-up time. For each:
  - **Resolved (vX.Y.ZZ)** — verified addressed by a prior cycle's work but the per-cycle
    re-auditor missed flipping the status; flip it now and cite the closing version.
  - **Promoted to <destination + ID>** — file the substantive home in KNOWN_ISSUES /
    BRAINSTORM as a phase-spanning carry-over to the next phase, then flip status.
  - **Active (with phase-spanning postscript)** — observation still stands and is
    intentionally being carried forward to a future phase. Add a one-line postscript
    naming the destination phase / SOUP file / future cycle. This is the only
    acceptable Active state at phase close.
  - **Logged-only** — confirmed not actionable on full-phase reassessment.

Surprises that remain Active without a phase-spanning postscript at wrap-up are
themselves a wrap-up finding — surface them as such in your wrap-up report.

**Cross-cycle drift check on `multi-agent/plans/next/`:** scan SOUP files
(`*_SOUP.md`) in `multi-agent/plans/next/` for any **self-referential**
phase-number mentions in their bodies (filenames excluded). The forbidden
form is the SOUP file naming its own past, current, or future phase
identity (e.g., "this is the Phase X plan", "Phase X follow-up", "(was
PHASE15_BRAINSTORM.md)"). Cross-references to OTHER, actually-closed
phases ARE allowed as historical anchors and should NOT be flagged (e.g.,
"see Phase 14 Supplemental cycle 14S.1a v0.14.68"). Use
`grep -nE "Phase [0-9]" multi-agent/plans/next/*_SOUP.md` as a starting
point, then triage hits to separate self-refs from cross-refs. Flag only
self-refs in the wrap-up audit report. `PHASES.txt` is excluded
entirely — it's the orchestrator's free-form scratchpad and may mention
phase numbers freely. See `multi-agent/AGENT_CONVENTIONS.md § Future-phase
planning surfaces` for the canonical self-reference vs. cross-reference
distinction.

**ROADMAP update at wrap-up closeout:** if priorities in this phase closed
during cycles covered by this wrap-up, ensure the corresponding
`✓ DONE (v0.x.xx)` markers landed in `ROADMAP.md`'s entry for this phase.
ROADMAP entries for closed priorities accumulate retrospectively as
priorities close; do NOT add forward-looking ROADMAP content. See
`AGENT_CONVENTIONS.md § ROADMAP.md` for the retrospective + light-reading
role.

Write your report as a new "Wrap-up audit" cycle section in
the audit log file. At cycle closeout (if no actionable follow-up is needed), write one
CHANGELOG entry for the wrap-up audit with consolidated authorship.

**Phase-final X-bump responsibility (when this is the last CHANGELOG entry of
the phase).** Per `AGENT_CONVENTIONS.md § CHANGELOG.md and DEVLOG.md`, the
phase final-closeout CHANGELOG entry bumps `X` to the just-completed phase
number and resets `YY` to `00`. If your wrap-up audit produces a CHANGELOG
entry AND no Template F/G remediation cycle will follow (i.e., this is
genuinely the phase's last CHANGELOG entry), version your entry as
`v0.<N>.00` where `<N>` = the phase number you just wrapped up. If
remediation IS needed (Template F → Template G follows), do NOT bump `X`
here — Template G's closeout entry is the last-of-phase and owns the
X-bump. When uncertain whether remediation will follow, default to NOT
bumping (it's safer to under-bump and let G handle it than to double-bump).

**SURPRISE_LOG (optional new entries):** Filing is opt-in. As the final overseer with
the broadest cross-cycle perspective, you are well-positioned to surface meta-observations
that no single cycle's auditor or implementer was sized to see. If you notice cross-cycle
patterns, structural drifts, or fresh-eyes opinions on user intent that did not justify a
formal wrap-up finding, append SURPRISE entries with IDs of the form
`[SURPRISE:<N>:wrap-up:3:D:<X>]` (use `wrap-up` as the cycle name).


Required checkpoint reads (quality guardrail):
- If you have not read your agent file this session, or if the session has been compacted
  since you last read it, read it now.
- Same for AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md
- ensure that you have completed all that is expected of you from your role and template.
- emit a git commit block per the "Convention for agent-produced git commit blocks" section in this workflow file (mid-cycle form for non-closeout rounds; closeout form when you write the CHANGELOG/DEVLOG entry), placed before your handoff prompt(s).
```

### Template E — Role 1 Post-Wrap-Up Triage

```text
Below N and <N> = <phase number to be filled in by user>

You are acting as Role 1 — Auditor / Instruction Writer from
multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md.

Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md

Role 3 has finished the wrap-up audit in the audit log file and found additional
actionable items. Review the wrap-up audit against the live code, decide which findings
should become active repair work now, and translate them into concrete implementation
instructions in a new "post-wrap-up remediation" cycle section of the audit log file. Do
not rewrite the Final Overseer report. Do NOT write to CHANGELOG.md in this round. Sync
HANDOFF.md plus TASK.md to point to the new next action.


Required checkpoint reads (quality guardrail):
- If you have not read your agent file this session, or if the session has been compacted
  since you last read it, read it now.
- Same for AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md
- ensure that you have completed all that is expected of you from your role and template.
- emit a git commit block per the "Convention for agent-produced git commit blocks" section in this workflow file (mid-cycle form for non-closeout rounds; closeout form when you write the CHANGELOG/DEVLOG entry), placed before your handoff prompt(s).
```

### Template F — Role 2 Post-Wrap-Up Implementation

```text
Below N and <N> = <phase number to be filled in by user>

You are acting as Role 2 — Implementer / Supplemental Auditor from
multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md.

Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md
Target cycle: post-wrap-up remediation

Role 1 has translated actionable wrap-up findings into repair instructions in the
post-wrap-up remediation cycle of the audit log file. 
Per the Role 2 description: assume the audit may not be exhaustive and that it may contain errors; document any in-situ catches; flag observations or push back when the audit or instructions have erred, drifted from reality, got something wrong, or where auditor judgment is otherwise needed; praise the auditor's clever instruction-writing where warranted.
Implement only that triaged subset,
perform the justified validation, append your implementation report in the same cycle
section of the audit log file, and update HANDOFF.md and TASK.md. Do NOT write to
CHANGELOG.md until the post-wrap-up cycle closes.

At round end, declare either "re-audit needed" or "skip-reaudit recommended" per the
Skip-Reaudit Criteria.


Required checkpoint reads (quality guardrail):
- If you have not read your agent file this session, or if the session has been compacted
  since you last read it, read it now.
- Same for AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md
- ensure that you have completed all that is expected of you from your role and template.
- emit a git commit block per the "Convention for agent-produced git commit blocks" section in this workflow file (mid-cycle form for non-closeout rounds; closeout form when you write the CHANGELOG/DEVLOG entry), placed before your handoff prompt(s).
```

### Template G — Role 1 Final Closeout After Wrap-Up Remediation

```text
Below N and <N> = <phase number to be filled in by user>

You are acting as Role 1 — Auditor / Instruction Writer from
multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md.

Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md
Target cycle: post-wrap-up remediation

Role 2 has completed the post-wrap-up remediation round. Re-audit the live code, determine
whether the actionable findings from the Final Overseer audit are now resolved, and append
the final closeout judgment to the post-wrap-up remediation cycle in the audit log file.
Per the Role 1 description: Leave praise in the cycle section of the audit log file to the implementer when the implementer finds gaps in the audit or identifies clever ways to solve problems encountered in the code reality; push back against deviations from instructions when appropriate: push back when the implementer's deviations, fixes, or observations don't hold up under re-audit.
If closed: update SPEC priority status table, write the cycle-closeout CHANGELOG entry
with consolidated authorship, and sync HANDOFF.md + TASK.md if the target file is now
fully closed.

**Phase-final X-bump responsibility.** Per
`AGENT_CONVENTIONS.md § CHANGELOG.md and DEVLOG.md`, this template's
closeout CHANGELOG entry IS the last-of-phase entry by construction (it
follows the wrap-up audit's remediation). Bump `X` to the just-completed
phase number and reset `YY` to `00`: version your closeout entry as
`v0.<N>.00` where `<N>` = the phase number being closed. Earlier CHANGELOG
entries during the phase kept `X` at the prior phase number; this entry
is the milestone marker.


Required checkpoint reads (quality guardrail):
- If you have not read your agent file this session, or if the session has been compacted
  since you last read it, read it now.
- Same for AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md
- ensure that you have completed all that is expected of you from your role and template.
- emit a git commit block per the "Convention for agent-produced git commit blocks" section in this workflow file (mid-cycle form for non-closeout rounds; closeout form when you write the CHANGELOG/DEVLOG entry), placed before your handoff prompt(s).
```

### Template H — Role 1 Cycle Closeout After Skip-Reaudit (new in v2)

```text
Below N and <N> = <phase number to be filled in by user>

You are acting as Role 1 — Auditor / Instruction Writer from
multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md.

Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md
Closed cycle: <cycle name> or <cycle index>
Next target cycle: <next cycle name> or <next cycle index> (optional; omit if this is the last cycle)

Role 2 recommended skip-reaudit for the closed cycle and the orchestrator accepted.
Perform these steps in order:

1. Lightweight verification: spot-check 2-3 of the surfaces Role 2 claims to have changed.
   Confirm no obvious drift. This is NOT a full re-audit — just a sanity pass to catch
   gross divergence.
2. Update the SPEC's priority status table for the closed cycle.
3. Write the cycle-closeout CHANGELOG entry with consolidated authorship. Survey the
   audit log file's cycle section for all `**Authors:**` lines, deduplicate (user first),
   and emit one consolidated line.
4. Update HANDOFF.md and TASK.md.
5. (Optional) If there is a next cycle queued in the strategy file, produce a handoff
   prompt for the next cycle's Role 1 initial audit in Template A format, subject to
   the four "Convention for agent-produced handoff prompts" rules at the top of the
   Prompt Templates section: omit the `Below N` line, hard-code concrete paths,
   include the two guardrail blocks verbatim, and precede the launcher with an
   Orchestrator advisory naming the next cycle's R1 assignee from
   `PHASE<N>_STRATEGY.md`. Set `Target cycle:` to that cycle's name and index from the
   strategy file. Otherwise, stop — this was the final cycle.


Required checkpoint reads (quality guardrail):
- If you have not read your agent file this session, or if the session has been compacted
  since you last read it, read it now.
- Same for AGENT_CONVENTIONS.md.

Before giving your wrap-up summary comments to the user:
- re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md
- ensure that you have completed all that is expected of you from your role and template.
- emit a git commit block per the "Convention for agent-produced git commit blocks" section in this workflow file (mid-cycle form for non-closeout rounds; closeout form when you write the CHANGELOG/DEVLOG entry), placed before your handoff prompt(s).
```

### Template I — Strategist: Phase Plan of Attack (STRATEGY.md generator)

Run ONCE per phase, before any audit cycle. Output goes to
`multi-agent/plans/PHASE<N>_STRATEGY.md`. The agent running this prompt does
not audit or implement — it produces the plan-of-attack document that drives
cycle execution.

```text
Below N and <N> = <phase number to be filled in by user>

You are acting as a Phase Strategist from
multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md.

Run ONCE per phase, before any audit cycle. You are not auditing or
implementing — you are producing a plan-of-attack document that will drive
cycle execution.

Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file (to produce): multi-agent/plans/PHASE<N>_STRATEGY.md

## Goal

Produce PHASE<N>_STRATEGY.md containing:
1. A phase summary.
2. A cycle table — priorities split into execution-ordered "beefy chunks,"
   each worth a full audit → implement → re-audit loop.
3. Per-cycle agent assignments for Role 1 / Role 2 / Role 3, with a PRIMARY
   choice AND at least one ALTERNATIVE recommendation per role per cycle.
4. Deviation rationale for any cycle that differs from the baseline strategy.
5. A Final Overseer entry (primary + alt) for phase close.

Do NOT start cycles. Do NOT edit the SPEC. Propose the STRATEGY.md draft in
chat first and wait for user approval before writing the file.

## Inputs to read before proposing

1. multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md — especially
   "Cycle Granularity — substantive vs batched."
2. The target SPEC, end-to-end. Note priority order, explicit pair directives
   (e.g., "implement X and Y in the same Role 2 round"), and any priorities
   already CLOSED in the status table.
3. The sibling AUDIT_LOG.md (current cycle sections only) to confirm which
   priorities are genuinely open.
4. multi-agent/AGENT_CONVENTIONS.md § Scope authority — never silently
   descope.
5. multi-agent/project_context/HANDOFF.md — honor any prior-chat agreements
   on cycle granularity or agent assignments.

## User agent roster (keep up to date as models evolve)

Tier 1 — deep reasoning (use on audit/closeout of semantically tricky cycles):
  - Claude Code — Opus (Effort: High; Extra High for phase-ending/Final Overseer)
  - Github Copilot — Claude Opus (Effort: High; Extra High for phase-ending/Final Overseer)
  - Github Copilot — GPT-5.5 (Reasoning: Extra High)
  - Github Copilot — GPT-5.5 (Reasoning: High)
  - Codex — GPT-5.5 (Reasoning: Extra High)
  - Codex — GPT-5.5 (Reasoning: High)
  - Gemini 3.1 Pro Preview (Thinking: high recommended for audit/overseer work)
  - Gemini 2.5 Pro (Thinking: high recommended for audit/overseer work)

Tier 2 — balanced / default:
  - Claude Code — Sonnet (Effort: Extra High for audit/re-audit; High for implementation)
  - Github Copilot — Claude Sonnet (Effort: Extra High for audit/re-audit; High for implementation)
  - Github Copilot — GPT-5.5 (Reasoning: Medium)
  - Codex — GPT-5.5 (Reasoning: Medium)

Tier 3 — fast / cheap (mechanical cycles, spot-checks):
  - Claude Code — Haiku (Effort: High default; Medium/Low sufficient for simple spot-checks)
  - Github Copilot — GPT-5.5 (Reasoning: Low)
  - Github Copilot — Claude Haiku (Effort: High default)
  - Codex — GPT-5.5 (Reasoning: Low)
  - Gemini 3 Flash Preview
  - Gemini 3.1 Flash Lite Preview

Execution environment: VSCode. Chat extensions for Copilot, Claude, Codex;
terminal for Gemini (more stable). Single-window preferred.

## Baseline strategies — deviate only with rationale

**Claude Code available (preferred):**
- R1 primary: Claude Code — Sonnet (Effort: Extra High)
  R1 alt: Github Copilot — Claude Sonnet (Effort: Extra High); or Github Copilot — GPT-5.5 (Reasoning: High)
- R2 primary: Github Copilot — GPT-5.5 (Reasoning: Medium or High)
  R2 alt: Codex — GPT-5.5 (Reasoning: High); or Github Copilot — Claude Sonnet (Effort: High)
- R3 primary: Claude Code — Sonnet (Effort: Extra High; different chat session from R2)
  R3 alt: Github Copilot — Claude Sonnet (Effort: Extra High); upgrade to Opus-tier for
  phase-ending or high-nuance cycles
- Final Overseer primary: Gemini 3.1 Pro Preview (Thinking: high)
  Final Overseer alt: Codex — GPT-5.5 (Reasoning: High); or Gemini 2.5 Pro (Thinking: high)
- Post-overseer triage auditor: same as R1
- Post-overseer implementer: same as R2

**Claude Code unavailable:**
- R1 primary: Github Copilot — GPT-5.5 (Reasoning: High)
  R1 alt: Github Copilot — Claude Opus (Effort: High) / Claude Sonnet (Effort: Extra High)
- R2 primary: Codex — GPT-5.5 (Reasoning: Medium or High)
  R2 alt: Github Copilot — GPT-5.5 (Reasoning: Medium)
- R3 primary: Github Copilot — GPT-5.5 (Reasoning: High; different session)
  R3 alt: Gemini 3.1 Pro Preview (Thinking: high)
- Final Overseer: Gemini 3.1 Pro Preview (Thinking: high)
- Post-overseer triage auditor: same as R1
- Post-overseer implementer: same as R2

## Deviation guidance — when to upgrade or downgrade

Upgrade R1 (and usually R3) to **Tier 1** when the cycle involves:
- Semantic nuance (mixed-meaning labels, subtle invariants, nontrivial math
  or biological correctness).
- Full-surface cross-cutting passes (help-string rewrites spanning all parser
  groups, cross-file doc sweeps).
- Phase-ending closeouts (file moves, reference propagation across many
  files, archival operations).
- Decisions where accepting auditor drift would cost downstream rework.

Keep R1 at **Tier 2** when the cycle is:
- Mechanical renames, deprecations, default flips, single-flag additions.
- Contained parser-group changes with no cross-pipeline impact.
- Runtime wiring of a flag whose semantics are already fully spec'd.

R2 selection:
- Default to GPT-5.5 (Reasoning: Medium) for mechanical cycles.
- Upgrade to GPT-5.5 (Reasoning: High) for multi-file substantive cycles.
- Upgrade to Codex — GPT-5.5 (Reasoning: High or Extra High) for cycles touching runtime
  correctness, test-harness wiring, or SPEC-mandated paired priorities.
- Flag in STRATEGY.md which cycles Role 2 is expected to be able to declare
  skip-reaudit-eligible (almost always only purely mechanical cycles).

R3 selection:
- Match R1 tier for the same cycle.
- Always run in a DIFFERENT chat session from R2 so the auditor re-reads code
  fresh rather than remembering implementation internals.
- For light cycles (audit-only, no runtime code), consider deferring R3 to
  the next cycle's R3. Use the form `R3: Defer to <next cycle> R3` in that
  cycle's cell, and mirror it in the absorbing cycle's Notes column. See the
  "Deferred R3" rules in the Cycle Granularity section of
  spec_plan_three_role_audit_loop-v2.md.

Final Overseer:
- Default Gemini 3.1 Pro Preview (Thinking: high). Runs once, AFTER the last
  execution cycle of the phase closes. Scope: full phase audit + cross-cycle
  drift check + comparison against KNOWN_ISSUES.md and BRAINSTORM.md.
- The phase archive (`git mv` of SPEC + AUDIT_LOG + STRATEGY + FEEDBACK into
  `multi-agent/plans/archived/<YYYYMMDD>-`) is a SEPARATE operation that
  happens AFTER Final Overseer completes (and after any post-wrap-up
  remediation cycles close). The orchestrator performs the archive outside the
  cycle structure. See § Standard Execution Loop / Step 8 — Phase Archive for
  the full sequencing rule. STRATEGY.md authors should write the Final
  Overseer entry's `Trigger:` field as: "after cycle <X> closes (the last
  execution cycle of the phase). The phase archive is a separate operation
  performed after Final Overseer + any post-wrap-up cycles close — never
  within Final Overseer or any execution cycle."

## Performance tuning — effort, reasoning, and thinking budgets

All three frontier vendors expose a comparable runtime knob that trades tokens and
latency for deeper reasoning. The model name selects the model; the toggle selects how
hard it thinks. Notation in this project is parallel:

| Vendor | Model examples | Toggle name | Toggle values |
|---|---|---|---|
| Anthropic (Claude Code, Github Copilot — Claude) | Sonnet, Opus, Haiku | **Effort** | Low \| Medium \| **High** (default) \| Extra High \| Max |
| OpenAI (Codex, Github Copilot — GPT) | GPT-5.5, GPT-5.4 | **Reasoning** | Low \| Medium \| **High** \| Extra High |
| Google (Gemini terminal) | Gemini 3.1 Pro Preview, 2.5 Pro | **Thinking** | low / medium / high (CLI flag value, varies by Gemini CLI version) |

Always write agent specs as `<vendor harness> — <model> (<Toggle>: <value>)`. Examples:
`Claude Code — Sonnet (Effort: Extra High)`,
`Codex — GPT-5.5 (Reasoning: High)`,
`Github Copilot — GPT-5.5 (Reasoning: Medium)`,
`Gemini 3.1 Pro Preview (Thinking: high)`.

Higher toggle values pay off most on judgment-heavy tasks (auditing, re-auditing, phase
closeout); implementation from a clear spec is more mechanical and rarely needs above the
default.

### Per-role recommendations

| Role | Task | Claude Effort | GPT Reasoning | Gemini Thinking |
|---|---|---|---|---|
| R1 — Auditor (Sonnet/GPT-5.5) | Single-surface audit | **Extra High** | **High** | high |
| R1 — Auditor (Sonnet/GPT-5.5) | Paired/complex audit | **Max** | **Extra High** | high |
| R2 — Implementer | Mechanical implementation of clear spec | **High** (default) | **Medium** | medium |
| R2 — Implementer | Complex cross-module repair / dependency fix | **Extra High** | **High** | high |
| R3 — Re-auditor / Closeout | Verify implementation matches findings | **Extra High** | **High** | high |
| R1/R3 — Auditor (Opus) | Any audit | **High** — Opus reasons deeply at High | n/a | n/a |
| Final Overseer | Cross-cycle phase closeout | **Extra High** (Opus) | **High** (Codex) | high (Gemini) |

Principle: for Opus, High is already strong — use Extra High only for consequential
phase-ending or Final Overseer judgments. Max effort is reserved for the most complex
multi-priority Sonnet audits where missing a cross-module interaction would cost
downstream rework. The exact Gemini Thinking CLI flag varies by Gemini version; calibrate
per your current Gemini CLI. This project does not yet have a fixed CLI invocation
standard for thinking budgets.

## Cycle granularity rules (from this workflow's Cycle Granularity section)

- A cycle must be beefy enough to justify three role-rounds.
- Pair priorities the SPEC explicitly directs be implemented together.
- Batch mechanical priorities into one cycle per phase — this is where
  skip-reaudit pays back the coordination cost.
- Substantive priorities stand alone, or pair only with semantically-coupled
  siblings.
- Audit-only phases (no code changes): typically 1 cycle, R3 often skipped.
- Phase-ending closeouts (doc sweeps, file moves, archival): their own cycle,
  Tier 1 R1 and R3.
- Do NOT silently descope priorities. If a priority seems too thin to stand
  alone, pair it with a substantive priority — never drop it.
- Do NOT reorder SPEC priorities against declared dependencies; reorder only
  within groups the SPEC itself treats as reorderable.

## Output format — multi-agent/plans/PHASE<N>_STRATEGY.md

Produce a markdown file with these sections in this order:

### Header
    # Phase <N> — Strategy (plan of attack)

    **Generated:** YYYY-MM-DD HH:MM EST
    **Author:** <agent and model running this prompt>
    **SPEC revision:** <last-modified date or git commit>
    **Phase title:** <one line>

### Phase summary
One paragraph: priority count (active / closed), dominant themes, known
risks, whether mostly mechanical or substantive.

### Cycle table
Columns: **Cycle index** | **Cycle name** | **Priorities** | **R1 (audit)** |
**R2 (implement)** | **R3 (closeout)** | **Notes**

- Rows ordered by execution order.
- **Cycle index:** simple integers starting at 1 and incrementing by 1 for
  each row. The final numbered cycle before the Final Overseer pass carries
  the highest index. Use this index throughout the workflow to reference
  cycles concisely (e.g., "Target cycle: 3").
- **Cycle name:** the named form — `Phase<N>.<letter>` (e.g., `14S.1a`,
  `14S.1b`) or `Phase<N>S.<letter>` for supplemental phases. Names are
  stable, phase-scoped, and more descriptive than the index.
- Both index and name should appear in handoff prompts the closing agent
  produces for the next cycle so any downstream agent can locate the cycle
  unambiguously.
- Each role cell lists `Primary: <agent + level>` on one line and
  `Alt: <agent(s) + level>` on the next (use `<br>` for in-cell line breaks).
  Bundle the toggle value with the agent name as a single specification, using the
  uniform `<harness> — <model> (<Toggle>: <value>)` form — e.g.,
  `Claude Code — Sonnet (Effort: Extra High)`,
  `Codex — GPT-5.5 (Reasoning: High)`,
  `Github Copilot — GPT-5.5 (Reasoning: Medium)`,
  `Gemini 3.1 Pro Preview (Thinking: high)`.
  This ensures advisory blocks and user-facing prompts carry the full
  assignment without a separate lookup.
- An R3 cell may instead read `Defer to <next cycle index / name> R3` (no
  Primary/Alt needed) for light cycles per the "Deferred R3" rules. The
  absorbing cycle must mirror the deferral in its Notes column.
- Notes column flags skip-reaudit eligibility, SPEC-mandated paired
  priorities, different-session requirements, deferred-R3 absorption, and
  any other execution constraints.

### Deviation rationale
For each row where agent assignment deviates from baseline: one or two
sentences on why (semantic nuance / blast radius / phase-ending / Claude Code
quota / etc.).

### Execution summary
2-4 sentences: total cycle count, total priorities covered, count of Tier 1
upgrades, Final Overseer timing, known risks or unknowns.

### Final Overseer
Named primary + alt agent, trigger condition (after which cycle closes),
scope (full phase audit + cross-cycle drift + KNOWN_ISSUES/BRAINSTORM
comparison).

### Amendment log
Placeholder section for dated amendments if cycle boundaries shift
mid-phase. Never rewrite earlier rows — append amendments only.

## Final chat output — next-action handoff prompt

After STRATEGY.md is written (or, if still pending approval, after the draft
is approved), your closing chat message to the user must include a ready-to-
paste launcher prompt for Role 1 initial audit on cycle index 1. This is
analogous to the handoff prompts that Templates B / C / H require at the end
of their rounds.

The handoff prompt must:
- Be a `text` code block.
- Follow Template A format (use Template A from this workflow file as the
  structural reference), subject to the four "Convention for agent-produced
  handoff prompts" rules at the top of the Prompt Templates section: **omit
  the `Below N and <N> = ...` line**, hard-code every `PHASE<N>_*.md` path
  with the concrete filename (including supplemental-phase filenames where
  applicable, e.g., `PHASE14_SUPPLEMENTAL-SPEC.md`), include the two
  guardrail blocks verbatim, and **precede the code block with an
  Orchestrator advisory** naming the cycle index 1 R1 assignee from
  `PHASE<N>_STRATEGY.md`.
- Set `Target cycle:` to the cycle index 1 value filled in as
  `<cycle name> (index 1) — <priorities in that cycle>`.
- Set `Current round: initial audit`.

This gives the orchestrator a one-click kickoff for the phase without having
to reassemble the launcher from Template A manually.

## Rules

1. Propose the draft in chat first. Wait for user approval before writing
   the file to disk.
2. If the user has already confirmed cycle granularity or agent assignments
   (in a prior chat or HANDOFF.md), match those. Flag any proposed
   deviations explicitly for user review.
3. Every R1/R2/R3 cell MUST include a Primary and at least one Alt. Do not
   leave any role cell with only one agent.
4. At phase close, archive PHASE<N>_STRATEGY.md alongside SPEC and
   AUDIT_LOG: multi-agent/plans/archived/YYYYMMDD-PHASE<N>_STRATEGY.md.
5. STRATEGY.md is a living document during the phase. Append amendments
   with date + rationale; never rewrite history.
6. Your closing chat message must include the Role 1 initial-audit launcher
   prompt for cycle index 1 (see "Final chat output" above).
```

---

### Template J — Outgoing Orchestrator: Succession Briefing (new in v3)

Run ONCE per phase, **at phase close, AFTER the Final Overseer has signed
off**, BEFORE the phase archive. Output goes to
`multi-agent/plans/PHASE<N>_SUCCESSION_BRIEFING.md`. The agent running this
prompt is the **outgoing Orchestrator** writing for its successor (the
next-phase Orchestrator).

**Why this template exists.** The formal phase artifacts (SPEC, AUDIT_LOG,
STRATEGY, BACKGROUND, SURPRISE_LOG) archive intact and capture the
contract + execution history of the phase. They do NOT capture the
**accumulated wisdom** the Orchestrator developed across the phase: which
patterns recur, which blind spots are durable, which deliberation patterns
worked, what didn't, recommended working style for the next-phase
Orchestrator. The Succession Briefing is that wisdom transfer.

**Why the Orchestrator writes this, not the Final Overseer.** Final
Overseer's structural value is independence from the Orchestrator's
accumulated frame. If Final Overseer wrote the Succession Briefing, they
would either lose that independence (now thinking like an Orchestrator) or
write a briefing without the deliberation context (worse than the
outgoing Orchestrator's). The cleanest split: Final Overseer audits
independently; outgoing Orchestrator inherits Final Overseer's findings
and writes the wisdom transfer.

**Why AFTER Final Overseer signs off, not before.** Writing the briefing
before Final Overseer would (a) introduce bias into Final Overseer's
review (they might read the briefing as a hint), and (b) miss any
findings Final Overseer surfaces. Writing AFTER lets Final Overseer's
findings become inputs to the briefing.

```text
Below N and <N> = <phase number to be filled in by user>

You are acting as the OUTGOING Orchestrator from
multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md.

Run ONCE at phase close, AFTER the Final Overseer has signed off, BEFORE
the phase archive. You are writing the Succession Briefing for the
next-phase Orchestrator. You are not auditing or implementing — you are
producing a wisdom-transfer document.

Spec file (now closed):              multi-agent/plans/PHASE<N>_SPEC.md
Audit log file (now closed):         multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file (now closed):          multi-agent/plans/PHASE<N>_STRATEGY.md
Background scratchpad (now closed):  multi-agent/plans/PHASE<N>_BACKGROUND.md
Surprise log (now closed):           multi-agent/plans/PHASE<N>_SURPRISE_LOG.md
Final Overseer entry:                multi-agent/plans/PHASE<N>_AUDIT_LOG.md
                                     (Final Overseer wrap-up section)
Briefing file (to produce):          multi-agent/plans/PHASE<N>_SUCCESSION_BRIEFING.md

## Goal

Produce PHASE<N>_SUCCESSION_BRIEFING.md as a wisdom-transfer document for
the next-phase Orchestrator. The briefing is NOT a recreation of the
formal artifacts (those archive intact); it captures what's NOT in the
formal artifacts.

Required sections:

1. **Header.** Phase number, date, outgoing-Orchestrator authorship,
   Final Overseer outcome (CLOSE / SEND-BACK-FOR-REMEDIATION-DONE / etc.).
2. **Phase summary.** One paragraph: theme, headline accomplishments,
   total cycle count, total commits, headline metrics (e.g.,
   "X cycles closed, Y mid-phase amendments, Z supplemental cycles
   inserted").
3. **Cross-cycle pattern memory.** Recurring patterns that emerged
   during this phase. For each pattern: name it, describe what it looks
   like, give specific cycle references where it surfaced, recommend how
   the next Orchestrator should watch for it. Examples from Phase 15:
     - GAP-1 = parity table omission. Mandate cell-by-cell verification.
     - Authorship/Effort drift. Run a sweep periodically.
     - Help-text contract simplification. R3 inline-remediation precedent.
     - Static-default overfit. Prefer biology priors when empirical
       defaults derive from amplification-rich fixtures.
4. **Recurring blind spots that need vigilance.** Things that slipped
   through more than once even with active vigilance. Specific
   cross-pipeline parity blind spot lives here. Each entry: what it is,
   why it's durable, recommended counter-discipline.
5. **Deliberation patterns that worked.** Specific patterns the
   Principal-Orchestrator deliberation produced that landed cleanly.
   Examples: pre-R3 audits with locked-contract checklist + cell-by-cell
   parity verification; orchestrator-outside-cycle clarifications
   appended via corrections appendix; multi-cycle splits when scope
   ballooned.
6. **Things that didn't work + lessons.** Approaches the phase tried
   that failed. Examples from Phase 15: GMM `learn` mode in raw RCN
   space (artifact-tight cutoffs); GMM `learn` mode at all (continuous
   amplification distributions don't yield clean valleys); the static-
   default sweep producing dataset-overfit defaults. Each entry: what
   was tried, why it failed, what was done instead, what the next
   Orchestrator should NOT re-attempt without new information.
7. **Brainstorm seeds + SOUP entries that emerged but weren't
   formalized.** Things surfaced during cycle execution that became
   future-phase candidates. Cross-reference SOUP file additions made
   this phase + relevant BRAINSTORM entries that emerged organically.
8. **Open SURPRISE/ISSUE entries with cross-phase implications.**
   Active SURPRISE_LOG and KNOWN_ISSUES entries whose landing zones are
   future-phase. Each entry: identifier, summary, recommended next-phase
   action.
9. **Recommended Orchestrator working style for next phase.**
   Concrete recommendations: model + Effort choice (e.g., "Claude Code
   Opus Max held up well across N cycle rounds"); expected cycle count
   range based on this phase's pattern; recurring user-feedback memories
   to load on cold-start; mid-flight pushback frequency observed; rough
   work-effort budget per cycle.
10. **Final Overseer's findings + carry-forwards.** Summarize Final
    Overseer's verdict + any items they flagged for next-phase pickup.
11. **Praise + things to keep doing.** Things that worked that should
    NOT be discarded by the next Orchestrator looking for novelty.
    Includes specific Principal-side directives that became
    load-bearing (e.g., "feedback memory X became load-bearing this
    phase").
12. **Things to avoid repeating.** Specific anti-patterns the next
    Orchestrator should not re-introduce. Cross-reference the
    "Things that didn't work" section if it overlaps.

## Inputs to read before drafting

1. multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md — the
   Orchestrator role section (this defines what the briefing should
   transfer).
2. The just-closed phase artifacts in full:
   - PHASE<N>_SPEC.md
   - PHASE<N>_AUDIT_LOG.md (especially the Final Overseer wrap-up
     entry)
   - PHASE<N>_STRATEGY.md (cycle table + amendment log)
   - PHASE<N>_BACKGROUND.md (decision history)
   - PHASE<N>_SURPRISE_LOG.md (all entries; note Resolved vs Active)
3. Recent CHANGELOG.md entries for the phase (use grep to find them by
   version anchor).
4. Recent DEVLOG.md entries from the phase (orchestrator-housekeeping +
   planning-surface engineering).
5. multi-agent/AGENT_CONVENTIONS.md (for the current convention state
   at phase close).
6. multi-agent/tracking/KNOWN_ISSUES.md (Active entries; flag any with
   cross-phase implications).
7. multi-agent/tracking/BRAINSTORM.md (Active entries; flag any
   relevant to next-phase candidates).
8. The Orchestrator's auto-memory store at
   `~/.claude/projects/<project>/memory/` — the feedback memories that
   accumulated during the phase. The Succession Briefing should
   surface these explicitly so the next Orchestrator knows to load
   them.

## Output format

Use this scaffold for PHASE<N>_SUCCESSION_BRIEFING.md:

```markdown
# Phase <N> Succession Briefing

**Generated:** <YYYY-MM-DD HH:MM TZ>
**Phase:** <N> — <phase title>
**Outgoing Orchestrator:** <agent + version + Effort> (chats:
                            <list of chat IDs or "session-anchored">)
**Final Overseer outcome:** <CLOSE / SEND-BACK-DONE / etc.>
**Phase close date:** <YYYY-MM-DD>
**Phase total:** <N cycles closed, M mid-phase amendments,
                  K supplemental cycles inserted>

## 1. Phase summary

<one paragraph>

## 2. Cross-cycle pattern memory

### Pattern: <name>
- **What it looks like:** ...
- **Cycle references:** ...
- **Recommended counter-discipline:** ...

[repeat per pattern]

## 3. Recurring blind spots

[as above]

## 4. Deliberation patterns that worked

[as above]

## 5. Things that didn't work + lessons

[as above]

## 6. Brainstorm seeds + SOUP entries that emerged

[as above; cross-reference file paths]

## 7. Open SURPRISE/ISSUE entries with cross-phase implications

[as above]

## 8. Recommended Orchestrator working style for next phase

[as above]

## 9. Final Overseer's findings + carry-forwards

[summarize]

## 10. Praise + things to keep doing

[as above]

## 11. Things to avoid repeating

[as above]
```

## Rules

1. Propose the draft in chat first. Wait for user (Principal) approval
   before writing the file to disk.
2. The Succession Briefing is wisdom transfer, NOT a re-summary of the
   formal artifacts. Resist the urge to recap the SPEC; instead capture
   what's NOT in the SPEC.
3. Every cross-cycle pattern + blind spot entry MUST cite specific cycle
   references. Generic patterns without references are weaker than
   concrete examples.
4. At phase archive (which follows this briefing), the Principal moves
   PHASE<N>_SUCCESSION_BRIEFING.md to
   multi-agent/plans/archived/YYYYMMDD-PHASE<N>_SUCCESSION_BRIEFING.md
   alongside SPEC, AUDIT_LOG, STRATEGY, BACKGROUND, SURPRISE_LOG.
5. The briefing is read on cold-start by the next-phase Orchestrator as
   part of Template I (Strategist) inputs. Reference the briefing path
   in the next phase's STRATEGY.md "Pre-cycle context" if appropriate.
6. Authorship line: include the outgoing Orchestrator's full attribution
   (e.g., `Authors: John M. Urban, Claude Code 2.1.118 (claude-opus-4-7
   ; Effort: Max)`). If the role spanned multiple Claude chats, list
   them all per AGENT_CONVENTIONS.md authorship-consolidation rules.
7. Do NOT include implementation details that belong in the formal
   artifacts. The briefing is for things that don't fit there.
8. Do NOT execute git commands. Emit a commit block per
   feedback_commit_message_file_convention.md; the Principal runs it.
```

---

## 2-Role Mode

If only two agents are available:

1. Role 1 performs both the initial audits and the final wrap-up audit.
2. Role 2 performs implementation plus in-situ supplemental auditing.
3. Keep the same file-update and closeout rules (SPEC stays clean, AUDIT_LOG holds
   round-by-round history, one CHANGELOG entry per cycle at closeout).
4. Skip-reaudit declaration by Role 2 still applies, and Role 1 still performs lightweight
   spot-check at closeout if orchestrator accepts the skip.

---

## Notes For The User

This file works best when you tell each agent its role before asking it to read the file.
The workflow is intentionally more structured and less verbose than a raw historical prompt,
but the prompt templates are designed to preserve the behavior of a successful human-tested
multi-agent loop.

**v2-specific notes:**

- The v2 workflow is opt-in. The original workflow remains at
  `multi-agent/workflows/spec_plan_three_role_audit_loop-v1.md`. Pick one per target
  SPEC; do not mix.
- When starting a new SPEC under v2, create the sibling `AUDIT_LOG.md` file up front.
  Template skeleton:

  ```markdown
  # PHASE <N> AUDIT LOG

  Sibling to `multi-agent/plans/PHASE<N>_SPEC.md`. Captures round-by-round history of the
  audit-implement-reaudit loop for this phase under
  `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md`.

  ## Cycle: <first cycle name> — OPEN (awaiting Role 1 initial audit)
  ```

- For an in-flight SPEC that was previously run under v1 (embedded audit blocks), you can
  migrate to v2 by moving the embedded audit blocks into a new AUDIT_LOG file. Do not
  mutate historical CHANGELOG entries.

- If the cycle is "Phase 14 Supplemental Phase 1" (batched cycle containing
  14-S10 + 14-S4 + 14-S5 + 14-S29 + 14-S30 + 14-S27 + 14-S28 + 14-S22 + 14-S23 + 14-S26),
  the cycle name can be literally "Phase 1 — structural parser changes". The priority
  status table in the SPEC tracks each priority's individual close state; the CHANGELOG
  entry covers the whole batch.
