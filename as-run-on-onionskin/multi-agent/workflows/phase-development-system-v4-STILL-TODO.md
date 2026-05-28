# Phase Development System v4 — Still TODO

**Sibling to:** `multi-agent/workflows/phase-development-system-v4.md`

**Purpose:** Punch list of everything left to do before v4 can be promoted from DRAFT to active default. Captures the latest audit findings, the deferred Pass 4c spec, the phase-trial-use prerequisite, and the promotion checklist. Single source of truth for "where are we with v4?"

**Last updated:** 2026-05-03 EDT (after v0.14.89.9 / Pass 5f committed; Pass 5g audit landed but not yet applied).

**Current v4 file state:** 2659 lines, DRAFT, structurally complete; v3 remains active default. Phase 15 closed under v3.

---

## Top-level work remaining (in execution order)

1. **Pass 5g — clear last reviewer-audit findings** (see § Pass 5g findings below). Includes 1 Principal decision (Finding 2) before it can fully apply.
2. **Phase-trial use** — use v4 as DRAFT for one full phase (likely Phase 16) to surface real-use friction before locking template bodies. (See § Phase-trial use.)
3. **Pass 4c — full prompt-body inlining for all 22 templates** (see § Pass 4c specification). Hard prerequisite: legacy -v2 self-reference cleanup throughout pasted bodies. Incorporates phase-trial findings and resolves any tensions surfaced by Pass 5g Finding 2 + Finding 3.
4. **Promotion to active default** (see § Promotion checklist). Updates agent bootstrap files + AGENT_CONVENTIONS.md + archives v3 files. Single-commit ceremony.

The order matters: 5g cleans the base; phase-trial surfaces structural problems Pass 4c would otherwise entrench; Pass 4c locks the template bodies; promotion declares v4 active.

---

## Pass 5g findings (latest audit; not yet applied)

Latest reviewer round verdict: **CLEAN WITH MINOR POLISH.** All 10 high-value invariants pass (artifact creation timing, SURPRISE_LOG required-at-Stage-4, Final Overseer reads live not archived, honest "self-contained at policy layer" framing, 22-template count, Appendix A mappings accurate after Pass 5b/5c/5d/5e/5f, Phase Development Rules migrated + cross-referenced from Stages 2/3, audit-loop-v3 § Required Inputs migrated to v4 § 1, Templates A/B/C/D all SURPRISE-aware, archive-timing discipline consistent across § 2 / § 9 / § 11, Common Failure Modes count 16+7=23 matches Appendix A claim).

The seven findings + two optional polish items below are all polish-tier — none block promotion, but they do affect the base Pass 4c will inline against.

### Finding 1 — MEDIUM: SURPRISE_LOG skeleton drops v3's "copy preamble + Conventions verbatim from PHASE15" instruction

**Citation:** v4 file lines 2424-2456 vs `spec_plan_three_role_audit_loop-v3.md` lines 713-717.

**What v3 says:** SURPRISE_LOG is "created by copying the preamble + Conventions sections verbatim from the latest live PHASE<N-1>_SURPRISE_LOG.md (or, if this is the first phase using the convention, from `multi-agent/plans/PHASE15_SURPRISE_LOG.md` as the canonical reference). Update the title heading to PHASE <N> and the Entries section starts empty."

**What v4 says:** A 7-line empty skeleton (title + 5-line description + `(No entries filed yet.)`); delegates to `AGENT_CONVENTIONS.md § SURPRISE_LOG.md` for the rest.

**Why it matters:** Following v4 verbatim, an agent creating PHASE16's SURPRISE_LOG gets a 7-line stub instead of the structurally rich preamble + Conventions sections that PHASE15 establishes as canonical. Gap surfaces only at first triage when the file lacks status-taxonomy + ID-format + landing-zone docs the templates assume readers can find at the top of the file.

**Fix path:** Either (a) inline the v3 instruction verbatim into v4 § 16 SURPRISE_LOG skeleton block, OR (b) verify `AGENT_CONVENTIONS.md § SURPRISE_LOG.md` actually carries the preamble + Conventions template and tighten the cross-reference to "see AGENT_CONVENTIONS.md § SURPRISE_LOG.md for the full preamble + Conventions blocks, which MUST be copied verbatim — the 'No entries filed yet' line below is just the empty Entries section."

