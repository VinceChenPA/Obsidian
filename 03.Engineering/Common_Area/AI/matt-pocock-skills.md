# Matt Pocock Skills · AI 编码代理方法论

**来源**: [mattpocock/skills](https://github.com/mattpocock/skills) （2026-09-06 核对至 **v1.2.3**）
**设计目标**: 给 Claude Code / Codex / 其他终端编码代理使用的斜杠命令集，解决 AI 编码中的常见失败模式。
**分发方式**: Claude Code 官方插件（`claude plugins install mattpocock-skills`，托管只读、自动更新）或 skills.sh 复制可编辑文件（`npx skills@latest add mattpocock/skills`，支持 Codex 等，含 `agents/openai.yaml` Codex 元数据）。
**核心理念**: 小而精、可组合、基于软件工程经典原理（DDD、TDD、Pragmatic Programmer）。
**技能分类轴**: **User-invoked**（只能人敲，负责编排）/ **Model-invoked**（模型可自动调用，承载可复用纪律）；按目录分桶 `engineering/`、`productivity/`（随插件发布）+ `misc/`、`in-progress/`（beta 不发布）、`deprecated/`（空）。

---

## 四大失败模式与对应解法

### #1 代理没理解需求
- **问题**: 你说的和代理理解的不一致
- **解法**: 盘问（Grilling）— 让代理在动手前详细问你

### #2 代理话太多
- **问题**: 代理用 20 个词说 1 个词能表达的事
- **解法**: 建立统一领域语言 — `CONTEXT.md` 词典

### #3 代码跑不通
- **问题**: 代理写代码没有反馈回路，盲目编码
- **解法**: 反馈回路（静态类型、浏览器、自动化测试）+ `/tdd` 红-绿-重构 + `/diagnosing-bugs` 调试回路

### #4 代码变成泥球
- **问题**: AI 加速软件熵，代码迅速腐化
- **解法**: 每天关注架构设计，用 Deep Module 理论重构

---

## Skills 全景（v1.2.3）

按目录分桶：`engineering/`（代码工作）、`productivity/`（非代码工作流）、`misc/`（保留不推广）、`in-progress/`（beta 通道）、`deprecated/`（已弃用，现空）。随 Claude Code 插件发布的是 **engineering + productivity** 两桶。

### engineering/（代码工作，每日使用）

**User-invoked（只能手动触发，负责编排）**

| Skill | 一句话 | 最佳使用时机 |
|---|---|---|
| `/ask-matt` | 路由：问"该用哪个技能/流程" | 不确定用哪个 |
| `/grill-with-docs` | **盘问 + 建 CONTEXT.md + 更新 ADR** 三位一体 | 每次开始新功能前（有代码库时） |
| `/triage` | Issue/外部 PR 状态机管理 | 处理流入的 issue |
| `/improve-codebase-architecture` | 扫描代码库找深化机会 → HTML 报告 → 盘问选中项 | 每几天跑一次 |
| `/setup-matt-pocock-skills` | 一次性初始化 repo 配置（issue tracker/triage labels/domain docs） | 新 repo 首次使用工程技能前 |
| `/to-spec` | 当前会话 → spec 发布到 issue tracker | 需求确认后（不盘问，只综合） |
| `/to-tickets` | spec/计划 → 曳光弹垂直切片 tickets（声明阻塞边） | 拆分任务 |
| `/implement` | 逐 ticket 构建，内部驱动 `/tdd`，收尾跑 `/code-review` | 执行阶段 |
| `/wayfinder` | 超大会话的共享决策图（decision tickets），逐张解析到路径清晰 | 巨型任务、路线不明 |

**Model-invoked（模型或人都可触发，承载可复用纪律）**

| Skill | 一句话 | 最佳使用时机 |
|---|---|---|
| `/tdd` | 红-绿-重构 + 垂直切片（seam 处测试） | 开发功能、修 Bug |
| `/diagnosing-bugs` | 纪律化调试回路（反馈回路→最小化→假设→插桩→修复→回归测试） | 难复现的 Bug、性能退化 |
| `/code-review` | 双轴并行审查：Standards（含 Fowler smell 基线）+ Spec | 提交前审 diff |
| `/codebase-design` | 深模块词汇（module/interface/depth/seam/adapter） | 设计模块接口 |
| `/domain-modeling` | 打磨领域语言，维护 CONTEXT.md + ADR | 术语含糊时 |
| `/research` | 后台 agent 查一手来源，产出带引用的 Markdown | 委托阅读调研 |
| `/prototype` | 一次性原型回答设计问题（单个共享 HTML / 多套 UI 变体） | 设计不确定时 |
| `/resolving-merge-conflicts` | 逐 hunk 按意图解决 merge/rebase 冲突 | 已在冲突中 |
| `/wizard` | 生成交互 bash wizard，引导人类完成仅人能做的步骤 | 配凭据/CI secrets/第三方面板/一次性迁移 |

### productivity/（非代码工作流）

**User-invoked**

| Skill | 作用 |
|---|---|
| `/grill-me` | 无状态盘问（不写 CONTEXT.md/ADR，无 working directory 时用） |
| `/handoff` | 把当前会话压成交接文档给另一 agent（仅跨 harness/目录/同事时用） |
| `/teach` | 多会话教学，用当前目录做有状态工作区 |
| `/to-questionnaire` | 无法独自回答的决策 → 变成问卷发给能回答的人 |
| `/wait-what` | 一句话没懂时：用 CONTEXT.md 词汇、简练英文重讲一遍 |

**Model-invoked**

| Skill | 作用 |
|---|---|
| `/grilling` | 盘问原语（round-by-round frontier），grill-me/grill-with-docs/triage/wayfinder 的底层 |
| `/writing-for-agents` | 写 agent 消费的文档（skills、AGENTS.md/CLAUDE.md、指针指向的 docs） |

### 其他桶（不随插件发布）

- **misc/**：`git-guardrails-claude-code`（防误操作提交）、`migrate-to-shoehorn`（迁移类型断言）、`scaffold-exercises`、`setup-pre-commit`
- **in-progress/**（beta）：`claude-handoff`、`loop-me`、`setup-ts-deep-modules`、`writing-beats`、`writing-fragments`、`writing-shape`
- **deprecated/**：空

### 历史更名速查（读旧文用）

- `to-prd` → `to-spec`；`to-plan`/`to-issues` 合并 → `to-tickets`（v1.1）
- `diagnose` → `diagnosing-bugs`；`caveman`、`zoom-out` 删除（v1.0）
- `write-a-skill` → `writing-great-skills` → `writing-for-agents`（v1.2，还管 AGENTS/CLAUDE 文档）
- `decision-mapping` → `wayfinder`（v1.1）；`ubiquitous-language` → `domain-modeling`、`design-an-interface` → `codebase-design`、`qa` → `triage`+`to-tickets`、`request-refactor-plan` → `to-spec`+`improve-codebase-architecture`（v1.2）
- `review`(in-progress) → `code-review`（v1.1）；`grill-with-docs` 不再"内联 domain-modeling"，而是独立技能协作

---

## 核心文件体系

### CONTEXT.md — 领域语言词典

项目根目录下的 `CONTEXT.md` 是整个 skill 体系的**基石**。它定义项目中的业务术语标准名称，让代理用最少的词表达最精确的含义。

示例对比：
- **没有 CONTEXT.md**: "There's a problem when a lesson inside a section of a course is made 'real'"
- **有 CONTEXT.md**: "There's a problem with the materialization cascade"

其他好处：
- 变量、函数、文件名保持一致
- 代理导航代码更快
- 节省大量 token（思考成本降低）

### docs/adr/ — 架构决策记录

当满足**三个条件**时才记录 ADR：
1. 难以逆转
2. 没有上下文会让人困惑
3. 是真正的权衡取舍

### docs/agents/ — 代理配置

由 `/setup-matt-pocock-skills` 生成，包含：
- `issue-tracker.md` — Issue 跟踪方式
- `triage-labels.md` — 标签映射
- `domain.md` — 领域文档消费规则

### .out-of-scope/ — 已拒绝需求存档

记录已决定不做或拒绝的功能请求，避免代理反复提出相同建议。

---

## 关键问题：opencode + Copilot Agent Mode 双环境配置

两套环境的文件读取规则：

| 文件 | opencode | Copilot Agent Mode |
|---|---|---|
| `AGENTS.md` | **读取**（主要） | 读取 |
| `CLAUDE.md` | 读取 | **读取** |
| `.github/copilot-instructions.md` | 忽略 | **读取**（最高优先级） |
| `.agents/skills/*/SKILL.md` | **读取** ✅ 原生支持 | 读取（通过 CLAUDE.md/AGENTS.md 引用） |
| `.github/skills/` | 忽略 | 生态预留 |
| `.opencode/skills/*/SKILL.md` | **读取** | 忽略 |
| `.claude/skills/*/SKILL.md` | **读取**（兼容） | 忽略 |

### 解决方案：双目录协作 + 分层配置

**opencode 官方已原生支持 `.agents/skills/`**（详见 [docs](https://opencode.ai/docs/skills/)），这是当前最优的共享路径。

```
project/
├── .agents/skills/              ← 共享：opencode + Claude Code 都读取
│   ├── grill-me/SKILL.md
│   └── tdd/SKILL.md
├── .github/
│   ├── copilot-instructions.md  ← Copilot 专属指令（opencode 忽略）
│   └── skills/                  ← Copilot 生态预留（未来扩展）
├── CLAUDE.md                    ← 共享：工程方法论（两边都读）
├── AGENTS.md                    ← 共享：项目上下文/偏好（两边都读）
├── .opencode/
│   └── skills/                  ← 专属：opencode 斜杠命令（Copilot 忽略）
└── .claude/
    └── skills/                  ← 专属：Claude Code 兼容路径
```

### 各目录职责

| 目录 | 谁读 | 格式 | 作用 |
|---|---|---|---|
| `.agents/skills/` | opencode ✅, Claude Code ✅ | `SKILL.md` | **共享技能目录**，工具中立 |
| `.opencode/skills/` | opencode 仅 | `SKILL.md` | opencode 专属覆盖 |
| `.claude/skills/` | opencode ✅（兼容）, Claude Code ✅ | `SKILL.md` | Claude Code 专属覆盖 |
| `.github/skills/` | Copilot 生态（预留） | 自然语言 md | Copilot 专属技能定义 |
| `.github/copilot-instructions.md` | Copilot ✅ | 自然语言 md | Copilot 项目级指令 |

### 每份文件的职责

**`CLAUDE.md`** — 核心工程规范，两边共享：
```markdown
# 工程规范

## 领域语言
- CONTEXT.md 定义了所有业务术语标准名称
- 代码命名与 CONTEXT.md 保持一致

## 工作流
- 新功能前：先盘问方案，再动手
- 编码时：先写类型，再写实现，TDD 垂直切片
- 调试时：先建反馈回路 → 列假设 → 逐一验证
- 定期检查架构深度

## Skills
### grill-me
当用户说"盘问我"或"grill me"时：
逐个验证每个设计分支，一次问一个问题，提供推荐答案。
能用代码库验证的问题不需要问我。

### tdd
用红-绿-重构循环，一个测试对应一段实现（垂直切片）。
测试通过公开接口验证行为，不测实现细节。
```

**`AGENTS.md`** — 项目上下文 + 工具特定配置：
```markdown
# 项目配置

## 项目
- 名称/用途/仓库地址

## 环境偏好
- 语言、模型、超时设置等

## 工具特定说明
- opencode: session memory 用法、技能路径
- 通用: 写入 supermemory 前检查隐私信息
```

**`.github/copilot-instructions.md`** — Copilot 行为微调（仅 Copilot 读取）：
```markdown
参考 CLAUDE.md 中的工程规范。

额外要求：
- 函数注释用 JSDoc
- 先提方案，确认后再实现
- 回复简洁，去掉客套话
```

---

## 同一套技能在两套环境都生效

opencode 原生支持 `.agents/skills/`、`.opencode/skills/` 和 `.claude/skills/` 三个路径。Claude Code 也读 `.agents/skills/` 和 `.claude/skills/`。Copilot Agent Mode 读 `CLAUDE.md` / `AGENTS.md` 中的自然语言指令。

策略：**`.agents/skills/` 作为共享 SKILL.md 源，`CLAUDE.md` 提供自然语言定义供 Copilot 读取**

```
.agents/skills/（SKILL.md 格式）
    ├── opencode: 直接通过 skill 工具加载 ✅
    ├── Claude Code: 直接通过 skill 工具加载 ✅
    └── Copilot: 通过 CLAUDE.md 引用（自然语言触发）
```

### 三套环境的实际读取链路

| 环境 | 技能定义来源 | 触发方式 |
|---|---|---|
| opencode CLI | `.agents/skills/*/SKILL.md` | skill 工具自动发现，或 `/skill-name` 斜杠命令 |
| Claude Code CLI | `.agents/skills/*/SKILL.md` | skill 插件加载 |
| Copilot Agent Mode | `CLAUDE.md` / `AGENTS.md` 中的文字描述 | 自然语言触发 |

### 推荐配置

**`.agents/skills/grill-me/SKILL.md`** — 共享技能定义（opencode + Claude Code 原生加载）：
```markdown
---
name: grill-me
description: 盘问设计方案。Use when 用户说"盘问我"或"grill me"。
---

逐个验证每个设计分支，一次问一个问题，提供你的推荐答案。
能用代码库验证的问题不需要问我。
```

**`CLAUDE.md`** — 为 Copilot 提供同等的自然语言技能描述：
```markdown
## Skills

### grill-me
当我说"盘问我"或"grill me"时：
逐个验证每个设计分支，一次问一个问题，提供推荐答案。
能用代码库验证的问题不需要问我。

### tdd
用红-绿-重构循环，一个测试对应一段实现（垂直切片）。
测试通过公开接口验证行为，不测实现细节。
```

**`.github/copilot-instructions.md`** — Copilot 专属引用：
```markdown
参考 CLAUDE.md 中的工程规范和 Skills 定义。
本项目的共享技能定义在 .agents/skills/ 目录下，遇到相关任务时读取对应内容。
```

### 为什么 `.agents/skills/` 是最佳共享路径

- **opencode 官方已原生支持**（同 `.opencode/skills/` 同级）
- **Claude Code 兼容**（也可以用自己的 `.claude/skills/`）
- **工具中立** — `.agents/` 正在成为跨工具标准目录名
- 如果某个工具需要专属覆盖，可以在自己的目录（`.opencode/skills/`、`.claude/skills/`）放同名 skill，优先级高于 `.agents/skills/`

### 三套环境触发方式对比

| 技能 | opencode | Claude Code | Copilot Agent Mode |
|---|---|---|---|
| grill-me | `/grill-me` 或 "盘问我" | `/grill-me` 或 "盘问我" | "盘问我"或 "grill me" |
| tdd | `/tdd` 或 "tdd" | `/tdd` 或 "tdd" | "用 TDD 方式写" |
| diagnosing-bugs | `/diagnosing-bugs` 或"调试这个 bug" | `/diagnosing-bugs` | "按 diagnosing-bugs 流程调试" |

**关键原则**：`.agents/skills/` 是共享源（opencode + Claude Code 原生加载），`CLAUDE.md` 为 Copilot 提供同等的自然语言版本。一套技能定义，三套环境都可以使用。

---

## VSCode + GitHub Copilot 适配方案

Matt 的 skills 为 **Claude Code（CLI 代理）** 设计，以下为在 Copilot 中的适配：

### 1. CONTEXT.md（最高 ROI）
项目根目录创建 `CONTEXT.md` 定义业务术语。Copilot 补全会自然引用，生成更一致的命名。

### 2. 共享 CLAUDE.md
按上述分层方案配置 `CLAUDE.md`，Copilot Agent Mode 会自动读取其中的技能定义。

### 3. 盘问流程 → Copilot Chat
手动执行：在 Chat 中先描述方案 → 让 Copilot 反问 → 达成共识 → 再写代码。

### 4. TDD 循环 → 内联补全
先写测试的函数签名和断言 → Copilot 自动补全实现 → 跑测试红了 → 补充细节 → 绿。

### 5. 架构重构 → Copilot 对话
Copilot Chat 中引导："分析这个模块的深度，找重构机会，用 CONTEXT.md 的术语描述"

---

## 项目目录完整推荐结构

```
project-root/
├── CONTEXT.md                    # 领域语言词典（最高优先）
├── CLAUDE.md                     # 共享工程规范 + Skills 定义（Copilot 读取）
├── AGENTS.md                     # 项目配置 + 工具特定偏好（两边读）
├── .agents/
│   └── skills/                   # 共享技能目录（opencode + Claude Code 原生加载）
│       ├── grill-me/SKILL.md
│       └── tdd/SKILL.md
├── .github/
│   ├── copilot-instructions.md   # Copilot 专属指令（opencode 忽略）
│   └── skills/                   # Copilot 生态预留（可选）
├── .opencode/
│   └── skills/                   # opencode 专属覆盖（优先级最高）
├── .claude/
│   └── skills/                   # Claude Code 专属覆盖（可选）
├── docs/
│   ├── adr/                      # 架构决策记录
│   │   ├── 0001-why-postgres.md
│   │   └── 0002-auth-strategy.md
│   └── agents/                   # 代理配置文件
│       ├── issue-tracker.md
│       ├── triage-labels.md
│       └── domain.md
├── .out-of-scope/                # 已拒绝需求存档
└── .scratch/                     # 本地 Issue 跟踪（可选）
```

---

## 快速开始

```bash
# Claude Code 用户（官方插件，托管只读、自动更新）
claude plugins install mattpocock-skills
# 或 /plugin install mattpocock-skills

# Codex / 其他代理 / 想自己改文件（复制可编辑版本）
npx skills@latest add mattpocock/skills
# 选择需要的 skill（务必含 setup-matt-pocock-skills）→ 运行 /setup-matt-pocock-skills 初始化

# 只装单个 skill
npx skills add mattpocock/skills --skill=grill-me -y -g

# opencode + Copilot Agent Mode + Claude Code 三环境用户
# 1. 创建 CONTEXT.md（最重要）
# 2. 在 .agents/skills/ 创建 SKILL.md（opencode + Claude Code 原生加载）
# 3. 在 CLAUDE.md 写入同等的自然语言技能定义（Copilot 读取）
# 4. 可选：.github/copilot-instructions.md 微调 Copilot 行为
# 5. 可选：.opencode/skills/ 添加 opencode 专属覆盖
```

**核心原则**: `.agents/skills/` 是共享 SKILL.md 源（opencode + Claude Code 原生加载），`CLAUDE.md` 为 Copilot 提供同等的自然语言版本。一套技能定义，三套环境都可以使用。
