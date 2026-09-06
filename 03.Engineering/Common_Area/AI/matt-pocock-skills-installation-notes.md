# Matt Pocock Skills - 安装记录

首次安装: 2026-06-29(仅 3 项)
全量同步: 2026-09-06(对齐上游 v1.2.3)
环境: opencode (本机 + 阿里云服务器)
安装路径: `~/.agents/skills/` (opencode + Claude Code 共享)

## 首次安装(2026-06-29,仅 3 项)

| 技能 | 文件 | 类型 | 说明 |
|------|------|------|------|
| `grill-me` | `.agents/skills/grill-me/SKILL.md` | user-invoked | 盘问入口，`disable-model-invocation: true`，委托给 `/grilling` |
| `grilling` | `.agents/skills/grilling/SKILL.md` | model-invoked | 实际盘问逻辑，逐个问问题+提供推荐答案 |
| `domain-modeling` | `.agents/skills/domain-modeling/SKILL.md` | model-invoked | 主动打磨领域模型，维护 CONTEXT.md 和 ADR |

## 全量同步(v1.2.3,2026-09-06)

按上游 **v1.2.3** 目录结构补齐 **19 项**(仓库技能,另有 `find-skills` 为本仓库外技能,合计 20),`~/.agents/skills/` 与 `~/.claude/skills/` 同步:

| 桶 | 技能 |
|---|---|
| engineering · user-invoked | `grill-with-docs`、`to-spec`、`to-tickets`、`implement`、`triage`、`wayfinder`、`improve-codebase-architecture` |
| engineering · model-invoked | `tdd`、`code-review`、`codebase-design`、`domain-modeling`、`diagnosing-bugs`、`prototype`、`research` |
| productivity · user-invoked | `grill-me`、`handoff`、`teach`、`to-questionnaire` |
| productivity · model-invoked | `grilling` |

> 注:未装(上游仍发布、本机未取):`ask-matt`、`setup-matt-pocock-skills`、`resolving-merge-conflicts`、`wizard`、`wait-what`、`writing-for-agents`;misc/ 与 in-progress/ 桶不随 Claude Code 插件发布,按需单独取;`~/.config/opencode/skills/` 为本机 opencode 侧子集。

## 来源

上游仓库源码镜像(2026-09-06 拉至 v1.2.3 tag):
- 原始仓库:https://github.com/mattpocock/skills
- 本地镜像:`D:\Sources\skills`(对应 v1.2.3 tag,git worktree 检出验证)
- 官方安装:v1.2 起 Claude Code 用插件 `claude plugins install mattpocock-skills`(托管只读);skills.sh `npx skills@latest add mattpocock/skills` 为 Codex/其他代理/可编辑副本路径
- 分类:engineering/(代码)、productivity/(非代码)、misc/、in-progress/(beta)、deprecated/(空)
- 每技能含 `agents/openai.yaml`(Codex 元数据);`AGENTS.md` 为 `CLAUDE.md` 的 symlink

## 使用方法

opencode 重启后自动发现,用户说"盘问我"/"grill me" 或 "domain modeling"/"领域建模" 即可触发;完整流程技能(to-spec → to-tickets → implement → code-review)见 [[03.Engineering/Common_Area/AI/matt-pocock-skills|方法论]]。
