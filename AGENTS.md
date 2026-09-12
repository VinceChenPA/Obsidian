# AGENTS.md — Obsidian Vault

个人 Obsidian 知识库，Markdown 笔记，中文为主。本文件供 AI 代理读取，所有说明以此为准。

## 同步规则（重要）

- **修改前先 `git pull --ff-only`**，修改完成后 `git push`
- 本库 wikilink 使用**带路径形式**（如 `[[05.Health/中医/风寒|风寒]]`），**移动/重命名文件必须同步更新所有引用链接**（用 `git mv` 保留历史）
- 提交信息：英文小写短句（如 "update health moc"）
- 不确定笔记归属时先读 `Home.md` 与对应 `MOC.md`

## 目录结构

- `Home.md` — 全库导航入口（根级）
- `log.md` — 追加式操作日志（根级，格式：`## [YYYY-MM-DD] <操作> | <说明>`）
- `00.DailyLogs/` — 日期命名的日志（`YYYY-MM-DD.md`）
- `01.ReadingLogs/` — 读书/文章笔记，含 `MOC.md`
- `02.Domain/` — 领域知识（金融 BASEL 3 等），含 `MOC.md`
- `03.Engineering/` — 工程技术（Python/SQL/dbt/k8s/GCP/Linux/AI），子目录含 `MOC.md`
- `04.Career/` — 职业发展，含 `MOC.md`
- `05.Health/` — 健康医学（`心脏瓣膜病/` 与 `中医/` 两个子目录），含 `MOC.md`
- `06.Personal/` — 个人项目与思考，含 `MOC.md`
- `99.Attachments/` — 附件（图片等）
- `Templates/` — 笔记模板

> 外部导入/速记内容消化后统一转为对应主题笔记（读入类放 `01.ReadingLogs/`），不留原始导入目录。

每个主分类内都有 `MOC.md`（内容地图），先读对应 MOC 再操作。

## 约定

- **语言**：笔记以简体中文为主，技术名词可保留英文
- **新建笔记**：`# 标题` 与文件名一致；主动链接相关已有笔记；健康类一律放 `05.Health/`
- **frontmatter**：统一使用 `tags` / `created` / `updated` / `status` / `source` 五键；tags 至少含一个 `type/*` 类型标签
- **tag 体系**：`type/note`（普通笔记）、`type/quicknote`（快速捕获，`status: unread` 待消化，统一暂存 `03.Engineering/Common_Area/quicknote.md`）、`type/log`（日期日志）；其余为主题词
- **tag 命名规范**：只允许字母（含中文）/ 数字 / `_` / `-` / `/`，**禁止空格**；多词标签用连字符连接（如 `DeepSeek-Harness`、`Matt-Pocock`）——含空格的标签 Obsidian 会报"不被允许的标签名"且不登记（2026-09-06 全库审计修复）
- **日志**：`00.DailyLogs/` 只放日期文件（`YYYY-MM-DD.md`），非日期内容请归入主题目录
- 不确定放哪个目录时询问用户，不要自作主张

## AI 工作流

### 触发短语

| 用户说 | 执行 |
|---|---|
| "消化 XXX"、"XXX 整理入库"、"存进知识库" | Ingest 完整流程 |
| "清空 quicknote"、"清空收件箱" | 批量 Ingest 收件箱 unread 条目 |
| "检查/验证知识库" | Lint |
| 纯提问（如"讲了什么"） | 只回答不落库；有归档价值时询问（Query） |

- 入库请求同时给素材 + 意图最明确；"只问问、不用存"则明确不落库
- 意图含糊时（如只贴内容无指令），先问一句是否入库

### Ingest（消化新内容）

来源：`03.Engineering/Common_Area/quicknote.md` 及外部导入内容。消化步骤：

1. 读取来源，与用户确认要点和归属
2. 在对应目录写主题笔记（读入类放 `01.ReadingLogs/`）
3. 更新对应 `MOC.md`，为新增笔记补一行摘要 + `[[wikilink]]`
4. 更新受影响的其他页面（交叉引用、相关笔记链接）
5. 从 `quicknote.md` 移除已消化条目
6. 在根级 `log.md` 追加条目：`## [YYYY-MM-DD] ingest | <标题>`

一次 ingest 可触及多个页面，属正常。

### Query（问答归档）

对知识库提问得到的分析、对比、新连接，若值得保留，应归档为对应目录的新笔记或在相关笔记/MOC 中补充，不让可复用结论只留在会话记录中；归档后同步更新交叉引用，并在 `log.md` 追加 `## [YYYY-MM-DD] query | <主题>`。

### Lint（健康检查）

定期或用户要求时执行（可用 `vault-lint` skill）：检查页面间矛盾、过时内容、孤儿页、被提及但无独立页的概念、缺失交叉引用与死链、MOC 遗漏、frontmatter/tag 合规。低风险项直接修复，内容判断项与用户确认，完成后在 `log.md` 追加 `## [YYYY-MM-DD] lint | <摘要>`。

## 环境

- OS: Linux（非 Windows，无 `.cmd` 工具）
- Remote: `git@github.com:VinceChenPA/Obsidian.git`（SSH）
- 无构建/测试/lint 命令
