---
name: alma-heartbeat
description: Ciclo proativo. Aciona via cron a cada 30min ou manualmente. Lê HEARTBEAT.md e executa checklist. Anti-spam, nunca repete aviso em 4h.
license: proprietary
---

# Heartbeat workflow

1. Lê `/root/.hermes/HEARTBEAT.md` (checklist do que rastrear)
2. Lê `/root/.hermes/memory/corrections.md` (últimas 10)
3. Lê `/root/.hermes/memory/patterns.md` (flag SWITCH_STRATEGY)
4. Lê `/home/almarev/brain/STATUS.md` (estado real do sistema, auto-atualizado 24h)
5. Lê `/root/.hermes/memory/heartbeat-log.md` (últimas 8, anti-spam)
6. Para cada item do checklist, decide ação
7. Se aciona, manda 1 mensagem objetiva no Telegram
8. SEMPRE escreve em `heartbeat-log.md`

## Silêncio é positivo
Não manda "tudo ok". Spam de heartbeat é pior que silêncio.

## Critério de envio
Novo (não avisou em 4h) + acionável (sugestão concreta) + Mario age em menos de 5min.

## Briefing diário 07:15
Quando o heartbeat roda como check-in da manhã, ele não é anti-spam silencioso comum: entregar um briefing operacional em PT-BR, direto, a menos que não haja absolutamente nada novo. Protocolo:

1. Ler `JOB.md` no diretório do Hermes se existir e obedecer regras do job.
2. Ler `/home/almarev/brain/STATUS.md`, `/home/almarev/brain/02-alma-rev/_CURRENT.md` e `/home/almarev/brain/agents/default/heartbeat-state.md` se acessíveis.
3. Separar métrica validada de sinal/anotação. Se live state não foi lido, dizer explicitamente "dado não validado".
4. Se houver estado recente de Instantly/outbound, incluir estado, bloqueio e próximo movimento.
5. Consolidar todas as pendências do Mario em uma única lista numerada, resolvível por palavra ou resposta curta; não espalhar pendências em blocos por área.
6. Usar o formato: resumo executivo; prioridades de hoje; atividades por área/agente com estado/meta/métrica/bloqueio/condição de pausa; fila única do Mario; o que conclui hoje; o que conclui na semana.

## Pitfalls conhecidos
- Na VPS do Mario, `hermes-gateway` roda como user service de root. Validar com `hermes gateway status` ou `XDG_RUNTIME_DIR=/run/user/0 systemctl --user is-active hermes-gateway.service`. `systemctl is-active hermes-gateway.service` como system unit pode retornar inactive/absent e não significa gateway down.
- Quando Mario pedir para "pausar o alarme" ou reclamar de ruído, não olhe só `hermes cron list`: também inspecione o crontab (`/root/run-heartbeat.sh`) e diferencie watchdog específico (ex: `orion-watchdog`) de heartbeat geral. Se o watchdog já estiver pausado, o alarme pode estar vindo do checklist do heartbeat, então ajuste/silencie a condição ruidosa em vez de reportar "já está pausado".
- Para ORION/Apollo quota: se Mario disser que quer "refresh/use it" e houver uma conta Apollo com créditos, trate como rota de recuperação ativa. Próximo passo é trocar `APOLLO_API_KEY` de forma secret-safe, reiniciar workers e validar logs, não repetir alerta de quota.
