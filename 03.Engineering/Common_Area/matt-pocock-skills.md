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

## VSCode + GitHub Copilot 适配方案

Matt 的 skills 为 **Claude Code（CLI 代理）** 设计，Copilot 是编辑器内联补全 + Chat，以下为适配方案：

### 1. CONTEXT.md（最高 ROI）
项目根目录创建 `CONTEXT.md` 定义业务术语。Copilot 补全会自然引用，生成更一致的命名。

### 2. Copilot Custom Instructions
在 `.github/copilot-instructions.md` 配置：
```markdown
要求：
- 变量名与函数名与 CONTEXT.md 中的领域语言保持一致
- 回复直接给技术内容，去掉客套话
- 优先写测试，遵循 TDD 风格
```

### 3. 盘问流程 → Copilot Chat
手动执行：在 Chat 中先描述方案 → 让 Copilot 反问 → 达成共识 → 再写代码。

### 4. TDD 循环 → 内联补全
先写测试的函数签名和断言 → Copilot 自动补全实现 → 跑测试红了 → 补充细节 → 绿。

### 5. 架构重构 → Copilot 对话
Copilot Chat 中引导："分析这个模块的深度，找重构机会，用 CONTEXT.md 的术语描述"

---

## 项目目录推荐结构

```
project-root/
├── CONTEXT.md                    # 领域语言词典（最高优先）
├── AGENTS.md                     # 代理说明（opencode / Claude Code）
├── .github/
│   └── copilot-instructions.md   # Copilot 自定义指令
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

# VSCode + Copilot 用户
# 1. 创建 CONTEXT.md（最重要）
# 2. 配置 .github/copilot-instructions.md
# 3. 在 Copilot Chat 中手动执行盘问和 TDD 流程
# 4. 如果在 opencode 中使用，将 skill 转为 opencode skill 格式
```

**核心原则**: 先用类型和语言对齐，再用 TDD 建立反馈回路，最后通过 Deep Module 重构保持架构健康。
