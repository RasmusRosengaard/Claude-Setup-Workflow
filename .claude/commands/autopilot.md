---
description: Toggle autopilot (auto-edit) mode on/off — let Claude apply edits without asking, while hard guardrails still block push/release. Usage: /autopilot [on|off|status]
---

Argument (may be empty): $ARGUMENTS

Autopilot lets Claude work without prompting on every file edit, while the
`deny` guardrails in `.claude/settings.json` still hard-block risky actions
(git push, git tag, npm publish, gh release, repo delete). Those guardrails
are NOT part of the toggle — they always apply.

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
4. Before enabling, sanity-check that `.claude/settings.json` still contains the
   deny list (push/tag/publish/release/repo-delete). If it's missing, warn the
   user and offer to restore it before turning autopilot on — never enable
   auto-edit without the guardrails in place.

## After changing
- Confirm the new state in one line, e.g. "Autopilot ON — edits auto-apply;
  push/release still blocked."
- Note that a mode change takes full effect on the next session start; the deny
  guardrails apply immediately regardless.
- Remind the user they can also flip edit-acceptance live in the CLI with
  Shift+Tab, but the deny list is what actually enforces the restrictions.
