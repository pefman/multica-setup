# How it works

What Multica gives you, and how this kit uses it.

## Multica's building blocks

- **Workspace** — a team's container: issues, projects, agents, squads,
  skills, autopilots all live in one.
- **Runtime** — a connected computer and the AI coding tools on it
  (Claude Code, Codex, Grok, opencode, pi, …). Agents don't run anywhere
  until a runtime executes them.
- **Agent** — a reusable teammate: name, instructions (injected on every
  run), runtime + model, attached skills, access. It only acts when
  triggered.
- **Issue** — the unit of work: description, status, assignee (person,
  agent, *or squad*), comments, runs, linked PRs. The board (kanban) is
  just issues grouped by status.
- **Squad** — a named group led by a leader agent. Assign an issue to the
  squad: the leader is woken, reads the issue, and hands the work to the
  right member with an `@`-mention. When a member posts an update, the
  leader is woken again and decides the next step. The coordination loop
  runs without you touching it.
- **Skill** — a reusable working method (a `SKILL.md` + optional files),
  attachable to many agents. Instructions say *who* an agent is; skills say
  *how* the team does a kind of work.
- **Project** — groups related issues and adds shared context (its
  description enters every run inside it) plus resources (repos, local
  directories).
- **Autopilot** — scheduled (cron) or webhook-triggered runs. In
  `create_issue` mode each trigger lands on the board as a normal issue.

## How this kit maps onto it

| Kit piece | Multica object |
|---|---|
| `workspace` in the manifest | a new or existing workspace |
| `project` | Multica project (shared description = team context) |
| `team.roles` | agents, one per role, with rendered `roles/*.md` instructions |
| `team.squad_name` | squad (leader = the `lead` role, members = the rest) |
| `team.skills` | workspace skills: local folders from `skills/`, plus the official `multica-cli` skill imported by URL |
| `autopilots` | autopilots with cron triggers; runbooks from `autopilots/*.md` |

Every agent gets four shared skills, which is where most of the "team
discipline" actually lives:

1. **`multica-cli`** (official) — how to operate the board safely through
   the CLI from inside a run (reading issues cheaply, posting comments,
   status/mention side effects).
2. **`kanban-contract`** — who may move which status, the no-`done` /
   no-merge rules, the end-of-run check.
3. **`handoff-protocol`** — the one-comment handoff anatomy and the exact
   mention mechanics that make agents actually trigger each other.
4. **`pr-conventions`** / **`delivery-report`** — how work reaches you:
   named/linked PRs and a fixed report format for `in_review`.

Role instructions (`roles/*.md`) add each role's specifics: what they own,
what they check first, what they may modify, and their boundaries. Squad
instructions (`squad/`) give the leader its routing table, rework-loop
policy, and the sign-off red line. The project description carries the
goal/boundaries that every run sees.

## Why this shape

- **Handoffs are explicit and auditable**: one comment with Done / Evidence
  / Questions / Ask, and an exact `@mention` (plain `@name` triggers
  nobody — the roster carries the real mention markdown).
- **The leader is a router, not a worker**: it can't hide implementation
  problems behind "the squad handled it", and you see every routing
  decision on the issue timeline.
- **Human checkpoints are structural, not a habit**: agents physically
  can't `done` or merge, so the only way work closes is through you.
- **Everything is re-runnable**: `apply` is idempotent and driven by the
  files in this repo — the team definition is versioned like code.