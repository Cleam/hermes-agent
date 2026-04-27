# 第 11 章：MCP 集成

> 相关源码：`tools/mcp_tool.py`、`mcp_serve.py`、`hermes_cli/mcp_config.py`

---

## 什么是 MCP

MCP（Model Context Protocol，模型上下文协议）是一个开放协议，允许 AI 模型与外部工具和数据源以标准化方式集成。

你可以把 MCP 理解为 AI 工具调用的"USB 接口"——任何实现了 MCP 协议的服务，都可以即插即用地接入 Hermes。

**主要用途**：
- 连接文件系统工具（精细的文件权限控制）
- 连接 GitHub、Jira、数据库等服务
- 接入企业内部知识库和 API
- 扩展 Hermes 能力而无需修改 Python 代码

---

## 配置 MCP 服务器

MCP 服务器配置在 `~/.hermes/config.yaml` 的 `mcp_servers` 节下：

### stdio 传输（本地进程）

```yaml
# ~/.hermes/config.yaml
mcp_servers:
  # 文件系统工具（Node.js 包）
  filesystem:
    command: npx
    args:
      - "-y"
      - "@modelcontextprotocol/server-filesystem"
      - "/home/user/projects"    # 允许访问的目录
    timeout: 30                  # 连接超时（秒）

  # GitHub 集成
  github:
    command: npx
    args:
      - "-y"
      - "@modelcontextprotocol/server-github"
    env:
      GITHUB_PERSONAL_ACCESS_TOKEN: "${GITHUB_TOKEN}"  # 从环境变量读取
```

### HTTP/StreamableHTTP 传输（远程服务）

```yaml
# ~/.hermes/config.yaml
mcp_servers:
  # 远程 MCP 服务器（HTTP）
  remote_api:
    url: "https://mcp.example.com/api"
    timeout: 60

  # 本地 HTTP MCP 服务
  local_server:
    url: "http://localhost:3000/mcp"
```

---

## 常用 MCP 服务器示例

### 文件系统工具

```yaml
mcp_servers:
  filesystem:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-filesystem", "/home/user"]
```

提供的工具：`read_file`、`write_file`、`list_directory`、`create_directory`、`move_file`、`search_files`

### GitHub 集成

```yaml
mcp_servers:
  github:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-github"]
    env:
      GITHUB_PERSONAL_ACCESS_TOKEN: "ghp_xxxxx"
```

提供的工具：`create_issue`、`create_pull_request`、`search_code`、`get_file_contents` 等

### PostgreSQL 数据库

```yaml
mcp_servers:
  postgres:
    command: npx
    args:
      - "-y"
      - "@modelcontextprotocol/server-postgres"
      - "postgresql://user:pass@localhost/mydb"
```

提供的工具：`query`、`list_tables`、`describe_table`

### Brave 搜索

```yaml
mcp_servers:
  brave_search:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-brave-search"]
    env:
      BRAVE_API_KEY: "BSA_xxxxx"
```

---

## MCP 工作原理

```mermaid
sequenceDiagram
    participant Agent as AIAgent
    participant MCP_TOOL as tools/mcp_tool.py
    participant SERVER as MCP 服务器进程
    participant EXT as 外部服务

    Note over MCP_TOOL,SERVER: 启动时建立连接
    MCP_TOOL->>SERVER: 启动进程 / 建立 HTTP 连接
    SERVER-->>MCP_TOOL: 返回工具列表 (tools/list)
    MCP_TOOL->>MCP_TOOL: 注册 MCP 工具到 registry

    Note over Agent,SERVER: 运行时工具调用
    Agent->>MCP_TOOL: 调用 MCP 工具
    MCP_TOOL->>SERVER: tools/call 请求
    SERVER->>EXT: 实际操作
    EXT-->>SERVER: 结果
    SERVER-->>MCP_TOOL: tool_result
    MCP_TOOL-->>Agent: JSON 字符串结果
```

---

## 自动重连与容错

`tools/mcp_tool.py` 实现了健壮的连接管理：

- **自动重连**：连接断开时自动尝试重连
- **指数退避**：重连间隔 1s、2s、4s、8s、16s（最多 5 次）
- **工具缓存**：重连后重新获取工具列表
- **超时控制**：每个 MCP 服务器可单独配置超时

```yaml
mcp_servers:
  slow_server:
    url: "https://slow-mcp.example.com"
    timeout: 120  # 慢服务器给更长超时
```

---

## Sampling（采样）支持

高级功能：MCP 服务器可以**请求 LLM 完成**（Sampling）。

这意味着 MCP 服务器可以让 Hermes 的 LLM 参与服务器内部的决策，而不只是单向工具调用。`tools/mcp_tool.py` 实现了完整的 Sampling 支持。

---

## Hermes 作为 MCP 服务端

Hermes 不只能作为 MCP 客户端（连接别的 MCP 服务器），还能作为 MCP 服务端让别的工具连接它：

```bash
# 启动 Hermes MCP 服务
python mcp_serve.py
```

这让 VS Code Copilot、Zed、JetBrains 等支持 MCP 的编辑器可以将 Hermes 作为工具后端使用（通过 `acp_adapter/`）。

---

## 管理 MCP 配置

```bash
# 通过 Hermes CLI 管理 MCP 服务器配置
hermes config edit  # 直接编辑配置文件

# 或者手动编辑
nano ~/.hermes/config.yaml
```

Hermes 在启动时自动连接所有配置的 MCP 服务器，并将其工具注册到工具注册表。

---

## 安全注意事项

- **只连接信任的 MCP 服务器**——MCP 服务器能看到你的工具调用内容
- 使用环境变量（`env:` 配置）传递密钥，不要硬编码在 `config.yaml` 中
- 本地 stdio 服务器比远程 HTTP 服务器风险更低
- 对于敏感操作（数据库写入等），考虑使用 Docker 后端隔离

---

## 本章小结

- MCP 是 AI 工具集成的开放标准，Hermes 完整支持 stdio 和 HTTP 传输
- 在 `~/.hermes/config.yaml` 的 `mcp_servers:` 下配置 MCP 服务器
- 支持文件系统、GitHub、PostgreSQL、Brave 搜索等 100+ 官方 MCP 服务器
- 自动重连 + 指数退避，确保连接稳定性
- 支持 Sampling：MCP 服务器可以请求 LLM 参与决策
- Hermes 也能作为 MCP 服务端（`mcp_serve.py`），供编辑器插件连接
- 只连接信任的 MCP 服务器——它们能看到工具调用内容
