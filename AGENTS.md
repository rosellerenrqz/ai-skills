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

## Frontmatter MUST be valid YAML (critical)

The `skills` CLI discovers skills by parsing each `SKILL.md`'s YAML frontmatter — it does **not** use `marketplace.json` for discovery. If the frontmatter fails to parse, the CLI **silently skips the entire skill** (e.g. it reports "Found 1 skill" instead of 2), so the slash command never installs and is never available in the agent.

The most common breakage is the `description` value. A plain (unquoted) scalar must not contain:

- `: ` (colon followed by a space) — YAML reads it as the start of a new mapping key → `bad indentation of a mapping entry`
- embedded `"` or `'` quotes

Safe patterns for `description`:

- **Preferred — folded block scalar.** Lets you use colons and quotes as literal text:
  ```yaml
  description: >-
    Suggest a conventional-commit message from staged changes. Examples — "write my commit message", "name a branch".
  ```
  (Note: write `Examples —` with an em-dash, not `Examples:`.)
- Or keep it a single plain line with **no `: ` and no embedded quotes**.

After adding or editing any skill, verify the frontmatter parses and confirm discovery before pushing:

```sh
# 1. Validate YAML frontmatter parses
node -e 'const fs=require("fs"),yaml=require("js-yaml");const fm=fs.readFileSync("skills/<name>/SKILL.md","utf8").match(/^---\n([\s\S]*?)\n---/)[1];console.log(yaml.load(fm).name)'

# 2. Confirm the CLI discovers ALL skills (count should match skills/ folder count)
#    A clean-dir dry run prints "Found N skills"
```
