# Home · 知识库导航

个人知识库。内容按主题分布在 6 个主分类中，各分类内以 `MOC.md`（内容地图）作为索引，本页是唯一入口。

## 主分类

### 工程技术（最活跃）
- [[03.Engineering/MOC|工程技术 · 内容地图]] — Python / SQL / dbt / k8s / GCP / Linux / AI 辅助开发
  - [[03.Engineering/Python/MOC|Python]]
  - [[03.Engineering/SQL/MOC|SQL]]

### 阅读与输入
- [[01.ReadingLogs/MOC|阅读笔记 · 内容地图]] — 读书笔记与文章摘要
- 每日日志（00.DailyLogs/）— 按日期命名（`YYYY-MM-DD.md`）

### 领域与职业
- [[02.Domain/MOC|领域知识 · 内容地图]] — 金融（BASEL 3）
- [[04.Career/MOC|职业发展 · 内容地图]] — 面试 / 领导力 / 工作感悟
- [[05.Health/MOC|健康 · 内容地图]] — 心脏瓣膜病 / 中医
- [[06.Personal/MOC|个人 · 内容地图]] — 量化 / 亲子 / 行程 / 职业思考
- `06.Personal/行程归档/` — 过期行程版本存档

## 原始来源与附件
- `98.Raw/` — 原始来源层：文章 / 报告 / 转录等**不可变原文**（LLM 只读，不改不删、不补 frontmatter）
- `99.Attachments/` — 附件（图片等）统一存放

## 工具与模板
- `quicknote.md` — 全库收件箱（快速捕获，待消化；耐久原文放 `98.Raw/`，临时速记放这里）
- [[Templates/generic_template|通用模板]] / [[Templates/QuickNote_Template|QuickNote 模板]]

## 使用指引（AI 代理必读）
- 操作前先 `git pull --ff-only`，完成后 `git push`
- 先读对应 `MOC.md`，再定位/创建笔记
- 本库 wikilink 使用带路径形式（`[[分类目录/文件名|显示名]]`），移动文件必须同步更新引用
- 新笔记命名：描述性标题即可（中文/英文均可），避免空格与特殊字符
- 原始来源先落 `98.Raw/`（保留原文），再在 `00.`–`06.` 写主题笔记并在 `source` 登记来源路径
- 健康类笔记统一放 `05.Health/`（中医在 `05.Health/中医/`）