### Finding 2 — MEDIUM: v3 Template B "Role 2 writes CHANGELOG at skip-reaudit declaration" conflicts with v4 § 8 Step 3 + Failure Mode #5 (PRINCIPAL DECISION REQUIRED)

**Citations:**
- v3 Template B in `spec_plan_three_role_audit_loop-v3.md` lines 1807-1812: "If you are recommending skip-reaudit, write the cycle's CHANGELOG/DEVLOG entry now — you are the de facto closing agent if the skip is accepted."
- v4 § 8 Step 3 in v4 file lines 1180-1183: "If skip-reaudit accepted by Principal: Role 1 performs lightweight spot-check ... then closes the cycle (SPEC priority status table update + CHANGELOG entry ...)"
- v4 § 13 Failure Mode #5 in v4 file lines 2101-2102: "A role writes a CHANGELOG entry mid-cycle instead of at cycle closeout."

**Why it matters:** Under v4, an agent pasting v3 Template B verbatim is told to write the CHANGELOG at skip-reaudit declaration. v4 § 8 Step 3 says Role 1 writes it at Template H closeout. v4 Failure Mode #5 frames mid-cycle CHANGELOG as a failure mode. The three positions are not obviously reconcilable. v3 carried this same tension but did not surface it; v4's chance to clarify was not taken.

**Decision required from Principal:**
- **Option A (recommended by reviewer):** Preserve v3 Template B behavior + add explicit exception clause to v4 § 8 Step 3 / Failure Mode #5: "Exception: Role 2 may pre-write the CHANGELOG entry at skip-reaudit declaration as the de-facto closing agent draft; Role 1 at Template H closeout finalizes/edits-in-place rather than re-writing."
- **Option B:** Drop the pre-write instruction (move to v4 single-author rule); Role 1 at Template H is the sole CHANGELOG writer for the skip-reaudit branch. Flag as Pass 4c update to Template B body (when inlined).

Either way, the decision must be surfaced explicitly in v4 so the next agent doesn't have to reconcile silently. **This is the only Pass 5g item that blocks itself on Principal input.**

### Finding 3 — LOW-MEDIUM: v4 § 1 universal assignment-line scaffold includes `Surprise log file:`; v3 Templates E/F/G/H don't

**Citations:**
- v4 § 1 scaffold: v4 file lines 284-294
- v3 Templates E/F/G/H: `spec_plan_three_role_audit_loop-v3.md` lines 2046-2197 — only Spec / Audit log / Strategy file lines; no Surprise log file line

**Why it matters:** v4 generalizes the assignment-line scaffold to include `Surprise log file:` for every cycle round. v3 Templates A/B/C/D have it; E/F/G/H do not. Agent pasting E/F/G/H verbatim from v3 will not surface the SURPRISE_LOG to those rounds, even though Templates E (post-wrap-up triage) and G (final closeout) plausibly should sweep SURPRISE_LOG too.

**Fix path:** Add a one-liner to v4 § 14's Templates E/F/G/H index entries: "Note: when pasting from v3, add `Surprise log file: multi-agent/plans/PHASE<N>_SURPRISE_LOG.md` to the assignment-line block to align with v4 § 1 universal scaffold." Also queue as Pass 4c body-inlining edit.

### Finding 4 — LOW: Template I's "closing chat message must include Role 1 launcher" not surfaced in v4 § 14 or § 7

**Citation:** v3 `spec_plan_three_role_audit_loop-v3.md` lines 2475-2499 require the Strategist to emit the cycle 1 R1 launcher at end-of-chat.

**Why it matters:** Important behavioral expectation. v4 § 14 index for Template I just describes the STRATEGY.md output. v4 § 7 (Stage 4) likewise doesn't mention the kickoff handoff. Agent pasting Template I body verbatim will see it; agent reading only v4 might miss it.

**Fix path:** Add a single line to the Template I row in v4 § 14: "Also emits the cycle 1 R1 launcher prompt + Orchestrator advisory at end of chat (per Template I body)." Optionally repeat in § 7's "Stage transition out".

