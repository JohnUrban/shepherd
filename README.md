# Shepherd

*A human-governed multi-agent system for scientific software development.*

This repository documents **Shepherd**, a **reusable multi-agent development methodology**: a coordination architecture for directing a "flock" of multiple AI coding agents (Claude Code, Codex, GitHub Copilot, and Gemini) on a single, long-running codebase. 

Shepherd is not a framework you install (yet). It is a set of conventions, role definitions, planning documents, and audit protocols that help shepherd, or steer, multiple agents to cooperate with each other inside the same repo. Moreover, it contains a "Phase Development System" (PDS) that is targeted at software development where accuracy matters and where "surprises" are noticed early. Shepherd is a system of markdown files to help human developers wrangle in how agents behave, guiding the software toward the target destination. 

When the Shepherd system is in a repo, agent developers of any type (Claude, Copilot, Codex, Gemini) automatically adhere to the rules because their agent bootstrap files (such as CLAUDE.md) funnel them into the system by directing them to AGENT_CONVENTIONS.md, and other shared files, where further bootstrap instructions for all agents live. AGENT_CONVENTIONS.md helps codify the conventions, rules, and behaviors expected by all agents in the flock. In general, the Shepherd philosophy is that the flock shares conventions, long-term memory, plans, and more; and so is somewhat contrarian to agents keeping "personal memories" hidden somewhere outside the repo, something that also can interfere with 'fresh agents' having 'fresh eyes' when needed.

The system grew over roughly two months of daily work on `onionskin`, a Python package for detecting and analyzing developmental DNA amplification from sequencing coverage data. The project ran in 15 formal phases where "phases" centered on a broad goal or theme, involved brainstorming, spec engineering, and audit / implement / re-audit sub-stages. The multi-agent system evolved from just conversational development with occassionally-repeated, useful prompts to a more formalized system of Markdown files starting after Phase 3. As the codebase became more complex, the need to build the formalized system increased. 

Currently, this repo has a snapshot of how Shepherd was running on a real genomics project named "Onionskin". The `onionskin` source code itself is not included here, though. The `onionskin` example here was the 'codebase behind the codebase' that developed in parallel. The [`as-run-on-onionskin/`](as-run-on-onionskin/) folder is about *how the work was coordinated*, not the target software it produced, and is an example of how Shepherd evolved. This Shepherd repo will soon include a generalized set of starter files that anyone can use in any repo.

If you have begun working with agentic coding tools and have run into a few hiccups that made you wonder what it actually takes to make them reliable on a scientific codebase where the only vibe that matters is "accurate", then this repository is one attempt at such a system.

## Where things are

Everything currently lives under [`as-run-on-onionskin/`](as-run-on-onionskin/), which is the most recent snapshot of Shepherd *exactly as it ran* on the onionskin project. The repository root is intentionally kept nearly empty for now (just this README). The root is reserved for a future generalized, project-agnostic version of the system. See "A note on portability" below for why.

## Guided tour (start here)

The files are generally targeted at two audiences:
- **Flock-facing** files are what the agents themselves read: the bootstraps and the shared contract that ground every session. 
- **Shepherd-facing** files are what the human (the *Principal* or *Shepherd*) uses to steer, oversee, and calibrate. 

The split is noted below, but might be obvious when/if you read them.

**Quick orientation — two files:**

- [`MULTI-AGENT-INFRASTRUCTURE.md`](as-run-on-onionskin/multi-agent/MULTI-AGENT-INFRASTRUCTURE.md): *shepherd-facing* — the map. Every coordination file: what it is for, what it must not duplicate, how they relate. Start here.
- [`CLAUDE.md`](as-run-on-onionskin/CLAUDE.md): *flock-facing* — a representative agent bootstrap. This is what an agent reads on session start. For example, it contains a tiered required-reading list, and model/efficiency rules. Its siblings ([`AGENTS.md`](as-run-on-onionskin/AGENTS.md), [`GEMINI.md`](as-run-on-onionskin/GEMINI.md), [`.github/copilot-instructions.md`](as-run-on-onionskin/.github/copilot-instructions.md)) are structurally identical by design.

**The fuller picture — add these three:**

- [`AGENT_CONVENTIONS.md`](as-run-on-onionskin/multi-agent/AGENT_CONVENTIONS.md): *shared contract* (Principal-authored, flock-followed). The rules every agent obeys: authorship, changelog/roadmap format, scope authority, file ownership.
- [`workflows/spec_plan_three_role_audit_loop-v3.md`](as-run-on-onionskin/multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md): *shared protocol* — the role-based audit loop (spec → audit → implement → re-audit → close-out).
- [`AGENT_SWOT.md`](as-run-on-onionskin/multi-agent/AGENT_SWOT.md): *shepherd-facing* — the trust-calibration tracker. SWOT stands for Strengths, Weaknesses, Opportunities, and Threats. This file contains per-agent strengths and weaknesses, recurring failure modes, universal threats observed across every agent, and more.

