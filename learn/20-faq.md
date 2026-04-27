# 第 20 章：常见问题解答（FAQ）

> 本章汇集最常见的问题，帮助你快速找到答案。

---

## 产品定位类

### Q1：Hermes 和 ChatGPT 有什么区别？

**A**：核心区别在于工具调用能力和持久性：

| 维度 | Hermes Agent | ChatGPT |
|------|-------------|---------|
| 工具调用 | ✅ 真实执行（终端、文件、浏览器） | 有限（代码解释器在沙盒内） |
| 持久记忆 | ✅ 跨会话记住你 | 有限（Projects 功能） |
| 自我改进 | ✅ 自动生成技能 | ❌ |
| 本地运行 | ✅ 完全本地 | ❌（必须联网） |
| 消息平台 | ✅ 18+ 平台 | ❌ |
| 模型选择 | ✅ 109+ 提供商 | 限 OpenAI 模型 |
| 开源 | ✅ | ❌ |

简而言之：ChatGPT 是"对话工具"，Hermes 是"能干活的 AI 同事"。

---

### Q2：Hermes 和 Claude Code / Cursor 有什么区别？

**A**：主要区别是定位和部署方式：

- **Claude Code / Cursor**：IDE 插件，专注代码开发，绑定特定编辑器
- **Hermes Agent**：独立 CLI 工具，不绑定 IDE，支持多模型、多平台

Hermes 的独特优势：
- 不需要特定 IDE，在任何终端运行
- 支持 109+ 模型提供商（不只是 Anthropic）
- 18+ 消息平台接入
- 持久记忆和自我改进技能
- 开源，可自托管

---

### Q3：Hermes 和 OpenClaw 是什么关系？

**A**：**Hermes Agent 是 OpenClaw 的直接继承者**。如果你之前使用过 OpenClaw，现在它已更名为 Hermes Agent，由 Nous Research 持续维护。

迁移方式：

```bash
# 安装 Hermes 后，迁移 OpenClaw 配置
hermes claw migrate
```

这会自动迁移你的 OpenClaw 配置、记忆和技能到 Hermes 格式。

---

## 使用场景类

### Q4：Hermes 适合日常编程使用吗？

**A**：非常适合！Hermes 是优秀的编程助手：

```bash
# 进入项目目录
cd /my/project

# 创建项目上下文文件（可选，但推荐）
cat > AGENTS.md << 'EOF'
## 项目：我的 FastAPI 服务
- 语言：Python 3.12
- 测试：pytest tests/
- 启动：uvicorn app.main:app --reload
EOF

# 开始编程
hermes
你：帮我给 user API 添加分页功能
```

Hermes 会直接读取代码、修改文件、运行测试——全程在终端完成。

---

### Q5：Hermes 能作为长期个人助理吗？

**A**：这正是 Hermes 的核心设计目标之一。通过以下组合：

- **记忆系统**：记住你的偏好、工作环境、常用信息
- **Telegram 网关**：手机随时访问
- **SOUL.md**：定义助理人格
- **定时任务**：每日摘要、提醒

配置步骤详见 [第 15 章：实战案例](15-real-world-use-cases.md)。

---

### Q6：记忆功能可靠吗？会"记错"吗？

**A**：记忆系统有其局限性，需要理解：

**可靠的地方**：
- MEMORY.md 和 USER.md 是纯文本文件，内容完全可审查
- Agent 主动写入（不是"自动学习"黑盒）
- 你可以随时查看和修改

**可能的问题**：
- LLM 可能提炼出不够准确的摘要
- 旧的记忆可能与当前情况不符
- 长期积累可能出现矛盾信息

**最佳实践**：
```bash
# 定期审查记忆
cat ~/.hermes/memories/MEMORY.md
cat ~/.hermes/memories/USER.md

# 发现错误时直接编辑文件
nano ~/.hermes/memories/MEMORY.md

# 或者告诉 Agent
你：你关于我服务器 IP 的记忆是错的，正确的是 10.0.1.6，请更新
```

---

### Q7：技能（Skill）和工具（Tool）有什么区别？

**A**：

| | 工具（Tool） | 技能（Skill） |
|--|------------|-------------|
| **本质** | Python 函数 | Markdown 指令文件 |
| **执行** | 直接调用代码 | 注入为用户消息，引导行为 |
| **创建** | 写 Python 代码 | 写 Markdown |
| **示例** | `terminal`（执行命令）| `github/code-review`（如何做代码审查）|

