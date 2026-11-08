# n8n-ai-career-automation-agent
# 🚀 Autonomous AI Career Pipeline & Executive Briefing Agent

An end-to-end autonomous job ingestion, qualification, and outreach pipeline built on **n8n**, **Claude 3.5 / LLM APIs**, **Google Workspace (Sheets, Drive, Gmail)**, and **Multi-Platform Job APIs**.

This system automates role discovery, matches qualifications using advanced LLM reasoning, drafts hyper-personalized outreach emails with attached CVs, and produces a daily executive briefing to prioritize inbound responses.

---

## 📌 Architecture Overview

![Workflow Architecture](workflow-architecture.png)

The project consists of two scheduled autonomous tracks:

### Pipeline A: Ingestion, Matching & Outreach (Runs Every 2 Hours)
1. **Multi-Source Fetching**: Queries RapidAPI / JSearch every 2 hours for newly posted Junior/Entry-level opportunities across 5 distinct domains: *Product Operations, Data Analysis, QA, Project Management, and Operations*.
2. **Deduplication & Recency Filtering**: Normalizes timestamps and filters out duplicate postings or roles older than 7 days.
3. **LLM Evaluation & Pitch Crafting**: Claude evaluates role requirements against candidate qualifications, generates an alignment score ($0-100$), and writes a tailored outreach pitch for high-priority matches ($Score \ge 85$).
4. **Data Persistence**: Logs candidate match records, company metadata, and application status directly into Google Sheets.
5. **Asset Distribution & Staging**: Conditionally checks for recruiter email availability, streams the resume PDF directly from Google Drive as a binary asset, and creates a pre-staged draft in Gmail ready for manual approval.

### Pipeline B: Executive Briefing & Inbox Triage (Sun–Thu at 16:00)
1. **Inbox Scanning**: Queries incoming Gmail messages from the past 24 hours for recruiter interactions.
2. **Pipeline Status Reconciliation**: Reads current pipeline updates directly from the Google Sheets tracker.
3. **Structured Synthesis**: Claude produces a daily Hebrew briefing categorized by urgency (🔴 High, 🟡 Medium, 🟢 Low) and highlights drafted outreach hooks awaiting review.

---

## 🛠️ Tech Stack & Integrations

* **Workflow Orchestration**: n8n
* **LLM Reasoning Engine**: Anthropic Claude API (LangChain Agent Architecture)
* **Storage & Persistence**: Google Sheets API
* **Asset Storage**: Google Drive API (Binary stream extraction)
* **Communication**: Gmail API (Draft generation & notification dispatch)
* **Data Sources**: RapidAPI JSearch / Job Board Webhooks

---

## 🧠 Agent Prompts & System Architecture

### 1. Matcher & Outreach Engine (`Claude Agent (Match All 5 Roles)`)

```text
System: You are an autonomous executive talent acquisition agent acting on behalf of [CANDIDATE_NAME].
Your task is to analyze raw job listings and evaluate suitability against the candidate's core profile:

Candidate Background:
- Academic: Degree in Economics & Hi-Tech.
- Technical Skills: Python, SQL, n8n workflow design, LangChain, API integrations, data parsing, machine learning fundamentals.
- Target Roles: Junior Data Operations, Product Operations, Technical QA Analyst, Project Specialist.
- Key Strengths: Rapid workflow automation, data-driven optimization, cross-functional collaboration.

Evaluation Process:
1. Review raw job specifications (title, qualifications, requirements, company description).
2. Compute an alignment score (0 to 100).
3. If Score >= 85:
   - Identify recruiter or hiring manager contact details if present in the posting.
   - Craft a concise, high-impact outreach hook highlighting relevant project experience and technical fit.
   - Generate an engaging, professional email subject line.

Output Schema:
Return ONLY a valid JSON object without surrounding markdown fences or conversational preambles:
{
  "match_score": <integer 0-100>,
  "role_title": "<string>",
  "company": "<string>",
  "is_fit": <boolean>,
  "recruiter_email": "<string or null>",
  "email_subject": "<string>",
  "outreach_email": "<string>"
}
```

### 2. Daily Executive Briefing Engine (`Claude Agent (Generate Daily Digest)`)

```text
System: You are [CANDIDATE_NAME]'s Executive Career Digest Agent.
Format the daily digest cleanly in Hebrew across all 5 career tracks (Product, Ops, Data, QA, Project Management).

Structure:
1) Job applications overview by domain
2) Pipeline & interview updates (reconciled with tracker database)
3) Inbox triage ranked by urgency (🔴 High, 🟡 Medium, 🟢 Low)
4) Ready hook drafts in Gmail awaiting approval

Input Data:
- Last 24h Gmail messages
- Google Sheets active pipeline rows

Tone: Concise, professional, executive, and action-oriented.
```

---

## ⚙️ Setup & Deployment Guide

### 1. Prerequisites
* Running n8n instance (n8n Cloud or self-hosted Docker).
* Google Cloud Console OAuth2 credentials with scopes:
  * `https://www.googleapis.com/auth/spreadsheets`
  * `https://www.googleapis.com/auth/drive.readonly`
  * `https://www.googleapis.com/auth/gmail.modify`
* Anthropic API Key (or OpenAI / Google Gemini key).
* RapidAPI Key for multi-board job queries.

### 2. Installation
1. Clone this repository:
   ```bash
   git clone https://github.com/<your-username>/n8n-ai-career-automation-agent.git
   cd n8n-ai-career-automation-agent
   ```
2. In n8n, navigate to **Workflows** -> **Import from File**.
3. Select `workflows/automated-job-pipeline-agent.json`.
4. Connect credentials for Google Workspace, Anthropic, and RapidAPI.
5. In the **Download file (Google Drive)** node, update the `File ID` parameter with your hosted PDF resume ID.
6. Toggle **Publish** in the top-right corner to activate automated scheduling.

---

## 🛡️ Privacy & Security Best Practices
* Credentials, secrets, and authorization tokens are managed via n8n encrypted environment storage and never committed to source control.
* Personally Identifiable Information (PII) has been sanitized using placeholders across all documentation and templates.
