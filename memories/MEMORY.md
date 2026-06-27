ORION lead factory was resumed on 2026-06-27: `orion-watchdog` cron resumed and `alma-orion@1..5.service` plus `alma-orion-watchdog.timer` are active/enabled.
§
Na VPS do Mario, `hermes-gateway` roda como user service de root (`XDG_RUNTIME_DIR=/run/user/0 systemctl --user ...`); não há Unix user `almarev`, então units ALMA não devem usar `User=almarev` sem criar o user.
§
Hermes TTS uses `elevenlabs`; Twilio voice number for Hermes is a local São Paulo BR number, +551150281678, SID PN890f19fb4c1d2e370f6d20679da8c61e. It has Voice/Fax/SIP capabilities, messaging is unavailable, and inbound call webhooks point to ElevenLabs/Twilio endpoints.
§
Reminder handling: if Mario asks in shorthand/noisy text to send or remind something at a time, schedule a durable Telegram reminder when the time can be inferred; ask only for missing reminder text or genuinely impossible timing, and state explicitly if no job was created.
§
Mario prefere mails.do no fluxo de lead verification/enrichment do Lead Factory, depois da busca ICP e antes da verificação da conta.
§
ALMA Rev Instantly outbound requires `insert_unsubscribe_header=true` and visible unsubscribe footer/link in every email step: `https://almarev.com/api/v1/public/unsubscribe?email={{email}}`.
§
`getalmarev.com` is a functioning workspace; the active issue is sender/provider alignment and deliverability, not workspace availability. `tryalmarev.com` and other domains remain warmup-only until the next gate.
§
ALMA Rev sender separation: the `@getalmarev.com` sending accounts live in Gmail Workspace; `mario@almarev.com` is reserved for owned/warm newsletter-style communication, and cold/outbound should use warmed `@getalmarev.com` sender accounts, especially `mario@getalmarev.com`, once warmed.
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
Hermes default profile speed tuning keeps OpenAI Codex `gpt-5.5` as primary, with lighter OpenRouter models for auxiliary tasks and a fallback to Claude Sonnet 4.
§
No perfil default do Hermes, o Composio está filtrado para Gmail, Notion, Outlook, LinkedIn, Instagram e Supabase; `resources` e `prompts` ficam desativados. Granola também foi registrado no MCP do perfil e exige OAuth para uso.
§
Mario prefere priorizar RevOps, fechamento, oferta/pricing e higiene de pipeline antes de customer success, retenção/expansão e market intelligence pesada quando ainda não há clientes.
§
Hermes voice stack on Mario's VPS still has Retell/Vapi plumbing, but Bland AI is now the preferred call-flow path when switching away from ElevenLabs-style orchestration.
§
No perfil default do Hermes, o Telegram home channel ficou configurado em `TELEGRAM_HOME_CHANNEL=8293697290`, então DMs do Mario passam a ter destino padrão para respostas e deliveries proativos.
§
Mario clarified that getalmarev.com is a functioning workspace; the active problem is sender/provider alignment and deliverability (not a workspace outage).