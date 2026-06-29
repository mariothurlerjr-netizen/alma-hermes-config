# Live inventory revalidation

Context: Outbound mailbox counts and sender pool composition can change during the day.

Lesson:
- Never reuse an earlier snapshot count when answering how many accounts are READY / WATCH / HOLD.
- Revalidate the live inventory immediately before stating totals.
- If the user reports that some accounts changed status, treat the prior count as stale and restate the baseline after recheck.
- When reporting counts, mention the classification basis used for that specific recheck.

Useful phrasing:
- "Snapshot anterior ficou velho, vou revalidar live antes de te dar o total."
- "O número certo agora é o do inventário atual, não o do início da conversa."