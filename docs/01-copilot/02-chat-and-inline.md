# 聊天与内联交互

## 本章解决什么
本章说明 VS Code 中的四种主要 AI 交互界面（Agents Window、Chat View、Inline Chat、Quick Chat）的适用场景、入口与使用要点，并介绍内联建议（Inline Suggestions）与发送/排队/纠偏的行为模型。

## 四种交互界面怎么选
下面的矩阵帮助快速判断每个界面的适用场景与入口（优先给出命令面板命令）：

| 界面 | 界面类型 | 适合场景 | 特点 | 入口 |
|---|---:|---|---|---|
| Agents Window（Preview） | 独立窗口 / 面板 | 复杂、多步、跨文件任务 | 支持计划、运行工具、查看日志；适合高自主性 agent 工作流 | 命令面板 `Chat: Open Agents Window`（可能标注为 Preview） |
| Chat View（Chat） | 侧边栏或独立面板（与代码并列） | 持续对话、代码探索与补丁生成 | 长对话、上下文跟踪、适合逐步交互与协作 | 命令面板 `Chat: Open Chat`；侧边栏图标；快捷键（Mac: `⌃⌘I` / Win/Linux: `Ctrl+Alt+I`） |
| Inline Chat（内联聊天） | 编辑器内联气泡（就地） | 就地修改、针对选中代码的精确指令 | 不离开编辑上下文、适合小范围精确改动 | 快捷键（Mac: `⌘I` / Win/Linux: `Ctrl+I`）；或在命令面板中搜索 "Inline Chat" |
| Quick Chat（快速聊天） | 轻量弹出 / 快速面板 | 临时查询、便捷问答 | 低摩擦、短会话、快速答案 | 快捷键（Mac: `⇧⌥⌘L` / Win/Linux: `Ctrl+Shift+Alt+L`）；或在命令面板中搜索 "Quick Chat" |

注意：内联建议（Inline Suggestions）是编辑器级别的实时补全能力，不等同于上述聊天面板，是独立的编辑交互能力。

## Chat View
- 适用：持续对话、探索大范围代码、让 AI 生成补丁或解释结果。
- 入口：`Chat: Open Chat`（命令面板）或侧边栏图标；快捷键：Mac `⌃⌘I`，Win/Linux `Ctrl+Alt+I`。
- 行为：发送请求后会得到一次或多段回复；如果请求触发工具执行（如运行测试），聊天界面会在工具完成后继续处理后续交互。

## Agents Window（Preview）
- 适用：需要多步计划、工具链调用或跨文件变更的高阶任务。
- 入口：`Chat: Open Agents Window`（命令面板，可能标注为 Preview）。
- 说明：该窗口通常允许启动或审查 agent 的计划、查看运行日志与逐步接受修改。

## Inline Chat
- 适用：就地修改、快速重构、针对选中代码的精确指令。
- 入口：在命令面板中搜索“Inline Chat”或使用快捷键；快捷键：Mac `⌘I`，Win/Linux `Ctrl+I`。

## Quick Chat
- 适用：临时询问、片段级别问题。
- 入口：在命令面板中搜索“Quick Chat”或使用快捷键；快捷键：Mac `⇧⌥⌘L`，Win/Linux `Ctrl+Shift+Alt+L`。

## 内联建议（Inline Suggestions）
- 适用：实时补全、片段建议。触发方式：在编辑器中输入代码，建议自动出现。
- 接受：`Tab`；拒绝：`Esc`；切换建议：Option/Alt + `]` 或 `[`；部分接受（逐词）：`Cmd+→`（Mac）或 `Ctrl+→`（Win/Linux）。

## 发送、排队与纠偏
- 排队（Queue / Add to Queue）：当你选择“排队”时，新消息会在当前响应完成后自动发送到模型，而不会中断正在进行的执行。
- 转向（Steer / Steer with Message）：发送一条用于“转向”的消息会在当前工具执行完成后被处理；如果 agent 正在运行外部工具，转向通常等待当前执行阶段结束，然后应用新的指令。
- 停止并发送（Stop and Send / Cancel & Send）：会取消当前正在进行的请求或正在运行的工具（若支持取消），并立即将新消息作为新的请求开始处理。

行为要点：排队不会中断当前运行，转向会在当前执行点后引入新指令，而停止会尝试取消当前执行并立刻处理新请求；具体表现取决于当前交互是否涉及外部工具或长时任务。

## 常用快捷键
- 打开 Chat 视图：Mac `⌃⌘I`，Win/Linux `Ctrl+Alt+I`
- 启动 Inline Chat：Mac `⌘I`，Win/Linux `Ctrl+I`
- 打开 Quick Chat：Mac `⇧⌥⌘L`，Win/Linux `Ctrl+Shift+Alt+L`
- 接受内联补全：`Tab`；拒绝：`Esc`

- 注意：上述快捷键可在 VS Code 中重新映射；在“Keyboard Shortcuts”（键盘快捷方式）中验证或修改快捷键分配。

**发送后的审查与接受：** 内联聊天与 Agents 修改会以 diff 或编辑建议形式出现；使用 Keep / Undo（保留 / 撤销）来逐项接受或拒绝更改。若修改范围较大或涉及安全/合规，请参考第 07 章关于审查、检查点与回滚的详细流程。

## 官方参考
- [Chat View 与 Agents 文档（官方）](https://code.visualstudio.com/docs/chat/chat-overview)
- [Inline Suggestions 与编辑器集成（官方）](https://code.visualstudio.com/docs/copilot/ai-powered-suggestions)

---

← 上一章：[概览与安装](01-overview-and-setup.md) ｜ 下一章：[上下文与提示工程](03-context-and-prompts.md)
