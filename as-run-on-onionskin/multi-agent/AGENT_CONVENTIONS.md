# Agent Conventions

These documentation and editorial conventions apply to **all agents** working on
onionskin (Claude Code, Codex, Gemini, Copilot, etc.).

**Mode awareness:** the conventions in this file describe how phase-mode
development operates (cycle conventions, SPEC/AUDIT_LOG/STRATEGY discipline,
R1/R2/R3 role separation, deferral discipline, scope authority, etc.). The
project is sometimes between formal phases (inter-phase mode); during those
periods, work is intentionally conversational + lightweight + ceremony-free,
and the structured conventions in this file apply only opportunistically. See
`CLAUDE.md § Inter-phase development mode` for that lighter pattern. When a
phase is active (HANDOFF.md cold-start says so), the conventions below are
load-bearing; when no phase is active, treat them as reference rather than
contract.

**Directory-level mode indicator** (structural backstop for HANDOFF cold-start):
the `multi-agent/plans/` top level encodes the project's current mode.

- **Phase mode:** `multi-agent/plans/PHASE<N>_*.md` files exist at the top
  level. The active phase plan + its siblings (AUDIT_LOG, STRATEGY, etc.)
  live there. Conventions in this file are load-bearing.
- **Interphase mode:** no `PHASE<N>_*.md` files at the top level.
  - Check `multi-agent/plans/interphase/` for live inter-phase work
    (`<TOPIC>.md` files, no required prefix). If non-empty, an inter-phase
    surgical task is in progress.
  - Empty `interphase/` plus no top-level phase files = project is idle
    between phases.

In either mode, `multi-agent/plans/next/<THEME>_SOUP.md` carries
future-phase candidates (themed; speculative) and `multi-agent/plans/archived/<YYYYMMDD>-*.md`
holds closed phase plans + archived interphase items in one chronological
bucket. The progression `next/` → `interphase/` → `PHASE<N>_*.md` (top
level) → `archived/` is a loose lifecycle hint, not a required path.

---

## Critical invariants (must never be violated)

CHANGELOG.md, DEVLOG.md, and ROADMAP.md are authoritative records of project history and direction.

- Never write partial, speculative, or placeholder entries
- Only record finalized, validated changes
- Preserve existing formatting exactly
- Do not rewrite, reorder, or delete historical entries
- When in doubt, do not modify — ask or defer

Violating these rules corrupts project history and is considered a critical error.

---

## File placement — no outputs at the repo root (critical)

Agents must **never** create files or directories at the repository root. This includes
log files, temp files, intermediate outputs, test redirects, and any other artifacts.

**All outputs must go to `dev/`** — specifically `dev/runs/<run_name>/` for any
exploratory run or debugging output. Use `bash scripts/dev_run.sh ...` for these.

Common violations to avoid:
- Redirecting `make test` output to a dot-file at the root (e.g., `.ai_make_test.log`)
- Writing temp score files or intermediate TSVs to the root (e.g., `.tmp_score_calls/`)
- Creating hidden directories at the root for any reason other than standard tooling
  (`.venv/`, `.git/`, `.github/`, `.vscode/`, `.pytest_cache/` are the only acceptable
  hidden entries; all others are violations)

The `.gitignore` already excludes `*.log` and `dev/`, but gitignore is not a license to
litter the repo root. The rule is: **write to `dev/`, not to the root.**

Violations will be cleaned up and the agent responsible will be noted in `AGENT_SWOT.md`.

---

---

## Copy-paste output formatting (required)

Any content the user is expected to copy and paste — handoff launcher prompts, git
command blocks, orientation pointers, scratch-file content, or any multi-line text
meant to be transferred verbatim — MUST be enclosed in a fenced code block (triple
backticks). Plain prose or inline code loses structure on paste and cannot be
one-click-copied.

- **Handoff launcher prompts** → ` ```text ... ``` `
- **Git command blocks** → ` ``` ... ``` ` or ` ```bash ... ``` `
- Any other multi-line copy-paste content → appropriate language fence or plain ` ``` `

VS Code and similar markdown environments render fenced blocks with a copy button.
Inline text or inline code (single backticks) does not get that button. This is a
non-negotiable formatting requirement for all agents on this project.

---

## User Interaction Workflow
After proposing a plan of action (especially one involving file modifications), all agents MUST wait for explicit user approval (e.g., "go ahead," "proceed," "implement the plan") before executing the plan. This ensures all plans are reviewed before any changes are made.

---

## Scope authority — the user controls all scope decisions (critical)

The user is the **sole authority** on what is in scope or out of scope for a phase or priority.
Agents do not decide scope. This rule overrides any agent instinct toward caution or narrowing.

### Narrowing is prohibited without approval

- Agents must **not** remove, defer, or shrink any spec item without explicit user approval.
- Agents must **not** move intended phase work to `KNOWN_ISSUES.md`, out-of-scope notes, or any
  parking area without explicit user approval.
- If an agent believes a scope item is risky, complex, or has an unforeseen dependency, it must
  raise that concern as an **open question to the user** and wait for a decision — then proceed
  with whatever scope the user confirms.
- The default answer to "should we narrow this?" is **no**. Assume the full intended scope is
  wanted unless the user explicitly says otherwise.
- Silently scoping down a priority and moving work to `KNOWN_ISSUES.md` without user approval is
  a **critical failure**.

Case study: In Phase 14 Supplemental v0.14.68, `--hmm-emodel` was retired as a
deprecated alias even though the user had not approved retiring PuffStep HMM synonyms.
Phase 15 SPEC15.1 restored it as a live alias and codified that no PuffStep HMM synonym
may be retired without explicit user approval.

### Broadening is encouraged and should be surfaced

- When agents find adjacent work that fits the spirit of the phase — work the user did not
  explicitly list but would likely want — they should surface it and recommend including it.
- Agents are "in the trenches" with the code and are well-positioned to see work the user did
  not anticipate. Surface that work; do not suppress it.
- The default answer to "should we broaden this?" is **yes — raise it for user approval**.
- Broadening proposals that the user approves are in scope. Broadening proposals the user
  declines are then and only then moved to `KNOWN_ISSUES.md` or `BRAINSTORM.md`.

### Priorities must be substantive

- In the multi-agent audit-implement-reaudit loop, each priority must justify the token cost of
  the full loop. A priority that amounts to a single flag rename, a single line of documentation,
  or a cosmetic help-string tweak is not substantive enough to stand alone.
- When writing or revising a phase spec, aim to encapsulate the **full spirit** of what the user
  is attempting to accomplish — not just the narrow literal items written down.
- Agents should actively propose consolidating thin priorities into substantive ones rather than
  accepting a phase of many tiny items.

### `KNOWN_ISSUES.md` is not a descoping mechanism

- `KNOWN_ISSUES.md` is for genuine future-phase work that the user has **agreed to defer**, not
  a holding pen for current-phase scope that an agent unilaterally decided was too much.
- Items must not be added to `KNOWN_ISSUES.md` as a substitute for doing the current-phase work.
- If an agent believes an item genuinely belongs in a future phase, it must ask the user before
  deferring it.

### Deferral is NOT an ordinary disposition (critical — strict pathology-only rule)

**Default expectation for ALL agents (R1, R2, R3, orchestrator, any role):** implement what was scoped. Deferral is NOT a viable ordinary disposition. The recurring failure mode in Phase 14 + Phase 15 has been agents treating deferral as a default-available menu option ("close PARTIAL + file follow-up `[ISSUE:YYYY-MM-DD:N]`"); this rule eliminates that pattern.

**Sole carve-out: pathology.** The only legitimate reason to NOT implement a scoped deliverable is that implementation would be PATHOLOGICAL — impossible OR genuinely harmful. Concrete pathology examples:

- Implementation requires breaking a locked `DECISIONS.md` design decision.
- Implementation produces incoherent output: cross-pipeline schema mismatch; algorithmic incoherence; data corruption.
- Implementation reveals the original design is structurally flawed (the design itself is pathological — needs SPEC re-engineering, not "deferral").
- Upstream prerequisite is missing on the actual tree (R1 assumed something landed that didn't).
- Implementation would break a CRITICAL gate (`make puff-compare 28/28`, etc.).
- Implementation requires a function/library/data that genuinely does not exist anywhere accessible.

**NOT pathological — these are NOT valid deferral reasons:**

- "Big" / "complex" / "lots of code" / "hard to test."
- "Could be cleaner if we redesigned X first."
- "Out of scope estimate" / "would extend cycle."
- "Low priority for the phase."
- "Dependency not yet imported" — just import it; if it needs adding to requirements, request permission to add it.
- "I see an alternative approach" — that's `[ALTERNATIVE_APPROACH_PROPOSAL]` (see separate section below), not pathology.

**4-step protocol when an agent suspects pathology:**

1. **Raise `[PATHOLOGY_CANDIDATE]` immediately, mid-flight.** Any agent (R1/R2/R3/orchestrator) raises in real time as soon as the concern emerges — do NOT bank concerns to end-of-round. Surface in chat AND in the cycle's audit log entry. Required fields:
   - Deliverable affected (SPEC ID + sub-deliverable; e.g., "SPEC15.15 d3").
   - Concrete pathology evidence (file:line citations; reproducible failure mode; structural incoherence).
   - Why pathological vs merely complex (the affirmative argument that this isn't just "big work").
   - Alternative implementations attempted + why they failed (or why no alternative is workable).
   - Suggested resolution (reject + implement; confirm + landing zone; restructure SPEC; extend cycle).

2. **Block the affected deliverable's work immediately.** Don't continue trying to implement the pathology-flagged item. The cycle's other deliverables continue uninterrupted unless they depend on the blocked one. Strict mode: agent stops mid-flight on the flagged item.

3. **Independent audit.** A different role from the one that raised the candidate audits the claim through code (NOT by accepting reasoning or being persuaded by logic). The audit can be done by:
   - The orchestrator (Claude in real-time chat) if available.
   - A different agent (R3 audits R2's claim; an independent fresh-chat agent audits R1's claim; etc.).
   - The audit produces `[PATHOLOGY_AUDIT_VERIFIED]` (confirms pathology is real) or `[PATHOLOGY_AUDIT_REJECTED]` (disagrees; pathology claim doesn't hold).

4. **Principal decides.** Principal sees both the raise and the audit, decides:
   - **Reject:** confirms `[PATHOLOGY_AUDIT_REJECTED]`; block lifts; agent implements original SPEC. Marker becomes `[PATHOLOGY_REJECTED:IMPLEMENT]`.
   - **Accept with deferral:** confirms `[PATHOLOGY_AUDIT_VERIFIED]` + decides disposition (explicit landing zone with exit conditions; SPEC restructure; new cycle slot; etc.); block lifts per decision. Marker becomes `[PATHOLOGY_CONFIRMED_DEFER:<landing-zone>]`.
   - **Accept with restructure:** confirms verified pathology but rejects deferral; instructs SPEC re-engineering. Marker becomes `[PATHOLOGY_CONFIRMED_RESTRUCTURE]`.

**Marker scheme (use these tokens verbatim in audit log + chat):**

| Marker | Who emits | Meaning |
|---|---|---|
| `[PATHOLOGY_CANDIDATE]` | Any agent | "I think this is pathological. Blocking this deliverable. Awaiting independent audit + Principal decision." |
| `[PATHOLOGY_AUDIT_VERIFIED]` | Independent auditor | "Audit confirms pathology is real. Principal decision needed on disposition." |
| `[PATHOLOGY_AUDIT_REJECTED]` | Independent auditor | "Disagree. Pathology claim doesn't hold. Recommend Principal confirm: agent implements original SPEC." |
| `[PATHOLOGY_REJECTED:IMPLEMENT]` | Principal-confirmed post-audit | Agent implements original SPEC; block lifted. |
| `[PATHOLOGY_CONFIRMED_DEFER:<zone>]` | Principal-confirmed post-audit | Deferral approved; landing zone documented; block lifted per decision. |
| `[PATHOLOGY_CONFIRMED_RESTRUCTURE]` | Principal-confirmed post-audit | SPEC re-engineering required; original SPEC contract amended. |

**Per-role notes (any agent can raise + audit; these are emphasis points):**

- **R1 (initial audit):** never decides "this deliverable can be deferred" while writing the repair contract. If R1 sees something that looks pathological, raise `[PATHOLOGY_CANDIDATE]` mid-audit and STOP writing the R2 contract for that deliverable until the block lifts. R1 does NOT produce R2 instructions for pathology-flagged items.
- **R2 (implementer):** if pathology surfaces during implementation, STOP work on that item, raise mid-flight (real-time chat + audit log), wait for decision. R2 does NOT silently emit a sidecar / placeholder / TODO / partial implementation as a substitute. R2 does NOT propose follow-up `[ISSUE:YYYY-MM-DD:N]` filings as a deferral mechanism — those are silent narrowing.
- **R3 (re-auditor):** **errs strongly toward rejecting deferral requests.** Confirms a deferral only if (a) R2 explicitly raised `[PATHOLOGY_CANDIDATE]` AND (b) R3 independently verifies pathology through code auditing — not by accepting R2's reasoning or being persuaded by logic. If R3 finds R2 silently deferred without raising the candidate, R3 REJECTS the cycle close and instructs R2 to either implement OR raise the candidate properly. R3 does NOT close cycles PARTIAL with deferred deliverables; "PARTIAL" + new ISSUE filing IS silent narrowing dressed up as orchestrator process.

**Cross-agent skepticism (critical).** All agents treat other agents' deferral plans with HIGH skepticism. The recurring failure mode is agents agreeing with each other's plausible-sounding deferral reasoning rather than independently verifying. Counter-discipline: assume the deferral claim is wrong; require independent code-grounded evidence before agreeing. Default toward rejecting other agents' deferral plans/actions.

**Cycle-state implications.**
- A cycle with one or more open `[PATHOLOGY_CANDIDATE]` markers stays OPEN until each is decided.
- Cycles never close PARTIAL with un-audited / un-decided pathology candidates.
- The orchestrator's job at cycle transitions (per `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md` § Orchestrator decision guide) explicitly includes: catching silent deferrals masquerading as PARTIAL closeouts and reopening the cycle when caught.

**Why this rule is strict.** Phase 15 demonstrated repeatedly that "future cycle" deferrals tend to never materialize OR balloon into their own scope problem (cycle 15.7a R3 originally proposed PARTIAL + new ISSUE for SPEC15.15 d3/d4/d6/d8; Principal-orchestrator override 2026-05-04 reopened the cycle for in-cycle Stage G implementation). Cycles with substantive R1 audits already represent significant planning investment; the most honest disposition is "implement what was scoped." The pathology-only carve-out is genuine but rare — most cycles will have ZERO pathology candidates raised; the strict rule prevents the recurring failure pattern without flooding the Principal with trivial requests.

### Alternative-approach proposals — distinct from pathology (separate protocol)

**Concept:** an agent mid-flight sees a markedly better implementation approach than the one specified by the SPEC / R1 audit. NOT a deferral; NOT pathology. It's a SPEC amendment request.

**When to raise:**

- **Concrete reasoning required.** Markedly better runtime, memory consumption, parallelization, structural cleanliness, observed-bug-pattern avoidance, or similar measurable advantage. NOT vague preference.
- **Roughly-equivalent alternatives are NOT raise-worthy.** If the agent's preferred approach is ~equivalent to the SPEC's approach, just implement what was specified.

**Protocol:**

1. **Raise `[ALTERNATIVE_APPROACH_PROPOSAL]` mid-flight** in chat + audit log. Required fields:
   - Deliverable affected.
   - SPEC's specified approach (briefly).
   - Proposed alternative approach (concrete; spec'd out, not vague).
   - Concrete reasoning (runtime/memory/parallelization/bug-avoidance/structural — measurable or specific).
   - Estimated trade-off (e.g., "saves ~5 minutes per run" or "halves memory consumption" or "exposes a bug pattern X already saw twice this phase").

2. **Block the affected deliverable** until decision.

3. **Orchestrator + Principal review.** Independent audit not required (this isn't a pathology claim) — Principal can decide directly based on the proposal's concrete reasoning. Decision options:
   - **`[ALTERNATIVE_ACCEPTED:SPEC_AMENDED]`** — SPEC amendment approved; agent implements per new contract.
   - **`[ALTERNATIVE_REJECTED:IMPLEMENT_ORIGINAL]`** — SPEC stays; agent implements per original.

**Why distinct from pathology:** alternative-approach proposals are about DOING THE WORK BETTER, not about not doing the work. They're a SPEC engineering surface mid-cycle. Conflating them with pathology weakens both protocols — keep them separate.

**Cross-references:**
- `feedback_scope_authority.md` — broader scope-authority rule (this section is a sub-discipline).
- `feedback_deferral_pathology_only.md` — short-form recall version of the strict pathology-only rule.
- Cycle 15.7a Principal-orchestrator override 2026-05-04 (in `multi-agent/plans/PHASE15_AUDIT_LOG.md`) — trigger lesson + worked example of catching a silent deferral and reopening the cycle.
- `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md § Common Failure Modes` — failure-mode entry codifying this discipline in the v3 workflow.

---

## CHANGELOG.md and DEVLOG.md — past-looking records

**Two-file split (prospective from v0.14.64):**

- **`CHANGELOG.md`** (repo root) records **onionskin product changes** — code in
  `onionskin.py`/`onionskin_core/`/`scripts/`/`tests/`, CLI flags, output schemas,
  and user-facing docs (`README.md`, `PIPELINE_SPEC.md`,
  `ONIONSKIN_FULL_HANDOFF.md`), plus conceptual/mathematical evolution of the
  pipeline itself.
- **`multi-agent/DEVLOG.md`** records **dev-system changes** — workflow files,
  `AGENT_CONVENTIONS.md`, agent bootstrap files (`CLAUDE.md`, `AGENTS.md`,
  `GEMINI.md`, `.github/copilot-instructions.md`), SPEC / BRAINSTORM / FEEDBACK /
  STRATEGY / AUDIT_LOG engineering rounds without code changes,
  `AUDIT_HISTORY.md` process rules, `DECISIONS.md`, `project_context`
  shared-memory files when not tied to code, and **any other edits to files
  under `multi-agent/` that aren't code/test/user-doc changes** (this is the
  backstop — anything under `multi-agent/` that doesn't produce a CHANGELOG
  entry produces a DEVLOG entry, if it's worth logging at all).

**Routing rule:** an entry goes in whichever file matches its **primary**
content. Sessions with zero code changes and zero user-facing doc changes go in
`DEVLOG.md`. Sessions that change code, tests, CLI, or user-facing docs go in
`CHANGELOG.md`. Genuinely mixed sessions use the file matching the primary
content with a brief cross-reference note (*"See [other-file] v0.X.YY for the
parallel [product|dev-system] work"*) in the other.

**Default bias: when uncertain, DEVLOG.** `CHANGELOG.md` is reserved for
**actual onionskin program changes** (code, CLI, output schemas, user-facing
docs). Everything else about the work and collaboration — workflow engineering,
planning surfaces, convention edits, audit-trail entries not tied to code,
`multi-agent/` maintenance — goes in `DEVLOG.md`.

**Split version streams (prospective from v0.14.70.1; X-bump trigger revised
2026-04-30).** CHANGELOG and DEVLOG no longer share a single counter; they use
parallel streams that stay aligned via an anchor relationship.

- **CHANGELOG** uses `vR.X.YY` (e.g., `v0.14.70`).
  - `R` = number of Release Bundles shipped. `R=0` while pre-release; bumps to
    `1` at the first Release Bundle ship as part of the release protocol
    (Release Bundle protocol itself is not yet engineered — TBD).
  - `X` = number of fully-completed development phases (equivalently, the most
    recent completed Phase number). During Phase 15 development, `X=14` because
    14 phases were complete. The phase final-closeout CHANGELOG entry bumps `X`
    to the just-completed phase number and is itself versioned `vR.X.00` (for
    Phase 15 close: `v0.15.00`).
  - `YY` = monotonic CHANGELOG-entry counter; increments by 1 with each
    CHANGELOG entry. Resets to `00` at the phase-final-closeout `vR.X.00`
    milestone. Behavior at Release Bundle ship is TBD.
- **DEVLOG** uses `vR.X.YY.Z` (e.g., `v0.14.70.3`). The `vR.X.YY` portion is the
  **anchor** — it matches the latest CHANGELOG version that has been cut at the
  time the DEVLOG entry is written. `.Z` is a per-anchor counter starting at 1,
  incrementing by 1 for each new DEVLOG entry while the anchor remains the same.
  `.Z` resets to 1 each time a new CHANGELOG version is cut.
- **Anchor during a phase before its final closeout**: dev-system entries
  anchor to the latest CHANGELOG version cut so far. Until the phase
  final-closeout entry lands, that anchor stays at `v0.<X-prior>.YY` (e.g.,
  during Phase 15, dev-system DEVLOG anchors to v0.14.YY). The CHANGELOG only
  bumps to `v0.15.00` at Phase 15's final-closeout entry.
- **Total ordering.** A DEVLOG entry's anchor encodes its temporal position:
  v0.14.70.3 unambiguously came after CHANGELOG v0.14.70 and before the next
  CHANGELOG cut. This restores cross-file ordering without leaving gaps in
  either file.
- **`onionskin --version`** reads CHANGELOG only; DEVLOG `.Z` has no effect on
  user-facing version reporting.
- **Cross-references** use the full version string. The component count
  disambiguates the file: 3-component → CHANGELOG, 4-component → DEVLOG.

**Historical note on the X-bump rule.** Prior to 2026-04-30, `X` bumped at the
first product-code-touching entry of a new phase (first product entry of new
phase = `v0.X.00`). On 2026-04-30 the trigger flipped to phase-end: `X` now
bumps at the phase final-closeout CHANGELOG entry. Phases 1–14 closed under
the prior rule and are not retroactively renumbered; Phase 15 is the first
phase to close under the new rule.

**Historical entries (pre-v0.14.64)** were written before the file split and are
mixed; do NOT retroactively migrate. **Entries v0.14.64 through v0.14.69 (DEVLOG)
and through v0.14.70 (CHANGELOG)** used the older shared-stream scheme where
both files drew from one counter. Gaps in either file's version sequence (e.g.,
CHANGELOG v0.14.64–v0.14.67 missing) reflect dev-system-only versions recorded
in DEVLOG under that older scheme. Do not retroactively renumber. The new scheme
starts at v0.14.70.1.

---

- Format (CHANGELOG): `## [vR.X.YY] — YYYY-MM-DD HH:MM EST`
- Format (DEVLOG, new scheme): `## [vR.X.YY.Z] — YYYY-MM-DD HH:MM EST`
- `R` = Release Bundle counter (= number of Release Bundles shipped). `R=0`
  pre-release; bumps to `1` at the first Release Bundle ship and so on, as
  part of the release protocol (TBD — protocol not yet engineered).
