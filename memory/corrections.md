# Corrections from Mário

Formato:
## YYYY-MM-DD HH:MM | <skill ou contexto>
- O que fiz: <ação original>
- Correção: <feedback literal>
- Lição: <regra>
- Status: APPLIED | NOT_APPLIED

---

## 2026-05-23 12:24 UTC | ORION
- O que fiz: interpretei o bloqueio Apollo como falta de crédito para extrair leads novos.
- Correção: “não estou extraindo novos leads do apollo, mas usando os que já extrai e já consumi créditos”.
- Lição: ORION deve drenar `hub_leads`/leads Apollo já pagos antes de qualquer sourcing live; erro de crédito Apollo não deve bloquear processamento de contatos já armazenados.
- Status: APPLIED

[2026-05-28T16:53:48Z] Mario correction: I failed a shorthand reminder request ('msg hj as 8') by not creating a cron job and leaving it unscheduled after asking for clarification. SWITCH_STRATEGY: for noisy reminder requests, create the cron job when time/content are inferable; if content is missing, explicitly say no reminder was scheduled and ask for the exact text.

[2026-05-28T16:54:24Z] Mario correction: I incorrectly told Mario there was no message job after cron list showed `lembrete-reuniao-alisa-rogerio-video` scheduled for 2026-05-29 11:00 UTC, equivalent to 08:00 Brazil. SWITCH_STRATEGY: when checking missed reminders, interpret existing cron jobs by timezone and inspect prompt/script before saying no job exists.

[2026-06-19T18:25:00Z] Mario correction: I said 77 leads that received the 3rd email should be treated as sequence-closed/cooldown. Correction: after the 3rd email, the agreed next step is a connection/call via CLAIRE for eligible leads. SWITCH_STRATEGY: for ALMA Rev cadence status, treat post-3rd-email as call-readiness/CLAIRE queue, not recycle/cooldown by default.

## 2026-06-25T22:58:14+00:00 | Instantly sender readiness
- O que fiz: interpretei contas com score alto no Instantly como usáveis para produção controlada.
- Correção: Tirando getslmarev/getalmarev e tryslmarev/tryalmarev, todas as contas estão aquecendo ainda. Vão ficar prontas na próxima terça.
- Lição: Para ALMA Rev Instantly, readiness operacional por domínio vence score bruto da API. Até terça, tratar só getalmarev.com e tryalmarev.com como domínios prontos; demais domínios permanecem warmup-only mesmo com score alto.
- Status: APPLIED

## 2026-06-25T23:00:53+00:00 | Instantly sender readiness
- O que fiz: mantive `tryalmarev.com` como domínio pronto depois da correção anterior.
- Correção: tryalmarev teve envios para spam esta semana; deixar aquecer até terça também.
- Lição: Para ALMA Rev Instantly, `tryalmarev.com` deve ficar warmup-only até terça quando houver spam recente no warmup, mesmo com score alto ou cold sends sem bounce.
- Status: APPLIED

## 2026-06-25T23:03:43+00:00 | Instantly sender readiness
- O que fiz: usei score e domínio como critérios principais de readiness.
- Correção: só habilitar contas que não tiveram spam nos últimos 7 dias.
- Lição: Para ALMA Rev Instantly, zero warmup spam nos últimos 7 dias é hard gate por mailbox antes de colocar em cold outbound.
- Status: APPLIED

## 2026-06-26T02:46:15Z | alma-brain freshness
- O que fiz: eu e outros agentes confundimos fontes antigas com estado vigente no brain e às vezes puxamos informação velha.
- Correção: “vc e o claude se confundem muito no brain... as vezes vcs puxam informcoes antigas”.
- Lição: Para qualquer resposta baseada no brain, primeiro estabelecer hierarquia temporal e canônica: STATUS/_CURRENT/business-plan canônico + decisões recentes + dado vivo quando operacional. Arquivo antigo só entra como histórico, nunca como verdade final sem checar `status`, `last_reviewed`, timestamp e conflito com fontes mais novas.
- Status: APPLIED

