# 🤖 LinkedIn AI Outreach Automation — n8n System

> A fully automated, end-to-end LinkedIn lead generation and meeting booking system built with **n8n**, **Unipile API**, **OpenAI**, and **Google Workspace**. From prospect discovery to booked calendar meeting — completely hands-free.

---

## 📌 What This Does

This system runs two parallel n8n workflows that work together to automate your entire LinkedIn outreach funnel:

| | Flow 1 — Prospect Finder | Flow 2 — Conversation Handler |
|---|---|---|
| **Trigger** | Every 5 hours | Every 30 minutes |
| **Job** | Find & connect with new prospects | Nurture conversations & book meetings |
| **Output** | Connection requests sent + logged | Meetings booked on Google Calendar |

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      FLOW 1 — PROSPECT FINDER                   │
│                                                                 │
│  ⏰ Every 5 Hours                                               │
│       │                                                         │
│       ▼                                                         │
│  🔍 Search LinkedIn (Unipile API)                               │
│       │  keywords: startup founder, automation manager          │
│       │  locations: US, UK, UAE, Canada, Australia              │
│       ▼                                                         │
│  📋 Fetch Existing Sheet + Sent Invites (dedup check)           │
│       │                                                         │
│       ▼                                                         │
│  🧠 AI Score Each Prospect (GPT)                                │
│       │  Score 1–10 based on title, company, relevance          │
│       ▼                                                         │
│  ✅ Quality Filter: Score ≥ 6                                   │
│       │                                                         │
│       ▼                                                         │
│  📨 Send LinkedIn Connection Request (Unipile API)              │
│       │                                                         │
│       ▼                                                         │
│  📊 Log to Google Sheet (Name, Score, Status, Cursor)           │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                  FLOW 2 — CONVERSATION HANDLER                  │
│                                                                 │
│  ⏰ Every 30 Minutes                                            │
│       │                                                         │
│       ▼                                                         │
│  📋 Load Active Prospects from Google Sheet                     │
│       │  (status: Connection Sent / Connected)                  │
│       ▼                                                         │
│  🔗 Check Connection Status (Unipile API)                       │
│       │                                                         │
│   ┌───┴──────────────────┐                                      │
│   │ Not Connected → Skip │                                      │
│   └──────────────────────┘                                      │
│       │ Connected ✓                                             │
│       ▼                                                         │
│  💬 Fetch LinkedIn Chat & Analyze Conversation                  │
│       │                                                         │
│   ┌───┴──────────────────────────────────────┐                  │
│   │ No Message Yet → Send Personalized Pitch  │                 │
│   └──────────────────────────────────────────┘                  │
│       │ Message exists                                          │
│       ▼                                                         │
│  🤖 AI Conversation Handler                                     │
│       │  + Google Calendar Availability Check                   │
│       │  + Free Slot Detection                                  │
│       ▼                                                         │
│  📅 If Meeting Agreed → Create Google Meet + Send Link          │
│       │                                                         │
│       ▼                                                         │
│  📊 Update Google Sheet Status                                  │
│       (Meeting Booked / Not Interested / Active)                │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| **n8n** | Workflow automation engine (self-hosted or cloud) |
| **Unipile API** | LinkedIn search, connection requests, messaging |
| **OpenAI (GPT-4)** | Prospect scoring + AI conversation replies |
| **Google Sheets** | Central CRM / prospect tracking database |
| **Google Calendar** | Check real-time availability for meetings |
| **Google Meet** | Auto-create meeting events and links |

---

## 📂 Repository Structure

```
linkedin-automation-n8n/
│
├── flow1_prospect_finder_V2_-_Without_Credentials.json    # Flow 1: Prospect Finder
├── flow2_conversation_handler_V2_-_Without_Credentials.json  # Flow 2: Conversation Handler
│
├── README.md                    # This file
└── google-sheet-template/
    └── SHEET_COLUMNS.md         # Required column structure for Google Sheet
```

---

## 📊 Google Sheet — Required Columns

Your Google Sheet (tab name: `Prospects`) must have these columns:

| Column | Description |
|--------|-------------|
| `Name` | Prospect's full name |
| `Title` | LinkedIn headline/job title |
| `Company` | Company name |
| `LinkedIn URL` | Profile URL (used as unique key) |
| `Provider ID` | Unipile internal ID |
| `AI Score` | GPT score out of 10 |
| `AI Reason` | Why AI gave that score |
| `Connection Status` | `Connection Sent` / `Connected` |
| `Conversation Status` | `Pitch Sent` / `Active` / `Meeting Booked` / `Not Interested - Closed` |
| `Date Sent` | Date connection was sent |
| `Location` | Prospect's location |
| `Last Updated` | Timestamp of last automation action |
| `Email` | Auto-populated from Unipile if available |
| `Cursor` | Pagination cursor for LinkedIn search |

