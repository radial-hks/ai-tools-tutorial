# AI Coding Dictionary 与配套工具

## 本章解决什么

讲清楚 Matt Pocock 生态中除 Skills 之外的**三个重要配套资源**：AI Coding Dictionary（术语词典）、Sandcastle（沙箱编排）、Evalite（LLM 评测），以及若干辅助仓库。读完后你可以判断哪些对团队有用。

---

## 一、AI Coding Dictionary 📖

**仓库地址：** <https://github.com/mattpocock/dictionary-of-ai-coding>
**在线版：** <https://aicodingdictionary.com>
**星数：** 3,905 ★

### 是什么

一本**用大白话解释 AI 编程术语**的词典——60+ 个术语，分成 7 个章节。每个术语用一两段人话讲清楚，不预设你懂机器学习。

### 为什么对团队重要

| 痛点 | 有了词典之后 |
| --- | --- |
| 开会时听到 "Token"、"Context Window"、"Inference" 一头雾水 | 翻词典，30 秒搞懂一个术语 |
| 看 AI 工具的定价页，不明白 "Input Tokens" 和 "Output Tokens" 为什么价格不同 | 「输入 Token」和「输出 Token」有清晰的定义和区别 |
| Agent 说 "context is degrading" 但你不知道什么意思 | 查 "Context" + "Context Window" → 明白是「对话太长，AI 开始忘记开头说了什么」 |
| 不知道 "Prefix Cache" 是什么、能不能省钱 | 查词典 → 理解后用缓存省 Token 费用 |

### 词典结构速览

| 章节 | 术语数量 | 核心内容 | 谁该读 |
| --- | --- | --- | --- |
| Section 1 — The Model | 15 个 | AI、Model、Token、Inference、Parameters、Effort 等 | 所有人 |
| Section 2 — Sessions, Context Windows & Turns | 8 个 | Context、Context Window、Session、Turn、Stateless 等 | 所有人 |
| Section 3 — Tools & Environment | 5 个 | Tool、Tool Call、Filesystem、Environment 等 | 技术人员 |
| Section 4 — Agents & Skills | 7 个 | Agent、Skill、Subagent、Harness 等 | 技术人员 |
| Section 5 — Editing, Generation & Review | 8 个 | Spec-Driven Development、TDD、Code Review 等 | 技术人员 |
| Section 6 — Failure Modes | 8 个 | Hallucination、Drift、Context Degradation 等 | 所有人 |
| Section 7 — Advanced | 15 个 | Embedding、RAG、Fine-tuning、MCP 等 | 技术人员 |

### 关键术语举例

| 术语 | 词典解释（翻译） |
| --- | --- |
| **Token** | AI 读写的「最小文字单元」。不是字符、不是单词——大约 1 个 Token ≈ 0.75 个英文单词 ≈ 0.5 个汉字。ChatGPT 说「你好」，算 3 个 Token |
| **Context Window** | AI 的「短期记忆容量」。一次能「看见」多少 Token。超过就「忘记」开头。GPT-4o 的 Context Window 是 128K Token——大约一本中篇小说 |
| **Inference** | AI 「思考并输出」的过程。你发消息 → AI 推理 → 返回结果，这个过程叫一次 Inference |
| **Effort** | 你要求 AI「多想一会」的程度。高 Effort = AI 花更多计算量深入思考。OpenAI o 系列模型的核心概念 |
| **Prefix Cache** | 「AI 发现你的开头和上次一样，跳过重复计算」。省钱技巧：把固定指令（System Prompt）放在最前面，后续请求会命中缓存 |
| **Hallucination** | AI 「一本正经地胡说八道」。原因是它擅长预测「下一个 Token 像什么」，不是「什么是对的」 |
| **RAG** | Retrieval-Augmented Generation：AI 在回答前先查资料库，把查到的东西和你的问题一起喂给模型。减少幻觉的主要手段 |
| **TDD** | Test-Driven Development：先写一个失败的测试 → 写最少代码让它通过 → 重构。本词典里最重要的工程术语之一 |

> 💡 **建议团队用法：** 把在线版链接放在团队 Wiki 首页——非技术同事遇到听不懂的 AI 术语时，30 秒自服务查词典。不需要开 AI 对话，不需要问别人。

---

## 二、Sandcastle 🏰

**仓库地址：** <https://github.com/mattpocock/sandcastle>
**星数：** 7,536 ★

### 是什么

一个 TypeScript 库，用 **`sandcastle.run()` 一个方法**，在隔离沙箱中编排多个 AI 编码 Agent。Agent 在独立分支上工作，完成后自动合并回主干。

### 解决什么问题

当你需要同时让多个 Agent 做不同的任务（比如并行修三个 Bug），手动管理 Git 分支、切换上下文、合并冲突是一个噩梦。Sandcastle 自动处理这些。

### 工作原理

