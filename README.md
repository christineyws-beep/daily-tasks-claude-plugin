# Daily Tasks — Claude Code Email Triage Plugin

A Claude Code slash command (`/daily-tasks`) that triages your Gmail inbox, creates reply drafts, adds action items to Google Tasks, and summarizes kept newsletters — all in one shot.

## What it does

1. **Fetches emails** from Gmail (today, or catch up on multiple days)
2. **Categorizes** each email: actionable, newsletter, promo, or noise
3. **Creates Gmail drafts** for emails that need a reply (in-thread, with placeholders)
4. **Creates Google Tasks** for all action items (email and non-email)
5. **Summarizes** your kept newsletters with bullet points
6. **Reports** a clean summary of what needs your attention

## Setup

### Prerequisites
- [Claude Code](https://claude.com/claude-code) CLI installed
- Gmail and Google Tasks connected via [Rube MCP](https://composio.dev/)

### Install

Copy the skill file to your Claude Code commands directory:

```bash
cp daily-tasks.md ~/.claude/commands/daily-tasks.md
```

Then invoke it in Claude Code:

```
/daily-tasks          # triage today's emails
/daily-tasks 3d       # catch up on last 3 days
/daily-tasks since:2026-03-12   # everything since a date
```

Or just say "check my email" or "catch me up since Thursday".

## How it learns

The skill reads a feedback file (`email-triage-feedback.md`) before each run. After triage, you mark corrections — which emails should have been archived, which senders to always skip. The skill gets smarter each session.

### Customizing for your inbox

The skill ships with example rules. To personalize:

1. Run `/daily-tasks` once
2. Review the triage output
3. Edit `email-triage-feedback.md` with your corrections
4. Update the sender rules in Step 0 of `daily-tasks.md` to match your preferences

## Features

- **Flexible time ranges**: today, N days back, or since a specific date
- **Smart categorization**: ACTION_EMAIL, ACTION_TASK, NEWS, PROMO, AUTOMATED
- **In-thread drafts**: replies stay in the conversation, not as new threads
- **Newsletter summaries**: bullet-point highlights for newsletters you actually read
- **Feedback loop**: learns your preferences session by session
- **Google Tasks integration**: every action item gets a task with context and due date

## Roadmap

- [ ] Auto-archive noise emails after categorization
- [ ] Auto-unsubscribe from unwanted senders
- [ ] Calendar event detection (RSVPs, deadlines)
- [ ] De-duplicate across sessions
- [ ] Weekly digest mode

## License

MIT — see [LICENSE](LICENSE)
