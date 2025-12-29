---
description: Create a conventional commit with smart message generation
allowed-tools:
  - Bash(git add:*)
  - Bash(git status:*)
  - Bash(git diff:*)
  - Bash(git commit:*)
  - Bash(git log:*)
  - AskUserQuestion
argument-hint: "[optional message] or empty for auto-generation"
model: haiku
---

# Conventional Commit

Create commits following the [Conventional Commits](https://www.conventionalcommits.org) specification.

## Context

- Current git status: !`git status`
- Current git diff: !`git diff HEAD`
- Current branch: !`git branch --show-current`
- Recent commits: !`git log --oneline -10`

## Task

Input: $ARGUMENTS

Parse the input to determine mode:

| Input | Mode |
|-------|------|
| Empty/none | AUTO-GENERATE MODE |
| Message text | CUSTOM MESSAGE MODE |

> **Note:** Always use `git commit -am` to stage and commit together.

**Commit Format**
```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

**Valid Types:**
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

**Scope**: Based on project structure (e.g., `api`, `ui`, `db`, `auth`)

**Naming Rules:**
1. Type and description are required
2. Description should be lowercase, imperative mood ("add feature" not "added feature")
3. No period at the end of description
4. Breaking changes use exclamation mark after type or `BREAKING CHANGE` in footer

### AUTO-GENERATE MESSAGE MODE

**Steps:**
1. **Check for changes**:
   - If no staged or unstaged changes, inform user "Nothing to commit"
   - If there are changes, stage all with `git add -A`

2. **Analyze and generate options**:
   - Analyze the diff to understand what changed
   - Determine appropriate type based on changes
   - Generate 2-3 commit message options with different types/scopes where applicable

3. **Present options to user**:
   - Use `AskUserQuestion` tool:
     - Header: "Commit msg"
     - Question: "Which commit message would you like to use?"
     - Option 1: Primary suggested message 
     - Options 2-3: Alternative messages
     - User can select "Other" to provide custom message
   - multiSelect: false

4. **Create the commit**:
   - NEVER use `git commit -m` - always use `git commit -am`
   ```bash
   git commit -am "<selected-message>"
   ```

### CUSTOM MESSAGE MODE

**Steps:**
1. **Check for changes**:
   - If no staged or unstaged changes, inform user "Nothing to commit"
   - If there are changes, stage all with `git add -A`

2. **Validate message format**:
   - Check it follows conventional commit format
   - If invalid, suggest corrections and ask user to confirm

3. **Create the commit**:
   - NEVER use `git commit -m` - always use `git commit -am`
   ```bash
   git commit -am "<message>"
   ```
**Handle pre-commit hook**:
- If pre-commit hook fails:
 * CRITICAL: STOP IMMEDIATELY - Do not proceed with any other actions
 * DO NOT attempt to fix, analyze, or investigate any issues reported by the hook
 * DO NOT edit any files or make any changes whatsoever
 * DO NOT offer suggestions or solutions for the failures
 * DO NOT use any tools (Read, Edit, Write, Grep, etc.)
 * ONLY show the exact git commit command that failed:
   ```bash
   git commit -am "your-rejected-message"
   ```
 * Inform the user: "Pre-commit hook failed. Please fix the issues reported above and re-run the command manually."
 * END EXECUTION - Do not continue to step 7
