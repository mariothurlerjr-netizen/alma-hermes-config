---
name: alma-context
description: Carrega contexto Mário + ALMA. Aciona sempre no primeiro turn e quando menção a ALMA Rev, ALMA Agentic, agentes, JT Shirts, job search, cliente ou prospect.
license: proprietary
---

# Contexto ALMA

## Stack
- VPS root, repos ativos em `/home/almarev/agentic` (alma-agentic), `/root/alma-aios` (legacy), almarev-site, hermes-agent, brain-mcp
- `cortex` FastAPI na porta `8002` (não confundir com `8001`, legado jt-sdr)
- Workspaces ativos: ALMA Rev, Vet Agentic
- Workstreams encerrados: JT Shirts e mtmalls. Não tratar como projetos ativos, clientes ativos, labs ativos, ou próximos passos operacionais sem Mario reabrir explicitamente.
- Model routing: veja `references/model-routing.md` quando o usuário perguntar sobre usar modelos mais leves para conversa e modelos mais fortes para execução.

## ALMA Rev produtos
- Estado canônico muda rápido: antes de responder sobre positioning, pricing, ICP, funnel ou oferta ALMA Rev, ler `02-alma-rev/_CURRENT.md` no brain. Se houver conflito, o brain vence este resumo.
- Current baseline 2026-06-17: ALMA Rev é Growth Operation para US professional-services firms, operator/done-for-you, Diagnostic US$2K + Retainer US$4K/mo, tecnologia backstage.
- Funnel atual: cold email com CTA AURA → reunião usando resultado AURA → VERA paid diagnostic → implementação/retainer via Hub agents.
- Domínios: `getalmarev.com` é usado para inbox/outbound; `almarev.com` é usado em assets públicos/campanhas como `/aura` e `/blog`. Não corrigir automaticamente um para o outro sem checar contexto.
- Separação de sender ALMA Rev: `mario@almarev.com` fica para canal owned/warm, newsletter, digest e relacionamento com quem já interagiu com AURA/conteúdo. Cold/outbound deve usar contas aquecidas `@getalmarev.com`, especialmente `mario@getalmarev.com`, quando estiverem prontas.
- Outbound: Instantly with campaigns `AlmaREV Launch`; status operational via API v2 + Cortex AURA cross-check.
- Pitfall Instantly: “contas ativas” pode aparecer como contagem de **leads em campanhas ativas**, não mailboxes. Validar separando `/campaigns/analytics` (`leads_count`, `contacted_count`, `emails_sent_count`) de `/accounts` e do painel Cortex `/api/agenda/email-capacity`.
- Deliverability / daily outbound ops: when Mario falar de volume diário, aquecimento, spam, contas prontas ou “usar todos os leads”, seguir `references/alma-rev-deliverability-daily-ops.md`.
- Email base hygiene: use `references/email-verification.md` and `leadgen-verification-ops`. Preferred verifier can change by operator decision; current direction is MillionVerifier-first for lead hygiene when Mario has switched to it. Treat older verifier stacks as retired unless Mario explicitly reopens them. Keep provider/account failures loud so billing/auth problems are visible.- Daily operating loop: when Mario pedir o trabalho do dia, usar os 4 blocos de `references/daily-operating-loop.md`, começando por Instantly/inbox, depois balanceamento ICP, depois completude/enrichment, e por fim market/content watch. Quando houver conteúdo externo para mineração de GTM, incluir vídeos do Alfredo Soares como fonte recorrente de ideias, não como one-off.


## ALMA Agentic
- `/home/almarev/agentic` é o stack técnico multi-tenant em produção/operacional.
- ORION lead-gen roda como `alma-orion@1..5.service`, gera lead-base e dashboard em `app.almarev.com/dashboard/leads`.
- LANCE (text SDR), CLAIRE (voice SDR), Clara e AURA existem como agentes/workstreams.

## Encerrados / histórico
- JT Shirts: encerrado. Contexto histórico: US uniform company, Fractional Head of Sales, ICP restaurants/franchises, field service, multi-site healthcare, stack Snov.io/Instantly/Python Google Places API. Usar só para arqueologia, migração, cleanup ou análise histórica.
- mtmalls: encerrado. Usar só para arqueologia, cleanup ou referência histórica.
- Pitfall: se STATUS.md, brain antigo, memória antiga ou resumo de stack listar JT Shirts/mtmalls como ativos, tratar como stale até haver decisão nova explícita do Mario reabrindo.

## Job Search ativo
- Posicionamento: Sales Director / VP Sales / CRO. NÃO General Manager.
- Estratégia: passa em todo processo, decide no offer.
- LinkedIn entra como filtro de vagas e triagem de oportunidades, não como fonte genérica de browsing contínuo.
- Instagram entra mais como canal de monitoramento de campanha do que como superfície operacional primária.
- Para reativação de integrações Composio com toolkit filtrado, ver `references/composio-app-priorities.md`.

## Regras de stack
- OAuth SEMPRE preferido. API key só como último recurso.
- Na VPS ALMA atual não assumir Unix user `almarev`; serviços systemd rodam majoritariamente como root/dev. Antes de usar `User=almarev` em unit file, verificar se o usuário existe ou criar explicitamente.

