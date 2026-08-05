# 配置 VS Code Remote-WSL

> 本文档从 `02-WSL安装与基础配置.md` 独立出来，集中介绍 WSL 中开发工具的安装与配置。
> 基础环境（WSL 安装、换源、更新、必备工具）见 `02-WSL安装与基础配置.md`。
> GTKWave 的安装与验证已并入 `02-WSL安装与基础配置.md` 7.1 节（WSLg 验证）。

---

## 目录

1. [配置 VS Code Remote-WSL](#1-配置-vs-code-remote-wsl)

---

## 1. 配置 VS Code Remote-WSL

这是 FPGA 开发的核心工作方式：**VS Code 在 Windows 界面，代码和命令跑在 WSL 里**。

### Remote-WSL 是什么？（先理解再配置）

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

### 1.1 安装 VS Code 扩展

在 Windows 的 VS Code 中安装扩展：**Remote - WSL**（发布者：Microsoft）。

或按 `Ctrl+Shift+X` → 搜索 `Remote - WSL` → 安装。

这个扩展会自动安装其他需要的 Remote 插件。

### 1.2 从 WSL 终端打开 VS Code

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

### 1.3 如何确认你在 Remote 模式

打开 VS Code 后，查看左下角：

```
┌──────────────────┐
│ >< WSL: Ubuntu-24│  ← 看到这个绿色角标，说明已连入 WSL
└──────────────────┘
```

终端（Ctrl+`）会直接打开 WSL 的 Bash。

### 1.4 项目目录选择

| 存放位置 | 路径 | 性能 | 推荐 |
|----------|------|------|------|
| WSL 内部 | `~/projects/my_project` | ⚡ 极快 | ✅ **推荐** |
| Windows `/mnt/` | `/mnt/d/Stduy/my_project` | 🐢 慢 5-10 倍 | ❌ 不推荐 |

> 开发代码放在 WSL 内（`/home/<user>/`）。仅在需要跨系统共享文件时才放在 `/mnt/` 下。
