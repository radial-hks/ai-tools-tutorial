# 新工具文档模板

> 新增工具分享时，复制此目录到 `docs/NN-tool-name/` 并修改内容。

## 文档结构

```
docs/NN-tool-name/
├── README.md                    ← 本期导航（复制此文件修改）
├── 01-what-is-<tool>.md         ← 工具是什么、安装启用
├── 02-basic-usage.md            ← 基础使用、快捷键
├── 03-context-and-prompts.md   ← 上下文管理、提示工程（如适用）
├── 04-advanced-features.md      ← 进阶功能
├── 05-best-practices.md         ← 最佳实践、避坑、学习路径
└── 06-demo-guide.md             ← 实机演示步骤（给分享者用）
```

## 命名规则

- 目录名：`NN-tool-name`（NN 是两位数字序号，如 `02-copilot-skills`）
- 文件名：`NN-topic.md`（两位数字序号 + 短横线 + 主题）
- 最后一篇文档用 `NN-demo-guide.md` 给分享者用

## 文档规范

- 中文撰写
- 术语首次出现时用粗体 + 括号标注英文原词，如 **智能体（Agent）**
- 快捷键同时标注 Mac 和 Windows/Linux
- 代码示例用三个反引号标注语言
- 每篇文档结尾加"上一节 / 下一节"导航链接
