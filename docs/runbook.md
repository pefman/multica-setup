# Runbook: operating & troubleshooting

Day-to-day commands, and what to do when something doesn't move.

## Every day

```bash
# What's out there, live:
bin/multica-setup --manifest my-project.json status

# Add work (then assign in the UI, or via CLI):
multica issue create --title "Add X" --status backlog --description-file x.md
multica issue assign <ISSUE> --to "<SquadName>"        # or a specific agent

# Watch a run:
multica issue get <ISSUE>
multica issue runs <ISSUE>                             # run history
multica daemon logs --follow                           # daemon side (only if tasks won't start)
```

All board operations have a CLI — and every agent on the team uses the
same CLI through its `multica-cli` skill, so what you do in a terminal is
exactly what the team does on a run.

## Updating the team

Edit the files in this repo (roles, skills, squad instructions, autopilot
runbooks) and re-run:

```bash
bin/multica-setup --manifest my-project.json plan     # see the drift
bin/multica-setup --manifest my-project.json apply    # sync it
```

`apply` is idempotent: it creates what's missing and updates what drifted
(instructions, skills, squad membership/instructions, autopilot runbooks
and cron). It never deletes or archives on its own.

Per-object tweaks that don't belong in the repo (one-off model change, WIP
limit): use the CLI directly, e.g.
`multica agent update <id> --model <model>` — the next `apply` will report
that field as drift (fix it in the manifest when it becomes permanent).

## New project on the same team shape

Just run `init` again — it writes a fresh manifest for the new workspace
and chains `check` → `plan` → (ask) `apply` → (ask) `smoke`:

```bash
bin/multica-setup init --new-workspace "My Other Project" \
  --slug my-other-project --project "My Other Project" --runtime opencode
```

Agents/skills/squads are per-workspace, so a new workspace gets a fresh copy
of the team. To reuse a hand-tuned manifest instead, copy it, point it at
the new workspace, adjust `project` + `repos`, run `check` → `plan` →
`apply`.

## The autopilots (what runs on its own)

| Runbook | Schedule (default) | What it does | Needs |
|---|---|---|---|
| `daily-standup` | weekdays 08:30 | Board report: counts, in-flight, blocked, "waiting on you", decisions needed | — |
| `stall-radar` | weekdays 13:00 | Nudges quiet `in_progress`/`blocked` issues **on their own issue** (@ the owner agent); escalates to you what it can't unblock | — |
| `weekly-hygiene` | Monday 09:00 | Board↔merged-PR reconciliation, stale flags, repo hygiene pass, runtime disk usage | repos for the repo pass |
| `pr-patrol` | daily 09:30 | Open PRs unmerged > 3d → @ author; unreviewed > 24h → nudge reviewer; **deletes branches of merged PRs** (safe); closed-unmerged > 14d and orphan branches → listed for your approval, never deleted | repos, `git` (+ `gh` for GitHub) in the runtime |
| `docs-check` | daily 17:30 | 24h commit window vs README/docs/CHANGELOG; one `backlog` issue per concrete docs gap | repos, `git` in the runtime |

