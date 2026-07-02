# HEARTBEAT — DEFAULT (crons legados, sempre silencioso)

O cron `default-heartbeat` te acorda 2x/dia (manha e inicio da noite). Regras:
1. Execute APENAS itens na janela atual que ainda nao rodaram hoje (registro em
   /home/almarev/brain/agents/default/heartbeat-state.md, secao "## Heartbeat runs").
2. Voce NUNCA manda mensagem proativa pro Mario. Responda sempre exatamente [SILENT].
3. Desvio relevante: registre em /home/almarev/brain/agents/shield/inbox.md (o SHIELD pageia se for critico).
4. Referencia: o audit noturno (alma-rev-campaign-audit, 23:00) segue como cron de script separado, sem agente.

## Janela 07:00-08:00 — CHECKIN DA MANHA (1x/dia)

Check-in da manha (07:15 America/Sao_Paulo) - SILENCIOSO desde 2026-07-02: o briefing pro Mario e do JARVIS; o default NUNCA manda mensagem proativa. Faca o trabalho normalmente: leia /home/almarev/agentic-v2/hermes-agent/JOB.md se existir, /home/almarev/brain/STATUS.md, /home/almarev/brain/02-alma-rev/_CURRENT.md e /home/almarev/brain/agents/default/heartbeat-state.md. Escreva o briefing completo (resumo executivo, prioridades de hoje, estado outbound/Instantly) em /home/almarev/brain/agents/default/checkin-manha.md (sobrescreva) e atualize o heartbeat-state. Depois responda exatamente [SILENT].

## Janela 07:00-08:00 — STATUS INSTANTLY MANHA (1x/dia)

Status Instantly da manha - SILENCIOSO desde 2026-07-02 (default NUNCA manda mensagem proativa; desvios chegam ao Mario via SHIELD ou consolidacao do ORCHESTRATOR). Leia metricas live (sends, replies, bounces, spam, warmup) e compare com o baseline em /home/almarev/brain/agents/default/heartbeat-state.md. Escreva o status (com numeros; desvio relevante = destaque com numero e acao sugerida) em /home/almarev/brain/agents/default/status-instantly.md (sobrescreva) e atualize o heartbeat-state. Se houver desvio relevante, registre tambem 1 linha em /home/almarev/brain/agents/shield/inbox.md pra o SHIELD avaliar paging. Depois responda exatamente [SILENT].

## Janela 18:00-18:25 — CHECKIN DA NOITE (1x/dia)

Check-in da noite (18:30 America/Sao_Paulo) - SILENCIOSO desde 2026-07-02: a consolidacao pro Mario e do ORCHESTRATOR; o default NUNCA manda mensagem proativa. Compare o realizado de hoje com o plano da manha (heartbeat-state em /home/almarev/brain/agents/default/heartbeat-state.md): o que fechou, o que travou e por que, prioridade de amanha. Escreva em /home/almarev/brain/agents/default/checkin-noite.md (sobrescreva) e atualize o heartbeat-state. Depois responda exatamente [SILENT].

## Janela 18:00-18:25 — STATUS INSTANTLY NOITE (1x/dia)

Status Instantly da noite - SILENCIOSO desde 2026-07-02 (default NUNCA manda mensagem proativa; desvios chegam ao Mario via SHIELD ou consolidacao do ORCHESTRATOR). Mesmo procedimento da manha: metricas live vs baseline do heartbeat-state, escreva em /home/almarev/brain/agents/default/status-instantly.md (sobrescreva, secao noite) e atualize o heartbeat-state. Desvio relevante = registre 1 linha em /home/almarev/brain/agents/shield/inbox.md. Depois responda exatamente [SILENT].
