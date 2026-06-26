---
created_by: hermes
created_at: 2026-06-26T00:00:00Z
source: telegram
---

# Email base hygiene / verification

## Durable preference
- Mario prefers Mails.so as the default verifier for lead-base hygiene.
- Treat Snov and other older verifier stacks as retired for this workflow unless Mario explicitly reopens them.

## Operating rule
- Verification is a preprocessing step, separate from sender warmup and separate from deliverability monitoring.
- Use verification to split records into:
  - valid
  - risky / catch-all
  - invalid
  - unknown
- Do not treat a verified base as ready to send if sender warmup or inbox health is not clean.

## Practical note
- Prefer the cheaper/direct verifier when the goal is base hygiene, not enrichment.
- Keep verification and sending on separate decision paths so provider changes do not affect outbound logic.

## Integration notes
- Mails.so accepts the `x-mails-api-key` header.
- In code, accept common env aliases so operators can rotate secrets without changing call sites:
  - `MAILSSO_API_KEY`
  - `MAILS_API_KEY`
  - `MAILS_SO_API_KEY`
  - `X_MAILS_API_KEY`
- If Mails.so is configured and returns a provider error, do not silently fall back to an older verifier; fail fast so the operator sees the real account/billing problem.
- A `402 Payment Required` from Mails.so is a provider/account state issue, not a transport bug.
- MillionVerifier already exists in the stack as a separate adapter and can be used as the next verifier in a waterfall when Mails.so is unavailable or intentionally swapped out.

## Why this matters
- Keeps outbound experiments from polluting the warm sender set.
- Lets copy testing and list hygiene proceed independently.
- Reduces the chance of wasting send volume on obviously bad records.
