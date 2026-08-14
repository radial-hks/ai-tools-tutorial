# 隐私与部署权衡

> 本章承接 [第 3 章](03-relation-to-hermes-memory.md) 的隐私话题，把三家记忆系统"数据放哪、会不会外泄、要花多少钱"摊开讲。三家项目都是快速迭代项目，**本文核验于 2026-08**（TencentDB 仓库内最新记录 2026-08-13，Hindsight HEAD 2026-08-14）；个别文档自相矛盾或无法从仓库确认处已标注 **待核验**，发布前请技术负责人按团队环境复核。

## 本章解决什么

前几章介绍了三个记忆系统"能干什么"、怎么和 Hermes 接。但真正决定"要不要给团队上记忆系统"的，往往不是功能，而是三个问题：

1. **数据放哪？** 本机、公司服务器，还是别人的云端？
2. **会不会外泄？** 记忆里的内容会不会被发给模型服务商、被同步服务器看到？
3. **要花多少钱？** 除了聊天本身的模型费，记忆系统还有哪些隐藏开销？

本章不替你选"最好的"，而是让你在把客户信息、代码库、团队经验交给任何系统之前，能看清每一条数据的流向，自己作清醒决策。**先看数据流向，再谈功能好坏。**

> ⚠️ 一句话结论先放这里：**三家默认形态都是"数据不出机器"**，但都留了"外发开关"；开了开关，数据就离开你的电脑。云形态则是从第一天起就把数据放在别人服务器上。

---

## 三种部署形态：数据流到哪里

### 形态 A：纯本地文件 / SQLite（Mnemosyne 默认）

Mnemosyne 是"零服务"形态：一个 pip 包 + 一个 SQLite 文件，没有常驻进程。

```text
对话 / 工具调用 ──► 本地 Python 进程（pip 包，无服务）
                        │
                        ▼
        ~/.hermes/mnemosyne/data/mnemosyne.db（单文件 SQLite）
                        │
                        ├─ 本地 fastembed 生成向量（模型缓存 ~/.hermes/cache/fastembed）
                        └─ 可选外发：远程 LLM 整合 / 远程 embedding / sync（见下）
```

- 好处：最轻、最好备份（拷走一个 `.db` 文件即可）、默认零外发。
- 代价：记忆只属于这一台机器；换机器要自己 export/import；没有团队共享界面。

### 形态 B：本机容器（TencentDB 三件套 / Hindsight Docker）

两种都住在你自己机器（或公司服务器）的 Docker 里，数据默认留在本机磁盘卷。

```text
TencentDB：Claude Code / Hermes / dsh ──► proxy(:8096) ──► 上游 LLM（转发对话）
                                              │
                                              ▼
                                    memory-core(:8420) ──► SQLite + 本地文件（Docker volume）
                                    memory-hub(:8125 面板 / :8424 知识服务)

Hindsight：Agent（LLM wrapper / MCP）──► hindsight-api(:8888) ──► pg0 嵌入式 PostgreSQL / 外部 PG
                                              │
                                              ▼
                                     Control Plane UI(:9999) 可视化、查询
```

- 好处：功能最全（团队权限、面板、Wiki/CodeGraph 等），数据还在你手里。
- 代价：要装 Docker、要维护；⚠️ **容器住在你的机器，不等于内容不出机器**——容器进程调 LLM API 时，对话照样发到模型服务商。

### 形态 C：云端 SaaS（Hindsight Cloud / TencentDB 云形态）

```text
Hindsight Cloud：本机 Agent ──► api.hindsight.vectorize.io（Vectorize 托管后端，团队共享 bank）
TencentDB 云形态：TDAI_DEPLOY_MODE=service ──► TCVDB + COS + Redis（云托管，多租户）
```

- 好处：跨设备、跨团队开箱即用，不用自己运维数据库。
- 代价：记忆存在别人服务器上；能不能接受，取决于记的是什么。

### 三形态对照

