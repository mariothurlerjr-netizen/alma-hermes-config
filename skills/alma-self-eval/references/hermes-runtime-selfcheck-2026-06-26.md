# Hermes runtime selfcheck, 2026-06-26

## Trigger
Mario reported that Hermes was bugging after a memory increase around the 2.5GB range. The key correction was that “memória cheia” referred to runtime/process memory or configured memory ceiling, not only the persistent `MEMORY.md` / `USER.md` char budget.

## Durable lesson
For Hermes instability, diagnose three separate planes before answering:

1. **Runtime memory**
   - Check OS memory and gateway process RSS.
   - Treat a 2.5GB-ish ceiling as runtime/process memory unless proven otherwise.
   - Watch for RSS growth and restart loops.

2. **Model/context health**
   - Inspect gateway logs for repeated `context-overflow`, `compression exhaustion`, `Auto-resetting session`, `Skipping transcript persistence`, and `SIGTERM`.
   - Context overflow can look like “the bot is alive but every short message fails”.
   - Compacted summaries can preserve malformed setup state and keep re-poisoning fresh turns.

3. **Persistent memory budget**
   - Separately check `MEMORY.md` and `USER.md` char usage.
   - Do not say memory is healthy only because these files are under budget if runtime RSS or context is failing.

## Watchdog pattern
Install a script-only/no-agent watchdog for this class of failure:

- silent on healthy runs, no LLM tokens;
- alerts only on new log failures, RSS threshold breach, or persistent memory budget breach;
- stores baseline state so historical failures are not repeatedly reported;
- uses character offsets or robust file-position tracking for UTF-8 logs, because byte offsets from `stat().st_size` can overshoot a decoded Python string and cause false historical re-scans;
- keeps thresholds explicit, e.g. warn near 1.8GB RSS and critical near 2.3GB when the practical ceiling is around 2.5GB.

## User-facing response pattern
When Mario asks for autoconferência:

1. Run live checks first.
2. Install or verify the watchdog immediately.
3. Report current health in compact bullets:
   - RAM/RSS state;
   - context/compaction log state;
   - `MEMORY.md` / `USER.md` usage;
   - watchdog path, cadence, next run.
4. If adding memory about the correction, word it declaratively and avoid turning transient thresholds into universal laws.
