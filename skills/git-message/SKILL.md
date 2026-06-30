---
name: git-message
description: >-
  Suggest a conventional-commit message (and optionally a branch name) from staged changes — read-only, never runs git. Triggered by /git-message or whenever the user asks for help writing a commit message, naming a commit, wording a commit, or naming a branch for their changes — even if they don't use the word git-message. Examples — "what's a good commit message for this", "write my commit message", "name a branch for these changes".
---

# git-message — Commit Message & Branch Name Suggester

Reads your staged changes and suggests a conventional-commit message. With `-branch`, it also suggests a branch name. **This skill is read-only** — it inspects the repository and prints suggestions for you to copy. It never runs `git commit`, `git checkout`, `git add`, or any state-changing command.

## Invariants

- **Read-only, always.** The only git commands this skill runs are read-only inspection commands: `git status`, `git diff --cached`, `git diff`, `git branch --show-current`, `git symbolic-ref`, `git log`. Never run `git commit`, `git checkout`, `git switch`, `git add`, `git reset`, `git stash`, or anything that mutates the working tree, index, or refs.
- **Suggest, never apply.** Output is text the user copies. Do not offer to commit or create the branch for them, and do not present a command-runner step that would do so.
- **No trailers.** Never append `Co-Authored-By:`, `🤖 Generated with Claude Code`, `Generated with Claude Code`, or any other trailer to the suggested message. The message body ends at the last content line.
- **Conventional Commits format.** The subject line is always `type(scope): subject` (scope optional). Body and footers only when the diff warrants them.
- **Derive from the actual diff.** Base every suggestion on what the diff actually shows — changed files, added/removed code, new dependencies. Do not invent changes that aren't in the diff or pad the message with generic phrasing.
- **One subject, imperative, ≤ 72 chars.** Subject line is a single imperative phrase ("add", not "added"/"adds"), no trailing period, 72 characters or fewer.

## Invocation

```
/git-message
/git-message -branch
```

## Argument Handling

- The command is always prefixed with `/git-message`.
- No flag → suggest a commit message only.
- `-branch` → suggest a commit message **and** a branch name. Order of flag and any other tokens does not matter; the presence of `-branch` anywhere in the args enables branch suggestion.

## What gets read

1. `git status --short` — to see staged vs unstaged state.
2. `git diff --cached` — the staged changes. **This is the primary source.**
3. `git branch --show-current` and the default branch (`git symbolic-ref refs/remotes/origin/HEAD` falling back to checking for `main`/`master`) — only when `-branch` is used.

## Workflow

### `/git-message` — Suggest a commit message

1. Run `git status --short` to determine what is staged.
2. **If nothing is staged:**
   - Run `git diff --stat` to check whether there are unstaged changes.
   - If unstaged changes exist, tell the user nothing is staged and offer to base the suggestion on **unstaged** changes instead (read via `git diff`). Wait for confirmation before reading unstaged changes — do not silently fall back.
   - If there are no changes at all, report `No changes to describe — working tree is clean.` and stop.
3. Read the staged diff with `git diff --cached`.
   - **Large diffs:** if the diff is very large, do not dump the whole thing into reasoning. Use `git diff --cached --stat` for the file-level shape and read the key hunks (`git diff --cached -- {path}`) for the files that carry the intent. Summarize from those.
4. Determine the **type** from the dominant change:
   - `feat` — a new capability or user-facing feature
   - `fix` — a bug fix
   - `refactor` — code change that neither fixes a bug nor adds a feature
   - `docs` — documentation only
   - `test` — adding or correcting tests only
   - `chore` — tooling, deps, config, housekeeping
   - `build`, `ci`, `perf`, `style` — when clearly applicable
   - When changes span multiple types, pick the type of the **primary intent** of the diff; mention the rest in the body.
5. Determine the **scope** (optional): the package, module, directory, or feature area most affected (e.g. `auth`, `timesheet`, `api`). Omit the scope if the change is broad or no single area dominates.
6. Write the **subject**: `type(scope): subject` — imperative mood, lowercase after the colon, no trailing period, ≤ 72 chars.
7. Write a **body** only when the change needs explanation the subject can't carry — what changed and *why*, wrapped at ~72 chars, as bullet points or short paragraphs. Skip the body for small, self-evident changes.
8. **Do not add any trailer.**
9. Output the message in a single fenced code block so the user can copy it cleanly, followed by a one-line note on what it was derived from (e.g. staged vs unstaged, number of files).

### `/git-message -branch` — Suggest a commit message and a branch name

1. Do everything in `/git-message` first.
2. Determine the current branch (`git branch --show-current`) and the default branch.
3. **Suggest a branch name** derived from the commit type and subject:
   - Format: `{type}/{short-kebab-summary}` (e.g. `feat/git-message-skill`, `fix/blog-date-display`).
   - Use the commit `type` as the prefix. Use a 2–5 word kebab-case summary of the subject — lowercase, hyphens only, no scope punctuation.
   - Keep it under ~40 characters.
4. **If the current branch is the default branch** (`main`/`master`), add a note: the user is committing on the default branch, and this branch name is what they'd use to move the work onto its own branch (e.g. `git switch -c {name}`) — shown as a suggestion only, not run.
5. **If already on a non-default branch**, still suggest the name but note they may already be on a suitable branch.
6. Output the branch name on its own line below the commit message block.

## Output Format

### Commit message only

```
feat(timesheet): add commit-message cleanup command

Add a -clean subcommand that removes leaked commit-recommendation
text from logged timesheet files and renumbers the remaining notes.
```

_Derived from 3 staged files._

### With `-branch`

Same message block as above, then:

```
Suggested branch: feat/timesheet-clean-command
```

> You're on `main` — to move this work onto its own branch: `git switch -c feat/timesheet-clean-command` (suggestion only; not run).

## Handoff

After suggesting:

```
Commit message suggested from {staged|unstaged} changes ({N} files).
Copy the message above into your commit.
  /git-message            — message only
  /git-message -branch    — message + branch name
```
