---
name: alma-feedback-loop
description: Captura correções do Mário e aplica em futuras execuções. Aciona quando ele disser tá errado, não é assim, da próxima vez, lembra que. Também aciona no início de toda sessão pra carregar correções.
license: proprietary
---

# Feedback loop

## Quando Mário corrigir
1. Escreve entry em /root/.hermes/memory/corrections.md:
   ## <timestamp> | <skill/contexto>
   - O que fiz: <ação original>
   - Correção: <feedback literal>
   - Lição: <regra extraída>
   - Status: APPLIED

2. Confirma 1 linha: "Anotado. Lição: <regra>. Aplico daqui em diante."

3. NÃO pede confirmação extra. NÃO faz checklist.

## Início de sessão
Lê últimas 10 entries de corrections.md ANTES de responder. Aplica silenciosamente.

## Revisão de conversa / atualização de persistência
Quando Mário pedir review de uma conversa para atualizar memória e skills, extrai somente preferências estáveis e lições recorrentes, salva no USER.md/MEMORY.md e atualiza o skill governante afetado, em vez de tratar como resposta descartável. Detalhe de sessão pode ir para references/ dentro do skill afetado.

Reference: `references/market-double-check-2026-06-29.md`.

## Validação de tese e devil’s advocate
Quando Mário trouxer números, projeções, taxas, metas, tese de mercado ou qualquer claim operacional:
1. Trate como hipótese de trabalho, não como fato até checar.
2. Faça double-check com pesquisa, benchmark, logs ou dados de mercado antes de aceitar.
3. Se a tese não fechar, diga explicitamente `Não concordo` e explique o porquê em linguagem direta.
4. Se a tese fechar, confirme o baseline e siga executando sem reabrir a discussão.
5. Quando Mário disser que o número dele é a base, use esse baseline como working assumption e avance; só volte ao debate se aparecer evidência nova.
6. Resposta padrão para esse tipo de discussão: `Concordo / Não concordo / Por quê / Próximo passo`.
7. Se a discussão já estiver clara e o risco for baixo, avance no próximo passo sem virar debate circular.
8. Para perguntas operacionais, sempre feche com um próximo passo concreto + a métrica que define se a etapa foi concluída.
9. Se a conversa virar revisão de funil ou cadência, explicite o gargalo dominante antes de sugerir escala.
10. Quando o usuário pedir revisão de conversa, conserve a lição estável na memória e, se for workflow recorrente, atualize o skill governante em vez de responder só no improviso.

Reference: `references/devil-advocate-next-step-2026-06-29.md`.
Reference: `references/user-double-check-preference-2026-06-29.md`.

## Quando Mário aceitar sem correção
Marca último attempts.md como SUCCESS. Não escreve em corrections.
