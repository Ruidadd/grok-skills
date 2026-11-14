---
name: ai-run
description: >-
  Use whenever a bot hands writing, coding, planning, review, research,
  video/engine builds, or fast Gemini tasks to Claude/Codex/Gemini via the
  shared ai-run CLI (never call claude/codex/agy directly). Build uses latest
  Fable; plan/write uses latest Opus.
---
# ai-run — 各取所长的模型路由

Never call `claude -p` / `codex exec` / `agy` directly. Use `ai-run <type> --bot <你的名字> [--out FILE] [-i img ...] [--effort low|medium|high|xhigh] [--model NAME] < prompt.md`. Prompt always via stdin/file (empty → exit 65). Every call is logged to `/home/box/tools/ai-run/log.jsonl`.

**分工（2026-11-13）：** Claude 承担长规划、复杂程序、视频/引擎实现；**plan/write/short/审查用最新 Opus（别名 `opus`）；build 用最新 Fable（别名 `fable`，当前家族为 Fable 5.1）**。可用 `--model` 或环境变量 `CLAUDE_MODEL` 覆盖。Codex 做中等代码与静态图表/信息图，并可用 `-i` 贴图；Gemini 做 `fast` 与 `research` CLI（失败自动退到 Claude write）；需要配乐时用 Gemini API 的 Lyria（默认 plain-style 不加 BGM）。`youtube` 仍只走 box 浏览器 Gemini web。DeepSeek（`dsh`）为可选省额度编程备胎，默认不经 ai-run。

| type | 用途 | 实际调用 |
|---|---|---|
| plan | 分镜、设计、架构、难推理、长规划 | `claude -p --model opus --effort xhigh`，15 分钟超时，超时或空输出自动降到 high 重跑 |
| write | 草稿、文章、简报、文案 | `claude -p --model opus --effort high` |
| short | 短文案、标题、字幕 | `claude -p --model opus --effort medium` |
| build | **复杂程序、视频/引擎实现（Remotion / Blender，及已安装的 Unity/Unreal）** | `claude -p --model fable --effort high`（可用 `--effort medium\|high\|xhigh` 或 `--model`），`--permission-mode bypassPermissions`，默认超时 **90 分钟**（`AI_RUN_BUILD_TIMEOUT`，秒）。超时且 stdout≥200 字节时 **soft-ok（exit 0）**，提示从 CWD 产物续跑/审片，避免熔断整单 |
| code | 中等脚本、数据、**静态图表/信息图代码**；可 `-i` 贴图 | `codex exec -s workspace-write -C $PWD -c model_reasoning_effort=high` |
| review | 审文档或渲染帧（跨模型） | Codex high + `-i`；`--reviewer claude` → `claude -p --model opus --effort high` |
| fast | 翻译、格式整理、批量小任务、短文摘要 | `agy -p=... --model gemini-3.8-flash-{low\|medium\|high}`（默认 high）。**不做**规划、写代码、审图 |
| youtube | YouTube 视频理解 | 不调 CLI，exit 2 → 用 box 浏览器 Gemini web (wuruidav2@gmail.com) |
| research | 深度调研/长文档 | **默认 Gemini CLI**；失败或空输出自动 fallback 到 Claude write（`--model opus`）。`--cli` 仍可用（同路径）。不再对「未加 --cli」瞬间 exit 2 |

Notes:
- Rule: review with the model that did NOT build it（`build`/Fable 写的 → 默认 Codex 审；`code`/Codex 建的 → `--reviewer claude` / Opus）。
- **不要**把静态图表流水线改成 `build`；**不要**把复杂应用/视频/引擎实现继续丢给 `code`。
- 这台制作机没有独显，不要在上面安装 Unity / Unreal 做硬渲。二维用 Remotion，三维用已安装的 Blender；Unity/Unreal 只在已有 GPU 的机器上、并且已安装且有许可证时由 Claude Code（Fable）调用。
- Circuit breaker unchanged: 同一任务第一次 `ai-run` exit≠0 立刻停，不换 type 连环重试。`research` 的 CLI→write 与 `build` 的 timeout soft-ok 是 **同 type 内** 恢复，不算连环重试。

Examples:
```
ai-run plan --bot 动画大师 --out storyboard.md < brief.md
ai-run build --bot 动画大师 --effort xhigh < remotion_impl.md
ai-run research --bot 调研助手 --out report.md < brief.md
ai-run code --bot 动画大师 -i ref.png < chart_infographic.md
ai-run review --bot 动画大师 -i frames/sheet.png < checklist.md
echo "把 hello 翻成日语，只输出译文" | ai-run fast --bot 调研助手
```
Exit codes: 0 ok（含 build 超时但有可用部分输出的 soft-ok）, 2 = use Gemini web（仅 youtube）, 64 bad args, 65 empty prompt, other = tool error (stderr shown). For `code`, if Codex exits 0 but its output admits a write/read-only failure, ai-run treats it as exit 1.
