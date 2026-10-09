---
tags:
  - type/note
  - Claude-Code
  - AI-Agent
  - AI编码
  - harness
  - 规则文件
created: 2026-10-09
updated: 2026-10-09
status: done
source:
  - https://code.claude.com/docs/en/memory
  - https://code.claude.com/docs/en/worktrees
  - https://code.claude.com/docs/en/skills
  - https://code.claude.com/docs/en/commands
  - https://code.claude.com/docs/en/goal
  - https://code.claude.com/docs/en/scheduled-tasks
  - "[[03.Engineering/Common_Area/AI/OpenCode-v2-架构深读]]"
  - "[[03.Engineering/Common_Area/AI/Loop-Engineering-Opencode-Research]]"
  - "[[03.Engineering/Common_Area/AI/Loop-Engineering]]"
  - "[[03.Engineering/Common_Area/AI/AI Agent 规则文件体系]]"
  - "[[03.Engineering/Common_Area/AI/matt-pocock-skills]]"
  - "[[03.Engineering/Common_Area/AI/matt-pocock-skills-installation-notes]]"
  - "[[03.Engineering/Common_Area/AI/Matt-Pocock-AI-工作流详解]]"
  - "[[03.Engineering/Common_Area/AI/Snippets]]"
  - "[[03.Engineering/Common_Area/AI/Spec-Driven Development]]"
  - "[[03.Engineering/Common_Area/AI/Frontier-Engineering]]"
  - "[[03.Engineering/Common_Area/AI/copilot vs opencode]]"
  - "[[03.Engineering/Common_Area/AI/AI时代的程序员]]"
---
# Claude Code

> 本库中被反复当作对照物的那个 harness，此前一直没有独立页面。本文是**实体/概念页**：它是什么、规则文件怎么被发现、skill/agent 怎么摆、哪些命令和原语被本库笔记实际引用过，以及它与 opencode / DSH 在本库既有笔记所划定的坐标轴上的差别。
> **证据分级**：本页混合了两类来源——(a) 本库已有笔记（以 wikilink 指认）；(b) 2026-10-09 亲自抓取的 Anthropic 官方文档（`code.claude.com/docs/`，见 frontmatter `source` 与文末参考）。凡这两类都支撑不到的陈述，一律标 `（待核实）`。

## 它是什么

Claude Code 是 Anthropic 官方的**终端编码 agent（CLI harness）**。本库的定位锚点有三条：

- 它是 AI 编程工具"第三代（Agent 自主循环）"三件套之一，与 Codex、opencode 并列——第一代是自动补全（Copilot 早期），第二代是对话式（ChatGPT）（[[03.Engineering/Common_Area/AI/Loop-Engineering|Loop Engineering]]）。
- 它是本库多篇笔记的**默认对照组**：opencode 的缺口矩阵以它做基准（[[03.Engineering/Common_Area/AI/Loop-Engineering-Opencode-Research|Loop Engineering with opencode]] §6），OpenCode v2 架构深读以它做"假设形状"的对照（§「对照 Claude Code：两种'假设'的形状」）。
- 它同时是一个**产品/生态节点**：Matt Pocock 的 skills 以它的官方插件为主要分发方式（`claude plugins install mattpocock-skills`，托管只读、自动更新），本机 `~/.claude/skills/` 与 `~/.agents/skills/` 因此保持同步（[[03.Engineering/Common_Area/AI/matt-pocock-skills|Matt Pocock Skills]]、[[03.Engineering/Common_Area/AI/matt-pocock-skills-installation-notes|安装记录]]）。

## 规则文件发现：`CLAUDE.md`

这是它与本库实践（`AGENTS.md`）差异最实质的一处。官方 memory 文档给出的**作用域一览**（由宽到窄，宽者先载入）：

| 作用域 | 位置 | 用途 |
|---|---|---|
| Managed policy | macOS `/Library/Application Support/ClaudeCode/CLAUDE.md`；Linux/WSL `/etc/claude-code/CLAUDE.md`；Windows `C:\Program Files\ClaudeCode\CLAUDE.md` | 组织级指令 |
| User | `~/.claude/CLAUDE.md` | 跨项目的个人偏好 |
| Project | `./CLAUDE.md` 或 `./.claude/CLAUDE.md` | 团队共享的项目指令 |
| Local | `./CLAUDE.local.md` | 个人项目偏好，建议 gitignore |

发现与载入机制（官方 memory 文档）：

