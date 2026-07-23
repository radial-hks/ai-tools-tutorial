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
│   └── 04-pi/
├── assets/
└── templates/
```

## 第 1 期：VS Code Copilot（2026-07）

| 文档 | 内容 |
|------|------|
| [本期首页](docs/01-copilot/README.md) | 任务入口、阅读建议、官方基线 |
| [概览与安装](docs/01-copilot/01-overview-and-setup.md) | 产品边界、账号、订阅、安装与验证 |
| [聊天与内联交互](docs/01-copilot/02-chat-and-inline.md) | Chat View、Agents Window、Inline Chat、Quick Chat、内联建议 |
| [上下文与提示](docs/01-copilot/03-context-and-prompts.md) | 隐式上下文、显式引用、图片与浏览器上下文、Prompt 四要素 |
| [Agent 与工作流](docs/01-copilot/04-agents-and-workflows.md) | 五个会话配置维度、Plan -> Implement -> Review |
| [定制 Copilot](docs/01-copilot/05-customization.md) | Instructions、Prompt File、Skill、Custom Agent、Hook、Plugin |
| [MCP 与工具](docs/01-copilot/06-mcp-and-tools.md) | 工具来源、MCP Server、配置、信任、沙箱和排错 |
| [安全与排错](docs/01-copilot/07-safety-and-troubleshooting.md) | 审查清单、权限、隐私、回滚和诊断入口 |
| [讲师演示附录](docs/01-copilot/appendix-demo-guide.md) | 60-90 分钟演示流程、准备清单和 Prompt 速查 |

## 后续规划

| 期次 | 主题 | 状态 | 预计 |
|------|------|------|------|
| 01 | VS Code Copilot 团队参考手册 | 维护中 | 2026-07 |
| 02 | Copilot Skill 安装与使用 | 规划中 | TBD |
| 03 | Hermes Agent 基础 | 规划中 | TBD |
| 04 | Pi / OMP 工作流 | 规划中 | TBD |

## 贡献方式

1. 文档使用 Markdown，中文撰写。
2. 新增工具文档时，复制 [templates/tool-doc-template.md](templates/tool-doc-template.md) 作为起点。
3. 截图、录屏和设计图放入 `assets/`。
4. 涉及 VS Code / Copilot 快速迭代功能时，注明核验月份和官方参考链接。