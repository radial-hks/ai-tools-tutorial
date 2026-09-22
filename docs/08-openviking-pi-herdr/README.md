# OpenViking + Pi + herdr 多 Agent 组合 —— 团队参考手册

**受众：** 技术人员建议精读全文；无编程基础的美术、建模、TA 同事重点读第 1 章（了解"有什么"）和第 4 章前半段（场景速查），了解"这套组合能干什么、什么时候用"即可。

**核验基线：** 内容按本机实际环境与官方文档核验于 2026-09（OpenViking v0.4.x、pi-herdr-agents v2.0.x、pi + herdr）。三者迭代都很快，发布前建议技术负责人复核版本号与团队部署形态。

**一句话定位：** 前七期讲了"用工具写代码、复用流程、执行任务、多 Agent 协作、长期记忆"。本期把这套拼图合起来：用 **OpenViking** 给多个 agent 装上同一份跨会话长期记忆，用 **Pi** 做父会话编排，用 **herdr** 承载并行子代理，串成一条「记忆 → 编排 → 执行 → 沉淀」的完整闭环。

**标记约定：** 沿用全手册的符号：

- 🤖 **AI 可替代**：让 Agent 帮你执行这一步。
- ⚠️ **必须由你决定**：账号、密钥、路径、最终效果、安全确认等，需要你亲自判断。
- 💡 **小提示**：补充说明、记忆口诀或可选做法。

## 三件套一句话分工

| 工具 | 大白话 | 你能用它做什么 |
| --- | --- | --- |
| **OpenViking** | 跨会话的长期记忆库 | 每个 prompt 前自动召回相关记忆、每个 turn 后自动沉淀，让多个 agent 共享同一份记忆 |
| **Pi**（`pi`） | 终端里的编码 agent | 父会话拆解任务、派发子代理、综合审查合并、最终落盘提交 |
| **herdr** | 终端工作区管理器 | 把多个子代理放进独立 pane / worktree 并行跑，一眼看到「另一个面板在跑什么」 |

## 按任务查找

| 任务类型 | 目标文档 |
| --- | --- |
| 搞清楚 OpenViking 是什么、解决什么问题 | [01-what-is-openviking.md](01-what-is-openviking.md) |
| 装好 OpenViking 服务并把 Pi 接上记忆扩展 | [02-installation.md](02-installation.md) |
| 用 Pi + herdr 做多 Agent 编排（派子代理、worktree、并行） | [03-pi-herdr-multiagent.md](03-pi-herdr-multiagent.md) |
| 跟完三件套端到端闭环 + 两个团队场景实例 | [04-combo-workflow.md](04-combo-workflow.md) |
| 主力用 Copilot，想给自己的 Copilot 装上 OpenViking 记忆 | [05-openviking-copilot.md](05-openviking-copilot.md) |

## 章节导航（估算阅读时间）

| 章 | 标题 | 内容 | 受众 | 估计阅读时间 |
| ---: | --- | --- | --- | ---: |
| 1 | [OpenViking 是什么](01-what-is-openviking.md) | 定位、核心概念（Resource / Memory / Skill、L0–L2 分层）、为什么 Agent 需要它 | 全员 | 8 分钟 |
| 2 | [安装 OpenViking 并接入 Pi](02-installation.md) | 部署服务、安装 pi 扩展、配置凭据与 peer、验证召回 | 技术人员 | 15 分钟 |
| 3 | [Pi + herdr 多 Agent 编排](03-pi-herdr-multiagent.md) | `subagent` / `/subagent` / `/worktree`、角色、并行与受管 worktree | 技术人员 | 12 分钟 |
| 4 | [三件套组合工作流](04-combo-workflow.md) | 端到端闭环 + 多视角调研 / 批量文档两个场景实例 | 全员 | 10 分钟 |
| 5 | [OpenViking × Copilot](05-openviking-copilot.md) | openviking-copilot 插件安装配置、已知坑，主力 Copilot 同事的记忆路线 | Copilot 用户 + 技术人员 | 10 分钟 |

## 阅读建议

- **只想了解「有什么、要不要用」**：读 [第 1 章](01-what-is-openviking.md) + [第 4 章](04-combo-workflow.md) 的「三件套分工」和「两个团队场景」，共约 15 分钟。
- **想直接上手跑通闭环**：按顺序读 [第 2 章](02-installation.md) → [第 3 章](03-pi-herdr-multiagent.md) → [第 4 章](04-combo-workflow.md)。
- **已经在用第 4 期的 Pi + herdr**：跳过第 1 章，直接看 [第 3 章](03-pi-herdr-multiagent.md) 的编排用法，再回 [第 4 章](04-combo-workflow.md) 看闭环。
- **日常主力是 Copilot（公司内部大多数同事）**：读 [第 1 章](01-what-is-openviking.md) 搞懂记忆库，再直接看 [第 5 章](05-openviking-copilot.md) 装插件；第 2–4 章的 Pi 路线可作为以后进阶的参考。
- **之前没接触过 Pi / 记忆系统**：先回顾 [第 4 期](../../docs/04-pi/README.md) 和 [第 5 期](../../docs/05-agent-memory/README.md)，再回本期。

## 官方基线参考

- OpenViking：<https://github.com/volcengine/OpenViking> ｜ 官方文档 <https://docs.openviking.ai/>（中文 <https://docs.openviking.ai/zh/getting-started/01-introduction>）
- Pi：<https://github.com/earendil-works/pi> ｜ npm 包 <https://www.npmjs.com/package/@earendil-works/pi-coding-agent>
- herdr：<https://herdr.dev>
- pi-herdr-agents：<https://github.com/giuseppecrj/pi-herdr-agents>

> ⚠️ **内部发布提示：** 三者迭代都很快（核验期间 OpenViking 处于 v0.4.x、pi-herdr-agents v2.0.x）。面向同事发布前，请技术负责人复核版本号、模型路由与团队部署形态，并把团队专属配置补充到 [第 2 章](02-installation.md) 和 [第 3 章](03-pi-herdr-multiagent.md)。

---

← 返回主页：[AI 工具团队参考手册](../../README.md) ｜ 下一章：[OpenViking 是什么](01-what-is-openviking.md)
