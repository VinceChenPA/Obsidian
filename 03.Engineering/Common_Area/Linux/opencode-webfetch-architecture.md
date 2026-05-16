---
tags: [opencode, webfetch, playwright, defuddle, web-scraping]
---

# opencode 自定义 webfetch 工具架构

## 架构: 两级降级策略

```
请求 → fetch HTTP → Defuddle 提纯 → wordCount ≥ 15? → 是 → 返回 clean markdown
                                  ↓ 否
                           Playwright Chromium (headless)
                                  ↓
                           Defuddle 提纯 → 返回 clean markdown
```

## 三层对比

| 方案 | 内存 | 进程 | 耗时 (github.blog) | 返回 |
|-----|------|------|-------------------|------|
| Webget (内置) | 0 | 0 | ~2.2s | 183KB 原始 HTML |
| fetch + Defuddle | ~0.5MB | 0 | ~3.0s | 11KB markdown |
| Playwright 直取 | ~224MB | 5 | ~5.9s | 11KB markdown |

## 系统依赖

阿里云服务器 (Debian 13, 2核 1.6GB) 缺少 Playwright Chromium 依赖。手动下载 43 个 .so 到 `~/.local/lib/playwright-deps`，通过 `LD_LIBRARY_PATH` 加载。

```bash
export LD_LIBRARY_PATH="$HOME/.local/lib/playwright-deps${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}"
```

需安装的包: `libnspr4 libnss3 libatk1.0-0t64 libatk-bridge2.0-0t64 libxcomposite1 libxdamage1 libxfixes3 libxrandr2 libgbm1 libxkbcommon0 libasound2t64 libatspi2.0-0t64 libxrender1 libdrm2 libxi6`

## 工具文件

- 项目级: `.opencode/tools/webfetch.ts`
- 全局: `~/.config/opencode/tools/webfetch.ts`
- 依赖: `@opencode-ai/plugin`, `defuddle`, `playwright`
- 运行逻辑: 原生 HTTP fetch → Defuddle → 内容过薄 → Chromium headless

## 参考资料

- [opencode Custom Tools](https://opencode.ai/docs/custom-tools/)
- [Defuddle](https://github.com/kepano/defuddle) — HTML 提纯为 markdown
