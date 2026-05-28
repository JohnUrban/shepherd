# Shepherd

*A human-governed multi-agent system for scientific software development.*

This repository documents **Shepherd**, a **reusable multi-agent development methodology** — a coordination layer for directing several AI coding agents (Claude Code, Codex, GitHub Copilot, and Gemini) on a single, long-running codebase — shown running on a real genomics project.

It is not a framework you install. It is a set of conventions, role definitions, planning surfaces, and audit protocols, expressed as plain markdown files that the agents themselves read. The system was built and refined over roughly two months across 15 formal development phases and ~437 versioned releases while developing `onionskin`, a Python package for detecting and analyzing developmental DNA amplification from sequencing coverage data. (The `onionskin` source code itself is not included here — this repository is about *how the work was coordinated*, not the science it produced.)

If you have read headlines about agentic coding tools and wondered what it actually takes to make them reliable on a real, correctness-sensitive scientific codebase, this repository is one practitioner's answer.

## Where things are

Everything currently lives under [`as-run-on-onionskin/`](as-run-on-onionskin/) — this is Shepherd *exactly as it ran* on the onionskin project. The repository root is intentionally kept near-empty for now (just this README), reserved for a future generalized, project-agnostic version of the system. See "A note on portability" below for why.

## Guided tour (start here)

The files fall into two audiences. **Flock-facing** files are what the agents themselves read — the bootstraps and the shared contract that ground every session. **Shepherd-facing** files are what the human (the *Principal*) uses to steer, oversee, and calibrate. The split is noted below and becomes obvious once you read a few.

**Five minutes:**

- [`MULTI-AGENT-INFRASTRUCTURE.md`](as-run-on-onionskin/multi-agent/MULTI-AGENT-INFRASTRUCTURE.md) — *shepherd-facing* — the map. Every coordination file: what it is for, what it must not duplicate, how they relate. Start here.
- [`CLAUDE.md`](as-run-on-onionskin/CLAUDE.md) — *flock-facing* — a representative agent bootstrap: what an agent reads on session start, its tiered required-reading list, and its model/efficiency rules. Its siblings ([`AGENTS.md`](as-run-on-onionskin/AGENTS.md), [`GEMINI.md`](as-run-on-onionskin/GEMINI.md), [`.github/copilot-instructions.md`](as-run-on-onionskin/.github/copilot-instructions.md)) are structurally identical by design.

**Fifteen minutes (add these):**

- [`AGENT_CONVENTIONS.md`](as-run-on-onionskin/multi-agent/AGENT_CONVENTIONS.md) — *shared contract* (Principal-authored, flock-followed) — the rules every agent obeys: authorship, changelog/roadmap format, scope authority, file ownership.
- [`workflows/spec_plan_three_role_audit_loop-v3.md`](as-run-on-onionskin/multi-agent/workflows/spec_plan_three_role_audit_loop-v3.md) — *shared protocol* — the role-based audit loop (spec → audit → implement → re-audit → close-out).
- [`AGENT_SWOT.md`](as-run-on-onionskin/multi-agent/AGENT_SWOT.md) — *shepherd-facing* — the trust-calibration tracker: per-agent strengths, weaknesses, and recurring failure patterns, plus the universal threats observed across every agent.

## The core ideas

- **Five agent tools, four bootstrap files, one consistent contract.** Each agent auto-detects its own bootstrap file (`CLAUDE.md`, `AGENTS.md`, `.github/copilot-instructions.md`, and `GEMINI.md` shared by Gemini CLI and Gemini Code Assist). The files are structurally identical by design — same required reading, same rules, same workflow — so any agent on any session is grounded the same way. (In this repo they sit under `as-run-on-onionskin/`; in a live project they sit at the repo root, where the tools auto-detect them.)
- **A "must-not-duplicate" file-ownership model.** Each file owns one domain: completed history (`CHANGELOG.md`), forward plans (`ROADMAP.md` and the phase specs), design rationale (`DECISIONS.md`), and current session state (`HANDOFF.md`, `TASK.md`). Keeping these domains separate is what prevents the contradiction and drift that otherwise accumulate over a long agent project.
- **A tiered required-reading startup protocol.** A fixed, ordered set of files every agent reads on session start, so a fresh session is never starting cold.
- **Session-boundary handoffs.** `HANDOFF.md` is updated at the end of every session — the precise bridge between sessions and across different agents. It is the countermeasure to context loss and post-compaction state loss.
- **Role-based audit loops.** Reusable 2- and 3-role protocols (Principal, Orchestrator, reviewers, Final Overseer) for auditing a spec section, implementing against the written findings, re-auditing, and closing out.
- **Per-agent SWOT tracking.** [`as-run-on-onionskin/multi-agent/AGENT_SWOT.md`](as-run-on-onionskin/multi-agent/AGENT_SWOT.md) logs each tool's recurring strengths and failure patterns over time, so trust is calibrated by evidence rather than vibes.

The system exists to close one specific gap: in scientific work, **getting code to run is not the same thing as knowing it is doing the right thing.** The conventions, audit loops, and failure-mode tracking are all in service of that distinction.

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

In this repository the defining files live under `as-run-on-onionskin/`. To try the system on your own codebase, copy the bootstrap files and the `multi-agent/` directory up to *your* repository root, then point an agent at the two files that define it:

> *"Read `CLAUDE.md` and `multi-agent/AGENT_CONVENTIONS.md` to acquaint yourself with this repository and the development rules we are abiding by."*

(Substitute the relevant bootstrap file for your agent: `AGENTS.md` for Codex, `.github/copilot-instructions.md` for Copilot, `GEMINI.md` for Gemini.)

**A note on portability.** This repository shows Shepherd *coupled to a real project*, not as a finished drop-in template. The conventions, role definitions, and workflow protocols are general in design, but in this instance they are wired to onionskin — the bootstrap files and workflows reference onionskin-specific quality checks, build targets, and pipeline context, and the phase plans and audit logs are full of onionskin detail. That coupling is deliberate: the included phase plans, audit logs, and brainstorm files are left intact as worked examples of the system in motion — evidence of how it actually ran, not templates to lift verbatim. As one measure of the discipline the system enforced: agents ran a substantial test suite (determinism, integration, and pipeline-parity checks) via the `Makefile` targets after changes — the suite itself is not published here, but its scale and the `Makefile` that drove it are part of the record.

To adapt it today, you would copy `as-run-on-onionskin/multi-agent/AGENT_CONVENTIONS.md`, the `as-run-on-onionskin/multi-agent/workflows/` directory, and the bootstrap files (`as-run-on-onionskin/CLAUDE.md`, `as-run-on-onionskin/AGENTS.md`, `as-run-on-onionskin/GEMINI.md`, `as-run-on-onionskin/.github/copilot-instructions.md`) to your repo root, then strip the onionskin-specific hooks (QC targets, pipeline references, project context) and substitute your own. A clean, project-agnostic "blank-slate" version that drops into any repository is the clear next step — and the honest current state is that it does not exist yet. What exists is the working system and a legible map of which layer is portable and which is project-specific.

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

Built by John M. Urban while developing `onionskin`. A field report on the lessons learned — *"Good news: humans are still needed in science"* — is on the Biofinysics blog: https://biofinysics.blogspot.com/2026/05/good-news-humans-are-still-needed-in.html

This is a working system extracted from a live project, not a packaged product. It is shared in the spirit of the field report: agentic AI is a genuine force multiplier, but on real scientific work the domain expert stays in the loop — and this is the scaffolding that keeps them there.
