---
name: timesheet
description: Log daily work sessions by task — no time tracking, just clear records. Triggered by /timesheet or whenever the user mentions logging work, recording what they did today, tracking tasks for a session, or says anything like "log my session", "what did I work on", "end of day recap", or wants to record tasks for a specific date — even if they don't use the word timesheet explicitly.
---

# timesheet — Session Logger

Records what you worked on in a session. Task-based only — no time tracking, no durations, just clear plain-English records written to markdown files in your project.

## Invariants

- **Append-only on session content.** Every `## Session` block in `docs/timesheets/{date}.md` is immutable once written. New sessions are always appended to the end — never inserted or used to rewrite prior sessions.
- **Global day title is the one exception.** The first line of `docs/timesheets/{date}.md` is `# {date} — {global day title}`. This line is rewritten every time a new session is added so it always reflects all sessions for that day. Only this line changes — everything below it is untouched.
- **Append-only on the index.** `TIMESHEETS.md` rows for dates that already exist must not be rewritten. Only the `Sessions` count, `Day Title`, and `Last Session Title` columns on that date's row may change — all other existing row content stays exactly as written.
- A date with N existing sessions always produces session N+1 as the next entry. Count by reading the file; do not assume.
- **Extract work from the current conversation.** Do not ask the user what they worked on. Read the conversation history and extract completed work items from it — what was built, fixed, changed, debugged, analyzed, or discussed. Do not infer from file diffs or code changes that were not part of the conversation.
- Do not track time or durations. This skill is task-based only.
- Session titles must use plain language. Avoid technical jargon, acronyms, or internal IDs in titles.
- A session is scoped to a single conversation. Do not merge sessions across conversations.

## Global Day Title

The global day title is a noun-phrase title that captures the major work themes of the entire day. It is derived from the collective tasks and notes across every session — not just the latest one.

### How to derive it

1. Read all tasks and notes across every session in the file.
2. Cluster them into 2–4 major themes based on what kind of work was done (e.g. feature work, bug fixes, content changes, infrastructure, analysis).
3. Name each cluster as a concise noun phrase — not a verb, not a sentence.
4. Join them: `{Theme A}, {Theme B}, and {Theme C}`.

Good example:
> `Variant Integration, Content Processing, and Build Stability Improvements`

That title comes from tasks like: unifying a workflow button, fixing variant loading and path issues, fixing blog date display, adding table support, fixing content conversion, fixing image upload timeouts, committing variants, fixing blog rendering, clearing test data, and fixing import paths. The model clustered those into three themes rather than listing every task.

Rules:
- Re-derive from scratch after every new session by reading all tasks and notes in the file.
- Use noun phrases only — no verbs, no "Fixed X and Y", no sentence structure.
- 2–4 themes maximum. If everything belongs to one theme, one phrase is fine.
- Do not reference session numbers, dates, or timestamps.
- If only one session exists, the global title may match that session's title.

## Invocation

```
/timesheet {date}
/timesheet {date} {optional: extra context}
/timesheet view {date}
/timesheet summary {date}
/timesheet auto {date}
/timesheet auto {date} {optional: extra context}
```

## Argument Handling

- The command is always prefixed with `/timesheet`.
- `{date}` is the second token. Accepts MM-DD-YYYY, YYYY-MM-DD, or MM/DD/YYYY. Normalize to YYYY-MM-DD internally. If omitted, use today's date.
- `{extra context}` — everything after the date is optional supplemental text. When provided, it is merged with what was extracted from the conversation (not a replacement). Use this to add context that isn't visible in the conversation (e.g. a deployment that happened outside the chat).
- `view {date}` — displays all sessions for that date without writing anything.
- `summary {date}` — generates a short end-of-day summary from all sessions on that date.
- `auto {date}` — logs the current session then immediately runs summary.

## Workflow

### `/timesheet {date}` — Log a session

