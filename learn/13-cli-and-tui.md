# 第 13 章：CLI 与 TUI

> 相关源码：`hermes_cli/main.py`、`cli.py`、`hermes_cli/commands.py`、`ui-tui/`

---

## CLI 命令参考

Hermes 提供丰富的命令行接口（CLI）。以下是完整命令列表：

### 核心命令

| 命令 | 说明 |
|------|------|
| `hermes` | 启动交互式聊天（默认） |
| `hermes --tui` | 启动 TUI 界面（需要 Node.js） |
| `hermes --oneshot "问题"` | 一次性问答，不进入交互模式 |
| `hermes --oneshot < input.txt` | 从标准输入读取问题 |

### 配置与设置

| 命令 | 说明 |
|------|------|
| `hermes setup` | 引导式配置向导 |
| `hermes model` | 交互式选择模型 |
| `hermes config show` | 显示当前配置 |
| `hermes config set <key> <value>` | 设置配置项 |
| `hermes config edit` | 打开编辑器编辑配置 |

### 工具与技能

| 命令 | 说明 |
|------|------|
| `hermes tools` | 查看可用工具列表 |
| `hermes tools --disable <toolset>` | 禁用工具集 |
| `hermes skills browse` | 浏览可用技能 |
| `hermes skills install <skill>` | 安装技能 |
| `hermes skills create <name>` | 创建新技能 |
| `hermes skills publish <name>` | 发布技能到 Skills Hub |

### 消息网关

| 命令 | 说明 |
|------|------|
| `hermes gateway` | 前台运行消息网关 |
| `hermes gateway start` | 后台启动网关 |
| `hermes gateway stop` | 停止网关 |
| `hermes gateway status` | 查看网关状态 |
| `hermes gateway install` | 安装为系统服务 |
| `hermes gateway uninstall` | 卸载系统服务 |
| `hermes gateway setup` | 引导配置网关平台 |

### 会话与历史

| 命令 | 说明 |
|------|------|
| `hermes sessions browse` | 浏览历史会话 |
| `hermes logs` | 查看最近日志 |
| `hermes logs --follow` | 实时跟踪日志 |
| `hermes logs --level error` | 只显示错误 |

### 计划任务

| 命令 | 说明 |
|------|------|
| `hermes cron list` | 列出计划任务 |
| `hermes cron status` | 查看计划任务状态 |

### 记忆与 Honcho

| 命令 | 说明 |
|------|------|
| `hermes honcho setup` | 配置 Honcho 记忆 |
| `hermes honcho status` | 查看 Honcho 状态 |
| `hermes honcho migrate` | 从内置迁移到 Honcho |

### 系统管理

| 命令 | 说明 |
|------|------|
| `hermes doctor` | 诊断安装和配置 |
| `hermes update` | 更新到最新版本 |
| `hermes version` | 显示版本信息 |
| `hermes status` | 查看 Agent 运行状态 |
| `hermes uninstall` | 卸载 Hermes |

### 多配置文件

| 命令 | 说明 |
|------|------|
| `hermes -p <name>` | 使用指定配置文件启动 |
| `hermes profile list` | 列出所有配置文件 |
| `hermes profile status` | 查看当前配置文件状态 |
| `hermes sethome <path>` | 设置工作目录 |

---

## 斜杠命令参考

在交互式聊天中，斜杠命令（来自 `hermes_cli/commands.py` 的 `COMMAND_REGISTRY`）提供会话控制：

### 会话管理

| 命令 | 别名 | 说明 |
|------|------|------|
| `/new` | | 开始新会话（清空上下文） |
| `/reset` | | 重置会话 |
| `/clear` | | 清屏 |
| `/redraw` | | 重绘界面 |
| `/history` | | 查看会话历史 |
| `/save` | | 保存当前会话 |
| `/retry` | | 重试上一条消息 |
| `/undo` | | 撤销上一轮对话 |
| `/title` | | 设置会话标题 |
| `/branch` | | 创建会话分支 |
| `/compress` | | 手动压缩上下文 |
| `/rollback` | | 回滚到之前状态 |
| `/snapshot` | | 创建快照 |
| `/resume` | | 恢复历史会话 |

