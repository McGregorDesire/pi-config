# my-lean-pi-setup 对照清单与本机安装配置指南

> 对照来源：<https://github.com/McGregorDesire/my-lean-pi-setup/blob/main/README.zh-CN.md>
> 本机快照时间：2026-09-16 · Pi `0.85.1` · macOS arm64

本文件做三件事：

1. 列出 README 中每个组件的**本机真实安装状态**（已装 / 未装 / 用别的方案替代）；
2. 给出**每个组件完整的安装、配置、验证、卸载方法**；
3. 标出**冲突与风险点**（重复注册工具、双份 RTK 改写等）。

---

## 一、结论速览

README 共列出 11 类组件。本机情况：**5 项已装、6 项未装**（其中 4 项已由等价方案覆盖）。

| # | README 组件 | 状态 | 本机实际实现 |
|---|---|---|---|
| 1 | `billion-context-pi-lean` | ✅ 已装 | `git:github.com/kunkun9527/billion-context-pi-lean` (`0.1.69-lean.1`) |
| 2 | `pi-slim` | ✅ 已装 | `npm:pi-slim` (`0.2.1`) |
| 3 | Headroom / noheadroom | ✅ 已装 | `npm:@raquezha/noheadroom` (`0.3.2`) + `headroom` CLI (`0.37.0`)，代理健康 |
| 4 | RTK + `pi-rtk-optimizer` | ✅ 已装 | `rtk 0.49.0`（Homebrew）+ `npm:pi-rtk-optimizer` (`0.9.0`) |
| 5 | `pi-context-view` | ✅ 已装 | `npm:pi-context-view` (`0.5.2`) |
| 6 | `pi-subagents-lean` | ❌ 未装 | 已用 **`pi-interactive-subagents` 3.7.2**（另一种上游实现）替代 |
| 7 | `pi-web-access-lean` | ❌ 未装 | 已用**本地扩展** `web-search/` + `web-fetch/` 替代 |
| 8 | `pi-hashline-edit-pro-lean` | ❌ 未装 | 无替代，仍用 Pi 内置 `read`/`edit` |
| 9 | `rpiv-ask-user-question-lean` | ❌ 未装 | 已用**本地扩展** `extensions/ask-user-question.ts` 替代 |
| 10 | `rpiv-todo-lean` | ❌ 未装 | 无替代（当前无 `todo` 工具） |
| 11 | 精选 `AGENTS.md` 模板 | ❌ 未装 | `~/.pi/agent/AGENTS.md` 不存在 |
| — | 前置：mattpocock/skills | ✅ 已装 | `~/.agents/skills/`（21 个技能） |

### 本机 `~/.pi/agent/settings.json` 现状

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

> 注意：`pi-cache-graph` 不在 README 清单里，是本机额外安装的观测扩展。

---

## 二、本机环境基线（安装前置条件）

| 工具 | 本机版本 / 路径 | 说明 |
|---|---|---|
| Pi | `0.85.1` | `pi --version` |
| Node.js | `v22.22.3` · `~/.local/bin/node` | `pi-slim` 要求 ≥20.6，`pi-context-view`/hashline 要求 ≥22.19 |
| npm | `10.9.8` · `~/.local/bin/npm` | `pi install npm:*` 需要 |
| uv | `0.9.22` · `~/.local/bin/uv` | 安装 `headroom-ai` 用 |
| Homebrew | `/opt/homebrew/bin/brew` | 安装 `rtk`、`ffmpeg` 用 |
| rtk | `0.49.0` · `/opt/homebrew/bin/rtk` | `pi-rtk-optimizer` 要求 ≥0.23.0 |
| headroom | `0.37.0` · `~/.local/bin/headroom`（uv tool `headroom-ai`） | 代理需监听 `127.0.0.1:8788` |
| Python | 系统 `3.9.6`；另有 `~/.local/bin/python3.13` | ⚠️ 新版 Headroom 需 Python ≥3.10，故走 uv 独立环境 |
| ffmpeg | `/opt/homebrew/bin/ffmpeg` | `youtube-transcript` 依赖 |
| yt-dlp | ❌ 未安装 | `youtube-transcript` 需要 |
| Docker | `/usr/local/bin/docker`（daemon 不可用） | 本机**不用** Docker 后端方案 |
| `skills` CLI | ❌ 未全局安装 | 用 `npx skills@latest ...` 调用 |

