---
description: Interactively set up a project — interview the user on what they want, then scaffold agents, skills, Docker, CI, and a GitHub link, and sync CLAUDE.md and the README.
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
- **Does `package.json` declare `"type": "module"`?** Record this — it decides
  whether generated hooks must be `.cjs` (see Step 3). Any modern Vite/Next
  starter is ESM by default.
- **Is the directory empty or non-empty?** If it already has files, note which —
  you'll need this to decide between tooling-up vs. scaffolding-new in Step 2, and
  to protect existing files in Step 4.
- **Tooling availability** — if a GitHub link looks likely, check the tools NOW,
  not at push time: run `gh --version` and `gh auth status`. If `gh` is missing or
  unauthenticated, remember it and surface the requirement up front in Step 3 so
  you never promise a push you can't perform.
Briefly tell the user what you found (one or two lines).

## Step 2 — Profile the project first (use the AskUserQuestion tool)
Before offering the menu, understand what you're building.

**First, resolve intent — do not infer it from pre-existing files.** A non-empty
directory does NOT mean "add tooling to this codebase" — the files present may be
unrelated, or may be `.claude/` tooling from a prior run. Ask explicitly:
> Are we tooling up the **existing** project in this directory, or **scaffolding a
> new app** here (and adding tooling around it)?

