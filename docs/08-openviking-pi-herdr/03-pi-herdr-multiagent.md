# Pi + herdr 多 Agent 编排

## 本章解决什么

上一节装好了 OpenViking 的 Pi 集成，上一期（第 4 期）也讲过 [herdr 的 workspace / tab / pane 概念](../../04-pi/05-herdr.md) 和「开两个面板各跑一个 agent」。本章在此基础上往前一步：**不用手动开面板、手动盯状态，让父会话自己把任务拆开、并行派给多个子代理干活，干完自动把结果送回父会话合并**。读完你能判断什么时候该派多个 agent、怎么写派发指令，以及怎么保证「模型在哪个环境跑」和「要读/写的文件在哪」对得上。

## 一句话说清楚

**pi-herdr-agents 是 Pi 的社区多 Agent 编排包（Multi-Agent Orchestration Package）：它把每个子代理（Subagent）当成一个真实的 Pi 进程，跑在 herdr 的独立面板（pane）或受管 Git worktree 里，父会话非阻塞地派发、子代理干完自动回收结果。**

> 记住口诀：**父会话是「项目经理」，子代理是「临时工」**——父会话负责拆任务、派活、审查、合并；子代理只领一个边界明确的任务，干完交付就退出，不推送、不合并、不越权。

## 它是什么

pi-herdr-agents（npm 包名即 `pi-herdr-agents`）装在 Pi 里后，给父会话新增一套工具和命令：`subagent`（派发子代理）、`subagent_interrupt`（打断）、`subagent_cancel`（取消一个正在跑的普通子代理，取消后不再重试/恢复）、`subagent_send` / `subagent_stop`（给常驻专员续活/停机）、`worktree_list` / `worktree_remove`（受管 worktree 清单/清理）等。核心是前两个——把活派出去、把结果收回来。

它和上一期 herdr 的区别在于**自动化程度**：

| 维度 | 上一期手动 herdr | 本章 pi-herdr-agents |
| --- | --- | --- |
| 开面板 | 你手动 `herdr pane split` | 父会话一次调用自动开 |
| 派任务 | 你在面板里手动敲 | 父会话把任务文案传进去 |
| 盯状态 | 你 `herdr pane read` 去看 | 父会话输入框上方有实时小组件 |
| 收结果 | 你切过去复制 | 子代理干完，结果自动回到父会话 |
| 隔离写入 | 你自己 `git worktree add` | 一次调用自动建受管 worktree |

> 💡 一句话：**herdr 管窗口，pi-herdr-agents 管「派活—干活—交活」这条流水线。** 前提仍然是「在 herdr 里启动 Pi」（环境变量 `HERDR_ENV=1`），因为子代理的落点就是 herdr 的 pane。

## 子代理的两种落点

派发时给每个子代理选一个「干活的地方」。选错地方，轻则互相踩文件，重则读不到该读的东西。

### 落点一：herdr 普通 pane（读任务为主）

子代理跑在扩展自己管理的 `Agents` 标签页的一个 pane 里，和父会话共享同一个 Git 工作目录。默认每个标签页放 4 个 pane，满了自动开新标签页。

- **适合**：只读的侦查（scout）、审查（reviewer）、交互式讨论（planner）。
- **不适合**：多个会同时写文件的子代理——它们共用同一个工作目录和 Git 索引，互相覆盖。

```typescript
subagent({
  name: "Scout auth",
  agent: "scout",
  model: "<provider>/<fast-tier-id>",
  thinking: "low",
  task: "梳理认证流程，列出相关文件和风险点，不要改文件。",
});
```

### 落点二：受管 Git worktree（隔离写入）

派发时带上 `worktree` 参数，扩展会在 `~/.herdr/worktrees/` 下从指定的**已提交基准**（`base`，默认当前分支的已提交 `HEAD`）建一个独立的 Git worktree，子代理从那个目录启动、写入、可选地提交。

- **适合**：会写文件、会产生提交的子代理（worker）。
- **关键点**：一个并行写任务 = 一个独立 worktree；**父会话里没提交的改动不会被复制过去**，所以子代理要看的未提交内容得先提交，或写进任务文案里。干完后 worktree **保留**，供父会话审查，不会自动合并、推送或删除。

```typescript
subagent({
  name: "Ticket 123",
  agent: "worker",
  model: "<provider>/<mid-tier-id>",
  thinking: "medium",
  worktree: { branch: "ticket/123", base: "main" },
  task: "实现 ticket 123，跑通相关测试，提交并回报 commit SHA。不要 push、不要 merge、不要删 worktree。",
});
```

### 怎么选

