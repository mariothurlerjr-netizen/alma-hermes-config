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
- `GET https://api.instantly.ai/api/v2/accounts/analytics/daily?start_date=YYYY-MM-DD&end_date=YYYY-MM-DD&emails=...` for per-mailbox campaign sends
- `POST https://api.instantly.ai/api/v2/accounts/warmup-analytics` with `{"emails": [...]}` for warmup sent/received/inbox/spam per mailbox
- `PATCH https://api.instantly.ai/api/v2/campaigns/{campaign_id}` with `{"email_list": [...]}` to update campaign sender pool
- `PATCH https://api.instantly.ai/api/v2/campaigns/{campaign_id}` with `{"daily_limit": N}` to update campaign cap
- `POST https://api.instantly.ai/api/v2/campaigns/{campaign_id}/pause` to pause campaigns. Prefer this over `PATCH {"status":2}` when PATCH returns Instantly 500.
- `POST https://api.instantly.ai/api/v2/campaigns/{campaign_id}/activate` to activate campaigns
- `PATCH https://api.instantly.ai/api/v2/campaigns/{campaign_id}` with `{"sequences": [...]}` to update campaign email bodies

Provider quirk: prefer `curl --compressed -A 'curl/8.5.0'` for Instantly API calls. Python `urllib` can trigger Cloudflare 1010 browser-signature blocks even with valid auth. Do not record the API key in logs or chat.
- `PATCH https://api.instantly.ai/api/v2/campaigns/{campaign_id}` with `{"sequences": [...]}` to update sequence bodies/signature variables

API quirk:

- Prefer `curl --compressed -A 'curl/8.5.0'` for Instantly API calls from Hermes scripts. Python `urllib` can trigger Cloudflare 1010 browser-signature blocks even with a valid API key. Capture the workaround as implementation detail, not as a claim that the API is down.

Existing helper:

- `/home/almarev/agentic/integrations/instantly/accounts.py`
- Cron script created for the daily conservative optimizer: `~/.hermes/scripts/instantly_campaign_optimizer.py`
- Cron job: `instantly-campaign-optimizer-daily`, job id `48ee17f65d77`, schedule `0 13 * * *`, `no_agent=True`
- Daily campaign audit details: `references/alma-rev-campaign-audit.md` (covers end-of-day Telegram report, cap verification, sent/click/reply/check metrics, and scanner-vs-human traffic read)

## Interpretation

- Instantly `accounts` count is registered mailboxes, not usable sending capacity.
- `status=1` means connected/active in Instantly, but score still matters.
- `stat_warmup_score >= 99` is treated as warmed enough for conservative campaign inclusion.
- Score `0` accounts stay in warmup only. Do not add them to cold campaigns.
- `email_list` on a campaign is the actual sender pool for that campaign.
- `daily_limit` is campaign-level volume. Updating sender pool without raising daily_limit improves distribution without increasing total volume.
- Instantly does not reliably enforce a fixed per-mailbox quota inside one shared campaign sender pool. Treat per-mailbox numbers as an operating allocation/expectation derived from campaign caps and sender mix. If Mario needs hard per-score or per-mailbox control, split senders into separate campaigns/lane-specific caps.
- When applying tomorrow's plan, patch `daily_limit` and `email_list` separately if a combined PATCH errors. Verify with a fresh `GET /campaigns?limit=100` after writes.

## Sender inventory classification

When scaling ALMA Rev sender pools, do not treat excluded accounts as abandoned or deleted. "Exclude" should mean excluded from the current cold-campaign sender pool only. Keep every mailbox classified so warming/near-ready accounts are not forgotten.

Use this operating model:

- `active`: score 99-100, `status=1`, no setup issue. Eligible for real campaign volume.
- `near-ready`: score 96-98, `status=1`, no setup issue. Eligible only for controlled low-volume drip if Mario wants capillarity, typically 1-2 cold emails/day per inbox, or keep warmup-only if risk control matters more.
- `warming`: below 96 or recently added. Keep warmup active, zero cold outbound.
- `quarantine`: bounce/error/reputation/config issue. Zero cold outbound until the issue is resolved.

