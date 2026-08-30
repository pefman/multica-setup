---
name: kanban-contract
description: The status rules of this team's board — which status means what, who is allowed to move an issue where, and the hard boundaries around done/merge/deploy. Consult before changing any issue status, and whenever you finish a run.
---

# Kanban contract

The board is the single source of truth for "where is the work". Every
status change you make must be **true at the moment you make it** and
**consistent with who you are**.

## The columns and what they mean

| Status | Meaning | Who starts it |
|---|---|---|
| `backlog` | Parked. Not scoped or not yet wanted. **Nothing is triggered here** — assigning an agent to a backlog issue does not run it. | owner / lead (triage) |
| `todo` | Scoped and ready to start. Assigning/mentioning an agent here **starts a run immediately**. | lead (triage) or owner |
| `in_progress` | Actively being worked. The agent that is working it sets this **at the start** of its work. | the working agent |
| `in_review` | A result exists and waits for the next eyes (lead or owner). This is the **highest status any member may deliver to**. | the delivering agent |
| `blocked` | Cannot continue: waiting on the owner, another role, or the outside world. Always paired with a comment that names what is needed and from whom. | any agent, owner |
| `done` | Closed out. **Members never set this.** The lead may set `done --no-start` for board administration (feature close-out after owner approval + merged PR; start-decision / kickoff / finished phase children / autopilot-run issues). The owner may always set it. | owner / lead (admin only) |
| `cancelled` | No longer pursued; the record stays. Owner (or lead with owner's go-ahead for scope cuts). | owner / lead |

There is no fixed flow between statuses — but the ownership table above is
fixed. If you are not the role that owns a transition, you don't make it.

## Feature shape (how work is packed)

Default unit of work is a **Feature parent** with three staged children:
Research (stage 1) → Implement (stage 2) → Verify (stage 3). One branch and
one PR per feature, keyed to the parent id. The Feature parent is what
moves to `in_review` for the owner; phase children deliver to the lead.
Trivial one-shot fixes may be a single issue — the lead decides.

## The rules that never bend

1. **Members never set `done`.** Delivery ends at `in_review`. The lead
   closes admin/delivery tickets with evidence as above; if the owner asks
   a member to "close it out", the member reports and leaves `done` to the
   lead or owner.
2. **Agents never merge PRs and never deploy.** Those are owner actions
   (deploy may be *executed* by the release agent **after** the owner
   approves, per the release role's rules).
3. **`in_progress` belongs to exactly one agent at a time.** If your run
   ends without producing the issue's deliverable (you answered questions,
   or you blocked), leave the status as you found it — do not claim the
   issue is still "being worked" when your turn is over.
4. **`blocked` is a comment, not a status.** A blocked issue without a
   recent comment saying what is needed and from whom is a dead issue. Set
   it only together with that comment.
5. **One active issue per agent.** Your concurrency is 1: if you pick up
   `todo` work while another issue of yours is `in_progress`, finish or
   block the first one first. The board shows the queue; don't juggle
   behind it.
6. **Status follows the deliverable, not the effort.** "I worked hard on it"
   is not `in_progress` forever; "the PR is open and verified" is
   `in_review` on the Feature parent, not `done`.

## Finishing a run: the 3-second check

Before you stop, answer three questions:
- Did I produce the issue's deliverable? → delivered: `in_review` + handoff
  comment (handoff-protocol skill); not delivered: status as found, or
  `blocked` with a comment.
- Is the status I leave **true in my absence**? Someone reading the board
  without any context from you should understand exactly where things stand.
- Did I `@` the next owner (or the owner) in a comment? If there is a next
  step and no mention, you have not finished.

## When the platform and you disagree

- A failed run with no other runs in flight rolls `in_progress` back to
  `todo` automatically — that's the platform, not a mistake you made.
- A merged PR with close intent can flip an issue to `done` automatically.
  Since this team never merges without the owner, that only happens with the
  owner's act — but if you see an issue you didn't close sitting in `done`,
  say so on the issue.
- Custom statuses (if the workspace defines any, e.g. `code_review`) inherit
  their category's behavior: a `code_review` status in the `in_review`
  category is a valid delivery target and works like `in_review`.
- Stage barriers: when every child in a Multica `--stage` group finishes,
  the Feature parent's assignee (usually the lead) is woken to route the
  next phase — that is expected, not a bug.
