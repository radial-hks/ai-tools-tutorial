# 常用 Skill 推荐与团队实践

## 本章解决什么

给美术、建模同事一些「Skill 能干嘛」的实际场景启发，以及团队推广时怎么落地、安装第三方 Skill 前必须注意什么。读完你能判断哪些场景值得做 Skill、哪些要谨慎。

## 适合美术/建模同事的场景示例

> ⚠️ **以下均为「示例/占位」场景**，用于说明 Skill 适合干什么，并非「这些 Skill 已在某个仓库现成可用」。具体某个 Skill 是否已存在、是否需要自行开发，发布前需技术负责人核实并给出团队推荐清单（核验月份 2026-08）。

| 场景 | 你可以做一个 Skill 让 Copilot 自动做 | 附带资源举例 |
| --- | --- | --- |
| 资源命名检查 | 按「角色名_部位_后缀」规范检查并整理文件命名 | 命名规则表（模板） |
| 贴图清单生成 | 扫描目录，生成每个模型用到的贴图清单和缺失报告 | 清单模板（CSV/Markdown） |
| 导出流程把关 | 按团队规范检查 FBX/glTF 导出设置、单位、命名 | 导出检查清单 |
| 提交信息规范 | 按团队格式自动写变更说明 | 提交信息模板 |
| 重复性报告 | 统计素材数量、大小、命名合规率，生成周报 | 报告模板 |

> 💡 判断一个场景值不值得做 Skill 的三条标准：**① 重复出现；② 步骤固定可写成清单；③ 最好需要固定模板或脚本。** 三条都满足，就值得做。

## 团队推广建议

1. **从 1 个低风险场景起步**：先选一个「只读、不改文件」的场景（如生成命名清单），让大家看到价值再扩展。
2. **放进仓库共享**：团队 Skill 统一放 `.github/skills/`，提交到 Git，新同事拉代码即用，不用单独安装。
3. **给 `description` 下功夫**：自动加载全靠 `description` 写得准。写清「干什么 + 何时用」，同事不用记命令名也能被自动触发。
4. **用 Agent Customizations 编辑器统一管理**：让大家在同一个入口创建、查看、管理 Skill。
5. **先试后推广**：在复制出来的测试文件夹里验证，确认结果正确再进正式项目。
6. **逐步沉淀**：把同事反复让 Copilot 做的事，用「create a skill from how we just debugged that」的方式沉淀成 Skill，而不是每次口述。

## 安全与审查注意（最重要）

> ⚠️ **Skill 里的脚本会被 Copilot 真正执行，模板和资源会被读取。装第三方 Skill 前，必须先把源码看一遍。**

官方文档在「使用共享 Skill」一节明确提醒：**使用别人写的 Skill 前，务必审查它是否符合你的需求和**安全标准。具体注意：

| 注意点 | 为什么 |
| --- | --- |
| 安装前通读 `SKILL.md` 和附带的每个脚本 | 脚本可能包含你不想执行的命令（删文件、改配置、联网上传等） |
| 看脚本里有没有 `rm`、覆盖写入、网络请求、密钥读取 | 这些是高风险动作 |
| 优先用「只读」Skill 练手 | 生成清单/报告类不碰原文件，风险最低 |
| 关注 VS Code 终端工具的执行控制 | 终端工具对脚本执行有控制（含 auto-approve 选项、可配置白名单），了解它能在执行前拦一道 |
| 团队分发的 Skill 走 Code Review | 自建 Skill 提交前照常走评审，像审代码一样审 `SKILL.md` |

> 💡 一句话安全口诀：**「只读」的随便试，「会改/会跑」的先看源码再放行；拿不准就问技术负责人。**

## 学习路径（给想继续深入的人）

1. 读 [第 2 章](02-install-and-use.md) 装一个官方示例，体验自动加载与手动调用。
2. 读 [第 3 章](03-create-skill.md) 抄最小示例改出自己的第一个 Skill。
3. 用 `/create-skill` 让 AI 帮你生成，再人工审一遍。
4. 遇到「一套流程 + 固定模板」的重复任务，就沉淀成 Skill。
5. 团队里建一份「推荐 Skill 清单」，注明每个 Skill 的来源、是否审过源码。

## 官方参考

- Agent Skills 官方文档（含使用共享 Skill 的安全提示）: <https://code.visualstudio.com/docs/agent-customization/agent-skills>
- 终端工具与脚本执行控制: <https://code.visualstudio.com/docs/agents/run/tools#_run-terminal-commands>
- 安全审查与 auto-approve: <https://code.visualstudio.com/docs/agents/run/security#_approvals-and-review>
- Agent 插件（分发 Skill）: <https://code.visualstudio.com/docs/agent-customization/agent-plugins>

---

← 上一章：[Skill 与 Prompt File / Instructions 的选型](04-skill-vs-others.md) ｜ 返回主页：[AI 工具团队参考手册](../../README.md)
