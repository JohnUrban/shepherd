# Dev-system DEVLOG

All notable changes to the **onionskin dev-system** are documented here.

The dev-system covers the multi-agent methodology and process infrastructure for
developing onionskin: workflow files, `AGENT_CONVENTIONS.md`, agent bootstrap
files (`CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `.github/copilot-instructions.md`),
phase-engineering artifacts (BRAINSTORM, FEEDBACK, SPEC reordering when scope is
unchanged, STRATEGY, AUDIT_LOG), `AUDIT_HISTORY.md` process rules, `DECISIONS.md`,
`project_context` shared-memory files when they're not tied to code changes, and
**any other edits to files under `multi-agent/` that aren't code/test/user-doc
changes**.

For onionskin **product** changes (code, CLI flags, output schemas, tests,
user-facing docs), see the root `CHANGELOG.md` instead.

## Routing rule

An entry goes here (DEVLOG) when the session's work is primarily about:
- multi-agent workflow/convention engineering
- SPEC / BRAINSTORM / FEEDBACK / STRATEGY / AUDIT_LOG engineering rounds that do not touch code
- agent-file drift fixes, re-read rule changes, and similar convention edits
- AUDIT_LOG.md and DEVLOG.md rule updates themselves
- any other edits under `multi-agent/` that aren't code/test/user-doc changes
- any session with **zero code changes** and **zero user-facing doc changes**

An entry goes in the root `CHANGELOG.md` when the session's work changes:
- code in `onionskin.py`, `onionskin_core/`, `scripts/`, `tests/`
- CLI parser surface, help text, or flag semantics
- output schemas or pipeline step outputs
- user-facing docs (`README.md`, `PIPELINE_SPEC.md`, `ONIONSKIN_FULL_HANDOFF.md`)

For genuinely mixed sessions, the entry goes in whichever CHANGELOG matches its
**primary** content, with a brief cross-reference (*"See [other-file] v0.X.YY
for the parallel [product|dev-system] work"*) in the other.

**Default bias: when uncertain, DEVLOG.** `CHANGELOG.md` is reserved for actual
onionskin program changes. Everything else about the work and collaboration —
workflow engineering, planning surfaces, convention edits, `multi-agent/`
maintenance — goes here.

## Format

Entries use the header `## [vR.X.YY.Z] — YYYY-MM-DD HH:MM EDT` (4-component version,
new scheme prospective from v0.14.70.1; X-bump trigger revised 2026-04-30 — see
that date's entry) instead of `CHANGELOG.md`'s 3-component `vR.X.YY` form. Otherwise
the format mirrors CHANGELOG, with one difference at the bottom:

- Entries use a `**Scope:**` line instead of `**Roadmap:**`. The Scope line names
  the affected workflow/convention artifact(s). Example:
  `**Scope:** workflow-v2, AGENT_CONVENTIONS.md, all 4 agent bootstrap files`.

All other rules (authorship, timestamp with timezone, reverse chronological order,
no rewriting of history) apply identically. See `AGENT_CONVENTIONS.md` for full
CHANGELOG/DEVLOG rules.

## Versioning

**Split streams, prospective from v0.14.70.1; X-bump trigger revised 2026-04-30.**
DEVLOG entries use the form `vR.X.YY.Z` where the `vR.X.YY` portion is the
**anchor** — it matches the latest CHANGELOG version at the time the DEVLOG entry
is written — and `.Z` is a per-anchor counter starting at 1 and incrementing by 1
for each new DEVLOG entry while the anchor remains the same. When a new CHANGELOG
entry is cut (incrementing `YY`, `X`, or `R`), the DEVLOG counter `.Z` resets to
1 under the new anchor.

- `R` = number of Release Bundles shipped (`R=0` pre-release; bumps at Release
  Bundle ship per the release protocol — TBD).
- `X` = number of fully-completed development phases (= most recent completed
  Phase number). Bumps at the phase final-closeout CHANGELOG entry to the
  just-completed phase number.
- `YY` = monotonic CHANGELOG-entry counter (resets to `00` at the
  phase-final-closeout `vR.X.00` milestone).

**Anchor selection.** The anchor is the latest CHANGELOG version that has been cut
at the time of writing. During a phase, CHANGELOG `X` stays at the prior phase
number (e.g., during Phase 15, CHANGELOG anchors are `v0.14.YY`). CHANGELOG only
bumps `X` to the just-completed phase number at that phase's final-closeout entry.
Dev-system planning for an upcoming phase anchors to the latest CHANGELOG entry,
which during the in-progress phase is one of its own `v0.<X-prior>.YY` entries.

**Total ordering.** A DEVLOG v0.14.70.3 unambiguously came after CHANGELOG v0.14.70
(because the anchor matches) and before whatever the next CHANGELOG cut is.

**Historical entries (pre-v0.14.70.1)** used the older shared-stream scheme where
both CHANGELOG and DEVLOG drew from a single `v0.X.YY` counter; v0.14.69 is the
last DEVLOG entry under that older scheme. Do not retroactively renumber.

**Cross-references** between the two files use the full version string. The
component count disambiguates the file: 3-component → CHANGELOG (`v0.14.70`),
4-component → DEVLOG (`v0.14.70.3`).

---

## [v0.15.00.3] — 2026-05-06 EDT

`multi-agent/plans/interphase/` directory created as the staging area for active inter-phase planning notes; agent bootstrap files + AGENT_CONVENTIONS.md + HANDOFF.md updated to encode the directory-based mode indicator.

**Trigger:** Principal direction 2026-05-06 to formalize inter-phase work staging. Three options were considered: (1) live planning files in `multi-agent/plans/` alongside phase plans, (2) separate top-level interphase staging dir, (3) `multi-agent/plans/interphase/` subdirectory mirroring the existing `next/` + `archived/` subdirectory pattern. Principal selected Option 3 for structural-clarity benefit (directory presence at top level encodes phase-vs-interphase mode) + consistency with existing subdirectory convention. Filename convention: `<TOPIC>.md` (no required prefix; loose).

**Three-stage progression now formalized:**
- `multi-agent/plans/next/<THEME>_SOUP.md` — future-phase candidates (speculative)
- `multi-agent/plans/interphase/<TOPIC>.md` — active inter-phase work (NEW)
- `multi-agent/plans/PHASE<N>_*.md` (top level) — active phase plans (when phase mode)
- `multi-agent/plans/archived/<YYYYMMDD>-*.md` — closed items of all kinds (chronological bucket)

**What changed:**

- **`multi-agent/plans/interphase/` directory created** with `README.md` documenting filename + lifecycle + mode-indicator conventions. README explicitly notes that when a phase opens, active items in `interphase/` should either be archived (default) or graduate into the new phase's planning.
- **`CLAUDE.md` § Inter-phase development mode** extended with a "Directory-level mode indicator" subsection (phase mode = `PHASE<N>_*.md` at top level of `plans/`; interphase mode = check `plans/interphase/`) + explicit reference to `plans/interphase/<TOPIC>.md` as the staging location for inter-phase planning notes + cross-reference to `interphase/README.md`.
- **`AGENTS.md`, `GEMINI.md`, `.github/copilot-instructions.md`:** sections 8 + 8a rewritten as conditional (mirroring the earlier CLAUDE.md update at v0.15.00.1 — these mirror files still had stale `PHASE14_SUPPLEMENTAL-SPEC.md` "active" pointers); new "Inter-phase development mode" section added (parallel to CLAUDE.md's). All three agent bootstrap files now reflect the inter-phase mode + directory-based indicator + `plans/interphase/` staging convention.
- **`multi-agent/AGENT_CONVENTIONS.md` Mode awareness callout** extended with the "Directory-level mode indicator" sub-block (same content as the agent-bootstrap-file version) + the three-stage `next/` → `interphase/` → `PHASE<N>_*.md` → `archived/` progression note.
- **`multi-agent/project_context/HANDOFF.md` cold-start** extended with a "Mode indicator (structural)" block (one-line each for phase / interphase mode) + a routing pointer to `plans/interphase/` for active inter-phase work + cross-reference to the `interphase/README.md`. Currently states `interphase/` is empty (no live surgical work staged).

**Why minimal:** Per Principal direction, inter-phase mode is intentionally non-prescriptive. The `interphase/README.md` documents conventions but does not impose required workflow ceremony; agent files frame inter-phase mode descriptively (when to use `interphase/<TOPIC>.md`, when to archive) without prescribing R1/R2/R3 discipline analogs.

**Notes on existing agent-file drift:** the three mirror files (`AGENTS.md`, `GEMINI.md`, `copilot-instructions.md`) had not been updated at v0.15.00.1 when the inter-phase section was first added to `CLAUDE.md`; this entry brings all four bootstrap files into sync.

**No code/test/user-facing-doc changes.** Pure dev-system file lifecycle + workflow convention.

---

## [v0.15.00.2] — 2026-05-06 EDT

`HANDOFF.md` + `TASK.md` archived + rewritten as fresh minimal session-bookmark + task-queue files.

**Trigger:** post-Phase-15-archive cleanup. The pre-archive `HANDOFF.md` had grown to 1928 lines (94 cold-start bullets, mostly stale; "Last Action" section of 1184 lines duplicating CHANGELOG + AUDIT_LOG content; "Current State" section frozen at v0.14.64-era v2 workflow rollout; "Next Action" section with explicitly-HISTORICAL launcher pointers). The pre-archive `TASK.md` had a "Previously completed" section that self-acknowledged duplication ("see CHANGELOG/DEVLOG for full history") plus a "Context (minimal)" section frozen at the Phase 14 Supplemental + mid-Phase-15 era. Both files were doing more harm than help for cold-start agent orientation: file-purpose intent ("session bookmark", "Max 5 tasks total") was being violated by accumulation.

**What changed:**

- `multi-agent/project_context/HANDOFF.md` (1928 lines) → `multi-agent/plans/archived/20260506-HANDOFF.md` (preserved verbatim) via `git mv`. New `HANDOFF.md` written at original path with 23 lines: file-purpose statement, single cold-start bullet for Phase 15 closed + inter-phase mode pointer, "Where current + forward work lives" block (KNOWN_ISSUES + BRAINSTORM + SOUP files + CLAUDE.md + AGENT_CONVENTIONS.md pointers), "Where historical context lives" block (CHANGELOG entries + archived phase plans + git log retrieval channels).
- `multi-agent/project_context/TASK.md` (124 lines) → `multi-agent/plans/archived/20260506-TASK.md` (preserved verbatim) via `git mv`. New `TASK.md` written at original path with 16 lines: rules header + single immediate-next-task entry (no phase-execution work queued; inter-phase mode pointer; near-term priorities + future-phase-candidates routing) + per-task history retrieval pointer.

**Filter applied (per Principal direction 2026-05-06):** "still true" = currently relevant or forward-looking, NOT historical-fact-that-stays-true. Anything stale, outdated, or self-acknowledged-duplicative was pruned. Anything orienting an agent to current state or forward direction stayed. Net reduction: 2052 → 39 lines in the live files (~98%). Zero information lost — old content preserved in archived dated-prefix files + git history + CHANGELOG + audit logs.

**Why archive rather than delete:** the bloated predecessors are now ignorable but recoverable if anyone ever needs to reconstruct in-flight Phase 15 cycle context not already covered by CHANGELOG / AUDIT_LOG entries. Archived to `multi-agent/plans/archived/20260506-{HANDOFF,TASK}.md` (the same convention used for archived phase plans). The new live files cross-reference the archives explicitly so retrieval is intentional.

**No code/test/user-facing-doc changes.** Pure dev-system file lifecycle.

---

## [v0.15.00.1] — 2026-05-05 EDT

CLAUDE.md + AGENT_CONVENTIONS.md — inter-phase development mode formally acknowledged.

**Trigger:** Phase 15 closed + archived at v0.15.00 (2026-05-05). Project entered an inter-phase period during which development is intentionally unstructured — targeted surgical work, one task at a time, conversational rather than ceremony-heavy. CLAUDE.md sections 8 + 8a previously pointed at `PHASE14_SUPPLEMENTAL-SPEC.md` as "active" (already stale before Phase 15; now even more stale post-Phase-15-archive); no section described how to operate without an active phase.

**What changed:**

- **CLAUDE.md sections 8 + 8a:** rewritten as conditional. If a phase is active (HANDOFF.md cold-start says so), read `PHASE<N>_SPEC.md` + `PHASE<N>_AUDIT_LOG.md`. If inter-phase mode, skip those reads — see new § Inter-phase development mode below. Historical Phase 14 Supplemental + Phase 15 archive paths cited as examples.
- **CLAUDE.md new § "Inter-phase development mode" subsection** (placed between the tiered reading section and the Planning model section). Describes the lighter pattern: conversational R1-discuss → R2-implement → R3-check; no formal SPEC/AUDIT_LOG/STRATEGY artifacts required; tasks land via normal commits + CHANGELOG/DEVLOG entries; near-term priorities live in `KNOWN_ISSUES.md`; longer-term in `BRAINSTORM.md` or `plans/next/<THEME>_SOUP.md`. When inter-phase work accumulates (multiple related tasks; unifying theme; substantive scope), it can graduate into a formal phase via the SOUP→BRAINSTORM→SPEC lifecycle. Section explicitly disclaims being a rule-set: "intentionally unstructured… conversational rather than ceremony-heavy." Phase-mode discipline remains the reference for HOW phases are run when active; inter-phase mode does not invalidate or replace it.
- **AGENT_CONVENTIONS.md preamble:** added a "Mode awareness" callout after the opening line. Notes that the file describes phase-mode operation; in inter-phase mode the structured conventions apply only opportunistically; cross-references `CLAUDE.md § Inter-phase development mode` for the lighter pattern. Notes that when a phase is active, the conventions are load-bearing; when no phase is active, treat them as reference rather than contract.

**Why minimal:**

The user direction was explicit: don't delete information (preserve for next phase); make conditionals so phase-mode discipline activates automatically when a phase fires; describe inter-phase mode descriptively but do NOT make it structured / rule-bound. The inter-phase section is intentionally short + qualitative + offers no enforcement mechanism.

**No code changes; no test changes; no user-facing doc changes.** Pure dev-system documentation.

---

## [v0.14.92.1] — 2026-05-04 EDT

CLAUDE.md — authorship version-lookup guidance for Claude Code VSCode vs terminal contexts.

Trigger: agents running in the VSCode extension were previously sourcing their version from
`claude --version` (homebrew-managed CLI, lags behind the extension-bundled binary) rather
than `$CLAUDE_CODE_EXECPATH --version` (set by the extension, authoritative). Separately,
`$CLAUDE_CODE_EXECPATH` is empty in terminal sessions, so terminal agents need the
`which claude` + `claude --version` fallback instead. Neither path can resolve the Effort
toggle — that must always come from the user.

**What changed:** Added new section `## Claude Code–specific: authorship version lookup`
to `CLAUDE.md` between the model-selection and token-efficiency sections. The section
provides a three-row table:
- VSCode extension → `$CLAUDE_CODE_EXECPATH --version` (try first; authoritative)
- Terminal CLI → `which claude` + `claude --version` (homebrew-lagged fallback)
- Other harnesses (Codex, Copilot, Gemini, ChatGPT) → ask user / read UI metadata

Also states explicitly that the Effort toggle cannot be determined programmatically in any
environment and must be confirmed by the user as the first immediate action if not
pre-filled in the launcher.

Cross-reference: `AGENT_CONVENTIONS.md § Authorship conventions § Per-agent format` holds
the canonical per-harness format table; this CLAUDE.md addition is the version-lookup
companion for Claude Code agents specifically.

**Authors:** John M. Urban, Claude Code 2.1.126 (claude-sonnet-4-6 ; Effort: High)

**Scope:** CLAUDE.md

## [v0.14.90.1] — 2026-05-04 EDT

### Anti-deferral discipline — strict pathology-only rule + 4-step protocol + marker scheme baked into AGENT_CONVENTIONS + audit-loop-v3 + v4

**Scope:** `multi-agent/AGENT_CONVENTIONS.md` (new subsection in § Scope authority), `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md` (new failure mode #18 + R1/R2/R3 "must not" pointers), `multi-agent/workflows/phase-development-system-v4.md` (new failure mode #21a). Plus new memory file `feedback_deferral_pathology_only.md` (Principal-side instance, not tracked in repo).

**Trigger:** Phase 15 cycle 15.7a R3 round (2026-05-04) surfaced the recurring deferral-as-ordinary-disposition failure mode in concrete form. R3 (Claude Code 2.1.126 Opus Max) proposed framing-(a)+(c)-hybrid closeout with SPEC15.13/14/16 → DONE + SPEC15.15 → PARTIAL + new `[ISSUE:2026-05-04:1]` filed for the deferred SPEC15.15 d3/d4/d6/d8 work. Principal-orchestrator override 2026-05-04 reopened cycle 15.7a + issued a new R2 Stage G repair contract for the originally-scoped deliverables. Same pattern surfaced earlier at cycle 15.6a-S1 (R2's "all-levels-mean" trajectory clustering — Principal mental-simulation pushback forced redesign in-cycle). Recurring across all agents (Codex, Copilot, Claude). Principal observation: "Why are agents always pushing things to some far away future cycle when the future cycle is never really better than the original cycle?... the only thing that makes sense is to either not close this cycle because it is not done... OR we do it immediately in a follow-up supplemental cycle... I don't want them to continue thinking of deferring as an ordinary viable option. We do way too much planning for that."

**The fix is the strict pathology-only rule + 4-step protocol + cross-agent skepticism.**

Items addressed:

(1) **`multi-agent/AGENT_CONVENTIONS.md § Scope authority`** — appended new subsection "Deferral is NOT an ordinary disposition (critical — strict pathology-only rule)" (~100 lines). Codifies:
  - Default expectation for ALL agents (R1/R2/R3/orchestrator): implement what was scoped.
  - Sole carve-out: pathology (impossible OR genuinely harmful implementation). Concrete pathology examples: locked DECISIONS.md violation; cross-pipeline schema mismatch; algorithmic incoherence; data corruption; structurally-flawed design; missing upstream prerequisite; CRITICAL gate breakage; non-existent function/library/data.
  - NOT pathology: "big" / "complex" / "out of scope estimate" / "could be cleaner later" / "low priority" / "dependency not yet imported" / "I see an alternative approach."
  - 4-step protocol when pathology suspected: (a) any agent raises `[PATHOLOGY_CANDIDATE]` mid-flight (real-time chat + audit log) with required fields (deliverable affected; concrete pathology evidence with file:line citations; why pathological vs merely complex; alternatives attempted; suggested resolution); (b) block affected deliverable immediately (cycle's other deliverables continue); (c) independent audit by a different role produces `[PATHOLOGY_AUDIT_VERIFIED]` or `[PATHOLOGY_AUDIT_REJECTED]` (independent code-grounded verification, NOT acceptance of raising agent's reasoning); (d) Principal decides — `[PATHOLOGY_REJECTED:IMPLEMENT]` / `[PATHOLOGY_CONFIRMED_DEFER:<landing-zone>]` / `[PATHOLOGY_CONFIRMED_RESTRUCTURE]`.
  - Marker scheme (6 tokens): use verbatim in audit log + chat for grep-able provenance.
  - Per-role notes (any agent can raise; emphasis points only): R1 never decides deferrals during initial audit; R2 never silently emits sidecar/placeholder/TODO substitutes or files follow-up `[ISSUE:YYYY-MM-DD:N]` entries as deferral mechanisms; R3 errs strongly toward REJECTING deferrals — confirms only on independent code-grounded pathology verification, never on R2's reasoning alone.
  - Cross-agent skepticism (critical): all agents err toward REJECTING other agents' deferral plans; independent investigation required; default toward rejection.
  - Cycle-state implications: cycles with open `[PATHOLOGY_CANDIDATE]` markers stay OPEN; never close PARTIAL with un-decided pathology; orchestrator catches silent deferrals masquerading as PARTIAL closeouts and reopens.
  - Distinct concept "Alternative-approach proposals — distinct from pathology" with separate `[ALTERNATIVE_APPROACH_PROPOSAL]` marker + protocol — for SPEC amendment requests where agent sees a markedly better implementation approach (concrete reasoning: runtime/memory/parallelization/structural cleanliness/observed bug pattern; NOT vague preference). Different protocol (no independent audit required; Principal decides directly). Markers `[ALTERNATIVE_ACCEPTED:SPEC_AMENDED]` or `[ALTERNATIVE_REJECTED:IMPLEMENT_ORIGINAL]`.
  - Cross-references to `feedback_scope_authority.md` (broader scope-authority rule), `feedback_deferral_pathology_only.md` (this rule's short-form recall), cycle 15.7a Principal-orchestrator override 2026-05-04 (trigger lesson + worked example).

(2) **`multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md` § Common Failure Modes** — appended new failure mode #18 ("Deferral as ordinary disposition"). Cross-references the AGENT_CONVENTIONS canonical text + names cycle 15.7a R3 round + cycle 15.6a-S1 R2 round as worked examples + summarizes the 4-step protocol + per-role flavor + cross-agent skepticism + ALTERNATIVE_APPROACH_PROPOSAL distinction. Also appended pointer text to R1/R2/R3 "must not" sections directing agents to the canonical rule.

(3) **`multi-agent/workflows/phase-development-system-v4.md` § 13 Common failure modes** — appended new failure mode #21a between #21 (authorship drift) and #22 (Orchestrator drift toward implementation). Mirrors the v3 failure mode #18 content + cross-references the AGENT_CONVENTIONS canonical text. v4 is in-flight DRAFT; this addition lands cleanly alongside the existing failure modes.

(4) **NEW memory file `feedback_deferral_pathology_only.md`** (Principal-side, not tracked in repo). Short-form recall version of the rule for future Claude sessions: pathology examples + 4-step protocol summary + marker scheme + per-role flavor + cross-agent skepticism + ALTERNATIVE_APPROACH_PROPOSAL distinction. Distinct from existing `feedback_scope_authority.md` (broader scope-authority rule) — this is a sub-discipline of that. Index entry added to MEMORY.md.

Why three places (canonical + v3 + v4): `AGENT_CONVENTIONS` is canonical source-of-truth for cross-agent applicable rules; audit-loop-v3 § Common Failure Modes surfaces the rule in immediately-relevant active-cycle context for v3 phases (Phase 15 is on v2 archived but agents writing future v3 cycle launchers see this); v4 § 13 entry #21a is the v4 equivalent for future phases using v4. R1/R2/R3 "must not" pointers in v3 give role-specific reminders at the role boundary.

No code changes, no test changes, no user-facing doc changes — planning-surface engineering only (convention update + workflow file updates + memory). R2 Stage G implementation work-in-progress in the working tree is intentionally excluded from this commit (will land in R2's own commit at end of round).

**Cross-reference (cycle 15.7a Principal-orchestrator override commit 2026-05-04):** the orchestrator-outside-cycle commit immediately preceding this one captured the cycle-15.7a-specific override (revert R3's closeout-specific edits + append Stage G repair contract + re-frame HANDOFF/TASK). This DEVLOG entry generalizes the lesson into project conventions so future cycles default to the strict rule rather than re-hitting the same failure mode.

**Files changed:**
- `multi-agent/AGENT_CONVENTIONS.md` — new ~100-line subsection in § Scope authority.
- `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md` — new failure mode #18 + R1/R2/R3 "must not" pointers (~50 lines added).
- `multi-agent/workflows/phase-development-system-v4.md` — new failure mode #21a (~45 lines added).
- `multi-agent/DEVLOG.md` — this entry.
- NEW out-of-repo memory: `~/.claude/projects/-Users-johnurban-searchPaths-github-onionskin/memory/feedback_deferral_pathology_only.md`.

**Authors:** John M. Urban (Principal); Claude Code 2.1.126 (claude-opus-4-7 ; Effort: Max).

---

## [v0.14.89.10] — 2026-05-03 EDT

### Authorship/Effort attribution drift counter-discipline — confirm-as-first-action rule + authoritative-source rule baked into AGENT_CONVENTIONS + audit-loop-v3 + v4

**Scope:** `multi-agent/AGENT_CONVENTIONS.md`, `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md`, `multi-agent/workflows/phase-development-system-v4.md`. Plus new memory file `feedback_agent_self_attribution_unreliable.md` (Principal-side instance, not tracked in repo).

**Authors:** John M. Urban (Principal); Claude Code 2.1.126 (claude-opus-4-7 ; Effort: Max).

**Trigger:** Phase 15 cycle 15.6a-S1 R2 round (2026-05-03) surfaced the recurring authorship/Effort attribution drift failure mode in concrete form. Copilot's launcher specified `Reasoning: High` (per Principal's compromise from initial Medium proposal); Copilot's Stage A diagnostic authorship line wrote `Reasoning: Extra High` anyway. Same fabricate-plausible-value pattern documented in v4 § 13 failure mode #21 from prior occurrences. The Principal's observation: "all agents do this; agents cannot reliably introspect their reasoning level; the only true authoritative source is asking me." The fix is two-part counter-discipline plus an explicit authoritative-source taxonomy.

**Items addressed:**

1. **`AGENT_CONVENTIONS.md § Authorship conventions § Per-agent format § Rules`** — appended two bullets after the existing `(claude-sonnet-4-6 ; Effort: Extra High)` failure-mode-to-avoid bullet:
   - **Confirm the toggle as the FIRST IMMEDIATE ACTION of a round (when not in the launcher prompt).** Cites Phase 15 cycle 15.6a-S1 as the surfacing instance. The fix has two parts: (a) when drafting a launcher prompt for an agent, name the reasoning/effort level explicitly in the prompt body so the agent has nothing to fabricate (strongly preferred); (b) when the launcher does not name the level, agent must ask the user to confirm BEFORE any substantive work — catch the user at the keyboard before they walk away.
   - **What is and is not authoritative for the toggle value:** the user's explicit confirmation in the current chat is authoritative; STRATEGY-row primary/alt assignments are suggestive only (Principal often deviates per resource constraints); agent personal-memory entries are stale snapshots and NOT authoritative (the user changes the toggle at will whenever he sees fit); the agent's own self-reported authorship line is NOT authoritative (it's the failure mode being defended against); best guess when the user is unavailable is the most recent value confirmed in the current chat session.

2. **`spec_plan_three_role_audit_loop-v3.md § Common Failure Modes`** — appended new failure mode #17 covering the same authorship/Effort attribution drift pattern with the two-part counter-discipline (launcher names the level OR agent asks first) plus the authoritative-source taxonomy. Cross-references `AGENT_CONVENTIONS.md § Authorship conventions § Per-agent format § Rules` for the full rule.

3. **`phase-development-system-v4.md § 13 Common failure modes` entry #21 (Authorship/Effort attribution drift)** — expanded the existing bullet from a thin "Orchestrator periodically sweeps for drift" framing to the full counter-discipline + authoritative-source taxonomy. Adds the Phase 15 cycle 15.6a-S1 example alongside the earlier Phase 14 v0.14.69/v0.14.70.1 example (both are now documented historical instances of the same failure mode, supporting the "recurring across all agents" claim).

4. **NEW memory file `feedback_agent_self_attribution_unreliable.md`** — Principal-side personal memory at `~/.claude/projects/-Users-johnurban-searchPaths-github-onionskin/memory/feedback_agent_self_attribution_unreliable.md`. Captures the rule from the Principal's perspective: agents fabricate values; Principal must remind himself to name the level in launcher prompts OR confirm immediately when the agent asks. Cross-references the in-repo conventions for the agent-side rule. Not tracked in the repo (memory dir is outside the repo).

**Why these three places:**

- AGENT_CONVENTIONS is the canonical source-of-truth for authorship conventions; the rule must live there for cross-agent applicability.
- audit-loop-v3 § Common Failure Modes is where active-cycle agents look during their work; failure mode #17 surfaces the rule in the immediately-relevant context.
- v4 § 13 entry #21 is the v4 equivalent of v3's #17, expanded with the Phase 15 example and the explicit two-part counter-discipline. Future phases using v4 inherit the rule directly.
- The memory file is the Principal-side instance; reminds Claude (in future Claude sessions) to bake the confirmation step into every launcher prompt drafted.

**Validation:** No code changes. No test changes. No user-facing doc changes. Planning-surface engineering only — convention update + workflow file updates + memory.

**Files changed:**
- MODIFIED: `multi-agent/AGENT_CONVENTIONS.md` (+20 lines: 2 new bullets in Authorship § Per-agent format § Rules)
- MODIFIED: `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md` (+13 lines: new failure mode #17)
- MODIFIED: `multi-agent/workflows/phase-development-system-v4.md` (+33/-6 lines: expanded failure mode #21)
- MODIFIED: `multi-agent/DEVLOG.md` (this entry)
- NEW (out-of-repo): `~/.claude/.../memory/feedback_agent_self_attribution_unreliable.md`

---

## [v0.14.89.9] — 2026-05-03 EDT

### Phase development workflow v4 (DRAFT) — Pass 5f (fourth-round Copilot polish: Stage-5-fallback removal, v3-prompt-body legacy -v2 cleanup warning, stale line ranges dropped, "When To Use" omission DRAFT-aware reword)

**Scope:** `multi-agent/workflows/phase-development-system-v4.md` (2629 → 2659 lines; +54/-24 — Pass 5e's +3/-3 row swap was already committed under v0.14.89.8 at 895f3af, so this commit's Pass-5f-only diff is +54/-24 against that base).

**Authors:** John M. Urban (Principal); Copilot (fourth-round review of post-Pass-5e file v0.14.89.8); Claude Code 2.1.126 (claude-opus-4-7 ; Effort: Max).

v4 DRAFT — Pass 5f. v3 remains active default. Closes the fourth round of reviewer findings, this round from Copilot's review of v0.14.89.8 (Pass 5e). All four Copilot items addressed; no items deferred from this round.

**Items addressed (Copilot fourth-round review):**

1. **Stage-5 fallback removed from Stage 3 closeout (single AUDIT_LOG creation point).** v4 § 6 Stage 3 closeout still said the sibling `PHASE<N>_AUDIT_LOG.md` "is created at this point (or at the start of the first cycle in Stage 5)" — the parenthetical Stage-5 fallback contradicted Pass 5e's table + § 16 startup checklist which both make Stage 3 closeout the only allowed creation point. Pass 5f removes the parenthetical and explicitly states the rule: **Stage 3 closeout is the only allowed creation point under v4 (no Stage 5 fallback)** so that the artifact's existence is deterministic by the time Stage 4 / Stage 5 begin. Cross-references the § 1 lifecycle artifact-table and § 16 startup checklist for matching specification. The skeleton header markdown block is unchanged.

2. **v3-prompt-body legacy -v2 normalization warning added to § 14.** Copilot caught that v4 correctly says prompt bodies are still sourced from v3 until Pass 4c, but does not warn that the referenced v3 prompt bodies still contain legacy `-v2` self-identification text. `grep` against `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md` and `multi-agent/workflows/phase-development-system_PDS-v3.md` confirms many template lines that read `Read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md and follow it.`, `re-read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md`, and `You are acting as {ROLE_NAME} from multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md.` (carried forward from the v2 → v3 bump that was not exhaustively swept inside template bodies). The v2 paths are now archived under `multi-agent/workflows/archived/`, so cold-pasting these template bodies verbatim into a v4-launched prompt would (a) point the receiving agent at an archived file, and (b) signal the wrong workflow generation. Pass 5f adds an explicit "Legacy -v2 self-references inside v3 prompt bodies — REQUIRED cleanup before pasting" bullet to v4 § 14's "Notes on template usage under v4 DRAFT" subsection. The Orchestrator (or whoever drafts the prompt) MUST normalize all `-v2.md` self-references in pasted bodies to `-v3.md` (v3 still active) or `phase-development-system-v4.md` (once v4 promotes). Cleanup also lands as part of Pass 4c when prompt bodies are inlined into v4 § 14 directly (now documented as a Pass 4c prerequisite).

3. **Stale source-line-range policy added to Appendix A; PDS-v3 mapping table line ranges dropped.** Copilot caught that Appendix A's PDS-v3 mapping rows have stale line ranges in at least some rows (e.g., 0e row said "lines 698-770" but next section starts at 761; 0i row said "lines 890-930" but section actually starts at 902). The line ranges drift across patches and reviewers had no way to know which rows were stale. Pass 5f adds an explicit policy paragraph above the PDS-v3 mapping table: **section names are authoritative; line numbers are not maintained** (reviewers should `grep '^## <section name>'` in v3 source to locate a row's source). All explicit `(lines N-M)` ranges in PDS-v3 mapping rows have been dropped, replaced with full canonical PDS-v3 section heading text (e.g., "§ Typical form of the Opening brainstorm prompt to Agent 1 from Orchestrator, soup-to-brainstorm optional" instead of "§ Soup-to-brainstorm Opening prompt (lines 378-510)"). Policy explicitly notes that the audit-loop-v3 mapping table above is also covered (its ranges are best-effort but not maintained — same policy applies; ranges there were not regenerated as part of this pass to keep diff minimal — future pass can do the audit-loop-v3 row cleanup if reviewer flags it).

4. **"When To Use" omission rationale reworded for DRAFT-aware framing.** Copilot caught that Appendix A's audit-loop-v3 mapping row said "(omitted; v4 is the only system going forward)" with notes "v4 doesn't need a 'when to use' section — it IS the system" — but v4 is still DRAFT and v3 remains the active default. Pass 5f reframes: now reads "(omitted; v4 is intended to be the unified target workflow once promoted)" with notes "v4 doesn't need a 'when to use' section — it is being engineered as the unified target workflow that subsumes audit-loop's prior single-purpose framing. v3 remains the active default until v4 promotes; the omission rationale is forward-looking, not a claim that v4 is already active." Reviewer-facing mapping text no longer overstates activation status while the document is still under review.

**v4 file structure at end of Pass 5f:**
- 2659 lines total (2629 → 2659; +30 net after combining +54 insertions / -24 deletions for the Pass-5f-only diff)
- 16 numbered sections (§ 1 through § 16) + Appendix A
- 0 STUB tags remaining
- DRAFT status explicit in header; v3 remains active default until Pass 4c

**Reviewer credit:** Copilot's fourth-round review of v0.14.89.8 caught all four items in this Pass 5f. My self-audit caught none. The rotating-second-channel-reviewer pattern (Codex → Codex → Codex → Copilot across Pass 5b/5c/5d/5e/5f) continues to be the only reliable way to surface these polish-tier issues; recommend continuing rotation on future v4 polish rounds.

**Remaining work for v4 → active (unchanged from Pass 5e):**
- **Pass 4c (deferred):** full prompt-body inlining for all 22 template bodies (0a-0l + A-J), with legacy -v2 self-reference cleanup as a hard prerequisite (now documented in § 14 + appendix per Pass 5f Item 2).
- **Reviewer pass:** if either reviewer returns a fifth round of findings, those land as Pass 5g or fold into Pass 4c.
- **Promotion (separate commit, after stability):** update agent files + AGENT_CONVENTIONS.md + archive v3 files.

**Validation:** No new tests run — Pass 5f is markdown-only inside `multi-agent/workflows/`, identical risk profile to Pass 5b/5c/5d/5e. R3 test runs from earlier today (`make test` 205/1; `make full-twin` 4/4; `make puff-compare` 28/28; `make toy` clean) remain the latest validation snapshot.

No code changes. No test changes. No user-facing doc changes. Planning-surface engineering only.

**Files changed:**
- MODIFIED: `multi-agent/workflows/phase-development-system-v4.md` (2629 → 2659 lines after Pass 5f)
- MODIFIED: `multi-agent/DEVLOG.md` (this entry)

---

## [v0.14.89.8] — 2026-05-03 EDT

### Phase development workflow v4 (DRAFT) — Pass 5e (third-round Codex polish: § 1 lifecycle artifact-table realigned with § 16 startup checklist for AUDIT_LOG + SURPRISE_LOG creation stages)

**Scope:** `multi-agent/workflows/phase-development-system-v4.md` (2629 → 2629 lines; +3/-3, table-row content swap only).

**Authors:** John M. Urban (Principal); Codex (review of v0.14.89.7); Claude Code 2.1.126 (claude-opus-4-7 ; Effort: Max).

v4 DRAFT — Pass 5e. v3 remains active default. Closes the third round of reviewer findings from Codex's review of v0.14.89.7 (Pass 5d). Single polish item; no functional content change — only realigns the § 1 lifecycle artifact-table rows to reflect the canonical creation stages already documented in the § 16 startup checklist + the Pass 5d single-creation-point decision for SURPRISE_LOG.

**Item addressed (Codex third-round review):**

1. **Lifecycle artifact-table creation-stage realignment.** v0.14.89.7's table at § 1 still listed `PHASE<N>_AUDIT_LOG.md` and `PHASE<N>_SURPRISE_LOG.md` under Stage 5's "New artifacts produced" column even though prose under the same row already said AUDIT_LOG is created at SPEC lock and SURPRISE_LOG is created at Stage 4 — and the § 16 startup checklist explicitly creates AUDIT_LOG at Stage 3 (SPEC lock; checklist step 3) and SURPRISE_LOG at Stage 4 (Strategist kickoff; checklist step 4). The table-row-vs-checklist mismatch was a leftover from before Pass 5d's single-creation-point decision. Pass 5e moves the artifacts to the stages where they're actually created:

   - **Stage 3 row "New artifacts" now lists:** `PHASE<N>_SPEC.md` + `PHASE<N>_AUDIT_LOG.md` skeleton (created empty at SPEC lock — see § 16 startup checklist).
   - **Stage 4 row "New artifacts" now lists:** `PHASE<N>_STRATEGY.md` + `PHASE<N>_SURPRISE_LOG.md` empty-required skeleton (created before first Template A launcher — see § 16 startup checklist + skeleton). Stage 4 row "Active reads" expands to include AUDIT_LOG (which now exists from Stage 3) and the "Archived at end of stage" cell notes SPEC + STRATEGY + AUDIT_LOG + SURPRISE_LOG all stay live into Stage 5.
   - **Stage 5 row "New artifacts" now lists:** optionally `PHASE<N>_BACKGROUND.md` (per-phase planning scratchpad — created when deliberation surfaces non-trivial decision history); **cycle rounds append to existing AUDIT_LOG + SURPRISE_LOG** (no new top-level files at Stage 5 other than optional BACKGROUND).

   Net change: 3 row-content swaps in the lifecycle table (+3/-3 lines); no other text touched. Table now matches startup checklist. Reader sees consistent answer to "when is AUDIT_LOG/SURPRISE_LOG created" whether they look at the table, the prose paragraph, or the startup checklist.

**v4 file structure at end of Pass 5e:**
- 2629 lines total (unchanged — pure row-content swap)
- 16 numbered sections (§ 1 through § 16) + Appendix A
- 0 STUB tags remaining
- DRAFT status explicit in header; v3 remains active default until Pass 4c

**Reviewer credit:** Codex caught this in their third-round review of v0.14.89.7. My self-audit again missed it (same pattern as Pass 5d — Codex catches all). Cross-channel review continues to be the only reliable way to surface these table-vs-checklist drift issues; recommend Codex (or another second-channel reviewer) on every future v4 polish round.

**Remaining work for v4 → active (unchanged):**
- **Pass 4c (deferred):** full prompt-body inlining for all 22 template bodies (0a-0l + A-J).
- **Reviewer pass:** if Codex returns a fourth round of findings, those land as Pass 5f or fold into Pass 4c.
- **Promotion (separate commit, after stability):** update agent files + AGENT_CONVENTIONS.md + archive v3 files.

**Validation:** No new tests run — Pass 5e is markdown-only, identical risk profile to Pass 5b/5c/5d. R3 test runs from earlier today (`make test` 205/1; `make full-twin` 4/4; `make puff-compare` 28/28; `make toy` clean) remain the latest validation snapshot.

No code changes. No test changes. No user-facing doc changes. Planning-surface engineering only.

**Files changed:**
- MODIFIED: `multi-agent/workflows/phase-development-system-v4.md` (2629 lines unchanged; +3/-3 row swap)
- MODIFIED: `multi-agent/DEVLOG.md` (this entry)

---

## [v0.14.89.7] — 2026-05-03 EDT

### Phase development workflow v4 (DRAFT) — Pass 5d (second-round Codex review remediation: template count, self-contained qualifier, single SURPRISE_LOG creation point, Appendix A mapping refresh, Stage 2/3 discipline-rule cross-references)

**Scope:** `multi-agent/workflows/phase-development-system-v4.md` (2580 → 2629 lines; +70/-21).

**Authors:** John M. Urban (Principal); Codex (review of v0.14.89.6); Claude Code 2.1.126 (claude-opus-4-7 ; Effort: Max).

v4 DRAFT — Pass 5d. v3 remains active default. Closes the second round of reviewer findings from Codex's review of v0.14.89.6 (Pass 5b + 5c). All five Codex items addressed; no items deferred from this round.

**Items addressed (Codex second-round review):**

1. **Template count fixed: 13 → "all 22 template bodies (0a-0l + A-J; 12 + 10)."** Previous v0.14.89.6 said "13 templates" in three places (Appendix A "Items v3 had that v4 omits" first bullet + same paragraph's Pass-4c sentence + reviewer checklist). Codex's count: 0a-0l = 12 upstream-stage templates; A-J = 10 cycle-execution / Strategist / Final-Overseer / Succession templates; total 22 template bodies. Updated all three references to use "22 template bodies (0a-0l + A-J; 12 + 10)" wording so the count is unambiguous.

2. **"Self-contained" contradiction resolved via layered qualifier.** v0.14.89.6 said v4 file is "self-contained" if used directly (Migration path § + Appendix A In-flight v3→v4 §) but § 14 still required v3 source files for prompt bodies until Pass 4c lands. Pass 5d qualifies "self-contained" with **layer**: v4 is self-contained at the **policy / stage-structure layer** (conceptual framing, Orchestrator role, the eight stages, conventions, failure modes, migration paths, sibling-file skeletons), but **prompt bodies for all 22 templates remain cross-referenced to v3 source files until Pass 4c lands**. Reader is told either to wait for Pass 4c or to paste the relevant template body in from v3 alongside v4. Pass 4c stays on the v4 → active promotion gate per Appendix A reviewer checklist.

3. **Single SURPRISE_LOG creation point chosen: Stage 4 (Strategist kickoff), before first Template A launcher.** v0.14.89.6 said "created empty at Stage 4 or first cycle" in three places (Stage 5 artifact table + § 1 prose + § 16 sibling-file skeleton template + Stage 4 startup-checklist step that did not actually create it). Pass 5d picks Stage 4 as the canonical creation point, removes the "or first cycle" ambiguity from all three locations, and adds an explicit creation step to the Stage 4 startup-checklist ("Also create empty multi-agent/plans/PHASE<N>_SURPRISE_LOG.md before the first Template A launcher — required sibling artifact whose presence is a precondition for Templates A/B/C carry-over scans"). Now: one creation point. The phase ends up with at minimum a present-but-empty SURPRISE_LOG.md regardless of whether any surprise entries get filed.

4. **Appendix A mapping rows refreshed for Required Inputs + Phase Development Rules.** v0.14.89.6's mapping table still said Required Inputs were "Distributed: § 7 Stage 4 + § 8 Stage 5 + § 14 Templates" and Phase Development Rules were "Distributed across v4 § 1, § 2, § 3, § 6, § 8, § 12" — but Pass 5b had already migrated Required Inputs to a single canonical home in v4 § 1 (subsection "Required inputs / variable definitions (cycle execution)") and Pass 5c had migrated Phase Development Rules to a single canonical home in v4 § 8 (subsection "Phase Development discipline rules (stale-context handling)"). Pass 5d updates both rows to reflect "Full migration" with a single canonical destination + Pass version that landed the migration. Mapping table now matches the actual file structure.

5. **Stage 2 + Stage 3 cross-reference to § 8 discipline rules.** Phase Development discipline rules (Item E from Pass 5c) explicitly apply to every stage and every role, but the rules' canonical home is § 8 Stage 5 for locality of reference. Pass 5d adds short cross-reference paragraphs in § 5 Stage 2 (BRAINSTORM iteration; placed after Multi-pass iteration pattern) and § 6 Stage 3 (SPEC engineering; placed after SPEC engineering rounds) telling upstream-stage agents to follow the same § 8 discipline. Stage 2's pointer highlights the general re-read / show-snippet / only-trust-current-file / post-change-verify rules; Stage 3's pointer specifically names the "Be thorough during BRAINSTORM and SPEC engineering" subsection (CLI flag organization rules) + the "Aim for substantive priorities" subsection.

**v4 file structure at end of Pass 5d:**
- 2629 lines total (2580 → 2629; +49 net after combining +70 insertions / -21 deletions)
- 16 numbered sections (§ 1 through § 16) + Appendix A
- 0 STUB tags remaining
- DRAFT status explicit in header; v3 remains active default until Pass 4c

**Reviewer credit:** Codex's second-round review of v0.14.89.6 caught all five items in this Pass 5d. My own self-audit had not surfaced any of these gaps before Codex's review — the cross-checking pattern from Pass 5b + 5c (Codex catches some, I catch some) collapsed in this round to "Codex catches all"; my self-audit was insufficient as a sole review channel. Recommend continued use of Codex (or another second-channel reviewer) for v4 reviewer rounds going forward.

**Remaining work for v4 → active (unchanged from Pass 5b + 5c):**
- **Pass 4c (deferred):** full prompt-body inlining for all 22 template bodies (0a-0l + A-J). Estimated ~1500 additional lines; pushes v4 to ~4100 lines after Pass 4c lands.
- **Reviewer pass:** if Codex (or another reviewer) returns a third round of findings, those land as Pass 5e or fold into Pass 4c.
- **Promotion (separate commit, after stability):** update agent files + AGENT_CONVENTIONS.md + archive v3 files.

**Validation:** No new tests run for this pass — Pass 5d is markdown-only inside `multi-agent/workflows/`, identical risk profile to Pass 5b + 5c. R3 test runs from earlier today (`make test` 205/1; `make full-twin` 4/4; `make puff-compare` 28/28; `make toy` clean) remain the latest validation snapshot.

No code changes. No test changes. No user-facing doc changes. Planning-surface engineering only.

**Files changed:**
- MODIFIED: `multi-agent/workflows/phase-development-system-v4.md` (2580 → 2629 lines after Pass 5d)
- MODIFIED: `multi-agent/DEVLOG.md` (this entry)

---

## [v0.14.89.6] — 2026-05-03 EDT

### Phase development workflow v4 (DRAFT) — Pass 5b + 5c combined (reviewer-finding remediation: SURPRISE_LOG required, archived→live wording fix, Required-inputs subsection, Appendix A honest gaps, Phase 0-5 example skeleton, Phase Development Rules migration)

**Scope:** `multi-agent/workflows/phase-development-system-v4.md` (2324 → 2580 lines).

**Authors:** John M. Urban (Principal); Claude Code 2.1.126 (claude-opus-4-7 ; Effort: Max).

v4 DRAFT — Pass 5b (4 items) + Pass 5c (1 item) merged into one combined commit because each item alone is too thin to merit its own DEVLOG entry. v3 remains active default. v4 promotion still gated on reviewer + Pass 4c full template inlining; this commit closes out the gaps surfaced by Codex's review of v0.14.89.5 + my own self-audit. Pass 4c remains deferred.

**Items addressed (combined audit + Codex review gap inventory):**

1. **Item A (Pass 5b) — "archived phase artifacts" → "live phase artifacts" wording fix.** Two locations corrected: v4 § 2 Interaction model (line ~540) + v4 § 9 Stage 6 Final Overseer (line ~1377). The archive happens at Stage 8 (after Succession Briefing at Stage 7), NOT at Stage 6. Final Overseer reads the LIVE phase artifacts (SPEC, AUDIT_LOG, STRATEGY, BACKGROUND, SURPRISE_LOG) to reach independent conclusions. Codex caught this; my own audit missed it.

2. **Item B (Pass 5b) — SURPRISE_LOG required (not optional).** Three locations reframed: v4 § 1 phase artifacts table (changed Stage-5 row from "optionally PHASE<N>_SURPRISE_LOG.md" to "required, created empty at Stage 4 or first cycle"); v4 § 1 prose (replaced with explicit "SURPRISE_LOG required + BACKGROUND optional" subsection explaining mandatory R1 carry-over scan + R3 closeout review gate per Templates A/B/C); v4 § 16 sibling-file skeleton template (added complete empty SURPRISE_LOG.md skeleton block; changed "optional" to "required, created empty at Stage 4 or at the first cycle's R1 round, even if the phase ends up with no entries filed"). Codex caught this; my own audit missed it.

3. **Item F (Pass 5b) — added § 1 "Required inputs / variable definitions (cycle execution)" subsection.** Migrated from `audit-loop-v3 § Required Inputs` (lines 82-128). Defines TARGET_FILE, AUDIT_LOG_FILE, STRATEGY_FILE, TARGET_CYCLE, UPDATE_FILES, round-type enum, mode (2-role / 3-role), plus suggested assignment-line scaffold the Orchestrator embeds at the top of each cycle prompt. Generalizes for v4's single-file system; under v4 the Orchestrator drafts prompts so the Principal need not handle these directly. My own audit caught this; Codex missed it.

4. **Item G (Pass 5b) — softened Appendix A "Items v3 had that v4 omits" from "None expected" to honest gaps list.** Now lists four documented gaps: (i) full prompt-body inlining for the 13 templates 0a-0l + A-J (Pass 4c deferred); (ii) Template I § Agent selection and performance tuning — agent tier roster + per-role baselines (currently cross-referenced; lands with Pass 4c); (iii) PDS-v3 § Standard Copy-Paste Prompts (intentionally NOT MIGRATED — duplicative); (iv) PDS-v3 § Concrete Examples (intentionally NOT MIGRATED — historical orientation). Reviewer instruction restated: surface any OTHER omission as a finding. Codex caught this framing was too generous; my own audit caught the same.

5. **Item H (Pass 5b) — added Phase 0-5 SPEC skeleton to v4 § 3 § Phase-group SPEC structure.** Migrated ~10-line markdown example from `PDS-v3 § Phase-group SPEC structure` (lines 306-337). Shows H2-headed phase-groups with implementation-order + rationale lines + dependency-ordered priorities. Notes that the cycle name in PHASE<N>_AUDIT_LOG.md can literally track the phase-group label, and that priority identifiers (SPEC<N>.<idx>) stay stable across reorganization. My own audit caught this; Codex missed it.

6. **Item E (Pass 5c) — migrated PDS-v3 § Phase Development Rules into v4 § 8 as new "Phase Development discipline rules (stale-context handling)" subsection.** Verbatim migration of ~85 lines from `PDS-v3 lines 340-425`, placed after the 14 Shared rules and before AUDIT_LOG.md structure. Eight subsections: "Before making any decisions" (re-read modify-target files, no prior context, assume codebase changed); "For each file you modify" (show snippet, confirm assumptions, then apply); "Do not rely on" (previous audits / earlier session context / remembered file structure); "Only trust the current file contents"; "Before inserting into CHANGELOG.md or DEVLOG.md" (locate latest entry, confirm insertion point, route by primary content); "After making changes" (re-read modified files, confirm conventions, check AGENT_CONVENTIONS.md); "On cold starts" (read agent file → AGENT_CONVENTIONS → v4 workflow); "After finishing up" (re-read agent file + AGENT_CONVENTIONS at wrap-up checkpoint as deliberate quality guardrails); "Before giving wrap-up summary" (verify all asks completed); "Compaction awareness" (cold-start treatment after compaction); "Be thorough during BRAINSTORM and SPEC engineering" (CLI flag organization rules, universal vs pipeline-specific overrides, search markdown + code + KNOWN_ISSUES + BRAINSTORM); "Aim for substantive priorities" (cross-reference to § 3). Restates compaction-awareness alongside Shared rule #11 for emphasis (this discipline lives in both places under v3 + carried forward intact in v4). My own audit caught this; Codex missed it.

**Codex review credit:** Codex's review of v0.14.89.5 caught 3 of 4 gaps that my own audit missed (archived→live wording bug; SURPRISE_LOG optional/required mismatch; Appendix A "None expected" too generous). My own audit caught 3 gaps that Codex missed (Required-inputs subsection; Phase 0-5 example skeleton; Phase Development Rules migration). Cross-checking against both is what produced this combined Pass 5b + 5c gap inventory; neither audit alone would have caught all 6.

**v4 file structure at end of Pass 5b + 5c:**
- 2580 lines total (2324 → 2580; +256 net after combining +285 insertions / -29 deletions)
- 16 numbered sections (§ 1 through § 16) + Appendix A
- 0 STUB tags remaining
- DRAFT status explicit in header; v3 remains active default until reviewer + Pass 4c

**Remaining work for v4 → active (unchanged from Pass 5):**
- **Pass 4c (deferred from Pass 4b):** full prompt-body inlining for all 13 templates (0a-0l + A-J). Estimated ~1500 additional lines, pushing v4 to ~4000 lines after Pass 4c.
- **Reviewer pass:** Principal + reviewer agent walks Appendix A's reviewer-checklist. Pass 5b + 5c addressed the gaps from the first reviewer round; further rounds may surface more.
- **Promotion (separate commit, after stability):** update agent files + AGENT_CONVENTIONS.md + archive v3 files.

**Validation:** R3 test runs at 16:33-16:37 today (`make test` 205 passed / 1 skipped; `make toy` complete; `make full-twin` 4/4 passed; `make puff-compare` 28 files match) confirm no code-path regressions from this planning-surface work — expected since the changes are markdown-only inside `multi-agent/workflows/`.

No code changes. No test changes. No user-facing doc changes. Planning-surface engineering only.

**Files changed:**
- MODIFIED: `multi-agent/workflows/phase-development-system-v4.md` (2324 → 2580 lines after Pass 5b + 5c)
- MODIFIED: `multi-agent/DEVLOG.md` (this entry)

---

## [v0.14.89.5] — 2026-05-03 EDT

### Phase development workflow v4 (DRAFT) — Pass 5 (Appendix A v3→v4 mapping; merger structurally complete; awaiting reviewer + Pass 4c full template inlining)

**Scope:** `multi-agent/workflows/phase-development-system-v4.md` (2181 → 2324 lines).

**Authors:** John M. Urban (Principal); Claude Code 2.1.126 (claude-opus-4-7 ; Effort: Max).

v4 DRAFT — Pass 5 (final scaffold pass). v3 remains active default. v4 promotion gated on reviewer + Pass 4c full template inlining.

Pass 5 filled Appendix A — the v3→v4 mapping table for reviewer validation. Five sub-tables / sections:

1. **Audit-loop-v3 → v4 mapping** (17 sections of audit-loop-v3 mapped to v4 destinations). Confirms full migration of: Orchestrator role section (lines 132-510 → v4 § 2); Pre-cycle Phase Strategy (→ v4 § 7); Cycle Granularity (→ v4 § 3); Target File Structure (→ v4 § 8); CHANGELOG rules (→ v4 § 8 + § 12); Roles section (→ v4 § 8 + § 9); Shared Rules (→ v4 § 8); Standard Execution Loop steps 1-8 (→ v4 § 8 + § 9 + § 11); Required File Updates (→ v4 § 8); Acceptance + Closeout Rules (→ v4 § 8); Common Failure Modes (→ v4 § 13); Prompt Templates (→ v4 § 14 as templates-index with v3 cross-references; full inlining deferred to Pass 4c); 2-Role Mode (→ v4 § 15); Notes For The User (→ v4 § 16).
2. **PDS-v3 → v4 mapping** (PDS-v3 sections mapped). Confirms: Orchestrator role cross-reference section (already pointed to audit-loop-v3 § Orchestrator under v3) → v4 inlines canonical version; "What PDS is" → v4 § 1; "Four important files" → v4 § 1 (artifacts table reorganized as stage-by-stage); "Three main stages" → v4 § 1 (reframed as eight stages); "Substantive priorities" → v4 § 3; Phase 0-5 example → v4 § 3 + § 16; Phase Development Rules → distributed across v4 § 1, 2, 3, 6, 8, 12; 12 upstream-stage prompt templates → v4 § 14 cross-referenced.
3. **Orchestrator-companion file (v2; archived under v3) → v4** mapping. Confirms: Decision Guide → migrated to v4 § 2 (already in audit-loop-v3 § Orchestrator); Agent selection + performance tuning → lives inside Template I (cross-referenced); Standard Copy-Paste Prompts → NOT MIGRATED (duplicative with audit-loop-v3 Templates A-J); Concrete Examples → NOT MIGRATED (historical orientation only).
4. **Items v3 had that v4 omits** — none expected; reviewer-find prompt for surfacing any actually-omitted content as a finding.
5. **Items v4 adds beyond v3** — 7 new entries: eight-stage lifecycle spine; phase artifacts table; eight-stages-at-a-glance overview; § 13 v3+v4 failure modes (7 new entries from Phase 15 patterns); § 16 v4 stability criteria + v5 considerations; Appendix A v3→v4 mapping itself; cleaner end-to-end narrative flow.

Plus a reviewer checklist for v4 → active promotion (10-item gate covering content preservation, Stage transitions, Orchestrator role consistency, conventions match, failure modes coverage, templates index completeness, Pass 4c prerequisite, migration paths, no stubs, conditions for active promotion).

**v4 file structure at end of Pass 5:**
- 2324 lines total
- 16 numbered sections (§ 1 through § 16) + Appendix A
- 0 STUB tags remaining (all sections populated)
- DRAFT status explicit in header; v3 remains active default until reviewer + Pass 4c

**Remaining work for v4 → active:**
- **Pass 4c (deferred from Pass 4b):** full prompt-body inlining for all 13 templates (0a-0l + A-J). Currently Section 14 is templates-index with v3 cross-references; full inlining will replace cross-references with verbatim prompt bodies + v4 path updates throughout. Estimated ~1500 additional lines, pushing v4 to ~3800 lines.
- **Reviewer pass:** Principal + reviewer agent walks Appendix A's reviewer-checklist. May surface findings requiring Pass 5b polish.
- **Promotion (separate commit, after stability):** update agent files + AGENT_CONVENTIONS.md + archive v3 files. See Appendix A § "Reviewer checklist for v4 → active promotion" for the full checklist.

No code changes. No test changes. No user-facing doc changes. Planning-surface engineering only.

**Files changed:**
- MODIFIED: `multi-agent/workflows/phase-development-system-v4.md` (2181 → 2324 lines after Pass 5)
- MODIFIED: `multi-agent/DEVLOG.md` (this entry)

---

## [v0.14.89.4] — 2026-05-03 EDT

### Phase development workflow v4 (DRAFT) — Pass 4b (templates index with v3 cross-references)

**Scope:** `multi-agent/workflows/phase-development-system-v4.md` (2116 → 2181 lines).

**Authors:** John M. Urban (Principal); Claude Code 2.1.126 (claude-opus-4-7 ; Effort: Max).

v4 DRAFT — Pass 4b. v3 remains active default.

Pass 4b filled § 14 Prompt templates as a **templates index with v3 cross-references**. Decision rationale: the canonical prompt bodies for all 13 templates live in v3 source files (Templates 0a-0l in PDS-v3; Templates A-J in audit-loop-v3) totaling ~1700 lines of dense template prose. Migrating verbatim would push v4 to ~3700+ lines. Since v4 is a DRAFT being developed alongside still-active v3, a templates-index-with-cross-references is the right level of abstraction for the DRAFT phase. Full prompt-body inlining will happen in a Pass 4c polish round AFTER v4 stabilizes through review + at least one full-phase trial use.

The index is a 4-section table covering:

- **Stages 1-3 (12 upstream-stage templates 0a-0l):** Soup-to-BRAINSTORM transfer (Opening / Auditor / Follow-up / Closeout); BRAINSTORM iteration (Opening / Follow-up Opening / Returning Agent 1 / Returning Agents 2+); SPEC engineering (Opening / Subsequent Agent 2 / Subsequent Agent 1 / Additional Agent 2). Each entry lists purpose, when to use, and prompt-body source location in PDS-v3.
- **Stage 4 (Template I — Strategist):** Phase Plan of Attack STRATEGY.md generator. Cross-references audit-loop-v3.
- **Stage 5 (Templates A/B/C/H — cycle execution):** Initial Audit / Implementation / Re-Audit / Cycle Closeout After Skip-Reaudit. Each cross-references audit-loop-v3 with purpose + when-to-use notes.
- **Stage 6 (Templates D/E/F/G — Final Overseer + post-wrap-up):** Final Wrap-Up Audit / Post-Wrap-Up Triage / Post-Wrap-Up Implementation / Final Closeout After Wrap-Up Remediation. Cross-references audit-loop-v3.
- **Stage 7 (Template J — Succession Briefing):** Outgoing Orchestrator wisdom transfer. Cross-references audit-loop-v3.

Plus "Notes on template usage under v4 DRAFT" — explicit guidance for cold-paste from v3 source while v4 is still draft; reminder that templates inherit § 12 conventions; Orchestrator advisory pattern; STRATEGY.md drives agent assignments.

1 STUB remains: Appendix A v3→v4 mapping table (Pass 5).

No code changes. No test changes. No user-facing doc changes.

**Files changed:**
- MODIFIED: `multi-agent/workflows/phase-development-system-v4.md` (2116 → 2181 lines after Pass 4b)
- MODIFIED: `multi-agent/DEVLOG.md` (this entry)

---

## [v0.14.89.3] — 2026-05-03 EDT

### Phase development workflow v4 (DRAFT) — Pass 4a (conventions + failure modes + notes/migration)

**Scope:** `multi-agent/workflows/phase-development-system-v4.md` (1711 → 2116 lines).

**Authors:** John M. Urban (Principal); Claude Code 2.1.126 (claude-opus-4-7 ; Effort: Max).

v4 DRAFT — fourth pass (split into 4a + 4b due to size of templates section). v3 remains active default.

Pass 4a migrated 3 of 4 remaining sections (Section 14 templates deferred to Pass 4b):

- **§ 12 — Authorship + commit-block conventions.** Authorship line format (uniform `<harness> — <model> (<Toggle>: <value>)` form); authorship consolidation at cycle closeout (survey + dedupe from AUDIT_LOG); convention for agent-produced handoff prompts (5 rules: omit `Below N`, hard-code `N`, include guardrail blocks verbatim, precede with Orchestrator advisory, fenced-code-block format); convention for agent-produced git commit blocks (Form A mid-cycle; Form B closeout with `--file=<scratch>`; appending-to-already-committed-entry workflow; rules common to both forms; git-discipline principles — no interactive git, one concept per commit, implementer-splits-stages pattern); Principal owns git operations.
- **§ 13 — Common failure modes.** 16 v2-introduced failure modes (still active under v3+v4) verbatim from audit-loop-v3; 7 NEW v3+v4 failure modes from Phase 15 patterns (GAP-1 conflation of structural deliverables with calibration evidence; help-text contract simplification; empirical-default overfit; cross-pipeline parity blindspot; authorship/Effort attribution drift; Orchestrator role drift toward implementation; mid-flight pushback notes that bypass Principal authorization). Each failure mode includes counter-discipline guidance.
- **§ 16 — Notes / migration paths.** Notes for working with the file (Principal doesn't need to read end-to-end; Orchestrator does); migration paths (in-flight v3 → v4: don't until v4 stabilizes; v2 → v4: skip v3; v1 → v4: same as v1 → v2 + read v4 § 2); phase startup checklist (8 stages); per-phase sibling-file skeleton template (AUDIT_LOG initial); v4 stability criteria + v5 considerations deferred.

2 STUBs remain: § 14 Prompt templates (Pass 4b); Appendix A v3→v4 mapping table (Pass 5).

No code changes. No test changes. No user-facing doc changes.

**Files changed:**
- MODIFIED: `multi-agent/workflows/phase-development-system-v4.md` (1711 → 2116 lines after Pass 4a)
- MODIFIED: `multi-agent/DEVLOG.md` (this entry)

---

## [v0.14.89.2] — 2026-05-03 EDT

### Phase development workflow v4 (DRAFT) — Pass 3 (upstream stages from PDS-v3)

**Scope:** `multi-agent/workflows/phase-development-system-v4.md` (1411 → 1711 lines).

**Authors:** John M. Urban (Principal); Claude Code 2.1.126 (claude-opus-4-7 ; Effort: Max).

v4 DRAFT — third of five planned passes. v3 remains active default.

Pass 3 migrated 4 sections from PDS-v3 (the upstream stages):

- **§ 1 — Conceptual framing.** What PDS is; the four-tier brainstorming-surfaces hierarchy (`tracking/BRAINSTORM.md` reservoir → `plans/next/<THEME>_SOUP.md` → per-phase BRAINSTORM staging → SPEC); phase artifacts produced + consumed across the 8-stage lifecycle as a table; identifier system across artifacts (SOUP/BRAIN/SPEC IDs + Q/JQ/ISSUE/SURPRISE references); CHANGELOG vs DEVLOG routing summary; the eight stages at a glance.
- **§ 4 — Stage 1: SOUP.** File conventions (theme prefix + `_SOUP.md` suffix; never scope-anchoring; never numbered; SELF-reference ban while in `next/`); pre-promotion vs post-promotion locations; SOUP ID labeling at promotion (one-time, then read-only); promotion process (7-step `git mv` + label + author BRAINSTORM seed + iterate via FEEDBACK + auditor verifies + archive).
- **§ 5 — Stage 2: BRAINSTORM iteration.** BRAINSTORM file conventions (BRAIN IDs + `Source:` field citing SOUP IDs + tracking entries + prior archived plans); FEEDBACK file conventions (audit-log analog for engineering stages; Q/JQ identifiers; first entry triggered by SOUP-to-BRAINSTORM transfer); multi-pass iteration pattern (Agent 1 writer + Agent 2 auditor + Orchestrator-Principal deliberation).
- **§ 6 — Stage 3: SPEC engineering.** SPEC file conventions (substantive Priorities with SPEC IDs + `Source:`/`Covers:` field citing BRAIN IDs); SPEC ID freezing rule (reorderable during engineering, frozen at SPEC lock for cross-reference stability); SPEC engineering rounds (same multi-agent pattern as Stage 2); SPEC engineering closeout (BRAINSTORM + FEEDBACK archived; AUDIT_LOG sibling created with skeleton header).

5 STUBs remain: § 12 Authorship + commit-block conventions; § 13 Common failure modes; § 14 Prompt templates; § 16 Notes / migration paths; Appendix A v3→v4 mapping table. These land in Pass 4 + Pass 5.

No code changes. No test changes. No user-facing doc changes. Planning-surface engineering only.

**Files changed:**
- MODIFIED: `multi-agent/workflows/phase-development-system-v4.md` (1411 → 1711 lines after Pass 3)
- MODIFIED: `multi-agent/DEVLOG.md` (this entry)

---

## [v0.14.89.1] — 2026-05-03 EDT

### Phase development workflow v4 (DRAFT) — Pass 1 (skeleton) + Pass 2 (Orchestrator role + cycle execution mechanics) merged into single new file

**Scope:** NEW `multi-agent/workflows/phase-development-system-v4.md` (391 → 1411 lines so far).

**Authors:** John M. Urban (Principal); Claude Code 2.1.126 (claude-opus-4-7 ; Effort: Max).

v4 DRAFT — first two of five planned passes. v3 (the two-file system at `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md` + `multi-agent/workflows/phase-development-system_PDS-v3.md`) remains the **current active default** until v4 stabilizes through review + at least one phase of trial use. Phase 15 closed at v0.14.89 under v3; future phases may use v4 once approved.

**Pass 1 (skeleton).** Created the new file with header marking v4 as DRAFT / IN DEVLOPMENT / not yet active, full v1/v2/v3/v4 lineage in revision markers, migration path guidance (don't migrate in-flight phases to v4 until stable), table of contents (16 numbered sections + Appendix A v3→v4 mapping), and 17 STUB-tagged section scaffolds with bullet-list contents identifying what to migrate from v3 in each subsequent pass. The 8-stage lifecycle spine (SOUP → BRAINSTORM → SPEC engineering → Strategist kickoff → Cycle execution → Final Overseer → Succession Briefing → Phase Archive) is the unifying narrative thread.

**Pass 2 (Orchestrator role + cycle execution mechanics).** Migrated 8 sections from v3:

- **§ 2 — The Orchestrator role.** Full role definition migrated from audit-loop-v3 § "The Orchestrator (phase-spanning AI role; v3)". Lifecycle reframed to map to v4's Stages 1-8 (Stages 1-3 brainstorm + SPEC engineering; Stage 4 Strategist kickoff; Stage 5 cycle execution; Stages 6-8 wind-down). All subsections preserved: why the role exists, lifecycle, entry point starts at SOUP, multi-chat continuity, deliberation pattern (Principal ↔ Orchestrator), boundaries, interaction model, recommended model + agent choice, decision guide for cycle transitions, what gets archived at phase close.
- **§ 3 — Substantive priorities + cycle granularity.** Migrated from audit-loop-v3 § Cycle Granularity. Substantive vs thin priority distinction; phase-group SPEC structure; deferred R3 rules; who decides substantive vs thin.
- **§ 7 — Stage 4: Strategist kickoff.** Migrated from audit-loop-v3 § Pre-cycle Phase Strategy. STRATEGY.md production via Template I; living-document amendment-log discipline; stage transition out to cycle execution.
- **§ 8 — Stage 5: Cycle execution (audit-implement-reaudit).** Migrated full cycle execution mechanics from audit-loop-v3: Standard Execution Loop Steps 1-5; Role 1 + Role 2 role definitions (responsibilities, must-not, expected output, further clarifications); skip-reaudit criteria; shared rules for every cycle-execution role; AUDIT_LOG.md round-by-round structure; required file updates per cycle (SPEC, AUDIT_LOG, CHANGELOG, HANDOFF, TASK + optional ROADMAP/DECISIONS); acceptance and closeout rules.
- **§ 9 — Stage 6: Final Overseer.** Migrated from audit-loop-v3: independence requirement (fresh chat; structurally independent from Orchestrator's accumulated frame); Role 3 role definition (responsibilities, must-not, expected output); post-wrap-up remediation loop; Final Overseer normally one pass only.
- **§ 10 — Stage 7: Succession Briefing.** Migrated framing from audit-loop-v3 § Template J: what the briefing is + isn't; required sections enumerated; why Orchestrator writes (not Final Overseer); why AFTER Final Overseer signs off (not before). Full Template J prompt scaffold deferred to Pass 4.
- **§ 11 — Stage 8: Phase Archive.** Migrated from audit-loop-v3 § Standard Execution Loop Step 8: archive is NOT a cycle deliverable; happens after all cycles + Final Overseer + post-wrap-up remediation + Succession Briefing complete; files moved to archived/; Principal-run; why archive-last matters; what cycle-scoped closeout work IS in scope; what MAY include; audit-time check; outgoing Orchestrator chat closes.
- **§ 15 — 2-Role Mode.** Migrated as-is with one v4 addition (Final Overseer collapses into Role 1 in 2-role mode but independence discipline still applies).

**What remains for future passes:**

- **Pass 3 (upstream stages from PDS-v3):** § 1 Conceptual framing (four-tier brainstorming hierarchy + phase artifacts + 8-stage spine); § 4 Stage 1 SOUP; § 5 Stage 2 BRAINSTORM iteration; § 6 Stage 3 SPEC engineering.
- **Pass 4 (conventions + templates):** § 12 Authorship + commit-block conventions; § 13 Common failure modes; § 14 Prompt templates (consolidated A-J + upstream-stage templates 0a-0l from PDS-v3); § 16 Notes / migration paths.
- **Pass 5 (cohesion + Appendix A):** Deduplication sweep across all sections; cross-reference cleanup; Appendix A v3→v4 mapping table for review.

No code changes. No test changes. No user-facing doc changes. Planning-surface engineering only — building v4 alongside v3 as a draft. v3 remains active; agent files (CLAUDE.md, AGENTS.md, GEMINI.md, .github/copilot-instructions.md) and AGENT_CONVENTIONS.md continue to reference v3 as the current default.

**Files changed:**
- NEW: `multi-agent/workflows/phase-development-system-v4.md` (391 → 1411 lines after Pass 1 + Pass 2)
- MODIFIED: `multi-agent/DEVLOG.md` (this entry)

---

## [v0.14.88.4] — 2026-05-03 EDT

### Workflow files v2 → v3 bump — formalize the Orchestrator role + add Succession Briefing template; deprecate orchestrator-companion file (content migrated to audit-loop-v3 § Orchestrator); v1 + v2 + deprecated v3-orchestrator-companion all moved to archived/

**Scope:** `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md` (NEW; copied from v2 + substantially expanded with Orchestrator role section + Template J Succession Briefing); `multi-agent/workflows/phase-development-system_PDS-v3.md` (NEW; copied from v2 + Orchestrator role section + cross-references swept); `multi-agent/workflows/archived/` (v1 + v2 files moved here by Principal; v3 orchestrator-companion file moved here by orchestrator after content migration); `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `.github/copilot-instructions.md` (Workflow files section updated: v3 entry added as current default; v1 + v2 entries marked deprecated and archived); `multi-agent/AGENT_CONVENTIONS.md` (workflow-file path references swept v2 → v3).

**Authors:** John M. Urban (Principal; analytic framework + scope authority + role naming + entry-point decisions + archival operations); Claude Code 2.1.126 (claude-opus-4-7 ; Effort: Max) (Orchestrator; drafting + transcription + cross-reference sweeps).

Substantial dev-system engineering bump. Three workflow files were at v2; under v3 the system reduces to two active files (the orchestrator-companion file is deprecated since the AI Orchestrator role drafts custom prompts rather than copy-pasting standard templates).

**1. Orchestrator role formalized (spec_plan_three_role_audit_loop-v3.md § The Orchestrator).** A phase-spanning AI agent (typically Claude Code Opus Max) that begins at SOUP stage, drives BRAINSTORM iteration + SPEC engineering + Strategist kickoff + cycle execution + Final Overseer pass + Succession Briefing emission at phase close. Multi-chat continuity via memory + planning surfaces (the role is conceptually continuous; chat handoffs are normal + optional, recommended when context becomes a hindrance). Strategist function (formerly a one-shot pre-cycle role per Template I) absorbed as the Orchestrator's kickoff act. Bidirectional deliberation pattern (Principal ↔ Orchestrator) explicitly documented — Orchestrator surfaces options + drafts artifacts; Principal authorizes, refines, redirects; Orchestrator may push back on Principal's first instinct when surfacing deferred risks; mid-flight pushback notes drafted by Orchestrator + injected by Principal. Boundaries: never runs git, never narrows scope without Principal authorization, never acts as Final Overseer (Final Overseer's structural independence preserved). Recommended model: Claude Code Opus Max; Codex GPT-5.5 Extra High acceptable; Gemini excluded; Github Copilot acceptable for narrow R2 but not Orchestrator. Decision Guide for cycle transitions migrated from the deprecated v2 orchestrator-companion file as a new subsection.

**2. Template J — Succession Briefing (new in v3).** Outgoing Orchestrator's wind-down deliverable, written AFTER Final Overseer signs off (so Final Overseer's findings can be incorporated), BEFORE phase archive. Output to `multi-agent/plans/PHASE<N>_SUCCESSION_BRIEFING.md`. Wisdom transfer to next-phase Orchestrator: cross-cycle pattern memory, recurring blind spots, deliberation patterns that worked, things that didn't, brainstorm seeds, open SURPRISE/ISSUE entries with cross-phase implications, recommended Orchestrator working style for next phase. NOT a recreation of the formal artifacts (those archive intact); captures what's NOT in the formal artifacts. Final Overseer ≠ next Orchestrator (independence preserved); outgoing Orchestrator passes the baton via Succession Briefing.

**3. PDS-v3 (phase-development-system_PDS-v3.md).** Header bumped + revision marker + Orchestrator role cross-reference section near top clarifying AI Orchestrator vs Principal terminology. Cross-references swept v2 → v3 (templates inside the file now point at v3 workflow file). Future v4 plan documented (merge audit-loop-v3 + PDS-v3 into unified `phase_workflow-v4.md` after v3 stabilizes through one or two phases).

**4. Orchestrator-companion file (v3-bumped, then deprecated + archived).** The companion file `orchestrator.spec_plan_three_role_audit_loop-v2.md` was originally a copy-paste prompt source for the human orchestrator. Under v3, with the AI Orchestrator role formalized, the file's core "copy-paste prompts" purpose is obsolete (AI Orchestrator drafts cycle-specific custom prompts; standard templates aren't directly matchable). The file was bumped to v3 first (audience clarification + cross-reference updates + Section 0 reframed as Orchestrator's kickoff), then deprecated. Useful content (Decision Guide for cycle transitions) migrated into audit-loop-v3 § Orchestrator. Other sections (Standard Copy-Paste Prompts duplicative with Templates A-J; Concrete Examples historical orientation; Agent Selection duplicative with Template I) not migrated. The v3-bumped file moved to `archived/` alongside v1 + v2 of the same lineage.

**5. Archive structure.** Principal created `multi-agent/workflows/archived/` and moved the v1 + v2 files of all three lineages there. After Decision Guide migration, the v3-bumped orchestrator-companion file was also moved to archived/. Active workflow directory now contains only two v3 files: `spec_plan_three_role_audit_loop-v3.md` + `phase-development-system_PDS-v3.md`.

**6. Agent file Workflow-files sections updated** in `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `.github/copilot-instructions.md`. v3 entry added as current default; v1 + v2 entries marked deprecated (paths point at archived/). v3 entry documents the Orchestrator role highlights, Succession Briefing addition, orchestrator-companion deprecation, and v4 consolidation plan.

**7. AGENT_CONVENTIONS.md path references swept.** All `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md` references → v3 (path correctness). All `multi-agent/workflows/phase-development-system_PDS-v2.md` references → v3. One straggler reference (filename without prefix) also updated. Internal prose using "v2 workflow" terminology left as-is — those rules continue under v3 (v3 is additive over v2); future polish round can refresh prose terminology.

No code changes. No test changes. No user-facing doc changes (CLAUDE.md / AGENTS.md / GEMINI.md / .github/copilot-instructions.md are agent bootstrap files; AGENT_CONVENTIONS.md is dev-system convention spec).

**Files changed:**
- NEW: `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md`
- NEW: `multi-agent/workflows/phase-development-system_PDS-v3.md`
- NEW (then archived): `multi-agent/workflows/archived/orchestrator.spec_plan_three_role_audit_loop-v3.md`
- MOVED (Principal-run): `multi-agent/workflows/{spec_plan_three_role_audit_loop,phase-development-system_PDS,orchestrator.spec_plan_three_role_audit_loop}-v{1,2}.md` → `multi-agent/workflows/archived/`
- MODIFIED: `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `.github/copilot-instructions.md`, `multi-agent/AGENT_CONVENTIONS.md`, `multi-agent/DEVLOG.md` (this entry)

---

## [v0.14.88.3] — 2026-05-03 EDT

### SECOND orchestrator clarification on cycle 15.4a-S4 (post-Stage-A.5-results review) — DROP `learn` mode + GMM machinery + guard rails entirely; static defaults `LO=0, HI=1.75` (biology priors, not data-derived); supersedes R1 F8(4)-(7) + first orchestrator clarification

**Scope:** `multi-agent/plans/PHASE15_AUDIT_LOG.md` (cycle 15.4a-S4 section — new SECOND orchestrator clarification entry appended at top above R2 Stage A diagnostic, newest-first; cycle heading updated to note the second clarification).

**Authors:** John M. Urban (analytic framework + decision); Claude Code 2.1.126 (claude-opus-4-7 ; Effort: Max) (transcription).

R2's Stage A.5 empirical sweep produced the median-of-per-dataset-medians defaults `LO=0.6352, HI=1.3380` from the GMM log-space valley algorithm shipped under R1's F8 + the first orchestrator clarification. User review of the per-manifest plots at `dev/runs/15-4a-S4-stage-A/plots/` revealed that the GMM-derived valley algorithm produces ARTIFACT-tight cutoffs because background and amplification distributions overlap continuously (not bimodally) in the user fixture; the "valleys" the GMM finds are math-geometry boundaries (where the narrow central peak's PDF drops below the long-tail's), not biological boundaries. Plots show empirical cutoffs cut INTO visible background mass on most manifests; σ-floor sanity check fired on 7 of 9 manifests confirming the cutoffs were tighter than the background distribution's natural 2σ width.

**Decision (orchestrator-direction; user-confirmed 2026-05-03 late-afternoon):** drop `learn` mode entirely. The mode is too suggestive of being the "principled" choice when empirical evidence shows it isn't; removing it from the flag surface prevents users from picking it under that misimpression. Use biology-grounded static defaults `LO=0, HI=1.75` (no LO filter by default, since most samples don't have under-replication; HI=1.75 excludes ≥2-fold-amplified bins). Help text recommends `LO=0.25 to 0.5` for samples with under-replication (UR) domains.

**Drops from cycle 15.4a-S4 scope** (delta from R1 audit + first clarification): `--first-pass-background-bounds learn` value entirely; GMM/k-means infrastructure (no `sklearn.mixture.GaussianMixture` or `sklearn.cluster.KMeans` import; no fit logic; no valley-point computation; no σ-floor sanity check); hard guard rails (G_L_hard=0.25, G_R_hard=1.75); aliases `legacy/whole/naive/background` (only `none` retained as alias for `0,inf`); Stage B.3 (learn mode implementation); C.1 `test_compute_first_pass_background_mask_learn_mode`; σ-floor pushback discipline item.

**Retains:** `chrom-background-mad` as default sigma-source value (LD-1 unchanged; same concept — MAD over bins from chrom-median sample tracks restricted to first-pass background mask; with LO=0 the LO filter doesn't actively exclude anything, but HI=1.75 still excludes ≥2-fold-amplified bins, preserving the semantic distinction from `whole-chrom-mad`); all Stage A artifacts at `dev/runs/15-4a-S4-stage-A/` (script, sweep TSV, summary JSON, model-selection TSV, plots, R2 Stage A diagnostic report) as documentation; Stage B sub-contracts B.1, B.2, B.4, B.5, B.6, B.7, B.8, B.9, B.NaN (B.2 + B.9 reframed; others unchanged); Stage A.7 authorization gate (already passed).

**Reframed:** B.2 CLI spec — `--first-pass-background-bounds LO,HI` accepts numeric pair OR `none` alias only (default `0,1.75`); help text recommends `LO=0.25-0.5` for UR. B.9 calibration sweep — TSV still produced; F5 acceptance criterion downgraded from hard gate to evidence (chrom-background-mad preferred for being PRINCIPLED, not for producing measurably smaller sigma; if not materially smaller, documented and accepted). C.1 — drop learn-mode test; add `test_compute_first_pass_background_mask_default_LO_zero`. C.8 — TSV verification reframed as evidence + documentation, not pass/fail gate. B.9 TSV schema simplified to mirror cycle 15.4a-S3's calibration TSV extended with `chrom-background-mad` column (no GMM/valley/σ-floor columns).

**Now moot but preserved per append-only discipline:** R1 F2 (sklearn dep check), R1 F8 (GMM `learn` mode pipeline), R1 F9 (static-default sweep TSV schema with GMM columns), first orchestrator clarification (raw → log-space refit + valley + hard guard rail), BACKGROUND.md mid-Stage-A decision-history entry on log-space refit. Future agents reading the cycle 15.4a-S4 section see the trajectory: R1 proposed GMM + μ±3σ → first clarification refined to log-space + valley + guard rails → R2 Stage A executed and produced empirical evidence → SECOND clarification (this entry) dropped the entire approach in favor of biology priors.

Cycle 15.4a-S4 stays OPEN. Stage A.7 authorization gate passed; R2 launcher updated separately (orchestrator action). All R1 audit items not explicitly superseded by this clarification remain in force.

No code changes. No test changes. No user-facing doc changes. Planning-surface engineering only — drops a single algorithmic approach in favor of biology priors after empirical evidence showed the algorithm was unsound for the data.

**Files changed:** `multi-agent/plans/PHASE15_AUDIT_LOG.md`, `multi-agent/DEVLOG.md` (this entry).

---

## [v0.14.88.2] — 2026-05-03 EDT

### Orchestrator clarification on cycle 15.4a-S4 R1 audit F8(5) cutoff derivation rule (post-R1-audit; pre-R2-launch) — `μ ± 3σ` rule SUPERSEDED by **valley + hard guard rail** algorithm per user direction 2026-05-03

**Scope:** `multi-agent/plans/PHASE15_AUDIT_LOG.md` (cycle 15.4a-S4 section — new orchestrator clarification entry appended above R1's initial audit, newest-first; cycle heading updated to note the clarification).

**Authors:** John M. Urban (analytic framework + decision); Claude Code 2.1.126 (claude-opus-4-7 ; Effort: Max) (transcription).

R1's initial audit (2026-05-03; commit `d1c2b5d`) specified F8(5) cutoff derivation rule as `LO = μ_bg − 3σ_bg, HI = μ_bg + 3σ_bg` using the GMM background-component fit parameters. User pushback 2026-05-03 (post-R1-audit; pre-R2-launch) identified that this rule conflates two distinct concerns: **MAD-estimation correctness** (don't artificially narrow the MAD by truncating background distribution) and **mask-permissiveness** (don't admit amplification overlap into the background mask). R1's `μ ± 3σ` rule is correct for the first concern (a wide cutoff floor) but does NOT bound the second (e.g., `μ_bg=1.0, σ_bg=0.3` gives `HI=1.9`, admitting ~2-fold amplification signal as "background", which would inflate background-MAD with amplification variance — defeating the purpose).

**Refined cutoff derivation algorithm** (orchestrator clarification appended to PHASE15_AUDIT_LOG.md cycle 15.4a-S4 section): fit 2-3 component GMM → identify background component (centered ~1.0) → compute valley points V_L (between left-distribution and background) and V_R (between background and right-distribution = amplification) via joint-PDF argmin or PDF crossing → apply hard guard rails G_L_hard=0.25 and G_R_hard=1.75 (the latter rationaled as "HI > 1.75 admits ~2-fold amplification signal as background — unacceptable") → cutoffs are LO = max(V_L, G_L_hard) and HI = min(V_R, G_R_hard) → σ-floor sanity check (HI ≥ μ_bg + Nσ_bg AND LO ≤ μ_bg − Nσ_bg for chosen N) flags overlap-pathological fits but does NOT auto-widen the cutoffs.

**Surviving R1 work** preserved verbatim per append-only discipline: F1-F4, F5, F6, F7, F8(1)-(4), F8(6)-(8), F9 base schema, F10-F14, all Stage A sub-stages, all Stage B sub-stages, all Stage C contracts, validation gates, out-of-scope guardrails. Only F8(5) is refined; F9 TSV schema is extended with valley/guard-rail columns. Cycle stays OPEN; same R2 next-assignee. R2 prompt updated separately (orchestrator action; not in this DEVLOG entry) to reference the new clarification.

No code changes. No test changes. No user-facing doc changes. Planning-surface engineering only — refines a single audit-time finding before R2 begins Stage A.

**Files changed:** `multi-agent/plans/PHASE15_AUDIT_LOG.md`, `multi-agent/DEVLOG.md` (this entry).

---

## [v0.14.88.1] — 2026-05-03 EDT

### Orchestrator housekeeping post-cycle-15.4a-S3 closeout — R3 authorship correction (Effort: High → Max), forward-looking Opus Effort sweep to Max, CROSS_PIPELINE_UNIFICATION_SOUP.md Item 7 added (posterior-inherited-background design space)

**Scope:** `multi-agent/plans/PHASE15_AUDIT_LOG.md` (cycle 15.4a-S3 R3 entry Authors line + Decision-section consolidated authorship); `multi-agent/project_context/HANDOFF.md` (R3 closeout cold-start orientation + Last Action consolidated authorship + cycle 15.4a-S4 next-assignee line); `CHANGELOG.md` (`[v0.14.88]` Authors line); `multi-agent/plans/PHASE15_STRATEGY.md` (cycle table rows 1-10 + amendment rows + Final Overseer line); `multi-agent/project_context/TASK.md` (cycle 15.4a-S4 task entry); `multi-agent/plans/next/CROSS_PIPELINE_UNIFICATION_SOUP.md` (Item 7 added).

**Authors:** John M. Urban, Claude Code 2.1.126 (claude-opus-4-7 ; Effort: Max).

Three orchestrator-housekeeping operations between cycle 15.4a-S3 closeout (v0.14.88; CHANGELOG cut at HEAD `8774ddc`) and cycle 15.4a-S4 R1 launch.

**1. R3 attribution correction.** Cycle 15.4a-S3's Role 1 Re-Audit (Template C, 2026-05-02) was performed at Effort: Max but recorded as Effort: High in five places: `PHASE15_AUDIT_LOG.md` R3 Authors line + Decision-section consolidated authorship; `HANDOFF.md` cold-start orientation R3 attribution + Last Action consolidated authorship; `CHANGELOG.md` `[v0.14.88]` Authors line. All five flipped to `Effort: Max` to match what the round actually was.

**2. Forward-looking Opus Effort sweep to Max.** Per user direction 2026-05-03 ("I will be using Max for all probably"). All `Claude Code — Opus (Effort: High)` and `Claude Code — Opus (Effort: Extra High)` references in `PHASE15_STRATEGY.md` cycle table (rows 1-10 + amendment rows 4a-S2 / 4a-S3 / 4a-S4 / 4b / 6b / 6a-S1 / 7b) + Final Overseer line + Deviation rationale text; `TASK.md` cycle 15.4a-S4 task entry; `HANDOFF.md` cycle 15.4a-S4 next-assignee line — all flipped to `Claude Code — Opus (Effort: Max)`. Affects both open + closed cycles' planning-document records (the audit-log historical attributions for closed cycles are NOT flipped — those record what was actually used at the time). The strategy table is now uniform on Max for all R1+R3 Opus entries; documents the going-forward standard for the remaining Phase 15 cycles (15.4a-S4, 15.6a-S1, 15.7a, 15.7b, 15.8a, 15.9a, 15.10a) plus the post-15.10a Final Overseer pass.

**3. `CROSS_PIPELINE_UNIFICATION_SOUP.md` Item 7 added — posterior-inherited-background `specific` vs `union` vs hybrid design space.** Captures the future-looking design call for `--posterior-inherited-background` per user direction during cycle 15.4a-S4 R1 launcher pre-check. Cycle 15.4a-S4 lands the controller-level FIRST-pass mask only (cross-pipeline shared by virtue of being computed once pre-pipeline at the controller level). Cycle 15.7a (re-scoped SPEC15.15) lands per-pipeline pass-2 background-region detection + posterior-inheritance with `specific` default. Future phase explores `union` (conservative; treat anything any pipeline excluded as excluded) and three hybrid options (intersection / majority / weighted-union / pipeline-confidence-derived) with empirical-evaluation open questions. Item 7 also documents the SOUP-vs-FLAT_SOUP routing rationale (posterior-inheritance-of-pass-2 = cross-pipeline composition, structurally cross-pipeline-mask-unification, hence this SOUP rather than FLAT_SOUP's upstream-prior-architecture theme). Forward-pointer to be added from the cycle 15.4a-S4 R1 audit's existing out-of-scope guardrail when the audit is written.

No code changes. No test changes. No user-facing doc changes.

**Files changed:** `multi-agent/plans/PHASE15_AUDIT_LOG.md`, `multi-agent/plans/PHASE15_STRATEGY.md`, `multi-agent/plans/next/CROSS_PIPELINE_UNIFICATION_SOUP.md`, `multi-agent/project_context/HANDOFF.md`, `multi-agent/project_context/TASK.md`, `CHANGELOG.md`, `multi-agent/DEVLOG.md` (this entry).

---

## [v0.14.86.1] — 2026-04-30 EDT

### Versioning rule revision — `X` bumps at phase final closeout (was: phase start); new `R` component for Release Bundle counter

**Scope:** `CHANGELOG.md` preamble, `multi-agent/AGENT_CONVENTIONS.md` § CHANGELOG.md and DEVLOG.md (split-streams subsection + format-spec list), `multi-agent/DEVLOG.md` (Format + Versioning sections), `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md` Templates D + G.

**Authors:** John M. Urban, Claude Code 2.1.117 (claude-opus-4-7 ; Effort: Max).

User-driven rule revision, surfaced 2026-04-30 mid Phase 15 cycle 15.6b closeout window. The prior X-bump rule ("`X` bumps when a new ROADMAP phase begins shipping product changes; first product entry of new phase = `v0.X.00`") was operationally awkward: cycle 15.4b should have triggered a bump from v0.14.YY → v0.15.00 since it was the first Phase 15 product-code-touching entry, but the bump never happened and Phase 15 development continued on the v0.14.YY counter. Rather than retroactively renumber, the user proposed flipping the trigger to phase-end and re-defining the variable so existing version history validates under the new rule.

**1. New rule (CHANGELOG version format `vR.X.YY`).**
- `R` = number of Release Bundles shipped. `R=0` pre-release; bumps at Release Bundle ship as part of release protocol (Release Bundle protocol itself is not yet engineered — TBD).
- `X` = number of fully-completed development phases (equivalently, the most recent completed Phase number). During Phase 15 development, `X=14` because 14 phases were complete. The phase final-closeout CHANGELOG entry bumps `X` to the just-completed phase number and is itself versioned `vR.X.00` — for Phase 15 close: `v0.15.00`.
- `YY` = monotonic CHANGELOG-entry counter; increments by 1 with each CHANGELOG entry. Resets to `00` at the phase-final-closeout `vR.X.00` milestone. Behavior at Release Bundle ship is TBD.

**2. Why this is cleaner.** Existing version history (Phase 15 work shipping under v0.14.77-86) is correct under the new rule — `X=14` because 14 phases are complete during Phase 15 development. No retroactive renumber needed. The leading `0` was previously unnamed ("pre-release marker"); under the new rule it gets a real semantic (`R = release bundles shipped`). Phase boundaries become clean `vR.X.00` milestones marking completed phases, not opening shots.

**3. Variable-naming option chosen — Option 1 (R X Y) over Option 2 (R N X).** The letter `X` already meant "major phase" across existing docs (`AGENT_CONVENTIONS.md` line 195: *"X = major phase, aligned with ROADMAP Phase numbers"*). Option 1 leaves `X` in place with the same meaning — only its bump trigger flips and a new leading `R` prepends. Option 2 would have renamed `X→N` for the phase letter, forcing rewrites across every existing schematic reference. Option 1 minimizes doc churn while still cleanly distinguishing release-facing (R) from development-facing (X) cadence.

**4. Files touched.**
- `CHANGELOG.md` preamble (lines 23–46): replaced versioning paragraph with R/X/YY definitions + Historical note flagging the rule flip and confirming Phases 1–14 are not retroactively renumbered.
- `multi-agent/AGENT_CONVENTIONS.md` § CHANGELOG.md and DEVLOG.md: revised "Split version streams" subsection with R/X/YY component definitions and revised X-bump trigger; revised format-spec bullet list (lines for `R`, `X`, `YY`); added Historical note paragraph.
- `multi-agent/DEVLOG.md` Format + Versioning sections: updated schematic to `vR.X.YY.Z`; added R/X/YY component breakdown; revised anchor-selection text to reflect that during a phase, CHANGELOG `X` stays at the prior phase number.
- `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md` Template D (Final Wrap-Up Audit): added phase-final X-bump responsibility note with conditional ("if this is genuinely the phase's last CHANGELOG entry") and default-to-not-bumping safety rule when remediation may follow.
- `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md` Template G (Final Closeout After Wrap-Up Remediation): added phase-final X-bump responsibility note (this template's closeout entry is the last-of-phase by construction).

**5. Deliberate non-changes.**
- Phases 1–14 CHANGELOG entries NOT retroactively renumbered (per user direction).
- v0.14.77 through v0.14.86 (Phase 15 in-progress entries) NOT renumbered — they're correct under the new rule.
- Schematic `v0.X.YY` references in workflow files (e.g., `CLOSED v0.X.YY` cycle-section format on line 220, 248, 270 of v2 workflow) NOT swept to `vR.X.YY` — the existing schematic remains correct as a placeholder for "the version this entry got" and adding `R` would imply false specificity. The full `vR.X.YY` form is documented in CHANGELOG/DEVLOG/AGENT_CONVENTIONS preambles where it's load-bearing.
- `new-rule.txt` scratch file at repo root deleted post-merge (no longer needed).

**6. Open question parked for future engineering.** Behavior at Release Bundle ship: when `R` bumps from 0→1, does `YY` reset to `00` (giving a clean `v1.<X>.00` milestone) or continue from the prior counter (giving `v1.<X>.<continuing-YY>`)? Answered when the Release Bundle protocol itself is engineered. The current convention text reads "TBD" on this point.

**Files changed:** `CHANGELOG.md`, `multi-agent/AGENT_CONVENTIONS.md`, `multi-agent/DEVLOG.md`, `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md`, `new-rule.txt` (deleted).

---

## [v0.14.84.1] — 2026-04-30 EDT

### Cross-pipeline structural unification — SURPRISE #8 promoted to a new SOUP file + step-number-leakage filed as KNOWN_ISSUES + AGENT_CONVENTIONS naming rule curbing step-number-in-code-identifiers

**Scope:** new `multi-agent/plans/next/CROSS_PIPELINE_UNIFICATION_SOUP.md` (5 items, 4 open brainstorm questions) + `multi-agent/plans/next/SUMMIT_SOUP.md` (cross-reference paragraph added) + `multi-agent/tracking/KNOWN_ISSUES.md` `[ISSUE:2026-04-30:2]` (HMM pipeline step-number leakage with three-category scope + per-cycle in-phase fix guidance) + `multi-agent/plans/PHASE15_SURPRISE_LOG.md` `[SURPRISE:15:15.6a:1:A:8]` status flip from Active → Promoted with postscript + `multi-agent/AGENT_CONVENTIONS.md` § Code-identifier naming (new top-level section curbing step-numbers-in-identifiers).

**Authors:** John M. Urban, Claude Code 2.1.117 (claude-opus-4-7 ; Effort: Max).

Triggered by user-surfaced observation post-cycle-15.6a closeout: *"`run_step16_hmm_aps` is a bad name choice — step numbers are not necessarily stable and that type of name causes an alignment burden."* Pattern check across the codebase showed the issue is way bigger than 5 function names: ~450+ step-numbered occurrences across ~20 files (242 in `engines/hmm_engine.py` alone), across three categories — step-numbered dict keys in `build_hmm_steps()`, step-numbered function names, step-numbered parameter and local-variable names. Growth and RMS use semantic identifiers throughout; HMM is the cross-pipeline asymmetry. Each Phase 15 step renumber (cycle 15.2a, cycle 15.4b) forces sweep updates across the leakage surface; sweeps are routinely incomplete (cycle 15.4b Final Overseer caught 5 missed sites; cycle 15.6a R1 caught additional SPEC line annotation drift). The function/param/variable names being step-numbered is the load-bearing cause of the recurring maintenance cost.

**1. New SOUP file: `multi-agent/plans/next/CROSS_PIPELINE_UNIFICATION_SOUP.md`.** Captures cross-pipeline structural unification as a future-phase candidate. Five items: (1) `build_hmm_steps()` semantic-key refactor + 242 engine call site updates [highest-leverage single change]; (2) function/parameter/variable renames cascading from Item 1; (3) HMM-thinner-APS-path surfaces audit + port (`build_amplicon_importance`, `build_scalar_orderings`, `write_aps_plots` — all Growth+RMS-only today); (4) abstract-base-class scaffold to prevent future drift (ABC vs Protocol vs lint-script open question); (5) AGENT_CONVENTIONS guards. Four open brainstorm questions: phase boundaries (one phase or phase-group?); SUMMIT_SOUP merger?; ABC-vs-Protocol-vs-lint for Item 4; backward-compat for public CLI vs internal renames.

**2. SUMMIT_SOUP.md cross-reference.** New paragraph at top of SUMMIT_SOUP positions it as one specific instance (summit-algorithm parity) of the broader cross-pipeline structural divergence pattern captured in CROSS_PIPELINE_UNIFICATION_SOUP. The two SOUPs cross-reference each other; they may graduate independently or merge at graduation time.

**3. `[ISSUE:2026-04-30:2]` in KNOWN_ISSUES.** Concrete actionable inventory of HMM pipeline step-number leakage with three-category scope (dict keys, function names, parameter/variable names) + per-cycle in-phase fix guidance for Phase 15's remaining cycles. Mandatory in-flight rename: cycle 15.7a renames `run_step17_hmm_saps` → `run_hmm_saps` as part of net-new SAPS implementation. Bulk Category 2+3 renames land in cycle 15.9a (SPEC15.21 sub-deliverable). Category 1 (`build_hmm_steps` semantic-key refactor + 242 engine call sites) deferred to future phase per CROSS_PIPELINE_UNIFICATION_SOUP graduation. Issue exit condition: codebase grep-clean of step-numbered code identifiers (excepting intentional surfaces).

**4. SURPRISE_LOG status flip.** `[SURPRISE:15:15.6a:1:A:8]` (the high-weight cross-pipeline-structural-divergence meta-observation that triggered this work) flipped from Active → Promoted to CROSS_PIPELINE_UNIFICATION_SOUP + `[ISSUE:2026-04-30:2]`. Original four-hypothesis interpretation analysis preserved for provenance. User's effective direction was a hybrid of hypotheses (b) and (c): the workflow needed cross-cycle drift-detection (Template C closeout review gates + the SURPRISE_LOG file itself are the workflow-level fix per `v0.14.83.1`); the structural-unification SOUP is the codebase-level fix.

**5. AGENT_CONVENTIONS § Code-identifier naming (new top-level section).** Curbs the behavior that surfaced this issue: forbids pipeline step numbers in function names, parameter names, local variables, and dict keys. Allows them on output directory names + user-facing diagnostic strings + intentional algorithm-step artifacts (e.g., `_summits_step1.tsv` referring to the 4-step convergence). Documents the cross-pipeline consistency check (Growth + RMS already comply; HMM is the asymmetry awaiting `[ISSUE:2026-04-30:2]` refactor). Future code reviews + audits enforce. Cross-references the SOUP + KNOWN_ISSUE + SURPRISE entry.

**Why this matters.** The cross-pipeline structural divergence pattern that the user has been calling out for multiple cycles (per `feedback_cross_pipeline_parity_blindspot.md`) now has both a workflow-level mitigation (the v2 workflow's SURPRISE_LOG closeout review gates from `v0.14.83.1`) AND a codebase-level home for the structural fix (the new SOUP + the active KNOWN_ISSUE for the most concrete subset). Each Phase 15 cycle's discrete cross-pipeline-port priority (SPEC15.4 ODW, SPEC15.6 reliability, SPEC15.7 summit refinement, SPEC15.8 summit↔timing convergence, SPEC15.18 `--peak-summary` extension, SPEC15.22 gap-analysis) closes one microcosm of the pattern; future-phase structural-unification work absorbs what remains. The naming-convention rule prevents future instances of the leakage from accruing.

**Phase 15 in-flight effects on remaining cycles:**
- Cycle 15.6b (next, determinism diagnostic) — unaffected unless adjacent renames are natural to in-flight work.
- Cycle 15.6a-S1 (Stage D APS analytics) — unaffected.
- Cycle 15.7a (HMM-specific late-phase work) — **mandatory in-flight rename** of `run_step17_hmm_saps` → `run_hmm_saps` since SAPS is net-new code (don't perpetuate the pattern in net-new code).
- Cycle 15.8a (cross-pipeline analysis-surface parity) — opportunistic in-flight renames + SPEC15.17 master APS catalog deliverable now informs Item 3 of the new SOUP.
- Cycle 15.9a (architectural cleanup) — bulk Category 2+3 renames added as a SPEC15.21 sub-deliverable (per `[ISSUE:2026-04-30:2]` landing zone).
- Cycle 15.10a (Phase closeout) — fallback landing zone for any Category 2+3 renames not picked up earlier; KNOWN_ISSUES sweep includes verifying `[ISSUE:2026-04-30:2]` exit condition.

**Files changed:** `multi-agent/AGENT_CONVENTIONS.md`, `multi-agent/DEVLOG.md`, `multi-agent/plans/PHASE15_SURPRISE_LOG.md`, `multi-agent/plans/next/CROSS_PIPELINE_UNIFICATION_SOUP.md` (new), `multi-agent/plans/next/SUMMIT_SOUP.md`, `multi-agent/tracking/KNOWN_ISSUES.md`.

---

## [v0.14.83.1] — 2026-04-30 EDT

### Per-phase SURPRISE_LOG.md introduced — peripheral-vision observations sibling to AUDIT_LOG, with status taxonomy + three-venue triage model wired into v2 workflow

**Scope:** new sibling planning-surface file class (`PHASE<N>_SURPRISE_LOG.md`); `multi-agent/AGENT_CONVENTIONS.md` § SURPRISE_LOG.md (new section) + planning-content routing table row + three-triage-venues subsection; `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md` § Target File Structure (new "What goes in PHASE<N>_SURPRISE_LOG.md" + "Phase startup — creating per-phase sibling files" subsections) + Templates A/B/C/D additions (carry-over scans, status-update guidance, optional new-entry filing, closeout review gates, end-of-phase sweep); `multi-agent/plans/PHASE15_SURPRISE_LOG.md` (canonical first instance) status taxonomy expansion.

**Authors:** John M. Urban, Claude Code 2.1.117 (claude-opus-4-7 ; Effort: Max).

The SURPRISE_LOG accumulates **fresh-eyes observations** that an agent noticed during a phase activity but deliberately did NOT include in the formal AUDIT_LOG report — adjacent drift, hygiene-drift-adjacent items, smells without concrete repair, opinions on drift from user intent, cross-cycle pattern observations, heads-up notes for future cycles. The file fills a gap between AUDIT_LOG (strictly in-scope per cycle SPEC contract), KNOWN_ISSUES (long-lived actionable issues with concrete exit conditions), and BRAINSTORM (speculative future-direction ideas). Each of those existing surfaces has its own scope constraint that systematically filters out the *peripheral vision* category — observations that look off but don't justify a finding-with-repair-contract right now. SURPRISE_LOG is that home, accumulating across cycles + phases + roles, so the user can ask "did you notice anything off that did not make it into your formal report?" and get a written record back.

**1. Per-phase sibling file convention.** `PHASE<N>_SURPRISE_LOG.md` joins `PHASE<N>_SPEC.md` / `PHASE<N>_AUDIT_LOG.md` / `PHASE<N>_STRATEGY.md` / `PHASE<N>_FEEDBACK.md` as the fifth canonical per-phase planning-surface file. Created at phase startup by copying preamble + Conventions sections verbatim from the latest live `PHASE<N-1>_SURPRISE_LOG.md` (or from `PHASE15_SURPRISE_LOG.md` as the canonical reference for first-time use). Archived alongside its siblings into `multi-agent/plans/archived/` at phase close per the workflow's existing archive convention.

**2. Status taxonomy designed to prevent Active-forever drift.** Entries pass through `Active` (initial state) → `In-flight (cycle X.Y stage Z | [ISSUE:...] | DECISIONS [date])` (actively being addressed but not yet verified closed) → `Resolved (vX.Y.ZZ)` (verified closed at re-audit / version commit). Side states: `Promoted to <destination + ID>` (substantive home moved to KNOWN_ISSUES / BRAINSTORM / DECISIONS / SPEC), `Superseded by [SURPRISE:...]` (sharper observation in same log), `Logged-only` (recorded but not actionable). **No entry should remain Active forever** — at re-audit closeout, Active entries either flip to one of the other states or get an explicit deferral postscript naming a specific future cycle / SPEC ID / phase-closeout milestone.

**3. Three triage venues.** A SURPRISE entry transitions out of Active at one of three venues, each with the same three-outcome menu (integrate / elevate / dismiss-with-reasoning):
- **Orchestrator + assistant orchestrator (outside the cycle process).** User can triage open entries between cycles when drafting the next R1 launcher. An assistant agent can *propose* an integration via launcher language; only the user authorizes scope per the scope-authority rule.
- **Cycle Role 1 (Template A) at initial-audit time.** R1 scans Active SURPRISE entries whose Likely landing zone tags this cycle and triages each. Disposition recorded both in the audit log and on the SURPRISE entry's status line.
- **Cycle Role 1 re-audit (Template C) at closeout — MANDATORY.** R3 reviews two populations: (a) entries authored during this cycle, (b) Active/In-flight entries whose Likely landing zone tagged this cycle. Missing this triage is what re-introduces Active-forever drift; the workflow now requires the review gate.

A fourth venue (Final Overseer / Template D at phase wrap-up) carries entries forward to the next phase or marks them resolved if prior-cycle work closed them but the per-cycle re-auditor missed flipping the status.

**4. Workflow file (`spec_plan_three_role_audit_loop-v2.md`) integration.**
- New § Target File Structure subsection "What goes in PHASE<N>_SURPRISE_LOG.md" alongside the existing SPEC + AUDIT_LOG sections.
- New § Phase startup subsection on creating per-phase sibling files.
- **Template A** (Role 1 initial audit): two new mandatory blocks — KNOWN_ISSUES carry-over scan (issues whose Likely landing zone tags this cycle must be triaged) + SURPRISE_LOG carry-over scan (Active surprises whose Likely landing zone tags this cycle must be triaged). Plus optional new-entry filing block.
- **Template B** (Role 2 implementation): SURPRISE status update guidance (flip to In-flight when picking up an entry as in-cycle work) + optional new-entry filing block.
- **Template C** (Role 1 re-audit): mandatory SURPRISE_LOG closeout review gate covering both this-cycle-authored entries and Active/In-flight entries tagged-here. Disposition options: Resolved with version cite / In-flight with destination cite / Promoted with destination ID / Active-with-postscript / Logged-only.
- **Template D** (Role 3 final wrap-up): SURPRISE end-of-phase sweep — phase-spanning carry-forward or close-out for any remaining Active/In-flight entries. Plus optional new-entry filing block.

All filing is **opt-in for any role** (auditor / implementer / re-auditor / overseer); triage at the appropriate venues is **mandatory**. The "Quality > coverage" discipline is explicit in each template — agents should not write filler entries to fulfill a perceived quota.

**5. KNOWN_ISSUES carry-over scan added to Template A.** Adjacent enhancement: Template A now has an explicit instruction to scan `multi-agent/tracking/KNOWN_ISSUES.md` at audit-start for entries whose Likely landing zone tags the cycle. This formalizes the pattern that was previously ad-hoc (e.g., the cycle 15.6a R1 launcher manually called out `[ISSUE:2026-04-30:1]` as a carry-over). The Template A scan is now the standing convention; per-launcher carry-over callouts remain valid for cycle-specific emphasis.

**6. AGENT_CONVENTIONS.md additions.** New `## SURPRISE_LOG.md (per-phase) — peripheral-vision observations` section sibling to the existing § KNOWN_ISSUES.md and § BRAINSTORM.md sections. Documents location, purpose, in/out scope, status taxonomy summary, promotion paths, authoring discipline, three-triage-venues subsection, and three-outcome menu. New row added to the planning-content routing table for the SURPRISE_LOG content type.

**7. PHASE15_SURPRISE_LOG.md preamble updates.** Status taxonomy expanded from 4 states (Active / Resolved / Superseded / Logged-only) to 6 states (adds In-flight + Promoted) with explicit verification gates. New "When to flip status (which agent role does what)" subsection lists the role-by-role responsibilities aligned with the workflow's per-template additions. New invariant statement: "Active-forever is a smell — explicit deferral with a landing zone is the discipline."

**Why this matters.** The audit-implement-reaudit workflow's per-cycle scope discipline keeps audits focused but also creates filtering pressure: anything an agent notices that is "out of scope" was previously asked to either be silently dropped or be filed in KNOWN_ISSUES (actionable) or BRAINSTORM (speculative future). The SURPRISE_LOG fills the peripheral-vision gap. The status taxonomy + three-venue triage model + Template C mandatory review gate together prevent the new surface from becoming a graveyard of unactioned smells — every entry has a clear path out of Active state, and re-audits enforce the path.

**Files changed:** `multi-agent/AGENT_CONVENTIONS.md`, `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md`, `multi-agent/plans/PHASE15_SURPRISE_LOG.md`, `multi-agent/DEVLOG.md`.

---

## [v0.14.76.5] — 2026-04-28 EDT

### Phase 15 (HMM completeness + enrichment) SPEC engineering stage — CLOSED (consolidated cycle entry)

Phase 15 SPEC engineering stage closes after five iteration rounds
spanning 2026-04-28 through 2026-04-28 with the user-gated decision
to lock the SPEC. Codex Agent 2 declared CLEAN at the Round 5 final
audit (FEEDBACK § "SPEC engineering Round 5 final audit"). Per the
PDS-v2 closeout flow, `PHASE15_BRAINSTORM.md` and `PHASE15_FEEDBACK.md`
archive to `multi-agent/plans/archived/`; `PHASE15_SPEC.md` stays live
as the audit/implement-stage contract; sibling `PHASE15_AUDIT_LOG.md`
is created with a skeleton header (cycle 15.1a OPEN).

**Cycle artifacts at closeout.**

- `multi-agent/plans/PHASE15_SPEC.md` — **20 substantive priorities
  (SPEC15.1 through SPEC15.20) organized into 9 phase-groups (Phase
  1–9; Phase 0 holds 4 RESOLVED markers without SPEC IDs).** Each
  phase-group becomes one batched cycle in the audit/implement stage
  (cycles 15.1a through 15.9a). All 34 BRAIN entries cited
  (mechanically auditable; SPEC-to-BRAIN coverage check passes).
  Universal naming + design decisions section locks 21 cross-cutting
  rules inherited by audit/implement.
- `multi-agent/plans/archived/20260428-PHASE15_BRAINSTORM.md` —
  archived this round. 34 BRAIN entries (BRAIN15.1–34) covering all
  21 SOUP IDs from the archived
  `multi-agent/plans/archived/20260428-PHASE15_HMM_SOUP.md` + 9
  user-introduced no-SOUP-source entries + 4 RESOLVED markers.
- `multi-agent/plans/archived/20260428-PHASE15_FEEDBACK.md` —
  archived this round. Full Q&A trail across the soup-to-brainstorm
  transfer (Round 1, Round 1 follow-up, Round 2, Round 2.5, Round 3
  brainstorming aggregation, Round 4 brainstorming Q15-Q22, Round 5
  brainstorming Q23-Q24, Codex brainstorming-stage code/tracking
  audit, Round 6 brainstorming follow-up + addenda, Codex narrow
  re-audit, SPEC engineering Round 1, Codex SPEC Round 1 audit, SPEC
  engineering Round 2, Codex SPEC Round 3 audit, SPEC engineering
  Round 3 + post-commit correction + SUMMIT_SOUP port, Codex SPEC
  Round 4 audit, SPEC engineering Round 4 editorial cleanup, Codex
  SPEC Round 5 final audit).
- `multi-agent/plans/PHASE15_AUDIT_LOG.md` — created this round with
  skeleton header per PDS-v2 § 2 ("the sibling AUDIT_LOG is created
  at this point...with a skeleton header"). Cycle 15.1a opens with
  status "OPEN (awaiting Role 1 initial audit)."

**SPEC engineering rounds (consolidated authorship trail).**

- **Round 1 (2026-04-28):** Agent 1 first-pass authoring — 20 SPEC
  priorities authored; 9 open questions (JQ25–JQ33) raised.
- **Round 1 audit (2026-04-28):** Codex (Agent 2) audit — verdict
  OPEN with 6 substantive findings + 8 JQ resolutions.
- **Round 2 (2026-04-28):** Agent 1 triage — all 6 Codex findings
  ACCEPTED + applied; all 8 JQ resolutions LOCKED in design-decisions
  rules 10-19; 4 own-pass additions (Makefile sweep, user-facing doc
  cross-alignment sweep, stage-1 detection mechanism explicit lock,
  default-flip protocol structure).
- **Round 3 audit (2026-04-28):** Codex (Agent 2) audit — verdict
  OPEN with 5 findings framed around the user's direction (chat
  2026-04-28) to generalize 5 concepts to all pipelines in Phase 15
  (not deferred).
- **Round 3 (2026-04-28):** Agent 1 triage — all 5 Codex findings
  ACCEPTED + applied; ~95% of all-pipeline generalization landed in
  Phase 15 (SPEC15.4 + SPEC15.6 + SPEC15.7 + SPEC15.8 substantively
  rewritten with cross-pipeline sub-deliverables); design-decisions
  rules 20 + 21 added (rule 20 = user-requested all-pipeline
  generalization scope; rule 21 = HMM-only concepts that legitimately
  stay HMM-specific).
- **Round 3 follow-up — SUMMIT_SOUP port (2026-04-28):** post-commit
  code-dive verification at user request revealed RMS already has
  sliding-offset (Round-3-initial framing was wrong); cross-pipeline
  parity questions ported from briefly-committed `PHASE16_SPEC.md`
  (deleted) into `multi-agent/plans/next/SUMMIT_SOUP.md` (Item 5
  expansion + Open brainstorm questions 6-10) per user direction.
- **Round 4 audit (2026-04-28):** Codex (Agent 2) audit — verdict
  OPEN-but-small (no substantive design gaps; only stale Phase 16 /
  HMM-only wording in the live SPEC).
- **Round 4 (2026-04-28):** Agent 1 editorial cleanup — all 5 Codex
  editorial findings ACCEPTED + applied (rule 16 superseded-pointer;
  status-table cross-pipeline labels; SPEC15.7 goal correction;
  Self-audit Round 3 HISTORICAL RECORD header + per-bullet SUPERSEDED
  markers; HANDOFF cold-start cleanup).
- **Round 5 audit (2026-04-28):** Codex (Agent 2) final audit —
  verdict CLEAN; SPEC engineering closeout candidate.

**Substantive design content captured in the SPEC.**

The SPEC inherits all design content from the archived BRAINSTORM
+ FEEDBACK without re-litigation. Highlights of the 21 locked
cross-cutting design decisions:

- Bayesian Origin Detection Window (ODW) System with 3 detector
  modes + 2 tunable knobs + new dedicated `--odw-*` argparse group
  (replaces the 1.25× hard-threshold rule for stage-over-stage
  active-growth detection).
- Cross-pipeline rename: `latest_activity_stage` → `last_active_stage`;
  `peak_rcn_stage` retired in favor of existing `max_rcn_stage`;
  `summit_excess` → `summit_rcn` (raw-RCN migration with floor
  semantics opt-in for sample-level aggregation).
- 4-metric naming framework (`onset_stage`, `last_active_stage`,
  `max_rcn_stage`, `regression_stage`).
- All-pipeline generalization scope (rule 20): ODW, summit
  stage-selection, summit↔timing convergence, reliability
  scoring, APS feature modes, clustering defaults.
- HMM-only concepts that remain HMM-specific (rule 21): SAPS, HMM
  trajectory clustering, ghost levels, narrowest summit-state
  strategy.
- Step-less `plots/` and `notebooks/` directories cross-pipeline.
- Migration-with-provenance convention extends across all tracking
  files at phase close.
- 1.25× hard rule retired in favor of Bayesian framework.
- Stage 1 growth detection uses background/chrom-median comparison
  under same variance-aware primitive (no special-case stage-1
  thresholds).
- HMM 0-based statepath default-flip protocol structurally codified
  (7a land flag / 7b validate / 7c conditional flip in same cycle OR
  explicit user-approved deferral).
- HMM two-pass renorm directory layout LOCKED to option (B) with
  `.pass_metadata.json` sentinel for atomicity + provenance.
- APS sample posterior clustering vs HMM trajectory/fork clustering
  kept distinctly named (both Phase 15 deliverables).
- Class-aware reliability scoring minimum rule + 4-DS1-chr-II
  calibration-amplicon regression gate.

**Deferred to a future summit-methodology phase via SUMMIT_SOUP.md.**

Per Round 3 follow-up + Round 5 confirmation: cross-pipeline
summit-refinement-algorithm parity completion (port
`refine_summit_parabola` to Growth + HMM; potentially unify HMM's
local `_estimate_parabola_bp` with shared; potentially generalize
HMM's multi-stage-aware filtering) lives at
`multi-agent/plans/next/SUMMIT_SOUP.md` § Item 5 expansion + Open
brainstorm questions 6-10. Items 2 + 3 + 4 in SUMMIT_SOUP.md got
migration-with-provenance markers documenting their absorption into
Phase 15. SUMMIT_SOUP graduates to its own phase via standard PDS-v2
SOUP → BRAINSTORM → SPEC lifecycle when the orchestrator decides
(post-Phase-15 close).

**Archive operations executed at closeout (orchestrator-driven).**

- `git mv multi-agent/plans/PHASE15_BRAINSTORM.md multi-agent/plans/archived/20260428-PHASE15_BRAINSTORM.md`
- `git mv multi-agent/plans/PHASE15_FEEDBACK.md multi-agent/plans/archived/20260428-PHASE15_FEEDBACK.md`

The archived `multi-agent/plans/archived/20260428-PHASE15_HMM_SOUP.md`
(SOUP file, archived at v0.14.76.2 brainstorming-stage transfer
closeout) is unchanged.

**Verification posture.** SPEC IDs sequential (SPEC15.1 through
SPEC15.20); BRAIN coverage complete (BRAIN15.1–34 all cited);
substantive-priorities rule satisfied (no thin priorities standing
alone); no active Phase 16 references in the live SPEC (all hits are
historical/provenance-labeled per Codex Round 4 + Round 5 grep
verification). `make test` not run (zero product code touched in
SPEC engineering work). No auditor pass needed for the closeout
itself — Codex already verified the SPEC at the Round 5 final audit
(verdict CLEAN).

**Next step.** Audit/implement stage opens per
`multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md`. Cycle
15.1a (Phase 1: HMM CLI correctness; SPEC15.1) is the first
implementation cycle. Sibling `PHASE15_AUDIT_LOG.md` records
round-by-round history of audit/implement work going forward.

**Authors:** John M. Urban, Claude Code 2.1.112 (claude-opus-4-7 ; Effort: High), Codex (gpt-5-codex ; Reasoning: Extra High)

**Scope:** `multi-agent/plans/PHASE15_SPEC.md` (20-priority / 9-phase-group SPEC; locked at SPEC engineering closeout); `multi-agent/plans/archived/20260428-PHASE15_BRAINSTORM.md` (archived from `multi-agent/plans/PHASE15_BRAINSTORM.md` via `git mv`); `multi-agent/plans/archived/20260428-PHASE15_FEEDBACK.md` (archived from `multi-agent/plans/PHASE15_FEEDBACK.md` via `git mv`); `multi-agent/plans/PHASE15_AUDIT_LOG.md` (NEW — skeleton header per PDS-v2 § 2); `multi-agent/project_context/HANDOFF.md` + `multi-agent/project_context/TASK.md` (closeout refresh).

---

## [v0.14.76.4] — 2026-04-28 EDT

### Phase 15 (HMM completeness + enrichment) brainstorming stage — CLOSED (consolidated cycle entry)

Phase 15 brainstorming stage closes after six iteration rounds + one
broad Codex Agent 2 code/tracking audit + one narrow Codex Agent 2
re-audit. The brainstorming stage opened immediately after the
soup-to-brainstorm transfer closed at v0.14.76.2 with a 29-BRAIN-entry
baseline (BRAIN15.1–29) covering all 21 SOUP IDs. Iteration expanded
the BRAINSTORM to **34 BRAIN entries** (BRAIN15.1–34), substantively
deepened ten of the existing entries (BRAIN15.5/7/8/17/18/19/22/24/27/28/33),
and produced one cross-cutting design system (the Bayesian Origin
Detection Window System) that did not exist at transfer-closeout time.
Codex's narrow re-audit verdict: **SUBSTANTIVELY CLEAN** — no missing
HMM-completeness pillar, no need for another broad brainstorming pass.
The single bookkeeping-fix recommendation (stale `v0.14.77` reference
at `PHASE15_BRAINSTORM.md:6`) was applied at closeout.

**Cycle artifacts at closeout.**

- `multi-agent/plans/PHASE15_BRAINSTORM.md` — 34 BRAIN entries
  (BRAIN15.1 through BRAIN15.34) covering all 21 SOUP IDs + 9
  user-introduced no-SOUP-source entries (BRAIN15.24/25/26/27/28/29/31/33/34)
  + 4 RESOLVED markers (BRAIN15.13/15/30/32). Lifecycle-position
  line updated to drop the stale `v0.14.77` audit-version reference
  (the audit happened, but no CHANGELOG version was cut for it).
- `multi-agent/plans/PHASE15_FEEDBACK.md` — full Q&A trail across
  Round 3 aggregation pass + 8 Codex audit findings + 2
  user-prompted findings (Round 3); Round 4–6 substantive design
  follow-ups; Round 6 Bayesian ODW System addenda 1, 2, 3; Codex
  narrow re-audit appended at line 1612.
- `multi-agent/project_context/DECISIONS.md` 2026-04-28 entry — APS
  feature mode candidates accepted/rejected catalog, RCN-vs-excess
  decision (gates BRAIN15.27 + BRAIN15.28). Authored mid-cycle to
  prevent re-litigation during SPEC engineering.
- `multi-agent/plans/next/APS_SOUP.md` and
  `multi-agent/plans/next/UNIFIED-RCN_SOUP.md` — SOUPs created
  mid-cycle to capture deferred items + future-phase candidates
  surfaced during brainstorming. Includes
  `[ISSUE:2026-04-22:1]` migration with full provenance per the
  KNOWN_ISSUES migration rule (codified at v0.14.76.3).
- `multi-agent/tracking/KNOWN_ISSUES.md` — `[ISSUE:2026-04-22:1]`
  removed after migration to APS_SOUP.md.

**Substantive design content captured during the brainstorming stage.**

- **Bayesian Origin Detection Window (ODW) System (BRAIN15.7
  deliverable item 6 + Round 6 addenda 1/2/3 in FEEDBACK).** The
  legacy 1.25× hard-threshold rule for stage-over-stage growth
  detection (currently at `onionskin_core/timing.py:370–393`) is
  retired in Phase 15 in favor of a parameterized statistical system
  exposed via a new dedicated `--odw-*` argparse group. Three
  detector formulations (`hardthreshold` legacy / `gaussian_overlap`
  / `bayesian_posterior` default) selectable via
  `--odw-active-stage-detector`, with two tunable knobs:
  `--odw-prob-threshold` (default `0.9`) and `--odw-fold-threshold`
  (default `1.25`). The `bayesian_posterior` default tests posterior
  P(ratio ≥ fold-threshold) ≥ prob-threshold using a log-ratio
  Normal model with MAD-derived sigma — preserves the
  biological-meaningfulness anchor of the 1.25× threshold while
  adding variance-awareness from the per-stage MAD bedGraphs
  shipped at v0.5.47. The Gaussian-overlap formulation reuses the
  existing dip-test machinery at `onionskin_core/aps.py:993`
  flipped for the upward direction. Outputs feed `onset_stage`,
  `last_active_stage`, the per-transition active mask, amplicon
  classification, and (optionally) `regression_stage` —
  conceptually a system, not a single-metric detector. The new ODW
  argparse group joins the BRAIN15.16 cross-cutting Universal-in-spirit
  list alongside APS, Timing, Shape scoring, Overlap, Asymmetric
  Triangle Model.
- **Four-metric naming framework (BRAIN15.24).** `onset_stage`
  (already exists) + `last_active_stage` (renaming target —
  currently `latest_activity_stage` at
  `onionskin_core/timing.py:300–317, 466–468, 511–527`) +
  `max_rcn_stage` (already exists at
  `onionskin_core/aps.py:1016`) + `regression_stage` (already
  exists at `onionskin_core/aps.py:1017`). All four are kept as
  distinct metrics with single canonical names; `peak_rcn_stage`
  is officially retired as a noun. Cross-pipeline rename
  contained to `onionskin_core/timing.py` plus downstream callers.
- **HMM completeness gap-audit table (BRAIN15.5).** Comprehensive
  matrix of analysis families × pipeline coverage with explicit
  Phase 15 entry-pointers. Not a "build everything" list — many
  cells are already covered by other BRAIN entries; the table is
  the cross-reference index that ties them together.
- **APS clustering-concept disambiguation (BRAIN15.27 +
  BRAIN15.19).** Splits APS sample posterior clustering (HMM
  step-14, `aps_clusters.tsv`,
  `onionskin_core/hmm_ported_analyses.py:149–184`) from HMM
  trajectory/fork clustering (HMM step-16→17,
  `trajectory_clusters.tsv`, same file at lines 380–439). Composite
  multi-feature APS clustering modes + 3-layer PCA design captured
  in BRAIN15.27 with concrete file targets including
  `scripts/aps_cluster_experiments.py` (currently growth-only).
- **HMM summit refinement per-amplicon stage-selection menu
  (BRAIN15.8).** Six candidate stage-selection strategies catalogued
  for SPEC engineering to compare; ODW-confined refinement;
  sliding-offset sub-grid considerations.
- **HMM shape-score + multistage unification + state-path growth
  credibility (BRAIN15.18).** Within-ODW non-monotonic-growth
  handling; sub-priorities a/b/c/d/e for SPEC engineering to
  partition.

**Tracking-cleanup BRAIN entries marked RESOLVED during the cycle
via direct code audit.**

- BRAIN15.30 (HMM seq filtering CLI flags) — already implemented
  at `onionskin_core/engines/hmm_engine.py:121–130, 1000–1002`.
- BRAIN15.32 (HMM fork age tracking) — already implemented at
  `onionskin_core/hmm_fork_travel.py:12, 242–243`.

These RESOLVED markers reflect the user-flagged pattern that
`tracking/BRAINSTORM.md` had become stale ("we should do X" entries
where X was already done). Default rule going forward: verify in
code first before opening a new BRAIN entry from a tracking-file
hint.

**Codex audit + re-audit (Agent 2) contributions.**

- Initial broad code/tracking audit (mid-Round-3): 8 substantive
  findings + 2 user-prompted findings — all ACCEPTED in Round 6
  follow-up; folded into existing BRAIN entries; no new BRAIN IDs
  created per Codex's own recommendation.
- Narrow re-audit after Round 6 (appended at FEEDBACK line 1612):
  verdict SUBSTANTIVELY CLEAN; one bookkeeping correction (stale
  `v0.14.77` audit-version label in BRAINSTORM lifecycle line) —
  applied at closeout. Codex also refreshed `HANDOFF.md` and
  `TASK.md` mid-audit so post-compaction agents would not chase
  already-addressed Round 6 work.

**Bookkeeping correction at closeout.**

- `multi-agent/plans/PHASE15_BRAINSTORM.md:6` — stale
  `v0.14.77` audit-version label (intentionally backed out from
  the CHANGELOG flow earlier in the cycle) replaced with date-only
  references for both the broad code/tracking audit and the
  narrow re-audit. The historical Round 6 FEEDBACK triage note at
  `PHASE15_FEEDBACK.md:1485` is left intact (FEEDBACK is
  append-only by convention — never delete or rewrite history;
  the new re-audit section explicitly documents the correction
  in-line at line 1612+).
- `multi-agent/project_context/HANDOFF.md` and `TASK.md` —
  cold-start orientation lines updated from "brainstorming-stage
  closeout prep" to "brainstorming stage CLOSED at v0.14.76.4 /
  awaiting SPEC engineering kickoff prompt." Reminder of the
  rename decisions inherited by SPEC engineering surfaced
  prominently in HANDOFF (last_active_stage, max_rcn_stage,
  summit_rcn, --odw-* group).

**Files NOT archived at this transition (per the lifecycle rule).**

Per `AGENT_CONVENTIONS.md § Future-phase planning surfaces`,
BRAINSTORM and FEEDBACK both stay LIVE during SPEC engineering —
they archive together when SPEC engineering closes. Nothing
additional archives at this brainstorming-stage closeout. The SOUP
file was already archived at v0.14.76.2 (transfer closeout) to
`multi-agent/plans/archived/20260428-PHASE15_HMM_SOUP.md`.

**Workflow-doc convention work bundled into this cycle but
captured in earlier closeouts.**

- v0.14.76.3 (DEVLOG) — PDS-v2 stale-CHANGELOG-reference repair +
  AGENT_CONVENTIONS Form A2 codification + LOG-cadence-vs-commit-cadence
  canonical distinction. Triggered mid-cycle when Codex wrote a
  CHANGELOG entry for what should have been an intermediate audit
  pass.
- v0.14.76.1 (DEVLOG) — type-prefixed identifier system
  (`SOUP<N>.<idx>` / `BRAIN<N>.<idx>` / `SPEC<N>.<idx>`) — Phase
  15 was the first cycle to use it end-to-end.
- AGENT_CONVENTIONS additions during this cycle (already
  fossilized via earlier intermediate commits): ID-context-in-chat
  rule (§ Cross-referencing); KNOWN_ISSUES migration-with-provenance
  rule (§ KNOWN_ISSUES.md); LOG-cadence-vs-commit-cadence sub-section
  + Three forms (A / A2 / B) commit-form documentation (§ Git
  commit convention).

**Why this entry warrants DEVLOG (not CHANGELOG).** Brainstorming-stage
closeouts are dev-system planning work — zero product code touched
this cycle. Per `AGENT_CONVENTIONS.md` routing rule, the entry
goes in DEVLOG.

**Verification posture.** `make test` not run (zero product code
touched). No auditor pass needed for the closeout itself — Codex
already verified the brainstorming stage at the narrow re-audit
(verdict SUBSTANTIVELY CLEAN). Closeout fix is a single
1-line bookkeeping edit + project-context refresh.

**Next step.** SPEC engineering kickoff per `multi-agent/workflows/phase-development-system_PDS-v2.md`
§ "Typical form of the opening SPEC engineering prompt". Continue
Q&A in the same `PHASE15_FEEDBACK.md` (do not start a new file).
Use `SPEC15.<idx>` IDs. Group the 34 BRAIN entries into 5–10
substantive priorities per the v2 substantive-priorities rule;
each priority carries `Source:` / `Covers:` fields citing its BRAIN
IDs. The user will provide the opening prompt.

**Authors:** John M. Urban, Claude Code 2.1.112 (claude-opus-4-7 ; Effort: High), Codex (gpt-5-codex ; Reasoning: Extra High)

**Scope:** `multi-agent/plans/PHASE15_BRAINSTORM.md` (34-BRAIN-entry brainstorming-stage final form; lifecycle-position line bookkeeping fix at closeout dropping the stale `v0.14.77` audit-version label); `multi-agent/plans/PHASE15_FEEDBACK.md` (Round 3 aggregation + 8 Codex findings + 2 user-prompted findings; Round 4–6 substantive design follow-ups; Bayesian ODW System addenda 1/2/3; Codex narrow re-audit appended); `multi-agent/project_context/DECISIONS.md` (2026-04-28 entry); `multi-agent/plans/next/APS_SOUP.md` + `multi-agent/plans/next/UNIFIED-RCN_SOUP.md` (mid-cycle SOUP creation); `multi-agent/tracking/KNOWN_ISSUES.md` ([ISSUE:2026-04-22:1] migration); `multi-agent/project_context/HANDOFF.md` + `multi-agent/project_context/TASK.md` (closeout refresh).

---

## [v0.14.76.3] — 2026-04-28 EDT

### Workflow-doc fixes: PDS-v2 stale CHANGELOG-references repair + AGENT_CONVENTIONS Form A2 (intermediate iterative commit) codification + LOG-cadence-vs-commit-cadence canonical distinction

Dev-system workflow-document fixes prompted by an audit-loop bug
that surfaced during the Phase 15 brainstorming-stage Codex audit
(2026-04-28). Codex wrote a CHANGELOG entry mid-cycle for what
should have been an intermediate audit pass — the cause was
identified as stale prompt language in
`multi-agent/workflows/phase-development-system_PDS-v2.md` that
implied agents should write CHANGELOG entries during stage
iterations. This entry repairs those prompts and codifies the
intended LOG-cadence-vs-commit-cadence convention more
prominently across both `phase-development-system_PDS-v2.md` and
`AGENT_CONVENTIONS.md`. The pattern was already in active use
(Phase 15 soup-to-brainstorm Round 1 / Round 2.5 / Round 4 /
Round 5 commits use it) — this entry just makes the convention
discoverable to future agents.

**PDS-v2 stale-reference repair (13 lines fixed in
`multi-agent/workflows/phase-development-system_PDS-v2.md`):**

- 10 occurrences of *"After finishing up, before writing your
  CHANGELOG entry:"* (in the brainstorming-stage opening prompt,
  brainstorming-stage Agent 2 prompt, returning brainstorming
  Agent 1 prompt, returning Agent 2+ brainstorming prompt,
  opening SPEC engineering Agent 1 prompt, subsequent SPEC
  Agent 2 prompt, subsequent SPEC Agent 1 prompt, additional SPEC
  Agent 2 rounds prompt) → replaced with *"After finishing up:"*.
  Preserves the read-agent-file + read-AGENT_CONVENTIONS bullet
  recommendations underneath; drops the wrong implication that an
  entry should be written during iteration.
- 2 occurrences of *"Your contribution should be recorded as a
  CHANGELOG entry as well, and you will possibly want to update
  TASK.md and/or HANDOFF.md when you are done."* (in the Agent 2
  brainstorming-audit prompt and the returning Agent 2+
  brainstorming-audit prompt) → replaced with explicit
  no-LOG-during-iteration rule + intermediate-commit-IS-welcome
  rule + scratch-file convention pointer + orchestrator-decides
  framing.
- Line 198 (v2-rule canonical statement *"One CHANGELOG entry per
  cycle closeout"*) → replaced with *"One CHANGELOG/DEVLOG entry
  per cycle closeout"* + routing-rule clarification + explicit
  *"No CHANGELOG/DEVLOG entry is written during stage iteration —
  only at cycle closeouts"* + *"Intermediate git commits ARE
  welcome (and encouraged) at logical checkpoints during
  iteration — the rule is about LOG-entry cadence, not commit
  cadence"* + scratch-file pattern pointer.
- Line 315 (*"Before inserting into CHANGELOG.md:"*) → replaced
  with *"Before inserting into CHANGELOG.md or DEVLOG.md (only at
  cycle closeout — not during stage iteration):"* + added a
  fourth bullet pointing at AGENT_CONVENTIONS routing rule.

**AGENT_CONVENTIONS § Git commit convention additions (87 lines
added across `multi-agent/AGENT_CONVENTIONS.md`):**

- New canonical sub-section *"LOG-entry cadence vs commit cadence
  (canonical distinction)"* — explicit rule that the two cadences
  are independent. Stage iterations don't write LOG entries;
  intermediate commits ARE encouraged. Cross-references PDS-v2's
  matching language.
- *"Two forms"* sub-section heading expanded to *"Three forms"*
  with new **Form A2: Intermediate iterative commit (scratch-file
  body, NO LOG entry)** added between the existing Form A (terse
  mid-cycle `-m`) and Form B (closeout with LOG fossilization).
  Form A2 uses `changelog-entry-<topic>.txt` repo-root scratch
  file with verbose body, where `<topic>` is a descriptive
  non-version slug (NOT a `v0.X.YY[.Z]` version string — those
  are reserved for Form B). Three real-world examples included
  from the active Phase 15 brainstorming-stage commits.
- *"Use Form A vs Form A2"* discriminator added: Form A for
  mechanical audit-loop rounds (one-line summary suffices); Form
  A2 for substantive design discussions / multi-decision rounds
  where the body captures content that helps future agents
  understand what changed and why.
- *"Rules"* bullet list at the bottom of § Git commit convention
  updated from *"Mid-cycle commits use `-m`...; closeout commits
  use `--file=`..."* to a three-form description naming all of A,
  A2, B explicitly + the version-string vs descriptive-slug rule
  for the scratch filenames.

**Why this matters / why it warrants a DEVLOG entry:**

The intermediate-commit pattern (Form A2) has been in active use
across the Phase 15 brainstorming-stage cycle since 2026-04-27,
but was undocumented — it wasn't in the convention; it was just a
pattern emerging from session-level practice. When Codex
attempted the brainstorming-stage audit, the stale PDS-v2 prompt
language led it to follow Form B (CHANGELOG-fossilizing closeout
commit) for what should have been Form A2 (intermediate audit
commit, no LOG insertion). The user is reverting Codex's
mid-cycle CHANGELOG entry separately. This entry codifies the
convention so future audit agents follow Form A2 by default.

**Cross-pipeline consistency check:** the matching language now
appears in both `multi-agent/AGENT_CONVENTIONS.md` § Git commit
convention AND
`multi-agent/workflows/phase-development-system_PDS-v2.md` § How
the Phase Development System Works (canonical statement) + § Two
opening/returning brainstorming prompts (per-prompt audit
guidance). Cross-references between the two locations point each
other at the right home.

**Verification posture:** no code changes; planning/workflow
files only. `make test` not run (zero product code touched). No
auditor pass — this is a small dev-system workflow-doc fix.

**Authors:** John M. Urban, Claude Code 2.1.112 (claude-opus-4-7 ; Effort: High)

**Scope:** `multi-agent/workflows/phase-development-system_PDS-v2.md` (13 stale-CHANGELOG-reference lines repaired across the v2-rule statement, opening + returning brainstorming prompts, opening + subsequent SPEC engineering prompts, and Agent 2 brainstorming-audit prompts); `multi-agent/AGENT_CONVENTIONS.md` (new § "LOG-entry cadence vs commit cadence (canonical distinction)" sub-section; "Two forms" → "Three forms" with new Form A2 codifying the intermediate-iterative-commit pattern; rules-list updated to reference all three forms).

---

## [v0.14.76.2] — 2026-04-28 EDT

### Phase 15 (HMM completeness + enrichment) soup-to-brainstorm transfer — CLOSED (consolidated cycle entry)

First end-to-end execution of the SOUP → BRAINSTORM → SPEC
lifecycle codified at v0.14.75.3 / v0.14.75.4 and the first cycle
to use the SOUP/BRAIN/SPEC type-prefixed identifier system codified
at v0.14.76.1. Cycle ran across multiple rounds (Round 1, Round 1
follow-up, Round 2, Round 2.5) with one auditor pass + one narrow
re-audit, and produced the 29-BRAIN-entry baseline that opens the
Phase 15 brainstorming stage.

**Cycle artifacts at closeout.**

- `multi-agent/plans/PHASE15_BRAINSTORM.md` — 29 BRAIN entries
  (BRAIN15.1 through BRAIN15.29) covering all 21 SOUP IDs.
- `multi-agent/plans/PHASE15_FEEDBACK.md` — full Q&A trail across
  Round 1 (Q1–Q11), Round 1 follow-up (Q12–Q14), Round 2 (post
  user answers + multi-round APS design discussion), Round 2.5
  (narrow source-line provenance fix per Codex auditor finding),
  the auditor section, and this closeout entry.
- `multi-agent/plans/archived/20260428-PHASE15_HMM_SOUP.md` — the
  archived SOUP (orchestrator-archived from
  `plans/PHASE15_HMM_SOUP.md` via `git mv` per the
  archive-operations-orchestrator-only convention).
- `multi-agent/project_context/DECISIONS.md` 2026-04-28 entry —
  APS feature mode candidates accepted/rejected catalog +
  RCN-vs-excess design decision (canonical home for the rejection
  rationales of Fourier descriptors / curvature features /
  multi-family BIC vectors / `peak_rcn`-as-distinct-feature).
- `multi-agent/plans/next/APS_SOUP.md` — new SOUP authored
  mid-cycle to capture deferred IBM APS items + amplicon-level
  clustering future-phase candidate + rejected-features
  provenance section.
- `multi-agent/plans/next/UNIFIED-RCN_SOUP.md` — new SOUP authored
  mid-cycle to preserve the long-horizon
  unified-per-sample-smoothed-RCN-intermediates idea (Q10 user
  direction).

**Round-by-round summary.**

- **Round 1 (2026-04-27, Agent 1):** SOUP promoted from
  `plans/next/` to `plans/PHASE15_HMM_SOUP.md`; one-time SOUP
  labeling pass added 21 SOUP15.X markers; PHASE15_BRAINSTORM
  authored with 24 initial BRAIN entries (21 transcribed-from-SOUP
  with 3 splits and 3 consolidations, all flagged for user
  approval; plus 1 user-introduced entry BRAIN15.24
  `peak_rcn_stage` audit added at user direction during the same
  session); FEEDBACK Round 1 entry seeded with 11 open questions
  (Q1–Q11) covering documentation tensions, audit timing,
  IBM-C5/C6/C7B promotion decisions, sourcing notes, and SOUP-body
  cruft handling.
- **Round 1 follow-up (2026-04-28, Agent 1):** user answered Q1–Q11
  + approved all 6 splits/consolidations; BRAIN15.10 / 15.12 /
  15.17 / 15.19 / 15.22 / 15.24 updated per Q1 / Q2 / Q3 / Q5 / Q6
  / Q8 answers; BRAIN15.15 marked RESOLVED per Q7 investigation
  (14-S20 already verified Growth+RMS share dedup, HMM uses its
  own step-8 merging by design); BRAIN15.18 expanded with
  multistage-unification (4-mode flag) + state-path evolution
  checks per Q9 user content; new BRAIN15.25 (HMM 2-pass
  background-region-anchored chromosome-specific renormalization)
  added per Q4; new BRAIN15.26 (chrom-median default switch,
  late-Phase-15 work) added per Q9 + [ISSUE:2026-04-21:1]; new
  SOUP files `APS_SOUP.md` + `UNIFIED-RCN_SOUP.md` created in
  `plans/next/` per Q10 / Q11 directions; FEEDBACK Round 1
  follow-up section appended with 3 new questions (Q12–Q14).
- **Round 2 (2026-04-28, Agent 1):** Q12 mixed answer (IBM-C6B
  ignored; IBM-C6C promoted with substantial new design); Q13
  transitioned into a multi-round APS feature-mode design
  discussion that produced BRAIN15.27 (composite multi-feature APS
  clustering modes with three-layer PCA composition: Layer 1
  amplicon-level shape PCA, Layer 2 sample-level shape PCA with
  raw/reduced variants, Layer 3 matrix-level PCA pre-projection;
  seven new `--aps-feature` modes; four new flags) and BRAIN15.28
  (`summit_excess` → `summit_rcn` rename + RCN-as-default
  migration; preserves excess+floor as opt-in for sample-level APS
  aggregation per user reasoning). Q14 confirmed SOUP filenames.
  New DECISIONS.md 2026-04-28 entry appended capturing the
  rejection rationales for Fourier descriptors / curvature
  features / multi-family BIC vectors / `peak_rcn` as a distinct
  feature, plus the RCN-vs-excess design decision. APS_SOUP.md
  IBM-C7B marked RESOLVED-by-existing-and-by-BRAIN15.27 (the
  existing `--aps-feature shape` v0.3.32 implementation already
  fulfills the documented IBM-C7B scope; BRAIN15.27 covers the
  richer interpretations). New BRAIN15.29 added: one-pass
  summit → timing → updated-summit → updated-timing convergence
  per Q12's IBM-C6C promotion (NOT EM-iterative per user explicit
  direction; sibling to but distinct from IBM-C6E which stays
  deferred). APS_SOUP.md IBM-C6C marked PROMOTED-to-BRAIN15.29.
  FEEDBACK Round 2 follow-up section appended.
- **Round 2.5 (2026-04-28, Agent 1):** Codex (Agent 2 auditor)
  declared the transfer OPEN with one narrow finding: BRAIN15.27
  and BRAIN15.28 had `**Source:**` lines saying "Round 1
  follow-up" but those entries were authored in Round 2. Agent 1
  applied the narrow fix (3 substitutions across the 2 entries);
  BRAIN15.29 was already correctly labeled. FEEDBACK Round 2.5
  follow-up section appended documenting the fix. Re-audit by
  Codex declared CLEAN, clearing the transfer for closeout.

**Splits and consolidations across SOUP → BRAINSTORM (all
user-approved Round 1).**

- 3 splits: SOUP15.5 → BRAIN15.5 + BRAIN15.8 + BRAIN15.13;
  SOUP15.8 (######OTHER cluster) → BRAIN15.12 + BRAIN15.17 +
  BRAIN15.23; SOUP15.18 (CORRECTNESS BUG) → BRAIN15.9 +
  BRAIN15.10.
- 3 consolidations: SOUP15.7 + SOUP15.12 → BRAIN15.7 (the SOUP
  itself flags the merge); SOUP15.17 + SOUP15.18 → BRAIN15.9;
  SOUP15.10 + SOUP15.8 item 2 → BRAIN15.12.

**6 user-introduced no-SOUP-source BRAIN entries** (each logged in
FEEDBACK with the user-directive provenance, per the
AGENT_CONVENTIONS rule allowing such entries when added at
BRAINSTORM authoring time at user direction):

- BRAIN15.24 — `peak_rcn_stage` term audit (Round 1, 2026-04-27).
- BRAIN15.25 — HMM 2-pass background-region-anchored
  chromosome-specific renormalization (Round 1 follow-up,
  2026-04-28 Q4).
- BRAIN15.26 — HMM `--norm-mode` chrom-median default switch
  (Round 1 follow-up, 2026-04-28 Q9).
- BRAIN15.27 — Composite multi-feature APS clustering modes with
  three-layer PCA composition (Round 2, 2026-04-28 Q13 multi-round
  design).
- BRAIN15.28 — `summit_excess` → `summit_rcn` rename + RCN
  migration (Round 2, 2026-04-28 same discussion).
- BRAIN15.29 — Summit-then-timing-then-updated-summit-then-updated-timing
  convergence one-pass; IBM-C6C promotion (Round 2, 2026-04-28
  Q12).

**2 RESOLVED markers** (closeout markers in BRAINSTORM, no Phase
15 implementation work):

- BRAIN15.13 — `--hmm-smooth-halfwidth` APS split (resolved in
  Phase 14 Supplemental cycle 14S.23 as
  `--hmm-aps-smooth-halfwidth`; tracked as
  [ISSUE:2026-04-19:3]).
- BRAIN15.15 — Cross-pipeline dedup code unification (resolved by
  Phase 14 Supplemental cycle 14-S20 — Growth + RMS share
  `dedup_calls_by_peak_proximity()` in
  `onionskin_core/rcn_mean_shift_helpers.py`; HMM uses its own
  step-8 merging by design).

**Mechanical coverage at closeout (auditor-verified CLEAN).**

- All 21 SOUP IDs (SOUP15.1 through SOUP15.21) cited in at least
  one BRAIN entry's `Source:` field — mechanically auditable via
  `grep -nE "SOUP15\.[0-9]+" multi-agent/plans/PHASE15_BRAINSTORM.md`.
- 29 BRAIN IDs sequential 1–29, no gaps.
- All open FEEDBACK questions (Q1–Q11 + Q12–Q14) resolved.
- SOUP body integrity preserved — only the one-time SOUP labeling
  pass touched the body.
- Source-line provenance labels correct after Round 2.5 fix
  (BRAIN15.27 / BRAIN15.28 / BRAIN15.29 all read "Round 2
  follow-up").

**Carry-over to Phase 15 brainstorming stage.**

The brainstorming-stage agent (next prompt in PDS-v2) inherits a
substantive set of activities flagged during the transfer:

- BRAIN15.24 `peak_rcn_stage` term audit — natural first activity
  per Q2 user direction. Audit primary entry point: search code
  for similar concepts under different names (the user flagged
  the concept may already be in Growth or RMS code) +
  pin down user-recalled tentative semantics + propagate any
  rename across planning surfaces and code.
- BRAIN15.18 sub-priorities (d) + (e) enrichment from
  `tracking/BRAINSTORM.md`, `tracking/KNOWN_ISSUES.md`,
  `tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` per Q9 / Q10 user
  direction.
- BRAIN15.22 IBM-C5 cross-pipeline dynamic origin-detection design
  sketch per Q3 staged plan (design → SPEC ID → implementation,
  all within Phase 15).
- General BRAINSTORM enrichment from tracking files per Q10 user
  note.

**Optional follow-on cleanup work** (not blocking brainstorming
stage, deferred from this closeout per the narrow-fix-only
discipline):

- Update `multi-agent/tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md`
  IBM-C7B entry to mark RESOLVED with closeout pointer to ROADMAP
  Priority 6.1 + BRAIN15.27.
- Update `multi-agent/tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md`
  IBM-C6C entry to mark RESOLVED-via-promotion-to-BRAIN15.29.
- Update `ROADMAP.md` Priority 6.3 from `✗ DEFERRED` to `✓ DONE`
  per the DECISIONS.md 2026-04-28 entry (subsumed by 6.1's
  `--aps-feature shape` implementation).
- Update `ROADMAP.md` Priority 5.0.2 from `✗ DEFERRED` to
  `✓ PROMOTED-to-Phase-15` (or mark resolved when BRAIN15.29
  lands).

**Verification posture.** No code changes in this cycle; planning
files only. `make test` not run (zero product code touched).
Auditor (Codex, Round 1 audit + narrow re-audit) declared CLEAN
at the re-audit pass.

**Authors:** John M. Urban (orchestrator), Claude Code 2.1.112 (claude-opus-4-7 ; Effort: High) (Agent 1), Codex GPT-5.5 (Reasoning: High) (Agent 2 auditor)

**Scope:** New artifacts: `multi-agent/plans/PHASE15_BRAINSTORM.md`, `multi-agent/plans/PHASE15_FEEDBACK.md`, `multi-agent/plans/archived/20260428-PHASE15_HMM_SOUP.md` (orchestrator-archived from `plans/PHASE15_HMM_SOUP.md`), `multi-agent/plans/next/APS_SOUP.md`, `multi-agent/plans/next/UNIFIED-RCN_SOUP.md`. Modified: `multi-agent/project_context/DECISIONS.md` (new 2026-04-28 entry), `multi-agent/project_context/HANDOFF.md` (cold-start orientation refreshed; new Last Action entry), `multi-agent/project_context/TASK.md` (immediate-next-tasks rotated).

---

## [v0.14.76.1] — 2026-04-27 EDT

### ID system formalization across phase-development stages: `SOUP<N>.<idx>` / `BRAIN<N>.<idx>` / `SPEC<N>.<idx>` with Phase-15 cutover

Codifies a unified, type-prefixed ID scheme covering the full
SOUP → BRAINSTORM → SPEC lifecycle so provenance citations are
mechanically auditable from any context (CHANGELOG, ROADMAP,
KNOWN_ISSUES, FEEDBACK, AUDIT_LOG, prose) without surrounding-text
disambiguation. The scheme is opt-in starting **Phase 15 onward**;
Phases 12 / 13 / 14 / 14 Supplemental keep their pre-cutover formats
(bare `<N>.<idx>`, `14-S<idx>`) under the archived-content-is-immutable
rule. No code touched; no product behavior change.

**Format chosen.**
- **SOUP IDs** — `SOUP<N>.<idx>` in `plans/PHASE<N>_<THEME>_SOUP.md`.
  Assigned exactly once at SOUP-promotion time (the file move from
  `plans/next/` to `plans/`) as a one-time labeling pass; SOUP body is
  read-only after.
- **BRAIN IDs** — `BRAIN<N>.<idx>` in `plans/PHASE<N>_BRAINSTORM.md`.
  Assigned during BRAINSTORM authoring; each entry's body carries a
  `Source:` field citing the SOUP IDs it covers (plus any
  tier-1/tier-2 cross-citations from `tracking/BRAINSTORM.md`,
  `tracking/KNOWN_ISSUES.md`, etc.). May be reordered/renumbered
  during BRAINSTORM iteration; freezes when SPEC engineering begins.
- **SPEC IDs** — `SPEC<N>.<idx>` in `plans/PHASE<N>_SPEC.md` (Option B
  style: drop the "Priority" word; the typed ID itself signals the
  entry is a SPEC priority). Assigned during SPEC engineering; each
  entry cites the BRAIN IDs it covers in `Source:` / `Covers:`.
  Reassignable during SPEC engineering, frozen at SPEC-lock.

**Why type-prefixed (not bare).** `SOUP15.3` is globally unique and
greppable in isolation; bare `15.3` would collide with version
strings, line numbers, dates, and other numeric noise. The prefix +
phase number combination makes file disambiguation unnecessary in
cross-references. Treats IDs as **provenance citations**, not stable
handles that follow an idea unchanged through every stage — each
stage's IDs reflect that stage's organizational logic; cross-stage
references read the source artifact's native ID.

**Splits and consolidations are first-class.** A SPEC entry may cite
multiple BRAIN IDs in its `Source:` field (consolidation:
`Source: BRAIN15.5, BRAIN15.12, BRAIN15.17`); a BRAIN ID may be cited
by multiple SPEC entries (split: each SPEC entry cites the BRAIN ID
individually, with an optional annotation in the BRAINSTORM entry
noting the split for archival traceability). Same pattern for
SOUP → BRAINSTORM, with the addition that SOUP-stage splits require
user approval via FEEDBACK (since splitting a single SOUP entry into
multiple BRAIN entries is interpretive, not just labeling).

**CHANGELOG and ROADMAP citation conventions updated.** CHANGELOG
`**Roadmap:**` lines and ROADMAP retrospective entry headings should
reference Phase 15+ priorities by their typed-prefix ID — e.g.,
*"Phase 15 — HMM completeness work covering SPEC15.3, SPEC15.7,
SPEC15.12"* and *"## SPEC15.1 — title ✓ DONE (v0.X.YY)"*.

**Files changed.**
- `multi-agent/AGENT_CONVENTIONS.md` — canonical home of the rules.
  - § Future-phase planning surfaces / SOUP → BRAINSTORM → SPEC
    lifecycle: extended steps 1–7 to add the live-in-`plans/`
    intermediate stage for SOUP and the SOUP-labeling pass at
    promotion.
  - § Identifier system across phase-development stages: rewritten
    end-to-end with the new formats, the Phase-15 cutover note, the
    splits/consolidations conventions, the mechanical auditability
    greps, and the CHANGELOG/ROADMAP citation rules. FEEDBACK and
    AUDIT_LOG cross-reference behavior also documented (they don't
    introduce their own provenance IDs; they cite the artifact-native
    IDs they discuss).
- `multi-agent/workflows/phase-development-system_PDS-v2.md` —
  operational mirror of the rules.
  - § Four important files: rewritten to reflect the live-in-`plans/`
    SOUP stage, the SOUP labeling pass, BRAIN-ID assignment + Source:
    convention during BRAINSTORM authoring, and `SPEC<N>.<idx>`
    heading style.
  - **soup-to-brainstorm prompt** (Agent 1, ~line 377): updated
    throughout to use the new ID formats; explicit step-by-step
    instructions for the SOUP labeling pass + BRAIN ID + Source:
    citations + FEEDBACK seeding + iterative-pass pattern.
  - **soup-to-brainstorm AUDITOR prompt** (Agent 2, ~line 462):
    drafted with 10 verification criteria covering SOUP-ID coverage,
    BRAIN-ID assignment, Source provenance, scope drift, splits,
    SOUP body integrity (post-labeling), and formatting/convention
    compliance. Output structure matches the user's prompt-style
    template.
  - **soup-to-brainstorm FOLLOW-UP prompt** (Agent 1 next round,
    ~line 531): drafted for next-pass after FEEDBACK responses
    (addresses user answers to prior open questions, addresses
    auditor findings, updates BRAINSTORM + FEEDBACK accordingly,
    SOUP body remains read-only).
  - **soup-to-brainstorm CLOSEOUT prompt** (~line 588): drafted for
    archive (SOUP `git mv` to `plans/archived/`) + DEVLOG-routed
    commit, with a baseline "BRAINSTORM contains N BRAIN IDs covering
    M SOUP IDs plus K cross-tier-sourced ideas" recorded for future
    BRAINSTORM-stage tracking.

**Pending, not in this entry.** None of the new ID formats are
applied to any actual artifact yet — `HMM_SOUP.md` does NOT yet have
`SOUP15.<idx>` tags (it's still a `plans/next/` SOUP, not promoted).
The labeling pass happens at promotion-time per the lifecycle, so the
first concrete application of these IDs will land in the Phase 15
SOUP-promotion + BRAINSTORM-authoring session that the user has
queued as the next-major-step.

**Authors:** John M. Urban, Claude Code 2.1.112 (claude-opus-4-7 ; Effort: High)

**Scope:** `multi-agent/AGENT_CONVENTIONS.md` (§ Future-phase
planning surfaces lifecycle steps + § Identifier system across
phase-development stages — full rewrite); `multi-agent/workflows/
phase-development-system_PDS-v2.md` (§ Four important files +
soup-to-brainstorm transfer prompt + new auditor / follow-up /
closeout prompts).

---

## [v0.14.75.4] — 2026-04-27 EDT

### Wave F remediation: SOUP self-reference rule reconciliation + downstream-reference cleanup + ROADMAP preamble retrospective rewrite (per Codex GPT-5.5 verification of v0.14.75.3)

Single-commit follow-up to the DEVPLAN-ROADMAP-AND-NEXT-DIRECTORY-SYSTEM
cycle closed at v0.14.75.3, addressing three real findings surfaced by
independent verification from Codex GPT-5.5 (Reasoning: Extra High). All
changes are dev-system or doc-only; the lone touch to product code
(`onionskin.py:1188`) is a help-string text swap with zero behavioral
effect (`PHASE16_BRAINSTORM.md` → `SUMMIT_SOUP.md` in a user-facing help
string that points at the planning surface). `make test` 128/128 PASS at
closeout.

**Codex Finding 1 — Reconcile the codified rule with implementation
practice.** v0.14.75.3 codified "SOUP body MUST NEVER mention a phase
number" across `AGENT_CONVENTIONS.md § Future-phase planning surfaces`,
`workflows/phase-development-system_PDS-v2.md`, and
`workflows/spec_plan_three_role_audit_loop-v2.md` Template D wrap-up
audit scan. But the implementation honored a relaxed spirit ("no
SELF-references; cross-references to actually-closed phases ARE allowed
as historical anchors") that the codified rule didn't reflect. Fixed in
all three files: rule now distinguishes forbidden self-references
(SOUP names its own past/current/future phase identity, e.g., "this is
the Phase X plan", "Phase X follow-up", "(was PHASE15_BRAINSTORM.md)")
from allowed cross-references (e.g., "see Phase 14 Supplemental cycle
14S.1a v0.14.68"). Wrap-up audit Template D scan instructions updated to
require triage of grep hits rather than blanket flagging. Plus the
SOUP files' bodies brought into compliance with the relaxed rule:

- `multi-agent/plans/next/SUMMIT_SOUP.md`: trimmed the rename-history
  paragraph that named `PHASE-TBD_SUMMIT_BRAINSTORM.md` and
  `PHASE16_BRAINSTORM.md` (replaced with a one-line "rename history is
  preserved by `git log --follow`" note per the convention).
- `multi-agent/plans/next/FWD-ARCH_SOUP.md`: same treatment for the
  paragraph that named `PHASE16_BRAINSTORM.md` and
  `PHASE17_BRAINSTORM.md` plus narrated the file's own self-described-as-
  Phase-11 history. Maintenance-rule wording also updated to allow
  cross-references explicitly.
- `multi-agent/plans/next/HMM_SOUP.md`: maintenance-rule wording updated
  to allow cross-references explicitly (no body content needed
  trimming — HMM_SOUP didn't carry a self-ref rename narrative).

**Codex Finding 2 — Update downstream stale references.**
v0.14.75.3 renamed the three SOUP files but didn't enumerate references
to the old `PHASE15/16/17_BRAINSTORM.md` filenames in other parts of the
codebase. Codex's verification found 7 active references that were now
broken; all updated to point at the new SOUP filenames:

- `onionskin.py:1188` — help string for `--bootstrap-summits` (or
  similar) that pointed at `PHASE16_BRAINSTORM.md` for the planned
  Summit Methodology pass; now points at
  `multi-agent/plans/next/SUMMIT_SOUP.md`.
- `multi-agent/tracking/KNOWN_ISSUES.md` lines 307 + 375 — both
  references to `PHASE15_BRAINSTORM.md` updated to `HMM_SOUP.md`.
- `multi-agent/tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` lines 145,
  196, 222, 432 — references to `PHASE15_BRAINSTORM.md` /
  `PHASE17_BRAINSTORM.md` updated to `HMM_SOUP.md` / `FWD-ARCH_SOUP.md`,
  plus stale narratives like `(which despite the filename describes
  itself as "Phase 12" internally)` and `(filename says 17, body says
  Phase 14)` removed since the SOUP body cleanup made those framings
  obsolete.
- `multi-agent/project_context/HANDOFF.md` and
  `multi-agent/project_context/TASK.md` — top entries updated to
  reflect DEVPLAN-ROADMAP-AND-NEXT-DIRECTORY-SYSTEM is CLOSED at
  v0.14.75.4, the phase-system overhaul is fully complete, and the
  next viable work is small dev-system cleanup (entries 8 + 8a stale
  pointers, ONIONSKIN_FULL_HANDOFF.md header refresh) or Phase 15 SPEC
  engineering under the new SOUP→BRAINSTORM→SPEC lifecycle.
- Historical references in HANDOFF "Last Action" archive narrative,
  CHANGELOG, DEVLOG, AUDIT_HISTORY, the AUDIT_LOG cycle history, and
  the DEVPLAN-ROADMAP-AND-NEXT-DIRECTORY-SYSTEM file itself (which
  describes the rename plan) are intentionally NOT updated — they
  preserve the provenance of what files were called when those entries
  were written. The workflow file's didactic example use of
  `(was PHASE15_BRAINSTORM.md)` as a forbidden-pattern illustration is
  also preserved.

**Codex Finding 3 — ROADMAP.md preamble retrospective rewrite.**
v0.14.75.3's Wave E added retrospective entries for Phase 12 / 13 / 14
/ 14 Supplemental at the bottom of `ROADMAP.md` in the new
light-reading + completion-markers pattern, but didn't update the file's
top preamble which still described ROADMAP as "the planned evolution of
onionskin beyond the current Phase 3.23+ development state" + had a
"Current state snapshot (Phase 5 entry)" + a "next work should
emphasize" section — all forward-looking framing that contradicted the
new role. Fixed: opening paragraphs now state ROADMAP is retrospective +
light-reading; cross-reference to `AGENT_CONVENTIONS.md § ROADMAP.md`
for canonical role/update-trigger rules; cross-reference to
`multi-agent/plans/next/` for candidate future phases. The "How to read
this file" section retained but trimmed; the "Current state snapshot"
and "next work should emphasize" sections removed in favor of a
"Historical preamble (preserved)" note explaining that earlier phase
entries (Phase 4 through Phase 11) reflect the older forward-looking
convention while Phase 12+ entries follow the new light-reading
pattern.

**Codex Finding 4 (post-Wave-F first pass) — `DECISIONS.md` references
to renamed-away `PHASE16_BRAINSTORM.md`.** Codex's re-verification
caught two stale references at `multi-agent/project_context/DECISIONS.md`
lines 753 and 784 that v0.14.75.3 missed (the original DEVPLAN's Finding 2
inventory hadn't enumerated `DECISIONS.md`). Both updated to point at
`multi-agent/plans/next/SUMMIT_SOUP.md` with parenthetical "(formerly
the Phase 16 brainstorm at the time of decision)" preserving the
historical context that the decision was originally framed around the
Phase 16 numbering. The 2026-04-23 decision header that read "Phase 16
— Summit Methodology as a dedicated future phase" updated to
"Summit Methodology as a dedicated future phase (originally framed as
'Phase 16')" so the historical framing is preserved without a
forward-looking commitment to a specific phase number.

**Codex Finding 5 (pre-existing typo, also caught) —
AGENT_CONVENTIONS.md:267 said the Final Overseer wrap-up was "Role 3 /
Template I" but Template I is actually the Strategist / Plan-of-Attack
generator (verified at `spec_plan_three_role_audit_loop-v2.md:1621`).
The Final Overseer Wrap-Up Audit is Template D (verified at
`spec_plan_three_role_audit_loop-v2.md:1416`). Fixed: AGENT_CONVENTIONS
line 267 now says "Role 3 / Template D". This typo predates this
DEVPLAN cycle (it landed at v0.14.73.1 when the post-Final-Overseer
archive timing rule was first codified) but was caught here. While
fixing, also corrected three Template-I→D references in the DEVPLAN file
itself (lines 290, 388, 540) that prescribed the `next/`-directory
wrap-up scan as living in Template I; the actual implementation in Wave
C correctly added the scan to Template D, so the implementation was
right but the DEVPLAN's prescription text was wrong. Now both match.

**Verification posture:** the DEVPLAN's § 9 acceptance criteria (as
relaxed in v0.14.75.3's self-audit) all still hold. `make test` 128/128
PASS. Codex GPT-5.5 (Reasoning: Extra High) was the verifier on both
Findings 1+2+3 (initial Wave F response) and Findings 4+5 (post-Wave-F
re-pass) and is co-authored on this entry for that work. User signalled
intent to have GPT re-verify the additional Wave F fixes before
declaring full closure.

**Self-audit observations from Wave F (recorded for the next
verifier):**

1. The grep `grep -nE "Phase [0-9]" multi-agent/plans/next/*_SOUP.md`
   still returns hits — but per the now-codified relaxed rule, those
   hits are all legitimate cross-references to actually-closed phases
   (Phase 11, 13, 14, 14 Supplemental). A clean self-reference scan
   would need a smarter pattern (e.g., looking for "this is the Phase
   X plan", "Phase X will/may decide", "(was PHASE<N>_*.md)", etc.).
   The Template D wrap-up audit instructions already prescribe triage;
   this is a known limitation of the simple grep.
2. The DEVPLAN file at
   `multi-agent/plans/next/DEVPLAN-ROADMAP-AND-NEXT-DIRECTORY-SYSTEM.md`
   itself contains many `PHASE15/16/17_BRAINSTORM.md` references —
   these are intentional content of the rename plan and stay. The
   DEVPLAN file's own status could be moved (it's now executed; could
   be archived or marked CLOSED in the file's status frontmatter), but
   that's out of scope for Wave F. Worth a small future cleanup or fold
   into the "Phase 14 Supplemental archive" step that's still pending.
3. Workflow v1 was left untouched per v0.14.75.3 § 4.5 decision; still
   the right call, no change needed.

**Authors:** John M. Urban, Claude Code 2.1.112 (claude-opus-4-7 ; Effort: High), Codex GPT-5.5 (Reasoning: Extra High)

**Scope:** `multi-agent/AGENT_CONVENTIONS.md` (§ Future-phase planning
surfaces rule reconciliation: self-ref vs. cross-ref distinction +
audit-time check guidance; § Active phase SPEC/PLAN files
Template-I→D typo fix at line 267);
`multi-agent/workflows/phase-development-system_PDS-v2.md` (mirror rule
reconciliation in § Four important files);
`multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md` (Template D
wrap-up audit scan instructions updated for triage); SOUP file body
cleanup (`HMM_SOUP.md`, `SUMMIT_SOUP.md`, `FWD-ARCH_SOUP.md` —
maintenance rule + rename-history paragraphs); downstream reference
updates (`onionskin.py` line 1188 help string,
`multi-agent/tracking/KNOWN_ISSUES.md` ×2 references,
`multi-agent/tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` ×4 references
+ stale narrative cleanup, `multi-agent/project_context/DECISIONS.md`
×2 references + 1 header update for the Summit Methodology decision);
HANDOFF.md + TASK.md top-section refresh; `ROADMAP.md` preamble rewrite
(lines 1–50 replaced with retrospective framing); plus 3 Template-I→D
prescription-text corrections in the DEVPLAN file itself.

---

## [v0.14.75.3] — 2026-04-27 EDT

### DEVPLAN-ROADMAP-AND-NEXT-DIRECTORY-SYSTEM cycle CLOSED — ROADMAP retrospective role + plans/next/ + SOUP convention + DEVPLAN naming convention codified

Implements the 7-deliverable dev-system maintenance plan stored at
`multi-agent/plans/next/DEVPLAN-ROADMAP-AND-NEXT-DIRECTORY-SYSTEM.md`
(drafted at v0.14.75.2). Pure dev-system work, no product code touched.
Single cycle, executed in 5 waves (A through E) plus a Wave A.1 fix-up
correction. `make test` 128/128 PASS at closeout.

**Wave A (commit `ebc7adf`) + Wave A.1 (commit `81ab15d`) — SOUP file
renames + body cleanup.** Three files in `multi-agent/plans/next/`:

- `PHASE15_BRAINSTORM.md` → `HMM_SOUP.md` (title cleanup; ~25 `Phase 15` /
  `Phase 12` self-references replaced with theme-based language; legitimate
  cross-references to closed Phase 11/13/14/14-Supplemental preserved as
  historical anchors).
- `PHASE16_BRAINSTORM.md` → `SUMMIT_SOUP.md` (title + opening cleanup;
  `Phase 15` cross-refs to HMM SOUP retargeted to "the HMM-completeness
  phase").
- `PHASE17_BRAINSTORM.md` → `FWD-ARCH_SOUP.md` (title + opening cleanup;
  `Priority 14.1` heading renamed to `Candidate priority` since the file
  never represented an actual Phase 14 plan; new `## Audit findings
  (2026-04-27)` section appended at bottom per user direction, recording
  the file's still-meaningful slice + recommendations to defer promotion
  or fold into HMM-completeness BRAINSTORM at promotion time).

The Wave A.1 fix-up corrected a `git mv` ordering issue: cleanup edits
made BEFORE the rename were captured by `git mv` as 100%-similarity
renames (the modifications stayed in the working tree as unstaged), so
Wave A's commit message accurately described the intended scope but the
actual committed content was still the original framing. Wave A.1
caught up the body content. Root-cause-mitigation for future agents:
do content edits AFTER `git mv`, not before.

**Wave B (commit `7814a63`) — `multi-agent/AGENT_CONVENTIONS.md`
additions:**

1. Rewrote `## ROADMAP.md` section to codify the **retrospective +
   light-reading + SPEC-formalization-trigger** role: ROADMAP is no
   longer a forward-looking compass; the phase system itself IS the
   actual roadmap; CHANGELOG `**Roadmap:**` lines point at phase plans,
   NOT at sections in `ROADMAP.md`; new entries added once a SPEC
   formalizes (not at brainstorm/SOUP); `✓ DONE (v0.x.xx)` markers
   accumulate as priorities close; ROADMAP MUST NOT speculate about
   un-formalized phases.
2. Added new `## Future-phase planning surfaces` section codifying:
   `multi-agent/plans/next/` rules; the `<THEME>_SOUP.md` filename
   convention (vague-thematic prefix + `_SOUP` suffix; never
   scope-anchoring; never numbered); the no-internal-phase-numbers rule;
   audit-time check; `PHASES.txt` orchestrator-scratchpad exception; the
   full SOUP → BRAINSTORM → SPEC lifecycle (incl. the
   BRAINSTORM-from-SOUP step explicitly triggering the first
   `PHASE<N>_FEEDBACK.md` entry); the post-promotion archive-filename
   pattern `<YYYYMMDD>-PHASE<N>_<THEME>_SOUP.md` (carries both new-phase
   identity + old-theme identity); the four-tier hierarchy that
   distinguishes `tracking/BRAINSTORM` (tier 1) from `tracking/KNOWN_ISSUES`
   (tier 2) from per-phase `PHASE<N>_BRAINSTORM` (tier 3) from SOUP
   (tier 4).
3. Added new `## DEVPLAN naming convention` section codifying
   `DEVPLAN-<topic>.md` / `DEVLOGPLAN-<topic>.md` for dev-system work
   plans (no phase numbers; NOT product phases; usually impromptu;
   optional pre-implementation plans).
4. Updated existing `## BRAINSTORM.md` section header with disambiguation
   ("tracking-dir reservoir") + cross-reference to the new four-tier
   hierarchy. Same pattern in the duplicate `## BRAINSTORM.md
   (conceptual scratchpad)` section deeper in the file.
5. Updated `## KNOWN_ISSUES.md` location bullet with the four-tier
   cross-reference.
6. Updated planning-content routing table: added SOUP-stage row,
   per-phase BRAINSTORM-staging row, DEVPLAN row; revised ROADMAP row
   to retrospective framing.
7. Updated `## Cross-referencing` section to clarify CHANGELOG
   `**Roadmap:**` lines point at phase plans, not at `ROADMAP.md`
   sections.

**Wave C (commit `d1cf3b6`) — workflow file updates (the two cassettes):**

- `multi-agent/workflows/phase-development-system_PDS-v2.md` (primary):
  expanded SOUP file description in `### Four important files` with the
  full convention; `BRAINSTORM-from-SOUP triggers first FEEDBACK entry`
  rule explicitly named; new `**Promotion (SOUP → BRAINSTORM):**`
  paragraph with pre-promotion phase-number scan + archive-filename
  pattern; cross-reference to `AGENT_CONVENTIONS § Future-phase planning
  surfaces` as canonical home; fixed typo
  (`multi-agent/plans/<THEME>_SOUP.md` → `multi-agent/plans/next/<THEME>_SOUP.md`).
- `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md`
  (secondary): `## Required File Updates — v2 § Optional additional
  artifacts when relevant` ROADMAP bullet rewritten to retrospective
  framing; `### Template D — Role 3 Final Wrap-Up Audit` gains a
  cross-cycle drift check that scans `multi-agent/plans/next/*_SOUP.md`
  bodies for phase-number mentions (with `PHASES.txt` excluded) plus a
  ROADMAP-update-at-wrap-up reminder.
- `multi-agent/workflows/orchestrator.spec_plan_three_role_audit_loop-v2.md`:
  no ROADMAP mentions found; no edits needed (verified by grep).
- Workflow v1: left untouched per the DEVPLAN's § 4.5 decision (deprecated;
  preserved as historical sibling).

**Wave D (commit `28696bc`) — agent-file tier-list updates** propagated
identically to all four agent files (`CLAUDE.md`, `AGENTS.md`, `GEMINI.md`,
`.github/copilot-instructions.md`) per the agent-file consistency rule:
entry 3 (ROADMAP) rewritten to retrospective; entry 9 (BRAINSTORM)
updated with four-tier cross-reference; new entry 9a added for
`multi-agent/plans/next/`; § Planning model section rewritten to
phase-system-as-roadmap framing first.

**Wave E (commit `e841765`) — `ROADMAP.md` retrospective backfill:**
light-reading entries for Phase 12 (v0.12.25), Phase 13 (v0.13.80;
final wrap-up v0.13.81), Phase 14 (v0.14.46), Phase 14 Supplemental
(v0.14.75) added in the new pattern: closeout version + date,
archive-path pointers, one-paragraph what-it-was-about overview, bulleted
priority list with `✓ DONE (v0.x.xx)` markers per priority. Sources:
the four archived SPEC files (grepped for canonical priority lists per
the DEVPLAN's § 4.7 implementer guidance).

**Self-audit findings (recorded for next-agent visibility):**

1. The DEVPLAN's § 9 acceptance criterion 1 (`grep -nE "Phase [0-9]"
   multi-agent/plans/next/*_SOUP.md` returns 0 hits) is **too strict** as
   written. SOUP files legitimately mention CROSS-references to actual
   closed phases (Phase 11, 13, 14, 14 Supplemental) as historical anchors;
   only SELF-references (mentions of the file's own past or future phase
   number) need to be removed. My implementation honors the SPIRIT of the
   criterion (no self-references) but not the letter. The criterion in
   the DEVPLAN should be relaxed to "no self-references" if the DEVPLAN
   is ever re-run or referenced. Per-file remaining-hit counts for
   verification: HMM_SOUP.md ~14 hits (all cross-refs), SUMMIT_SOUP.md
   ~9 hits (all cross-refs), FWD-ARCH_SOUP.md ~12 hits (all cross-refs
   plus narrative about the file's own rename history in the new SOUP
   framing — those are legitimate historical record).
2. Wave A's commit message was accurate-by-intent but inaccurate-by-actual
   because of the `git mv` ordering issue; Wave A.1 corrected this. The
   git-discipline implication ("do content edits AFTER `git mv`, not
   before") may be worth folding into AGENT_CONVENTIONS or the workflow
   files in a future cycle. Deferred per user direction.
3. Out-of-scope-but-noted: the four agent files' entries 8 + 8a still
   point at `multi-agent/plans/PHASE14_SUPPLEMENTAL-SPEC.md` /
   `-AUDIT_LOG.md` (LIVE paths) but those files are now archived. This
   was supposed to be part of the manual archive operation (workflow v2
   § Step 8) at commit `67cd2a3` and didn't happen. The agent files
   currently have stale active-phase pointers. Updating them was OUT OF
   SCOPE for this DEVPLAN (the DEVPLAN focused on the ROADMAP-and-
   plans/next/ system, not active-phase pointer hygiene). Worth
   addressing in a separate small DEVLOG-routed cleanup whenever
   convenient.
4. The ONIONSKIN_FULL_HANDOFF.md version-header refresh (Phase 14
   Supplemental wrap-up audit Finding E) remains a separate maintenance
   item, also out-of-scope for this DEVPLAN.

**Verification (recommended) by independent agent:** see the Codex
GPT-5.5 (Reasoning: Extra High) verification prompt emitted to chat at
DEVPLAN-cycle closeout. The prompt is self-contained and exercises every
acceptance criterion against the live state.

**Authors:** John M. Urban, Claude Code 2.1.112 (claude-opus-4-7 ; Effort: High)

**Scope:** `multi-agent/AGENT_CONVENTIONS.md` (3 new sections + ROADMAP
section rewrite + 5 cross-reference / disambiguation edits);
`multi-agent/workflows/phase-development-system_PDS-v2.md` (SOUP convention
+ promotion mechanic in § Four important files); `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md`
(ROADMAP-update bullet + Template D wrap-up audit additions);
`CLAUDE.md` + `AGENTS.md` + `GEMINI.md` + `.github/copilot-instructions.md`
(tier-list entries 3 + 9 + new 9a + Planning model rewrite);
`ROADMAP.md` (retrospective backfill for Phase 12, 13, 14, 14 Supplemental);
`multi-agent/plans/next/PHASE15_BRAINSTORM.md` → `HMM_SOUP.md` (rename +
body cleanup); `multi-agent/plans/next/PHASE16_BRAINSTORM.md` →
`SUMMIT_SOUP.md` (rename + body cleanup); `multi-agent/plans/next/PHASE17_BRAINSTORM.md`
→ `FWD-ARCH_SOUP.md` (rename + body cleanup + bottom audit-findings
section).

---

## [v0.14.75.2] — 2026-04-26 ~23:30 EDT

### DEVPLAN drafted and committed: ROADMAP role + `plans/next/` directory mechanics + SOUP convention + DEVPLAN naming convention

Captures a multi-part dev-system maintenance plan that emerged during the
Phase 14 Supplemental Final Overseer wrap-up second-pass audit (Finding D).
The plan itself is a planning surface, not implementation work; this entry
records its existence and the user clarifications baked into it so the
agreed scope survives context loss.

**File:** `multi-agent/plans/next/DEVPLAN-ROADMAP-AND-NEXT-DIRECTORY-SYSTEM.md`
(originally drafted in commit `0124a26`; subsequent commit-pending edits
resolve three open questions per user clarifications received during the
drafting session and add a deliverable for the DEVPLAN naming convention
itself).

**Scope of the queued plan (7 deliverables):**

1. `AGENT_CONVENTIONS.md § ROADMAP.md` — codify the role shift: ROADMAP
   is now retrospective + light-reading; phase entries get added once a
   SPEC is fully formalized (not at brainstorm stage); completion
   markers (`✓ DONE (v0.x.xx)` / `◑ PARTIAL (v0.x.xx)`) accumulate as
   priorities close.
2. `AGENT_CONVENTIONS.md § Future-phase planning surfaces` — new section
   codifying `plans/next/` rules + the SOUP convention + the SOUP →
   BRAINSTORM → SPEC promotion mechanic. Includes the
   "no internal phase numbers in SOUP-stage files" rule (only the
   filename carries the phase number, if any). Notes `PHASES.txt` as the
   orchestrator's free-form scratchpad (per user clarification:
   intentionally flat; not authoritative; not auto-maintained by agents).
3. `AGENT_CONVENTIONS.md § DEVPLAN naming convention` — short new section
   codifying `DEVPLAN-<topic>.md` (or `DEVLOGPLAN-<topic>.md`) as the
   pattern for dev-system work plans. Such files: never carry phase
   numbers; are NOT product phases; are usually impromptu; may live in
   `plans/next/` if a formal pre-implementation plan is written, or
   may fire directly as DEVLOG-routed work without any pre-plan. THIS
   DEVPLAN file is the canonical example.
4. Rename + clean — `plans/next/PHASE15_BRAINSTORM.md` /
   `PHASE16_BRAINSTORM.md` / `PHASE17_BRAINSTORM.md` rename to
   vague-thematic `_SOUP.md` filenames per user resolution of § 7
   question 1 (Option B with vague names like `HMM_SOUP.md` rather
   than `HMM_COMPLETENESS_SOUP.md`; the `_SOUP` suffix is retained
   per § 7 question 1a to keep the lifecycle marker explicit).
   Content cleanup to remove internal phase-number mentions;
   `PHASE17_BRAINSTORM.md` needs heavier work because its body still
   says "Phase 11 Implementation Specification".
5. Workflow v2 + orchestrator-v2 updates — drop forward-looking ROADMAP
   prescriptions; replace with retrospective SPEC-formalization-trigger
   prescriptions; add a `next/`-directory phase-number scan to wrap-up
   audit Templates.
6. Agent-file tier-list updates — `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`,
   `.github/copilot-instructions.md` each get an updated ROADMAP entry
   reflecting the retrospective role + a new `plans/next/` entry.
7. `ROADMAP.md` backfill — light-reading retrospective entries for
   Phase 12, Phase 13, Phase 14, Phase 14 Supplemental in the new
   pattern.

**Sequencing note:** the DEVPLAN explicitly is NOT an in-flight cycle.
It lives in `plans/next/` as a stored plan-of-attack to be executed
whenever the orchestrator schedules it (most natural: after Phase 14
Supplemental closes — which has now happened — and before or alongside
Phase 15 cold-start). Pure dev-system; no product code touched.

**Provenance:** drafted at HEAD `eae7be5` (Final Overseer wrap-up second
pass); first committed at `0124a26`; finalized by this entry after the
user's two-pass clarifications on ROADMAP role + the DEVPLAN naming
convention were folded in.

**Authors:** John M. Urban, Claude Code 2.1.112 (claude-opus-4-7 ; Effort: Extra High)

**Scope:** `multi-agent/plans/next/DEVPLAN-ROADMAP-AND-NEXT-DIRECTORY-SYSTEM.md` (new file in `plans/next/` queue)

---

## [v0.14.75.1] — 2026-04-26 ~22:45 EDT

### Manual updates to audit / implement / re-audit lanaguage
- further clarification sections in "Roles" sections
- echoes of "further clarifications" in the "Template" sections
- this is an attempt to recapture some of the more conversational and collaborative tone we originally had when it was just conversation, not totally formalized

**Authors:** John M. Urban, Claude Code 2.1.112 (claude-opus-4-7 ; Effort: High)

**Scope:** workflow-v2

---

## [v0.14.73.1] — 2026-04-26 EDT

### Phase-archive sequencing rule + git-discipline principles — codified into v2 workflow and AGENT_CONVENTIONS

Cycle 14S.5a's Role 1 initial audit (run earlier in this session) prescribed three
implementation steps, the third of which moved the five Phase 14 Supplemental
planning files into `multi-agent/plans/archived/` with a `20260426-` prefix.
Codex (Role 2) faithfully executed all three. The archive was out-of-scope per
the v2 STRATEGY's "after Final Overseer wrap-up + any post-wrap-up cycles close"
sequencing — Phase 14 Supplemental had not yet hit Final Overseer (Role 3
Template I); cycle 14S.5a was the last *execution* cycle, not the last event of
the phase. Post-archive, the planning files would have been at archived paths
while Final Overseer still needed to write into the live AUDIT_LOG. The error
was caught in the same session, the archive parts were surgically reverted via
`git reset --soft 32b1fb8` + targeted file edits (preserving Codex's keeper
changes — tracking-dir reorg + AGENT_CONVENTIONS CLI additions — at commit
459ed47), and a backup branch `backup/14s5a-codex-state` preserves the original
two-commit Codex implementation at 766fcd1 for recoverability.

This entry codifies the rules that should have prevented the audit from
prescribing the archive in the first place, plus three git-discipline principles
that surfaced as adjacent root causes during the recovery.

**`multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md`** — additions:

- New `### Step 8 — Phase Archive (separate from cycle structure)` subsection
  added to § Standard Execution Loop after Step 7 (Post-Wrap-Up Remediation
  Loop). States explicitly that the archive operation (`git mv` of
  `PHASE<N>_SPEC.md`, `PHASE<N>_AUDIT_LOG.md`, `PHASE<N>_STRATEGY.md`,
  `PHASE<N>_FEEDBACK.md`, and any chronological backup files into
  `multi-agent/plans/archived/` with `YYYYMMDD-` prefix) is **NOT a cycle
  deliverable** — it is performed by the orchestrator outside the cycle
  structure, after (1) the last execution cycle closes, (2) Final Overseer
  wrap-up audit closes, AND (3) any post-wrap-up triage/remediation cycles
  close. Includes a "Why archive-last matters" explanation (Final Overseer +
  remediation cycles write into the live AUDIT_LOG; archiving early forces those
  writes into archived paths and breaks the immutability rule). Distinguishes
  cycle-scoped closeout work (reference propagation, tracking-dir moves, CLI
  conventions codification — all in-scope) from the archive operation itself
  (out-of-scope for any execution cycle, including the last). Closes with an
  audit-time check warning.

- Template A (Role 1 Initial Audit, line ~1262) and Template C (Role 1
  Re-Audit, line ~1376): each gains an "Archive-timing check (do not skip)"
  paragraph instructing R1 to STOP and re-check § Step 8 if the audit's repair
  instructions propose `git mv` of the SPEC/AUDIT_LOG/STRATEGY/FEEDBACK files
  into `archived/`. Template C adds the additional Re-Audit-specific
  instruction: if Role 2 already performed an unauthorized archive, treat the
  archive as out-of-scope, instruct the orchestrator to revert it (preserving
  any in-scope keeper changes), and continue the re-audit on the post-revert
  state.

- Template I (Final Overseer / Role 3 wrap-up audit) trigger paragraph
  (line ~1717) replaced with explicit "AFTER the last execution cycle closes,
  not before" wording + a clear note that the phase archive is a SEPARATE
  post-Final-Overseer operation (cross-references Step 8). Removes the prior
  ambiguity that allowed the archive to be conflated with cycle-internal
  closeout work.

- § Convention for agent-produced git commit blocks (line ~1130) gains a new
  **Git-discipline principles** subsection with three numbered rules:
  1. **No interactive git in copy-paste blocks — ever.** Forbidden flags listed
     explicitly: `git add -p`, `git add -i`, `git rebase -i`, bare `git commit`
     (which opens `$EDITOR`), `git mergetool`, any `--edit`. Prescribes explicit
     per-file `git add <path>`. Cycle 14S.5a Codex's `git add -p` skipped four
     agent files + `TASK.md` and was a major contributor to the recovery cost.
  2. **One concept per commit.** Each commit block represents a single coherent
     change: one cycle round, one role's output, or one named operation.
     Bundling implementation + closeout doc sweep + phase archive into one
     commit makes surgical reverts (like 14S.5a's archive-only revert) painful;
     splitting into per-concept commits keeps reverts cheap.
  3. **Implementer-splits-stages pattern.** Role 2 (and Role 3 wrap-up
     implementers) executing multi-stage instructions SHOULD emit one commit
     block per stage rather than a single end-of-round mega-block. The 14S.5a
     case study is canonical — the archive stage was reverted while keeping
     prior stages intact precisely because (counterfactually) per-stage commits
     would have made it a one-line `git revert`.

**`multi-agent/AGENT_CONVENTIONS.md`** — § Active phase SPEC/PLAN files: the
prior single-line "When a phase is finished, move the SPEC/PLAN file..." rule
expanded with explicit timing language. "Phase finished" now means **after the
Final Overseer wrap-up audit (Role 3 / Template I) has closed AND any
post-wrap-up remediation cycles triggered by that audit have themselves
closed.** A phase is NOT "finished" merely because its last execution cycle
closed; the Final Overseer pass must complete first. New bullet codifies that
under v2 the archive is performed by the orchestrator as a separate
post-Final-Overseer step (cross-references workflow § Step 8). Mechanics
bullet preserved. Immutability bullet preserved.

**Provenance.** The behavioral rule itself was already implicit across multiple
files (STRATEGY-template "after Final Overseer" wording; AGENT_CONVENTIONS'
"Once archived... immutable historical provenance"; the Phase 10 archive that
correctly followed this pattern). What this entry adds is *explicit, indexed,
referenceable* codification — every place an agent might naturally land
(workflow § Step 8, Template A, Template C, Template I, AGENT_CONVENTIONS
§ Active phase SPEC/PLAN files) now contains the rule, so a future Role 1 audit
cannot repeat the 14S.5a mistake by accident. The git-discipline principles
formalize three previously-unwritten norms that the user has consistently
applied but were not codified anywhere agents would discover them.

**Authors:** John M. Urban, Claude Code 2.1.112 (claude-opus-4-7 ; Effort: Extra High)

**Scope:** workflow-v2 (new § Step 8 — Phase Archive, archive-timing checks in
Templates A and C, tightened Template I trigger paragraph, new
Git-discipline-principles subsection in § Convention for agent-produced git
commit blocks); AGENT_CONVENTIONS.md (§ Active phase SPEC/PLAN files —
"phase finished" timing clarified, post-Final-Overseer archive operation
codified)

---

## [v0.14.70.4] — 2026-04-25 EDT

### Copy-paste output formatting rule — fenced code blocks required for all copy-paste content

Agents on this project routinely emit handoff prompts and git command blocks that the
user must copy and paste into a new chat or terminal. The desired pattern is a fenced
code block (triple backticks), which renders with a copy button in VS Code and preserves
indentation and structure on paste. The failure mode is inline prose or inline backtick
code, which has no copy button and loses formatting.

This entry codifies a new rule requiring fenced code blocks for all copy-paste content:

**`AGENT_CONVENTIONS.md`** — new section `## Copy-paste output formatting (required)`
added between the "File placement" section and "User Interaction Workflow". Covers:
- Handoff launcher prompts → `\`\`\`text ... \`\`\``
- Git command blocks → `\`\`\` ... \`\`\`` or `\`\`\`bash ... \`\`\``
- Any other multi-line copy-paste content → appropriate fence
- Explicit rationale: VS Code renders fenced blocks with a copy button; inline text
  does not. This is non-negotiable for all agents on this project.

**`AGENT_CONVENTIONS.md` `## Git commit convention` `### Rules`** — new bullet at top:
"Output the block in a fenced code block (triple backticks) — never as inline prose or
inline code. See 'Copy-paste output formatting' section above."

**`multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md`
`Convention for agent-produced handoff prompts`** — added rule 5: launcher prompt body
must be in a ` ```text ``` ` fenced code block; advisory sits outside; references
AGENT_CONVENTIONS copy-paste rule.

**`multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md`
`Convention for agent-produced git commit blocks`** — added:
- Bold requirement immediately after the "agent only outputs the commands" paragraph:
  the block MUST be a fenced code block; inline-formatted commands lose the copy button.
- New bullet in "Rules common to both forms": output as fenced code block; cross-reference
  to AGENT_CONVENTIONS copy-paste section.

**Authors:** John M. Urban, Claude Code 2.1.112 (claude-opus-4-7 ; Effort: High)

**Scope:** AGENT_CONVENTIONS.md (new copy-paste formatting section + git rules bullet),
workflow-v2 (handoff prompt rule 5, git commit block fencing requirement + rules bullet)

---


## [v0.14.70.3] — 2026-04-25 ~13:00 EDT

### Phase 14 Supplemental cycle 14S.2a CLOSED — Phase 2 audit-only deliverables

Cycle 14S.2a (index 3) closed via Template H lightweight closeout. Skip-reaudit
recommended by Role 2 and accepted by orchestrator; cycle additionally uses the
**Deferred-R3 rule** per STRATEGY (R3 scope absorbed into 14S.3a R3). No code
changes — this cycle produced audit-only deliverables across `multi-agent/tracking/`
and `multi-agent/plans/next/PHASE15_BRAINSTORM.md`. Routes to DEVLOG by AGENT_CONVENTIONS
routing rule (audit/tracking artifacts, no code or user-facing docs).

**5 priorities CLOSED v0.14.70.3** in `PHASE14_SUPPLEMENTAL-SPEC.md` priority status
table:

- **14-S12** — RCN-profile flag survey (deliverable produced 2026-04-23 in
  `PHASE14_SUPPLEMENTAL-FEEDBACK.md`; downstream actions absorbed into 14-S27 v0.14.70).
- **14-S25** — `-rcn-` prefix convention audit (deliverable produced 2026-04-23 in
  FEEDBACK; help-string direction routes to 14-S8; convention codification rides
  14-S16 closeout).
- **14-S14** — Internal Stage-1/Stage-2 detection-pass terminology audit. Created
  `multi-agent/tracking/STAGE-1-2-DETECTION-PASS-TERMINOLOGY-AUDIT.md` (16 KB) with
  three classified tables (detection-pass / biological-stage / ambiguous), critical
  invariant ("Biological-stage terminology is NEVER renamed"), and harvest
  methodology. Becomes the canonical decision surface for a future renaming phase.
- **14-S15** — HMM PuffStep synonym audit. Replaced the all-TBD stub at
  `multi-agent/plans/next/PHASE15_BRAINSTORM.md` lines 251–269 with a populated
  7-row table covering all canonical HMM flag pairs. Findings: 6 of 7 active
  synonyms still registered AND mentioned in canonical help text (consistent with
  Q15 direction); 1 pair (`--hmm-emodel`) already retired via `_DEPRECATED_FLAGS`
  pre-parse gate during cycle 14S.1a (v0.14.68); no missing-mention or
  stale-alias cases. Phase 15 decision surface articulated (keep all / retire some
  / help-text rewrites only).
- **14-S21** — `--hmm-0-based-statepath` impact audit. Appended a "Test enumeration
  addendum" + "Live-code verification 2026-04-25" stamp to `PHASE15_BRAINSTORM.md`
  § 14-S21 audit. Enumeration found that no existing test in `tests/` directly
  asserts on raw HMM state-path integer values — `stage_state_trail` assertions in
  `tests/test_pipeline.py` and `tests/test_rcn_final_classification.py` reference
  RCN-pipeline classification state (per-developmental-stage trail), not HMM
  state-path values, so they are unaffected by the future flag. Phase 15 will need
  three NEW tests (emit-boundary shift verification; adaptive defaults; user-set
  override semantics) rather than gating existing assertions.

**Cycle structure:** 5 priorities, R1 → R2 → Template H (R1 closeout). 2 priorities
(14-S12, 14-S25) had deliverables already produced 2026-04-23 during SPEC engineering;
their cycle work was confirmation that downstream routing remained correct. 3 priorities
(14-S14, 14-S15, 14-S21) had Role 1 pre-fill audit data into the AUDIT_LOG, which Role 2
formatted/integrated into the destination files plus added supplemental sites the
initial harvest had underrepresented (notably `tests/test_pipeline.py:764–765`
detection-pass tokens for 14-S14 and `tests/test_rcn_final_classification.py:85,96,150,196`
state-trail assertions for 14-S21).

**R3 scope deferred to cycle 14S.3a R3** per the STRATEGY Deferred-R3 rule. When
14S.3a's R3 (Role 1 re-audit closeout) runs, it must additionally verify the three
14S.2a deliverables exist with stated content and that 14S.3a's broad help-string
rewrites have not introduced semantic conflicts with the 14-S15 PuffStep findings or
the 14-S25 `-rcn-` audit recommendations.

**Files touched:**

- `multi-agent/tracking/STAGE-1-2-DETECTION-PASS-TERMINOLOGY-AUDIT.md` (NEW, ~16 KB)
- `multi-agent/plans/next/PHASE15_BRAINSTORM.md` (14-S15 stub replaced; 14-S21 addendum
  + verification stamp appended)
- `multi-agent/plans/PHASE14_SUPPLEMENTAL-SPEC.md` (priority status table: 5 priorities
  → CLOSED v0.14.70.3)
- `multi-agent/plans/PHASE14_SUPPLEMENTAL-AUDIT_LOG.md` (cycle 14S.2a section: Role 1
  initial audit, Role 2 implementation report, Role 1 Template H closeout; cycle
  heading CLOSED)
- `multi-agent/project_context/HANDOFF.md`, `multi-agent/project_context/TASK.md`
  (state advanced to next cycle)

No code changes. No tests run (no runtime files were touched). The 14-S14 audit's
recommendation that biological-stage terminology must NEVER be renamed is captured
explicitly in the tracking file's "Critical invariant" section and is re-stated here
to ensure any future renaming-phase agent reads it before touching tokens.

**Authors:** John M. Urban, Claude Code 2.1.109 (claude-sonnet-4-6 ; Effort: Extra High)

**Scope:** PHASE14_SUPPLEMENTAL cycle 14S.2a closeout, multi-agent/tracking/ (new file), multi-agent/plans/next/PHASE15_BRAINSTORM.md (14-S15 + 14-S21 sections), PHASE14_SUPPLEMENTAL-SPEC.md (priority status table), PHASE14_SUPPLEMENTAL-AUDIT_LOG.md (cycle section closed)

---

## [v0.14.70.2] — 2026-04-25 EDT

### Authorship corrections + new rule: agents cannot self-determine runtime toggle

Dev-system session. No code changes.

**1. Authorship corrections (in-place edits to source-of-truth files).** When asked what model and effort was running, an agent reported "Claude Sonnet 4.6 with Effort: Extra High" — a guess based on the system prompt's model identifier and the agent's sense of what felt apt for the work. The user's GUI showed Opus with Effort = High. The system-prompt claim was wrong, the Effort claim was a fabrication. Two prior DEVLOG entry authorship lines were corrected in-place:

- `v0.14.69` — was `Claude Code 2.1.109 (claude-sonnet-4-6)` (no toggle), now `Claude Code 2.1.109 (claude-opus-4-7 ; Effort: High)`.
- `v0.14.70.1` — was `Claude Code 2.1.109 (claude-sonnet-4-6 ; Effort: Extra High)`, now `Claude Code 2.1.109 (claude-opus-4-7 ; Effort: High)`.

Original git commit messages for both entries (and the `v0.14.69-followup1` commit) were left as-is per user direction — the cost of amending those commits is higher than the value of correcting the historical commit-log text. The CHANGELOG/DEVLOG file (the source of truth for the human-readable record) is now correct.

**2. New rule in AGENT_CONVENTIONS § Authorship: agents cannot self-determine the runtime toggle.** Codified what the failure mode revealed:

- The Effort / Reasoning / Thinking toggle is set in the host UI and is **not exposed to the agent** through any system prompt, environment variable, or tool.
- Agents must ASK the user for the toggle value before writing any authorship line that includes a toggle clause. No inference, no "what feels apt," no copying from prior sessions.
- The same caution applies to the model field when the system prompt's claim could conflict with the GUI selection. If there's any chance of disagreement between sources, ask the user.
- "Known" in the existing "be generous with detail" rule means "told to you by the user this session" or "visible in a tool result," NOT "guessed."

The new rule is positioned at the top of the Rules list (right under "Source of truth") so it's read before any of the formatting guidance.

**Files touched:**
- `multi-agent/DEVLOG.md` (authorship lines for v0.14.69 and v0.14.70.1 corrected in-place; this v0.14.70.2 entry added).
- `multi-agent/AGENT_CONVENTIONS.md` (§ Authorship Rules: new "Agents cannot self-determine the runtime toggle" rule + worked-example failure mode + tightened "be generous with detail" wording).

No code changes. No tests run.

**Authors:** John M. Urban, Claude Code 2.1.109 (claude-opus-4-7 ; Effort: High)

**Scope:** AGENT_CONVENTIONS.md (§ Authorship Rules), DEVLOG.md (in-place authorship corrections, v0.14.69 and v0.14.70.1)

---

## [v0.14.70.1] — 2026-04-25 EDT

### Split version streams for CHANGELOG / DEVLOG + appending strategy codified

Dev-system session. No code changes. **First DEVLOG entry under the new split-stream versioning scheme** (anchor v0.14.70 + per-anchor counter `.1`).

**1. Split version streams (prospective from this entry).** CHANGELOG and DEVLOG previously shared a single `v0.X.YY` counter. This produced gaps in both files (CHANGELOG jumping v0.14.63 → v0.14.68 because v0.14.64–v0.14.67 were DEVLOG entries) and made both files harder to read linearly. Replaced with a parallel scheme:

- `CHANGELOG.md` keeps `v0.X.YY` (e.g., `v0.14.70`); `YY` increments only when product code is touched. `X` bumps when product code is first touched in a new ROADMAP phase (no longer at planning start — dev-system planning for an upcoming phase anchors to the prior phase's last CHANGELOG version until the first product entry of the new phase is cut).
- `DEVLOG.md` uses `v0.X.YY.Z` (e.g., `v0.14.70.1`). The `v0.X.YY` portion is the **anchor** — the latest CHANGELOG version at the time the DEVLOG entry was written. `.Z` is a per-anchor counter starting at 1, incrementing by 1 per DEVLOG entry while the anchor remains the same. `.Z` resets to 1 each time a new CHANGELOG version is cut.

Total ordering across the two files is preserved by the anchor: a DEVLOG v0.14.70.3 unambiguously came after CHANGELOG v0.14.70 and before whatever the next CHANGELOG cut is. No gaps in either file going forward. `onionskin --version` continues to read CHANGELOG only; the DEVLOG `.Z` ticker has no effect on user-facing version reporting.

Cross-references between the two files use the full version string. Component count disambiguates the file: 3-component → CHANGELOG, 4-component → DEVLOG.

**Migration boundary.** Entries through CHANGELOG v0.14.70 and DEVLOG v0.14.69 used the old shared-stream scheme. The new scheme starts at this entry (v0.14.70.1). Historical entries are not retroactively renumbered. Both preambles now explain the migration.

**2. Appending-to-existing-entry strategy codified (Q1).** The git commit convention previously covered mid-cycle and closeout commits but not the case where the user asks an agent to append content to an already-written CHANGELOG/DEVLOG entry. Added a subsection to AGENT_CONVENTIONS § Git commit convention and a parallel paragraph to the workflow file's "Convention for agent-produced git commit blocks" section:

- *Scenario (a) — original commit already happened*: edit the entry in-place; create a NEW scratch file `changelog-entry-<version>-followup<N>.txt` (`-followup1`, `-followup2`, etc.) containing just the appended subsection content; emit the standard three-command closeout block using the follow-up file. The original commit fossilizes the original entry; the follow-up commit fossilizes the appended content. Together they tell the story; the integrated entry lives in the CHANGELOG/DEVLOG file itself.
- *Scenario (b) — original commit not yet happened*: edit the entry in-place; rewrite the existing `changelog-entry-<version>.txt` with the full updated entry. The user's existing closeout commands still work unchanged.
- The agent picks the scenario via `git log --oneline` or by asking the user.

This entry is a real-world example of the codified rule — it was the question that prompted the v0.14.69 follow-up commit (`changelog-entry-v0.14.69-followup1.txt`) earlier today.

**3. Placeholder cleanup.** Replaced the older `<X.YY.ZZ>` placeholder in scratch-file names (which incorrectly implied a fixed 3-numeric-component form) with `<version>`, defined as the full version string of the entry being committed (`v0.X.YY` for CHANGELOG entries, `v0.X.YY.Z` for DEVLOG entries under the new scheme). Updated in AGENT_CONVENTIONS and the workflow file.

**Files touched:**
- `CHANGELOG.md` (preamble: split-streams versioning paragraph + historical-entries note replacing the older "shared stream" paragraph)
- `multi-agent/DEVLOG.md` (preamble: Format and Versioning sections rewritten for split scheme; this entry header inaugurates the new scheme)
- `multi-agent/AGENT_CONVENTIONS.md` (§ CHANGELOG.md and DEVLOG.md: versioning rules rewritten; § Git commit convention: `<version>` placeholder cleanup + new "Appending to an already-written CHANGELOG/DEVLOG entry" subsection)
- `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md` (Convention for agent-produced git commit blocks: `<version>` placeholder cleanup + appending-to-existing-entry paragraph)

No code changes. No tests run.

**Authors:** John M. Urban, Claude Code 2.1.109 (claude-opus-4-7 ; Effort: High)

**Scope:** versioning scheme split (CHANGELOG/DEVLOG), AGENT_CONVENTIONS.md (§ CHANGELOG/DEVLOG + § Git commit convention), workflow-v2 (git commit convention), CHANGELOG/DEVLOG preambles

**Authorship correction (post-commit, recorded in v0.14.70.2):** the originally-committed authorship line for this entry read `claude-sonnet-4-6 ; Effort: Extra High`. That was wrong on both fields — the runtime model is Opus and the GUI-set Effort was High. Source-of-truth file corrected here in-place; original commit message left as-is. See v0.14.70.2 for the rule that prevents this failure mode going forward.

---

## [v0.14.69] — 2026-04-24 EDT

### Workflow v2 correctness fixes + effort/thinking-level guidance + session-cadence rewording + parallel toggle notation + per-role git commit convention

Dev-system session. No code changes.

**1. Template B conditional handoff emission (correctness fix).**
Template B previously told Role 2 to "Provide both handoff prompts (re-audit and skip-reaudit-accepted) so the orchestrator can pick" unconditionally. This caused Codex (Role 2, cycle 14S.1a) to emit both prompts even though it declared re-audit needed — and the orchestrator accidentally pasted the wrong one (skip-reaudit variant). Root cause: the dual-prompt rule should be conditional on Role 2's declaration.

Fix: Template B in `spec_plan_three_role_audit_loop-v2.md` and the corresponding launcher in `orchestrator.spec_plan_three_role_audit_loop-v2.md` now read:
- If skip-reaudit is recommended → emit both prompts (Template C + Template H); orchestrator picks.
- If re-audit is declared needed → emit only Template C. Orchestrator may override to skip, but that decision belongs to the orchestrator, not the implementer.

**2. Template B CHANGELOG gap fix (correctness fix).**
Template B said "Do NOT write to CHANGELOG.md in this round" unconditionally, but the "Who writes CHANGELOG" section already designated Role 2 as the closing agent when skip-reaudit is accepted. Result: when skip-reaudit was accepted, no agent ever wrote the CHANGELOG entry for that cycle.

Fix: the "Do NOT write CHANGELOG" instruction now has an explicit exception — if you are recommending skip-reaudit, write the cycle's CHANGELOG/DEVLOG entry now (you are the de facto closing agent if the skip is accepted). If re-audit is declared needed, do not write an entry; Role 3 writes it at closeout. Mirrored in the orchestrator launcher.

**3. Session-boundary cadence + natural-boundary section reword (AGENT_CONVENTIONS.md).**
The existing section advised agents to surface a "natural boundary" recommendation at a triggering boundary. Updated for v2 and for token-efficiency clarity:
- Added "Recommended cadence for the orchestration chat" guidance: one orchestration chat per cycle (not per phase or per role). Explains the cache-TTL rationale (5-min TTL → idle gaps between roles cause cache misses) and why per-cycle is the sweet spot.
- Updated triggering boundary "Priority closeout" (v1 language) to "Cycle closeout" (v2 language): a cycle is fully closed when R3 has confirmed or skip-reaudit is accepted by the orchestrator.
- Added cache-TTL note to the "Why this matters" paragraph.

**4. Claude effort levels, GPT reasoning levels, Gemini thinking budgets.**
The workflow system had no guidance on model performance settings. Added throughout:

*In `spec_plan_three_role_audit_loop-v2.md`:*
- Agent tier roster: added Effort levels to Claude models (Opus, Sonnet, Haiku) and thinking-budget notes to Gemini models.
- Baseline strategies: Claude agent specs now include Effort level; Gemini specs include thinking budget note.
- Advisory block "Assigned agent" format: now specifies "effort/reasoning/thinking level" as part of the agent spec to be copied from STRATEGY.md.
- STRATEGY.md cycle table format: agent cells now bundle the performance level with the agent name (e.g., `Claude Code — Sonnet (Effort: Extra High)`). Advisory blocks copy verbatim, so full assignment flows to every handoff automatically.
- New section "Performance tuning — effort, reasoning, and thinking levels": per-role effort table for Claude; GPT reasoning level convention; Gemini thinking budget guidance.

*In `orchestrator.spec_plan_three_role_audit_loop-v2.md`:*
- New end section "Agent selection and performance tuning": self-contained human-orchestrator reference covering all three agent families, tier tables with performance settings, baseline assignment tables, upgrade/downgrade rules, and the per-role Claude effort table. Also covers GPT reasoning level convention and Gemini thinking budget guidance.

Summary of per-role effort recommendations (Claude):

| Role | Task | Effort |
|---|---|---|
| R1 Auditor (Sonnet) | Single-surface audit | Extra High |
| R1 Auditor (Sonnet) | Paired/complex audit | Max |
| R2 Implementer (Sonnet) | Mechanical implementation | High (default) |
| R2 Implementer (Sonnet) | Complex cross-module repair | Extra High |
| R3 Re-auditor / Closeout (Sonnet) | Verify implementation | Extra High |
| R1/R3 Auditor (Opus) | Any audit | High |
| Final Overseer (Opus) | Phase closeout | Extra High |

**5. Parallel toggle notation across vendors (terminology cleanup).**
Earlier framing treated GPT reasoning level as "embedded in the model name" and Claude Effort as a separate toggle. That asymmetry was wrong — the model is just GPT-5.5; reasoning is a runtime toggle on it, the same as Effort on Claude or Thinking on Gemini. Cleaned up so all three vendors are described in parallel form `<harness> — <model> (<Toggle>: <value>)`:

- Claude (Claude Code, Github Copilot — Claude): **Effort** — Low | Medium | High (default) | Extra High | Max
- OpenAI (Codex, Github Copilot — GPT): **Reasoning** — Low | Medium | High | Extra High
- Google (Gemini terminal): **Thinking** — low / medium / high (CLI flag, varies by Gemini version)

Updated throughout both workflow files: tier rosters, baseline strategies, deviation guidance, performance tuning section, advisory block format, STRATEGY.md cycle table cell format. Examples: `Codex — GPT-5.5 (Reasoning: High)` (was `Codex GPT-5.5 High reasoning`); `Github Copilot — GPT-5.5 (Reasoning: Medium)`; `Gemini 3.1 Pro Preview (Thinking: high)` (was `Gemini 3.1 Pro Preview (thinking budget: high)`). Per-role recommendation table reorganized into columns Claude Effort | GPT Reasoning | Gemini Thinking so all three are visible side-by-side.

**6. Codex self-identification fix (AGENT_CONVENTIONS authorship table).**
Cycle 14S.1a's closeout commit message had `Codex (GPT-5)` — wrong on two counts: GPT-5 vs GPT-5.5, and missing the reasoning level. Updated the per-agent authorship format table:
- All rows now show parallel `<harness> <version> (<model-id> ; <Toggle>: <value>)` form (semicolon inside parens since model and toggle are two fields).
- Codex row strengthened: "**Always include both model name AND reasoning value — emitting just 'Codex (GPT-5)' or omitting reasoning is incorrect.** Model name is GPT-5.5 (or GPT-5.4 if user confirms older), not 'GPT-5'."
- All other rows (Claude Code, Gemini CLI, Gemini Code Assist, GitHub Copilot, ChatGPT) updated to include the appropriate toggle in their format and example.
- Rules tightened: "include version AND model ID AND the runtime toggle value … all three when available. The toggle is the project's standard parallel-naming convention; do not omit it when known."
- Fixed pre-existing typo: stray comma in `Codex-cli 0.118-alpha.2,  (GPT-5.4 …)` → clean form.

**7. Per-role git commit convention (v2 batching ≠ commit batching).**
The v2 workflow batches CHANGELOG entries (one per cycle) but does not require batching commits. Established that each agent role emits its own commit at the end of its session, keeping git history granular for bisect/blame even though changelog is consolidated. Two forms:

*Mid-cycle commits* (R1 audit, R2 implementation when re-audit declared, etc.): single-line `-m` form with `Phase <N>, cycle <name> (index <i>), Role <r> <round type> (Template <X>): <summary>. | Authors: <consolidated authorship with toggle>` message.

*Closeout commits* (the role that writes the CHANGELOG/DEVLOG entry): the closing agent (1) writes the entry into CHANGELOG.md or DEVLOG.md, (2) writes a verbatim copy to a top-level scratch file `changelog-entry-v<X.YY.ZZ>.txt` (gitignored via `changelog-entry-*.txt`), and (3) emits three commands — `git add ...`, `git commit --file=changelog-entry-v<X.YY.ZZ>.txt`, `rm changelog-entry-v<X.YY.ZZ>.txt`. The `--file=` form fossilizes the entry verbatim in git history (searchable via `git log` for full entry text; original wording preserved against future reformatting/migration). Version-tagged filename eliminates wrong-file risk. The general convention going forward is: any commit that lands a CHANGELOG/DEVLOG entry uses the `--file=` form.

Codified in three places:
- New "Convention for agent-produced git commit blocks" section in `spec_plan_three_role_audit_loop-v2.md`, adjacent to the existing handoff-prompt convention.
- A bullet added to the "Before giving your wrap-up summary" guardrail block in every template (workflow file replace_all + orchestrator launchers replace_all).
- `AGENT_CONVENTIONS.md § Git commit convention` rewritten to reflect the two forms, the `--file=` rationale, the toggle-in-authorship requirement, and a pointer to the workflow file for full template-level details.

`.gitignore` already updated with `changelog-entry-*.txt` (user did this manually).

**Files touched:**
- `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md` (Template B conditional handoff + CHANGELOG rule; tier roster; baseline strategies; advisory block format; STRATEGY table format; new performance tuning section; parallel toggle notation throughout; new git-commit convention; replace_all on wrap-up guardrail block)
- `multi-agent/workflows/orchestrator.spec_plan_three_role_audit_loop-v2.md` (Template B mirror; new agent selection + performance tuning section at end with parallel toggle framing; replace_all on wrap-up guardrail block)
- `multi-agent/AGENT_CONVENTIONS.md` (session-boundary section: cadence guidance + cache-TTL note + v2 cycle-closeout trigger; per-agent authorship table rewritten to include toggle uniformly with strengthened Codex guidance; `Git commit convention` section rewritten to v2 mid-cycle/closeout split with `--file=` fossilization)

No code changes. No tests run (no runtime code affected).

**Authors:** John M. Urban, Claude Code 2.1.109 (claude-opus-4-7 ; Effort: High)

(Authorship corrected 2026-04-25 — original commit message recorded `claude-sonnet-4-6` without an Effort value; both fields were unverified guesses. See v0.14.70.2 for the rule that prevents this going forward.)

**Scope:** workflow-v2 (Template B conditional handoff + CHANGELOG gap; performance tuning guidance throughout), AGENT_CONVENTIONS.md (session-boundary cadence + v2 trigger update), orchestrator-v2 (new agent selection + performance tuning section)

---

## [v0.14.67] — 2026-04-24 ~03:15 EDT

### Orchestrator advisory block required above every agent-emitted handoff launcher

Small workflow UX improvement. Dev-system only; no code changes.

**Change.** The "Convention for agent-produced handoff prompts" note in `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md` grows a fourth rule: every agent-emitted handoff must be preceded by an "Orchestrator advisory" block naming the next assignee, role, template, cycle, and execution-relevant notes — all drawn from `PHASE<N>_STRATEGY.md`. This saves the orchestrator (the human user) from having to consult STRATEGY.md themselves each time a role handoff happens; the advisory tells them which agent to paste the launcher into.

The advisory sits OUTSIDE the ```text code block so the code block stays clean for paste. Required format codified in the Convention note:

```
**Orchestrator advisory (who runs this next):**
- **Cycle:** <index + name> — <one-line priority summary>
- **Role & template:** <Role 1|2|3> <round type> → use Template <A|B|C|D|E|F|G|H>
- **Assigned agent (per PHASE<N>_STRATEGY.md):**
  - Primary: <primary agent from the relevant role column>
  - Alt: <alt agent(s) from the relevant role column>
- **Notes:** <execution-relevant notes from STRATEGY Notes column, or "none">

Paste the following into the next agent's fresh chat:
```

**Mapping from STRATEGY.md role columns to workflow role + template** (also codified in the Convention note, because the naming was ambiguous otherwise):

| STRATEGY column | Next workflow role + template |
|---|---|
| R1 (audit)      | Role 1 initial audit, Template A |
| R2 (implement)  | Role 2 implementation, Template B |
| R3 (closeout)   | Role 1 re-audit, Template C (or Template H if skip-reaudit accepted) |

Final Overseer pass (Template D) draws from the "Final Overseer" row of STRATEGY.md, not a per-cycle column.

When Role 2 emits BOTH a re-audit handoff AND a skip-reaudit-accepted handoff, each gets its own advisory — the skip-reaudit variant uses Template H; the re-audit variant uses Template C; both draw the assignee from the cycle's STRATEGY R3 column.

**Downstream wording updates:**
- Template B handoff-prompt instruction updated to require an advisory per handoff (two advisories total — one for each of the two handoffs).
- Template C closing instruction updated to require an advisory before the next-cycle initial-audit launcher.
- Template H step 5 updated to require an advisory before the next-cycle initial-audit launcher.
- Template I's "Final chat output" bullet list updated to require an advisory preceding the cycle-index-1 Role 1 launcher.
- Count of rules at the top of the Prompt Templates section updated from three to four throughout these four templates.

**Follow-up (same session) — CHANGELOG vs DEVLOG routing tightened.** User asked a closeout gut-check on whether the conventions were strong enough to reliably route onionskin-program changes to CHANGELOG and everything else ("our work and collaboration and efforts") to DEVLOG. Assessment found the routing rules strong for clear cases but with a naive-agent gap: `STRATEGY.md` was not explicitly named; `multi-agent/` maintenance edits not matching a specific bucket lacked an explicit default. Fix: (a) added `STRATEGY.md` and `AUDIT_LOG.md` to the DEVLOG coverage lists in both `AGENT_CONVENTIONS.md` and `DEVLOG.md` preamble; (b) added a backstop clause — "any other edits to files under `multi-agent/` that aren't code/test/user-doc changes" — to both; (c) added an explicit "Default bias: when uncertain, DEVLOG. CHANGELOG is reserved for actual onionskin program changes" paragraph to both; (d) mirrored the updates in the planning-content routing table row for dev-system work. Converts "strong for clear cases" into "hard to get wrong even for naive agents."

**Files touched:**
- `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md` (Convention note: 4th rule + role-column mapping; Templates B / C / H / I: updated handoff-prompt instructions)
- `multi-agent/AGENT_CONVENTIONS.md` (CHANGELOG/DEVLOG section: STRATEGY/AUDIT_LOG naming + backstop + default-bias paragraph; planning-content routing table: parallel update to dev-system row)
- `multi-agent/DEVLOG.md` (preamble: STRATEGY/AUDIT_LOG naming + backstop + default-bias paragraph)

No code changes. No tests run (no runtime code affected).

**Authors:** John M. Urban, Claude Code 2.1.109 (claude-opus-4-7)

**Scope:** workflow-v2 (Convention note expanded to 4 rules, Templates B/C/H/I handoff instructions updated), orchestrator-advisory convention introduced, CHANGELOG/DEVLOG routing tightened (AGENT_CONVENTIONS.md + DEVLOG.md preamble)

---

## [v0.14.66] — 2026-04-24 ~02:30 EDT

### Strategist next-action handoff prompt — Template I closes by emitting Role 1 launcher for cycle index 1

Small but meaningful addition to the v2 workflow. Dev-system only; no code changes.

**Change.** Template I (Strategist: Phase Plan of Attack) in `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md` now requires the Strategist's closing chat message to include a ready-to-paste Role 1 initial-audit launcher prompt (Template A format) for the phase's cycle index 1, with the concrete phase number `N` substituted (not the `<N>` placeholder). This is analogous to the handoff-prompt requirements already in Templates B / C / H — every role that closes a round emits the launcher for the next role, so the orchestrator never has to reassemble a launcher manually from a template.

Specifics:
- New "Final chat output — next-action handoff prompt" section added to Template I, naming the format requirements (Template A structural reference; concrete N; concrete file paths, including supplemental-phase filename deviations; `Target cycle:` filled in as `<cycle name> (index 1) — <priorities in that cycle>`; `Current round: initial audit`).
- New Rule #6 added to Template I's rules list codifying the requirement.
- Mirror note added to the orchestrator file's "0. Strategist" launcher prompt so the user sees the expectation at paste time.

**Follow-up clarification (same session):** Generalized the hard-coded-N convention to ALL agent-produced handoff prompts (not just Template I's closing launcher). The `<N>` placeholder form in the orchestrator file exists for user convenience when cold-pasting; agents already know the phase number and must emit concrete N in their handoffs. Added a top-of-section "Convention for agent-produced handoff prompts" note in `spec_plan_three_role_audit_loop-v2.md`'s Prompt Templates header stating the rule once, then tightened the handoff-prompt instruction wording in Templates B / C / H to point at it (each now says "with the concrete phase number hard-coded (no `<N>` placeholders)").

**Second follow-up (same session) — guardrail-block parity.** User caught that spec_plan Templates A–H did not carry the two standard guardrail blocks ("Required checkpoint reads" + "Before giving your wrap-up summary comments to the user") that the orchestrator launcher prompts all carry. Result: agent-emitted handoffs "in Template A format" silently dropped those blocks, making cold-pasted launchers and agent handoffs non-equivalent in effect. Fix: propagated both guardrail blocks verbatim into Templates A, B, C, D, E, F, G, H inside `spec_plan_three_role_audit_loop-v2.md` (matching the orchestrator's per-template wording — Template A uses the stricter "— do not skip, especially post-compaction" variant; the rest use the plain "(quality guardrail)" variant). Extended the top-of-section Convention note to require the guardrail blocks in handoff prompts. Updated Template I's "Final chat output" spec to require the guardrail blocks in the Strategist-emitted Role 1 launcher. Updated the handoff-prompt instructions in Templates B / C / H to name the guardrail-block requirement explicitly.

**Third follow-up (same session) — supplemental-phase convention.** Simulation of a naive agent emitting the Strategist handoff launcher exposed one remaining variance: the `Below N and <N> = ...` line is ambiguous for phases whose file names deviate from the plain `PHASE<N>_*.md` pattern (e.g., Phase 14 Supplemental). Fix: added an explicit supplemental-phase rule to the Convention note — `Below N and <N> = <integer> (Phase <integer> <qualifier>)`, example `Below N and <N> = 14 (Phase 14 Supplemental)`, and all downstream path lines use concrete supplemental filenames rather than `PHASE<N>_*.md` substitution. Keeps the Below-N line informational (which is its role for supplementals) while keeping paths unambiguous.

**Fourth follow-up (same session) — drop `Below N` line from agent handoffs.** Revisited the `Below N` line's purpose: it only earns its keep in the orchestrator cold-paste launchers, where it lets the user set N once and have the rest of the prompt resolve via `<N>` substitution. In an agent-emitted handoff everything is already hard-coded, which makes the line redundant ceremony that future agents would cargo-cult without understanding. Rewrote the "Convention for agent-produced handoff prompts" note in `spec_plan_three_role_audit_loop-v2.md` into a 3-rule form: (1) omit the `Below N and <N> = ...` line entirely; (2) hard-code every `PHASE<N>_*.md` path with the concrete filename (supplemental filenames included); (3) include the two guardrail blocks verbatim. Retired the separate supplemental-phase sub-rule — no longer needed, since removing the line collapses the supplemental edge case. Tightened Template I's "Final chat output" spec and the handoff-prompt instructions in Templates B / C / H to point at the 3-rule convention instead of duplicating wording.

**Files touched:**
- `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md` (Template I: new "Final chat output" section + new Rule #6; Prompt Templates header: new convention note; Templates B / C / H: tightened handoff-prompt instructions)
- `multi-agent/workflows/orchestrator.spec_plan_three_role_audit_loop-v2.md` (Strategist launcher closing note)

No code changes. No tests run (no runtime code affected).

**Authors:** John M. Urban, Claude Code 2.1.104 (claude-opus-4-7)

**Scope:** workflow-v2 (Template I + orchestrator Strategist launcher), next-action handoff-prompt convention extended to Strategist

---

## [v0.14.65] — 2026-04-24 ~02:00 EDT

### Phase strategy artifact (STRATEGY.md) + Deferred-R3 rule + Cycle-index column + launcher-prompt format refresh + first STRATEGY produced

Dev-system session extending the v2 workflow with a pre-cycle planning step, a new R3 batching rule, a concrete launcher-prompt format across all templates and examples, and a Cycle-index column in the STRATEGY table; plus producing the first STRATEGY.md instance for Phase 14 Supplemental. No code changes.

**1. New planning artifact: `PHASE<N>_STRATEGY.md`.**

Introduced a third planning file that sits parallel to `PHASE<N>_SPEC.md` and `PHASE<N>_AUDIT_LOG.md`. STRATEGY.md is produced once per phase, after SPEC is locked and before the first audit cycle. It splits the phase into execution-ordered cycles ("beefy chunks"), assigns primary + at-least-one-alternative agent to each role per cycle, and names a Final Overseer. It is a living document during the phase (amendments appended, never rewritten) and is archived alongside SPEC + AUDIT_LOG at phase close.

**2. Template I — Strategist: Phase Plan of Attack.**

New prompt template added to `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md` (~200 lines). Self-contained and copy-pasteable. Contains: the user's 18-agent roster tiered into deep-reasoning / balanced / fast groups; two baseline strategies (Claude Code available vs. unavailable); deviation guidance (when to upgrade R1 to Tier 1; when to hold at Tier 2); cycle granularity rules; STRATEGY.md output format (header / phase summary / cycle table / deviation rationale / execution summary / Final Overseer / amendment log). Rules enforce: every R1/R2/R3 cell must list a Primary AND at least one Alt; draft proposed in chat first and user approval required before writing to disk; no silent descoping.

Corresponding updates elsewhere:
- `spec_plan_three_role_audit_loop-v2.md`: new "Pre-cycle: Phase Strategy (STRATEGY.md)" section inserted after Required Inputs and before Cycle Granularity, pointing at Template I.
- `orchestrator.spec_plan_three_role_audit_loop-v2.md`: Step 0 prepended to the Start-To-Finish Pipeline; new "0. Strategist — Phase Plan of Attack" launcher prompt prepended to Standard Copy-Paste Prompts.

**3. Deferred-R3 rule codified.**

New "Deferred R3 (bundled into next cycle's R3)" subsection in the Cycle Granularity section of `spec_plan_three_role_audit_loop-v2.md`. Six rules: (1) eligibility — audit-only cycles or cycles with no runtime code; (2) closeout still happens via Template H lightweight closeout by R1, recording the deferral in the CHANGELOG/DEVLOG entry; (3) absorbing cycle's R3 scope grows to include the deferred verification; (4) STRATEGY.md must explicitly name the absorbing cycle in the deferring cycle's R3 cell and mirror the deferral in the absorbing cycle's Notes; (5) no chaining — one-cycle deferral max; (6) final-cycle edge case — Final Overseer absorbs if the deferring cycle is the last before Final Overseer.

Propagated to: Template I R3 selection guidance, Template I output-format rules, and the orchestrator file's Key v2 Concepts list.

**4. First STRATEGY.md produced — Phase 14 Supplemental.**

NEW file `multi-agent/plans/PHASE14_SUPPLEMENTAL-STRATEGY.md`. 7 execution cycles + 1 Final Overseer pass covering 22 substantive-or-mechanical priorities across 25 total. Phase 1 split into 1a (mechanical, skip-reaudit eligible) and 1b (substantive, S27+S28 paired). Phase 3 split into 3a (targeted help) and 3b (S8 global pass). Phase 2 uses the new Deferred-R3 pattern: 14S.2a R3 defers to 14S.3a R3. Three Opus-tier upgrades (1b, 3b, 5a) justified in the deviation rationale section. Final Overseer = Gemini 3.1 Pro Preview after 14S.5a closes.

**5. Launcher-prompt format refresh across all 9 templates (A–I) + 3 concrete examples.**

User-initiated refactor of the prompt format in `orchestrator.spec_plan_three_role_audit_loop-v2.md`:

- Each launcher-style prompt now opens with `Below N and <N> = <phase number to be filled in by user>`, so a user only has to set `N` once and the rest of the concrete file paths populate via `PHASE<N>_SPEC.md` / `PHASE<N>_AUDIT_LOG.md` / `PHASE<N>_STRATEGY.md` substitution.
- Replaced all `{TARGET_FILE}` / `{AUDIT_LOG_FILE}` / `{TARGET_CYCLE}` / `{CLOSED_CYCLE}` / `{NEXT_CYCLE}` placeholders with concrete file-path patterns and `<cycle name> or <cycle index>` / `<closed cycle name> or <closed cycle index>` forms. Applied to both workflow files (templates A–I in `spec_plan_three_role_audit_loop-v2.md`; all launcher prompts in `orchestrator.spec_plan_three_role_audit_loop-v2.md`; the Quick Launcher Pattern shell; and the Required-Inputs suggested assignment line).
- Template B and Template H updated so agent-generated handoff prompts for the next role follow the same concrete-path + cycle-index/name format, pulled from `PHASE<N>_STRATEGY.md`. Template C similarly instructs the re-audit closing agent to produce the next cycle's initial-audit handoff prompt in that format.
- Three concrete-example sections in the orchestrator file rewritten to forward-looking Phase 15 (Phase 12 / Phase 11 references retired) and to the new format. The Batched Cycle example retained as Phase 14 Supplemental 1a since it's the most immediate concrete demonstration.
- Documented supplemental-phase file-name substitution rule (`PHASE14_SUPPLEMENTAL-*.md` vs. plain `PHASE<N>_*.md`) so the convention gracefully handles historical or supplemental phases.

**6. Cycle-index column added to STRATEGY.md format.**

The cycle table in STRATEGY.md now has an explicit `Cycle index` first column holding simple integers 1–X (X = last numbered cycle before Final Overseer). The existing `Cycle` column renamed to `Cycle name` (e.g., `14S.1a`). Agents and humans can now reference a cycle by name, by index, or by both. Deferred-R3 cells now include the absorbing cycle's index (e.g., `Defer to 14S.3a R3 (index 4)`). Applied to:

- Template I output-format spec in `spec_plan_three_role_audit_loop-v2.md`.
- `multi-agent/plans/PHASE14_SUPPLEMENTAL-STRATEGY.md` current-instance table (7 rows, indices 1–7).

**Files touched:**
- `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md` (Pre-cycle section + Template I + Deferred-R3 subsection + Templates A–I format refresh + Cycle-index column spec + Required-Inputs suggested assignment line + handoff-prompt format instruction in Templates B/C/H)
- `multi-agent/workflows/orchestrator.spec_plan_three_role_audit_loop-v2.md` (Step 0 + Strategist launcher + Key v2 Concepts bullets for STRATEGY.md and Deferred-R3 + all 8 launcher prompts in new format + Quick Launcher Pattern shell refresh + three concrete examples rewritten to forward-looking Phase 15 (first + wrap-up) and Phase 14 Supplemental 1a (batched-cycle))
- `multi-agent/plans/PHASE14_SUPPLEMENTAL-STRATEGY.md` (new file, now includes Cycle-index column)

No code changes. No tests run (no runtime code affected).

**Authors:** John M. Urban, Claude Code 2.1.104 (claude-opus-4-7)

**Scope:** workflow-v2 (spec_plan + orchestrator), new STRATEGY.md planning artifact class with Cycle-index column, launcher-prompt format refresh, Phase 14 Supplemental STRATEGY.md

---

## [v0.14.64] — 2026-04-24 ~00:00 EDT

### Workflow v2 adoption, Phase 14 Supplemental mid-stream migration, and CHANGELOG/DEVLOG split

Cross-phase methodology engineering session. No code changes. Five interlocking
pieces of work:

**1. Phase 14 Supplemental SPEC reordered into dependency phases.**

The existing SPEC was in chronological (Q-round) order, not implementation-dependency
order. Audit found multiple violations: 14-S8 (full help-string pass) sat at position
8 but depended on 14-S22/S23/S26/S27/S28/S29/S30 later; 14-S7 (placeholder-flag help)
sat at position 7 but depended on 14-S26 at position 22; 14-S11 (flag ordering) sat
too early; 14-S16 (closeout doc sweep) sat in the middle but must be last; 14-S20 (APS
re-frame) came before 14-S22 (which adds the APS flag being re-framed); etc.

SPEC rewritten with priority blocks in dependency order under six phase headings:
- **Phase 0:** 14-S1, S2, S3 (already CLOSED)
- **Phase 1** (structural parser changes): 14-S10, S4, S5, S29, S30, S27, S28, S22, S23, S26
- **Phase 2** (audit-only deliverables): 14-S12, S25, S14, S15, S21
- **Phase 3** (help-string polish): 14-S6, S7, S9, S13, S19, S20, S8 — S8 last as integrating pass
- **Phase 4** (structural polish + tooling): 14-S11, S24
- **Phase 5** (closeout): 14-S17, S18, S16

Priority identifiers unchanged. Byte-level verification confirmed all 30 priority
blocks identical (md5 `f598ad58...`) between chronological and dependency-ordered
versions. Chronological layout preserved at
`multi-agent/plans/PHASE14_SUPPLEMENTAL-SPEC.UNORDERED.md` for reference.

**2. v2 workflow system drafted (3 new files; v1 preserved).**

Three new workflow files at `multi-agent/workflows/`:
- `spec_plan_three_role_audit_loop-v2.md` (1153 lines)
- `orchestrator.spec_plan_three_role_audit_loop-v2.md` (706 lines)
- `phase-development-system_PDS-v2.md` (696 lines)

Two pre-existing workflow files renamed to `-v1` suffix; PDS already had `-v1`. v2
is opt-in per phase. Key v2 changes:

- **Sibling `PHASE<N>_AUDIT_LOG.md` file** for round-by-round audit/implementation
  history. SPEC stays clean as the authoritative contract.
- **CHANGELOG/DEVLOG batching**: one entry per cycle closeout, not per role-round
  (~3x reduction in entries per phase). Entry routes to CHANGELOG or DEVLOG by
  cycle scope.
- **Cycle granularity**: substantive priorities get individual cycles; thin
  priorities batch into phase-group cycles.
- **Substantive-priorities rule** formalized: aim 5–10 per phase, not 30 thin ones.
  "Atomize" language re-scoped to pipeline-step separation only (not priorities).
- **Authorship consolidation at cycle closeout**: closing agent surveys AUDIT_LOG
  cycle section, unions `**Authors:**` lines, deduplicates (user first), emits one
  consolidated line in the cycle-closeout entry.
- **Role 2 skip-reaudit declaration** (safety-gated, orchestrator-approved). New
  Template H added for Role 1 cycle-closeout-after-skip-reaudit (lightweight
  spot-check + SPEC status update + cycle-closeout entry). Role 1 still
  spot-checks at closeout even when skip is accepted.
- **Compaction awareness**: re-reads that would be skipped on "already read this
  session" become mandatory if the session has been compacted.
- Re-read patterns from v1 preserved as quality guardrails (not pruned — user's
  empirical experience that re-reads are the only reliable defense against
  convention drift confirmed).

**3. Phase 14 Supplemental migrated to v2 mid-stream.**

- **NEW file** `multi-agent/plans/PHASE14_SUPPLEMENTAL-AUDIT_LOG.md` (270 lines).
- Embedded audit blocks moved from SPEC byte-exactly: Role 2 implementation report
  for 14-S1/S2/S3, Role 1 Re-Audit, and the Implementation-ordering summary. Three
  extracted blocks verified content-identical post-migration.
- SPEC cleaned: audit blocks removed (2149 → 2009 lines); priority status table
  added (30 rows); `## Phase 0 — Closed priorities` heading added; v2 migration
  note added to preamble.
- Historical CHANGELOG entries v0.14.48 and v0.14.49 preserved unchanged — they
  remain per-round under v1 conventions. AUDIT_LOG's historical cycle section
  records the consolidated author list that *would* have been written under v2.

**4. AGENT_CONVENTIONS.md + 4 agent files updated for v2 (10 parallel edits).**

`AGENT_CONVENTIONS.md` (5 edits):
- "Audits require a CHANGELOG entry" rule softened — v2 workflow-driven audits
  route to AUDIT_LOG and consolidate into one entry per cycle closeout.
- `YY` session-counter definition clarified for v2 granularity (~5–15 cycle-closeout
  entries per phase, not per role-round).
- New `## AUDIT_LOG.md — v2 workflow audit trail` maintenance-rules section.
- Planning-content routing table gained AUDIT_LOG row.
- New "Multi-round cycle authorship (v2)" bullet in Authorship conventions.

`CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `.github/copilot-instructions.md` (5 parallel
edits each — Edits 6 through 10):
- Stale `PHASE11_SPEC.md` pointer fixed to `PHASE14_SUPPLEMENTAL-SPEC.md` in all 4
  agent files (resolves the inline 14-S18 deferred task).
- New `PHASE14_SUPPLEMENTAL-AUDIT_LOG.md` pointer added to tiered reading list.
- New "Workflow files — v1 and v2 (pick one per phase)" section block added.
- Planning-model section gained AUDIT_LOG paragraph.
- **Re-read contradiction resolved**: the pre-existing conflict between
  CLAUDE.md's `### File reads` "Read each file at most once per session" rule
  and the workflow files' mandated re-reads (before every CHANGELOG write; before
  wrap-up) was long-standing. Resolved to: "once per session AS A DEFAULT" with
  explicit exceptions for (a) workflow-mandated checkpoint re-reads and (b)
  post-compaction re-reads. CLAUDE.md's `Do not re-read AGENT_CONVENTIONS.md
  mid-session if you already loaded it` bullet removed entirely — it directly
  contradicted the workflow files. Same reconciliation applied to the other 3
  agent files' token-efficiency sections.

**Validation:** post-pass verification via grep confirmed: 0 `PHASE11_SPEC`
references remain across all 5 files; 0 old "once per session; track" bullets
remain across 4 agent files; 4/4 agent files carry the new "AS A DEFAULT" wording;
4/4 agent files carry the v2 workflow pointer block; `AGENT_CONVENTIONS.md`
carries all 5 edits with expected pattern counts.

**5. CHANGELOG split into CHANGELOG (product) + DEVLOG (dev-system).**

Historical CHANGELOG reflected two mixed concerns — onionskin product changes
and multi-agent methodology/workflow engineering — with recent entries running
~80% meta-work. Future readers trying to answer "what changed in onionskin the
software?" would have to filter heavily. Prospectively split into:

- **Root `CHANGELOG.md`** — onionskin product changes only: code in
  `onionskin.py`/`onionskin_core/`/`scripts/`/`tests/`, CLI flags, output schemas,
  user-facing docs.
- **New `multi-agent/DEVLOG.md`** — dev-system changes: workflow files,
  AGENT_CONVENTIONS.md, agent bootstrap files, SPEC/BRAINSTORM/FEEDBACK
  engineering, AUDIT_HISTORY process, DECISIONS, project_context when not
  tied to code.

Rules (now codified in `AGENT_CONVENTIONS.md`):
- Shared `v0.X.YY` version stream. A version appears in one file or the other,
  not both (except as cross-reference).
- Dev-system entries use `**Scope:**` line (naming affected workflow/convention
  artifacts) instead of `**Roadmap:**`.
- Routing by primary content. Mixed entries go in whichever matches primary
  content with a brief cross-reference in the other.
- Historical CHANGELOG entries (pre-v0.14.64) are NOT rewritten or migrated. The
  split applies prospectively. Root CHANGELOG preamble gains a scope-clarification
  note pointing at DEVLOG for dev-system history going forward.
- This entry (v0.14.64) is the switchover point. It's pure dev-system, so it
  lives here in DEVLOG, not in CHANGELOG.

Additional files updated for the split:
- `AGENT_CONVENTIONS.md` — new `## DEVLOG.md — dev-system changelog` section;
  updates to existing `## CHANGELOG.md` section clarifying product scope; updates
  to v2-related passages to say "CHANGELOG or DEVLOG, whichever matches cycle
  scope"; new `**Scope:**` line rule for DEVLOG entries.
- `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `.github/copilot-instructions.md` — DEVLOG
  added to tiered reading list.
- `multi-agent/project_context/HANDOFF.md` — v0.14.64 reference updated to point
  at DEVLOG.
- `multi-agent/AUDIT_HISTORY.md` — v0.14.64 row's "CHANGELOG written" note updated
  to "DEVLOG written".
- Three v2 workflow files (`spec_plan`, `orchestrator`, `PDS`) — passages about
  CHANGELOG batching updated to cover the CHANGELOG/DEVLOG routing choice.

**Files changed this round:**
- `multi-agent/plans/PHASE14_SUPPLEMENTAL-SPEC.md` — reordered + v2-migrated
- `multi-agent/plans/PHASE14_SUPPLEMENTAL-SPEC.UNORDERED.md` — **NEW** (chronological backup)
- `multi-agent/plans/PHASE14_SUPPLEMENTAL-AUDIT_LOG.md` — **NEW** (v2 audit log)
- `multi-agent/workflows/spec_plan_three_role_audit_loop.md` → renamed to `-v1.md`
- `multi-agent/workflows/orchestrator.spec_plan_three_role_audit_loop.md` → renamed to `-v1.md`
- `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md` — **NEW**
- `multi-agent/workflows/orchestrator.spec_plan_three_role_audit_loop-v2.md` — **NEW**
- `multi-agent/workflows/phase-development-system_PDS-v2.md` — **NEW**
- `multi-agent/AGENT_CONVENTIONS.md` — v2 edits + DEVLOG/CHANGELOG split rules
- `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `.github/copilot-instructions.md` — v2 + DEVLOG
- `multi-agent/DEVLOG.md` — **NEW** (this file)
- `CHANGELOG.md` — preamble gained scope-clarification note; v0.14.64 routed here to DEVLOG
- `multi-agent/project_context/HANDOFF.md` — session update + orientation pointer
- `multi-agent/project_context/TASK.md` — cycle planning update
- `multi-agent/AUDIT_HISTORY.md` — audit row

No code changes. No tests run (no runtime code affected).

**Authors:** John M. Urban, Claude Code 2.1.104 (claude-opus-4-7)

**Scope:** workflow-v2 (new), `AGENT_CONVENTIONS.md` (+5 edits + DEVLOG split rules),
all 4 agent bootstrap files (+5 parallel edits + DEVLOG reading list), Phase 14
Supplemental SPEC (reordered + migrated), `PHASE14_SUPPLEMENTAL-AUDIT_LOG.md` (new),
`CHANGELOG.md` / `DEVLOG.md` split (new prospective routing rule).
