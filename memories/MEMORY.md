### ORION
- Path: `/home/almarev/agentic/agents/orion/`; dashboard `app.almarev.com/dashboard/leads`.
- `claude -p` em systemd precisa `stdin=DEVNULL`.
- Apollo 422 pode ser `insufficient credits`; com créditos exauridos, processar leads já extraídos, não extrair novos. Fallback Apollo-free: Tavily + Snov/Browserbase.
- ORION pausado intencionalmente desde 2026-06-13: Hermes cron `fc52bbc58989` pausado, `alma-orion-watchdog.timer` disabled/inactive e `alma-orion@1..5` disabled/inactive.
§
ORION watchdog cron `fc52bbc58989` uses rolling `delivered_24h`; `/root/.hermes/scripts/orion_watchdog.py` treats flat or decreasing delivery counts as KPI movement, not worker-health failure. Hard failures are missing/stale/erroring workers or failed repairs.
§
LANCE is intentionally paused because it is not fully configured yet. On 2026-05-24, `alma-lance.timer` was disabled and stopped; `alma-lance.service` remained inactive.
§
Na VPS do Mario, `hermes-gateway` roda como user service de root (`XDG_RUNTIME_DIR=/run/user/0 systemctl --user ...`); não há Unix user `almarev`, então units ALMA não devem usar `User=almarev` sem criar o user.
§
Hermes TTS on Mario's VPS is configured to provider `elevenlabs`; `/root/.hermes/.env` contains `ELEVENLABS_API_KEY` copied from existing ALMA env. Reminder call script path: `/root/.hermes/scripts/reminder_reuniao_alisa_call.py`, uses Twilio env and requires `HERMES_REMINDER_TO_PHONE` or `MARIO_PHONE` for phone-call destination.
§
Reminder handling: if Mario asks in shorthand/noisy text to send or remind something at a time, schedule a durable Telegram reminder when the time can be inferred; ask only for missing reminder text or genuinely impossible timing, and state explicitly if no job was created.
§
Mario has a personal Apollo account with roughly 2.5k credits available as a potential ORION credit source; do not paste Apollo secrets in chat, use secret-safe env update on the VPS.
§
ALMA Rev Instantly outbound requires `insert_unsubscribe_header=true` and visible unsubscribe footer/link in every email step: `https://almarev.com/api/v1/public/unsubscribe?email={{email}}`.