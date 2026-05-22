# ORION Apollo-free fallback

Session learning, 2026-05-22.

## Trigger

Mario challenged the first resolution because Apollo was externally blocked by credits but the business objective was to keep ORION producing leads:

```text
Alguma maneira de seguir sem o Apollo? Lembre que o objetivo é resolver e vc é o responsável por isso
```

Treat this as an operational correction: an external vendor quota is not a complete answer when there is an alternative discovery path.

## Correct response pattern

1. Preserve the Apollo diagnostic from `references/orion-apollo-422.md`: inspect the 422 body and detect `insufficient credits` explicitly.
2. Do not leave the pipeline stopped if another source can produce candidate firms.
3. Add a degraded candidate discovery fallback, then reuse existing enrichment/scoring/delivery plumbing.
4. Validate with output, not just service health.

## ORION fallback architecture

Observed working path:

```text
Apollo /mixed_companies/search
  -> 422 body says insufficient credits
  -> catch ApolloInsufficientCredits in ORION core
  -> tavily_candidates.list_candidates(...)
  -> try_claim_firm workspace dedupe
  -> Browserbase/LinkedIn + Snov/email enrichment where possible
  -> scoring_engine
  -> lead-base markdown + orion_leads DB row
```

Implementation points from the session:

- Add/maintain `agents/orion/adapters/tavily_candidates.py` as an Apollo-free `Firm` producer.
- Import it in `agents/orion/core.py` with the Apollo adapter.
- In `_pull_next_candidate`, call Apollo first, catch `ApolloInsufficientCredits`, then call Tavily fallback.
- Use deterministic sharding by `instance_id` so `alma-orion@1..5` start on different state/subvertical pairs.
- Create synthetic stable IDs as `tavily:<sha1(domain)[:12]>` so existing dedupe ledger works.
- Store provenance in `firm.raw_jsonb`, at minimum:
  - `source: tavily_search`
  - `primary_domain`
  - `description`
  - `search_title`
  - `search_url`
- In source collection, avoid double prefixing: keep `tavily:<id>` as-is, only prefix Apollo IDs with `apollo:`.
- If scoring relies on Apollo-only headcount/ARR, add conservative fallback context for Tavily candidates. In this session, a direct CPA-firm domain was treated as enough ICP evidence to keep delivery moving, while acknowledging quality degradation.

## Verification commands

Compile touched modules:

```bash
cd /home/almarev/agentic
/home/almarev/agentic/.venv/bin/python -m py_compile \
  agents/orion/adapters/apollo.py \
  agents/orion/adapters/tavily_candidates.py \
  agents/orion/core.py \
  agents/orion/scoring_engine.py \
  workers/orion_worker.py
```

Reload workers. If `systemctl restart` or `systemctl stop` blocks because the worker is sleeping, inspect MainPID and kill only that worker PID so systemd reloads code:

```bash
pid=$(systemctl show -p MainPID --value alma-orion@1.service)
kill -KILL "$pid"
sleep 12
systemctl is-active alma-orion@1.service
systemctl show -p MainPID -p ActiveState -p SubState alma-orion@1.service --no-pager
```

Start the expected cluster:

```bash
for i in 2 3 4 5; do
  systemctl reset-failed alma-orion@$i.service 2>/dev/null || true
  systemctl start alma-orion@$i.service
done
```

Health/output checks:

```bash
curl -sS http://127.0.0.1:8002/agents/orion/health | python3 -m json.tool

tail -120 /home/almarev/brain/agents/orion/logs/instance-1.jsonl

cd /home/almarev/agentic
/home/almarev/agentic/.venv/bin/python - <<'PY'
import os
from pathlib import Path
for line in Path('.env').read_text(errors='ignore').splitlines():
    if not line.strip() or line.lstrip().startswith('#') or '=' not in line:
        continue
    k, v = line.split('=', 1)
    os.environ.setdefault(k, v.strip().strip('"').strip("'"))
from agents.orion import db
conn = db._get_conn()
with conn.cursor() as cur:
    cur.execute("""
    SELECT orion_id, firm_name, firm_url, score_final, instance_id, created_at
      FROM orion_leads
     WHERE created_at >= now() - interval '10 minutes'
     ORDER BY created_at DESC LIMIT 10
    """)
    for row in cur.fetchall():
        print(row)
conn.close()
PY
```

Expected healthy degraded mode:

```text
apollo quota depleted
Apollo credits exhausted; using Tavily candidate fallback
tavily candidate fallback returned N firms
instance=<n> claimed firm=<name> source=tavily
delivered lead ... score=...
missing_instance_ids: []
```

## Tradeoffs to report

Be explicit but do not frame as failure:

- Quality is lower without Apollo because firm names, headcount, ARR and decision-maker identity are inferred from web search and enrichment.
- Some candidates need cleanup or stronger gating.
- The pipeline objective is still satisfied if workers are active and fresh leads are delivered to DB/brain.

## Pitfalls

- Do not stop at "Apollo has no credits" when the user asked for resolution.
- Do not return `[]` silently from Apollo on quota exhaustion; that hides the cause and may create retry loops.
- Do not verify only `systemctl active`; verify a delivered lead row or lead-base file.
- Do not echo `.env` or API keys. Parse only required vars and print `PRESENT/MISSING` if needed.
- Do not source `/home/almarev/agentic/.env` directly if it has shell syntax issues; use a small Python parser.
