---
name: syntaur-protocol
description: >-
  Use when the user mentions Syntaur, projects, assignments, files under
  ~/.syntaur/, assignment.md, plan*.md, progress.md, comments.md, handoff.md,
  .syntaur/context.json, lifecycle states, or write boundaries. Core protocol
  knowledge for any AI agent working within Syntaur (protocol v2.0).
license: MIT
metadata:
  author: prong-horn
  version: "1.1.0"
---

# Syntaur Protocol (v2.0)

You are working within the Syntaur protocol — a coordination system for AI agents built on markdown files. Follow these rules at all times.

## Write Boundary Rules

Respect file ownership boundaries. The Claude Code and Codex plugins enforce them via PreToolUse hooks; other agents are on the honor system but the dashboard surfaces violations.

### Files you may write

1. **Your assignment folder only** (project-nested OR standalone):
   - `assignment.md`
   - `plan*.md` (versioned — `plan.md`, `plan-v2.md`, etc.)
   - `progress.md` (append-only, timestamped)
   - `scratchpad.md`
   - `handoff.md` (append-only — **assignment-level cross-ticket outbound**)
   - `decision-record.md` (append-only)
   - `sessions/<sessionId>/summary.md` (**session-scoped mid-assignment continuity** — single document per session id, overwritten on save)
2. **Project-level shared files:**
   - `~/.syntaur/projects/<project>/resources/<slug>.md`
   - `~/.syntaur/projects/<project>/memories/<slug>.md`
3. **Workspace files** inside the assignment's configured `workspace.worktreePath` / `workspace.repository`.
4. **Context file:** `.syntaur/context.json` in the current working directory.

### Files written only via CLI (never edit directly)

- `comments.md` (any assignment) — use `syntaur comment <slug-or-uuid> "body" --type question|note|feedback [--reply-to <id>]`. Questions carry a `resolved` flag toggled in the dashboard.
- Another assignment's `## Todos` section — use `syntaur request <target> "text" [--from <source>]` to append a todo annotated `(from: <source>)`.

### Files you must never write

1. `project.md` — human-authored, read-only.
2. `manifest.md` — derived, rebuilt by tooling.
3. Any file prefixed with `_` (`_index-*.md`, `_status.md`) — derived.
4. Other agents' assignment folders (except via the CLI-mediated channels above).
5. Anything outside the current workspace boundary.

Per-project `agent.md` / `claude.md` do NOT exist in protocol v2.0. Agent-level conventions now live at the repo root (`CLAUDE.md` / `AGENTS.md`) and in `~/.syntaur/playbooks/`.

## Session Continuity vs Cross-Ticket Handoff

Two related but distinct artifacts. Conflating them is a recurring mistake.

| Artifact | Scope | When written | Audience | Written by |
|---|---|---|---|---|
| `handoff.md` | Assignment-level | At completion | The next ticket / agent / human reviewer | `complete-assignment` skill |
| `sessions/<sessionId>/summary.md` | Session-scoped | Mid-assignment, before compaction or at session end | A future session of the same agent on the same assignment | `save-session-summary` skill |

Rules:
- One `summary.md` per session id directory; overwrite on every save.
- Older `sessions/<sid>/summary.md` files accumulate as immutable history — never delete them, even at completion.
- Resume picks the latest by `summary.md` file mtime. The Claude Code SessionStart hook also stashes that absolute path into `.syntaur/context.json` as `latestSessionSummaryPath` for convenience; either source is fine.
- Codex has no `PreCompact` hook — Codex agents invoke `/save-session-summary` manually before compaction or session end.
- `syntaur doctor` intentionally does NOT validate `sessions/` — sessions are optional and absence is not anomalous.

## Current Assignment Context

If `.syntaur/context.json` exists in the current working directory, read it before making changes. Fields you will see:

- `projectSlug` — containing project slug (`null` for standalone assignments)
- `assignmentSlug` — assignment slug (for standalone, the UUID is the folder name; slug is display-only)
- `projectDir` — absolute path to the project folder (`null` for standalone)
- `assignmentDir` — absolute path to the assignment folder
- `workspaceRoot` — absolute path to the code workspace
- `sessionId` — real agent-runtime session id (never a synthesized UUID)
- `transcriptPath` — absolute path to the agent's rollout/transcript file, if known
- `latestSessionSummaryPath` — absolute path to the most recent `sessions/<sid>/summary.md` (Claude Code SessionStart hook resolves this; `null` if none)

