# Instantly readiness projection

Use this note when Mario asks how many accounts will be READY on a future date like amanhã, quarta, ou próxima semana.

## Rule
- READY is not a generic warmup label.
- READY only counts after the account has completed the configured clean window with no spam signal.
- In the current ALMA outbound logic, the conservative gate is the full clean streak, then warmup health, then minimum warmup weeks, then placement.

## Projection method
1. Revalidate the live account inventory first.
2. Read the current `clean_streak_days` for each account.
3. Apply the date offset explicitly, day by day.
4. Count only accounts that satisfy the full gate on that target date.
5. Keep paused or spam-flagged accounts out of the READY bucket until they clear the signal and finish the gate.

## Pitfall
- Do not project from a summary like “31 warming, 2 paused_spam” without checking the current date and live state.
- Do not infer a future READY count from screenshots alone.
- Do not confuse “will have another clean day” with “already qualified as READY”.

## Example
If the live state today is:
- 31 warming
- 2 paused_spam
- 0 ready

Then the future READY count stays 0 until accounts actually finish the full gate and exit the paused state.