### Finding 5 — LOW: v4 § 14 cleanup note implies "v3 source files" (plural) carry -v2 self-references; only audit-loop-v3 does

**Citations:**
- v4 cleanup note: v4 file lines 2259-2276
- PDS-v3 grep verified: zero `-v2.md` self-references inside template bodies

**Why it matters:** Trivially imprecise — agents grepping for `-v2.md` will find none in PDS-v3 templates and proceed correctly. Just slightly misleading framing.

**Fix path:** Tighten "v3 source files still contain many template lines" → "audit-loop-v3 templates (A-J) still contain many template lines" + note that PDS-v3 templates (0a-0l) are clean of this issue.

### Finding 6 — LOW: § 11 cites Phase 14 historical example future readers won't recognize

**Citation:** v4 file lines 1812-1815: "tracking-directory moves (e.g., `multi-agent/BRAINSTORM.md` → `multi-agent/tracking/BRAINSTORM.md`); CLI conventions codification in `AGENT_CONVENTIONS.md`."

**Why it matters:** That move happened in a closeout sweep cycle months ago; a future Phase 17 reader won't recognize it. Generic framing would age better.

**Fix path:** Replace the parenthetical example with "(e.g., reorganizing files into a new tracking subdirectory; renaming long-lived planning files)."

### Finding 7 — LOW: § 16 startup-checklist step 3 doesn't cross-reference § 6 for AUDIT_LOG skeleton content

**Citation:** v4 file lines 2377-2379: "Stage 3 (SPEC). ... ; create skeleton `PHASE<N>_AUDIT_LOG.md`."

**Why it matters:** The skeleton template lives in two places (§ 6 SPEC engineering closeout block at lines 1095-1103, and § 16 sibling-file skeleton template at 2404-2412). Step 3 of the checklist doesn't point at either. Reader following the checklist might create an empty file.

**Fix path:** Add cross-reference: "...create skeleton `PHASE<N>_AUDIT_LOG.md` (use the skeleton block in § 6 SPEC engineering closeout / § 16 Per-phase sibling-file skeleton template)."

### Optional polish (not findings; nice-to-have)

- v4 § 16 step 4 uses bold "Also create empty SURPRISE_LOG..." inline with the Stage 4 step. Cleaner read if hoisted to its own numbered sub-step (4a / 4b).
- v4 lines 2516-2522's "Source-line-range policy" is a nice addition explaining why audit-loop-v3 row line numbers may have drifted. Consider moving the policy paragraph to the top of Appendix A so reviewers see it before the first table, not between the two tables.

---

## Phase-trial use (second prerequisite for Pass 4c)

**What v4's header says:** v4 promotion gated on "review + at least one phase of trial use."

**What that means in practice:**
- Phase 16 (or whichever next phase the Principal opens) runs as v4-DRAFT trial.
- During the trial, the Orchestrator emits cycle prompts using v4's templates index + cross-paste from v3 source files (Templates 0a-0l + A-J), with the legacy `-v2.md` cleanup applied per v4 § 14's "REQUIRED cleanup" rule.
- Agents follow v4 conventions (8-stage lifecycle spine, single-source artifact creation, SURPRISE_LOG required, etc.).
- Friction surfaces during trial — anything that doesn't read cleanly, anything where the Orchestrator has to interpret around a wording choice, anything where agents ask "wait, is X v3-stuff or v4-stuff?" — gets logged as Pass 4c input.

**Why this is non-skippable:** Pass 4c is ~1500 lines of verbatim template prose. If the structural framing has problems that only surface in real use, those problems will be entrenched after Pass 4c ships. Trial-use feedback is the only realistic way to catch them before locking template bodies.

**Optional acceleration:** if Phase 16 is small / mechanical / very-well-scoped, Principal may declare the trial "satisfied" early and proceed to Pass 4c. But at least one phase from BRAINSTORM through Final Overseer through Succession Briefing should run end-to-end under v4-DRAFT before Pass 4c locks bodies.

---

## Pass 4c specification

