# HANDOFF

This file is a session bookmark — last action, current state, next action — for agents returning to the project. For deeper history, see retrieval channels below.

## Cold-start orientation (read this first)

- **Phase 15 CLOSED + ARCHIVED at v0.15.00 (2026-05-05).** Project is in **inter-phase mode** — see `CLAUDE.md § Inter-phase development mode` for the lighter operational pattern. No phase is currently active (no `PHASE<N>_*.md` files at top level of `multi-agent/plans/`). Phase 16 has not yet opened; next-phase initiation is a separate Principal-driven activity (SOUP triage → BRAINSTORM → SPEC engineering → STRATEGY → cycle execution).

## Mode indicator (structural)

- **Phase mode:** `multi-agent/plans/PHASE<N>_*.md` files exist at top level. Read those for active phase plan + AUDIT_LOG + STRATEGY.
- **Interphase mode:** no `PHASE<N>_*.md` files at top level. Active inter-phase work (if any) lives in `multi-agent/plans/interphase/<TOPIC>.md`.

## Where current + forward work lives

- **Active inter-phase planning notes (if any):** `multi-agent/plans/interphase/<TOPIC>.md`. See `multi-agent/plans/interphase/README.md` for the directory's filename + lifecycle conventions. Currently empty (no active surgical work staged).
- **Near-term priorities (concrete, actionable):** `multi-agent/tracking/KNOWN_ISSUES.md`.
- **Long-term ideas reservoir (speculative):** `multi-agent/tracking/BRAINSTORM.md`.
- **Future-phase candidates (themed):** `multi-agent/plans/next/<THEME>_SOUP.md` (current SOUP files include `SUMMIT_SOUP`, `FLAT_SOUP`, `APS_SOUP`, `UNIFIED-RCN_SOUP`, `TRAJECTORY_SOUP`, `PIPELINE_PARITY_SOUP`).
- **Workflow conventions:** `CLAUDE.md` (project orientation, tiered reading, dev rules, inter-phase mode) + `multi-agent/AGENT_CONVENTIONS.md` (cross-agent conventions, phase-mode discipline, ID conventions, tracking-file lifecycle rules).

## Where historical context lives

- **Phase 15 narrative + statistics + awards + timeline + lessons:** `CHANGELOG.md [v0.15.00]` close entry.
- **Phase 15 cycle-by-cycle CHANGELOG entries:** `[v0.14.77]` through `[v0.14.97]`.
- **Phase 15 audit history (round-by-round; SPEC + STRATEGY + AUDIT_LOG + Final Overseer Pass 1+2 reports):** `multi-agent/plans/archived/20260505-PHASE15_*.md`.
- **Earlier phases:** `multi-agent/plans/archived/<YYYYMMDD>-PHASE<N>_*.md`.
- **Pre-2026-05-06 HANDOFF + TASK files (this file's bloated predecessors with detailed mid-phase cycle bullets):** `multi-agent/plans/archived/20260506-HANDOFF.md` + `multi-agent/plans/archived/20260506-TASK.md`. Archived rather than deleted; useful only if reconstructing in-flight Phase 15 cycle context that isn't already covered by CHANGELOG / AUDIT_LOG entries.
- **Recent commits:** `git log --oneline -20` (or with date filters for cross-phase work).
