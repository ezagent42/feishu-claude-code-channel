# Feishu

Connect a Feishu (Lark) bot to your Claude Code with an MCP server.

When the bot receives a message, the MCP server forwards it to Claude and provides tools to reply, react, and edit messages.

## Prerequisites

- [Bun](https://bun.sh) — the MCP server runs on Bun. Install with `curl -fsSL https://bun.sh/install | bash`.

## Quick Setup
> Default pairing flow for a single-user DM bot. See [ACCESS.md](./ACCESS.md) for groups and multi-user setups.

**1. Create a Feishu application and bot.**

Go to [飞书开放平台](https://open.feishu.cn) (or [Lark Developer](https://open.larksuite.com) for international) and create a new app (创建自建应用).

Navigate to **添加应用能力** and enable **机器人** (Bot).

**2. Configure event subscription.**

Navigate to **事件订阅** → **事件配置**. Add the event `im.message.receive_v1` (接收消息).

Set the subscription method to **长连接** (Long Connection / WebSocket) — this means the bot connects outward and does not need a public URL.

**3. Enable permissions.**

Navigate to **权限管理** and enable:

- `im:message` — Read messages
- `im:message:send_as_bot` — Send messages as bot
- `im:resource` — Access message resources (files, images)
- `im:chat` — Access chat info
- `im:message.reactions:write_only` — Add reactions

**4. Get credentials.**

Navigate to **凭证与基础信息**. Copy the **App ID** and **App Secret**.

**5. Publish the app.**

Submit the app for review (发布应用). The tenant admin needs to approve it. For development/testing, you can use a test tenant (测试企业).

**6. Install the plugin.**

These are Claude Code commands — run `claude` to start a session first.

Install the plugin:
```
/plugin install feishu@claude-plugins-official
```

**7. Give the server the credentials.**

```
/feishu:configure cli_xxxx your_app_secret
```

Writes `FEISHU_APP_ID=...` and `FEISHU_APP_SECRET=...` to `~/.claude/channels/feishu/.env`.

For Lark international, add `--lark`:
```
/feishu:configure cli_xxxx your_app_secret --lark
```

**8. Relaunch with the channel flag.**

The server won't connect without this — exit your session and start a new one:

```sh
claude --channels plugin:feishu@claude-plugins-official
```

**9. Pair.**

With Claude Code running from the previous step, DM your bot on Feishu — it replies with a pairing code. In your Claude Code session:

```
/feishu:access pair <code>
```

Your next DM reaches the assistant.

**10. Lock it down.**

Pairing is for capturing IDs. Once you're in, switch to `allowlist` so strangers don't get pairing-code replies. Ask Claude to do it, or `/feishu:access policy allowlist` directly.

## Access control

See **[ACCESS.md](./ACCESS.md)** for DM policies, group chats, mention detection, delivery config, skill commands, and the `access.json` schema.

Quick reference: IDs are Feishu **open_id** strings (e.g. `ou_xxxx`). Default policy is `pairing`. Group chats are opt-in per chat_id.

## Tools exposed to the assistant

| Tool | Purpose |
| --- | --- |
| `reply` | Send to a chat. Takes `chat_id` + `text`, optionally `reply_to` (message ID) for threading and `files` (absolute paths) for attachments — max 10 files, 25MB each. Auto-chunks; images and files sent as separate messages. Returns the sent message ID(s). |
| `react` | Add an emoji reaction to any message by ID. Use Feishu emoji_type strings like `THUMBSUP`, not Unicode emoji. |
| `edit_message` | Edit a message the bot previously sent. Useful for "working…" → result progress updates. Only works on the bot's own text/post messages. |
| `fetch_messages` | Pull recent history from a chat (oldest-first). Capped at 50 per call. Each line includes the message ID so the model can `reply_to` it. |
| `download_attachment` | Download attachments from a specific message by ID to `~/.claude/channels/feishu/inbox/`. Returns file paths + metadata. Use when `fetch_messages` shows a message has attachments. |

## Attachments

Attachments are **not** auto-downloaded. The `<channel>` notification lists
each attachment's name and type — the assistant calls
`download_attachment(chat_id, message_id)` when it actually wants the file.
Downloads land in `~/.claude/channels/feishu/inbox/`.

## Lark international

Set `FEISHU_DOMAIN=https://open.larksuite.com` in `~/.claude/channels/feishu/.env`, or use the `--lark` flag when configuring:

```
/feishu:configure cli_xxxx your_app_secret --lark
```
