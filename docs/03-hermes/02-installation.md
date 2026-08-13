# 安装 Hermes

## 本章解决什么

本章帮你在 Windows 电脑上安装 Hermes Agent。这里把“安装路径”和“谁来陪你操作”分开讲：

| 路线 | 适合你如果…… | 你需要做什么 |
| --- | --- | --- |
| 🥇 **让 Copilot 陪跑安装** | 你已经有 VS Code Copilot，但不熟悉命令行 | 把本章 Prompt 发给 Copilot，让它按你的选择一步步解释 |
| 🥈 **WSL2 安装** | 你愿意在 Windows 里的 Linux 环境使用 Hermes | 安装 WSL2 + Ubuntu，然后运行官方 `install.sh` |
| 🥉 **Windows 桌面 / 原生安装** | 你更想用普通 Windows 软件体验，或不想维护 WSL | 下载 Hermes Desktop，或在 PowerShell 运行官方 `install.ps1` |

> 🎯 本章面向没有编程背景的同事。每一步都会说明“为什么这么做”和“看到什么说明成功”。

> ⚠️ **发布前团队补充项：** 官方安装命令已核验，但团队可能有统一模型供应商、代理、镜像源、API Key 发放方式或指定桌面安装包。请技术负责人在发布前补充这些内部信息。

---

## 🥇 方式一：让 Copilot 陪你安装（首选）

### 核心思路

你已经有一个 AI 助手 Copilot，就让它帮你读本章、解释每一步、提醒你哪里需要自己决定。

### 第 1 步：打开 Copilot

打开 VS Code，按 `Ctrl+Alt+I`（Windows）打开 Copilot Chat 面板。

也可以按 `Ctrl+Shift+P`，输入 `Chat: Open Chat`，再按回车。

### 第 2 步：复制这个 Prompt 给 Copilot

```text
请陪我安装 Hermes Agent。我没有命令行经验，请用适合非程序员的方式一步步带我做。

请先问我三个问题：
1. 我是否愿意使用 WSL2（Windows 里的 Linux 环境）？
2. 我是否更希望使用 Windows 桌面应用或原生 PowerShell 安装？
3. 团队是否已经给了我模型账号、API Key、代理或安装包链接？

然后请根据我的回答选择安装方式：
- 如果我愿意用 WSL2：先确认 WSL2 和 Ubuntu 是否安装成功，再按 Hermes 官方 WSL2/Linux 安装命令执行。
- 如果我不想用 WSL2：优先让我下载 Hermes Desktop；如果没有安装包，再使用 Windows PowerShell 官方 install.ps1。

要求：
- 每一步告诉我在哪里点击、复制哪条命令、看到什么说明成功。
- 如果是在 VS Code 中操作，请优先让我通过 WSL 扩展或 Remote - SSH 连接本地 WSL，再在 VS Code 终端中执行。
- 涉及 API Key、密码、安全弹窗、是否覆盖文件时，明确标注“必须由我决定”。
- 不要让我把 API Key 发给你；只指导我在 Hermes 自己的 setup/model 向导里输入。
- 安装完成后，带我运行 hermes --version、hermes doctor 和 hermes。
```

### 第 3 步：跟着 Copilot 走，但这些事必须你亲自判断

| 决策环节 | 为什么必须你来定 | 提示 |
| --- | --- | --- |
| ⚠️ 选择 WSL2、桌面应用还是原生 PowerShell | 取决于你的电脑环境和使用习惯 | 不确定就选桌面应用；需要 Linux 工具再选 WSL2 |
| ⚠️ 输入 Windows 管理员确认 / 安全弹窗 | 这是系统权限确认 | 只在官方来源或团队指定安装包上点“是” |
| ⚠️ 设置 Linux 用户名和密码 | 这是你自己的 WSL 身份 | 小写英文名，密码自己保存 |
| ⚠️ 输入模型 API Key / OAuth 登录 | 这是你的账号权限 | 不要发给 Copilot，不要写在聊天里 |
| ⚠️ 是否处理正式项目文件 | 文件风险由你承担 | 第一次先用测试目录 |

---

## 🥈 方式二：WSL2 命令行安装

WSL2 可以理解成“Windows 里的 Linux 小电脑”。如果你以后要做较多自动化、脚本或开发相关任务，WSL2 是稳定选择。

