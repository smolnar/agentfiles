---
description: "Autonomous PR fixing agent. Evaluates PR changes against coding standards, fixes issues, runs tests, and repeats until clean."

on:
  pull_request:
    types: [review_requested]

engine: copilot
model: claude-opus-4.6

imports:
  - .github/agents/autofix.agent.md

permissions:
  contents: read
  pull-requests: read

concurrency:
  cancel-in-progress: true
  group: autofix-agent-${{ github.ref }}

runtimes:
  ruby:
    version: "3.3.6"
  node:
    version: "22"

safe-outputs:
  push-to-pull-request-branch:
    max: 5
    protected-files: fallback-to-issue
  add-comment:
    max: 3
    hide-older-comments: true
  create-pull-request-review-comment:
    max: 10
---

# Github Copilot - Autofix

**IMPORTANT**: Proceed only when the review is required by either `@copilot` or `@copilot[bot]`. Skip this workflow if the review is requested by some other user and not a copilot bot users.

**Instructions**:

Run the autofix agent on this pull request - #${{ github.event.pull_request.number }}.
