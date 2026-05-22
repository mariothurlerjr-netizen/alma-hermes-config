### ORION (lead generation pipeline)
- Path: `/home/almarev/agentic/agents/orion/`; dashboard em `app.almarev.com/dashboard/leads`.
- `claude -p` em systemd precisa `stdin=DEVNULL`.
- Apollo 422 pode ser `insufficient credits` mesmo com quota interna restante. Em 2026-05-22 ORION ganhou fallback Apollo-free via Tavily candidate search + Snov/Browserbase enrichment; 5 workers ativos entregando leads mesmo sem Apollo credits.
- Watchdog `orion-watchdog` (`fc52bbc58989`) roda a cada 5min via cron script-only (`~/.hermes/scripts/orion_watchdog.py`), habilita/inicia `alma-orion@1..5`, verifica health/stale heartbeats e só alerta no Telegram se intervir ou falhar.
§
ORION watchdog cron `fc52bbc58989` uses rolling `delivered_24h`; this metric can decrease as leads age out. The watchdog script `/root/.hermes/scripts/orion_watchdog.py` treats decreases as baseline movement, not failure, to avoid false `no delivered_24h growth` alerts.