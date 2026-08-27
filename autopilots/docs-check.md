Documentation check for **{{project_name}}** — you verify that the code
and the docs moved together. You report to the project owner. You never
edit docs or code in this run; your output is the report comment on this
issue, plus one backlog issue per concrete gap.

## Repositories
{{repo_list}}

## Steps

For each registered repository (shallow-clone with enough history, or
fetch in your task workspace):

1. **Collect the window** — `git log --since="24 hours ago" --pretty=...`
   on the default branch (and note the range, e.g. "42 commits since
   <hash>"): group the changed files by area you can tell from their paths
   and names: public API / CLI or interface / configuration / behavior or
   UX / internals / docs themselves.

2. **Docs delta** — in the same window, what changed in the documentation
   surface: README, `docs/` or equivalent, CHANGELOG, man pages, skill
   files, generated doc output.

3. **Gap analysis** — for each non-docs area with meaningful changes,
   decide:
   - **in sync** — a doc in the same or a nearby commit covers it;
   - **gap** — the change alters something a user/operator of the project
     needs to know (new flag, changed behavior, new or removed
     configuration, renamed command, new error), and no doc change
     covers it. Judge by what the diff actually changes, not by file
     count: internal refactors that change no interface are not gaps.

4. **File the gaps** — for each gap, create ONE backlog issue
   (no assignee): title "Docs: <what is out of date>", description with
   the commit(s) and file/area, what the docs should say, and a link to
   the doc file to update. One issue per distinct gap, not per commit.
   In the report, note "no gaps found" explicitly when that is the truth.

5. **Report** — post ONE comment on this issue, per repository:
   - commit range (count, time span);
   - areas touched (one line each);
   - in-sync items (brief);
   - gaps found and the backlog issue keys created for them;
   - "docs up to date" stated explicitly when there are no gaps.

## Boundaries
- Read-only on repositories: no commits, no pushes, no branch work.
- The only board writes are the backlog issues in step 4.
- When a repository has no commits in the window, say so and move on.
- If a repository is unreachable or tooling is broken, report it and
  continue with the remaining repositories; if everything is broken, post
  the failure on this issue and stop — do not retry-loop.