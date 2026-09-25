---
tags:
  - type/note
created: 2026-05-14
updated: 2026-09-25
status: done
source:
---
# opencode 版本记录

| 日期 | 版本 | 备注 |
|------|------|------|
| 2026-05-14 | 1.14.50 | 从 1.14.39 升级（npm 全局安装） |
| 2026-09-25 | **2.0.16** | 迁移到 V2（官方 curl installer）。插件 API、Server API、CLI 配置均有破坏性变化，详见 [[03.Engineering/Common_Area/Linux/opencode-v2-migration\|opencode V1→V2 迁移记录]] |

## 当前版本（V2，2026-09-25 起）

- 路径: `~/.opencode/bin/opencode`（`opencode` 命令解析到它）
- 安装方式: 官方 curl installer（单文件 ELF 二进制）
- 管理命令: `opencode service status|restart`、`opencode mcp list`、`opencode plugin list`、`opencode debug config`
- V2 已移除 `opencode db` 子命令（记忆表改用 node:sqlite 直连）
- npm 源: `https://registry.npmmirror.com`（淘宝镜像）

## 遗留 V1（勿用于当前数据）

- 系统级 `/usr/bin/opencode` 仍为 1.14.50（npm 全局）；V1/V2 共享 `~/.local/share/opencode/opencode.db`，旧版 `run` 会报 `Session not found` / `Unexpected server error`
- 历史 npm 全局用户级路径 `~/.npm-global/bin/opencode` 已不存在

## 历史安装信息（V1，已废弃）

- 路径: `/usr/lib/node_modules/opencode-ai/`
- 二进制: `/usr/bin/opencode` → `../lib/node_modules/opencode-ai/bin/opencode`
- 升级命令: `sudo npm install -g opencode-ai@latest`
