---
name: routine-healthcheck
description: >-
  Use when a weekly healthcheck should audit routines across the bot fleet for
  failures, idle runs and wasted quota, and propose trims, staying silent when
  everything is healthy.
---
# Routine healthcheck

Data sources (use these only):
- `/home/box/agent-data/search-index.db` (messages / conversation activity; there is no ReadTranscript tool and no agent-transcripts folder)
- the current `<automation_status>` / routine status for this bot and teammates when available
- `/home/box/tools/ai-run/log.jsonl` for handoff volume and exit codes

1. Cheap check first: tally the last 7 days of routine runs per bot (count, failed, ran with no output) from search-index.db, and ai-run calls per bot from the log with one `jq` or Python one-liner.
2. Flag only: routines that failed 2+ times, routines that ran but produced nothing yet did not exit early, routines doing heavy work in Grok instead of `ai-run`, ai-run retry chains (exit≠0 then write→short→code→fast), and duplicate or overlapping routines across bots.
3. If nothing is flagged, end silently.
4. Otherwise send one short report in Chinese: a small table (bot, routine, runs, failures, problem) and at most 3 proposed changes, then a widget asking which to apply (multiSelect). Times in the user's local zone.
5. Never pause, delete or edit any routine, or change another bot, until the user picks.
6. Keep Grok usage low: summary reads only; long analysis goes to `ai-run short`.
