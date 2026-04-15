---
name: configure
description: Set up the Telegram channel — save the bot token and review access policy. Use when the user pastes a Telegram bot token, asks to configure Telegram, asks "how do I set this up" or "who can reach me," or wants to check channel status.
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Bash(ls *)
  - Bash(mkdir *)
  - Bash(chmod *)
  - Bash(echo *)
  - Bash(cat *)
  - Bash(grep *)
---

# /telegram:configure — Telegram Channel Setup

Project-aware setup. Detects whether to use project-level (`.claude/telegram/`)
or user-level (`~/.claude/channels/telegram/`) state. Prefers project-level
when a `.claude/` directory exists in the working directory.

Arguments passed: `$ARGUMENTS`

---

## Resolve state directory

1. If `TELEGRAM_STATE_DIR` env var is set, use that.
2. Else if `.claude/` exists in the current working directory, use `.claude/telegram/`.
3. Else use `~/.claude/channels/telegram/`.

Use this resolved path for ALL file operations below.

---

## Dispatch on arguments

### No args — status and guidance

Read both state files and give the user a complete picture:

1. **State dir** — show which directory is being used and why.

2. **Token** — check `<state_dir>/.env` for `TELEGRAM_BOT_TOKEN`.
   Show set/not-set; if set, show first 10 chars masked (`123456789:...`).

3. **Access** — read `<state_dir>/access.json` (missing file = defaults:
   `dmPolicy: "pairing"`, empty allowlist). Show:
   - DM policy and what it means in one line
   - Allowed senders: count, and list display names or IDs
   - Pending pairings: count, with codes and display names if any

4. **What next** — end with a concrete next step based on state:
   - No token → *"Run `/telegram:configure <token>` with the token from
     BotFather."*
   - Token set, policy is pairing, nobody allowed → *"DM your bot on
     Telegram. It replies with a code; approve with `/telegram:access pair
     <code>`."*
   - Token set, someone allowed → *"Ready. DM your bot to reach the
     assistant."*

Push toward lockdown — always.

### `<token>` — save it

1. Treat `$ARGUMENTS` as the token (trim whitespace). BotFather tokens look
   like `123456789:AAHfiqksKZ8...` — numeric prefix, colon, long string.
2. `mkdir -p <state_dir>`
3. Read existing `.env` if present; update/add the `TELEGRAM_BOT_TOKEN=` line,
   preserve other keys. Write back, no quotes around the value.
4. `chmod 600 <state_dir>/.env` — the token is a credential.
5. If `.gitignore` exists at the project root and doesn't already contain
   `.claude/telegram/`, append it. The token is a secret.
6. Confirm, then show the no-args status so the user sees where they stand.

### `clear` — remove the token

Delete the `TELEGRAM_BOT_TOKEN=` line (or the file if that's the only line).

---

## Implementation notes

- The state dir might not exist if the server hasn't run yet. Missing file
  = not configured, not an error. Create dirs as needed.
- The server reads `.env` once at boot. Token changes need a session restart
  or `/reload-plugins`. Say so after saving.
- `access.json` is re-read on every inbound message — policy changes via
  `/telegram:access` take effect immediately, no restart.
