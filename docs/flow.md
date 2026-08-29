# The flow: kanban + handoffs

The working agreement of every team this kit creates. It is enforced three
ways: agent instructions, shared skills, and squad instructions — so it
survives no matter which agent picks up an issue.

## The board

| Column | Status | Who puts work here |
|---|---|---|
| Backlog | `backlog` | owner / lead (triage). **Parking lot — nothing triggers.** |
| To Do | `todo` | lead/owner, once scoped with acceptance criteria. Assigning an agent here **starts** it — the lead only puts the first task of a new scope here on the owner's go in the start-decision ticket. |
| In Progress | `in_progress` | the agent working it, set at the start of the work. |
| In Review | `in_review` | the agent delivering. **The top of the agent world.** |
| Done | `done` | **the owner.** No agent may set it. |
| Blocked | `blocked` | anyone, always with a comment naming the blocker and who must clear it. |
| Cancelled | `cancelled` | owner (lead only with the owner's go-ahead). |

## One issue's life (typical repo work)

```
owner/lead: write the issue, scope it            → backlog
lead:        plans subtasks (parked, unassigned) → backlog
lead:        opens start-decision ticket, @owner → todo (owner's to answer)
owner:       replies @lead: go                   → lead assigns first subtask
lead:        routes to the engineer (@mention, adds constraints)
engineer:    works, opens PR  ISSUE-12 <summary> → in_progress
engineer:    hands off (@reviewer, Done/Evidence/Questions/Ask)
reviewer:    fresh-eyes pass → findings, or "ready to merge"
            · findings  → @engineer rework (same PR, same branch)
            · clean     → hand off to owner (or lead, who reports)
( qa:        runs it, verifies acceptance criteria → pass/fail report )
lead/agent:  delivery report to owner            → in_review
owner:       reviews in browser, merges PR, closes → done (or sends back = rework)
```

Who routes depends on `team.routing` in the manifest:

- `hybrid` (default): ambiguous work → squad (the lead routes); scoped work
  may go straight to the right agent — fewer hops.
- `leader-only`: everything goes through the squad; one entry point.

## The handoff (the whole game)

A handoff is **one comment** in the issue timeline:

```
@<NextOwner> — handoff

Done: one line — what you just delivered.
Evidence: PR link / commands + results / files / screenshots.
Questions: open questions, or "none".
Ask: what the next owner should do, concretely.
```

Rules that make it work:

- The next owner is triggered by the **exact mention markdown** from the
  squad roster. Typing `@name` by hand triggers nobody — agents are
  instructed to copy the roster's mention syntax.
- One next owner per comment. `@all` is clutter, not coordination.
- No evidence, no handoff. A handoff you can't back with a link or command
  output is a request to redo the work.
- Status and comment move together: deliver → `in_review` + handoff comment.
  Neither without the other.
- When the squad leader is in the loop, members report back to the leader
  (it is re-woken automatically on member comments) and the leader issues
  the next handoff — you only ever talk to the lead and to the owner-level
  reports.

## The red lines (all agents)

1. **Never merge a PR.** The owner merges (an agent may merge only after
   the owner explicitly instructs it for a specific PR in an issue).
2. **Never set `done`.** Deliveries end at `in_review`.
3. **Never deploy** — except the release agent executing a deploy the owner
   approved, per the documented repo procedure.
4. **`blocked` is a comment, not a status**: always paired with what is
   needed and from whom.
5. **One active issue per agent** (default concurrency 1): the board shows
   the queue, agents don't juggle behind it.
6. A run that only answers questions leaves every status untouched.
7. **The lead never self-starts new scope.** The first task of a scope
   starts only on the owner's reply in the start-decision ticket
   @-mentioning the lead; the approval lives on that ticket, not in chat.

## What you do (the human loop)

- **Intake**: drop requirements in as issues (or ask the helper agent to
  draft them). Everything starts in `backlog`; the lead's standup surfaces
  what's ready to scope.
- **Watch**: the board + your inbox. The daily standup issue tells you the
  state of the world every morning; deliveries `@`-mention you.
- **Go / no-go**: the lead parks the plan and opens a start-decision ticket
  assigned to you. Your reply @-mentioning the lead is the recorded go —
  without the @-mention the lead never hears it. Close the ticket when you
  are done deciding.
- **Review & merge**: open the PR (or ask for a local server), verify,
  merge or send it back with `@role` comments. Merge + close = `done`.
- **Tune the team**: when an agent keeps making the same mistake, fix its
  `roles/<role>.md` (or a skill) and re-run `apply` — the correction lands
  on every future run.

## Recurring automation

- **Daily standup** (lead, `create_issue` mode): inventory, in-flight scan,
  blocked/stale flags, "needs your decision" list — one report comment,
  read-only on the board.
- **Weekly hygiene** (report-only pass): status reconciliation against
  merged PRs, stale flags, repo-hygiene findings (concrete ones become
  `backlog` issues), disk-usage report with prune *candidates* (it never
  deletes).

Both runbooks live in `autopilots/` and are fully editable; add your own
by dropping a `SKILL.md`-style runbook file and referencing it in the
manifest.