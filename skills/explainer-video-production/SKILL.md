---
name: explainer-video-production
description: >-
  Use when asked to make an explainer, history, biography, tech-principle, or
  research video (YouTube 16:9 or Shorts 9:16). Covers confirmation page,
  storyboard, Remotion/HTML/Blender, TTS, and review before delivery.
---
# Explainer video production
All visuals in a video must follow the visual-quality-pipeline skill (/home/box/agent-data/workflows/visual-quality-pipeline/SKILL.md). Read it before the formal build. It covers planning, code-drawn numbers, a shared tokens.json, real animation at 1080p or above, and 2-3 rounds of frame review. Where the two skills conflict, it wins.

Model calls: never call claude or codex directly; always go through the shared ai-run skill (`ai-run <mode> --bot <bot name> < prompt.md`).
- `ai-run plan` for confirmation pages, storyboards, and scripts.
- `ai-run code` (Codex) for all code: Remotion, HTML animation, Blender Python, and slide code.
- `ai-run review -i <frames>` for rendered-frame review.
- `ai-run write` for narration copy.

## Circuit breaker (mandatory)
In one run of a video task or routine: if the first `ai-run` call exits non-zero, stop immediately. Do not chain write→short→code→fast (or any other mode) as a retry. Fix the cause or report the failure; only a new user ask (or a new routine fire) starts a fresh run.

## Step 0 — Confirmation page (new videos only)
Skip this only for small edits to an already-approved piece (fix tweak, one-shot fix, re-encode). For any new video:
1. Generate a confirmation page with `ai-run plan` (Grok reads only the summary). Include: topic angle, target duration, storyboard outline (one sentence + shot description per segment), 2–3 style keyframes, and voiceover style.
2. Deliver that page to the user and wait for their red-pen review and explicit OK.
3. Only after confirmation enter visual-quality-pipeline formal production.

## Production steps
1. Clarify only what changes the result: topic, length, aspect ratio (16:9 / 9:16), style, language. Otherwise decide and start (after Step 0 when required).
2. Research and fact-check; write a scene-by-scene storyboard (`ai-run plan`) with narration lines and timings. Report contested claims as contested. Budget narration at about 4 Chinese characters per second.
3. Visuals: build per visual-quality-pipeline with `ai-run code`. Remotion is preferred. An HTML animation with a seek hook like window.__go(t), recorded frame by frame with headless Chrome + Playwright, is acceptable when it meets the same bar. For tech-principle topics, consider the Anatomy skill (github.com/wheresryan22/anatomy) as the visual layer. Use 3D (Blender) only when truly needed, and budget render time first.
4. Real photos only from public-domain or CC sources, credited on screen and listed in sources.txt.
5. Audio: TTS narration aligned per scene (speed up slightly rather than overrun), light background music ducked under narration.
6. Export H.264 High yuv420p + AAC-LC + faststart, under the chat attachment limit (aim <100MB; offer split parts). Fully decode-check, then review sample frames with `ai-run review -i` and revise 2-3 rounds before sending.
7. Reliability: max 2 heavy processes at once, run long jobs with nohup/setsid in resumable chunks, poll briefly.
8. Deliver the video, 2 sample frames, and an honest list of compromises. Never publish anywhere without the owner's review.
