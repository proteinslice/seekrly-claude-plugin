# Seekrly tools

"Confirm" = runs as soon as it's called: show the user what it will do and ask once
before the call. Every tool also needs `account:read`
(always granted).

## Account, profile and search

| Tool                    | What it does                                                                                                            | Permission                                    | Confirm |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------- | --------------------------------------------- | ------- |
| `get_profile`           | Connected account: name, email, plan                                                                                    | account:read                                  |         |
| `get_career_profile`    | Career profile in JSON Resume format + job preferences; `resumeId` puts that resume first (for filling one application) | profile:read (+ resumes:read with `resumeId`) |         |
| `update_career_profile` | Change headline, summary, contact details, preferences, facts or whole resume sections                                  | profile:update                                | Yes     |
| `search`                | Find applications and contacts by keyword                                                                               | account:read (+ area view)                    |         |

## Applications

| Tool                 | What it does                                                          | Permission          | Confirm |
| -------------------- | --------------------------------------------------------------------- | ------------------- | ------- |
| `list_applications`  | Filter by status, keyword, company (`companyId`), applied-date range  | applications:read   |         |
| `get_application`    | Full details: description, skills, status history, resume used, notes | applications:read   |         |
| `create_application` | Save a job (status defaults to Bookmarked); `companyId` adds it at one of the user's companies | applications:create |         |
| `update_application` | Change status or details (only the fields passed); `companyId` moves it to another company   | applications:update |         |
| `delete_application` | Delete with its interviews and notes; contacts and follow-ups stay    | applications:delete | Yes     |

## Companies

Made from the company name on applications and contacts (same name, ignoring case and
endings like Inc or Ltd, = same company); `create_company` is only for one the user wants
before any job or contact there. Companies have no permissions of their own: they use the
applications ones. A company with `inDirectory: true` takes its profile from Seekrly's
shared directory: only its name can change, and corrections go through
`request_company_change`. To add an application at a company, pass its `companyId` to
`create_application`.

| Tool                            | What it does                                                                                     | Permission                              | Confirm |
| ------------------------------- | ------------------------------------------------------------------------------------------------ | --------------------------------------- | ------- |
| `list_companies`                | Counts and stage; filter by name, stage, source, industry; sort by recent, name, applications    | applications:read                       |         |
| `search_companies`              | By name: the user's companies (`mine`), then directory companies they haven't saved              | applications:read                       |         |
| `get_company`                   | Profile (read-only when `inDirectory`), applications, people (works there or recruiting), notes  | applications:read                       |         |
| `create_company`                | Add one by `name` or `directoryId`; returns the saved one (`created: false`) if it already exists | applications:update                     |         |
| `update_company`                | Rename; edit website, careers page, links, industry, locations, description (only fields passed) | applications:update + applications:read |         |
| `delete_company`                | Delete with its notes; only when it has no applications or contacts                              | applications:delete + applications:read | Yes     |
| `find_duplicate_companies`      | Groups of the user's companies that look like one company saved twice                            | applications:read                       |         |
| `merge_companies`               | Move everything of `mergeIds` into `keepId`; their names land there from then on                 | applications:update                     | Yes     |
| `dismiss_duplicate_companies`   | "These are different companies": stop suggesting the group                                       | applications:update                     |         |
| `link_company_to_directory`     | Link to a directory entry (its profile then comes from there); `directoryId: null` unlinks       | applications:update                     |         |
| `suggest_company_for_directory` | Share an own company's public profile with the directory, after Seekrly's review                 | applications:update + applications:read | Yes     |
| `request_company_change`        | Send Seekrly a correction for a directory company (up to 10 a day)                               | applications:update + applications:read | Yes     |

## Contacts

| Tool             | What it does                                                                     | Permission      | Confirm |
| ---------------- | -------------------------------------------------------------------------------- | --------------- | ------- |
| `find_contacts`  | Filter by keyword, name, email, company (text or `companyId`), application       | contacts:read   |         |
| `get_contact`    | Phone, address, socials                                                          | contacts:read   |         |
| `create_contact` | Save a person, optionally linked to applications (`applicationIds`, any company) | contacts:create |         |
| `update_contact` | Edit details; `linkApplicationIds` / `unlinkApplicationIds`                      | contacts:update |         |
| `delete_contact` | Delete with their follow-ups                                                     | contacts:delete | Yes     |

## Interviews

| Tool               | What it does                                                                                                      | Permission                            | Confirm |
| ------------------ | ----------------------------------------------------------------------------------------------------------------- | ------------------------------------- | ------- |
| `list_interviews`  | Filter by application, outcome, dates; `upcoming=true`                                                            | interviews:read                       |         |
| `get_interview`    | One round, with notes                                                                                             | interviews:read                       |         |
| `create_interview` | Add a round (needs date, start time, timezone); moves the application to Interviewing; schedules a reminder email | interviews:create + applications:read |         |
| `update_interview` | Reschedule, change details, record outcome                                                                        | interviews:update                     |         |
| `delete_interview` | Delete one round                                                                                                  | interviews:delete                     | Yes     |

## Follow-ups (reminders on every plan; Email ones can send themselves with scheduledSend on paid plans)

