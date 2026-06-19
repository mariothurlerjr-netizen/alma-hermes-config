# Telegram agent topics for ALMA

Use when Mario asks why agents like IRIS/MUSE are not in the `Alma AIOS` Telegram space, or asks to create agent channels/topics.

## Decision tree

1. Check available messaging targets first with `send_message(action="list")`.
   - If only `telegram:Mario (dm)` appears, Hermes cannot address the `Alma AIOS` chat yet.
   - Do not claim topics were created until a target/chat_id/thread_id is visible or verified.
2. Distinguish Telegram object type:
   - Channel: bot can post if admin, but cannot create per-agent forum topics.
   - Supergroup/forum with Topics enabled: bot can create topics if admin with topic-management permission.
   - DM with Topics enabled: Hermes can create DM topics via Bot API 9.4+ if the user enabled Topics in the bot DM.
3. Required setup for operational agent lanes:
   - Preferred group name is `Alma Agents` when Mario is separating operational lanes from the historical `Alma AIOS` space.
   - Add Jarvis Bot to the target supergroup/forum.
   - Give admin permission, including manage topics.
   - Send a message there mentioning the bot or run `/sethome` in that chat/topic so Hermes receives an update and learns the route.
4. If `send_message(action="list")` still only shows `telegram:Mario (dm)`, do not proceed as if the group is routable. Tell Mario exactly which trigger is missing: bot admin/topic permissions and a group message such as `/sethome` or an @mention.
5. After the target is visible, create or route per-agent topics/lanes for ORION, Sentinel, IRIS, MUSE, Shield, LANCE, CLAIRE, Clara, AURA, Scribe, Granola, Cortex and Hermes as appropriate.
6. Treat `@alma_aios_bot` as a possible phase-2 dedicated bot/profile, not a prerequisite for using `Alma Agents` with the current Jarvis/Hermes gateway.

## Pitfall

Do not infer that old ORION/Sentinel lanes prove current Hermes gateway access. They may have been created manually or by an older config. Current routing is authoritative.