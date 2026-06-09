# ORION, trocar Apollo key para conta com créditos pessoais

Use quando ORION estiver ativo mas logs mostrarem `Apollo credits exhausted` / `insufficient credits`, e Mario oferecer uma conta Apollo alternativa com créditos.

## Princípios

- Não pedir nem aceitar secret colado no chat/Telegram.
- Só precisa da key Apollo, o e-mail da conta não importa para o runtime.
- ORION lê `/home/almarev/agentic/.env` via `EnvironmentFile` em `alma-orion@.service`.
- Antes de alterar env, sempre fazer backup timestampado.
- Depois da troca, reiniciar `alma-orion@*.service` e validar por logs e output real.

## Procedimento seguro

Use concatenação no script para evitar que ferramentas de log/mascara confundam a linha de env com secret exposto.

```bash
ssh root@72.60.136.186

read -s NEW_APOLLO_KEY
export NEW_APOLLO_KEY

cp /home/almarev/agentic/.env /home/almarev/agentic/.env.bak.apollo-$(date +%Y%m%d-%H%M%S)

python3 - <<'PY'
import os
from pathlib import Path

p = Path('/home/almarev/agentic/.env')
key = os.environ['NEW_APOLLO_KEY']
prefix = 'APOLLO_' + 'API_' + 'KEY='

lines = p.read_text().splitlines()
out = []
seen = False
for line in lines:
    if line.startswith(prefix):
        out.append(prefix + key)
        seen = True
    else:
        out.append(line)
if not seen:
    out.append(prefix + key)

p.write_text('\n'.join(out) + '\n')
PY

unset NEW_APOLLO_KEY
systemctl restart 'alma-orion@*.service'
```

## Validação mínima

```bash
systemctl list-units 'alma-orion@*.service' --all --no-pager
journalctl -u 'alma-orion@*.service' -n 120 --no-pager | sed -E 's/(x-api-key|api[_-]?key|token)[=: ]+[^ ]+/\1=***MASKED***/Ig'
python3 - <<'PY'
from pathlib import Path
for i in range(1, 6):
    p = Path(f'/home/almarev/brain/agents/orion/logs/instance-{i}.jsonl')
    if p.exists():
        print('--- instance', i)
        print('\n'.join(p.read_text(errors='ignore').splitlines()[-8:]))
PY
```

Procure ausência de `insufficient credits`, busca Apollo retornando firms, e evidência de lead/score/delivery real. Se ainda aparecer quota depleted, a key nova não foi aplicada, está sem créditos, ou o processo não reiniciou com env novo.
