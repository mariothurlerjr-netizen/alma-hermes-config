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
7. Cross-check AURA conversions by POSTing unique lead emails to Cortex:
   - `POST {CORTEX_URL}/api/dashboard/aura-by-emails`
   - headers: `X-API-Key`, `X-Workspace-Id: aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa`, `X-Caller-Role: mario_admin`
   - body: `{ "emails": [...] }`

## Interpretation pitfalls
- If `open_tracking=false`, report that opens cannot be meaningfully measured; do not say “nobody opened” as a behavioral conclusion.
- Text-only/link-light campaigns may also have click tracking effectively unavailable.
- `emails_sent_count` can count two inboxes/messages while `contacted_count` counts unique leads; explain both when totals look doubled.
- Current timezone launch expected roughly 60/day across 2 getalmarev.com inboxes; incomplete totals during the day may simply mean a later timezone window is still open.
- The original single campaign `ea5dbfd4-c9ad-481b-9ad9-0b985d2f31ca` was superseded/deleted; a 404 on `GET /campaigns/{id}` is not necessarily a failure if the 4 timezone campaigns exist.

## Response shape Mario prefers
- Start with the operational answer, not methodology.
- Include: active campaigns, total loaded leads, sent so far vs expected/day, remaining today, replies, bounces, AURA conversions.
- Mention tracking caveat only once, crisply.
- No secrets, no raw emails unless explicitly requested.
