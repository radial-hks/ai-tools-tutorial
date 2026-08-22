# Matt Pocock 与 GitHub 生态

## 本章解决什么

讲清楚三件事：**Matt Pocock 是谁**、**他的 GitHub 上有什么**、**为什么本期教程值得你花时间读**。读完你可以判断自己需要精读哪些章节。

## Matt Pocock 是谁

**Matt Pocock 是全球最知名的 TypeScript 教育者之一，Total TypeScript 创始人。**

他的影响力来自三个方面：

| 维度 | 说明 | 对你意味着什么 |
| --- | --- | --- |
| **TypeScript 教育** | 创建 Total TypeScript 课程平台，数以万计的开发者通过他的课程系统学习 TS | 他的代码质量和方法论经过大规模生产验证 |
| **开源贡献** | 曾是 XState、Zod 等核心 TypeScript 生态项目的核心维护者 | 他对「好代码长什么样」有深刻理解 |
| **AI 工程实践** | 2025 年起全力投入 AI 编程工具链建设，开源了 228k+ stars 的 Skills 仓库 | 他是全球最早系统性思考「Agent 怎么写出好代码」的人之一 |

> 💡 **一句话记住他**：他把几十年软件工程经验"蒸馏"成了一套 Agent 能用的技能库——让 AI 不只是"写代码"，而是「用正确的工程方法写代码」。

## 他的 GitHub 上有什么

Matt Pocock 的 GitHub（<https://github.com/mattpocock>）有 100+ 个公开仓库。我们按实用价值分成四层：

```text
                    ┌─────────────────────────────┐
                    │  Skills for Real Engineers  │  ← 核心（228k+ ★）
                    │  22 个 Agent Skill          │
                    ├─────────────────────────────┤
                    │  AI Coding Dictionary       │  ← 基础层（3.9k ★）
                    │  术语扫盲                    │
                    ├──────────────┬──────────────┤
                    │  Sandcastle  │  Evalite     │  ← 工具层
                    │  沙箱编排    │  LLM 评测    │
                    ├──────────────┴──────────────┤
                    │  课程/实验/辅助仓库          │  ← 参考层
                    │  (course-video-manager 等)   │
                    └─────────────────────────────┘
```

### 第一层：Skills for Real Engineers（核心）

**仓库地址：** <https://github.com/mattpocock/skills>

这是本期的核心讲解对象。一句话定义：

> 一套可复用的 **Agent Skill 集合**，涵盖软件工程全流程：从需求澄清 → 原型验证 → 规范编写 → 任务拆分 → TDD 实现 → 代码审查 → 架构改进 → Bug 诊断。

22 个 Skill 被组织成三个目录：

| 目录 | 用途 | 包含 Skill 数 | 谁调用 |
| --- | --- | --- | --- |
| `skills/engineering/` | 日常编码工作 | 17 个 | 用户手动 + 模型自动 |
| `skills/productivity/` | 非编码辅助 | 6 个 | 仅用户手动 |
| `skills/misc/` | 低频工具 | 4 个 | 按需 |

> 💡 第 2 章会对每个 Skill 做分类详解；第 3 章展示它们如何在实际场景中串联工作。

### 第二层：AI Coding Dictionary（基础）

**仓库地址：** <https://github.com/mattpocock/dictionary-of-ai-coding>

一本「AI 编程术语词典」——把 Token、Context Window、Inference、Effort、Prefix Cache 等 60+ 个术语翻译成人话。

> ⚠️ 这对团队里**非技术同事**特别有用：读完后，AI 编程不再是一团黑话。

详见第 4 章。

### 第三层：Sandcastle + Evalite（工具）

