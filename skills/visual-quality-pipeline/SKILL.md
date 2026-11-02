---
name: visual-quality-pipeline
description: >-
  Use whenever making a chart, infographic, data graphic, or explainer/animated
  video with Claude Code or Codex, so the output is planned, code-rendered,
  reviewed from real frames, and revised before delivery.
---
# Visual quality pipeline

One-shot generation is banned. Every chart or video goes plan -> build -> render -> look -> revise.

## Hard rules
- Numbers and text are never drawn by an image-generation model. Charts come from code (ECharts, Vega-Lite or Plotly) with one theme. Image models only for backgrounds or illustrations with no text.
- Every visual reads colors, font sizes, spacing and fonts from one `tokens.json`. Chinese text uses Noto Sans CJK SC (installed). Never hardcode one-off colors or sizes.
- Output at least 1920x1080 (or 1080x1920 vertical), 30 or 60 fps, CRF 18 or lower. Charts exported as SVG or PNG at 2x.
- Videos are real animation (Remotion preferred): items build in, numbers count up, transitions between shots. A still image with a slow zoom is not acceptable.
- Never deliver something you have not looked at as rendered frames.

## Effort levels
| Stage | Command |
|---|---|
| Storyboard / design plan (once) | `ai-run plan --bot <名字> --out plan.md < brief.md` (claude --effort max) |
| Writing chart or video code | `ai-run code --bot <名字> < task.md` (codex, reasoning effort high) |
| Reviewing rendered frames | `ai-run review --bot <名字> -i sheet.png < checklist.md` (codex high); builder was Codex → add `--reviewer claude` |
| Short captions / copy | `ai-run short --bot <名字> < copy.md` (claude --effort medium) |
Pass long prompts via a file (`< prompt.md`), never an empty stdin.

## Steps
1. **Plan.** At max effort, write a storyboard: per shot, the one message, the visual, the data source, duration. Attach 1-2 reference style images if available. Verify every number against its source.
2. **Design tokens.** Create or reuse `tokens.json` (palette, type scale, margins, font paths). Reuse the previous project's file if it worked.
3. **Build.** Codex writes chart code / Remotion components that read the tokens. Reuse saved components before writing new ones.
4. **Render keyframes.** Charts: render the PNG. Video: render, then pull 3 frames per shot into one contact sheet (`ffmpeg -i out.mp4 -vf "fps=1/2,scale=640:-1,tile=4x3" sheet.png`).
5. **Review with a different model** than the builder (Claude reviews Codex's work or the reverse), scoring the checklist below; output a numbered list of concrete fixes.
6. **Revise** 2-3 rounds until every checklist item passes. Look at the new frames yourself each round.
7. **Final pass.** Check audio-caption-picture sync, watch the full-length contact sheet, confirm resolution and file size with `ffprobe`. Then deliver.
8. **Save wins.** Store components and tokens that worked in a reusable template folder for next time.

## Review checklist
1. Every number matches its source.
2. No text overflows, overlaps or gets cut off.
3. Contrast is readable on phone size.
4. Elements align to a grid; consistent margins.
5. One message per shot or chart; nothing crowded.
6. Chinese punctuation and line breaks are correct.
7. Colors and fonts all come from the tokens.
8. Motion has purpose; no static slide plus zoom.
9. Resolution and frame rate meet the rules.
10. Looks like a professional explainer channel, not a PPT.
