# SBW Release Check

A pre-launch security and compliance check for [Claude Code](https://claude.com/claude-code), made by a UI/UX designer who ships her own tools.

## Why I made this

I'm a UI/UX designer, not a career developer. Since starting [Small Batch Works](https://smallbatchworks.ca), I've been building and deploying real tools for local businesses and community groups (sign-up pages, admin dashboards, small web apps) with AI as my build partner.

Getting things to work was the easy part. What kept bothering me was what I couldn't see: Is that key supposed to be public? Can a stranger read the sign-up list? Am I following Canadian privacy rules?

So I treated the worry like a design problem and turned it into a checklist I run before anything goes live.

## How it was made

I talked my situation through with ChatGPT and Gemini, asked what someone in my position should check before launch, and pushed back where the answers didn't fit. That gave me two documents: a security audit and a compliance guide. I then asked Claude, which has been building my projects with me, to review them, correct the parts that didn't fit Saskatchewan, and merge them into one release check and one Claude Code skill.

## What it does

- **Part I: Security.** Secrets and environment variables, authentication, authorization, database rules (e.g. Supabase RLS), APIs, file uploads, dependencies, logging, abuse protection, browser and deployment settings, plus native-app and payment checks when they apply.
- **Part II: Compliance.** Privacy, anti-spam (CASL), accessibility, dark patterns and asset licensing, adjusted for Saskatchewan, Canada.
- **One verdict:** READY, READY AFTER FIXES, NOT READY, or INCONCLUSIVE.

Two rules matter most to me:

1. **It reports first and changes nothing** until I review and approve the fixes.
2. **It runs in a fresh session**, not the one that built the app, so the work isn't grading itself.

## What's in this repo

| Path | What it is |
| --- | --- |
| `sbw-release-check/SKILL.md` | The Claude Code skill (the working version) |
| `docs/release-check-standard.md` | The full standard: how it's applied, the original security audit, the Saskatchewan corrections, and the original compliance guide |

## How to use it

1. Copy the `sbw-release-check` folder into `~/.claude/skills/` (for all your projects) or into `.claude/skills/` inside one project.
2. Open the project you want to check in a **new** Claude Code session.
3. Run `/sbw-release-check`, or ask Claude to run the release check.
4. Answer the few questions it asks (who the client is, whether it holds real data, what access you can share).
5. Read the report. Approve the fixes you want, then run the check again.

## Using it outside Saskatchewan

The compliance part reflects Saskatchewan and Canadian rules as I understood them in November 2026 (PIPEDA, LA FOIP, CASL, The Accessible Saskatchewan Act, SLGA raffle rules). Some details come from secondary sources. If you're somewhere else, replace the "Saskatchewan corrections" with the rules that apply to you.

## Limits

- This is not legal advice and not a replacement for a professional security review. It's a way to make sure I ask the right questions every time.
- An AI reviewing AI-built code can miss things. Treat a READY verdict as "the known risks were checked", not "this can never be attacked".
- Laws change. Re-check the sources at least once a year.

Suggestions are welcome. If you're a developer and see something missing, please open an issue.

## License

[MIT](LICENSE) © 2026 Miyoung Cho
