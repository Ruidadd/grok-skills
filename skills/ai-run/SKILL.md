---
name: ai-run
description: >-
  Use whenever a bot hands writing, coding, planning or review work to Claude
  Code, Codex, or Antigravity (agy): call ai-run instead of claude/codex/agy
  directly.
---
# ai-run — 各取所长的模型路由

Never call `claude -p` / `codex exec` / `agy` directly. Use `ai-run <type> --bot <你的名字> [--out FILE] [-i img ...] [--effort low|medium|high|xhigh] < prompt.md`. Prompt always via stdin/file (empty → exit 65). Every call is logged to `/home/box/tools/ai-run/log.jsonl`.

**分工（2026-11-03）：** Claude 承担长规划、复杂程序、视频实现（用量可最大）；Codex 做中等代码与静态图表/信息图；Gemini 做 `fast` 与 youtube/research；DeepSeek（`dsh`）为可选省额度编程备胎，默认不经 ai-run。

| type | 用途 | 实际调用 |
|---|---|---|
| plan | 分镜、设计、架构、难推理、长规划 | `claude -p --effort xhigh`，15 分钟超时，超时或空输出自动降到 high 重跑 |
| write | 草稿、文章、简报、文案 | `claude -p --effort high` |
| short | 短文案、标题、字幕 | `claude -p --effort medium` |
| build | **复杂程序、视频实现（Remotion 等）** | `claude -p --effort high`（可用 `--effort medium\|high\|xhigh`），`--permission-mode bypassPermissions`，默认超时 30 分钟（`AI_RUN_BUILD_TIMEOUT`） |
| code | 中等脚本、数据、**静态图表/信息图代码** | `codex exec -s workspace-write -C $PWD -c model_reasoning_effort=high`（在当前目录可写） |
| review | 审文档或渲染帧（跨模型） | Codex high + `-i`；`--reviewer claude` → `claude -p --effort high` |
| fast | 翻译、格式整理、批量小任务、短文摘要 | `agy -p=... --model gemini-3.8-flash-{low\|medium\|high}`（默认 high；`--effort` 切换档位）。**不做**规划、写代码、审图 |
| youtube | YouTube 视频理解 | 不调 CLI，exit 2 → 用 box 浏览器 Gemini web (wuruidav2@gmail.com) |
| research | 深度调研/长文档 | exit 2 → Gemini web；`--cli` 时用 gemini CLI（免费 key） |

Notes:
- Subscription login for `agy` has Gemini 3.8 Flash (low/medium/high), Claude Opus/Sonnet 4.6, GPT-OSS 120B — **no Gemini 4**. Gemini 4 needs Google Cloud billing (not configured).
- Rule: review with the model that did NOT build it（`build`/Claude 写的 → 默认 Codex 审；`code`/Codex 建的 → `--reviewer claude`）。
- **不要**把静态图表流水线改成 `build`；**不要**把复杂应用/视频实现继续丢给 `code`。
- Circuit breaker unchanged: 同一任务第一次 `ai-run` exit≠0 立刻停，不换 type 连环重试。

Examples:
```
ai-run plan --bot 动画大师 --out storyboard.md < brief.md
ai-run build --bot 动画大师 --effort xhigh < remotion_impl.md
ai-run code --bot 动画大师 < chart_infographic.md
ai-run review --bot 动画大师 -i frames/sheet.png < checklist.md
ai-run review --bot 动画大师 --reviewer claude -i frames/sheet.png < checklist.md
ai-run write --bot 一线建设者情报 --out brief.md < prompt.md
echo "把 hello 翻成日语，只输出译文" | ai-run fast --bot 调研助手
echo "整理这段 bullet" | ai-run fast --bot PPT大师 --effort low --out out.txt
```
Exit codes: 0 ok, 2 = use Gemini web, 64 bad args, 65 empty prompt, other = tool error (stderr shown). For `code`, if Codex exits 0 but its output admits a write/read-only failure, ai-run treats it as exit 1.
