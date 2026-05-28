# CLAUDE.md

## Purpose
You are continuing development of **onionskin**, a Python package for detecting,
characterizing, and analyzing developmental DNA amplification domains from bedGraph
coverage data. Treat paths in this document as relative to the repository root
(`onionskin/`) unless otherwise stated.

## Required reading — tiered by session type

### Always read first (every session)
1. **`multi-agent/project_context/HANDOFF.md`** — session bookmark: last action, current
   state, next action
2. **`multi-agent/project_context/TASK.md`** — immediate task queue and blockers

### Read only when relevant (not every session)
3. **`ROADMAP.md`** — **retrospective + light-reading bird's-eye echo** of what
   the phase plans are doing. NOT a forward-looking compass. The phase system
   itself (SPEC + AUDIT_LOG + STRATEGY + FEEDBACK files in `multi-agent/plans/`
   and `multi-agent/plans/archived/`) is the actual roadmap; CHANGELOG
   `**Roadmap:**` lines that say `Phase X — Phase Y` are pointing at phase
   plans, not at sections in `ROADMAP.md`. Use `grep -A 20 "Phase <N>"` to find
   a specific closed-phase entry; do not read the full file unless planning
   across phases. New ROADMAP entries get added when a SPEC formalizes
   (not at brainstorm/SOUP stage); see `AGENT_CONVENTIONS.md § ROADMAP.md`
   for the full role.
4. **`CHANGELOG.md`** (product changes) and **`multi-agent/DEVLOG.md`** (dev-system
   changes) — read the **last 2 entries only** using `offset`/`limit`; use
   `grep -n "\[v0\." <file> | tail -5` first to find the right line numbers;
   do not read the full file. Split took effect at v0.14.64 — see `CHANGELOG.md`
   preamble for routing rules. Read whichever matches your current task (product
   vs dev-system); read both if you need cross-session context.
5. **`multi-agent/AGENT_CONVENTIONS.md`** — read once per session when you will write
   a CHANGELOG entry or commit; skip if the session involves no deliverables
6. **`multi-agent/tracking/KNOWN_ISSUES.md`** — read when suggesting next steps, closing out a Priority
   or Phase, or choosing among near-term side quests
7. **`multi-agent/full_instructions/PIPELINE_SPEC.md`** — only when working on pipeline
   steps, normalization, detection, output schemas, or CLI flags
8. **`multi-agent/plans/PHASE<N>_SPEC.md`** — active detailed phase plan/spec
   (conditional on a phase being active). If `HANDOFF.md` cold-start says a
   phase is currently active, read this file as the detailed current-phase
   source of truth when working on that phase. If `HANDOFF.md` says the
   project is in inter-phase mode (no phase active; previous phase closed +
   archived), skip this read — see § Inter-phase development mode below.
   Historical phase plans are archived at `multi-agent/plans/archived/<YYYYMMDD>-PHASE<N>_*.md`
   (e.g., Phase 14 Supplemental at `multi-agent/plans/archived/20260427-PHASE14_SUPPLEMENTAL-SPEC.md`,
   Phase 15 at `multi-agent/plans/archived/20260505-PHASE15_*.md`).
8a. **`multi-agent/plans/PHASE<N>_AUDIT_LOG.md`** — sibling to the active SPEC
   under the v2/v3 workflow (see "Workflow files" section below; conditional
   on a phase being active). When a phase is active, read this for round-by-
   round audit/implementation/closeout history. The SPEC stays static during
   implementation; AUDIT_LOG is the moving document. Use `offset`/`limit` or
   `grep -n "^## Cycle:" PHASE<N>_AUDIT_LOG.md` to read only the current
   cycle's section — do not read the whole file unless you need cross-cycle
   history. Skip this read in inter-phase mode (no active AUDIT_LOG to read).
9. **`multi-agent/tracking/BRAINSTORM.md`** — long-lived ideas reservoir
   (tier 1 in the four-tier hierarchy in `AGENT_CONVENTIONS.md § Future-phase
   planning surfaces`). Read when exploring future directions, naming
   questions, synthesis ideas, or work that is not yet committed to the
   active phase spec/plan. Distinct from per-phase `PHASE<N>_BRAINSTORM.md`
   formalization staging files (tier 3) and from SOUP files (tier 4 — see
   entry 9a below).
