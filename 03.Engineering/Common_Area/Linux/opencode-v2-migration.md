---
tags:
  - type/note
  - opencode
created: 2026-09-25
updated: 2026-09-25
status: done
source:
---
# opencode V1→V2 迁移记录（2026-09-25）

从 V1（1.x）迁移到 V2（2.0.16）的环境变更、破坏性变化与清理清单。

## 升级方式

- V2 用官方 curl installer 安装为单文件二进制：`~/.opencode/bin/opencode`（约 200MB ELF）
- 系统级 `/usr/bin/opencode`（V1 1.14.50，npm 全局）仍存在，未卸载；**不要用它操作 V2 已迁移的数据库**

## 三处破坏性变化（官方定义）

1. **插件 API 重写**——V1 插件实现不能在 V2 运行
2. **Server API 与客户端契约变更**
3. **CLI 配置**：分层 `tui.json(c)` → 单一全局 `~/.config/opencode/cli.json`

## 配置形态变化

| V1 | V2 |
|----|----|
| `provider` | `providers` |
| `mcp`（直接嵌套） | `mcp.servers`（`enabled` → 反向 `disabled`） |
| `permission`（按工具分组） | `permissions`（有序数组） |
| `snapshot` | `snapshots` |
| `plugin` | `plugins` |
| `compaction.preserve_recent_tokens` / `reserved` | `compaction.keep.tokens` / `buffer` |
| `compaction.tail_turns` / `prune` | 无对应（V2 忽略并警告） |

- V1 配置仍被兼容读取（不重写源文件），部分字段被静默或警告忽略
- `lsp: true` 被接受，但 V2 不再运行语言服务（无 LSP 工具与诊断）
- `opencode db` 子命令被移除

## 自定义工具：tools 目录 → 插件

- V1：`.opencode/tools/*.ts`（全局 `~/.config/opencode/tools/`）自动发现
- V2：**该机制失效**，工具须由插件通过 `ctx.tool.transform()` 注册
  - 插件放 `~/.config/opencode/plugins/`（项目 `.opencode/plugins/`），自动加载并热重载
  - default 导出为 `{ id, setup }`（本地插件无需 import 官方包，运行时仅校验形状）
  - 注册项：`{ name, description, input(JSON Schema), options: { codemode: true }, execute }`
- 已迁移实例：webfetch（见 [[03.Engineering/Common_Area/Linux/opencode-webfetch-architecture|opencode webfetch 架构]]）

## web 服务

- `opencode web` → `opencode service set/start` + `opencode pair`；认证用户名固定为 `opencode`
- 见 [[03.Engineering/Common_Area/Linux/opencode-web-setup|opencode web 部署记录]]

## 记忆机制

- 跨会话记忆仍在 SQLite 的 `session_memory` 表（V2 迁移后保留）
- 读写方式改为 node:sqlite 直连（`opencode db` 已移除），命令见 `~/oc_ws/AGENTS.md`

## 迁移后清理清单（2026-09-25）

| 目标 | 说明 |
|------|------|
| `~/.config/opencode/supermemory.jsonc`、npm 缓存中的 supermemory/codemem | V1 插件残留，V2 不兼容 |
| `~/.cache/opencode/packages/`（56M） | V1 插件/LSP 缓存 |
| `~/.local/state/oc-web/`、`~/.local/bin/oc-web.v1.bak` | V1 web 运行数据 / 含明文凭据备份 |
| `.opencode/tools/`（全局+项目） | V1 tools 机制，已失效 |
| `@opencode-ai/plugin` 依赖、项目 `.opencode/node_modules`（39M） | V1 API 与旧工具依赖，无使用者 |
| `oc`、`switch-web.sh` | 调用 V1 命令的过时脚本 |

## 经验教训

- **不要让旧版二进制操作新版迁移后的 DB**（V1 曾因此连续 10 天 cron 失败）
- 迁移后逐一验证：MCP（`opencode mcp list`）、插件（`opencode plugin list`）、模型、权限
- V1 兼容层允许渐进迁移，但插件与自定义工具必须重写

## 参考

- [Migrate from V1](https://opencode.ai/v2/docs/migrate-v1)
- [V2 Plugins](https://opencode.ai/v2/docs/build/plugins)
