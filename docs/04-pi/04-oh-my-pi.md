# Oh My Pi（OMP）

## 本章解决什么

讲清楚 Oh My Pi 是什么、它和 Pi 是什么关系、比 Pi 多了哪些能力，以及什么时候该用 `omp` 而不是 `pi`。

## 一句话说清楚

**Oh My Pi（简称 OMP，命令名 `omp`）是 Pi 的「功能加强版」。**

它站在 Pi 同一套底座上，把很多常用能力（子代理、待办清单、浏览器、Python、代码智能等）**预先装好**了，开箱即用。你不需要理解它和 Pi 内部差多少，只需要记住一句话：

> **Pi 是「极简版」，Oh My Pi 是「全装版」。** 想轻、想自己搭，用 Pi；想开箱即用一堆现成能力，用 OMP。

## 和 Pi 的关系（怎么看出来的）

OMP 和 Pi 明显同源，几个证据：

> 💡 这一节是「怎么看出两者同源」的技术细节，只想上手用的人可以直接跳过。

| 证据 | 说明 |
| --- | --- |
| 命令风格一致 | `omp "消息"`、`omp -p`、`omp -c`、`omp -r` 等用法与 `pi` 完全对应 |
| 环境变量同源 | OMP 复用了 `PI_*` 系列变量（如 `PI_SMOL_MODEL`、`PI_SLOW_MODEL`） |
| 目录结构镜像 | Pi 用 `~/.pi/agent`，OMP 用 `~/.omp/agent`（会话、配置、扩展目录一应俱全） |
| 工具集是超集 | Pi 只有读/写/改/跑四个基础工具，OMP 默认带一整套 |

> 💡 两者是**两个独立的可执行程序**，配置和会话**不共享**：`pi` 的会话在 `~/.pi/agent/sessions/`，`omp` 的在 `~/.omp/agent/sessions/`。别搞混了。

## OMP 比 Pi 多了什么

OMP 默认启用的工具更全：

| 工具 | 作用 |
| --- | --- |
| `read` / `bash` / `edit` / `write` | 基础文件与命令操作（与 Pi 默认一致） |
| `grep` / `glob` | 文件内容与文件名搜索（Pi 额外内置 `grep`/`find`/`ls`，需手动启用；OMP 默认启用 `grep`+`glob`） |
| `lsp` | 代码智能：定义跳转、类型检查、诊断 |
| `python` | 在本地跑 Python（需 `omp setup python` 安装依赖） |
| `notebook` | 编辑 Jupyter notebook |
| `inspect_image` | 用视觉模型分析图片 |
| `browser` | 浏览器自动化（Puppeteer） |
| `computer` | 桌面截图与输入（默认关闭） |
| `task` | 启动子代理并行处理多个任务 |
| `todo` | 维护待办清单 |
| `web_search` | 联网搜索 |
| `ask` | 向用户提问（交互模式） |

此外还有一批子命令和特性：`omp setup`（初始化）、`omp models`（列/搜模型）、`omp plugin`（插件管理）、`omp commit`（生成提交信息）、`omp share`、`omp update` 等。

## 安装与初始化

> ⚠️ 团队内 OMP 的分发方式以技术负责人为准（统一安装包、镜像或脚本）。本章给出通用流程，具体命令请向负责人确认后补充。

通用流程：

1. 安装 `omp` 到本地（本机实测版本 `17.2.15`，位于 `~/.local/bin/omp`）。
2. 运行 `omp setup` 完成初始化（含登录模型、可选装 Python 等依赖）。
3. 运行 `omp --version` 验证。
4. 运行 `omp` 进入交互界面，或 `omp "消息"` 直接带任务启动。

> 🤖 安装、初始化遇到报错时，把整段输出发给 Copilot 或 OMP 自己，让它解释并给下一步命令。

## 常用用法速查

| 场景 | 命令 |
| --- | --- |
| 交互模式 | `omp` |
| 带任务启动 | `omp "列出 src/ 下的文件"` |
| 一条命令出结果 | `omp -p "总结这个仓库"` |
| 引用文件 | `omp @prompt.md "回答这个"` |
| 继续上次会话 | `omp -c` |
| 指定模型（模糊匹配） | `omp --model opus "重构这段代码"` |
| 导出会话为 HTML | `omp --export <session.jsonl>` |
| 导入 Claude Code 会话 | `omp --from-claude` |

## 什么时候用 OMP、什么时候用 Pi

| 你的需求 | 选 Pi 还是 OMP |
| --- | --- |
| 只想轻量地在终端里读文件、改代码、跑命令 | Pi |
| 想要开箱即用的子代理、待办、LSP、浏览器、Python | OMP |
| 想自己搭工作流、尽量少依赖 | Pi |
| 要处理「需要多个子任务并行」的大活 | OMP（`task` 子代理） |
| 团队已有统一配置好的 OMP | OMP |

> 💡 两者不冲突，可以都装。日常轻活用 `pi`，重活用 `omp`。

## 下一步

无论用 Pi 还是 OMP，都可以放进 herdr 里分栏并行。[第 5 章：herdr 工作区](05-herdr.md) 教你一个屏幕管多个 agent。

---

← 上一章：[使用 Pi Agent](03-pi-agent.md) ｜ 下一章：[herdr 工作区](05-herdr.md)
