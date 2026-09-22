# OpenViking × Copilot：给主力 Copilot 的同事装上记忆

**受众：** 日常以 VS Code Copilot / Copilot CLI 为主力的同事（第 1、2 期的读者）。已经在用 Pi + herdr 的同事，重点看「与本期其他章的关系」一节即可。

**核验基线：** 内容按插件源码仓库 radial-hks/copilot-ov-capture（v0.4.12）与 OpenViking 官方文档核验于 2026-09。该插件为公司内部维护、迭代较快，发布前建议技术负责人复核版本号与团队服务器地址。

**一句话定位：** 本期前几章讲的是「给 Pi + herdr 装 OpenViking 记忆」。本章给**主力用 Copilot 的同事**一条对等的路：装一个 **openviking-copilot** 插件，让 VS Code Copilot / Copilot CLI 也接上**同一份** OpenViking 记忆库——会话不再"结束即失忆"。

## 本章解决什么

前四章把 OpenViking 的「记忆 + 资源 + 技能」能力接到了 **Pi + herdr** 这条进阶路线上。但公司里大多数同事的日常主力是 **VS Code Copilot / Copilot CLI**（第 1、2 期讲过）。对这批同事，前几章的安装命令用不上，而"失忆"的痛点却一点不少：

- **会话结束即遗忘**：你上周在 Copilot 里讲过的项目背景、跑通的流程、踩过的坑，换个会话它就全忘了；
- **跨项目上下文丢失**：同一个坑，换一个仓库、换一台机器，又得从头讲一遍；
- **团队经验沉淀不下来**：A 同事在项目里摸清的做法，B 同事的 Copilot 完全不知道。

本章回答三件事：

1. 为什么主力 Copilot 的同事也需要 OpenViking，这条路线和第 2 章 Pi 路线是什么关系；
2. **openviking-copilot** 插件是什么、怎么装、怎么配；
3. 落地时最容易踩的两个坑（团队最值钱的部分）。

读完你可以给自己（或帮同事）的 VS Code Copilot / Copilot CLI 装上记忆。

## 为什么主力 Copilot 的同事也需要 OpenViking

### 痛点：Copilot 的「上下文」和「记忆」是两回事

第 1 章讲过：**上下文（Context）** 是当前会话里 AI 看得到的东西，会话结束就没了；**记忆（Memory）** 是跨会话沉淀下来的事实、偏好、经验。Copilot 原生只有前者——它很会"读当前工作区"，但不会"记住上次讲过的话"。

| 场面 | 典型例子 |
| --- | --- |
| 重复自我介绍 | "我是负责角色建模的小李，我们项目用 Maya，单位是厘米" |
| 重复讲项目背景 | "这个项目输出目录在 `D:/Projects/Demo/export`，贴图用 PBR 流程" |
| 重复踩坑 | "上次说过：批量处理前必须先备份原文件" |
| 团队经验不通 | 一个同事跑通的坑，另一个同事的 Copilot 从头再踩一遍 |

### 关键：和 Pi 路线共用同一份记忆库

这不是给 Copilot 另起一套记忆系统，而是把 Copilot 也接到**同一个 OpenViking 记忆库**上：

| | 第 2 章路线（Pi + herdr） | 本章路线（Copilot） |
| --- | --- | --- |
| 宿主 | Pi 编码 agent | VS Code Copilot / Copilot CLI |
| 接入方式 | Pi 官方扩展 | openviking-copilot 插件 |
| 记忆库 | OpenViking | **同一个** OpenViking |
| 记忆分区 | 按 **Peer（对等节点）** 隔离 | **同一个** Peer |

两条路线殊途同归：**同一个项目（同一个 Peer），Pi 用户沉淀的记忆，Copilot 同事的 Copilot 能自动召回；反过来也一样。** 这就是第 4 章闭环里"多 agent 共享记忆"拼图的 Copilot 半边。

## openviking-copilot 插件：一个包，双宿主

### 它是谁

| 项 | 值 |
| --- | --- |
| 仓库 | radial-hks/copilot-ov-capture |
| 插件名 | openviking-copilot |
| 当前版本 | v0.4.12 |
| 打包格式 | **Agent Plugins 1.0** |
| 宿主 | Copilot CLI + VS Code Copilot（一个包同时覆盖） |
| 维护方 | radial-hks 组织（公司内部维护） |

