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
│   │   ├── README.md
│   │   ├── 01-what-is-skill.md
│   │   ├── 02-install-and-use.md
│   │   ├── 03-create-skill.md
│   │   ├── 04-skill-vs-others.md
│   │   └── 05-recommendations-and-practice.md
│   ├── 03-hermes/
│   │   ├── README.md
│   │   ├── 01-what-is-hermes.md
│   │   ├── 02-installation.md
│   │   ├── 03-cli-tui-gateway.md
│   │   ├── 04-memory-and-skills.md
│   │   ├── 05-hermes-vs-copilot.md
│   │   └── appendix-wsl-guide.md
│   └── 04-pi/
│       ├── README.md
│       ├── 01-why-pi.md
│       ├── 02-installation.md
│       ├── 03-pi-agent.md
│       ├── 04-oh-my-pi.md
│       ├── 05-herdr.md
│       └── 06-cooperation.md
│   └── 05-agent-memory/
│       ├── README.md
│       ├── 01-why-agent-memory.md
│       ├── 02-memory-systems-comparison.md
│       ├── 03-relation-to-hermes-memory.md
│       ├── 04-privacy-and-deployment.md
│       └── appendix-hands-on.md
│   └── 06-dsh/
│       ├── README.md
│       ├── 01-what-is-dsh.md
│       ├── 02-installation.md
│       ├── 03-web-gui.md
│       ├── 04-agent-presets.md
│       ├── 05-cordis-compositions.md
│       ├── 06-dynamic-plugins.md
│       └── 07-cooperation-and-selection.md
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

## 第 2 期：Copilot Skill 安装与使用（2026-08）

| 文档 | 内容 |
| ------ | ------ |
| [本期首页](docs/02-copilot-skills/README.md) | 任务入口、阅读建议、官方基线与发布提示 |
| [什么是 Copilot Skill](docs/02-copilot-skills/01-what-is-skill.md) | Skill 的定义、解决什么问题、与 Instructions / Prompt File / Custom Agent 的区别 |
| [安装与使用现有 Skill](docs/02-copilot-skills/02-install-and-use.md) | 获取来源、安装到项目/个人目录、启用/调用、验证生效 |
| [创建自定义 Skill](docs/02-copilot-skills/03-create-skill.md) | `SKILL.md` 完整结构、最小示例、命名规范、目录要求、测试排错 |
| [Skill 与 Prompt File / Instructions 的选型](docs/02-copilot-skills/04-skill-vs-others.md) | 三者对比、决策表、选型口诀、组合使用 |
| [常用 Skill 推荐与团队实践](docs/02-copilot-skills/05-recommendations-and-practice.md) | 美术/建模场景示例、团队推广、安全审查注意 |

## 第 3 期：Hermes Agent（2026-08）

| 文档 | 内容 |
| ------ | ------ |
| [本期首页](docs/03-hermes/README.md) | 任务入口、阅读建议、官方基线与发布提示 |
| [Hermes 是什么](docs/03-hermes/01-what-is-hermes.md) | 产品定位、核心能力、适用场景 |
| [安装 Hermes](docs/03-hermes/02-installation.md) | 🥇 Copilot 陪跑安装、WSL2 + Ubuntu、VS Code 连接 WSL、Windows 桌面/原生安装、验证与排错 |
| [使用 Hermes](docs/03-hermes/03-cli-tui-gateway.md) | Dashboard、桌面应用、CLI、TUI、Gateway 网关的正确分工 |
| [记忆与技能系统](docs/03-hermes/04-memory-and-skills.md) | Memory 与 Skill 的概念、隐私边界和常用命令 |
| [Hermes 与 Copilot](docs/03-hermes/05-hermes-vs-copilot.md) | 定位差异、协作方式、选型建议 |
| [WSL 保姆级补充指南](docs/03-hermes/appendix-wsl-guide.md) | WSL 加速安装、日常管理、开发三件套、文件互访、网络与备份 |

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

## 第 5 期：Agent 记忆系统（2026-08）