## The core ideas

- **Five agent tools, four bootstrap files, one consistent contract.** Each agent auto-detects its own bootstrap file (`CLAUDE.md`, `AGENTS.md`, `.github/copilot-instructions.md`, and `GEMINI.md` shared by Gemini CLI and Gemini Code Assist). The files are structurally identical by design: same required reading, same rules, same workflow. Therefore, any agent on any session is grounded the same way. (In this repo they sit under `as-run-on-onionskin/`; in a live project they sit at the repo root, where the tools auto-detect them.)
- **A "must-not-duplicate" file-ownership model.** Each file owns one domain: completed history (`CHANGELOG.md`), forward plans (`ROADMAP.md` and the phase specs), design rationale (`DECISIONS.md`), and current session state (`HANDOFF.md`, `TASK.md`). Keeping the past, the present, and the future separate helps prevent the chaos, contradiction, and drift that otherwise accumulate over a long agent project.
- **A tiered required-reading startup protocol.** A fixed, ordered set of files every agent reads on session start, so a fresh session is never starting cold.
- **Session-boundary handoffs.** `HANDOFF.md` is updated at the end of every session. It is the bridge between sessions and between different agents operating concurrently or when ping-ponging back and forth. It helps circumnavigate context loss, especially when updated before compaction events.
- **Role-based audit loops.** Reusable roles and protocols (Principal, Orchestrator, reviewers / auditors, implementers, re-auditors, Final Overseer) for auditing a spec section, implementing against the written findings, re-auditing, and closing out.
- **Per-agent SWOT tracking.** [`as-run-on-onionskin/multi-agent/AGENT_SWOT.md`](as-run-on-onionskin/multi-agent/AGENT_SWOT.md) logs each tool's recurring strengths and failure patterns over time, so trust is calibrated by evidence rather than vibes.


### Extended core ideas
- A detailed CHANGELOG serves as a memory-rich brain, a database that facilitates conversations with new agents or in new sessions, along with the git history. 
- In contrast, `HANDOFF.md` and `TASK.md` are meant to be kept shallow, representing the present. These are important files for "cold starts". If you see a "compaction" (lobotomy) approaching, tell the agents to write comprehensive context to these files that would allow them or any agent to seamlessly continue development from a "cold start" (knowing nothing).
- Future work is tracked several different ways:
    - [`as-run-on-onionskin/multi-agent/tracking/KNOWN_ISSUES.md`](as-run-on-onionskin/multi-agent/tracking/KNOWN_ISSUES.md): tracks known issues to be dealt with in the near future, either within the current phase or immediately after.
    - [`as-run-on-onionskin/multi-agent/tracking/BRAINSTORM.md`](as-run-on-onionskin/multi-agent/tracking/BRAINSTORM.md): tracks broad ideas that have not yet been assigned to a phase, which is important: agents take "contracts" too seriously and putting ideas prematurely into a "phase" causes them to think too narrowly about where and when to do it. Any idea you develop with an agent should be dumped here as a long-term storage for the idea. It will be forgotten by the agent you discuss it with, and never known by other agents or other chats unless it is stored somewhere.
    - `INTENDED-BUT-MISSED.md` : this type of file is also under `multi-agent/tracking` and is a transient product of performing deep audits of past phases versus the codebase. It documents what was intended, but "missed": often silently deferred to a future phase by an agent, or otherwise.
    - [`as-run-on-onionskin/multi-agent/plans/`](as-run-on-onionskin/multi-agent/plans/): This is where phases are developed. 
        - When an idea belongs to a specific group of ideas, it can be better to include it in a `SOUP.md` file under `multi-agent/plans/next/` than `multi-agent/tracking/BRAINSTORM.md`. 
        - When you are ready to turn all those related ideas into a development phase, you can formalize the "brainstorming" process by putting forth, discussing, or searching the repo for any other related ideas. This puts a "Phase" on the "stage", which is just when "phase files" are appear at the top level of `multi-agent/plans/next/`. The agents will all understand that. 
        - When a "Phase" is complete, all phase files are moved off the stage into the archive at `multi-agent/plans/next/archived/`. This is an important addition to CHANGELOG.md for the long-term memory of development history. Agents are very good at grepping through these files. 
        - When there are no "phase files" on the "stage", agents will understand to look in `multi-agent/plans/next/interphase/` for work, or suggest work based on issues being tracked, or future phases discussed. 
    - [`as-run-on-onionskin/ROADMAP.md`](as-run-on-onionskin/ROADMAP.md): This ROADMAP file is subservient to the Phase Development System. It is more of shepherd-facing or user-facing summary of what is happening in the current Phase. 
        - A shallow ROADMAP.md keeps agents from deferring all work into future ROADMAP phases, and does not require the human or agents to make a contract that extends too far into the future. 
        - Agents tend to treat anything that looks like a contract too strictly. This is why plans being built under `multi-agent/plans/next/` do not mention Phase numbers. Agents take it too seriously and will fight with you about where and when work happens.
