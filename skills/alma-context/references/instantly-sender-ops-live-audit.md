# Instantly sender ops live audit

Session-derived notes for answering "how is it programmed tomorrow?" and similar live sender questions.

## What to check live
- `list_accounts(100)` for the real mailbox pool.
- `/campaigns` for active campaigns, `daily_limit`, and `campaign_schedule`.
- `/emails?campaign_id=...&email_type=sent` grouped by `eaccount` or `from_address_email` for today's per-account send counts.

## Pitfalls
- `status == 1` alone does not mean the mailbox is usable for blast if `warmup_status == -1`.
- `campaign_schedule.schedules[0].timing.from/to` and `timezone` can be the only reliable source for when the campaign is actually allowed to run.
- Campaign top-level fields like `start_hour` / `end_hour` may be null even when the schedule exists.
- For daily send projections, use the campaign timezone date, not UTC by default.
- If `clean_streak_days` is missing, do not invent readiness. Fall back to the explicit warmup status and the live mailbox audit.

## Operational reading
- If the campaign is active and the current weekday is enabled in the schedule, the plan is live for that day.
- The useful answer is usually: active campaign name, window, cap, ready pool, paused-spam exclusions, and per-account send distribution.
