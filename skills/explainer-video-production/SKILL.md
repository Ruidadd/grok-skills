---
name: explainer-video-production
description: >-
  Use when asked to make an explainer, history, biography, tech-principle, or
  research video (YouTube 16:9 or Shorts 9:16). Covers confirmation page,
  storyboard, Claude Code driving Remotion/HTML/Blender/Unity, TTS, and review
  before delivery. Grok only dispatches.
---
# Explainer video production
All visuals in a video must follow the visual-quality-pipeline skill. Read it before the formal build. It covers planning, code-drawn numbers, a shared tokens.json, real animation at 1080p or above, and 2-3 rounds of frame review. Where the two skills conflict, it wins.

## Quota (hard rule)
Grok only dispatches: take the brief, deliver the confirmation page, call `ai-run`, and return paths plus a short summary. Do not use Grok's GenerateVideo. Do not write storyboards, implementation code, or renders in the Grok turn. All production — including Blender, Remotion, Unity (only if already installed), and VO alignment — goes through `ai-run`, and Claude Code invokes the tools on this machine.

Model calls: never call claude or codex directly; always go through the shared ai-run skill (`ai-run <mode> --bot <bot name> < prompt.md`).
- `ai-run plan` for confirmation pages, storyboards, and scripts.
- `ai-run build` (Claude Code) for video / complex implementation. The build prompt must tell Claude Code to call local tools itself: `blender --background --python <script.py>` (binary `/home/box/.local/bin/blender`, install dir `/home/box/.local/blender`, 5.2.4 LTS); Remotion; HTML animation recorded with headless Chrome + Playwright; Unity only if already installed and licensed.
- `ai-run code` (Codex) only for medium scripts or static chart/slide helpers inside the pipeline.
- `ai-run review -i <frames>` for rendered-frame review (Claude built → default Codex reviewer; do not add `--reviewer claude` unless Codex built).
- `ai-run write` for narration copy.

## Circuit breaker (mandatory)
In one run of a video task or routine: if the first `ai-run` call exits non-zero, stop immediately. Do not chain write→short→code→build→fast (or any other mode) as a retry. Fix the cause or report the failure; only a new user ask (or a new routine fire) starts a fresh run.

## Step 0 — Confirmation page (new videos only)
Skip this only for small edits to an already-approved piece (fix tweak, one-shot fix, re-encode). For any new video:
1. Generate a confirmation page with `ai-run plan` (Grok reads only the summary). Include: topic angle, target duration, storyboard outline (one sentence + shot description per segment), 2–3 style keyframes, and voiceover style.
2. Deliver that page to the user and wait for their red-pen review and explicit OK.
3. Only after confirmation enter visual-quality-pipeline formal production.

## Production steps
1. Clarify only what changes the result: topic, length, aspect ratio (16:9 / 9:16), style, language. Otherwise decide and start (after Step 0 when required).
2. Research and fact-check; write a scene-by-scene storyboard (`ai-run plan`) with narration lines and timings. Report contested claims as contested. Budget narration at about 4 Chinese characters per second.
3. Visuals: build per visual-quality-pipeline with `ai-run build` (Claude Code). Pick the tool in the build prompt:
   - 2D motion graphics / UI explainers: Remotion, or HTML + `window.__go(t)` recorded frame by frame with headless Chrome + Playwright, same quality bar.
   - 3D / spatial / mechanical topics: Blender Python, rendered headless. Budget render time first.
   - Unity: only if the editor is already installed and licensed on this machine; do not install Unity during a job, and do not fall back to Grok rendering if it is missing — use Blender or Remotion instead.
   For tech-principle topics, consider the Anatomy skill (github.com/wheresryan22/anatomy) as a visual layer when it fits.
4. Real photos only from public-domain or CC sources, credited on screen and listed in sources.txt.
5. Audio: TTS narration aligned per scene (speed up slightly rather than overrun). Default no BGM (plain-style); if music is used, duck ~20dB under VO and no motion SFX.
6. Export H.264 High yuv420p + AAC-LC + faststart, under the chat attachment limit (aim <100MB; offer split parts). Fully decode-check, then review sample frames with `ai-run review -i` and revise 2-3 rounds before sending.
7. Reliability: max 2 heavy processes at once, run long jobs with nohup/setsid in resumable chunks, poll briefly.
8. Deliver the video, 2 sample frames, and an honest list of compromises. Never publish anywhere without the owner's review. In chat, return only paths plus a short summary.

## 风格
默认按 [plain-style](sand-workflow:plain-style)（少修饰、可核实）；与 visual-quality-pipeline 一起用。交付附 motion_ledger。
