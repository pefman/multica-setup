---
name: delivery-report
description: The standard report you post when you deliver work to in_review — the owner reads these to decide merge/approve/ship. Use every time you move an issue to in_review, and for delivery reports to the owner.
---

# Delivery report

When you deliver (move an issue to `in_review`), the owner gets exactly one
comment that lets them decide **merge / approve / ship / rework** without
re-reading your whole run. This is the format. The owner is busy; a report
they have to decipher is a failed delivery.

## Format

```
@<Owner> — delivery, ready for review

**Conclusion:** <one line: what this is and its state — "Pricing page
implemented and reviewed, ready to merge" or "QA passed, awaiting merge">

**What changed:**
- <2–5 bullets, outcome level, not commit level>

**Evidence:**
- PR: <link> (or: branch <name>, or: no code change — <artifact>)
- Verified: <build/lint/tests commands + result; screenshots/links where visual>

**Needs your call:**
- <decision 1: what, options, your recommendation>   (or: none)

**Next step (yours):** <merge PR #n / approve / ship / review findings above>
Nothing merges or ships without your approval.
```

## Rules

- **Conclusion first.** If the owner reads only the first line, they should
  know the state of the work. No "I've completed the task you requested…"
  openings.
- **Outcome level**: "pricing table renders with per-tier limits" — not
  "modified price.js lines 40–55". The owner reviews outcomes; the PR diff
  is there if they want depth.
- **Evidence is real and linked.** A PR link, a passing check with its
  command, a screenshot. "Tests pass" without the command or result is
  wallpaper.
- **Needs your call** is the only place for questions. If there are none,
  write `none` — the owner should not have to hunt for buried questions.
  Each decision: the question, the options, your recommendation with a
  reason.
- **Always end with the owner's next action.** One verb: merge, approve,
  ship, review. And always the reminder that merging/shipping is theirs.
- One report per delivery. If the owner comes back with changes, fix them,
  and post a **follow-up report** (same format, Conclusion: "All findings
  addressed, ready to merge") — do not edit the old comment.

## For squad deliveries

On squad-owned issues the lead writes the delivery report to the owner
(same format), and members report to the lead (handoff-protocol format)
instead. The owner sees one voice per issue: the lead's.

## Length budget

A delivery report fits in 15–25 lines. Longer than that and you are
restating the run log — cut it. The execution log already shows everything
you did; the report exists to say what it *means*.