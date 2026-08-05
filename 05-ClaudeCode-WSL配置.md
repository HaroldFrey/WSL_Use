# 在 WSL 中使用 Claude Code（配置指南）

> **文档编号**：05
> **前置条件**：已完成 [02-WSL安装与基础配置.md](02-WSL安装与基础配置.md)、[03-配置VS Code Remote-WSL.md](03-配置VS%20Code%20Remote-WSL.md)
> **目的**：在 WSL 里用上 Claude Code（VS Code 扩展版），与 Windows 共用同一套配置和 skills

---

## 目录

1. [为什么要单独配置](#1-为什么要单独配置)
2. [登录页一直显示的真相](#2-登录页一直显示的真相)
3. [步骤 1：链接 settings.json](#3-步骤-1链接-settingsjson)
4. [步骤 2：链接 skills 目录](#4-步骤-2链接-skills-目录)
5. [验证](#5-验证)
6. [使用方式](#6-使用方式)
7. [常见问题](#7-常见问题)
8. [在 WSL 中使用 Windows 的 Vivado（实测）](#8-在-wsl-中使用-windows-的-vivado实测)

---

## 1. 为什么要单独配置

Claude Code 的配置是**按环境独立**的：

| 环境 | 读取的配置文件 |
|------|---------------|
| Windows 窗口 | `C:\Users\<用户名>\.claude\settings.json` |
| WSL 窗口 | `/home/<用户名>/.claude/settings.json` |

WSL 是独立的 Linux 系统（数据在 D 盘虚拟磁盘里），它**不会自动读取** Windows 的配置。所以 WSL 里第一次用 Claude Code 时，一切都是"空白"状态。

**好消息**：不用在 WSL 里重复安装任何东西——
- ❌ 不需要 Node.js（VS Code 扩展自带 CLI 运行环境）
- ❌ 不需要安装 Claude Code CLI
- ❌ 不需要 cc-switch（配置共用同一份）
- ❌ 不需要新的 API Key（同一把钥匙）

**只需要两个软链接**，让 WSL 直接使用 Windows 的配置和 skills。

---

## 2. 登录页一直显示的真相

WSL 里打开 Claude Code 如果**一直停留在 "Welcome to Claude Code" 登录选择页**，原因只有一个：

> **`~/.claude/settings.json` 不存在**——Claude Code 读不到任何 API 配置（`ANTHROPIC_AUTH_TOKEN`），只能要求你登录。

Windows 不显示登录页，是因为配置里已有 DeepSeek 的 API 配置，启动即跳过登录。

**解法**：见下一步，把 Windows 的配置链接过来。

---

## 3. 步骤 1：链接 settings.json

在 **Ubuntu 终端**执行：

```bash
ln -s /mnt/c/Users/<你的用户名>/.claude/settings.json ~/.claude/settings.json
```

**效果**：
- WSL 的 Claude Code 读取这个链接时，实际读取的是 Windows 的配置文件
- 用 cc-switch 在 Windows 切换配置，WSL **自动同步**（同一份文件）
- 登录页不再出现，启动即用

> 💡 为什么推荐软链接而不是复制：一份配置两边共用，永远同步，不用维护两份。

**方案 B：直接复制（可行，但需手动同步）**

```bash
cp /mnt/c/Users/<你的用户名>/.claude/settings.json ~/.claude/settings.json
```

| 对比 | 软链接（推荐） | 直接复制 |
|------|---------------|----------|
| 配置同步 | ✅ 永远同步（同一份文件） | ❌ 改一边不影响另一边，需手动再复制 |
| cc-switch 切换 | ✅ WSL 自动跟着变 | ❌ 切换后 WSL 还是旧配置 |
| 适用场景 | 日常使用 | 想让 WSL 配置独立、或链接不便时 |

---

## 4. 步骤 2：链接 skills 目录

```bash
ln -s /mnt/c/Users/<你的用户名>/.claude/skills ~/.claude/skills
```

两边共用同一套 skills（`/ls`、`/pwd`、`/git-push` 等），永久同步，无需在 WSL 侧重复制作。

**方案 B：直接复制**——想让 WSL 侧独立维护 skills 时：

```bash
cp -r /mnt/c/Users/<你的用户名>/.claude/skills ~/.claude/
```

> 复制后是两份独立目录，改一边不影响另一边；以后 Windows 侧新增 skill 不会自动出现在 WSL。

---

## 5. 验证

```bash
ls -la ~/.claude/settings.json   # 应显示 -> /mnt/c/Users/.../settings.json
ls ~/.claude/skills/             # 应列出全部 skill 目录
```

然后在 WSL 的 VS Code（左下角有绿色 `>< WSL: Ubuntu` 角标的窗口）打开 Claude Code：

- ✅ **直接进入对话界面，无登录页** → 配置成功
- ❌ 仍显示登录页 → 检查软链接是否建立、settings.json 是否可读

---

## 6. 使用方式

| 入口 | 说明 |
|------|------|
| **VS Code 侧边栏**（聊天泡泡图标） | WSL 窗口里直接对话，选代码提问 |
| **VS Code 右上角图标** | 在 WSL 终端里运行 claude 命令（如果已装 CLI） |

> WSL 侧未装 CLI 时，用侧边栏入口即可——功能一致，支持 `/skill名` 触发自定义 skill。

---

## 7. 常见问题

### Q1：为什么不在 WSL 里安装 CLI？

不需要。VS Code 扩展自带运行环境；日常在 WSL 窗口的侧边栏使用即可。若以后想用终端 CLI，再 `npm install -g @anthropic-ai/claude-code`。

### Q2：软链接会不会影响 Windows 侧？

不会。软链接只是 WSL 侧的"快捷方式"，指向 Windows 文件——Windows 那边照常使用，两边读到的是同一份数据。

### Q3：改了 Windows 配置，WSL 需要重启吗？

不需要。Claude Code 会热加载 settings.json，下一轮对话即刻生效。

### Q4：skills 链接后，WSL 里能直接用 `/git-push` 等命令吗？

能。skill 触发逻辑只认 skills 目录内容，链接过来后与 Windows 完全一致。

---

## 8. 在 WSL 中使用 Windows 的 Vivado（实测）

### 8.1 能直接用吗？

**可以**。WSL 能直接调用 Windows 上的程序（原理见 [01 §5.2](01-WSL概述.md#52-程序互相调用)）。Vivado 本体继续在 Windows 上跑（综合/实现/烧录），WSL 负责调用它、编写和管理 TCL 脚本。

### 8.2 本机 Vivado 版本与位置（实测）

| 版本 | 启动器命令 | 安装位置 |
|------|-----------|----------|
| **2017.4** | `vivado17` | `D:\App_install_Lcoation\Vivado201704\Vivado\2017.4\bin\vivado.bat` |
| **2018.3** | `vivado18` | `D:\App_install_Lcoation\Vivado201803\Vivado\2018.3\bin\vivado.bat` |
| **2019.2** | `vivado19` / `vivado`（默认） | `D:\App_install_Lcoation\Vivado201902\Vivado\2019.2\bin\vivado.bat` |

> 本机装了 3 个版本（2017.4 / 2018.3 / 2019.2），在 WSL 里做了**启动器脚本**（`~/bin/vivado`、`~/bin/vivado17/18/19`），`~/bin` 已加入 PATH，开新终端即可直接用。

### 8.3 三种调用方式

**① 启动 Vivado GUI**（在 Ubuntu 终端）：

```bash
vivado19 &        # 或 vivado17 / vivado18 / vivado（默认 2019.2）
```

**② 批处理模式**（跑 TCL 脚本，不开界面，适合自动化）：

```bash
vivado19 -mode batch -source run.tcl
```

**③ 查版本**（快速验证通路）：

```bash
vivado17 -version   # Vivado v2017.4 (64-bit) ✅（三个版本均实测通过）
```

> 💡 启动器脚本原理：一行 `exec cmd.exe /c "D:\...\vivado.bat" "$@"`——把参数原样传给 Windows 版 Vivado。想加更多版本，照葫芦画瓢复制脚本改路径即可。

### 8.4 路径转换（关键坑）

WSL 的 `/mnt/d/...` 路径 Vivado **不认**，传参前要转成 Windows 格式：

```bash
wslpath -w /mnt/d/project/run.tcl     # 输出: D:\project\run.tcl
```

示例——把 WSL 里的 TCL 脚本交给 Vivado 执行：

```bash
TCL_WIN=$(wslpath -w /mnt/d/project/run.tcl)
cmd.exe /c "D:\...\vivado.bat" -mode batch -source "$TCL_WIN"
```

### 8.5 注意事项

| 事项 | 说明 |
|------|------|
| GUI 窗口 | 弹出的是 **Windows 原生窗口**（不是 WSLg 窗口），正常 |
| GUI 启动慢 | 2019.2 首次启动 30 秒~1 分钟才出窗口，属正常，耐心等 |
| TCL 脚本 | 在 WSL 里写/管理（能用 Linux 工具链），再传给 Windows Vivado 执行 |
| 文件访问 | 跨系统（/mnt/d）访问比原生慢，大工程建议把工程目录放 Windows 侧 |
| Vivado 版本 | 本机 3 个版本：2017.4 / 2018.3 / 2019.2（`vivado17`/`vivado18`/`vivado19`） |

### 8.6 Vivado 工程建在哪？（实测结论）

**结论：工程建在 Windows 侧（D 盘），WSL 通过 `/mnt/d` 参与管理。**

| 位置 | 可行性 | 原因 |
|------|--------|------|
| WSL 内部（`~/`） | ❌ 不行 | Vivado 拿到 UNC 路径（`\\wsl.localhost\...`）打不开工程（实测） |
| **Windows D 盘** | ✅ **推荐** | Vivado 原生速度访问；WSL 通过 `/mnt/d` 同一目录可见 |

**推荐结构**：

```
D:\FPGA_Self_Stduy\vivado_projects\
    ├── project_a\
    │   ├── project_a.xpr        ← Vivado 工程文件
    │   ├── rtl\                 ← RTL 源码（Vivado 直接引用）
    │   └── scripts\             ← TCL 脚本
    └── project_b\
```

**工作流**：VS Code（WSL 窗口）打开 `/mnt/d/FPGA_Self_Stduy/vivado_projects/<工程>/` 写 RTL 和 TCL → git 版本管理 → `vivado19 -mode batch -source run.tcl` 交给 Windows Vivado 综合/实现/烧录。

---

*本文档是 [WSL_Use](../WSL_Use) 项目的一部分，记录在 Windows 下使用 WSL 进行 FPGA 开发的环境搭建过程。*
