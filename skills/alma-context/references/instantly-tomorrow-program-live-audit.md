# Instantly tomorrow-program live audit

When the user asks how a campaign is programmed for tomorrow, answer from the live sender/campaign state, not from a cached snapshot alone.

## Check order
1. Active campaigns, campaign schedule, timezone, and daily limit.
2. Mailbox pool health, especially warmup status and clean streak.
3. Per-account sent counts for today, grouped by sender address.
4. Exclusions, paused mailboxes, and any accounts that moved into spam.

## Pitfall
- Old test campaigns can still appear in the sequencer after a vertical rollout. If the intended plan differs from the live campaign list, call out the drift explicitly and separate:
  - intended tomorrow program
  - live active sequencer
  - paused or excluded senders

## Useful output shape
- campaign name
- timezone and send window
- daily limit
- ready pool
- excluded pool
- per-account distribution
- drift versus intended plan