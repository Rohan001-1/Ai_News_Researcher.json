# AI News Finder — Automated WhatsApp News Digest

An n8n workflow that automatically finds the latest AI news twice daily, filters out previously sent stories, and delivers a clean formatted digest to WhatsApp using Green API.

---

## How It Works

```
Schedule Trigger (8AM / 7PM)
        ↓
News Finder And Definer (AI Agent)
  ├── Tavily Search Tool
  └── Simple Memory (deduplication)
        ↓
   Success                   Error
     ↓                          ↓
Data Formatter           Error Understander
(AI Agent)               (AI Agent)
     ↓                          ↓
Success HTTP             Error HTTP
(WhatsApp)               (WhatsApp Alert)
```

---

## What Each Node Does

**Schedule Trigger** — Fires automatically at 8:00 AM and 7:00 PM every day.

**News Finder And Definer** — The main AI agent. Searches Tavily for fresh AI news, checks memory to avoid duplicate stories, and outputs structured JSON with summaries.

**Search in Tavily** — Gives the News Finder agent web search capability. Searches for latest AI agent tools, model releases, and automation news.

**Simple Memory** — Stores previously sent news titles so the same story is never sent twice. Remembers the last 5 conversations by default.

**Data Formatter** — Takes the JSON output from News Finder and converts it into a clean, readable WhatsApp message with emojis, sections, and a daily agent building tip.

**Error Understander** — If News Finder fails, this agent reads the raw error, explains what went wrong in plain English, and suggests a quick fix.

**Date & Time Tool** — Used by Error Understander to include the current time in the alert message.

**Success HTTP** — Sends the formatted news digest to WhatsApp via Green API.

**Error HTTP** — Sends the error alert to WhatsApp via Green API.

---

## Credentials Required

| Node | Credential Type |
|------|----------------|
| News Finder And Definer | Mistral Cloud API |
| Data Formatter | Mistral Cloud API |
| Error Understander | Mistral Cloud API |
| Search in Tavily | Tavily API |
| Success HTTP + Error HTTP | Green API (configured manually in node) |

---

## Green API Setup (WhatsApp)

Green API is used to send WhatsApp messages from n8n.

1. Go to [console.green-api.com](https://console.green-api.com) and create an account using your email
2. Click **Create Instance** and select a plan (free developer plan is available)
3. Once the instance is created, open it and scan the QR code using WhatsApp on your phone: **WhatsApp → Settings → Linked Devices → Link a Device**
4. After scanning, your instance status will change to **Authorized**
5. Enable the **"Receive webhooks on incoming messages and files"** option in instance settings
6. Copy your **idInstance** and **apiTokenInstance** from the instance dashboard

### Configure the HTTP Nodes

Open both **Success HTTP** and **Error HTTP** nodes and update the URL and body:

**URL format:**
```
https://api.green-api.com/waInstance[YOUR_ID_INSTANCE]/sendMessage/[YOUR_API_TOKEN_INSTANCE]
```

**chatId format:**
```
91XXXXXXXXXX@c.us
```
- `91` = India country code (change based on your country)
- `XXXXXXXXXX` = 10 digit WhatsApp number (no spaces or dashes)
- `@c.us` = required suffix, always add this

---

## Customizing the AI Prompts

Each AI agent has a system prompt you can edit to change behavior:

**News Finder And Definer** — Edit the search queries, number of news items, output JSON format, or deduplication logic.

**Data Formatter** — Edit the WhatsApp message layout, sections, tone, language, or the daily tip topics.

**Error Understander** — Edit the error alert format, language, or the level of technical detail shown.

To edit: open the agent node → click the **System Message** field → update the text.

---

## Adjusting the Schedule

Open the **Schedule Trigger** node and change `triggerAtHour` values to your preferred times. Currently set to `8` (8:00 AM) and `19` (7:00 PM).

---

## Notes

- Activate the workflow after setup — it will not run on schedule in inactive state
- Tavily free plan has a monthly search limit — monitor usage if running twice daily
- Simple Memory resets if n8n restarts — this is expected behavior for buffer memory
- If the same news appears repeatedly, reduce `contextWindowLength` in the Simple Memory node to a lower value like `5`