**类比**：工具是`锤子`（实物），技能是`操作手册`（如何用锤子建房子）。

技能的最大优势：**不需要编程**就能扩展 Agent 能力。你甚至可以直接让 Agent 帮你创建技能：

```
你：帮我创建一个技能，记录如何部署这个项目到 AWS EC2
# Agent 会总结部署步骤，创建 SKILL.md 文件
```

---

## 技术配置类

### Q8：MCP 是必须的吗？

**A**：**不是必须的**。Hermes 内置了大量工具（终端、文件、网络、浏览器等），MCP 是可选的**增强**：

- **不配置 MCP**：使用 Hermes 内置工具，功能已经非常完整
- **配置 MCP**：可以接入更多专业工具（精细文件权限、数据库、GitHub 等）

如果你只是用 Hermes 做日常编程和运维，内置工具已经足够。MCP 适合需要精细集成特定服务的场景。

---

### Q9：如何避免 Agent 执行危险命令？

**A**：多层防护机制：

1. **SOUL.md 约束**：定义绝对禁止的操作
   ```markdown
   ## 绝对禁止
   - 不执行 rm -rf / 等系统级删除
   - 删除超过 1GB 数据前必须确认
   ```

2. **审批系统**：消息网关模式下危险命令需要 `/approve` 确认

3. **Docker 沙箱**：`terminal.backend: docker` 隔离执行环境

4. **工具强制策略**：
   ```yaml
   agent:
     tool_use_enforcement: "auto"  # 模型自行判断是否需要工具
   ```

5. **交互模式确认**：本地 CLI 中危险命令会请求用户确认

---

### Q10：如何迁移配置和记忆到新机器？

**A**：非常简单，备份整个 `~/.hermes/` 目录：

```bash
# 旧机器：备份
tar -czf hermes-backup.tar.gz ~/.hermes/

# 传输到新机器（例如通过 scp）
scp hermes-backup.tar.gz user@new-machine:~/

# 新机器：安装 Hermes
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash

# 恢复数据
tar -xzf ~/hermes-backup.tar.gz -C ~/
```

核心文件：
- `config.yaml`：配置
- `.env`：API 密钥
- `memories/`：记忆（最重要！）
- `skills/`：自定义技能

---

## 调试类

### Q11：如何调试 Hermes 的执行过程？

**A**：多种调试手段：

```bash
# 1. 查看实时日志
hermes logs --follow

# 2. 只看错误
hermes logs --level error

# 3. 全面诊断
hermes doctor

# 4. 查看 Token 使用（在 Hermes 交互中）
/usage

# 5. 查看会话洞察
/insights

# 6. 查看 Agent 状态
/status
```

---

### Q12：最值得阅读的核心源码文件是哪些？

**A**：按优先级：

1. **`run_agent.py`**：Agent 核心循环（必读）
2. **`model_tools.py`**：工具编排机制
3. **`hermes_cli/main.py`**：CLI 入口和命令路由
4. **`tools/registry.py`**：工具注册表（简短但关键）
5. **`hermes_cli/config.py`**：配置系统和默认值
6. **`hermes_cli/commands.py`**：斜杠命令注册表

详细阅读指南见 [第 16 章：源码阅读指南](16-source-code-reading.md)。

---

### Q13：Hermes 支持 Windows 吗？

**A**：**原生 Windows 不支持**，但可以通过以下方式使用：

- **WSL2（推荐）**：在 Windows 上运行 WSL2，安装 Ubuntu 后即可正常使用
- **Docker Desktop**：通过 Docker 运行（功能受限）

TUI 模式（`hermes --tui`）在 WSL2 中工作正常。PTY 桥接（仪表盘嵌入 TUI）不支持原生 Windows（只支持 POSIX）。

---

### Q14：如何查看 Hermes 当前版本？

```bash
hermes version
```

更新到最新版本：

```bash
hermes update
```

---

## 本章小结

- Hermes 与 ChatGPT 的核心区别：真实工具调用、持久记忆、自我改进、多平台、开源
- Hermes 是 OpenClaw 的继承者，通过 `hermes claw migrate` 迁移
- 技能（Skill）= Markdown 指令文件；工具（Tool）= Python 函数
- MCP 是可选增强，内置工具已足够日常使用
- 危险命令防护：SOUL.md 约束 + 审批系统 + Docker 沙箱
- 迁移：备份 `~/.hermes/`，新机器解压即可
- 调试：`hermes doctor`、`hermes logs`、`/usage`、`/status`
- 不支持原生 Windows，推荐使用 WSL2
