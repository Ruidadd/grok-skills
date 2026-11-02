---
name: transcript-healthcheck
description: >-
  Use when a scheduled healthcheck should scan the last day of bot conversations
  for recurring friction (errors, retries, user corrections, stuck tools) and
  propose fixes, staying silent when nothing is found.
---
# Transcript healthcheck

Data source: the ReadTranscript tool (cursor namespace). Do not look for a transcripts folder on disk; none exists.

1. Cheap check first: use ReadTranscript to list conversations of this bot and its teammate bots active in the last 24 hours. If none, end silently with no message.
2. For each active conversation, read only the recent part and note friction: tool errors, repeated retries, the user correcting the bot, a bot saying it can't do something, a routine run that failed or ran with no output.
3. Group findings by root cause. Ignore one-off noise; keep only issues seen twice or with real user impact.
4. If nothing qualifies, end silently.
5. Otherwise send the user one short report in Chinese: at most 3 issues, each with the bot, what happened (with time in Asia/Tokyo), the likely cause, and one concrete fix. End with a widget asking which fixes to apply.
6. Never apply a fix, change another bot, or create anything until the user picks.
7. Keep Grok usage low: no deep re-reading, no web research; hand any long analysis to `ai-run short` and read only its summary.
