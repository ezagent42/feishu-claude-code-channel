# Feishu

[中文文档](./README.zh-CN.md)

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

