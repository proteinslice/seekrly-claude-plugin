---
name: seekrly-track-job
description: Save a job to the user's Seekrly tracker or move it through the pipeline. Use when the user pastes a job posting, link or screenshot and wants to save, bookmark or track it, says they applied, got rejected, got an offer, or otherwise changes an application's status. Also for companies: everything at one company, adding an application or a company, editing, merging, deleting or suggesting a company for the shared directory.
---

# Track a job in Seekrly

Follow the ground rules in the `seekrly` skill (ids, confirmations, dates).

## Saving a new job

1. **Extract** from what the user shared - only what's actually there:
   `company`, `role` (job title), `jobUrl`, `companyUrl` (the same job on the
   company's own careers site, when `jobUrl` is a job board that sends applicants
   there), `location`, `salary`, `experience`
   (e.g. "3-5 years"), `employmentType` (Full-time, Contract…), `skills` (short
   keywords, max 50), `description` (the job description text), `contactPerson`.
   Don't invent missing fields; leave them out.
2. **Check for duplicates**: `search` with the company name (and role). If the same
   company + role already exists, show it and ask whether to update that one instead.
3. **Pick the status**:
   - Just saving / "bookmark this" → `Bookmarked` (the default).
   - "I applied" → `Applied`, with `appliedDate` (`YYYY-MM-DD`, not in the future).
     Ask for the date if it wasn't today and they didn't say.
   - Mid-application → `Applying`.
4. **Resume used** (only when applied): if the user names one, resolve it with
   `list_resumes` and pass `resumeId`; otherwise the default resume is recorded.
5. **Show what you'll save** and ask "Save it, or change something?": role,
   company, status, applied date (in words), location, salary, experience,
   employment type, skills and the job link, plus the first lines of the
   description. Leave out fields you don't have.
6. After a yes, call `create_application` and confirm in one line: role,
   company, status.
7. **Offer next steps** (one short line, don't do them unasked): add the recruiter
   as a contact (`create_contact` with `applicationIds`, or `update_contact` with
   `linkApplicationIds` if they're already saved), plan a follow-up, or
   tailor a resume for it (`seekrly-tailor-resume`).

If the posting names a recruiter or hiring manager with an email or LinkedIn URL,
offer to save them as a contact linked to the application (check `find_contacts`
first).

## Updating status

1. Resolve the application (`search` or `list_applications` with `query`). If more
   than one matches, ask which.
2. Say what will change and ask (one line is enough, e.g. "Mark Backend Engineer
   at Acme as Rejected?"), then call `update_application` with only the changed
   fields, e.g.
   `{ applicationId, status: "Rejected" }`. Moving to Applied or later without an
   `appliedDate` on record: pass one (ask if unknown).
3. For interviews use `create_interview` (see `seekrly-interview-prep`) - it moves the
   application to Interviewing by itself.
4. Useful extras when relevant: a note with the reason or feedback (`add_note` on the
   application), or marking related follow-ups done.

## Bulk updates

For "mark everything with no reply for 30 days as No Response": list first
(`list_applications`, filter by status/date), show the user the matching list and
ask before updating each one.

## Companies

Applications and contacts with the same company name (ignoring case and endings
like Inc or Ltd) share one company, so saving a job creates its company. Find a
company's id with `list_companies` (`query`) or `search_companies`.

- **Everything at one company** ("what do I have at Acme?"): `get_company` -
  profile, applications, people, interviews, follow-ups, notes and look-alike
  companies. Filter `list_applications` or `find_contacts` by `companyId` for more.
- **Add an application at a company** ("add another role at Acme"): collect the
  job as in "Saving a new job", then `create_application` with that `companyId`
  (and the company's name in `company`). To move an application to another of the
  user's companies, `update_application` with the new `companyId`.
- **Add a company before any job there** ("track Acme, I want to work there"):
  `search_companies` first. Already saved (`mine`)? Use it. In the directory? Offer
  `create_company` with that `directoryId` (its profile comes from there); otherwise
  `create_company` with `name` and any website, careers page or industry the user gave.
  Show what you'll save first.
- **Edit a company**: `update_company` with only the fields that change (name,
  website, careersUrl, socials, industry, locations, description; `socials` and
  `locations` replace the whole list). Show before → after first. A company with
  `inDirectory: true` only takes a new name; for wrong details, offer
  `request_company_change` (say what's wrong, give the right values in `fields`;
  two-step: show the preview, repeat with the token only after a clear yes).
- **Directory**: `get_company` may return `directoryMatch` for an own company - offer
  `link_company_to_directory` with that id (its profile then comes from the directory;
  `directoryId: null` unlinks). To share an own company with other Seekrly users,
  `suggest_company_for_directory` (two-step; only the public profile is shared, never
  the user's applications, people or notes, and Seekrly reviews it first).
- **Saved twice under different names**: `find_duplicate_companies`, show the groups
  and ask which name to keep. Then `merge_companies` with `keepId` and `mergeIds`
  (two-step: show the preview, repeat with the token only after a clear yes).
  Applications, contacts and notes move to the kept company. If the user says a group
  is really different companies, `dismiss_duplicate_companies` so it isn't suggested
  again.
- **Delete a company**: only one with no applications or contacts; otherwise offer to
  merge it. `delete_company` is two-step and also deletes its notes.
- **Notes about a company** (culture, people, why the user is interested): `add_note`
  with `target: "company"`.
