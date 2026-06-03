# Autonomous Revenue Engine

An n8n workflow that turns inbound leads into qualified sales conversations — automatically.

![n8n](https://img.shields.io/badge/n8n-workflow-EA4B71) ![Vapi](https://img.shields.io/badge/Voice%20AI-Vapi.ai-5A67D8) ![OpenAI](https://img.shields.io/badge/LLM-GPT--4o-412991) ![CRM](https://img.shields.io/badge/CRM-HubSpot%20%7C%20Salesforce-FF7A59) ![Supabase](https://img.shields.io/badge/Database-Supabase-3ECF8E)

---

## What it does

A lead fills out a form → a voice AI calls them 30 seconds later → GPT-4o scores the call → the CRM gets updated → if the score is 60+, sales gets a Slack alert with a calendar link. Everything is logged. If anything breaks, the workflow diagnoses and recovers itself.

Three sub-flows running in one system:

**Sub-flow 1 — Main Sales Pipeline**
Form submission → data normalization → CRM routing (HubSpot or Salesforce based on company size and budget) → 30-day recency check → Vapi outbound call

**Sub-flow 2 — Post-Call Intelligence**
Vapi webhook → GPT-4o transcript analysis → qualification score 0-100, sentiment, budget, timeline, pain points, next action → CRM update → Supabase log → Slack hot lead alert or cold lead notice

**Sub-flow 3 — Self-Healing Error Handler** (separate workflow)
Error Trigger → JS error classifier → GPT-4o-mini diagnosis → recovery routing: wait and retry / auto data correction / Gmail fallback / Slack escalation → Supabase error log

---

## Stack

| Layer | Tool |
|---|---|
| Orchestration | n8n |
| Voice AI | Vapi.ai |
| LLM — call analysis | OpenAI GPT-4o |
| LLM — error diagnosis | OpenRouter (GPT-4o-mini) |
| CRM | HubSpot or Salesforce (one variable switches between them) |
| Database | Supabase |
| Notifications | Slack + Gmail |

---

## Files

| File | Description |
|---|---|
| `Autonomous_Revenue_Engine_CLEAN.json` | Main workflow — Sub-flows 1 and 2 |
| `Error_Handler_CLEAN.json` | Error Handler — Sub-flow 3 |

---

## Setup

### 1. Supabase — create the tables

Go to Supabase → SQL Editor and run:

```sql
CREATE TABLE IF NOT EXISTS call_logs (
  id SERIAL PRIMARY KEY,
  call_id TEXT, crm_ref TEXT, crm_target TEXT,
  qualification_score INT, sentiment TEXT, budget TEXT, timeline TEXT,
  next_action TEXT, summary TEXT, transcript TEXT, recording_url TEXT,
  duration_seconds INT, analyzed_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS error_logs (
  id SERIAL PRIMARY KEY,
  node_name TEXT, error_type TEXT, error_message TEXT,
  recovery_strategy TEXT, diagnosis TEXT, severity TEXT,
  can_auto_recover BOOLEAN, detected_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 2. HubSpot — create custom contact properties

Go to HubSpot → Settings → Properties → Contact and create:

| Property name | Type |
|---|---|
| qualification_score | Number |
| last_call_sentiment | Single-line text |
| last_call_summary | Multi-line text |
| last_call_next_action | Single-line text |
| last_call_budget | Single-line text |
| last_call_timeline | Single-line text |
| last_call_at | Date |

### 3. Vapi — configure the assistant

- Create an assistant in the Vapi dashboard
- Set the server URL to your n8n Vapi webhook URL: `https://your-n8n-domain/webhook/vapi-webhook`
- Add a phone number (buy from Vapi or import from Twilio)

### 4. Import into n8n

- Import `Autonomous_Revenue_Engine_CLEAN.json` as a new workflow
- Import `Error_Handler_CLEAN.json` as a separate workflow
- In the main workflow → Settings → Error Workflow → select the Error Handler workflow
- Connect all credentials (HubSpot, Salesforce, Slack, OpenRouter, Gmail)
- Replace all `YOUR_*` placeholders with your actual values
- Activate both workflows

### 5. Replace placeholders

Search the JSON files for these and replace with your actual values:

```
YOUR_VAPI_API_KEY
YOUR_VAPI_ASSISTANT_ID
YOUR_VAPI_PHONE_NUMBER_ID
YOUR_HUBSPOT_CRED_ID
YOUR_SALESFORCE_CRED_ID
YOUR_OPENROUTER_CRED_ID
YOUR_SUPABASE_URL
YOUR_SUPABASE_SERVICE_KEY
YOUR_ERROR_WORKFLOW_ID
```

---

## CRM Routing Logic

The workflow decides which CRM to use based on the form submission:

- Company size `200+` OR monthly budget `$20k+` → **Salesforce**
- Everything else → **HubSpot**

To change the logic, edit the `isEnterprise` condition in the **Data Normalizer** node.

---

## Test the pipeline

Submit a test lead using the n8n form trigger URL (shown when the workflow is active), or send a POST request:

```bash
curl -X POST https://your-n8n-domain/webhook/lead-incoming \
  -H "Content-Type: application/json" \
  -d '{
    "first_name": "Test",
    "last_name": "Lead",
    "email": "test@example.com",
    "phone": "+15551234567",
    "company": "Acme Corp",
    "source": "website"
  }'
```

---

## Built by

[Apex Cycle](https://github.com/Mohamed-Hishamx) — AI automation systems for sales and operations
