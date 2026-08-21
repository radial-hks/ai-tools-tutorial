# Cordis 组合系统

> ⚠️ 本章面向技术人员。Cordis 是 DSH 的底层架构引擎，理解它需要一些编程概念。如果你是美术或建模同事，可以跳过本章，不影响日常使用。

## 本章解决什么

讲清楚 DSH 的核心架构：Cordis 是什么、Host 平面和 Agent Preset 平面怎么分工、一个能力是怎么被「组合」进去的。

## Cordis 是什么

**Cordis** 是 DSH 的能力编排引擎。它的核心思想很简单：

> **每一个能力都是一个「插件行（plugin row）」，写在 `cordis.yml` 文件里。DSH 启动时，把这些行按规则层叠起来，形成最终的能力组合。**

你可以把 `cordis.yml` 想象成「DSH 的基因序列」——每一行编码一个能力，行的排列顺序决定了谁先谁后、谁覆盖谁。

### 一个 Plugin Row 长什么样

```yaml
# 这是 agent.cordis.yml 中的一行
- id: tool-bash                    # 行 ID（唯一标识）
  name: '@deepseek-ai/dsh-tool-bash'  # npm 包名
  disabled: !!js process.platform === 'win32'  # 可选：在 Windows 上禁用
  config:                          # 可选：传给插件的配置
    timeoutMs: 60000
```

每一行的关键字段：

| 字段 | 说明 | 必填 |
| --- | --- | --- |
| `id` | 唯一标识符（同文件内不重复） | ✅ |
| `name` | npm 包名，指向具体的能力实现 | ✅ |
| `disabled` | 是否禁用（支持 `!!js` 动态表达式） | ❌ |
| `config` | 传给插件的配置参数 | ❌ |
| `group` | 是否是分组行（`cordis:group`），用于包裹子行 | ❌ |
| `isolate` | 隔离域（见下文） | ❌ |

## 两个平面：Host vs Agent Preset

这是理解 DSH 架构最关键的一点。

DSH 把能力分在两个「平面」里：

```
┌─────────────────────────────────────────┐
│           Agent Preset 平面              │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐ │
│  │ persona │  │ tool-*  │  │ skill-* │ │  ← 每个会话自己的工具、人格、规则
│  └─────────┘  └─────────┘  └─────────┘ │
├─────────────────────────────────────────┤
│              Host 平面                    │
│  ┌─────────┐  ┌──────────┐  ┌─────────┐ │
│  │ sandbox │  │ sessions │  │ llm     │ │  ← 所有会话共享的基础设施
│  └─────────┘  └──────────┘  └─────────┘ │
└─────────────────────────────────────────┘
```

### Host 平面——共享基础设施

Host 平面存放所有会话共享的东西：

| 放什么 | 为什么放这里 |
| --- | --- |
| 沙箱与审批机制（sandbox / approval） | 所有 Agent 共用同一套安全策略 |
| 会话持久化（session persistence） | 所有会话共用同一个存储后端 |
| 模型路由（model route） | 所有 Agent 共用同一个模型供应商配置 |
| 子代理注册表（subagent registry） | 进程级单例，跨会话查询 |
| 网页搜索服务（web search） | 所有 Agent 共用同一套搜索后端 |
| Token 计量器（token meter） | 跨会话统计，浏览器读取 |

> 💡 判断标准：一个东西如果**多个会话需要共享**或者**在 Agent 存在之前就要解析注入**，它就属于 Host 平面。

### Agent Preset 平面——每个会话自己的配置

Agent Preset 平面存放每个会话私有的东西：

| 放什么 | 为什么放这里 |
| --- | --- |
| Persona（系统提示词） | 每个 Preset 可以有不同的人格 |
| 工具（tools） | 不同 Preset 给 AI 不同的工具集 |
| 技能注册（skill registration） | 每个 Preset 可以贡献不同的技能 |
| 计划模式（plan mode） | 每个 Agent 自己的状态 |
| 压缩策略（compaction） | 每个 Preset 可以有不同的压缩配置 |
| 工作流引擎（workflow engine） | 仅供当前 Agent 使用 |

