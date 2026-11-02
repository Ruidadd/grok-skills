---
name: ai-run
description: 'Use whenever a bot hands writing, coding, planning or review work to Claude Code, Codex, or Antigravity (agy): call ai-run instead of claude/codex/agy directly.'
---
# ai-run — 各取所长的模型路由

Never call `claude -p` / `codex exec` / `agy` directly. Use `ai-run <type> --bot <你的名字> [--out FILE] [-i img ...] [--effort low|medium|high] < prompt.md`. Prompt always via stdin/file (empty → exit 65). Every call is logged to `/home/box/tools/ai-run/log.jsonl`.

| type | 用途 | 实际调用 |
|---|---|---|
| plan | 分镜、设计、架构、难推理 | `claude -p --effort xhigh`，15 分钟超时，超时或空输出自动降到 high 重跑 |
| write | 草稿、文章、简报、文案 | `claude -p --effort high` |
| short | 短文案、标题、字幕 | `claude -p --effort medium` |
| code | 脚本、应用、数据、图表/Remotion 代码、信息图代码 | `codex exec -s workspace-write -C $PWD -c model_reasoning_effort=high`（在当前目录可写） |
| review | 审文档或渲染帧（跨模型） | Codex high + `-i`；`--reviewer claude` → `claude -p --effort high` |
| fast | 翻译、格式整理、批量小任务、短文摘要 | `agy -p=... --model gemini-3.8-flash-{low\|medium\|high}`（默认 high；`--effort` 切换档位）。**不做**规划、写代码、审图 |
| youtube | YouTube 视频 | 不调 CLI，exit 2 → 用 box 浏览器 Gemini web (wuruidav2@gmail.com) |
| research | 深度调研/长文档 | exit 2 → Gemini web；`--cli` 时用 gemini CLI（免费 key） |

Notes:
- Subscription login for `agy` has Gemini 3.8 Flash (low/medium/high), Claude Opus/Sonnet 4.6, GPT-OSS 120B — **no Gemini 4**. Gemini 4 needs Google Cloud billing (not configured).
- Rule: review with the model that did NOT build it (Claude wrote → Codex reviews; Codex built → `--reviewer claude`).

Examples:
```
ai-run plan --bot 动画大师 --out storyboard.md < brief.md
ai-run code --bot 动画大师 < build_remotion.md
ai-run review --bot 动画大师 --reviewer claude -i frames/sheet.png < checklist.md
ai-run write --bot 一线建设者情报 --out brief.md < prompt.md
echo "把 hello 翻成日语，只输出译文" | ai-run fast --bot 调研助手
echo "整理这段 bullet" | ai-run fast --bot PPT大师 --effort low --out out.txt
```
Exit codes: 0 ok, 2 = use Gemini web, 64 bad args, 65 empty prompt, other = tool error (stderr shown). For `code`, if Codex exits 0 but its output admits a write/read-only failure, ai-run treats it as exit 1.