| 文档 | 内容 |
| ------ | ------ |
| [本期首页](docs/05-agent-memory/README.md) | 任务入口、阅读建议、官方基线与发布提示 |
| [为什么 Agent 需要长期记忆](docs/05-agent-memory/01-why-agent-memory.md) | 记忆 ≠ 聊天记录、四类记忆资产（Chat Memory / Skill / Wiki / CodeGraph）与 L0–L3 蒸馏 |
| [三个记忆系统横向对比](docs/05-agent-memory/02-memory-systems-comparison.md) | TencentDB Agent Memory vs Mnemosyne vs Hindsight：架构、集成、基准、决策表与选型口诀 |
| [与 Hermes Memory 的关系](docs/05-agent-memory/03-relation-to-hermes-memory.md) | 从"记什么"到"记忆架构选型"、给本机 Hermes 接 Mnemosyne / TencentDB / Hindsight 三条路径 |
| [隐私与部署权衡](docs/05-agent-memory/04-privacy-and-deployment.md) | local-first vs 云端、成本构成、隐私红线与团队落地路线 |
| [本机实测记录](docs/05-agent-memory/appendix-hands-on.md) | 三家系统真实安装命令、输出、踩坑与回退清单 |

## 第 6 期：DeepSeek Harness（DSH）团队参考手册（2026-08）

| 文档 | 内容 |
| ------ | ------ |
| [本期首页](docs/06-dsh/README.md) | 任务入口、阅读建议、官方基线与发布提示 |
| [DSH 是什么](docs/06-dsh/01-what-is-dsh.md) | 产品定位、核心概念（Profile / Cordis / Agent Preset）、三种入口模式、与现有工具差异 |
| [安装 DSH](docs/06-dsh/02-installation.md) | 前置条件、npm 安装、配置模型、首次启动、验证与排错 |
| [Web GUI 使用](docs/06-dsh/03-web-gui.md) | 界面布局、会话管理、消息交互、快捷键、设置 |
| [Agent Preset（模式）](docs/06-dsh/04-agent-presets.md) | 四种 shipped preset（standard / cordis / code / minimal）能力对比、选型、自定义 preset |
| [Cordis 组合系统](docs/06-dsh/05-cordis-compositions.md) | Host 平面 vs Agent Preset 平面、plugin row、profile patch 层叠、isolate realm |
| [动态插件](docs/06-dsh/06-dynamic-plugins.md) | 动态插件生命周期（define → run → update → stop → undefine）、Host/Client 双端 |
| [工具协作与选型](docs/06-dsh/07-cooperation-and-selection.md) | 六件套协作全景、选型决策表、学习路径建议 |

## 后续规划

| 期次 | 主题 | 状态 | 预计 |
| ------ | ------ | ------ | ------ |
| 01 | VS Code Copilot 团队参考手册 | 维护中 | 2026-07 |
| 02 | Copilot Skill 安装与使用 | 已完成并按官方文档核验 | 2026-08 |
| 03 | Hermes Agent 基础 | 已完成并按官方文档核验 | 2026-08 |
| 04 | Pi / OMP / herdr 工作流 | 已完成并按本机环境核验 | 2026-08 |
| 05 | Agent 记忆系统 | 已完成并按本机环境核验（含全链路 LLM 实测） | 2026-08 |
| 06 | DeepSeek Harness（DSH） | 已完成并按本机环境核验 | 2026-08 |
| 07 | Agent 基础设施选型总览 | 规划中 | TBD |

> 第 5–7 期的正式规划见 [proposals/2026-08-12-后续期次扩展提案.md](proposals/2026-08-12-后续期次扩展提案.md)。

## 贡献方式

1. 文档使用 Markdown，中文撰写。
2. 新增工具文档时，复制 [templates/tool-doc-template.md](templates/tool-doc-template.md) 作为起点。
3. 截图、录屏和设计图放入 `assets/`。
4. 涉及 VS Code / Copilot / Hermes Agent 快速迭代功能时，注明核验月份和官方参考链接。
