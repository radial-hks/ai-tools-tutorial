# 共享术语表

> 本术语表服务于整个 AI 工具团队参考手册。解释尽量使用一句话，详细教程请看各章节正文。

## 通用 AI 术语

| 术语 | 解释 |
| ------ | ------ |
| AI 工具 | 帮助你写作、编程、整理文件、处理资料或自动化任务的 AI 软件。 |
| Prompt | 你发给 AI 的请求，应包含目标、上下文、约束和验收。 |
| Context（上下文） | AI 处理请求时可见的文件、选区、终端输出、图片或外部资料。 |
| Token | 模型处理文本的计量单位，影响上下文容量、速度和费用。 |
| Agent（智能体） | 能围绕任务使用工具、读取上下文并多步推进的 AI 工作方式。 |
| Diff | 文件修改前后的对比，用于审查和回滚。 |
| API Key | 调用模型或服务的访问凭证，不能发给 AI 聊天窗口或公开群聊。 |
| OAuth | 通过网页登录授权第三方工具访问账号的方式。 |

## VS Code Copilot 术语

| 术语 | 解释 |
| ------ | ------ |
| Inline Suggestions | 编辑器中随输入出现的灰色补全建议。 |
| Chat View | VS Code 侧边栏中的代码优先聊天界面。 |
| Agents Window | 面向多任务编排的独立 Agent 窗口，状态以当前 VS Code 版本为准。 |
| Inline Chat | 在编辑器或终端当前位置发起的就地聊天。 |
| Quick Chat | 顶部轻量聊天面板，适合临时问题。 |
| Agent type / 运行位置 | Agent 在哪里运行，例如本地、Copilot CLI、Cloud Agent 或第三方提供者。 |
| Agent role / 角色 | Agent 如何行动，例如 Agent、Plan、Ask 或 Custom Agent。 |
| Permission level | 当前会话允许 AI 使用工具、编辑文件或运行命令的自主程度。 |
| Checkpoints | 聊天编辑过程中的快照，可辅助回退，但不能替代 Git 提交。 |

## VS Code Copilot 定制术语

| 术语 | 解释 | 常见位置 |
| ------ | ------ | ---------- |
| Instructions | 项目或文件范围内的规则与约定，Chat / Agent 会按适用范围加载。 | `.github/copilot-instructions.md`, `.github/instructions/*.instructions.md` |
| Prompt Files | 可手动调用的复用 Prompt。 | `.github/prompts/*.prompt.md` |
| Agent Skills | 可复用能力包，可包含流程、脚本、模板和资源。 | `.github/skills/<name>/SKILL.md` |
| Custom Agents | 自定义角色、指令、工具、模型和交接动作。 | `.github/agents/*.agent.md` |
| Hooks | Agent 生命周期中的确定性命令，通常用于审计、格式化或阻断危险操作。 | `.github/hooks/*.json` |
| Agent Plugins（Preview） | 打包分发 Skills、Agents、Hooks、MCP 配置等定制内容。 | `plugin.json` |
| MCP | Model Context Protocol，用于把外部工具、资源、Prompt 或 App 暴露给 AI。 | `.vscode/mcp.json` |
| MCP Resources | MCP Server 提供的只读上下文资源。 | Server 定义 |
| MCP Apps | MCP Server 提供的交互式 UI 组件。 | Server 定义 |
| BYOK | Bring Your Own Key，用自己的模型凭证接入语言模型；不等同于获得 GitHub Copilot 服务。 | 语言模型配置 |

## Copilot Skill 术语

