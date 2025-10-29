## N8N Automations

This repository contains a collection of reusable n8n workflows for email handling, lead research, LinkedIn outreach, meeting preparation, and a RAG-based agent system with document ingestion.

### Table of Contents
- Agents to Apps - RAG System-Final
- Email Automation
- Google Maps Automation
- LinkedinColdDM
- MeetingAutomationMain

### Requirements
- n8n (self-hosted or n8n Cloud)
- Required credentials and API keys:
  - Google: Gmail OAuth2, Google Sheets OAuth2, Google Sheets Trigger OAuth2
  - Google Gemini (PaLM / Generative Language) API key
  - Supabase (DB + Vector Store) connection and access keys
  - RapidAPI: `youtube-transcript3` (optional, only if you enable it)
  - Telegram Bot API (optional, for Telegram messaging nodes)
  - Serper.dev (`X-API-KEY` for Google Maps search)
  - Tavily (optional, for news/search)

---

### Agents to Apps - RAG System-Final
- Purpose: Ingest documents (PDF/CSV/TXT) into a vector store (Supabase) and answer questions via a Google Gemini-powered chat agent using RAG.
- Features:
  - File upload via webhook: `WebhookExtractor` (POST /upload)
  - File-type specific extraction: PDF/CSV/TXT
  - Text chunking: Recursive Character Text Splitter
  - Embeddings: Google Gemini Embeddings
  - Storage & retrieval: Supabase Vector Store (`documents` table, `match_documents` query)
  - RAG Agent: `AI Agent` uses the document retrieval tool
  - Chat trigger: `When chat message received`
  - Optional YouTube transcript ingestion (RapidAPI) and storing to Supabase (disabled by default)
  - Optional Telegram output
- Required settings:
  - Supabase credentials and `documents` table
  - Google Gemini Chat and Embeddings API
  - (Optional) RapidAPI key, Telegram bot token
- Endpoints:
  - Chat webhook: `ChatWebhook` (POST /upload)
  - Upload webhook: `WebhookExtractor` (POST /upload)

### Email Automation
- Purpose: Monitor Gmail inbox, extract sender name, classify the email, label it, and auto-reply.
- Flow:
  - `Gmail Trigger` polls every minute
  - `Information Extractor` extracts sender name (uses Gemini)
  - `If` branches on presence of a name, prepares greeting, then `Merge`
  - `Collect Infos` gathers email body, messageId, threadId
  - `Mail Classifier` classifies text (Gemini)
  - Gmail labeling (INBOX, CATEGORY_SOCIAL, etc.) and reply
- Required settings: Gmail OAuth2, Google Gemini API

### Google Maps Automation
- Purpose: Search companies on Google Maps, collect contact details, and write to Google Sheets; optionally enrich with Gemini for email/background.
- Flow:
  - Chat trigger starts the Agent; uses `Map Search Tool` (Serper.dev) and `Call 'SheetAutomation'`
  - Google Sheets trigger loops over added rows; HTTP Request asks Gemini for email/background; `Update row in sheet` writes the result
- Required settings: Serper.dev `X-API-KEY`, Google Sheets OAuth2, (optional) HTTP Bearer/Header Auth, Gemini API

### LinkedinColdDM
- Purpose: From Google Sheets person/company data, generate a personalized LinkedIn cold DM icebreaker and write it back to the same row.
- Flow:
  - Manual trigger
  - Read rows from Google Sheets; normalize company name via `CompanyNameChanger`
  - `AI Agent` (Gemini + Structured Output Parser) generates an icebreaker
  - `Update row in sheet` writes the result to `Icebreaker` column (matched by `Linkedin Url`)
- Required settings: Google Sheets OAuth2, Google Gemini API

### MeetingAutomationMain
- Purpose: Enrich company/person info from Cal.com bookings (name + email), perform website analysis and news search, and send a concise email report.
- Flow:
  - `Cal.com Trigger` → `Message a model` (Gemini) extracts email domain/company (JSON output)
  - `AI Agent` tools: Website analysis (external workflow), LinkedIn research (optional), Gemini-based news search
  - Optional Tavily flow to locate LinkedIn URL
  - `Send a message` sends the report via Gmail
- Required settings: Cal.com API, Gmail OAuth2, Google Gemini API, (optional) Tavily, Jina/Authorization headers

---

### Setup
1. Import all `.json` workflow files into n8n.
2. Configure node credentials per environment for each workflow:
   - Gmail, Google Sheets, Google Sheets Trigger, Google Gemini (PaLM), Supabase, Serper.dev, RapidAPI, Telegram, Cal.com, Tavily, etc.
3. Ensure the `documents` table and `match_documents` function exist in Supabase (for the RAG flow).
4. For webhooks, verify the n8n base URL and endpoint paths (e.g., `/upload`).
5. Store environment secrets via n8n Credentials/Environment.

### Usage
- Email Automation: Once activated, Gmail inbox is monitored and replies are sent automatically.
- Google Maps Automation: Start queries via chat or add rows to the configured Google Sheet.
- LinkedinColdDM: Run manually; results are written to the `Icebreaker` column.
- MeetingAutomationMain: Runs on Cal.com bookings and emails a report.
- Agents to Apps - RAG System-Final: POST PDF/CSV/TXT to the `/upload` webhook; use the Chat trigger for questions.

### Notes
- Some nodes are disabled by default (e.g., RapidAPI transcript flow, certain tool workflows). Enable and set credentials as needed.
- Telegram, Tavily, RapidAPI, Serper are optional services; required only if you enable the related nodes.

### License
Add your repository license terms here (e.g., MIT).


