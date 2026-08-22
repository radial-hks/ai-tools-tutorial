# 场景实战：从想法到交付

## 本章解决什么

用**四个完整的端到端场景**，展示 Matt Pocock 的 Skill 如何在实际工作中串联使用。读完你会知道「遇到 X 情况时，依次用哪些 Skill、每个阶段要做什么」。

> 💡 每个场景都标注了 🤖（AI 做）和 ⚠️（你做）的分工。建议先通读场景一和二，它们是最高频的使用模式。

---

## 场景一：修一个线上 Bug 💥

**背景：** 美术同事反馈——角色模型导出工具偶发性输出空文件。你不知道原因，代码是你三个月前写的。

### 全流程

```text
🤖 /ask-matt "偶发性空文件输出 Bug，不确定原因"
   └→ Agent 建议：用 diagnosing-bugs

🤖 diagnosing-bugs（Agent 自动加载）
   ├── 阶段 1：构建反馈回路
   │   └── Agent 分析：这是个间歇性 Bug，需要能稳定复现
   │   └── Agent 尝试方案 7（属性/模糊测试）：
   │       写一个脚本，用 1000 组随机输入跑导出工具
   │       跑完后发现：当输入文件超过 50MB 时，50% 概率输出空文件
   │   └── 回路建立：现在有一个能稳定复现的条件了
   │
   ├── 阶段 2：最小化
   │   └── Agent 去掉无关代码，把测试范围缩到「文件流写入」模块
   │
   ├── 阶段 3：假设与验证
   │   └── Agent 假设：writeStream 在 buffer 满时未 flush
   │   └── 用回路验证：添加 flush() 后再跑 1000 次 → 0 次失败
   │   └── 假设确认 ✅
   │
   ├── 阶段 4：修复与回归
   │   └── Agent 用 /tdd 写回归测试：
   │       红：模拟 50MB+ 输入 → 测试期望输出非空且完整
   │       绿：添加 flush() 修复
   │   └── 测试通过，回路验证通过
   │
   └── code-review（Agent 自动触发）
       ├── Standards 轴：flush 调用是否放在 finally 块？是否覆盖了异常路径？
       └── Spec 轴：是否解决了「偶发性空文件」的全部触发条件？
```

### 你在这个场景中做了什么

| 步骤 | 分工 |
| --- | --- |
| 描述 Bug 症状 | ⚠️ 你 |
| 选择诊断方向 | 🤖 Agent |
| 构建反馈回路 | 🤖 Agent 尝试，⚠️ 你确认回路是否正确 |
| 提出和验证假设 | 🤖 Agent |
| 确认修复方案 | ⚠️ 你 |
| 写回归测试和修复代码 | 🤖 Agent（TDD 驱动） |
| 最终审查 | 🤖 Agent + ⚠️ 你确认 |

> 💡 这个场景的关键是「反馈回路」。Agent 没有一上来就猜 Bug 在哪——它先花时间建立了一个能 100% 复现的条件，然后 Bug 就解决了一大半。

---

## 场景二：开发一个新功能 🆕

**背景：** 你要给项目加一个「批量重命名工具」——根据角色名和贴图类型自动生成规范文件名。你知道大致要什么，但细节没想清楚。

### 全流程

