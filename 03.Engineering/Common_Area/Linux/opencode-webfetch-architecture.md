---
tags: [opencode, webfetch, playwright, defuddle, web-scraping]
---

# opencode 自定义 webfetch 工具架构

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

- 项目级: `.opencode/tools/webfetch.ts`
- 全局: `~/.config/opencode/tools/webfetch.ts`
- 依赖: `@opencode-ai/plugin`, `defuddle`, `playwright`
- 参数: `url` (必填), `format` (markdown/text/html), `extract` (boolean, 默认 true)

## 参考资料

- [opencode Custom Tools](https://opencode.ai/docs/custom-tools/)
- [Defuddle](https://github.com/kepano/defuddle) — HTML 提纯为 markdown
- [Playwright](https://playwright.dev/) — 浏览器自动化
