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
- ALMA Rev é brand-master única; positioning atual (#182, 2026-06-17): Growth Operation para US professional-services firms, operator/done-for-you, Diagnostic US$2K + Retainer US$4K/mo, tecnologia backstage.
- ALMA Rev produto/site/copy/outbound = US-first e 100% inglês; nada em português em campanha, site, headlines, CTA ou footer quando evitável. Conversa Mario↔Hermes = PT-BR.
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
- Espera que Hermes sinalize proativamente quando ALMA outbound/ops não está maximizando capacidade ou resultado, propondo uma estratégia concreta de maximização.

## Stack que ele toca diariamente
- VPS: `root@72.60.136.186:/root/alma-aios` (Ubuntu, systemd, Postgres local).
- Repos ativos: alma-aios, alma-agentic, almarev-site, mtmalls, hermes-agent, brain-mcp.
- LLM default: Claude Code (Anthropic Pro Max) + OpenAI Codex (ChatGPT Pro).
- Infra: FastAPI + Next.js + Postgres async (SQLAlchemy) + Twilio + ElevenLabs + Vapi + HubSpot + Stripe.
§
Mario prefere revisões de status por área, com estado atual, pendências, bloqueios e próximo movimento.
§
Mario prefere split-model workflow.
§
Mario prefere ensino em camadas: fatos duráveis, procedimentos, contexto operacional e handoffs.
§
Mario quer o daily do Hermes em 4 frentes: Instantly/email, balanceamento de leads entre ICPs, completude/enrichment da base, e market/content watch para Instagram, blog e email.
§
Mario quer que vídeos do Alfredo Soares sejam acompanhados como fonte recorrente de ideias de GTM e marketing.