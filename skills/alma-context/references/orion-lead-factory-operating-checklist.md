---
created_by: hermes
created_at: 2026-06-29T00:00:00Z
source: session-review
---

# ORION lead factory operating checklist

Use this when the conversation is about ORION outbound pace, classification, or send-readiness.

## Operating sequence
1. Freeze any blast scale-up while input hygiene is unstable.
2. Gate entry before enrichment, dedupe, and classify first.
3. Treat `sub_vertical` as required, if it is empty, quarantine the lead.
4. Validate email deliverability before loading into cadence.
5. Keep sender health as the hard limiter, not raw lead volume.
6. Send in small batches and monitor bounce, reply, and spam signals.

## Core metrics
- homologated leads percentage
- leads with `sub_vertical` filled
- ready accounts vs resting accounts
- bounces per 100 sent
- days to cap

## Practical rule
If the next step is obviously safe and low risk, continue the plan instead of stopping for extra explanation.
