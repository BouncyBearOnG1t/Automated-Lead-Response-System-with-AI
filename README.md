# Automated Lead Response System with AI (n8n + Ollama)

This project automates lead capture, scoring, CRM updates, and personalized email follow-up using n8n, a local LLM via Ollama, Google Sheets, HubSpot, and Gmail.

![Workflow overview](screenshots/workflow.png)


## Problem Statement

Manually handling inbound leads from forms is slow and inconsistent:

- New leads sit in inboxes before anyone responds.
- Lead quality (budget, use case, company size) is not evaluated consistently.
- Data is scattered across the form tool, spreadsheets, and CRM.
- Prospects receive generic emails instead of tailored responses.

This workflow turns that manual process into a repeatable, automated pipeline.

## Solution Overview

Whenever a lead submits a form (Tally or any form tool that can POST to a webhook), the workflow:

1. Captures the submission via n8n Webhook.
2. Normalizes raw fields into a clean schema (name, email, company, use case, budget, etc.).
3. Sends the structured data to a local LLM (Gemma 3 via Ollama) to:
   - Score the lead and assign a tier (A/B/C).
   - Draft a personalized email subject and body.
4. Logs all leads and scores to Google Sheets for tracking.
5. Creates or updates the contact in HubSpot with key attributes.
6. Sends a personalized follow-up email via Gmail.

The result: every lead gets a quick, consistent, and contextual reply without manual effort.

## Architecture

High-level architecture:

- **Trigger:** Webhook (n8n) receiving form submissions (e.g., Tally).
- **Pre-processing:** Edit Fields node to rename and clean incoming fields.
- **AI Layer:** Message a Model node calling local Ollama (Gemma 3) to generate:
  - `score` (0–100)
  - `tier` (`A`, `B`, `C`)
  - `email_subject`
  - `email_body`
- **Transformation:** Code in JavaScript node to parse the LLM JSON and shape final output.
- **Storage & Analytics:** Google Sheets node appending or updating a row.
- **CRM:** HubSpot node to create or update a contact.
- **Communication:** Gmail node to send the personalized email.

Workflow diagram (simplified):

`Webhook → Edit Fields → Message a model → Code in JavaScript → Sheets → HubSpot → Gmail`

## Tech Stack

- **n8n** – workflow automation engine
- **Ollama + Gemma 3** – local LLM for lead scoring and email generation
- **Tally (or any form tool)** – lead capture
- **Google Sheets** – lead log and simple analytics
- **HubSpot** – CRM for contacts
- **Gmail** – sending personalized follow-up emails

## Files

- `Automated Lead Response System with AI.json` – exported n8n workflow (no credentials included).
- `screenshots.png` – workflow overview.
- `sample-data/sample-tally-submission.json` – anonymized sample payload for testing.


## Getting Started

### Prerequisites

- n8n instance (cloud or self-hosted).
- Ollama running locally with a Gemma 3 model pulled.
- Google account with access to Google Sheets.
- HubSpot account with API/OAuth credentials.
- Gmail account or Google Workspace account.

### Setup Steps

1. **Import the workflow**

   - In n8n, open the editor, click the three dots (**…**) → *Import from file*.
   - Select `workflow.json`.

2. **Configure credentials**

   Recreate the following credentials in your n8n instance:

   - Webhook URL (connect it to your form tool).
   - Ollama (base URL, typically `http://localhost:11434`).
   - Google Sheets OAuth.
   - HubSpot OAuth/API key.
   - Gmail OAuth.

3. **Update node-specific settings**

   - Google Sheets node: set spreadsheet ID and sheet name.
   - HubSpot node: map fields (first name, company, score/tier, etc.).
   - Gmail node: set `from` address and tweak email copy if needed.

4. **Test with sample data**

   - Use the `sample-data/sample-tally-submission.json` payload.
   - Trigger the Webhook manually from n8n or your form tool’s test feature.
   - Confirm entries appear in Sheets, HubSpot, and that a test email is delivered.


## Sample Use Case

- A founder embeds a lead form on their landing page.
- When someone submits interest, the workflow:
  - Scores them based on company size, budget, and use case.
  - Logs them in Sheets with their score and source.
  - Adds/updates the contact in HubSpot.
  - Sends a tailored email (e.g., higher-touch tone for A-tier leads).

## Future Improvements

- Add Slack notifications for A-tier leads.
- Track first-response time by logging email send timestamps.
- Add multi-language support for email generation.
- Enrich lead data via third-party APIs (Clearbit, etc.).
