# 创建自定义 Skill

## 本章解决什么

手把手教你自己写一个 Skill：`SKILL.md` 的完整结构、每个字段怎么填、命名和目录有什么硬性要求、写完怎么测试和排错。读完你能从零做出一个能用的 Skill。

> 💡 写 Skill 不要求会编程——正文就是 Markdown 文字。只有当你的 Skill 需要「自动化执行某件事」时才需要附带脚本，纯流程类 Skill 完全可以只有一份 `SKILL.md`。

## 最小可运行示例

先看一个完整的最小例子（改编自官方文档「网页测试」示例，改成团队里更通用的「整理贴图文件」场景）：

```markdown
---
name: texture-rename
description: 把散乱的贴图文件按「角色名_部位_后缀」的规则批量重命名。当用户提到整理贴图、重命名纹理、按规范命名贴图时使用。
---

# 贴图重命名

## 何时使用
当用户需要按团队命名规范整理贴图文件时使用本 Skill。

## 命名规则
- 格式：`角色名_部位_后缀`，例如 `knight_body_diffuse.png`
- 角色名用英文小写，多个单词用短横线连接
- 部位只允许：body / face / hand / weapon

## 步骤
1. 先列出目标目录里所有贴图文件，不做任何修改
2. 对照上面的命名规则，给出重命名方案清单
3. 等你确认后再执行重命名

## 验收
- 所有文件名符合「角色名_部位_后缀」格式
- 未确认前不覆盖任何原文件
```

把这个内容存为 `.github/skills/texture-rename/SKILL.md` 就完成了一个最小 Skill。

## `SKILL.md` 完整结构

一个 `SKILL.md` 由两部分组成：

```text
┌─ YAML frontmatter（前置信息）──┐
│  name / description / 可选字段  │
└────────────────────────────────┘
┌─ 正文（Markdown 指令）─────────┐
│  干什么、何时用、步骤、示例      │
└────────────────────────────────┘
```

### 前置信息（frontmatter）字段表

以下字段均核验自官方文档：

| 字段 | 是否必填 | 说明 |
| --- | --- | --- |
| `name` | ✅ 必填 | Skill 唯一标识。只能小写字母、数字、短横线，**必须与父目录名一致**，最长 64 字符。 |
| `description` | ✅ 必填 | 说明「干什么 **以及何时用**」。要同时写清能力和使用场景，帮助 Copilot 判断何时加载，最长 1024 字符。 |
| `argument-hint` | 可选 | 作为斜杠命令调用时，输入框里显示的提示文字（例如 `[test file] [options]`）。 |
| `user-invocable` | 可选 | 是否出现在 `/` 菜单里，默认 `true`；设 `false` 则隐藏菜单、只允许自动加载。 |
| `disable-model-invocation` | 可选 | 是否禁止 Copilot 自动加载，默认 `false`；设 `true` 则只能手动 `/` 调用。 |
| `context` | 可选（实验性） | 加载方式，默认 `inline`（指令加入父会话上下文）；设 `fork` 则在独立子代理里运行、只回传最终结果。 |

> 💡 记住两个必填字段的分工：**`name` 管「它叫什么」，`description` 管「什么时候该用它」。** `description` 写得好不好，直接决定 Copilot 能不能在需要时自动挑中它。

### 正文（body）怎么写

正文就是给 Copilot 看的操作手册，建议写清这些内容（官方文档的原话要点）：

- 这个 Skill 帮你达成什么；
- 什么时候该用；
- 一步一步的流程；
- 期望的输入和输出示例；
- 对 Skill 目录内脚本/资源的引用。

## 命名与目录的硬性要求

这些是「写错就静默失败」的坑，务必照做：

1. **只用小写字母、数字、短横线**（kebab-case），例如 `texture-rename`。
2. **不要用**斜杠、冒号、点、命名空间前缀（如 `myorg/skillname`、`myorg:skillname`）——用这些字符会导致 Skill **静默不加载**。
3. **目录名 = `name` 字段**，必须完全一致；不一致也不加载。
4. **最长 64 字符**。

