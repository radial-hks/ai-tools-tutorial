# 上下文与提示

## 本章解决什么
本章说明如何把 VS Code 中的 Context（上下文）传给 AI（official English: Context），以及如何构造可执行的 Prompt（提示）。区分编辑器自动提供的隐式上下文与 Agent（Agent）可主动检索或决定的额外上下文，给出实践模板与风险提示。

## 上下文从哪里来
编辑器在会话启动时通常会隐式包含少量上下文，例如当前活动文件、当前选区以及文件名（参见官方文档）。下表列出常见的上下文项示例，作为通过输入框的显式引用（例如使用自动补全/选择器）可添加的内容。是否可用由客户端版本与输入自动补全决定；Agent 在运行时可决定是否主动检索额外文件或在 codebase 中搜索（即“主动发现”）。请勿假设表中所有项会自动传送。

| 上下文项（示例） | 说明 |
|---|---|
| `file` | 通过选择器把某个文件或一组文件的内容包含为上下文 |
| `folders` / `symbols` | 通过选择器定位项目中的文件夹或代码符号以便引用相关位置 |
| `codebase` | 对整个仓库做范围性搜索或索引查询（作为显式请求发起） |
| `selection` | 把当前编辑器选中的文本片段作为上下文 |
| `terminalSelection` | 通过选择器或复制操作把终端中选中的文本或输出作为上下文 |
| `changes` | 把未提交或暂存的 Git 变更摘录作为上下文（显式请求） |
| `problems` | 引用问题面板（Problems）中列出的 diagnostics / 错误信息 |
| `fetch` | 通过 `#fetch` 或选择器提供的 URL 抓取外部网页或文档片段 |

## 使用 # 引用显式上下文
在输入框内使用 `#`（或编辑器的 Add Context 按钮 / 拖放文件）显式引用上下文项。常见示例（实际语法和可选项由当前自动补全决定）：

- `#file` 然后通过自动补全/选择器选择要包含的文件或文件集（例如单个文件或 glob）。
- `#selection` 把当前编辑器选区传入会话。
- `#fetch` 然后通过自动补全或输入 URL 选择要抓取的页面或文档。
- `#folders` / `#symbols` 可用时通过选择器定位文件夹或代码符号。

这些只是示例；确切的命令和可选项以当前输入框的自动补全为准。

## 图片与浏览器上下文
可以把图片（截图、设计稿）拖入聊天，或通过浏览器集成交互式选择页面元素来添加 HTML/CSS 片段或截图作为上下文。集成浏览器通常允许选择 DOM 元素并附带截图或所选元素的 HTML/CSS 片段；控制台输出或浏览器工具产生的日志在支持的工具中可能作为额外输出提供。

隐私提示：在附加页面或浏览器内容前，先确认没有泄露敏感信息或受版权/隐私限制的内容。

## 写出可执行的请求
使用严格的四元素 Prompt 方法（必须包含且按顺序给出）：

目标、上下文、约束、验收

例如（开发者示例 1）：

目标：实现用户登录 API（English: Goal）。
上下文：#file:src/auth/*.ts，#changes 包含注册分支的最新修改（English: Context）。
约束：使用 Express + TypeScript，覆盖单元测试；不改动现有数据库模式（English: Constraints）。
验收：新增 endpoint /login，包含 3 个单元测试通过，CI 本地运行无失败（English: Acceptance）。

开发者示例 2（重构）：

目标：把 `utils/parse.ts` 中的回调改为 async/await。
上下文：#file:utils/parse.ts、#selection 指向具体函数。
约束：功能不变，保留现有外部 API，新增单元测试覆盖边界情况。
验收：变更通过 lint、测试，提交 diff 描述清晰。

美术/建模示例 1：

目标：把 1024px 素材批量缩放为 512px 并去除透明通道。
上下文：上传的素材压缩包或 `#file:tools/image_batch.py`（如果已有脚本）。
约束：使用 Pillow，输出 PNG，不改变文件名结构。
验收：输出文件夹包含按要求尺寸的 512px PNG，压缩比合格。

美术/建模示例 2：

目标：生成可用于实时渲染的 PBR base color 贴图（1024->512）。
上下文：提供的艺术草图（图片附件）和目标引擎（例如 Unity URP）。
约束：输出 PNG 512x512，sRGB 色域，命名约定为 name_basecolor.png。
验收：贴图在目标引擎中加载无色偏，材质渲染结果截图作为验收。

## 管理长会话
分会话（separate sessions）用于不同任务或长期任务的阶段化（例如 research / implement / review）。仅在客户端支持时使用会话压缩（compact）或分叉（fork）。输入时可通过 slash 自动补全（/compact、/fork）发现这些命令；若没有补全，说明当前版本不支持该特性。

会话策略建议：

- 对无关任务新开会话，避免上下文混淆。
- 对探索性任务使用只读、Ask 风格会话；对改动性任务使用受控权限并逐步放开。
- 在会话中定期导出关键上下文（例如测试输出、诊断）以便审计。

## 常用任务模板
- 生成单元测试：目标/上下文/约束/验收 四段，附示例输入输出。
- 重构函数：目标/上下文（#selection）/约束（保持 API）/验收（测试通过）。
- 生成发布说明：目标/上下文（#changes）/约束（按类型分组）/验收（Markdown 格式）。

## 风险与限制
- 上下文可能不完整或已过期（stale）：AI 可能基于旧代码做出修改。
- 上下文可能不可信：不要把来自未知源的文本或脚本作为权威依据。
- 任何可能包含密钥、密码、私有凭证的内容都不得粘贴入聊天或上下文（绝对禁止粘贴 secrets）。
- Agent 或模型可能会“编造”不存在的文件路径或依赖（hallucination），因此在允许其改写代码前，请先审查计划和 diff。

## 官方参考
- VS Code Chat 概览（English）： [Chat in Visual Studio Code](https://code.visualstudio.com/docs/chat/chat-overview)
- Copilot Chat: 上下文与工作流（English）： [Copilot Chat Context and Workflow](https://code.visualstudio.com/docs/chat/copilot-chat-context)

← 上一节：[聊天与内联交互](02-chat-and-inline.md) ｜ 下一节：[Agent 与工作流 →](04-agents-and-workflows.md)
