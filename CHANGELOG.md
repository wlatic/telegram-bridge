# Changelog

## 0.5 (2026-03-26)

Initial release.

### Features
- Bidirectional JSONL transcript monitoring
- Forum Topics per tmux session
- Commands: /sessions, /new, /resume, /kill, /approval, /screenshot, /help
- Working... indicator with dots, ✓ Done on idle
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
