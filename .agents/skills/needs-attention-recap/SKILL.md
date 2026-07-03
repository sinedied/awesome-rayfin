---
name: needs-attention-recap
description: |
  Use when a maintainer wants a prioritized "what needs attention" recap of a GitHub repo —
  e.g. "what needs attention in awesome-rayfin?", "give me the weekly repo recap", "what
  should I look at this week?", "any stale PRs or untriaged issues?", "maintenance digest".
  Surveys untriaged/unanswered issues, stale and review-ready PRs, the good-first-issue /
  help-wanted backlog, and recent activity, then produces a ranked digest with direct links.
  Works for any cadence (daily, weekly, ad hoc) and any runner (local agent, cron, cloud
  automation) — the caller decides when to run it. Do NOT use to triage a single issue, to
  review a specific PR, or to make any changes — this skill is read-only and reporting-only.
metadata:
  author: microsoft
  version: "0.1.0"
compatibility: Requires the GitHub CLI (`gh`) authenticated with read access to the target repo.
---

# Needs-Attention Recap

Produce a concise, prioritized recap of what a maintainer should look at in a GitHub repo.
**Read-only**: never label, comment, close, merge, or otherwise modify anything — only report.

## Inputs
- **repo** — `owner/name`. Default to the current repository if run inside a clone
  (`gh repo view --json nameWithOwner -q .nameWithOwner`).
- **window** — lookback for "recent activity" and staleness. Default **7 days**. The caller
  may override (e.g. "last 24h", "last 2 weeks").

## Steps

1. **Confirm the repo and window**, then read its labels once so you use ones that actually
   exist (don't invent labels):
   ```bash
   gh label list -R <repo> --limit 100
   ```

2. **Gather data with `gh`** (JSON, so you can sort/filter). Adjust label names to the repo's
   actual labels from step 1. Run only the queries relevant to what exists:
   - **Untriaged issues** — open issues missing a triage signal (no `triaged` label, or no
     labels at all):
     ```bash
     gh issue list -R <repo> --state open --limit 100 \
       --json number,title,labels,createdAt,updatedAt,comments,author,url
     ```
     Treat as untriaged: no labels, or labeled `needs-info`, or not labeled `triaged`.
   - **Unanswered issues** — open issues with zero comments (or no maintainer reply),
     oldest first.
   - **Open PRs** — with review state and staleness:
     ```bash
     gh pr list -R <repo> --state open --limit 100 \
       --json number,title,isDraft,reviewDecision,createdAt,updatedAt,author,labels,url,mergeable,statusCheckRollup
     ```
     Flag: **review-ready** (not draft, `reviewDecision` empty/`REVIEW_REQUIRED`), **stale**
     (no update within the window), **failing checks** (`statusCheckRollup` has failures),
     and **conflicts** (`mergeable == "CONFLICTING"`).
   - **Contributor backlog** — open `good first issue` / `help wanted` issues, so newcomers
     have something to pick up:
     ```bash
     gh issue list -R <repo> --state open --label "good first issue" --json number,title,url
     gh issue list -R <repo> --state open --label "help wanted" --json number,title,url
     ```
   - **Recent activity** — issues/PRs opened, closed, or merged within the window, for a
     pulse of momentum (`gh issue list --search "updated:>=<date>"`,
     `gh pr list --search "merged:>=<date>"`).

3. **Rank by urgency**, most urgent first. Suggested priority order:
   1. PRs with failing CI or merge conflicts (blocked, needs author/maintainer).
   2. Review-ready PRs waiting the longest.
   3. Untriaged or unanswered issues, oldest first.
   4. Stale PRs/issues (no movement in the window) that may need a nudge or closing.
   5. Contributor backlog health (counts of `good first issue` / `help wanted`).
   Use age, staleness, and activity to break ties. Don't pad the list — surface what genuinely
   needs a human, not everything.

4. **Output** a scannable Markdown recap:
   - A one-line header: repo, date, and window (e.g. *"awesome-rayfin — needs attention, last 7 days"*).
   - A 1–2 sentence **TL;DR** of the most important thing(s).
   - Sections in priority order (**PRs needing review**, **Blocked PRs**, **Issues to triage**,
     **Unanswered issues**, **Contributor backlog**, **Recent activity**). Omit any section
     that's empty.
   - Each item on one line: `#<number> <title> — <why it needs attention> (<age/last update>)`
     with a direct link. Keep it short.
   - End with a short **Suggested next actions** list (3–5 bullets), phrased as recommendations
     for the human — never actions you took.
   - If nothing needs attention, say so plainly instead of manufacturing items.

## Usage examples
This skill has **no built-in schedule** — invoke it however you like:
- **Ad hoc / local:** ask your agent *"run the needs-attention recap for microsoft/awesome-rayfin (last 7 days)"*.
- **Weekly digest:** wire the same prompt into any scheduler — a gh-aw workflow on a `schedule:`
  cron, a GitHub Agents automation, or an external cron that runs your agent with this skill.
- **Different cadence/window:** pass a different window (e.g. *"last 24 hours"* for a daily standup digest).

## Gotchas
- Read the repo's real labels first (step 1); label names vary between repos.
- `reviewDecision` is empty for PRs that have had no review yet — that usually means
  *review-ready*, not *approved*. Don't confuse the two.
- Drafts are not review-ready; list them separately or skip them.
- Stay read-only. If asked to also act (label, comment, close), that's a different skill —
  do the recap and hand off.
