---
name: grab-assignment
description: >-
  Discover and claim a pending Syntaur assignment from a mission. Use when the user
  wants to start working on a Syntaur assignment, claim a task, or set up their
  working context for a mission.
license: MIT
metadata:
  author: prong-horn
  version: "1.0.0"
---

# Grab Assignment

Claim a pending Syntaur assignment and set up the current workspace.

## Input

Expects two arguments from the user:

- First (required): the mission slug (e.g., `build-auth-system`)
- Second (optional): a specific assignment slug to grab. If omitted, list pending assignments and pick one.

## Pre-flight Check

1. Check if `.syntaur/context.json` already exists in the current working directory.
   - If it exists, read it and warn the user: "You already have an active assignment: `<assignmentSlug>` in mission `<missionSlug>`. Grabbing a new assignment will replace this context. Proceed?"
   - If the user says no, stop.

## Step 1: Discover the Mission

Read the mission entry files:

- `~/.syntaur/missions/<mission-slug>/manifest.md`
- `~/.syntaur/missions/<mission-slug>/mission.md`
- `~/.syntaur/missions/<mission-slug>/agent.md`
- `~/.syntaur/missions/<mission-slug>/claude.md` if it exists

Note the `workspace` field in `mission.md` frontmatter if present. This indicates which project/codebase grouping the mission belongs to.

## Step 2: Find Pending Assignments

List assignment directories and check their status:

```bash
grep -l "status: pending" ~/.syntaur/missions/<mission-slug>/assignments/*/assignment.md
```

If no pending assignments exist, tell the user and stop.

If the user specified an assignment slug, verify it exists and is pending. If it is not pending, report its current status and stop.

If no specific assignment was requested, present the list of pending assignments with their titles and priorities, and ask the user which one to grab. If there is only one pending assignment, grab it automatically.

## Step 3: Claim the Assignment

Run the Syntaur CLI commands to assign and start the assignment:

```bash
syntaur assign <assignment-slug> --agent <your-agent-name> --mission <mission-slug>
```

Then:

```bash
syntaur start <assignment-slug> --mission <mission-slug>
```

> **Agent identity:** Use an identifier for your agent platform — e.g., `claude`, `cursor`, `codex`, `opencode`, etc.

If either command fails, report the error and stop. Common failures:
- Assignment has unmet dependencies (cannot start until dependencies are `completed`)
- Assignment is not in `pending` status
- Mission not found

## Step 4: Read Assignment Context and Set Workspace

After successfully starting the assignment, read the full assignment file:

```bash
cat ~/.syntaur/missions/<mission-slug>/assignments/<assignment-slug>/assignment.md
```

Extract from the frontmatter:
- `title` — the assignment title
- `workspace.repository` — the code repository path (may be null)
- `workspace.worktreePath` — the worktree path (may be null)
- `workspace.branch` — the branch name (may be null)
- `dependsOn` — list of dependency slugs
- `priority` — priority level

If `workspace.repository` and `workspace.worktreePath` are both null, set them to the current working directory. This is critical because write boundaries use the workspace path to determine which files the agent is allowed to edit. Update the assignment.md frontmatter:

```yaml
workspace:
  repository: /absolute/path/to/cwd
  worktreePath: /absolute/path/to/cwd
```

## Step 5: Create Context File

Ensure the `.syntaur` directory exists in the current working directory:

```bash
mkdir -p .syntaur
```

Write `.syntaur/context.json` with:

```json
{
  "missionSlug": "<mission-slug>",
  "assignmentSlug": "<assignment-slug>",
  "missionDir": "/absolute/path/to/.syntaur/missions/<mission-slug>",
  "assignmentDir": "/absolute/path/to/.syntaur/missions/<mission-slug>/assignments/<assignment-slug>",
  "workspaceRoot": "<workspace path or current working directory>",
  "title": "<assignment title>",
  "branch": "<workspace.branch or null>",
  "grabbedAt": "<ISO 8601 timestamp>",
  "sessionId": "<uuid>"
}
```

Use absolute paths (expand `~` to the actual home directory). If `workspace.repository` is a remote URL (starts with `https://`), set `workspaceRoot` to the current working directory instead.

## Step 6: Register Agent Session

Generate a UUID for this session and register it:

```bash
syntaur track-session --mission <missionSlug> --assignment <assignmentSlug> --agent <your-agent-name> --session-id <uuid> --path $(pwd)
```

Store the session ID in `.syntaur/context.json`.

## Step 7: Load Playbooks

Read all playbook files from `~/.syntaur/playbooks/` and treat their content as active behavioral rules:

```bash
ls ~/.syntaur/playbooks/*.md 2>/dev/null
```

For each file found, read it and follow its directives. Playbooks take precedence over default conventions when they conflict.

## Step 8: Report to User

Summarize:
- Which assignment was grabbed
- The objective (first paragraph from assignment.md body)
- The acceptance criteria (the checkbox list)
- The workspace path (if set)
- Suggest next step: create an implementation plan with `plan-assignment`
