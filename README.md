# telegram-bridge

![Version](https://img.shields.io/badge/version-0.5-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Bash](https://img.shields.io/badge/bash-4%2B-orange)

**Telegram ↔ tmux bridge for Claude Code**

Control multiple Claude Code sessions from your phone via Telegram Forum Topics. One topic per session, fully bidirectional.

## Features

- **One bot, multiple sessions** — each tmux session gets its own Forum Topic in a Telegram group
- **Bidirectional** — messages from Telegram inject into sessions, Claude's responses stream back via JSONL transcript monitoring
- **Works regardless of input source** — type in terminal or Telegram, output always appears in both places
- **"Working..." indicator** — progress dots while Claude thinks or runs tools, "✓ Done" when idle
- **Preview + expandable blockquote** — first 200 chars visible for long responses, tap to expand
- **Session lifecycle** — auto-creates topics for new sessions, deletes when sessions end, renames when an external renamer (e.g. janitor) renames the tmux session
- **Resume conversations** — browse and resume old conversations from Telegram
- **File sending** — send documents, photos, video from phone directly to a session
- **Voice messages** — transcribed via Whisper (Groq or OpenAI) and injected into the session as text
- **Terminal screenshots** — `/screenshot` captures the tmux pane and sends it as a code block
- **Permission approvals** — `/approval yes|no` for tool permission prompts without switching to a terminal
- **Pure bash + curl + jq** — no Python, no Node, no external dependencies beyond standard CLI tools

## Commands

| Command | Description |
|---|---|
| `/sessions` | List active Claude Code sessions |
| `/new [danger\|normal]` | Launch a new session (default: normal) |
| `/resume` | Browse and resume old conversations |
| `/kill N` | Kill session N (requires `ENABLE_KILL=true`) |
| `/approval [yes\|no]` | Approve/deny a pending permission prompt |
| `/screenshot` | Capture and send current terminal screen |
| `/help` | Show command reference |
| *any text* | Routes to the session mapped to this topic |
| *file/photo* | Downloads to `~/tmp/` and notifies the session |
| *voice message* | Transcribed and injected into session |

## Architecture

Three layers handle the bridge between Telegram and your Claude Code sessions:

1. **Telegram Bot API polling** — long-polls `getUpdates` every 2 seconds, routes incoming messages to the right session by Forum Topic ID
2. **JSONL transcript monitoring** — one background watcher per session tails the Claude Code conversation JSONL file using byte-offset tracking, extracts new assistant and user messages, and forwards them to Telegram
3. **tmux integration** — `send-keys` injects text into sessions, session list detection creates/deletes/renames topics as sessions come and go, PID tracking distinguishes renames from new sessions

```
Phone (Telegram)
    ↕ Bot API (HTTPS polling)
telegram-bridge (bash, runs in tmux)
    ↕ JSONL watchers + tmux send-keys
Claude Code sessions (separate tmux sessions)
```

The bridge runs as its own tmux session. It self-launches: calling `telegram-bridge` when it's already running is a no-op.

## Setup

### 1. Create the Telegram bot

1. Message [@BotFather](https://t.me/BotFather) on Telegram
2. `/newbot` — pick a name and username
3. Copy the bot token

### 2. Create a Forum-enabled group

1. Create a new Telegram group
2. Go to group settings and enable **Topics** (this turns it into a Forum group)
3. Add your bot to the group as an **admin** with at least the **Manage Topics** permission
4. **Disable Group Privacy**: in BotFather, go to your bot → Bot Settings → Group Privacy → Turn **OFF**. The bot needs to read all messages in topics, not just commands.

### 3. Get the group chat ID

Send any message in the group, then check the bot's updates:

```bash
curl -s "https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates" | jq '.result[0].message.chat.id'
```

The group ID will be a negative number (e.g. `-1001234567890`).

### 4. Get your Telegram user ID

Message [@userinfobot](https://t.me/userinfobot) on Telegram — it replies with your numeric user ID.

### 5. Configure

```bash
mkdir -p ~/.config/telegram-bridge
cp .env.example ~/.config/telegram-bridge/.env
```

Edit `~/.config/telegram-bridge/.env` and fill in:
- `TELEGRAM_BOT_TOKEN` — from step 1
- `TELEGRAM_GROUP_ID` — from step 3
- `TELEGRAM_ALLOWED_USER` — from step 4

### 6. Install

```bash
cp telegram-bridge ~/bin/
# or anywhere in your PATH
chmod +x ~/bin/telegram-bridge
```

### 7. Run

```bash
telegram-bridge
```

This launches a background tmux session called `telegram-bridge`. It will auto-detect existing Claude Code sessions and create Forum Topics for them.

## Configuration

All config lives in `~/.config/telegram-bridge/.env`:

| Variable | Required | Description |
|---|---|---|
| `TELEGRAM_BOT_TOKEN` | Yes | Bot API token from @BotFather |
| `TELEGRAM_GROUP_ID` | Yes | Forum-enabled group chat ID (negative number) |
| `TELEGRAM_ALLOWED_USER` | Yes | Your Telegram user ID (only this user can interact) |
| `ENABLE_KILL` | No | Set to `true` to enable the `/kill` command (default: `false`) |
| `GROQ_API_KEY` | No | For voice transcription via Groq Whisper (free tier available) |
| `OPENAI_API_KEY` | No | For voice transcription via OpenAI Whisper (fallback if no Groq key) |

### Optional environment variables

These can be set in your shell or `.env` to customize paths:

| Variable | Default | Description |
|---|---|---|
| `TELEGRAM_BRIDGE_PROJECT_DIR` | `$HOME` | Working directory for new Claude Code sessions |
| `TELEGRAM_BRIDGE_CONV_DIR` | `$HOME/.claude/projects` | Directory containing Claude Code JSONL conversation files |

### State files

The bridge stores state in `~/.config/telegram-bridge/`:

- `state.json` — topic-to-session mappings, Telegram update offset
- `offsets/` — byte offsets for JSONL file watchers (survive restarts)
- `watchers/` — PID files for background watcher processes
- `last_sent/` — echo suppression (tracks last message sent via Telegram per session)

## Integration

### Works with any tmux session manager

The bridge detects Claude Code sessions by checking `#{pane_current_command}` for `claude`. Any tool that launches Claude Code in a tmux session is compatible.

### Session renaming

Compatible with janitor-style session renamers. The bridge tracks the pane PID of each session, so when an external tool renames a tmux session (e.g. from `claude-0326` to `fix-login-bug`), the bridge detects the PID match and updates the Forum Topic name instead of treating it as a delete + create.

### Startup scripts

Add to your startup script to auto-launch:

```bash
# In your .bashrc, .profile, or session launcher
telegram-bridge  # idempotent — won't double-launch
```

### Cron watchdog

Ensure the bridge stays running with a cron entry:

```bash
*/5 * * * * /path/to/telegram-bridge >> ~/.config/telegram-bridge/logs/telegram-bridge-cron.log 2>&1
```

### Stopping

```bash
tmux kill-session -t telegram-bridge
```

## Requirements

- **bash** 4+
- **curl**
- **jq**
- **tmux**
- **Claude Code** (any version that writes JSONL transcripts to `~/.claude/projects/`)

## Topic colors

The bridge uses Forum Topic icon colors to indicate session type:

| Color | Meaning |
|---|---|
| Green (9367192) | Normal mode session |
| Red (16478047) | Danger mode session (permissions skipped) |
| Blue (7322096) | Resumed conversation |

## Credits

- Inspired by [ccgram](https://github.com/alexei-led/ccgram) — JSONL transcript monitoring approach
- Inspired by [ccbot](https://github.com/six-ddc/ccbot) — Forum Topics per session pattern
- Built with Claude Code by [wlatic](https://github.com/wlatic)

## License

[MIT](LICENSE)