**What Pass 4c is:** The full prompt-body inlining round for v4 § 14 (Prompt templates). Replaces the current "templates-index with v3 cross-references" with verbatim prompt-body prose for all 22 templates.

**Scope: 22 template bodies.**
- **Stages 1-3 upstream-stage templates (12):** 0a (Soup-to-BRAINSTORM Opening), 0b (Soup-to-BRAINSTORM Auditor), 0c (Soup-to-BRAINSTORM Follow-up), 0d (Soup-to-BRAINSTORM Closeout), 0e (Brainstorm Opening — post-soup or no-soup), 0f (Brainstorm Follow-up Opening Agents 2+), 0g (Returning BRAINSTORM Agent 1), 0h (Returning BRAINSTORM Agents 2+), 0i (Opening SPEC Engineering Agent 1), 0j (Subsequent SPEC Engineering Agent 2), 0k (Subsequent SPEC Engineering Agent 1), 0l (Additional SPEC Engineering Agent 2).
- **Stage 4 Strategist template (1):** I (Phase Plan of Attack STRATEGY.md generator).
- **Stage 5 cycle-execution templates (4):** A (Initial Audit), B (Implementation), C (Re-Audit), H (Cycle Closeout After Skip-Reaudit).
- **Stage 6 Final Overseer + post-wrap-up templates (4):** D (Final Wrap-Up Audit), E (Post-Wrap-Up Triage), F (Post-Wrap-Up Implementation), G (Final Closeout After Wrap-Up Remediation).
- **Stage 7 Succession Briefing template (1):** J (Outgoing Orchestrator wisdom transfer).

**Estimated size:** ~1500 additional lines, pushing v4 from ~2659 to ~4100+ lines after Pass 4c lands.

### Pass 4c hard prerequisites

These must all be true before Pass 4c starts:

1. **Pass 5g findings cleared.** All 7 findings + 2 polish items applied OR explicitly deferred-into-Pass-4c with rationale documented.
2. **Finding 2 decision committed.** Principal has chosen Option A (preserve + exception) or Option B (single-author + drop pre-write). Pass 4c implements the chosen option in Template B's inlined body.
3. **Phase-trial use complete.** At least one phase has run end-to-end under v4-DRAFT and friction has been logged.
4. **Legacy `-v2.md` self-reference sweep ready.** The audit-loop-v3 source bodies (A-J templates) currently contain ~30+ `-v2.md` lines (`Read multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md and follow it.`, `re-read …-v2.md`, `You are acting as {ROLE_NAME} from …-v2.md`). These all become `-v4.md` (or removed where redundant) when inlined into v4 § 14. PDS-v3 source bodies (0a-0l templates) are clean — no cleanup needed.

### Pass 4c work breakdown (suggested)

1. **Pass 4c.1:** Inline Templates 0a-0l (12 upstream-stage templates) verbatim from PDS-v3, with header normalization (`-v3.md` → `-v4.md` references inside each template body). No legacy `-v2.md` cleanup required for this batch.
2. **Pass 4c.2:** Inline Template I (Strategist) verbatim from audit-loop-v3, with `-v2.md` cleanup throughout. Add Template I's "must emit cycle 1 R1 launcher at end of chat" requirement explicitly (per Pass 5g Finding 4).
3. **Pass 4c.3:** Inline Templates A-D (cycle execution + Final Overseer wrap-up audit) verbatim from audit-loop-v3, with `-v2.md` cleanup. Apply Pass 5g Finding 2 decision in Template B (Option A: preserve pre-write + add exception note; Option B: drop pre-write). Apply Pass 5g Finding 3 by adding `Surprise log file:` to Template D's assignment-line block.
4. **Pass 4c.4:** Inline Templates E-H (post-wrap-up + skip-reaudit closeout) verbatim from audit-loop-v3, with `-v2.md` cleanup. Apply Pass 5g Finding 3 by adding `Surprise log file:` to Templates E/F/G/H assignment-line blocks.
5. **Pass 4c.5:** Inline Template J (Succession Briefing) verbatim from audit-loop-v3, with `-v2.md` cleanup.
6. **Pass 4c.6:** Final integration sweep — re-grep entire v4 file for any remaining `-v2.md` / `-v3.md` references that should be `-v4.md`; refresh Appendix A "Items v3 had that v4 omits" to remove the "templates not inlined" gap (since inlining is now done); refresh Pass-4c-prerequisite mentions throughout the file (now satisfied); update v4 header status from DRAFT to one of (CANDIDATE / READY-FOR-PROMOTION) per Principal preference.

