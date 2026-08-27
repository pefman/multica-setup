# multica-setup

Bootstrap a working [Multica](https://multica.ai) project team in a few
commands: a squad of agent teammates with clear roles and shared
instructions, a kanban flow with explicit handoffs, and recurring
automations — all created and kept in sync **through the `multica` CLI**.

## What you get

- **A team** — Lead (coordinates, never implements), Engineer (builds),
  Reviewer (fresh-eyes checks), QA (verifies behavior), Release (ships
  approved work), Helper (workspace office). Each is a first-class Multica
  agent with a written job description — and the squad gets a random fun
  name, like "Galloping Otter".
- **A flow** — issues move Backlog → Todo → In Progress → In Review.
  Every handoff is one structured comment (`Done / Evidence / Questions /
  Ask` + an exact `@mention`). Agents never merge, never deploy, never set
  `done` — **you** review, merge, and close.
- **Automations** — a weekday standup, a midday stall radar that nudges
  quiet work, and a weekly board/repo hygiene pass run themselves and
  report to you. Give it git repos and it adds a **PR patrol** (stale
  PRs, branch cleanup) and a **docs check** (24h commits vs documentation)
  — and tells you it did.

## Start here

### Easiest: just run it

```bash
bin/multica-setup
```

A menu appears. Pick **1) Start a new project**: you answer ~4 questions
(workspace, project name, git repos), see exactly what will be created,
and press Enter to proceed with the sensible defaults. With git repos it
also enables the **pr-patrol** and **docs-check** autopilots — and tells
you so; without repos they stay off. When it's done, your team is live.

### Or, straight to the wizard / scripted

```bash
bin/multica-setup init                                              # guided, no menu
bin/multica-setup init --auto --new-workspace "My Project" \
  --slug my-project --project "My Project" --runtime opencode \
  --repos https://github.com/you/my-project \
  --roles lead,engineer,reviewer --smoke                            # scripts/CI, no prompts
```

`--smoke` creates a throwaway issue and chases it through the whole flow,
so you know it works end-to-end.

### Zero local files

```bash
bin/multica-setup init --auto --in-project --new-workspace "My Project" \
  --slug my-project --project "My Project" --runtime opencode
```

The spec is stored inside Multica (in the project's description), read
back through the CLI — no manifest file, and anyone with the CLI can
manage it:

```bash
bin/multica-setup --workspace my-project --project "My Project" apply
```

### Day to day

```bash
bin/multica-setup --manifest my-project.json status   # live team/board view
multica issue create --title "Add X" --description-stdin < x.md   # then assign
```

Create issues and assign them to the squad (or a single agent when the
owner is obvious). You get `@`-mentioned when work lands in `in_review` —
review, merge, done.

### When you're done with it

```bash
bin/multica-setup --manifest my-project.json teardown --yes
```

Archives everything the tool created. To remove the workspace shell
itself, delete the workspace in the Multica UI (the CLI can't).

## Pick your workflow

| You want to… | Run |
|---|---|
| Just try it out | `bin/multica-setup` (menu) |
| Set up a project, guided | `bin/multica-setup init` |
| Set up in a script / CI | `bin/multica-setup init --auto --new-workspace … --project … --runtime …` |
| No local files (spec lives in Multica) | `init --auto --in-project …`, then `--workspace <slug> --project <name> <command>` |
| Hand-write your own spec | copy `templates/project.example.json` → `check` → `plan` → `apply` |
| Manage a team/workspace you already have | `bin/multica-setup init --workspace <slug>` (or the menu: 2 → pick) |
| See what would change (read-only) | `bin/multica-setup --manifest my-project.json plan` |

## How it works

**The spec is yours; the state is Multica's.** The `multica` CLI is the
only source of truth for what exists — every command asks it live. Your
*spec* (workspace, project, team, runtimes, automations) lives either in
a local manifest file (git-track it like a terraform file) or inside
Multica itself (`--in-project`). `plan` diffs the spec against live
state; `apply` creates/updates whatever differs and is idempotent — safe
to re-run any time.

Apply works in this order: workspace → shared skills → agents → squad →
project → autopilots. `smoke` then proves the flow with a throwaway issue.

## Good to know

- **One team per workspace.** Agents and squads are workspace-scoped, so
  two managed projects in the same workspace would collide on names.
- **Slugs are global on Multica Cloud** (all users share the namespace).
  "workspace slug already exists" means someone else took it — pick a
  longer slug, or `init --workspace <slug>` to adopt an existing one.
- **Teardown archives, it doesn't delete.** If you tear down and re-create
  in the same workspace, archived teammates are automatically restored.
- **New workspace?** If its runtimes don't appear, your daemon may need a
  re-login/restart; `check` tells you.
- **Piped input never auto-applies.** In a script, use `--auto` (explicit);
  interactively you get an "apply now?" prompt.
- **Git autopilots need git in your runtime.** `pr-patrol` and `docs-check`
  are enabled only when you give `init` repositories. Your runtime machines
  need `git` (and `gh` for GitHub PR checks) installed.
- **Nothing merges, ships, or closes without you.** That's the design,
  not a bug (see below).

## The one rule that matters most

**Nothing merges, ships, or closes without you.** Agents deliver to
`in_review` with a delivery report; they never merge PRs, never deploy,
never set `done`. The board is yours to move at the end of the line.

## Customizing

- **Team shape**: drop or add roles in the manifest (`lead` is required;
  the others optional). Name agents whatever you like.
- **Instructions & skills**: edit `roles/*.md`, `skills/*/SKILL.md`,
  `squad/squad-instructions.md`, or `autopilots/*.md`, then re-run
  `apply` — drift is synced to the workspace. Add new runbooks under
  `autopilots/` and reference them by `key` in the manifest.
- **Models & runtimes**: per-role `runtime` (provider or runtime
  name/ID) and optional `model` / `thinking_level` overrides.

## Requirements

- The [`multica` CLI](https://multica.ai/docs/cli), installed and logged
  in (`multica auth status`, `multica daemon status`) — this tool drives
  Multica exclusively through it, so it works the same against Multica
  Cloud or a self-hosted instance (CLI profile decides).
- At least one online runtime in the target workspace (a connected
  computer with AI coding tools — Claude Code, Codex, Grok, opencode,
  pi, …). `check` tells you which runtimes exist and which are online.
- Python 3.10+ (standard library only — no pip installs).

## Docs & troubleshooting

- [`docs/runbook.md`](docs/runbook.md) — operations: the menu, the
  commands, what a run does, and fixes for common failures.
- [`docs/how-it-works.md`](docs/how-it-works.md) — the design.
- [`docs/flow.md`](docs/flow.md) — the kanban + handoff contract the
  agents follow.

## Repository map

| Path | What it is |
|---|---|
| `bin/multica-setup` | The bootstrap CLI — run it bare for the interactive menu, or use a subcommand (`init / check / plan / apply / status / smoke / teardown`) |
| `templates/` | Manifest example + field reference |
| `roles/` | Instruction templates per role (rendered into each agent) |
| `skills/` | Shared skills: `handoff-protocol`, `kanban-contract`, `pr-conventions`, `delivery-report` |
| `squad/` | Squad instructions (routing, handoff discipline, sign-off policy) |
| `autopilots/` | Runbooks: `daily-standup`, `stall-radar`, `weekly-hygiene`, `pr-patrol`*, `docs-check`* (*git repos required) |
| `docs/` | `how-it-works.md`, `flow.md`, `runbook.md` |