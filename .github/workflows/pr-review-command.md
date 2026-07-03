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
  copilot-requests: write # Native billing: use the Actions token for Copilot inference (billed to the org), no PAT needed.

timeout-minutes: 15

imports:
  - .github/workflows/shared/pr-review.md
---

Review this pull request following the shared PR-review instructions.