Each Pass 4c.N can be a separate commit (one DEVLOG entry per pass).

### Pass 4c risk profile

- **High token cost.** ~1500 lines of dense prompt prose; reviewing requires sustained attention.
- **High consistency risk.** 22 templates × ~70 lines average × ~10 conventions each = many opportunities for one template to drift from another. Mitigation: a uniform sweep at Pass 4c.6 against a checklist (`-v2.md`, `-v3.md` references; assignment-line scaffold completeness; SURPRISE_LOG awareness; § 12 conventions inheritance).
- **Once landed, hard to undo.** After promotion, agents cold-paste v4 templates directly. Errors get baked into multiple phases before they're caught.
- **Mitigation: do not skip phase-trial.** The combination of (real-use friction logged during trial) + (uniform sweep at Pass 4c.6) is the safety net.

---

## Promotion checklist (after Pass 4c lands)

This is the single-commit ceremony that flips v4 from DRAFT to active default. Pulled from v4 Appendix A "Reviewer checklist for v4 → active promotion" + extended with the file-update operations.

### Promotion gates (must all be true)

- [ ] Pass 4c is complete (all 22 template bodies inlined; legacy `-v2.md` swept).
- [ ] No content from v3 is lost (audit-loop-v3 → v4 + PDS-v3 → v4 mappings in Appendix A all account-for).
- [ ] All 8 stages documented + have clear transitions.
- [ ] Orchestrator role definition consistent with how it's been used in trial phase (cycle-spanning AI agent + Principal partnership; multi-chat continuity; bidirectional deliberation).
- [ ] § 12 conventions match current practice (handoff prompts, commit blocks, authorship consolidation).
- [ ] § 13 failure modes capture the patterns observed in Phase 15 + the trial phase.
- [ ] § 14 templates index covers all 22 template bodies (0a-0l + A-J; 12 upstream-stage + 10 cycle-execution / Strategist / Final-Overseer / Succession).
- [ ] Migration paths in § 16 are correct.
- [ ] No active stubs / TODOs in the body (all placeholders resolved).
- [ ] Trial-phase Orchestrator confirms v4 as-written matches what they actually did.
- [ ] All Pass 5g findings either applied OR explicitly resolved-in-Pass-4c.

### Promotion file-update operations (single commit)

When all gates pass, the promotion commit does:

