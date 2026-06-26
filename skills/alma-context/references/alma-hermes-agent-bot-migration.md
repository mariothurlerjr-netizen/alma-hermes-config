# ALMA Hermes agent bot migration

Use when Mario asks to migrate Claude/LangGraph/OpenClaw-style agents into Hermes, create one bot per agent, rename bot identities, or coordinate the ALMA agent fleet in Hermes profiles.

## Target architecture

- One Hermes profile per ALMA agent.
- Each profile owns its own identity files:
  - `/root/.hermes/profiles/<profile>/SOUL.md`
  - `/root/.hermes/profiles/<profile>/CLAUDE.md`
  - `/root/.hermes/profiles/<profile>/.env`
- Telegram separation requires one Telegram bot token per profile. Reusing the same token across profiles makes the profiles separate in Hermes but not separate Telegram bots.
- Keep a migration manifest at `/root/.hermes/alma-agent-bot-migration.yaml` or equivalent, mapping: profile, bot display name, role, old runtime, current status, notes.

## Canonical ALMA profile split

- `jarvis`: executive assistant and router.
- `orchestrator`: COO of the ALMA agent fleet, board, registry, routing.
- `orion`: lead factory and enrichment, no outbound sending.
- `lance`: outbound queue, Instantly, compliance, sender health.
- `sentinel`: market/prospect/content signals with source and timestamp.
- `shield`: reliability, memory/context/gateway/system health.
- `iris`: metrics, dashboards, analytics, digests, anomaly detection. Do not treat IRIS as closer or marketer.
- `muse`: content and creative drafts, no publishing without approval.
- `aura`: workflow/diagnostic integrations.
- `claire`: voice SDR, Twilio/ElevenLabs/Vapi. Cold dialing stays paused unless Mario reopens.
- `clara`: interview agent.
- `austen`: AE relationship copy, follow-ups and objection drafts.

## Secret-safe token workflow

1. Never ask Mario to paste Telegram bot tokens into Telegram chat.
2. Provide or use an interactive installer on the VPS that prompts with hidden input and writes each token to the right profile `.env`.
3. The installer should update only `TELEGRAM_BOT_TOKEN` and preserve existing `.env` lines such as `TELEGRAM_ALLOWED_USERS`.
4. Validate tokens with Telegram `getMe` without printing token values.
5. After tokens are installed, start or restart each selected profile gateway and verify:
   - the bot responds only to Mario,
   - the loaded SOUL identity matches the profile,
   - session/memory does not bleed across agents.

## Rollout sequence

1. Inventory current profiles with `hermes profile list` and verify each profile has `SOUL.md`, `CLAUDE.md`, `.env`.
2. Check whether profiles share the same `TELEGRAM_BOT_TOKEN`. If they do, call this out as the main blocker for separate Telegram bots.
3. Create or update the migration manifest before changing tokens.
4. Install tokens profile by profile via hidden input.
5. Validate token identities with `getMe`.
6. Start a small first wave of gateways before enabling all bots. Recommended first wave: `jarvis`, `orchestrator`, `orion`, `lance`, `iris`, `sentinel`, `shield`.
7. Keep risky operational agents paused until Mario explicitly reopens their workflows, especially `lance` outbound and `claire` cold dialing.

## Pitfalls

- A profile alias is not a bot. It only gives CLI routing.
- A profile with `SOUL.md`/`CLAUDE.md` is not a separate Telegram bot until it has its own token and gateway instance/routing.
- Bot username changes usually require BotFather. Display names/descriptions/commands can be automated, but `@username` is constrained by Telegram.
- Do not restart every gateway blindly. Validate a first wave, then expand.
