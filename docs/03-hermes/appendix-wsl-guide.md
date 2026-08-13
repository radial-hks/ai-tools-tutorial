# WSL 保姆级补充指南

> 本章是 [安装 Hermes](02-installation.md) 第 2 条路线（WSL2）的延伸补充，面向用 Windows 电脑的同事。

**来源与核验:** 基于 Bilibili 视频《Windows跑AI Agent，WSL才是终极答案》（[BV1pYNm69EPm](https://www.bilibili.com/video/BV1pYNm69EPm/)，UP主 啪啪虾技术）文字稿整理，提取于 2026-08-13；其中 WSL 相关命令已对照微软官方 WSL 文档交叉核验于 2026-08。视频属第三方内容、非官方文档，个别与官方不一致处已修正或标注，发布前建议技术负责人复核。

**标记约定:** 与 [本期首页](README.md) 一致：🤖 AI 可替代、⚠️ 必须由你决定、💡 小提示。

## 本章解决什么

[02-installation.md](02-installation.md) 的 WSL 部分只讲了「装上并跑通」的最小路径。本章补充视频里那些让它「更好用」的内容：为什么用 WSL、装前检查、安装选项、日常管理、开发三件套、文件互访、网络与备份。

## 为什么推荐在 WSL 里跑 AI Agent

AI 大模型的训练语料里，命令行操作大多是 Linux 和 macOS 风格。在 Windows 原生的 PowerShell 里跑 Agent，容易遇到命令报错、执行慢、浪费 token。WSL（**Windows Subsystem for Linux，适用于 Windows 的 Linux 子系统**）让一台 Windows 电脑同时拥有两套系统的优势：

| 好处 | 大白话 |
| --- | --- |
| Agent 更稳、更省 | AI 在 Linux 环境里写命令出错率更低 |
| 与生产环境一致 | 代码最终多半部署到 Linux 服务器，在 WSL 里开发差异更小 |
| 环境隔离更安全 | Agent 在 WSL 里误操作，通常不伤 Windows 主系统，最坏删掉重装一个 WSL 即可 |

## 装前检查：开启 CPU 虚拟化

WSL2 依赖 CPU 虚拟化，绝大多数电脑默认已开启。确认方法：

1. 搜索栏打开「**任务管理器**」。
2. 切到「**性能 → CPU**」，看右下角是否显示「**虚拟化已启用**」。
3. 若未启用：重启进 BIOS——
   - **Intel CPU**：找到「英特尔 VMX 虚拟化平台」，开启；
   - **AMD CPU**：找到「SVM Mode」，开启。

> ⚠️ 不同主板品牌、不同年份的 BIOS 菜单名称可能不同；公司电脑若被 IT 锁定 BIOS，请联系技术负责人，不要自己乱改。

## 安装选项补充

[02 章](02-installation.md) 用了最基础的 `wsl --install`。视频补充了几个实用选项（均对照微软官方文档核验）：

```powershell
# 国内网络建议加 --web-download，从在线源下载而非微软商店，速度更快
wsl --install --web-download

# 列出所有可在线安装的发行版（-o 是 --list --online 的简写）
wsl --list --online

# 安装指定发行版，并用 --location 指定装到非 C 盘
wsl --install RockyLinux --location D:\WSL\RockyLinux
```

> 💡 **「发行版」是什么？** 就是不同团队基于 Linux 内核做出的不同系统版本（Ubuntu、RockyLinux 等），概念有点像安卓手机的不同定制系统。默认装的是 **Ubuntu**，也是最流行、教程最多的一个。

### 想要同一发行版的多个实例？

> ⚠️ 视频里提到用 `wsl --install Ubuntu --name ubuntu2` 装第二个实例，但微软官方文档里 `--name` 是 `wsl --mount`（挂载磁盘）的参数，**不是**安装参数，此处不采用。可靠的「克隆」做法是用导出/导入，见下文 [备份、还原与克隆](#备份还原与克隆)。

## WSL 日常管理

```powershell
# 更新 WSL 到最新版
wsl --update

# 启动默认发行版 / 退出
wsl
exit

# 启动指定发行版；设置默认发行版
wsl -d RockyLinux
wsl --set-default RockyLinux
```

> 💡 关闭 WSL 窗口后，对应的 Linux 虚拟机会自动「关机」。下次要用再启动即可，不必手动管理电源。

## 开发三件套（装 Agent 前先备好）

装 Hermes / Pi 等 Agent 前，把三个基础工具准备好（视频内容，命令均已核验）：

```bash
# 1. Git：先确认，没有就装
git -v
sudo apt update && sudo apt install -y git

# 2. Python：Ubuntu 自带 python3，装这个包让 python 命令也能用
sudo apt install -y python-is-python3

# 3. Node.js：用 NVM（Node 版本管理器）安装，方便以后切换版本
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
source ~/.bashrc
nvm install 24
node -v
npm -v
```

> ⚠️ NVM 的 `v0.40.0` 和 Node 的 `24` 是视频录制时的版本，安装前请以 NVM 官网 / Node 官网当前版本为准。
>
> 🤖 装好后，可以让 AI 读一下 Windows 宿主机上的 Git 配置，再在 WSL 里配一份相同的（用户名、邮箱等），省得手动填。

## 文件互访

| 方向 | 怎么做 |
| --- | --- |
| Windows 看 Linux 文件 | 资源管理器左下角 Linux（企鹅）图标 → 选择发行版；或在 WSL 里执行 `explorer.exe .` |
| Linux 看 Windows 文件 | C/D 盘挂在 `/mnt/c`、`/mnt/d`，例如 `cd /mnt/c/Users/你的用户名/Desktop`；`df -h` 可看所有挂载 |
| 给 AI 发图片 | 截图保存到桌面后，直接**拖进 WSL 窗口**，图片会以挂载路径形式传入，AI 能识别 |

> ⚠️ **重要：** WSL 项目文件夹尽量放 Linux 原生目录（如 `~/projects`），别放 Windows 目录（`/mnt/c/...`）。跨系统读写有额外开销、速度慢。这与 [02 章](02-installation.md) 的路径建议一致。

## 网络：localhost 端口转发与镜像模式

WSL 默认用 **NAT 网络**：Linux 在内部虚拟网络里，有自己的虚拟 IP；但 WSL 会把 Linux 里监听的端口**自动转发**到 Windows 的 localhost，所以 Windows 浏览器直接访问 `localhost:端口` 就能连到 WSL 里的服务。

想让**局域网其他设备**也访问 WSL 服务，可改用镜像网络模式。在 `C:\Users\你的用户名\.wslconfig` 里写：

```ini
[wsl2]
networkingMode=mirrored
```

保存后关闭所有 WSL 窗口，等约一分钟让 WSL 重启。镜像模式下 WSL 直接镜像宿主机网卡、共用局域网 IP；还可能需要管理员身份改防火墙放行。

> ⚠️ 改 `.wslconfig` 会重启所有 WSL 实例，先保存好手头工作；防火墙操作请让技术负责人或 AI 指导，别自己乱放行端口。

## WSL1 vs WSL2

视频所有功能基于 **WSL2**。一张表看清区别：

| 特性 | WSL1 | WSL2 |
| --- | --- | --- |
| 本质 | 翻译层，把 Linux 指令翻译成 Windows 内核指令 | 基于 Hyper-V 的**真 Linux 内核** |
| Docker | ❌ 无法运行 | ✅ 可以运行 |
| 显卡直通 | ❌ | ✅ 自带，无需配置 |
| 维护成本 | 翻译层工作量巨大 | 大大降低 |

> 💡 装好 WSL 后，用 `wsl --list --verbose` 确认发行版的 `VERSION` 是 `2`（[02 章](02-installation.md) 已讲）。

## 备份、还原与克隆

WSL 可以把整个发行版打包成文件，方便迁移到别的电脑或克隆多份：

```powershell
# 导出备份（打包成 tar 文件）
wsl --export Ubuntu ubuntu.tar

# 还原 / 克隆（导入到指定目录，换个名字就是新实例）
wsl --import Ubuntu D:\WSL\Ubuntu ubuntu.tar
wsl --import Ubuntu2 D:\WSL\Ubuntu2 ubuntu.tar
```

> 🤖 换电脑前先 `wsl --export` 打包，新电脑上 `wsl --import` 恢复，比重新装一遍快得多。

## 安全：关闭自动挂载（可选）

默认情况下，WSL 里的 AI 能看到并操作 Windows 文件（`/mnt/c` 等）。如果希望进一步隔离，在 Linux 的 `/etc/wsl.conf` 里加：

```ini
[automount]
enabled = false
```

保存后关闭虚拟机、等至少 8 秒再重启。重启后 `df -h` 不再显示 Windows 目录，AI 也看不到 Windows 文件了。

> ⚠️ 这是一把双刃剑：更安全，但也失去了文件互访的便利。只有对安全要求高的场景才需要。

## 显卡直通（可选）

WSL2 自带**显卡直通**，直接调用 Windows 宿主机显卡，无需配置：

```bash
nvidia-smi   # 显示显卡驱动版本、CUDA 版本、显卡型号、显存
```

> ⚠️ **不要在 WSL 里装任何显卡驱动**，直接用 Windows 宿主机的驱动即可。装错驱动反而容易把环境搞坏。

## 常见问题补充

| 问题 | 解决 |
| --- | --- |
| `wsl --install` 下载慢 | 加 `--web-download` 从在线源下载 |
| 不知道装了哪些发行版 | `wsl -l`（`--list`）或 `wsl -l -v` 看版本 |
| 分不清 C/D 盘在 WSL 里的位置 | `df -h` 查看，通常挂在 `/mnt/c`、`/mnt/d` |
| 想用 Windows 上的 VS Code 打开 WSL 项目 | 在 WSL 项目目录执行 `code .`（需先装 VS Code 的 WSL 扩展） |
| WSL 里的服务 Windows 访问不到 | 确认走 localhost 端口转发；局域网访问再考虑镜像网络模式 |

## 官方参考

- 微软 WSL 基础命令: <https://learn.microsoft.com/zh-cn/windows/wsl/basic-commands>
- 微软 WSL 高级配置（.wslconfig / wsl.conf）: <https://learn.microsoft.com/zh-cn/windows/wsl/wsl-config>
- 视频来源: <https://www.bilibili.com/video/BV1pYNm69EPm/>

---

← 上一章：[安装 Hermes](02-installation.md) ｜ 返回主页：[AI 工具团队参考手册](../../README.md)