---

## 三、已安装组件（含安装与配置方法）

### 1. `billion-context-pi-lean` ✅

**作用**：长会话历史压缩与按需召回；提供 `compress` 与 `acp_context` 两个工具。

```bash
# 安装（本机采用的方式）
pi install git:github.com/kunkun9527/billion-context-pi-lean

# 或本地克隆安装
git clone https://github.com/kunkun9527/billion-context-pi-lean.git
cd billion-context-pi-lean && npm install && pi install ./
```

- **依赖**：`billion-context-pi@0.1.69`、`typebox@1.3.7`（随包自动安装，勿手动升级）
- **配置**：无需配置文件；ACP 状态存于会话目录的 `*.jsonl.acp.json`
- **验证**：
  ```bash
  pi list | grep billion-context
  # 会话内：调用 acp_context({"op":"acp_status","args":{"scope":"uncompressed"}})
  ```
- **卸载**：`pi remove git:github.com/kunkun9527/billion-context-pi-lean`
- **⚠️ 禁止**与其它 `billion-context-pi` 扩展同时加载（会重复注册 ACP 工具与 hooks）

---

### 2. `pi-slim` ✅

**作用**：把 Pi 默认注入的文档说明块改为按需启用，净省约 309 tokens。新增 `/pi` 命令临时恢复原始文档模式。

```bash
pi install npm:pi-slim
```

- **配置**：无配置文件
- **用法**：日常直接提问（精简版 system prompt）；需要查 Pi 文档时用 `/pi 如何编写扩展命令？`
- **验证**：`pi list | grep pi-slim`；用 `/context injections` 对比 base prompt 是否已移除 Pi documentation 块
- **卸载**：`pi remove npm:pi-slim`

---

### 3. Headroom / `noheadroom` ✅

**作用**：运行期把超大工具输出（尤其 `read`）压缩后再进上下文。上游 Headroom 默认保护 `read`/`bash` 不压缩，`noheadroom` 做了 Pi 适配绕过该限制（`read` 走 `smart_crusher`，节省 60–90%）。

#### 3.1 安装 Headroom 后端（本机方式：uv，无需 Docker）

```bash
uv tool install "headroom-ai[proxy]"     # 本机已装，headroom-ai v0.37.0
headroom --version                        # → headroom, version 0.37.0
```

> 若用 pip：`pip install "headroom-ai[proxy]"`（需 Python ≥3.10；本机系统 Python 为 3.9.6，故用 uv 更稳）。
> `uv tool list` 应显示 `headroom-ai v0.37.0`。

#### 3.2 启动代理

```bash
# 前台启动（默认端口 8787，本项目需要 8788）
headroom proxy --port 8788

# 健康检查
curl -fsS http://127.0.0.1:8788/health
# → {"service":"headroom-proxy","status":"healthy","ready":true,...}
```

本机 `noheadroom` 配置为 `"autoStart": true`，Pi 启动时会自动拉起代理，一般无需手动执行。

#### 3.3 安装 Pi 侧桥接扩展

```bash
pi install npm:@raquezha/noheadroom
```

#### 3.4 配置文件 `~/.pi/agent/headroom/settings.json`

本机实际内容：

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

| 字段 | 说明 |
|---|---|
| `baseUrl` | 代理地址，仅允许 localhost；远程需 `PI_HEADROOM_ALLOW_REMOTE=1` |
| `autoStart` | `true` 时自动启动本地代理；用 Docker 后端应设 `false` |
| `command` | headroom 可执行文件绝对路径（PATH 不稳时必填） |
| `mode` | `normal` / `quiet` / `silent`，也可用 `PI_HEADROOM_MODE` |
| `minContextTokens` | 上下文达到该规模才触发压缩 |
| `minMessageChars` | 单条消息字符数阈值 |

