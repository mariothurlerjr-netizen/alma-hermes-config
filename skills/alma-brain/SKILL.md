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
5. Se a pergunta atravessar **Agentic / Rev / Local**, ler primeiro os `_CURRENT.md` canônicos relevantes e usar `references/alma-current-state-entrypoints.md` como mapa rápido.
6. Se o tema for reuniões/calls/recaps, ler `references/granola-meeting-notes.md` e depois buscar em `05-meeting-notes/granola/`.

## Protocolo anti-stale / anti-confusão

Antes de responder com base no brain, montar explicitamente a hierarquia de verdade:

1. **Estado vivo** quando a pergunta for operacional (Instantly, Postgres, systemd, nginx, APIs, repo): brain dá contexto, mas a resposta final usa dado live verificado.
2. **Arquivos canônicos atuais**: `STATUS.md`, `_CURRENT.md`, `business-plan.md` com `status: canonical`, docs marcados como fonte canônica no próprio `_CURRENT.md`.
3. **Decisões recentes**: `decisions.md`, notas com timestamp mais novo e updates/addenda. Se uma nota antiga tem addendum mais novo, o addendum vence.
4. **Arquivos históricos**: qualquer nota sem `status: canonical`, com `obsolete`, `superseded`, nome antigo, ou timestamp anterior a uma decisão mais nova. Usar só como contexto, não como verdade operacional.

Regras obrigatórias:
- Sempre verificar `frontmatter.status`, `last_reviewed`, `timestamp`, `last_modified` e termos como `OBSOLETO`, `superseded`, `CURRENT`, `canonical`, `pendente`.
- Se duas fontes conflitarem, declarar o conflito e escolher a fonte canônica mais recente ou o dado vivo.
- Nunca responder “o brain diz X” quando X veio de arquivo antigo sem cruzar com `_CURRENT.md`/`STATUS.md`.
- Para pedidos do tipo “leia o brain inteiro e reveja”, tratar como sinal de stale-read: refazer leitura ampla, listar fontes mais recentes consideradas e corrigir a conclusão anterior.

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

Support file with a concise write-validation + reporting note: `references/brain-write-frontmatter-and-report-matrix.md`.
Support file for cross-domain status reads: `references/alma-current-state-entrypoints.md`.

When a raw capture starts looking reusable, promote it deliberately:
- `raw/` = session capture, scratchpad, one-off derivation
- `agents/hermes/lessons/` = reusable operating pattern, template, or workflow
- keep the raw artifact as the provenance trail, then promote the distilled version into `agents/hermes/`
- if the user keeps asking for the same shape again, prefer turning it into a reusable template instead of leaving it as a one-off note

When Mario diz **"anota no brain X"**, **"salva isso"**, **"registra X"**:

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

1. Mario pergunta algo → lê `STATUS.md` primeiro quando a pergunta for sobre estado, depois cruza com a entidade/pasta específica.
2. Quando Mario pedir **“all status”**, **“tudo que temos”** ou equivalente, sintetizar em mapa operacional por área, sempre nesta ordem:
   - `STATE CURRENT` com uma linha de verdade viva, se houver.
   - `Infra / serviços`
   - `Agentes / automações`
   - `Business P0`
   - `Pendências manuais`
   - `In progress` e `Blocked`
   - `Próximo movimento`
   Usar `STATUS.md` + `_CURRENT.md` + notas/recentes relevantes, sem misturar histórico velho com estado vivo. Se houver execução-plan ativa ou task list atual, incluir só o que ainda está aberto.
3. Para perguntas sobre papel/função de agente ALMA (ex.: IRIS, LUNA, MUSE, LANCE), não confiar em uma única fonte: comparar `STATUS.md`, `platform/runbooks/<agente>.md`, `03-alma-agentic/agent-roster-map.md`, `03-alma-agentic/decisions.md` e arquivos recentes do agente. Se houver drift entre desenho antigo, uso atual e decisão explícita do Mario, declarar a inconsistência e tratar a decisão mais recente do Mario como operacional.
   - Decisão operacional explícita do Mario (2026-06-19): **IRIS = metrics/dashboard/analytics/digest/anomaly detection/cost-performance reporting**. Não tratar IRIS como sales closer, account research ou marketing sem nova revisão explícita. Marketing fica em LUNA/MUSE/SOL/ALEXANDER; vendas/outbound ficam em ORION/LANCE/CLAIRE/NOVA/TRUTH ENGINE conforme contexto.
