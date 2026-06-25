# ALMA agent/systemd health audit

Use when Mario asks whether the VPS agents are installed, rotating, alive, healthy, or asks to pause/stop all agents temporarily.

## Scope

This checks the live VPS/service layer, not just Brain status. Brain `STATUS.md` is a baseline, but systemd/journal/timers are the source for whether agents are actually rotating now.

## Fast audit sequence

1. Read Brain `STATUS.md` first for expected paused/active state.
2. Inspect systemd services and timers:
   ```bash
   systemctl list-units --all --type=service --type=timer 'alma-*' 'hermes-*' 'brain-*' 'cortex*' 'jt-*' --no-pager --plain
   systemctl list-unit-files 'alma-*' 'hermes-*' 'brain-*' 'cortex*' 'jt-*' --no-pager --plain
   systemctl --failed --no-pager --plain
   systemctl list-timers --all --no-pager --plain | egrep -i 'alma|orion|lance|claire|iris|sentinel|shield|granola|hermes|brain|cortex|jt'
   ```
3. Check Hermes gateway as root user service:
   ```bash
   XDG_RUNTIME_DIR=/run/user/0 systemctl --user status hermes-gateway --no-pager -l
   ```
4. Confirm core processes and ports:
   ```bash
   ps -eo pid,ppid,etimes,stat,cmd --sort=cmd | egrep -i 'alma|orion|lance|claire|iris|sentinel|shield|granola|cortex|hermes|brain|uvicorn|python'
   ss -ltnp | egrep ':8000|:8001|:8002|:8003|:80|:443'
   curl -fsS -m 5 http://127.0.0.1:8000/health
   curl -fsS -m 5 http://127.0.0.1:8002/health
   curl -fsS -m 5 http://127.0.0.1:8003/health
   ```
5. Check recent journal for rotating one-shots and watchdogs:
   ```bash
   journalctl -u 'alma-orion@*' -u alma-orion-watchdog.service --since '2 hours ago' --no-pager -n 120 -o short-iso
   journalctl -u alma-watchdog.service --since '30 minutes ago' --no-pager -n 80 -o short-iso
   ```
6. Check Hermes cron jobs only as supplemental state. ORION may be intentionally maintained by systemd while old Hermes cron `fc52bbc58989` remains paused.

## Pause-all agents runbook

When Mario says “pause all agents”, treat it as pausing proactive/agentic work, not core infra. Keep core serving layers alive unless explicitly told otherwise: `alma-aios`, `alma-cortex`, `alma-hub`, `alma-brain-mcp`, plus Hermes gateway for communication.

1. Snapshot before changing state:
   ```bash
   stamp=$(date -u +%Y%m%dT%H%M%SZ)
   out=/root/alma-agents-paused-$stamp.txt
   systemctl list-unit-files 'alma-*.timer' --no-pager --plain > "$out"
   systemctl list-units --all --type=service --type=timer 'alma-*' 'leadgen-*' --no-pager --plain >> "$out"
   crontab -l > /root/crontab-before-agent-pause-$stamp.txt 2>/dev/null || true
   ```
2. Disable and stop systemd agent timers:
   ```bash
   mapfile -t timers < <(systemctl list-unit-files 'alma-*.timer' --no-pager --plain | awk '$2=="enabled"{print $1}')
   ((${#timers[@]})) && systemctl disable --now "${timers[@]}"
   ```
3. Stop/disable long-running agent services that are not named `alma-*`. Durable pitfall: `leadgen-loop.service` respawns ALMA Rev leadgen and will survive an `alma-*` timer sweep.
   ```bash
   systemctl disable --now leadgen-loop.service alma-telegram-bot.service 2>/dev/null || true
   systemctl stop alma-inbox-sync.service 2>/dev/null || true
   systemctl reset-failed leadgen-loop.service alma-inbox-sync.service alma-telegram-bot.service 2>/dev/null || true
   ```
