# Changelog

## 0.5.1 (2026-03-26)

### New Commands
- /status — session status with uptime, all sessions from General
- /cancel — send Escape to interrupt Claude
- /close — kill session from its topic (no number needed)
- /restart — kill and resume same conversation

### Status Detection
- Claude Code hooks integration (PreToolUse, Stop, SubagentStart, SubagentStop)
- Hook script: telegram-bridge-hook writes status files for reliable detection
- Falls back to pane scraping if hooks not configured
- Agent name shown in Working... indicator
- 15s cooldown before Done fires
- Permission prompt detection with notification

### Message Handling
- JSONL-based output monitoring (replaces pane scraping)
- Preview + expandable blockquote for long messages (200 char preview)
- Working... resets properly between messages (no orphans)
- Queued message indicator when Claude is busy
- Task notifications displayed from queue-operation entries
- Internal messages filtered (task-notification, teammate-message, system-reminder XML)
- HTML escaping fixed (sed instead of bash substitution)
- Plaintext fallback on HTML parse errors

### Infrastructure
- Always targets pane :1.1 (orchestrator, never agent pane)
- PID-based rename detection for janitor integration
- Session topics deleted on end (not just closed)
- /kill enabled via ENABLE_KILL=true
- Cron watchdog every 5 minutes
- Official Telegram plugin uninstalled (conflicts with getUpdates)

## 0.5 (2026-03-26)

Initial release.

### Features
- Bidirectional JSONL transcript monitoring
- Forum Topics per tmux session
- Commands: /sessions, /new, /resume, /kill, /approval, /screenshot, /help
- Working... indicator with dots, Done on idle
- Preview + expandable blockquote for long messages
- File/photo/video sending from Telegram to session
- Voice message transcription (Groq/OpenAI Whisper)
- Session lifecycle management (create, rename, delete topics)
- PID-based rename detection for external session renamers
- Echo suppression for Telegram-sent messages
- HTML escaping with plaintext fallback
- Cron watchdog support
- Allowlist-based security

### Architecture
- Pure bash + curl + jq
- JSONL byte-offset tracking (survives restarts)
- Background watchers per session (one subshell each)
- Telegram Bot API long polling
