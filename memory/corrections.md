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

## 2026-06-25T22:58:14+00:00 | Instantly sender readiness
- O que fiz: interpretei contas com score alto no Instantly como usáveis para produção controlada.
- Correção: Tirando getslmarev/getalmarev e tryslmarev/tryalmarev, todas as contas estão aquecendo ainda. Vão ficar prontas na próxima terça.
- Lição: Para ALMA Rev Instantly, readiness operacional por domínio vence score bruto da API. Até terça, tratar só getalmarev.com e tryalmarev.com como domínios prontos; demais domínios permanecem warmup-only mesmo com score alto.
- Status: APPLIED

## 2026-06-25T23:00:53+00:00 | Instantly sender readiness
- O que fiz: mantive `tryalmarev.com` como domínio pronto depois da correção anterior.
- Correção: tryalmarev teve envios para spam esta semana; deixar aquecer até terça também.
- Lição: Para ALMA Rev Instantly, `tryalmarev.com` deve ficar warmup-only até terça quando houver spam recente no warmup, mesmo com score alto ou cold sends sem bounce.
- Status: APPLIED

## 2026-06-25T23:03:43+00:00 | Instantly sender readiness
- O que fiz: usei score e domínio como critérios principais de readiness.
- Correção: só habilitar contas que não tiveram spam nos últimos 7 dias.
- Lição: Para ALMA Rev Instantly, zero warmup spam nos últimos 7 dias é hard gate por mailbox antes de colocar em cold outbound.
- Status: APPLIED

## 2026-06-26T02:46:15Z | alma-brain freshness
- O que fiz: eu e outros agentes confundimos fontes antigas com estado vigente no brain e às vezes puxamos informação velha.
- Correção: “vc e o claude se confundem muito no brain... as vezes vcs puxam informcoes antigas”.
- Lição: Para qualquer resposta baseada no brain, primeiro estabelecer hierarquia temporal e canônica: STATUS/_CURRENT/business-plan canônico + decisões recentes + dado vivo quando operacional. Arquivo antigo só entra como histórico, nunca como verdade final sem checar `status`, `last_reviewed`, timestamp e conflito com fontes mais novas.
- Status: APPLIED

## 2026-06-26T02:51:58Z | project-status
- O que fiz: Tratei JT Shirts e mtmalls como projetos/workstreams ativos na lista de stack.
- Correção: "jtshirts e mtmalls estao encerrados"
- Lição: JT Shirts e mtmalls são workstreams encerrados; só tratar como ativos se Mario reabrir explicitamente.
- Status: APPLIED

## 2026-06-27T00:00:00Z | clarification
- O que fiz: interpretei a mensagem do Mario sem o contexto suficiente.
- Correção: "Eu acho q vc entendeu errado"
- Lição: quando a correção vier sem o ponto exato, parar e pedir a distinção específica antes de assumir qual parte está errada.
- Status: APPLIED

## 2026-06-27T00:00:00Z | lead factory flow
- O que fiz: associei o lead factory a verifier errado e tratei a etapa de verificação como genérica.
- Correção: "Lead factory: vc busca as contas no icp, rodarmos o mails.do e verificamos a conta"
- Lição: o fluxo canônico do Lead Factory é ICP search -> mails.do -> account verification.
- Status: APPLIED
## 2026-06-29T13:28:24+00:00 | research-double-check
- O que fiz: respondi sem explicitar a regra de devil's advocate e double-check do número do mercado.
- Correção: Mario quer que eu revise as informações que ele passa, pesquise, valide se faz sentido e o corrija quando não fizer.
- Lição: antes de aceitar uma tese operacional, fazer checagem externa e contradizer a hipótese se o mercado não sustentar.
- Status: APPLIED

## 2026-06-29T13:33:19+00:00 | review-numbers-verdict
- O que fiz: respondi com faixas de benchmark, mas não deixei o veredito explícito em formato agree/disagree.
- Correção: Mario quer que eu revise o número e diga claramente se concordo ou não, sempre como devil's advocate.
- Lição: toda tese numérica precisa sair com veredito explícito + motivo + próximo passo.
- Status: APPLIED


## 2026-06-29T16:23:18+00:00 | instantly-ready-count-mixup
- O que fiz: misturei o recorte do inventário live completo com o recorte das screenshots e soltei o número errado de READY.
- Correção: Mario disse que continua errado.
- Lição: quando citar READY/WATCH/HOLD, separar explicitamente snapshot global versus subset de imagens.
- Status: APPLIED

## 2026-06-30T00:00:00Z | multi-agent-ops-status
- O que fiz: tratei a operação como se eu respondesse por um único agente e descrevi o dia em termos genéricos.
- Correção: "você hoje responde por vários agentes" e o recorte correto é prioridades, atividades, metas e o que a gente conclui hoje e na semana.
- Lição: quando falar de rotina diária, enquadrar o trabalho como cockpit multi-agente e resumir por prioridades, atividades, metas e entregas do dia/semana, sempre por agente.
- Status: APPLIED
