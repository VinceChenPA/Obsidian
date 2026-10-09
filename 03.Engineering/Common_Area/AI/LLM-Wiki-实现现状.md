---
tags:
  - type/note
  - LLM-Wiki
  - knowledge-management
created: 2026-10-09
updated: 2026-10-09
status: done
source: 2026-10-09 逐组件核对 + vault-lint 首轮全库扫描（会话问答归档）
---
# LLM Wiki 实现现状（本库核对）

> 2026-10-09 对「LLM Wiki 模式在本库是否已落地」做了一次逐组件核对，并用新做的 `vault-lint` skill 跑了首轮全库 lint。**结论：模式已实现**；四个缺口里三个已闭合，剩一个需用户提供事实（行程矛盾，见文末）。

## 一、模式组件对照（含证据）

| 模式要求（见 [[03.Engineering/Common_Area/AI/llm-wiki|原文]]） | 本库实现 | 证据 |
|---|---|---|
| Schema 层：定义结构、约定与工作流 | ✅ | `AGENTS.md`：Ingest / Query / Lint 三章 + 触发短语表 + 五键 frontmatter / tag 规则 / 三层结构映射 |
| Wiki 层：LLM 生成与维护的 markdown | ✅ | `00.`–`06.` 六个主分类 + 9 个 `MOC.md`（含 `03.Engineering` 的 Python / SQL / AI 子 MOC） |
| index：每页链接 + 一行摘要的目录 | ✅ 改写为入口 + MOC | `Home.md` 为唯一入口，各分类 `MOC.md` 承载目录；抽查覆盖：AI 20/20、Career 7/7、Health 8/8、Personal 8/8 |
| log：可解析的追加式时间线 | ✅ | `log.md`，格式 `## [YYYY-MM-DD] <操作> \| <说明>`，POSIX / PowerShell 两种过滤写法已写明 |
| 原始来源层：不可变、LLM 只读 | ✅ 2026-10-09 补建 | `98.Raw/`（此前只有 `99.Attachments/` 的图片，消化后即删原件）；规则见 [[98.Raw/README\|原始来源层说明]] |
| Ingest / Query / Lint 三操作可执行 | ✅ | 三者均有真实执行记录（`log.md`）；Lint 已固化为全局 `vault-lint` skill，`AGENTS.md` 保留自带清单兜底 |
| 矛盾显式标记（「已被 XX 取代」） | ✅ 已启用 | [[03.Engineering/Common_Area/AI/LLM-Wiki-中文实践指南\|实践指南]] 的二次更正即首例；`AGENTS.md` 约定新增「矛盾保留」|
| 可选 CLI 检索（qmd 等） | ⛔ 未做 | 原文标注为可选；当前规模（90 篇）下 `Home → MOC → 笔记` 足够 |

## 二、首轮全库 lint 基线（2026-10-09）

规模：117 个文件（含附件）、90 篇笔记、9 个 MOC。

| 层 | 结果 |
|---|---|
| 规范层 | 五键 frontmatter 覆盖 100%；tag 字符集违规 0；新查日期格式，2 处中文长日期（`NumPy.md`、`IJP Interview.md`）已修 |
| 结构层 | 死链 0、MOC 遗漏 0、真孤儿 0（`00.DailyLogs/` 6 篇无入链属设计，已在 `AGENTS.md` 写明豁免） |
| H1 | 45 篇 H1 与文件名不一致 → 经决定**放宽规则**：H1 用描述性标题即可，不强制等于文件名（`AGENTS.md` 已改） |
| 内容层 | 子代理扫描出 18 项；事实类 9 项已修（`sed` 删空行命令、`grep` 名称与 `GREP_OPTIONS`、TAVR 50.7% 时间口径、V1/V2 混淆 3 处、DSH 对 skill 路径的错误表述、中文长日期），6 类按用户选择处理，1 类待事实 |

**方法论教训（已回写进 skill）**：扫描器出过两类整批误报——路径分隔符（`\` vs `/`）导致 183 个假死链，以及表格内的转义管道（反斜杠加竖线）被当成链接目标的一部分。`vault-lint` 因此新增「Scan mechanics」小节：先复核再结论，逐类抽样看原文。

## 三、待办

- **行程矛盾（需用户给事实）**：`06.Personal/贵州亲子自驾行程.md` 与 `06.Personal/云南亲子自驾行程.md` 都声称 2026.7.22–7.31、同一车同一行人、路线互斥；且贵州篇标题「第三版（兴义深度版）」的内容实际等于已归档的 `行程归档/贵州亲子自驾行程-v2.md`。需确认哪次真实成行，另一篇标「未采用」并归档。
- 内容层其余各项（去重、概念页、机制统一、中医两页重切、明文口令）见 `log.md` 2026-10-09 的后续条目。

## 相关

- [[03.Engineering/Common_Area/AI/llm-wiki|LLM Wiki —— 模式原文]]
- [[03.Engineering/Common_Area/AI/LLM-Wiki-中文实践指南|LLM Wiki 中文实践指南]]
- [[03.Engineering/Common_Area/AI/LLM-Wiki-取用知识前后对比|LLM Wiki 前后：取用知识的区别]]
- [[03.Engineering/Common_Area/AI/AI-工作流触发指南|AI 工作流触发指南]]
- [[98.Raw/README|原始来源层说明]]
