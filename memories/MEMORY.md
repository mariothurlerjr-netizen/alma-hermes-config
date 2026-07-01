ORION lead factory was resumed on 2026-06-27: `orion-watchdog` cron resumed and `alma-orion@1..5.service` plus `alma-orion-watchdog.timer` are active/enabled.
§
Na VPS do Mario, `hermes-gateway` roda como user service de root (`XDG_RUNTIME_DIR=/run/user/0 systemctl --user ...`); não há Unix user `almarev`, então units ALMA não devem usar `User=almarev` sem criar o user.
§
Hermes TTS uses `elevenlabs`; Twilio voice number for Hermes is a local São Paulo BR number, +551150281678, SID PN890f19fb4c1d2e370f6d20679da8c61e. It has Voice/Fax/SIP capabilities, messaging is unavailable, and inbound call webhooks point to ElevenLabs/Twilio endpoints.
§
Reminder handling: if Mario asks in shorthand/noisy text to send or remind something at a time, schedule a durable Telegram reminder when the time can be inferred; ask only for missing reminder text or genuinely impossible timing, and state explicitly if no job was created.
§
Mario prefere mails.do no fluxo de lead verification/enrichment do Lead Factory, depois da busca ICP e antes da verificação da conta.
§
ALMA Rev Instantly outbound requires `insert_unsubscribe_header=true` and visible unsubscribe footer/link in every email step: `https://almarev.com/api/v1/public/unsubscribe?email={{email}}`.
§
For Instantly warmup screenshots, orange bars mean spam. Mario's last-4-days criterion is the previous 4 calendar days all spam-free; 3-4 days are WATCH, fewer than 3 or bad signal are HOLD.
§
ALMA Rev sender separation: the `@getalmarev.com` sending accounts live in Gmail Workspace; `mario@almarev.com` is reserved for owned/warm newsletter-style communication, and cold/outbound should use warmed `@getalmarev.com` sender accounts, especially `mario@getalmarev.com`, once warmed.
§
Hermes default profile has `security.redact_secrets=false` after Composio/snippet corruption; keep handling secrets operationally safe by never printing/pasting them, using env files or Python writers instead of chat snippets.
§
Mario clarified that “memória cheia” in the Hermes bug report can mean runtime/RSS or configured process memory around 2.5GB, not only MEMORY.md/USER.md char usage; future Hermes diagnostics must separate OS/RSS memory, model context overflow, and persistent memory char budgets.
§
Claude Code on Mario's VPS is OAuth-authenticated as Mario's Claude Max account; use `/usr/local/bin/hermes-claude` for safe one-shot delegation. Prefer OAuth/device-flow or existing local auth, never paste Anthropic/Claude secrets in chat.
§
Telegram bot @username is effectively fixed after creation; clean path for a new @ is creating a new bot with /newbot and swapping the token. Heartbeat cron on `/root/run-heartbeat.sh` auto-loads `alma-memory-loop` alongside `alma-heartbeat`, `alma-brain`, `alma-style` and `alma-context`.
§
ALMA Hermes Telegram bot fleet is preserved but inactive; per-profile gateways, tokens, SOUL/CLAUDE files, and registry remain for reactivation.
§
ALMA operating model: Hermes is the default cockpit; extra bots only when overload signals appear.
§
No perfil default do Hermes, o Composio agora está com `Tools: all` no `hermes mcp list` e `hermes mcp test composio` descobre 508 tools; `resources` e `prompts` continuam desativados.
§
Mario prefere priorizar RevOps, fechamento, oferta/pricing e higiene de pipeline antes de customer success, retenção/expansão e market intelligence pesada quando ainda não há clientes.
§
Hermes voice stack on Mario's VPS still has Retell/Vapi plumbing, but Bland AI is now the preferred call-flow path when switching away from ElevenLabs-style orchestration.
§
No perfil default do Hermes, o Telegram home channel ficou configurado em `TELEGRAM_HOME_CHANNEL=8293697290`, então DMs do Mario passam a ter destino padrão para respostas e deliveries proativos.
§
Ao citar números operacionais de contas/caixas, eu devo revalidar o inventário live antes de afirmar totais, porque o conjunto ativo pode mudar durante o dia.
§
No callback da CLAIRE, `agent_ed6f1959bf5d153b6f8bbebc06` é o candidato a `RETELL_AGENT_ID_REV_CALLBACK`; fallback para `RETELL_AGENT_ID` se vazio.
§
Para rotação Instantly/ALMA Rev, contas novas entram em 5 envios/dia; contas que já aguentaram 20/dia sem spam mas seguem em 10/dia devem ficar em 5/dia e subir +5/dia até ~35-40/dia, com revisão diária.