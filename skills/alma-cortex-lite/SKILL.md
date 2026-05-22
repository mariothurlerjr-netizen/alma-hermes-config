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
1. Lê `wiki/hot.md` se existir; se o arquivo não existir, usa `STATUS.md` + busca específica no brain. Não trava nem inventa hot file.
2. Lista 3 items mais quentes de Open Loops + Active Pipeline.
3. Recomenda 1 com justificativa.
4. Executa se autonomia clara, pergunta confirmação curta caso contrário.

## Debug técnico em agentes/APIs

Referência de sessão: `references/orion-apollo-422.md`.

1. Quando um erro externo vem só como status code (`422`, `400`, `403`, etc.), reproduz a chamada mínima e lê o body da resposta antes de patchar lógica. Em Apollo, `422` pode ser quota/credits exhausted, não payload inválido.
2. Diferencia falha recuperável de condição terminal. Exemplo ORION: Apollo `insufficient credits` deve parar o worker com status terminal (`RestartPreventExitStatus=42`) em vez de loopar a cada heartbeat.
3. Depois de patchar worker systemd, verifica se o processo carregou o código novo. Se `systemctl stop`/SIGTERM fica preso por sleep handler, confirma PID; se necessário, mata o PID antigo e observa restart/exit esperado.
4. Verificação mínima: `py_compile`, diff do arquivo, status systemd, log tail com a causa legível.
