# 第 06 章：模型提供商

> 相关源码：`hermes_cli/providers.py`、`hermes_cli/config.py`、`run_agent.py`

---

## 提供商全景

Hermes 通过 **`hermes_cli/providers.py`** 管理 109+ 模型提供商，同时通过 [models.dev](https://models.dev) 目录动态获取最新模型列表。

| 提供商 | 类型 | 特点 |
|-------|------|------|
| **OpenRouter** | 聚合器 | 一个 Key 访问 200+ 模型，推荐新手 |
| **OpenAI** | 直连 | GPT-4o、o1、o3 系列 |
| **Anthropic** | 直连 | Claude 3.5/3.7 Sonnet、Claude 3 Opus |
| **Google Gemini** | 直连 | Gemini 2.5 Pro/Flash |
| **Ollama** | 本地 | 完全本地运行，无需 API Key |
| **Hugging Face** | 云端/本地 | 开源模型托管 |
| **NVIDIA NIM** | 云端 | GPU 加速推理 |
| **z.ai / GLM** | 国内 | 智谱 AI GLM 系列 |
| **Kimi / Moonshot** | 国内 | Moonshot AI |
| **MiniMax** | 国内 | MiniMax M 系列 |
| **Nous Portal** | Nous Research | 内部模型访问 |
| **Xiaomi MiMo** | 国内 | 小米 MiMo 系列 |
| **GitHub Copilot** | OAuth | 通过 Copilot 订阅访问 |
| **Arcee AI** | 云端 | 企业级模型 |
| **Qwen（OAuth）** | 国内 | 阿里通义千问，OAuth 认证 |

---

## 切换模型

```bash
# 交互式模型选择菜单
hermes model

# 直接设置模型
hermes config set model "anthropic/claude-3.5-sonnet"
hermes config set model "google/gemini-2.5-pro"
hermes config set model "openai/gpt-4o"
hermes config set model "ollama/llama3.2"  # 本地 Ollama
```

---

## 各提供商配置示例

### OpenRouter（推荐新手）

```bash
# ~/.hermes/.env
OPENROUTER_API_KEY=sk-or-v1-xxxxx
```

```yaml
# ~/.hermes/config.yaml
model: "anthropic/claude-3.5-sonnet"  # 通过 OpenRouter 路由到 Claude
```

OpenRouter 的优势：
- 一个 Key 访问所有主流模型
- 自动故障转移
- 统一计费
- 新注册有免费额度

### Anthropic（直连）

```bash
# ~/.hermes/.env
ANTHROPIC_API_KEY=sk-ant-xxxxx
```

```yaml
model: "claude-3-5-sonnet-20241022"
```

### OpenAI（直连）

```bash
# ~/.hermes/.env
OPENAI_API_KEY=sk-xxxxx
```

```yaml
model: "gpt-4o"
```

### Google Gemini（直连）

```bash
# ~/.hermes/.env
GOOGLE_API_KEY=xxxxx
```

```yaml
model: "gemini-2.5-pro"
```

### Ollama（本地模型，无需 API Key）

首先启动 Ollama 服务：

```bash
ollama serve
ollama pull llama3.2
```

然后配置 Hermes：

```yaml
# ~/.hermes/config.yaml
model: "ollama/llama3.2"
```

Ollama 完全本地运行，**不需要 API Key**，保护数据隐私。

### 国内提供商

**Kimi / Moonshot：**
```bash
MOONSHOT_API_KEY=sk-xxxxx
```
```yaml
model: "moonshot-v1-8k"
```

**MiniMax：**
```bash
MINIMAX_API_KEY=xxxxx
MINIMAX_GROUP_ID=xxxxx
```

**通义千问（Qwen OAuth）：**
通过 OAuth 认证，运行：
```bash
hermes setup  # 选择 Qwen OAuth 提供商，会引导 OAuth 流程
```

---

## 传输协议类型

Hermes 支持三种 API 传输协议，通过 `api_mode` 参数控制：

| 模式 | 说明 | 适用场景 |
|------|------|---------|
| `chat_completions` | OpenAI 兼容聊天接口 | 大多数提供商 |
| `anthropic_messages` | Anthropic 原生接口 | Claude 直连 |
| `codex_responses` | OpenAI Responses API | GPT-4o、Codex 等 |

通常无需手动设置——Hermes 根据提供商自动选择。

---

## OAuth 提供商

部分提供商使用 OAuth 认证而非静态 API Key：

- **GitHub Copilot**（`copilot-acp`）
- **通义千问**（`qwen-oauth`）
- **Google Gemini CLI**（`google-gemini-cli`）
- **OpenAI Codex**（`openai-codex`）

配置 OAuth 提供商：

```bash
hermes setup  # 引导向导自动处理 OAuth 流程
```

OAuth Token 存储在 `~/.hermes/.env`，Hermes 负责自动刷新。

---

## 故障转移配置（Fallback Providers）

配置主模型失败时的备用提供商：

```yaml
# ~/.hermes/config.yaml
model: "anthropic/claude-3.5-sonnet"  # 主模型

fallback_providers:
  - provider: openrouter
    model: "google/gemini-2.5-pro"    # 第一备用
  - provider: openai
    model: "gpt-4o"                   # 第二备用
```

当主模型遇到速率限制（429）或服务不可用（503）时，Hermes 自动切换到备用提供商。

---

## 速率限制与重试配置

```yaml
# ~/.hermes/config.yaml
agent:
  api_max_retries: 3   # 每个提供商最大重试次数
```

重试策略（来自 `agent/retry_utils.py`）：
- 使用**指数退避**：1s → 2s → 4s
- 只对可重试错误重试（429 速率限制、503 服务不可用）
- 不重试认证错误（401）或请求格式错误（400）

---

## 如何查看可用提供商

```bash
# 在 hermes 交互界面中
hermes model   # 显示完整的提供商和模型列表

# 或者查看配置
cat ~/.hermes/config.yaml
```

---

## 本章小结

- Hermes 支持 109+ 模型提供商，推荐新手先用 **OpenRouter**（一个 Key 通吃）
- 国内用户可选：GLM、Kimi/Moonshot、MiniMax、通义千问 等
- 本地运行（无需 API Key）：**Ollama**
- 使用 `hermes model` 交互切换，或 `hermes config set model <name>` 直接设置
- 支持故障转移（Fallback Providers）：主模型失败自动切换备用
- API 重试：指数退避，可重试错误自动重试最多 3 次
- OAuth 提供商通过 `hermes setup` 引导配置