9a. **`multi-agent/plans/next/`** — candidate future phases in early-stage
   SOUP form (`<THEME>_SOUP.md` files) plus the orchestrator's free-form
   `PHASES.txt` scratchpad and any queued `DEVPLAN-<topic>.md` dev-system
   plans. Read when the orchestrator is planning which theme to bring onto
   the stage next, or when contributing fresh ideas to a SOUP file. SOUP
   files MUST NOT mention phase numbers in their bodies — flag any hits
   for cleanup. See `AGENT_CONVENTIONS.md § Future-phase planning surfaces`
   for the SOUP convention + SOUP→BRAINSTORM→SPEC lifecycle.

### Read only on cold start or when blocked
10. **`multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md`** — full project context:
   architecture, module list, design decisions, test matrix, biological background, gotchas.
   **Skip if HANDOFF.md gives sufficient orientation.** Read in full only when starting
   from zero context or genuinely blocked on architecture questions.

### Inter-phase development mode

When the project is between formal phases — `HANDOFF.md` cold-start says no
phase is currently active and the previous phase is closed + archived —
development is intentionally unstructured. Targeted surgical work, one
task at a time, conversational rather than ceremony-heavy.

**Directory-level mode indicator** (structural backstop for HANDOFF cold-start):

- **Phase mode:** `multi-agent/plans/PHASE<N>_*.md` files exist at the top
  level of `multi-agent/plans/`. The active phase plan + its siblings
  (AUDIT_LOG, STRATEGY, etc.) live there.
- **Interphase mode:** no `PHASE<N>_*.md` files at top level of `plans/`.
  Active inter-phase items (if any) live in `multi-agent/plans/interphase/<TOPIC>.md`.
  Empty `interphase/` plus no top-level phase files = project is idle
  between phases.

When in doubt, defer to `HANDOFF.md` cold-start as the explicit signal; the
directory structure backstops it.

**In practice during inter-phase periods:**

- The user may surface a single concrete change ("fix X", "add Y flag",
  "investigate Z behavior") in chat. Discussion may resemble an R1 audit
  pattern but stays conversational; no formal SPEC / AUDIT_LOG / STRATEGY
  artifacts required.
- Implementation may be done by an agent (e.g., Copilot) like an R2;
  orchestrator may verify like an R3 from a different session — but these
  roles are conventional, not contractual. There is no SPEC contract to
  verify against; the user's stated intent is the contract.
- Planning notes + work-in-progress for inter-phase tasks land in
  `multi-agent/plans/interphase/<TOPIC>.md` (filename is loose; topic-focused;
  no required prefix). When complete, `git mv` to
  `multi-agent/plans/archived/<YYYYMMDD>-<TOPIC>.md` — same archive bucket
  as phase archives, keeping chronology preserved across modes.
- Tasks land via normal commits + CHANGELOG entries (or `multi-agent/DEVLOG.md`
  if dev-system) — no per-cycle bookkeeping required. Version bumps follow
  the standard `vR.X.YY` rule (`YY` increments per CHANGELOG entry).
- Near-term concrete priorities live in `multi-agent/tracking/KNOWN_ISSUES.md`;
  longer-term ideas in `multi-agent/tracking/BRAINSTORM.md`; themed future-phase
  candidates in `multi-agent/plans/next/<THEME>_SOUP.md`.
- When inter-phase work accumulates (multiple related tasks; a unifying
  theme emerges; substantive scope discovered), the orchestrator + Principal
  may decide to graduate it into a formal phase via the SOUP→BRAINSTORM→SPEC
  lifecycle. When a new phase opens, active items in `interphase/` should
  either (a) be archived if complete or no longer relevant, or (b) graduate
  into the new phase's planning if directly absorbed.

See `multi-agent/plans/interphase/README.md` for the directory's filename +
lifecycle conventions in more detail.

The phase-mode discipline (the rest of this document + `multi-agent/AGENT_CONVENTIONS.md`)
remains the reference for HOW phases are run when they are active. Inter-phase
mode does not invalidate or replace any of that — it just acknowledges that
useful project work also happens outside formal phase ceremony.

