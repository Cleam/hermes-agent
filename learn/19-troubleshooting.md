# 第 19 章：故障排查

> 相关命令：`hermes doctor`、`hermes logs`、`hermes version`

---

## 快速诊断

**遇到任何问题，先运行：**

```bash
hermes doctor    # 全面诊断
hermes version   # 确认版本
hermes logs --level error  # 查看最近错误
```

---

## 安装类问题

### 问题 1：安装后 `hermes` 命令找不到

**现象**：
```
bash: hermes: command not found
```

**可能原因**：PATH 未更新

**排查步骤**：
```bash
# 检查 hermes 是否安装
which hermes
ls ~/.local/bin/hermes  # 或者
ls ~/bin/hermes

# 检查 PATH
echo $PATH
```

**解决方案**：
```bash
# 重新加载 Shell 配置
source ~/.bashrc   # Bash
source ~/.zshrc    # Zsh

# 如果还不行，手动添加 PATH
export PATH="$HOME/.local/bin:$PATH"
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
```

---

### 问题 2：Python 版本不满足要求

**现象**：
```
Error: Python 3.10 found, but 3.11+ required
```

**解决方案**：
```bash
# 使用 pyenv 安装 Python 3.12
pyenv install 3.12.0
pyenv local 3.12.0

# 或重新用 uv 创建虚拟环境
uv venv .venv --python 3.12
source .venv/bin/activate
uv pip install -e ".[all]"
```

---

### 问题 3：Node.js 版本问题（TUI 无法启动）

**现象**：
```
error: hermes --tui requires Node.js 18+
Current version: v16.x.x
```

**解决方案**：
```bash
# 使用 nvm 升级 Node.js
nvm install 20
nvm use 20
nvm alias default 20

# 或系统安装（Ubuntu）
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
```

**注意**：不使用 TUI 功能时，Node.js 不是必须的。纯 CLI 模式（`hermes` 不带 `--tui`）不需要 Node.js。

---

### 问题 4：依赖安装失败

**现象**：
```
ERROR: Could not install packages due to an OSError
```

**解决方案**：
```bash
# 清理缓存重试
uv cache clean
uv pip install -e ".[all]" --no-cache-dir

# 检查磁盘空间
df -h

# 如果是网络问题，使用国内镜像（uv 支持）
uv pip install -e ".[all]" --index-url https://pypi.tuna.tsinghua.edu.cn/simple/
```

---

## 模型配置问题

### 问题 5：API Key 无效

**现象**：
```
Error: Invalid API key. Please check your API key in ~/.hermes/.env
```

**排查步骤**：
```bash
# 检查 .env 文件
cat ~/.hermes/.env

# 检查变量名是否正确
# OpenAI: OPENAI_API_KEY
# Anthropic: ANTHROPIC_API_KEY
# OpenRouter: OPENROUTER_API_KEY
```

**解决方案**：
```bash
# 重新运行配置向导
hermes setup

# 或直接编辑
nano ~/.hermes/.env
```

---

### 问题 6：模型请求超时

**现象**：
```
Error: Request timed out after 60 seconds
```

**解决方案**：
```yaml
# ~/.hermes/config.yaml - 增加超时
agent:
  gateway_timeout: 3600  # 秒

# 或配置备用提供商
fallback_providers:
  - provider: openrouter
    model: "google/gemini-2.5-pro"
```

---

### 问题 7：速率限制（Rate Limit）

**现象**：
```
Error: 429 Too Many Requests
```

**解决方案**：
- Hermes 会自动重试（指数退避，最多 3 次）
- 如果频繁触发，配置多个提供商作为 Fallback
- 考虑切换到 OpenRouter（它会自动分流到多个提供商）

---

## 运行时问题

### 问题 8：CLI 启动失败

**现象**：启动 `hermes` 后立即退出，或显示错误

**排查步骤**：
```bash
# 查看详细错误信息
hermes --help  # 至少应该显示帮助
hermes doctor  # 诊断

# 查看日志
cat ~/.hermes/logs/errors.log | tail -50
```

