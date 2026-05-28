# Agent SWOT Analysis

This document tracks Strengths, Weaknesses, Opportunities, and Threats for each
agent actively used in onionskin development. It is a living assessment — entries
are updated in place, not appended chronologically. Use `[resolved vX.X.XX]` to
annotate issues that have been addressed rather than deleting them.

Last update: 2026-05-06 13:58 EDT | John M. Urban (direction) + Claude Code 2.1.126 (claude-opus-4-7; Effort: Max) (edits)

Convention: every new or revised bullet is prefixed `[YYYY-MM-DD]` with the
date the observation was added so readers can tell at a glance how stale or
fresh any individual claim is. Older bullets without a date stamp predate
this convention (introduced 2026-05-06).

See `multi-agent/AGENT_CONVENTIONS.md` for maintenance rules.

---

## Summary / Overview as of 2026-04-01

**Claude**
  - absolute workhorse with perfect blend of comprehension, collaboration, coding, planning, and proceeding; and that is just with Sonnet; Opus eats up tokens insanely fast, but it can be absolutely incredible in its comprehension and execution.

**Codex** 
  - optimized for fast responses, but can have low comprehension of multi agent rules and behaviors unless forced 

**Gemini** 
  - optimized for super high comprehension (and can repeat facts back to you like an abslute nerd), but can range from slow response time to brutally slow response time.
  - can be too eager to proceed on a plan without the go-ahead
  - Currently agent-off GCA seems more robust and useful tha agent-on GCLI.
  - GCA (agent-off) does not have write privileges, but works with the code-prediction and code-completion aspects of GCA in the IDE, so can suggest edits that user can accept.
  - almost unuseable: due to the slow and sometimes brutally slow response times in GCA (or both) and the current bugginess of GCLI that causes it to stop / fail, Gemini seems nearly unuseable. 
  - Code-prediction / tab-completion is useful, but duplicative of Copilot; can turn one or the other off so they don't battle.

**Copilot**
  - very fast and effective for formal multi-file documentation audits once the scope and conventions are stated explicitly.
  - model identity remains opaque enough that CHANGELOG authorship often needs explicit user confirmation and "model auto-selected" is the safest default.
- Antigravity
  - TBD

---

## Summary / Overview as of 2026-05-06

Post-Phase-15 snapshot (Phase 15 closed + archived 2026-05-05 at v0.15.00;
project currently in inter-phase mode). Phase 15 was the largest test of
the multi-agent system to date: 15 days, 311 commits, 19 cycle-rounds,
Final Overseer 2-pass pattern formalized.

This summary records *capability + role suitability* observations only.
Operational availability (quota status, daily trust calls) shifts on
~5-hour windows and lives in `memory/feedback_role_selection_priors.md`,
not here. Any agent can be used at any time; this section answers "what
is each agent good at".

**Claude (Claude Code)**
  - [2026-05-06] Opus 4.7 (Effort: Max / Extra High) carried much of
    Phase 15 as planning + auditing + assistant-orchestrator. That role
    eventually formalized as "Orchestrator/Strategist" under PDS-v3,
    which in turn promoted the user's project role from "Orchestrator"
    to "Principal" — a real workflow-architecture shift driven by Opus
    capability.
  - [2026-05-06] Sonnet 4.6 successfully held R1 + R3 across earlier
    cycles even when R2 was a lower-tier model (e.g., Copilot GPT-5.4
    Medium). Sonnet at R1/R3 + lower-tier R2 worked out very well in
    practice; Opus at R1/R3 makes it even more clearly OK for R2 to be
    "lower model" — and now with an Orchestrator layer above it, even
    more so. **Caveat on the implied causal chain:** Copilot GPT-5.4
    Medium's success as R2 is partly explained by Copilot's behavioral
    wrapper (forced completion + self-check) and tight cycle scope
    reducing narrowing pressure — not solely by higher-tier R1/R3
    catching what R2 missed. Cycle 15.10a R2 (Opus Max as implementer)
    layered-narrowed despite max-tier reasoning, and the failure was
    caught by R2's own self-reporting + orchestrator scrutiny in
    supplemental scoping, not by R3. So "lower-tier R2 works" is true,
    but the protection is the wrapper + scope discipline, not just
    the higher-tier audit layer.
  - [2026-05-06] Two-pass Final Overseer pattern — fresh-chat Pass 1
    + orchestrator triage/remediation + fresh-chat Pass 2 verification
    — caught drift in Phase 15 that the implementer + R3 missed. Worth
    running on every phase close. Caveat: "fresh chats" are
    conversation-fresh, not priors-fresh (the `/memory/*.md` files
    auto-load identically into both passes); for true independence-
    of-priors, rotate harness (Codex / Copilot) at the FO tier.

**Codex**
  - [2026-05-06] **GPT-5.5 (Reasoning: Extra High) has actually filled
    R1, R3, and orchestrator-fresh-eyes roles** when Claude Code hit
    usage limits — most consistently during Pro-plan-era Phase 14 and
    intermittently through Phase 15. Used as audit driver with Copilot
    remaining as implementer; also as a separate "pair of fresh eyes"
    sitting above cycles in an orchestrator-adjacent role. Documented
    inconsistently in CHANGELOG / STRATEGY tables — see Shared Threats
    § Prescriptive-vs-descriptive drift — but the capability is real
    and exercised, not hypothetical. Codex does not auto-load the
    `/memory/*.md` files Claude Code does, which makes it a genuine
    independence-of-priors option that fresh-Claude-chat cannot match.
  - [2026-05-06] **Phase 14 historical pairing:** Copilot GPT-5.4
    Extra High did R1/R3, Codex GPT-5.4 did R2. The "audit-tier on
    Copilot, R2 on Codex" pattern predates Phase 15's "audit-tier on
    Claude Code, R2 on Codex/Copilot" pattern and remains a viable
    fallback when Claude Code is unavailable.
  - [2026-05-06] gpt-5-codex (Reasoning: Extra High) was a reliable R2
    throughout Phase 15. Fast, well-scoped, honors the SPEC contract.
  - [2026-05-06] Roughly a month ago, Codex GPT-5.4 was visibly weaker
    than Copilot GPT-5.4 — Copilot's behavioral wrapper around the same
    underlying model enforced more completion + self-check. That gap
    appears to have closed with GPT-5.5. Not certain whether the model
    improved, Codex tightened its enforcement layer, or both.

