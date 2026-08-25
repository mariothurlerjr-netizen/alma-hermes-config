# HEARTBEAT — DEFAULT (crons legados, sempre silencioso)

O cron `default-heartbeat` te acorda 2x/dia (manha e inicio da noite). Regras:
1. Execute APENAS itens na janela atual que ainda nao rodaram hoje (registro em
   /home/almarev/brain/agents/default/heartbeat-state.md, secao "## Heartbeat runs").
2. Voce NUNCA manda mensagem proativa pro Mario. Responda sempre exatamente [SILENT].
3. Desvio relevante: registre em /home/almarev/brain/agents/shield/inbox.md (o SHIELD pageia se for critico).
4. A operacao Instantly/Maildoso foi encerrada por Mario em 2026-08-25. Nao consultar campanhas, contas, cap, warmup, DNS auth, seed, placement ou deliverability; nao gerar status nem alertas dessa infraestrutura.

## Janela 07:00-08:00 — CHECKIN DA MANHA (1x/dia)

Check-in da manha (07:15 America/Sao_Paulo) - SILENCIOSO desde 2026-07-02: o briefing pro Mario e do JARVIS; o default NUNCA manda mensagem proativa. Leia /home/almarev/agentic-v2/hermes-agent/JOB.md se existir, /home/almarev/brain/STATUS.md, /home/almarev/brain/02-alma-rev/_CURRENT.md e /home/almarev/brain/agents/default/heartbeat-state.md. Escreva o briefing interno sem status de Instantly/Maildoso ou sender health em /home/almarev/brain/agents/default/checkin-manha.md (sobrescreva) e atualize o heartbeat-state. Depois responda exatamente [SILENT].

## STATUS INSTANTLY MANHA, ENCERRADO

Nao executar. Nao consultar nem escrever status. Nao registrar desvio no inbox do SHIELD.

## Janela 18:00-18:25 — CHECKIN DA NOITE (1x/dia)

Check-in da noite (18:30 America/Sao_Paulo) - SILENCIOSO desde 2026-07-02: a consolidacao pro Mario e do ORCHESTRATOR; o default NUNCA manda mensagem proativa. Compare o realizado de hoje com o plano da manha (heartbeat-state em /home/almarev/brain/agents/default/heartbeat-state.md): o que fechou, o que travou e por que, prioridade de amanha. Escreva em /home/almarev/brain/agents/default/checkin-noite.md (sobrescreva) e atualize o heartbeat-state. Depois responda exatamente [SILENT].

## STATUS INSTANTLY NOITE, ENCERRADO

Nao executar. Nao consultar nem escrever status. Nao registrar desvio no inbox do SHIELD.
