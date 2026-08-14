# 与 Hermes Memory 的关系：从"记什么"到"记忆架构选型"

## 本章解决什么

第 3 期《记忆与技能系统》回答了"**记什么**"：让 Hermes 记住少量长期偏好、约定和环境信息，并划下"不记密码、不记 Token"的隐私红线。本章把问题往前推一步——"**用什么架构记**"。Hermes 内置的 Memory 是一个够用但很小的小笔记本；当记忆要变成团队资产、要跨框架复用、要支持检索与沉淀时，就需要给 Hermes 接一个外部记忆系统。本章给出三条给本机 Hermes 接外部记忆的可操作路径：**Mnemosyne**（单机升级）、**TencentDB Agent Memory**（团队中枢）、**Hindsight**（学习型记忆），包括照抄官方文档的安装命令、已知坑和升级时机。

> 阅读前提：建议先读第 2 章《三个记忆系统横向对比》了解这三家系统本身；本章只讲"怎么把它们接到 Hermes 上"。

---

## 回顾：Hermes 内置 Memory 的边界

按第 3 期第 4 章的口径，内置 Memory 的设计边界很明确：

| 边界 | 第 3 期口径 |
| --- | --- |
| 形态 | 像助理的小笔记本，记少量长期重要信息 |
| 容量 | 官方默认有**字符上限**，只保留真正长期有用的信息 |
| 生效方式 | 会话开始时注入上下文，不是可查询、可检索的数据库 |
| 位置 | Memory 文件保存在**本机** Hermes 数据目录中 |
| 管理入口 | `hermes memory setup`（配置外部记忆提供方）、`hermes memory status`（查看状态） |
| 隐私红线 | 使用远程模型时，相关记忆会作为上下文发给模型服务商；**不记密码、API Key、Token、身份证号、客户敏感数据** |

也就是说，内置 Memory 的核心定位是"单 Agent 的个人小笔记本"，它没有：

- **容量与检索**：超过字符上限后只能靠覆盖和淘汰，不能按语义查回一条旧记忆；
- **团队隔离与共享**：没有"谁建的、谁能看"的归属概念，所有成员共享同一份本机数据；
- **资产组织**：只有偏好/事实，没有技能（Skill）、文档（Wiki）、代码图（CodeGraph）这类结构化资产；
- **跨框架复用**：记忆只存在于 Hermes 自己的存储里，Pi、Copilot、Claude Code 等其他框架读不到。

`hermes memory setup` 这个命令的存在，本身就说明官方预留了"换更大内存"的接口——下一节的三条升级路径，都是从这里接出去的。

---

## 为什么内置 Memory 不够用

把第 3 期"记什么/不记什么"的表格往下想一层，会发现**记什么**和**用什么记**是两件事：

1. **单 Agent 少量偏好 → 团队资产**。第 3 期记的是"我喜欢简洁步骤"这类个人偏好；而"发布前必须先跑检查清单"这种团队 SOP，需要能被多人、多 Agent 共享——内置小笔记本做不到归属和权限控制。
2. **字符上限 → 知识沉淀**。项目知识（排障记录、架构决策、验收标准）会持续增长，小笔记本很快写满；真正需要的是"能检索、能整合、能淘汰"的记忆架构。
3. **本机单点 → 跨框架复用**。同一个团队可能同时用 Hermes、Pi、Claude Code；记忆如果能沉淀到统一的存储，任何框架都能"直接读档"，避免每个 Agent 从零开始。

一句话：**第 3 期决定"记什么"，本期决定"用什么架构记"。** 架构选错，内容再多也检索不回来。

---

## 三条升级路径

三条路径都能把 Hermes 的内置 Memory 换成外部实现，但定位完全不同。以下命令均照抄官方文档与源码，**核验于 2026-08**；三家项目迭代都很快，落地前请技术负责人复核版本号。

### 路径 A：Mnemosyne 插件（推荐起步）

**Mnemosyne** 是 Hermes-first 的记忆层：一个 pip 包 + 一个 SQLite 文件，默认零云、数据不出机器。它有两条接入路线，官方推荐原生插件（**Memory Provider（记忆提供方）**）：