```text
✅ .github/skills/texture-rename/SKILL.md   （name: texture-rename）
❌ .github/skills/TextureRename/SKILL.md    （大写，非法）
❌ .github/skills/myorg:rename/SKILL.md     （冒号/前缀，非法）
❌ .github/skills/rename/SKILL.md           （目录名与 name 不一致）
```

> 💡 如果 Skill 是通过**插件**分发的，插件名会自动作为命令前缀（例如 `/my-plugin:test-runner`），你**不要**自己往 `name` 里加前缀。

## 目录要求与附带资源

Skill 目录里 `SKILL.md` 是唯一必填文件，其余按需添加：

```text
.github/skills/my-skill/
├── SKILL.md           ← 必填
├── script.py          ← 可选：脚本
├── template.md        ← 可选：模板
└── examples/          ← 可选：示例
```

**要让 Copilot 用到附带文件，必须在正文里用相对路径的 Markdown 链接引用它**，例如：

```markdown
参考 [测试模板](./test-template.js) 的标准结构。
```

> ⚠️ 没被正文引用的文件**不会被加载**。写了脚本却忘了引用，等于白放。

## 用 AI 帮你生成 Skill

不一定要手写，官方支持几种「让 AI 代劳」的方式：

- 在聊天输入框输入 **`/create-skill`**，描述你想要的能力（例如「一个用于运行和调试集成测试的 Skill」），Agent 会提问补全细节并生成 `SKILL.md` 和目录结构。
- 调试完一个复杂问题后，对 Copilot 说「**create a skill from how we just debugged that**」，把刚才的多步过程沉淀成 Skill。
- 在 Agent Customizations 编辑器的下拉菜单里选 **Generate Skill**。

> 🤖 这些生成步骤都可以让 AI 代劳；但生成出来的内容要 ⚠️ 你审一遍再提交——尤其是里面涉及的路径和命令。

## 访问控制四象限

`user-invocable` 和 `disable-model-invocation` 两个字段组合出四种行为（官方文档表格）：

| 配置 | 出现在 `/` 菜单 | 被 Copilot 自动加载 | 适用场景 |
| --- | --- | --- | --- |
| 默认（两个字段都不写） | ✅ | ✅ | 通用型 Skill |
| `user-invocable: false` | ❌ | ✅ | 背景知识型，只在相关时被加载 |
| `disable-model-invocation: true` | ✅ | ❌ | 只在需要时手动跑的 Skill |
| 两个都设 | ❌ | ❌ | 相当于禁用 |

## 测试与排错

写完后的验证清单：

1. **重载窗口**：命令面板 `Reload Window`，让 Copilot 重新发现。
2. **看 `/` 菜单**：输入 `/`，确认名字出现（除非 `user-invocable: false`）。
3. **手动调用一次**：`/你的技能名` 加一句需求，看是否按正文步骤走。
4. **测自动加载**：不提技能名、只描述任务，看 Copilot 会不会自动挑中——这最能检验 `description` 写得好不好。
5. **看诊断**：Chat 视图右上菜单 **Diagnostics**，或 `Developer: Show Agent Debug Logs`。

常见坑对照：

| 现象 | 可能原因 |
| --- | --- |
| 完全不出现、不加载 | 目录位置不对 / 目录名与 `name` 不一致 / 用了非法字符 / 没重载窗口 |
| 出现在菜单但不会自动加载 | `disable-model-invocation: true`，或 `description` 没写清「何时使用」 |
| 不出现但能自动加载 | `user-invocable: false`（这是预期行为） |
| 脚本/模板没被用到 | 正文里没写相对路径引用 |

## 官方参考

- Agent Skills 官方文档（SKILL.md 格式、命名、forked context）: <https://code.visualstudio.com/docs/agent-customization/agent-skills>
- Agent Customization 总览（生成、编辑器、评估）: <https://code.visualstudio.com/docs/agent-customization/overview>

---

← 上一章：[安装与使用现有 Skill](02-install-and-use.md) ｜ 下一章：[Skill 与 Prompt File / Instructions 的选型](04-skill-vs-others.md)
