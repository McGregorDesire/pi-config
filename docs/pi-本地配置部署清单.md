# 本地 Pi 配置部署清单

> 用途：在新设备上重建当前这台机器的 Pi 工作环境。
>
> 快照时间：2026-09-17
>
> 当前环境：macOS arm64，Pi `0.85.1`，Node.js `v22.22.3`
>
> 安全说明：本文不包含 API key、登录 cookie 或模型 provider 的密钥值。`models.json`、浏览器 profile 和部分配置文件含有敏感信息，不要提交到公开仓库。

## 1. 配置范围

Pi 的全局配置目录是：

```text
~/.pi/agent/
```

本仓库是可版本控制的配置源：

```text
/Users/xckj/project/cra/pi-config/
```

新设备部署时分为四类：

| 类别 | 迁移方式 | 是否建议直接复制 |
|---|---|---:|
| Pi 设置和已安装包 | 复制 `settings.json`，再执行 `pi install` | 是 |
| 自定义扩展和技能 | 从本仓库复制源码，再安装各自依赖 | 是 |
| 外部程序 | 用 Homebrew、uv 或 npm 重新安装 | 否，重新安装更可靠 |
| 凭据和运行状态 | 通过安全渠道单独迁移，或在新设备重新登录 | 默认不复制 |

不要把整个仓库直接克隆到 `~/.pi/agent`。应当把需要的文件复制到已有配置目录中。

## 2. Pi 核心设置

文件：`~/.pi/agent/settings.json`

当前内容：

```json
{
  "packages": [
    "npm:pi-cache-graph",
    "git:git@github.com:McGregorDesire/pi-interactive-subagents.git",
    "npm:pi-context-view",
    "npm:pi-slim",
    "npm:pi-rtk-optimizer",
    "npm:@raquezha/noheadroom",
    "git:github.com/kunkun9527/billion-context-pi-lean"
  ],
  "lastChangelogVersion": "0.85.1",
  "theme": "dark",
  "tuiMode": "fullscreen"
}
```

新设备上的最小恢复步骤：

```bash
mkdir -p ~/.pi/agent
# 从本仓库或安全备份恢复 settings.json
cp /path/to/pi-config/settings.json ~/.pi/agent/settings.json
```

本仓库当前没有根目录 `settings.json`；上面的命令中的源路径需要使用你的安全备份，或者手动创建为本文列出的内容。复制后也可以直接逐个安装第 3 节中的包。

## 3. 已安装 Pi 包

当前 `pi list` 中有 7 个用户包：

| 包 | 版本 | 用途 | 额外配置 |
|---|---:|---|---|
| `npm:pi-cache-graph` | `1.0.2` | 查看上下文缓存命中率、累计命中率和 token 统计 | 无 |
| `git:git@github.com:McGregorDesire/pi-interactive-subagents.git` | `3.7.2` | 在 tmux 等复用器中运行交互式 subagent | 无 |
| `npm:pi-context-view` | `0.5.2` | 查看上下文使用量、注入项和配置 | `extensions/pi-context-view.json` |
| `npm:pi-slim` | `0.2.1` | 移除默认 Pi 文档提示，减少上下文 token | 无 |
| `npm:pi-rtk-optimizer` | `0.9.0` | 自动调用 RTK 改写 bash 输出并做输出压缩 | `extensions/pi-rtk-optimizer/config.json` |
| `npm:@raquezha/noheadroom` | `0.3.2` | 把 Pi 请求接入本机 Headroom 压缩代理 | `headroom/settings.json` |
| `git:github.com/kunkun9527/billion-context-pi-lean` | `0.1.69-lean.1` | 会话压缩和按需召回 | 无 |

在新设备上逐个安装：

```bash
pi install npm:pi-cache-graph
pi install git:git@github.com:McGregorDesire/pi-interactive-subagents.git
pi install npm:pi-context-view
pi install npm:pi-slim
pi install npm:pi-rtk-optimizer
pi install npm:@raquezha/noheadroom
pi install git:github.com/kunkun9527/billion-context-pi-lean
```

