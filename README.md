# Agentic AI Marketing Workflow - n8n

**Version:** 1.0  
**Built with:** n8n, Groq (Llama3), Serper (Google Search), Hunter.io, Google Workspace

## Project Overview

This n8n workflow automates B2B lead generation and outreach. It performs the following autonomous steps:

1. **Scrapes Google** to find companies in a specific industry and city.
2. **Verifies Emails** using Hunter.io to ensure high deliverability.
3. **Human-in-the-Loop Review:** A manual approval step to accept/reject leads.
4. **Generates Personalized Emails** using AI (Groq/Llama3).
5. **Sends Emails** via Gmail (OAuth2).
6. **Generates LinkedIn Posts** to market the outreach activity.
7. **Database Management:** Automatically saves and updates all data in Google Sheets.

## Setup & Credentials

Before running the workflow, ensure the following credentials are configured in n8n:

### 1. Google Cloud (Sheets & Gmail)
- **Type:** OAuth2
- **Scopes Required:**
  - `https://www.googleapis.com/auth/spreadsheets` (Read/Write)
  - `https://www.googleapis.com/auth/gmail.send` (Send emails)
- **Setup:** Create a Project in Google Cloud Console, enable Sheets & Gmail APIs, and generate Client ID/Secret.

### 2. Serper.dev (Google Search API)
- **Type:** Header Auth
- **Header Name:** `X-API-KEY`
- **Value:** `[Your_Serper_API_Key]`
- **Purpose:** Used to find company websites and descriptions.

### 3. Hunter.io (Email Finder)
- **Type:** Query Param Auth (or Header)
- **Key:** `api_key`
- **Value:** `[Your_Hunter_API_Key]`
- **Purpose:** Used to find and verify professional email addresses.

### 4. Groq (AI Model)
- **Type:** Header Auth
- **Header Name:** `Authorization`
- **Value:** `Bearer [Your_Groq_API_Key]`
- **Purpose:** Uses llama-3.3-70b to write cold emails and social media posts.

## Free Tier Limitations (Important)

Be aware of the daily/monthly limits for the free services used in this workflow:

| Service | Free Limit | Consequence of Exceeding |
|---------|------------|--------------------------|
| Serper.dev | ~100 searches/month | Search results will return empty errors. |
| Hunter.io | 25 searches/month | Email verification will fail (returns null). |
| Groq AI | Rate limits apply (RPM) | AI generation nodes will error out (429 Too Many Requests). |
| Gmail | 500 emails/day | Account may be temporarily suspended if marked as spam. |

**Pro Tip:** For testing, set your Loop node to process only 1-3 items at a time to save credits.


# How to Run the Workflow

You can trigger this workflow in **two ways** depending on whether you are testing or running it in production.

---

## Method 1: Manual Execution (Testing)

Use this method when developing or testing inside the **n8n editor**.

### Steps

1. Open the **"Edit Fields"** (or **"Manual Trigger"**) node at the start of the workflow.
2. Paste the following JSON data structure into the input parameters:

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

3. Click **Execute Workflow**.

This will immediately start the workflow and allow you to debug each node step-by-step.

---

## Method 2: Form Submission (Production)

Use this method to launch the agent via a **user-friendly web form**.

### Steps

1. Ensure the **On Form Submission** trigger node is **active**.
2. Click the **Production URL** (or **Test URL**) in the node settings.
3. Fill in the form fields:

   * **Industry:** (e.g., Software)
   * **City:** (e.g., Colombo)
   * **Description:** (e.g., *We sell AI services...*)
4. Submit the form.

Once submitted, the workflow will run automatically in the background.

---

# How to Handle the "Human Approval" Loop

During execution, the workflow will **pause** at the **"Wait for Approval"** node for **every lead found**.

### Steps to Approve or Reject Leads

1. Go to **Executions** in the n8n sidebar.
2. Open the **Pending Execution** (you will see a spinning or paused status).
3. Click the **Form URL** provided in the execution output.
4. Choose one of the following actions:

   * **Yes – Save to Sheet** → Processes and stores the lead
   * **No – Discard** → Skips the lead
5. *(Optional)* Enter a **Manual Email** if the automation failed to detect one.

The workflow will continue automatically based on your decision.

---

# Output Data (Google Sheets)

The workflow automatically populates a connected **Google Sheet** with structured lead data.

### Stored Columns

#### Company Information

* Company Name
* Industry
* Location
* Website
* LinkedIn URL

#### Contact Information

* Verified Email
* Confidence Score

#### AI-Generated Content

* Personalized Email Draft
* Social Media Post
* Hashtags

#### Status Tracking

* Lead Found → Company Found → SENT

---

### Metadata

* **Created by:** Niwantha Sithumal
* **Date:** 2026-01-27
