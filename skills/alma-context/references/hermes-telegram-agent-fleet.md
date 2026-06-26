# Hermes Telegram agent fleet migration

Use when migrating ALMA agents from Claude/LangGraph/legacy workflows into Hermes profiles with one Telegram bot per agent.

## Durable pattern

1. Treat Hermes profile identity and Telegram bot identity as separate layers:
   - Hermes profile: `~/.hermes/profiles/<profile>/`, `SOUL.md`, `CLAUDE.md`, `.env`, config, alias.
   - Telegram bot: BotFather token + immutable-ish `@username` + display name/description/commands.
2. Do not assume a renamed display name changed the bot `@username`. Normal BotFather flow changes display name only. For a clean new `@`, create a new bot with `/newbot` and replace the profile token.
3. One Telegram bot token should map to one active Hermes gateway/profile. If two profiles share one token, start only one of them. Running two pollers against the same token causes identity/polling conflict.
4. Store tokens only in each profile `.env` as `TELEGRAM_BOT_TOKEN`; configs should reference `${TELEGRAM_BOT_TOKEN}` and `${TELEGRAM_ALLOWED_USERS}`.
5. Keep unrelated OAuth MCP servers out of gateway profile configs unless required. Gateway startup should not open headless OAuth prompts for Supabase/Apollo just because an agent bot starts.
6. Run each bot gateway as an isolated user systemd instance, e.g. `hermes-gateway@orion.service`, with `ExecStart=... hermes_cli.main --profile %i gateway run --replace`.
7. Validate with Telegram `getMe`, profile list, process list, and failed units. Report usernames and running states, never token values.

## Inter-bot Telegram coordination

Use when Mario asks to let ALMA bots talk to each other.

- Telegram bot-to-bot DMs are not a usable control plane. Put the bots in the same approved group/forum/topic and route by explicit `@botusername` mentions.
- One bot should only answer when directly mentioned or replied to. Keep `telegram.require_mention: true` / exclusive mention routing enabled so a group message does not wake the whole fleet.
- Human authorization and bot authorization are separate. Keep Mario in `TELEGRAM_ALLOWED_USERS`; add ALMA bot IDs there only when needed for inter-bot traffic, and set `TELEGRAM_ALLOW_BOTS=mentions` rather than opening `TELEGRAM_ALLOW_ALL_USERS`.
- Hermes Telegram must propagate Telegram's `from_user.is_bot` into `SessionSource.is_bot`; otherwise the central gateway authorizer cannot distinguish ALMA bot senders from unauthorized humans. If inter-bot messages are dropped, inspect `gateway/platforms/telegram.py::_build_message_event` and `gateway/run.py::_is_user_authorized` before blaming tokens or group permissions.
- Smoke test in the real group/topic with two bots and an explicit mention chain. A passing `getMe` only proves the token identity, not inter-bot delivery.

## Suggested ALMA profile roles

- `jarvis`: main executive entry and router.
- `orchestrator`: fleet coordination, board, priority.
- `orion`: lead factory, sourcing/enrichment, CSV, no outbound sending.
- `lance`: outbound queue, Instantly, compliance, sender health.
- `sentinel`: external signals, market, prospects, content intel.
- `shield`: system health, logs, memory, gateway failures.
- `iris`: metrics, dashboard, analytics, digests, anomalies.
- `muse`: content, creative, hooks, drafts.
- `aura`: workflows, diagnostics, integrations.
- `claire`: voice SDR, Twilio/ElevenLabs/Vapi, paused unless reopened.
- `clara`: interview agent.
- `austen`: AE/relationship copy, replies, follow-up.

## Secret handling

If the user pasted tokens in chat, do not repeat them back. Install them immediately into profile `.env` files if requested, validate via `getMe`, and recommend BotFather rotation after smoke tests because the tokens are now in chat history.

## Files used in the June 2026 migration

- Manifest: `/root/.hermes/alma-agent-bot-migration.yaml`
- Registry: `/root/.hermes/alma-agent-bot-registry.json`
- Token installer: `/root/.hermes/scripts/install_alma_bot_tokens.py`
- Systemd template: `/root/.config/systemd/user/hermes-gateway@.service`

These paths are operational references, not hard requirements for future migrations. Verify current state before acting.
