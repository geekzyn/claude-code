---
description: Rebase current branch onto latest main
allowed-tools:
  - Bash(git fetch:*)
  - Bash(git rebase:*)
  - Bash(git status:*)
  - Bash(git branch:*)
  - Bash(git stash:*)
  - Bash(git log:*)
  - Bash(git diff:*)
  - Bash(git rev-list:*)
  - AskUserQuestion
argument-hint: "[--abort] to abort ongoing rebase"
model: haiku
---

# Sync Branch with Main

Safely rebase your current branch onto the latest main.

## Current Git State

```bash
!git branch --show-current
```

```bash
!git status
```

## Rebase Status

```bash
!test -d .git/rebase-merge && echo "REBASE IN PROGRESS" || echo "No rebase in progress"
```

## Branch Position

```bash
!echo "Commits ahead of main: $(git rev-list --count origin/main..HEAD 2>/dev/null || echo 'unknown')"
```

```bash
!echo "Commits behind main: $(git rev-list --count HEAD..origin/main 2>/dev/null || echo 'unknown')"
```

## Recent Main Branch Commits

```bash
!git log origin/main --oneline -5 2>/dev/null || echo "Run 'git fetch' first"
```

## User Input

$ARGUMENTS

## Workflow

### Handle Special Cases First

**If `--abort` argument provided:**
1. Check if rebase is in progress
2. Run `git rebase --abort`
3. Confirm rebase was aborted
4. Show current status

**If rebase already in progress:**
1. Show current rebase status
2. List conflicting files
3. Use `AskUserQuestion` to ask what to do:
   - Header: "Rebase action"
   - Question: "There is a rebase in progress. What would you like to do?"
   - Options:
     - "Continue" (description: "Continue rebase (only if conflicts are resolved)")
     - "Abort" (description: "Abort the rebase and return to previous state")
     - "Show conflicts" (description: "Show which files have conflicts")
   - multiSelect: false

### Normal Sync Workflow

#### Step 1: Pre-flight Checks

1. **Check for uncommitted changes**:
   - If dirty working tree, use `AskUserQuestion` to ask:
     - Header: "Uncommitted"
     - Question: "You have uncommitted changes. What would you like to do?"
     - Options:
       - "Stash" (description: "Stash changes and continue (recommended)")
       - "Commit first" (description: "Commit changes before rebasing")
       - "Abort" (description: "Cancel the rebase")
     - multiSelect: false
   - If user selects "Stash", run `git stash push -m "Auto-stash before sync"`
   - If user selects "Commit first", suggest using `/commit` command
   - Remember to pop stash after rebase if stashed

2. **Verify not on main**:
   - If on main, just do `git pull --rebase` instead

#### Step 2: Fetch Latest

1. Run `git fetch origin`
2. Show what's new on origin/main:
   - Number of new commits
   - Summary of changes

#### Step 3: Rebase onto Main

Run `git rebase origin/main`

**If successful:**
- Show success message
- Show new commit graph
- Pop stash if we stashed earlier

**If conflicts occur:**
- STOP and inform user
- List conflicting files
- Use `AskUserQuestion` to ask:
  - Header: "Conflicts"
  - Question: "Conflicts detected during rebase. What would you like to do?"
  - Options:
    - "Show conflicts" (description: "Show which files have conflicts")
    - "Help resolve" (description: "Get guidance on resolving conflicts")
    - "Abort" (description: "Abort the rebase")
  - multiSelect: false
- If user selects "Show conflicts", list the conflicting files
- If user selects "Help resolve", provide detailed guidance:

```
Conflicts detected in:
- <file1>
- <file2>

To resolve:
1. Edit the conflicting files to resolve conflicts
2. Stage resolved files: git add <file>
3. Continue rebase: git rebase --continue

Or abort: /rebase --abort
```

#### Step 4: Post-Sync Summary

```
Sync complete!

Branch: <branch-name>
Status: Rebased onto main (was X commits behind)
Commits on branch: Y

Your branch is now up-to-date with main.
Next: Continue developing or /push your changes.
```
