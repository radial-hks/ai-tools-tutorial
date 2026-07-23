# VS Code Copilot 团队参考手册重构实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 将 `docs/01-copilot/` 重构为准确、可检索、可维护的 VS Code Copilot 团队参考手册，并保留独立的讲师附录。

**Architecture:** 按“概览与安装、交互界面、上下文、Agent 工作流、定制、MCP 与工具、安全与排错”划分七个互不重叠的主题章。核心定义只保留一处，首页按任务提供入口，讲师附录通过链接复用核心概念；根目录和共享参考同步更新。

**Tech Stack:** Markdown、Git 文件历史、VS Code Markdown diagnostics、Ruby 标准库检查链接、Python 标准库校验 JSON 示例

---

## 文件结构

| 路径 | 操作 | 单一职责 |
|------|------|----------|
| `docs/01-copilot/README.md` | 重写 | 手册定位、版本基线、任务入口和章节导航 |
| `docs/01-copilot/01-overview-and-setup.md` | 重命名并重写 | 产品边界、账号、安装与验证 |
| `docs/01-copilot/02-chat-and-inline.md` | 重命名并重写 | 四种交互界面和基础操作 |
| `docs/01-copilot/03-context-and-prompts.md` | 重写 | 上下文、提示和会话管理 |
| `docs/01-copilot/04-agents-and-workflows.md` | 重命名并重写 | 会话配置维度和 Agent 工作流 |
| `docs/01-copilot/05-customization.md` | 重命名并重写 | 六类定制方式的选择与最小配置 |
| `docs/01-copilot/06-mcp-and-tools.md` | 重命名并重写 | 内置工具、MCP 能力、配置、安全和排错 |
| `docs/01-copilot/07-safety-and-troubleshooting.md` | 重命名并重写 | 审查、权限、隐私、回滚和诊断 |
| `docs/01-copilot/appendix-demo-guide.md` | 重命名并精简 | 讲师演示流程与准备材料 |
| `README.md` | 修改 | 仓库级入口和本期目录 |
| `docs/_shared/glossary.md` | 修改 | 跨文档统一术语 |
| `docs/_shared/cheat-sheet.md` | 修改 | 跨工具 VS Code 基础快捷键 |

## Task 1: 固定迁移映射与手册入口

**Files:**
- Modify: `docs/01-copilot/README.md`
- Rename: `docs/01-copilot/01-what-is-copilot.md` → `docs/01-copilot/01-overview-and-setup.md`
- Rename: `docs/01-copilot/02-basic-usage.md` → `docs/01-copilot/02-chat-and-inline.md`
- Rename: `docs/01-copilot/04-agent-mode.md` → `docs/01-copilot/04-agents-and-workflows.md`
- Rename: `docs/01-copilot/05-advanced-customization.md` → `docs/01-copilot/05-customization.md`
- Rename: `docs/01-copilot/06-mcp-and-external-tools.md` → `docs/01-copilot/06-mcp-and-tools.md`
- Rename: `docs/01-copilot/07-best-practices.md` → `docs/01-copilot/07-safety-and-troubleshooting.md`
- Rename: `docs/01-copilot/08-demo-guide.md` → `docs/01-copilot/appendix-demo-guide.md`

- [ ] **Step 1: 使用 Git 重命名文件**

```bash
git mv docs/01-copilot/01-what-is-copilot.md docs/01-copilot/01-overview-and-setup.md
git mv docs/01-copilot/02-basic-usage.md docs/01-copilot/02-chat-and-inline.md
git mv docs/01-copilot/04-agent-mode.md docs/01-copilot/04-agents-and-workflows.md
git mv docs/01-copilot/05-advanced-customization.md docs/01-copilot/05-customization.md
git mv docs/01-copilot/06-mcp-and-external-tools.md docs/01-copilot/06-mcp-and-tools.md
git mv docs/01-copilot/07-best-practices.md docs/01-copilot/07-safety-and-troubleshooting.md
git mv docs/01-copilot/08-demo-guide.md docs/01-copilot/appendix-demo-guide.md
```

