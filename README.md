# Feishu

Connect a Feishu bot to Claude Code with an MCP server.

When the bot receives a message, the MCP server forwards it to Claude and provides tools to reply, react, and edit messages.

## Prerequisites

- [Bun](https://bun.sh) — the MCP server runs on Bun. Install with `curl -fsSL https://bun.sh/install | bash`

## Quick Setup
> Default pairing flow for a single-user DM bot. See [ACCESS.md](./ACCESS.md) for groups and multi-user setups.

**1. Create a Feishu app and bot**

Go to the [Feishu Open Platform](https://open.feishu.cn) (or [Lark Developer](https://open.larksuite.com) for the international version) and create a custom app.

Navigate to **Add App Capabilities** and enable **Bot**.

**2. Configure event subscriptions**

Go to **Event Subscriptions** → **Event Configuration** and add the event `im.message.receive_v1` (receive messages).

Select **Long Connection** (WebSocket) as the subscription method — the bot connects outbound, no public IP required.

**3. Enable permissions**

Go to **Permissions & Scopes** and enable the following:

- `im:message` — read messages
- `im:message:send_as_bot` — send messages as bot
- `im:resource` — access message resources (files, images)
- `im:chat` — access chat info
- `im:message.reactions:write_only` — add emoji reactions

**4. Get credentials**

Go to **Credentials & Basic Info** and copy the **App ID** and **App Secret**.

**5. Publish the app**

Submit the app for review and get it approved by a tenant admin. You can use a test enterprise for development.

**6. Install the plugin**

The following are Claude Code commands — start a session with `claude` first.

Add the marketplace, then install the plugin:
```
/plugin marketplace add ezagent42/ezagent42
/plugin install feishu@ezagent42
```

**7. Configure credentials**

```
/feishu:configure cli_xxxx your_app_secret
```

This writes `FEISHU_APP_ID=...` and `FEISHU_APP_SECRET=...` to `~/.claude/channels/feishu/.env`.

For Lark international, add the `--lark` flag:
```
/feishu:configure cli_xxxx your_app_secret --lark
```

**8. Restart with channel flag**

Exit the current session and restart with:

```sh
claude --dangerously-load-development-channels plugin:feishu@ezagent42
```

> **Note:** The Feishu channel is not yet on the Claude Code official approved channels allowlist. You must use the `--dangerously-load-development-channels` flag to start. This flag only bypasses the allowlist check and does not skip other security checks.

**9. Pair**

After starting Claude Code, DM your bot on Feishu — it will reply with a pairing code. In the Claude Code session, run:

```
/feishu:access pair <code>
```

Your DM messages will now be forwarded to Claude.

**10. Lock down access**

Pairing is only used to obtain your ID. After pairing, switch to `allowlist` mode to prevent strangers from triggering pairing. Ask Claude to do it, or run `/feishu:access policy allowlist` directly.

## Access Control

See **[ACCESS.md](./ACCESS.md)** — covers DM policies, group chats, mention detection, delivery configuration, skill commands, and the `access.json` structure.

Quick reference: IDs use the Feishu **open_id** format (e.g. `ou_xxxx`). Default policy is `pairing`; group chats must be enabled per chat_id.

## Tools Available to Claude

| Tool | Purpose |
| --- | --- |
| `reply` | Send a message to a chat. Params: `chat_id` + `text`, optional `reply_to` (message ID for quote-reply) and `files` (absolute paths for attachments) — up to 10 files, 25MB each. Auto-chunked; images and files sent as separate messages. Returns sent message ID. |
| `react` | Add an emoji reaction to a message. Uses Feishu emoji_type strings like `THUMBSUP`, not Unicode. |
| `edit_message` | Edit a message previously sent by the bot. Useful for "processing..." → result progress updates. Only works on bot's own text/rich-text messages. |
| `fetch_messages` | Fetch recent chat history (chronological order). Up to 50 messages per call. Each line includes a message ID usable for `reply_to`. |
| `download_attachment` | Download attachments from a message to `~/.claude/channels/feishu/inbox/`. Returns file path and metadata. Use when `fetch_messages` shows a message has attachments. |

## Attachments

Attachments are **not** downloaded automatically. The `<channel>` notification lists each attachment's name and type — Claude calls `download_attachment(chat_id, message_id)` to fetch them on demand. Files are saved to `~/.claude/channels/feishu/inbox/`.

## Lark International

Set `FEISHU_DOMAIN=https://open.larksuite.com` in `~/.claude/channels/feishu/.env`, or use the `--lark` flag during configuration:

```
/feishu:configure cli_xxxx your_app_secret --lark
```

---

# 飞书

将飞书机器人连接到 Claude Code 的 MCP 服务器。

机器人收到消息后，MCP 服务器将其转发给 Claude，并提供回复、表情回应和编辑消息的工具。

## 前置条件

- [Bun](https://bun.sh) — MCP 服务器基于 Bun 运行。安装命令：`curl -fsSL https://bun.sh/install | bash`

## 快速配置
> 默认配对流程，适用于单用户私聊机器人。群聊和多用户配置请参见 [ACCESS.md](./ACCESS.md)。

**1. 创建飞书应用和机器人**

前往[飞书开放平台](https://open.feishu.cn)（国际版用 [Lark Developer](https://open.larksuite.com)），创建自建应用。

进入 **添加应用能力**，启用 **机器人**。

**2. 配置事件订阅**

进入 **事件订阅** → **事件配置**，添加事件 `im.message.receive_v1`（接收消息）。

订阅方式选择 **长连接**（WebSocket）— 机器人主动外连，无需公网 IP。

**3. 开通权限**

进入 **权限管理**，开通以下权限：

- `im:message` — 读取消息
- `im:message:send_as_bot` — 以机器人身份发送消息
- `im:resource` — 访问消息资源（文件、图片）
- `im:chat` — 访问会话信息
- `im:message.reactions:write_only` — 添加表情回应

**4. 获取凭证**

进入 **凭证与基础信息**，复制 **App ID** 和 **App Secret**。

**5. 发布应用**

提交应用审核（发布应用），租户管理员审批通过。开发测试可使用测试企业。

**6. 安装插件**

以下为 Claude Code 命令 — 先运行 `claude` 启动会话。

先添加市场，再安装插件：
```
/plugin marketplace add ezagent42/ezagent42
/plugin install feishu@ezagent42
```

**7. 配置凭证**

```
/feishu:configure cli_xxxx your_app_secret
```

将 `FEISHU_APP_ID=...` 和 `FEISHU_APP_SECRET=...` 写入 `~/.claude/channels/feishu/.env`。

国际版 Lark 加 `--lark` 参数：
```
/feishu:configure cli_xxxx your_app_secret --lark
```

**8. 使用 channel 标志重启**

退出当前会话，使用以下命令重新启动：

```sh
claude --dangerously-load-development-channels plugin:feishu@ezagent42
```

> **注意：** 飞书 channel 当前尚未进入 Claude Code 官方 approved channels allowlist，仅支持使用 `--dangerously-load-development-channels` 标志启动。该标志仅绕过 allowlist 检查，不跳过其他安全检查。

**9. 配对**

启动 Claude Code 后，在飞书中私聊你的机器人 — 机器人会回复一个配对码。在 Claude Code 会话中运行：

```
/feishu:access pair <code>
```

之后你的私聊消息就会发送给 Claude。

**10. 锁定访问**

配对仅用于获取 ID。配对完成后，切换为 `allowlist` 模式以防止陌生人触发配对。让 Claude 帮你操作，或直接运行 `/feishu:access policy allowlist`。

## 访问控制

详见 **[ACCESS.md](./ACCESS.md)** — 包括私聊策略、群聊、提及检测、投递配置、技能命令和 `access.json` 结构。

快速参考：ID 为飞书 **open_id** 格式（如 `ou_xxxx`）。默认策略为 `pairing`，群聊需按 chat_id 单独启用。

## 提供给 Claude 的工具

| 工具 | 用途 |
| --- | --- |
| `reply` | 向会话发送消息。参数：`chat_id` + `text`，可选 `reply_to`（消息 ID，用于引用回复）和 `files`（绝对路径，用于附件）— 最多 10 个文件，每个 25MB。自动分段；图片和文件作为单独消息发送。返回已发送的消息 ID。 |
| `react` | 对指定消息添加表情回应。使用飞书 emoji_type 字符串如 `THUMBSUP`，非 Unicode。 |
| `edit_message` | 编辑机器人之前发送的消息。适用于"处理中…" → 结果的进度更新。仅支持机器人自己发送的文本/富文本消息。 |
| `fetch_messages` | 获取会话的近期历史消息（按时间正序）。每次最多 50 条。每行包含消息 ID，可用于 `reply_to`。 |
| `download_attachment` | 下载指定消息的附件到 `~/.claude/channels/feishu/inbox/`。返回文件路径和元数据。当 `fetch_messages` 显示消息有附件时使用。 |

## 附件

附件 **不会** 自动下载。`<channel>` 通知会列出每个附件的名称和类型 — Claude 在需要时调用 `download_attachment(chat_id, message_id)` 下载。文件保存在 `~/.claude/channels/feishu/inbox/`。

## Lark 国际版

在 `~/.claude/channels/feishu/.env` 中设置 `FEISHU_DOMAIN=https://open.larksuite.com`，或配置时使用 `--lark` 参数：

```
/feishu:configure cli_xxxx your_app_secret --lark
```
