---
name: git-workflow
description: "Manages Git version control and GitHub integration for a solo developer project. Use this skill whenever any of the following are involved: committing changes, pushing to GitHub, creating or switching branches, writing commit messages, updating or creating a .gitignore file, staging files, checking git status or logs, undoing changes, handling merge conflicts, or any other git/version-control operation. Trigger this skill even when the user says things like 'save my progress', 'push this up', 'make a new branch for this feature', 'don't track X', or 'what did I change?' — these all imply git operations. This skill enforces user-approval before any commit or push, auto-manages .gitignore hygiene, and opens new branches for major changes."
---

# Git Workflow Skill

This skill governs all Git and GitHub operations for the project. Follow every
rule here precisely and consistently — version control hygiene matters.

---

## Core Principles

1. **Never commit or push without explicit user approval.** Always show a summary
   of what will be committed and ask for a go-ahead before running `git commit`
   or `git push`.
2. **Keep .gitignore clean and correct.** Before staging any files, check whether
   any of them (or files they'd introduce) should be ignored. Update .gitignore
   first if needed.
3. **Branch for major changes.** New features, significant refactors, or breaking
   changes go on a dedicated branch — never directly on `main`/`master`.
4. **Write meaningful commit messages.** Follow the Conventional Commits format
   described below.

---

## Workflow: Making a Commit

Follow these steps in order. Do not skip steps.

### Step 1 — Assess scope

Determine whether the change is **minor** or **major**:

| Type | Examples | Branch needed? |
|---|---|---|
| Minor / patch | Bug fix, typo, config tweak, dependency bump, docs update | No — commit on current branch |
| Major | New feature, significant refactor, new module/page, breaking change | Yes — open a new branch |

### Step 2 — Branch (if major)

If the change is major and you're on `main`/`master` (or another protected branch):

```bash
# Use a short, descriptive, kebab-case name
git checkout -b feature/<short-description>
# or: fix/<short-description>, refactor/<short-description>, chore/<short-description>
```

Tell the user which branch was created and why.

### Step 3 — Update .gitignore

Before staging anything, run through the **.gitignore checklist** below and
read `references/gitignore-patterns.md` for the full pattern reference.

**Checklist — always ignore:**
- `node_modules/`
- Build & dist outputs (`dist/`, `build/`, `out/`, `.next/`, etc.)
- Environment & secret files (`.env`, `.env.*`, `*.pem`, `*.key`, `secrets/`)
- IDE/editor settings (`.vscode/`, `.idea/`, `*.suo`, `*.swp`)
- OS-generated files (`.DS_Store`, `Thumbs.db`, `desktop.ini`)
- Logs (`*.log`, `logs/`, `npm-debug.log*`)
- Runtime/cache dirs (`.cache/`, `.parcel-cache/`, `__pycache__/`, `*.pyc`)
- Coverage & test artifacts (`coverage/`, `.nyc_output/`)
- Lock files that shouldn't be tracked (consult user — often `package-lock.json`
  or `yarn.lock` *should* be tracked; confirm intent)

**Rules for editing .gitignore:**
- Group patterns by category with a comment header (e.g., `# Dependencies`)
- Use a blank line between groups
- Never delete existing entries without asking the user
- Always append new patterns; keep the file sorted within each group
- After editing, run `git status` to confirm newly-ignored files aren't staged

Read `references/gitignore-patterns.md` for an extensive pattern list and
formatting template to use as the base when creating a new .gitignore.

### Step 4 — Stage files

```bash
git add <specific files or paths>
```

Prefer explicit paths over `git add .` unless the entire working tree should
be staged. Tell the user exactly which files are being staged.

### Step 5 — Show summary and request approval

Present a clear summary to the user:

```
📋 Commit Summary
─────────────────────────────────────────
Branch:   feature/user-auth
Files:    src/auth.js, src/routes/login.js, README.md
Message:  feat(auth): add JWT-based login flow

Modified:  src/auth.js        — new validateToken() function
Added:     src/routes/login.js — POST /login endpoint
Modified:  README.md          — updated setup instructions
─────────────────────────────────────────
Approve this commit? (yes / edit message / cancel)
```

**Do not run `git commit` until the user explicitly approves.** If they ask to
edit the message, update it and show the summary again before proceeding.

### Step 6 — Commit

```bash
git commit -m "<type>(<scope>): <subject>"
```

See **Commit Message Format** below.

### Step 7 — Push

After committing, ask: *"Push to GitHub now?"*

If approved:
```bash
git push origin <branch-name>
```

If it's a new branch:
```bash
git push -u origin <branch-name>
```

Confirm the push succeeded and show the remote URL if it's a new branch.

---

## Commit Message Format

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<optional scope>): <short imperative description>

<optional body — explain WHY, not WHAT>

