# 第 15 章：实战案例

> 本章展示 5 个真实使用场景，充分利用 Hermes Agent 的核心功能

---

## 案例一：个人智能助理

### 场景描述

将 Hermes 配置为全天候的个人助理，通过 Telegram 随时随地与它交互，它能记住你的偏好，自动完成日常任务。

### 关键功能

- **记忆系统**：跨会话记住你的日程、偏好、常用信息
- **SOUL.md**：定义助理人格和行为规范
- **Telegram 网关**：手机随时访问
- **定时任务（Cron）**：自动执行日常任务

### 配置步骤

**1. 配置 SOUL.md（个性化人格）**

```markdown
# ~/.hermes/SOUL.md

你是我的个人助理，了解我的工作和生活习惯。

## 基本原则
- 用简洁中文回答，重要事项先给结论
- 记住重要信息，下次不用重复告知
- 主动提醒我可能忽略的事项

## 关于我
- 我是软件工程师，工作在上海
- 上班时间 10:00-19:00，周末不工作
- 偏好简洁的技术性解答
```

**2. 配置 Telegram 接入**

```bash
# ~/.hermes/.env
TELEGRAM_BOT_TOKEN=你的Bot_Token
TELEGRAM_ALLOWED_USERS=你的用户名
```

**3. 配置每日摘要（定时任务）**

```bash
# 在 Hermes 中创建定时任务
你：每天早上 9 点帮我生成今日任务摘要，包括昨天未完成的任务和今天的计划
# Agent 会调用 cronjob 工具创建定时任务
```

**4. 训练记忆**

第一次告诉 Hermes 关于你的信息：

```
你：记住，我的家庭服务器 IP 是 192.168.1.100，用户名是 ubuntu
    我的 GitHub 用户名是 alice，常用邮箱是 alice@example.com
```

之后 Hermes 会自动记住这些信息，下次使用时无需重复。

### 实际对话示例

```
[早上 9:00 Telegram 自动消息]
Hermes：早上好！今日任务摘要：
- 📌 未完成（昨日）：PR #123 代码审查
- 📅 今日安排：下午 3 点团队会议
- ⏰ 提醒：服务器证书将在 3 天后过期

[你，下午]
你：帮我看看服务器状态

[Hermes 自动连接已知服务器]
Hermes：正在检查 192.168.1.100 ...
CPU：23%，内存：67%，磁盘：45%
所有服务正常运行。
```

---

## 案例二：编程助手

### 场景描述

将 Hermes 作为日常编程的 AI 搭档：代码审查、调试、重构、自动化测试——全程在终端完成。

### 关键功能

- **终端工具**：执行代码、运行测试
- **文件工具**：读写代码文件
- **浏览器工具**：查阅文档
- **GitHub 技能**：代码审查、PR 管理
- **子智能体**：并行分析大型代码库

### 典型工作流

```bash
# 进入项目目录后启动 Hermes
cd /path/to/my-project
hermes

# 或使用 AGENTS.md 预置项目上下文
cat > AGENTS.md << 'EOF'
## 项目：FastAPI 微服务
- 主语言：Python 3.12
- 测试框架：pytest
- 代码风格：Black + Ruff
- 运行测试：pytest tests/ -v
- 启动服务：uvicorn app.main:app --reload
EOF
```

**代码调试会话**：

```
你：我的 API 一直返回 500 错误，帮我排查
    
Hermes：
[调用 terminal] 运行测试...
[调用 read_file] 查看 app/routes/users.py...
[调用 terminal] 检查日志...

找到问题：user_service.py 第 47 行，get_user_by_email() 
在用户不存在时返回 None，但调用方未处理 None 值。

建议修复：
```python
# 修改前
user = await user_service.get_user_by_email(email)
return user.to_dict()  # ← None 时报 AttributeError

# 修改后
user = await user_service.get_user_by_email(email)
if user is None:
    raise HTTPException(status_code=404, detail="User not found")
return user.to_dict()
```

要我直接修改代码并运行测试验证吗？

你：是的，帮我修改并验证

Hermes：
[调用 patch] 应用代码修改...
[调用 terminal] pytest tests/test_users.py -v...
✅ 所有 12 个测试通过。
```

**并行代码库分析**：

```
你：我们的代码库有 50 个 Python 文件，帮我找出所有潜在的安全问题

Hermes：好的，我会分组并行分析...
[delegate_task] 分析 api/ 目录（12 文件）
[delegate_task] 分析 models/ 目录（8 文件）
[delegate_task] 分析 services/ 目录（18 文件）
... 4 个子 Agent 并行分析 ...

汇总报告：发现 7 个安全问题：
1. SQL 注入风险：user_service.py:83
2. 未验证文件类型：upload_handler.py:45
...
```

---

## 案例三：运维自动化助手

### 场景描述

将 Hermes 作为 DevOps 助手，监控服务器、处理告警、执行部署。

### 关键功能

