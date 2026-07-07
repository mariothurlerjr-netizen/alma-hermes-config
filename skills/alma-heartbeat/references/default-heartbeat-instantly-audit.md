# Default heartbeat Instantly audit pattern

Use when `/root/.hermes/HEARTBEAT.md` asks the default profile to write silent Instantly status files.

## Scope

- Default profile never delivers a proactive message. Final cron output remains exactly `[SILENT]`.
- Execute only items in the current BRT window and only if not already marked in `/home/almarev/brain/agents/default/heartbeat-state.md` for the current BRT date.
- Write findings to `/home/almarev/brain/agents/default/status-instantly.md`, `/home/almarev/brain/agents/default/checkin-manha.md` or `/home/almarev/brain/agents/default/checkin-noite.md`, then update heartbeat state.

## Live metric sources

Use brain for context, but validate current numbers live:

1. `TZ=America/Sao_Paulo date` to determine current window.
2. Instantly API v2 via existing `/home/almarev/agentic` integration:
   - `/campaigns` for active campaigns, daily limits and sender allocation.
   - `/accounts` for connected/setup/warmup counts and warmup score min/avg/max.
   - `/campaigns/analytics` with `start_date`/`end_date` for sends, replies, bounces and campaign rows.
3. Postgres `alma_agentic` for LANCE replies, meetings, discoveries and AURA visits when credentials are available through existing project config or local service defaults.
4. Local gate files:
   - `/home/almarev/agentic/logs/seed-tests.json`
   - `/home/almarev/agentic/logs/onda1-preauth.json`

## Important split

Separate metrics into these buckets instead of mixing totals:

- `ALMAREV-V3-*` sprint campaigns.
- Legacy/non-V3 ALMA Rev campaigns.
- ALMA Local campaigns.
- Instantly account/warmup health.
- Postgres business outcomes, especially LANCE replies, interested replies, meetings and discoveries.

## SHIELD-worthy deviation

Append one line to `/home/almarev/brain/agents/shield/inbox.md` when any of these are true:

- ALMAREV-V3 has non-zero sends while `seed-tests.json` is absent and only `onda1-preauth.json` exists.
- Sends increased materially while replies remain zero and prior state had a scale/gate warning.
- Bounce/spam trips or approaches configured circuit-breaker thresholds.
- Warmup/account health degrades in a way that changes sending safety.

The SHIELD line should be short: timestamp, default-heartbeat window, numeric observation, why it matters, and suggested SHIELD eval. Do not page Mario directly from default.

## Output files

`status-instantly.md` should include:

- source line, explicitly naming live endpoints plus Postgres if used;
- campaign/account summary;
- sends/replies/bounces/spam split by V3 and total;
- gate/deviation section;
- active campaigns observed;
- action suggested for fleet agents.

`heartbeat-state.md` should include:

- BRT date and window;
- exact items executed;
- latest live metrics;
- any deviation registered;
- per-window execution markers for the current date.

## Night-window check-in pattern

For the `18:00-18:25` default heartbeat window, run both night tasks when not already marked for the current BRT date:

1. Read `checkin-manha.md` before writing the night check-in, so the night file compares morning plan vs realized state rather than producing a standalone status dump.
2. Collect fresh live Instantly + Postgres metrics, then overwrite `status-instantly.md` with a night section. Do not append a second stale morning report unless the active `HEARTBEAT.md` explicitly asks for append semantics.
3. Write `checkin-noite.md` with: what closed, what stuck and why, what changed vs morning, status by agent/area, a single Mario-dependency queue, and tomorrow's priority.
4. Update `heartbeat-state.md` with distinct markers such as `instantly_status_morning_executed` and `instantly_status_night_executed`; avoid one ambiguous `instantly_status_executed` marker that can hide whether the night status ran.
5. If V3 sends continue while `seed-tests.json` is absent, append a new SHIELD inbox line with the updated numbers even if a morning line already exists. This is a persistent gate/governance deviation, not a duplicate deliverability alarm.

## Practical collector pattern

When Python `requests` helpers may hit provider quirks or hide endpoint details, a temporary Python collector that shells out to `curl --compressed -A 'curl/8.5.0'` is acceptable for the heartbeat. Keep it secret-safe:

- load `/home/almarev/agentic/.env` without printing values;
- pass the Instantly token only as an Authorization header;
- emit aggregate JSON only, never raw env or secrets;
- query `/campaigns`, `/accounts`, and `/campaigns/analytics` for both today and sprint start;
- split analytics into `ALMAREV-V3-*`, Rev legacy, Local, and total;
- use `psql "$POSTGRES_URL" -t -A` for Postgres metrics if Python DB deps are not guaranteed in the Hermes runtime.
