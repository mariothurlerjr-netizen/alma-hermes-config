# ALMA Local CRM outcomes

Use when Mario asks what a call tag/stage means, especially CLAIRE/Local outcomes from Retell, call review, coach, or local CRM.

## `CHECK_SENT`
Definition: the prospect agreed to receive the free check, snapshot, diagnostic, report, or link. It is not enough that CLAIRE offered it. There must be acceptance from the prospect.

Operational meaning:
- CRM stage becomes `check_sent`.
- Funnel order: `new → calling → connected → check_sent → check_done → booked`.
- It is non-terminal: not a booked meeting yet.
- It is treated as a strong positive signal/hard win by the coach, alongside booked, because the prospect accepted the next asset.
- Retell tag `CHECK_SENT` maps to outcome/stage `check_sent` and triggers a Telegram notification with a ready-to-copy SMS follow-up.

Follow-up implication:
- Next goal is converting `check_sent` to `check_done`, then `booked`.
- If transcript shows only “send info” from a gatekeeper without true prospect acceptance, treat as possible false positive and inspect transcript before counting as a strong win.

Known source locations in `/home/almarev/agentic`:
- `scripts/classify_outcomes.py`: classifier definition and regexes.
- `cortex/local_crm.py`: local funnel stages.
- `cortex/retell.py`: Retell tag mapping, DB update, Telegram/SMS side effects.
- `cortex/claire_coach.py`: winner/hard-win treatment.
- `cortex/local_calls.py`: statistics.
