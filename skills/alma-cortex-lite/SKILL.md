---
name: alma-cortex-lite
description: Orquestrador POC. Aciona em pedido vago, multi-frente, ou quando Mário pede pra decidir o que fazer.
license: proprietary
---

# CORTEX Lite

## Triagem
| Pedido | Agent canônico | Ação |
|---|---|---|
| Outbound, SDR, cold email | LANCE | Sugere prompt, não executa |
| Voice call | CLAIRE | Sugere |
| Monitoring, alerts | SENTINEL | Sugere |
| Content, LinkedIn | LUNA/SOL | Rascunha com alma-style |
| Cliente, deal | CORTEX | Lê alma-brain primeiro |
| Job search | Direto | Aplica estratégia |
| Técnica, código, VPS | Direto | Opera no repo ativo correto; para ALMA Agentic use `/home/almarev/agentic` |

## Regra de ouro
Para ALMA Agentic operacional, o repo ativo é `/home/almarev/agentic`. Antes de editar, confirma contexto no brain quando a tarefa menciona entidade ALMA; depois executa e valida no serviço/DB/health correspondente.

## Pedido vago
1. Lê `wiki/hot.md` se existir; se o arquivo não existir, usa `STATUS.md` + busca específica no brain. Não trava nem inventa hot file.
2. Lista 3 items mais quentes de Open Loops + Active Pipeline.
3. Recomenda 1 com justificativa.
4. Executa se autonomia clara, pergunta confirmação curta caso contrário.

## Agentic org design
Quando Mario disser que os cargos da empresa são ocupados por agentes, trate a empresa como um org chart agentic, não como estrutura humana tradicional.

- Comece por mapear a empresa em camadas, normalmente: control, revenue, pipeline, billing/recovery e infra.
- Para cada agente, registre um scorecard mínimo, missão, inputs, outputs, decisões sem escalada, triggers de escalada, KPI e falha típica.
- Se o estado do brain mostrar agentes ativos, pausados ou inativos, mantenha todos no mapa, mas separe claramente o que está vivo do que está dormindo.
- Não pare em nomes de agente. Sempre explicite handoff, owner do KPI e o que é pipeline-machine versus role operacional.
- Quando houver sobreposição de função, priorize uma única rota de responsabilidade antes de propor novos agentes.

Ver referência: `references/agentic-org-matrix.md`.

## Debug técnico em agentes/APIs

Referências de sessão:
- `references/orion-apollo-422.md` — diagnosticar Apollo `422` por body e não por status.
- `references/orion-apollo-free-fallback.md` — manter ORION entregando leads sem créditos Apollo usando Tavily + enriquecimento posterior.
- `references/orion-stored-apollo-leads.md` — processar leads Apollo já pagos em `hub_leads` antes de live sourcing/fallback.
- `references/orion-apollo-personal-credits.md` — trocar `APOLLO_API_KEY` para uma conta Apollo alternativa com créditos sem expor secret no chat, reiniciar workers e validar logs/output.
- `references/alma-local-followup-timestamps.md` — diferenciar horário de criação do follow-up vs horário real da call (`local_call_follow_ups.created_at` vs `local_call_log.dispatched_at`).
- `references/alma-local-callback-only-dialer.md` — pausar discagem de base fria/existente sem bloquear callback aceito; usar `local_call_follow_ups` como gate.
- `references/alma-local-check-submit-phone-null.md` — corrigir `/api/local/check/submit` quando payload sem telefone tenta criar `local_leads` e viola `phone NOT NULL`.

### ALMA Local / CLAIRE dialer

- Quando Mario pausar ou mudar a estratégia da CLAIRE Local, interprete como bloqueio da base fria/existente por padrão. Preserve follow-up de callback quando o prospect já aceitou ser chamado de volta.
- Para callbacks aceitos, o gate canônico é `local_call_follow_ups.kind='callback' AND status='pending' AND due_at <= now()` com DNC/janela/max attempts preservados, não `local_leads.status IN ('new','retry')`.
- Leads `completed` podem continuar elegíveis para callback aceito. Não usar status `completed` como veto absoluto nesse caso.
- Antes de responder “vai ligar” ou “não vai ligar”, verifique systemd/cron e DB: timers ativos, `local_dialer_config.paused`, callbacks pendentes/vencidos, janela local.

1. Quando um erro externo vem só como status code (`422`, `400`, `403`, etc.), reproduz a chamada mínima e lê o body da resposta antes de patchar lógica. Em Apollo, `422` pode ser quota/credits exhausted, não payload inválido.
2. Não trate bloqueio externo como resposta final se existe rota substituta. Se Mario disser que o objetivo é resolver, você é responsável por manter o pipeline produzindo: implemente fallback degradado, rode, valide entrega real e só então reporte tradeoffs.
3. Diferencia falha recuperável, condição terminal e modo degradado. Exemplo ORION: Apollo `insufficient credits` não deve loopar nem necessariamente parar tudo; pode acionar fallback Tavily candidate search + Browserbase/Snov enrichment enquanto créditos não voltam.
4. Em ORION, separa live sourcing de processamento de leads já pagos. Se existem registros em `hub_leads` (`source in ('apollo_outbound','apollo')`), Apollo sem crédito não deve bloquear geração: drena stored leads primeiro, usando `orion_processed_firms` como ledger/dedupe (`hub:<id>`). Só cai para Apollo/Tavily quando o estoque armazenado acabar.
5. Cuidado com RLS/permissões: `hub_leads` tem policy por `app.workspace_id`; worker com `alma_service_account` precisa `set_config('app.workspace_id', ...)` e `GRANT SELECT`. Não use `FOR UPDATE SKIP LOCKED` sem privilégio de update, porque vira `permission denied for table hub_leads`; prefira sharding por instance + `ON CONFLICT` no ledger.
6. Depois de patchar worker systemd, verifica se o processo carregou o código novo. Se `systemctl stop`/SIGTERM fica preso por sleep handler, confirma PID; se necessário, mata o PID antigo e observa restart/exit esperado.
7. Verificação mínima: `py_compile`, diff do arquivo, status systemd, health HTTP, log tail com a causa legível e evidência de output real (lead no DB/brain, não só serviço running).
