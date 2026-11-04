---
name: Codex 信息图
description: >-
  用 Codex 做静态信息图（PNG/SVG）；可选动效短视频的实现默认改走 Claude（ai-run build）。一律强制
  visual-quality-pipeline：数字只由代码绘制，经 plan→实现→render→跨模型审图→2–3 轮修订后再交付。
---
# Codex 信息图

何时用：用户要信息图、数据图解、科普长图；或同一课题还要可选动效短视频。静态图的代码侧用 Codex（`ai-run code`）；**视频实现默认 Claude（`ai-run build`）**。

**硬依赖：** 动手前先 `Skill` 加载 [visual-quality-pipeline](sand-workflow:visual-quality-pipeline)，并完整遵守其硬规则与审图清单。本 skill 只补充输入、目录、命令与交付约定；冲突时以 visual-quality-pipeline 为准。

## 不做
- 不用文生图模型画数字、坐标轴、标签或正文（仅可做无字插画/背景）。
- 不跳过 plan / 审图 / 修订直接交付。
- 不做 Muse；不把「静态图加缓慢缩放」当成动效视频。
- 不把复杂应用/视频实现继续丢给 `ai-run code`。

## 输入（开工前凑齐）
1. **结论句**：这张图要让人 3 秒内看懂的一句话。
2. **数据**：带单位的表或 JSON，并注明每个数字的来源；缺来源就先停，问清再画。
3. **规格**：横版 `1920x1080` 或竖版 `1080x1920`；静态只要 PNG/SVG，还是另要动效短视频。
4. **受众与语言**：默认简体中文；字体用已安装的 Noto Sans CJK SC。

## 目录（每个课题一套）
```text
infographic/<topic>-YYYY-MM-DD/
  brief.md          # 结论句、受众、规格、数据与来源
  plan.md           # ai-run plan 产出的分镜/版式
  tokens.json       # 唯一设计系统（色、字号、间距、字体路径）
  data.json         # 核实后的数据
  src/              # 图表或 Remotion 代码
  out/              # PNG/SVG/MP4
  review/           # 渲染帧拼图与审图意见
```
尽量复用上一次可用的 `tokens.json` 与组件，而不是每次从零造主题。

## 流程（一律经 ai-run，禁止直调 claude/codex）
把提示写入文件再喂给 stdin；`--bot` 填当前执行 bot 名。第一次失败（exit≠0）立刻停，不换模式连环重试。

1. **Plan**（Claude xhigh）
   ```bash
   ai-run plan --bot <bot> --out plan.md < brief.md
   ```
   产出：每屏一个信息点、版式、数据字段映射、时长（若有视频）。

2. **Tokens**  
   写或复用 `tokens.json`。后续代码只读 tokens，禁止散落硬编码色值/字号。

3. **Build**
   - **静态信息图**（Codex high）：
     ```bash
     ai-run code --bot <bot> < task.md
     ```
     ECharts / Vega-Lite / Plotly / SVG，导出 ≥2x 的 PNG，并保留可编辑源码。
   - **动效/视频实现**（Claude，默认 high；重活可用 `--effort xhigh`）：
     ```bash
     ai-run build --bot <bot> [--effort xhigh] < task.md
     ```
     Remotion 等真实入场/计数/转场；先加载相关 remotion-* skill。

4. **Render**  
   输出到 `out/`。视频再用 ffmpeg 抽关键帧拼成 `review/sheet.png`（见 visual-quality-pipeline）。

5. **Review**（跨模型）
   - Codex 建的静态图 → `--reviewer claude`
     ```bash
     ai-run review --bot <bot> --reviewer claude -i review/sheet.png < checklist.md
     ```
   - Claude `build` 的视频 → 默认 Codex 审（不要加 `--reviewer claude`）
     ```bash
     ai-run review --bot <bot> -i review/sheet.png < checklist.md
     ```
   按 visual-quality-pipeline 十条清单出「可执行修改列表」。

6. **Revise** 2–3 轮：每轮改代码 → 重渲染 → 再看帧。清单未过不交付。

7. **Deliver**  
   聊天里只给结论、路径、1–2 张关键图/封面；完整源码与 `out/` 留在课题目录。附 `ffprobe`/`file` 确认的分辨率与体积。

## task.md 最小模板
- 读 `plan.md`、`tokens.json`、`data.json`；数字必须与 `data.json` 一致。
- 实现 `src/`，写入 `out/`；禁止用文生图画字。
- 中文用 Noto Sans CJK SC；出血与边距遵守 tokens。
- 结束后列出生成文件路径与如何本地重跑渲染的一条命令。

## 验收才算完成
- [ ] 每个数字能对上 `data.json` 与来源说明
- [ ] 无叠字、裁切、低对比
- [ ] 颜色/字体全部来自 `tokens.json`
- [ ] 静态 ≥1080 边；视频 ≥1080p 且为真动画
- [ ] 跨模型审图 + 至少 2 轮可见修订记录在 `review/`