Expected: 七个文件以 rename 状态出现，`03-context-and-prompts.md` 保持原路径。

- [ ] **Step 2: 重写本期首页**

首页必须包含以下入口：

```markdown
# VS Code Copilot 团队参考手册

> 面向需要在 VS Code 中使用 AI 完成开发、脚本和项目协作任务的团队成员。
>
> 内容核验于 2026-07。Preview 或 Experimental 功能会在正文中单独标注。

## 按任务查找

| 我想要…… | 从这里开始 |
|-----------|------------|
| 安装并确认 Copilot 可用 | [概览与安装](01-overview-and-setup.md) |
| 提问、解释代码或就地修改 | [聊天与内联交互](02-chat-and-inline.md) |
| 给 AI 提供正确上下文 | [上下文与提示](03-context-and-prompts.md) |
| 规划并执行多文件任务 | [Agent 与工作流](04-agents-and-workflows.md) |
| 让 AI 遵守项目规范 | [定制 Copilot](05-customization.md) |
| 连接浏览器、数据库或外部服务 | [MCP 与工具](06-mcp-and-tools.md) |
| 审查改动或排查异常 | [安全与排错](07-safety-and-troubleshooting.md) |
```

再添加七章职责与阅读时间表、独立的讲师附录入口、按任务跳读建议，以及 Agents、Chat、Customization 三个官方基线链接。

- [ ] **Step 3: 验证首页链接**

```bash
ruby -e 'File.read("docs/01-copilot/README.md").scan(/\]\(([^)#]+\.md)(?:#[^)]+)?\)/).flatten.each { |p| abort("missing: #{p}") unless File.file?(File.expand_path(p,"docs/01-copilot")) }; puts "README links OK"'
```

Expected: `README links OK`。

## Task 2: 重写概览与交互界面

**Files:**
- Modify: `docs/01-copilot/01-overview-and-setup.md`
- Modify: `docs/01-copilot/02-chat-and-inline.md`

- [ ] **Step 1: 重写概览与安装章**

使用标题：`本章解决什么`、`Copilot 能做什么`、`Copilot 不能替你做什么`、`账号、订阅与组织策略`、`安装与启用`、`验证是否可用`、`下一步`、`官方参考`。

必须说明：AI 功能内置于 VS Code；GitHub 账号可启用免费或付费计划；组织策略可能禁用 Agent；BYOK 是语言模型接入方式，不等同于获得 GitHub Copilot 服务。删除“五个层次”表和“不绑定 GitHub 即可使用 Copilot”的歧义表述。验证步骤覆盖内联建议、Chat View 和 Agent。

- [ ] **Step 2: 重写聊天与内联交互章**

使用标题：`本章解决什么`、`四种交互界面怎么选`、`Chat View`、`Agents Window（Preview）`、`Inline Chat`、`Quick Chat`、`内联建议`、`发送、排队与纠偏`、`常用快捷键`、`官方参考`。

选择矩阵把 Agents Window、Chat View、Inline Chat、Quick Chat 分开；内联建议作为编辑能力单列。保留 Queue、Steer、Stop and Send 的准确行为。每种界面给出适用场景和至少一个入口，优先写 Command Palette 命令，再写快捷键。

- [ ] **Step 3: 添加导航并检查冲突术语**

概览章链接首页和第 2 章；交互章链接第 1、3 章。Run:

```bash
rg -n '五个层次|三种核心交互模式|不绑定 GitHub|永远生效' docs/01-copilot/01-overview-and-setup.md docs/01-copilot/02-chat-and-inline.md
```

Expected: 无输出。

## Task 3: 重写上下文与 Agent 工作流

**Files:**
- Modify: `docs/01-copilot/03-context-and-prompts.md`
- Modify: `docs/01-copilot/04-agents-and-workflows.md`

- [ ] **Step 1: 重写上下文与提示章**

