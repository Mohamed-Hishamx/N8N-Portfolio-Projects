# 🚗 Quicky Car AI Support Chatbot

An AI-powered customer support chatbot for **Quicky Car Egypt**, a car rental platform. Built with **n8n**, **Supabase Vector Store**, **OpenRouter**, and **HuggingFace** — enabling customers to ask about pricing, rental policies, branch locations, and more in real time.

> Built by [Apex Cycle](https://apexcycle.io) — AI & Automation Agency

---

## 🧠 How It Works

```
User (Chat UI)
     │
     ▼ HTTP POST (message + sessionId)
n8n Webhook
     │
     ▼
AI Agent (OpenRouter LLM)
     ├── Simple Memory (conversation history per session)
     └── Supabase Vector Store Tool (RAG knowledge retrieval)
              │
              └── HuggingFace Embeddings (all-MiniLM-L6-v2)
     │
     ▼
Respond to Webhook → { "reply": "..." }
     │
     ▼
Chat UI renders response
```

**Retrieval-Augmented Generation (RAG):** The agent only answers from the ingested knowledge base — it won't hallucinate policies or prices.

**Session Memory:** Each browser tab gets a unique `sessionId`, keeping multi-turn conversations coherent.

---

## 📁 Project Structure

```
├── chatui.html                          # Frontend chat interface (standalone HTML)
├── Knowledge_Base_Ingestion.json        # n8n workflow — AI agent + webhook + RAG
├── Quicky_Car_Knowledge_Base_Ingestion.json  # n8n workflow — one-time KB ingestion
└── README.md
```

---

## 🛠️ Tech Stack

| Layer | Tool |
|---|---|
| Workflow Automation | [n8n](https://n8n.io) |
| LLM | OpenRouter (any model, e.g. GPT-4o, Claude) |
| Vector Store | Supabase (pgvector) |
| Embeddings | HuggingFace — `sentence-transformers/all-MiniLM-L6-v2` |
| Memory | n8n Buffer Window Memory |
| Frontend | Vanilla HTML + Tailwind CSS |

---

## ⚙️ Setup Guide

### Prerequisites

- n8n instance (self-hosted or cloud)
- Supabase project with `pgvector` enabled
- OpenRouter API key
- HuggingFace API key

---

### Step 1 — Supabase Setup

Run the following SQL in your Supabase SQL Editor to create the required tables and the similarity search function:

```sql
-- Enable pgvector
create extension if not exists vector;

-- Documents table
create table documents (
  id bigserial primary key,
  content text,
  metadata jsonb,
  embedding vector(384)
);

-- Similarity search function (used by n8n Vector Store node)
create or replace function match_documents (
  query_embedding vector(384),
  match_count int default 5,
  filter jsonb default '{}'
)
returns table (
  id bigint,
  content text,
  metadata jsonb,
  similarity float
)
language plpgsql
as $$
begin
  return query
  select
    documents.id,
    documents.content,
    documents.metadata,
    1 - (documents.embedding <=> query_embedding) as similarity
  from documents
  where documents.metadata @> filter
  order by documents.embedding <=> query_embedding
  limit match_count;
end;
$$;
```

> The embedding dimension `384` matches `all-MiniLM-L6-v2`. Change it if you switch models.

---

### Step 2 — Configure n8n Credentials

In your n8n instance, add the following credentials:

| Credential Type | Where to get it |
|---|---|
| **OpenRouter API** | [openrouter.ai/keys](https://openrouter.ai/keys) |
| **Supabase API** | Project Settings → API in your Supabase dashboard |
| **HuggingFace API** | [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens) |

---

### Step 3 — Import n8n Workflows

1. In n8n, go to **Workflows → Import from File**
2. Import `Quicky_Car_Knowledge_Base_Ingestion.json` first
3. Import `Knowledge_Base_Ingestion.json` second
4. Assign your saved credentials to the nodes in each workflow

---

### Step 4 — Ingest the Knowledge Base

1. Open the **Quicky Car - Knowledge Base Ingestion** workflow
2. Click **Execute Workflow** (manual trigger)
3. This chunks and embeds the policy document into Supabase — run this once, or whenever you update your knowledge base

---

### Step 5 — Configure the Chat UI

Open `chatui.html` and update the webhook URL:

```javascript
// Line ~50 in chatui.html
const WEBHOOK_URL = "YOUR_N8N_WEBHOOK_URL_HERE";
```

Replace with your n8n **Production Webhook URL** from the `chat` webhook node.

Then open the HTML file in any browser — no build step required.

---

### Step 6 — Activate the Agent Workflow

1. Open the **Knowledge Base Ingestion** workflow in n8n
2. Toggle it to **Active**
3. Test by sending a message in the chat UI

---

## 💬 Example Conversations

> **User:** What's the price to rent a Kia Sportage?  
> **AI:** The Kia Sportage (Automatic) is available at 1,500 EGP per day. It seats 5, fits 4 bags, and includes Apple CarPlay and a panoramic sunroof. 🚙

> **User:** Do you have 24/7 roadside assistance?  
> **AI:** Yes! Quicky Car offers free 24/7 roadside assistance within Cairo and Giza. If your car breaks down, a replacement vehicle of the same class or higher will be delivered within 90 minutes.

> **User:** I'm 22, can I rent?  
> **AI:** Yes, the minimum age is 21. However, since you're under 25, a Young Driver Fee of 150 EGP per day will apply.

---

## 🔐 Security Notes

- **Never commit API keys or credential IDs** to this repository
- All credential IDs in the workflow JSONs have been replaced with placeholder strings (`YOUR_*_CREDENTIAL_ID`)
- The webhook path has been replaced with `YOUR_WEBHOOK_PATH` — set a unique path in n8n
- Use n8n's built-in credential vault to store all secrets

---

## 📄 Knowledge Base

The knowledge base (`Edit Fields` node in the ingestion workflow) currently covers:

- **Fleet & Pricing** — Economy, SUV, Luxury classes with daily EGP rates
- **Rental Requirements** — Age, license, Young Driver Fee
- **Security Deposits** — By vehicle class, credit card only
- **Fuel Policy** — Full-to-full, refueling fee details
- **Branch Locations** — Airport (24/7), Nasr City, Zayed City
- **Late Return Policy** — 2-hour grace period
- **Insurance** — Basic coverage + Zero Deductible Shield option
- **Roadside Assistance** — 24/7, 90-minute replacement guarantee

To update the KB, edit the text in the `Edit Fields` node and re-run the ingestion workflow.

---

## 🚀 Built by Apex Cycle

This project is part of Apex Cycle's AI automation portfolio.  
Interested in a similar solution for your business? Reach out via [apexcycle.io](https://apexcycle.io)