4. Pause Hermes cron jobs that run ALMA agent work (`cronjob action=list`, then `pause` every active agent job). Do not guess IDs.
5. Comment root crontab proactives with a dated marker instead of deleting them. Include `/root/run-heartbeat.sh`, ALMA brain state refresh/update/status/hygiene, and warmup monitor if the user asked for all agents paused.

   Exception, Instantly warmup/readiness: if Mario clarifies that account warmup must keep being observed, remember that Instantly itself does the warming. ALMA should only keep read-only readiness monitors alive, not outbound/leadgen agents. Re-enable or keep only:
   - `alma-warmup-snapshot.timer`, reads Instantly warmup into `warmup_history`.
   - `alma-deliverability.timer`, reports per-inbox warmup health and `ready-to-activate` status.
   - `alma-lance-inbox-health.timer`, classifies inbox health and may pause risky accounts.
   Keep outbound, leadgen, content, digest, heartbeat, ORION and LANCE work paused unless explicitly resumed.
6. Kill detached agent processes after stopping their services, then re-check for respawn:
   ```bash
   pkill -f '/home/almarev/agentic/.venv/bin/python -u -m agents\.leadgen\.overnight' 2>/dev/null || true
   pkill -f 'mirofish run --files uploads/seed-alma-local' 2>/dev/null || true
   pkill -f '/usr/bin/claude -p --output-format json --model haiku Classify this email reply' 2>/dev/null || true
   sleep 3
   ps -eo pid,ppid,etimes,stat,cmd --sort=cmd | egrep -i 'agents\.|orion|lance|claire|iris|sentinel|shield|granola|mirofish run|claude -p --output-format json --model haiku Classify this email reply' | grep -v grep || true
   ```
7. Write an operational note in `agents/hermes/` in the brain with what was paused and what core services were intentionally left alive.

Verification before saying “paused”:

```bash
systemctl list-unit-files 'alma-*.timer' --no-pager --plain | awk '$2=="enabled"{print $1}'
systemctl list-units --type=timer 'alma-*' --state=active --no-pager --plain
ps -eo pid,ppid,etimes,stat,cmd --sort=cmd | egrep -i 'agents\.|orion|lance|claire|iris|sentinel|shield|granola|mirofish run' | grep -v grep || true
systemctl list-units --all --type=service 'alma-aios.service' 'alma-cortex.service' 'alma-hub.service' 'alma-brain-mcp.service' --no-pager --plain
```

## Known durable pitfalls

- The VPS has no Unix user `almarev` unless explicitly created. Units with `User=almarev` fail with `status=217/USER`. Fix by creating the user intentionally or removing/changing the `User=` line, then `systemctl daemon-reload`, `reset-failed`, and rerun the service.
- `alma-watchdog.timer` can appear `active (elapsed)` with `Trigger: n/a` if monotonic settings are exhausted or overridden poorly. Prefer a calendar schedule for recurring watchdogs:
  ```ini
  [Timer]
  OnBootSec=
  OnUnitActiveSec=
  OnCalendar=*:0/5
  AccuracySec=30s
  Persistent=true
  ```
  Verify with `systemctl show alma-watchdog.timer --all | egrep -i 'Calendar|Next|SubState'` and wait through one tick before declaring fixed.
- One-shot services are normally `inactive (dead)` between runs. Judge them by timer next/last run, exit status, and recent journal, not by active state alone.

## Verification bar before reporting green

- `systemctl --failed` shows zero failed units, or any failed units are explicitly classified as unrelated/expected.
- All intended long-running services are `active`.
- Relevant timers show a future `NEXT`, not `-`/elapsed, unless intentionally one-shot.
- At least one recent tick of repaired timers/services is observed in journal.
- Health endpoints for `alma-aios` (`:8000`), `cortex` (`:8002`), and `brain-mcp` (`:8003`) return ok/alive.
