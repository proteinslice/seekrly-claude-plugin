---
name: seekrly-tailor-resume
description: Build or tailor a resume in Seekrly - tailor an existing resume to a job, create a new resume from the career profile or from a resume file the user shares, change its template, change its look (font size, margins, spacing, colors) or fit it on fewer pages, download or share it. Use for "tailor my resume for this job", "make a resume for…", "turn this PDF into a Seekrly resume", "download/share my resume".
---

# Build and tailor resumes

Follow the ground rules in the `seekrly` skill. Never invent experience, employers,
dates, degrees, numbers or skills - only use facts from Seekrly or from the user.

## Know the two kinds

`list_resumes` returns each resume's `kind`:

- **built** - made in Seekrly; content is JSON Resume, readable with `get_resume`
  and editable with `update_resume`.
- **uploaded** - the user's own PDF/DOC file; content can't be read or edited here.
  To work from it, ask the user to share the file or paste its text in the chat, then
  build a new resume with `create_resume`.

## Tailor to a job

1. **Get the job**: from the chat, or the tracked application (`get_application`
   → `description`, `skills`).
2. **Get the source**: the user's chosen built resume (`get_resume` →
   `resumeData`), or the career profile (`get_career_profile`) if there's none.
3. **Plan the changes** and show them briefly before saving:
   - Reorder and rewrite highlights to lead with what the job asks for, keeping the
     user's facts and numbers.
   - Sharpen `basics.label` (headline) and `basics.summary` for the role.
   - Surface matching skills that the user really has; don't add ones they don't.
   - List any gaps honestly ("the job wants Kubernetes; nothing in your profile
     mentions it - want to add something true?").
4. **Show it, then save.** Show the tailored resume before saving: the headline,
   summary, each job with its highlights, skills, and what changed from the
   source. Ask "Save it, or change something?" Then save. Prefer a **new**
   resume so the original stays intact:
   `create_resume` with `name` like "Backend Engineer - Stripe", the full
   `resumeData`, and a `template` (modern, classic, minimal, twocolumn, timeline;
   keep the source's template and `design` unless asked). Replacing an existing resume's content
   uses `update_resume`, which saves right away: send the **complete** JSON Resume
   document (not a diff), and confirm the change with the user first.
5. Offer to link it to the application (`update_application` with `resumeId`) or to
   attach it to an outreach email (`seekrly-outreach`).

## Build from a file the user shares

Read the whole file and map it into JSON Resume: `basics` (name, label, email,
phone, url, summary, location, profiles), `work` (one entry per job; bullets into
`highlights`), `education`, `skills` (groups with `keywords`), `projects`,
`certificates`, `awards`, `languages`, `volunteer`, `publications`. Keep the user's
wording, dates as `YYYY-MM`, and leave out what isn't there. Before calling
`create_resume`, show what you extracted (sections and entry counts, anything
unclear) and ask them to confirm.

## Look and page fit

Both `create_resume` and `update_resume` take an optional `design`; with
`update_resume` only the fields you send change (ask first, like content).

- `fontSize`: `xxsmall`, `xsmall`, `small` (default), `medium`, `large`.
- `margin` (page margins): `minimal`, `narrow`, `normal` (default).
- `sectionSpacing` (space between sections and entries): `tight`, `compact`,
  `normal` (default).
- `primaryColor`, `textColor`, `backgroundColor`: hex colors like `#2563eb`.

When the user wants fewer pages or a denser resume ("fit on one page", "less
white space", "smaller margins"), try `margin: "narrow"` and
`sectionSpacing: "compact"` first, then `minimal` / `tight`, and only then a
smaller `fontSize`, so the text stays readable. Don't cut their content to make
it fit unless they ask.

## Labels, download and sharing

- Rename, version label or make default: `update_resume` with `name`, `version` or
  `makeDefault` - applies directly.
- File: `download_resume` returns a link valid for 5 minutes; give it only to the
  user and fetch a new one if it expired.
- Public link: `share_resume` with `visibility` `public` (ask first; explain who can
  see it) or `private` (immediate). `get_resume` shows the current link, whether it's
  password-protected, and view/download stats.
- Password-protected link: never ask for, suggest or handle a password. The user
  sets one in Seekrly on the Resumes page (the link is in the `share_resume`
  description). Making a password-protected resume public removes its password, so
  say so before the call.
