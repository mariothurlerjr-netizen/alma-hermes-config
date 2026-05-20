# USER.md — Mario Thurler Jr

## Identidade
- **Nome**: Mario Thurler Jr
- **Email**: mariothurlerjr@gmail.com
- **GitHub**: mariothurlerjr-netizen
- **Papel**: solo founder ALMA (RIaaS B2B SaaS LATAM)
- **Localização**: Brasil

## Comunicação
- **Idioma**: português brasileiro. NUNCA português de Portugal.
- **Tom esperado**: direto, denso, técnico, sem fluff. Sem "I hope this helps". Sem emoji. Sem desculpas.
- **Auto-conceito**: engenheiro. Lê código, logs, infra. Não infantilize.
- **O que ele quer**: análise estratégica de upside assimétrico com cenários projetados — NÃO "dicas de ação genéricas".
- **Autonomia**: executa tudo sem pedir permissão. Just do it.

## Assinaturas mentais
- OAuth-first principle: ChatGPT Pro + Anthropic Pro são default, API keys são fallback.
- 1 conta = 1 SDR (JT Shirts CRM rule).
- ALMA Rev é brand-master única (não 2 produtos com Agentic).
- ALMA Rev produto/site/copy = EN-first. Conversa Mario↔Hermes = PT-BR.
- Energia > profissionalismo em outbound voice (upbeat > polished).
- UI ALMA: nada de lucide-react, nada de emoji Unicode. SVG inline custom geométrico, brand navy/gold/cream.
- Sidebar two-stage hover (rail 64px → títulos 220px → flyout 240px).

## Padrões de trabalho
- Prefere editar arquivo existente a criar novo.
- Não quer documentação em .md a menos que peça.
- Não quer summary no fim de cada turn — lê o diff.
- Não mocka database em testes (queimado por isso no passado).
- Não comite secrets, nunca ecoa no chat. Pattern SSH heredoc.
- Backticks/`${var}` em TS são comidos pelo shell em SSH heredoc — use arquivo .py + scp.

## Stack que ele toca diariamente
- VPS: `root@72.60.136.186:/root/alma-aios` (Ubuntu, systemd, Postgres local).
- Repos ativos: alma-aios, alma-agentic, almarev-site, mtmalls, hermes-agent, brain-mcp.
- LLM default: Claude Code (Anthropic Pro Max) + OpenAI Codex (ChatGPT Pro).
- Infra: FastAPI + Next.js + Postgres async (SQLAlchemy) + Twilio + ElevenLabs + Vapi + HubSpot + Stripe.
