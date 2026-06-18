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
- `PATCH https://api.instantly.ai/api/v2/campaigns/{campaign_id}` with `{"sequences": [...]}` to update campaign email bodies

Provider quirk: prefer `curl --compressed -A 'curl/8.5.0'` for Instantly API calls. Python `urllib` can trigger Cloudflare 1010 browser-signature blocks even with valid auth. Do not record the API key in logs or chat.
- `PATCH https://api.instantly.ai/api/v2/campaigns/{campaign_id}` with `{"sequences": [...]}` to update sequence bodies/signature variables

API quirk:

- Prefer `curl --compressed -A 'curl/8.5.0'` for Instantly API calls from Hermes scripts. Python `urllib` can trigger Cloudflare 1010 browser-signature blocks even with a valid API key. Capture the workaround as implementation detail, not as a claim that the API is down.

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

## Campaign signature ops

Instantly account-level signatures are inserted in campaign bodies with `{{accountSignature}}`, not `{{signature}}`. When Mario asks to use the sender/account owner's signature across campaigns:

1. Fetch campaign `sequences` live.
2. Back up the original sequence JSON before patching.
3. Skip placeholder campaigns whose body is only `{{body}}` unless explicitly asked.
4. Replace manual signoffs such as `Mário`, `Mário Thurler`, or `— Mario` with `<div>{{accountSignature}}</div>`.
5. Put `{{accountSignature}}` immediately before the visible unsubscribe footer when present.
6. Verify every target body contains `{{accountSignature}}` and that manual signoff blocks are gone.

Do not rely on the account signature being appended automatically. In sent-email inspection, rendered campaign emails only showed manual copy unless the sequence body included the account-signature variable.

## Current known shape from 2026-06-16 session

At the time of setup:

- 31 Instantly accounts registered.
- 5 accounts score >=99, 1 account score 98, 25 accounts score 0.
- The 4 active regional campaigns originally used only `mario@getalmarev.com` and `m.thurler@getalmarev.com`.
- `claire@getalmarev.com` had score 100 and was added to all 4 active regional campaigns.
- The new domain accounts were left unused for cold campaigns because score 0.

## Signature operations

Use this when Mario asks whether campaign emails use mailbox signatures, account owner signatures, or copy-level signatures.

- Instantly account-level signature variable is `{{accountSignature}}`, not `{{signature}}`.
- Account signatures must be configured per mailbox/account first. The campaign sequence body imports the sender account's signature only when `{{accountSignature}}` is present.
- To standardize active ALMA Rev campaigns, patch each sequence variant body to include `<div>{{accountSignature}}</div>` before the visible unsubscribe footer.
- Remove old manual signoff blocks from the copy before inserting the variable to avoid duplicate sender identity. Common old blocks: `Mário`, `Mário Thurler`, `— Mario`, `AlmaREV · Revenue Architecture`, `almarev.com/blog · instagram.com/almarevenue`.
- Keep visible unsubscribe footer/link in every step: `https://almarev.com/api/v1/public/unsubscribe?email={{email}}`.
- Do not edit placeholder routing campaigns whose body is just `{{body}}`; those inherit body content from lead/custom variables.
- Before patching sequences, save a JSON backup of the target campaign objects under `/root/.hermes/tmp/`.
- After patching, verify each targeted body has `{{accountSignature}}`, old manual blocks are gone, and the signature appears before the unsubscribe footer where present.

## Campaign copy and language audit

Use this when Mario asks whether active ALMA Rev outbound copy contains Portuguese, broken links, footer leakage, or other copy-level issues.

1. Fetch live `campaigns?limit=100` and filter active regional campaigns whose name starts with `AlmaREV Launch`.
2. Inspect every `sequences[].steps[].variants[]` subject and body, not just step 1.
3. Strip HTML for readable text, but also inspect raw HTML when auditing links because malformed anchors can hide inside tags.
4. For language checks, distinguish persuasive copy from compliance/legal footer. Portuguese in `Alma Consultoria em TI LTDA`, `Avenida das Nações Unidas`, `São Paulo`, or `Brasil/Brazil` is mailing-address/legal signal, not sales copy.
5. Also inspect account signatures via `accounts?limit=100` for Portuguese leakage because `{{accountSignature}}` renders from the sender account, not from the campaign body.
6. Treat non-breaking spaces and legal address accents as false positives in automated PT detection. Report them separately as footer/legal, not as body copy.
7. If minimizing Brazilian signal for US outbound, prefer changing only country display to `Brazil` and keep official street names intact.
8. When auditing CTAs, verify both visible text and `href`; broken `href` can exist even when visible URL text looks correct. Save backups under `/root/.hermes/tmp/` before patching sequence JSON.

## Response pattern

Lead with exact counts and timestamp, then state what is actually usable:

- total registered accounts
- warmed/usable accounts
- nearly ready accounts
- cold/warmup-only accounts
- campaign sender pools
- changes applied
- whether cron will stay silent or alert

For copy/language audits, answer in separated buckets:

- subject lines
- body copy
- CTA/link text
- unsubscribe/compliance text
- account signatures
- legal mailing address/footer

Avoid saying all registered accounts are production capacity. Avoid calling legal-address Portuguese "copy" unless explicitly discussing visual country/company-origin signal.
