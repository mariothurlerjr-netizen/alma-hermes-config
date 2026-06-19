# Instantly outbound status runbook — ALMA Rev

Use when Mario asks whether outbound emails were sent, whether there are more to send, opens/replies/bounces, or AURA Assessment conversions.

## Context
- Current ALMA Rev launch outbound was split into timezone campaigns named `AlmaREV Launch — ...`.
- Source of truth for live sends is Instantly API v2, not ORION logs.
- Relevant UI/API code exists in:
  - `/root/alma-aios/ops/outbound_report.py`
  - `/home/almarev/agentic/hub/src/app/api/outbound/route.ts`
  - `/home/almarev/agentic/hub/src/app/api/prospecting/funnel/route.ts`
- Brain reference: `02-alma-rev/launch-4-campanhas-timezone-2026-06-08.md`.

## Safe status check pattern
1. Load env from `/root/alma-aios/.env` without printing secrets.
2. Call `GET https://api.instantly.ai/api/v2/campaigns?limit=100` with `Authorization: Bearer $INSTANTLY_API_KEY`.
3. Filter campaigns where `name` starts with `AlmaREV Launch`.
4. For per-campaign analytics, call:
   - `GET /campaigns/analytics/overview?id={campaign_id}`
   - Use `id=...`; `campaign_id=...` can return global/default-looking totals and mislead.
   - If every timezone campaign shows the same `emails_sent_count` (ex: 55/55/55/55) or totals equal per-campaign count × campaign count, suspect this exact bug before interpreting volume. Patch any codepath using `campaign_id=` for analytics, including cron reporters and dashboard API routes.
5. For lead inventory/status, call:
   - `POST /leads/list` with JSON `{ "campaign": campaign_id, "limit": 100, "starting_after": cursor }`
   - Use `campaign`, not `campaign_id`, for this endpoint in the app code path.
   - Paginate with `next_starting_after`.
6. Derive:
   - loaded leads = count of paginated lead rows
   - sent emails = `emails_sent_count` from analytics overview
   - contacted unique = `contacted_count`
   - replies = `reply_count_unique`
   - bounces = `bounced_count`
   - opportunities = `total_opportunities`
   - last executed recipients = lead `status_summary.lastStep.timestamp_executed/from`
   - sent today = count lead rows whose `status_summary.lastStep.timestamp_executed` is >= today 00:00 UTC; label this as lead rows touched today, because `emails_sent_count` is cumulative and can include follow-up touches
7. For cadence spacing questions, fetch campaign details with `GET /campaigns/{id}?include_sequences=true` and inspect each sequence step:
   - `step.delay` + `delay_unit` gives spacing between touches
   - current ALMA Rev launch pattern should be D0, D+3, D+6, D+9, D+12 when every step has delay 3 days after step 1
   - remind Mario that Mon-Fri scheduling can turn D+3 into 4-5 calendar days when the due date lands on a weekend
8. For delivery-guarantee / "will these all go today?" questions, do not wait until the final send window closes. Compute backlog by campaign (`loaded - contacted`, minus bounced/suppressed where identifiable), compare against remaining window + daily limit, then schedule or run checks before each relevant timezone window closes:
   - mid-window check while there is still time to react
   - pre-close check 10-20 minutes before CT/ET/unzoned windows close
   - pre-close check before West closes
   - final post-window check only as verification, not first detection
   - Phrase answer as "capacity and window exist; Instantly can still throttle by warmup/reputation/eligibility" unless you have verified zero backlog.
9. Cross-check AURA conversions by POSTing unique lead emails to Cortex:
   - `POST {CORTEX_URL}/api/dashboard/aura-by-emails`
   - headers: `X-API-Key`, `X-Workspace-Id: aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa`, `X-Caller-Role: mario_admin`
   - body: `{ "emails": [...] }`

## Interpretation pitfalls
- If Mario asks about “active accounts/contas ativas” in Instantly, disambiguate before answering: `accounts/mailboxes` are sender inboxes; `leads loaded in active campaigns` are campaign inventory; `contacted_count` is active outreach. Do not label loaded leads as active accounts.
- Use this operational split for the launch campaigns: loaded = inventory/backlog in active campaigns; contacted = leads already touched and eligible for performance interpretation; queued = loaded - contacted, still waiting for first touch; emails_sent = total touches/messages.
- In Cortex Agenda, `/api/agenda/email-capacity` exposes this split as `campaign_inventory` from `GET /campaigns/analytics`; prefer it over raw campaign lead counts when explaining current funnel state.
- If `open_tracking=false`, report that opens cannot be meaningfully measured; do not say “nobody opened” as a behavioral conclusion.
- Text-only/link-light campaigns may also have click tracking effectively unavailable.
- `emails_sent_count` can count multiple touches/messages while `contacted_count` counts unique leads; explain both when totals look doubled.
- Bounce handling: Instantly keeps bounced rows in `/leads/list` for audit/inventory, but they stop being eligible for future touches. In current launch data, CT's 1 bounce appears as lead `status = -1` while analytics `bounced_count = 1`. Do not claim the row was deleted from the loaded list, say it is excluded from ongoing sends.
- Current timezone launch expected roughly 60/day across 2 getalmarev.com inboxes; incomplete totals during the day may simply mean a later timezone window is still open.
- The original single campaign `ea5dbfd4-c9ad-481b-9ad9-0b985d2f31ca` was superseded/deleted; a 404 on `GET /campaigns/{id}` is not necessarily a failure if the 4 timezone campaigns exist.

## Response shape Mario prefers
- Start with the operational answer, not methodology.
- Include: active campaigns, total loaded leads, sent so far vs expected/day, remaining today, replies, bounces, AURA conversions.
- Mention tracking caveat only once, crisply.
- No secrets, no raw emails unless explicitly requested.
