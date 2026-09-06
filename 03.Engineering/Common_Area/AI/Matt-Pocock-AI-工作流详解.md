---
tags:
  - type/note
  - AI编码
  - Matt-Pocock
created: 2026-09-03
updated: 2026-09-03
status: done
source: https://www.aihero.dev/posts
---
# Matt Pocock AI 工作流详解

> 7 阶段宏观框架 + 核心 pipeline 命令顺序(v1.1)。记录于 2026-09-03,综合 Matt Pocock 2026-04 "AI Coding for Real Engineers" 研讨会、aihero.dev 官方文档、skills 仓库 v1.1 release notes 整理。
> 相关笔记: [[03.Engineering/Common_Area/matt-pocock-skills|Matt Pocock Skills — AI 编码方法论]]、[[03.Engineering/Common_Area/AI/AI Workflow 编排（grill-with-docs × OpenSpec × Matt ticket flow）|AI 工作流编排]]、[[03.Engineering/Common_Area/AI/Spec-Driven Development|Spec-Driven Development]]

## 核心论点

**软件工程基本功并未过时,它们反而是让 AI agent 发挥价值的"杠杆层"**——坏代码库只会产生坏 agent。所有技巧都源自经典理论:Brooks《设计原本》(共享设计概念)、Ousterhout《软件设计哲学》(深模块)、Pragmatic Programmer(反馈速度 = 开发速度上限)。

## 一、底层原理(为什么流程要这样设计)

1. **"聪明区 vs 愚蠢区"(Smart zone / Dumb zone)** — LLM 注意力关系随 token 平方级增长,质量约在 **~10 万 token** 开始明显退化(与宣称的 100 万 context 无关,那只是"更大片的愚蠢区")。任务必须**切片到能装进一次干净会话**。
2. **LLM 像《记忆碎片》主角** — 每次新会话都重置回系统提示词。所以**不要无限压缩(compact)长会话,而是频繁干净重置**;系统提示词要尽量小(见过有人塞 250K token,直接进愚蠢区)。
3. **会话间只能靠文件传递** — 聊天记录不是接口,**每步的产物文件是下一步的输入**(decisions → spec → tickets → 实现 → review)。
4. **深模块(Deep modules)** — 小接口 + 大实现,agent 易导航、测试边界清晰;浅模块(大量小文件互相耦合)是 agent 质量的天花板。改进方向用 improve-codebase-architecture 技能。
5. 典型会话形态:系统提示(小)→ 探索 → 实现 → 测试反馈循环。

## 二、宏观 7 阶段框架(适用于任何 AI 编码方法:Ralph loop / GSD / Spec Kit 皆同构)

| 阶段 | 目的 | 关键产物 |
|---|---|---|
| 1. Idea | 明确要做什么 | 问题陈述 |
| 2. Research(可选) | 探索外部依赖(API 集成等) | `research.md` 缓存 — 只在本次迭代存活,防腐烂误导 agent |
| 3. Prototype(可选) | 用一次性原型定 UI/体验/架构口味 | 可提交的 prototype 代码(可并入正式实现) |
| 4. Spec(PRD) | 文档化终点状态 | 产品需求文档(问题/方案/用户故事/实现决策/测试决策/out-of-scope) |
| 5. Tickets(Kanban) | 拆成带阻塞关系的 ticket | 垂直切片任务列表(GitHub Issues / Linear / 本地 md) |
| 6. Execution | 真正构建 | 工作代码(逐 ticket 实现,常 AFK 跑) |
| 7. QA | 人验证 | QA 计划 + 反馈 → 回 Kanban 循环(6-7 反复迭代) |

注:AI 倾向按水平层实现(先 DB → 再 API → 再前端),集成问题到最后才暴露;应拆成**垂直切片/曳光弹**,每个 ticket 跨全部层,立即得到完整反馈闭环,还可并行。

## 三、命令顺序(核心 pipeline)

### 1. 安装

```bash
npx skills@latest add mattpocock/skills            # 一键全量
npx skills add mattpocock/skills --skill=grill-me -y -g   # 单个
# 仓库级工程流配置(选本地文件模式可零依赖)
/setup-matt-pocock-skills
```

### 2. 标准流程(一个会话能装下的小功能,v1.1 现名)

```
① /grill-with-docs   有代码库:访谈 + 边问边写 CONTEXT.md + ADR(推荐默认)
   或 /grill-me       无代码库 / 非编码事项:纯访谈,零文件,stateless
② /to-spec           把访谈会话直接合成规范(原 to-prd,改名统一叫 spec)
③ /to-tickets        把 spec 切成曳光弹垂直切片 ticket,带阻塞边(原 to-issues)
④ /implement         逐 frontier ticket 实现(TDD:red → green → refactor)
⑤ /code-review       全新会话 + 更强模型审查 diff
```

### 3. 超大任务(装不进一个 agent session,路线不明)

