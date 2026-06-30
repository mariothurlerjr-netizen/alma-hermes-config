# CLAIRE callback bridge, gating and rollout

Use this when enabling the Rev "interested" reply callback path.

## What the bridge does
- Lives in `agents/iris/callback_bridge.py`
- Is non-blocking, failures are swallowed so email reply flow keeps moving
- Only enqueues a call when both flags are on:
  - `REV_CALLBACK_ENABLED=1`
  - `CLAIRE_CALLS_ENABLED=1`
- Uses `RETELL_AGENT_ID_REV_CALLBACK` when present, otherwise falls back to `RETELL_AGENT_ID`

## Enable sequence
1. Create or choose the dedicated Retell callback agent in Retell first.
2. Set `RETELL_AGENT_ID_REV_CALLBACK` in `/home/almarev/agentic/.env`.
3. Flip `REV_CALLBACK_ENABLED=1` in the env file.
4. Restart `alma-cortex.service` so the worker picks up the env change.

## What Hermes can do directly
- Hermes can wire the local fallback path, edit `.env`, and restart the Cortex service.
- Hermes cannot create or duplicate the Retell agent from the local ALMA stack unless a dedicated Retell admin/API path is added later.
- So the handoff is: create agent in Retell, then let Hermes connect the id locally.

## Browser fallback requirements
- A plain public Retell homepage URL is not enough.
- To use the dashboard path, provide either a logged-in browser session or the exact dashboard page already open.
- The useful target is the Agents / Voice agents settings area, where the agent UUID or copyable id is visible.
- If the browser session is not authenticated, switch to the API-key path instead of wasting cycles on the public site.

## Verify
- Service comes back `active (running)`.
- Env contains the callback flag set and the callback agent id when dedicated routing is desired.
- If no callback agent id is set, confirm the main agent id is still a valid fallback.
- On a test reply, look for `outbound_call_attempts.campaign_id = 'interested-callback'` and a populated `provider_call_sid`.

## Notes
- This path is safe to leave in the codebase while disabled.
- If the callback is not supposed to fire yet, the correct state is flag off, not code removal.
