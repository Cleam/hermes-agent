# 第 14 章：消息网关与多平台接入

> 相关源码：`gateway/run.py`、`gateway/platforms/`、`gateway/config.py`

---

## 消息网关概述

Hermes 的消息网关（Messaging Gateway）将 AI Agent 接入各种消息平台，让你可以通过 Telegram 发消息给它，通过企业微信分配任务，或通过 Webhook 集成自动化流程。

本质上，网关是一个"消息路由器"：

```
Telegram 消息 ──→
Discord 消息  ──→  gateway/run.py  ──→  AIAgent  ──→  回复
微信消息      ──→
Webhook 请求  ──→
```

---

## 支持的平台（18+）

| 平台 | 文件 | 说明 |
|------|------|------|
| **Telegram** | `telegram.py` | 最常用，支持语音消息转文字 |
| **Discord** | `discord.py` | 支持频道和私信 |
| **Slack** | `slack.py` | Slash 命令支持 |
| **WhatsApp** | `whatsapp.py` | WhatsApp Business API |
| **Signal** | `signal.py` | Signal CLI 集成 |
| **Email** | `email.py` | SMTP/IMAP |
| **Matrix** | `matrix.py` | 去中心化通讯 |
| **Mattermost** | `mattermost.py` | 私有部署聊天 |
| **钉钉（DingTalk）** | `dingtalk.py` | 企业钉钉机器人 |
| **飞书（Feishu）** | `feishu.py` | 字节跳动飞书 |
| **微信（WeChat）** | `weixin.py` | 微信公众号/小程序 |
| **企业微信（WeCom）** | `wecom.py` | 腾讯企业微信 |
| **QQ 机器人** | `qqbot.py` | QQ 官方机器人 |
| **BlueBubbles** | `bluebubbles.py` | iMessage (macOS) |
| **Webhook** | `webhook.py` | 接收 HTTP 请求 |
| **API Server** | `api_server.py` | REST API 接口 |
| **SMS** | `sms.py` | 短信 |
| **元宝（YuanBao）** | `yuanbao.py` | 腾讯元宝 |

---

## 启动消息网关

```bash
# 前台运行（调试时使用）
hermes gateway

# 后台启动
hermes gateway start

# 查看状态
hermes gateway status

# 停止
hermes gateway stop

# 安装为系统服务（开机自启）
hermes gateway install
hermes gateway uninstall

# 引导配置
hermes gateway setup
```

---

## Telegram 机器人配置示例

### 1. 创建 Telegram 机器人

访问 [@BotFather](https://t.me/botfather)，发送 `/newbot`，获取 Bot Token。

### 2. 配置 Hermes

```bash
# ~/.hermes/.env
TELEGRAM_BOT_TOKEN=1234567890:ABCDefGHIjklMNOpqrSTUvwxyz
TELEGRAM_ALLOWED_USERS=your_username,friend_username
```

或通过引导向导：

```bash
hermes gateway setup  # 选择 Telegram
```

### 3. 启动网关

```bash
hermes gateway start
```

现在你可以通过 Telegram 与 Hermes 对话了。

---

## 访问控制

各平台支持用户白名单，防止未授权使用：

```bash
# ~/.hermes/.env

# Telegram：允许的用户名（不带 @）
TELEGRAM_ALLOWED_USERS=alice,bob,charlie

# Discord：允许的用户 ID
DISCORD_ALLOWED_USERS=123456789,987654321

# Slack：允许的用户 ID
SLACK_ALLOWED_USERS=U01234567,U09876543

# 允许所有用户（不推荐在公开平台上使用）
GATEWAY_ALLOW_ALL_USERS=false   # 默认 false
```

---

## 危险命令审批流程

在消息平台上，当 Agent 准备执行危险命令（如删除文件、修改系统配置），会触发审批请求：

```
[Hermes → 你（Telegram）]
⚠️ 危险命令审批请求
Agent 想要执行：
  rm -rf /home/user/old_project/

这将永久删除目录及其所有内容。
请回复 /approve 批准或 /deny 拒绝。
```

```
[你 → Hermes]
/approve

[Hermes → 你]
✅ 已批准，正在执行...
删除完成。
```

---

## 钉钉机器人配置示例

```bash
# ~/.hermes/.env
DINGTALK_APP_KEY=your_app_key
DINGTALK_APP_SECRET=your_app_secret
DINGTALK_ALLOWED_USERS=user1,user2
```

---

## 飞书机器人配置示例

```bash
# ~/.hermes/.env
FEISHU_APP_ID=cli_xxxxx
FEISHU_APP_SECRET=xxxxxxxxxx
FEISHU_ALLOWED_USERS=ou_xxxxx
```

---

## 企业微信配置示例

```bash
# ~/.hermes/.env
WECOM_CORP_ID=ww_xxxxx
WECOM_APP_SECRET=xxxxxxxxxx
WECOM_AGENT_ID=1000001
```

---

## Webhook 平台

Webhook 平台让 Hermes 可以接收来自任意 HTTP 客户端的消息：

```bash
# ~/.hermes/.env
WEBHOOK_PORT=8080
WEBHOOK_SECRET=your_secret_token  # 验证请求合法性
```

向 Hermes 发送消息：

```bash
curl -X POST http://localhost:8080/webhook \
  -H "Content-Type: application/json" \
  -H "X-Webhook-Secret: your_secret_token" \
  -d '{"message": "检查服务器状态"}'
```

---

## API Server 平台

API Server 提供 RESTful 接口，适合程序化集成：

```bash
# ~/.hermes/.env
API_SERVER_PORT=9000
API_SERVER_TOKEN=your_api_token
```

```bash
# 发送请求
curl -X POST http://localhost:9000/api/chat \
  -H "Authorization: Bearer your_api_token" \
  -H "Content-Type: application/json" \
  -d '{"message": "你好", "session_id": "user_123"}'
```

---

## 语音消息支持

Telegram 等平台支持语音消息，Hermes 通过 Whisper 自动转录：

1. 用户发送语音消息
2. 网关调用 Whisper API 转文字
3. 文字传给 AIAgent 处理
4. Agent 回复文字（或 TTS 语音）

---

## Web 仪表盘

```bash
# 启动仪表盘（本地 Web 界面）
hermes dashboard
```

仪表盘在 `localhost` 上的某个端口提供 Web 界面，通过浏览器访问。内嵌 TUI 界面（通过 PTY 桥接）。

---

## 多平台会话连续性

同一用户通过不同平台发送的消息，可以共享会话历史（需要适当配置 session_id 映射）。这让你可以在手机 Telegram 上开始一个任务，在桌面 Slack 上继续。

---

## 本章小结

- Hermes 网关支持 18+ 消息平台，中国平台包括：钉钉、飞书、微信、企业微信、QQ 机器人
- 使用 `hermes gateway start` 启动，`hermes gateway install` 注册为系统服务
- 各平台通过 `*_ALLOWED_USERS` 和 `GATEWAY_ALLOW_ALL_USERS` 控制访问权限
- 危险命令会触发审批流程（`/approve` / `/deny`）
- Webhook 和 API Server 平台支持程序化集成
- 语音消息通过 Whisper 自动转录
- `hermes dashboard` 提供 Web 仪表盘访问