- **逐级向上 + 拼接**：载入当前工作目录及其**所有父目录**的 `CLAUDE.md` / `CLAUDE.local.md`，全部**拼接**进上下文而非互相覆盖；跨目录时按"文件系统根 → 工作目录"排序，同一目录内 `CLAUDE.local.md` 追加在 `CLAUDE.md` 之后。
- **子目录按需**：工作目录**之下**的子目录 `CLAUDE.md` 不在启动时载入，而是 Claude 首次读/写/编辑该子目录中某个文件时才载入。
- **`@path` 导入**：`CLAUDE.md` 可用 `@path/to/file` 导入其它文件（相对路径以**含导入语句的文件**为基准），可递归，**最大深度 4 跳**；项目级文件导入工作目录之外的路径属"外部导入"，首次会遇到审批弹窗。
- **体量建议**：官方建议单个 `CLAUDE.md` **200 行以内**（超 4 MiB 直接跳过）；更长的规则应拆到 `.claude/rules/`（可带 `paths` frontmatter 做路径域限定）或 skill。
- **与 `AGENTS.md` 的关系**：默认只有工作目录及其父目录中**不存在** `CLAUDE.md` / `.claude/CLAUDE.md` / `CLAUDE.local.md` 时，才读 `AGENTS.md`（v2.1.277+；`~/.claude/CLAUDE.md` 与 managed `CLAUDE.md` **不参与**这个判定）；可用 `/config` 的 **Project instructions** 改为 `claude-md-and-agents-md` 等四种取值。
- **另一套记忆系统：Auto memory**——由 Claude 自己写的笔记，存于 `~/.claude/projects/<project>/memory/`（`MEMORY.md` 作索引 + 每主题一个文件），每次会话载入 `MEMORY.md` 的前 200 行或前 25KB。这条官方机制对应本库 [[03.Engineering/Common_Area/AI/Snippets|Snippets]] 里"Memories 分为手动编写的 CLAUDE.md 和 Auto Memory"的说法。
- 相关命令：`/init` 生成起始 `CLAUDE.md`；`/memory` 浏览与编辑；`/context` 查看 **Memory files** 列表确认哪些文件真正载入了。

## 与 opencode 的 `CLAUDE.md` 兼容路径（本库 V1 实测结论）

本库 [[03.Engineering/Common_Area/AI/AI Agent 规则文件体系|AI Agent 规则文件体系]] 记录的是**反方向**的兼容：

- opencode **V1（1.18.9）源码实测**：全局 `~/.config/opencode/AGENTS.md` 不存在时，回退读 `~/.claude/CLAUDE.md`（二选一、不叠加）；项目级按 `AGENTS.md → CLAUDE.md → CONTEXT.md` 优先级，"第一种有匹配的类型生效"。即 `CLAUDE.md` 曾是 opencode 的第一兼容回退层；兼容模式可用 `OPENCODE_DISABLE_CLAUDE_CODE*` 环境变量关闭。
- **该回退链已在 opencode V2 移除**：V2 官方说明保留 `AGENTS.md`，不再提供 `CLAUDE.md` 回退（同页 2026-09-25 补充，详见 [[03.Engineering/Common_Area/Linux/opencode-v2-migration|opencode V1→V2 迁移记录]]）。`CONTEXT.md` 也已在官方规格中标注 deprecated。
- **易踩的坑**：某层放 `AGENTS.md`、另一层放 `CLAUDE.md`，会因"第一种类型赢"导致父/子层某些文件被跳过；本库结论是全链路统一用 `AGENTS.md`。

## Skill 与 agent 布局

官方文档确认的 skills 加载位置（即"personal / project / nested / plugin / enterprise"多层）：

| 位置 | 路径 | 生效范围 |
|---|---|---|
| Personal | `~/.claude/skills/<name>/SKILL.md` | 本机所有项目 |
| Project | `.claude/skills/<name>/SKILL.md` | 本仓库会话（提交后可共享给团队） |
| Nested | `<subdir>/.claude/skills/<name>/SKILL.md` | 在该子目录启动的会话；或 Claude 首次读该子目录文件时载入 |
| Plugin | `<plugin>/skills/<name>/SKILL.md` | 插件启用的地方，以 `/plugin-name:skill-name` 调用 |

