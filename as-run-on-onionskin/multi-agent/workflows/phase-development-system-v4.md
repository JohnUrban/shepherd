# Phase Development System — v4 (DRAFT; in development)

**Status:** **DRAFT — IN DEVELOPMENT.** Not yet the active workflow. v3 (the
two-file system at `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md`
+ `multi-agent/workflows/phase-development-system_PDS-v3.md`) remains the
**current active default** until v4 stabilizes through review + at least one
phase of trial use. Do NOT pick v4 for an in-flight phase yet.

**Last updated:** 2026-05-03

**Purpose of v4:** consolidate the v3 two-file workflow (cycle execution
mechanics in audit-loop-v3 + upstream stages in PDS-v3) into a single
unified phase-development workflow document that covers the full
SOUP-to-archive lifecycle, with the formalized Orchestrator role as the
unifying spine.

## Revision markers

- **v4 (current; DRAFT).** Merges v3's two-file system into one document.
  Same conventions, same role definitions, same templates — reorganized
  under a unified Stage 1-8 lifecycle spine. The Orchestrator role
  (formalized in v3) becomes the unifying narrative thread that runs from
  Stage 1 (SOUP) through Stage 8 (Phase Archive). New filename drops the
  redundant `_PDS` suffix that the v1-v3 lineage carried. v4 is
  drop-in-compatible with v3 in terms of conventions + templates; only the
  organization differs.
- **v3 (still active).** Two-file system:
  `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md` (cycle
  execution: Strategist + R1/R2/R3 + Final Overseer + Succession Briefing)
  + `multi-agent/workflows/phase-development-system_PDS-v3.md` (upstream
  stages: SOUP + BRAINSTORM + SPEC engineering). Adds the formal
  Orchestrator role; multi-chat continuity; Succession Briefing
  (Template J).
- **v2 (deprecated; archived at `multi-agent/workflows/archived/`).** Three-file system:
  audit-loop-v2 + PDS-v2 + orchestrator-v2 (companion). Introduced
  AUDIT_LOG sibling, per-cycle CHANGELOG batching, substantive-priorities
  rule, cycle granularity, authorship consolidation, Role 2 skip-reaudit
  declaration. Phase 14 Supplemental migrated mid-stream to v2; Phase 15
  used v2 throughout.
- **v1 (deprecated; archived at `multi-agent/workflows/archived/`).** Three-file system:
  audit-loop-v1 + PDS-v1 + orchestrator-v1. Per-role-round CHANGELOG
  entries; audit trail embedded in the SPEC.

## Migration path from v3

If you want to migrate an in-flight phase from v3 to v4: don't, until v4
stabilizes. Phase 15 closes out under v3. Phase 16 onward may use v4 if
v4 is approved as the active workflow by then.

If a future phase uses v4 directly (no v3 history), the v4 file is
**self-contained at the policy / stage-structure layer** (conceptual
framing, Orchestrator role, the eight stages, conventions, failure
modes, migration paths, sibling-file skeletons). However, **prompt
bodies for all 22 templates remain cross-referenced to v3 source files
until Pass 4c lands** — see § 14 for the index + cross-reference
locations. To use v4 fully self-contained at the prompt layer too,
either wait for Pass 4c or paste the relevant template body in from
v3 alongside v4. (Pass 4c is on the v4 → active promotion gate; see
Appendix A reviewer checklist.)

## CHANGELOG vs DEVLOG note (carried forward from v0.14.64)

Throughout this file, "the cycle-closeout LOG entry" means either
`CHANGELOG.md` (product-scope cycles) or `multi-agent/DEVLOG.md`
(dev-system-scope cycles), routed by primary content. Shared `vR.X.YY`
version stream. See `multi-agent/AGENT_CONVENTIONS.md` for the routing
rule.

---

# Table of contents

