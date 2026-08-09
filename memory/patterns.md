# Patterns detected

## <task type>
- Falhas: N consecutivas
- Abordagem que falhou: <descrição>
- Hipótese: <por que>
- Próxima abordagem: <nova estratégia>
- Flag: SWITCH_STRATEGY | RESOLVED
- Data: YYYY-MM-DD

---

## heartbeat revenue-open-loop stale
- Falhas: 3+ heartbeats consecutivos com mesmo sinal operacional.
- Abordagem que falhou: registrar só em SHIELD/state que há 29 replies/2 interested, 0 discoveries, campanhas pausadas e seed-test ausente.
- Hipótese: o sinal deixou de ser deliverability e virou follow-through comercial travado, sem dono claro para converter interesse em discovery ou declarar encerramento do sprint.
- Próxima abordagem: quando o mesmo bloqueio aparecer por 48h+ sem delta, resumir como open loop de receita e pedir decisão curta: reativar follow-through manual, fechar pós-sprint, ou arquivar/test-excluir replies.
- Flag: SWITCH_STRATEGY
- Data: 2026-08-07

## lance-standup-artifacts-late
- Falhas: LANCE registrou artifacts `RECUPERADO_TARDE` e status diz deadline 08h perdido historicamente.
- Abordagem que falhou: depender de fallback depois da janela STANDUP para gerar health/replies/placement, mantendo o agente OFF_TRACK sem bloquear a rotina cedo.
- Hipótese: o watchdog/check-in não está tratando perda de SLA como falha própria do agente, só como recuperação de arquivo.
- Próxima abordagem: quando artifacts LANCE chegarem depois de 08h, classificar como falha de SLA operacional e abrir correção de scheduler/gate, não só status informativo.
- Flag: SWITCH_STRATEGY
- Data: 2026-08-07

## orion-status-active-but-units-inactive
- Falhas: STATUS.md mostra ORION ativo por atividade de brain/log, mas systemd live pode estar com `alma-orion@1..5` e `alma-orion-watchdog.timer` inactive.
- Abordagem que falhou: confiar no status agregado de activity 7d como prova de produção diária ativa.
- Hipótese: atividade de agente/arquivo mascara ausência de geração real, e a fábrica diária fica sem dono quando os workers/timer estão parados.
- Próxima abordagem: em heartbeat com ORION ativo no STATUS, validar systemd + último run lead factory; se produção diária estiver zero/stale, reportar como open loop operacional ou marcar pausado explicitamente.
- Flag: SWITCH_STRATEGY
- Data: 2026-08-09
