---
description: Scaffold a custom skill (on-demand playbook) that matches this project's conventions — interviews for name, trigger, and steps, then writes .claude/skills/<name>/SKILL.md.
---

You are the custom-skill generator. A skill is an on-demand playbook Claude
loads only when a task matches its description — use it for specialized,
repeatable workflows (not always-on rules, which belong in CLAUDE.md). Interview
the user, then write one well-formed skill. Look at existing
`.claude/skills/*/SKILL.md` first for tone and structure.

## Step 1 — Interview (use the AskUserQuestion tool where it helps)
Gather:
- **Name** — short kebab-case (becomes the folder name and the `name:` field).
- **Trigger / description** — WHEN this skill should load. This is the single
  most important field: it's what Claude matches on. Make it specific and
  action-oriented ("Use when the user asks to X"), not a vague topic label.
- **Steps** — the actual procedure the skill encodes: an ordered list of what to
  do, with any stop conditions or guardrails. Push for concrete, project-specific
  steps rather than generic advice (generic advice is noise Claude already knows).
- **Bundled files?** — ask whether it needs helper scripts, templates, or
  reference docs alongside SKILL.md. If so, scaffold placeholders in the folder.
- **Scope** — ALWAYS ask: project-level (`./.claude/skills/`) or user-level
  (`~/.claude/skills/`, i.e. C:\Users\rasmu\.claude\skills\). Do not assume.

## Step 2 — Write the file
Write `<scope>/.claude/skills/<name>/SKILL.md` with YAML frontmatter
(`name`, `description`) followed by the playbook: a titled heading, the ordered
steps, and a short notes/guardrails section. Keep it lean — reference longer
material in bundled files rather than inlining walls of text. Do not overwrite an
existing skill of the same name without confirming.

## Step 3 — Report
Show the created path, restate the trigger in one line so the user knows when it
will fire, and note that project skills apply only when working under that
project's directory.
