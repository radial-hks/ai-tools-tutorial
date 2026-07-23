# 定制 Copilot

本章解决什么：团队如何分层、命名并存放 VS Code 的 AI 定制资源（指令、提示、技能、智能体、钩子、插件），并给出最小可用示例与限制说明（基于 2026-07 官方文档）。

决策矩阵（快速判定）：
- 每次请求自动应用、用于项目规则 → Instructions
- 手动/斜杠触发、可复用的单次任务模板 → Prompt File
- Agent 在相关任务时自动加载、包含脚本/模板/资源 → Agent Skill
- 切换角色 / 限制工具集 / 固定交接 → Custom Agent
- 在 Agent 生命周期上执行确定性命令 → Hook
- 安装/分发打包（预览）→ Agent Plugin (Preview)

各项简明规范：每个类型下按同一四项小节展示：适用场景 / 默认位置 / 最小示例 / 限制。

## Instructions（项目指令）
- 适用场景：定义项目级编码规范、架构偏好、审查要求，自动随每次聊天请求加入模型上下文。
- 默认位置：`.github/copilot-instructions.md`（仓库根）；按文件匹配的可放 `.github/instructions/*.instructions.md`。
- 最小示例（`.github/copilot-instructions.md`）：

```markdown
## 团队约定（示例）
- 语言：TypeScript 5.x
- 代码风格：使用 Prettier/ESLint 约定
- 日志：使用 team-logger 包并包含 traceId
```
- 限制：作用于 Chat/Agent 模式请求（会话级），不保证影响编辑时的内联补全；请避免把能从代码静态分析得出的规则重复写入。

## Prompt File（提示词文件）
- 适用场景：把常用命令或复用工作流封成 `/` 命令（手动触发），如生成组件、准备 PR 描述。
- 默认位置：`.github/prompts/*.prompt.md`。
- 最小示例（`.github/prompts/scaffold.prompt.md`）：

```yaml
---
name: scaffold
description: 生成基础 React 组件
agent: agent
---
生成一个函数式 React + TypeScript 组件，包含 Props 类型和基本样式引用。
```
- 限制：需要用户触发（slash 或运行命令）；Prompt 的工具列表可覆盖 agent 的工具优先级；不要将大量全局规则写入 prompt 文件，使用 instructions 更合适。

## Agent Skill（技能）
- 适用场景：可复用、可被 Agent 自动加载的能力包，包含 `SKILL.md`、脚本与示例（测试、部署流程等）。Agent 会在任务匹配时逐步加载内容。
- 默认位置：`.github/skills/<name>/SKILL.md`。
- 最小示例（`.github/skills/web-test/SKILL.md`）：

```yaml
---
name: web-test
description: 生成并运行网页端测试用例（示例）
user-invocable: true
---
# 步骤
1. 发现目标页面
2. 生成测试模板
```
- 限制：`name` 必须为小写 kebab-case，目录名需匹配；技能可设为 `disable-model-invocation: true` 强制手动调用；大型技能可选实验性 `context: fork`。

## Custom Agent（自定义智能体）
- 适用场景：为特定角色定制工具集、模型与指令（如安全审查、计划者），并支持 handoffs（交接按钮）。
- 默认位置：`.github/agents/*.agent.md`。
- 说明（可选字段）：`tools`、`model` 与 `handoffs` 为可选配置项。
  - `tools`：限定该 agent 可调用的工具（如搜索、测试）；作者可列出想要可用的工具名称。
  - `model`：建议或锁定使用的模型/模型系列（非必需，UI/设置仍可能覆盖）。
  - `handoffs`：定义用户与外部流程或人工接手的交接点（按钮/说明）。
- 最小示例（`.github/agents/security.agent.md`）：

```yaml
---
name: security
description: 安全审查员（示例）
tools: ['search']
---
审查变更以发现输入校验、认证与授权问题。
```
- 限制：自定义智能体可在 UI 下切换，agent-scoped hooks 为预览功能；若工具不可用会被忽略；agent 文件可为 `user-invocable: false`。
不可用的工具名会被忽略；作者请从“Configure Tools”界面或自动完成中选择可用工具。

