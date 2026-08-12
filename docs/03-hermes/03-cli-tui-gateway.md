# 使用 Hermes

## 本章解决什么

Hermes 有多个入口：**Dashboard、桌面应用、命令行（CLI）、终端界面（TUI）和 Gateway（消息平台网关）**。它们不是同一件事，本章帮你选对入口。

> 💡 记不住命令时，可以问 Copilot：“我想让 Hermes 做某某任务，应该用什么命令？请先给我低风险检查命令，不要直接修改文件。”

## 五种入口怎么选

| 入口 | 适合你如果…… | 启动方式 |
| --- | --- | --- |
| **Dashboard** | 想在浏览器里管理配置、会话、API Key 和本地 Hermes 状态 | `hermes dashboard` |
| **桌面应用** | 喜欢图形界面，不想长期面对命令行 | 双击 Hermes Desktop，或运行 `hermes desktop` |
| **CLI** | 愿意复制命令，想快速进入对话或执行一次任务 | `hermes` 或 `hermes chat` |
| **TUI** | 想在终端里使用更现代的界面 | `hermes --tui` |
| **Gateway** | 想让 Hermes 接入 Telegram、Discord、Slack、WhatsApp 等消息平台 | `hermes gateway setup` / `hermes gateway run` |

> ⚠️ Gateway 不是“添加模型的地方”。模型供应商和 API Key 主要通过 `hermes model`、`hermes setup` 或 dashboard 配置；Gateway 主要负责消息平台接入。

---

## Dashboard（推荐给 VS Code + WSL 用户）

### 启动

如果 Hermes 安装在 WSL 中，推荐在 **VS Code 已连接 WSL** 的终端里运行：

```bash
hermes dashboard
```

正常情况下，Hermes 会启动本地 Web Dashboard，并在终端输出访问地址；浏览器也可能自动打开。

### 为什么适合日常使用

- 图形界面比纯命令行更直观。
- 配置、会话、API Key、状态检查都集中在一个页面。
- 终端日志仍留在 VS Code 中，出错时可以直接复制给 Copilot 辅助排查。
- 如果你在 WSL 中安装 Hermes，dashboard 也在同一个 WSL 环境里运行，不容易出现“装在 A 环境、运行在 B 环境”的混乱。

### 日常启动流程

1. 打开 VS Code。
2. 通过 WSL 扩展或 Remote - SSH 连接本地 WSL。
3. 打开 VS Code 终端，确认路径是 Linux 风格，例如 `/home/你的用户名/...`。
4. 运行：

```bash
hermes dashboard
```

5. 在浏览器里使用 dashboard；如果报错，把终端输出交给 Copilot 辅助分析。

> 💡 这是 R 的日常用法：安装、启用、异常处理都放在 VS Code 里完成。Copilot 负责解释和排查，Hermes dashboard 负责实际使用和配置。

---

## 桌面应用

### 启动

如果你用 Hermes Desktop 安装器安装，直接从开始菜单或桌面快捷方式打开。

如果你已经用命令行安装了 Hermes，也可以尝试运行：

```bash
hermes desktop
```

### 适合做什么

- 和 Hermes 对话，不想记命令。
- 查看会话、配置、文件和终端相关面板。
- 交给非技术同事试用。

### 使用提醒

- ⚠️ 第一次启动可能需要完成后端依赖安装或配置模型账号。
- ⚠️ 如果要求输入 API Key，只在 Hermes 的配置界面中输入，不要贴给 Copilot。
- ⚠️ 做批量文件操作前，先让 Hermes “只列计划和文件清单，不修改”。

---

## 命令行（CLI）

### 启动对话

最简单：

```bash
hermes
```

或显式进入 chat：

```bash
hermes chat
```

### 单次提问

如果只想问一次，不进入连续对话：

```bash
hermes chat -q "DDS 和 PNG 贴图格式有什么区别？"
```

### 常用命令速查

| 你要做什么 | 命令 | 说明 |
| --- | --- | --- |
| 启动 Hermes | `hermes` | 默认进入交互聊天 |
| 单次提问 | `hermes chat -q "问题"` | 适合快速问答或脚本调用 |
| 打开 dashboard | `hermes dashboard` | 浏览器管理界面 |
| 选择模型/输入 API Key | `hermes model` | 退出聊天后在终端运行 |
| 完整配置向导 | `hermes setup` | 初次配置或重新配置 |
| 检查环境 | `hermes doctor` | 安装后排错优先跑它 |
| 查看状态 | `hermes status` | 查看 agent、auth、platform 状态 |
| 管理工具 | `hermes tools` | 开关 Hermes 可用工具 |
| 管理技能 | `hermes skills list` | 注意是 `skills` 复数 |
| 打开 TUI | `hermes --tui` | 终端现代界面 |
| 查看帮助 | `hermes --help` | 命令总入口 |

