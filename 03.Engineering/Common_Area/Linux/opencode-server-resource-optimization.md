---
tags:
  - type/note
  - opencode
created: 2026-09-25
updated: 2026-09-25
status: done
source:
---
# opencode 服务器资源优化（1.6GB 阿里云实例）

低内存服务器（2 核 1.6GB）上运行 opencode V2 的资源分析与两项优化：**MCP 启动精简**、**交换空间启用**。

## 背景

- 服务器：阿里云 Debian 13，2 核 1.6GB，**原无 swap**
- opencode V2 相关进程峰值接近 900MB；Playwright 降级时 Chromium 另需 ~370MB → 存在 OOM 风险

## 优化一：精简 MCP 启动命令

### 问题

MCP 通过 `npx` 启动时，进程链为 `npm exec` → `sh -c` → `node` 三层，`npm exec` 层纯属包装开销（每层 RSS 80–100MB）：

| MCP | 优化前 | 优化前 RSS |
|-----|--------|-----------|
| bing-search | `npm exec` + `sh` + `node` | ~191MB |
| getnote（×2 份） | 6 进程 | ~339MB |
| **合计** | **9 进程** | **~530MB** |

### 方案

改为直接 `node` 调用已安装包的入口，用绝对路径不依赖 PATH：

- 全局 `~/.config/opencode/opencode.json`：
  ```json
  "getnote": {
    "type": "local",
    "command": [
      "/usr/bin/node",
      "/home/vince/.npm-global/lib/node_modules/@getnote/mcp/dist/index.js"
    ]
  }
  ```
- 项目 `~/oc_ws/opencode.json`：
  ```json
  "bing-search": {
    "type": "local",
    "command": [
      "/usr/bin/node",
      "/home/vince/.npm-global/lib/node_modules/bing-cn-mcp/build/index.js"
    ]
  }
  ```
- `bing-cn-mcp` 由 npx 临时缓存改为全局安装（`npm i -g bing-cn-mcp`），不再依赖易失的 `~/.npm/_npx/` 目录

### 效果

| 指标 | 优化前 | 优化后 |
|------|--------|--------|
| MCP 进程数 | 9 | 3 |
| MCP RSS 合计 | ~530MB | ~252MB |
| 系统 available 内存 | 812MB | 868–920MB |

- 验证：`opencode mcp list` 三个 `connected`；`bing_search`、`getnote.get_quota` 实测通过
- V2 在配置变化后会**自动重载 MCP**，无需重启服务

## 优化二：启用 swap

### 问题

`/swapfile` 已于 2026-09-13 创建（2GB），但**从未 `mkswap` / `swapon`**，`/etc/fstab` 也无条目 → 系统一直零 swap。

### 方案

`~/oc_ws/add-swap.sh`（幂等脚本，需 sudo）：`chmod 600` → `mkswap` → `swapon` → 写入 `/etc/fstab` → 设 `vm.swappiness=10`。

```bash
sudo bash ~/oc_ws/add-swap.sh
```

### 验证（2026-09-25）

| 项 | 值 |
|----|----|
| `/proc/swaps` | `/swapfile` file 2097148 KB，prio -2 |
| 文件权限 | `-rw-------`（600） |
| 持久化 | `/etc/fstab`: `/swapfile none swap sw 0 0` |
| swappiness | 10（持久化到 `/etc/sysctl.d/99-swappiness.conf`） |
| 当前使用 | 0B（内存充足，未触发换出） |

## 当前资源基线（2026-09-25）

| 进程 | RSS | 说明 |
|------|-----|------|
| `opencode serve --service` | ~387MB | V2 主服务 |
| `node bing-cn-mcp` | ~100MB | 项目级 MCP |
| `node @getnote/mcp` ×2 | ~153MB | 全局 MCP，两个 location 各一份（V2 正常行为） |
| **合计** | **~625MB / 4 进程** | |

系统：used ~771MB / available ~911MB；Swap 2GB 未用；负载 ~0.1（2 核）；PSI 内存压力 avg10 = 0.00。

## 结论与后续

- 内存余量 + swap 兜底后，Playwright 降级已无 OOM 风险
- 若仍需压缩内存，可考虑：减少 location 数量（避免全局 MCP 重复启动）、或禁用不常用的 MCP
- 相关笔记：[[03.Engineering/Common_Area/Linux/opencode-v2-migration|opencode V1→V2 迁移记录]]、[[03.Engineering/Common_Area/Linux/opencode-webfetch-architecture|opencode webfetch 架构]]
