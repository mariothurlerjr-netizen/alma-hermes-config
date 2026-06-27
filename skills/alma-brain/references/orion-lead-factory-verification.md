# ORION Lead Factory verification notes

## Current verifier preference
- Mario moved the verifier/enrichment preference from MillionVerifier/MailVerifier to **mails.so**.
- Treat this as the current default for lead verification/enrichment until Mario changes it again.

## Operational shape
- ORION lead search should be reasoned about separately from verifier choice.
- When checking status, distinguish:
  - cron/job paused or running
  - worker/code actually active
  - lead source stage, Apollo search/filter, enrichment, dedupe
  - verifier/enrichment provider choice

## Useful rule of thumb
- Apollo first, then only verify/enrich leads that already passed ICP.
- Do not spend verifier credits on obvious rejects or duplicates.
- If the verifier changes again, update the class-level skills, not just one-off memory.