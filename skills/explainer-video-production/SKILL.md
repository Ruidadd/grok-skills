---
name: explainer-video-production
description: >-
  Use when asked to make an explainer, history, biography, or tech-principle
  animation video with narration and subtitles.
---
# Explainer video production
1. Clarify only what changes the result: topic, length, aspect ratio (16:9 / 9:16), style, language. Otherwise decide and start.
2. Research and fact-check; write a scene-by-scene storyboard with narration lines and timings. Report contested claims as contested.
3. Visuals: prefer a polished single-file HTML/CSS/JS/SVG animation (optionally written by an external LLM the owner has approved) with a seek hook like window.__go(t); record it deterministically frame by frame with headless Chrome + Playwright piped to ffmpeg. 3D (Blender) only when truly needed; budget render time first.
4. Real photos only from public-domain or CC sources, credited on screen and listed in sources.txt.
5. Audio: TTS narration aligned per scene (speed up slightly rather than overrun), light background music ducked under narration.
6. Export H.264 High yuv420p + AAC-LC + faststart, under the chat attachment limit (aim <100MB; offer split parts). Fully decode-check and inspect sample frames before sending.
7. Reliability: max 2 heavy processes at once, run long jobs with nohup/setsid in resumable chunks, poll briefly.
8. Deliver the video, 2 sample frames, and an honest list of compromises. Never publish anywhere without the owner's review.
