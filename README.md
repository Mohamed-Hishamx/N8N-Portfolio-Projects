# N8N-Portfolio-Projects

A collection of n8n workflow exports covering data ingestion, API integration, AI classification, notifications and RAG chatbots. Each folder has its own README with setup notes.

**Status:** these are portfolio and learning builds. The JSON files are exports, and behavior in a live n8n instance may differ. Test with synthetic data before activating any workflow against real contacts, customers or patients.

## Projects

| Project | What it does | Notes |
|---|---|---|
| [Sales Alert](Sales%20Alert) | Daily Google Sheets check, Telegram alert when Technology sales are below target | Alert branch fixed. Sums all matching rows in the sheet, see its README |
| [Task Tracker - WhatsApp Reminders](Task%20Tracker%20-%20WhatsApp%20Reminders) | Groups open tasks per staff member and sends each one a WhatsApp reminder via Twilio | Recipient now comes from the sheet row. Twilio sandbox only |
| [Automated Lead Capture & Notification System](Automated%20Lead%20Capture%20%26%20Notification%20System) | Captures leads, de-duplicates, saves to Airtable, notifies the team | Two versions, see its README |
| [Autonomous Revenue Engine](Autonomous%20Revenue%20Engine) | Lead scoring and follow-up flows plus an error handler workflow | Prototype. Recovery is diagnosis and logging, not automatic replay |
| [CarRentProject](CarRentProject) | RAG chatbot for a car rental business (Supabase pgvector) | Answer quality not formally evaluated |
| [Clinic Project](Clinic%20Project) | Email triage and calendar booking for a clinic | Demo only, no real patient data. Not a clinical safety system |
| [Complaint-Automation](Complaint-Automation) | Classifies CFPB complaints and logs them to Sheets | Demo dataset |
| [SaaS Customer Support AI Automation](SaaS%20Customer%20Support%20AI%20Automation) | Ticket classification and reply drafting from email and webchat | Prototype |
| [gcc-jobs-pipeline](gcc-jobs-pipeline) | Ingests GCC job postings from company career APIs into Postgres | Exported CSV/XLSX files are dated snapshots, not current vacancies |
| [n8n Community Challenge Human in the Loop | May 2026](n8n%20Community%20Challenge%20Human%20in%20the%20Loop%20%7C%20May%202026) | Content pipeline with human approval via Telegram | Challenge entry |

## Importing a workflow

1. In n8n choose **Workflows > Import from file** and pick the JSON from the project folder.
2. Create the credentials the nodes ask for (Google Sheets, Telegram, Twilio, Airtable, Supabase, and so on).
3. Replace personal placeholders such as spreadsheet IDs, chat IDs and phone numbers with your own.
4. Run once manually with test data and check each node output before turning the workflow on.
