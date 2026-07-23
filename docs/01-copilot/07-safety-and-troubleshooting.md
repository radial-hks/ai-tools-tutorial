# 安全与排错

## 本章解决什么

本章为在 IDE/Agent/MCP 工作流中安全地使用 AI 提供简洁可执行的指南：定义目标与验收、任务前的准备、运行期间的监控与控制、接受改动前的核查、最小权限原则、密钥与隐私策略、回滚与恢复步骤、常见故障排查表与诊断入口，以及可信的官方参考链路。

## 开始任务前

- 目标/范围/验收：在开始前写明目标、输入边界、不可接受的变更与验收条件（可运行测试或对比用例）。
- 基线与状态检查：在本地运行：

```bash
git status --porcelain
git rev-parse --abbrev-ref HEAD
git rev-parse HEAD
```

- 选择角色/运行时/模型/权限：指定执行者（你/Agent）、运行环境、模型类别与最小权限（仅读或需确认的写权限）。
- 高风险备份：对高影响分支或生成资产，创建明确的检查点（commit 或临时分支）：

```bash
git add -A && git commit -m "checkpoint: pre-<task>-backup"
# 或创建分支保留当前状态
git branch checkpoint/pre-<task>
```

（检查点不等同于最终提交；在重要变更前请使用 Git commit。）

## Agent 工作期间

- 检查工具调用与命令：在 Agent 运行时审阅每个预期的外部调用或预工具钩子；拒绝含糊或超范围的命令。
- 队列、引导与停止：对长期或并行任务使用明确队列与批准点；需要时发送 Stop/Abort 并等待 Agent 停止。
- 停止范围漂移：若输出或变更超出既定范围，立即中断并回到“开始任务前”。
- 不盲目批准：对任何会修改仓库或外部系统的操作，先在差异（diff）视图审查并只在满足验收后批准。
- 隔离并行改动：并行实验应使用独立分支或工作树，避免在同一分支并发合并未审查的更改。

## 接受改动前

在接受或合并任何 AI 产生的改动前逐项核查（可用复选项以便手动打勾）：

- [ ] 需求：改动满足明确的功能/非功能需求。
- [ ] 边界输入：处理异常、空值、边界条件的测试或说明已覆盖。
- [ ] 安全：无硬编码密钥/凭证、无注入风险、权限最小化。
- [ ] 依赖：新增依赖已审查来源与许可，且版本固定。
- [ ] 测试：单元/集成/关键路径测试已通过或有快速可执行验证步骤。
- [ ] 生成文件：列出所有自动生成文件及其预期位置。
- [ ] 许可/IP 风险：没有引入不兼容许可证或可疑外部代码。
- [ ] 回退计划：有明确回滚步骤（见“回滚与恢复”）。

## 权限与工具最小化

- 最小权限原则：Agent 和工具默认只启用只读或审查模式，写/执行权限仅在明确批准后临时放开。 
- 计划阶段使用只读权限与模拟（dry-run）；当需要写时，弹出明确批准请求。
- 对外网络、文件系统、shell 的访问使用沙箱/白名单，限制域名与路径。

示例沙箱策略要点：只允许写入临时工作目录，禁止读取敏感配置目录，网络仅限指定域名。

## 密钥、隐私与组织策略

- 切勿在聊天或补全中粘贴密钥或凭证。使用环境变量、秘密管理器或平台提供的 secret 注入机制。
- 审阅提供者与组织策略：核对云/模型提供方的数据处理条款、公司数据分类政策与合规需求。
- 不做无法证明的隐私承诺：说明数据如何被使用与保留，并让合规/安全团队批准高敏感度数据处理。

## 回滚与恢复

- 保留明确的 Keep/Undo 语义：
  - Stage/Accept（暂存或接受）意味着你已审查并打算将改动纳入历史。 
  - Discard（放弃）会移除未保存的编辑或临时补丁。
- 检查点不是提交：显式检查点（IDE 的检查点/快照）便于临时恢复，但不能替代 Git 提交或分支保护。请在关键步骤前用 `git commit` 或新分支保存状态。
- 验证状态再做破坏性回滚：在执行 `git reset --hard`、`git checkout --force` 或远程回滚前，先运行 `git status`, `git log --oneline -n 5`，并确认远程/同事同步策略。

示例回滚检查：

```bash
git status
git log --oneline -n 5
# 若确定回滚：
# git reset --hard <commit>
# 或恢复单文件： git checkout -- <file>
```

## 常见问题

| 症状 | 首要检查 | 诊断入口 |
|---|---|---|
| 内联补全建议缺失 | 是否已登录、扩展启用、文件语言识别 | 打开 Chat/状态栏，查看扩展页和 Output 日志 |
| Agent 不可用或未响应 | 网络/凭证/Agent 服务进程 | Developer: 打开 Agent Debug 面板；查看 Agent 日志 |
| 指令/Skill 未加载 | 指令文件语法或 applyTo 不匹配 | 打开 Chat 自定义/指令面板；检查 `.instructions.md` 路径 |
| MCP Server 启动失败 | 端口被占用、依赖未安装 | Output → MCP: 列表服务器；查看服务器日志（MCP UI 或控制台） |
| 回答逐渐偏离目标（drift） | Prompt 未限制上下文或缺乏约束 | 在会话中使用 Steer/Stop；检查 prompt 历史与 Plan |
| 上下文过长导致截断 | 选择过多文件或未压缩历史 | 使用 `#file` 精准引用，或 `/compact` 压缩会话 |
| 需要过多确认或循环确认 | 权限策略过保守或提示不明确 | 检查授权级别与审批策略；优化 prompt 明确期望 |

（诊断无单一原因，按上述首要检查逐步排查。）

## 诊断入口

- IDE 菜单与面板（名称随版本略有差异，按关键字查找）：
  - Developer: Open Agent Debug Panel（开发者工具 → Agent Debug）
  - Chat 面板菜单：Show Agent Debug Logs / Show Chat Logs
  - Output 面板：选择 MCP/Agent/Extensions 下拉项查看运行日志
  - View → Problems：查看语法/类型错误影响 AI 补全
  - Extensions → 扩展日志或输出（扩展页内的“运行日志”按钮）
  - MCP 管理：MCP List Servers / Show Server Logs（如果已安装 MCP）

## 官方参考

- GitHub Copilot 文档：[docs/01-copilot/06-mcp-and-external-tools.md](docs/01-copilot/06-mcp-and-external-tools.md)
- GitHub Copilot 概览（官方）：https://docs.github.com/en/copilot
- GitHub 代码安全指南（官方）：https://docs.github.com/en/code-security
- GitHub Security Lab（官方研究与教程）：https://securitylab.github.com/

---

← 上一节：[MCP 外部工具](06-mcp-and-external-tools.md) ｜ [返回本期首页](README.md)
