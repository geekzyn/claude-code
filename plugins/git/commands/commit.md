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

# Conventional Commit Creation

Create commits following the [Conventional Commits](https://www.conventionalcommits.org) specification.

## Current Git State

- Current git status: !`git status`
- Current git diff (staged and unstaged changes): !`git diff HEAD`
- Current branch: !`git branch --show-current`
- Recent commits: !`git log --oneline -10`

## Conventional Commit Format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

**Valid Types** (per project's conventional-pre-commit config):

| Type | Purpose |
|------|---------|
| `feat` | A new feature |
| `fix` | A bug fix |
| `docs` | Documentation only changes |
| `refactor` | Code change that neither fixes nor adds feature |
| `test` | Adding or correcting tests |
| `build` | Changes to build system or dependencies |
| `ci` | Changes to CI configuration |
| `chore` | Other changes that don't modify src or test or formatting changes |
| `revert` | Reverts a previous commit |

**Scope Examples** (based on project structure):
- `adf` - Azure Data Factory changes
- `dbx` - Azure Data Databricks changes
- `metadata` - Metadata changes
- `ci` - CI/CD pipeline changes

**Rules:**
1. Type and description are required
2. Description should be lowercase, imperative mood ("add feature" not "added feature")
3. No period at the end of description
4. Breaking changes use exclamation mark after type or BREAKING CHANGE in footer

## User Input

$ARGUMENTS

## Instructions

> **⚠️ CRITICAL: When running `git commit`, ALWAYS use `-am` flag, NEVER use `-m` alone.**
> **Example: `git commit -am "message"` ✅ | `git commit -m "message"` ❌**

1. **Check for changes**:
   - If no staged or unstaged changes, inform user "Nothing to commit"
   - If there are unstaged changes, automatically stage all changes with `git add -A`

2. **If no commit message argument provided**:
   - Analyze the diff to understand what changed
   - Determine appropriate type based on changes:
     - New files in `src/data_factory/` -> `feat(adf)`
     - Modified pipeline files -> `refactor(adf)` or `fix(adf)`
     - Changes to `.pipelines/` -> `ci`
     - Changes in `src/metadata/` -> `feat(metadata)` or `fix(metadata)`
   - Generate 2-3 commit message options with different types/scopes where applicable
   - Use the `AskUserQuestion` tool to present options:
     - Option 1: Primary suggested message (add "(Recommended)" to the label)
     - Option 2-3: Alternative messages with different type or scope if applicable
     - User can select "Other" to provide a custom message
   - Use question header: "Commit msg"

3. **If commit message argument provided**:
   - Validate it follows conventional commit format
   - If invalid, suggest corrections
   - If valid, proceed with commit

4. **Create the commit**:
   - IMPORTANT: ALWAYS use `-am` flag (not just `-m`) to stage and commit together
   - The `-a` flag automatically stages all modified tracked files
   - Command format:
   ```bash
   git commit -am "<message>"
   ```
   - NEVER use `git commit -m` - always use `git commit -am`

6. **Handle pre-commit hook**:
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

7. **Post-commit**: Show commit hash and suggest next steps (push, create PR)

## Example Commit Messages

Based on project structure:
- `feat(adf): add new data pipeline for customer ingestion`
- `fix(adf): correct pipeline trigger configuration`
- `ci: improve build validation steps`
- `chore: update pre-commit hook versions`
- `refactor(metadata): restructure pipeline configuration tables`
