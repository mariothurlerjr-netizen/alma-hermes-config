# MEMORY.md — Hermes operational memory

Persistente entre sessões. Atualizar quando aprender algo novo. Linhas curtas; detalhe vai no brain (`/home/almarev/brain`).

## Projetos ativos (estado em 2026-05-20)

### Workstream B — go-live portal cliente
- Stripe TEST + Resend + PostHog + DNS app.almarev.com + SSL = DONE.
- Faltam 5 ações Mario externas: Stripe Pub Key, nginx swap, Clerk Production, role admin, E2E test.
- Workstream B é o **gate único de receita** — sem ele, zero pagantes externos.

### ORION (lead generation pipeline)
- Phase 1 fully implemented + DISABLED aguardando autorização Mario.
- Stack: Apollo + Snov + Claude OAuth (no LLM cost via subscription).
- Performance: ~50 leads/h por instância, 67% ICP accuracy, 25% email discovery.
- Path: `/home/almarev/agentic/agents/orion/` + integrado em `app.almarev.com/dashboard/leads`.
- 4 leads reais já gerados via smoke test 2026-05-19.
- Bug crítico fixado: `claude -p` subprocess precisa `stdin=DEVNULL` no systemd (10x speedup).

### CLAIRE (voice SDR — lab estratégico)
- JT Shirts é o lab. Calibragem vira feature B2B do ALMA Agentic.
- Stack atual: ElevenLabs Conv AI + Twilio. Migração Vapi.ai em Sem 4.
- v28 mantém prod com volume reduzido (5-10/dia) após carrier flag.
- System prompt parametrizado por workspace (não hardcode).

### Clara (interview agent)
- Dual-channel SMS/WhatsApp deployed 2026-05-19.
- WhatsApp uses audio generation; SMS usa welcome message text otimizado.
- Geographic Twilio from-number routing implementado.

### mtmalls.com
- Sprint 1 fechado 2026-05-16. 4 agentes: CONCIERGE, NEGOTIATOR, SENTINEL, PILOT.
- Stack: VPS systemd + Postgres local + Claude OAuth subprocess + OpenAI embeddings.
- Clerk app separado "MallsOS" (não reusar almarev).

## VPS state

### Disk (após cleanup 2026-05-19)
- 76% → 52% used (47G livres).
- Backups em `/root/backups/cleanup-20260519/`.
- Apagados: Mario AI Assistant, n8n, reacher, revenue-trust, aios-core, jarvis-archived, alma-board, agentic-v3.

### Services ativos
- `alma-aios` (systemd, FastAPI :8000, legacy, 85% morto — só claire+board ativos).
- `cortex` (FastAPI :8002).
- Hermes gateway (systemd --user, hermes-gateway.service).
- OpenClaw gateway (port 18789, systemd --user).
- ORION worker (DISABLED).

### Postgres clusters
- `alma_aios` (150t, legacy).
- `alma_agentic` (30t, novo, multi-tenant com workspace_id FK + RLS).
- Database `almarev` NÃO EXISTE (confusão antiga).

## Operational conventions

### LLM
- Default = OAuth (Claude Code CLI ou ChatGPT Codex CLI). $0 incremental.
- API keys (ANTHROPIC_API_KEY, OPENAI_API_KEY) = fallback only.
- Subprocess wrappers chamando `claude -p`: REMOVER ANTHROPIC_API_KEY do env, senão cai em API key zerada.

### Multi-tenant
- workspace_id FK obrigatório em qualquer tabela nova em `alma_agentic`.
- Clerk publicMetadata + RLS pra isolation.
- API proxy de Next → Cortex injeta workspace_id automaticamente.

### Communication channels
- Email: Resend.
- SMS/WhatsApp: Twilio (geographic routing per from-number).
- Voice: ElevenLabs (migrando pra Vapi).
- Calendly: prefill URLs em template; webhook signing bloqueado (tier limit) → Nginx rate limit ativo.

## Limites conhecidos
- Claude Chrome NÃO funciona em Stripe/Clerk/Cloudflare (anti-bot). Mario faz manual.
- Twilio Trust Hub: Individual bloqueia outbound US. Business CNPJ Brasileiro aceito (approval 3-5d).
- Snov.io: dedupe por parent domain (subsidiárias consumem créditos extras).
- mails.so ~1/3 hit rate em SMB.
- Apollo `/people/match` retorna company context errado → Snov fallback.

## Brain integration
- Vault real: `/home/almarev/brain`.
- MCP server: `alma-brain-mcp` v0.2.0 com OAuth2 (Claude.ai web custom connector).
- Skills `alma-*` devem ler `/home/almarev/brain`, NÃO `/root/.hermes/brain` (placeholder antigo — corrigido 2026-05-20).
- Hot file: `/home/almarev/brain/wiki/hot.md` — LER PRIMEIRO sempre.
<!-- autopush smoke 2026-05-20T17:36:26Z -->
