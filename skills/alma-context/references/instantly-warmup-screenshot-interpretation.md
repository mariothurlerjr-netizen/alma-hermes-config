# Instantly warmup screenshot interpretation

Use when Mario shares warmup screenshots or asks whether colored bars mean spam.

## Visual read
- Orange bars in the warmup chart indicate spam events, not generic activity.
- The summary line `saved from spam` is a separate metric and can be lower than the number of orange bars visible in the chart.
- A screenshot can show recent spam even when the inbox state table has a clean streak for some accounts.

## Operational rule
- Treat the screenshot as a signal, not the source of truth.
- Reconcile it against the live database / state table before answering counts or readiness.
- If OCR is needed, enlarge and sharpen the image first, then use tesseract or another OCR path to extract account names and weekly totals.

## Pitfall
- Do not collapse screenshot evidence into the live inventory without checking both.
- Do not interpret a `4+ days clean` intuition unless the user explicitly asked for streak-based readiness; a bounded 4-day zero-spam window is a different criterion.
