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
- Domínio: getalmarev.com (NUNCA almarev.com)
- Outbound: Instantly, 4282 leads, warmup ativo

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
- LANCE pause/resume: ver `references/lance-systemd-pause.md`. Use quando Mario pedir pra pausar LANCE, quando `alma-lance.timer` reaparecer, ou quando `systemctl disable --now` travar.