| 子代理要干什么 | 落点 |
| --- | --- |
| 读代码、摸清现状、审查改动 | 普通 pane |
| 交互式讨论、你来我往地设计 | 普通 pane |
| 单个或顺序写文件（一前一后不打架） | 父会话工作目录即可 |
| 多个并行、各自独立写文件/提交 | 每个写任务一个受管 worktree |

> ⚠️ **worktree 是 Git 隔离，不是安全沙箱**：子代理和你同账号、同权限，worktree 只是把「工作目录 + 索引 + HEAD」分开，仓库对象库和大部分引用仍然共享。

## 派发与回收机制

### 非阻塞派发（fire-and-forget）

`subagent` 调用**立刻返回** `started`，父会话不用等、不用轮询、不用去 tail 会话文件——继续聊下一句或接着干别的。多个子代理并行跑，各自独立推进。父会话输入框上方会出现一个实时小组件：

```text
╭─ Subagents ──────────────────── 1 active · 2 open ─╮
│ 01:23  Scout: Auth (scout)             active · read 7m │
│ 00:45  Reviewer (reviewer)                   waiting 2m │
│ 00:12  Scout: DB (scout)                      starting… │
╰─────────────────────────────────────────────────────────╯
```

### 有界结果自动回收

子代理干完（成功、失败、或向你求助）后，父会话会自动收到一条**有界结果（bounded subagent_result）**并开启新一轮对话，把结果带进上下文。结果太大（超过约 16000 字符）时父上下文会做节略、保留开头和结尾，完整内容留在子会话文件里，需要时可去取。普通 pane 在结果送达后自动关闭；受管 worktree 则保留其根 shell 供后续审查。

### 分工：父会话协调、子代理叶子

| 谁 | 干什么 | 不干什么 |
| --- | --- | --- |
| **父会话（协调者，coordinator）** | 拆解任务、分配边界、给出目标/允许改的文件/验证方式/提交指令 | 不替子代理干活，但**拥有**审查、合并、最终验证 |
| **子代理（叶子，leaf）** | 领一个边界明确的结果，读代码、跑验证、写文件、按需提交 | 不递归派发、**不 push、不 merge、不切分支、不删 worktree** |

> 💡 一句话：**子代理交付 = 交接给你审查，不等于验收通过。** 拿到结果后，父会话还要看 diff、跑测试、决定怎么合并。

## 使用方式

### 安装与 /reload

在 herdr 里启动 Pi 后安装包，装完 `/reload`（或重启会话）让工具生效：

```bash
pi install npm:pi-herdr-agents
```

之后在 herdr 里 `/reload` 一下。装任何 Pi 包前先看一眼包源码。

> ⚠️ **从 2.x 升到 3.x 的注意**：`pi-herdr-agents` 3.0 起定位改为「执行宿主」，**不再随包内置任何角色**——2.x 的 scout / worker / reviewer / planner 等七个内置角色、`/plan` 命令和 orchestrate 技能全部拆到可选角色包。想要这些角色，装官方 `pi-herdr-roles` 包（含 scout / planner / worker / reviewer / adversarial-reviewer / visual-tester 六个通用角色、`/plan` 和 `/skill:orchestrate`），或把角色定义复制进项目的 `.pi/agents/` 自行维护。对 2.x 升级的完整迁移说明见官方 3.0.0 release notes。

### 自然语言派发（最省事）

直接说人话，让父会话自己拆、自己派：

```text
用两个 scout 并行梳理认证流程和会话模块，然后汇总它们的发现。
```

### /subagent 命令（指定角色）

按名字直接派一个角色，`/subagent list` 列出当前可用角色：

```text
/subagent scout 分析认证模块，报告相关文件和风险
/subagent worker 修复登录页的 off-by-one 错误并跑测试
```

常用角色（agent role）有：**scout**（侦察/梳理）、**worker**（实现边界任务）、**reviewer**（审查）、**planner**（规划，交互式）。

> ⚠️ 3.0 起角色不再随主包内置：主包只提供派发/回收机制（执行宿主），上面这些角色来自 `pi-herdr-roles` 角色包或项目自定义。角色是扩展点，可在项目 `.pi/agents/` 或全局 `~/.pi/agent/agents/` 里自定义。

### 相关技能

把「典型派发」固化成可复用的配方：

- **pi-herdr-agents（主包自带）**：操作指南技能——派发、监督、打断、取消、恢复、worktree、常驻专员、模型路由的完整操作说明，子代理按需加载。
- **orchestrate**（随 `pi-herdr-roles` 角色包提供）：有界审查——父会话先物化证据（仓库、base/head SHA、diff），并行派多个只读 reviewer，父会话综合结论。适合代码评审、PR 评审。