| 术语 | 解释 |
| ------ | ------ |
| SKILL.md | 定义单个 Skill 的 Markdown 文件：顶部是 YAML frontmatter（`name`、`description` 等），正文是给 AI 看的操作指令。 |
| Skill 目录名 | 存放 Skill 的文件夹名，必须与 `SKILL.md` 的 `name` 字段完全一致，否则静默不加载。 |
| name（Skill 字段） | Skill 唯一标识，只能小写字母、数字、短横线，最长 64 字符。 |
| description（Skill 字段） | 描述 Skill 干什么以及何时用，决定 Copilot 能否按需自动加载它。 |
| argument-hint（Skill 字段） | 作为斜杠命令调用时，聊天输入框里显示的提示文字，例如 `[test file] [options]`。 |
| disable-model-invocation | `SKILL.md` 前置字段：设 `true` 时禁止模型自动加载，只能手动 `/` 调用。 |
| user-invocable | `SKILL.md` 前置字段：控制 Skill 是否出现在 `/` 菜单，默认 `true`。 |
| 三级加载 | Skill 的按需加载机制：先读 name/description 判断相关性，再加载正文，最后才读取被引用的脚本/资源。 |
| context（Skill 字段） | 实验性字段，控制加载方式：默认 `inline`（指令加入父会话上下文），设 `fork` 则在独立子代理中运行、只回传最终结果。 |
| Agent Skills 标准 | Skill 的开放标准（agentskills.io），跨 VS Code、Copilot CLI、云智能体可移植。 |

## Hermes Agent 术语

| 术语 | 解释 |
| ------ | ------ |
| Hermes Agent | Nous Research 推出的个人 AI Agent，可通过桌面、CLI、TUI 和 Gateway 使用工具完成多步任务。 |
| Hermes CLI | 在终端中运行的 Hermes 命令行入口，例如 `hermes`、`hermes chat -q`、`hermes setup`。 |
| Hermes TUI | 终端里的现代交互界面，使用 `hermes --tui` 启动。 |
| Hermes Desktop | Hermes 的桌面应用入口，适合不想长期使用命令行的同事。 |
| Hermes Gateway | Hermes 的消息平台网关，用于接入 Telegram、Discord、Slack、WhatsApp 等平台；不是模型配置入口。 |
| Toolset | 一组 Hermes 可用工具，例如文件、终端、浏览器、网页搜索、技能等。 |
| Hermes Memory | Hermes 的长期记忆系统，用于保存少量长期偏好、环境事实和约定；不要保存密码、Token 或 API Key。 |
| Hermes Skill | Hermes 的可复用流程说明或能力包，通常以 `SKILL.md` 形式描述触发条件、步骤、坑和验收。 |
| Profile | Hermes 的隔离配置空间，不同 Profile 可以有独立配置、会话、记忆和技能。 |
| Cron | Hermes 的定时任务系统，用于按计划运行提示词或脚本。 |

## Pi 工作流术语

| 术语 | 解释 |
| ------ | ------ |
| Pi / Pi Agent | 跑在终端里的极简 AI 编程助手，命令名 `pi`；默认只带读/写/改/跑命令四个基础工具，可无限扩展。 |
| Oh My Pi（OMP） | Pi 的「功能加强版」，命令名 `omp`；开箱即用子代理、待办清单、LSP、浏览器、Python、联网搜索等。 |
| herdr | 终端工作区管理器：把多个终端组织成 workspace / tab / pane，并识别每个 pane 里跑的 agent。 |
| workspace / tab / pane | herdr 的三层结构：工作区（w1）→ 标签页（w1:t1）→ 面板（w1:p1）。 |
| agent 状态（herdr） | herdr 对 pane 内 agent 的归类：idle / working / blocked / done / unknown。 |
| AGENTS.md | Pi 每次启动自动加载的项目说明文件，可写项目约定、常用命令、注意事项；也可用 CLAUDE.md。 |
| Pi 包（Pi Package） | 把扩展、技能、主题等打包分发的方式，通过 npm 或 git 安装。 |
| Pi Skill（技能） | Pi 按需加载的能力包，按触发条件自动启用；放在 `~/.pi/agent/skills/` 或 `.pi/skills/`。 |
| Pi Extension（扩展） | 用 TypeScript 写的自定义工具、命令、快捷键、UI；放在 `~/.pi/agent/extensions/` 或 `.pi/extensions/`。 |
| Pi Theme（主题） | Pi 的配色方案，热重载即时生效；放在 `~/.pi/agent/themes/` 或 `.pi/themes/`。 |
