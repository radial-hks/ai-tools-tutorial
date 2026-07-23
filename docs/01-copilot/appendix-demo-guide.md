# 讲师演示附录

> 给分享者使用的 60-90 分钟演示脚本。概念定义以正文为准，本附录只保留演示流程、准备清单和可复制 Prompt。

## 讲解前必读

- [概览与安装](01-overview-and-setup.md)
- [聊天与内联交互](02-chat-and-inline.md)
- [上下文与提示](03-context-and-prompts.md)
- [Agent 与工作流](04-agents-and-workflows.md)
- [定制 Copilot](05-customization.md)
- [MCP 与工具](06-mcp-and-tools.md)
- [安全与排错](07-safety-and-troubleshooting.md)

## 演示概览

| 阶段 | 时长 | 主题 | 对应章节 |
|------|------|------|----------|
| 0 | 5min | 开场与边界 | [01](01-overview-and-setup.md) |
| 1 | 10min | 内联建议 | [02](02-chat-and-inline.md) |
| 2 | 15min | Chat 与 Inline Chat | [02](02-chat-and-inline.md) |
| 3 | 10min | 上下文引用 | [03](03-context-and-prompts.md) |
| 4 | 15min | Plan -> Implement -> Review | [04](04-agents-and-workflows.md) |
| 5 | 10min | Instructions 与 Prompt File | [05](05-customization.md) |
| 6 | 10min | MCP 概念展示 | [06](06-mcp-and-tools.md) |
| 7 | 10min | Q&A 与排错 | [07](07-safety-and-troubleshooting.md) |

## 演示前准备

- [ ] VS Code 已更新并登录 Copilot。
- [ ] Demo 项目已初始化 Git，且有一个干净提交。
- [ ] 字体、投影、网络和代理已检查。
- [ ] 准备一个小型 Python 或 JavaScript 文件用于补全和 Inline Chat。
- [ ] 准备一张 UI 草图或网页截图用于图片上下文演示。
- [ ] 如果演示 MCP，只使用已安装并信任的 Playwright MCP Server。
- [ ] 备用方案：录屏、截图、已完成 diff、离线 Prompt 文本。

## 详细流程

### 阶段 0：开场与边界

目标：让听众知道 Copilot 能帮忙写代码、解释代码、修改文件和执行多步任务，但不能替代审查和专业判断。

动作：打开 [概览与安装](01-overview-and-setup.md)，展示本期导航和安全章入口。

### 阶段 1：内联建议

目标：展示“打字时灰色建议，Tab 接受”。

动作：新建 `demo.py`，输入：

```python
def calculate_area(width, height):
```

预期：出现函数体建议。接受后强调“建议需要人审”。如果没有建议，检查登录、文件语言模式和网络；转到 Chat 演示。

### 阶段 2：Chat 与 Inline Chat

目标：展示提问、解释代码和就地修改。

```text
解释当前文件的作用，用非程序员也能理解的语言说明。
```

```text
目标：给选中的函数添加输入校验。
上下文：当前选中函数。
约束：保持函数签名不变，不引入新依赖。
验收：空值输入不会抛出未处理异常。
```

如果 Inline Chat 不弹出，使用 Chat View 并附加 `#selection`。

### 阶段 3：上下文引用

目标：说明好结果来自好上下文。

```text
目标：解释 #file 中的数据流。
上下文：选择当前 demo 文件，必要时添加 #selection。
约束：先列出入口和输出，再解释每一步。
验收：输出 Markdown 表格，包含函数名、输入、输出、风险。
```

若某个 `#` 引用不存在，按输入框自动补全选择当前客户端支持的上下文。

### 阶段 4：Plan -> Implement -> Review

目标：展示复杂任务先规划、再执行、最后审查。

```text
目标：给 demo 脚本增加命令行参数 --dry-run。
上下文：#file 当前 demo 脚本。
约束：先输出计划，不要直接修改；保持默认行为不变。
验收：计划列出要改的函数、测试点和回滚方式。
```

```text
按刚才批准的计划实现。完成后运行最小验证，并列出 diff 中需要我重点审查的地方。
```

如果 Agent 偏离范围，使用 Steer 或 Stop，并回到计划阶段。

### 阶段 5：Instructions 与 Prompt File

目标：展示如何减少重复说明。

动作：创建 `.github/copilot-instructions.md`，写入 3-5 条项目约定；再展示 `.github/prompts/*.prompt.md` 如何变成可复用命令。

```markdown
---
description: 为指定脚本生成边界测试建议
---
目标：为 ${input:file} 生成测试建议。
约束：只输出建议，不修改文件。
验收：按正常输入、空输入、异常输入分组。
```

### 阶段 6：MCP 概念展示

目标：说明 MCP 是外部工具来源，不等于所有工具。

动作：展示已安装 Playwright MCP Server 的工具列表；只做无风险页面截图或读取页面标题。不要现场安装未知 Server。

### 阶段 7：Q&A 与排错

目标：把问题落到安全、权限、上下文和日志入口。

动作：打开 [安全与排错](07-safety-and-troubleshooting.md)，按症状表回答问题。

## Prompt 速查

```text
目标：解释当前文件给非程序员听。
上下文：#file 当前文件。
约束：不要改文件；不要使用未解释的术语。
验收：输出 5 条以内的要点。
```

```text
目标：把选中函数改得更健壮。
上下文：#selection。
约束：保持函数签名和外部行为；新增必要的错误处理。
验收：列出修改点和需要人工审查的风险。
```

```text
目标：为当前变更生成提交说明。
上下文：#changes。
约束：按功能、修复、文档分类。
验收：输出一段中文 commit message 和三条 PR 摘要。
```

---

返回：[VS Code Copilot 团队参考手册](README.md)