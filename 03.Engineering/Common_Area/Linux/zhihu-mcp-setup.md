---
tags:
  - type/note
  - opencode
  - zhihu
  - mcp
created: 2026-09-25
updated: 2026-09-25
status: done
source: https://github.com/Douyh123/zhihu-mcp
---
# 知乎 MCP 配置（zhihu-mcp）

在 1.6GB 阿里云服务器上部署知乎 MCP（Python + Playwright + FastMCP），用于抓取知乎文章/搜索/发布。

## 背景：为什么需要它

知乎反爬严格，常规抓取全部失败：

| 方式 | 结果 |
|------|------|
| webfetch（HTTP） | `403 Forbidden` |
| Playwright（无头） | 风控页 `40362 请求存在异常` |
| exa 抓取 | `CRAWL_UNKNOWN_ERROR` |
| Wayback 存档 | 无该文章存档 |
| 知乎 API（直连） | 需客户端签名/登录态 |

## 选型

- 采用 [`Douyh123/zhihu-mcp`](https://github.com/Douyh123/zhihu-mcp)：Python + Playwright + FastMCP，HTTP MCP 服务，扫码登录
- 另一个候选 `iteng007/zhihu-mcp-server`（Node，zse96 签名）**已归档**，未采用

## 部署

```bash
# 1. 克隆（放 github_repos，已被 oc_ws 的 .gitignore 忽略）
cd ~/oc_ws/github_repos && git clone --depth 1 https://github.com/Douyh123/zhihu-mcp

# 2. 依赖（uv）
cd zhihu-mcp
uv venv .venv --python 3.13
uv pip install --python .venv -r requirements.txt   # fastmcp/playwright/loguru/pydantic
.venv/bin/python -m playwright install chromium     # 下载 chromium-1243

# 3. 登录（无头扫码，见下）
PYTHONUNBUFFERED=1 .venv/bin/python login_headless.py --timeout 300

# 4. 启动服务
./serve.sh start   # → http://127.0.0.1:18060/mcp
```

**服务管理**：`./serve.sh {start|stop|status|logs}`（脚本自动设置 `LD_LIBRARY_PATH=~/.local/lib/playwright-deps`、setsid nohup、PID 文件在 `~/.local/state/zhihu-mcp/`）

## 无头扫码登录（关键改造）

原项目 `login.py` 依赖图形界面显示二维码，服务器无 GUI。改造 `login_headless.py`：

1. 无头打开 `zhihu.com/signin`
2. 用页面内 JS 定位二维码元素（本项目页面为 `CANVAS.Qrcode-qrcode`），裁剪截图
3. 用 Pillow 渲染为**终端 ASCII 二维码**（半块字符，适配终端 2:1 宽高比）
4. 同一会话轮询等待扫码（最长 300s），成功后保存 cookies

- cookies 保存到 `cookies/cookies.json`（含 `d_c0`/`z_c0`/`SESSIONID`，**敏感，勿提交**）
- 二维码约 5 分钟失效；超时需重新生成
- 二维码图片同时存 `cookies/login_qrcode.png`（裁剪版）与 `login_page.png`（整页备查）

## opencode 接入

`~/.config/opencode/opencode.json`：

```json
"zhihu": {
  "type": "remote",
  "url": "http://127.0.0.1:18060/mcp",
  "oauth": false,
  "disabled": false
}
```

- 注意 `oauth: false`：本地服务无需鉴权，避免 V2 走 OAuth 流程
- 连接验证：`opencode mcp list`（显示 `zhihu connected`，11 个工具）

## 工具（11 个）

`check_login_status`、`get_login_qrcode`、`delete_cookies`、`publish_article`、`publish_video`、`search_content`、`get_recommend_list`、`get_feed_detail`（含正文+评论，自动存 md）、`post_comment`、`get_user_profile`、`reply_comment`

## 注意事项与踩坑

- **每次工具调用都会启动/关闭一个 Chromium**（不常驻）：单次约 3-5s、峰值 +300MB 内存；内存紧张的机器需注意
- **Playwright 版本 GC 互删**：Node 版（webfetch 插件）与 Python 版（本 MCP）共存于 `~/.cache/ms-playwright`；执行任一 `playwright install` 时，会清理"无 `.links` 引用"的浏览器版本——曾导致 Node 版 `chromium-1223` 被删。修复：在对应项目目录重新 `playwright install chromium`
- **`main.py` headless 参数 bug**：原代码 `headless = not args.no_headless` 且 `--no-headless` 默认 True → 永远非无头。已改为默认无头（`--no-headless` 可切换），服务器必须用 `--headless`
- **`check_login_status` 可能误报未登录**（首页选择器过时，返回"no login indicators found"），但实际抓取正常——以 `get_feed_detail` 实际结果为准
- cookies 有效期有限，失效后需重新扫码
- 项目自带 `skills/zhihu-auth`、`skills/zhihu-publish`（工具用法说明）

## 验证

首次实测抓取知乎文章成功（标题与正文完整返回），见 [[03.Engineering/Common_Area/AI/Frontier-Engineering|Frontier Engineering]] 的来源链接。
