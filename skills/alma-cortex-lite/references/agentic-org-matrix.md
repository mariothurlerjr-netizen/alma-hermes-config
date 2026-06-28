---
created_by: hermes
created_at: 2026-06-28T00:00:00Z
source: session-review
---

# Agentic org matrix

Use this as the base template when the company is modeled as roles occupied by agents.

## Canonical layers
- Control, health, and system integrity
- Revenue, acquisition, and conversion
- Product pipeline, diagnosis, and delivery
- Billing, recovery, and retention
- Infra and message/queue glue

## Per-agent scorecard
- Mission
- Inputs
- Outputs
- Decisions allowed without escalation
- Escalation triggers
- KPI
- Common failure mode

## Handoff rules
- Each handoff must name source agent, target agent, trigger, and expected output.
- Avoid overlap by assigning a single owner per KPI.
- Dormant or paused agents still remain in the map, but should not receive active responsibilities until explicitly re-enabled.

## Useful operating questions
- What is the agent responsible for?
- What evidence proves it did the job?
- What happens when it cannot decide?
- Which upstream agent feeds it and which downstream agent consumes its output?
- Is this a real operating role or only a machine step in the pipeline?
