# 团队落地指南

## 本章解决什么

回答三个实操问题：**团队该不该用这套 Skill**、**怎么安装**、**怎么和现有工具配合**。读完你可以决定是否引入以及引入的节奏。

---

## 一、适用判断

### 这套 Skill 适合谁

| 角色 | 适合程度 | 说明 |
| --- | --- | --- |
| **程序员 / 技术美术** | 🟢 非常适合 | 日常编码、调试、重构全流程覆盖 |
| **美术 / 建模同事** | 🟡 部分适合 | 本身不写代码，但团队 Agent 行为更可控 → 间接受益 |
| **实习生 / 新同事** | 🟢 非常有价值 | 这套 Skill 强制了工程纪律，相当于一个「AI 版的导师 code review」 |
| **技术负责人** | 🟢 战略价值高 | 统一团队的 Agent 工作方式，减少「每个人都在瞎用 AI」的混乱 |

### 什么情况下不该引入

| 条件 | 判断 |
| --- | --- |
| 团队还没有稳定使用 AI 编程工具 | ❌ 先掌握基础，再引入 Skill（参考第 1–4 期教程） |
| 团队成员反对 AI 参与代码生成 | ❌ 先解决文化问题，再谈工具 |
| 项目极简单（单文件脚本） | 🟡 可以不用——这套 Skill 的收益在复杂项目上最明显 |
| 团队已经有一套成熟的 AI 工作流 | 🟡 选择性引入——可以只挑几个最需要的 Skill（如 `diagnosing-bugs`、`tdd`） |

---

## 二、安装路径对比

Matt Pocock 的 Skills 仓库提供了**三种安装方式**，对应三种不同的使用哲学：

| 安装方式 | 命令 | 适合谁 | 文件归属 | 更新方式 |
| --- | --- | --- | --- | --- |
| **Claude Code 插件** | `claude plugins install mattpocock-skills` | Claude Code 用户 | 插件托管，只读 | 作者发布时自动更新 |
| **skills.sh 拷贝** | `npx skills@latest add mattpocock/skills` | 想自定义的团队 | 你的仓库，可编辑 | 手动 `npx skills update` |
| **手动下载** | 直接 clone 或下载 ZIP | 想深度修改的团队 | 你的仓库，完全可控 | 手动管理 |

### 推荐路径：skills.sh 拷贝

对于团队使用，推荐**第二种方式（skills.sh 拷贝）**：

```bash
# 在项目根目录执行
npx skills@latest add mattpocock/skills

# 安装过程会让你选择：
# 1. 安装哪些 Skill（建议全选工程类 + grill-me）
# 2. 安装到哪个 Agent（Claude Code / Codex / 其他）
# 3. 是否创建配置文件
```

安装完成后，**在 Agent 中运行一次 `/setup-matt-pocock-skills`**：

```
/setup-matt-pocock-skills

→ Agent 会问你：
  1. 用哪个 Issue Tracker？（GitHub Issues / Linear / 本地文件）
  2. Triage 时用什么标签？
  3. 文档放在哪个目录？
```

### 最小可用集

不急着一口气装全部 22 个 Skill。建议**先装这 5 个核心 Skill**，用熟后再扩展：

| Skill | 为什么先装它 |
| --- | --- |
| `grill-with-docs` | 最常用的入口——每次写代码前盘问需求 |
| `tdd` | 保证 Agent 写的代码有测试、能跑通 |
| `diagnosing-bugs` | Bug 是常态，结构化诊断效率远高于瞎猜 |
| `code-review` | 相当于 AI 版 code review，保证质量底线 |
| `to-spec` | 让需求不再「聊完就忘」 |

---

## 三、与现有工具链的配合

Matt Pocock 的 Skill 是**跨 Agent 通用的**——它不绑定任何一个特定工具。以下是和你已知六件套的配合方式：

### 与 Claude Code 配合（推荐路径）

```text
Claude Code（终端里的 AI 编程 Agent）
  ├── 安装 Skills 后，/grill-with-docs、/tdd 等可以直接用
  ├── 优势：Claude Code 是目前对 Skill 支持最好的 Agent
  └── 劣势：需要订阅 Claude，需要终端操作
```

这是 Matt Pocock 自己使用的环境，也是 Skill 测试最充分的环境。

### 与 Copilot 配合

```text
VS Code Copilot
  ├── Copilot Chat / Agent 模式下可以加载 Skill
  ├── 安装方式：把 Skill 文件放到 .github/skills/ 目录
  ├── 优势：不离开编辑器，美术同事更友好
  └── 注意：Copilot 对 Skill 的支持还在快速迭代中，
            部分功能（如自动调用 model-invoked Skill）
            可能和 Claude Code 有差异
```

> 💡 如果你想在 Copilot 里用这些 Skill，回顾 [第 2 期教程](../../docs/02-copilot-skills/02-install-and-use.md) 的安装流程——路径一致。

### 与 Hermes 配合

```text
Hermes Agent
  ├── Hermes 支持 Skill，安装方式：npx skills@latest add
  ├── 优势：如果你已经在用 Hermes 做自动化任务，
          配上 Skill 后 Agent 的行为更可控
  └── 注意：Hermes 的工具集和 Claude Code 不同，
           部分 Skill（如 git 操作相关）需要适配
```