## 2026-06-26T02:51:58Z | project-status
- O que fiz: Tratei JT Shirts e mtmalls como projetos/workstreams ativos na lista de stack.
- Correção: "jtshirts e mtmalls estao encerrados"
- Lição: JT Shirts e mtmalls são workstreams encerrados; só tratar como ativos se Mario reabrir explicitamente.
- Status: APPLIED

## 2026-06-27T00:00:00Z | clarification
- O que fiz: interpretei a mensagem do Mario sem o contexto suficiente.
- Correção: "Eu acho q vc entendeu errado"
- Lição: quando a correção vier sem o ponto exato, parar e pedir a distinção específica antes de assumir qual parte está errada.
- Status: APPLIED

## 2026-06-27T00:00:00Z | lead factory flow
- O que fiz: associei o lead factory a verifier errado e tratei a etapa de verificação como genérica.
- Correção: "Lead factory: vc busca as contas no icp, rodarmos o mails.do e verificamos a conta"
- Lição: o fluxo canônico do Lead Factory é ICP search -> mails.do -> account verification.
- Status: APPLIED
## 2026-06-29T13:28:24+00:00 | research-double-check
- O que fiz: respondi sem explicitar a regra de devil's advocate e double-check do número do mercado.
- Correção: Mario quer que eu revise as informações que ele passa, pesquise, valide se faz sentido e o corrija quando não fizer.
- Lição: antes de aceitar uma tese operacional, fazer checagem externa e contradizer a hipótese se o mercado não sustentar.
- Status: APPLIED

## 2026-06-29T13:33:19+00:00 | review-numbers-verdict
- O que fiz: respondi com faixas de benchmark, mas não deixei o veredito explícito em formato agree/disagree.
- Correção: Mario quer que eu revise o número e diga claramente se concordo ou não, sempre como devil's advocate.
- Lição: toda tese numérica precisa sair com veredito explícito + motivo + próximo passo.
- Status: APPLIED


## 2026-06-29T16:23:18+00:00 | instantly-ready-count-mixup
- O que fiz: misturei o recorte do inventário live completo com o recorte das screenshots e soltei o número errado de READY.
- Correção: Mario disse que continua errado.
- Lição: quando citar READY/WATCH/HOLD, separar explicitamente snapshot global versus subset de imagens.
- Status: APPLIED

## 2026-06-30T00:00:00Z | multi-agent-ops-status
- O que fiz: tratei a operação como se eu respondesse por um único agente e descrevi o dia em termos genéricos.
- Correção: "você hoje responde por vários agentes" e o recorte correto é prioridades, atividades, metas e o que a gente conclui hoje e na semana.
- Lição: quando falar de rotina diária, enquadrar o trabalho como cockpit multi-agente e resumir por prioridades, atividades, metas e entregas do dia/semana, sempre por agente.
- Status: APPLIED
|
## 2026-07-01 16:05 UTC | model routing
- O que fiz: usei um modelo/contexto que o Mario apontou como errado.
- Correção: "o model ta errado".
- Lição: quando Mario sinalizar modelo errado, parar e alinhar qual routing/modelo ele quer antes de seguir.
- Status: APPLIED
|

## 2026-07-02T17:57:30+00:00 | youtube-content
- O que fiz: tratei o link do YouTube sem confirmar o vídeo/transcript corretamente.
- Correção: Mario reenviou o link correto: https://youtu.be/PsE9u37gJjU?is=syrpMi2YKprqn0Fw
- Lição: em links de YouTube, confirmar video_id, título e fonte antes de resumir; se transcript falhar por bloqueio da VPS, rotular explicitamente a leitura como metadata-grounded.
- Status: APPLIED

## 2026-07-08T13:17:00Z | ORION lead factory
- O que fiz: tratei o pull Apollo como recuperação de estoque e respondi status sem assumir explicitamente a falha da missão diária open-web/no-Apollo.
- Correção: Mario reforçou que a missão do ORION é gerar 100 leads/dia sem depender do Apollo, perguntou por que gerou zero e por que não avisei antes.
- Lição: produção zero ou abaixo da meta diária é desvio crítico; ORION deve avisar proativamente com causa e plano de desbloqueio, e Apollo/manual pull não substitui a fábrica diária própria.
- Status: APPLIED

