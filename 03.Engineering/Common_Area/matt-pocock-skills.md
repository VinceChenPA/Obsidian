# Matt Pocock Skills · AI 编码代理方法论

**来源**: [mattpocock/skills](https://github.com/mattpocock/skills) （68k+ stars）
**设计目标**: 给 Claude Code / 终端编码代理使用的斜杠命令集，解决 AI 编码中的常见失败模式。
**核心理念**: 小而精、可组合、基于软件工程经典原理（DDD、TDD、Pragmatic Programmer）。

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
- **解法**: TDD 红-绿-重构循环 + Diagnose 调试流程

### #4 代码变成泥球
- **问题**: AI 加速软件熵，代码迅速腐化
- **解法**: 每天关注架构设计，用 Deep Module 理论重构

---

## Skills 全景

### 工程类（核心）

| Skill | 一句话 | 最佳使用时机 |
|---|---|---|
| `/grill-with-docs` | **盘问 + 建领域词典 + ADR** 三位一体 | 每次开始新功能前 |
| `/grill-me` | 精简版盘问（不写文档） | 快速对齐想法 |
| `/tdd` | 红-绿-重构 + 垂直切片（一个测试→一段代码） | 开发功能、修 Bug |
| `/diagnose` | 6 阶段调试法（反馈回路→复现→假设→插桩→修复→复盘） | 难复现的 Bug、性能退化 |
| `/improve-codebase-architecture` | 用 Deep Module 理论找重构机会 | 每几天跑一次 |
| `/triage` | Issue 状态机管理（GitHub/Linear/本地） | 处理 Issue |
| `/to-prd` | 对话内容 → PRD → 发布到 Issue Tracker | 需求确认后 |
| `/to-issues` | 计划/PRD → 垂直切片 Issue 列表 | 拆分任务 |
| `/prototype` | 快速原型验证（Logic 终端 / UI 多方案） | 设计不确定时 |
| `/zoom-out` | 让代理跳出局部给全局视角 | 遇到不熟悉的代码 |
| `/setup-matt-pocock-skills` | 一次性初始化项目配置 | 新项目首次使用 |

### 效率类

| Skill | 作用 |
|---|---|
| `/caveman` | 极简模式，减少 75% token 消耗 |
| `/write-a-skill` | 创建新 skill 的脚手架 |

### 杂项

`git-guardrails`（防误操作提交）、`migrate-to-shoehorn`（迁移类型断言）、`scaffold-exercises`、`setup-pre-commit`

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
| `.opencode/skills/*/SKILL.md` | **读取** | 忽略 |

### 解决方案：分层配置

```
project/
├── CLAUDE.md                    ← 共享：工程方法论（两边都读）
├── AGENTS.md                    ← 共享：项目上下文/偏好（两边都读）
├── .github/
│   └── copilot-instructions.md  ← 专属：Copilot 行为微调（opencode 忽略）
└── .opencode/
    └── skills/                  ← 专属：opencode 斜杠命令（Copilot 忽略）
```

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

Copilot Agent Mode 也支持 skills，但用的是 **`CLAUDE.md` / `AGENTS.md` 中的自然语言指令**，没有 opencode 的 `SKILL.md` 元数据层。

策略：**`CLAUDE.md` 作为单一事实来源，`.opencode/skills/` 只作为快捷入口**

```
CLAUDE.md（定义所有技能的文字指令）
    ├── Copilot Agent Mode: 直接读取，自然语言触发
    └── opencode: 通过 SKILL.md 引用 CLAUDE.md 的内容
```

### 示例

**`CLAUDE.md`** 中定义技能：
```markdown
## Skills

### grill-me
当我说"盘问我"或"grill me"时：
逐个验证每个设计分支，一次问一个问题，提供你的推荐答案。
能用代码库验证的问题不需要问我。

### tdd
用红-绿-重构循环，一个测试对应一段实现（垂直切片）。
测试通过公开接口验证行为，不测实现细节。
```

**`.opencode/skills/grill-me/SKILL.md`** — opencode 快捷入口（引用 CLAUDE.md）：
```markdown
---
name: grill-me
description: 盘问设计方案。Use when 用户说"盘问我"或"grill me"。
---

见项目根目录 CLAUDE.md 中的 `## Skills` → `### grill-me` 章节。
按该流程严格执行。
```

**`.opencode/skills/tdd/SKILL.md`** — 同理：
```markdown
---
name: tdd
description: TDD 红-绿-重构循环。Use when 用户说"tdd"或"测试驱动"。
---

见项目根目录 CLAUDE.md 中的 `## Skills` → `### tdd` 章节。
按该流程严格执行。
```

### 两套环境下的使用方式对比

| 技能 | opencode 触发 | Copilot Agent Mode 触发 |
|---|---|---|
| grill-me | `/grill-me` 或 "盘问我" | "盘问我"或 "grill me" |
| tdd | `/tdd` 或 "tdd" | "用 TDD 方式写" |
| diagnose | `/diagnose" | "按 diagnose 流程调试" |
| 其他 | 自然语言 | 自然语言 |

**关键原则**：`CLAUDE.md` 是所有技能的唯一事实来源。opencode 的 `.opencode/skills/` 只是添加了斜杠命令快捷方式，不重复定义指令内容。这样修改 `CLAUDE.md` 中的技能逻辑，两边同时生效。

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
├── CLAUDE.md                     # 共享工程规范 + Skills 定义（两边读）
├── AGENTS.md                     # 项目配置 + 工具特定偏好（两边读）
├── .github/
│   └── copilot-instructions.md   # Copilot 专属指令（opencode 忽略）
├── .opencode/
│   └── skills/
│       ├── grill-me/SKILL.md     # opencode 斜杠命令快捷入口
│       └── tdd/SKILL.md
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
# Claude Code 用户
npx skills@latest add mattpocock/skills
# 选择需要的 skill → 运行 /setup-matt-pocock-skills 初始化

# opencode + Copilot Agent Mode 双环境用户
# 1. 创建 CONTEXT.md（最重要）
# 2. 创建 CLAUDE.md 写入共享技能定义
# 3. 在 .opencode/skills/ 创建 SKILL.md 快捷入口（引用 CLAUDE.md）
# 4. 可选：.github/copilot-instructions.md 微调 Copilot 行为
```

**核心原则**: CLAUDE.md 是单一事实来源，opencode skills 只是快捷入口，两边修改都在 CLAUDE.md 中。
