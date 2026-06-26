ORION legacy workers are paused/disabled since 2026-06-13; Lead Factory cron `5cd7d23f5197` remains active in CSV-only mode, and the watchdog treats flat/decreasing `delivered_24h` as KPI movement rather than worker failure.
§
Na VPS do Mario, `hermes-gateway` roda como user service de root (`XDG_RUNTIME_DIR=/run/user/0 systemctl --user ...`); não há Unix user `almarev`, então units ALMA não devem usar `User=almarev` sem criar o user.
§
Hermes TTS uses `elevenlabs`; Twilio env + phone vars needed. Mario prefere voz JARVIS, grave e limpa. JT Shirts and mtmalls are closed; cold-base dialing stays paused unless Mario reabrir.
§
Reminder handling: if Mario asks in shorthand/noisy text to send or remind something at a time, schedule a durable Telegram reminder when the time can be inferred; ask only for missing reminder text or genuinely impossible timing, and state explicitly if no job was created.
§
Mario has a personal Apollo account with roughly 2.5k credits available as a potential ORION credit source; do not paste Apollo secrets in chat, use secret-safe env update on the VPS.
§
ALMA Rev Instantly outbound requires `insert_unsubscribe_header=true` and visible unsubscribe footer/link in every email step: `https://almarev.com/api/v1/public/unsubscribe?email={{email}}`.
§
ALMA Rev ops: Instantly warms inboxes; ORION only lead-gen/enrichment. Sender gate: zero warmup spam in last 7d. `getalmarev.com` partially usable; `tryalmarev.com`/others warmup-only until next gate. Mario flagged stale brain confusion: use live data + STATUS/_CURRENT/canonical recent docs before treating old notes as current.
§
ALMA Rev sender separation: `mario@almarev.com` is reserved for owned/warm newsletter-style communication; cold/outbound should use warmed `@getalmarev.com` sender accounts, especially `mario@getalmarev.com`, once warmed.
§
Hermes default profile has `security.redact_secrets=false` after Composio/snippet corruption; keep handling secrets operationally safe by never printing/pasting them, using env files or Python writers instead of chat snippets.
§
Mario clarified that “memória cheia” in the Hermes bug report can mean runtime/RSS or configured process memory around 2.5GB, not only MEMORY.md/USER.md char usage; future Hermes diagnostics must separate OS/RSS memory, model context overflow, and persistent memory char budgets.
§
Claude Code on Mario's VPS is OAuth-authenticated as Mario's Claude Max account; use `/usr/local/bin/hermes-claude` for safe one-shot delegation. Prefer OAuth/device-flow or existing local auth, never paste Anthropic/Claude secrets in chat.
§
Telegram bot @username is effectively fixed after bot creation for normal BotFather operations; BotFather can change display name via /setname/Edit Name, but clean path for a new @ is creating a new bot with /newbot and replacing the token in the relevant Hermes profile .env. @BotSupport exceptions are possible but not reliable.
§
ALMA Hermes Telegram bot fleet is preserved but inactive: per-profile gateways for orchestrator, orion, lance, sentinel, shield, iris, muse, aura, claire, clara, and austen are stopped/disabled as user systemd services. Profiles, tokens, SOUL/CLAUDE files, and registry remain for on-demand reactivation if overload signals warrant workers.
§
ALMA agent operating model: Hermes is the default cockpit, additional bots only when overload signals appear. Mario sees LinkedIn as useful for filtering jobs, Instagram for campaign monitoring, and All Grow may also be worth connecting.
§
On Mario's VPS Hermes default profile uses OpenAI Codex provider with model `gpt-5.5`; OpenAI Codex OAuth credential has ~3h+ cooldown remaining but reverted to original config. OpenRouter fallback available. For Hermes slowness, check provider rate limits first, then model latency and context compression before blaming VPS CPU/RAM.
§
Hermes speed tuning applied on default profile: primary remains OpenAI Codex `gpt-5.5`; auxiliary compression/title/search/etc use OpenRouter `openai/gpt-4o-mini`; Telegram/CLI toolsets are trimmed; emergency fallback is OpenRouter `anthropic/claude-sonnet-4` only after primary failure/quota. Backup: `/root/.hermes/config.yaml.bak-speed-tuning-20260626-1300`.
§
Manual Telegram getUpdates calls against an already-polling Hermes bot gateway can trigger HTTP 409 Conflict and log a polling-conflict warning. For live gateway verification, prefer systemd/journal/service state or a non-polling probe rather than calling getUpdates directly on the active bot token.
§
No perfil default do Hermes, o Composio MCP está ativo em `/root/.hermes/config.yaml` apontando para `https://backend.composio.dev/v3/mcp/21c80646-00d9-45bc-a84a-52e3f1d3ba66?user_id=mario` com `x-api-key: ${COMPOSIO_API_KEY}`; `hermes mcp test composio` conectou e descobriu 508 tools.