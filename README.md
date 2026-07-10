# N8n Inbound Lead Qualification System

A production-ready automation pipeline that captures inbound leads from multiple sources, deduplicates them, scores them with AI, routes them by priority, escalates ignored leads, and generates weekly reports.

## Overview

Built entirely on self-hosted n8n with Groq as the AI backend. No paid middleware, no Zapier, no Make.com.

## Architecture — 3 Workflows

### 1. B2B Main Workflow
Captures leads from 3 sources and runs them through a full qualification pipeline.

**Triggers:**
- Webhook — contact form submissions
- Gmail Trigger — email inquiries
- Google Sheets Trigger — LinkedIn exports

**Pipeline:**
- Per-source normalization into `{name, email, message, source}`
- Deduplication against existing leads in data table
- AI scoring via Groq (score 1-10 with intent classification)
- Conditional routing by score
- Discord notifications for high-priority leads
- Logging to 3 separate data tables

### 2. Escalation Workflow
Runs every 30 minutes. Detects high-score leads that haven't received a response within 2 hours and sends an escalation alert to Discord.

### 3. Error Workflow
Catches any uncaught node failure in the main workflow and sends an alert to Discord with the failed node name and error message.

### Weekly Summary (inside B2B)
Runs every Monday morning. Aggregates all leads from the past week and sends a formatted report to Discord with counts by source, score bucket, and escalation status.

## AI Scoring

Model: Groq (openai/gpt-oss-20b)

Scores 1-10:
- 1-2: Spam, recruiter pitch, junk
- 3-4: Low quality, vague, no real need
- 5-6: Moderate interest, thin on detail
- 7-8: Clear need, genuine intent
- 9-10: High priority, urgency + budget/timeline stated

Intent classification: `genuine_inquiry`, `vague_interest`, `spam_or_pitch`, `not_a_lead`

Edge cases handled:
- Messages under 5 words cap at score 4
- Urgency without stated need caps at 4
- Invalid/malformed emails score ≤2
- Non-English messages judged on content not language
- Forwarded email boilerplate ignored

## Data Tables

| Table | Contents |
|---|---|
| AllClients | All leads: name, email, source, isDuplicate, message |
| RepB2B | High-score leads: name, email, score, intent, summary, notified_at, responded, escalated |
| ArchivB2B | Low/moderate leads: score, intent, summary, email, source, name |
| LogInsert | Full log of all processed leads |

## Tech Stack

- n8n (self-hosted, npm, Windows)
- Groq Cloud (openai/gpt-oss-20b)
- Discord (notifications, escalation, weekly report, error alerts)
- Gmail
- Google Sheets
- Webhooks
- n8n Data Tables

## Edge Cases Tested

- Empty table on first run
- Missing name field → defaults to "Unknown"
- Invalid/malformed emails → score ≤2
- Spam and recruiter pitches → score 1-2
- Non-English messages (French, Arabic, German)
- Emoji and special characters
- Duplicate submissions across sources
- Short/empty messages → cap at 4
- Urgency without stated need → cap at 4

## Test Results

- 10 webhook leads via bash script
- 7 Google Sheets leads
- 2 Gmail leads
- 19 unique leads processed end-to-end
- Escalation fired correctly on unresponded high-score leads
- Weekly report confirmed correct counts

## Known Limitations

- Race condition on near-simultaneous duplicate submissions from different sources
- `responded` field marked manually — no automated mechanism
- Gmail messages contain raw HTML entities — not cleaned
- `AllClients` fetches all rows on every lead — doesn't scale indefinitely

## Setup

1. Import the 3 workflow JSON files into your n8n instance
2. Reconnect credentials (Groq, Gmail, Google Sheets, Discord)
3. Create the 4 data tables with the correct column names
4. Update webhook URLs and spreadsheet IDs
5. Activate all 3 workflows

## Author

Ahmed Yassine Loussaief — n8n Automation Specialist
[LinkedIn](https://www.linkedin.com/in/ahmedyassine-loussaief-6393a7339/) | 