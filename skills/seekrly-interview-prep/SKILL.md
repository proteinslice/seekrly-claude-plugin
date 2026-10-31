---
name: seekrly-interview-prep
description: Schedule, reschedule and prepare for interviews tracked in Seekrly, and record how they went. Use when the user has an interview coming up, got an interview invite, wants to practise or prepare for a round, needs to move an interview, or wants to log the outcome or feedback.
---

# Interviews

Follow the ground rules in the `seekrly` skill.

## Scheduling a round

1. Resolve the application (`search` / `list_applications`). No application yet?
   Create it first (`seekrly-track-job`).
2. Collect: `round` (e.g. "Recruiter screen", "System design"), `type` (Video,
   Phone, In-Person, Assessment, Other), and the **date, start time and timezone**.
   If the time or timezone is missing or ambiguous, ask - never assume. Read
   numeric dates back in words ("Tuesday 6 October?").
3. Optional: `meetingLink`, `location`, interviewer `contactId` (from
   `find_contacts`), and a first `note` (agenda, prep).
4. Show the round before saving: application, round, type, date and time in
   words with the timezone, link or location, interviewer and note. Ask "Add
   it, or change something?"
5. After a yes, `create_interview` with `scheduledAt` as `YYYY-MM-DDTHH:MM:SS±HH:MM`. It moves
   the application to Interviewing (if earlier) and schedules the user's reminder
   email. Confirm with the date and time in words.

Rescheduling or changing details: `update_interview` with only the changed fields
(a new `scheduledAt` needs date, time and offset; the reminder re-arms).

## Preparing

1. Gather context: `get_interview` (round, type, notes), `get_application`
   (job description, skills, history), the interviewer if linked (`get_contact`,
   notes), and the user's background (`get_career_profile` or their resume).
2. Build a focused prep pack for *this* round:
   - What the round likely covers, given the type and the job description.
   - Likely questions, with talking points drawn from the user's real experience
     (STAR outlines for behavioural ones).
   - Gaps between the job and the profile, and how to address them honestly.
   - Smart questions to ask the interviewer.
   - Logistics: time in words, link/location.
3. Offer a mock interview: ask one question at a time and give feedback.
4. Offer to save the prep as a note on the interview: show the note text first,
   then `add_note` (or, after `list_notes`, `update_note`) to update an
   existing prep note (`update_note`, after `list_notes`).

## After the interview

- Outcome: `update_interview` with `outcome` (Passed, Failed, No Show, Cancelled).
- Feedback and impressions: `add_note` on the interview.
- Next step: schedule the next round, update the application status, or plan a
  thank-you / follow-up email (`seekrly-outreach`).
