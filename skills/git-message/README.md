# git-message

Suggest a conventional-commit message — and optionally a branch name — from your staged changes. **Read-only:** it inspects the repo and prints suggestions for you to copy. It never runs `git commit`, `git checkout`, `git add`, or any state-changing command.

---

## Install

**Via marketplace CLI:**
```bash
npx skills add rosellerenrqz/ai-skills --skill git-message
```

**Globally (available in all projects):**
```bash
npx skills add rosellerenrqz/ai-skills --skill git-message -g
```

**Manual (no CLI needed):**
```bash
git clone https://github.com/rosellerenrqz/ai-skills.git
cp -r ai-skills/skills/git-message ~/.claude/skills/   # global
# or
cp -r ai-skills/skills/git-message .claude/skills/     # project-local
```

---

## Commands

```
/git-message            Suggest a conventional-commit message from staged changes
/git-message -branch    Suggest a commit message + a branch name from staged changes
```

---

## Usage examples

### Suggest a commit message

```
/git-message
```

Reads your staged changes (`git diff --cached`), picks the right conventional-commit type and scope, and prints a ready-to-copy message:

```
feat(timesheet): add commit-message cleanup command

Add a -clean subcommand that removes leaked commit-recommendation
text from logged timesheet files and renumbers the remaining notes.
```

If **nothing is staged**, it tells you and offers to base the suggestion on your unstaged changes instead — it won't silently guess. If the working tree is clean, it says so and stops.

### Suggest a commit message and a branch name

```
/git-message -branch
```

Does everything above, then suggests a branch name derived from the commit:

```
Suggested branch: feat/timesheet-clean-command
```

If you're currently on `main` (or `master`), it adds a note showing the `git switch -c` command you'd run to move the work onto its own branch — as a **suggestion only**, never executed.

---

## Output format

- **Subject:** `type(scope): subject` — imperative mood, no trailing period, ≤ 72 characters. Scope is optional and omitted when the change is broad.
- **Body:** included only when the change needs the "why" explained; wrapped at ~72 characters. Small, self-evident changes get a subject only.
- **No trailers:** the message never includes `Co-Authored-By` or `Generated with Claude Code` lines.

### Commit types used

| Type | When |
| ---- | ---- |
| `feat` | a new capability or user-facing feature |
| `fix` | a bug fix |
| `refactor` | code change that neither fixes a bug nor adds a feature |
| `docs` | documentation only |
| `test` | adding or correcting tests only |
| `chore` | tooling, deps, config, housekeeping |
| `build` / `ci` / `perf` / `style` | when clearly applicable |

When a diff spans multiple types, the message uses the type of the **primary intent** and mentions the rest in the body.

---

## What gets created

Nothing. This skill is **read-only** — it writes no files and runs no state-changing git commands. The only git commands it runs are inspection commands (`git status`, `git diff`, `git diff --cached`, `git branch --show-current`, `git log`). All output is text for you to copy.

---

## Version

`1.3.0` — see [CHANGELOG](../../CHANGELOG.md)
