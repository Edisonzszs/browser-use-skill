---
name: browser-use
version: 1.2.0
author: Hermes Agent
license: MIT
description: Use when browser automation needs Playwright (CLI/MCP/script), Camoufox anti-fingerprint Firefox, or attaching to an existing Chrome session — headed browsing, login-state reuse, protected PDF extraction, anti-bot evasion, or choosing between uv/npm and MCP/CLI.
metadata:
  hermes:
    tags: [browser, automation, playwright, camoufox, anti-fingerprint, pdf, scraping]
    related_skills: [computer-use]
---

# Browser Use Skill

## When to Use

- 需要通过浏览器自动化执行操作（导航、点击、填表、截图、下载）
- 下载浏览器中预览的受保护 PDF / 文件，或提取嵌入的 pdf.js 数据
- 绕过网站的直接下载限制获取资源
- 需要反指纹 / 反爬 / 按代理伪造 GeoIP 的隐身浏览器
- 询问 Playwright 用 uv 还是 npm 安装、用 MCP 还是 CLI
- 需要接管已有的 Chrome / Edge 登录态

**Don't use for:**
- Hermes 内置 `browser_*` 工具能搞定的简单页面访问、表单填写、截图、轻量爬取 — 那个零配置直接用，别启动本 Skill
- 纯 HTTP API 调用（用 curl / web_extract 即可）

> 简单判断：**能用内置 browser 工具解决的，别启动这个 Skill。** 本 Skill 的开销是额外的安装/配置步骤，换来更强的控制力和场景覆盖。

## 前置依赖

