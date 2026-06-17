# ALMA Local Check submit, `local_leads.phone` NOT NULL

## Trigger

Use when `/api/local/check/submit` or Sentinel reports:

```text
check_submit falhou: null value in column "phone" of relation "local_leads" violates not-null constraint
```

Typical context: `/local/check` payload contains `name`, `email`, `segment`, `priority`, but `phone` is empty/null. This can come from stale browser retry queue/localStorage, direct probes, or an older frontend submit shape.

## Root cause pattern

`local_check.py` has two persistence targets with different semantics:

- `local_check_submissions`: capture table, may store partial submissions (`phone` nullable, `local_lead_id` nullable).
- `local_leads`: call-ready table, `phone` is `NOT NULL` because dialer/CLAIRE expects dialable leads.

If backend creates `local_leads` unconditionally after failing to match token/phone, a phone-less submission explodes before the capture row is saved.

## Correct fix

Do not relax the database constraint unless the product decision changes. Keep `local_leads` dialable-only.

Patch pattern in `/home/almarev/agentic/cortex/local_check.py`:

```python
# Create a dialable lead only when we have a valid phone.
if not lead_id and phone:
    cur.execute("""INSERT INTO local_leads
         (workspace_id, business_name, first_name, phone, segment, source, priority)
       VALUES (%s, %s, %s, %s, %s, 'check_inbound', %s)
       RETURNING id""", (...))
```

Then still insert `local_check_submissions` with `local_lead_id = None` and `phone = None`.

## Verification

1. Compile: `.venv/bin/python -m py_compile cortex/local_check.py`.
2. Restart: `systemctl restart alma-cortex.service && systemctl is-active alma-cortex.service`.
3. Smoke no-phone payload against localhost:
   - endpoint: `http://127.0.0.1:8002/api/local/check/submit`
   - expect HTTP 200 with `{"ok": true, "lead_id": null, "suppressed": false}`.
4. DB check:
   - `local_check_submissions` has the no-phone row with `local_lead_id IS NULL`.
   - no `local_leads` row with `phone IS NULL` was created.
   - delete the smoke submission after checking.
5. Public smoke through nginx/proxy:
   - `https://almarev.com/local/check/submit` with watchdog payload should return 200.
6. Tail logs after restart and ensure no new `NotNullViolation`.

## Pitfall

If the frontend currently requires phone, do not assume every submit has phone. `flushQueue()` can replay older payloads from `localStorage`, and bots/probes can call the endpoint directly. Backend must preserve invariant independently of frontend validation.