```text
🤖 /grill-with-docs
   ├── Agent 盘问：
   │   Q: 哪些文件类型需要重命名？规则是什么？
   │   A: .png, .tga, .fbx；规则：{角色}_{类型}_{序号}
   │
   │   Q: 如果两个角色同名怎么办？
   │   A: 加部门前缀：{部门}_{角色}_{类型}_{序号}
   │
   │   Q: 已有文件被重名覆盖怎么处理？
   │   A: 检测冲突，输出 conflicts.csv，不自动覆盖
   │
   │   Q: 这个流程里「角色」的定义是什么？
   │   A: 好问题……我们定义一下：角色 = 源文件所在文件夹名
   │
   │   Q: 「类型」的判断依据是什么？
   │   A: 后缀映射表，如 _D → Diffuse, _N → Normal
   │
   ├── Agent 提炼统一语言 → 写入 CONTEXT.md：
   │   - Asset: 一个需要管理的资源文件
   │   - Batch Rename: 根据 Asset 的 Character 和 Type 属性生成规范文件名
   │   - Conflict: 两个 Asset 生成相同目标文件名的情况
   │   - Mapping Table: 后缀 → Type 的对照表
   │
   └── Agent 记录 ADR：
       为什么用「文件夹名 = 角色名」而不是「读取元数据」？
       → 因为美术同事的工作流就是按角色建文件夹，
         读取元数据需要额外工具，增加使用门槛。

🤖 /to-spec
   └── Agent 根据对话内容生成 Spec，发布到 GitHub Issue #142

⚠️ 你 review Spec，确认无误

🤖 /to-tickets
   └── Agent 拆分工单：
       Ticket 1: 实现后缀→类型映射表（不依赖其他）
       Ticket 2: 实现文件名解析器（依赖 Ticket 1）
       Ticket 3: 实现冲突检测逻辑（依赖 Ticket 2）
       Ticket 4: 实现批量重命名执行器（依赖 Ticket 3）
       Ticket 5: 添加 CLI 入口和用户提示（依赖 Ticket 4）

🤖 /implement
   └── Agent 依次实现每个 Ticket：
       Ticket 1: /tdd → red: 测试后缀映射 → green: 实现映射表
       Ticket 2: /tdd → red: 测试解析 → green: 实现解析器
       ...（每个 Ticket 都走 TDD 循环）
       全部完成后 → code-review（双轴审查）
```

### 你在这个场景中做了什么

| 步骤 | 分工 |
| --- | --- |
| 提出需求 | ⚠️ 你 |
| 回答盘问、澄清术语 | ⚠️ 你 |
| 确认统一语言和 ADR | ⚠️ 你 |
| Review Spec | ⚠️ 你 |
| 拆分工单 | 🤖 Agent，⚠️ 你确认依赖关系 |
| 实现代码 | 🤖 Agent（TDD 驱动） |
| 最终验收 | ⚠️ 你 |

> 💡 注意这个流程里**你从「写代码的人」变成了「做决策的人」**。Agent 负责执行，你负责在每个决策点确认方向。

---

## 场景三：改进已有代码架构 🏗️

**背景：** 项目代码写了一年，感觉越来越难改。你不知道从哪下手，但知道需要整理。

### 全流程

```text
🤖 /improve-codebase-architecture
   ├── Agent 扫描代码库...
   │
   ├── 生成 HTML 报告，列出「可加深的模块」：
   │   ┌─────────────────────────────────────────┐
   │   │ 1. FileRenamer (浅模块 ⚠️)              │
   │   │    接口：5 个公开方法，3 个配置对象       │
   │   │    功能：只做重命名                       │
   │   │    建议：合并为 1 个入口方法 + 1 个配置   │
   │   │                                          │
   │   │ 2. AssetScanner (浅模块 ⚠️)             │
   │   │    接口：扫描 + 过滤 + 排序 +...          │
   │   │    功能：只是遍历文件夹                    │
   │   │    建议：scan() 返回迭代器，过滤交给调用方 │
   │   │                                          │
   │   │ 3. ExportPipeline (可加深 ✓)             │
   │   │    接口：run() 一个方法                    │
   │   │    功能：编排整个导出流程                  │
   │   │    现状：已经是深度模块                    │
   │   └─────────────────────────────────────────┘
   │
   ⚠️ 你选择 FileRenamer 作为重构目标

🤖 Agent 用 /grill-with-docs 风格盘问：
   Q: 你希望 FileRenamer 的调用方式是什么？
   A: new FileRenamer({ pattern: "{char}_{type}_{seq}" }).rename(files)
   
   Q: 现有的 validateFileName、getFileType、resolveConflict 
      这些方法外部会用到吗？
   A: 不会，它们只是内部步骤

🤖 /to-spec → 生成重构 Spec
🤖 /to-tickets → 拆分重构工单
🤖 /implement → TDD 重构（先写测试锁住现有行为，再重构）
```

### 你在这个场景中做了什么

| 步骤 | 分工 |
| --- | --- |
| 感觉代码难改，决定整理 | ⚠️ 你 |
| 扫描和分析 | 🤖 Agent |
| 选择重构目标 | ⚠️ 你 |
| 回答重构方向问题 | ⚠️ 你 |
| 执行重构 | 🤖 Agent（TDD 保证行为不变） |
| 确认重构后功能正常 | ⚠️ 你 |

