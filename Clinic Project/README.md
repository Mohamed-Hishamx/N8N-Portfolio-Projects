# ADHD Clinic — Complete n8n Automation (Final Version)
## Two Workflows — Full Project Delivery

---

## Files
| File | Milestone | What it does |
|------|-----------|--------------|
| `MILESTONE_1_email_triage.json` | Email auto-reply, AI classification, safety flagging, Sheets logging, Drive folders |
| `MILESTONE_2_v2_google_calendar_booking.json`| Full Google Calendar booking system — no third-party platform needed |

---

## How the Full System Works End-to-End

```
PATIENT SENDS EMAIL
        ↓
┌─────────────────────────────────┐
│        BOTH WORKFLOWS           │
│  Gmail Trigger → Safety Filter  │
│  → AI Intent Detection          │
└─────────────────────────────────┘
        ↓
  What type of email?
  │
  ├── General Enquiry (ADHD info, follow-up, medication, GP)
  │         → MILESTONE 1: AI classifies → picks template
  │           → personalises → auto-replies → logs to Sheet
  │           → creates Drive folder
  │
  └── Booking Request (patient wants an appointment)
            → MILESTONE 2: reads consultant's Google Calendar
              → finds next 3 'Available' slots
              → sends patient 3 options to choose from
              → patient replies "Option 1"
              → system books it in Google Calendar
              → removes the slot (prevents double booking)
              → sends confirmation email to patient
              → logs to Bookings sheet
```

---

## Consultant Calendar Convention
Consultants must mark open time in Google Calendar using this naming:

| Calendar Event Title | What it means |
|----------------------|---------------|
| `Available` | Open for patient bookings |
| `Available 10:00–12:00` | Specific open window |
| `Blocked — Personal` | Do not book |
| `OOO` / `Holiday` | Do not book |

The system searches for events with 'Available' in the title.

---

## Google Sheets Setup

### Tab 1: Enquiries (Milestone 1)
```
Timestamp | Date | Time | Name | Email | Category | AI Confidence | Status | Auto Replied | Subject | AI Reasoning | Location | GP Name | Is Child | Drive Folder
```

### Tab 2: Bookings (Milestone 2)
```
Timestamp | Patient Name | Patient Email | Appointment Type | Preferred Date | Status | Slot 1 | Slot 2 | Slot 3 | Thread ID | Calendar Event ID | Notes
```

---

## All REPLACE_ Values Checklist

**Credentials (set once in n8n Settings → Credentials):**
- Gmail OAuth2
- Google Calendar OAuth2
- Google Sheets OAuth2
- Google Drive OAuth2
- OpenAI API Key

**In workflow nodes:**
- `REPLACE_GMAIL_CREDENTIAL_ID`
- `REPLACE_GOOGLE_CALENDAR_CREDENTIAL_ID`
- `REPLACE_GOOGLE_SHEETS_CREDENTIAL_ID`
- `REPLACE_GOOGLE_DRIVE_CREDENTIAL_ID`
- `REPLACE_OPENAI_CREDENTIAL_ID`
- `REPLACE_ADMIN_EMAIL@yourclinic.co.uk`
- `REPLACE_CLINIC_EMAIL@yourclinic.co.uk`
- `REPLACE_WITH_GOOGLE_SHEET_ID`
- `REPLACE_WITH_CONSULTANT_CALENDAR_ID`
- `REPLACE_WITH_PARENT_DRIVE_FOLDER_ID`
- `REPLACE_WITH_CLINIC_NAME`
- `REPLACE_WITH_CLINIC_ADDRESS`
- `REPLACE_WITH_PHONE_NUMBER`
- `REPLACE_WITH_WEBSITE`
- `REPLACE_WITH_PREPARATION_INSTRUCTIONS`

---

## GDPR Compliance Summary
- Patient data flows only through: Gmail, Google Calendar, Google Sheets, Google Drive, OpenAI
- OpenAI only receives email subject + body (not used for training on API plan)
- n8n execution logs: set retention to 30 days (Settings → Executions)
- All Google services covered by Google Workspace DPA
- No data stored in any other third-party system
