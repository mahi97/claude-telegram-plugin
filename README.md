# Telegram Plugin for Claude Code (Patched)

Patched fork of `telegram@claude-plugins-official` with fixes for multi-session stability and reply-to context.

## What's Different From the Official Plugin

| Feature | Official | This Fork |
|---------|----------|-----------|
| Multiple sessions | Compete for messages, random delivery | **PID lock** — first instance polls, others serve tools only |
| Agent Team support | Teammates spawn competing bots | PID lock prevents conflicts automatically |
| Reply-to context | Not passed to Claude | **reply_to_message_id + reply_to_text** included in channel meta |
| Network errors | 409-only retry (fixed in v0.0.6) | All-error retry (inherited from v0.0.6) |
| State directory | Always `~/.claude/channels/telegram/` | **Auto-detects project-level** `.claude/telegram/` |
| First-run setup | Manual dir creation + file editing | **Auto-creates dirs**, guides with `/telegram:configure` |

Based on official v0.0.6. See upstream issues: [#38098](https://github.com/anthropics/claude-code/issues/38098), [#39876](https://github.com/anthropics/claude-code/issues/39876).

## Prerequisites

- [Bun](https://bun.sh) — install with `curl -fsSL https://bun.sh/install | bash`

## Quick Setup

**1. Create a bot with [@BotFather](https://t.me/BotFather).**

Send `/newbot`, pick a name and username. Copy the token (`123456789:AAH...`).

**2. Add the marketplace to your project.**

In your project's `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "mahi97-telegram": {
      "source": {
        "source": "github",
        "repo": "mahi97/claude-telegram-plugin"
      }
    }
  },
  "enabledPlugins": {
    "telegram@mahi97-telegram": true
  }
}
```

Or add it globally in `~/.claude/settings.json` if you want it in all projects.

**3. Start Claude Code with the channel flag.**

```bash
claude --channels plugin:telegram@mahi97-telegram
```

The plugin auto-creates `.claude/telegram/` in your project. On first run it prints:

```
telegram channel: TELEGRAM_BOT_TOKEN required
  Run: /telegram:configure <token>
```

**4. Configure the token.**

```
/telegram:configure 123456789:AAHfiqksKZ8...
```

This saves the token to `.claude/telegram/.env` and adds `.claude/telegram/` to `.gitignore`.

**5. Restart and pair.**

```bash
claude --channels plugin:telegram@mahi97-telegram
```

DM your bot on Telegram — it replies with a 6-character pairing code:

```
/telegram:access pair <code>
```

**6. Lock it down.**

```
/telegram:access policy allowlist
```

Done. Your bot only responds to approved users.

## Multi-Project Setup

Each project can have its own bot (or share one). The plugin auto-detects:

- **Project has `.claude/`** → uses `.claude/telegram/` (project-scoped)
- **No `.claude/`** → falls back to `~/.claude/channels/telegram/` (user-scoped)
- **`TELEGRAM_STATE_DIR` env var** → overrides both (explicit path)

Different tokens = different bots. Same token = same bot, separate access control per project.

## Disabling the Official Plugin

If you previously had `telegram@claude-plugins-official` enabled, disable it in `~/.claude/settings.json`:

```json
{
  "enabledPlugins": {
    "telegram@claude-plugins-official": false
  }
}
```

This prevents the official plugin from spawning competing instances.

## Tools Exposed to the Assistant

| Tool | Purpose |
|------|---------|
| `reply` | Send to a chat. Supports `reply_to` for threading and `files` for attachments. |
| `react` | Add an emoji reaction (Telegram whitelist only). |
| `edit_message` | Edit a bot message. Useful for progress updates. |
| `download_attachment` | Download file attachments from inbound messages. |

## Reply-To Context

When a user replies to a specific message in Telegram, the inbound channel notification includes:

- `reply_to_message_id` — ID of the message being replied to
- `reply_to_text` — first 200 characters of that message

This lets Claude understand which message the user is referring to.

## Access Control

See [ACCESS.md](./ACCESS.md) for DM policies, groups, mention detection, and the full `access.json` schema.

## License

Apache 2.0 (same as the official plugin).
