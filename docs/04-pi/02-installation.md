# 安装 Pi

## 本章解决什么

带你在一台电脑上把 Pi 装好、登录模型、跑通第一句话。全程尽量少碰命令行，适合无编程基础的同事。

## 前置条件

| 条件 | 说明 | 检查方法 |
| --- | --- | --- |
| 一台 Mac 或 Linux（Windows 见下文） | Pi 原生跑在终端里 | — |
| **Node.js** 环境 | Pi 用 npm 安装（官方安装脚本会自动处理） | 让 Copilot 帮你确认，或运行 `node -v` |
| 一个模型账号或 API Key | 决定 Pi 用哪个「大脑」 | 见下文「登录模型」 |

> 🤖 前置环境的检查与安装，可以直接把本节内容发给 Copilot，让它陪你一步步做。
>
> ⚠️ **Windows 用户**：Pi 官方支持 Windows，但建议配合 WSL（Windows Subsystem for Linux）使用体验更好；具体以官方平台说明为准。

## 安装方式

### 方式一：官方安装脚本（推荐）

```bash
curl -fsSL https://pi.dev/install.sh | sh
```

装完后新开一个终端，运行 `pi --version`，能看到版本号（本机实测为 `0.84.1`）即成功。

### 方式二：npm 全局安装

```bash
npm install -g --ignore-scripts @earendil-works/pi-coding-agent
```

> 💡 `--ignore-scripts` 表示安装时跳过依赖附带的一些自动脚本；Pi 的正常安装用不到它们。

### 方式三：团队统一分发

> ⚠️ 如果团队使用统一镜像、代理或内网安装包，请以技术负责人提供的安装方式为准，并补充到此处。

## 登录模型（让 Pi 找到「大脑」）

Pi 本身不带模型，需要接入一个模型提供方。两种方式：

### 方式 A：用已有订阅（最简单）

启动 Pi 后输入 `/login`，按提示选择提供方（如 Anthropic、OpenAI、GitHub Copilot 等），走浏览器授权即可。

```bash
pi
/login   # 选择 provider，完成授权
```

### 方式 B：用 API Key

```bash
export ANTHROPIC_API_KEY=sk-ant-...   # 示例：换成你实际的 key
pi
```

> ⚠️ **API Key 等同于账号密码**：不要贴进聊天、不要写进会被提交到 Git 的文件里、不要发给任何 AI 保管。配置方式以团队规范为准。

## 切换模型

Pi 支持大量提供方和模型。装好后：

| 操作 | 做法 |
| --- | --- |
| 换模型 | 在 Pi 里输入 `/model` 选择；或快捷键 `Ctrl+L` |
| 列出可用模型 | 命令行运行 `pi --list-models` |
| 刷新模型目录 | 运行 `pi update --models` |

## 验证安装

按顺序做三件事：

1. 终端运行 `pi --version`，确认能打印版本号。
2. 运行 `pi -p "用一句话介绍你自己"`，确认能收到回复（`-p` 表示跑完就退出）。
3. 运行 `pi`，进入交互界面，输入 `/model` 选一个模型，再随便聊一句。

三步都通过，安装就完成了。

## 常见问题

| 现象 | 可能原因 | 处理 |
| --- | --- | --- |
| `pi: command not found` | 安装后没新开终端，或 PATH 没刷新 | 新开终端再试；或把安装脚本提示的 PATH 加上 |
| 能启动但回复报错 | 没登录模型 / API Key 配错 | 重新 `/login` 或检查环境变量 |
| 想用公司代理/镜像 | 网络环境限制 | ⚠️ 找技术负责人要团队配置 |
| 想彻底卸载 | — | 用 `npm uninstall -g @earendil-works/pi-coding-agent`（若用脚本安装则按脚本说明清理） |

> 🤖 报错信息看不懂时，直接把整段报错发给 Copilot 或 Pi 自己，让它解释并给命令。

## 下一步

装好并登录后，进入 [第 3 章：使用 Pi Agent](03-pi-agent.md)，学会日常操作。

---

← 上一章：[为什么需要 Pi](01-why-pi.md) ｜ 下一章：[使用 Pi Agent](03-pi-agent.md)