## Hook（钩子，Preview）
- 适用场景：在确定的 Agent 生命周期点运行命令（格式化、阻断危险操作、审计等），以代码形式强制执行或检查结果。
- 默认位置：`.github/hooks/*.json`（工作区）；也支持用户目录与 agent frontmatter（agent-scoped hooks，需启用设置）。
- 最小示例（`.github/hooks/format.json`）：

```json
{
  "hooks": {
    "PostToolUse": [
      { "type": "command", "command": "npx prettier --write ." }
    ]
  }
}
```
- 限制：Hooks 为预览（Preview）；官方定义的生命周期事件（截至 2026-07）包括：`SessionStart`、`UserPromptSubmit`、`PreToolUse`、`PostToolUse`、`PreCompact`、`SubagentStart`、`SubagentStop`、`Stop`；退出码与行为：`0` 解析 stdout，`2` 为阻断错误（停止并向模型展示 stderr），其他码显示警告并继续。Hook 输入/输出通过 stdin/stdout 的 JSON 交互。请避免让 agent 修改并直接执行未经审核的 hook 脚本。

## Agent Plugin（插件，Preview）
- 适用场景：将技能、智能体、钩子与 MCP 配置打包并通过市场或本地路径分发给团队/组织。
- 默认位置：插件需要 `plugin.json`（根）作为清单，插件可包含 `skills/`、`agents/`、`hooks`、`.mcp.json` 等（详见官方插件页面）。
- 最小示例（`plugin.json` 精简）

```json
{ "name": "my-dev-tools", "skills": "skills/", "agents": "agents/", "hooks": "hooks.json" }
```
- 限制：插件支持多格式（Copilot/Claude 等），为预览功能；安装插件前请审查内容与权限；插件内的 hooks/MCP 会随插件启用而运行。

## 逐步采用（Progressive adoption）
1. 初始化项目指令（`.github/copilot-instructions.md`）→ 覆盖团队约定。
2. 将高频手动任务抽成 Prompt File（`.github/prompts/*.prompt.md`）。
3. 将可复用、需脚本与示例的工作流打包为 Skill（`.github/skills/<name>/SKILL.md`）。
4. 当需要稳定角色与工具限制时创建 Custom Agent（`.github/agents/*.agent.md`）。
5. 只有当需要确定性自动化或组织级分发时引入 Hooks / Agent Plugins（两者均处于预览，注意审计）。

## 故障排查（Troubleshooting）
- 打开自定义编辑器：运行命令 `Chat: Open Customizations`（Agent Customizations 编辑器，预览）。
- 管理 Hooks：运行 `Chat: Configure Hooks` 或在 `.github/hooks/` 检查 JSON 文件。
- 查看诊断与加载信息：在 Chat 视图右上菜单选择 Diagnostics，或运行 `Developer: Show Agent Debug Logs` / `Developer: Open Agent Debug Panel` 以查看调试输出与 Hook 日志。

注：BYOK（自带 API Key）配置属于第 01 章的模型/凭证管理，不在本节作为独立定制类型；请参阅 [概览与安装](01-overview-and-setup.md) 获取 BYOK 细节。

## 参考（官方文档）
- 概览：https://code.visualstudio.com/docs/agent-customization/overview
- 指令（Instructions）：https://code.visualstudio.com/docs/agent-customization/custom-instructions
- 提示文件（Prompt files）：https://code.visualstudio.com/docs/agent-customization/prompt-files
- 技能（Agent Skills）：https://code.visualstudio.com/docs/agent-customization/agent-skills
- 自定义智能体（Custom agents）：https://code.visualstudio.com/docs/agent-customization/custom-agents
- 钩子（Hooks, Preview）：https://code.visualstudio.com/docs/agent-customization/hooks
- 插件（Agent plugins, Preview）：https://code.visualstudio.com/docs/agent-customization/agent-plugins

---

← 上一节：[Agent 与工作流](04-agents-and-workflows.md) ｜ 下一节：[MCP 与工具 →](06-mcp-and-tools.md)
