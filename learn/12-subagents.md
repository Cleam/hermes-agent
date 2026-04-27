# 第 12 章：子智能体与并行处理

> 相关源码：`tools/delegate_tool.py`、`tools/code_execution_tool.py`、`batch_runner.py`、`tools/mixture_of_agents_tool.py`

---

## 为什么需要子智能体

单个 Agent 是串行的——它一次只能做一件事，工具调用按顺序执行。

对于需要**并行化**的场景——同时分析 10 个文件、同时查询多个 API、分治大型任务——Hermes 提供了子智能体（Subagent）机制。

把主 Agent 想象成**项目经理**，子智能体是**具体执行的工程师**。项目经理分配任务，多个工程师并行工作，最后汇总结果。

---

## delegate_task 工具

`tools/delegate_tool.py` 实现了 `delegate_task` 工具：

```python
# 主 Agent 的工具调用
{
  "tool": "delegate_task",
  "instructions": "分析 user_service.py 文件，找出所有安全漏洞，特别关注 SQL 注入和权限检查",
  "files": ["user_service.py"],      # 可选：传入文件上下文
  "budget_tokens": 5000,             # 可选：限制子 Agent 的 Token 预算
  "context": "这是一个 Flask 应用的用户服务模块"
}
```

### 内部实现

```python
# tools/delegate_tool.py（简化）
import concurrent.futures

def delegate_task(instructions: str, task_id: str = None, **kwargs):
    """在线程池中启动子 Agent 实例"""
    
    # 子 Agent 有独立的：
    # - 对话上下文
    # - 工具集
    # - 迭代预算（从父 Agent 预算中划拨）
    child_agent = AIAgent(
        # 继承父 Agent 的模型配置
        # 但有独立的消息历史
        iteration_budget=ChildBudget(parent_budget, tokens=budget_tokens)
    )
    
    # 在线程池中执行
    with ThreadPoolExecutor() as executor:
        future = executor.submit(child_agent.run_conversation, instructions)
        result = future.result()
    
    return json.dumps({"result": result["final_response"]})
```

---

## 并行子任务示例

主 Agent 同时启动多个子 Agent 处理不同任务：

```
你：分析我们代码库中所有 Python 文件的代码质量，生成报告

Agent 内部执行：
1. 调用 terminal 工具获取所有 .py 文件列表（30 个文件）
2. 将文件分成 5 组，每组 6 个文件
3. 同时启动 5 个 delegate_task 工具调用（并行）：
   - 子 Agent 1：分析 api/routes/*.py
   - 子 Agent 2：分析 models/*.py
   - 子 Agent 3：分析 services/*.py
   - 子 Agent 4：分析 utils/*.py
   - 子 Agent 5：分析 tests/*.py
4. 等待所有子 Agent 完成（约 30 秒，而串行需要 150 秒）
5. 汇总所有子 Agent 的结果，生成完整报告
```

---

## 预算隔离

子 Agent 的迭代预算从主 Agent 的预算中划拨：

```
主 Agent 预算：90 次迭代
  ├── 子 Agent 1：分配 20 次
  ├── 子 Agent 2：分配 20 次
  ├── 子 Agent 3：分配 20 次
  └── 主 Agent 剩余：30 次
```

这防止子 Agent 无限消耗资源。`_last_resolved_tool_names` 是进程全局变量，`delegate_tool.py` 中的 `_run_single_child()` 在子 Agent 执行前后保存/恢复这个全局，避免状态污染。

---

## execute_code 工具（Python 代码执行）

`tools/code_execution_tool.py` 提供 `execute_code` 工具，允许 Agent 执行任意 Python 代码：

```python
# Agent 调用 execute_code
{
  "tool": "execute_code",
  "code": """
import pandas as pd
import json

df = pd.read_csv('data.csv')
result = {
    'rows': len(df),
    'columns': list(df.columns),
    'summary': df.describe().to_dict()
}
print(json.dumps(result, indent=2))
  """,
  "language": "python"
}
```

`execute_code` 与 `terminal` 的区别：
- `terminal`：运行 Shell 命令
- `execute_code`：运行完整的 Python 脚本，可以导入库、处理数据

---

## 后台任务

```
# 在对话中启动后台任务
/background 每隔 30 分钟检查一次服务器状态，如有异常发送告警

# 查看正在运行的 Agent
/agents

# 后台任务完成后，通过消息平台通知（如 Telegram）
```

`terminal` 工具也支持后台执行：

```python
# 带通知的后台进程
{
  "tool": "terminal",
  "command": "python train_model.py",
  "background": True,
  "notify_on_complete": True  # 完成后通知
}
```

通知详细程度通过 `display.background_process_notifications` 控制：
- `all`：运行中更新 + 最终消息（默认）
- `result`：只有最终消息
- `error`：只在出错时通知
- `off`：不通知

---

## 批处理运行器（batch_runner.py）

`batch_runner.py` 专为大规模批量处理设计：

```bash
# 对多个输入并行运行 Agent 任务
python batch_runner.py \
  --config batch_config.yaml \
  --concurrency 5            # 同时运行 5 个 Agent
```

```yaml
# batch_config.yaml 示例
tasks:
  - input: "分析 report_2024_01.pdf"
  - input: "分析 report_2024_02.pdf"
  - input: "分析 report_2024_03.pdf"
  # ... 更多任务

agent_config:
  model: "google/gemini-2.5-flash"  # 使用便宜模型降低成本
  max_turns: 30
```

批处理适用场景：
- 批量文档分析
- 批量代码审查
- 批量数据处理
- 研究任务并行化

---

## MoA 模式（Mixture of Agents）

`tools/mixture_of_agents_tool.py` 实现了 Mixture of Agents（MoA）模式：

多个 Agent 独立回答同一个问题，然后由一个"聚合 Agent"综合所有答案：

```
问题 ──→ Agent A (GPT-4o)       ──→
        Agent B (Claude 3.5)    ──→  聚合 Agent → 最终答案
        Agent C (Gemini 2.5)    ──→
```

MoA 模式可以利用不同模型的不同优势，生成比单个模型更全面的答案。

---

## 本章小结

- `delegate_task` 工具在线程池中启动独立子 Agent，支持真正的并行处理
- 子 Agent 有独立的上下文和工具集，但共享父 Agent 的迭代预算（通过划拨实现隔离）
- `execute_code` 工具执行完整 Python 脚本，比 `terminal` 更适合数据处理
- `/background` 命令启动后台任务，`/agents` 查看正在运行的 Agent
- `batch_runner.py` 支持大规模并行批处理
- MoA（Mixture of Agents）模式：多模型并行回答 + 聚合，提高答案质量