安装后验证：

```bash
pi list
```

### 3.1 常用命令

| 组件 | 命令或工具 | 说明 |
|---|---|---|
| cache graph | `/cache graph` | 三种缓存统计视图 |
| cache graph | `/cache stats` | 查看统计 |
| cache graph | `/cache export` | 导出 CSV 到项目根目录 |
| interactive subagents | `/plan`、`/iterate`、`/subagent` | 需要可用的 tmux、cmux、zellij、Herdr 或 WezTerm 复用器 |
| context view | `/context`、`/context usage` | 查看上下文使用情况 |
| context view | `/context injections` | 查看注入内容 |
| context view | `/context config` | 查看颜色配置 |
| pi-slim | `/pi <请求>` | 临时恢复 Pi 默认文档提示并处理请求 |
| billion-context | `compress`、`acp_context` | 会话压缩与召回 |

不要同时安装完整 `billion-context-pi` 和 `billion-context-pi-lean`，否则会重复注册工具和 hooks。

## 4. 本地自定义扩展

Pi 实际从 `~/.pi/agent/extensions/` 加载扩展。当前有效的自定义扩展如下：

| 文件或目录 | 功能 | 仓库状态 | 新设备动作 |
|---|---|---|---|
| `ask-user-question.ts` | 通过 TUI 弹窗向用户提问 | 已纳入仓库 | 复制 |
| `bash-guard/` | 在 bash 执行前拦截危险命令并交互确认 | 已纳入仓库 | 复制并 `npm install` |
| `browser/` | Playwright 浏览器工具 | 已纳入仓库 | 复制、`npm install`、安装 Chromium |
| `custom-header.ts` | 自定义 Pi 顶部 `Π` 标题 | 已纳入仓库 | 复制 |
| `herdr-agent-state.ts` | Herdr agent 状态集成 | 仅在本机存在 | 从旧设备单独复制，或不部署 |
| `prompt-snippets/` | 临时启用 prompt 行为片段 | 已纳入仓库 | 复制 |
| `web-fetch/` | 抓取网页、PDF 并转换为 Markdown | 已纳入仓库 | 复制并 `npm install` |
| `web-search/` | Google Custom Search API 搜索 | 已纳入仓库 | 复制并配置凭据 |

仓库中的 `interactive-subagents/` 和 `observational-memory/` 只有说明性 README，它们不是当前实际加载的本地扩展。真正的 interactive subagents 由第 3 节的 Pi 包提供；observational memory 当前没有安装。

### 4.1 复制扩展源码

在新设备上：

```bash
PI_DIR="${PI_CODING_AGENT_DIR:-$HOME/.pi/agent}"
REPO="/path/to/pi-config"

mkdir -p "$PI_DIR/extensions"

cp "$REPO/extensions/ask-user-question.ts" "$PI_DIR/extensions/"
cp "$REPO/extensions/custom-header.ts" "$PI_DIR/extensions/"
cp -R "$REPO/extensions/bash-guard" "$PI_DIR/extensions/"
cp -R "$REPO/extensions/browser" "$PI_DIR/extensions/"
cp -R "$REPO/extensions/prompt-snippets" "$PI_DIR/extensions/"
cp -R "$REPO/extensions/web-fetch" "$PI_DIR/extensions/"
cp -R "$REPO/extensions/web-search" "$PI_DIR/extensions/"
```

`herdr-agent-state.ts` 不在当前仓库中。如果新设备也使用 Herdr，从旧设备通过安全渠道复制：

```bash
cp ~/.pi/agent/extensions/herdr-agent-state.ts \
  "$PI_DIR/extensions/herdr-agent-state.ts"
```

不需要复制任何 `node_modules`。扩展目录中的 `package-lock.json` 应保留，依赖通过 `npm ci` 重建。

### 4.2 安装扩展依赖

