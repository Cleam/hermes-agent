# 第 01 章：安装指南

> 相关源码：`scripts/install.sh`、`setup-hermes.sh`、`pyproject.toml`、`package.json`

---

## 系统要求

| 要求 | 说明 |
|------|------|
| 操作系统 | Linux、macOS、WSL2（Windows）、Termux（Android） |
| Python | **3.11+**（推荐 3.12） |
| Node.js | 18+（TUI 模式 `hermes --tui` 需要） |
| 包管理器 | `uv`（推荐）或 `pip` |
| 磁盘空间 | 约 500MB（含依赖） |

> 💡 **提示**：Termux（Android）也受支持，安装前参考 `constraints-termux.txt` 中的依赖约束。

---

## 方式一：一键安装（推荐普通用户）

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

脚本会自动：
1. 检测操作系统和 Python 版本
2. 安装 `uv`（如未安装）
3. 创建虚拟环境并安装 Hermes
4. 配置 PATH

安装完成后，重新加载 Shell：

```bash
source ~/.bashrc   # Bash 用户
# 或
source ~/.zshrc    # Zsh 用户
```

验证安装：

```bash
hermes version
hermes doctor
```

---

## 方式二：贡献者安装（源码安装）

适合希望修改源码或提交 PR 的开发者：

```bash
# 克隆仓库
git clone https://github.com/NousResearch/hermes-agent.git
cd hermes-agent

# 使用官方安装脚本（处理所有依赖）
./setup-hermes.sh
```

`setup-hermes.sh` 会：
1. 创建 `.venv` 虚拟环境
2. 安装所有 Python 依赖（含开发依赖）
3. 安装 Node.js 依赖（`ui-tui/` 目录）
4. 将 `hermes` 命令链接到 PATH

---

## 方式三：手动安装（高级用户）

如果你需要精细控制安装过程：

```bash
# 1. 克隆仓库
git clone https://github.com/NousResearch/hermes-agent.git
cd hermes-agent

# 2. 创建虚拟环境（指定 Python 3.11）
uv venv venv --python 3.11
source venv/bin/activate   # Linux/macOS
# 或 venv\Scripts\activate  # WSL2

# 3. 安装 Python 依赖
uv pip install -e ".[all,dev]"

# 4. 安装 Node.js 依赖（可选，TUI 需要）
cd ui-tui
npm install
cd ..

# 5. 验证
python -m hermes_cli.main version
```

---

## 首次配置

安装完成后，运行引导向导：

```bash
hermes setup
```

向导会引导你：
1. 选择模型提供商（OpenAI / Anthropic / Google / Ollama 等）
2. 输入 API 密钥（保存到 `~/.hermes/.env`）
3. 选择默认模型
4. 配置基本偏好

也可以单独配置模型：

```bash
hermes model
```

---

## 验证安装

```bash
# 检查版本
hermes version

# 全面诊断（检查配置、API 连通性、工具可用性）
hermes doctor

# 查看帮助
hermes --help
```

`hermes doctor` 的输出示例：
```
✅ Python 3.12.0
✅ Node.js 20.11.0 (TUI 可用)
✅ ~/.hermes/config.yaml 存在
✅ ~/.hermes/.env 存在
✅ OpenRouter API 连通
⚠️  browser 工具需要 Playwright：运行 playwright install
```

---

## 安装 Playwright（浏览器自动化）

如果需要浏览器工具（`browser_navigate`、`browser_click` 等）：

```bash
# 在虚拟环境中安装 Playwright
playwright install chromium
```

---

## 目录结构说明

安装后，Hermes 使用 `~/.hermes/` 作为数据目录：

```
~/.hermes/
├── config.yaml          # 主配置文件（非敏感设置）
├── .env                 # API 密钥（chmod 600 自动应用）
├── memories/
│   ├── MEMORY.md        # Agent 学到的通用知识
│   └── USER.md          # 关于你的个人信息
├── skills/              # 用户技能（自动生成 + 手动添加）
├── logs/
│   ├── agent.log        # 主日志（INFO 级别以上）
│   ├── errors.log       # 错误日志（WARNING 级别以上）
│   └── gateway.log      # 消息网关日志
├── sessions/            # 会话历史（SQLite）
└── profiles/            # 多配置文件目录（可选）
```

---

## 卸载

```bash
hermes uninstall
```

这会删除 `hermes` 命令，但**保留** `~/.hermes/` 目录（你的配置和记忆）。如果需要完全清除：

```bash
hermes uninstall
rm -rf ~/.hermes/
```

---

## 常见安装问题

### Python 版本不满足要求

```
错误: Python 3.10 found, but 3.11+ required
```

解决：
```bash
# 使用 pyenv 安装指定版本
pyenv install 3.12.0
pyenv local 3.12.0

# 或使用 uv 指定版本
uv venv venv --python 3.12
```

### Node.js 未安装（TUI 无法启动）

```
错误: hermes --tui requires Node.js 18+
```

解决：
```bash
# Ubuntu/Debian
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# macOS
brew install node
```

注意：如果不使用 TUI 功能，Node.js 不是必须的。`hermes`（纯 CLI 模式）不需要 Node.js。

### uv 未找到

```
错误: uv: command not found
```

解决：
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.bashrc
```

### 权限错误（Permission denied）

```bash
# 检查虚拟环境目录权限
ls -la ~/.hermes/
# 确保 .env 权限正确
chmod 600 ~/.hermes/.env
```

### WSL2 特别说明

在 WSL2 中，确保使用 Linux 文件系统路径（`~/`）而不是 Windows 路径。如果遇到行尾符问题：

```bash
# 安装前设置 git 配置
git config --global core.autocrlf false
```

### Termux（Android）安装

```bash
pkg install python nodejs git
pip install uv
# 安装时使用约束文件
uv pip install -e "." -c constraints-termux.txt
```

---

## 本章小结

- Hermes 支持 Linux / macOS / WSL2 / Termux，需要 Python 3.11+
- **普通用户**：一键脚本 `curl ... | bash`
- **贡献者**：克隆仓库 + `./setup-hermes.sh`
- **高级用户**：`uv venv` + `uv pip install -e ".[all,dev]"`
- 安装后必须运行 `hermes setup` 配置 API 密钥
- 使用 `hermes doctor` 诊断安装问题
- 数据目录在 `~/.hermes/`，`hermes uninstall` 只删除命令，不删除数据