#### 3.5 会话内命令

| 命令 | 作用 |
|---|---|
| `/headroom` | 查看状态与会话统计 |
| `/headroom on` \| `off` | 实时开关压缩 |
| `/headroom health` | 检查/启动代理 |
| `/headroom stats` | 查看后端原始指标 |

- **验证**：`curl -fsS http://127.0.0.1:8788/health` 返回 `ready: true`；会话内 `/headroom health`
- **卸载**：`pi remove npm:@raquezha/noheadroom`；后端 `uv tool uninstall headroom-ai`
- **隐私**：默认仅发往 `127.0.0.1`，不离开本机

---

### 4. RTK + `pi-rtk-optimizer` ✅

**作用**：RTK 把常用 shell 命令改写成低 token 输出形式；`pi-rtk-optimizer` 在 Pi 内自动改写 `bash` 命令并压缩工具输出。

#### 4.1 安装 RTK

```bash
brew install rtk        # 本机已装 rtk 0.49.0（homebrew-core）
rtk --version           # → rtk 0.49.0
```

#### 4.2 安装 Pi 扩展

```bash
pi install npm:pi-rtk-optimizer
```

> 替代安装方式：`pi install git:github.com/MasuRii/pi-rtk-optimizer`

#### 4.3 配置文件 `~/.pi/agent/extensions/pi-rtk-optimizer/config.json`

本机实际内容：

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
    "sourceCodeFiltering": "none",
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

| 字段 | 默认 | 说明 |
|---|---|---|
| `mode` | `"rewrite"` | `rewrite`=自动改写；`suggest`=只提示不改写 |
| `guardWhenRtkMissing` | `true` | rtk 缺失时原样执行命令 |
| `readCompaction.enabled` | `false` | 有损压缩 `read` 输出，**默认关闭**以保持代码精确 |
| `sourceCodeFiltering` | `"none"` | `none` / `minimal` / `aggressive` |
| `truncate.maxChars` | `12000` | 硬截断上限（1000–200000） |
| `smartTruncate.maxLines` | `220` | 智能行截断（40–4000） |

#### 4.4 会话内命令

`/rtk`（设置面板）、`/rtk show`、`/rtk path`、`/rtk verify`、`/rtk stats`、`/rtk clear-stats`、`/rtk reset`、`/rtk help`

- **验证**：`rtk --version`；会话内 `/rtk verify` 与 `/rtk show`
- **注意**：0.6.0+ 已移除 `rewriteGitGithub` 等分类开关，改写策略统一由 `rtk rewrite` 决定
- **编辑失败排查**：若出现 "old text does not match"，用 `/rtk` 临时关闭 read 压缩 → 重新 read → 编辑 → 再打开
- **卸载**：`pi remove npm:pi-rtk-optimizer`

---

### 5. `pi-context-view` ✅

**作用**：可视化上下文占用，并检查 system prompt、工具定义、扩展注入等"看不见的部分"。**不压缩任何内容**。

```bash
pi install npm:pi-context-view
```

| 命令 | 作用 |
|---|---|
| `/context` | 等价 `/context usage` |
| `/context usage` | 上下文占用可视化 |
| `/context injections` | 查看隐藏注入（含工具定义） |
| `/context config` | 生成 `~/.pi/agent/extensions/pi-context-view.json` |

- **配置**：本机 `~/.pi/agent/extensions/pi-context-view.json` 已自定义各类别配色（`systemPromptColor`、`toolOutputColor` 等）
- **验证**：会话内 `/context injections`，确认与 README 基准的 token 数接近
- **卸载**：`pi remove npm:pi-context-view`
- **备注**：README 基准用的是 `0.4.3`，本机为 `0.5.2`，数值可能有小幅差异

