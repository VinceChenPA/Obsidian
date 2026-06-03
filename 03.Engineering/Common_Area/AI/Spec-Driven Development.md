# Spec-Driven Development (SDD)

## 概述

Spec-Driven Development（SDD）是一种以**规范（spec）优先于代码**的开发方法论。核心思想：先写 spec，再写代码，spec 作为活的真相源与代码共存。

与 TDD 的区别：TDD 是开发者单元级驱动；SDD 是业务级验收驱动，spec 既是文档也是测试。

## 标准流程（5 步闭环）

1. **Write Spec** — 用业务语言写需求/场景，定义"做什么"，不涉及技术实现
2. **Review & Align** — 人与 AI 对齐 spec，确认需求理解正确
3. **Generate Plan** — AI 基于 spec 制定技术方案、架构决策、任务分解
4. **Implement** — AI 按 plan 执行编码，每一步对照 spec 验证
5. **Validate & Archive** — 验证实现是否通过 spec 场景，归档，spec 作为活文档保留

## 三大工具对比

### 1. OpenSpec

- **开源**（MIT），Node.js CLI（`@fission-ai/openspec`）
- **最轻**：3 步走 `/opsx:propose → apply → archive`
- **无绑定**：支持 25+ AI 工具，包括 opencode、Claude Code、Cursor、Copilot
- **Brownfield 优先**：适合现有项目渐进引入
- **核心概念**：Delta Spec（只记录变更差异，归档时自动合并到主 spec）
- **目录结构**：`openspec/specs/`（主 spec）+ `openspec/changes/`（变更）

#### OpenSpec 命令

| 命令 | 用途 |
|---|---|
| `/opsx:propose` | 创建变更 + 生成所有规划 artifacts |
| `/opsx:apply` | 按 tasks.md 实施编码 |
| `/opsx:archive` | 合并 delta spec 到主 spec，归档变更 |
| `/opsx:explore` | 探索式对话，需求不清晰时使用 |
| `/opsx:verify` | 验证实现是否与 artifacts 一致（扩展模式） |

- **VS Code 集成**：通过 GitHub Copilot（`.github/prompts/`）或 Continue 插件（`.continue/prompts/`）
- **Copilot 语法**：`/opsx-propose`（短横线）
- **依赖**：Node.js 20.19+

### 2. GitHub Spec Kit

- **开源**（MIT），由 GitHub 官方出品
- **Python CLI**（`specify-cli`），需 `uv` 包管理器
- **流程最重**：7 步流水线 `constitution → specify → clarify → plan → tasks → implement → taskstoissues`
- **宪法驱动**：`constitution.md` 定义 9 条不可变原则（测试先行、库优先、简洁至上等）
- **可扩展**：Extensions（新命令）+ Presets（覆盖模板）+ 项目 override
- **支持 30+ AI 工具**，但偏重正式工程

#### Spec Kit 命令

| 命令 | 用途 |
|---|---|
| `/speckit.constitution` | 建立项目宪法（行为准则） |
| `/speckit.specify` | 写功能 spec（只说什么，不说怎么） |
| `/speckit.clarify` | 澄清模糊点（NEEDS CLARIFICATION 标记） |
| `/speckit.plan` | 制定技术实现方案 |
| `/speckit.tasks` | 生成可执行任务清单 |
| `/speckit.implement` | 按任务清单实施 |
| `/speckit.taskstoissues` | 转换任务为 GitHub Issues |

- **VS Code 集成**：通过 GitHub Copilot（`.github/prompts/`）
- **Copilot 语法**：`/speckit.specify`（点号）或 `/speckit-specify`（短横线）
- **依赖**：Python 3.11+ + uv，较厚重

### 3. Kiro（AWS）

- **商业闭源**，AWS 出品
- **全栈 IDE/CLI**：不仅是 SDD，还包含多模态聊天、agent hooks、MCP、autopilot
- **工具锁定**：只能在 Kiro IDE/CLI 中使用
- **Credits 计费**，适合愿意付费的一体化体验

### 对比总览

| 维度 | OpenSpec | Spec Kit | Kiro |
|---|---|---|---|
| 厂商 | Fission AI（开源 MIT） | GitHub（开源 MIT） | AWS（闭源商业） |
| 运行时 | Node.js | Python + uv | 自有 CLI/IDE |
| 流程重量 | 轻（3步） | 重（7步 + 宪法） | 中（功能内嵌） |
| 工具绑定 | 无（25+ agent） | 无（30+ agent） | 强锁定 |
| Brownfield | 优先支持 | 支持 | 支持 |
| VS Code 方式 | Copilot / Continue 插件 | GitHub Copilot 插件 | 自有 IDE |
| 适用场景 | 个人/小团队、快速迭代 | 团队正式工程、规范驱动 | 追求一体化体验 |

## AI Vibe Coding 下的 SDD 实践建议

- **小功能 / 个人项目** → OpenSpec（3 步搞定，`/opsx-propose → apply → archive`）
- **团队 / 正式项目** → Spec Kit（宪法约束 + 模板驱动，保证 AI 输出一致性）
- **关键原则**：
  - spec 与代码一起提交，永不丢弃
  - 每次变更产生 delta spec diff
  - 归档前用 verify 检查 AI 实现是否与 spec 一致
  - 一个 change 只做一件事

## 参考

- [OpenSpec GitHub](https://github.com/Fission-AI/OpenSpec)
- [GitHub Spec Kit](https://github.com/github/spec-kit)
- [Kiro](https://kiro.dev/)
