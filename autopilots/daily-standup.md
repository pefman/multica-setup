Daily standup for **{{project_name}}** (workspace board review). You are
reporting to the project owner. This run is **read-mostly**: you never
change another issue's status, never reassign, never post to other issues.
Your output is the report comment on this issue.

## Steps

1. **Inventory** — list the board: `multica issue list --limit 200 --output json`
   (paginate with `--offset` if there are more). Count per status:
   backlog / todo / in_progress / in_review / blocked / done (7 days) /
   cancelled (7 days).

2. **In-flight scan** — for every `in_progress` issue: who is working it,
   what it is (one line each). Flag any that has had no activity (comment,
   run, or status change) for more than 24 hours — list it as "possibly
   stuck".

3. **Blocked and stale** — list every `blocked` issue with its blocker
   (read the latest comment). List `in_review` items older than 3 days as
   "waiting on you" — these are the owner's queue. Flag-only: never
   auto-advance, unassign, or ping other agents about them.

4. **Needs the owner** — collect every open question or decision that
   currently belongs to the owner (from blocked issues, `in_review`
   deliveries, and `Questions:` sections of recent handoffs). Deduplicate;
   present as a numbered list, each with the issue key and the ask.

5. **Report** — post ONE comment on this issue:
   - one-paragraph health summary (where the project stands, momentum up or
     down, the single most important thing the owner should look at);
   - counts by status;
   - in-flight list (issue key — owner — one line; possibly-stuck marked);
   - blocked list with blockers;
   - "waiting on you" list (in_review > 3 days);
   - "needs your decision" list.
   "All clear, nothing to do" is a valid report — say so explicitly.

6. **Close this issue** — `multica issue status <this issue> done
   --no-start`. The report is the deliverable; leaving run issues open
   only clutters the board and inflates the counts you report tomorrow.

## Boundaries
- Read-only on the board, except closing this issue in step 6: no status
  changes on other issues, no assignments, no comments on other issues, no
  agent triggering.
- If a tool or the runtime is broken, post the failure on this issue and
  stop — do not retry-loop.