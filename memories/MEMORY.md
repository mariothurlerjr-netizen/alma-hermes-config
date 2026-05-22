### ORION (lead generation pipeline)
- Path: `/home/almarev/agentic/agents/orion/`; dashboard em `app.almarev.com/dashboard/leads`.
- `claude -p` em systemd precisa `stdin=DEVNULL`.
- Apollo 422 pode ser `insufficient credits` mesmo com quota interna restante. Em 2026-05-22 ORION ganhou fallback Apollo-free via Tavily candidate search + Snov/Browserbase enrichment; 5 workers ativos entregando leads mesmo sem Apollo credits.