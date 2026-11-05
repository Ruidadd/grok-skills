---
name: transcript-healthcheck
description: >-
  Use when a scheduled healthcheck should scan the last day of bot conversations
  for recurring friction (errors, retries, user corrections, stuck tools) and
  propose fixes, staying silent when nothing is found.
---
# Transcript healthcheck

## Data source（权威）
`/home/box/sand-data/search-index.db`（SQLite）。**不要**用已不存在的 `ReadTranscript`，也**不要**依赖 `/home/box/agent-data/agent-transcripts`（常缺）。

### 窗口与过滤
- 时间窗：自上次成功巡检起，或未知时用最近 24h；用 `messages.timestamp_ms`（epoch ms）。时区报告用 Asia/Shanghai（用户）或例行里写明的时区，但查询用 ms 绝对值。
- 只读表：`messages`（`agent_id, entry_id, role, timestamp_ms, body, in_main_chat`）、必要时 `agents`。
- 过滤：`in_main_chat = 1` 优先；跳过纯 routine 心跳噪音（如 Auto-start Mihomo 静默成功）。可按 `agent_id` 聚合活跃 bot。
- 查询示例：
```sql
SELECT agent_id, role, timestamp_ms, substr(body,1,400)
FROM messages
WHERE timestamp_ms >= ? AND in_main_chat = 1
ORDER BY timestamp_ms DESC;
```
（`?` = 窗口起点 ms。用 Python `sqlite3`；机器上未必有 `sqlite3` CLI。）

## 步骤
1. **Cheap check：** 按窗口查 `messages` 行数与活跃 `agent_id`。若 0 条，静默结束。
2. 按 bot 抽样近期正文，记摩擦：tool/ai-run 失败与熔断、反复重试、用户纠正、bot 声称做不到、例行失败或空跑。
3. 按根因归并。忽略一次性噪音；只留出现 ≥2 次或有真实用户影响的问题。
4. 无合格项 → 静默结束。
5. 否则用中文发短报告：最多 3 条，每条含 bot、发生了什么（带时间）、可能原因、一条可执行修复。末尾用 widget 问修哪些。
6. **用户未点选前**，不改 bot、不改技能、不新建例行。
7. 省 Grok：不做深度重读/网页调研；长分析丢 `ai-run short`，对话只读摘要。
