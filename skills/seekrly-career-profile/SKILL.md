---
name: seekrly-career-profile
description: Read and update the user's Seekrly career profile - headline, summary, contact details, links, work history, education, skills, projects, job preferences (roles, salary, locations, notice period) and facts about the user. Use when the user wants to update their profile, add a new job or skill, change job preferences, or fill the profile from a resume they share.
---

# Career profile

Follow the ground rules in the `seekrly` skill. The profile feeds Seekrly's resumes,
autofill and job matching, so only save facts the user confirmed.

## Reading

`get_career_profile` returns JSON Resume sections (`basics`, `work`, `education`,
`skills`, `projects`, `certificates`, `awards`, `languages`, `volunteer`,
`publications`, `interests`, `references`) plus `jobPreferences`
(`preferredRole`, `yearsExperience`, `currentCtc`, `expectedCtc`, `employmentType`,
`shiftPreference`, `jobType`, `preferredLocations`, `noticePeriod` in days) and
`facts`: free-text notes the user wants Seekrly AI and autofill to know (e.g.
"Needs visa sponsorship for the US"). Follow them when you write for the user.
Each project has `links` (`{ name, url }`, e.g. "Live", "GitHub"), not one `url`.

## Updating

1. **Always read first** with `get_career_profile`.
2. Build the change with the same field names:
   - `basics`: only the fields passed change; `null` clears one. `location` fields
     merge. `profiles` replaces the whole list of links. Phone format:
     `+<country code> <number>`.
   - `jobPreferences`: fields passed replace those preferences; others stay.
   - `facts`: replaces the whole list. To add one, send the existing facts plus
     the new one.
   - **Sections replace the whole section.** To add one job, send the complete
     `work` list: existing entries plus the new one. Never send a partial list
     unless the user wants entries removed.
3. Show the change in the chat first (each field or section, before → after)
   and ask "Update it, or change something?" Say plainly if a section will lose
   entries: removed entries can't be restored.
4. After a yes, call `update_career_profile` once. It saves right away; don't ask
   again.

Not editable here: `basics.name`, `basics.email`, the photo, gender and date of
birth - the user changes those on the Seekrly Profile page.

## Filling the profile from a resume or details

If the user just tells you the details, map them to the sections above and
follow the update steps above. Anything about the user that fits no field (work
authorization or visa status, availability to start, willingness to relocate or
travel, driving licence, security clearance) goes in `facts`, one short
sentence each, added to the existing facts; never put something there that fits
another field. For a resume:

1. Ask the user to attach their resume (PDF or DOCX) or paste its text.
2. Read the whole file and map it to the sections above: dates `YYYY-MM`, bullets
   into `highlights`, skills grouped with `keywords`, a project's links into
   `links`. Details about the user with no field go in `facts` (see above).
   Copy everything exactly; leave out anything the resume doesn't say.
3. Compare with `get_career_profile` and show what would be added or replaced.
   Keep existing entries unless the resume clearly replaces them (a section
   replaces its whole list).
4. Follow the update steps above.
