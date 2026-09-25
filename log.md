# Log

追加式操作日志（ingest / query / lint）。格式：`## [YYYY-MM-DD] <操作> | <说明>`，可用 `grep "^## \[" log.md | tail -5` 查看最近 5 条。

## [2026-09-13] setup | 建立 AI 工作流：AGENTS.md 新增 Ingest/Query 章节，创建 log.md
## [2026-09-13] setup | 引入 vault-lint 健康检查工作流（skill + AGENTS.md）
## [2026-09-13] lint | 首轮全库检查：修 3 处 MOC 遗漏（llm-wiki/dbt 外部表/行程归档）、1 处简写 wikilink；frontmatter 历史欠账与 1 处死链待决
## [2026-09-13] lint | 批量规范：78 篇补齐 frontmatter（五键/type 标签，created/updated 取自 git 历史）、41 篇补 H1、清除最后 1 处死链
## [2026-09-13] ingest | LLM Wiki 中文实践指南（原文总结＋本库实践与真实例子），更新 AI/MOC
## [2026-09-13] setup | AGENTS.md 增加工作流触发短语约定
## [2026-09-13] setup | 触发短语表补齐 Query 归档触发（三工作流全覆盖）
## [2026-09-13] ingest | AI 工作流触发指南（触发方式/例子/技巧），更新 AI/MOC
## [2026-09-13] setup | quicknote.md 移至库根作为全库收件箱（更新 AGENTS/Home 引用）
## [2026-09-13] ingest | 清空收件箱：2 条旧条目（PR precheckers、pandas→polars）经确认后丢弃
## [2026-09-13] query | LLM Wiki 前后取用知识区别（对比表+本库实例），更新 AI/MOC
## [2026-09-17] ingest | Frontier Engineering（Kiro 10 条原则全文 drilldown），更新 AI/MOC
## [2026-09-25] lint | opencode webfetch 两篇笔记标注 V2 迁移现状（插件路径、注册方式、配置字段 providers、历史内容保留）
## [2026-09-25] lint | opencode V2 迁移同步：更新 version-log/web-setup/nginx-reverse-proxy 三篇至 V2 现状，新增 V2 迁移总纲笔记（含清理清单），AI 目录两篇研究补 V2 版本提示
## [2026-09-25] query | opencode 服务器资源优化归档：MCP 启动精简（去 npm exec 包装层，省 ~280MB）+ 启用 2GB swap（含资源基线）
## [2026-09-25] setup | 配置知乎 MCP（Douyh123/zhihu-mcp）：无头扫码登录改造 + opencode remote 接入（oauth:false），首次抓取文章验证成功
## [2026-09-25] ingest | 知乎翻译版《Frontier Engineering 十大原则》（与既有笔记同源）：仅补充中文翻译来源链接，清理移动端速记 未命名.md
