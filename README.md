# multica-setup

Bootstrap a working [Multica](https://multica.ai) project team in a few
commands: a squad of agent teammates with clear roles and shared
instructions, a kanban flow with explicit handoffs, and recurring
automations — all created and kept in sync **through the `multica` CLI**.

The result is a real working team, not four agents that all start shouting
at once:

- **A team** — Lead (coordinates, never implements), Engineer (builds),
  Reviewer (fresh-eyes checks), QA (verifies behavior), Release (ships
  approved work), Helper (workspace office). Each is a first-class Multica
  agent with a written job description.
- **A flow** — issues move Backlog → Todo → In Progress → In Review.
  Every handoff is one structured comment (`Done / Evidence / Questions /
  Ask` + an exact `@mention`). Agents never merge, never deploy, never set
  `done` — **you** review, merge, and close.
- **Automations** — a daily standup issue and a weekly board/repo hygiene
  pass run themselves and report to you.

## Requirements

- The [`multica` CLI](https://multica.ai/docs/cli), installed and logged in
  (`multica auth status`, `multica daemon status`) — this tool drives
  Multica exclusively through it, so it works the same against Multica
  Cloud or a self-hosted instance (CLI profile decides).
- At least one online runtime in the target workspace (a connected
  computer with AI coding tools — Claude Code, Codex, Grok, opencode, pi,
  …). `check` tells you which runtimes exist and which are online.
- Python 3.10+ (standard library only — no pip installs).

## Quickstart

```bash
# Just run it — it shows a menu and asks the questions:
# new project (guided wizard), or pick an existing manifest
# (check / plan / apply / status / smoke / teardown).
#
# The wizard asks ~3 questions (workspace, project name), then shows
# exactly what it will create — press Enter to proceed with the defaults
# (the team gets a random fun name like "Galloping Otter"), or n to
# customize roles, runtimes, repos, and automations.
bin/multica-setup

# Or go straight to the wizard:
bin/multica-setup init

# Non-interactive (scripts/CI) — same result, no prompts:
bin/multica-setup init --auto --new-workspace "My Project" \
  --slug my-project --project "My Project" --runtime opencode \
  --roles lead,engineer,reviewer --smoke
```

If you stopped before `apply` (or ran with `--no-apply`), pick it up later
with the manifest `init` wrote:

```bash
bin/multica-setup --manifest my-project.json apply   # idempotent — safe to re-run
bin/multica-setup --manifest my-project.json smoke   # prove the flow end-to-end
```

Then start working: create issues in the workspace and assign them to the
squad (or directly to an agent when the owner is obvious). You will be
`@`-mentioned when work lands in `in_review` — review, merge, done.

```bash
# Day to day:
bin/multica-setup --manifest my-project.json status   # live team/board view
multica issue create --title "Add X" --description-stdin < x.md   # then assign
```

Prefer to hand-maintain a manifest instead of the guided `init`? Copy
`templates/project.example.json` to `my-project.json`, fill it in, then run
`check` → `plan` → `apply` → `smoke` as above.

## Repository map

| Path | What it is |
|---|---|
| `bin/multica-setup` | The bootstrap CLI — run it bare for the interactive menu, or use a subcommand (`init / check / plan / apply / status / smoke / teardown`) |
| `templates/` | Manifest example + field reference |
| `roles/` | Instruction templates per role (rendered into each agent) |
| `skills/` | Shared skills: `handoff-protocol`, `kanban-contract`, `pr-conventions`, `delivery-report` |
| `squad/` | Squad instructions (routing, handoff discipline, sign-off policy) |
| `autopilots/` | Runbooks: `daily-standup`, `weekly-hygiene` |
| `docs/` | `how-it-works.md`, `flow.md` (the kanban + handoff contract), `runbook.md` (ops & troubleshooting) |

## The one rule that matters most

**Nothing merges, ships, or closes without you.** Agents deliver to
`in_review` with a delivery report; they never merge PRs, never deploy,
never set `done`. The board is yours to move at the end of the line.

## Customizing

- **Team shape**: drop or add roles in the manifest (`lead` is required;
  the others optional). Name agents whatever you like.
- **Instructions & skills**: edit `roles/*.md`, `skills/*/SKILL.md`,
  `squad/squad-instructions.md`, or `autopilots/*.md`, then re-run `apply`
  — drift is synced to the workspace. Add new runbooks under `autopilots/`
  and reference them by `key` in the manifest.
- **Models & runtimes**: per-role `runtime` (provider or runtime name/ID)
  and optional `model` / `thinking_level` overrides in the manifest.
- **Teardown**: `bin/multica-setup --manifest my-project.json teardown --yes`
  archives everything the tool created (workspace deletion is UI-only).