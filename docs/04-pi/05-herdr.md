# herdr 工作区

## 本章解决什么

讲清楚 herdr 是什么、为什么它能让你「一个屏幕并行多个 agent」、以及最实用的一个场景：**查看另一个面板（pane）在跑什么**。这是本套工作流里最容易解决实际痛点的部分。

## 一句话说清楚

**herdr 是「终端工作区管理器」：它把多个终端组织成工作区（workspace）、标签页（tab）、面板（pane），并能识别每个面板里跑的是哪个 AI agent。**

> 记住口诀：**Pi 和 Oh My Pi 是「干活的 agent」，herdr 是「管窗口的调度员」。** 它不在乎里面跑的是 `pi` 还是 `omp`，它只负责排列窗口、查看状态、派发命令。

## 为什么需要 herdr

没有 herdr 时，你想同时干两件事，只能手动开两个终端窗口来回切；更麻烦的是——**你没法一眼看到「另一个 agent 干到哪了」**。有了 herdr：

- 一个屏幕左右分栏，左边一个 agent、右边一个 agent，互不干扰。
- 每个面板里跑的 agent 会被识别出来，并报告状态：**idle**（空闲）、**working**（干活中）、**blocked**（卡在等你确认）。
- 可以用命令行查看任意面板的**实时屏幕内容**、派发命令、等待输出。

## 核心概念

| 概念 | 说明 | 例子 |
| --- | --- | --- |
| **workspace** | 工作区，最顶层 | `w1` |
| **tab** | 工作区里的标签页 | `w1:t1` |
| **pane** | 标签页里的面板（左右/上下分栏的最小单元） | `w1:p1`、`w1:p4` |
| **agent 状态** | herdr 识别 pane 里的 agent 并归类 | `idle` / `working` / `blocked` / `done` / `unknown` |

> 💡 这些 ID 是 herdr 的公开编号：`w1:p1` 就是「工作区 1 的第 1 个面板」。本机当前就在 herdr 里跑（环境变量 `HERDR_ENV=1`，当前面板 `HERDR_PANE_ID` 可查）。

## 最实用场景：查看另一个面板在跑什么

这是 herdr 对团队协作最有价值的功能。三步：

```bash
# 1. 列出当前工作区所有面板：拿到 pane_id、agent 类型、状态、会话路径
herdr pane list

# 2. 查看某个面板的元数据（状态、标题、聚焦等）
herdr pane get <pane_id>

# 3. 直接读取该面板的实时终端输出（滚动缓冲）
herdr pane read <pane_id>
```

> ⚠️ **重要教训**：不要默认「别的面板也是 pi」。不同面板可能跑不同 agent（`pi`、`omp` 等），会话文件路径也各不相同（`~/.pi/agent/sessions/` vs `~/.omp/agent/sessions/`）。**以 `pane list` 返回的 `agent_session.value`（即会话文件路径）为准。**
>
> 💡 本仓库根目录的 `AGENTS.md` 已把「查看其他面板一律用 herdr」固化为项目约定，任何新会话都会自动遵守。

## 常用 pane 命令

| 命令 | 作用 |
| --- | --- |
| `herdr pane list` | 列出所有面板 |
| `herdr pane current --current` | 显示当前面板 |
| `herdr pane get <pane_id>` | 查看单个面板 |
| `herdr pane read <pane_id>` | 读取面板终端输出 |
| `herdr pane split --current --direction right --cwd "$PWD"` | 在右侧分出一个新面板 |
| `herdr pane run <pane_id> <命令>` | 在指定面板里运行命令 |
| `herdr pane send-text <pane_id> <文本>` | 向面板发送文本 |
| `herdr pane send-keys <pane_id> <按键>` | 向面板发送按键 |
| `herdr pane close <pane_id>` | 关闭面板 |

## 其他命令组

| 命令组 | 作用 |
| --- | --- |
| `herdr workspace` | 工作区管理（`workspace list` 等） |
| `herdr tab` | 标签页管理 |
| `herdr agent` | 识别/控制面板里的 agent（`agent list`） |
| `herdr worktree` | Git worktree 助手 |
| `herdr terminal` | 终端管理 |
| `herdr notification` | 通知管理 |

> 🤖 herdr 的 CLI 以本机安装版本为准：先 `herdr --help` 看总览，再 `herdr pane`、`herdr agent` 等看各组子命令。`herdr --skill` 会输出一份专门给 AI agent 看的完整说明书。

## 在 herdr 里并行开 agent

想在旁边再开一个 agent 一起干：

1. 用一个空闲的 shell 面板（停在命令行提示符、前台没有别的程序）。
2. 在右侧分一个新面板：`herdr pane split --current --direction right --cwd "$PWD" --no-focus`。
3. 在该面板里启动 agent（例如 `pi` 或 `omp`），或直接 `herdr pane run <新面板id> pi`。

> ⚠️ 面板 ID、agent 名、工作目录这些，务必以 herdr 命令返回的 JSON 为准，不要凭感觉猜。

## 下一步

三个工具都讲完了，最后看 [第 6 章：三者配合与选型](06-cooperation.md)，把 Copilot / Hermes / Pi / OMP / herdr 串成一张完整的选型地图。

---

← 上一章：[Oh My Pi](04-oh-my-pi.md) ｜ 下一章：[三者配合与选型](06-cooperation.md)