---

## 四、未安装组件（完整安装方法）

以下 6 项本机未安装。安装前请先看 **第八节冲突清单**，避免重复注册工具。

### 6. `pi-subagents-lean` ❌（已有替代方案）

**README 方案**：基于 `@tintinweb/pi-subagents@0.19.0`，单一 `subagent` 工具，268 tokens。

```bash
pi install npm:@ssk_dev/pi-subagents-lean
# 或
pi install git:github.com/kunkun9527/pi-subagents-lean
```

**本机现状**：已装 `pi-interactive-subagents 3.7.2`（上游 `HazAT/pi-interactive-subagents`，经 `McGregorDesire` fork 的 git 源）。二者是**不同上游**，工具集也不同：

| 项目 | `pi-interactive-subagents`（本机已装） | `pi-subagents-lean`（未装） |
|---|---|---|
| 工具 | `subagent`、`subagent_interrupt`、`subagents_list`、`subagent_resume` | 单一 `subagent` |
| 运行方式 | 终端复用器 pane（Herdr/cmux/tmux/zellij/WezTerm）异步运行 | 后台运行 + steering |
| 命令 | `/plan`、`/iterate`、`/subagent` | — |
| 上游依赖 | `HazAT/pi-interactive-subagents` | `@tintinweb/pi-subagents@0.19.0` |

**建议**：保留现有 `pi-interactive-subagents`。若改用 lean 版，必须先 `pi remove git:git@github.com:McGregorDesire/pi-interactive-subagents.git`，否则重复注册 `subagent` 工具。

**依赖**：`pi-interactive-subagents` 需在 Herdr/cmux/tmux/zellij/WezTerm 内启动 Pi：

```bash
tmux new -A -s pi 'pi'
# 或 zellij --session pi 后运行 pi
# 可选：export PI_SUBAGENT_MUX=herdr|cmux|tmux|zellij|wezterm
```

### 7. `pi-web-access-lean` ❌（已有替代方案）

**README 方案**：把 `web_search`/`source_check`/`fetch_content`/`get_search_content` 合并为单一 `web_access`（152 tokens），并支持本地 PDF。

```bash
pi install npm:@ssk_dev/pi-web-access-lean
# 或
pi install git:github.com/kunkun9527/pi-web-access-lean
```

**本机现状**：使用两个本地扩展替代：

| 路径 | 提供的工具 |
|---|---|
| `~/.pi/agent/extensions/web-search/index.ts` | `web_search`（需 `~/.pi/agent/extensions/web-search/auth.example.json` 复制为真实配置） |
| `~/.pi/agent/extensions/web-fetch/index.ts` | `web_fetch`（含 `node_modules`，已 `npm install`） |

**升级到 lean 版的步骤**（可选）：

```bash
# 1) 先移除本地重复实现
rm -rf ~/.pi/agent/extensions/web-search ~/.pi/agent/extensions/web-fetch
# 2) 安装 lean 版
pi install npm:@ssk_dev/pi-web-access-lean
# 3) 重启 pi 或 /reload
```

**lean 版用法**：`web_access` 的 `op` 支持 `search` / `check` / `fetch` / `get` / `help`

```json
{ "op": "search", "input": "Pi coding agent extensions" }
```

### 8. `pi-hashline-edit-pro-lean` ❌（无替代）

**README 方案**：基于 `pi-hashline-edit-pro@3.0.4`，用四字符 HASH 锚点做精确编辑与撤销，423 tokens。

```bash
pi install git:github.com/kunkun9527/pi-hashline-edit-pro-lean
```

**提供的工具**：

```text
read
replace
insert
undo_last_change
anchor_grep        # 已注册但默认关闭
```

