---
name: alma-context
description: Carrega contexto Mário + ALMA. Aciona sempre no primeiro turn e quando menção a ALMA Rev, ALMA Agentic, agentes, JT Shirts, job search, cliente ou prospect.
license: proprietary
---

# Contexto ALMA

## Stack
- VPS root, repos ativos em `/home/almarev/agentic` (alma-agentic), `/root/alma-aios` (legacy), almarev-site, mtmalls, hermes-agent, brain-mcp
- `cortex` FastAPI na porta `8002` (não confundir com `8001`, jt-sdr)
- Workspaces: ALMA Rev, Vet Agentic, JT Shirts

## ALMA Rev produtos
- Revenue Leak Diagnostic Standard: $7.5K / 4 semanas
- Revenue Leak Diagnostic Lite: $3.5K / 2 semanas
- Fractional Revenue Leadership
- ICP: SaaS B2B + agencies + professional services, $500K-$5M ARR
- Domínios: `getalmarev.com` é usado para inbox/outbound; `almarev.com` é usado em assets públicos/campanhas como `/aura` e `/blog`. Não corrigir automaticamente um para o outro sem checar contexto.
- Outbound: Instantly com campanhas `AlmaREV Launch`; status operacional via API v2 + Cortex AURA cross-check.

## ALMA Agentic
- `/home/almarev/agentic` é o stack técnico multi-tenant em produção/operacional.
- ORION lead-gen roda como `alma-orion@1..5.service`, gera lead-base e dashboard em `app.almarev.com/dashboard/leads`.
- LANCE (text SDR), CLAIRE (voice SDR), Clara e AURA existem como agentes/workstreams.

## JT Shirts (engagement paralelo)
- US uniform company, Fractional Head of Sales
- ICP: restaurants/franchises, field service, multi-site healthcare
- Stack: Snov.io, Instantly, Python Google Places API

## Job Search ativo
- Posicionamento: Sales Director / VP Sales / CRO. NÃO General Manager.
- Estratégia: passa em todo processo, decide no offer

## Regras de stack
- OAuth SEMPRE preferido. API key só como último recurso.

## Runbooks operacionais
- Agent/systemd health audit: ver `references/agent-systemd-health-audit.md`. Use quando Mario perguntar se os agentes estão instalados, vivos, “rotating”, rodando na VPS, ou quando houver falha em timers/services ALMA. Sempre cruzar Brain `STATUS.md` com systemd, timers, journal, failed units e health endpoints.
- Instantly outbound status: ver `references/instantly-outbound-status.md`. Use quando Mario perguntar se os emails/campanhas foram enviados, se ainda falta disparo, se houve reply/bounce/open, ou se alguém fez AURA Assessment.
- LANCE pause/resume: ver `references/lance-systemd-pause.md`. Use quando Mario pedir pra pausar LANCE, quando `alma-lance.timer` reaparecer, ou quando `systemctl disable --now` travar.
- ORION pause/resume: ver `references/orion-systemd-pause.md`. Use quando Mario pedir pra parar/pausar/congelar geração de leads. Ordem crítica: pausar `orion-watchdog` antes de parar `alma-orion@1..5`, depois verificar cron, systemd enabled/active e process table.
- ORION pause/resume: ver `references/orion-systemd-pause.md`. Use quando Mario pedir pra parar geração de leads, pausar ORION ou retomar lead-gen. Sequência correta: pausar `orion-watchdog`, parar/desabilitar `alma-orion@1..5`, limpar `failed` se necessário, verificar inactive/disabled/sem processo.