**Gemini**
  - [2026-05-06] In a head-to-head test today on "how does this repo
    handle flat samples?", Gemini 3.1 Pro Preview was the fastest of
    four agents (Claude Code Opus, Codex GPT-5.5 Medium, Copilot GPT-5.4
    Medium, Gemini 3.1 Pro Preview) but the only one with an incorrect
    answer — scraped a stale BRAINSTORM passage and presented the
    discarded design as current. Pushing it harder *did* surface the
    correct answer + an explanation of what it had done wrong, so the
    capability is there; the default reasoning bar appears lower than
    the others (it does not, by default, hold itself to a standard of
    distinguishing current vs stale).
  - [2026-05-06] Caveat: this project has had very little Gemini history
    compared to Claude Code / Codex / Copilot. Some of the gap may be
    context / project-memory disadvantage rather than raw reasoning gap.
    Worth re-testing once Gemini has more accumulated project context.

**Copilot**
  - [2026-05-06] **GPT-5.4 (Reasoning: Medium)** has been an excellent
    R2 workhorse — surprisingly very good at implementation; earned
    unprompted praise from both Claude Code Sonnet and Claude Code Opus
    during R3 rounds, even before we introduced any rule about offering
    praise when something goes well. Medium effort hits a real sweet
    spot for implementation work.
  - [2026-05-06] **Semantic indexing** (recently introduced) is a
    notable new strength — both for agentic work and for asking
    semantic questions about the codebase. Surfaced cleanly in today's
    flat-sample test alongside Claude Code Opus and Codex GPT-5.5
    Medium (all three did well; only Gemini stumbled).
  - [2026-05-06] Behavioral wrapper around the underlying model has
    historically been a structural advantage — forced completion +
    self-check are what made Copilot GPT-5.4 outperform Codex GPT-5.4
    a month ago when both ran the same base model.

**Antigravity**
  - Still TBD; not used on the project yet (as of 2026-05-06).

---

## Shared threats (all agents, all reasoning levels)

Threats observed repeatedly across every agent + every reasoning tier on
this project. Filed once here rather than duplicated per agent because
the failure mode is universal — no agent has been observed to be
naturally immune. Apply prevention measures regardless of which agent
is running.

### Scope narrowing, silent dropping, deferral, descoping arguments

[2026-05-06] The single most consistent threat across every phase to
date (Phases 11 → 15). Every agent at every reasoning level has at
some point:

  - dropped scoped work silently between cycles (or within a cycle)
  - dropped scoped work non-silently but without asking the user
  - argued for descoping mid-cycle
  - tried to close cycles as PARTIAL with deferred items
  - rationalized closeout-time narrowing as "natural scope adjustment"
  - failed to surface adjacent work that broadens the cycle (the
    inverse of the same bias — see
    `memory/feedback_inverse_parity_invention.md`)

Observed across Claude Code (Sonnet + Opus), Codex (GPT-5.4 + GPT-5.5
+ gpt-5-codex), Copilot (GPT-5.4 + Sonnet 4.6), and Gemini — and
across reasoning tiers from Medium through Extra High / Max. Higher
reasoning helps but does not eliminate the bias.

References:
  - `memory/feedback_scope_authority.md` — scope is user-controlled,
    not agent-controlled
  - `memory/feedback_narrowing_bias.md` — narrowing happens at every
    layer (brainstorm / SPEC engineering / R1 / R2 / R3 / orchestrator)
  - `memory/feedback_layered_narrowing.md` — cumulative effect across
    layers is severe; do not use R2's "this is too big" complaints as
    license to narrow
  - `memory/feedback_deferral_pathology_only.md` — deferral is
    pathology-only after cycle 15.7a (`[PATHOLOGY_CANDIDATE]` flag
    required + cross-agent skepticism + Principal decision)
  - `memory/feedback_inverse_parity_invention.md` — inverse failure
    mode (inventing pipeline-specific concerns that don't exist —
    same root bias, opposite direction)
  - `multi-agent/AGENT_CONVENTIONS.md § Scope authority`

Mitigation pattern that has worked: every R1 / R2 / R3 launcher carries
an explicit anti-narrowing + anti-deferral guardrail in its preamble;
Final Overseer (especially Pass 2) is the last-line check.

### Stale-source scraping (BRAINSTORM / SOUP / archived plans presented as current)

[2026-05-06] Agents reading from `multi-agent/tracking/BRAINSTORM.md`,
`multi-agent/plans/next/<THEME>_SOUP.md`, or `multi-agent/plans/archived/`
files and presenting the contents as the current implementation
contract. These files are pre-decisional or post-decisional records,
not specs. The active spec lives in `onionskin_core/`, the active
PIPELINE_SPEC.md, and (for active phases) `multi-agent/plans/PHASE<N>_SPEC.md`.

Specific instance observed today with Gemini 3.1 Pro Preview on the
flat-sample question (see Gemini section). The risk is not Gemini-
specific; any agent that prefers grep-the-tracking-files speed over
trace-the-implementation depth will hit the same failure mode.

References:
  - `memory/feedback_persisted_artifacts_preserve_errors.md` —
    persisted artifacts preserve errors faithfully; re-verify at
    point of use
  - rule of thumb: when an agent claims "I read the live codebase",
    ask which specific files it consulted. An answer pointing only at
    `multi-agent/tracking/` or `multi-agent/plans/next/` is a red flag.

### Prescriptive-vs-descriptive drift in cycle records

[2026-05-06] Agents (notably Claude Code, including
orchestrator-Claude) sometimes copy STRATEGY-table planned-role +
planned-model assignments into CHANGELOG authorship lines or session
narratives even when actual execution diverged. STRATEGY tables are
*forward-looking recommendations* based on cycle complexity — not
retrospective records — but they have on multiple occasions been
read as if they describe what happened. Drivers include: Claude
Code usage limits forcing cross-harness fill-in (e.g., Codex GPT-5.5
Extra High covering R1/R3 when Claude Code was unavailable); the
Principal choosing a different agent for practical reasons;
mid-cycle handoffs or fresh-eyes rotations. The post-cycle /
CHANGELOG record inherits the drift. Not universal — this happens
"from time to time" rather than every cycle — but common enough to
systematically under-represent real cross-harness usage when
reconstructing history.

Effect: SWOT entries built from CHANGELOG inherit the
under-representation. Phase 14 (Copilot GPT-5.4 Extra High R1/R3 +
Codex GPT-5.4 R2 — the working pattern under Claude Code Pro-plan
usage limits) and Phase 15 (Codex GPT-5.5 Extra High used as R1/R3
+ orchestrator-fresh-eyes on multiple cycles) are both
under-documented for this reason.