- `/toggle-anchor-grep`：在内置 `grep` 与 `anchor_grep` 间切换
- `/toggle-auto-read`：控制写入后自动读取与编辑后 diff
- **必须**复制 `read`/`anchor_grep` 返回的四字符锚点，不可臆造
- `replace` 使用 `replacement_lines: string[]`：`[]` 删除所选范围，`[""]` 写入一个空行
- **要求**：Node.js ≥ 22.19.0（本机 v22.22.3 ✅）
- **⚠️ 影响面大**：会替换内置 `read`/`edit` 语义；升级后需**新建会话**才能拿到新 Schema
- **⚠️ 禁止**与其它 Hashline 包装同时加载
- **卸载**：`pi remove git:github.com/kunkun9527/pi-hashline-edit-pro-lean`

> 与 `pi-rtk-optimizer` 的交互：rtk 的 read compaction 需保持 `readCompaction.enabled: false`（本机已是），否则锚点可能被截断。

### 9. `rpiv-ask-user-question-lean` ❌（已有替代方案）

**README 方案**：结构化问卷提问工具，215 tokens。

```bash
pi install npm:@ssk_dev/rpiv-ask-user-question-lean
# 或
pi install git:github.com/kunkun9527/rpiv-ask-user-question-lean
```

**本机现状**：已有本地 `~/.pi/agent/extensions/ask-user-question.ts`，提供 `ask_user_question` 工具（功能等价：单选/多选、选项校验、推荐排序）。

**建议**：二者**不能共存**（重复注册同名工具）。若要换 lean 版：

```bash
mv ~/.pi/agent/extensions/ask-user-question.ts ~/.pi/agent/extensions/ask-user-question.ts.bak
pi install npm:@ssk_dev/rpiv-ask-user-question-lean
```

- 每次调用可提 1–4 个问题，每题 2–4 个选项
- **卸载**：`pi remove npm:@ssk_dev/rpiv-ask-user-question-lean`

### 10. `rpiv-todo-lean` ❌（无替代）

**README 方案**：任务拆解、依赖追踪、状态流转，246 tokens。

```bash
pi install npm:@ssk_dev/rpiv-todo-lean
# 或
pi install git:github.com/kunkun9527/rpiv-todo-lean
```

**工具**：`todo`，`action` 支持 `create` / `list` / `get` / `update` / `delete` / `clear`

**用法约定**：多步骤任务先建列表，始终只保留一个 `in_progress`，完成后置 `completed`。

**依赖**：`@juicesharp/rpiv-todo@2.10.1`、`@juicesharp/rpiv-i18n@2.10.1`（随包安装）
**⚠️ 禁止**与其它 `rpiv-todo` 包装同时加载
**卸载**：`pi remove npm:@ssk_dev/rpiv-todo-lean`

> 说明：README 与 AGENTS.md 模板中的 "使用 `todo` 推进多步骤任务" 依赖此扩展。本机未装，因此当前无 `todo` 工具。

### 11. 精选 `AGENTS.md` 模板 ❌

**作用**：结果先行、多步骤编号、需求对齐（配合 `grilling` skill）、"不写代码优先"的决策天梯。

**放置位置**（Pi 启动自动加载）：

- 全局：`~/.pi/agent/AGENTS.md`
- 项目级：项目根目录 `./AGENTS.md`（或上级目录）

**安装**（本机尚无该文件，直接下载即可）：

```bash
# 简体中文版
mkdir -p ~/.pi/agent
curl -fsSL https://raw.githubusercontent.com/McGregorDesire/my-lean-pi-setup/main/agents/zh-CN/AGENTS.md \
  -o ~/.pi/agent/AGENTS.md

# 英文版
curl -fsSL https://raw.githubusercontent.com/McGregorDesire/my-lean-pi-setup/main/agents/en/AGENTS.md \
  -o ~/.pi/agent/AGENTS.md
```

**验证**：`ls -l ~/.pi/agent/AGENTS.md`；会话内 `/context injections` 应能看到 instruction files 占位。

**⚠️ 必须改的部分**：模板中的 `Subagents Delegation` 章节默认写的是 `Explore` / `Plan` / `general-purpose`，与本机已装的 `pi-interactive-subagents`（`planner`/`scout`/`worker`/`reviewer`/`visual-tester`）**不匹配**，需自行替换。

