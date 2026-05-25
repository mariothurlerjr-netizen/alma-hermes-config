# HEARTBEAT - Checklist proativo

Roda a cada 30min via cron. Para cada item:
1. Verifica se condição se aplica AGORA
2. Se sim, age (Telegram do Mario ou executa)
3. Se não, silêncio (NÃO manda "tudo ok")

## Antes de qualquer ação
- LÊ `/root/.hermes/memory/corrections.md` (não repete erro)
- LÊ `/root/.hermes/memory/patterns.md` (padrões de falha)
- LÊ `/home/almarev/brain/STATUS.md` (estado real, auto-gerado 24h — agentes, ORION metrics, workspaces)
- LÊ `/root/.hermes/memory/heartbeat-log.md` (últimas 8 entradas, anti-spam — não repete aviso em 4h)

## Checklist

### 1. Workstream B — go-live gates
Algum dos 5 itens externos pendentes do Mario (Stripe Pub Key, nginx swap, Clerk Production, role admin, E2E test) virou bloqueante novo? Avisa.

### 2. ORION workers
ORION worker rodando? Reply rate sendo medido? Apollo credits OK? Se Phase 1 atingir reply rate ≥2%, sinaliza Mario pra autorizar Phase 2 (Tavily upgrade + Browserbase).

### 3. Active pipeline JT Shirts
CLAIRE pausada — algum sinal novo (Trust Hub, Vapi migration, carrier flag)?
LANCE parado desde 09/04 — algum motivo pra reativar (campanha agendada, conta nova)?

### 4. Open loops esquecidos no Brain
Algum item em `00-execution-plans/` ou `02-alma-rev/` sem atualização há mais de 48h e marcado como ativo? Avisa com sugestão.

### 5. Services críticos
- alma-cortex.service (port 8002) — active?
- hermes-gateway.service — active as user service? Use `hermes gateway status` or `XDG_RUNTIME_DIR=/run/user/0 systemctl --user is-active hermes-gateway.service`; system unit inactive/absent alone is not down.
- brain-mcp uvicorn (port 8003) — respondendo?
- Postgres alma_agentic — acessível?

Qualquer down → mensagem objetiva imediata.

### 6. VPS disk
Disco > 80%? Avisa com cleanup candidate (cache npm/pip/whisper geralmente).

## Silêncio é positivo
Não manda "tudo ok". Spam é pior que silêncio. Critério: novo (não avisou em 4h) + acionável (sugestão concreta) + Mario age em menos de 5min.

## Onde escrever
SEMPRE append em `/root/.hermes/memory/heartbeat-log.md` com timestamp + decisão (mandou ou ficou em silêncio + razão de uma linha).