```bash
cd "$PI_DIR/extensions/bash-guard" && npm ci
cd "$PI_DIR/extensions/browser" && npm ci
cd "$PI_DIR/extensions/web-fetch" && npm ci
```

browser 扩展还需要下载 Chromium：

```bash
cd "$PI_DIR/extensions/browser"
npx playwright install chromium
```

### 4.3 bash-guard

当前行为：

- 默认对危险 bash 命令弹出确认窗口。
- 高风险显示 `🛑`，中风险显示 `⚠️`。
- 弹窗标题、风险原因和命令标签已中文化。
- `Run`、`Abort`、风险等级和命令本身保持英文。
- 非交互模式默认拒绝风险命令。
- 通过 `/bash-guard` 在交互式防护和禁用模式之间切换。
- `--bash-guard-auto-allow` 允许无 UI 模式放行普通风险命令。
- `--bash-guard-disabled` 启动时禁用防护，但灾难性操作仍保留硬拦截。

常用验证：

```text
/bash-guard
```

注意：已退役的旧版 `rtk.ts` 不应恢复。`~/.pi/agent/extensions/rtk.ts.bak` 和 `extensions/.retired/rtk.ts` 只是备份，不是活动扩展。

### 4.4 browser

browser 默认关闭，按会话开启：

```text
/browser
/browser on
/browser off
```

启用后提供：

- `browser_goto`
- `browser_eval`
- `browser_console`
- `browser_network`
- `browser_fill`
- `browser_click`
- `browser_screenshot`
- `browser_close`

浏览器持久化 profile 位于：

```text
~/.pi/agent/extensions/browser/.profile
```

该目录可能包含 cookie、localStorage、IndexedDB 和登录状态。新设备默认重新登录；只有在确认需要迁移登录态时，才通过安全渠道复制整个 `.profile`。

可选环境变量：

```bash
PI_BROWSER_HEADFUL=1
PI_BROWSER_PROFILE=/path/to/temporary-profile
```

### 4.5 prompt-snippets

快捷键或命令：

```text
Alt+S
/snippets
```

当前片段文件：

- `ask-questions.md`
- `delegate-exploration.md`
- `diagnose-report.md`
- `orchestrator-mode.md`
- `session-kickoff.md`
- `verify-not-assume.md`

片段每次发送消息后会重置为关闭状态。

### 4.6 web-search

扩展从以下位置读取 Google Custom Search 凭据：

```text
~/.pi/agent/extensions/web-search/auth.json
```

也支持环境变量：

```bash
GOOGLE_SEARCH_API_KEY=...
GOOGLE_CSE_ID=...
```

当前仓库只有 `auth.example.json`，没有真实 `auth.json`。新设备配置时：

```bash
cd "$PI_DIR/extensions/web-search"
cp auth.example.json auth.json
chmod 600 auth.json
# 编辑 auth.json，填入真实 google_search_api_key 和 google_cse_id
```

不要把真实 `auth.json` 提交到 Git。

### 4.7 web-fetch

`web-fetch` 的依赖由 `package.json` 管理，包含 Readability、Linkedom、Turndown 和 PDF 解析库。安装依赖后即可使用 `web_fetch`。

## 5. 技能

### 5.1 当前 `~/.pi/agent/skills`

当前本地 Pi 技能目录实际有 3 个技能：

| 技能 | 用途 |
|---|---|
| `agentic-search` | 深度网页研究、来源抓取、引用和多步研究流程 |
| `analyze-sessions` | 统计 session 成本、分析 prompt 模式、检索历史会话 |
| `auto-skill-installer` | 检测缺少的能力并安装匹配技能 |

复制方式：

```bash
PI_DIR="${PI_CODING_AGENT_DIR:-$HOME/.pi/agent}"
REPO="/path/to/pi-config"
mkdir -p "$PI_DIR/skills"
cp -R "$REPO/skills/analyze-sessions" "$PI_DIR/skills/"
```

`agentic-search` 和 `auto-skill-installer` 当前来自本机技能目录，不在本仓库中，需要从旧设备安全复制或按其来源重新安装。