`init` always enables standup, stall-radar, and hygiene; **pr-patrol and
docs-check are enabled only when you give it git repositories** — it tells
you which case happened ("…are ON because git repos are registered" /
"…are OFF (register repos to enable them)"). `check` reminds you if repos
are registered but the git runbooks are missing from the manifest, and
warns if a repo is unreachable from your machine (the runtime machines may
still reach it — it's a probe, not a verdict).

To change schedules/selection: edit the manifest's `autopilots` list
(or the wizard's customize path, `n`), re-`apply`.

## MCP servers (external tools for agents)

The manifest's `mcp` section registers servers in the workspace's MCP
library (`multica workspace mcp list`) and assigns them to agents
(`multica agent mcp list <agent-id>`). `init` enables the bundled
presets (`context7`, `firecrawl`) by default; `--no-mcp` turns them off
and `--mcp X,Y` picks a different set. Two transport shapes:

- **stdio** (e.g. `context7`) — runs as a process **on the runtime
  machine**, so that machine needs the binary (`node`/`npx` for
  context7). `init` can install node on the machine it runs on (or print
  the command), and `check` warns when a prerequisite is missing here.
- **hosted** (e.g. `firecrawl`) — plain HTTP, no local prerequisites;
  the keyless tier is rate-limited, a key raises it.

Operational notes:

- The server-side config is **write-only** — it may hold API tokens and
  is never returned to anyone. So `apply` never re-pushes or diffs an
  existing library entry; to change a server's config, edit the manifest
  and use `multica workspace mcp update <name>`, or remove + re-add.
- **Keys are references, not values**: write
  `"headers": {"Authorization": "Bearer ${FIRECRAWL_API_KEY}"}` and the
  value is expanded from the environment at `apply` time. An unset
  variable fails the apply and names the variable. Never put a raw token
  in the manifest (it would land in git).
- **Removal**: dropping a server from the manifest unassigns it from the
  agents on the next `apply`; the library entry is kept (it may be shared)
  — remove it explicitly with `multica workspace mcp remove <name>`.
- The agent's instructions gain a short "MCP tools" section listing what
  each attached server is for, so the agent actually reaches for it.

## The menu (bare `bin/multica-setup`)

Lists **live workspaces from the multica CLI** (the source of truth).
Local `*.json` specs in the directory are only shown as annotations
(`manifest: <file>`); specs whose workspace is not in this account land
under "local specs with no workspace in this account" and are driven with
`--manifest <file>`. A workspace without a local spec lists its
multica-setup-managed projects (spec read back from the project
description via the CLI); with none of either it offers read-only status
and adoption via `bin/multica-setup init --workspace <slug>` (add
`--in-project` to keep the spec in Multica instead of a local file).

## Troubleshooting

**Parked build never starts after the owner said yes**
A plain "yes" in chat triggers nobody — only the exact mention markdown
does. Reply on the start-decision ticket with a comment that @-mentions
the lead (copy the mention syntax from the squad roster). The approval is
recorded on the ticket; from there the lead assigns the first subtask.

**Tasks never start / issue stuck after assign**
- `multica daemon status` — is the daemon running?
- `multica runtime list` — is the runtime the agent is bound to **online**?
  Offline runtimes queue tasks silently until they return.
- The agent is bound to a runtime on another computer? Check that machine.
- `multica daemon logs --follow` for the daemon's view.

**I created a new workspace and it has no runtimes**
The daemon registers runtimes per workspace at login. Run
`multica login` (or `multica daemon restart`) on the machine, re-check with
`bin/multica-setup ... check`, then re-run `apply` — it is idempotent.

**apply fails with "workspace slug already exists"**
On Multica Cloud, slugs are unique across **all users** — short/generic
ones (`testing`, `d1`, …) are often already owned by another account's
workspace and will never appear in your `workspace list` (so no pre-flight
can see them either). Pick a longer, unique slug in the manifest and
re-run `apply` — or if you meant to manage an existing workspace,
`bin/multica-setup init --workspace <slug>`.

**Re-initializing a torn-down team in the same workspace**
Teardown *archives* agents (the CLI cannot delete them), and archived
agents keep their names reserved. `apply`/`init` handle this automatically:
archived agents are **restored** and re-synced. If you ever see
"an agent named X already exists" instead, your tool is older than 1.5 —
or the name belongs to a non-archived agent you manage yourself.

**Why did `init` (wizard) skip `apply`?**
With piped/non-interactive stdin, every unanswered prompt falls back to its
default — so the wizard *never* auto-applies: a partial pipe could otherwise
land on the first workspace with "apply = yes" by default. Scripted runs use
`init --auto ...` (explicit); interactive runs get the "Run apply now?"
prompt.

**A run fails**
`multica issue runs <ISSUE>` → open the failed run → `run-messages` shows
the transcript. The issue rolls back to `todo` automatically when a run
fails and nothing else is in flight; fix and re-assign (or
`multica issue rerun <ISSUE>`).

**An agent does the wrong thing repeatedly**
That's a team-definition bug, not a one-off. Find the responsible
`roles/<role>.md` or `skills/*/SKILL.md`, make the rule explicit there,
`apply`. Don't fix it in individual issue comments — it won't stick.

**PR doesn't show on the issue / status doesn't flip**
The PR title must carry the issue identifier (the `pr-conventions` skill
enforces this: `ISSUE-12 Add X`). Connect GitHub in the workspace settings
for auto-linking and CI display.

**Two agents arguing in a loop**
The squad instructions cap rework at two rounds, then the lead escalates
to you with both sides on the record. If you see three rounds, the leader
isn't following its instructions — tighten `squad/squad-instructions.md`
and re-apply.

**An autopilot misbehaves**
`multica autopilot get <id> --output json` (triggers, schedule, run
history via `multica autopilot runs <id>`). Pause it in the UI or via
`multica autopilot update <id> --status paused`; edit the runbook file and
re-`apply` to change its behavior.

**Tearing a project down**

```bash
bin/multica-setup --manifest my-project.json teardown --yes
```

Archives the agents, squad, and autopilots it created, deletes the local
skills and the project. Open issues on an archived squad transfer to the
former leader. The **workspace itself can only be deleted in the web UI**.

## Safety notes

- This tool only ever talks to Multica through your logged-in `multica`
  CLI, using `--workspace-id` per call — it never changes your CLI
  profile, default workspace, or daemon.
- Nothing here stores credentials; the CLI's own config files hold your
  token (don't commit `~/.multica/`).
- Skills are plain prompts handed to coding tools — review third-party
  skills (e.g. the `multica-cli` import) before attaching them; this repo's
  own skills are under your git control.