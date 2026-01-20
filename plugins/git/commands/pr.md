---
description: Create a pull request in Azure DevOps
allowed-tools:
  - Bash(git log:*)
  - Bash(git diff:*)
  - Bash(git branch:*)
  - Bash(git status:*)
  - Bash(git remote get-url:*)
  - Bash(git rev-parse:*)
  - Bash(git push:*)
  - Bash(az repos pr:*)
  - Bash(az devops:*)
  - AskUserQuestion
argument-hint: "[--draft] or [custom title]"
model: haiku
---

# Azure DevOps Pull Request

Create pull requests following Azure DevOps conventions.

## Context

- Current git status: !`git status -sb`
- Current branch: !`git branch --show-current`
- Remote URL: !`git remote get-url origin`
- Commits vs main: !`git log origin/main..HEAD --oneline 2>/dev/null || git log main..HEAD --oneline`
- Files changed: !`git diff origin/main..HEAD --stat 2>/dev/null || git diff main..HEAD --stat`
- Full diff: !`git diff origin/main..HEAD 2>/dev/null || git diff main..HEAD`

## Task

Input: $ARGUMENTS

Parse the input to determine options:

| Input | Effect |
|-------|--------|
| Empty | Auto-generate title |
| `--draft` | Create as draft PR |
| Custom text | Use as PR title |

### PRE-FLIGHT CHECKS

**Steps:**
1. **Check for uncommitted changes**:
   - Warn user if there are uncommitted changes

2. **Check for unpushed commits**:
   - If unpushed commits exist, use `AskUserQuestion`:
     - Header: "Unpushed"
     - Question: "You have unpushed commits. What would you like to do?"
     - Options:
       - "Push now" (description: "Push commits to remote before creating PR")
       - "Continue anyway" (description: "Create PR without local commits - not recommended")
     - multiSelect: false
   - If user selects "Push now", execute `git push -u origin <branch>` first
   - If push fails, abort PR creation

3. **Verify Azure CLI**:
   - Check if `az` is available and logged in
   - If not, prompt: `az login`

### GENERATE PR CONTENT

**Valid Types (based on commits):**
| Type | Purpose |
|------|---------|
| `feat` | A new feature |
| `fix` | A bug fix |
| `docs` | Documentation only changes |
| `refactor` | Code change that neither fixes nor adds feature |
| `test` | Adding or correcting tests |
| `build` | Changes to build system or dependencies |
| `ci` | Changes to CI configuration |
| `chore` | Other changes that don't modify src or test |
| `revert` | Reverts a previous commit |

**Title Generation:**
1. Review ALL commits on the branch (not just the latest)
2. Determine primary type based on commits
3. Keep title under 80 characters, imperative mood

**If no custom title provided:**
- Generate 2-3 title options based on commits
- Use `AskUserQuestion`:
  - Header: "PR title"
  - Question: "Which title would you like to use?"
  - Options: 2-3 generated titles (mark best as "(Recommended)")
  - User can select "Other" to provide custom title 
  - multiSelect: false

**Description:** Use project's PR template format from `docs/pull_request_template.md` if available.

### CREATE PR

**Steps:**
1. **Execute az command**:
   - Write description to temp file:
     - DO NOT add Claude co-authorship footer to commits
     ```bash
     cat > /tmp/pr_description.txt << 'EOF'
     <description content>
     EOF
     ```
   - Create PR:
     ```bash
     az repos pr create \
       --title "<title>" \
       --description "$(cat /tmp/pr_description.txt)" \
       --source-branch "<current-branch>" \
       --target-branch "<main-branch>" \
       --org "" \
       --project "" \
       --repository "" \
       --auto-complete false \
       [--draft true if requested]
     ```
   - Clean up: `rm -f /tmp/pr_description.txt`

2. **Output**: Show PR number, URL, and status. Keep response minimal.
