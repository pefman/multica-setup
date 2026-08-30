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
| `repos` | recommended | List of GitHub repo URLs (use **https** — your daemon can't use your SSH keys, and `git@…` URLs make agents re-register the repo mid-run), attached as project resources so runs know which code to use. |
| `lead_role` | optional | Role whose agent is set as project lead (default: `lead`). The lead *marks* coordination; it does not change permissions or auto-assign. |

### `team` — the agents and how they work together

`team.squad_name` — name of the squad that owns project work.
`team.routing` — `"hybrid"` (default: the lead routes ambiguous work, scoped
work can be assigned straight to a member) or `"leader-only"` (everything
goes through the lead). Reflected in the squad instructions.

`team.roles` — list of agent definitions:

| Field | Required | Meaning |
|---|---|---|
| `role` | yes | One of `lead`, `engineer`, `reviewer`, `qa`, `release`, `docs`, `helper` — selects the instruction template from `roles/<role>.md`. The `lead` role must be present (it is the squad leader). |
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

Available in this repo: `daily-standup` (weekday morning board report by
the lead), `stall-radar` (weekday midday: nudges quiet in-flight issues on
their own issue), `weekly-hygiene` (board + repo + disk maintenance pass),
and the git runbooks `pr-patrol` (stale open/unreviewed PRs, deletes
branches of merged PRs, lists other branch-cleanup candidates for approval)
and `docs-check` (24h commit window vs docs; one backlog issue per gap;
runs as the Docs role when the team has one, otherwise the lead).
`init` enables **daily-standup only** by default; the others are opt-in
via the wizard customize step or by adding them to the manifest. Git
runbooks only appear in the customize catalog when repositories are
registered.

### `mcp` — external tools for agents (optional)

Registers MCP servers in the workspace's MCP library and assigns them to
agents. The stored configuration is **write-only** on the server side (it
may hold API tokens and is never returned to anyone, not even the owner),
so `apply` never re-pushes or diffs an existing library entry.

| Field | Required | Meaning |
|---|---|---|
| `servers` | yes | Map of server name → `mcpServers`-format entry: `{"command", "args"}` (stdio — runs on the runtime machine) or `{"url"}` (hosted HTTP, optionally `headers`). |
| `attach_to` | no | `"all"` (default) or a list of roles — which agents get each listed server. |

Bundled presets (enabled by default in `init` — `--no-mcp` disables
them, `--mcp X,Y` picks a different set, and the wizard's customize step
can change the selection):
- `context7` — up-to-date, version-specific library docs (stdio,
  `npx @upstash/context7-mcp`, no key; the runtime machine needs node)
- `firecrawl` — web search + scraping (hosted
  `https://mcp.firecrawl.dev/mcp`; the keyless tier has usage limits —
  raise them with a custom entry carrying
  `"headers": {"Authorization": "Bearer ${FIRECRAWL_API_KEY}"}`)

Any other server (GitHub, Playwright, Sentry, …) works as a custom entry
in the same `servers` map. String values may reference environment
variables as `${VAR_NAME}` — expanded from the local environment at apply
time; **never put raw tokens in the manifest** (an unset variable fails
the apply, naming the variable).

## Conventions the manifest bakes in

- **No auto-merge, no auto-deploy.** Every delivery parks in `in_review`
  for the owner; merges and deploys are human actions. `done` is the
  exception to "no auto": the lead sets it on a delivery only after the
  owner's approval on the thread and the linked PR is merged, and it
  closes start-decision / kickoff / feature parents / autopilot-run issues
  that no longer serve a purpose — so the board self-cleans without you.
- **Features, not ticket spam**: new scope becomes one Feature parent with
  three staged children (Research → Implement → Verify); one branch/PR per
  feature. Docs ride Implement. Trivial fixes may be a single issue.
- **Handoffs are one comment** (Done / Evidence / Questions / Ask + exact
  `@mention`), enforced by the `handoff-protocol` skill and the squad
  instructions.
- **Role contract:** agents refuse out-of-role asks and point at the right
  owner (`role-contract` skill, attached to all).
- **One active issue per agent** (`max_concurrent_tasks: 1` default).
- **Backlog is a parking lot**: nothing triggers there; the lead (or you)
  promotes scoped work to `todo`.
- **Helper is out of the squad by default** (`in_squad: false`) so it is
  not mistaken for a delivery route.