# 安装 OpenViking 并接入 Pi

## 本章解决什么

带你在本机把两件事装好：一是跑起 **OpenViking 服务端**，二是给 **Pi** 装上官方扩展、让 Pi 具备跨会话记忆。全程以官方文档为准，命令都是真实可复制的。

> ⚠️ **本机版本事实（供参考）**：pi `1.2.0`、herdr `0.9.3`、OpenViking 服务端 `v0.5.0`。OpenViking 及其 Pi 集成都是快速迭代项目，下文涉及的版本与命令核验于 **2026-10**（Windows 原生环境），动手前请以官方最新文档为准。

## 整体思路：三步

| 步骤 | 做什么 | 验证 |
| --- | --- | --- |
| 第一步 | 安装并启动 OpenViking 服务端 | `curl http://localhost:1933/health` 返回 `{"status": "ok"}` |
| 第二步 | 给 Pi 装 OpenViking 扩展 | `pi list` 能看到 `extensions/openviking` |
| 第三步 | 验证召回与上下文接管生效 | Pi 底部状态栏出现 `OV ✓` |

## 第一步：安装并启动 OpenViking 服务端

### 前置条件

| 条件 | 说明 | 检查 |
| --- | --- | --- |
| Python 3.10 及以上 | 服务端是 Python 包 | `python3 --version` |
| 一个 Embedding 模型 | 向量化与语义检索用 | 见下文「配置模型」 |
| （可选）一个 VLM 模型 | 多模态理解与记忆提取用 | 见下文「配置模型」 |

> 💡 模型可用火山引擎（豆包）、OpenAI、Codex OAuth、Kimi、GLM、本地 Ollama 等。⚠️ 团队若有统一模型账号，以技术负责人给的配置为准。

### 安装

用官方推荐的方式安装：

```bash
# uv（官方推荐）
uv tool install openviking --upgrade

# 或 pip
pip install openviking --upgrade --force-reinstall
```

装完会多出两个命令：客户端 `ov`（别名 `openviking`）和服务端 `openviking-server`。

### 配置模型

首次配置直接走向导，最省事：

```bash
openviking-server init    # 交互式选择 provider，生成配置模板
openviking-server doctor  # 校验配置是否可用（各 provider 鉴权状态）
```

向导会在 `~/.openviking/ov.conf` 生成配置。核心是两段：`embedding`（向量化）和 `vlm`（多模态 / 记忆提取）。手动写的话长这样（以火山引擎为例）：

```json
{
  "embedding": {
    "dense": {
      "api_base": "https://ark.cn-beijing.volces.com/api/v3",
      "api_key": "<your-api-key>",
      "provider": "volcengine",
      "dimension": 1024,
      "model": "doubao-embedding-vision-251215",
      "input": "multimodal"
    }
  },
  "vlm": {
    "api_base": "https://ark.cn-beijing.volces.com/api/v3",
    "api_key": "<your-api-key>",
    "provider": "volcengine",
    "model": "doubao-seed-2-0-lite-260428"
  }
}
```

> 把上面的 `<your-api-key>` 换成你的真实 API Key，模型名和 endpoint 以团队或云厂商实际提供的为准。

### 启动服务

```bash
openviking-server
# 看到这一行即启动成功：
# INFO:     Uvicorn running on http://0.0.0.0:1933
```

默认监听 `1933` 端口。配置在别处时用 `openviking-server --config /path/to/ov.conf`；改端口用 `--port`。

### 验证服务

```bash
curl http://localhost:1933/health
# {"status": "ok"}
```

> 💡 Web Studio 也会在 `http://localhost:1933/studio` 提供（pip / pipx 安装即自带，无需 Docker），可以在浏览器里可视化浏览记忆库。

### 配置 CLI（可选但推荐）

给 `ov` 命令写一份连接配置 `~/.openviking/ovcli.conf`：

```json
{
  "url": "http://localhost:1933",
  "api_key": "your-key"
}
```

> ⚠️ 本地默认模式无需 API Key；只有服务端启用鉴权后才需要填。API Key 等同于密码，别写进会被提交到 Git 的文件。

## 第二步：给 Pi 装上 OpenViking 扩展

### 前置条件

- 已装好 **Pi**：`npm i -g @earendil-works/pi-coding-agent`；
- OpenViking 服务端已启动、可达（`curl http://localhost:1933/health`）。

### 安装扩展