**前置推荐**（本机已完成）：

```bash
npx skills@latest add mattpocock/skills
```

本机结果：`~/.agents/skills/` 下 21 个技能（`grill-me`、`grilling`、`codebase-design`、`diagnosing-bugs`、`domain-modeling`、`prototype`、`writing-for-agents`、`wizard` 等）。

---

## 五、本机额外已装（不在 README 清单中）

| 组件 | 版本 | 位置 | 说明 |
|---|---|---|---|
| `pi-cache-graph` | `1.0.2` | `~/.pi/agent/npm/node_modules/` | `/cache graph`、`/cache stats`、`/cache export`，观测缓存命中率 |
| `custom-header.ts` | — | `~/.pi/agent/extensions/` | 大号 Π 标题 |
| `bash-guard/` | — | `~/.pi/agent/extensions/` | 危险命令拦截，含 `--bash-guard-disabled` 等 flag |
| `browser/` | — | `~/.pi/agent/extensions/` | Playwright 无头 Chromium，默认关闭，`/browser on` 启用 |
| `prompt-snippets/` | — | `~/.pi/agent/extensions/` | 可复用行为规则片段 |
| `rtk.ts` | — | `~/.pi/agent/extensions/` | 轻量 RTK 改写扩展（与 `pi-rtk-optimizer` 功能重叠，见第八节） |
| `herdr-agent-state.ts` | — | `~/.pi/agent/extensions/` | Herdr 状态上报 |
| `agentic-search` | — | `~/.pi/agent/skills/` | 深度检索技能 |
| `analyze-sessions` | — | `~/.pi/agent/skills/` | 会话成本/提示词挖掘 |
| `auto-skill-installer` | — | `~/.pi/agent/skills/` | 自动补齐技能 |

---

## 六、推荐启用顺序（按 README 建议）

1. **先量基线** → `pi-context-view`：`/context` 看当前占用
2. **削静态 prompt** → `pi-slim`：`/pi` 按需恢复文档
3. **削命令输出** → RTK + `pi-rtk-optimizer`
4. **压活动上下文** → Headroom / `noheadroom`（先确认 `curl http://127.0.0.1:8788/health`）
5. **压历史会话** → `billion-context-pi-lean`
6. **替换精简工具** → 第 6–10 项，**一次只换一个**，换完用 `/context` 对比
7. **再量一次** → 与第 1 步对比

---

## 七、原始基准数据（README 实测）

| 组件 | 精简版 | 锁定原版 | 节省 | 降幅 |
|---|---:|---:|---:|---:|
| `billion-context-pi-lean` | 675 | 6,061 | 5,386 | 88.9% |
| `pi-subagents-lean` | 268 | 1,416 | 1,148 | 81.1% |
| `pi-web-access-lean` | 152 | 2,376 | 2,224 | 93.6% |
| `pi-hashline-edit-pro-lean` | 351 | 1,410 | 1,059 | 75.1% |
| `rpiv-ask-user-question-lean` | 215 | 1,258 | 1,043 | 82.9% |
| `rpiv-todo-lean` | 256 | 904 | 648 | 71.7% |
| **合计** | **1,917** | **13,425** | **11,508** | **85.7%** |

> 口径：Pi `0.84.4` + `pi-context-view 0.4.3` + 模型 `GPT-5.6-SOL`，token 按 `ceil(字符数 / 4)` 估算。

---

## 八、冲突与风险清单（重要）

### 8.1 双份 RTK 改写 → **结论：保留 `pi-rtk-optimizer`，退役 `rtk.ts`**

两个实现都监听 `tool_call` 并改写同一条 bash 命令，且**改写能力完全等价**——都把 `rtk rewrite` 当作唯一事实来源：

