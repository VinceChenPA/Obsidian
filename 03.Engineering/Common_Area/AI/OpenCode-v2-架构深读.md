---
tags:
  - type/note
  - opencode
  - AI-Agent
  - 事件溯源
  - 架构
created: 2026-09-25
updated: 2026-09-25
status: done
source: https://zhuanlan.zhihu.com/p/2076054500859720931
---
# OpenCode v2 架构深读：把编码 agent 当数据库来设计

> 来源：知乎专栏《OpenCode 深读：把编码 agent 当数据库来设计》<https://zhuanlan.zhihu.com/p/2076054500859720931>（2026-09-25 经 zhihu-mcp 抓取）
> 一句话：OpenCode v2 的会话引擎不是聊天循环，而是**事件溯源的持久系统**——对话是追加日志，状态是投影，循环是持久队列，上下文有版本基线，压缩有明确的丢失语义。
> 相关笔记：[[03.Engineering/Common_Area/AI/Loop-Engineering-Opencode-Research|Loop Engineering 研究报告]]、[[03.Engineering/Common_Area/AI/Frontier-Engineering|Frontier Engineering]]、[[03.Engineering/Common_Area/AI/DeepSeek Harness 最佳实践|DeepSeek Harness 最佳实践]]、[[03.Engineering/Common_Area/AI/AI Agent 规则文件体系|AI Agent 规则文件体系]]、[[03.Engineering/Common_Area/Linux/opencode-v2-migration|opencode V1→V2 迁移记录]]

## 核心论点

编码 agent 会话在三个维度"失控"：

1. **活得比内存久**——任务跑数小时，中途崩溃、重启、换机器，内存里的对话状态全丢；
2. **比上下文窗口大**——工具输出、文件内容、命令结果不断涌入，压缩又必然丢信息；
3. **比单个客户端宽**——TUI、IDE、桌面、手机都要看到同一会话的实时状态。

OpenCode 的回答：**别把会话放内存，放进数据库，用事件日志组织它**。这也顺带解释了"为什么必须有 server"——既然循环本身是持久的、可远程唤醒的，TUI 天然只是"订阅事件 + 发指令"的客户端；**client/server 不是产品功能，而是这套架构的必然推论**。

## 四层架构：日志 → 队列 → 基线 → 外存

> 每一层都在把"上下文窗口里的东西"搬到"窗口外的持久化结构"，并给窗口内留一个有界、结构化的视图。

### 1. 会话即日志（事件溯源与投影）

`core/src/session/projector.ts`：把每次变化记录为事件（`SessionV1.Event`），再由 projector 投影成 SQLite 关系表（`SessionTable`/`MessageTable`/`PartTable`/`SessionInputTable`）。

- **成本核算带符号**：`applyUsage(db, sessionID, value, sign = 1)`，撤销时传 `-1`——回滚消息时其消耗的 token 与成本被精确减回。"撤销不是删掉显示，而是对日志做补偿——账要对得上。"
- 由此解释三个"功能"的本质：`/undo`、revert / unrevert = 对事件流补偿或重放；`fork`（任意消息处分叉）= 从日志某点重放到平行分支（事件溯源里的"分支"）；崩溃恢复 = 日志还在，重启重新投影即重建全部状态。

### 2. 循环即队列（durable runner）

`session/runner/index.ts` 注释："Drains eligible durable work"——runner 不是"跑一个 while 循环"，而是**排空已记录的持久化工作**：每次 LLM 调用、每次工具执行，先记录为待办，再被消费。

`run-coordinator.ts`（不到 100 行）实现三个协调语义：

- **按 key 串行**：同一会话同时只有一个执行者，不同会话并行；
- **唤醒合并（wake coalescing）**：执行中产生新工作不打断当前执行，结束后无缝续跑一轮（`pendingWake`）；
- **可中断**：`Fiber.interrupt` 杀掉执行 fiber，等待清理完成。

对比传统 `while(未完成){调模型;执行工具;}`：断电即丢、并发请求打架、重复唤醒跑两份——OpenCode 把这三点全部下沉为协调器语义。

### 3. 上下文即基线（SystemContext 与纪元）

`system-context/index.ts`：把特权系统上下文（AGENTS.md、全局规则等）建模为**可独立刷新的类型化源**；每个源有 key、load、baseline（首次渲染进模型的内容）、update（变更时告知模型的增量）。每个源的最新值持久化为结构化 Snapshot，绑定 `baseline_seq` 序号——即 `context-epoch.ts` 的**"纪元"**。

重启会话时用 `reconcile`（对账）比较磁盘当前值与快照：没变 → Unchanged（不浪费上下文）；变了 → 生成 `ContextUpdated` 事件，原子推进基线。

**最见纪律性的一点**：规则源**读不到时拒绝静默降级**——保留已确认快照，宁可阻塞初始化（`InitializationBlocked`），也不让模型在不知情的情况下跑在残缺规则上（"读不到 ≠ 移除"）。

### 4. 工具输出即外存（ToolOutputStore）

