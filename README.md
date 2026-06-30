# ai-skills

A growing collection of reusable AI skills for Claude Code, Codex, Cursor, and other coding agents.

Install any skill with one command — no config, no setup.

---

## Available Skills

| Skill         | Version | Description                                                            | Docs |
| ------------- | ------- | ---------------------------------------------------------------------- | ---- |
| `timesheet`   | 1.2.0   | Log daily work sessions by task — no time tracking, just clear records | [README](skills/timesheet/README.md) |
| `git-message` | 1.3.0   | Suggest a conventional-commit message (and branch name) from staged changes — read-only | [README](skills/git-message/README.md) |

---

## Install

### Option A — Via marketplace CLI (recommended)

Requires the `skills` CLI to be available. Install all skills or pick one:

```bash
# All skills
npx skills add rosellerenrqz/ai-skills --all

# Single skill
npx skills add rosellerenrqz/ai-skills --skill timesheet

# Globally (available in every project)
npx skills add rosellerenrqz/ai-skills --skill timesheet -g

# For a specific agent
npx skills add rosellerenrqz/ai-skills --skill timesheet -g -a claude-code
npx skills add rosellerenrqz/ai-skills --skill timesheet -g -a codex
npx skills add rosellerenrqz/ai-skills --skill timesheet -g -a cursor
```

### Option B — Local install (works on any device, no CLI needed)

Clone this repo and copy the skill(s) you want into your Claude configuration:

```bash
git clone https://github.com/rosellerenrqz/ai-skills.git
```

**Install globally** (available in all your projects):

```bash
cp -r ai-skills/skills/timesheet ~/.claude/skills/
```

**Install per-project** (only available in the current project):

```bash
cp -r ai-skills/skills/timesheet .claude/skills/
```

> Claude Code looks for skills in `~/.claude/skills/` (global) and `.claude/skills/` (project-local).
> Both locations work — project-local takes precedence if both exist.

---

## Usage

Once installed, use the skill in Claude Code:

```
/timesheet 05-04-2026                                  — log a session from conversation history
/timesheet 05-04-2026 also deployed to staging         — log + merge extra context
/timesheet view 05-04-2026                             — view all sessions for a date
/timesheet summary 05-04-2026                          — generate end-of-day summary
/timesheet auto 05-04-2026                             — log + summarize in one step

/git-message                                           — suggest a commit message from staged changes
/git-message -branch                                   — suggest a commit message + branch name
```

---

## Skill Layout

```
ai-skills/
└── skills/
    ├── timesheet/
    │   ├── SKILL.md      — skill definition and instructions (read by the agent)
    │   └── README.md     — human-facing docs: commands, examples, output format
    └── git-message/
        ├── SKILL.md
        └── README.md
```

Each skill is a folder with both files. Adding a new skill = adding a new folder with both.

---

## Adding a New Skill

1. Create a new folder: `skills/your-skill-name/`
2. Add a `SKILL.md` with this frontmatter:

```markdown
---
name: your-skill-name
description: What it does and when to trigger it.
---

# Your Skill Name

...instructions...
```

3. Add a `README.md` covering: what the skill does, install commands, all `/commands` with examples, and what output files it creates
4. Add it to `.claude-plugin/marketplace.json` (bump the top-level `version` using semver)
5. Add a row to the Available Skills table above (with a link to the skill README)
6. Add an entry to `CHANGELOG.md`
7. Push to GitHub — it's immediately installable via both methods above

---

## Versioning

This repo uses [semantic versioning](https://semver.org/):

- **Patch** (1.0.x) — fixes or copy improvements to existing skills
- **Minor** (1.x.0) — new skill added
- **Major** (x.0.0) — breaking change to skill interface or format

The version in `.claude-plugin/marketplace.json` is the authoritative source. Each skill also carries its own `version` so consumers can track per-skill updates independently.

See [CHANGELOG.md](CHANGELOG.md) for the full history.
