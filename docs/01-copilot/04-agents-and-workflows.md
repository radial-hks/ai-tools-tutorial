# Agent 与工作流

## 本章解决什么
本章定义在 VS Code 中使用 Agent（official English: Agent）时的决策维度与推荐工作流，说明如何选择运行位置、Agent 角色、模型与权限，以及如何把 Plan→Implement→Review 的流程落地。

## 一个会话的五个配置维度
五个独立维度（每项独立选择）：

| 维度 | 说明 |
|---|---|
| Interaction surface | 交互面：编辑器内联（Inline Chat）、聊天面板、Agents 窗口等，决定用户如何与 Agent 交互 |
| Runtime location | 运行位置：本地、CLI、云端或第三方提供者，影响可用资源与数据流向 |
| Agent role | Agent 角色：Agent、Plan、Ask、Custom Agent（由团队定义）等人格/职责分工 |
| Language model | 语言模型：选择模型（官方/第三方），影响能力与成本 |
| Permission level | 权限等级：只读、需确认、较高自主权；决定 Agent 是否能直接修改文件或运行命令 |

每个维度相互独立地影响行为；例如 `Plan` 是一个角色而非运行位置，`Permission level` 与 `Runtime location` 也应分开考虑。

## 选择运行位置
运行位置选项（说明）：

- 本地 agents（Local agents）：在本机执行，能够以交互方式访问工作区和本地工具；模型请求与数据处理取决于所选模型提供者与策略配置。
- Copilot CLI：可在本机作为后台或交互式工具运行，适合批量任务或本地自动化工作流。
- Cloud agents：在云端运行，通常能使用更大算力与远程服务，需依据所用提供者的隐私与数据处理政策决定是否合适。
- 第三方提供者：直接以供应商模型/Agent 运行，注意合规、成本与数据政策。

选择要点：根据任务需求、环境与数据策略选择运行位置；敏感数据、审计要求或合规性可能倾向本地或受控提供者。

## 选择 Agent 角色
常见角色（内建/推荐的明确定义）：

- `Agent`：通用的实现/执行角色，负责在其被授予的权限范围内编辑文件、运行工具与命令，把计划落地（可具备写权限）。
- `Plan`：面向阅读的规划角色，生成分步骤实现计划、变更清单与验证方案（通常为只读）。
- `Ask`：面向阅读的询问/探索角色，用于信息收集、澄清需求与快速调查（通常为只读）。
- `Custom Agent`：由用户或团队定义的人格/指令/工具/模型组合；行为、权限与工具由配置决定。参见下一章：[定制 Copilot →](05-customization.md)。

说明：审查（Review）通常是一个工作流步骤，由人工审阅者或具备相应审查能力的 Agent 执行，而不必被视为独立内建角色。保持“角色（role）”与“运行位置（runtime）”以及“权限（permission）”相互独立；选择角色并不等同于决定运行位置或自动赋予权限。

## 选择模型与权限
模型选择与权限分离：

- 模型（Language model）：选择适合任务的模型与提供者；不同模型/提供者在能力、延迟与成本，以及数据处理策略上存在差异。
- 权限（Permission level）：由当前会话或 Agent 配置提供的审批/权限选项决定，范围从需要确认的交互提示到更高的自主权；在界面中选择的具体权限名称可能因客户端版本而异。建议初学者先使用需确认的较保守设置，并在验证后逐步放宽。

## Plan → Implement → Review
工作流样例与一个完整可执行 Prompt（使用四元素方法：目标/上下文/约束/验收）：

完整 Prompt：

目标：实现仓库的 feature-flag 配置，新增 `featureX` 的开关及其后备逻辑。
上下文：#file:src/config/*.ts，#codebase 包含现有 feature-flag 用法，#changes（当前分支未提交变更）。
约束：不改变现有 API 行为，新增单元测试覆盖开关开启/关闭两种场景；在实现前先输出变更计划。
验收：本地运行测试全部通过，提交包含清晰的变更说明与单元测试。

流程步骤（建议）：

1. Plan（生成计划）：要求 Agent 列出要修改的文件、每步实现细节、预期测试。
2. Review Plan（审查计划）：人工检查并批准或要求修改。
3. Implement（执行）：在受控权限下应用变更，分小次提交并在每次提交前显示将要执行的改动。
4. Run Verification（运行验证）：运行测试、lint，生成运行结果。
5. Review Diff（审查差异）：人工审查最终 diff 并合并或创建 PR。

## 并行会话与任务边界
并行会话适合相互独立的任务（例如同时实现两个不冲突的 feature）。避免并行会话同时修改相同文件范围，若必须并行，严格约束每个会话的文件域并在合并前手工解决冲突。

## 何时不该使用 Agent
- 极小的编辑（例如单行注释或拼写修正）直接手动修改更快。
- 高风险或含敏感操作（修改生产数据库配置、删除数据）的任务，除非有可靠的审查与回滚策略。
- 任务需求高度不确定或模糊，应先人工澄清再启动 Agent。
- 涉及密钥、凭证、或受法规限制的数据处理，不要将敏感信息暴露给云端 Agent。始终避免在上下文中粘贴 secrets。

## 官方参考
- VS Code Agents 概览（English）： [Agents in Visual Studio Code](https://code.visualstudio.com/docs/agents/overview)
- Agents 规划与工作流（English）： [Agents - Planning and Workflows](https://code.visualstudio.com/docs/agents/planning)

← 上一节：[上下文与提示 →](03-context-and-prompts.md) ｜ 下一节：[定制 Copilot →](05-customization.md)
