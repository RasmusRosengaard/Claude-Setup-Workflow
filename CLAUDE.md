# Claude Setup Workflow

A collection of reusable Claude Code slash-command wizards for bootstrapping and
maintaining projects. See `README.md` for the human-facing overview and install.

## What this repo is
Each file in `.claude/commands/` is a self-contained slash command (a markdown
prompt with `description` frontmatter). They're meant to be copied to a user's
`~/.claude/commands/` so they work in every project. This repo has no build step
and no runtime — the "product" is the command prompts themselves.

## Layout
- `.claude/commands/`        — the seven wizard commands (the product)
- `.claude/settings.json`    — example permission guardrails (allow/deny)
- `README.md`                — human-facing docs and install instructions
- `CLAUDE.md`                — this file

## Conventions for editing commands
- Every command starts with YAML frontmatter containing a specific, trigger-
  oriented `description:` — that's how Claude matches and surfaces it.
- Write the body as instructions TO Claude (second person): "You are…", "Ask…",
  "Then…". Commands that take input reference `$ARGUMENTS`.
- Favor specificity over generic advice. A command that says "follow best
  practices" is noise; one that names the exact step, tool, or guardrail is
  signal — the generators deliberately push users toward project-specific detail.
- Confirm before outward-facing or irreversible actions (GitHub push, deletes),
  and ask scope (project vs user level) rather than assuming.

## Keeping copies in sync
Commands may exist both here and in the maintainer's `~/.claude/commands/`. If you
edit one, mirror the change to the other copy.
