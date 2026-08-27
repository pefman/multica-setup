---
name: pr-conventions
description: How this team names and writes branches and pull requests, and the hard rule that agents never merge. Apply whenever you open, update, or touch a pull request.
---

# PR conventions

Pull requests are how finished work reaches the owner's eyes. A PR that
doesn't follow these conventions won't link to its issue, won't show CI on
the board, and the owner can't tell what it is without opening it.

## Naming

- **Branch**: `<issue-prefix>-<issue-number>-<short-kebab-slug>`
  e.g. `MP-42-add-pricing-page`. The issue identifier goes first — that is
  what the board matches on.
- **Title**: `<ISSUE-PREFIX>-<n> <imperative summary>`
  e.g. `MP-42 Add pricing page`. One line, what the change delivers. The
  issue identifier in the title is **mandatory** — without it the PR will
  not auto-link to the issue and the board's "Pull requests" panel stays
  empty.
- **Body**: four short sections:
  - **What** — 2–4 lines: what changed and why (the issue's ask, in your words).
  - **How** — the approach in a sentence or two; link the issue.
  - **Verified** — what you ran (build/lint/tests commands) and the results;
    screenshots or links where the change is visual or behavioral.
  - **Open points** — decisions you made that the owner might want to
    override, or "none".

## Rules

- **One issue → one PR.** If a review finds changes, push to the **same
  branch/PR** — never open a second PR to fix a first one.
- **You never merge.** Not even a green CI, not even a "ready to merge"
  review, not even if the owner says "it's fine, merge it" in a vague
  comment — the owner performs the merge (or explicitly assigns the merge
  to you in an issue, in writing, for that PR). If the owner @-mentions you
  with "merge it", confirm which PR, then merge **only that one**, and
  report the merge on the issue.
- **Small is better.** If a branch grows beyond what the issue asks, split
  the extra work into a new issue instead of smuggling it in.
- **Keep the branch current**: rebase onto the default branch before
  requesting review if it has drifted; re-resolve conflicts yourself, they
  are part of the delivery.
- No secrets, no generated artifacts, no `.env` files, no WIP commits with
  secrets in their messages. The last commit is what the reviewer reads.
- After the owner merges, your follow-up (if the issue asks) is to clean up
  (delete the branch via the merge, or note it) — then report on the issue.

## When you have no PR to open

Not every deliverable is code. If your work produced docs, a runbook, or a
board-only outcome, the "PR" is your delivery comment (delivery-report
skill) — same discipline: identify the issue, give evidence, name the
open points.