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
| Técnica, código, VPS | Direto | Opera no repo ativo correto; para ALMA Agentic use `/home/almarev/agentic` |

## Regra de ouro
Para ALMA Agentic operacional, o repo ativo é `/home/almarev/agentic`. Antes de editar, confirma contexto no brain quando a tarefa menciona entidade ALMA; depois executa e valida no serviço/DB/health correspondente.

## Pedido vago
1. Lê `wiki/hot.md` se existir; se o arquivo não existir, usa `STATUS.md` + busca específica no brain. Não trava nem inventa hot file.
2. Lista 3 items mais quentes de Open Loops + Active Pipeline.
3. Recomenda 1 com justificativa.
4. Executa se autonomia clara, pergunta confirmação curta caso contrário.

## Debug técnico em agentes/APIs

Referências de sessão:
- `references/orion-apollo-422.md` — diagnosticar Apollo `422` por body e não por status.
- `references/orion-apollo-free-fallback.md` — manter ORION entregando leads sem créditos Apollo usando Tavily + enriquecimento posterior.

1. Quando um erro externo vem só como status code (`422`, `400`, `403`, etc.), reproduz a chamada mínima e lê o body da resposta antes de patchar lógica. Em Apollo, `422` pode ser quota/credits exhausted, não payload inválido.
2. Não trate bloqueio externo como resposta final se existe rota substituta. Se Mario disser que o objetivo é resolver, você é responsável por manter o pipeline produzindo: implemente fallback degradado, rode, valide entrega real e só então reporte tradeoffs.
3. Diferencia falha recuperável, condição terminal e modo degradado. Exemplo ORION: Apollo `insufficient credits` não deve loopar nem necessariamente parar tudo; pode acionar fallback Tavily candidate search + Browserbase/Snov enrichment enquanto créditos não voltam.
4. Depois de patchar worker systemd, verifica se o processo carregou o código novo. Se `systemctl stop`/SIGTERM fica preso por sleep handler, confirma PID; se necessário, mata o PID antigo e observa restart/exit esperado.
5. Verificação mínima: `py_compile`, diff do arquivo, status systemd, health HTTP, log tail com a causa legível e evidência de output real (lead no DB/brain, não só serviço running).
