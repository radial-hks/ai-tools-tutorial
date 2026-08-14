# Agent 记忆系统团队参考手册

**受众:** 团队成员（含美术、建模同事）。本期讲"Agent 怎么记住东西"，不需要编程基础；想动手实测的同事读附录即可。

**核验基线:** 内容基于三个开源项目在 2026-08 的官方文档与源码快照核验，并做了本机实测（macOS）。快速迭代项目请以官方最新文档为准，发布前建议技术负责人按团队环境复核。

**一句话定位:** 前 4 期讲了"用工具写代码（Copilot）、复用流程（Skill）、执行任务（Hermes）、多 Agent 协作（Pi）"。本期补上缺的那一环——**记忆层**：Agent 怎么记住你的偏好、团队的知识和项目的经验，让你下次不用从头教起。

**标记约定:** 沿用全手册的两个符号：

- 🤖 **AI 可替代**：可以让 Copilot 或 Hermes 帮你解释、生成命令、排查错误。
- ⚠️ **必须由你决定**：涉及账号、密钥、隐私、是否把数据放云端等，需要你亲自判断。

## 按任务查找

| 任务类型 | 目标文档 |
| --- | --- |
| 理解"Agent 记忆"到底是什么、为什么需要 | [01-why-agent-memory.md](01-why-agent-memory.md) |
| 想知道三家主流记忆系统选哪个 | [02-memory-systems-comparison.md](02-memory-systems-comparison.md) |
| 已经在用 Hermes，想给它接更强记忆 | [03-relation-to-hermes-memory.md](03-relation-to-hermes-memory.md) |
| 关心数据放哪、会不会外泄、要花多少钱 | [04-privacy-and-deployment.md](04-privacy-and-deployment.md) |
| 想看真实安装命令和踩坑记录 | [appendix-hands-on.md](appendix-hands-on.md) |

## 章节导航（估算阅读时间）

| 章 | 标题 | 内容 | 估计阅读时间 |
| ---: | --- | --- | ---: |
| 1 | [为什么 Agent 需要长期记忆](01-why-agent-memory.md) | 四类记忆资产（Chat Memory / Skill / Wiki / CodeGraph）与 L0–L3 蒸馏 | 8 分钟 |
| 2 | [三个记忆系统横向对比](02-memory-systems-comparison.md) | TencentDB Agent Memory vs Mnemosyne vs Hindsight | 12 分钟 |
| 3 | [与 Hermes Memory 的关系](03-relation-to-hermes-memory.md) | 从"记什么"到"记忆架构选型"，怎么给本机 Hermes 接记忆插件 | 10 分钟 |
| 4 | [隐私与部署权衡](04-privacy-and-deployment.md) | local-first vs 云端、成本、隐私红线 | 8 分钟 |
| 附 | [本机实测记录](appendix-hands-on.md) | 三家系统真实安装命令、输出与踩坑 | 20 分钟 |

## 阅读建议

- **只想搞懂概念**：读 [第 1 章](01-why-agent-memory.md)。
- **要选型**：先读 [第 2 章](02-memory-systems-comparison.md) 的对比表，再按需深入。
- **团队已用 Hermes**：读 [第 3 章](03-relation-to-hermes-memory.md)。
- **动手党**：直接翻 [附录](appendix-hands-on.md)，跟着命令走一遍。

## 三个系统一句话速览

| 系统 | 一句话 | 部署形态 |
| --- | --- | --- |
| **TencentDB Agent Memory** | 腾讯云开源的"团队记忆中枢"，把对话、技能、文档、代码四类资产统一管理 | Docker 三件套，纯本地 |
| **Mnemosyne** | zero-cloud 记忆层：一个 pip 包 + 一个 SQLite 文件，Hermes/Pi 原生支持 | 纯 Python，零服务 |
| **Hindsight** | 主打"让 Agent 学习而不只是记住"，Retain / Recall / Reflect 三阶段闭环 | Docker 或嵌入式 Python |

## 官方基线参考

- TencentDB Agent Memory: <https://github.com/TencentCloud/TencentDB-Agent-Memory>（中文文档：README_CN.md / INSTALL_CN.md）
- Mnemosyne: <https://github.com/mnemosyne-oss/mnemosyne> ｜ 官网 <https://mnemosyne.site/>
- Hindsight: <https://github.com/vectorize-io/hindsight> ｜ 文档 <https://hindsight.vectorize.io/> ｜ 论文 <https://arxiv.org/abs/2512.12818>

> ⚠️ **内部发布提示：** 三家项目迭代都很快（核验期间 TencentDB 三周三个版本）。面向同事发布前，请技术负责人复核版本号、LLM 供应商配置与团队部署形态，并把团队专属配置补充到 [第 4 章](04-privacy-and-deployment.md)。

---

← 返回主页：[AI 工具团队参考手册](../../README.md) ｜ 下一章：[为什么 Agent 需要长期记忆](01-why-agent-memory.md)
