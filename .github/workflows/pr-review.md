---
description: |
  Automatic substantive review of pull requests to the Rayfin templates gallery,
  on open/update. Shares its logic with the /review command via a shared component.

on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  contents: read
  pull-requests: read
  actions: read

timeout-minutes: 15

imports:
  - .github/workflows/shared/pr-review.md
---

Review this pull request following the shared PR-review instructions.