1. **Update v4 header.** Change status from `DRAFT — IN DEVELOPMENT` to `Active default workflow.` Update revision markers' v4 entry to drop `(current; DRAFT)` and add a date.
2. **Update agent bootstrap files.** In each of `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `.github/copilot-instructions.md`:
   - Change v4 entry from "(future)" or "(planned)" framing to "(current default)."
   - Mark v3 entries as deprecated (paths point to `archived/` after archive operation below).
   - Update any references to "the active workflow" to point at `multi-agent/workflows/phase-development-system-v4.md`.
3. **Update `multi-agent/AGENT_CONVENTIONS.md`.** Sweep all references to `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md` and `multi-agent/workflows/phase-development-system_PDS-v3.md` → `multi-agent/workflows/phase-development-system-v4.md`. Reframe internal prose using v3 two-file framing to v4 single-file framing where it appears.
4. **Update `CLAUDE.md` "Workflow files" section.** Carry the v4 entry to the top as "current default"; demote v3 entries to "deprecated; archived."
5. **Archive v3 files.** `git mv multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md multi-agent/workflows/archived/` and `git mv multi-agent/workflows/phase-development-system_PDS-v3.md multi-agent/workflows/archived/`. (Note: not `git rm` — preserve for historical reference.)
6. **Archive this STILL-TODO file.** `git mv multi-agent/workflows/phase-development-system-v4-STILL-TODO.md multi-agent/workflows/archived/<YYYYMMDD>-phase-development-system-v4-STILL-TODO-AT-PROMOTION.md`. Promotion means this file's job is done.
7. **CHANGELOG/DEVLOG entry.** Single closeout entry under whatever version number is next, with `**Authors:**` line consolidated from all v4 development passes' authorships (this is a meta-cycle closeout — the consolidated authorship list spans ~all of Pass 1 → 5g → 4c.1 → 4c.6 + the trial-phase Orchestrator).

### Promotion is reversible (until next phase opens)

If the trial-phase Orchestrator surfaces a structural problem during promotion review, promotion can be aborted: `git revert` the promotion commit and continue Pass 4c.7+ work. Once a phase opens under v4-as-active, reverting becomes much harder (agents will have started cold-pasting v4 templates).

---

## Pass-by-pass history (summary table)

| Pass | DEVLOG version | What happened | Status |
|---|---|---|---|
| 1+2 | v0.14.89.1 | Skeleton + Orchestrator role + cycle-execution mechanics migrated | DONE |
| 3 | v0.14.89.2 | Upstream stages from PDS-v3 migrated (§§ 1, 4, 5, 6) | DONE |
| 4a | v0.14.89.3 | Conventions + failure modes + notes/migration (§§ 12, 13, 16) | DONE |
| 4b | v0.14.89.4 | Templates index with v3 cross-references (§ 14) | DONE |
| 5 | v0.14.89.5 | Appendix A v3 → v4 mapping | DONE |
| 5b+5c | v0.14.89.6 | First reviewer round (Codex + self-audit; 6 items) | DONE |
| 5d | v0.14.89.7 | Second reviewer round (Codex; 5 items) | DONE |
| 5e | v0.14.89.8 | Third reviewer round (Codex; 1 lifecycle-table item) | DONE |
| 5f | v0.14.89.9 | Fourth reviewer round (Copilot; 4 items) | DONE |
| **5g** | (next) | **Fifth reviewer round (audit shared 2026-05-03; 7 findings + 2 polish + 1 Principal decision)** | **TODO** |
| Phase-trial | (next) | **Phase 16 trial use of v4-DRAFT; friction logged for Pass 4c** | **TODO** |
| 4c.1-4c.6 | (next) | **Full prompt-body inlining for all 22 templates with `-v2.md` sweep** | **TODO** |
| Promotion | (final) | **v4 → active default; v3 → archived; agent files + AGENT_CONVENTIONS.md updated** | **TODO** |

---

## Cross-references

- Active v4 file: `multi-agent/workflows/phase-development-system-v4.md`
- v3 source files (still active default; cited from v4): `multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md`, `multi-agent/workflows/phase-development-system_PDS-v3.md`
- v1 + v2 archives: `multi-agent/workflows/archived/`
- DEVLOG history of v4 development: `multi-agent/DEVLOG.md` entries v0.14.89.1 → v0.14.89.9
- Agent bootstrap files that will need updating at promotion: `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `.github/copilot-instructions.md`
- Conventions file that will need updating at promotion: `multi-agent/AGENT_CONVENTIONS.md`

---

## Open Principal questions (parked here so they don't get lost)

1. **Finding 2 decision (Pass 5g).** Option A (preserve v3 Template B pre-write + add exception clause to v4 § 8 Step 3 / Failure Mode #5) OR Option B (drop pre-write; Role 1 sole CHANGELOG writer for skip-reaudit; flag as Pass 4c Template B body update)?
2. **Phase-trial selection.** Phase 16 as the trial? Or wait for a smaller phase (Phase 15.x supplemental) to test v4 on a lower-risk surface first?
3. **Pass 4c.6 status target.** When Pass 4c finishes, should v4 header read `CANDIDATE`, `READY-FOR-PROMOTION`, or skip the intermediate state and go straight to `Active default` in the same commit as promotion?
4. **Trial-phase Orchestrator participation in promotion review.** Should the trial-phase Orchestrator co-sign the promotion commit (i.e., be listed in `**Authors:**` of the promotion DEVLOG/CHANGELOG entry as confirming v4 matches their experience)?

These can stay parked until each prerequisite step gets there. None blocks Pass 5g.
