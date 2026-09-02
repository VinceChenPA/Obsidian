# AI 工作流编排（grill-with-docs × OpenSpec × Matt 票流）

> 主题：把 Matt Pocock 的 grill-with-docs / to-spec / to-tickets / implement / code-review 技能与 OpenSpec 规范驱动开发串联成一条完整交付管道。
> 一句话：**grill 负责想、OpenSpec 负责存、to-tickets 负责拆、implement 负责做、code-review 负责查**——按顺序串联，谁强谁上，内容不重复。

## 为什么能配合

Matt 流（to-spec → to-tickets → implement → code-review）与 OpenSpec 默认流（propose → tasks → apply → archive）高度同构，但各有所长；grill-with-docs 是两套都缺的**前置决策锻造器**。

## 阶段分工

| 阶段 | 主力工具 | 理由 |
|---|---|---|
| 1. 决策锻造 | grill-with-docs | 唯一需要"人拍板"的地方；产出 CONTEXT.md + ADR |
| 2. 规范起草 | to-spec | 产出含 User Stories / seams / Testing Decisions，正是 OpenSpec proposal 的短板 |
| 3. 规范落盘 | OpenSpec change | 转录 to-spec 草案为 proposal/delta spec；Requirement/Scenario 结构可验收 |
| 4. 任务拆分 | to-tickets | tracer-bullet 垂直切片 + blocker 图 + 单上下文窗口尺寸，强于 OpenSpec 扁平 tasks.md |
| 5. 执行 | implement | tdd / typecheck / 全量测试 → code-review 的既定收尾协议 |
| 6. 审查 | code-review | Spec 轴直接读 OpenSpec 增量 spec，与 Standards 轴并行双轴审查 |
| 7. 归档 | openspec archive | delta 合并进 specs/，changes/ 移入 archive/ |

## 编排规则（避免双写）

- **规范只落一处盘**：to-spec 产物是"起草输出"，转录进 `openspec/changes/<name>/` 后即 OpenSpec 为准；tracker issue 只放链接不放内容。
- to-spec 说 "Do NOT interview"、grill-with-docs 说 "interview relentlessly"——正好前后衔接：盘问只发生在 grill 阶段，to-spec 合成时信息已齐。
- OpenSpec 自动生成的 `tasks.md` 可忽略，以 to-tickets 的票为准；implement 逐票跑。
- 中途发现新决策 → 轻量补盘问，追加 ADR / 更新 CONTEXT.md，再 `/opsx:sync` 对齐。

## 命令顺序

```text
# 终端（一次性）
npm install -g @fission-ai/openspec@latest && cd your-project && openspec init
# tracker 按需：/setup-matt-pocock-skills（选本地文件模式可零依赖）

# 对话内
① grill-with-docs：想法是「<一句话>」     # 逐问回答 → CONTEXT.md + docs/adr/
② to-spec（不盘问，只综合当前对话+代码库） # 产出 spec 草案
③ AI：转录为 openspec change，术语对齐 CONTEXT.md、design.md 引用 ADR
④ to-tickets：增量 spec 拆垂直切片，确认粒度后发布
   （本地 .scratch/ 或 GitHub）
⑤ implement：逐 frontier 票执行（tdd → typecheck → 全量测试）
⑥ code-review：diff 固定点 → Standards + Spec 双轴
   （Spec 轴显式传 openspec/changes/<name>/specs/ 路径）

# 终端（收尾）
openspec validate <change> && openspec archive
```

## 注意事项（坑）

1. **code-review 的 Spec 轴按序找规范**：commit issue 引用 → 用户路径 → docs/specs/.scratch。OpenSpec 下通常找不到 issue 引用，需**显式传 `openspec/changes/<name>/specs/` 路径**。
2. **to-spec 需已配置 tracker**（缺 `docs/agents/issue-tracker.md` 时先跑 setup）；单人本地项目用"本地文件"模式 `.scratch/` 最省事，GitHub issues 留给真人协作场景。

## 分层记忆模型

| 文档 | 回答的问题 | 角色 |
|---|---|---|
| SOP（本类文档） | 怎么做（流程顺序） | 流程规范 |
| AGENTS.md | 听谁的（行为边界） | 行为规范 |
| CONTEXT.md | 叫什么（术语共识） | 语言规范 |
| ADR（docs/adr/） | 为什么（架构取舍） | 决策记录 |
| OpenSpec specs/ | 系统行为是什么（delta 演进） | 活文档 |

## SOP 模板（可直接落地为 command / skill）

```markdown
# SDD 全流程 SOP（v1）
> 适用范围：新功能/变更从想法到归档的完整交付。

## 前置条件
- 项目已 openspec init（有 openspec/specs/）
- tracker 已配置（本地 .scratch/ 或 GitHub），否则先跑 setup-matt-pocock-skills

## 流程
### Phase 1 · 决策锻造（唯一盘问点）
输入：一句话想法
动作：grill-with-docs 逐问作答直到共识
产出：CONTEXT.md（术语）+ docs/adr/NNNN-*.md（决策）
退出条件：用户确认"已达成共享理解"

### Phase 2 · 规范起草与落盘
1. to-spec 综合草案（User Stories / seams / Testing Decisions）
2. 转录为 openspec/changes/<slug>/：proposal.md ← 草案主体（术语对齐 CONTEXT.md）；
   specs/*.md ← 增量 Requirement + Scenario（GIVEN-WHEN-THEN）；design.md 引用 ADR 编号
3. tasks.md 删除或仅作速览（拆分权交给 to-tickets）

### Phase 3 · 拆票
to-tickets 基于增量 spec 拆分；验收：垂直切片、单票单上下文窗口、blocker 图经用户确认

### Phase 4 · 逐票实施
implement 沿 frontier 逐票（tdd → typecheck → 单测）；每票完成即勾选

### Phase 5 · 双轴审查
code-review：固定点 = 首票起点 commit；Spec 轴源 = openspec/changes/<slug>/specs/
失败回 Phase 4，直到双轴通过

### Phase 6 · 收尾归档
openspec validate <slug> && openspec archive

## 常见错误
- to-spec 前跳过 grill → AI 把未确认假设写进规范
- 内容双写（proposal 与 issue 各一份）→ 只 OpenSpec 落盘
- code-review 忘记传 spec 路径 → Spec 轴找不到规范
- 中途新决策不回 Phase 1 → 轻量补盘问记 ADR
```

## 参考

- 本地 Matt Pocock skills 仓库：`~/oc_ws/github_repos/skills`（skills/engineering/{to-spec,to-tickets,implement,code-review,grill-with-docs}）
- 技能安装位置：`~/.config/opencode/skills/`（grill-with-docs 委托 grilling + domain-modeling）
- [[03.Engineering/Common_Area/matt-pocock-skills|Matt Pocock Skills]]
- [[03.Engineering/Common_Area/AI/Spec-Driven Development|Spec-Driven Development (SDD)]]