- `X` = number of fully-completed development phases (= most recent completed
  Phase number). During Phase 15 development, `X=14`. **Bump `X` at the phase
  final-closeout CHANGELOG entry** to the just-completed phase number; that
  entry is versioned `vR.X.00` (for Phase 15 close: `v0.15.00`). Phases 1–14
  closed under the prior "X bumps at phase start" rule and are not
  retroactively renumbered (see Historical note above).
- `YY` = zero-indexed CHANGELOG entry counter, incrementing by 1 per
  CHANGELOG entry. Resets to `00` at the phase-final-closeout `vR.X.00`
  milestone.
- `Z` (DEVLOG only, new scheme) = per-anchor DEVLOG counter starting at 1,
  resetting to 1 each time a new CHANGELOG version is cut.
- **Under the v2 audit/implement workflow** (`multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md`),
  one CHANGELOG-or-DEVLOG entry covers a full audit-implement-reaudit cycle, not
  an individual role-round. A typical phase should produce ~5–15 cycle-closeout
  entries (CHANGELOG + DEVLOG combined), not one per priority × role-round.
  Historical entries written under v1 conventions (per-role-round) are not
  rewritten.
- Entry headers must include a full date+time stamp — month-only is too coarse when
  multiple sessions happen in one day
- **Common timestamp failure mode:** Agents often write `YYYY-MM-DD EST` (no time) or
  `YYYY-MM-DD HH:MM` (no timezone). **Both `HH:MM` and `EST` are required.** Correct:
  `2026-03-29 17:00 EST`. If the exact time is unknown (retroactive entry), use an
  approximate time prefixed with `~`: `2026-03-29 ~17:00 EST`.
- Each entry must include a `**Authors:**` line immediately before the `**Roadmap:**`
  line — see the Authorship conventions section below for format
- Each `CHANGELOG.md` entry must include a `**Roadmap:**` line pointing to the
  relevant ROADMAP priority or phase. `DEVLOG.md` entries use a `**Scope:**` line
  instead, naming the affected workflow/convention artifact(s) — e.g.,
  `**Scope:** workflow-v2, AGENT_CONVENTIONS.md, all 4 agent bootstrap files`.
- **`**Authors:**` and `**Roadmap:**` (or `**Scope:**`) must be separated by a blank
  line** so they render as separate paragraphs in markdown. Adjacent lines without a
  blank line between them merge into one line when rendered.
- Entries are reverse chronological (newest first)
- Pre-convention entries (`[Phase x.xx]` format) are historical — do not rename them
- Only record finalized, validated changes — never speculative or partial entries
- Do not rewrite or amend historical entries
- **Certain audits require a CHANGELOG-or-DEVLOG entry (nuance for v2 workflow cycles):**
  - *Formal audits* — using `multi-agent/audits/` instruction files, or ad-hoc audits the user explicitly requests — require their own entry describing scope, findings, and actions taken, even if the audit resulted in no changes. The entry goes in `CHANGELOG.md` if the audit targeted product surfaces (code, CLI, user-facing docs) and in `DEVLOG.md` if it targeted dev-system surfaces (workflow files, conventions, agent bootstrap files).
  - Brainstorming/SPEC feedback audits recorded in PHASE<N>_FEEDBACK.md or PHASE<N>_AUDIT_LOG.md do not get standalone CHANGELOG/DEVLOG entries unless the user explicitly asks for one; codebase inspection during planning does not make the session product-scope. 
  - CHANGELOG requires actual code/test/CLI/schema/user-doc changes.
  - When an audit should be reported, the right place is usually DEVLOG, not CHANGELOG.
- **Other audits should NOT be given their own CHANGELOG-or-DEVLOG entry (nuance for v2 workflow cycles):**
  - *Workflow-driven audits inside an audit-implement-reaudit cycle* under `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md` are recorded round-by-round in the sibling `PHASE<N>_AUDIT_LOG.md`. The cycle produces ONE consolidated entry at cycle closeout with authorship surveyed and unioned from the cycle's AUDIT_LOG rounds. The entry goes in `CHANGELOG.md` or `DEVLOG.md` by the cycle's primary content (code changes → CHANGELOG; workflow/convention changes only → DEVLOG). Do not write an entry per role-round under v2.

---

## ROADMAP.md — bird's-eye retrospective + light-reading overview
- **Role:** ROADMAP is now **retrospective and light-reading**, not a forward-looking
  compass. The phase system itself (the SPEC + AUDIT_LOG + STRATEGY + FEEDBACK files
  in `multi-agent/plans/` and `multi-agent/plans/archived/`) is the actual roadmap;
  CHANGELOG `**Roadmap:**` lines that say "Phase X — Phase Y" are pointing at the
  phase plans themselves, NOT at sections in `ROADMAP.md`. ROADMAP.md is a thin
  human-readable echo of what the phase plans are doing — useful for orientation,
  not for driving work.
- **What used to be the case:** earlier in the project, ROADMAP tried to look far
  ahead and required constant churn. That role is retired. Speculative future
  thinking now lives in `multi-agent/plans/next/` (see § Future-phase planning
  surfaces below).
- **When does a new phase get a ROADMAP entry?** Once its SPEC is fully formalized
  (NOT at brainstorm stage; NOT at SOUP stage). The entry is a short, reader-friendly
  overview of what the SPEC plans to do.
- **Headings** use "Phase X" and "Priority X.Y" — do **not** rename these to version
  numbers.
- **Status markers** added as priorities close: `✓ DONE (v0.x.xx)`,
  `◑ PARTIAL (v0.x.xx)`, unmarked = not started.
- **Phase numbers** align with the CHANGELOG middle digit: Phase 4 work → `v0.4.xx`
  entries.
- **Edit the entry for the currently-active phase** as priorities close (add status
  markers); leave older phase entries as permanent retrospective records (they
  reflect final state, not active state).
- **ROADMAP MUST NOT speculate** about future phases that have not yet hit
  formalized-SPEC stage. SOUP-stage and BRAINSTORM-stage thinking does not belong
  in ROADMAP.
- **Do not rewrite or remove entries speculatively.** Closed phases keep their
  entries; the entries are updated only when the final state is known.
- Each Priority block that is newly written or substantially expanded should include
  an `**Authors:**` line (same format as CHANGELOG — see Authorship conventions below),
  placed after the Priority heading and before the first sub-heading or body text.

## Active phase SPEC/PLAN files — detailed current-phase planning
- Location: `multi-agent/plans/`
- The active phase SPEC/PLAN file is the detailed source of truth for the **current** phase
- Detailed phase plans stay detailed; they are **never** collapsed into some second archive file
- Edit the active phase SPEC/PLAN file in place, conservatively, as planning evolves or as work
  is marked complete/partial
