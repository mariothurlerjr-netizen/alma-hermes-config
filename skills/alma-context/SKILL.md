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
- Pitfall de recorte diário: quando Mario pedir "hoje de quais contas e qtas por conta", usar `references/instantly-sent-today-by-account.md`, ler `/emails` por `campaign_id` + `email_type=sent`, agrupar por `eaccount` ou `from_address_email`, e respeitar o timezone da campanha.
- Deliverability / daily outbound ops: when Mario falar de volume diário, aquecimento, spam, contas prontas ou “usar todos os leads”, seguir `references/alma-rev-deliverability-daily-ops.md`.
- Ramp-up pitfall: newly added inboxes start at 5 sends/day, while warmed mailboxes can run at the normal steady-state ceiling (currently 40/day in the live stack unless a newer operational override is active).
- Email base hygiene: use `references/email-verification.md` and `leadgen-verification-ops`. Preferred verifier can change by operator decision; current direction is mails.so-first for lead hygiene when Mario switches to it. Treat older verifier stacks as retired unless Mario explicitly reopens them.
- Lead search / ORION verification details: see `references/orion-lead-factory-verification.md`. Keep provider/account failures loud so billing/auth problems are visible.
- Daily operating loop: when Mario pedir o trabalho do dia, usar os 4 blocos de `references/daily-operating-loop.md`, começando por Instantly/inbox, depois balanceamento ICP, depois completude/enrichment, e por fim market/content watch. Quando houver conteúdo externo para mineração de GTM, incluir vídeos do Alfredo Soares como fonte recorrente de ideias, não como one-off.
- Broad business-success mandate: when Mario says algo como “make ALMA successful”, “arruma o negócio”, or gives a global revenue command, use `references/alma-business-success-command.md`. Treat it as execution mandate: brain first, live-state verification, rank revenue gates, execute the next safe move, and write operating artifacts to `agents/hermes/` only when they are useful.
- Outbound metrics: volume, opens, replies, CTR and readiness are signals; the truth metric for current outbound is the free diagnosis booked/completed and the downstream revenue it creates.

## ALMA Agentic
- `/home/almarev/agentic` é o stack técnico multi-tenant em produção/operacional.
- ORION lead-gen roda como `alma-orion@1..5.service`, gera lead-base e dashboard em `app.almarev.com/dashboard/leads`.
- Para perguntas de saúde/capacidade do ORION, usar `references/orion-healthscore-and-content-status.md` e separar stack health, mailbox health, throughput e forecast.
- LANCE (text SDR), CLAIRE (voice SDR), Clara e AURA existem como agentes/workstreams.
- Para mapa curto e limpo de funções atuais, consulte `references/agent-role-map.md`.

## Encerrados / histórico
- JT Shirts: encerrado. Contexto histórico: US uniform company, Fractional Head of Sales, ICP restaurants/franchises, field service, multi-site healthcare, stack Snov.io/Instantly/Python Google Places API. Usar só para arqueologia, migração, cleanup ou análise histórica.
- mtmalls: encerrado. Usar só para arqueologia, cleanup ou referência histórica.
- Pitfall: se STATUS.md, brain antigo, memória antiga ou resumo de stack listar JT Shirts/mtmalls como ativos, tratar como stale até haver decisão nova explícita do Mario reabrindo.

## Job Search ativo
- Posicionamento: Sales Director / VP Sales / CRO. NÃO General Manager.
- Estratégia: passa em todo processo, decide no offer.
- LinkedIn entra como filtro de vagas e triagem de oportunidades, não como fonte genérica de browsing contínuo.
- Quando Mario envia CV/resume + LinkedIn URL, fazer matching entre os dois, extrair role family, keywords e shortlist priorizada. Ver `references/job-search-linkedin-matching.md`.
- Se o CV e o LinkedIn divergirem, preferir o sinal mais recente e apontar o mismatch como risco de conversão.
- Instagram entra mais como canal de monitoramento de campanha do que como superfície operacional primária.
- Para reativação de integrações Composio com toolkit filtrado, ver `references/composio-app-priorities.md`.

