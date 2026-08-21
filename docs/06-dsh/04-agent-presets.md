# Agent Preset（模式）

## 本章解决什么

DSH 自带四种 Agent Preset（预设模式），每种给 AI 分配的工具和规则都不同。本章帮你搞清楚：四种模式各有什么能力、什么场景该用哪个、以及如何创建自己的预设。

## 四种 Shipped Preset 一览

| Preset | 中文名 | 一句话 | 工具数量 | 适合谁 |
| --- | --- | --- | --- | --- |
| **standard** | 标准模式 | 功能完整的编码 Agent | 完整（~20 个工具） | 日常开发首选 |
| **cordis** | 创造模式 | 标准模式 + 运行时检查 + 插件创作 | 完整 + Cordis 工具集 | 想定制 Agent 的开发者 |
| **code** | PTC 模式 | 标准模式 + Code Mode SDK | 完整 + 代码批量执行 | 喜欢一次写多步操作的开发者 |
| **minimal** | 极简模式 | 只有持久 bash + 文件编辑器 | 最少（2 个工具） | 极简主义、脚本任务 |

## 四种模式的工具清单对比

### standard（标准模式）⭐ 推荐日常使用

| 工具类别 | 包含什么 |
| --- | --- |
| Shell | `bash`（macOS/Linux）/ `pwsh`（Windows）——一次性 |
| 文件系统 | `read`、`write`、`edit`、`glob`、`grep` |
| 网页搜索 | `web_search` |
| 技能系统 | `skill`（加载与调用） |
| 子代理与工作流 | `subagent`、`subagent_fork`、`workflow`、`ralph` |
| 任务管理 | `goal`、`todo_write`、`ask_user_question` |
| 计划模式 | `exit_plan_mode`（用户可触发） |
| 后台任务 | `job_list`、`job_output`、`job_kill` |
| 上下文 | 自动压缩、token 计量 |

> 💡 标准模式就是你打开 DSH 后最可能用的模式——够全面，够日常。

### cordis（创造模式）——你在看的这个文档就是这个模式写的

在标准模式基础上增加：

| 新增能力 | 说明 |
| --- | --- |
| **Cordis 检查工具** | `cordis_inspect_list`、`cordis_inspect_query`、`cordis_inspect_self`——查看运行时服务、事件、插槽、工具等 |
| **动态插件工具** | `cordis_define`、`cordis_run`、`cordis_stop`、`cordis_undefine`——在运行时定义、激活、停止、删除插件 |
| **组合创作技能** | `editing-cordis-compositions`——教 AI 如何正确编写 `cordis.yml` |

> ⚠️ cordis 模式是**信任边界（trust boundary）而非安全沙箱（sandbox）**——AI 可以评估和修改它运行在其中的代码。把它当作有 shell 访问权限来对待。

### code（PTC 模式）

在标准模式基础上增加：

| 新增能力 | 说明 |
| --- | --- |
| **Code Mode SDK** | AI 不是一次做一个工具调用，而是写一段 TypeScript 程序，一次执行多步操作。原本需要 5 轮往返的操作，变成一段代码。 |

> 💡 PTC = Programmable Tool Composition。适合「我知道要干嘛，不想等 AI 一步步来」的场景。

### minimal（极简模式）

| 工具 | 说明 |
| --- | --- |
| 持久 bash | 一个保持状态的 bash 会话（不是每次新建） |
| `str_replace_editor` | 基于字符串替换的文件编辑器（需绝对路径） |

> 💡 minimal 模式故意阉割了大部分工具，只留两个。适合极简主义、脚本自动化、或者你想「限制 AI 别乱跑」的场景。

## 如何切换 Preset

在 Web GUI 里切换：

1. 输入框打 `/preset` 或点击右上角 Preset 选择器
2. 从列表里选一个
3. 新会话会使用新 Preset（已有会话不变）

在 `settings.yaml` 里设默认：

```yaml
agent-presets:
  default: standard   # 可选: standard / cordis / code / minimal
```

## 选型建议

| 你的需求 | 推荐 Preset | 理由 |
| --- | --- | --- |
| 日常写代码、改文件、搜东西 | **standard** | 够用，不多不少 |
| 想写自定义插件扩展 DSH | **cordis** | 唯一有动态插件和运行时检查工具的模式 |
| 想批量执行多步操作、减少往返 | **code** | Code Mode 把多步合为一段 TypeScript |
| 只想要一个终端 + 编辑器 | **minimal** | 最轻量，没有杂音 |
| 想让 AI 帮你创建新的 Agent Preset | **cordis** | 只有它能改组合文件 |

## 创建自定义 Preset

DSH 允许你从 shipped preset 复制一份，修改成你自己的版本：

1. 找到要修改的原型 Preset（如 `cordis`），位于：
   ```
   /opt/homebrew/lib/node_modules/@deepseek-ai/dsh/config/agent-presets/cordis/
   ```

2. 不要直接改原文件（升级会覆盖）——**把整个目录复制到用户目录**：

   ```bash
   cp -r /opt/homebrew/lib/node_modules/@deepseek-ai/dsh/config/agent-presets/cordis/ \
        ~/.dsh/agent-presets/my-custom/
   ```

3. 编辑 `agent.cordis.yml`——这就是你的 Preset 的组合文件，每一行就是一个能力
4. 编辑 `preset.yml`——改名字和描述
5. 重启 DSH，你的自定义 Preset 会出现在列表里

> ⚠️ 关于如何正确编写 `agent.cordis.yml`（哪些放 Host 平面、哪些放 Preset 平面、什么时候需要 `isolate` realm），请读 [第 5 章：Cordis 组合系统](05-cordis-compositions.md)。

## 下一步

知道四种模式怎么选了？想了解底层的 Cordis 组合系统是怎么工作的，继续看 [第 5 章：Cordis 组合系统](05-cordis-compositions.md)。不想深究原理的话，直接跳到 [第 7 章：工具协作与选型](07-cooperation-and-selection.md)。

---

← 上一章：[Web GUI 使用](03-web-gui.md) ｜ 下一章：[Cordis 组合系统](05-cordis-compositions.md)