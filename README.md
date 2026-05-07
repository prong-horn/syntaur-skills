# syntaur-skills (archived)

> **This repo is archived.** Skills moved into the main [`prong-horn/syntaur`](https://github.com/prong-horn/syntaur) repo as of `syntaur@0.8.0` (May 2026).

## Where to find the skills now

[`prong-horn/syntaur/skills/`](https://github.com/prong-horn/syntaur/tree/main/skills)

11 skills ship from there:

- `syntaur-protocol`
- `grab-assignment`
- `plan-assignment`
- `complete-assignment`
- `create-assignment`
- `create-project`
- `manage-statuses`
- `clear-assignment`
- `save-session-summary`
- `track-session` (Claude Code agent session registration)
- `track-server` (tmux dev-server tracking; was previously named `track-session` here for Codex — renamed in 0.8.0 to avoid collision)

## How to install

### Cross-agent (recommended)

```bash
npx skills add prong-horn/syntaur
```

Works for Claude Code, Codex, Cursor, OpenCode, Gemini CLI, Cline, Copilot, and ~45 other agents via [skills.sh](https://skills.sh).

### Claude Code plugin marketplace

Install the syntaur plugin via Claude Code's `/plugin` UI, or run `syntaur install-plugin --enable`.

### syntaur CLI (full feature set)

```bash
npm install -g syntaur@latest
```

## Why was this repo retired?

When the syntaur project was small, the skills lived in their own repo and were vendored into the main syntaur repo as a git submodule. As the project matured this caused real friction: two repos to keep in lockstep, duplicate skill copies for Claude users (vendored + platform), no clean cross-agent install story, and a recurring "plugin invisible to Claude" bug from the install flow.

`syntaur@0.8.0` collapses everything to a single canonical `<repo>/skills/` directory inside the main repo, which is now `npx skills add`-compatible directly. See the [v0.8.0 release notes](https://github.com/prong-horn/syntaur/releases/tag/v0.8.0) for the full design.

## Historical content

Git history of skills authored before 0.8.0 is preserved on the `main` branch of this archived repo. The same skills (with `manage-statuses`, `clear-assignment`, `save-session-summary` added and `track-server` renamed) live at `prong-horn/syntaur/skills/` going forward.
