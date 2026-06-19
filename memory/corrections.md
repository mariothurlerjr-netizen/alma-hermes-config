# Corrections from Mário

Formato:
## YYYY-MM-DD HH:MM | <skill ou contexto>
- O que fiz: <ação original>
- Correção: <feedback literal>
- Lição: <regra>
- Status: APPLIED | NOT_APPLIED

---

## 2026-05-23 12:24 UTC | ORION
- O que fiz: interpretei o bloqueio Apollo como falta de crédito para extrair leads novos.
- Correção: “não estou extraindo novos leads do apollo, mas usando os que já extrai e já consumi créditos”.
- Lição: ORION deve drenar `hub_leads`/leads Apollo já pagos antes de qualquer sourcing live; erro de crédito Apollo não deve bloquear processamento de contatos já armazenados.
- Status: APPLIED

[2026-05-28T16:53:48Z] Mario correction: I failed a shorthand reminder request ('msg hj as 8') by not creating a cron job and leaving it unscheduled after asking for clarification. SWITCH_STRATEGY: for noisy reminder requests, create the cron job when time/content are inferable; if content is missing, explicitly say no reminder was scheduled and ask for the exact text.

[2026-05-28T16:54:24Z] Mario correction: I incorrectly told Mario there was no message job after cron list showed `lembrete-reuniao-alisa-rogerio-video` scheduled for 2026-05-29 11:00 UTC, equivalent to 08:00 Brazil. SWITCH_STRATEGY: when checking missed reminders, interpret existing cron jobs by timezone and inspect prompt/script before saying no job exists.

[2026-06-19T18:25:00Z] Mario correction: I said 77 leads that received the 3rd email should be treated as sequence-closed/cooldown. Correction: after the 3rd email, the agreed next step is a connection/call via CLAIRE for eligible leads. SWITCH_STRATEGY: for ALMA Rev cadence status, treat post-3rd-email as call-readiness/CLAIRE queue, not recycle/cooldown by default.
