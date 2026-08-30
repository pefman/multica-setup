# multica-setup

Bootstrap a working [Multica](https://multica.ai) project team in a few
commands: a squad of agent teammates with clear roles and shared
instructions, a kanban flow with explicit handoffs, and recurring
automations — all created and kept in sync **through the `multica` CLI**.

## What you get

- **A team** — Lead (coordinates, never implements), Engineer (builds),
  Reviewer (fresh-eyes checks), QA (verifies behavior), Release (ships
  approved work), Docs (keeps the repositories' documentation in sync),
  Helper (workspace office). Each is a first-class Multica
  agent with a written job description — and the squad gets a random fun
  name, like "Galloping Otter".
- **A flow** — new work is planned as a **Feature** with three staged
  phases (Research → Implement → Verify), one branch/PR per feature.
  Issues move Backlog → Todo → In Progress → In Review. Every handoff is
  one structured comment (`Done / Evidence / Questions / Ask` + an exact
  `@mention`). Agents never merge and never deploy — **you** review and
  merge, and once you approve the Lead closes the Feature with the
  evidence. The board keeps itself clean: decision tickets, kickoff,
  Feature parents, and autopilot runs are closed by the team.
- **Automations** — a weekday standup runs by default, reports to you, and
  closes its own issue. Stall radar, weekly hygiene, **PR patrol**, and
  **docs check** are opt-in (customize or manifest).

## Start here

### Easiest: just run it

```bash
bin/multica-setup
```

A menu appears. Pick **1) Start a new project**: you answer ~4 questions
(workspace, project name, git repos), see exactly what will be created,
and press Enter to proceed with the sensible defaults (standup only;
other autopilots stay off until you opt in). When it's done, your team is
live — and the Lead opens a **kickoff issue** that pings you: tell it what
to start working on.

### Or, straight to the wizard / scripted

```bash
bin/multica-setup init                                              # guided, no menu
bin/multica-setup init --auto --new-workspace "My Project" \
  --slug my-project --project "My Project" --runtime opencode \
  --repos https://github.com/you/my-project \
  --roles lead,engineer,reviewer --smoke                            # scripts/CI, no prompts
```

`--smoke` creates a throwaway issue and chases it through the whole flow,
so you know it works end-to-end. (After `apply` the kickoff issue is
opened automatically — the Lead greets you and asks for your first pitch;
`--no-kickoff` skips it.)

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
review and merge; the Lead closes the issue once your approval and the
merged PR are on record.

### Update a project you already have

```bash
bin/multica-setup --manifest my-project.json plan   # see what will change (read-only)
bin/multica-setup --manifest my-project.json apply  # sync it
```

The prompts and behavior come from this repo — `roles/*.md`, `skills/`,
`squad/`, `autopilots/`. To bring a live project up to date (after you edit a
role, add a skill, or pull a new version of this tool), just re-run `apply`
against the same workspace and project. It diffs your spec against what's
live and updates whatever drifted — agent instructions, skills, autopilots —
while leaving the rest alone. It's idempotent: re-running when nothing changed
does nothing. If your spec lives inside Multica rather than in a local file,
use `--workspace`/`--project` instead of `--manifest`:

```bash
bin/multica-setup --workspace my-project --project "My Project" apply
```

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
project → autopilots, then `kickoff` opens the first issue — the Lead
pings you and asks for your first idea pitch. `smoke` proves the whole
flow with a throwaway issue.

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
- **Autopilots default to standup only.** `init` enables `daily-standup`;
  hygiene, stall-radar, pr-patrol, and docs-check are opt-in via customize
  or the manifest. Git runbooks need `git` (and `gh` for GitHub) on the
  runtime when you enable them.
- **stdio MCP presets need node in your runtime.** `context7` runs as
  `npx` on the runtime machine, so it needs `node`; hosted presets like
  `firecrawl` need nothing locally. `init` can install node on the machine
  it runs on (other runtime machines are on you), and `check` warns when
  a prerequisite is missing.
- **Nothing merges or ships without you.** Closing is the team's bookkeeping:
  after your approval and the merged PR, the Lead closes the delivery;
  decision tickets and autopilot runs close themselves. The board cleans
  up without you, but the line stays yours (see below).
- **Nothing starts without you either.** The lead parks a Feature
  (Research → Implement → Verify) and opens a start-decision ticket
  assigned to you; only your reply @-mentioning the lead starts Research
  (the ticket then closes itself with your decision recorded on it).
- **Use https repo URLs in the manifest.** Your daemon can't use your SSH
  keys; with `git@…` URLs the agents re-register the repo as https
  mid-run (which works, but you'll end up with both entries registered).
- **PR/CI on the board needs a GitHub integration.** Linking PRs and CI
  status to issues is done by Multica watching GitHub (set it up in the
  workspace settings, web UI). Without it, agents record the branch and
  PR on the issue as metadata (`pr_url`, `branch`) instead — the board
  stays accurate, the auto-links don't.

## The one rule that matters most

**Nothing merges or ships without you.** Agents deliver to `in_review`
with a delivery report; they never merge PRs and never deploy. After your
approval and the merged PR, the Lead closes the issue — the bookkeeping
is theirs, the decisions are yours, and the board stays clean.

## Customizing

- **Team shape**: drop or add roles in the manifest (`lead` is required;
  the others optional). Name agents whatever you like.
- **Instructions & skills**: edit `roles/*.md`, `skills/*/SKILL.md`,
  `squad/squad-instructions.md`, or `autopilots/*.md`, then re-run
  `apply` — drift is synced to the workspace. Add new runbooks under
  `autopilots/` and reference them by `key` in the manifest.
- **Models & runtimes**: per-role `runtime` (provider or runtime
  name/ID) and optional `model` / `thinking_level` overrides.
- **MCP tools**: the team gets the bundled presets (`context7`,
  `firecrawl`) by default — `--no-mcp` turns them off, `--mcp X,Y` picks
  a different set. Presets live in `templates/mcp/`; any
  `mcpServers`-format entry works as a custom server. Keys are referenced
  as `${ENV_VAR}` — never in the file.

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
| `bin/multica-setup` | The bootstrap CLI — run it bare for the interactive menu, or use a subcommand (`init / check / plan / apply / status / smoke / kickoff / teardown`) |
| `templates/` | Manifest example + field reference |
| `templates/mcp/` | Bundled MCP server presets (`context7`, `firecrawl`) — on by default in `init` (`--no-mcp` disables, `--mcp` overrides) |
| `roles/` | Instruction templates per role (rendered into each agent) |
| `skills/` | Shared skills: `handoff-protocol`, `kanban-contract`, `pr-conventions`, `delivery-report` |
| `squad/` | Squad instructions (routing, handoff discipline, sign-off policy) |
| `autopilots/` | Runbooks: `daily-standup` (default), plus opt-in `stall-radar`, `weekly-hygiene`, `pr-patrol`, `docs-check` |
| `docs/` | `how-it-works.md`, `flow.md`, `runbook.md` |
| `llms.txt` | Guide for AI assistants editing this repository (repo map, editing rules, verification, conventions) |