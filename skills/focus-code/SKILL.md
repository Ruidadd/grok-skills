---
name: focus-code
description: 写程序/脚本/静态图表时的分流入口：用判定卡选 build（复杂，Claude）或 code（中等，Codex），带最小验收与熔断。
---
# 代码工作流（薄入口）

## 判定卡
| 满足任一 → `build`（Claude） | 否则 → `code`（Codex） |
|---|---|
| 多文件/多模块应用、新项目骨架 | 单脚本、单模块改动 |
| Remotion/视频实现、动效 | 静态信息图（PNG/SVG，配合 visual-quality-pipeline） |
| 需长超时、多包依赖、架构决策 | 数据清洗、小工具、补丁 |
架构/长规划先 `plan`；小整理/格式 `fast`（agy）。DeepSeek `dsh` 暂不进路由，仅手动。

## 目录
`code/<topic>-YYYY-MM-DD/{brief,plan?,src,tests?,out,notes.md}`；大仓用 git worktree。

## 验收
1. 有可运行命令且实际跑过。
2. 有测试则全绿；没有则写明手测了什么。
3. 实现完成后 `review` 跨模型：Claude 建→Codex 审；Codex 建→`--reviewer claude`。
4. notes.md 记录：如何跑、路径、已知限制。对话只回路径+摘要。

## 熔断
第一次 exit≠0 立刻停并汇报，不连环重试、不换 type 绕过。复杂实现交工程类 bot 主执行；其他 bot 只改本职小脚本。