> 💡 本章只讲“装上并跑通”的最小路径；想装得更顺手（国内加速、其他发行版、文件互访、备份等），见补充指南 [WSL 保姆级补充指南](appendix-wsl-guide.md)。
>
> 💡 **日常小 tips：把 VS Code 连接到本地 WSL。** 如果你已经在用 VS Code 和 Copilot，建议先在 VS Code 里安装 **WSL** 扩展；如果团队环境是通过 SSH 进入本机 WSL，也可以使用 **Remote - SSH**。连接成功后，VS Code 的文件区、终端和 Copilot Chat 都会运行在同一个 WSL 项目环境里：安装 Hermes、启动 dashboard、排查报错都可以在同一个窗口完成。

### 第 1 步：启用 WSL2

1. 打开 Windows **开始菜单**，搜索 `PowerShell`。
2. 右键 **Windows PowerShell**，选择 **以管理员身份运行**。
3. ⚠️ 如果弹出安全确认，点击 **是**。
4. 优先安装 Ubuntu：

```powershell
wsl --install -d Ubuntu
```

如果这条命令提示系统不支持 `-d Ubuntu`，再退回通用命令：

```powershell
wsl --install
```

安装完成后，按提示重启电脑。

### 第 1.5 步：第一次打开 Ubuntu

重启后，请按这个顺序检查：

1. 打开 Windows **开始菜单**，搜索 `Ubuntu`。
2. 第一次启动会出现一个黑色终端窗口，并提示 `Installing, this may take a few minutes...`。
3. 等它提示创建用户后，再继续下一步。
4. 如果没有自动出现 Ubuntu，可以打开 **Microsoft Store** 搜索 `Ubuntu`，安装官方 Ubuntu 应用后再启动。

> ⚠️ 如果公司电脑有 IT 管控，WSL、虚拟化或 Microsoft Store 可能被限制。这种情况不要强行绕过，联系技术负责人或 IT。

### 第 2 步：创建 Linux 用户名和密码

重启后，如果 Ubuntu 窗口要求输入用户名和密码：

1. 在 `Enter new UNIX username:` 后输入小写英文用户名，例如 `xiaoming`。
2. 在 `New password:` 后输入密码。屏幕不显示字符是正常的。
3. 在 `Retype new password:` 后再输一次。

> ⚠️ 这个密码以后执行 `sudo` 时会用到，请自己保存。

### 第 3 步：确认 WSL 正常工作

在 Ubuntu 窗口中输入：

```bash
whoami
```

如果显示你刚才设置的用户名，说明 WSL 可用。

建议再确认是 WSL2。在 PowerShell 中输入：

```powershell
wsl --list --verbose
```

看到 Ubuntu 的 `VERSION` 是 `2`，说明是 WSL2。

如果 `VERSION` 是 `1`，可以在 PowerShell 中执行：

```powershell
wsl --set-version Ubuntu 2
```

### 第 3.5 步：更新 Ubuntu 基础工具

回到 Ubuntu 窗口，先更新基础工具：

```bash
sudo apt update
sudo apt install -y curl git
```

这里会要求输入刚才创建的 Linux 密码。输入时屏幕不显示字符是正常的。

### 第 3.6 步：在 VS Code 中连接 WSL

如果你已经安装 VS Code，推荐这样操作：

1. 在 VS Code 扩展市场安装 **WSL** 扩展；如果你的团队使用 SSH 方式连接本地 WSL，则安装 **Remote - SSH**。
2. 按 `Ctrl+Shift+P`，输入 `WSL: Connect to WSL`，连接 Ubuntu。
3. 连接成功后，左下角会出现类似 `WSL: Ubuntu` 的绿色远程标识。
4. 打开 VS Code 的终端，此时终端路径应该是 Linux 风格，例如 `/home/你的用户名/...`。
5. 按 `Ctrl+Alt+I` 打开 Copilot Chat，把本章“Copilot 陪跑安装 Prompt”发给它，让它陪你完成后续安装。

> 🤖 这一步是日常最省心的做法：Copilot 在 VS Code 里解释命令，终端在 WSL 里执行命令，Hermes 也安装在同一个 WSL 环境里。

