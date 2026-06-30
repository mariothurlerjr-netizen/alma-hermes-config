---
created_by: hermes
created_at: 2026-06-30T00:00:00Z
source: session-2026-06-30-scale-decision-template
---

# ALMA Rev scale decision template

Use this when Mario asks when to scale, hold, or pause an outbound campaign after a first-day or first-wave test.

## Canonical decision block
- input: current reply rate, positive reply rate, bounce rate, spam signal
- approximation: fit is real / warm but unproven / weak
- next action: scale 20% / keep / pause and revise

## Practical thresholds
- Scale when deliverability is clean, bounce is low, spam is absent, and positive replies are real, not just curiosity.
- Keep when replies exist but fit is not yet proven, or one vertical works while another underperforms.
- Pause when bounce spikes, spam appears, inbox placement drops, or replies stay useless after enough volume.

## Guardrails
- Increase volume by about 20% on the next batch after a win.
- Do not double volume in one step.
- Do not change offer and CTA at the same time.
- If one variable changes, change only one thing per test.

## Example
- input: reply rate 4%, positive reply rate 50%, bounce 0.4%, spam none
- approximation: fit is real, deliverability is stable
- next action: scale 20%
