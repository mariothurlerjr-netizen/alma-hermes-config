# ORION Apollo 422 diagnostic

Session learning, 2026-05-22.

## Symptom

`alma-orion@1.service` was active and heartbeating, but every scan logged only:

```text
422 Client Error: Unprocessable Entity for url: https://api.apollo.io/api/v1/mixed_companies/search
```

The initial patch treated this as pagination/cursor rejection. That was incomplete because the log did not include the Apollo response body.

## Correct diagnostic sequence

1. Inspect service + logs:

```bash
systemctl status 'alma-orion@1.service' --no-pager -l || true
journalctl -u 'alma-orion@1.service' --since '2 hours ago' --no-pager -n 120 || true
```

2. Reproduce the API call with the real env, but do not echo secrets:

```bash
cd /home/almarev/agentic
/home/almarev/agentic/.venv/bin/python - <<'PY'
import os, requests
for line in open('.env', encoding='utf-8'):
    line=line.strip()
    if not line or line.startswith('#') or '=' not in line:
        continue
    k,v=line.split('=',1)
    if k=='APOLLO_API_KEY':
        os.environ[k]=v.strip().strip('"').strip("'")
key=os.environ['APOLLO_API_KEY']
headers={'Cache-Control':'no-cache','Content-Type':'application/json','x-api-key':key}
payload={
  'organization_industries':['accounting'],
  'organization_num_employees_ranges':['1,10','11,20','21,50'],
  'organization_locations':['United States'],
  'page':1,
  'per_page':25,
}
r=requests.post('https://api.apollo.io/api/v1/mixed_companies/search',headers=headers,json=payload,timeout=30)
print('STATUS', r.status_code)
print(r.text[:1000].replace(key, 'REDACTED'))
PY
```

3. Interpret Apollo `422` by body, not status alone. In this session the body was:

```text
You have insufficient credits! Upgrade your plan to increase your number of lead credits.
```

## Fix pattern

- Add a typed exception like `ApolloInsufficientCredits` in the adapter.
- In `_search`, if `r.status_code == 422`, inspect `r.text`; raise the typed exception when body contains `insufficient credits`; otherwise log the 422 body and call `raise_for_status()`.
- Let the worker catch the typed exception and exit with the service's `RestartPreventExitStatus` (`42` in ORION), marking heartbeat `quota_exhausted`.
- Do not keep returning `[]`, because the worker treats empty pull as retryable and will loop forever.

## Verification

```bash
cd /home/almarev/agentic
/home/almarev/agentic/.venv/bin/python -m py_compile agents/orion/adapters/apollo.py workers/orion_worker.py
systemctl reset-failed alma-orion@1.service
systemctl start alma-orion@1.service
systemctl show -p Result -p ExecMainStatus alma-orion@1.service --no-pager
tail -120 /home/almarev/brain/agents/orion/logs/instance-1.jsonl
```

Expected while Apollo has no credits:

```text
Result=exit-code
ExecMainStatus=42
apollo quota depleted
Apollo credits exhausted; stopping worker with RestartPreventExitStatus=42
```