> 💡 判断标准：一个东西如果**只服务于当前 Agent**，而且其他 Preset 不需要访问它，它就属于 Preset 平面。

## Profile 的 Patch 层叠

一个 Profile 启动时，会按以下顺序层层叠加配置：

```
空根节点
  └─ 第 1 层：dsh.profile.bundles 中每个 Bundle 的 patch
  └─ 第 2 层：Profile 自身的 cordis.patch.yml
  └─ 第 3 层：全局 $DSH_HOME/cordis.patch.yml
  └─ 第 4 层：命令行 --patch 覆盖（临时）
```

后面层可以覆盖前面层的配置（比如禁用某个行、改配置参数）。这就是 DSH「可组合」的来源——你不需要改源码，只需在 patch 文件里加一行就能定制。

### 举个例子

假设默认的 `@deepseek-ai/dsh-base` Bundle 里有一行：

```yaml
- id: sandbox-policy
  name: '@deepseek-ai/dsh-sandbox-policy'
  config:
    mode: workspace-write     # 默认策略
```

你想改成 `danger-full-access`，不需要改 Bundle 源码，在 `~/.dsh/cordis.patch.yml` 里加：

```yaml
- id: sandbox-policy
  name: '@deepseek-ai/dsh-sandbox-policy'
  config:
    mode: danger-full-access     # 覆盖默认策略
```

## isolate Realm（隔离域）

Cordis 里还有一个重要的概念：**isolate realm**。

有些能力不能在 Host 平面的全局注册表里注册（会和其他 Preset 冲突），也不能直接放在 Preset 文件里（会被外部的 Host 读取器看到）。这时就需要用 `isolate` 把它包起来：

```yaml
- id: compaction
  name: cordis:group      # 这是一个分组行
  group: true
  isolate:
    compaction: true       # 在隔离域里注册 'compaction' 服务
    toolResultPruner: true # 在隔离域里注册 'toolResultPruner' 服务
  config:
    - id: compaction-basic
      name: '@deepseek-ai/dsh-compaction-basic'
```

- 不写 `isolate` → 注册到全局 Host 注册表（所有会话可见）
- `isolate: { key: true }` → 注册到当前 Agent 的私有域（其他 Preset 看不到）
- `isolate: { key: "shared-label" }` → 多个 Preset 共享同一个实例（较少用）

> 💡 `isolate` 是写自定义 Preset 时最容易出错的地方。如果忘了加 `isolate` 却注册了 Preset 级服务，DSH 会在挂载时报错。

## 两个 Preset 的 Cordis 文件对比

以 standard 和 cordis 为例：

| 行 | standard | cordis | 说明 |
| --- | --- | --- | --- |
| persona | ✅ | ✅（覆盖为 Cordis 专用人格） | cordis 改了提示词 |
| tool-fs | ✅ | ✅ | 完全一样 |
| cordis 检查/动态插件工具 | ❌ | ✅ | cordis 独有的 |
| editing-cordis-compositions 技能 | ❌ | ✅ | cordis 独有的 |

> 你可以打开这些文件看看：
> - `/opt/homebrew/lib/node_modules/@deepseek-ai/dsh/config/agent-presets/standard/agent.cordis.yml`
> - `/opt/homebrew/lib/node_modules/@deepseek-ai/dsh/config/agent-presets/cordis/agent.cordis.yml`

## 下一步

理解了两个平面和 Patch 层叠后，可以看 [第 6 章：动态插件](06-dynamic-plugins.md)——这是 Cordis 架构最实用的体现：不用重启 DSH，在运行时临时定义并运行一个新能力。

---

← 上一章：[Agent Preset](04-agent-presets.md) ｜ 下一章：[动态插件](06-dynamic-plugins.md)