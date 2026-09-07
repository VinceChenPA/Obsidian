---
created: 2026-09-07
updated: 2026-09-07
tags:
  - type/note
  - engineering/ai
  - loop-engineering
  - ai-agent
status: active
source: https://www.runoob.com/ai-agent/loop-engineering.html
---

# Loop Engineering（循环工程）

## 概述

Loop Engineering 是 2026 年 6 月在 AI 编程社区爆火的新概念，由 Google 工程师 **Addy Osmani** 命名并系统化。

一句话定义：**把工程师从"提示 Agent 的人"变成"设计提示 Agent 的系统"的工程师**——你不该再手动提示 coding agent，而应设计让 Agent 自己提示自己的 Loop。

> Prompt Engineering 并未消亡：一个 Loop 由多个 Prompt 组成，写得差的 Prompt 放进 Loop 只会让糟糕的工作以更快的速度产出。Loop Engineering 是 Prompt Engineering 之上的层次，不是替代。

## 起源（2026-06 引爆点）

- **Boris Cherny**（Anthropic Claude Code 负责人）公开演讲："我不再直接提示 Claude 了。我有一套 Loop 在运行，它们负责提示 Claude 并决定下一步做什么。我的工作是编写 Loop。"
- **Peter Steinberger**（OpenClaw 作者，GitHub 历史上获星最快的新仓库）2026-06-07 推文："You shouldn't be prompting coding agents anymore. You should be designing loops that prompt your agents."
- 随后 Addy Osmani 在 Substack 发文正式命名并系统化为独立工程学科。

## AI 工程演进四层次

| 工程阶段 | 核心思想 | 关注点 | 人的角色 |
|---|---|---|---|
| Prompt Engineering | 通过设计提示词获得更好输出 | 怎么问问题 | 提问者 |
| Context Engineering | 组织并提供完整背景信息 | 给 AI 什么信息 | 信息组织者 |
| Harness Engineering | 连接模型、工具、数据形成工作流 | 如何调用能力 | 系统设计者 |
| Loop Engineering | 构建目标驱动的自主闭环系统 | 如何持续完成目标 | 规则制定者 |

背景：AI 编程工具三代演进——第一代自动补全（Copilot 早期）→ 第二代对话式（ChatGPT）→ 第三代 Agent 自主循环（Claude Code / Codex / opencode）。第三代意味着工程师核心竞争力从"会写提示词"变为"会设计 Loop"。

## 内循环 vs 外循环

- **内循环**（Agent 内置）：感知 → 推理 → 行动 → 观察 → 再循环（读文件 → 改代码 → 跑测试 → 读错误 → 再修改）
- **外循环**（工程师设计）：按计划发现任务 → 分派 Agent → 验证结果 → 记录状态 → 开启下一轮

Loop 的力量不在任何单独步骤，而在于**闭环**：测试失败不只是错误消息而是新上下文；类型错误不只是阻断而是错误假设的信号；Review 评论不只是反馈而是驱动下一步的新观察。

## 六大构成要素

1. **自动触发器（Automations）**：定义"什么时候、做什么"。定时调度（如 `/loop --schedule`、cron）。⚠️ 定时 Loop 每次触发消耗 Token，建议先慢节奏（每天一次）观察成本。
2. **并行隔离（Git Worktrees）**：多 Agent 各自独立工作目录/分支，共享 Git 历史但文件改动隔离。审查瓶颈（而非工具限制）决定可并行 Agent 数量上限。
3. **技能文件（Skills）**：SKILL.md 沉淀项目约定、构建步骤、"我们不这样做是因为那次事故"。避免每次新对话从零推断规范。
4. **连接器（Connectors/MCP）**：打通 Issue 追踪、数据库、Slack 等外部系统。必须最小权限；高风险操作（推送/合并/外部通知）要求人工审批。
5. **子 Agent（Maker-Checker）**：写代码的 Agent 与检查代码的 Agent 分离——写代码的模型评分自己作业会过于宽容。检查者用独立（更强）模型对抗性审查。Claude Code 的 `/goal` 也由单独模型判断"是否完成"。
6. **持久记忆（Memory）**：状态写在文件里、文件放在仓库里——"仓库记得，即使模型不记得"（TODO.md 状态文件惯例）。

## 五种常见 Loop 模式

| 模式 | 核心观察信号 | 停止条件 | 典型场景 |
|---|---|---|---|
| 测试驱动 Loop | 测试通过/失败 | 目标测试全部通过 | Bug 修复、回归测试 |
| 编译器驱动 Loop | 类型/编译错误 | 类型检查零错误 | TS 迁移、依赖升级、重构 |
| Review 驱动 Loop | 人工 Review 评论 | 所有评论被处理或合理忽略 | PR Review 跟进 |
| 运行时调试 Loop | 日志/堆栈/HTTP 响应 | 复现 → 假设 → 验证修复 | 生产 Bug、性能问题 |
| 产品迭代 Loop | 截图/浏览器检查 | 与设计稿对齐 | 落地页、UI 调整 |

