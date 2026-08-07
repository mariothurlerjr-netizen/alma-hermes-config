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
