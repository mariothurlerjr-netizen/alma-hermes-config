# ALMA Rev Instantly sender ops

Use when Mario asks to verify Instantly accounts/mailboxes, warmup scores, sender capacity, or to improve active cold email campaigns.

## Live lookup pattern

Load env without printing secrets:

```bash
set -a; . /home/almarev/agentic/.env >/dev/null 2>&1; set +a
```

Useful API endpoints:

- `GET https://api.instantly.ai/api/v2/accounts?limit=100`
- `GET https://api.instantly.ai/api/v2/campaigns?limit=100`
- `PATCH https://api.instantly.ai/api/v2/campaigns/{campaign_id}` with `{"email_list": [...]}` to update campaign sender pool

Existing helper:

- `/home/almarev/agentic/integrations/instantly/accounts.py`
- Cron script created for the daily conservative optimizer: `~/.hermes/scripts/instantly_campaign_optimizer.py`
- Cron job: `instantly-campaign-optimizer-daily`, job id `48ee17f65d77`, schedule `0 13 * * *`, `no_agent=True`

## Interpretation

- Instantly `accounts` count is registered mailboxes, not usable sending capacity.
- `status=1` means connected/active in Instantly, but score still matters.
- `stat_warmup_score >= 99` is treated as warmed enough for conservative campaign inclusion.
- Score `0` accounts stay in warmup only. Do not add them to cold campaigns.
- `email_list` on a campaign is the actual sender pool for that campaign.
- `daily_limit` is campaign-level volume. Updating sender pool without raising daily_limit improves distribution without increasing total volume.

## Conservative optimization rule used for ALMA Rev

1. List all accounts and campaigns.
2. Find active regional campaigns where name starts with `AlmaREV Launch`.
3. Find warmed accounts on the same mature sending domain, currently `getalmarev.com`, with `stat_warmup_score >= 99` and `status=1`.
4. Patch each active regional campaign's `email_list` to include those warmed same-domain accounts.
5. Preserve existing senders.
6. Do not remove senders automatically.
7. Do not raise `daily_limit` automatically.
8. Stay silent when no changes are made; alert only on changes or errors.

## Current known shape from 2026-06-16 session

At the time of setup:

- 31 Instantly accounts registered.
- 5 accounts score >=99, 1 account score 98, 25 accounts score 0.
- The 4 active regional campaigns originally used only `mario@getalmarev.com` and `m.thurler@getalmarev.com`.
- `claire@getalmarev.com` had score 100 and was added to all 4 active regional campaigns.
- The new domain accounts were left unused for cold campaigns because score 0.

## Response pattern

Lead with exact counts and timestamp, then state what is actually usable:

- total registered accounts
- warmed/usable accounts
- nearly ready accounts
- cold/warmup-only accounts
- campaign sender pools
- changes applied
- whether cron will stay silent or alert

Avoid saying all registered accounts are production capacity.
