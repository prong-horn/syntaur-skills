---
name: create-assignment
description: >-
  Create a new Syntaur assignment within a mission (or as a one-off). Use when the user
  wants to add a task, create an assignment, or break down work within a Syntaur mission.
license: MIT
metadata:
  author: prong-horn
  version: "1.0.0"
---

# Create Assignment

Create a new assignment within a Syntaur mission.

## Input

Expects arguments from the user:

- First (required): the assignment title (e.g., `"Add login endpoint"`)
- `--mission <slug>` (required unless `--one-off`): the mission to add the assignment to
- `--one-off` (optional): create a standalone mission+assignment in one step
- `--slug <slug>` (optional): override the auto-generated assignment slug
- `--priority <level>` (optional): `low`, `medium` (default), `high`, or `critical`
- `--depends-on <slug[,slug...]>` (optional): comma-separated list of assignment slugs this depends on
- `--dir <path>` (optional): override the default mission directory

If no title was provided, ask the user what the assignment should be called.

If neither `--mission` nor `--one-off` was provided, check if there is an active assignment context in `.syntaur/context.json`. If present, default `--mission` to that context's `missionSlug` and confirm with the user: "Add this assignment to mission `<missionSlug>`?"

If there is no active context and no mission flag, ask the user which mission to add it to, or whether it should be a one-off.

## Step 1: Run the CLI

Build the command from the parsed arguments:

```bash
syntaur create-assignment "<title>" --mission <slug> [--slug <slug>] [--priority <level>] [--depends-on <slugs>] [--dir <path>]
```

Or for one-off:

```bash
syntaur create-assignment "<title>" --one-off [--slug <slug>] [--priority <level>] [--dir <path>]
```

If the command fails (e.g., mission not found, slug collision), report the error and suggest fixes.

## Step 2: Read the Created Assignment

After successful creation, extract the assignment slug and directory from the CLI output. Read the generated `assignment.md`:

```bash
cat ~/.syntaur/missions/<mission-slug>/assignments/<assignment-slug>/assignment.md
```

## Step 3: Guide Next Steps

Tell the user:
- The assignment was created with its slug, priority, and location
- Files created: `assignment.md`, `plan.md`, `scratchpad.md`, `handoff.md`, `decision-record.md`
- Suggest they edit `assignment.md` to fill in the objective, acceptance criteria, and context
- If dependencies were set, note them
- Suggest running `grab-assignment` to claim and start working on it
