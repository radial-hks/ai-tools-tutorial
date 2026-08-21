# DeepSeek Harness（DSH）团队参考手册

**受众：** 技术人员建议精读全文（第 5–6 章涉及 Cordis 组合系统和动态插件，技术门槛偏高）；无编程基础的美术、建模同事建议重点读第 1–4 章。

**核验基线：** 内容按本机实际环境核验于 2026-08，dsh `0.1.1-rc.2`。发布前建议技术负责人按团队实际模型供应商和安装分发方式做最后确认。

**一句话定位：** DeepSeek Harness（`dsh`）是一套可组合的 AI Agent 运行时框架。它通过 **Cordis 组合系统** 把所有能力（终端、文件、模型、工具、技能等）都变成「插件行」，按规则层叠成你需要的 Agent；支持 Web GUI、终端 TUI 和无头模式三种使用方式，并允许在运行中动态定义并加载新插件。

**标记约定：** 全手册沿用三个符号标注任务分工：

- 🤖 **AI 可替代**：让 DSH 帮你解释、生成命令、排查错误。
- ⚠️ **必须由你决定**：账号、密钥、路径、最终效果、安全确认等，需要你亲自判断。
- 💡 **小提示**：补充说明、记忆口诀或可选做法。

## 按任务查找

| 任务类型 | 目标文档 |
| --- | --- |
| 搞懂 DSH 到底是什么、和 Copilot / Hermes / Pi 有什么不同 | [01-what-is-dsh.md](01-what-is-dsh.md) |
| 安装并启动 DSH | [02-installation.md](02-installation.md) |
| 上手使用 Web GUI（界面、会话、快捷键） | [03-web-gui.md](03-web-gui.md) |
| 了解四种 Agent 模式的能力差异、何时用哪个 | [04-agent-presets.md](04-agent-presets.md) |
| 理解 Cordis 组合系统（Host 平面 vs Preset 平面、插件行、patch 层叠） | [05-cordis-compositions.md](05-cordis-compositions.md) |
| 掌握动态插件生命周期（define → run → update → stop → undefine） | [06-dynamic-plugins.md](06-dynamic-plugins.md) |
| 搞清楚 DSH / Copilot / Hermes / Pi / OMP / herdr 六者协作全景与选型 | [07-cooperation-and-selection.md](07-cooperation-and-selection.md) |

## 章节导航（估算阅读时间）

| 章 | 标题 | 内容 | 受众 | 估计阅读时间 |
| ---: | --- | --- | --- | ---: |
| 1 | [DSH 是什么](01-what-is-dsh.md) | 产品定位、核心概念（Profile/Cordis/Agent Preset）、三种入口模式、与现有工具差异 | 全员 | 6 分钟 |
| 2 | [安装 DSH](02-installation.md) | 前置条件、npm 安装、配置模型、首次启动、验证与排错 | 全员 | 10 分钟 |
| 3 | [Web GUI 使用](03-web-gui.md) | 界面布局、会话管理、消息交互、快捷键、设置 | 全员 | 10 分钟 |
| 4 | [Agent Preset（模式）](04-agent-presets.md) | 四种 shipped preset 能力对比、选型建议、创建自定义 preset | 全员 | 8 分钟 |
| 5 | [Cordis 组合系统](05-cordis-compositions.md) | Host 平面 vs Agent Preset 平面、plugin row、profile patch 层叠、isolate realm | 技术人员 | 12 分钟 |
| 6 | [动态插件](06-dynamic-plugins.md) | 动态插件生命周期、Host/Client 双端、定义与激活流程 | 技术人员 | 12 分钟 |
| 7 | [工具协作与选型](07-cooperation-and-selection.md) | 六件套协作全景、选型决策表、学习路径建议 | 全员 | 8 分钟 |

## 阅读建议

- **只想装好能用**：读 [第 2 章](02-installation.md)，把命令发给 Pi / Copilot / DSH，让它边解释边带你装。
- **先搞清楚「DSH 又是啥」**：读 [第 1 章](01-what-is-dsh.md)。
- **已经装好了、想学会日常用**：从 [第 3 章](03-web-gui.md) 开始。
- **想知道四种模式怎么选**：读 [第 4 章](04-agent-presets.md)。
- **想理解 DSH 的架构原理**：读 [第 5 章](05-cordis-compositions.md)。
- **想写自定义插件扩展 DSH**：读 [第 6 章](06-dynamic-plugins.md)。
- **不确定该用 DSH / Copilot / Hermes / Pi 中的哪一个**：读 [第 7 章](07-cooperation-and-selection.md) 的选型表。

## 官方基线参考

- DSH npm 包：<https://www.npmjs.com/package/@deepseek-ai/dsh>
- 安装位置：`/opt/homebrew/lib/node_modules/@deepseek-ai/dsh/`
- 用户配置：`~/.dsh/settings.yaml`
- 用户会话：`~/.dsh/sessions/`
- Web GUI 默认端口：`http://127.0.0.1:3080`

---

← 返回主页：[AI 工具团队参考手册](../../README.md) ｜ 下一章：[DSH 是什么](01-what-is-dsh.md)