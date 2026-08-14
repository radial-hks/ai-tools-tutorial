# 三个记忆系统横向对比

## 本章解决什么

上一章讲了 Agent 为什么需要长期记忆，这一章直接回答更现实的问题：**我们团队该选哪一套？**

本期候选的三家开源系统——**TencentDB Agent Memory**（腾讯云开源的 Agent 团队记忆系统）、**Mnemosyne**（希腊神话中的记忆女神）、**Hindsight**（"事后之明"）——都能让 Agent"记住东西"，但定位完全不同：一家是团队记忆中枢，一家是本地轻量记忆层，一家是"会学习"的记忆引擎。选错了不是不能用，而是要么太重、要么缺权限、要么接不上我们正在用的 **Hermes** 和 **Pi**。

本章不需要你懂代码：先看[总对比表](#总对比表)找到你关心的维度，再按[决策表](#决策表按场景选)对号入座。涉及版本和分数的内容都标注了核验时间，拿不准的地方写"**待核验**"，别当定论。

> 三家都是快速迭代项目，本文事实核验于 **2026-08**，选型前请以官方最新文档为准。

---

## 三家一句话定位

| 系统 | 一句话 | 它解决什么问题 |
| --- | --- | --- |
| **TencentDB Agent Memory** | 腾讯云开源的"团队记忆中枢"：**让 Agent 沉淀经验，让人专注创造** | 减少使用 Agent 时的重复工作——项目背景不该换个会话再讲一遍、文档不该每个 Agent 从第一页重读；把对话、技能、文档、代码统一沉淀为可复用的记忆资产，让下一个 Agent"直接读档" |
| **Mnemosyne** | zero-cloud 记忆层：一个 pip 包 + 一个 SQLite 文件，任何 Agent 都能用 | 不想为记忆起任何服务：纯本地、默认零外发，给 Hermes / Pi / Claude Code 等加记忆只需"一行配置" |
| **Hindsight** | 让 Agent **"学习而不只是记住"**（learn, not just remember） | 普通记忆系统只会召回对话历史；它把记忆变成结构化事实 + 时间线 + 实体图，还能在已有记忆上推理出**新的结论** |

---

## 总对比表

| 维度 | TencentDB Agent Memory | Mnemosyne | Hindsight |
| --- | --- | --- | --- |
| **定位** | 团队记忆中枢（Memory Hub），四类资产统一管理 | 通用、Hermes-first 的本地记忆层 | 学习型记忆引擎（Retain / Recall / Reflect） |
| **记忆组织方式** | 四类资产 + L0–L3 蒸馏层级，按需注入 | **BEAM** 双层记忆 + 混合评分（50% 向量 + 30% 全文 + 20% 重要度） | 世界事实 / 经验 / 实体摘要 / 演化信念四张逻辑网络 |
| **存储后端** | SQLite + 本地文件（Docker 卷）；云形态用 TCVDB + COS + Redis | 单文件 SQLite（36 张表），向量与全文检索全在进程内 | PostgreSQL 14+（pgvector 向量 + GIN 全文 + 递归 CTE 图查询）；开发用内嵌 pg0 |
| **部署形态** | Docker 三件套（官方推荐）；也支持源码直跑 / K8s / 云服务 | 纯 Python 库，零服务零守护进程 | Docker（full / slim 镜像）或 Python 嵌入式 daemon |
| **Agent 集成** | Claude Code / Codex / Hermes / OpenClaw / CodeBuddy / WorkBuddy / dsh / 通用 OpenAI 兼容 | Hermes / Pi / Claude Code / Codex / Cursor / OpenClaw / 任意 MCP 客户端 | 约 50 个官方集成（Claude Code / Codex / Cursor / Hermes / OpenClaw / LangGraph…）+ MCP |
| **默认隐私** | 纯本地；四级可见性，新资产默认私有；是否含遥测**待核验** | 默认零外发；可选 sync / 远程 embedding / 远程 LLM 才会联网 | 本地嵌入式数据不出机器；另有托管云（团队共享 bank）；本地默认无鉴权 |
| **基准与论文** | PersonaMem 48% → 76%（+59%） | BEAM 100K 65.2%（v3.0.0）；LongMemEval 98.9%（口径不同） | LongMemEval 91.4% / LoCoMo 89.61%（arXiv 论文）；BEAM 10M 64.1% 排 #1 |
| **License** | MIT | MIT | 根 LICENSE 为 MIT（个别子包标 Apache 2.0，**待核验**） |
| **编程语言 / 依赖** | Node.js ≥ 22.16 + Docker；附 TS / Python SDK | 纯 Python ≥ 3.10，核心依赖仅 PyYAML；可选 extras | Python ≥ 3.11 + PostgreSQL；Rust CLI；Node ≥ 22（客户端 / 面板） |
| **上手难度** | 中：配 .env 两组 LLM 参数 → 一键起三件套 → 面板建 Team/Agent | 低：pip 一条命令 + 两条 CLI 命令 | 中：Docker 一条命令能跑，但全量镜像大、模型首次下载多 |

> ⚠️ **基准口径提醒**：三家用的数据集、指标、判分模型（judge）都不一样，**横向分数仅供参考，不能拿来做严格排序**。详见[基准与成熟度](#基准与成熟度)。

---

## 分维度展开

### 架构与存储

#### TencentDB：三件套 + 四类资产

| 组件 | 端口 | 职责 |
| --- | --- | --- |
| **memory-core**（记忆内核） | 8420 | L0–L3 记忆读写、鉴权、Skill / RAG 数据面；SQLite 存储；/v2、/v3 API |
| **memory-hub**（合并镜像） | 8125 面板 + 8424 知识服务 | 团队记忆管理面板（Team Memory Control）+ Wiki / CodeGraph 引擎，一个容器跑两个进程 |
| **proxy**（透明代理） | 8096 | 透明转发 LLM 请求（Anthropic + OpenAI 双协议）：会话初始化、上下文注入、对话回流；自身不落记忆数据 |

记忆按**四类资产（Memory Assets）**组织：**聊天记忆（Chat Memory）**、**技能（Skill）**、**知识库（Wiki）**、**代码图谱（CodeGraph）**。Chat Memory 内部再分四级蒸馏（Distillation）：

| 层级 | 保存什么 | 用途 |
| --- | --- | --- |
| **L0 对话** | 原始对话与完整上下文 | 核对原话、时间和来源 |
| **L1 原子事实** | 事实、偏好、约束与事件 | 精确召回可执行信息 |
| **L2 场景** | 围绕项目/场景组织的知识块 | 快速恢复一个工作场景 |
| **L3 长期画像** | 长期画像、稳定模式 | 让 Agent 迅速进入用户和团队语境 |

生成与召回都分层：平时用 L2/L3 快速进入语境，需要具体事实时经 **BM25 + 向量检索 + RRF** 回到 L1/L0。谁能用哪份资产由 **Fixed Binding + ACL** 决定，可见性分 `private / team / restricted / agent` 四级，新记忆和技能**默认私有**——"分享是一个明确动作，不是默认泄漏"。

#### Mnemosyne：单文件 SQLite + BEAM 三层

全部数据落在一个 SQLite 文件里（默认 `~/.hermes/mnemosyne/data/mnemosyne.db`），向量索引（sqlite-vec）和全文索引（FTS5）都在进程内完成，**没有外部数据库、没有独立服务**。记忆组织采用 **BEAM（Bilevel Episodic-Associative Memory，双层情景-联想记忆）**：

| 层 | 作用 | 特点 |
| --- | --- | --- |
| **工作记忆（Working Memory）** | 热上下文，自动注入 prompt | 上限 1 万条，TTL 默认 168 小时淘汰 |
| **情景记忆（Episodic Memory）** | `sleep()` 整合后的长期存储 | 混合检索，可跨会话召回 |
| **草稿区（Scratchpad）** | 临时推理区 | 不可搜索、不参与整合 |

检索是线性混合评分：**50% 向量相似度 + 30% FTS5 全文 + 20% 重要度**（官方 benchmarking 文档另写 0.5/0.4/0.1，两处矛盾，**待核验**），向量可压缩到每条 48 字节。检索引擎可选三种：默认线性混合、四路融合的 **Polyphonic**、带查询扩展的 **Enhanced**（环境变量切换）；没有向量库时会自动退化到纯全文检索，**不会因此罢工**。另有**知识图谱（TripleStore）**存时序三元组，支持"as-of 某个时间点"查询。长期记忆靠 `sleep()` 整合：把过期内容按来源分组、LLM 摘要后并入情景记忆，原记录只打时间戳**不删除**，属叠加式整合。

#### Hindsight：api / embed / control-plane + PostgreSQL + Retain/Recall/Reflect

| 组件 | 说明 |
| --- | --- |
| **hindsight-api** | 核心记忆引擎（FastAPI），默认端口 8888，内置 MCP server |
| **hindsight-embed / hindsight-all** | Python 嵌入式用法：进程内 server 或自动后台 daemon，无需自己装 PostgreSQL（用内嵌 pg0） |
| **hindsight-control-plane** | Web 可视化面板（Next.js），默认端口 9999，看记忆库和实体图 |
| **hindsight-cli** | Rust 写的命令行（retain / recall / reflect），带 TUI 浏览器 |

存储上**只支持 PostgreSQL 系**（官方刻意不做存储抽象）：pgvector 管向量、tsvector + GIN 管全文、JSONB 管关系数据、**递归 CTE** 管图查询（不引入图数据库）；开发默认用内嵌的 **pg0**（PostgreSQL + pgvector 单二进制）。

记忆闭环是 **Retain / Recall / Reflect** 三阶段：**Retain**（记住）由 LLM 后台抽取事实、实体、关系和时间数据；**Recall**（检索）是语义 + 关键词 + 图谱 + 时间**四路并行检索**，经 RRF 融合后由 **交叉编码器（cross-encoder）** 重排；**Reflect**（反思）在已有记忆上推理，产出带证据引用的新洞察（Mental Models → Observations → Raw Facts 优先级）。官方对比普通 **RAG（检索增强生成）** 记忆：RAG 只做语义相似度检索，Hindsight 多了实体图多跳、时间范围过滤、实体消解和信念巩固。

### Agent 集成矩阵

| Agent | TencentDB Agent Memory | Mnemosyne | Hindsight |
| --- | --- | --- | --- |
| **Hermes** | ✓ 原生 provider 插件（memory_tencentdb）+ Proxy 两种方式 | ✓✓ 原生插件 mnemosyne-hermes（官方推荐）+ MCP | ✓ 一等集成（官方博客专文）+ MCP |
| **Pi** | ✗ 仓库内未发现适配（**待核验**；可走通用 OpenAI 兼容 Proxy 路径） | ✓✓ 官方扩展：`pi install npm:@mnemosyne-oss/pi-mnemosyne` | 未列入官方集成清单（**待核验**；可用 MCP / SDK） |
| **Claude Code** | ✓ Proxy 环境变量（`ANTHROPIC_BASE_URL` 指向 8096） | ✓ MCP（stdio）+ `/remember`、`/recall` 斜杠命令 | ✓ `npx hindsight-cc` |
| **Codex** | ✓ `config.toml` 配 model_provider（Responses API） | ✓ MCP（`.codex/mcp.json`） | ✓ 官方集成（有 OAuth 令牌共享坑） |
| **OpenClaw** | ✓ Proxy + 原生插件脚本 | ✓ 原生 provider（推荐）+ MCP SSE | ✓ 官方集成清单含 OpenClaw |
| **其他** | CodeBuddy / WorkBuddy / dsh / 任意 OpenAI 兼容客户端（需 4 个 header） | Cursor / Windsurf / OpenWebUI / Zero / 任意 MCP 客户端 / 直接 `import mnemosyne` | 约 50 个集成：LiteLLM、LangGraph、AutoGen、Aider、n8n 等 + MCP |

三家的"接入范式"各不相同，理解后选型更清楚：

| 范式 | 谁在用 | 一句话原理 |
| --- | --- | --- |
| **Proxy 透传** | TencentDB | 把 LLM 请求先发给它的代理（proxy），代理负责"注入记忆 → 转发上游 → 回流对话"，客户端几乎不用改代码，只要改 base URL |
| **MCP 标准** | Mnemosyne（也有原生插件） | 通过 **MCP（Model Context Protocol）** 给任何支持 MCP 的客户端提供 `remember / recall` 工具，一行配置接入 |
| **两行代码 Wrapper** | Hindsight | 把现有 LLM 客户端包一层（`wrap_openai` / `wrap_anthropic`），调用前自动召回、调用后自动保存 |

**本团队栈（Hermes + Pi）的友好度**：

- **Mnemosyne 是唯一对 Hermes 和 Pi 都有"官方原生"支持的一家**，接入成本最低：Hermes 一条 `pip install mnemosyne-hermes` + 一条配置命令，Pi 一条 `pi install`。
- **TencentDB** 对 Hermes 也有原生插件，但整体是"团队中枢"路线：要 Docker 三件套 + 面板 + 权限体系，Pi 没有现成适配（**待核验**，可走通用 OpenAI 兼容路径）。
- **Hindsight** 对 Hermes 有官方一等集成，Pi 未列入清单（**待核验**）；它是学习型记忆，写代码场景集成丰富，但默认依赖 PostgreSQL（开发用内嵌 pg0 可免）。

### 基准与成熟度

| 系统 | 基准成绩 | 论文 / 版本 | 注意事项 |
| --- | --- | --- | --- |
| **TencentDB** | PersonaMem：48% → **76%**（相对 +59%） | 无论文；当前 v2.0.1-beta.1（核验期 3 周 3 个版本，迭代密集） | 仅此一项公开基准，无复现脚本/数据集链接（**待核验**） |
| **Mnemosyne** | BEAM 100K：**65.2%**（v3.0.0 点测）；LongMemEval 98.9% Recall@All@5 | 无论文；当前 3.16.0 | ⚠️ BEAM 数字是 v3.0.0（2026-05）的，**当前代码未重跑**（官方列为 open task）；LongMemEval 指标口径与别家不同 |
| **Hindsight** | LongMemEval **91.4%**；LoCoMo 89.61%；BEAM 10M 档 64.1% 排 #1 | arXiv:2512.12818《Hindsight is 20/20》（2025-12）；当前 0.9.1 | 论文称开源 20B 模型把准确率从 39% 提到 83.6%、超过 full-context GPT-4o；数据由 Virginia Tech + Washington Post 独立复现，其余厂商分数为自报 |

**三条诚实提醒**：

1. **判分模型不同，分数不可直接比**：Mnemosyne 的 BEAM 100K 65.2% 用 Llama 3.3 70B 作答 + DeepSeek V4 Flash 判分；Hindsight 同档 73.4% 用 Llama-4-Maverick 判分。不同 judge 的分数横向比较没有意义。
2. **档位也不同**：Mnemosyne 的 65.2% 是 100K 档，Hindsight 的 64.1% 是 10M 档，规模差 100 倍。
3. **各家"基准口径不同，横向分数仅供参考"**——真要看能力差距，请用同一数据集 + 同一 judge 自己复跑（两家的复现命令都在各自仓库的 benchmark 目录）。

**成熟度信号**：

- **TencentDB**：2026-07-21 首发 → 08-03 v2.0.0 → 08-13 v2.0.1-beta.1，约 3 周 3 个版本；中文文档最全（README_CN / INSTALL_CN / ROADMAP_CN 全套），有 `mem:sync` / `mem:create-skill` 会话指令。
- **Mnemosyne**：3.x 起严格 SemVer，文档目录完整（architecture / benchmarking / hermes-integration 等）；已知维护问题：`config.yaml` 约半数配置键在进程启动时被环境变量读走，改了要重启（issue #482 未解决，**待核验**）。
- **Hindsight**：有论文背书 + 第三方独立复现（Virginia Tech + Washington Post）+ 商业托管产品（Hindsight Cloud），生态文档最系统。

### 上手门槛

| 系统 | 最短路径 | 资源成本 |
| --- | --- | --- |
| **TencentDB**（Docker 三件套） | `git clone` → `cp .env.example .env`（填两组 LLM 参数）→ `./start-all.sh` → 面板 `http://localhost:8125` 建 Team/Agent → 客户端指到 proxy 8096 | Docker + Node 22.16；多架构镜像公开可拉 |
| **Mnemosyne**（pip） | `pip install "mnemosyne-memory[all]"` → `mnemosyne store "用户喜欢深色模式"` → `mnemosyne recall "preferences"` | 纯 Python ≥ 3.10；embeddings 档约 800MB、all 档约 1.5GB（最低配 core 只要 ~50MB，但需远程 embedding）；本地 GGUF 首次下载约 656MB（可关） |
| **Hindsight**（Docker 或嵌入式） | `docker run -p 8888:8888 -p 9999:9999 -e HINDSIGHT_API_LLM_API_KEY=$OPENAI_API_KEY -v hindsight-data:/home/hindsight/.pg0 ghcr.io/vectorize-io/hindsight:latest`；或 `pip install hindsight-all` | Python ≥ 3.11 或 Node ≥ 22；full 镜像约 9GB（AMD64）/ 3.7GB（ARM64），slim 约 500MB；本地模型首次下载多 |

一句话总结：**TencentDB 是"起服务"，Mnemosyne 是"装个库"，Hindsight 介于两者之间（一条 Docker 命令或一个 pip 包都能跑）**。

**动手前先知道的坑**（完整踩坑记录见[本机实测附录](appendix-hands-on.md)）：

- **TencentDB**：`.env` 里 `KNOWLEDGE_PUBLIC_BASE_URL` 必须带 `/v3` 后缀，否则面板打不开；`MEMORY_CORE_GATEWAY_API_KEY` 保持留空即可（当前版本设非空会让 proxy 鉴权失败，上游已知问题）；Claude Code 不弹选择表单多半是 `PROXY_ENABLE_SESSION_INIT` 没开。
- **Mnemosyne**：Hermes 侧要用 `hermes memory off` 关外部 provider，别用 `hermes tools disable memory`（会把 provider 工具一起禁用）；自定义 embedding 模型必须显式设 `MNEMOSYNE_EMBEDDING_DIM`，否则启动即退出；`mnemosyne export` 会把 `--help` 当真参数，查帮助用 `mnemosyne --help`。
- **Hindsight**：容器内以 UID 1000 跑，bind mount 目录权限不对会报 `Permission denied`（用命名卷最省事）；Intel Mac 装 `hindsight-all` 会静默回退旧版，改用 `hindsight-all-slim`；存过记忆后再改 embedding 维度会丢数据，动手前想清楚。

---

## 决策表：按场景选

| 你的场景（口语版） | 推荐 | 一句话理由 |
| --- | --- | --- |
| 我就一个人在本机用，想让 Hermes / Pi 记住我的偏好，越简单越好 | **Mnemosyne** | 一个 pip 包一个文件，Hermes 和 Pi 都有官方原生接入，数据默认不出机器 |
| 团队要一个共享的记忆资产中枢：沉淀技能、文档、代码图谱，还要管权限 | **TencentDB Agent Memory** | 四类资产 + 团队/可见性管理 + 管理面板，开箱就是"团队版"；适合"下一个 Agent 直接读档" |
| 我要把记忆嵌进自己的程序/产品，或给一堆写代码工具加"会学习"的记忆 | **Hindsight** | 两行代码 wrapper + 约 50 个官方集成，Reflect 还能在记忆上推理出新结论 |
| 我就想给 Claude Code / Codex / Cursor 加个记忆，不想折腾 | **Mnemosyne**（MCP 一行配置）或 **Hindsight**（`npx hindsight-cc`） | 两家都有官方现成接法，选哪个取决于你要"记住"还是要"学习" |
| 公司有隐私红线，数据绝不能出内网 | 三家都支持本地；**Mnemosyne** 最省事 | 默认零外发；TencentDB 纯本地 Docker；Hindsight 本地版数据不出机器（模型另算） |
| 公司已有 PostgreSQL / 云数据库设施，想要可视化记忆面板 | **Hindsight**（或 TencentDB） | Hindsight 原生吃 PostgreSQL 且自带 Control Plane 面板；TencentDB 自带管理面板，但要走它的三件套 |

> 拿不准时先问三个问题：**几个人用？**（单人 → Mnemosyne，多人 → TencentDB）**要不要推理出新结论？**（要 → Hindsight）**怕不怕折腾？**（怕 → Mnemosyne）。

## 选型口诀

> 个人用，Mnemosyne 一条 pip 就搞定；
> 团队用，TencentDB 三件套管得清；
> 要学习，Hindsight 两行代码自动记。
>
> 选型别纠结，按场景对号入座就行。

## 官方参考

- TencentDB Agent Memory：<https://github.com/TencentCloud/TencentDB-Agent-Memory>（中文文档 README_CN.md / INSTALL_CN.md）
- Mnemosyne：<https://github.com/mnemosyne-oss/mnemosyne> ｜ PyPI：<https://pypi.org/project/mnemosyne-memory/> ｜ Pi 扩展：<https://github.com/mnemosyne-oss/pi-mnemosyne>
- Hindsight：<https://github.com/vectorize-io/hindsight> ｜ 文档：<https://hindsight.vectorize.io/> ｜ 论文：<https://arxiv.org/abs/2512.12818>

---

← 上一章：[为什么 Agent 需要长期记忆](01-why-agent-memory.md) ｜ 下一章：[与 Hermes Memory 的关系](03-relation-to-hermes-memory.md)