使用标题：`本章解决什么`、`上下文从哪里来`、`使用 # 引用显式上下文`、`图片与浏览器上下文`、`写出可执行的请求`、`管理长会话`、`常用任务模板`、`风险与限制`、`官方参考`。

区分活动文件/选区等隐式上下文与 Agent 自主搜索。引用表保留 `#file`、文件夹/符号、`#codebase`、`#selection`、`#terminalSelection`、`#changes`、`#problems`、`#fetch`，并提醒可用项以输入框补全为准。Prompt 方法收敛为“目标、上下文、约束、验收”四要素，保留开发和美术/建模各两个示例。

- [ ] **Step 2: 重写 Agent 与工作流章**

使用标题：`本章解决什么`、`一个会话的五个配置维度`、`选择运行位置`、`选择 Agent 角色`、`选择模型与权限`、`Plan → Implement → Review`、`并行会话与任务边界`、`何时不该使用 Agent`、`官方参考`。

五维表列出交互界面、运行位置、Agent 角色、语言模型、权限级别。运行位置列出 Local、Copilot CLI、Cloud、第三方；角色列出 Agent、Plan、Ask、Custom Agent。明确角色、运行位置、权限互相独立。工作流给出一个带验收标准的完整 Prompt，并要求审查计划、实施、验证和 diff。

- [ ] **Step 3: 添加导航并验证五维概念**

上下文章链接第 2、4 章；Agent 章链接第 3、5 章。Run:

```bash
for term in '交互界面' '运行位置' 'Agent 角色' '语言模型' '权限级别'; do rg -q "$term" docs/01-copilot/04-agents-and-workflows.md || exit 1; done
! rg -n '三种内置 Agent 类型|Agent 类型（进阶）' docs/01-copilot/04-agents-and-workflows.md
```

Expected: exit code 0，无错误输出。

## Task 4: 重写定制章

**Files:**
- Modify: `docs/01-copilot/05-customization.md`

- [ ] **Step 1: 建立选择矩阵**

使用这六种需求映射：每次自动应用项目规范 → Instructions；手动调用可复用请求 → Prompt File；自动加载流程/脚本/资源 → Agent Skill；切换角色和工具集 → Custom Agent；生命周期执行确定性命令 → Hook；安装分发现成定制 → Agent Plugin（Preview）。删除原稿暗示包含关系的层次树。

- [ ] **Step 2: 按统一模板重写六类定制**

每类使用“适用场景、默认位置、最小示例、限制”。覆盖以下路径：

```text
.github/copilot-instructions.md
.github/instructions/*.instructions.md
.github/prompts/*.prompt.md
.github/skills/<name>/SKILL.md
.github/agents/*.agent.md
.github/hooks/*.json
```

说明 Skill 在任务匹配时由 Agent 自动加载；Custom Agent 的 `tools`、`model`、`handoffs` 为可选配置；Plugin 标注 Preview。Hooks 事件和退出码必须重新对照官方文档。

- [ ] **Step 3: 添加渐进采用与排错**

顺序为：`/init` → Prompt File 或 Skill → Custom Agent → 按需 MCP/Hooks/Plugin。加入 `Chat: Open Customizations`（Preview）、customization diagnostics 和 Agent Debug Logs。

- [ ] **Step 4: 验证结构**

