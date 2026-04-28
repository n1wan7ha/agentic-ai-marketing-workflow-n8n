# Agentic AI Marketing Workflow — n8n

**Version:** 1.1  
**Built with:** n8n · Groq (Llama 3.3-70b) · Serper.dev · Hunter.io · Google Workspace (Sheets + Gmail)  
**Created by:** Niwantha Sithumal · 2026-01-27

---

## Project Overview

This n8n workflow automates end-to-end B2B lead generation and outreach. It performs the following autonomous steps:

1. **Scrapes Google** via Serper.dev to find companies in a target industry and city
2. **Verifies Emails** using Hunter.io to ensure high deliverability
3. **Human-in-the-Loop Review** — a manual approval step to accept or reject each lead
4. **Generates Personalized Emails** using Groq AI (Llama 3.3-70b)
5. **Sends Emails** via Gmail (OAuth2)
6. **Generates LinkedIn Posts** to market the outreach activity
7. **Database Management** — automatically saves and updates all data in Google Sheets

---

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running
- API keys for: Serper.dev, Hunter.io, Groq
- A Google Cloud project with Sheets & Gmail APIs enabled (for OAuth2)
- A Google Sheet created and ready to receive data

---

## Quick Start

### 1. Clone & Configure Environment

Copy the example env file and fill in your real API keys:

```powershell
Copy-Item .env.example .env
```

Then open `.env` and replace the placeholder values:

```env
SERPER_API_KEY=your_serper_api_key_here
HUNTER_API_KEY=your_hunter_api_key_here
GROQ_API_KEY=your_groq_api_key_here
GOOGLE_SHEET_ID=your_google_sheet_id_here
```

> **Finding your Google Sheet ID:**  
> Open your Google Sheet. The ID is in the URL:  
> `https://docs.google.com/spreadsheets/d/`**`THIS_IS_YOUR_SHEET_ID`**`/edit`

### 2. Start the Application

```powershell
docker-compose up -d
```

The n8n UI will be available at **http://localhost:5678**

### 3. First-Time Account Setup

On first launch, n8n will show a **"Set up owner account"** screen.  
Fill in your email, name, and password (8+ chars, 1 number, 1 capital letter), then click **Next**.

### 4. Import the Workflow

1. Go to **Overview → Workflows**
2. Click the folder icon (Import) or **Create workflow → Import from file**
3. Select `agentic-ai-mkt workflow.json` from the project root
4. Save the workflow

### 5. Configure OAuth2 Credentials

Some nodes require Google OAuth2 (Sheets + Gmail). To set these up:

1. Go to **http://localhost:5678/home/credentials**
2. Click **Add credential**
3. Add **Google Sheets OAuth2** — used by `Append row in sheet` and `Update row in sheet`
4. Add **Gmail OAuth2** — used by `Send a message`

> **Google Cloud Setup:**  
> - Create a project at [console.cloud.google.com](https://console.cloud.google.com)  
> - Enable **Google Sheets API** and **Gmail API**  
> - Create OAuth2 Client ID/Secret (Desktop or Web app type)  
> - Add `http://localhost:5678` as an authorized redirect URI

---

## Stopping & Restarting

```powershell
# Stop
docker-compose down

# Start again
docker-compose up -d

# View logs
docker-compose logs -f

# Check status
docker compose ps
```

> **Note for PowerShell users:** Use separate commands instead of `&&` (not supported in Windows PowerShell).

---

## How to Run the Workflow

You can trigger this workflow in **two ways**.

### Method 1: Manual Execution (Testing)

1. Open the workflow in the n8n editor
2. Click the **"Edit Fields"** node at the start
3. Paste the following into the input parameters:

```json
[
  {
    "client_description": "AI-powered marketing automation platform for B2B companies",
    "target_industry": "Software",
    "target_country": "Sri Lanka",
    "target_city": "Colombo"
  }
]
```

4. Click **Execute Workflow**

### Method 2: Form Submission (Production)

1. Ensure the **On Form Submission** trigger node is **active** (toggle on)
2. Click the **Production URL** in the trigger node settings
3. Fill in the form fields:
   - **Industry** (e.g., Software)
   - **City** (e.g., Colombo)
   - **Description** (e.g., We sell AI services...)
4. Submit the form — the workflow runs automatically in the background

---

## Handling the Human Approval Step

The workflow **pauses** at the **"Wait for Approval"** node for every lead found.

1. Go to **Executions** in the n8n sidebar
2. Open the **Pending Execution** (spinning or paused status)
3. Click the **Form URL** shown in the execution output
4. Choose:
   - **Yes – Save to Sheet** → processes and stores the lead
   - **No – Discard** → skips the lead
5. *(Optional)* Enter a **Manual Email** if automation failed to detect one

The workflow continues automatically based on your decision.

---

## Output Data (Google Sheets)

The workflow populates a connected Google Sheet with structured lead data.

| Category | Columns |
|----------|---------|
| **Company Info** | Company Name, Industry, Location, Website, LinkedIn URL |
| **Contact Info** | Verified Email, Confidence Score |
| **AI Content** | Personalized Email Draft, Social Media Post, Hashtags |
| **Status** | Lead Found → Company Found → SENT |

---

## API Credentials Reference

| Service | Type | Where to Get It |
|---------|------|-----------------|
| Serper.dev | Header Auth (`X-API-KEY`) | [serper.dev](https://serper.dev) |
| Hunter.io | Query Param (`api_key`) | [hunter.io](https://hunter.io) |
| Groq AI | Header Auth (`Bearer`) | [console.groq.com](https://console.groq.com) |
| Google Sheets | OAuth2 | Google Cloud Console |
| Gmail | OAuth2 | Google Cloud Console |

### Free Tier Limits

| Service | Free Limit | What Happens When Exceeded |
|---------|------------|----------------------------|
| Serper.dev | ~100 searches/month | Empty search results / errors |
| Hunter.io | 25 searches/month | Email verification returns null |
| Groq AI | Rate limited (RPM) | 429 Too Many Requests error |
| Gmail | 500 emails/day | Account may be temporarily suspended |

> **Pro Tip:** For testing, configure your Loop node to process only 1–3 items at a time to conserve API credits.

---

## Docker Configuration

The `docker-compose.yml` is pre-configured to:

- Load all API keys from `.env` automatically
- Allow `$env.*` variable access inside n8n nodes (`N8N_BLOCK_ENV_ACCESS_IN_NODE=false`)
- Allow `$env.*` in n8n expressions (`N8N_BLOCK_ENV_ACCESS_IN_EXPRESSIONS=false`)
- Persist workflow data across restarts via Docker volume (`n8n_data`)
- Store local files in `./n8n_local/`

---

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| `access to env vars denied` | Old container running without the flag | Restart with `docker-compose down` then `docker-compose up -d` |
| `$env.GOOGLE_SHEET_ID` error | Placeholder value in `.env` | Replace with actual Sheet ID from your Google Sheets URL |
| `&&` not working in terminal | Windows PowerShell limitation | Run commands separately: `docker-compose down` then `docker-compose up -d` |
| Workflow not visible after restart | Fresh Docker volume | Re-import `agentic-ai-mkt workflow.json` from the project root |
| OAuth2 nodes fail | Credentials not configured | Add Google Sheets + Gmail credentials in n8n Credentials section |
