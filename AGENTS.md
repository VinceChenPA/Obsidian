# AGENTS.md — Obsidian Vault

个人 Obsidian 知识库，Markdown 笔记，中文为主。本文件供 AI 代理读取，所有说明以此为准。

## 同步规则（重要）

- **修改前先 `git pull --ff-only`**，修改完成后 `git push`
- 本库 wikilink 使用**带路径形式**（如 `[[05.Health/中医/风寒|风寒]]`），**移动/重命名文件必须同步更新所有引用链接**（用 `git mv` 保留历史）
- 提交信息：英文小写短句（如 "update health moc"）
- 不确定笔记归属时先读 `Home.md` 与对应 `MOC.md`

## 目录结构

- `Home.md` — 全库导航入口（根级）
- `00.DailyLogs/` — 日期命名的日志（`YYYY-MM-DD.md`）
- `01.ReadingLogs/` — 读书/文章笔记，含 `MOC.md`
- `02.Domain/` — 领域知识（金融 BASEL 3 等），含 `MOC.md`
- `03.Engineering/` — 工程技术（Python/SQL/dbt/k8s/GCP/Linux/AI），子目录含 `MOC.md`
- `04.Career/` — 职业发展，含 `MOC.md`
- `05.Health/` — 健康医学（`心脏瓣膜病/` 与 `中医/` 两个子目录），含 `MOC.md`
- `06.Personal/` — 个人项目与思考，含 `MOC.md`
- `99.Attachments/` — 附件（图片等）
- `Templates/` — 笔记模板
- `Omnivore/` — Omnivore 导入、待消化的文章

每个主分类内都有 `MOC.md`（内容地图），先读对应 MOC 再操作。

## 约定

- **语言**：笔记以简体中文为主，技术名词可保留英文
- **新建笔记**：`# 标题` 与文件名一致；主动链接相关已有笔记；健康类一律放 `05.Health/`
- **frontmatter**：统一使用 `tags` / `created` / `updated` / `status` / `source` 键，tags 填真实值不要留空
- 不确定放哪个目录时询问用户，不要自作主张

## 环境

- OS: Linux（非 Windows，无 `.cmd` 工具）
- Remote: `git@github.com:VinceChenPA/Obsidian.git`（SSH）
- 无构建/测试/lint 命令
