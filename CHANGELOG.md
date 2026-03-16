# Changelog

## v0.2.0 — 2026-03-15

### Added
- **Step 0 — Learned preferences**: Skill now loads sender rules and category preferences before categorizing, making triage much tighter out of the box
- **Flexible time ranges**: Support for `/daily-tasks 3d`, `since:YYYY-MM-DD`, natural language ("catch me up since Thursday"), and `Nd` format
- **Newsletter summaries**: Automatically fetches full email content and generates bullet summaries for kept newsletters
- **Feedback loop**: Running feedback file (`email-triage-feedback.md`) with session-by-session corrections that compound over time
- **Pagination for multi-day ranges**: Fetches up to 200 emails for catch-up sessions

### Changed
- Step 2 renamed from "Fetch Today's Emails" to "Determine Time Range & Fetch Emails"
- Output now includes a "Newsletter Highlights" section for kept newsletters
- Description updated to reflect catch-up capability

### Sender Rules (example — customize for your inbox)
- 4 newsletters surfaced with summaries (Lenny's, Half-Caste Woman, Irina Malkova, TheSequence)
- eBay: only direct buyer/seller messages; all alerts/relists/returns archived
- Stanford Alumni: only Pride/LGBTQ+ or class reunion content
- All PROMO, Google Alerts, Glassdoor, grocery notifications auto-archived

## v0.1.0 — 2026-03-02

### Added
- Initial release
- Gmail fetch via Rube MCP
- 5-category classification (ACTION_EMAIL, ACTION_TASK, NEWS, PROMO, AUTOMATED)
- In-thread Gmail draft creation
- Google Tasks bulk insert
- Clean summary output
