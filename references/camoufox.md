---
description: Camoufox 详解 —— 反指纹 Firefox、包选择、三层依赖、无 sudo 补库、指纹注入
---

# Camoufox 详解

Camoufox = 改过的 Firefox + **Playwright 兼容的 Python API**。页面操作仍按 Playwright 模型写（`page.goto()` / `page.click()` / `page.screenshot()`）——只改浏览器初始化那一步。它在 **C++ 层**注入指纹（navigator / 屏幕 / WebGL / 字体 / WebRTC / GeoIP / 时区 / locale），JS 检测不到；底层用 [BrowserForge](https://github.com/daijro/browserforge) 按真实流量统计分布生成指纹。

适用：目标站有反爬 / 指纹检测，需要隐身 Firefox、指纹注入/轮换、按代理 IP 伪造 GeoIP。

## 包选择：`camoufox` vs `cloverlabs-camoufox`

| | `camoufox`（官方稳定）| `cloverlabs-camoufox`（活跃 alpha）|
|---|---|---|
| PyPI 版本 | 0.4.11 | 0.6.0 |
| 定位 | 稳定，**发布有延迟** | **浏览器开发主线** |
| 额外能力 | 基础指纹注入 | per-context 指纹 + `fingerprint_preset` 真实指纹预设 |

- 两个包**共用 `camoufox` import/CLI 命名空间**，**装在各自 venv 里，别混**。
- **Camoufox 是 Python-only，没有 npm 包**，只能 `uv`/`pip` 装。

## 安装

```bash
pip install -U "camoufox[geoip]"              # 稳定
pip install -U "cloverlabs-camoufox[geoip]"     # 活跃 alpha
```

`geoip` extra 用代理时**强烈建议**：按出口 IP 算经纬度/时区/国家/locale，避免代理露馅。

下载浏览器二进制（从 GitHub release 拉，缓存 `~/.cache/camoufox/browsers/`）：

```bash
camoufox fetch
```

要最新预发布：

```bash
python -m camoufox sync
python -m camoufox set official/prerelease
python -m camoufox fetch
```

有桌面环境到此即可直接用。**无桌面 Linux 还要补系统库** → 见下文。

## 三层依赖模型

跑通 Camoufox 要凑齐三层，缺任一层都起不来：

1. **Python 包** — PyPI，`uv`/`pip` 装
2. **浏览器二进制** — `camoufox fetch`，缓存 `~/.cache/camoufox/browsers/`
3. **系统 GTK/X11/ALSA 库** — OS 层，有桌面的机器自带；headless server / WSL 缺这层

> 第 1 层只在 PyPI、conda-forge 没有，所以 pixi 装不了（例外：pixi **项目**支持 PyPI 依赖，见方案 C）。

## 跑页面脚本

```python
from camoufox.sync_api import Camoufox
with Camoufox() as browser:
    page = browser.new_page()
    page.goto("https://example.com")
```

```bash
uv run --with "cloverlabs-camoufox[geoip]" script.py
```

常用参数：`headless`、`os`（`"windows"/"macos"/"linux"` 或列表）、`geoip`、`proxy`、`humanize`（拟人鼠标）、`locale`、`block_images`、`block_webrtc`、`config`（手动覆盖指纹）、`fingerprint_preset`（cloverlabs 真实预设）。

```python
with Camoufox(fingerprint_preset=True, os="macos") as browser:
    ...
```

## 补齐系统库（无桌面 / 无 sudo）

无桌面库 + 无 sudo 的机器跑 Camoufox 会报 `libgtk-3.so.0: cannot open shared object file`。用 pixi 装用户态库：

**实测硬依赖：`gtk3` + `alsa-lib`（装 gtk3 自动拖入传递依赖）。不要装 `nss`，Camoufox 浏览器自带。**

### 方案 A（默认）：全局库 env + uv

预装一份全局 env，任意脚本复用：

```bash
pixi global install -e camoufox-libs gtk3 alsa-lib
# 可选：编辑 ~/.pixi/manifests/pixi-global.toml 把 [envs.camoufox-libs] exposed = {} 避免命令污染 PATH
```

```bash
CFDIR=$(ls -d ~/.cache/camoufox/browsers/official/*/ | tail -1)
export LD_LIBRARY_PATH="$HOME/.pixi/envs/camoufox-libs/lib:$CFDIR"
uv run --with "cloverlabs-camoufox[geoip]" script.py
```

### 方案 B：`pixi exec` 临时跑（不建项目、不留痕迹）

```bash
CFDIR=$(ls -d ~/.cache/camoufox/browsers/official/*/ | tail -1)
pixi exec -s gtk3 -s alsa-lib -- bash -c \
  "export LD_LIBRARY_PATH=\"$CONDA_PREFIX/lib:$CFDIR\"; uv run --with cloverlabs-camoufox[geoip] script.py"
```

### 方案 C：pixi 项目（长期固定项目，`pixi run` 一条命令）

```bash
pixi init camoufox-runner && cd camoufox-runner
```

`pixi.toml`：

```toml
[workspace]
channels = ["conda-forge"]
platforms = ["linux-64"]

[dependencies]
python = "3.12.*"
gtk3 = "*"
alsa-lib = "*"

[pypi-dependencies]
cloverlabs-camoufox = { version = "*", extras = ["geoip"] }

[activation.env]
LD_LIBRARY_PATH = "$CONDA_PREFIX/lib:/home/<you>/.cache/camoufox/browsers/official/<ver>"
```

```bash
camoufox fetch          # 仍需手动下载浏览器
pixi run python script.py
```

> pixi 项目支持 `[pypi-dependencies]`，三层全包办不用 uv。`[activation.env]` 是必须的——裸 `pixi run` 不会自动设 LD path。

### 怎么选

- **方案 A【默认】**：多脚本复用同一份库
- **方案 B**：一次性临时跑，不留痕迹
- **方案 C**：长期项目，`pixi run` 一条命令

## 第三方 Camoufox agent CLI

不想写 Python 脚本、想像 playwright-cli 那样用 shell 子命令驱动：

| 项目 | 特点 | 适合 |
|---|---|---|
| `camoufox-cli`（Bin-Huang） | `@e1` ref 风格、持久指纹身份、多身份并行、代理轮换、跨平台 | 需要持久身份 / 多身份管理 |
| `camoufox-browser`（rlgrpe） | 语义标签 ref、自带 MCP、Linux/macOS | 需要 MCP 集成 |

```bash
# camoufox-cli
camoufox-cli open https://example.com
camoufox-cli snapshot -i
camoufox-cli click @e1

# camoufox-browser
camoufox-browser open https://example.com
camoufox-browser click 'button:Sign in'
```

> 两者都是年轻第三方项目，按实验工具看待稳定性。
