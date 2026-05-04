# AGENTS.md

This repository contains reusable AI skills installable via `npx skills add`.

## What this repo is

A collection of skills for Claude Code, Codex, Cursor, and other AI coding agents.
Each skill lives in `skills/{skill-name}/SKILL.md` and can be installed individually
or all at once.

## For agents working in this repo

- Skills are in `skills/` — one folder per skill, each with a `SKILL.md` and a `README.md`
- Do not modify `SKILL.md` files unless explicitly asked
- When adding a new skill, copy the structure of an existing one — including its `README.md`
- Keep skill descriptions specific and trigger-friendly — they are the primary
  routing mechanism that determines when a skill is activated
- Every skill folder must have a `README.md` that documents: what the skill does,
  how to install it, all commands with examples, and what output files it creates

## Skill naming conventions

- Folder name: lowercase, hyphens only (e.g. `timesheet`, `daily-standup`)
- `name` in frontmatter: matches the folder name
- `description` in frontmatter: describes both what it does AND when to trigger it