### 5.2 `~/.agents/skills`

本机另外安装了 Matt Pocock 技能集合，共 21 个：

```text
ask-matt
code-review
codebase-design
diagnosing-bugs
domain-modeling
grill-me
grill-with-docs
grilling
handoff
implement
improve-codebase-architecture
prototype
setup-matt-pocock-skills
teach
to-spec
to-tickets
triage
wait-what
wayfinder
wizard
writing-for-agents
```

来源和重装命令：

```bash
npx skills@latest add mattpocock/skills
```

当前没有 `~/.pi/agent/AGENTS.md`。如果新设备需要统一的 agent 工作规范，应单独创建或维护该文件；不要把它误认为已有配置。

### 5.3 仓库中可选但当前未启用的技能

仓库还包含以下技能，当前没有复制到 `~/.pi/agent/skills/`：

- `pdf-reader`：需要 Python 虚拟环境和 `requirements.txt`。
- `web-debug`：配合 browser 扩展的前端调试流程。
- `youtube-transcript`：需要 Python、`yt-dlp` 和 `ffmpeg`。

如果部署这些技能：

```bash
cp -R "$REPO/skills/pdf-reader" "$PI_DIR/skills/"
python3 -m venv "$PI_DIR/skills/pdf-reader/.venv"
"$PI_DIR/skills/pdf-reader/.venv/bin/pip" install \
  -r "$PI_DIR/skills/pdf-reader/requirements.txt"
```

## 6. 外部依赖

当前机器的主要版本和路径：

| 程序 | 版本 | 当前路径 | 用途 |
|---|---:|---|---|
| Pi | `0.85.1` | `~/.local/bin/pi` | 主程序 |
| Node.js | `v22.22.3` | `~/.local/bin/node` | Pi 和扩展运行时 |
| npm | `10.9.8` | `~/.local/bin/npm` | 安装扩展依赖 |
| uv | `0.9.22` | `~/.local/bin/uv` | 安装 Headroom |
| RTK | `0.49.0` | `/opt/homebrew/bin/rtk` | bash 命令改写 |
| Headroom | `0.37.0` | `~/.local/bin/headroom` | 本地上下文压缩代理 |
| Homebrew | `7.0.2` | `/opt/homebrew/bin/brew` | 安装 RTK 等工具 |
| ffmpeg | 已安装 | `/opt/homebrew/bin/ffmpeg` | YouTube transcript 的可选依赖 |
| yt-dlp | 未安装 | - | YouTube transcript 依赖 |
| Docker | daemon 不可用 | `/usr/local/bin/docker` | 当前方案不使用 |

Node.js 建议使用 `>=22.19`，因为部分扩展和 hashline 方案要求该版本。新设备先确认：

```bash
node --version
npm --version
pi --version
```

### 6.1 Headroom

安装后端：

```bash
uv tool install "headroom-ai[proxy]"
```

当前 Headroom 配置：`~/.pi/agent/headroom/settings.json`

```json
{
  "enabled": true,
  "baseUrl": "http://127.0.0.1:8788",
  "autoStart": true,
  "command": "/Users/xckj/.local/bin/headroom",
  "mode": "normal",
  "minContextTokens": 20000,
  "minMessageChars": 2000,
  "timeoutMs": 30000
}
```

新设备必须把 `command` 改成新设备上的实际路径：

```bash
command -v headroom
```

当前配置让 `noheadroom` 在 Pi 启动时自动启动本机代理。健康检查：

```bash
curl -fsS http://127.0.0.1:8788/health
```

预期返回包含：

```text
"status":"healthy"
"ready":true
```

会话内命令：

```text
/headroom
/headroom health
/headroom on
/headroom off
/headroom stats
```

Headroom 默认只连接 `127.0.0.1`，请求不会发往远程 Headroom 服务。

### 6.2 RTK

安装：

```bash
brew install rtk
rtk --version
```