Only after they answer, ask the rest (infer what you can from Step 1; only ask
what you couldn't determine):
- **Purpose** — what is this project / who is it for? (one line is fine)
- **Success metric / KPI** — is there a known goal to optimize for
  (latency, test coverage, uptime, conversion)? Skip if none.
- **Testing** — do they want a test setup, and roughly what kind (unit, e2e, none)?
- **Language / framework** — confirm or choose the stack. If they chose
  "scaffold a new app," this is the starter to generate (e.g. `npm create vite`).
- **Model orchestration** — this is important; treat it as its own mini-interview,
  not an afterthought. Ask which model plays each role, offering a sensible default
  for each and letting the user override. Walk the roles explicitly:
  - **This setup session** — which model should run the wizard itself? The
    interview and scaffolding benefit from a strong model (e.g. Opus 4.8). You
    can't change the model mid-session, so if they're on a weaker one, tell them
    to `/model` up now and re-run, or proceed and note it.
  - **Implementation (main session)** — the everyday coding model, pinned via
    `"model"` in `.claude/settings.json`. A cheaper model (e.g. `claude-sonnet-5`)
    is a common default here.
  - **Planning / architecture (thinking)** — heavy reasoning. If they want a
    stronger model for this (e.g. Opus 4.8), it's delivered as an **architect
    subagent** pinned to that model, since Claude Code runs one main model per
    session (switchable with `/model`) and does NOT auto-swap mid-session.
  - **Code review** — if they scaffold a code-reviewer agent, ask which model it
    runs on (a strong model catches more).
  Present the common split plainly: cheap model for the main session + Opus 4.8
  reserved for planning/review via pinned subagents. If they only want one model
  everywhere, skip the subagents. Note that a pinned main **model only takes
  effect on the next session start**; subagent `model:` pins apply as soon as the
  agent is invoked. Carry these choices into the Step 3 "Model configuration" and
  agent options.
Use these answers to tailor every later choice — the Docker base image, the CI
test command, the CLAUDE.md conventions.

## Step 2b — If scaffolding a NEW app into a non-empty dir (safe-merge)
Only relevant when Step 2 said "scaffold a new app" AND the directory already
holds files you must not lose (`CLAUDE.md`, `README.md`, `.claude/`, `.git/`).
Generators like `npm create vite@latest .` refuse to run in, or clobber, a
non-empty dir. Use the safe-merge pattern:
1. Run the generator into a fresh temp dir (e.g. `../<name>-scaffold` or a
   scratch path), not the project root.
2. Show the user the generated file list. Copy over only the files that do NOT
   collide with protected ones. For genuine collisions (e.g. the starter ships
   its own `README.md`), show a diff and let the user choose per file — never
   silently overwrite `CLAUDE.md`, `README.md`, or anything under `.claude/`.
3. Merge, don't replace, shared config: fold the starter's `package.json`
   scripts/deps into any existing manifest rather than overwriting it.
4. Delete the temp dir when done.
If the directory is empty, just scaffold in place — no merge needed.

## Step 3 — Ask what to scaffold (use the AskUserQuestion tool)
Present a multi-select menu of setup options. Nothing is scaffolded by default —
the user opts in. Only offer things that are NOT already present, and TAILOR the
menu to the stack identified in Step 2: mark items that clearly fit the stack
with "(recommended)" and pre-select them, and drop options that don't apply. When
an option needs a concrete tool, pick it from the stack — e.g. formatter/test
hooks use prettier + vitest/jest for Node, ruff/black + pytest for Python, gofmt
+ `go test` for Go, rustfmt + `cargo test` for Rust. If the stack has no standard
tool for something, say so rather than scaffolding a no-op. Typical options:
- **Code-reviewer agent** — a subagent in `.claude/agents/` that reviews diffs;
  pin its `model:` to the review model chosen in Step 2's orchestration.
- **Docker + compose** — a `Dockerfile` and `docker-compose.yml` for local dev.
- **GitHub link** — create/link a repo and push (`gh repo create`), add remote.
  Requires `gh` installed + authenticated (checked in Step 1). If push is on the
  guardrail deny list (below), be explicit that **Claude cannot run the push — the
  user runs it** — don't imply otherwise.
- **CI pipeline** — a `.github/workflows/` file running lint + tests on PRs.
- **Release skill** — a `.claude/skills/` playbook for cutting releases.
- **Conventions in CLAUDE.md** — architecture principles, style, commands.
- **Guardrail deny policy** — which actions stay HARD-BLOCKED for Claude via the
  `deny` list in `.claude/settings.json`. Do NOT hard-code this — ask it as a
  question. Offer the usual candidates (`git push`, `git tag`, `npm publish`,
  `gh release`, `gh repo delete`) pre-checked as sensible defaults, but let the
  user uncheck any. **Reconcile with the GitHub-link step:** if the user wants
  Claude to run `git push` / `gh repo create --push`, leave `git push` OUT of the
  deny list; if they keep push denied, state plainly that push is a user-run step.
  Whatever policy they choose, document it in a line in CLAUDE.md so it's
  discoverable.
- **Autopilot (auto-edit) mode** — let Claude apply edits without prompting, while
  the chosen deny policy above still blocks the guarded actions. If chosen, enable
  `acceptEdits` in `.claude/settings.local.json` and install the `/autopilot`
  command so the user can toggle it later. Ask whether to start it ON or OFF, and
  **at enable time state the two things users get wrong:** (a) `acceptEdits` only
  takes effect on the **next session start**, and (b) it **never covers Bash** —
  shell commands still prompt regardless. Don't let the user think "ON" silences
  everything.
- **Model configuration** — apply the orchestration roles chosen in Step 2: set
  the implementation model in `.claude/settings.json` (`"model"`), and for each
  role that got a stronger model (planning, review), scaffold the matching pinned
  subagent — e.g. `.claude/agents/architect.md` with `model:` set to the planning
  model — so heavy reasoning and review are delegated there while the main session
  stays on the cheaper model.
- **Guardrail hooks** — install event-driven hooks (run by the harness, not
  Claude, so they're deterministic — unlike CLAUDE.md rules). Create
  `.claude/hooks/` with three Node scripts and wire them in `.claude/settings.json`.
  **Naming: use `.cjs`, not `.js`.** The scripts use CommonJS (`require`), which
  throws `ReferenceError: require is not defined` in any project with
  `"type": "module"` in `package.json` (the Vite/Next default you detected in
  Step 1). Naming them `.cjs` forces CommonJS regardless of package type. (Only
  use `.js` if you instead write the scripts in ESM `import` syntax.)
  1. **block-protected.cjs** (PreToolUse on `Edit|Write`) — reads the hook JSON on
     stdin, and if `tool_input.file_path` matches a protected pattern (`.env*`,
     `.claude/settings.json`, anything under `.git/`) prints a JSON
     `hookSpecificOutput.permissionDecision: "deny"` and exits 2 to block it.
     This is the key guardrail — it enforces what a `deny` list can't (file paths)
     and holds even under autopilot. Confirm the protected list with the user.
  2. **format-file.cjs** (PostToolUse on `Edit|Write`) — runs the project's
     formatter (e.g. `npm run format`) if one exists; no-ops cleanly otherwise.
     **Scope it first (see below) so it never reformats hand-authored files.**
  3. **run-tests.cjs** (Stop) — runs the test suite when Claude finishes; no-ops
     until tests exist; surfaces a `systemMessage` on failure.
  Use Node (cross-platform, no `jq` dependency). Wire commands as
  `node "$CLAUDE_PROJECT_DIR/.claude/hooks/<script>.cjs"` with `"shell": "bash"`.
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
- **Do all `.claude/settings.json` writes in ONE final pass, and install the
  block-protected hook LAST.** Once that hook is live it denies edits to
  `settings.json` itself, so any later attempt to add the model pin, fix a hook
  path, or adjust permissions via the Edit tool will be blocked mid-run. Batch the
  permissions/deny policy, the `"model"` pin, and the hook wiring into a single
  write, then activate block-protected. If you genuinely must edit `settings.json`
  after the guardrail is live, use a non-Edit method (e.g. a short Node script),
  since the Edit tool is now blocked by design.
- **Scope the formatter before running it.** `prettier --write .` (or equivalent)
  rewrites EVERYTHING, including hand-authored `.claude/commands/*.md`, `CLAUDE.md`,
  and `README.md`. Before running any format command or enabling the format hook,
  write a `.prettierignore` (or the tool's equivalent) that excludes `.claude/`
  and the docs, and tell the user `--write .` touches all files, not just source.
- **Test setup uses a minimal, self-authored component — not the framework's demo
  art.** Framework starters ship demo templates that can fight the test runner
  (e.g. Vite's Vue demo uses an SVG sprite `<use href="/icons.svg">` that crashes
  jsdom). If you scaffold a sample test, write it against a tiny component you
  author, so the first test run is green.
- For the GitHub link, confirm before any network action (repo creation, push).
  Verify `gh` is installed + authenticated first (from Step 1); if not, stop and
  give the user the exact `gh auth login` / install step rather than failing at
  push. If push is on the deny list, hand the user the `git push` command to run
  themselves — don't attempt it.
- **On Windows, offer a `.gitattributes`** (`* text=auto eol=lf`) when initializing
  git, so the first commit doesn't flood CRLF warnings.
- Keep CLAUDE.md lean: add a short section per item, reference longer docs
  rather than inlining them.

## Step 5 — Sync the docs (CLAUDE.md + README) and report
### CLAUDE.md (Claude-facing)
- **Useful commands** — replace any placeholder/example commands with THIS
  project's real ones, derived from the stack (Step 2): the actual package
  manager and scripts (e.g. `pnpm dev`, `cargo test`, `uv run pytest`,
  `make build`). Never leave generic `npm run …` lines that don't exist here.
- **Layout / Tooling** — add or refresh a section so CLAUDE.md accurately lists
  what now exists (agents, skills, hooks, Docker, CI) and drops references to
  things that aren't there.
- **Guardrail policy** — record the deny policy the user chose in Step 3 (which
  actions are hard-blocked, and whether push is Claude-run or user-run) so it's
  discoverable and the autopilot/GitHub steps don't contradict each other.

### README.md (human-facing)
Update the README so it describes the project as it now stands — do this every
run; it is not a menu item. Note this means the CURRENT project's README, not
this wizard's own repo.
- **What to write** — pull the one-line purpose from the Step 2 profile; document
  the real stack, prerequisites, and how to install / run / test using the same
  real commands you put in CLAUDE.md (never generic `npm run …` that don't exist
  here). Add a short "what's here" note only for tooling you actually scaffolded
  in Step 4 (Docker, CI, agents, autopilot) — nothing aspirational.
- **If a README already exists** — treat hand-written prose, badges, screenshots,
  and license as sacred. Refresh only the factual sections (title/intro, stack,
  setup/run/test, layout); show the user a diff and confirm before replacing any
  substantial existing content. Prefer editing over rewriting.
- **If none exists** — create a lean one; don't just duplicate CLAUDE.md, which
  serves a different (agent) audience.

### Report
- Summarize what you created as a file tree, and list any follow-ups the user
  still needs to do by hand — call out the timing-gated ones explicitly:
  `gh auth login`, set secrets, install deps, run `git push` yourself (if denied),
  and note that the **model pin and autopilot mode both take effect next session**.

Guardrails: confirm before outward-facing or hard-to-reverse actions (pushing to
GitHub, deleting files). Prefer editing over recreating. Match the project's
existing style and indentation.
