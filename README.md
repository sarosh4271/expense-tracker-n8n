# 💸 AI Expense Tracker — n8n + Gemini + Google Sheets + Gmail

Message an expense in plain English, an AI agent extracts the amount, category and date and logs it to Google Sheets. Every Sunday, a second workflow reads the week's spending and emails a summary — no manual entry, no spreadsheet formulas.

Built entirely with free tools as a portfolio project to learn n8n and AI-agent workflows.

![status](https://img.shields.io/badge/status-working-brightgreen) ![n8n](https://img.shields.io/badge/built%20with-n8n-orange) ![cost](https://img.shields.io/badge/cost-%240-blue)

## Demo

https://github.com/user-attachments/assets/REPLACE_WITH_YOUR_UPLOADED_VIDEO_LINK

*(Upload `demo.mp4` to a GitHub issue or release to get a shareable link, then paste it here — GitHub auto-embeds it.)*

## How it works

**Logger workflow** — `Chat Trigger → AI Agent → IF → Google Sheets → Reply`
1. You type something like `"bought petrol for 1500"` into a hosted n8n chat.
2. An AI Agent (OpenAI `gpt-5-nano`) reads the message and returns structured JSON: `date`, `amount`, `category`, `note`.
3. An IF node checks a valid amount was found.
4. Valid → the row is appended to Google Sheets and the bot replies with a confirmation.
5. Invalid (e.g. "hello", no amount) → the bot replies with usage instructions instead.

**Reporter workflow** — `Schedule Trigger → Google Sheets → Code → AI Chain → Gmail`
1. Every Sunday evening, the workflow reads every row in the sheet.
2. A Code node filters rows to the last 7 days and does the arithmetic — total spend, per-category breakdown, biggest expense. (Math is deliberately done in code, not by the model — LLMs are unreliable at summing numbers.)
3. That summary is handed to an AI chain, which writes a short, friendly report from the numbers.
4. Gmail sends the report to my inbox.

## Why AI extraction instead of regex

An earlier version parsed messages with a keyword list and regex, which broke on anything not written exactly as expected. Structured output from an LLM (JSON in, JSON out, enforced with a schema) handles messy input like `"1200 for uber yesterday"` or `"paid the electricity bill, around 4500"` without hardcoding phrasing.

## Stack

| Component | Tool | Cost |
|---|---|---|
| Automation | [n8n](https://n8n.io) (cloud trial / self-hosted) | Free |
| AI extraction & summary | OpenAI (`gpt-5-nano`) | Free tier / pay-as-you-go |
| Data store | Google Sheets | Free |
| Chat input | n8n hosted Chat Trigger | Free |
| Output | Gmail | Free |

## Google Sheet layout

Sheet name: `Expenses`, column A formatted as **Plain text** (prevents auto date-reformatting):

| Date | Amount | Category | Note | Raw |
|---|---|---|---|---|
| 2026-09-28 | 850 | Food | lunch | 4f2a1c9e-... |

`Raw` stores the n8n execution ID (not the original message) — it's used as the matching column so re-running an execution updates its row instead of duplicating it.

## Setup

1. **Google Sheet:** create a sheet with the headers above; set column A to Plain text.
2. **OpenAI API key:** from [platform.openai.com](https://platform.openai.com/api-keys).
3. **n8n:** import `logger-workflow.json` and `reporter-workflow.json` (in this repo) into a new n8n instance.
4. **Credentials:** add your OpenAI key and connect Google Sheets + Gmail via OAuth in n8n's credential manager.
5. **Logger:** open the Chat Trigger node → enable *Make Chat Publicly Available* → publish the workflow → open the hosted chat URL.
6. **Reporter:** set the Schedule Trigger to your preferred day/time and timezone, then activate.

## What I'd improve next

- Multi-expense messages in one line ("lunch 850, uber 400")
- A `/undo` command to delete the last logged row
- Budget alerts when a category passes a monthly limit
- Self-hosting on a free-tier VM for an always-on webhook (no tunnel/URL churn)

## Files

- `logger-workflow.json` — expense logging workflow (import into n8n)
- `reporter-workflow.json` — weekly summary workflow (import into n8n)
- `demo.mp4` — 30-second walkthrough

---

Built as a learning project. Feedback welcome — feel free to open an issue or fork it.
