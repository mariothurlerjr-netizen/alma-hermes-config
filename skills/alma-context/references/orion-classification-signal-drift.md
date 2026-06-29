# ORION classification signal drift

Session note, 2026-06-29.

## Lesson
- An empty `by_sub_vertical` array is a classification failure signal upstream, not proof that leads are absent.
- Treat it as pipeline drift before scoring, then trace whether the value is produced at write time or lost at read/query time.

## Checks
1. Inspect the write path that emits sub-vertical metadata.
2. Inspect the scoring step that consumes the metadata.
3. Inspect the metrics/query layer that aggregates `by_sub_vertical`.
4. If the array is empty, do not conclude "no demand" until the chain above is verified.

## Operational implication
- Small controlled outbound batches should not be used as a workaround for a missing classification layer.
- Fix signal integrity first, then re-evaluate sender health and batch size.
