---
name: school-grade-agent
description: Turns your school's grade-notification emails into a daily grade log in Google Sheets, a printable homework checklist, and a short family digest of which class to focus on and which teacher to see for tutoring. Uses Claude in Chrome on your school's parent portal plus Gmail and Google Sheets. Use when the user says "run" or "grade run", or when a new grade email arrives.
---

# School Grade Agent

A grade email lands. Claude checks the portal, logs every class, and tells the family what to do next.

**What you get each run**
1. A row per class in a Google Sheet (grade %, major/minor breakdown, missing, late), so you have a trend line.
2. A one-page printable checklist: everything due in the next ~2 weeks, plus anything missing that can still be turned in.
3. A short email digest: what moved, the one class to focus on, and the teacher's tutoring window.

---

## One-time setup (about 20 minutes)

1. **Turn on grade notifications** in your school's parent portal (Blackbaud, PowerSchool, Canvas, Skyward, etc.) so every new grade sends you an email.
2. **Connect Gmail and Google Drive** in Claude's connector settings.
3. **Install Claude in Chrome** and sign in to the parent portal in Chrome yourself. Claude uses your existing session. It never needs your password.
4. **Create a Google Sheet** named `Grade Tracker` with three tabs:
   - `Log`: date, class, teacher, grade_pct, major_pct, major_count, minor_pct, minor_count, missing, late
   - `Trend`: date, then one column per class (grade %)
   - `Upcoming`: due_date, day, class, assignment, category, points, status, source
5. **Fill in the CONFIG block below** and save the skill.
6. **Optional:** create a scheduled task that runs daily (for example, 4 PM). The run exits quietly if no new grade email arrived.

## CONFIG (fill this in)

```yaml
student_first_name: ""          # used in the digest greeting
portal_url: ""                  # the page that lists every class and its grade
notification_sender: ""         # e.g. notifications@yourschool.myschoolapp.com
grade_tracker_sheet_id: ""      # from the Sheet URL
checklist_drive_folder_id: ""   # where checklist PDFs are saved
digest_recipients: []           # parent/guardian emails (add the student if they're old enough)
run_time_zone: "America/Chicago"
grade_weighting:                # per class, if the portal doesn't show it
  default: { major: 0.6, minor: 0.4 }
late_work_policy:               # per teacher: accepts | no_late | case_by_case
  default: case_by_case
tutoring_hours: {}              # per teacher, e.g. "Mr. Smith: Mon/Wed 4:00–4:30"
```

---

## Rules (read first)

- **Never type a password, code, or credential.** If the portal or Google asks for one, stop and ask the user to sign in themselves.
- **Reuse the one browser tab** the user has open. Don't open new tabs or tab groups.
- **Dates are local.** Use `run_time_zone` for "today," filenames, and labels, never UTC.
- **One run per day in the log.** If you run twice on the same day, overwrite that day's rows instead of appending duplicates.

## Workflow

### 1. Gate on email
Search Gmail for `from:<notification_sender> newer_than:1d`. If nothing is new and the user didn't ask for a full refresh, report "no new grades" and stop. Open each new email and note the class and assignment it mentions.

### 2. Read the portal
In the user's tab, open `portal_url`. For each class, capture the overall %, the major and minor category % with item counts, and the counts of missing and late items. If a class has a grade-details view, open it. That's where the category breakdown lives.

### 3. Verify every new score
Open each assignment named in the emails and read its actual status and score.
- **"Completed" or "Submitted" is not a grade.** Count it only once a number is posted.
- **Ignore future-dated items** that already show a score until their due date.

### 4. Log it
Append one row per class to `Log`, and one row (date plus each class %) to `Trend`. Find the next empty row by looking, not from memory. Confirm the paste landed where you expected.

### 5. Build the upcoming list
Walk the portal's assignment calendar from today through the end of next Friday (or 9 days out, whichever is later), plus one more month for majors. Keep anything due in that window plus anything Missing or To-do. Write it to `Upcoming`.
If a teacher keeps their real schedule somewhere else (like Google Classroom), add those items too, mark their `source`, and let the portal date win on conflicts.

### 6. Printable checklist (one Letter page)
- Header with the date range and a "Checked by / Date" line.
- **Still owed** box: missing work that can still be turned in. Use `late_work_policy`; never list work from a `no_late` teacher as recoverable.
- **This week** and **Next week** sections, grouped by day, with checkboxes. Tag tests and projects **MAJOR**.
- **Later** footer listing upcoming majors beyond the window. Flag back-to-back crunches.

Save as `Checklist-YYYY-MM-DD.pdf` to `checklist_drive_folder_id`. Look at the rendered page before saving.

### 7. Family digest (only when something moved)
Send only if a grade posted that changed a class %, a category %, or added a zero. Send as HTML, written to the student in second person.

- **Lead:** one bold sentence with the number that matters.
- **Action list:** what's due tomorrow, what's missing, then dated majors.
- **Per class that moved:** `Class — X%` (Major X% on N, Minor X% on N, N missing) with ⬆/⬇ and the change, then:
  - 🎯 **Action:** the one specific thing to do
  - 📅 **Coming up:** the next major
  - *Note:* the category-level pattern (e.g. "quizzes 68%, homework 95%: it's test prep, not effort")
- **Focus pick:** the weakest category with the most points still ahead, plus that teacher's `tutoring_hours`.
- **Where it stands:** the overall average versus last run, in one line.
- **Encouragement (one line, every digest):** name one real improvement with its number, then the next habit. Example: "🎉 Back-to-back 100s in CS! You've climbed 16 points since August. That's what consistency looks like." Praise only what the data shows. No generic "great job."
- Attach the checklist PDF and link the Sheet.

**Target the gap, not perfection.** Show the math behind any "if you score X on the test" claim.

### 8. Report back
In a few lines: what moved, what was written where, what was emailed, and anything pending. Confirm that no password was entered.

---

## Privacy
Grades are a student's private record. Keep the Sheet and Drive folder shared only with family, and don't post screenshots with names or school details.
