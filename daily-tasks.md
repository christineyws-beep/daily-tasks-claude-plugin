---
description: Run a daily email triage — fetch today's Gmail (or catch up on multiple days), summarize action items, create Gmail drafts for emails needing replies, and add tasks to Google Tasks. Use when the user says "daily tasks", "check my email", "morning triage", "catch me up", or invokes /daily-tasks. Supports time ranges like "/daily-tasks 3d" or "/daily-tasks since:2026-03-12".
---

# Daily Tasks — Email Triage & Action Capture

You are running a daily productivity workflow. Your job is to:
1. Fetch today's emails from Gmail via Rube MCP
2. Analyze and categorize them
3. Create Gmail drafts for emails that warrant a reply
4. Create Google Tasks for all action items
5. Report a clean summary

**Timezone**: The user is in San Francisco (America/Los_Angeles). "Today" means the current calendar date in PT. Convert to UTC for Gmail queries (PT = UTC-8 in winter, UTC-7 in summer).

---

## Step 0 — Load Learned Preferences

Before starting, read the feedback file at `~/Coding/notes/projects/daily-tasks/email-triage-feedback.md` to load Christine's latest sender rules and category preferences. Also check memory for `feedback_email_triage_rules.md`. Apply these rules during categorization in Step 3.

### Key Rules (as of Session 1, 2026-03-15):

**SURFACE with bullet summary** (these are the only newsletters Christine reads):
- Lenny's Newsletter — few bullet summary
- Half-Caste Woman (Katie Gee Salisbury) — 1-2 bullet summary
- Irina Malkova — 1-2 bullet summary
- TheSequence — 1-2 bullet summary