官方安装器一行搞定。pi 与其他记忆插件共用同一个安装器，会询问语言、要安装的 harness（接入哪种 agent）、下载源和 OpenViking 凭据，每一步都是幂等的：

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/volcengine/OpenViking/main/examples/memory-plugin-shared/install.sh) --harness pi
```

GitHub 访问困难的地区，用火山引擎镜像运行同一个安装器：

```bash
bash <(curl -fsSL https://ovrelease.tos-cn-beijing.volces.com/memory-plugin-shared/install.sh)
```

安装器会把扩展复制到 `~/.pi/agent/extensions/openviking`（保留已有的 `config.json`），用 `pi install` 完成注册，并配置 `~/.openviking/ovcli.conf`。

也可以从仓库手动复制安装：

```bash
git clone https://github.com/volcengine/OpenViking.git
cd OpenViking
mkdir -p ~/.pi/agent/extensions
cp -rL examples/pi-coding-agent-extension ~/.pi/agent/extensions/openviking
pi install ~/.pi/agent/extensions/openviking
```

### Windows 原生安装注意（2026-10 实测）

上面两条安装路径都假定你在 bash 环境。Windows 原生跑 Pi 时有四个坑，都是实测踩过的：

| 坑 | 说明 | 正确做法 |
| --- | --- | --- |
| `bash <(curl ...)` 不可用 | 进程替换是 bash 语法，PowerShell 没有 | 先把 `install.sh` 下载下来再执行，或直接走上面的「手动复制安装」 |
| 来源是 WSL 里的 symlink | WSL 侧 `~/.pi/agent/extensions/openviking` 可能本身是指向源码仓库的软链 | **`cp -rL` 解引用**复制真实内容；`cp -a` 只会复制出悬空链接 |
| 凭据重复配置 | Windows 的 `%USERPROFILE%\.openviking\ovcli.conf` 与 WSL 侧本来就是同一把 key | 直接复用，无需覆盖；不确定时先 sha256 比对 |
| omp 不读 pi 的扩展目录 | omp 是 pi 内核独立发行版，自动发现根目录是 `~/.omp/agent/extensions/` | 在该目录下建指向 pi 侧真身目录的 **junction**（`New-Item -ItemType Junction`），保持单一真身 |

### 配置

扩展的**凭据**（连到哪个服务端、用什么 key）解析顺序：`OPENVIKING_*` 环境变量 → `~/.openviking/ovcli.conf` → `~/.openviking/ov.conf`。刚装完还没配过，可运行一次向导：

```bash
node ~/.pi/agent/extensions/openviking/scripts/setup.mjs
```

扩展的**行为**配置存在 `~/.pi/agent/extensions/openviking/config.json`（重装会保留你的修改）。关键几项：

| 配置项 | 默认值 | 作用 |
| --- | --- | --- |
| `scoreThreshold` | 0.35 | 召回结果的最低分数 |
| `recallTokenBudget` | 2000 | 每次召回注入的 Token 预算 |
| `commitTokenThreshold` | 20000 | 累计多少 Token 后提交一次会话 |
| `peerSource` | git | Peer 派生方式（按 Git 仓库 origin） |
| `takeover.enabled` | true | 上下文接管开关 |
| `takeover.tokenThreshold` | 30000 | 上下文接管触发的 Token 阈值 |

## 第三步：认识 Pi 里的原生工具集

装好后，Pi 会自动获得 7 个 `viking_*` 原生工具，模型可以按需调用它们查询和写入记忆：

| 工具 | 作用 |
| --- | --- |
| `viking_search` | 语义搜索记忆库 |
| `viking_read` | 读取一个 `viking://` URI 的内容 |
| `viking_browse` | 浏览目录结构 |
| `viking_remember` | 写入一条记忆 |
| `viking_forget` | 删除一条记忆 |
| `viking_add_resource` | 添加一个资源（URL） |
| `viking_archive_expand` | 展开归档概览（配合上下文接管） |

### 自动召回与上下文接管

扩展接入后，Pi 的会话流程变成：

- **自动召回（Recall）**：每个 Prompt 前向 OpenViking 做语义召回，命中的记忆作为隐藏上下文注入，模型"自动想起"之前的事；
- **自动捕获（Capture）**：每个 turn（每一轮对话）把用户与助手消息（含工具调用）捕获进 Session，累计 Token 超过阈值后提交；
- **上下文接管（Context Takeover）**（默认开启）：提交后，用归档概览替换已提交的本地历史，模型上下文里只保留近期 live tail——长会话不再撑爆上下文窗口。

> ⚠️ **核验时间：2026-10**。上下文接管属于快速迭代功能，默认开启且 fail-open（写不进去会回退到保留完整历史），具体阈值以 `config.json` 和官方文档为准。

## 验证安装生效

按顺序做三件事：

1. `curl http://localhost:1933/health` 返回 `{"status": "ok"}`——服务端在线；
2. `pi list` 能看到 `extensions/openviking`——扩展已注册；
3. 启动 `pi`，底部状态栏出现 `OV ✓`；输入 `/viking` 查看当前会话映射；再问 Pi 一个你在早前会话里提过的事，确认召回生效。

`/viking commit` 可手动强制提交当前会话。

## 常见故障排查

| 现象 | 排查方向 |
| --- | --- |
| 状态栏显示 `OV ✗` 或提示 server not reachable | `curl http://localhost:1933/health`；检查 `~/.openviking/ovcli.conf` 的 endpoint |
| 扩展没有加载 | `pi list` 应能看到 `extensions/openviking`；重跑安装器或执行 `pi install ~/.pi/agent/extensions/openviking` |
| 报错提到 `@mariozechner/*` | 是 pi 迁移到 `@earendil-works/*` 之前的旧副本，重跑安装器 |
| OpenViking 返回 401 / 403 | 检查 `OPENVIKING_API_KEY`；trusted-mode 部署还要检查 `OPENVIKING_ACCOUNT` 和 `OPENVIKING_USER` |
| 召回为空 | 确认服务端已有记忆，且 prompt 长度超过 `minQueryLength` |

> 🤖 报错看不懂时，把整段报错发给 Pi 或 Copilot，让它解释并给命令。

## 参考

- OpenViking 官网：<https://www.openviking.ai/>
- 快速开始（服务端模式）：<https://docs.openviking.ai/zh/getting-started/03-quickstart-server>
- Pi 集成官方文档：<https://docs.openviking.ai/zh/agent-integrations/11-pi>
- Pi 扩展源码：<https://github.com/volcengine/OpenViking/tree/main/examples/pi-coding-agent-extension>
- OpenViking 安装器（memory-plugin-shared）：<https://github.com/volcengine/OpenViking/tree/main/examples/memory-plugin-shared>

---

← 上一节：[OpenViking 是什么](01-what-is-openviking.md) ｜ 下一节：[Pi + herdr 多 Agent 组合](03-pi-herdr-multiagent.md)
