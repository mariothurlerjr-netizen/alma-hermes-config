# LUNA content digest operations

Use when Mario asks what an ALMA Rev weekly email/newsletter is, whether it was sent externally, who received it, or why a blog digest arrived in his inbox.

## What it is

- Unit: `alma-content-digest.timer` / `alma-content-digest.service`
- Schedule: Thursdays at `13:00 UTC` (`10:00 BRT`)
- Script: `/home/almarev/agentic/agents/luna/content_digest.py`
- Module run: `cd /home/almarev/agentic && .venv/bin/python -m agents.luna.content_digest`
- Sender: `Mario Thurler Jr <mario@almarev.com>` via Resend
- Subject currently set in code: `What's new from alma rev this week`
- Purpose: weekly warm content digest for AURA completers, not Instantly cold outbound.

## Live checks

Check the timer and latest run:

```bash
systemctl status alma-content-digest.timer alma-content-digest.service --no-pager
journalctl -u alma-content-digest.service -n 80 --no-pager
```

Check recipients and posts for the current/last run without exposing secrets:

```bash
cd /home/almarev/agentic
set -a; . ./.env 2>/dev/null; set +a
.venv/bin/python - <<'PY'
import os, psycopg2
from psycopg2.extras import RealDictCursor
conn = psycopg2.connect(os.getenv('POSTGRES_URL'))
cur = conn.cursor(cursor_factory=RealDictCursor)
queries = [
    "select * from content_digest_log order by sent_at desc limit 20",
    "select slug,title,published_at from blog_posts where status='published' and published_at > now() - interval '7 days' order by published_at desc",
    "select lower(email) email, count(*) sessions, max(completed_at) last_completed from aura_sessions where completed_at is not null and email is not null and email <> '' group by 1 order by last_completed desc",
]
for q in queries:
    print('---', q)
    cur.execute(q)
    for r in cur.fetchall():
        print(dict(r))
PY
```

## Pitfalls

- `content_digest_log` is the source of truth for who was marked sent for a given ISO week. Do not infer only from inbox copy.
- The audience query is `aura_sessions.completed_at IS NOT NULL`, so test AURA sessions can enter the warm audience unless filtered.
- Internal/test addresses may look real if they are `@almarev.com`. Filter patterns should exclude `smoke`, `prewarm`, `test+`, `+test`, `e2e+`, and `claire+sync` unless Mario explicitly wants internal QA recipients.
- Distinguish this from Instantly outbound. This digest uses Resend and LUNA content, not campaign sequences.
- When Mario asks “foi pra outras pessoas?”, answer from the DB recipient list and separate real external humans from internal/test addresses.

## Safe fixes

- For recipient bugs, patch `TEST_RE` in `agents/luna/content_digest.py`, then run:

```bash
cd /home/almarev/agentic
.venv/bin/python -m py_compile agents/luna/content_digest.py
```

- Do not delete `content_digest_log` to “unsend” or retry unless Mario explicitly asks. The primary remediation is audience filtering for future runs.
