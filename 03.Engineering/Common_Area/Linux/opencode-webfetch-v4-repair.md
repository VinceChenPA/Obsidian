---
tags:
  - opencode
  - webfetch
  - deepseek
  - config
  - type/note
created: 2026-05-22
updated: 2026-09-25
status: done
source:
---

# opencode 配置与 webfetch 修复 2026-05-22

> **2026-09-25 更新（V2 迁移）**：本文是 V1 时期（1.x）的修复记录，问题描述与修复内容作为历史保留。OpenCode 现已升级到 V2：配置字段 `provider` 在 V2 中为 `providers`；webfetch 已迁移为 V2 插件（现状见 [[03.Engineering/Common_Area/Linux/opencode-webfetch-architecture|opencode webfetch 架构]]）。

## DeepSeek V4 context/output limit 配置

opencode 的配置文件 `~/.config/opencode/opencode.json` 中：

- `model` 字段**必须是字符串**（如 `"deepseek/deepseek-v4-flash"`），不支持对象格式
- context/output limit 配置在 `provider.deepseek.models.<model-id>.limit` 下（V2 中为 `providers.deepseek`）
- 字段名：`context`（context limit）、`output`（output limit）

当前配置的两个模型：

| 模型 | context | output |
|------|---------|--------|
| deepseek-v4-flash | 1,000,000 | 384,000 |
| deepseek-v4-pro | 1,000,000 | 384,000 |

Schema 定义：https://opencode.ai/config.json

验证命令：`opencode debug config`（V2 中仍可用，输出配置源列表）

## webfetch 工具修复

### 背景
自定义 webfetch 工具使用两级降级策略抓取网页：优先原生 fetch + Defuddle 提纯，内容不足时降级到 Playwright Chromium。

### 排查发现的问题

| 问题 | 说明 |
|------|------|
| 无错误处理 | fetchViaHttp 抛出异常时工具直接崩溃 |
| HTTP 超时仅 15s | 从国内访问外站经常不够 |
| 无重试机制 | 网络闪断直接失败 |
| Defuddle 解析未保护 | 解析报错也会崩溃 |

### 修复内容

1. **超时提升**：HTTP 15s → 30s（可配置），Playwright 30s → 60s
2. **自动重试**：HTTP 失败后自动重试 1 次（间隔递增）
3. **全面错误保护**：execute 整体 try/catch，失败返回 `[webfetch error] 原因`
4. **Defuddle 安全调用**：解析异常返回空结果，不阻塞降级流程
5. **Accept 头**：添加 `text/html,...` 请求头
6. **package.json**：添加 `"type": "module"` 消除加载警告

### 文件位置（V1 历史；2026-09-25 已迁移到 V2 插件）

- 全局：`~/.config/opencode/tools/webfetch.ts`（已删除）
- 项目：`.opencode/tools/webfetch.ts`（已删除）
- 依赖：`~/.config/opencode/node_modules/`
- 现址：`~/.config/opencode/plugins/webfetch.ts`（V2 插件）
