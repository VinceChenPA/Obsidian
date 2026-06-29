# Matt Pocock Skills - 安装记录

安装日期: 2026-06-29
环境: opencode (阿里云服务器)
安装路径: `~/oc_ws/.agents/skills/` (opencode + Claude Code 共享)

## 已安装技能

| 技能 | 文件 | 类型 | 说明 |
|------|------|------|------|
| `grill-me` | `.agents/skills/grill-me/SKILL.md` | user-invoked | 盘问入口，`disable-model-invocation: true`，委托给 `/grilling` |
| `grilling` | `.agents/skills/grilling/SKILL.md` | model-invoked | 实际盘问逻辑，逐个问问题+提供推荐答案 |
| `domain-modeling` | `.agents/skills/domain-modeling/SKILL.md` | model-invoked | 主动打磨领域模型，维护 CONTEXT.md 和 ADR |

## 来源

全部来自 Matt Pocock skills 仓库原始内容 (英文版):
- https://github.com/mattpocock/skills
- `grill-me` → `skills/productivity/grill-me/SKILL.md`
- `grilling` → `skills/productivity/grilling/SKILL.md`
- `domain-modeling` → `skills/engineering/domain-modeling/SKILL.md`

## 使用方法

opencode 重启后自动发现，用户说"盘问我"/"grill me" 或 "domain modeling"/"领域建模" 即可触发。
