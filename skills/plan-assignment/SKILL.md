---
name: plan-assignment
description: >-
  Create a detailed implementation plan for the current Syntaur assignment. Use when
  the user wants to plan their work, write a plan.md, or design an approach for an
  active assignment.
license: MIT
metadata:
  author: prong-horn
  version: "1.0.0"
---

# Plan Assignment

Create a detailed implementation plan for your current Syntaur assignment.

## Input

Optional: the user may provide focus areas or notes to guide the plan.

## Step 1: Load Context

Read `.syntaur/context.json` from the current working directory.

If the file does not exist, tell the user: "No active assignment found. Run `grab-assignment` first to claim an assignment."

Extract:
- `missionSlug` — the mission slug
- `assignmentSlug` — the assignment slug
- `assignmentDir` — absolute path to the assignment folder
- `missionDir` — absolute path to the mission folder
- `workspaceRoot` — absolute path to the workspace (may be null)

## Step 2: Load Playbooks

Read all playbook files from `~/.syntaur/playbooks/`:

```bash
ls ~/.syntaur/playbooks/*.md 2>/dev/null
```

For each file found, read it and follow its directives. Playbooks may contain rules about planning conventions, required steps, or quality expectations.

## Step 3: Read Assignment Details

Read the following files to understand the assignment:

1. `<assignmentDir>/assignment.md` — extract the objective, acceptance criteria, context section, and any Q&A
2. `<missionDir>/agent.md` — extract conventions and boundaries
3. `<missionDir>/claude.md` if it exists — extract platform-specific instructions
4. `<missionDir>/mission.md` — extract the mission goal for broader context

If the assignment has dependencies (`dependsOn` in frontmatter), read the handoff.md from each dependency's assignment folder for integration context:
- `<missionDir>/assignments/<dep-slug>/handoff.md`

## Step 4: Explore Workspace (if set)

If `workspaceRoot` is not null:

1. Check if the workspace directory exists
2. Explore the codebase structure to understand what exists:
   - Find key files (e.g., `**/*.ts`, `**/package.json`, `**/*.md`)
   - Search for relevant patterns mentioned in the assignment
   - Read key files like `package.json`, `tsconfig.json`, or entry points
3. Note any existing patterns, conventions, or architecture you discover

If `workspaceRoot` is null, skip this step and note in the plan that no workspace is configured.

## Step 5: Write the Plan

Read the existing `<assignmentDir>/plan.md` to see its current frontmatter structure. Preserve the YAML frontmatter fields (`assignment`, `status`, `created`, `updated`) and update the `updated` timestamp. Change the `status` field from `draft` to `in_progress` if it is still `draft`.

Replace the markdown body with a detailed implementation plan containing:

1. **Overview** — one paragraph summarizing the approach
2. **Tasks** — numbered list of implementation tasks, each with:
   - Description of what to do
   - Files to create or modify (with paths)
   - Dependencies on other tasks
   - Estimated complexity (low/medium/high)
3. **Acceptance Criteria Mapping** — for each criterion from assignment.md, which task(s) address it
4. **Risks and Open Questions** — anything that might block or complicate implementation
5. **Testing Strategy** — how to verify the implementation works

Update `<assignmentDir>/plan.md`, preserving the existing frontmatter and replacing only the body content.

## Step 6: Report to User

After writing the plan:
1. Summarize the plan (number of tasks, key decisions)
2. Note any open questions or risks that need human input
3. Suggest next step: begin implementing the first task, or complete the assignment when all work is done

**Recordkeeping reminder for implementation:**
- Check off acceptance criteria in `assignment.md` as each one is completed, not in a batch at the end
- Update the `## Progress` section in `assignment.md` after each meaningful milestone
- The assignment file is a live document — it should reflect current state at all times