```bash
pip install "mnemosyne-memory[embeddings]"   # 先装核心 + 本地向量（约 800 MB；全功能 [all] 约 1.5 GB）
pip install mnemosyne-hermes                 # Hermes 插件（0.6.0，要求 mnemosyne-memory[embeddings]>=3.11.1）
mnemosyne-hermes install                     # 创建插件 symlink 供 Hermes 发现
hermes config set memory.provider mnemosyne  # 切换外部提供方
hermes memory setup                          # 走官方配置向导
```

验证（核验于 2026-08）：

```bash
hermes memory status                         # 应显示 Provider: mnemosyne
hermes tools list | grep mnemosyne_          # 应看到 remember/recall/stats 等工具
```

要点与坑（核验于 2026-08）：

- 默认数据库就在 **`~/.hermes/mnemosyne/data/`** 下（主库 `mnemosyne.db`，单文件、36 张表），升级不改隐私模型。
- Hermes provider 自动做：系统提示注入、回合前预取（`<memory-context>`）、回合后自动写入，日常使用基本无感；另有 MCP 路线可走（在 `config.yaml` 的 `mcp.servers` 里配 `command: mnemosyne, args: ["mcp"]`），但原生插件是推荐项。
- 管理命令：`hermes mnemosyne {stats|sleep|export|import|...}`（`sleep` 跑记忆整合；`export` 做备份）。
- ⚠️ 不要用 `hermes tools disable memory`（会连 provider 工具一起禁用）；只关外部提供方用 `hermes memory off`。
- ⚠️ `hermes memory status` 只反映本地 provider 注册，**不是连通性测试**。
- 已知坑：Mnemosyne 的 `config.yaml` 约半数配置键需要**进程重启**才生效；`mnemosyne export`/`remember` 会把 `--help` 当位置参数（真的会导出字符串），查帮助用 `mnemosyne --help`。
- **Docker 形态**：容器里 Hermes home 是 `/opt/data/` 而非 `~/.hermes/`，必须用持久 **side-venv（侧边虚拟环境）** wrapper 模式：`mnemosyne-hermes install --mode wrapper --python <venv python>`，且 venv 必须与 gateway 同 Python major/minor；**绝不要**把 profile 直接 symlink 到 `site-packages/mnemosyne_hermes`。
- 隐私：默认零外发；外发仅限可选的 sync、远程 embedding、远程 LLM；本地 GGUF 首次下载约 656 MB。

### 路径 B：TencentDB（团队记忆中枢）

**TencentDB Agent Memory** 是腾讯云开源的"团队记忆中枢"，把对话、技能、文档、代码沉淀为四类**记忆资产（Memory Asset）**，带团队级权限（Team/User/Agent 四级可见性 + **ACL（访问控制列表）**）。它没有 pip 一键安装，官方形态是 Docker 三件套（memory-core :8420 / memory-hub :8125+8424 / proxy :8096）：

```bash
git clone https://github.com/TencentCloud/TencentDB-Agent-Memory.git
cd TencentDB-Agent-Memory/deploy/global-images
cp .env.example .env
# 编辑 .env：MEMORY_LLM_BASE_URL/API_KEY/MODEL 与 PROXY_UPSTREAM_URL/API_KEY/MODEL 填真值（两组可相同）
# 例：MEMORY_LLM_BASE_URL=https://api.deepseek.com/v1、MEMORY_LLM_MODEL=deepseek-chat；PROXY_UPSTREAM_* 同理
./verify.sh          # 可选：LLM 通路预检（--skip-llm 可跳过）
./start-all.sh       # 一键起；面板 http://localhost:8125，管理员 key 在 ./.admin-key（sk-mem-...）
```

启动后给 Hermes 接入有两条路（核验于 2026-08）：

