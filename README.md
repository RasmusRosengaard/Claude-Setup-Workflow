# Claude Setup Workflow

A collection of reusable **[Claude Code](https://claude.com/claude-code) slash-command wizards** for bootstrapping and maintaining projects. Drop them into your user-level Claude Code config and they work in every project you open.

The centerpiece is **`/setup-project`** — an interactive wizard that profiles your project, then scaffolds exactly the tooling you ask for (agents, hooks, Docker, CI, GitHub, conventions), tailored to your stack.

## What's included

| Command | What it does |
|---------|--------------|
| `/setup-project` | Interactive project-bootstrap wizard: surveys the repo, profiles purpose/KPI/stack/models, then scaffolds only what you pick — with stack-aware suggestions. |
| `/new-agent` | Scaffold a custom subagent (name, purpose, tools, model) with correct frontmatter. |
| `/new-skill` | Scaffold an on-demand skill (`SKILL.md`) with a specific trigger and steps. |
| `/new-command` | Scaffold a custom slash command (`$ARGUMENTS` handling, prompt body). |
| `/autopilot` | Toggle auto-edit mode on/off — let Claude apply edits without prompting, while hard guardrails always block push/tag/publish/release. |
| `/refresh-context` | Incrementally update `CLAUDE.md` as the project grows, without clobbering hand-written guidance. |
| `/handoff` | Dump current work state to a markdown file so a fresh session (or teammate) can resume cleanly. |

## Install

These are **user-level** commands, so they're available in every project.

```bash
git clone https://github.com/RasmusRosengaard/Claude-Setup-Workflow.git
# macOS / Linux
cp Claude-Setup-Workflow/.claude/commands/*.md ~/.claude/commands/
# Windows (PowerShell)
Copy-Item Claude-Setup-Workflow/.claude/commands/*.md $HOME/.claude/commands/
```

Restart Claude Code (or run `/hooks` once) and type `/setup-project` in any project.

> Prefer project-scoped commands? Copy them into a project's `.claude/commands/` instead of `~/.claude/commands/` — they'll only appear inside that project.

## How it works

Everything here is a **slash command** — a markdown file with `description` frontmatter and a prompt body that instructs Claude. When you type `/setup-project`, Claude follows those instructions: it interviews you with multiple-choice questions, then creates files. Nothing is scaffolded by default — you opt in, and the wizard reconciles both ways (it also removes items you uncheck that it created earlier).

The wizard can install **guardrail hooks** — deterministic, harness-run checks (not prompt instructions) that, for example, block edits to `.env` / `.git` / `settings.json` even when autopilot is on. Generated hooks use Node (cross-platform, no `jq` needed).

## Requirements

- **Claude Code** (CLI, desktop, or IDE extension)
- **Node.js** — only if you use the generated guardrail hooks
- **git** / GitHub — only for the optional GitHub-link step in `/setup-project`

## License

MIT — see `LICENSE` (add one if you want others to reuse freely).
