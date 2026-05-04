# timesheet

Log daily work sessions by task — no time tracking, no durations, just clear plain-English records written to markdown files in your project.

---

## Install

**Via marketplace CLI:**
```bash
npx skills add rosellerenrqz/ai-skills --skill timesheet
```

**Globally (available in all projects):**
```bash
npx skills add rosellerenrqz/ai-skills --skill timesheet -g
```

**Manual (no CLI needed):**
```bash
git clone https://github.com/rosellerenrqz/ai-skills.git
cp -r ai-skills/skills/timesheet ~/.claude/skills/   # global
# or
cp -r ai-skills/skills/timesheet .claude/skills/     # project-local
```

---

## Commands

```
/timesheet {date}                          Log a session — extracts work from the conversation automatically
/timesheet {date} {extra context}          Log a session, merging extra context with the conversation
/timesheet view {date}                     Display all sessions for a date
/timesheet summary {date}                  Generate an end-of-day summary
/timesheet auto {date}                     Log + summarize in one step
```

`{date}` accepts any of these formats: `MM-DD-YYYY`, `YYYY-MM-DD`, `MM/DD/YYYY`. Omit it to default to today.

---

## Usage examples

### Log a session

```
/timesheet 05-04-2026
```

No input needed. Claude reads the current conversation and extracts every distinct thing that was accomplished — features built, bugs fixed, changes made, problems debugged — and writes them as the session notes automatically.

> **Invoke this at the end of the conversation where the work happened.** The skill reads the current conversation window only — it has no access to work done in a different session or chat. If invoked in a fresh conversation with no prior context, it will tell you rather than fabricate notes.

### Log a session with extra context

If something happened outside the conversation (e.g. a manual deployment, a call, an action taken in another tool), add it after the date and it will be merged in:

```
/timesheet 05-04-2026 also deployed to staging and cleared the Redis cache manually
```

### View sessions for a date

```
/timesheet view 05-04-2026
```

Displays all sessions logged for that date. Nothing is written.

### Generate an end-of-day summary

```
/timesheet summary 05-04-2026
```

Appends a 3–5 sentence plain-English summary of all sessions to the date file.

### Log + summarize in one step

```
/timesheet auto 05-04-2026
```

Logs the current session (asks for notes if not provided inline), then immediately generates the summary.

---

## What gets created

### Session file — `docs/timesheets/{date}.md`

Each date gets its own file. The first line is a **global day title** — a 2–4 theme noun phrase that reflects all sessions for that day, updated each time a new session is added.

```markdown
# 2026-05-04 — Variant Integration, Content Processing, and Build Stability Improvements

## Session 1 — Variant Integration and Blog Content Fixes

### Notes

1. Unified "Analyze Suggestions" and "Create Page" into a single button — full page creation workflow now happens in one click.
2. Fixed dynamic loading and file naming for committed variants — variants were not loading due to casing and path issues.
3. Fixed blog date display — was showing full timestamp instead of just the date.
4. Added table support and blog post detail page handling.
5. Fixed heading styling preservation in content conversion.
6. Fixed image upload timeout during variant commits.
7. Analyzed token cost per variant and per page across the workflow.
8. Committed multiple new custom variants to the client site.
9. Debugged and fixed blog post detail page rendering error.
10. Cleared out test blog data from the studio.
11. Fixed section path reference for how-it-works variant imports.
12. Fixed statistics section type alias in the content structure.

---

## Session 2 — ...

### Notes

1. ...

---
```

**Rules:**
- Sessions are append-only — existing entries are never modified or removed
- The global day title (line 1) is the only line that updates, re-derived from all sessions each time
- Session N+1 is always based on counting existing `## Session` headings in the file

### Index — `TIMESHEETS.md`

A top-level index of all logged dates:

```markdown
# Timesheets

| Date       | Sessions | Summarized | Day Title                                                         | Last Session Title                      |
| ---------- | -------- | ---------- | ----------------------------------------------------------------- | --------------------------------------- |
| 2026-05-04 | 2        | No         | Variant Integration, Content Processing, and Build Stability ...  | Variant Integration and Blog Content... |
```

---

## Global day title

The title at the top of each date file is a noun-phrase summary of everything worked on that day — not a list, but a grouped theme title.

| Sessions | Example title |
| -------- | ------------- |
| 1 | `Variant Integration and Blog Content Fixes` |
| 3+ | `Variant Integration, Content Processing, and Build Stability Improvements` |

It is re-derived from scratch after each new session by clustering all tasks into 2–4 themes. It never contains verbs, session numbers, or dates.

---

## Notes format

Notes within each session are a numbered list — one item per distinct thing done, past tense, with the "why" included when relevant:

```
1. Fixed dynamic loading for committed variants — variants were not loading due to casing and path issues.
2. Fixed blog date display — was showing full timestamp instead of just the date.
3. Analyzed token cost per variant and per page across the workflow.
```

---

## Version

`1.1.0` — see [CHANGELOG](../../CHANGELOG.md)
