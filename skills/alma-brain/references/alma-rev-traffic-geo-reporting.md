# ALMA Rev traffic geo reporting

Use when Mario asks for page visits, country/state breakdown, whether Brazil is inflating traffic, or correlation between campaign lanes and page visits.

## Live source pattern

Primary server-side source is nginx access logs on the VPS:

- `/var/log/nginx/access.log*`
- Include rotated `.gz` files for multi-day windows.
- Filter to public page `GET 200` only.
- Exclude internal/watchdog/API/assets/scans before interpreting demand.

A reusable report script exists:

```bash
python3 /root/.hermes/scripts/alma_geo_traffic_report.py
```

It writes no external state except GeoIP cache under `/root/.hermes/tmp/ip_api_geo_cache.json` and prints a report covering:

- last 24h
- since `2026-06-16`
- non-BR public pageviews and unique IPs
- non-BR email UTM pageviews and unique IPs
- countries
- US states
- email UTM events by country/state/campaign lane/page

If the script is missing in a future environment, recreate the approach rather than treating the report as unavailable.

## Geo enrichment

The script uses `http://ip-api.com/batch` with fields:

```text
status,countryCode,country,regionName,region,city,query,isp,org
```

Cache responses to avoid rate limits. Do not paste raw secrets. nginx IPs are not secrets, but avoid turning reports into giant raw IP dumps unless Mario asks.

## Filtering rules

Filter out:

- Brazil (`countryCode = BR`) when Mario asks for campaign signal, because Mario's own platform usage can inflate Brazil.
- Internal server IPs, especially `72.60.136.186`.
- Watchdogs and probes by UA: `ALMA-Watchdog`, curl, python/aiohttp, Go HTTP clients, uptime monitors.
- Obvious bots/scanners: bot/spider/crawler, Palo Alto scans, Censys, zgrab, masscan, nuclei.
- Static assets and platform/API routes: `/_next/`, `/api/`, `/local/`, `/webhooks`, `/health`, image/font/js/css extensions.
- Exploit probes under blog paths such as `wp-`, `phpunit`, `vendor`.

Keep only ALMA Rev public pages such as:

- `/`
- `/aura`
- `/local`
- `/local/check`
- `/assessment` and `/assessment/...` when measuring the newer AURA/hub assessment path
- `/mario`
- `/blog`
- `/pt`
- `/pricing`
- `/diagnostic`
- `/contact`
- real `/blog/...` article slugs

Pitfall: the reusable script may lag behind live campaign paths. Before concluding “zero visits” for Local or assessment funnels, verify the script allowlist includes `/local`, `/local/check`, and `/assessment`; if not, run a targeted nginx-log count for those paths or update the script allowlist.

## Campaign lane mapping from UTM

Use `utm_source=email` as the email click subset.

Map lane from `utm_content`:

- contains or ends `_CT` / `CT` -> `CT`
- contains or ends `_West` / `West` -> `West`
- contains or ends `_US` / `US` -> `US unzoned`
- contains or ends `_ET` / `ET` -> `ET`
- `utm_campaign=ybhadu_bdgvif` -> `unknown legacy fgfc4`

Report the raw `utm_campaign` when lane cannot be inferred.

## Interpretation pitfalls

Microsoft traffic is often not human intent:

- Quincy, Washington, Microsoft Azure `westus2`
- Boydton, Virginia, Microsoft Azure `eastus2`
- Des Moines, Iowa, Microsoft Azure `centralus`
- London/City of London, Microsoft Azure `uksouth`

When email UTM visits are concentrated in Microsoft Azure IPs/regions, classify as likely Safe Links/security scanner/inbox prefetch. Treat as delivery/security-inspection signal, not buyer intent.

For ALMA Rev campaign signal, report three levels by default:

1. Raw public pageviews.
2. Non-BR public pageviews, because Mario's own usage inflates Brazil.
3. Non-BR excluding Microsoft/Azure (`isp` or `org` contains `Microsoft` or `Azure`) as the conservative human-intent baseline.

If email UTM non-BR excluding Microsoft/Azure is zero, say that directly. Do not soften it as “some clicks” without distinguishing scanner-like traffic.

Intent signal needs stronger evidence:

- non-BR
- not obvious cloud/datacenter ISP/org
- multi-page navigation
- repeated visit outside immediate delivery window
- form/event/POST or meaningful referrer
- identifiable company network, not Microsoft/AWS/GCP scanner infra

## Visit-growth playbook

When Mario asks how to maximize visits/clicks for ALMA Rev outbound, ground the answer in the current filtered baseline and optimize for **human-intent visits**, not raw clicks.

Recommended strategy:

1. Use landing pages by psychological tension, not a generic `/aura` destination:
   - `/revenue-leak-score`: fear of leaving money on the table, broadest starting point.
   - `/founder-sales-ceiling`: founder-led bottleneck and private insufficiency.
   - `/proposal-silence`: concrete professional-services pain after proposals.
   - `/growth-operator-gap`: operational envy, similar firms growing with less chaos.
2. Make email CTA a micro-click, not a meeting ask: “I put the 7-point check here”, “worth checking which leak applies”, “2-minute score, no signup”.
3. Add page events scanners cannot fake well: scroll 50%, active time >20s, start check, 3 answers, result viewed, CTA click, second page, return visit.
4. Run controlled tests in 150-account cells by angle. Winner is `start_check / 100 contacted`, not raw pageviews.
5. Segment follow-up by behavior:
   - Microsoft/Azure only: scanner probable, normal cadence.
   - Human-intent visit: short 1:1 follow-up tied to the page/angle.
   - Multi-event visitor: manual priority.

Use language around envy, insufficiency, founder bottleneck, and revenue leak. Avoid positioning the page as a generic demo or product tour.

## Response shape

Lead with exact timestamp/source, then:

1. Public non-BR pageviews and unique IPs.
2. Email UTM non-BR pageviews and unique IPs.
3. Top countries.
4. Top US states.
5. Email UTM by country/state/lane/page.
6. Interpretation: scanner vs likely human.

Keep Brazil separate when Mario says he is using/filling the platform.
