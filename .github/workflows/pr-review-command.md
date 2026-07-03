---
description: |
  On-demand substantive PR review, invoked by a maintainer with the /review command.
  Shares its logic with the automatic PR review via a shared component.

on:
  slash_command: "review"

permissions:
  contents: read
  pull-requests: read
  actions: read

timeout-minutes: 15

imports:
  - .github/workflows/shared/pr-review.md
---

Review this pull request following the shared PR-review instructions.