**常见原因**：
- `config.yaml` 语法错误：用 `hermes config show` 验证
- 虚拟环境未激活：`source .venv/bin/activate`

---

### 问题 9：TUI 启动失败

**现象**：
```
hermes --tui
Error: Failed to start TUI
```

**排查步骤**：
```bash
# 检查 Node.js
node --version    # 需要 18+
npm --version

# 检查 ui-tui 依赖
cd ui-tui
npm install
npm run build
```

**解决方案**：
```bash
# 重新安装 TUI 依赖
cd ui-tui
rm -rf node_modules
npm install

# 使用普通 CLI 模式作为临时替代
hermes  # 不带 --tui
```

---

### 问题 10：工具调用失败

**现象**：Agent 尝试调用工具但失败，返回错误

**排查步骤**：
```bash
# 检查工具可用性
hermes doctor

# 查看具体错误
hermes logs --follow  # 实时查看工具调用日志
```

**常见原因**：

| 工具 | 常见问题 | 解决方案 |
|------|---------|---------|
| `browser_*` | Playwright 未安装 | `playwright install chromium` |
| `web_search` | 搜索 API Key 缺失 | 配置搜索引擎 API Key |
| `terminal` | Docker 后端但 Docker 未启动 | `sudo systemctl start docker` |
| `ha_*` | HOME_ASSISTANT_URL 未设置 | 配置 Home Assistant 连接 |

---

### 问题 11：MCP 服务器连接失败

**现象**：
```
Error: Failed to connect to MCP server 'filesystem'
```

**排查步骤**：
```bash
# 手动测试 MCP 服务器
npx -y @modelcontextprotocol/server-filesystem /home/user

# 检查 MCP 配置
cat ~/.hermes/config.yaml | grep -A 20 mcp_servers
```

**相关源码**：`tools/mcp_tool.py`

---

### 问题 12：记忆文件损坏

**现象**：Hermes 每次都说不记得之前的信息

**排查步骤**：
```bash
# 检查记忆文件
cat ~/.hermes/memories/MEMORY.md
cat ~/.hermes/memories/USER.md

# 检查文件权限
ls -la ~/.hermes/memories/
```

**解决方案**：
```bash
# 如果文件损坏，重置
echo "" > ~/.hermes/memories/MEMORY.md
echo "" > ~/.hermes/memories/USER.md

# 确保文件可写
chmod 644 ~/.hermes/memories/*.md
```

---

## 日志位置速查

```
~/.hermes/logs/agent.log    - 主日志（INFO+）
~/.hermes/logs/errors.log   - 错误日志（WARNING+）
~/.hermes/logs/gateway.log  - 网关日志
```

查看日志：
```bash
hermes logs                   # 最近日志（通过 Hermes CLI）
hermes logs --follow          # 实时跟踪
hermes logs --level error     # 只看错误
cat ~/.hermes/logs/agent.log | grep "ERROR" | tail -20
```

---

## 导出诊断信息

当向社区寻求帮助时，先运行：

```bash
hermes doctor > diagnosis.txt 2>&1
hermes version >> diagnosis.txt
cat ~/.hermes/logs/errors.log | tail -100 >> diagnosis.txt
echo "---config---" >> diagnosis.txt
hermes config show >> diagnosis.txt
```

> ⚠️ 分享诊断信息前，确保删除其中的 API 密钥等敏感信息！

---

## 本章小结

- **第一步**：运行 `hermes doctor` 做全面诊断
- **查日志**：`hermes logs --level error` 看错误，`hermes logs --follow` 实时跟踪
- 常见安装问题：PATH 未更新、Python 版本不足、Node.js 版本不足
- 常见运行问题：API Key 无效、速率限制（自动重试）、工具依赖缺失
- MCP 问题：手动测试服务器进程是否能正常启动
- 记忆问题：检查 `~/.hermes/memories/` 下的文件权限和内容
- 寻求帮助时：用 `hermes doctor` 导出诊断，**删除敏感信息**后分享
