# Hermes Agent 团队参考手册

**受众:** 团队成员，尤其是无编程基础的美术、建模同事。

**核验基线:** 内容已按 Hermes Agent 本地官方文档快照核验于 2026-08-12。发布前仍建议技术负责人按团队实际账号、模型供应商和 Windows 安装包分发方式做最后确认。

**一句话定位:** Hermes Agent 是 Nous Research 推出的个人 AI Agent。它不只是回答问题，还可以在你授权后调用终端、文件、浏览器、技能和其他工具来完成多步任务。

**标记约定:** 全手册用两个符号标注任务分工：

- 🤖 **AI 可替代**：可以让 Copilot 或 Hermes 帮你解释、生成命令、排查错误、整理步骤。
- ⚠️ **必须由你决定**：涉及账号、密钥、路径、最终效果、安全确认、是否覆盖文件等，需要你亲自判断。

> ⚠️ **内部发布提示：** 安装命令已替换为 Hermes Agent 官方文档中的真实命令；但团队内部可能使用统一模型账号、代理、镜像源或指定安装包。面向同事发布前，请把这些团队专属配置补充到第 2 章。

## 按任务查找

| 任务类型 | 目标文档 |
| --- | --- |
| 了解 Hermes 是什么、能做什么 | [01-what-is-hermes.md](01-what-is-hermes.md) |
| 安装 Hermes（Copilot 陪跑 / WSL + VS Code / Windows 桌面或原生命令行） | [02-installation.md](02-installation.md) |
| 打开并使用 Hermes（dashboard / 桌面 / CLI / TUI / Gateway） | [03-cli-tui-gateway.md](03-cli-tui-gateway.md) |
| 理解记忆和技能系统 | [04-memory-and-skills.md](04-memory-and-skills.md) |
| 对比 Hermes 与 Copilot、选择合适工具 | [05-hermes-vs-copilot.md](05-hermes-vs-copilot.md) |

## 章节导航（估算阅读时间）

| 章 | 标题 | 内容 | 估计阅读时间 |
| ---: | --- | --- | ---: |
| 1 | [Hermes 是什么](01-what-is-hermes.md) | 产品定位、核心能力、适用场景与边界 | 6 分钟 |
| 2 | [安装 Hermes](02-installation.md) | Copilot 陪跑、WSL2 + Ubuntu、VS Code 连接 WSL、Windows 桌面/原生安装、验证与排错 | 25 分钟 |
| 3 | [使用 Hermes](03-cli-tui-gateway.md) | Dashboard、桌面应用、命令行、TUI、Gateway 的正确分工 | 10 分钟 |
| 4 | [记忆与技能系统](04-memory-and-skills.md) | Memory 与 Skill 的概念、隐私边界和常用命令 | 8 分钟 |
| 5 | [Hermes 与 Copilot](05-hermes-vs-copilot.md) | 定位差异、协作方式、选型建议 | 6 分钟 |

## 阅读建议

- **只想装好就能用**：读 [第 2 章](02-installation.md)，把“Copilot 陪跑 Prompt”发给 Copilot，让它边解释边带你安装。
- **先搞清楚这是什么**：读 [第 1 章](01-what-is-hermes.md)，再决定是否安装。
- **已经装好了**：从 [第 3 章](03-cli-tui-gateway.md) 开始，了解 dashboard、桌面、命令行和 TUI 等入口。
- **不确定该用 Copilot 还是 Hermes**：读 [第 5 章](05-hermes-vs-copilot.md) 的对比表。

## 官方基线参考

- Hermes Agent 官网与下载入口: <https://hermes-agent.nousresearch.com/>
- Hermes Agent 文档: <https://hermes-agent.nousresearch.com/docs/>
- Hermes Agent GitHub: <https://github.com/NousResearch/hermes-agent>

---

← 返回主页：[AI 工具团队参考手册](../../README.md) ｜ 下一章：[Hermes 是什么](01-what-is-hermes.md)
