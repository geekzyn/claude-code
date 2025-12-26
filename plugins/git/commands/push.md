---
description: Stage, commit (using /commit logic), and push to remote
allowed-tools:
  - Bash(git add:*)
  - Bash(git status:*)
  - Bash(git diff:*)
  - Bash(git commit:*)
  - Bash(git push:*)
  - Bash(git log:*)
  - Bash(git branch:*)
  - Bash(git rev-parse:*)
  - AskUserQuestion
argument-hint: "[optional commit message]"
model: haiku
---

# Quick Push Workflow

Orchestrates: stage all → commit (follows `/commit` rules) → push to remote.

## Current Git State

```bash
!git branch --show-current
```

```bash
!git rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null || echo "No upstream branch set"
```

```bash
!git status --short
```

## Unpushed Commits

```bash
!git log @{u}..HEAD --oneline 2>/dev/null || echo "No upstream to compare"
```

## User Input (optional commit message)

$ARGUMENTS

## Workflow

1. **Pre-flight checks**:
   - If on `main`, use `AskUserQuestion` to confirm:
     - Header: "Push to main"
     - Question: "You're on the main branch. What would you like to do?"
     - Options:
       - "Create branch" (description: "Create a feature branch first (recommended)")
       - "Push to main" (description: "Push directly to main (not recommended)")
       - "Cancel" (description: "Don't push")
     - multiSelect: false
     - If user selects "Create branch", invoke `/branch` command
     - If user selects "Cancel", abort
   - If no changes and no unpushed commits: "Nothing to push"
   - If only unpushed commits exist (no new changes): skip to push step

2. **Stage all changes**: `git add -A`

3. **Commit**: Follow the same rules as `/commit` command
   - If message provided in $ARGUMENTS, use it
   - If no message, analyze diff and generate 2-3 conventional commit message options
   - Use `AskUserQuestion` to present options (same as `/commit`):
     - Header: "Commit msg"
     - Question: "Which commit message would you like to use?"
     - Options: 2-3 generated messages (mark best as "(Recommended)")
     - User can select "Other" to provide custom message
     - multiSelect: false
   - Handle pre-commit hooks (re-stage if hooks modify files)

4. **Push**:
   - If upstream exists: `git push`
   - If no upstream: `git push -u origin <branch-name>`

5. **Summary**: Show commit hash, files changed, suggest `/pr` for next step

## Error Handling

| Error | Action |
|-------|--------|
| On main branch | Block, suggest `/branch` |
| Pre-commit hook fails | Show error, don't push |
| Push rejected (non-fast-forward) | Suggest `/rebase` |
| No upstream | Auto-set with `-u` flag |
