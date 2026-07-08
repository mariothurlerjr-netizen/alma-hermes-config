---
name: alma-heartbeat
description: Ciclo proativo. Aciona via cron a cada 30min ou manualmente. Lê HEARTBEAT.md e executa checklist. Anti-spam, nunca repete aviso em 4h.
license: proprietary
---

# Heartbeat workflow

1. Lê `/root/.hermes/HEARTBEAT.md` como fonte autoritativa do ciclo.
2. Lê `/home/almarev/brain/agents/default/heartbeat-state.md` antes de agir.
3. Executa somente os itens da janela atual que ainda não constem como executados no state.
4. Registra todo resultado em arquivo/state; não depende de memória conversacional.
5. Se precisar de contexto operacional, lê `/home/almarev/brain/STATUS.md` e arquivos indicados pelo próprio `HEARTBEAT.md`.
6. Se detectar desvio relevante fora da janela atual, registra em arquivo para SHIELD/ORCHESTRATOR; não pageia Mario diretamente.
7. Resposta final do cron default deve ser exatamente `[SILENT]`, sem relatório, prefixo, sufixo ou combinação com conteúdo.

## Silêncio é obrigatório no default
O perfil default nunca manda mensagem proativa para Mario. Heartbeats e rituais continuam rodando e escrevendo em arquivo, mas a entrega final do job default é sempre `[SILENT]`. Spam de heartbeat é pior que silêncio.

## Critério de ação
Executar apenas o que está explicitamente na janela atual do `HEARTBEAT.md` e ainda não foi marcado no `heartbeat-state.md`. Ação fora da janela vira anotação em arquivo, não notificação.

## Briefing diário 07:15
Quando o heartbeat roda como check-in da manhã, ele só pode entregar briefing se o `JOB.md`/mensagem do cron permitir entrega. Se a instrução ativa disser `SILENT`, `responda sempre [SILENT]`, ou que o perfil default nunca manda mensagem proativa, essa instrução vence e a resposta final deve ser exatamente `[SILENT]`. Quando permitido, entregar um briefing operacional em PT-BR, direto, a menos que não haja absolutamente nada novo. Protocolo:

1. Ler `JOB.md` no diretório do Hermes se existir e obedecer regras do job.
2. Ler `/home/almarev/brain/STATUS.md`, `/home/almarev/brain/02-alma-rev/_CURRENT.md` e `/home/almarev/brain/agents/default/heartbeat-state.md` se acessíveis.
3. Separar métrica validada de sinal/anotação. Se live state não foi lido, dizer explicitamente "dado não validado".
4. Se houver estado recente de Instantly/outbound, incluir estado, bloqueio e próximo movimento.
5. Consolidar todas as pendências do Mario em uma única lista numerada, resolvível por palavra ou resposta curta; não espalhar pendências em blocos por área.
6. Usar o formato: resumo executivo; prioridades de hoje; atividades por área/agente com estado/meta/métrica/bloqueio/condição de pausa; fila única do Mario; o que conclui hoje; o que conclui na semana.

## Support files
- `references/default-heartbeat-instantly-audit.md` — padrão operacional para o default heartbeat auditar Instantly silenciosamente, separar ALMAREV-V3/legacy/Local, atualizar state e registrar desvios em SHIELD sem pagear Mario.

## Pitfalls conhecidos
- Em heartbeat/status escrito para arquivo, não despejar saída crua de `psql -F '|'` em métricas compostas. Transformar `5|2` em rótulo legível, ex.: `5 total / 2 interested`, `86 total / 0 email-attributed`. Esses arquivos viram insumo de ORCHESTRATOR/SHIELD, então separador bruto vira ambiguidade operacional.
- Na VPS do Mario, `hermes-gateway` roda como user service de root. Validar com `hermes gateway status` ou `XDG_RUNTIME_DIR=/run/user/0 systemctl --user is-active hermes-gateway.service`. `systemctl is-active hermes-gateway.service` como system unit pode retornar inactive/absent e não significa gateway down.
- Quando Mario pedir para "pausar o alarme" ou reclamar de ruído, não olhe só `hermes cron list`: também inspecione o crontab (`/root/run-heartbeat.sh`) e diferencie watchdog específico (ex: `orion-watchdog`) de heartbeat geral. Se o watchdog já estiver pausado, o alarme pode estar vindo do checklist do heartbeat, então ajuste/silencie a condição ruidosa em vez de reportar "já está pausado".
- Para ORION/Apollo quota: se Mario disser que quer "refresh/use it" e houver uma conta Apollo com créditos, trate como rota de recuperação ativa. Próximo passo é trocar `APOLLO_API_KEY` de forma secret-safe, reiniciar workers e validar logs, não repetir alerta de quota.
