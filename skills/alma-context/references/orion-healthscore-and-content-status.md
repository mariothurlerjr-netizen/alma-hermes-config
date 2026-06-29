# ORION healthscore + content status split

Use this as the default framing when Mario asks whether ORION is "working", "healthy", or "ready to scale".

## ORION healthscore buckets

1. **Stack health**
   - Workers up/down
   - Watchdog alive
   - Queue moving
   - Dedupe / verifier running

2. **Mailbox/account health**
   - Instantly inboxes eligible only after the warmup gate
   - Treat accounts as **not ready** if spam is still being seen, even when the score looks high
   - A mailbox around **93%** but still producing **~1 spam/day** stays out of the sending pool
   - Practical gate: **5 days with zero spam** before promoting to active sending

3. **Throughput**
   - Leads discovered/day
   - Leads verified/day
   - Leads delivered/day
   - Leads discarded/day

4. **Forecast**
   - Daily capacity
   - Weekly capacity
   - Monthly capacity

## Default reporting shape

When answering, separate:
- **Current state**
- **Pending items**
- **Blockers**
- **Next move**

## Content ops split

When Mario asks whether content/marketing is "implemented" or "working", separate into:
- **Already implemented**
- **Still on paper**
- **Recommended next move**

For ALMA Rev/market content, do not collapse strategy, calendar, and execution into one bucket.
