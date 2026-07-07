---
description: Scaffold a custom slash command that matches this project's conventions — interviews for name, purpose, and arguments, then writes .claude/commands/<name>.md.
---

You are the custom-command generator. A slash command is a reusable prompt the
user triggers with `/name` — use it for interactive, user-initiated workflows
(like /setup-project or /autopilot). Interview the user, then write one
well-formed command. Look at existing `.claude/commands/*.md` first for tone and
structure.

## Step 1 — Interview (use the AskUserQuestion tool where it helps)
Gather:
- **Name** — short kebab-case (becomes the filename and the `/name` invocation).
- **Purpose / description** — what running the command does. This becomes the
  `description:` frontmatter shown in the command list. Keep it one clear line.
- **Arguments?** — does it take input (e.g. `/autopilot on`)? If so, the command
  body should reference `$ARGUMENTS` and explain how to interpret each value.
- **Behavior** — the actual instructions Claude follows when the command runs:
  an ordered procedure, any questions to ask the user (via AskUserQuestion), and
  guardrails (confirm before irreversible or outward-facing actions).
- **Scope** — ALWAYS ask: project-level (`./.claude/commands/`) or user-level
  (`~/.claude/commands/`, i.e. C:\Users\rasmu\.claude\commands\). Do not assume.

## Step 2 — Write the file
Write `<scope>/.claude/commands/<name>.md` with YAML frontmatter (`description`)
followed by the prompt body written as instructions TO Claude (second person:
"You are...", "Do X, then Y"). If it takes input, include the
`Argument: $ARGUMENTS` line and interpretation logic. Do not overwrite an
existing command of the same name without confirming.

## Step 3 — Report
Show the created path and the exact invocation (e.g. `/deploy prod`). Note that
project commands are available only when working under that project's directory,
and offer to promote it to user-level if they want it everywhere.
