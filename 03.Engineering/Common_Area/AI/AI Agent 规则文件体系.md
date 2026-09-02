# AI Agent 规则文件体系（AGENTS.md / CONTEXT.md / CLAUDE.md …）

> 记录于 2026-09-02，基于 opencode 1.18.9 源码（`session/instruction.ts`、`fs-util.ts`）逐行核对，v1.18.9 与 main 分支实现完全一致。

## 一、各工具约定文件一览

**规则/指令类（自动注入上下文）**

| 文件 | 工具 |
|---|---|
| `AGENTS.md` | opencode、Codex、Windsurf（2025 起生态主流） |
| `CLAUDE.md` | Claude Code |
| `.cursor/rules/*.mdc` / `.cursorrules` | Cursor |
| `.github/copilot-instructions.md` | GitHub Copilot |
| `GLOBAL_RULES.md`、`.windsurf/rules/*.md` | Windsurf |
| `CONVENTIONS.md` | aider（代码风格约束） |
| `CONTEXT.md` | 领域上下文（opencode 第三级回退，已 deprecated，见下文） |

**技能/方法类（按需调用）**
- `SKILL.md` — 技能定义，位于 `.opencode/skills/`、`.agents/skills/`、`.claude/skills/` 等
- 配套格式文档：`CONTEXT-FORMAT.md`、`ADR-FORMAT.md`、`GLOSSARY-FORMAT.md`

**文档类（AI 会主动读取）**
- `README.md`、`docs/adr/*.md`（架构决策记录）、`docs/glossary.md`

## 二、AGENTS.md 与 CONTEXT.md 的区别

- **AGENTS.md = 行为指南（怎么做）**：给 AI 的操作指令，记录项目目的、命令、偏好、坑；每次会话自动加载。
- **CONTEXT.md = 领域知识库（说什么）**：术语表/统一语言/实体关系/歧义裁定/ADR，由 grill-with-docs / domain-modeling 技能维护，人类与 AI 共用，保证讨论用词一致。
- 一句话：**AGENTS.md 告诉 AI 怎么干活，CONTEXT.md 告诉所有人话该怎么说**。

## 三、opencode 加载顺序机制（多 repo / 嵌套工作区）

### 3.1 判定基准（InstanceState.context）
- `ctx.directory`：会话工作目录（cwd，最深）
- `ctx.worktree`：workspace root（项目根 / git 根）；非 git 项目且全局会话时 = `"/"`
- 注：worktree 首次打开项目时解析，之后固化在 ProjectTable；后续从子目录打开只作为 sandbox 追加，不改变 worktree

### 3.2 系统提示注入（每次会话，`systemPaths()`）
1. **全局**：`~/.config/opencode/AGENTS.md`；不存在则回退 `~/.claude/CLAUDE.md`（二选一，不叠加）
2. **项目级**：按文件类型优先级 `AGENTS.md → CLAUDE.md → CONTEXT.md`，**第一种有匹配的类型生效，其余类型不查**（避免跨类型堆叠）
3. `findUp(file, start=ctx.directory, stop=ctx.worktree)`：从会话目录逐级向上**直到 workspace root（含）**，收集**每一层**该文件——嵌套 repo 中从子目录启动，会同时注入子 repo 与父 workspace 各层的全部同名文件（从近到远）

### 3.3 动态加载（`resolve()`，按需）
- AI 用 Read 读取某文件时，从该文件所在目录向上 walk 到项目根，为每一层附加最近的规则文件（每层只取最近一个，per-message 只附加一次）
- 效果：多 repo workspace 中，即使会话不在子 repo 根目录，只要读到子 repo 内文件，就会自动带上该 repo 的规则

### 3.4 补充规则来源（`config.instructions`）
- `opencode.json` 的 `instructions` 字段可追加任意 md：支持 glob（`packages/*/AGENTS.md`）、`~/` 展开、绝对路径、远程 URL（5s 超时）
- 所有指令文件与 AGENTS.md 合并注入

### 3.5 对多 repo workspace 的实用结论
1. **每个子 repo 放自己的 AGENTS.md**（如 `github_repos/obsidian/`、`github_repos/skills/`）→ 从该目录打开会话时两者都会注入
2. 在 workspace 根（如 `/home/vince/oc_ws`）打开 → 只注入根级文件；子 repo 规则在 AI 读取其中文件时按需动态加载
3. **类型勿混用**：某层放 `AGENTS.md`、另一层放 `CLAUDE.md`，会因"第一种类型赢"导致父/子层某些文件被跳过——全链路保持一致用 `AGENTS.md`
4. 同级 `AGENTS.md` 优先于 `CLAUDE.md`，`CLAUDE.md` 优先于 `CONTEXT.md`（deprecated）
5. Claude Code 兼容模式可用环境变量关闭：`OPENCODE_DISABLE_CLAUDE_CODE*`

## 参考资料
- opencode Rules 文档：https://opencode.ai/docs/rules/
- 源码：`packages/opencode/src/session/instruction.ts`（v1.18.9 与 main 一致）、`packages/core/src/fs-util.ts`（findUp/globUp）

## 附：CONTEXT.md 支持状态确认（2026-09-02，非源码渠道核实）

- **当前仍受支持**：仅当项目树中既无 AGENTS.md 也无 CLAUDE.md 时，作为第三级回退被查找；行为正常
- **官方文档已不收录**：最新 rules 文档（opencode.ai/docs/rules/）只提 AGENTS.md 与 CLAUDE.md（兼容回退），全文未出现 CONTEXT.md
- **官方规格明确标注 deprecated**：仓库 specs/v2/session.md 的 V2 对照清单写明 "Decide whether V2 also discovers legacy CLAUDE.md and deprecated CONTEXT.md"——V2 架构下是否继续支持仍未决定，地位不保
- **无移除时间表**：GitHub 无正式移除公告；"will be removed" 仅见第三方 OCX 文档转述
- 官方自己根目录也放 CONTEXT.md，但用于存领域模型术语（人类/被阅读），非注入规则

**结论**：新项目勿用 CONTEXT.md 承载规则，统一 AGENTS.md；需自动注入的自定义 md 用 `opencode.json` 的 `instructions` 字段，不受 deprecated 影响。
