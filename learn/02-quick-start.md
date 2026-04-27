# 第 02 章：快速开始

> 相关源码：`cli.py`、`hermes_cli/main.py`、`hermes_cli/commands.py`

---

## 第一次启动

安装并完成 `hermes setup` 配置后，直接输入：

```bash
hermes
```

你会看到欢迎界面（具体样式取决于皮肤配置）。默认启动交互式 CLI 模式，等待你的输入。

如果想要更美观的终端界面（需要 Node.js）：

```bash
hermes --tui
```

---

## 配置第一个模型

如果跳过了 `hermes setup`，可以单独配置模型：

```bash
hermes model
```

这会显示交互式菜单，引导你：
1. 选择提供商（建议新手选 **OpenRouter**，一个 API Key 访问 200+ 模型）
2. 输入 API Key（存储在 `~/.hermes/.env`，自动 chmod 600）
3. 选择默认模型（建议 `anthropic/claude-3.5-sonnet` 或 `google/gemini-2.5-pro`）

**OpenRouter 获取免费额度**：访问 [openrouter.ai](https://openrouter.ai) 注册即可获得免费额度。

---

## 第一次对话

启动 Hermes 后，直接输入：

```
你好！你能帮我做什么？
```

Hermes 会介绍自己的能力。然后尝试一个更具体的请求：

```
帮我查一下今天的天气（北京）
```

Hermes 会自动调用 `web_search` 工具搜索结果，并返回格式化的天气信息。

---

## 第一次工具调用

### 终端工具（Terminal Tool）

```
帮我列出当前目录下的文件，并告诉我最大的文件是哪个
```

Hermes 会执行类似：
```bash
ls -lhS
```
并分析输出，给出回答。

### 文件工具（File Tool）

```
帮我创建一个 Python 脚本，计算 1 到 100 的质数，保存为 primes.py
```

Hermes 会使用 `write_file` 工具创建文件。

### 网页工具（Web Tool）

```
帮我搜索 Python 异步编程的最佳实践，总结成要点
```

Hermes 会使用 `web_search` + `web_extract` 工具，汇总信息。

---

## 第一次查看日志

在另一个终端窗口：

```bash
# 查看实时日志
hermes logs --follow

# 只看错误
hermes logs --level error

# 查看特定会话的日志
hermes logs --session <session_id>
```

日志文件位置：
- `~/.hermes/logs/agent.log`（INFO 级别以上）
- `~/.hermes/logs/errors.log`（WARNING 级别以上）

---

## 第一次中断任务

当 Agent 在执行耗时任务时，如果想中断：

**CLI 模式**：按 `Ctrl+C`

**消息平台（网关模式）**：发送 `/stop` 命令

```
/stop
```

如果需要等待当前工具执行完再停止（优雅停止）：

```
/stop
```

Agent 会完成当前工具调用后停止，不会中途截断命令。

---

## 基本斜杠命令

在对话中，可以使用斜杠命令控制 Agent 行为：

```bash
# 开始新会话（清空上下文）
/new

# 重试上一条消息
/retry

# 手动压缩上下文（节省 Token）
/compress

# 查看当前状态（模型、上下文使用量等）
/status

# 查看 Token 使用统计
/usage

# 查看命令帮助
/help
```

---

## 会话搜索

Hermes 使用 SQLite FTS5 索引所有历史会话，可以快速搜索：

```bash
# 通过 CLI 命令浏览会话
hermes sessions browse

# 在对话中搜索历史会话
你：搜索我之前关于 Docker 部署的对话
```

Agent 会调用 `session_search` 工具搜索历史会话并汇总相关信息。

---

## 一次性模式（Oneshot Mode）

如果不需要交互式对话，可以直接传入问题并获取结果：

```bash
hermes --oneshot "帮我生成一个随机密码，长度 20 位，包含大小写字母和特殊字符"
```

或通过管道：

```bash
echo "分析这段代码的时间复杂度：$(cat sort.py)" | hermes --oneshot
```

---

## 查看 Agent 状态

```bash
# 查看当前 Agent 状态（是否在运行、当前会话等）
hermes status
```

---

## 第一个完整工作流示例

下面是一个完整的使用示例，展示 Hermes 如何处理一个真实任务：

**你**：
```
分析我的系统性能，找出最消耗资源的进程，并生成一份报告保存到 ~/report.md
```

**Hermes 内部执行流程**：

1. 调用 `terminal` 工具执行 `top -bn1 | head -20`（查看进程）
2. 调用 `terminal` 工具执行 `free -h`（查看内存）
3. 调用 `terminal` 工具执行 `df -h`（查看磁盘）
4. 分析所有数据
5. 调用 `write_file` 工具生成 Markdown 报告
6. 返回摘要

**你**：
```
/save  # 保存这个会话，方便以后查阅
```

---

## 配置文件概览

首次使用时，可以查看默认配置：

```bash
# 查看当前配置
hermes config show

# 修改单个配置项
hermes config set display.skin ares

# 打开编辑器编辑配置
hermes config edit
```

---

## 本章小结

- 启动：`hermes`（CLI）或 `hermes --tui`（TUI 界面）
- 模型配置：`hermes model`，建议新手先试 OpenRouter
- 工具调用是自动的——直接描述任务，Hermes 自行决定用哪些工具
- Ctrl+C（CLI）或 `/stop`（消息平台）可以中断任务
- 斜杠命令（/new、/retry、/compress、/status）用于控制会话
- `hermes sessions browse` 和 `session_search` 工具用于搜索历史
- `--oneshot` 参数适合脚本集成
- 详细配置见 [第 05 章：配置系统](05-configuration.md)