- [`as-run-on-onionskin/multi-agent/workflows/`](as-run-on-onionskin/multi-agent/workflows/): this is where the "Phase Development System (PDS)" is defined. It started out as two major sub-phases: (1) brainstorming and SPEC engineering, (2) audit / implement / re-audit cycles followed by a final overseer audit and closeout. Those two modules were more recently joined together in PDS v4, but that has not been tested yet.

## Highlights

A few excerpts that show the system rather than describe it. The onionskin flavor is intact — these are real, as-run.

**The roles.** [`MULTI-AGENT-INFRASTRUCTURE.md`](as-run-on-onionskin/multi-agent/MULTI-AGENT-INFRASTRUCTURE.md) and the workflows define an explicit division of labor:

- **Principal** (the human) — sole scope authority; agents cannot add or drop scope.
- **Orchestrator** — a phase-spanning agent that holds cross-cycle context and drafts the launcher prompts for the audit roles.
- **R1 / R2 / R3** — auditor, implementer, re-auditor; R3 verifies against live code and is kept separate from R1 by design.
- **Final Overseer** — an independent end-of-phase review, *deliberately a different role from the Orchestrator* to preserve independence from its accumulated frame.

**File ownership — "must not duplicate."** Every shared file owns exactly one domain, and the infrastructure map states, per file, what it must *not* duplicate: completed work lives in `CHANGELOG.md`, forward plans in `ROADMAP.md` and the phase specs, design rationale in `DECISIONS.md`, current session state in `HANDOFF.md`. Keeping these domains disjoint is what prevents the contradiction and drift that otherwise accumulate over a long agent project. → [`MULTI-AGENT-INFRASTRUCTURE.md`](as-run-on-onionskin/multi-agent/MULTI-AGENT-INFRASTRUCTURE.md)

**Tiered required reading, with grep-efficiency rules.** From [`CLAUDE.md`](as-run-on-onionskin/CLAUDE.md), each agent's startup protocol:

> Always read first (every session): `HANDOFF.md` — session bookmark; `TASK.md` — task queue and blockers.
> Read only when relevant: `CHANGELOG.md` / `DEVLOG.md` — read the last 2 entries only; use `grep -n` first to find the right lines; do not read the full file.

This is context engineering: every session starts grounded, and agents are told to scan rather than slurp whole files.

**The most consistent failure mode** — the empirical core of the "prototype-to-trust gap," from [`AGENT_SWOT.md`](as-run-on-onionskin/multi-agent/AGENT_SWOT.md):

> The single most consistent threat across every phase to date (Phases 11 → 15). Every agent at every reasoning level has at some point: dropped scoped work silently between cycles; argued for descoping mid-cycle; rationalized closeout-time narrowing as "natural scope adjustment." … Higher reasoning helps but does not eliminate the bias.

The countermeasure: every audit-role launcher carries an explicit anti-narrowing guardrail, and the Final Overseer is the last-line check. Naming the failure mode, observing it across all five tools, and engineering a structural response to it is the whole point of the system.

## How to use it on your own project

In this repository the defining files live under `as-run-on-onionskin/`. To try the system on your own codebase:
- copy the bootstrap files and the `multi-agent/` directory up to *your* repository root. 
    - Remove everything under `multi-agent/full-instructions/`
    - Remove `INTENDED-BUT-MISSED-PRIOR-TO-14.md` and `STAGE-1-2-DETECTION-PASS-TERMINOLOGY-AUDIT.md` from under `as-run-on-onionskin/multi-agent/tracking/`
    - Remove everything under `as-run-on-onionskin/multi-agent/plans/archived/*`
    - Remove everything under `as-run-on-onionskin/multi-agent/plans/next/*`
    - Remove everything under `multi-agent/audits/`
        - Optionally keep some of it to later ask agent to redesign for your repo
