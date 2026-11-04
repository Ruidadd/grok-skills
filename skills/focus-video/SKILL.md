---
name: focus-video
description: 做解说/科普/Shorts 等视频时的总入口：确认页→plan→tokens→build 实现→抽帧→跨模型 review→≥2 轮修订后交付。
---
# 视频工作流（薄入口）
详细规范见 `explainer-video-production` 与 `visual-quality-pipeline`，本文只定路由与门禁。

## 触发
新视频、改版成片、动画/Remotion、口播视频。静态信息图不走这里（走 focus-code → `code`）。

## 目录
`video/<topic>-YYYY-MM-DD/{brief,plan,tokens.json,script,src,out,review,sources.txt}`

## 步骤与 ai-run type
| 步骤 | type | 后端 |
|---|---|---|
| 确认页 / 分镜 | `plan` | Claude |
| 口播稿 / 字幕 | `write` / `short` | Claude |
| 实现（Remotion 等） | `build` | Claude |
| 审帧 | `review` | Claude 建 → Codex 审 |
| 素材调研 | `research` / `youtube` | Gemini（按需） |

## 门禁
1. 新片必须先过确认页，用户确认后再开工。
2. 数字/图表只由代码绘制；先 `tokens.json`。
3. 渲染后抽帧，跨模型 review，≥2 轮可见修订（≤3 为目标）。
4. 交付：成片 ≥1080p、2 张样帧、ffprobe 信息、妥协清单；对话只回路径+摘要。

## 熔断
同任务 ai-run 第一次 exit≠0 立刻停并汇报，不换 type 重试。单 bot 单任务执行，总入口 bot 验收。

## 风格
默认 profile=`plain`。交付前加载并执行 [plain-style](sand-workflow:plain-style) 验收，附 `motion_ledger` 与素材来源表。不通过不交。