```bash
ruby -e 's=File.read("docs/01-copilot/05-customization.md"); abort("unbalanced fences") unless s.scan(/^```/).length.even?; abort("old hierarchy remains") if s.include?("Instructions（编码规范，永远生效）"); puts "customization structure OK"'
```

Expected: `customization structure OK`。

## Task 5: 重写 MCP 与工具章

**Files:**
- Modify: `docs/01-copilot/06-mcp-and-tools.md`

- [ ] **Step 1: 重建章节结构**

使用标题：`本章解决什么`、`工具和 MCP 的关系`、`MCP Server 能提供什么`、`安装 MCP Server`、`配置 mcp.json`、`管理工具与服务器`、`信任、密钥与沙箱`、`排错`、`官方参考`。

明确 MCP Server 可提供 Tools、Resources、Prompts、MCP Apps，并区分内置工具、扩展工具和 MCP 工具。

- [ ] **Step 2: 更新安装和配置**

保留 Marketplace `@mcp`、`MCP: Add Server`、工作区 `.vscode/mcp.json`、用户配置入口。JSON 示例包含一个 HTTP server 和一个 stdio server；示例后禁止硬编码 API key，改用 input variables 或环境文件。

- [ ] **Step 3: 补充信任和沙箱**

说明本地 Server 可运行任意代码、首次启动信任、`MCP: Reset Trust` 和最小工具集。macOS/Linux 示例使用 `sandboxEnabled: true`；Windows 当前不支持该能力，标注核验于 2026-07。

- [ ] **Step 4: 校验主 JSON 示例**

将主示例复制到 `/tmp/copilot-mcp-example.json`，Run:

```bash
python3 -m json.tool /tmp/copilot-mcp-example.json >/dev/null && rm /tmp/copilot-mcp-example.json
```

Expected: exit code 0。

## Task 6: 重写安全与排错章

**Files:**
- Modify: `docs/01-copilot/07-safety-and-troubleshooting.md`

- [ ] **Step 1: 按风险控制重写**

使用标题：`本章解决什么`、`开始任务前`、`Agent 工作期间`、`接受改动前`、`权限与工具最小化`、`密钥、隐私与组织策略`、`回滚与恢复`、`常见问题`、`诊断入口`、`官方参考`。

审查清单覆盖需求、边界输入、安全、依赖、测试、生成文件和许可证风险。明确 Stage 接受待处理编辑、Discard 丢弃待处理编辑、Checkpoints 不能替代 Git 提交。

- [ ] **Step 2: 建立按症状排错表**

覆盖：内联建议不出现、Agent 不可用、Instructions/Skills 未加载、MCP Server 不启动、响应偏离、上下文过长、工具确认过多。每项提供“先检查什么”和“诊断入口”，不承诺单一原因。

- [ ] **Step 3: 删除易过期内容并验证**

模型建议仅保留按任务复杂度选择，并以当前选择器和组织策略为准。删除具体型号排名、固定额度承诺、四阶段学习路径和 ASCII 速查。结尾链接第 6 章和首页。Run:

```bash
rg -n 'GPT-4\.1|Claude Sonnet|第 1 天|第 2-4 周|一页纸速查' docs/01-copilot/07-safety-and-troubleshooting.md
```

Expected: 无输出。

## Task 7: 迁移讲师演示附录

**Files:**
- Modify: `docs/01-copilot/appendix-demo-guide.md`

- [ ] **Step 1: 改为纯演示手册**

保留演示概览、环境准备、步骤、备用方案、节奏和 Prompt 速查。删除重复概念定义，改为指向核心章节的“讲解前必读”。

- [ ] **Step 2: 收敛 60–90 分钟流程**

阶段为：开场与边界、内联建议、Chat/Inline Chat、上下文、Plan→Implement→Review、Instructions、MCP 概念展示、Q&A。每阶段写目标、动作、预期结果、失败备用方案。MCP 只展示已安装并信任的 Playwright server。

- [ ] **Step 3: 更新链接和 Prompt**

所有链接使用新文件名。Prompt 包含目标和验收条件；Agent 演示使用小型可回滚任务，不现场生成完整注册系统。

- [ ] **Step 4: 检查旧链接和重复定义**

```bash
! rg -n '01-what-is-copilot|02-basic-usage|04-agent-mode|05-advanced-customization|06-mcp-and-external-tools|07-best-practices|什么是 AI 编程助手.*比喻' docs/01-copilot/appendix-demo-guide.md
```

Expected: exit code 0，无输出。

## Task 8: 同步仓库入口与共享参考

**Files:**
- Modify: `README.md`
- Modify: `docs/_shared/glossary.md`
- Modify: `docs/_shared/cheat-sheet.md`

- [ ] **Step 1: 更新根目录入口**

目录树和“本期内容”表使用新文件名、新标题；状态使用“维护中”；讲师附录不计入核心章节编号。

- [ ] **Step 2: 统一术语表**

增加并区分 Agents Window、Chat View、Agent type/运行位置、Agent role/角色、Permission level、MCP Resources、MCP Apps。把 Ask/Agent/Plan 定义为内置角色；Plugins 改为 `Agent Plugins（Preview）`；删除“Instructions 永远生效”和“BYOK 不需要订阅”的无边界简写。

- [ ] **Step 3: 校对共享快捷键**

共享速查只保留 VS Code 基础快捷键。Copilot 专用快捷键仅放第 2 章。

- [ ] **Step 4: 检查用户文档旧路径**

```bash
rg -n '01-what-is-copilot|02-basic-usage|04-agent-mode|05-advanced-customization|06-mcp-and-external-tools|07-best-practices|08-demo-guide' README.md docs/01-copilot docs/_shared
```

Expected: 无输出。

## Task 9: 全仓质量验证

**Files:**
- Verify: all `*.md`

- [ ] **Step 1: 检查本地 Markdown 链接**

```bash
ruby -e '
errors=[]
Dir.glob("**/*.md").each do |file|
  File.read(file).scan(/\[[^\]]*\]\(([^)]+)\)/).flatten.each do |target|
    next if target.start_with?("http://", "https://", "mailto:", "#")
    path=target.split("#",2).first
    next if path.empty?
    errors << "#{file}: #{target}" unless File.exist?(File.expand_path(path, File.dirname(file)))
  end