## Regras de stack
- OAuth SEMPRE preferido. API key só como último recurso.
- Na VPS ALMA atual não assumir Unix user `almarev`; serviços systemd rodam majoritariamente como root/dev. Antes de usar `User=almarev` em unit file, verificar se o usuário existe ou criar explicitamente.

## Runbooks operacionais
- Hub Team roster / agent identity correction: quando Mario apontar texto errado no Hub (`/team`, cards de AI Employees, Agentic Lab, org chart), trate como correção de identidade operacional, não só copy. Para IRIS, o papel canônico é metrics/dashboard/analytics/digest/anomaly detection, não follow-up/scheduling/closer. Corrigir tanto runtime DB (`team_agents`, `agent_goals`) quanto fontes persistentes (`migrations/0041_team_cockpit.sql`, `migrations/0042_agent_descriptions.sql`, `hub/src/lib/agents.ts`, `cortex/team.py`, SOUL/template, prompt decomposer quando aplicável), depois `npm run build`, reiniciar `alma-cortex`/`alma-hub` e validar via `/team/agents` e `/api/cortex/team/agents`.
- Agent/systemd health audit + pause-all: ver `references/agent-systemd-health-audit.md`. Use quando Mario perguntar se os agentes estão instalados, vivos, “rotating”, rodando na VPS, quando houver falha em timers/services ALMA, ou quando pedir pra pausar/parar todos os agentes. Sempre cruzar Brain `STATUS.md` com systemd, timers, journal, failed units, processos soltos, Hermes cron e health endpoints.
- `references/instantly-outbound-status.md`: ver quando Mario perguntar se os emails/campanhas foram enviados, se ainda falta disparo, se houve reply/bounce/open, ou se alguém fez AURA Assessment.
- `references/instantly-warmup-screenshot-interpretation.md`: leitura rápida de screenshots do Instantly warmup, sinais de spam recente e classificação READY/WATCH/HOLD. Orange bars in the warmup chart mean spam events. Use screenshot como sinal visual, mas feche a decisão pelo estado live, com `clean_streak_days` como verdade para READY/WATCH/HOLD e sem extrapolar contagens do print para o inventário total.
- `references/instantly-capacity-maximization.md`. Use when Mario perguntar sobre plano Instantly, cap de leads, volume diário, capacidade das contas, tamanho de cadência, campanhas pausadas/ativas, ou se estamos maximizando outbound. Sempre separar lead cap workspace-wide, capacidade de mailboxes, daily_limit de campanhas e número real de steps na sequência.
- `references/instantly-sender-ops-live-audit.md`. Use when the question is "how is it programmed tomorrow?", "who sends tomorrow?", or any live sender-window check. Read active campaigns, schedule timezone, and per-account today counts from the live API before answering.
- `references/instantly-tomorrow-program-live-audit.md`. Use when a snapshot or export still shows old campaign names, mixed senders, or a target limit that may not match live state. Explicitly separate intended tomorrow program, live active sequencer, exclusions, and drift before answering.
- `references/instantly-readiness-projection.md`. Use when the pergunta for projeção de quantas contas ficam prontas amanhã/quarta/na próxima data. Revalidar o estado live e projetar a partir de `clean_streak_days`, `warmup_status` e janela de segurança, não de resumo anterior nem de screenshot isolado.
- `references/instantly-warmup-streak-check.md`. Use when Mario asks for exact warmup streak buckets like "4 days", "5 days", or when his estimate disagrees with the DB total. Count `clean_streak_days` exactly, separate it from `current_cold_limit`, and verify against the daily log before answering.
- Warmup screenshot pitfall: orange segments in Instantly should be treated as spam signals. Do not trust the summary card or screenshot alone for the final eligibility list, reconcile against the live daily log before answering.
- Warmup wording pitfall: when Mario says "nos últimos quatro dias a conta tem que ter enviado zero spam", interpret that as the last 4 calendar days all spam-free. Do not rewrite it as a generic "4+ days clean" streak unless he explicitly says "4 dias ou mais".
- Final list hygiene: after spotting a typo or OCR drift in a sender address, reconcile the whole list again before returning the clean/dirty split. A stale include in the "clean" bucket is a real error, not cosmetic.


