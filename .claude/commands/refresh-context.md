---
description: Re-scan the project and update CLAUDE.md incrementally — refresh structure, commands, and conventions without clobbering hand-written content. Like /init but respectful of what's already there.
---

You are refreshing this project's CLAUDE.md so it doesn't rot as the codebase
grows. This is INCREMENTAL — you preserve the user's hand-written guidance and
only update what has actually drifted.

## Step 1 — Read what exists
Read the current `CLAUDE.md` in full. Note which sections are hand-authored
opinion (conventions, architecture stances, "why" notes) versus auto-derivable
facts (file layout, build/test commands, dependencies).

## Step 2 — Scan the project for drift
Inspect the real project (scoped to THIS project directory only — ignore any
parent working folders):
- Directory structure and key modules.
- Scripts / commands from the manifest (`package.json`, `Makefile`, etc.).
- New tooling: `.claude/` agents, skills, commands, hooks; Docker; CI.
- Dependencies and language/framework changes.
Compare against what CLAUDE.md currently claims.

## Step 3 — Propose a diff, then apply
- Update only the auto-derivable sections that are stale (layout, commands, tooling).
- NEVER overwrite hand-written conventions, architecture principles, or "why"
  notes. If one of those looks contradicted by the code, FLAG it for the user
  instead of silently rewriting it.
- Keep CLAUDE.md lean — prefer references over inlined walls of text; if a
  section has outgrown ~30 lines, suggest splitting it to a docs/ file.
- Show the user a short summary of what you changed and what you flagged.

## Guardrails
Additive and surgical by default. When unsure whether something is hand-written
intent or stale fact, ask rather than overwrite.
