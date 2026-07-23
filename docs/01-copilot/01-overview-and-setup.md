# 概览与安装

## 本章解决什么
本章说明如何在 Visual Studio Code 中开始使用 Copilot（包含 GitHub Copilot 提供的 AI 功能），涵盖账号与订阅、组织策略影响、安装/启用流程，以及如何验证内联建议（Inline Suggestions）、聊天视图（Chat View）与 Agent 能否使用。提供故障排查指引并指向第 07 章的详细排错与安全检查。

## Copilot 能做什么
- 内联补全（Inline Suggestions）：在编辑器中基于上下文给出实时补全建议，按 `Tab` 接受。
- 聊天视图（Chat View / Chat）：在侧边栏以对话方式提问、让 AI 解释或修改代码。
- 内联聊天（Inline Chat）：针对选中代码在编辑器中就地发起对话并应用修改。
- Agents（在 VS Code 文档中称为 Agents Window，注意目前为 Preview）：用于更高自主性的多步、多文件任务编排与工具调用（例如运行命令、读取/修改多个文件）。
- Quick Chat：轻量化临时对话入口，适合快速查询。

这些功能由 VS Code 与 GitHub 的服务与模型能力共同提供；编辑器本身也包含用于触发和呈现这些交互的本地 UI。请始终审查 AI 生成的更改。

## Copilot 不能替你做什么
- 替代你的专业判断或对生成代码的安全性承担责任。
- 在未经校验的场景下自动发布或替你决定架构选择。

## 账号、订阅与组织策略
- 登录（GitHub sign-in）会将你的编辑器与 GitHub 帐号关联，GitHub 账号可启用免费额度或付费 Copilot 计划（具体套餐、价格与配额以 GitHub 官方说明为准）。
- 组织（Organization）策略或管理员设置可能会禁用或限制某些 AI 功能或 Agents。若在企业环境中无法使用某些功能，请联系组织管理员。
- BYOK（Bring Your Own Key）/自带模型密钥是将特定语言模型或模型提供商接入编辑器的一种方式，它与获取 GitHub Copilot 服务（由 GitHub 提供的托管计划）不是同一件事：BYOK 是连接模型的方式，而 Copilot 服务是包含授权、计量和产品集成的服务。

## 安装与启用
1. 安装或更新到最新稳定版的 Visual Studio Code（参见 VS Code 官方下载页面）。
2. 在编辑器内通过扩展视图或内置提示安装或启用 Copilot / AI 功能扩展（以编辑器中显示的官方扩展名称为准）。
3. 使用 GitHub 账号登录（Sign in with GitHub）并完成授权；若订阅生效，相关付费或免费额度将关联到你的账号。
4. 在组织环境中，如被策略限制，请向管理员确认是否需要例外或配置更改。

## 验证是否可用
- 内联建议：在打开的代码文件中输入几行代码，观察是否出现灰色补全（Inline Suggestions）；尝试按 `Tab` 接受。
- Chat View：运行命令面板中的 `Chat: Open Chat`（或编辑器侧边栏的 Chat 图标），确认可发送并收到响应。
-- Agent 可用性：在命令面板中查找 `Chat: Open Agents Window`（文档可能标注为 Preview）或搜索 `Agents`；若命令或窗口不可见，说明组织策略或当前扩展版本未启用该预览功能。

若上述任何一步失败，请参见第 07 章“故障排查与安全”获取详细的诊断步骤（包括日志位置、权限与网络设置）。

## 下一步
- 继续下一章了解聊天与内联交互：[聊天与内联交互](02-chat-and-inline.md)

## 官方参考
- [VS Code Agents 概览（官方文档，Agents / Agents Window 概述）](https://code.visualstudio.com/docs/agents/overview)
- [VS Code AI 与 Language Models 设置（官方文档，包含 BYOK/模型接入说明）](https://code.visualstudio.com/docs/agent-customization/language-models)
- [GitHub Copilot 文档（Copilot 说明与订阅入口）](https://docs.github.com/en/copilot)

---

返回主页：[章节简介](README.md) ｜ 下一章：[聊天与内联交互](02-chat-and-inline.md)
