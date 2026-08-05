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

> 💡 为什么用软链接而不是复制：一份配置两边共用，永远同步，不用维护两份。

---

## 4. 步骤 2：链接 skills 目录

```bash
ln -s /mnt/c/Users/<你的用户名>/.claude/skills ~/.claude/skills
```

两边共用同一套 skills（`/ls`、`/pwd`、`/git-push` 等），永久同步，无需在 WSL 侧重复制作。

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

*本文档是 [WSL_Use](../WSL_Use) 项目的一部分，记录在 Windows 下使用 WSL 进行 FPGA 开发的环境搭建过程。*
