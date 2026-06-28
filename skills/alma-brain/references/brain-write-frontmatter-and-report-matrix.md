# Brain write frontmatter and report matrix

## Brain write pitfall
When writing to `/home/almarev/brain/agents/hermes/` with `brain_write`, the file must include frontmatter keys:
- `agent`
- `workspace`
- `timestamp`
- `type`

The `created_by`, `created_at`, and `source` fields alone are not enough for validation.

## Operational report matrix used in this session
Decision authority and reporting shape:

- Level 0: autonomous execution for routine, reversible, in-lane work
- Level 1: bounded discretion inside a defined boundary
- Level 2: escalate before acting for strategy, pricing, ICP, budget, brand, or production-risk changes
- Level 3: prohibited autonomous actions for destructive or policy-changing moves

Weekly report fields:
- what happened
- quantities / volume
- blockers
- decisions made autonomously
- escalations needed
- next-day / next-cycle plan

Daily capture fields:
- output produced
- count / volume
- blockers
- changes observed during the day
- next planned action
