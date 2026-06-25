# Instantly capacity maximization, ALMA Rev

Use when Mario asks about Instantly plan, lead cap, daily send volume, cadence length, campaign utilization, or whether ALMA outbound is being maximized.

## Core principle
Mario expects proactive maximization. If available outbound capacity is underused, flag it explicitly and propose a concrete capacity strategy. Do not only report current metrics.

## Required checks
1. Count all leads in the Instantly workspace, not only the active campaign. The plan cap is workspace-wide.
2. Split by campaign:
   - active Launch campaigns,
   - ALMA Local or other active campaigns,
   - paused/old tests such as Clay Copy Test.
3. Count active mailboxes via `/accounts`, not campaign lead counts.
4. Sum active mailbox `daily_limit` to estimate sender-side daily capacity.
5. Read campaign `sequences` to get true cadence step count. Do not infer cadence length from analytics steps already sent, because later steps may not have become due yet.
6. Sum campaign `daily_limit` separately from mailbox capacity. Campaign cap can bottleneck below mailbox cap.
7. Inspect paused campaigns before recommending reactivation: sequence version, daily limits, already-contacted count, replies, bounces, and whether they are copy tests.

## Formulas
- Sustainable new leads/day = `daily_send_capacity / cadence_steps`.
- Monthly sends = `daily_send_capacity * business_days`.
- Monthly new leads needed = `monthly_sends / cadence_steps`.
- Batch duration at full capacity = `(batch_leads * cadence_steps) / daily_send_capacity`.

Example with verified 2026-06 session numbers:
- 30 active accounts, 891 emails/day.
- Launch cadence has 5 steps.
- Sustainable intake = `891 / 5 = ~178 new leads/business day`.
- 22 business-day month = `19,602 sends`, requiring `~3,920 new leads/month`.
- 1,000 loaded leads at 5 steps = 5,000 sends, enough for `~5.6 business days` at 891/day.

## Interpretation pitfalls
- “Only 330 leads in Launch” can coexist with a full 1k plan because the plan counts all workspace leads, including ALMA Local and paused copy-test campaigns.
- Analytics showing only steps 1-3 sent does not mean the campaign has 3 steps. Fetch campaign `sequences`.
- Paused copy-test leads are not automatically invalid. Decide between preserving them as an experiment, reactivating with real daily limits, or exporting/deduping/migrating non-touched leads into the current Launch cadence.
- If a paused campaign has tiny daily limits, reactivating it will not materially increase throughput unless its limits are raised.

## Default maximization recommendation pattern
For 30 accounts around 900/day and a 5-step cadence:
- Upgrade Instantly lead cap to at least 10k if plan is near 1k.
- Keep Launch inventory fed with roughly 180 clean new leads/day.
- Load initial buffer of 2k-3k clean leads after cap upgrade.
- Raise campaign caps only after confirming mailbox capacity and lead inventory.
- Throttle only when commercial or deliverability signals justify it: too many replies/meetings to handle, bounce/spam degradation, domain reputation decline, or inbox placement issues.

For 60 accounts around 1,800/day and 5 steps:
- Sustainable intake is roughly 360 new leads/day.
- 10k lead cap becomes minimum; 25k is the more comfortable operating cap.

## Conservative domain-only relaunch
When outbound is at zero and Mario asks to turn on only a specific safe domain, optimize for clean re-entry, not max capacity:
- Restrict `email_list` to that domain before activation.
- Set a deliberately low campaign cap first, for example 10/day per campaign or roughly 80/day across 8 Launch/Clay campaigns, even if sender-side capacity is higher.
- Verify no other active ALMA-ish campaign is using non-approved domains.
- Increase volume only after bounce/complaint/inbox-placement signals are acceptable.

This is a temporary deliverability-protection mode. Once signals are clean, move back to the normal capacity math.

## Response style for Mario
Start with the literal answer in 1-3 lines, then show the capacity math. If capacity is under 70% utilized without an explicit reason, call it out as a red flag and give the exact bottleneck and next action.