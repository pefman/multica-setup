---
name: handoff-protocol
description: How work changes hands on this team. Use whenever you finish work on an issue and the next step belongs to someone else, or when you are handed work and need to know what to do with it.
---

# Handoff protocol

Work on this team moves by **one comment at a time**. A handoff is complete
only when the next owner has been triggered and knows exactly what is asked.
A status change without a handoff comment is a dropped ball.

## Anatomy of a handoff comment

Post a single comment in the issue's timeline (never split it across several
comments). It has four labeled parts, in this order:

```
@<NextOwner> — handoff

Done: one line — what you just delivered.
Evidence: PR link(s), command output, file paths, screenshot — whatever the
         next owner needs to see without re-doing your work.
Questions: open questions you could not settle, or "none".
Ask: what the next owner should do, as one or two concrete steps.
```

Rules:
- Keep it short. If `Done` needs more than two sentences, you are restating
  context the next owner can read on the issue — cut it.
- `Evidence` is mandatory for anything the next owner must trust: a PR link,
  a passing test run, a screenshot. No evidence, no handoff.
- `Ask` is a verb phrase: "review PR #42", "verify the pricing page",
  "merge and ship". Never "let me know what you think".

## Mention mechanics (this is where handoffs die)

- Trigger someone by `@`-mentioning them **with the exact mention markdown
  from your squad roster** (e.g. `[@Name](mention://agent/<uuid>)`). A plain
  `@name` typed by hand triggers **nobody**.
- Mention exactly the person who does the next step. Mentioning extra people
  wakes extra runs and clutters the timeline.
- When you are handed work *to you* by an `@mention`, the issue (or the
  comment you were mentioned in) is your task. Do it, then hand off per the
  protocol. Do not re-mention your sender to acknowledge — your working
  badge and execution log are the acknowledgment.

## Handoff chains

Default shape: Feature phases Research → Implement → Verify, then owner.
- Research (engineer) finishes → `@` lead (lead assigns Implement).
- Implement (engineer) finishes → `@` lead (lead assigns Verify) — or `@`
  reviewer/QA when the issue was assigned to you directly.
- Verify fails → `@` engineer with repro steps; rework on the **same PR**.
- Verify passes → `@` lead; lead posts the delivery report on the
  **Feature parent** and moves the parent to `in_review` for the owner.
- Reviewer (code review on a direct assign) says "changes requested" → `@`
  engineer with numbered findings. "Ready to merge" → `@` lead or owner.
- The **owner merges** — no agent ever merges.

On **squad-owned** issues the lead is the router: members report back to the
lead (the lead is re-triggered automatically on member comments and when a
stage barrier completes) and the lead issues the next handoff. On issues
assigned to you **directly**, you hand off to the next role or the owner
yourself.

Do not create new issues to hand off work. Phases already exist under the
Feature; rework and polish stay on the same PR.

## When the next step is "the human"

The owner is a member of the workspace. `@`-mention the owner in the comment
and state, explicitly: what you need from them (merge, approve, decide,
credentials) and by when if it matters. Then leave the issue in the status
that reflects the wait (`in_review` for delivered work, `blocked` when you
cannot proceed at all). Do not nudge again unless a full day passes — one
clear ask lands in their inbox.

## Anti-patterns (do not do these)

- Moving a status to `in_review` without the handoff comment (or the other
  way around).
- "Work complete!" with no evidence and no ask.
- `@all` or mentioning the whole team to signal anything.
- Answering a question and then also delegating the question to someone
  else — pick one: answer it (if it's yours) or hand it off (if it's not).
- Handing off work you have not verified at all. A handoff you cannot back
  with evidence is a request to redo it.
- Opening a sibling ticket for docs, polish, or "next steps" instead of
  handing off — say it in the handoff; the lead opens a new Feature later
  if needed.