## 2026-07-08T13:20:00Z | ORION lead factory free sourcing
- O que fiz: propus Ample/cheap extractor como parte da recuperação de produção.
- Correção: Mario não quer assinar extractor novo; quer maneira gratuita de gerar leads.
- Lição: plano de desbloqueio ORION deve priorizar fontes gratuitas/open-web e infraestrutura própria; paid extractors entram só se Mario pedir explicitamente.
- Status: APPLIED

## 2026-07-08T13:23:21Z | ORION ICP size filter
- O que fiz: propus sourcing gratuito/open-web sem explicitar filtro operacional forte de tamanho de empresa.
- Correção: Mario reforçou que ALMA Rev deve atacar empresas com no máximo ~$10M de faturamento; empresas grandes, bancos gigantes e instituições grandes não servem para a consultoria.
- Lição: todo lead ORION precisa passar por size gate antes de contar: excluir enterprise/large firms usando proxies de headcount, professional count, office count, AUM, naming e sinais de site quando revenue não estiver disponível.
- Status: APPLIED

## 2026-07-09T01:23:30+00:00 | email-routing/getalmarev
- O que fiz: tratei getalmarev como possivelmente dependente de Maildoso.
- Correção: "O getalmarev está direto no Gmail, não no maildoso"
- Lição: getalmarev.com usa Gmail/Google Workspace direto como fonte primária, não Maildoso.
- Status: APPLIED

## 2026-07-15T10:48:27Z | mailbox-reconciliation-autonomy
- O que fiz: pedi aprovação/esperei confirmação para seguir com a reconciliação de contas após identificar o recorte operacional.
- Correção: "Pode seguir. Não precisava nem da minha aprovação"
- Lição: Em reconciliação de mailboxes/warmup e remediações operacionais de baixo risco, Jarvis deve executar direto quando o próximo passo seguro estiver claro. Só pedir aprovação se for prospect-facing, gated, custo novo, destrutivo ou sem credencial/fonte autenticada.
- Status: APPLIED


## 2026-07-27T14:06:01.470909+00:00 | MUSE/content research
- O que fiz: recomendei direção criativa de IG primeiro por julgamento interno e só pesquisei depois que o Mario pediu.
- Correção: "Toda vez que você for fazer alguma coisa, pesquise para ver qual é o mais assertivo, o que vai dar mais resultado, e o que vai ter mais visualização e mais alcance".
- Lição: antes de criar ou recomendar conteúdo, formato, visual, hook, script, carrossel, blog, Reel ou campanha, pesquisar referências/benchmarks atuais e escolher o caminho mais assertivo para resultado, visualização, alcance e conversão.
- Status: APPLIED
## 2026-07-27T15:52:55+00:00 | alma-agent-roster
- O que fiz: tratei SAGE como o bot/agente novo relevante nesta frente sem registrar Friday Audit.
- Correção: Mario informou que além do SAGE existe um bot novo chamado Friday Audit, responsável só por auditorias.
- Lição: separar SAGE de Friday Audit; SAGE = Growth R&D/outbound experiments, Friday Audit = auditorias.
- Status: APPLIED

## 2026-07-30T17:22:14+00:00 | alma-style/telegram-readability
- O que fiz: formatei respostas no Telegram com blocos densos e títulos próximos do conteúdo.
- Correção: Mario pediu títulos em negrito, espaço depois do título, conteúdo separado, espaço entre tópicos.
- Lição: em respostas no Telegram, usar títulos em negrito, linha em branco depois do título, conteúdo em bloco separado e linha entre tópicos para facilitar leitura rápida.
- Status: APPLIED

## 2026-07-30T17:25:29+00:00 | outbound-reporting/test-exclusion
- O que fiz: reportei 2 interested no Sprint Cold Email V3 sem excluir provável teste da base operacional.
- Correção: Mario disse que os 2 interessados provavelmente foram teste e que tudo que é teste deve ser excluído da base de relatórios.
- Lição: relatórios operacionais devem separar produção real de teste; testes não entram em métricas de interessados, replies, reuniões, conversões ou performance.
- Status: APPLIED

