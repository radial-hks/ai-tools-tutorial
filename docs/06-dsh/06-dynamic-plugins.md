# 动态插件

> ⚠️ 本章面向技术人员。动态插件是 DSH 最强大的扩展机制，但也是 trust boundary（信任边界）而非 sandbox（安全沙箱）——你可以把它当作有 shell 访问权限来操作。

## 本章解决什么

学会动态插件的完整生命周期：什么时候该用、怎么定义、怎么运行、怎么更新、怎么停止、怎么删除。本章假设你已经在 cordis Preset 下工作（只有 cordis 模式提供动态插件工具）。

## 什么时候该用动态插件

动态插件是在 DSH **运行时临时扩展能力**的机制。它适合这些场景：

| 场景 | 适合吗 | 理由 |
| --- | --- | --- |
| 想给当前会话临时加一个工具 | 🟢 最佳场景 | 定义 → 运行，不需要改任何源文件 |
| 想在 Web GUI 上加一个可视化组件 | 🟢 很适合 | Host 端跑逻辑，Client 端跑界面 |
| 想对 DSH 做永久性修改 | 🔴 不适合 | 用自定义 Preset（第 4 章），重启后还在 |
| 想分发一个团队共享的扩展 | 🔴 不适合 | 用 Skill 或自定义 Preset（静态的，可复制） |

> 💡 动态插件的核心价值是「**临时、快速、不需要重启**」。如果要永久修改，去改 Preset 文件。

## 动态插件的生命周期

```
define → run → update → stop → undefine
  │       │       │       │       │
  │       │       │       │       └── 永久删除
  │       │       │       └── 暂时停用（保留代码，可以重新 run）
  │       │       └── 换一个新版本
  │       └── 首次激活 / 重启 / 回滚
  └── 创建代码（只记录，不执行）
```

### 六个操作的详细说明

| 操作 | 工具 | 说明 |
| --- | --- | --- |
| **定义** | `cordis_define` | 创建代码包，只验证语法和参数，不执行 apply |
| **运行** | `cordis_run` | 激活一个代码包——首次激活或回滚用 `mode: "run"`，换版本用 `mode: "update"` |
| **更新** | `cordis_run` + `mode: "update"` | 停止当前版本，激活新版本 |
| **停止** | `cordis_stop` | 停掉当前运行的版本（取消审批请求），保留所有代码和授权 |
| **删除** | `cordis_undefine` | 永久停止并删除插件和所有代码包 |

## 动态插件有两个端

| 端 | 在哪里运行 | 适合做什么 |
| --- | --- | --- |
| **Host** | DSH 的 Node.js 进程 | 文件、网络、命令、Agent/Session 访问、Host 事件、模型工具、JSON 接口供 Client 调用 |
| **Client** | 你的浏览器页面 | 主题、布局、页面状态、工具卡片、Slot UI |

Host 和 Client 通过 **Package 私有的 JSON 方法**通信：

- Host 端用 `harness.handle(method, handler)` 注册可被 Client 调用的方法
- Client 端用 `host.call(method, args)` 调用 Host 的方法
- 方向是 Client → Host，数据必须是可序列化的 JSON

## 实际操作示例

### 1. 定义一个新插件

```
给 AI 的指令：
  定义一个动态插件，在 Host 端注册一个叫 `hello` 的工具，
  当 AI 调用它时，返回 "Hello from my first dynamic plugin!"

  idPrefix 用 "hello"
```

AI 会调用 `cordis_define`，创建第一个代码包（Package），返回 `pluginId` 和 `packageId`。

### 2. 激活这个插件

```
给 AI 的指令：
  运行刚才定义的插件（用刚刚返回的 pluginId 和 packageId）
```

AI 调用 `cordis_run`，插件开始生效——你会在工具列表里看到 `hello` 工具。

### 3. 修改插件：定义新版本

```
给 AI 的指令：
  修改 hello 插件，加上当前时间戳
```

AI 调用 `cordis_define`（`kind: "existing"`，用同一个 `pluginId`），创建一个新 Package，然后用 `cordis_run`（`mode: "update"`）切过去。

### 4. 停止 / 删除

- 暂时不用了：`cordis_stop`——保留代码，随时可以 `cordis_run` 回来
- 确认不要了：`cordis_undefine`——永久删除，不可恢复

## 安全与注意事项

| 注意点 | 说明 |
| --- | --- |
| **不是安全沙箱** | 动态插件能访问实时运行时的 Service、Event、Tool，把它当作 shell 访问对待 |
| **可能触发审批** | Client 端插件需要用户审批后才能激活 |
| **失败不回滚** | 更新失败不会自动回到旧版本，需要手动 `cordis_run` 以 `mode: "run"` 回滚 |
| **停止不丢代码** | `cordis_stop` 只停用，不删除；版本指针和授权也保留 |
| **重启后不保留** | 动态插件只在当前 DSH 进程中有效，重启 DSH 就没了 |

## 什么时候把动态插件升级为永久 Preset

如果发现某个动态插件的功能要长期使用：

| 步骤 | 操作 |
| --- | --- |
| 1 | 把动态插件的 Host/Client 代码整理好 |
| 2 | 复制一个 shipped preset 到 `~/.dsh/agent-presets/my-preset/` |
| 3 | 在 `agent.cordis.yml` 里添加对应的 plugin row，引用你的代码逻辑 |
| 4 | 重启 DSH，切换到新 Preset |

> 💡 从「临时动态插件」到「永久 Preset」是 DSH 的典型开发流程：先用动态插件快速验证想法，确认好用后再固化为 Preset。

## 下一步

动态插件是 DSH 的终极武器。学完了？来看 [第 7 章：工具协作与选型](07-cooperation-and-selection.md)，把 DSH 放进你的工具矩阵里，看看六件套全景怎么排布。

---

← 上一章：[Cordis 组合系统](05-cordis-compositions.md) ｜ 下一章：[工具协作与选型](07-cooperation-and-selection.md)