| 方案 | 依赖 | 安装方式 |
|---|---|---|
| **Playwright CLI（默认）** | Node.js >= 18 | `npm i -g @playwright/cli@latest` |
| **Camoufox / Python 脚本** | Python >= 3.10 + [uv](https://docs.astral.sh/uv/) | `uv tool install camoufox` 或 `uv run --with camoufox …` |

## 方案选择

**默认用 Node + Playwright CLI（`@playwright/cli`），不要再问用户用哪个方案。** 这是官方对 coding agent 的首选（token 最省）。

```bash
npm install -g @playwright/cli@latest
playwright-cli install --skills              # 落地官方 skill，拿到完整命令面
playwright-cli open https://example.com
playwright-cli snapshot                      # 拿 eN refs
playwright-cli click e15
```

> `npx @playwright/cli@latest <cmd>`（免全局安装）和 `npm i -g` 后直接 `playwright-cli <cmd>` 等价，按手感选。完整命令、会话、attach 现有 Chrome 等见 [references/playwright.md](references/playwright.md)。

**只有以下情况才偏离默认**（用户明示，或场景明确需要）：

| 改用 | 触发条件 |
|---|---|
| **接管现有 Chrome** | 用户要在已登录的当前浏览器会话里直接操作（复用登录态） |
| **Playwright MCP** | 用户明确要 MCP，或要跨多轮维持同一浏览器上下文做探索式 / 自愈式长程自动化 |
| **Playwright 脚本（库）** | 用户要交付可复现的 Python/Node 脚本、批量任务或接 CI |
| **Camoufox** | 目标站有反爬 / 指纹检测，需要隐身 Firefox、指纹注入、按代理 IP 伪造 GeoIP/时区/locale。详见 [references/camoufox.md](references/camoufox.md) |

用户若直接点名某方案，按用户来，不用再确认。

### MCP vs CLI 核心结论

- **CLI（`@playwright/cli` + SKILLS）= coding agent 首选**：token 更省，不往上下文塞庞大工具 schema 和冗长无障碍树。
- **MCP** 适合需要持久状态 + 富内省 + 反复推理页面结构的长程 agentic loop（探索 / 自愈 / 自治）。

> uv vs npm 的深度分析（心智模型、Python wheel 自带 Node driver 等）见 [references/playwright.md](references/playwright.md)。一句话结论：要驱动浏览器做事 → npm + `@playwright/cli`；要写 Python 脚本或接 Camoufox → uv。

## 接管已有 Chrome

两种方式：

1. **Playwright CLI attach**（推荐）：`playwright-cli attach --cdp=chrome` — 详见 references/playwright.md
2. **open-claude-in-chrome MCP**：Claude 官方 Chrome 扩展的 clean-room 实现，移除域名黑名单，支持任意 Chromium。GitHub: `noemica-io/open-claude-in-chrome`。常用工具：`read_page`（DOM）、`javascript_tool`（执行 JS）、`tabs_context_mcp`（tab 列表）。

## pdf.js 数据提取

当页面嵌入了 pdf.js 阅读器时，可利用 `PDFViewerApplication.pdfDocument.getData()` 提取 PDF：

```javascript
(async () => {
  const iframe = document.querySelector('iframe');
  const win = iframe.contentWindow;
  const app = win.PDFViewerApplication;
  const doc = app.pdfDocument;
  const data = await doc.getData();
  const blob = new Blob([data], {type: 'application/pdf'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = '文件名.pdf';
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  URL.revokeObjectURL(url);
})();
```

提取前先在 Console 执行 `PDFViewerApplication.pagesCount` 确认 PDF 已加载完。该方法利用的是用户在浏览器中已合法访问的资源。

## 跨 Agent 平台安装

本 skill 基于 SKILL.md 标准格式，可直接被以下 Agent 平台消费：

| Agent 平台 | 安装路径 | 说明 |
|---|---|---|
| **Hermes Agent** | `~/.hermes/skills/browser-use/` | 直接放入，自动扫描 frontmatter 触发 |
| **Codex CLI (OpenAI)** | 项目 `AGENTS.md` 或全局 `~/.codex/` | 读取 AGENTS.md 作为 system instructions |
| **OpenCode** | 项目 `AGENTS.md` 或全局配置 | 遵循 AGENTS.md 约定 |
| **OpenClaw** | 项目/全局 skills 目录 | 按平台 skill 约定放入即可 |

> 所有 Agent 平台共享同一份 SKILL.md，无需额外适配。references/ 子目录通过相对路径引用，保持完整。

## Common Pitfalls

1. **`playwright-cli install --skills` 不执行**：确保 `@playwright/cli` 已全局安装，且当前目录有写权限。`npx` 方式也可以但可能因缓存版本过旧导致 skill 内容不准。
2. **Camoufox 包混装**：`camoufox` 和 `cloverlabs-camoufox` 共用 `camoufox` 命名空间，装在同一个 venv 会冲突。用独立 venv 隔离。
3. **Windows URL `&` 截断**：cmd.exe 会把 `&` 解释为命令分隔符。用 `^&` 转义，或改用 Git Bash。
4. **Playwright MCP `--isolated` vs `--user-data-dir`**：同一 profile 同时只能一个浏览器实例使用，并发场景必须 `--isolated` 或各自 `--user-data-dir`。
5. **pdf.js 提取时机**：`PDFViewerApplication.pdfDocument` 可能在 iframe 加载后仍为 null（异步渲染），先执行 `PDFViewerApplication.pagesCount` 确认非空再提取。
6. **Camoufox 无 sudo 补库**：无 root 的 Linux 机器需用 pixi 用户态安装 GTK/ALSA（见 camoufox.md 方案 A/B/C），否则 Firefox 启动报 `libgtk-3.so: cannot open shared object file`。

## Verification Checklist

- [ ] Playwright CLI 已安装：`playwright-cli --version` 返回版本号
- [ ] 官方 skill 已落地：`playwright-cli install --skills` 无报错
- [ ] 能打开页面并拿到 snapshot：`playwright-cli open <url>` + `playwright-cli snapshot` 返回 eN refs
- [ ] Camoufox（如使用）：`python -m camoufox fetch` 成功下载浏览器二进制
- [ ] 接管 Chrome（如使用）：`playwright-cli attach --cdp=chrome` 连接成功
- [ ] pdf.js 提取（如使用）：`PDFViewerApplication.pagesCount` 返回非零值