> 💡 这些技能本质都是「父会话协调 + 子代理并行取证 + 父会话综合」，只是把每一步的边界和产物契约写得更严。团队内部用自然语言派发即可，遇到评审再套技能。3.0 之前主包还曾内置 deep-research-campaign（多视角深度调研）技能，现已随角色拆包移除，需要深度调研工作流时自行定制或关注角色包生态。

## 模型路由与执行环境一致性

派发前最重要的一条检查：**确认子代理用的模型跑在哪个环境，和它要读/写的文件在不在同一台机器上。** 模型和文件系统不一致，会出现「模型看着正常，其实根本看不到目标文件」的隐性失败。

- 需要**读本机源码、写本机文件**的子代理 → 用**本机 herdr pane 模型**（在本机跑的模型）。
- 某些模型实际跑在**远程运行时（如 NAS 上的 Hermes）**，其工作目录是那台机器的（例如 `cwd=/root`），**看不到本机（Windows）的文件系统** → 不要派它去读/写本机文件。
- 这类远程模型只用于「远程那台机器本机可达、不需要本机文件系统」的任务。

> ⚠️ 一句话口诀：**派活前先对一遍「模型在哪跑」和「文件在哪存」，两者必须一致。** 不确定时，用本机模型做本机文件任务最稳。

模型选择上，`subagent` 的 `model` 参数接受精确的 `provider/model-id`（推荐显式指定），或按任务类别走 `task:recon` / `task:coding` / `task:review` 等短名（需要先在配置里写好候选）。先选「模型档位」（快档做机械活/侦查，中档做实现/审查，前沿档做架构/安全/疑难诊断），再在该模型支持的范围内选 thinking 档。

## 最小实战示例

场景：给仓库写一份「认证模块」技术总结，同时另一个人想修复两个互不干扰的 bug。拆成两个只读侦查 + 一个并行写任务。示例里的 `<...>` 是占位符，替换成你已认证的真实模型 ID：

**第 1 步：拆任务、派两个只读 scout（普通 pane，并行）**

```typescript
subagent({
  name: "Auth scout",
  agent: "scout",
  model: "<provider>/<fast-tier-id>",
  thinking: "low",
  task: "梳理认证模块：列出相关文件、调用链和风险点，不要改文件。",
});

subagent({
  name: "Session scout",
  agent: "scout",
  model: "<provider>/<fast-tier-id>",
  thinking: "low",
  task: "梳理会话模块：列出相关文件、调用链和风险点，不要改文件。",
});
```

两个调用立刻返回 `started`，各自在独立 pane 里跑。

**第 2 步：并行派两个独立写任务（各一个受管 worktree）**

```typescript
subagent({
  name: "API bug",
  agent: "worker",
  model: "<provider>/<mid-tier-id>",
  thinking: "medium",
  worktree: { branch: "bugs/api", base: "main" },
  task: "修复 API 层 bug，跑通测试，提交并回报 commit SHA。不要 push、不要 merge。",
});

subagent({
  name: "UI bug",
  agent: "worker",
  model: "<provider>/<mid-tier-id>",
  thinking: "medium",
  worktree: { branch: "bugs/ui", base: "main" },
  task: "修复 UI 层 bug，跑通测试，提交并回报 commit SHA。不要 push、不要 merge。",
});
```

**第 3 步：回收与合并（父会话做）**

每个子代理干完，结果自动回到父会话。父会话逐个审查：

```bash
# 用结果里给的 workspace id 和 worktree 路径核对
git -C <worktree-path> status --short
git -C <worktree-path> log --oneline <base-sha>..HEAD
git -C <worktree-path> diff <base-sha>...HEAD
```

确认无误后，父会话按仓库规范合并/发布，最后显式清理 worktree（`/worktree remove` 或 `worktree_remove`）。**合并、推送、清理都只由父会话来做，子代理不做。**

## 版本事实

以下为本机核验的版本，供参考；这类工具迭代很快，快速变化的功能（模型路由、面板布局、监督机制、清理规则等）以本机安装版本为准：

- Pi：1.2.0
- herdr：0.9.3
- pi-herdr-agents：3.2.1（另装 pi-herdr-roles 角色包提供通用角色）

核验时间：2026-10（Windows 原生环境）。

## 参考

- pi-herdr-agents 官方仓库（README、文档、技能、ADR）：<https://github.com/giuseppecrj/pi-herdr-agents>
- 受管 worktree 操作文档：<https://github.com/giuseppecrj/pi-herdr-agents/blob/main/docs/worktree-subagents.md>
- orchestrate 审查技能：<https://github.com/giuseppecrj/pi-herdr-agents/blob/main/skills/orchestrate/SKILL.md>
- Pi：<https://github.com/earendil-works/pi>
- herdr：<https://herdr.dev>

---

← 上一节：[安装与启用](02-installation.md) ｜ 下一节：[组合工作流](04-combo-workflow.md)
