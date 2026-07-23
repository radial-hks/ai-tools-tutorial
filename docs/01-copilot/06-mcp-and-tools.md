# MCP 与工具

## 本章解决什么

本章简洁说明：什么是 MCP（Model Context Protocol），它与 VS Code 内建工具、扩展提供的工具、以及 MCP 工具之间的区别；如何在工作区或用户级别配置 MCP Server；以及管理、信任与排错的要点（截至 2026-07）。

## 工具和 MCP 的关系

- **VS Code 内建工具**：编辑器自带功能（文件、终端、LSP）——不是 MCP。
- **扩展贡献的工具**：由 VS Code 扩展直接注册的命令或视图——与编辑器进程更紧密。
- **MCP 工具（MCP servers）**：独立进程或远端服务，按 Model Context Protocol 暴露工具、资源与 prompt 文件，供支持 MCP 的客户端（如 Copilot、Copilot CLI、部分 IDE）调用。

说明：MCP 是一套协议与实现生态，不应被当作“所有工具”的同义词；区分三类工具有助于安全与运维决策。

## MCP Server 能提供什么

- **Tools（工具）**：可调用的能力（例如：搜索、PR 操作、浏览器自动化）。
- **Resources（资源）**：可供模型访问的上下文数据（文件、数据库片段、索引结果）。
- **Prompts / Prompt Files**：可重用的提示模板，用于标准化对话或自动化任务。
- **MCP Apps / Agent Apps**：在客户端内运行的交互式小应用（某些客户端支持）。

使用要点：客户端会发现已注册的工具并在需要时请求执行；工具和资源由服务器端决定授权范围与输入输出契约。

## 安装 MCP Server

- 推荐优先使用受信任的扩展市场包（在扩展商店搜索 `@mcp` 标签）。
- 常用客户端命令：`MCP: Add Server`、`MCP: List Servers`、`MCP: Open User Configuration`（命令面板）。
- 配置作用域：
  - 工作区（`.vscode/mcp.json`）：项目范围，适合团队共享（可提交到仓库）。
  - 用户级（用户配置）：影响所有本地工作区；可在命令面板用 `MCP: Open User Configuration` 打开并编辑。
  - 远程工作区：在 VS Code Remote / Codespaces 等远程会话中，服务器发现与权限可能由远程宿主决定；请参阅远程环境文档以确认权限边界。

## 配置 mcp.json

下面给出一个严格的、可解析的示例（主示例只包含必要字段）。不要在 JSON 中放置密钥或凭据。

```json
{
  "servers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp"
    },
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@microsoft/mcp-server-playwright"]
    }
  }
}
```

要点：
- 切勿在仓库或 `mcp.json` 中硬编码 API 密钥或机密；使用环境变量、外部凭据文件或客户端的凭据输入机制。
- 官方配置参考：参见下方“官方参考”链接中的 MCP 配置页面。

扩展：本地 stdio 服务器可启用局部沙箱（macOS / Linux 支持 `sandboxEnabled`）。MCP sandboxing is currently not available on Windows, verified 2026-07。下面是一个包含沙箱约束的示例（单独示例，仍为严格 JSON）：

```json
{
  "servers": {
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@microsoft/mcp-server-playwright"],
      "sandboxEnabled": true
    }
  },
  "sandbox": {
    "filesystem": {
      "allowWrite": ["${workspaceFolder}/.tmp"],
      "denyRead": ["${workspaceFolder}/.env", "${workspaceFolder}/.git"],
      "denyWrite": []
    },
    "network": {
      "allowedDomains": ["example.com"],
      "deniedDomains": []
    }
  }
}
```

说明：沙箱设置显示了显式的文件系统与网络许可——这会限制工具能访问的路径和域名，但不能保证绝对安全。

注意：示例中使用 `npx -y` 启动本地服务器 —— 该命令在首次运行时会下载并执行远程发布包；请在第一次启动前审查发布者和配置。

## 管理工具与服务器

- 发现与添加：使用 `MCP: Add Server`（命令面板）或在 `.vscode/mcp.json` 中声明服务器。
  - 列表与状态：`MCP: List Servers` 可查看可用服务器与启用状态。部分客户端也在设置或扩展页提供“Configure Tools”按钮。
  - 启动/停止/日志：对于本地 stdio 服务器，客户端通常在启动时启动子进程并记录 stdout/stderr；可通过运行 `MCP: List Servers`，选择服务器，然后选择 “Show Output”（或使用扩展提供的日志视图）查看运行日志。
- 最小化工具面：仅启用当前任务需要的工具，避免一次暴露过多能力。

## 信任、密钥与沙箱

- **信任提示**：首次连接不受信任的 MCP Server 时，客户端会提示授予信任（请审查发布者与配置）。
- **重置信任**：使用 `MCP: Reset Trust`（或在客户端设置中重置）可撤销已授予的信任。
- **密钥与凭据**：不要把凭据写入 `mcp.json`；使用环境变量、操作系统凭据存储或客户端的安全输入界面。
- **沙箱说明**：沙箱能限制文件系统与网络访问（见示例），但并非万全；在生产环境优先使用受管理的远端 MCP 服务并执行代码审计。

提示：某些客户端在直接以文件方式打开 `mcp.json` 时可能会自动启动声明的本地服务器——这会触发或绕过部分交互式信任对话，具体行为以客户端版本为准，请审查客户端文档并在首次启用前手动核验配置。

## 排错

- 常见症状与排查：
  - 无法发现服务器：检查 `.vscode/mcp.json` 语法与所属作用域；运行 `MCP: List Servers`。
  - 工具调用失败：查看服务器日志（stdout/stderr）与客户端工具调用错误信息。
  - 权限/网络被阻止：确认本机防火墙、代理与远程工作区的网络策略。
- 日志与命令（示例）：
  - 在 VS Code 中：`View → Output`，选择 MCP 或扩展的日志通道。
  - 本地 stdio 服务器输出通常可在扩展的日志视图查看，或通过运行 `MCP: List Servers`，选择服务器，然后选择 “Show Output”。
  - 如果需要手动运行解析与验证：

```bash
python3 -c "import json,re; s=open('.vscode/mcp.json','r',encoding='utf-8').read(); print('ok' if 'servers' in json.loads(s) else 'invalid')"
```

## 官方参考

- MCP 官网（协议与生态）：https://modelcontextprotocol.io/
- VS Code — Add and manage MCP servers: https://code.visualstudio.com/docs/agent-customization/mcp-servers
- VS Code — MCP configuration reference: https://code.visualstudio.com/docs/agents/reference/mcp-configuration
- VS Code — Use tools in chat: https://code.visualstudio.com/docs/chat/chat-tools
- GitHub Copilot — MCP 概念页（介绍与客户端支持）：https://docs.github.com/en/copilot/concepts/context/mcp

---

← 上一节：[定制 Copilot](05-customization.md) ｜ 下一节：[安全与排错](07-safety-and-troubleshooting.md)