### 第 4 步：安装 Hermes

在 Ubuntu 窗口或 VS Code 的 WSL 终端中运行 Hermes 官方安装命令：

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

安装结束后，让当前窗口加载新的 PATH：

```bash
source ~/.bashrc
```

然后验证：

```bash
hermes --version
hermes doctor
```

如果 `hermes --version` 能输出版本号，说明命令已经安装成功。

### 第 5 步：首次配置模型

启动 Hermes：

```bash
hermes
```

首次使用通常需要配置模型供应商和账号。你也可以单独运行：

```bash
hermes setup
# 或者只配置模型：
hermes model
```

> ⚠️ API Key 或 OAuth 登录只在 Hermes 的配置向导里输入，不要粘贴给 Copilot 或发到群里。

### WSL2 文件路径怎么写

Windows 的：

```text
C:\Users\xiaoming\Desktop\Project
```

在 WSL2 里写成：

```text
/mnt/c/Users/xiaoming/Desktop/Project
```

> 💡 如果是长期开发项目，官方建议放在 WSL 自己的 Linux 文件系统里，例如 `~/projects/demo`，速度和权限更稳定。只有需要 Windows 软件直接打开的文件，才放在 `/mnt/c/...`。

---

## 💡 R 的日常工作流：VS Code + WSL + Copilot + Hermes dashboard

如果你已经习惯 VS Code，可以把 Hermes 的安装、启用、升级和异常处理都放在 VS Code 里完成。

| 场景 | 推荐做法 | 为什么方便 |
| --- | --- | --- |
| **安装** | VS Code 连接本地 WSL 后，让 Copilot 陪你执行本章 WSL 安装流程 | Copilot 解释命令，WSL 终端直接执行，不需要在多个窗口之间来回复制 |
| **启用** | 在 VS Code 的 WSL 终端运行 `hermes dashboard` | dashboard 是图形化入口，终端日志和项目文件仍在 VS Code 里可见 |
| **升级源码版 Hermes** | 在 Copilot 中创建一个专门的 Hermes 升级 Agent / 升级助手 | 它负责检查仓库状态、拉取更新、运行验证和记录异常；避免在当前 Hermes 会话里手动更新自己 |
| **异常处理** | 把报错、日志和检查结果交给 Copilot 继续分析 | 安装失败、PATH 问题、依赖缺失、dashboard 启动异常都可以在同一个 VS Code 工作区里补充修改和复查 |

给 Copilot 的升级助手 Prompt 可以这样写：

```text
你是我的 Hermes 源码版升级助手。请在 VS Code 的 WSL 环境中协助我升级 Hermes。

要求：
1. 先检查当前目录、git 状态、当前分支和远端，不要直接覆盖本地修改。
2. 如果有未提交修改，先提醒我决定是否保留、提交或暂存。
3. 按 Hermes 官方源码升级方式拉取更新，并运行必要的安装/依赖/健康检查。
4. 升级后运行 hermes --version 和 hermes doctor。
5. 如果 dashboard、依赖、PATH 或权限出现异常，请给出检查命令和修复建议。
6. 不要要求我把 API Key、Token 或密码发给你。
```

> ⚠️ 这是“源码版用户”的进阶工作流。如果你使用的是团队提供的 Desktop 安装包或普通安装器，升级方式应以团队发布说明为准。

---

## 🥉 方式三：Windows 桌面 / 原生安装

如果你不想维护 WSL2，可以使用 Windows 原生方式。

### 选择 A：Hermes Desktop 安装器（更适合非技术同事）

1. 打开 Hermes Agent 官网：

```text
https://hermes-agent.nousresearch.com/
```

1. 下载 Hermes Desktop 的 Windows 安装器。
2. 双击安装器，按提示安装。
3. 第一次启动时，桌面应用会在后台完成 Hermes CLI、Python、Node、Git 等依赖引导。
4. 按界面提示配置模型账号或运行 setup。

> ⚠️ 本手册不写死 `.exe` 文件名，因为安装包名称可能随版本变化。请以官网或团队技术负责人提供的下载链接为准。

### 选择 B：Windows PowerShell 官方命令

如果没有桌面安装器，或者技术负责人要求使用命令行安装，打开普通 PowerShell（不需要管理员），运行：

