# Changelog

All notable changes to this project are documented here.

Follows [semantic versioning](https://semver.org/): MAJOR.MINOR.PATCH

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
