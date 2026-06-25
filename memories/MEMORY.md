Legacy ORION workers remain intentionally paused since 2026-06-13 (`fc52bbc58989` paused, `alma-orion@1..5` disabled), but controlled ORION Lead Factory cron `5cd7d23f5197` is active in CSV-only mode for ALMA Rev lead production.
§
ORION watchdog cron `fc52bbc58989` uses rolling `delivered_24h`; `/root/.hermes/scripts/orion_watchdog.py` treats flat or decreasing delivery counts as KPI movement, not worker-health failure. Hard failures are missing/stale/erroring workers or failed repairs.
§
LANCE paused intentionally; `alma-lance.timer` disabled/stopped, `alma-lance.service` inactive since 2026-05-24.
§
Na VPS do Mario, `hermes-gateway` roda como user service de root (`XDG_RUNTIME_DIR=/run/user/0 systemctl --user ...`); não há Unix user `almarev`, então units ALMA não devem usar `User=almarev` sem criar o user.
§
Hermes TTS uses `elevenlabs`; reminder calls need Twilio env + `HERMES_REMINDER_TO_PHONE`/`MARIO_PHONE`. CLAIRE/ALMA Local cold-base dialing stays paused until Mario explicitly approves; do not auto-resume from “segunda” alone. Accepted callback follow-ups are allowed.
§
Reminder handling: if Mario asks in shorthand/noisy text to send or remind something at a time, schedule a durable Telegram reminder when the time can be inferred; ask only for missing reminder text or genuinely impossible timing, and state explicitly if no job was created.
§
Mario has a personal Apollo account with roughly 2.5k credits available as a potential ORION credit source; do not paste Apollo secrets in chat, use secret-safe env update on the VPS.
§
ALMA Rev Instantly outbound requires `insert_unsubscribe_header=true` and visible unsubscribe footer/link in every email step: `https://almarev.com/api/v1/public/unsubscribe?email={{email}}`.
§
ALMA Rev ops: Instantly handles inbox warmup; ORION only does lead-gen/enrichment. Cap 25-30/day per warmed inbox; post-E3 -> CLAIRE; no-phone -> ORION.
§
ALMA Rev sender separation: `mario@almarev.com` is reserved for owned/warm newsletter-style communication; cold/outbound should use warmed `@getalmarev.com` sender accounts, especially `mario@getalmarev.com`, once warmed.