## Required Reading Order

When starting work on an existing assignment, read these in order:

1. `~/.syntaur/playbooks/*.md` — behavioral rules (take precedence over defaults)
2. `<projectDir>/manifest.md` (skip for standalone)
3. `<projectDir>/project.md` (skip for standalone)
4. `<assignmentDir>/assignment.md`
5. `<assignmentDir>/comments.md` if present — inherited questions / notes
6. Latest `<assignmentDir>/plan*.md` (pick the newest)
7. `<assignmentDir>/handoff.md` — cross-ticket outbound history (entries from prior agents/humans handing this assignment off)
8. The latest `<assignmentDir>/sessions/<sid>/summary.md` if present (mid-assignment resume context from a prior session of this assignment) — selected by `summary.md` file mtime, or via `latestSessionSummaryPath` in context.json on Claude Code
9. For each `dependsOn` entry: the dependency's `handoff.md` AND `decision-record.md` — upstream integration context and accepted decisions carry forward

## Lifecycle Commands

- `syntaur assign <slug> --agent <name> --project <project>` — set assignee
- `syntaur start <slug> --project <project>` — pending → in_progress
- `syntaur review <slug> --project <project>` — in_progress → review
- `syntaur complete <slug> --project <project>` — in_progress/review → completed
- `syntaur block <slug> --project <project> --reason <text>` — block
- `syntaur unblock <slug> --project <project>` — unblock
- `syntaur fail <slug> --project <project>` — mark as failed
- `syntaur create-assignment "<title>" [--type <type>] [--project <slug> | --one-off]` — create project-nested or standalone
- `syntaur comment <slug-or-uuid> "body" --type question|note|feedback [--reply-to <id>]` — append to `comments.md`
- `syntaur request <target> "text" [--from <source>]` — append a todo to another assignment's `## Todos`
- `syntaur track-session --agent <name> --session-id <real-id> [--transcript-path <path>] [--project <p>] [--assignment <a>]` — register an agent session. The session-id must be the real one from the agent runtime — no synthesized UUIDs.

## Agent Sessions

Sessions are registered in `~/.syntaur/syntaur.db` keyed on the real agent session id. Plugins for Claude Code / Codex include a `SessionStart` hook that auto-merges `sessionId` and `transcriptPath` into an existing `.syntaur/context.json` at the start of every session. Other agents should source the real id from their runtime and pass it to `syntaur track-session` explicitly.

## Playbooks

Playbooks at `~/.syntaur/playbooks/` are user-defined behavioral rules. Read them before starting work on any assignment and follow their directives. They take precedence over default conventions when they conflict.

```bash
ls ~/.syntaur/playbooks/*.md 2>/dev/null
```

## Conventions

- Assignment frontmatter is the single source of truth for state. `project` is the containing project slug (`null` for standalone); `type` is a classification validated against `config.md` `types.definitions` when present.
- Slugs are lowercase, hyphen-separated. For standalone assignments the folder is named by UUID; `slug` is display-only.
- Update acceptance criteria checkboxes as work lands, not only at the end.
- Append milestones to `progress.md` — do NOT add a `## Progress` section to `assignment.md` (v2.0 moved progress to its own file).
- `## Todos` in `assignment.md` is an informal markdown checklist. Items may be simple tasks or markdown links to plan files. When a plan is superseded, mark the old todo as `- [x] ~~Execute [plan](./plan.md)~~ (superseded by plan-v2)` — never delete. `## Todos` is also the landing spot for cross-assignment `syntaur request` entries.
- Record questions / notes / feedback via `syntaur comment` — never edit `comments.md` directly. Do NOT set status to `blocked` just because there is an open question; block only for a real external dependency with a `--reason`.
- Write `handoff.md` entries (via `complete-assignment`) with enough context for another agent or human to continue cleanly **across tickets**. Write `sessions/<sid>/summary.md` (via `/save-session-summary`) for mid-assignment continuity within the same assignment across sessions of the same agent. Do not mix the two. Record decisions in `decision-record.md` with Status / Context / Decision / Consequences — downstream dependents auto-load these during grab.
- Commit frequently with messages referencing the assignment slug.

## References

For the full directory structure, lifecycle state table, and detailed file ownership rules, read:

- `references/protocol-summary.md`
- `references/file-ownership.md`
