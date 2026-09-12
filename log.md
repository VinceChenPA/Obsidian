# Log

追加式操作日志（ingest / query / lint）。格式：`## [YYYY-MM-DD] <操作> | <说明>`，可用 `grep "^## \[" log.md | tail -5` 查看最近 5 条。

## [2026-09-13] setup | 建立 AI 工作流：AGENTS.md 新增 Ingest/Query 章节，创建 log.md
## [2026-09-13] setup | 引入 vault-lint 健康检查工作流（skill + AGENTS.md）
## [2026-09-13] lint | 首轮全库检查：修 3 处 MOC 遗漏（llm-wiki/dbt 外部表/行程归档）、1 处简写 wikilink；frontmatter 历史欠账与 1 处死链待决
