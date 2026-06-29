# Agent role map, ORION/IRIS/Instagram health

## Classification rules
- **Granola** is software/integration, not an agent.
- **IRIS** is the reporting/digest agent, its canonical job is daily + weekly digest of the vault and operational summaries.
- **ORION** is the lead-generation agent and should be evaluated by live throughput, not by stale STATUS counts.

## Operational priorities
- **ORION**: keep workers running, generate leads, and measure throughput from live data when the user asks for counts.
- **Instagram monitoring**: evaluate account health monthly, and consider an account healthy only if it has had **no spam in the last 7 days**.
- **IRIS**: summarize state, digests, and anomalies; do not reclassify it as a sales or outreach agent unless the user explicitly changes the role.

## Reporting guidance
- If the vault has no reliable forecast for leads today/tomorrow/this week, say so explicitly instead of inventing numbers.
- Separate **agent**, **software/integration**, and **monitoring rule** into different blocks when presenting priorities.
