# alma-hermes-config

Configuração customizada do runtime Hermes Agent (v0.13.0, Nous Research) usado como camada cockpit/operacional do **ALMA Platform**.

Path canônico na VPS: `/root/.hermes/`
Service: `hermes-gateway.service` (systemd --user, sob root)
Binary: `/home/almarev/agentic-v2/hermes-agent/.venv/bin/hermes`
Symlink: `/usr/local/bin/hermes` + `/root/.local/bin/hermes`

## O que contém

| Item | Função |
|---|---|
| `SOUL.md` | Persona Hermes ALMA-aware (substituí o boilerplate Nous Research) |
| `HEARTBEAT.md` | Checklist do ciclo proativo 30min (cron) |
| `memories/USER.md` | Identidade Mario + estilo de comunicação |
| `memories/MEMORY.md` | Estado operacional persistente cross-session |
| `memories/corrections.md` | Corrections log (anti-repetição de erros) |
| `memories/patterns.md` | Padrões de falha detectados |
| `skills/alma-brain/` | Skill READ-ONLY pro vault `/home/almarev/brain` |
| `skills/alma-context/` | Skill de contexto operacional |
| `skills/alma-cortex-lite/` | Skill ponte pro Cortex |
| `skills/alma-feedback-loop/` | Loop de correções |
| `skills/alma-heartbeat/` | Skill do ciclo proativo |
| `skills/alma-self-eval/` | Auto-avaliação de qualidade de resposta |
| `skills/alma-style/` | Style guide PT-BR + anti-padrões |
| `config.yaml` | Provider config (OpenAI Codex OAuth) + MCP servers (brain-alma) |

## O que NÃO está versionado

- `.env` (secrets: TELEGRAM_BOT_TOKEN, ALMA_BRAIN_MCP_KEY, GITHUB_TOKEN)
- `state.db` / `oauth.db` (sessões SQLite)
- `sessions/` / `logs/` / `checkpoints/` (runtime artifacts)
- `brain/` (placeholder antigo — o brain real está em `/home/almarev/brain` versionado em `mariothurlerjr-netizen/alma-brain`)
- `memory/heartbeat-log.md` (append-only log, regerável)

## MCP integrations

- **brain-alma** (`http://127.0.0.1:8003/mcp`): 7 tools — brain_search, brain_read, brain_write, brain_query, brain_list, brain_recent, brain_link_check. Auth via `ALMA_BRAIN_MCP_KEY` bearer.

## Restore em VPS nova

```bash
# 1. Instalar Hermes upstream
cd /home/almarev/agentic-v2 && git clone https://github.com/NousResearch/hermes-agent.git
cd hermes-agent && uv venv && uv pip install -e .

# 2. Restore esta config
cd /root && git clone https://github.com/mariothurlerjr-netizen/alma-hermes-config.git .hermes-restore
cp -r .hermes-restore/* /root/.hermes/

# 3. Recriar .env (secrets fora do repo)
# 4. Habilitar systemd user service + linger
loginctl enable-linger root
systemctl --user enable hermes-gateway
```