| 维度 | A 纯本地/SQLite | B 本机容器 | C 云端 SaaS |
| --- | --- | --- | --- |
| 代表 | Mnemosyne 默认 | TencentDB 三件套 / Hindsight Docker | Hindsight Cloud / TencentDB 云形态 |
| 数据位置 | 本机 SQLite 文件 | 本机/公司服务器磁盘卷 | 服务商托管后端 |
| 默认外发 | 零 | 仅 LLM 调用 | 一切（数据在服务商侧） |
| 团队共享 | 手动 export/import | 面板 + 四级权限 | 开箱即用（共享 bank） |
| 运维成本 | 几乎为零 | 中（Docker + 更新 + 鉴权） | 低（交给服务商） |
| 换机器 | 拷 .db / export | 备份 volume | 登录即用 |

---

## 逐家隐私模型

### Mnemosyne：默认零外发，但有几个"外发开关"

Mnemosyne 的默认承诺是**零外发（Zero-cloud）**：无遥测、无分析、无云依赖，数据存本机 SQLite。但文档明确列了 4 个可联网子系统 + 1 个下载，**任何一个被打开，内容就会离开机器**：

| 开关 | 开启方式 | 外发什么 | 备注 |
| --- | --- | --- | --- |
| 远程 LLM 整合 | `MNEMOSYNE_LLM_BASE_URL` / `MNEMOSYNE_LLM_PROVIDER` | `sleep()` 整合时把工作记忆发给 LLM 做摘要 | 不设则走本地 GGUF 回退 |
| 远程 embedding | `MNEMOSYNE_EMBEDDING_API_URL`（默认回退 OpenRouter） | 写入**和查询时**的 embedding 请求 | ⚠️ 查询也会外发，不只是写入 |
| Sync | `mnemosyne sync` | 按增量协议传输共享记忆 | 可加密，见下 |
| 本地 GGUF 回退 | 默认启用 | 首次使用从 Hugging Face 下载约 **656 MB** 模型 | 是下载不是上传；`MNEMOSYNE_LLM_ENABLED=false` 关闭 |

> ⚠️ **最容易踩的坑**：如果图省事配了远程 embedding（比如 OpenRouter），**每次 recall 查询的 embedding 请求也会发到该 API**——你以为只有"写入"外发，其实"读取"也外发。要真正零外发，用本地 fastembed（embedding 模型约 130 MB，缓存在 `~/.hermes/cache/fastembed`）；整合用本地 GGUF 回退（首次约 656 MB）或干脆 `MNEMOSYNE_LLM_ENABLED=false`。

**Sync 的加密与边界**（两个事实，很多人搞错）：

- 开启 `--encrypt` 后，用 Fernet（AES-128-CBC）或 PyNaCl SecretBox（XSalsa20-Poly1305）做客户端加密，**密钥永不离机**；远程 sync 服务器只能看到**元数据**（事件 ID、时间戳、操作类型、设备 ID），记忆内容、importance、source、向量全部加密。
- `mnemosyne sync` **不复制私有库**，只复制"共享面（Shared Surface）"——一个独立的专用库，只含 `scope='global'` 的行；session 级记忆永不 sync。想镜像整个库（含 session 记忆）用 `export/import`，不是 sync。

### TencentDB：默认纯本地，云形态要显式切换

TencentDB 的默认部署是**纯本地**：SQLite + 本地文件（源码直跑在 `~/.memory-tencentdb/memory-tdai`，Docker 方式用命名卷），Embedding 默认关闭、走 BM25，**除 LLM API 外无必需外部服务**，文档把"离线部署"列为适用场景。

切到云形态不是"开个开关"，而是**换部署模式**：`TDAI_DEPLOY_MODE=service` → 数据落到 TCVDB + COS + Redis（多租户）。云和本地是两套部署，不是同一个部署的"云端同步"。

⚠️ **必须知道的两点**：

1. **两组 LLM 参数，转发即外发**。`.env` 要填两组独立的 LLM 参数：memory 组（`MEMORY_LLM_BASE_URL / API_KEY / MODEL`，供记忆内核做 embed/summarize、wiki 摄取）和 proxy 组（`PROXY_UPSTREAM_URL / API_KEY / MODEL`，供 proxy 转发你的对话到上游模型）。**即使全本地部署，proxy 也会把你的对话转发给上游 LLM 服务商**（除非用内网自建模型）。
2. **Gateway 鉴权 Key 当前保持留空**。官方 README 写 `MEMORY_CORE_GATEWAY_API_KEY` 默认 `local`，但当前脚本（v2.0.1-beta.1）core 侧默认留空，且**设非空会破坏 proxy 鉴权**（上游已知不兼容）。本地体验留空即可；**监听非回环地址前必须重新核验该字段与上游修复状态**，否则拿到端口即可当 system_admin。