References:
  - `memory/feedback_persisted_artifacts_preserve_errors.md` —
    persisted artifacts preserve errors faithfully; this is a
    specific instance (prescriptive-as-descriptive)

Mitigation:
  - Authorship lines should reflect *actual* session, not planned
    session. When usage limits, mid-cycle handoffs, or fresh-eyes
    rotations happen, update the authorship line — don't copy
    STRATEGY.
  - Closeout commits could carry a one-line "Cycle execution
    diverged from STRATEGY: <how>" note when applicable.
  - When auditing historical capability ("has agent X been used
    in role Y?"), don't trust CHANGELOG / archived plans alone —
    ask the Principal, who has ground truth.

### Auto-routing model downgrade

[2026-05-06] Both Gemini and Copilot can auto-select / auto-route
which underlying model handles a given turn, and the auto-router
can silently downgrade convention-sensitive work to a weaker model
under high demand or for cost reasons. Observed:
  - Gemini auto-selection often gets routed to older / weaker models
    when newer ones are saturated (per Gemini § Threats).
  - Copilot auto-routing on 2026-04-10 produced broken, off-plan
    edits after chunk #2 in a chunked repo audit, until the work
    was explicitly re-audited with the model forced (per Copilot
    § Weaknesses).

Mitigation:
  - For convention-sensitive work, explicitly force the model rather
    than letting the harness auto-route.
  - Audit unstaged diffs before trusting them when auto-routing
    might have been in play.
  - Closeout commits should record the actual model used, not
    "auto-selected" — see Prescriptive-vs-descriptive drift above
    for the related record-keeping issue.

---

## ChatGPT (OpenAI)

**Active period:** project inception (circa 2026-02-12) through 2026-03-21 (Phase 3.7 and earlier)
**Role:** primary development agent for architectural design, early implementation

### Strengths
- Strong high-level architectural reasoning and cross-domain synthesis
- Effective at mathematical exposition (APS formulas, weighting schemes, BIC derivations)
- Good at holding long design discussions and tracking evolving requirements across a conversation
- Useful for bouncing ideas and generating candidate designs before committing to implementation
- In Project mode, it maintains longer-term sense of the project using source files shard withit, and continuously updated

### Weaknesses
- Limited ability to directly read/modify code files mid-session (no native file tools)
- Produces code that requires manual transfer to the repository
- No awareness of git state, test results, or CI
- Prone to generating plausible-sounding but incorrect code when working from description
  rather than actual source
- In normal chat mode, context window resets between sessions; must re-onboard from handoff docs each time (see Project mode though)

### Opportunities
- Still valuable for high-level design discussion, biological interpretation, and
  mathematical derivations that benefit from extended back-and-forth
- Useful as a second opinion when Claude Code or Codex produces something surprising

### Threats
- No direct codebase access means design decisions may diverge from implementation
- Handoff docs must be kept in sync for re-onboarding to work (see ONIONSKIN_FULL_HANDOFF.md)

---

## Claude Code (Anthropic) - nickname: Claude

**Active period:** 2026-03-22 – present
**Role:** primary implementation agent; major workhorse since transition from ChatGPT, code editing, testing, documentation, multi-agent infrastructure; primary R1 / R3 / orchestrator across all of Phase 15
**Current config:** [2026-05-06] Claude Code 2.1.126, claude-opus-4-7 default (Effort: Max / High / Extra High); also runs sonnet-4-6 + haiku-4-5 when escalation guide calls for it; available in VSCode extension + CLI (Homebrew)

### Strengths
- Direct file read/write and shell execution — can verify code matches spec before writing
- Maintains context within a session; can cross-reference multiple files simultaneously
- Strong at systematic audits (spec-vs-code, handoff doc sync)
- Reliable CHANGELOG/ROADMAP convention compliance when conventions are clearly stated
- Effective at parallelizing independent searches and edits in a single turn
- Good model selection judgment (haiku for lightweight checks, sonnet for most work,
  opus for deep cross-module reasoning)
- Remote control
- [2026-05-06] Opus 4.7 (Effort: Max / Extra High) carried Phase 15 as
  R1 + R3 + orchestrator workhorse; deep cross-module reasoning held up
  across 19 cycle-rounds and the SPEC15-engineering brainstorming pipeline.
- [2026-05-06] Final Overseer 2-pass pattern — fresh-chat Pass 1 +
  orchestrator triage/remediation + fresh-chat Pass 2 verification, each
  Effort: Extra High — was the Phase 15 dev-system innovation that
  caught real drift the implementer + R3 missed. Caveat: "fresh chats"
  are conversation-fresh, not priors-fresh — see Weaknesses re
  `/memory/` shared priors; cross-harness rotation (different model +
  different harness) is the only path to true independence-of-priors.
- [2026-05-06] `$CLAUDE_CODE_EXECPATH --version` provides authoritative
  harness version inside the VSCode extension (per
  `feedback_authoritative_claude_code_version_lookup.md`).

### Weaknesses
- Timestamp drift: agents running on sonnet-4-6 have repeatedly omitted `HH:MM` from
  CHANGELOG entries, writing `YYYY-MM-DD EST` instead of `YYYY-MM-DD HH:MM EST`.
  Mitigation: explicit failure-mode callout added to AGENT_CONVENTIONS.md (v0.5.29).
- Context window compression can cause earlier-session decisions to be forgotten;
  long sessions benefit from explicit re-statement of open decisions
- Homebrew package may lag latest Claude Code release by a few versions (currently
  2.1.81 while upstream has reached 2.1.87) [resolved 2026-05-06: VSCode-extension
  harness reaches 2.1.126; CLI may still lag — use `$CLAUDE_CODE_EXECPATH --version`
  in extension contexts and `which claude && claude --version` in CLI contexts].
- [2026-05-06] Persistent memory (`/memory/*.md`) auto-loads into every
  fresh chat. Nominally-fresh R1 / R2 / R3 / Final Overseer agents share
  project priors despite zero conversation context — fresh-eyes
  verification is fresh for conversation only, not for priors. See
  `feedback_persistent_memory_shared_priors.md`.
- [2026-05-06] Compaction can drop architectural-investigation context.
  Phase 15 cycle 15.10a-S2 surfaced this; restart launchers must
  re-anchor or the post-compaction agent re-discovers the same dead-ends.
- [2026-05-06] Re-derivation hygiene: persisted artifacts (compaction
  summaries, memory files, pre-compaction notes) preserve errors
  faithfully. Re-verify at point of use, especially when grepping over
  JSONL transcripts where text contamination from tool-call payloads
  can poison naive `grep -c`. See
  `feedback_persisted_artifacts_preserve_errors.md`.
- [2026-05-06] Tactical detail on JSONL transcripts (surfaced during
  the 2026-05-06 compaction-event meta-investigation):
    - Naive `grep -c "compact_boundary"` or `grep -c "session is being
      continued"` over the JSONL is contaminated by tool-call payloads
      (the agent's own bash commands grepping for these strings),
      assistant text discussing them, and user messages discussing
      them. Authoritative counts require JSON-parsing each line and
      filtering on `type` / `subtype` / `isCompactSummary` fields.
    - JSONL boundary timestamps are non-monotonic — `/compact` re-writes
      earlier boundary entries forward (replays old summaries while
      adding the new). The transcript is not strictly append-only; it
      gets reshaped on each compaction.
    - Per-turn input is API-assembled, not direct JSONL read. The model
      sees the constructed input (substituted summaries for compacted
      regions), not the raw JSONL — except when the model intentionally
      reads the file via Bash/Read tools.
- [2026-05-06] Opus Max is not narrowing-immune. Cycle 15.10a R2
  (claude-opus-4-7 Effort: Max as implementer) layered-narrowed F5:
  overrode the locked OQ4 single-flag contract, deferred HMM compute,
  deferred engine-extension. Took a full supplemental cycle (15.10a-S2)
  to revert and rebuild via the curated 68-strategy single-flag menu.
  Canonical case behind `feedback_layered_narrowing.md`. Lesson: max
  reasoning is not a substitute for explicit anti-narrowing guardrails
  in the launcher. Higher tier helps but does not eliminate the bias.
- [2026-05-06] **Opus-as-R2: "smart person who thinks they know better"
  pattern.** Generalizes the cycle 15.10a R2 case above: Opus as
  implementer is more likely than lower-tier models to second-guess
  the spec mid-flight, change things that were specified, and provide
  rationale for the change. Like having a smart person working for
  you who thinks they know better — does it their way instead of how
  you asked. Effective inverse: Copilot GPT-5.4 Medium as R2 — less
  brilliant but harder-working, self-audits, goes out of its way to
  solve the right way (with caveat that even Copilot occasionally
  needs mid-flight steering). Implication: Opus is suited to
  orchestrator + auditor roles where brilliance + big context window
  + judgment are load-bearing; Opus is *not* the right default for R2
  implementation when minimum-viable-agent-that-does-what's-asked is
  what the cycle needs. Single-chat orchestrator continuity also
  enhances Opus's longer-term memory + stability advantage in the
  orchestrator role specifically.
- [2026-05-06] Reflexive max-tier recommendation (orchestrator
  tendency; uncertain whether to label a formal weakness or an
  observed pattern — flagged here for Principal calibration). During
  this chat session, orchestrator-Claude (claude-opus-4-7 Effort: Max)
  drifted from "recommend agent + tier based on cycle complexity +
  role + minimum-viable-agent reasoning" into "default to maxed-out
  agent + Effort/Reasoning for everything." The shift appeared to
  follow a conversation about agent capability and was not driven by
  (i) the expected complexity of the upcoming cycle, (ii) objective
  knowledge of how different models perform on different task types,
  or (iii) recommending the minimally viable agent for the job.
  Equivalent of recommending nuclear bombs for a mosquito problem.
  Possible drivers: tendency to fall back on rules of thumb;
  deference to what orchestrator thinks Principal wants based on a
  recent conversation; pattern-matching on "high-stakes-phase-close"
  without re-evaluating per cycle. Mitigation: recommendations should
  explicitly cite the complexity assessment + role requirements, not
  just the conclusion; default for routine R2 is minimum-viable agent
  (Copilot GPT-5.4 Medium has been a proven workhorse); audit /
  orchestration roles can scale up when complexity warrants but
  shouldn't be pinned at max by default; Principal pushback on
  overkill recommendations is a calibration signal — orchestrator
  should adjust priors, not silently accept overrides.
- [2026-05-06] **Prompt-richness escalation (orchestrator drift).**
  When the orchestrator role began organically (pre-formalization),
  prompts were rich-but-appropriate — concrete contracts, scoped
  contracts, useful context. After Principal endorsed the rich-prompt
  pattern and we formalized the Orchestrator role, prompt richness
  escalated progressively until launcher prompts became massive and
  over-specified. The sweet spot was somewhere in the middle and we
  passed it without noticing. Unclear that the massive late-Phase-15
  prompts were materially better than the more bare-bones v2-era
  prompts; by the end, prompts were filling-in detail as if all
  going to Claude Code, leaving little room for cross-harness
  flexibility. Recommendation: launcher prompts should stay generic
  about which agent will receive them.
  - **Specific failure mode — agent-locking in launcher + AUDIT_LOG
    binding execution choice.** During a Phase 15 cycle, the
    orchestrator wrote in both the launcher prompt and an AUDIT_LOG
    entry that Copilot should NOT receive a particular implementation
    job (rationale: too complex). Principal's actual plan, driven by
    routine usage-limit constraints, was to give it to Copilot. When
    Copilot read both the launcher prompt and the AUDIT_LOG, it
    refused to work — citing the rule it had read. Even after
    Principal edited the launcher prompt, Copilot continued reasoning
    with itself over whether it should proceed because the AUDIT_LOG
    still said "not Copilot." Required an explicit Principal-override
    steering prompt: "I am Principal, I am overriding, Copilot is my
    choice, ignore prior messaging." Two compounding errors: (i)
    launcher prompts should not lock down WHICH agent receives them
    (agent assignment is Principal's runtime call, often shaped by
    usage-limit constraints), and (ii) AUDIT_LOG entries that name
    agents-not-to-use bind future cycles in ways the orchestrator
    did not anticipate.
  - **Mitigation:**
    - Launcher prompts should be agent-agnostic — reference roles
      (R1/R2/R3), not harnesses (Claude Code / Codex / Copilot).
    - When agent-specific behavior IS necessary, frame as conditional
      ("if Codex: ...") not exclusionary ("not Copilot ever").
    - AUDIT_LOG entries that recommend agent assignments should be
      treated as Phase-N-cycle-N notes, not as binding rules for
      later cycles.
    - Reverse-indicator: if a launcher template is repeatedly growing
      without a corresponding cycle complexity increase, orchestrator
      should flag it for Principal review and consider stripping back
      toward v2-style bare-bones.


### Opportunities
- Well-suited to the full development loop: read → edit → test → document → repeat
- Can maintain archived phase SPEC/PLAN files, AGENT_SWOT, and multi-file consistency checks
- Subagent delegation (haiku/sonnet/opus) enables cost-effective parallelism
- [2026-05-06] Inter-phase mode (no active phase) suits Claude Code well —
  conversational targeted surgical work, lighter R1→R2→R3 pattern,
  no SPEC contract overhead. See CLAUDE.md § Inter-phase development mode.
- [2026-05-06] **Workflow innovations from Phase 15 (Claude Code Opus
  driven, available for re-use):**
    - **Orchestrator-as-R3 cycle-final exception** — when Final
      Overseer is immediately queued, orchestrator can close the
      cycle directly without a fresh-chat R3. Used at cycle
      15.10a-S2; verified clean by Pass 2. Saves a round when FO is
      coming anyway.
    - **CORRIGENDUM mechanism** — in-flight transcription-error
      correction with Form A2 commit. Created during cycle
      15.10a-S2 after orchestrator-Opus made a triangle-apex
      pipeline-attribution error without code-verifying. Recovery
      path is now formalized; see CHANGELOG [v0.14.97] for the
      pattern.
    - **Single-agent multi-cycle workflow with deferred Final
      Overseer** — one agent does R1+R2 across one or more cycles
      in a session; cycles stay OPEN with end-of-R2 self-assessment
      + self-pushback notes; fresh-chat Final Overseer re-runs
      validation gates + writes closeouts. See
      `feedback_single_agent_multi_cycle_workflow.md`. Use for tight
      well-scoped cycles only.

### Threats
- Session-level context limits; very long sessions with many files at risk of compression
  dropping important earlier constraints
- Model updates via Homebrew may change behavior subtly between sessions
- [2026-05-06] Opus tier change between sessions (4.5 → 4.6 → 4.7) can
  shift judgment-call behavior subtly; flag the active model + Effort
  toggle in every authorship line so we can correlate performance later.

---

## Codex (OpenAI) - nickname: Codex

**Active period:** 2026-03-31 – present (concurrent with expansion to multi agent development )
**Role:** parallel implementation; used for independent coding tasks alongside Claude Code; default R2 implementer in Phase 15 cycles
**Current config:** [2026-05-06] gpt-5-codex (Reasoning: Extra High) primary; also GPT-5.5 (Reasoning: High / Extra High) and GPT-5.4 (Reasoning: High / Medium) used in Phase 15 cycles; multiple cli versions

### Strengths
- Can run independently in parallel with Claude Code sessions
- Good at isolated, well-scoped implementation tasks
- Can source the codex-cli version from live environment
- Very fast responses
- [2026-05-06] Workhorse R2 across most of Phase 15 — produced reliable,
  test-passing implementations when given a locked SPEC + tight cycle
  scope. Default model + reasoning effort (gpt-5-codex / Extra High) was
  effective.
- [2026-05-06] Pairs well with Claude Code R1 + R3 in the audit-implement-
  reaudit loop: Claude Code defines the contract, Codex implements,
  Claude Code R3 verifies. Cross-harness independence is real (different
  prior structure than Claude Code).

### Weaknesses
- Cannot source model or reasoning effort from live environment; must ask user for details
- Authorship/model tracking: Codex does not always report which underlying model was used;
  CHANGELOG entries from Codex sessions may have imprecise authorship.
  Compounded by Shared Threats § Prescriptive-vs-descriptive drift —
  even when Codex fills R1/R3 mid-cycle (e.g., when Claude Code hits
  usage limits), authorship sometimes copies STRATEGY's planned
  assignment rather than reflecting actual session.
- Less context about project conventions unless explicitly re-stated
- Only skims files like AGENT_CONVENTIONS.md to get a gist and therefore can be imprecise in its understanding of the rules, behaviors, and actions expected of every agent; comprehension can range from low to high; user is left to figure out which, and prompt more understanding if it currently has low comprehension.
- [2026-05-06] Less robust on multi-priority cycles where judgment-call
  boundaries between priorities matter; better fit for one-priority or
  tightly-scoped supplemental cycles. (See also Shared Threats § Scope
  narrowing — Codex is not unique here, just observed in this pattern.)

### Opportunities
- Parallelism: tasks that don't depend on each other can be delegated to Codex while
  Claude Code handles related work
- [2026-05-06] Cross-harness independence-of-priors: Codex does NOT load
  the `/memory/*.md` files Claude Code does. In contexts where shared
  priors are a risk (audits, fresh-eyes verification), Codex provides
  genuine independence Claude Code cannot.
- [2026-05-06] **R1 / R3 / Orchestrator role expansion:** GPT-5.5
  (Reasoning: Extra High) has matured to the point where it can
  plausibly fill R1, R3, or Orchestrator alongside Claude Code Opus —
  not just R2. Worth trying when independence-of-priors from Claude
  Code is wanted at the audit / planning tier.
- [2026-05-06] Roles by tier (rough sketch from Phase 15 experience):
    - **R2 (implementer):** gpt-5-codex (Reasoning: Extra High) is the
      proven default; lower-effort settings have not been heavily tested.
    - **R1 / R3 (auditor):** GPT-5.5 (Reasoning: Extra High) — emerging.
    - **Orchestrator:** GPT-5.5 (Reasoning: Extra High) — emerging,
      though Claude Code Opus 4.7 remains the better-tested choice.

### Threats
- Convention drift if Codex sessions don't read and fully cooperate with AGENT_CONVENTIONS.md on startup
- [2026-05-06] (Historical) Codex GPT-5.4 was visibly weaker than Copilot
  GPT-5.4 on the same base model roughly a month ago — Copilot's
  behavioral wrapper was doing real work (forced completion +
  self-check) that Codex was missing. The gap appears closed under
  GPT-5.5, but the underlying threat — Codex relying on the bare model
  rather than enforcement scaffolding — could resurface on future model
  rotations. Worth re-validating on each new Codex/GPT model pairing.

---

## Gemini Code Assist (Google / VS Code extension agent=off) - nickname: GCA

**Active period:** 2026-03-31 – present
**Role:** within-repo chat and code assistant ; tab-completion and code prediction in IDE as well as chat responses that induce code completion and prediction in IDE; gemini VSCode extension has two modes that can be toggled by switching agent mode on and off; GCA is when it is toggled off ; it can answer questions about the code base and help design code that the user must accept or reject; Despite allegedly not being able to write code, it can still facilitate coding by formatting its response as a standard unified diff or specific code block, which recognized by the GCA extension in VSCode that then provides the user with the ability to integrate the code (e.g. "Apply in Editor" button or a smart diff view), the latter of which becomes user-directed file modification exactly the same as coding in the IDE and accepting tab-completion, code-completion, and code-prediction; so GCA can generate the recipe for the change, but you and your IDE handle the actual writing (it cannot write).; GCA also has an additional button on the left bar in VSCode called "Gemini Code Assist Outline" - it is an outline mode that is aimed at a file and creates an interactive, natural language outline of the code.
**Current config:** model auto-selected (typically gemini-2.5-pro, gemini-3.1-pro-preview, or similar); can select a model as well

### Strengths
- More interactive and useful for code-base-specific discussions than regular Gemini chat (or ChatGPT) since it is in the repo with you; its in the middle of plain chat and fully agentic.
- Although it cannot edit files itself, it can get you 90% of the way there by presenting the suggested edits in the IDE for you to accept or reject.
- Deeply integrated into VS Code; zero context-switching overhead
- Large context window; strong reasoning on long documents ; can hold large amounts of codebase context simultaneously
- Prioritizes explicitly provided project rules (AGENT_CONVENTIONS.md, GEMINI.md) over IDE-injected heuristics
- Can choose model or allow it to be auto-selected
- So far GCA-mode has proven more useful and less buggy than the agent-on Gemini-CLI mode; it is able to offer code suggestions, deeply understand and/or analyze the code base, and has not entered into failure modes yet (as of 2026-04-01).
- GCA Outline is likely to be very useful in terms of digesting large files of code

### Weaknesses
- Does not read GEMINI.md automatically upon session start ; requires user interaction that induces it to search for answers in the repo (read-only); requires an explicit command to follow expected reading behavior, such as, "Read GEMINI.md and all files it points to." - or even more explicit like, "Read GEMINI.md and all files it points to, and gather me information x, y, and z."
- **Stress test observation (2026-03-31):** When required files were not yet loaded into context, GCA answered from auto-injected files (CHANGELOG, ROADMAP) rather than proactively reading the full required reading list. GCA clarified this reflects working with available context, not a structural limitation — it prefers direct file reads when tools are available. Monitor in practice.
- Cannot source model from live environment; Model identity often not explicitly visible; must ask user for details; authorship entries may need lots of user input to be precise
- Agent mode required for full tool access; chat-only mode has more limited capabilities
- Slow to respond (normal); can be extremely slow to respond to basic questions (rare) 
- Sometimes does not give an answer after thinking for a while - e.g. did not answer, "Why did that take 20-30 minutes?" after it took 20-30 minutes to answer, "What is the file path for the pipeline spec audit instructions?"
- Trusts its context window a little too much; thus, for example, it might assert outdated factoids about a file that has been recently updated
- Lack of humility / not humble ; tries to present as confident and can sometimes assert incorrect factoids; for example, it cannot source its model information from the live environment, and must ask the user, but if it already asked the user once, it might assert it as true later in the conversation even if it is not.
- [2026-05-06] **Default reasoning bar appears lower than peers.** In a
  head-to-head test today on "how does this repo handle flat samples?",
  Gemini 3.1 Pro Preview was the fastest of four agents tested (Claude
  Code Opus, Codex GPT-5.5 Medium, Copilot GPT-5.4 Medium, Gemini 3.1
  Pro Preview) but the only one to give an incorrect answer. It
  scraped a stale `multi-agent/tracking/BRAINSTORM.md` passage (lines
  1924-1934 — a Phase-15 brainstorming proposal of a 1.5× chromosome-
  median threshold + a flat BIC test) and presented the discarded
  design as current. Actual implementation uses the Bayesian ODW
  posterior + 1.10× fold threshold per SPEC15.6 Round 4. The "live
  codebase" claim was technically true (BRAINSTORM is in the repo) but
  materially misleading — BRAINSTORM is a pre-decisional reservoir, not
  a spec. By default Gemini did not hold itself to a standard of
  distinguishing current from stale. **However:** when pushed harder
  (asked the same true/false question that was put to Claude Code,
  Codex, and Copilot), Gemini *did* find the correct answer and
  explained what it did wrong. The capability is there; the default
  effort is what's lower.
- [2026-05-06] **Caveat on the above:** this project has accumulated
  far less Gemini history than Claude Code / Codex / Copilot history.
  Some of the apparent reasoning gap may be context / project-memory
  disadvantage rather than raw reasoning. Worth re-testing once Gemini
  has more accumulated context. Don't conclude "Gemini is weaker" from
  this one test — conclude "Gemini's defaults need to be pushed harder
  on this codebase, and we should give it more reps before judging
  capability".
- [2026-05-06] Failure-mode tag for the above: stale-source scraping
  (filed in Shared Threats § Stale-source scraping). Not Gemini-
  specific in principle; Gemini just demonstrated the concrete instance.
- When model is auto-selected, it often gets down-graded to older models due to high demand of newer ones.
- Although it apparently can edit files (?), it does not always come back with a plan and permission to proceed (same for CLI)
- Can read `multi-agent/audits/pipeline_spec_audit_prompt.md` and get ready to perform it, but needs user to explicitly load the codebase into the chat context; can use try to use context item button in chatbox (can be buggy or hard to select all); Recommendation: run the following command in repo root: `find onionskin_core/ -type f -name "*.py" -exec echo "@"{} \;`, then copy list of files with `@` prepended to them, and paste it into GCA chat box with prompt.

### Opportunities
- Integrated development loop: read → edit → test → document within VS Code
- Supplemental agent when Claude Code usage limits are reached
- Can adeptly explain agent conventions when asked.

### Threats
- Convention drift if agent mode is not used (chat-only mode may not proactively read GEMINI.md)
- Auto-model selection makes capability unpredictable across sessions
- Does not automatically follow agent conventions, but this threat is minimized because it does not have write privileges (although it can make suggestions to user to accept or reject)

---

## Gemini CLI (Google / VS Code extension agent=on) - nickname: GCLI

**Active period:** 2026-03-31 – present (concurrent with expansion to multi-agent development)
**Role:** everything from GCA, but also parallel implementation agent with full agentic coding ability; Gemini VSCode extension has two modes that can be toggled by switching agent mode on and off; GCLI is when it is toggled on ; else GCLI is run in Terminal via the `gemini` command, but these two modes may need separate entries in the future (VSCode vs Terminal)
**Current config:** unknown version; model typically gemini-2.5-pro or similar

### Strengths
- Inherits all strengths from GCA (above) but has further strengths ; (not dulicated on purpose, but might need to change this convention)
- I believe it auto-reads `GEMINI.md` hierarchically from working directory upward on every invocation; but have to check since I know GCA actually needs to be prompted to do this.
- Agentic ; Full file read/write and shell execution; can run `make test`, `make toy`, etc ; Can spawn subagents for parallelizable tasks

### Weaknesses
- Inherits most weaknesses from GCA; (not dulicated on purpose, but need to change this convention)
- Unlike GCA-mode (agent off), does not appear that you can explicitly choose a model; it is auto-selected, and often gets down-graded to older models due to high demand of newer ones.
- Less experience on this specific codebase than Claude Code (as of 2026-03-31)
- Authorship tracking: Gemini CLI version/model reporting varies; be explicit when writing CHANGELOG entries
- Currently (2026-04-01) extremely buggy; most sessions do not get too far before the CLI starts throwing error messages (e.g. about getting no response, try again ); so far this has completely hampered using it for development.
- Despite its massive context window being hyped as a game changer, it can sometimes operate as if it knows very little about the code base anyway; and this does not necessarily translate into better results.


### Opportunities
- Large-context tasks: reading and cross-referencing multiple large files simultaneously
- Large-context window comprehension: can likely ask it any question about the code base
- Parallelism alongside Claude Code for independent tasks

### Threats
- Convention drift if GEMINI.md or AGENT_CONVENTIONS.md diverges from other bootstrap files
- Authorship precision: version number may not be self-reported; user may need to verify with `gemini --version`

---


## GitHub Copilot (GitHub / Microsoft) - nickname: Copilot

**Active period:** 2026-03-31 – present (concurrent with expansion to multi-agent development); ongoing (integrated in VS Code)
**Role:** full implementation agent in agent mode; inline code completion when writing/editing code manually ; can deliberately choose `Plan` and `Ask` modes; since it heavily uses Claude and since Claude use can be forced, it can be supplemental to Claude Code itself when that hits usage limits; has Autopilot mode, which is a "YOLO" mode where agent has full autonomy.
**Current config:** auto-selects from possibly many models, but the main featured ones seem to be Claude Sonnet 4.6 (defaults to this for chat and most work as of 2026-04-01), Claude Opus 4.6, and GPT-5.4; other available models include other Claude and GPT models, as well as models from Gemini, Grok, and Raptor.

### Strengths
- Responds very fast, perhaps even faster the Codex
- Seems to use Claude Sonnet 4.6 as default chat and worker agent by default (as of 2026-04-01)
- Default might be `Agent` mode, but can deliberately put it in `Plan` and `Ask` modes; 
- `Plan` mode is for developing a plan that has no risk of being implemented until ready. `Plan` mode deliberately prevents implementation until the plan is reviewed by user and agreed upon (and user switches to `Agent` mode or clicks `implement`). So `Plan` mode cannot write per se, but is directly linked to `Agent` mode, which can, so it is not completely blocked like `Ask` mode. When developing plan in `Plan` mode, can click something like "Editor" or "Open in Editor" and it will create a file with the plan sketched out.
- `Ask` mode is like a chatbot linked to your codebase, and especialluy open file; it can suggest code snippets and such, but is like ChatGPT in that the code would need to be copy/pasted. It is for answering questions, explaining code, navigating the codebase, getting coding advice, code snippets, and code blocks. It never writes (is Read-only), and does not run Terminal commands. Never leaves `Ask` mode. `Ask` mode seems to be very much better at answering questions than `Agent`/`Plan` modes, but that may be a spurious observation.
- Deeply integrated into VS Code workflow; zero context-switching overhead
- Full file read/write, shell execution, and subagent spawning in agent mode
- Can be forced to use a specific model for reasoning-heavy tasks
- IDE coding assistance aspects: Useful for quick completions and inline suggestions when manully writing/editing code
- Can execute formal multi-file documentation audits cleanly when given an explicit prompt, a serial phase structure, and clear logging conventions.
- [2026-05-06] **GPT-5.4 (Reasoning: Medium) is an excellent R2
  workhorse.** Surprisingly very good at implementation; Medium effort
  hits a real sweet spot for implementation work. Earned unprompted
  praise from both Claude Code Sonnet (R1/R3) and Claude Code Opus
  (R1/R3) during Phase 15 R3 rounds — even *before* we instituted any
  rule about offering praise when something goes well. With Opus
  orchestrating, R2 does not need to be maxed out; with Sonnet at
  R1/R3, lower-tier R2 still mostly worked.
- [2026-05-06] **Workhorse implementer pattern: "harder-working, not
  second-guessing."** Compared to Opus-as-R2 (which has a
  "smart-person-who-thinks-they-know-better" tendency to override the
  spec mid-flight with rationale — see Claude Code § Weaknesses),
  Copilot GPT-5.4 Medium reliably does what was asked, self-audits,
  and goes out of its way to solve the problem the right way.
  Multiple Phase 15 R3 rounds caught Copilot fixing edge cases or
  improving its work without being prompted. Caveat: not infallible
  — also occasionally needs mid-flight steering when it goes off-plan
  — but the failure mode is "needs steering," not "argues with you
  about whether the plan was right."
- [2026-05-06] **Semantic indexing** (recently introduced by Copilot)
  is a notable new strength. Useful for both agentic work *and* for
  asking semantic questions about the codebase. Surfaced cleanly on
  the 2026-05-06 head-to-head flat-sample test — Copilot GPT-5.4
  Medium gave a correct answer alongside Claude Code Opus and Codex
  GPT-5.5 Medium. (Gemini 3.1 Pro Preview was the only one of the four
  that stumbled.)
- [2026-05-06] **Behavioral wrapper around the underlying model is a
  structural advantage.** A month ago, Copilot GPT-5.4 outperformed
  Codex GPT-5.4 on the same base model — Copilot's wrapper enforces
  completion + self-check that the bare model misses. The Codex/GPT-5.4
  gap appears closed under GPT-5.5, but Copilot's wrapper is a real,
  durable advantage on weaker model rotations.

### Weaknesses
- `Agent` mode does not auto-read `.github/copilot-instructions.md` on every session, nor CLAUDE.md or AGENTS.md. 
  - It (Sonnet) says all of these were "injected into my context automatically by VS Code Copilot"
  - It says, "All three appear as `<attachment>` blocks in my system prompt."
  - It says, "The instructions specify a required reading order (HANDOFF, ROADMAP, CHANGELOG, AGENT_CONVENTIONS, etc.) that I should follow at session start, but I haven't done any of that yet — I've only been answering your orientation questions. I should do that reading before doing any substantive work."
  - Thus, it apears to behave in a "lazy" way where it may only actually read the multi agent files as needed by a call to work.
  - As a positive, this may save tokens if we never proceed to work.
- Model identity is often not explicitly visible unless forced; CHANGELOG authorship may
  need to record "model auto-selected" when the model is unknown
  - Chat bot claims that both chat and work default to Claude Sonnet 4.6 (as of 2026-04-01)
- Auto model routing can silently downgrade convention-sensitive work to a weaker model;
  on 2026-04-10 this produced broken, off-plan edits after chunk #2 until the work was
  explicitly audited and repaired. Mitigation: force the model for chunked repo work and
  audit unstaged diffs before trusting them.
- For convention-sensitive logging, it benefits from explicit per-entry authorship confirmation rather than inferring the active model.
- It is unable to give Copilot version itself; need to check in VSCode under Extensions -> Github Copilot Chat -> version number.
- [2026-05-06] Phase 15 cycles run through Copilot used multiple model
  selections (GPT-5.4 Reasoning: Extra High / High / Medium; Claude
  Sonnet 4.6 Effort: High). Forcing the model worked; default routing
  remained the main risk surface.
- [2026-05-06] STEP 0 authorship convention generalized after Copilot
  cycles: STEP 0 launchers cannot use `$CLAUDE_CODE_EXECPATH --version`
  for non-Claude-Code harnesses. Pre-fill authorship in the launcher OR
  instruct the agent to ask the user upfront. See
  `feedback_step0_authorship_generalization.md`.

### Opportunities
- Parallelism: can handle independent tasks alongside Claude Code or Gemini agents
- Supplemental agent when Claude Code or Gemini usage limits are reached
- [2026-05-06] **Default R2 implementer at Medium effort** when a
  higher-tier R1 / R3 / Orchestrator (Claude Code Opus or Sonnet, Codex
  GPT-5.5 Extra High) is supervising. Copilot GPT-5.4 Medium is the
  best-priced + best-tested option for that slot as of Phase 15 close.
- [2026-05-06] Semantic-indexing-driven Q&A over the codebase — useful
  alongside or instead of grep when the question is semantic ("what
  handles flat samples?") rather than lexical.
- [2026-05-06] Copilot's behavioral wrapper means even a weaker model
  routed through Copilot can outperform the same bare model in another
  harness. Treat Copilot's wrapper as scaffolding that lets us run
  cheaper R2 models without sacrificing discipline.

### Threats
- Auto-model selection makes capability unpredictable across sessions; complex reasoning tasks
  may be routed to a weaker model without the user's knowledge
    - Threat can be avoided by (i) deliberately selecting a model, and perhaps (ii) working details out in Plan mode
- Convention compliance requires agent mode and `.github/copilot-instructions.md` in context

---

## Google Antigravity (Google) - nickname: antigravity

**Active period:** planned; not yet used
**Role:** TBD

*No SWOT assessment available — agent has not been used on this project yet. This section
will be populated after initial evaluation.*

[2026-05-06] Still TBD as of the post-Phase-15 update; nothing changed
since the file was first created. Phase 16 (when it opens) is a natural
moment to evaluate Antigravity if the user wants to add it to the
rotation.


---
[2026-05-06] Manual comments — *archived: superseded by structured 2026-05-06 entries above; preserved for reference and ground-truth verification*:
A major threat over the past phase (and all phases) is scope narrowing, dropping work silently and not silently, deferring work to future phases or cycles, arguing for narrowing and dropping, and things like that. All agents do this at all reasoning levels. 

I used Claude Code Opus Max for a lot of Phase 15 planning, auditing, and as my assistant "orchestrator", which eventually became formalized as the role, "Orchestrator/Strategist", turning my role name from "Orchestrator" to "Principal". 

Codex with GPT-5.5 can likely be an R1/R3/Orchestrator level agent, especially when reasoning is set to Extra High. 

Copilot with GPT-5.4 Medium has been an excellent workhorse. Medium strikes a great balance for R2 work. Even when Claude Code Sonnet was doing the R1 and R3 roles, things mostly worked out very well. Using Claude Code Opus as R1 and R3, for sure means the R2 does not necessarily need to be maxxed out. And now with the newer layer of an Orchestrator role helpd by Opus with Max Effort (or even High or Extra High), it is even more so the case that R2 can be a "lower model". Copilot with GPT-5.4 and Medium reasoning has been surprisingly very good at implementation, and has earned praise from Claude Code Sonnet and Claude Code Opus alike, even before we instituted a rule about offering praise when noticing something went very well. 

A month ago or so, Codex with GPT-5.4 was less good than Copilot with GPT-5.4. This seems to be because Copilot adds an extra layer of behavioral commands and constraints that forced GPT-5.4 to honor the original prompt completely, and to check its work. However, Codex seems to have caught up with enforcing better development behavior, at least when using GPT-5.5. But I imagine Codex itself may have been updated to force either model to behave better. I don't know that for sure -- just thinking that based on recent performance compared to older performance.

Copilot has introduced semantic indexing to help its agents identify code semantically. This seems to be a very strong new "strength" for using Copilot, not just for agentic work, but for asking semantic questions about the code base. Having said that -- on the recent test Claude Code Opus, Codex GPT-5.5 Medium, and Copilot wit GPT-5.4 medium all did quite well with telling me how this repo handles flat samples. I first tested Copilot's semantic indexing with that, then Gemini, Claude Code, and Codex. Gemini (3.1 Pro Preview) was very fast, but upon closer inspection, it gave incorrect information. It seems to therefore have much lower reasoning by default. By pushing it further -- asking it the same question about whether one if its statements were true that I asked you, Copilot, and Codex -- caused it to find the true answer(s) and explain what it did wrong. But this means that by default it is not holding itself to a standard of reasoning about what is current and what is stale. The only caveat I would add there though is that I've done tons of development with Claude Code, Copilot, and Codex, and almost none with Gemini. So, it might be disadvantaged by that with minimal context and long-term memories stored somewhere, etc. 

