# Telegram inter-bot conversations for ALMA Hermes profiles

Use this when Mario wants ALMA agent bots to talk to each other in a Telegram group/forum.

## Durable pattern

1. Confirm runtime first:
   - `hermes profile list` must show each profile gateway `running`.
   - `systemctl --user --state=failed list-units 'hermes-gateway@*.service'` must be empty.
   - Validate each bot token with `getMe` without printing tokens.
2. Confirm group membership with Telegram `getChatMember(chat_id, user_id=<bot_id>)` for each bot. A profile running locally is not proof the bot is in the group.
3. Authorize bot senders explicitly:
   - keep Mario's Telegram user ID in `TELEGRAM_ALLOWED_USERS`.
   - append the ALMA bot IDs to `TELEGRAM_ALLOWED_USERS` for every participating profile.
   - set `TELEGRAM_ALLOW_BOTS=mentions` for every participating profile.
4. Keep human access closed. Do not use `TELEGRAM_ALLOW_ALL_USERS=true` for this pattern.
5. Conversation path is shared group/forum + explicit @mentions. Telegram bot-to-bot DM is not reliable/supported as an operational bus.
6. If topics/lanes are needed, check `can_manage_topics` from `getChatMember`. Admin status alone is not enough. `can_manage_topics=false` still allows conversation, but blocks create/rename-topic automation.
7. For production-grade autonomous cross-agent work, prefer Hermes Kanban or webhooks as the durable bus, with Telegram as the human-visible control surface.

## Smoke test

Send one message into the group/topic from an existing routable Hermes target:

`@<target_bot_username> responda apenas <PROFILE>_OK se recebeu esta mensagem do bot Hermes.`

Then verify:
- the target profile session/search/logs show the incoming bot-authored message;
- the group receives the expected response;
- no unrelated bot responds, which confirms mention routing is not too broad.

## Implementation note

Hermes must preserve Telegram `message.from_user.is_bot` into `SessionSource.is_bot` for `TELEGRAM_ALLOW_BOTS=mentions` to work at the gateway authorization layer. If future code refactors the Telegram adapter, keep this propagation covered by a regression test.