> 💡 关键技巧：重构前先用 TDD 锁住现有行为——写测试覆盖当前功能，重构后测试仍然通过，说明行为没变。这是安全重构的底线。

---

## 场景四：接手一个你不熟悉的老项目 📖

**背景：** 你被分配去维护一个同事离职前写的项目。代码没有文档，术语混乱，你甚至不确定哪些文件是核心。

### 全流程

```text
🤖 /ask-matt "接手一个老项目，代码不熟，文档缺失"
   └→ Agent 建议：先 research + domain-modeling，再 grill-with-docs

🤖 research "这个项目的技术栈是什么？核心依赖有哪些？"
   └── Agent 读取 package.json、tsconfig.json、主要源文件
   └── 输出调研报告 → docs/research/tech-stack-overview.md
       包含：技术栈清单、依赖版本、入口文件、构建流程

🤖 domain-modeling（Agent 自动触发）
   └── Agent 发现代码中有多种命名：
       "asset", "resource", "file", "item" → 可能指同一个东西
   └── Agent 提问：
       Q: 代码里 'asset' 和 'resource' 看起来是同一个概念，对吗？
       A: 读了一下代码，是的，都是指「需要处理的文件」
   └── Agent 写入 CONTEXT.md：
       Asset: 需要处理的文件。代码中 'resource' 和 'item' 是历史遗留命名，
       未来统一用 'asset'。

🤖 /grill-with-docs
   └── 你主动触发，想更深入了解项目
   Q: 这个项目的主要业务流程是什么？
   A: 我看了一遍代码：扫描文件夹 → 解析文件名 → 按规则分类 → 输出报告
   
   Q: 哪部分是团队自定义的？哪部分是第三方库？
   A: 分类规则是团队自定义的（在 rules/ 目录），其他用的标准库
   
   └── Agent 把核心概念写入 CONTEXT.md，关键决策写入 ADR

🤖 /improve-codebase-architecture
   └── 了解项目的模块结构，识别可能的改进点
   └── 不立即重构，只记录为「未来改进清单」
```

### 你在这个场景中做了什么

| 步骤 | 分工 |
| --- | --- |
| 接收任务 | ⚠️ 你 |
| 了解技术栈 | 🤖 Agent |
| 统一术语 | 🤖 Agent 提问 + ⚠️ 你确认 |
| 建立项目心智模型 | 🤖 Agent + ⚠️ 你 |
| 记录未来改进点 | 🤖 Agent 扫描 + ⚠️ 你决定 |

> 💡 接手老项目时，**最大的风险不是代码复杂，而是你理解的和代码实际做的不一致**。`domain-modeling` + `grill-with-docs` 帮你快速建立准确的心智模型。

---

## 四个场景的 Skill 调用速查

| 场景 | 入口 Skill | 核心 Skill | 收尾 Skill |
| --- | --- | --- | --- |
| 修 Bug | `diagnosing-bugs` | `diagnosing-bugs`（完整四阶段） | `tdd`（回归测试）+ `code-review` |
| 做功能 | `/grill-with-docs` | `/tdd` + `/implement` | `code-review` |
| 改架构 | `/improve-codebase-architecture` | `codebase-design` + `/tdd` | `code-review` |
| 接手老项目 | `/ask-matt` → `research` | `domain-modeling` + `/grill-with-docs` | 记录改进清单 |

## 通用工作流模板

不管具体场景是什么，Matt Pocock 的 Skill 遵循一个统一模式：

```text
1. 对齐需求
   ├── /ask-matt（不知道该用哪个 Skill 时）
   ├── /grill-with-docs（需要写代码时）
   └── /grill-me（不需要写代码时）

2. 固化需求
   ├── /to-spec（把对话变成 Spec）
   └── /to-tickets（把 Spec 拆成工单）
       或 /wayfinder（任务太大时先铺路）

3. 实现
   ├── /implement（完整流程）
   │   └── 内部调用：/tdd → /tdd → ... → code-review
   └── 或手动 /tdd（单步控制）

4. 交付
   └── code-review（审查通过后合并）
```

> 💡 不需要每次都走完四步。小改动可能只走 TDD 就够了。关键是你知道每步对应哪个 Skill，需要的时候能找到。

---

← 上一章：[Skills 仓库深度解析](02-skills-engineering.md) ｜ 下一章：[AI Coding Dictionary 与配套工具](04-dictionary-and-other-tools.md)