> 💡 **Agent Plugins 1.0** 是一套与厂商无关的 AI 编码 Agent 插件打包规范（第 1 期第 5 章的 [Agent Plugin 定制机制](../../01-copilot/05-customization.md) 提过它的 VS Code 预览形态）。一个插件目录里放 `plugin.json` 清单、`skills/` 技能和可选的 `mcp.json` 声明，符合规范的客户端都用同样的方式加载它。

### 能力一览

插件同时提供「模型按需调用」和「生命周期自动触发」两类能力：

| 能力 | 作用 | 触发时机 |
| --- | --- | --- |
| **MCP（Model Context Protocol）记忆工具** | 模型按需调用 `find` / `search` / `read` 召回、`remember` / `add_resource` / `forget` 沉淀、`add_skill` 写技能 | 对话中模型主动调用 |
| **自动 Recall 注入** | 每条 prompt 发出前自动检索团队记忆，把最相关条目注入为 `<openviking-context>` 块——Copilot 不用调任何工具就"记得" | 每次提问自动 |
| **SessionStart profile 注入** | 新会话第一条 prompt 注入你的个人画像（profile.md）与偏好/实体清单，第一句话就"认识你" | 会话开场自动 |
| **自动会话 Capture** | 会话结束把「用户输入 + 模型回复」上传，服务端异步提取记忆（约 1–2 分钟） | 会话结束自动 |
| **PreCompact 归档** | 长会话触发压缩前，先把未归档增量上传并 commit，压缩不丢内容 | 压缩前自动 |
| **viking:// URI 守卫** | 拦截 Copilot 误把 `viking://` 资源当本地文件读写，提示改用 openviking MCP 工具 | 调用工具前自动 |
| **doctor 自检** | 9 项体检（Node / 凭据 / 连通 / 鉴权 / 技能接口 / peer / 捕获管线 / 插件注册 / Hook 命令路径） | 手动运行 |

> 💡 记住分工：**自动召回让你"不用记"**（每条 prompt 前自动带上相关记忆），**自动捕获让你"不用存"**（会话结束自动沉淀）。这两件是"免工具调用"的，靠生命周期**钩子（Hook）**驱动，比让模型自己想起来去查更省 token、也更可靠。
>
> **隐私边界**：自动捕获只上传「你发给 Copilot 的消息」和「模型的文本回复」；工具调用的输入/输出、与 OpenViking 无关的 VS Code 操作**不上传**（hook 只在 Copilot 会话生命周期触发）。

## 安装：三条命令 + 一次凭据

### 前置条件

| 条件 | 说明 | 检查 |
| --- | --- | --- |
| VS Code | ≥ 1.102（Agent plugins + hooks 支持） | `code --version` |
| Node | ≥ 18 | `node --version` |
| 团队 OpenViking 服务 | 内网服务，管理员为每人签发 user key | 见下文「凭据」 |

### 路径 1：Copilot CLI 市场安装（推荐）

在终端里跑（macOS/Linux 用 bash，Windows 用 PowerShell）：

```bash
# 如果团队环境已预注册 openviking-team 市场，只需这一行
copilot plugin install openviking-copilot@openviking-team

# 如果提示 marketplace unknown，先注册一次内部市场，再安装
copilot plugin marketplace add radial-hks/copilot-ov-capture
copilot plugin install openviking-copilot@openviking-team

# 验证已加载
copilot plugin list
```

`copilot plugin list` 应能看到 **openviking-copilot**。

### VS Code 端如何生效

CLI 安装的插件落在 `~/.copilot/installed-plugins/`，**VS Code 会自动发现、两边共用，无需重复安装**。装完后在 VS Code 里执行 `Developer: Reload Window`（`Ctrl+Shift+P` → 输入 `Reload Window`）让插件重新加载。

### 路径 2：VS Code 图形安装（备选）

没有可用的 Copilot CLI 时，改走 VS Code 图形界面：先在用户级 `settings.json` 加一行（合并，不覆盖现有设置）：