权限模型值得一提：四级可见性 **private / team / restricted / agent**，新 Chat Memory 和 Skill **默认私有**——"分享是一个明确动作，不是默认泄漏"。文档未提及任何遥测/回传（是否完全无遥测 **待核验**）。

### Hindsight：自托管 vs Cloud，边界非常清楚

Hindsight 官方把边界划得很直白：

| 维度 | 本地自托管（Self-hosted） | Hindsight Cloud |
| --- | --- | --- |
| 形态 | `hindsight-all` / `hindsight-embed` / Docker | 注册 `ui.hindsight.vectorize.io`，API 在 `api.hindsight.vectorize.io` |
| 数据位置 | 全部本地（pg0 嵌入式 PostgreSQL 或自管 PG），**数据不出机器** | Vectorize 托管后端，团队共享 bank（如 `team-myproject`） |
| 鉴权 | 默认无（可配 `HINDSIGHT_API_TENANT_API_KEY`） | API key（`hsk_...`）+ bank ID，本地配置 0600 权限 |
| 适用 | 单开发者 / 隐私敏感 / 离线 | 团队共享项目知识、跨机器 |

⚠️ **自托管只解决"记忆"这一环**。官方博客点破的边界："模型"和"记忆"是两回事——自托管能把记忆留在自己基础设施上，但 LLM 调用仍需要 API key 发给模型服务商（除非 ollama / lmstudio / llamacpp 全本地）。另外：

- 本地自托管**默认无鉴权**：只在本机跑没问题；一旦局域网可访问，务必配 `HINDSIGHT_API_TENANT_API_KEY`。
- 数据隔离两种做法：**每用户一个 bank**（最强隔离）或**单 bank + tags 过滤**（可跨用户聚合）。

---

## 成本构成：除了聊天费，还有三笔隐藏账

记忆系统不会替你付模型钱，反而**多烧**三类资源：

| 成本项 | 本地方案 | 云端方案 |
| --- | --- | --- |
| **LLM API（内部记忆处理）** | 三家都要额外调 LLM：Mnemosyne `sleep()` 整合、TencentDB L0–L3 蒸馏/归档、Hindsight retain 事实抽取 + reflect。这些 token 在聊天之外**另外计费** | 同左；Hindsight Cloud 的记忆处理额度归属待核验 |
| **Embedding（向量化）** | 本地免费：fastembed / bge-small-en-v1.5（首次下载约 130 MB）；Mnemosyne 本地 GGUF 首次约 656 MB | 远程按量计费；且 Mnemosyne 查询时也外发 embedding，**用量约翻倍** |
| **磁盘 / 内存** | Mnemosyne：单 SQLite 文件（MB 级），内存按 profile 约 50 MB（core）~ 1.5 GB（all）；TencentDB：三容器（memory-core 镜像约 920 MB）；Hindsight：full 镜像约 **9 GB（AMD64）** / 3.7 GB（ARM64）、slim 约 500 MB，API 建议 2 GB 内存 | 几乎不占本机资源 |

> 💡 给 WSL 同事的提示：Docker 镜像（尤其 Hindsight full 约 9 GB）很占 WSL 虚拟磁盘，装前先 `df -h` 看一眼；数据卷放 Linux 原生目录（呼应 [WSL 指南](../03-hermes/appendix-wsl-guide.md) 的建议）。

---

## 隐私红线（与第 3 期口径一致）

第 3 期 [记忆与技能系统](../03-hermes/04-memory-and-skills.md) 定过一条红线，本期**原样沿用**：记忆会在注入上下文时发给模型服务商，云形态更会离开本机，所以——