```powershell
iex (irm https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.ps1)
```

安装完成后，关闭并重新打开 PowerShell，再运行：

```powershell
hermes --version
hermes doctor
hermes
```

Windows 原生安装的数据目录通常在：

```text
%LOCALAPPDATA%\hermes
```

### Windows 原生 vs WSL2 怎么选

| 你在意什么 | 推荐 |
| --- | --- |
| 不想碰 Linux、希望像普通软件一样使用 | Windows Desktop / 原生 PowerShell |
| 想用 dashboard 的嵌入式终端、POSIX 工具或 Linux 开发环境 | WSL2 |
| 主要是聊天、文件整理、Gateway、Cron、浏览器工具 | Windows 原生通常就够 |
| 已经有 WSL2 项目和 Linux 工具链 | WSL2 更顺手 |

---

## 三种路径对比

| | 🥇 Copilot 陪跑 | 🥈 WSL2 手动 | 🥉 Windows 桌面 / 原生 |
| --- | --- | --- | --- |
| 适合人群 | 所有同事 | 愿意使用 Linux 环境的人 | 不想维护 WSL 的同事 |
| 安装难度 | 低，AI 陪你解释 | 中，需要命令行 | 低到中，取决于是否有桌面安装包 |
| 核心命令 | 由 Copilot 引导 | `install.sh` | Desktop 或 `install.ps1` |
| 后续使用 | 取决于最终选择的安装方式 | Ubuntu/WSL/VS Code 终端 | PowerShell 或桌面应用 |
| 文件路径 | Copilot 帮你转换 | `/mnt/c/...` 或 `~/projects` | `C:\Users\...` |

---

## 常见安装问题

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| `hermes: command not found` | PATH 尚未刷新 | WSL 运行 `source ~/.bashrc`；Windows 关闭并重开 PowerShell |
| `curl` 提示命令不存在 | WSL 系统缺少 curl | 运行 `sudo apt update && sudo apt install curl -y` |
| `wsl --install` 不可用 | Windows 版本较旧或公司策略限制 | 更新 Windows 10/11，或联系 IT/技术负责人 |
| 找不到 Ubuntu | Ubuntu 未安装或首次启动未完成 | 开始菜单搜索 Ubuntu；必要时从 Microsoft Store 安装官方 Ubuntu |
| VS Code 没有连到 WSL | 未安装 WSL 扩展，或仍在本机 Windows 窗口 | 安装 WSL 扩展，执行 `WSL: Connect to WSL`，确认左下角显示 `WSL: Ubuntu` |
| 输入 Linux 密码时看不到字符 | Linux 正常安全行为 | 直接盲打密码后回车 |
| API Key 配置失败 | 复制不完整、供应商不对或网络问题 | 重新运行 `hermes model`，向技术负责人确认供应商和密钥 |
| `hermes dashboard` 启动失败 | 端口占用、依赖缺失或 PATH 未刷新 | 先运行 `hermes doctor`，把错误信息交给 Copilot 辅助排查 |
| Windows 安装后旧 PowerShell 找不到 hermes | User PATH 未在旧窗口刷新 | 关闭窗口，重新打开 PowerShell |

## 安装完成后的最小验证

无论哪种安装方式，最后都建议跑这三步：

```bash
hermes --version
hermes doctor
hermes
```

- `hermes --version`：确认命令存在。
- `hermes doctor`：检查依赖和配置问题。
- `hermes`：进入 Hermes 对话界面。

如果你准备使用 dashboard，再额外验证：

```bash
hermes dashboard
```

看到浏览器打开本地管理页面，或终端显示 dashboard 服务地址，说明 dashboard 入口可用。

## 官方参考

- 安装指南: <https://hermes-agent.nousresearch.com/docs/getting-started/installation>
- Windows 原生指南: <https://hermes-agent.nousresearch.com/docs/user-guide/windows-native>
- Windows WSL2 指南: <https://hermes-agent.nousresearch.com/docs/user-guide/windows-wsl-quickstart>
- 平台支持: <https://hermes-agent.nousresearch.com/docs/getting-started/platform-support>

---

← 上一章：[Hermes 是什么](01-what-is-hermes.md) ｜ 下一章：[使用 Hermes](03-cli-tui-gateway.md)