```jsonc
"chat.plugins.marketplaces": ["radial-hks/copilot-ov-capture"]
```

然后：`Ctrl+Shift+X` 打开 Extensions → 搜索 `@agentPlugins` → 找到 **openviking-copilot** → Install（首次会弹市场信任确认）→ `Ctrl+Shift+P` → `Developer: Reload Window`。图形安装会把插件放进 VS Code 的 agent-plugins 目录（`~/.vscode/agent-plugins/github.com/radial-hks/copilot-ov-capture/`）。

### 装完自检（doctor）

插件装好后，跑一次自诊断（9 项体检，FAIL 项自带修复提示）：

```bash
# macOS / Linux
node ~/.copilot/installed-plugins/openviking-team/openviking-copilot/scripts/ov-doctor.mjs
```

```powershell
# Windows
node "%USERPROFILE%\.copilot\installed-plugins\openviking-team\openviking-copilot\scripts\ov-doctor.mjs"
```

> 路径不存在时，先 `copilot plugin list` 找到插件目录再对路径运行。其中 **Hook 命令路径** 项必须 PASS——它直接对应"hooks 静默失效"类问题（见下文「已知坑」）。

## 配置：凭据 + Peer

装完插件本体只完成了一半，还需要两处配置。

### 服务端 endpoint 与 API Key

手工创建 `~/.openviking/ovcli.conf`（Windows 为 `%USERPROFILE%\.openviking\ovcli.conf`）：

```json
{ "url": "<团队 OpenViking 服务器地址>", "api_key": "<你的 user key>" }
```

- **url**：团队 OpenViking 服务地址（本地模式是 `http://localhost:1933`；团队内网服务由管理员私发地址）。
- **api_key**：管理员为每人私发的 user key（一串 token）。本地未启用鉴权的服务端可不填。
- ⚠️ 该文件含个人凭据，**不进任何 git 仓库**；用记事本另存为 UTF-8 无 BOM（Node 的 `JSON.parse` 对 BOM 敏感）。

凭据解析顺序：环境变量（`OPENVIKING_URL` / `OPENVIKING_API_KEY`）→ `~/.openviking/ovcli.conf` → `~/.openviking/ov.conf`。配置改动会被运行中的代理自动读取，无需重启。

### Peer：按项目区分记忆

**Peer（对等节点）** 是 OpenViking 里"把一个项目的记忆归拢在一起"的分区（第 1 章已介绍）。Copilot 插件会从会话所在目录推导工作区 Peer，让项目经验自动归到 `peers/<peer_id>/` 下：

- **有 origin 的仓库**：无需任何配置。Peer 取归一化的 Git origin，格式为 `github.com-<owner>-<repo>`（例如 `github.com-radial-hks-zetetic`）；同一个仓库的所有 clone、所有同事机器推导出同一个 Peer，项目记忆自动聚合。
- **想自定义**：仓库根建 `.openviking/config.json` 提交进 git，全团队生效：

```json
{ "version": 1, "peer": { "id": "my-project" } }
```

- **非 git 目录**：不发 Peer，记忆进用户级空间，不会为临时目录凭空造空 Peer。
- ⚠️ `url` / `api_key` 这类凭据写进工作区配置会被剥离——服务器地址永远只看 `ovcli.conf`。

> 💡 Peer 是"同 peer 即同记忆"的关键：Copilot 同事和 Pi/herdr 用户只要在同一个仓库里干活，就落在同一个 Peer，两边写的记忆互相可见。

## 已知坑（团队落地最值钱的部分）

> 本节核验时间：2026-09。这两条坑都来自插件真实排障记录（changelog），落地时最常踩。

### 坑 1：hooks 超时字段必须写 `timeoutSec`

Copilot CLI 对 hook 条目里的**未知字段**会整体静默丢弃——不报错、不提示，只是那个 hook 不再触发。插件早期版本用过 `timeout` 字段（很多 hook 示例里都这么写），结果整个 hook 条目被丢弃，自动召回/自动捕获**静默失效**。正确字段是 `timeoutSec`（单位：秒）。