1. [Conceptual framing](#1-conceptual-framing)
2. [The Orchestrator role](#2-the-orchestrator-role)
3. [Substantive priorities + cycle granularity](#3-substantive-priorities--cycle-granularity)
4. [Stage 1: SOUP](#4-stage-1-soup)
5. [Stage 2: BRAINSTORM iteration](#5-stage-2-brainstorm-iteration)
6. [Stage 3: SPEC engineering](#6-stage-3-spec-engineering)
7. [Stage 4: Strategist kickoff (Orchestrator's kickoff act)](#7-stage-4-strategist-kickoff-orchestrators-kickoff-act)
8. [Stage 5: Cycle execution (audit-implement-reaudit)](#8-stage-5-cycle-execution-audit-implement-reaudit)
9. [Stage 6: Final Overseer](#9-stage-6-final-overseer)
10. [Stage 7: Succession Briefing](#10-stage-7-succession-briefing)
11. [Stage 8: Phase Archive](#11-stage-8-phase-archive)
12. [Authorship + commit-block conventions](#12-authorship--commit-block-conventions)
13. [Common failure modes](#13-common-failure-modes)
14. [Prompt templates](#14-prompt-templates)
15. [2-Role Mode](#15-2-role-mode)
16. [Notes / migration paths](#16-notes--migration-paths)

---

# 1. Conceptual framing

## What PDS is

This is the multi-agent system by which **Phases** are brainstormed
(often from SOUP files), planned, spec'd, audited, implemented, and
finalized. Under v4, the system is captured in this single document
(was two files under v3: audit-loop-v3 + PDS-v3).

A **phase** in onionskin is a coherent block of related development
work — typically organized around a theme (e.g., "Phase 15: HMM
completeness") and containing 5-15 substantive priorities (numbered
SPEC<N>.<idx>) that ship across multiple cycles of audit-implement-reaudit
work.

## The four-tier brainstorming-surfaces hierarchy

See `multi-agent/AGENT_CONVENTIONS.md § Future-phase planning surfaces`
for the canonical rules. Summary of the four tiers, ordered from
"long-lived ideas reservoir" toward "active per-phase planning":

1. **`multi-agent/tracking/BRAINSTORM.md`** — long-lived ideas
   reservoir. Cross-phase. Entries here may seed future SOUP files.
2. **`multi-agent/plans/next/<THEME>_SOUP.md`** — pre-BRAINSTORM
   scratchpad for candidate future phases. Theme-named (vague / broad /
   lightly-evocative); never scope-anchoring; never numbered. The body
   MUST NOT contain SELF-references to phase numbers while in `next/`
   (rename-to-renumber discipline). Cross-references to actually-closed
   phases ARE allowed as historical anchors.
3. **`multi-agent/plans/PHASE<N>_BRAINSTORM.md`** (per-phase staging) —
   the live, open-for-expansion BRAINSTORM file derived from a labeled
   SOUP. Each entry has a `BRAIN<N>.<idx>` ID and a `Source:` field
   citing SOUP IDs + other source citations.
4. **`multi-agent/plans/PHASE<N>_SPEC.md`** — the culmination. Locked
   contract for the phase's audit-implement-reaudit cycles. Each
   priority has a `SPEC<N>.<idx>` ID and a `Source:` / `Covers:` field
   citing BRAIN IDs.

## Phase artifacts produced + consumed during the 8-stage lifecycle

| Stage | New artifacts produced | Active reads / amendments | Archived at end of stage |
|---|---|---|---|
| Stage 1: SOUP | `<THEME>_SOUP.md` in `plans/next/` (informal); `PHASE<N>_<THEME>_SOUP.md` in `plans/` (post-promotion, SOUP IDs labeled) | `tracking/BRAINSTORM.md`, `tracking/KNOWN_ISSUES.md` | SOUP archived after BRAINSTORM transfer verified clean (Stage 2 closeout) |
| Stage 2: BRAINSTORM iteration | `PHASE<N>_BRAINSTORM.md`, `PHASE<N>_FEEDBACK.md` | SOUP (read-only), BRAINSTORM, FEEDBACK | SOUP archived at start of stage close; BRAINSTORM + FEEDBACK stay live for Stage 3 |
| Stage 3: SPEC engineering | `PHASE<N>_SPEC.md`; `PHASE<N>_AUDIT_LOG.md` skeleton (created empty at SPEC lock — see § 16 startup checklist) | BRAINSTORM, FEEDBACK, SPEC | BRAINSTORM + FEEDBACK archived at SPEC lock |
| Stage 4: Strategist kickoff | `PHASE<N>_STRATEGY.md`; `PHASE<N>_SURPRISE_LOG.md` empty-required skeleton (created before first Template A launcher — see § 16 startup checklist + skeleton) | SPEC, AUDIT_LOG | none (SPEC + STRATEGY + AUDIT_LOG + SURPRISE_LOG stay live) |
| Stage 5: Cycle execution | optionally `PHASE<N>_BACKGROUND.md` (per-phase planning scratchpad — created when deliberation surfaces non-trivial decision history); cycle rounds append to existing AUDIT_LOG + SURPRISE_LOG (no new top-level files at this stage other than optional BACKGROUND) | SPEC (priority status table updates), AUDIT_LOG (every round), STRATEGY (mid-phase amendments), SURPRISE_LOG (every cycle's R1 carry-over scan + R3 closeout review gate are mandatory), BACKGROUND (decision history when present), HANDOFF, TASK, CHANGELOG/DEVLOG (one per cycle close) | none |
| Stage 6: Final Overseer | (none new — wrap-up audit appended to AUDIT_LOG) | SPEC, AUDIT_LOG, STRATEGY, BACKGROUND, SURPRISE_LOG, KNOWN_ISSUES, BRAINSTORM | none |
| Stage 7: Succession Briefing | `PHASE<N>_SUCCESSION_BRIEFING.md` | All Stage 6 reads + Final Overseer's wrap-up entry | none (Briefing stays for Stage 8 archive) |
| Stage 8: Phase Archive | (none new — `git mv` operations) | All artifacts | SPEC + AUDIT_LOG + STRATEGY + BACKGROUND + SURPRISE_LOG + SUCCESSION_BRIEFING all → `plans/archived/<YYYYMMDD>-*` |

**SURPRISE_LOG required + BACKGROUND optional.** `PHASE<N>_SURPRISE_LOG.md`
is a required sibling artifact, **created empty at Stage 4 (Strategist
kickoff) before the first Template A launcher fires** — see the Stage 4
startup-checklist step in § 16 and the skeleton template in the same
section. Every cycle's Templates A
(R1 initial audit), B (R2 implementation), and C (R1 re-audit) reference
the surprise log: A's mandatory KNOWN_ISSUES + SURPRISE_LOG carry-over
scan; B's status updates as the implementer picks up surprise entries
as in-cycle work; C's mandatory closeout review gate that flips status
on every cycle-tagged entry. The file may be empty (no surprises filed
in a phase is acceptable) but the file must exist so the templates'
`Surprise log file:` reference resolves and the carry-over scans are
non-skip-able.

`PHASE<N>_BACKGROUND.md` is optional/conditional. Created only when the
phase develops non-trivial decision history during cycle execution that
warrants a per-phase planning surface. Phase 15 had both files (the
deliberation that produced the cycle 15.4a-S3 + 15.4a-S4 split was
captured in BACKGROUND; surprise entries filed against multiple cycles
populated SURPRISE_LOG).

## Identifier system across the artifacts

- **`SOUP<N>.<idx>`** — entries within a labeled SOUP. Assigned at
  promotion (one-time labeling pass). E.g., `SOUP15.1`.
- **`BRAIN<N>.<idx>`** — entries within a per-phase BRAINSTORM. Each
  cites SOUP IDs in `Source:`. E.g., `BRAIN15.7 Source: SOUP15.3,
  SOUP15.5`.
- **`SPEC<N>.<idx>`** — priorities in the SPEC. Each cites BRAIN IDs in
  `Source:` / `Covers:`. E.g., `SPEC15.4 Covers: BRAIN15.7, BRAIN15.12`.
- **`Q<idx>` / `JQ<idx>`** — questions raised in FEEDBACK during
  iteration.
- **`[ISSUE:YYYY-MM-DD:N]`** — entries in `multi-agent/tracking/KNOWN_ISSUES.md`.
- **`[SURPRISE:N:cycle:round:I:idx]`** — entries in
  `PHASE<N>_SURPRISE_LOG.md`.

SPEC IDs are reorderable during SPEC engineering; once the SPEC is
locked at the start of cycle execution, IDs freeze and inserts get the
next-available number even if it appears "out of order" in the file's
reading sequence.

See `multi-agent/AGENT_CONVENTIONS.md § Identifier system` for full
format rules.

## CHANGELOG vs DEVLOG routing

Cycle closeouts produce **one CHANGELOG OR DEVLOG entry per cycle**,
routed by primary content:

- **Code/CLI/test/user-doc changes** → `CHANGELOG.md`
- **Workflow/convention/SPEC-engineering changes (no code)** →
  `multi-agent/DEVLOG.md`

Brainstorming-stage and SPEC-engineering-stage cycles (Stages 1-3) are
typically dev-system → DEVLOG. Cycle execution (Stage 5) cycles that
touch product code typically → CHANGELOG.

**No CHANGELOG/DEVLOG entry is written during stage iteration** — only
at cycle closeouts. **Intermediate git commits ARE welcome** at logical
checkpoints during iteration (e.g., between BRAINSTORM rounds, between
audit and implementation). Intermediate commits use the same
`changelog-entry-<topic>.txt` repo-root scratch-file convention as
closeout commits, but the scratch file does NOT get inserted into
CHANGELOG/DEVLOG — only the cycle-closeout commit fossilizes a LOG
entry. The intermediate scratch file is just a verbose commit message
for git history.

The two LOG files share a single `vR.X.YY` version stream; a version
appears in exactly one file. See `multi-agent/AGENT_CONVENTIONS.md §
CHANGELOG.md and DEVLOG.md` for the full routing rule.

## The eight stages at a glance

Stage 1: **SOUP.** Pre-phase ideas collection in `plans/next/<THEME>_SOUP.md`.

Stage 2: **BRAINSTORM iteration.** Per-phase BRAINSTORM authored from a
labeled SOUP; iterative refinement via FEEDBACK.

Stage 3: **SPEC engineering.** SPEC derived from BRAINSTORM with
substantive Priorities; iteration via FEEDBACK; SPEC locked at end of
stage.

Stage 4: **Strategist kickoff (Orchestrator's kickoff act).** STRATEGY.md
produced from SPEC; cycle table + agent assignments.

Stage 5: **Cycle execution (audit-implement-reaudit).** R1/R2/R3 rounds
across all cycles in the phase; one CHANGELOG/DEVLOG entry per cycle
close.

Stage 6: **Final Overseer.** End-of-phase independent wrap-up audit;
post-wrap-up remediation loop if actionable findings.

Stage 7: **Succession Briefing.** Outgoing Orchestrator writes wisdom
transfer for next-phase Orchestrator.

Stage 8: **Phase Archive.** Principal `git mv`s all phase artifacts to
`plans/archived/`.

## Required inputs / variable definitions (cycle execution)

When a Stage-5 cycle round (R1/R2/R3) is dispatched to an agent — whether
via Templates A/B/C or via an Orchestrator-customized prompt — the
following variables must be supplied or confirmable from context. These
formalize what the Orchestrator embeds into the per-cycle prompt; the
Principal need not handle them directly under v4 because the Orchestrator
drafts prompts.

1. `TARGET_FILE` — the SPEC file that is the authoritative working
   contract. Default: `multi-agent/plans/PHASE<N>_SPEC.md` (or
   `PHASE<N>_SUPPLEMENTAL-SPEC.md` for supplemental phases).
2. `AUDIT_LOG_FILE` — the sibling AUDIT_LOG. Default: if `TARGET_FILE` is
   `PHASE<N>_SPEC.md`, then `AUDIT_LOG_FILE` is the sibling
   `PHASE<N>_AUDIT_LOG.md` (analogous for supplemental phases).
3. `STRATEGY_FILE` — `PHASE<N>_STRATEGY.md` (the Stage 4 plan-of-attack
   doc; new in v3 / carried forward in v4).
4. `TARGET_CYCLE` — the cycle being worked in this round. Either:
   - one substantive priority (see § 3 Substantive priorities), OR
   - a named batched group of thin priorities (e.g., "Phase 1 renames"
     covering 14-S4 + 14-S5 + 14-S29 + 14-S30).
5. `UPDATE_FILES` — files that must be synced at cycle boundaries.
   Default set for onionskin:
   - `CHANGELOG.md` or `multi-agent/DEVLOG.md` — at cycle closeout only,
     routed by primary content
   - `multi-agent/project_context/HANDOFF.md` — at any state change
   - `multi-agent/project_context/TASK.md` — at any state change
   - `AUDIT_LOG_FILE` — every role-round
6. **Round type** — one of:
   - initial audit (R1)
   - implementation (R2)
   - re-audit (R1)
   - skip-reaudit declaration (R2 only)
   - final wrap-up audit (R3 — Final Overseer)
   - post-wrap-up triage
   - post-wrap-up implementation
   - post-wrap-up closeout
7. **Mode** — 2-role mode or 3-role mode (see § 15).

Suggested assignment-line scaffold the Orchestrator embeds at the top of
each cycle prompt (substitute concrete filenames for supplemental
phases):

```text
Below N and <N> = <phase number to be filled in by user>

You are acting as {ROLE_NAME} from multi-agent/workflows/phase-development-system-v4.md.
Spec file: multi-agent/plans/PHASE<N>_SPEC.md
Audit log file: multi-agent/plans/PHASE<N>_AUDIT_LOG.md
Strategy file: multi-agent/plans/PHASE<N>_STRATEGY.md
Surprise log file: multi-agent/plans/PHASE<N>_SURPRISE_LOG.md
Target cycle: <cycle name> or <cycle index>
Current round: {ROUND_TYPE}
```

(Carried forward from `audit-loop-v3.md § Required Inputs` and
generalized for v4's single-file system.)

---

# 2. The Orchestrator role

**One-line summary:** the Orchestrator is an AI agent (recommended: Claude
Code Opus Max) that initializes via the Strategist function (Template I;
phase strategy doc; Stage 4) and continues across the whole phase as an
orchestration layer between the Principal (human; phase owner / scope
authority) and the audit-loop agents (R1/R2/R3). The Orchestrator role
**absorbs the prior Strategist function** — Strategist is no longer a
one-shot role but the kickoff act of the Orchestrator's lifecycle when
the phase transitions from Stage 3 (SPEC engineering) to Stage 4
(Strategist kickoff) to Stage 5 (cycle execution).

The Orchestrator does NOT replace the Principal's role as scope authority.
Per `multi-agent/AGENT_CONVENTIONS.md § Scope authority`, the Principal is
the sole decision-maker on scope. The Orchestrator surfaces options +
drafts artifacts; the Principal authorizes, refines, or redirects.

## Why the role exists

- **Cross-cycle pattern memory.** Phase 15's GAP-1 pattern (cycle
  15.4a-S3 Copilot R2 missed the structural parity table; R3 had to
  remediate inline) is exactly the kind of recurring blind spot a
  phase-spanning Orchestrator flags pre-emptively. Without the
  Orchestrator, each cycle's R1/R3 must rediscover patterns the prior
  cycle already surfaced.
- **Buffer the Principal.** R1/R2/R3 are typically launched in fresh
  chat sessions. The Orchestrator carries phase context so the Principal
  does not have to re-bootstrap context with every audit-loop agent.
- **Catch what falls between roles.** Authorship/Effort attribution
  drift, scope drift, help-text omissions, mid-cycle scope shifts —
  these don't belong inside any single audit-loop role's deliverable but
  accumulate across the phase. The Orchestrator owns them.
- **Translate empirical results back to design.** When R2's
  implementation produces unexpected results (e.g., cycle 15.4a-S4's GMM
  `learn` mode artifacts), the Orchestrator helps interpret + recommends
  pivots before R3 begins, often via orchestrator-outside-cycle
  clarifications appended to the audit log under append-only discipline.
- **Reduce per-agent context cost.** Audit-loop agents in fresh chats
  can start clean without inheriting earlier-cycle deliberation; the
  Orchestrator carries that and feeds only what's relevant via prompts +
  audit-log clarifications.

## Lifecycle — mapped to v4's eight stages

The Orchestrator role spans the full phase across the v4 eight-stage
spine:

**Stages 1-3 (Brainstorm + SPEC engineering).** Orchestrator joins at
Stage 1 (SOUP) when a candidate future-phase SOUP file is being shaped
or brought onto the stage as the next-phase BRAINSTORM. The Orchestrator
drives prompts to brainstorming-stage agents (Agent 1 writer + Agent 2
auditor) per the upstream-stage templates in Section 14, deliberates
with the Principal on scope + naming + decisions, and helps shape the
SPEC through engineering rounds. The role is conceptually continuous
across these stages even when the chat instance changes (see § Multi-chat
continuity below).

**Stage 4 (Strategist kickoff = Orchestrator's kickoff act).** At the
SPEC-locked-to-cycle-execution transition, the Orchestrator initializes
its formal cycle-execution mode by producing the phase strategy doc at
`multi-agent/plans/PHASE<N>_STRATEGY.md` per **Template I — Strategist:
Phase Plan of Attack** (Section 14). STRATEGY.md splits the phase into
execution-ordered cycles, assigns primary + alternative agents per role
per cycle, and states deviations from the baseline strategy. This is
the Orchestrator's first formal cycle-execution deliverable. Run once
per phase; amend (never rewrite) as the phase progresses.

**Stage 5 (cycle execution).** Across all cycles in the phase, the
Orchestrator:

- Drafts R1/R2/R3 launcher prompts and mid-flight pushback notes (rich
  prompts with locked-contract checklists, validation gate lists, scope
  boundaries, pushback discipline). The Principal launches the prompts
  in fresh chats.
- Pre-checks R2 output before R3 launches (catches issues cheaply;
  cycle 15.4a-S3 GAP-1 was caught this way).
- Authors orchestrator-outside-cycle clarifications when scope shifts
  mid-cycle. These are appended to the cycle's audit-log section under
  append-only discipline (e.g., cycle 15.4a-S3 corrections appendix;
  cycle 15.4a-S4 FIRST + SECOND clarifications).
- Captures decision history in `multi-agent/plans/PHASE<N>_BACKGROUND.md`
  (per-phase planning scratchpad; promoted from `<theme>.tmp.md` when
  deliberation captures non-trivial decision history worth preserving).
- Updates STRATEGY.md with mid-phase amendments + amendment-log entries
  when cycle ordering, scope, or assignments change.
- Updates SPEC.md when scope decisions affect priorities (rare;
  typically via SPEC priority status table changes at cycle closeout).
- Triages SURPRISE_LOG entries at cross-cycle level (deciding whether a
  surprise authorizes a supplemental cycle or stays Active for
  future-phase pickup).
- Authors DEVLOG entries for planning-surface engineering and
  orchestrator-housekeeping commits (e.g., Effort: Max sweeps, SOUP
  additions, mid-cycle clarifications). DEVLOG entries follow the
  routing rule in `AGENT_CONVENTIONS.md`.
- Drafts commit messages per `feedback_commit_message_file_convention.md`
  (scratch file at repo root as `changelog-entry-<version>.txt`).
- Maintains authorship/Effort consistency across artifacts (catches
  drift; applies retroactive corrections as a discrete commit).
- Adds SOUP entries (`multi-agent/plans/next/<THEME>_SOUP.md`) for
  deferred future-phase work surfaced during cycle execution.
- Translates empirical results back to design decisions through
  deliberation with the Principal (see § Deliberation pattern below).

**Stages 6-8 (wind-down).** After the final R3 closes the last cycle:

1. **Stage 6 — Final Overseer pass.** Final Overseer (separate
   fresh-chat role) does the end-of-phase independent review. **The
   Orchestrator does NOT become the Final Overseer** — Final Overseer's
   value is structural independence from the Orchestrator's accumulated
   frame. If post-wrap-up remediation is needed (Templates E/F/G), the
   Orchestrator drafts those launchers but Final Overseer's wrap-up
   audit itself is independent.
2. **Stage 7 — Succession Briefing.** After Final Overseer signs off,
   the outgoing Orchestrator writes the **Succession Briefing** at
   `multi-agent/plans/PHASE<N>_SUCCESSION_BRIEFING.md` per **Template J
   — Succession Briefing** (Section 14). The briefing transfers wisdom
   (cross-cycle pattern memory, recurring blind spots, deliberation
   patterns that worked, things that didn't, brainstorm seeds, open
   cross-phase entries, recommended working style) to the next-phase
   Orchestrator. The briefing is NOT a recreation of the formal
   artifacts (SPEC/AUDIT_LOG/STRATEGY/BACKGROUND/SURPRISE_LOG); those
   archive intact.
3. **Stage 8 — Phase Archive.** SPEC + AUDIT_LOG + STRATEGY + BACKGROUND
   + SURPRISE_LOG + SUCCESSION_BRIEFING all move to
   `multi-agent/plans/archived/<date>-PHASE<N>_*.md`. Archive operations
   are Principal-run per
   `feedback_archive_operations_orchestrator_only.md` (Orchestrator
   proposes the `git mv` in commit blocks but does not execute).
4. **Outgoing Orchestrator's chat closes.** Next-phase Orchestrator
   (recommended: fresh chat, possibly different AI instance) bootstraps
   by reading the archived prior-phase artifacts + Succession Briefing.

## Entry point: starts at SOUP

**The Orchestrator role starts at SOUP — when a candidate future-phase
SOUP file (`multi-agent/plans/next/<THEME>_SOUP.md`) is being shaped or
when the Principal is bringing a SOUP onto the stage as the next-phase
BRAINSTORM.** This is the **default entry point**. Starting at SOUP
gives the Orchestrator the strongest cross-cycle pattern memory because
the deliberation patterns that emerge in brainstorming + SPEC
engineering carry through to cycle execution.

**Late entry is acceptable.** If circumstances require an Orchestrator
to join at the Strategist stage (or even mid-cycle-execution), the
incoming chat bootstraps from the existing artifacts (SPEC + BRAINSTORM
+ SOUP + recent audit-log entries + feedback memories). Some pre-cycle
deliberation context is lost; the SPEC contract is inherited cleanly.
This is suboptimal but workable when the long form isn't feasible.

The Principal chooses the entry point. Phase 15 used a hybrid (some
Orchestrator-like activity from BRAINSTORM stage onward across multiple
chats; formal Strategist-onward activity from v0.14.76.5 SPEC closeout
through to phase end).

## Multi-chat continuity within a single Orchestrator role

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

1. **Memory files** at `~/.claude/projects/<project>/memory/`
   (auto-memory) plus project-specific memory in
   `multi-agent/project_context/` and feedback memories under the
   agent's persistent memory store.
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

## Deliberation pattern (Principal ↔ Orchestrator)

The Orchestrator is not a passive context buffer. The role works
through **bidirectional deliberation** with the Principal that actively
shapes scope evolution mid-phase.

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
  Orchestrator flags it (e.g., "the empirical defaults are
  dataset-overfit; biology priors are more robust"). The Principal may
  accept, reject, or refine.
- **Mid-flight pushback notes.** When an audit-loop agent is hitting
  unexpected results, the Orchestrator drafts a pushback note + the
  Principal injects it into the running agent's chat.
- **Empirical results trigger pivots.** When Stage A diagnostic results
  surface that the locked algorithm doesn't fit reality, the
  Orchestrator helps interpret + drafts orchestrator-outside-cycle
  clarifications. The Principal authorizes the pivot.
- **Scope decisions ALWAYS authorized by Principal.** Per
  `multi-agent/AGENT_CONVENTIONS.md § Scope authority`, narrowing is
  prohibited without Principal approval; broadening is encouraged but
  surfaced and acknowledged. The Orchestrator does not silently descope
  or expand.

This deliberation pattern produces scope evolution mid-phase that's
disciplined (recorded in BACKGROUND.md + audit-log clarifications) but
responsive (not locked by R1's initial audit alone). The role only
works if the deliberation is bidirectional — a passive context-buffer
would not catch what's surfaced through these exchanges.

## Boundaries (what the Orchestrator does NOT do)

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
- **Does not act as Final Overseer.** Final Overseer must be
  structurally independent from the Orchestrator's accumulated frame.
  The wind-down Succession Briefing (Stage 7) is the Orchestrator's
  last act; Final Overseer (Stage 6) precedes it as a fresh-chat
  independent review.
- **Does not override audit-log append-only discipline.** Corrections
  appendices APPEND; never edit prior R1/R2/R3 entries. The trajectory
  (R1 audit → corrections appendix → R2 implementation → R3 closeout)
  is preserved as decision history.
- **Does not bypass the Principal's authorization for design pivots.**
  Even when the Orchestrator identifies a clear flaw (e.g., GMM `learn`
  mode artifacts), the pivot lands only after Principal authorization.

## Interaction model

**With Principal (human; scope authority):**

- Principal feeds Orchestrator the chat output + audit log entries
  from audit-loop agent sessions.
- Orchestrator integrates, identifies what landed/missed, flags drift.
- Principal asks Orchestrator for pre-checks before R3 launches to
  catch issues cheaply.
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
- Final Overseer reads the **live phase artifacts** (SPEC, AUDIT_LOG,
  STRATEGY, BACKGROUND, SURPRISE_LOG) + reaches its own conclusions
  independently. The phase artifacts are NOT yet archived at Stage 6 —
  archive happens at Stage 8 after Succession Briefing (Stage 7).
- After Final Overseer signs off, Orchestrator writes the Succession
  Briefing incorporating Final Overseer's findings.

## Recommended model + agent choice

- **Claude Code Opus 4.7 — Effort: Max** is the recommended primary.
  Reasoning quality + long-context handling matter more than speed.
- **Codex GPT-5.5 (Reasoning: Extra High)** is acceptable as
  alternative.
- **Gemini is excluded** per project memory
  `feedback_role_selection_priors.md` (historical thin/shallow audits;
  once destroyed the codebase during implementation).
- **Github Copilot** is acceptable for narrow R2 implementation roles
  but is NOT recommended for the Orchestrator role — its working style
  (target-and-go) doesn't match the orchestration role's cross-cycle
  pattern recognition + bidirectional deliberation requirements.

## Decision guide for cycle transitions

When uncertain what prompt to draft next during cycle execution
(Stage 5), use this decision tree (Principal-Orchestrator joint
decision; Orchestrator drafts, Principal authorizes):

1. **New cycle starts** → draft Auditor Template A launcher (Role 1
   initial audit).
2. **Auditor wrote instructions** → draft Implementer Template B
   launcher (Role 2 implementation).
3. **Implementer finished with "re-audit needed" declaration** → draft
   Auditor Template C launcher (Role 1 re-audit).
4. **Implementer finished with "skip-reaudit recommended"**:
   - If Principal ACCEPTS the skip → draft Auditor Template H launcher
     (Role 1 cycle closeout after skip-reaudit).
   - If Principal REJECTS the skip (force full re-audit) → draft
     Auditor Template C launcher instead.
5. **Cycle closed, next cycle queued** → draft Auditor Template A
   launcher for the next cycle (or Template H already covers this
   transition if skip-reaudit was accepted).
6. **All cycles closed** → draft Final Overseer Template D launcher
   (Stage 6 begins).
7. **Final Overseer found actionable work** → draft Auditor Template E
   launcher (triage into post-wrap-up remediation cycle).
8. **Auditor triaged findings** → draft Implementer Template F
   launcher.
9. **Post-wrap-up implementation finished** → draft Auditor Template G
   launcher (final closeout).
10. **Final Overseer signed off + post-wrap-up remediation closed** →
    Outgoing Orchestrator writes the Succession Briefing (Stage 7) per
    Template J. Then Phase Archive (Stage 8) — Principal runs `git mv`.

### Deciding whether to accept a Role 2 skip-reaudit

Default to **NO skip** (force full re-audit) when any of these apply:

- The cycle involves design judgment or help-string quality.
- The cycle is a substantive single-priority cycle (not a thin-priority
  batched cycle).
- The cycle touches runtime wiring, not just parser surface.
- Tests did not run green on the first try.
- The Principal-Orchestrator pair is new to v3/v4; build trust in the
  skip criteria before using them liberally.

Default to **YES, accept skip** only when ALL skip criteria are met
AND the cycle is a batched mechanical cycle (renames, help-string
appends with no judgment, flag additions with clear spec, etc.).

Even when accepting a skip, Role 1 still does a lightweight
spot-check at closeout (Template H). That's the backstop.

## What gets archived at phase close (Stage 8 inputs)

At phase close, the following move to
`multi-agent/plans/archived/<date>-PHASE<N>_*`:

- `PHASE<N>_SPEC.md`
- `PHASE<N>_AUDIT_LOG.md`
- `PHASE<N>_STRATEGY.md`
- `PHASE<N>_BACKGROUND.md`
- `PHASE<N>_SURPRISE_LOG.md`
- `PHASE<N>_SUCCESSION_BRIEFING.md` (NEW under v3+)
- Any per-phase `BRAINSTORM.md` or `FEEDBACK.md` if not already
  archived (they typically archive at end of Stage 3)

Archive operations are **Principal-run** per project memory
`feedback_archive_operations_orchestrator_only.md`. The Orchestrator
emits the `git mv` operations in a commit block; the Principal runs
them. (The memory's "orchestrator-only" naming refers to the v2
human-orchestrator framing; under v3+ the AI Orchestrator drafts and
the Principal executes.)

---

# 3. Substantive priorities + cycle granularity

This is a v2-introduced concept that v3 + v4 retain. **Do not run a full
audit-implement-reaudit loop on single-line flag renames or trivial
help-string tweaks.** Bundle thin work into substantive cycles.

## Substantive priority

A priority is substantive when any one of these is true:

- It plausibly requires multi-file implementation across 3+ surfaces.
- It involves design judgment (help-string quality bar, terminology
  harmonization, cross-pipeline framing).
- It touches runtime wiring (not just parser surface).
- A Role 1 audit would reasonably produce 2+ concrete findings.

Substantive priorities get their **own cycle**: one Role 1 audit → one
Role 2 implement → one Role 1 re-audit (or Role 2 skip-reaudit
declaration) → closeout.

## Thin priority

A priority is thin when all of these are true:

- Implementation is a single rename, single help-string edit, or single
  trivial wire.
- No ambiguity in what the final state should be.
- A Role 1 audit would produce ≤1 concrete finding (essentially just
  "do what the priority says").
- Testing surface is covered by the existing test suite without new
  fixtures.

Thin priorities **must be batched** into a named batched cycle with
related work. Common batch units:

- A phase group (e.g., Phase 1 of a reordered SPEC).
- A theme group (e.g., "all the audit-only deliverables").
- A single Role 2 round spans the whole batched cycle.

Rule of thumb: **prefer 5-10 substantive cycles per phase over 30 thin
cycles.** The three-role loop is expensive; every cycle should justify
the cost.

## Who decides substantive vs thin

- When the SPEC is engineered (Stage 3), the SPEC's phase/group
  structure usually implies the cycle granularity. See § Phase-group
  SPEC structure below.
- At Role 1 initial audit, Role 1 may propose consolidation ("these
  three priorities are thin — recommend batching"). Orchestrator +
  Principal authorize.
- Never batch priorities across phase boundaries. Phase structure
  encodes hard dependencies.

## What to do with an existing SPEC that has too many thin priorities

If a SPEC was engineered with many thin priorities (common in older
phases), at Stage 4 (Strategist kickoff) the Orchestrator + Principal
may consolidate them into batched cycles in STRATEGY.md without
modifying the SPEC. The SPEC priority status table tracks each
priority's individual close state; the CHANGELOG entry covers the
whole batch. See § Phase-group SPEC structure for the recommended
structure for new SPECs.

## Phase-group SPEC structure (optional but recommended for new phases)

When a phase has >10 priorities, organize them into phase-groups in
the SPEC with dependency-ordered H2 headings. Example: Phase 14
Supplemental was reorganized into Phase 0 / 1 / 2 / 3 / 4 / 5 sub-groups,
each containing related priorities. This makes batched cycle
granularity natural at Stage 4 — each phase-group can become one
batched cycle in the audit loop, with the cycle name in the AUDIT_LOG
literally tracking the phase-group label.

Skeleton:

```markdown
## Phase 0 — Closed priorities

... (closed priorities, kept for cross-reference)

## Phase 1 — Structural parser changes

*Implementation order: P1 → P2 → P3 → ...*
*Rationale: all structural changes land first so the CLI surface is final
before help-string polish (Phase 3) touches it.*

... (priorities in dependency order)

## Phase 2 — Audit-only deliverables

... (priorities)

## Phase 3 — Help-string polish

... (priorities)

## Phase 4 — Tooling

... (priorities)

## Phase 5 — Closeout

... (priorities)
```

The cycle name in `PHASE<N>_AUDIT_LOG.md` can literally be "Phase 1 —
Structural parser changes" covering all the priorities in that
phase-group. Priority identifiers (`SPEC<N>.<idx>`) stay stable across
the reorganization. (Carried forward from `PDS-v3 § Phase-group SPEC
structure`.)

## Deferred R3 (bundled into next cycle's R3)

Some cycles do not merit a standalone Role 3 closeout round — typically
audit-only cycles that produced tracking documents or appendices with
no runtime code changes. Rather than run a full R3 session that
spot-checks trivial deliverables, the cycle may **defer its R3 scope
to the next cycle's R3 auditor**.

Rules for deferring R3:

1. **Eligibility.** A cycle may defer R3 only when its primary output
   is documentation or audit artifacts and it contains no runtime code
   changes (or only trivial help-string edits already covered by R1
   initial audit). Substantive runtime-wiring cycles MUST run their
   own R3.
2. **Closeout still happens.** The deferring cycle still closes with
   its own CHANGELOG/DEVLOG entry and its own priority-status-table
   update. R1 performs a lightweight closeout (Template H-style
   skip-reaudit flow), then records in the cycle-closeout entry that
   "R3 scope deferred to cycle <absorbing cycle>."
3. **Absorbing cycle's R3 scope grows.** The next cycle's R3 auditor
   adds "Deferred verification from cycle <X>: <one-line scope>" to
   their closeout scope. They re-check the deferred cycle's
   deliverables before signing off.
4. **Naming in STRATEGY.md.** The deferring cycle's R3 cell must
   explicitly name the absorbing cycle (e.g., `R3: Defer to 14S.3a R3`).
   The absorbing cycle's Notes column must mirror the deferral (e.g.,
   `R3 also verifies 14S.2a deferred deliverables`).
5. **No chaining.** A cycle may only defer to the immediately-next R3.
   The absorbing cycle's R3 itself cannot be deferred.
6. **Final-cycle edge case.** If a deferring cycle is the last cycle
   before the Final Overseer pass, the Final Overseer absorbs the
   deferred scope instead. Call this out explicitly in STRATEGY.md.

---

# 4. Stage 1: SOUP

Stage 1 is pre-phase ideas collection. SOUP files are the **primordial
soup** of ideas around a theme — informal, theme-named, never numbered.

## SOUP file conventions

**Pre-promotion location:** `multi-agent/plans/next/<THEME>_SOUP.md`

- Filename uses a vague / broad / lightly-evocative theme prefix +
  `_SOUP.md` suffix.
  - Examples: `HMM_SOUP.md`, `SUMMIT_SOUP.md`, `FWD-ARCH_SOUP.md`.
  - **NEVER scope-anchoring** (`HMM_SOUP`, not `HMM_COMPLETENESS_SOUP`).
  - **NEVER numbered** (`HMM_SOUP`, not `PHASE15_SOUP`).
- The SOUP body MUST NOT contain SELF-references to phase numbers while
  in `next/`. No "this is the Phase X plan", no "Phase X follow-up",
  no rename-history narrative naming the SOUP's own past
  `PHASE<N>_*.md` filenames. Rename history is preserved by `git log
  --follow` — doesn't need to be in the body.
- Cross-references to actually-closed phases ARE allowed as historical
  anchors.
- The self-reference ban makes "rename to renumber" a zero-content-edit
  operation.

**Post-promotion location:** `multi-agent/plans/PHASE<N>_<THEME>_SOUP.md`

- The same file is moved here (via `git mv`) at the moment the
  Orchestrator decides to promote the theme to be the next phase's
  starting point.
- The phase number `<N>` is committed at this move.
- The SOUP receives a **one-time SOUP ID labeling pass** (every
  discrete entry tagged with `SOUP<N>.<idx>`) at this same moment —
  this is the **only** allowed body modification.
- After the labeling pass, the SOUP body is **read-only forever**;
  the labeled file serves as the immutable
  source-of-truth-for-provenance during BRAINSTORM construction.
- The SOUP stays live in `plans/` (alongside the BRAINSTORM that's
  being authored from it) until the soup-to-brainstorm transfer is
  verified complete (Stage 2 closeout).

See `multi-agent/AGENT_CONVENTIONS.md § Future-phase planning surfaces`
for the canonical rules.

## Promotion process (SOUP → live `plans/` SOUP → BRAINSTORM seed)

When the Orchestrator + Principal decide a SOUP is the next thing to
formalize:

1. Move the SOUP via `git mv multi-agent/plans/next/<THEME>_SOUP.md multi-agent/plans/PHASE<N>_<THEME>_SOUP.md` (phase number `<N>` is committed here).
2. Perform the one-time SOUP ID labeling pass on the SOUP body — adds
   `SOUP<N>.<idx>` tags to each discrete entry. After this pass, the
   SOUP body is read-only forever.
3. Author `PHASE<N>_BRAINSTORM.md` from the labeled SOUP, assigning
   `BRAIN<N>.<idx>` BRAIN IDs to each BRAINSTORM entry and citing SOUP
   IDs in `Source:` fields. (This step is the start of Stage 2.)
4. Seed `PHASE<N>_FEEDBACK.md` with the first entry (open questions,
   line-items-for-approval, transfer-status block).
5. (Iterative) Multi-pass refinement via FEEDBACK: agent flags
   questions, Principal answers, agent does a follow-up pass updating
   BRAINSTORM with back-references in FEEDBACK. Repeats until transfer
   is faithful and complete.
6. Auditor (Agent 2) verifies the transfer per the auditor prompt
   (Section 14 templates): every SOUP ID is referenced in at least one
   BRAIN ID's `Source:`, no silent scope drift, etc. Verdict appended
   to FEEDBACK.
7. **Once the auditor declares CLEAN**, the SOUP file is archived:
   `git mv multi-agent/plans/PHASE<N>_<THEME>_SOUP.md multi-agent/plans/archived/<YYYYMMDD>-PHASE<N>_<THEME>_SOUP.md`.

The archive filename carries both lineages: `PHASE<N>` links to the
phase identity, `<THEME>` preserves the SOUP's theme name from its
`next/` era, `<YYYYMMDD>` is the archival date. SOUP archival happens
AFTER the soup-to-brainstorm transfer is verified, not at the start of
BRAINSTORM authoring — this preserves SOUP read-accessibility during
the (often iterative) transfer.

## Stage transition out

The official BRAINSTORM iteration stage (Stage 2) begins after SOUP
archival.

---

# 5. Stage 2: BRAINSTORM iteration

Stage 2 is open-for-expansion BRAINSTORM iteration. After SOUP archival,
the per-phase BRAINSTORM is open for new ideas, deeper analysis, and
additional sourcing from `tracking/BRAINSTORM.md` and
`tracking/KNOWN_ISSUES.md`.

## BRAINSTORM file conventions

**Location:** `multi-agent/plans/PHASE<N>_BRAINSTORM.md`

- Authored from the labeled SOUP (lives in `plans/`, sibling to the
  SOUP during the transfer).
- Each entry receives a `BRAIN<N>.<idx>` BRAIN ID and a `Source:` field
  citing the SOUP IDs it covers, plus any other source citations:
  - `tracking/BRAINSTORM.md` long-lived entries
  - `tracking/KNOWN_ISSUES.md` `[ISSUE:YYYY-MM-DD:N]` IDs
  - prior archived phase plans (e.g., `BRAIN15.5 Source: SOUP15.3,
    archived/20250628-PHASE12_BRAINSTORM.md § X`)
- BRAINSTORM is more organized than SOUP: themed sections, identified
  candidate priorities, open questions hoisted into a `## Open
  questions` block.
- Not necessarily in final-implementation order; that's the SPEC's job.
- Phase number is fully committed at this point.

## FEEDBACK file conventions

**Location:** `multi-agent/plans/PHASE<N>_FEEDBACK.md`

FEEDBACK is the audit-log analog for the engineering stages (Stages 2 +
3). Where human and agent developers leave feedback:

- **First** on the SOUP-to-BRAINSTORM transition (the BRAINSTORM-from-
  SOUP authoring step **triggers the first FEEDBACK entry** — that's
  the canonical starting point for FEEDBACK; typically the first entry
  contains agent open-questions, line-items-for-approval, and a
  transfer-status block).
- **Then** on BRAINSTORM as it develops in Stage 2.
- **Later** on SPEC as it develops in Stage 3 from BRAINSTORM.

FEEDBACK uses `Q<idx>` / `JQ<idx>` for question identifiers; it does
not introduce its own provenance-IDs.

## Multi-pass iteration pattern

Stage 2 uses an Agent-1-writer + Agent-2-auditor + Orchestrator-coordinator
pattern:

1. **Agent 1 (writer)** authors / extends the BRAINSTORM, drawing on
   the labeled SOUP + FEEDBACK Q's + the Principal's direction.
2. **Agent 2 (auditor)** reviews the BRAINSTORM against SOUP coverage
   + scope intent + clarity. Files findings + Q's into FEEDBACK.
3. **Agent 1 follow-up** addresses Agent 2's audit + Principal's Q
   answers in a follow-up BRAINSTORM pass. Updates back-references in
   FEEDBACK.
4. **Orchestrator deliberates with Principal** on convergence — when
   is BRAINSTORM "done enough" to start SPEC engineering?
5. Repeat until convergence.

See Section 14 for the upstream-stage prompt templates (0a-0h).

**Discipline rules apply.** Stage 2 agents (writer + auditor) follow
the **Phase Development discipline rules** documented under § 8 Stage
5 "Phase Development discipline rules (stale-context handling)." The
rules apply to every stage and every role, not just cycle-execution
roles — the placement under § 8 is for locality of reference (cycle
execution is where the discipline matters most), but Stage 2 agents
must follow the same re-read / show-snippet / only-trust-current-file
/ post-change-verify discipline. The "Be thorough during BRAINSTORM
and SPEC engineering" subsection of § 8 is specifically tuned for this
stage.

## Stage transition out

Stage 3 (SPEC engineering) opens when BRAINSTORM is sufficiently
developed (Principal authorizes). BRAINSTORM stays live as a reference
during Stage 3.

---

# 6. Stage 3: SPEC engineering

Stage 3 is the conversion of BRAINSTORM into a structured SPEC with
substantive Priorities. The SPEC is the **authoritative contract** for
the audit-implement-reaudit cycles in Stage 5.

## SPEC file conventions

**Location:** `multi-agent/plans/PHASE<N>_SPEC.md`

The culmination of BRAINSTORM and FEEDBACK — the ideas mapped to the
most sensible ordering and organized into **substantive Priorities**.

- Each Priority has a `SPEC<N>.<idx>` ID (e.g., `SPEC15.1`,
  `SPEC15.2`).
- Each Priority has a `Source:` / `Covers:` field citing the BRAIN IDs
  it covers (one, many, or none — Priorities authored from new
  SPEC-stage thinking cite the FEEDBACK discussions that produced them).
- Priorities must be **substantive** (see § 3 above for the
  substantive-vs-thin distinction). Thin priorities should be batched
  into substantive groups during SPEC engineering, not deferred to
  Stage 4.
- Phase-group structure (Phase 0 / 1 / 2 / etc.) is recommended for
  SPECs with many priorities — eases batched-cycle granularity at
  Stage 4.

## SPEC ID freezing rule

SPEC IDs are **reorderable during SPEC engineering**. Once the SPEC is
locked at the start of cycle execution (Stage 4 kickoff), IDs freeze
and inserts get the next-available number even if it appears "out of
order" in the file's reading sequence. This preserves cross-reference
stability across the phase.

## SPEC engineering rounds

Same multi-agent pattern as Stage 2:

1. **Agent 1 (writer)** drafts an initial SPEC from BRAINSTORM,
   organizing entries into substantive Priorities with dependency
   ordering.
2. **Agent 2 (auditor)** reviews the SPEC against BRAINSTORM coverage
   + ordering rationale + substantive-priority discipline. Files
   findings into FEEDBACK.
3. **Agent 1 follow-up** addresses audit + Principal direction in
   updated SPEC.
4. **Orchestrator deliberates with Principal** on whether SPEC is
   complete + whether priorities are substantive enough.
5. Repeat until SPEC engineering closes.

See Section 14 for SPEC-engineering prompt templates (0i-0l).

**Discipline rules apply.** Stage 3 agents (writer + auditor) follow
the **Phase Development discipline rules** documented under § 8 Stage
5 "Phase Development discipline rules (stale-context handling)." Of
particular importance for SPEC engineering: the "Be thorough during
BRAINSTORM and SPEC engineering" subsection (CLI flag organization
rules, universal vs pipeline-specific overrides, search markdown +
code + KNOWN_ISSUES + BRAINSTORM) and the "Aim for substantive
priorities" subsection (cross-reference to § 3 Substantive priorities
+ cycle granularity).

## SPEC engineering closeout

When the SPEC engineering closes:

- BRAINSTORM is moved to
  `multi-agent/plans/archived/<YYYYMMDD>-PHASE<N>_BRAINSTORM.md`.
- FEEDBACK is moved to
  `multi-agent/plans/archived/<YYYYMMDD>-PHASE<N>_FEEDBACK.md`.
- For both, the `<YYYYMMDD>` prefix is the date the SPEC is finished.
- The SPEC stays live going into Stage 4.
- The sibling `multi-agent/plans/PHASE<N>_AUDIT_LOG.md` is created at
  this point (SPEC lock — Stage 3 closeout) with a skeleton header.
  This is the **only allowed creation point** under v4 (no Stage 5
  fallback) so that the artifact's existence is deterministic by the
  time Stage 4 / Stage 5 begin. See § 1 lifecycle artifact-table and
  § 16 startup checklist for the matching canonical specification.

  ```markdown
  # PHASE <N> AUDIT LOG

  Sibling to `multi-agent/plans/PHASE<N>_SPEC.md`. Captures
  round-by-round history of the audit-implement-reaudit loop for this
  phase under the v4 phase-development workflow.

  ## Cycle: <first cycle name> — OPEN (awaiting Role 1 initial audit)
  ```

## Stage transition out

Stage 4 (Strategist kickoff) opens once the SPEC is locked. The
Orchestrator transitions from upstream-stages mode to cycle-execution
mode by producing STRATEGY.md per Template I.

---

# 7. Stage 4: Strategist kickoff (Orchestrator's kickoff act)

Stage 4 is the Orchestrator's transition from upstream-stages mode
(Stages 1-3: SOUP, BRAINSTORM, SPEC engineering) into cycle-execution
mode (Stage 5+). The Orchestrator's kickoff act is producing the phase
strategy doc.

## STRATEGY.md production

Before running the first audit cycle of the phase, the Orchestrator
produces a plan-of-attack document at `multi-agent/plans/PHASE<N>_STRATEGY.md`
(sibling to `PHASE<N>_SPEC.md` and `PHASE<N>_AUDIT_LOG.md`) using
**Template I — Strategist: Phase Plan of Attack** (Section 14). STRATEGY.md
splits the phase into execution-ordered cycles, assigns primary +
alternative agents per role per cycle, names a Final Overseer, and
states deviations from the baseline strategy.

This is the Orchestrator's first formal cycle-execution deliverable.
Run once per phase; the output drives Stage 5 cycle execution.

## STRATEGY.md is a living document during the phase

Amendments append with date + rationale to a `## Amendment log` section
at the bottom of STRATEGY.md. Never rewrite history. Common amendments:

- Cycle ordering changes (e.g., insert a supplemental cycle to address
  an `[ISSUE:YYYY-MM-DD:N]` or a `[SURPRISE:N:cycle:round:I:idx]` that
  surfaced mid-phase).
- Agent assignment changes (e.g., a Role 2 primary agent becomes
  unavailable; alternative is promoted).
- Scope changes per Principal authorization (must be reflected in
  SPEC.md priority status table too).

## Stage transition out

Stage 5 (Cycle execution) begins with the first R1 launcher (Template
A). The Orchestrator drafts that launcher per the cycle table in
STRATEGY.md.

---

# 8. Stage 5: Cycle execution (audit-implement-reaudit)

Stage 5 is the bulk of the phase. The Orchestrator drives one cycle at
a time through R1/R2/R3 rounds until all cycles close.

## Standard execution loop

**Step 1 — Initial Audit.** Role 1 audits the target cycle, determines
whether it is open or closed, writes findings and exact instructions
into the cycle section of `AUDIT_LOG_FILE` (not the SPEC). Role 1 does
NOT write to CHANGELOG in this round.

**Step 2 — Implementation.** After explicit Principal approval, Role 2
implements the audited open items, runs the justified validation,
appends an implementation report to the cycle section of `AUDIT_LOG_FILE`,
and updates HANDOFF.md + TASK.md. Role 2 does NOT write to CHANGELOG.
At round end, Role 2 declares either "re-audit needed" or "skip-reaudit
recommended (pending Principal approval)".

**Step 3 — Re-Audit OR Skip-Reaudit Closeout.**
- **If re-audit needed:** Role 1 reads the actual changed code and
  decides whether the cycle is now closed or still open. If still open,
  Role 1 writes new findings and instructions into the cycle section of
  AUDIT_LOG_FILE. If closed, Role 1 updates the SPEC priority status
  table and writes the cycle-closeout CHANGELOG entry with consolidated
  authorship.
- **If skip-reaudit accepted by Principal:** Role 1 performs lightweight
  spot-check (2-3 surfaces), then closes the cycle (SPEC priority status
  table update + CHANGELOG entry with consolidated authorship). Use
  Template H for the lightweight closeout flow.

**Step 4 — Iterate Until Closed.** Repeat Steps 2 and 3 until the cycle
is explicitly closed by an auditing role (or by Role 1 at skip-reaudit
closeout).

**Step 5 — Cycle Transition.** When one cycle is closed, move to the
next user-designated cycle and repeat the loop. The Orchestrator drafts
the next R1 launcher.

When all cycles are nominally closed, transition to Stage 6 (Final
Overseer pass).

## Role 1 — Auditor / Instruction Writer

Role 1 performs deep audits and writes the repair contract.

**Primary responsibilities:**

1. Read the target cycle scope from `TARGET_FILE` (the SPEC) and the
   surrounding context needed to understand its intent.
2. Audit the live code, tests, scripts, and documentation relevant to
   that cycle.
3. Determine whether the cycle is closed, partially complete, or still
   open.
4. If open, append audit findings and exact implementation instructions
   into the cycle's section of `AUDIT_LOG_FILE` (NOT the SPEC).
5. At cycle closeout: update the SPEC's priority status table. Write
   the cycle's CHANGELOG entry using the authorship consolidation rule.
6. Re-audit Role 2's implementation by reading the actual changed code
   rather than trusting the implementation report.
7. Close the cycle only after verifying that the findings are actually
   resolved.
8. If Role 3 identifies actionable wrap-up findings, review that
   wrap-up audit, decide which findings become active repair work, and
   translate them into concrete implementation instructions in a new
   cycle section of `AUDIT_LOG_FILE`.
9. After Role 2 completes any post-wrap-up remediation, perform the
   final closeout audit.

**Role 1 must NOT:**

- trust the implementation report without inspecting the code
- skip `scripts/` or `tests/` or documentation when they are plausibly
  affected
- silently narrow or broaden scope — scope changes belong in an
  explicit audit finding for the Principal to approve; never relegate
  in-scope work to KNOWN_ISSUES without asking the Principal first
- edit the SPEC body with audit findings; findings go to
  `AUDIT_LOG_FILE`

**Further clarification:**

- Looking for ways to broaden scope or improve the phase is allowable,
  but it must be raised to the Principal's attention for approval.
- It is best to assume the implementation was NOT exhaustive and that
  Role 1 can add value by looking for gaps and pushing back when needed.
- Praise to the implementer is encouraged when the implementer finds
  gaps in the audit or identifies clever ways to solve problems
  encountered in the code reality.

**Expected output from Role 1:**

- An audit subsection in `AUDIT_LOG_FILE` under the current cycle.
- Exact repair instructions with file/function references when possible.
- At cycle closeout: a `CHANGELOG.md` entry with consolidated authorship
  + a SPEC priority status-table update.
- An explicit close / still-open judgment after each re-audit.
- When needed, an auditor triage of Final Overseer findings and a final
  post-remediation closeout judgment.

## Role 2 — Implementer / Supplemental Auditor

Role 2 implements the audited instructions and performs in-situ
auditing while working.

**Primary responsibilities:**

1. Read the current audited cycle section in `AUDIT_LOG_FILE` and the
   related priority block(s) in the SPEC.
2. Implement only the audited open items unless code reality forces a
   documented deviation.
3. While editing, inspect surrounding code carefully enough to catch
   audit misses in the same neighborhood.
4. If a miss is found:
   - Fix it if it is clearly in scope and low-risk, then document it in
     the implementation report.
   - Otherwise flag it clearly in the implementation report.
5. Run the targeted validation justified by the changed surface.
6. Append an implementation report into the same cycle section of
   `AUDIT_LOG_FILE`.
7. Do NOT write to `CHANGELOG.md` in mid-cycle rounds. CHANGELOG is
   written once at cycle closeout by the closing agent (per the
   CHANGELOG batching rule).
8. Update `HANDOFF.md` and `TASK.md` to reflect the new state.
9. If the round comes from Role 3 follow-up findings, implement only
   the actionable subset that Role 1 has triaged into concrete
   instructions in `AUDIT_LOG_FILE`.
10. **At the end of the implementation round, decide whether re-audit
    is needed or whether skip-reaudit is safe.** See Skip-Reaudit
    Criteria below.

**Role 2 must NOT:**

- Start coding before explicit Principal approval for the
  implementation round.
- Silently diverge from the audit instructions.
- Assume the audit was exhaustive.
- Close the cycle without either a verifying re-audit from Role 1 or a
  Principal-approved skip-reaudit declaration.
- Write to CHANGELOG.md in mid-cycle rounds (batching rule).

**Further clarification:**

- It is allowable to diverge from audit instructions when code reality
  forces a deviation, but it must be reported.
- It is best to assume the audit was NOT exhaustive and that Role 2 can
  add value by looking for gaps and pushing back when needed.
- Role 2's secondary role is supplemental auditor — report audit
  findings that were not addressed by the auditor, and whether or not
  Role 2 already implemented fixes based on those findings, for the
  auditor to incorporate in their re-audit.
- Praise to the auditor is encouraged when the instructions left are
  elegant or when the auditor identifies clever ways to solve problems.
- Praise to the re-auditor is also encouraged when a re-audit finds
  gaps in the first-pass implementation.

**Expected output from Role 2:**

- The code changes themselves.
- An implementation report appended to the cycle section in
  `AUDIT_LOG_FILE`.
- Synced `HANDOFF.md` and `TASK.md`.
- Clear note of tests run and any deviation from the audit instructions.
- An explicit decision at round end: "re-audit needed" OR
  "skip-reaudit recommended" (with reasoning, for Principal to approve).

### Skip-Reaudit Criteria

Role 2 may **recommend** skipping re-audit only if **ALL** of the
following are true:

1. Every item in Role 1's audit instructions was implemented exactly as
   specified.
2. Zero divergences from the audit instructions.
3. All justified tests passed green (e.g., `make test` fully green;
   targeted regressions covering the changed surface all pass).
4. No in-situ audit miss was found that required structural or design
   decisions. (Trivial fixes handled inline are OK; decisions are not.)
5. No ambiguity was encountered during implementation.
6. The cycle is mechanical (thin-priority batched cycle or
   straightforward substantive priority without design judgment).

Role 2 **must request re-audit** (never skip) if ANY of these are true:

- Any divergence from instructions, even justified.
- Any test failed at any point, even if later fixed.
- Any audit miss that required judgment to handle.
- The cycle involves design judgment (help-string quality bar,
  terminology harmonization, cross-pipeline framing, Universal
  promotion, closeout doc sweeps).
- Any uncertainty about whether the implementation fully satisfied the
  instructions.

**The Principal has final approval on any skip-reaudit.** Role 2's
declaration is a recommendation, not a decision. If the Principal
overrides, re-audit happens.

## Shared rules for every cycle-execution role

1. Work one cycle at a time.
2. Inspect live code rather than trusting markdown reports.
3. Include `scripts/` and `tests/` in audits and re-audits whenever
   they are plausibly affected.
4. Keep the round-by-round audit trail in `AUDIT_LOG_FILE`. Keep the
   SPEC clean.
5. Record each cycle closeout as a single CHANGELOG entry with
   consolidated authorship. Do not write CHANGELOG entries per
   role-round.
6. Update `HANDOFF.md` and `TASK.md` whenever a round changes the next
   actionable state.
7. Follow `multi-agent/AGENT_CONVENTIONS.md` for authorship,
   timestamps, approval rules, and scope authority.
8. Do not rewrite history in `AUDIT_LOG_FILE`. Append new rounds; do
   not erase prior rounds.
9. When code reality differs from the written audit, prefer
   correctness, then document the divergence explicitly in
   AUDIT_LOG_FILE.
10. Surface scope questions explicitly. If adjacent work clearly falls
    within the spirit of the current phase, flag it as a broadening
    opportunity in the audit output — do not silently add it, but also
    do not relegate it to KNOWN_ISSUES without asking. The Principal is
    the sole authority on all scope decisions.
11. **Compaction awareness:** If the session has been compacted since
    you last read a required file, treat yourself as a cold start and
    re-read. Re-reads are cheap insurance against format drift; skipping
    them after compaction risks drift that costs far more to fix.
12. **Substantive-cycle rule:** Do not run a full
    audit-implement-reaudit loop on thin priorities. Bundle thin
    priorities into batched cycles. See § 3 (Substantive priorities +
    cycle granularity).
13. **Skip-reaudit is a Role 2 recommendation, not a decision:** Only
    the Principal can accept a skip-reaudit. All skip criteria must be
    met or re-audit is required.
14. **Authorship consolidation at cycle closeout:** The closing agent
    MUST survey AUDIT_LOG_FILE's cycle section and consolidate all
    `**Authors:**` lines into one deduplicated line in the CHANGELOG
    entry.

## Phase Development discipline rules (stale-context handling)

You are in a repo under active development by multiple agents. Assume
that in the time between each interaction, other agents may have
already made changes to the codebase, making your context window
stale. To combat stale knowledge, do all of the following. (Carried
forward verbatim from `PDS-v3 § Phase Development Rules`; applies to
every role in every stage, but especially to R1/R2/R3 in cycle
execution.)

**Before making any decisions:**
- Re-read all files you will modify.
- Do not rely on prior context or memory.
- Assume the codebase may have changed since your last interaction.

**For each file you modify:**
- Show the current relevant snippet.
- Confirm it matches your assumptions.
- Then apply changes.

**Do not rely on:**
- Previous audits.
- Earlier session context.
- Remembered file structure.

**Only trust the current file contents.**

**Before inserting into `CHANGELOG.md` or `multi-agent/DEVLOG.md`** (only at
cycle closeout — not during stage iteration):
- Locate the most recent entry.
- Confirm insertion point explicitly.
- Do not assume location from memory.
- Route by primary content per `AGENT_CONVENTIONS.md` (brainstorming +
  SPEC engineering cycles are typically DEVLOG; product-feature cycles
  are typically CHANGELOG).

**After making changes:**
- Re-read the modified files.
- Confirm they follow conventions exactly.
- Check against `AGENT_CONVENTIONS.md`.

**On cold starts:**
- Read one agent file (CLAUDE.md, AGENTS.md, GEMINI.md, or
  `.github/copilot-instructions.md`).
- Then read `multi-agent/AGENT_CONVENTIONS.md`.
- Then read this v4 workflow file (or your assigned section).

**After finishing up:**
- Read one agent file again at the wrap-up checkpoint.
- Then re-read `multi-agent/AGENT_CONVENTIONS.md`.
- These re-reads are deliberate quality guardrails against format drift
  — not redundancy.

**Before giving your wrap-up summary comments to the Principal:**
- Ensure that you have completed all that was asked of you from the
  prompt, from this file, and from `AGENT_CONVENTIONS.md`.

**Compaction awareness (also covered in shared rule #11 above; restated
for emphasis):**
- If the session has been compacted since you last read a required
  file, treat yourself as a cold start and re-read. Compaction voids
  the "I already read it this session" assumption. Required re-reads
  are cheap insurance against format drift and convention drift that
  costs far more to fix later.

**Be thorough during BRAINSTORM and SPEC engineering stages** (Stages 2
+ 3; carried into cycle-execution audits when changes touch CLI flag
surfaces):
- Identify all code that touches or is touched by changes you are
  proposing, and that will need to be updated in parallel.
- Identify CLI flags that need updates:
  - If a CLI flag was previously restricted to one pipeline but now
    applies to more, it needs a CLI flag in the universal section as
    well as pipeline-specific flags that can override the universal.
  - Help-strings may need to be updated.
- If a new CLI flag is needed:
  - If it will eventually apply to multiple pipelines, then there
    needs to be one in the universal section, and pipeline-specific
    versions in the pipeline sections that can override the universal.
  - Place the CLI flag in the organization category that makes most
    sense:
    - Applies to all pipelines → Universal, with possible
      pipeline-specific overrides.
    - APS-related → APS section.
    - Timing-related → Timing section.
    - HMM-related → HMM section.
    - Etc.
- Search through markdown files as well as code:
  - Look for overlapping ideas in `multi-agent/tracking/KNOWN_ISSUES.md`
    and `multi-agent/tracking/BRAINSTORM.md`.
  - Include updates needed for `README.md` and similar documentation
    files.

**Aim for substantive priorities** (carried forward from v2; restated
here for stage-iteration awareness — see § 3 for the full rule):
- A Priority that amounts to a single flag rename, a single
  help-string tweak, or a single trivial wire is NOT substantive enough
  to stand alone.
- Bundle related thin work into substantive priorities OR into
  phase-groups that become batched cycles in the audit loop.
- Target: 5-10 substantive priorities per phase, not 30 thin ones.
- "Atomize steps" still applies to pipeline-step separation, not to
  priorities.

## AUDIT_LOG.md structure

Round-by-round history of the audit-implement-reaudit loop. Every
Role 1 audit round, Role 2 implementation round, Role 1 re-audit or
skip-reaudit closeout, Role 3 wrap-up audit, post-wrap-up triage /
implementation / closeout — all go here under dated, role-labeled
subsections per cycle.

**Structure:**

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

**Newest-first throughout** — both rounds within a cycle and cycles
across the file. 'Newest' = timestamp of the most recent round in that
cycle.

When a cycle closes, update the Cycle heading to `CLOSED v0.X.YY` and
update the priority status table in the SPEC.

## Required file updates per cycle

Minimum required artifacts:

1. **`TARGET_FILE` (the SPEC).** Priority status table updated at cycle
   closeout. Scope-change edits only (Principal-approved). NOT the
   place for audit findings, implementation reports, or closeout
   judgments.
2. **`AUDIT_LOG_FILE`.** Role 1 audit findings + exact instructions
   (every audit); Role 2 implementation reports (every implementation);
   Role 1 closeout judgments (at cycle closeout); Role 3 wrap-up audit;
   Role 1 post-wrap-up triage + closeout. Every round carries a
   `**Authors:**` line.
3. **`CHANGELOG.md` (or `multi-agent/DEVLOG.md` per routing rule).**
   One entry per cycle closeout. Consolidated authorship (survey
   AUDIT_LOG_FILE cycle section for all `**Authors:**` lines,
   deduplicate, Principal first). Not per role-round.
4. **`multi-agent/project_context/HANDOFF.md`.** Update at any state
   change between rounds.
5. **`multi-agent/project_context/TASK.md`.** Update at any state
   change between rounds.

If the wrap-up audit opens new actionable work, `HANDOFF.md` and
`TASK.md` should move back from "closed" language to the specific next
repair action owned by Role 1 or Role 2.

Optional additional artifacts when relevant:

- `ROADMAP.md` if a priority within an already-formalized-SPEC phase
  closes (add a `✓ DONE (v0.x.xx)` or `◑ PARTIAL (v0.x.xx)` marker to
  the existing retrospective entry). See
  `multi-agent/AGENT_CONVENTIONS.md § ROADMAP.md`.
- `DECISIONS.md` if the work settles a reusable design decision.
- Other docs/specs if the audited cycle explicitly requires
  synchronization.

## Acceptance and closeout rules

A target cycle is closed only when ALL of the following are true:

1. An auditing role has inspected the actual changed code (OR Role 2
   skip-reaudit was accepted by Principal AND Role 1 has done
   lightweight spot-check at closeout).
2. The findings in the latest audit round are either resolved or
   explicitly reclassified.
3. The implementation round(s) recorded their validation surface in
   AUDIT_LOG_FILE.
4. `AUDIT_LOG_FILE`, `CHANGELOG.md`, `HANDOFF.md`, `TASK.md`, and the
   SPEC priority status table are consistent with the latest state.
5. No high-confidence open findings remain in the audited cycle.

The full phase is closed only when all targeted cycles are closed and
the Final Overseer wrap-up audit (Stage 6) does not identify remaining
blocking findings, or when any actionable wrap-up findings have gone
through the post-wrap-up remediation cycle and been closed by Role 1.

## Stage transition out

Once the last cycle closes, Stage 6 (Final Overseer pass) opens. The
Orchestrator drafts the Template D launcher.

---

# 9. Stage 6: Final Overseer

Stage 6 is the end-of-phase independent review. After all execution
cycles in Stage 5 have closed, Final Overseer (Role 3 in 3-role mode)
performs a full-file wrap-up audit before any phase archive.

## Independence requirement

**Final Overseer must be a fresh chat, structurally independent from
the Orchestrator's accumulated frame.** This is the structural value of
the role — the Orchestrator has been deliberating + drafting + pre-
checking across the phase; Final Overseer's value is reading the
finished phase artifacts with fresh eyes.

The Orchestrator does NOT brief the Final Overseer directly. Final
Overseer reads the **live phase artifacts** (SPEC, AUDIT_LOG, STRATEGY,
BACKGROUND, SURPRISE_LOG) independently and reaches its own conclusions.
The phase artifacts are NOT yet archived at Stage 6 — archive happens
at Stage 8 after the Succession Briefing (Stage 7) is written. After
Final Overseer signs off, the Orchestrator incorporates Final Overseer's
findings into the Succession Briefing (Stage 7).

## Role 3 — Final Overseer / Wrap-Up Auditor

Role 3 is optional in 2-role mode (in which Role 1 performs the
wrap-up) and recommended in 3-role mode.

**Primary responsibilities:**

1. Read the full target SPEC, the full AUDIT_LOG file, and sample the
   codebase against both.
2. Audit the actual codebase against the claims made by Role 1 and
   Role 2 in AUDIT_LOG.
3. Look for:
   - Missed issues inside already-closed cycles.
   - Inconsistencies between audits and implementation.
   - Adjacent issues that fit the spirit of the target phase or plan.
4. When the target is a phase spec, also compare it against the live
   near-term and speculative backlog surfaces to identify overlap,
   already-parked work, and items that still fit the spirit of the
   current phase. For onionskin, this usually means checking
   `multi-agent/tracking/KNOWN_ISSUES.md` and
   `multi-agent/tracking/BRAINSTORM.md`.
5. Write a wrap-up audit cycle section in `AUDIT_LOG_FILE`.
6. At wrap-up cycle closeout (no follow-up actionable work found):
   update SPEC priority status table if applicable, and write the
   wrap-up audit CHANGELOG entry with consolidated authorship.
7. Update `HANDOFF.md` and `TASK.md` only if the wrap-up audit creates
   concrete next actions.
8. Separate actionable follow-up findings from future-looking
   observations so Role 1 can triage the implementation-worthy items
   cleanly.

**Role 3 must NOT:**

- Edit code unless the Principal explicitly converts the round into an
  implementation round.
- Trust either audit reports or implementation reports without checking
  live code.
- Write audit content to the SPEC body; audit content goes to
  `AUDIT_LOG_FILE`.

**Expected output from Role 3:**

- A wrap-up audit cycle section in `AUDIT_LOG_FILE`.
- A `CHANGELOG.md` entry (one, at wrap-up cycle closeout) with
  consolidated authorship.
- A clear statement of whether the target file is fully closed or has
  remaining findings.
- When relevant, a short overlap assessment against `KNOWN_ISSUES.md`
  and `BRAINSTORM.md` identifying which findings are already parked
  versus newly surfaced.
- When relevant, a clearly labeled actionable-follow-up block that can
  be handed back to Role 1.

## Post-wrap-up remediation loop

If the wrap-up audit finds actionable issues:

1. Role 1 reviews the wrap-up audit and decides which findings become
   active repair work.
2. Role 1 opens a new "post-wrap-up remediation" cycle section in
   `AUDIT_LOG_FILE` and translates the actionable findings into
   concrete instructions.
3. After explicit Principal approval, Role 2 implements that
   post-wrap-up repair set, reports the validation in AUDIT_LOG_FILE,
   declares re-audit or skip-reaudit.
4. Role 1 performs the final closeout (full re-audit or lightweight
   spot-check depending on Role 2 declaration + Principal decision).
   At closeout: single CHANGELOG entry for the post-wrap-up remediation
   cycle.
5. Role 3 usually does not re-audit this second pass unless the
   Principal explicitly requests it.

The Final Overseer normally does ONE pass only.

## Stage transition out

After Final Overseer signs off (and any post-wrap-up remediation
closes), Stage 7 (Succession Briefing) opens. The outgoing Orchestrator
writes the Succession Briefing per Template J.

---

# 10. Stage 7: Succession Briefing

Stage 7 is the **outgoing Orchestrator's wind-down deliverable**. After
Final Overseer signs off + any post-wrap-up remediation closes, the
outgoing Orchestrator writes the Succession Briefing at
`multi-agent/plans/PHASE<N>_SUCCESSION_BRIEFING.md` per **Template J —
Succession Briefing** (Section 14).

## What the Succession Briefing is

A wisdom-transfer document for the next-phase Orchestrator. The
briefing is NOT a recreation of the formal artifacts (SPEC, AUDIT_LOG,
STRATEGY, BACKGROUND, SURPRISE_LOG); those archive intact in Stage 8.
The briefing captures what's NOT in the formal artifacts:

1. Phase summary (theme, headline accomplishments, total cycle count,
   total commits, headline metrics).
2. Cross-cycle pattern memory (recurring patterns; cite specific cycle
   references; recommend counter-discipline for next Orchestrator).
3. Recurring blind spots that need vigilance.
4. Deliberation patterns that worked.
5. Things that didn't work + lessons.
6. Brainstorm seeds + SOUP entries that emerged but weren't formalized.
7. Open SURPRISE/ISSUE entries with cross-phase implications.
8. Recommended Orchestrator working style for next phase (model + Effort
   choice; expected cycle count range; recurring user-feedback memories
   to load on cold-start).
9. Final Overseer's findings + carry-forwards.
10. Praise + things to keep doing.
11. Things to avoid repeating.

See Template J in Section 14 for the full output scaffold.

## Why the Orchestrator writes this, not the Final Overseer

Final Overseer's structural value is independence from the
Orchestrator's accumulated frame. If Final Overseer wrote the Succession
Briefing, they would either lose that independence (now thinking like
an Orchestrator) or write a briefing without the deliberation context
(worse than the outgoing Orchestrator's). The cleanest split: Final
Overseer audits independently; outgoing Orchestrator inherits Final
Overseer's findings and writes the wisdom transfer.

## Why AFTER Final Overseer signs off, not before

Writing the briefing before Final Overseer would (a) introduce bias
into Final Overseer's review (they might read the briefing as a hint),
and (b) miss any findings Final Overseer surfaces. Writing AFTER lets
Final Overseer's findings become inputs to the briefing.

## Stage transition out

Stage 8 (Phase Archive) opens after the Succession Briefing is written
and reviewed by the Principal.

---

# 11. Stage 8: Phase Archive

Stage 8 is the final operation: moving the phase artifacts to
`archived/`. The phase archive is **NOT a cycle deliverable**. It is a
separate operation that happens AFTER:

1. The last execution cycle of the phase closes (Stage 5 complete).
2. Final Overseer's wrap-up audit closes (Stage 6).
3. Any post-wrap-up triage / remediation / closeout cycles close
   (Stage 6 remediation loop).
4. The Succession Briefing is written (Stage 7).

The Principal performs the archive as an operation outside the cycle
structure, after confirming all wrap-up + briefing work is complete.

## Files moved to archive

At phase close, the following move to `multi-agent/plans/archived/<YYYYMMDD>-PHASE<N>_*.md`:

- `PHASE<N>_SPEC.md`
- `PHASE<N>_AUDIT_LOG.md`
- `PHASE<N>_STRATEGY.md`
- `PHASE<N>_BACKGROUND.md`
- `PHASE<N>_SURPRISE_LOG.md`
- `PHASE<N>_SUCCESSION_BRIEFING.md` (NEW under v3+)
- Any per-phase `BRAINSTORM.md` or `FEEDBACK.md` if not already
  archived (they typically archive at end of Stage 3).

## Archive operations are Principal-run

Per project memory `feedback_archive_operations_orchestrator_only.md`,
archive operations are Principal-run. The Orchestrator emits the `git
mv` operations in a commit block; the Principal runs them. (The
memory's "orchestrator-only" naming refers to the v2
human-orchestrator framing; under v3+ the AI Orchestrator drafts and
the Principal executes.)

## Why archive-last matters

Final Overseer reads SPEC + AUDIT_LOG + STRATEGY at their live paths
and writes its wrap-up findings INTO the AUDIT_LOG. Writing into an
archived file violates the AGENT_CONVENTIONS.md rule "Once archived, a
phase SPEC/PLAN file should be treated as immutable historical
provenance." Post-wrap-up remediation cycles also write to the live
AUDIT_LOG. Archiving early forces those writes into archived paths,
which breaks immutability and confuses provenance.

## What cycle-scoped closeout work IS in scope

Reference propagation across agent files / workflows / live plan files;
tracking-directory moves (e.g., `multi-agent/BRAINSTORM.md` →
`multi-agent/tracking/BRAINSTORM.md`); CLI conventions codification in
AGENT_CONVENTIONS.md. These are doc-sweep-style edits that ship within
a cycle's atomic commit. The actual `git mv` of the
SPEC/AUDIT_LOG/STRATEGY/FEEDBACK files into `archived/` does NOT.

## What cycle-scoped closeout work MAY include

Transitional accommodations for the eventual archive: updating
agent-file active-phase pointers to flag that the phase will close
soon, and pre-staging any archive-related rationale in DECISIONS.md.
The pointer can name the *current* live SPEC path (still active) and
note the planned post-Overseer archival; the actual rewrite to
"archived at `<date>-<file>`" happens AFTER the archive operation.

## Audit-time check

Any Role 1 audit whose repair instructions propose archiving
SPEC/AUDIT_LOG/STRATEGY/FEEDBACK files into `archived/` MUST stop and
re-check. The archive is a post-Final-Overseer event; if the audit is
for a within-phase cycle (including the last cycle of the phase), the
archive is out of scope.

## Outgoing Orchestrator chat closes

After the Principal completes the archive, the outgoing Orchestrator
chat closes. The next-phase Orchestrator (recommended: fresh chat,
possibly different AI instance) bootstraps by reading the archived
prior-phase artifacts + Succession Briefing.

## Stage transition out

Phase complete. Next phase begins at Stage 1 with a new Orchestrator.

---

# 12. Authorship + commit-block conventions

## Authorship line format

Every agent-authored entry (audit-log rounds, CHANGELOG/DEVLOG entries,
scratch files, etc.) carries a `**Authors:**` line in the uniform form:

```
Authors: <Principal>, <harness> <version> (<model> ; <Toggle>: <value>)[, <next agent>...]
```

Examples:

- `Authors: John M. Urban, Claude Code 2.1.118 (claude-opus-4-7 ; Effort: Max)`
- `Authors: John M. Urban, Codex-cli 0.126.0-alpha.8 (GPT-5.5 ; Reasoning: Extra High)`
- `Authors: John M. Urban, GitHub Copilot (GPT-5.4 ; Reasoning: Extra High)`

The Principal (human) is listed first. Each agent that contributed to
the round / entry is listed in the order of their contribution. See
`multi-agent/AGENT_CONVENTIONS.md § Authorship` for full rules.

## Authorship consolidation at cycle closeout

The closing agent (typically Role 1 at re-audit closeout, or the agent
that writes the CHANGELOG/DEVLOG entry) **MUST survey
AUDIT_LOG_FILE's cycle section** and consolidate all `**Authors:**`
lines from the cycle's rounds into one deduplicated line in the
CHANGELOG/DEVLOG entry. The Principal is listed first; agents follow in
the order they appeared across the cycle's rounds; duplicates removed.

Example:

```
Cycle rounds had:
- R1 audit Authors: John M. Urban, Claude Code 2.1.118 (claude-opus-4-7 ; Effort: Max)
- R2 implementation Authors: John M. Urban, GitHub Copilot (GPT-5.4 ; Reasoning: Extra High)
- R3 closeout Authors: John M. Urban, Claude Code 2.1.118 (claude-opus-4-7 ; Effort: Max)

Consolidated for CHANGELOG entry:
Authors: John M. Urban, Claude Code 2.1.118 (claude-opus-4-7 ; Effort: Max), GitHub Copilot (GPT-5.4 ; Reasoning: Extra High)
```

(Claude Code appears once even though it contributed in R1 and R3.)

## Convention for agent-produced handoff prompts

When an agent emits a handoff prompt at end-of-round for the next role
(per Templates B / C / H / I in Section 14):

1. **Omit the `Below N and <N> = ...` line entirely.** The concrete
   paths in the prompt self-identify the phase; the line is vestigial
   in a hard-coded handoff.
2. **Hard-code `N` throughout the rest of the prompt.** Substitute
   every `PHASE<N>_*.md` path with the concrete filename (e.g.,
   `PHASE15_SPEC.md`), and use concrete supplemental-phase filenames
   where applicable (e.g., `PHASE14_SUPPLEMENTAL-SPEC.md`). No
   uninstantiated `<N>` placeholders.
3. **Include the two standard guardrail blocks verbatim.** Agent-emitted
   handoffs MUST carry the "Required checkpoint reads" block (agent
   file + AGENT_CONVENTIONS.md re-read guardrail) and the "Before
   giving your wrap-up summary comments to the user" block
   (workflow-file re-read + role completeness check). These are what
   make cold-paste launchers and agent-emitted handoffs identical in
   effect. Copy them from the relevant template (A/B/C/D/E/F/G/H)
   verbatim.
4. **Precede the launcher code block with an Orchestrator advisory
   block.** Each agent-emitted handoff must be introduced by a short,
   structured advisory naming the next assignee, role, template, and
   cycle — pulled from `PHASE<N>_STRATEGY.md`. The advisory sits
   OUTSIDE the ```text code block so the code block stays clean for
   paste. Required format:

   ```markdown
   **Orchestrator advisory (who runs this next):**
   - **Cycle:** <index + name> — <one-line priority summary>
   - **Role & template:** <Role 1|2|3> <round type> → use Template <A|B|C|D|E|F|G|H>
   - **Assigned agent (per PHASE<N>_STRATEGY.md):**
     - Primary: <primary agent + effort/reasoning/thinking level from the relevant role column>
     - Alt: <alt agent(s) + level from the relevant role column>
   - **Notes:** <execution-relevant notes from STRATEGY Notes column>

   Paste the following into the next agent's fresh chat:
   ```

   Mapping from STRATEGY.md role columns to workflow role + template:

   | STRATEGY column | Next workflow role + template |
   |---|---|
   | R1 (audit)      | Role 1 initial audit, Template A |
   | R2 (implement)  | Role 2 implementation, Template B |
   | R3 (closeout)   | Role 1 re-audit, Template C (or Template H if skip-reaudit accepted) |

   The Final Overseer pass (Template D) uses the "Final Overseer" row
   of STRATEGY.md, not a per-cycle role column.

5. **Output each launcher prompt in a `\`\`\`text` fenced code block.**
   Never emit the prompt body as inline chat prose. VS Code and
   similar markdown environments render fenced code blocks with a copy
   button that preserves indentation and structure on paste. The
   Orchestrator advisory (rule 4) sits OUTSIDE the fence so the paste
   target is clean.

## Convention for agent-produced git commit blocks

Every agent role, when wrapping up its session, MUST emit a `git`
command block in addition to its handoff prompt(s). The block sits
between the role's wrap-up summary and the Orchestrator advisory +
handoff prompt. It tells the Principal exactly which commands to run to
commit the role's work. Applies to all role-templates (A, B, C, D, E,
F, G, H, I, J).

The agent does NOT run any of these commands itself — git control
belongs to the Principal. The agent only outputs the commands.

**The block MUST be a fenced code block (triple backticks), not inline
prose or inline code.** Never write git commands as plain sentences or
single-backtick inline code.

There are two forms depending on whether the role writes a
CHANGELOG/DEVLOG entry:

### A. Mid-cycle commit

For most rounds — R1 audit, R2 implementation when re-audit is declared
needed, etc. No CHANGELOG/DEVLOG entry is written this round; the
commit message is a one-line summary of the role's work:

```
git add <explicit list of files; never `git add .` or `-A`>
git commit -m "Phase <N>, cycle <name> (index <i>), Role <r> <round type> (Template <X>): <one-line summary>. | Authors: <consolidated authorship including toggle, per AGENT_CONVENTIONS authorship section>"
```

### B. Closeout commit

For the role that writes the CHANGELOG or DEVLOG entry — typically Role
1 at re-audit closeout, Role 2 at skip-reaudit acceptance (Template H),
Role 3 at wrap-up audit closeout, etc.

The closing agent:
1. Writes the CHANGELOG/DEVLOG entry into the appropriate file.
2. Writes a verbatim copy of that entry to a top-level scratch file
   named `changelog-entry-<version>.txt` (with the actual version
   number; the file is gitignored via `changelog-entry-*.txt` so it
   cannot be accidentally committed).
3. Emits THREE commands for the Principal:

```
git add <explicit list of files; include CHANGELOG.md or DEVLOG.md and any other files modified>
git commit --file=changelog-entry-<version>.txt
rm changelog-entry-<version>.txt
```

**Why `--file=` and not `-m`:** the closeout commit fossilizes the
CHANGELOG/DEVLOG entry verbatim in git history, so `git log` becomes
directly searchable for the full entry text and the original wording is
preserved regardless of whether the CHANGELOG/DEVLOG file is ever later
reformatted, migrated, or split. The version-tagged filename
eliminates "did I pick the wrong txt file?" risk.

`<version>` is the full version string of the entry being committed:
`vR.X.YY` for a CHANGELOG entry (e.g., `v0.14.89`) or `vR.X.YY.Z` for a
DEVLOG entry (e.g., `v0.14.89.1`). See
`multi-agent/AGENT_CONVENTIONS.md § CHANGELOG.md and DEVLOG.md` for
the version-format rules. The gitignore pattern `changelog-entry-*.txt`
covers both forms.

### Appending to an already-committed entry

If the Principal requests appending content to an existing
CHANGELOG/DEVLOG entry whose original commit has already happened,
edit the entry in-place to add the appended subsection, then create a
NEW scratch file `changelog-entry-<version>-followup<N>.txt` (suffix
`-followup1`, `-followup2` for subsequent follow-ups) containing just
the appended subsection content. Use the new follow-up file in the
three-command closeout block. If the original commit has NOT yet
happened, edit the entry in-place and rewrite the existing
`changelog-entry-<version>.txt` with the full updated entry — the
Principal's existing commands still work. The agent picks the scenario
by checking `git log --oneline` for the version commit, or by asking
the Principal.

### Rules common to both forms

- Output the block as a fenced code block (triple backticks) — never
  inline prose or inline code.
- List explicit files in `git add`. Never use `git add .` or
  `git add -A`.
- Include the full authorship line with runtime toggle values (Effort /
  Reasoning / Thinking) per the AGENT_CONVENTIONS authorship rules.
- The commit block goes BEFORE the Orchestrator advisory + handoff
  prompt in the agent's wrap-up output, so the Principal runs git
  commands first, then pastes the handoff into the next chat.

### Git-discipline principles

1. **No interactive git in copy-paste blocks — ever.** Every command in
   the block must run cleanly in a non-interactive shell. Forbidden:
   `git add -p`, `git add -i`, `git rebase -i`, `git commit` without
   `-m`/`--file=` (which opens `$EDITOR`), `git mergetool`, any
   `--edit` flag. Use explicit per-file `git add <path>` commands. If
   a hunk-level split is genuinely needed, instruct the Principal
   separately in prose ("after the block runs, please review and stage
   the remaining hunks of `<file>` interactively"); never embed the
   interactive step inside the fenced block.

2. **One concept per commit.** Each commit block represents a single
   coherent change: one cycle round, one role's output, or one named
   operation (e.g., a phase archive). Do not bundle "implementation +
   closeout doc sweep + phase archive" into one commit; do not bundle
   two roles' output. If a round produces work in two distinct
   conceptual buckets, the agent MAY split into two sequential commits
   within the same fenced block — but each commit message must stand
   alone and the file lists must not overlap.

3. **Implementer-splits-stages pattern for multi-step implementations.**
   When a Role 2 executes instructions that decompose into ordered
   stages (e.g., "Stage 1: move tracking files; Stage 2: propagate
   references; Stage 3: add new convention text"), the implementer
   SHOULD emit one commit block per stage rather than a single
   end-of-round mega-block. Each stage commit gets its own one-line
   message describing just that stage's concept; the implementation
   report in the AUDIT_LOG enumerates all stages. This makes
   mid-implementation surgical reverts cheap.

## Principal owns git operations

Per `feedback_git_control.md` (project memory): git control belongs to
the Principal. Agents emit commit blocks but never run them.

The Orchestrator role explicitly does NOT run git commands during
normal operation (per § 2 Boundaries). Time-bounded narrow exceptions
may be authorized by the Principal (e.g., "you are authorized to run
`git add` + `git commit` for the v4 merger work only, scoped to the v4
file" — see git-authorization patterns in project memory) but these
are scoped exceptions, not the default.

---

# 13. Common failure modes

Watch for these repeatedly. Each was observed at least once during
v2 + v3 phase execution; the v4 spine doesn't make any of them go
away — only systematic vigilance does.

## v2-introduced failure modes (still active under v3+v4)

1. Role 1 trusts the implementation report instead of reading the
   changed code.
2. Role 2 trusts the audit blindly and misses obvious nearby drift.
3. One role audits only core modules and forgets `scripts/` or
   `tests/`.
4. A role writes audit content into the SPEC instead of
   `AUDIT_LOG_FILE`.
5. A role writes a CHANGELOG entry mid-cycle instead of at cycle
   closeout.
6. The closing agent forgets to consolidate authorship and writes a
   CHANGELOG entry with only its own author line, missing contributors
   from prior rounds.
7. `HANDOFF.md` or `TASK.md` lag behind the actual state.
8. A role silently narrows or broadens scope. **The failure is the
   silence, not the scope change:** raising a scope expansion as an
   explicit finding is correct behavior; quietly omitting work or
   relegating it to KNOWN_ISSUES without Principal approval is not.
9. A role rewrites prior audit/implementation content in
   AUDIT_LOG_FILE instead of appending a new round entry.
10. Role 3 mixes actionable findings and future-phase suggestions
    without labeling which is which.
11. Role 1 leaves Final Overseer findings only in the wrap-up section
    instead of opening a post-wrap-up remediation cycle in
    AUDIT_LOG_FILE.
12. Role 3 checks only the live code and spec text, but forgets to
    compare the target plan against `KNOWN_ISSUES.md` and
    `BRAINSTORM.md` for overlap and in-scope follow-up.
13. Role 2 recommends skip-reaudit when criteria are not all met. Role
    2 must be honest about its own uncertainty; better to request
    re-audit than to push past a cycle with a silent drift.
14. Principal accepts a skip-reaudit without Role 1 performing the
    lightweight spot-check at closeout.
15. An agent running thin priorities as individual cycles instead of
    batching them (violates substantive-cycle rule).
16. After a compaction event, an agent skips the required checkpoint
    reads on the assumption "I already read it this session"
    (compaction voids that assumption).

## v3+v4 failure modes (Orchestrator + Phase 15 patterns)

17. **Conflation of structural deliverables with calibration evidence
    (cycle 15.4a-S3 GAP-1 pattern).** R2 produces calibration TSV +
    prose evidence and skips the explicit cross-pipeline parity table.
    R3 has to remediate inline. Counter-discipline: R1's audit must
    explicitly mandate the structural parity table at audit-time, with
    a row count + column scaffold; R2 must produce the table inline in
    the implementation report (not as appendix).
18. **Help-text contract simplification (cycle 15.4a-S3/S4 pattern).**
    R2 implements flag with technically-correct help text but loses
    the user-direction guidance the orchestrator clarification
    specified. Counter-discipline: R1 audit specifies the help text
    verbatim or near-verbatim in the audit findings; R2 includes help
    text in the implementation report's contract verification.
19. **Empirical-default overfit to amplification-rich fixtures (cycle
    15.4a-S4 pattern).** R2's Stage A.5 sweep produces empirically
    "valid" defaults that are dataset-specific and don't generalize.
    Biology priors are more robust for static defaults.
    Counter-discipline: Orchestrator should challenge empirical
    defaults that derive from non-representative data; biology priors
    should win when the empirical evidence is artifact-tight.
20. **Cross-pipeline parity blindspot (recurring; per
    `feedback_cross_pipeline_parity_blindspot.md`).** Mandate parity
    table verification cell-by-cell. R3 must independently confirm each
    cited file:line on the live tree, not rubber-stamp the table.
21. **Authorship/Effort attribution drift — agents fabricate
    plausible-looking reasoning/effort values in authorship lines
    regardless of actual setting.** Recurring across all agents
    (Copilot, Codex, Claude, Gemini). Agents cannot reliably introspect
    their own reasoning/effort level; the value is set in the host UI
    and not exposed through any system prompt, environment variable,
    or tool. Examples: an Opus session running at Effort: Max gets
    written as "Effort: High" (early Phase 14); Copilot at Reasoning:
    High writes "Reasoning: Extra High" in its authorship line (Phase 15
    cycle 15.6a-S1 R2 round, 2026-05-03). Counter-discipline (two parts):
    (a) when drafting a launcher prompt for an agent, name the
    reasoning/effort level explicitly in the prompt body so the agent
    has nothing to fabricate. This is the strongly-preferred path.
    (b) when the launcher does not name the level, the agent must ask
    the user to confirm as the FIRST IMMEDIATE ACTION of the round,
    BEFORE any substantive work — catch the user at the keyboard
    before they walk away. Authoritative source for the toggle value
    is the user's explicit confirmation in the current chat;
    STRATEGY-row primary/alt assignments are suggestive only; agent
    personal-memory entries are stale snapshots and NOT authoritative
    (the user changes the toggle at will whenever he sees fit). The
    Orchestrator periodically sweeps for drift and applies retroactive
    corrections as a discrete commit; R3 verifies attribution at
    cycle close against the user's confirmation, not against the
    agent's self-report. See `multi-agent/AGENT_CONVENTIONS.md
    § Authorship conventions § Per-agent format § Rules` for the
    full rule.
21a. **Deferral as ordinary disposition — agents treat "close PARTIAL
    + file follow-up `[ISSUE:YYYY-MM-DD:N]`" as a default-available
    menu option for cycle closeout.** Recurring across all agents.
    Surfaced explicitly Phase 15 cycle 15.7a R3 round (2026-05-04):
    R3 proposed framing-(a)+(c)-hybrid closeout with SPEC15.15
    d3/d4/d6/d8 deferred to a new `[ISSUE:2026-05-04:1]`;
    Principal-orchestrator overrode 2026-05-04 because the deferral
    pattern is silent narrowing dressed up as orchestrator process.
    Same pattern surfaced earlier at cycle 15.6a-S1 (R2's
    "all-levels-mean" trajectory clustering — Principal mental-
    simulation pushback forced redesign in-cycle). The fix is the
    strict pathology-only rule per `multi-agent/AGENT_CONVENTIONS.md
    § Scope authority § Deferral is NOT an ordinary disposition
    (critical — strict pathology-only rule)`. Summary:
    - **Default expectation for ALL agents (R1/R2/R3/orchestrator):**
      implement what was scoped. Sole carve-out is PATHOLOGY
      (impossible OR genuinely harmful implementation). "Big" /
      "complex" / "out of scope estimate" / "could be cleaner later"
      / "low priority for the phase" are NOT pathology.
    - **4-step protocol:** (1) any agent raises `[PATHOLOGY_CANDIDATE]`
      mid-flight (real-time chat + audit log) with required fields;
      (2) block affected deliverable immediately; (3) independent
      audit by a different role produces `[PATHOLOGY_AUDIT_VERIFIED]`
      or `[PATHOLOGY_AUDIT_REJECTED]` (independent code-grounded
      verification, NOT acceptance of raising agent's reasoning);
      (4) Principal decides — `[PATHOLOGY_REJECTED:IMPLEMENT]` /
      `[PATHOLOGY_CONFIRMED_DEFER:<landing-zone>]` /
      `[PATHOLOGY_CONFIRMED_RESTRUCTURE]`.
    - **Cross-agent skepticism:** all agents err toward REJECTING
      other agents' deferral plans; independent investigation
      required; default toward rejection.
    - **Per-role flavor:** R1 never decides deferrals during initial
      audit (raises `[PATHOLOGY_CANDIDATE]` and stops writing R2
      contract for the flagged item); R2 never silently emits
      sidecar/placeholder/TODO substitutes or files follow-up ISSUEs
      as deferral mechanisms; R3 errs strongly toward rejecting
      deferrals — confirms only on independent code-grounded pathology
      verification, never on R2's reasoning alone.
    - **Distinct concept:** `[ALTERNATIVE_APPROACH_PROPOSAL]` is a
      SPEC amendment request mid-cycle (markedly better runtime/
      memory/parallelization/structural cleanliness — concrete
      reasoning), NOT a deferral. Different protocol (no independent
      audit needed; Principal decides directly). See AGENT_CONVENTIONS
      § Alternative-approach proposals — distinct from pathology.
22. **Orchestrator role drift toward implementation.** Orchestrator
    starts drafting code or running git operations that exceed the
    role's boundary. Counter-discipline: re-read § 2 Boundaries
    periodically; defer to R2 for code; defer to Principal for git.
23. **Mid-flight pushback notes that bypass Principal authorization.**
    Orchestrator drafts a pushback note for an in-flight agent, then
    feeds it directly without Principal review. Counter-discipline:
    Principal explicitly approves every mid-flight injection; the
    Orchestrator drafts only.

---

# 14. Prompt templates

**Status under v4 DRAFT.** This section is a **templates index with
v3 cross-references**. The canonical prompt bodies for all templates
currently live in v3:

- Templates 0a-0l (upstream stages) → `multi-agent/workflows/phase-development-system_PDS-v3.md`
- Templates A-J (Strategist, cycle execution, Final Overseer, post-wrap-up, Succession) → `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md`

When v4 stabilizes through review + at least one full-phase trial use,
the full prompt bodies will be inlined into this section in a Pass 4c
polish round (replacing the v3 cross-references with verbatim bodies +
v4 path references throughout). Until then, agents using v4 should
follow the cross-references to v3 for the prompt bodies; the structural
context for each template (purpose, when to use, where the prompt
output lands) lives in this section.

The conventions for **agent-produced handoff prompts** (5 rules) and
**agent-produced git commit blocks** (Form A mid-cycle, Form B closeout
+ git-discipline principles) live in § 12 above. Every template below
inherits those conventions.

## Templates index

### Stage 1-3: upstream stages (per PDS-v3 § "Typical form of..." sections)

| Template | Purpose | When to use | Prompt body source |
|---|---|---|---|
| **0a — Soup-to-BRAINSTORM Opening (Agent 1)** | Promote a candidate SOUP from `plans/next/<THEME>_SOUP.md` to `plans/PHASE<N>_<THEME>_SOUP.md`; one-time SOUP ID labeling pass; author initial BRAINSTORM + seed FEEDBACK | Stage 2 opening (after Principal + Orchestrator decide to promote a SOUP) | PDS-v3 § "Typical form of the Opening brainstorm prompt to Agent 1 from Orchestrator, soup-to-brainstorm optional" |
| **0b — Soup-to-BRAINSTORM Auditor (Agent 2)** | Audit Agent 1's transfer for faithfulness, completeness, conformance; verdict in FEEDBACK as `## Auditor findings — round <N>` | After Agent 1 declares transfer ready for review | PDS-v3 § "Typical form of the soup-to-brainstorm AUDITOR prompt to Agent 2 from Orchestrator" |
| **0c — Soup-to-BRAINSTORM Follow-up (Agent 1)** | Address auditor findings + user FEEDBACK answers in subsequent transfer pass | After Agent 2 audits or Principal answers FEEDBACK questions | PDS-v3 § "Typical form of the soup-to-brainstorm FOLLOW-UP prompt to Agent 1 from Orchestrator" |
| **0d — Soup-to-BRAINSTORM Closeout** | Archive the SOUP file once auditor declares CLEAN; append closeout entry to FEEDBACK; emit closeout commit block | After Agent 2 verdicts CLEAN | PDS-v3 § "Typical form of the soup-to-brainstorm CLOSEOUT prompt to Orchestrator's chosen agent" |
| **0e — Brainstorm Opening (Agent 1; post-soup-transition or no-soup)** | Author / extend the BRAINSTORM with new ideas, deeper analysis, additional sourcing from `tracking/BRAINSTORM.md` and `tracking/KNOWN_ISSUES.md` | Stage 2 opening when not coming from SOUP, or after SOUP-to-BRAINSTORM closeout | PDS-v3 § "Typical form of the Opening brainstorm prompt to Agent 1 from Orchestrator, after soup-to-brainstorm transition or when there is no soup" |
| **0f — Brainstorm Follow-up Opening (Agents 2+)** | Subsequent agents add fresh perspectives + audit in FEEDBACK | After Agent 1's first BRAINSTORM authoring | PDS-v3 § "Typical form of the follow-up opening brainstorm prompts to Agents 2+ from Orchestrator" |
| **0g — Brainstorm Returning (Agent 1)** | Agent 1 returns to BRAINSTORM after FEEDBACK answers + auditor feedback | Subsequent BRAINSTORM iteration rounds | PDS-v3 § "Typical form of the returning BRAINSTORM prompts to Agent 1 from Orchestrator" |
| **0h — Brainstorm Returning (Agents 2+)** | Subsequent auditor agents return for additional iteration rounds | Subsequent auditor rounds | PDS-v3 § "Typical form of the returning BRAINSTORM prompts to Agent 2+ from Orchestrator" |
| **0i — SPEC Engineering Opening (Agent 1)** | First conversion of BRAINSTORM into a SPEC with substantive Priorities + SPEC IDs | Stage 3 opening | PDS-v3 § "Typical form of opening SPEC ENGINEERING stage prompt to Agent 1 from Orchestrator" |
| **0j — SPEC Engineering Subsequent (Agent 2)** | Audit the SPEC against BRAINSTORM coverage + ordering rationale + substantive-priority discipline; findings into FEEDBACK | After Agent 1's first SPEC draft | PDS-v3 § "Typical form of subsequent SPEC ENGINEERING stage prompt to Agent 2 from Orchestrator" |
| **0k — SPEC Engineering Subsequent (Agent 1)** | Agent 1 returns for follow-up SPEC pass after Agent 2 audit + Principal direction | Subsequent SPEC engineering rounds | PDS-v3 § "Typical form of subsequent SPEC ENGINEERING stage prompt to Agent 1 from Orchestrator" |
| **0l — SPEC Engineering Additional (Agent 2)** | Subsequent SPEC engineering audits as iterations continue | Subsequent SPEC engineering audits | PDS-v3 § "Typical form of additional rounds of SPEC ENGINEERING stage prompts to Agent 2 from Orchestrator" |

### Stage 4: Strategist kickoff (Orchestrator's kickoff act)

| Template | Purpose | When to use | Prompt body source |
|---|---|---|---|
| **I — Strategist: Phase Plan of Attack** | Produce `multi-agent/plans/PHASE<N>_STRATEGY.md` — cycle table with primary + alternative agents per role per cycle, deviation rationale, Final Overseer entry | Stage 4 (once per phase, after SPEC is locked) | audit-loop-v3 § "Template I — Strategist: Phase Plan of Attack (STRATEGY.md generator)" |

### Stage 5: cycle execution (audit-implement-reaudit)

| Template | Purpose | When to use | Prompt body source |
|---|---|---|---|
| **A — Role 1 Initial Audit** | Audit a cycle deeply against the live codebase; write findings + repair instructions into `PHASE<N>_AUDIT_LOG.md` cycle section. Includes KNOWN_ISSUES + SURPRISE_LOG carry-over scans + archive-timing check. | First round of every new cycle (Step 1 of standard execution loop) | audit-loop-v3 § "Template A — Role 1 Initial Audit" |
| **B — Role 2 Implementation** | Implement the audited findings; in-situ supplemental auditing while working; declare re-audit-needed OR skip-reaudit-recommended at round end. Includes SURPRISE_LOG status updates + optional new entries. | After Principal approves R1's audit (Step 2 of standard execution loop) | audit-loop-v3 § "Template B — Role 2 Implementation" |
| **C — Role 1 Re-Audit** | Re-audit the live changed code; close cycle (with consolidated authorship in CHANGELOG entry) OR write follow-up findings. Includes SURPRISE_LOG closeout review gate (mandatory) + archive-timing check. | After Role 2 declares re-audit-needed (Step 3 of standard execution loop, re-audit branch) | audit-loop-v3 § "Template C — Role 1 Re-Audit" |
| **H — Role 1 Cycle Closeout After Skip-Reaudit** | Lightweight spot-check (2-3 surfaces) + close cycle (SPEC priority status update + CHANGELOG entry with consolidated authorship). Sometimes also opens the next cycle's R1 audit in same chat. | After Principal accepts Role 2's skip-reaudit recommendation (Step 3 of standard execution loop, skip-reaudit branch) | audit-loop-v3 § "Template H — Role 1 Cycle Closeout After Skip-Reaudit" |

### Stage 6: Final Overseer + post-wrap-up remediation

| Template | Purpose | When to use | Prompt body source |
|---|---|---|---|
| **D — Role 3 Final Wrap-Up Audit** | Full-file wrap-up audit at phase close. Independence requirement (fresh chat, structurally independent from Orchestrator). Includes SURPRISE_LOG end-of-phase sweep + cross-cycle drift check on `plans/next/` SOUP files + ROADMAP update + phase-final X-bump responsibility. | Step 6 (after all execution cycles closed) | audit-loop-v3 § "Template D — Role 3 Final Wrap-Up Audit" |
| **E — Role 1 Post-Wrap-Up Triage** | Translate Final Overseer's actionable wrap-up findings into concrete repair instructions in a new "post-wrap-up remediation" cycle section of `AUDIT_LOG_FILE`. | If Final Overseer finds actionable work (Step 7, triage round) | audit-loop-v3 § "Template E — Role 1 Post-Wrap-Up Triage" |
| **F — Role 2 Post-Wrap-Up Implementation** | Implement Role 1's triaged repair set; report validation in AUDIT_LOG_FILE; declare re-audit or skip-reaudit. | After Role 1 triages Final Overseer findings (Step 7, implementation round) | audit-loop-v3 § "Template F — Role 2 Post-Wrap-Up Implementation" |
| **G — Role 1 Final Closeout After Wrap-Up Remediation** | Re-audit + close the post-wrap-up remediation cycle; CHANGELOG entry; phase-final X-bump if this is genuinely the last LOG entry of the phase. | After Role 2 finishes post-wrap-up implementation (Step 7, final closeout) | audit-loop-v3 § "Template G — Role 1 Final Closeout After Wrap-Up Remediation" |

### Stage 7: Succession Briefing

| Template | Purpose | When to use | Prompt body source |
|---|---|---|---|
| **J — Outgoing Orchestrator: Succession Briefing** | Produce `multi-agent/plans/PHASE<N>_SUCCESSION_BRIEFING.md` — wisdom transfer to next-phase Orchestrator. NOT a recreation of formal artifacts. Required sections: phase summary, cross-cycle pattern memory, recurring blind spots, deliberation patterns that worked, things that didn't, brainstorm seeds, open SURPRISE/ISSUE entries with cross-phase implications, recommended Orchestrator working style, Final Overseer's findings + carry-forwards, praise + things to keep doing, things to avoid repeating. | Stage 7 (after Final Overseer signs off, BEFORE Stage 8 phase archive) | audit-loop-v3 § "Template J — Outgoing Orchestrator: Succession Briefing (new in v3)" |

## Notes on template usage under v4 DRAFT

- **Cold-paste from v3 source.** Until full template bodies are inlined
  in this section, agents pasting templates should pull the body from
  the v3 source file noted in the index above and update file
  references in the prompt:
  - `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md` →
    keep as v3 reference (v3 is still active)
  - `multi-agent/workflows/phase-development-system_PDS-v3.md` → keep
    as v3 reference
  - When v4 becomes the active workflow, references update to
    `multi-agent/workflows/phase-development-system-v4.md`.
- **Legacy -v2 self-references inside v3 prompt bodies — REQUIRED
  cleanup before pasting.** The v3 source files still contain many
  template lines that read `Read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md
  and follow it.` or similar (carried forward from the v2 → v3 bump
  that was not exhaustively swept inside template bodies). The v2
  paths are now archived under `multi-agent/workflows/archived/`, so
  pasting these template bodies verbatim into a v4-launched prompt
  would (a) point the receiving agent at an archived file, and (b)
  signal the wrong workflow generation. **Before emitting any
  cold-pasted template body**, the Orchestrator (or whoever drafts the
  prompt) MUST normalize all `-v2.md` self-references in the pasted
  body to either `-v3.md` (v3 still active) or `phase-development-system-v4.md`
  (once v4 promotes). This includes `Read … and follow it` lines, `re-read …`
  lines, `You are acting as {ROLE_NAME} from …` lines, and any
  workflow-source citation embedded in the prompt body. The cleanup
  also lands as part of Pass 4c when the prompt bodies are inlined into
  v4 § 14 directly (see Appendix A reviewer checklist + Pass 4c
  prerequisite list).
- **Templates inherit v4 § 12 conventions.** Every template's handoff
  prompt + commit block follows the rules in § 12 (5-rule handoff
  prompt convention; Form A vs Form B commit blocks; git-discipline
  principles).
- **Orchestrator advisory pattern.** Every cycle-execution handoff
  prompt (Templates B / C / H emit handoffs at end-of-round; Template I
  emits the kickoff handoff for cycle 1; Templates D / E / F / G emit
  handoffs as the wrap-up loop progresses) gets a preceding Orchestrator
  advisory block per § 12 rule 4.
- **STRATEGY.md drives agent assignments.** The Orchestrator advisory
  in each handoff names the next assignee from `PHASE<N>_STRATEGY.md`'s
  cycle table (R1, R2, R3 columns) or Final Overseer row. The Mapping
  table in § 12 rule 4 is the role + template mapping.

---

# 15. 2-Role Mode

If only two agents are available (instead of three):

1. Role 1 performs both the initial audits and the final wrap-up audit.
2. Role 2 performs implementation plus in-situ supplemental auditing.
3. Keep the same file-update and closeout rules (SPEC stays clean,
   AUDIT_LOG holds round-by-round history, one CHANGELOG entry per
   cycle at closeout).
4. Skip-reaudit declaration by Role 2 still applies, and Role 1 still
   performs lightweight spot-check at closeout if Principal accepts the
   skip.
5. The Orchestrator role still spans the phase; the only difference is
   that Final Overseer collapses into Role 1 (the same fresh chat that
   does cycle audits also does the wrap-up audit). The independence
   discipline applies — Final Overseer's wrap-up audit chat should be
   structurally independent from the Orchestrator's accumulated frame
   even when filled by Role 1.

---

# 16. Notes / migration paths

## Notes for working with this file

This file works best when you tell each agent its role before asking
it to read the file. The workflow is intentionally more structured
than a raw historical prompt, but the prompt templates in Section 14
are designed to preserve the behavior of a successful human-tested
multi-agent loop.

The Principal does not need to read every section before launching a
phase. The Orchestrator (typically Claude Code Opus Max) reads the
file end-to-end at phase kickoff and references specific sections as
each stage transition arrives. The Principal can spot-check sections
when curious or when a specific question arises (e.g., "what is the
substantive vs thin priority distinction?" → § 3).

## Migration paths

### In-flight v3 → v4

**Don't, until v4 stabilizes.** v3 (the two-file system at
`spec_plan_three_role_audit_loop-v3.md` + `phase-development-system_PDS-v3.md`)
remains the active default. v4 is a DRAFT being developed alongside
v3. Once v4 stabilizes through review + at least one phase of trial
use, future phases may use v4. In-flight phases finish under v3.

If a future phase uses v4 directly (no v3 history), the v4 file is
**self-contained at the policy / stage-structure layer** — read it
end-to-end as the phase begins. **Prompt bodies for the 22 templates
remain cross-referenced to v3 source files until Pass 4c lands** (see
§ 14). For full prompt-layer self-containment, either wait for Pass 4c
or paste the relevant template body in from v3.

### In-flight v2 → v4

Skip v3 entirely. The v3 split (audit-loop-v3 + PDS-v3) was a way-station
that v4 consolidates. If a phase that started under v2 wants to migrate
to v4, the migration is the same as v2 → v3 (which itself is mostly
a name-update on workflow file references) plus reading the new v4
Orchestrator-role section.

### In-flight v1 → v4

Move embedded audit blocks from the SPEC into a new sibling
`AUDIT_LOG.md` file. Do not mutate historical CHANGELOG entries. Read
the v4 Orchestrator-role section (§ 2). Otherwise the migration
follows the same pattern as v1 → v2 (which introduced the AUDIT_LOG
sibling concept).

## Phase startup checklist

When opening a new phase under v4:

1. **Stage 1 (SOUP).** If a candidate SOUP exists in
   `multi-agent/plans/next/<THEME>_SOUP.md`, the Principal +
   Orchestrator decide to promote it to be the next phase's starting
   point. Promotion: `git mv` to `multi-agent/plans/PHASE<N>_<THEME>_SOUP.md`;
   one-time SOUP ID labeling pass (§ 4).
   - If no SOUP, can start at Stage 2 directly (Orchestrator authors
     `PHASE<N>_BRAINSTORM.md` from scratch).
2. **Stage 2 (BRAINSTORM).** Author `PHASE<N>_BRAINSTORM.md` from the
   labeled SOUP (§ 5); seed `PHASE<N>_FEEDBACK.md`; iterate.
3. **Stage 3 (SPEC).** Engineer `PHASE<N>_SPEC.md` from BRAINSTORM
   (§ 6); SPEC engineering rounds; archive BRAINSTORM + FEEDBACK at
   SPEC lock; create skeleton `PHASE<N>_AUDIT_LOG.md`.
4. **Stage 4 (Strategist kickoff).** Orchestrator produces
   `PHASE<N>_STRATEGY.md` per Template I (§ 14); cycle table + agent
   assignments + Final Overseer entry (§ 7). **Also create empty
   `multi-agent/plans/PHASE<N>_SURPRISE_LOG.md`** before the first
   Template A launcher — required sibling artifact whose presence is a
   precondition for Templates A/B/C carry-over scans (see § 1
   "SURPRISE_LOG required + BACKGROUND optional"). Use the empty-file
   skeleton from § 16 "Per-phase sibling-file skeleton template."
5. **Stage 5 (Cycle execution).** First R1 launcher emitted via
   Template A; iterate through all cycles per the standard execution
   loop (§ 8).
6. **Stage 6 (Final Overseer).** Final Overseer pass after last
   execution cycle closes (§ 9).
7. **Stage 7 (Succession Briefing).** Outgoing Orchestrator writes
   `PHASE<N>_SUCCESSION_BRIEFING.md` per Template J after Final
   Overseer signs off (§ 10).
8. **Stage 8 (Phase Archive).** Principal `git mv`s artifacts to
   `multi-agent/plans/archived/<YYYYMMDD>-PHASE<N>_*.md` (§ 11).

## Per-phase sibling-file skeleton template

When creating `PHASE<N>_AUDIT_LOG.md` at the SPEC-lock transition
(end of Stage 3 / start of Stage 4):

```markdown
# PHASE <N> AUDIT LOG

Sibling to `multi-agent/plans/PHASE<N>_SPEC.md`. Captures
round-by-round history of the audit-implement-reaudit loop for this
phase under the v4 phase-development workflow.

## Cycle: <first cycle name> — OPEN (awaiting Role 1 initial audit)
```

When creating `PHASE<N>_STRATEGY.md` at Stage 4 kickoff: use Template
I (Section 14) to generate.

When creating `PHASE<N>_BACKGROUND.md` (optional; only if the phase
develops non-trivial decision history during cycle execution): start
informal; promote to a per-phase planning surface when the
deliberation captures non-trivial decision history worth preserving
(per `multi-agent/AGENT_CONVENTIONS.md § Per-phase planning
surfaces`).

When creating `PHASE<N>_SURPRISE_LOG.md` (**required**; created empty
**at Stage 4 (Strategist kickoff), before the first Template A
launcher fires**, even if the phase ends up with no entries filed):
see `multi-agent/AGENT_CONVENTIONS.md § SURPRISE_LOG.md` for the full
convention. Status taxonomy: every entry passes through `Active` →
`In-flight` → `Resolved (vR.X.YY)`, with side states for promotion /
supersession / logged-only.

The file MUST exist before any cycle's Templates A/B/C run because
those templates reference `Surprise log file:` and require carry-over
scans that are non-skip-able. An empty SURPRISE_LOG.md (just the
header) is acceptable; an absent file breaks template invariants.
Stage 4 is the canonical creation point so that the first cycle's R1
launcher inherits a present-but-empty file (one creation point, not
"at Stage 4 or at the first cycle's R1 round" — that ambiguity was
removed in Pass 5d).

Skeleton template for an empty `PHASE<N>_SURPRISE_LOG.md` at Stage 4
creation:

```markdown
# PHASE <N> SURPRISE LOG

Sibling to `multi-agent/plans/PHASE<N>_SPEC.md` and
`PHASE<N>_AUDIT_LOG.md`. Holds fresh-eyes observations that did NOT
make it into the formal AUDIT_LOG report — out-of-scope, hygiene-drift-
adjacent, smells without concrete repair, cross-cycle patterns. See
`multi-agent/AGENT_CONVENTIONS.md § SURPRISE_LOG.md` for the full
convention. Status taxonomy: `Active` → `In-flight` → `Resolved
(vR.X.YY)` with side states for promotion / supersession / logged-only.

(No entries filed yet.)
```

## v4 stability + future v5

v4 is currently a DRAFT (see file header). Stabilization criteria:

- v4 has been used as the active workflow for at least one full phase
  end-to-end (Stage 1 through Stage 8).
- The next-phase Orchestrator successfully bootstrapped from the prior
  phase's Succession Briefing.
- No structural defects surfaced during use that required a v4.x
  in-place revision (small clarifications are fine; structural
  defects requiring a v5 are not).

After stabilization: v4 becomes the active default; v3 files move to
`multi-agent/workflows/archived/` (parallel to v1 + v2). Agent files
+ AGENT_CONVENTIONS.md update v3 references → v4. v5 considerations
deferred until v4 has been stable across multiple phases.

---

# Appendix A — v3 → v4 mapping

This appendix maps every v3 section to its v4 location so reviewers can
confirm no content was lost in the merger. Use it during v4-DRAFT
review to validate the merger before promoting v4 to active.

## Audit-loop-v3 → v4

`multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md` is ~2700
lines as of v4-DRAFT creation. Section-by-section mapping:

| Audit-loop-v3 section | v4 section | Notes |
|---|---|---|
| Header + revision markers (lines 1-30) | v4 header + revision markers | Updated to reflect v4 in lineage; v3 framing preserved as historical note |
| § Purpose (lines 45-65) | v4 § 1 Conceptual framing (introductory paragraphs) | Reframed to describe full PDS, not just audit-loop |
| § When To Use | (omitted; v4 is intended to be the unified target workflow once promoted) | v4 doesn't need a "when to use" section — it is being engineered as the unified target workflow that subsumes audit-loop's prior single-purpose framing. v3 remains the active default until v4 promotes; the omission rationale is forward-looking, not a claim that v4 is already active. |
| § Required Inputs (lines 82-130) | v4 § 1 "Required inputs / variable definitions (cycle execution)" subsection | **Full migration** (added in Pass 5b). Single canonical home at the end of § 1 — TARGET_FILE / AUDIT_LOG_FILE / STRATEGY_FILE / TARGET_CYCLE / UPDATE_FILES / round-type / mode + suggested assignment-line scaffold. Generalized for v4's single-file system; under v4 the Orchestrator drafts prompts so the Principal need not handle these directly. |
| § The Orchestrator (phase-spanning AI role; v3) (lines 132-510) | v4 § 2 The Orchestrator role | **Full migration.** Lifecycle reframed to map to v4 Stages 1-8. All subsections preserved (why role exists, lifecycle, entry point starts at SOUP, multi-chat continuity, deliberation pattern, boundaries, interaction model, recommended model + agent choice, decision guide for cycle transitions, what gets archived at phase close). |
| § Pre-cycle: Phase Strategy (lines 514-533) | v4 § 7 Stage 4: Strategist kickoff | Migrated as Strategist kickoff being Orchestrator's kickoff act |
| § Cycle Granularity — substantive vs batched (lines 535-610) | v4 § 3 Substantive priorities + cycle granularity | Full migration |
| § Target File Structure And Editing Conventions (lines 614-705) | v4 § 8 Stage 5 (subsections "AUDIT_LOG.md structure" + "Required file updates per cycle") | Reorganized into Stage 5 |
| § Rules for updating CHANGELOG (lines 724-775) | v4 § 8 Stage 5 (subsection "Shared rules for every cycle-execution role") + § 12 Authorship + commit-block conventions | Distributed across § 8 + § 12 |
| § Roles (lines 777-1273) | v4 § 8 Stage 5 (Role 1, Role 2 subsections) + § 9 Stage 6 (Role 3 subsection) | Role 1 + Role 2 in cycle execution; Role 3 in Final Overseer stage |
| § Shared Rules — v2 (lines 1273-1308) | v4 § 8 Stage 5 (subsection "Shared rules for every cycle-execution role") | Full migration |
| § Standard Execution Loop (lines 1310-1414) | v4 § 8 Stage 5 (subsection "Standard execution loop") + § 9 Stage 6 (Step 6) + § 9 Stage 6 (Step 7 post-wrap-up remediation) + § 11 Stage 8 (Step 8 phase archive) | Steps 1-5 → § 8; Steps 6-7 → § 9; Step 8 → § 11 |
| § Required File Updates (lines 1414-1456) | v4 § 8 Stage 5 (subsection "Required file updates per cycle") | Full migration |
| § Acceptance And Closeout Rules (lines 1458-1474) | v4 § 8 Stage 5 (subsection "Acceptance and closeout rules") | Full migration |
| § Common Failure Modes (lines 1476-1507) | v4 § 13 Common failure modes (v2-introduced subsection) + new v3+v4 failure modes subsection | Full migration of the 16 v2-introduced modes; 7 new v3+v4 modes added |
| § Prompt Templates (lines 1510-2750) | v4 § 14 Prompt templates (templates-index with v3 cross-references) | DRAFT phase: index + cross-references to v3 source; full inlining deferred to Pass 4c after v4 stabilizes |
| § 2-Role Mode (lines 2760-2769) | v4 § 15 2-Role Mode | Full migration + v4 addition (Final Overseer collapses into Role 1; independence discipline still applies) |
| § Notes For The User (lines 2773-2800) | v4 § 16 Notes / migration paths | Full migration + v4 stability criteria + migration paths for v1/v2/v3 → v4 |

## PDS-v3 → v4

`multi-agent/workflows/phase-development-system_PDS-v3.md` is ~1100
lines as of v4-DRAFT creation. Section-by-section mapping below.

**Source-line-range policy:** **section names are authoritative; line
numbers are not maintained.** v3 source files may shift line numbers
across patches; the `## <section name>` headings are the stable
reference. Reviewers should grep `^## <section name>` in the v3 source
to locate a row's source. (Earlier rounds of v4 development included
explicit `(lines N-M)` ranges in mapping rows; some had drifted by
Pass 5f — they were dropped to avoid creating a maintenance burden.
This policy applies to the audit-loop-v3 mapping table above as well;
ranges there are best-effort but not maintained.)

| PDS-v3 section | v4 section | Notes |
|---|---|---|
| Header + revision markers | v4 header + revision markers | Updated to reflect v4 |
| § The Orchestrator role spans this file (v3-added) | v4 § 2 The Orchestrator role | Already cross-referenced audit-loop-v3 § Orchestrator under v3; v4 inlines the canonical version |
| § What PDS is | v4 § 1 Conceptual framing (subsection "What PDS is") | Migrated; expanded to describe v4 as the unified system |
| § How the Phase Development System Works (Four important files / Three main stages) | v4 § 1 Conceptual framing (subsections "Phase artifacts produced + consumed during the 8-stage lifecycle" + "The eight stages at a glance") | Reorganized as stage-by-stage table; "three main stages" reframed as eight stages under v4's lifecycle spine |
| § Substantive priorities (new in v2) | v4 § 3 Substantive priorities + cycle granularity | Full migration; consolidated with audit-loop-v3 § Cycle Granularity |
| § Phase 0 — Closed priorities through § Phase 5 — Closeout (Phase 14 Supplemental example) | v4 § 3 (subsection "Phase-group SPEC structure") | Phase 14 Supplemental example reproduced as the skeleton in Pass 5b's Item H |
| § Phase Development Rules | v4 § 8 Stage 5 "Phase Development discipline rules (stale-context handling)" subsection | **Full migration** (added in Pass 5c). Verbatim ~85-line subsection placed after the 14 Shared rules and before AUDIT_LOG.md structure. Eight subsections (Before-decisions / for-each-modify / do-not-rely-on / only-trust-current / before-CHANGELOG-DEVLOG-insertion / after-changes / on-cold-starts / after-finishing-up + before-wrap-up + compaction-awareness + thorough-during-BRAINSTORM-and-SPEC-engineering + aim-for-substantive-priorities). Cross-referenced from Stage 2 + Stage 3 (Pass 5d) so upstream-stage agents follow the same discipline. |
| § Typical form of the Opening brainstorm prompt to Agent 1 from Orchestrator, soup-to-brainstorm optional | v4 § 14 Prompt templates → Template 0a (cross-referenced) | DRAFT phase: index entry; full body in PDS-v3 |
| § Typical form of the soup-to-brainstorm AUDITOR prompt to Agent 2 from Orchestrator | v4 § 14 → Template 0b (cross-referenced) | Same |
| § Typical form of the soup-to-brainstorm FOLLOW-UP prompt to Agent 1 from Orchestrator | v4 § 14 → Template 0c (cross-referenced) | Same |
| § Typical form of the soup-to-brainstorm CLOSEOUT prompt to Orchestrator's chosen agent | v4 § 14 → Template 0d (cross-referenced) | Same |
| § Typical form of the Opening brainstorm prompt to Agent 1 from Orchestrator, after soup-to-brainstorm transition or when there is no soup | v4 § 14 → Template 0e (cross-referenced) | Same |
| § Typical form of the follow-up opening brainstorm prompts to Agents 2+ from Orchestrator | v4 § 14 → Template 0f (cross-referenced) | Same |
| § Typical form of the returning BRAINSTORM prompts to Agent 1 from Orchestrator | v4 § 14 → Template 0g (cross-referenced) | Same |
| § Typical form of the returning BRAINSTORM prompts to Agent 2+ from Orchestrator | v4 § 14 → Template 0h (cross-referenced) | Same |
| § Typical form of opening SPEC ENGINEERING stage prompt to Agent 1 from Orchestrator | v4 § 14 → Template 0i (cross-referenced) | Same |
| § Typical form of subsequent SPEC ENGINEERING stage prompt to Agent 2 from Orchestrator | v4 § 14 → Template 0j (cross-referenced) | Same |
| § Typical form of subsequent SPEC ENGINEERING stage prompt to Agent 1 from Orchestrator | v4 § 14 → Template 0k (cross-referenced) | Same |
| § Typical form of additional rounds of SPEC ENGINEERING stage prompts to Agent 2 from Orchestrator | v4 § 14 → Template 0l (cross-referenced) | Same |
| § MOVING ON TO THE "AUDIT / IMPLEMENT" CYCLES | v4 § 6 Stage 3 (closeout) + § 7 Stage 4 (kickoff) | Stage transition; covered in stage flow |
| § Typical form of Opening Prompt to "The Strategist" before kicking off the "AUDIT / IMPLEMENT" cycles -- from Orchestrator | v4 § 14 → Template I (cross-referenced) | Same |
| § Typical form of Opening Prompt for "AUDIT / IMPLEMENT" cycles to Agent 1 from Orchestrator | v4 § 14 → Template A (cross-referenced) | Same |

## Orchestrator companion file (v2; archived under v3) → v4

The v2 file `orchestrator.spec_plan_three_role_audit_loop-v2.md`
(originally a copy-paste prompt source for the human orchestrator) was
deprecated under v3. Its useful content (Decision Guide for cycle
transitions; Agent Selection and performance tuning) was migrated:

| Companion-file section | Destination |
|---|---|
| § Decision Guide for the Orchestrator | v4 § 2 The Orchestrator role (subsection "Decision guide for cycle transitions") + (subsection "Deciding whether to accept a Role 2 skip-reaudit") — same as audit-loop-v3 location |
| § Agent selection and performance tuning | v4 § 14 Template I (Strategist) — Agent tier roster + baseline assignments + per-role recommendations live inside Template I's body in audit-loop-v3 (currently cross-referenced from v4) |
| § Standard Copy-Paste Prompts (Templates 0-7 + 3b) | NOT MIGRATED — duplicative with audit-loop-v3 Templates A-J. AI Orchestrator drafts cycle-specific custom prompts; standard templates aren't directly matchable |
| § Concrete Examples (Phase 15 Start, Batched Cycle, After Final Overseer) | NOT MIGRATED — historical orientation only; v4 § 8/9 stage descriptions already convey the same flow |

## Items v3 had that v4 omits

Honest list of v3 content not yet present verbatim in v4 (DRAFT-phase
gaps; each is intentional + has a remediation path).

- **Full prompt-body inlining for all 22 template bodies (0a-0l + A-J;
  12 upstream-stage templates + 10 cycle-execution / Strategist /
  Final-Overseer / Succession templates).** v4 § 14 is a
  templates-index-with-cross-references; the canonical prompt prose
  lives in v3 source files (~1700 lines). Rationale: migrating verbatim
  during DRAFT pushes v4 to ~3700+ lines before review. Pass 4c
  (deferred) will inline all 22 template bodies after v4 stabilizes
  through one phase of trial use.
- **Template I § Agent selection and performance tuning — agent tier
  roster (haiku / sonnet / opus / extra-high) + per-role baseline
  recommendations.** Currently cross-referenced from audit-loop-v3
  Template I; the roster table itself is not reproduced inline in v4 §
  14. Will land with Pass 4c when Template I body is inlined.
- **PDS-v3 § Standard Copy-Paste Prompts (Templates 0-7 + 3b)** and
  **PDS-v3 § Concrete Examples (Phase 15 Start, Batched Cycle, After
  Final Overseer)** — intentionally NOT MIGRATED (duplicative with
  audit-loop-v3 Templates A-J + already covered by v4 § 8/9 stage
  descriptions). Documented in mapping table above with NOT MIGRATED
  flag; not a regression.
- (Reviewers: if you find OTHER content in v3 missing from v4 without a
  documented row in this appendix, surface it as a finding. The four
  bullets above are the known gaps as of Pass 5c; treat any further
  omission as unintentional and worth flagging.)

## Items v4 adds beyond v3

These are net-new in v4:

1. **The eight-stage lifecycle spine** (Stages 1-8 as a unifying
   narrative thread). v3 had a "three main stages" framing in PDS-v3
   that was less granular. v4's eight stages cleanly map every
   workflow operation to a stage.
2. **§ 1 Conceptual framing — phase artifacts table.** A stage-by-stage
   table showing what artifacts are produced, actively read/amended,
   and archived at each stage. v3 had this information distributed
   across multiple sections; v4 consolidates as a single reference
   table.
3. **§ 1 Conceptual framing — eight stages at a glance.** A one-line
   summary per stage. v3 didn't have this overview.
4. **§ 13 v3+v4 failure modes (7 new entries).** Phase 15 surfaced
   recurring patterns (GAP-1 conflation; help-text contract
   simplification; empirical-default overfit; cross-pipeline parity
   blindspot; authorship/Effort attribution drift; Orchestrator role
   drift toward implementation; mid-flight pushback notes that bypass
   Principal authorization). v3 had the v2-introduced 16 failure modes
   only.
5. **§ 16 v4 stability criteria + v5 considerations.** Explicit
   criteria for v4 → active default transition, plus deferred v5
   thinking.
6. **Appendix A — v3 → v4 mapping table** (this appendix). Aids
   reviewer validation of the merger.
7. **Cleaner narrative flow.** v4's stage-by-stage organization is
   easier to read end-to-end than v3's two-file split where the
   reader had to mentally bridge from PDS-v3 stages to audit-loop-v3
   roles.

## Reviewer checklist for v4 → active promotion

Before promoting v4 from DRAFT to active default, reviewer should
confirm:

- [ ] No content from v3 is lost (per audit-loop-v3 → v4 + PDS-v3 → v4
  mappings above).
- [ ] All 8 stages are documented + have clear transitions.
- [ ] The Orchestrator role definition is consistent with how it's been
  used in Phase 15 (cycle-spanning AI agent + Principal partnership;
  multi-chat continuity; bidirectional deliberation).
- [ ] § 12 conventions match current practice (handoff prompts, commit
  blocks, authorship consolidation).
- [ ] § 13 failure modes capture the patterns observed in Phase 15.
- [ ] § 14 templates index covers all 22 template bodies (0a-0l + A-J;
  12 upstream-stage + 10 cycle-execution / Strategist / Final-Overseer /
  Succession).
- [ ] Pass 4c (full template body inlining) is identified as a
  prerequisite if reviewer wants self-contained v4 before promotion;
  OR cross-references to v3 are acceptable interim if reviewer agrees.
- [ ] Migration paths in § 16 are correct.
- [ ] No active stubs / TODOs in the body (all placeholders resolved).

Once these are confirmed, v4 can be promoted to active:

1. Update agent files (CLAUDE.md, AGENTS.md, GEMINI.md,
   .github/copilot-instructions.md) to reference v4 as current default
   + mark v3 as deprecated/archived.
2. Update `multi-agent/AGENT_CONVENTIONS.md` v3 path references → v4.
3. Move v3 files to `multi-agent/workflows/archived/`:
   `spec_plan_three_role_audit_loop-v3.md` and
   `phase-development-system_PDS-v3.md`.
4. Inline full template bodies into v4 § 14 (Pass 4c, if not already
   done).
5. Optional: write v4 stability DEVLOG entry summarizing the
   promotion.
