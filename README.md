# syntaur-skills

Agent-agnostic skills for the [Syntaur](https://github.com/prong-horn/syntaur) coordination protocol. Works with any AI coding agent — Claude Code, Cursor, Codex, OpenCode, and more.

## Prerequisites

Install the Syntaur CLI:

```bash
npm install -g syntaur
syntaur setup
```

## Install

If you use Claude Code or Codex, install the `syntaur` package — these skills ship automatically with its plugin install flow:

```bash
npm install -g syntaur
syntaur install-plugin        # Claude Code → ~/.claude/skills/
syntaur install-codex-plugin  # Codex → ~/.codex/skills/
```

For any other AI coding agent (Cursor, OpenCode, custom runtimes), install the skills directly:

```bash
npx skills add prong-horn/syntaur-skills
```

Or install a specific skill:

```bash
npx skills add prong-horn/syntaur-skills --skill syntaur-protocol
```

> **Note:** If you install `syntaur` AND also run `npx skills add prong-horn/syntaur-skills` on the same machine, you'll get the same skills installed once — they use the same names. The `syntaur` CLI detects existing copies and skips already-current ones.

## Skills

| Skill | Description |
|-------|-------------|
| `syntaur-protocol` | Core protocol knowledge — write boundaries, lifecycle states, conventions. Auto-activates when working with Syntaur files. |
| `grab-assignment` | Discover and claim a pending assignment from a project. Sets up working context. |
| `plan-assignment` | Create a detailed implementation plan for the current assignment. |
| `complete-assignment` | Write a handoff and transition an assignment to review or completed. |
| `clear-assignment` | Drop the active assignment from session context without transitioning lifecycle state. Inverse of `grab-assignment`. |
| `create-project` | Create a new project with full scaffolding (manifest + indexes + resources + memories). |
| `create-assignment` | Create a new assignment within a project (or as a standalone one-off at `~/.syntaur/assignments/<uuid>/`). |
| `manage-statuses` | List / add / rename / remove / reorder custom assignment statuses (and transitions); writes to `~/.syntaur/config.md`. |

## How it works

Syntaur is a coordination protocol for AI agents built on markdown files. Projects contain assignments, assignments have lifecycle states, and agents follow write boundary rules about which files they can modify. Everything lives under `~/.syntaur/` and is managed via the `syntaur` CLI.

These skills teach your agent the protocol so it can participate in Syntaur-coordinated work — regardless of which coding agent you use.

For platform-specific integrations (hooks, commands, sandbox enforcement), see the [Syntaur plugin system](https://github.com/prong-horn/syntaur).