<optional footer — e.g., "Closes #12", "BREAKING CHANGE: ...">
```

**Types:**
- `feat` — new feature
- `fix` — bug fix
- `docs` — documentation only
- `style` — formatting, no logic change
- `refactor` — code restructure, no feature/fix
- `test` — adding or fixing tests
- `chore` — build, config, dependency updates

**Rules:**
- Subject line: ≤72 characters, lowercase, no trailing period, imperative mood
  ("add login page" not "added login page")
- Body: wrap at 72 chars, use blank line to separate from subject
- Reference issues in footer when applicable

---

## Branch Naming

| Purpose | Pattern | Example |
|---|---|---|
| New feature | `feature/<name>` | `feature/dark-mode` |
| Bug fix | `fix/<name>` | `fix/broken-redirect` |
| Refactor | `refactor/<name>` | `refactor/db-layer` |
| Docs | `docs/<name>` | `docs/api-reference` |
| Chore/config | `chore/<name>` | `chore/update-deps` |

All names: lowercase, kebab-case, no spaces, concise (2–4 words max).

---

## Pull Requests (when merging a feature branch)

When a feature branch is ready to merge into `main`:

1. Ensure the branch is pushed to GitHub
2. Remind the user to open a Pull Request on GitHub (Claude Code cannot open
   PRs directly, but can help draft the PR title and description)
3. Suggest the PR title follows the same Conventional Commits format
4. After the PR is merged, offer to delete the local and remote branch:
   ```bash
   git branch -d feature/<name>
   git push origin --delete feature/<name>
   ```

---

## Common Operations (Quick Reference)

**Check what's changed:**
```bash
git status
git diff
```

**View recent history:**
```bash
git log --oneline -10
```

**Undo unstaged changes to a file:**
```bash
git checkout -- <file>
```

**Undo last commit (keep changes staged):**
```bash
git reset --soft HEAD~1
```

**Stash work in progress:**
```bash
git stash push -m "description of wip"
git stash pop   # restore later
```

**Merge main into feature branch (keep branch up to date):**
```bash
git checkout feature/<name>
git merge main
```

---

## Safety Rules

- **Never force-push to `main`** without explicit discussion and user consent.
- **Never commit secrets.** If a `.env` or credential file is accidentally staged,
  abort immediately, add it to .gitignore, and advise the user to rotate the
  exposed credential.
- **Never amend a commit that has already been pushed** without warning the user
  about the consequences (requires force-push, rewrites history).
- If a merge conflict occurs, present the conflicting sections clearly and help
  the user resolve them before proceeding.

---

## Stale Branch Cleanup

Stale branches accumulate over time and clutter the repo. Claude should
proactively offer cleanup at natural checkpoints (e.g., after a PR merge,
or when the user asks about branches).

### When to offer cleanup

- After merging a feature branch into `main`
- When the user asks "what branches do I have?" or similar
- When `git branch` output shows 5 or more branches

### Step 1 — Identify candidates

```bash
# List all local branches with their last commit date
git branch -v

# List branches already merged into main (safe to delete)
git branch --merged main

# List remote branches
git branch -r

# Check which remote branches have been deleted on GitHub
git remote prune origin --dry-run
```

### Step 2 — Present a cleanup report

Show the user a clear summary before touching anything:

```
🌿 Branch Cleanup Report
──────────────────────────────────────────────
Current branch: main

Merged into main (safe to delete):
  ✓ feature/dark-mode        last commit: 3 weeks ago
  ✓ fix/broken-redirect      last commit: 5 days ago

Unmerged (caution — review before deleting):
  ⚠ feature/old-experiment   last commit: 6 weeks ago

Remote tracking refs that no longer exist on GitHub:
  origin/feature/dark-mode
  origin/fix/broken-redirect
──────────────────────────────────────────────
Which would you like to delete? (all merged / select / none)
```

**Do not delete any branch without explicit user approval.**

### Step 3 — Delete approved branches

For each approved branch:

```bash
# Delete local branch (use -d for merged, -D only if user explicitly confirms for unmerged)
git branch -d feature/<n>

# Delete remote tracking ref
git push origin --delete feature/<n>
```

Prune stale remote refs in one pass:
```bash
git remote prune origin
```

### Step 4 — Confirm

After deletion, run `git branch -v` again and show the user the clean branch
list.

### Safety rules for cleanup

- **Never delete `main`, `master`, `develop`, or any branch the user flags as
  protected.**
- For **unmerged** branches, always use `-d` first (Git will refuse if unmerged).
  Only use `-D` (force) if the user explicitly says they don't need the work.
- If unsure whether a branch has open work, suggest the user check GitHub for
  an associated open PR before deleting.

---

## Reference Files

- `references/gitignore-patterns.md` — Full .gitignore pattern library and
  formatted template. Read this whenever creating or significantly updating
  a .gitignore file.