end
abort(errors.join("\n")) unless errors.empty?
puts "Local Markdown links OK"
'
```

Expected: `Local Markdown links OK`。

- [ ] **Step 2: 检查围栏和 H1**

```bash
ruby -e '
Dir.glob("docs/01-copilot/*.md").each do |f|
  s=File.read(f)
  abort("unbalanced fences: #{f}") unless s.scan(/^```/).length.even?
  abort("expected one H1: #{f}") unless s.lines.count { |l| l.start_with?("# ") } == 1
end
puts "Markdown structure OK"
'
```

Expected: `Markdown structure OK`。

- [ ] **Step 3: 检查术语和旧路径**

```bash
! rg -n '三种核心交互模式|五个层次|永远生效|三种内置 Agent 类型|08-demo-guide|04-agent-mode' README.md docs/01-copilot docs/_shared
```

Expected: exit code 0，无输出。

- [ ] **Step 4: 检查官方链接**

```bash
rg -o 'https://(code\.visualstudio\.com|docs\.github\.com)[^ )]+' docs/01-copilot/*.md | cut -d: -f2- | sort -u | while IFS= read -r url; do code=$(curl -L -s -o /dev/null -w '%{http_code}' "$url"); case "$code" in 2*|3*) ;; *) echo "$code $url"; exit 1;; esac; done
```

Expected: 无输出，exit code 0。

- [ ] **Step 5: 检查 diagnostics 和差异**

用 VS Code diagnostics 检查根 README、`docs/_shared/`、`docs/01-copilot/`，期望无新增错误。Run:

```bash
git --no-pager diff --check
git --no-pager diff --stat
git status --short
```

Expected: `diff --check` 无输出；只包含计划内文档；无临时文件。

- [ ] **Step 6: 执行三项检索抽查**

从本期首页开始，人工确认两次点击内可回答：如何区分运行位置/角色/权限；如何配置项目规范；如何安全安装和限制 MCP Server。答案分别位于第 4、5、6 章。

## 实施约束

- 每次手工编辑使用小范围 patch，不改写其他期次
- 首次实质编辑后立即运行对应任务的验证命令
- 官方事实以实施当天页面为准；冲突时以官方文档为准并标注核验日期
- 不创建 Git commit，除非用户明确要求
- 不引入构建依赖；优先使用 Ruby、Python、curl 和 ripgrep