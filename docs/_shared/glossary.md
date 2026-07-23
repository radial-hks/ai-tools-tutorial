# 术语表

> 跨工具共享的 AI 概念速查。每次新增工具文档时，如果涉及新术语，在此补充。

## 通用术语

| 术语 | 解释（给小白） | 比喻 |
|------|---------------|------|
| **LLM** | 大语言模型，AI 的大脑 | 像 AI 的"知识库+推理引擎" |
| **Prompt** | 你对 AI 说的那句话 | 像点菜——说越清楚，上菜越对 |
| **Context（上下文）** | AI 需要知道的背景信息 | 像你让同事帮忙，得告诉他是哪个文件 |
| **Token** | AI 处理文本的最小单位，影响消耗和费用 | 像字数计费——越长越贵 |
| **Agent（智能体）** | 能自主完成多步任务的 AI | 像实习生——你说任务，他自己搞定 |
| **Inline（内联）** | 就在当前编辑位置弹出 | 像在代码中间直接对话 |
| **Diff** | 代码改动的对比视图 | 红色删的、绿色加的，和 Git diff 一样 |

## VS Code Copilot 术语

| 术语 | 解释 |
|------|------|
| **Inline Suggestions** | 打字时出现的灰色补全建议，Tab 接受 |
| **Chat View** | 侧边栏的聊天面板 |
| **Agents Window** | 独立的智能体窗口，跨项目 |
| **Inline Chat** | 在编辑器代码中间弹出的聊天框 |
| **Quick Chat** | 顶部的轻量聊天面板 |
| **Ask / Agent / Plan** | 三种内置 Agent 模式：只读问答 / 可编辑 / 先规划 |
| **Checkpoints** | 聊天过程中的文件快照，可回滚 |

## VS Code Copilot 定制术语

| 术语 | 解释 | 文件格式 |
|------|------|---------|
| **Instructions** | 项目编码规范，永远生效 | `.github/copilot-instructions.md` |
| **Prompt Files** | 可复用任务模板，用 `/命令` 调用 | `.github/prompts/*.prompt.md` |
| **Custom Agents** | 专属角色（安全审查员、测试专家等） | `.github/agents/*.agent.md` |
| **Agent Skills** | 能力包，含脚本/模板 | `.github/skills/<name>/SKILL.md` |
| **Hooks** | 生命周期自动化（格式化、阻止危险命令等） | `.github/hooks/*.json` |
| **Plugins** | 打包分发以上所有 | `plugin.json` |
| **MCP** | 让 AI 连接外部工具（浏览器、数据库等） | `.vscode/mcp.json` |
| **BYOK** | 自带 API Key，不绑定 GitHub 账号 | `chatLanguageModels.json` |

## Hermes 术语（后续）

| 术语 | 解释 |
|------|------|
| **Hermes** | Nous Research 的 AI Agent 平台 |
| **Skill** | Hermes 的可复用工作流文件 |
| **Memory** | Hermes 的跨会话持久记忆 |

## Pi / OMP 术语（后续）

| 术语 | 解释 |
|------|------|
| **Pi** | Nous Research 的模型调度器 |
| **OMP** | Open Model Platform |