```text
sandcastle.run([
  { agent: "fix-login-bug",    branch: "fix/login-bug" },
  { agent: "add-export-csv",   branch: "feat/export-csv" },
  { agent: "refactor-scanner", branch: "refactor/scanner" },
])

→ Sandcastle 自动：
  1. 为每个任务创建隔离沙箱（Docker / Podman / Vercel）
  2. 在每个沙箱里让 Agent 在独立分支上工作
  3. Agent 完成后，自动合并分支
  4. 有冲突时通知你手动解决
```

### 支持的沙箱类型

| 沙箱类型 | 说明 | 适用场景 |
| --- | --- | --- |
| Docker | 本地 Docker 容器 | 日常本地开发 |
| Podman | 无 root 权限的 Docker 替代 | 企业环境有权限限制时 |
| Vercel Sandbox | 云端 Firecracker microVM | CI/CD 流水线 |
| 自定义 | 用 `createIsolatedSandboxProvider` 自己实现 | 特殊需求 |

### 团队适用判断

| 条件 | 判断 |
| --- | --- |
| 团队有多个 Agent 同时工作 | ✅ 适合，Sandcastle 就是为这个设计的 |
| 需要 Agent 产出的代码自动合并 | ✅ 适合 |
| 只在本地用一个 Agent | ❌ 不需要，手动 git 足够 |
| 团队不熟悉 Docker | 🟡 需要先学 Docker 基础 |

> 💡 Sandcastle 是进阶工具——如果你还没用熟单个 Agent 的 Skill，建议先用好 Skills 仓库，需要并行 Agent 时再引入 Sandcastle。

---

## 三、Evalite 📊

**仓库地址：** <https://github.com/mattpocock/evalite>
**星数：** 1,661 ★

### 是什么

用 TypeScript 写**评测用例**来验证 LLM 应用的输出质量。类似给 AI 输出写单元测试。

### 解决什么问题

当你用 AI 做关键任务（比如自动分类素材、生成命名），你不能靠「看着还行」来判断——你需要可重复的、自动化的评测。

### 适用场景举例

| 场景 | Evalite 怎么用 |
| --- | --- |
| AI 自动分类 3D 素材类型 | 准备 100 个已知类型的素材 → 跑 AI 分类 → 用 Evalite 算准确率 |
| AI 生成文件命名 | 定义命名规则 → AI 生成 1000 个命名 → 用 Evalite 检查格式合规率 |
| 换了新模型，不知道效果变好还是变差 | 用 Evalite 跑同一套评测 → 对比得分 |

> 💡 Evalite 适合**需要评估 AI 输出质量的场景**。如果你的 AI 用法只是「聊天→写代码→你人工看」，暂时不需要。

---

## 四、其他辅助仓库速览

### agent-rules-books（418 ★）

**仓库地址：** <https://github.com/mattpocock/agent-rules-books>

将 Clean Code、Refactoring、DDD、Clean Architecture、DDIA 等经典编程书籍的核心规则，提炼成 Agent 可读的 `AGENTS.md` 格式。

> 💡 如果你想让团队的 Agent 遵循特定的代码规范（而不是每次对话口述一遍），可以参考这个仓库的做法——把规则写成 `AGENTS.md`。

### course-video-manager（671 ★）

**仓库地址：** <https://github.com/mattpocock/course-video-manager>

Matt Pocock 自己用的课程视频管理系统。这个仓库不是给你用的，但它是 `CONTEXT.md` + ADR 实践的**真实范例**——你可以看他的 `CONTEXT.md` 是怎么写的、ADR 是怎么记录的。

### node-DeepResearch（84 ★）

**仓库地址：** <https://github.com/mattpocock/node-DeepResearch>

一个 Node.js 实现的深度研究 Agent——不断搜索、阅读网页、推理，直到找到答案或超出 Token 预算。如果你对「Agent 怎么自己查资料」感兴趣，可以看源码学习。

### ai-hero-cli（124 ★）

**仓库地址：** <https://github.com/mattpocock/ai-hero-cli>

Matt Pocock 的 AI 实验工具集合。包含各种小工具和实验性功能——适合想了解 AI 工具前沿玩法的人。

---

## 工具选型决策表

| 你的需求 | 用这个 | 优先级 |
| --- | --- | --- |
| 让 AI 按工程纪律写代码 | Skills 仓库 | ⭐⭐⭐ 首选 |
| 理解 AI 编程术语 | AI Coding Dictionary | ⭐⭐⭐ 基础 |
| 并行编排多个 Agent | Sandcastle | ⭐⭐ 进阶 |
| 评估 AI 输出质量 | Evalite | ⭐⭐ 进阶 |
| 给 Agent 注入工程规则 | agent-rules-books（参考） | ⭐ 参考 |
| 学习 Agent 工程实践 | course-video-manager（参考） | ⭐ 参考 |

---

← 上一章：[场景实战：从想法到交付](03-scenarios-workflows.md) ｜ 下一章：[团队落地指南](05-team-adoption.md)