| 内容 | 能不能进记忆 | 原因 |
| --- | --- | --- |
| 密码、API Key、Token | ❌ 绝对不行 | 会随上下文发给模型服务商；云形态直接存在别人服务器 |
| 身份证号等证件信息 | ❌ 绝对不行 | 与第 3 期红线一致 |
| 客户敏感数据（合同、财务、个人资料） | ❌ 绝对不行 | 本地加密也挡不住"远程 LLM / sync / 共享"三个出口 |
| 公司内部规范（命名规范、发布流程） | ✅ 可以 | 这正是记忆系统该记的 |
| 个人偏好、工具路径、项目约定 | ✅ 可以 | 记忆系统的核心价值 |
| **不确定能不能记的** | ⚠️ **不记** | 按第 3 期原则：**"不确定就不记"**，需要时临时告诉 Agent |

> ⚠️ **记忆不是密码库**：密钥、凭据放团队的 Secret Manager / 环境变量（与 WSL 指南"快照不带密钥"同一逻辑），永远不要为了"方便"把凭据写进任何记忆系统。

---

## 团队落地建议：先本地，后共享

结合团队实际（Windows 同事用 WSL 跑 Linux/Docker、代码仓库在 GitHub、协作走飞书），推荐"**先本地、后共享**"的三步路线：

**阶段一：个人本地试点（1–2 周）**
- 每人用自己的机器跑 Mnemosyne（最轻，零服务）或 Hermes 自带记忆，先记个人偏好和项目约定。
- 用**本地 embedding + 本地或公司批准的 LLM**，保持数据不出机器。
- 验证点：Agent 是不是真的"越用越懂你"，记了之后有没有带来实际效率提升。

**阶段二：小团队共享（试点通过后）**
- 需要共享团队知识时，再上 TencentDB 三件套（本机容器、有面板和四级权限），或 Hindsight 自托管。
- 共享内容**只放团队知识**（流程、规范、常见坑），不放个人敏感信息；从默认 private 起步，显式分享。
- 验证点：新同学加入时，能不能从共享记忆"直接读档"、少问几轮。

**阶段三：按需上云（确有跨设备/跨团队需求时）**
- 考虑 Hindsight Cloud（共享 bank）或 TencentDB 云形态——只放**公开/内部非敏感**知识。
- **客户项目、涉密内容一律保持本地**，这一步没有例外。

**上线前检查清单**（技术负责人逐项打勾）：

- [ ] 飞书文档/群里复制的敏感内容，确认没被粘进任何记忆（"不确定就不记"）
- [ ] WSL 同事的数据卷在 Linux 原生目录，磁盘空间够（Hindsight full 约 9 GB）
- [ ] TencentDB 若监听非回环地址，已重新核验 `MEMORY_CORE_GATEWAY_API_KEY`（当前版本留空可用，但暴露端口即 system_admin；需上游修复 proxy auth 后再上生产）
- [ ] 每个同事用自己的 API key，不共享账号
- [ ] 定期 `mnemosyne export` 或备份 Docker volume；`mnemosyne doctor` 体检（🤖 可让 Hermes 帮你生成备份脚本）
- [ ] 团队专属配置（默认 LLM、bank 命名、共享范围）已填到下方占位区

**团队专属配置（待技术负责人补充）**：

| 项 | 团队决定 | 备注 |
| --- | --- | --- |
| 默认 LLM 供应商 / 模型 | （待补充） | 记忆处理与聊天是否同源 |
| 共享记忆范围 | （待补充） | 哪些 scope / bank 可以共享 |
| 客户项目隔离策略 | （待补充） | 建议：一律本地 |

---

## 官方参考

- Mnemosyne：<https://github.com/mnemosyne-oss/mnemosyne> ｜ 文档：`docs/sync.md` / `docs/security.md` / `docs/architecture.md`
- TencentDB Agent Memory：<https://github.com/TencentCloud/TencentDB-Agent-Memory> ｜ 文档：`README.deployment.md` / `INSTALL_CN.md`
- Hindsight：<https://hindsight.vectorize.io/> ｜ 博客《Hermes + Hindsight Open Stack》（2026-07-17，"模型 vs 记忆"边界）
- 本期相关：[第 1 章：为什么 Agent 需要长期记忆](01-why-agent-memory.md) ｜ [第 2 章：三个记忆系统横向对比](02-memory-systems-comparison.md) ｜ [第 3 章：与 Hermes Memory 的关系](03-relation-to-hermes-memory.md)

---

← 上一章：[与 Hermes Memory 的关系](03-relation-to-hermes-memory.md) ｜ 下一章：[本机实测记录](appendix-hands-on.md)