## Runbooks operacionais
- Hub Team roster / agent identity correction: quando Mario apontar texto errado no Hub (`/team`, cards de AI Employees, Agentic Lab, org chart), trate como correção de identidade operacional, não só copy. Para IRIS, o papel canônico é métricas/dashboard/analytics/digest/anomaly detection, não follow-up/scheduling/closer. Corrigir tanto runtime DB (`team_agents`, `agent_goals`) quanto fontes persistentes (`migrations/0041_team_cockpit.sql`, `migrations/0042_agent_descriptions.sql`, `hub/src/lib/agents.ts`, `cortex/team.py`, SOUL/template, prompt decomposer quando aplicável), depois `npm run build`, reiniciar `alma-cortex`/`alma-hub` e validar via `/team/agents` e `/api/cortex/team/agents`.
- Agent/systemd health audit + pause-all: ver `references/agent-systemd-health-audit.md`. Use quando Mario perguntar se os agentes estão instalados, vivos, “rotating”, rodando na VPS, quando houver falha em timers/services ALMA, ou quando pedir pra pausar/parar todos os agentes. Sempre cruzar Brain `STATUS.md` com systemd, timers, journal, failed units, processos soltos, Hermes cron e health endpoints.
- Instantly outbound status: ver `references/instantly-outbound-status.md`. Use quando Mario perguntar se os emails/campanhas foram enviados, se ainda falta disparo, se houve reply/bounce/open, ou se alguém fez AURA Assessment.
- Instantly capacity maximization: ver `references/instantly-capacity-maximization.md`. Use quando Mario perguntar sobre plano Instantly, cap de leads, volume diário, capacidade das contas, tamanho de cadência, campanhas pausadas/ativas, ou se estamos maximizando outbound. Sempre separar lead cap workspace-wide, capacidade de mailboxes, daily_limit de campanhas e número real de steps na sequência.
- `Telegram agent topics`: ver `references/telegram-agent-topics.md`. Use quando Mario perguntar por que agentes como IRIS/MUSE não aparecem no canal/grupo `Alma AIOS`, ou pedir pra criar lanes/tópicos de agentes no Telegram. Primeiro verificar targets do gateway; distinguir canal vs supergrupo/forum; bot precisa ser admin com permissão de gerenciar tópicos.
- `Telegram inter-bot conversations`: ver `references/telegram-interbot-conversations.md`. Use quando Mario pedir para bots/agentes ALMA conversarem entre si em grupo Telegram. Validar gateways, `getMe`, `getChatMember`, `TELEGRAM_ALLOW_BOTS=mentions`, bot IDs na allowlist e caveat de `can_manage_topics`.
- `Granola meeting context`: ver `references/granola-meeting-notes.md`. Use quando Mario pedir para ler recaps de reuniões, extrair follow-ups, ou reutilizar contexto de calls; trate como insumo recorrente, não como verdade canônica se conflitar com STATUS/_CURRENT/decisions.
- `HERALD activity notification`: ver `references/herald-activity-notification.md`. Use quando o problema for mensagem entrando mas o alerta não disparar, ou quando for preciso garantir notificação em tempo real com fallback de watchdog.
- `Hermes Telegram agent fleet`: ver `references/hermes-telegram-agent-fleet.md`. Use quando Mario pedir migração de agentes Claude/LangGraph/OpenClaw para Hermes profiles/bots, criação de um bot por agente, troca de token do BotFather, ou start/validação de gateways por profile.

- LANCE pause/resume: ver `references/lance-systemd-pause.md`. Use quando Mario pedir pra pausar LANCE, quando `alma-lance.timer` reaparecer, ou quando `systemctl disable --now` travar.
- ORION pause/resume: ver `references/orion-systemd-pause.md`. Use quando Mario pedir pra parar geração de leads, pausar ORION ou retomar lead-gen. Sequência correta: pausar `orion-watchdog`, parar/desabilitar `alma-orion@1..5`, limpar `failed` se necessário, verificar inactive/disabled/sem processo.
- CLAIRE dialer operations: ver `references/claire-dialer-operations.md`. Use quando Mario pedir pra pausar/retomar dialer, fim de semana, ligações da CLAIRE ou checar se ela está discando. Regra padrão: dialer só segunda a sexta; pausar outbound/reminder sem desligar polling/QA/coach.
- ALMA Local CRM outcomes: ver `references/local-crm-outcomes.md`. Use quando Mario perguntar o significado de outcomes/tags/stages como `CHECK_SENT`, `BOOKED`, `CALLBACK`, ou quando uma notificação de call precisar ser interpretada.
- LUNA content digest: ver `references/luna-content-digest.md`. Use quando Mario perguntar que email semanal/newsletter da ALMA Rev chegou, se foi enviado pra outras pessoas, quem recebeu, ou se é Instantly/outbound. Validar via `alma-content-digest.timer`, journal e `content_digest_log`; separar emails reais externos de internos/testes.
