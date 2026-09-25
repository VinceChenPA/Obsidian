---
created: 2026-05-16
tags: [opencode, webfetch, playwright, defuddle, web-scraping, type/note]
updated: 2026-09-25
status: done
source:
---

# opencode 自定义 webfetch 工具架构

> **2026-09-25 更新（V2 迁移）**：OpenCode 已升级到 V2，实现从 V1 自定义工具（`.opencode/tools/*.ts`）迁移为 V2 插件（`~/.config/opencode/plugins/webfetch.ts`）。两级降级策略、效率实测与系统依赖均不变；工具注册方式的现状见下文「工具文件」节。

## 架构: 两级降级策略

```
请求 → fetch HTTP → Defuddle 提纯 → content.length ≥ 100 && wordCount ≥ 15? → 是 → 返回 clean markdown
                                  ↓ 否 (SPA 空壳)
                           Playwright Chromium (headless) — 懒加载
                                  ↓
                           Defuddle 提纯 → 返回 clean markdown
```

- 浏览器懒启动：首次降级时调用 `chromium.launch()`，耗时 ~3s
- session 内复用：后续降级直接复用已有浏览器进程，~500ms
- Playwright 异常时静默降回 fetch 结果

## 效率实测 (阿里云服务器, Debian 13)

| 场景 | Webget (原生 fetch) | webfetch (fetch+Defuddle) | Playwright 直取 |
|-----|-------------------|-------------------------|----------------|
| example.com | 851ms / 0.5KB | **831ms** / 0.2KB md | 3042ms |
| github.blog | 2202ms / 183KB | **2965ms** / 11KB md | 5852ms |

### Token 消耗对比 (github.blog 为例)

| 方案 | 返回大小 | 估算 tokens | 压缩比 |
|-----|---------|------------|-------|
| Webget | 183KB raw HTML | ~47000 | 100% |
| webfetch | 11KB markdown | ~2800 | **5.9%** |
| **每次节省** | **172KB** | **~44000 tokens** | **94%** |

### 系统花销

| 指标 | Webget | webfetch | Playwright |
|-----|--------|---------|-----------|
| 额外内存 | 0 | ~0.5MB (Node RSS) | ~224MB (5 进程) |
| 额外进程 | 0 | 0 | 5 个 Chromium |
| CPU 开销 | 极小 | +Defuddle 解析 ~400ms | 浏览器引擎 完整加载 |

## 系统依赖

阿里云服务器 (Debian 13, 2核 1.6GB) 缺少 Playwright Chromium 依赖。手动下载 43 个 .so 到 `~/.local/lib/playwright-deps`，通过 `LD_LIBRARY_PATH` 加载。

```bash
export LD_LIBRARY_PATH="$HOME/.local/lib/playwright-deps${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}"
```

### Chromium headless shell 依赖

全部 44 个依赖已满足。手动提取的包:
`libnspr4 libnss3 libatk1.0-0t64 libatk-bridge2.0-0t64 libxcomposite1 libxdamage1 libxfixes3 libxrandr2 libgbm1 libxkbcommon0 libasound2t64 libatspi2.0-0t64 libxrender1 libdrm2 libxi6`

若完整 Chromium (非 headless shell) 还需: `libcups2t64 libcairo2 libpango-1.0-0`（headless 模式不依赖）

## 设计迭代

1. **Playwright 直取** — 每次启动 Chromium，~224MB 开销，太重
2. **fetch + Defuddle 纯 HTTP** — 零额外内存，但不支持 SPA 页面
3. **两级降级 (最终)** — 95% 情况走 HTTP，5% SPA 自动降级浏览器

## 工具文件

现状（V2 插件，2026-09-25 迁移）：

- 插件：`~/.config/opencode/plugins/webfetch.ts`（V2 服务端自动发现并热加载）
- 注册方式：`export default { id, setup }` + `ctx.tool.transform()` 注册工具（与内置 webfetch 同名，覆盖内置版）
- 依赖：`defuddle`, `playwright`（安装在 `~/.config/opencode/node_modules/`）
- 参数：`url` (必填), `format` (markdown/text/html), `extract` (boolean, 默认 true), `timeout` (ms, 默认 30000)
- 插件重载/卸载时自动关闭 Chromium（cleanup），避免孤儿进程
- 调试日志：`/tmp/opencode/webfetch-plugin.log`（setup/cleanup/每次调用）

V1 历史形态（已废弃，2026-09-25 删除）：

- 项目级 `.opencode/tools/webfetch.ts`、全局 `~/.config/opencode/tools/webfetch.ts`
- 依赖 `@opencode-ai/plugin`（V1 API）；V1 的 tools 目录机制在 V2 不再被加载

## 参考资料

- [opencode V2 插件文档](https://opencode.ai/v2/docs/build/plugins) — V2 自定义工具注册方式
- [opencode Custom Tools (V1)](https://opencode.ai/docs/custom-tools/) — 历史参考
- [Defuddle](https://github.com/kepano/defuddle) — HTML 提纯为 markdown
- [Playwright](https://playwright.dev/) — 浏览器自动化
