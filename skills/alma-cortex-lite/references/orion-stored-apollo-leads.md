# ORION: processar leads Apollo já pagos em `hub_leads`

## Quando aciona
- ORION está vivo, mas sem entregar leads.
- Apollo retorna `insufficient credits`, mas Mario diz que não está extraindo leads novos.
- Existem contatos já carregados em `hub_leads` com `source in ('apollo_outbound', 'apollo')`.

## Lição operacional
Não concluir que ORION está bloqueado por Apollo sem crédito até separar:
1. **Live sourcing Apollo**: nova extração, consome crédito.
2. **Stored paid leads**: processamento de `hub_leads`, crédito já consumido.

ORION deve drenar `hub_leads` antes de qualquer fallback live Apollo/Tavily.

## Diagnóstico rápido
```bash
sudo -u postgres psql -d alma_agentic -P pager=off -c "
select count(*) as eligible_stored_paid_leads
from hub_leads h
where h.status='new'
  and h.email is not null and h.email<>''
  and h.source in ('apollo_outbound','apollo')
  and not exists (
    select 1 from orion_processed_firms p
    where p.workspace_id='aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa'::uuid
      and p.firm_apollo_id='hub:' || h.id::text
  );"
```

Verificar output real, não só heartbeat:
```bash
sudo -u postgres psql -d alma_agentic -P pager=off -c "
select count(*) filter (where created_at >= now() - interval '20 minutes') as leads_last_20m,
       count(*) filter (where created_at::date = current_date) as leads_today
from orion_leads;

select firm_apollo_id,status,instance_id,processed_at
from orion_processed_firms
where firm_apollo_id like 'hub:%'
order by processed_at desc nulls last
limit 10;"
```

## Hotfix pattern usado
- `agents/orion/db.py`: função `claim_stored_hub_lead(workspace_id, instance_id)`.
- Usa `orion_processed_firms` como ledger/dedupe com chave sintética `hub:<hub_leads.id>`.
- Usa `set_config('app.workspace_id', workspace_id, true)` por causa da policy RLS de `hub_leads`.
- `alma_service_account` precisa `GRANT SELECT ON hub_leads`.
- Evitar `FOR UPDATE SKIP LOCKED` se a role só tem `SELECT`; isso causa `permission denied for table hub_leads`. Em vez disso, shard por worker com `mod(abs(hashtext(h.id::text)), 5) = (instance_id - 1) % 5` e deixe o `ON CONFLICT` no ledger resolver corrida residual.
- `agents/orion/core.py`: `_pull_next_candidate()` deve tentar stored `hub_leads` primeiro; só depois live Apollo/Tavily.
- Para stored hub leads, não chamar Apollo `/people/match`; contato/e-mail já estão armazenados e pagos.
- `agents/orion/scoring_engine.py`: scoring precisa aceitar `decision_maker_name`, `decision_maker_email`, `decision_maker_title`, `industry`, `hub_source` vindos de `raw_jsonb` do stored lead.

## Validação esperada
- Logs: `claimed firm=<name> source=hub_leads`.
- Logs: `delivered lead orion_id=... firm=... score=...`.
- DB: `orion_leads` cresce hoje/últimos minutos.
- `orion_processed_firms` mostra `hub:<uuid>` com `delivered` ou `discarded`.

## Pitfall de interpretação
Tavily no limite pode aparecer como soft-fail de enriquecimento (`linkedin LIGHT`, `google_news`). Isso não deve bloquear entrega básica se o stored lead já tem nome/e-mail. Diferenciar enriquecimento degradado de sourcing bloqueado.
