---
tags:
  - type/note
created: 2026-05-14
updated: 2026-09-25
status: done
source:
---
# opencode web 部署记录

> **2026-09-25 更新（V2）**：V2 移除了 `opencode web` 命令，web UI 由共享后台服务承载，改用 `opencode service` + `opencode pair`；认证用户名固定为 `opencode`（不可配置）。详见 [[03.Engineering/Common_Area/Linux/opencode-v2-migration|opencode V1→V2 迁移记录]]。

## 架构

```
用户 → Nginx (:80) → opencode 后台服务 (:4096, 127.0.0.1)
         ↓
    Basic Auth（用户名固定 opencode，密码由 service 配置）
```

## 组件

### Nginx 反向代理

- 配置文件: `/etc/nginx/sites-available/opencode-web`
- 监听 `0.0.0.0:80`，反向代理到 `127.0.0.1:4096`
- 支持 WebSocket 升级
- 详见 [[03.Engineering/Common_Area/Linux/nginx-reverse-proxy|Nginx 反向代理]]

### opencode 服务（V2）

- 启动脚本: `~/.local/bin/oc-web`（V2 版）
  - `opencode service set port/hostname` 配置监听（默认 `127.0.0.1:4096`）
  - `opencode service start` 启动服务，最后 `opencode pair` 输出配对链接
  - 可用 `OC_WEB_PASSWORD` 环境变量固定密码（默认由 V2 生成强密码）
- 同步副本: `~/oc_ws/oc-web`（内容与上者一致，2026-09-25 替换）
- 服务参数持久化在 `~/.config/opencode/service.json`

### 历史（V1，已废弃）

- `opencode web --port <p> --hostname <h>` 启动独立 web 进程
- 通过环境变量 `OPENCODE_SERVER_USERNAME` / `OPENCODE_SERVER_PASSWORD` 注入 Basic Auth
- `~/oc_ws/oc-web` 曾是带 `start|stop|status|restart|logs` 子命令的管理脚本（自管 PID/日志）

## 清理记录

- 2026-05-14：清理两个旧 opencode web 实例（`0.0.0.0:3001` 和旧 serve 命令实例）；卸载 lunel-cli（npm 全局包）及相关文件/脚本/引用
- 2026-09-25：V2 迁移后清理 V1 残留——删除 `~/.local/state/oc-web/`（V1 web 日志）、`~/.local/bin/oc-web.v1.bak`（含明文凭据）与 `~/oc_ws/oc`、`~/oc_ws/switch-web.sh`（调用 V1 命令的过时脚本）；`~/.cache/opencode/packages/`（56M，V1 插件/LSP 缓存）一并清除