- `Granola meeting context`: ver `references/granola-meeting-notes.md`. Use quando Mario pedir para ler recaps de reuniões, extrair follow-ups, ou reutilizar contexto de calls; trate como insumo recorrente, não como verdade canônica se conflitar com STATUS/_CURRENT/decisions.
- `HERALD activity notification`: ver `references/herald-activity-notification.md`. Use quando o problema for mensagem entrando mas o alerta não disparar, ou quando for preciso garantir notificação em tempo real com fallback de watchdog.
- `Hermes Telegram agent fleet`: ver `references/hermes-telegram-agent-fleet.md`. Use quando Mario pedir migração de agentes Claude/LangGraph/OpenClaw para Hermes profiles/bots, criação de um bot por agente, troca de token do BotFather, ou start/validação de gateways por profile.
- LANCE pause/resume: ver `references/lance-systemd-pause.md`. Use quando Mario pedir pra pausar LANCE, quando `alma-lance.timer` reaparecer, ou quando `systemctl disable --now` travar.
- ORION pause/resume: ver `references/orion-systemd-pause.md`. Use quando Mario pedir pra parar geração de leads, pausar ORION ou retomar lead-gen. Sequência correta: pausar `orion-watchdog`, parar/desabilitar `alma-orion@1..5`, limpar `failed` se necessário, verificar inactive/disabled/sem processo.
- `references/orion-lead-factory-verification.md`: flow canônico do Lead Factory, `ICP search -> mails.do -> account verification`.
- `references/orion-lead-factory-operating-checklist.md`: checklist curto para travar blast, gate de entrada, classificar `sub_vertical`, limitar por saúde do sender e enviar em lotes pequenos.
- `references/orion-classification-signal-drift.md`: empty `by_sub_vertical` means upstream classification drift, not necessarily no leads.
- CLAIRE dialer operations: ver `references/claire-dialer-operations.md`. Use quando Mario pedir pra pausar/retomar dialer, fim de semana, ligações da CLAIRE ou checar se ela está discando. Regra padrão: dialer só segunda a sexta; pausar outbound/reminder sem desligar polling/QA/coach.

- `references/claire-callback-bridge.md`: use quando o assunto for o callback de replies interessados da Rev. Cobre o gate duplo (`REV_CALLBACK_ENABLED` + `CLAIRE_CALLS_ENABLED`), fallback para `RETELL_AGENT_ID`, a sequência segura de enable/restart/verify, and the boundary that Hermes only wires the local env/id; the Retell agent itself must already exist.
- LUNA content digest: ver `references/luna-content-digest.md`. Use quando Mario perguntar que email semanal/newsletter da ALMA Rev chegou, se foi enviado pra outras pessoas, quem recebeu, ou se é Instantly/outbound. Validar via `alma-content-digest.timer`, journal e `content_digest_log`; separar emails reais externos de internos/testes.
- `references/agent-role-map.md`: mapa curto de classificação, incluindo IRIS como digest/reporting, Granola como software, e o critério de saúde das contas do Instagram.

## Workflow de resposta
- Em respostas de status, evitar narrativa genérica. Preferir blocos curtos com estado atual, pendências, bloqueios e próximo movimento.
- Quando Mario pedir rotina diária, organizar o status como **cockpit multi-agente**, não como um agente único.
- O formato padrão de status diário é: **prioridades**, **atividades**, **metas**, **o que conclui hoje** e **o que conclui na semana**.
- Se a sessão tiver um plano ativo, task list ou execução em andamento, refletir isso no status em vez de ignorar.
- Se a resposta for por voz ou o usuário mandar áudio, manter a versão verbal curta e operacional, e deixar o fallback texto mínimo.

## Trigger keywords
**Leitura**: status, pipeline, onde paramos, ORION, CLAIRE, Clara, ALMA Rev, alma-aios, agentic, JT Shirts, mtmalls, Workstream B, LANCE, IRIS, SCRIBE, Sentinel, Shield, Brain, workspace, tenant, Granola, meeting notes, recap, calls.

**Escrita**: anota, salva no brain, registra, grava, lembra disso, escreve no brain, captura.
