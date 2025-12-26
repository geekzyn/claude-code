---
description: Revert one or more commits safely
allowed-tools:
  - Bash(git log:*)
  - Bash(git revert:*)
  - Bash(git status:*)
  - Bash(git branch:*)
  - Bash(git show:*)
  - Bash(git diff:*)
  - AskUserQuestion
argument-hint: "[commit-hash] or [HEAD~n] or empty to select interactively"
model: haiku
---

# Revert Commits

Safely revert one or more commits by creating new commits that undo their changes.

## Current Git State

```bash
!git branch --show-current
```

```bash
!git status
```

## Recent Commits

```bash
!git log --oneline -10
```

## User Input

$ARGUMENTS

## Workflow

### Step 1: Pre-flight Checks

1. **Check for uncommitted changes**:
   - If dirty working tree, use `AskUserQuestion` to ask:
     - Header: "Uncommitted"
     - Question: "You have uncommitted changes. What would you like to do?"
     - Options:
       - "Stash" (description: "Stash changes and continue (recommended)")
       - "Commit first" (description: "Commit changes before reverting")
       - "Abort" (description: "Cancel the revert")
     - multiSelect: false
   - If user selects "Stash", run `git stash push -m "Auto-stash before revert"`
   - If user selects "Commit first", suggest using `/commit` command
   - Remember to pop stash after revert if stashed

2. **Check for revert in progress**:
   ```bash
   test -f .git/REVERT_HEAD && echo "REVERT IN PROGRESS" || echo "No revert in progress"
   ```
   - If revert in progress, use `AskUserQuestion` to ask:
     - Header: "Revert action"
     - Question: "There is a revert in progress. What would you like to do?"
     - Options:
       - "Continue" (description: "Continue revert (only if conflicts are resolved)")
       - "Abort" (description: "Abort the revert and return to previous state")
       - "Show conflicts" (description: "Show which files have conflicts")
     - multiSelect: false

### Step 2: Identify Commits to Revert

**If commit hash/reference provided as argument:**
1. Validate the commit exists: `git show <commit> --oneline --no-patch`
2. Show what will be reverted:
   ```bash
   git show <commit> --stat
   ```
3. Proceed to Step 3

**If no argument provided:**
1. Show recent commits for selection
2. Use `AskUserQuestion` to ask:
   - Header: "Select commit"
   - Question: "Which commit would you like to revert?"
   - Options (based on recent commits, max 4):
     - Option 1: Most recent commit with hash and message
     - Option 2: Second most recent
     - Option 3: Third most recent
     - Option 4: "Multiple commits" (description: "Revert a range of commits")
   - multiSelect: false
3. If "Multiple commits" selected, ask for the range

### Step 3: Confirm Revert

Show the commit(s) to be reverted with their changes:
```bash
git show <commit> --stat
```

Use `AskUserQuestion` to confirm:
- Header: "Confirm"
- Question: "This will create a new commit that undoes the changes. Proceed?"
- Options:
  - "Yes, revert" (description: "Create revert commit (recommended)")
  - "Show full diff" (description: "See complete changes before deciding")
  - "Cancel" (description: "Abort the revert")
- multiSelect: false

### Step 4: Execute Revert

**For single commit:**
```bash
git revert <commit> --no-edit
```

**For multiple commits (oldest to newest):**
```bash
git revert <oldest-commit>^..<newest-commit> --no-edit
```

**If successful:**
- Show the new revert commit(s)
- Pop stash if we stashed earlier
- Show summary

**If conflicts occur:**
- STOP and inform user
- List conflicting files
- Use `AskUserQuestion` to ask:
  - Header: "Conflicts"
  - Question: "Conflicts detected during revert. What would you like to do?"
  - Options:
    - "Show conflicts" (description: "Show which files have conflicts")
    - "Help resolve" (description: "Get guidance on resolving conflicts")
    - "Abort" (description: "Abort the revert")
  - multiSelect: false

### Step 5: Post-Revert Summary

```
Revert complete!

Branch: <branch-name>
Reverted: <commit-hash> - <commit-message>
New commit: <new-commit-hash>

The changes have been undone. Next steps:
- Review the revert with: git show HEAD
- Push the revert: /push
- Or undo the revert: git reset --hard HEAD~1
```

## Error Handling

| Scenario | Action |
|----------|--------|
| Dirty working tree | Stash first, pop after |
| Invalid commit hash | Show error, list recent commits |
| Conflicts during revert | Stop, show files, guide resolution |
| Merge commit | Require `-m` flag, explain parent selection |
| Already reverted | Warn user, suggest checking history |
| Stash pop conflicts | Inform user, suggest manual resolution |

## Handling Merge Commits

If the commit to revert is a merge commit:
1. Detect with: `git cat-file -p <commit> | grep -c '^parent'`
2. If more than 1 parent, inform user:
   ```
   This is a merge commit. Reverting merge commits requires specifying which parent to keep.

   Parent 1: Usually the branch that was merged INTO (e.g., main)
   Parent 2: Usually the branch that was merged FROM (e.g., feature branch)
   ```
3. Use `AskUserQuestion` to ask:
   - Header: "Merge parent"
   - Question: "Which parent should be kept when reverting this merge?"
   - Options:
     - "Parent 1" (description: "Keep the mainline, undo the merged changes")
     - "Parent 2" (description: "Keep the merged changes, undo mainline")
     - "Cancel" (description: "Abort the revert")
   - multiSelect: false
4. Execute with: `git revert <commit> -m <parent-number> --no-edit`

## Conflict Resolution Guidance

When conflicts occur, provide file-specific advice:

- **JSON files** (ADF pipelines): Be careful with JSON structure, validate after resolving
- **YAML files** (pipelines, config): Watch indentation carefully
- **SQL files**: Review logic changes carefully

Remind user that after resolving:
1. `git add <resolved-files>`
2. `git revert --continue`
3. Or `git revert --abort` to cancel

## Examples

```bash
# Revert the last commit
/revert HEAD

# Revert a specific commit by hash
/revert abc1234

# Revert the commit 3 commits ago
/revert HEAD~3

# Interactive selection (no argument)
/revert
```