| | `~/.pi/agent/extensions/rtk.ts` | `npm:pi-rtk-optimizer` |
|---|---|---|
| 改写来源 | `pi.exec("rtk", ["rewrite", cmd])` | `resolveRtkRewrite` → 同一个 `rtk rewrite` |
| 接受退出码 | `0` / `3` | `0` / `3` |
| 无匹配 | `1` → 原样放行 | `1` → 原样放行 |
| 已改写命令 | `cmd.startsWith("rtk ")` | 剥离环境变量前缀后判 `rtk`（`splitLeadingEnvAssignments`） |
| 缺 rtk | 放行 + TUI 状态提示 | 放行 + 警告 |

两者对同一条命令产出**逐字节相同**的结果。`pi-rtk-optimizer` 额外具备：

- **输出压缩管线**（ANSI 剥离 / 测试聚合 / 构建过滤 / git 压缩 / lint 聚合 / 搜索分组 / 截断）——RTK 省 token 的主要来源，`rtk.ts` 完全没有
- `/rtk` TUI 配置面板、`/rtk stats`、节省量追踪
- `which`/`where` 显式解析 rtk 可执行文件（含缓存与警告）
- 退出码 `2`（`rtk denied rewrite`）与异常码的上报
- 会话级 `RTK_DB_PATH` 隔离：注入 `export RTK_DB_PATH='/tmp/pi-rtk-optimizer/history.db'; `
- Windows 兼容与管道安全改写
- 与 hashline 锚点兼容的 read 压缩

`rtk.ts` 的优势仅在于体积（~150 行、type-only import、`RTK_DISABLED=1` 支持）。

#### 实测：重复加载不会破坏命令，但有代价

```bash
rtk rewrite 'rtk git status'                            # exit=3，输出与输入相同 → 双方都判为未改变（幂等）
rtk rewrite "export RTK_DB_PATH='...'; rtk git status"  # exit=1，空输出 → rtk.ts 判为 no-match，放行
rtk rewrite 'FOO=1 git status'                          # exit=3 → FOO=1 rtk git status
rtk rewrite 'git status'                                # 单次进程开销实测 ≈ 22.6ms（10 次平均）
```

1. **不存在"改写两次导致命令损坏"**，原来的担心可以消除
2. 但每次 bash 调用多一个子进程；**不可改写**的命令（如 `npm test`，exit 1）两个扩展各起一次，白花 ≈45ms
3. **加载顺序会让 optimizer 的 DB 隔离静默失效**：若 `rtk.ts` 先注册，命令已变成 `rtk git status`，optimizer 走 `reason: "already_rtk"` 直接跳过 → **不注入 `RTK_DB_PATH`、不显示改写通知**

#### 处理

```bash
# 保留 pi-rtk-optimizer；改名而非删除（rtk.ts 未纳入本仓库 git 版本控制）
mv ~/.pi/agent/extensions/rtk.ts ~/.pi/agent/extensions/rtk.ts.bak

# 重启 pi 或 /reload 后确认
#   /rtk verify  → 应显示 rtk 路径
#   /rtk show    → mode=rewrite, outputCompaction.enabled=true
```

> 旁证：`rtk.ts` 头部注释称"Shared with Oh My Pi (OMP)"，但本机 OMP **未安装**（`omp`/`oh-my-pi` 均无，`~/.omp`、`~/.config/omp` 不存在），该保留理由不成立；其权限为 `600`、修改时间 `9月16 11:39`（其它扩展均为 `9月3`），也不在 `git ls-files` 中。

#### 反向选择的唯一场景

若坚持**极简 + 零损失过滤**（不要任何输出压缩/截断碰工具结果，只要命令改写），则反过来：

```bash
pi remove npm:pi-rtk-optimizer
```

此时 `rtk.ts` 的 150 行、type-only import、`RTK_DISABLED=1` 才成为优点。按本机现有保守配置（`readCompaction.enabled: false`、`sourceCodeFiltering: "none"`）判断，这条不适用。

### 8.2 重复注册同名工具

