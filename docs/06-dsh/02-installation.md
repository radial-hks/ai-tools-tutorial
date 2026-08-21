# 安装 DSH

## 本章解决什么

从零开始安装 DSH，配置模型，验证能正常启动。

## 前置条件

| 条件 | 最低要求 | 怎么检查 | 如果没有 |
| --- | --- | --- | --- |
| Node.js | ≥ 20 | `node --version` | [nodejs.org](https://nodejs.org/) 下载安装 |
| pnpm | 最新 | `pnpm --version` | `npm install -g pnpm` |
| 模型 API 密钥 | 至少一个供应商 | 看你的账号 | 去对应的模型供应商注册获取 |

> ⚠️ DSH 本身不提供模型服务——你需要有一个模型供应商的 API 密钥（如 DeepSeek 官方、OpenRouter 等）。团队可能统一分配，请向技术负责人确认。

## 安装 DSH

```bash
# 全局安装
npm install -g @deepseek-ai/dsh

# 验证安装
dsh --version
# 应输出类似: 0.1.1-rc.2
```

安装后，`dsh` 命令和配置文件目录 `~/.dsh/` 会自动创建。

## 首次启动前：配置模型

DSH 默认使用 DeepSeek 官方模型，但你可能需要用自己的供应商。配置文件在 `~/.dsh/settings.yaml`：

```yaml
# ~/.dsh/settings.yaml（示例）
agent-default-model:
  provider: openrouter          # 模型供应商
  model: deepseek/deepseek-v4-pro  # 模型 ID

llm-pi-ai:
  providers:
    openrouter:                 # 供应商名称（自定义）
      apiKeyEnv: OPENROUTER_API_KEY  # 从哪个环境变量读密钥
      models:
        - id: deepseek/deepseek-v4-flash
          name: "DeepSeek V4 Flash"
        - id: deepseek/deepseek-v4-pro
          name: "DeepSeek V4 Pro"
```

> ⚠️ 密钥通过环境变量传入，不要在 `settings.yaml` 里明文写密钥。启动前先 `export OPENROUTER_API_KEY=sk-xxx` 或在 shell 配置文件中写入。

配置好以后，可以启动 DSH 了。

## 🤖 配置模型参数说明

`agent-default-model` 下支持的字段（均由 AI 自动管理，一般无需手动修改）：

| 字段 | 说明 | 示例 |
| --- | --- | --- |
| `provider` | 模型供应商名称 | `deepseek-official`、`openrouter` |
| `model` | 模型 ID | `deepseek-v4-pro`、`deepseek/deepseek-v4-flash` |
| `reasoningEffort` | 推理深度（可选） | `xhigh`、`high`、`medium`、`low` |

## 首次启动 Web GUI

```bash
# 设置密钥环境变量
export OPENROUTER_API_KEY=sk-xxx    # macOS/Linux

# 启动
dsh web
```

第一次启动时，DSH 会自动初始化 Web Profile：

1. 终端会打印启动日志，最后显示 Web GUI 地址
2. 默认地址是 `http://127.0.0.1:3080`
3. 用浏览器打开这个地址，你会看到 DSH 的 Web 界面

> 💡 如果端口被占用，可以指定其他端口：`dsh web --port 3000`

## 验证安装成功

打开 Web GUI 后，做这三步验证：

| 步骤 | 操作 | 期望结果 |
| --- | --- | --- |
| 1 | 在输入框里打 `你好，请告诉我当前工作目录` | AI 回复包含你的当前目录路径 |
| 2 | 打 `/model` 看看可用模型列表 | 能看到你在 `settings.yaml` 里配的模型 |
| 3 | 打 `!pwd` 试一下终端 | 在消息区显示当前目录输出 |

## 其他启动方式

| 模式 | 命令 | 说明 |
| --- | --- | --- |
| Web GUI | `dsh web` | 浏览器界面（推荐日常使用） |
| 无头模式 | `dsh --profile headless "总结这个仓库"` | 跑一个任务、输出结果、退出 |
| 终端 TUI | `dsh --profile tui` | 终端里的键盘界面（需单独安装 TUI profile） |

> 💡 无头模式非常适合写脚本或 CI 流水线：「跑完就退，给我结果」。

## 排错

| 现象 | 可能原因 | 解决 |
| --- | --- | --- |
| `command not found: dsh` | npm 全局路径不在 PATH | 检查 `npm root -g`，把它的 bin 目录加进 PATH |
| 启动后模型调用失败 | 密钥未设或环境变量名不对 | 确认 `apiKeyEnv` 对应的环境变量已 export，且密钥有效 |
| Web GUI 打不开 | 端口被占用或防火墙 | 换端口 `dsh web --port 3000`，或检查 `127.0.0.1` 和 `localhost` |
| `pnpm` 相关报错 | pnpm 未安装或版本过低 | `npm install -g pnpm@latest` |

## 下一步

装好能跑了？去看 [第 3 章：Web GUI 使用](03-web-gui.md)，学会日常操作。

---

← 上一章：[DSH 是什么](01-what-is-dsh.md) ｜ 下一章：[Web GUI 使用](03-web-gui.md)