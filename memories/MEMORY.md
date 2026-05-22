### ORION (lead generation pipeline)
- Path: `/home/almarev/agentic/agents/orion/`; Apollo + Snov + Claude OAuth; dashboard em `app.almarev.com/dashboard/leads`.
- `claude -p` em systemd precisa `stdin=DEVNULL`.
- Apollo 422 pode ser `insufficient credits` mesmo com quota interna restante. Em 2026-05-22 worker foi patchado para detectar isso e sair `status=42` (RestartPreventExitStatus), sem loop.