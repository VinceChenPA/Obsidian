# Nginx 反向代理配置

## 场景：opencode web 对外访问

### 架构

```
用户 → Nginx (:80) → opencode web (:4096, 127.0.0.1)
```

- Nginx 监听 `0.0.0.0:80`，反向代理到 `127.0.0.1:4096`
- opencode web 内置 Basic Auth 认证（环境变量 `OPENCODE_SERVER_USERNAME` / `OPENCODE_SERVER_PASSWORD`）
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
curl -s -o /dev/null -w "%{http_code}" http://localhost/           # 应返回 401
curl -s -o /dev/null -w "%{http_code}" -u "user:pass" http://localhost/  # 应返回 200
```

### opencode web 启动

启动脚本 `~/.local/bin/oc-web`：
- 默认监听 `127.0.0.1:4096`
- 通过环境变量注入认证凭据

管理脚本 `~/oc_ws/oc-web`：
- 支持 `start|stop|status|restart|logs` 子命令
- 也注入了认证凭据
