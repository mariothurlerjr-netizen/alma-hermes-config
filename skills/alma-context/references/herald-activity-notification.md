---
created_by: hermes
created_at: 2026-06-26T18:10:00Z
source: conversation-review
---

# HERALD activity notification pattern

## Problem observed
A conversation stream can go silent in the primary watch path if the agent only relies on one downstream notifier or one watchdog surface.

## Durable fix pattern
- Emit an immediate Telegram notification from the message ingress path when the message is non-spam.
- Add a second watcher on the conversation table or equivalent durable record as a backup signal.
- Restart the service that owns the ingress code after patching so the runtime loads the new notifier.

## Operational detail
In the reviewed session, the working pieces were:
- `receive_message` path updated to notify Telegram on non-spam activity.
- `alma-ops-sentinel` updated to monitor `herald_conversations` as a backup.
- `alma-cortex` restarted to pick up the ingress change.

## Verification idea
When the fix is live, a new inbound message should create:
1. A direct Telegram alert from the ingress path.
2. A backup signal from the sentinel if the primary path fails.

## Pitfall
Do not treat a quiet watch window as absence of activity until both the ingress notifier and the backup table watcher have been checked.