# Instantly warmup streak check, June 2026

Use this when Mario asks for counts like "who is on day 4/5/6" or wants to test tomorrow's warmup cut.

## What to check
- `inbox_health_state.clean_streak_days` is the source of truth for streak buckets.
- `inbox_health_state.state` tells you whether the inbox is still `WARMING`, already `PAUSED_SPAM`, or `READY`.
- `inbox_health_state.current_cold_limit` is the current send cap and can be a better operational bucket than streak alone.
- `inbox_health_log` is useful for verifying whether a mailbox is actually incrementing day by day or stuck on a spam signal.

## Pitfall
- Do not infer a "day 4 / day 5" population from the warmup UI alone.
- Do not mix up streak buckets with the current ramp bucket.
- If Mario is thinking in terms of a desired ramp curve, restate the live state first, then compare it with that intended curve.

## Practical pattern
1. Count exact streak buckets, especially `clean_streak_days = 4`, `5`, `6`, `7`.
2. Separate them from `current_cold_limit` buckets, because a mailbox can be 6 clean days while still being capped at 3, 10, 20, or 30.
3. If the human estimate differs from the DB total, double-check with both the state table and the daily log before answering.
4. For automation changes, confirm the current ramp constants in code before suggesting a policy like "increase by 4 up to 20".
