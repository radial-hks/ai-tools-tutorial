# 术语表

> 跨工具共享的 AI 概念速查。正文负责完整说明，术语表只给最短定义。

## 通用术语

| 术语 | 解释 |
|------|------|
| LLM | 大语言模型，负责理解和生成文本、代码或多模态内容。 |
| Prompt | 你发给 AI 的请求，应包含目标、上下文、约束和验收。 |
| Context（上下文） | AI 处理请求时可见的文件、选区、终端输出、图片或外部资料。 |
| Token | 模型处理文本的计量单位，影响上下文容量、速度和费用。 |
| Agent（智能体） | 能围绕任务使用工具、读取上下文并多步推进的 AI 工作方式。 |
| Diff | 文件修改前后的对比，用于审查和回滚。 |

## VS Code Copilot 术语

| 术语 | 解释 |
|------|------|
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
|------|------|----------|
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

## 后续工具术语

| 术语 | 解释 |
|------|------|
| Hermes | 后续规划中的 Agent 平台主题。 |
| Pi / OMP | 后续规划中的模型调度与平台主题。 |