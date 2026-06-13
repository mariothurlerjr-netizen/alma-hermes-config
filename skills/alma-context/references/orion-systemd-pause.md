# ORION pause/resume runbook

Use when Mario asks to stop, pause, freeze, or resume lead generation.

## Pause ORION lead generation

1. Pause the Hermes cron watchdog first, otherwise it may repair/restart workers if re-enabled:
   - cron job name: `orion-watchdog`
   - known job id: `fc52bbc58989`
   - action: `cronjob(action="pause", job_id="fc52bbc58989")`
2. Stop and disable the systemd watchdog timer and all ORION workers:
   - `systemctl disable --now alma-orion-watchdog.timer alma-orion@1.service alma-orion@2.service alma-orion@3.service alma-orion@4.service alma-orion@5.service`
3. Clean noisy failed state if systemd shows stopped workers as failed after disable:
   - `systemctl reset-failed alma-orion-watchdog.service alma-orion@1.service alma-orion@2.service alma-orion@3.service alma-orion@4.service alma-orion@5.service`
4. Verify all layers:
   - Hermes cron: watchdog `enabled=false` / `state=paused`
   - systemd watchdog timer: `inactive` and `disabled`
   - systemd worker active state: `inactive` for `alma-orion@1..5.service`
   - systemd worker enabled state: `disabled` for `alma-orion@1..5.service`
   - process table: no `alma-orion`, `agents/orion`, `orion_worker`, or lead-gen process remains
5. Update durable state/memory: ORION is intentionally paused, watchdogs paused, workers stopped/disabled.

## Resume ORION lead generation

1. Enable and start workers:
   - `systemctl enable --now alma-orion@1.service alma-orion@2.service alma-orion@3.service alma-orion@4.service alma-orion@5.service`
2. Resume watchdog:
   - `cronjob(action="resume", job_id="fc52bbc58989")`
3. Verify workers are active and watchdog is scheduled.

## Pitfalls

- Stopping workers without pausing `orion-watchdog` is incomplete. The watchdog exists to repair/restart ORION.
- Do not run `/root/.hermes/scripts/orion_watchdog.py` as a "check" while ORION is intentionally paused. It is not dry-run safe and will re-enable/start `alma-orion@1..5.service`; verify pause via `cronjob(action="list")`, `systemctl is-active/is-enabled`, and process table instead.
- `disable --now` can leave units in `failed`; run `reset-failed` after confirming they are stopped so status output is clean.
- Do not treat the systemd service absence/inactivity alone as sufficient. Verify watchdog, enabled state, active state, and process table.
