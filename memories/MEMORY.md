Legacy ORION workers remain intentionally paused since 2026-06-13 (`fc52bbc58989` paused, `alma-orion@1..5` disabled), but controlled ORION Lead Factory cron `5cd7d23f5197` is active in CSV-only mode for ALMA Rev lead production.
§
ORION watchdog cron `fc52bbc58989` uses rolling `delivered_24h`; `/root/.hermes/scripts/orion_watchdog.py` treats flat or decreasing delivery counts as KPI movement, not worker-health failure. Hard failures are missing/stale/erroring workers or failed repairs.
§
Na VPS do Mario, `hermes-gateway` roda como user service de root (`XDG_RUNTIME_DIR=/run/user/0 systemctl --user ...`); não há Unix user `almarev`, então units ALMA não devem usar `User=almarev` sem criar o user.
§
Hermes TTS uses `elevenlabs`; reminder calls need Twilio env + phone vars. JT Shirts and mtmalls are closed workstreams. CLAIRE/ALMA Local cold-base dialing stays paused unless Mario reopens it; accepted callback follow-ups are allowed.
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
Hermes secret redaction can corrupt example shell snippets that contain secret-looking env assignments such as `COMPOSIO_API_KEY=...` before delivery/persistence; for secret-save instructions, avoid emitting literal assignment snippets and prefer an attached script file or Python writer that never prints the secret.
§
Mario clarified that “memória cheia” in the Hermes bug report can mean runtime/RSS or configured process memory around 2.5GB, not only MEMORY.md/USER.md char usage; future Hermes diagnostics must separate OS/RSS memory, model context overflow, and persistent memory char budgets.
§
Mario is willing to give Hermes access to Claude Code for use when necessary; prefer OAuth/device-flow or existing local Claude CLI auth, never paste Anthropic/Claude secrets in chat.
§
Claude Code on Mario's VPS is OAuth-authenticated as Mario's Claude Max account. `/usr/local/bin/hermes-claude` is the safe Hermes wrapper for one-shot Claude Code delegation; bwrap and socat are installed so `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB=1` works.
§
Telegram bot @username is effectively fixed after bot creation for normal BotFather operations; BotFather can change display name via /setname/Edit Name, but clean path for a new @ is creating a new bot with /newbot and replacing the token in the relevant Hermes profile .env. @BotSupport exceptions are possible but not reliable.
§
ALMA Hermes Telegram bot migration: per-profile gateways are enabled via user systemd template `/root/.config/systemd/user/hermes-gateway@.service`. Running profiles include orchestrator `@Nyx_Mario_bot`, orion `@ALMAAILeadBot`, lance `@ALMACriticBot`, sentinel `@sentinel_almabot`, shield `@ALMACOOBot`, iris `@ALMACTOBot`, muse `@alma_aios_bot`, aura `@almaco_ceo_bot`, claire `@alma_cfo_bot`, and austen `@alma_cmo_bot`. CLARA remains stopped and needs its own unique bot token if it should run separately.