- **SSH 终端后端**：安全连接远程服务器
- **定时任务**：定期健康检查
- **Home Assistant 工具**：与智能家居/IoT 联动
- **批处理**：多服务器并行检查

### 配置 SSH 后端

```yaml
# ~/.hermes/config.yaml
terminal:
  backend: ssh
  ssh:
    host: 10.0.1.5
    user: ubuntu
    key_file: ~/.ssh/id_rsa
```

### 监控脚本（通过 Cron）

```
你：每 5 分钟检查一次生产服务器状态，如果 CPU > 90% 或内存 > 95%，
    通过 Telegram 发送告警

Hermes：好的，我来创建监控任务...
[调用 cronjob 工具创建定时任务]
```

### 自动部署流程

```
你：将 main 分支部署到生产服务器

Hermes：开始部署流程...
[terminal] git pull origin main
[terminal] docker build -t myapp:latest .
[terminal] docker push registry.example.com/myapp:latest
[terminal] SSH → 生产服务器
[terminal] docker pull registry.example.com/myapp:latest
[terminal] docker-compose up -d --no-deps app
[terminal] curl http://localhost/health
✅ 部署成功！服务健康检查通过。
```

---

## 案例四：个人知识管理

### 场景描述

将 Hermes 作为第二大脑，管理笔记、归档知识、快速检索历史对话。

### 关键功能

- **记忆工具**：保存重要知识片段
- **FTS5 会话搜索**：全文搜索所有历史对话
- **技能系统**：为常见知识管理任务创建技能
- **文件工具**：管理笔记文件

### 工作流示例

**保存学习笔记**：

```
你：我刚学完 Rust 的生命周期（Lifetime）机制，帮我总结要点并保存到记忆中

Hermes：好的，以下是 Rust 生命周期的核心要点：
1. 生命周期注解（'a）用于告诉编译器引用的有效范围...
[调用 memory 工具，写入 MEMORY.md]
已保存到记忆中。下次问我 Rust 相关问题时会自动参考这些知识。
```

**检索历史对话**：

```
你：上周我问过关于 Redis 集群的配置问题，帮我找到那个对话

Hermes：
[调用 session_search 工具]
找到 3 个相关会话：
1. 2024-01-15 "Redis 主从复制配置" - 包含主从配置步骤
2. 2024-01-18 "Redis Sentinel 设置" - 包含 HA 配置
3. 2024-01-20 "Redis 集群分片" - 包含集群分片方案

你想查看哪个会话的详细内容？
```

**知识库搜索**：

```bash
# CLI 搜索
hermes sessions browse
```

---

## 案例五：自动化任务流水线

### 场景描述

构建完全自动化的任务流水线：定时抓取数据、分析处理、生成报告、推送通知。

### 关键功能

- **定时任务（Cron）**：触发自动化流程
- **批处理（batch_runner.py）**：并行处理大量任务
- **子智能体**：分布式数据处理
- **Webhook 平台**：接收外部触发

### 每日报告流水线

**1. 定义流水线任务（Cron）**

```
你：每天晚上 11 点执行以下流程：
    1. 爬取我关注的 10 个技术博客的新文章
    2. 用 AI 过滤出与 Python/Cloud 相关的文章
    3. 生成摘要报告（Markdown 格式）
    4. 保存到 ~/reports/YYYY-MM-DD.md
    5. 通过 Telegram 发送摘要

Hermes：好的，我来设置这个自动化流水线...
```

**2. 批量数据处理**

```python
# batch_runner.py 使用示例
# hermes batch_runner.py --config blog_monitor.yaml

# blog_monitor.yaml
tasks:
  - input: "摘要 https://blog1.example.com/latest"
  - input: "摘要 https://blog2.example.com/latest"
  # ... 10 个博客 ...

agent_config:
  model: "google/gemini-2.5-flash"  # 使用便宜模型
  max_turns: 10
  concurrency: 5                    # 5 个 Agent 并行
```

**3. Webhook 触发集成**

```bash
# 配置 Webhook 监听
WEBHOOK_PORT=8080

# 外部系统触发（如 GitHub Actions 完成后通知）
curl -X POST http://your-server:8080/webhook \
  -d '{"message": "CI/CD 流水线完成，请分析测试报告并决定是否部署"}'
```

---

## 本章小结

- **个人助理**：SOUL.md 定义人格 + 记忆系统 + Telegram 网关 + Cron 定时任务
- **编程助手**：终端/文件/浏览器工具 + AGENTS.md 项目上下文 + 子智能体并行分析
- **运维自动化**：SSH 后端 + 定时监控 + 自动部署流程
- **知识管理**：memory 工具 + FTS5 会话搜索 + 技能积累
- **自动化流水线**：Cron 触发 + batch_runner.py 并行 + Webhook 集成

这些场景展示了 Hermes 各功能模块的组合能力——越用越聪明（技能自动积累），越用越了解你（记忆系统），越用越高效（并行子智能体）。