Pi 扩展配置：`~/.pi/agent/extensions/pi-rtk-optimizer/config.json`

当前关键设置：

```json
{
  "enabled": true,
  "mode": "rewrite",
  "guardWhenRtkMissing": true,
  "showRewriteNotifications": true,
  "outputCompaction": {
    "enabled": true,
    "stripAnsi": true,
    "readCompaction": { "enabled": false },
    "truncate": { "enabled": true, "maxChars": 12000 },
    "sourceCodeFilteringEnabled": false,
    "preserveExactSkillReads": false,
    "smartTruncate": { "enabled": false, "maxLines": 220 },
    "aggregateTestOutput": true,
    "filterBuildOutput": true,
    "compactGitOutput": true,
    "aggregateLinterOutput": true,
    "groupSearchOutput": true,
    "trackSavings": true
  }
}
```

只保留 `pi-rtk-optimizer` 这一份 Pi RTK 改写入口。旧版 `extensions/rtk.ts` 已退役，不要在新设备恢复它，否则会造成重复改写或绕过 optimizer 的输出压缩。

### 6.3 可选的 YouTube/PDF 依赖

只有部署对应技能时才需要：

```bash
brew install yt-dlp ffmpeg
```

## 7. 其他配置文件与运行状态

### 7.1 `pi-context-view`

文件：`~/.pi/agent/extensions/pi-context-view.json`

当前只配置 TUI 各类颜色，不包含模型或凭据。可以直接从旧设备复制：

```bash
cp ~/.pi/agent/extensions/pi-context-view.json \
  "$PI_DIR/extensions/pi-context-view.json"
```

### 7.2 模型和凭据

| 文件 | 当前情况 | 迁移建议 |
|---|---|---|
| `~/.pi/agent/models.json` | 4 个自定义 provider，文件权限 `600`，包含 `apiKey` | 通过安全渠道迁移或在新设备重建，绝不提交 Git |
| `~/.pi/agent/auth.json` | 当前为空对象 `{}`，权限 `600` | 默认不必迁移，按需重新认证 |
| `~/.pi/agent/models-store.json` | 当前为空对象 `{}` | 默认不必迁移 |
| `~/.pi/agent/trust.json` | 当前只信任旧设备项目路径 `/Users/xckj/project/cra` | 不要直接复制，迁移后重新信任新路径 |

当前 `models.json` 中的 provider ID 和模型名称如下；完整 endpoint 和 key 不写入本文：

| Provider ID | API 类型 | 模型 |
|---|---|---|
| `vulcanapi` | `openai-responses` | `gpt-5.5`、`gpt-6-astra` |
| `jaycesky1818` | `openai-responses` | `gpt-5.5`、`gpt-6-astra` |
| `apijustwokericu` | `openai-completions` | `gpt-5.6-sol` |
| `vulcanapi-copy` | `openai-responses` | `deepseek-v4.1-flash`、`deepseek-v4.1-flash-expires-on-0910`、`glm-5.3`、`kimi-k3` |

在新设备恢复敏感文件后设置权限：

```bash
chmod 600 ~/.pi/agent/models.json
chmod 600 ~/.pi/agent/auth.json
```

### 7.3 Sessions

`~/.pi/agent/sessions/` 保存历史会话、压缩数据和部分工具状态。它不是新设备启动所必需的，默认不迁移。

只有需要继续查看旧对话时才迁移，并注意：

- 文件可能很大。
- 可能包含源代码、命令输出、路径和敏感信息。
- 复制后应保持原有目录结构。
- 不要把整个 sessions 目录提交到公开 Git 仓库。

### 7.4 浏览器 profile

`~/.pi/agent/extensions/browser/.profile` 可能包含网页登录状态。默认重新登录，只有明确需要保持登录态时才迁移。

## 8. 推荐的新设备部署顺序

### 第一步：安装基础环境

确认以下命令可用：

```bash
node --version       # >= 22.19
npm --version
pi --version
brew --version       # macOS
uv --version
```

安装外部工具：

