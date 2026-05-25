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

## Pitfalls conhecidos
- Na VPS do Mario, `hermes-gateway` roda como user service de root. Validar com `hermes gateway status` ou `XDG_RUNTIME_DIR=/run/user/0 systemctl --user is-active hermes-gateway.service`. `systemctl is-active hermes-gateway.service` como system unit pode retornar inactive/absent e não significa gateway down.
