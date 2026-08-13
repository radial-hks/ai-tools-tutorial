# Skill 与 Prompt File / Instructions 的区别与选型

## 本章解决什么

团队里已经有了 **Instructions（指令）** 和 **Prompt File（提示词文件）**，现在又多了 **Skill（技能）**，到底什么时候用哪个？本章给一张可以直接查的决策表，帮你不再纠结。

## 先分清三样东西

| 定制类型 | 一句话定义 | 触发方式 | 能带脚本/资源吗 |
| --- | --- | --- | --- |
| **Instructions** | 项目规则和规范，像「公司规章制度」 | 自动（每次请求或按文件匹配） | ❌ 纯文字 |
| **Prompt File** | 存好的单条 `/` 命令，像「一张便利贴模板」 | 手动调用 | ❌ 纯文字 |
| **Skill** | 可复用能力包：说明 + 脚本 + 模板 + 资源，像「一本手册 + 工具箱」 | **按需自动加载**，也可手动调用 | ✅ 能 |

> 💡 记忆口诀：**Instructions 管「规矩」，Prompt File 管「一条命令」，Skill 管「一套手艺 + 工具箱」。**

## 决策表（先看这张）

官方文档给出「按目的和触发方式选定制类型」的对照表，本地化整理如下：

| 你的需求 | 用什么 | 例子 | 触发方式 |
| --- | --- | --- | --- |
| 给整个项目立统一规则 | 始终启用的 Instructions | 「必须用指定日志库和错误处理方式」 | 每次请求自动带上 |
| 给特定文件/任务立规则 | 文件匹配 Instructions | 「改 `.tsx` 时用 React 规范」 | 文件匹配或任务匹配时带上 |
| 教它一套**带资源**的工作流 | **Agent Skill** | 「用模板 + 脚本生成一个服务」 | 任务匹配时自动加载，或手动调用 |
| 存一条**按需运行**的任务 | Prompt File | 「生成一个 React 组件」 | 你手动 `/` 调用 |
| 要一个**特定角色 + 工具配置** | Custom Agent | 「用只读工具做代码审查」 | 你切换过去 / 当子代理用 |
| 接外部系统 | MCP | 「查数据库 / 更新 issue」 | 任务需要时调用工具 |
| 在特定时刻**强制执行**动作 | Hook | 「编辑后跑格式化 / 拦截危险命令」 | 事件发生时自动跑 |

### 三种「文字类」定制怎么选（重点）

排除 MCP / Hook / Custom Agent 这些「不是文字配置」的选项后，剩下 Instructions、Prompt File、Skill 三选一，口诀：

| 判断问题 | 答案 → 选择 |
| --- | --- |
| 是不是「每次都要遵守」的通用规则？ | 是 → **Instructions** |
| 是不是「一条我偶尔想手动触发」的命令？ | 是 → **Prompt File** |
| 是不是「一套步骤，且可能带脚本/模板/资源」？ | 是 → **Skill** |

再简化成一句：

> **规则 → Instructions；一条命令 → Prompt File；一套手艺（可能要带工具）→ Skill。**

## 官方对比：Skill vs Instructions

官方文档对「Skill 和自定义指令」有一张专门对比表（[第 1 章](01-what-is-skill.md) 已给出），此处补一句选型结论：

- 想让**每个对话都遵守**的编码标准、命名规范、提交规范 → **Instructions**（纯文字即可，无需脚本）。
- 想教 Copilot 一套**带脚本/示例/资源**的专项流程，还希望**跨工具复用** → **Skill**。

## 官方对比：Skill vs Prompt File

两者都能「手动 `/` 调用」，核心区别在「能不能带资源 + 会不会自动加载」：

| 对比项 | Prompt File | Skill |
| --- | --- | --- |
| 内容 | 仅提示词文字 | 指令 + 脚本 + 模板 + 资源 |
| 触发 | 只能你手动调用 | 可自动加载，也可手动调用 |
| 本质 | 一条命令的封装 | 一个能力包 |
| 官方提示 | Agent Host 上运行的 Agent 不用 Prompt File，需迁移为 Skill | 跨 VS Code / CLI / 云智能体通用 |

> 💡 官方文档特别提醒：**在 Agent Host 上运行的 Agent 不使用 Prompt File**。所以如果团队会用到 Agent Host，或希望「既能自动又能手动、还能带脚本」，选 **Skill** 更稳妥。官方还提供了 Prompt File 一键迁移为 Skill 的实验性工具。

## 三者如何配合使用

它们不是「三选一」，而是可以叠着用。以「准备一次提交流程」为例：

1. **Instructions** 写好仓库的编码规范、提交信息格式约定；
2. **Skill** 提供「准备 PR」的步骤、脚本和模板；
3. 需要临时换一种风格时，再用 **Prompt File** 覆盖某一步的默认行为。

每一层各管一件事，互不打架：

> 改规范不用动工作流，换工作流不用动角色。Skill 负责「怎么干」，Instructions 负责「什么规则」，Prompt File 负责「偶尔的快捷方式」。

## 选型速查（复制到笔记）

```text
需要带脚本/模板/资源？ ──是──> Skill
        │否
        ▼
每次都自动生效？ ──是──> Instructions
        │否
        ▼
偶尔手动触发一条命令？ ──是──> Prompt File
```

## 官方参考

- 定制类型对比（何时用哪一种）: <https://code.visualstudio.com/docs/agents/concepts/customization>
- Agent Skills 官方文档: <https://code.visualstudio.com/docs/agent-customization/agent-skills>
- Prompt Files 官方文档: <https://code.visualstudio.com/docs/agent-customization/prompt-files>
- Custom Instructions 官方文档: <https://code.visualstudio.com/docs/agent-customization/custom-instructions>

---

← 上一章：[创建自定义 Skill](03-create-skill.md) ｜ 下一章：[常用 Skill 推荐与团队实践](05-recommendations-and-practice.md)
