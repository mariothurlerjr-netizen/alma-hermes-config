---
name: alma-self-eval
description: Avalia se ações deram resultado e troca estratégia em padrão de falha. Aciona ao final de toda ação significativa e no heartbeat pra varrer attempts.md.
license: proprietary
---

# Self-eval

## Após cada ação
Registra em /root/.hermes/memory/attempts.md:
## <timestamp> | <task type>
- Ação: <o que fiz>
- Abordagem: <estratégia>
- Outcome: SUCCESS | FAILURE | UNKNOWN
- Sinal: <evidência>
- Próximo: <continuar | trocar | escalar>

## Outcome
- SUCCESS: Mário aceitou, agradeceu, usou sem mudança
- FAILURE: corrigiu, descartou, reclamou
- UNKNOWN: sem sinal claro (não inventa SUCCESS)

## Detecção padrão (no heartbeat)
3+ FAILURE consecutivas no mesmo task type:
- Escreve patterns.md com flag SWITCH_STRATEGY
- Heartbeat avisa Mário próximo ciclo
- Próxima execução usa abordagem alternativa

## Autoconferência de runtime Hermes
Quando Mário reportar que “você bugou”, “memória cheia”, gateway reiniciando, respostas travando, ou loop de compaction/contexto:
1. Separar explicitamente três camadas antes de diagnosticar:
   - memória de processo/RSS/limite em GB;
   - overflow de contexto/modelo/compaction;
   - memória persistente `MEMORY.md`/`USER.md` por char budget.
2. Não assumir que “memória cheia” significa `MEMORY.md`. Conferir RSS/gateway logs/config e só então responder.
3. Procurar sinais no log: `context-overflow`, `compression exhaustion`, `Auto-resetting session`, `Skipping transcript persistence`, `SIGTERM` repetido.
4. Se houver padrão repetido, criar ou verificar watchdog script-only/no-agent que fique silencioso quando saudável e alerte só em regressão.
5. Baseline do watchdog deve ignorar histórico antigo e monitorar apenas novos eventos. Para logs UTF-8, usar offset por caractere ou inode+posição segura, não `stat().st_size` como offset de string.
6. Preferir alertas objetivos com threshold de RSS, frequência de resets e budgets de memória persistente.

Referência: `references/hermes-runtime-selfcheck-2026-06-26.md`.

## Trocar estratégia
Quando SWITCH_STRATEGY ativa:
- Pergunta direto: "tô errando em X, qual a abordagem certa?"
- Pesquisa alma-brain por casos similares
- Tenta abordagem oposta
