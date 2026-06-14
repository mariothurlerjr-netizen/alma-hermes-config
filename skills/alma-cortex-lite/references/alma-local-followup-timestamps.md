# ALMA Local follow-up timestamps

Use this when Mario asks when a CLAIRE/ALMA Local follow-up, report email, callback, or `check_sent` happened.

## Durable lesson

A follow-up row has two relevant timestamps:

- `local_call_follow_ups.created_at`: when the follow-up/task was created, often by webhook or later intent backfill.
- `local_call_log.dispatched_at`: when the actual call occurred.

If Mario asks "quando foi isso?" after a heartbeat about a pending follow-up, report both timestamps unless the wording clearly asks for only one. If he asks "quando ocorreu a call?", answer only `local_call_log.dispatched_at` and convert UTC to Brazil time.

## Query pattern

```sql
SELECT f.id::text,
       f.kind,
       f.status,
       f.email,
       f.detail,
       f.created_at AT TIME ZONE 'UTC' AS followup_created_utc,
       f.due_at AT TIME ZONE 'UTC' AS followup_due_utc,
       l.business_name,
       l.first_name,
       l.phone,
       l.city,
       cl.dispatched_at AT TIME ZONE 'UTC' AS call_utc,
       cl.outcome,
       cl.result,
       cl.transcript_summary
  FROM local_call_follow_ups f
  LEFT JOIN local_leads l ON f.lead_id = l.id
  LEFT JOIN local_call_log cl ON f.call_log_id = cl.id
 WHERE f.workspace_id = 'dddddddd-dddd-dddd-dddd-dddddddddddd'
   AND (
        f.email ILIKE '%<email-or-domain>%'
        OR l.business_name ILIKE '%<business>%'
        OR f.detail::text ILIKE '%<name-or-keyword>%'
   )
 ORDER BY f.created_at DESC
 LIMIT 10;
```

## Output format

Prefer concise PT-BR:

- `A call ocorreu sexta, 12/jun/2026, 11:19:36 no Brasil. Em UTC: 2026-06-12 14:19:36.`
- If ambiguity matters: `O follow-up foi criado X, mas a call original ocorreu Y.`

Do not over-explain the funnel unless Mario asks.