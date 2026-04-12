---
name: syntaur-protocol
description: >-
  Use when the user mentions Syntaur, missions, assignments, files under ~/.syntaur/,
  assignment.md, plan.md, handoff.md, .syntaur/context.json, lifecycle states, or
  write boundaries. Core protocol knowledge for any AI agent working within Syntaur.
license: MIT
metadata:
  author: prong-horn
  version: "1.0.0"
---

# Syntaur Protocol

You are working within the Syntaur protocol — a coordination system for AI agents built on markdown files. Follow these rules at all times.

## Write Boundary Rules

Respect file ownership boundaries.

### Files you may write

1. Your assignment folder only:
   - `assignment.md`
   - `plan.md`
   - `scratchpad.md`
   - `handoff.md`
   - `decision-record.md`
2. Mission-level shared files:
   - `~/.syntaur/missions/<mission>/resources/<slug>.md`
   - `~/.syntaur/missions/<mission>/memories/<slug>.md`
3. Workspace files inside the assignment's configured workspace root
4. `.syntaur/context.json` in the current working directory

### Files you must never write

1. `mission.md`, `agent.md`, `claude.md` — human-authored, read-only
2. `manifest.md` — derived, rebuilt by tooling
3. Any file prefixed with `_` (`_index-*.md`, `_status.md`) — derived
4. Other agents' assignment folders
5. Anything outside the current workspace boundary

## Current Assignment Context

If `.syntaur/context.json` exists in the current working directory, read it before making changes. Use it to determine:

- `missionSlug`
- `assignmentSlug`
- `missionDir`
- `assignmentDir`
- `workspaceRoot`
- `sessionId` if present

## Required Reading Order

When you are working on an existing assignment, read these in order:

1. `<missionDir>/manifest.md`
2. `<missionDir>/agent.md`
3. `<missionDir>/mission.md`
4. `<missionDir>/claude.md` if it exists
5. `<assignmentDir>/assignment.md`
6. `<assignmentDir>/plan.md`
7. `<assignmentDir>/handoff.md`

## Lifecycle Commands

Use the `syntaur` CLI for state transitions:

- `syntaur assign <slug> --agent <name> --mission <mission>`
- `syntaur start <slug> --mission <mission>` — pending → in_progress
- `syntaur review <slug> --mission <mission>` — in_progress → review
- `syntaur complete <slug> --mission <mission>` — in_progress/review → completed
- `syntaur block <slug> --mission <mission> --reason <text>` — block an assignment
- `syntaur unblock <slug> --mission <mission>` — unblock
- `syntaur fail <slug> --mission <mission>` — mark as failed

## Playbooks

Playbooks are user-defined behavioral rules stored in `~/.syntaur/playbooks/`. Each playbook is a markdown file with imperative rules that agents must follow. When you begin work on any assignment, read all playbook files and follow their directives. Playbooks take precedence over default conventions when they conflict.

```bash
ls ~/.syntaur/playbooks/*.md 2>/dev/null
```

## Conventions

- Assignment frontmatter is the single source of truth for all assignment state.
- Slugs are lowercase and hyphen-separated.
- Update acceptance criteria checkboxes as work lands, not only at the end.
- Keep the `## Progress` section in `assignment.md` current after meaningful milestones.
- Write handoffs with enough context for another agent or human to continue cleanly.
- Commit frequently with messages referencing the assignment slug.

## References

For the full directory structure, lifecycle state table, and detailed file ownership rules, read:

- `references/protocol-summary.md`
- `references/file-ownership.md`
