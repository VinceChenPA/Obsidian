---
tags:
  - type/note
created: 2026-05-14
updated: 2026-09-25
status: done
source:
---
# Nginx 反向代理配置

## 场景：opencode web 对外访问

### 架构

```
用户 → Nginx (:80) → opencode web (:4096, 127.0.0.1)
```

- Nginx 监听 `0.0.0.0:80`，反向代理到 `127.0.0.1:4096`
- opencode 服务内置 Basic Auth 认证（V2：用户名固定 `opencode`，密码由 `opencode service set password` 配置；V1 为环境变量 `OPENCODE_SERVER_USERNAME` / `OPENCODE_SERVER_PASSWORD`）
- 支持 WebSocket 升级

### Nginx 配置

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:4096;
        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_read_timeout 86400;
    }
}
```

### 文件位置

- 配置文件: `/etc/nginx/sites-available/opencode-web`
- 启用方式: 软链接到 `/etc/nginx/sites-enabled/`

### 管理命令

```bash
# 测试配置
sudo nginx -t

# 重载
sudo systemctl reload nginx

# 验证
curl -s -o /dev/null -w "%{http_code}" http://localhost/                     # 应返回 401
curl -s -o /dev/null -w "%{http_code}" -u "opencode:密码" http://localhost/  # 应返回 200
```

### opencode 服务启动（V2）

启动脚本 `~/.local/bin/oc-web`（V2 版）：
- `opencode service set port/hostname` 配置监听（默认 `127.0.0.1:4096`）
- `opencode service start` 启动，最后 `opencode pair` 输出配对链接
- 同步副本 `~/oc_ws/oc-web`；参数持久化在 `~/.config/opencode/service.json`
- 详见 [[03.Engineering/Common_Area/Linux/opencode-web-setup|opencode web 部署记录]]

> V1 历史：以 `opencode web --port/--hostname` 启动独立进程，通过 `OPENCODE_SERVER_USERNAME/PASSWORD` 环境变量注入认证凭据。
