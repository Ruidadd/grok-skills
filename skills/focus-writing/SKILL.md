---
name: focus-writing
description: 长文、简报、公众号、副业文案的写作入口：brief→plan→write→跨模型审稿→落盘，强制 --out，正文不贴进对话。
---
# 写作工作流（薄入口）

## 触发与分流
- 公众号文章 → 加载 `aws-wechat-article-main` 链（选题/写稿/审稿/排版/发布）。
- 情报简报、长文、产品/副业文案 → 本流程 + 视需要 `copywriting` / `content-strategy`。

## 目录
`writing/<channel>-<topic>-YYYY-MM-DD/{brief,outline,draft.md,review.md,final.md}`

## 步骤与 ai-run type
| 步骤 | type | 后端 |
|---|---|---|
| 大纲/策略 | `plan` | Claude |
| 正文 | `write` | Claude |
| 标题/钩子/摘要 | `short` | Claude |
| 审稿（结构/事实/敏感） | `review` | Codex（跨模型） |
| 批量清洗/翻译 | `fast` | agy |
| 深度事实 | `research` | Gemini |

## 门禁
1. brief 写明受众、结论、长度、渠道后再动笔。
2. 超 800 字必须 `--out` 落盘；对话只回路径+3 行摘要。
3. final.md 末尾附「结论句 + 数据来源」，可直接作为视频/信息图 brief 复用。

## 熔断
第一次 exit≠0 立刻停并汇报，不换 type 重试。

## 风格
交付前加载并执行 [plain-style](sand-workflow:plain-style) 验收（写作清单 + 黑名单）。不通过不交。
