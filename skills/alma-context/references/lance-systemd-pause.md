# LANCE systemd pause/resume

Session source: 2026-05-24 Telegram, Mario asked to keep LANCE paused because it was not fully configured.

## Units observed

- `alma-lance.timer`: runs `alma-lance.service` every 5 minutes.
- `alma-lance.service`: one-shot LANCE cadence orchestrator.
- `alma-aura-hotpath.service`: related hot-path trigger, but do not stop it unless Mario specifically asks to pause AURA follow-up routing too.

## Pause workflow

1. Check live state:

```bash
systemctl list-units --all --type=service --no-pager | grep -Ei 'lance|outreach|sdr' || true
systemctl list-timers --all --no-pager | grep -Ei 'lance|outreach|sdr' || true
systemctl list-unit-files --no-pager | grep -Ei 'lance|outreach|sdr' || true
```

2. Normal path:

```bash
systemctl disable alma-lance.timer
systemctl stop alma-lance.timer
systemctl stop alma-lance.service || true
systemctl reset-failed alma-lance.service || true
```

3. If `systemctl disable --now` or stop hangs/timeouts, do not loop the same command. Use the deterministic fallback:

```bash
rm -f /etc/systemd/system/timers.target.wants/alma-lance.timer
busctl call org.freedesktop.systemd1 /org/freedesktop/systemd1 org.freedesktop.systemd1.Manager StopUnit ss alma-lance.timer replace
busctl call org.freedesktop.systemd1 /org/freedesktop/systemd1 org.freedesktop.systemd1.Manager StopUnit ss alma-lance.service replace || true
```

4. Verify:

```bash
test -e /etc/systemd/system/timers.target.wants/alma-lance.timer && echo symlink-present || echo symlink-removed
systemctl is-active alma-lance.timer 2>/dev/null || true
systemctl is-enabled alma-lance.timer 2>/dev/null || true
systemctl is-active alma-lance.service 2>/dev/null || true
systemctl list-timers --all --no-pager | grep -E 'alma-lance' || true
```

Expected paused state:

- symlink removed
- timer inactive
- timer disabled
- service inactive
- timer may still appear in `list-timers --all` with next run `-`, which is acceptable

## Resume workflow

```bash
systemctl enable alma-lance.timer
systemctl start alma-lance.timer
systemctl list-timers --all --no-pager | grep -E 'alma-lance'
```

## Pitfalls

- `systemctl disable --now alma-lance.timer` hung in this environment. If a systemctl command times out, switch to symlink removal plus `busctl StopUnit` instead of retrying.
- `STATUS.md` in the brain is auto-generated and read-only. Do not edit it to reflect pause state. Update operational memory, or write a Hermes note under allowed brain paths if needed.
