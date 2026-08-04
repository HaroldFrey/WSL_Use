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

### 2.1 更新 WSL 平台（推荐）

在管理员 PowerShell 中执行：

```powershell
# 方式 1：从 GitHub 官网直接下载（国内推荐，绕过 Microsoft Store）
wsl --update --web-download

# 方式 2：从 Microsoft Store 更新（方式 1 慢或失败时换这个）
wsl --update
```

这条命令会自动完成：
- 下载并安装最新的 WSL 2 平台组件（内核、WSLg、dxcore）
- 将 WSL 2 设为默认版本

**命令解释**：

| 部分 | 含义 |
|------|------|
| `wsl --update` | 把 WSL 平台组件（内核、WSLg、dxcore 等）更新到最新版 |
| `--web-download` | 强制从 GitHub 官网直接下载安装包，绕过 Microsoft Store——Store 下载慢或失败时用这个 |

> 💡 下载慢是正常现象（GitHub 从国内访问较慢），耐心等待即可。如果电脑**从未使用过 WSL**，需先启用"虚拟机平台"和"适用于 Linux 的 Windows 子系统"两个 Windows 功能（方法见 [2.2](#22-如果上面的命令不行手动安装)），启用后需**重启电脑**。

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
WSL 版本： 2.7.11.0
内核版本： 6.18.33.2-2
WSLg 版本： 1.0.73.2
```

---

## 3. 安装 Ubuntu 发行版

### 3.1 图形界面安装（推荐）

打开 **Microsoft Store** → 搜索 **"Ubuntu"** → 选择带 LTS 标识的版本（如 Ubuntu 24.04 LTS）→ 点击 **安装**。

> 💡 选择最新的 **LTS（长期支持版）**，不要选中间版本（如 24.10 非 LTS）。LTS 支持周期长、社区资源多。

> ⚠️ **实测教训**：商店里 "Ubuntu"（通用版）和 "Ubuntu 24.04 LTS" **都是 24.04 基础**（包名 `CanonicalGroupLimited.Ubuntu` / `CanonicalGroupLimited.Ubuntu24.04LTS`）。**装一个就够了**——装两个会得到两套独立的 Ubuntu 系统，占两份空间，容易混淆。

### 3.2 命令行安装

```powershell
# 查看所有可用发行版
wsl --list --online

# 直接安装 Ubuntu
wsl --install -d Ubuntu-24.04

# 或者安装最新 Ubuntu
wsl --install -d Ubuntu

# 安装到自定义位置（避免占用 C 盘空间，需新版 WSL 才支持）
# 例如装在 D 盘：会自动创建 D:\WSL\Ubuntu-24.04 目录存放虚拟磁盘 ext4.vhdx
wsl --install -d Ubuntu-24.04 --location D:\WSL\Ubuntu-24.04
```

> ⚠️ **实测教训**：命令行下载可能卡住（光标一直闪、下载速度 KB/s 级、安装包迟迟不落地）。处理顺序：① 任务管理器检查是否有残留的 `wslinstaller` 进程，有就结束它再重试；② 网络问题（`raw.githubusercontent.com` 连不上）可稍后重试；③ 仍不行就改用商店安装（[3.1](#31-图形界面安装推荐)），装完用 [3.4](#34-商店版数据位置与迁移到-d-盘) 的方法迁移到 D 盘。

### 3.3 首次启动：创建用户（实测流程）

点击开始菜单中的 **Ubuntu** 图标（或运行 `wsl`），首次启动会初始化系统（约 1-3 分钟，显示 `Installing, this may take a few minutes...`），然后提示创建用户：

```
Enter new UNIX username: harold     ← 输入用户名（小写英文，不必与 Windows 用户名一致）
New password: ********               ← 输入密码（屏幕无任何显示，正常，输完回车）
Retype new password: ********
Full Name []:                        ← 以下全部直接回车跳过
Room Number []:
Work Phone []:
Home Phone []:
Other []:
Is the information correct? [Y/n] y
```

成功后进入终端，提示符为 `harold@DESKTOP-XXX:~$`（**`$` = 普通用户**；`#` = root 管理员）。

> ⚠️ **实测教训**：如果初始化后**没有弹出创建用户提示**、直接进入 `root@...:~#`，不用卸载重装！补建用户即可：
> ```bash
> adduser harold           # 按提示设置密码，其余直接回车
> usermod -aG sudo harold  # 加入 sudo 组（才能使用 sudo 命令）
> ```
> 然后在 Windows PowerShell 里设置默认用户：
> ```powershell
> wsl -d Ubuntu -u root -e sh -c "echo '[user]' > /etc/wsl.conf && echo 'default=harold' >> /etc/wsl.conf"
> ```
> 关闭 WSL 窗口重新进入，即以 `harold` 登录。

### 3.4 商店版数据位置与迁移到 D 盘

商店版安装的 Ubuntu，**系统数据（ext4.vhdx 虚拟磁盘）固定存放在 C 盘**（实测约 1.3G，会随使用继续增长）：

```
C:\Users\<用户名>\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu_...\LocalState\ext4.vhdx
```

**迁移到 D 盘**（导出/导入法，实测可用）：

```powershell
# 1. 关闭所有 WSL 实例
wsl --shutdown

# 2. 导出为 tar 备份文件
wsl --export Ubuntu D:\WSL\ubuntu-backup.tar

# 3. 注销原发行版（自动删除 C 盘的数据）
wsl --unregister Ubuntu

# 4. 导入到 D 盘新位置
wsl --import Ubuntu D:\WSL\Ubuntu-24.04 D:\WSL\ubuntu-backup.tar --version 2

# 5. 修复默认用户（实测：导入后默认登录用户会变回 root）
wsl -d Ubuntu -u root -e sh -c "printf '[user]\ndefault=harold\n' > /etc/wsl.conf"
wsl --shutdown
```

**迁移后验证**（实测步骤）：

```bash
wsl -l -v                    # 确认 Ubuntu 为 VERSION 2
wsl -d Ubuntu                # 进入系统
whoami                       # 应为 harold（不是 root）
id                           # 应包含 sudo 权限组
ls ~/                        # 原用户文件都在
```

> 💡 实测结论：**用户账号和所有文件都会随导出完整迁移**，但 `wsl --import` 会把默认登录用户重置为 root，需要第 5 步的 `wsl.conf` 修复。迁移成功后删除 `ubuntu-backup.tar` 释放空间。

---

## 4. WSL 初次启动与基础配置

### 4.1 启动方式与工作目录

点击开始菜单中的 **Ubuntu** 图标，或在 PowerShell/终端中运行：

```powershell
wsl              # 启动默认发行版，目录跟随当前 Windows 目录
wsl ~            # 启动并直接进入家目录（推荐）
wsl -d Ubuntu    # 指定发行版
```

> 💡 **实测解释**：在 `C:\Users\HaroldFrey` 目录输入 `wsl` 启动，提示符会显示 `harold@DESKTOP-XXX:/mnt/c/Users/HaroldFrey$`——**这是启动目录，不是家目录**。WSL 会自动跟随启动时所在的 Windows 目录（映射为 `/mnt/c/...` 路径）。用 `wsl ~` 直接进家目录 `/home/harold`。

创建用户流程见 [3.3](#33-首次启动创建用户实测流程)。

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

**Windows 11 已自带 Windows Terminal**（实测版本 1.24.x），无需安装。开始菜单搜索 **"终端"** 即可打开。在开始菜单打开的"Windows PowerShell"实际上也可能已经运行在 Windows Terminal 里（Win11 默认托管方式）。

安装后，它会自动识别 WSL 发行版，你可以：
- 在同一个窗口里开多个标签（PowerShell + WSL + CMD 混用）
- Ctrl+Shift+D 分屏
- Ctrl+Shift+T 新建标签

### 5.2 配置默认终端为 WSL（实测）

打开 Windows Terminal → `Ctrl + ,` 设置 → 左侧"启动" → "默认配置文件" → 选择 **Ubuntu**。

之后每次打开终端，直接进入 WSL。

> ⚠️ **实测教训**：
> - **务必从"终端"打开**——开始菜单的"Windows PowerShell"快捷方式会绕过此设置（它是独立快捷方式，与默认配置文件无关）
> - 下拉里可能出现**多个 Ubuntu 项**（商店版、WSL 动态版、已卸载发行版的"幽灵项"）——选名字为 **Ubuntu** 的那个；"幽灵项"（如已卸载的 "Ubuntu 24.04.1 LTS"）选错会启动失败
> - 清理幽灵项：设置 → 配置文件 → 删除；或直接编辑 `settings.json`（`%LOCALAPPDATA%\Packages\Microsoft.WindowsTerminal_8wekyb3d8bbwe\LocalState\settings.json`）
> - 设置生效后打开终端应显示 `harold@DESKTOP-XXX:~$`（直接在家目录）

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

> ⚠️ **实测修正**：Ubuntu 24.04 的 apt 源文件**不在** `/etc/apt/sources.list`（该文件在 24.04 里只是空壳），真正的源在 **`/etc/apt/sources.list.d/ubuntu.sources`**（Deb822 新格式）。

```bash
# 备份原有源（24.04 实际文件）
sudo cp /etc/apt/sources.list.d/ubuntu.sources /etc/apt/sources.list.d/ubuntu.sources.bak

# 编辑源列表（使用 Ubuntu 清华镜像）
sudo sed -i 's|http://archive.ubuntu.com|https://mirrors.tuna.tsinghua.edu.cn|g' /etc/apt/sources.list.d/ubuntu.sources
sudo sed -i 's|http://security.ubuntu.com|https://mirrors.tuna.tsinghua.edu.cn|g' /etc/apt/sources.list.d/ubuntu.sources

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
D:\FPGA_Self_Stduy\WSL_Use\    ← Windows 端：存放文档（本项目实际路径）
    ├── 01-WSL概述.md
    ├── 02-WSL安装与基础配置.md
    └── ...

~/projects/                     ← WSL 端：存放代码和开发项目
    ├── fpga_testbench/
    ├── tcl_scripts/
    └── ...

~/shared/                       ← 可选：软链接到 Windows 某个目录
```

### 9.1 你装的东西到底在哪儿（实测）

Linux 终端里的一切操作都发生在**虚拟磁盘 ext4.vhdx** 内，本机迁移后它物理上位于 **D 盘**：

| 路径 | 物理位置 |
|------|----------|
| 非 `/mnt/` 路径（`~`、`/home/`、`/usr/`，apt/pip 安装的一切） | `D:\WSL\Ubuntu-24.04\ext4.vhdx` 内（D 盘） |
| `/mnt/d/...` | D 盘真实目录 |
| `/mnt/c/...` | C 盘真实目录 |

> 判断规则：**路径不以 `/mnt/` 开头 → 全在 D 盘虚拟磁盘里**；`/mnt/` 开头 → 直接落在对应盘符的真实目录。
>
> ⚠️ 性能提醒：开发文件放 `~`（vhdx 内，速度快）；`/mnt/` 跨系统访问慢 5-10 倍，仅共享文件使用。
>
> 📈 观察方式：Windows 资源管理器看 `D:\WSL\Ubuntu-24.04\ext4.vhdx` 的大小（随使用增长，稀疏分配，D 盘有足够空间）。

### 9.2 从 WSL 快速访问 Windows 目录

```bash
# 创建符号链接，方便快速跳转
ln -s /mnt/d/FPGA_Self_Stduy/WSL_Use ~/docs/WSL_Use

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
# 方案 1：更新平台/下载改用 GitHub 官网直连（推荐先试这个）
wsl --update --web-download

# 方案 2：使用代理
set HTTP_PROXY=http://proxy:port
set HTTPS_PROXY=http://proxy:port
wsl --install

# 方案 3：手动下载安装包
# 从 Microsoft Store 直接搜索安装 Ubuntu
```

### 9.7 `wsl --list --online` 报 `WSL/WININET_E_CANNOT_CONNECT`

`wsl --list --online` 需要访问 GitHub 上的发行版清单（`raw.githubusercontent.com/microsoft/WSL/...`），国内网络经常连不上。这是**网络问题**，不是配置错误：
- 稍后重试（网络波动时有时能通）
- 或绕过它：改用商店安装（[3.1](#31-图形界面安装推荐)），或用命令行安装卡住时参考 [3.2](#32-命令行安装) 的实测教训

### 9.8 卸载应用后数据残留（实测 1.3G 没删掉）

在商店/设置里卸载 Ubuntu 应用，**不会**自动注销 WSL 注册表、也**不会**删除虚拟磁盘数据。彻底删除必须执行：

```powershell
wsl --unregister Ubuntu
```

注销后 WSL 注册表清空、1.3G 的 vhdx 数据自动删除；最后手动删除残留空壳目录 `C:\Users\<用户名>\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu_...`。

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
