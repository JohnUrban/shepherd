# Interphase staging

This directory holds **active interphase planning notes + work-in-progress files** for periods when no formal phase is open.

Inter-phase mode is intentionally lighter than phase mode: targeted surgical work, one task at a time, conversational rather than ceremony-heavy. See `CLAUDE.md § Inter-phase development mode` for the operational pattern. This directory is the natural staging area for planning notes during those periods.

## Directory + lifecycle conventions

- **Filename:** `<TOPIC>.md`. No required prefix. Loose convention; topic-focused, not date-prefixed (date lives in git log + the archive's prefix when archived).
- **Content:** whatever fits the work. Surgical-task scratch notes, R1-style discussion captures, R2 implementation drafts, R3-style verification notes, design exploration, or short specs. Lighter than phase SPECs; no SPEC/AUDIT_LOG/STRATEGY ceremony required.
- **When complete:** `git mv` the file to `multi-agent/plans/archived/<YYYYMMDD>-<TOPIC>.md` (same convention as phase archives). Keeps the archive bucket chronological across phase + interphase work.

## Mode indicator

The presence + absence of files at the directory level encodes the project's current mode:

- **Phase mode:** `multi-agent/plans/PHASE<N>_*.md` files exist at the top level of `plans/`. The active phase plan + its siblings (AUDIT_LOG, STRATEGY, etc.) live there. `interphase/` is empty.
- **Interphase mode:** no `PHASE<N>_*.md` files at top level. Active inter-phase items (if any) live in `interphase/`. `interphase/` may also be empty if there's no active surgical work right now; in that case the project is just idle between phases.

`multi-agent/project_context/HANDOFF.md` cold-start states the mode explicitly; the directory structure here is the structural backstop for that signal.

## Relationship to other planning surfaces

- `multi-agent/plans/next/<THEME>_SOUP.md` — speculative future-phase candidates (themed; SOUP format).
- `multi-agent/plans/interphase/<TOPIC>.md` — active inter-phase work (THIS directory).
- `multi-agent/plans/PHASE<N>_*.md` — active phase plans (top level; only when a phase is open).
- `multi-agent/plans/archived/<YYYYMMDD>-*.md` — closed phases + archived interphase items, all in one chronological bucket.

The progression `next/` → `interphase/` → `PHASE<N>_*.md` → `archived/` is a loose lifecycle hint, not a required path. An idea may go directly from BRAINSTORM/SOUP to phase plan (skipping interphase), or land in interphase as a one-off without ever becoming a phase.

## When a phase opens

When the Principal opens a new phase, active items in `interphase/` should either (a) get archived to `multi-agent/plans/archived/<YYYYMMDD>-<TOPIC>.md` if the work is complete or no longer relevant, or (b) graduate into the new phase's planning if the work directly contributes to phase scope. Default to (a) unless the work is structurally absorbed by phase deliverables. The directory should typically be empty (or near-empty) while a phase is active.
