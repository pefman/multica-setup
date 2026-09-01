# AGENTS.md

multica-setup bootstraps a Multica project team (agent squad + kanban flow +
autopilots) from a JSON manifest. The tool is one Python file —
`bin/multica-setup` (3.10+, standard library only, ~2800 lines) — that drives
Multica exclusively through the `multica` CLI (subprocess, `--output json`);
it never calls any API directly. All other directories (`roles/`, `skills/`,
`squad/`, `autopilots/`, `templates/mcp/`) are template data rendered and
pushed into Multica by `apply`.

- Start with `llms.txt` — the detailed AI guide (repo map, per-addition
  checklists, calibrated `multica` 0.4.x CLI response quirks). Keep it in
  sync when behavior changes.

## Commands (there is no test suite)

- Subcommands: `init / check / plan / apply / status / smoke / kickoff /
  teardown`; bare run = interactive menu. `check` is a read-only pre-flight
  (CLI, auth, daemon, runtimes, templates).
- After ANY edit to `bin/multica-setup`:
  `python3 -c "import ast; ast.parse(open('bin/multica-setup').read())"`
- Dry run, no side effects (needs a logged-in `multica` CLI):
  `bin/multica-setup --manifest templates/project.example.json plan`
- `plan` is a dry `apply`; both diff spec vs live CLI state; `apply` is
  idempotent. Re-runs are no-ops when nothing drifted.
- Full E2E: scratch workspace only —
  `bin/multica-setup init --auto --new-workspace "<name>" --slug <random> --project "<name>" --runtime pi`
(the chain ends with `kickoff` after apply, skip with `--no-kickoff`: a
   `Kickoff:` todo issue assigned to the squad, whose lead comments a greeting
   + pitch question; `--no-apply` stops after check+plan; add `--smoke` to
   also chase a throwaway issue through the whole flow)
  → verify via the `multica` CLI → `bin/multica-setup --workspace <slug> --project "<name>" teardown --yes`
  → ask the user to delete the workspace shell in the web UI (the CLI can't).
- NEVER run `apply`/`teardown` against a real/production workspace without
  explicit instruction. Piped input never auto-applies; use `--auto` for
  scripted runs.

## Architecture notes

- Flow: manifest → `Manifest` dataclass → `build_context()` → `render()` →
  `Multica` client → `Plan` steps in this order: workspace → skills → agents
  → squad → projects → autopilots. A manifest may hold **several projects**
  per workspace (`projects` list; the singular `project` is shorthand for one)
  — one shared team/squad serves all of them, each project carries its own
  repos and `lead_role`, and agents/squad prompts list every project with a
  hard rule to work only in the issue's project's repos (ask when unsure).
- `render()` substitutes `{{key}}` strictly: a key missing from context, or
  left unrendered, raises `ManifestError`. Context keys are exactly: `projects_names`, `projects_section`, `repo_list`
  (union over all projects), `squad_name`, `issue_prefix`,
  `roster`, `routing_table`, `routing_bullets`,
  plus `<role>_name` per manifest role. Adding a `{{placeholder}}` to any
  template requires adding the key in `build_context()`.
- The spec lives in a local `<workspace>.json`, or (with `--in-project`) in
  the managed project's description as an HTML comment block, read back via
  `multica project list`.
- `Multica` client methods wrap CLI calls and `unwrap_list()` JSON
  envelopes; `MulticaError` wraps non-zero exits with the CLI's stderr.
  Envelope shapes are calibrated to multica 0.4.x — re-verify after CLI
  upgrades (list in `llms.txt`).
- Autopilot defaults: `default_ap` is standup-only. Optional runbooks live
  in `OPTIONAL_AP_DEFS` + `GIT_AP_DEFS` (catalog / opt-in). Git ones need
  `git` (and `gh`) on the runtime when enabled; `docs-check` is assigned
  to the Docs role when the team has one, otherwise to the lead. An
  autopilot may pin its issues to one project via `project` (name; default:
  the first project).
- Every workspace has Multica's built-in `mika` agent (system_key `mika`):
  the server refuses to archive/disable it ("built into Multica"), so the
  tool never manages it and `status` hides `system_key` agents from the
  agent list.
- `init` enables the bundled MCP presets (`context7`, `firecrawl`) by
  default; `--no-mcp` disables, `--mcp a,b` overrides. Interactive init
  asks for a Firecrawl API key (N = keyless); `--auto` uses
  `FIRECRAWL_API_KEY` from the environment when set. Custom
  mcpServers-format entries in the manifest work without code changes.

## Docs to keep in sync with any behavior change

All of: `README.md` ("Good to know" + repo map), `docs/runbook.md` (commands
and autopilot tables), `templates/project.schema.md` (manifest field
reference), `llms.txt`.

## Adding things — touch every listed place

- **Role**: `ROLES` + `ROLE_BLURBS` (top of file) + `roles/<key>.md` +
  `ROUTE_TABLE`/`ROUTE_BULLETS` + the skill `attach_to` lists in `cmd_init`
  + a line in `templates/project.schema.md` + a line in README. `lead` is
  the only required role; new roles must be optional.
- **Autopilot**: `autopilots/<key>.md` (house voice: `## Steps` numbered,
  `## Boundaries` explicit, output is ONE comment on the autopilot's own
  issue) + a line in the schema + README repo map + `docs/runbook.md` table
  + catalog/defaults in `cmd_init` (`default_ap` = standup;
  `OPTIONAL_AP_DEFS` / `GIT_AP_DEFS` for opt-in).
- **Skill**: `skills/<name>/SKILL.md` (frontmatter `name:` + `description:`)
  + a `skills` manifest entry (`source: local` or `url`,
  `attach_to: all` or a role list).
- **MCP preset**: `templates/mcp/<name>.json` (mcpServers-format) + name in
  `MCP_PRESETS` + entry in `MCP_BLURBS` + `MCP_PREREQS` if stdio + a line in
  the schema. Server configs reference `${ENV_VAR}` — expand at apply time,
  never store raw tokens.

## Code style

- Standard library only; no type hints; no comments except
  `# ---- section ----` banners; keep functions small. Match existing style.

## Git conventions

- Commit identity via environment variables only (never touch git config):
  `GIT_AUTHOR_NAME="Peter Frank" GIT_AUTHOR_EMAIL="info@peterfrank.se"`
  (same values for `GIT_COMMITTER_NAME`/`GIT_COMMITTER_EMAIL`).
- Tag a release per coherent feature batch: `git tag -a <version> -m
  "multica-setup <version>: <summary>"`, push `origin main <tag>`.
- Never commit user files that appear in the working tree (e.g. generated
  `*.json` manifests); stage exactly the files you changed.
