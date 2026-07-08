---
description: Toggle autopilot (auto-edit) mode on/off — let Claude apply edits without asking, while hard guardrails still block push/release. Usage: /autopilot [on|off|status]
---

Argument (may be empty): $ARGUMENTS

Autopilot lets Claude work without prompting on every file edit, while the
`deny` guardrails in `.claude/settings.json` still hard-block whichever risky
actions this project configured (typically some of git push, git tag, npm
publish, gh release, repo delete — the exact set was chosen at setup time). Those
guardrails are NOT part of the toggle — they always apply. Autopilot also
**never covers Bash**: shell commands still prompt even when it's ON — it only
auto-accepts file edits.

## What to do
1. Read `.claude/settings.local.json` (create it if missing — this file is
   personal and should be gitignored, so mode preference doesn't get committed).
2. Interpret `$ARGUMENTS`:
   - `on`  → set `"permissions": { "defaultMode": "acceptEdits" }`
   - `off` → set `"permissions": { "defaultMode": "default" }`
   - `status` or empty → just report the current `defaultMode` and the active
     deny guardrails; make no changes.
3. Preserve any other keys already in `settings.local.json` — only change
   `permissions.defaultMode`.
4. Before enabling, sanity-check that `.claude/settings.json` still contains a
   `deny` list. If it's empty or missing, warn the user and offer to restore the
   configured guardrails before turning autopilot on — never enable auto-edit
   without guardrails in place.

## After changing
- Confirm the new state in one line, e.g. "Autopilot ON — file edits auto-apply;
  Bash still prompts; push/release still blocked."
- Note that a mode change takes full effect on the next session start; the deny
  guardrails apply immediately regardless.
- Remind the user that autopilot auto-accepts **file edits only** — Bash/shell
  commands still prompt every time.
- Remind the user they can also flip edit-acceptance live in the CLI with
  Shift+Tab, but the deny list is what actually enforces the restrictions.
