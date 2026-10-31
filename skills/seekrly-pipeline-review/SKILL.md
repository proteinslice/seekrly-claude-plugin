---
name: seekrly-pipeline-review
description: Review the user's job search in Seekrly - what's due today or this week, upcoming interviews, follow-ups to send, stale applications and overall progress. Use for "what should I do today", "weekly review", "how's my job search going", "what's coming up" or "who haven't I heard back from".
---

# Review the job-search pipeline

Follow the ground rules in the `seekrly` skill. This skill only reads data; it
changes nothing unless the user then asks for an action.

## Gather (in parallel where you can)

- **Upcoming interviews**: `list_interviews` with `upcoming: true` (and `to` = end of
  the period for "this week").
- **Follow-ups due**: `list_follow_ups` with `status: "Pending"` and `dueTo` = today
  (or the end of the period). Overdue ones are Pending with a past date.
- **Needs a nudge**: `list_follow_ups` with `nudgeDue: true` - sent follow-ups nobody
  has replied to. Offer to draft the next message or `snooze_follow_up_nudge`.
- **Failed sends**: `list_follow_ups` with `status: "Failed"` - emails that
  didn't go out (`sendError` says why). Surface these first.
- **Active pipeline**: `list_applications` per relevant status - `Applied`,
  `Interviewing`, `Negotiating`, `Offer` - or without a filter for everything
  (newest activity first). Page with `offset` if `total` is larger.
- **Stale applications**: `Applied` with an old `appliedDate` and no recent
  `updatedAt` (e.g. 14+ days) - candidates for a follow-up or `No Response`.

If an area isn't shared (permission error), skip it and mention it once at the end.

## Present

Lead with what needs action, most urgent first. A good shape:

1. **Today / overdue** - failed sends, interviews today, follow-ups due or overdue.
2. **This week** - upcoming interviews (date, time in words, company, round, link).
3. **Waiting to hear back** - applied, no movement for a while; suggest a follow-up.
4. **Pipeline at a glance** - counts by stage (e.g. "12 applied · 3 interviewing ·
   1 offer").
5. **Suggested next steps** - 2-4 concrete actions you can do for them ("draft a
   follow-up to Priya at Stripe", "prep for Thursday's system design round").

Keep it scannable: short bullets, dates in words, no raw ids or ISO timestamps.
Don't pad with generic job-search advice.

## Acting on it

When the user picks an action, hand off to the matching skill: follow-ups and
emails → `seekrly-outreach`, interviews → `seekrly-interview-prep`, status changes
→ `seekrly-track-job`.
