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

Copy a manifest, point it at a new workspace (or an existing one), adjust
`project` + `repos`, run `check` → `plan` → `apply`. Agents/skills/squads
are per-workspace, so a new workspace gets a fresh copy of the team.

## Troubleshooting

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