- "Phase finished" means: **after the Final Overseer wrap-up audit (Role 3 / Template D) has
  closed AND any post-wrap-up remediation cycles triggered by that audit have themselves
  closed.** A phase is NOT "finished" merely because its last execution cycle closed; the
  Final Overseer pass must complete first. Do not move the SPEC/PLAN file (or its sibling
  `_AUDIT_LOG.md` / `_STRATEGY.md` / `_FEEDBACK.md` files) into `archived/` before that
  point — even if a closeout-doc-sweep cycle is in progress, the phase planning files
  themselves stay at their live paths until after Final Overseer.
- Under the v2 workflow, the archive operation is performed by the orchestrator as a
  separate post-Final-Overseer step (NOT a Role 2 deliverable inside any execution cycle,
  including the last cycle of the phase). See
  `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md` § Standard Execution
  Loop / Step 8 — Phase Archive for the detailed mechanics.
- Mechanics: move the SPEC/PLAN file (and siblings) to `multi-agent/plans/archived/`
  using `git mv`, prepend the archival date as `YYYYMMDD-`, then update agent-file
  active-phase pointers to reflect the archived state.
- Once archived, a phase SPEC/PLAN file should be treated as immutable historical provenance,
  not as a living planning surface
- Archived phase SPEC/PLAN files replace the old role ROADSTRAVELED tried to play

---

## Future-phase planning surfaces — `multi-agent/plans/next/`

Candidate future phases live as files in `multi-agent/plans/next/`. They are
intentionally early-stage and disorganized — raw material from which formal
phase plans are built when the orchestrator is ready to bring a candidate onto
the stage.

### SOUP convention (filename + lifecycle marker)

- Files in `plans/next/` use the `<THEME>_SOUP.md` filename pattern (e.g.,
  `HMM_SOUP.md`, `SUMMIT_SOUP.md`, `FWD-ARCH_SOUP.md`).
- `<THEME>` is a vague / broad / lightly-evocative noun phrase — NOT
  scope-anchoring (`HMM_SOUP`, not `HMM_COMPLETENESS_SOUP`) and NOT a phase
  number (`HMM_SOUP`, not `PHASE15_SOUP`). Vague themes don't pre-suppose final
  scope and don't create renumbering pressure when the queue reorders.
- The `_SOUP` suffix is retained on every file in `plans/next/` so the
  lifecycle marker stays explicit even when files are referenced by name
  elsewhere.
- **No SELF-references to phase numbers.** The SOUP body MUST NOT mention
  any phase number that refers to *this file's own* past, current, or
  future identity. That includes: any "this is the Phase X plan" framing;
  any "Phase X will decide / Phase X follow-up" framing where Phase X is
  the eventual landing spot for this SOUP; any rename-history narrative
  that names the SOUP's own past `PHASE<N>_*.md` filenames; any
  "promoted to Phase Y" forward-reference. The reason: self-references
  break under the orchestrator's "rename to renumber" mechanic. Phase
  numbers for this SOUP's own identity are assigned only at promotion.
  Rename history is preserved by `git log --follow`; it does not need to
  be in the file body.
