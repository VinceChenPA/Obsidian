# opencode web 部署记录

## 架构

```
用户 → Nginx (:80) → opencode web (:4096, 127.0.0.1)
         ↓
    Basic Auth (OPENCODE_SERVER_USERNAME / OPENCODE_SERVER_PASSWORD)
```

## 组件

### Nginx 反向代理
- 配置文件: `/etc/nginx/sites-available/opencode-web`
- 监听 `0.0.0.0:80`，反向代理到 `127.0.0.1:4096`
- 支持 WebSocket 升级

### opencode web 服务
- 启动脚本: `~/.local/bin/oc-web`
  - 默认监听 `127.0.0.1:4096`
  - 内置 OPENCODE_SERVER_USERNAME / OPENCODE_SERVER_PASSWORD 导出
- 管理脚本: `~/oc_ws/oc-web`
  - 支持 start/stop/status/restart/logs 子命令
  - 也包含认证凭据导出

## 清理记录

- 清理了两个旧 opencode web 实例（0.0.0.0:3001 和旧 serve 命令实例）
- 卸载了 lunel-cli（npm 全局包）及相关所有文件/脚本/引用
