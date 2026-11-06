---
tags:
  - type/note
  - Agent-编排
  - DeepSeek-Harness
created: 2026-11-07
updated: 2026-11-07
status: done
source: 98.Raw/2026-11-07-zhihu-dsh-super-agent.md
---
# Super-Agent 架构（DeepSeek Harness）

> 知乎答主 cxuan 对 DSH 团队「Super-agent」构想的解读与拆解，2026-11-07 抓取。
> 原文见 [[98.Raw/2026-11-07-zhihu-dsh-super-agent|原始回答]]（只读层）；回答页 154 赞、8 条评论。

## 一句话

**单任务的上限取决于模型能力，多任务的上限取决于人的注意力**（约 3–5 个 Session）。所以要让系统接管上下文与并发，把 Session 从用户侧淡化——用户始终只面对一个 Super-agent。

## 问题从哪来

- 模型变强后，人从执行层上移到「顶层决策与验收」，形态演化为 Personal Agent；但这类产品本身创新有限，主要是更好用、延迟更低。
- 现有 Agent 仍是「单 Session + 手动切换」：人脑并行上限只有 3–5 个 Session，再多记不住、效果断崖。
- 靠 MCP 扩展也绕不开上下文混乱、单会话同步阻塞。
- 结论：需要**跨会话保持上下文、同时推进多项任务**的架构。

## 与 Claude Code 的分工差异

| | Claude Code 路线 | DSH Super-agent |
|---|---|---|
| 优化对象 | 一个任务**内部**的专家交接（单 Agent + Main Loop，subagent 流程提炼为 Skills，读写交给 Tools/MCP） | 一组任务**之间**的长期并行与少操心 |
| 关心的问题 | 多专家反复交接是否丢信息、增延迟 | 跨任务如何持续并行推进 |

## 五块架构

| 模块 | 要点 |
|---|---|
| **双层架构** | **Router**：常驻全局路由层，汇总跨任务信息与决策，只做分发、不执行任务；多节点结构，轻量模型即可。**Worker**：上下文隔离、工具完整、专注单一任务，需要最强模型；Worker 间不互相通信，可 fork subagent 自成 Agent Teams。过滤后的消息多数直接投给 Worker，Router 只负责新任务冷启动与跨任务协调 |
| **事件驱动** | IM 群消息、邮件日程、PR 评论、CI 结果等环境事件称 **Trigger**；一串相关 Trigger 构成 **Thread**（同一 PR、同一邮件往来、同一 IM 话题）。两层降噪：程序层 **Fast Path** 先滤掉大部分噪音，语义层 **Slow Path** 才进 LLM 判断 |
| **上下文注入** | 借鉴编译优化：**Inline** 把近期高频信息内联进窗口，**Spill** 把超预算/低频信息留在窗口外只留引用或摘要。类比内存页换入换出，但关键差异是——缺页由内核自动完成，Spill 后要模型**自己意识到缺什么**再主动读回，因此对模型能力要求更高 |
| **多信道交互** | **异步 ReAct Loop**：`start()` 立即返回，不阻塞前台对话，实现全双工；替代「一问一答、必须等完」的阻塞式 ReAct。原文称该机制**已在 DSH 中实现但尚未默认开启**（待核对版本） |
| **计算/状态/环境分离** | 计算（Agent Loop）进云端可替换容器，会话与任务状态单独存库——容器挂了换一个、状态还在即可续跑；环境**不搬上云**，本地装轻量 daemon 只执行命令并回传，即 **BYOM（Bring Your Own Machine）**。代价：云端被攻破会危及所有接入设备 |
| **Watchdog** | 主系统之外的独立外部控制回路（Control Loop），定时消息驱动的 Agent Session。理由：Agent 失效常常不是进程没了，而是 **LLM 内部死锁或偏离目标**，传统存活探测/保活脚本发现不了；Watchdog 用于及时介入、纠偏，也可直接问它错误原因 |

## 两个补充机制

- **链式召回（Chain of Recall）**：给 Trigger / Thread / Worker 都编 ID，注入上下文时顺手带上「下一跳」ID，对象在底层连成一张引用图。回答「昨天那个问题修得怎么样了」时，从消息 → Thread → Worker → PR Thread → 修改记录逐跳查。靠的是**能实时渲染的地图**，而不是巨型记忆库。
- **投机执行（Speculative Execution）**：结果未定前按环境信号在后台预跑，用额外算力换人类等待时间；有价值就交付，没价值就丢弃。

## 作者评价与边界

- 作者认为这是**从底层动刀**（session 共享、缓存、fork、worktree、文件系统），而非继续堆 tools/skills，明确看好这条通往 AGI 的路线。
- 原文为口语化解读，含少量错字（如「Woker」「绕要绕去」）；观点属答主个人推断，非 DSH 官方文档，引用时注意区分。
- 未验证项：异步 ReAct Loop 的「已实现、未默认开启」说法；BYOM 的安全模型细节。

## 相关

- [[03.Engineering/Common_Area/AI/DeepSeek Harness 最佳实践|DeepSeek Harness 最佳实践（软件工程师视角）]]
- [[03.Engineering/Common_Area/AI/Claude Code|Claude Code — 规则文件/skill 布局/命令清单与 opencode·DSH 对照]]
- [[03.Engineering/Common_Area/AI/OpenCode-v2-架构深读|OpenCode v2 架构深读]]
- [[03.Engineering/Common_Area/AI/Loop-Engineering|Loop Engineering（循环工程）]]
- [[03.Engineering/Common_Area/AI/AI Agent 规则文件体系|AI Agent 规则文件体系]]
