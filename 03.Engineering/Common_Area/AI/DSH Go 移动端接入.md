---
tags:
  - type/note
  - AI
  - DSH
  - 移动端
created: 2026-11-07
updated: 2026-11-07
status: done
source: https://github.com/xiazhi88/dshgo + 本机 3080 实例排查
---

# DSH Go 移动端接入

> 用手机/平板连本机 DSH 的可行路径：App + 电脑端 `dshgo` 插件。核心结论：**App 不能直接连 3080**，必须由插件把实例改写成 loopback 后再转发到局域网端口（默认 3081）。

## 一、架构

```
┌──────────┐   局域网 / Tailscale   ┌──────────────────┐
│ 手机 App │ ────────────────────> │ dshgo :3081      │
│          │ <──────────────────── │   ↓ 改写 Host/Origin
└──────────┘                       │ dsh web :3080    │
                                   └──────────────────┘
```

- `dshgo` 是**装在 dsh web 实例里的插件**，它转发到宿主实例自身端口（`upstreamPort` 自动检测），没有“指定 upstream 地址”的配置项。
- 因此「固定到 3080 实例」= 把插件装进 3080 实例的 profile，而不是在 App 里填 3080。

## 二、为什么不能直连 3080

`dsh web` 有三道限制，手机端一条都绕不过：

| 限制 | 说明 |
|---|---|
| 只绑 loopback | `dsh web` 固定监听 `127.0.0.1`；官方禁了 `--host 0.0.0.0`（会向网络暴露任意代码执行） |
| `/api` 的 Host/Origin 栅栏 | 只认 loopback 或 `--trusted-host` 白名单，且 `Origin` 必须与 `Host` 同源 |
| 会话 cookie 绑定 authority | cookie 名为 `dsh-auth-<authority hash>`，校验时比对签发时的 authority |

插件把入站请求的 `Host`/`Origin` 统一改写成 `127.0.0.1:<dshPort>` 再转发，栅栏永远看到 loopback，cookie authority 也始终一致。另外补两件事：首屏注入一次 launch token（token 每次启动都变）、注入 `crypto.randomUUID` 与 `AbortSignal.any` 两个 polyfill（非安全上下文/旧 WebView 缺失会导致发送失败）。

## 三、安装与配置（电脑端）

```bash
# 1. 先确保只有一个 dsh web 实例（见下方「两个坑」）
pkill -f "dsh web --host 127.0.0.1 --port 0"

# 2. 装到 web profile（~/.dsh/profiles/web）
dsh plugin --profile web add @xiazhi88/dshgo -w
# 或 npm 源更快：dsh plugin --profile web add @xiazhi88/dshgo -w

# 3. 必须重启 dsh web（插件后端半只在启动时挂载）
~/dsh_ws/dsh-web.sh restart

# 4. 验证
curl -s http://127.0.0.1:3080/__dshgo__/info
```

期望返回：

```json
{
  "name": "dshgo",
  "listening": true,
  "port": 3081,
  "upstreamPort": 3080,
  "addresses": ["http://192.168.1.100:3081"],
  "tailscale": ["http://100.101.102.103:3081"]
}
```

插件配置（`~/.dsh/profiles/web/cordis.patch.yml`，可选）：

```yaml
- id: dshgo
  config:
    port: 3081      # 对外端口，默认 3081；被占时自动往后试 10 个
    bind: 0.0.0.0   # 对外绑定地址
```

装好后设置里出现一级入口「局域网访问」（本机地址、二维码、访问密码）。

## 四、App 端

1. 首页 `+` 添加连接：`http://<电脑局域网IP>:3081`（出门用 Tailscale 填 `http://100.x.x.x:3081`）
2. 若电脑端设了访问密码，首次进入解锁；App 走指纹/面部
3. 保存后即固定在连接列表，可多台电脑切换

## 五、两个坑（本机环境）

1. **两个实例共用同一 profile**：本机 3080 常驻实例与 `dsh web --port 0`（44363）临时实例都读 `~/.dsh/profiles/web`。若临时实例还在，它也会加载 `dshgo` 并可能先占 3081，把错误实例暴露出去。装插件前务必先停临时实例。
2. **装/卸后必须重启 `dsh web`**：只改配置不重启，页签会出现但提示「服务端半没有运行」。

其他注意：

- 版本要求 DSH ≥ `0.1.2-rc.1`（本机 `0.2.1-rc.2` 满足）；通知与 App 内会话状态依赖 `session/list` RPC 与 `/api/remote.mux` WebSocket（`0.1.2-alpha.2` 起）。
- 旧名 `dsh-lan` 升级需先 `dsh plugin --profile web remove dsh-lan`，否则设置里会出现两个同名页签。
- 移动端布局已内联，无需另装 `dsh-web-mobile`。
- 本机状态：2026-11-07 排查时**尚未安装**该插件，3081 端口空闲。

## 六、安全边界

`3081` 等于把可执行任意命令的 DSH 暴露到局域网：

- 只在可信网络（家里 / 自己的 WiFi）使用，**不要做公网端口转发**
- 需要公网访问用 Tailscale（设备级私有网络，点对点）
- 可选访问密码：闸门拦在注入 launch token 之前；scrypt 加盐存 `~/.dsh/dshgo-auth.json`（0600）；改密码会轮换签名密钥，所有已解锁设备立即失效；改密码只接受本机请求

## 相关

- [[03.Engineering/Common_Area/AI/DeepSeek Harness 最佳实践|DeepSeek Harness 最佳实践]]
- [[03.Engineering/Common_Area/AI/Super-Agent-架构（DSH）|Super-Agent 架构（DSH）]]
- [[03.Engineering/Common_Area/AI/MCP|MCP 协议]]
- 上游：[xiazhi88/dshgo](https://github.com/xiazhi88/dshgo)（App + 插件，MIT）
