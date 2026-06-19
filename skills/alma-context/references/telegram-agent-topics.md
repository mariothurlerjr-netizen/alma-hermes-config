# Telegram agent topics for ALMA

Use when Mario asks why agents like IRIS/MUSE are not in the `Alma AIOS` Telegram space, or asks to create agent channels/topics.

## Decision tree

1. Check available messaging targets first with `send_message(action="list")`.
   - If only `telegram:Mario (dm)` appears, Hermes cannot address the `Alma AIOS`/`Alma Agents` chat yet.
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

## Creating forum topics directly via Telegram Bot API

When `send_message(action="list")` shows `telegram:Alma Agents / topic 1 (group)` or `channel_directory.json` shows a target like `-100...:1`, Hermes can use the Telegram Bot API directly to create topics if the bot is admin with manage-topics permission.

Safe procedure:

1. Load `TELEGRAM_BOT_TOKEN` from `~/.hermes/.env` without printing it.
2. Strip the topic suffix from the target to get the base chat id: `telegram:-1003702982318:1` -> chat_id `-1003702982318`.
3. Verify forum status with `getChat(chat_id)` and require `type=supergroup`, `is_forum=true`.
4. Call `createForumTopic` once per lane:
   - `chat_id=<base chat id>`
   - `name=<topic title>`
5. Capture `result.message_thread_id` for each created topic.
6. Seed each topic with one short purpose message via `sendMessage(chat_id, message_thread_id, text)`.
7. Send a compact index to the general topic (`message_thread_id=1`) using `send_message` or Bot API.
8. If a bulk script times out, do not assume total failure. Probe with one small `createForumTopic` or inspect the successful API response before retrying. Retrying blindly can create duplicate topics.

Minimal Python pattern:

```python
from pathlib import Path
import os, json, urllib.request, urllib.parse

for line in Path('/root/.hermes/.env').read_text(errors='ignore').splitlines():
    if '=' in line and not line.strip().startswith('#'):
        k, v = line.split('=', 1)
        os.environ.setdefault(k.strip(), v.strip().strip('"').strip("'"))

token = os.environ['TELEGRAM_BOT_TOKEN']
chat_id = '-1003702982318'
base = f'https://api.telegram.org/bot{token}/'

def post(method, payload, timeout=20):
    data = urllib.parse.urlencode(payload).encode()
    req = urllib.request.Request(base + method, data=data)
    with urllib.request.urlopen(req, timeout=timeout) as r:
        return json.loads(r.read().decode())

chat = post('getChat', {'chat_id': chat_id})
assert chat['result']['type'] == 'supergroup'
assert chat['result'].get('is_forum') is True

topic = post('createForumTopic', {'chat_id': chat_id, 'name': 'IRIS - Metrics'})
thread_id = topic['result']['message_thread_id']
post('sendMessage', {
    'chat_id': chat_id,
    'message_thread_id': thread_id,
    'text': 'IRIS: metrics, dashboards, KPIs, digests, anomaly detection and cost/performance reporting.',
})
```

## Recommended ALMA Agents lanes

Use these as the default operational forum topics unless Mario changes the roster:

- `ORION - Lead Gen`: lead generation B2B, ICP lists, sourcing, scoring, delivery of qualified accounts.
- `LANCE - Outbound`: outbound sequencing, cold email/LinkedIn flow, cadence health, replies and handoff.
- `CLAIRE - Voice SDR`: voice outbound, calls, callbacks, call QA, engagement signals.
- `IRIS - Metrics`: metrics, dashboards, KPIs, daily/weekly digests, anomaly detection, cost/performance reporting. Mario explicitly chose this role over sales/closer/research on 2026-06-19.
- `MUSE - Content`: content strategy and content factory, ideation, drafts, review queue, learning loop.
- `Sentinel - Signals`: external signals, news, market events, CRM/prospect triggers and monitoring.
- `Shield - Health`: system health, compliance checks, guardrails, incident and risk alerts.
- `AURA - Assessments`: public diagnostic/assessment flow, intake, report narrative, conversion bridge.
- `Clara - Interviews`: interview agent, async stakeholder questions, WhatsApp/SMS conversation flow.
- `Scribe - Audit Intake`: Revenue Engine Audit intake, stakeholder data collection, handoff to analysis/reporting.
- `Granola - Meetings`: meeting notes import, summaries, action extraction and knowledge capture.
- `Cortex - Orchestration`: backend orchestration, agent routing, mission planning, cross-agent execution.
- `Hermes - Ops`: operator/meta-agent lane, gateway, memory, cron, infra, routing, debugging, coordination.
- `NOVA - Proposals`: proposals, sales enablement, commercial assets and closing handoff.

## Pitfalls

- Do not infer that old ORION/Sentinel lanes prove current Hermes gateway access. They may have been created manually or by an older config. Current routing is authoritative.
- Do not treat `/analytics` being labeled Marketing as evidence that IRIS owns marketing. Marketing lanes are LUNA/MUSE/SOL/ALEXANDER; IRIS owns metrics/dashboard/reporting by Mario's current decision.
- `createForumTopic` requires the base chat id, not the `chat_id:thread_id` composite from Hermes targets.
- Keep topic names short. Telegram topic titles become noisy if they include full descriptions.