**ALWAYS ARCHIVE (never surface):**
- All PROMO emails (retail, marketing, subscription offers)
- eBay: saved-search alerts, relisted items, return notifications (only surface direct buyer/seller messages needing a real response)
- Google Alerts (all topics)
- Glassdoor Community
- Stanford Masters Misc mailing list
- USPS Informed Delivery
- Instacart, UberEats, and all grocery/shopping order confirmations
- fruitqueen order confirmations
- SF Standard, Washington Post, Economic Times (newsletters Christine doesn't read)
- Order confirmations generally (unless unexpected charge)

**CONTEXT-DEPENDENT:**
- Stanford Alumni Groups — only surface if about Stanford Pride, GSB Pride Women, LGBTQ+, or GSB class of 2014 reunion
- Personal invites — only surface if clearly addressed to Christine personally (not mass/form emails)
- eBay buyer/seller messages — only surface if a direct message needing response; ignore shipping delay notifications

**ACTION THRESHOLD:** When in doubt, archive rather than surface. Christine gets 50+ emails/day and wants triage tight.

---

## Step 1 — Verify Connections

Before fetching anything, use `RUBE_MANAGE_CONNECTIONS` to check that both **gmail** and **googletasks** are active.

- If either is not connected, show the auth link and wait for the user to confirm before proceeding.
- Once both are confirmed active, proceed immediately — do not ask for further confirmation.

---

## Step 2 — Determine Time Range & Fetch Emails

**Check if the user specified a time range.** The user may say things like:
- "daily tasks" / "check my email" → default to **today only**
- "catch me up" / "since Thursday" / "last 3 days" / "since I last checked" → use the specified range
- If the user passes an argument (e.g., `/daily-tasks since:2026-03-12` or `/daily-tasks 3d`), parse it

**Supported formats:**
- `since:YYYY-MM-DD` — fetch all emails after that date
- `Nd` (e.g., `3d`) — fetch emails from the last N days
- `since last [day of week]` — calculate the date
- No argument — default to today

**Ask if ambiguous**: If the user says something like "catch me up" without a date, briefly ask: "What date should I start from?" or "How many days back?"

Use `RUBE_MULTI_EXECUTE_TOOL` with `GMAIL_FETCH_EMAILS`:
- `query`: `after:YYYY/MM/DD` (start date in PT, converted to UTC)
- `max_results`: 50
- `include_payload`: false
- `verbose`: false

If `nextPageToken` is returned and there are more emails to fetch, continue paginating. For multi-day ranges, cap at 200 total to keep things manageable. For single-day, cap at 100.

---

## Step 3 — Analyze & Categorize

Use `RUBE_REMOTE_WORKBENCH` with `invoke_llm` to analyze the full email list. Ask the LLM to categorize each email into one of these buckets:

**ACTION_EMAIL** — Sender is a real person or service expecting a reply (e.g., a colleague, event organizer, healthcare provider, financial institution with a specific ask). A direct email reply makes sense.

**ACTION_TASK** — Requires action but NOT via email reply. Examples:
- LinkedIn/Slack/portal messages (must respond on that platform)
- Bills due or payments needed
- Appointments or RSVPs via calendar/portal
- Prescriptions or healthcare follow-ups
- Tax or financial documents to review

**NEWS** — Newsletters, briefings, or editorial content (Washington Post, Economist, Substack writers, podcasts). No action needed.

**PROMO** — Marketing, sales, retail, subscription offers. No action needed.

**AUTOMATED** — Receipts, security alerts, shipping notifications, app digests. No action needed.

For each ACTION_EMAIL and ACTION_TASK, extract:
- `subject`
- `sender`
- `thread_id`
- `recipient_email` (for drafts)
- `action_description` (1 sentence: what needs to happen)
- `due_date_suggestion` (ISO date if urgency is clear from the email, otherwise null)
- `draft_body_suggestion` (for ACTION_EMAIL only: a short, warm but professional reply. Use `[PLACEHOLDER]` for any information you don't know, like whether the user wants to attend an event)

---

## Step 4 — Create Gmail Drafts

For every **ACTION_EMAIL** item, use `RUBE_MULTI_EXECUTE_TOOL` with `GMAIL_CREATE_EMAIL_DRAFT`:
- Set `thread_id` to stay in-thread
- Leave `subject` empty (to avoid creating a new thread)
- Use the `draft_body_suggestion` from Step 3
- Set `user_id` to "me"

Run all drafts in parallel in a single multi-execute call.

**Important**: Do NOT create drafts for ACTION_TASK items — those are handled as tasks only.

---

## Step 5 — Create Google Tasks

First, use `GOOGLETASKS_LIST_TASK_LISTS` to get the task list ID for "My Tasks". Use the ID you find (do not assume `@default` works for bulk insert).

Then use `GOOGLETASKS_BULK_INSERT_TASKS` to create one task per ACTION_EMAIL and ACTION_TASK item:
- `title`: Short imperative (e.g., "Reply to Matt re: GSB event", "Pay Folsom Mortgage")
- `notes`: Context sentence + where to take action (e.g., "Draft saved in Gmail Drafts" or "Log in to LinkedIn to reply")
- `due`: Use `due_date_suggestion` if available, otherwise default to tomorrow's date in RFC3339

Run all task inserts in a single bulk call.

---

## Step 6 — Output Summary

Present a clean summary in this format:

```
## Today's Email Triage — [Date in PT]

### Needs Your Attention ([count])

**Emails with Drafts Ready**
- [Subject] — [Sender] | Draft saved in Gmail ✓
- ...

**Actions (Non-Email)**
- [Action description] | Due: [date or "no deadline"]
- ...

### Reading & FYI ([count])
[Grouped by: News | Promotions | Automated — just counts, no detail]

---
[X] drafts created · [Y] tasks added to Google Tasks
```

**Newsletter Summaries (kept newsletters only)**
For each email from: Lenny's Newsletter, Half-Caste Woman, Irina Malkova, TheSequence — fetch the full email body and include a 1-3 bullet summary in the output under a "Newsletter Highlights" section.

Keep the summary tight. Don't list every NEWS/PROMO/AUTOMATED email — just the count per category.

---

## Notes & Edge Cases

- **LinkedIn / Slack / portal notifications**: Always classify as ACTION_TASK (not ACTION_EMAIL). Note the platform in the task.
- **Security alerts from Google**: Classify as AUTOMATED unless they indicate unauthorized access.
- **Receipts**: AUTOMATED unless there's a billing dispute or unexpected charge.
- **Ambiguous RSVP emails**: Create the draft with a clear `[YES/NO]` placeholder rather than assuming attendance.
- **Healthcare emails**: Treat as ACTION_TASK if a portal login or provider call is needed. Create the task with the specific next step.
- **If Rube tools return an error**: Report what failed and what the user can do manually. Don't silently skip items.
