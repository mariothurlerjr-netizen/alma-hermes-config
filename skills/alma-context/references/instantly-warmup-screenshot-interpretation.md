---
created_by: hermes
created_at: 2026-06-29T00:00:00Z
source: telegram
---

# Instantly warmup screenshot interpretation

Use this when Mario sends Instantly warmup screenshots and asks for account status.

## Rules
- Treat the screenshot as a *signal*, not the source of truth for the full pool.
- Use live mailbox state for the final classification.
- Classify by current clean streak:
  - `READY` = `clean_streak_days >= 5`
  - `WATCH` = `clean_streak_days 3-4`
  - `HOLD` = `clean_streak_days < 3` or paused for spam/error
- A screenshot can show recent spam recovery in the chart while the live state already reached 5 clean days.
- Do not add up screenshots to infer the total account inventory. Recompute the consolidated totals from live state.

## What to report
- mailbox email
- current status bucket
- clean streak days
- days left to READY when not READY
- whether the screenshot suggests recent spam recovery
