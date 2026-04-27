# 第 10 章：上下文管理

> 相关源码：`agent/context_compressor.py`、`agent/prompt_builder.py`、`run_agent.py`

---

## 上下文窗口是什么

把上下文窗口（Context Window）想象成**会议桌上能摆放的资料**。

桌子有固定大小（比如 200,000 个 Token）。每轮对话、每次工具调用的结果都会占用桌面空间。当桌面快满的时候，你需要把一些旧资料"归档"——这就是上下文压缩（Context Compression）。

不同模型的上下文窗口大小：
- GPT-4o：128k tokens
- Claude 3.5 Sonnet：200k tokens
- Gemini 2.5 Pro：1M tokens
- Ollama/本地模型：通常 8k~32k tokens

---

## 自动压缩机制

Hermes 在 `agent/context_compressor.py` 中实现了自动压缩：

```python
# agent/context_compressor.py 核心逻辑（简化）
class ContextCompressor:
    def should_compress(self, messages: list, model: str) -> bool:
        """检查是否需要压缩"""
        usage_ratio = self.estimate_token_count(messages) / self.get_context_limit(model)
        return usage_ratio >= self.threshold  # 默认 0.85（85%）
    
    def compress(self, messages: list, focus: str = None) -> list:
        """
        压缩策略：
        1. 保留第一条系统消息（始终保留）
        2. 保留最近几轮对话（始终保留）
        3. 将中间的历史轮次总结为摘要
        """
        # 使用摘要模型（compression.summary_model）生成中间历史的摘要
        summary = self.summarize_middle_turns(messages, focus)
        return [system_msg, summary_msg, *recent_msgs]
```

### 压缩配置

```yaml
# ~/.hermes/config.yaml
compression:
  enabled: true
  threshold: 0.85        # 触发压缩的使用率阈值（85%）
  # summary_model: "google/gemini-2.5-flash"  # 用于生成摘要的模型
```

`summary_model` 是专门用于压缩摘要的模型，建议使用快速、便宜的模型（如 Gemini Flash）而不是主模型，节省成本。

---

## 手动压缩

不想等到自动触发，可以手动压缩：

```
/compress

# 带焦点压缩（保留与特定主题相关的细节）
/compress Python 性能优化

# 查看当前 Token 使用量
/usage
```

---

## /usage 命令

查看详细的 Token 使用统计：

```
/usage
```

输出示例：
```
📊 Token 使用统计
上下文窗口：128,000 tokens
当前使用：45,231 tokens (35.3%)
系统提示：3,200 tokens
对话历史：38,000 tokens
工具结果：4,031 tokens

估计剩余：82,769 tokens
```

---

## SOUL.md：持久化人格文件

`~/.hermes/SOUL.md` 是 Agent 的"灵魂文件"——定义其人格、行为边界和持久指令：

```markdown
# ~/.hermes/SOUL.md

你是一位专注于 Python 和云原生技术的高级工程师助手。

## 行为准则
- 回答时先给出简洁结论，再展开细节
- 代码示例优先使用 Python 3.12+ 语法
- 遇到安全风险时主动提醒
- 不执行破坏性命令（rm -rf / 等）除非明确确认

## 专业背景
- 精通：Python、Go、Kubernetes、AWS
- 熟悉：React、PostgreSQL、Redis
- 当前项目：微服务架构改造

## 输出偏好
- 用中文回答，代码注释也用中文
- 错误信息和日志保持英文原文
- 复杂主题使用 Markdown 表格和列表
```

SOUL.md 的内容会永久包含在系统提示中，不受压缩影响。

---

## 项目上下文文件

Hermes 自动读取当前目录下的上下文文件：

| 文件名 | 作用 |
|--------|------|
| `AGENTS.md` | 项目特定的 Agent 指令（仓库级别） |
| `HERMES.md` | 项目的 Hermes 专属配置 |
| `CLAUDE.md` | 兼容 Claude Code 格式 |

这些文件通常放在项目根目录：

```markdown
# AGENTS.md（放在项目根目录）

## 项目说明
这是一个 FastAPI + PostgreSQL 的 REST API 项目。

## 代码规范
- 使用 Black 格式化，行宽 88
- 测试使用 pytest，覆盖率要求 80%+
- 所有 API 端点必须有 OpenAPI 文档

## 常用命令
- 启动开发服务器：`uvicorn app.main:app --reload`
- 运行测试：`pytest tests/ -v`
- 数据库迁移：`alembic upgrade head`

## 目录结构
app/
├── api/        # API 路由
├── models/     # 数据库模型
├── schemas/    # Pydantic 模型
└── main.py     # 应用入口
```

每次在该项目目录启动 Hermes，这些指令就自动生效——不需要每次重新解释项目背景。

---

## 提示缓存保护策略

提示缓存（Prompt Caching）是 Hermes 节省 Token 成本的重要机制。Anthropic、OpenAI 等主要提供商都支持缓存相同的前缀 Token，大幅减少费用。

**关键规则**：系统提示一旦建立，**不在会话中途修改**。

这意味着：
- 技能安装后默认**下次会话生效**，不立即改变当前系统提示
- 记忆更新写入文件，**下次会话**才会注入新的记忆内容
- 如果需要立即生效：使用 `--now` 标志（但会破坏缓存，增加 Token 消耗）

```
# 安装技能立即生效（破坏缓存，慎用）
hermes skills install my-skill --now
```

---

## 上下文窗口使用量可视化

```
[====================..........] 68% 使用
当前：87,040 / 128,000 tokens
压缩阈值：108,800 tokens（85%）

距离自动压缩还剩：21,760 tokens
```

---

## 本章小结

- 上下文窗口 = 会议桌，有固定大小（取决于模型）
- **自动压缩**在使用率达到 85%（可配置）时触发，保留首尾、摘要中间
- **手动压缩**：`/compress [焦点话题]`
- **`/usage`** 查看 Token 使用详情
- **`SOUL.md`**（`~/.hermes/SOUL.md`）定义 Agent 持久人格和指令
- **`AGENTS.md`** / **`HERMES.md`** 定义项目级上下文
- **提示缓存保护**：系统提示不在会话中途修改，技能/记忆变更默认下次生效