1. **原生 Memory Provider 插件**（`MemoryCore/hermes-plugin/`）：Python provider 是 thin HTTP client + process supervisor，走 **8420 Gateway**（Node.js sidecar）。放置规则：Hermes 启动时扫描 `<hermes-agent-checkout>/plugins/memory/<name>/`（bundled）与 `$HERMES_HOME/plugins/<name>/`（即 `~/.hermes/`），**目录名必须精确为 `memory_tencentdb`**（`plugin.yaml` 里 `name: memory_tencentdb`）。生命周期映射：`prefetch` → `POST /recall`；`sync_turn` → `POST /capture`（后台线程，最多 4 并发 + 熔断器 5 连败暂停 60s）。事实卡未给出插件的一键安装命令，确切步骤**待核验**官方 INSTALL 文档。
2. **Proxy 方式**：在 `~/.hermes/config.yaml` 的 model 段把 `base_url` 指向 `http://<proxy-host>:<port>/hermes/<spaceId>`，并加四个请求头：`x-team-id` / `x-agent-id` / `x-task-id` / `x-conversation-id`——与 Claude Code 走同一条注入链路。

坑（核验于 2026-08）：

- ⚠️ **`x-task-id` 当前版本必填**，否则 session bypass，记忆注入/回流不生效（本机实测复现：不带它连发 5 轮对话，proxy 一次 L0 都不写入；带上后全链路立即跑通）；`x-conversation-id` 需静态配置、开新对话要手动更换。
- ⚠️ 部分客户端 tool call 后续请求不携带 headers，会导致那几轮跳过注入。
- ⚠️ `MEMORY_CORE_GATEWAY_API_KEY` 官方 README 表格写默认 `local`，但当前脚本 core 侧默认**留空**，且**设非空会让 proxy 鉴权失败**（上游已知不兼容）。本地体验保持留空即可；要暴露到公网前，先等官方修复 proxy auth 并重新核验。
- 首会话会弹 Team→Agent→Task 三级选择表单；选择项为空说明还没在面板建 Team/Agent（业务用户要在其团队下建 Agent）。
- 版本节奏快：v2.0.1-beta.1（2026-08-13），核验期间三周三个版本。
- 数据默认落本地（SQLite + 本地文件，源码直跑在 `~/.memory-tencentdb/memory-tdai`，Docker 用 named volumes）；除 LLM API 外无必需外部服务。

### 路径 C：Hindsight（学习型记忆，无原生插件）

**Hindsight** 主打"让 Agent 学习而不只是记住"（**Retain / Recall / Reflect** 三阶段闭环）。它**没有 Hermes 原生 Memory Provider 插件**——官方集成列表里有 Hermes 的名字，但走的是通用接入模式（MCP / LLM Wrapper / HTTP，recall-before / retain-after）。这意味着不能像路径 A/B 那样 `hermes config set memory.provider`，需要自己把 Hindsight 注册成 Hermes 的 MCP server。

官方推荐的 Docker 部署（核验于 2026-08，命令抄自根 README）：

```bash
export OPENAI_API_KEY=sk-xxx

docker run -it --pull always --name hindsight --restart unless-stopped -p 8888:8888 -p 9999:9999 \
  -e HINDSIGHT_API_LLM_API_KEY=$OPENAI_API_KEY \
  -v hindsight-data:/home/hindsight/.pg0 \
  ghcr.io/vectorize-io/hindsight:latest
```

- API 在 `http://localhost:8888`，管理 UI（Control Plane）在 `http://localhost:9999`。
- **MCP server 挂在 `http://localhost:8888/mcp/<bank_id>/`**（默认无鉴权，单 bank 27 个工具）：把该端点注册进 Hermes 的 `mcp.servers`（HTTP 传输；Hermes 侧字段写法**待核验**），即可在会话里调用 retain/recall/reflect。
- 嵌入式轻量用法：`pip install hindsight-all -U`（Python ≥3.11；Intel Mac x86_64 请装 `hindsight-all-slim`，否则会静默回溯旧版）。

门槛（为什么说这条路径要慎重）：full 镜像约 **9 GB（AMD64）/ 3.7 GB（ARM64）**（slim 约 500 MB）；存储要 **PostgreSQL**（默认嵌入式 pg0 仅建议开发用，不推荐生产）；LLM API key 必需（除非 ollama/lmstudio/llamacpp 全本地）。已知坑：embedding 维度启动时锁定，**存了数据后改维度会丢数据**；Docker bind mount 需注意容器 UID 1000 的写权限；国内网络建议本地 embedding + `HF_ENDPOINT=https://hf-mirror.com`。官方博客还点破一个边界：**"模型"和"记忆"是两回事**——自托管只能把记忆留在自己基础设施上，模型要么本地要么走云 API。

