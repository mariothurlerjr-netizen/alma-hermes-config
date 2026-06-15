# ALMA Local: callback-only dialer mode

Contexto aprendido em 2026-06-15: quando Mario pausa a estratégia da CLAIRE Local por mudança de abordagem, isso bloqueia discagem da base fria/existente, mas não deve bloquear follow-up de callback quando a pessoa já aceitou ser chamada de volta.

## Regra operacional

- Bloqueio de estratégia nova: não discar base fria/existing leads sem warm-trigger/disclosure aprovado.
- Exceção permitida: callbacks explicitamente aceitos pelo prospect continuam válidos.
- O filtro correto para callback aceito é `local_call_follow_ups.kind='callback'`, `status='pending'`, `due_at <= now()` ou `due_at IS NULL`, ligado ao `lead_id`.
- Não exigir `local_leads.status IN ('new','retry')` para callback aceito. Leads `completed` podem ter pedido callback e ainda precisam ser elegíveis.
- Continuar respeitando DNC, `calling`, janela local, max attempts, phone presente e segmentos/timezones ativos.

## Implementação aplicada

Arquivo ativo do dialer Local: `/home/almarev/alma-local/dialer/orchestrator.py`.

Patch de referência:
- `CALLBACK_ONLY` default true via env `ALMA_LOCAL_CALLBACK_ONLY`, opt-out com `0/false/no`.
- `_get_eligible_leads()` em callback-only faz join com `local_call_follow_ups` e seleciona só callbacks pendentes/vencidos.
- `_update_result()` marca callback follow-up como `done` quando a chamada conecta.

## Verificação mínima

- Rodar com venv do Agentic, não Python cru:
  `cd /home/almarev/alma-local/dialer && /home/almarev/agentic/.venv/bin/python -m py_compile orchestrator.py && /home/almarev/agentic/.venv/bin/python orchestrator.py --dry-run`
- Se dry-run mostrar 0, checar se é janela local, não filtro errado.
- Query útil:
  `SELECT f.id, f.due_at, l.business_name, l.status, l.callback_at FROM local_call_follow_ups f JOIN local_leads l ON l.id=f.lead_id WHERE f.kind='callback' AND f.status='pending' ORDER BY f.due_at;`

## Pitfall

Não desligar `alma-claire-intents` como reação automática se a intenção for preservar follow-ups aceitos. O risco é ele re-armar base antiga, mas também é a rota que descobre callbacks reais em transcrições. Se a estratégia estiver em revisão, preferir deixar discagem callback-only e decidir separadamente se extração de intents deve rodar ou ficar pausada temporariamente.