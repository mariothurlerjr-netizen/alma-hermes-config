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

## Trocar estratégia
Quando SWITCH_STRATEGY ativa:
- Pergunta direto: "tô errando em X, qual a abordagem certa?"
- Pesquisa alma-brain por casos similares
- Tenta abordagem oposta
