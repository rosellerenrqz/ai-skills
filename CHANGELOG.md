# Changelog

All notable changes to this project are documented here.

Follows [semantic versioning](https://semver.org/): MAJOR.MINOR.PATCH

---

## [1.3.0] — 2026-06-30

### Added — `git-message`
- New skill. `/git-message` suggests a conventional-commit message from staged changes (`git diff --cached`), choosing the commit type and scope from the actual diff. `/git-message -branch` also suggests a `{type}/{kebab-summary}` branch name and, when you're on the default branch, shows the `git switch -c` command to move the work onto its own branch.
- **Read-only by design** — runs only inspection commands (`git status`, `git diff`, `git branch`, `git log`) and prints suggestions to copy. It never runs `git commit`, `git checkout`, `git add`, or any state-changing command.
- Messages omit `Co-Authored-By` / `Generated with Claude Code` trailers. Subject lines are imperative, ≤ 72 chars, with a body only when the change needs explaining.
- When nothing is staged, it offers to base the suggestion on unstaged changes instead of guessing silently.

---

## [1.2.0] — 2026-06-30

### Added — `timesheet`
- `/timesheet -time {file}` — estimates work hours from note complexity and writes a per-session estimate plus a day total. Three complexity tiers (light/standard/heavy), summed per session, floored to a minimum of 8 hrs for the day total, rounded to the nearest half hour. Idempotent — managed time lines are replaced in place on re-run. This is the only command that records time; default logging stays time-free.
- `/timesheet -clean {file}` — removes git commit-recommendation content (conventional-commit subject lines, commit bodies, `Co-Authored-By` / `Generated with Claude Code` trailers) that was accidentally captured into session notes, then renumbers remaining notes and re-derives the day title. Legitimate work notes that merely mention committing are preserved.
- `{file}` argument for both commands accepts a full path or a date that resolves to `docs/timesheets/{date}.md`.

### Changed — `timesheet`
- Extraction now explicitly excludes git commit-message recommendations so proposed commit text is never logged as work, even when the user asked for a commit message in the same conversation.

---

## [1.1.0] — 2026-05-04

### Changed — `timesheet`
- Invocation is now `/timesheet {date}` (slash command prefix)
- Session notes are automatically extracted from the current conversation — no user input required
- Notes are written in plain English only; file names, function names, and code identifiers are excluded
- Added global day title: a noun-phrase theme derived from all sessions for the day, updated on each new session
- Added minimum of 5 notes when sufficient work exists in the conversation; over-consolidation is no longer allowed
- System noise (session startup messages, CLI upgrade notices, tool invocations) is excluded from extracted notes
- Optional extra context can be passed inline after the date to supplement conversation extraction
- Added per-skill `README.md` documenting commands, usage examples, and output format
- Added fallback behavior: when conversation context is too sparse, the skill reports it rather than fabricating notes

---

## [1.0.0] — 2026-05-04

### Added
- Initial release
- `timesheet` skill (v1.0.0) — log daily work sessions by task
- `.claude-plugin/marketplace.json` with schema versioning and per-skill versions
- Local install instructions for use on any device without the marketplace CLI
