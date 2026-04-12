---
name: complete-assignment
description: >-
  Write a handoff and transition the current Syntaur assignment to review or completed.
  Use when the user wants to finish an assignment, write a handoff, or submit work for review.
license: MIT
metadata:
  author: prong-horn
  version: "1.0.0"
---

# Complete Assignment

Write a handoff for your current Syntaur assignment and transition it to `review` or `completed`.

## Input

Optional: the user may pass `--complete` to transition directly to `completed` instead of `review`. However, `--complete` is only allowed if ALL acceptance criteria are met. If any criteria are unmet, always transition to `review` regardless of the flag, and inform the user why.

## Step 1: Load Context

Read `.syntaur/context.json` from the current working directory.

If the file does not exist, tell the user: "No active assignment found. Run `grab-assignment` first."

Extract: `missionSlug`, `assignmentSlug`, `assignmentDir`, `missionDir`.

## Step 2: Load Playbooks

Read all playbook files from `~/.syntaur/playbooks/`:

```bash
ls ~/.syntaur/playbooks/*.md 2>/dev/null
```

Verify your work complies with their rules. If any playbook has completion-related rules (e.g., "run tests before done"), follow them before proceeding.

## Step 3: Verify Acceptance Criteria

Read `<assignmentDir>/assignment.md` and find the `## Acceptance Criteria` section.

Review each criterion (checkbox item). For each:
- If you believe it is met, note why (what was implemented, where)
- If it is NOT met, flag it clearly

If any criteria are not met, warn the user: "The following acceptance criteria are not yet met: [list]. Do you want to proceed with the handoff anyway?"

If the user says no, stop.

## Step 4: Write Handoff Entry

Read `<assignmentDir>/handoff.md` to see its current content and frontmatter.

Append a new handoff entry to the markdown body. Read the current `handoffCount` from the frontmatter and use `handoffCount + 1` as the entry number. The entry must follow this format:

```markdown
## Handoff <N>: <ISO 8601 timestamp>

**From:** <your-agent-name>
**To:** human
**Reason:** <Why this handoff is happening, e.g., "Assignment complete, handing off for review.">

### Summary
<One paragraph summarizing what was accomplished and what remains>

### Current State
- <What is working>
- <What is not working or partially done>
- <Acceptance criteria status: N of M met>

### Next Steps
- <Recommended next actions for the reviewer or next agent>

### Important Context
- <Anything the next agent/human needs that is not in the assignment or plan>
```

Also update the handoff.md frontmatter: set `updated` to the current timestamp and increment the `handoffCount` by 1.

## Step 5: Update Acceptance Criteria Checkboxes

In `<assignmentDir>/assignment.md`, update the acceptance criteria checkboxes to reflect the current state. Check off criteria that were met (change `- [ ]` to `- [x]`).

Ideally, criteria should have been checked off incrementally during implementation. If they are already checked, verify they are still accurate.

## Step 6: Close Session (optional)

If `.syntaur/context.json` includes a `sessionId` and the Syntaur dashboard is running, mark the session as completed:

```bash
curl -s -X PATCH "http://localhost:$(cat ~/.syntaur/dashboard-port 2>/dev/null || echo 4800)/api/agent-sessions/<session-id>/status" \
  -H "Content-Type: application/json" \
  -d '{"status":"completed","missionSlug":"<mission-slug>"}'
```

If this fails (e.g., dashboard not running), it is non-critical — the session will be reconciled automatically.

## Step 7: Transition Assignment State

If the user requested `--complete` and all criteria are met:

```bash
syntaur complete <assignment-slug> --mission <mission-slug>
```

Otherwise, transition to review:

```bash
syntaur review <assignment-slug> --mission <mission-slug>
```

If the command fails, report the error. Common failures:
- Assignment is not in `in_progress` status
- Mission not found

## Step 8: Clean Up Context

Delete the context file:

```bash
rm .syntaur/context.json
```

## Step 9: Report to User

Summarize:
- Assignment slug and title
- New status (review or completed)
- Number of acceptance criteria met vs total
- If transitioned to `review`, a human reviewer will check the work. If any criteria were unmet, they may send it back to `in_progress`.
