# AI 工具入门到进阶

> 面向工程部门新人（美术 / 开发 / 建模 / 实习生）的 AI 工具使用分享系列文档。
>
> 每期聚焦一个工具，从 "零基础能用" 到 "进阶定制" 的完整路径。

## 仓库结构

```
ai-tools-tutorial/
├── README.md                  ← 你在这里
├── docs/
│   ├── _shared/               ← 跨工具共享的概念和速查
│   │   ├── glossary.md         ← 术语表（Agent/Skill/Prompt/MCP…）
│   │   └── cheat-sheet.md      ← 通用快捷键速查
│   ├── 01-copilot/            ← 第 1 期：VS Code Copilot 入门到进阶
│   │   ├── README.md           ← 本期导航
│   │   ├── 01-what-is-copilot.md
│   │   ├── 02-basic-usage.md
│   │   ├── 03-context-and-prompts.md
│   │   ├── 04-agent-mode.md
│   │   ├── 05-advanced-customization.md
│   │   ├── 06-mcp-and-external-tools.md
│   │   ├── 07-best-practices.md
│   │   └── 08-demo-guide.md    ← 实机演示步骤与讲解思路
│   ├── 02-copilot-skills/     ← 第 2 期（规划）：Skill 安装与使用
│   ├── 03-hermes/             ← 第 3 期（规划）：Hermes Agent
│   └── 04-pi/                 ← 第 4 期（规划）：Pi / OMP
├── assets/                    ← 截图、设计图等媒体资源
└── templates/                 ← 文档模板（新增工具时复制使用）
    └── tool-doc-template.md
```

## 本期内容：VS Code Copilot（2026-07）

| 文档 | 内容 |
|------|------|
| [01-什么是 Copilot](docs/01-copilot/01-what-is-copilot.md) | 概念、安装启用、五种 AI 层次 |
| [02-基础使用](docs/01-copilot/02-basic-usage.md) | 三种交互模式、快捷键、斜杠命令 |
| [03-上下文与提示工程](docs/01-copilot/03-context-and-prompts.md) | #引用/@引用/图片/浏览器、八大原则 |
| [04-Agent 模式](docs/01-copilot/04-agent-mode.md) | 三种内置 Agent、Plan→实现工作流 |
| [05-进阶定制](docs/01-copilot/05-advanced-customization.md) | Instructions/Prompts/Agents/Skills/Hooks |
| [06-MCP 外部工具](docs/01-copilot/06-mcp-and-external-tools.md) | MCP 是什么、安装与配置 |
| [07-最佳实践](docs/01-copilot/07-best-practices.md) | 总结、避坑、学习路径 |
| [08-实机演示指南](docs/01-copilot/08-demo-guide.md) | 9 阶段演示步骤、概念讲解判断、Prompt 速查 |

## 后续规划

| 期次 | 主题 | 状态 | 预计 |
|------|------|------|------|
| 01 | VS Code Copilot 入门到进阶 | 已完成 | 2026-07 |
| 02 | Copilot Skill 安装与使用 | 规划中 | TBD |
| 03 | Hermes Agent 基础 | 规划中 | TBD |
| 04 | Pi / OMP 工作流 | 规划中 | TBD |

## 贡献方式

1. 本仓库使用 Git SSH 推送
2. 文档用 Markdown 格式，中文撰写
3. 新增工具文档时，复制 `templates/tool-doc-template.md` 作为起点
4. 截图等媒体放 `assets/` 目录