1. Read `AGENTS.md` if it exists.
2. Read `docs/timesheets/{date}.md` if it exists — count the existing `## Session` headings to determine N. The new session number is N+1.
3. Read `TIMESHEETS.md` if it exists — locate the row for `{date}` to verify session count. If the file does not exist, it will be created.
4. **Extract work from the current conversation.** Scan the full conversation history — from the first message to the one just before this invocation — and identify every distinct thing that was accomplished: features built, bugs fixed, changes made, problems debugged, topics analyzed, decisions reached. If extra context was provided after the date in the invocation, merge it in as additional items.
   - **If the conversation contains meaningful work context**, extract it and proceed without prompting the user.
   - **If the conversation is sparse or only contains the `/timesheet` invocation itself** (i.e., this skill was called in a fresh or unrelated conversation), do not fabricate notes. Instead, tell the user: *"I don't see enough work context in this conversation to log a session. Describe what you worked on and I'll log it — or invoke `/timesheet` at the end of the conversation where the work happened."* Then wait for their response before proceeding.
   - **Exclude from extraction:** session startup messages, CLI upgrade notices, tool invocations (including invoking the timesheet skill itself), system reminders, meta-conversation about Claude or its tools, and any infrastructure noise that is not actual work the user performed.
5. From the extracted work, produce:
   - A short, plain-language **session title** (max 8 words).
   - A numbered list of **notes** — one item per distinct completed task, plain English, past tense, with the "why" or outcome included where it adds clarity (e.g. `Fixed dynamic loading for committed variants — variants were not loading due to casing and path issues.`). **Produce a minimum of 5 notes when the conversation contains sufficient work.** Do not over-consolidate — if 8 distinct things were done, write 8 notes, not 3 broad ones. If the conversation genuinely contains fewer than 5 distinct work items, write only what is real and do not pad.
6. Append the new session block to the end of `docs/timesheets/{date}.md`.
7. Re-derive the **global day title** from all tasks and notes in the file (existing sessions + the one just appended). Rewrite only the first line of the file to reflect the updated title. Do not touch anything else.
8. In `TIMESHEETS.md`: if a row for `{date}` already exists, update only `Sessions`, `Day Title`, and `Last Session Title` on that row — leave every other row and every other column on that row unchanged. If no row exists for `{date}`, append a new row.
9. Confirm to the user with the session number, session title, and updated day title.

### `/timesheet view {date}`

1. Read `docs/timesheets/{date}.md` if it exists.
2. If no file exists, report: `No sessions logged for {date}.`
3. Display the global day title and all sessions cleanly. Do not modify any files.

### `/timesheet summary {date}`

1. Read `docs/timesheets/{date}.md`.
2. Write a 3–5 sentence plain-English summary covering all sessions.
3. Append the summary block to the end of `docs/timesheets/{date}.md`. Do not modify anything above the insertion point.
4. In `TIMESHEETS.md`, set `Summarized` to `Yes` on the row for `{date}` — leave all other columns and rows unchanged.

### `/timesheet auto {date}`

Run in sequence:

1. `/timesheet {date}` — extract work from the conversation and log the session.
2. `/timesheet summary {date}` — generate the end-of-day summary.

## File Format

### `docs/timesheets/{date}.md`

```markdown
# {date} — {global day title}

## Session 1 — {title}

### Notes

1. {what was done and why, one item per task}
2. {what was done and why}

---

## Session 2 — {title}

### Notes

1. {what was done and why}

---
```

The first line (`# {date} — {global day title}`) is the only line that is rewritten on each new session. All `## Session` blocks below it are permanent.

### Session block to append

```markdown
## Session {N} — {title}

### Notes

1. {what was done and why, one item per task}
2. {what was done and why}

---
```

A real example of a well-formed session:

```markdown
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
```

## TIMESHEETS.md Index Format

```markdown
# Timesheets

| Date       | Sessions | Summarized | Day Title                          | Last Session Title      |
| ---------- | -------- | ---------- | ---------------------------------- | ----------------------- |
| YYYY-MM-DD | N        | Yes / No   | {global day title for that date}   | {title of last session} |
```

Rules:

- One row per date.
- When adding a new session to an existing date: increment `Sessions`, update `Day Title` (re-derived from all sessions), and update `Last Session Title`. All other columns on that row are untouched. All other rows are untouched.
- When a summary is generated: set `Summarized` to `Yes` on that date's row only.
- Never delete rows.
- Never rewrite or reorder existing rows.

## Handoff

After logging:

```
Session {N} logged for {date}
Session title: {session title}
Day title:     {global day title}
Next: /timesheet summary {date}   — generate end-of-day summary
  or: /timesheet view {date}      — review all sessions
  or: /timesheet auto {date}      — log + summarize in one step
```
