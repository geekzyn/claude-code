---
description: Create a feature branch following Conventional Branch specification
allowed-tools:
  - Bash(git checkout:*)
  - Bash(git branch:*)
  - Bash(git status:*)
  - Bash(git fetch:*)
  - Bash(git pull:*)
  - AskUserQuestion
argument-hint: "[type/description], 'delete' to remove current branch, or 'delete all' to remove all branches"
model: haiku
---

# Conventional Branch Creation

Create branches following the [Conventional Branch](https://conventional-branch.github.io) specification.

## Current Git State

- Current git status: !`git status`
- Current git diff (staged and unstaged changes): !`git diff HEAD`
- Current branch: !`git branch --show-current`

## Conventional Branch Rules

**Valid Types:**
| Type | Purpose |
|------|---------|
| `feature` | New features |
| `bugfix` | Bug fixes |
| `hotfix` | Urgent production fixes |
| `release` | Release preparation |
| `chore` | Non-code tasks (docs, deps) |

**Naming Rules:**
1. Use only lowercase letters (a-z), numbers (0-9), and hyphens (-)
2. No consecutive hyphens (`feature/new--login` is invalid)
3. No leading or trailing hyphens (`feature/-login` is invalid)
4. Format: `<type>/<description>`

## User Input

$ARGUMENTS

## Instructions

### DELETE MODE

**If arguments equal "delete all"**:
1. Fetch remote branches and prune deleted ones: `git fetch --prune`
2. Checkout to main if not already there: `git checkout main`
3. Pull latest changes: `git pull`
4. List all local branches: `git branch --format='%(refname:short)'`
5. Filter out 'main' from the list
6. For each branch in the filtered list:
   - Delete it: `git branch -D <branch-name>`
7. Output: "Deleted X branches, checked out to main, and pulled latest changes. Deleted branches: [list of branch names]"
8. Stop execution (do not proceed with branch creation)

**If arguments equal "delete"**:
1. Check current branch name
2. If current branch IS main:
   - Output error: "Cannot delete the main branch"
   - Stop execution
3. If current branch IS NOT main:
   - Store the current branch name for output message
   - Fetch remote branches and prune deleted ones: `git fetch --prune`
   - Checkout to main: `git checkout main`
   - Pull latest changes: `git pull`
   - Delete the branch: `git branch -D <previous-branch-name>`
   - Output: "Deleted branch [branch-name], checked out to main, and pulled latest changes"
   - Stop execution (do not proceed with branch creation)

### CREATE MODE

1. **If no arguments provided**:
   - Use AskUserQuestion to ask the user for the branch type:
     - Header: "Branch type"
     - Question: "What type of branch do you want to create?"
     - Options:
       - "feature" (description: "New features or enhancements")
       - "bugfix" (description: "Bug fixes")
       - "hotfix" (description: "Urgent production fixes")
       - "chore" (description: "Non-code tasks like docs, deps, or configs")
     - multiSelect: false
   - Then ask the user directly (via text response) for the branch description: "What would you like to name this [type] branch? (use spaces or hyphens)"
   - Wait for the user to provide the description in their next message
   - After receiving the description, combine into format: `<type>/<description>` and proceed with validation.

2. **If arguments provided**: Parse the input. Accept formats like:
   - `feature/add-user-auth` (complete branch name)
   - `feature add-user-auth` (type and description separated by space)
   - `feature user auth` (type and keywords to convert to hyphenated)
   - If user types `feat`, convert to `feature`
   - If user types `fix`, convert to `bugfix`

3. **Validate the branch name**:
   - Check type is valid (feature, bugfix, hotfix, release, chore)
   - Ensure description is lowercase with hyphens
   - No consecutive, leading, or trailing hyphens
   - Convert spaces to hyphens, remove special characters

4. **Determine source branch**:
   - If current branch IS the main branch:
     - Automatically use main as the source branch (skip asking the user)
   - If current branch IS NOT the main branch:
     - Ask user to select source branch using AskUserQuestion:
       - Present two options:
         - "Current branch" (with description showing the actual current branch name)
         - "main" (to create from main branch)
       - Use header: "Source branch"
       - Question: "Which branch should be used as the source?"

5. **Check for conflicts**:
   - Ensure branch does not already exist locally or remotely
   - If dirty working tree, warn user before proceeding

6. **Create and checkout the branch**:
   - If user selected "main" and currently not on main:
     ```bash
     git checkout -b <branch-name> main
     ```
   - If user selected "Current branch":
     ```bash
     git checkout -b <branch-name>
     ```

7. **Output**: Confirm branch was created from which source branch and suggest next steps (e.g., "Use /commit after making changes")
