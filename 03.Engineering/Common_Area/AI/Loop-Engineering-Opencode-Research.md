---
created: 2026-09-07
updated: 2026-09-07
tags:
  - type/note
  - engineering/ai
  - loop-engineering
  - opencode
status: active
source: https://addyosmani.com/blog/loop-engineering/
---

# Loop Engineering with opencode — 一手来源研究报告

> 用开源 AI 编程 agent **opencode** 实践 "Loop Engineering"（循环工程）：概念、官方能力、缺口、社区框架、对照与实操配方。

## 研究日期与范围

- **研究日期**：2026-09-07。所有网页均于当日抓取。
- **opencode 版本基准**：文档源为 `anomalyco/opencode` 的 `dev` 分支（2026-09-07 当日状态）；当日最新正式 release 为 **v1.18.29**（2026-09-04 发布，[GitHub Releases API](https://api.github.com/repos/anomalyco/opencode/releases/latest)）。
- **来源类型**：全部为一手来源——作者原文（Addy Osmani）、opencode 官方文档源（mdx，与 opencode.ai/docs 同源）、官方 GitHub 仓库/issue/PR（经 GitHub API 实时确认状态）、官方 GitHub Actions workflow、开源框架仓库原始文件。
- **访问限制说明（诚实声明）**：Anthropic 官方文档站 `code.claude.com` 在本机（境内网络）多次直连超时（≥3 次尝试均失败），故第 6 节中 Claude Code 侧功能描述**未能逐字核对** Anthropic 原文，采用两条**间接一手证据**：① Addy Osmani 原文中的对照表格；② opencode issue #18001 评论中对官方文档 URL（`code.claude.com/docs/en/scheduled-tasks#run-a-prompt-repeatedly-with-/loop`）及其功能要点的转述。凡来自转述处均明确标注。
- 报告内每条主张后以括号标注来源 URL；分析性结论标注"（推断：…）"以区别于事实。

---

## TL;DR（要点）

1. **概念权威**：Loop Engineering 由 Google 工程师 Addy Osmani 定义——"循环工程是取代"你作为给 agent 写 prompt 的人"：你设计的是那个替你写 prompt 的系统"；一个 Loop 由 **Automations（自动化）+ Worktrees + Skills + Plugins/Connectors + Sub-agents 五件套**组成，外加第 6 件事：**落在磁盘上的状态记忆**（"agent 会忘，仓库不会"）（https://addyosmani.com/blog/loop-engineering/）。
2. **opencode 能力盘点**：五件套中 opencode **原生支持四件半**——headless `run`（脚本/自动化友好）、session 延续（`--continue`/`--session`）、primary/subagent + 权限系统（含 Maker-Checker 场景）、`SKILL.md` skills、MCP、plugins（事件钩子）、`serve` HTTP/OpenAPI 服务、GitHub Actions 集成（含 `schedule` cron 事件）。
3. **唯一的官方缺口是"内建定时自动化"**（在 TUI 里 `/loop`、`/goal`、`/schedule` 之类的原生循环/调度原语）。证据链：功能请求 #11232（已因 60 天无活动被机器人自动关闭）、#41906（`/schedule`，2026-08-12，**被创始人 jlongster 认领**）、#41907（`/loop`）、#18001（`/loop`，43 👍）、实现 PR **#44191**（cron tools，2026-08-22 提交）——**全部 open 未合并**，官方文档中不存在这些命令（https://github.com/anomalyco/opencode/issues/44191、/41906、/41907、/18001、/11232）。
4. **官方给出的"外部调度"路径**：CLI 文档明确 `run` "useful for scripting, automation"；GitHub 集成文档提供 **`schedule`（cron）事件**的官方示例 workflow；`serve` 暴露 OpenAPI 3.1 供程序化驱动（https://raw.githubusercontent.com/anomalyco/opencode/dev/packages/web/src/content/docs/{cli,github,server}.mdx）。
5. **社区框架**：`cobusgreyling/loop-engineering`（MIT）是目前**对 opencode 支持最完整的 Loop 工程框架**——直接用 cron/systemd 每个 tick 调 `opencode run`，配合 `STATE.md` 状态 + `loop-triage` skill + implementer/verifier 子代理分离 + git worktree 隔离，并提供 L1（报告-only）→ L2（辅助修复）→ L3（无人值守）渐进路线（https://raw.githubusercontent.com/cobusgreyling/loop-engineering/main/examples/opencode/daily-triage.md）。
6. **opencode 自带防失控原语**：`doom_loop` 权限（同一工具调用重复 3 次即触发询问，默认 `ask`）、agent `steps` 步数上限、细粒度 `allow/ask/deny` 权限矩阵（`--auto` 也绝不覆盖显式 `deny`）——这是社区框架"风险刹车"主张之外的官方地基（https://raw.githubusercontent.com/anomalyco/opencode/dev/packages/web/src/content/docs/{agents,permissions}.mdx）。
7. **对照 Claude Code 的差距与补位**：opencode 缺 Claude Code 的 `/loop`（节奏性重复）、`/goal`（跑到验证条件成立为止）、desktop Automations 面板；可用 cron 包装 `opencode run`（固定节奏）、外部循环 + verifier 子代理（run-until-done）、GitHub Actions schedule（桌面级 automations）三层补位。
8. **结论倾向**（推断：综合上述一手证据）："Loop Engineering" 的核心是**把调度、记忆、验证搬到 agent 之外的普通文件与普通进程**——恰好是 headless 优先的 opencode + 社区 cron/systemd 模式的主场；内建 `/loop` 类原语缺失是暂时性差距，不是结构性障碍。

---

## 1. 概念权威定义（Addy Osmani 原文）

来源：https://addyosmani.com/blog/loop-engineering/（2026-09-07 抓取全文，以下引语均出自该页）。

### 1.1 一句话定义

> "Loop engineering is replacing yourself as the person who prompts the agent. You design the system that does it instead."（循环工程就是：取代"你作为那个给 agent 写 prompt 的人"——你设计的是替你写 prompt 的那套系统。）
> "A loop here can be thought of a recursive goal where you define a purpose and the AI iterates until complete."（一个 loop 可被想成一个递归目标：你定义目的，AI 迭代直到完成。）

背景引语（原文引用）：
- Peter Steinberger： "You shouldn't be prompting coding agents anymore. You should be designing loops that prompt your agents."
- Boris Cherny（Anthropic Claude Code 负责人）： "I don't prompt Claude anymore. I have loops running that prompt Claude and figuring out what to do. My job is to write loops".

### 1.2 五件套 + 第 6 件事（原文）

> "A loop needs five things and then one place to remember stuff."

| # | 原语 | Loop 中的职责 |
|---|------|--------------|
| 1 | **Automations** | 按计划自动触发，自行做 discovery + triage |
| 2 | **Worktrees** | 让并行工作的多个 agent 互不踩踏（"One agent's edits literally can not touch the other one's checkout"） |
| 3 | **Skills** | 把 agent 原本只能靠猜的项目知识写下来（SKILL.md） |
| 4 | **Plugins and connectors** | 把 agent 接进你已在用的工具（MCP 之上的连接器） |
| 5 | **Sub-agents** | 一个 agent 出主意，**另一个**去检查它（maker-checker 分离："the model that wrote the code is way too nice grading its own homework"） |

第 6 件事——记忆/状态：

> "The sixth thing, the memory. A markdown file, or a Linear board, anything that lives outside the single conversation and holds what's done and what is next. … the model forgets everything between runs so the memory has to be on disk and not in the context. **The agent forgets, the repo doesn't.**"

### 1.3 原文中的对照表（注意：表内只有 Codex 与 Claude Code 两列，没有 opencode）

| Primitive | Codex app | Claude Code |
|-----------|-----------|-------------|
| Automations | Automations tab；`/goal` run-until-done | Scheduled tasks and cron、`/loop`、`/goal`、hooks、GitHub Actions |
| Worktrees | 内建 worktree per thread | `git worktree`、`--worktree`、`isolation: worktree` |
| Skills | Agent Skills（SKILL.md） | Agent Skills（SKILL.md） |
| Plugins/Connectors | Connectors(MCP) + plugins | MCP servers + plugins |
| Sub-agents | `.codex/agents/`（TOML） | `.claude/agents/`、agent teams |
| State | Markdown / Linear | Markdown（AGENTS.md、progress files）/ Linear via MCP |

（此表同时说明：本文所研究的 opencode 属于"形状相同但名单未列"的同类工具。）

### 1.4 "一个 loop 长什么样"（原文所描绘的目标形态）

> "An automation runs every morning on the repo. Its prompt calls a triage skill that reads yesterday's CI failures, the open issues, the recent commits, and writes the findings into a markdown file or a Linear board. For each finding that is worth doing the thread opens an isolated worktree and sends a sub-agent to draft the fix, and a second sub-agent reviews that draft against the project skills and the existing tests. Connectors let the loop open the PR and update the ticket. Anything the loop can not handle lands in the triage inbox for me. The state file is the spine of the whole thing…"

### 1.5 风险与警告（原文原话要点）

原文在 "What the loop still does not do for you" 一节给出了三个会随 loop 变强而**更尖锐**的问题：

1. **验证仍在你身上**："A loop running unattended is also a loop making mistakes unattended. … 'done' is a claim and not a proof."
2. **理解腐化**（comprehension debt）："The faster the loop ships code you did not write, the bigger the gap between what exists and what you actually get."
3. **认知投降**（cognitive surrender）："When the loop runs itself its very tempting to stop having an opinion and just take whatever it gives back."
4. **Token 成本**（在引言中即警告）："you absolutely have to be careful about token costs (usage patterns can vary wildly if you are token rich or poor)."
5. 结尾立场："**Build the loop. But build it like someone who intends to stay the engineer**, not just the person who presses go."（此外原文还强调：直接 prompt 也仍然有效，"It's all about finding the right balance"。）

---

## 2. opencode 官方能力盘点（2026-09-07，dev 分支文档）

文档均抓自 https://raw.githubusercontent.com/anomalyco/opencode/dev/packages/web/src/content/docs/*.mdx（与 opencode.ai/docs 同源）。

### 2.1 Headless 非交互 `run`（loop 的地基）

来源：docs/cli.mdx。官方对 `run` 的定位原话：**"Run opencode in non-interactive mode by passing a prompt directly. This is useful for scripting, automation…"**

关键参数：
- `--continue` / `-c`、`--session` / `-s`：延续上次会话（跨 tick 记忆的会话层手段）
- `--fork`：fork 会话
- `--agent`：指定 agent（配合自定义 primary/subagent 完成 maker/checker 分工）
- `--model` / `-m`：覆盖模型
- `--file` / `-f`：把文件附加进消息
- `--attach`：附着到一个运行中的 `opencode serve` 实例（"avoid MCP server cold boot times on every run"）
- `--auto`：自动批准未显式 deny 的权限请求
- `--format json`：输出原始 JSON 事件流（供外部程序解析）
- `--title`、`--share`、`--thinking` 等

会话管理命令：`session list/delete`、`export [sessionID]`（导出 JSON）、`import`、`stats`（token 用量与成本统计，支持 `--days/--models/--project`——loop 预算监控工具）。

> 注意：CLI 全量命令表（tui/agent/attach/auth/github/mcp/models/run/serve/session/stats/export/import/web/acp/plugin/pr/db/debug）中**不存在** `/loop`、`/goal`、`/schedule` 或任何 cron 子命令——与第 3 节的缺口结论一致。

### 2.2 Session 延续机制

- TUI/CLI：`opencode [project]` 的 `-c/--continue` 与 `-s/--session`（docs/cli.mdx）。
- HTTP API：`POST /session`（创建，可带 `parentID`）、`GET /session/:id/children`（**子会话**）、`PATCH`、`/session/:id/fork`、`/session/:id/summarize`、`/session/:id/diff`、`POST /session/:id/prompt_async`（异步发消息）等（docs/server.mdx）。
- 会话树导航键（TUI）：子会话之间用 `session_child_cycle`/`session_parent` 等（docs/agents.mdx）。
- **自动压缩**：长上下文超过阈值会触发隐藏的 `compaction` 系统 agent 自动总结（docs/agents.mdx）；可用 `OPENCODE_DISABLE_AUTOCOMPACT` 关闭，plugins 可用 `experimental.session.compacting` 钩子定制压缩内容（docs/plugins.mdx）——与 Addy "模型会忘，记忆放磁盘"的原则呼应：上下文会被压缩，状态必须外置。

### 2.3 Agent 系统与 Maker–Checker

来源：docs/agents.mdx。

- **两类 agent**：primary（直接对话，Tab 切换）与 subagent（主 agent 经 Task 工具调用，或用户 `@名字` 手动调）。
- **内置**：primary `build`（全权限）、primary `plan`（只读分析，"without making any actual modifications"）；subagent `general`（可并行多任务）、`explore`（只读探索）、`scout`（只读外部依赖研究）。
- **Maker–Checker 支持方式**：定义任意自定义 agent（JSON 或 Markdown frontmatter），各自可配不同 `model`、`prompt`、`permission`（如 reviewer 配 `edit: deny`）→ 制造者与检查者可用不同模型不同权限；`permission.task` 用 glob 控制某 agent 能派哪些子 agent。
- **自定义 agent 示例**（文档原样）：

```markdown
title="~/.config/opencode/agents/review.md"
---
description: Reviews code for quality and best practices
mode: subagent
model: anthropic/claude-sonnet-4-20250514
temperature: 0.1
permission:
  edit: deny
  bash: deny
---
You are in code review mode. Focus on:
- Code quality and best practices
- Potential bugs and edge cases
...
```

- 命名约定：Markdown 文件名即 agent 名（`review.md` → `review`）。
- **token 成本刹车（官方原生）**：agent 配置里的 `steps` 字段——"Control the maximum number of agentic iterations an agent can perform before being forced to respond with text only. This allows users who wish to control costs to set a limit on agentic actions." 未设置则 agent "will continue to iterate until the model chooses to stop or the user interrupts the session"（⚠️ 文档标注旧字段 `maxSteps` 已弃用，新字段为 `steps`）。

### 2.4 Skills（SKILL.md）

来源：docs/skills.mdx。

- 目录约定：`.opencode/skills/<name>/SKILL.md`、`~/.config/opencode/skills/`，另兼容 `.claude/skills/` 与 `.agents/skills/`（Claude Code / agent 兼容共享）。
- 发现机制：从当前目录向上走到 git worktree 根，沿途加载所有匹配目录；**按需经原生 `skill` 工具加载**（不是全部塞进上下文）。
- 格式：frontmatter 只认 `name`（必须小写 + 连字符）、`description`（1–1024 字符）、可选 `license`/`compatibility`/`metadata`。
- 权限门控：`permission.skill` 支持通配符 `allow/deny/ask`（如 `"internal-*": "deny"`）。
- 文档实例（原样截取结构）：`git-release` skill——"Draft release notes from merged PRs…"。
- **Loop 意义**（推断：与 Addy 的 "skills" 原语一一对应）：把"每次循环都要重新解释一遍的项目知识"写进磁盘上的 SKILL.md，让每次 tick 的冷启动 agent 按需读它。

### 2.5 MCP、Plugins 与事件钩子

来源：docs/plugins.mdx、docs/cli.mdx（`mcp add/list/auth`）。

- MCP：`opencode mcp add/list/auth/logout/debug` 管理；server 有 `/mcp` 状态接口与 `POST /mcp` 动态添加。
- Plugins：本地 `.opencode/plugins/*.{js,ts}`（自动加载）或 npm 包（`plugin` 数组）；**事件钩子清单**与 loop 直接相关的有：
  - 会话类：`session.created`、`session.idle`、`session.compacted`、`session.diff`、`session.error`、`session.status`、`session.updated`
  - 工具类：`tool.execute.before` / `tool.execute.after`（可做审计/拦载）
  - 消息类、文件类、LSP、权限、TUI（`tui.command.execute` 等）
- 插件回调里可直接执行 shell（文档示例：`session.idle` 时 `osascript` 发系统通知）→ **这就是在进程内实现"tick 后通知"等 loop 行为的官方挂点**（推断）。

### 2.6 HTTP Server API（外部编排入口）

来源：docs/server.mdx。

> "The `opencode serve` command runs a headless HTTP server that exposes an **OpenAPI** endpoint… Use the opencode server to interact with opencode **programmatically**."

- `opencode serve`（默认 127.0.0.1:4096），`OPENCODE_SERVER_PASSWORD`/`OPENCODE_SERVER_USERNAME` 做 basic auth。
- OpenAPI 3.1 spec 端点：`http://localhost:4096/doc`；官方 SDK 即由该 spec 生成。
- 与 loop 相关的 API 面（摘要）：
  - Session：`POST /session`、`GET /session/:id/children`、`POST /session/:id/fork`、`POST /session/:id/abort`、`POST /session/:id/summarize`、`POST /session/:id/diff`、`/session/:id/command`（执行 slash command）、`/session/:id/shell`
  - 消息：`POST /session/:id/message`、`POST /session/:id/prompt_async`
  - 事件流：`GET /event`（SSE，首个事件 `server.connected`，其后总线事件）
- `opencode run --attach http://localhost:4096` 可附着既有 server（避免每次 run 的 MCP 冷启动）。

### 2.7 GitHub 集成（含官方 `schedule` cron 事件——这是目前文档层唯一的"官方定时"）

来源：docs/github.mdx、仓库 `opencode github install`。

- 触发面：在 issue/PR 评论中提及 `/opencode` 或 `/oc`；支持的事件类型表：`issue_comment`、`pull_request_review_comment`、`issues`、`pull_request`、**`schedule`（"Cron-based schedule… Output goes to logs and PRs"）、`workflow_dispatch`**。
- **官方 Schedule 示例**（docs/github.mdx 原样，含注释）——这是"定时自动化"在 opencode 生态里的官方样板：

```yaml
title=".github/workflows/opencode-scheduled.yml"
name: Scheduled OpenCode Task
on:
  schedule:
    - cron: "0 9 * * 1" # Every Monday at 9am UTC
jobs:
  opencode:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: write
      pull-requests: write
      issues: write
    steps:
      - name: Checkout repository
        uses: actions/checkout@v6
        with:
          persist-credentials: false
      - name: Run OpenCode
        uses: anomalyco/opencode/github@latest
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        with:
          model: anthropic/claude-sonnet-4-20250514
          prompt: |
            Review the codebase for any TODO comments and create a summary.
            If you find issues worth addressing, open an issue to track them.
```

- 官方注意事项："For scheduled events, the `prompt` input is **required** since there's no comment to extract instructions from. **Scheduled workflows run without a user context to permission-check**, so the workflow must grant `contents: write` and `pull-requests: write` if you expect OpenCode to create branches or PRs."（→ 无人值守 = 权限必须显式给足，这与社区框架的 L1 只读起步哲学形成对照。）

---

## 3. 定时自动化：官方现状与缺口（issue/PR 实证）

所有状态均于 2026-09-07 经 GitHub API 实时核实（https://api.github.com/repos/anomalyco/opencode/issues/{n}）。

### 3.1 文档层证据：无内建命令

- 官方 CLI 文档全命令列表（docs/cli.mdx）无 `loop/schedule/goal`；docs/github.mdx 的 `schedule` 是 GitHub Actions 事件而非 TUI/CLI 内建能力。
- 因此可确认（事实）：**截至 dev 分支 2026-09-07 状态，opencode 没有内建的 `/loop`、`/schedule`、`/goal` 或 cron 服务**；docs 层面把"自动化"交给外部（cron/脚本调用 `run`、GitHub Actions）。

### 3.2 追踪中的请求与实现（全部 open / 未合并）

| 编号 | 类型 | 标题 | 状态（2026-09-07） | 关键点 |
|------|------|------|-------------------|--------|
| [#11232](https://github.com/anomalyco/opencode/issues/11232) | issue | Feature Request: Native Scheduling for Opencode | **closed / not_planned**（2026-08-20 由机器人以"60 天无活动自动关闭"结束；曾被创始人 jlongster 认领过） | 提议 `opencode schedule --cron …`；34 次反应（22 👍 + 12 ❤️）；评论指向 #2828/#5408/#5895/#5891 等一批重叠请求 |
| [#18001](https://github.com/anomalyco/opencode/issues/18001) | issue | Implement /loop command for automated iterative task execution | **open**（43 👍，11 条评论，assignee rekram1-node） | 用户故事：`/loop 60m 'check MCP kanban, code task, commit, and push'`；评论引用 Claude Code scheduled-tasks 官方文档并列出 fixed-interval / dynamic self-paced / stop conditions / autonomous sentinel 四种形态 |
| [#41907](https://github.com/anomalyco/opencode/issues/41907) | issue | /loop — dynamically repeat a command N times | **open**（2026-08-12） | `/loop 5 <cmd>`、`/loop until-pass <cmd>`；指出 `session.command` 执行路径是 single-shot |
| [#41906](https://github.com/anomalyco/opencode/issues/41906) | issue | /schedule — run recurring background tasks (cron-like routines) | **open**（2026-08-12，**assignee = jlongster（opencode 创始人）**） | 提议 `/schedule every 5m <prompt>` 与 `/schedule cron '*/5 * * * *' <prompt>`；指出仓库内已有内部 `BackgroundJob.Service`（`packages/opencode/src/background/job.ts`，现用于压缩/摘要清理）但无用户可见的 recurring 调度 |
| [#44191](https://github.com/anomalyco/opencode/pull/44191) | **PR** | feat(core): cron tools for scheduled prompts | **open，未合并**（2026-08-22，作者 paolodelia99） | 社区实现：新增 `cron_add`/`cron_list`/`cron_delete` 工具 + 内置 skill `scheduling.md` + `CronService`（PriorityQueue）+ `CronDeliveryPort`；PR 自称 "Inspired by Claude code's /loop"、"Partially Closes #41907, #18001"；设计目标是 TUI 与桌面 app 使用，不暴露给 SDK |
| [#13414](https://github.com/anomalyco/opencode/pull/13414) | PR(draft) | feat(opencode): add automations backend | **closed，未合并**（2026-02-13 开，2026-05-02 关闭；作者 GriffinBoris） | automations 领域 + scheduler + server 路由 + OpenAPI/SDK 暴露，标签 needs:issue |
| [#13413](https://github.com/anomalyco/opencode/pull/13413) | PR(draft) | feat(app): add automations UI | **closed，未合并**（2026-04-14 由机器人关闭；作者 GriffinBoris） | Automations 页面 + schedule editor |

另有社区旁证（#11232 评论）：joshue031 在个人 fork 分支 `feature/automations` 实现了 Automations tab 并在评论中称其服务端支持 `POST /automation/{id}/run` 等 HTTP 路由（未提 PR；同评论透露 opencode 有"通知/收件箱"类 UI 概念）；issue #18001 评论中 yondifon 发布过独立仓库 `yondifon/opencode-loop`（plugin 路线实现）。

### 3.3 小结（推断）

内部 `BackgroundJob` 服务 + 两轮 automations/loop 请求 + 一份未合并的 cron 实现 PR，说明：**官方路线图上"定时自动化"位置明确（41906 由创始人认领即证据），但截至本报告日尚未落地**；官方推荐路径是 GitHub Actions `schedule` + headless `run` + `serve` API（事实，见 2.6/2.7）。社区若不想等，OS 级 cron/systemd 是当前事实标准（见第 4 节框架的做法）。

---

## 4. 社区框架 cobusgreyling/loop-engineering（对 opencode 支持最完整的 Loop 框架）

仓库：https://github.com/cobusgreyling/loop-engineering （MIT 协议；README 自称是学术论文 [Lulla et al. 2026, arXiv:2608.21884] "the community reference they reviewed"，并列出 Addy Osmani 原文与 Cobus Greyling Substack 为思想来源）。以下均为 raw.githubusercontent.com/cobusgreyling/loop-engineering/main/ 下原文件。

### 4.1 定位与 CLI

README 原话："This is a **pattern library for operating agents around a codebase**. It is not a 'rewrite the module' button." 与 "**Stop prompting. Design the loop. Get a score.**"

```bash
npx @cobusgreyling/loop init . --pattern daily-triage --tool opencode   # --tool 默认 claude，可换 grok/codex/opencode
npx @cobusgreyling/loop doctor .
npx @cobusgreyling/loop cost --pattern daily-triage --level L1
```

- 命令集：`init` · `doctor` · `status` · `audit` · `cost`。
- 渐进模型："Week one is **report-only**"，随后 "Roll out **L1 report → L2 assisted → L3 unattended** only after the verifier has been right for a week."
- 模式库（含节奏/周1模式/成本）：Daily Triage (1d–2h, L1)、Thin loop（事件+1d）、PR Babysitter (5–15m)、CI Sweeper (5–15m, L2)、Dependency Sweeper、Changelog Drafter、Post-Merge Cleanup、Issue Triage。
- 明示 opencode 为第一等公民：文档树 `examples/{claude-code,grok,codex,openclaw,opencode,github-actions}/`。

### 4.2 opencode 专用示例摘录（examples/opencode/daily-triage.md 原样要点）

该文件开宗明义（与本报告第 3 节结论互相印证）：

> "Same pattern as Grok and Claude Code; **scheduling runs from cron/systemd** and each tick invokes `opencode run` instead of a TUI `/loop`."

**L1 报告-only（第 1 周）**——cron/systemd 定时器每早启动一个全新 opencode 会话，prompt 强制先读 `STATE.md` 以跨会话携带状态：

```bash
opencode run \
  "Run the loop-triage skill. Read STATE.md first. Append high-priority items under High Priority and Watch List. Update Last run timestamp. Do not edit source code. End with a 5-line summary." \
  --title "Daily triage — repo:${PWD##*/}"
```

高频节奏的 cron 行（原样）：

```cron
0 */2 * * * cd /repo && opencode run "Run loop-triage. Report obvious small wins only. Update STATE.md. No code changes."
```

**L3（第 3 周+）小型自动修复**——命名 `implementer`/`verifier` agent + git worktree 隔离 + `--dir` 传入 worktree 路径 + `--file` 传 diff 给 verifier：

```bash
FIX_ID="$(date +%Y%m%d%H%M%S)"
WORKTREE="../wt-small-fix-$FIX_ID"
git worktree add "$WORKTREE" -b "loop/small-fix-$FIX_ID"
opencode run \
  "Run loop-triage. For one high-priority single-file bugfix: implement the minimal fix, run tests, and write a summary plus diff path. Escalate ambiguous or denylisted paths." \
  --agent implementer \
  --dir "$WORKTREE"
DIFF_FILE="$(mktemp /tmp/loop-diff.XXXXXX.patch)"
git -C "$WORKTREE" diff > "$DIFF_FILE"
opencode run "Review this diff against project rules and tests. APPROVE or REJECT only." \
  --agent verifier \
  --file "$DIFF_FILE"
```

作者注释（原样）：“The verifier sees only the diff; the implementer works only inside the worktree. This preserves the same **maker/checker split** Claude Code expresses with `isolation: worktree`.”

**Goal 模式替代**（run-until-done 的一次性变体）：

```bash
opencode run "Goal: all tests on main pass and lint is clean. Stop when tests pass and write the evidence."
DIFF_FILE="$(mktemp /tmp/goal-diff.XXXXXX.patch)"
git diff > "$DIFF_FILE"
opencode run "Verify the goal is complete. APPROVE only if tests pass and the diff is minimal." \
  --agent verifier \
  --file "$DIFF_FILE"
```

### 4.3 Starter 文件结构（starters/minimal-loop-opencode/，全部原样引用）

目录内容（GitHub contents API 证实）：`README.md`、`AGENTS.md`、`LOOP.md`、`opencode.json.example`、`STATE.md.example`、`skills/`（空目录，等待用户把 `templates/SKILL.md.loop-triage` 拷入）。

**① opencode.json.example**（三个 agent 已配好，直接对应第 2.3 节的官方 agent 语法）：

```json
{
  "$schema": "https://opencode.ai/config.json",
  "agent": {
    "loop-triage": {
      "name": "loop-triage",
      "description": "Report-only daily triage loop. Reads STATE.md, updates high-priority items, and does not edit source code in L1 mode.",
      "mode": "primary",
      "prompt": "Read AGENTS.md, LOOP.md, STATE.md, and skills/loop-triage/SKILL.md. Run report-only triage. Update STATE.md. Do not edit source code unless the human has explicitly enabled L2.",
      "permission": { "bash": "ask", "edit": "ask" }
    },
    "implementer": {
      "name": "implementer",
      "description": "L2 implementer agent for minimal scoped fixes inside an isolated worktree.",
      "mode": "subagent",
      "prompt": "Implement only the requested minimal fix. Stay within the supplied worktree, respect AGENTS.md and LOOP.md, run documented tests, and stop for human approval on denylisted paths.",
      "permission": { "bash": "ask", "edit": "ask" }
    },
    "verifier": {
      "name": "verifier",
      "description": "Checker agent for L2+ changes. Reviews diffs and test evidence; APPROVE or REJECT only.",
      "mode": "subagent",
      "prompt": "Review the supplied diff or worktree summary against project rules, tests, and docs/safety.md. Do not edit files. Respond with APPROVE or REJECT and concise evidence.",
      "permission": { "bash": "ask", "edit": "deny" }
    }
  }
}
```

**② STATE.md.example**（磁盘记忆的最小形态——Addy 所谓"第 6 件事"的实例化）：

```markdown
# Loop State — My Project
Last run: never

## High Priority (loop is acting or waiting on human)

## Watch List

## Recent Noise (ignored this run)

---
Run log: —
```

**③ AGENTS.md**（安全规则落盘）：

```markdown
# AGENTS.md — Opencode Minimal Loop
These rules are loaded by opencode before loop work.

## Loop Mode
- Start in L1 report-only mode.
- Read `STATE.md` before any triage.
- Update `STATE.md` after every loop run.
- Do not edit source code until the human explicitly enables L2.

## Safety
- Never push or merge without human approval.
- Never edit `.env`, `.env.*`, `auth/`, `payments/`, `secrets/`, or `credentials/`.
- Use a git worktree for every code-changing attempt.
- Max 3 fix attempts per item; escalate after that.

## Verification
- For L2+ changes, dispatch a verifier sub-agent after implementation.
- Run the project's documented tests before proposing a fix.
- Record test evidence in `STATE.md`.
```

**④ LOOP.md**（配置/预算/人闸）：

```markdown
# Loop Configuration — Minimal Triage (Opencode)

## Active Loops
| Pattern | Cadence | Status | Command |
|---------|---------|--------|---------|
| Daily Triage | 1d | L1 report-only | `opencode run "Run loop-triage" --agent loop-triage` via cron/systemd |

## Human Gates
- No auto-fix until L2 checklist complete.
- All high-risk paths require human review (see docs/safety.md denylist).

## Worktrees
- Use an explicit `git worktree` and run opencode with `--dir <path>` for implementer runs (L2+).
- One worktree per fix attempt; discard after verifier REJECT.

## Connectors (MCP)
- MCP optional for L1 report-only loops.
- For L2+: GitHub MCP can read CI/issues; scope connectors to read + comment until trusted.

## Budget
- Max sub-agent spawns per run: 0 (L1).
- Review STATE.md daily.
- If token spend hits 80% of daily cap, switch to report-only.
```

**⑤ skills/loop-triage/SKILL.md**（来自 templates/SKILL.md.loop-triage）——frontmatter + 输出协议（High-Priority / Watch / Noise / State Updates 四段式），关键规则原话："Only put something in 'High-Priority' if a reasonable engineer would want to know about it today." "Never propose architectural overhauls during triage — this skill is for signal, not invention."

### 4.4 运维与暂停

```bash
crontab -l
systemctl --user list-timers
opencode session list
opencode export > loop-session.json    # 审计导出
```

- 暂停机制："disable the cron/systemd timer or set `loop-pause-all` in `STATE.md` and teach the skill to stop acting."
- 何时该选 opencode 而非 TUI 系 agent（README 原话三连）：① "You want **scheduling without a TUI** — cron/systemd can call `opencode run` in CI, in tmux, and on a headless box." ② "You want to keep your **state and skills in plain files** (`STATE.md`, `skills/`) that survive a host change." ③ 你本来就在用 systemd timers / cron。

---

## 5. 官方如何自用（dogfooding 证据）

### 5.1 GitHub workflow（仓库内的自用机器人）

来源：https://github.com/anomalyco/opencode/blob/dev/.github/workflows/opencode.yml（2026-09-07 原样）：

```yaml
name: opencode
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
jobs:
  opencode:
    if: |
      contains(github.event.comment.body, ' /oc') || startsWith(github.event.comment.body, '/oc') ||
      contains(github.event.comment.body, ' /opencode') || startsWith(github.event.comment.body, '/opencode')
    runs-on: blacksmith-4vcpu-ubuntu-2404
    permissions:
      id-token: write
      contents: read
      pull-requests: read
      issues: read
    steps:
      - name: Checkout repository
        uses: actions/checkout@34e114876b0b11c390a56381ad16ebd13914f8d5 # v4.3.1
      - uses: ./.github/actions/setup-bun
      - name: Run opencode
        uses: anomalyco/opencode/github@2c14fc5586fe0b88e5c04732d2e846769cc35671 # latest
        env:
          OPENCODE_API_KEY: ${{ secrets.OPENCODE_API_KEY }}
          OPENCODE_PERMISSION: '{"bash": "deny"}'
        with:
          model: opencode/claude-opus-4-5
```

要点：
- 官方自用目前是 **事件驱动**（issue/PR 评论 `/oc`），**不是 schedule 驱动**——与第 3 节"官方尚无内建定时自动化"互为印证。
- **权限收缩做法**：环境变量注入 `OPENCODE_PERMISSION: '{"bash": "deny"}'` ——整体禁掉 bash，agent 只能编辑文件/读代码（在自己的 runner 里）。
- 模型用自家 `opencode/claude-opus-4-5`（OpenCode Zen 路由）。
- 旁证：v1.18.29 release 作者字段为 `opencode-agent[bot]`（官方用自己 agent 生成 release 的迹象；未经 release notes 正文证实，标注为推断级别信息）。
- 社区模板呼应：docs/github.mdx 的 review/triage 官方示例同样使用该 action + 只读权限（contents/pull-requests/issues 全 read）。

---

## 6. 对照 Claude Code `/loop` `/goal` 的差距与补位方案

### 6.1 Claude Code 侧事实（间接一手证据，见"访问限制"声明）

依据 ① Addy Osmani 原文对照表与正文：Claude Code 具备 "**Scheduled tasks and cron**, `/loop`, `/goal`, hooks, GitHub Actions"；`/loop` 按节奏重跑，`/goal` 持续运行直到用户写明的条件为真，且每轮之后由**另一个独立小模型**判定是否完成（"the agent that wrote the code isn't the one grading it"）；还有 subagent 的 `isolation: worktree` 与 `--worktree` flag。

依据 ② opencode issue #18001 评论（2026-04-15，bradystroud，转述官方文档 URL `https://code.claude.com/docs/en/scheduled-tasks#run-a-prompt-repeatedly-with-/loop`）：Claude Code `/loop` 的官方形态为四要素——**fixed interval mode**（`/loop 5m <prompt>`）；**dynamic/self-paced mode**（`/loop <prompt>` 不带间隔，模型自行决定何时唤醒自己——等 build 就每 60s 轮询、等队列就每 30min，避免烧 prompt-cache TTL）；**stop conditions**（模型判定无事可做或用户中断即停）；**autonomous variant**（无用户输入的哨兵 prompt，全无人值守）。

### 6.2 差距矩阵

| 原语（Claude Code/Codex 参考） | opencode 现状（2026-09-07） | 差距 |
|---|---|---|
| `/loop <间隔> <prompt>`（节奏性重跑） | 无内建命令（#41907/#18001 open，#44191 未合并） | **缺内建**，但 cron/systemd 每 tick `opencode run` 100% 等价（cobus 框架做法） |
| `/loop` 动态自定节奏 | 同上 | 缺；需要 agent 内有能力感知时间并自唤醒（无原生支持证据；#18001 评论认为这是与 cron 的本质区别） |
| `/goal <条件>`（run-until-done + 独立判停模型） | 无内建；`general`/自定义 verifier subagent 可模拟"第二个模型判停" | **缺内建**，可用外层循环实现（cobus goal-engineering 仓库提供 `/goal` skill 路线） |
| Scheduled tasks / Automations（cron 注册 + 结果收件箱） | GitHub Actions `schedule` + 结果落 PR/log（官方支持）；desktop app 侧 automations 未合并（#13413/#13414） | 官方支持路径存在但不在 TUI/客户端内 |
| `isolation: worktree`、`--worktree` | 无同名 flag；靠 shell 侧 `git worktree` + `opencode run --dir` | 同能力、不同操作面（开源 CLI 本就 headless 化，差异小） |
| Hooks（事件驱动触发） | plugins 事件钩子（session.idle / tool.execute / command 等）+ GitHub 事件 | **能力齐平甚至更广** |

### 6.3 补位方案（三层，均为事实可组合项）

1. **固定节奏 = cron/systemd + `opencode run`**：把 6.1 的 fixed interval 翻译为 cron 行（见 4.2 示例），prompt 中强制先读 `STATE.md` 与 SKILL。
2. **run-until-done = 外层 while/until 脚本 + verifier subagent**：外部循环每次只问"条件是否成立"，由 verifier agent（不同模型、`edit: deny`）做独立判停——结构上复刻 `/goal` 的 maker/checker 分离（见 4.2 的 Goal Mode 摘录）。
3. **无人值守 automations = GitHub Actions `schedule` + `workflow_dispatch` + `anomalyco/opencode/github@latest`**：官方文档第 2.7 节样板直接可用，注意官方警告（无人值守需显式授予 contents/pull-requests 写权限）。

等待官方原语的跟踪方式：盯 #41906（创始人认领）与 #44191（cron 实现 PR）的状态即可。

---

## 7. 实操配方（可直接复制）与风险刹车

### 7.1 第一天配方：L1 报告-only 循环（只读、零代码改动）

```bash
# 1) 三份文件落盘（见第 4.3 节内容）
mkdir -p skills/loop-triage
cp templates/SKILL.md.loop-triage skills/loop-triage/SKILL.md   # 取自 cobusgreyling/loop-engineering
cp starters/minimal-loop-opencode/STATE.md.example STATE.md
cp starters/minimal-loop-opencode/opencode.json.example opencode.json

# 2) 先手动跑一次验证（读状态 → 输出报告 → 更新状态，不改源码）
opencode run "Run the loop-triage skill. Read STATE.md first. Do not edit source code. End with a 5-line summary." --agent loop-triage

# 3) 注册定时（crontab -e）
0 9 * * * cd /path/to/repo && opencode run "Run loop-triage. Update STATE.md. No code changes." --agent loop-triage --continue --auto >> /tmp/loop-triage.log 2>&1
```

> 提示：`--auto` + 只读 agent 权限（verifier/loop-triage 配 `edit: ask`、外层可再加 `OPENCODE_PERMISSION`）即可在无人值守下保持"报告-only"刹车。状态文件被 git 跟踪、diff 可见，形成审计痕迹（推断自第 4.3 节设计 + 官方权限文档）。

### 7.2 一周验证后：L2 辅助修复（maker/checker + worktree）

原样命令见 4.2 节"L3（第 3 周+）小型自动修复"代码块（`git worktree add` → `opencode run --agent implementer --dir "$WORKTREE"` → `git diff` → `opencode run --agent verifier --file "$DIFF_FILE"`）。判据："APPROVE or REJECT only"。

### 7.3 风险刹车清单（来源逐一标注）

| 刹车 | 机制 | 来源 |
|---|---|---|
| 死循环防护 | `doom_loop` 权限：**同一工具调用以完全相同输入重复 3 次**时触发，默认 `ask`（人工介入）；文档原文 "triggered when the same tool call repeats 3 times with identical input" | docs/permissions.mdx |
| 步数上限 | agent `steps`：限制 agentic 迭代次数，到顶强制转纯文本总结；⚠️ 旧名 `maxSteps` 已弃用 | docs/agents.mdx |
| token 用量可见性 | `opencode stats [--days] [--models] [--project]`；`session export`（JSON 审计） | docs/cli.mdx |
| 显式 deny 优先 | `--auto` 只自动批准**未显式 deny** 的请求："Explicit `deny` rules are still enforced." | docs/permissions.mdx |
| 细粒度命令白/黑名单 | `bash: {"*": "ask", "git *": "allow", "rm *": "deny"}`，最后匹配者生效；另有 `external_directory`（工作区外路径默认 ask）与 `.env` 读取默认拒绝 | docs/permissions.mdx |
| 只读起步 | 内置 `plan` agent / 自定义 agent `edit: deny, bash: deny`；官方 GitHub action 样板即 `bash: deny` | docs/agents.mdx、opencode.yml |
| 上下文上限 | provider/model `limit`（context/output，opencode 配置 schema，非本次抓取范围，另见 opencode.ai/config.json）；`OPENCODE_DISABLE_AUTOCOMPACT`；输出上限 `OPENCODE_EXPERIMENTAL_OUTPUT_TOKEN_MAX`；bash 默认超时 `OPENCODE_EXPERIMENTAL_BASH_DEFAULT_TIMEOUT_MS` | docs/cli.mdx（env 表） |
| 进程级兜底 | 外层：cron 行内 `timeout`/`flock` 包一层；STATE.md 设 `loop-pause-all` 可让 skill 停止行动（cobus 约定） | cobusgreyling 仓库 |
| 人的闸门 | cobus L1-L3 渐进："only after the verifier has been right for a week"；Addy："done' is a claim and not a proof" | 两来源 |

> 与用户常引用的 `MAX_ITERATIONS` 对应物说明（推断+事实）：**未在 opencode 官方文档/环境变量表中找到名为 `MAX_ITERATIONS` 的全局环境变量**；等价刹车是 agent 级 `steps` 配置 + 权限矩阵 + `doom_loop`，无全局限流变量（若需要每会话硬上限，可在外层循环脚本实现计数——cobus 的 "Max 3 fix attempts per item" 即属此类约定）。

### 7.4 需要避免的反模式（Addy 原文警示，非新造）

直接 prompt 仍然有效；loop 是杠杆不是替代品；同样的 loop 不同人用结果相反——"One uses it to move faster on work they understand deeply. The other uses it to avoid understanding the work at all."（https://addyosmani.com/blog/loop-engineering/）

---

## 8. 来源清单

全部一手来源，抓取于 2026-09-07：

**概念权威**
- Addy Osmani, *Loop Engineering*（正文全文）: https://addyosmani.com/blog/loop-engineering/

**opencode 官方文档源（dev 分支，与 opencode.ai/docs 同源）**
- CLI: https://raw.githubusercontent.com/anomalyco/opencode/dev/packages/web/src/content/docs/cli.mdx
- Agents: …/agents.mdx
- Commands: …/commands.mdx
- Skills: …/skills.mdx
- Plugins: …/plugins.mdx
- Permissions: …/permissions.mdx
- Server: …/server.mdx
- GitHub: …/github.mdx
- （tui/ecosystem 等未抓取，报告未引用）
- 配置 schema（引用自记忆/既有知识，非本次抓取）: https://opencode.ai/config.json

**opencode 官方 GitHub**
- README: https://raw.githubusercontent.com/anomalyco/opencode/dev/README.md
- dogfooding workflow: https://github.com/anomalyco/opencode/blob/dev/.github/workflows/opencode.yml
- 最新 release（v1.18.29, 2026-09-04）: https://api.github.com/repos/anomalyco/opencode/releases/latest
- Issue/PR（状态经 GitHub API 实时核实，2026-09-07）:
  - PR #44191 cron tools for scheduled prompts（open）: https://github.com/anomalyco/opencode/pull/44191
  - Issue #41906 /schedule（open，jlongster 认领）: https://github.com/anomalyco/opencode/issues/41906
  - Issue #41907 /loop N times（open）: https://github.com/anomalyco/opencode/issues/41907
  - Issue #18001 /loop（open，43 👍）: https://github.com/anomalyco/opencode/issues/18001
  - Issue #11232 Native Scheduling（closed/not_planned）: https://github.com/anomalyco/opencode/issues/11232
  - PR #13414 automations backend（closed draft）: https://github.com/anomalyco/opencode/pull/13414
  - PR #13413 automations UI（closed draft）: https://github.com/anomalyco/opencode/pull/13413

**开源框架 cobusgreyling/loop-engineering（MIT）**
- README: https://raw.githubusercontent.com/cobusgreyling/loop-engineering/main/README.md
- examples/opencode/daily-triage.md: …/examples/opencode/daily-triage.md
- starters/minimal-loop-opencode/{AGENTS.md, LOOP.md, opencode.json.example, STATE.md.example}（目录列表经 GitHub contents API 确认）
- templates/SKILL.md.loop-triage
- 关联（未深挖）: github.com/cobusgreyling/goal-engineering（/goal skill 路线）、github.com/cobusgreyling/memory-engineering

**对照用（Claude Code；间接一手证据，code.claude.com 境内直连超时未能逐字核对）**
- Addy Osmani 原文中的对照表（Automations/Worktrees/Skills/… 双列）
- opencode issue #18001 评论对官方文档的转述（URL: https://code.claude.com/docs/en/scheduled-tasks#run-a-prompt-repeatedly-with-/loop）

## 相关笔记

- [[03.Engineering/Common_Area/AI/Loop-Engineering|Loop Engineering（循环工程）— 概念速览]]
- [[03.Engineering/Common_Area/AI/Spec-Driven Development|Spec-Driven Development (SDD)]]
- [[03.Engineering/Common_Area/AI/MCP|MCP 协议]]
