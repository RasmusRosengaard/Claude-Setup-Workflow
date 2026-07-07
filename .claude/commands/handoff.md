---
description: Dump the current work state to a markdown file so work can be resumed later (by you, a teammate, or a fresh session). Usage: /handoff [optional-filename]
---

Argument (may be empty): $ARGUMENTS

You are writing a handoff document — a snapshot of the current state of work so
someone (possibly a fresh Claude session with no memory of this conversation)
can pick it up cleanly.

## Step 1 — Gather state
- Current goal: what is being worked on and why.
- Progress: what's done, what's in flight, what's not started.
- Current diff: run `git status` and `git diff` (and staged changes) to capture
  uncommitted work; summarize it.
- Key decisions made this session and their rationale.
- Known issues, blockers, or things deliberately left undone.
- Exact next steps — concrete, actionable, in order.

## Step 2 — Write the file
Default path: `HANDOFF.md` in the project root, unless `$ARGUMENTS` gives a name
or path. Structure it:
- **Goal** — one paragraph.
- **State** — done / in-progress / todo.
- **Uncommitted changes** — summary of the working diff.
- **Decisions & rationale** — bullets.
- **Blockers / open questions.**
- **Next steps** — numbered, actionable.
- **How to verify** — commands to run to confirm things work.
Write for a reader with zero context from this conversation. Be concrete: name
files, commands, and paths.

## Step 3 — Report
Show the path written and a one-line summary. Do not commit or push it unless the
user asks.
