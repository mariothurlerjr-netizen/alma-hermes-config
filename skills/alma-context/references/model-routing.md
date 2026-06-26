# Model routing

Use this note when the user asks about selecting lighter models for normal chat and stronger models for execution-heavy work.

## Core distinction
- **Conversation model**: optimize for latency, readability, and low-friction back-and-forth.
- **Execution model**: optimize for deeper reasoning, code changes, multi-step plans, and tasks where correctness matters more than speed.
- **Profile/provider selection**: the active Hermes profile can pin a provider/model; a task or worker may also override it. Do not assume the visible chat model is the same as the execution model.

## Operating rules
- Prefer a lighter model for ordinary dialogue, synthesis, and short coordination turns when the task does not need heavy reasoning.
- Prefer a stronger model for implementation, debugging, architecture, and any task that may need long context or multi-step planning.
- If the user asks whether the model can be selected, answer explicitly that yes, model choice is configurable at the profile/workflow level; then state the current active model if known.
- When describing model behavior, separate **what is currently configured** from **what is recommended**.
- Before claiming a model is active, verify the live config or runtime state rather than inferring it from memory.

## Pitfall
- Do not collapse “chat model”, “executor model”, and “profile default” into one thing. That confusion causes wrong advice about cost, speed, and where to change settings.
