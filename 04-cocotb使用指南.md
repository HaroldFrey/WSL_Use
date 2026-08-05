# cocotb 使用指南——用 Python 验证你的 RTL

> **文档编号**：04
> **前置条件**：已完成 `02-WSL安装与基础配置.md`（WSL 环境 + Python 3.12 + make）
> **目的**：介绍 cocotb 是什么、如何在 WSL 中安装，以及用 Python 编写 RTL 测试

---

## 目录

1. [cocotb 是什么](#1-cocotb-是什么)
2. [工作原理](#2-工作原理)
3. [安装](#3-安装)
4. [第一个测试：计数器示例](#4-第一个测试计数器示例)
5. [运行与查看结果](#5-运行与查看结果)
6. [常用操作速查](#6-常用操作速查)
7. [常见问题](#7-常见问题)

---

## 1. cocotb 是什么

**cocotb**（Coroutine-based COsimulation TestBench）是**用 Python 写 RTL 测试**的开源验证框架。让你用 Python 的简洁语法驱动 Verilog/SystemVerilog 信号、写断言、做仿真，替代传统的 SystemVerilog testbench。

**和传统方式的对比**：

| | 传统 SV testbench | cocotb |
|--|------------------|--------|
| 语言 | SystemVerilog | **Python** |
| 写起来 | 语法繁琐，需要完整的 SV 知识 | 简洁直观，普通 Python 语法 |
| 复用 | 每次重写 | Python 生态随便用（numpy、正则、pip 包） |
| 仿真器 | 依赖各家仿真器的 TB 机制 | 同一套测试代码，换仿真器不用改 |

**适合谁**：RTL 工程师快速验证模块功能；对 SystemVerilog 验证方法学（UVM）不熟悉的人。

---

## 2. 工作原理

```
你的测试代码（Python）          DUT（Verilog/SystemVerilog）
┌──────────────────┐           ┌────────────────────┐
│ test_counter.py  │  cocotb  │   counter.sv        │
│ 控制时钟、信号     │ ────────→ │   被测模块           │
│ 写断言检查结果     │  驱动/采样 │                     │
└──────────────────┘           └────────────────────┘
          │
          ▼
    HDL 仿真器（iverilog / verilator / Vivado xsim / Questa）
```

- cocotb 作为 Python 库，被 **HDL 仿真器**加载，通过 VPI 接口直接驱动/采样 DUT 的信号
- 你的测试写 `async def` 协程，用 `await` 等待时钟沿、延迟等事件
- 一个测试文件（`.py`） + 一个 Makefile + DUT（`.sv`），三件套即可运行

---

## 3. 安装

### 3.1 安装 HDL 仿真器（二选一）

```bash
# 方案 A：Icarus Verilog（开源，推荐入门，cocotb 官方默认支持）
sudo apt install iverilog -y
iverilog -V    # 应显示 Icarus Verilog version 12.0

# 方案 B：Verilator（开源，速度快但语法要求严格）
# sudo apt install verilator -y
```

> 💡 cocotb 也支持 Vivado xsim、Questa、VCS 等商业仿真器（`SIM=xcelium` 等参数），本指南以 iverilog 为例。

### 3.2 安装 cocotb（venv 虚拟环境，推荐）

Ubuntu 24.04 的 pip 受 PEP 668 保护，**不允许直接装系统包**，用 venv 虚拟环境最规范：

```bash
# 1. 创建虚拟环境（路径自定，如 ~/cocotb-env）
python3 -m venv ~/cocotb-env

# 2. 激活（每次开新终端跑测试前都要激活）
source ~/cocotb-env/bin/activate

# 3. 安装 cocotb
pip install cocotb

# 4. 验证
cocotb-config --version    # 应显示 2.x.x

# 5.（可选）安装 pytest：断言失败时能显示更友好的报错信息
pip install pytest
```

> ⚠️ **PEP 668 报错**：直接在系统里 `pip install cocotb` 会报 `externally-managed-environment`——这是 Ubuntu 24.04 的保护机制，**不要**用 `--break-system-packages` 绕过，用 venv 即可。
>
> ⚠️ **每次使用前记得 `source` 激活**：新开终端后 venv 不会自动生效，未激活时 `cocotb-config` 找不到。

---

## 4. 第一个测试：计数器示例

### 4.1 文件结构

```
~/projects/cocotb_counter/
├── counter.sv        ← 被测模块（DUT）
├── test_counter.py   ← cocotb 测试代码
└── Makefile          ← 构建/运行脚本
```

### 4.2 DUT：counter.sv（8 位计数器）

```systemverilog
module counter #(
    parameter WIDTH = 8
) (
    input  wire             clk,
    input  wire             rst,
    output reg [WIDTH-1:0]  count
);
    always @(posedge clk) begin
        if (rst)
            count <= 0;
        else
            count <= count + 1;
    end
endmodule
```

### 4.3 测试代码：test_counter.py

```python
import cocotb
from cocotb.clock import Clock
from cocotb.triggers import FallingEdge

@cocotb.test()
async def test_counter(dut):
    """测试计数器复位后正常递增"""
    # 启动 10ns 周期时钟（cocotb 2.x 参数名为 unit，旧版为 units）
    cocotb.start_soon(Clock(dut.clk, 10, unit="ns").start())

    # 复位
    await FallingEdge(dut.clk)
    dut.rst.value = 1
    await FallingEdge(dut.clk)
    assert dut.count.value == 0, f"复位失败: {dut.count.value}"

    # 释放复位，验证递增 1-4
    dut.rst.value = 0
    for i in range(1, 5):
        await FallingEdge(dut.clk)
        assert dut.count.value == i, f"期望 {i}, 实际 {dut.count.value}"

    print("counter 测试通过！")
```

**逐行解释**：

| 代码 | 作用 |
|------|------|
| `@cocotb.test()` | 标记这是一个测试用例 |
| `Clock(dut.clk, 10, units="ns")` | 生成 10ns 周期时钟 |
| `cocotb.start_soon(...)` | 后台启动时钟协程 |
| `await FallingEdge(dut.clk)` | 等待时钟下降沿（同步时序） |
| `dut.rst.value = 1` | 给信号赋值（驱动） |
| `dut.count.value` | 读取信号当前值（采样） |
| `assert ...` | 断言，失败则测试失败并打印信息 |

### 4.4 Makefile

```makefile
SIM ?= icarus
TOPLEVEL_LANG ?= verilog
VERILOG_SOURCES = $(PWD)/counter.sv
TOPLEVEL = counter
COCOTB_TEST_MODULES = test_counter    # cocotb 2.x 写法（旧版 MODULE 已弃用）

include $(shell cocotb-config --makefiles)/Makefile.sim
```

| 变量 | 含义 |
|------|------|
| `SIM` | 仿真器（icarus / verilator / xcelium ...） |
| `TOPLEVEL_LANG` | DUT 语言（verilog / systemverilog） |
| `VERILOG_SOURCES` | DUT 源文件列表 |
| `TOPLEVEL` | 顶层模块名（即 DUT 名） |
| `COCOTB_TEST_MODULES` | Python 测试文件名（不含 .py，cocotb 2.x 写法；旧版用 `MODULE`，兼容但会提示弃用） |

---

## 5. 运行与查看结果

```bash
source ~/cocotb-env/bin/activate    # 激活 venv
cd ~/projects/cocotb_counter
make
```

预期输出关键行：

```
     -!- test_counter.test_counter passed ✓
TEST                           PASS ✓
```

全部 `PASS` 即测试通过。

**排错提示**：

| 报错 | 原因 | 解决 |
|------|------|------|
| `make: *** No rule to make target '/Makefile.sim'` | `cocotb-config` 在 make 中找不到 | 确认 venv 已激活；若 PATH 含空格的 Windows 路径（WSL 常见），Makefile 中改写成 `COCOTB_CONFIG = /home/<用户名>/cocotb-env/bin/cocotb-config`，再 `include $(shell $(COCOTB_CONFIG) --makefiles)/Makefile.sim` |
| `No module named cocotb` | venv 未激活 | `source ~/cocotb-env/bin/activate` |
| `iverilog: command not found` | 仿真器未装 | `sudo apt install iverilog -y` |
| 测试失败但无具体信息 | 断言报错 | 查看输出中 `Traceback` 和 `AssertionError` |

---

## 6. 常用操作速查

```python
# 时钟：多种周期（cocotb 2.x 参数名为 unit）
cocotb.start_soon(Clock(dut.clk, 10, unit="ns").start())    # 10ns
cocotb.start_soon(Clock(dut.clk, 100, unit="ps").start())   # 100ps

# 等待事件
await RisingEdge(dut.clk)      # 上升沿
await FallingEdge(dut.clk)     # 下降沿
await Timer(5, units="ns")     # 固定延迟
await ReadOnly()               # 信号稳定后（组合逻辑结果）

# 信号赋值（驱动）
dut.data_in.value = 0xFF
dut.enable.value = 1
dut.data.value = 0b1010

# 信号读取（采样）
val = dut.data_out.value
if dut.valid.value == 1:
    ...

# 日志输出
dut._log.info("状态信息")       # 带模块名的日志
```

---

## 7. 常见问题

### Q: cocotb 和 UVM 有什么区别？

cocotb 是轻量的 Python 验证方案，适合模块级快速验证；UVM 是重型的 SystemVerilog 验证方法学，适合大型项目/SoC 验证。新手和中小模块 cocotb 上手更快。

### Q: 能测 SystemVerilog 接口（AXI-Stream 等）吗？

可以。`TOPLEVEL_LANG = systemverilog`，配合 `cocotb-bus` 库（`pip install cocotb-bus`）有现成的 AXI-Stream / AXI-Lite 总线驱动。

### Q: 换 Vivado 的 xsim 仿真器怎么跑？

```bash
make SIM=xsim          # Vivado 自带仿真器（需 Vivado 在 PATH 中）
```

### Q: 波形文件怎么生成？

```bash
# Makefile 中加：
#   COMPILE_ARGS += --trace   (iverilog 生成 fst 波形)
# 运行后：
gtkwave dump.fst      # 用 GTKWave 打开波形
```

### Q: 运行时有 `units 参数改名`、`MODULE deprecated` 之类的警告？

cocotb 2.x 的 API 变化：`Clock(..., units=...)` 改名为 `unit=...`；Makefile 的 `MODULE` 改为 `COCOTB_TEST_MODULES`。**警告不影响结果**，但建议按新写法更新（本文档代码已更新）。

### Q: 提示 `pytest not found`？

可选优化。安装 pytest 后断言失败时会显示更详细的报错（哪个值、期望什么）：`pip install pytest`。

### Q: 用 wsl.exe 从 Windows 侧创建 Makefile，`$(PWD)` 变成了空路径？（实测坑）

通过 `wsl.exe -u 用户 -- bash -c '...'` 多层 shell 传参时，`$(PWD)`、`$(shell ...)` 会被中间层 shell **提前展开成空**，导致 Makefile 变成 `VERILOG_SOURCES = /counter.sv`、`include /Makefile.sim`，报 `No rule to make target`。

**解法（二选一）**：
- 直接在 WSL 终端里用 `vim`/`gedit` 等编辑器创建 Makefile（推荐，不会经过多层 shell）
- 若必须从 Windows 侧写入：先把内容 **base64 编码**（不含 shell 特殊字符），再在 WSL 里 `echo '<编码>' | base64 -d > Makefile`

---

> 📌 cocotb 官方文档：https://docs.cocotb.org