### Planning model
- The **phase system itself** (SPEC files in `multi-agent/plans/` + archived
  ones in `multi-agent/plans/archived/`) is the actual roadmap. The active
  phase SPEC/PLAN file in `multi-agent/plans/` is the detailed source of
  truth for the current phase and is archived intact when the phase ends.
- `ROADMAP.md` is a **retrospective + light-reading** echo of the phase
  system — not a forward-looking compass. Updated when a SPEC formalizes;
  accumulates `✓ DONE (v0.x.xx)` markers as priorities close.
- `multi-agent/plans/next/` holds **SOUP files** (`<THEME>_SOUP.md`) — the
  pre-BRAINSTORM scratchpad for candidate future phases. SOUP files
  promote to per-phase `PHASE<N>_BRAINSTORM.md` in `multi-agent/plans/`
  when the orchestrator brings them onto the stage. See
  `AGENT_CONVENTIONS.md § Future-phase planning surfaces` for the
  SOUP→BRAINSTORM→SPEC lifecycle and the four-tier
  brainstorming-surfaces hierarchy.
- `multi-agent/tracking/KNOWN_ISSUES.md` is the long-lived near-term
  actionable issue registry for side quests and concrete short-horizon
  follow-up work.
- `multi-agent/tracking/BRAINSTORM.md` is the long-lived unstructured
  reservoir for future ideas that are not yet committed (tier 1 in the
  four-tier hierarchy; distinct from per-phase BRAINSTORM staging files
  in `multi-agent/plans/` which are tier 3).
- Under the v2 audit/implement workflow, a sibling `PHASE<N>_AUDIT_LOG.md`
  holds round-by-round history (audit findings, implementation reports,
  closeout judgments) while the SPEC stays clean as the contract. Both
  archive together at phase close.
- Dev-system work plans use `DEVPLAN-<topic>.md` naming (no phase
  numbers; not part of the phase system) — see `AGENT_CONVENTIONS.md
  § DEVPLAN naming convention`.
- `ROADSTRAVELED` is retired; do not use it for new planning work.

### Workflow files — v1, v2, and v3 (pick one per phase; v3 is the current default)

- **v3 (current default):** `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md` +
  `multi-agent/workflows/phase-development-system_PDS-v3.md`.
  Adds the formal **Orchestrator role** — a phase-spanning AI agent that begins
  at SOUP stage, drives BRAINSTORM iteration + SPEC engineering + Strategist
  kickoff + cycle execution + Final Overseer pass + Succession Briefing
  emission at phase close. Multi-chat continuity via memory + planning
  surfaces (the role is conceptually continuous; chat handoffs are normal +
  optional). Strategist function absorbed as the Orchestrator's kickoff act
  (no longer a separate role). New artifact at phase close:
  `PHASE<N>_SUCCESSION_BRIEFING.md` (Template J) — wisdom transfer from
  outgoing Orchestrator to next-phase Orchestrator. The v2
  `orchestrator.spec_plan_three_role_audit_loop-*.md` companion file is
  deprecated under v3; its useful content (Decision Guide for cycle
  transitions) was migrated into
  `spec_plan_three_role_audit_loop-v3.md § Orchestrator`. Future v4 plan:
  merge audit-loop-v3 + PDS-v3 into a unified `phase_workflow-v4.md` after
  v3 stabilizes through one or two phases.
- **v2 (deprecated; archived):**
  `multi-agent/workflows/archived/spec_plan_three_role_audit_loop-v2.md` +
  `multi-agent/workflows/archived/phase-development-system_PDS-v2.md` +
  `multi-agent/workflows/archived/orchestrator.spec_plan_three_role_audit_loop-v2.md`.
  Introduces sibling `PHASE<N>_AUDIT_LOG.md` for round-by-round history; one
  CHANGELOG entry per cycle closeout with consolidated authorship;
  substantive-priorities rule; cycle granularity (substantive vs batched);
  Role 2 may recommend skip-reaudit on mechanical cycles. Phase 14
  Supplemental was mid-stream migrated to v2 (2026-04-23). Phase 15 used v2
  throughout.