### 运行控制

| 命令 | 别名 | 说明 |
|------|------|------|
| `/stop` | | 停止当前任务 |
| `/approve` | | 批准危险命令 |
| `/deny` | | 拒绝危险命令 |
| `/background` | `/bg` | 切换到后台运行 |
| `/agents` | | 查看活跃子智能体 |
| `/queue` | | 查看任务队列 |
| `/steer` | | 引导 Agent 方向 |
| `/status` | | 查看当前状态 |

### 配置与功能

| 命令 | 别名 | 说明 |
|------|------|------|
| `/model` | | 切换模型 |
| `/personality` | | 切换人格 |
| `/skills` | | 技能管理 |
| `/tools` | | 工具管理 |
| `/memory` | | 查看/管理记忆 |
| `/usage` | | Token 使用统计 |
| `/insights` | | 会话洞察 |
| `/platforms` | | 消息平台状态 |
| `/cron` | | 计划任务管理 |
| `/profile` | | 切换配置文件 |
| `/sethome` | | 设置工作目录 |

---

## TUI 界面

TUI（终端用户界面，Terminal User Interface）是 Hermes 的美化界面，使用 Ink（React）构建：

### 启动 TUI

```bash
hermes --tui

# 或通过环境变量
HERMES_TUI=1 hermes
```

### TUI 特性

- **多行输入**：支持 `Shift+Enter` 换行，`Enter` 发送
- **自动补全**：斜杠命令、文件路径自动补全
- **会话选择器**：图形化浏览和恢复历史会话
- **实时流式输出**：模型回复逐字显示
- **工具活动指示**：显示正在执行的工具
- **审批提示**：危险命令通过 TUI 弹窗确认

### TUI 进程模型

```
hermes --tui
  └─ Node.js (Ink/React)  ──stdio JSON-RPC──  Python (tui_gateway/)
       └─ 渲染 UI                                  └─ AIAgent + 工具
```

两个进程通过标准输入输出通信，Python 处理逻辑，TypeScript 处理界面。

### 仪表盘中的 TUI

```bash
# 启动 Web 仪表盘（包含嵌入式 TUI）
hermes dashboard
```

仪表盘通过 `hermes_cli/pty_bridge.py` 将 `hermes --tui` 嵌入到 Web 界面中，通过 WebSocket PTY 提供浏览器访问。

---

## 皮肤（Skin）系统

Hermes 的 CLI 界面支持主题定制（`hermes_cli/skin_engine.py`）：

```bash
# 切换皮肤
/skin ares       # 红铜战神主题
/skin mono       # 简洁灰度
/skin slate      # 冷色蓝调
/skin default    # 经典金色

# 或通过配置
hermes config set display.skin ares
```

内置皮肤：
- `default`：经典 Hermes 金色/卡哇伊风格
- `ares`：红铜战神主题，自定义 spinner 翅膀
- `mono`：纯净灰度，极简风格
- `slate`：开发者专用冷色蓝调

---

## 自动补全

CLI 使用 `prompt_toolkit` 提供 `SlashCommandCompleter` 自动补全：

- 输入 `/` 后显示所有可用斜杠命令
- 输入文件路径时自动补全文件名
- `Tab` 键触发补全

---

## 关键键绑定

| 操作 | 键绑定 |
|------|--------|
| 发送消息 | `Enter` |
| 换行（TUI） | `Shift+Enter` |
| 中断任务 | `Ctrl+C` |
| 退出 | `/exit` 或 `Ctrl+D` |
| 历史消息 | 上下方向键 |

---

## 本章小结

- `hermes` CLI 提供 20+ 主命令，涵盖设置、工具、网关、会话、系统管理
- 斜杠命令（`/new`、`/stop`、`/model` 等）在交互模式中控制会话行为
- TUI（`hermes --tui`）提供美化界面，需要 Node.js，支持多行输入和自动补全
- TUI 底层是 Ink/React（Node.js）+ JSON-RPC + Python（tui_gateway）的两进程架构
- 皮肤系统（`/skin <name>`）支持主题切换：default、ares、mono、slate
- `hermes --oneshot` 适合脚本集成，无需进入交互模式
