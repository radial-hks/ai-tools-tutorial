# 进阶定制：让 Copilot 懂你的项目

这一节是 "进阶" 部分。以下功能让 Copilot 了解你的项目规范、自动化重复任务。

## 定制功能总览

| 定制方式 | 作用 | 复杂度 | 适合新人？ |
|---------|------|--------|-----------|
| Custom Instructions | 项目编码规范 | 低 | 适合，最优先学 |
| Prompt Files | 常用任务模板 | 低 | 适合，简单实用 |
| Custom Agents | 专属角色 | 中 | 了解概念即可 |
| Agent Skills | 能力包 | 中 | 了解概念即可 |
| Hooks | 自动化触发 | 中高 | 后期再学 |
| Plugins | 打包分发 | 高 | 后期再学 |

定制选项的层次关系：

```
Instructions（编码规范，永远生效）
  └─ Prompt Files（任务模板，手动调用）
      └─ Skills（能力包，含脚本/模板）
          └─ Custom Agents（角色 + 工具限制 + Handoff）
              └─ Hooks（生命周期自动化）
                  └─ Plugins（打包分发以上所有）
```

---

## 1. Custom Instructions（自定义指令）—— 永远生效的规则

**是什么**：在项目中放一个 Markdown 文件，告诉 AI 你的编码规范、技术栈、架构约定。AI 在每次对话中都会自动遵守。

**怎么做**：

1. 在项目根目录创建 `.github/copilot-instructions.md`
2. 写入你的规则，例如：

```markdown
# 项目编码规范

## 技术栈
- 前端：React 18 + TypeScript
- 后端：Node.js + Express
- 数据库：PostgreSQL

## 命名规范
- 组件名用 PascalCase
- 变量名用 camelCase
- 常量用 ALL_CAPS

## 错误处理
- async 操作必须用 try/catch
- 错误必须带上下文信息记录日志
```

3. 保存即可。以后每次对话，AI 都会遵守这些规则。

**快速生成**：在聊天中输入 `/init`，AI 会分析你的项目并自动生成初始的指令文件。

**文件级指令**：如果不同文件类型需要不同规则，创建 `.instructions.md` 文件：

```yaml
---
applyTo: "**/*.tsx"
---
# React 组件规范
- 使用函数组件和 Hooks
- props 用 TypeScript interface 定义
- 样式用 CSS Modules
```

**注意事项**：
- 保持指令文件简洁——它们在每次对话中都会加载，只放 AI 无法从代码推断的信息
- 自定义指令不影响内联补全建议（打字时的灰色提示），只影响 Chat 和 Agent 模式
- 用 `applyTo` 分隔规则，不要把所有东西放一个文件

---

## 2. Prompt Files（提示词文件）—— 可复用的任务模板

**是什么**：把常用的 prompt 存成文件，用 `/命令名` 快速调用。

**怎么做**：

1. 在 `.github/prompts/` 目录下创建 `.prompt.md` 文件，例如 `new-component.prompt.md`：

```yaml
---
description: 生成一个新的 React 组件
agent: agent
tools: ['search/codebase', 'vscode/askQuestions']
---

根据模板生成一个新的 React 组件。
要求：
- 使用函数组件和 TypeScript
- 样式用 CSS Modules
- 包含基本的 Props 类型定义
```

2. 在聊天中输入 `/new-component` 即可调用

**变量**：支持 `${input:变量名}` 让用户在调用时填入参数：

```yaml
函数路径：${input:file_path:输入要测试的文件路径}
```

**Tool 引用**：在 body 中用 `#tool:工具名` 引用工具，如 `#tool:browser`、`#tool:vscode/askQuestions`

---

## 3. Custom Agents（自定义智能体）—— 专属角色

**是什么**：创建有特定人设和工具权限的 Agent。比如安全审查员、测试专家、架构师。

**怎么做**：

1. 在 `.github/agents/` 目录下创建 `.agent.md` 文件，例如 `security-reviewer.agent.md`：

```yaml
---
name: security-reviewer
description: 安全审查专家，检查代码中的安全漏洞
tools: ['search', 'editor']
handoffs:
  - label: 开始修复
    agent: agent
    prompt: 修复上面发现的安全问题
---

你是安全审查专家。审查代码时：
- 检查认证和授权
- 验证输入处理
- 查找 SQL 注入和 XSS 漏洞
- 按严重程度分类报告问题
```

2. 在聊天 Agent 选择器中选择 `@security-reviewer`

**Handoff（交接）**：一个 Agent 完成后，可以一键切换到下一个 Agent。

```
Plan Agent → [交接按钮] → Implement Agent → [交接按钮] → Review Agent
```

