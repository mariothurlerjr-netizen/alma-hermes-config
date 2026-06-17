# ALMA Rev / ALMA Local CLAIRE call follow-ups

Use when Mario asks variants of:
- “quantas pessoas pediram pra mandar email nas calls?”
- “mandamos esses emails?”
- “quem pediu email/callback na CLAIRE?”
- “check sent”, “send report email”, “calls que viraram email”

## Source of truth

Live data is in Postgres under `/home/almarev/agentic/.env` `POSTGRES_URL`. Do not answer from memory.

Relevant tables:

- `local_call_log`: one row per CLAIRE/Retell local call. Important fields: `local_lead_id`, `phone`, `result`, `outcome`, `transcript_summary`, `transcript_full`, `dispatched_at`.
- `local_call_follow_ups`: structured post-call tasks. Important fields: `kind`, `status`, `email`, `detail`, `completed_at`, `result_note`.
- `local_leads`: business-level state. Important fields: `business_name`, `email`, `check_sent_at`, `email_status`, `callback_at`.

## Interpretation

- `lower(local_call_log.outcome)='check_sent'` means the call outcome was “send/check by email”. Count this for “pessoas pediram/aceitaram email nas calls”.
- `local_call_follow_ups.kind='send_report_email'` is the structured send queue. Count this for “tem email pronto pra enviar / foi enviado”.
- `local_call_follow_ups.status='done'` plus a Resend-like id in `result_note` means sent.
- `status='pending'` means still needs sending.
- Some `CHECK_SENT` calls can lack a structured follow-up because the email was only captured in transcript/summary. Extract cautiously, create follow-up only when the address is unambiguous.
- Do not treat `check_sent_at` alone as proof of email delivery. It can mark the call outcome/lead state. Use `local_call_follow_ups.status` and `result_note` for delivery evidence.

## Live audit SQL

```bash
set -a; . /home/almarev/agentic/.env >/dev/null 2>&1; set +a
psql "$POSTGRES_URL" -X -A -F $'\t' -c "
SELECT lower(outcome) AS outcome_norm, count(*) calls, count(DISTINCT local_lead_id) unique_leads
FROM local_call_log
WHERE lower(coalesce(outcome,''))='check_sent'
GROUP BY 1;

SELECT ll.business_name, ll.email AS lead_email, ll.check_sent_at,
       l.outcome, f.kind, f.status, f.email, f.completed_at, f.result_note
FROM local_call_log l
JOIN local_leads ll ON ll.id=l.local_lead_id
LEFT JOIN local_call_follow_ups f ON f.call_log_id=l.id AND f.kind='send_report_email'
WHERE lower(coalesce(l.outcome,''))='check_sent'
ORDER BY l.dispatched_at DESC;
"
```

## Send pending structured email follow-ups

Use the project venv because system Python may lack `resend`/`psycopg2`.

```bash
set -a; . /home/almarev/agentic/.env >/dev/null 2>&1; set +a
PYTHONPATH=/home/almarev/agentic /home/almarev/agentic/.venv/bin/python - <<'PY'
import os, psycopg2, psycopg2.extras
from cortex.emails.client import send_local_check_report

with psycopg2.connect(os.environ['POSTGRES_URL']) as conn:
    with conn.cursor(cursor_factory=psycopg2.extras.RealDictCursor) as cur:
        cur.execute("""
            SELECT f.id::text, f.email, f.lead_id::text AS lead_id,
                   l.business_name, l.first_name, l.segment
            FROM local_call_follow_ups f
            LEFT JOIN local_leads l ON l.id=f.lead_id
            WHERE f.kind='send_report_email'
              AND f.status IN ('pending','failed')
              AND f.email IS NOT NULL AND f.email <> ''
            ORDER BY f.created_at ASC
        """)
        for r in cur.fetchall():
            check_link = f"https://almarev.com/local/check?l={(r['lead_id'] or '')[:12]}&src=email"
            try:
                res = send_local_check_report(r['email'], r['first_name'] or 'there', r['segment'] or 'local service', check_link) or {}
                ok = bool(res.get('id')) and not res.get('error')
                cur.execute("""
                    UPDATE local_call_follow_ups
                       SET status=%s, completed_at=now(), result_note=%s
                     WHERE id=%s::uuid
                """, ('done' if ok else 'failed', str(res.get('id') or res)[:300], r['id']))
                if ok:
                    cur.execute("UPDATE local_leads SET check_sent_at=COALESCE(check_sent_at, now()) WHERE id=%s::uuid", (r['lead_id'],))
            except Exception as e:
                cur.execute("""
                    UPDATE local_call_follow_ups
                       SET status='failed', completed_at=now(), result_note=%s
                     WHERE id=%s::uuid
                """, (str(e)[:300], r['id']))
        conn.commit()
PY
```

## Pitfalls from 2026-06-16 session

- Mario’s shorthand “emails passados?” can mean “people from calls who asked us to send email”, not Instantly sequence emails. If prior context includes calls/CLAIRE, check `local_call_log` before answering Instantly funnel counts.
- The call transcript may spell emails phonetically, for example “customer service at access heating dot com”. Prefer exact `@` addresses from summary, lead email, or follow-up row. If only phonetic text exists, avoid sending unless unambiguous.
- If creating a missing `send_report_email` follow-up from a `CHECK_SENT` call, record `email_status='captured_from_call'` or an equivalent note and verify after sending with the audit SQL above.