Daily limit semantics matter: Instantly campaign `daily_limit` is campaign-level volume, not per-sender volume. Adding more senders improves distribution and reputation load, but it does not increase total campaign sends unless `daily_limit` is also raised. If four regional campaigns each have `daily_limit=110`, the total ceiling is 440/day.

For scaling plans, report both:

- campaign total daily ceiling, summed across active campaigns
- average expected sends per eligible mailbox/day

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

## Campaign audit automation

When Mario asks for “audit day/night”, “audits from day to night”, or continuous review of an ALMA Rev outbound launch, create or update a script-only cron that reports twice daily rather than relying on one-off manual checks.

Default audit content:

- Instantly campaign state: active regional campaigns, `daily_limit`, sender count, total configured cap.
- Instantly account analytics for today and launch window: sent, contacted, bounced, clicks, unique clicks, replies, unique replies, bounce rate, click rate.
- AURA/Postgres funnel: check started (`aura_sessions.started_at`), check completed (`completed_at`), completed with phone, `claire_meetings`, `inbox_messages` replies.
- Traffic signal: run/filter ALMA geo traffic report and separate non-BR, email UTM, and Microsoft/Azure scanner-like traffic.
- Alerts: configured cap drift from expected plan, bounce rate above threshold, click rate below 1% after meaningful send volume.

Implementation pattern used successfully:

- Script path: `/root/.hermes/scripts/alma_rev_campaign_audit.py`.
- Cron shape: `no_agent=True`, schedule `0 15,23 * * *`, deliver to origin/current Telegram thread.
- Use Instantly via curl with `-A 'curl/8.5.0'`; build auth header inside script without printing the token.
- Use `psql "$POSTGRES_URL"` for DB metrics if Python `psycopg2` is unavailable in the Hermes runtime.
- For the launch read, treat day 6 onward as the first serious interpretation window; before that, report data but avoid overfitting.

## Response pattern

Lead with exact counts and timestamp, then state what is actually usable:

- total registered accounts
- warmed/usable accounts
- nearly ready accounts
- cold/warmup-only accounts
- campaign sender pools
- changes applied
- whether cron will stay silent or alert

When Mario asks why a daily limit is low, whether senders are being forgotten, or which campaign to use for the next day, give a **per-mailbox operating plan**, not just campaign-level totals. Include, for each relevant account:

- email address
- `stat_warmup_score`
- `status`, `setup_pending`, `daily_limit`, `warmup_status`
- campaign sent today and over the last 7 days from `GET /api/v2/accounts/analytics/daily`
- warmup sent/received/inbox/spam from `POST /api/v2/accounts/warmup-analytics`
- planned next-day send allocation for that mailbox

Mario's preference in this class of task: do not wait for perfect 99-100 score if an inbox is connected, setup-clean, warmup-active, has real warmup traffic, and no bounce/reputation issue. Put lower-score healthy inboxes into a low-volume active lane instead of letting them be forgotten. Example operating lanes:

- `active_full`: score 96-100, clean status, typical allocation 4-8 cold emails/day depending on recent sent volume and domain mix.
- `active_low_volume`: score 80-95 with warmup healthy and only minor spam signal, typical allocation 1-3/day.
- `warmup_only`: setup/config/status issue, broken warmup, or insufficient warmup traffic.
- `quarantine`: explicit reputation/config/bounce issue.

For next-day campaign plans, report both the summed campaign `daily_limit` and the implied per-mailbox allocation. Make the distinction explicit: campaign caps are the configured guarantees; per-mailbox counts are expected allocation unless senders are split into separate campaigns. If the allocation plan sums to 137, set campaign caps around 140 and split by region rather than over-allocating every regional campaign.

For copy/language audits, answer in separated buckets:

- subject lines
- body copy
- CTA/link text
- unsubscribe/compliance text
- account signatures
- legal mailing address/footer

Avoid saying all registered accounts are production capacity. Avoid calling legal-address Portuguese "copy" unless explicitly discussing visual country/company-origin signal.
