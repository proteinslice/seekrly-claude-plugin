# Seekrly for Claude

Seekrly is your career command center: applications, recruiter contacts,
interviews, follow-ups, resumes and your career profile in one place. This
plugin connects Claude to your Seekrly account, so Claude can act on your real
job-search data instead of starting from scratch.

It adds the Seekrly MCP server (`https://seekrly.com/mcp`) and skills that
teach Claude how to use it safely.

## What you can do

- **Track jobs:** save a posting from any site and keep every application's
  stage up to date.
- **Review your pipeline:** see what's due this week, upcoming interviews and
  leads that have gone quiet.
- **Tailor resumes:** build ATS-ready resumes for each role and share them.
- **Reach out:** write to recruiters and hiring managers from your own email,
  and plan follow-ups.
- **Prepare for interviews:** schedule rounds, practise and record how each
  one went.
- **Keep your career profile:** experience, skills and wins that every resume
  is built from.

Try: "What's due in my job search this week?", "Save this job posting to my
tracker", "Tailor my resume for this job".

## Install

In Claude Code:

```
/plugin marketplace add <owner>/<repo>
/plugin install seekrly@seekrly
```

Then run `/mcp`, choose `seekrly` and sign in with your Seekrly account. You
need a Seekrly account: [seekrly.com](https://seekrly.com).

## Your data

- **Sign-in and permissions.** You sign in to Seekrly with OAuth and choose
  what Claude may view or change on the consent screen. It starts at view
  only. Change or revoke access any time in Seekrly → Settings → Security.
- **What Seekrly receives:** only the requests needed for what you asked, such
  as a job posting to save or an email you approved. Seekrly never receives
  your conversation with Claude or your chat history.
- **What Claude receives:** only the Seekrly data needed to answer you, within
  the permissions you granted. Never your password, email credentials or API
  keys. Resume download links expire after 5 minutes.
- **Sensitive actions need your OK.** Sending an email, deleting anything and
  rewriting your profile or a resume each show you a preview and wait for a
  clear yes.
- **No training.** Seekrly never uses what you send through Claude to train AI
  models or for advertising.

The plugin itself runs no code on your computer. It only points Claude at the
Seekrly server.

Privacy policy: [seekrly.com/privacy](https://seekrly.com/privacy) · Terms:
[seekrly.com/terms](https://seekrly.com/terms) · Docs:
[seekrly.com/docs/mcp](https://seekrly.com/docs/mcp) · Support:
[connect@seekrly.com](mailto:connect@seekrly.com)

## For maintainers

```
.claude-plugin/plugin.json        manifest (name, version, icon, privacy URL)
.claude-plugin/marketplace.json   lists this plugin at "./" so the repo is installable
.mcp.json                         the MCP server (http, https://seekrly.com/mcp)
skills/                           copied from ../chatgpt-plugin/skills; don't edit here
```

- Edit skills in `../chatgpt-plugin/skills`, then copy them here:
  `rm -rf skills && cp -R ../chatgpt-plugin/skills skills`. Check they match
  with `diff -r skills ../chatgpt-plugin/skills`.
- Check before publishing: `claude plugin validate . --profile directory`.
- Test locally: `claude --plugin-dir .`, then `/mcp` → `seekrly` → sign in.
  Claude Code signs in through a callback on `localhost`; check this works
  against production before publishing.
- Bump `version` in `.claude-plugin/plugin.json` for every release, together
  with `../chatgpt-plugin/plugin.json`.