- ⚠️ 自己写或改插件 hooks 时，超时字段**只能写 `timeoutSec`**，别照抄网上示例里的 `timeout`。
- 🤖 排查"hook 没触发又不报错"时先怀疑这个：把插件升级到 v0.4.12（已修），或在 `Developer: Show Agent Debug Logs` 里确认 hook 是否加载。

### 坑 2：跨宿主 cwd 不同，别依赖相对路径

Copilot CLI 执行插件 hook 时，**cwd 是插件安装目录**；VS Code 执行时，**cwd 是当前工作区**。两个宿主的工作目录完全不同，跨宿主启动**不能只依赖相对路径**（`./scripts/xxx` 在两边会解析到不同位置）。

插件自己的解法是"自定位引导"：hook 命令用 `node -e` 在 Node 进程内依次探测 `PLUGIN_ROOT` 环境变量 → 当前 cwd → `~/.copilot/installed-plugins/` 扫描，命中后再执行真实脚本。所以对你来说：

- 升级到 v0.4.12 即可，无需手动处理路径；
- 若要自己加 hook，跨宿主路径别用相对路径，用绝对路径或同样的自定位方式。

### 其他常见症状速查

| 症状 | 处理 |
| --- | --- |
| Copilot 说没有 openviking MCP 工具 | CLI 用户 `copilot plugin list` 确认已装 + 终端重启；图形安装用户检查 `chat.plugins.marketplaces` 并 Reload Window |
| MCP 启动报 `Process exited with code 1` / `Cannot find module ...${PLUGIN_ROOT}...` | ≤0.4.9 的已知缺口（宿主不展开 `${PLUGIN_ROOT}` 占位符），升级到 ≥0.4.10 并 Reload Window |
| 改了插件源码没生效 | CLI 缓存陷阱：重新 `copilot plugin install openviking-copilot@openviking-team` 才生效 |
| 会话没被捕获 | Copilot CLI 的 `-p` 非交互模式**不触发插件 hooks**（仅支持 repo hooks）；验证捕获用交互式会话或 VS Code |
| 服务器不可达、召回为空 | 插件会**静默跳过、绝不阻塞你的提问**——先 `curl <服务器地址>/health` 确认连通 |

## 与本期其他章的关系

本章是第 4 章"多 agent 共享记忆"闭环里**缺的 Copilot 半边**：

- [第 1 章](01-what-is-openviking.md) 解释了 OpenViking 是什么、Peer 怎么按项目隔离记忆——本章的 Peer 配置是它的落地；
- [第 2 章](02-installation.md) 给 Pi + herdr 装上了记忆扩展——本章给 Copilot 装上了**对等的插件**，两条路线共用**同一个 OpenViking 记忆库**；
- [第 3 章](03-pi-herdr-multiagent.md)、[第 4 章](04-combo-workflow.md) 的编排与闭环，主体是 Pi + herdr。但"记忆"这一环是双宿主共享的：Pi 用户沉淀的项目经验，Copilot 同事下次提问会自动召回；Copilot 同事踩坑后捕获的教训，Pi 用户同样读得到。

一句话：**不管你在哪条路线上干活，只要同一个仓库（同一个 Peer），大家都用同一份记忆。** 主力 Copilot 的同事不必迁移到 Pi + herdr，也能搭上这趟"长期记忆"的车。

## 版本事实

- 插件当前版本 **v0.4.12**，核验时间 **2026-09**。
- 该插件目前为**公司内部维护**（radial-hks 组织），路线目标是演进为 OpenViking 官方 Agent 适配器（v0.4.11–0.4.12 已做许可证与去私有化改造，为上游化铺路）。
- OpenViking 服务端与插件都在快速迭代，发布前建议技术负责人复核版本号与团队部署形态。

## 参考

- radial-hks/copilot-ov-capture 仓库：<https://github.com/radial-hks/copilot-ov-capture>
- OpenViking 官方文档：<https://openviking.ai/>
- OpenViking Agent Plugins 1.0 插件包（官方）：<https://docs.openviking.ai/zh/agent-integrations/15-agent-plugins>

---

← 上一节：[三件套组合工作流](04-combo-workflow.md) ｜ 返回主页：[AI 工具团队参考手册](../../README.md)
