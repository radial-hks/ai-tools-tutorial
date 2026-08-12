# Project Instructions

## 查看其他 Pane（herdr 面板）的内容

本仓库常运行在 herdr 管理的工作区里，左右会同时开多个 Pane。需要查看"另一个 Pane 在跑什么"时，**一律通过 herdr CLI，不要凭猜测去翻会话目录**：

```bash
herdr pane list                 # 列出所有 Pane：pane_id、agent 类型、状态、cwd、agent_session 路径
herdr pane get <pane_id>        # 查看单个 Pane 的元数据（状态、终端标题、聚焦等）
herdr pane read <pane_id>       # 直接读取该 Pane 的实时终端输出（滚动缓冲）
```

要点：

- `herdr pane list` 是唯一权威入口，能一次列全所有 Pane（含 pi、omp 等不同 agent）。
- 不要默认"别的 Pane 也是 pi"：不同 Pane 可能跑不同 agent，会话文件路径也各不相同
  （pi → `~/.pi/agent/sessions/<项目>/...`；omp → `~/.omp/agent/sessions/<项目>/...`）。
  以 `pane list` 返回的 `agent_session.value` 为准，必要时再 `cat` 那个 `.jsonl` 跟踪执行内容。
- 环境变量 `HERDR_SOCKET_PATH`、`HERDR_PANE_ID`（当前 Pane）、`HERDR_TAB_ID` 可辅助定位。
- `herdr pane read` 读到的是 Pane 的屏幕滚动缓冲，是最接近"直接看到画面"的通道。

## 其他约定

- 修改文件后按需自查（如有构建/检查脚本再补充）。
- 保持回复简洁。
