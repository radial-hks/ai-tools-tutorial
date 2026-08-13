# 安装与使用现有 Skill

## 本章解决什么

讲清楚三件事：现成 Skill 从哪里找、怎么装进你的项目（或只装给自己）、装好后怎么调用并确认它真的生效了。读完你可以把别人写好的 Skill 用起来。

## 从哪里获取 Skill

Skill 是「一个目录 + 一份 `SKILL.md`」，所以获取方式就是「拿到这个目录」。官方文档提到的来源有这几类：

| 来源 | 说明 | 是否需要核实 |
| --- | --- | --- |
| 社区合集仓库 `github/awesome-copilot` | 官方文档点名的社区合集，含 Skill、Custom Agent、Instructions、Prompt | ⚠️ 具体条目发布前需技术负责人核实 |
| 参考仓库 `anthropics/skills` | 官方文档点名的参考 Skill 仓库 | ⚠️ 具体条目发布前需技术负责人核实 |
| **Agent 插件（Plugin）** | 打包分发的 Skill，装插件时一起带进来 | ⚠️ 装前看插件内容和权限 |
| 团队自己的仓库 | 把 Skill 放进 `.github/skills/` 提交到 Git，同事拉代码即共享 | ✅ 团队自建，最可控 |
| 扩展（Extension）贡献的 Skill | 某些 VS Code 扩展通过 `chatSkills` 贡献点自带 Skill | 装扩展即自动带上 |

> ⚠️ 官方文档只保证「这两个仓库、插件、扩展」这类**来源渠道**存在；仓库里具体有哪些 Skill、哪个适合美术/建模，属于会随时间变化的信息，发布前请技术负责人核实并给出团队推荐清单（核验月份 2026-08）。

## Skill 装在哪里

Skill 分「项目级」和「个人级」两种，放的位置不同。官方文档给出的默认位置：

| 类型 | 默认位置 | 谁用得到 |
| --- | --- | --- |
| **项目 Skill** | `.github/skills/`（也支持 `.claude/skills/`、`.agents/skills/`） | 提交到仓库后，团队所有成员共享 |
| **个人 Skill** | `~/.copilot/skills/`（也支持 `~/.claude/skills/`、`~/.agents/skills/`） | 只有你自己，所有项目通用 |

> 💡 团队场景优先用**项目 Skill**：放进 `.github/skills/` 提交到仓库，同事拉代码就自动带上，不用每个人单独装。

## 三种安装方式

### 方式一：直接复制目录（最通用）

把整个 Skill 目录复制到项目里的 `.github/skills/` 下。以安装一个叫 `my-skill` 的 Skill 为例：

```text
你的项目/
└── .github/
    └── skills/
        └── my-skill/          ← 整个目录复制进来
            ├── SKILL.md
            └── ...（脚本、模板等）
```

复制后**重载 VS Code 窗口**（命令面板 `Reload Window`），让 Copilot 重新发现 Skill。

> ⚠️ 目录名必须和 `SKILL.md` 里 `name` 字段**完全一致**，否则 Skill 会静默不加载（详见 [第 3 章](03-create-skill.md) 的命名规范）。

### 方式二：用 Agent Customizations 编辑器（图形界面）

适合不想手敲路径的同事：

1. 打开 Chat 视图，点右上角 **Configure Chat（齿轮图标）**，进入 **Agent Customizations 编辑器**。
2. 切到 **Skills** 标签页。
3. 从下拉菜单选 **New Skill (Workspace)**（项目级）或 **New Skill (User)**（个人级），选择位置并命名。
4. 编辑器会生成 `SKILL.md`，你补上前置信息和正文即可。

> 💡 这个编辑器也用来**发现和管理**所有定制内容；官方建议优先用它统一操作。命令面板里搜 `Chat: Open Customizations` 也能打开。

### 方式三：通过插件安装

有些 Skill 被打包进 **Agent 插件（Agent Plugin）**。从编辑器的对应市场/标签页安装插件后，插件里的 Skill 会出现在 **Configure Skills** 菜单里，和你自己定义的 Skill 并列。

> ⚠️ 插件里的 hooks / MCP 会随插件启用而运行，安装前务必审查插件内容和权限（详见 [第 5 章](05-recommendations-and-practice.md)）。

## 如何启用与调用

Skill 装好后有**两种被使用的方式**，由 `SKILL.md` 的前置信息（frontmatter）字段控制：

| 方式 | 怎么触发 | 谁决定 |
| --- | --- | --- |
| **自动加载** | 你正常提需求，Copilot 判断任务相关时自己加载 | 由 `description` 写得好不好决定 |
| **手动调用（斜杠命令）** | 在聊天输入框输入 `/`，从列表里选这个 Skill | 你主动点名 |

具体行为由两个字段配合（详细见 [第 3 章](03-create-skill.md)）：

- **默认**：既出现在 `/` 菜单，也能被自动加载。
- `user-invocable: false`：不出现在 `/` 菜单，只能被自动加载（适合「背景知识型」Skill）。
- `disable-model-invocation: true`：不能被自动加载，只能手动 `/` 调用（适合「只在需要时跑」的 Skill）。

调用时还能在斜杠命令后面追加上下文，例如：

```text
/webapp-testing for the login page
/github-actions-debugging PR #42
```

## Skill 是怎么被用起来的（三级加载）

官方文档把 Skill 的使用拆成三步，理解它能帮你写对 `description`：

1. **发现（Discovery）**：Copilot 读 `SKILL.md` 前置信息里的 `name` 和 `description`，判断「这个任务和哪个 Skill 相关」。
2. **加载指令（Instructions loading）**：把 `SKILL.md` 正文加载进上下文，让 Copilot 拿到详细步骤。
3. **读取资源（Resource access）**：Copilot 用到脚本/模板时，才去读 Skill 目录里的对应文件——**没被正文引用的文件不会被加载**。

> 💡 关键结论：**`description` 决定它会不会被自动挑中；正文里用相对路径引用到的文件才会被读取。** 装一堆 Skill 也不浪费上下文，因为只有相关的才会加载。

## 如何验证生效

装完怎么确认「真的用上了」？按顺序试：

1. **看 `/` 菜单**：在聊天输入框输入 `/`，列表中应该出现这个 Skill 的名字（除非 `user-invocable: false`）。
2. **直接调用一次**：输入 `/技能名` 加一句需求，看它是否按 `SKILL.md` 的步骤执行。
3. **看诊断信息**：Chat 视图右上菜单选 **Diagnostics**，或运行 `Developer: Show Agent Debug Logs`，查看 Skill 的加载与匹配信息。
4. **反向验证自动加载**：不提技能名、只描述任务（例如「帮我跑一下网页测试」），看 Copilot 是否自动采用了对应 Skill。

> ⚠️ 若第 1、2 步就看不到 Skill，先检查三件事：目录位置对不对（项目级要在 `.github/skills/`）、目录名和 `name` 是否一致、是否已重载窗口。

## 官方参考

- Agent Skills 官方文档: <https://code.visualstudio.com/docs/agent-customization/agent-skills>
- 使用共享 Skill 章节（awesome-copilot / anthropics/skills）: <https://code.visualstudio.com/docs/agent-customization/agent-skills#_use-shared-skills>
- Agent 插件: <https://code.visualstudio.com/docs/agent-customization/agent-plugins>

---

← 上一章：[什么是 Copilot Skill](01-what-is-skill.md) ｜ 下一章：[创建自定义 Skill](03-create-skill.md)
