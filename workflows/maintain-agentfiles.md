---
on:
  schedule: weekly on Wednesday around 9am
  workflow_dispatch:

imports:
  - .github/agents/maintain-agentfiles.agent.md

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
  coding guidelines (instructions, skills, and reference examples) accordingly.
---

# Maintain Agent Files

Run the `maintain-agentfiles` agent on this repository.