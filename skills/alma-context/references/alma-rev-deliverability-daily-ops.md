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