- **Cross-references to actually-closed phases ARE allowed** as historical
  anchors when they add useful context (e.g., "see Phase 14 Supplemental
  cycle 14S.1a v0.14.68 for the prior naming-debt cleanup", or "depends
  on Phase 11.1 work that landed at v0.10.32"). These don't break under
  rename of THIS file because they point at OTHER, stable phase
  identities. Use them sparingly — they don't replace explicit
  cross-reference paths to the relevant archived SPEC files when those
  exist.
- Multiple SOUP files for unrelated future themes coexist freely.
- **Audit-time check** for any agent reading or modifying a `plans/next/`
  file: if the body mentions a SELF-referential phase number, flag it
  for cleanup or clean it up if obvious. Cross-references to closed
  phases are NOT flagged. The wrap-up audit Template D in
  `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md` includes
  a `next/`-directory self-reference scan as part of cross-cycle drift
  checks. (The simple grep `grep -nE "Phase [0-9]" multi-agent/plans/next/*_SOUP.md`
  is a starting point; results require triage to separate self-refs from
  legitimate cross-refs.)

### `PHASES.txt` — orchestrator scratchpad (exception)

- `multi-agent/plans/next/PHASES.txt` is the **orchestrator's free-form
  scratchpad** for thinking about phases and how to organize them. Stays flat
  and free-form. Phase numbers may appear freely. Not authoritative; not
  auto-maintained by agents — agents should leave it alone unless explicitly
  asked to update it.

### SOUP → BRAINSTORM → SPEC lifecycle

A future phase progresses through these stages:

1. **SOUP stage in `plans/next/`** (`plans/next/<THEME>_SOUP.md`).
   Pre-BRAINSTORM scratchpad per the convention above. Owned by the
   orchestrator + occasional contributor. No phase number; theme name
   only. SOUP entries are unstructured and untagged at this stage.

2. **Promotion → live SOUP in `plans/`** (`plans/PHASE<N>_<THEME>_SOUP.md`).
   When the orchestrator decides a SOUP is the next thing to formalize,
   the SOUP file is **moved** from `plans/next/` to `plans/` with a
   filename that gains the assigned `PHASE<N>_` prefix and matches the
   sibling BRAINSTORM/SPEC pattern. **At this same step, every discrete
   idea/entry in the SOUP body is labeled with a `SOUP<N>.<idx>` ID** as
   a one-time labeling pass (see § Identifier system below — this is
   the single allowed modification to the SOUP body). Phase number IS
   now committed at this point (because the file lives at
   `PHASE<N>_<THEME>_SOUP.md`). The SOUP file is now LIVE in `plans/`
   alongside the BRAINSTORM that's about to be authored from it. After
   the SOUP labeling pass, the SOUP body is again read-only — it
   serves as the immutable source-of-truth-for-provenance during
   BRAINSTORM construction. This live-in-`plans/` stage parallels how
   BRAINSTORM stays live in `plans/` during SPEC engineering and how
   SPEC stays live in `plans/` during implementation: working files
   stay live during active work; archive only when work is done.

3. **BRAINSTORM authoring + first FEEDBACK entry**
   (`plans/PHASE<N>_BRAINSTORM.md` + `plans/PHASE<N>_FEEDBACK.md`). An
   agent (typically the orchestrator's designated planning agent)
   authors the BRAINSTORM from the now-labeled SOUP. Each BRAINSTORM
   entry receives a **`BRAIN<N>.<idx>` ID** and a `Source:` field
   citing the `SOUP<N>.<idx>` IDs it covers (plus any other source
   citations from `tracking/BRAINSTORM.md`, `tracking/KNOWN_ISSUES.md`,
   etc.). This step is iterative — partial transfer + FEEDBACK Q&A +
   further transfer is the expected pattern, not a single pass. **The
   BRAINSTORM-from-SOUP authoring step also triggers the first entry
   in `PHASE<N>_FEEDBACK.md`** (the Q&A surface that runs alongside
   SPEC engineering); the first FEEDBACK entry typically captures the
   agent's open questions, line-items-for-approval, and
   transfer-status notes.

4. **SOUP archival** (after BRAINSTORM transfer is verified complete).
   The SOUP file moves from
   `plans/PHASE<N>_<THEME>_SOUP.md` to
   `plans/archived/<YYYYMMDD>-PHASE<N>_<THEME>_SOUP.md` — the archive
   filename carries both lineages: `PHASE<N>` links the archived SOUP
   to its phase identity, `<THEME>` preserves the theme name from the
   SOUP's `next/` lifetime, `<YYYYMMDD>` is the date of archival.
   "Verified complete" means: every `SOUP<N>.<idx>` ID in the SOUP body
   is referenced in at least one BRAINSTORM entry's `Source:` field
   (mechanically auditable), AND any open FEEDBACK questions about
   transfer scope have been resolved.

5. **BRAINSTORM → SPEC stage.** Standard existing pattern: SPEC gets
   authored from BRAINSTORM; SPEC entries receive `SPEC<N>.<idx>` IDs
   (see § Identifier system); SPEC entries cite the BRAIN IDs they
   cover in a `Source:` / `Covers:` field. BRAINSTORM gets archived
   alongside SPEC at phase close.

6. **SPEC active.** Standard existing pattern: SPEC + AUDIT_LOG +
   STRATEGY + FEEDBACK live at `multi-agent/plans/`. ROADMAP.md gets
   a retrospective entry at SPEC formalization (see § ROADMAP.md).

7. **Phase close + archive.** Standard pattern (workflow v2 § Step 8
   — Phase Archive).

The promotion mechanic (steps 2–4) is documented operationally in
`multi-agent/workflows/phase-development-system_PDS-v3.md` (the SPEC-engineering
"early leg" of the dev pipeline) including agent prompts for the
soup-to-brainstorm transfer, audit, and closeout. This conventions file
is the canonical rules home; the workflow file is the operational
reference.

### Identifier system across phase-development stages

Different artifacts use different identifier formats, but ALL share a
common shape: `<TYPE><N>.<idx>` where `<TYPE>` is a short uppercase word
naming the artifact tier, `<N>` is the phase number, and `<idx>` is a
sequential index within the artifact. This makes IDs **globally unique
across the project** and trivially grep-able from any context (CHANGELOG,
ROADMAP, KNOWN_ISSUES, anywhere) without needing surrounding-text
disambiguation.

Treat these IDs as **provenance citations**, not as stable handles that
follow an idea unchanged through every stage — each stage's IDs reflect
that stage's organizational logic. Cross-references between artifacts
use the source artifact's native ID; the type-prefix and phase number
together make file disambiguation usually unnecessary.

**Cutover note:** these conventions apply starting from **Phase 15
onward**. Phases 12, 13, 14, and 14 Supplemental used pre-cutover ID
formats (bare `<N>.<idx>` for Phase 12/13/14; `14-S<idx>` for Phase 14
Supplemental); those archived plans are preserved in their original
form per the archived-content-is-immutable rule and are NOT migrated to
the new format. Cross-references from Phase 15+ artifacts to closed
Phase ≤14 priorities use the original format (e.g., "see Phase 14
Supplemental cycle 14S.1a, priority 14-S10").

**SOUP ID — `SOUP<N>.<idx>` in `plans/PHASE<N>_<THEME>_SOUP.md`.**
- Assigned exactly once: at the moment of SOUP promotion from
  `plans/next/` to `plans/PHASE<N>_<THEME>_SOUP.md` (lifecycle step 2
  above).
- One ID per discrete idea/entry. If an entry is a multi-line scratch
  containing genuinely-distinct sub-ideas, the agent can either assign
  one ID for the cluster (and flag the ambiguity in FEEDBACK for user
  resolution) OR escalate via FEEDBACK before assigning. Splitting one
  scratch into multiple IDs is allowed only via that escalation
  because it amounts to interpretation, not just labeling.
- Format: `SOUP<N>.1`, `SOUP<N>.2`, ..., placed at the start of each
  tagged entry. Sequential per file, starting at 1. Concrete example
  for Phase 15: `SOUP15.1`, `SOUP15.2`, `SOUP15.3`, ...
- Format inside an entry depends on the entry's structure: `## SOUP15.3
  title-or-topic` for heading-form entries, `- SOUP15.3 — text` for
  bullet-form entries, or just `SOUP15.3` inline as a leading marker
  for paragraph-form entries.
- Once assigned, SOUP body is read-only forever. The labeling pass is
  the only allowed body modification.
- Cited from BRAINSTORM as `Source: SOUP15.3, SOUP15.7`. The phase
  number in the ID makes the source file implicit; cross-phase
  citations (rare) read the same since the `<N>` disambiguates.

**BRAIN ID — `BRAIN<N>.<idx>` in `plans/PHASE<N>_BRAINSTORM.md`.**
- Assigned during BRAINSTORM authoring (lifecycle step 3). Every
  BRAINSTORM entry has a BRAIN ID.
- Format: `BRAIN<N>.1`, `BRAIN<N>.2`, ..., per BRAINSTORM file.
  Sequential, starting at 1. Concrete example for Phase 15:
  `BRAIN15.1`, `BRAIN15.2`, `BRAIN15.3`, ...
- Heading-form entries use `## BRAIN15.3 — title` (Option B style: drop
  any "Priority" or "Idea" label word; the typed ID is itself the
  category-labeling token).
- BRAIN IDs reflect BRAINSTORM organization, which need not match SOUP
  order or future SPEC order. BRAIN IDs may be reordered during
  BRAINSTORM iteration as the agent reorganizes; renumbering during
  BRAINSTORM iteration is allowed because the BRAINSTORM is the active
  surface at this stage. Once SPEC engineering begins (BRAINSTORM
  hands off to SPEC), BRAIN IDs freeze and become permanent
  cross-references.
- Each BRAIN entry's body includes a `Source:` field citing the
  `SOUP<N>.<idx>` IDs it covers, plus any other tier-1/tier-2 citations.
  Example: `Source: SOUP15.3, [ISSUE:2026-04-19:1], tracking/BRAINSTORM.md [2026-04-14] entry on HMM peak_rcn_stage`.
- Cited from SPEC as `Source: BRAIN15.5, BRAIN15.12` (phase number is
  the disambiguator).

**SPEC ID — `SPEC<N>.<idx>` in `plans/PHASE<N>_SPEC.md`.**
- Assigned during SPEC engineering (lifecycle step 5). Every SPEC
  Priority has its own ID.
- Format: `SPEC<N>.1`, `SPEC<N>.2`, ..., where `<N>` is the phase
  number. Sequential within the SPEC; need not match BRAIN ID order.
  Concrete example for Phase 15: `SPEC15.1`, `SPEC15.2`, `SPEC15.3`,
  ...
- Heading-form: `## SPEC15.3 — title` (Option B style: drop the
  "Priority" word; `SPEC15.3` itself signals the entry is a SPEC
  priority).
- During SPEC engineering (before SPEC is locked), IDs may be
  reassigned freely as priorities are added/removed/reordered. The
  goal at this stage is human-readable narrative flow, not stable IDs.
- After SPEC is locked (audit/implement stage begins), IDs freeze.
  Priorities added later get the next-available number even if it's
  "out of order" by appearance in the file (e.g., `SPEC15.27` may
  slot between `SPEC15.3` and `SPEC15.4` in the file's reading order).
  Section headings within the SPEC handle the human-readable
  organization; numerical ID order need not match.
- Each SPEC entry's body includes a `Source:` or `Covers:` field
  citing the `BRAIN<N>.<idx>` IDs from BRAINSTORM that the priority
  covers (one, many, or none — a SPEC priority may also be authored
  from new SPEC-stage thinking, in which case it cites the FEEDBACK
  discussions that produced it).

**Splits and consolidations across stages.**
- **Consolidation (multiple BRAIN IDs → 1 SPEC ID):** the SPEC entry's
  `Source: BRAIN15.5, BRAIN15.12, BRAIN15.17, BRAIN15.23` field lists
  all BRAIN IDs covered. Each BRAIN entry stays in BRAINSTORM
  unchanged; provenance is in the SPEC.
- **Split (1 BRAIN ID → multiple SPEC IDs):** the SPEC entries each
  cite the BRAIN ID (`Source: BRAIN15.12`); the BRAINSTORM entry for
  `BRAIN15.12` may also receive an annotation noting the split (e.g.,
  "Split into SPEC15.7, SPEC15.8, SPEC15.9 at SPEC stage") — this
  annotation is added at SPEC-lock and lives in the BRAINSTORM file,
  which is then archived alongside SPEC at phase close.
- **Same applies to SOUP → BRAINSTORM:** consolidation = BRAIN entry's
  `Source:` lists multiple `SOUP<N>.<idx>` IDs; split = multiple BRAIN
  entries cite one `SOUP<N>.<idx>`, with a FEEDBACK trail documenting
  why the split was made (since splitting at the BRAINSTORM stage is
  interpretive and requires user approval per the soup-to-brainstorm
  transfer convention).

**Auditability checks (mechanical).**
- During soup-to-brainstorm transfer audit: `grep -nE "SOUP<N>\.[0-9]+"
  PHASE<N>_<THEME>_SOUP.md` for source IDs; `grep -nE "SOUP<N>\.[0-9]+"
  PHASE<N>_BRAINSTORM.md` for citations in `Source:` fields. Any SOUP
  ID present in the SOUP file but absent from BRAINSTORM citations is
  potentially a missed entry — flag explicitly.
- During SPEC audit: `grep -nE "BRAIN<N>\.[0-9]+" PHASE<N>_BRAINSTORM.md`
  for source IDs; `grep -nE "BRAIN<N>\.[0-9]+" PHASE<N>_SPEC.md` for
  citations. Unreferenced BRAIN IDs represent ideas that didn't make
  it into SPEC — flag for explicit user decision (intentional drop?
  deferred? merged into a SPEC body without explicit citation?).
- These greps are clean precisely because the type-prefix +
  phase-number combination is globally unique and free of
  false-positive collisions with version strings or other numeric
  patterns.

**FEEDBACK references (no own provenance IDs).** FEEDBACK uses
`Q<idx>` / `JQ<idx>` for question/judgment-question identifiers
(existing convention from earlier phases). FEEDBACK entries reference
SOUP IDs, BRAIN IDs, and SPEC IDs by their native formats; FEEDBACK
doesn't introduce its own provenance-IDs since it's a Q&A surface,
not an idea registry.

**AUDIT_LOG references (no own provenance IDs).** AUDIT_LOG cycle
headings use `Cycle: <cycle-name>` (e.g., `Cycle: 15.1a — covers
SPEC15.1, SPEC15.2, SPEC15.5`). Cycle indices (`15.1a`, `15.1b`, ...)
are STRATEGY-defined groupings of SPEC IDs; the AUDIT_LOG
round-by-round content references SPEC IDs directly.

**CHANGELOG and ROADMAP references.** CHANGELOG entries' `**Roadmap:**`
line and ROADMAP entry headings should use the typed-ID-prefixed forms
when referencing Phase 15+ priorities — e.g., a CHANGELOG `**Roadmap:**`
line might read "Phase 15 — HMM completeness work covering SPEC15.3,
SPEC15.7, SPEC15.12". This makes the IDs grep-able across the entire
project history. ROADMAP retrospective entries follow the same
convention: `## SPEC15.1 — title ✓ DONE (v0.X.YY)`.

### Four-tier hierarchy of brainstorming/idea surfaces (don't conflate them)

The project has four distinct surfaces for nascent ideas. They serve different
roles and should not be confused:

1. **`multi-agent/tracking/BRAINSTORM.md`** — the **ultimate scratch pad / note
   pad of ideas** jotted down freely. Long-lived, general, not phase-scoped.
   Used as a **SOURCE** when authoring per-phase BRAINSTORM files (helps make
   them meatier).
2. **`multi-agent/tracking/KNOWN_ISSUES.md`** — long-lived registry focused on
   **near-term actionable known issues**, not broad speculative ideas. Also
   used as a **SOURCE** during SPEC engineering. See dedicated section below.
3. **`multi-agent/plans/PHASE<N>_BRAINSTORM.md`** (during SPEC engineering) —
   per-phase formalization staging file. Created by SOURCING from
   `tracking/BRAINSTORM.md` + `tracking/KNOWN_ISSUES.md` + the phase's own SOUP
   file (archived alongside as the BRAINSTORM is being authored). Has
   structure: themed sections, open questions, candidate priorities. Becomes
   the input to the SPEC.
4. **`multi-agent/plans/next/<THEME>_SOUP.md`** — pre-BRAINSTORM scratchpad
   (this section). Promotes to a per-phase BRAINSTORM when the orchestrator
   decides the theme is next.

When references in the codebase or docs say bare "BRAINSTORM" without a
qualifier, default reading: tier 1 (`tracking/BRAINSTORM.md`). Per-phase
BRAINSTORM files (tier 3) are typically referenced by their full path
(`multi-agent/plans/PHASE<N>_BRAINSTORM.md`).

---

## DEVPLAN naming convention — `DEVPLAN-<topic>.md`

Dev-system work plans use a separate naming pattern from product phases:

- Filename: `DEVPLAN-<topic>.md` (or `DEVLOGPLAN-<topic>.md` if preferred —
  both forms are valid). Topic is descriptive, free-form, no phase number.
- These files are **NOT product phases.** They do not get phase numbers, are
  not part of `ROADMAP.md`, and do not appear in `PHASES.txt`. They route to
  `DEVLOG.md` (not `CHANGELOG.md`) when implemented.
- Most dev-system work is **impromptu** and does not need a formal pre-
  implementation plan — a quick DEVLOG entry can stand alone. Write a
  `DEVPLAN-<topic>.md` only when:
  - The scope is multi-part and worth preserving against context loss.
  - You want to align with the user before committing to specific changes.
  - You want a queueable plan that can fire later, not now.
- DEVPLAN files MAY live in `multi-agent/plans/next/` if a formal pre-
  implementation plan is written and the work is queued for later. They do
  NOT need to be promoted to `multi-agent/plans/` the way a SOUP file would
  be — when a DEVPLAN is implemented, it can fire directly as an impromptu
  DEVLOG-routed cycle.
- Canonical example:
  `multi-agent/plans/next/DEVPLAN-ROADMAP-AND-NEXT-DIRECTORY-SYSTEM.md`.

---

## BRAINSTORM.md (tracking-dir reservoir) — speculative and loosely structured idea reservoir
- **Location:** `multi-agent/tracking/BRAINSTORM.md` (the long-lived reservoir
  — distinct from per-phase `PHASE<N>_BRAINSTORM.md` formalization staging
  files in `multi-agent/plans/`; see § Future-phase planning surfaces above
  for the full four-tier hierarchy).
- BRAINSTORM is where most future ideas should go first
- It does **not** need to be linear, phase-ordered, or implementation-ready
- Use BRAINSTORM for speculative ideas, naming options, future synthesis questions, and design
  possibilities that are not yet committed to the active phase plan
- When an idea becomes current-phase committed work, move or restate it in the active phase
  SPEC/PLAN file and, if appropriate, summarize it in ROADMAP

## KNOWN_ISSUES.md — actionable near-term issue registry
- **Location:** `multi-agent/tracking/KNOWN_ISSUES.md`. This is the long-lived
  near-term-issue surface (tier 2 in the four-tier hierarchy described in
  § Future-phase planning surfaces above). Distinct from per-phase BRAINSTORM
  formalization staging files in `multi-agent/plans/`.
- `multi-agent/tracking/KNOWN_ISSUES.md` is for real, near-term issues the project expects to address
  soon, especially side quests that are not yet formally placed into the active phase spec
- Use it for concrete bugs, incomplete capability gaps, and architectural cleanup items that are
  actionable now or very soon
- Agents should consult it when suggesting next steps, especially at Priority or Phase closeout
- If a known issue is promoted into the active phase spec, record that move in `CHANGELOG.md`
  and remove the entry from `KNOWN_ISSUES.md`
- If a known issue is solved directly or indirectly, record the resolution in `CHANGELOG.md`
  and remove the entry from `KNOWN_ISSUES.md`
- If a known issue is dropped, record the rationale in `DECISIONS.md` and note the drop in
  `CHANGELOG.md`
- **If a known issue is migrated from `KNOWN_ISSUES.md` to a new long-lived planning home**
  (e.g., a SOUP file, a per-phase BRAINSTORM, a tracking-tier file like
  `tracking/BRAINSTORM.md`), the migration is a three-step move:
  1. Add the issue's content to the new home, restructured as appropriate for that surface.
  2. **Annotate the new-home entry with a provenance line** of the form:
     `**Formerly lived in `KNOWN_ISSUES.md` and was referred to as `[ISSUE:YYYY-MM-DD:N]`.**`
     This preserves greppability of the original ID after the entry is removed from
     `KNOWN_ISSUES.md`, so historical references in CHANGELOG / DEVLOG / archived plans
     still resolve.
  3. **Delete the entry from `KNOWN_ISSUES.md`** (don't leave a stub) — the provenance
     line at the new home is the canonical history pointer.
  Rationale: leaving stubs in `KNOWN_ISSUES.md` after migration creates a maintenance
  burden (the stub stays "open" indefinitely and competes with real near-term issues);
  migrating cleanly with a provenance line at the new home keeps `KNOWN_ISSUES.md`
  focused on its core purpose while preserving traceability.
- Do not use `KNOWN_ISSUES.md` for speculative future ideas; those still belong in
  `multi-agent/tracking/BRAINSTORM.md`
- **Write entries immediately as they emerge** — do not defer new KNOWN_ISSUES items to
  end-of-session. Context compaction can silently discard deferred items before they are
  written to disk.
- **Entries are reverse chronological (newest first).** New top-level `[ISSUE:YYYY-MM-DD:N]`
  entries go at the top of the file (after the preamble), matching the convention used
  by `CHANGELOG.md`, `multi-agent/DEVLOG.md`, `multi-agent/AUDIT_HISTORY.md`,
  `multi-agent/plans/PHASE<N>_AUDIT_LOG.md`, and `multi-agent/plans/PHASE<N>_SURPRISE_LOG.md`.
  Out-of-order entries pre-dating this rule will be re-sorted at the next phase's
  closeout sweep (Phase 15 → cycle 15.10a SPEC15.20 sweep); do not re-sort retroactively
  outside of that sweep. In-place expansions of an EXISTING entry stay where the entry
  sits — they are not "new entries" for this rule's purpose.

---

## SURPRISE_LOG.md (per-phase) — peripheral-vision observations
- **Location:** `multi-agent/plans/PHASE<N>_SURPRISE_LOG.md`. Per-phase sibling
  to `PHASE<N>_SPEC.md`, `PHASE<N>_AUDIT_LOG.md`, `PHASE<N>_STRATEGY.md`, and
  `PHASE<N>_FEEDBACK.md`. Created at phase startup; archived alongside its
  siblings at phase close.
- **Purpose:** captures fresh-eyes observations that an agent noticed during a
  phase activity but deliberately did NOT include in the formal audit /
  implementation / closeout report because the observation was either
  (a) out of the cycle's scope, (b) hygiene drift adjacent to the work but not
  load-bearing, (c) an opinion or "smell" rather than a concrete finding, or
  (d) a cross-cycle pattern that doesn't belong inside any single cycle's
  section.
- **What goes in it:** adjacent drift, cross-cycle patterns, smells without
  concrete repair, opinions on drift from user intent, heads-up notes for
  future cycles.
- **What does NOT go in it:** concrete findings WITH a repair contract (those
  are AUDIT_LOG findings); re-derivable observations (e.g., file size); routine
  restatements of locked DECISIONS; emotional venting / blame / model-vs-model
  commentary.
- **Versus other surfaces:**
  - vs `PHASE<N>_AUDIT_LOG.md` — AUDIT_LOG is strictly in-scope per cycle's SPEC
    contract; SURPRISE_LOG is for *out-of-scope* observations from the same
    round.
  - vs `multi-agent/tracking/KNOWN_ISSUES.md` — KNOWN_ISSUES holds long-lived
    *actionable* near-term issues with concrete exit conditions; SURPRISE_LOG
    is for observations that are NOT yet actionable, may never be, or are smells
    without a clear repair.
  - vs `multi-agent/tracking/BRAINSTORM.md` — BRAINSTORM is for speculative
    future-direction ideas; SURPRISE_LOG is for "this looks off in the *current*
    tree", not "here is something we could do later".
- **Status taxonomy:** entries pass through `Active` → `In-flight` →
  `Resolved (vX.Y.ZZ)`, with side states `Promoted to <destination + ID>`,
  `Superseded by [SURPRISE:...]`, and `Logged-only`. **Active-forever is a
  smell** — explicit deferral with a landing zone is the discipline. See the
  PHASE<N>_SURPRISE_LOG.md preamble for the full taxonomy + when each role
  flips status.
- **Promotion paths** (when a smell stops being just a smell):
  - **Smell → KNOWN_ISSUE:** real + actionable + has a clear exit condition →
    file `[ISSUE:YYYY-MM-DD:N]` in `tracking/KNOWN_ISSUES.md`, flip SURPRISE
    status to `Promoted to KNOWN_ISSUES [ID]`.
  - **Smell → SPEC amendment:** user authorizes scope expansion → SPEC body
    gains a new priority, SURPRISE status flips to `Resolved (vX.Y.ZZ)` at
    closeout.
  - **Smell → DECISIONS lock:** user resolves a definitional ambiguity → write
    a DECISIONS entry citing the SURPRISE ID, flip status to `Resolved`.
  - **Smell → BRAINSTORM entry:** genuinely speculative future-phase work →
    file a BRAIN entry in `tracking/BRAINSTORM.md`, flip SURPRISE status to
    `Promoted to BRAINSTORM [date entry title]` (not Resolved — BRAINSTORM
    entries are not closure).
- **Authoring discipline:** entries are agent-controlled (NOT user-controlled
  the way SPEC scope is). The user reads the SURPRISE_LOG as input, not as a
  contract. Be honest about epistemic confidence — uncertainty surfacing is
  rewarded over polished claims. Append-only by default; do not delete entries
  if they turn out wrong, change status to `Logged-only` with a postscript.
- **Adding entries does NOT increment the version.** No CHANGELOG/DEVLOG entry
  for SURPRISE_LOG additions. If a SURPRISE entry catalyzes actual repair work
  in a later cycle, that cycle's CHANGELOG/DEVLOG closeout entry can reference
  the SURPRISE ID for context.
- **Workflow integration** — the v2 audit-implement-reaudit workflow
  (`multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md`) wires
  SURPRISE_LOG into Templates A/B/C/D so each round has an opportunity to file
  + a closeout review gate. See that file for the per-template additions.

### Three triage venues (where Active surprises get a disposition)

A SURPRISE entry transitions out of `Active` at one of three triage venues. Each
venue has the same three-outcome menu (integrate / elevate / dismiss-with-reasoning),
but the venues differ in *who* runs the triage and *when*.

1. **Orchestrator + assistant orchestrator (outside the cycle process).** The user
   (orchestrator), optionally with an assistant agent, can triage open SURPRISE
   entries between cycles — typically while drafting the next cycle's R1 launcher.
   This is the venue where a surprise can be folded into a cycle's R1 launcher as
   an explicit carry-over (parallel to how a KNOWN_ISSUES entry tagged with a
   cycle's landing zone gets called out in the launcher). **Scope authority
   constraint:** an assistant agent can *propose* an integration via launcher
   language; only the user authorizes scope. The assistant should not silently fold
   surprises into a launcher without the user's sign-off, same as for any scope
   change.
2. **Cycle Role 1 (Template A) at initial-audit time.** R1 scans Active SURPRISE
   entries whose Likely landing zone tags this cycle, this cycle's SPEC IDs, or
   this cycle's deliverables, and triages each. Disposition is recorded both in
   the audit log (as a finding or carry-over verification) AND on the SURPRISE
   entry's status line. See Template A's SURPRISE_LOG carry-over scan block.
3. **Cycle Role 1 re-audit (Template C) at closeout.** R3 reviews two populations:
   (a) entries authored during this cycle (`[SURPRISE:N:thiscycle:*:*:*]`) and
   (b) Active/In-flight entries whose Likely landing zone tagged this cycle. The
   closeout disposition flips Active/In-flight to Resolved / Promoted /
   Active-with-postscript / Logged-only. This is the **mandatory** triage venue —
   missing it means a cycle closes with un-dispositioned surprises and the
   Active-forever drift returns. See Template C's SURPRISE_LOG closeout review
   gate block.

A fourth venue exists for phase-spanning sweeps: the Final Overseer (Role 3,
Template D) at phase wrap-up. That venue carries entries forward to the next phase
or marks them resolved if they were closed by prior-cycle work that the per-cycle
re-auditor missed flipping. See Template D's SURPRISE_LOG end-of-phase sweep block.

### Triage outcomes (the three-option menu, applied at any venue)

- **Integrate into the cycle's repair contract** — fold the surprise into the
  active or upcoming cycle's audit/implementation work. SURPRISE status flips to
  `In-flight (cycle X.Y stage Z)` at pickup; the re-auditor at closeout flips to
  `Resolved (vX.Y.ZZ)` if verification succeeds.
- **Elevate / promote** — file `[ISSUE:YYYY-MM-DD:N]` in `tracking/KNOWN_ISSUES.md`
  or a BRAIN entry in `tracking/BRAINSTORM.md` (or write a DECISIONS lock, or
  authorize a SPEC amendment). SURPRISE status flips to `Promoted to <destination
  + ID>`. Closure of the destination resolves the SURPRISE.
- **Dismiss with reasoning** — confirm the observation is not real (code evolved,
  reassessment changed the read) or not actionable. Flip to `Logged-only` with a
  one-line postscript explaining why.

A fourth practical option — `Active (with explicit deferral postscript)` — is
acceptable when the surprise is real, the right home is genuinely a future cycle
or phase milestone, and the postscript names the specific landing zone. This is
deferral-with-discipline; Active-without-postscript is the smell the system is
designed to prevent.

---

## ROADSTRAVELED — retired
- `ROADSTRAVELED` is retired and must not be used for new work
- Historical content from that older system may be preserved under
  `multi-agent/plans/archived/20260412-ROADSTRAVELED.md` for reference only
- Do not copy new ROADMAP or SPEC content into any ROADSTRAVELED-style file

---

## Planning-content routing table

Use this table when deciding **where** planning or history content belongs.

| Content type | Primary destination | Notes |
|---|---|---|
| speculative idea, rough future direction, naming candidate, open synthesis question (general, not tied to a specific upcoming phase) | `multi-agent/tracking/BRAINSTORM.md` | Long-lived reservoir (tier 1 in § Future-phase planning surfaces hierarchy) |
| early-stage scratchpad for a specific upcoming theme/phase that will eventually be formalized | `multi-agent/plans/next/<THEME>_SOUP.md` | SOUP-stage tier (tier 4); promotes to per-phase BRAINSTORM in `plans/` when the orchestrator brings it onto the stage |
| dev-system maintenance plan (workflow + conventions edits; no product code) — pre-implementation | `multi-agent/plans/next/DEVPLAN-<topic>.md` | Optional pre-plan for dev-system work; impromptu DEVLOG entries can stand alone without a DEVPLAN |
| actionable near-term bug, incomplete capability, or side quest not yet promoted into the active phase spec | `multi-agent/tracking/KNOWN_ISSUES.md` | Long-lived registry (tier 2); consult especially when choosing next work after a Priority or Phase closes |
| detailed current-phase plan, implementation ordering, active scope clarification, done/partial markers for the current phase | active phase SPEC/PLAN file in `multi-agent/plans/` | This is the detailed source of truth for the current phase |
| per-phase formalization staging — themed sections + open questions + candidate priorities, sourced from `tracking/BRAINSTORM` + `tracking/KNOWN_ISSUES` + the phase's SOUP file | `multi-agent/plans/PHASE<N>_BRAINSTORM.md` | Tier 3; BRAINSTORM-stage; created during SPEC engineering; archived alongside SPEC at phase close |
| retrospective bird's-eye phase summary, completion-marker accumulation, light-reading overview of closed and active phase plans | `ROADMAP.md` | Retrospective + light-reading; updated when a SPEC formalizes (not at brainstorm/SOUP stage); add `✓ DONE (v0.x.xx)` markers as priorities close. Do NOT use ROADMAP for forward speculation. |
| validated completed onionskin **product** work (code, CLI, tests, user-facing docs), audits performed, shipped fixes | `CHANGELOG.md` | Past-looking record. Uses `**Roadmap:**` line (which points at phase plans, not at ROADMAP.md sections). |
| validated completed **dev-system** work — workflow files, `AGENT_CONVENTIONS.md`, agent bootstrap files, SPEC / BRAINSTORM / FEEDBACK / STRATEGY / AUDIT_LOG engineering without code, and any other `multi-agent/` edits not tied to code/test/user-doc changes | `multi-agent/DEVLOG.md` | Past-looking record. Uses `**Scope:**` line instead of `**Roadmap:**`. Shares version stream with `CHANGELOG.md`. Default bias: when uncertain, DEVLOG. |
| immediate next actions, blockers, 1–3 step tactical queue | `multi-agent/project_context/TASK.md` | Tactical only; do not duplicate ROADMAP |
| session bookmark, latest action, current state, next move | `multi-agent/project_context/HANDOFF.md` | Session bridge only |
| round-by-round audit findings, implementation reports, closeout judgments for the active phase (v2 workflow) | `multi-agent/plans/PHASE<N>_AUDIT_LOG.md` | Sibling to active SPEC; keeps SPEC clean as the contract; archived alongside SPEC at phase close |
| peripheral-vision observation that fell outside a cycle's formal repair contract — adjacent drift, smell, cross-cycle pattern, opinion on drift from user intent, heads-up for future cycles | `multi-agent/plans/PHASE<N>_SURPRISE_LOG.md` | Per-phase sibling to AUDIT_LOG; agent-authored fresh-eyes observations with a status taxonomy that prevents Active-forever drift; promotes to KNOWN_ISSUES / DECISIONS / SPEC amendment / BRAINSTORM when the substantive home becomes clear |
| completed phase detailed provenance | archived SPEC/PLAN files in `multi-agent/plans/archived/` | Archive intact; do not collapse into a second file |

If a note fits multiple categories, prefer the **most specific** file and summarize elsewhere only when needed.

---

## Cross-referencing
- CHANGELOG → ROADMAP: use the `**Roadmap:**` line in each entry. Note: this line
  points at the **phase plans themselves** (the SPEC files in `multi-agent/plans/`
  and `multi-agent/plans/archived/`) — phrased as `Phase X — Phase Y` titles —
  not at sections in `ROADMAP.md`. The phase system IS the project's roadmap;
  `ROADMAP.md` is the retrospective light-reading echo of it.
- ROADMAP → CHANGELOG: use the `✓ DONE (v0.x.xx)` or `◑ PARTIAL (v0.x.xx)` tag
- Active phase SPEC/PLAN ↔ ROADMAP current phase: keep the current phase summary in ROADMAP
  consistent with the detailed active phase SPEC/PLAN file, but do not try to make ROADMAP carry
  the full detail

### ID-context-in-chat rule (human readability)

When agents refer to project IDs in **chat output to the user** — `BRAIN<N>.<idx>`,
`SPEC<N>.<idx>`, `SOUP<N>.<idx>`, `IBM-C<N>...`, `[ISSUE:YYYY-MM-DD:N]`, `BRAIN<N>.<idx>`
references, etc. — agents MUST attach a brief in-situ topic phrase the first time the
ID appears in the message (and ideally on subsequent uses, when ambiguity matters).
This is a human-readability rule, not a file-format rule.

**Examples:**

- ✗ "Fold IBM-C8B + IBM-C12 + IBM-C14 design content into BRAIN15.7." — forces the
  user to grep four files to know what's being proposed.
- ✓ "Fold IBM-C8B (Stage-1 pre-amp isolation) + IBM-C12 (APS locus diagnostics —
  `best_onset_stage` / `post_support` / `dip_rate`) + IBM-C14 (Keep/exclude
  recommendation enhancements) into BRAIN15.7 (amplicon reliability scoring +
  flat-sample detection)."
- ✗ "Per `[ISSUE:2026-04-28:3]` we need to extend `--peak-summary`."
- ✓ "Per `[ISSUE:2026-04-28:3]` (Max projection RCN profiles — extend `--peak-summary`
  to RMS+HMM with raw/normalized/smoothed variants), we need to..."

**Why:** The ID system is optimized for agent throughput (greppable, mechanically
auditable). Humans pay a cost when IDs are referenced without context — they have
to mentally lookup or grep, which slows decision-making materially. A 5-word topic
phrase next to the ID makes the message readable end-to-end without breaking flow.

**In file content:** the same rule applies in FEEDBACK questions, FEEDBACK
follow-up sections, BRAINSTORM cross-references, and any other surface where the
user might read the content. In file content where only agents read (e.g., audit
findings, mechanical-coverage tables, internal cross-reference lists), bare IDs
are fine because agents can grep cheaply.

**Rule of thumb:** if the surrounding sentence reads naturally without the ID,
the topic phrase is helping; if it reads redundantly with the ID, the phrase
isn't needed. Err on the side of including it.

---

## Code-identifier naming — no pipeline step numbers in identifiers (critical)

Function names, parameter names, local variables, and dict keys MUST NOT
embed pipeline step numbers. Step numbers are **output-layout properties**
(they tell users where artifacts live in the directory tree) and are
**user-facing diagnostic strings** (they help users navigate `[hmm] step15:
...` log lines back to output dirs). They are NOT function-behavior
properties and they are subject to renumbering whenever pipeline structure
evolves.

### Why this rule exists

Phase 15 alone renumbered HMM pipeline steps twice (cycle 15.2a SAPS
placeholder insertion; cycle 15.4b gap-analysis-before-APS reorder). Each
renumber forced renames across function names, parameter names, local
variables, log message prefixes, dict keys, output filename prefixes,
AUDIT_LOG cross-references, SPEC line annotations, and tests. The cycle
15.4b Final-Overseer caught 5 missed sites despite a careful R1+R2 sweep.
Cycle 15.6a R1's `[SURPRISE:15:15.6a:1:A:3]` caught additional SPEC
line-number staleness. Function/param/variable names being step-numbered
is the load-bearing cause of much of that maintenance — they encode WHERE
a function sits in the pipeline sequence rather than WHAT the function
does. Decoupling code identifiers from step numbers eliminates an entire
class of audit findings forever.

### What is forbidden

- **Function names** that embed step numbers: `run_step15_hmm_gap_analysis`,
  `_load_step12_growth`, `_discover_step11_stage_files`. Use semantic names:
  `run_hmm_gap_analysis`, `_load_fork_travel_growth`,
  `_discover_amplicon_metrics_stage_files`.
- **Parameter names** that leak step numbers into call sites: `step4_dir`,
  `step11_dir`, `step12_dir`, `step14_dir`. Use semantic names:
  `mednorm_dir`, `amplicon_metrics_dir`, `fork_travel_dir`,
  `multistage_unification_dir`.
- **Local variables** that propagate step-numbered parameter naming:
  `step11_rows`. Use semantic names: `amplicon_metrics_rows`.
- **Dict keys** that index by step number: `steps["step15"]`,
  `steps["step16"]`. Use semantic keys matching Growth + RMS conventions:
  `steps["gap-analysis"]`, `steps["aps"]`. (See `[ISSUE:2026-04-30:2]` for
  the active inventory of HMM-side step-numbered keys awaiting refactor.)

### What is allowed (intentional uses of step numbers)

- **Output directory names** (`15-gap-analysis/`, `16-aps/`, `19-clustering/`)
  — user-facing layout where ordering matters; step numbers are the layout's
  primary signal.
- **User-facing diagnostic strings / log message prefixes** — `[hmm] step15:
  ...` (debatable; a hybrid like `[hmm] gap-analysis (step 15): ...` is
  preferable for renumber-resistance, but the prefix form is acceptable
  given user-facing diagnostic context). Either is fine; do not block
  pre-existing prefix forms during in-flight refactors.
- **Intentional algorithm-step artifacts** — `_summits_step1.tsv`,
  `_timing_step2.tsv`, `_summits_step3.tsv`, `_timing_step4.tsv` in
  `summit_convergence.py` refer to the **4-step convergence algorithm**
  (step1=initial summit, step2=initial timing, step3=refined summit,
  step4=final timing), NOT pipeline step positions. Different concept;
  the step-numbered filename IS the data. Keep.

### Cross-pipeline consistency check

Growth and RMS already follow this rule. Compare:
- Growth: `growth_steps["origins"]`, `growth_steps["stage-medians"]`;
  functions like `compute_evidence_track`, `refine_origin_for_call`,
  `write_stage_median_within_calls`.
- RMS: `rcn_ms_steps["chrom-norm"]`, `rcn_ms_steps["summit-refinement"]`;
  functions like `_per_stage_one`, `_classify_unified_stage_calls`,
  `run_rcn_mean_shift`.
- HMM (the asymmetry, awaiting refactor per `[ISSUE:2026-04-30:2]`):
  `steps["step15"]`, `steps["step16"]`; functions like
  `run_step15_hmm_gap_analysis`, `run_step16_hmm_aps`.

Future code reviews + audits enforce. When adding a new helper or
refactoring an existing one, prefer semantic names. When touching an
existing step-numbered identifier as part of in-flight cycle work, rename
opportunistically per the cycle's scope authority.

### Cross-references

- `multi-agent/tracking/KNOWN_ISSUES.md` `[ISSUE:2026-04-30:2]` — concrete
  three-category inventory of HMM pipeline step-number leakage + per-cycle
  in-phase fix guidance.
- `multi-agent/plans/next/CROSS_PIPELINE_UNIFICATION_SOUP.md` Items 1-2 —
  future-phase substantive home for the bulk refactor of HMM step-number
  leakage.
- `multi-agent/plans/PHASE15_SURPRISE_LOG.md` `[SURPRISE:15:15.6a:1:A:8]`
  — broader cross-pipeline structural divergence meta-observation that this
  rule prevents future instances of.

---

## CLI Flag Naming and Structure Conventions

Use the live Phase 14 Supplemental parser structure (CLOSED v0.14.74, 2026-04-26)
in `onionskin.py` as the model for future CLI work. The conventions below codify
the patterns established across Phase 14 + Phase 14 Supplemental.

1. When a concept is shared across pipelines, add a canonical flag in the `Universal` parser
   group first. See `onionskin.py:build_parser()` for the current owning pattern.
2. When one pipeline needs custom behavior for that shared concept, add a pipeline-specific
   override in the owning parser group using the `--<pipeline>-<concept>` naming pattern
   (`--growth-*`, `--rms-*`, `--hmm-*`).
3. Pipeline-specific overrides must semantically inherit from the universal flag or runtime
   default when unset. Document this as an "unset/omitted inherits" rule, not as a literal
   `default None` requirement, because the live parser uses both `argparse.SUPPRESS` and `None`
   depending on whether an override is already active or only reserved for a later phase.
4. The owning inheritance pattern lives at the `onionskin.py` boundary. Follow the existing
   resolution model such as `onionskin.py:_resolve_override_value()` instead of inventing ad hoc
   fallback logic in downstream code.
5. Universal help text should mention the pipeline-specific override surfaces when that context
   helps users understand precedence or coverage.
6. Keep abstract flag naming focused on the concept, not the units. Put units such as `kb` in
   help strings unless a flag is intentionally engine-local and already has a stable native name.
7. Remove legacy CLI aliases through a pre-parse redirect path, not through hidden argparse
   aliases. Follow the live `_DEPRECATED_FLAGS` / `_check_deprecated_flags()` pattern in
   `onionskin.py` so removed flags fail clearly with a redirect before normal parsing begins.
8. Treat `onionskin_core/` parameter names as internal APIs. They do not need to mirror top-level
   CLI renames. CLI normalization and type conversion belong in `onionskin.py` before calls cross
   into core modules.
9. **`halfwidth` vs `window` distinction.** When a numeric flag value represents a search
   radius (the half-span used to construct a window centered on a reference point), the flag
   name MUST use `-halfwidth` (e.g., `--growth-scan-halfwidth`,
   `--rms-peak-search-halfwidth`, `--refine-summit-halfwidth`). When the value represents
   a full-span window width directly (no reference point in the middle), the flag name MUST
   use `-window` or another full-span term. Per Q7/Q8 user direction (Phase 14 Supplemental).
   Mismatched naming was a major source of confusion pre-Phase 14 and was systematically
   corrected in cycles 14S.1a / 14S.1b.
10. **Universal-tier group pattern (Universal-in-spirit groups).** A cross-pipeline shared
    concept may live in a *named parser group* of its own (e.g., `Asymmetric Triangle Model`,
    `Shape scoring`, `APS`, `Timing`, `Overlap resolution / Deduplication / twin-peak
    decomposition`, `Summit Refinement`) rather than being moved into the `Universal`
    parser section. These groups behave Universal-in-spirit: flags inside them apply across
    pipelines (or are forward-declared with `[..; planned]` step mentions for pipelines
    that have not wired the flag yet), and pipeline-specific overrides for those flags use
    the `--<pipeline>-<concept>` naming pattern in their own pipeline group. The
    "Universal" parser-section name is overloaded with the cross-pipeline concept; agents
    must distinguish "Universal parser section" (a specific section header in `--help`
    output) from "Universal-in-spirit" (the cross-pipeline semantic property). Per Q20 user
    direction (Phase 14 Supplemental).
11. **Step-mention convention in help strings.** Every flag's help string should end with a
    bracketed step mention identifying which output directory step(s) the flag affects.
    Format: `[<pipeline>: <NN>-<step-name>]` where `<pipeline>` is one of `growth`, `rms`,
    `hmm`, and `<NN>-<step-name>` is the full output-directory step name from
    `onionskin_core/output_layout.py` (e.g., `02-calls`, `06-HMM`,
    `09-summit-refinement`, `14-aps`). For Universal-in-spirit flags, list all pipelines
    using semicolon-separated form: `[hmm: 14-aps; growth: 13-aps; rms: 12-aps]`. For
    flags forward-declared in pipelines that have not wired the flag yet, use
    `[..; planned]` or `[..; planned (Phase <N>)]`. For cross-cutting infrastructure flags
    (threads, RNG seed, verbosity) use `[applies to all pipelines]`. Section headers MAY
    carry the step mention when all flags in the group share a step (the Timing group is
    an example). Per cycles 14S.3a / 14S.3b user direction (Phase 14 Supplemental).
12. **No engine modules have argparse.** Per the 14-S10 decision (Phase 14
    Supplemental, see `multi-agent/project_context/DECISIONS.md` `[2026-04-24] Standalone
    engine CLIs removed as maintained user surfaces`) and its completion under cycle 15.9a
    SPEC15.21 d10 (2026-05-04, v0.14.94), engine modules under `onionskin_core/engines/`
    do not contain argparse parsers, do not expose standalone CLIs, and are invoked from
    `onionskin.py` via Python signatures only. The growth engine's transitional
    `_run_argv()` parser + `onionskin.py:_build_ms_argv()` argv-translation layer were
    eliminated in cycle 15.9a; `run_multistage()` now accepts an `argparse.Namespace`
    constructed by `onionskin.py:_make_growth_namespace()`. Future engine files added to
    `onionskin_core/engines/` must follow the same pattern: pure Python API (Namespace or
    kwargs), invoked exclusively through `onionskin.py`; no internal argparse parsers.

---

## Cross-pipeline-parity audit checklist (R1 audit + SPEC engineering)

**Locked 2026-05-05 cycle 15.10a Stage D F10 (SPEC15.20 mid-phase amendment 2026-04-29
per `[ISSUE:2026-04-29:7]` resolution).** Project intent (per user direction
2026-04-29): *"the only true difference between pipelines is how they call amplicons.
Then almost everything thereafter is game for each pipeline at least by analogy."* But
"X-pipeline-only" defects keep being discovered mid-cycle, often during cycle-internal
audit + implementation rather than during planning. This checklist makes the discipline
explicit at the audit/SPEC-engineering layer.

**Apply at:**
- Every Role 1 audit (initial + re-audit + post-wrap-up triage).
- Every SPEC-engineering pass that touches a pipeline-specific surface.

**Checklist items:**

1. **If the cycle touches a pipeline-specific surface, explicitly verify the
   cross-pipeline analog or document why the pipeline-specific scope is correct.**
   Cite the project-intent quote: *"the only true difference between pipelines is
   how they call amplicons."*
2. **For shared concepts (timing, ODW, reliability, flat-sample, gap-analysis,
   shape filter, summit refinement, posterior, MAD emission, output-layout step
   ordering, CLI flag scope, summit aggregation), produce an explicit per-pipeline
   parity matrix:** Does growth have it? Does RMS? Does HMM? If asymmetric, is the
   asymmetry intentional (e.g., HMM-specific state-path-derived analysis) OR a defect?
3. **Audit step numbers / file names / CLI flag scopes for uniformity** across pipelines
   for shared concepts. Asymmetries should be flagged as findings — even when intentional,
   they warrant an explicit DECISIONS.md or SPEC entry capturing why.
4. **Surface candidate `[PATHOLOGY_CANDIDATE]` items mid-flight** when a pipeline-only
   defect is discovered during implementation; do NOT silently narrow scope to skip
   the cross-pipeline analog.

**Tools (recommended; not enforced):** A grep-driven helper script could scan for
`growth-only`, `RMS-only`, `HMM-only` patterns + check for cross-pipeline import
asymmetries. Future tooling work; not blocking.

**Why it matters (concrete prior misses):**
- HMM step15-aps did NOT emit MAD bedGraphs while growth + RMS APS did (cycle 15.4a-S1).
- HMM gap-analysis was at step 19 (last) while growth + RMS placed it before APS (cycle 15.4a-S1).
- `_finalize_amplicon_recommendations()` was growth-only (cycle 15.4a-S1).
- `--compute-aps` only suppressed growth APS while RMS + HMM ran APS regardless (cycle 15.4a-S1).
- PuffStep parity test path mismatch (cycle 15.4a — HMM-only controller narrowed chromosomes).
- HMM step-16 APS call site silently bypassed `--aps-shape-no-normalize` and the new
  aggregation-choice surface (cycle 15.6a-S1).

Each instance was a planning miss the SPEC engineering pass + R1 audit should have caught.
This checklist exists to catch them before they ship.

---

## PIPELINE_SPEC.md — maintenance rules
1. Any time a pipeline step changes (new normalization, detection logic, output column,
   changed default, changed formula), update PIPELINE_SPEC.md to match
2. Any time a new CLI flag is added that affects pipeline behavior, add it to Appendix A
3. Any time a new output file is added, add it to Appendix B and the relevant step
4. Any time CHANGELOG.md is updated, check whether it merits updating PIPELINE_SPEC.md;
   produce an update plan and report it to the user before making changes
5. Do not assume the spec is correct — verify against the code; code is always ground truth
6. "Where the code lives" sections do not include line numbers (too brittle to maintain);
   file paths and function names are sufficient
7. The spec is not authoritative on defaults — verify against `onionskin.py` argparse

---

## README.md — maintenance rules
1. When a capability listed under "Current limitations" or "Future directions" is completed,
   remove or reclassify it — do not leave completed work described as incomplete
2. When a major new capability is added (new mode, new output category, new CLI flag with
   user-facing impact), add it to the relevant section
3. The output directory layout tree must reflect the actual emitted structure; update it
   when directory layout changes
4. The module list must include all active modules; add new modules when created
5. Phase/version marker in "Current capabilities" should reflect the current release
6. README.md is user-facing — do not add internal implementation details; keep it
   accurate and honest but do not overwhelm it with every minor change

---

## ONIONSKIN_FULL_HANDOFF.md — maintenance rules
1. Any time significant new modules, design decisions, test coverage, architecture
   changes, or priorities shift, check whether ONIONSKIN_FULL_HANDOFF.md needs updating
2. Produce an update plan and report it to the user before making changes
3. Version header should reflect the current phase and version

---

## Agent file consistency — critical rule

If you modify ANY agent instruction file, you MUST IMMEDIATELY propagate the same
conceptual change to ALL other agent instruction files.

→ Modifying one file without updating the others is a CRITICAL FAILURE.
→ Agent files that contradict each other are also a CRITICAL FAILURE.

### Authoritative list of agent instruction files

- `CLAUDE.md`
- `AGENTS.md`
- `GEMINI.md`
- `.github/copilot-instructions.md`

### What "same conceptual change" means

- The same rule, constraint, or behavior is enforced in each file
- The same intent is preserved
- Wording may be adapted for agent-specific context, but meaning must not change

### Canonical documents agents must read

- `multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md` — canonical **context**
  document: architecture, design decisions, biological background, test matrix, gotchas.
  All agents are directed to read this. It is not the place for editorial rules.
- `AGENT_CONVENTIONS.md` — canonical **rules** document: editorial conventions,
  maintenance rules, consistency requirements. This file.
- `ROADMAP.md` — canonical bird's-eye source for current priorities and future directions
- The active phase SPEC/PLAN file in `multi-agent/plans/` — canonical detailed source for the
  current phase
- `BRAINSTORM.md` — canonical reservoir for speculative future ideas not yet committed to the
  active phase plan
- `CHANGELOG.md` — canonical record of completed work

### Bidirectional consistency

- If an agent file is updated → propagate to all other agent files; check whether
  `ONIONSKIN_FULL_HANDOFF.md` or `AGENT_CONVENTIONS.md` also needs updating
- If `ONIONSKIN_FULL_HANDOFF.md` is updated → check whether any agent file contains
  a redundant summary that has become inconsistent
- If `AGENT_CONVENTIONS.md` is updated → it takes effect for all agents automatically
  (all bootstrap files point here); no propagation needed unless agent-specific behavior
  differs

---

## Authorship conventions

Every CHANGELOG entry, every DEVLOG entry, and every new/substantially-expanded ROADMAP
Priority block must include an `**Authors:**` line.

### CHANGELOG placement

Immediately before the `**Roadmap:**` line at the end of each entry:

```markdown
**Authors:** John M. Urban, Claude Code 2.1.86 (claude-sonnet-4-6)

**Roadmap:** Priority X.Y — ...
```

### DEVLOG placement

Immediately before the `**Scope:**` line at the end of each entry:

```markdown
**Authors:** John M. Urban, Claude Code 2.1.104 (claude-opus-4-7)

**Scope:** workflow-v2, AGENT_CONVENTIONS.md, all 4 agent bootstrap files
```

The `**Scope:**` line replaces `**Roadmap:**` for DEVLOG entries because
dev-system work doesn't map cleanly to ROADMAP priorities. Scope should name the
affected workflow/convention artifact(s) concretely.

### ROADMAP placement

After the `## Priority X.Y — Title` heading, before the first sub-heading or body text:

```markdown
## Priority 8.4.1 — `--stage-values` continuous column support

**Authors:** John M. Urban, Claude Code 2.1.86 (claude-sonnet-4-6)

### Concept
...
```

### Per-agent format

All authorship signatures bundle the **runtime toggle** (Effort for Claude, Reasoning for
GPT, Thinking for Gemini) alongside the model ID inside parens, separated by a semicolon
(`<model-id> ; <Toggle>: <value>`). The toggle is required when known; if it is genuinely
unknown after asking the user, fall back to the model-only form and note the omission.

| Agent | Format | Example | Notes | Information user can provide |
|-------|--------|---------|-------| -----------------------------|
| Claude Code | `Claude Code <version> (<model-id> ; Effort: <value>)` | `Claude Code 2.1.109 (claude-sonnet-4-6 ; Effort: Extra High)` | Do not use the example; source from the live environment. Effort toggle is selectable in the Claude Code UI; ask user if not visible to you. | Effort toggle visible in the model selector / settings |
| Codex | `Codex-cli <version> (<model-id> ; Reasoning: <value>)` | `Codex-cli 0.118-alpha.2 (GPT-5.5 ; Reasoning: Medium)` | For version, sample `codex --version` from the live environment. If unable, look in ~/.vscode/extensions. **Always include both model name AND reasoning value — emitting just "Codex (GPT-5)" or omitting reasoning is incorrect.** Model name is GPT-5.5 (or GPT-5.4 if user confirms older), not "GPT-5". | Model selected at bottom right of chat box; reasoning level selected to its right |
| Gemini CLI | `Gemini CLI <version> (<model-id> ; Thinking: <value>)` | `Gemini CLI 0.1.9 (gemini-2.5-pro ; Thinking: high)` | Do not hallucinate; source from the live environment. The thinking budget is set via CLI flag; ask user for the value used if not visible. | Model and thinking budget reported at end of each comment (e.g. "Generated by Gemini 3.1 Pro Preview, thinking high") |
| Gemini Code Assist | `Gemini Code Assist (<model-id> ; Thinking: <value>)` | `Gemini Code Assist (gemini-2.5-pro ; Thinking: high)` | Do not trust context window alone; source from live environment or ask user. | Bottom-left corner shows model selection; thinking budget reported at end of each comment |
| GitHub Copilot | `GitHub Copilot (<model-id> ; <Toggle>: <value>)` where Toggle is **Effort** for Claude-family models, **Reasoning** for GPT-family models, **Thinking** for Gemini-family models. If model is auto-selected, use `GitHub Copilot (model auto-selected)`. | `GitHub Copilot (gpt-5-codex ; Reasoning: High)` or `GitHub Copilot (claude-sonnet-4-6 ; Effort: Extra High)` | Copilot routes to GPT, Claude, or Gemini — record the specific model AND the appropriate toggle value visible in the VS Code UI. | Model + toggle selectable in the Copilot chat UI |
| ChatGPT | `ChatGPT (<model-id> ; Reasoning: <value>)` | `ChatGPT (GPT-5.3 ; Reasoning: High)` | TBD | TBD |


**Rules**:
- **Source of truth:** The `<model-id>` MUST be sourced from the live operational
  environment for the current session. Do NOT use a value from memory or training
  data, as it may be outdated or incorrect.
- **Agents cannot self-determine the runtime toggle (Effort / Reasoning / Thinking).**
  The toggle value is set in the host UI (Claude Code GUI, Codex chat box, Copilot
  chat box, Gemini CLI flag, etc.) and is **not exposed to the agent** through any
  system prompt, environment variable, or tool. Agents have no programmatic way to
  read the current toggle value. If you have not been told the toggle value by the
  user this session, you MUST ask the user before writing any authorship line that
  includes a toggle clause. Do not infer it from the work being done; do not pick
  "what feels apt"; do not copy a prior session's value. The same caution applies
  to the model field when the system prompt's claim could conflict with the GUI
  selection (e.g., system prompt says one model but the GUI shows another) — ask
  the user to confirm.
- **Failure mode to avoid:** Earlier in the v0.14.69/v0.14.70.1 work, an agent
  wrote `(claude-sonnet-4-6 ; Effort: Extra High)` for an Opus-with-Effort-High
  session. Both fields were unverified. The lesson: never write a toggle value
  without asking; never write a model value when the GUI and system prompt could
  disagree without asking.
- **Confirm the toggle as the FIRST IMMEDIATE ACTION of a round (when not in the
  launcher prompt).** Surfaced explicitly Phase 15 cycle 15.6a-S1 (2026-05-03):
  Copilot wrote `Reasoning: Extra High` in its Stage A diagnostic authorship
  line when the actual user-selected reasoning level was High. This is the
  recurring "authorship/Effort attribution drift" failure mode (see
  `multi-agent/workflows/phase-development-system-v4.md § 13`). Agents must
  ask the user to confirm the toggle BEFORE doing any substantive work — catch
  the user at the keyboard before they walk away. If the launcher prompt
  already names the toggle explicitly (the recommended pattern), no ask is
  required; otherwise ask immediately.
- **What is and is not authoritative for the toggle value:**
  - Authoritative: the user's explicit confirmation in the current chat.
  - Suggestive only: the `PHASE<N>_STRATEGY.md` cycle-row primary/alt
    assignment. The user often deviates per resource constraints.
  - NOT authoritative: agent personal memory entries. They are stale
    snapshots; the user changes the toggle at will whenever he sees fit.
  - NOT authoritative: the agent's own self-reported authorship line (the
    failure mode being defended against).
  - Best guess when the user is unavailable: the most recent value
    confirmed in the current chat session.
- **Be generous with detail**: include version AND model ID AND the runtime toggle value (Effort for Claude, Reasoning for GPT, Thinking for Gemini) — all three when available. The toggle is the project's standard parallel-naming convention; do not omit it when known. But "known" means "told to you by the user this session" or "visible in a tool result," NOT "guessed."
- **User feedback**: When `<version>`, `<model-id>`, and/or `<reasoning effort>` are unknown or cannot be sourced from the live operational environment, ask the user to give you those details 
- **Help user gather information**: If you do not know this information, and the user does not, ask user to report information they see (i) at the bottom of chat boxes for model and reasoning selection, (ii) model information that might be reported at the end of each comment (e.g. Gemini Code Assist and CLI do this), and (iii) version information from a related command found in ~/.vscode/extensions.
- **Dynamic Routing / Auto-switching:** Because environments like Gemini Code Assist dynamically switch models mid-session (e.g., between Gemini 2.5 Pro and 3.1 Pro Preview), agents MUST ALWAYS ask the user to confirm the exact model used for the final generation before writing the `CHANGELOG.md` entry. Do not guess based on earlier turns.
- **Multi agent authorship**: Multiple agents contributing to one entry: list comma-separated, user first
- **Multi-round cycle authorship (v2):** For cycles run under the v2 audit-implement-reaudit workflow, the closing agent at cycle closeout consolidates authorship by surveying every `**Authors:**` line in the cycle's `AUDIT_LOG.md` rounds, deduplicating (user first, then other contributors in order of first contribution to the cycle), and emitting one unified `**Authors:**` line in the cycle-closeout CHANGELOG-or-DEVLOG entry (whichever matches the cycle's primary scope). This extends the per-entry "user first" rule to a per-cycle consolidation.
- **Use what you have**: If the exact version is unknown but the model is known (or vice versa), include what is known
- **Final check**: Confirm authorship signature with user 
- **Historical**: For retroactive entries where version cannot be determined, use `~` prefix: `Claude Code ~2.1.81 (claude-sonnet-4-6)`

---

## AGENT_SWOT.md — maintenance rules

Location: `multi-agent/AGENT_SWOT.md`

`AGENT_SWOT.md` is a manager-facing document tracking Strengths, Weaknesses, Opportunities,
and Threats for each agent actively used in the project. It is not a changelog — each
section represents the current assessment of that agent, updated in place.

1. Update when an agent exhibits a surprising or pattern-establishing behavior: a consistent
   strength, a systematic failure mode, a new capability, or a risk that emerged in practice
2. Update after any session that reveals new information about an agent's behavior on this codebase
3. The document has one section per agent; add a new section when a new agent joins the project
4. Include a "Planned (not yet used)" stub for agents that are known to be coming
5. Entries within each SWOT quadrant are brief bullets — no lengthy prose
6. Do not remove entries that have become outdated; instead annotate with `[resolved vX.X.XX]`
   so the history of known issues is preserved

---

## AUDIT_HISTORY.md — maintenance rules

Location: `multi-agent/AUDIT_HISTORY.md`

`AUDIT_HISTORY.md` is the authoritative log of all codebase and documentation audits. All audits require both an update to `CHANGELOG.md` (for product-scope audits) or `multi-agent/DEVLOG.md` (for dev-system-scope audits) AND `AUDIT_HISTORY.md`, all associated with the same version bump. Thus, after performing an audit, the following procedure MUST be followed to log the audit in `AUDIT_HISTORY.md`:

1.  **Determine the next version number.** After performing the audit, but before documenting it in the `CHANGELOG.md`, determine the version number that will be associated with the audit in both `AUDIT_HISTORY.md` and `CHANGELOG.md` by incrementing the latest version from `CHANGELOG.md` (e.g., if the latest is `v0.5.36`, the audit will be `v0.5.37`).
2.  **Log the audit in `AUDIT_HISTORY.md`.** Add a new row to the top of the audit history table (table must be ordered newest to oldest). This row MUST be filled out completely, using the new version number in the `related_changelog_entry` column.
3.  **Log the audit in `CHANGELOG.md`.** After the audit is complete and the history is logged, add the new versioned entry to `CHANGELOG.md`. The body of this entry must summarize the audit's scope, findings, and all actions taken. It MUST also explicitly state that `AUDIT_HISTORY.md` was updated.
4.  **Scope of "audit":** This entire procedure applies to all forms of audit, including:
    - Formal audits using the instruction files in `multi-agent/audits/`.
    - Informal requests from the user to check, verify, or audit any part of the codebase or documentation.
    - **NOT** the round-by-round audits inside a v2 audit-implement-reaudit cycle.
      Those are recorded in the sibling `PHASE<N>_AUDIT_LOG.md` (see the
      AUDIT_LOG.md section below) and consolidated into ONE CHANGELOG entry
      at cycle closeout. The AUDIT_HISTORY table row is only added if the user
      explicitly asks for a formal AUDIT_HISTORY entry tied to that cycle.

---

## AUDIT_LOG.md — v2 workflow audit trail

Location: `multi-agent/plans/PHASE<N>_AUDIT_LOG.md` (sibling to the active phase
SPEC during the audit-implement-reaudit stage under v2 workflow
`multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md`).

1. Every Role 1 audit round, Role 2 implementation round, Role 1 re-audit closeout
   or skip-reaudit lightweight closeout, Role 3 wrap-up audit, and post-wrap-up
   triage/implementation/closeout round goes into `AUDIT_LOG.md` under the current
   cycle section. Heading conventions:
   - `## Cycle: <name> — OPEN` or `## Cycle: <name> — CLOSED v0.X.YY` at cycle level
   - `### <timestamp> — Role <N> <round-type>` at round level
2. Each round subsection carries a `**Authors:**` line per the Authorship
   conventions below.
3. Do NOT write audit findings, implementation reports, or closeout judgments
   into the SPEC itself during the audit-implement-reaudit stage. The SPEC stays
   clean as the authoritative contract. Only the SPEC's priority status table and
   user-approved scope changes are modified during implementation.
4. Add new rounds at the top of the cycle's section (newest-first per the
   workflow rule at `spec_plan_three_role_audit_loop-v3.md` § "What goes in
   AUDIT_LOG.md"); do not rewrite or remove prior rounds. Historical integrity
   is enforced the same way it is for `CHANGELOG.md`.
5. When a phase closes out, `AUDIT_LOG.md` is archived alongside the SPEC under
   the same `YYYYMMDD-` date prefix in `multi-agent/plans/archived/`.

See `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md` for the full
cycle-section format, round heading conventions, and the authorship consolidation
rule applied at cycle closeout.

---

## project_context — shared short-term memory

`multi-agent/project_context/` contains files shared across all agents for short-term
coordination. These supplement (do not replace) CHANGELOG.md, ROADMAP.md, and
ONIONSKIN_FULL_HANDOFF.md.

### Active files

| File | Purpose | Update frequency |
|------|---------|-----------------|
| `HANDOFF.md` | Bookmark: last action, current state, mandatory next action | Every session end |
| `TASK.md` | Queue: immediate actionable tasks (≤5), blockers, minimal context | Frequently |
| `DECISIONS.md` | Rationale: design decisions, why alternatives were rejected | When decisions occur |

### Usage rules

- Do not modify template structure or headings
- Do not invent new sections or expand existing ones beyond their defined scope
- Each file has a single defined purpose — do not mix roles across files
- Do not duplicate or restate information that exists in another file
- Be concise and structured — prefer bullet points over prose
- Prefer facts over interpretation; do not speculate
- If unsure where something belongs, do not write it — ask or defer

Violations of these rules are considered incorrect task execution.

### HANDOFF.md — mandatory session-end rule

**HANDOFF.md MUST be updated before ending any session in which work was performed.**
This includes sessions that complete a task, sessions that leave work unfinished, and
sessions that make decisions the next agent needs to know about. A session that ends
without updating HANDOFF.md leaves the next agent blind.

### Session-boundary recommendations — surfacing natural end-points

This rule pairs with the HANDOFF.md mandatory session-end rule above. Together they
formalize both *when* to end a session and *how* to end it cleanly.

**Why this matters.** Long chats accumulate context-window cost and are subject to
silent auto-compression that can drop detail an agent later needs. The prompt cache has
a 5-minute TTL — long idle gaps between turns (e.g., waiting for another agent to finish
its session) cause cache misses that make the next turn slower and more expensive. The
onionskin project is explicitly designed for session-based work (HANDOFF.md, TASK.md,
CHANGELOG, and the tiered CLAUDE.md reading list all exist so a cold-start agent can
orient in ~10 file reads). Agents should help the user use that system.

**Recommended cadence for the orchestration chat.** Under the v2 workflow, every agent
role (R1, R2, R3) already starts in a fresh chat via the handoff prompt — those
boundaries are handled automatically. The orchestration chat (the user's coordination
session with Claude Code) is the one to watch. The recommended cadence is **one
orchestration chat per cycle**: orient → review R1 output → assign R2 → review R2 output
→ decide skip/re-audit → review R3 → close cycle → start fresh. Per-phase orchestration
chats burn tokens, lose context to compaction, and accumulate idle gaps between roles that
miss the cache. Per-role orchestration sessions are too granular. Per-cycle is the sweet
spot.

#### When to surface a recommendation

At a natural task boundary, the agent should surface a single-sentence recommendation
that the user consider starting a new chat. Be conservative — fire at true closeouts,
not at every visible milestone. Triggers fall into two categories: triggering boundaries
(fire on) and qualifying conditions (make an already-valid trigger more urgent, but do
not trigger on their own).

**Triggering boundaries** (these alone can cause the recommendation to fire):

- **Phase closeout** — phase is CLOSED, archive operation complete.
- **Cycle closeout** (v2) — a cycle has fully closed: either Role 3 confirmed at
  re-audit closeout, or skip-reaudit was accepted by the orchestrator; no in-flight
  follow-up. Replaces "priority closeout" from v1.
- **Major recon audit or cross-file review complete** — findings written, user has
  acknowledged, no follow-up in flight.
- **Returning to the project after a long idle period** (>1 hour within the same chat).

**Qualifying conditions** (make a real boundary more valuable to honor; do NOT trigger
the recommendation on their own):

- Conversation has grown very long (rough markers: >40 turns, or multiple large-file
  reads accumulated across the history).
- A CHANGELOG entry closing a self-contained unit was just written.
- Token-budget concerns (user has mentioned them, or an extended idle return is imminent).

**Guard — DO NOT surface while any of these is true** (even if a triggering boundary
appears nearby):

- An audit → implement → re-audit cycle is mid-flight.
- A SPEC-engineering round has unanswered open questions or pending user decisions that
  this session expects to integrate.
- The user has named a task as "in progress" this session.
- The user has declined a recommendation earlier in this session (unless a clearly
  different *triggering boundary* is subsequently hit).

The intent of the guard: never recommend a break during the middle of a workflow cycle,
because doing so is the exact failure mode the rule is designed to prevent.

#### How to surface (tone rules)

Use approximately this one-sentence form:

> "We have reached a natural boundary to start a new chat. If you plan to start a new
> chat, I can make sure all handoff documents are ready for a cold start. Just say 'Yes',
> and wait for my go-ahead before moving on from this chat."

Tone rules:

- **One sentence** for the surface message. No digression. No trade-off table. If the
  user wants more context, they will ask.
- **Do not re-surface** in the same session if the user has already declined or ignored
  the recommendation — unless a clearly different boundary is subsequently hit (e.g.,
  a whole new priority has closed).
- **Do not force the break.** The user decides.

#### What to do when the user says "Yes"

Perform a **larger-than-normal handoff-prep pass** before giving the final go-ahead. The
goal: any agent starting cold from the next chat should orient without needing to ask
questions.

1. **HANDOFF.md** — verify Last Action names the version bump(s), what was produced, and
   where results live. Verify Current State captures phase status and any blockers.
   Verify Next Action names the concrete next step(s) with enough detail that a cold-start
   agent can pick up without re-reading the current chat.
2. **TASK.md** — verify 1–3 concrete immediate tasks are listed. Strike through stale
   completed tasks; remove tasks that no longer apply.
3. **CHANGELOG.md** — verify the latest entry exists and describes the work fully,
   including authorship, version, timestamp. If an audit happened, verify the entry says
   so and names the audit.
4. **AUDIT_HISTORY.md** — if an audit happened this session, verify a row exists and
   points to the CHANGELOG entry.
5. **DECISIONS.md** — if any non-obvious design choice was made this session, verify a
   note exists per the DECISIONS.md rules below.
6. **README.md / PIPELINE_SPEC.md / ONIONSKIN_FULL_HANDOFF.md** — if user-facing changes
   landed this session, verify these are updated per their respective maintenance rules.
7. **No orphaned state** — verify no uncommitted work is mid-stream in a way that would
   confuse the next agent. If uncommitted work exists (partial edits, partial test
   runs, etc.), note it in HANDOFF.md explicitly so the next agent knows.
8. **Cold-start readability test** — re-read HANDOFF.md and TASK.md as if you had no
   session context. Could a cold-start agent pick up the work? If not, add what is
   missing.
9. **Orientation pointer for the next chat's first prompt.** Produce a short text block
   the user can paste into the first prompt of the new chat. Goal: give the next agent
   everything it needs to orient without re-reading this entire chat. Form:

   ```
   Cold-start on onionskin.
   Read (in order): multi-agent/project_context/HANDOFF.md,
   multi-agent/project_context/TASK.md, multi-agent/AGENT_CONVENTIONS.md (once per
   session).
   Last version: vX.X.XX (<timestamp>).
   Active work: <one-line summary of the current task>.
   Open questions pending user answer: <Q-IDs or "none">.
   Blockers: <one-line or "none">.
   First action I should take: <concrete pointer — read which file, audit which
   priority, integrate which answer set, etc.>.
   ```

   The orientation pointer is NOT a chat summary. It is a compact pointer that tells the
   next agent exactly where to start and what context matters. It augments HANDOFF.md
   rather than duplicating it — HANDOFF.md is authoritative and long-lived; the pointer
   is one-shot, purpose-built for this transition, and can be thrown away after use.

After the prep pass AND after producing the orientation pointer, confirm to the user
explicitly: "Handoff is ready for cold start. Safe to leave this chat." Paste the
orientation pointer in the same message so the user can copy it directly. Only then does
the user end the chat. The two-step pattern — `Yes → prep pass + orientation pointer →
go-ahead` — is deliberate. It ensures the user does not leave before the handoff is
actually ready, and gives the next chat a warm-start prompt instead of a cold one.

#### What to do when the user declines or does not acknowledge

Continue working normally. Do not re-surface in the same session unless a clearly
different natural boundary is subsequently hit.

### DECISIONS.md — when to write and when to consult

**Write to DECISIONS.md when:**
- A non-obvious design choice was made (especially if alternatives were considered)
- Something was tried and rejected — so future agents don't re-try it
- A question was re-litigated that should be settled

**Consult DECISIONS.md when:**
- About to make a design choice in an area where past decisions may apply
- Evaluating an approach that "seems reasonable" — check if it was already rejected
- Writing code that touches APS, summit estimation, shape filter, smoothing

**Routing rule:** If the reasoning behind a change matters for future decisions → DECISIONS.md.
If it is a one-time implementation note → CHANGELOG.md. Both can reference each other.

DECISIONS.md is **not** required reading on every session startup. It is reference
material, consulted at appropriate decision points.

---

## BRAINSTORM.md — conceptual scratchpad (operational rules)

Location: `multi-agent/tracking/BRAINSTORM.md`. See also the earlier
§ Future-phase planning surfaces (the four-tier hierarchy that
distinguishes this long-lived `tracking/` reservoir from per-phase
`PHASE<N>_BRAINSTORM.md` formalization staging files in `plans/`).
This section covers operational rules for the long-lived reservoir.

A free-form scratchpad for speculative ideas, naming candidates, and unconventional directions
that are not yet ready for ROADMAP or DECISIONS.md.

1. Write here when an idea is worth capturing but NOT yet committed to the project
2. Do NOT promote a BRAINSTORM entry to ROADMAP or code without explicit user direction
3. When an idea is committed to ROADMAP: update or remove its entry here; note where it landed
4. When an idea is abandoned: note the reason briefly, then remove the entry

BRAINSTORM.md is **not** required reading on session startup and is not a commitment document.
It is a conceptual staging area.

**Write entries immediately as they emerge** — do not defer new BRAINSTORM items to
end-of-session. Context compaction can silently discard deferred items before they are
written to disk.

---

## multi-agent/audits/ — audit task prompts

Location: `multi-agent/audits/`

Three reusable prompts for specific audit tasks. Use these when running the corresponding audit;
do not use them as general reference documents.

| File | Purpose |
|------|---------|
| `pipeline_spec_audit_prompt.md` | Audit `PIPELINE_SPEC.md` for drift from the codebase |
| `onionskin_full_handoff_audit_prompt.md` | Audit `ONIONSKIN_FULL_HANDOFF.md` for drift from code, spec, and agent files |
| `deep_understanding_audit_prompt.md` | Structured deep-understanding assessment of the full system |

These files are not startup reading and do not define conventions. They are invocation prompts —
reference or paste them when you want to run the corresponding audit.

---

## Git commit convention

Agents do **not** automatically execute git commits. After completing a role's work,
the agent must output the exact commands needed for the user to commit manually. The
user runs them.

Under the v2 audit-implement-reaudit workflow, each agent role (R1, R2, R3, Final
Overseer, post-wrap-up roles, Strategist) emits its own commit at the end of its
session. The CHANGELOG/DEVLOG batching rule (one entry per cycle closeout) is
independent of the commit cadence — git history stays granular for bisect/blame even
though the changelog is batched.

### LOG-entry cadence vs commit cadence (canonical distinction)

The cadence of CHANGELOG/DEVLOG entries and the cadence of git commits are
**independent**. The two-rule pair:

- **LOG-entry cadence:** ONE entry per cycle closeout (per the v2 batching rule).
  Stage iterations (brainstorming-stage rounds, SPEC-engineering rounds, audit-loop
  R1/R2/R3 rounds, etc.) do NOT write LOG entries. Audits inside a cycle go in
  `PHASE<N>_AUDIT_LOG.md`; brainstorming-stage Q&A goes in `PHASE<N>_FEEDBACK.md`.
  The closeout consolidates authorship and produces the single LOG entry.
- **Commit cadence:** intermediate commits ARE welcome (and encouraged) at logical
  checkpoints during iteration — every role-round, every brainstorming-stage round,
  every SPEC-engineering iteration can produce its own commit. Granular git history
  is good for `git log` / `git bisect` / `git blame`. The intermediate commits do
  NOT insert into `CHANGELOG.md` / `DEVLOG.md` — only the cycle-closeout commit
  does.

This separation is what makes Form C (below) possible: a substantive commit message
captured in a scratch file, fossilized in git via `git commit --file=...`, but with
no LOG insertion until the cycle closes.

Cross-reference: `multi-agent/workflows/phase-development-system_PDS-v3.md` §
*"How the Phase Development System Works"* → "One CHANGELOG/DEVLOG entry per cycle
closeout" + the brainstorming/SPEC-stage iteration prompts in the same file.

### Three forms

**A. Mid-cycle commit, terse** (audit-loop rounds — R1 audit, R2 implementation when
re-audit is declared, etc., where the round's work is mechanical and a one-line
summary suffices). No CHANGELOG/DEVLOG entry written this round; the message is a
one-line summary inline via `-m`:

```
git add <file1> <file2> ...
git commit -m "Phase <N>, cycle <name> (index <i>), Role <r> <round type> (Template <X>): <one-line summary>. | Authors: <consolidated authorship including toggle>"
```

Use Form A when the round's outputs are mechanical or terse-describable in one line.

**A2. Mid-cycle / intermediate commit, verbose (scratch-file body, NO LOG entry)** —
for stage iterations under PDS-v2 (brainstorming-stage rounds, SPEC-engineering
rounds, soup-to-brainstorm transfer rounds, etc.) where the round's work is
substantive enough that a one-line `-m` summary loses too much context. The commit
message body lives in a `changelog-entry-<topic>.txt` scratch file at repo root,
where `<topic>` is a descriptive non-version slug (NOT a `v0.X.YY[.Z]` version
string — those are reserved for Form B). The scratch file does NOT get inserted into
`CHANGELOG.md`/`DEVLOG.md` — only the cycle-closeout commit fossilizes a LOG entry.

```
git add <file1> <file2> ...
git commit --file=changelog-entry-<topic>.txt
rm changelog-entry-<topic>.txt
```

Examples of `<topic>` slugs from real intermediate commits:

- `changelog-entry-phase15-soup-to-brainstorm-round1.txt`
- `changelog-entry-phase15-brainstorming-round4.txt`
- `changelog-entry-phase15-brainstorming-round5.txt`

The `changelog-entry-*.txt` gitignore pattern covers Form A2 too. The body should
include `**Authors:**` (or `| Authors:` line) to match the project's authorship
convention; even though the body isn't going into a LOG file, the authorship line
makes `git log` searchable.

**Use Form A vs Form A2:**
- Form A (terse `-m`) — short mechanical work; audit-loop R1/R2/R3 rounds where
  the round summary fits in one line.
- Form A2 (verbose scratch file) — substantive design discussions, multi-decision
  rounds, or rounds where the commit message body captures content that helps
  future agents understand what changed and why.

Both are intermediate commits. Neither inserts into LOG files. Cycle closeout uses
Form B (next).

**B. Closeout commit** (the role that writes the CHANGELOG/DEVLOG entry — typically
Role 1 at re-audit closeout, Role 2 at skip-reaudit acceptance, Role 3 at wrap-up
closeout, etc.).

The closing agent (1) writes the CHANGELOG/DEVLOG entry, (2) writes a verbatim copy
to a top-level scratch file `changelog-entry-<version>.txt` (gitignored via
`changelog-entry-*.txt`), and (3) emits three commands:

```
git add <file1> <file2> ...   # include CHANGELOG.md or DEVLOG.md and any other modified files
git commit --file=changelog-entry-<version>.txt
rm changelog-entry-<version>.txt
```

The `--file=` form fossilizes the entry verbatim in git history, making `git log`
directly searchable for full entry text and preserving the original wording even if
the CHANGELOG/DEVLOG file is later reformatted, migrated, or split. The
version-tagged filename eliminates wrong-file-pasted-by-mistake risk.

`<version>` in the scratch filename is the full version string — `v0.X.YY` for a
CHANGELOG entry (e.g., `changelog-entry-v0.14.70.txt`) or `v0.X.YY.Z` for a DEVLOG
entry (e.g., `changelog-entry-v0.14.70.1.txt`). The gitignore pattern
`changelog-entry-*.txt` covers both forms.

### Appending to an already-written CHANGELOG/DEVLOG entry

If the user requests appending content to an existing CHANGELOG/DEVLOG entry, the
strategy depends on whether the original commit has happened yet. The agent picks
the right scenario by checking `git log --oneline` for the version commit, or by
asking the user.

**Scenario (a) — original commit already happened.** Edit the entry in CHANGELOG.md
or DEVLOG.md to add the appended subsection in-place. Then create a NEW scratch
file at repo root named `changelog-entry-<version>-followup<N>.txt` (suffix
`-followup1`, `-followup2`, etc. for subsequent follow-ups under the same entry).
The file contains JUST the appended subsection content (with a one-line lead
identifying it as a follow-up to the entry version). Emit the standard three-command
closeout block using the new follow-up file:

```
git add <files modified by the append>
git commit --file=changelog-entry-<version>-followup<N>.txt
rm changelog-entry-<version>-followup<N>.txt
```

The original commit fossilized the original entry; the follow-up commit fossilizes
just the appended content. Together they tell the story; the integrated entry lives
in the CHANGELOG/DEVLOG file itself.

**Scenario (b) — original commit not yet happened.** Edit the entry in-place to add
the appended content, then rewrite the existing scratch file
`changelog-entry-<version>.txt` with the FULL updated entry. The user's existing
three-command closeout block still works unchanged — only the file content is newer.

### Authorship in commit messages

The `| Authors:` field mirrors the CHANGELOG `**Authors:**` line and makes authorship
grep-able in `git log --oneline`. Always include the runtime toggle value (Effort for
Claude, Reasoning for GPT, Thinking for Gemini) per the AGENT_CONVENTIONS authorship
table. Example:

```
v0.14.69: workflow v2 corrections + effort/thinking guidance | Authors: John M. Urban, Claude Code 2.1.109 (claude-sonnet-4-6 ; Effort: Extra High)
```

### Rules
- Output the block in a fenced code block (triple backticks) — never as inline prose or
  inline code. See "Copy-paste output formatting" section above.
- List every file that was changed in `git add` — never `git add -A` or `git add .`
- Do not run `git add` or `git commit` automatically — git control belongs to the user
- **Three commit forms** (see § Three forms above): (A) terse mid-cycle commit via
  `-m "..."` for mechanical audit-loop rounds; (A2) verbose intermediate commit via
  `--file=changelog-entry-<topic>.txt` (NO LOG insertion) for substantive
  brainstorming/SPEC-engineering iteration rounds; (B) closeout commit via
  `--file=changelog-entry-<version>.txt` (WITH LOG insertion) at cycle closeout.
  The `<topic>`-slug filename in Form A2 must NOT be a `v0.X.YY[.Z]` version
  string — those are reserved for Form B closeout commits.
- The git commit block is emitted BEFORE the Orchestrator advisory + handoff prompt in
  the agent's wrap-up output, so the user runs git first, then pastes the handoff
- Commit IDs are not recorded in CHANGELOG.md — use `git log` to cross-reference
- Full template-level convention with examples and edge cases is codified in
  `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md` § "Convention for
  agent-produced git commit blocks"
