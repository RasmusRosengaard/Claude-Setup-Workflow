---
description: Interactively set up a project — interview the user on what they want, then scaffold agents, skills, Docker, CI, and a GitHub link, and update CLAUDE.md.
---

You are running the project-setup wizard. Your job is to interview the user about
what they want, then scaffold it. Work in the current project directory.

## Step 1 — Survey what already exists
Before asking anything, quietly inspect the project so you don't propose
duplicates or clobber existing config:
- Is there a `CLAUDE.md`? A `.claude/` dir with agents/skills/commands?
- Is it a git repo? Does it have a GitHub remote (`git remote -v`)?
- Is there a `package.json` / language manifest, `Dockerfile`, `docker-compose.yml`,
  or `.github/workflows/`?
Briefly tell the user what you found (one or two lines).

## Step 2 — Profile the project first (use the AskUserQuestion tool)
Before offering the menu, understand what you're building. Ask about:
- **Purpose** — what is this project / who is it for? (one line is fine)
- **Success metric / KPI** — is there a known goal to optimize for
  (latency, test coverage, uptime, conversion)? Skip if none.
- **Testing** — do they want a test setup, and roughly what kind (unit, e2e, none)?
- **Language / framework** — confirm or choose the stack.
- **Models** — which model should run implementation (the main session), and do
  they want a stronger model for planning/architecture? A common split is a
  cheaper model (e.g. `claude-sonnet-5`) for the main session plus Opus 4.8
  (`claude-opus-4-8`) reserved for heavy reasoning. Explain that Claude Code
  runs one main model per session (switchable with `/model`); the split is
  achieved by pinning a separate **architect subagent** to the stronger model,
  not by auto-swapping mid-session. If they only want one model, skip the split.
Prefer inferring answers from Step 1 (a manifest usually reveals the stack) and
only ask what you couldn't determine. Use these answers to tailor every later
choice — the Docker base image, the CI test command, the CLAUDE.md conventions.

## Step 3 — Ask what to scaffold (use the AskUserQuestion tool)
Present a multi-select menu of setup options. Nothing is scaffolded by default —
the user opts in. Only offer things that are NOT already present, and TAILOR the
menu to the stack identified in Step 2: mark items that clearly fit the stack
with "(recommended)" and pre-select them, and drop options that don't apply. When
an option needs a concrete tool, pick it from the stack — e.g. formatter/test
hooks use prettier + vitest/jest for Node, ruff/black + pytest for Python, gofmt
+ `go test` for Go, rustfmt + `cargo test` for Rust. If the stack has no standard
tool for something, say so rather than scaffolding a no-op. Typical options:
- **Code-reviewer agent** — a subagent in `.claude/agents/` that reviews diffs.
- **Docker + compose** — a `Dockerfile` and `docker-compose.yml` for local dev.
- **GitHub link** — create/link a repo and push (`gh repo create`), add remote.
- **CI pipeline** — a `.github/workflows/` file running lint + tests on PRs.
- **Release skill** — a `.claude/skills/` playbook for cutting releases.
- **Conventions in CLAUDE.md** — architecture principles, style, commands.
- **Autopilot (auto-edit) mode** — let Claude apply edits without prompting,
  with hard guardrails that always block push/tag/publish/release. If chosen,
  write the `deny` list into `.claude/settings.json`, enable `acceptEdits` in
  `.claude/settings.local.json`, and install the `/autopilot` command so the
  user can toggle it on/off later. Ask whether to start it ON or OFF.
- **Model configuration** — set the implementation model in
  `.claude/settings.json` (`"model"`). If they chose a planning/implementation
  split in Step 2, also scaffold an `.claude/agents/architect.md` subagent with
  `model:` set to the stronger model (e.g. Opus 4.8) so heavy reasoning is
  delegated there while the main session stays on the cheaper model.
- **Guardrail hooks** — install event-driven hooks (run by the harness, not
  Claude, so they're deterministic — unlike CLAUDE.md rules). Create
  `.claude/hooks/` with three Node scripts and wire them in `.claude/settings.json`:
  1. **block-protected.js** (PreToolUse on `Edit|Write`) — reads the hook JSON on
     stdin, and if `tool_input.file_path` matches a protected pattern (`.env*`,
     `.claude/settings.json`, anything under `.git/`) prints a JSON
     `hookSpecificOutput.permissionDecision: "deny"` and exits 2 to block it.
     This is the key guardrail — it enforces what a `deny` list can't (file paths)
     and holds even under autopilot. Confirm the protected list with the user.
  2. **format-file.js** (PostToolUse on `Edit|Write`) — runs the project's
     formatter (e.g. `npm run format`) if one exists; no-ops cleanly otherwise.
  3. **run-tests.js** (Stop) — runs the test suite when Claude finishes; no-ops
     until tests exist; surfaces a `systemMessage` on failure.
  Use Node (cross-platform, no `jq` dependency). Wire commands as
  `node "$CLAUDE_PROJECT_DIR/.claude/hooks/<script>.js"` with `"shell": "bash"`.
  Point the format/test hooks at THIS project's real formatter and test runner
  (from Step 2), not a generic `npm run`; if the stack has none, skip that hook
  rather than scaffolding a no-op. PIPE-TEST each script with synthesized hook
  JSON before finishing (feed it a protected path and a normal one; confirm block
  vs allow), then tell the user hooks activate on next session start / `/hooks`.
Let the user pick several. Ask a short follow-up only where a choice needs it
(e.g. public vs private for the GitHub repo) — the stack is already known from Step 2.

The menu is authoritative: treat an option the user leaves **unchecked as an
explicit "no."** If that item already exists from a PREVIOUS run of this wizard
(e.g. a code-reviewer agent you scaffolded earlier, now unchecked), remove it —
show the user the file and confirm the deletion first. Never silently keep
something they just declined, and never touch files the wizard didn't create.

## Step 4 — Scaffold (and reconcile) each item
For each option, add it if selected, remove it if declined:
- Create the files with sensible, project-specific defaults (infer language,
  framework, and package manager from what you found in Step 1 — do not guess
  blindly; ask if truly ambiguous).
- Never overwrite an existing file without showing the user and confirming.
- For the GitHub link, confirm before any network action (repo creation, push).
- Keep CLAUDE.md lean: add a short section per item, reference longer docs
  rather than inlining them.

## Step 5 — Update CLAUDE.md and report
- **Useful commands** — replace any placeholder/example commands with THIS
  project's real ones, derived from the stack (Step 2): the actual package
  manager and scripts (e.g. `pnpm dev`, `cargo test`, `uv run pytest`,
  `make build`). Never leave generic `npm run …` lines that don't exist here.
- **Layout / Tooling** — add or refresh a section so CLAUDE.md accurately lists
  what now exists (agents, skills, hooks, Docker, CI) and drops references to
  things that aren't there.
- Summarize what you created as a file tree, and list any follow-ups the user
  still needs to do by hand (e.g. `gh auth login`, set secrets, install deps).

Guardrails: confirm before outward-facing or hard-to-reverse actions (pushing to
GitHub, deleting files). Prefer editing over recreating. Match the project's
existing style and indentation.
