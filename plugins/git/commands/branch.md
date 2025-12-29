---
description: Create a feature branch following Conventional Branch specification
allowed-tools:
  - Bash(git checkout:*)
  - Bash(git branch:*)
  - Bash(git status:*)
  - Bash(git fetch:*)
  - Bash(git pull:*)
  - AskUserQuestion
argument-hint: "[type/description], 'delete' to remove current branch"
model: haiku
---

# Conventional Branch

Create branches following the [Conventional Branch](https://conventional-branch.github.io) specification.

## Context

- Current git status: !`git status`
- Current branch: !`git branch --show-current`

## Task

Input: $ARGUMENTS

Parse the **first word** to determine mode:

| First Word | Mode |
|------------|------|
| `delete` | DELETE MODE |
| anything else | CREATE MODE |

### DELETE MODE

**Steps**:
1. If current branch IS main/master:
   - Output error: "Cannot delete the main/master branch"
   - Stop execution
2. If current branch IS NOT main/master:
   - Store the current branch name for output message
   - Fetch remote branches and prune deleted ones: `git fetch --prune`
   - Checkout to main: `git checkout main`
   - Pull latest changes: `git pull`
   - Delete the branch: `git branch -D <previous-branch-name>`
   - Output: "Deleted branch [branch-name], checked out to main, and pulled latest changes"
   - Stop execution (do not proceed with branch creation)


### CREATE MODE

**Valid Types:**
| Type | Purpose |
|------|---------|
| `feature/` or `feat/` | New features |
| `bugfix/` or `fix/` | Bug fixes |
| `hotfix` | Urgent production fixes |
| `release` | Release preparation |
| `chore` | Non-code tasks (docs, deps) |

**Naming Rules:**
1. Use only lowercase letters (a-z), numbers (0-9), and hyphens (-)
2. No consecutive hyphens (`feature/new--login` is invalid)
3. No leading or trailing hyphens (`feature/-login` is invalid)
4. Format: `<type>/<description>`

**Steps**:
1. **Validate the branch name**:
   - Check type is valid and follows naming rules. 
   - Convert spaces to hyphens, remove special characters

2. **Determine source branch**:
   - If current branch IS the main/master branch:
     - Automatically use main/master as the source branch (skip asking the user)
   - If current branch IS NOT the main/master branch:
     - Ask user to select source branch using AskUserQuestion:
       - Present two options:
         - "Current branch" (with description showing the actual current branch name)
         - "main/master" (to create from main/master branch)
       - Use header: "Source branch"
       - Question: "Which branch should be used as the source?"

3. **Check for conflicts**:
   - Ensure branch does not already exist locally or remotely
   - If dirty working tree, warn user before proceeding

4. **Create and checkout the branch**:
   - If user selected "main/master" and currently not on main/master:
     ```bash
     git checkout -b <branch-name> main || git checkout -b <branch-name> master
     ```
   - If user selected "Current branch":
     ```bash
     git checkout -b <branch-name>
     ```

5. **Output**: Keep response minimal.