| Tool               | What it does                                                                                                                                                                           | Permission                                                        | Confirm          |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- | ---------------- |
| `list_follow_ups`  | Filter by contact, application, status (Pending, Sent, Replied, No Response, Failed), due dates; `locked` says why only status can change                                              | followups:read                                                    |                  |
| `create_follow_up` | Plan a follow-up (Email, LinkedIn, Call, Whatsapp, SMS, In-Person) about one of the contact's applications (`applicationId`); `scheduledSend` (Email, paid plans) sends it at the time | followups:create + contacts:read (+ email:send for scheduledSend, resumes:read with `resumeId`) | scheduledSend    |
| `update_follow_up` | Reschedule, change draft, set status (e.g. Replied), turn `scheduledSend` on/off; channel fixed; sent/sending/auto-send emails: status only                                            | followups:update (+ email:send to turn scheduledSend on)          | scheduledSend on |
| `send_follow_up`   | Send a pending Email follow-up now, as drafted; returns `sending: true`, then it becomes Sent (or Failed with `sendError`)                                                             | followups:update + email:send + contacts:read                     | Yes              |
| `delete_follow_up` | Delete one                                                                                                                                                                             | followups:delete                                                  | Yes              |

Follow-up reminders work on every plan. Automatic sending (`scheduledSend`) is Pro and up, with a daily
limit per plan; on Free the follow-up is saved as a plain reminder (`send.state` `not_in_plan`).

## Outreach templates

| Tool | What it does | Permission | Confirm |
| --- | --- | --- | --- |
| `list_outreach_templates` | The user's saved templates and Seekrly's ready-made ones (`builtIn`), with `{firstName}`, `{name}`, `{company}`, `{role}`, `{myName}` fields and how many replies each got | followups:read | |
| `save_outreach_template` | Save a new template, or change one of the user's own (`templateId`) | followups:create (+ followups:update to change one) | |
| `delete_outreach_template` | Delete one of the user's own templates | followups:delete | Yes |

Writing a message from a template? Fill its fields (reword a sentence rather than leave a field empty)
and pass its id as `templateId` to `create_follow_up` or `send_email`, so its reply rate counts it.

## Notes

`target` is `application`, `contact`, `interview` or `company`. Every note tool also
needs view access to that record's area (company notes: applications:read).
`get_application`, `get_interview` and `get_company` return the newest 20 notes and
`notesTotal`; page the rest with `list_notes`. Each record (application, interview, contact or company) can hold as many notes
as the user's plan allows; at the cap `add_note` says so and suggests upgrading.

| Tool          | What it does                                           | Permission                      | Confirm |
| ------------- | ------------------------------------------------------ | ------------------------------- | ------- |
| `list_notes`  | Notes on an application, contact, interview or company, newest first; `limit` (≤ 100) and `offset` page them | notes:read + that area's view   |         |
| `add_note`    | Add a note                                             | notes:create + that area's view |         |
| `update_note` | Replace a note's text (returns the previous text)      | notes:update + that area's view |         |
| `delete_note` | Delete a note                                          | notes:delete + that area's view | Yes     |

## Resumes

| Tool              | What it does                                                                                                             | Permission     | Confirm                      |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------ | -------------- | ---------------------------- |
| `list_resumes`    | All resumes; `kind` = built (editable JSON) or uploaded (file only)                                                      | resumes:read   |                              |
| `get_resume`      | Built resume content (JSON Resume), template, sharing status and stats                                                   | resumes:read   |                              |
| `download_resume` | Short-lived download link (5 min) for the file                                                                           | resumes:read   |                              |
| `create_resume`   | Build a new resume from JSON Resume content; templates: modern, classic, minimal, twocolumn, timeline; optional `design` | resumes:create |                              |
| `update_resume`   | Rename / version / make default (direct); replace content, template or `design` of a built resume                        | resumes:update | Content/template/design only |
| `share_resume`    | Public or private share link (password-protected links are set up in Seekrly only, never through the app)              | resumes:share  | Public only                  |
| `delete_resume`   | Delete a resume and its file; its public link stops working                                                              | resumes:delete | Yes                          |

## Email

| Tool         | What it does                                                                                                                                                                                                                         | Permission                 | Confirm |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------- | ------- |
| `send_email` | Send from the user's Gmail/SMTP to a contact (`contactId`) or address (`email`, saved as a new contact), optional resume attachment; returns `sending: true` + `followUpId` (logged as a follow-up: Sent, or Failed with the reason). With `sendAt` it's scheduled instead (`sending: false`, an auto-send follow-up; paid plans) | email:send + contacts:read (+ followups:create with `sendAt`, contacts:create for a new address, resumes:read with `resumeId`) | Yes     |

| Tool               | What it does                                                                                                         | Permission | Confirm |
| ------------------ | -------------------------------------------------------------------------------------------------------------------- | ---------- | ------- |
| `get_email_status` | Whether Gmail/SMTP is connected (`configured`) and the plan's automatic sending (`autoSend`: included, daily limit); `settingsUrl` | email:send |         |

If Gmail/SMTP isn't connected, the tools return a Settings link - give it to the user.

## Support

| Tool              | What it does                                                                                                     | Permission   | Confirm |
| ----------------- | ---------------------------------------------------------------------------------------------------------------- | ------------ | ------- |
| `contact_support` | Message Seekrly's team (`topic`, `subject`, `message`): restore something deleted by mistake, a bug, billing, feedback; reply goes to the account email | account:read | Yes     |