```bash
brew install rtk
uv tool install "headroom-ai[proxy]"
```

### 第二步：恢复核心设置和 Pi 包

```bash
mkdir -p ~/.pi/agent
# 恢复 settings.json，或执行第 3 节的 7 条 pi install 命令
pi list
```

### 第三步：复制本仓库扩展

执行第 4.1 节的复制命令，然后安装依赖：

```bash
PI_DIR="${PI_CODING_AGENT_DIR:-$HOME/.pi/agent}"

cd "$PI_DIR/extensions/bash-guard" && npm ci
cd "$PI_DIR/extensions/browser" && npm ci
npx playwright install chromium
cd "$PI_DIR/extensions/web-fetch" && npm ci
```

### 第四步：恢复扩展配置

至少恢复或建立：

```text
~/.pi/agent/extensions/pi-context-view.json
~/.pi/agent/extensions/pi-rtk-optimizer/config.json
~/.pi/agent/headroom/settings.json
```

把 `headroom/settings.json` 中的 `command` 改为：

```bash
command -v headroom
```

按需配置：

```text
~/.pi/agent/extensions/web-search/auth.json
```

### 第五步：恢复技能

复制仓库中需要的技能，并重新安装 `~/.agents/skills` 下的技能集合：

```bash
npx skills@latest add mattpocock/skills
```

`agentic-search`、`analyze-sessions` 和 `auto-skill-installer` 需要从旧设备或对应源恢复。

### 第六步：安全恢复模型配置

在确认新设备安全后，恢复 `models.json` 或手动创建 provider。恢复后执行：

```bash
chmod 600 ~/.pi/agent/models.json
pi --version
```

不要复制旧的 `trust.json`，在新设备首次进入项目时重新确认信任。

### 第七步：启动并验证

启动 Pi 后执行：

```text
/reload
```

然后验证：

```text
/context
/cache stats
/headroom health
/bash-guard
/browser
```

终端中验证：

```bash
pi list
rtk --version
curl -fsS http://127.0.0.1:8788/health
```

## 9. 日常维护

更新 Pi 包：

```bash
pi update
pi list
```

修改仓库中的扩展后同步到本机，再在 Pi 中执行：

```text
/reload
```

扩展依赖发生变化时重新安装：

```bash
cd ~/.pi/agent/extensions/bash-guard && npm ci
cd ~/.pi/agent/extensions/browser && npm ci
cd ~/.pi/agent/extensions/web-fetch && npm ci
```

排查常见问题：

| 现象 | 检查项 |
|---|---|
| Headroom 不工作 | `headroom --version`、`command -v headroom`、8788 端口和 `/headroom health` |
| RTK 不改写 | `rtk --version`、optimizer 的 `enabled`、`pi list` |
| browser 工具不可见 | 先运行 `/browser on`，确认 Chromium 已安装 |
| web-search 没有结果 | `auth.json` 或 `GOOGLE_SEARCH_API_KEY`/`GOOGLE_CSE_ID` |
| bash-guard 无弹窗 | 是否处于无 UI 模式、是否启用了 disabled flag |
| subagent 无法启动 | 是否在 tmux、cmux、zellij、Herdr 或 WezTerm 中运行 Pi |
| 模型不可用 | 检查 `models.json` 的 provider、endpoint、key 和文件权限 |

## 10. 不要迁移的内容

除非有明确需求，否则新设备不要复制：

- `~/.pi/agent/sessions/`
- `~/.pi/agent/extensions/browser/.profile`
- `~/.pi/agent/extensions/rtk.ts.bak`
- `~/.pi/agent/extensions/.retired/`
- 各扩展目录下的 `node_modules/`
- 旧设备的 `trust.json`
- 未脱敏的 `models.json` 到公开仓库
- 真实的 `web-search/auth.json` 到公开仓库

当前配置的核心原则是：Pi 包由 `settings.json` 管理，自定义扩展由本仓库管理，外部程序由系统包管理器管理，凭据和运行状态单独保护。
