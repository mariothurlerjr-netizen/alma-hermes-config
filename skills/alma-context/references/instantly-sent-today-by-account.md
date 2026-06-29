# Instantly: sent today by account

Use this when Mario asks for "hoje de quais contas e qtas por conta" or any per-account send breakdown.

## Rule
- ` /campaigns/analytics ` gives campaign-level totals, not per-account send counts.
- To get per-account sends for today, read ` /emails ` with `campaign_id` + `email_type=sent` and bucket by `eaccount` or `from_address_email`.
- Respect the campaign timezone when deciding what counts as "today".

## Pitfall
- Do not infer per-account counts from campaign totals. A single campaign can mix several sender inboxes.
- Do not mix UTC day boundaries with campaign-local day boundaries.

## Output shape
- Campaign name
- account email
- sent_today count
- optional total campaign sends today