- **v1 (deprecated; archived):**
  `multi-agent/workflows/archived/spec_plan_three_role_audit_loop-v1.md` +
  `multi-agent/workflows/archived/phase-development-system_PDS-v1.md` +
  `multi-agent/workflows/archived/orchestrator.spec_plan_three_role_audit_loop-v1.md`.
  Per-role-round CHANGELOG entries; audit trail embedded in the SPEC.
- Pick one system per phase; do not mix.

## Dev rule (critical)
All runs MUST write to:

    dev/runs/<run_name>/

Never write outputs into repo root or tracked directories.

Use:

    bash scripts/dev_run.sh ...

for exploratory runs or debugging.

## Development workflow requirements

### Testing — risk-based matrix

Run only the tests that could plausibly be broken by your changes. State which tests
you ran and why in the session notes or CHANGELOG entry.

| Changed area | Tests to run |
|---|---|
| HMM engine only (`hmm_engine.py`) | `make puff-compare` only |
| Single-mode logic | `make test`, `make single` |
| Multistage growth logic | `make test`, `make toy` |
| RCN mean-shift | `make test`, `make toy` |
| Twin-peak / overlap resolution | `make twin`, `make full-twin` |
| Shape filter / BIC | `make shape-score-bic` |
| CLI flags only (no logic change) | `make test` (smoke only) |
| Cross-cutting (core modules, detection params) | `make test`, `make toy`, `make single`, `make twin` |
| Phase boundary / pre-release | Full suite (see release workflow below) |

**Frequency:** Run targeted tests after each implementation. Run the full suite at
phase boundaries and before releases. `make test-single` (T1-T12) is slow — run
only before releases or when single-mode detection logic changes.

**Test output:** Suppress passing test output to avoid filling context. Use:

    make <target> 2>&1 | tail -20

or redirect passing output and only capture failures:

    make <target> > /tmp/test_out.txt 2>&1 && echo "PASS" || { tail -40 /tmp/test_out.txt; echo "FAIL"; }

For `make puff-compare`, capture the full summary table (it is compact and informative).

### Deliverables

After each implementation:
1. Update `CHANGELOG.md` (product changes) or `multi-agent/DEVLOG.md` (dev-system
   changes) — route by primary content per the rule in `AGENT_CONVENTIONS.md` —
   and update `multi-agent/project_context/HANDOFF.md`.
2. Update `multi-agent/project_context/TASK.md` — mark completed tasks, add next tasks.
3. Ensure no empty output directories unless intentional; outputs must be deterministic.
4. Write new `KNOWN_ISSUES.md` and `BRAINSTORM.md` entries **as they emerge** during the
   session — do not defer to end-of-session. Context compaction can discard deferred items.

## Mandatory release workflow
1. Full release bundles only — never patch-only deliverables
2. Zip root must be a single `onionskin/` directory
3. Permissions must be normal
4. Before releasing, run full suite: `make test`, `make toy`, `make single`, `make twin`
5. For releases with twin-peak or overlap changes, also run:
   `make full`, `make full-no-split`, `make full-twin`
6. Do not claim features are complete unless they are actually wired and emitted

## Scope authority (critical — read before every planning or implementation session)

The **user is the sole authority** on phase and priority scope. Agents do not decide scope.

- **Narrowing is prohibited without user approval.** Do not remove, shrink, or defer any spec
  item without asking first. Do not move intended phase work to `KNOWN_ISSUES.md` without
  explicit user approval. If you think something is risky or complex, raise it as an open
  question — do not silently descope it.
- **Broadening is encouraged.** When you find adjacent work that fits the spirit of the phase,
  surface it and recommend including it. The user is almost always open to broadening.
- **Priorities must be substantive.** Each priority in the audit-implement-reaudit loop must
  justify its token cost. Thin single-item priorities should be consolidated, not accepted.
- **`KNOWN_ISSUES.md` is not a descoping mechanism.** It is for work the user has agreed to
  defer — not for current-phase work an agent decided was too much.

Full rule: `multi-agent/AGENT_CONVENTIONS.md § Scope authority`

