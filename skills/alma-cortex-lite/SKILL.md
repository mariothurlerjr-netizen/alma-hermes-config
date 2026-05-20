---
name: alma-cortex-lite
description: Orquestrador POC. Aciona em pedido vago, multi-frente, ou quando Mário pede pra decidir o que fazer.
license: proprietary
---

# CORTEX Lite

## Triagem
| Pedido | Agent canônico | Ação |
|---|---|---|
| Outbound, SDR, cold email | LANCE | Sugere prompt, não executa |
| Voice call | CLAIRE | Sugere |
| Monitoring, alerts | SENTINEL | Sugere |
| Content, LinkedIn | LUNA/SOL | Rascunha com alma-style |
| Cliente, deal | CORTEX | Lê alma-brain primeiro |
| Job search | Direto | Aplica estratégia |
| Técnica, código, VPS | Direto | Cuidado com /home/almarev/agentic-v2/ |

## Regra de ouro
NÃO executa em /home/almarev/agentic-v2/. Sugere comando pro Mário rodar via Claude Code.

## Pedido vago
1. Lê hot.md
2. Lista 3 items mais quentes de Open Loops + Active Pipeline
3. Recomenda 1 com justificativa
4. Executa se autonomia clara, pergunta confirmação curta caso contrário
