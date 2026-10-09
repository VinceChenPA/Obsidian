# Log

追加式操作日志（ingest / query / lint）。格式：`## [YYYY-MM-DD] <操作> | <说明>`。查最近 5 条（按当前 shell 任选其一）：POSIX `grep "^## \[" log.md | tail -5`；PowerShell `Select-String -Path log.md -Pattern '^## \[' | Select-Object -Last 5`。

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
## [2026-09-25] setup | dsh web profile 接入知乎 MCP（dsh-mcp-client + insert patch 语法），启动 dsh web 验证连接成功
## [2026-09-25] ingest | OpenCode 深读《把编码 agent 当数据库来设计》（知乎）：事件溯源四层架构/压缩交接协议/Effect/对照 Claude Code，含本机实践对照表，更新 AI 两级 MOC
## [2026-09-25] lint | 将 V2 架构视角回填：《AI Agent 规则文件体系》补 SystemContext 运行时机制（类型化源/快照/纪元/对账/拒绝静默降级），《V1→V2 迁移记录》补「现象-原因」对照表
## [2026-09-25] lint | zhihu-mcp 解析器改进回归验证（搜索 1→6 条、author/votes 填充、去重生效）：笔记补改进记录，本地 commit 2e3e4eb
## [2026-10-09] lint | LLM Wiki 模式符合性核对（三层架构＋Ingest/Query/Lint＋索引/日志全部到位）：修 AGENTS.md 环境漂移（Linux 单一平台→Windows/Linux 双平台）、Lint 死引用（被引用的 vault-lint skill 实际不存在，改为 schema 自带三层清单）、补「矛盾保留（已被 XX 取代）」约定；GEMINI.md 收敛为 AGENTS.md 指针（原小写 kebab-case 命名约定作废）；log.md 头部补 PowerShell 查询写法
## [2026-10-09] lint | 规则层改为彻底跨平台（取代上一条的"双平台并列"做法）：AGENTS.md 环境章节去掉 Windows 绝对路径与主机相关措辞，要求一律用相对仓库根路径、不写死系统假设、命令给 POSIX 与 PowerShell 两种写法；log.md 与《LLM Wiki 中文实践指南》的 log 查询示例同步并列
## [2026-10-09] setup | 建立原始来源层 `98.Raw/`（补齐 LLM Wiki 三层架构中缺失的 raw sources 层）：新增目录 README（只读/不批量规范化/更正写 wiki 层）；AGENTS.md 补三层映射、原始来源不可变约定、Ingest 改为从 raw 读取且**原文保留**（取代"不留原始导入目录"）、Lint 规范层标注 raw 豁免；Home.md 加入口与使用指引；实践指南三层架构表与 Web Clipper 行同步。存量原文按决定不迁移（只对新增来源生效）
