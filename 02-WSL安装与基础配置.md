# WSL 安装与基础配置

> **文档编号**：02  
> **最后更新**：2026-07-30  
> **前置条件**：已阅读 [01-WSL概述.md](01-WSL概述.md)  
> **目的**：一步步完成 WSL 2 + Ubuntu 的安装与基础配置，配好 VS Code 开发环境

---

## 目录

- [1. 安装前检查](#1-安装前检查)
- [2. 安装 WSL 2](#2-安装-wsl-2)
- [3. 安装 Ubuntu 发行版](#3-安装-ubuntu-发行版)
- [4. WSL 初次启动与基础配置](#4-wsl-初次启动与基础配置)
- [5. 终端选择与美化](#5-终端选择与美化)
- [6. 包管理器与基础工具安装](#6-包管理器与基础工具安装)
- [7. 配置 VS Code Remote-WSL](#7-配置-vs-code-remote-wsl)
- [8. 验证 WSLg 图形功能](#8-验证-wslg-图形功能)
- [9. 文件存放策略](#9-文件存放策略)
- [10. 常见问题排查](#10-常见问题排查)
- [11. 安装后自检清单](#11-安装后自检清单)

---

## 1. 安装前检查

### 1.1 系统要求

| 项目 | 要求 | 查看方式 |
|------|------|----------|
| Windows 版本 | Windows 10 2004+ 或 Windows 11 | `winver`（运行窗口输入） |
| 体系架构 | x64（ARM64 也支持，但不适用于 FPGA 工具） | 设置 → 系统 → 关于 |
| 虚拟化 | BIOS 中开启虚拟化（Intel VT-x / AMD-V） | 任务管理器 → 性能 → CPU → 虚拟化：已启用 |

### 1.2 检查虚拟化是否开启

打开 **任务管理器**（Ctrl+Shift+Esc）→ 性能 → CPU → 查看右下角"虚拟化"：

```
虚拟化: 已启用  ✅  ← 继续下一步
虚拟化: 已禁用  ❌  ← 需要进 BIOS 开启
```

如果虚拟化被禁用：
1. 重启电脑，按 F2/Del/F10（按品牌不同）进入 BIOS
2. 找到 **Intel Virtualization Technology** 或 **AMD SVM Mode** → 设为 **Enabled**
3. 保存退出，重启

### 1.3 管理员终端准备

以下操作需要 **管理员权限**。右键开始菜单 → **Windows PowerShell (管理员)** 或 **终端 (管理员)**。

---

## 2. 安装 WSL 2

### 2.1 一条命令搞定（推荐）

在管理员 PowerShell 中执行：

```powershell
wsl --install
```

这条命令会自动完成：
- 启用"虚拟机平台"和"适用于 Linux 的 Windows 子系统"两个 Windows 功能
- 下载并安装最新的 WSL 2 Linux 内核
- 将 WSL 2 设为默认版本

执行后**重启电脑**。

### 2.2 如果上面的命令不行（手动安装）

某些旧版 Windows 或企业版可能需要手动执行：

```powershell
# 步骤 1：启用 Windows 功能
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# 步骤 2：重启电脑
# restart

# 步骤 3：下载并安装 WSL 2 内核更新包
# 打开浏览器访问：https://aka.ms/wsl2kernel
# 下载并安装 msi 包

# 步骤 4：将 WSL 2 设为默认版本
wsl --set-default-version 2
```

### 2.3 确认安装结果

```powershell
wsl --version
```

期望输出类似：

```
WSL 版本： 2.x.x
内核版本： 5.x.x
WSLg 版本： 1.x.x
```

---

## 3. 安装 Ubuntu 发行版

### 3.1 图形界面安装（推荐）

打开 **Microsoft Store** → 搜索 **"Ubuntu"** → 选择带 LTS 标识的版本（如 Ubuntu 24.04 LTS）→ 点击 **安装**。

> 💡 选择最新的 **LTS（长期支持版）**，不要选中间版本（如 24.10 非 LTS）。LTS 支持周期长、社区资源多。

### 3.2 命令行安装

```powershell
# 查看所有可用发行版
wsl --list --online

# 直接安装 Ubuntu
wsl --install -d Ubuntu-24.04

# 或者安装最新 Ubuntu
wsl --install -d Ubuntu
```

### 3.3 安装完成后

Store 或命令行安装完成后，**Ubuntu 图标会出现在开始菜单**。

---

## 4. WSL 初次启动与基础配置

### 4.1 首次启动

点击开始菜单中的 **Ubuntu** 图标，或运行：

```powershell
wsl
```

首次启动会提示创建用户：

```
Installing, this may take a few minutes...
Please create a default UNIX user account.
New username: yourname
New password: ********
Retype new password: ********
```

设定后即进入 WSL 终端：

```
yourname@DESKTOP:~$
```

### 4.2 更新软件包

这是装完系统后的标准操作：

```bash
# 更新软件包列表
sudo apt update

# 升级所有已安装的软件包
sudo apt upgrade -y

# 清理不再需要的包
sudo apt autoremove -y
```

### 4.3 确认 WSL 版本

```bash
# 在 WSL 中查看当前是 WSL 1 还是 WSL 2
wsl.exe -l -v
```

确保显示 `VERSION 2`：

```
  NAME            STATE           VERSION
* Ubuntu-24.04    Running         2
```

### 4.4 设置默认用户（可选）

如果 `wsl` 命令进入后不是你的用户：

```powershell
# 在 PowerShell 中设置默认用户
<发行版名> config --default-user <你的用户名>

# 例如：
Ubuntu-24.04 config --default-user harold
```

---

## 5. 终端选择与美化

### 5.1 Windows Terminal（强烈推荐）

Windows Terminal 比传统控制台体验好得多：多标签、GPU 渲染、主题、分屏。

在 Microsoft Store 搜索 **"Windows Terminal"** 安装。

安装后，它会自动识别 WSL 发行版，你可以：
- 在同一个窗口里开多个标签（PowerShell + WSL + CMD 混用）
- Ctrl+Shift+D 分屏
- Ctrl+Shift+T 新建标签

### 5.2 配置默认终端为 WSL

打开 Windows Terminal → 设置 → 启动 → 默认配置文件 → 选择 **Ubuntu-24.04**。

之后每次打开终端，直接进入 WSL。

### 5.3 可选：Oh My Zsh（Zsh shell）

Zsh 比 Bash 在补全、提示、主题上更强——但如果你习惯 Bash，可以跳过。

```bash
# 安装 Zsh
sudo apt install zsh -y

# 安装 Oh My Zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# 设为默认 shell
chsh -s $(which zsh)
```

---

## 6. 包管理器与基础工具安装

### 6.1 apt 加速（换国内源）

```bash
# 备份原有源
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak

# 编辑源列表（使用 Ubuntu 清华镜像）
sudo sed -i 's|http://archive.ubuntu.com|https://mirrors.tuna.tsinghua.edu.cn|g' /etc/apt/sources.list
sudo sed -i 's|http://security.ubuntu.com|https://mirrors.tuna.tsinghua.edu.cn|g' /etc/apt/sources.list

# 更新
sudo apt update
```

### 6.2 安装必备工具

```bash
# 开发工具全家桶
sudo apt install -y build-essential    # gcc, g++, make 等编译工具
sudo apt install -y git                # 版本控制
sudo apt install -y curl wget          # 网络工具
sudo apt install -y vim nano           # 文本编辑器
sudo apt install -y python3 python3-pip python3-venv  # Python 环境
sudo apt install -y tcl                # TCL 解释器（FPGA 脚本常用）
sudo apt install -y tree htop neofetch # 辅助工具
```

### 6.3 配置 Git

```bash
git config --global user.name "HaroldFrey"
git config --global user.email "your@email.com"

# 换行符处理：WSL 中保持 LF，Windows 中保持 CRLF
git config --global core.autocrlf input

# 查看配置
git config --list
```

---

## 7. 配置 VS Code Remote-WSL

这是本项目的核心工作方式。

### 7.1 安装 VS Code 扩展

在 Windows 的 VS Code 中安装扩展：**Remote - WSL**（发布者：Microsoft）。

或按 `Ctrl+Shift+X` → 搜索 `Remote - WSL` → 安装。

这个扩展会自动安装其他需要的 Remote 插件。

### 7.2 从 WSL 终端打开 VS Code

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

### 7.3 如何确认你在 Remote 模式

打开 VS Code 后，查看左下角：

```
┌──────────────────┐
│ >< WSL: Ubuntu-24│  ← 看到这个绿色角标，说明已连入 WSL
└──────────────────┘
```

终端（Ctrl+`）会直接打开 WSL 的 Bash。

### 7.4 项目目录选择

| 存放位置 | 路径 | 性能 | 推荐 |
|----------|------|------|------|
| WSL 内部 | `~/projects/my_project` | ⚡ 极快 | ✅ **推荐** |
| Windows `/mnt/` | `/mnt/d/Stduy/my_project` | 🐢 慢 5-10 倍 | ❌ 不推荐 |

> 开发代码放在 WSL 内（`/home/<user>/`）。仅在需要跨系统共享文件时才放在 `/mnt/` 下。

---

## 8. 验证 WSLg 图形功能

WSLg 是 WSL 2 的内置图形支持。验证它工作正常是环境搭建的重要一步。

### 8.1 快速测试：装一个小 GUI 验证

```bash
# 安装一个轻量 GUI 编辑器
sudo apt install gedit -y

# 启动它（注意末尾的 & 表示后台运行）
gedit &

# 如果 gedit 窗口出现在 Windows 桌面上 → WSLg 正常 ✅
```

### 8.2 安装 GTKWave（FPGA 波形查看器）

这是你在 WSL 中最重要的 GUI 工具之一：

```bash
# 安装
sudo apt install gtkwave -y

# 验证
gtkwave --version
# 应该显示 GTKWave 版本号

# 启动（即使没有波形文件也能看到界面）
gtkwave &
# GTKWave 主窗口出现在 Windows 桌面上
```

### 8.3 WSLg 不工作的排查

| 问题 | 可能原因 | 解决 |
|------|----------|------|
| `gedit &` 无窗口 | WSLg 未正确启动 | `wsl --shutdown` 然后重新打开 WSL |
| 窗口无法打开 | 显卡驱动问题 | 更新 Windows 显卡驱动 |
| 中文乱码 | 缺少中文字体 | `sudo apt install fonts-noto-cjk -y` |
| 应用启动报 `cannot open display` | DISPLAY 变量未设置（不应发生） | 检查 `echo $DISPLAY`，应为 `:0` |

> 💡 WSLg 自动设置 `DISPLAY` 和 `WAYLAND_DISPLAY` 环境变量，你不需要手动配置任何东西。

### 8.4 安装中文字体（推荐）

WSL 默认不含中文字体，GUI 应用可能显示方块：

```bash
# 安装 Noto 中文字体（Google 出品，覆盖简繁日韩）
sudo apt install fonts-noto-cjk -y

# 刷新字体缓存
fc-cache -fv
```

---

## 9. 文件存放策略

```
D:\Stduy\WSL_Use\              ← Windows 端：存放文档
    ├── 01-WSL概述.md
    ├── 02-WSL安装与基础配置.md
    └── ...

~/projects/                     ← WSL 端：存放代码和开发项目
    ├── fpga_testbench/
    ├── tcl_scripts/
    └── ...

~/shared/                       ← 可选：软链接到 Windows 某个目录
```

### 8.1 从 WSL 快速访问 Windows 目录

```bash
# 创建符号链接，方便快速跳转
ln -s /mnt/d/Stduy/WSL_Use ~/docs/WSL_Use

# 之后直接
cd ~/docs/WSL_Use
```

---

## 10. 常见问题排查

### 9.1 `wsl: command not found`

确保 PowerShell 以**管理员身份**运行，再执行安装命令。

### 9.2 虚拟化错误

```
Please enable the Virtual Machine Platform Windows feature
```

解决：
```powershell
# 管理员 PowerShell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all
# 重启
```

### 9.3 WSL 启动失败："未安装内核组件"

下载安装 WSL 2 内核更新包：https://aka.ms/wsl2kernel

### 9.4 `localhost` 转发不工作

WSL 2 使用动态 IP，但 localhost 转发应该是自动的。如果不工作：

```powershell
# 管理员 PowerShell 重置网络
wsl --shutdown
netsh winsock reset
# 重启电脑
```

### 9.5 磁盘空间问题

WSL 2 的虚拟磁盘（`.vhdx` 文件）只增不减。如果占用过大：

```powershell
# 查看 WSL 虚拟磁盘文件位置
# 默认在：%LOCALAPPDATA%\Packages\<发行版>\LocalState\ext4.vhdx

# 在 WSL 中先清理
sudo apt clean
sudo apt autoremove

# 关闭 WSL 后压缩磁盘
wsl --shutdown
diskpart
  select vdisk file="C:\path\to\ext4.vhdx"
  compact vdisk
```

### 9.6 网络问题（wsl --install 下载失败）

如果 `wsl --install` 在下载时卡住或失败：

```powershell
# 方案 1：使用代理
set HTTP_PROXY=http://proxy:port
set HTTPS_PROXY=http://proxy:port
wsl --install

# 方案 2：手动下载安装包
# 从 Microsoft Store 直接搜索安装 Ubuntu
```

---

## 11. 安装后自检清单

一条一条过，确保环境就绪：

| # | 检查项 | 命令 | 期望结果 |
|---|--------|------|----------|
| 1 | WSL 版本 | `wsl --version` | `2.x.x` |
| 2 | 发行版版本 | `wsl -l -v` | `Ubuntu-24.04 Running 2` |
| 3 | 进入 WSL | `wsl` | 进入 Bash，显示 `user@machine:~$` |
| 4 | 软件包更新 | `sudo apt update` | 无报错 |
| 5 | gcc 可用 | `gcc --version` | 显示版本号 |
| 6 | git 可用 | `git --version` | 显示版本号 |
| 7 | Python 可用 | `python3 --version` | 显示版本号 |
| 8 | TCL 可用 | `tclsh <<< "puts hello"` | 输出 `hello` |
| 9 | code 命令 | `code --version` | 显示版本号 |
| 10 | VS Code Remote | `code .` | VS Code 打开，左下角显示 `WSL: Ubuntu-24` |
| 11 | Windows 文件可见 | `ls /mnt/c/` | 列出 C 盘目录 |
| 12 | Windows → WSL | 资源管理器输入 `\\wsl$\` | 看到 Ubuntu 目录树 |

全部通过 → 🎉 WSL 环境就绪，可以开始开发了。

---

## 下一步

安装完成后，参考下一个文档进行面向 FPGA 开发的工具链配置：
`03-FPGA开发环境搭建.md`（待编写）。

---

*本文档是 [WSL_Use](../WSL_Use) 项目的一部分，记录在 Windows 下使用 WSL 进行 FPGA 开发的环境搭建过程。*
