# Project manifest reference

A manifest is a JSON file describing one Multica project: where it lives
(workspace), what it is (project + repos), who works on it (team), and what
runs on its own (autopilots). `bin/multica-setup` reads a manifest and
creates or updates everything it describes — idempotently, so you can
re-run `apply` after editing templates or the manifest.

See `templates/project.example.json` for a complete, runnable example.

## Top-level keys

### `workspace` — where the project lives

| Field | Required | Meaning |
|---|---|---|
| `mode` | yes | `"existing"` or `"new"` |
| `slug` | if `existing` | Slug of the workspace to use (from `multica workspace list`). |
| `name` | if `new` | Display name of the new workspace. |
| `slug` | if `new` | URL slug — permanent, lowercase letters/digits/hyphens. |
| `issue_prefix` | if `new` | Issue key prefix, e.g. `SP` → `SP-12`. |
| `description` | optional | Workspace description. |

`mode: "existing"` never modifies the workspace itself — only the objects
inside it. `mode: "new"` creates the workspace; note the slug cannot be
changed after creation. After creating a new workspace the local daemon may
need a moment (or a `multica daemon restart` / re-login) to register
runtimes for it — if `check` reports no online runtime in the new
workspace, that is why.

### `project` — the work container

| Field | Required | Meaning |
|---|---|---|
| `name` | yes | Project title (e.g. `Sample Project MVP`). |
| `description` | recommended | Goes into the execution context of **every** run on this project's issues. Put the goal, technical boundaries, and delivery conventions here — one paragraph each, not a wiki. |
| `repos` | recommended | List of GitHub repo URLs, attached as project resources so runs know which code to use. |
| `lead_role` | optional | Role whose agent is set as project lead (default: `lead`). The lead *marks* coordination; it does not change permissions or auto-assign. |

### `team` — the agents and how they work together

`team.squad_name` — name of the squad that owns project work.
`team.routing` — `"hybrid"` (default: the lead routes ambiguous work, scoped
work can be assigned straight to a member) or `"leader-only"` (everything
goes through the lead). Reflected in the squad instructions.

`team.roles` — list of agent definitions:

| Field | Required | Meaning |
|---|---|---|
| `role` | yes | One of `lead`, `engineer`, `reviewer`, `qa`, `release`, `helper` — selects the instruction template from `roles/<role>.md`. The `lead` role must be present (it is the squad leader). |
| `name` | yes | Agent name — must be unique in the workspace. |
| `runtime` | yes | Provider of the runtime to bind (`grok`, `opencode`, `pi`, `claude`, `codex`, …) or an exact runtime name/ID. Resolved against the online runtimes of the target workspace. |
| `model` | optional | Model identifier override (e.g. `claude-sonnet-4-6`, `local-llama-8080/qwen3.8-27b`). Empty/null = runtime default. |
| `thinking_level` | optional | e.g. `low`/`medium`/`high`/`xhigh` (Claude-style) — runtime/model specific; leave out if unsure. |
| `max_concurrent_tasks` | optional | WIP limit for this agent (default: 1). |
| `in_squad` | optional | `false` keeps the agent out of the squad (e.g. a workspace-level helper). Default: true. |
| `squad_role` | optional | One-line role description shown to the squad leader when delegating. Default: a per-role line. |

Access for every bootstrapped agent is set to *entire workspace* so the
squad leader (and you) can trigger every member.

`team.skills` — list of skill assignments:

| Field | Meaning |
|---|---|
| `name` | Skill name in the workspace. |
| `source` | `"local"` — imported from this repo's `skills/<name>/` folder, or `"url"` — imported from `url`. |
| `url` | For `source: "url"` — GitHub/ClawHub/Skills.sh URL of the skill. For a GitHub repo that does not have `SKILL.md` at its root, point at the skill directory: `.../tree/main/<skill-dir>` (e.g. the official `multica-cli` skill: `https://github.com/multica-ai/multica-cli/tree/main/skills/multica-cli`). |
| `attach_to` | `"all"` or a list of role keys the skill is attached to. |

Local skills are re-imported on every `apply` so edits to this repo's
`skills/` propagate. URL skills are imported once (conflicts are skipped);
update them later with `multica skill refresh <id>`.

### `autopilots` — recurring work

| Field | Required | Meaning |
|---|---|---|
| `key` | yes | Selects the runbook: the file `autopilots/<key>.md` in this repo is rendered into the autopilot's description. Custom runbooks: add the file. |
| `name` | yes | Autopilot title (shown in Multica). |
| `assignee_role` | yes | Role whose agent executes the autopilot. |
| `mode` | yes | `create_issue` (each trigger lands as a real issue on the board — recommended) or `run_only` (history only, needs the runtime online at trigger time). |
| `cron` | yes | 5-field cron expression, e.g. `30 8 * * 1-5`. |
| `timezone` | yes | IANA timezone, e.g. `Europe/Stockholm`. |
| `issue_title` | if `create_issue` | Title template for created issues; only `{{date}}` (UTC, `YYYY-MM-DD`) is interpolated. |

Available in this repo: `daily-standup` (morning board report by the lead)
and `weekly-hygiene` (board + repo + disk maintenance pass, report-only).

## Conventions the manifest bakes in

- **No auto-merge, no auto-done, no auto-deploy.** Every delivery parks in
  `in_review` for the owner; `done`, merges, and deploys are human actions.
- **Handoffs are one comment** (Done / Evidence / Questions / Ask + exact
  `@mention`), enforced by the `handoff-protocol` skill and the squad
  instructions.
- **One active issue per agent** (`max_concurrent_tasks: 1` default).
- **Backlog is a parking lot**: nothing triggers there; the lead (or you)
  promotes scoped work to `todo`.