---

## ⚙️ Setup Guide

### Prerequisites

- n8n instance (self-hosted via Docker, or [n8n.cloud](https://n8n.io))
- [Unipile](https://www.unipile.com/) account with LinkedIn account connected
- OpenAI API key
- Google account with Sheets, Calendar, and Meet enabled

---

### Step 1 — Import Both Workflows

1. Open your n8n instance
2. Click **"Import from file"** (top right menu)
3. Import `flow1_prospect_finder_V2_-_Without_Credentials.json`
4. Repeat for `flow2_conversation_handler_V2_-_Without_Credentials.json`

---

### Step 2 — Set Up Credentials

Go to **Settings → Credentials** in n8n and create:

**Google Sheets OAuth2**
- Type: `Google Sheets OAuth2 API`
- Authorize with your Google account

**Unipile API (HTTP Header Auth)**
- Type: `Header Auth`
- Header name: `X-API-KEY`
- Value: Your Unipile API key

**OpenAI**
- Type: `OpenAI API`
- API Key: Your OpenAI key

---

### Step 3 — Update Configuration Values

Search for `************************` in both workflow JSONs and replace:

| Placeholder | Replace With |
|-------------|-------------|
| `account_id: ****` | Your Unipile LinkedIn account ID |
| `X-API-KEY: ****` | Your Unipile API key |
| `myProviderId = '****'` | Your own LinkedIn Provider ID (from Unipile) |
| Google Sheet ID | Your Google Sheet document ID |

---

### Step 4 — Customize Your Search

In **Flow 1**, find the `Search LinkedIn Prospects` node and edit the JSON body:

```json
{
  "api": "classic",
  "category": "people",
  "keywords": "startup founder automation manager",
  "location": ["103644278", "102095887", "105080838", "101949407", "106981407"],
  "limit": 35
}
```

- Change `keywords` to match your target audience
- Update `location` with LinkedIn location codes for your target regions

**Common LinkedIn Location Codes:**
- `103644278` — United States
- `102095887` — United Kingdom
- `105080838` — Australia
- `101949407` — Canada
- `106981407` — United Arab Emirates
- `102713980` — Pakistan

---

### Step 5 — Customize Your Pitch Message

In **Flow 2**, find the `Send Initial Pitch Message` node and update the `text` field:

```
Hey {FirstName}! Thanks for connecting 🙌

I help businesses like yours eliminate repetitive manual tasks through 
smart automation — saving 10–20+ hours per week.

Are you currently handling any processes manually that you'd love to automate?
```

Make it your own — keep it short, personal, and question-based.

---

### Step 6 — Activate Both Workflows

1. Open each workflow
2. Toggle the **Active** switch (top right)
3. Flow 1 will run every 5 hours, Flow 2 every 30 minutes

---

## 🔄 How the Two Flows Work Together

```
Flow 1 runs → Writes prospects to Google Sheet
                        │
                        ▼
              Flow 2 reads from the same Sheet
                        │
                        ▼
              Handles conversations + books meetings
```

The **Google Sheet is the shared brain** — Flow 1 writes, Flow 2 reads and updates.

---

## 📈 Expected Results (Based on Typical Usage)

| Metric | Approximate Range |
|--------|------------------|
| Prospects searched per run | ~35 per 5 hours |
| AI filter pass rate | ~60–70% (score ≥ 6) |
| Connection acceptance rate | 20–40% (varies by niche) |
| Reply rate from pitch | 10–25% |
| Meeting booking rate | 5–15% of replies |

> Results vary based on your niche, message quality, and LinkedIn account warmth.

---

## 🚨 Important Notes

**LinkedIn Limits:** LinkedIn has daily connection and message limits. Keep your volumes reasonable:
- Max ~20–30 connection requests/day
- Max ~10–15 messages/day
- Use rate limit delay nodes (already included in Flow 1)

**Unipile Account:** Make sure your LinkedIn account is properly warmed up before running at full volume.

**No Credentials Included:** All API keys and IDs have been removed from these workflow files. Never commit credentials to GitHub.

---

## 🤝 Contributing

Found a bug or want to improve the system? Feel free to:
- Open an issue
- Submit a pull request
- Fork and customize for your use case

---

## 📬 Connect With Me

Built by **Muhammad Sohaib**

- 🔗 LinkedIn: [[your-linkedin-url](https://www.linkedin.com/in/muhammad-s-71a050203?utm_source=share&utm_campaign=share_via&utm_content=profile&utm_medium=ios_app)]
- 💻 GitHub: [https://github.com/Shubeetheanalyst/]
- 📧 Email: [hafizsohaib478@gmail.com]

---

## 📄 License

MIT License — free to use, modify, and distribute. Attribution appreciated.

---

> ⭐ If this helped you, please star the repo — it helps others find it!
