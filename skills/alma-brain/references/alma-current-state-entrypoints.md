# ALMA current-state entrypoints

Use these as the first canonic read when the conversation spans more than one ALMA line of business.

## Canonical `_CURRENT.md` files

- `/home/almarev/brain/03-alma-agentic/_CURRENT.md`
  - Technical backstage for ALMA Agentic.
  - Use for Hub, Cortex, agent fleet, systemd, webhooks, technical P0s, and state/delivery drift.

- `/home/almarev/brain/02-alma-rev/_CURRENT.md`
  - Canonical live state for ALMA Rev.
  - Use for positioning, market, pricing, ICP, funnel, cadence, outbound, and site/copy state.

- `/home/almarev/brain/13-alma-local/_CURRENT.md`
  - Canonical live state for ALMA Local.
  - Use for local-service ICP, recovered revenue, call flow, Retell/CLAIRE, follow-ups, and offer status.

## Reading rule

When the user asks for status, where we left off, or a cross-domain comparison:

1. Read the relevant `_CURRENT.md` files first.
2. Treat each file as the live canonical state for its domain.
3. Keep the domains separated in the final answer unless the user explicitly asks for a merged synthesis.
4. If the files conflict, state the conflict and prefer the newest canonical file or the live system state.
5. Use the file-specific canonical source list inside each `_CURRENT.md` before falling back to older notes.