- **skills 是 Agent Skills 开放标准**（`SKILL.md` + frontmatter），Claude Code 在其上扩展了调用控制（`disable-model-invocation`）、子代理执行（`context: fork` + `agent:`）、动态上下文注入（`` !`cmd` ``）等；**旧的 `.claude/commands/*.md` 自定义命令已并入 skills**，`.claude/skills/deploy/SKILL.md` 与 `.claude/commands/deploy.md` 都产生 `/deploy`。
- **子代理**：自定义 subagent 放 **`.claude/agents/`**（个人级为 `~/.claude/agents/`），以 frontmatter + markdown body 定义——这正是本库 Loop 研究引用的"`.claude/agents/`、agent teams"（[[03.Engineering/Common_Area/AI/Loop-Engineering-Opencode-Research|Loop Engineering with opencode]] §1.3 对照表）。
- **`.agents/skills/` 的共享约定**：本库安装记录与技能方法论笔记把 `~/.agents/skills/`（机器级）与项目内 `.agents/skills/` 当作**opencode + Claude Code 共享**的技能目录，并把 `.claude/skills/` 标为"Claude Code 专属覆盖"（[[03.Engineering/Common_Area/AI/matt-pocock-skills-installation-notes|安装记录]]；[[03.Engineering/Common_Area/AI/matt-pocock-skills|Matt Pocock Skills]] 的"各目录职责"表）。**注**：本次抓取的官方 memory / worktrees / skills 文档中未见 `.agents/skills/` 作为 Claude Code 的读取路径，故这条仅作为本库实践记录保留，不作为官方事实（待核实）。
- 本机现状：`~/.agents/skills/` 与 `~/.claude/skills/` 同步装了 Matt Pocock skills v1.2.3 的 **19 项**仓库技能（另 `find-skills` 为仓库外技能，合计 20）；`~/.config/opencode/skills/` 是 opencode 侧的**子集**（grill 族 + tdd/implement/to-spec/to-tickets/code-review/domain-modeling/teach）。

## 本库笔记实际引用过的命令与原语

下表只收本库笔记**已经引用过**的项，并用这次抓取的官方文档核对；未核对到的单独列出。

| 命令/原语 | 本库出处 | 官方文档核对结果（2026-10-09） |
|---|---|---|
| `/loop [interval] [prompt]` | 缺口矩阵的主条目 | ✅ bundled **skill**：按间隔重复跑同一 prompt；省略间隔则由 Claude 自行决定节奏（self-paced）；别名 `/proactive` |
| `/goal <条件>` | 缺口矩阵 + Loop Engineering §子 Agent | ✅ 设置完成条件，**每轮结束由一个模型判定**是否达成 / 是否不可能；一个会话同时只能有一个 goal；本质是 session 级的 prompt-based Stop hook |
| `/compact` | [[03.Engineering/Common_Area/AI/Matt-Pocock-AI-工作流详解\|Matt Pocock AI 工作流详解]]（会话过大逼近聪明区 → `/compact` 为默认动作） | ✅ 内置命令，"摘要对话以腾出上下文"；**不能**通过 Skill 工具调用（对比 `/loop`、`/code-review` 是 skill） |
| `/handoff` | [[03.Engineering/Common_Area/AI/matt-pocock-skills|Matt Pocock Skills]] 第 80 行 | ⚠️ 这是 **Matt Pocock 的 skill**（"把当前会话压成交接文档给另一 agent"），**不是** Claude Code 内置命令 |
| `isolation: worktree` | 缺口矩阵 + cobus 框架作者注释 | ✅ subagent frontmatter 字段：在 `.claude/agents/` 中给某 subagent 写 `isolation: worktree`，它每次都在自己的临时 worktree 里跑 |
| `--worktree` / `-w <name>` | 对照表 Worktrees 行 | ✅ CLI flag：默认在 `.claude/worktrees/<name>/` 建 worktree、分支名 `worktree-<name>`；会话隔离期间四类检查（文件编辑、命令工作目录、git 重定向、命令形态）会拦截越界调用 |
| `/batch <instruction>` | 未见于本库 | ✅ bundled skill：把大改动拆成 5–30 个独立单元，每个单元在**独立 worktree** 里由后台 subagent 实现 |
| `/schedule` | 未见于本库 | ✅ 管理 cloud **routines**（在云端执行，独立于任何打开的会话），别名 `/routines` |
| `/code-review`（别名 `/review`） | 作为 Matt skills 同名能力的对照面 | ✅ bundled skill：审当前 diff 或指定 PR/分支/路径的**正确性 bug**，`--fix` 可直接应用；`ultra` 走云端多智能体评审 |
| `/clear` `/resume` `/branch` `/rewind` `/diff` `/context` `/doctor` | 未见于本库 | ✅ 均为官方命令表中的内置命令（本行仅作索引，非本库结论） |
| Hooks（事件驱动触发） | 对照表 Automations 行 | ✅ 官方有 hooks 体系（含 `InstructionsLoaded`、`WorktreeCreate`/`WorktreeRemove`、prompt-based Stop hook 等）；`/goal` 即建在其上 |
| `/loop --schedule` | [[03.Engineering/Common_Area/AI/Loop-Engineering|Loop Engineering]] 第 49 行 | ⚠️ 与官方 `/loop [interval] [prompt]` 形态不符（待核实） |

