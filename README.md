# Browser Use Skill

浏览器自动化 Skill — 用 Playwright（CLI / MCP / Python·Node 脚本）、Camoufox（反指纹 Firefox）或接管已有 Chrome 登录态来操控页面。

跨 Agent 平台可用：Hermes Agent / Codex CLI / OpenCode / OpenClaw。

---

## 📋 能做什么

- 🌐 **浏览器自动化** — 导航、点击、填表、截图、文件上传/下载
- 📄 **受保护 PDF 提取** — 拦截 pdf.js 嵌入数据，绕过直接下载限制
- 🎭 **反指纹 / 反爬** — Camoufox 隐身 Firefox，C++ 层注入指纹
- 🔗 **接管已有 Chrome** — 复用当前浏览器登录态
- 📦 **可复现脚本** — Python / Node 自动化脚本交付、批量任务、CI 集成

## 🏗️ 仓库结构

```
browser-use-skill/
├── SKILL.md                        # 主 Skill（Agent 平台直接消费）
├── LICENSE.txt
└── references/
    ├── playwright.md               # Playwright 详解（CLI/MCP/脚本三入口）
    └── camoufox.md                 # Camoufox 反指纹 Firefox 完整文档
```

## 🚀 安装

将 `SKILL.md` + `references/` 放入对应 Agent 平台的 skills 指令目录：

| Agent 平台 | 安装路径 |
|---|---|
| **Hermes Agent** | `~/.hermes/skills/browser-use/` |
| **Codex CLI (OpenAI)** | 项目 `AGENTS.md` 或全局 `~/.codex/` |
| **OpenCode** | 项目 `AGENTS.md` 或全局配置 |
| **OpenClaw** | 项目/全局 skills 目录 |

所有平台共享同一份 SKILL.md，无需额外适配。

## 📌 方案选择

| 方案 | 默认？ | 适用场景 |
|---|---|---|
| **Playwright CLI** (`@playwright/cli`) | ✅ 默认 | Agent 驱动浏览器，token 最省 |
| **Playwright MCP** | — | 长程 agentic loop（探索 / 自愈 / 自治） |
| **Playwright 脚本** | — | 可复现脚本 / 批量 / CI |
| **Camoufox** | — | 反指纹 / 反爬 / GeoIP 伪造 |
| **接管已有 Chrome** | — | 复用已登录的浏览器会话 |

### 快速开始（默认 Playwright CLI）

```bash
npm install -g @playwright/cli@latest
playwright-cli install --skills
playwright-cli open https://example.com
playwright-cli snapshot
playwright-cli click e15
```

## 📄 License

MIT
