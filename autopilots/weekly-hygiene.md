Weekly board and repository hygiene for **{{project_name}}**. You are the
{{squad_name}}'s periodic maintenance pass, reporting to the project owner.
Mostly report-only: status changes only where the runbook below explicitly
allows them, and only toward `done` with evidence. You never merge, deploy,
delete, or archive anything.

## Repositories
{{repo_list}}

## Steps

1. **Inventory** — `multica issue list --limit 200 --output json`
   (paginate if needed). Count per status.

2. **Status reconciliation vs merged PRs** — for the registered repositories,
   list recently merged PRs (e.g. `gh pr list --repo <owner/repo> --state
   merged --limit 30 --json number,title,mergedAt` if GitHub CLI is
   available; otherwise compare via `multica issue pull-requests <id>`). For
   each non-done issue, check its linked PRs: if its PR is merged and the
   work is not superseded by a newer issue, flip the issue to `done` with
   `--no-start` and note the PR number as evidence. Never flip to any other
   status, and never flip an issue whose PR is not merged.

3. **Stale flag** — for each issue sitting in `in_review` or `in_progress`
   for more than 3 days, list it with days stuck (use the issue's timeline /
   updated_at). Flag only — do not advance or reassign.

4. **Repo hygiene pass** — for each registered repository (shallow-clone or
   fetch as appropriate, in your task workspace): dead code or orphaned
   files, TODO/FIXME clusters that have aged, dependencies that are far
   behind their upstream (report only — do not bump), convention drift that
   a future cleanup issue could target. Findings go into the report as
   candidates for new backlog issues — create a backlog issue only for
   findings that are concrete and actionable (name the file/area, state the
   fix), one issue per finding, `backlog` status, no assignee.

5. **Disk hygiene (runtime host)** — run `multica daemon disk-usage` and
   `du -sh` on the daemon's workspaces root. Report total size, task-dir
   count, top-5 largest entries. FLAG if total > 5 GB or the volume is > 80%
   full; when flagged, list prune candidates (task dirs older than 7 days
   whose issue is done/cancelled) in the report — report only, never delete.

6. **Report** — post ONE comment on this issue:
   - one-paragraph health summary;
   - counts by status;
   - status fixes applied, each with its evidence (PR # or check);
   - stale items with durations;
   - repo hygiene findings and any backlog issues created (keys);
   - disk section (total, top-5, flag state, prune candidates if flagged);
   - "all clear" stated explicitly if nothing needed doing.

7. **Close this issue** — `multica issue status <this issue> done
   --no-start`. The report is the deliverable; leaving run issues open
   only clutters the board.

## Boundaries
- Status changes: only `→ done` with merged-PR evidence, plus closing
  this issue in step 7; always `--no-start`.
- Never cancel, delete, or archive issues; never reassign; never merge PRs,
  deploy, or touch production; never delete runtime files.
- If tooling or the runtime is broken, post the failure on this issue and
  stop — do not retry-loop.