## 构建方法论

### 渐进四阶段（自主程度）

1. **只读**：发现问题、分类任务、写状态文件（人类审查 TODO.md 决定处理顺序）← 推荐第一个 Loop
2. **草稿**：起草修复、跑测试、写入分支（人类审查 diff 后手动 push）
3. **半自动**：开 Draft PR、跑 CI、通知 Slack（人类审查 PR 手动 Merge）
4. **全自动**：Maker + Checker 双 Agent，CI 通过自动合并（异常人工介入）

### 任务定义要点

- 从窄任务开始（"修复 test/checkout/tax.spec.ts 中失败的税额计算测试"优于"修复 checkout 问题"）
- 明确告知验证方式（验证命令/成功标准写进指令，让"完成"可测量）
- 设置保险机制（先只写文件不做外部操作）
- 偏好小的可逆变更；尊重现有代码模式
- 人类保留判断席位（产品判断、架构决策、最终 Review）

## 故障模式与风险

### 四类故障模式

| 故障模式 | 表现 | 根因 | 解法 |
|---|---|---|---|
| 空转 Thrashing | 反复修改不收敛 | 目标不清/信号噪声/改动太大 | 缩小目标、减小 diff、更可靠验证命令 |
| 过拟合测试 | 测试全过但功能错误 | 测试覆盖窄 | 自动测试 + 人工验收 + E2E |
| 上下文漂移 | 基于过期假设工作 | 未刷新上下文 | 重要观察后重新收集上下文 |
| 不安全自主 | 无授权破坏性操作 | 权限过宽、无停止条件 | 最小权限、高风险人工审批、明确停止规则 |

### 三大风险

1. **验证仍是人的责任**：无人值守的 Loop 也是无人值守制造错误的 Loop；"通过了验证"是声明不是证明。
2. **理解债（Comprehension Debt）**：Loop 越快，你真正理解的代码比例越低——解药是读 Loop 产出的代码。
3. **认知投降（Cognitive Surrender）**：Loop 运转时接受任何返回最舒适，是最隐性危险。同一 Loop 两人用可得出相反结果：一个深化理解，一个回避理解。

## opencode 支持映射（本机 1.18.9 实测）

| Loop 要素 | opencode 机制 | 命令/配置 |
|---|---|---|
| 自动触发器 | `opencode run` 无头模式 + crontab/systemd；`--auto` 自动批准权限 | `opencode run "msg" -s <id> --fork`；`--command`；`--format json` |
| 常驻/服务化 | `opencode serve` + `run --attach`；web 模式已有 | `opencode run --attach http://localhost:4096 --dir <path>` |
| 并行隔离 | git worktree + `--dir`；`opencode pr <n>` 检出 PR 分支跑 agent | `git worktree add ... && opencode run --dir <wt>` |
| 技能文件 | 原生一等公民：`.opencode/skills/` + 全局 `~/.config/opencode/skills/` 自动发现 SKILL.md | skill 工具按需注入 |
| 连接器 | `opencode mcp` 命令 + opencode.json `mcp` 段 | GitHub/Slack/DB 等 server |
| 子 Agent | `opencode agent create`（`.opencode/agent/*.md`，可指定 model/tools）；task 工具派发 | reviewer agent 配更强模型 = Maker-Checker |
| 持久记忆 | 三机制：AGENTS.md 自动加载 / session_memory 表（`opencode db`）/ 仓库内状态文件 | `opencode session list`、`--continue` |
| 会话延续 | `--continue` / `-s <session_id>` / `--fork` 跨触发延续上下文 | 配合 cron 形成跨会话 Loop |

**实践路径**：opencode 本身是第三代 agent（内置感知-行动-观察内循环），用户作为 Loop 工程师用 `cron + run + agent + skill + mcp + session_memory` 搭建外循环——已有实例：每日新闻 cron（12:00 定时 `opencode run`）。建议从只读分类 Loop（写 TODO.md）起步，逐步向半自动 Draft PR 升级。

## 相关笔记

- [[03.Engineering/Common_Area/AI/Loop-Engineering-Opencode-Research|Loop Engineering with opencode — 一手来源研究报告]]
- [[03.Engineering/Common_Area/AI/Spec-Driven Development|Spec-Driven Development (SDD)]]
- [[03.Engineering/Common_Area/AI/AI Agent 规则文件体系|AI Agent 规则文件体系]]
- [[03.Engineering/Common_Area/AI/DeepSeek Harness 最佳实践|DeepSeek Harness 最佳实践]]
- [[03.Engineering/Common_Area/AI/MCP|MCP 协议]]
