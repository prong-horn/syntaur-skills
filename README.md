# syntaur-skills

Agent-agnostic skills for the [Syntaur](https://github.com/prong-horn/syntaur) coordination protocol. Works with any AI coding agent — Claude Code, Cursor, Codex, OpenCode, and more.

## Prerequisites

Install the Syntaur CLI:

```bash
npm install -g syntaur
syntaur setup
```

## Install

```bash
npx skills add prong-horn/syntaur-skills
```

Or install a specific skill:

```bash
npx skills add prong-horn/syntaur-skills --skill syntaur-protocol
```

## Skills

| Skill | Description |
|-------|-------------|
| `syntaur-protocol` | Core protocol knowledge — write boundaries, lifecycle states, conventions. Auto-activates when working with Syntaur files. |
| `grab-assignment` | Discover and claim a pending assignment from a project. Sets up working context. |
| `plan-assignment` | Create a detailed implementation plan for the current assignment. |
| `complete-assignment` | Write a handoff and transition an assignment to review or completed. |
| `create-project` | Create a new project with full scaffolding (manifest + indexes + resources + memories). |
| `create-assignment` | Create a new assignment within a project (or as a standalone one-off at `~/.syntaur/assignments/<uuid>/`). |

## How it works

Syntaur is a coordination protocol for AI agents built on markdown files. Projects contain assignments, assignments have lifecycle states, and agents follow write boundary rules about which files they can modify. Everything lives under `~/.syntaur/` and is managed via the `syntaur` CLI.

These skills teach your agent the protocol so it can participate in Syntaur-coordinated work — regardless of which coding agent you use.

For platform-specific integrations (hooks, commands, sandbox enforcement), see the [Syntaur plugin system](https://github.com/prong-horn/syntaur).
