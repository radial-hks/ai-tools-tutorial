# 本机实测记录（附录）

## 本章解决什么

这一章是本期五篇正文的"证据链"：三家系统在本机的真实安装命令、真实输出和真实踩坑。所有命令都在同一台机器上跑过（核验于 2026-08-14），你可以照着复现，也可以只看[结论表](#实测结论一览)了解"哪家跑起来最顺"。

> 🤖 本章所有操作都可由 Hermes 或 Copilot 陪你完成；⚠️ 涉及你的账号、密钥和数据的步骤必须自己把关。

## 实测环境

| 项 | 值 |
| --- | --- |
| 系统 | macOS（Apple Silicon） |
| Python | 3.14.3（venv 隔离环境） |
| Node | v24.15.0 |
| Docker | OrbStack（Docker Desktop 亦可；官方脚本兼容 colima / OrbStack） |
| Hermes | v0.20.0（本机原生安装，gateway 未运行） |
| Pi | 已安装，工作区由 herdr 管理 |
| 调研目录 | `/tmp/issue5-research/`（三家浅克隆 + venv，临时目录不入库） |

---

## Mnemosyne：一条 pip 命令就能跑

### 安装

```bash
python3 -m venv /tmp/issue5-research/venv
/tmp/issue5-research/venv/bin/pip install mnemosyne-memory
```

安装结果：**3.16.0**，Python 3.14 下无兼容问题（官方 CI 只测到 3.13，我们实测 3.14 可用，但新版本发布后建议重验）。

### CLI 实测

```bash
cd /tmp/issue5-research/mnemo-demo
../venv/bin/mnemosyne store "User prefers dark mode UI for all tools"
# → Stored: db574fcffa9e7124

../venv/bin/mnemosyne store "User prefers dark mode UI for all tools"   # 重复写入
# → Stored: db574fcffa9e7124   ← 同一个 ID，去重生效

../venv/bin/mnemosyne recall "UI theme preference"
# → Results for: UI theme preference
#     ID: db574fcffa9e7124
#     Content: User prefers dark mode UI for all tools
#     Score: 0.312

../venv/bin/mnemosyne stats
# → Total memories: 1 / Working memory: 1
#   Banks: default
#   DB path: /Users/radial/.hermes/mnemosyne/data/mnemosyne.db
```

三个观察：

1. **开箱即用**：没配任何 embedding 服务，store/recall/stats 全通。
2. **天然去重**：同一条内容重复 store 返回同一个 ID。
3. **默认库位置就在 Hermes 家目录下**（`~/.hermes/mnemosyne/`）——"Hermes-first"不是口号。注意：单独跑 CLI 时它也是写这个库，如果之后给 Hermes 接了 Mnemosyne，这里的数据 Hermes 都能看到。

### 接本机 Hermes（原生插件）

```bash
source ~/.hermes/hermes-agent/venv/bin/activate
python -m pip install mnemosyne-hermes

mkdir -p ~/.hermes/plugins/mnemosyne
ln -sfn "$(python -c 'import pathlib, mnemosyne_hermes; print(pathlib.Path(mnemosyne_hermes.__file__).resolve().parent)')"/* ~/.hermes/plugins/mnemosyne/

hermes config set memory.provider mnemosyne
hermes memory status
```

`hermes memory status` 实测输出：

```text
  Built-in (MEMORY.md / USER.md):
    Memory injection:   enabled ✓
    User profile:       enabled ✓
    Memory tool:        enabled ✓
  Provider:  mnemosyne
  Plugin:    installed ✓
  Status:    available ✓
```

> ⚠️ 官方提醒（核验于 2026-08）：**不要用 `hermes tools disable memory`**——那会连 provider 工具一起禁用；只想关外部 provider 用 `hermes memory off`。内置 Memory 和外部 provider 是两套机制，切换期间保留内置 Memory 作回退。

### 接 Pi（官方扩展）

```bash
pi install npm:@mnemosyne-oss/pi-mnemosyne
# → Installed npm:@mnemosyne-oss/pi-mnemosyne
```

> ⚠️ 实测安装时 npm 报了 1 个 high severity 漏洞（npm audit 提示）。功能不影响，但团队批量安装时建议先 `npm audit` 确认。

### Mnemosyne 小结

- 三家之中**唯一一条命令跑通全流程**的一家。
- 数据自始至终只有一个 SQLite 文件，位置透明。
- Hermes 和 Pi 两条官方接入都在本机验证通过。

---

## TencentDB Agent Memory：三件套部署

### 拉起 Docker 引擎

本机 Docker 引擎由 **OrbStack** 提供，平时不常驻。启动：

```bash
open -a OrbStack && sleep 20
docker info   # 确认 Server Version 出现
```

### 部署三件套

```bash
cd /tmp/issue5-research/TencentDB-Agent-Memory/deploy/global-images
cp .env.example .env
# 编辑 .env：两组 LLM 参数（MEMORY_LLM_* 与 PROXY_UPSTREAM_*）
PULL=1 ./start-all.sh
```

镜像拉取后，三件套全部 healthy：

```text
[ok] tdai-memory-core healthy      → http://localhost:8420/
[ok] tdai-memory-hub healthy       → Panel http://localhost:8125/
[ok] tdai-proxy healthy            → http://localhost:8096/
```

### 踩坑 1：上游脚本乱码 bug（本机真实遇到）

第一次跑 `start-all.sh` 时，memory-core 起好后脚本在打印 admin key 那一步崩了：

```text
start-memory-core.sh: line 175: ADMIN_KEY_FILE�: unbound variable
```

原因：**v2.0.1-beta.1 的 `start-memory-core.sh` 里有一处中文字符串乱码**，破坏了变量引用。修复（字节级替换掉乱码序列后重跑）：

```bash
python3 - <<'EOF'
import glob, re
for f in glob.glob('*.sh'):
    b = open(f, 'rb').read()
    fixed = re.sub(rb'\$ADMIN_KEY_FILE[\x80-\xbf\xef].{0,3}', b'$ADMIN_KEY_FILE', b)
    if fixed != b:
        open(f, 'wb').write(fixed)
        print('fixed', f)
EOF
./start-all.sh   # 重跑成功
```

> 💡 给团队的启示：beta 版工具按官方脚本走不通时，先看报错行，别急着放弃。这个 bug 大概率在后续版本修复，升级前重验。

### 管理 API 实测（无 LLM 也能跑的部分）

admin `user_key` 保存在 `.admin-key`，用它验证鉴权：

```bash
KEY=$(cat ./.admin-key)
curl -sS -X POST http://localhost:8420/v3/meta/auth/verify \
  -H "x-tdai-user-key: $KEY" -H "x-tdai-service-id: default" \
  -H "Content-Type: application/json" -d "{\"user_key\":\"$KEY\"}"
# → {"code":0,...,"data":{"valid":true,...,"user_type":"system_admin"}}
```

创建 Team 和 Agent（字段名注意是 `name` + `owner_user_id`，不是 `team_name`）：

```bash
curl -sS -X POST http://localhost:8420/v3/meta/team/create \
  -H "x-tdai-user-key: $KEY" -H "x-tdai-service-id: default" \
  -H "Content-Type: application/json" \
  -d '{"name":"tutorial-test","owner_user_id":"usr-xxxx"}'
# → {"code":0,...,"data":{"team_id":"team-1q2012f7c2",...}}
```

### 踩坑 2：文档与脚本打架的 Gateway Key

官方部署 README 表格写 `MEMORY_CORE_GATEWAY_API_KEY` 默认值是 `local`；但当前脚本里 core 侧默认**留空**（关闭 Bearer 校验），且脚本注释明确警告：

```text
MEMORY_CORE_GATEWAY_API_KEY 非空 —— proxy 的 sessionInit/auth 目前会因缺 Bearer 而失败。
本地体验请把 .env 里的 MEMORY_CORE_GATEWAY_API_KEY 留空。
```

**实测结论：留空即可跑通本地三件套；设非空反而破坏 proxy 鉴权。** 官方文档与脚本不一致，以当前脚本行为为准（核验于 2026-08）。

### 踩坑 3：记忆提取链路必须真 LLM Key

三件套能起、面板能开、Team/Agent 能建，但**记忆的 embed / summarize / 提取全部依赖 `.env` 里的真实 LLM 参数**。占位 Key 只能验证部署与 API，跑不了记忆链路——这也是我们实测时"服务全绿但记忆空转"的原因。

### TencentDB 小结

- 部署层最重（三容器 + 两组 LLM 参数），但一条 `start-all.sh` 就能起，面板功能完整。
- beta 版有脚本 bug 和文档矛盾，都在上面记录并修复/绕过了。
- 记忆链路的完整验证需要真实 LLM Key（见文末[待补实测](#待补实测全链路验证)）。

---

## Hindsight：嵌入式服务

### 安装

```bash
/tmp/issue5-research/venv/bin/pip install hindsight-all
```

> ⚠️ **API 与官方 README 有漂移（本机真实踩坑）**：根 README 示例是 `client.put(agent_id=..., content=...)` / `client.search(...)`，但 PyPI 安装到的 `hindsight-all 0.8.6` + `hindsight-client 0.9.1` 里客户端方法是 `retain(bank_id=...)` / `recall(bank_id=...)`，且 `banks` 不是方法而是 `BanksAPI` 对象（要 `client.banks.list()`）。**照抄 README 会连续报错**，以 `dir(HindsightClient)` 和 `inspect.signature` 为准。

### 启动嵌入式服务

```python
from hindsight import start_server, HindsightClient

server = start_server(
    llm_provider="openai",          # OpenAI 兼容协议
    llm_api_key="sk-...",           # 占位 Key 时服务能起，LLM 步骤会失败
    llm_base_url="https://api.deepseek.com/v1",
    llm_model="deepseek-chat",
    timeout=300.0,                  # 首次启动要下载内嵌 PostgreSQL，30s 默认超时不够
)
client = HindsightClient(base_url=server.url)
```

实测输出要点：

```text
Loading weights: 105/105 ... 199/199   ← 本地 embedding 模型（bge-small）自动下载并加载
Application startup complete.          ← 内嵌 pg0（PostgreSQL+pgvector）启动成功
SERVER_URL http://127.0.0.1:65047
```

两个观察：

1. **内嵌 PostgreSQL 和本地 embedding 全自动**，不用自己装数据库、不用配模型。
2. **首次启动默认 30s 超时不够**（要下载 pg0 + 模型），`start_server(..., timeout=300)` 解决；`HindsightServer` 类反而不接受 `timeout` 参数（又一个 API 不一致点）。

### 写入（Retain）实测：卡在 LLM 上

```python
nb = client.create_bank(bank_id="demo-bank")
r = client.retain(bank_id="demo-bank", content="User prefers Python for data analysis")
```

占位 Key 下的真实报错：

```text
ServiceException: (500) Internal Server Error
{"detail":"Fact extraction failed: 1/1 chunks failed. ... AuthenticationError: 401 ..."}
```

结论明确：**Hindsight 的 Retain 是 LLM 驱动的**（提取事实/实体/时间线），没有有效 Key 连写入都过不去——这和 Mnemosyne"无 LLM 也能存"形成鲜明对比，也是第 2 章决策表里"要学习 vs 要轻量"的具体体现。

### Hindsight 小结

- 服务本身启动顺滑（内嵌数据库 + 本地模型全自动），但"用起来"必须有真 LLM Key。
- 客户端 SDK 与 README 示例漂移明显，编程同事接手时以运行时内省为准。
- 完整 Retain / Recall / Reflect 闭环见[待补实测](#待补实测全链路验证)。

---

## 实测结论一览

| 维度 | Mnemosyne | TencentDB Agent Memory | Hindsight |
| --- | --- | --- | --- |
| 无 LLM Key 能否跑通核心链路 | ✅ store/recall 全通 | ⚠️ 部署+API 通，记忆链路不通 | ⚠️ 服务+建库通，Retain 不通 |
| 安装难度 | 最低（pip 一条） | 中（Docker 三件套 + .env） | 中（pip 一条，但首次下载多） |
| 本机真实踩坑数 | 0（Python 3.14 无问题） | 3（脚本乱码 / 文档矛盾 / Key 依赖） | 2（API 漂移 / 30s 超时） |
| Hermes 原生接入 | ✅ 插件装好、status 显示 available | 有插件与 Proxy 两条路（本次未实测到记忆链路） | 无原生插件（MCP/HTTP 通用接入） |
| Pi 原生接入 | ✅ 扩展安装成功 | 无（待核验） | 未列入官方清单（待核验） |

一句话：**轻量个人用 Mnemosyne 最顺；团队中枢 TencentDB 部署可跑但 beta 味重；Hindsight 要配好 LLM 才能发挥"学习型"优势。**

---

## 待补实测：全链路验证

以下验证需要**真实的 OpenAI 兼容 LLM Key**（如 DeepSeek），技术负责人提供后可一键补测：

1. **TencentDB**：把 `.env` 里两组占位参数换成真值 → 重启 → 面板建 Team/Agent → 通过 proxy 跑一轮对话 → 面板查看 Chat Memory L0–L3 是否生成。
2. **Hindsight**：把 `llm_api_key` 换真值 → `retain` 写入 → `recall` 检索 → `reflect` 推理 → `list_memories` 核对。
3. **Mnemosyne**：`MNEMOSYNE_EMBEDDING_API_URL` 指远程 embedding 时验证"查询也外发"的隐私行为。

---

## 清理与回退

实测留下的本机痕迹及回退方式：

| 痕迹 | 回退 |
| --- | --- |
| Hermes memory.provider = mnemosyne | `hermes config set memory.provider ''` 回到内置 Memory（插件保留不影响） |
| Hermes 插件 `mnemosyne-hermes` | 保留即可；彻底移除：卸载 pip 包 + 删 `~/.hermes/plugins/mnemosyne/` |
| Pi 扩展 pi-mnemosyne | `pi uninstall npm:@mnemosyne-oss/pi-mnemosyne`（或保留） |
| TencentDB 三个容器 + 数据卷 | `cd deploy/global-images && ./stop-all.sh`（保留数据）；`./stop-all.sh --purge`（连数据一起清） |
| Hindsight 下载的模型与 pg0 | `pip uninstall hindsight-all`；模型缓存在 HF cache 目录 |
| `/tmp/issue5-research/` 调研目录 | 直接删除（不入库） |

---

← 上一章：[隐私与部署权衡](04-privacy-and-deployment.md) ｜ 返回本期首页：[README](README.md)
