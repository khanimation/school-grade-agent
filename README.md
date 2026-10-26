# 🐯 School Grade Agent

A Claude skill for parents. When the school emails a grade, Claude checks the parent portal, logs every class to Google Sheets, builds a printable homework checklist, and sends the family a short digest of where to focus.

**Progress over perfection.**

## What you get
- **Grade log:** one row per class per run, so you get a trend line.
- **Printable checklist:** this week and next week, MAJOR tags, and missing work that can still be turned in.
- **Family digest:** what moved, the one class to focus on, that teacher's tutoring hours, and one encouragement line backed by a real number.

## Install (about 20 minutes)
1. Turn on grade email notifications in your school's parent portal.
2. In Claude, connect **Gmail** and **Google Drive**, and install **Claude in Chrome**. Sign in to the portal in Chrome yourself.
3. Download `SKILL.md`, put it in a folder named `school-grade-agent`, and zip that folder.
4. In Claude, go to **Settings → Capabilities → Skills** and upload the zip.
5. Open `SKILL.md` and fill in the CONFIG block, or just say "run" and let Claude ask you for the details.

**Shortcut:** screenshot `grade-agent-starter-skill.png`, send it to Claude, and say *"Turn this into a skill and save it."*

## Rules it follows
- Never types a password. If a login asks for one, it stops and asks you.
- "Completed" is not graded. It only trusts a posted score.
- Emails only when a grade actually moves.

Built with Claude · Google Sheets · Gmail
