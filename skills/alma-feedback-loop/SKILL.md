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

## Quando Mário aceitar sem correção
Marca último attempts.md como SUCCESS. Não escreve em corrections.
