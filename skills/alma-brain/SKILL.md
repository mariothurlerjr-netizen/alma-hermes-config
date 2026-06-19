---
name: alma-brain
description: Acessa o vault ALMA em /home/almarev/brain via brain-alma MCP (HTTP/8003). LÊ qualquer lugar do vault. ESCREVE só em áreas autorizadas (raw/, agents/hermes/, 04-personal/scratchpad-*, append-only logs). NÃO escreve em planos curados (00-execution-plans/, 03-alma-agentic/decisions.md, platform/*.md, INDEX.md, STATUS.md). Aciona quando Mario disser status, pipeline, onde paramos, anota isso, salva no brain, ou citar entidade por nome.
license: proprietary
---

# Skill alma-brain

## Acesso

- **Vault root**: `/home/almarev/brain`
- **Via**: MCP server `brain-alma` em `http://127.0.0.1:8003/mcp`
- **Tools disponíveis**: brain_search, brain_read, brain_write, brain_query, brain_list, brain_recent, brain_link_check

## Entry points de leitura (sempre nessa ordem)

1. `wiki/hot.md` se existir. Se não existir, não tratar como erro fatal.
2. `STATUS.md` (auto-gerado 24h) — entry point pra qualquer pergunta de "como está / onde paramos / quantos / quando".
3. `INDEX.md` — mapa do vault pra navegar pastas.
4. Pasta específica via `brain_search` ou `brain_read`.

## Estrutura do vault

| Pasta | Conteúdo | Hermes pode escrever? |
|---|---|---|
| `00-execution-plans/` | Planos executivos curados Mario | ❌ READ-ONLY |
| `01-job-search/` | Job search Mario | ❌ READ-ONLY |
| `02-alma-rev/` | Positioning, ICP, brand voice | ❌ READ-ONLY |
| `03-alma-agentic/decisions.md` | Decisões técnicas canônicas | ❌ READ-ONLY |
| `03-alma-agentic/2026-05/` | Activity logs auto-gerados Cortex | ✅ append-only |
| `04-personal/` | Preferências, agenda Mario | ⚠️ só `scratchpad-*.md` |
| `05-meeting-notes/granola/` | Granola notes | ⚠️ só append-only |
| `06-claire-calls/` | Logs CLAIRE | ❌ agent-owned |
| `07-lance-outreach/` | LANCE logs | ❌ agent-owned |
| `08-scribe-content/` | SCRIBE drafts | ❌ agent-owned |
| `09-iris-reports/` | IRIS digests | ❌ agent-owned |
| `10-signals/` | Sentinel signals | ❌ agent-owned |
| `agents/hermes/` | **Workspace próprio Hermes** | ✅ WRITE LIVRE |
| `lead-base/` | Leads ORION | ❌ agent-owned |
| `platform/` | Cross-tenant runbooks curados Mario | ❌ READ-ONLY |
| `raw/` | **Capture rápido sem curadoria** | ✅ WRITE LIVRE |
| `99-archive/` | Logs antigos | ❌ READ-ONLY |
| `INDEX.md` / `STATUS.md` | Meta/auto-gerado | ❌ NÃO TOCAR |

## Protocolo de escrita

Quando Mario diz **"anota no brain X"**, **"salva isso"**, **"registra X"**:

1. **Decide WHERE**:
   - Captura rápida sem categoria clara → `raw/YYYY-MM-DD-<slug>.md`
   - Observação sobre o próprio Hermes (decisão de design, problema detectado, padrão útil) → `agents/hermes/YYYY-MM-DD-<slug>.md`
   - Preferência ou contexto pessoal Mario → considera primeiro `~/.hermes/memories/MEMORY.md` (mais near-term que brain)
   - Aprendizado técnico re-aplicável → `agents/hermes/lessons/<topic>.md`

2. **Estrutura mínima** de qualquer .md escrito:
   ```markdown
   ---
   created_by: hermes
   created_at: 2026-MM-DDTHH:MM:SSZ
   source: telegram | cli | heartbeat
   ---

   # <Título conciso>

   <Conteúdo, com referências [[wiki-link]] quando aplicável>
   ```

3. **Confirma WHAT** ao Mario antes de escrever em pasta sensível:
   - `04-personal/scratchpad-*.md` → não precisa confirmar
   - Qualquer outra área não listada como "WRITE LIVRE" → pergunta "vou anotar em X, ok?" antes

4. **Após escrever**: confirma path + 1 linha do conteúdo. Mario vê `git diff` no auto-push.

## Operações proibidas

- **NUNCA** modificar arquivos curados manualmente (00-, 02-, 03-decisions, platform/, INDEX, STATUS)
- **NUNCA** tocar `/home/almarev/agentic-v2/`, `/home/almarev/agentic/`, `/root/alma-aios/` (esse é skill alma-cortex-lite, não alma-brain)
- **NUNCA** deletar arquivos do brain (não tem brain_delete_tool, mas se aparecer não usar)
- Se em dúvida sobre onde escrever → `raw/` é sempre a opção segura

## Workflow padrão

1. Mario pergunta algo → lê `STATUS.md` primeiro
2. Cruza com entidade específica → `brain_search` na pasta certa
3. Para perguntas sobre papel/função de agente ALMA (ex.: IRIS, LUNA, MUSE, LANCE), não confiar em uma única fonte: comparar `STATUS.md`, `platform/runbooks/<agente>.md`, `03-alma-agentic/agent-roster-map.md`, `03-alma-agentic/decisions.md` e arquivos recentes do agente. Se houver drift entre desenho antigo, uso atual e decisão explícita do Mario, declarar a inconsistência e tratar a decisão mais recente do Mario como operacional.
   - Decisão operacional explícita do Mario (2026-06-19): **IRIS = metrics/dashboard/analytics/digest/anomaly detection/cost-performance reporting**. Não tratar IRIS como sales closer, account research ou marketing sem nova revisão explícita. Marketing fica em LUNA/MUSE/SOL/ALEXANDER; vendas/outbound ficam em ORION/LANCE/CLAIRE/NOVA/TRUTH ENGINE conforme contexto.
4. Se a pergunta gira em torno de um número específico (ex.: "630 contas ativas"), fazer também busca exata pelo número no vault antes de concluir que não está registrado; auditorias e logs costumam conter o número sem os termos semânticos esperados
4. Para status operacional de email/cadência/funil ALMA Rev (ex.: quantos receberam email 1/2/3, quantos estão na cadeia), usar o brain para contexto e depois buscar dado vivo no stack agentic/Instantly quando a pergunta for atual. Ver `references/alma-rev-email-funnel-status.md`.
5. Para auditoria diária de campanha ALMA Rev (confirmar envio, cap, clicks, replies, checks, scanner vs humano, ou report proativo no Telegram), seguir `references/alma-rev-campaign-audit.md`.
6. Para visitas de páginas ALMA Rev, país/estado, exclusão de Brasil, ou correlação entre campanha e tráfego, usar logs nginx + GeoIP e seguir `references/alma-rev-traffic-geo-reporting.md`.
7. Para sender ops do Instantly (quantas contas existem, score/warmup, quais podem entrar em campanha, otimização diária de `email_list`, assinatura de campanhas), consultar dado vivo e seguir `references/alma-rev-instantly-sender-ops.md`.
5. Para sender ops do Instantly (quantas contas existem, score/warmup, quais podem entrar em campanha, otimização diária de `email_list`), consultar dado vivo e seguir `references/alma-rev-instantly-sender-ops.md`.
6. Para CLAIRE / ALMA Local call follow-ups (quem pediu email/callback em ligação, se os emails foram enviados, pendências `CHECK_SENT`), consultar dado vivo no Postgres e seguir `references/alma-rev-claire-call-followups.md`. Não confundir “emails passados” nesse contexto com Instantly sequence emails.
7. Se faltar dado depois de 2 reads + 1 search → fala "não está no vault", NÃO inventa
8. Se Mario manda anotar → escreve em `raw/` ou `agents/hermes/` com frontmatter, confirma path

## Trigger keywords

**Leitura**: status, pipeline, onde paramos, ORION, CLAIRE, Clara, ALMA Rev, alma-aios, agentic, JT Shirts, mtmalls, Workstream B, LANCE, IRIS, SCRIBE, Sentinel, Shield, Brain, workspace, tenant.

**Escrita**: anota, salva no brain, registra, grava, lembra disso, escreve no brain, captura.
