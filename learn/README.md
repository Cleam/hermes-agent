# Hermes Agent 中文教程

> 欢迎来到 Hermes Agent 的中文学习资料库。本教程基于真实源码编写，帮助你从使用者成长为贡献者。

---

## 教程目标

- 🚀 **快速上手**：30 分钟内完成安装、配置并开始第一次对话
- 🧠 **深入理解**：掌握 Agent 循环、工具系统、记忆系统等核心机制
- 🔧 **二次开发**：能够为 Hermes 添加自定义工具、技能和插件
- 🌐 **生产部署**：将 Hermes 部署为多平台消息助手或自动化服务

---

## 目标读者

| 读者类型 | 推荐路径 |
|---------|---------|
| 普通用户（想用 AI 助手） | 00 → 01 → 02 → 05 → 13 → 15 |
| 开发者（想集成 / 扩展） | 00 → 01 → 03 → 04 → 07 → 08 → 11 → 16 |
| 运维 / DevOps | 00 → 01 → 17 → 14 → 18 → 19 |
| 研究人员 / 贡献者 | 全部章节，重点看 04、09、12、16 |

---

## 前置知识

- 基本命令行操作（Linux / macOS / WSL2）
- 了解 Python 基础（第 07、16 章需要）
- 有大语言模型使用经验（理解 API Key、token 等概念）

---

## 章节导航

### 基础篇

| 章节 | 文件 | 内容概要 |
|------|------|---------|
| [概述](00-overview.md) | `00-overview.md` | Hermes 是什么，能做什么，与同类产品的对比 |
| [安装](01-installation.md) | `01-installation.md` | 各平台安装方法，常见问题排查 |
| [快速开始](02-quick-start.md) | `02-quick-start.md` | 第一次对话、第一次工具调用的完整流程 |

### 架构篇

| 章节 | 文件 | 内容概要 |
|------|------|---------|
| [核心架构](03-core-architecture.md) | `03-core-architecture.md` | 整体组件图，各模块职责 |
| [Agent 循环](04-agent-loop.md) | `04-agent-loop.md` | run_conversation() 深度解析，工具调用流程 |
| [配置系统](05-configuration.md) | `05-configuration.md` | config.yaml 结构，三种配置加载器，多配置文件 |
| [模型提供商](06-model-providers.md) | `06-model-providers.md` | 109+ 提供商，切换方法，OAuth 提供商 |

### 功能篇

| 章节 | 文件 | 内容概要 |
|------|------|---------|
| [工具系统](07-tools-system.md) | `07-tools-system.md` | 工具注册、发现、调用链，如何添加自定义工具 |
| [技能系统](08-skills-system.md) | `08-skills-system.md` | 技能 vs 工具，SKILL.md 格式，自动生成技能 |
| [记忆系统](09-memory-system.md) | `09-memory-system.md` | 内置记忆、插件记忆、FTS5 会话搜索 |
| [上下文管理](10-context-management.md) | `10-context-management.md` | 上下文窗口，自动压缩，SOUL.md，项目上下文 |
| [MCP 集成](11-mcp-integration.md) | `11-mcp-integration.md` | MCP 协议配置，stdio/HTTP 传输，Hermes 作为 MCP 服务端 |
| [子智能体](12-subagents.md) | `12-subagents.md` | delegate_task，并行处理，批处理，MoA 模式 |

### 操作篇

| 章节 | 文件 | 内容概要 |
|------|------|---------|
| [CLI 与 TUI](13-cli-and-tui.md) | `13-cli-and-tui.md` | 完整命令参考，TUI 界面，快捷键 |
| [消息网关](14-web-and-messaging.md) | `14-web-and-messaging.md` | 18+ 平台接入，网关配置，审批流程 |
| [实战案例](15-real-world-use-cases.md) | `15-real-world-use-cases.md` | 5 个真实场景：个人助理、编程、运维、知识管理、自动化 |

### 进阶篇

| 章节 | 文件 | 内容概要 |
|------|------|---------|
| [源码阅读指南](16-source-code-reading.md) | `16-source-code-reading.md` | 源码模块关系图，推荐阅读顺序 |
| [部署与运维](17-deployment-and-ops.md) | `17-deployment-and-ops.md` | 本地、Docker、VPS、NixOS、多配置文件部署 |
| [安全最佳实践](18-security-best-practices.md) | `18-security-best-practices.md` | API 密钥保护，工具审批，容器沙箱，访问控制 |
| [故障排查](19-troubleshooting.md) | `19-troubleshooting.md` | 常见问题诊断与解决 |
| [常见问题](20-faq.md) | `20-faq.md` | FAQ：与 ChatGPT / Claude Code 对比、迁移等 |

### 附录

| 文件 | 内容 |
|------|------|
| [`assets/architecture.mmd`](assets/architecture.mmd) | 系统架构 Mermaid 图 |
| [`assets/agent-loop.mmd`](assets/agent-loop.mmd) | Agent 循环流程图 |
| [`assets/tool-flow.mmd`](assets/tool-flow.mmd) | 工具调用时序图 |
| [`assets/memory-flow.mmd`](assets/memory-flow.mmd) | 记忆流程图 |
| [`assets/skill-flow.mmd`](assets/skill-flow.mmd) | 技能激活流程图 |
| [`assets/deployment.mmd`](assets/deployment.mmd) | 部署架构图 |

---

## 推荐学习路径

### 🚀 快速上手路径（1 小时）
```
00-overview → 01-installation → 02-quick-start → 05-configuration → 13-cli-and-tui
```

### 🧠 深度理解路径（半天）
```
03-core-architecture → 04-agent-loop → 07-tools-system → 09-memory-system → 10-context-management
```

### 💻 开发者路径（全天）
```
03 → 04 → 07 → 08 → 11 → 12 → 16-source-code-reading
```

### 🌐 部署运维路径（2 小时）
```
17-deployment-and-ops → 14-web-and-messaging → 18-security-best-practices → 19-troubleshooting
```

---

## 如何配合源码使用

建议边读文档边打开对应源文件：

```bash
# 克隆仓库（贡献者）
git clone https://github.com/NousResearch/hermes-agent.git
cd hermes-agent

# 常用文件速查
cat run_agent.py         # Agent 核心循环（约 12k 行）
cat model_tools.py       # 工具编排
cat toolsets.py          # 工具集定义
cat hermes_cli/main.py   # CLI 入口
cat hermes_cli/config.py # 配置系统
cat tools/registry.py    # 工具注册表
```

每个章节末尾都有 **相关源码** 参考，方便直接跳转查看。

---

## 版本说明

本教程基于 Hermes Agent 最新主分支编写，部分 API 可能随版本迭代有所变化。遇到不一致时，以实际源码为准。

```bash
hermes version  # 查看当前版本
hermes update   # 更新到最新版本
```

---

> 💡 **提示**：如果你是第一次接触 AI Agent，建议从 [00-overview.md](00-overview.md) 开始，了解整体概念后再动手安装。如果你急着试用，可以直接跳到 [01-installation.md](01-installation.md)。