## 2026-07-31T13:06:22+00:00 | Instantly/getalmarev reservation
- O que fiz: tratei `getalmarev.com` como reservado para resposta/1:1.
- Correção: "Getalmarev não está reservado pra resposta"
- Lição: `getalmarev.com` não está reservado para resposta/1:1; em outbound, avaliar e usar por gates vivos de warmup, compliance, sender health, sending_gap, bounce/reputation e aprovação quando aplicável.
- Status: APPLIED

## 2026-07-31T13:09:15+00:00 | outbound-reporting/source-of-truth
- O que fiz: mandei informações sobre campanhas de e-mail sem garantir que a fonte era live, correta e separada de teste/noise.
- Correção: "Pq vc tá mandando informações sobre as campanhas de e-mail e as suas informações não estão certas"
- Lição: Sentinel só deve reportar status ou métrica de campanhas ALMA Rev quando consultar fonte viva Instantly/Cortex produção com timestamp; nunca usar brain stale, memória, cron antigo, webhook/test replies ou inferência como fonte final.
- Status: APPLIED

## 2026-07-31T13:12:28+00:00 | sentinel-role-boundary
- O que fiz: enviei informação interna de empresa/campanhas que o Mario já tem via link/dashboard e que sobrepõe outros bots.
- Correção: Mario disse que Sentinel não precisa mandar informação da própria empresa/campanhas; deve trazer informação de mercado, cobertura, possibilidades, concorrência e sinais externos dentro do job description.
- Lição: Sentinel deve focar output proativo em market intelligence externo acionável: mercado, ICP, verticais, concorrência, coverage/opportunity e sinais que mudam decisão. Métrica interna de campanha só entra quando Mario pedir explicitamente, com fonte viva e timestamp.
- Status: APPLIED


## 2026-07-31T17:17:19+00:00 | whatsapp-relationship-style
- O que fiz: sugeri mensagem de WhatsApp com tom mais polido/consultivo e com risco de soar crítica ou posicionamento fechado.
- Correção: Mario quer versão mais curta, no jeito dele, sem crítica, apresentando como uma possibilidade de posicionamento de mercado.
- Lição: drafts de WhatsApp para relacionamento devem soar como Mario: curto, natural, exploratório, sem pitch, sem crítica, com linguagem de possibilidade e conversa.
- Status: APPLIED


## 2026-07-31T17:30:00+00:00 | whatsapp-human-connection
- O que fiz: resumi uma mensagem de WhatsApp de forma seca, indo direto ao artigo sem abertura humana.
- Correção: Mario quer incluir conexão humana quando adequado, como “Fred, tudo bem?” e “como tem sido a semana em São Paulo?”.
- Lição: drafts de WhatsApp para relacionamento devem abrir com uma ponte humana breve antes do assunto, mantendo naturalidade, contexto e calor sem alongar demais.
- Status: APPLIED


## 2026-07-31T17:25:00+00:00 | whatsapp-positioning-clarity
- O que fiz: resumi a mensagem sobre Sequoia como "camada operada em cima da ferramenta", deixando confuso e apagando a palavra serviço.
- Correção: Mario apontou que o ponto importante é serviço, não apenas camada em cima da ferramenta.
- Lição: ao resumir a tese Services: The New Software para WhatsApp, manter explícito que empresas de software podem vender serviço/trabalho pronto além da ferramenta. Evitar formulação vaga como "camada em cima da ferramenta" sem dizer serviço.
- Status: APPLIED


## 2026-07-31T17:28:12.988004+00:00 | sequoia-service-as-software-language
- O que fiz: formulei a tese como “empresas de software podem capturar mais valor quando vendem também o serviço”, o que sugere software + serviço como duas coisas separadas.
- Correção: Mario corrigiu que a tese é vender o software como serviço/trabalho pronto, não vender também serviço em cima do software.
- Lição: ao explicar Sequoia/service-as-software, usar “vender o software como serviço/trabalho pronto” ou “software se apresenta como serviço”; evitar “vender também serviço” quando isso separar produto e serviço.
- Status: APPLIED