### 给美术/建模同事的低风险示例

先检查，不要直接修改：

```bash
hermes chat -q "请检查 /mnt/c/Users/xiaoming/Desktop/export-test 目录，列出所有 PNG 和 FBX 文件，不要修改任何文件。"
```

让 Hermes 先给计划：

```bash
hermes chat -q "我想把测试目录里的 PNG 转成 JPG。请先给出执行计划、会处理的文件列表和风险点，不要实际转换。"
```

确认后再执行：

```bash
hermes chat -q "按刚才确认的计划，在测试目录中执行转换。保留原始 PNG，不要覆盖。"
```

> ⚠️ 不建议第一次就对正式项目目录运行批量修改。先复制一份测试目录。

---

## 终端界面（TUI）

### 启动

```bash
hermes --tui
```

也可以把 TUI 设为默认界面，但初学者先用命令启动即可。

### 它和 CLI 的关系

TUI 不是另一套 Agent。它和 CLI 使用同一个 Hermes 后端、同一套会话、同一套工具和 Slash Command，只是界面更现代：

- 有更清晰的状态栏。
- `/help`、`/model`、会话切换等会以更友好的面板出现。
- 适合长对话和观察工具调用过程。

### 常见操作

| 操作 | 方式 |
| --- | --- |
| 输入并发送消息 | 输入文字后按 `Enter` |
| 打开帮助 | `/help` |
| 切换模型 | `/model` |
| 查看用量 | `/usage` |
| 退出 | `/quit` 或终端关闭 |

> 💡 官方 TUI 启动命令是 `hermes --tui`，不是 `hermes tui`。

---

## Gateway（消息平台网关）

### Gateway 是什么

Gateway 是 Hermes 的“消息平台入口”。它可以把同一个 Hermes Agent 接到 Telegram、Discord、Slack、WhatsApp、Signal、Email 等平台。

通俗理解：

- CLI/TUI/桌面/dashboard：你在电脑上直接使用 Hermes。
- Gateway：你在聊天软件里和 Hermes 对话。

### 基础命令

首次配置：

```bash
hermes gateway setup
```

前台运行（适合调试）：

```bash
hermes gateway run
```

安装为后台服务后管理：

```bash
hermes gateway install
hermes gateway start
hermes gateway status
hermes gateway stop
```

> ⚠️ Gateway 需要平台 Token 或账号配置，例如 Telegram Bot Token、Discord Bot Token。这些属于敏感信息，只能按团队流程配置，不要发给 AI 聊天窗口。

### 什么时候需要 Gateway

| 场景 | 是否需要 Gateway |
| --- | --- |
| 只在自己电脑上用 Hermes | 不需要 |
| 想在 Telegram/Slack/Discord 里发消息给 Hermes | 需要 |
| 想让定时任务或 webhook 把结果发到群里 | 通常需要 |
| 只是想换模型 | 不需要，用 `hermes model` 或 dashboard |

---

## 🤖 vs ⚠️：使用 Hermes 时的分工

| 环节 | 谁来做 | 说明 |
| --- | --- | --- |
| 查命令、解释报错 | 🤖 Copilot 可帮忙 | 把非敏感错误信息发给 Copilot |
| 规划任务步骤 | 🤖 Hermes 可帮忙 | 先让它列计划，不要直接修改 |
| 启动 dashboard、查看日志、检查异常 | 🤖 Copilot 可陪跑 | 在 VS Code 终端里完成最方便 |
| 执行文件和终端操作 | 🤖 Hermes 执行 | 在工具权限范围内完成 |
| 决定目标、路径、输出格式 | ⚠️ 你决定 | 项目标准只有你知道 |
| 输入 API Key / Token | ⚠️ 你决定 | 只在 Hermes 配置向导、dashboard 或团队指定位置输入 |
| 确认批量修改结果 | ⚠️ 你检查 | 抽查文件，必要时用备份回滚 |

## 官方参考

- CLI 命令参考: <https://hermes-agent.nousresearch.com/docs/reference/cli-commands>
- TUI 文档: <https://hermes-agent.nousresearch.com/docs/user-guide/tui>
- Messaging Gateway 文档: <https://hermes-agent.nousresearch.com/docs/user-guide/messaging/>
- Web Dashboard 文档: <https://hermes-agent.nousresearch.com/docs/user-guide/features/web-dashboard>

---

← 上一章：[安装 Hermes](02-installation.md) ｜ 下一章：[记忆与技能系统](04-memory-and-skills.md)
