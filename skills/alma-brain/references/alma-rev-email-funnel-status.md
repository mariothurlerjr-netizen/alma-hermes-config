# ALMA Rev email funnel status lookup

Use this when Mario asks variants of:
- "por que temos 630 contas ativas?"
- "quantas contas receberam email 1/2/3?"
- "quantas estão dentro da cadeia/cadência?"
- "status dos emails / por etapa"

## Durable source pattern

1. Read `STATUS.md` and search the brain for the exact number first, especially `630`, campaign names, or `funnel_snapshots`.
2. Brain usually gives the last audited explanation, but Mario often wants live counts. If the question asks "temos" / "agora" / current funnel, query live systems through the agentic stack, not only the vault.
3. The currently useful live reporter is:

```bash
python3 /home/almarev/agentic/ops/funnel_report.py daily
```

It pulls Instantly campaign analytics and `/campaigns/analytics/steps`, then tries DB snapshot/status. If the active Python env lacks `psycopg2`, the Instantly step counts can still be valid while DB status/snapshot fails. Do not treat that as total failure, but state the source boundary.

4. For DB-side status counts, use `psql` with env loaded from `/home/almarev/agentic/.env`, avoiding secret echo:

```bash
set -a; . /home/almarev/agentic/.env >/dev/null 2>&1; set +a
psql "$POSTGRES_URL" -X -A -F $'\t' -c "
SELECT COALESCE(NULLIF(instantly_status,''),'sem status') AS status, COUNT(*)
FROM leads GROUP BY 1 ORDER BY 2 DESC;

SELECT COALESCE(NULLIF(instantly_campaign,''),'sem campanha') AS campaign,
       instantly_status, COUNT(*)
FROM leads
WHERE instantly_status IS NOT NULL OR instantly_campaign IS NOT NULL
GROUP BY 1,2 ORDER BY 3 DESC;
"
```

## Interpretation pattern

- `leads.instantly_status='active'` is the number of leads enrolled/active in Instantly campaigns. It is not equal to leads already contacted.
- `contacted_count` is companies/accounts that received at least one email.
- `/campaigns/analytics/steps` gives touches per email step. Instantly step index may be zero-based in API output; existing reporter converts step 0 to email 1.
- `emails_sent_count` is total touches, not unique accounts.
- If email 1 and email 2 counts are equal, everyone who got email 1 has also got email 2. If email 3 is lower, the difference is waiting for email 3.
- Untouched but active/enrolled = `active status total - contacted_count`.

## Response shape Mario likes

Lead with exact numbers and source timestamp. Then give the operational reading in plain bullets:
- total active/enrolled
- touched/contacted
- per email step counts
- untouched but in campaign
- replies/bounces/unsubs
- per-campaign breakdown only if useful

Avoid explaining Instantly concepts unless needed. Mario asked for the answer, not a tutorial.
