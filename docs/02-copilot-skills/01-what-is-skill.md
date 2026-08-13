# 什么是 Copilot Skill

## 本章解决什么

用最通俗的话讲清楚三件事：**技能（Agent Skill）** 到底是什么、它解决什么问题、它和第 1 期已经介绍过的 Instructions / Prompt File / Custom Agent 有什么区别。读完后你可以判断「我该不该用 Skill」。

> 💡 第 1 期 [定制 Copilot](../../docs/01-copilot/05-customization.md) 里有一小段 Skill 的简要介绍，本章是它的**深度展开**；两篇可以对照着看，但本章不重复那些基础内容。

## 一句话说清楚

**Agent Skill 是一个「可复用能力包」：一个文件夹，里面有一份 `SKILL.md` 说明文件，再加上完成这件事需要的脚本、模板、示例等资源。Copilot 在遇到相关任务时，会按需自动加载它。**

用生活化的方式理解：

> 普通聊天里的每一次提问，都像你**当场口述**一遍做事流程，说完就忘。
>
> Skill 则像一本**贴好标签的手册**：你把「某类活怎么干」预先写成手册，还把手册要用的**工具箱**（脚本、模板）一起放进一个文件夹。以后你一提相关需求，Copilot 就自动抽出这本手册照着干；你也可以直接喊一句命令把它调出来。

## Skill 解决什么问题

对美术、建模同事来说，Skill 主要解决「**重复交代 + 交代不清**」两类麻烦：

| 痛点 | 没有 Skill 时 | 有 Skill 时 |
| --- | --- | --- |
| 每次都要重新解释流程 | 每次聊天都口述一遍「怎么整理贴图」「怎么检查命名」 | 流程写进 `SKILL.md`，Copilot 自动照着做 |
| 步骤漏了关键细节 | AI 凭印象做，可能漏掉团队约定 | 手册写全步骤、坑和验收标准，不易漏 |
| 需要固定模板/脚本 | 临时让 AI 现写，质量不稳定 | 脚本、模板直接放进 Skill 文件夹，随用随取 |
| 想分享给同事复用 | 每个人自己摸索 | Skill 放进仓库，同事共享同一套做法 |

官方对 Skill 给出的四个好处，翻译成大白话：

- **专精（Specialize Copilot）**：让 Copilot 在某类任务上更专业，不用每次重复背景。
- **省重复（Reduce repetition）**：一次写好，所有对话自动复用。
- **可组合（Compose capabilities）**：多个 Skill 拼起来做更复杂的流程。
- **省上下文（Efficient loading）**：只加载相关内容，不浪费 AI 的「脑子容量」。

## Skill 长什么样

一个 Skill 就是一个目录，至少含一个 `SKILL.md`，还可以带其他资源。以官方文档里「网页测试」这个例子为例：

```text
.github/skills/
└── webapp-testing/           ← 目录名 = Skill 的 name
    ├── SKILL.md              ← 必填：说明这个 Skill 干嘛、怎么干
    ├── test-template.js      ← 可选：测试模板脚本
    └── examples/             ← 可选：示例场景
        └── ...
```

`SKILL.md` 顶部是一段 **YAML frontmatter（元信息头）**，写 `name`、`description` 等；下面正文是具体指令。最小示例长这样（完整字段在 [第 3 章](03-create-skill.md) 展开）：

```markdown
---
name: webapp-testing
description: 使用 Playwright 创建和运行网页测试的指南（示例）。当你需要做浏览器测试时使用。
---

# Web Application Testing with Playwright

## 何时使用
当你需要创建、调试网页测试时使用本 Skill。

## 步骤
1. 参考 [测试模板](./test-template.js) 的标准结构
2. 确定要测试的用户流程
3. 在 `tests/` 目录新建测试文件
...
```

## 和 Instructions / Prompt File / Custom Agent 的区别

这是本章重点。第 1 期讲过，Copilot 有多个「定制」工具，名字容易混。一张表先看清 Skill 的独特位置：

| 定制类型 | 一句话 | 什么时候生效 | 能装脚本/资源吗 |
| --- | --- | --- | --- |
| **Instructions（指令）** | 项目规则和编码规范 | 每次都自动带上 | 不能，纯文字 |
| **Prompt File（提示词文件）** | 存好的一条 `/` 命令，手动触发 | 你主动调用 | 不能，纯文字 |
| **Agent Skill（技能）** | 可复用能力包：说明 + 脚本 + 模板 + 资源 | **按需自动加载**，也可手动调用 | ✅ 能 |
| **Custom Agent（自定义智能体）** | 一个「换了人设/工具」的聊天角色 | 你切换过去 | 不能（但可引用 Skill） |

关键就一句：**只有 Skill 是「说明 + 脚本 + 模板 + 资源」的完整能力包，并且能被 Copilot 按任务自动加载。**

官方文档给了一张「Agent Skills vs 自定义指令」的对比表，翻译如下：

| 对比项 | Agent Skill | 自定义指令（Instructions） |
| --- | --- | --- |
| 目的 | 教它**专项能力和工作流** | 定义**编码标准和规范** |
| 可移植性 | 跨 VS Code、Copilot CLI、Copilot 云智能体通用 | 仅 VS Code 和 GitHub.com |
| 内容 | 指令 + 脚本 + 示例 + 资源 | 仅指令文字 |
| 生效范围 | 针对具体任务，**按需加载** | 每次请求都带上（或按文件匹配） |
| 标准 | 开放标准（agentskills.io） | VS Code 专属 |

> 💡 一句话记住四种东西的分工：**Instructions 定「规矩」，Prompt File 存「一条命令」，Skill 装「一套手艺 + 工具箱」，Custom Agent 换「一个人」。**

## 开放标准，跨工具通用

Skill 遵循 **Agent Skills 开放标准**（见 <https://agentskills.io/>）。这意味着你在 VS Code 里写的 Skill，不是「只能给 VS Code 用」：

- **GitHub Copilot in VS Code**：聊天和 Agent 模式可用；
- **GitHub Copilot CLI**：在终端里工作时可访问；
- **GitHub Copilot cloud agent**：自动编码任务中可用。

简单说：**写一次，多处复用。** 这也是它比 Instructions（VS Code 专属）更「通用」的原因。

## 谁适合用 Skill

- ✅ 团队里有**重复出现、步骤固定**的任务（整理资源、检查命名、出报告）。
- ✅ 需要**附带脚本或模板**才能做好的任务。
- ✅ 想**把团队经验沉淀下来**，让新同事一装就能用同一套做法。
- ❌ 只是临时问一句、以后不会再做的事，不值得做成 Skill。

## 适合先从哪些任务开始

1. 从 [第 2 章](02-install-and-use.md) 装一个官方示例 Skill，体验「自动加载 + 手动调用」两种方式。
2. 在 [第 3 章](03-create-skill.md) 照着最小示例改一个自己的 Skill。
3. 在复制出来的测试文件夹里试用，确认结果正确后再放进正式项目。

> ⚠️ Skill 里的脚本和模板**真的会被 Copilot 执行或读取**，安装第三方 Skill 前要先看源码（详见 [第 5 章](05-recommendations-and-practice.md) 的安全提醒）。

## 官方参考

- Agent Skills 官方文档: <https://code.visualstudio.com/docs/agent-customization/agent-skills>
- 定制类型对比: <https://code.visualstudio.com/docs/agents/concepts/customization>
- Agent Skills 开放标准: <https://agentskills.io/>

---

← 上一章：[Copilot Skill 团队参考手册](README.md) ｜ 下一章：[安装与使用现有 Skill](02-install-and-use.md)
