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
5. [iverilog 使用详解](#5-iverilog-使用详解)
6. [GTKWave 使用入门](#6-gtkwave-使用入门)
7. [与 cocotb 配合](#7-与-cocotb-配合)
8. [常见问题](#8-常见问题)

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

## 5. iverilog 使用详解

### 5.1 基本三步

```bash
iverilog -o sim counter.sv tb_counter.v   # ① 编译：Verilog 源码 → 仿真程序 sim
vvp sim                                   # ② 运行：执行仿真，生成波形
gtkwave dump.vcd &                        # ③ 看波形
```

### 5.2 常用选项

| 选项 | 作用 | 示例 |
|------|------|------|
| `-o <文件>` | 指定输出文件名 | `-o sim` |
| `-s <模块>` | 指定顶层模块（多模块时） | `-s counter` |
| `-g2012` | 按 SystemVerilog-2012 标准编译（.sv 文件） | `-g2012 counter.sv` |
| `-Wall` | 显示全部警告（找潜在问题） | `-Wall counter.sv` |
| `-I <目录>` | 添加 include 搜索目录（`include "xxx.v"`） | `-I include/` |
| `-y <目录>` | 添加模块库目录（自动按需搜索模块） | `-y lib/` |
| `-tn` | 只做语法检查，不生成仿真程序 | `-tn counter.sv` |

> 💡 实测：`.sv` 文件不写 `-g2012` 一般也能编译（iverilog 按扩展名自动识别），但加 `-g2012` 最保险（严格按 SV-2012 标准）。

### 5.3 语法检查（写代码后先跑这个）

```bash
iverilog -tn counter.sv && echo "语法 OK"    # 无输出 = 通过
```

写完 RTL 先语法检查再仿真，是省时间的好习惯。

### 5.4 多文件项目

```bash
# 所有源文件一起列出即可，iverilog 自动解析模块依赖关系
iverilog -o sim top.sv sub_module1.sv sub_module2.sv tb_top.v
```

### 5.5 波形语句 `$dumpfile` / `$dumpvars`（测试台里）

```verilog
initial begin
    $dumpfile("dump.vcd");     // 波形文件名（扩展名决定格式：.vcd 或 .fst）
    $dumpvars(0, tb_counter);  // 记录范围：0 = 该模块及以下所有层级
end
```

> 这两句写在测试台的 `initial` 块里，仿真一运行就自动开始记录波形。

---

## 6. GTKWave 使用入门

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

**工具栏图标速查**（实测体验）：

| 图标 | 作用 |
|------|------|
| Zoom In / Zoom Out（放大/缩小镜） | 缩放时间轴（或 `Ctrl+滚轮`） |
| Zoom Fit | 一键显示全部波形 |
| Zoom to Start / Zoom to End | 跳到波形开头 / 末尾 |
| Search（放大镜+箭头） | 搜索信号跳变沿（找某次上升/下降） |
| Mark（旗帜） | 在时间轴打标记，量两点之间的时间 |
| Append / Replace | 把左侧选中信号加入波形区 / 替换现有波形 |
| 波形区右键菜单 | Set Cursor（精确定位时间）、更多缩放选项 |

> 这些图标只改变"查看方式"，**不会修改波形数据本身**——随便点，多试试就熟了。

### 纯 iverilog 手动流程（不用 make，理解原理）

不依赖 cocotb 的最简方式，4 步走通"编译 → 仿真 → 波形"：

**① 写测试台 `tb_counter.v`**（手动驱动时钟/复位）：

```verilog
`timescale 1ns/1ps
module tb_counter;
    reg clk = 0;
    reg rst = 1;
    wire [7:0] count;

    counter dut (.clk(clk), .rst(rst), .count(count));  // 例化被测模块

    always #5 clk = ~clk;          // 每 5ns 翻转 → 10ns 周期时钟

    initial begin
        $dumpfile("dump.vcd");     // ⭐ 生成波形
        $dumpvars(0, tb_counter);  //    记录所有信号
        #10 rst = 0;               // 复位后释放
        #100 $finish;              // 仿真结束
    end
endmodule
```

**② 编译**（iverilog 把两个文件编译成仿真程序 `sim`）：

```bash
iverilog -o sim counter.sv tb_counter.v
```

**③ 运行仿真**（vvp 执行，自动生成 `dump.vcd`）：

```bash
vvp sim
```

**④ 打开波形**：

```bash
gtkwave dump.vcd &
```

> 💡 这套流程的好处：**每一步都看得见、可理解**（编译 → 运行 → 波形）。cocotb 只是把这套流程用 Python 断言自动化了，底层还是这几步。

---

## 7. 与 cocotb 配合

cocotb 用 iverilog 作默认仿真器，生成波形的方法（**实测有效**）：

```makefile
# Makefile 中加一行：
WAVES=1
```

运行 `make` 后生成波形文件 `sim_build/counter.fst`，然后用 GTKWave 打开：

```bash
gtkwave sim_build/counter.fst
```

> ⚠️ **实测修正**：`COMPILE_ARGS += --trace` 是 **Verilator** 的写法，iverilog 会报 `invalid option -- '-'`。iverilog + cocotb 请用 `WAVES=1`。

**想要 VCD 格式？** cocotb 的 iverilog 支持固定生成 `.fst`（源码里写死了扩展名），用 `fst2vcd` 转换（GTKWave 自带）：

```bash
fst2vcd sim_build/counter.fst > sim_build/counter.vcd   # 注意：输出到标准输出，用 > 重定向
gtkwave sim_build/counter.vcd                           # 打开 VCD
```

> 传统 iverilog 方式（不依赖 cocotb）可以直接生成 VCD：测试台里写 `$dumpfile("dump.vcd"); $dumpvars;`（见 [§6](#6-gtkwave-使用入门) 手动流程和 [§5.5](#55-波形语句-dumpfile--dumpvars测试台里)）。

**推荐流程**：cocotb 写断言验证功能 → GTKWave 看波形分析时序，两者互补。

---

## 8. 常见问题

### Q: `gtkwave` 启动没有窗口 / 窗口点不出来

WSLg 会话异常。`wsl --shutdown`（Windows PowerShell 中执行）后重新打开终端再试。

### Q: 波形里中文显示为方块

缺少中文字体：`sudo apt install fonts-noto-cjk -y`，然后 `fc-cache -fv`。

### Q: 提示 `cannot open display`

不应发生（WSLg 自动设置 DISPLAY）。检查 `echo $DISPLAY` 应为 `:0`；仍异常则 `wsl --shutdown` 重启。

---

*本文档是 [WSL_Use](../WSL_Use) 项目的一部分，记录在 Windows 下使用 WSL 进行 FPGA 开发的环境搭建过程。*
