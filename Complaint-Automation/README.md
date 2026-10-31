# CFPB Complaint Automation (n8n + AI Classification)

## 📌 Project Overview

This project is an end-to-end data automation workflow built with **n8n** that processes consumer complaint data, classifies complaints using an **AI language model**, and routes them into structured outputs for downstream analysis.

The workflow demonstrates how unstructured text data can be reliably ingested, validated, classified, and stored using automation-first design principles.

---

## 🎯 Business Use Case

Organizations that handle consumer complaints (e.g., financial institutions, regulators, customer support teams) often receive large volumes of free-text submissions.

Manually reviewing and categorizing these complaints is:
- Time-consuming
- Inconsistent
- Difficult to scale

This automation solves the problem by:
- Automatically classifying complaints using AI
- Enforcing consistent categorization rules
- Producing structured, analytics-ready outputs

---

## 🧠 Workflow Logic (Step-by-Step)

1. **Webhook Ingestion**
   - Accepts incoming complaint data via HTTP POST
   - Supports batched payloads

2. **Data Normalization**
   - Splits payloads into individual complaint records
   - Validates required fields

3. **AI Classification**
   - Uses an LLM via OpenRouter
   - Classifies each complaint into exactly one category:
     - Mortgage Issue
     - Credit Card Problem
     - General Inquiry

4. **Conditional Routing**
   - Routes each complaint based on its assigned category

5. **Structured Storage**
   - Appends results into category-specific Google Sheets
   - Enables easy review and reporting

---

## 🛠 Tech Stack

- n8n – Workflow orchestration
- OpenRouter (LLM) – AI-based classification
- Google Sheets – Structured output storage
- HTTP Webhooks – Real-time ingestion

---

## 🗂 Output Schema

Each stored record includes:
- Original complaint text
- AI-assigned category
- Ingestion timestamp
- Request metadata

Complaints are separated by category for clarity and analysis.

---

## 🧪 Validation & Error Handling

- Invalid or empty complaints are filtered
- AI output is constrained to predefined categories
- Workflow can be extended with retries, alerts, or logging

---

## 🚀 How to Run the Workflow

1. Import the workflow JSON into n8n
2. Configure credentials:
   - OpenRouter API key
   - Google Sheets OAuth
3. Activate the workflow
4. Send POST requests to the webhook endpoint

### Example Payload

```json
{
  "body": [
    {
      "complaint_what_happened": "I am having issues with my mortgage payments."
    }
  ]
}
```

---

## ⭐ Portfolio Highlights

- Real-world AI-powered automation
- Unstructured to structured data transformation
- Clear business relevance
- Scalable and auditable workflow design

---

## 🔮 Possible Enhancements

- Store data in PostgreSQL instead of Sheets
- Add sentiment analysis
- Add SLA tracking
- Build BI dashboards on top of outputs

---

## 📎 Notes

This project is designed as a portfolio demonstration and follows patterns commonly used in production automation systems.