## Coding style
- Clear, modular Python
- Prefer deterministic and interpretable logic
- Preserve explicit output naming and schemas
- Avoid vague refactors
- Do not re-ask settled high-level design questions unless blocked
- Only ask questions if they block implementation or involve unavoidable tradeoffs

## Important caution
This project is not a generic CNV caller. It is specifically targeting broad
developmental amplification with summit-centered gradient structure and developmental
progression.

## Claude Code–specific: model selection for subagents

| Task                                                              | Model           |
|-------------------------------------------------------------------|-----------------|
| Most coding, editing, spec updates                                | sonnet (default)|
| Deep cross-module reasoning, math correctness, ambiguous behavior | opus            |
| Lightweight verification, applying known fixes, simple checks     | haiku           |

Escalation: **haiku → sonnet → opus**. Do not escalate unless genuinely uncertain.
Do not make changes when uncertain — escalate first.

## Claude Code–specific: authorship version lookup

When writing an authorship line, the harness version must be sourced from the live
environment — not from memory, training data, or a prior session's value. Use the
method that matches your runtime context:

| Context | Command | Notes |
|---------|---------|-------|
| VSCode extension | `$CLAUDE_CODE_EXECPATH --version` | `$CLAUDE_CODE_EXECPATH` is set by the extension; gives the extension-bundled version. Use this first — it is authoritative for VSCode sessions. |
| Terminal (CLI) | `which claude` then `claude --version` | The CLI is typically homebrew-managed and may lag behind the VSCode extension version. Use when `$CLAUDE_CODE_EXECPATH` is empty. |
| Other harnesses (Codex, Copilot, Gemini, ChatGPT) | No equivalent env var | Ask the user, or read harness-reported version from the UI / end-of-comment metadata. See `AGENT_CONVENTIONS.md § Authorship conventions § Per-agent format` for per-harness guidance. |

**The Effort toggle (High / Max / Extra High) cannot be determined programmatically
in any environment.** It is set in the UI and not exposed to the agent. If it was not
pre-filled in the launcher prompt, ask the user to confirm it as the first immediate
action of the round — before any substantive work.

## Claude Code–specific: token efficiency

### File reads
- Read each file at most once per session AS A DEFAULT; track what you have already read.
  Exception — **do not skip** these re-reads:
  - Workflow-mandated checkpoint re-reads (AGENT_CONVENTIONS.md, your agent file, and the
    active workflow file itself) at the checkpoints defined by the workflow you are
    following. Typically these fire before every CHANGELOG write and before wrap-up
    summaries. They are deliberate quality guardrails against format drift, not
    redundancy.
  - Post-compaction re-reads. If the session has been compacted since your last read of
    a required file, treat yourself as a cold start and re-read.
- Use `Grep` or `Glob` before opening any file — confirm the target exists and find
  the relevant section first
- Never read `CHANGELOG.md`, `multi-agent/DEVLOG.md`, `ROADMAP.md`, or
  `ONIONSKIN_FULL_HANDOFF.md` in full; always use `limit`/`offset` or targeted grep
  (see tiered reading section above)
- Under the v2 audit/implement workflow, read only the current cycle section of
  `PHASE<N>_AUDIT_LOG.md`. Use
  `grep -n "^## Cycle:" multi-agent/plans/PHASE<N>_AUDIT_LOG.md` to locate section
  boundaries, then `offset`/`limit`.
- Do not read files unrelated to the current task

### Subagents
- Use `Grep`/`Glob` directly for any targeted search (known file, class, function, flag)
- Reserve subagents (Agent tool) for open-ended cross-codebase exploration that
  genuinely requires multiple rounds; do not spawn a subagent for a task you can
  complete with 1–3 direct tool calls
- When a subagent is necessary, scope its file list tightly — do not let it re-read
  the full codebase

### General
- Fix all findings in a single pass
- Stop early when no high-confidence findings remain
- Re-read AGENT_CONVENTIONS.md and your agent file at the checkpoints defined by
  the active workflow file (typically before every CHANGELOG write; before wrap-up
  summaries). These re-reads are deliberate quality guardrails against format drift
  — not redundant reads. Compaction voids "already read": if the session has been
  compacted since your last read, treat yourself as a cold start and re-read.
