# AI 工具团队参考手册

> 面向工程、美术、建模、技术美术和实习同事的 AI 工具使用文档。
>
> 每期聚焦一个工具，目标是提供可检索、可维护、可演示的团队参考资料。

## 仓库结构

```text
ai-tools-tutorial/
├── README.md
├── docs/
│   ├── _shared/
│   │   ├── glossary.md
│   │   └── cheat-sheet.md
│   ├── 01-copilot/
│   │   ├── README.md
│   │   ├── 01-overview-and-setup.md
│   │   ├── 02-chat-and-inline.md
│   │   ├── 03-context-and-prompts.md
│   │   ├── 04-agents-and-workflows.md
│   │   ├── 05-customization.md
│   │   ├── 06-mcp-and-tools.md
│   │   ├── 07-safety-and-troubleshooting.md
│   │   └── appendix-demo-guide.md
│   ├── 02-copilot-skills/
│   ├── 03-hermes/
│   │   ├── README.md
│   │   ├── 01-what-is-hermes.md
│   │   ├── 02-installation.md
│   │   ├── 03-cli-tui-gateway.md
│   │   ├── 04-memory-and-skills.md
│   │   └── 05-hermes-vs-copilot.md
│   └── 04-pi/
│       ├── README.md
│       ├── 01-why-pi.md
│       ├── 02-installation.md
│       ├── 03-pi-agent.md
│       ├── 04-oh-my-pi.md
│       ├── 05-herdr.md
│       └── 06-cooperation.md
├── assets/
└── templates/
```

## 第 1 期：VS Code Copilot（2026-07）

| 文档 | 内容 |
| ------ | ------ |
| [本期首页](docs/01-copilot/README.md) | 任务入口、阅读建议、官方基线 |
| [概览与安装](docs/01-copilot/01-overview-and-setup.md) | 产品边界、账号、订阅、安装与验证 |
| [聊天与内联交互](docs/01-copilot/02-chat-and-inline.md) | Chat View、Agents Window、Inline Chat、Quick Chat、内联建议 |
| [上下文与提示](docs/01-copilot/03-context-and-prompts.md) | 隐式上下文、显式引用、图片与浏览器上下文、Prompt 四要素 |
| [Agent 与工作流](docs/01-copilot/04-agents-and-workflows.md) | 五个会话配置维度、Plan -> Implement -> Review |
| [定制 Copilot](docs/01-copilot/05-customization.md) | Instructions、Prompt File、Skill、Custom Agent、Hook、Plugin |
| [MCP 与工具](docs/01-copilot/06-mcp-and-tools.md) | 工具来源、MCP Server、配置、信任、沙箱和排错 |
| [安全与排错](docs/01-copilot/07-safety-and-troubleshooting.md) | 审查清单、权限、隐私、回滚和诊断入口 |
| [讲师演示附录](docs/01-copilot/appendix-demo-guide.md) | 60-90 分钟演示流程、准备清单和 Prompt 速查 |

## 第 3 期：Hermes Agent（2026-07）

| 文档 | 内容 |
| ------ | ------ |
| [本期首页](docs/03-hermes/README.md) | 任务入口、阅读建议、官方基线与发布提示 |
| [Hermes 是什么](docs/03-hermes/01-what-is-hermes.md) | 产品定位、核心能力、适用场景 |
| [安装 Hermes](docs/03-hermes/02-installation.md) | 🥇 Copilot 陪跑安装、WSL2 + Ubuntu、VS Code 连接 WSL、Windows 桌面/原生安装、验证与排错 |
| [使用 Hermes](docs/03-hermes/03-cli-tui-gateway.md) | Dashboard、桌面应用、CLI、TUI、Gateway 网关的正确分工 |
| [记忆与技能系统](docs/03-hermes/04-memory-and-skills.md) | Memory 与 Skill 的概念、隐私边界和常用命令 |
| [Hermes 与 Copilot](docs/03-hermes/05-hermes-vs-copilot.md) | 定位差异、协作方式、选型建议 |

## 第 4 期：Pi 工作流（2026-08）

| 文档 | 内容 |
| ------ | ------ |
| [本期首页](docs/04-pi/README.md) | 任务入口、阅读建议、三大件分工与官方基线 |
| [为什么需要 Pi](docs/04-pi/01-why-pi.md) | Pi 的定位、与 Copilot / Hermes 的差异、三大件分工 |
| [安装 Pi](docs/04-pi/02-installation.md) | 前置条件、官方脚本 / npm 安装、登录模型、验证与排错 |
| [使用 Pi Agent](docs/04-pi/03-pi-agent.md) | 交互界面、常用命令与快捷键、会话、AGENTS.md 与定制 |
| [Oh My Pi](docs/04-pi/04-oh-my-pi.md) | OMP 与 Pi 的关系、增强能力、安装初始化与选型 |
| [herdr 工作区](docs/04-pi/05-herdr.md) | workspace / tab / pane 概念、查看其他面板、并行多 agent |
| [三者配合与选型](docs/04-pi/06-cooperation.md) | Copilot / Hermes / Pi / OMP / herdr 协作全景与选型地图 |

## 后续规划

| 期次 | 主题 | 状态 | 预计 |
| ------ | ------ | ------ | ------ |
| 01 | VS Code Copilot 团队参考手册 | 维护中 | 2026-07 |
| 02 | Copilot Skill 安装与使用 | 规划中 | TBD |
| 03 | Hermes Agent 基础 | 已完成并按官方文档核验 | 2026-08 |
| 04 | Pi / OMP / herdr 工作流 | 已完成并按本机环境核验 | 2026-08 |

## 贡献方式

1. 文档使用 Markdown，中文撰写。
2. 新增工具文档时，复制 [templates/tool-doc-template.md](templates/tool-doc-template.md) 作为起点。
3. 截图、录屏和设计图放入 `assets/`。
4. 涉及 VS Code / Copilot / Hermes Agent 快速迭代功能时，注明核验月份和官方参考链接。
