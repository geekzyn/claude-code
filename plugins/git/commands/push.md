---
description: Push commits to remote repository
allowed-tools:
  - Bash(git push:*)
  - Bash(git status:*)
  - Bash(git log:*)
  - Bash(git branch:*)
  - Bash(git rev-parse:*)
  - AskUserQuestion
model: haiku
---

# Push

Push local commits to remote repository.

## Context

- Current git status: !`git status`
- Current branch: !`git branch --show-current`
- Upstream branch: !`git rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null || echo "No upstream branch set"`
- Unpushed commits: !`git log @{u}..HEAD --oneline 2>/dev/null || echo "No upstream to compare"`

## Task

**Steps:**
1. **Check if on main/master branch**:
   - If on `main` or `master`, use `AskUserQuestion` to confirm:
     - Header: "Push to main"
     - Question: "You're on the main branch. Are you sure you want to push?"
     - Options:
       - "Yes" (description: "Push directly to main")
       - "Cancel" (description: "Don't push")
     - multiSelect: false
     - If user selects "Cancel", abort execution

2. **Check for unpushed commits**:
   - If no unpushed commits: "Nothing to push"

3. **Push to remote**:
   - If upstream exists: `git push`
   - If no upstream: `git push -u origin <branch-name>`

4. **Output**: Show pushed commits summary
