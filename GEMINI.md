---
tags:
  - config/gemini
  - type/note
created: 2025-10-09
updated: 2026-10-09
status: done
source: AGENTS.md
---
# Gemini Configuration for My Obsidian Vault

> **本库唯一的规则源是 [`AGENTS.md`](AGENTS.md)。** 开始任何操作前先完整读取它，再按其中的目录结构、五键 frontmatter、tag 体系与 Ingest / Query / Lint 工作流执行。
>
> 本文件只是给 Gemini 的入口指针，不再重复规则。历史上这里写的"小写 kebab-case 文件名"等约定与 `AGENTS.md` 冲突，已于 2026-10-09 作废（同一约定只保留在 `AGENTS.md` 一处）。

## 入口文件

- `AGENTS.md` — 唯一规则源：同步规则、目录结构、frontmatter/tag 约定、三种工作流、触发短语
- `Home.md` — 全库导航入口
- 各主分类 `MOC.md` — 内容地图，先读 MOC 再定位或新建笔记
- `log.md` — 追加式操作日志（`## [YYYY-MM-DD] <操作> | <说明>`）
- `quicknote.md` — 全库收件箱（`type/quicknote`、`status: unread`，待消化）

## 仍需记住的最小事项

- 笔记以简体中文为主，技术名词可保留英文；新建笔记的 `# 标题` 与文件名一致
- wikilink 使用带路径形式（如 `[[05.Health/中医/风寒|风寒]]`），移动/重命名文件必须同步更新所有引用
- 修改前先 `git pull --ff-only`，完成后 `git push`；提交信息用英文小写短句
- 主动链接相关已有笔记；归属不确定时先读 `Home.md` 与对应 `MOC.md`，或直接问我
- 纯提问只回答不落库；有归档价值的结论先问我是否归档
