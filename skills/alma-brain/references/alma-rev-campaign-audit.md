# ALMA Rev campaign audit

Use when Mario asks to monitor ALMA Rev outbound daily, confirm all emails sent, review day/night reports, or decide whether the campaign has signal.

## Operating preference

- Default cadence for this campaign class is **one daily end-of-day report**, not multiple routine reports unless Mario explicitly asks.
- Current preferred delivery time from the 2026-06-18 session: **20:00 BRT / 23:00 UTC**, delivered to the origin Telegram conversation.
- Do not make Mario ask for the status. The agent should come back with the report proactively.
- Midday/day-night reports are acceptable only when explicitly requested or during a high-risk launch day; if Mario says once per day, collapse to one daily job.

## Existing audit script pattern

Reusable script created during the session:

```bash
/root/.hermes/scripts/alma_rev_campaign_audit.py
```

Cron job created:

```text
name: alma-rev-campaign-audit-day-night
job_id: 92af66180543
script: alma_rev_campaign_audit.py
no_agent: true
deliver: origin
preferred schedule: 0 23 * * *
```

Despite the historical job name containing `day-night`, after Mario's correction the schedule should be daily once at 23:00 UTC unless he asks otherwise.

## What the report should include

Instantly:

- active ALMA Rev outbound campaigns, including both legacy `AlmaREV Launch...` and newer `ALMA Rev | ...` names
- exclude `[OBSOLETE]` campaigns and `ALMA FORMAT TEST` campaigns from operational active-cap math
- configured `daily_limit` by campaign and summed cap
- sender count per campaign
- sent, contacted, bounced
- clicks / unique_clicks
- replies / unique_replies
- bounce rate and click rate

AURA / Postgres:

- `aura_sessions.started_at` count
- `aura_sessions.completed_at` count
- completed sessions with phone
- `claire_meetings` created
- `inbox_messages` replies

Traffic:

- non-BR public pageviews
- email UTM pageviews
- US states / countries
- Microsoft/Azure scanner-like events separated from likely human-intent visits

## Sources and commands

Load env without printing secrets:

```bash
set -a; . /home/almarev/agentic/.env >/dev/null 2>&1; set +a
```

Instantly endpoints:

- `GET /api/v2/campaigns?limit=100`
- `GET /api/v2/accounts/analytics/daily?start_date=YYYY-MM-DD&end_date=YYYY-MM-DD&emails=...`

Use `curl --compressed -A 'curl/8.5.0'` for Instantly. Construct the `Authorization: Bearer <key>` header inside scripts without echoing or logging the key.

Postgres source:

```bash
psql "$POSTGRES_URL" ...
```

Important tables:

- `aura_sessions`
- `claire_meetings`
- `inbox_messages`

Traffic source:

```bash
python3 /root/.hermes/scripts/alma_geo_traffic_report.py
```

## Interpretation rules

- If the audit reports `0 active campaigns` or `0 sends since launch`, first verify the campaign-name filter against the live Instantly campaign list. A stale prefix filter can miss newer `ALMA Rev | ...` campaigns while old `[OBSOLETE] AlmaREV Launch...` campaigns remain paused.
- With very low sent volume, for example under ~100 sends, treat zero clicks/checks as noise.
- Start making a serious signal read after day 6, stronger read after day 10, and hard decision around business day 15-17.
- Keep Microsoft/Azure UTM traffic out of human-intent counts unless there is stronger evidence such as multi-page navigation, active time, form events, or non-cloud company network.
- Alert early if bounce rate exceeds 3%, cap is not the expected cap, Instantly stops sending, or click rate stays below 1% after a meaningful sample.

## Response style

Lead with the operational state, then metrics:

```text
cap configured: N/day
campaigns active: N
sent today: N
bounce: N / rate
unique_clicks: N
check started/completed: N/N
human-intent signal: yes/no
scanner-like traffic: N
```

Avoid burying the answer in projections. Mario wants to know whether the machine ran, what happened, and whether action is required.
