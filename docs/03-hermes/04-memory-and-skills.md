# 记忆与技能系统

## 本章解决什么

Hermes 有两个越用越顺手的能力：**Memory（记忆）** 和 **Skill（技能）**。

- Memory 让 Hermes 记住少量长期重要的信息。
- Skill 让 Hermes 在特定任务中加载一套成熟做法。

> 💡 简单比喻：Memory 像助理的小笔记本；Skill 像助理的工作手册或 SOP。

---

## Memory（记忆）

### 一句话解释

**Memory 让 Hermes 记住你告诉过它的长期偏好和环境信息，下次不用重复。**

例如：

- “我的贴图默认输出 JPG，质量 90。”
- “这个项目的素材测试目录是 `D:/Projects/Demo/export-test`。”
- “批量处理前必须先列计划，不要直接覆盖原文件。”

### Memory 适合记什么

| 类型 | 好例子 | 不适合 |
| --- | --- | --- |
| 个人偏好 | “我喜欢简洁步骤，不要长篇解释。” | 一次性闲聊 |
| 项目约定 | “贴图命名使用小写加下划线。” | 临时文件列表 |
| 环境信息 | “Maya 安装在指定路径。” | 密码、Token、API Key |
| 反复纠正 | “以后不要直接删除原文件，先备份。” | 当天临时进度 |

### 怎么让 Hermes 记住

最自然的方式是在对话里直接说：

```text
请记住：我处理贴图时，默认先在测试目录试跑，不要直接覆盖正式目录。
```

Hermes 会在合适时使用自己的 `memory` 工具保存这类信息。

> ⚠️ Memory 不是无限笔记本。官方默认有字符上限，目的是只保留真正长期有用的信息。

### 怎么查看和控制 Memory

常见做法：

| 需求 | 方式 |
| --- | --- |
| 配置外部记忆提供方 | `hermes memory setup` |
| 查看记忆系统状态 | `hermes memory status` |
| 查看 Hermes 的学习轨迹 | `hermes journey` |
| 如果启用了写入审批 | 在会话里用 `/memory pending`、`/memory approve`、`/memory reject` |

> 初学者不需要先学会所有管理命令。最重要的是知道：可以要求 Hermes 记住偏好，也可以要求它删除或更正错误记忆。

### 隐私边界

⚠️ 这一点非常重要：

- Memory 文件保存在本机 Hermes 数据目录中。
- 但当 Hermes 使用远程模型时，相关记忆会作为上下文发送给模型服务商。
- 所以 **不要把密码、API Key、Token、身份证号、客户敏感数据写进 Memory**。

如果你不确定某条信息能不能记，按更保守的原则处理：不要记；需要时临时告诉 Hermes。

---

## Skill（技能）

### 一句话解释

**Skill 是一份可复用的任务说明书，让 Hermes 在遇到特定任务时知道成熟流程、注意事项和验证方法。**

它不一定是“代码插件”。很多 Skill 本质上是 Markdown 文档：告诉 Hermes 什么时候使用、按什么步骤做、有哪些坑、如何验收。

### Skill 能帮什么

| 技能类型 | 例子 |
| --- | --- |
| 文件处理 | 批量重命名、分类、生成清单 |
| 图片/媒体 | 图片格式转换、截图分析、素材整理 |
| 文档处理 | PDF、Word、Excel、Markdown 处理 |
| 开发流程 | 代码审查、测试驱动开发、GitHub 工作流 |
| 团队 SOP | 发布流程、交付检查清单、项目命名规范 |

### 安装和管理技能

Hermes 的技能命令使用 **`skills` 复数**：

```bash
# 浏览技能
hermes skills browse

# 搜索技能
hermes skills search image

# 安装技能
hermes skills install <技能标识>

# 查看已安装技能
hermes skills list

# 卸载技能
hermes skills uninstall <技能名>
```

安装后，很多技能可以在会话中用 Slash Command 调用：

```text
/skill-name 帮我处理这个任务
```

也可以直接用自然语言描述任务，Hermes 会在需要时加载匹配的技能。

### Skill 和 Copilot 怎么配合

- 🤖 **Copilot 帮你找或写 Skill 草稿**：你描述团队流程，让 Copilot 帮你整理成“触发条件、步骤、注意事项、验收”。
- 🤖 **Hermes 使用 Skill 执行任务**：真正执行时由 Hermes 按 Skill 调用工具。
- ⚠️ **你审查 Skill 是否符合团队流程**：尤其是涉及删除、覆盖、上传、对外发送的流程。

### 自定义技能的基本结构

一个标准 Skill 通常是一个 `SKILL.md` 文件，包含：

```markdown
---
name: example-skill
description: Use when ...
---

# Example Skill

## When to Use

什么时候使用。

## Procedure

具体步骤。

## Pitfalls

常见坑。

## Verification

如何确认成功。
```

> ⚠️ 技能名建议使用小写英文和连字符，例如 `texture-batch-check`。复杂技能请交给技术同事审查后再共享。

---

## Memory 和 Skill 的区别

| | Memory（记忆） | Skill（技能） |
| --- | --- | --- |
| 作用 | 记住少量长期事实 | 复用一套工作流程 |
| 比喻 | 助理的小笔记本 | 助理的 SOP 手册 |
| 典型内容 | 偏好、路径、约定 | 步骤、命令、坑、验收 |
| 生效方式 | 会话开始时注入上下文 | 需要时按技能加载 |
| 风险 | 记错会长期影响判断 | 步骤写错会重复执行错流程 |
| 管理入口 | `hermes memory ...`、`hermes journey`、`/memory ...` | `hermes skills ...`、`/skills ...`、`/skill-name` |

## ⚠️ 你必须亲自决定的事

| 决策环节 | 为什么必须你来 |
| --- | --- |
| 让 Hermes 记住什么 | 只有你知道哪些信息长期有效 |
| 是否删除或更正记忆 | 错误记忆会影响未来任务 |
| 安装哪些 Skill | 装太多不相关技能会增加噪音 |
| Skill 是否符合团队规范 | 自动化流程需要人工审查 |
| 是否把 Skill 分享给团队 | 团队级流程会影响其他人 |

## 官方参考

- Persistent Memory: <https://hermes-agent.nousresearch.com/docs/user-guide/features/memory>
- Skills System: <https://hermes-agent.nousresearch.com/docs/user-guide/features/skills>
- Skills Catalog: <https://hermes-agent.nousresearch.com/docs/reference/skills-catalog>

---

← 上一章：[使用 Hermes](03-cli-tui-gateway.md) ｜ 下一章：[Hermes 与 Copilot](05-hermes-vs-copilot.md)