`tool-output-store.ts`：每条工具输出落盘到托管目录，**保留 7 天**；进入上下文的只有有界预览（≤2000 行 / 50KB），且用**头尾采样**（前一半行 + 后一半行）——因为命令输出通常"开头有报错摘要、结尾有结论，中间是重复正文"，比单纯截断信息密度高得多。

## 压缩即交接（compaction 工程学）

`session/compaction.ts` 把"summarize the conversation"做成一个完整的**信息交接协议**：

- **预算（硬编码在文件顶部）**：剩余 20,000 tokens 触发压缩；最近 8,000 tokens 原文保留；单条工具输出截断 2,000 字符；摘要产出上限 4,096 tokens；
- **摘要模板是固定 schema**：Objective（目标）/ Important Details（约束与决策）/ Work State（Completed·Active·Blocked 三态）/ Next Move（编号的下一步）/ Relevant Files（文件 + 为什么重要）；
- **增量更新的丢失语义**（防止"摘要的摘要"逐级衰减）：
  - 上一份摘要合并后即被丢弃——"没带进新摘要的内容将永久丢失"（逼模型面对丢失责任）；
  - 冲突时对话获胜——陈述修正后的事实，丢弃旧说法；
  - 完成的工作从 Active 迁移到 Completed（状态机迁移，而非重新描述）；
  - 不提及摘要/压缩过程本身（防止模型把"我被压缩过"当成任务上下文）。

对照 Anthropic 长任务 harness 的 `claude-progress.txt` 交接文件：**殊途同归——长任务的上下文管理，本质是设计一份"交接文档"的 schema**。区别在于 Anthropic 在博客里教你怎么写 prompt，OpenCode 把 schema、预算、丢失语义直接写进了源码。

## 为什么这一切长在 Effect 上

- **可中断的并发**：`Fiber.interrupt` / `FiberSet` / `Deferred` 做执行池与完成通知（手写极易出错）；
- **副作用显式化**：LLM 调用、DB 写入、文件读取都出现在类型签名里（失败类型可枚举）；
- **依赖注入**：每个模块是 `Context.Service`，Layer 组装依赖图——`TestContext`、测试数据库、假 LLM 可整层替换，核心逻辑才敢全开源。

本质是**用类型系统驯服 agent 运行时**：agent 是大规模、可中断、可恢复的副作用编排，恰好是纯回调或 async/await 最难写对的程序形态。

## 对照 Claude Code：两种"假设"的形状

Anthropic 的定义："harness 的每一个组件，都编码了一条'这件事模型自己做不好'的假设。"

| 维度 | Claude Code | OpenCode v2 |
|---|---|---|
| 假设形状 | **prompt 形状**：subagent / planner / evaluator，靠编排多个模型实例互相制衡 | **存储形状**：日志 / 快照 / 基线 / 外存，靠数据结构保证一致性 |
| 不确定性处理 | 交给"另一个模型的判断" | 消灭在"数据库的事务语义"里 |

两条路线没有高下，但共同验证了同一件事：**编码 agent 的竞争已经从"模型能力"转移到"会话状态的工程化"**——谁能让状态在崩溃、压缩、重启、多端之间保持一致，谁就能把任务时长从分钟推向天。

## 本机对照（实践映射）

| 文章概念 | 本机对应物 |
|---|---|
| 事件投影的 SQLite | `~/.local/share/opencode/opencode.db`（与 `session_memory`、`session_v2` 等表同库） |
| ToolOutputStore 落盘 | `~/.local/share/opencode/tool-output/`（大输出被截断为预览，完整内容留存于此） |
| SystemContext 的源 | 全局/项目 `AGENTS.md` 与 `instructions` 配置 |
| compaction 预算 | 配置里的 `compaction`（V2 已移除 `tail_turns`/`prune`，见 [[03.Engineering/Common_Area/Linux/opencode-v2-migration\|迁移记录]]） |
| client/server 推论 | V2 后台服务 + web/TUI/桌面多客户端共享同一会话（`opencode service` 管理） |
| location 概念 | 同一 MCP 在 `/home/vince` 与项目目录各拉起一份（本机 getnote ×2 的成因） |

> 实用推论：`opencode.db` 的会话/事件表与 `tool-output/` 的留存文件不是缓存垃圾，而是这套持久化设计的正常产物——清理时勿误删数据库。

## 参考资料（文章所列，均为官方来源）

- `anomalyco/opencode` 源码（MIT）：`packages/core/src/session/projector.ts`、`session/runner/index.ts`、`session/run-coordinator.ts`、`system-context/index.ts`、`session/context-epoch.ts`、`session/compaction.ts`、`tool-output-store.ts`
- opencode 官方文档《Server》<https://opencode.ai/docs/server>
- Anthropic 工程博客《Effective harnesses for long-running agents》——"跨上下文窗口保持进度"仍是 open problem
- Anthropic 工程博客《Harness design for long-running application development》——"每个 harness 组件都是一条模型做不好的假设"