### 与 Pi 配合

```text
Pi（终端编程助手）
  ├── Pi 内部可以启动 Claude Code 作为子 Agent
  ├── 路径：Pi 负责任务规划 → 委托 Claude Code + Skills 执行
  └── 优势：Pi 的编排能力 + Matt Pocock 的工程纪律 = 强强联合
```

### 与 DSH 配合

```text
DeepSeek Harness
  ├── DSH 的动态插件和 Skill 是不同层面的扩展：
      - Skill = 教 Agent "怎么做"（流程、纪律、方法）
      - Plugin = 给 Agent "做的手段"（新工具、新能力）
  ├── 配合方式：在 DSH 的 Agent Preset 中配置 Skill 目录路径
  └── 注意：DSH 对 Skill 标准的支持取决于具体 Preset 配置
```

### 选型速查：我该用什么 Agent 跑这些 Skill？

| 你的情况 | 推荐 Agent | 原因 |
| --- | --- | --- |
| 我习惯在终端工作 | Claude Code | 最成熟、支持最完整 |
| 我不喜欢终端，习惯 VS Code | Copilot | 不离开编辑器 |
| 我需要 Agent 做自动化/长流程任务 | Hermes | Hermes 擅长长时间运行的任务 |
| 我想让 Pi 做总指挥 | Pi → Claude Code | Pi 编排 + Claude Code 执行 |
| 我已有 DSH 环境 | DSH | 在 Preset 中配置 Skill 路径 |

---

## 四、推广节奏建议

不建议一次性全团队铺开。分三步走：

### 第一步：个人试点（1-2 周）

1. 技术负责人或 1-2 个感兴趣的程序员先安装试用
2. 跑通场景一（修 Bug）和场景二（做新功能）
3. 记录：哪些 Skill 好用、哪些水土不服、哪些术语需要团队定制

### 第二步：小团队推广（2-4 周）

1. 把试用反馈写成团队内部的「Skill 使用指南」（1 页 A4 足矣）
2. 程序员团队全员安装
3. 定一个简单的团队约定：
   - 新功能：必须先 `/grill-with-docs` 再 `/implement`
   - Bug：必须先走 `diagnosing-bugs` 的阶段 1（建立反馈回路）
   - PR 提交前：跑一次 `code-review`

### 第三步：按需定制（持续）

1. 根据团队项目特点，修改 Skill 中的默认行为
   - 例如：团队用飞书而不是 GitHub Issues → 改 `/to-spec` 和 `/triage` 的输出目标
2. 把团队特有的规范写入 `CONTEXT.md`
3. 如果发现某个 Skill 反复需要手动调整 → 考虑 fork 一份团队版

> ⚠️ **不要一开始就大量定制。** 先用原版跑 2-4 周，知道「为什么原版这样设计」后再改。原版背后是 Matt Pocock 几十年的工程经验，很多设计决定不是偶然的。

---

## 五、常见问题

### Q: 这套 Skill 是免费的还是付费的？

**A:** 完全免费开源。Matt Pocock 的 Skills 仓库使用 MIT 协议。安装工具 `skills.sh` 也免费。

### Q: 安装后 Agent 变慢了怎么办？

**A:** Skill 是按需加载的——不是每个 Skill 每次对话都加载。只有当前任务相关的 Skill 才会被激活。如果确实感觉慢，试试：
- 精简安装的 Skill 数量（先用 5 个核心的）
- 检查 `CONTEXT.md` 是否过长（控制在 200 行以内）

### Q: 团队用的不是 Claude Code / Copilot 怎么办？

**A:** 这套 Skill 是跨 Agent 的——核心是 `SKILL.md` 文本文件，任何支持 Skill 标准的 Agent 都能加载。如果你的 Agent 不支持 Skill 标准，也可以把 `SKILL.md` 的内容作为 Instructions 或 Prompt 的一部分手动注入（效果会打折扣，但总比没有好）。

### Q: Skill 文件在项目仓库里，会不会被不小心提交到公共仓库？

**A:** `skills.sh` 安装的 Skill 文件默认放在项目 `skills/` 目录下，会进入 Git。如果你不希望公开：
- 加入 `.gitignore`
- 或者放在 Agent 的全局配置目录（如 `~/.claude/skills/`）

### Q: 和团队已有的 AGENTS.md / .cursorrules 冲突吗？

**A:** 不冲突——它们在不同层面工作：

| 文件 | 层面 | 作用 |
| --- | --- | --- |
| `AGENTS.md` / `.cursorrules` | 全局规则 | 每次对话自动加载——定「规矩」 |
| Skill（`SKILL.md`） | 专项能力 | 按任务按需加载——装「手艺」 |

如果一个 Skill 的建议和你 AGENTS.md 的规则矛盾，Agent 会优先遵循 AGENTS.md（因为它是每次都加载的硬规则）。

---

← 上一章：[AI Coding Dictionary 与配套工具](04-dictionary-and-other-tools.md) ｜ 返回主页：[AI 工具团队参考手册](../../README.md)