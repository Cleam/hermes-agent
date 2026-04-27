# 第 05 章：配置系统

> 相关源码：`hermes_cli/config.py`、`cli.py`（`load_cli_config`）、`gateway/run.py`、`hermes_constants.py`

---

## 配置文件概览

Hermes 使用两个配置文件，职责严格分开：

| 文件 | 用途 | 权限 |
|------|------|------|
| `~/.hermes/config.yaml` | 非敏感设置（模型、工具集、显示等） | 默认权限 |
| `~/.hermes/.env` | **仅** API 密钥和密码 | 自动 chmod 600 |

> ⚠️ **安全规则**：永远不要把 API 密钥放进 `config.yaml`，只用 `.env`。

---

## config.yaml 结构（DEFAULT_CONFIG）

```yaml
# 完整配置项参考（来自 hermes_cli/config.py 的 DEFAULT_CONFIG）

# 当前使用的模型（空字符串表示由提供商决定）
model: ""

# 启用的工具集列表
toolsets:
  - hermes-cli

# Agent 行为配置
agent:
  max_turns: 90              # 最大工具调用轮次
  gateway_timeout: 1800      # 消息网关超时（秒）
  api_max_retries: 3         # API 重试次数
  tool_use_enforcement: "auto"  # auto | required | none

# 终端后端配置
terminal:
  backend: "local"           # local | docker | ssh | modal | singularity | daytona
  timeout: 180               # 命令执行超时（秒）
  # cwd: "/path/to/dir"      # 工作目录（消息网关模式使用）

# 浏览器配置
browser:
  headless: true

# 上下文压缩
compression:
  enabled: true
  threshold: 0.85            # 触发压缩的上下文使用率阈值（85%）
  # summary_model: "google/gemini-2.5-flash"  # 压缩摘要模型

# 记忆系统
memory:
  provider: builtin          # builtin | honcho | mem0 | supermemory | byterover
                             # | hindsight | holographic | openviking | retaindb

# 显示设置
display:
  skin: default              # default | ares | mono | slate | 自定义皮肤名称

# MCP 服务器（可选）
# mcp_servers:
#   filesystem:
#     command: npx
#     args: ["-y", "@modelcontextprotocol/server-filesystem", "/home/user"]
```

---

## .env 文件格式

```bash
# ~/.hermes/.env
# 只放 API 密钥和密码，不放其他配置

OPENROUTER_API_KEY=sk-or-v1-xxxxx
OPENAI_API_KEY=sk-xxxxx
ANTHROPIC_API_KEY=sk-ant-xxxxx
GOOGLE_API_KEY=xxxxx

# 消息平台 Token
TELEGRAM_BOT_TOKEN=xxxxx:xxxxx
DISCORD_BOT_TOKEN=xxxxx
SLACK_BOT_TOKEN=xoxb-xxxxx
```

---

## 三种配置加载器

Hermes 内部有**三套**配置加载逻辑，分别用于不同场景。理解这个区分很重要：

### 1. load_cli_config()（CLI 模式专用）

```python
# cli.py 中调用
config = load_cli_config()
```

- 合并硬编码的 CLI 默认值 + 用户 `config.yaml`
- 用于 `hermes`（交互式聊天）

### 2. load_config()（大多数子命令）

```python
# hermes_cli/config.py
from hermes_cli.config import load_config
config = load_config()
```

- 基于 `DEFAULT_CONFIG` 深度合并用户 `config.yaml`
- 用于 `hermes tools`、`hermes setup`、`hermes model`、大部分 CLI 子命令

### 3. 直接 YAML 加载（网关运行时）

```python
# gateway/run.py
with open(config_path) as f:
    config = yaml.safe_load(f)
```

- 网关直接读取原始 YAML
- 不经过 `DEFAULT_CONFIG` 的深度合并

> 💡 如果你发现某个配置项在 CLI 中生效但网关中不生效（或反之），很可能是因为你在错误的加载器管辖范围内寻找问题。

---

## 常用配置操作

```bash
# 查看当前配置（合并后的完整配置）
hermes config show

# 设置单个配置项（支持点记法）
hermes config set model "anthropic/claude-3.5-sonnet"
hermes config set agent.max_turns 120
hermes config set display.skin ares
hermes config set terminal.backend docker

# 打开编辑器编辑配置文件
hermes config edit

# 重置为默认值（危险！）
# 手动删除 ~/.hermes/config.yaml 即可
```

---

## 多配置文件（Profiles）

Hermes 支持多个完全隔离的配置文件实例——适合"工作"和"个人"分离，或管理多个 Telegram 机器人：

```bash
# 使用指定配置文件启动
hermes -p work

# 配置文件有独立的 HERMES_HOME 目录
# ~/.hermes/profiles/work/config.yaml
# ~/.hermes/profiles/work/.env
# ~/.hermes/profiles/work/memories/
# ...

# 查看当前配置文件
hermes profile list
hermes profile status

# 切换工作目录
hermes -p work sethome /path/to/work/dir
```

配置文件切换的实现原理：

```python
# hermes_cli/main.py
def _apply_profile_override(profile_name: str):
    """在所有模块导入前设置 HERMES_HOME 环境变量"""
    hermes_home = Path.home() / ".hermes" / "profiles" / profile_name
    os.environ["HERMES_HOME"] = str(hermes_home)
```

所有 `get_hermes_home()` 调用都读取 `HERMES_HOME` 环境变量，因此自动隔离。

---

## 提供商配置

可以在 `config.yaml` 中配置多个提供商和故障转移：

```yaml
# 主模型
model: "anthropic/claude-3.5-sonnet"

# 故障转移提供商列表
fallback_providers:
  - provider: openrouter
    model: "google/gemini-2.5-pro"
  - provider: openai
    model: "gpt-4o"
```

---

## 工具集配置

控制 Agent 可以使用哪些工具：

```yaml
toolsets:
  - hermes-cli     # 核心工具集（web, terminal, file, browser 等）
  - code-execution  # 代码执行工具
```

禁用特定工具集（通过 CLI）：

```bash
hermes tools --disable browser
```

---

## 本章小结

- Hermes 使用两个配置文件：`config.yaml`（设置）和 `.env`（密钥）
- 密钥**只放** `.env`，不放 `config.yaml`
- 三种配置加载器：`load_cli_config()`（CLI）、`load_config()`（子命令）、直接 YAML（网关）
- 使用 `hermes config set <key> <value>` 修改配置
- 多配置文件（Profiles）通过 `hermes -p <name>` 启动，每个 Profile 有独立的 `~/.hermes/profiles/<name>/` 目录
- 配置版本由 `_config_version` 管理，Hermes 会自动迁移旧版本配置
