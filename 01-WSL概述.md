# WSL 概述

> **文档编号**：01  
> **最后更新**：2026-07-30  
> **目的**：了解 WSL 是什么，能做什么，为后续环境搭建奠定认知基础

---

## 目录

- [1. 什么是 WSL](#1-什么是-wsl)
- [2. WSL 1 vs WSL 2](#2-wsl-1-vs-wsl-2)
- [3. WSL 能做什么](#3-wsl-能做什么)
- [4. WSL 与 VS Code 配合](#4-wsl-与-vs-code-配合)
- [5. 对 FPGA 工程师的意义](#5-对-fpga-工程师的意义)
- [6. 常见 Linux 发行版选择](#6-常见-linux-发行版选择)

---

## 1. 什么是 WSL

**WSL（Windows Subsystem for Linux）** 是微软开发的技术，让你在 Windows 中**直接运行 Linux 环境**，无需安装虚拟机或双系统。

把它理解为一个"**嵌入在 Windows 里的 Linux**"——你可以打开一个终端，里面跑的就是真正的 Linux 命令行，可以执行 Linux 程序、安装 Linux 软件包、运行脚本。

### 它不是什么

| 误解 | 实际情况 |
|------|----------|
| 它是一台虚拟机 | WSL 2 虽然是轻量 VM，但不需你管理虚拟硬件、ISO 镜像 |
| 它是双系统 | 不需要分区、重启切换，两个系统**同时运行** |
| 它是 Cygwin / Git Bash | 比它们完整得多——WSL 2 运行**真实的 Linux 内核** |

---

## 2. WSL 1 vs WSL 2

微软提供了两个版本：

| 特性 | WSL 1 | WSL 2 |
|------|-------|-------|
| 架构原理 | 将 Linux 系统调用翻译为 Windows 系统调用 | 在轻量级虚拟机中运行完整 Linux 内核 |
| 文件系统性能 | 跨系统文件访问快，Linux 原生文件操作一般 | Linux 原生文件操作极快（ext4），跨系统访问较慢 |
| 内核兼容性 | 部分兼容（无 Docker、无 systemd） | **完全兼容**（支持 Docker、systemd、所有内核特性） |
| 网络 | 与 Windows 共享网络栈 | 独立网络（NAT），有独立 IP |
| 内存占用 | 极低 | 动态分配，可自动回收 |
| 启动速度 | 瞬间 | 很快（1-2 秒） |
| 推荐场景 | 简单脚本、少量 Linux 工具 | **首选推荐**——几乎所有场景 |

### 建议

> **本项目使用 WSL 2。** 除非有特殊理由需要 WSL 1（比如某些 VPN 兼容性问题），否则一律使用 WSL 2。

---

## 3. WSL 能做什么

### 开发环境
- 运行 Linux 原生的编译器、解释器、构建工具（gcc、make、Python、Node.js、Rust 等）
- 使用 Bash / Zsh / Fish 作为终端和脚本环境
- 通过 VS Code Remote-WSL 扩展直接在 Linux 环境中编码

### 服务器与运维
- 运行 Docker 容器（WSL 2 原生支持）
- 使用 ssh、rsync、curl、git 等 Linux 工具
- 本地测试服务：Nginx、PostgreSQL、Redis 等

### 日常命令行工具
- 文本处理三剑客：grep、sed、awk
- 文件操作：find、rsync、tar
- 网络调试：curl、nmap、tcpdump

### Windows ↔ Linux 打通
- 从 WSL 调用 Windows 程序：`code .`、`explorer.exe .`
- 从 Windows 调用 WSL 程序：`wsl ls`、`wsl python3`
- 剪贴板互通
- 文件系统互相可见（`/mnt/c/` 对应 `C:\`；`\\wsl$\` 访问 Linux 文件）

---

## 4. WSL 与 VS Code 配合

这是本项目最推荐的工作方式：

```
┌────────────────────────────────────────┐
│          VS Code (Windows 端)           │
│  ┌──────────────────────────────────┐  │
│  │   Remote-WSL 扩展                  │  │
│  │   实际代码执行在 WSL 中             │  │
│  │   终端就是 WSL 的 Bash              │  │
│  │   文件存储在 WSL 文件系统中         │  │
│  └──────────────────────────────────┘  │
└────────────────────────────────────────┘
```

### 关键点

- **Remote-WSL 扩展**：VS Code 的 UI 在 Windows 上运行，但代码、终端、工具链全部在 WSL 中
- **文件放哪里**：代码文件放在 WSL 内部（`/home/<user>/`），不要放在 `/mnt/c/` —— 后者性能差很多
- **终端集成**：VS Code 内置终端自动连接 WSL 的 Bash
- **Git 集成**：正常使用，Git 运行在 WSL 中

---

## 5. 对 FPGA 工程师的意义

### 你日常可能用到的场景

| 场景 | WSL 的价值 |
|------|------------|
| **编写 TCL 脚本** | Linux 的 Bash + TCL 环境，脚本可复用于 Linux 服务器 |
| **编写 Bash 脚本** | 原生的 Bash 环境，不是 Git Bash 的模拟层 |
| **Verilog / SystemVerilog 开发** | 运行开源仿真工具（Verilator、Icarus Verilog）、lint 工具 |
| **文件处理与自动化** | 用 grep/sed/awk 处理约束文件、日志、网表 |
| **版本控制** | Git 运行在 Linux 中，与服务器/CI 环境一致 |
| **Python 脚本** | 数据处理、自动化流程（FPGA 常用 Python 辅助开发） |
| **与 Vivado 配合** | Vivado 本身在 Windows 上跑，但 TCL 脚本可以在 WSL 中编写和管理 |

### 现实约束

> ⚠️ **Vivado / Quartus 等 FPGA IDE 本身运行在 Windows 上**，不能（也不需要）安装在 WSL 里。WSL 的角色是提供 **脚本开发环境、命令行工具、自动化流程**，最终编译和综合还是在 Windows 的 Vivado 中完成。

---

## 6. 常见 Linux 发行版选择

在 Microsoft Store 中可以直接安装以下发行版：

| 发行版 | 特点 | 推荐人群 |
|--------|------|----------|
| **Ubuntu** | 最流行，文档和社区支持最多，软件包最全 | ⭐ **首选推荐** |
| Debian | 稳定保守，软件包版本较旧 | 追求稳定性的用户 |
| Kali Linux | 安全渗透测试工具集 | 安全工程师 |
| Alpine | 超轻量（基于 musl libc） | Docker 容器用 |

### 本项目选择

> **Ubuntu（最新 LTS 版本）**——社区最大、问题最好查、软件包最丰富、与 Vivado TCL 环境的兼容性最好。

---

## 下一步

安装 WSL 2 + Ubuntu：参见下一个文档 `02-WSL安装与基础配置.md`（待编写）。

---

*本文档是 [WSL_Use](../WSL_Use) 项目的一部分，记录在 Windows 下使用 WSL 进行 FPGA 开发的环境搭建过程。*
