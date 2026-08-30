# The flow: kanban + handoffs

The working agreement of every team this kit creates. It is enforced three
ways: agent instructions, shared skills, and squad instructions — so it
survives no matter which agent picks up an issue.

## The board

| Column | Status | Who puts work here |
|---|---|---|
| Backlog | `backlog` | owner / lead (triage). **Parking lot — nothing triggers.** |
| To Do | `todo` | lead/owner, once scoped. Assigning an agent here **starts** it — the lead only puts the first phase of a new Feature here on the owner's go in the start-decision ticket. |
| In Progress | `in_progress` | the agent working it, set at the start of the work. |
| In Review | `in_review` | the agent delivering. **The top of the member world**; Feature parents land here for the owner. |
| Done | `done` | owner, or lead for board admin (feature close-out after approval + merge; start-decision / kickoff / finished phases / autopilot runs). Members never set it. |
| Blocked | `blocked` | anyone, always with a comment naming the blocker and who must clear it. |
| Cancelled | `cancelled` | owner (lead only with the owner's go-ahead). |

## One Feature's life (typical repo work)

```
owner pitch / lead scopes MVP as one Feature     → backlog (parent)
lead:  creates 3 staged children (parked):
         stage 1 Research → Engineer
         stage 2 Implement → Engineer (docs on same PR)
         stage 3 Verify → QA (or Reviewer)
lead:  opens start-decision ticket, @owner       → todo (owner's to answer)
owner: replies @lead: go                         → lead assigns Research
engineer Research: approach + risks, no PR       → handoff to lead
lead:  assigns Implement
engineer Implement: branch FEATURE-n-slug, PR    → in_progress
engineer: hands off to lead (@mention)
lead:  assigns Verify
qa/reviewer: pass/fail on same PR                → handoff to lead
lead:  delivery report on Feature parent         → parent in_review
owner: reviews, merges PR                        → lead closes parent done
```

Trivial one-shot fixes may skip the three-phase tree (single issue) — the
lead judges. Follow-up ideas become **new Feature parents** after delivery,
not extra siblings under the current feature.

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
  (it is re-woken automatically on member comments and when a stage barrier
  completes) and the leader issues the next handoff — you only ever talk to
  the lead and to the owner-level reports.
- Do not open sibling tickets to hand off; rework stays on the same PR.

## The red lines (all agents)

1. **Never merge a PR.** The owner merges (an agent may merge only after
   the owner explicitly instructs it for a specific PR in an issue).
2. **Members never set `done`.** Deliveries end at `in_review`. The lead
   may close admin/delivery tickets with evidence (`done --no-start`).
3. **Never deploy** — except the release agent executing a deploy the owner
   approved, per the documented repo procedure.
4. **`blocked` is a comment, not a status**: always paired with what is
   needed and from whom.
5. **One active issue per agent** (default concurrency 1): the board shows
   the queue, agents don't juggle behind it.
6. A run that only answers questions leaves every status untouched.
7. **The lead never self-starts new scope.** The first task of a Feature
   starts only on the owner's reply in the start-decision ticket
   @-mentioning the lead; the approval lives on that ticket, not in chat.
8. **One Feature = three phases max** at plan time (plus start-decision).
   No scaffold/wire/polish/Docs siblings for the same outcome.
9. **Out-of-role asks are refused.** Agents only take work that matches
   their role contract; otherwise one short refuse + point at the right
   owner (`role-contract` skill).

## What you do (the human loop)

- **Intake**: drop requirements in as issues (or ask the helper agent to
  draft them). Everything starts in `backlog`; the lead's standup surfaces
  what's ready to scope.
- **Watch**: the board + your inbox. The daily standup issue tells you the
  state of the world every morning; deliveries `@`-mention you.
- **Go / no-go**: the lead parks the Feature (three phases) and opens a
  start-decision ticket assigned to you. Your reply @-mentioning the lead
  is the recorded go — without the @-mention the lead never hears it. The
  lead closes the ticket after recording the decision.
- **Review & merge**: open the PR (or ask for a local server), verify,
  merge or send it back with `@role` comments. After merge + your approval,
  the lead closes the Feature parent.
- **Tune the team**: when an agent keeps making the same mistake, fix its
  `roles/<role>.md` (or a skill) and re-run `apply` — the correction lands
  on every future run.

## Recurring automation

- **Daily standup** (lead, `create_issue` mode) — **on by default in
  `init`**: inventory, in-flight scan, blocked/stale flags, "needs your
  decision" list — one report comment, read-only on the board.
- **Opt-in** (add to the manifest or enable in the wizard customize step):
  weekly hygiene, stall-radar, pr-patrol, docs-check. Hygiene and
  docs-check can file backlog issues; keep them off until you want that.

Runbooks live in `autopilots/` and are fully editable; add your own by
dropping a runbook file and referencing it in the manifest.
