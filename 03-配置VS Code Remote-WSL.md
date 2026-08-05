# 配置 VS Code Remote-WSL

> 本文档专门介绍 VS Code Remote-WSL 插件的作用与使用。
> 基础环境（WSL 安装、换源、更新、必备工具）见 `02-WSL安装与基础配置.md`。

---

## 目录

1. [Remote-WSL 是什么？](#1-remote-wsl-是什么先理解再配置)
2. [安装 VS Code 扩展](#2-安装-vs-code-扩展)
3. [从 WSL 终端打开 VS Code](#3-从-wsl-终端打开-vs-code)
4. [如何确认你在 Remote 模式](#4-如何确认你在-remote-模式)
5. [项目目录选择](#5-项目目录选择)

---

## 1. Remote-WSL 是什么？

**一句话**：让你的 **VS Code 界面在 Windows 上，但代码、终端、命令全跑在 WSL 里**。

**为什么需要它**：你的代码在 WSL 里（`~/projects`）。没有它的话：
- VS Code 打开 WSL 文件只能走 `\\wsl$` 慢通道
- 终端是 Windows 的 PowerShell，**跑不了** `make`、`gcc`、`tclsh`、Vivado 脚本——它们只存在于 Linux 里

**工作方式**（前端/后端分离）：

```
Windows 桌面                        WSL 内部
┌──────────────┐   连接   ┌────────────────────┐
│ VS Code 界面  │ ──────→ │ VS Code Server(后端) │
│ 编辑/查看代码  │  Remote │  真正执行命令的引擎   │
│ 插件/主题     │         │  make/gcc/tclsh     │
└──────────────┘         └────────────────────┘
```

- **前端**（Windows 的 VS Code）：你看到的界面、编辑器、插件
- **后端**（WSL 里自动装的 VS Code Server）：实际读写文件、跑命令
- 连接后，VS Code 左下角出现绿色 `>< WSL: Ubuntu-24` 角标

> 💡 类比：就像手机遥控智能电视——界面在手机上，真正播放的是电视。或"远程桌面进 Linux，只不过窗口是 VS Code 而不是黑终端"。

**和直接打开 `\\wsl$` 文件的区别**：

| | 直接打开 `\\wsl$` 的文件 | Remote-WSL |
|--|------------------------|-------------|
| 看/改代码 | ✅ 可以 | ✅ 可以 |
| 终端能跑 Linux 命令 | ❌ 还是 PowerShell | ✅ 直接是 Ubuntu bash |
| 调试（断点） | ❌ 不支持 | ✅ 支持 |
| 性能 | 慢（走 9P 协议） | 快（后端在 WSL 内） |

## 2. 安装 VS Code 扩展

在 Windows 的 VS Code 中安装扩展：**WSL**（发布者：Microsoft，4000 万+ 下载量）。

或按 `Ctrl+Shift+X` → 搜索 `WSL` → 安装。

> 💡 **注意**：这个插件**旧名叫 "Remote - WSL"**，微软后来改名为 **"WSL"**——是同一个东西。搜索时可能看到旧名或新名，认准发布者是 **Microsoft** 即可。

这个扩展会自动安装其他需要的 Remote 插件。

> ⚠️ **插件在 WSL 侧安装**：连接 WSL 后，之前装过的插件分两类——
> - **UI 类**（主题、图标、中文界面）→ 自动通用，无需操作 ✅
> - **工具类**（Python、Verilog 语法、GitLens 等）→ 需在 WSL 侧**单独安装**：扩展市场里会显示"在 WSL 中安装"（Install in WSL）按钮，点一下即可。因为 WSL 是独立环境，工具类插件要装进 WSL 里才能配合 Linux 工具链工作。
> - Claude Code 的 skill → 不受影响，用户级配置（`~/.claude/skills/`），任何窗口都能用。

## 3. 从 WSL 终端打开 VS Code

```bash
# 在 WSL 中进入你的项目目录
cd ~/projects/my_fpga_project

# 用 VS Code 打开当前目录
code .
```

首次运行 `code .` 时，VS Code 会自动：
1. 安装 WSL 里的 VS Code Server
2. 连接到 WSL
3. 打开当前目录作为工作区

### 日常启动的两种方式

| 方式 | 操作 | 特点 |
|------|------|------|
| **A：从终端进（推荐）** | 打开 Windows Terminal → `wsl` → `cd ~/project` → `code .` | 直接打开指定目录，一步到位 |
| **B：从桌面图标进** | 点 VS Code 图标 → 左下角绿色角标 `><` → **Connect to WSL** → 选 Ubuntu | 先连上 WSL，再 File → Open Folder 选目录 |

> ⚠️ 直接双击桌面 VS Code 图标，默认打开的是**本地 Windows 模式**（不连 WSL）——看左下角有没有绿色角标判断。日常建议用**方式 A**。

## 4. 如何确认你在 Remote 模式

打开 VS Code 后，查看左下角：

```
┌──────────────────┐
│ >< WSL: Ubuntu-24│  ← 看到这个绿色角标，说明已连入 WSL
└──────────────────┘
```

终端（Ctrl+`）会直接打开 WSL 的 Bash。

## 5. 项目目录选择

| 存放位置 | 路径 | 性能 | 推荐 |
|----------|------|------|------|
| WSL 内部 | `~/projects/my_project` | ⚡ 极快 | ✅ **推荐** |
| Windows `/mnt/` | `/mnt/d/Stduy/my_project` | 🐢 慢 5-10 倍 | ❌ 不推荐 |

> 开发代码放在 WSL 内（`/home/<user>/`）。仅在需要跨系统共享文件时才放在 `/mnt/` 下。
