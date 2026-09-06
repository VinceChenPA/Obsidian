---
tags:
  - type/note
  - AI编码
  - DeepSeek Harness
  - Agent 编排
created: 2026-09-06
updated: 2026-09-06
status: done
source: DeepSeek Harness 使用实践总结（会话经验）
---
# DeepSeek Harness 最佳实践

> 软件工程师视角下的 DSH（DeepSeek Harness）使用模式，基于 2026-09-06 实际操作经验整理。
> 相关笔记：[[03.Engineering/Common_Area/AI/Matt-Pocock-AI-工作流详解|Matt Pocock AI 工作流详解]]、[[03.Engineering/Common_Area/AI/AI Workflow 编排（grill-with-docs × OpenSpec × Matt 票流）|AI 工作流编排]]、[[03.Engineering/Common_Area/AI/AI Agent 规则文件体系|AI Agent 规则文件体系]]、[[03.Engineering/Common_Area/matt-pocock-skills|Matt Pocock Skills — AI 编码方法论]]

## 核心心智模型

**不要"和一个 agent 聊天"，要把 repo 里的活拆给多个上下文隔离、有校验、有检查点的执行单元。**

DSH 的核心价值 = 上下文隔离（subagent）+ 并行 fan-out（workflow）+ 跨轮持久目标（goal）+ 可复用流程（skill）。会话上下文是稀缺资源，**能交给子代理的就不占主线**。

## 一、仓库侧（最值得先落地）

1. **每个 repo 根放 `AGENTS.md`**（本 Obsidian 库即为范例）：声明目录结构、构建/测试/lint 命令、代码风格、提交规范、链接规则。这直接决定 agent 行为一致性与产出质量上限，参见 [[03.Engineering/Common_Area/AI/AI Agent 规则文件体系|规则文件体系]]。
2. **修改前同步、修改后验证**：改代码前先同步远端，改完跑 lint/test，而不是让 agent "声称完成"。
3. **让 git + 仓库成为跨会话记忆**，不要把对话历史当记忆——会话会重置，代码不会（与 Matt 的 "会话间只能靠文件传递" 同理）。

## 二、编排层：按任务量级选工具

| 场景 | 工具/形态 |
|---|---|
| 单步任务 / 澄清 | 直接对话 |
| 多步任务 | `todo_write` 拆清单，边做边更新 |
| 复杂改动，先共识再动手 | **Plan mode**：先出完整计划，人审阅批准后再执行 |
| 独立自包含子任务（实现某模块、调研某问题） | `subagent`：后台并行、隔离上下文 |
| 大批量 fan-out（全库审计、迁移、多文件重构、多角度研究） | `workflow`：编排写成 JS 脚本，子任务结果可做 schema 校验 |
| 跨很多轮自动推进的长目标 | `goal`：持久化，可暂停/续跑/多轮自动继续 |
| 长命令（build/test/迁移） | background job：后台跑，`job_output` 收结果，不轮询不 sleep |
| fresh-agent 反复迭代（仅明确要求时） | `ralph`：每轮全新上下文，工作区作共享记忆 |

## 三、软件工程场景 → 现成技能映射

| 场景 | 技能 |
|---|---|
| 写新功能 | `tdd`（红-绿-重构，强制可验证） |
| 审 PR / 分支 / WIP | `code-review`（并行跑 Standards 与 Spec 两条线，对照报告） |
| 疑难 bug / 性能回退 | `diagnosing-bugs`（完整诊断循环） |
| 查 API / 文档事实 | `research`（高可信一手来源，落成 markdown） |
| 接口 / 模块设计 | `codebase-design`（deep module 词汇）、`domain-modeling` |
| 先验证 UI/状态模型 | `prototype` |
| 决策压力测试 | `grilling` |

## 四、黄金流程（feature 开发可直接复用）

1. Plan mode 写实现计划（含验收标准）→ 人批准
2. 按 `tdd` 先写失败测试 → 实现 → 重构，让 agent 自己跑测试
3. 小步提交，git 记录清晰
4. 完成后用 `code-review`（对比基线 commit）自查
5. 并行调研的依赖项丢给后台 subagent，主流程不等它

## 五、反模式（避开）

- 单个会话吞大而全的改动 → 上下文膨胀、无法验证
- 事事开 subagent / workflow → 编排开销超过收益
- 让 agent 输出无法自证的结果（不跑测试就说"应该没问题"）
- 直接改核心库之前不做计划、不留检查点
- agent 未读文件就声称"已检查"——工具层也要求先读后改（read → edit）

## 六、与既有方法论的关系

与 Matt Pocock 流程同构：plan/spec ≈ 4. Spec 阶段，subagent 调研 ≈ 2. Research，垂直切片 + 逐票实现 ≈ 5-6 阶段；DSH 的 `goal`/`workflow` 提供了"票的并行执行层"，`skill` 体系则是 repo 内 `.agents/skills/` 的会话内等价物。核心约束一致：**切小、可验证、文件即接口、频繁干净重置**。