| 仓库 | 星数 | 一句话 | 适合谁 |
| --- | --- | --- | --- |
| [Sandcastle](https://github.com/mattpocock/sandcastle) | 7.5k | 在隔离沙箱中编排多个 AI 编码 Agent，自动合并分支 | 需要批量跑 Agent 任务的开发者 |
| [Evalite](https://github.com/mattpocock/evalite) | 1.6k | 用 TypeScript 评测 LLM 应用的输出质量 | 需要评估 AI 输出质量的开发者 |
| [agent-rules-books](https://github.com/mattpocock/agent-rules-books) | 418 | 将 Clean Code、DDD、Refactoring 等经典书籍的规则提炼成 AGENTS.md | 想在项目中引入工程纪律的团队 |

详见第 4 章。

### 第四层：课程 / 实验 / 辅助仓库（参考）

| 仓库 | 说明 |
| --- | --- |
| [total-typescript-monorepo](https://github.com/mattpocock/total-typescript-monorepo) | Total TypeScript 课程平台的内部工具链 |
| [course-video-manager](https://github.com/mattpocock/course-video-manager) | 课程视频管理系统（671 ★），展示了 CONTEXT.md + ADR 的实际用法 |
| [node-DeepResearch](https://github.com/mattpocock/node-DeepResearch) | 深度研究 Agent 的 Node.js 实现（84 ★） |
| [ai-hero-cli](https://github.com/mattpocock/ai-hero-cli) | AI 实验工具集合（124 ★） |

这些仓库主要适合想深入研究 AI Agent 工程化的人参考。

## 为什么这套 Skills 值得关注

在 Matt Pocock 之前，社区对「Agent 技能」的理解停留在「写一段 prompt 指导 AI」。他的核心贡献是把**软件工程纪律**注入了 Agent 行为：

| 他解决了什么问题 | 具体怎么做 |
| --- | --- |
| Agent 不懂你真正要什么 | `/grill-me` / `/grill-with-docs`：先盘问需求，建立共识语言，再动手 |
| Agent 写的代码跑不通 | `/tdd`：红→绿→重构循环；`/diagnosing-bugs`：结构化诊断流程 |
| Agent 把代码堆成屎山 | `/improve-codebase-architecture`：定期扫描；`/codebase-design`：模块设计词汇表 |
| Agent 一次改太多改坏了 | `/to-spec` → `/to-tickets` → `/implement`：拆成小步 tracer bullet |
| 团队没有统一的 AI 工作方式 | 一整套可安装、可定制的 Skill 文件，团队共享 |

> 💡 用一句话概括他的哲学：**「反馈回路优先」**——没有能复现的命令，不许谈假设。每一个 Skill 的核心都是"先建立一个能快速验证的反馈循环"。

## 和前 6 期教程的关系

| 前 6 期 | 本期怎么衔接 |
| --- | --- |
| 第 1 期 Copilot | 这些 Skill 直接可在 Copilot Chat / Agent 模式下使用 |
| 第 2 期 Copilot Skill | 本期是**真实世界的 Skill 最佳实践**——用 22 个 Skill 展示什么叫"好 Skill" |
| 第 3 期 Hermes | Hermes 也支持 Skill，安装方式类似（`npx skills@latest add`） |
| 第 4 期 Pi | Pi + Claude Code 路径可直接用 Claude Code 插件安装 |
| 第 5 期 Agent 记忆 | 这些 Skill 本身不涉及长期记忆，但 `CONTEXT.md` 和 ADR 起到了"项目级记忆"的作用 |
| 第 6 期 DSH | DSH 的动态插件机制和 Skill 是不同层面的扩展：Skill 是"教 Agent 做事的方法"，DSH Plugin 是"给 Agent 添加能力" |

## 下一步

想了解每个 Skill 的具体功能和适用场景？看 [第 2 章：Skills 仓库深度解析](02-skills-engineering.md)。想直接看实际怎么用？跳到 [第 3 章：场景实战](03-scenarios-workflows.md)。

---

← 上一章：[本期首页](README.md) ｜ 下一章：[Skills 仓库深度解析](02-skills-engineering.md)