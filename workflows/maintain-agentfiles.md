---
on:
  schedule: weekly on Wednesday around 9am
  workflow_dispatch:

permissions:
  issues: read
  contents: read
  pull-requests: read
  actions: read

tools:
  github:
    toolsets: [default, search]
  edit:
  bash:
    - "find *"
    - "cat *"
    - "ls *"
    - "wc *"

safe-outputs:
  create-pull-request:
    title-prefix: "[guidelines] "
    labels: [automation, guidelines]
    draft: true

timeout-minutes: 60

description: >
  Analyzes PR review feedback to identify recurring patterns and updates
  coding guidelines, instruction files, and skills accordingly.
---

# PR Review Feedback Analyzer and Agent Files Updater

You are a senior engineering coach. Your job is to analyze all pull request review feedback
from the repository since the last workflow run and distill recurring patterns into
actionable coding guidelines.

## Step 1: Determine the Last Workflow Run

Use the GitHub API to find the last successful run of this workflow (excluding the current run):

```
GET /repos/${{ github.repository }}/actions/workflows/maintain-agentfiles.yml/runs?status=success&per_page=2
```

Take the second result (the most recent _prior_ run) and extract its `created_at` timestamp. That timestamp is your cutoff date (`$LAST_RUN_DATE`).

If no previous successful run exists, fall back to 30 days ago as the cutoff.

## Step 2: Gather Pull Requests

Use the GitHub search tool to find all **merged** pull requests in this repository
since the last workflow run. Use a search query like:

```
is:pr is:merged repo:${{ github.repository }} merged:>=$LAST_RUN_DATE
```

If no merged PRs are found, output a noop message saying "No merged PRs found since the last workflow run" and stop.

## Step 3: Collect All Review Feedback

For **each** merged pull request found:

1. Fetch all **review comments** (inline code review comments on diffs).
2. Fetch all **pull request reviews** (the top-level review body with approve/request-changes/comment).
3. Fetch all **issue comments** (general conversation comments on the PR).

Collect every piece of textual feedback. Ignore bot-generated comments (from GitHub Actions, Dependabot, etc.).
Focus only on human reviewer feedback.

## Step 4: Analyze for Recurring Patterns

Examine all collected feedback across all PRs and identify **recurring themes and patterns**.
Focus specifically on:

- **Code quality**: DRY violations, complexity, readability, maintainability concerns
- **Architecture**: Service design, separation of concerns, dependency management, layering
- **Programming style**: Idiomatic Ruby/Rails patterns, functional vs OOP preferences, method design
- **Engineering practices**: Error handling, logging, performance, security, testing approaches
- **Code style**: Formatting, naming conventions, comment usage, code organization
- **Naming**: Variable names, method names, class names, module names
- **Testing**: Test structure, mocking patterns, coverage expectations, test clarity
- **Specific implementation patterns**: Preferred ways to solve common problems (e.g., service objects, form objects, query objects, etc.)

For each pattern you identify, note:
- How many times it appeared across different PRs
- Example quotes from the actual review comments
- Whether the feedback was consistent (same direction) or contradictory

**Only include patterns that appear in at least 2 different PRs.**
Ignore one-off nitpicks or PR-specific feedback. Try to focus on finding patterns.

## Step 5: Read Existing Guidelines

Read the existing instruction and skill files in the repository to understand what guidelines are already documented:

- `.github/instructions/*.instructions.md` — Ruby and Rails coding style
- `.github/skills/**/*` — API design guidelines
- `.github/agents/*.agent.md` - Agents

## Step 6: Update or Create Guideline Files

Based on the patterns identified in Step 4, update the existing files or create new ones:

### Updating Existing Files

If the pattern fits within the scope of an existing instruction or skill file:

- Add the new guideline in the appropriate section
- Use the same format and style as existing entries (bullet points, code examples with Good/Bad patterns)
- Include a brief code example where it helps clarify the guideline
- Do NOT remove or change existing guidelines unless the review feedback explicitly contradicts them

### Creating New Files

If the patterns form a coherent new topic not covered by existing files, create a new file:

- **New instruction file**: `.github/instructions/<topic>.instructions.md` with proper YAML frontmatter:
  ```yaml
  ---
  description: '<description of the instruction scope>'
  applyTo: '<glob pattern>'
  ---
  ```

- **New skill file**: `.github/skills/<topic>/SKILL.md` with proper YAML frontmatter:
  ```yaml
  ---
  name: <topic>
  description: '<description of the skill>'
  ---
  ```

Follow these formatting rules for all files:
- Use `#` for the main title, `##` for sections, `###` for subsections
- Write in imperative mood ("Use", "Prefer", "Avoid")
- Be specific and actionable — every guideline should be immediately applicable
- Include code examples with Good/Bad patterns where helpful
- Keep entries concise — one bullet point per guideline

## Step 7: Create a Summary

In the pull request description, include:

1. **Summary of analyzed PRs**: How many PRs were analyzed, date range
2. **Identified patterns**: List each pattern with frequency count and example quotes
3. **Changes made**: Which files were updated or created, and what was added
4. **Patterns skipped**: Any patterns that were too infrequent or ambiguous to codify

## Important Rules

- **SECURITY**: Do not include any secrets, tokens, or sensitive information from PR reviews.
- **Conservative changes**: Only codify patterns that are clearly recurring and consistent. When in doubt, skip.
- **Preserve existing content**: Never remove existing guidelines. Only add or refine.
- **Match existing style**: New entries must match the tone, format, and specificity of existing entries in each file.
- **No duplicates**: Check that you're not adding a guideline that already exists in the file.