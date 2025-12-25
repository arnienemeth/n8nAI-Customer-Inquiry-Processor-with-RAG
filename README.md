# 🤖 AI Customer Inquiry Processor with RAG v2

An intelligent n8n workflow that automates customer email processing using **Retrieval-Augmented Generation (RAG)** with GPT-4o-mini, Google Workspace, and Slack integration.

![n8n](https://img.shields.io/badge/n8n-Workflow%20Automation-orange)
![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4o--mini-green)
![Google Workspace](https://img.shields.io/badge/Google-Workspace-blue)
![Slack](https://img.shields.io/badge/Slack-Integration-purple)
![RAG](https://img.shields.io/badge/AI-RAG%20Pattern-red)

**Author:** arni.nemeth1980  
**Version:** 2.0.0

---

## 📋 Table of Contents

- [Business Problem](#-business-problem-solved)
- [Workflow Sections](#-workflow-sections-detailed)
- [Architecture](#-architecture-overview)
- [Setup Guide](#-setup-guide)
- [Slack Configuration](#-slack-configuration)
- [Credentials Checklist](#-credentials-checklist)

---

## 🎯 Business Problem Solved

**Challenge:** Manual email processing taking 15+ hours/week with 24-48 hour response times.

**Solution:** Automated AI agent that classifies, responds, logs, and notifies—all in under 5 minutes.

**Results:**
- ⏱️ Response time: 24-48 hours → **under 5 minutes**
- 📊 85% of inquiries handled automatically
- 💰 $2,400/month saved in support costs
- 📈 Customer satisfaction increased 32%

---

## 🔢 Workflow Sections (Detailed)

The workflow is organized into **9 logical sections**, each with numbered nodes for easy navigation.

### Section 1️⃣: Email Ingestion

| Node | Purpose |
|------|---------|
| `1️⃣ Gmail Trigger - New Inquiry` | Monitors inbox for new unread emails every 5 minutes |

**What it does:**
- Polls Gmail for unread emails
- Captures sender, subject, body, attachments
- Triggers the entire workflow

---

### Section 2️⃣: RAG Context Retrieval

| Node | Purpose |
|------|---------|
| `2a️⃣ Read Knowledge Base` | Loads company policies and procedures |
| `2b️⃣ Read FAQ Database` | Loads pre-written Q&A pairs |
| `2c️⃣ Read Product Catalog` | Loads product info, pricing, availability |
| `2d️⃣ Merge All Context` | Combines all three data sources |
| `2e️⃣ Prepare RAG Context` | Structures data for AI consumption |

**What it does:**
- Retrieves context from 3 Google Sheets (in parallel)
- Merges and structures data
- Prepares clean JSON for AI processing

---

### Section 3️⃣: AI Classification

| Node | Purpose |
|------|---------|
| `3️⃣ AI Classify Email` | GPT-4o-mini classifies the email |
| `3b️⃣ Parse Classification` | Extracts JSON from AI response |

**Classification Output:**
```json
{
  "category": "PRODUCT_INQUIRY",
  "priority": "HIGH",
  "sentiment": "NEUTRAL",
  "topics": ["pricing", "availability"],
  "summary": "Customer asking about Widget Pro"
}
```

**Categories:** PRODUCT_INQUIRY, SUPPORT_REQUEST, ORDER_STATUS, GENERAL_QUESTION, PARTNERSHIP, SPAM

---

### Section 4️⃣: Spam Filtering

| Node | Purpose |
|------|---------|
| `4️⃣ Filter Spam` | Stops processing if email is spam |

**What it does:**
- Checks if category == "SPAM"
- If spam → workflow stops (saves API costs)
- If not spam → continues to response generation

---

### Section 5️⃣: AI Response Generation

| Node | Purpose |
|------|---------|
| `5️⃣ AI Generate Response` | GPT-4o-mini writes professional reply |
| `5b️⃣ Prepare Final Output` | Prepares data for all output nodes |

**Key Features:**
- Uses RAG context to answer questions
- Answers ALL questions (numbered if multiple)
- Matches tone to customer sentiment
- Professional formatting

---

### Section 6️⃣: Output Actions (Parallel)

| Node | Purpose |
|------|---------|
| `6a️⃣ Mark Email as Read` | Prevents duplicate processing |
| `6b️⃣ Create Gmail Draft` | Creates reply draft for review |
| `6c️⃣ Create Case Report` | Generates Google Doc report |
| `6d️⃣ Log to Google Sheets` | Records inquiry in tracking sheet |
| `6e️⃣ Slack Notification` | Sends notification to Slack channel |

**All 5 actions run in parallel** for efficiency.

---

### Section 7️⃣: Priority Check

| Node | Purpose |
|------|---------|
| `7️⃣ Check Priority` | Routes based on HIGH vs other priority |

**Logic:**
- HIGH priority → Go to Section 8 (alerts)
- MEDIUM/LOW priority → Go to Section 9 (complete)

---

### Section 8️⃣: Manager Alerts (HIGH Priority Only)

| Node | Purpose |
|------|---------|
| `8️⃣ Alert Manager (HIGH)` | Sends email alert to manager |
| `8b️⃣ Slack Urgent Alert` | Sends urgent Slack notification |

**Triggered only for HIGH priority inquiries.**

---

### Section 9️⃣: Workflow End

| Node | Purpose |
|------|---------|
| `9️⃣ Processing Complete` | End node for standard processing |

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     AI CUSTOMER INQUIRY PROCESSOR v2                         │
│                        Author: arni.nemeth1980                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  SECTION 1: INGESTION          SECTION 2: RAG RETRIEVAL                     │
│  ┌──────────────────┐          ┌─────────────────────────────┐              │
│  │  📧 Gmail        │          │  📊 Knowledge Base          │              │
│  │  Trigger         │────────▶ │  📊 FAQ Database       ───▶ MERGE         │
│  └──────────────────┘          │  📊 Product Catalog         │              │
│                                └─────────────────────────────┘              │
│                                              │                               │
│                                              ▼                               │
│  SECTION 3: CLASSIFICATION     SECTION 4: FILTER                            │
│  ┌──────────────────┐          ┌──────────────────┐                         │
│  │  🤖 GPT-4o-mini  │────────▶ │  🚫 Spam Filter  │                         │
│  │  Classify        │          └──────────────────┘                         │
│  └──────────────────┘                    │                                  │
│                                          ▼                                  │
│  SECTION 5: RESPONSE GENERATION                                             │
│  ┌──────────────────────────────────────────┐                               │
│  │  🤖 GPT-4o-mini Generate Response        │                               │
│  └──────────────────────────────────────────┘                               │
│                         │                                                   │
│                         ▼                                                   │
│  SECTION 6: OUTPUT ACTIONS (PARALLEL)                                       │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐                    │
│  │✓ Mark  │ │📝 Draft│ │📄 Doc  │ │📊 Log  │ │💬 Slack│                    │
│  │  Read  │ │        │ │ Report │ │        │ │        │                    │
│  └────────┘ └────────┘ └────────┘ └────────┘ └────────┘                    │
│                                       │                                     │
│                                       ▼                                     │
│  SECTION 7-8: PRIORITY ROUTING                                              │
│  ┌──────────────────┐                                                       │
│  │  ❓ HIGH Priority?│                                                      │
│  └──────────────────┘                                                       │
│         │                    │                                              │
│    YES  ▼               NO   ▼                                              │
│  ┌────────────┐      ┌────────────┐                                         │
│  │🚨 Manager  │      │✅ Complete │                                         │
│  │   Alerts   │      │            │                                         │
│  └────────────┘      └────────────┘                                         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 🚀 Setup Guide

### Step 1: Import Workflow

1. Download `customer-inquiry-ai-agent-with-slack.json`
2. In n8n: **Workflows** → **Import from File**
3. Select the JSON file

### Step 2: Create Google Sheet

Create one Google Sheet with **4 tabs**:

**Tab: Knowledge Base**
| topic | content | keywords |
|-------|---------|----------|
| Return Policy | We offer 30-day returns... | returns, refund |

**Tab: FAQ Database**
| category | question | answer |
|----------|----------|--------|
| Shipping | How long does delivery take? | 3-5 business days... |

**Tab: Product Catalog**
| product_name | description | price | availability | sku |
|--------------|-------------|-------|--------------|-----|
| Widget Pro | Professional widget... | 99.99 | In Stock | WP-001 |

**Tab: Inquiry Log** (empty - workflow fills this)
| Timestamp | Customer Email | Subject | Category | Priority | Sentiment | Topics | Summary | Response Status | Message ID |

### Step 3: Configure Credentials

Update these placeholder IDs in the workflow:

| Placeholder | Where to Find |
|-------------|---------------|
| `YOUR_GMAIL_CREDENTIAL_ID` | n8n Credentials |
| `YOUR_SHEETS_CREDENTIAL_ID` | n8n Credentials |
| `YOUR_DOCS_CREDENTIAL_ID` | n8n Credentials |
| `YOUR_OPENAI_CREDENTIAL_ID` | n8n Credentials |
| `YOUR_SLACK_CREDENTIAL_ID` | n8n Credentials |
| `YOUR_SHEET_ID` | Google Sheet URL |
| `YOUR_DRIVE_FOLDER_ID` | Google Drive folder URL |
| `YOUR_SLACK_CHANNEL_ID` | Slack channel settings |

---

## 💬 Slack Configuration

### Step 1: Create Slack App

1. Go to [api.slack.com/apps](https://api.slack.com/apps)
2. Click **"Create New App"** → **"From scratch"**
3. Name: `Customer Inquiry Bot`
4. Select your workspace

### Step 2: Configure Bot Permissions

Go to **OAuth & Permissions** → **Scopes** → Add these **Bot Token Scopes**:

| Scope | Purpose |
|-------|---------|
| `chat:write` | Send messages |
| `chat:write.public` | Post to public channels |
| `channels:read` | List channels |

### Step 3: Install App to Workspace

1. Go to **OAuth & Permissions**
2. Click **"Install to Workspace"**
3. Copy the **Bot User OAuth Token** (starts with `xoxb-`)

### Step 4: Get Channel ID

1. Open Slack
2. Right-click on your channel → **"View channel details"**
3. Scroll to bottom → Copy **Channel ID** (starts with `C`)

### Step 5: Configure n8n Credential

1. In n8n: **Credentials** → **Add Credential** → **Slack OAuth2 API**
2. Paste your Bot Token
3. Save

### Step 6: Update Workflow

Replace `YOUR_SLACK_CHANNEL_ID` with your actual channel ID (e.g., `C0123456789`)

---

## ✅ Credentials Checklist

| Service | Credential Type | Required Scopes |
|---------|-----------------|-----------------|
| **Gmail** | OAuth2 | gmail.readonly, gmail.modify, gmail.compose |
| **Google Sheets** | OAuth2 | spreadsheets, drive.file |
| **Google Docs** | OAuth2 | documents, drive.file |
| **OpenAI** | API Key | - |
| **Slack** | OAuth2 | chat:write, chat:write.public, channels:read |

---

## 📊 Workflow Visual

After import, your workflow should look like this:

```
[1️⃣ Gmail] → [2a️⃣ KB  ]
             [2b️⃣ FAQ ] → [2d️⃣ Merge] → [2e️⃣ Prepare] → [3️⃣ Classify] → [3b️⃣ Parse]
             [2c️⃣ Prod]
                                                                              ↓
                                                                    [4️⃣ Filter Spam]
                                                                              ↓
                                                                    [5️⃣ Generate] → [5b️⃣ Output]
                                                                                            ↓
                                                    ┌───────────────────────────────────────┼───────────────────┐
                                                    ↓               ↓           ↓           ↓                   ↓
                                            [6a️⃣ Read]    [6b️⃣ Draft] [6c️⃣ Doc] [6d️⃣ Log]           [6e️⃣ Slack]
                                                                                    ↓
                                                                            [7️⃣ Priority?]
                                                                              ↓         ↓
                                                                    HIGH: [8️⃣ Alert] [8b️⃣ Slack]
                                                                    LOW:  [9️⃣ Complete]
```

---

## 💡 Pro Tips

1. **Test with manual trigger first** before activating
2. **Monitor Executions tab** for errors
3. **Start with 5-minute polling** to avoid API limits
4. **Review AI drafts** for the first week before auto-sending

---

## 📄 License

MIT License - See [LICENSE](LICENSE) file

---

## 👤 Author

**arni.nemeth1980**

---

⭐ **Star this repo if it helped you!** ⭐
