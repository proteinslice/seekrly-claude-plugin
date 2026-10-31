---
name: seekrly-outreach
description: Reach out to recruiters, hiring managers and referrals through Seekrly - draft and send emails from the user's own Gmail/SMTP (optionally with a resume attached), plan and update follow-ups, and keep contacts up to date. Use for "email the recruiter", "follow up with…", "ask for a referral", "remind me to ping…", "they replied".
---

# Outreach and follow-ups

Follow the ground rules in the `seekrly` skill. Emails go out from the user's own
account and **can't be unsent** - always draft first and get a clear yes.

## Who

1. Resolve the person: `find_contacts` (name, email, company) or `search`.
2. Context helps the message: the linked applications (`get_application`), past notes
   (`list_notes` on the contact), and the user's background
   (`get_career_profile`).
3. No saved contact? `send_email` accepts `email` (and optional `name`,
   `applicationId`) and saves them as a new contact - mention that to the user.
4. Before the first email in a conversation, `get_email_status`. If `configured` is
   false, tell the user and give them `settingsUrl` to connect Gmail or SMTP before
   drafting. `autoSend.included` says whether follow-ups can send themselves on
   their plan.

## Drafting

- Short and specific: who the user is, the role (with company), one or two concrete
  points of fit from their real profile, and a clear ask. Usually under 150 words.
- Match the user's tone; don't overclaim or invent shared history.
- Plain text body; blank lines separate paragraphs; no HTML.
- Show the full draft in the chat before any tool call: to, subject, the whole
  body, attachment and sender. Ask "Send this, or change something?" Revise and
  show it again until they approve. Never send a draft the user hasn't seen.

## Sending (two-step)

1. `send_email` with `contactId` **or** `email`, `subject`, `body`, optional
   `resumeId` (from `list_resumes`) and `sender` (auto/gmail/smtp).
2. The first call returns a preview + `confirmationToken`. Show it in full:
   recipient, subject, the whole body and the attachment. Ask "Send it?"
3. Only on a clear yes, repeat the identical call with the token. The result is
   `sending: true` with a `followUpId`: the email goes out within seconds. Say
   "Sending to …", not "Sent". If the user asks whether it went out, check
   `list_follow_ups` for that contact: Sent, or Failed with `sendError`. If
   Gmail/SMTP isn't connected, give the user the Settings link from the error.
4. Offer to plan a follow-up (below).

To send it later instead ("send it Monday at 9"), add `sendAt` to the same
two-step call: a date and time with the user's timezone offset (ask for the
time if they didn't give one). It's saved as a follow-up that sends itself
then (`sending: false`, `sendAt`, `followUpId`); say "Scheduled for …".
`update_follow_up` reschedules it, or cancels it with `scheduledSend: false`.
Scheduling is a Pro feature: on Free the call fails; tell the user and offer
to send it now (the same call without `sendAt`).

## Follow-ups

- Plan: `create_follow_up` with `contactId`, `channel` (Email, LinkedIn, Call,
  Whatsapp, SMS, In-Person), `scheduledDate` (`YYYY-MM-DD`, or with a time and
  offset), optional `subject`/`message` draft and, for Email, a `resumeId`.
  A typical gap is 5-7 business days after an application or email.
  **Before calling, show the follow-up:** contact, channel, date and time in
  words, whether it sends automatically, and the full subject and message. Ask
  "Create it, or change something?" 
- By default a follow-up is a reminder and nothing is sent. Only when the user
  asks for the email to go out by itself, pass `scheduledSend: true` on an Email
  follow-up with a time in `scheduledDate`: Seekrly sends it from their
  Gmail/SMTP then (right away if the time has passed). It returns a preview
  first; confirm with the user, then call again with the token. Automatic
  sending is a paid feature (Pro and up) with a daily limit: on Free the
  follow-up is saved as a normal reminder (`send.state` `not_in_plan`) - tell
  the user they'll need to send it themselves, or upgrade. Plain reminders work
  on every plan.
- Update: `update_follow_up` - reschedule, edit the draft, set `status`
  (`Replied` when they answered, `No Response`, `Sent`) or turn `scheduledSend`
  on/off (turning it on asks for confirmation). The channel can't change: plan
  a new follow-up for another channel. An Email follow-up that was sent, is
  sending or is set to auto-send is frozen - only its status changes; to edit an
  auto-send email, turn `scheduledSend` off first. `list_follow_ups` returns
  `locked` with the reason when a follow-up is frozen; check it before offering
  to edit the draft.
- Send a planned email now: `send_follow_up` (confirm first). It sends the saved
  draft as is, so first show that draft in full (from `list_follow_ups`: to,
  subject, message, attachment) and offer to edit it with `update_follow_up`.
  Like `send_email` it returns `sending: true`; it then shows as Sent, or Failed
  with `sendError`.
- A Failed follow-up: tell the user the `sendError` and offer to fix the cause
  (e.g. reconnect Gmail) and send again. A follow-up's email is sent at most once
  (`send.state` `already_sent`); to send a new message, plan a new follow-up.

## Keeping contacts tidy

- New person: `create_contact` (check `find_contacts` first), link to the
  applications they help with in `applicationIds`; `company` is where they work (an
  agency recruiter's agency, not the job's company). Show the details you'll save first.
- Already saved, now helping with another job: `update_contact` with
  `linkApplicationIds`.
- Changes (new title, email, LinkedIn URL): `update_contact` with only those fields.
- Log what happened ("Call with Priya: next round in 2 weeks") with `add_note` on the
  contact.

## Stop or ask

- No recipient address on the contact: ask for it and save it with
  `update_contact` before sending.
- Never send, schedule an auto-send or mark something Sent on the user's behalf
  without their clear yes to that exact message.
- Don't invent shared history, referrals or claims about the user; ask instead.