---

## 三条路径对比

| 维度 | A Mnemosyne | B TencentDB | C Hindsight |
| --- | --- | --- | --- |
| 安装难度 | **低**：两条 pip + 两条 hermes 命令，零服务 | 中：Docker 三件套 + `.env` 两组 LLM 参数 | 中高：大镜像 + PostgreSQL + LLM key，无原生插件 |
| 团队共享能力 | 弱：默认单机；`sync` 只同步"共享面"（标了 `sync_surface_id` 的 global 行），session 级记忆永不 sync | **强**：Team/User/Agent 四级可见性 + ACL，资产默认私有、分享是明确动作 | 中：按 bank 隔离；云端版提供团队共享 bank，本地自托管偏个人 |
| 资产类型 | 记忆为主（**工作记忆（Working Memory）**/情景记忆、事实三元组、图谱） | **四类资产**：Chat Memory（L0–L3 蒸馏）/ Skill / Wiki / CodeGraph | 结构化事实 + 时间 + 实体图 + **心智模型（Mental Model）** |
| 隐私 | 默认零外发（SQLite 本机） | 纯本地 Docker；默认 key=local，公网必须换 | 本地 pg0 数据不出机器；模型仍需外部 LLM（除非全本地） |

## 从"记什么"到"用什么架构记"

第 3 期与本期是同一件事的两个层次（文字版递进图）：

```text
第 3 期 · 内容层：决定"记什么"
   记偏好 / 记约定 / 记环境信息 / 不记密码、Token、客户数据
        ↓
本期 · 架构层：决定"用什么架构记"
   ① 内置小笔记本（默认，够个人用）
   ② Mnemosyne    —— 单机升级：还是一个人，但记忆变多、能检索
   ③ TencentDB    —— 团队升级：多人多 Agent 共享资产，带权限
   ④ Hindsight    —— 能力升级：时间查询、实体图、心智模型（学习型）
```

先想清楚"这段记忆是给谁用的、要多长命、要不要分享"，再选架构；而不是反过来被工具带着走。

## 给团队的起步建议

1. **先用路径 A（Mnemosyne）**：两条命令、零服务、不改变隐私模型，是把本机 Hermes 从"小笔记本"升级成"可检索记忆"的最低成本方案，适合"个人偏好 + 单个 Hermes 用户"。
2. **出现"团队资产"需求时再上 B（TencentDB）**：当同事之间开始互相要 SOP、要排障记录，或者需要 Wiki/CodeGraph、需要"谁能看"的权限控制时，再接受 Docker 三件套的复杂度；它的四类资产模型正好承接第 1 章讲的团队资产概念。
3. **有"学习型"需求或已有 PostgreSQL 基础设施时考虑 C（Hindsight）**：需要时间范围查询、实体关系推理、跨会话信念巩固的场景；否则它的部署成本（大镜像 + PG + LLM key + 手动接 MCP）暂时不划算。
4. **切换注意**：`memory.provider` 是单一配置项，同一时刻一个 Hermes 实例只接一个外部提供方；切换前用 `hermes memory status` 确认现状，必要时先导出备份（Mnemosyne 用 `mnemosyne export backup.json`）。
5. **红线延续第 3 期**：密钥/Token 不进任何记忆；B、C 默认私有、分享是明确动作；接外部系统后 `hermes memory status` 只能确认 provider 注册，真正的连通性要靠一次真实对话验证。

---

## 官方参考

- Hermes Memory（第 3 期口径）：<https://hermes-agent.nousresearch.com/docs/user-guide/features/memory>
- Mnemosyne：<https://github.com/mnemosyne-oss/mnemosyne> ｜ Hermes 集成：docs/hermes-integration.md（仓库内）
- TencentDB Agent Memory：<https://github.com/TencentCloud/TencentDB-Agent-Memory> ｜ INSTALL_CN.md（仓库内）
- Hindsight：<https://github.com/vectorize-io/hindsight> ｜ 文档 <https://hindsight.vectorize.io/>

---
← 上一章：[三个记忆系统横向对比](02-memory-systems-comparison.md) ｜ 下一章：[隐私与部署权衡](04-privacy-and-deployment.md)
