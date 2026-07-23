# MCP：连接外部工具

## 什么是 MCP

MCP（Model Context Protocol，模型上下文协议）是一个开放标准，让 AI 能连接外部工具和服务。

简单理解：Copilot 默认只能操作 VS Code 内部的文件和终端。通过 MCP，你可以让它：
- 查询数据库
- 调用外部 API
- 操作浏览器（Playwright MCP）
- 访问 GitHub（GitHub MCP）
- 连接任何 MCP 兼容的工具

比喻：MCP 就像给 AI 装了"手"，让它不只会在编辑器里改字，还能去操作其他软件。

## 如何安装 MCP Server

### 方法一：从扩展商店安装（推荐）

1. 打开扩展视图 `⇧⌘X`（Mac）/ `Ctrl+Shift+X`（Win）
2. 搜索 `@mcp`，查看可用的 MCP Server
3. 点击 Install 安装

例如安装 Playwright MCP Server（用于浏览器操作）：

1. 搜索 `@mcp playwright`
2. 安装
3. 确认信任此服务器
4. 在聊天中使用：`打开 code.visualstudio.com，拒绝 cookie，截图首页`

### 方法二：手动配置

创建 `.vscode/mcp.json` 文件：

```json
{
  "servers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp"
    },
    "playwright": {
      "command": "npx",
      "args": ["-y", "@microsoft/mcp-server-playwright"]
    }
  }
}
```

### 配置位置

| 位置 | 路径 | 作用范围 |
|------|------|---------|
| 工作区 | `.vscode/mcp.json` | 当前项目，可提交到 Git 共享给团队 |
| 用户级 | 运行 `MCP: Open User Configuration` | 所有项目 |

### 方法三：命令行添加

```bash
code --add-mcp <server-name> --command <command> --args <args>
```

## MCP Server 类型

| 类型 | 传输方式 | 适合场景 |
|------|---------|---------|
| Local（stdio） | 本地子进程 | IDE 内工具、项目特定集成 |
| Remote（HTTP/SSE） | 网络请求 | 团队共享服务、云端 API |

## 安全提示

- 本地 MCP Server 可以在你的机器上运行任意代码
- 只安装来自可信来源的 MCP Server
- 安装前审查发布者和配置
- 企业/组织管理员可能需要先在组织设置中启用 MCP

## 工具管理

安装 MCP Server 后：
- 在聊天输入框点击 "Configure Tools" 按钮查看所有可用工具
- 可以按需开关特定工具
- 在 Prompt File / Custom Agent 中用 `<server>/*` 引用某个 MCP Server 的所有工具

---

← 上一节：[进阶定制](05-advanced-customization.md) ｜ 下一节：[最佳实践 →](07-best-practices.md)
