---
created_by: hermes
created_at: 2026-06-26T00:00:00Z
source: telegram
---

# ALMA Rev deliverability + daily outbound ops

## What changed in this session

- Do not use ICP as an exclusion filter at this stage.
- Treat all leads as usable inputs for copy testing, deliverability testing, and sequence learning.
- The immediate goal is volume with inbox health, not closing.
- Current north star is 900 emails/day, but only as long as warm accounts stay clean.

## Daily deliverability check

1. Count total sending accounts.
2. Count accounts ready to send today.
3. Separate the 3 fully warmed accounts from the rest still in warmup.
4. Check whether any of the 3 warmed accounts started landing in spam.
5. If spam appears, pause or reduce volume for a few days before resuming.
6. Verify the warmup pipeline in the dosso system and whether the remaining accounts/domains are still scheduled to unlock next Tuesday.

## Mailbox reconnection provider selection

When reactivating `@getalmarev.com` accounts in Instantly, do not assume Google/Gmail just because the domain resembles Workspace mail. If Mario specifies Maildoso/M365, reconnect those inboxes using the Maildoso / Microsoft 365 path in Instantly, not Google OAuth.

Operational sequence:
1. Identify the exact affected accounts first.
2. Confirm current provider/auth type in Instantly before changing anything.
3. Reconnect with the intended provider class (`Maildoso/M365` when specified), then verify SMTP/IMAP or Microsoft auth health.
4. Keep daily limit conservative during re-entry, starting at 5/day for newly reconnected or recently risky inboxes unless a newer live readiness rule overrides it.
5. Recheck SPF/DKIM/DMARC, but do not treat DNS being correct as proof that the inbox provider connection is correct.

## Daily operating rule

- Use the warmed accounts as the active base.
- Keep outbound running, but do not sacrifice deliverability for raw volume.
- If the inbox health drops, treat that as a signal to slow down and recover.
- Use replies, OOO, and list mistakes as signal, not failure.

## CRM / RevOps minimum

- Every opportunity needs an owner.
- Every opportunity needs a next step.
- Mario and Hermes review hot vs stalled opportunities in the Hub CRM.
- The goal is not a perfect forecast yet, only no orphaned opportunities.

## Messaging / CTA

- Primary CTA today is the 5-minute free diagnosis.
- Email copy should move the lead toward that diagnosis, not toward a hard close.
- Claire can assist on follow-up and calls, but email remains the main motion.

## Quick status labels

- `ready_now`: fully warmed and safe to send.
- `warming`: still in warmup, do not force volume.
- `recovering`: previously spammed, needs reduced volume and observation.
