# Automated Lead Capture & Notification System

A production n8n workflow that captures inbound leads, scores them, checks for duplicates, saves to Airtable, and instantly notifies the team on Telegram — fully automated, zero manual steps.

---

## What it does

When someone fills out a form, the workflow:

1. **Validates** the submission — rejects anything missing a name or contact info before it touches the database
2. **Scores the lead from 0–100** based on data quality (has phone? detailed message? email provided?) and classifies it as High Priority / Medium Priority / Low Priority
3. **Checks for duplicates** — searches Airtable by email AND phone before writing anything
4. **Creates or updates** the Airtable record depending on whether the lead already exists
5. **Notifies the team on Telegram** instantly with the lead's name, contact, score, tier, and whether they're new or returning
6. **Responds to the webhook** so the form gets a proper confirmation back

---

## The Telegram message looks like this

```
High Priority Lead — New

👤 Name: Ahmed Al-Rashid
📧 Email: ahmed@example.com
📞 Phone: +971 50 123 4567
💬 Message: I'm looking for a 2BR apartment in downtown...
📊 Score: 80/100
🔗 Source: Website Form

✅ Saved to records.
```

---

## Stack

| Piece | What it's doing |
|---|---|
| n8n | Workflow engine |
| Webhook | Receives form submissions |
| Airtable | Stores and deduplicates leads |
| Telegram | Real-time team notifications |
| JavaScript (Code node) | Validation, scoring, duplicate logic |

---

## Files

| File | Description |
|---|---|
| `Automated_Lead_Capture.json` | Original version — basic capture and notify |
| `Automated_Lead_Capture_V2.json` | Current version — adds validation, scoring, and duplicate handling |

---

## Scoring logic

| Signal | Points |
|---|---|
| Has phone number | +30 |
| Has email | +20 |
| Message longer than 80 characters | +30 |
| Message longer than 20 characters | +10 |
| Any message at all | +10 |

- **70–100 → High Priority** — follow up within the hour
- **40–69 → Medium Priority** — follow up same day
- **0–39 → Low Priority** — add to nurture sequence

---

## How to use it

1. Import the V2 JSON into n8n via **Settings → Import Workflow**
2. Connect your Airtable and Telegram credentials
3. Activate the workflow and copy the webhook URL
4. Point your form's submission endpoint to that URL
5. Submit a test — you should see the Airtable record and Telegram message come through within seconds

---

## Airtable setup

Your Leads table needs these columns: `Name`, `Email`, `Phone`, `Message`, `Lead Source`, `Status`

Status options: `New`, `Re-engaged`, `In Progress`, `Closed`

---

Built by [Mohamed Hisham](mailto:MohameddHisham@outlook.com) — Automation & Workflow Engineer