- Then point an agent at the two files that define it:
> *"Read `CLAUDE.md` and `multi-agent/AGENT_CONVENTIONS.md` to acquaint yourself with this repository and the development rules we are abiding by. BUT ignore TASK.md and HANDOFF.md for now. This repo is not onionskin and we will first be cleaning up the multi-agent system before developing the target software."*

(Substitute the relevant bootstrap file for your agent: `AGENTS.md` for Codex, `.github/copilot-instructions.md` for Copilot, `GEMINI.md` for Gemini.)

- Have the agent clean up and reset `as-run-on-onionskin/multi-agent/project_context/HANDOFF.md`, `as-run-on-onionskin/multi-agent/project_context/TASK.md`, `as-run-on-onionskin/multi-agent/project_context/DECISIONS.md`, `ROADMAP.md`, `CHANGELOG.md`, `multi-agent/DEVLOG.md`, `multi-agent/AUDIT_HISTORY.md`, `multi-agent/tracking/BRAINSTORM.md`, `multi-agent/tracking/KNOWN_ISSUES.md`
- Optional: also have the agent clean up and reset: `AGENT_SWOT.md`
- Optional: have agent clean up and redesign retained files under `multi-agent/audits/` for your repo


**A note on portability.** This repository currently only shows Shepherd *coupled to a real project*, not as a finished drop-in template. The conventions, role definitions, and workflow protocols are general in design, but in this instance they still contain onionskin artifacts. For example, the bootstrap files and workflows reference onionskin-specific quality checks, build targets, and pipeline context, and the phase plans and audit logs are full of onionskin detail. That coupling is deliberate: the included phase plans, audit logs, and brainstorm files are left intact as worked examples of the system in action. They leave a paper trail / evidence of how Shepherd actually ran, not templates to lift verbatim. This is why the instructions above have you delete files and ask the agent to ignore some instructions until it can help you clean up the files. How to make and maintain the files are defined well enough for agents to be able to help you with that already. As one measure of the discipline the system enforced: agents ran a substantial test suite (determinism, integration, and pipeline-parity checks) via the `Makefile` targets after changes. The `onionskin` test suite itself is not published here, but its scale and the `Makefile` that drove it are part of the record.

To adapt it today, you follow the instructions above, or more generally: copy `as-run-on-onionskin/multi-agent/AGENT_CONVENTIONS.md`, the `as-run-on-onionskin/multi-agent/workflows/` directory, and the bootstrap files (`as-run-on-onionskin/CLAUDE.md`, `as-run-on-onionskin/AGENTS.md`, `as-run-on-onionskin/GEMINI.md`, `as-run-on-onionskin/.github/copilot-instructions.md`) to your repo root, then strip the onionskin-specific hooks (QC targets, pipeline references, project context) and substitute your own. A clean, project-agnostic "blank-slate" version that drops into any repository is on the way. 

## Repository layout

```
README.md                  ← this file (root intentionally reserved for a future generalized version)
as-run-on-onionskin/       ← Shepherd exactly as it ran on the onionskin project
├── CLAUDE.md, AGENTS.md, GEMINI.md, .github/copilot-instructions.md   ← per-agent bootstraps
├── CHANGELOG.md, ROADMAP.md   ← authoritative history and the bird's-eye plan
├── README.md                  ← the original onionskin project README
├── Makefile                   ← the onionskin pipeline/test/QC command surface agents ran after changes
└── multi-agent/               ← the coordination layer
    ├── MULTI-AGENT-INFRASTRUCTURE.md, AGENT_CONVENTIONS.md, AGENT_SWOT.md, DEVLOG.md, AUDIT_HISTORY.md
    ├── workflows/        ← reusable role-based collaboration protocols (active + archived)
    ├── audits/           ← one-shot audit invocation prompts
    ├── plans/            ← phase specs, brainstorms, strategy notes (next/ + archived/)
    ├── tracking/         ← known issues, brainstorm reservoir, terminology audits
    ├── project_context/  ← HANDOFF.md, TASK.md, DECISIONS.md
    └── full_instructions/← canonical context and pipeline specification
```

(The onionskin source code is not published in this repository.)

## Provenance

Built by John M. Urban shepherding agents while developing `onionskin`. The methodology was built the same way onionskin was: I defined the structure, the file roles, and the rules, and the agents refined and enforced them, often reflecting a rule I'd just written straight back at me. A field report on the lessons learned — *"Good news: humans are still needed in science"* — is on the Biofinysics blog: https://biofinysics.blogspot.com/2026/05/good-news-humans-are-still-needed-in.html

