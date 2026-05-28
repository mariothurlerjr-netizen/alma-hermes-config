### ORION (lead generation pipeline)
- Path: `/home/almarev/agentic/agents/orion/`; dashboard em `app.almarev.com/dashboard/leads`.
- `claude -p` em systemd precisa `stdin=DEVNULL`.
- Apollo 422 pode ser `insufficient credits` mesmo com quota interna restante. Em 2026-05-22 ORION ganhou fallback Apollo-free via Tavily candidate search + Snov/Browserbase enrichment; 5 workers ativos entregando leads mesmo sem Apollo credits.
- Watchdog `orion-watchdog` (`fc52bbc58989`) roda a cada 5min via cron script-only (`~/.hermes/scripts/orion_watchdog.py`), habilita/inicia `alma-orion@1..5`, verifica health/stale heartbeats e só alerta no Telegram se intervir ou falhar.
§
ORION watchdog cron `fc52bbc58989` uses rolling `delivered_24h`; `/root/.hermes/scripts/orion_watchdog.py` treats flat or decreasing delivery counts as KPI movement, not worker-health failure. Hard failures are missing/stale/erroring workers or failed repairs.
§
ORION should not extract new leads from Apollo when Apollo credits are exhausted; it should process already extracted/stored Apollo leads whose credits were already consumed.
§
LANCE is intentionally paused because it is not fully configured yet. On 2026-05-24, `alma-lance.timer` was disabled and stopped; `alma-lance.service` remained inactive.
§
Na VPS do Mario, `hermes-gateway` roda como user service de root (`XDG_RUNTIME_DIR=/run/user/0 systemctl --user ...` ou `hermes gateway status`); a system unit `hermes-gateway.service` pode aparecer ausente/inativa sem indicar falha.
§
Hermes TTS on Mario's VPS is configured to provider `elevenlabs`; `/root/.hermes/.env` contains `ELEVENLABS_API_KEY` copied from existing ALMA env. Reminder call script path: `/root/.hermes/scripts/reminder_reuniao_alisa_call.py`, uses Twilio env and requires `HERMES_REMINDER_TO_PHONE` or `MARIO_PHONE` for phone-call destination.