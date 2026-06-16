# CLAIRE dialer operations

Use when Mario asks to pause/resume the dialer, weekend handling, CLAIRE outbound/reminder behavior, or whether voice calls are active.

## Components
- Outbound dialing timer: `alma-claire-outbound.timer`.
- Reminder/contact timer: `alma-claire-reminder.timer`.
- ALMA Local legacy/base dialer cron: `/home/almarev/alma-local/dialer/cron_trigger.sh` in root crontab, logs to `/var/log/alma-local-dialer.log`, writes calls to `local_call_log`.
- Non-dialing support timers that can stay active unless explicitly requested otherwise:
  - `alma-claire-calendly-poll.timer`
  - `alma-claire-qa.timer`
  - `alma-claire-coach.timer`

## Policy
- CLAIRE/dialer works Monday-Friday only unless Mario explicitly overrides.
- For weekend or temporary pause requests, stop only the dialing/contact timers by default. Keep polling, QA, and coach active because they do not initiate outbound calls.

## Pause dialer
```bash
systemctl disable --now alma-claire-outbound.timer alma-claire-reminder.timer
systemctl is-active alma-claire-outbound.timer alma-claire-reminder.timer || true
systemctl is-enabled alma-claire-outbound.timer alma-claire-reminder.timer || true
pgrep -af 'claire_outbound|claire_reminder|Retell|Vapi|Twilio|alma-local/dialer' || true
systemctl --failed --no-pager
```

If the question is "fizemos ligações hoje?" or any call audit, also check the ALMA Local legacy dialer path. The CLAIRE systemd timers can be disabled while `/home/almarev/alma-local/dialer/cron_trigger.sh` is still active from root crontab and placing Retell calls.

```bash
crontab -l | grep -n 'alma-local/dialer/cron_trigger' || true
tail -120 /var/log/alma-local-dialer.log 2>/dev/null
```

To pause the legacy/base dialer, comment the crontab line rather than killing Cortex/Retell webhooks:

```bash
tmp=$(mktemp)
crontab -l > "$tmp"
python3 - "$tmp" <<'PY'
import sys
p=sys.argv[1]
lines=open(p).read().splitlines()
out=[]
for line in lines:
    if '/home/almarev/alma-local/dialer/cron_trigger.sh' in line and not line.lstrip().startswith('#'):
        out.append('# ' + line + '  # paused by Hermes, cold/base dialing not approved')
    else:
        out.append(line)
open(p,'w').write('\n'.join(out)+'\n')
PY
crontab "$tmp"
rm -f "$tmp"
crontab -l | grep -n 'alma-local/dialer/cron_trigger' || true
pgrep -af '/home/almarev/alma-local/dialer/cron_trigger|dialer/orchestrator' || true
```

## Resume dialer
```bash
systemctl enable --now alma-claire-outbound.timer alma-claire-reminder.timer
systemctl status alma-claire-outbound.timer alma-claire-reminder.timer --no-pager
systemctl list-timers 'alma-claire-*' --no-pager
systemctl --failed --no-pager
```

## Scheduling a resume
If Mario says “voltamos segunda” or equivalent, create a durable cron job for Monday 09:00 BRT / 12:00 UTC that enables the two dialer timers and verifies active/enabled status. The prompt must be self-contained and include the exact timers.

## Call audit queries

When Mario asks whether calls happened today, do not rely only on Brain STATUS. Query the live DB in BRT bounds and separate AURA/CLAIRE attempts from ALMA Local calls:

```bash
set -a; [ -f /home/almarev/agentic/.env ] && . /home/almarev/agentic/.env; set +a
psql "$POSTGRES_URL" -X -A -F $'\t' <<'SQL'
with bounds as (
  select (date_trunc('day', now() at time zone 'America/Sao_Paulo') at time zone 'America/Sao_Paulo') as start_utc,
         ((date_trunc('day', now() at time zone 'America/Sao_Paulo') + interval '1 day') at time zone 'America/Sao_Paulo') as end_utc
)
select 'outbound_call_attempts' as source, coalesce(call_status,'(null)') as status, coalesce(campaign_id,'(null)') as campaign, count(*)
from outbound_call_attempts,bounds
where created_at >= start_utc and created_at < end_utc
group by 1,2,3
union all
select 'local_call_log' as source, coalesce(result,'(null)') as status, coalesce(outcome,'(null)') as campaign, count(*)
from local_call_log,bounds
where dispatched_at >= start_utc and dispatched_at < end_utc
group by 1,2,3
order by 1,4 desc;

with bounds as (
  select (date_trunc('day', now() at time zone 'America/Sao_Paulo') at time zone 'America/Sao_Paulo') as start_utc,
         ((date_trunc('day', now() at time zone 'America/Sao_Paulo') + interval '1 day') at time zone 'America/Sao_Paulo') as end_utc
)
select to_char(l.dispatched_at at time zone 'America/Sao_Paulo','YYYY-MM-DD HH24:MI:SS') as brt,
       left(l.retell_call_id,18) as call_id,
       coalesce(ll.business_name,'') as business,
       coalesce(ll.city,'') as city,
       coalesce(ll.state,'') as st,
       coalesce(l.result,'') as result,
       coalesce(l.outcome,'') as outcome,
       coalesce(l.window_label,'') as window
from local_call_log l
left join local_leads ll on ll.id=l.local_lead_id, bounds b
where l.dispatched_at >= b.start_utc and l.dispatched_at < b.end_utc
order by l.dispatched_at;
SQL
```

## Verification standard
Before saying the dialer is paused/resumed, verify:
- target timers are `inactive/disabled` for pause or `active/enabled` for resume;
- legacy ALMA Local crontab is commented if the request is to pause all/base/cold dialing;
- no outbound-related process is running after pause;
- `systemctl --failed` is clean or failures are unrelated and named.