| 工具名 | 冲突双方 | 处理 |
|---|---|---|
| `ask_user_question` | 本地 `ask-user-question.ts` ↔ `rpiv-ask-user-question-lean` | 只能留一个 |
| `subagent` | `pi-interactive-subagents` ↔ `pi-subagents-lean` | 只能留一个 |
| `web_search` / `web_fetch` | 本地 `web-search`/`web-fetch` ↔ `pi-web-access-lean` | 只能留一个 |
| `billion-context-pi` 系列 | lean 版 ↔ 原版 | 只能留一个 |
| `read` / `replace` | 内置 `read`/`edit` ↔ `pi-hashline-edit-pro-lean` | 装 hashline 后语义整体切换，需新开会话 |

### 8.3 安全与隐私

- **切勿**把 API key / 内部端点提交到公开配置。`web-search/auth.example.json` 是模板，真实文件应被忽略
- Headroom 仅允许 localhost；`PI_HEADROOM_ALLOW_REMOTE=1` 会把会话内容发往远端，谨慎开启
- `bash-guard` 在非交互模式下需 `--bash-guard-auto-allow`，否则可能阻塞

### 8.4 版本升级后的检查

- `pi-rtk-optimizer` 的 peerDependencies 只声明到 `^0.80.0`，本机 Pi 为 `0.85.1`；升级后跑一次 `/rtk show` 确认无异常
- `pi-context-view` 版本（`0.5.2`）高于 README 基准（`0.4.3`），token 数会略有差异
- `billion-context-pi-lean` 锁定 `billion-context-pi@0.1.69`，不要手动升级该依赖

---

## 九、一键核对脚本

```bash
#!/usr/bin/env bash
# 用途：核对本机与 README 清单的差异
echo "== Pi 版本 ==" && pi --version

echo "== 已安装包 ==" && pi list

echo "== 二进制度 =="
for x in rtk headroom uv brew node ffmpeg yt-dlp; do
  printf '%-10s ' "$x"; command -v "$x" || echo "(缺失)"
done

echo "== 后端健康 =="
curl -fsS --max-time 3 http://127.0.0.1:8788/health || echo "Headroom 代理未响应"

echo "== 配置文件 =="
for f in ~/.pi/agent/settings.json \
         ~/.pi/agent/headroom/settings.json \
         ~/.pi/agent/extensions/pi-rtk-optimizer/config.json \
         ~/.pi/agent/extensions/pi-context-view.json \
         ~/.pi/agent/AGENTS.md; do
  [ -e "$f" ] && echo "✅ $f" || echo "❌ $f"
done

echo "== 本地扩展 =="
ls -1 ~/.pi/agent/extensions/
```

---

## 十、卸载与回滚速查

```bash
# Pi 包
pi remove npm:pi-cache-graph
pi remove npm:pi-context-view
pi remove npm:pi-slim
pi remove npm:pi-rtk-optimizer
pi remove npm:@raquezha/noheadroom
pi remove git:github.com/kunkun9527/billion-context-pi-lean
pi remove git:git@github.com:McGregorDesire/pi-interactive-subagents.git

# CLI 工具
brew uninstall rtk
uv tool uninstall headroom-ai

# 本地扩展（移除前建议 .bak 改名而非直接删）
cd ~/.pi/agent/extensions
for e in ask-user-question.ts custom-header.ts herdr-agent-state.ts rtk.ts \
         bash-guard browser prompt-snippets web-search web-fetch interactive-subagents; do
  [ -e "$e" ] && mv "$e" "$e.bak"
done

# 回滚配置：直接编辑 ~/.pi/agent/settings.json 删除对应 packages 项
```

---

**一句话总结**：README 的上下文优化主线（`pi-context-view` → `pi-slim` → RTK → Headroom → `billion-context-pi-lean`）本机**已全部装好并处于健康状态**；未装的 6 项集中在"精简工具替换"和"AGENTS.md 模板"，其中 3 项已有等价替代。装新项前务必先处理第 8 节的三处重复注册风险。