```
/wayfinder → /to-spec → /to-tickets → /implement
```

- /wayfinder 在 issue tracker 上画 `wayfinder:map` 决策图:每个 ticket 是"一个问题"(decision ticket),按 HITL(grilling/prototype)或 AFK(research)分型,逐张解析直到路径清晰;雾区(fog of war)随解析逐步明朗。
- 地图清空即交接(mattpocock 2026-08 发推确认上述顺序;若发现任务其实很小可直接 /implement)。
- 其他路由参考:决策已清 → 直接 /to-spec;已有会话过大 → /handoff。

## 四、各阶段要点

**① 访谈(Grill)— 生成前先榨出决策**(整套系统的论点)
- 40~80 个问题、通常 4 轮;**一次一个、每个附推荐答案**(可只回"同意");依赖先决的先问;能查代码库就不问你(先探索)。
- 产出共享理解(shared design concept)。**不要 /clear**,同一会话直接 /to-spec。
- 盘问不出答案的问题(如"UI 长什么样")→ 转 /prototype 做出一次性版本再答。
- 模型选择:grilling 最吃模型理解力,用最好的;实现环节容忍便宜模型。
- 家族:grill-me(stateless 前门)、grill-with-docs(读代码库,写 CONTEXT.md+ADR)、grilling(model-invoked 原语)、wayfinder(巨型任务)、triage(issue 积压盘问)。

**② 写 spec** — 终点文档:问题、方案、用户故事、实现决策、测试决策、out-of-scope。产出后不反复打磨文档,靠"信任 agent 摘要 + 盯模块划分是否贴合现有代码库"。

**③ 拆 tickets** — 垂直切片,每 ticket 声明阻塞边与验收标准、测试要求、HITL/AFK 分类;DAG 化才能多 agent 并行。

**④ 实现** — TDD 红-绿-重构:先写失败测试,防止 agent 事后写出"附和已实现代码"的假测试。执行形态:
- 人工逐 ticket 执行;或 Ralph loop(bash 循环:给 agent 本地 ticket 文件 + 近期 commit + 实现提示词);或 AFK agent(Docker 沙箱,先单次迭代观察调提示词再信任长跑)。
- 实现提示词约定:任务选择优先级、探索仓库、TDD、跑反馈循环(类型检查/测试);错误信息与测试失败必须自动回传。

**⑤ 审查** — 关键原则:**全新上下文**(自己的聪明区)里审查,而非实现会话耗尽后;审查模型 ≥ 实现模型(Pocock 实践:Sonnet 实现 / Opus 审查)。标准分发:pull vs push —— 实现者按需拉取技能规范,审查者被直接推给质量规范逐条对照。自动化可以,但人类负责 QA 与代码 review(审美与架构判断)。

**其他常用技能**:/research(子 agent 外部调研,不占主上下文)、/prototype、/handoff(会话交接/压缩)、/triage(杂务整理成 ticket)、/improve-codebase-architecture(找浅模块→深模块重构)、/ask-matt(路由:不知用哪个就问)、/domain-modeling(统一语言维护)。

## 五、v1.1 变更(vs v1.0)

- to-prd 改名 **to-spec**;"spec" 成为贯穿线术语(文档开头仍提 PRD 便于发现)。
- to-plan + to-issues 合并为 **to-tickets**;to-issues 删除。
- 新增 **/wayfinder**(原 decision-mapping 毕业),定位为"超大会话的 situational on-ramp",主流程仍是 grill 引导的 idea → ship 链。
- grill-with-docs 内联集成 /domain-modeling;tdd/codebase-design 共用术语。
- v1.2 开发中(mattpocock 2026-08 提及)。

## 六、本地已安装情况(opencode 侧)

`~/.config/opencode/skills/` 已装 grill-me / grilling / grill-with-docs / domain-modeling(覆盖①及文档层);缺 to-spec / to-tickets / implement / code-review 等下游。仓库级工程流:`.agents/skills/`(旧,已 gitignore)与全局路径并存,安装记录见 [[03.Engineering/Common_Area/matt-pocock-skills-installation-notes|安装记录]]。

## 慎用提醒

五步仪式适合"犯错成本高"的工作(数据模型、公共 API、钱/认证相关);改一行表单验证跑全套反而亏——**按犯错成本调节仪式强度**(第三方分析观点;alexrusin 拆解:grilling 与书面产物纪律价值最高,spec→tickets 仅在大任务跨会话时值回票价)。

## 参考来源

- 研讨会视频:Full Walkthrough: Workflow for AI Coding (AI Engineer 2026-04-24)
- 官方文档站:https://www.aihero.dev/posts(7 阶段 / grill-me / to-spec / to-tickets / wayfinder 各篇)
- skills 仓库:https://github.com/mattpocock/skills (MIT,含 docs/)
- 解析文章:https://blog.alexrusin.com/agentic-coding-pipeline-matt-pocock-skills/
