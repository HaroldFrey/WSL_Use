# iverilog 与 GTKWave 安装指南（WSL）

> **文档编号**：06
> **前置条件**：已完成 [02-WSL安装与基础配置.md](02-WSL安装与基础配置.md)
> **目的**：在 WSL 中安装 HDL 仿真器（iverilog）和波形查看器（GTKWave），并学会基本使用

---

## 目录

1. [它们是什么](#1-它们是什么)
2. [检查是否已安装](#2-检查是否已安装)
3. [安装](#3-安装)
4. [验证安装](#4-验证安装)
5. [GTKWave 使用入门](#5-gtkwave-使用入门)
6. [与 cocotb 配合](#6-与-cocotb-配合)
7. [常见问题](#7-常见问题)

---

## 1. 它们是什么

| 工具 | 角色 | 通俗类比 |
|------|------|----------|
| **iverilog**（Icarus Verilog） | HDL **仿真器**——把 Verilog 代码编译并跑仿真 | "跑 Verilog 代码的引擎" |
| **GTKWave** | **波形查看器**——把仿真产生的波形可视化 | "波形示波器" |

**工作流程**：

```
写 Verilog 代码 → iverilog 仿真 → 生成波形文件（.fst/.vcd）→ GTKWave 打开查看
```

iverilog 是 cocotb 官方默认支持的仿真器（见 [04-cocotb使用指南.md](04-cocotb使用指南.md)），GTKWave 通过 WSLg 图形界面直接在 Windows 桌面上显示波形。

---

## 2. 检查是否已安装

在 Ubuntu 终端执行：

```bash
iverilog -V 2>&1 | head -1        # 应显示 Icarus Verilog version 12.0
gtkwave --version | head -1       # 应显示 GTKWave Analyzer v3.3.x
```

有输出版本号 = 已安装 ✅（本机实测：iverilog 12.0 / GTKWave 3.3.116，均已就绪）
显示 `command not found` = 未安装，执行第 3 步。

---

## 3. 安装

如果未安装，在 Ubuntu 终端执行：

```bash
# 一条命令装两个（iverilog + gtkwave）
sudo apt install iverilog gtkwave -y
```

> 💡 GTKWave 是 GUI 程序，通过 WSLg 显示（WSL 2 自带图形支持，无需额外配置）。首次使用建议同时装中文字体（波形里的中文标签不乱码）：
> ```bash
> sudo apt install fonts-noto-cjk -y
> ```

---

## 4. 验证安装

```bash
iverilog -V 2>&1 | head -1    # Icarus Verilog version 12.0
gtkwave --version | head -1   # GTKWave Analyzer v3.3.116
gtkwave                      # 启动后 Windows 桌面上应弹出 GTKWave 主窗口
```

GTKWave 主窗口能弹出 = WSLg 图形通路正常 ✅

> ⚠️ 若窗口只在任务栏有图标但点不出来，`wsl --shutdown` 后重开（详见 [02 §7.2](02-WSL安装与基础配置.md#72-wslg-不工作的排查)）。

---

## 5. GTKWave 使用入门

### 打开波形文件

```bash
gtkwave dump.fst       # 打开 FST 波形（推荐，文件小、速度快）
gtkwave dump.vcd       # 打开 VCD 波形（通用格式，文件较大）
gtkwave dump.fst &
```

打开后界面：

```
┌─────────────────────────────────────────┐
│ 左栏：信号列表（SST）                     │
│   点击信号 → 拖到下方波形区              │
│ 下方：波形显示区（时间轴 + 信号波形）     │
│ 工具栏：缩放、搜索、书签                 │
└─────────────────────────────────────────┘
```

基本操作：

| 操作 | 方法 |
|------|------|
| 添加信号到波形区 | 左栏选中信号 → 点"Append"（或直接拖拽） |
| 放大/缩小 | 工具栏放大镜按钮，或 `Ctrl+滚轮` |
| 查看某时刻值 | 点击波形区，看右侧的数值栏 |
| 批量添加 | 左栏 `Ctrl+A` 全选 → Append |

### 让 iverilog 生成波形（不依赖 cocotb）

```bash
# 编译并生成 VCD 波形
iverilog -o sim counter.v tb_counter.v
vvp sim

# 测试台里要有：
#   $dumpfile("dump.vcd"); $dumpvars;
```

---

## 6. 与 cocotb 配合

cocotb 用 iverilog 作默认仿真器，生成波形的方法（见 [04 §7](04-cocotb使用指南.md#7-常见问题)）：

```makefile
# Makefile 中加一行：
COMPILE_ARGS += --trace
```

运行 `make` 后生成 `dump.fst`，然后用 GTKWave 打开：

```bash
gtkwave dump.fst
```

**推荐流程**：cocotb 写断言验证功能 → GTKWave 看波形分析时序，两者互补。

---

## 7. 常见问题

### Q: `gtkwave` 启动没有窗口 / 窗口点不出来

WSLg 会话异常。`wsl --shutdown`（Windows PowerShell 中执行）后重新打开终端再试。

### Q: 波形里中文显示为方块

缺少中文字体：`sudo apt install fonts-noto-cjk -y`，然后 `fc-cache -fv`。

### Q: 提示 `cannot open display`

不应发生（WSLg 自动设置 DISPLAY）。检查 `echo $DISPLAY` 应为 `:0`；仍异常则 `wsl --shutdown` 重启。

---

*本文档是 [WSL_Use](../WSL_Use) 项目的一部分，记录在 Windows 下使用 WSL 进行 FPGA 开发的环境搭建过程。*