## 2026-07-31T17:30:36.804174+00:00 | sequoia-service-as-software-language-2
- O que fiz: chamei a ideia de “tese” e formulei como software entregue como serviço.
- Correção: Mario corrigiu que não é para chamar de tese na mensagem; é um estudo/leitura de mercado: empresas pagam mais por serviço e a tendência é vender serviço com software.
- Lição: para mensagem 1:1 sobre o artigo da Sequoia, usar linguagem simples: “estudo/artigo mostra que empresas pagam mais por serviço do que por software; tendência é vender serviço com software por trás”. Evitar soar acadêmico com “tese”.
- Status: APPLIED


## 2026-07-31T17:46:28.519581+00:00 | alma-not-growth-only
- O que fiz: reduzi a leitura de serviços comprados pela ALMA para growth/agency/SDR/follow-up.
- Correção: Mario lembrou que já tínhamos alinhado que o escopo inclui serviços financeiros, operações e RevOps, não só growth.
- Lição: quando discutir service-as-software/ALMA, partir de operating functions amplas: growth, finance/cash flow, operations e RevOps. Growth é wedge, não categoria inteira.
- Status: APPLIED

## 2026-08-06T14:32:28+00:00 | MUSE visual assets
- O que fiz: usei ou considerei composição de picture-in-picture e possivelmente uma imagem já usada.
- Correção: "Remember that we are not putting pictures in pictures" e "I think that this picture was used".
- Lição: não usar imagem dentro de imagem em assets ALMA Rev, salvo pedido explícito; antes de reaproveitar imagem, verificar histórico de uso/publicação para evitar repetição.
- Status: APPLIED


## 2026-08-07T14:34:39Z | MUSE/visual-quality
- O que fiz: gerei ou propus imagem com aparência ruim e elemento estranho/mesclado que não estava claro visualmente.
- Correção: "This image is very bad, and it has a strange business that we have combined. What is it?"
- Lição: em assets visuais ALMA Rev, rejeitar composições com elementos artificiais, mesclados, deformados ou ambíguos antes de mandar para aprovação; se houver qualquer objeto estranho na cena, refazer com composição limpa e verificável.
- Status: APPLIED


## 2026-08-19T03:09:15Z | MUSE/reference-review intent
- O que fiz: analisei posicionamento, modelo e performance da conta enviada, sem começar pela qualidade das imagens.
- Correção: "Quero q vc olhe a qualidade das imagens" e "Olhe essa qualidade tbm e veja como implementar".
- Lição: quando Mario enviar um perfil ou referência visual para MUSE, priorizar auditoria objetiva da qualidade visual e transformar os padrões aprováveis em implementação concreta no pipeline criativo da ALMA Rev; estratégia de negócio entra só como contexto secundário.
- Status: APPLIED


## 2026-08-19T14:03:11Z | MUSE/visual-reference fidelity
- O que fiz: transformei a referência Devolvi Meu Crachá em uma direção genérica de talking head/founder footage, perdendo a composição e o tratamento visual específicos dos exemplos.
- Correção: "It's not the same, can you explain me the differences?"
- Lição: ao replicar uma referência visual aprovada pelo Mario, decompor e preservar elementos observáveis concretos, incluindo tipo de imagem, tratamento gráfico, enquadramento, proporção, tipografia, cor, cenário, pose e nível de produção; não substituir a referência por um arquétipo amplo como talking head ou conteúdo humano.
- Status: APPLIED


## 2026-08-19T14:20:00Z | MUSE/visual-pattern replication
- O que fiz: criei um mockup com headshot estático, fundo desfocado e texto de Reel, usando uma abstração superficial da referência em vez de medir e reproduzir o sistema visual real das peças enviadas.
- Correção: "It's still not the same as the models I sent you. How do you analyze and see what we need to have the same pattern?"
- Lição: antes de gerar um exemplo baseado em referências visuais, analisar os assets exatos em uma matriz comparativa e extrair regras mensuráveis de formato, origem da imagem, layout, tipografia, escala, posição, cor, textura, edição e recorrência; só produzir depois de validar que o template cobre o padrão dominante, sem usar aproximações como headshot desfocado.
- Status: APPLIED
