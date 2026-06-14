# CLAIRE dialer operations

Use when Mario asks to pause/resume the dialer, weekend handling, CLAIRE outbound/reminder behavior, or whether voice calls are active.

## Components
- Outbound dialing timer: `alma-claire-outbound.timer`.
- Reminder/contact timer: `alma-claire-reminder.timer`.
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
pgrep -af 'claire_outbound|claire_reminder|Retell|Vapi|Twilio' || true
systemctl --failed --no-pager
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

## Verification standard
Before saying the dialer is paused/resumed, verify:
- target timers are `inactive/disabled` for pause or `active/enabled` for resume;
- no outbound-related process is running after pause;
- `systemctl --failed` is clean or failures are unrelated and named.