> **本库的诚实声明依然有效**：Loop-Engineering-Opencode-Research 记录当时 `code.claude.com` 境内直连多次超时，第 6 节的 Claude Code 侧描述来自两条**间接一手证据**（Addy Osmani 原文对照表、opencode issue #18001 评论的转述）；该限制不影响本页——本页相关条目已于 2026-10-09 直连官方文档逐条核对。间接证据中未被本次核对覆盖的只有 `--worktree` flag 的原始出处，现已由官方 worktrees 文档确认。

## 与 opencode / DSH 的对照（沿用本库既有坐标轴）

**轴 1：假设的形状**（[[03.Engineering/Common_Area/AI/OpenCode-v2-架构深读|OpenCode v2 架构深读]]）

| 维度 | Claude Code | OpenCode v2 |
|---|---|---|
| 假设形状 | **prompt 形状**：subagent / planner / evaluator，靠编排多个模型实例互相制衡 | **存储形状**：日志 / 快照 / 基线 / 外存，靠数据结构保证一致性 |
| 不确定性处理 | 交给"另一个模型的判断" | 消灭在"数据库的事务语义"里 |

**轴 2：Loop 工程五件套**（[[03.Engineering/Common_Area/AI/Loop-Engineering-Opencode-Research|Loop Engineering with opencode]] §1.3 原文对照表）

| 原语 | Claude Code |
|---|---|
| Automations | Scheduled tasks and cron、`/loop`、`/goal`、hooks、GitHub Actions |
| Worktrees | `git worktree`、`--worktree`、`isolation: worktree` |
| Skills | Agent Skills（`SKILL.md`） |
| Plugins/Connectors | MCP servers + plugins |
| Sub-agents | `.claude/agents/`、agent teams |
| State | Markdown（`AGENTS.md`、progress files）/ Linear via MCP |

**轴 3：仓库载体与"关注点分裂"**（[[03.Engineering/Common_Area/AI/Frontier-Engineering|Frontier Engineering]] 原则 03）：Kiro 的 steering 文件是"约定与编码标准"的载体；本库的落法是**一套 `AGENTS.md` + 一份 skill 定义，多 harness 共用**，而 Claude Code 世界对应的文件是 `CLAUDE.md` + `.claude/skills/`。同一段工程规范要在两个世界各存一份，是本库采用"`.agents/skills/` 作共享源 + `CLAUDE.md` 供 Copilot 读自然语言版"这一配置的动因（[[03.Engineering/Common_Area/AI/matt-pocock-skills|Matt Pocock Skills]]「双目录协作」节）。

**轴 4：生态位置**（[[03.Engineering/Common_Area/AI/copilot vs opencode|copilot vs opencode]]、[[03.Engineering/Common_Area/AI/Spec-Driven Development|Spec-Driven Development]]）

- Spec-Driven Development 类工具（如 OpenSpec）宣称无绑定、支持 25+ AI 工具含 Claude Code，但**语法有分歧**：短横线式 `/opsx-propose` 是 Copilot 语法，**冒号式 `/opsx:propose` 才是 Claude Code 语法**——这解释了为什么 Claude Code 的插件 skill 以 `/plugin-name:skill-name` 命名。
- 跨工具可迁移性（[[03.Engineering/Common_Area/AI/Snippets|Snippets]]）：`CLAUDE.md` 的内容可以改格式搬到 Cursor 复用；MCP 是跨平台标准，同一台 MCP server 可被 Cursor / Windsurf / Claude Code 同时调用。**唯一不好迁移的是 Memories**（本地状态）。
- DSH 侧：本库记录 DSH 的 `skill` 体系同样是**全局技能目录**（本机为 `~/.config/opencode/skills/` 与 `~/.claude/skills/`，非 repo 内路径）按需注入（[[03.Engineering/Common_Area/AI/DeepSeek Harness 最佳实践|DeepSeek Harness 最佳实践]]）；DSH 的 `goal` / `workflow` 与 Claude Code 的 `/goal` / subagent+worktree 属"同一问题的不同落法"（待核实——本页未逐条核对 DSH 的原语语义）。

**不在上述坐标轴上的差异**：opencode 侧"**缺内建定时自动化**"是本库 Loop 研究反复强调的结构性差异（其 `/loop`、`/schedule`、`/goal` 请求截至 2026-09-07 全部 open 未合并），而 Claude Code 这三个原语都是内建的——这是本库笔记里唯一一处"Claude Code 有、opencode 缺"的明确清单。

