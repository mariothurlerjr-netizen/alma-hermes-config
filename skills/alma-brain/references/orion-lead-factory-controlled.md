# ORION Lead Factory controlled production

Use this when Mario asks to reactivate ORION, produce homologated ALMA Rev leads, generate Instantly-ready CSVs, or audit the daily lead factory.

## Operating mode

- Default mode is controlled and auditable, not full blast.
- Produce CSVs first; only load into Instantly when explicitly running import mode or when the current runbook says import is enabled.
- Keep Apollo email reveal as last resort after Prospeo and Findymail; cap reveal usage explicitly.
- Never open phone numbers or enrich LinkedIn URLs for this flow.

## Canonical daily spec

- Minimum: 300 homologated leads/day.
- Good: 600/day.
- Aggressive: 1,000/day.
- Instantly hard cap: 25,000 leads; stop/review at 20,000.
- US-only companies, 5-50 employees, professional services verticals.
- Valid contact titles: Owner, Founder, Co-Founder, Managing Partner, Managing Director, Principal, President, CEO at small company.
- Reject IC/intern/assistant/marketing/ops titles, BR, Microsoft/Azure/Fortune 500, generic/invalid email.

## Pipeline shape

1. Dedupe against current Instantly workspace before output.
2. Search/filter ICP in Apollo without opening email.
3. For candidates that pass ICP, identify owner/decision maker from firm site/search content if Apollo people payload is too thin.
4. Waterfall email in order:
   - Prospeo
   - Findymail
   - Apollo email reveal, capped and last resort only
5. Bucket output CSV by timezone:
   - `alma-rev-launch-ET.csv`
   - `alma-rev-launch-CT.csv`
   - `alma-rev-launch-MT.csv`
   - `alma-rev-launch-PT.csv`
6. Daily report must include raw collected, ICP passed, email found, reject reasons, homologation rate, Instantly loaded, space to 25k, days to cap, Apollo reveal credits/attempts.

## Implementation notes from 2026-06-19

- The controlled runner created in-session lives at `/home/almarev/agentic/ops/orion_lead_factory_daily.py` with wrapper `/root/.hermes/scripts/orion_lead_factory_daily.sh` and cron `orion-lead-factory-daily-briefing`.
- Apollo `mixed_people/search` is deprecated for API callers; use `mixed_people/api_search` for people search or `organizations/search` when company-level payload is needed.
- Apollo people search can return obfuscated/thin people payloads and org objects with only boolean `has_*` fields. For hard ICP, prefer organization search then classify/owner-ID via firm website/Tavily/LLM before any email spend.
- Instantly workspace capacity should be based on total loaded rows for cap accounting, while dedupe should use normalized existing email addresses.
- If multiple `.env` files carry Apollo keys, do a live low-cost API probe and prefer the key that authenticates; do not assume the repo-local `.env` is current.

## Implementation notes from 2026-06-25

- When the daily briefing says ORION is projecting zero leads, first distinguish legacy workers from the controlled factory: legacy `orion-watchdog`/`alma-orion@1..5` may be intentionally paused while `orion-lead-factory-daily-briefing` remains active and should be repaired independently.
- For ALMA Rev controlled factory throughput, prioritize vertical searches that have better contactability: `interior_design_firm` and `public_relations_agency` before broader consulting/law/accounting fallback.
- The wrapper may need to run with the Hermes/agentic Python venv when reusable dependencies like `tavily-python` are installed there. Prefer setting `PY=/home/almarev/agentic-v2/hermes-agent/.venv/bin/python` in `/root/.hermes/scripts/orion_lead_factory_daily.sh` over silently using `/usr/bin/python3`.
- When owner-id/Tavily yields zero accepted firms but Apollo org search is healthy, add a Snov domain-search fallback for verified personal prospects on the same domain. Count a fallback lead only when it has non-generic email, first/last name, and acceptable or absent decision-maker title.
- Empty env values can mask valid later values in another `.env`. While loading env files, skip blank values and only set defaults from non-empty values.
- Keep the run CSV-only unless Mario explicitly asks for Instantly import. Use smoke runs like `--target 3 --max-raw 80 --per-page 10 --max-apollo-reveal 0 --sleep 0` to verify yield without filling campaigns.

## Pitfalls

- Do not count a lead as homologated unless it has a valid non-generic personal email and all required fields.
- Do not spend Apollo reveal credits before the candidate passes ICP.
- Do not treat a smoke run with zero accepted leads as infra failure if reject reasons show filters are active; inspect source quality/query next.
- Do not auto-enable legacy ORION full-blast workers/watchdogs when Mario asks for controlled daily lead production.
- Do not let an exhausted Apollo key in `/root/alma-aios/.env` shadow a working key in `/home/almarev/agentic/.env`; probe each candidate key with a low-cost organization search and select the usable one.
- Do not rely solely on Tavily/owner-id for acceptance when Tavily quota is exhausted or site pages are thin. Fall back to domain-level verified personal emails, then report the weaker source in `source_quality`.
