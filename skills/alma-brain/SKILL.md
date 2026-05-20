---
name: alma-brain
description: Acessa o vault ALMA em /home/almarev/brain em modo READ-ONLY. Primeiro stop SEMPRE = STATUS.md (auto-gerado 24h). Aciona quando Mario disser status, pipeline, onde paramos, ou citar entidade por nome (ORION, CLAIRE, JT Shirts, ALMA Rev, mtmalls, Workstream B, etc.).
license: proprietary
---

# Skill alma-brain

## Filesystem (READ-ONLY)

- **Vault root**: `/home/almarev/brain`
- **STATUS** (auto-gerado): `/home/almarev/brain/STATUS.md` — LER PRIMEIRO sempre. Atualiza 24h via `brain-mcp/scripts/generate_status.py`. Tem agentes + ORION metrics + workspaces ativos.
- **INDEX** (mapa do vault): `/home/almarev/brain/INDEX.md` — use pra navegar entre pastas.
- **Wiki sintetizado**: `/home/almarev/brain/wiki/sistema/`, `/home/almarev/brain/wiki/workspaces/`

## Estrutura do vault (pastas numeradas)

| Pasta | Conteúdo |
|---|---|
| `00-execution-plans/` | Planos executivos (north star, roadmap, build sequences) |
| `01-job-search/` | Job search ativo |
| `02-alma-rev/` | Positioning, ICP, brand voice, Revenue Leak Diagnostic |
| `03-alma-agentic/` | Decisões técnicas, architecture, MCP config, tenant patterns |
| `04-personal/` | Preferências Mario, idioma, agenda |
| `05-meeting-notes/granola/` | Meeting notes Granola |
| `06-claire-calls/` | Logs CLAIRE outbound |
| `07-lance-outreach/` | LANCE LinkedIn + email logs |
| `08-scribe-content/` | SCRIBE drafts, posts |
| `09-iris-reports/` | IRIS daily/weekly/monthly digests |
| `10-signals/` | Sentinels (LinkedIn, news, CRM) |
| `agents/` | Per-agent workspace docs (ORION + futuros) |
| `lead-base/` | Leads aprovados ORION |
| `platform/` | Cross-tenant positioning, runbooks, service catalog |
| `99-archive/` | Logs antigos |

## Workflow padrão

1. Lê `STATUS.md` (sempre — é o entry point).
2. Identifica entidade na pergunta do Mario (agente, projeto, workspace).
3. Pula pra pasta relevante via `INDEX.md` ou grep direto:
   ```bash
   grep -r "termo" /home/almarev/brain --include="*.md" -l | head -10
   ```
4. Lê o arquivo .md específico em vez de despejar pastas inteiras.
5. Se faltar dado depois de grep + 2 reads, fala "não está no vault" — NÃO invente.

## Operações permitidas
- read, grep, find no vault.

## Operações proibidas
- write, edit, delete (POC read-only).
- Mexer em `/home/almarev/agentic-v2/`, `/home/almarev/agentic/`, `/root/alma-aios/`.
- Se Mario pedir pra escrever no vault, recusa explicando POC read-only.

## Trigger keywords
status, pipeline, onde paramos, ORION, CLAIRE, Clara, ALMA Rev, alma-aios, agentic, JT Shirts, mtmalls, Workstream B, LANCE, IRIS, SCRIBE, Sentinel, Shield, Brain, workspace, tenant.
