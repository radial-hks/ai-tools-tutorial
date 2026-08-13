# Copilot Skill 团队参考手册

**受众:** 团队成员，尤其是无编程基础的美术、建模同事。

**核验基线:** 内容按 VS Code 官方文档《Use Agent Skills in VS Code》及 Agent Customization 系列页面核验于 2026-08（官方页面标注更新日期 2026-08-12）。发布前建议技术负责人按团队实际账号、插件市场和内部 Skill 清单做最后确认。

**一句话定位:** **技能（Agent Skill）** 是放在仓库里的「可复用能力包」——一个文件夹里装着说明文件（`SKILL.md`）再加上脚本、模板和示例。Copilot 在遇到相关任务时会**按需自动加载**它，也可以像命令一样手动调用。

**标记约定:** 全手册沿用三个符号标注任务分工：

- 🤖 **AI 可替代**：让 Copilot 帮你解释、生成 `SKILL.md`、排查错误、整理步骤。
- ⚠️ **必须由你决定**：账号、密钥、是否安装第三方 Skill、是否覆盖文件、安全确认、最终效果验收等，需要你亲自判断。
- 💡 **小提示**：补充说明、记忆口诀或可选做法。

> ⚠️ **内部发布提示：** Skill 的目录路径、字段名均来自官方文档；但「团队推荐哪些现成 Skill」「是否统一分发自建 Skill」属于团队决策，面向同事发布前，请把团队专属清单补充到 [第 5 章](05-recommendations-and-practice.md)。

## 按任务查找

| 任务类型 | 目标文档 |
| --- | --- |
| 搞懂 Skill 是什么、和 Instructions / Prompt File 有什么区别 | [01-what-is-skill.md](01-what-is-skill.md) |
| 找现成 Skill、装到 `.github/skills/`、调用并验证生效 | [02-install-and-use.md](02-install-and-use.md) |
| 自己写一个 Skill（`SKILL.md` 结构、命名、测试排错） | [03-create-skill.md](03-create-skill.md) |
| 拿不准该用 Skill / Prompt File / Instructions 中的哪一个 | [04-skill-vs-others.md](04-skill-vs-others.md) |
| 看适合美术/建模的示例、团队推广与安全审查建议 | [05-recommendations-and-practice.md](05-recommendations-and-practice.md) |

## 章节导航（估算阅读时间）

| 章 | 标题 | 内容 | 估计阅读时间 |
| ---: | --- | --- | ---: |
| 1 | [什么是 Copilot Skill](01-what-is-skill.md) | Skill 的定义、解决什么问题、与 Instructions / Prompt File / Custom Agent 的区别 | 6 分钟 |
| 2 | [安装与使用现有 Skill](02-install-and-use.md) | 获取来源、安装到项目或个人目录、启用/调用、验证生效 | 12 分钟 |
| 3 | [创建自定义 Skill](03-create-skill.md) | `SKILL.md` 完整结构、最小示例、命名规范、目录要求、测试排错 | 15 分钟 |
| 4 | [Skill 与 Prompt File / Instructions 的选型](04-skill-vs-others.md) | 三者对比、决策表、选型口诀、如何组合使用 | 8 分钟 |
| 5 | [常用 Skill 推荐与团队实践](05-recommendations-and-practice.md) | 美术/建模场景示例、团队推广、安全审查注意 | 8 分钟 |

## 阅读建议

- **只想装个现成 Skill 用起来**：读 [第 2 章](02-install-and-use.md)，把 Skill 目录复制进项目就能开始。
- **先搞清楚「Skill 到底是啥」**：读 [第 1 章](01-what-is-skill.md)。
- **想给自己或团队写一个 Skill**：读 [第 3 章](03-create-skill.md)，从最小示例照着改。
- **已经在用 Instructions 或 Prompt File，纠结要不要换成 Skill**：读 [第 4 章](04-skill-vs-others.md) 的决策表。
- **想直接找适合美术/建模的现成例子**：读 [第 5 章](05-recommendations-and-practice.md)。

## 官方基线参考

- Agent Skills 官方文档: <https://code.visualstudio.com/docs/agent-customization/agent-skills>
- Agent Customization 系列总览（Instructions / Prompt Files / Custom Agents 等）: <https://code.visualstudio.com/docs/agent-customization/overview>
- 定制类型对比（何时用哪一种）: <https://code.visualstudio.com/docs/agents/concepts/customization>
- Agent Skills 开放标准: <https://agentskills.io/>

---

← 返回主页：[AI 工具团队参考手册](../../README.md) ｜ 下一章：[什么是 Copilot Skill](01-what-is-skill.md)