## 本库把 Claude Code 当对照物时的注意点

1. **不要当成"更强"的同义词**：本库两份笔记的结论都是"两条路线没有高下"（架构深读）与"内建 `/loop` 缺失是暂时性差距，不是结构性障碍"（Loop 研究）。
2. **命令归属要看清**：`/handoff` 是 Matt Pocock skill 而非内置命令；`/loop`、`/code-review` 是 bundled skill；`/compact`、`/clear`、`/context` 是 built-in 命令；`/schedule` 管的是**云端 routines**（不等于"本机定时任务"）。这几类在官方命令表里用 **Skill** / **Workflow** 标注区分，混用会导致对照表失真。
3. **规则文件不要双写**：opencode V2 已移除 `CLAUDE.md` 回退，本库结论是统一 `AGENTS.md`；只有在需要给 Claude Code / Copilot 供文件时，才用 `CLAUDE.md`（可用 `@AGENTS.md` 导入或 symlink 共享一份内容）。
4. **本页未覆盖**：权限模型（`/permissions`、auto mode）、`settings.json`、MCP 接入、插件市场、云会话（`/teleport`、`/schedule` 的 routine 语义）、Agent SDK——本库笔记均无相关内容，需要时再单独取证。

## 相关

- [[03.Engineering/Common_Area/AI/OpenCode-v2-架构深读|OpenCode v2 架构深读]] — "两种假设的形状"对照表的来源
- [[03.Engineering/Common_Area/AI/Loop-Engineering-Opencode-Research|Loop Engineering with opencode]] — §6 缺口矩阵、§1.3 五件套对照表、访问限制声明
- [[03.Engineering/Common_Area/AI/Loop-Engineering|Loop Engineering]] — 三代演进、`/goal` 的独立判停模型
- [[03.Engineering/Common_Area/AI/AI Agent 规则文件体系|AI Agent 规则文件体系]] — `CLAUDE.md` 在 opencode V1 的回退链与 V2 的移除
- [[03.Engineering/Common_Area/AI/matt-pocock-skills|Matt Pocock Skills]] — 官方插件分发、三环境读取链路、`.agents/skills/` 共享方案
- [[03.Engineering/Common_Area/AI/matt-pocock-skills-installation-notes|Matt Pocock Skills 安装记录]] — 本机 `~/.claude/skills/` 与 `~/.agents/skills/` 的实际状态
- [[03.Engineering/Common_Area/AI/Matt-Pocock-AI-工作流详解|Matt Pocock AI 工作流详解]] — `/compact` 与 `/handoff` 的定位
- [[03.Engineering/Common_Area/AI/Snippets|Snippets]] — 五层概念里 Claude Code 的 Rules/Memories/Skill 映射
- [[03.Engineering/Common_Area/AI/Spec-Driven Development|Spec-Driven Development]] — `/opsx:propose` 冒号语法归属
- [[03.Engineering/Common_Area/AI/Frontier-Engineering|Frontier Engineering]] — steering 文件 / skills / MCP 的"为 Agent 构建"视角
- [[03.Engineering/Common_Area/AI/DeepSeek Harness 最佳实践|DeepSeek Harness 最佳实践]] — DSH 的全局技能目录与原语对照
- [[03.Engineering/Common_Area/AI/copilot vs opencode|copilot vs opencode]] — 生态定位坐标
- [[03.Engineering/Common_Area/AI/AI时代的程序员|AI 时代的程序员]] — 工具演进下"人负责什么"的早期论点
- [[03.Engineering/Common_Area/Linux/opencode-v2-migration|opencode V1→V2 迁移记录]] — `CLAUDE.md` 回退移除的原始出处

## 参考（本页亲自抓取，2026-10-09）

- Claude Code 官方文档 · Memory（`CLAUDE.md` 作用域、载入机制、`@` 导入、Auto memory）: <https://code.claude.com/docs/en/memory>
- · Skills（`~/.claude/skills/` 与 `.claude/skills/`、Agent Skills 标准、bundled skills）: <https://code.claude.com/docs/en/skills>
- · Run parallel sessions with worktrees（`--worktree`、`isolation: worktree`、隔离检查）: <https://code.claude.com/docs/en/worktrees>
- · Commands（全命令表，标注 built-in / Skill / Workflow）: <https://code.claude.com/docs/en/commands>
- · Keep Claude working toward a goal（`/goal` 的判停模型与语义）: <https://code.claude.com/docs/en/goal>
- · Run prompts on a schedule（`/loop` 与调度选型对比）: <https://code.claude.com/docs/en/scheduled-tasks>
