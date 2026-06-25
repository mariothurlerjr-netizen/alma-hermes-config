# HEARTBEAT - Checklist proativo

Roda a cada 30min via cron. Para cada item:
1. Verifica se condição se aplica AGORA
2. Se sim, age (Telegram do Mario ou executa)
3. Se não, silêncio (NÃO manda "tudo ok")

## Antes de qualquer ação
- LÊ `/root/.hermes/memory/corrections.md` (não repete erro)
- LÊ `/root/.hermes/memory/patterns.md` (padrões de falha)
- LÊ `/home/almarev/brain/STATUS.md` (estado real, auto-gerado 24h — agentes, ORION metrics, workspaces)
- LÊ `/root/.hermes/memory/heartbeat-log.md` (últimas 8 entradas, anti-spam)

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

### 7. Síntese estratégica do Brain
- Algum arquivo em `00-execution-plans/` ou `02-alma-rev/` foi atualizado nas últimas 24h mas o Mario não foi avisado?
- Há decisões em `03-alma-agentic/decisions.md` marcadas como pendentes de ação há >72h?
- `STATUS.md` mostra alguma métrica fora do padrão (ORION leads < média semanal, reply rate caindo, funnel travado)?
- Se detectar padrão ou oportunidade clara, sintetiza em 2-3 linhas e envia — não espera estar quebrado.

### 8. Sinais de funil
- Novas sessões em `web_events` sem follow-up programado nas últimas 24h?
- Algum lead chegou ao stage AURA Assessment mas não foi contatado?
- Emails `@getalmarev.com` com bounce rate >3% nesta semana?

### 9. Ritmo de conteúdo
- Última publicação no blog foi há >7 dias? (verificar `content_queue` no DB cortex)
- `content_publisher.timer` ativo e rodando?

### 10. Hermes sobre si mesmo
- Tem erros no heartbeat-log das últimas 4 entradas que nunca foram reportados?
- Tem items marcados como TODO em `~/.hermes/memories/MEMORY.md`?

## Critério de envio

**Alertas operacionais** (service down, disk, erro): não repetir em **4h**
**Síntese de negócio / insights / open loops**: não repetir em **24h**

Critério geral: novo + acionável (Mario age em menos de 5min) + não foi avisado no window acima.

## Silêncio é positivo
Não manda "tudo ok". Spam é pior que silêncio.

## Onde escrever
SEMPRE append em `/root/.hermes/memory/heartbeat-log.md` com timestamp + decisão (mandou ou ficou em silêncio + razão de uma linha).
