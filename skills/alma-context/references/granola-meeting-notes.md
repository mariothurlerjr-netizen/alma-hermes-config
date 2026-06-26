---
created_by: hermes
created_at: 2026-06-26T00:00:00Z
source: telegram
---

# Granola meeting context

Granola is a meeting-notes ingestion source, not a general productivity surface.

## Verification pattern
1. `hermes mcp list` should show `granola` registered.
2. `hermes mcp test granola` should confirm authorization.
3. If the test returns an OAuth URL or otherwise stalls in a headless session, treat that as auth pending, not as a product failure.

## Operating rule
- After authorization, recaps and notes from meetings should be treated as recurring operational context.
- When they matter for ALMA memory, ingest them into the brain under `05-meeting-notes/granola/` as append-only notes.
- Keep Granola separate from Composio in reasoning and troubleshooting.