4. Se a pergunta gira em torno de um número específico (ex.: "630 contas ativas"), fazer também busca exata pelo número no vault antes de concluir que não está registrado; auditorias e logs costumam conter o número sem os termos semânticos esperados.
5. Para status operacional de email/cadência/funil ALMA Rev (ex.: quantos receberam email 1/2/3, quantos estão na cadeia), usar o brain para contexto e depois buscar dado vivo no stack agentic/Instantly quando a pergunta for atual. Ver `references/alma-rev-email-funnel-status.md`.
6. Para perguntas do tipo "empresa/processos/sugestões de implementação", separar sempre em `already implemented`, `still on paper` e `recommended next move`. A referência compacta está em `references/strategy-implementation-split.md`.
7. Para diagnóstico de “travado”, deliverability, domínio/sender ou campanha ALMA Rev (`getalmarev`, `tryalmarev`, Instantly, warmup, reply rate), NÃO responder só de memória nem só de um arquivo. Ler o estado canônico (`STATUS.md`, `03-alma-agentic/_CURRENT.md`, `02-alma-rev/business-plan.md` quando envolver GTM), depois as notas específicas (`deliverability-*`, `outbound-*`) e então verificar dado vivo no stack/env/systemd. Separar: (a) decisão/brain canônico, (b) estado live, (c) drift operacional encontrado. Checar explicitamente campanhas `[OBSOLETE]` ainda ativas e conexões `provider_code`/`warmup_status` das caixas antes de concluir causa-raiz. Se Mario mandar “leia o brain inteiro e reveja”, tratar como correção: refazer a leitura ampla e entregar diagnóstico corrigido direto.
8. Para auditoria diária de campanha ALMA Rev (confirmar envio, cap, clicks, replies, checks, scanner vs humano, ou report proativo no Telegram), seguir `references/alma-rev-campaign-audit.md`.
9. Para ORION Lead Factory controlado (leads homologados, CSV Instantly-ready, dedupe, Prospeo/Findymail/Apollo reveal waterfall, cap 25k), seguir `references/orion-lead-factory-controlled.md` antes de mexer em workers/cron. Se a conversa estiver sobre a etapa de origem/qualificação, descrever o fluxo como `ICP search -> mails.do -> account verification`.
10. Para visitas de páginas ALMA Rev, país/estado, exclusão de Brasil, ou correlação entre campanha e tráfego, usar logs nginx + GeoIP e seguir `references/alma-rev-traffic-geo-reporting.md`.
11. Para sender ops do Instantly (quantas contas existem, score/warmup, quais podem entrar em campanha, otimização diária de `email_list`, assinatura de campanhas), consultar dado vivo e seguir `references/alma-rev-instantly-sender-ops.md`.
12. Para lead search / ORION lead factory, seguir `references/orion-lead-factory-verification.md` antes de concluir que o pipeline está parado, e separar estado do cron, worker e verificador.
13. Para CLAIRE / ALMA Local call follow-ups (quem pediu email/callback em ligação, se os emails foram enviados, pendências `CHECK_SENT`), consultar dado vivo no Postgres e seguir `references/alma-rev-claire-call-followups.md`. Não confundir “emails passados” nesse contexto com Instantly sequence emails.
13. Se faltar dado depois de 2 reads + 1 search → fala "não está no vault", NÃO inventa.
14. Se Mario manda anotar → escreve em `raw/` ou `agents/hermes/` com frontmatter, confirma path.
15. When a user asks for the next artifact in a sequence and it is already clear from context, execute it directly instead of pausing on an intermediate summary or confirmation.
16. Quando um passo de verificação ou refresh for o próximo passo natural e for de baixo risco, execute-o sem pedir permissão. Não pare em um artefato intermediário se a próxima checagem honesta já estiver clara, por exemplo, depois de registrar um estado `PROVISIONAL`, refrescar `STATUS.md` ou checar resposta live antes de devolver a conclusão.

## Status delivery rules
- Em respostas de status, evitar narrativa genérica. Preferir blocos curtos com estado atual, pendências, bloqueios e próximo movimento.
- Se a sessão tiver um plano ativo, task list ou execução em andamento, refletir isso no status em vez de ignorar.
- Se a resposta for por voz ou o usuário mandar áudio, manter a versão verbal curta e operacional, e deixar o fallback texto mínimo.
## Trigger keywords

**Leitura**: status, pipeline, onde paramos, ORION, CLAIRE, Clara, ALMA Rev, alma-aios, agentic, JT Shirts, mtmalls, Workstream B, LANCE, IRIS, SCRIBE, Sentinel, Shield, Brain, workspace, tenant, Granola, meeting notes, recap, calls.

**Escrita**: anota, salva no brain, registra, grava, lembra disso, escreve no brain, captura.