在 frontmatter 中定义：
```yaml
handoffs:
  - label: 开始实现    # 按钮显示文字
    agent: implementer  # 目标 agent
    prompt: 按上面的计划实现  # 预填的 prompt
    send: false         # false=需确认, true=自动提交
```

---

## 4. Agent Skills（智能体技能）—— 可复用的能力包

**是什么**：比 Prompt File 更强大，可以包含脚本、模板、示例文件。是一个开放标准，跨 VS Code / Copilot CLI / Cloud Agent 通用。

**怎么做**：

1. 在 `.github/skills/` 下创建目录，如 `api-testing/`
2. 创建 `SKILL.md`：

```yaml
---
name: api-testing
description: API 测试技能，用于创建和运行 API 测试用例
---

# API 测试技能

## 使用流程
1. 读取 API 定义文件
2. 生成测试用例
3. 运行测试并报告结果

参考模板：[测试模板](./test-template.js)
```

3. 可在目录中放入脚本和模板文件
4. 用 `/api-testing` 调用，或让 AI 自动匹配使用

**快速生成**：输入 `/create-skill` 并描述你想要的技能。

**关键规则**：
- `name` 必须和目录名一致（如 `api-testing` → `skills/api-testing/SKILL.md`）
- 只允许小写字母、数字、连字符
- 目录中额外文件需在 SKILL.md 中用 Markdown 链接引用才会被加载

---

## 5. Hooks（钩子）—— 自动化触发

**是什么**：在 Agent 生命周期的关键点自动执行 shell 命令。确定性的、代码驱动的自动化。

**怎么做**：

在 `.github/hooks/` 下创建 JSON 文件，如 `format.json`：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "type": "command",
        "command": "npx prettier --write ."
      }
    ]
  }
}
```

这样 Agent 每次修改文件后，自动运行 Prettier 格式化。

**8 个生命周期事件**：

| 事件 | 触发时机 | 典型用途 |
|------|---------|---------|
| SessionStart | 新会话开始 | 初始化资源、检查项目状态 |
| UserPromptSubmit | 用户发送消息 | 审计请求、注入系统上下文 |
| PreToolUse | 工具执行前 | 阻止危险操作（如 rm -rf） |
| PostToolUse | 工具执行后 | 运行格式化、测试 |
| PreCompact | 上下文压缩前 | 保存重要状态 |
| SubagentStart | 子 Agent 启动 | 跟踪嵌套使用 |
| SubagentStop | 子 Agent 完成 | 汇总结果 |
| Stop | 会话结束 | 生成报告、清理资源 |

**退出码**：0=成功，2=阻止操作，其他=警告

---

## 6. Plugins（插件）—— 打包分发

**是什么**：把上述所有定制（Skills + Agents + Hooks + MCP）打包成一个可安装的插件，分发给团队。

**结构**：
```
my-plugin/
  plugin.json          # 元数据
  skills/              # 技能
  agents/              # 自定义 Agent
  hooks.json           # 钩子配置
  .mcp.json            # MCP 服务器配置
```

**安装**：通过 `chat.pluginLocations` 或 `chat.plugins.marketplaces` 配置。

> 新人暂时不需要关注 Plugin，了解概念即可。

---

## BYOK（自带 API Key）

不绑定 GitHub 账号，用自己的 API Key 连接任意模型：
- 通过 `chatLanguageModels.json` 配置
- 支持 OpenAI、Anthropic、Azure 等提供商
- 甚至支持本地模型（如 Ollama）
- 不需要 Copilot 订阅

## 常用 AI 设置

通过 `File > Preferences > Settings`（Mac: `Code > Settings`）搜索：

| 设置项 | 说明 | 默认值 |
|--------|------|--------|
| `chat.agent.enabled` | 启用或禁用 Agent 功能 | `true` |
| `chat.agent.maxRequests` | Agent 最多可发送的请求数 | `25` |
| `github.copilot.enable` | 为各语言启用或禁用内联建议 | `{ "*": true }` |
| `github.copilot.nextEditSuggestions.enabled` | 启用下一次编辑建议（NES） | `true` |
| `chat.disableAIFeatures` | 禁用并隐藏所有内置 AI 功能 | `false` |
| `github.copilot.chat.localeOverride` | 指定聊天回复语言（`zh` = 中文） | `auto` |
| `chat.checkpoints.enabled` | 启用或禁用检查点 | `true` |
| `inlineChat.defaultModel` | 内联聊天的默认模型 | N/A |
| `chat.editing.autoAcceptDelay` | 自动接受编辑的延迟（0 = 禁用） | `0` |

### 让 AI 用中文回复

在设置中搜索 `localeOverride`，将其设为 `zh`。

---

← 上一节：[Agent 模式](04-agent-mode.md) ｜ 下一节：[MCP 外部工具 →](06-mcp-and-external-tools.md)
